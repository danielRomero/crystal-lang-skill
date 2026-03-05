Essential Tooling for AI Agents
-------------------------------
Crystal Compiler:
   Core compilation and development tools:

   .. code-block:: bash

      # Compile to executable
      crystal build app.cr                    # Development build
      crystal build app.cr --release          # Optimized build
      crystal build app.cr --single-module     # Single module for distribution

      # Run directly
      crystal run app.cr                      # Compile and run
      crystal eval 'puts "Hello"'             # Execute code directly

      # Type checking
      crystal tool format app.cr               # Format code
      crystal tool expand app.cr               # Show macro expansion
      crystal tool implementations app.cr       # Show method implementations

Testing Framework:
   Built-in testing with Crystal spec:

   .. code-block:: crystal

      require "spec"

      describe Calculator do
        describe "#add" do
          it "adds two numbers" do
            calc = Calculator.new
            result = calc.add(2, 3)
            result.should eq(5)
          end

          it "handles negative numbers" do
            calc = Calculator.new
            result = calc.add(-1, 1)
            result.should eq(0)
          end
        end

        describe "#divide" do
          it "raises error for division by zero" do
            calc = Calculator.new
            expect_raises(DivisionByZeroError) do
              calc.divide(10, 0)
            end
          end
        end
      end

      # Run tests
      # crystal spec

      # Run specific file
      # crystal spec spec/calculator_spec.cr

      # Run with coverage
      # crystal spec --coverage

Dependency Management:
   Shards package manager:

   .. code-block:: crystal

      # shard.yml - project dependencies
      name: my_app
      version: 0.1.0
      targets:
        my_app:
          main: src/my_app.cr
      dependencies:
        lucky:
          github: luckyframework/lucky
          version: ~> 1.0
        crest:
          github: mamantoha/crest
          version: ~> 1.0
      development_dependencies:
          ameba:
            github: crystal-ameba/ameba
            version: ~> 1.0

      # Install dependencies
      shards install

      # Update dependencies
      shards update

      # List installed shards
      shards list

      # Build project
      shards build

Code Formatting:
   Consistent code style:

   .. code-block:: bash

      # Format all Crystal files
      crystal tool format src/

      # Format specific file
      crystal tool format src/main.cr

      # Check formatting without changing
      crystal tool format --check src/

      # Format with custom rules
      crystal tool format --no-colors src/

Static Analysis:
   Ameba linter for code quality:

   .. code-block:: crystal

      # .ameba.yml - configuration
      AllCops:
        Enabled: true
        Exclude:
          - lib/
          - spec/

      Layout/LineLength:
        Enabled: true
        MaxLength: 100

      Style/VerboseBlock:
        Enabled: true

      # Run linter
      ameba src/

      # Fix auto-correctable issues
      ameba --fix src/

      # Generate configuration
      ameba --gen-config

Documentation Generation:
   Built-in documentation tools:

   .. code-block:: crystal

      # Documented class
      # A user management service that handles authentication and authorization.
      #
      # ```
      # service = UserService.new(db_connection)
      # user = service.authenticate("alice", "password")
      # ```
      class UserService
        # Authenticates a user with username and password.
        #
        # *username* - The user's login name
        # *password* - The user's password
        #
        # Returns the authenticated User or raises AuthenticationError
        def authenticate(username : String, password : String) : User
          # Implementation
        end
      end

      # Generate documentation
      crystal docs

      # Serve documentation locally
      crystal docs --serve

Benchmarking:
   Performance measurement tools:

   .. code-block:: crystal

      require "benchmark"

      # Simple benchmark
      Benchmark.ips do |x|
        x.report("array iteration") do
          [1, 2, 3, 4, 5].each { |n| n * 2 }
        end

        x.report("range iteration") do
          (1..5).each { |n| n * 2 }
        end
      end

      # Memory benchmark
      Benchmark.memory do |x|
        x.report("string concatenation") do
          result = ""
          1000.times { |i| result += i.to_s }
        end
      end

      # Custom benchmark
      start_time = Time.monotonic
      # Code to benchmark
      end_time = Time.monotonic
      elapsed = end_time - start_time
      puts "Elapsed: #{elapsed.total_milliseconds}ms"

Debugging:
   Debugging tools and techniques:

   .. code-block:: crystal

      # Debug builds with symbols
      crystal build app.cr -d

      # GDB debugging
      gdb ./app

      # LLDB debugging (macOS)
      lldb ./app

      # Debug prints with compile-time flags
      {% if flag?(:debug) %}
        puts "Debug: variable value = #{variable}"
      {% end %}

      # Using pp for pretty printing
      require "pp"
      pp complex_object

      # Backtrace on error
      begin
        risky_operation
      rescue ex
        puts ex.inspect_with_backtrace
      end

Environment Variables:
   Configuration and deployment:

   .. code-block:: crystal

      # Access environment variables
      ENV["DATABASE_URL"]?               # String?
      ENV["PORT"]?.try(&.to_i) || 3000   # Int32 with default

      # Compile-time environment access
      {% if env("RAILS_ENV") == "production" %}
        puts "Production mode"
      {% else %}
        puts "Development mode"
      {% end %}

      # Environment-specific configuration
      struct Config
        def self.from_env
          new(
            database_url: ENV["DATABASE_URL"] || "sqlite://app.db",
            port: ENV["PORT"]?.try(&.to_i) || 3000,
            workers: ENV["WORKERS"]?.try(&.to_i) || 4,
            debug: ENV["DEBUG"]? == "true"
          )
        end
      end

Build Optimization:
   Release build optimizations:

   .. code-block:: bash

      # Optimized release build
      crystal build app.cr --release --no-debug --progress

      # Single binary distribution
      crystal build app.cr --release --single-module

      # Cross-compilation
      crystal build app.cr --target x86_64-unknown-linux-gnu --release

      # Static linking
      crystal build app.cr --release --static

      # Size optimization
      crystal build app.cr --release --no-debug --single-module --static

