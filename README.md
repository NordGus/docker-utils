# Docker Utils
This is a repository for simple Dockerfiles I use in my personal projects.

# Dockerfile.new-rails-app
Create rails applications without having to install Ruby's Runtime.
## Usage
1. Build image:
    ```bash
    docker build -t my-app - < Dockerfile.new-rails-app
    ```
    Ruby version and Rails version are optional build args:
    ```bash
    docker build --build-arg RUBY_VERISON=your.ruby.version \
    --build-arg RAILS_VERSION=your.rails.version \ 
    -t my-app - < /path/to/docker-utils/Dockerfile.new-rails-app
    ```
2. Run:
    ```bash
    docker run -it --rm --name my-app -v "$PWD":/var my-app rails new my-app --database=postgresql
    ```