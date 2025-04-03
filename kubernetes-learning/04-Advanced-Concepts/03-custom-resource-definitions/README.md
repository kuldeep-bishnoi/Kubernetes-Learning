# Custom Resource Definitions (CRDs)

## Introduction
Custom Resource Definitions (CRDs) extend the Kubernetes API by defining custom resources, allowing you to add and manage new types of objects in your Kubernetes cluster. With CRDs, you can create domain-specific abstractions and build automation around these resources using controllers, enabling Kubernetes to manage application-specific workflows and configurations.

## How CRDs Work
1. **Define a CRD**: Register a YAML definition of your custom resource with the Kubernetes API server
2. **Create Custom Resources**: Once registered, users can create instances of the custom resource
3. **Build Controllers**: Optionally create a custom controller to watch and manage these resources

## Key Concepts

### Custom Resource Definition (CRD)
- The schema that defines the new resource type
- Describes the structure, validation rules, and metadata for the custom resource

### Custom Resource (CR)
- An instance of a resource defined by a CRD
- Represents a specific configuration or state that should exist

### Controllers
- Software that implements the business logic for custom resources
- Reconciles the actual state with the desired state described in the CR
- Usually follows the Kubernetes operator pattern

## Example CRD Structure
```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: backups.example.com
spec:
  group: example.com
  versions:
    - name: v1
      served: true
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                cronSchedule:
                  type: string
                storageLocation:
                  type: string
                retentionPolicy:
                  type: object
                  properties:
                    retentionDays:
                      type: integer
                      minimum: 1
              required: ["cronSchedule", "storageLocation"]
  scope: Namespaced
  names:
    plural: backups
    singular: backup
    kind: Backup
    shortNames: ["bk"]
```

## Use Cases for CRDs
1. **Infrastructure Management**: Define complex infrastructure components like databases and message queues
2. **Application Configuration**: Manage application-specific configuration in a Kubernetes-native way
3. **Operational Tasks**: Define and automate recurring operations like backups, upgrades, and scaling
4. **Policy Enforcement**: Implement custom policies and compliance rules
5. **Platform Extensions**: Extend Kubernetes to handle specialized workloads and services

## Best Practices

### Design
1. Follow Kubernetes API conventions for object structure
2. Use proper versioning from the start
3. Define detailed validation rules in your schema
4. Implement status subresources to track object state
5. Consider using controller-gen or kubebuilder tools

### Implementation
1. Design controllers to be idempotent and resilient
2. Implement proper error handling and retry logic
3. Use finalizers to handle resource cleanup
4. Include events for debugging and monitoring
5. Implement proper status updates

### Versioning and Updates
1. Start with a planned versioning strategy
2. Support conversion between versions
3. Use the "storage version" flag correctly
4. Consider backward compatibility for updates

## Examples
1. Basic CRD Definition
2. CRD with Schema Validation
3. CRD with Status Subresource
4. Custom Controller Implementation
5. Operator Pattern Implementation

## Related Concepts
- Kubernetes Operators
- Admission Controllers
- API Aggregation
- Client Libraries and API Machinery
- Controller Runtime and Kubebuilder

## Further Reading
- [Kubernetes CRD Documentation](https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/)
- [Operator Pattern](https://kubernetes.io/docs/concepts/extend-kubernetes/operator/)
- [Kubebuilder](https://book.kubebuilder.io/)
- [Operator SDK](https://sdk.operatorframework.io/) 