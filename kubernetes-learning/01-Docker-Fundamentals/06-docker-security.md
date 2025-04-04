# Docker Security Fundamentals

This document provides a comprehensive overview of Docker security concepts, best practices, and common vulnerabilities - essential knowledge for creating secure containerized environments.

## Container Security Fundamentals

### Container Security vs. Virtual Machine Security

| Aspect | Containers | Virtual Machines |
|--------|------------|------------------|
| Isolation | Process-level isolation | Hardware-level isolation |
| Kernel | Shared with host | Separate kernel |
| Security Boundary | Thinner (container escape can affect host) | Stronger isolation boundary |
| Attack Surface | Potential access to host kernel | Limited to VM |
| Resource Overhead | Minimal | Significant |

### Docker's Security Model

Docker's security relies on multiple Linux kernel features working together:

1. **Namespaces**: Isolation of processes, network, mounts, etc.
2. **Control Groups (cgroups)**: Resource limitations and accounting
3. **Capabilities**: Fine-grained privilege control
4. **Seccomp**: System call filtering
5. **AppArmor/SELinux**: Mandatory access control
6. **User Namespaces**: UID/GID isolation and mapping

## Security Risks in Container Environments

### Container Escape Vulnerabilities

Container escape occurs when a process inside a container gains access to the host system or other containers. Causes include:

1. Misconfigured privileges or capabilities
2. Kernel vulnerabilities
3. Mounted sensitive host directories
4. Container runtime vulnerabilities
5. Privileged containers

### Common Attack Vectors

1. **Excessive Privileges**: Running containers as root or with privileged flag
2. **Vulnerable Images**: Outdated packages with known CVEs
3. **Secrets Management**: Hardcoded credentials in images
4. **Host Resource Access**: Mounting sensitive host paths
5. **Network Exposure**: Unintended network exposure
6. **Insecure Configuration**: Default configurations that are not production-ready

## Docker Security Best Practices

### 1. Use Official and Verified Images

```bash
# Use official images when possible
docker pull nginx:alpine

# Scan images for vulnerabilities
docker scan nginx:alpine
```

### 2. Keep Images Updated

```bash
# Regular updates for security patches
docker pull nginx:alpine
docker build --no-cache -t myapp .
```

### 3. Minimize Image Size

- Use minimal base images (Alpine, distroless)
- Implement multi-stage builds
- Remove unnecessary tools and packages

```dockerfile
# Multi-stage build example
FROM node:16 AS build
WORKDIR /app
COPY . .
RUN npm ci && npm run build

FROM node:16-alpine
COPY --from=build /app/dist /app
USER node
CMD ["node", "/app/index.js"]
```

### 4. Run Containers as Non-Root

```dockerfile
# Create a non-root user
RUN addgroup -g 1000 appuser && \
    adduser -u 1000 -G appuser -s /bin/sh -D appuser

# Set ownership
RUN chown -R appuser:appuser /app

# Switch to non-root user
USER appuser

# Start application
CMD ["node", "app.js"]
```

### 5. Use Docker Content Trust

Docker Content Trust ensures image integrity by verifying digital signatures:

```bash
# Enable Docker Content Trust
export DOCKER_CONTENT_TRUST=1

# Sign and push image
docker trust sign myregistry/myimage:latest

# View trust data
docker trust inspect --pretty myregistry/myimage:latest
```

### 6. Limit Container Resources

```bash
# Set memory and CPU limits
docker run -d --name app \
  --memory="500m" \
  --memory-swap="1g" \
  --cpus="0.5" \
  nginx
```

### 7. Use Read-Only Containers

```bash
# Make container filesystem read-only
docker run --read-only \
  --tmpfs /tmp \
  --tmpfs /var/run \
  nginx
```

### 8. Implement Proper Network Segmentation

```bash
# Create isolated networks
docker network create backend
docker network create frontend

# Connect containers to appropriate networks
docker run --network=backend --name db postgres:13
docker run --network=frontend --network=backend --name app myapp
```

### 9. Use Security Scanning Tools

```bash
# Scan for vulnerabilities with Docker Scout
docker scout cves nginx:alpine

# Alternative tools: Trivy, Clair, Anchore
```

### 10. Implement Secrets Management

```bash
# Create Docker secrets
docker secret create db_password password.txt

# Use secrets in containers
docker service create \
  --name app \
  --secret db_password \
  myapp
```

## Linux Security Features in Docker

### 1. Capabilities

Docker drops most capabilities by default, allowing for principle of least privilege:

```bash
# Grant specific capabilities
docker run --cap-add NET_ADMIN nginx

# Drop specific capabilities
docker run --cap-drop ALL --cap-add NET_BIND_SERVICE nginx

# List all capabilities
docker run --rm alpine capsh --print
```

Common capabilities:

| Capability | Description |
|------------|-------------|
| NET_ADMIN | Configure networks, firewalls |
| SYS_ADMIN | Perform system administration tasks |
| SYS_PTRACE | Debug processes |
| CHOWN | Change file ownership |
| SETUID/SETGID | Change process UID/GID |
| NET_BIND_SERVICE | Bind to privileged ports (<1024) |

### 2. Seccomp (Secure Computing Mode)

Seccomp filters available system calls:

```bash
# Run with custom seccomp profile
docker run --security-opt seccomp=/path/to/seccomp.json nginx
```

Example seccomp profile structure:
```json
{
  "defaultAction": "SCMP_ACT_ERRNO",
  "architectures": ["SCMP_ARCH_X86_64"],
  "syscalls": [
    {
      "name": "accept",
      "action": "SCMP_ACT_ALLOW"
    },
    {
      "name": "bind",
      "action": "SCMP_ACT_ALLOW"
    }
    // Additional allowed syscalls...
  ]
}
```

### 3. AppArmor / SELinux

Mandatory Access Control systems:

```bash
# Run with custom AppArmor profile
docker run --security-opt apparmor=custom-profile nginx

# Run with SELinux options
docker run --security-opt label=type:container_t nginx
```

### 4. User Namespaces

Map container user IDs to non-privileged host IDs:

```bash
# Configure Docker daemon to use user namespaces
# /etc/docker/daemon.json
{
  "userns-remap": "default"
}

# Restart Docker daemon
systemctl restart docker
```

## Container Image Security

### 1. Vulnerability Scanning

Tools for scanning Docker images:

1. **Docker Scout**: Built into Docker Desktop
   ```bash
   docker scout cves myapp:latest
   ```

2. **Trivy**: Open-source scanner
   ```bash
   trivy image myapp:latest
   ```

3. **Anchore**: Enterprise container security platform
4. **Clair**: Open-source project for static analysis of vulnerabilities

### 2. Image Signing and Verification

```bash
# Sign an image (Docker Content Trust)
export DOCKER_CONTENT_TRUST=1
docker push myregistry/myapp:latest

# Verify signature
docker trust inspect --pretty myregistry/myapp:latest
```

### 3. Base Image Selection Guidelines

1. Choose minimal images (Alpine, distroless)
2. Use specific tags, not `latest`
3. Prefer official images over community images
4. Consider using scratch for static binaries

```dockerfile
# Using distroless for Java applications
FROM gcr.io/distroless/java:11
COPY target/*.jar /app.jar
CMD ["/app.jar"]

# For Go applications, use scratch
FROM scratch
COPY go-binary /app
CMD ["/app"]
```

## Runtime Security Monitoring

### 1. Logging and Monitoring

```bash
# Enable Docker logging
docker run --log-driver=syslog nginx

# Monitor container events
docker events

# View container logs
docker logs --follow container_name
```

### 2. Audit Logging

Configure Docker daemon to create audit logs:

```json
// /etc/docker/daemon.json
{
  "log-driver": "syslog",
  "log-opts": {
    "syslog-address": "udp://syslog-server:514"
  }
}
```

### 3. Runtime Security Tools

- **Falco**: Runtime security monitoring
- **Docker Bench for Security**: Security best practices check
- **Aqua Security / Sysdig Secure**: Commercial runtime security

## Security in Orchestrated Environments

### Kubernetes-Specific Security Concerns

1. **Pod Security**: Use Pod Security Standards (PSS)
2. **RBAC**: Implement Role-Based Access Control
3. **Network Policies**: Define network segmentation
4. **Security Contexts**: Apply security settings at pod/container level
5. **Secrets Management**: Use Kubernetes Secrets properly

## Practical Security Examples

### Secure Dockerfile Example

```dockerfile
# Multi-stage build for a Node.js application
FROM node:16 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Production image
FROM node:16-alpine
# Install security updates
RUN apk update && apk upgrade

# Create non-root user
RUN addgroup -g 1000 nodeapp && \
    adduser -u 1000 -G nodeapp -s /bin/sh -D nodeapp

# Set up application
WORKDIR /app
COPY --from=builder --chown=nodeapp:nodeapp /app/dist /app
COPY --from=builder --chown=nodeapp:nodeapp /app/node_modules /app/node_modules
COPY --from=builder --chown=nodeapp:nodeapp /app/package.json /app

# Use non-root user
USER nodeapp

# Expose only required port
EXPOSE 3000

# Define healthcheck
HEALTHCHECK --interval=30s --timeout=3s \
  CMD wget -qO- http://localhost:3000/health || exit 1

# Start application
CMD ["node", "index.js"]
```

### Running a Secure Container

```bash
docker run \
  --name secure-app \
  --read-only \
  --tmpfs /tmp \
  --cap-drop ALL \
  --cap-add NET_BIND_SERVICE \
  --security-opt no-new-privileges \
  --security-opt seccomp=/etc/docker/seccomp-profiles/default.json \
  --health-cmd "curl -f http://localhost:3000/health || exit 1" \
  --health-interval 30s \
  --memory="500m" \
  --cpus="0.5" \
  -p 3000:3000 \
  my-secure-app:latest
```

## Common Interview Questions on Docker Security

1. **Q: What's the difference between user namespaces and the USER instruction in a Dockerfile?**
   A: The USER instruction in a Dockerfile simply changes the user within the container. User namespaces map UIDs/GIDs between the container and host, providing stronger isolation. Even if a container runs as root internally, with user namespaces it can map to a non-privileged user on the host.

2. **Q: What is a "privileged" container and why is it dangerous?**
   A: A privileged container runs with nearly all the capabilities of the host machine, essentially bypassing the security isolation that containers normally provide. It's dangerous because it can access host devices, modify kernel parameters, and potentially escape container boundaries.

3. **Q: How would you securely handle secrets in Docker containers?**
   A: Best practices include: use Docker secrets or Kubernetes secrets, mount temporary filesystems, use environment variables (cautiously), leverage a secrets management tool (HashiCorp Vault, AWS Secrets Manager), and never bake secrets into images.

4. **Q: What's the security significance of using immutable containers?**
   A: Immutable containers (read-only) enhance security by preventing runtime changes to the filesystem, reducing the attack surface for malware, and enforcing infrastructure-as-code practices. Any changes must go through the CI/CD pipeline, improving auditability.

5. **Q: How do Docker capabilities relate to Linux capabilities?**
   A: Docker capabilities are a subset of Linux kernel capabilities. Docker drops non-essential capabilities by default to reduce the attack surface. Specific capabilities can be added or dropped to fine-tune container privileges without requiring full root access.

6. **Q: How can you prevent a container from accessing sensitive kernel parameters?**
   A: Use seccomp profiles to restrict system calls, drop the SYS_ADMIN capability, use AppArmor/SELinux profiles, disable privileged mode, and use the --security-opt no-new-privileges flag to prevent privilege escalation.

7. **Q: What are common Docker image vulnerabilities and how do you mitigate them?**
   A: Common vulnerabilities include outdated packages, unnecessary tools, exposed secrets, and large attack surfaces. Mitigate by regularly scanning images, using minimal base images, implementing multi-stage builds, and keeping images updated with security patches.

8. **Q: How do you implement defense in depth for Docker environments?**
   A: Implement multiple security layers: secure host configuration, minimal base images, non-root users, capability restrictions, read-only filesystems, network segmentation, regular scanning, runtime monitoring, and secure CI/CD practices.

## Resources for Further Learning

- [Docker Security Documentation](https://docs.docker.com/engine/security/)
- [CIS Docker Benchmark](https://www.cisecurity.org/benchmark/docker)
- [NIST Container Security Guide](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-190.pdf)
- [Docker Bench Security](https://github.com/docker/docker-bench-security)
- [OWASP Docker Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Docker_Security_Cheat_Sheet.html) 