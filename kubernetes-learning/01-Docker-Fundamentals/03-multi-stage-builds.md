# Docker Multi-Stage Builds

## 🎯 Overview

Multi-stage builds are a powerful feature in Docker that allow you to create smaller, more efficient images by using multiple `FROM` statements in your Dockerfile. This technique is particularly useful for compiled languages where you need build tools during compilation but not in the final production image.

## 🤔 The Problem Multi-Stage Builds Solve

Imagine you're baking a cake:
1. You need many ingredients and tools (flour, sugar, bowls, mixers)
2. After baking, you only need the cake itself, not all the mess
3. Traditional Docker builds would package everything together

Multi-stage builds are like:
1. Making the cake in one kitchen
2. Taking only the finished cake to a clean, simple serving plate
3. Leaving all the tools, ingredients, and mess behind

## 📊 Before Multi-Stage Builds

**Option 1: One large image with everything**
- ✅ Simple Dockerfile
- ❌ Very large final image size
- ❌ Includes build tools (security risk)
- ❌ More vulnerabilities to patch

```dockerfile
FROM maven:3.8-jdk-11

WORKDIR /app
COPY . .
RUN mvn package

EXPOSE 8080
CMD ["java", "-jar", "target/myapp.jar"]
```

**Option 2: Shell script to build outside Docker**
- ✅ Small final image
- ❌ Complex build process
- ❌ Build environment inconsistencies
- ❌ Not fully containerized

```bash
# build.sh
mvn package
docker build -t myapp .
```

```dockerfile
FROM openjdk:11-jre-slim
COPY target/myapp.jar /app/myapp.jar
CMD ["java", "-jar", "/app/myapp.jar"]
```

## 📋 Multi-Stage Build Solution

A multi-stage build:
1. Uses multiple `FROM` statements in a single Dockerfile
2. Each `FROM` statement begins a new stage
3. Files can be copied from earlier stages to later ones
4. Only the final stage content exists in the resulting image

```dockerfile
# Stage 1: Build the application
FROM maven:3.8-jdk-11 AS build
WORKDIR /app
COPY pom.xml .
COPY src ./src
RUN mvn package

# Stage 2: Runtime stage with minimal footprint
FROM openjdk:11-jre-slim
WORKDIR /app
COPY --from=build /app/target/*.jar app.jar
EXPOSE 8080
CMD ["java", "-jar", "app.jar"]
```

## 🔍 How Multi-Stage Builds Work

1. Docker builds each stage as an intermediate image
2. Each stage can be used as a source for files in later stages
3. Only the contents of the final stage are included in the output image
4. Intermediate stages are cached but not included in the final image

## 💡 Key Benefits

- **Smaller Images**: Final image only includes what's needed to run the application
- **Improved Security**: Build-time dependencies and tools don't appear in the final image
- **Simplified Process**: One Dockerfile handles the entire build process
- **Best of Both Worlds**: Build tools available during build, minimal runtime image

## 🛠️ Practical Examples

### Example 1: Go Application

```dockerfile
# Stage 1: Build the application
FROM golang:1.17-alpine AS build

WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download

COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o main .

# Stage 2: Create minimal runtime image
FROM alpine:3.15

RUN apk --no-cache add ca-certificates
WORKDIR /root/
COPY --from=build /app/main .
CMD ["./main"]
```

### Example 2: React + Nginx Web App

```dockerfile
# Stage 1: Build the React application
FROM node:16-alpine AS build

WORKDIR /app
COPY package*.json ./
RUN npm install

COPY . .
RUN npm run build

# Stage 2: Serve the app with NGINX
FROM nginx:alpine
COPY --from=build /app/build /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### Example 3: Python Application with Dependencies

```dockerfile
# Stage 1: Build dependencies
FROM python:3.9 AS builder

WORKDIR /app
COPY requirements.txt .
RUN pip wheel --no-cache-dir --wheel-dir /app/wheels -r requirements.txt

# Stage 2: Create the final image
FROM python:3.9-slim

WORKDIR /app
COPY --from=builder /app/wheels /wheels
COPY --from=builder /app/requirements.txt .
RUN pip install --no-cache /wheels/*

COPY . .
CMD ["python", "app.py"]
```

## 🔨 Naming and Using Build Stages

You can name stages using `AS` and refer to them:

```dockerfile
FROM golang:1.17 AS builder
# ... build steps ...

FROM alpine:3.15
COPY --from=builder /app/binary /app/
```

You can even use an external image as a source:

```dockerfile
COPY --from=nginx:latest /etc/nginx/nginx.conf /nginx.conf
```

## 🧪 Hands-on Exercise: Multi-Stage Build for a Java Application

Let's create a simple Java application and optimize it using multi-stage builds:

### Step 1: Create a Simple Java Application

Create a directory for your project and add a simple Java application:

```bash
mkdir -p java-multistage/src/main/java/com/example
cd java-multistage
```

Create a `pom.xml` file:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>hello-docker</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
    </properties>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-jar-plugin</artifactId>
                <version>3.2.0</version>
                <configuration>
                    <archive>
                        <manifest>
                            <addClasspath>true</addClasspath>
                            <mainClass>com.example.HelloDocker</mainClass>
                        </manifest>
                    </archive>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

Create a Java file at `src/main/java/com/example/HelloDocker.java`:

```java
package com.example;

public class HelloDocker {
    public static void main(String[] args) {
        System.out.println("Hello from Docker Multi-Stage Build!");
        
        // Keep the application running to demonstrate container execution
        while (true) {
            try {
                System.out.println("Application running...");
                Thread.sleep(10000);
            } catch (InterruptedException e) {
                e.printStackTrace();
                break;
            }
        }
    }
}
```

### Step 2: Create a Multi-Stage Dockerfile

Create a `Dockerfile` in the project root:

```dockerfile
# Stage 1: Build the application
FROM maven:3.8-openjdk-11 AS build

WORKDIR /app
COPY pom.xml .
COPY src ./src

RUN mvn package

# Stage 2: Run the application
FROM openjdk:11-jre-slim

WORKDIR /app
COPY --from=build /app/target/hello-docker-1.0-SNAPSHOT.jar app.jar

CMD ["java", "-jar", "app.jar"]
```

### Step 3: Build the Docker Image

```bash
docker build -t hello-java-multistage .
```

### Step 4: Compare with Single-Stage Build

Create a `Dockerfile.single` for comparison:

```dockerfile
FROM maven:3.8-openjdk-11

WORKDIR /app
COPY pom.xml .
COPY src ./src

RUN mvn package

CMD ["java", "-jar", "target/hello-docker-1.0-SNAPSHOT.jar"]
```

Build the single-stage image:

```bash
docker build -t hello-java-single -f Dockerfile.single .
```

Compare the image sizes:

```bash
docker images | grep hello-java
```

You'll notice the multi-stage build image is significantly smaller!

### Step 5: Run the Container

```bash
docker run -it --rm hello-java-multistage
```

You should see output from your Java application.

## 📈 Advanced Techniques

### Building Multiple Images from One Dockerfile

You can build different images targeting specific stages:

```bash
# Build the full image (default)
docker build -t myapp .

# Build just the builder stage
docker build --target build -t myapp-builder .
```

### Using Build Arguments Across Stages

```dockerfile
ARG VERSION=latest

FROM baseimage:${VERSION} AS build
# ... build steps ...

FROM runtime:${VERSION}
# ... runtime steps ...
```

### Using Conditions in Multi-Stage Builds

```dockerfile
FROM base AS development
RUN apt-get update && apt-get install -y development-tools

FROM base AS production
COPY --from=development /app/build /app/build
RUN optimize-app

FROM ${BUILD_ENV:-production}
# This will use the production stage by default
# or development if BUILD_ENV=development
```

## 🏆 Best Practices

1. **Name your stages** for clarity and reference
2. **Put infrequently changing layers early** in the Dockerfile to maximize caching
3. **Only copy what you need** from one stage to the next
4. **Use appropriate base images** for each stage
5. **Optimize layer caching** in each stage
6. **Remove unnecessary files** as you go to keep intermediate images smaller
7. **Consider security scanning** on your final image
8. **Specify exact versions** of base images for reproducibility

## 🔄 Next Steps

Now that you understand Docker fundamentals including multi-stage builds, you're ready to move on to [Kubernetes Introduction](../../02-Kubernetes-Introduction/) to learn about container orchestration. 