# Custom Metrics in Kubernetes

This example demonstrates how to implement, expose, and monitor custom metrics from applications running in Kubernetes and how to use these metrics with Prometheus and the Kubernetes Horizontal Pod Autoscaler.

## Overview

Custom metrics allow you to measure application-specific behavior beyond standard CPU and memory usage. They enable:

- Fine-grained monitoring of business-relevant metrics
- Autoscaling based on application-specific load indicators
- Better observability of application health and performance
- Enhanced alerting capabilities

## Prerequisites

- A Kubernetes cluster with Prometheus installed (see [Prometheus and Grafana example](../01-prometheus-grafana/))
- The Prometheus Adapter for custom metrics API
- Basic familiarity with Prometheus metrics and PromQL

## Types of Custom Metrics

### Application Metrics

These are specific to your application's behavior:

- Request counts and latencies
- Queue lengths
- Error rates
- Business metrics (e.g., orders processed, active users)

### System Metrics

These relate to your application's resource usage:

- Network connections
- File descriptors
- Thread counts
- Garbage collection metrics

## Implementing Custom Metrics

### Example Application: Order Processing Service

We'll implement a simple order processing service in Python using Flask and the Prometheus client library.

```python
# app.py
from flask import Flask, request, jsonify
from prometheus_client import Counter, Histogram, Gauge, Summary, generate_latest
from prometheus_client import CONTENT_TYPE_LATEST
import time
import random
import threading

app = Flask(__name__)

# Define Prometheus metrics
ORDERS_TOTAL = Counter('app_orders_total', 'Total number of orders received', ['status'])
ORDER_SIZE = Summary('app_order_size_dollars', 'Order size in dollars')
ITEMS_COUNT = Histogram('app_items_per_order', 'Number of items per order', buckets=[1, 2, 5, 10, 20, 50])
QUEUE_GAUGE = Gauge('app_order_queue_size', 'Current number of orders in processing queue')
PROCESSING_TIME = Histogram('app_order_processing_seconds', 'Time spent processing orders', 
                           buckets=[0.1, 0.5, 1, 2, 5, 10, 30])

# Simulated order queue
order_queue = []
queue_lock = threading.Lock()

@app.route('/metrics')
def metrics():
    return generate_latest(), 200, {'Content-Type': CONTENT_TYPE_LATEST}

@app.route('/order', methods=['POST'])
def create_order():
    # Simulate receiving an order
    data = request.json
    
    if not data or 'items' not in data:
        ORDERS_TOTAL.labels(status='invalid').inc()
        return jsonify({"error": "Invalid order data"}), 400
    
    # Record metrics about the order
    items_count = len(data['items'])
    order_total = sum(item.get('price', 0) for item in data['items'])
    
    # Update Prometheus metrics
    ORDERS_TOTAL.labels(status='received').inc()
    ORDER_SIZE.observe(order_total)
    ITEMS_COUNT.observe(items_count)
    
    # Add to processing queue
    with queue_lock:
        order_queue.append(data)
        current_queue_size = len(order_queue)
        QUEUE_GAUGE.set(current_queue_size)
    
    return jsonify({"status": "order received", "queue_position": current_queue_size}), 202

@app.route('/process', methods=['POST'])
def process_orders():
    """Endpoint to trigger order processing"""
    processed = 0
    errors = 0
    
    with queue_lock:
        if not order_queue:
            return jsonify({"status": "no orders to process"}), 200
            
        # Process up to 5 orders
        to_process = min(5, len(order_queue))
        
        for _ in range(to_process):
            if not order_queue:
                break
                
            # Process order and record metrics
            start_time = time.time()
            order = order_queue.pop(0)
            
            # Simulate processing time and success/failure
            time.sleep(random.uniform(0.1, 2))
            success = random.random() > 0.1  # 10% chance of failure
            
            processing_time = time.time() - start_time
            PROCESSING_TIME.observe(processing_time)
            
            if success:
                ORDERS_TOTAL.labels(status='processed').inc()
                processed += 1
            else:
                ORDERS_TOTAL.labels(status='failed').inc()
                errors += 1
        
        # Update queue gauge
        QUEUE_GAUGE.set(len(order_queue))
    
    return jsonify({
        "processed": processed,
        "errors": errors,
        "remaining": len(order_queue)
    }), 200

@app.route('/health')
def health():
    return jsonify({"status": "healthy"})

@app.route('/generate-load', methods=['POST'])
def generate_load():
    """Generate random orders for testing"""
    count = int(request.args.get('count', 10))
    
    for _ in range(count):
        items = []
        item_count = random.randint(1, 20)
        
        for i in range(item_count):
            items.append({
                "id": f"item-{i}",
                "name": f"Product {i}",
                "price": round(random.uniform(5, 100), 2)
            })
        
        with queue_lock:
            order_queue.append({"items": items})
            QUEUE_GAUGE.set(len(order_queue))
            
        ORDERS_TOTAL.labels(status='received').inc()
        ORDER_SIZE.observe(sum(item["price"] for item in items))
        ITEMS_COUNT.observe(item_count)
    
    return jsonify({"status": "load generated", "orders_created": count}), 201

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8080)
```

### Kubernetes Deployment

Create a Dockerfile for the application:

```dockerfile
# Dockerfile
FROM python:3.9-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY app.py .

EXPOSE 8080

CMD ["python", "app.py"]
```

Requirements file:

```
# requirements.txt
flask==2.0.1
prometheus-client==0.11.0
```

Deploy the application to Kubernetes:

```yaml
# custom-metrics-app.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: order-processor
  labels:
    app: order-processor
spec:
  replicas: 1
  selector:
    matchLabels:
      app: order-processor
  template:
    metadata:
      labels:
        app: order-processor
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/path: "/metrics"
        prometheus.io/port: "8080"
    spec:
      containers:
      - name: order-processor
        image: ${YOUR_REGISTRY}/order-processor:latest  # Replace with your image
        ports:
        - containerPort: 8080
        resources:
          limits:
            cpu: "500m"
            memory: "256Mi"
          requests:
            cpu: "100m"
            memory: "128Mi"
        readinessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 10
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 15
          periodSeconds: 20
---
apiVersion: v1
kind: Service
metadata:
  name: order-processor
  labels:
    app: order-processor
spec:
  ports:
  - port: 80
    targetPort: 8080
    name: http
  selector:
    app: order-processor
```

## ServiceMonitor for Prometheus Operator

If you're using the Prometheus Operator:

```yaml
# service-monitor.yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: order-processor
  labels:
    app: order-processor
    release: prometheus  # match your Prometheus Operator release label
spec:
  selector:
    matchLabels:
      app: order-processor
  endpoints:
  - port: http
    interval: 15s
```

## Testing Custom Metrics

### Generate Test Load

```bash
# Forward port to the service
kubectl port-forward svc/order-processor 8080:80

# Generate some test orders
curl -X POST http://localhost:8080/generate-load?count=50

# Process some orders
curl -X POST http://localhost:8080/process
```

### View Metrics

```bash
# View raw metrics from the application
curl http://localhost:8080/metrics

# Sample output
# HELP app_orders_total Total number of orders received
# TYPE app_orders_total counter
app_orders_total{status="received"} 50.0
app_orders_total{status="processed"} 12.0
app_orders_total{status="failed"} 3.0
# HELP app_order_queue_size Current number of orders in processing queue
# TYPE app_order_queue_size gauge
app_order_queue_size 35.0
# ...
```

### Querying Metrics in Prometheus

Access your Prometheus UI and try these queries:

- Total orders received: `sum(app_orders_total{status="received"})`
- Processing success rate: `sum(app_orders_total{status="processed"}) / sum(app_orders_total{status="received"} - app_orders_total{status="invalid"})`
- Average order size: `rate(app_order_size_dollars_sum[5m]) / rate(app_order_size_dollars_count[5m])`
- Current queue size: `app_order_queue_size`
- 95th percentile of processing time: `histogram_quantile(0.95, sum(rate(app_order_processing_seconds_bucket[5m])) by (le))`

## Custom Metrics Adapter for HPA

The Prometheus Adapter allows Kubernetes to use Prometheus metrics for Horizontal Pod Autoscaling.

### Installing the Prometheus Adapter

Using Helm:

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

helm install prometheus-adapter prometheus-community/prometheus-adapter \
  --namespace monitoring \
  --set prometheus.url=http://prometheus-server.monitoring.svc \
  --set prometheus.port=80 \
  -f custom-values.yaml
```

Create `custom-values.yaml` with your custom metrics configuration:

```yaml
# custom-values.yaml
rules:
  custom:
  - seriesQuery: 'app_order_queue_size{namespace!="",pod!=""}'
    resources:
      template: "<<.Resource>>"
    name:
      matches: "app_order_queue_size"
      as: "orders_queue_size"
    metricsQuery: 'sum(<<.Series>>{<<.LabelMatchers>>}) by (<<.GroupBy>>)'
```

### Creating an HPA with Custom Metrics

```yaml
# custom-metrics-hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: order-processor-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: order-processor
  minReplicas: 1
  maxReplicas: 10
  metrics:
  - type: Pods
    pods:
      metric:
        name: orders_queue_size  # This must match the name in the Prometheus Adapter config
      target:
        type: AverageValue
        averageValue: 10  # Scale up when there are more than 10 orders in queue per pod
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 60
      policies:
      - type: Pods
        value: 2
        periodSeconds: 60
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Pods
        value: 1
        periodSeconds: 60
```

Apply the HPA configuration:

```bash
kubectl apply -f custom-metrics-hpa.yaml
```

### Testing the HPA

```bash
# Generate a lot of load to trigger scaling
curl -X POST http://localhost:8080/generate-load?count=200

# Watch the HPA react
kubectl get hpa order-processor-hpa -w

# Watch the pods scale up
kubectl get pods -l app=order-processor -w
```

## Creating Grafana Dashboards for Custom Metrics

Create a Grafana dashboard to visualize your custom metrics:

1. Log into Grafana
2. Create a new dashboard
3. Add panels with these queries:

- Orders Received Rate:
  ```
  sum(rate(app_orders_total{status="received"}[5m]))
  ```

- Current Queue Size:
  ```
  sum(app_order_queue_size)
  ```

- Orders Processing Time (95th percentile):
  ```
  histogram_quantile(0.95, sum(rate(app_order_processing_seconds_bucket[5m])) by (le))
  ```

- Order Size Distribution:
  ```
  sum(rate(app_items_per_order_bucket[5m])) by (le)
  ```

## Best Practices for Custom Metrics

1. **Use appropriate metric types:**
   - Counter: For values that only increase (e.g., total requests)
   - Gauge: For values that go up and down (e.g., queue size)
   - Histogram/Summary: For measuring distributions (e.g., request duration)

2. **Choose meaningful names:**
   - Use prefixes for your application (e.g., `app_`, `http_`)
   - Include units in the name (e.g., `_seconds`, `_bytes`)
   - Follow the Prometheus naming conventions

3. **Add relevant labels:**
   - Include labels for filtering and aggregation
   - Don't use high-cardinality labels (like unique IDs)
   - Common labels: status, method, endpoint, service

4. **Optimize metric collection:**
   - Set appropriate scrape intervals (usually 15-30s)
   - Consider the overhead of complex metrics
   - Use bucketing wisely in histograms

5. **Document your metrics:**
   - Add clear help text to each metric
   - Maintain a catalog of available metrics
   - Explain what each metric represents and how it should be interpreted

## Troubleshooting Custom Metrics

### Common Issues

1. **Metrics not showing in Prometheus:**
   - Check that annotations are correct
   - Verify the ServiceMonitor is correctly configured
   - Check Prometheus target status for errors

2. **Metrics available in Prometheus but not in the metrics API:**
   - Check the Prometheus Adapter configuration
   - Verify the metrics query matches your needs
   - Look at adapter logs for errors

3. **HPA not scaling based on custom metrics:**
   - Ensure metric names match between adapter and HPA
   - Check if the metric values make sense for your scaling targets
   - Verify the HPA can access the metrics API

### Debugging Commands

```bash
# Check if metrics are exposed by your application
kubectl exec -it <pod-name> -- curl localhost:8080/metrics

# Check if ServiceMonitor is configured correctly
kubectl get servicemonitor -n <namespace>

# Check Prometheus targets
kubectl port-forward -n monitoring svc/prometheus-server 9090:80
# Then visit http://localhost:9090/targets in your browser

# Check custom metrics API
kubectl get --raw "/apis/custom.metrics.k8s.io/v1beta1" | jq .

# Check specific metric
kubectl get --raw "/apis/custom.metrics.k8s.io/v1beta1/namespaces/default/pods/*/orders_queue_size" | jq .
```

## Conclusion

Custom metrics enable you to implement advanced monitoring and autoscaling based on application-specific indicators. By exposing metrics from your applications and configuring the appropriate adapters and HPA rules, you can build a more responsive and efficient Kubernetes environment that scales based on real application needs rather than just CPU and memory usage. 