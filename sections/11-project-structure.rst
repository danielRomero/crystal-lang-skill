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

