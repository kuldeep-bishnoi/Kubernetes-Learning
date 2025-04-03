# Dockerfile Creation

## 🎯 Overview

A Dockerfile is a text file that contains instructions for building a Docker image. It's like a recipe that tells Docker how to prepare and package your application. In this section, we'll learn how to create efficient Dockerfiles for different types of applications.

## 📝 What You'll Learn

- Dockerfile syntax and commands
- Best practices for creating efficient Dockerfiles
- How to build and tag images
- How to push images to a registry

## 📋 Basic Dockerfile Structure

Here's a simple Dockerfile structure:

```dockerfile
# Base image
FROM ubuntu:20.04

# Metadata (optional)
LABEL maintainer="Your Name <your.email@example.com>"
LABEL version="1.0"

# Set environment variables
ENV APP_HOME=/app

# Set working directory
WORKDIR $APP_HOME

# Install dependencies
RUN apt-get update && \
    apt-get install -y python3 python3-pip && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Copy application files
COPY . .

# Install Python dependencies
RUN pip3 install -r requirements.txt

# Expose port
EXPOSE 5000

# Command to run when container starts
CMD ["python3", "app.py"]
```

## 📖 Common Dockerfile Instructions

| Instruction | Description | Example |
|-------------|-------------|---------|
| `FROM` | Specifies the base image | `FROM python:3.9-slim` |
| `WORKDIR` | Sets the working directory | `WORKDIR /app` |
| `COPY` | Copies files from host to container | `COPY . .` |
| `ADD` | Similar to COPY but can extract archives and fetch URLs | `ADD https://example.com/file.tar.gz /app/` |
| `RUN` | Executes commands during build | `RUN pip install -r requirements.txt` |
| `ENV` | Sets environment variables | `ENV DEBUG=false` |
| `EXPOSE` | Documents which ports the container listens on | `EXPOSE 80` |
| `CMD` | Default command to run when container starts | `CMD ["python", "app.py"]` |
| `ENTRYPOINT` | Configures container to run as executable | `ENTRYPOINT ["python"]` |
| `VOLUME` | Creates a mount point for external volumes | `VOLUME /data` |
| `USER` | Sets the user for subsequent instructions | `USER appuser` |
| `ARG` | Defines build-time variables | `ARG VERSION=latest` |
| `LABEL` | Adds metadata to the image | `LABEL environment="production"` |

## 🏗️ Building Your First Docker Image

Let's create a simple web application and Dockerize it:

1. Create a new directory for your project:
   ```bash
   mkdir my-docker-app
   cd my-docker-app
   ```

2. Create a simple Python web application (`app.py`):
   ```python
   from flask import Flask
   app = Flask(__name__)
   
   @app.route('/')
   def hello():
       return "Hello, Docker World!"
   
   if __name__ == '__main__':
       app.run(host='0.0.0.0', port=5000)
   ```

3. Create a `requirements.txt` file:
   ```
   flask==2.0.1
   ```

4. Create a Dockerfile:
   ```dockerfile
   FROM python:3.9-slim
   
   WORKDIR /app
   
   COPY requirements.txt .
   RUN pip install --no-cache-dir -r requirements.txt
   
   COPY . .
   
   EXPOSE 5000
   
   CMD ["python", "app.py"]
   ```

5. Build the Docker image:
   ```bash
   docker build -t my-flask-app .
   ```

6. Run the container:
   ```bash
   docker run -p 5000:5000 my-flask-app
   ```

7. Access your application at http://localhost:5000

## 🛠️ Building Images for Different Languages

### Node.js Application

```dockerfile
FROM node:14-alpine

WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .

EXPOSE 3000

CMD ["npm", "start"]
```

### Java Application

```dockerfile
FROM maven:3.8-openjdk-11 AS build

WORKDIR /app
COPY pom.xml .
COPY src ./src

RUN mvn package -DskipTests

FROM openjdk:11-jre-slim

WORKDIR /app
COPY --from=build /app/target/*.jar app.jar

EXPOSE 8080
CMD ["java", "-jar", "app.jar"]
```

### Go Application

```dockerfile
FROM golang:1.16-alpine AS build

WORKDIR /app
COPY go.* ./
RUN go mod download

COPY . .
RUN go build -o server .

FROM alpine:3.14

WORKDIR /app
COPY --from=build /app/server .

EXPOSE 8080
CMD ["./server"]
```

## 🏆 Best Practices for Dockerfile Creation

1. **Use a specific tag** for your base image (e.g., `python:3.9-slim` instead of just `python`)
2. **Use smaller base images** when possible (Alpine or slim variants)
3. **Combine RUN commands** with `&&` to reduce layers
4. **Clean up after installations** to reduce image size
5. **Use .dockerignore** file to exclude unnecessary files
6. **Copy dependencies first, then application code** to leverage Docker's layer caching
7. **Set a non-root user** for better security
8. **Use multi-stage builds** for compiled languages to reduce final image size
9. **Specify default values** for environment variables
10. **Document exposed ports** with the EXPOSE instruction

## 🧪 Exercises

1. Create a Dockerfile for a simple HTML website using NGINX
2. Build a Docker image for a React application
3. Create a multi-stage Dockerfile for a compiled language of your choice

Each exercise has sample code and solutions in the [exercises](./exercises/) directory.

## 📦 Pushing Images to Docker Hub

After building your image, you can share it via Docker Hub:

1. Tag your image with your Docker Hub username:
   ```bash
   docker tag my-flask-app yourusername/my-flask-app
   ```

2. Log in to Docker Hub:
   ```bash
   docker login
   ```

3. Push the image:
   ```bash
   docker push yourusername/my-flask-app
   ```

## 📚 Additional Resources

- [Dockerfile reference](https://docs.docker.com/engine/reference/builder/)
- [Best practices for writing Dockerfiles](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)
- [.dockerignore file](https://docs.docker.com/engine/reference/builder/#dockerignore-file)

## 🔄 Next Steps

Continue to [Multi-Stage Builds](../03-multi-stage-builds/) to learn how to optimize your Docker images. 