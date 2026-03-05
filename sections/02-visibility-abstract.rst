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

