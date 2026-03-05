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

