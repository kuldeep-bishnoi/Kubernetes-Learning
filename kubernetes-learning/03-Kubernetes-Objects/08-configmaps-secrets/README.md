# Kubernetes ConfigMaps and Secrets

ConfigMaps and Secrets are Kubernetes objects that help you manage configuration data separately from application code, making your applications more portable and easier to maintain.

## ConfigMaps

A ConfigMap is an API object used to store non-confidential data in key-value pairs. Pods can consume ConfigMaps as environment variables, command-line arguments, or configuration files in a volume.

![ConfigMap Diagram](https://raw.githubusercontent.com/kubernetes/website/main/content/en/examples/configmap/configmap-diagram.svg)

### Use Cases for ConfigMaps

- Environment-specific configuration settings (dev, test, prod)
- Application configuration files
- Command-line arguments and flags
- Setting environment variables for applications
- Storing non-sensitive configuration data

## Secrets

A Secret is an object that contains a small amount of sensitive data such as a password, a token, or a key. This information might otherwise be put in a Pod specification or in a container image, but using Secrets helps reduce the risk of exposing sensitive data.

![Secret Usage](https://raw.githubusercontent.com/kubernetes/website/main/content/en/examples/pods/secret/secret-diagram.svg)

### Use Cases for Secrets

- Database credentials
- API tokens and keys
- TLS certificates
- SSH keys
- OAuth tokens
- Service account tokens

## ConfigMaps vs Secrets

| Feature | ConfigMap | Secret |
|---------|-----------|--------|
| Purpose | Non-sensitive configuration data | Sensitive data that needs protection |
| Storage | Stored as plaintext | Base64 encoded by default, can be encrypted |
| Size Limit | 1MB | 1MB |
| Use Cases | App config, env vars, CLI args | Passwords, tokens, certificates |
| Visibility | Easily viewable | Restricted visibility |

## Creating ConfigMaps

### Imperative Method

```bash
# From literal values
kubectl create configmap app-config \
  --from-literal=APP_ENV=production \
  --from-literal=APP_DEBUG=false

# From a file
kubectl create configmap app-config-file \
  --from-file=app-config.properties

# From multiple files in a directory
kubectl create configmap app-config-dir \
  --from-file=config-dir/
```

### Declarative Method (YAML)

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_ENV: production
  APP_DEBUG: "false"
  app.properties: |
    property.1=value-1
    property.2=value-2
    property.3=value-3
  config.yaml: |
    apiVersion: v1
    kind: Service
    metadata:
      name: my-service
    spec:
      selector:
        app: MyApp
      ports:
      - protocol: TCP
        port: 80
        targetPort: 9376
```

## Creating Secrets

### Imperative Method

```bash
# From literal values
kubectl create secret generic db-credentials \
  --from-literal=username=devuser \
  --from-literal=password='S!B\*d$zDsb='

# From files
kubectl create secret generic tls-certs \
  --from-file=cert=path/to/cert.pem \
  --from-file=key=path/to/key.pem

# TLS secret
kubectl create secret tls tls-secret \
  --cert=path/to/tls.cert \
  --key=path/to/tls.key
```

### Declarative Method (YAML)

⚠️ **Note**: Sensitive data must be base64 encoded for YAML. Don't use this approach for truly sensitive data as it will be visible in your version control.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-credentials
type: Opaque
data:
  username: ZGV2dXNlcg==  # 'devuser' in base64
  password: UyFCXCpkJHpEc2I9  # 'S!B\*d$zDsb=' in base64
```

For truly sensitive Secret management, consider solutions like:
- Sealed Secrets
- Vault
- External Secret Operators
- Cloud provider secret management services

## Using ConfigMaps in Pods

### As Environment Variables

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: config-env-example
spec:
  containers:
  - name: app
    image: my-app:1.0
    env:
    # Set individual environment variables from ConfigMap
    - name: APP_ENV
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: APP_ENV
    # Load all key-value pairs from ConfigMap as environment variables
    envFrom:
    - configMapRef:
        name: app-config
```

### As Command-Line Arguments

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: config-cmd-example
spec:
  containers:
  - name: app
    image: my-app:1.0
    command: ["/bin/sh", "-c"]
    args: ["echo $(APP_ENV) && python app.py"]
    env:
    - name: APP_ENV
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: APP_ENV
```

### As Files in a Volume

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: config-volume-example
spec:
  containers:
  - name: app
    image: my-app:1.0
    volumeMounts:
    - name: config-volume
      mountPath: /etc/config
  volumes:
  - name: config-volume
    configMap:
      name: app-config
      # Optionally specify specific items only
      items:
      - key: app.properties
        path: app.properties
      - key: config.yaml
        path: config.yaml
```

## Using Secrets in Pods

### As Environment Variables

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-env-example
spec:
  containers:
  - name: app
    image: my-app:1.0
    env:
    # Set individual environment variables from Secret
    - name: DB_USERNAME
      valueFrom:
        secretKeyRef:
          name: db-credentials
          key: username
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: db-credentials
          key: password
    # Load all key-value pairs from Secret as environment variables
    envFrom:
    - secretRef:
        name: db-credentials
```

### As Files in a Volume

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-volume-example
spec:
  containers:
  - name: app
    image: my-app:1.0
    volumeMounts:
    - name: secret-volume
      mountPath: /etc/secrets
      readOnly: true
  volumes:
  - name: secret-volume
    secret:
      secretName: db-credentials
      # Optionally set file permissions
      defaultMode: 0400
```

## Special Types of Secrets

### Docker Registry Credentials

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: docker-registry-credentials
type: kubernetes.io/dockerconfigjson
data:
  .dockerconfigjson: <base64-encoded-docker-config>
```

### TLS Certificates

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: tls-secret
type: kubernetes.io/tls
data:
  tls.crt: <base64-encoded-cert>
  tls.key: <base64-encoded-key>
```

### Service Account Tokens

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: service-account-token
  annotations:
    kubernetes.io/service-account.name: my-service-account
type: kubernetes.io/service-account-token
```

## Dynamic Updates

Both ConfigMaps and Secrets can be updated without rebuilding the application container. However, how the updates are reflected depends on how they are consumed:

- **Environment Variables**: Not automatically updated; pod must be restarted
- **Volume Mounts**: Automatically updated with some latency (typically a few minutes)
- **Mounted Files**: Volume contents updated, but application must watch for changes

## Best Practices

### ConfigMap Best Practices

1. **Use for Non-Sensitive Data Only**: Never store credentials or keys in ConfigMaps
2. **Organize by Application**: Create separate ConfigMaps for different applications
3. **Group by Update Frequency**: Keep data that changes together in the same ConfigMap
4. **Keep Size Small**: Stay well under the 1MB limit
5. **Use Descriptive Names**: Name ConfigMaps to describe their content and purpose
6. **Version Your Configurations**: Consider adding version info in names or labels
7. **Consider Immutability**: Use immutable ConfigMaps when possible

### Secret Best Practices

1. **Keep Secrets Small**: Stay well under the 1MB limit
2. **Don't Check Into Version Control**: Use tools like Sealed Secrets or external secret stores
3. **Enable Encryption at Rest**: Configure etcd encryption for Secrets
4. **Limit Access**: Use RBAC to restrict which users can read and modify Secrets
5. **Rotate Regularly**: Create processes to rotate Secrets regularly
6. **Use Namespaces**: Keep Secrets isolated in appropriate namespaces
7. **Set Resource Quotas**: Limit the number of Secrets in a namespace

## Common Issues and Troubleshooting

### ConfigMap Issues

- **ConfigMap Not Found**: Verify it exists in the correct namespace
- **Invalid YAML Format**: Check your YAML syntax
- **Size Limit Exceeded**: Keep ConfigMaps under 1MB
- **Key Not Found**: Verify the key exists in the ConfigMap
- **Permission Issues**: Check RBAC permissions

### Secret Issues

- **Secret Not Found**: Verify it exists in the correct namespace
- **Invalid Base64 Encoding**: Ensure data is properly base64-encoded
- **Size Limit Exceeded**: Keep Secrets under 1MB
- **Incorrect Secret Type**: Use the appropriate Secret type for your data
- **Secret Not Mounting**: Check volume mounts and permissions

## Advanced Topics

### Immutable ConfigMaps and Secrets

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: immutable-config
data:
  APP_ENV: production
immutable: true
```

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: immutable-secret
type: Opaque
data:
  username: ZGV2dXNlcg==
immutable: true
```

### Using ConfigMaps for Kustomize Configurations

```yaml
# kustomization.yaml
configMapGenerator:
- name: app-config
  literals:
  - APP_ENV=production
  - LOG_LEVEL=info
```

### Using External Secret Management

- **Sealed Secrets**: Encrypt Secrets for safe storage in Git
- **HashiCorp Vault**: External secret management system with Kubernetes integration
- **AWS Secrets Manager or Azure Key Vault**: Cloud provider solutions
- **External Secrets Operator**: Fetch secrets from external APIs

## Examples

See the [examples](./examples/) directory for sample ConfigMap and Secret configurations.

## Next Steps

- Learn about [Persistent Volumes](../09-persistent-volumes/) for persistent storage
- Explore [Networking and Ingress](../10-networking-ingress/) for exposing services externally
- Study [Resource Management](../11-resource-management/) for CPU and memory control 