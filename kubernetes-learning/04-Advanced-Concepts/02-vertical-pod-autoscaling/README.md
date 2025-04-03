# Vertical Pod Autoscaling (VPA)

## Introduction
Vertical Pod Autoscaling automatically adjusts the CPU and memory resource requests and limits of containers in pods, helping to ensure that applications receive the resources they need while maximizing cluster resource utilization efficiency.

## How VPA Works
1. **Monitoring**: The VPA continually monitors the actual resource usage of your containers.
2. **Analysis**: It analyzes usage patterns to determine optimal resource requirements.
3. **Recommendation**: It calculates recommended CPU and memory settings.
4. **Application**: It applies these recommendations using one of three update policies:
   - `Off` - Recommendations only (no automatic updates)
   - `Initial` - Only applies to new pods
   - `Auto` - Automatically updates existing pods by evicting and recreating them

## Key Components
- **VPA Recommender**: Monitors resource usage and provides recommendations
- **VPA Updater**: Applies recommendations by evicting pods when necessary
- **VPA Admission Controller**: Modifies pod resource specifications during pod creation
- **Custom Resource Definition (CRD)**: `VerticalPodAutoscaler` resource defining the configuration

## Installation
VPA is not installed by default in Kubernetes clusters. To install VPA:

```bash
# Clone the VPA repository
git clone https://github.com/kubernetes/autoscaler.git

# Navigate to the VPA directory
cd autoscaler/vertical-pod-autoscaler

# Deploy VPA
./hack/vpa-up.sh
```

## Example VPA Configuration
```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: my-app-vpa
spec:
  targetRef:
    apiVersion: "apps/v1"
    kind: Deployment
    name: my-app
  updatePolicy:
    updateMode: "Auto"  # Options: Off, Initial, Auto
  resourcePolicy:
    containerPolicies:
      - containerName: '*'
        minAllowed:
          cpu: 100m
          memory: 50Mi
        maxAllowed:
          cpu: 1
          memory: 500Mi
        controlledResources: ["cpu", "memory"]
```

## Best Practices
1. **Start with "Off" mode**: Use this to observe recommendations without applying changes
2. **Set minimum and maximum bounds**: Define resource floors and ceilings to prevent extreme scaling
3. **Consider using VPA in "Initial" mode** with HPA: VPA sets initial values, HPA handles scaling
4. **Test thoroughly**: Evaluate VPA behavior in non-production environments first
5. **Monitor resource trends**: Use Kubernetes dashboards to track how VPA is adjusting resources

## VPA Limitations
1. VPA requires pod restarts to apply new resource settings when using "Auto" mode
2. Cannot be used with HPA for the same resource metric (e.g., CPU)
3. Does not work with DaemonSets
4. Recommendations may fluctuate with changing workloads
5. Beta feature, not recommended for critical production workloads without thorough testing

## Examples
1. Basic VPA Configuration
2. VPA with Resource Policies
3. VPA in Recommendation-Only Mode
4. VPA with Per-Container Configuration
5. Integration with HPA for Different Metrics

## Related Concepts
- Horizontal Pod Autoscaler (HPA)
- Cluster Autoscaler (CA)
- Quality of Service (QoS) Classes
- ResourceQuotas and LimitRanges
- Pod Disruption Budgets (PDBs)

## Further Reading
- [Kubernetes VPA Documentation](https://github.com/kubernetes/autoscaler/tree/master/vertical-pod-autoscaler)
- [VPA Design Proposal](https://github.com/kubernetes/design-proposals-archive/blob/main/autoscaling/vertical-pod-autoscaler.md)
- [Resource Management for Pods and Containers](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/)
- [Understanding Kubernetes Limits and Requests](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#resource-requests-and-limits-of-pod-and-container) 