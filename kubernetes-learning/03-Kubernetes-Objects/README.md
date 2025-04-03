# Kubernetes Objects

This directory contains information about various Kubernetes objects, their usage, and examples.

## Object Categories

### Workload Resources
Resources that manage the lifecycle of container workloads:

1. [Pods](./01-pods/) - The smallest deployable units in Kubernetes.
2. [ReplicaSets](./02-replicasets/) - Ensures that a specified number of pod replicas are running at any given time.
3. [Deployments](./03-deployments/) - Provides declarative updates for Pods and ReplicaSets.
4. [StatefulSets](./05-statefulsets/) - Manages the deployment and scaling of a set of Pods with unique identities and stable storage.
5. [DaemonSets](./06-daemonsets/) - Ensures that all (or some) nodes run a copy of a Pod.
6. [Jobs and CronJobs](./07-jobs-cronjobs/) - Run tasks that execute for a finite period of time, either once or on a schedule.

### Service Resources
Resources that define how to access applications:

1. [Services](./04-services/) - An abstraction that defines a logical set of Pods and a policy by which to access them.
2. [Networking and Ingress](./10-networking-ingress/) - Manages external access to services, providing HTTP routing and load balancing.

### Configuration and Storage Resources
Resources for application configuration and persistent data:

1. [ConfigMaps and Secrets](./08-configmaps-secrets/) - Manage configuration data and sensitive information separately from application code.
2. [Persistent Volumes](./09-persistent-volumes/) - Provides storage resources that outlive the lifecycle of Pods.

### Security Resources
Resources for securing Kubernetes clusters and applications:

1. [Security](./11-security/) - RBAC, Pod Security Standards, Network Policies, and Security Contexts.

## Learning Path

For those new to Kubernetes, we recommend following this progressive learning path:

1. **Start with Pods** - Understand the basic building block of Kubernetes applications.
2. **Move to ReplicaSets** - Learn how to run multiple identical Pods to provide reliability.
3. **Graduate to Deployments** - Discover how to manage application updates and rollbacks.
4. **Explore Services** - Find out how to expose your applications through stable network endpoints.
5. **Study StatefulSets** - Learn about stateful applications with stable identity and storage.
6. **Discover DaemonSets** - Understand how to run daemon processes on nodes.
7. **Master Jobs and CronJobs** - Learn how to run short-lived tasks either once or on a recurring schedule.
8. **Configure with ConfigMaps and Secrets** - Manage application configuration and sensitive data separately from your application code.
9. **Implement Persistent Volumes** - Learn how to use persistent storage for your applications, ensuring data survives Pod restarts and rescheduling.
10. **Secure your Applications** - Apply security best practices with RBAC, Pod Security Standards, and Network Policies.
11. **Set up Networking and Ingress** - Configure advanced networking and expose applications to the outside world.

Each section contains detailed documentation and practical YAML examples to help you understand and implement Kubernetes objects efficiently.

## Additional Kubernetes Concepts

After understanding the core objects, explore these additional Kubernetes concepts:

1. Horizontal Pod Autoscaling (HPA)
2. Vertical Pod Autoscaling (VPA)
3. Custom Resource Definitions (CRDs) and Operators
4. Helm Charts and Package Management
5. Monitoring and Observability
6. Multi-Cluster Management 