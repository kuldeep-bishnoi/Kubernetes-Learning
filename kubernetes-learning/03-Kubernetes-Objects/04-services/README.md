# Kubernetes Services

## What is a Service?

A **Service** is an abstraction that defines a logical set of Pods and a policy to access them. Services enable network connectivity to a set of Pods, allowing them to be discovered and reached.

Services solve several critical challenges in containerized environments:
- **Pod IP Instability**: Pods are ephemeral and their IPs change when they restart
- **Load Balancing**: Traffic needs to be distributed among Pods
- **Service Discovery**: Applications need to find each other
- **External Access**: External clients need to access applications in the cluster

Think of Services as a stable network address for a set of Pods.

## How Services Work

```
┌──────────────────────────────────────────────────────────────┐
│                         Service                              │
│                                                              │
│    ┌ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┐          │
│      Service IP: 10.0.0.1                                    │
│      Port: 80                                                │
│    └ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┘          │
│                              │                               │
│                              ▼                               │
│    ┌──────────────┐    ┌──────────────┐    ┌──────────────┐ │
│    │     Pod      │    │     Pod      │    │     Pod      │ │
│    │  10.1.0.1    │    │  10.1.0.2    │    │  10.1.0.3    │ │
│    └──────────────┘    └──────────────┘    └──────────────┘ │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

When a Service is created, it gets a stable IP address. The service watches for Pods that match its selector, and automatically routes traffic to them. Even as Pods come and go, the Service's IP address remains stable.

## Service Types

Kubernetes offers several types of Services to cater to different access requirements:

### ClusterIP (Default)

```
┌─ Cluster ───────────────────────────────────────────┐
│                                                      │
│   ┌─────────────────┐        ┌─────────────────┐    │
│   │                 │        │                 │    │
│   │    Pod A        │◄──────▶│  ClusterIP      │    │
│   │                 │        │  Service        │    │
│   └─────────────────┘        └─────────────────┘    │
│                                                      │
└──────────────────────────────────────────────────────┘
```

- Internal access only (within the cluster)
- Gets a stable internal cluster IP
- Ideal for internal communication between applications

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: ClusterIP  # Default type, can be omitted
  selector:
    app: nginx
  ports:
  - port: 80        # Port exposed by the service
    targetPort: 80  # Port on the Pod
```

### NodePort

```
┌─ Cluster ─────────────────────────────────────────────────┐
│                                                            │
│   ┌─ Node ───────────────────────────────────────────┐    │
│   │                                                  │    │
│   │  NodePort                                        │    │
│   │  (30000-32767)                                   │    │
│   │       │                                          │    │
│   │       ▼                                          │    │
│   │  ┌─────────────────┐     ┌─────────────────┐    │    │
│   │  │                 │     │                 │    │    │
│   │  │  ClusterIP      │────▶│     Pod         │    │    │
│   │  │  Service        │     │                 │    │    │
│   │  └─────────────────┘     └─────────────────┘    │    │
│   │                                                  │    │
│   └──────────────────────────────────────────────────┘    │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

- Exposes the Service on each Node's IP at a static port
- Creates a ClusterIP Service automatically
- Accessible from outside the cluster
- Port range: 30000-32767

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
  - port: 80            # ClusterIP port
    targetPort: 80      # Pod port
    nodePort: 30080     # Port on Node (optional, auto-assigned if not specified)
```

### LoadBalancer

```
┌─ Cloud Provider ───────────────────────────────┐
│                                                │
│  Load Balancer                                 │
│  (External IP)                                 │
│       │                                        │
└───────┼────────────────────────────────────────┘
        │
        ▼
┌─ Cluster ───────────────────────────────────────────┐
│                                                      │
│   ┌─────────────────┐      ┌─────────────────┐      │
│   │                 │      │                 │      │
│   │    NodePort     │─────▶│  ClusterIP      │      │
│   │                 │      │  Service        │      │
│   └─────────────────┘      └─────────────────┘      │
│                                  │                   │
│                                  ▼                   │
│                           ┌─────────────────┐       │
│                           │                 │       │
│                           │     Pod         │       │
│                           │                 │       │
│                           └─────────────────┘       │
│                                                      │
└──────────────────────────────────────────────────────┘
```

- Exposes the Service externally using a cloud provider's load balancer
- Creates a NodePort and ClusterIP Services automatically
- Provides an external IP address
- Requires cloud provider support

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: LoadBalancer
  selector:
    app: nginx
  ports:
  - port: 80          # Service port
    targetPort: 80    # Pod port
```

### ExternalName

```
┌─ Cluster ───────────────────────────────────────────┐
│                                                      │
│   ┌─────────────────┐      ┌─────────────────┐      │
│   │                 │      │                 │      │
│   │  Application    │─────▶│  ExternalName   │ ───┐ │
│   │  Pod            │      │  Service        │    │ │
│   └─────────────────┘      └─────────────────┘    │ │
│                                                    │ │
└────────────────────────────────────────────────────┼─┘
                                                     │
                                                     ▼
                                         ┌─────────────────┐
                                         │  External       │
                                         │  Service/DNS    │
                                         └─────────────────┘
```

- Maps a Service to a DNS name, not to selectors and pods
- Acts as a CNAME record
- Useful for integrating external services into the cluster

```yaml
apiVersion: v1
kind: Service
metadata:
  name: external-db
spec:
  type: ExternalName
  externalName: db.example.com
```

## When to Use Each Service Type

Selecting the right service type is crucial for your application's networking requirements. Here's a comprehensive guide on when to use each type:

### ClusterIP

**Use ClusterIP when:**
- You need internal communication between applications in your cluster
- Your service should only be accessible from inside the cluster
- You're building microservices that talk to each other
- You want to expose a backend service to a frontend service
- You need a stable internal endpoint for your application

**Best for:**
- Internal microservices
- Backend APIs
- Database services
- Internal caching layers
- Service-to-service communication

**Example scenarios:**
- A backend API that only needs to be accessed by frontend pods
- A Redis cache that's only used by application servers within the cluster
- Internal service discovery where external access is not required

**Considerations:**
- Cannot be accessed from outside the cluster without additional proxying
- Simplest and most secure service type
- Default service type if none is specified

### NodePort

**Use NodePort when:**
- You need direct external access to your service
- You're in development or testing environments
- You need to expose services in on-premises clusters without load balancers
- You want to control your own load balancing
- You have a limited number of services that need external access

**Best for:**
- Development and testing environments
- Demo applications
- On-premises clusters without external load balancers
- Exposing services when you have limited external IPs

**Example scenarios:**
- A web application in a development environment
- Exposing a dashboard or admin interface during testing
- Providing services in edge environments with limited infrastructure

**Considerations:**
- Security implications of exposing node ports (require firewall rules)
- Limited to ports 30000-32767
- Every node in the cluster exposes the port, even if it doesn't run a pod
- Not ideal for production use cases with high availability requirements
- Does not provide load balancing automatically

### LoadBalancer

**Use LoadBalancer when:**
- You need to expose services to external users (internet traffic)
- You're running in cloud environments
- You need automatic load balancing across nodes
- You want a dedicated external IP for your service
- You have production workloads requiring high availability

**Best for:**
- Production web applications
- Public APIs
- Frontend services
- Any service that requires external traffic with high availability

**Example scenarios:**
- A public-facing web application
- Customer-facing APIs
- Any service you would traditionally put behind a load balancer

**Considerations:**
- Requires cloud provider support or external load balancer implementation
- Usually incurs additional cost (one load balancer per service)
- Provides automatic scaling and high availability
- Managed by your cloud provider, so less operational overhead
- For multiple services, consider Ingress instead to share a single load balancer

### Headless Services (ClusterIP with None)

**Use Headless Services when:**
- You need direct communication with specific pods
- You're working with stateful applications
- You need DNS for pods but don't need load balancing
- You're implementing service discovery at the application layer
- You're using StatefulSets

**Best for:**
- Databases with primary/replica architecture
- StatefulSets where identity matters
- Applications that need to connect to specific instances
- Custom service discovery implementations

**Example scenarios:**
- Distributed databases like Cassandra, MongoDB, or Elasticsearch
- Message brokers with specific addressing requirements
- Applications that need to know all endpoints for client-side load balancing

**Considerations:**
- No single service IP, returns individual Pod IPs
- DNS resolves to all Pod IPs
- No load balancing provided by Kubernetes
- Requires applications that understand how to work with multiple endpoints

### ExternalName

**Use ExternalName when:**
- You need to abstract external service access
- You want to refer to external services using internal DNS names
- You're migrating services to Kubernetes gradually
- You need to integrate with external services

**Best for:**
- Integration with external databases
- Connecting to external APIs
- Migration scenarios
- Abstracting environment-specific external services

**Example scenarios:**
- A database hosted outside the cluster
- Third-party APIs that your applications need to access
- Legacy systems that haven't been migrated to Kubernetes yet
- Services in different regions or cloud providers

**Considerations:**
- Only provides DNS CNAME record, no proxying or port remapping
- Does not perform health checking
- Cannot be used for TCP or UDP services (DNS resolution only)
- No load balancing or failover capabilities

## Service Discovery

Kubernetes provides two main approaches to service discovery:

### Environment Variables

When a Pod is created, Kubernetes automatically injects environment variables for each active Service:

```
NGINX_SERVICE_HOST=10.0.0.1
NGINX_SERVICE_PORT=80
```

### DNS

Kubernetes creates DNS records for Services and Pods:

- **Service DNS**: `<service-name>.<namespace>.svc.cluster.local`
- **Pod DNS**: `<pod-ip-with-dashes>.<namespace>.pod.cluster.local`

## Service Ports

Service objects allow you to define multiple ports and named ports:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
  - name: http
    port: 80
    targetPort: 9376
  - name: https
    port: 443
    targetPort: https  # Refers to the name of a port in the container
```

- **port**: The port exposed by the Service
- **targetPort**: The port on the Pod to which traffic is forwarded
- **nodePort**: The port on the Node for NodePort Services (30000-32767)

## Service Selectors

Services use label selectors to identify Pods:

```yaml
spec:
  selector:
    app: MyApp
    tier: frontend
```

The Service will target any Pod with labels matching **all** of the key-value pairs.

## Headless Services

A headless Service does not allocate a ClusterIP but returns the IPs of its Pods directly:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: headless-service
spec:
  clusterIP: None  # This makes it headless
  selector:
    app: MyApp
  ports:
  - port: 80
    targetPort: 80
```

Useful for:
- StatefulSets with stable network identities
- Direct Pod-to-Pod communication
- Client-side service discovery

## Service Networking

### kube-proxy

kube-proxy is a network proxy that runs on each node and is responsible for implementing Service abstractions. It has different operation modes:

#### iptables mode (default)

Uses iptables rules to perform packet redirection:
- Random pod selection
- Relatively efficient
- Connection failure if the selected pod is not available

#### IPVS mode

Uses Linux IP Virtual Server for more efficient load balancing:
- Multiple load balancing algorithms (round-robin, least connection, etc.)
- Better performance with large services
- Requires the IPVS kernel modules

## ExternalIPs

Services can be exposed on external IPs:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
  - port: 80
    targetPort: 80
  externalIPs:
  - 80.11.12.10
```

## Service Topology

Service topology allows you to route traffic based on Node topology:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
  - port: 80
  topologyKeys:
  - "kubernetes.io/hostname"
  - "topology.kubernetes.io/zone"
  - "topology.kubernetes.io/region"
  - "*"
```

This prioritizes routing traffic to pods on the same node, then same zone, then same region, and finally to any available pod.

## Session Affinity

Services can maintain session affinity to direct all requests from a client to the same Pod:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
  - port: 80
  sessionAffinity: ClientIP
  sessionAffinityConfig:
    clientIP:
      timeoutSeconds: 10800  # 3 hours
```

Supported values:
- **None** (default): Random Pod selection
- **ClientIP**: Stick requests from the same client IP to the same Pod

## Service Without Selectors

A Service without selectors doesn't automatically link to Pods but can be used with manually created Endpoints:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  ports:
  - port: 80
    targetPort: 80
---
apiVersion: v1
kind: Endpoints
metadata:
  name: my-service  # Must match Service name
subsets:
- addresses:
  - ip: 192.168.0.1
  - ip: 192.168.0.2
  ports:
  - port: 80
```

This is useful for:
- Connecting to external databases
- Pointing to services in a different namespace
- Migration scenarios

## Common Service Issues & Troubleshooting

### Can't Connect to Service

1. Check if the service exists:
   ```bash
   kubectl get service my-service
   ```

2. Verify endpoint creation:
   ```bash
   kubectl get endpoints my-service
   ```

3. Check if selector matches Pod labels:
   ```bash
   kubectl get pods --selector=app=MyApp
   ```

4. Validate that Pods are Running and Ready:
   ```bash
   kubectl get pods --selector=app=MyApp -o wide
   ```

5. Test service DNS resolution:
   ```bash
   kubectl run -it --rm debug --image=busybox -- nslookup my-service
   ```

### LoadBalancer Service Stuck in Pending

If a LoadBalancer Service is stuck in "Pending" state:
- Ensure the cloud provider is configured correctly
- Check if the cloud provider has quotas for load balancers
- Verify cloud credentials and permissions

### Traffic Not Reaching Pods

1. Check if targetPort matches the container port:
   ```bash
   kubectl describe pods --selector=app=MyApp | grep -i port
   ```

2. Verify that the application is listening on the correct port and interface

3. Test Pod-to-Pod communication:
   ```bash
   kubectl exec -it <pod-name> -- curl <pod-ip>:<port>
   ```

## Best Practices

1. **Use Meaningful Names**: Give Services descriptive names that reflect their purpose
2. **Define Health Checks**: Ensure Pods have proper liveness and readiness probes
3. **Select by App & Role**: Use selectors that include both the application and its role
4. **Use Named Ports**: Define named ports for better readability
5. **Keep Services Focused**: One Service should serve one logical application function
6. **Consider Session Affinity**: Use when your application requires sticky sessions
7. **Document ExternalName Services**: Clearly document external dependencies
8. **Use Appropriate Service Type**: Don't use LoadBalancer when NodePort will do
9. **Prefer DNS Over Environment Variables**: DNS is more flexible and easier to manage
10. **Labels and Annotations**: Add metadata to aid discovery and management

## Examples

### Simple Web Application Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service
  labels:
    app: web
spec:
  selector:
    app: web
  ports:
  - name: http
    port: 80
    targetPort: 8080
```

### Multi-Port Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-multiport-service
spec:
  selector:
    app: multi-port-app
  ports:
  - name: http
    port: 80
    targetPort: 8080
  - name: https
    port: 443
    targetPort: 8443
  - name: admin
    port: 8080
    targetPort: 9090
```

### Internal Database Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: db
  namespace: backend
  labels:
    app: db
    tier: database
spec:
  selector:
    app: db
    tier: database
  ports:
  - port: 5432
    targetPort: 5432
  clusterIP: None  # Headless service for direct Pod addressing
```

### Public Web Service with LoadBalancer

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-public
  labels:
    app: web
    exposure: public
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-type: "nlb"  # Cloud provider specific
spec:
  type: LoadBalancer
  selector:
    app: web
    tier: frontend
  ports:
  - name: http
    port: 80
    targetPort: 8080
  - name: https
    port: 443
    targetPort: 8443
```

## Exercises

1. **Create a ClusterIP Service**:
   - Deploy a simple application with 3 replicas
   - Create a ClusterIP Service to access it
   - Test access from another Pod using the Service name

2. **Create and Test a NodePort Service**:
   - Create a NodePort Service for the same application
   - Access the application from outside the cluster using a Node's IP and the NodePort

3. **Use a Headless Service with StatefulSet**:
   - Deploy a StatefulSet with multiple replicas
   - Create a headless Service for it
   - Test DNS resolution for individual Pods

4. **Implement Service with Multiple Ports**:
   - Deploy an application that listens on multiple ports
   - Create a Service that exposes all these ports
   - Test connectivity to each port

## Next Steps

Now that you understand Kubernetes Services, you can explore other ways to expose applications:

- [Ingress Controllers](../../04-Networking/03-ingress)
- [Network Policies](../../04-Networking/04-network-policies)
- Service Meshes like Istio, Linkerd, or Consul

## Additional Resources

- [Kubernetes Services Documentation](https://kubernetes.io/docs/concepts/services-networking/service/)
- [Connecting Applications with Services](https://kubernetes.io/docs/concepts/services-networking/connect-applications-service/)
- [DNS for Services and Pods](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/)
- [Service Topologies](https://kubernetes.io/docs/concepts/services-networking/service-topology/) 