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

