# Helm Testing

This example demonstrates how to implement and run tests for Helm charts using the built-in testing framework. Helm tests allow you to verify that your chart works as expected when installed in a cluster.

## Overview of Helm Tests

Helm tests are Kubernetes resources (typically Pods) that execute after a chart is installed to verify that it's working correctly. They are defined in the `templates/tests/` directory of a chart and are annotated with `helm.sh/hook: test`.

## When to Use Helm Tests

- To verify that your chart installs and configures resources correctly
- To ensure services are accessible and functioning
- To validate that configuration values are applied properly
- To check integrations between components

## Test Structure

```
web-app/
├── Chart.yaml
├── values.yaml
├── templates/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── configmap.yaml
│   ├── _helpers.tpl
│   ├── NOTES.txt
│   └── tests/
│       ├── test-connection.yaml
│       ├── test-mariadb-connection.yaml
│       └── test-api-endpoints.yaml
└── README.md
```

## Example Chart Configuration

### Chart.yaml

```yaml
apiVersion: v2
name: web-app
description: A web application with tests
type: application
version: 0.1.0
appVersion: "1.0.0"
dependencies:
  - name: mariadb
    version: 11.0.14
    repository: https://charts.bitnami.com/bitnami
    condition: mariadb.enabled
```

### values.yaml

```yaml
replicaCount: 1

image:
  repository: nginx
  tag: 1.21.6
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 80

api:
  endpoints:
    - /api/health
    - /api/version
    - /api/users

mariadb:
  enabled: true
  auth:
    rootPassword: "password"
    database: "app_database"
    username: "app_user"
    password: "app_password"
```

## Basic Test Examples

### Connection Test

This basic test verifies that the service is accessible:

```yaml
# templates/tests/test-connection.yaml
apiVersion: v1
kind: Pod
metadata:
  name: "{{ include "web-app.fullname" . }}-test-connection"
  labels:
    {{- include "web-app.labels" . | nindent 4 }}
  annotations:
    "helm.sh/hook": test
    "helm.sh/hook-delete-policy": "before-hook-creation,hook-succeeded"
spec:
  containers:
    - name: wget
      image: busybox
      command: ['wget']
      args: ['{{ include "web-app.fullname" . }}:{{ .Values.service.port }}']
  restartPolicy: Never
```

### Database Connection Test

This test verifies that the application can connect to the database:

```yaml
# templates/tests/test-mariadb-connection.yaml
apiVersion: v1
kind: Pod
metadata:
  name: "{{ include "web-app.fullname" . }}-test-db"
  labels:
    {{- include "web-app.labels" . | nindent 4 }}
  annotations:
    "helm.sh/hook": test
    "helm.sh/hook-delete-policy": "before-hook-creation,hook-succeeded"
spec:
  containers:
    - name: mysql-client
      image: mariadb:10.5
      env:
        - name: MYSQL_PWD
          value: {{ .Values.mariadb.auth.password | quote }}
      command:
        - /bin/bash
        - -ec
        - |
          mysql -h {{ .Release.Name }}-mariadb -u {{ .Values.mariadb.auth.username }} \
            -e "SELECT 1;" {{ .Values.mariadb.auth.database }}
  restartPolicy: Never
```

### API Endpoints Test

This test verifies that all API endpoints defined in values are accessible:

```yaml
# templates/tests/test-api-endpoints.yaml
apiVersion: v1
kind: Pod
metadata:
  name: "{{ include "web-app.fullname" . }}-test-api"
  labels:
    {{- include "web-app.labels" . | nindent 4 }}
  annotations:
    "helm.sh/hook": test
    "helm.sh/hook-delete-policy": "before-hook-creation,hook-succeeded"
spec:
  containers:
    - name: endpoint-tester
      image: curlimages/curl:7.80.0
      command:
        - /bin/sh
        - -ec
        - |
          SERVICE="{{ include "web-app.fullname" . }}:{{ .Values.service.port }}"
          {{- range .Values.api.endpoints }}
          echo "Testing endpoint: {{ . }}"
          curl -s -f http://$SERVICE{{ . }}
          {{- end }}
  restartPolicy: Never
```

## Advanced Test Examples

### Parallel Tests with Results

This example shows how to run multiple tests in parallel and collect their results:

```yaml
# templates/tests/test-suite.yaml
{{- range $i, $endpoint := .Values.api.endpoints }}
---
apiVersion: v1
kind: Pod
metadata:
  name: "{{ include "web-app.fullname" $ }}-test-{{ $i }}"
  labels:
    {{- include "web-app.labels" $ | nindent 4 }}
    test-endpoint: "{{ $endpoint | replace "/" "-" }}"
  annotations:
    "helm.sh/hook": test
    "helm.sh/hook-delete-policy": "before-hook-creation,hook-succeeded"
spec:
  containers:
    - name: tester
      image: curlimages/curl:7.80.0
      command:
        - /bin/sh
        - -c
        - |
          echo "Testing endpoint: {{ $endpoint }}"
          if curl -s -f "http://{{ include "web-app.fullname" $ }}:{{ $.Values.service.port }}{{ $endpoint }}"; then
            echo "✅ Test passed for {{ $endpoint }}"
            exit 0
          else
            echo "❌ Test failed for {{ $endpoint }}"
            exit 1
          fi
  restartPolicy: Never
{{- end }}
```

### Load Test

This test verifies that the application can handle a certain amount of load:

```yaml
# templates/tests/test-load.yaml
apiVersion: v1
kind: Pod
metadata:
  name: "{{ include "web-app.fullname" . }}-test-load"
  labels:
    {{- include "web-app.labels" . | nindent 4 }}
  annotations:
    "helm.sh/hook": test
    "helm.sh/hook-delete-policy": "before-hook-creation,hook-succeeded"
spec:
  containers:
    - name: load-tester
      image: fortio/fortio:latest
      command:
        - fortio
        - load
        - -qps=50
        - -t=30s
        - -c=25
        - http://{{ include "web-app.fullname" . }}:{{ .Values.service.port }}/
  restartPolicy: Never
```

### Security Test

This test checks for basic security configurations:

```yaml
# templates/tests/test-security.yaml
apiVersion: v1
kind: Pod
metadata:
  name: "{{ include "web-app.fullname" . }}-test-security"
  labels:
    {{- include "web-app.labels" . | nindent 4 }}
  annotations:
    "helm.sh/hook": test
    "helm.sh/hook-delete-policy": "before-hook-creation,hook-succeeded"
spec:
  containers:
    - name: security-check
      image: aquasec/trivy:latest
      args:
        - --quiet
        - --severity=CRITICAL
        - "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
  restartPolicy: Never
```

## Running Tests

### Basic Test Execution

To run all tests for a chart:

```bash
helm test release-name
```

### Running Tests with Logs

To see the output from test pods:

```bash
helm test release-name --logs
```

### Running Tests in a Specific Namespace

```bash
helm test release-name --namespace your-namespace
```

## Test Flags and Options

- `--logs`: Display the logs of the test pods
- `--filter=label=value`: Only run tests with matching labels
- `--timeout=300s`: Set a timeout for the tests
- `--parallel`: Run test pods in parallel instead of sequentially (Helm 3.7+)

## Test Best Practices

1. **Keep tests focused**: Each test should verify one specific aspect of functionality
2. **Use appropriate images**: Use lightweight images when possible to speed up test execution
3. **Implement proper cleanup**: Use appropriate delete policies to clean up test resources
4. **Provide clear output**: Tests should clearly indicate what they're testing and whether they passed or failed
5. **Test resources should be ephemeral**: Tests should not leave any resources behind
6. **Make tests idempotent**: Tests should be able to run multiple times without side effects
7. **Include a variety of test types**: Test connections, functionality, performance, and security
8. **Add labels to tests**: Organize tests with labels so they can be filtered during execution
9. **Set realistic timeouts**: Ensure tests don't run indefinitely if they fail
10. **Document test requirements**: If tests require specific configurations, document them clearly

## Example Application Deployment

The web application deployment that we're testing:

```yaml
# templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "web-app.fullname" . }}
  labels:
    {{- include "web-app.labels" . | nindent 4 }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      {{- include "web-app.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      labels:
        {{- include "web-app.selectorLabels" . | nindent 8 }}
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
```

## Testing Workflow

A typical workflow for developing and testing Helm charts:

1. **Develop chart**: Create or modify your Helm chart
2. **Add tests**: Create test files in the `templates/tests/` directory
3. **Lint chart**: Run `helm lint` to check for syntax errors
4. **Install chart**: Install the chart with `helm install --dry-run` to validate templates
5. **Install chart**: Install the actual chart with `helm install`
6. **Run tests**: Execute tests with `helm test`
7. **Review results**: Check test logs and results
8. **Clean up**: Uninstall the chart with `helm uninstall`

## Continuous Integration

Helm tests can be integrated into CI/CD pipelines:

```yaml
# Example GitHub Actions workflow
name: Test Helm Chart

on:
  push:
    paths:
      - 'charts/**'

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v2
        
      - name: Set up Kubernetes
        uses: helm/kind-action@v1.2.0
        
      - name: Add Helm repositories
        run: |
          helm repo add bitnami https://charts.bitnami.com/bitnami
          helm repo update
          
      - name: Install chart
        run: helm install test-release ./charts/web-app
        
      - name: Run tests
        run: helm test test-release --logs
        
      - name: Clean up
        run: helm uninstall test-release
```

## Debugging Helm Tests

If your tests fail, you can:

1. Check the logs of the test pod:
   ```bash
   kubectl logs pod/test-release-web-app-test-connection
   ```

2. Describe the test pod for more details:
   ```bash
   kubectl describe pod/test-release-web-app-test-connection
   ```

3. Examine test pod events:
   ```bash
   kubectl get events --field-selector involvedObject.name=test-release-web-app-test-connection
   ```

## Conclusion

Helm tests are a powerful way to validate that your Helm charts work correctly after installation. By writing comprehensive tests, you can ensure your applications are correctly deployed and functioning as expected. This helps catch issues early in the deployment process and builds confidence in your Helm charts. 