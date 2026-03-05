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

