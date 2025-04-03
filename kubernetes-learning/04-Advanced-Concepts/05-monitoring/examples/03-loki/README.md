# Log Aggregation with Loki in Kubernetes

This example demonstrates how to set up Grafana Loki for centralized log aggregation in a Kubernetes cluster, including deploying the Loki stack, configuring log sources, and exploring logs in Grafana.

## Overview

Loki is a horizontally-scalable, highly-available, multi-tenant log aggregation system designed specifically for Kubernetes. It's inspired by Prometheus but for logs instead of metrics, using the same label-based approach that makes it efficient and cost-effective.

Key benefits of Loki include:

- **Label-based indexing**: Like Prometheus, Loki indexes by metadata labels rather than full-text indexing, making it efficient and cost-effective
- **Kubernetes-native**: Designed to work seamlessly with Kubernetes
- **Integration with Grafana**: Native support in Grafana for querying and visualizing logs
- **Multi-tenancy**: Supports multiple tenants with isolation
- **Horizontally scalable**: Can scale to handle large volumes of logs

## Architecture

A typical Loki deployment consists of:

1. **Promtail**: Agent that collects logs from containers and ships them to Loki
2. **Loki**: Log aggregation system that stores and indexes logs
3. **Grafana**: Visualization platform to query and display logs

## Prerequisites

- A Kubernetes cluster
- Helm 3 installed
- kubectl installed and configured
- If you've deployed Prometheus and Grafana from the [first example](../01-prometheus-grafana/), you can reuse that Grafana instance

## Installation

### Option 1: Using Helm Chart (Recommended)

The Grafana Loki Helm chart provides a comprehensive deployment of the Loki stack:

```bash
# Add the Grafana Helm repository
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update

# Create a namespace for Loki
kubectl create namespace logging

# Install Loki stack (includes Loki, Promtail, and optionally Grafana)
helm install loki-stack grafana/loki-stack \
  --namespace logging \
  --set promtail.enabled=true \
  --set loki.persistence.enabled=true \
  --set loki.persistence.size=10Gi \
  --set grafana.enabled=false  # Set to true if you don't have Grafana already
```

### Option 2: Using Loki Operator (Advanced)

For more complex deployments, you can use the Loki Operator:

```bash
# Install the Loki Operator
kubectl apply -f https://raw.githubusercontent.com/grafana/loki/main/operator/bundle.yaml

# Create a simple LokiStack instance
cat <<EOF | kubectl apply -f -
apiVersion: loki.grafana.com/v1
kind: LokiStack
metadata:
  name: logging
  namespace: logging
spec:
  size: 1x.small
  storage:
    schemas:
    - version: v12
      effectiveDate: "2022-06-01"
    secret:
      name: loki-s3
      type: s3
EOF
```

### Verify the Installation

Check that all components are running:

```bash
kubectl get pods -n logging

# Example output
NAME                            READY   STATUS    RESTARTS   AGE
loki-stack-0                    1/1     Running   0          2m
loki-stack-promtail-abcd12345   1/1     Running   0          2m
loki-stack-promtail-efgh67890   1/1     Running   0          2m
```

## Configuring Promtail

Promtail is responsible for collecting and shipping logs to Loki. It runs as a DaemonSet, ensuring that logs from all nodes are collected.

### Default Configuration

The default Promtail configuration:

- Collects logs from all containers using the Kubernetes API
- Adds Kubernetes metadata as labels (namespace, pod, container)
- Follows standard log paths for containerized applications
- Ships logs to Loki with appropriate scrape configurations

### Custom Promtail Configuration

To customize Promtail, create a `values.yaml` file:

```yaml
# custom-promtail-values.yaml
promtail:
  config:
    lokiAddress: http://loki-stack:3100/loki/api/v1/push
    
    scrapeConfigs:
      # Standard Kubernetes pods scrape config
      - job_name: kubernetes-pods
        kubernetes_sd_configs:
          - role: pod
        relabel_configs:
          - source_labels: 
              - __meta_kubernetes_pod_controller_name
            regex: ([0-9a-z-.]+?)(-[0-9a-f]{8,10})?
            action: replace
            target_label: __tmp_controller_name
          - source_labels:
              - __meta_kubernetes_pod_label_app_kubernetes_io_name
              - __meta_kubernetes_pod_label_app
              - __tmp_controller_name
              - __meta_kubernetes_pod_name
            regex: ^;*([^;]+)(;.*)?$
            action: replace
            target_label: app
          - source_labels:
              - __meta_kubernetes_pod_label_app_kubernetes_io_component
              - __meta_kubernetes_pod_label_component
            regex: ^;*([^;]+)(;.*)?$
            action: replace
            target_label: component
          - action: replace
            source_labels:
            - __meta_kubernetes_pod_node_name
            target_label: node_name
          - action: replace
            source_labels:
            - __meta_kubernetes_namespace
            target_label: namespace
          - action: replace
            replacement: $1
            separator: /
            source_labels:
            - namespace
            - app
            target_label: job
          - action: replace
            source_labels:
            - __meta_kubernetes_pod_name
            target_label: pod
          - action: replace
            source_labels:
            - __meta_kubernetes_pod_container_name
            target_label: container
          - action: replace
            replacement: /var/log/pods/*$1/*.log
            separator: /
            source_labels:
            - __meta_kubernetes_pod_uid
            - __meta_kubernetes_pod_container_name
            target_label: __path__
          - action: replace
            regex: true
            source_labels:
            - __meta_kubernetes_pod_label_scrape_logs
            target_label: __tmp_scrape_logs
        
        # Add pipeline stages for parsing specific log formats 
        pipeline_stages:
          # Example: Parse JSON logs
          - json:
              expressions:
                level: level
                message: message
                timestamp: timestamp
          # Example: Add severity label based on log level
          - labels:
              level:
```

Apply the custom configuration:

```bash
helm upgrade loki-stack grafana/loki-stack \
  --namespace logging \
  -f custom-promtail-values.yaml
```

## Example Applications with Structured Logging

Let's deploy a sample application that generates structured logs:

```yaml
# structured-logging-app.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: structured-logger
  namespace: logging-demo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: structured-logger
  template:
    metadata:
      labels:
        app: structured-logger
    spec:
      containers:
      - name: logger
        image: busybox
        command:
        - /bin/sh
        - -c
        - |
          while true; do
            timestamp=$(date -u +"%Y-%m-%dT%H:%M:%SZ")
            level="INFO"
            if [ $((RANDOM % 10)) -eq 0 ]; then
              level="ERROR"
            fi
            message="Sample log message #$((i++))"
            echo "{\"timestamp\":\"$timestamp\",\"level\":\"$level\",\"message\":\"$message\",\"service\":\"demo-app\"}"
            sleep 1
          done
---
apiVersion: v1
kind: Namespace
metadata:
  name: logging-demo
```

Apply the manifest:

```bash
kubectl apply -f structured-logging-app.yaml
```

## Connecting Loki to Grafana

If you've already deployed Grafana from the first example, you need to add Loki as a data source:

### Automatic Configuration (via ConfigMap)

```yaml
# loki-datasource.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: loki-datasource
  namespace: monitoring  # Same namespace as your Grafana
  labels:
    grafana_datasource: "1"
data:
  loki-datasource.yaml: |-
    apiVersion: 1
    datasources:
    - name: Loki
      type: loki
      url: http://loki-stack.logging:3100
      access: proxy
      isDefault: false
      version: 1
```

Apply the ConfigMap:

```bash
kubectl apply -f loki-datasource.yaml
```

### Manual Configuration

1. Port-forward to Grafana:
   ```bash
   kubectl port-forward -n monitoring svc/prometheus-grafana 3000:80
   ```

2. Open Grafana in your browser (`http://localhost:3000`)

3. Go to Configuration > Data Sources > Add data source

4. Select Loki

5. Set the URL to `http://loki-stack.logging:3100`

6. Click "Save & Test"

## Querying Logs in Grafana

Loki uses a query language called LogQL, which is similar to PromQL.

### Basic Queries

Open Grafana's Explore view and try these queries:

1. Show all logs:
   ```
   {app="structured-logger"}
   ```

2. Filter by log level:
   ```
   {app="structured-logger", level="ERROR"}
   ```

3. Search for a specific term:
   ```
   {app="structured-logger"} |= "Sample"
   ```

### Advanced LogQL

1. Extract fields from JSON logs:
   ```
   {app="structured-logger"} | json
   ```

2. Filter based on extracted field:
   ```
   {app="structured-logger"} | json | level="ERROR"
   ```

3. Count logs by level:
   ```
   sum by(level) (count_over_time({app="structured-logger"} | json [1m]))
   ```

## Creating Dashboards for Logs

Create a Grafana dashboard to monitor your logs:

1. Create a new dashboard
2. Add a logs panel:
   - Title: "Application Logs"
   - Data source: Loki
   - Query: `{app="structured-logger"} | json`

3. Add a graph panel for log rates:
   - Title: "Log Rate by Level"
   - Data source: Loki
   - Query: `sum by(level) (rate({app="structured-logger"} | json [1m]))`

4. Add a stat panel for error count:
   - Title: "Error Count (Last 5m)"
   - Data source: Loki
   - Query: `sum(count_over_time({app="structured-logger", level="ERROR"} | json [5m]))`

## Log Retention and Storage

### Configuring Retention

Loki's retention period can be configured in the values file:

```yaml
# loki-retention-values.yaml
loki:
  config:
    table_manager:
      retention_deletes_enabled: true
      retention_period: 168h  # 7 days
```

Apply the configuration:

```bash
helm upgrade loki-stack grafana/loki-stack \
  --namespace logging \
  -f loki-retention-values.yaml
```

### Storage Considerations

For production environments:

1. **Storage Backend**: Configure a durable storage backend like S3, GCS, or Azure Blob Storage
2. **Resource Requirements**: Adjust resource requests and limits based on log volume
3. **Indexes**: Monitor index size and adjust retention separately from logs if needed

Example S3 configuration:

```yaml
# loki-s3-values.yaml
loki:
  config:
    schema_config:
      configs:
        - from: 2020-10-24
          store: boltdb-shipper
          object_store: s3
          schema: v11
          index:
            prefix: index_
            period: 24h
    storage_config:
      boltdb_shipper:
        active_index_directory: /data/loki/boltdb-shipper-active
        cache_location: /data/loki/boltdb-shipper-cache
        cache_ttl: 24h
        shared_store: s3
      aws:
        s3: s3://BUCKET_NAME:REGION
        s3forcepathstyle: true
        bucketnames: BUCKET_NAME
        access_key_id: ACCESS_KEY
        secret_access_key: SECRET_KEY
        s3: s3://BUCKET_NAME:REGION
```

## Integrating with Applications

### Log Format Best Practices

1. **Use structured logging**: Output logs in JSON format
2. **Include standard fields**:
   - timestamp (ISO8601 format)
   - level (INFO, WARN, ERROR)
   - message
   - service/component name

3. **Add contextual information**: 
   - request_id for tracing requests across services
   - user_id for user-related operations
   - relevant business metrics

### Example: Structured Logging in Node.js

```javascript
// Using Winston for structured logging
const winston = require('winston');

const logger = winston.createLogger({
  level: 'info',
  format: winston.format.json(),
  defaultMeta: { service: 'user-service' },
  transports: [
    new winston.transports.Console(),
  ],
});

// Example usage
app.get('/api/users', (req, res) => {
  logger.info('Fetching users', { 
    request_id: req.headers['x-request-id'],
    user_count: users.length,
    query_params: req.query
  });
  // ... handler code
});
```

### Example: Structured Logging in Go

```go
package main

import (
    "os"
    "github.com/sirupsen/logrus"
)

func main() {
    log := logrus.New()
    log.SetFormatter(&logrus.JSONFormatter{})
    log.SetOutput(os.Stdout)
    
    log.WithFields(logrus.Fields{
        "service": "order-service",
        "order_id": "12345",
        "amount": 42.95,
    }).Info("Order processed successfully")
    
    // ... application code
}
```

## Alert Rules for Logs

You can set up Grafana alerts based on log queries:

1. In Grafana, go to Alerting > Alert rules > New alert rule
2. Configure an alert based on a LogQL query:
   ```
   sum(count_over_time({app="structured-logger", level="ERROR"} [5m])) > 10
   ```
3. Set evaluation interval and notification channels

## Troubleshooting

### Common Issues

1. **Logs not appearing in Loki**:
   - Check that Promtail pods are running on all nodes
   - Verify Promtail can connect to Loki (check Promtail logs)
   - Ensure log paths are correctly configured

2. **High memory usage**:
   - Review label cardinality (too many unique label values)
   - Check chunk and index cache size configurations
   - Consider increasing resources or optimizing queries

3. **Slow queries**:
   - Narrow time ranges when querying
   - Add more specific label selectors
   - Use regex filters sparingly

### Debugging Commands

```bash
# Check Promtail logs
kubectl logs -n logging -l app=promtail

# Check Loki logs
kubectl logs -n logging service/loki-stack

# Verify Loki is receiving logs
kubectl port-forward -n logging service/loki-stack 3100:3100
curl http://localhost:3100/ready
curl http://localhost:3100/metrics | grep loki_distributor_received_samples_total
```

## Best Practices

1. **Control label cardinality**: Avoid using high-cardinality fields as labels
2. **Structure log messages**: Use structured logging formats like JSON
3. **Set appropriate retention**: Configure retention based on compliance and analysis needs
4. **Monitor Loki itself**: Set up alerts for Loki's health and performance
5. **Use LogQL pipelines efficiently**: Filter as early as possible in the query
6. **Implement log levels**: Use appropriate log levels to filter noise
7. **Correlate with metrics and traces**: Use common identifiers across observability signals
8. **Plan for scaling**: Consider how to scale Loki components as log volume grows

## Conclusion

You've now set up a complete log aggregation system with Loki in your Kubernetes cluster. This system provides:

- Centralized log collection and storage
- Powerful query capabilities through LogQL
- Integration with Grafana for visualization and alerting
- Efficient storage and indexing optimized for Kubernetes logs

This foundation can be extended with more advanced features like multi-tenancy, advanced parsing and additional log sources as your needs evolve. 