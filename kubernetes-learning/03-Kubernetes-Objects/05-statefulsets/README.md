# StatefulSets in Kubernetes

A StatefulSet is a Kubernetes workload API object used to manage stateful applications. Unlike Deployments, which are designed for stateless applications, StatefulSets provide guarantees about the ordering and uniqueness of Pods.

## How StatefulSets Work

```
┌─ Kubernetes Cluster ───────────────────────────────────────┐
│                                                            │
│  ┌─ StatefulSet (db) ────────────────────────────────────┐ │
│  │                                                        │ │
│  │  ┌────────────┐   ┌────────────┐   ┌────────────┐     │ │
│  │  │  Pod: db-0 │   │  Pod: db-1 │   │  Pod: db-2 │     │ │
│  │  │            │   │            │   │            │     │ │
│  │  │ PVC: vol-0 │   │ PVC: vol-1 │   │ PVC: vol-2 │     │ │
│  │  └────────────┘   └────────────┘   └────────────┘     │ │
│  │                                                        │ │
│  └────────────────────────────────────────────────────────┘ │
│                                                            │
│  ┌─ Headless Service (db) ─────────────────────────────────┐│
│  │                                                          ││
│  │  DNS:                                                    ││
│  │  db-0.db.namespace.svc.cluster.local -> Pod db-0 IP     ││
│  │  db-1.db.namespace.svc.cluster.local -> Pod db-1 IP     ││
│  │  db-2.db.namespace.svc.cluster.local -> Pod db-2 IP     ││
│  │                                                          ││
│  └──────────────────────────────────────────────────────────┘│
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

When you create a StatefulSet, Kubernetes:

1. Creates pods sequentially, in order by their ordinal index (0, 1, 2, etc.)
2. Waits for each pod to be Running and Ready before creating the next
3. Assigns each pod a unique, stable network identity with a predictable hostname
4. Creates and attaches a persistent volume to each pod (if volumeClaimTemplates are defined)
5. On deletion, removes pods in reverse order, from highest to lowest index
6. When scaling down, removes the highest-numbered pods first

## Key Features of StatefulSets

- **Stable, Unique Network Identifiers**: Each pod gets a hostname based on the StatefulSet name and ordinal index
- **Ordered Pod Creation and Deletion**: Pods are created and deleted in a defined order
- **Stable, Persistent Storage**: Each pod can have its own persistent storage that survives pod rescheduling
- **Ordered, Automated Rolling Updates**: Updates can be applied to pods in order, with guarantees for how many pods are updated at once

## When to Use StatefulSets

Use StatefulSets for applications that require one or more of the following:

- Stable, unique network identifiers
- Stable, persistent storage
- Ordered, graceful deployment and scaling
- Ordered, automated rolling updates

Common use cases include:

- Databases (MySQL, PostgreSQL, MongoDB)
- Distributed storage systems (Cassandra, Elasticsearch)
- Message brokers with persistent data (Kafka, RabbitMQ)
- Applications that require stable hostnames or coordination on startup

## Basic StatefulSet Example

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None              # Headless service for StatefulSet DNS
  selector:
    app: nginx
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "nginx"         # Must match the headless service name
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.20
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:        # PVCs created automatically for each pod
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
```

## StatefulSet Specifications

A StatefulSet configuration consists of:

### 1. Metadata

```yaml
metadata:
  name: mysql                 # StatefulSet name, will be used as prefix for pod names
  labels:
    app: mysql                # Labels for the StatefulSet
```

### 2. StatefulSet Spec

```yaml
spec:
  replicas: 3                 # Number of pods to run
  serviceName: mysql          # Name of the headless service
  podManagementPolicy: OrderedReady  # OrderedReady or Parallel
  updateStrategy:
    type: RollingUpdate
    rollingUpdate:
      partition: 0            # Only update pods with index >= partition
  selector:
    matchLabels:
      app: mysql              # Pod selector
```

### 3. Pod Template

```yaml
template:
  metadata:
    labels:
      app: mysql              # Labels for the pods (must match selector)
  spec:
    terminationGracePeriodSeconds: 10
    containers:
    - name: mysql
      image: mysql:5.7
      ports:
      - containerPort: 3306
        name: mysql
      env:
      - name: MYSQL_ROOT_PASSWORD
        valueFrom:
          secretKeyRef:
            name: mysql-secrets
            key: password
      volumeMounts:
      - name: data
        mountPath: /var/lib/mysql
```

### 4. Volume Claim Templates

```yaml
volumeClaimTemplates:         # Define PVC templates for each pod
- metadata:
    name: data                # Will create PVCs named data-mysql-0, data-mysql-1, etc.
  spec:
    accessModes: [ "ReadWriteOnce" ]
    storageClassName: "standard"
    resources:
      requests:
        storage: 10Gi
```

## Creating and Managing StatefulSets

### Creating a StatefulSet

```bash
# Create the headless service first
kubectl apply -f headless-service.yaml

# Create the StatefulSet
kubectl apply -f statefulset.yaml
```

### Checking StatefulSet Status

```bash
# Get all StatefulSets
kubectl get statefulsets

# Detailed information about a StatefulSet
kubectl describe statefulset mysql

# Watch the pods being created sequentially
kubectl get pods -l app=mysql -w
```

### Scaling a StatefulSet

```bash
# Scale up to 5 replicas
kubectl scale statefulset mysql --replicas=5

# Scale down to 3 replicas
kubectl scale statefulset mysql --replicas=3
```

### Updating a StatefulSet

```bash
# Update the container image
kubectl set image statefulset/mysql mysql=mysql:8.0

# Edit the StatefulSet directly
kubectl edit statefulset mysql

# Apply changes from an updated YAML file
kubectl apply -f updated-statefulset.yaml
```

### Deleting a StatefulSet

```bash
# Delete the StatefulSet but keep the PVCs (default)
kubectl delete statefulset mysql

# Delete the StatefulSet and its PVCs
kubectl delete statefulset mysql --cascade=foreground
```

## StatefulSet Update Strategies

StatefulSets support two update strategies:

### 1. RollingUpdate (Default)

Updates pods one at a time, in reverse order (highest to lowest):

```yaml
spec:
  updateStrategy:
    type: RollingUpdate
    rollingUpdate:
      partition: 0    # Only update pods with ordinalIndex >= partition
```

Setting the partition allows for canary testing by only updating pods with an index greater than or equal to the partition value.

### 2. OnDelete

Pods are only updated when manually deleted:

```yaml
spec:
  updateStrategy:
    type: OnDelete
```

With this strategy, you must manually delete pods to trigger updates.

## Pod Management Policies

StatefulSets support two pod management policies:

### 1. OrderedReady (Default)

```yaml
spec:
  podManagementPolicy: OrderedReady
```

- Sequential pod creation (0, 1, 2, ...)
- Sequential pod deletion (n, n-1, ...)
- Waits for each pod to be Running and Ready before creating the next

### 2. Parallel

```yaml
spec:
  podManagementPolicy: Parallel
```

- Parallel pod creation (all at once)
- Parallel pod deletion (all at once)
- Does not wait for pods to be Running and Ready before creating others

Use Parallel when pod startup order doesn't matter but you still need stable identities and storage.

## StatefulSet Pod Identity

Each StatefulSet pod gets three identifying attributes:

1. **Ordinal Index**: A numeric index (0, 1, 2, ...) used in the pod name
2. **Stable Network Identity**: Hostname follows the pattern `<statefulset-name>-<ordinal-index>`
3. **Stable Storage**: PVCs follow the pattern `<volume-name>-<statefulset-name>-<ordinal-index>`

These identities remain the same even if the pod is rescheduled on a different node.

## Accessing StatefulSet Pods

### 1. Individual Pod Access via DNS

With a headless service, each pod gets its own DNS address:

```
<pod-name>.<service-name>.<namespace>.svc.cluster.local
```

For example:
```
mysql-0.mysql.default.svc.cluster.local
mysql-1.mysql.default.svc.cluster.local
```

### 2. Service Access

A regular service can load balance across all pods:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql-read
spec:
  ports:
  - port: 3306
  selector:
    app: mysql
```

### 3. Headless Service for StatefulSet

A headless service is required for StatefulSets to provide DNS records:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql
spec:
  ports:
  - port: 3306
  clusterIP: None    # This makes it a headless service
  selector:
    app: mysql
```

## Common StatefulSet Patterns

### 1. Master-Slave Replication

For databases that use leader-follower replication:

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
spec:
  replicas: 3
  serviceName: mysql
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      initContainers:
      - name: init-mysql
        image: mysql:5.7
        command:
        - bash
        - "-c"
        - |
          set -ex
          # Generate mysql server-id from pod ordinal index.
          [[ `hostname` =~ -([0-9]+)$ ]] || exit 1
          ordinal=${BASH_REMATCH[1]}
          # Clone data from master if this is a slave.
          if [[ $ordinal -gt 0 ]]; then
            # Copy data from master if this is a slave.
            ncat --recv-only mysql-0.mysql 3307 | xbstream -x -C /var/lib/mysql
            # Prepare the backup.
            xtrabackup --prepare --target-dir=/var/lib/mysql
          fi
```

### 2. Distributed Consensus

For systems like etcd or ZooKeeper that need consensus:

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: zk
spec:
  serviceName: zk-headless
  replicas: 3
  template:
    spec:
      containers:
      - name: zookeeper
        env:
        - name: ZK_REPLICAS
          value: "3"
        - name: ZK_SERVER_ID
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        command:
        - sh
        - -c
        - "start-zookeeper \
          --servers=3 \
          --server_id=$(echo $ZK_SERVER_ID | sed 's/.*-//') \
          --data_dir=/var/lib/zookeeper"
```

### 3. Sharded Services

For distributed databases with sharding:

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mongodb-shard
spec:
  serviceName: mongodb-shard
  replicas: 3
  template:
    spec:
      containers:
      - name: mongodb
        command:
        - mongod
        - "--shardsvr"
        - "--replSet"
        - "shardrs"
```

## Advanced StatefulSet Concepts

### Init Containers for Bootstrapping

Use init containers to set up dependencies or prepare data:

```yaml
spec:
  template:
    spec:
      initContainers:
      - name: init-myservice
        image: busybox:1.28
        command: ['sh', '-c', 'until nslookup myservice; do echo waiting for myservice; sleep 2; done;']
```

### Pod Affinity and Anti-Affinity

Ensure high availability by placing pods on different nodes:

```yaml
spec:
  template:
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - mysql
            topologyKey: "kubernetes.io/hostname"
```

### PodDisruptionBudget for High Availability

Protect against voluntary disruptions:

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: mysql-pdb
spec:
  minAvailable: 2  # or maxUnavailable: 1
  selector:
    matchLabels:
      app: mysql
```

## Common StatefulSet Issues and Troubleshooting

### 1. Pods Stuck in Pending

Check for:
- Insufficient resources
- PV provisioning issues
- PVC binding problems

```bash
# Check pod status
kubectl describe pod mysql-0

# Check PVC status
kubectl get pvc
kubectl describe pvc data-mysql-0
```

### 2. StatefulSet Won't Scale Up

Check for:
- Storage class exists and is working
- PV provisioner is running
- Pod anti-affinity rules can be satisfied

### 3. Network Identity Issues

Check the headless service:

```bash
# Verify the headless service exists
kubectl get svc mysql

# Make sure it has the correct selector
kubectl describe svc mysql

# Test DNS resolution from a pod
kubectl run -it --rm debug --image=busybox -- nslookup mysql-0.mysql
```

### 4. PVC Not Releasing After Deletion

Check PVC protection:

```bash
# Check if PVC is in Terminating state
kubectl get pvc

# Check finalizers
kubectl get pvc data-mysql-0 -o yaml | grep finalizers
```

## StatefulSet Best Practices

1. **Use Headless Services**: Always create a headless service to provide DNS records for pods

2. **Set Appropriate Resource Requests/Limits**: Especially for databases, ensure sufficient resources

3. **Configure Readiness Probes**: Ensure each pod is truly ready before starting the next one

4. **Use Pod Anti-Affinity**: Distribute pods across different nodes for high availability

5. **Set Appropriate Update Strategy**: 
   - For production databases, use `OnDelete` or `RollingUpdate` with manual checks
   - For less critical services, use `RollingUpdate` with appropriate partition for canary testing

6. **Plan for Storage Performance**: Choose appropriate storage classes for your workload

7. **Backup Strategies**: Implement regular backups of persistent data

8. **Consider StatefulSet Alternatives**: 
   - Operators for complex stateful applications (e.g., Kafka, Cassandra)
   - Cloud managed services for databases

9. **Graceful Shutdown**: Set appropriate `terminationGracePeriodSeconds` for clean shutdown

10. **Use Init Containers**: For proper bootstrapping, especially for clustered applications

## Exercises

1. **Basic StatefulSet**:
   - Create a StatefulSet running a simple nginx web server with 3 replicas
   - Each pod should have its own PVC
   - Verify stable network identities by accessing individual pods

2. **MySQL Master-Slave Setup**:
   - Create a StatefulSet for MySQL with a master and two replicas
   - Configure replication using init containers
   - Test failover scenarios

3. **Advanced StatefulSet**:
   - Create a StatefulSet for a distributed application (e.g., Elasticsearch)
   - Configure pod anti-affinity for high availability
   - Implement a proper update strategy with canary testing

## Next Steps

- Learn about Operators, which extend Kubernetes for complex stateful applications
- Explore Helm Charts for deploying complex StatefulSet-based applications
- Understand PersistentVolumes and StorageClasses in depth

## Additional Resources

- [Kubernetes StatefulSets Documentation](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/)
- [StatefulSet Basics Tutorial](https://kubernetes.io/docs/tutorials/stateful-application/basic-stateful-set/)
- [Run a Replicated Stateful Application](https://kubernetes.io/docs/tasks/run-application/run-replicated-stateful-application/) 