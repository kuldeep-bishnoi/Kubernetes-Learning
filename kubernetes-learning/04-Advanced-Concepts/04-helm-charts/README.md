# Helm Charts & Package Management

## Introduction
Helm is the package manager for Kubernetes that simplifies the deployment and management of applications. It allows you to define, install, and upgrade complex Kubernetes applications using "charts" - packages of pre-configured Kubernetes resources.

## Key Concepts

### Chart
A Helm chart is a collection of files that describe a related set of Kubernetes resources. It contains:
- YAML manifests for Kubernetes resources
- Templates that generate Kubernetes manifests
- Default values that can be overridden
- Documentation and metadata

### Release
A release is an instance of a chart running in a Kubernetes cluster. One chart can be installed multiple times into the same cluster, creating a new release each time.

### Repository
A repository is a location where charts can be stored and shared. Helm can pull charts from multiple repositories simultaneously.

### Values
Values provide a way to override default configuration in a chart. They can be supplied via:
- The `values.yaml` file in the chart
- A separate YAML file specified with `-f` flag
- Command-line parameters with `--set`

## Helm Architecture

### Helm CLI
The command-line client that sends commands to the Kubernetes API server. It's responsible for:
- Creating and managing charts locally
- Managing repositories
- Interacting with the chart repository
- Installing, upgrading, and uninstalling charts
- Managing releases

### Charts
The packaging format for Helm. Charts contain:
- `Chart.yaml` - Metadata about the chart
- `values.yaml` - Default configuration values
- `templates/` - Directory of templates that generate Kubernetes manifests
- `charts/` - Directory of dependency charts
- `README.md` - Documentation
- `LICENSE` - License information (optional)
- `crds/` - Custom Resource Definitions (optional)

## Common Helm Commands

```bash
# Add a chart repository
helm repo add bitnami https://charts.bitnami.com/bitnami

# Update repositories
helm repo update

# Search for charts
helm search repo nginx

# Install a chart
helm install my-release bitnami/nginx

# Install with custom values
helm install my-release bitnami/nginx -f custom-values.yaml

# Install with specific values
helm install my-release bitnami/nginx --set replicaCount=3

# List releases
helm list

# Upgrade a release
helm upgrade my-release bitnami/nginx --set replicaCount=5

# Rollback a release
helm rollback my-release 1

# Uninstall a release
helm uninstall my-release

# Create a new chart
helm create my-chart

# Package a chart
helm package my-chart

# Install a chart from a local directory
helm install my-release ./my-chart
```

## Chart Structure
```
mychart/
  ├── Chart.yaml           # Chart metadata
  ├── values.yaml          # Default configuration values
  ├── charts/              # Dependency charts
  ├── templates/           # Template files
  │   ├── deployment.yaml  # Kubernetes manifests
  │   ├── service.yaml
  │   ├── ingress.yaml
  │   ├── _helpers.tpl     # Template helpers
  │   └── NOTES.txt        # Usage notes displayed after installation
  ├── templates/tests/     # Test files (optional)
  ├── crds/                # Custom Resource Definitions (optional)
  ├── README.md            # Documentation
  └── LICENSE              # License information (optional)
```

## Chart.yaml Example
```yaml
apiVersion: v2
name: mychart
version: 1.0.0
description: A Helm chart for Kubernetes
type: application
appVersion: "1.16.0"
dependencies:
  - name: mongodb
    version: 10.0.0
    repository: https://charts.bitnami.com/bitnami
maintainers:
  - name: John Doe
    email: john@example.com
```

## Templates and Values

### Example values.yaml
```yaml
replicaCount: 1

image:
  repository: nginx
  tag: 1.21
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 80

resources:
  limits:
    cpu: 100m
    memory: 128Mi
  requests:
    cpu: 50m
    memory: 64Mi
```

### Example template (deployment.yaml)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "mychart.fullname" . }}
  labels:
    {{- include "mychart.labels" . | nindent 4 }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      {{- include "mychart.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      labels:
        {{- include "mychart.selectorLabels" . | nindent 8 }}
    spec:
      containers:
        - name: {{ .Chart.Name }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          ports:
            - name: http
              containerPort: 80
              protocol: TCP
          resources:
            {{- toYaml .Values.resources | nindent 12 }}
```

## Templating Functions and Pipeline
Helm uses the Go templating language with additional functions:

- `{{ .Values.key }}` - Access values
- `{{ .Release.Name }}` - Access release information
- `{{ .Chart.Name }}` - Access chart information
- `{{ include "template.name" . }}` - Include a named template
- `{{ toYaml .Values.resources }}` - Convert to YAML
- `{{ if condition }}...{{ else }}...{{ end }}` - Conditional blocks
- `{{ range .Values.list }}...{{ end }}` - Loop through a list

### Pipelines
```yaml
# Indentation management
{{- toYaml .Values.resources | nindent 12 }}

# String transformation
{{ .Values.name | upper | quote }}

# Default values
{{ .Values.enabled | default false }}
```

## Chart Dependencies
Dependencies allow charts to rely on other charts, defined in Chart.yaml:

```yaml
dependencies:
  - name: apache
    version: 1.2.3
    repository: https://charts.bitnami.com/bitnami
  - name: mysql
    version: 8.0.0
    repository: https://charts.bitnami.com/bitnami
    condition: mysql.enabled
```

Update dependencies with:
```bash
helm dependency update ./mychart
```

## Chart Repositories
You can host your own chart repository using:
- GitHub Pages
- Object storage (S3, GCS)
- ChartMuseum
- Harbor
- Artifactory

## Best Practices
1. **Make Charts Configurable**: Design charts to be highly configurable with sensible defaults
2. **Version Your Charts**: Increment versions according to semantic versioning
3. **Document Your Charts**: Add detailed READMEs and helpful NOTES.txt
4. **Test Your Charts**: Use helm lint and Helm test capabilities
5. **Use Helper Templates**: Create reusable templates for common patterns
6. **Follow Naming Conventions**: Use consistent naming in templates and values
7. **Security Considerations**: Avoid hardcoding secrets, use Kubernetes secrets
8. **Application Versioning**: Keep track of the application version in appVersion
9. **Dependency Management**: Specify dependencies with version constraints

## Examples
1. Simple Application Deployment
2. Multi-tier Application with Dependencies 
3. Custom Resource Definition Management
4. Using Hooks for Pre/Post Installation Tasks
5. Secrets Management with Helm

## Related Concepts
- Kustomize
- Flux and ArgoCD for GitOps
- Helm Operators
- OCI Registries for Helm

## Further Reading
- [Helm Documentation](https://helm.sh/docs/)
- [Artifact Hub](https://artifacthub.io/)
- [Helm Best Practices](https://helm.sh/docs/chart_best_practices/)
- [Helm Charts GitHub](https://github.com/helm/charts) 