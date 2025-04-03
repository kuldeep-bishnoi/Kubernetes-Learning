# Docker Basics - Hands-on Practice

This guide contains hands-on exercises to help you understand Docker fundamentals. By completing these exercises, you'll gain practical experience with Docker containers and commands.

## Prerequisites

- Docker installed on your computer ([Installation Guide](https://docs.docker.com/get-docker/))
- Terminal or command prompt access

## Exercise 1: Hello Docker World

Let's start with the most basic Docker command to verify everything is working properly.

```bash
# Run the hello-world container
docker run hello-world
```

**What happened?**
1. Docker looked for the "hello-world" image locally (and didn't find it)
2. Docker pulled the image from Docker Hub (a public registry)
3. Docker created a container from the image
4. The container ran, displayed a message, and then exited

**Try this:**
- Run the command again. Notice how it's faster the second time (no need to pull the image).

## Exercise 2: Working with Docker Images

Let's explore Docker images - the blueprints used to create containers.

```bash
# List all images on your system
docker images

# Pull the official nginx web server image
docker pull nginx

# Pull a specific version of an image
docker pull ubuntu:20.04

# List images again to see what you pulled
docker images

# Search for images on Docker Hub
docker search python
```

## Exercise 3: Running Containers

Now let's run containers from the images we pulled.

```bash
# Run an nginx container in detached mode (in the background)
docker run --name my-nginx -d -p 8080:80 nginx

# Check running containers
docker ps

# Access the nginx web server in your browser
# Open: http://localhost:8080
```

**What did we do?**
- `--name my-nginx`: Named our container "my-nginx"
- `-d`: Ran in detached mode (in the background)
- `-p 8080:80`: Mapped port 8080 on our computer to port 80 in the container
- `nginx`: Used the nginx image we pulled earlier

## Exercise 4: Container Lifecycle

Let's learn how to manage the container's lifecycle.

```bash
# Stop the running nginx container
docker stop my-nginx

# Check all containers (including stopped ones)
docker ps -a

# Start the container again
docker start my-nginx

# Restart a container
docker restart my-nginx

# Stop the container again
docker stop my-nginx

# Remove the stopped container
docker rm my-nginx

# Run a container and remove it automatically when it stops
docker run --rm --name temp-nginx -d -p 8080:80 nginx

# Stop the temporary container
docker stop temp-nginx
# Note: The container is automatically removed after stopping
```

## Exercise 5: Exploring Running Containers

Let's interact with a running container.

```bash
# Start a new nginx container
docker run --name web-server -d -p 8080:80 nginx

# Get detailed information about the container
docker inspect web-server

# View the logs from the container
docker logs web-server

# Follow the logs (continuous output)
docker logs -f web-server
# (Press Ctrl+C to exit)

# Execute a command inside the running container
docker exec web-server ls -la /usr/share/nginx/html

# Start an interactive shell inside the container
docker exec -it web-server /bin/bash

# Inside the container, you can run commands
ls -la
cat /etc/nginx/nginx.conf
exit  # Exit the container shell

# Clean up
docker stop web-server
docker rm web-server
```

## Exercise 6: Running Different Applications

Let's try running different types of containers.

```bash
# Run a simple Ubuntu container interactively
docker run -it --name ubuntu-playground ubuntu:20.04 /bin/bash

# Inside the container, you can run Ubuntu commands
apt update
ls -la
echo "I'm inside a container!" > test.txt
cat test.txt
exit  # Exit the container

# The container has stopped, but it still exists
docker ps -a

# Start it again and reconnect
docker start ubuntu-playground
docker exec -it ubuntu-playground /bin/bash

# Check if your test.txt file is still there
cat test.txt

# Exit and clean up
exit
docker stop ubuntu-playground
docker rm ubuntu-playground
```

## Exercise 7: Container Resource Limits

Learn how to limit container resources.

```bash
# Run a container with memory limits
docker run --name limited-nginx -d -p 8080:80 --memory="200m" --cpus="0.5" nginx

# Check the resource limits
docker inspect limited-nginx | grep -A 10 "HostConfig"

# Clean up
docker stop limited-nginx
docker rm limited-nginx
```

## Challenge Exercise: Run a Multi-Container Application

Try to run a simple web application that uses both a web server and a database.

```bash
# First, create a network for the containers to communicate
docker network create my-app-network

# Run a MySQL database container
docker run --name app-db \
  --network my-app-network \
  -e MYSQL_ROOT_PASSWORD=mypassword \
  -e MYSQL_DATABASE=myapp \
  -d mysql:5.7

# Run a simple web application that connects to the database
docker run --name app-web \
  --network my-app-network \
  -e DATABASE_HOST=app-db \
  -e DATABASE_USER=root \
  -e DATABASE_PASSWORD=mypassword \
  -e DATABASE_NAME=myapp \
  -p 8080:80 \
  -d wordpress

# Visit http://localhost:8080 in your browser to see the WordPress setup page

# Clean up when done
docker stop app-web app-db
docker rm app-web app-db
docker network rm my-app-network
```

## What's Next?

After completing these exercises, you should be comfortable with:
- Running and managing Docker containers
- Working with Docker images
- Understanding container lifecycle
- Limiting container resources
- Basic container networking

Next, we'll learn how to create your own Docker images with Dockerfiles in the [next section](../02-dockerfile-creation/). 