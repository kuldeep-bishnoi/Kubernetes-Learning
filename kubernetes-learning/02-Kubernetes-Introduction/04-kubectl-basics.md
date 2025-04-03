# Kubectl: The Kubernetes Command-Line Tool

`kubectl` is the primary command-line interface for managing Kubernetes clusters. This tool allows you to deploy applications, inspect and manage cluster resources, and view logs.

## Installation

### macOS
```bash
# Using Homebrew
brew install kubectl

# Using curl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/darwin/amd64/kubectl"
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
```

### Linux
```bash
# Using apt
sudo apt-get update && sudo apt-get install -y kubectl

# Using curl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/kubectl
```

### Windows
```powershell
# Using Chocolatey
choco install kubernetes-cli

# Using curl
curl -LO "https://dl.k8s.io/release/v1.25.0/bin/windows/amd64/kubectl.exe"
```

Verify installation:
```bash
kubectl version --client
```

## Configuration

Kubectl uses a configuration file (kubeconfig) to communicate with the Kubernetes API server. By default, it looks for this file at `~/.kube/config`.

```bash
# View current configuration
kubectl config view

# Set the current context
kubectl config use-context my-cluster-name

# Get the current context
kubectl config current-context

# List all contexts
kubectl config get-contexts
```

## Basic Syntax

The basic syntax for kubectl commands is:

```
kubectl [command] [TYPE] [NAME] [flags]
```

- **command**: The operation to perform (create, get, apply, delete, etc.)
- **TYPE**: The resource type (pods, services, deployments, etc.)
- **NAME**: The name of the resource (optional for some commands)
- **flags**: Additional options specific to the command

## Common Operations with kubectl

### 1. Cluster Information

```bash
# View cluster information
kubectl cluster-info

# Check the health of cluster components
kubectl get componentstatuses

# Get detailed information about nodes
kubectl describe nodes

# Get node resource usage
kubectl top nodes
```

### 2. Working with Namespaces

```bash
# List all namespaces
kubectl get namespaces

# Create a new namespace
kubectl create namespace my-namespace

# Set a namespace for subsequent kubectl commands
kubectl config set-context --current --namespace=my-namespace

# Delete a namespace (and everything in it!)
kubectl delete namespace my-namespace
```

### 3. Managing Pods

```bash
# List all pods in the current namespace
kubectl get pods

# List all pods in all namespaces
kubectl get pods --all-namespaces

# Get detailed information about a pod
kubectl describe pod pod-name

# Delete a pod
kubectl delete pod pod-name

# Execute a command in a pod
kubectl exec -it pod-name -- /bin/bash

# Get pod logs
kubectl logs pod-name

# Get logs from a specific container in a multi-container pod
kubectl logs pod-name -c container-name

# Watch pod logs in real-time
kubectl logs -f pod-name
```

### 4. Deployments

```bash
# List all deployments
kubectl get deployments

# Create a deployment
kubectl create deployment nginx --image=nginx

# Scale a deployment
kubectl scale deployment nginx --replicas=5

# Update a deployment's image
kubectl set image deployment/nginx nginx=nginx:1.19

# View rollout status
kubectl rollout status deployment/nginx

# Rollback to a previous revision
kubectl rollout undo deployment/nginx

# View rollout history
kubectl rollout history deployment/nginx
```

### 5. Services

```bash
# List all services
kubectl get services

# Create a service to expose a deployment
kubectl expose deployment nginx --port=80 --type=LoadBalancer

# Get detailed information about a service
kubectl describe service nginx

# Delete a service
kubectl delete service nginx
```

### 6. ConfigMaps and Secrets

```bash
# Create a ConfigMap from a file
kubectl create configmap my-config --from-file=config.properties

# Create a ConfigMap from literal values
kubectl create configmap my-config --from-literal=key1=value1 --from-literal=key2=value2

# Create a Secret from literal values
kubectl create secret generic my-secret --from-literal=username=admin --from-literal=password=secret

# View ConfigMaps
kubectl get configmaps

# View Secrets
kubectl get secrets
```

### 7. Working with YAML Files

```bash
# Create resources from a YAML file
kubectl apply -f manifest.yaml

# Delete resources from a YAML file
kubectl delete -f manifest.yaml

# Edit a resource using the default editor
kubectl edit deployment nginx

# Create a YAML template for a resource
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > nginx-deployment.yaml
```

## Output Formatting

```bash
# Output in JSON format
kubectl get pods -o json

# Output in YAML format
kubectl get pods -o yaml

# Custom columns output
kubectl get pods -o custom-columns=NAME:.metadata.name,STATUS:.status.phase

# Output only resource names
kubectl get pods -o name

# Output in a table format with specific fields
kubectl get pods -o=custom-columns=NAME:.metadata.name,STATUS:.status.phase,NODE:.spec.nodeName
```

## Filtering and Sorting

```bash
# Filter pods by label
kubectl get pods -l app=nginx

# Filter pods by namespace
kubectl get pods --namespace=kube-system

# Sort output by a specific field
kubectl get pods --sort-by=.metadata.creationTimestamp
```

## Resource Monitoring

```bash
# Show resource usage for pods
kubectl top pods

# Show resource usage for nodes
kubectl top nodes

# Show detailed resource usage for a specific pod
kubectl top pod pod-name --containers
```

## Advanced Operations

### Port Forwarding

```bash
# Forward local port to a pod
kubectl port-forward pod/pod-name 8080:80

# Forward local port to a service
kubectl port-forward svc/service-name 8080:80
```

### Resource Quotas

```bash
# View resource quotas
kubectl get resourcequotas

# Describe a resource quota
kubectl describe resourcequota quota-name
```

### API Resources

```bash
# List all API resources
kubectl api-resources

# Get API versions
kubectl api-versions
```

## Practical Examples

### Scenario 1: Deploying a Simple Web Application

```bash
# Create a deployment
kubectl create deployment webapp --image=nginx

# Scale it to 3 replicas
kubectl scale deployment webapp --replicas=3

# Expose it as a service
kubectl expose deployment webapp --type=LoadBalancer --port=80

# Check if pods are running
kubectl get pods -l app=webapp -o wide

# Get the service URL
kubectl get service webapp
```

### Scenario 2: Debugging a Failed Pod

```bash
# Get information about the pod
kubectl describe pod failing-pod

# Check pod logs
kubectl logs failing-pod

# Check previous container logs if the container has restarted
kubectl logs failing-pod --previous

# Execute a command in the pod for debugging
kubectl exec -it failing-pod -- /bin/sh
```

### Scenario 3: Updating a Configuration

```bash
# Create a ConfigMap
kubectl create configmap app-config --from-literal=LOG_LEVEL=info

# Update the ConfigMap
kubectl edit configmap app-config

# Restart Deployment to pick up new config
kubectl rollout restart deployment webapp
```

## Working with Multiple Clusters

```bash
# Switch between clusters
kubectl config use-context cluster-name

# Run a command against a specific cluster
kubectl --context=cluster-name get pods
```

## Helpful Aliases and Bash Functions

Add these to your `.bashrc` or `.zshrc` file to make working with kubectl easier:

```bash
# Shorthand for kubectl
alias k=kubectl

# Get pods
alias kp='kubectl get pods'

# Get services
alias ks='kubectl get services'

# Get deployments
alias kd='kubectl get deployments'

# Watch pods
alias kpw='kubectl get pods -o wide --watch'

# Function to describe resource
kdesc() {
  kubectl describe "$1" "$2"
}

# Function to get logs
klog() {
  kubectl logs "$1"
}
```

## Tips for Working with kubectl

1. **Use kubectl explain**: Get documentation on resource fields
   ```bash
   kubectl explain pod.spec
   ```

2. **Use kubectl diff**: See what changes will be applied
   ```bash
   kubectl diff -f deployment.yaml
   ```

3. **Use kubectl dry-run**: Test commands without making changes
   ```bash
   kubectl create deployment test --image=nginx --dry-run=client -o yaml
   ```

4. **Use kubectl wait**: Wait for a specific condition
   ```bash
   kubectl wait --for=condition=Ready pods -l app=nginx
   ```

5. **Use jsonpath for complex queries**:
   ```bash
   kubectl get nodes -o jsonpath='{.items[*].status.addresses[?(@.type=="ExternalIP")].address}'
   ```

## Troubleshooting kubectl

### Common Issues

1. **Authentication problems**:
   ```bash
   # Check authentication
   kubectl auth can-i create pods
   ```

2. **Connection refused**:
   - Ensure the API server is running
   - Check if kubeconfig is correctly set up

3. **Resource not found**:
   - Check if you're in the correct namespace
   - Check if the resource exists

4. **Permission denied**:
   - Check RBAC permissions
   - Run `kubectl auth can-i` to verify permissions

## Further Reading

- [Official kubectl Reference](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands)
- [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
- [kubectl for Docker Users](https://kubernetes.io/docs/reference/kubectl/docker-cli-to-kubectl/) 