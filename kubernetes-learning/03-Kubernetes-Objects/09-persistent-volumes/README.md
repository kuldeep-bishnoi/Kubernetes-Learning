# Kubernetes Persistent Volumes

Persistent Volumes (PVs) provide a way to store data that outlives the lifecycle of a Pod. This document explains how Kubernetes handles storage resources and how to use them effectively.

![Kubernetes Persistent Volumes](https://miro.medium.com/max/1400/1*kF57zE9a5YCzhILHdmuRvQ.png)

## Table of Contents
- [Introduction to Persistent Storage](#introduction-to-persistent-storage)
- [Key Concepts](#key-concepts)
- [Persistent Volume Types](#persistent-volume-types)
- [Access Modes](#access-modes)
- [Reclaim Policies](#reclaim-policies)
- [Storage Classes](#storage-classes)
- [Volume Snapshots](#volume-snapshots)
- [Examples](#examples)
- [Best Practices](#best-practices)
- [Troubleshooting](#troubleshooting)
- [Further Reading](#further-reading)

## Introduction to Persistent Storage

In Kubernetes, applications often need to store and access data in a way that persists beyond the lifecycle of individual Pods. Unlike ephemeral storage (which is lost when a Pod terminates), persistent storage remains available even when Pods are rescheduled, restarted, or deleted.

Kubernetes provides a robust storage abstraction system through three main resources:
- **Persistent Volumes (PVs)**: Storage in the cluster provisioned by an administrator or dynamically provisioned
- **Persistent Volume Claims (PVCs)**: A user's request for storage
- **Storage Classes**: Defines how storage is provisioned dynamically

## Key Concepts

### Persistent Volume (PV)
A piece of storage in the cluster that has been provisioned by an administrator or dynamically provisioned using Storage Classes. PVs have a lifecycle independent of any Pod that uses them.

### Persistent Volume Claim (PVC)
A request for storage by a user. Claims can request specific size, access modes, or storage class. PVCs consume PV resources.

### Storage Class
Provides a way for administrators to describe the "classes" of storage they offer. Different classes might map to quality-of-service levels, backup policies, or arbitrary policies determined by the cluster administrators.

### Volume Binding Mode
Controls when volume binding and dynamic provisioning occurs. Options include:
- `Immediate`: Volume is bound or provisioned immediately
- `WaitForFirstConsumer`: Volume is bound or provisioned once a Pod using the PVC is created

### Dynamic Provisioning
Allows storage volumes to be created on-demand. Without dynamic provisioning, cluster administrators must manually make calls to their cloud or storage provider to create new storage volumes.

## Persistent Volume Types

Kubernetes supports many types of persistent volumes. Some of the most common include:

| Type | Description | Use Cases |
|------|-------------|-----------|
| `awsElasticBlockStore` | AWS EBS Volume | Applications on AWS |
| `azureDisk` | Azure Disk | Applications on Azure |
| `gcePersistentDisk` | Google Compute Engine Disk | Applications on GCP |
| `nfs` | Network File System | Shared read-write access |
| `hostPath` | Directory on node | Development/testing only |
| `local` | Local disk/partition/directory | High-performance storage with node affinity |
| `csi` | Container Storage Interface | Modern extensible storage interface |

### Static vs Dynamic Provisioning

**Static Provisioning**:
- Administrator creates PVs in advance
- Users create PVCs that bind to existing PVs
- Good for specific, pre-planned storage needs

**Dynamic Provisioning**:
- Uses StorageClasses to define how volumes are created
- PVs are created automatically when a PVC is created
- Streamlines storage provisioning and reduces manual intervention

## Access Modes

Persistent volumes can be mounted with different access modes:

| Access Mode | Description |
|-------------|-------------|
| `ReadWriteOnce` (RWO) | Volume can be mounted as read-write by a single node |
| `ReadOnlyMany` (ROX) | Volume can be mounted as read-only by many nodes |
| `ReadWriteMany` (RWX) | Volume can be mounted as read-write by many nodes |
| `ReadWriteOncePod` (RWOP) | Volume can be mounted as read-write by a single pod (K8s v1.22+) |

> Not all volume types support all access modes. Check your storage provider's documentation.

## Reclaim Policies

When a PVC is deleted, the PV it was using can be handled in different ways:

| Policy | Description |
|--------|-------------|
| `Retain` | PV is kept but considered "released." It cannot be used by a new PVC until manually reclaimed. |
| `Delete` | PV is automatically deleted. |
| `Recycle` (deprecated) | Basic scrubbing of data is performed before making it available again. |

## Storage Classes

Storage classes enable dynamic provisioning of PVs. A StorageClass provides a way to describe the "classes" of storage offered by the administrator.

Key attributes of a StorageClass:
- **Provisioner**: Which volume plugin to use (e.g., kubernetes.io/aws-ebs)
- **Parameters**: Specific to the provisioner (e.g., type: gp2 for AWS EBS)
- **reclaimPolicy**: What happens to volumes when released
- **volumeBindingMode**: When the binding and provisioning occurs
- **allowVolumeExpansion**: Whether PVCs can be expanded

### Default Storage Class

Clusters can define a default StorageClass. If a PVC doesn't specify a StorageClass, the default is used.

## Volume Snapshots

Kubernetes supports volume snapshots, allowing point-in-time copies of volumes:
- VolumeSnapshotClass: Defines the "class" for snapshots
- VolumeSnapshot: Represents a snapshot of a volume
- VolumeSnapshotContent: Represents the actual snapshot

Snapshots can be used to:
- Create backups
- Create new volumes from snapshots
- Migrate data between clusters
- Clone volumes for testing

## Examples

We provide several examples to demonstrate different aspects of persistent storage:

1. [Basic PV and PVC Example](./examples/01-basic-pv-pvc.yaml) - Simple static provisioning
2. [Storage Classes Example](./examples/02-storage-classes.yaml) - Dynamic provisioning with different storage classes
3. [StatefulSet with PVC Templates](./examples/03-statefulset-pvc.yaml) - Using PVCs with StatefulSets
4. [Volume Snapshots Example](./examples/04-volume-snapshots.yaml) - Creating and restoring from snapshots
5. [Multi-Tier Application Example](./examples/05-multi-tier-app.yaml) - Complex application with different volume types
6. [CSI Volumes Example](./examples/06-csi-volumes.yaml) - Using Container Storage Interface (CSI) drivers

## Best Practices

### Capacity Planning
- Request only what you need but include room for growth
- Consider using volume expansion when available
- Monitor storage usage to detect trends and potential issues

### Security
- Use StorageClass parameters for encryption when available
- Consider Pod Security Context for file ownership and permissions
- Use network policies to restrict access to storage services

### Performance
- Choose the right storage type for your workload (SSD vs HDD)
- Consider local volumes for high-performance needs
- Be aware of network bottlenecks for remote storage

### Reliability
- Use redundant storage options for critical data
- Implement regular backup procedures with volume snapshots
- Test recovery procedures from backups

### Management
- Label your PVs and PVCs for better organization
- Use namespaces to isolate storage resources
- Consider using Kubernetes operators for complex storage needs

### Cost Optimization
- Delete unused PVs to avoid unnecessary charges
- Use the appropriate storage class for your needs (don't use premium storage for non-critical data)
- Consider volume expansion instead of provisioning new, larger volumes

## Troubleshooting

### Common Issues

#### PVC in Pending State
- Check if a matching PV exists
- Check if the requested storage class exists
- Check if the access mode is supported by available PVs
- Check if requested size is available

```bash
kubectl describe pvc <pvc-name>
kubectl get storageclass
```

#### Volume Mount Issues
- Check if the PV/PVC is bound
- Check node events for mount errors
- Check Pod events for volume issues

```bash
kubectl describe pod <pod-name>
kubectl describe pv <pv-name>
```

#### Dynamic Provisioning Failures
- Check StorageClass provisioner is available
- Check cloud provider permissions
- Check cloud quotas and limits

```bash
kubectl describe storageclass <storage-class-name>
kubectl get events
```

## Further Reading

- [Kubernetes Documentation on Persistent Volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)
- [Kubernetes Documentation on Storage Classes](https://kubernetes.io/docs/concepts/storage/storage-classes/)
- [Container Storage Interface (CSI) Documentation](https://kubernetes-csi.github.io/docs/)
- [Kubernetes Volume Snapshots](https://kubernetes.io/docs/concepts/storage/volume-snapshots/)
- [Persistent Volumes with Stateful Applications](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/)

## Next Steps

- Learn about [Networking and Ingress](../10-networking-ingress/) for exposing services externally
- Study [Resource Management](../11-resource-management/) for CPU and memory control
- Explore [Custom Resources](../12-custom-resources/) to extend Kubernetes API 