# Docker Basics: Explained Simply

## What Are Containers?

Imagine you have a box of your favorite toys. You can take this box anywhere - to a friend's house, to school, or on vacation. No matter where you go, all your toys are exactly the same and work the same way.

Containers are like these toy boxes, but for computer programs! A container packages everything a program needs to run:
- The program itself
- Libraries (helper programs)
- Configuration files
- Everything else it needs

This way, your program works exactly the same on any computer that can run Docker - your laptop, a big server, or in the cloud!

### Example: See Containers in Action

Try running this command to see a simple container that says hello:

```bash
docker run hello-world
```

**What you'll see:**
```
Hello from Docker!
This message shows that your installation appears to be working correctly.

...more text explaining what happened...
```

## Why Use Containers?

### The Problem Containers Solve

Imagine you're baking cookies with friends:
- Your cookies come out flat
- Your friend's cookies are too hard
- Another friend's cookies are perfect

You all used the same recipe! Why are the cookies different? Because your kitchens are different - different ovens, ingredients, or tools.

This happens with software too. A program might work on the developer's computer but fail on yours because of tiny differences in the environment.

### How Containers Help

Containers solve this by packaging not just the program, but its entire environment.

It's like if you could package up your entire kitchen - oven, ingredients, tools, and even the air temperature - and ship it to your friends. Now everyone's cookies would come out exactly the same!

### Example: Run the Same Program on Any Computer

Try running a web server container:

```bash
docker run -d -p 80:80 nginx
```

**What you'll see:**
```
Unable to find image 'nginx:latest' locally
latest: Pulling from library/nginx
a2abf6c4d29d: Pull complete
a9edb18cadd1: Pull complete
589b7251471a: Pull complete
186b1aaa4aa6: Pull complete
b4df32aa5a72: Pull complete
a0bcbecc962e: Pull complete
Digest: sha256:0d17b565c37bcbd895e9d92315a05c1c3c9a29f762b011a10c54a66cd53c9b31
Status: Downloaded newer image for nginx:latest
7d14f39c1959c818373f7ccf14b5c98a8d0a9140867b09469f3c6ae3a11baee2
```

Now open your web browser and go to http://localhost:80
You'll see the NGINX welcome page!

## Containers vs. Virtual Machines

### Virtual Machines (Heavy)

Imagine building an entire tiny house, with its own foundation, walls, plumbing, electricity, and furniture, just to have a place to put one toy.

Virtual machines are like this - they:
- Contain an entire operating system
- Take up lots of space (gigabytes)
- Take minutes to start
- Need their own CPU and memory allocation

### Containers (Light)

Containers are like renting just one room in a hotel. The hotel already has the foundation, plumbing, and electricity - you just get your own space that's separated from others.

Containers:
- Share the host computer's operating system
- Are tiny (megabytes)
- Start in seconds
- Use less CPU and memory

### Example: See the Difference in Size

Let's compare the size of a container to a virtual machine:

```bash
docker images nginx
```

**What you'll see:**
```
REPOSITORY   TAG       IMAGE ID       CREATED         SIZE
nginx        latest    605c77e624dd   3 weeks ago     141MB
```

An Ubuntu virtual machine, however, would be at least 1.5GB - more than 10 times larger!

## How Docker Works: A Simple Explanation

Imagine Docker as a toy factory with three main parts:

1. **Blueprint (Docker Image)**: The detailed instructions for making a toy
2. **Toy (Container)**: The actual toy made from the blueprint
3. **Toy Box (Docker Registry)**: A place to store and share blueprints

### Docker Workflow

1. **Create/Find a Blueprint**: Write a Dockerfile or get an image from Docker Hub
2. **Build a Toy**: Create a container from the image
3. **Play!**: Run the container to use your program
4. **Share**: Store your image in a registry for others to use

### Example: See Images and Containers

View all images on your computer:
```bash
docker images
```

**What you'll see:**
```
REPOSITORY   TAG       IMAGE ID       CREATED         SIZE
nginx        latest    605c77e624dd   3 weeks ago     141MB
hello-world  latest    feb5d9fea6a5   16 months ago   13.3kB
```

View running containers:
```bash
docker ps
```

**What you'll see (if you ran nginx earlier):**
```
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS                NAMES
7d14f39c1959   nginx     "/docker-entrypoint.…"   10 minutes ago  Up 10 minutes  0.0.0.0:80->80/tcp   relaxed_swirles
```

## Basic Docker Commands with Examples

### Running Containers

```bash
# Run and immediately enter a Ubuntu container
docker run -it ubuntu bash
```

**What you'll see:**
```
Unable to find image 'ubuntu:latest' locally
latest: Pulling from library/ubuntu
e96e057aae67: Pull complete
Digest: sha256:0bced47fffa3361afa981854fcabcd4577cd43502c0a8e11d4b98bb416f6b20f
Status: Downloaded newer image for ubuntu:latest
root@3f4ab94a5b1b:/#
```

Now you're inside the container! Try commands like:
```bash
ls
cat /etc/os-release
```

Type `exit` when you're done.

### Stopping and Starting Containers

```bash
# List all running containers
docker ps

# Stop a container
docker stop 7d14f39c1959  # Use your container ID from docker ps

# List all containers (including stopped ones)
docker ps -a

# Start a stopped container
docker start 7d14f39c1959
```

### Example: Running a Container in the Background

Run a database container:
```bash
docker run -d --name my-database -e MYSQL_ROOT_PASSWORD=password mysql:5.7
```

**What you'll see:**
```
Unable to find image 'mysql:5.7' locally
5.7: Pulling from library/mysql
...several lines of downloading progress...
Digest: sha256:b3a86578a582617214477d91e47e850f9e24eb95819315e750921e486c73d785
Status: Downloaded newer image for mysql:5.7
a93572c9bc6f737ee32e335a6c200aae92feb6d0cbfd095d14e6358a1bd3d3de
```

Check that it's running:
```bash
docker ps
```

Connect to the MySQL database in the container:
```bash
docker exec -it my-database mysql -u root -p
```

Enter `password` when prompted, and you'll be in the MySQL shell:
```
mysql>
```

Type `exit` to leave the MySQL shell.

## Building Your Own Container

Imagine you're writing instructions for building your own custom toy. You write down each step: get these parts, connect them like this, paint it blue, etc.

In Docker, you write these instructions in a file called `Dockerfile`.

### Example: Create a Simple Web App Container

1. Create a new folder for your project:
```bash
mkdir my-first-docker-app
cd my-first-docker-app
```

2. Create a simple HTML file named `index.html`:
```html
<!DOCTYPE html>
<html>
<head>
    <title>My Docker Web App</title>
</head>
<body>
    <h1>Hello from my container!</h1>
    <p>This website is running in a Docker container that I built!</p>
</body>
</html>
```

3. Create a file named `Dockerfile` with these instructions:
```dockerfile
# Start with the nginx web server image
FROM nginx:alpine

# Copy our HTML file into the container
COPY index.html /usr/share/nginx/html/

# The container will run nginx automatically
```

4. Build your image:
```bash
docker build -t my-web-app .
```

**What you'll see:**
```
Sending build context to Docker daemon  3.072kB
Step 1/2 : FROM nginx:alpine
alpine: Pulling from library/nginx
...downloading progress...
Step 2/2 : COPY index.html /usr/share/nginx/html/
 ---> 12a3b4c5d6e7
Successfully built 12a3b4c5d6e7
Successfully tagged my-web-app:latest
```

5. Run your container:
```bash
docker run -d -p 8080:80 my-web-app
```

6. Visit http://localhost:8080 in your web browser to see your website!

## Sharing Data with Containers

Containers are normally isolated, like sealed boxes. But sometimes you want to share files between your computer and a container.

### Example: Share a Folder with a Container

1. Create a folder to share:
```bash
mkdir shared-folder
echo "This file is shared from my computer!" > shared-folder/hello.txt
```

2. Run a container that can access this folder:
```bash
docker run -it --name file-sharing -v $(pwd)/shared-folder:/shared-data ubuntu bash
```

3. Inside the container, look at the shared folder:
```bash
ls /shared-data
cat /shared-data/hello.txt
```

4. Create a new file from inside the container:
```bash
echo "This file was created inside the container!" > /shared-data/container-file.txt
exit
```

5. On your computer, check that the new file exists:
```bash
cat shared-folder/container-file.txt
```

## Connecting Containers Together

Containers can talk to each other, like toys that can interact. This is useful for building applications with multiple parts.

### Example: Connect a Web App to a Database

1. Create a network for the containers to communicate:
```bash
docker network create my-app-network
```

2. Start a MySQL database container:
```bash
docker run -d --name app-db \
  --network my-app-network \
  -e MYSQL_ROOT_PASSWORD=secretpassword \
  -e MYSQL_DATABASE=myapp \
  mysql:5.7
```

3. Start a WordPress container that connects to the database:
```bash
docker run -d --name app-web \
  --network my-app-network \
  -e WORDPRESS_DB_HOST=app-db \
  -e WORDPRESS_DB_USER=root \
  -e WORDPRESS_DB_PASSWORD=secretpassword \
  -e WORDPRESS_DB_NAME=myapp \
  -p 8080:80 \
  wordpress
```

4. Visit http://localhost:8080 in your web browser to see WordPress!

## What We've Learned

- **Containers** are lightweight packages that include everything a program needs to run
- **Docker** makes it easy to create, run, and share containers
- Containers help programs run the same way on any computer
- The basic Docker commands: `run`, `ps`, `stop`, `start`, `build`
- How to build your own container with a `Dockerfile`
- How to share files between your computer and containers
- How to connect multiple containers together

Now that you understand the basics, you can try creating more interesting containers for your own applications! 