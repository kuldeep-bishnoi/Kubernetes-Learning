# Kubernetes DaemonSets

DaemonSets ensure that a copy of a Pod runs on all (or some) nodes in a cluster. As nodes are added to the cluster, Pods are automatically added to them. As nodes are removed, those Pods are garbage collected.

![DaemonSet Diagram](https://raw.githubusercontent.com/kubernetes/website/main/content/en/examples/controllers/daemonset.svg)

## What is a DaemonSet?

A DaemonSet is a Kubernetes controller that ensures that a copy of a Pod runs on all or a subset of nodes in a cluster. DaemonSets are used for running cluster services that need to be present on every node, such as:

- Node monitoring agents
- Log collectors
- Storage daemons
- Network plugins
- Security agents

## Key Features of DaemonSets

- **Node Coverage**: Runs a Pod on all or selected nodes
- **Automatic Pod Scheduling**: Adds Pods to new nodes as they join the cluster
- **Automatic Garbage Collection**: Removes Pods when nodes are removed
- **Node Affinity**: Can target specific nodes using node selectors or affinity
- **Update Strategies**: Supports rolling updates similar to Deployments

## When to Use DaemonSets

DaemonSets are ideal for:

- **Infrastructure Services**: Running node-level services like monitoring agents
- **Logging**: Collecting logs from each node
- **Storage Provision**: Setting up storage drivers on each node
- **Networking**: Running network proxies or plugins on each node
- **Security**: Running security agents or scanners

## DaemonSet vs. Other Controllers

| Feature | DaemonSet | Deployment | StatefulSet |
|---------|-----------|------------|-------------|
| Purpose | Run Pods on every node | Run stateless applications | Run stateful applications |
| Pod Identity | Based on node | Random | Stable network identity |
| Pod Placement | One per node | Any node | Ordered placement |
| Scaling | Automatic with nodes | Manual or auto-scaled | Manual |
| Use Case | Node-level services | Stateless apps | Stateful apps |

## DaemonSet Specification

A basic DaemonSet YAML looks like this:

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-elasticsearch
  namespace: kube-system
  labels:
    k8s-app: fluentd-logging
spec:
  selector:
    matchLabels:
      name: fluentd-elasticsearch
  template:
    metadata:
      labels:
        name: fluentd-elasticsearch
    spec:
      tolerations:
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
      containers:
      - name: fluentd-elasticsearch
        image: quay.io/fluentd_elasticsearch/fluentd:v2.5.2
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
        volumeMounts:
        - name: varlog
          mountPath: /var/log
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
```

### Key Fields in DaemonSet Spec

- **selector**: Labels that match the Pod template
- **template**: Pod template definition for the DaemonSet
- **updateStrategy**: Defines how to update the Pods (RollingUpdate or OnDelete)
- **minReadySeconds**: Minimum time to consider a Pod ready
- **revisionHistoryLimit**: Number of old history templates to retain

## DaemonSet Update Strategies

DaemonSets support two update strategies:

### 1. RollingUpdate (Default)

When a DaemonSet is updated with RollingUpdate strategy:

```yaml
spec:
  updateStrategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
```

- **maxUnavailable**: Maximum number or percentage of Pods that can be unavailable during the update

### 2. OnDelete

With OnDelete strategy, Pods will only be updated when manually deleted:

```yaml
spec:
  updateStrategy:
    type: OnDelete
```

## Working with DaemonSets

### Creating a DaemonSet

```bash
kubectl apply -f daemonset.yaml
```

### Getting Information About DaemonSets

```bash
# List all DaemonSets
kubectl get daemonsets --all-namespaces

# Get details about a specific DaemonSet
kubectl describe daemonset fluentd-elasticsearch -n kube-system
```

### Updating a DaemonSet

```bash
# Edit the DaemonSet directly
kubectl edit daemonset fluentd-elasticsearch -n kube-system

# Apply changes from an updated YAML file
kubectl apply -f updated-daemonset.yaml
```

### Deleting a DaemonSet

```bash
kubectl delete daemonset fluentd-elasticsearch -n kube-system
```

## Node Selection and Scheduling

DaemonSets can be configured to run only on specific nodes using:

### Node Selectors

```yaml
spec:
  template:
    spec:
      nodeSelector:
        disk: ssd
```

### Node Affinity

```yaml
spec:
  template:
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: kubernetes.io/os
                operator: In
                values:
                - linux
```

### Taints and Tolerations

To run on nodes with specific taints:

```yaml
spec:
  template:
    spec:
      tolerations:
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
```

## Advanced DaemonSet Features

### Monitoring DaemonSet Status

```bash
kubectl rollout status daemonset/fluentd-elasticsearch -n kube-system
```

### Rolling Back a DaemonSet Update

```bash
kubectl rollout undo daemonset/fluentd-elasticsearch -n kube-system
```

### Pausing and Resuming a DaemonSet Rollout

```bash
# Pause
kubectl rollout pause daemonset/fluentd-elasticsearch -n kube-system

# Resume
kubectl rollout resume daemonset/fluentd-elasticsearch -n kube-system
```

## Common Use Cases for DaemonSets

### 1. Log Collection

Collecting logs from all nodes in the cluster:

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-elasticsearch
  namespace: logging
spec:
  selector:
    matchLabels:
      name: fluentd-elasticsearch
  template:
    metadata:
      labels:
        name: fluentd-elasticsearch
    spec:
      containers:
      - name: fluentd-elasticsearch
        image: quay.io/fluentd_elasticsearch/fluentd:v2.5.2
        volumeMounts:
        - name: varlog
          mountPath: /var/log
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
```

### 2. Node Monitoring

Running monitoring agents on all nodes:

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-exporter
  namespace: monitoring
spec:
  selector:
    matchLabels:
      app: node-exporter
  template:
    metadata:
      labels:
        app: node-exporter
    spec:
      hostNetwork: true
      containers:
      - name: node-exporter
        image: prom/node-exporter:v1.1.2
        ports:
        - containerPort: 9100
          hostPort: 9100
        volumeMounts:
        - name: proc
          mountPath: /host/proc
          readOnly: true
        - name: sys
          mountPath: /host/sys
          readOnly: true
      volumes:
      - name: proc
        hostPath:
          path: /proc
      - name: sys
        hostPath:
          path: /sys
```

### 3. Networking Plugins

Deploying a network plugin to all nodes:

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: calico-node
  namespace: kube-system
spec:
  selector:
    matchLabels:
      k8s-app: calico-node
  template:
    metadata:
      labels:
        k8s-app: calico-node
    spec:
      hostNetwork: true
      containers:
      - name: calico-node
        image: calico/node:v3.19.1
        env:
        - name: DATASTORE_TYPE
          value: "kubernetes"
        securityContext:
          privileged: true
        volumeMounts:
        - name: lib-modules
          mountPath: /lib/modules
          readOnly: true
        - name: var-run-calico
          mountPath: /var/run/calico
        - name: var-lib-calico
          mountPath: /var/lib/calico
      volumes:
      - name: lib-modules
        hostPath:
          path: /lib/modules
      - name: var-run-calico
        hostPath:
          path: /var/run/calico
      - name: var-lib-calico
        hostPath:
          path: /var/lib/calico
```

## Best Practices for DaemonSets

1. **Resource Limits**: Always set resource requests and limits, as DaemonSets run on all nodes.
   
2. **Minimalism**: Keep DaemonSet Pods lightweight, as they consume resources on every node.
   
3. **Node Selectors**: Use node selectors or affinity to target specific nodes when needed.
   
4. **Tolerations**: Add tolerations to ensure critical DaemonSets run on all nodes, including control plane nodes.
   
5. **Host Resources**: Be careful when mounting host paths and ensure proper permissions.
   
6. **Update Strategy**: Use RollingUpdate with appropriate maxUnavailable value to prevent disruptions.
   
7. **Pod Priority**: Set appropriate PriorityClass for critical DaemonSets.
   
8. **Health Checks**: Implement proper liveness and readiness probes for reliable operation.

## Troubleshooting DaemonSets

### Common Issues

1. **Pods not scheduling**: Check node selectors, taints, and tolerations.
   
2. **Update failures**: Check for resource constraints or Pod errors.
   
3. **Pods failing**: Check container logs and events.
   
4. **Permissions issues**: Ensure proper RBAC and host access.

### Debugging Commands

```bash
# Check DaemonSet status
kubectl get daemonset <daemonset-name> -o wide

# Check for Pod events
kubectl describe pod <pod-name>

# Check logs
kubectl logs <pod-name>

# Check nodes for matching labels
kubectl get nodes --show-labels
```

## DaemonSet Examples

See the [examples](./examples/) directory for sample DaemonSet configurations.

## Next Steps

- Learn about [Jobs and CronJobs](../07-jobs-cronjobs/) for running batch and scheduled tasks
- Explore [ConfigMaps and Secrets](../08-configmaps-secrets/) for configuration management
- Understand [Persistent Volumes](../09-persistent-volumes/) for persistent storage 