# Grafana Dashboards for Kubernetes Monitoring

This example demonstrates how to create, customize, and manage Grafana dashboards for monitoring Kubernetes clusters and applications.

## Overview

Grafana dashboards provide a visual representation of metrics collected from various sources. When combined with Prometheus in Kubernetes, you can create powerful visualizations to monitor:

- Cluster health and performance
- Node resource utilization
- Pod resource usage and availability
- Application-specific metrics
- Network traffic and latency

## Prerequisites

- A Kubernetes cluster
- Prometheus and Grafana installed (see the [01-prometheus-grafana](../01-prometheus-grafana/) example)
- Basic understanding of PromQL (Prometheus Query Language)

## Dashboard Types

### 1. Infrastructure Dashboards

These dashboards focus on Kubernetes infrastructure metrics:
- Node CPU, memory, disk, and network usage
- Cluster capacity and resource allocation
- Pod resource consumption

### 2. Application Dashboards

These dashboards focus on application-specific metrics:
- Request rates, latency, and error rates
- Business metrics (orders, users, etc.)
- Database performance and query statistics

### 3. Service Mesh Dashboards

If using a service mesh like Istio:
- Request volume, success rate, and latency by service
- Circuit breaker status
- mTLS encryption metrics

## Accessing Grafana

If you've followed the Prometheus-Grafana setup in the previous example, you can access Grafana:

```bash
# Port forward to Grafana
kubectl port-forward -n monitoring svc/prometheus-grafana 3000:80
```

Open your browser and navigate to `http://localhost:3000`. The default credentials are:
- Username: `admin`
- Password: `prom-operator` (unless you've changed it)

## Creating Dashboards

### Method 1: Using the Grafana UI

1. Log in to Grafana
2. Click the "+" icon in the sidebar and select "Dashboard"
3. Click "Add new panel"
4. Configure your panel:
   - Select Prometheus as the data source
   - Write a PromQL query
   - Choose visualization type (graph, gauge, etc.)
   - Set time range and refresh intervals
5. Save the panel
6. Add more panels as needed
7. Save the dashboard with a descriptive name

### Method 2: Importing JSON Dashboards

You can import pre-built dashboards:

1. Find a dashboard on [Grafana Labs](https://grafana.com/grafana/dashboards/)
2. Note the dashboard ID
3. In Grafana, click the "+" icon and select "Import"
4. Enter the dashboard ID
5. Select your Prometheus data source
6. Click "Import"

### Method 3: Declarative Configuration with ConfigMaps

For version-controlled, GitOps-friendly dashboard management:

```yaml
# kubernetes-dashboard.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: kubernetes-dashboard
  namespace: monitoring
  labels:
    grafana_dashboard: "true"  # Label for the Grafana sidecar to find
data:
  kubernetes-dashboard.json: |-
    {
      "annotations": {
        "list": [...]
      },
      "editable": true,
      "gnetId": null,
      "graphTooltip": 0,
      "id": null,
      "links": [],
      "panels": [
        {
          "aliasColors": {},
          "bars": false,
          "dashLength": 10,
          "dashes": false,
          "datasource": "Prometheus",
          "fill": 1,
          "fillGradient": 0,
          "gridPos": {
            "h": 8,
            "w": 12,
            "x": 0,
            "y": 0
          },
          "hiddenSeries": false,
          "id": 1,
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
              "expr": "sum(rate(container_cpu_usage_seconds_total{namespace=\"default\",container!=\"\"}[5m])) by (pod)",
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
      "refresh": "10s",
      "schemaVersion": 22,
      "style": "dark",
      "tags": [],
      "templating": {
        "list": []
      },
      "time": {
        "from": "now-6h",
        "to": "now"
      },
      "timepicker": {},
      "timezone": "",
      "title": "Kubernetes Pod Resources",
      "uid": "kubernetes-resources",
      "version": 1
    }
```

Apply the ConfigMap:

```bash
kubectl apply -f kubernetes-dashboard.yaml
```

Note: For this method to work, you need to have the Grafana sidecar configured to look for dashboards in ConfigMaps with the appropriate label.

## Essential Kubernetes Dashboards

### 1. Node Exporter Dashboard

This dashboard provides detailed metrics about nodes (CPU, memory, disk, network):

```bash
# Import dashboard ID 1860
```

### 2. Kubernetes Cluster Overview

A high-level view of your Kubernetes cluster:

```bash
# Import dashboard ID 315
```

### 3. Kubernetes Pods Dashboard

Detailed information about pods:

```bash
# Create a ConfigMap for the Pods dashboard
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: ConfigMap
metadata:
  name: kubernetes-pods-dashboard
  namespace: monitoring
  labels:
    grafana_dashboard: "true"
data:
  kubernetes-pods.json: |-
    {
      "annotations": {
        "list": []
      },
      "editable": true,
      "gnetId": null,
      "graphTooltip": 0,
      "id": null,
      "links": [],
      "panels": [
        {
          "aliasColors": {},
          "bars": false,
          "dashLength": 10,
          "dashes": false,
          "datasource": "Prometheus",
          "fill": 1,
          "fillGradient": 0,
          "gridPos": {
            "h": 8,
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
              "expr": "sum(kube_pod_container_resource_requests{resource=\"memory\",namespace=\"default\"}) by (pod)",
              "interval": "",
              "legendFormat": "{{pod}} - Request",
              "refId": "A"
            },
            {
              "expr": "sum(container_memory_usage_bytes{namespace=\"default\",container!=\"POD\",container!=\"\"}) by (pod_name)",
              "interval": "",
              "legendFormat": "{{pod_name}} - Usage",
              "refId": "B"
            }
          ],
          "thresholds": [],
          "timeFrom": null,
          "timeRegions": [],
          "timeShift": null,
          "title": "Memory Usage vs Request by Pod",
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
              "format": "bytes",
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
      "refresh": "10s",
      "schemaVersion": 22,
      "style": "dark",
      "tags": [],
      "templating": {
        "list": []
      },
      "time": {
        "from": "now-3h",
        "to": "now"
      },
      "timepicker": {},
      "timezone": "",
      "title": "Kubernetes Pods Memory",
      "uid": "kubernetes-pods-memory",
      "version": 1
    }
EOF
```

## Creating a Custom Application Dashboard

Here's how to create a dashboard for a custom application that exposes metrics:

### 1. Deploy a Sample Application with Metrics

```yaml
# metrics-demo-app.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: metrics-demo
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: metrics-demo
  template:
    metadata:
      labels:
        app: metrics-demo
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "8080"
        prometheus.io/path: "/metrics"
    spec:
      containers:
      - name: metrics-demo
        image: prom/prometheus:v2.35.0
        ports:
        - containerPort: 8080
        args:
        - --config.file=/etc/prometheus/prometheus.yml
        - --storage.tsdb.path=/prometheus
        - --web.console.libraries=/usr/share/prometheus/console_libraries
        - --web.console.templates=/usr/share/prometheus/consoles
        - --web.listen-address=:8080
---
apiVersion: v1
kind: Service
metadata:
  name: metrics-demo
  namespace: default
spec:
  selector:
    app: metrics-demo
  ports:
  - port: 8080
    targetPort: 8080
  type: ClusterIP
```

Apply the manifest:

```bash
kubectl apply -f metrics-demo-app.yaml
```

### 2. Create a Dashboard for the Application

```yaml
# app-dashboard.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-metrics-dashboard
  namespace: monitoring
  labels:
    grafana_dashboard: "true"
data:
  app-metrics.json: |-
    {
      "annotations": {
        "list": []
      },
      "editable": true,
      "gnetId": null,
      "graphTooltip": 0,
      "id": null,
      "links": [],
      "panels": [
        {
          "aliasColors": {},
          "bars": false,
          "dashLength": 10,
          "dashes": false,
          "datasource": "Prometheus",
          "description": "",
          "fill": 1,
          "fillGradient": 0,
          "gridPos": {
            "h": 8,
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
              "expr": "scrape_duration_seconds{job=\"kubernetes-service-endpoints\", namespace=\"default\", service=\"metrics-demo\"}",
              "interval": "",
              "legendFormat": "Scrape Duration",
              "refId": "A"
            }
          ],
          "thresholds": [],
          "timeFrom": null,
          "timeRegions": [],
          "timeShift": null,
          "title": "Metrics Scrape Duration",
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
              "format": "s",
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
        },
        {
          "aliasColors": {},
          "bars": false,
          "dashLength": 10,
          "dashes": false,
          "datasource": "Prometheus",
          "fill": 1,
          "fillGradient": 0,
          "gridPos": {
            "h": 8,
            "w": 12,
            "x": 12,
            "y": 0
          },
          "hiddenSeries": false,
          "id": 4,
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
              "expr": "up{job=\"kubernetes-service-endpoints\", namespace=\"default\", service=\"metrics-demo\"}",
              "interval": "",
              "legendFormat": "Up",
              "refId": "A"
            }
          ],
          "thresholds": [],
          "timeFrom": null,
          "timeRegions": [],
          "timeShift": null,
          "title": "Application Availability",
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
              "max": "1",
              "min": "0",
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
      "refresh": "10s",
      "schemaVersion": 22,
      "style": "dark",
      "tags": [],
      "templating": {
        "list": []
      },
      "time": {
        "from": "now-30m",
        "to": "now"
      },
      "timepicker": {},
      "timezone": "",
      "title": "Application Metrics Demo",
      "uid": "app-metrics-demo",
      "version": 1
    }
```

Apply the manifest:

```bash
kubectl apply -f app-dashboard.yaml
```

## Advanced Dashboard Features

### 1. Using Variables and Templates

Variables allow you to create dynamic dashboards:

```json
"templating": {
  "list": [
    {
      "allValue": null,
      "current": {
        "text": "All",
        "value": "$__all"
      },
      "datasource": "Prometheus",
      "definition": "label_values(kube_pod_info, namespace)",
      "hide": 0,
      "includeAll": true,
      "label": "Namespace",
      "multi": false,
      "name": "namespace",
      "options": [],
      "query": "label_values(kube_pod_info, namespace)",
      "refresh": 1,
      "regex": "",
      "skipUrlSync": false,
      "sort": 0,
      "tagValuesQuery": "",
      "tags": [],
      "tagsQuery": "",
      "type": "query",
      "useTags": false
    },
    {
      "allValue": null,
      "current": {
        "text": "All",
        "value": "$__all"
      },
      "datasource": "Prometheus",
      "definition": "label_values(kube_pod_info{namespace=\"$namespace\"}, pod)",
      "hide": 0,
      "includeAll": true,
      "label": "Pod",
      "multi": false,
      "name": "pod",
      "options": [],
      "query": "label_values(kube_pod_info{namespace=\"$namespace\"}, pod)",
      "refresh": 1,
      "regex": "",
      "skipUrlSync": false,
      "sort": 0,
      "tagValuesQuery": "",
      "tags": [],
      "tagsQuery": "",
      "type": "query",
      "useTags": false
    }
  ]
}
```

Then use the variables in queries:

```
sum(rate(container_cpu_usage_seconds_total{namespace="$namespace",pod="$pod",container!="POD",container!=""}[5m])) by (container)
```

### 2. Setting Up Alerts in Dashboards

You can create alerts directly from dashboard panels:

1. Edit a panel
2. Go to the "Alert" tab
3. Click "Create Alert"
4. Define conditions (e.g., "Avg() OF query(A,5m,now) IS ABOVE 0.8")
5. Set evaluation intervals
6. Add notifications

### 3. Dashboard Annotations

Annotations mark important events on dashboards:

```json
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
    },
    {
      "datasource": "Prometheus",
      "enable": true,
      "expr": "changes(kube_pod_container_status_restarts_total{namespace=\"$namespace\"}[1m]) > 0",
      "iconColor": "rgba(255, 96, 96, 1)",
      "name": "Pod Restarts",
      "showIn": 0,
      "step": "1m",
      "tagKeys": "pod",
      "textFormat": "Pod {{pod}} restarted",
      "titleFormat": "Pod Restart"
    }
  ]
}
```

## Exporting and Sharing Dashboards

### Exporting Dashboards

1. Open the dashboard you want to export
2. Click the share icon (next to the dashboard title)
3. Select the "Export" tab
4. Choose export format (JSON or JSON with default values)
5. Click "Save to file" to download the JSON file

### Sharing Dashboards

1. Upload your dashboard to [Grafana Labs](https://grafana.com/grafana/dashboards/)
2. Share the dashboard ID with others
3. For teams, use version control to track dashboard JSON files

## Dashboard Organization and Best Practices

### Folder Structure

Organize dashboards in folders by:
- Application/service
- Team ownership
- Environment (prod, staging, dev)

### Dashboard Naming Conventions

Adopt a consistent naming pattern:
- `[Team] - [Service/App] - [Purpose]`
- `Production - API - Overview`
- `Database - MySQL - Performance`

### Best Practices

1. **Start with templates**: Begin with pre-built dashboards and customize
2. **Use variables**: Make dashboards reusable across namespaces/apps
3. **Consistent time ranges**: Set default time ranges appropriate for metrics
4. **Add documentation**: Use panel descriptions and dashboard notes
5. **Balance information density**: Avoid cramming too many panels
6. **Use row collapsing**: Group related panels in collapsible rows
7. **Optimize queries**: Inefficient PromQL can slow down dashboards
8. **Version control**: Store dashboard JSONs in Git
9. **Test on small screens**: Ensure dashboards work on various displays
10. **Update regularly**: Review and update dashboards as systems evolve

## Troubleshooting

### Common Issues

1. **Data not showing**:
   - Check Prometheus connectivity
   - Verify metrics are being collected
   - Review PromQL syntax

2. **Dashboard loads slowly**:
   - Optimize complex queries
   - Reduce time range
   - Increase panel refresh intervals

3. **Incorrect metrics**:
   - Verify metric names and labels
   - Check that the right data source is selected
   - Review any transformations applied

## Conclusion

Grafana dashboards are powerful tools for visualizing Kubernetes metrics. By following the examples and best practices in this guide, you can create effective dashboards that provide valuable insights into your cluster and applications.

Effective dashboards should:
- Present relevant information clearly
- Enable quick problem identification
- Support drilling down into details
- Adapt to different monitoring needs
- Provide context through annotations and documentation

With proper dashboard design, your team can more easily understand system behavior, identify issues, and make data-driven decisions about your Kubernetes environment. 