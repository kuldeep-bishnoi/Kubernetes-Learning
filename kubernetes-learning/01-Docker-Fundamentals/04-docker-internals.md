# Docker Internals: How Docker Works Under the Hood

This document explains the technical internals of Docker and container technology - essential knowledge for technical interviews and advanced understanding.

## The Magic Behind Containers

Have you ever wondered how Docker actually makes containers work? Let's explore the "magic" behind Docker with simple explanations and examples.

Imagine Docker is like a toy box with special compartments. Each compartment can hold a different toy, and the toys can't mix with each other. This is basically what Docker does with computer programs!

## The Two Main Powers of Containers

Docker uses two special "superpowers" from Linux to make containers work:

1. **Namespaces** - For making walls between containers
2. **Control Groups (cgroups)** - For sharing resources fairly

Let's explore both of these superpowers!

## Namespaces: Creating Separate Spaces

### What Are Namespaces?

Namespaces are like invisible walls that separate containers. They make each container think it's the only one running on the computer.

Imagine you have three kids playing in one room, but you set up dividers so each child thinks they have their own room. That's what namespaces do!

### The Different Types of Namespaces

Docker uses several different namespaces, each one separating a different type of resource:

| Namespace | What It Separates | Like Giving Each Child... |
|-----------|-------------------|---------------------------|
| PID | Processes | Their own set of toys |
| Network | Network connections | Their own pretend phone |
| Mount | File systems | Their own bookshelf |
| UTS | Hostname | Their own name tag |
| IPC | Communication channels | Their own walkie-talkie |
| User | User IDs | Their own ID card |

### Let's See Namespaces in Action

Try running two containers and see how they're separated:

```bash
# Run a container and give it a name
docker run -d --name container1 nginx

# Run another container
docker run -d --name container2 nginx

# Check the process IDs in container1
docker exec container1 ps aux
```

**What you'll see:**
```
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.1  10628  5604 ?        Ss   00:15   0:00 nginx: master process nginx -g daemon off;
nginx       32  0.0  0.0  11088  2584 ?        S    00:15   0:00 nginx: worker process
```

```bash
# Check the process IDs in container2
docker exec container2 ps aux
```

**What you'll see:**
```
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.1  10628  5604 ?        Ss   00:16   0:00 nginx: master process nginx -g daemon off;
nginx       32  0.0  0.0  11088  2584 ?        S    00:16   0:00 nginx: worker process
```

Notice how both containers show process IDs starting at 1? That's because they each have their own PID namespace!

### The PID Namespace: Process Isolation

The PID namespace is like giving each child their own set of numbered toys. Even though they all have a toy #1, they're different toys.

On your computer, the same process might have different IDs:

```bash
# Find the "real" process ID on your host computer
docker inspect --format '{{.State.Pid}}' container1
```

This might show something like `1234`, which is the actual process ID on your computer!

### The Network Namespace: Network Isolation

Each container gets its own network setup:
- Its own IP address
- Its own network interfaces
- Its own routing rules

Let's see this in action:

```bash
# Check the IP address of container1
docker exec container1 ip addr
```

**What you'll see:**
```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
65: eth0@if66: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever
```

```bash
# Check the IP address of container2
docker exec container2 ip addr
```

You'll see a different IP address (probably `172.17.0.3`). Each container gets its own network!

### The Mount Namespace: Filesystem Isolation

Each container sees its own files and folders. It's like giving each child their own toy box where they can only see their own toys.

```bash
# Create a file in container1
docker exec container1 sh -c "echo 'Hello from container 1' > /test.txt"

# Try to see it from container2
docker exec container2 cat /test.txt
```

**What you'll see:**
```
cat: /test.txt: No such file or directory
```

Each container has its own separate files!

## Control Groups (cgroups): Sharing Resources Fairly

### What Are Control Groups?

Control groups (cgroups) make sure containers share computer resources fairly. They're like giving each child a specific amount of toys, snacks, or play time.

Cgroups control:
- How much CPU a container can use
- How much memory it can use
- How much disk space it can use
- How much network bandwidth it can use

### Setting Resource Limits for a Container

You can tell Docker exactly how much resources to give each container:

```bash
# Run a container with memory limits
docker run -d --name limited-container --memory="100m" --cpus="0.5" nginx
```

This container can only use:
- 100 megabytes of memory
- Half of one CPU core

Let's see what happens if a container tries to use too much memory:

```bash
# Start a container that will try to use too much memory
docker run --memory="100m" --name memory-test alpine sh -c "cat /dev/zero | head -c 200m > /dev/null"
```

The container will likely be killed by Docker because it tried to use more memory than allowed.

## Union Filesystem: Layer Cake Storage

### How Docker Images Work: Layers

Docker images are like layer cakes. Each instruction in a Dockerfile adds a new layer.

Imagine building a sandwich:
1. First bread slice (base layer)
2. Add cheese (another layer)
3. Add lettuce (another layer)
4. Add tomato (another layer)
5. Top bread slice (final layer)

Docker images work the same way! Let's see the layers in an image:

```bash
docker history nginx:alpine
```

**What you'll see:**
```
IMAGE          CREATED       CREATED BY                                      SIZE
c7e69fe2ebd1   2 weeks ago   /bin/sh -c #(nop)  CMD ["nginx" "-g" "daemon…   0B
<missing>      2 weeks ago   /bin/sh -c #(nop)  STOPSIGNAL SIGQUIT           0B
<missing>      2 weeks ago   /bin/sh -c #(nop)  EXPOSE 80                    0B
<missing>      2 weeks ago   /bin/sh -c #(nop)  ENTRYPOINT ["/docker-entr…   0B
<missing>      2 weeks ago   /bin/sh -c #(nop) COPY file:9e3b3b9756d5896a…   4.61kB
<missing>      2 weeks ago   /bin/sh -c #(nop) COPY file:0fd5fca330dcd6a7…   1.04kB
<missing>      2 weeks ago   /bin/sh -c #(nop) COPY file:0b866ff3fc1ef5b0…   1.96kB
<missing>      2 weeks ago   /bin/sh -c #(nop) COPY file:65504f71f5855ca0…   1.2kB
...
```

Each line is a layer in the image!

### The Copy-on-Write System

When a container runs, it gets a new writable layer on top of the read-only image layers. If a container modifies a file, it gets copied to the writable layer.

It's like giving each child a clear plastic sheet to draw on top of a picture book. The original book stays clean, and each child's drawings are separate!

Let's see this in action:

```bash
# Start two containers from the same image
docker run -d --name container-a nginx:alpine
docker run -d --name container-b nginx:alpine

# Modify a file in container-a
docker exec container-a sh -c "echo 'Hello from A' > /usr/share/nginx/html/index.html"

# The file is changed in container-a
docker exec container-a cat /usr/share/nginx/html/index.html
```

**What you'll see:**
```
Hello from A
```

```bash
# But container-b still has the original file
docker exec container-b cat /usr/share/nginx/html/index.html
```

**What you'll see:**
```
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...
```

This is copy-on-write in action!

## How Docker Commands Actually Work

When you run a Docker command, several different programs work together:

### The Docker Team:

1. **Docker Client**: The tool you use when typing `docker` commands
2. **Docker Daemon (dockerd)**: The main program that manages everything
3. **containerd**: The manager for container lifecycles
4. **runc**: The program that actually creates containers

Let's see how they work together:

### What Happens When You Run a Container

When you type `docker run nginx`, here's what happens:

1. The Docker Client sends your command to the Docker Daemon
2. The Docker Daemon asks containerd to create a container
3. containerd uses runc to set up the container
4. runc creates the namespaces and cgroups
5. runc starts the container's process
6. The container is now running!

It's like a factory assembly line where each program has a specific job to do.

## Container Networking Made Simple

### Default Bridge Network

By default, Docker creates a virtual network bridge called `docker0`. It's like a virtual network switch that all containers can plug into.

Let's look at the default network:

```bash
docker network ls
```

**What you'll see:**
```
NETWORK ID     NAME      DRIVER    SCOPE
0123456789ab   bridge    bridge    local
0987654321ba   host      host      local
abcdef123456   none      null      local
```

The "bridge" network is the default. Let's see the containers connected to it:

```bash
docker network inspect bridge
```

You'll see details about the network and the containers connected to it.

### Creating Custom Networks

You can create your own networks to group containers:

```bash
# Create a custom network
docker network create my-network

# Run containers in the custom network
docker run -d --name database --network my-network mysql:5.7 --environment MYSQL_ROOT_PASSWORD=secret

docker run -d --name web-app --network my-network nginx
```

Containers on the same network can talk to each other using their names as hostnames:

```bash
# Connect to the web-app container
docker exec -it web-app sh

# Ping the database container by name
ping database
```

It works like magic because Docker sets up DNS for the containers!

## Exploring Docker Security

Docker uses several security features to keep containers isolated:

### Capabilities: Fine-Grained Permissions

Linux capabilities are like special permissions for specific actions. Docker drops many capabilities by default for security.

Let's see what happens if we try to change the system time in a container:

```bash
# Try to change time in a regular container
docker run --rm ubuntu date -s "1 JAN 2020"
```

**What you'll see:**
```
date: cannot set date: Operation not permitted
Wed Jan  1 00:00:00 UTC 2020
```

You need special permission to do this. Let's add it:

```bash
# Run with the CAP_SYS_TIME capability
docker run --rm --cap-add SYS_TIME ubuntu date -s "1 JAN 2020"
```

Now it works!

### Seccomp: Filtering System Calls

Seccomp filters which system calls a container can make. It's like a bouncer that only lets certain people into a club.

Docker uses a default seccomp profile that blocks dangerous system calls.

## Testing Container Isolation

Let's do a practical test to see how container isolation works:

```bash
# Create a file on the host
echo "This is the host file" > host-file.txt

# Try to mount this file into two containers
docker run -v $(pwd)/host-file.txt:/shared-file.txt --name reader1 -d nginx
docker run -v $(pwd)/host-file.txt:/shared-file.txt --name reader2 -d nginx

# Look at the file in both containers
docker exec reader1 cat /shared-file.txt
docker exec reader2 cat /shared-file.txt

# Modify the file from reader1
docker exec reader1 sh -c "echo 'Modified by reader1' > /shared-file.txt"

# Now check both containers
docker exec reader1 cat /shared-file.txt
docker exec reader2 cat /shared-file.txt

# And check the host file
cat host-file.txt
```

All three (host and two containers) will show the same content after modification. This is because volumes connect the container's filesystem to the host's filesystem.

## Practical Debugging Skills

When working with Docker, you'll sometimes need to debug issues. Here are some useful commands:

```bash
# Check container logs
docker logs container-name

# Get detailed container info
docker inspect container-name

# See running processes in a container
docker top container-name

# Get container resource usage
docker stats container-name

# Execute a command in a running container
docker exec -it container-name sh
```

## Fun Experiments to Try

1. **Race for Resources**: Run multiple containers with CPU limits and see how they share resources
   ```bash
   docker run -d --cpus=".2" --name slow busybox md5sum /dev/urandom
   docker run -d --cpus=".8" --name fast busybox md5sum /dev/urandom
   docker stats
   ```

2. **Container Escape Room**: See what containers can and cannot access
   ```bash
   # Try to see host processes from inside a container
   docker run --rm ubuntu ps aux
   
   # Compare with host processes
   ps aux
   ```

3. **Container Filesystem Explorer**: Explore the layers
   ```bash
   docker run -it --name explorer alpine sh
   # Inside the container:
   touch /new-file.txt
   exit
   
   # Commit changes to a new image
   docker commit explorer explorer-image
   
   # See the new layer
   docker history explorer-image
   ```

## What We've Learned

- Docker uses **namespaces** to isolate containers
- **Control groups** limit resource usage
- **Union filesystems** create efficient layered storage
- The Docker **architecture** has multiple components working together
- Containers provide **isolation** but can share resources when needed
- Docker has built-in **security features** to protect containers

Now that you understand how Docker works under the hood, you can use it more effectively for your applications!

## Container Technology Fundamentals

### Linux Kernel Features That Enable Containers

Docker isn't a virtualization technology. Instead, it leverages Linux kernel features to create isolated environments called containers. The two primary Linux kernel technologies enabling containerization are:

1. **Namespaces** - For isolation
2. **Control Groups (cgroups)** - For resource limitation

## Linux Namespaces

Namespaces partition kernel resources so that one set of processes sees one set of resources while another set of processes sees a different set of resources. Docker uses the following namespaces:

| Namespace | Purpose | What It Isolates |
|-----------|---------|------------------|
| PID | Process isolation | Process IDs |
| Network | Network isolation | Network interfaces, IP routing tables, firewall rules, etc. |
| Mount | Filesystem isolation | Mount points |
| UTS | Hostname isolation | Hostname and domain name |
| IPC | IPC isolation | Inter-process communication resources, message queues |
| User | User isolation | User and group IDs |
| Cgroup | Cgroup isolation | Cgroup root directory |

### PID Namespace

The PID namespace provides process isolation:

- Each container has its own process numbering system starting at PID 1
- Processes in one container cannot see or affect processes in another container or the host
- The container's PID 1 is critical (like in Linux systems) and handles orphaned processes

Example of how processes appear differently inside and outside a container:

```
# On host, container process might be 12345
$ ps aux | grep nginx

# Inside container, same process appears as PID 1
$ docker exec container_name ps aux
```

### Network Namespace

Network namespaces virtualize the network stack:

- Each container gets its own network interfaces, IP addresses, routing tables, and firewall rules
- Containers can communicate with each other through virtual Ethernet devices (veth pairs)
- Docker creates a bridge network (`docker0`) by default that connects containers
- Port mappings work by configuring NAT rules using iptables

### Mount Namespace

Mount namespaces isolate the filesystem view:

- Each container has its own root filesystem
- Changes to the filesystem in one container don't affect others
- Volumes and bind mounts work by creating controlled access points across namespace boundaries

### User Namespace

User namespaces map users and groups between the container and host:

- A process can have different user IDs inside and outside a container
- Allows containers to run as root internally while mapping to non-privileged users on the host
- Enhances security by limiting the impact of container escapes

## Control Groups (cgroups)

While namespaces provide isolation, cgroups control resource allocation and limits:

### Resource Constraints

Cgroups allow Docker to:

- Limit CPU usage (shares, quotas, or specific cores)
- Restrict memory usage (RAM and swap)
- Control disk I/O priority and throughput
- Limit network bandwidth

### How Docker Uses cgroups

When you run a container with resource constraints:

```bash
docker run --memory=512m --cpus=0.5 nginx
```

Docker translates these parameters into cgroup settings:

- Creates cgroup entries under `/sys/fs/cgroup/`
- Configures limits in the appropriate cgroup subsystems
- Places container processes into these cgroups

### cgroup Subsystems

| Subsystem | Purpose |
|-----------|---------|
| cpu | Controls CPU shares and scheduling |
| memory | Controls memory and swap limits |
| blkio | Controls I/O operations on block devices |
| devices | Controls access to devices |
| freezer | Suspends/resumes processes |
| pids | Limits number of processes |
| net_cls, net_prio | Tags network packets from containers |

## Union Filesystem and Layered Storage

Docker images and containers use a layered filesystem approach:

### How Layers Work

- Docker images consist of multiple read-only layers
- Each instruction in a Dockerfile creates a new layer
- Layers contain only the differences from the previous layer
- Multiple containers can share the same underlying image layers
- Each container gets its own thin writable layer on top of the image

### Copy-on-Write (CoW)

- When a container modifies a file from the image, it's copied to the container's writable layer
- Original image layers remain unchanged
- Multiple containers can share the same base image without duplication
- Provides significant storage efficiency

### Storage Drivers

Docker supports multiple storage drivers that implement the layered filesystem approach:

| Driver | Description | Best For |
|--------|-------------|----------|
| overlay2 | Modern and efficient implementation using OverlayFS | Most Linux distributions, recommended |
| devicemapper | Uses device mapper thin provisioning | Older kernels where overlay2 isn't available |
| btrfs | Uses BTRFS filesystem | Systems with BTRFS as the backing filesystem |
| zfs | Uses ZFS filesystem | Systems with ZFS support |
| aufs | Original storage driver | Legacy systems (deprecated) |

## Docker Runtime and containerd

Modern Docker uses a layered architecture:

### Docker Engine Components

1. **dockerd (Docker daemon)**: 
   - Exposes the Docker API
   - Manages Docker objects
   - Communicates with containerd

2. **containerd**:
   - Container runtime that manages container lifecycle
   - Pulls/pushes images, manages storage
   - Invokes lower-level runc to create containers
   - Acts as the supervisor for container processes

3. **runc**:
   - Low-level container runtime based on OCI specs
   - Sets up namespaces, cgroups, and mounts
   - Spawns and runs container processes

4. **shim**:
   - Keeps container running if containerd or dockerd restarts
   - Reports exit status to containerd
   - Maintains STDIN/STDOUT streams

### Runtime Execution Flow

When you run `docker run`:

1. Docker client sends command to dockerd
2. dockerd instructs containerd to create container
3. containerd uses runc to create and start the container
4. runc configures namespaces, cgroups, and mounts
5. runc starts the container process
6. runc exits, leaving the container running
7. containerd-shim monitors the container process

## Container Networking Internals

Docker creates several network types with different implementations:

### Bridge Networking

The default network mode:

1. Docker creates a virtual bridge (`docker0`)
2. Each container gets a virtual Ethernet (veth) interface
3. One end of veth pair connects to container's network namespace
4. Other end connects to the bridge
5. NAT rules using iptables allow containers to access external networks
6. Port publishing works through iptables DNAT rules

Example of how Docker sets up bridge networking:

```
# Bridge interface on host
$ ip link show docker0

# veth pairs (virtual Ethernet devices)
$ ip link | grep veth

# NAT rules for container connectivity
$ iptables -t nat -L -n
```

### Host Networking

- Container shares host's network namespace
- No network isolation
- Container binds directly to host ports
- Higher performance but less isolation

### Overlay Networking

- Spans multiple Docker hosts
- Uses VXLAN encapsulation for cross-host communication
- Managed key-value store for network configuration
- Enables container communication across hosts

## Container Security

Understanding Docker's security model is critical:

### Security Boundaries and Concerns

- Containers share the host kernel (unlike VMs)
- Container escape vulnerabilities can affect the host
- Root in container vs. root on host (without user namespaces)
- Container processes can potentially access host resources

### Security Features

1. **Capabilities**: Fine-grained privileges instead of full root
   - Docker drops many capabilities by default
   - Can further restrict with `--cap-drop` or grant with `--cap-add`

2. **Seccomp**: System call filtering
   - Docker uses a default seccomp profile
   - Blocks dangerous syscalls
   - Can be customized for specific needs

3. **AppArmor/SELinux**: Mandatory Access Control
   - Confines container processes
   - Limits what files containers can access
   - Restricts capabilities even further

4. **User namespaces**:
   - Maps container UIDs to non-privileged host UIDs
   - Reduces impact of container escapes
   - Must be explicitly enabled

5. **Read-only containers**:
   - `--read-only` flag makes container filesystem immutable
   - Combined with specific writable volumes for required directories

## Debugging Container Issues

How to inspect and debug Docker's internal operations:

### Namespace Inspection

```bash
# List process namespaces
ls -la /proc/<pid>/ns/

# Enter a container's namespaces
nsenter -t <pid> -n ip addr
```

### cgroup Inspection

```bash
# View container's cgroups
cat /proc/<pid>/cgroup

# Examine memory limits
cat /sys/fs/cgroup/memory/docker/<container_id>/memory.limit_in_bytes
```

### Low-level Runtime Commands

```bash
# Directly run container with runc
runc run <container_id>

# List containers known to containerd
ctr containers ls
```

## Common Interview Questions and Answers

1. **Q: How does Docker achieve isolation without a hypervisor?**
   A: Docker uses Linux namespaces to isolate container processes, networking, and filesystems, while cgroups limit resource usage. Unlike VMs, containers share the host kernel but have their own isolated view of the system.

2. **Q: What happens when I run `docker run`?**
   A: The Docker client sends the command to dockerd, which asks containerd to create a container. containerd uses runc to set up namespaces, cgroups, and mount the container filesystem. runc then executes the container process within these isolated namespaces.

3. **Q: How does Docker implement resource limits?**
   A: Docker uses cgroups to control CPU, memory, and I/O resources. When you specify limits like `--memory=512m`, Docker configures the appropriate cgroup parameters to enforce these constraints.

4. **Q: How does Docker's layered filesystem work?**
   A: Docker uses union filesystems (like overlay2) to stack image layers. Each layer only contains the differences from the previous layer. When a container runs, it gets a thin writable layer on top of the read-only image layers, implementing copy-on-write for efficiency.

5. **Q: What's the difference between Docker's bridge, host, and overlay networks?**
   A: Bridge networking creates an isolated network on the host with NAT for external connectivity. Host networking uses the host's network stack directly with no isolation. Overlay networking spans multiple hosts, allowing containers on different hosts to communicate.

6. **Q: How does Docker handle inter-container communication?**
   A: Containers on the same network can communicate via their IP addresses or DNS names. Docker provides built-in DNS resolution so containers can find each other by name. Custom networks provide better isolation and automated DNS resolution.

7. **Q: What security features does Docker provide?**
   A: Docker provides security through capabilities (limited privileges), seccomp filters (restricted syscalls), AppArmor/SELinux profiles (access control), user namespaces (UID mapping), and read-only containers.

8. **Q: How does port mapping work in Docker?**
   A: Docker uses iptables DNAT rules to forward traffic from the host's port to the container's port. The Docker daemon configures these rules when you use the `-p` flag.

## Advanced Docker Concepts and Best Practices

### Optimizing Docker Images

1. **Multi-stage builds**:
   - Use one stage to build and another for runtime
   - Significantly reduces final image size
   - Eliminates build tools from runtime image

2. **Layer optimization**:
   - Combine commands to reduce layer count
   - Order instructions by change frequency
   - Use .dockerignore to exclude unnecessary files

3. **Base image selection**:
   - Use minimal images like Alpine or distroless
   - Consider security implications of base images
   - Balance size vs. functionality needs

### Container Orchestration Concepts

Understanding how Docker concepts map to Kubernetes:

| Docker Concept | Kubernetes Equivalent |
|----------------|------------------------|
| Container | Container within Pod |
| docker run | Pod/Deployment |
| Docker Compose | Pod/Deployment with multiple containers |
| Docker network | Service, NetworkPolicy |
| Docker volume | PersistentVolume, PersistentVolumeClaim |
| Docker Swarm service | Deployment, StatefulSet |
| Docker health check | Liveness/Readiness probes |

## Resources for Further Learning

- [Linux Namespaces Documentation](https://man7.org/linux/man-pages/man7/namespaces.7.html)
- [Control Groups Documentation](https://www.kernel.org/doc/Documentation/cgroup-v2.txt)
- [OCI Runtime Specification](https://github.com/opencontainers/runtime-spec)
- [Docker Security Documentation](https://docs.docker.com/engine/security/) 