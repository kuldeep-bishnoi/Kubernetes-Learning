# Kubernetes Deployments

Deployments are one of the most commonly used workload resources in Kubernetes. They provide declarative updates for Pods and ReplicaSets, making it easy to manage applications and their lifecycle.

![Deployment Architecture](https://raw.githubusercontent.com/kubernetes/website/main/content/en/examples/controllers/nginx-deployment.svg)

## What is a Deployment?

A Deployment provides declarative updates for Pods and ReplicaSets. You describe a desired state in a Deployment, and the Deployment Controller changes the actual state to the desired state at a controlled rate. Deployments manage ReplicaSets, which in turn manage Pods.

### Key Features of Deployments

- **Declarative Application Updates**: Define your desired state, and Kubernetes handles the rest
- **Rollout and Rollback**: Control how many pods are replaced at a time during updates
- **Scaling**: Easily scale applications up or down
- **Pause and Resume**: Pause a deployment to make multiple fixes, then resume it
- **Version History**: Track changes and revert to previous states when needed

## When to Use Deployments

Deployments are ideal for:

- Stateless applications
- Applications that require zero-downtime updates
- Scenarios where you need to roll back quickly
- When you need automatic scaling of application instances

## Deployment vs. Other Controllers

| Controller | Use Case | State Management | Update Strategy |
|------------|----------|------------------|-----------------|
| Deployment | Stateless applications | Maintains desired state | Rolling updates, rollbacks |
| StatefulSet | Stateful applications | Ordered deployment with stable identifiers | Ordered, controlled updates |
| DaemonSet | Running a pod on every node | Ensures one pod per node | Node-aware updates |
| Job/CronJob | One-off or scheduled tasks | Runs to completion | Not designed for updates |

## Deployment Specification

A basic Deployment YAML looks like this:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3                  # Number of desired pods
  selector:
    matchLabels:
      app: nginx               # Selects which pods to manage
  template:                    # Pod template
    metadata:
      labels:
        app: nginx             # Must match selector
    spec:
      containers:
      - name: nginx
        image: nginx:1.16
        ports:
        - containerPort: 80
```

### Key Fields in Deployment Spec

- **replicas**: Number of desired pods running at any given time
- **selector**: How the Deployment finds which pods to manage
- **template**: The pod template that defines what each pod should look like
- **strategy**: How to replace old pods with new ones during updates
- **revisionHistoryLimit**: Number of old ReplicaSets to retain
- **minReadySeconds**: Minimum time a pod should be ready before considered available
- **progressDeadlineSeconds**: Maximum time for a deployment before it's considered failed

## Deployment Strategies

Kubernetes Deployments support two update strategies:

### 1. RollingUpdate (Default)

- Gradually replaces old pods with new ones
- Configurable with `maxUnavailable` and `maxSurge` parameters
- Zero-downtime updates when configured properly

```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 25%   # Maximum number of pods that can be unavailable during update
      maxSurge: 25%         # Maximum number of pods that can exceed desired number of pods
```

### 2. Recreate

- Terminates all existing pods before creating new ones
- Results in downtime but ensures no two versions run simultaneously

```yaml
spec:
  strategy:
    type: Recreate
```

## Working with Deployments

### Creating a Deployment

```bash
kubectl apply -f deployment.yaml
```

### Checking Deployment Status

```bash
# Get all deployments
kubectl get deployments

# Get detailed information about a deployment
kubectl describe deployment nginx-deployment

# Check rollout status
kubectl rollout status deployment/nginx-deployment
```

### Updating a Deployment

You can update a deployment by:

1. Editing the YAML and reapplying:

```bash
kubectl apply -f updated-deployment.yaml
```

2. Using kubectl set commands:

```bash
# Update image
kubectl set image deployment/nginx-deployment nginx=nginx:1.17

# Scale a deployment
kubectl scale deployment/nginx-deployment --replicas=5
```

### Rolling Back a Deployment

```bash
# View revision history
kubectl rollout history deployment/nginx-deployment

# Roll back to the previous revision
kubectl rollout undo deployment/nginx-deployment

# Roll back to a specific revision
kubectl rollout undo deployment/nginx-deployment --to-revision=2
```

### Pausing and Resuming a Deployment

```bash
# Pause a deployment
kubectl rollout pause deployment/nginx-deployment

# Make multiple changes while paused
kubectl set resources deployment/nginx-deployment -c=nginx --limits=cpu=200m,memory=512Mi

# Resume the rollout
kubectl rollout resume deployment/nginx-deployment
```

## Deployment Patterns

### Blue-Green Deployments

Deploy the new version (green) alongside the old version (blue), then switch traffic all at once.

Implementation approach:
1. Create a new deployment with the updated version
2. Verify the new deployment is working
3. Update the service selector to point to the new deployment
4. Remove the old deployment when no longer needed

### Canary Deployments

Gradually shift traffic from the old version to the new version.

Implementation approach:
1. Deploy a small number of pods with the new version alongside the existing deployment
2. Direct a small percentage of traffic to the new version
3. Monitor for issues and gradually increase traffic to the new version
4. Eventually replace the old version completely

## Common Issues and Troubleshooting

### Deployment Stuck

If a deployment gets stuck, check:

```bash
kubectl describe deployment nginx-deployment
```

Common causes:
- Insufficient cluster resources
- Image pull failures
- Readiness probe failures
- Configuration errors

### Rollout Too Slow

If rollouts are too slow, adjust:
- `maxUnavailable` and `maxSurge` parameters
- Pod readiness probes and their timeouts
- `minReadySeconds` to an appropriate value

## Best Practices

1. **Set resource requests and limits** for predictable performance
2. **Configure probes properly** to ensure proper pod health detection
3. **Use labels and selectors** consistently and meaningfully
4. **Set an appropriate revisionHistoryLimit** to control resource usage
5. **Consider using pod disruption budgets** for high-availability workloads
6. **Implement progressive delivery patterns** for critical applications

## Deployment Examples

See the [examples](./examples/) directory for sample deployment configurations.

## Next Steps

- Learn about [Services](../04-services/) for exposing your deployments
- Explore [ConfigMaps and Secrets](../06-configs-and-secrets/) for configuration
- Understand [Horizontal Pod Autoscaler](../08-autoscaling/) for automatic scaling

## Additional Resources

- [Kubernetes Deployments Documentation](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
- [Rolling Update Strategy](https://kubernetes.io/docs/tutorials/kubernetes-basics/update/update-intro/)
- [Deployment Patterns](https://kubernetes.io/blog/2018/04/30/zero-downtime-deployment-kubernetes-jenkins/) 