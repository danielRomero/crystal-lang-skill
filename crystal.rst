Crystal Language Reference for AI Agents
========================================

Introduction for AI Agents
--------------------------
This document is optimized for AI agents and LLMs generating Crystal code. Crystal combines Ruby's elegant syntax with C's performance through LLVM compilation. Focus on type safety, zero-cost abstractions, and compile-time optimization.

Key for AI Agents:
  Crystal's compiler catches type errors before runtime - leverage this for robust code generation.

Core Principles for Code Generation
-----------------------------------

Type Safety First:
  Always use type restrictions. The compiler is your ally.

  .. code-block:: crystal

        # Good - explicit types
        def process(data : Array(String)) : Hash(String, Int32)
          data.tally
        end

        # Bad - no types (compiler will infer, but be explicit for clarity)
        def process(data)
          data.tally
        end

Zero-Cost Abstractions:
  High-level features compile to efficient machine code.

  .. code-block:: crystal

        # Blocks compile to inlined code - zero overhead
        [1, 2, 3].map(&.* 2)  # As fast as manual loop

Nil Safety:
  Use union types and nil checking.

  .. code-block:: crystal

        # Use ? for optional values
        name : String? = get_name()
        if name
          puts name.upcase  # Compiler knows name is String here
        end

Performance Patterns:
  Prefer structs for immutable data.

  .. code-block:: crystal

        # Structs are value types - stack allocated
        record Point, x : Int32, y : Int32

        # Classes are reference types - heap allocated
        class User
          property name : String
        end

Essential Syntax for Code Generation
------------------------------------

Variable Declaration:
  .. code-block:: crystal

        # Type inference
        count = 42
        message = "Hello"

        # Explicit types (preferred for clarity)
        age : Int32 = 25
        items : Array(String) = [] of String

Method Definitions:
  .. code-block:: crystal

        # Always include type restrictions
        def calculate_area(width : Float64, height : Float64) : Float64
          width * height
        end

        # Default values require named arguments
        def connect(host : String, port : Int32 = 80, secure : Bool = false)
          # implementation
        end

Control Flow:
  .. code-block:: crystal

        # Type-aware conditionals
        value = rand < 0.5 ? 42 : "hello"  # Int32 | String

        case value
        when Int32
          value * 2
        when String
          value.upcase
        end

Compilation Commands:
  .. code-block:: bash

        crystal build app.cr --release    # Optimized binary
        crystal run app.cr                # Run directly
        crystal spec                      # Run tests

Type System for AI Agents
-------------------------
Crystal's static type system provides compile-time safety. Focus on explicit type annotations and union types for robust code generation.

Type Inference Rules
~~~~~~~~~~~~~~~~~~~~

Local Variables:
  Inferred from assignment - use explicit types for clarity:

  .. code-block:: crystal

        count = 42        # Int32
        message = "Hello" # String

Instance Variables:
  Must be explicitly typed OR inferred:

     .. code-block:: crystal

        class Person
          @name : String    # Explicit annotation
          @age : Int32?     # Optional type

          def initialize(@name : String, @age : Int32? = nil)
          end

          def set_default_age
            @age = 0  # Inferred as Int32
          end
        end

Inference Patterns:
  * ``@value = 42`` → Int32
  * ``@name = name`` → from parameter type
  * ``@items = Array(String).new`` → Array(String)
  * ``def initialize(@name = "Default")`` → String

Common Pitfalls:
  * Avoid untyped instance variables in classes
  * Use ``?`` for optional values, not ``Nil``
  * Explicit types prevent runtime errors

Union Types
~~~~~~~~~~~
Union types handle multiple possible values safely:

.. code-block:: crystal

   # Basic union
   value : Int32 | String | Nil

   # Method with union parameter
   def process(data : String | Int32)
      case data
      when String
        data.upcase
      when Int32
        data * 2
      end
   end

Union Patterns:
  * ``value = rand < 0.5 ? 42 : "hello"`` → Int32 | String

  * ``name : String? = get_name()`` → String | Nil

Virtual Types:
   Related types collapse to common ancestor:

  .. code-block:: crystal

      class Foo; end
      class Bar < Foo; end
      class Baz < Foo; end

      foo = rand < 0.5 ? Bar.new : Baz.new  # Foo+ (any Foo subclass)

Type Flow Analysis:
   Compiler tracks type refinement:

  .. code-block:: crystal

      value = rand < 0.5 ? 42 : "hello"  # Int32 | String

      if value.is_a?(Int32)
        puts value + 10  # Compiler knows value is Int32 here
      else
        puts value.upcase  # Compiler knows value is String here
      end

      # Nil checking
      name : String? = get_name()
      if name
        puts name.upcase  # name is String (not String?) in this block
      end

Common Pitfalls:
   * Use ``is_a?`` for type checking, not ``typeof``

   * Union method calls only work with common methods

   * Virtual types (``+``) enable polymorphism

Structs vs Classes (Performance Critical)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Classes:
  Reference types, heap allocated, use for objects with identity:

  .. code-block:: crystal

     class User
       property name : String

       def initialize(@name : String)
       end
     end

     user1 = User.new("Alice")
     user2 = user1  # Same object
     user2.name = "Bob"
     puts user1.name  # "Bob" (changed)

Structs:
  Value types, stack allocated, use for immutable data:

  .. code-block:: crystal

     struct Point
       property x : Int32
       property y : Int32

       def initialize(@x : Int32, @y : Int32)
       end
     end

     p1 = Point.new(1, 2)
     p2 = p1  # Copy is made
     p2.x = 3
     puts p1.x  # 1 (unchanged)

Performance Rules:
   * Use structs for small, immutable data (Point, Color, Time)

   * Use classes for objects with identity and state (User, Database)

   * Most Crystal core types are structs (Int32, String, Array)

Records:
  Immutable structs with value-based equality:

  .. code-block:: crystal

     record User, id : Int32, name : String, email : String

     user = User.new(1, "John", "john@example.com")
     updated = user.copy_with(name: "Jane")  # Creates new instance

Common Pitfalls:
   * Structs cannot inherit from non-abstract structs

   * Mutable structs with reference fields cause unexpected behavior

   * Use records for data transfer objects

Structs vs. Classes (Value vs. Reference Types)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
This is a critical distinction in Crystal that affects performance, memory usage, and behavior.

Classes:
   Reference types. Variables hold a pointer to an object on the heap. They are passed by reference. Use for objects with identity and state that changes over time (e.g., `User`, `HTTP::Server`).

Structs:
   Value types. Variables hold the data directly. They are passed by value (a copy is made). Use for immutable data containers where identity doesn't matter (e.g., `Point`, `Time`, `Color`). Most of Crystal's core types (`Int32`, `String`, `Array`) are structs.

**Struct Advantages:**

Performance benefit:
   Avoid heap allocations for small, immutable data

Stack allocation:
   Faster access and automatic cleanup

Memory locality:
  Better cache performance

**Struct Limitations:**

Inheritance restrictions:
   Cannot inherit from non-abstract structs due to memory layout

Value semantics:
  Copying creates independent instances

  .. code-block:: crystal

     struct Point
       property x : Int32
       property y : Int32

       def initialize(@x, @y)
       end
     end

     p1 = Point.new(1, 2)
     p2 = p1  # Copy is made
     p2.x = 3
     p1.x  # => 1 (unchanged)

Passing by Value:
   A struct is *always* passed by value, even when returning `self`:

  .. code-block:: crystal

     struct Counter
       def initialize(@count : Int32)
       end

       def plus
         @count += 1
         self
       end
     end

     counter = Counter.new(0)
     counter.plus.plus # => Counter(@count=2)
     puts counter      # => Counter(@count=1)

Memory Layout and Inheritance:
  Structs have well-defined memory layout, which affects inheritance rules:

  .. code-block:: crystal

     # Struct implicitly inherits from Struct, which inherits from Value
     # Class implicitly inherits from Reference

     # A struct cannot inherit from a non-abstract struct
     abstract struct BaseStruct
     end

     struct ValidStruct < BaseStruct  # OK
     end

     struct InvalidStruct < Point  # Error: can't inherit from non-abstract struct
     end

   Memory layout reason: An array of points needs predictable size. If inheritance were allowed, the array would need to accommodate different sizes.

Mutable Structs and References:
   Be careful with mutable types inside structs:

   .. code-block:: crystal

      class Klass
        property array = ["str"]
      end

      struct Strukt
        property array = ["str"]
      end

      def modify(object)
        object.array << "foo"
        object.array = ["new"]
        object.array << "bar"
      end

      klass = Klass.new
      modify(klass) # => ["new", "bar"]
      klass.array   # => ["new", "bar"]  # Changed

      strukt = Strukt.new
      modify(strukt) # => ["new", "bar"]
      strukt.array   # => ["str", "foo"]  # Only first change applied

Records:
   The `record` macro creates immutable structs with equality based on value:

   .. code-block:: crystal

      record Point, x : Int32, y : Int32

      point = Point.new(1, 2)
      point.x  # => 1

      # Copy with changes
      updated = point.copy_with(x: 3)
      updated.x  # => 3
      point.x    # => 1 (unchanged)

Generics
~~~~~~~~
Crystal uses generics to write type-agnostic code for containers and algorithms with powerful type inference capabilities.

.. code-block:: crystal

   class Stack(T)
     @elements = [] of T

     def push(element : T)
       @elements.push(element)
     end

     def pop : T
       @elements.pop
     end
   end

   int_stack = Stack(Int32).new
   int_stack.push(10)

   string_stack = Stack(String).new
   string_stack.push("hello")

Advanced Generic Features
*************************

Free Variables:
   Type parameters become free variables in class methods and are inferred from arguments. This makes generic types less tedious to work with.

.. code-block:: crystal

   class MyBox(T)
      def initialize(@value : T)
      end

      def value
        @value
      end
   end

   # Type inference without explicit type arguments
   int_box = MyBox.new(1)          # : MyBox(Int32)
   string_box = MyBox.new("hello")   # : MyBox(String)

   # Works for other class methods too
   class MyBox(T)
      def self.nilable(x : T)
        MyBox(T?).new(x)
      end
   end

   MyBox.nilable(1)     # : MyBox(Int32 | Nil)
   MyBox.nilable("foo") # : MyBox(String | Nil)

Variable Arity:
   Use `*T` for generic classes with variable number of type arguments.

   .. code-block:: crystal

      class Foo(*T)
        getter content

        def initialize(*@content : *T)
        end
      end

      # 2 type variables (explicit)
      foo = Foo(Int32, String).new(42, "Answer")

      # 3 type variables (inferred)
      bar = Foo.new("Hello", ["Crystal", "!"], 140)
      typeof(bar) # => Foo(String, Array(String), Int32)

Module Inclusion:
   Generic modules propagate type variables through inclusion.

   .. code-block:: crystal

      module Moo(T)
         def t
           T
         end
      end

      class Foo(U)
         include Moo(U)

         def initialize(@value : U)
         end
      end

      foo = Foo.new(1)
      foo.t # Int32 (U becomes Int32, which makes T become Int32)

Generic Inheritance:
   Generic classes and structs can be inherited, either with specific instances or delegated type variables.

   .. code-block:: crystal

     class Parent(T)
     end

     class Int32Child < Parent(Int32)
     end

     class GenericChild(T) < Parent(T)
     end

Zero-Argument Generics:
   Generic types with variable arity can have zero arguments.

   .. code-block:: crystal

      class Parent(*T)
      end

      foo = Parent().new
      typeof(foo) # => Parent()

Generic Method Limitations:
   Type inference only works when type parameters can be bound from arguments.

   .. code-block:: crystal

      module Foo(T)
         def self.foo
           T  # Error: can't infer T
         end

         def self.foo(x : T)
           foo  # OK: T inferred from x
         end
      end

      Foo.foo(1)        # Error: can't infer type parameter T
      Foo(Int32).foo(1) # OK

Variable Arity Inheritance:
   Generic types with variable arity support inheritance with specific type arguments.

   .. code-block:: crystal

      class Parent(*T)
      end

      # Inherit with specific types
      class StringChild < Parent(String)
      end

      # Inherit with multiple types
      class Int32StringChild < Parent(Int32, String)
      end

Visibility Controls
~~~~~~~~~~~~~~~~~~~
Crystal provides sophisticated visibility controls for encapsulation and API design.

Private Methods:
   Only callable without explicit receiver (except `self`).

   .. code-block:: crystal

      class Example
        def public_method
          private_method  # OK - no receiver
          self.private_method  # Error - explicit receiver
        end

        private def private_method
          "I'm private"
        end
      end

Protected Methods:
   Callable on same type instances or same namespace.

   .. code-block:: crystal

      class Node
        protected def value
          @value
        end

        def compare(other : Node)
          self.value == other.value  # Can access protected method
        end
      end

Private Top-level:
   File-scoped visibility for internal helpers.

   .. code-block:: crystal

      # In file.cr
      private def internal_helper
        # Only visible in this file
      end

Private Types:
   Private types can only be referenced inside namespace where they are defined.

   .. code-block:: crystal

      class Foo
        private class Bar
        end

        Bar      # OK
        Foo::Bar # Error
      end

      Foo::Bar # Error

   `private` can be used with `class`, `module`, `lib`, `enum`, `alias` and constants:

   .. code-block:: crystal

      class Foo
        private ONE = 1

        ONE # => 1
      end

      Foo::ONE # Error

Protected Methods Details:
   Protected methods can be invoked on:

   * Instances of same type as current type

   * Instances in same namespace as current type

   .. code-block:: crystal

      # Same type instance
      class Person
        protected def say(message)
          puts message
        end

        def say_hello
          self.say "hello"  # OK, self is a Person
          other = Person.new
          other.say "hello" # OK, other is a Person
        end
      end

      # Same namespace
      module Namespace
        class Foo
          protected def foo
            puts "Hello"
          end
        end

        class Bar
          def bar
            # Works, because Foo and Bar are under Namespace
            Foo.new.foo
          end
        end
      end

   Protected methods are visible to subclasses and can be invoked from class scope:

   .. code-block:: crystal

      class Parent
        protected def self.protected_method
        end

        Parent.protected_method # OK

        def instance_method
          Parent.protected_method # OK
        end
      end

Abstract Classes & Methods:
   Abstract classes cannot be instantiated. They define a common interface (a "contract") that concrete subclasses must implement. This is a primary tool for polymorphism.

   .. code-block:: crystal

      abstract class Animal
        abstract def speak : String

        def greet
          "The animal says: #{speak}"
        end
      end

      class Dog < Animal
        override def speak : String
          "Woof!"
        end
      end

      # a = Animal.new # Compile-time error
      dog = Dog.new
      puts dog.greet    # => The animal says: Woof!

Virtual Types:
   Automatic virtual type resolution for "any class inheriting from Type".

   .. code-block:: crystal

      # Animal+ means "any class inheriting from Animal"
      def process_animals(animals : Array(Animal+))
        animals.each(&.speak)
      end

Enums & Flags:
   Enums provide type-safe sets of named values, with optional flags support for bitwise operations.

   .. code-block:: crystal

      enum Status
        Pending
        Approved
        Rejected
      end

      @[Flags]
      enum Permissions
        Read    = 1
        Write   = 2
        Execute = 4
      end

      # Automatic predicate methods
      status = Status::Approved
      status.approved?  # => true
      status.pending?   # => false

      # Flags support bitwise operations
      perms = Permissions::Read | Permissions::Write
      perms.read?     # => true
      perms.execute?  # => false

Enum Values:
   Values start at 0 and increment by 1, but can be overridden:

   .. code-block:: crystal

      enum Color
        Red        # 0
        Green      # 1
        Blue   = 5 # overwritten to 5
        Yellow     # 6 (5 + 1)
      end

      # Get underlying value
      Color::Green.value # => 1

      # Change underlying type
      enum Color : UInt8
        Red
        Green
        Blue
      end

Flags Enums:
   The `@[Flags]` annotation changes default values to powers of 2:

   .. code-block:: crystal

      @[Flags]
      enum IOMode
        Read  # 1
        Write # 2
        Async # 4
      end

      # Implicit constants added
      IOMode::None.value # => 0
      IOMode::All.value  # => 7

Predicate Methods:
   Automatic predicate methods for each member using `String#underscore`:

   .. code-block:: crystal

      enum Color
        Red
        Green
        Blue
      end

      color = Color::Blue
      color.red?    # => false
      color.blue?   # => true

      # For flags enums, uses includes?
      mode = IOMode::Read | IOMode::Async
      mode.read?    # => true
      mode.write?   # => false
      mode.async?   # => true

Symbol Casting:
   Methods with enum type restrictions accept both enum constants and symbols:

   .. code-block:: crystal

      def paint(color : Color)
        puts "Painting using color #{color}"
      end

      paint Color::Red    # Direct enum
      paint :red          # Symbol auto-casts to Color::Red
      paint :yellow        # Compile error: not a Color member

Enum Methods:
   Define methods on enums using patterns from Crystal stdlib:

   .. code-block:: crystal

      # Pattern from YAML parser in Crystal stdlib
      enum ScalarStyle
        ANY
        PLAIN
        SINGLE_QUOTED
        DOUBLE_QUOTED
        LITERAL
        FOLDED

        def quoted?
          single_quoted? || double_quoted?
        end

        def literal?
          literal? || folded?
        end

        def to_s
          super.downcase
        end
      end

      # Pattern with validation - common in Crystal
      enum DayOfWeek
        Monday    = 1
        Tuesday   = 2
        Wednesday = 3
        Thursday  = 4
        Friday    = 5
        Saturday  = 6
        Sunday    = 7

        def self.from_value(value : Int32) : self
          value = 7 if value == 0  # Handle Sunday as 0 or 7
          super(value)
        end

        def weekend?
          saturday? || sunday?
        end

        def weekday?
          !weekend?
        end

        def next_day : DayOfWeek
          DayOfWeek.from_value((value % 7) + 1)
        end
      end

      # Enum with class methods - Crystal stdlib pattern
      enum EventKind
        NONE
        STREAM_START
        STREAM_END
        ALIAS
        SCALAR

        def self.from_char(char : Char) : self?
          case char
          when '{' then STREAM_START
          when '}' then STREAM_END
          when '&' then ALIAS
          else nil
          end
        end

        def stream_event?
          stream_start? || stream_end?
        end
      end

Enum Creation:
   Create enums from integers (useful for C interop):

   .. code-block:: crystal

      Color.new(1) # => Green
      Color.new(10) # => 10 (valid but not a named constant)

Enum Value Types:
   Only integer types are allowed as underlying type. All enums inherit from Enum.

   .. code-block:: crystal

      enum Color : UInt8
        Red
        Green
        Blue
      end

      Color::Red.value # :: UInt8

Class Variables:
   Class variables are allowed in enums, but instance variables are not.

Records:
   The `record` macro creates immutable structs with equality based on value, perfect for data transfer objects.

   .. code-block:: crystal

      record User, id : Int32, name : String, email : String

      user = User.new(1, "John", "john@example.com")
      user.name  # => "John"

      # Copy with changes
      updated_user = user.copy_with(name: "Jane")
      updated_user.name  # => "Jane"

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

Concurrency for AI Agents
-------------------------
Crystal's concurrency uses lightweight **fibers** (spawned via `spawn`) and **channels** for communication. Do not share memory between fibers; communicate via channels.

.. code-block:: crystal

   ch1 = Channel(Int32).new
   ch2 = Channel(String).new

   spawn do
       sleep 1.second
       ch1.send(10)
   end

   spawn do
       sleep 0.5.seconds
       ch2.send("hello")
   end

   # Blocks until one channel has data
   select
   when value = ch1.receive
       puts "Received from ch1: #{value}"
   when value = ch2.receive
       puts "Received from ch2: #{value}"
   end

Select Patterns:
  * ``when value = channel.receive`` - blocking receive

  * ``when channel.send(value)`` - blocking send

  * ``when timeout(5.seconds)`` - timeout

  * ``else`` - non-blocking

  .. code-block:: crystal

     select
     when foo = foo_channel.receive
       puts foo
     when bar = bar_channel.receive?
       puts bar
     when baz_channel.send
       exit
     when timeout(5.seconds)
       puts "Timeout"
      else
        puts "No operations ready"
     end

Channel Best Practices:
   Use buffered channels for producers/consumers with different rates:

   .. code-block:: crystal

      # Buffered channel with capacity of 10
      channel = Channel(Int32).new(10)

      spawn do
        100.times do |i|
          channel.send(i)  # Won't block unless buffer is full
        end
      end

   Use closed channels for graceful shutdown:

   .. code-block:: crystal

      channel = Channel(String).new

      spawn do
        loop do
          if message = channel.receive?
            process_message(message)
          else
            break  # Channel closed, exit worker
          end
        end
      end

      # Close channel when done
      channel.close

Fiber Patterns:
   Worker pool pattern for concurrent processing:

   .. code-block:: crystal

      def create_worker_pool(size : Int32, &block : Int32 -> _)
        channel = Channel(Proc).new(size)

        size.times do |i|
          spawn do
            loop do
              if job = channel.receive?
                block.call(i)
              else
                break
              end
            end
          end
        end

        channel
      end

      # Usage
      workers = create_worker_pool(4) do |worker_id|
        puts "Worker #{worker_id} processing job"
      end

Advanced Concurrency Patterns:
   Complex concurrent workflows and synchronization:

   .. code-block:: crystal

      # Fan-out/Fan-in pattern
      def fan_out_fan_in(urls : Array(String)) : Array(String)
        # Fan-out: spawn workers for each URL
        results_channel = Channel(String).new

        urls.each do |url|
          spawn do
            response = HTTP::Client.get(url)
            results_channel.send(response.body)
          end
        end

        # Fan-in: collect all results
        results = Array(String).new(urls.size)
        urls.size.times do
          results << results_channel.receive
        end

        results
      end

      # Pipeline pattern
      def process_pipeline(data : Array(Int32)) : Array(String)
        # Stage 1: filter even numbers
        stage1 = Channel(Int32).new
        spawn do
          data.each { |n| stage1.send(n) if n.even? }
          stage1.close
        end

        # Stage 2: convert to strings
        stage2 = Channel(String).new
        spawn do
          loop do
            if num = stage1.receive?
              stage2.send(num.to_s)
            else
              break
            end
          end
          stage2.close
        end

        # Stage 3: collect results
        results = Array(String).new
        loop do
          if str = stage2.receive?
            results << str
          else
            break
          end
        end

        results
      end

      # Timeout pattern
      def with_timeout(timeout : Time::Span, &block)
        result_channel = Channel(typeof(block.call)).new(1)
        error_channel = Channel(Exception).new(1)

        spawn do
          begin
            result = block.call
            result_channel.send(result)
          rescue ex
            error_channel.send(ex)
          end
        end

        select
        when value = result_channel.receive
          value
        when error = error_channel.receive
          raise error
        when timeout(timeout)
          raise Timeout::Error.new("Operation timed out")
        end
      end

      # Resource pool pattern
      class ResourcePool(T)
        def initialize(@factory : -> T, @size : Int32)
          @resources = Channel(T).new(@size)
          @size.times { @resources.send(@factory.call) }
        end

        def acquire : T
          @resources.receive
        end

        def release(resource : T)
          @resources.send(resource)
        end

        def use(&block : T -> U) : U
          resource = acquire
          begin
            block.call(resource)
          ensure
            release(resource)
          end
        end
      end

      # Usage
      db_pool = ResourcePool(DB::Connection).new(
        ->{ DB.connect("postgres://...") },
        10
      )

      result = db_pool.use do |conn|
        conn.query_one("SELECT COUNT(*) FROM users", as: Int32)
      end

Metaprogramming: Macros & Annotations for AI Agents
---------------------------------------------------
**Macros** run at compile time and operate on Abstract Syntax Tree (AST) nodes. They eliminate boilerplate and create DSLs.

.. code-block:: crystal

   # Generate methods dynamically
   macro define_getter(*names)
       {% for name in names %}
         def {{name}}
           @{{name}}
         end
        {% end %}
      end


      class Person
      define_getter name, age, email
   end

Macro Patterns:
   * ``{{...}}`` - interpolate AST nodes

   * ``{% if condition %}`` - conditional generation

   * ``{% for item in collection %}`` - iteration

   * ``@type`` - access type information

   .. code-block:: crystal

      macro create_property(name, type)
        property {{name}} : {{type}}
      end

      macro debug_mode
        {% if flag?(:debug) %}
          puts "Debug mode enabled"
        {% else %}
          puts "Production mode"
        {% end %}
      end

      macro generate_accessors(*names)
        {% for name in names %}
          def {{name.id}}
            @{{name.id}}
          end

          def {{name.id}}=(value)
            @{{name.id}} = value
          end
        {% end %}
      end

Advanced Techniques:
   * ``*args`` - variadic arguments

   * ``@type.instance_vars`` - introspection

   * ``previous_def`` - call overridden method

   .. code-block:: crystal

      macro delegate(to, *methods)
        {% for method in methods %}
          def {{method.id}}(*args, **options)
            {{to.id}}.{{method.id}}(*args, **options)
          end
        {% end %}
      end

      macro trace_method(name)
        def {{name.id}}
          puts "Calling {{name.id}}"
          result = previous_def
          puts "Finished {{name.id}}"
          result
        end
      end

Macro Fundamentals
~~~~~~~~~~~~~~~~~~
Macro Interpolation:
   Use `{{...}}` to interpolate AST nodes and expressions:

   .. code-block:: crystal

      macro define_method(name, content)
        def {{name.id}}
          {{content}}
        end
      end

      define_method foo, 1  # Generates: def foo; 1; end

      macro create_property(name, type)
        property {{name}} : {{type}}
      end

      class User
        create_property email, String
        create_property age, Int32
      end

Macro Conditionals:
   Use `{% if condition %}` for conditional code generation:

   .. code-block:: crystal

      macro define_method(name, content)
        def {{name.id}}
          {% if content == 1 %}
            "one"
          {% elsif content == 2 %}
            "two"
          {% else %}
            {{content}}
          {% end %}
        end
      end

      macro debug_mode
        {% if flag?(:debug) %}
          puts "Debug mode enabled"
        {% else %}
          puts "Production mode"
        {% end %}
      end

Macro Iteration:
   Iterate over arrays and ranges:

   .. code-block:: crystal

      macro define_constants(count)
        {% for i in (1..count) %}
          PI_{{i.id}} = Math::PI * {{i}}
        {% end %}
      end

      define_constants(3)  # Creates PI_1, PI_2, PI_3

      macro generate_accessors(*names)
        {% for name in names %}
          def {{name.id}}
            @{{name.id}}
          end

          def {{name.id}}=(value)
            @{{name.id}} = value
          end
        {% end %}
      end

Advanced Macro Techniques
~~~~~~~~~~~~~~~~~~~~~~~~~
Variadic Arguments:
   Use `*` for variable number of arguments:

   .. code-block:: crystal

      macro define_dummy_methods(*names)
        {% for name, index in names %}
          def {{name.id}}
            {{index}}
          end
        {% end %}
      end

      define_dummy_methods foo, bar, baz

      macro delegate(to, *methods)
        {% for method in methods %}
          def {{method.id}}(*args, **options)
            {{to.id}}.{{method.id}}(*args, **options)
          end
        {% end %}
      end

Type Information & Introspection:
   Access type information with `@type` and other special variables:

   .. code-block:: crystal

      macro add_describe_methods
        def describe
          "Class is: " + {{ @type.stringify }}
        end

        def type_name
          {{ @type.name.stringify }}
        end
      end

      macro check_instance_vars
        {% for ivar in @type.instance_vars %}
          puts "{{ivar}}: {{ivar.type}}"
        {% end %}
      end

AST Manipulation:
   Direct manipulation of AST nodes:

   .. code-block:: crystal

      macro trace_method(name)
        def {{name.id}}
          puts "Calling {{name.id}}"
          result = previous_def
          puts "Finished {{name.id}}"
          result
        end
      end

      macro log_call(method_name, *args)
        puts "Calling {{method_name.id}} with: {{args}}"
        {{method_name.id}}({{args}})
      end

Constant and Variable Generation:
   Generate constants and variables dynamically:

   .. code-block:: crystal

      macro enum_values(*values)
        {% for value in values %}
          {{value.upcase}} = {{value.stringify}}
        {% end %}
      end

      macro create_counter(name)
        @@{{name}}_counter = 0

        def self.{{name}}_count
          @@{{name}}_counter
        end

        private def increment_{{name}}_counter
          @@{{name}}_counter += 1
        end
      end

Module and Class Generation:
   Generate entire types and modules:

   .. code-block:: crystal

      macro define_class(module_name, class_name, method, content)
        module {{module_name}}
          class {{class_name}}
            def initialize(@name : String)
            end

            def {{method}}
              {{content}} + @name
            end
          end
        end
      end

      define_class MyApp, User, greet, "Hello "

      macro create_service(name, *dependencies)
        class {{name}}Service
          {% for dep in dependencies %}
            @{{dep.id}} : {{dep.id}}Service
          {% end %}

          def initialize({% for dep in dependencies %}@{{dep.id}} : {{dep.id}}Service{% if !dep.last? %}, {% end %}{% end %})
          end
        end
      end

Advanced Macro Patterns:
   Sophisticated metaprogramming techniques:

   .. code-block:: crystal

      # DSL creation
      macro create_dsl(&block)
        class DSL
          def initialize
            @config = {} of String => String
          end

          def method_missing(name : Symbol, args, &block)
            if block
              @config[name.to_s] = block.call.to_s
            else
              @config[name.to_s] = args.first?.to_s
            end
            self
          end

          def to_h
            @config
          end
        end

        dsl = DSL.new
        dsl.instance_eval(&block)
        dsl.to_h
      end

      # Usage
      config = create_dsl do
        database "postgres://localhost/myapp"
        port 5432
        debug_mode do
          log_level "debug"
          enable_metrics true
        end
      end

      # Event system generation
      macro define_event_system(*events)
        {% for event in events %}
          class {{event.id}}Event
            include JSON::Serializable

            def initialize(@data : String)
            end
          end

          class {{event.id}}Handler
            def handle(event : {{event.id}}Event)
              # Default implementation
            end
          end
        {% end %}

        module EventBus
          @@handlers = {} of String => Array(->)

          def self.subscribe(event_type : T.class, &handler : T ->) forall T
            key = T.name
            @@handlers[key] ||= [] of ->
            @@handlers[key] << handler
          end

          def self.publish(event : T) forall T
            key = T.name
            if handlers = @@handlers[key]?
              handlers.each(&.call(event))
            end
          end
        end
      end

      define_event_system UserCreated, OrderPlaced, PaymentProcessed

      # Type-safe builder pattern
      macro create_builder(base_class)
        class {{base_class}}Builder
          def initialize
            @{{base_class.stringify.downcase.id}} = {{base_class}}.new
          end

          {% for ivar in base_class.resolve.instance_vars %}
            def {{ivar.name}}({{ivar.name}} : {{ivar.type}})
              @{{base_class.stringify.downcase.id}}.{{ivar.name}} = {{ivar.name}}
              self
            end
          {% end %}

          def build : {{base_class}}
            @{{base_class.stringify.downcase.id}}
          end
        end
      end

      class User
        property name : String
        property email : String
        property age : Int32
      end

      create_builder User

      user = UserBuilder.new
        .name("John")
        .email("john@example.com")
        .age(30)
        .build

Annotations
~~~~~~~~~~~
**Annotations** are metadata attached to types, methods, or variables. They are accessible at compile time via macros.

.. code-block:: crystal

   # Frameworks use annotations like this
   @[MyFramework::Route(path: "/users", method: "GET")]
   def users_index(context)
       # ...
   end

   # Database mapping
   @[DB::Field(primary_key: true)]
   property id : Int32

   @[DB::Field(unique: true)]
   property email : String

Annotation Processing:
   Read and process annotations in macros:

   .. code-block:: crystal

      macro process_annotations
        {% for method in @type.methods %}
          {% for ann in method.annotations %}
            {% if ann == MyFramework::Route %}
              register_route({{method.name}}, {{ann[:path]}}, {{ann[:method]}})
            {% end %}
          {% end %}
        {% end %}
      end

      macro validate_required_fields
        {% for ivar in @type.instance_vars %}
          {% if ivar.has_annotation?(Required) %}
            unless @{{ivar.id}}
              raise "{{ivar.id}} is required"
            end
          {% end %}
        {% end %}
      end

Custom Annotations:
   Define and use custom annotations:

   .. code-block:: crystal

      annotation Deprecated
      end

      annotation Experimental
      end

      @[Deprecated("Use new_method instead")]
      def old_method
        "old implementation"
      end

      @[Experimental]
      def new_feature
        "experimental feature"
      end

Annotation with Arguments:
   Pass arguments to annotations:

   .. code-block:: crystal

      annotation Route
        def initialize(@path : String, @method : String = "GET")
        end
      end

      annotation Validate
        def initialize(@rules : Array(String))
        end
      end

      @[Route("/api/users", "POST")]
      @[Validate(["email_required", "password_min_length"])]
      def create_user
        # implementation
      end

Macro Hooks:
   Use macro hooks for automatic processing:

   .. code-block:: crystal

      macro included
        {% if @type.annotation?(AutoJSON) %}
          # Generate JSON methods automatically
          def to_json(json : JSON::Builder)
            {% for ivar in @type.instance_vars %}
              json.field({{ivar.id.stringify}}, @{{ivar.id}})
            {% end %}
          end
        {% end %}
      end

      macro inherited
        puts "Class {{@type.name}} inherits from {{@type.superclass}}"
      end

      macro finished
        puts "Finished compiling {{@type.name}}"
      end

Real-World Macro Patterns
~~~~~~~~~~~~~~~~~~~~~~~~~
DSL Generation:
   Create domain-specific languages:

   .. code-block:: crystal

      macro define_dsl_method(name, type)
        def {{name.id}}(value : {{type.id}})
          @config[{{name.id.stringify}}] = value
        end

        def {{name.id}}
          @config[{{name.id.stringify}}]?.as({{type.id}}?)
        end
      end

      class Configuration
        def initialize
          @config = {} of String => String | Int32 | Bool
        end

        define_dsl_method host, String
        define_dsl_method port, Int32
        define_dsl_method debug, Bool
      end

Validation Macros:
   Generate validation logic:

   .. code-block:: crystal

      macro validates_presence_of(*fields)
        {% for field in fields %}
          def validate_{{field.id}}_presence
            if @{{field.id}}.nil? || @{{field.id}}.to_s.empty?
              errors.add({{field.id.stringify}}, "can't be blank")
            end
          end
        {% end %}
      end

      macro validates_length_of(field, min = nil, max = nil)
        def validate_{{field.id}}_length
          value = @{{field.id}}.to_s
          {% if min %}
            if value.size < {{min}}
              errors.add({{field.id.stringify}}, "too short (minimum is {{min}} characters)")
            end
          {% end %}
          {% if max %}
            if value.size > {{max}}
              errors.add({{field.id.stringify}}, "too long (maximum is {{max}} characters)")
            end
          {% end %}
        end
      end

Serialization Macros:
   Generate serialization methods:

   .. code-block:: crystal

      macro json_fields(*fields)
        def to_json(json : JSON::Builder)
          json.object do
            {% for field in fields %}
              json.field({{field.id.stringify}}, @{{field.id}})
            {% end %}
          end
        end

        def self.from_json(json : JSON::PullParser)
          instance = new
          json.on_key! do |key|
            {% for field in fields %}
              if key == {{field.id.stringify}}
                instance.@{{field.id}} = {{field.type}}.new(json)
              end
            {% end %}
          end
          instance
        end
      end

Testing Macros
~~~~~~~~~~~~~~
Generate test methods:

   .. code-block:: crystal

     macro test(description, &block)
       test_{{description.gsub(/\s+/, "_").id}} do
         {{block.body}}
       end
     end

     macro should(description, &block)
       it "should {{description}}" do
         {{block.body}}
       end
     end

     test "adds two numbers" do
       result = 2 + 3
       result.should eq(5)
     end

     should "return zero for empty array" do
       [].size.should eq(0)
     end

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

Compile-Time Features for AI Agents
-----------------------------------
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

Constant Evaluation:
   Compile-time computation and optimization:

   .. code-block:: crystal

      # Constants are evaluated at compile time
      PI_SQUARED = Math::PI * Math::PI

      # Complex expressions work too
      FACTORIAL_5 = (1..5).reduce(1) { |acc, i| acc * i }

      # Type-level computations
      ARRAY_SIZE = 10
      DEFAULT_ARRAY = Array(Int32).new(ARRAY_SIZE, 0)

Version Information:
   Access Crystal version at compile time:

   .. code-block:: crystal

      {% if compare_versions(Crystal::VERSION, "1.10.0") >= 0 %}
        # Use newer features
        def modern_feature
          "Available in 1.10.0+"
        end
      {% else %}
        # Fallback for older versions
        def modern_feature
          "Using fallback implementation"
        end
      {% end %}

Conditional Compilation:
   Include or exclude code based on conditions:

   .. code-block:: crystal

      {% if env("DATABASE_URL") %}
        require "db"
        DATABASE_URL = env("DATABASE_URL")
      {% else %}
        puts "Warning: No DATABASE_URL configured"
      {% end %}

      # Debug-only code
      {% if flag?(:debug) %}
        macro debug_log(message)
          puts "[DEBUG] {{message}}"
        end
      {% else %}
        macro debug_log(message)
          # No-op in release
        end
      {% end %}

Macro Compilation:
   Macros execute during compilation, not runtime:

  .. code-block:: crystal

     macro generate_methods(*names)
       {% for name in names %}
         def {{name.id}}_value
           {{name.id}}.to_s
         end
       {% end %}
     end

     # This generates methods at compile time
     generate_methods age, name, email

     # Resulting methods available at runtime
     puts age_value   # Works - method was compiled
     puts name_value   # Works - method was compiled

Architecture & Common Patterns for AI Agents
--------------------------------------------
Dependency Injection:
   Use constructor injection for testable code:

   .. code-block:: crystal

      class DatabaseService
        def initialize(@connection : DB::Connection)
        end

        def find_user(id : Int32) : User?
          @connection.query_one?("SELECT * FROM users WHERE id = ?", id)
        end
      end

      class UserService
        def initialize(@db : DatabaseService)
        end

        def get_user(id : Int32) : User?
          @db.find_user(id)
        end
      end

Service Layer Pattern:
   Separate business logic from data access:

   .. code-block:: crystal

      abstract class Service
        abstract def call(*args)
      end

      class CreateUser < Service
        def initialize(@user_repo : UserRepository, @email_service : EmailService)
        end

        def call(name : String, email : String) : User
          user = User.new(name, email)
          @user_repo.save(user)
          @email_service.send_welcome(email)
          user
        end
      end

Repository Pattern:
   Abstract data access with consistent interface:

   .. code-block:: crystal

      abstract class Repository(T)
        abstract def save(entity : T) : T
        abstract def find(id : Int32) : T?
        abstract def find_all : Array(T)
        abstract def delete(id : Int32) : Bool
      end

      class UserRepository < Repository(User)
        def initialize(@db : DB::Connection)
        end

        def save(user : User) : User
          # Implementation
        end

        def find(id : Int32) : User?
          # Implementation
        end

        def find_all : Array(User)
          # Implementation
        end

        def delete(id : Int32) : Bool
          # Implementation
        end
      end

Factory Pattern:
   Create objects with proper initialization:

   .. code-block:: crystal

      abstract class Notifier
        abstract def send(message : String) : Bool
      end

      class EmailNotifier < Notifier
        def send(message : String) : Bool
          # Send email implementation
        end
      end

      class SMSNotifier < Notifier
        def send(message : String) : Bool
          # Send SMS implementation
        end
      end

      class NotifierFactory
        def self.create(type : String) : Notifier
          case type.downcase
          when "email"
            EmailNotifier.new
          when "sms"
            SMSNotifier.new
          else
            raise ArgumentError.new("Unknown notifier type: #{type}")
          end
        end
      end

Observer Pattern:
   Event-driven architecture with type safety:

   .. code-block:: crystal

      abstract class Observer
        abstract def update(data : String)
      end

      class EventManager
        @observers = [] of Observer

        def subscribe(observer : Observer)
          @observers << observer
        end

        def notify(data : String)
          @observers.each(&.update(data))
        end
      end

      class Logger < Observer
        def update(data : String)
          puts "Log: #{data}"
        end
      end

      class EmailAlert < Observer
        def update(data : String)
          send_alert(data) if data.includes?("error")
        end
      end

Strategy Pattern:
   Pluggable algorithms with compile-time safety:

   .. code-block:: crystal

      abstract class CompressionStrategy
        abstract def compress(data : Bytes) : Bytes
        abstract def decompress(data : Bytes) : Bytes
      end

      class GzipCompression < CompressionStrategy
        def compress(data : Bytes) : Bytes
          # Gzip implementation
        end

        def decompress(data : Bytes) : Bytes
          # Gzip implementation
        end
      end

      class LzmaCompression < CompressionStrategy
        def compress(data : Bytes) : Bytes
          # LZMA implementation
        end

        def decompress(data : Bytes) : Bytes
          # LZMA implementation
        end
      end

      class Compressor
        def initialize(@strategy : CompressionStrategy)
        end

        def compress_file(path : String) : Bytes
          data = File.read(path)
          @strategy.compress(data)
        end
      end

Configuration Management:
   Type-safe configuration with validation:

   .. code-block:: crystal

      struct AppConfig
        property database_url : String
        property port : Int32
        property debug : Bool
        property log_level : String

        def self.from_env : self
          new(
            database_url: env("DATABASE_URL") || "sqlite://app.db",
            port: (env("PORT") || "3000").to_i,
            debug: env("DEBUG")? == "true",
            log_level: env("LOG_LEVEL") || "info"
          )
        end

        def validate!
          raise "Invalid database URL" unless @database_url.includes?("://")
          raise "Port must be positive" if @port <= 0
          raise "Invalid log level" unless ["debug", "info", "warn", "error"].includes?(@log_level)
        end
      end

Error Handling Patterns:
   Consistent error handling across layers:

   .. code-block:: crystal

      abstract class AppError < Exception
        getter code : String

        def initialize(@code : String, message : String)
          super(message)
        end
      end

      class ValidationError < AppError
        def initialize(field : String, message : String)
          super("VALIDATION_ERROR", "#{field}: #{message}")
        end
      end

      class NotFoundError < AppError
        def initialize(resource : String, id : Int32)
          super("NOT_FOUND", "#{resource} with id #{id} not found")
        end
      end

      class Service
        def handle_user(id : Int32) : User
          user = find_user(id)
          raise NotFoundError.new("User", id) unless user
          user
        rescue ex : DB::Error
          raise AppError.new("DATABASE_ERROR", "Failed to fetch user: #{ex.message}")
        end
      end

Standard Library: Collections & I/O for AI Agents
-----------------------------------------------------
Collections:
   Efficient, type-safe data structures:

   .. code-block:: crystal

      # Arrays - dynamic, growable collections
      numbers = [1, 2, 3, 4, 5]
      numbers << 6                    # Add element
      numbers.map(&.* 2)              # [2, 4, 6, 8, 10, 12]
      numbers.select(&.even?)           # [2, 4, 6]
      numbers.reduce(0) { |acc, n| acc + n }  # 21

      # Hashes - key-value mappings
      config = {
        "host"     => "localhost",
        "port"     => 5432,
        "database"  => "myapp",
        "ssl"       => true,
      }

      config["host"]?                 # "localhost" (safe access)
      config.fetch("timeout", 30)      # 30 (default value)
      config.transform_keys(&.upcase)   # {"HOST" => "localhost", ...}

      # Tuples - fixed-size, typed collections
      point = {10, 20}               # Tuple(Int32, Int32)
      point[0]                        # 10
      point[1]                        # 20

      # Named Tuples - self-documenting
      user = {
        id:    1,
        name:  "Alice",
        email: "alice@example.com",
      }  # NamedTuple(id: Int32, name: String, email: String)

      user[:name]                     # "Alice"
      user[:email]?                    # "alice@example.com"

String Manipulation:
   Powerful string processing with Unicode support:

   .. code-block:: crystal

      text = "Hello, Crystal World!"

      # Basic operations
      text.upcase                      # "HELLO, CRYSTAL WORLD!"
      text.downcase                    # "hello, crystal world!"
      text.capitalize                  # "Hello, crystal world!"
      text.titleize                    # "Hello, Crystal World!"

      # Splitting and joining
      words = text.split               # ["Hello,", "Crystal", "World!"]
      words.join("-")                  # "Hello,-Crystal,-World!"

      # Substitution
      text.gsub("Crystal", "Ruby")     # "Hello, Ruby World!"
      text.sub(/World/, "Universe")    # "Hello, Crystal Universe!"

      # Formatting
      "%s has %d items".format("Cart", 5)  # "Cart has 5 items"
      "Number: %0.2f".format(3.14159)       # "Number: 3.14"

File I/O:
   Simple, type-safe file operations:

   .. code-block:: crystal

      # Reading files
      content = File.read("config.json")           # String
      lines = File.read_lines("data.txt")          # Array(String)
      bytes = File.read_bytes("image.bin")         # Bytes

      # Writing files
      File.write("output.txt", "Hello, World!")
      File.write_lines("data.csv", ["name,age", "Alice,30"])

      # File operations
      File.exists?("config.yml")                   # Bool
      File.size("large_file.txt")                  # Int64
      File.rename("old.txt", "new.txt")            # Nil

      # Directory operations
      Dir.mkdir("logs") unless Dir.exists?("logs")
      Dir.entries(".")                             # Array(String)
      Dir.glob("*.cr")                            # Array(String)

JSON Processing:
   Built-in JSON serialization:

   .. code-block:: crystal

      # JSON parsing
      json_string = %({"name": "Alice", "age": 30, "active": true})
      data = JSON.parse(json_string)                 # JSON::Any

      data["name"].as_s              # "Alice"
      data["age"].as_i               # 30
      data["active"].as_bool          # true

      # Type-safe parsing
      struct User
        include JSON::Serializable

        property name : String
        property age : Int32
        property active : Bool
      end

      user = User.from_json(json_string)
      user.name                           # "Alice"
      user.age                            # 30

      # Serialization
      user.to_json                       # {"name":"Alice","age":30,"active":true}

HTTP Client:
   Simple HTTP requests with proper error handling:

   .. code-block:: crystal

      require "http/client"

      # GET request
      response = HTTP::Client.get("https://api.example.com/users")
      response.status_code                  # 200
      response.body                        # JSON string
      response.success?                    # true

      # POST request with JSON
      client = HTTP::Client.new("https://api.example.com")
      headers = HTTP::Headers{"Content-Type" => "application/json"}
      body = %({"name": "Bob", "email": "bob@example.com"})

      response = client.post("/users", headers, body)

      # Error handling
      begin
        response = HTTP::Client.get("https://example.com")
        puts response.body
      rescue ex : Socket::Addrinfo::Error
        puts "Network error: #{ex.message}"
      rescue ex : URI::Error
        puts "Invalid URL: #{ex.message}"
      end

Standard Library: Utilities for AI Agents
---------------------------------------------
Time and Date:
   Comprehensive date/time handling:

   .. code-block:: crystal

      # Current time
      now = Time.local                    # Time
      utc = Time.utc                      # Time

      # Creating specific times
      birthday = Time.local(1990, 5, 15, 14, 30, 0)
      meeting = Time.parse("2024-01-15 10:00", "%F %T", Time::Location.local)

      # Formatting
      now.to_s("%Y-%m-%d %H:%M:%S")        # "2024-01-15 10:30:45"
      now.to_rfc3339                        # "2024-01-15T10:30:45Z"

      # Time arithmetic
      tomorrow = now + 1.day
      last_week = now - 7.days
      in_2_hours = now + 2.hours

      # Comparison
      now > tomorrow?                       # false
      birthday < now?                        # true

Logging:
   Built-in logging with multiple levels:

   .. code-block:: crystal

      require "log"

      # Basic logging
      Log.info { "Application started" }
      Log.debug { "Processing request #{request_id}" }
      Log.warn { "Rate limit approaching" }
      Log.error { "Database connection failed" }

      # Structured logging
      Log.context.set(user_id: user.id, request_id: request.id)
      Log.info { "User action completed" }

      # Custom log configuration
      Log.setup(
        level: Log::Severity::Debug,
        backend: Log::IOBackend.new(File.new("app.log", "a"))
      )

YAML Processing:
   Human-readable configuration and data serialization:

   .. code-block:: crystal

      require "yaml"

      # YAML parsing
      yaml_string = %(
        name: "John Doe"
        age: 30
        hobbies:
          - reading
          - coding
          - gaming
        address:
          street: "123 Main St"
          city: "Anytown"
      )

      data = YAML.parse(yaml_string)  # YAML::Any
      data["name"].as_s               # "John Doe"
      data["hobbies"].as_a            # ["reading", "coding", "gaming"]

      # Type-safe parsing
      struct Person
        include YAML::Serializable

        property name : String
        property age : Int32
        property hobbies : Array(String)
      end

      person = Person.from_yaml(yaml_string)
      person.hobbies                  # ["reading", "coding", "gaming"]

      # YAML serialization
      person.to_yaml                  # name: John Doe\nage: 30\n...

URI Handling:
   URL parsing and manipulation:

   .. code-block:: crystal

      require "uri"

      # URI parsing
      uri = URI.parse("https://user:pass@example.com:8080/path?query=value#fragment")
      uri.scheme                      # "https"
      uri.host                        # "example.com"
      uri.port                        # 8080
      uri.path                        # "/path"
      uri.query                       # "query=value"
      uri.user                        # "user"
      uri.password                    # "pass"
      uri.fragment                    # "fragment"

      # URI building
      uri = URI.new(
        scheme: "https",
        host: "api.example.com",
        port: 443,
        path: "/v1/users",
        query: "limit=10&offset=0"
      )

      uri.to_s                        # "https://api.example.com/v1/users?limit=10&offset=0"

      # URL encoding/decoding
      URI.encode("hello world")       # "hello%20world"
      URI.decode("hello%20world")     # "hello world"

Path Operations:
   Cross-platform file path manipulation:

   .. code-block:: crystal

      require "path"

      # Path operations
      path = Path["/home/user/documents/file.txt"]
      path.parent                     # Path["/home/user/documents"]
      path.name                       # "file.txt"
      path.extension                  # "txt"
      path.stem                       # "file"

      # Path building
      home = Path.home                # Current user's home directory
      config_path = home / "config" / "app.yml"
      config_path.to_s                # "/home/user/config/app.yml"

      # Path normalization
      messy = Path["./foo/../bar/./baz"]
      messy.normalize                 # Path["bar/baz"]

      # Glob patterns
      Dir.glob("**/*.cr")             # All .cr files recursively
      Dir.glob("src/**/*.cr")         # .cr files in src/ directory

System Information:
   Environment and system utilities:

   .. code-block:: crystal

      # Environment variables
      ENV["HOME"]?                    # "/home/user" or nil
      ENV["PATH"]?.try(&.split(":"))  # Array of paths

      # System information
      System.cpu_count                # Number of CPU cores
      System.hostname                 # System hostname

      # Process information
      Process.pid                     # Current process ID
      Process.executable_path         # Path to current executable

      # Memory usage
      Process.rss                     # Resident Set Size in bytes

Random Number Generation:
   Cryptographically secure random numbers:

   .. code-block:: crystal

      require "random"

      # Random numbers
      Random.rand                     # Float64 between 0.0 and 1.0
      Random.rand(100)                # Int32 between 0 and 99
      Random.rand(1..100)             # Int32 between 1 and 100

      # Random bytes
      Random::Secure.random_bytes(16) # 16 random bytes

      # Random strings
      Random::Secure.hex(32)          # 32 character hex string
      Random::Secure.base64(32)       # 32 character base64 string

Base64 Encoding:
   Binary data encoding:

   .. code-block:: crystal

      require "base64"

      # Encoding
      data = "Hello, World!"
      encoded = Base64.encode(data)   # "SGVsbG8sIFdvcmxkIQ=="
      strict = Base64.strict_encode(data)  # "SGVsbG8sIFdvcmxkIQ"

      # Decoding
      Base64.decode(encoded)          # "Hello, World!"
      Base64.decode_string(encoded)   # "Hello, World!"

      # URL-safe encoding
      Base64.urlsafe_encode("user:pass")  # "dXNlcjpwYXNz"

Cryptography:
   Hash functions and encryption:

   .. code-block:: crystal

      require "digest"

      # Hash functions
      Digest::MD5.hexdigest("hello")  # "5d41402abc4b2a76b9719d911017c592"
      Digest::SHA1.hexdigest("hello") # "aaf4c61ddcc5e8a2dabede0f3b482cd9aea9434d"
      Digest::SHA256.hexdigest("hello") # "2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824"

      # HMAC
      require "openssl"
      key = "secret"
      data = "message"
      hmac = OpenSSL::HMAC.hexdigest(:sha256, key, data)

Regular Expressions:
   Advanced pattern matching:

   .. code-block:: crystal

      # Regex literals
      email_pattern = /(\w+)@(\w+\.\w+)/
      phone_pattern = /(\d{3})-(\d{3})-(\d{4})/

      # Matching
      if match = email_pattern.match("user@example.com")
        match[1]                      # "user"
        match[2]                      # "example.com"
        match.captures                # ["user", "example.com"]
      end

      # Substitution
      text = "Hello, 123-456-7890!"
      text.gsub(phone_pattern, "\\1-\\2-XXXX")  # "Hello, 123-456-XXXX!"

      # Validation
      def valid_email?(email : String) : Bool
        email =~ /\A[\w+\-.]+@[a-z\d\-]+(\.[a-z\d\-]+)*\.[a-z]+\z/i
      end

Process Management:
   Running external commands:

   .. code-block:: crystal

      # Simple command execution
      output = `ls -la`               # Run ls -la and capture output
      status = $?.success?            # Check exit status

      # Process execution with options
      process = Process.new("grep", ["-r", "pattern", "src/"], output: :inherit)
      process.wait                    # Wait for completion
      process.exit_status             # Exit code

      # Background processes
      Process.fork do
        # Child process code
        sleep 1
        puts "Child process done"
      end

      # Parent continues immediately
      puts "Parent process continuing"

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

Project Structure & Shards for AI Agents
----------------------------------------
Standard Project Layout:
   Crystal project directory structure:

   .. code-block:: crystal

      my_app/
      ├── src/
      │   ├── my_app.cr           # Main entry point
      │   ├── my_app/
      │   │   ├── *.cr           # Application modules
      │   │   └── version.cr      # Version information
      │   └── ext/               # C extensions
      ├── spec/
      │   ├── spec_helper.cr       # Test configuration
      │   └── my_app/
      │       └── *_spec.cr       # Test files
      ├── lib/                   # External dependencies
      ├── bin/                   # Executable scripts
      ├── config/                # Configuration files
      ├── tasks/                 # Build tasks
      ├── public/                # Static assets
      ├── shard.yml              # Dependencies
      ├── .ameba.yml            # Linter config
      ├── .gitignore
      └── README.md

Shard Configuration:
   Dependency management with shard.yml:

   .. code-block:: crystal

      name: my_awesome_app
      version: 0.1.0

      description: |
        An awesome Crystal application that demonstrates best practices
        for project structure and dependency management.

      authors:
        - John Doe <john@example.com>

      crystal: ">= 1.10.0"

      license: MIT

      targets:
        my_app:
          main: src/my_app.cr

      dependencies:
        # GitHub dependencies
        lucky:
          github: luckyframework/lucky
          version: ~> 1.0.0

        # Git dependencies
        custom_lib:
          git: https://github.com/user/custom_lib.git
          branch: main

        # Path dependencies
        local_lib:
          path: ../local_lib

        # Crystal shard registry
        crest:
          github: mamantoha/crest
          version: ~> 1.0

      development_dependencies:
        # Testing tools
        ameba:
          github: crystal-ameba/ameba
          version: ~> 1.0

        # Documentation
        markd:
          github: icyleaf/markd
          version: ~> 1.0

Development Dependencies:
   Tools for development and testing:

   .. code-block:: crystal

      # shard.yml development section
      development_dependencies:
        # Code quality
        ameba:
          github: crystal-ameba/ameba
          version: ~> 1.0

        # Testing utilities
        webmock:
          github: manastech/webmock.cr
          version: ~> 0.14

        # Documentation generation
        cadmium_tools:
          github: cadmiumcr/tools
          version: ~> 0.1

        # Benchmarking
        benchmark:
          github: crystal-benchmark/benchmark
          version: ~> 0.1

Build Tasks:
   Custom build automation with tasks:

   .. code-block:: crystal

      # tasks/build.cr
      require "task_manager"

      TaskManager.new do |tasks|
        tasks.define "build" do
          puts "Building application..."
          system("crystal build src/my_app.cr --release")
        end

        tasks.define "test" do
          puts "Running tests..."
          system("crystal spec")
        end

        tasks.define "lint" do
          puts "Running linter..."
          system("ameba src/")
        end

        tasks.define "ci" do
          tasks.run("lint")
          tasks.run("test")
          tasks.run("build")
        end
      end

      # Run tasks
      # crystal run tasks/build.cr build
      # crystal run tasks/build.cr ci

Configuration Management:
   Environment-specific configuration:

   .. code-block:: crystal

      # config/environments.cr
      abstract class AppConfig
        property database_url : String
        property port : Int32
        property log_level : String
        property workers : Int32

        abstract def self.load : self
      end

      # config/development.cr
      class DevelopmentConfig < AppConfig
        def self.load
          new(
            database_url: "sqlite://./dev.db",
            port: 3000,
            log_level: "debug",
            workers: 1
          )
        end
      end

      # config/production.cr
      class ProductionConfig < AppConfig
        def self.load
          new(
            database_url: ENV["DATABASE_URL"],
            port: ENV["PORT"]?.try(&.to_i) || 8080,
            log_level: "info",
            workers: ENV["WORKERS"]?.try(&.to_i) || 4
          )
        end
      end

      # src/my_app/config.cr
      module MyApp
        def self.config
          {% if env("CRYSTAL_ENV") == "production" %}
            ProductionConfig.load
          {% else %}
            DevelopmentConfig.load
          {% end %}
        end
      end

Module Organization:
   Logical code organization:

   .. code-block:: crystal

      # src/my_app/models/user.cr
      module MyApp::Models
        class User
          property id : Int32
          property name : String
          property email : String

          def initialize(@id : Int32, @name : String, @email : String)
          end
        end
      end

      # src/my_app/services/user_service.cr
      module MyApp::Services
        class UserService
          def initialize(@db : DB::Database)
          end

          def find_user(id : Int32) : MyApp::Models::User?
            # Implementation
          end
        end
      end

      # src/my_app/controllers/user_controller.cr
      module MyApp::Controllers
        class UserController
          def initialize(@user_service : MyApp::Services::UserService)
          end

          def show(id : Int32) : MyApp::Models::User?
            @user_service.find_user(id)
          end
        end
      end

Entry Point Structure:
   Clean application initialization:

   .. code-block:: crystal

      # src/my_app.cr
      require "./my_app/**"
      require "log"

      module MyApp
        VERSION = {{ `shards version #{__DIR__}`.chomp.stringify }}

        def self.run
          Log.info { "Starting MyApp v#{VERSION}" }

          config = self.config
          Log.info { "Configuration loaded: #{config}" }

          # Initialize services
          db = DB::Database.new(config.database_url)
          user_service = Services::UserService.new(db)

          # Start application
          server = HTTP::Server.new(config.port) do |context|
            # Handle requests
          end

          Log.info { "Server listening on port #{config.port}" }
          server.listen
        rescue ex
          Log.error { "Failed to start: #{ex.message}" }
          exit(1)
        end
      end

      # Run the application
      MyApp.run

Testing Structure:
   Organized test suite:

   .. code-block:: crystal

      # spec/spec_helper.cr
      require "spec"
      require "../src/my_app"

      Spec.before_each do
        # Setup before each test
      end

      Spec.after_each do
        # Cleanup after each test
      end

      # spec/my_app/models/user_spec.cr
      require "../spec_helper"

      describe MyApp::Models::User do
        describe ".new" do
          it "creates a user with valid attributes" do
            user = MyApp::Models::User.new(1, "Alice", "alice@example.com")
            user.id.should eq(1)
            user.name.should eq("Alice")
            user.email.should eq("alice@example.com")
          end
        end
      end

      # spec/my_app/services/user_service_spec.cr
      require "../spec_helper"

      describe MyApp::Services::UserService do
        describe "#find_user" do
          it "returns user when found" do
            # Setup test database
            db = setup_test_db
            service = MyApp::Services::UserService.new(db)

            user = service.find_user(1)
            user.should_not be_nil
            user.not_nil!.name.should eq("Alice")
          end

          it "returns nil when not found" do
            db = setup_test_db
            service = MyApp::Services::UserService.new(db)

            user = service.find_user(999)
            user.should be_nil
          end
        end
      end

Deployment Structure:
   Production-ready organization:

   .. code-block:: crystal

      # Dockerfile
      FROM crystallang/crystal:1.10.0-alpine AS builder
      WORKDIR /app
      COPY shard.yml shard.lock ./
      RUN shards install --production
      COPY . .
      RUN shards build --release --static

      FROM alpine:latest
      RUN apk add --no-cache ca-certificates
      WORKDIR /root
      COPY --from=builder /app/bin/my_app .
      CMD ["./my_app"]

      # docker-compose.yml
      version: '3.8'
      services:
        app:
          build: .
          ports:
            - "3000:3000"
          environment:
            - CRYSTAL_ENV=production
            - DATABASE_URL=postgresql://user:pass@db:5432/app
          depends_on:
            - db

        db:
          image: postgres:15
          environment:
            - POSTGRES_DB=app
            - POSTGRES_USER=user
            - POSTGRES_PASSWORD=pass
          volumes:
            - postgres_data:/var/lib/postgresql/data

      volumes:
        postgres_data:

Performance & Optimization for AI Agents
----------------------------------------
Memory Management:
   Efficient memory usage patterns.

   .. code-block:: crystal

      # Use structs for small, immutable data
      struct Point
        property x : Float64
        property y : Float64

        def initialize(@x : Float64, @y : Float64)
        end

        def distance(other : Point) : Float64
          dx = @x - other.x
          dy = @y - other.y
          Math.sqrt(dx * dx + dy * dy)
        end
      end

      # Pre-allocate arrays when size is known
      def process_items(items : Array(String)) : Array(Int32)
        result = Array(Int32).new(items.size)  # Pre-allocate
        items.each { |item| result << item.size }
        result
      end

      # Use String::Builder for concatenation
      def join_strings(strings : Array(String)) : String
        builder = String::Builder.new
        strings.each { |str| builder << str }
        builder.to_s
      end

      # Avoid unnecessary allocations in loops
      def inefficient_sum(numbers : Array(Int32)) : Int32
        sum = 0
        numbers.each do |n|
          temp = n * 2  # Unnecessary allocation
          sum += temp
        end
        sum
      end

      def efficient_sum(numbers : Array(Int32)) : Int32
        sum = 0
        numbers.each { |n| sum += n * 2 }  # Direct calculation
        sum
      end

Algorithm Optimization:
   Choose optimal algorithms.

   .. code-block:: crystal

      # Use appropriate data structures
      # Bad: O(n) lookup in array
      def find_user_bad(users : Array(User), id : Int32) : User?
        users.find { |user| user.id == id }
      end

      # Good: O(1) lookup in hash
      def find_user_good(users : Array(User), id : Int32) : User?
        user_map = users.map { |user| {user.id, user} }.to_h
        user_map[id]?
      end

      # Efficient string operations
      def process_text(text : String) : String
        # Bad: Multiple string allocations
        result = ""
        text.each_char do |char|
          result += char.upcase
        end
        result

        # Good: Single allocation
        text.upcase
      end

      # Lazy evaluation for large datasets
      def process_large_file(path : String)
        File.each_line(path) do |line|
          # Process line without loading entire file
          yield line.strip
        end
      end

Compilation Optimizations:
   Release build settings.

   .. code-block:: bash

      # Optimized compilation flags
      crystal build app.cr \
        --release \
        --no-debug \
        --single-module \
        --static \
        --progress

      # Profile-guided optimization (if supported)
      crystal build app.cr --release --profile

      # Link-time optimization
      crystal build app.cr --release --lto

Type System Optimization:
   Leverage compile-time type checking.

   .. code-block:: crystal

      # Use union types efficiently
      def process_value(value : Int32 | String | Float64)
        case value
        when Int32
          value * 2
        when String
          value.upcase
        when Float64
          value.round
        end
      end

      # Avoid nil checks with proper typing
      # Bad: Optional chaining everywhere
      def process_user(user : User?)
        user&.name&.upcase || "Unknown"
      end

      # Good: Handle nil explicitly
      def process_user(user : User?) : String
        if user
          user.name.upcase
        else
          "Unknown"
        end
      end

      # Use generic constraints for better optimization
      def process_numeric<T>(numbers : Array(T)) : T
        numbers.reduce(T.new(0)) { |acc, n| acc + n }
      end

Concurrency Optimization:
   Efficient parallel processing.

   .. code-block:: crystal

      # Use fibers for I/O-bound tasks
      def fetch_urls(urls : Array(String)) : Array(String)
        channel = Channel(String).new
        results = Array(String).new

        # Spawn fibers for concurrent requests
        urls.each do |url|
          spawn do
            response = HTTP::Client.get(url)
            channel.send(response.body)
          end
        end

        # Collect results
        urls.size.times do
          results << channel.receive
        end

        results
      end

      # Worker pool pattern
      class WorkerPool(T)
        def initialize(@size : Int32, &@proc : T -> _)
          @channel = Channel(T).new(@size)
          @workers = Array(Fiber).new(@size)

          @size.times do |i|
            @workers << spawn do
              loop do
                if item = @channel.receive?
                  @proc.call(item)
                else
                  break
                end
              end
            end
          end
        end

        def process(item : T)
          @channel.send(item)
        end

        def shutdown
          @channel.close
          @workers.each(&.join)
        end
      end

Memory Profiling:
   Identify memory bottlenecks.

   .. code-block:: crystal

      require "benchmark"

      # Memory benchmark
      Benchmark.memory do |x|
        x.report("string allocation") do
          1000.times do
            str = "Hello, World! " * 100
            str.size
          end
        end

        x.report("string builder") do
          1000.times do
            builder = String::Builder.new
            100.times { builder << "Hello, World! " }
            builder.to_s.size
          end
        end
      end

      # Custom memory tracking
      class MemoryTracker
        @@allocations = 0
        @@deallocations = 0

        def self.track_allocation
          @@allocations += 1
        end

        def self.track_deallocation
          @@deallocations += 1
        end

        def self.stats
          {
            allocations: @@allocations,
            deallocations: @@deallocations,
            active: @@allocations - @@deallocations
          }
        end
      end

      # Usage in custom types
      struct TrackedString
        def initialize(@value : String)
          MemoryTracker.track_allocation
        end

        def finalize
          MemoryTracker.track_deallocation
        end
      end

Cache Optimization:
   Implement effective caching.

   .. code-block:: crystal

      # Simple in-memory cache
      class Cache(K, V)
        def initialize(@capacity : Int32)
          @data = Hash(K, V).new
          @order = Array(K).new
        end

        def get(key : K) : V?
          if value = @data[key]?
            # Move to end (LRU)
            @order.delete(key)
            @order << key
            value
          else
            nil
          end
        end

        def set(key : K, value : V)
          if @data.has_key?(key)
            @order.delete(key)
          elsif @order.size >= @capacity
            oldest = @order.shift
            @data.delete(oldest)
          end

          @data[key] = value
          @order << key
        end
      end

      # Memoization for expensive computations
      def fibonacci(n : Int32) : Int32
        @@cache = Hash(Int32, Int32).new

        return @@cache[n] if @@cache.has_key?(n)

        result = if n <= 1
                  n
                else
                  fibonacci(n - 1) + fibonacci(n - 2)
                end

        @@cache[n] = result
        result
      end

Database Optimization:
   Efficient data access patterns.

   .. code-block:: crystal

      # Batch operations
      def insert_users_batch(users : Array(User)) : Nil
        values = users.map do |user|
          "(#{user.id}, '#{user.name}', '#{user.email}')"
        end.join(",")

        query = "INSERT INTO users (id, name, email) VALUES #{values}"
        @db.exec(query)
      end

      # Connection pooling
      class ConnectionPool
        def initialize(@url : String, @size : Int32)
          @pool = Channel(DB::Connection).new(@size)
          @size.times { @pool.send(DB.connect(@url)) }
        end

        def with_connection
          connection = @pool.receive
          begin
            yield connection
          ensure
            @pool.send(connection)
          end
        end
      end

      # Usage
      pool = ConnectionPool.new(ENV["DATABASE_URL"], 10)
      pool.with_connection do |conn|
        users = conn.query_all("SELECT * FROM users", as: User)
      end

Performance Monitoring:
   Runtime performance tracking.

   .. code-block:: crystal

      # Performance metrics
      struct Metrics
        property request_count = 0
        property total_time = 0.0
        property error_count = 0

        def record_request(time : Float64, success : Bool)
          @request_count += 1
          @total_time += time
          @error_count += 1 unless success
        end

        def average_time : Float64
          @request_count > 0 ? @total_time / @request_count : 0.0
        end

        def error_rate : Float64
          @request_count > 0 ? @error_count.to_f / @request_count : 0.0
        end
      end

      # Middleware for HTTP performance
      class PerformanceMiddleware
        def initialize(@app : HTTP::Handler, @metrics : Metrics)
        end

        def call(context)
          start_time = Time.monotonic
          begin
            @app.call(context)
            success = true
          rescue ex
            success = false
            raise ex
          ensure
            end_time = Time.monotonic
            duration = (end_time - start_time).total_milliseconds
            @metrics.record_request(duration, success)
          end
        end
      end

Documentation Best Practices: Code & Types for AI Agents
--------------------------------------------------------
Code Documentation:
   Clear, comprehensive API documentation:

   .. code-block:: crystal

      # User authentication and authorization service.
      #
      # This service handles user registration, login, and permission management.
      # It uses bcrypt for password hashing and JWT tokens for session management.
      #
      # ## Example
      #
      # ```
      # service = AuthService.new(database_connection)
      # token = service.register("alice@example.com", "password123")
      # user = service.authenticate(token)
      # ```
      #
      # ## Security Notes
      #
      # * Passwords are hashed using bcrypt with a cost factor of 12
      # * JWT tokens expire after 24 hours
      # * Rate limiting is applied to prevent brute force attacks
      class AuthService
        # Register a new user with email and password.
        #
        # *email* - User's email address (must be valid format)
        # *password* - User's password (minimum 8 characters)
        #
        # Returns JWT token on successful registration
        #
        # Raises `ValidationError` if email is invalid or password too short
        # Raises `DuplicateUserError` if email already exists
        #
        # ## Examples
        #
        # ```
        # service = AuthService.new(db)
        # token = service.register("user@example.com", "securepassword")
        # puts token  # => "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
        # ```
        def register(email : String, password : String) : String
          validate_email(email)
          validate_password(password)

          hashed_password = BCrypt::Password.create(password, cost: 12)
          user = create_user(email, hashed_password)
          generate_jwt_token(user)
        end

        # Authenticate user with JWT token.
        #
        # *token* - JWT token from previous authentication
        #
        # Returns authenticated user or `nil` if token is invalid
        #
        # ## Examples
        #
        # ```
        # user = service.authenticate("valid.jwt.token")
        # user.email  # => "user@example.com"
        #
        # user = service.authenticate("invalid.token")
        # user  # => nil
        # ```
        def authenticate(token : String) : User?
          payload = decode_jwt_token(token)
          return nil unless payload

          user_id = payload["user_id"].as_i
          find_user(user_id)
        rescue JWT::Error
          nil
        end
      end

Type Documentation:
   Document complex types and unions:

   .. code-block:: crystal

      # Result type for operations that can fail.
      #
      # This type provides a type-safe way to handle operations that might
      # succeed or fail without using exceptions.
      #
      # ## Type Parameters
      #
      # *T* - Success value type
      # *E* - Error type
      #
      # ## Examples
      #
      # ```
      # # Success case
      # result = Result(Int32, String).success(42)
      # result.success?  # => true
      # result.value     # => 42
      #
      # # Error case
      # result = Result(Int32, String).error("Invalid input")
      # result.success?  # => false
      # result.error     # => "Invalid input"
      # ```
      class Result(T, E)
        # Creates a successful result.
        #
        # *value* - The success value
        #
        # Returns `Result` with success state
        def self.success(value : T) : self
          new(true, value, nil)
        end

        # Creates an error result.
        #
        # *error* - The error value
        #
        # Returns `Result` with error state
        def self.error(error : E) : self
          new(false, nil, error)
        end

        # Returns `true` if the result is successful.
        #
        # ## Examples
        #
        # ```
        # Result.success(42).success?  # => true
        # Result.error("fail").success? # => false
        # ```
        def success? : Bool
          @success
        end

        # Returns the success value.
        #
        # Raises `NilAssertionError` if result is an error
        #
        # ## Examples
        #
        # ```
        # Result.success(42).value  # => 42
        # Result.error("fail").value # raises NilAssertionError
        # ```
        def value : T
          @value.not_nil!
        end

        # Returns the error value.
        #
        # Raises `NilAssertionError` if result is successful
        #
        # ## Examples
        #
        # ```
        # Result.success(42).error  # raises NilAssertionError
        # Result.error("fail").error # => "fail"
        # ```
        def error : E
          @error.not_nil!
        end
      end

Module Documentation:
   Document module functionality and usage:

   .. code-block:: crystal

      # String manipulation utilities for AI agents.
      #
      # This module provides common string operations optimized for
      # text processing and data cleaning tasks.
      #
      # ## Features
      #
      # * Safe string operations with nil handling
      # * Unicode-aware text processing
      # * Performance-optimized implementations
      #
      # ## Example
      #
      # ```
      # require "./string_utils"
      #
      # text = "  Hello, World!  "
      # cleaned = StringUtils.clean(text)
      # cleaned  # => "Hello, World!"
      #
      # words = StringUtils.extract_words(text)
      # words    # => ["Hello", "World"]
      # ```
      module StringUtils
        # Safely cleans and normalizes text.
        #
        # *text* - Input string (can be `nil`)
        #
        # Returns cleaned string or empty string if input is `nil`
        #
        # ## Operations
        #
        # * Removes leading/trailing whitespace
        # * Normalizes multiple spaces to single space
        # * Removes control characters
        # * Preserves Unicode characters
        #
        # ## Examples
        #
        # ```
        # StringUtils.clean(nil)           # => ""
        # StringUtils.clean("  hello  ")   # => "hello"
        # StringUtils.clean("hello\nworld")  # => "hello world"
        # ```
        def self.clean(text : String?) : String
          return "" unless text

          text
            .strip
            .gsub(/\s+/, " ")
            .gsub(/[\x00-\x1F\x7F]/, "")
        end

        # Extracts words from text.
        #
        # *text* - Input string
        #
        # Returns array of words (letters and numbers only)
        #
        # ## Examples
        #
        # ```
        # StringUtils.extract_words("Hello, world! 123")
        # # => ["Hello", "world", "123"]
        # ```
        def self.extract_words(text : String) : Array(String)
          text
            .downcase
            .scan(/[a-z0-9]+/)
            .map(&.[0])
        end
      end

Documentation Best Practices: API & Tools for AI Agents
-------------------------------------------------------
API Documentation:
   REST API endpoint documentation:

   .. code-block:: crystal

      # User management API endpoints.
      #
      # This controller provides CRUD operations for user management
      # with proper authentication and validation.
      #
      # ## Authentication
      #
      # All endpoints except `POST /users` require a valid JWT token
      # in the `Authorization: Bearer <token>` header.
      #
      # ## Rate Limiting
      #
      # API is rate-limited to 100 requests per minute per IP address.
      #
      # ## Error Responses
      #
      # * `400 Bad Request` - Invalid input data
      # * `401 Unauthorized` - Missing or invalid token
      # * `403 Forbidden` - Insufficient permissions
      # * `404 Not Found` - Resource not found
      # * `429 Too Many Requests` - Rate limit exceeded
      class UserController
        # Creates a new user account.
        #
        # ## Endpoint
        #
        # `POST /api/v1/users`
        #
        # ## Request Body
        #
        # ```json
        # {
        #   "email": "user@example.com",
        #   "password": "securepassword",
        #   "name": "John Doe"
        # }
        # ```
        #
        # ## Response
        #
        # ### 201 Created
        #
        # ```json
        # {
        #   "id": 123,
        #   "email": "user@example.com",
        #   "name": "John Doe",
        #   "created_at": "2024-01-15T10:30:00Z",
        #   "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
        # }
        # ```
        #
        # ### 400 Bad Request
        #
        # ```json
        # {
        #   "error": "Invalid email format"
        # }
        # ```
        #
        # ### 409 Conflict
        #
        # ```json
        # {
        #   "error": "Email already exists"
        # }
        # ```
        def create(context : HTTP::Server::Context)
          user_data = parse_user_data(context.request)
          validation_errors = validate_user_data(user_data)

          if validation_errors.any?
            context.response.status_code = 400
            context.response.print({error: validation_errors.join(", ")}.to_json)
            return
          end

          begin
            user = create_user(user_data)
            token = generate_jwt_token(user)

            context.response.status_code = 201
            context.response.print({
              id: user.id,
              email: user.email,
              name: user.name,
              created_at: user.created_at,
              token: token
            }.to_json)
          rescue ex : DuplicateUserError
            context.response.status_code = 409
            context.response.print({error: "Email already exists"}.to_json)
          end
        end
      end

Documeation Generation:
   Autoted documentation tools:

   .. code-block:: crystal

      # Generate API documentation
      #
      # Run from project root:
      # crystal docs

      # Serve documentation locally
      #
      # crystal docs --serve
      # Open http://localhost:3000

      # Generate specific format
      #
      # crystal docs --format html
      # crystal docs --format json

      # Custom documentation configuration
      #
      # In shard.yml:
      # documentation:
      #   homepage: https://myapp.com/docs
      #   source_code_uri: https://github.com/user/myapp/blob/{version}/src/%{path}#L%{line}
      #   logo: assets/logo.png

      # Include examples in documentation
      #
      # The following will be rendered as executable examples:
      #
      # ```
      # # This is an example that users can run
      # service = AuthService.new(db)
      # token = service.register("user@example.com", "password")
      # puts token
      # ```

README Documentation:
   Comprehensive project documentation:

   .. code-block:: markdown

      # MyApp

      A Crystal application for user management and authentication.

      ## Features

      * User registration and authentication
      * JWT-based session management
      * RESTful API with comprehensive documentation
      * Rate limiting and security measures
      * PostgreSQL database integration

      ## Installation

      ### Prerequisites

      * Crystal 1.10.0 or higher
      * PostgreSQL 12 or higher
      * Node.js (for asset compilation, if applicable)

      ### Setup

      ```bash
      # Clone the repository
      git clone https://github.com/user/myapp.git
      cd myapp

      # Install dependencies
      shards install

      # Setup database
      createdb myapp_development
      psql myapp_development < schema.sql

      # Configure environment
      cp .env.example .env
      # Edit .env with your settings

      # Run the application
      crystal run src/my_app.cr
      ```

      ## Usage

      ### API Endpoints

      #### Authentication
      - `POST /api/v1/users` - Register new user
      - `POST /api/v1/auth/login` - User login
      - `POST /api/v1/auth/logout` - User logout

      #### User Management
      - `GET /api/v1/users/profile` - Get current user
      - `PUT /api/v1/users/profile` - Update user profile
      - `DELETE /api/v1/users/account` - Delete account

      ### Configuration

      See `config/` directory for configuration options:
      - `database.cr` - Database settings
      - `server.cr` - Server configuration
      - `security.cr` - Security settings

      ## Development

      ### Running Tests

      ```bash
      # Run all tests
      crystal spec

      # Run with coverage
      crystal spec --coverage

      # Run specific test file
      crystal spec spec/auth_spec.cr
      ```

      ### Code Quality

      ```bash
      # Format code
      crystal tool format src/

      # Run linter
      ameba src/

      # Generate documentation
      crystal docs
      ```

      ## Deployment

      ### Docker

      ```bash
      # Build image
      docker build -t myapp .

      # Run container
      docker run -p 3000:3000 -e DATABASE_URL=... myapp
      ```

      ### Production Build

      ```bash
      # Build optimized binary
      crystal build src/my_app.cr --release --static

      # The binary will be available as `myapp`
      ./myapp
      ```

      ## Contributing

      1. Fork the repository
      2. Create a feature branch
      3. Make your changes
      4. Add tests for new functionality
      5. Run `crystal spec` and `ameba src/`
      6. Submit a pull request

      ## License

      MIT License - see LICENSE file for details.

References and Further Reading for AI Agents
============================================
This section provides curated references to official documentation, real-world examples, and additional resources for generating and understanding Crystal code.

Official Crystal Documentation
------------------------------

Crystal Language Website:
   The primary source for Crystal language information.

   * **URL:** https://crystal-lang.org/

   * **Content:** Language overview, getting started guides, API reference

   * **Use Case:** Quick lookups for syntax and standard library functions

Crystal API Documentation:
   Comprehensive API reference for all standard library modules.

   * **URL:** https://crystal-lang.org/api/

   * **Content:** Detailed class and method documentation with examples

   * **Use Case:** Understanding method signatures and return types

Crystal Book:
   In-depth guide to the Crystal programming language.

   * **URL:** https://crystal-lang.org/reference/

   * **Content:** Tutorials, syntax guides, best practices

   * **Use Case:** Learning advanced concepts and language features

Rosetta Code - Crystal:
   Collection of programming tasks solved in Crystal.

   * **URL:** https://rosettacode.org/wiki/Category:Crystal

   * **Content:** Practical code examples for common algorithms and problems

   * **Use Case:** Finding idiomatic solutions to programming challenges

Awesome Crystal:
   Curated list of Crystal libraries, tools, and resources.

   * **URL:** https://github.com/veelenga/awesome-crystal

   * **Content:** Frameworks, databases, web servers, and more

   * **Use Case:** Discovering libraries for specific use cases

Crystal Shards:
   Package manager and shard registry.

   * **URL:** https://github.com/crystal-lang/shards

   * **Content:** Dependency management and package discovery

   * **Use Case:** Managing project dependencies

num.cr:
   Arbitrary precision numeric library for Crystal.

   * **URL:** https://github.com/crystal-lang/num.cr

   * **Content:** Big integers, floats, and mathematical operations

   * **Use Case:** High-precision calculations and cryptographic applications

Crystal-DB:
   Database toolkit for Crystal.

   * **URL:** https://github.com/crystal-lang/crystal-db

   * **Content:** Database drivers and query builders

   * **Use Case:** Database integration and ORM patterns

Kemal:
   Fast, effective, simple web framework for Crystal.

   * **URL:** https://kemalcr.com/

   * **Content:** Web application framework with routing and middleware

   * **Use Case:** Building web APIs and applications

Lucky Framework:
   Full-featured web framework with conventions.

   * **URL:** https://luckyframework.org/

   * **Content:** MVC framework with built-in security and testing

   * **Use Case:** Rapid web application development

Crystal Performance Guide:
   Tips for writing high-performance Crystal code.

   * **URL:** https://crystal-lang.org/reference/guides/performance.html

   * **Content:** Memory management, compilation flags, profiling

   * **Use Case:** Optimizing code for production

Benchmark.cr:
   Performance benchmarking library.

   * **URL:** https://github.com/crystal-lang/benchmark.cr

   * **Content:** Measuring execution time and memory usage

   * **Use Case:** Performance testing and optimization

Crystal Security Guide:
   Security considerations for Crystal applications.

   * **URL:** https://crystal-lang.org/reference/guides/security.html

   * **Content:** Input validation, SQL injection prevention, authentication

   * **Use Case:** Building secure applications

Ameba:
   Static code analysis tool for Crystal.

   * **URL:** https://github.com/crystal-ameba/ameba

   * **Content:** Code quality checks and style enforcement

   * **Use Case:** Maintaining code quality and consistency

Crystal Spec:
   Built-in testing framework.

   * **URL:** https://crystal-lang.org/reference/syntax_and_semantics/spec.html

   * **Content:** Unit testing, mocking, and test organization

   * **Use Case:** Writing and running tests

WebMock.cr:
   HTTP request mocking for tests.

   * **URL:** https://github.com/manastech/webmock.cr

   * **Content:** Stubbing HTTP requests in test environments

   * **Use Case:** Testing code that makes HTTP calls

Crystal by Example:
   Practical examples and tutorials.

   * **URL:** https://crystal-lang.org/reference/

   * **Content:** Step-by-step guides and code samples

   * **Use Case:** Learning through hands-on examples

Crystal Forum:
   Community discussion and Q&A.

   * **URL:** https://forum.crystal-lang.org/

   * **Content:** Questions, announcements, and discussions

   * **Use Case:** Getting help and sharing knowledge

GitHub Issues and Pull Requests:
   Real-world code examples and discussions.

   * **URL:** https://github.com/crystal-lang/crystal/issues

   * **Content:** Bug reports, feature requests, and code reviews

   * **Use Case:** Understanding real-world usage and edge cases

JSON::Serializable:
   Built-in JSON serialization support.

   * **URL:** https://crystal-lang.org/api/JSON/Serializable.html

   * **Content:** Automatic JSON mapping for structs and classes

   * **Use Case:** API development and data exchange

HTTP::Client:
   HTTP client for making requests.

   * **URL:** https://crystal-lang.org/api/HTTP/Client.html

   * **Content:** GET, POST, and other HTTP methods

   * **Use Case:** Web scraping and API integration

YAML Support:
   YAML parsing and generation.

   * **URL:** https://crystal-lang.org/api/YAML.html

   * **Content:** Configuration file handling

   * **Use Case:** Application configuration

Regex Engine:
   Regular expression support.

   * **URL:** https://crystal-lang.org/api/Regex.html

   * **Content:** Pattern matching and text processing

   * **Use Case:** Input validation and parsing

Crystal Contributing Guide:
   How to contribute to the Crystal project.

   * **URL:** https://github.com/crystal-lang/crystal/blob/master/CONTRIBUTING.md

   * **Content:** Development setup, coding standards, testing

   * **Use Case:** Contributing code or documentation

Style Guide:
   Crystal coding style and conventions.

   * **URL:** https://github.com/crystal-lang/crystal/blob/master/STYLE.md

   * **Content:** Naming conventions, formatting, best practices

   * **Use Case:** Writing idiomatic Crystal code

Using These References
----------------------
When generating Crystal code, agents should:

1. **Cross-reference official docs** for accurate syntax and API usage

2. **Study Rosetta Code examples** for algorithmic patterns

3. **Use Awesome Crystal** to find appropriate libraries

4. **Check community resources** for real-world usage patterns

5. **Follow security and performance guides** for production-ready code

These references provide the foundation for generating robust, idiomatic Crystal code that follows community standards and best practices.

