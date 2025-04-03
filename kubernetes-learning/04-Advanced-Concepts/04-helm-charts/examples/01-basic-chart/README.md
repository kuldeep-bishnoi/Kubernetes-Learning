# Basic Helm Chart Example

This directory contains an example of a simple Helm chart for deploying a web application. The chart demonstrates Helm's templating capabilities and configuration management.

## Chart Structure
This example follows the standard Helm chart structure:

```
basic-web-app/
  ├── Chart.yaml           # Chart metadata
  ├── values.yaml          # Default configuration values
  ├── templates/           # Template files
  │   ├── deployment.yaml  # Deployment template
  │   ├── service.yaml     # Service template
  │   ├── configmap.yaml   # ConfigMap template for configuration
  │   ├── _helpers.tpl     # Template helpers
  │   └── NOTES.txt        # Usage notes displayed after installation
  └── README.md            # Documentation
```

## Creating the Chart

To create this chart structure:

```bash
# Create a new chart with the Helm scaffolding
helm create basic-web-app

# Clean up unnecessary templates
rm -rf basic-web-app/templates/tests/
rm -f basic-web-app/templates/hpa.yaml
rm -f basic-web-app/templates/serviceaccount.yaml
rm -f basic-web-app/templates/ingress.yaml
```

## Chart.yaml
The Chart.yaml file contains metadata about the chart:

```yaml
apiVersion: v2
name: basic-web-app
description: A basic web application Helm chart for Kubernetes
type: application
version: 0.1.0
appVersion: "1.0.0"
```

## values.yaml
The values.yaml file contains default configuration values:

```yaml
# Default values for basic-web-app
replicaCount: 1

image:
  repository: nginx
  tag: "1.21.6"
  pullPolicy: IfNotPresent

nameOverride: ""
fullnameOverride: ""

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

config:
  html: |
    <!DOCTYPE html>
    <html>
    <head>
      <title>Welcome to Helm!</title>
      <style>
        body {
          width: 35em;
          margin: 0 auto;
          font-family: Tahoma, Verdana, Arial, sans-serif;
        }
      </style>
    </head>
    <body>
      <h1>Welcome to Helm!</h1>
      <p>If you see this page, your Helm chart has been successfully deployed.</p>
      <p>For more examples and documentation, please visit
      <a href="https://helm.sh/">helm.sh</a>.</p>
    </body>
    </html>
```

## Templates

### _helpers.tpl
The helpers file contains reusable template snippets:

```tpl
{{/*
Expand the name of the chart.
*/}}
{{- define "basic-web-app.name" -}}
{{- default .Chart.Name .Values.nameOverride | trunc 63 | trimSuffix "-" }}
{{- end }}

{{/*
Create a default fully qualified app name.
*/}}
{{- define "basic-web-app.fullname" -}}
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
{{- define "basic-web-app.chart" -}}
{{- printf "%s-%s" .Chart.Name .Chart.Version | replace "+" "_" | trunc 63 | trimSuffix "-" }}
{{- end }}

{{/*
Common labels
*/}}
{{- define "basic-web-app.labels" -}}
helm.sh/chart: {{ include "basic-web-app.chart" . }}
{{ include "basic-web-app.selectorLabels" . }}
app.kubernetes.io/version: {{ .Chart.AppVersion | quote }}
app.kubernetes.io/managed-by: {{ .Release.Service }}
{{- end }}

{{/*
Selector labels
*/}}
{{- define "basic-web-app.selectorLabels" -}}
app.kubernetes.io/name: {{ include "basic-web-app.name" . }}
app.kubernetes.io/instance: {{ .Release.Name }}
{{- end }}
```

### configmap.yaml
This template creates a ConfigMap with HTML content:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ include "basic-web-app.fullname" . }}
  labels:
    {{- include "basic-web-app.labels" . | nindent 4 }}
data:
  index.html: |-
    {{- .Values.config.html | nindent 4 }}
```

### deployment.yaml
This template creates a Deployment:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "basic-web-app.fullname" . }}
  labels:
    {{- include "basic-web-app.labels" . | nindent 4 }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      {{- include "basic-web-app.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      labels:
        {{- include "basic-web-app.selectorLabels" . | nindent 8 }}
    spec:
      containers:
        - name: {{ .Chart.Name }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          ports:
            - name: http
              containerPort: 80
              protocol: TCP
          livenessProbe:
            httpGet:
              path: /
              port: http
          readinessProbe:
            httpGet:
              path: /
              port: http
          resources:
            {{- toYaml .Values.resources | nindent 12 }}
          volumeMounts:
            - name: config
              mountPath: /usr/share/nginx/html/index.html
              subPath: index.html
      volumes:
        - name: config
          configMap:
            name: {{ include "basic-web-app.fullname" . }}
```

### service.yaml
This template creates a Service:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ include "basic-web-app.fullname" . }}
  labels:
    {{- include "basic-web-app.labels" . | nindent 4 }}
spec:
  type: {{ .Values.service.type }}
  ports:
    - port: {{ .Values.service.port }}
      targetPort: http
      protocol: TCP
      name: http
  selector:
    {{- include "basic-web-app.selectorLabels" . | nindent 4 }}
```

### NOTES.txt
This file is displayed after chart installation:

```txt
Thank you for installing {{ .Chart.Name }}.

Your release is named {{ .Release.Name }}.

To get the application URL, run these commands:
{{- if contains "ClusterIP" .Values.service.type }}
  kubectl port-forward svc/{{ include "basic-web-app.fullname" . }} 8080:{{ .Values.service.port }}
  echo "Visit http://localhost:8080 to access your application"
{{- else if contains "NodePort" .Values.service.type }}
  export NODE_PORT=$(kubectl get svc {{ include "basic-web-app.fullname" . }} -o jsonpath="{.spec.ports[0].nodePort}")
  export NODE_IP=$(kubectl get nodes -o jsonpath="{.items[0].status.addresses[0].address}")
  echo "Visit http://$NODE_IP:$NODE_PORT to access your application"
{{- else if contains "LoadBalancer" .Values.service.type }}
  NOTE: It may take a few minutes for the LoadBalancer IP to be available.
  export SERVICE_IP=$(kubectl get svc {{ include "basic-web-app.fullname" . }} -o jsonpath="{.spec.status.loadBalancer.ingress[0].ip}")
  echo "Visit http://$SERVICE_IP:{{ .Values.service.port }} to access your application"
{{- end }}
```

## Using the Chart

### Installing the Chart
```bash
# From the parent directory
helm install my-web-app ./basic-web-app

# Or with custom values
helm install my-web-app ./basic-web-app --set replicaCount=3 --set service.type=NodePort
```

### Upgrading the Chart
```bash
# Update the configuration
helm upgrade my-web-app ./basic-web-app --set image.tag=1.22.0
```

### Uninstalling the Chart
```bash
helm uninstall my-web-app
```

## Customizing the Chart
You can create a custom values file, for example `custom-values.yaml`:

```yaml
replicaCount: 2
service:
  type: NodePort
resources:
  limits:
    memory: 256Mi
config:
  html: |
    <!DOCTYPE html>
    <html>
    <head>
      <title>Custom Configuration</title>
    </head>
    <body>
      <h1>This is a custom configuration!</h1>
    </body>
    </html>
```

Then install or upgrade with:
```bash
helm install my-web-app ./basic-web-app -f custom-values.yaml
# or
helm upgrade my-web-app ./basic-web-app -f custom-values.yaml
```

## Testing the Chart
Before installation, you can validate the chart and see what resources would be created:

```bash
# Lint the chart
helm lint ./basic-web-app

# Check generated templates
helm template ./basic-web-app

# Dry run the installation
helm install --dry-run --debug my-web-app ./basic-web-app
```