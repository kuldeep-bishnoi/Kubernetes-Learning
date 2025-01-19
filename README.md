# Kubernetes Learning

## Day 1: Docker Fundamentals

Docker is a platform that enables developers to build, ship, and run applications in containers. Containers are lightweight, portable, and self-sufficient units that can run anywhere, making them ideal for microservices and distributed applications.

### Understanding Containers vs. Virtual Machines
- **Containers**: Share the host OS kernel, making them lightweight and fast to start.
- **Virtual Machines**: Include a full OS, making them heavier and slower to start.
- **Analogy**: Containers are like apartments in a building, sharing the same infrastructure, while VMs are like separate houses.

### Challenges with Non-Containerized Applications
- Dependency conflicts
- Inconsistent environments
- Difficult scaling

### How Docker Solves These Challenges
- **Isolation**: Ensures applications run in their own environments.
- **Portability**: Run anywhere, from a developer's laptop to production.
- **Efficiency**: Share resources efficiently, reducing overhead.

### Docker Architecture
- **Docker Engine**: Core component that runs containers.
- **Docker Images**: Read-only templates used to create containers.
- **Docker Containers**: Running instances of Docker images.

### Task 1
- Create Docker architecture and workflow diagrams.
- Write a blog post about your learnings.

## Day 2: Dockerize an Application

Dockerizing an application involves creating a Dockerfile, building a Docker image, and running it as a container. This process ensures consistency across different environments.

### Steps to Dockerize a Project
1. **Clone a Sample Repository**: Use a sample project or your own.
2. **Create a Dockerfile**: Define the environment and commands to run the application.
3. **Build the Docker Image**: Use `docker build` to create an image from the Dockerfile.
4. **Run the Docker Container**: Use `docker run` to start the application in a container.
5. **Push to Docker Hub**: Share your image with others by pushing it to a repository.

### Additional Concepts
- **Docker Init**: A command to quickly set up a Docker project.
- **Docker Logs**: View logs from running containers for debugging.

### Task 2
- Dockerize a project and document the process.
- Explore the `docker init` command.

## Day 3: Docker Multi-Stage Builds

Multi-stage builds in Docker allow you to use multiple `FROM` statements in a Dockerfile to optimize the build process. This technique helps in reducing the size of the final image by copying only the necessary artifacts from intermediate stages.

### Example Dockerfile for Multi-Stage Builds

```Dockerfile
# Stage 1: Build the application
FROM node:14 AS builder
WORKDIR /app
COPY package.json ./
RUN npm install
COPY . .
RUN npm run build

# Stage 2: Serve the application
FROM nginx:alpine
COPY --from=builder /app/build /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

This Dockerfile uses two stages:
- **Builder Stage**: Uses a Node.js image to install dependencies and build the application.
- **Final Stage**: Uses an Nginx image to serve the built application, copying only the necessary files from the builder stage.

### Benefits of Multi-Stage Builds
- **Reduced Image Size**: Only the necessary files are included in the final image.
- **Improved Security**: By excluding build tools and dependencies from the final image.
- **Simplified Dockerfile**: Easier to manage and maintain.

### Steps to Implement Multi-Stage Builds
1. **Create a Multi-Stage Dockerfile**: Use multiple `FROM` statements to separate build and runtime environments.
2. **Build the Docker Image**: Use `docker build` to create an image from the Dockerfile.
3. **Run the Docker Container**: Use `docker run` to start the application in a container.
4. **Push to Docker Hub**: Share your image with others by pushing it to a repository.

### Additional Concepts
- **Docker Inspect**: View detailed information about Docker containers.
- **Docker Image Cleanup**: Remove unused images to free up space.

### Task 3
- Dockerize a project using multi-stage builds and document the process.
- Explore Docker best practices for writing Dockerfiles.

## Day 4: Why Kubernetes?

Kubernetes is an open-source platform designed to automate deploying, scaling, and operating application containers. It addresses many challenges associated with managing standalone containers.

### Challenges with Standalone Containers
- **Scalability**: Difficult to manage and scale multiple containers manually.
- **Load Balancing**: Requires manual configuration to distribute traffic.
- **High Availability**: Ensuring uptime and redundancy is complex.
- **Resource Management**: Inefficient use of resources without orchestration.

### How Kubernetes Solves These Challenges
- **Automated Scaling**: Automatically adjusts the number of running containers based on demand.
- **Service Discovery and Load Balancing**: Distributes network traffic to ensure stability.
- **Self-Healing**: Restarts failed containers and replaces them as needed.
- **Efficient Resource Utilization**: Schedules containers based on resource requirements.

### Use Cases for Kubernetes
- **Microservices Architecture**: Ideal for managing microservices-based applications.
- **CI/CD Pipelines**: Streamlines the deployment process.
- **Hybrid Cloud Environments**: Manages workloads across different cloud providers.
- **Batch Processing**: Efficiently handles batch jobs and workloads.
- **Dev/Test Environments**: Quickly spin up and tear down environments.

### When Not to Use Kubernetes
- **Simple Applications**: Overhead may not be justified for small apps.
- **Limited Resources**: Requires significant resources to run efficiently.
- **Short-Lived Applications**: Not ideal for apps with very short lifecycles.
- **Single Node Deployments**: Better suited for multi-node clusters.
- **Lack of Expertise**: Requires a learning curve and expertise to manage.

### Task 4
- Document the challenges of using standalone containers and how Kubernetes solves them.
- List use cases for when to use or not use Kubernetes.

## Day 5: Kubernetes Architecture

Kubernetes is a powerful container orchestration platform that automates the deployment, scaling, and management of containerized applications. Understanding its architecture is crucial for effective management.

### Key Components of Kubernetes Architecture
- **Control Plane**: Manages the Kubernetes cluster and includes components like the API Server, Scheduler, Controller Manager, etcd, and Cloud Controller Manager.
- **Worker Nodes**: Run the containerized applications and include components like the Kubelet, Kube Proxy, and Container Runtime Interface (CRI).
- **Addons**: Provide additional functionalities to the cluster, such as DNS, Dashboard, and Monitoring.

### Control Plane Components
- **API Server**: Acts as the front end for the Kubernetes control plane, handling RESTful requests.
- **Scheduler**: Assigns workloads to nodes based on resource availability and other constraints.
- **Controller Manager**: Ensures the desired state of the cluster by managing controllers.
- **etcd**: A distributed key-value store that holds the cluster state and configuration.
- **Cloud Controller Manager**: Integrates with cloud providers to manage cloud resources.

### Node Components
- **Kubelet**: An agent that runs on each node, ensuring containers are running as expected.
- **Kube Proxy**: Manages network communication between pods and services.
- **Container Runtime Interface (CRI)**: Provides a common interface for container runtimes like Docker, rkt, and cri-o.

### Addons
- **DNS**: Provides DNS resolution for services and pods.
- **Dashboard**: Offers a web-based interface for cluster management.
- **Monitoring**: Collects and displays cluster metrics and logs.

### Task 5
- Create Kubernetes architecture and end-to-end flow diagrams.
- Document the functions of each control plane component and addon.

## Day 6: Install Kubernetes Cluster Locally

Setting up a local Kubernetes cluster is a great way to practice and experiment with Kubernetes without needing cloud resources. Kind (Kubernetes IN Docker) is a tool that allows you to run Kubernetes clusters locally using Docker containers as nodes.

### Steps to Install a Kind Cluster
1. **Install a Single-Node Cluster**: Follow the Kind documentation to set up a single-node cluster with Kubernetes version 1.29.
2. **Delete the Cluster**: Use Kind commands to delete the cluster when done.
3. **Install a Multi-Node Cluster**: Set up a cluster named `cka-cluster2` with 1 control plane and 3 worker nodes using Kubernetes version 1.30.
4. **Install Kubectl**: Set up the `kubectl` client to interact with the cluster.
5. **Verify the Setup**: Use `kubectl get nodes` to ensure all nodes are ready and `docker ps` to verify nodes are running as containers.

### Additional Concepts
- **Kubernetes Context**: Switch between different clusters using `kubectl config use-context`.
- **Docker as Nodes**: Understand that Kind uses Docker containers to simulate Kubernetes nodes.

### Task 6
- Install and configure single-node and multi-node Kind clusters.

## Day 7: Pods in Kubernetes

Pods are the smallest deployable units in Kubernetes, encapsulating one or more containers. Understanding how to create and manage pods is fundamental to working with Kubernetes.

### Creating Kubernetes Objects
- **Imperative Method**: Directly create objects using commands or API calls.
- **Declarative Method**: Define objects in YAML or JSON files and apply them to the cluster.

### Sample Pod YAML
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    env: demo
    type: frontend
spec:
  containers:
  - name: nginx-container
    image: nginx
    ports:
    - containerPort: 80
```

### Task 7
- Create a pod using the imperative command with the nginx image.
- Generate a YAML file from the created pod and modify it to create a new pod.
- Troubleshoot and fix errors in a provided YAML configuration.

## Day 8: Replicasets and Deployments in Kubernetes

ReplicaSets and Deployments are key components in Kubernetes for managing the lifecycle of pods. They ensure that the desired number of pod replicas are running at all times.

### ReplicaSets
- **Purpose**: Maintain a stable set of replica pods running at any given time.
- **Use Case**: Ideal for stateless applications where the number of replicas can be easily scaled.

### Deployments
- **Purpose**: Provide declarative updates to applications, allowing for easy rollouts and rollbacks.
- **Features**: Supports rolling updates, scaling, and rollback to previous versions.

### Task 8
- Create and manage ReplicaSets and Deployments using the nginx image.
- Update images, scale replicas, and manage rollout history.
- Troubleshoot and fix issues in provided YAML configurations.

## Day 9: Services in Kubernetes

Services in Kubernetes provide a stable endpoint for applications, allowing communication between different components within the cluster and external access to applications. They are essential for microservices architecture and are often used in conjunction with Deployments and ReplicaSets.

### Types of Services
- **ClusterIP**: Exposes the service on a cluster-internal IP, making it accessible only within the cluster. This is the default service type.
- **NodePort**: Exposes the service on each node's IP at a static port, allowing external access. This is useful for development and testing.
- **LoadBalancer**: Provisions a load balancer for the service, typically used in cloud environments. This is ideal for production environments.
- **ExternalName**: Maps a service to an external DNS name. This is useful for integrating with external services.
- **Headless**: Does not assign a cluster-internal IP to the service, allowing direct communication with the pods. This is useful for service discovery in a multi-cluster environment.

### Sample YAML for Services
- **ClusterIP**:
  ```yaml
  apiVersion: v1
  kind: Service
  metadata:
    name: cluster-svc
    labels:
      env: demo
  spec:
    ports:
    - port: 80
      selector:
        env: demo
  ```
- **NodePort**:
  ```yaml
  apiVersion: v1
  kind: Service
  metadata:
    name: nodeport-svc
    labels:
      env: demo
  spec:
    type: NodePort
    ports:
    - nodePort: 30001
      port: 80
      targetPort: 80
    selector:
      env: demo
  ```
- **LoadBalancer**:
  ```yaml
  apiVersion: v1
  kind: Service
  metadata:
    name: loadbalancer-svc
    labels:
      env: demo
  spec:
    type: LoadBalancer
    ports:
    - port: 80
      targetPort: 80
    selector:
      env: demo
  ```
- **ExternalName**:
  ```yaml
  apiVersion: v1
  kind: Service
  metadata:
    name: externalname-svc
    labels:
      env: demo
  spec:
    type: ExternalName
    externalName: my.database.example.com
  ```
- **Headless**:
  ```yaml
  apiVersion: v1
  kind: Service
  metadata:
    name: headless-svc
    labels:
      env: demo
  spec:
    clusterIP: None
    ports:
    - port: 80
      targetPort: 80
    selector:
      env: demo
  ```

### Task 9
- Create a Deployment and expose it using ClusterIP and NodePort services.
- Demonstrate the differences between service types and discuss their use cases.
- Explore the use of Ingress for more advanced routing and load balancing.
- Discuss the limitations of NodePort and the benefits of using a LoadBalancer in a cloud environment.
- Consider the use of ExternalName for integrating with external services.
- Discuss the use of Headless Services for service discovery in a multi-cluster environment.
- Discuss the use of Service Mesh for advanced traffic management and security.

## Day 10: Namespaces in Kubernetes

Namespaces in Kubernetes provide a mechanism for isolating groups of resources within a single cluster. They are useful for organizing resources by environment, team, or application.

### Benefits of Using Namespaces
- **Resource Isolation**: Prevents accidental modification or deletion of resources.
- **Organizational Structure**: Allows resources to be grouped by type, environment, or domain.
- **Access Control**: Facilitates role-based access control (RBAC) by namespace.

### Task 10
- Create two namespaces, `ns1` and `ns2`, and deploy applications within them.
- Explore pod-to-pod and service-to-service communication across namespaces.
- Use Fully Qualified Domain Names (FQDN) for cross-namespace service access.

## Day 11: Multi-Container Pods in Kubernetes

Multi-container pods in Kubernetes allow you to run multiple containers within a single pod, sharing the same network and storage resources. This setup is useful for closely related processes that need to communicate or share data.

### Types of Multi-Container Pods
- **Sidecar Containers**: Extend and enhance the functionality of the main application container, often used for logging, monitoring, or proxying.
- **Init Containers**: Run before the main application container to perform setup tasks, such as waiting for dependencies to be ready.

### Sample YAML for Multi-Container Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app.kubernetes.io/name: MyApp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    env:
    - name: FIRSTNAME
      value: "kuldeep"
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox:1.28
    command: ['sh', '-c']
    args: ['until nslookup myservice.default.svc.cluster.local; do echo waiting for myservice; sleep 2; done']
```

### Task 11
- Create a multi-container pod using sidecar and init containers.
- Explore the use of environment variables within the pod.
- Understand the lifecycle of multi-container pods and how they interact with each other.

## Day 12: DaemonSets, Jobs, and CronJobs in Kubernetes

DaemonSets, Jobs, and CronJobs are Kubernetes objects that manage the lifecycle of pods in different ways, each serving specific use cases. These objects are essential for automating tasks, ensuring system-level applications run on each node, and scheduling jobs to run at specific times or intervals.

### DaemonSets
- **Purpose**: Ensure that a pod runs on each node in the cluster.
- **Use Case**: Deploy system-level applications like kube-proxy, monitoring agents, and network plugins.
- **Key Features**:
  - **Node Selection**: DaemonSets can be configured to run on specific nodes or node groups using node selectors.
  - **Rolling Updates**: Supports rolling updates for the pods, ensuring minimal downtime during updates.
  - **Self-Healing**: Automatically restarts or replaces pods that fail or are terminated.

### Sample YAML for DaemonSet
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: nginx-ds
  labels:
    env: demo
spec:
  selector:
    matchLabels:
      env: demo
  template:
    metadata:
      labels:
        env: demo
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
```

### Jobs
- **Purpose**: Run a batch of tasks to completion.
- **Use Case**: Execute one-time tasks like data processing, backups, or migrations.
- **Key Features**:
  - **Batch Processing**: Jobs are designed for batch processing, ensuring all tasks are completed before considering the job successful.
  - **Restart Policy**: Supports various restart policies, including "OnFailure" and "Never", to handle task failures.
  - **Parallelism**: Allows for parallel execution of tasks to speed up processing.

### Sample YAML for Job
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: example-job
spec:
  completions: 3
  parallelism: 3
  template:
    spec:
      containers:
      - name: pi
        image: perl
        command: ["perl"]
        args: ["-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: "OnFailure"
  backoffLimit: 4
```

### CronJobs
- **Purpose**: Schedule jobs to run at specific times or intervals.
- **Use Case**: Automate tasks like backups, report generation, and periodic data processing.

### Task 12
- Create a DaemonSet and a CronJob in Kubernetes.
- Understand cron syntax and schedule a CronJob to print "40daysofkubernetes" every 5 minutes.

## Day 13: Static Pods, Labels, and Selectors in Kubernetes

Static pods, labels, and selectors are crucial concepts in Kubernetes for managing and organizing resources efficiently. Understanding these concepts is essential for effective cluster management and resource allocation.

### Labels and Selectors
- **Labels**: Key-value pairs attached to Kubernetes objects for organization, grouping, and filtering purposes. Labels are used to categorize objects based on their characteristics, such as environment, application, or tier.
- **Selectors**: Filter objects based on labels, useful for querying and managing subsets of resources. Selectors are used in deployments, services, and other resources to target specific pods or nodes.

### Static Pods
- **Purpose**: Managed directly by the kubelet on each node, not through the Kubernetes API server. This approach bypasses the scheduler and allows for direct pod management on specific nodes.
- **Use Case**: Deploy critical system components like the API server, scheduler, and controller manager. Static pods are ideal for deploying components that require direct node management or have specific node requirements.

### Manual Pod Scheduling
- **nodeName Field**: Assigns a pod to a specific node, bypassing the scheduler. This field is used in pod definitions to specify the node where the pod should run. Manual pod scheduling is useful for scenarios where specific node requirements are necessary, such as GPU or high-performance computing.

### Task 13
- **Manually Schedule a Pod without the Scheduler**: Create a pod definition with the `nodeName` field to assign it to a specific node, ensuring the pod runs on that node regardless of the scheduler's decisions.
- **Manage Static Pods by Accessing the Control Plane Node**: Directly manage static pods on the control plane node by accessing the node's configuration files. This approach is useful for managing critical system components that require direct node management.
- **Create and Filter Pods using Labels and Selectors**: Use labels to categorize pods and selectors to filter and manage subsets of pods. This approach enables efficient pod management and scaling based on specific criteria.

### Additional Information
- **Label Selectors in Services**: Services use label selectors to identify the pods they should target. This ensures that services are always connected to the correct pods, even if pods are replaced or updated.
- **Label Selectors in Deployments**: Deployments use label selectors to identify the pods they manage. This allows deployments to scale, update, or roll back pods based on specific criteria.
- **Node Selectors**: Node selectors are used in pod definitions to specify the node requirements, such as node labels or taints. This ensures that pods are scheduled on nodes that meet the specified requirements.

## Day 14: Taints and Tolerations in Kubernetes

Taints and tolerations in Kubernetes are mechanisms to control pod scheduling on nodes, allowing you to define rules for where pods can and cannot run.

### Taints
- **Purpose**: Repel pods from nodes by marking nodes with specific characteristics.
- **Example**: `kubectl taint nodes node1 key=gpu:NoSchedule` prevents pods without a matching toleration from being scheduled on `node1`.

### Tolerations
- **Purpose**: Allow pods to be scheduled on nodes with matching taints.
- **Example**: A pod with a toleration for `gpu=true:NoSchedule` can be scheduled on nodes with the corresponding taint.

### Task 14
- Apply taints to worker nodes and create pods with matching tolerations.
- Manage taints on the control plane node and observe pod scheduling behavior.

## Day 15: Node Affinity in Kubernetes

Node affinity in Kubernetes provides advanced capabilities for controlling pod scheduling based on node labels, offering more flexibility and granularity than node selectors.

### Node Affinity
- **Purpose**: Define complex rules for pod placement based on node labels.
- **Types**:
  - **requiredDuringSchedulingIgnoredDuringExecution**: Strict rules that must be met for pod scheduling.
  - **preferredDuringSchedulingIgnoredDuringExecution**: Soft rules that influence pod placement but are not mandatory.

### Example YAML for Node Affinity
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: redis-3
spec:
  containers:
  - image: redis
    name: redis
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: disktype
            operator: In
            values:
            - ssd
```

### Task 15
- Create pods with node affinity and manage node labels to control scheduling.

## Day 16: Resource Requests and Limits in Kubernetes

Resource requests and limits in Kubernetes are crucial for managing resource allocation and ensuring that pods have the necessary resources to run effectively. This is particularly important in a multi-tenant cluster where multiple pods compete for resources.

### Resource Requests
- **Purpose**: Specify the minimum resources a pod needs to operate smoothly. This ensures that the pod is scheduled on a node that can provide the requested resources.
- **Example**: A pod with a memory request of `100Mi` is guaranteed to have at least `100Mi` of memory available. This is essential for ensuring that the pod has enough memory to run without being terminated due to out-of-memory errors.
- **Impact on Scheduling**: Resource requests influence the scheduling of pods. The scheduler ensures that pods are placed on nodes that can satisfy their resource requests. This prevents pods from being scheduled on nodes that do not have sufficient resources, which could lead to pod failures.

### Resource Limits
- **Purpose**: Define the maximum resources a pod can use, preventing it from consuming more than its fair share. This is essential for preventing a single pod from monopolizing resources and starving other pods.
- **Example**: A pod with a memory limit of `200Mi` cannot use more than `200Mi` of memory. If the pod attempts to use more memory, the kernel will terminate it to prevent memory exhaustion.
- **Impact on Pod Execution**: Resource limits directly impact the execution of pods. If a pod exceeds its resource limits, it may be terminated or have its resources throttled. This ensures that pods do not consume excessive resources and compromise the stability of the cluster.

### Example YAML for Resource Requests and Limits
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: memory-demo
  namespace: mem-example
spec:
  containers:
  - name: memory-demo-ctr
    image: polinux/stress
    resources:
      requests:
        memory: "100Mi"
      limits:
        memory: "200Mi"
    command: ["stress"]
    args: ["--vm", "1", "--vm-bytes", "150M", "--vm-hang", "1"]
```

### Task 16
- **Create a namespace and configure resource requests and limits for pods**: This involves creating a namespace for organizing pods and defining resource requests and limits for each pod to ensure they have the necessary resources to run effectively.
- **Install a metrics server and monitor resource usage**: This involves setting up a metrics server to collect resource usage data from pods and nodes. This data can be used to monitor resource utilization, identify bottlenecks, and optimize resource allocation.
- **Additional Best Practices**:
  - **Monitor resource usage regularly**: Regularly monitor resource usage to identify trends and patterns. This helps in optimizing resource allocation and preventing resource bottlenecks.
  - **Set realistic resource requests and limits**: Ensure that resource requests and limits are set realistically based on the pod's requirements. This prevents over-allocation or under-allocation of resources.
  - **Use resource quotas**: Implement resource quotas to limit the total amount of resources that can be consumed by pods in a namespace. This prevents a single namespace from monopolizing resources and ensures fair resource allocation across namespaces.

## Day 17: Autoscaling in Kubernetes

Autoscaling in Kubernetes allows for dynamic adjustment of resources based on demand, ensuring efficient resource utilization and application performance. This feature is particularly useful in environments where the workload is variable or unpredictable, as it enables the cluster to scale up or down to match the changing demands.

### Horizontal Pod Autoscaler (HPA)
- **Purpose**: Automatically scales the number of pod replicas based on observed CPU utilization or other select metrics. This ensures that the application can handle changes in traffic or workload without manual intervention.
- **Example Command**: `kubectl autoscale deployment php-apache --cpu-percent=50 --min=1 --max=10`
- **How it Works**: HPA periodically queries the resource utilization of the pods it manages and compares it to the target utilization specified in the configuration. If the current utilization exceeds the target, HPA increases the number of replicas to reduce the utilization. Conversely, if the utilization is below the target, HPA decreases the number of replicas to free up resources.

### Vertical Pod Autoscaler (VPA)
- **Purpose**: Automatically adjusts the resource requests and limits of containers in a pod based on observed usage. This ensures that pods are allocated the optimal amount of resources, preventing under-allocation or over-allocation.
- **How it Works**: VPA analyzes the historical resource usage of pods and adjusts their resource requests and limits accordingly. This process is done in three phases: recommendation, update, and apply. The recommendation phase determines the optimal resource allocation, the update phase updates the pod's configuration, and the apply phase applies the changes to the pod.

### Example YAML for HPA
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: php-apache
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: php-apache
  minReplicas: 1
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
```

### Task 17
- Set up autoscaling using HPA and VPA in Kubernetes.
- Follow the steps and commands provided in the README file.

## Day 18: Health Probes in Kubernetes

Health probes in Kubernetes are used to monitor the health of applications and take necessary actions to ensure high availability and self-healing.

### Types of Health Probes
- **Readiness Probe**: Ensures the application is ready to serve traffic.
- **Liveness Probe**: Restarts the application if health checks fail.
- **Startup Probe**: Used for legacy applications that need more time to start.

### Types of Health Checks
- **HTTP**: Checks the response of an HTTP request.
- **TCP**: Checks the ability to establish a TCP connection.
- **Command**: Executes a command inside the container.

### Example YAML for Health Probes
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hello
spec:
  containers:
  - name: liveness
    image: registry.k8s.io/e2e-test-images/agnhost:2.40
    args:
    - liveness
    livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
      initialDelaySeconds: 3
      periodSeconds: 3
    readinessProbe:
      httpGet:
        path: /healthz
        port: 8080
      initialDelaySeconds: 15
      periodSeconds: 10
```

### Task 18
- Create pods with liveness and readiness probes to monitor application health.

## Day 19: ConfigMaps and Secrets in Kubernetes

ConfigMaps and Secrets in Kubernetes are used to manage configuration data and sensitive information, respectively, outside of the pod manifest.

### ConfigMaps
- **Purpose**: Store configuration data as key-value pairs, allowing for easy management and reuse across multiple pods.
- **Example Command**: `kubectl create cm <configmapname> --from-literal=color=blue --from-literal=color=red`

### Secrets
- **Purpose**: Securely store sensitive information, such as passwords and API keys, and inject them into pods as environment variables or volume mounts.
- **Example Command**: `kubectl create secret generic <secretname> --from-literal=username=myuser --from-literal=password=mypassword`

### Example YAML for ConfigMap
```yaml
apiVersion: v1
data:
  firstname: kuldeep
  lastname: bishnoi
kind: ConfigMap
metadata:
  name: app-cm
```

### Example YAML for Secret
```yaml
apiVersion: v1
data:
  username: bXl1c2Vy
  password: bXlwYXNzd29yZA==
kind: Secret
metadata:
  name: app-secret
type: Opaque
```

### Task 19
- Explore ConfigMaps, Secrets, and environment variables in Kubernetes.
- Perform tasks related to securely distributing credentials using Secrets.

## Day 20: SSL/TLS in Kubernetes

SSL (Secure Sockets Layer) and TLS (Transport Layer Security) are cryptographic protocols designed to provide secure communication over a computer network. TLS is the successor to SSL and is widely used for secure communication between clients and servers.

### Key Concepts of SSL/TLS
- **Encryption**: Protects data by converting it into a secure format that can only be read by someone with the decryption key. This ensures that even if data is intercepted, it cannot be read or tampered with.
- **Authentication**: Verifies the identity of the parties involved in the communication. This ensures that the client is communicating with the intended server and not an imposter.
- **Integrity**: Ensures that the data has not been altered during transmission. This ensures that the data received is the same as the data sent, without any tampering or modification.

### How SSL/TLS Works
1. **Handshake Process**: Establishes a secure connection by exchanging keys and agreeing on encryption methods. This process involves the client and server exchanging information to establish a secure connection.
2. **Data Encryption**: Uses symmetric encryption for fast and secure data transmission. Symmetric encryption uses the same key for both encryption and decryption.
3. **Certificate Verification**: Uses digital certificates to authenticate the identity of the server and, optionally, the client. Digital certificates are issued by a trusted Certificate Authority (CA) and contain information about the identity of the server or client.

### SSL/TLS in Kubernetes
In Kubernetes, SSL/TLS is used to secure communication between components, such as between the API server and clients, and between pods. This ensures that all communication within the cluster is encrypted and authenticated.

### Task 20
- Understand the concepts of SSL/TLS and create diagrams to illustrate them.
- Implement SSL/TLS in a Kubernetes cluster to secure communication between components.
- Explore the use of SSL/TLS in Kubernetes for securing communication between the API server and clients, and between pods.

## Day 21: Managing TLS Certificates in Kubernetes

TLS certificates in Kubernetes are used to secure communication between components and authenticate users and services.

### Key Concepts
- **Client-Server Model**: TLS certificates are used to secure communication between clients and servers in a Kubernetes cluster.
- **Certificate Management**: Involves generating keys, creating CertificateSigningRequests (CSRs), and managing certificate approval and renewal.

### Commands for Certificate Management
- **Generate a Key File**: `openssl genrsa -out learner.key 2048`
- **Generate a CSR File**: `openssl req -new -key learner.key -out learner.csr -subj "/CN=learner"`
- **Approve a CSR**: `kubectl certificate approve <certificate-signing-request-name>`

### Task 21
- Generate a private key and CSR, create a CertificateSigningRequest, and manage certificate approval.

## Day 22: Kubernetes Authentication and Authorization

Authentication and authorization in Kubernetes are essential for securing access to the cluster and managing permissions. This ensures that only authorized users and services can interact with the cluster and its resources.

### Authentication: Verifying Identity
- **Kubeconfig**: Acts as a keycard containing certificates to identify users to the Kubernetes API server. The kubeconfig file is typically stored in the `~/.kube/config` directory.
- **API Calls**: Use `kubectl` commands to interact with the cluster, typically relying on the local kubeconfig file for authentication. For example, `kubectl get pods` to list all pods in the default namespace.

### Authorization: Managing Access
- **Node Authorizer**: Ensures nodes are authorized to communicate with the API server. This is crucial for ensuring that only trusted nodes can join the cluster.
- **ABAC**: Attribute-Based Access Control, associates users with permissions but can be complex to implement and manage.
- **RBAC**: Role-Based Access Control, the recommended method for managing permissions by creating roles and assigning users or groups. RBAC is more scalable and easier to manage than ABAC.
- **Webhooks**: Optional, for complex authorization logic using external tools. Webhooks allow for custom authorization logic to be implemented using external services.

### Authorization Modes
- Configurable in the API server, with a priority sequence for practical use. The authorization modes can be configured in the following order:
  1. Node
  2. ABAC
  3. RBAC
  4. Webhook

### Setting up RBAC in Kubernetes
To set up RBAC in Kubernetes, follow these steps:

1. **Create a Role**: Define a role that specifies the permissions for a particular resource. For example, create a role that allows read-only access to pods in the default namespace.
```yml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
rules:
- apiGroups: ["*"]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
```
2. **Create a RoleBinding**: Bind the role to a user or group. For example, bind the `pod-reader` role to the `developer` user.
```yml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: pod-reader-binding
roleRef:
  name: pod-reader
  kind: Role
subjects:
- name: developer
  kind: User
```
3. **Apply the Role and RoleBinding**: Apply the role and rolebinding configurations to the cluster using `kubectl apply`.
```bash
kubectl apply -f pod-reader.yaml
kubectl apply -f pod-reader-binding.yaml
```
4. **Test the Role**: Test the role by using the `kubectl` command to verify that the user has the correct permissions.
```bash
kubectl auth can-i get pods --as=developer
```
This should return `yes` if the user has the correct permissions.

### Task 22
- Explore authentication and authorization methods in Kubernetes.
- Configure RBAC roles and test access levels.
- Set up RBAC in a Kubernetes cluster and test the permissions.


## Day 23: Kubernetes Role-Based Access Control (RBAC)

RBAC in Kubernetes is a method for managing permissions and access to resources within a cluster. It allows administrators to define roles that specify the actions a user can perform on a resource, and then bind those roles to users or groups. This ensures that users only have access to the resources they need to perform their tasks, reducing the risk of unauthorized access or malicious activity.

### Key Concepts
- **Roles**: Define a set of permissions for accessing resources. Roles can be defined for a specific namespace or cluster-wide.
- **RoleBindings**: Associate users or groups with roles to grant permissions. RoleBindings can be used to bind roles to users, groups, or service accounts.
- **ClusterRoles**: Similar to Roles, but apply cluster-wide.
- **ClusterRoleBindings**: Bind ClusterRoles to users, groups, or service accounts, granting cluster-wide permissions.

### Commands
- **Generate CSR Key and File**: Use `openssl` to create keys and CSR files for authentication.
- **Check Identity**: `kubectl auth whoami` to verify your identity.
- **Verify Access**: `kubectl auth can-i` to check if you have access to specific resources.
- **Create Role**: `kubectl create role` to define a new role.
- **Create RoleBinding**: `kubectl create rolebinding` to bind a role to a user or group.
- **Create ClusterRole**: `kubectl create clusterrole` to define a new cluster-wide role.
- **Create ClusterRoleBinding**: `kubectl create clusterrolebinding` to bind a cluster-wide role to a user or group.

### Sample YAML
- **Role**: Defines permissions for resources like pods.
- **RoleBinding**: Associates users with roles to grant permissions.
- **ClusterRole**: Defines cluster-wide permissions for resources like nodes.
- **ClusterRoleBinding**: Associates users with cluster-wide roles to grant permissions.

### Task 23
- Explore RBAC by creating roles, role bindings, cluster roles, and cluster role bindings in Kubernetes.
- Test access levels and permissions using `kubectl auth can-i`.
- Implement RBAC to manage access to resources in a Kubernetes cluster.
- Understand the differences between Roles and ClusterRoles, and RoleBindings and ClusterRoleBindings.
- Learn how to use RBAC to implement least privilege access, ensuring users and services only have access to the resources they need.

## Day 24: Kubernetes ClusterRoles and ClusterRoleBindings

ClusterRoles and ClusterRoleBindings in Kubernetes extend Role-Based Access Control (RBAC) to manage permissions across the entire cluster. This allows for more comprehensive management of cluster-wide resources, ensuring that users or groups have the necessary permissions to perform tasks without compromising security.

### Key Concepts
- **ClusterRoles**: Define a set of permissions that apply across all namespaces within a cluster, enabling the management of cluster-wide resources such as nodes, persistent volumes, and cluster-scoped objects like ClusterRoles and ClusterRoleBindings themselves.
- **ClusterRoleBindings**: Associate users or groups with ClusterRoles, granting them cluster-wide permissions. This binding is not limited to a specific namespace, unlike RoleBindings.

### Commands
- **Create ClusterRole**: `kubectl create clusterrole node-reader --verb=get,list,watch --resource=nodes` - This command creates a ClusterRole named `node-reader` that grants permissions to get, list, and watch nodes.
- **Create ClusterRoleBinding**: `kubectl create clusterrolebinding node-reader-binding --clusterrole=node-reader --user=adam` - This command binds the `node-reader` ClusterRole to a user named `adam`, granting them cluster-wide permissions to read nodes.

### Key Points
- **Namespace Scope vs. Cluster Scope**: Roles are namespace-scoped, meaning they only apply to a specific namespace, whereas ClusterRoles are cluster-wide, applying to all namespaces within the cluster.
- **RoleBindings vs. ClusterRoleBindings**: Use RoleBindings to assign permissions to users or groups within a specific namespace, and ClusterRoleBindings to assign cluster-wide permissions.
- **Permission Management**: It is crucial to carefully manage permissions to ensure security and prevent unauthorized access to cluster resources.

### Task 24
- Explore ClusterRoles and ClusterRoleBindings by creating and managing them in Kubernetes.
- Test cluster-wide permissions and access levels to ensure they are functioning as intended.
- Implement ClusterRoles and ClusterRoleBindings to manage access to cluster-wide resources in a Kubernetes cluster.
- Understand the differences between Roles and ClusterRoles, as well as RoleBindings and ClusterRoleBindings, to effectively manage permissions in a cluster.
- Learn how to use ClusterRoles and ClusterRoleBindings to implement least privilege access, ensuring users and services only have access to the resources they need to perform their tasks.

## Day 25: Kubernetes Service Accounts

Service accounts in Kubernetes are used by applications, bots, or Kubernetes components to interact with other services, distinct from user accounts used by admins and developers. They provide a way for these entities to authenticate and access the Kubernetes API, ensuring secure and controlled interactions within the cluster.

### Key Concepts
- **Service Accounts**: Allow applications and components to authenticate and interact with the Kubernetes API, enabling them to perform actions such as creating or managing resources.
- **Default Service Accounts**: Automatically created in each namespace, such as `kube-system` and `kube-node-lease`, to facilitate interactions between components and the API.
- **Service Account Tokens**: Tokens generated for service accounts, which are used for authentication and can be rotated or updated as needed.
- **Service Account Secrets**: Secrets created for service accounts, which contain the service account token and are used for authentication.

### Commands
- **Create Service Account**: `kubectl create sa <name>` - This command creates a new service account with the specified name.
- **List Service Accounts**: `kubectl get sa` - This command lists all service accounts in the current namespace.
- **View Service Account Details**: `kubectl describe sa <name>` - This command provides detailed information about a specific service account.
- **Create Service Account Token**: `kubectl create token <service-account-name>` - This command generates a token for a service account, which can be used for authentication.
- **View Service Account Token**: `kubectl get secret <service-account-name>-token -o yaml` - This command displays the token for a service account.

### Task 25
- Create and manage service accounts in Kubernetes to enable secure interactions between applications and the API.
- Assign roles and role bindings to service accounts to grant access to specific resources and actions, ensuring least privilege access.
- Rotate service account tokens regularly to maintain security and prevent unauthorized access.
- Use service accounts to authenticate and authorize interactions between components and the Kubernetes API, ensuring a secure and controlled environment.

## Day 26: Kubernetes Network Policies

Network policies in Kubernetes allow you to control the inbound and outbound traffic to and from the cluster, enhancing security and traffic management. This is particularly important in a multi-tenant environment where multiple applications share the same cluster resources.

### Key Concepts
- **Network Policies**: Define rules to allow or deny traffic between pods and services. These policies are applied at the network layer (L3/L4) and can be based on pod labels, namespaces, and IP addresses.
- **Deny-All Policy**: Restricts all incoming traffic to the cluster, ensuring that no unauthorized access is allowed. This is a common starting point for network policy configuration.
- **Allow Policy**: Permits specific services to be accessed by certain pods on specific ports. This policy is used to selectively allow traffic to specific services or pods.
- **Ingress and Egress**: Network policies can control both ingress (incoming) and egress (outgoing) traffic, enabling fine-grained control over network communication within the cluster.

### Manifests
- **Kind Cluster YAML**: Configuration for setting up a cluster with specific networking settings, including network policies. This YAML file defines the cluster's network architecture and security settings.
- **Calico Installation**: Use Calico for advanced network policy management. Calico is a popular network policy engine that provides a flexible and scalable way to manage network policies in Kubernetes clusters.

### Task 26
- Implement network policies to control traffic in a Kubernetes cluster, ensuring that only authorized traffic is allowed.
- Use Calico to manage and enforce network policies, providing a robust and scalable network policy management solution.
- Explore other network policy engines, such as Cilium and Weave Net, to understand their features and use cases.
- Consider the importance of network policies in a multi-tenant environment and how they can help prevent lateral movement in case of a security breach.
- Document the process of implementing network policies and the benefits they bring to cluster security and traffic management.

## Day 27: Setup a Multi-Node Kubernetes Cluster Using Kubeadm

Kubeadm is a tool used to bootstrap a Kubernetes cluster, installing control plane components and CLI tools.

### Key Concepts
- **Kubeadm**: Installs control plane components like ApiServer, ETCD, Controller Manager, and Scheduler, along with CLI tools such as `kubeadm`, `kubelet`, and `kubectl`.
- **Installation**: The demo involves setting up Kubernetes on VMs, with recommendations for using Multipass on Mac Silicon chips.
- **Network Configuration**: Configuring the network settings is crucial for the cluster to function properly.
- **Security Groups**: Security groups are used to manage network traffic and ensure the cluster's security.
- **VM Provisioning**: Setting up the VMs is the first step in creating a Kubernetes cluster.

### Setup Steps
- **Provision VMs**: Set up 1 Master and 2 Worker nodes.
- **Security Groups**: Create and attach security groups to manage network traffic.
- **Network Configuration**: Disable source destination check for AWS EC2 VMs.

### Task 27
- Set up a multi-node Kubernetes cluster using kubeadm.
- Configure network settings and security groups for the cluster.
- Document the setup process and any issues encountered.

## Day 28: Docker Volumes and Persistent Storage

Docker volumes and bind mounts are essential for managing persistent storage in Docker containers. This allows data to persist beyond the container lifecycle, ensuring that important data is not lost when a container is restarted or deleted.

### Key Concepts
- **Docker Volumes**: Provide a way to store data persistently, independent of the container's lifecycle. Volumes are directories that are shared between the host machine and a container. They are created and managed by Docker, making it easy to persist data even after a container is deleted.
- **Bind Mounts**: Allow a host directory or file to be mounted into a container, enabling data sharing between the host and the container. Bind mounts are useful for sharing configuration files or data between the host and the container.
- **tmpfs Mounts**: A type of mount that stores data in the host's RAM, providing a fast and temporary storage solution. tmpfs mounts are ideal for applications that require high-performance storage for temporary data.
- **Named Volumes**: A type of volume that is given a specific name, making it easy to manage and reuse volumes across different containers.
- **Anonymous Volumes**: A type of volume that is not given a specific name, making it ideal for temporary storage needs.

### Task 28
- Explore Docker volumes, bind mounts, tmpfs mounts, named volumes, and anonymous volumes to manage persistent storage.
- Implement persistent storage solutions for Docker containers using volumes and bind mounts.
- Understand the use cases for each type of volume and mount, and apply them in real-world scenarios.
- Document the process of setting up and managing persistent storage for Docker containers.

## Day 29: Kubernetes Persistent Volumes and Storage Classes

Kubernetes Persistent Volumes (PV), Persistent Volume Claims (PVC), and Storage Classes are essential components for managing storage resources in a cluster. They provide a way to dynamically provision and manage storage for applications running in the cluster.

### Key Concepts
- **Persistent Volumes (PV)**: Storage resources provisioned by an administrator or dynamically using Storage Classes. PVs are cluster resources that are independent of the pod lifecycle.
- **Persistent Volume Claims (PVC)**: Requests for storage by a user, fulfilled by a PV. PVCs are used to request storage resources for a pod.
- **Storage Classes**: Define different types of storage available in a cluster, allowing for dynamic provisioning of PVs. Storage Classes are used to define the characteristics of a storage resource, such as its type, size, and access modes.

### Example YAML
- **Non-Persistent Volume**: Uses `emptyDir` for temporary storage. This type of volume is deleted when the pod is terminated.
- **Persistent Volume**: Defines a storage resource with specific capacity and access modes. PVs can be provisioned statically or dynamically using Storage Classes.
- **Persistent Volume Claim**: Requests storage with specific access modes and storage class. PVCs are used to bind a pod to a PV.

### Task 29
- Explore Persistent Volumes, Persistent Volume Claims, and Storage Classes in Kubernetes to understand how they interact and are used to manage storage resources.
- Implement storage solutions using PVs, PVCs, and Storage Classes to dynamically provision and manage storage for applications running in the cluster.
- Understand the different types of Persistent Volumes, such as Local, NFS, and Ceph, and how they can be used to meet specific storage needs.
- Learn how to create and manage Storage Classes to define different types of storage available in the cluster.
- Practice creating Persistent Volume Claims to request storage resources for pods and understand how they are bound to Persistent Volumes.
- Investigate how to use StatefulSets to manage stateful applications that require persistent storage.
- Explore the use of CSI (Container Storage Interface) drivers to extend the storage capabilities of Kubernetes clusters.
- Consider the security implications of using Persistent Volumes and how to implement encryption and access controls for storage resources.
- Document the process of implementing and managing Persistent Volumes, Persistent Volume Claims, and Storage Classes in a Kubernetes cluster.

## Day 30: Understanding DNS (Domain Name System)

DNS (Domain Name System) is a hierarchical system that translates human-readable domain names into IP addresses, enabling computers to identify each other on the network. It is a crucial part of the internet infrastructure, allowing users to access websites and services using easy-to-remember domain names instead of difficult-to-remember IP addresses.

### Key Concepts
- **Domain Names**: Human-readable names for websites and services, like `www.example.com`. These names are used to identify specific websites, services, or organizations on the internet.
- **IP Addresses**: Numerical labels assigned to devices on a network, used for identification and communication. IP addresses are used by computers to locate and communicate with each other.
- **DNS Resolution**: The process of translating domain names into IP addresses. This process involves a series of steps, including querying DNS servers, caching, and recursive resolution.
- **DNS Servers**: Specialized computers that store DNS records and respond to DNS queries. There are different types of DNS servers, including recursive resolvers, authoritative name servers, and caching name servers.
- **DNS Records**: Entries in a DNS server that map a domain name to an IP address or other DNS records. Common DNS records include A records, CNAME records, MX records, and NS records.
- **DNS Query**: A request sent to a DNS server to resolve a domain name to an IP address. DNS queries can be recursive or non-recursive, depending on the type of DNS server handling the query.

### Task 30
- Explore the DNS system and understand how domain names are resolved into IP addresses.
- Learn about the different types of DNS records and their uses.
- Understand the role of DNS servers in the DNS resolution process.
- Investigate how DNS caching works and its impact on DNS resolution performance.
- Consider the security implications of DNS spoofing and how to mitigate them.
- Document the process of setting up and managing DNS for a domain or network.

## Day 31: Understanding CoreDNS in Kubernetes

CoreDNS is a flexible, extensible DNS server used as the default DNS provider in Kubernetes, handling DNS-based service discovery within the cluster. It is designed to be highly available, scalable, and easy to manage, making it an ideal choice for Kubernetes clusters.

### Key Concepts
- **CoreDNS**: Provides DNS services for Kubernetes clusters, enabling service discovery and name resolution. It supports a wide range of plugins, allowing for customization and extension of its functionality.
- **Service Discovery**: Allows services within the cluster to discover and communicate with each other using DNS names. This enables efficient and dynamic communication between services, facilitating the deployment of complex applications.
- **Plugin Architecture**: CoreDNS's plugin architecture allows for the addition of new features and functionality as needed. This makes it highly adaptable to changing DNS requirements within the cluster.
- **High Availability**: CoreDNS is designed to be highly available, ensuring that DNS services are always accessible within the cluster. This is critical for ensuring the reliability and uptime of applications running within the cluster.

### Task 31
- Explore CoreDNS and its role in Kubernetes for DNS-based service discovery.
- Understand the plugin architecture and how it enables customization and extension of CoreDNS.
- Investigate how CoreDNS ensures high availability and reliability of DNS services within the cluster.
- Consider the security implications of using CoreDNS and how to implement access controls and encryption for DNS services.

## Day 32: Kubernetes Networking and Container Network Interface (CNI)

The Container Network Interface (CNI) is a specification and set of libraries for configuring network interfaces in Linux containers, used by Kubernetes to manage networking for pods. CNI provides a standardized way to configure network interfaces for containers, ensuring consistent networking across different environments.

### Key Concepts
- **CNI**: Provides a standardized way to configure network interfaces for containers, ensuring consistent networking across different environments.
- **Kubernetes Networking**: Manages communication between pods, services, and external networks using CNI plugins. This includes pod-to-pod communication, pod-to-service communication, and pod-to-external-network communication.
- **CNI Plugins**: Implement the CNI specification to provide network connectivity to pods. Examples of CNI plugins include Calico, Flannel, and Weave Net.
- **Pod Networking**: Each pod gets its own IP address and can communicate with other pods and services within the cluster.
- **Service Networking**: Services provide a stable network identity and load balancing for accessing pods.
- **Network Policies**: Allow administrators to define network traffic rules for incoming and outgoing traffic to pods.

### Task 32
- Explore Kubernetes networking and the role of CNI in managing network interfaces for pods.

## Day 33: Kubernetes Ingress

Ingress in Kubernetes is a resource that manages external access to services within a cluster, typically handling HTTP and HTTPS routes. It acts as a single entry point for incoming traffic, allowing for load balancing, SSL termination, and name-based virtual hosting.

### Key Concepts
- **Ingress**: Provides a way to expose services to the external world, managing routing and access control. It is a collection of rules that allow inbound traffic to reach the cluster services.
- **Deployment and Service**: Deploy applications and create services to manage internal communication. Services provide a stable network identity and load balancing for accessing pods.
- **Ingress Controller**: A component that watches the Ingress resources and updates the load balancer configuration to reflect the changes. Popular Ingress controllers include NGINX, HAProxy, and Traefik.
- **Ingress Resource**: A Kubernetes resource that defines the rules for routing traffic to services.

### Steps
- **Build and Push Docker Image**: Create a Docker image and push it to a registry like Docker Hub. This step ensures the application is packaged and ready for deployment.
- **Create Deployment and Service**: Deploy the application using a Deployment and create a Service to manage internal access. This step ensures the application is running and accessible within the cluster.
- **Set Up Ingress**: Configure Ingress to expose the application externally. This step involves creating an Ingress resource that defines the rules for routing traffic to the service.
- **Deploy Ingress Controller**: Deploy an Ingress controller to watch the Ingress resources and update the load balancer configuration accordingly.

### Task 33
- Deploy a hello world application and expose it using Ingress. This task involves building and pushing a Docker image, creating a Deployment and Service, setting up Ingress, and deploying an Ingress controller.
- Explore different Ingress controllers and their configurations.
- Understand how to use annotations to customize Ingress behavior.
- Learn how to use Ingress to manage SSL certificates and HTTPS traffic.

## Day 34: Upgrading a Multi-Node Kubernetes Cluster with Kubeadm

Upgrading a Kubernetes cluster using `kubeadm` involves updating the master and worker nodes, one minor version at a time.

### Key Concepts
- **Upgrade Process**: Upgrade the master node first, followed by the worker nodes.
- **Version Management**: Ensure components are upgraded to compatible versions.
- **Backup and Restore**: Essential for disaster recovery, ensuring data integrity and cluster availability.

### Steps
- **Update Kubeadm**: Use `apt-get` to update `kubeadm` to the latest version.
- **Plan Upgrade**: Use `kubeadm upgrade plan` to view available versions.
- **Backup ETCD**: Use `etcdctl` to backup the ETCD data before the upgrade.
- **Apply Upgrade**: Use `kubeadm upgrade apply` to upgrade system components.
- **Restore ETCD**: Use `etcdctl` to restore the ETCD data if the upgrade fails.

### Task 34
- Perform a step-by-step upgrade of a multi-node Kubernetes cluster using `kubeadm`.
- Explore the process of backing up and restoring ETCD in a Kubernetes cluster.
- Understand the importance of disaster recovery and maintaining cluster state.

## Day 35: Kubernetes ETCD Backup and Restore

ETCD is a distributed key-value store used by Kubernetes to store all cluster data, including configuration, state, and network policies. Backing up and restoring ETCD is crucial for disaster recovery, ensuring data integrity, and maintaining cluster availability. This process is essential for maintaining the reliability and resilience of the cluster.

### Key Concepts
- **ETCD**: A distributed key-value store that stores all Kubernetes cluster data, including configuration, state, and network policies.
- **Backup and Restore**: Essential for disaster recovery, ensuring data integrity, and maintaining cluster availability. This process involves creating snapshots of the ETCD data and restoring them in case of data loss or corruption.

### Steps for ETCD Backup and Restore
- **Backup**: Use `etcdctl` to create a snapshot of the ETCD data. This snapshot can be stored in a secure location for later use.
- **Restore**: Use `etcdctl` to restore the ETCD data from the snapshot. This process involves replacing the existing ETCD data with the data from the snapshot.

### Importance of ETCD Backup and Restore
- **Disaster Recovery**: ETCD backup and restore ensure that the cluster can recover from data loss or corruption, minimizing downtime and ensuring business continuity.
- **Data Integrity**: Regular backups ensure that the cluster data remains consistent and accurate, even in the event of data loss or corruption.
- **Cluster Availability**: ETCD backup and restore ensure that the cluster remains available and functional, even in the event of data loss or corruption.

### Task 35
- Explore the process of backing up and restoring ETCD in a Kubernetes cluster.
- Understand the importance of ETCD backup and restore for disaster recovery, data integrity, and cluster availability.
- Implement a regular backup and restore process for ETCD data to ensure cluster reliability and resilience.

## Day 36: Monitoring and Logging in Kubernetes

Monitoring and logging are essential for maintaining the health and performance of a Kubernetes cluster, providing insights into resource usage and application behavior.

### Key Concepts
- **Metrics Server**: Collects resource metrics from nodes and pods, enabling monitoring and scaling.
- **Logging**: Captures logs from applications and system components for analysis and troubleshooting.

### Task 36
- Install the Metrics Server and explore monitoring and logging in Kubernetes.
- Understand the importance of monitoring and logging for cluster health and performance.
- Explore different monitoring and logging tools and their configurations.
- Learn how to use annotations to customize monitoring and logging behavior.
- Understand how to use monitoring and logging to manage cluster resources and troubleshoot issues.

## Day 37: Troubleshooting Application Failures in Kubernetes

Troubleshooting application failures in Kubernetes involves identifying and resolving issues that prevent applications from running smoothly. This process is crucial for ensuring the reliability and availability of applications in a Kubernetes cluster.

### Key Concepts
- **Application Troubleshooting**: Involves diagnosing and fixing errors in application deployments and configurations. This includes analyzing logs, checking pod status, and verifying network connectivity.
- **Common Issues**: Include misconfigurations, resource constraints, network connectivity problems, and image issues. Misconfigurations can occur in YAML files, while resource constraints can lead to pod evictions. Network connectivity problems can prevent pods from communicating with each other, and image issues can cause pods to fail during deployment.
- **Troubleshooting Tools**: Kubernetes provides various tools to aid in troubleshooting, such as `kubectl describe`, `kubectl logs`, and `kubectl exec`. These tools allow you to inspect pod and container status, view logs, and execute commands inside running containers.

### Task 37
- Clone the example voting app repository and apply the Kubernetes manifests to deploy the application.
- Intentionally introduce errors in the application configuration or deployment to simulate real-world troubleshooting scenarios.
- Use Kubernetes troubleshooting tools to identify and fix the errors, practicing the process of diagnosing and resolving application failures in a Kubernetes environment.
- Document the troubleshooting process, including the errors introduced, the steps taken to identify them, and the solutions implemented to fix them. This documentation will serve as a reference for future troubleshooting exercises.

## Day 38: Troubleshooting Control Plane Failures in Kubernetes

Troubleshooting control plane failures in Kubernetes involves diagnosing and resolving issues with critical components that manage the cluster. The control plane is responsible for maintaining the desired state of the cluster, and any failures can have significant impacts on cluster functionality and availability.

### Key Concepts
- **Control Plane Components**: The control plane consists of the following essential components:
  - **API Server**: Acts as the front end for the Kubernetes control plane, handling RESTful requests.
  - **etcd**: A distributed key-value store that holds the cluster state and configuration.
  - **Controller Manager**: Ensures the desired state of the cluster by managing controllers.
  - **Scheduler**: Assigns workloads to nodes based on resource availability and other constraints.
- **Common Issues**: Control plane failures can be caused by a variety of issues, including:
  - **Component Crashes**: Unexpected termination of control plane components can lead to cluster instability.
  - **Configuration Errors**: Misconfigured components or incorrect cluster state can cause control plane failures.
  - **Network Connectivity Problems**: Loss of network connectivity between control plane components or between the control plane and worker nodes can disrupt cluster operation.
  - **Resource Constraints**: Insufficient resources, such as CPU or memory, can cause control plane components to fail or become unresponsive.
  - **Software Issues**: Bugs or incompatibilities in the Kubernetes software or its dependencies can lead to control plane failures.

### Task 38
- Follow the video and documentation to troubleshoot control plane failures in a Kubernetes cluster.
- Practice identifying and resolving common issues affecting the control plane components.
- Develop a systematic approach to troubleshooting, including:
  - Gathering information about the failure using tools like `kubectl` and system logs.
  - Isolating the problem to a specific component or area of the cluster.
  - Applying fixes or workarounds to restore cluster functionality.
  - Validating the resolution and ensuring the cluster is stable and functional.
- Consider the following best practices for control plane maintenance and troubleshooting:
  - Regularly back up etcd data to ensure cluster state can be restored in case of a failure.
  - Implement monitoring and alerting for control plane components to quickly identify issues.
  - Perform regular software updates and security patches to prevent known vulnerabilities.
  - Ensure resource availability and plan for capacity upgrades as the cluster grows.
  - Establish a disaster recovery plan to minimize downtime in case of a control plane failure.

## Day 39: Troubleshooting Worker Node Failures in Kubernetes

Troubleshooting worker node failures in Kubernetes involves diagnosing and resolving issues that affect the nodes responsible for running application workloads. This process is crucial for ensuring the reliability and availability of applications in a Kubernetes cluster.

### Key Concepts
- **Worker Nodes**: Run application workloads and are essential for cluster operation. They are responsible for executing the tasks assigned by the control plane.
- **Common Issues**: May involve node crashes, resource exhaustion, or network connectivity problems. These issues can be caused by a variety of factors, including hardware or software failures, misconfigurations, or resource constraints.

### Task 39
- Follow the video and task instructions to troubleshoot worker node failures in a Kubernetes cluster. This includes:
  - Identifying the symptoms of a node failure, such as pods not being scheduled or node status indicating a problem.
  - Gathering information about the failure using tools like `kubectl describe node`, `kubectl logs`, and system logs.
  - Isolating the problem to a specific component or area of the node, such as a network interface or a container runtime.
  - Applying fixes or workarounds to restore node functionality, such as restarting a service or updating a configuration.
  - Validating the resolution and ensuring the node is stable and functional.
  - Documenting the troubleshooting process and the steps taken to resolve the issue for future reference.

Additionally, it is essential to have a comprehensive understanding of the Kubernetes cluster's architecture, including the control plane and worker nodes, as well as the tools and techniques used for troubleshooting. This knowledge will aid in identifying and resolving node failures efficiently and effectively.

## Day 40: JSONPath and Advanced Kubectl Commands

JSONPath and advanced kubectl commands are essential tools for querying and manipulating Kubernetes resources, offering flexibility and precision in managing clusters. These tools enable administrators to efficiently manage and troubleshoot their clusters by providing a way to extract specific data from JSON documents and perform complex operations on Kubernetes resources.

### Key Concepts
- **JSONPath**: A query language for extracting data from JSON documents, used in kubectl to filter and format output. JSONPath allows administrators to specify a path to a specific part of a JSON document, making it easier to extract the required information.
- **Advanced Kubectl Commands**: Enable complex queries and operations on Kubernetes resources, enhancing cluster management. Advanced kubectl commands provide a range of options for filtering, sorting, and formatting output, making it easier to manage and troubleshoot Kubernetes resources.

### Task 40
- Understand JSON fundamentals and JSONPath usage in kubectl commands to effectively query and manipulate Kubernetes resources.
- Write advanced kubectl commands to sort and filter Kubernetes resources based on specific criteria, such as resource type, status, or labels.
- Practice using JSONPath expressions to extract specific data from Kubernetes resources, such as pod names or container statuses.
- Explore the use of advanced kubectl commands, such as `kubectl get` with `-o jsonpath`, to extract and format data from Kubernetes resources.
- Learn how to use kubectl plugins, such as `kubectl tree`, to visualize and manage Kubernetes resources in a more efficient way.
- Familiarize yourself with the kubectl command-line tool and its various options, including `--path`, `--query`, and `--output`, to effectively manage and troubleshoot Kubernetes resources.

## Day 41: CKA Exam Preparation Tips and Strategies

Preparing for the Certified Kubernetes Administrator (CKA) exam involves understanding the exam format, environment, and effective strategies for success. It is essential to have a comprehensive understanding of Kubernetes concepts, including cluster architecture, deployment management, and troubleshooting.

### Key Concepts
- **Exam Preparation**: Register for the exam, utilize study materials, and practice with sample tests to assess your knowledge and identify areas for improvement.
- **Exam Environment**: Familiarize yourself with the virtual machine setup and available resources, including the command-line interface and the `kubectl` tool.
- **Exam Strategy**: Prioritize tasks based on difficulty and time consumption, focusing on the most critical and time-sensitive tasks first.

### Pro Tips
- Use the pre-configured `kubectl` alias `k` for efficiency and speed in executing commands.
- Leverage bash auto-completion and `vi` editor shortcuts for faster command execution and navigation.
- Practice with sample questions and scenarios to simulate the exam environment and build your problem-solving skills.
- Focus on understanding the underlying concepts and principles of Kubernetes, rather than just memorizing commands and procedures.
- Develop a systematic approach to troubleshooting, including identifying symptoms, gathering information, isolating problems, and applying fixes.

### Task 41
- Review the exam preparation tips and strategies for the CKA exam, including key concepts, pro tips, and a systematic approach to troubleshooting.
- Practice with sample questions and scenarios to build your skills and confidence in managing Kubernetes clusters.
- Develop a study plan that covers all aspects of the exam, including cluster architecture, deployment management, and troubleshooting.
- Join online communities and forums to connect with other exam candidates, share knowledge, and learn from their experiences.

## Day 42: Deploying a Docker Images Registry on Kubernetes

Deploying a local Docker images registry on Kubernetes involves setting up a secure and scalable environment for storing and managing Docker images. This process is crucial for ensuring the efficient management of Docker images within a Kubernetes cluster.

### Key Concepts
- **Cluster Provisioning**: Set up a regional Kubernetes cluster with nodes across different availability zones to ensure high availability and scalability.
- **Sample Deployment**: Deploy the Docker registry with replicas, ensuring data persistence and resource management. This includes setting up a StatefulSet to manage the registry's state and a Deployment for the registry's frontend.
- **Storage**: Use Persistent Volumes (PVs) to store registry data, ensuring data survives pod restarts. This includes setting up a Persistent Volume Claim (PVC) to dynamically provision storage.
- **Container Configuration**: Expose the registry on port 5000 and configure health checks and resource limits to ensure the registry is running efficiently and within defined resource constraints.
- **Init Container**: Ensure the data directory is present using an Alpine-based image to initialize the registry's data directory.
- **Service Exposure**: Use a load balancer to expose the registry externally, making it accessible to other services within the cluster.
- **Network Policies**: Implement security measures to restrict network traffic to and from the registry, ensuring only authorized access.
- **Secrets and ConfigMap**: Manage sensitive information such as registry credentials and configuration settings using Secrets and ConfigMaps.
- **Backup and Recovery**: Define strategies for data backup and recovery to ensure business continuity in case of data loss or corruption.

### Additional Considerations
- **Security**: Implement additional security measures such as SSL/TLS certificates for encryption, role-based access control (RBAC) for authorization, and network policies for isolation.
- **Monitoring and Logging**: Set up monitoring and logging tools such as Prometheus and Grafana to monitor the registry's performance and logs for debugging purposes.
- **Scalability**: Ensure the registry is scalable to handle increased load and demand, including horizontal pod autoscaling and vertical pod autoscaling.
- **High Availability**: Implement high availability measures such as self-healing and automatic restarts to ensure the registry is always available.

### Task 42
- Deploy a Docker images registry on Kubernetes following the outlined requirements, including additional considerations for security, monitoring, scalability, and high availability.
- Document the deployment process, including the configuration files and commands used.
- Test the registry to ensure it is functioning as expected, including pushing and pulling images.
