# Setting Up Prometheus and Grafana in Kubernetes

This example demonstrates how to set up Prometheus and Grafana for monitoring a Kubernetes cluster using the Prometheus Operator through the kube-prometheus-stack Helm chart.

## Overview

The kube-prometheus-stack (formerly known as prometheus-operator) is a collection of Kubernetes manifests, Grafana dashboards, and Prometheus rules that provide easy to operate end-to-end Kubernetes cluster monitoring. It includes:

- Prometheus Operator
- Prometheus with pre-configured alerts
- Grafana with pre-configured dashboards
- Node exporter
- kube-state-metrics
- Alert Manager

## Prerequisites

- A running Kubernetes cluster
- Helm 3 installed
- kubectl installed and configured to access your cluster
- Basic knowledge of Kubernetes concepts

## Installation

### Step 1: Add the Prometheus Community Helm Repository

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

### Step 2: Create a Namespace for Monitoring

```bash
kubectl create namespace monitoring
```

### Step 3: Install kube-prometheus-stack

```bash
helm install prometheus prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --set prometheus.prometheusSpec.serviceMonitorSelectorNilUsesHelmValues=false \
  --set prometheus.prometheusSpec.podMonitorSelectorNilUsesHelmValues=false
```

### Step 4: Verify the Installation

Check if all the pods are running:

```bash
kubectl get pods -n monitoring
```

You should see pods for Prometheus, Grafana, Alert Manager, and more:

```
NAME                                                 READY   STATUS    RESTARTS   AGE
alertmanager-prometheus-kube-prometheus-alertmanager-0   2/2     Running   0          5m
prometheus-grafana-7d9574c4bf-jvg8h                     2/2     Running   0          5m
prometheus-kube-prometheus-operator-65b8b94d97-hx5mr    1/1     Running   0          5m
prometheus-kube-state-metrics-7c64748d54-6w28r          1/1     Running   0          5m
prometheus-prometheus-kube-prometheus-prometheus-0      2/2     Running   0          5m
prometheus-prometheus-node-exporter-2khpx              1/1     Running   0          5m
```

## Accessing the Dashboards

### Option 1: Port Forwarding

#### For Prometheus:

```bash
kubectl port-forward -n monitoring svc/prometheus-kube-prometheus-prometheus 9090:9090
```

Now you can access Prometheus at http://localhost:9090

#### For Grafana:

```bash
kubectl port-forward -n monitoring svc/prometheus-grafana 3000:80
```

Now you can access Grafana at http://localhost:3000

Default Grafana credentials:
- Username: admin
- Password: prom-operator (get it using the command below)

```bash
kubectl get secret -n monitoring prometheus-grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
```

### Option 2: Creating Ingress Resources (For Production)

For production environments, you might want to set up Ingress resources. Here's an example using Nginx Ingress Controller:

```yaml
# prometheus-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: prometheus-ingress
  namespace: monitoring
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
    nginx.ingress.kubernetes.io/use-regex: "true"
    nginx.ingress.kubernetes.io/rewrite-target: /$2
spec:
  ingressClassName: nginx
  rules:
  - host: monitoring.example.com
    http:
      paths:
      - path: /prometheus(/|$)(.*)
        pathType: Prefix
        backend:
          service:
            name: prometheus-kube-prometheus-prometheus
            port:
              number: 9090
```

```yaml
# grafana-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: grafana-ingress
  namespace: monitoring
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
spec:
  ingressClassName: nginx
  rules:
  - host: monitoring.example.com
    http:
      paths:
      - path: /grafana
        pathType: Prefix
        backend:
          service:
            name: prometheus-grafana
            port:
              number: 80
```

Apply these manifests:

```bash
kubectl apply -f prometheus-ingress.yaml
kubectl apply -f grafana-ingress.yaml
```

## Custom Configuration

### Customizing the Prometheus Installation

You can create a `values.yaml` file to customize the installation:

```yaml
# values.yaml
prometheus:
  prometheusSpec:
    retention: 15d
    resources:
      requests:
        memory: 2Gi
        cpu: 500m
      limits:
        memory: 3Gi
        cpu: 1000m
    storageSpec:
      volumeClaimTemplate:
        spec:
          storageClassName: standard
          accessModes: ["ReadWriteOnce"]
          resources:
            requests:
              storage: 50Gi

grafana:
  adminPassword: mySecurePassword
  persistence:
    enabled: true
    size: 10Gi
  additionalDataSources:
    - name: Loki
      type: loki
      url: http://loki-stack:3100
      access: proxy

alertmanager:
  config:
    global:
      resolve_timeout: 5m
    route:
      group_by: ['alertname', 'job']
      group_wait: 30s
      group_interval: 5m
      repeat_interval: 12h
      receiver: 'slack'
      routes:
      - match:
          alertname: Watchdog
        receiver: 'null'
    receivers:
    - name: 'null'
    - name: 'slack'
      slack_configs:
      - api_url: 'https://hooks.slack.com/services/T00000000/B00000000/XXXXXXXXXXXXXXXXXXXXXXXX'
        channel: '#monitoring'
        send_resolved: true
```

Then install or upgrade with the values file:

```bash
helm upgrade --install prometheus prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --values values.yaml
```

## Monitoring Applications with Prometheus

### ServiceMonitor Example

To monitor your applications with Prometheus, create a ServiceMonitor for your service:

```yaml
# my-app-servicemonitor.yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: my-app-monitor
  namespace: my-app-namespace
  labels:
    release: prometheus  # Match the label that Prometheus is looking for
spec:
  selector:
    matchLabels:
      app: my-app  # Should match your service's labels
  endpoints:
  - port: metrics  # The port name where metrics are exposed
    interval: 15s  # Scrape interval
    path: /metrics  # Path where metrics are exposed
  namespaceSelector:
    matchNames:
    - my-app-namespace
```

Apply the ServiceMonitor:

```bash
kubectl apply -f my-app-servicemonitor.yaml
```

### Example Application with Prometheus Metrics

Here's an example of a simple application that exposes Prometheus metrics:

```yaml
# prometheus-demo-app.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: prometheus-demo
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: prometheus-demo
  template:
    metadata:
      labels:
        app: prometheus-demo
    spec:
      containers:
      - name: prometheus-demo
        image: nilebox/prometheus-demo:latest
        ports:
        - containerPort: 8080
          name: metrics
---
apiVersion: v1
kind: Service
metadata:
  name: prometheus-demo
  namespace: default
  labels:
    app: prometheus-demo
spec:
  selector:
    app: prometheus-demo
  ports:
  - port: 8080
    targetPort: 8080
    name: metrics
---
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: prometheus-demo
  namespace: default
  labels:
    release: prometheus
spec:
  selector:
    matchLabels:
      app: prometheus-demo
  endpoints:
  - port: metrics
    interval: 15s
```

Apply the demo application:

```bash
kubectl apply -f prometheus-demo-app.yaml
```

## Creating Custom Grafana Dashboards

### Using the Grafana UI

1. Access Grafana using port forwarding or ingress
2. Log in with your credentials
3. Click on "+" > "Dashboard"
4. Add panels using PromQL queries
5. Save the dashboard

### Importing Dashboards as ConfigMaps

You can also define dashboards as ConfigMaps:

```yaml
# custom-dashboard.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-custom-dashboard
  namespace: monitoring
  labels:
    grafana_dashboard: "true"  # This label is important for automatic discovery
data:
  my-custom-dashboard.json: |-
    {
      "annotations": {
        "list": [
          {
            "builtIn": 1,
            "datasource": "-- Grafana --",
            "enable": true,
            "hide": true,
            "iconColor": "rgba(0, 211, 255, 1)",
            "name": "Annotations & Alerts",
            "type": "dashboard"
          }
        ]
      },
      "editable": true,
      "gnetId": null,
      "graphTooltip": 0,
      "id": 1,
      "links": [],
      "panels": [
        {
          "aliasColors": {},
          "bars": false,
          "dashLength": 10,
          "dashes": false,
          "datasource": "Prometheus",
          "fieldConfig": {
            "defaults": {
              "custom": {}
            },
            "overrides": []
          },
          "fill": 1,
          "fillGradient": 0,
          "gridPos": {
            "h": 9,
            "w": 12,
            "x": 0,
            "y": 0
          },
          "hiddenSeries": false,
          "id": 2,
          "legend": {
            "avg": false,
            "current": false,
            "max": false,
            "min": false,
            "show": true,
            "total": false,
            "values": false
          },
          "lines": true,
          "linewidth": 1,
          "nullPointMode": "null",
          "options": {
            "dataLinks": []
          },
          "percentage": false,
          "pointradius": 2,
          "points": false,
          "renderer": "flot",
          "seriesOverrides": [],
          "spaceLength": 10,
          "stack": false,
          "steppedLine": false,
          "targets": [
            {
              "expr": "sum(rate(container_cpu_usage_seconds_total{namespace=\"default\"}[5m])) by (pod)",
              "interval": "",
              "legendFormat": "{{pod}}",
              "refId": "A"
            }
          ],
          "thresholds": [],
          "timeFrom": null,
          "timeRegions": [],
          "timeShift": null,
          "title": "CPU Usage by Pod",
          "tooltip": {
            "shared": true,
            "sort": 0,
            "value_type": "individual"
          },
          "type": "graph",
          "xaxis": {
            "buckets": null,
            "mode": "time",
            "name": null,
            "show": true,
            "values": []
          },
          "yaxes": [
            {
              "format": "short",
              "label": null,
              "logBase": 1,
              "max": null,
              "min": null,
              "show": true
            },
            {
              "format": "short",
              "label": null,
              "logBase": 1,
              "max": null,
              "min": null,
              "show": true
            }
          ],
          "yaxis": {
            "align": false,
            "alignLevel": null
          }
        }
      ],
      "schemaVersion": 25,
      "style": "dark",
      "tags": [],
      "templating": {
        "list": []
      },
      "time": {
        "from": "now-6h",
        "to": "now"
      },
      "timepicker": {
        "refresh_intervals": [
          "5s",
          "10s",
          "30s",
          "1m",
          "5m",
          "15m",
          "30m",
          "1h",
          "2h",
          "1d"
        ]
      },
      "timezone": "",
      "title": "My Custom Dashboard",
      "uid": "custom-dashboard",
      "version": 1
    }
```

Apply the dashboard ConfigMap:

```bash
kubectl apply -f custom-dashboard.yaml
```

## Setting Up Alerting

### PrometheusRule Example

Define alerting rules with PrometheusRule resources:

```yaml
# custom-alerts.yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: custom-prometheus-rules
  namespace: monitoring
  labels:
    app: kube-prometheus-stack
    release: prometheus
spec:
  groups:
  - name: custom.rules
    rules:
    - alert: HighCPUUsage
      expr: sum(rate(container_cpu_usage_seconds_total{container!=""}[5m])) by (pod, namespace) > 0.5
      for: 10m
      labels:
        severity: warning
      annotations:
        summary: High CPU usage detected
        description: "Pod {{ $labels.pod }} in namespace {{ $labels.namespace }} has been using more than 50% CPU for the last 10 minutes."
    - alert: MemoryUsageHigh
      expr: sum(container_memory_usage_bytes{container!=""}) by (pod, namespace) / sum(container_spec_memory_limit_bytes{container!=""}) by (pod, namespace) * 100 > 80
      for: 10m
      labels:
        severity: warning
      annotations:
        summary: High memory usage detected
        description: "Pod {{ $labels.pod }} in namespace {{ $labels.namespace }} is using more than 80% of its memory limit."
    - alert: PodCrashLooping
      expr: increase(kube_pod_container_status_restarts_total[1h]) > 5
      for: 10m
      labels:
        severity: critical
      annotations:
        summary: Pod is crash looping
        description: "Pod {{ $labels.pod }} in namespace {{ $labels.namespace }} is crash looping ({{ $value }} restarts in the last hour)."
```

Apply the alerting rules:

```bash
kubectl apply -f custom-alerts.yaml
```

### Configuring AlertManager

Configure AlertManager to send notifications to external services like Slack, PagerDuty, or email. Here's an example for Slack:

Create a `alertmanager-config.yaml` file:

```yaml
# alertmanager-config.yaml
apiVersion: v1
kind: Secret
metadata:
  name: alertmanager-prometheus-kube-prometheus-alertmanager
  namespace: monitoring
stringData:
  alertmanager.yaml: |-
    global:
      resolve_timeout: 5m
      slack_api_url: 'https://hooks.slack.com/services/T00000000/B00000000/XXXXXXXXXXXXXXXXXXXXXXXX'
    
    route:
      group_by: ['alertname', 'job', 'severity']
      group_wait: 30s
      group_interval: 5m
      repeat_interval: 12h
      receiver: 'slack-notifications'
      routes:
      - match:
          alertname: Watchdog
        receiver: 'null'
      - match:
          severity: critical
        receiver: 'slack-critical'
        continue: true
    
    receivers:
    - name: 'null'
    - name: 'slack-notifications'
      slack_configs:
      - channel: '#monitoring'
        send_resolved: true
        title: '[{{ .Status | toUpper }}{{ if eq .Status "firing" }}:{{ .Alerts.Firing | len }}{{ end }}] Monitoring Alert'
        text: >-
          {{ range .Alerts }}
            *Alert:* {{ .Labels.alertname }}{{ if .Labels.severity }} - `{{ .Labels.severity }}`{{ end }}
            *Description:* {{ .Annotations.description }}
            *Details:*
            {{ range .Labels.SortedPairs }} • *{{ .Name }}:* `{{ .Value }}`
            {{ end }}
          {{ end }}
    - name: 'slack-critical'
      slack_configs:
      - channel: '#alerts-critical'
        send_resolved: true
        title: '[CRITICAL - {{ .Status | toUpper }}] Monitoring Alert'
        text: >-
          {{ range .Alerts }}
            *Alert:* {{ .Labels.alertname }}
            *Description:* {{ .Annotations.description }}
            *Details:*
            {{ range .Labels.SortedPairs }} • *{{ .Name }}:* `{{ .Value }}`
            {{ end }}
          {{ end }}
```

Apply the AlertManager configuration:

```bash
kubectl apply -f alertmanager-config.yaml
```

Restart AlertManager:

```bash
kubectl rollout restart statefulset/alertmanager-prometheus-kube-prometheus-alertmanager -n monitoring
```

## Cleanup

To remove the Prometheus and Grafana stack:

```bash
helm uninstall prometheus -n monitoring
```

If you used persistent volumes, you might want to clean them up as well:

```bash
kubectl delete pvc -l release=prometheus -n monitoring
```

## Troubleshooting

### Common Issues and Solutions

1. **Pods are not running**
   
   Check for pod status and errors:
   ```bash
   kubectl get pods -n monitoring
   kubectl describe pod <pod-name> -n monitoring
   ```

2. **Prometheus can't scrape metrics**
   
   Check if ServiceMonitor is correctly configured:
   ```bash
   kubectl get servicemonitor -A
   kubectl describe servicemonitor <servicemonitor-name> -n <namespace>
   ```

3. **Grafana dashboards not showing up**
   
   Check if ConfigMaps have the correct label:
   ```bash
   kubectl get configmap -n monitoring -l grafana_dashboard=true
   ```

4. **AlertManager not sending notifications**
   
   Check AlertManager configuration and logs:
   ```bash
   kubectl logs -f alertmanager-prometheus-kube-prometheus-alertmanager-0 -n monitoring -c alertmanager
   ```

## Best Practices

1. **Resource Planning**: Allocate sufficient resources for Prometheus, especially when monitoring large clusters
2. **Retention Period**: Configure appropriate data retention based on your needs and storage capacity
3. **Scrape Intervals**: Balance between freshness of data and load on your cluster
4. **Dashboard Organization**: Use folders and naming conventions for better organization
5. **Security**: Use secure passwords, network policies, and RBAC
6. **Backup**: Regularly backup Grafana dashboards and Prometheus rules
7. **Alerts**: Define meaningful alert thresholds and avoid alert fatigue

## Conclusion

You now have a fully functional Prometheus and Grafana setup for monitoring your Kubernetes cluster. This setup provides:

- Metrics collection and storage with Prometheus
- Visualization with Grafana
- Alerting with AlertManager
- Pre-configured dashboards for Kubernetes

You can extend this setup with additional exporters, custom metrics, and integrations with other monitoring systems. 