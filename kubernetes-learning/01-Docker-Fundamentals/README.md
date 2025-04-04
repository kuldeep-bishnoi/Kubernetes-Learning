# Docker Fundamentals

## Overview

Docker is a platform that enables developers to build, ship, and run applications in containers. This section covers the fundamental concepts of Docker, which is a prerequisite for understanding Kubernetes.

## Topics Covered

1. [**Docker Basics**](00-theory.md)
   - What are containers?
   - Containers vs. Virtual Machines
   - Docker architecture
   - Docker workflow

2. [**Hands-on Practice**](01-hands-on.md)
   - Running containers
   - Container lifecycle
   - Working with images
   - Exploring running containers

3. [**Dockerfile Creation**](02-dockerfile-creation.md)
   - Dockerfile syntax and best practices
   - Building images
   - Tagging and pushing images
   - Running containers

4. [**Multi-Stage Builds**](03-multi-stage-builds.md)
   - Optimizing Docker images
   - Reducing image size
   - Improving security
   - Best practices for production

5. [**Docker Internals**](04-docker-internals.md)
   - Linux namespaces
   - Control groups (cgroups)
   - Union filesystems
   - Container runtime architecture
   - OCI specifications

6. [**Docker Compose**](05-docker-compose.md)
   - Multi-container applications
   - Defining services, networks, and volumes
   - Development to production workflows
   - Scaling and extending services

7. [**Docker Security**](06-docker-security.md)
   - Security model and architecture
   - Container isolation mechanisms
   - Best practices for secure containers
   - Vulnerability scanning
   - Runtime protection

## Visualizing Docker Architecture

```
┌─────────────────────────────────────────────────────┐
│                     Docker Host                      │
│                                                     │
│    ┌─────────┐     ┌─────────┐     ┌─────────┐     │
│    │Container│     │Container│     │Container│     │
│    │    1    │     │    2    │     │    3    │     │
│    └─────────┘     └─────────┘     └─────────┘     │
│                                                     │
│    ┌─────────────────────────────────────────┐     │
│    │              Docker Engine               │     │
│    └─────────────────────────────────────────┘     │
│                                                     │
│    ┌─────────────────────────────────────────┐     │
│    │               Host OS                    │     │
│    └─────────────────────────────────────────┘     │
│                                                     │
│    ┌─────────────────────────────────────────┐     │
│    │              Infrastructure              │     │
│    └─────────────────────────────────────────┘     │
└─────────────────────────────────────────────────────┘
```

## Containers vs Virtual Machines

| Feature | Containers | Virtual Machines |
|---------|------------|------------------|
| Size | Lightweight (MBs) | Heavy (GBs) |
| Boot Time | Seconds | Minutes |
| Performance | Near-native | Virtualized |
| OS Kernel | Shared with host | Separate kernel |
| Isolation | Process-level | Hardware-level |
| Resource Usage | Efficient | Higher overhead |
| Portability | Highly portable | Less portable |

## Practical Skills You'll Learn

- Create, run, and manage Docker containers
- Write efficient Dockerfiles for your applications
- Optimize Docker images using multi-stage builds
- Understand container networking and storage
- Use Docker Compose for multi-container applications
- Follow best practices for container security
- Troubleshoot common Docker issues
- Understand the underlying technologies (namespaces, cgroups)
- Implement secure container workflows

## Quick Start

Before diving into each topic, ensure you have Docker installed on your system:

```bash
# Verify Docker installation
docker --version

# Run a simple container
docker run hello-world
```

## Interview Preparation

The Docker fundamentals section has been structured to prepare you for technical interviews by providing:

- Comprehensive explanations of core concepts
- Deep dives into underlying technology
- Best practices for real-world implementation
- Common interview questions and answers
- Practical examples that demonstrate key concepts

## Why Docker is Essential for Kubernetes

Docker provides the containerization technology that Kubernetes orchestrates. Understanding Docker concepts like images, containers, networking, and storage is crucial before diving into Kubernetes, as these are the building blocks that Kubernetes manages at scale.

## Learning Path

1. Start with the [theory section](00-theory.md) to understand core concepts
2. Follow the [hands-on exercises](01-hands-on.md) to gain practical experience
3. Learn [Dockerfile creation](02-dockerfile-creation.md) and [multi-stage builds](03-multi-stage-builds.md) for optimized images
4. Explore [Docker internals](04-docker-internals.md) to understand how containers work under the hood
5. Master [Docker Compose](05-docker-compose.md) for multi-container applications
6. Implement [Docker security](06-docker-security.md) best practices
7. Proceed to Kubernetes once you're comfortable with Docker fundamentals

## Additional Resources

- [Official Docker Documentation](https://docs.docker.com/)
- [Docker Hub](https://hub.docker.com/) - Public repository of Docker images
- [OCI Specifications](https://github.com/opencontainers/runtime-spec) - Container standards
- [Play with Docker](https://labs.play-with-docker.com/) - Free Docker playground 