# Kubernetes Architecture

## 🌟 Overview

Kubernetes has a distributed architecture that provides flexibility, scalability, and resilience. This guide explains the main components of a Kubernetes cluster and how they work together to orchestrate containerized applications.

## 🏗️ The Big Picture

A Kubernetes cluster consists of two main types of machines:

1. **Control Plane Node(s)**: The brain of the cluster, responsible for maintaining the desired state
2. **Worker Nodes**: Where your applications actually run

```
┌────────────────────────────────────────────────────────────────┐
│                      Kubernetes Cluster                         │
│                                                                │
│  ┌──────────────────────────────┐                              │
│  │      Control Plane Node      │                              │
│  │                              │                              │
│  │   ┌──────────┐ ┌──────────┐  │    ┌───────────────────┐    │
│  │   │  API     │ │Scheduler │  │    │   Worker Node 1   │    │
│  │   │  Server  │ │          │  │    │                   │    │
│  │   └──────────┘ └──────────┘  │    │  ┌─────┐ ┌─────┐  │    │
│  │                              │    │  │Pod 1│ │Pod 2│  │    │
│  │   ┌──────────┐ ┌──────────┐  │    │  └─────┘ └─────┘  │    │
│  │   │Controller│ │  etcd    │  │    │                   │    │
│  │   │ Manager  │ │          │  │    │  ┌───────────┐    │    │
│  │   └──────────┘ └──────────┘  │    │  │  Kubelet  │    │    │
│  │                              │    │  └───────────┘    │    │
│  └──────────────────────────────┘    │                   │    │
│                                      │  ┌───────────┐    │    │
│                                      │  │Kube-proxy │    │    │
│                                      │  └───────────┘    │    │
│                                      └───────────────────┘    │
│                                                                │
│                                      ┌───────────────────┐    │
│                                      │   Worker Node 2   │    │
│                                      │                   │    │
│                                      │  ┌─────┐ ┌─────┐  │    │
│                                      │  │Pod 3│ │Pod 4│  │    │
│                                      │  └─────┘ └─────┘  │    │
│                                      │                   │    │
│                                      │  ┌───────────┐    │    │
│                                      │  │  Kubelet  │    │    │
│                                      │  └───────────┘    │    │
│                                      │                   │    │
│                                      │  ┌───────────┐    │    │
│                                      │  │Kube-proxy │    │    │
│                                      │  └───────────┘    │    │
│                                      └───────────────────┘    │
└────────────────────────────────────────────────────────────────┘
```

## 🎮 Control Plane Components

The control plane is like the brain of Kubernetes, making global decisions about the cluster and detecting and responding to events.

### 1. kube-apiserver

**Function**: The front-end for the Kubernetes control plane, exposing the Kubernetes API.

**In simple terms**: It's like the receptionist at a hotel - all requests to view or change the state of the cluster go through here.

**Key responsibilities**:
- Validates and processes API requests
- Serves as the gateway for tools like kubectl
- Handles authentication and authorization
- RESTful interface for all cluster operations

**How to interact with it**:
```bash
# Get API server information
kubectl get --raw /api/v1
```

### 2. etcd

**Function**: Consistent and highly-available key-value store used to store all cluster data.

**In simple terms**: It's like the cluster's memory, storing the configuration and state of the entire cluster.

**Key characteristics**:
- Distributed database with strong consistency
- Stores cluster state and configuration
- Uses Raft consensus algorithm for reliability
- Critical component requiring backup for disaster recovery

**Backing up etcd**:
```bash
# Backup etcd data
ETCDCTL_API=3 etcdctl snapshot save snapshot.db
```

### 3. kube-scheduler

**Function**: Watches for newly created Pods with no assigned node, and selects a node for them to run on.

**In simple terms**: It's like an apartment manager, deciding which tenant (Pod) goes into which apartment (Node).

**Decision factors**:
- Resource requirements
- Hardware/software constraints
- Affinity/anti-affinity specifications
- Data locality
- Deadlines

**Scheduling process visualization**:
```
 ┌─────────────┐      ┌─────────────┐      ┌─────────────┐
 │  New Pod    │─────▶│   Filter    │─────▶│   Score     │
 │  Created    │      │   Nodes     │      │   Nodes     │
 └─────────────┘      └─────────────┘      └─────────────┘
                                                  │
                                                  ▼
                                           ┌─────────────┐
                                           │  Assign Pod │
                                           │   to Node   │
                                           └─────────────┘
```

### 4. kube-controller-manager

**Function**: Runs controller processes that regulate the state of the cluster.

**In simple terms**: It's like multiple department managers, each responsible for a different aspect of keeping the cluster running smoothly.

**Key controllers**:
- **Node Controller**: Notices and responds when nodes go down
- **Job Controller**: Watches for one-time task jobs and creates pods to run them
- **Endpoints Controller**: Populates the Endpoints object (joins Services & Pods)
- **Service Account & Token Controllers**: Create default accounts and API access tokens

**Controller loop concept**:
```
         ┌───────────────┐
         │ Desired State │
         └───────────────┘
                  │
                  ▼
┌─────────────────────────────────┐
│       Observe Current State     │◀─────┐
└─────────────────────────────────┘      │
                  │                       │
                  ▼                       │
┌─────────────────────────────────┐      │
│  Determine Differences (delta)  │      │
└─────────────────────────────────┘      │
                  │                       │
                  ▼                       │
┌─────────────────────────────────┐      │
│ Take Action To Reconcile States │──────┘
└─────────────────────────────────┘
```

### 5. cloud-controller-manager (optional)

**Function**: Interfaces with the underlying cloud provider's API.

**In simple terms**: It's like a translator between Kubernetes and the specific cloud provider (AWS, GCP, Azure, etc.).

**Responsibilities**:
- Node controller (for checking cloud provider about deleted nodes)
- Route controller (for setting up routes in cloud infrastructure)
- Service controller (for creating, updating, and deleting cloud provider load balancers)

## 👷 Worker Node Components

Worker nodes are the machines where your containerized applications actually run.

### 1. kubelet

**Function**: An agent that runs on each node, ensuring containers are running in a Pod.

**In simple terms**: It's like a diligent worker who makes sure that the containers described in PodSpecs are running and healthy.

**Key responsibilities**:
- Communicates with the API server
- Mounts volumes to the pod
- Downloads secrets
- Passes container information to the container runtime
- Reports node and pod status back to the master

**Viewing kubelet status**:
```bash
# On the node itself
systemctl status kubelet

# From another machine using kubectl
kubectl get nodes
kubectl describe node <node-name>
```

### 2. kube-proxy

**Function**: Maintains network rules on nodes to allow network communication to your Pods.

**In simple terms**: It's like a network plumber, setting up pipes so your applications can talk to each other and the outside world.

**Implementation modes**:
- **iptables mode** (default): Uses Linux iptables rules
- **IPVS mode**: For clusters with lots of services, more efficient
- **Userspace mode** (legacy): Early implementation, less efficient

**Network traffic flow with kube-proxy**:
```
                    ┌──────────────┐
                    │   Service    │
                    │ (Virtual IP) │
                    └──────────────┘
                           │
                           ▼
                    ┌──────────────┐
                    │  kube-proxy  │
                    │  (iptables)  │
                    └──────────────┘
                           │
                  ┌────────┴────────┐
                  ▼                  ▼
          ┌──────────────┐   ┌──────────────┐
          │     Pod      │   │     Pod      │
          │  (Backend 1) │   │  (Backend 2) │
          └──────────────┘   └──────────────┘
```

### 3. Container Runtime

**Function**: Software responsible for running containers.

**In simple terms**: It's the engine that actually runs your containers.

**Popular container runtimes**:
- **containerd**: Lightweight, focused on running containers
- **CRI-O**: Lightweight implementation of Container Runtime Interface
- **Docker Engine**: Common but being phased out by Kubernetes

**Container runtime interface (CRI)**:
```
┌─────────────────┐     ┌─────────────────────────┐
│     kubelet     │────▶│   Container Runtime     │
│                 │◀────│   Interface (CRI)       │
└─────────────────┘     └─────────────────────────┘
                                     │
             ┌─────────────────────┬─┴───────────────────────┐
             ▼                     ▼                         ▼
  ┌─────────────────┐    ┌─────────────────┐       ┌─────────────────┐
  │   containerd    │    │     CRI-O       │       │  Other CRI      │
  │                 │    │                 │       │  Implementations │
  └─────────────────┘    └─────────────────┘       └─────────────────┘
```

## 🧩 Kubernetes Add-ons

Add-ons extend the functionality of Kubernetes. Here are some essential ones:

### DNS

**Function**: Provides DNS records for Kubernetes services.

**Implementation**: Usually CoreDNS, a flexible DNS server.

**Why it matters**: Allows pods to locate services by name rather than IP address.

### Dashboard

**Function**: Web-based UI for Kubernetes cluster management.

**Key features**:
- Visualize applications running on your cluster
- Create and modify resources
- View logs and troubleshoot issues

### Network Plugin

**Function**: Implements the Kubernetes networking model.

**Popular options**:
- **Calico**: Good performance and security policies
- **Flannel**: Simple overlay network
- **Cilium**: Advanced networking and security with eBPF
- **Weave Net**: Resilient networking with DNS

## 🔄 How It All Works Together

Let's follow what happens when you run `kubectl create deployment nginx --image=nginx`:

1. **kubectl** sends a request to the **API Server**
2. **API Server** validates the request and persists it to **etcd**
3. **Controller Manager** notices a new deployment and creates a ReplicaSet
4. **Scheduler** assigns the new pods to worker nodes
5. **Kubelet** on the selected node receives the pod specs
6. **Kubelet** tells the **Container Runtime** to pull and run the nginx image
7. **Kubelet** monitors the pods and reports back to the **API Server**
8. **Kube-proxy** updates network rules to allow traffic to the pods

## 🔍 High Availability Control Plane

In production environments, the control plane is often replicated for high availability:

```
┌─────────────────────────────────────────────────────────────┐
│                  High Availability Cluster                   │
│                                                             │
│  ┌────────────────┐  ┌────────────────┐  ┌────────────────┐ │
│  │  Control Plane │  │  Control Plane │  │  Control Plane │ │
│  │     Node 1     │  │     Node 2     │  │     Node 3     │ │
│  └────────────────┘  └────────────────┘  └────────────────┘ │
│           │                  │                  │            │
│           └──────────────────┼──────────────────┘            │
│                              │                               │
│                      ┌───────────────┐                       │
│                      │  Load Balancer│                       │
│                      └───────────────┘                       │
│                              │                               │
│           ┌──────────────────┼──────────────────┐            │
│           │                  │                  │            │
│  ┌────────────────┐  ┌────────────────┐  ┌────────────────┐ │
│  │  Worker Node 1 │  │  Worker Node 2 │  │  Worker Node 3 │ │
│  └────────────────┘  └────────────────┘  └────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

Key points for HA setup:
- Multiple control plane nodes with synchronized etcd
- Load balancer in front of API servers
- etcd can be run as part of the control plane or externally

## 📝 Important Terminology

- **Node**: A worker machine in Kubernetes
- **Pod**: The smallest deployable unit, containing one or more containers
- **Service**: An abstraction that defines a logical set of Pods and a policy to access them
- **Namespace**: Virtual cluster within a physical cluster
- **Volume**: Directory accessible to containers in a pod
- **Deployment**: Manages the desired state for Pods and ReplicaSets
- **StatefulSet**: Manages stateful applications
- **DaemonSet**: Ensures specific Pods run on all (or some) Nodes
- **Job/CronJob**: Run tasks that complete (rather than running continuously)

## 🧪 Practical Examples

### Check Cluster Components

```bash
# List all nodes in the cluster
kubectl get nodes

# Check control plane components
kubectl get pods -n kube-system

# View detailed information about a node
kubectl describe node <node-name>

# Check API server logs (if running in pods)
kubectl logs -n kube-system kube-apiserver-<control-plane-node-name>

# Check system component status
kubectl get componentstatuses
```

### Verify Cluster Health

```bash
# Check overall cluster info
kubectl cluster-info

# Check health of all namespaces
kubectl get all --all-namespaces

# Perform more detailed cluster validation
kubectl get events --sort-by='.lastTimestamp'
```

## 📚 Additional Resources

- [Kubernetes Components Official Documentation](https://kubernetes.io/docs/concepts/overview/components/)
- [Kubernetes Architecture Explained](https://kubernetes.io/docs/concepts/architecture/)
- [Kubernetes The Hard Way](https://github.com/kelseyhightower/kubernetes-the-hard-way) - A tutorial that walks through setting up a Kubernetes cluster from scratch
- [Interactive Kubernetes Learning Environment](https://www.katacoda.com/courses/kubernetes) 