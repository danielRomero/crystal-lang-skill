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

