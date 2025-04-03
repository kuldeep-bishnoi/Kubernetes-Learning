# Helm Chart with Dependencies Example

This example demonstrates how to create a Helm chart for a multi-tier application with dependencies. We'll create a chart for a web application that depends on both a database and a Redis cache.

## Chart Structure

```
todo-app/
  ├── Chart.yaml           # Chart metadata and dependencies
  ├── values.yaml          # Default configuration values
  ├── charts/              # Directory for dependency charts
  ├── templates/           # Application templates
  │   ├── deployment.yaml  # Web app deployment
  │   ├── service.yaml     # Web app service
  │   ├── configmap.yaml   # Configuration
  │   ├── _helpers.tpl     # Helper templates
  │   └── NOTES.txt        # Usage notes
  └── README.md            # Documentation
```

## Creating the Parent Chart

Create the directory structure:
```bash
mkdir -p todo-app/templates
mkdir -p todo-app/charts
```

## Defining Dependencies in Chart.yaml

The `Chart.yaml` file includes dependencies on MySQL and Redis:

```yaml
apiVersion: v2
name: todo-app
description: A Todo List application with MySQL and Redis
type: application
version: 0.1.0
appVersion: "1.0.0"
dependencies:
  - name: mysql
    version: "9.x.x"
    repository: "https://charts.bitnami.com/bitnami"
    condition: mysql.enabled
  - name: redis
    version: "17.x.x"
    repository: "https://charts.bitnami.com/bitnami"
    condition: redis.enabled
```

## Default Values in values.yaml

Here's a sample `values.yaml` with application settings and overrides for dependencies:

```yaml
# Application configuration
replicaCount: 1

image:
  repository: ghcr.io/example/todo-app
  tag: latest
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 80

resources:
  limits:
    cpu: 200m
    memory: 256Mi
  requests:
    cpu: 100m
    memory: 128Mi

# Enable dependencies
mysql:
  enabled: true
  auth:
    database: todos
    username: todo_user
    password: todo_password
    rootPassword: root_password
  primary:
    persistence:
      size: 1Gi

redis:
  enabled: true
  auth:
    password: redis_password
  master:
    persistence:
      size: 1Gi

# Application-specific configuration
config:
  databaseUrl: "mysql://todo_user:todo_password@{{ .Release.Name }}-mysql/todos"
  redisUrl: "redis://:redis_password@{{ .Release.Name }}-redis-master:6379/0"
  logLevel: "info"
```

## Template Files

### _helpers.tpl

```tpl
{{/*
Expand the name of the chart.
*/}}
{{- define "todo-app.name" -}}
{{- default .Chart.Name .Values.nameOverride | trunc 63 | trimSuffix "-" }}
{{- end }}

{{/*
Create a default fully qualified app name.
*/}}
{{- define "todo-app.fullname" -}}
{{- if .Values.fullnameOverride }}
{{- .Values.fullnameOverride | trunc 63 | trimSuffix "-" }}
{{- else }}
{{- $name := default .Chart.Name .Values.nameOverride }}
{{- if contains $name .Release.Name }}
{{- .Release.Name | trunc 63 | trimSuffix "-" }}
{{- else }}
{{- printf "%s-%s" .Release.Name $name | trunc 63 | trimSuffix "-" }}
{{- end }}
{{- end }}
{{- end }}

{{/*
Create chart name and version as used by the chart label.
*/}}
{{- define "todo-app.chart" -}}
{{- printf "%s-%s" .Chart.Name .Chart.Version | replace "+" "_" | trunc 63 | trimSuffix "-" }}
{{- end }}

{{/*
Common labels
*/}}
{{- define "todo-app.labels" -}}
helm.sh/chart: {{ include "todo-app.chart" . }}
{{ include "todo-app.selectorLabels" . }}
app.kubernetes.io/version: {{ .Chart.AppVersion | quote }}
app.kubernetes.io/managed-by: {{ .Release.Service }}
{{- end }}

{{/*
Selector labels
*/}}
{{- define "todo-app.selectorLabels" -}}
app.kubernetes.io/name: {{ include "todo-app.name" . }}
app.kubernetes.io/instance: {{ .Release.Name }}
{{- end }}
```

### configmap.yaml

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ include "todo-app.fullname" . }}
  labels:
    {{- include "todo-app.labels" . | nindent 4 }}
data:
  config.json: |
    {
      "database_url": "{{ .Values.config.databaseUrl }}",
      "redis_url": "{{ .Values.config.redisUrl }}",
      "log_level": "{{ .Values.config.logLevel }}"
    }
```

### deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "todo-app.fullname" . }}
  labels:
    {{- include "todo-app.labels" . | nindent 4 }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      {{- include "todo-app.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      labels:
        {{- include "todo-app.selectorLabels" . | nindent 8 }}
    spec:
      containers:
        - name: {{ .Chart.Name }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          ports:
            - name: http
              containerPort: 3000
              protocol: TCP
          livenessProbe:
            httpGet:
              path: /health
              port: http
          readinessProbe:
            httpGet:
              path: /health
              port: http
          resources:
            {{- toYaml .Values.resources | nindent 12 }}
          volumeMounts:
            - name: config
              mountPath: /app/config.json
              subPath: config.json
          env:
            - name: NODE_ENV
              value: "production"
            {{- if .Values.mysql.enabled }}
            - name: DATABASE_URL
              value: {{ .Values.config.databaseUrl }}
            {{- end }}
            {{- if .Values.redis.enabled }}
            - name: REDIS_URL
              value: {{ .Values.config.redisUrl }}
            {{- end }}
      volumes:
        - name: config
          configMap:
            name: {{ include "todo-app.fullname" . }}
```

### service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ include "todo-app.fullname" . }}
  labels:
    {{- include "todo-app.labels" . | nindent 4 }}
spec:
  type: {{ .Values.service.type }}
  ports:
    - port: {{ .Values.service.port }}
      targetPort: http
      protocol: TCP
      name: http
  selector:
    {{- include "todo-app.selectorLabels" . | nindent 4 }}
```

### NOTES.txt

```txt
Thank you for installing {{ .Chart.Name }}.

Your release is named {{ .Release.Name }}.

To get the application URL, run these commands:
{{- if contains "ClusterIP" .Values.service.type }}
  kubectl port-forward svc/{{ include "todo-app.fullname" . }} 8080:{{ .Values.service.port }}
  echo "Visit http://localhost:8080 to access your application"
{{- else if contains "NodePort" .Values.service.type }}
  export NODE_PORT=$(kubectl get svc {{ include "todo-app.fullname" . }} -o jsonpath="{.spec.ports[0].nodePort}")
  export NODE_IP=$(kubectl get nodes -o jsonpath="{.items[0].status.addresses[0].address}")
  echo "Visit http://$NODE_IP:$NODE_PORT to access your application"
{{- else if contains "LoadBalancer" .Values.service.type }}
  NOTE: It may take a few minutes for the LoadBalancer IP to be available.
  export SERVICE_IP=$(kubectl get svc {{ include "todo-app.fullname" . }} -o jsonpath="{.spec.status.loadBalancer.ingress[0].ip}")
  echo "Visit http://$SERVICE_IP:{{ .Values.service.port }} to access your application"
{{- end }}

MySQL Connection Information:
{{- if .Values.mysql.enabled }}
  Host: {{ .Release.Name }}-mysql
  Database: {{ .Values.mysql.auth.database }}
  Username: {{ .Values.mysql.auth.username }}
{{- else }}
  MySQL is disabled.
{{- end }}

Redis Connection Information:
{{- if .Values.redis.enabled }}
  Host: {{ .Release.Name }}-redis-master
  Port: 6379
{{- else }}
  Redis is disabled.
{{- end }}
```

## Working with Dependencies

### Updating Dependencies

After defining dependencies in `Chart.yaml`, you need to download them:

```bash
# Add the Bitnami repository
helm repo add bitnami https://charts.bitnami.com/bitnami

# Update the repositories
helm repo update

# Download dependencies to the 'charts/' directory
helm dependency update ./todo-app
```

This will download the `mysql` and `redis` charts and place them in the `charts/` directory.

### Controlling Dependency Installation

Dependencies can be enabled or disabled using the `condition` field in `Chart.yaml`. In this example, 
MySQL and Redis will only be installed if `mysql.enabled` and `redis.enabled` are set to `true` 
in the values file.

You can also override settings for dependencies in your values file. For example, we're customizing 
the database name, username, and password for MySQL.

## Using the Chart

### Installing the Chart

```bash
# Install the chart with all dependencies
helm install my-todo-app ./todo-app

# Install without MySQL (using external database)
helm install my-todo-app ./todo-app --set mysql.enabled=false
```

### Overriding Dependency Values

You can override dependency values either in the parent chart's `values.yaml` or at installation time:

```bash
# Override MySQL settings
helm install my-todo-app ./todo-app \
  --set mysql.auth.database=custom_db \
  --set mysql.auth.username=custom_user \
  --set mysql.auth.password=custom_password
```

### Using a Custom Values File

Create a `custom-values.yaml` file:

```yaml
# Increase application replicas
replicaCount: 3

# Customize MySQL
mysql:
  auth:
    database: production_db
    username: prod_user
    password: prod_password
  primary:
    persistence:
      size: 10Gi

# Customize Redis
redis:
  master:
    persistence:
      size: 5Gi
```

Then install with:

```bash
helm install my-todo-app ./todo-app -f custom-values.yaml
```

## Managing Releases

### Upgrading the Chart

```bash
# Upgrade the application
helm upgrade my-todo-app ./todo-app --set replicaCount=5
```

### Inspecting the Installation

```bash
# List all releases
helm list

# Get details about the release
helm get all my-todo-app

# Get generated manifests
helm get manifest my-todo-app
```

### Uninstalling the Chart

```bash
helm uninstall my-todo-app
```

## Best Practices for Helm Dependencies

1. **Pin Version Numbers**: Always specify version constraints for dependencies
2. **Use Conditions**: Make dependencies optional when possible
3. **Document Values**: Document all available values that can be customized
4. **Test Dependency Updates**: Test your chart when upgrading dependencies
5. **Use Global Values**: Use global values for settings that affect multiple dependencies
6. **Alias Dependencies**: Use aliases when you need multiple instances of the same dependency
7. **Check Compatibility**: Ensure your application is compatible with the specific versions of dependencies

## Testing Dependencies

You can test dependency rendering with:

```bash
# See what templates will be rendered
helm template ./todo-app

# Validate the chart
helm lint ./todo-app

# Do a dry run install
helm install --dry-run --debug my-todo-app ./todo-app
``` 