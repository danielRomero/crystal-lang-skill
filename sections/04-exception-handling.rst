Exception Handling
------------------
Crystal's approach to error handling uses exceptions for exceptional cases and question methods for expected failures.

Raising Exceptions:
   Use the `raise` method to create and throw exceptions:

   .. code-block:: crystal

      raise "OH NO!"                    # String message
      raise Exception.new("Some error")  # Exception instance

   Only `Exception` instances or subclasses can be raised.

Custom Exceptions:
   Define custom exceptions by subclassing `Exception` following Crystal stdlib patterns:

   .. code-block:: crystal

      # Standard pattern from Crystal source
      class IndexError < Exception
        def initialize(message = "Index out of bounds")
          super(message)
        end
      end

      class ArgumentError < Exception
        def initialize(message = "Argument error")
          super(message)
        end
      end

      # Exception with cause chaining - common pattern
      class ParseException < Exception
        getter line_number : Int32
        getter column_number : Int32

        def initialize(@line_number : Int32, @column_number : Int32, message : String, @cause : Exception? = nil)
          super("Line #{@line_number}, column #{@column_number}: #{message}")
        end
      end

      # Domain-specific exceptions
      class ValidationError < Exception
        getter field : String

        def initialize(@field : String, message : String)
          super("#{field}: #{message}")
        end
      end

Rescuing Exceptions:
   Use `begin ... rescue ... end` for exception handling:

   .. code-block:: crystal

      begin
        raise "OH NO!"
      rescue
        puts "Rescued!"
      end

      # Output: Rescued!

   Access the rescued exception with a variable:

   .. code-block:: crystal

      begin
        raise "OH NO!"
      rescue ex
        puts ex.message
      end

      # Output: OH NO!

Type-Specific Rescue:
   Rescue specific exception types:

   .. code-block:: crystal

      begin
        raise MyException.new("OH NO!")
      rescue MyException
        puts "Rescued MyException"
      rescue ex : MyOtherException
        puts "Rescued MyOtherException: #{ex.message}"
      end

Multiple Rescue Clauses:
   Handle different exception types:

   .. code-block:: crystal

      begin
        # risky code
      rescue ex1 : MyException
        # only MyException...
      rescue ex2 : MyOtherException
        # only MyOtherException...
      rescue
        # any other kind of exception
      end

   Rescue multiple types with unions:

   .. code-block:: crystal

      begin
        # risky code
      rescue ex : MyException | MyOtherException
        # only MyException or MyOtherException
      end

Else and Ensure:
   Use `else` for no-exception case, `ensure` for cleanup:

   .. code-block:: crystal

      begin
        something_dangerous
      rescue
        # execute if an exception is raised
      else
        # execute if an exception isn't raised
      ensure
        # always execute this (cleanup)
      end

Short Syntax Forms:
   Exception handling has concise syntax for methods and blocks:

   .. code-block:: crystal

      def some_method
        something_dangerous
      rescue
        # execute if an exception is raised
      end

      # One-liner rescue
      content = File.read("config.yml") rescue nil

      # Suffix ensure
      result = some_operation ensure cleanup

Type Inference in Exception Handlers:
   Variables declared inside `begin` get `Nil` type in `rescue`/`ensure`:

   .. code-block:: crystal

      begin
        a = something_dangerous_that_returns_Int32
      ensure
        puts a + 1 # error, undefined method '+' for Nil
      end

Solution:
   Declare outside handler:

   .. code-block:: crystal

      a = 1
      begin
        something_dangerous
      ensure
        puts a + 1 # works
      end

Question Method Convention:
   Methods ending with `?` return nil instead of raising:

   .. code-block:: crystal

      array = [1, 2, 3]
      array[4]?      # => nil (no IndexError)
      array[4]       # Raises IndexError

