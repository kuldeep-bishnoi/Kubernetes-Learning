# Kubernetes Networking and Ingress

Networking is a critical component in Kubernetes that enables communication between applications, both within and outside the cluster. This document explains Kubernetes networking concepts, how to expose applications, and how to manage traffic with Ingress controllers.

![Kubernetes Networking](https://d33wubrfki0l68.cloudfront.net/e351b830334b8622a700a8da6568cb081c464a19/13020/images/docs/ingress.svg)

## Table of Contents
- [Kubernetes Networking Fundamentals](#kubernetes-networking-fundamentals)
- [Pod Networking](#pod-networking)
- [Services](#services)
- [Network Policies](#network-policies)
- [Ingress](#ingress)
- [Service Mesh](#service-mesh)
- [DNS in Kubernetes](#dns-in-kubernetes)
- [Examples](#examples)
- [Best Practices](#best-practices)
- [Troubleshooting](#troubleshooting)
- [Further Reading](#further-reading)

## Kubernetes Networking Fundamentals

Kubernetes networking adheres to the following fundamental requirements:

1. **Pods on a node can communicate with all pods on all nodes without NAT**
2. **Agents on a node can communicate with all pods on that node**
3. **Pods in the host network can communicate with all pods on all nodes**

The Kubernetes networking model is a flat network where every pod gets its own IP address, enabling pod-to-pod communication without the need for NAT.

### Container Network Interface (CNI)

Kubernetes uses CNI plugins to implement its networking model. Popular CNI plugins include:

- **Calico**: Network policy enforcement and routing
- **Flannel**: Simple overlay network focused on traffic encapsulation
- **Cilium**: eBPF-based networking, observability, and security
- **Weave Net**: Multi-host container networking
- **AWS VPC CNI**: Uses AWS VPC networking for pods
- **Azure CNI**: Integrates with Azure Virtual Networks

## Pod Networking

### Pod IP Address

Each Pod in a Kubernetes cluster receives a unique IP address. Containers within the same Pod share the network namespace and can communicate using localhost.

### Communication Patterns

1. **Container-to-Container**: Via localhost within the same Pod
2. **Pod-to-Pod**: Direct using Pod IPs (regardless of node)
3. **Pod-to-Service**: Using Service IP or DNS name
4. **External-to-Pod**: Through Services, Ingress, or direct node ports

## Services

Services provide stable endpoints for pods, regardless of their lifecycle or IP changes. Types of Services include:

### ClusterIP

- Default service type
- Internal-only IP accessible within the cluster
- Load balances traffic across pod replicas
- Use for internal communication between components

### NodePort

- Exposes the Service on a static port on each Node
- Accessible from outside using `<NodeIP>:<NodePort>`
- Limited to port range 30000-32767 by default
- Useful for development or temporary external access

### LoadBalancer

- Creates an external load balancer in cloud environments
- Automatically assigns an external IP/hostname
- Routes traffic to Service pods
- Best for production external-facing applications

### ExternalName

- Maps a Service to a DNS name, not to selectors
- Used for accessing external services through internal DNS names
- No proxying or redirection, just DNS CNAME

### Headless Services

- Services without ClusterIP (clusterIP: None)
- Return individual Pod IP addresses instead of a single service IP
- Used primarily with StatefulSets for direct Pod addressing

## Network Policies

Network Policies are specifications of how pods are allowed to communicate with each other and other network endpoints. They act as a firewall for pod traffic.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: api-allow
spec:
  podSelector:
    matchLabels:
      app: api
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: web
    ports:
    - protocol: TCP
      port: 8080
```

Key aspects:
- **podSelector**: Identifies which pods the policy applies to
- **policyTypes**: Whether policy applies to ingress, egress, or both
- **ingress/egress**: Rules defining allowed traffic
- **namespaceSelector**: Extend rules across namespaces
- **ipBlock**: Specify external CIDR blocks for traffic

> Note: Network Policies require a CNI plugin that supports them (e.g., Calico, Cilium).

## Ingress

Ingress manages external access to services, typically HTTP, providing features like:
- URL path-based routing
- SSL/TLS termination
- Name-based virtual hosting
- Load balancing
- Custom rules via annotations

### Ingress Controllers

Ingress requires an Ingress Controller to function. Popular controllers include:

- **Nginx Ingress Controller**
- **Traefik**
- **HAProxy**
- **AWS ALB Ingress Controller**
- **GCE Ingress Controller**
- **Istio Ingress**

### Basic Ingress Example

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: my-app.example.com
    http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 80
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-service
            port:
              number: 80
  tls:
  - hosts:
    - my-app.example.com
    secretName: example-tls-secret
```

### IngressClass

IngressClass separates controller implementations and allows multiple Ingress controllers in the same cluster.

```yaml
apiVersion: networking.k8s.io/v1
kind: IngressClass
metadata:
  name: nginx
  annotations:
    ingressclass.kubernetes.io/is-default-class: "true"
spec:
  controller: k8s.io/ingress-nginx
```

## Service Mesh

Service meshes provide advanced traffic management, security, and observability features for microservices.

Popular service mesh options:
- **Istio**: Comprehensive features for traffic management, security, and observability
- **Linkerd**: Lightweight, focused on simplicity and performance
- **Consul Connect**: HashiCorp's service mesh solution
- **AWS App Mesh**: AWS-managed service mesh

Key features:
- **Traffic Management**: Advanced routing, canary deployments, failover
- **Security**: mTLS encryption, identity, authorization policies
- **Observability**: Metrics, logs, traces, service dependency graphs
- **Resilience**: Circuit breaking, fault injection, retries

## DNS in Kubernetes

Kubernetes provides a DNS service for service discovery that creates records for Services and Pods.

### DNS Records

- **Services**:
  - Regular Service: `<service-name>.<namespace>.svc.cluster.local`
  - Headless Service: Returns individual Pod IPs instead of Service IP

- **Pods**:
  - With StatefulSet: `<pod-name>.<service-name>.<namespace>.svc.cluster.local`
  - Regular Pods: IP address with dashes `<ip-with-dashes>.<namespace>.pod.cluster.local`

### DNS Policies

- **Default**: Inherit DNS configuration from the node
- **ClusterFirst**: Send cluster-suffix queries to cluster DNS server, others to upstream
- **ClusterFirstWithHostNet**: Same as ClusterFirst but for hostNetwork pods
- **None**: Ignore DNS settings from Kubernetes

## Examples

We provide several examples to demonstrate different networking aspects:

1. [Basic Networking](./examples/01-basic-networking.yaml) - Simple pod-to-pod communication
2. [Network Policies](./examples/02-network-policies.yaml) - Implementing security rules
3. [Basic Ingress](./examples/03-basic-ingress.yaml) - Simple host and path-based routing
4. [Advanced Ingress](./examples/04-advanced-ingress.yaml) - Complex routing, rate limiting, and authentication
5. [External Services](./examples/05-external-services.yaml) - Connecting to services outside the cluster
6. [Multi-Cluster Networking](./examples/06-multi-cluster.yaml) - Communication across clusters

## Best Practices

### Service Design

- Use meaningful service names that reflect their function
- Group related services in the same namespace
- Prefer ClusterIP for internal services
- Use ExternalName for abstracting external services
- Consider headless services for StatefulSets

### Network Policies

- Implement a "deny all, allow specific" approach
- Create policies at namespace and application levels
- Label pods consistently to enable effective network policies
- Test policies thoroughly before applying in production
- Use network policy logging for debugging (if available)

### Ingress Management

- Use a consistent naming convention for Ingress resources
- Implement TLS for all production Ingress resources
- Configure appropriate health checks
- Set reasonable timeouts and connection limits
- Use annotations for controller-specific optimizations
- Consider Ingress API Gateway patterns for microservices

### Performance

- Choose the right CNI plugin for your needs
- Monitor network performance metrics
- Optimize service connection pooling
- Consider session affinity for stateful applications
- Implement proper DNS caching at pod level

### Security

- Encrypt traffic with TLS/mTLS where appropriate
- Implement network policies for all production workloads
- Use private endpoints where possible
- Consider a service mesh for zero-trust security
- Regularly audit network flows

## Troubleshooting

### Common Issues

#### Pod Connectivity Issues

- Check if pods are on the same or different nodes
- Verify network policy allows the traffic
- Check service DNS resolution
- Inspect CNI configuration
- Look for routing or firewall issues

```bash
# Test connectivity from a pod
kubectl exec -it <pod-name> -- curl <service-name>
kubectl exec -it <pod-name> -- nslookup <service-name>
```

#### Service Discovery Problems

- Check if service selectors match pod labels
- Verify endpoints are created
- Test DNS resolution

```bash
kubectl get endpoints <service-name>
kubectl exec -it <pod-name> -- nslookup <service-name>.<namespace>.svc.cluster.local
```

#### Ingress Issues

- Verify Ingress controller is running
- Check Ingress resource configuration
- Look at Ingress controller logs
- Verify DNS resolves to the right load balancer/IP
- Check TLS secrets are correctly created

```bash
kubectl get ingress
kubectl describe ingress <ingress-name>
kubectl logs -n <ingress-namespace> <ingress-controller-pod>
```

## Further Reading

- [Kubernetes Networking Documentation](https://kubernetes.io/docs/concepts/services-networking/)
- [Ingress Documentation](https://kubernetes.io/docs/concepts/services-networking/ingress/)
- [Service Documentation](https://kubernetes.io/docs/concepts/services-networking/service/)
- [Network Policy Documentation](https://kubernetes.io/docs/concepts/services-networking/network-policies/)
- [CNI Specification](https://github.com/containernetworking/cni/blob/master/SPEC.md)

## Next Steps

- Learn about [Resource Management](../11-resource-management/) for CPU and memory control
- Explore [Custom Resources](../12-custom-resources/) to extend Kubernetes API
- Study [Autoscaling](../13-autoscaling/) for automatic scaling of workloads 