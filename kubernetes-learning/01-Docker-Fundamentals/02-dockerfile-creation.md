# Creating Your First Dockerfile

## What is a Dockerfile?

A Dockerfile is like a recipe for making a container. It tells Docker exactly what ingredients (software) to include and how to prepare them (configuration).

Think of it like instructions for building a toy:
1. Start with a base (like a toy car kit)
2. Add parts (install programs)
3. Put things in the right places (copy files)
4. Set up how it should run (configure settings)
5. Say what happens when someone plays with it (specify the command to run)

## Dockerfile Instructions Made Simple

Here are the most important instructions you'll use in your Dockerfiles:

| Instruction | What It Does | Example |
|-------------|--------------|---------|
| `FROM` | Chooses a starting point | `FROM python:3.9` |
| `WORKDIR` | Sets the working folder | `WORKDIR /app` |
| `COPY` | Copies files into the container | `COPY . .` |
| `RUN` | Runs a command during building | `RUN pip install requests` |
| `ENV` | Sets environment variables | `ENV DEBUG=true` |
| `EXPOSE` | Tells which ports to listen on | `EXPOSE 8080` |
| `CMD` | Sets the default command to run | `CMD ["python", "app.py"]` |

## Your First Dockerfile: Step by Step

Let's create a simple Python web application and package it in a container:

### Step 1: Create a Project Folder

```bash
mkdir my-python-app
cd my-python-app
```

### Step 2: Create a Simple Python App

Create a file called `app.py`:

```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def hello():
    return "Hello from my Docker container!"

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

### Step 3: Create a Requirements File

Create a file called `requirements.txt`:

```
flask==2.0.1
```

### Step 4: Write Your Dockerfile

Create a file named `Dockerfile` (no extension):

```dockerfile
# Start with an official Python image
FROM python:3.9-slim

# Set the working directory in the container
WORKDIR /app

# Copy requirements file and install dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy the rest of the application code
COPY . .

# Tell Docker the container listens on port 5000
EXPOSE 5000

# Command to run when the container starts
CMD ["python", "app.py"]
```

### Step 5: Build Your Docker Image

```bash
docker build -t my-python-app .
```

**What you'll see:**
```
Sending build context to Docker daemon  4.096kB
Step 1/6 : FROM python:3.9-slim
3.9-slim: Pulling from library/python
a2abf6c4d29d: Pull complete 
c7619af9ba49: Pull complete 
649e9e7dcaad: Pull complete 
be3c2facea0c: Pull complete 
7a8324bf8ded: Pull complete 
Digest: sha256:0a9627668b9666f0b346385dd7484ca05f52d4b1eb4b8d53d373c3cce9f8c01f
Status: Downloaded newer image for python:3.9-slim
 ---> 618f4c98d2d4
Step 2/6 : WORKDIR /app
 ---> Running in a95c09b6c777
Removing intermediate container a95c09b6c777
 ---> 3dff4b56b177
Step 3/6 : COPY requirements.txt .
 ---> 8de1a3c0589e
Step 4/6 : RUN pip install --no-cache-dir -r requirements.txt
 ---> Running in 77e6520a1bef
Collecting flask==2.0.1
  Downloading Flask-2.0.1-py3-none-any.whl (94 kB)
...more output...
Successfully installed Flask-2.0.1 Jinja2-3.0.1 MarkupSafe-2.0.1 Werkzeug-2.0.1 click-8.0.1 itsdangerous-2.0.1
Removing intermediate container 77e6520a1bef
 ---> c0d294894ded
Step 5/6 : COPY . .
 ---> 10b1fe9cf1db
Step 6/6 : EXPOSE 5000
 ---> Running in 56e7a68e3de6
Removing intermediate container 56e7a68e3de6
 ---> 36ca4a36b7bf
Step 7/6 : CMD ["python", "app.py"]
 ---> Running in fb39c022afde
Removing intermediate container fb39c022afde
 ---> 05b0af294285
Successfully built 05b0af294285
Successfully tagged my-python-app:latest
```

### Step 6: Run Your Container

```bash
docker run -p 5000:5000 my-python-app
```

**What you'll see:**
```
 * Serving Flask app 'app' (lazy loading)
 * Environment: production
   WARNING: This is a development server. Do not use it in a production deployment.
   Use a production WSGI server instead.
 * Debug mode: off
 * Running on all addresses.
   WARNING: This is a development server. Do not use it in a production deployment.
 * Running on http://172.17.0.2:5000/ (Press CTRL+C to quit)
 * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
```

Now open a web browser and visit: http://localhost:5000 - you'll see "Hello from my Docker container!"

## Building Different Types of Applications

### JavaScript (Node.js) Application

#### File: package.json
```json
{
  "name": "node-app",
  "version": "1.0.0",
  "main": "index.js",
  "scripts": {
    "start": "node index.js"
  },
  "dependencies": {
    "express": "^4.17.1"
  }
}
```

#### File: index.js
```javascript
const express = require('express');
const app = express();
const port = 3000;

app.get('/', (req, res) => {
  res.send('Hello from Node.js in Docker!');
});

app.listen(port, () => {
  console.log(`App listening at http://localhost:${port}`);
});
```

#### File: Dockerfile
```dockerfile
FROM node:14-alpine

WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .

EXPOSE 3000

CMD ["npm", "start"]
```

#### Build and Run:
```bash
docker build -t my-node-app .
docker run -p 3000:3000 my-node-app
```

Visit http://localhost:3000 in your browser!

### HTML/Static Website

#### File: index.html
```html
<!DOCTYPE html>
<html>
<head>
    <title>Docker Static Website</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 40px; }
        h1 { color: #0066cc; }
    </style>
</head>
<body>
    <h1>Hello from NGINX in Docker!</h1>
    <p>This is a static website served from a Docker container.</p>
</body>
</html>
```

#### File: Dockerfile
```dockerfile
FROM nginx:alpine

COPY index.html /usr/share/nginx/html/

EXPOSE 80

# NGINX starts automatically, so no CMD needed
```

#### Build and Run:
```bash
docker build -t my-static-site .
docker run -p 8080:80 my-static-site
```

Visit http://localhost:8080 in your browser!

## Understanding Dockerfile Instructions

### FROM - The Starting Point

Every Dockerfile starts with a base image - like starting with a partially built toy:

```dockerfile
FROM ubuntu:20.04         # Start with Ubuntu Linux
FROM python:3.9           # Start with Python already installed
FROM node:14-alpine       # Start with Node.js on a tiny Alpine Linux
```

**Best Practice:** Choose the smallest base image that meets your needs. Alpine-based images are usually smaller.

### WORKDIR - Setting Your Work Folder

```dockerfile
WORKDIR /app
```

This creates a folder called `/app` in the container and makes it the working directory for subsequent instructions.

**Best Practice:** Always use WORKDIR instead of using lots of `cd` commands.

### COPY and ADD - Putting Files in the Container

```dockerfile
# Copy a single file
COPY index.html /app/

# Copy multiple files
COPY *.py /app/

# Copy a folder and its contents
COPY ./src /app/src

# ADD also lets you copy from URLs or extract archives
ADD https://example.com/file.txt /app/
ADD archive.tar.gz /app/
```

**Best Practice:** Use COPY unless you specifically need ADD's features (downloading or extracting).

### RUN - Executing Commands During Build

```dockerfile
# Install packages
RUN apt-get update && apt-get install -y curl

# Install dependencies
RUN pip install -r requirements.txt

# Create folders
RUN mkdir -p /app/data
```

**Best Practice:** Combine related RUN commands with `&&` to create fewer layers.

### ENV - Setting Environment Variables

```dockerfile
# Set one variable
ENV PORT=8080

# Set multiple variables
ENV NODE_ENV=production \
    DEBUG=false \
    LOGGING=1
```

**Best Practice:** Use ENV for configuration that might change between environments.

### EXPOSE - Declaring Ports

```dockerfile
# Tell Docker this container uses port 80
EXPOSE 80

# Expose multiple ports
EXPOSE 8080 8081
```

**Best Practice:** EXPOSE doesn't actually open ports - it's just documentation. You still need `-p` when running.

### CMD and ENTRYPOINT - Starting Your Application

```dockerfile
# Run a Python script
CMD ["python", "app.py"]

# Run a shell command
CMD ["sh", "-c", "echo 'Hello' && python app.py"]
```

**Best Practice:** Use the JSON array format `CMD ["executable", "param1", "param2"]` instead of shell form.

## Best Practices for Writing Dockerfiles

### 1. Keep Images Small

```dockerfile
# BAD: Installs lots of extra packages
RUN apt-get update && apt-get install -y build-essential python python-dev

# GOOD: Only installs what you need and cleans up
RUN apt-get update && \
    apt-get install -y --no-install-recommends python-minimal && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
```

### 2. Use .dockerignore

Create a `.dockerignore` file to prevent copying unnecessary files:

```
# Don't copy these into the image
node_modules
npm-debug.log
Dockerfile
.git
.env
```

### 3. Order Instructions by Changeability

Put things that change less often at the top of your Dockerfile:

```dockerfile
# 1. Base image - rarely changes
FROM node:14-alpine

# 2. Global dependencies - change occasionally
RUN npm install -g some-tool

# 3. Project dependencies - change more often 
COPY package.json .
RUN npm install

# 4. Application code - changes all the time
COPY . .
```

This improves build caching - Docker can reuse previous steps if they haven't changed.

### 4. Use Multi-Stage Builds for Smaller Images

```dockerfile
# Build stage
FROM node:14 AS build
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Production stage
FROM nginx:alpine
COPY --from=build /app/build /usr/share/nginx/html
EXPOSE 80
# NGINX starts automatically
```

This keeps build tools out of the final image, making it much smaller.

## Practical Exercise: Dockerizing a Web Application

Let's create a complete web application with a proper Dockerfile:

### Step 1: Set Up the Project

```bash
mkdir docker-web-app
cd docker-web-app
```

### Step 2: Create Application Files

#### File: app.py
```python
from flask import Flask, render_template
import os

app = Flask(__name__)

@app.route('/')
def home():
    return render_template('index.html', 
                          app_name="Docker Demo App",
                          environment=os.environ.get('APP_ENV', 'development'))

if __name__ == '__main__':
    port = int(os.environ.get('PORT', 5000))
    app.run(host='0.0.0.0', port=port)
```

#### File: requirements.txt
```
flask==2.0.1
```

#### Create a templates folder and index.html:
```bash
mkdir templates
```

#### File: templates/index.html
```html
<!DOCTYPE html>
<html>
<head>
    <title>{{ app_name }}</title>
    <style>
        body { 
            font-family: Arial, sans-serif; 
            margin: 40px; 
            background-color: #f5f5f5;
        }
        .container {
            background-color: white;
            border-radius: 10px;
            padding: 20px;
            box-shadow: 0 2px 4px rgba(0,0,0,0.1);
            max-width: 800px;
            margin: 0 auto;
        }
        h1 { color: #0066cc; }
    </style>
</head>
<body>
    <div class="container">
        <h1>{{ app_name }}</h1>
        <p>This app is running in <strong>{{ environment }}</strong> mode.</p>
        <p>It was built with Flask and is running inside a Docker container!</p>
    </div>
</body>
</html>
```

### Step 3: Create the Dockerfile

#### File: Dockerfile
```dockerfile
# Start with Python
FROM python:3.9-slim

# Set environment variables
ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    APP_ENV=production

# Set working directory
WORKDIR /app

# Install dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY . .

# Expose port
EXPOSE 5000

# Command to run the application
CMD ["python", "app.py"]
```

### Step 4: Create a .dockerignore File

#### File: .dockerignore
```
__pycache__
*.pyc
*.pyo
*.pyd
.Python
.env
.git
.gitignore
Dockerfile
README.md
```

### Step 5: Build and Run

```bash
docker build -t web-app .
docker run -p 5000:5000 -e APP_ENV=production web-app
```

### Step 6: Visit Your Application

Open http://localhost:5000 in your browser. You should see your web application running!

## Pushing Your Image to Docker Hub

To share your container with others, you can push it to Docker Hub (Docker's public registry):

### Step 1: Create a Docker Hub Account

Go to [Docker Hub](https://hub.docker.com/) and create a free account if you don't have one.

### Step 2: Log in to Docker Hub from Terminal

```bash
docker login
```

Enter your Docker Hub username and password.

### Step 3: Tag Your Image

```bash
docker tag web-app yourusername/web-app:v1.0
```

Replace `yourusername` with your actual Docker Hub username.

### Step 4: Push the Image

```bash
docker push yourusername/web-app:v1.0
```

**What you'll see:**
```
The push refers to repository [docker.io/yourusername/web-app]
a3b90d87111a: Pushed 
941d3c56e4cc: Pushed 
5e1e59279b86: Pushed 
73477885d19a: Mounted from library/python 
...
v1.0: digest: sha256:f3b1a5bc6a5364f563fbb53005b7af2e2522785d079c17a656e5b98d570357a6 size: 1995
```

### Step 5: Use Your Image Anywhere

Now anyone can use your image by running:

```bash
docker run -p 5000:5000 yourusername/web-app:v1.0
```

## What We've Learned

- Dockerfiles are recipes for building containers
- The basic Dockerfile instructions: FROM, WORKDIR, COPY, RUN, ENV, EXPOSE, CMD
- How to create Dockerfiles for different types of applications
- Best practices for writing efficient Dockerfiles
- How to build and run containers from your Dockerfiles
- How to share your container images with others through Docker Hub

Now you can containerize almost any application by writing your own Dockerfile! 