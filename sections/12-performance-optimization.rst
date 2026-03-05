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

