# Horizontal Pod Autoscaling (HPA)

This directory contains information about Horizontal Pod Autoscaling in Kubernetes.

## Introduction

Horizontal Pod Autoscaling (HPA) automatically scales the number of pod replicas in a deployment, replicaset, or statefulset based on observed CPU utilization or other metrics. HPA helps ensure that your application can handle increases in demand while also scaling down during periods of low activity, optimizing resource usage and cost.

## How HPA Works

1. **Metrics Collection**: HPA collects metrics from the Metrics Server or a custom metrics API.
2. **Decision Making**: Based on the collected metrics and defined thresholds, HPA calculates the desired number of replicas.
3. **Scaling Action**: HPA updates the replica count of the target resource (Deployment, ReplicaSet, StatefulSet).

## Key Components

- **Metrics Server**: A cluster add-on that collects resource metrics from kubelets and exposes them via the Kubernetes metrics API.
- **HorizontalPodAutoscaler Resource**: Defines the scaling behavior and metrics to monitor.
- **Custom Metrics API**: Optional component that allows scaling based on application-specific metrics.

## Common Metrics for Scaling

1. **CPU Utilization**: The most common metric, representing the average CPU usage across all pods.
2. **Memory Usage**: Average memory consumption across pods.
3. **Custom Metrics**: Application-specific metrics such as requests per second, queue length, etc.
4. **External Metrics**: Metrics from external sources like cloud provider services.

## Example HPA Configuration

A basic HPA that scales based on CPU utilization:

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: example-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: example-deployment
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 80
```

This HPA will maintain between 2 and 10 replicas of the `example-deployment`, aiming to keep the average CPU utilization at 80%.

## Best Practices

1. **Set Appropriate Resource Requests**: Since CPU-based autoscaling uses the CPU request as the baseline, ensure your containers have appropriate resource requests defined.
2. **Define Reasonable Min/Max Replicas**: Set minimum replicas to ensure baseline availability and maximum replicas to prevent runaway scaling.
3. **Choose Appropriate Metrics**: CPU is a good starting point, but consider using application-specific metrics for more precise scaling.
4. **Avoid Scaling Thrashing**: Configure appropriate cooldown periods to prevent rapid scaling up and down.
5. **Test Your Scaling Behavior**: Verify that your application scales appropriately under various load conditions.

## Examples

This directory contains examples demonstrating various HPA configurations:

1. [Basic CPU-based HPA](./examples/01-basic-cpu-hpa.yaml) - Simple HPA configuration based on CPU usage.
2. [Memory-based HPA](./examples/02-memory-hpa.yaml) - HPA configuration using memory as the scaling metric.
3. [Multi-metric HPA](./examples/03-multi-metric-hpa.yaml) - HPA using multiple metrics for scaling decisions.
4. [Custom Metrics HPA](./examples/04-custom-metrics-hpa.yaml) - HPA using application-specific metrics.
5. [Scaling with Behavior Rules](./examples/05-scaling-behavior.yaml) - HPA with custom scaling behavior rules.

## Limitations

1. HPA doesn't handle vertical scaling (changing CPU/memory resources for individual pods).
2. Cannot directly scale based on predicted future load.
3. Not ideal for applications with unpredictable spikes that require immediate scaling.

## Related Concepts

- **Vertical Pod Autoscaler (VPA)**: Adjusts CPU and memory requests of pod containers.
- **Cluster Autoscaler**: Scales the number of nodes in a cluster based on pod scheduling requirements.
- **Custom Metrics Adapter**: Extends HPA capabilities to use custom metrics from your applications.

## Further Reading

- [Kubernetes Documentation: Horizontal Pod Autoscaling](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)
- [Kubernetes HPA v2 API Reference](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.25/#horizontalpodautoscaler-v2-autoscaling)
- [KEDA - Kubernetes Event-driven Autoscaling](https://keda.sh/) 