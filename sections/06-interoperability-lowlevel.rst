C Bindings (Interoperability)
-----------------------------
Crystal has a simple syntax for binding to C libraries without writing any C code. This is a first-class feature for leveraging the vast ecosystem of existing C libraries.

.. code-block:: crystal

   # Binding to the C standard library's `puts` function
   @[Link("c")]
   lib LibC
      fun puts(s : Char*) : Int32
   end

   LibC.puts("Hello from C!")

Low-Level Primitives
--------------------
Crystal provides direct access to memory layout and pointers for systems programming.

.. code-block:: crystal

   struct Point
      @x : Int32
      @y : Int32
   end

   point = Pointer(Point).malloc(1)
   point.value.x = 10
   point.value.y = 20

   sizeof(Point)     # => 8
   offsetof(Point, @y) # => 4

Compile-Time Features
---------------------

Compile-Time Flags:
   Platform detection and conditional compilation.

   .. code-block:: crystal

      {% if flag?(:unix) %}
        puts "Running on Unix system"
      {% end %}

      {% if flag?(:linux) %}
        puts "Linux-specific code"
      {% elsif flag?(:windows) %}
        puts "Windows-specific code"
      {% end %}

      {% if flag?(:release) %}
        puts "Production build optimizations enabled"
      {% end %}

Available Flags:
   * Platform detection: `flag?(:unix)`, `flag?(:windows)`, `flag?(:linux)`

   * Feature flags: User-defined flags via `-D` or `--define`

   * Compiler options: `flag?(:release)`, `flag?(:debug)`

   * Standard library features: `preview_mt`, `without_openssl`

Unique Crystal Features
~~~~~~~~~~~~~~~~~~~~~~~
Tuple Types:
   Fixed-size, type-safe collections with compile-time guarantees.

   .. code-block:: crystal

      # Type-safe coordinate
      point = {10, 20}
      point[0]  # => 10 (Int32)
      point[1]  # => 20 (Int32)
      point[2]  # Compile error: index out of bounds

      # Tuple unions create position-wise unions
      def process(point : {Int32, Int32} | {String, String})
        # point is either {Int32, Int32} or {String, String}
      end

NamedTuple:
   Hash-like structures with compile-time key checking.

   .. code-block:: crystal

      person = {name: "John", age: 30, city: "NYC"}
      person[:name]   # => "John"
      person[:salary] # Compile error: undefined key

      # Named tuple unions create key-wise unions
      def process(data : {name: String, age: Int32} | {name: String, age: String})
        # data has name: String, age: (Int32 | String)
      end

Symbol Literals:
   Efficient, immutable string-like identifiers.

   .. code-block:: crystal

      # Symbols are interned strings - fast comparisons
      status = :success
      case status
      when :success then "OK"
      when :error   then "FAIL"
      end

Range Operations:
   Rich range syntax with inclusive/exclusive semantics.

   .. code-block:: crystal

      # Inclusive range
      1..5      # => 1, 2, 3, 4, 5

      # Exclusive range
      1...5     # => 1, 2, 3, 4

      # Character ranges
      'a'..'z'   # All lowercase letters

      # Range operations
      (1..10).includes?(5)  # => true
      (1..10).step(2)     # => [1, 3, 5, 7, 9]

String Interpolation:
   Compile-time optimized string building.

   .. code-block:: crystal

      name = "World"
      message = "Hello, #{name}!"  # Compiles to efficient IO operations

      # Multi-line interpolation
      greeting = %(
        Hello, #{name}!
        Welcome to Crystal.
      )

Regex Literals:
   Built-in regular expression support with capture groups.

   .. code-block:: crystal

      email_pattern = /(\w+)@(\w+\.\w+)/
      if match = email_pattern.match("user@example.com")
        match[1]  # => "user"
        match[2]  # => "example.com"
      end

Proc Literals:
   First-class function objects with type safety.

   .. code-block:: crystal

      add = ->(x : Int32, y : Int32) { x + y }
      add.call(2, 3)  # => 5

      # Method references
      [1, 2, 3].map(&.to_s)  # Reference to Int32#to_s

Exception Handling:
   Sophisticated exception handling with type safety.

   .. code-block:: crystal

      begin
        risky_operation
      rescue ex : KeyError
        puts "Missing key: #{ex.key}"
      rescue ex : ArgumentError
        puts "Invalid argument: #{ex.message}"
      rescue
        puts "Unknown error: #{typeof(ex)}"
      end

Short Syntax:
   Implicit `begin/end` in methods and blocks.

   .. code-block:: crystal

      def read_file_safe(path)
        File.read(path) rescue nil  # One-liner rescue
      end

Suffix Forms:
   Concise error handling for simple cases.

   .. code-block:: crystal

      def process_with_cleanup
        risky_operation ensure cleanup
      end

