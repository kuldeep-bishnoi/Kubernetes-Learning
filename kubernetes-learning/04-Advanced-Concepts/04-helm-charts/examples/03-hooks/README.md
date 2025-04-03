# Helm Hooks Example

This example demonstrates how to use Helm hooks to perform actions at specific points during a chart's lifecycle. Hooks allow you to intervene before or after certain events occur.

## What are Helm Hooks?

Hooks are special Kubernetes resources annotated to execute at specific points in a release's lifecycle, such as:
- Before installation
- After installation
- Before upgrade
- After upgrade
- Before deletion
- After deletion
- On test execution

## Hook Types

Helm provides several hook types:

- `pre-install`: Executes before any resources are created
- `post-install`: Executes after all resources are created
- `pre-delete`: Executes before deletion of releases
- `post-delete`: Executes after deletion of releases
- `pre-upgrade`: Executes before upgrading
- `post-upgrade`: Executes after upgrading
- `pre-rollback`: Executes before rolling back
- `post-rollback`: Executes after rolling back
- `test`: Executes when running helm test

## Example Application

Let's consider a web application that needs:
1. A database initialization job before installation
2. A notification after successful installation
3. A backup job before deletion
4. Cleanup tasks after deletion

## Project Structure

```
database-app/
├── Chart.yaml
├── values.yaml
├── templates/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── configmap.yaml
│   ├── _helpers.tpl
│   ├── NOTES.txt
│   └── hooks/
│       ├── pre-install-db-init.yaml
│       ├── post-install-notification.yaml
│       ├── pre-delete-backup.yaml
│       └── post-delete-cleanup.yaml
└── README.md
```

## Chart.yaml

```yaml
apiVersion: v2
name: database-app
description: An example application demonstrating Helm hooks
type: application
version: 0.1.0
appVersion: "1.0.0"
```

## values.yaml

```yaml
image:
  repository: nginx
  tag: 1.21.6
  pullPolicy: IfNotPresent

hooks:
  dbInit:
    image: postgres:14
    command: ["psql"]
  notification:
    image: curlimages/curl:7.80.0
    endpoint: "https://hooks.slack.com/services/YOUR_WEBHOOK"
  backup:
    image: bitnami/postgresql:14
    destination: "s3://backup-bucket/{{.Release.Name}}"
  cleanup:
    image: bitnami/kubectl:1.23

service:
  type: ClusterIP
  port: 80

database:
  name: myapp
  user: appuser
  password: apppassword
```

## Hook Examples

### Pre-Install: Database Initialization

This hook runs a Job to initialize the database before the main application is installed.

```yaml
# templates/hooks/pre-install-db-init.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: "{{ .Release.Name }}-db-init"
  annotations:
    "helm.sh/hook": pre-install
    "helm.sh/hook-weight": "0"
    "helm.sh/hook-delete-policy": hook-succeeded
  labels:
    app.kubernetes.io/managed-by: {{ .Release.Service | quote }}
    app.kubernetes.io/instance: {{ .Release.Name | quote }}
    app.kubernetes.io/version: {{ .Chart.AppVersion | quote }}
    helm.sh/chart: "{{ .Chart.Name }}-{{ .Chart.Version }}"
spec:
  template:
    metadata:
      name: "{{ .Release.Name }}-db-init"
      labels:
        app.kubernetes.io/managed-by: {{ .Release.Service | quote }}
        app.kubernetes.io/instance: {{ .Release.Name | quote }}
    spec:
      restartPolicy: Never
      containers:
        - name: db-init
          image: "{{ .Values.hooks.dbInit.image }}"
          env:
            - name: PGHOST
              value: postgres-service
            - name: PGUSER
              value: "{{ .Values.database.user }}"
            - name: PGPASSWORD
              value: "{{ .Values.database.password }}"
            - name: PGDATABASE
              value: "{{ .Values.database.name }}"
          command:
            - "/bin/bash"
            - "-c"
            - |
              psql <<EOF
              CREATE TABLE IF NOT EXISTS users (
                id SERIAL PRIMARY KEY,
                name VARCHAR(100) NOT NULL,
                email VARCHAR(100) UNIQUE NOT NULL,
                created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
              );
              
              CREATE TABLE IF NOT EXISTS items (
                id SERIAL PRIMARY KEY,
                user_id INTEGER REFERENCES users(id),
                title VARCHAR(255) NOT NULL,
                description TEXT,
                created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
              );
              
              INSERT INTO users (name, email) 
              VALUES ('Admin User', 'admin@example.com')
              ON CONFLICT (email) DO NOTHING;
              EOF
```

### Post-Install: Notification

This hook sends a notification after the application is successfully installed.

```yaml
# templates/hooks/post-install-notification.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: "{{ .Release.Name }}-notification"
  annotations:
    "helm.sh/hook": post-install
    "helm.sh/hook-weight": "0"
    "helm.sh/hook-delete-policy": hook-succeeded
  labels:
    app.kubernetes.io/managed-by: {{ .Release.Service | quote }}
    app.kubernetes.io/instance: {{ .Release.Name | quote }}
    helm.sh/chart: "{{ .Chart.Name }}-{{ .Chart.Version }}"
spec:
  template:
    metadata:
      name: "{{ .Release.Name }}-notification"
      labels:
        app.kubernetes.io/managed-by: {{ .Release.Service | quote }}
        app.kubernetes.io/instance: {{ .Release.Name | quote }}
    spec:
      restartPolicy: Never
      containers:
        - name: notification
          image: "{{ .Values.hooks.notification.image }}"
          command:
            - "/bin/sh"
            - "-c"
            - |
              curl -X POST -H 'Content-type: application/json' \
              --data '{"text":"Application {{ .Release.Name }} has been successfully deployed!"}' \
              {{ .Values.hooks.notification.endpoint }}
```

### Pre-Delete: Backup

This hook performs a backup before deleting the release.

```yaml
# templates/hooks/pre-delete-backup.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: "{{ .Release.Name }}-backup"
  annotations:
    "helm.sh/hook": pre-delete
    "helm.sh/hook-weight": "0"
    "helm.sh/hook-delete-policy": hook-succeeded
  labels:
    app.kubernetes.io/managed-by: {{ .Release.Service | quote }}
    app.kubernetes.io/instance: {{ .Release.Name | quote }}
    helm.sh/chart: "{{ .Chart.Name }}-{{ .Chart.Version }}"
spec:
  template:
    metadata:
      name: "{{ .Release.Name }}-backup"
      labels:
        app.kubernetes.io/managed-by: {{ .Release.Service | quote }}
        app.kubernetes.io/instance: {{ .Release.Name | quote }}
    spec:
      restartPolicy: Never
      containers:
        - name: backup
          image: "{{ .Values.hooks.backup.image }}"
          env:
            - name: PGHOST
              value: postgres-service
            - name: PGUSER
              value: "{{ .Values.database.user }}"
            - name: PGPASSWORD
              value: "{{ .Values.database.password }}"
            - name: PGDATABASE
              value: "{{ .Values.database.name }}"
            - name: BACKUP_DEST
              value: "{{ .Values.hooks.backup.destination }}"
          command:
            - "/bin/bash"
            - "-c"
            - |
              echo "Creating backup before deletion..."
              TIMESTAMP=$(date +%Y%m%d-%H%M%S)
              pg_dump -Fc > /tmp/backup-${TIMESTAMP}.dump
              
              # Upload to backup destination - in a real scenario, you would use
              # AWS CLI, GCP gsutil, or another appropriate tool
              echo "Backup created at /tmp/backup-${TIMESTAMP}.dump"
              echo "In a production environment, this would be uploaded to ${BACKUP_DEST}"
```

### Post-Delete: Cleanup

This hook performs cleanup tasks after the release is deleted.

```yaml
# templates/hooks/post-delete-cleanup.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: "{{ .Release.Name }}-cleanup"
  annotations:
    "helm.sh/hook": post-delete
    "helm.sh/hook-weight": "0"
    "helm.sh/hook-delete-policy": hook-succeeded
  labels:
    app.kubernetes.io/managed-by: {{ .Release.Service | quote }}
    app.kubernetes.io/instance: {{ .Release.Name | quote }}
    helm.sh/chart: "{{ .Chart.Name }}-{{ .Chart.Version }}"
spec:
  template:
    metadata:
      name: "{{ .Release.Name }}-cleanup"
      labels:
        app.kubernetes.io/managed-by: {{ .Release.Service | quote }}
        app.kubernetes.io/instance: {{ .Release.Name | quote }}
    spec:
      serviceAccountName: helm-cleanup-sa
      restartPolicy: Never
      containers:
        - name: cleanup
          image: "{{ .Values.hooks.cleanup.image }}"
          command:
            - "/bin/bash"
            - "-c"
            - |
              echo "Performing post-deletion cleanup..."
              # Find any orphaned PVCs
              kubectl get pvc -l app.kubernetes.io/instance={{ .Release.Name }} -o name | xargs -r kubectl delete
              
              # Find any orphaned Secrets not managed by Helm
              kubectl get secret -l app.kubernetes.io/instance={{ .Release.Name }} -o name | xargs -r kubectl delete
              
              echo "Cleanup completed!"
```

### Service Account for Cleanup

The cleanup job needs a ServiceAccount with appropriate permissions:

```yaml
# templates/hooks/cleanup-serviceaccount.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: helm-cleanup-sa
  annotations:
    "helm.sh/hook": pre-install
    "helm.sh/hook-weight": "-10"
    "helm.sh/hook-delete-policy": before-hook-creation
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: helm-cleanup-role
  annotations:
    "helm.sh/hook": pre-install
    "helm.sh/hook-weight": "-9"
    "helm.sh/hook-delete-policy": before-hook-creation
rules:
- apiGroups: [""]
  resources: ["persistentvolumeclaims", "secrets"]
  verbs: ["get", "list", "delete"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: helm-cleanup-binding
  annotations:
    "helm.sh/hook": pre-install
    "helm.sh/hook-weight": "-8"
    "helm.sh/hook-delete-policy": before-hook-creation
subjects:
- kind: ServiceAccount
  name: helm-cleanup-sa
roleRef:
  kind: Role
  name: helm-cleanup-role
  apiGroup: rbac.authorization.k8s.io
```

## Hook Weights

The `helm.sh/hook-weight` annotation determines the order of execution for hooks of the same type. Hooks with lower weights execute first.

## Hook Delete Policies

The `helm.sh/hook-delete-policy` annotation determines when hooks are deleted:

- `hook-succeeded`: Delete after hook succeeds
- `hook-failed`: Delete after hook fails
- `before-hook-creation`: Delete before creating a new hook

## Main Application Templates

### Deployment

```yaml
# templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "database-app.fullname" . }}
  labels:
    {{- include "database-app.labels" . | nindent 4 }}
spec:
  replicas: 1
  selector:
    matchLabels:
      {{- include "database-app.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      labels:
        {{- include "database-app.selectorLabels" . | nindent 8 }}
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

### Service

```yaml
# templates/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ include "database-app.fullname" . }}
  labels:
    {{- include "database-app.labels" . | nindent 4 }}
spec:
  type: {{ .Values.service.type }}
  ports:
    - port: {{ .Values.service.port }}
      targetPort: http
      protocol: TCP
      name: http
  selector:
    {{- include "database-app.selectorLabels" . | nindent 4 }}
```

## Testing Hooks

### Test Hook Example

You can also define test hooks that run when you execute `helm test`:

```yaml
# templates/tests/test-connection.yaml
apiVersion: v1
kind: Pod
metadata:
  name: "{{ include "database-app.fullname" . }}-test-connection"
  labels:
    {{- include "database-app.labels" . | nindent 4 }}
  annotations:
    "helm.sh/hook": test
spec:
  containers:
    - name: wget
      image: busybox
      command: ['wget']
      args: ['{{ include "database-app.fullname" . }}:{{ .Values.service.port }}']
  restartPolicy: Never
```

## Using the Chart

### Installing the Chart

```bash
# Install the chart
helm install my-app ./database-app
```

### Testing the Chart

```bash
# Test the installation
helm test my-app
```

### Upgrading the Chart

```bash
# Upgrade the chart
helm upgrade my-app ./database-app --set image.tag=1.22.0
```

### Uninstalling the Chart

```bash
# Uninstall the chart
helm uninstall my-app
```

## Hook Debugging

To debug hooks, you can check the created jobs:

```bash
# List all jobs
kubectl get jobs

# Check the logs of a specific hook job
kubectl logs job/my-app-db-init
```

You can also check the annotations on your hooks:

```bash
kubectl get job my-app-db-init -o jsonpath='{.metadata.annotations}'
```

## Best Practices for Hooks

1. **Use Idempotent Operations**: Hooks should be idempotent to handle multiple installations or upgrades
2. **Set Appropriate Delete Policies**: Clean up hook resources with appropriate delete policies
3. **Use Weights for Ordering**: Set weights to control the order of execution
4. **Create ServiceAccounts**: Use dedicated ServiceAccounts with minimal permissions for hooks
5. **Handle Failures Gracefully**: Ensure hooks fail clearly and don't leave the system in an inconsistent state
6. **Use Namespaces**: Use release namespaces for hook resources
7. **Log Verbosely**: Include clear logging in hook jobs for troubleshooting
8. **Test Hooks Thoroughly**: Test all hooks to ensure they work as expected

## Notes

- Hooks are not affected by the `--timeout` flag during installation. They can run indefinitely if not properly managed.
- Hooks don't participate in Helm's upgrade diffing. Every hook resource is recreated by default during upgrades.
- Failed hooks will cause the release to be marked as failed, but the resources that were created up to that point will NOT be rolled back.
- To ensure clean removal of all resources, consider using post-delete hooks with appropriate service accounts. 