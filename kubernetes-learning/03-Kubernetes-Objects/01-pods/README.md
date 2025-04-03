# Kubernetes Pods

## 🌟 Overview

Pods are the smallest deployable units in Kubernetes. A Pod represents a single instance of a running process in your cluster and can contain one or more containers. This guide covers everything you need to know about working with Pods in Kubernetes.

## 📋 What is a Pod?

A **Pod** is the smallest deployable unit in Kubernetes. It represents a single instance of a running process in your cluster.

Pods encapsulate:
- One or more application containers
- Storage resources
- A unique network IP
- Options that govern how the container(s) should run

Think of a Pod as a logical host for containers that are tightly coupled and need to share resources.

## Single-Container vs Multi-Container Pods

### Single-Container Pods

The most common use case is a single container per Pod:

```
┌─────────── Pod ───────────┐
│                           │
│  ┌─────── Container ────┐ │
│  │                      │ │
│  │     Application      │ │
│  │                      │ │
│  └──────────────────────┘ │
│                           │
└───────────────────────────┘
```

### Multi-Container Pods

Sometimes, you need multiple containers in a single Pod when containers are tightly coupled:

```
┌───────────────── Pod ─────────────────┐
│                                       │
│  ┌─────────────┐    ┌─────────────┐  │
│  │  Container  │    │  Container  │  │
│  │             │    │             │  │
│  │  Main App   │    │  Sidecar    │  │
│  │             │    │             │  │
│  └─────────────┘    └─────────────┘  │
│                                       │
└───────────────────────────────────────┘
```

Multi-container Pods share:
- Network namespace (same IP and port space)
- Storage volumes
- Same lifecycle (created and terminated together)

Common patterns for multi-container Pods include:
- **Sidecar**: Enhances the main container (e.g., logging agent)
- **Ambassador**: Proxies network connections (e.g., connection to external database)
- **Adapter**: Standardizes and normalizes output (e.g., transforming logs)

## 🔄 Pod Lifecycle

Pods follow a defined lifecycle, tracked through the `phase` field:

```
   ┌─────────┐
   │ Pending │
   └────┬────┘
        │
        ▼
   ┌─────────┐      ┌──────────┐
   │ Running │─────▶│ Succeeded │
   └────┬────┘      └──────────┘
        │
        ▼
   ┌─────────┐
   │ Failed  │
   └─────────┘
```

1. **Pending**: The Pod has been accepted by the cluster but one or more containers are not yet running. This includes time spent being scheduled and downloading images.

2. **Running**: The Pod has been bound to a node, and all containers have been created. At least one container is running, or is in the process of starting or restarting.

3. **Succeeded**: All containers in the Pod have terminated successfully and will not be restarted.

4. **Failed**: All containers in the Pod have terminated, and at least one container has terminated in failure (exited with non-zero status or was terminated by the system).

5. **Unknown**: The state of the Pod could not be determined.

### Pod Conditions

In addition to phases, Pods have a set of conditions that provide more detailed information:

- **PodScheduled**: The Pod has been scheduled to a node
- **ContainersReady**: All containers in the Pod are ready
- **Initialized**: All init containers have completed successfully
- **Ready**: The Pod is able to serve requests and should be added to load balancing pools

## 📝 Creating Pods

### Using YAML (Declarative)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
    environment: dev
spec:
  containers:
  - name: nginx
    image: nginx:1.19
    ports:
    - containerPort: 80
    resources:
      limits:
        memory: "128Mi"
        cpu: "500m"
      requests:
        memory: "64Mi"
        cpu: "250m"
```

Save this as `nginx-pod.yaml` and apply:

```bash
kubectl apply -f nginx-pod.yaml
```

### Using kubectl (Imperative)

```bash
kubectl run nginx-pod --image=nginx:1.19 --port=80
```

## 🛠️ Managing Pods

### Getting Information

```bash
# List all pods
kubectl get pods

# List pods with more details
kubectl get pods -o wide

# Describe a specific pod
kubectl describe pod nginx-pod

# Get YAML representation of a pod
kubectl get pod nginx-pod -o yaml
```

### Accessing Pods

```bash
# Get logs for a pod
kubectl logs nginx-pod

# For multi-container pods, specify the container
kubectl logs nginx-pod -c container-name

# Stream logs in real-time
kubectl logs -f nginx-pod

# Execute commands in a pod
kubectl exec -it nginx-pod -- /bin/bash

# For multi-container pods, specify the container
kubectl exec -it nginx-pod -c container-name -- /bin/bash
```

### Port Forwarding

To access a pod directly from your local machine:

```bash
kubectl port-forward nginx-pod 8080:80
```

Now you can access the pod at `http://localhost:8080`

### Deleting Pods

```bash
# Delete a specific pod
kubectl delete pod nginx-pod

# Delete using the YAML file
kubectl delete -f nginx-pod.yaml

# Delete all pods in the default namespace
kubectl delete pods --all
```

## 🧰 Advanced Pod Configuration

### Environment Variables

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: env-pod
spec:
  containers:
  - name: env-container
    image: busybox
    command: ["sh", "-c", "echo $ENVIRONMENT_VAR && sleep 3600"]
    env:
    - name: ENVIRONMENT_VAR
      value: "Hello from Kubernetes!"
```

### Resource Requests and Limits

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: resource-pod
spec:
  containers:
  - name: app
    image: nginx
    resources:
      requests:
        memory: "64Mi"   # Minimum memory needed
        cpu: "250m"      # 250 millicores = 1/4 CPU core
      limits:
        memory: "128Mi"  # Maximum memory allowed
        cpu: "500m"      # 500 millicores = 1/2 CPU core
```

### Volume Mounts

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: volume-pod
spec:
  containers:
  - name: web
    image: nginx
    volumeMounts:
    - name: html-volume
      mountPath: /usr/share/nginx/html
  volumes:
  - name: html-volume
    emptyDir: {}  # Simple ephemeral volume
```

### Init Containers

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: init-pod
spec:
  initContainers:
  - name: init-service
    image: busybox
    command: ['sh', '-c', 'until nslookup myservice; do echo waiting for myservice; sleep 2; done;']
  containers:
  - name: app
    image: nginx
```

### Pod Security Context

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: security-context-pod
spec:
  securityContext:
    runAsUser: 1000  # Run as non-root user
    fsGroup: 2000    # Set file permissions for volumes
  containers:
  - name: secure-container
    image: nginx
    securityContext:
      allowPrivilegeEscalation: false  # Prevent privilege escalation
```

### Probes (Health Checks)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: probes-pod
spec:
  containers:
  - name: app
    image: nginx
    livenessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 15
      timeoutSeconds: 2
      periodSeconds: 5
      failureThreshold: 3
    readinessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 5
```

## 📝 Pod Design Patterns

### Sidecar Pattern

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sidecar-pod
spec:
  containers:
  - name: main-app
    image: nginx
    volumeMounts:
    - name: shared-logs
      mountPath: /var/log/nginx
  - name: log-sidecar
    image: busybox
    command: ["sh", "-c", "tail -f /var/log/nginx/access.log"]
    volumeMounts:
    - name: shared-logs
      mountPath: /var/log/nginx
  volumes:
  - name: shared-logs
    emptyDir: {}
```

### Ambassador Pattern

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ambassador-pod
spec:
  containers:
  - name: main-app
    image: app-image
    env:
    - name: REDIS_HOST
      value: "localhost:6379"  # Connect to local ambassador
  - name: redis-ambassador
    image: redis-proxy
    ports:
    - containerPort: 6379
```

### Adapter Pattern

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: adapter-pod
spec:
  containers:
  - name: main-app
    image: app-image
    # Produces custom metrics/logs format
  - name: adapter
    image: metrics-adapter
    # Transforms metrics/logs to standardized format
```

## 🚫 Common Issues with Pods

### Pod Stuck in Pending

Possible causes:
- Insufficient cluster resources
- PersistentVolumeClaim not bound
- Node selector/affinity constraints can't be satisfied

### Pod Stuck in ContainerCreating

Possible causes:
- Image pull issues (wrong name, registry authentication)
- Volume mount problems
- Network issues

### Pod in CrashLoopBackOff

Possible causes:
- Application errors
- Resource constraints (OOMKilled)
- Misconfiguration (wrong command, args)

### Debugging Tips

```bash
# Check pod status with reason
kubectl get pod nginx-pod -o wide

# Get detailed description
kubectl describe pod nginx-pod

# Check logs - especially for crashing containers
kubectl logs nginx-pod --previous

# Check events in the namespace
kubectl get events --sort-by='.lastTimestamp'
```

## 🏆 Best Practices for Pods

1. **Keep pods small and focused** - One main process per container
2. **Use appropriate resource limits** to prevent one pod from consuming all resources
3. **Implement health checks** with liveness and readiness probes
4. **Don't use pods directly** - Use Deployments, StatefulSets, or DaemonSets instead
5. **Use labels effectively** for organization and selection
6. **Use annotations** for non-identifying metadata
7. **Set appropriate termination grace periods** for clean shutdowns
8. **Never use hostPort** unless absolutely necessary
9. **Use init containers** for setup and validation tasks
10. **Consider using pod security contexts** to enhance security

## 🧪 Practical Exercises

### Exercise 1: Create a Simple Pod

Create a pod running the `nginx` image and verify it's running.

### Exercise 2: Multi-Container Pod

Create a pod with two containers that share a volume.

### Exercise 3: Pod with Health Checks

Create a pod with both liveness and readiness probes.

### Exercise 4: Debugging a Broken Pod

Troubleshoot and fix a pod that's failing to start.

See the [exercises](./exercises) directory for detailed instructions and solutions.

## 🔄 Next Steps

After understanding Pods, continue to [ReplicaSets](../02-replicasets/) to learn how to manage multiple instances of your pods.

## 📚 Additional Resources

- [Pods Official Documentation](https://kubernetes.io/docs/concepts/workloads/pods/)
- [Pod Lifecycle](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/)
- [Init Containers](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/)
- [Pod Design Patterns](https://kubernetes.io/blog/2016/06/container-design-patterns/) 