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

Understanding Test Output:
   Crystal spec displays results with visual indicators and detailed failure information.

   Test Status Indicators:
      Each test is represented by a single character:

      - ``.`` (green) - Test passed
      - ``F`` (red) - Assertion failed (expected vs actual mismatch)
      - ``E`` (red) - Runtime error/exception raised
      - ``*`` (yellow) - Test skipped/pending

   Example Output (Colored):
      .. code-block:: text

         ......F.E..

         Failures:

           1) Calculator #add adds two numbers correctly
              Failure/Error: result.should eq(5)

                Expected: 5
                     got: 4

              # spec/calculator_spec.cr:7

           2) Calculator #divide handles division
              Error: Division by zero (DivisionByZeroError)

                spec/calculator_spec.cr:15:7 in 'divide'
                spec/calculator_spec.cr:20:3 in '->'
                ...

              # spec/calculator_spec.cr:20

         Finished in 2.34 milliseconds
         11 examples, 1 failures, 1 errors

   Non-Colored Output:
      Use ``--no-color`` for CI logs or plain text:

      .. code-block:: bash

         crystal spec --no-color

      Output without ANSI codes:

      .. code-block:: text

         ......F.E..

         Failures:

           1) Calculator #add adds two numbers correctly
              Failure/Error: result.should eq(5)

                Expected: 5
                     got: 4

              # spec/calculator_spec.cr:7

Assertion Failures vs Runtime Errors:
   Assertion Failures (F):
      Occur when an expectation is not met. Crystal shows:

      - Full test description
      - The failing assertion line
      - Expected value
      - Actual (got) value
      - File path and line number (prefixed with ``#``)

      .. code-block:: text

         Failure/Error: user.name.should eq("Alice")

           Expected: "Alice"
                got: "Bob"

         # spec/models/user_spec.cr:15

   Runtime Errors (E):
      Occur when code raises an exception during test execution. Shows:

      - Error type and message
      - Full stack trace
      - Test file and line where error occurred

      .. code-block:: text

         Error: Nil assertion failed (NilAssertionError)

           spec/models/user_spec.cr:15:7 in 'not_nil!'
           spec/models/user_spec.cr:22:3 in '->'
           src/models/user.cr:45:5 in 'find'

         # spec/models/user_spec.cr:22

Locating Failed Tests:
   Crystal always outputs the exact file and line at the end of each failure:

   .. code-block:: text

      # spec/models/user_spec.cr:15

   This format makes it easy to:
   - Jump directly to the failing test in your editor
   - Run a specific test: ``crystal spec spec/models/user_spec.cr:15``
   - Filter by line: ``crystal spec -l 15``

Useful Options for Debugging:
   .. code-block:: bash

      # Verbose mode - show test names as they run
      crystal spec -v

      # Run only specific test by line number
      crystal spec -l 15
      crystal spec --location spec/user_spec.cr:15

      # Run tests matching a pattern
      crystal spec -e "should validate"

      # Stop on first failure
      crystal spec --fail-fast

      # Show full compilation error trace
      crystal spec --error-trace

TAP Format for CI Integration:
   TAP (Test Anything Protocol) provides machine-readable output:

   .. code-block:: bash

      crystal spec --tap

   Example TAP output:

   .. code-block:: text

      TAP version 13
      1..11
      ok 1 - Calculator #add adds two numbers correctly
      ok 2 - Calculator #add handles negative numbers
      ...
      not ok 8 - Calculator #add adds two numbers correctly
        ---
        message: 'Expected: 5, got: 4'
        file: 'spec/calculator_spec.cr'
        line: 7
        ...
      not ok 9 - Calculator #divide handles division
        ---
        message: 'Division by zero'
        file: 'spec/calculator_spec.cr'
        line: 20
        ...

   TAP format is ideal for:
   - CI/CD pipelines (Jenkins, GitHub Actions, GitLab CI)
   - Test result aggregation tools
   - Automated test reporting systems

Exit Codes:
   Crystal spec returns specific exit codes:

   - ``0`` - All tests passed
   - ``1`` - One or more tests failed
   - ``2`` - Compilation error (specs didn't compile)

   Use in scripts:

   .. code-block:: bash

      crystal spec && echo "All tests passed" || echo "Tests failed"

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

Compiler Warnings and Deprecations:
    Always check compiler warnings. Crystal warns about deprecated method usage.

    .. code-block:: crystal

       # BAD - Deprecated syntax
       sleep(0.1)  # Warning: Deprecated ::sleep. Use ::sleep(Time::Span) instead

       # GOOD - Correct syntax
       sleep(0.1.seconds)

       # Other common deprecations
       Time.new(year, month, day)  # Deprecated - use Time.local instead
       File.exists?(path)          # Deprecated - use File.exists? (no 's') instead

    Handling Deprecation Warnings:

    .. code-block:: bash

       # Show all warnings during compilation
       crystal build app.cr --warnings=all

       # Treat warnings as errors (strict mode)
       crystal build app.cr --warnings=all --error-on-warnings

    When you see deprecation warnings:

    1. Read the warning message - it suggests the replacement
    2. Update your code to use the new API
    3. Check the Crystal changelog for migration notes
    4. Update immediately - deprecated methods may be removed in future versions

Static Analysis:
    Ameba is the standard linter for Crystal. Use it on all projects with `.cr` files to maintain code quality and catch common issues.

    Installation (add to shard.yml):

   .. code-block:: yaml

      development_dependencies:
        ameba:
          github: crystal-ameba/ameba
          version: ~> 1.0

   Basic usage:

   .. code-block:: bash

      ameba              # Check all files for issues
      ameba --fix        # Auto-fix correctable issues
      ameba src/         # Check specific directory

   Configuration (.ameba.yml):

   .. code-block:: yaml

      # Exclude common directories
      Excluded:
        - lib/
        - spec/

      # Enable common rules
      Lint/UnusedArgument:
        Enabled: true

      Style/RedundantReturn:
        Enabled: true

   When writing Crystal code:
   - Run `ameba` before committing code
   - Fix all reported issues or use `ameba --fix`
   - Follow the Crystal style guide conventions

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

