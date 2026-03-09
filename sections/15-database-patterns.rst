Database and SQL Patterns for AI Agents
---------------------------------------
Crystal's database ecosystem centers around the `crystal-db` library and database-specific drivers. This section covers essential patterns for safe, efficient database access.

Resource Management: Always Close ResultSets
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**CRITICAL**: Always close ResultSets to prevent connection pool exhaustion. Unclosed ResultSets keep database connections busy, eventually causing `DB::PoolRetryAttemptsExceeded` errors.

The Problem:
   .. code-block:: crystal

      # BAD - Connection leak
      private def execute_query
        SqliteOrm.with_connection do |conn|
          rs = conn.query(sql, args)
          rs.each do |row|
            results << process(row)
          end
          # ERROR: rs never closed!
        end
      end

The Solution:
   .. code-block:: crystal

      # GOOD - Proper cleanup with begin/ensure
      private def execute_query
        SqliteOrm.with_connection do |conn|
          rs = conn.query(sql, args)
          begin
            rs.each do |row|
              results << process(row)
            end
          ensure
            rs.close  # CRITICAL: Always close!
          end
        end
      end

      # BETTER - Block form (auto-closes)
      private def execute_query
        SqliteOrm.with_connection do |conn|
          conn.query(sql, args) do |rs|
            rs.each do |row|
              results << process(row)
            end
          end  # Auto-closes here
        end
      end

SQL Injection Prevention
~~~~~~~~~~~~~~~~~~~~~~~~

**Never interpolate user input into SQL**. Always use parameterized queries. This is a critical security requirement.

The Vulnerability:
   .. code-block:: crystal

      # BAD - SQL injection vulnerability
      private def build_where_clause
        @wheres.map do |condition, _|
          if condition.includes?(' ')
            condition  # ERROR: Raw string used directly!
          else
            "#{condition} = ?"
          end
        end
      end

      # If someone passes: condition: "id = 1 OR 1=1"
      # The raw string is inserted, allowing data access/modification

The Secure Approach:
   .. code-block:: crystal

      # GOOD - Use a proper Condition struct with whitelisted operators
      struct Condition
        VALID_OPERATORS = %w(= != > >= < <= LIKE IN)

        def initialize(@column : String, @operator : String, @value : DB::Any)
          raise ArgumentError.new("Invalid operator: #{@operator}") \
            unless VALID_OPERATORS.includes?(@operator)
        end

        def to_sql
          "#{@column} #{@operator} ?"  # Always parameterized!
        end
      end

      # Usage
      condition = Condition.new("name", "=", user_input)
      sql = "SELECT * FROM users WHERE #{condition.to_sql}"
      conn.exec(sql, condition.value)

Parameterized Query Patterns:
   .. code-block:: crystal

      # Simple parameter
      conn.exec("SELECT * FROM users WHERE id = ?", user_id)

      # Multiple parameters
      conn.exec("SELECT * FROM users WHERE name = ? AND age > ?", name, min_age)

      # Array parameters (IN clause)
      ids = [1, 2, 3]
      placeholders = ids.map { "?" }.join(", ")
      conn.exec("SELECT * FROM users WHERE id IN (#{placeholders})", args: ids)

SQLite-Specific Quirks
~~~~~~~~~~~~~~~~~~~~~~

SQLite is stricter than PostgreSQL/MySQL in several ways. Always test with real SQLite.

OFFSET Requires LIMIT:
   SQLite requires LIMIT when using OFFSET (unlike PostgreSQL/MySQL):

   .. code-block:: crystal

      # ERROR - SQLite requires LIMIT with OFFSET
      sql = "SELECT * FROM users OFFSET 100"
      # SQLite3::Exception: near "OFFSET": syntax error

      # CORRECT - Always include LIMIT
      if offset = @offset
        limit_val = @limit || -1  # -1 means unlimited in SQLite
        sql += " LIMIT #{limit_val} OFFSET #{offset}"
      elsif limit = @limit
        sql += " LIMIT #{limit}"
      end

Connection Context in Transactions
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In a connection pool architecture, you must pass the transaction connection through all method calls to maintain isolation.

The Problem:
   .. code-block:: crystal

      # BAD - Each repository call uses a different connection
      def find(id)
        SqliteOrm.with_connection do |conn|
          # query using conn
        end
      end

      # Inside a transaction...
      SqliteOrm.transaction do |tx|
        user = repo.find(1)   # ERROR: Uses different connection!
        repo.update(user)     # ERROR: Uses different connection!
        # Transaction isolation is broken!
      end

The Solution:
   .. code-block:: crystal

      # GOOD - Accept optional connection parameter
      def find(id : Int64, conn : DB::Database? = nil)
        SqliteOrm.with_connection(conn) do |c|
          # Use c (either passed conn or new one)
          rs = c.query("SELECT * FROM users WHERE id = ?", id)
          # ... process
        end
      end

      # Update QueryBuilder to accept and pass connection
      class QueryBuilder(T)
        def initialize(@model_class : T.class, @connection : DB::Database? = nil)
        end

        def execute
          SqliteOrm.with_connection(@connection) do |conn|
            # Use conn for queries
          end
        end
      end

      # Usage in transaction
      SqliteOrm.transaction do |tx|
        user = repo.find(1, tx)   # Uses transaction connection
        repo.update(user, tx)     # Same connection
      end

Array Parameters: Don't Impose Artificial Limits
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Crystal's DB library handles arrays of any size. Don't write case statements limiting parameters.

The Anti-Pattern:
   .. code-block:: crystal

      # BAD - Artificial limit imposed
      case values.size
      when 0 then conn.exec(sql)
      when 1 then conn.exec(sql, values[0])
      when 2 then conn.exec(sql, values[0], values[1])
      # ... up to 10
      else
        raise "Too many parameters"  # ERROR: Users with 11+ fields can't save!
      end

The Correct Approach:
   .. code-block:: crystal

      # GOOD - Crystal's DB library supports arrays directly
      if values.empty?
        conn.exec(sql)
      else
        conn.exec(sql, args: values)  # Unlimited parameters!
      end

Exception Handling
~~~~~~~~~~~~~~~~~~

Know your exception hierarchy. SQLite has its own exception types separate from crystal-db's generic exceptions.

Common Exceptions:
   .. code-block:: crystal

      # SQLite-specific exceptions
      begin
        conn.exec("INSERT INTO users (email) VALUES (?)", duplicate_email)
      rescue ex : SQLite3::Exception
        # Handle constraint violations, etc.
        if ex.message?.try(&.includes?("UNIQUE constraint failed"))
          raise DuplicateRecordError.new("Email already exists")
        end
      end

      # Generic DB exceptions
      rescue ex : DB::Error
        # Handle connection errors, pool exhaustion, etc.
      end

Testing:
   .. code-block:: crystal

      # Test for the specific exception type
      expect_raises(SQLite3::Exception) do
        repo.insert(duplicate_record)
      end

Best Practices Summary
~~~~~~~~~~~~~~~~~~~~~~

1. **Always close ResultSets** - Use `begin/ensure` or block form
2. **Never interpolate user input into SQL** - Always use parameterized queries
3. **Whitelist operators** - Validate SQL operators against allowed list
4. **Test with real SQLite** - SQLite has specific quirks (OFFSET requires LIMIT)
5. **Pass connection context** - Essential for transaction isolation
6. **Don't limit parameter counts** - Use `args: values` array directly
7. **Use specific exception types** - SQLite3::Exception vs DB::Error
8. **Use block forms** - They auto-close resources
