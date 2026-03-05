Methods & Instance Variables for AI Agents
------------------------------------------
Automatic Instance Variable Assignment:
   Crystal eliminates boilerplate with automatic assignment in constructors.

   .. code-block:: crystal

      class Person
        # Automatic assignment with default values
        def initialize(@name : String, @age : Int32 = 0, @email : String? = nil)
        end

        # Property macros with type restrictions
        property email : String?
        getter name : String
        setter age : Int32

        # Factory method pattern
        def self.named(name : String)
          new(name, 0, nil)
        end
      end

Method Overloading:
   Crystal supports method overloading based on argument count, types, and names.

   .. code-block:: crystal

      # Multiple initialize methods - Array pattern
      class Container(T)
        def initialize
          @items = [] of T
        end

        def initialize(initial_capacity : Int)
          if initial_capacity < 0
            raise ArgumentError.new("Negative capacity: #{initial_capacity}")
          end
          @items = Array(T).new(initial_capacity)
        end

        def initialize(size : Int, value : T)
          @items = Array(T).new(size, value)
        end
      end

      # Method overloading with blocks - Enumerable pattern
      def each(& : T ->)
        @items.each { |item| yield item }
      end

      def each
        Iterator.new(@items)
      end

Parameter Validation:
   Validate parameters in constructors and methods.

   .. code-block:: crystal

      class Buffer
        def initialize(capacity : Int)
          if capacity < 0
            raise ArgumentError.new("Negative buffer size: #{capacity}")
          end
          @capacity = capacity
          @size = 0
        end

        def read(slice : Bytes)
          if slice.size > @capacity - @size
            raise IndexError.new("Slice too large")
          end
          # implementation
        end
      end

Initialization Patterns:
   Initialize instance variables to avoid Nil types.

   .. code-block:: crystal

      class Database
        @connection : DB::Connection
        @pool_size : Int32

        def initialize(connection_string : String, @pool_size = 5)
          @connection = DB.connect(connection_string)
        end
      end

      # Lazy initialization pattern
      class Config
        @settings : Hash(String, String)?

        def settings
          @settings ||= load_settings
        end

        private def load_settings
          # Load configuration from file or environment
          Hash(String, String).new
        end
      end

Method Redefinition and previous_def:
   When redefining methods, use `previous_def` to call the original implementation.

   .. code-block:: crystal

      class Parent
        def greet
          "Hello"
        end
      end

      class Child < Parent
        def greet
          previous_def + ", World!"
        end
      end

      Child.new.greet # => "Hello, World!"

   Without arguments, `previous_def` receives all method parameters. You can also pass specific arguments.

   .. code-block:: crystal

      class Parent
        def process(x, y)
          x + y
        end
      end

      class Child < Parent
        def process(x, y)
          previous_def(x, y) * 2
        end
      end

Object Lifecycle Management:
   Crystal provides sophisticated control over object creation and cleanup.

   .. code-block:: crystal

      class Resource
        def initialize
          @file = File.open("data.txt")
        end

        # Finalizer called during garbage collection
        def finalize
          @file.close if @file
        end

        # Manual allocation (unsafe but powerful)
        def self.allocate_uninitialized
          ptr = Pointer(Resource).malloc(1)
          ptr.value
        end
      end

Modules & Mixins
----------------
Dual-Purpose Modules:
   Crystal modules serve as both namespaces AND mixins, providing powerful code organization.

   .. code-block:: crystal

      module Math
        PI = 3.14159

        def self.circle_area(radius)
          PI * radius ** 2
        end

        # Instance methods for inclusion
        def square(x)
          x * x
        end
      end

      # As namespace
      area = Math.circle_area(5)

      # As mixin
      class Calculator
        include Math  # Gets square method
      end

`extend self` Pattern:
   Create modules usable as both namespaces and included functionality.

   .. code-block:: crystal

      module Utils
        extend self

        def format_time(time : Time)
          time.to_s("%Y-%m-%d %H:%M")
        end

        def sanitize_string(str : String)
          str.strip.downcase
        end
      end

      # Can be called as Utils.format_time or included
      Utils.format_time(Time.local)

Module Type Checking:
   Use modules for polymorphic arrays and method parameters.

   .. code-block:: crystal

      module Drawable
        abstract def draw(canvas : Canvas)
      end

      class Circle
        include Drawable

        def draw(canvas : Canvas)
          # Draw circle
        end
      end

      class Rectangle
        include Drawable

        def draw(canvas : Canvas)
          # Draw rectangle
        end
      end

      # Polymorphic array
      shapes = [] of Drawable
      shapes << Circle.new(10)
      shapes << Rectangle.new(5, 10)

Module System & Code Organization:
   Crystal's require system and module hierarchy.

   .. code-block:: crystal

      # In file.cr - require relative files
      require "./utils"
      require "./models/user"

      # Module as namespace
      module MyApp
        module Database
          def self.connect
            # Database connection logic
          end
        end

        module Utils
          extend self

          def format_time(time : Time)
            time.to_s("%Y-%m-%d")
          end
        end
      end

      # Usage
      MyApp::Database.connect
      MyApp::Utils.format_time(Time.local)

      # Conditional requires
      {% if flag?(:windows) %}
        require "./platform/windows"
      {% else %}
        require "./platform/unix"
      {% end %}

Named Arguments & Splats:
   Arguments with default values must be called with their names. The splat operator (`*`) captures multiple arguments into a `Tuple`.

   .. code-block:: crystal

      def connect(host : String, port = 80, secure = false)
        # ...
      end

      connect("example.com", port: 8080)

      def log(level : Symbol, *messages)
        puts "#{level.upcase}: #{messages.join(", ")}"
      end

      log(:info, "User logged in", "From IP: 127.0.0.1")

`yield` and Block Control:
   The `yield` keyword invokes the block passed to a method. `with ... yield` ensures that the block's value is the method's return value.

   .. code-block:: crystal

      def measure_time
        start_time = Time.local
        yield
        elapsed = Time.local - start_time
        puts "Took #{elapsed.total_seconds} seconds."
      end

      measure_time do
        sleep 1.second
      end

`previous_def`:
   When overriding a method, `previous_def` can be used to call the superclass's implementation, allowing for augmentation rather than complete replacement.

   .. code-block:: crystal

      class Parent
        def greet
          "Hello"
        end
      end

      class Child < Parent
        override def greet
          previous_def + ", World!"
        end
      end

      Child.new.greet # => "Hello, World!"

Block Performance & Inlining:
   Crystal blocks are **always inlined** when used with `yield` - zero runtime overhead. This makes high-level abstractions as fast as hand-written code.

   .. code-block:: crystal

      # This is exactly as fast as a manual loop
      3.times do |i|
        puts i
      end

Block Capturing:
   Convert blocks to `Proc` objects with type signatures for callbacks.

   .. code-block:: crystal

      def register_callback(&callback : String -> Int32)
        @callback = callback
      end

      # Usage
      register_callback do |text|
        text.length
      end

Flexible Proc Types:
   Use underscore for flexible return types.

   .. code-block:: crystal

      # Accepts any return type
      def process_items(items, &block : Int32 -> _)
        items.map { |item| block.call(item) }
      end

Short Block Syntax:
   For single-parameter blocks that call one method, use the shorthand syntax.

   .. code-block:: crystal

      ["a", "b", "c"].map(&.upcase)  # => ["A", "B", "C"]
      [1, 2, 3].map(&.to_s)          # => ["1", "2", "3"]
      [1, 2, 3].map(&.*(2))          # => [2, 4, 6]

Block Control Flow:
   Blocks support `break` (exits method) and `next` (skips to next iteration) with optional return values.

   .. code-block:: crystal

      def find_first(array)
        array.each do |item|
          break item if item > 5
        end
      end

      find_first([1, 2, 8, 3])  # => 8

