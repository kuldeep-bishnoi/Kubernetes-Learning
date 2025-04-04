# Docker Compose: Multi-Container Applications

This document provides a detailed explanation of Docker Compose - an essential tool for defining and running multi-container Docker applications.

## Introduction to Docker Compose

### What is Docker Compose?

Docker Compose is a tool that allows you to define and manage multi-container Docker applications. With Compose, you use a YAML file to configure your application's services, networks, and volumes, then create and start all the services with a single command.

### Why Use Docker Compose?

1. **Simplifies Container Management**
   - Define your entire application stack in a single file
   - Start, stop, and rebuild services with one command
   - Simplify complex container relationships

2. **Reproducible Environments**
   - Guarantees the same environment every time
   - Source control your application configuration
   - Share development environments easily

3. **Development to Production Workflow**
   - Use the same application definition across environments
   - Local development environment mirrors production
   - Easy testing and staging setups

## Docker Compose File Structure

The `docker-compose.yml` file is the heart of Docker Compose. It defines everything about your application.

### Basic Structure

```yaml
version: '3.8'  # Compose file version

services:       # Your application's containers
  web:
    image: nginx:alpine
    ports:
      - "8080:80"
    volumes:
      - ./website:/usr/share/nginx/html
  
  database:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: example
      MYSQL_DATABASE: app_data

networks:       # Custom networks if needed
  backend:
    driver: bridge

volumes:        # Named volumes for persistence
  db_data:
```

### Key Sections

1. **version**: Specifies which Compose file format version to use
2. **services**: Defines the containers to be created
3. **networks**: Custom networks for container communication
4. **volumes**: Persistent storage for containers

## Key Concepts in Docker Compose

### Services

Services are the containers that make up your application. Each service is defined as a separate entry under the `services` section:

```yaml
services:
  web:
    build: ./web  # Build from Dockerfile in ./web directory
    image: myapp/web:latest  # Alternatively, use a pre-built image
    depends_on:
      - database  # This service depends on the database service
    ports:
      - "8000:8000"  # Port mapping: host:container
    environment:
      - DATABASE_URL=postgres://postgres:password@database:5432/db
```

### Build Configuration

You can define how to build container images in your Compose file:

```yaml
services:
  app:
    build:
      context: ./dir  # Build context (directory containing Dockerfile)
      dockerfile: Dockerfile.dev  # Alternative Dockerfile name
      args:  # Build arguments
        BUILD_ENV: development
```

### Networks

Networks enable communication between containers:

```yaml
services:
  web:
    networks:
      - frontend
      - backend
  
  database:
    networks:
      - backend

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
    internal: true  # Internal network (no access to the outside world)
```

### Volumes

Volumes provide persistent storage and data sharing:

```yaml
services:
  database:
    image: postgres:13
    volumes:
      - db_data:/var/lib/postgresql/data  # Named volume
      - ./init-scripts:/docker-entrypoint-initdb.d  # Bind mount

volumes:
  db_data:  # Named volume definition
```

### Environment Variables

```yaml
services:
  web:
    environment:  # Direct specification
      - NODE_ENV=development
      - DEBUG=true
    
  api:
    env_file: .env  # From .env file
```

## Advanced Docker Compose Features

### Dependencies and Startup Order

```yaml
services:
  web:
    depends_on:
      - database
      - redis
```

Note: `depends_on` only waits for containers to start, not for services inside to be ready.

### Healthchecks

```yaml
services:
  database:
    image: postgres:13
    healthcheck:
      test: ["CMD", "pg_isready", "-U", "postgres"]
      interval: 10s
      timeout: 5s
      retries: 5
```

### Scaling Services

```yaml
# Start multiple instances of a service
docker-compose up --scale worker=3
```

### Extending Compose Files

Base `docker-compose.yml`:
```yaml
services:
  web:
    image: nginx:alpine
    ports:
      - "80:80"
```

Production override `docker-compose.prod.yml`:
```yaml
services:
  web:
    restart: always
    environment:
      - NODE_ENV=production
```

Run with:
```bash
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up
```

## Real-World Examples

### Web Application with Database and Cache

```yaml
version: '3.8'

services:
  web:
    build: ./web
    ports:
      - "3000:3000"
    depends_on:
      - api
    environment:
      - API_URL=http://api:4000

  api:
    build: ./api
    ports:
      - "4000:4000"
    depends_on:
      - database
      - redis
    environment:
      - DATABASE_URL=postgres://postgres:password@database:5432/app
      - REDIS_URL=redis://redis:6379

  database:
    image: postgres:13
    volumes:
      - db_data:/var/lib/postgresql/data
    environment:
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=app

  redis:
    image: redis:6-alpine
    volumes:
      - redis_data:/data

volumes:
  db_data:
  redis_data:
```

### Microservices Architecture

```yaml
version: '3.8'

services:
  gateway:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf
    depends_on:
      - service-a
      - service-b
      - service-c

  service-a:
    build: ./service-a
    environment:
      - DATABASE_URL=mongodb://mongo-a:27017/service-a
    depends_on:
      - mongo-a

  service-b:
    build: ./service-b
    environment:
      - DATABASE_URL=postgres://postgres:password@postgres-b:5432/service-b
    depends_on:
      - postgres-b

  service-c:
    build: ./service-c
    environment:
      - REDIS_URL=redis://redis:6379
    depends_on:
      - redis

  mongo-a:
    image: mongo:4
    volumes:
      - mongo_a_data:/data/db

  postgres-b:
    image: postgres:13
    volumes:
      - postgres_b_data:/var/lib/postgresql/data
    environment:
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=service-b

  redis:
    image: redis:6-alpine
    volumes:
      - redis_data:/data

volumes:
  mongo_a_data:
  postgres_b_data:
  redis_data:
```

## Docker Compose Commands

### Basic Commands

```bash
# Start all services defined in docker-compose.yml
docker-compose up

# Start in detached mode (background)
docker-compose up -d

# Stop all services
docker-compose down

# Stop and remove volumes
docker-compose down -v

# View running containers
docker-compose ps

# View logs
docker-compose logs

# Follow logs
docker-compose logs -f

# Rebuild services
docker-compose build

# Pull latest images
docker-compose pull
```

### Service Management

```bash
# Start specific services
docker-compose up web database

# Restart a service
docker-compose restart web

# Stop a specific service
docker-compose stop database

# Run a command in a service container
docker-compose exec web npm test

# Run a one-off command in a new container
docker-compose run --rm web npm install
```

## Docker Compose vs. Kubernetes

| Feature | Docker Compose | Kubernetes |
|---------|----------------|------------|
| Scope | Single-host deployment | Multi-host cluster |
| Complexity | Simple, easy to learn | Complex, steep learning curve |
| Scaling | Limited scaling capabilities | Advanced auto-scaling |
| Self-healing | Limited | Automatic pod recovery |
| Load Balancing | Basic | Advanced with services and ingress |
| Rolling Updates | Basic | Sophisticated strategies |
| State Management | Simple volume management | StatefulSets and PVCs |
| Best For | Development, small projects | Production, large applications |

Docker Compose is often the starting point before moving to Kubernetes for production deployments.

## Best Practices

1. **Use Specific Versions**
   - Tag images with specific versions (not `latest`)
   - Specify compose file version explicitly

2. **Environment Variables**
   - Use `.env` files for configuration
   - Don't commit sensitive data like passwords
   - Consider using Docker secrets for production

3. **Networking**
   - Create separate networks for frontend and backend
   - Use internal networks for databases

4. **Volumes**
   - Use named volumes for persistent data
   - Document volume usage in comments

5. **Development vs. Production**
   - Use compose file overrides for environment-specific settings
   - Have development, staging, and production configurations

6. **Resource Limits**
   - Set memory and CPU limits
   ```yaml
   services:
     app:
       deploy:
         resources:
           limits:
             cpus: '0.5'
             memory: 512M
   ```

7. **Health Checks**
   - Define health checks for critical services
   - Helps with dependency management

## Common Interview Questions

1. **Q: What is the difference between `docker run` and `docker-compose up`?**
   A: `docker run` starts a single container from an image. `docker-compose up` starts multiple containers defined in a docker-compose.yml file, setting up networks and volumes automatically.

2. **Q: How do you share data between containers in Docker Compose?**
   A: You can use named volumes, bind mounts, or define a shared volume that multiple services use.

3. **Q: Can Docker Compose be used in production?**
   A: Yes, but it has limitations for high-availability, scaling, and multi-host deployments. For production at scale, Kubernetes or Docker Swarm are often better choices.

4. **Q: How does Docker Compose handle container dependencies?**
   A: With the `depends_on` parameter, which ensures containers start in the correct order. However, it only waits for containers to start, not for services to be ready.

5. **Q: What is the difference between version 2 and version 3 of Compose files?**
   A: Version 3 added support for Docker Swarm and removed some version 2 features like `volume_driver`. Version 3 also has a different networking model and adds deploy configuration.

6. **Q: How can you scale services with Docker Compose?**
   A: Using the `--scale` flag with `docker-compose up`, e.g., `docker-compose up --scale worker=3`.

7. **Q: How can you override configuration for different environments?**
   A: By using multiple compose files with `-f` flag, e.g., `docker-compose -f docker-compose.yml -f docker-compose.prod.yml up`.

8. **Q: What happens to data when you run `docker-compose down`?**
   A: Containers and networks are removed, but volumes persist by default unless you use the `-v` flag to remove them.

## Resources for Further Learning

- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [Docker Compose File Reference](https://docs.docker.com/compose/compose-file/)
- [Docker Compose in Production](https://docs.docker.com/compose/production/)
- [Docker Compose CLI Reference](https://docs.docker.com/compose/reference/) 