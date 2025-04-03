# Distributed Tracing in Kubernetes

This example demonstrates how to implement distributed tracing in a Kubernetes environment using OpenTelemetry and Jaeger to gain insights into request flows across microservices.

## Overview

Distributed tracing is a method used to profile and monitor applications, especially those built using a microservices architecture. It helps to pinpoint where failures occur and what causes poor performance by tracking requests as they flow through different services.

Key benefits of distributed tracing include:

- **End-to-end visibility**: Track requests across multiple services
- **Performance optimization**: Identify bottlenecks and latency issues
- **Dependency analysis**: Understand service dependencies and interactions
- **Error identification**: Isolate where errors occur in the request chain
- **Resource allocation**: Optimize resource usage based on observed patterns

## Tracing Concepts

### Trace

A trace represents the entire journey of a request as it moves through a distributed system. It consists of one or more spans.

### Span

A span represents a single operation within a trace. It has a name, start time, duration, and can contain tags and logs.

### Context Propagation

Context propagation refers to how trace information is passed between services, typically using HTTP headers or messaging middleware.

### Sampling

Sampling is the process of selecting which traces to collect, as collecting all traces can be resource-intensive.

## Architecture

This example implements distributed tracing with:

1. **OpenTelemetry**: For instrumenting applications and collecting traces
2. **Jaeger**: For storing, processing, and visualizing traces
3. **Sample microservices**: To demonstrate tracing across multiple services

## Prerequisites

- A Kubernetes cluster
- Helm 3 installed
- kubectl installed and configured
- Basic understanding of microservices

## Installing Jaeger

### Option 1: Using Helm Chart

```bash
# Add the Jaeger Helm repository
helm repo add jaegertracing https://jaegertracing.github.io/helm-charts
helm repo update

# Create a namespace for observability tools
kubectl create namespace observability

# Install Jaeger All-in-One for demo/development
helm install jaeger jaegertracing/jaeger \
  --namespace observability \
  --set allInOne.enabled=true \
  --set collector.enabled=false \
  --set agent.enabled=false \
  --set query.enabled=false
```

### Option 2: Using Jaeger Operator

For production environments, the Jaeger Operator provides more control:

```bash
# Install the Jaeger Operator
kubectl create namespace observability
kubectl apply -f https://github.com/jaegertracing/jaeger-operator/releases/download/v1.38.0/jaeger-operator.yaml -n observability

# Wait for the operator to be ready
kubectl wait --for=condition=available deployment/jaeger-operator -n observability --timeout=90s

# Create a simple Jaeger instance
cat <<EOF | kubectl apply -f -
apiVersion: jaegertracing.io/v1
kind: Jaeger
metadata:
  name: simple-jaeger
  namespace: observability
spec:
  strategy: allInOne
  allInOne:
    image: jaegertracing/all-in-one:latest
    options:
      log-level: info
  storage:
    type: memory
    options:
      memory:
        max-traces: 100000
  ingress:
    enabled: true
    hosts:
      - jaeger.example.com
EOF
```

### Verify Jaeger Installation

Check that Jaeger is running:

```bash
kubectl get pods -n observability
kubectl get svc -n observability
```

Access the Jaeger UI:

```bash
kubectl port-forward -n observability svc/jaeger-query 16686:16686
```

Then open `http://localhost:16686` in your browser.

## Installing OpenTelemetry Collector

The OpenTelemetry Collector receives, processes, and exports telemetry data:

```bash
# Add the OpenTelemetry Helm repository
helm repo add open-telemetry https://open-telemetry.github.io/opentelemetry-helm-charts
helm repo update

# Install the OpenTelemetry Collector
helm install opentelemetry-collector open-telemetry/opentelemetry-collector \
  --namespace observability \
  --set mode=daemonset \
  --set config.exporters.jaeger.endpoint=jaeger-collector.observability:14250 \
  --set config.exporters.jaeger.tls.insecure=true \
  --set config.service.pipelines.traces.exporters[0]=jaeger
```

## Sample Microservice Application

Let's deploy a sample application with multiple services to demonstrate tracing:

### Example: E-commerce Application

Our sample application consists of three services:

1. **Frontend Service**: Handles HTTP requests from users
2. **Product Service**: Manages product information
3. **Order Service**: Processes orders

#### Application Manifests

```yaml
# e-commerce-app.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: e-commerce
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  namespace: e-commerce
spec:
  replicas: 1
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - name: frontend
        image: ${YOUR_REGISTRY}/frontend:latest
        ports:
        - containerPort: 8080
        env:
        - name: OTEL_SERVICE_NAME
          value: "frontend"
        - name: OTEL_EXPORTER_OTLP_ENDPOINT
          value: "http://opentelemetry-collector.observability:4317"
        - name: PRODUCT_SERVICE_URL
          value: "http://product:8080"
        - name: ORDER_SERVICE_URL
          value: "http://order:8080"
---
apiVersion: v1
kind: Service
metadata:
  name: frontend
  namespace: e-commerce
spec:
  selector:
    app: frontend
  ports:
  - port: 8080
    targetPort: 8080
  type: ClusterIP
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: product
  namespace: e-commerce
spec:
  replicas: 1
  selector:
    matchLabels:
      app: product
  template:
    metadata:
      labels:
        app: product
    spec:
      containers:
      - name: product
        image: ${YOUR_REGISTRY}/product:latest
        ports:
        - containerPort: 8080
        env:
        - name: OTEL_SERVICE_NAME
          value: "product-service"
        - name: OTEL_EXPORTER_OTLP_ENDPOINT
          value: "http://opentelemetry-collector.observability:4317"
---
apiVersion: v1
kind: Service
metadata:
  name: product
  namespace: e-commerce
spec:
  selector:
    app: product
  ports:
  - port: 8080
    targetPort: 8080
  type: ClusterIP
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: order
  namespace: e-commerce
spec:
  replicas: 1
  selector:
    matchLabels:
      app: order
  template:
    metadata:
      labels:
        app: order
    spec:
      containers:
      - name: order
        image: ${YOUR_REGISTRY}/order:latest
        ports:
        - containerPort: 8080
        env:
        - name: OTEL_SERVICE_NAME
          value: "order-service"
        - name: OTEL_EXPORTER_OTLP_ENDPOINT
          value: "http://opentelemetry-collector.observability:4317"
---
apiVersion: v1
kind: Service
metadata:
  name: order
  namespace: e-commerce
spec:
  selector:
    app: order
  ports:
  - port: 8080
    targetPort: 8080
  type: ClusterIP
```

Apply the manifests:

```bash
kubectl apply -f e-commerce-app.yaml
```

### Service Implementation Examples

#### Frontend Service (Node.js with OpenTelemetry)

```javascript
// app.js
const express = require('express');
const axios = require('axios');
const { trace, context } = require('@opentelemetry/api');
const { NodeTracerProvider } = require('@opentelemetry/sdk-trace-node');
const { Resource } = require('@opentelemetry/resources');
const { SemanticResourceAttributes } = require('@opentelemetry/semantic-conventions');
const { BatchSpanProcessor } = require('@opentelemetry/sdk-trace-base');
const { OTLPTraceExporter } = require('@opentelemetry/exporter-trace-otlp-proto');
const { registerInstrumentations } = require('@opentelemetry/instrumentation');
const { ExpressInstrumentation } = require('@opentelemetry/instrumentation-express');
const { HttpInstrumentation } = require('@opentelemetry/instrumentation-http');

// Configure OpenTelemetry
const provider = new NodeTracerProvider({
  resource: new Resource({
    [SemanticResourceAttributes.SERVICE_NAME]: process.env.OTEL_SERVICE_NAME || 'frontend',
  }),
});

const exporter = new OTLPTraceExporter({
  url: process.env.OTEL_EXPORTER_OTLP_ENDPOINT || 'http://localhost:4317',
});

provider.addSpanProcessor(new BatchSpanProcessor(exporter));
provider.register();

registerInstrumentations({
  instrumentations: [
    new HttpInstrumentation(),
    new ExpressInstrumentation(),
  ],
});

const tracer = trace.getTracer('frontend-service');
const app = express();
const port = 8080;

const productServiceUrl = process.env.PRODUCT_SERVICE_URL || 'http://localhost:8081';
const orderServiceUrl = process.env.ORDER_SERVICE_URL || 'http://localhost:8082';

app.get('/', (req, res) => {
  res.send('Frontend Service');
});

app.get('/products', async (req, res) => {
  try {
    const response = await axios.get(`${productServiceUrl}/products`);
    res.json(response.data);
  } catch (error) {
    res.status(500).json({ error: 'Failed to fetch products' });
  }
});

app.post('/checkout', async (req, res) => {
  // Create a custom span for the checkout process
  const span = tracer.startSpan('checkout-process');
  
  // Set the current span as active
  context.with(trace.setSpan(context.active(), span), async () => {
    try {
      // Add custom attributes to the span
      span.setAttribute('checkout.items', 3);
      span.setAttribute('checkout.total', 129.99);
      
      // Get product details
      const productsResponse = await axios.get(`${productServiceUrl}/products`);
      
      // Create an order
      const orderResponse = await axios.post(`${orderServiceUrl}/orders`, {
        products: productsResponse.data.slice(0, 3),
        total: 129.99
      });
      
      span.end();
      res.json(orderResponse.data);
    } catch (error) {
      // Record error in span
      span.recordException(error);
      span.setStatus({ code: 2, message: error.message }); // SpanStatusCode.ERROR
      span.end();
      
      res.status(500).json({ error: 'Checkout failed' });
    }
  });
});

app.listen(port, () => {
  console.log(`Frontend service listening at http://localhost:${port}`);
});
```

#### Product Service (Python with OpenTelemetry)

```python
# app.py
from flask import Flask, jsonify
from opentelemetry import trace
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
from opentelemetry.exporter.otlp.proto.grpc.trace_exporter import OTLPSpanExporter
from opentelemetry.sdk.resources import Resource
from opentelemetry.semconv.resource import ResourceAttributes
import os
from opentelemetry.instrumentation.flask import FlaskInstrumentation
from opentelemetry.instrumentation.requests import RequestsInstrumentation
import time
import random

# Configure OpenTelemetry
resource = Resource.create({
    ResourceAttributes.SERVICE_NAME: os.getenv("OTEL_SERVICE_NAME", "product-service")
})

provider = TracerProvider(resource=resource)
otlp_exporter = OTLPSpanExporter(
    endpoint=os.getenv("OTEL_EXPORTER_OTLP_ENDPOINT", "http://localhost:4317")
)
provider.add_span_processor(BatchSpanProcessor(otlp_exporter))
trace.set_tracer_provider(provider)

tracer = trace.get_tracer("product-service")

app = Flask(__name__)
FlaskInstrumentation().instrument_app(app)
RequestsInstrumentation().instrument()

# Sample product data
products = [
    {"id": 1, "name": "Laptop", "price": 999.99},
    {"id": 2, "name": "Smartphone", "price": 699.99},
    {"id": 3, "name": "Headphones", "price": 149.99},
    {"id": 4, "name": "Tablet", "price": 299.99},
    {"id": 5, "name": "Smartwatch", "price": 199.99}
]

@app.route('/')
def home():
    return "Product Service"

@app.route('/products')
def get_products():
    with tracer.start_as_current_span("get_all_products") as span:
        # Simulate database query latency
        time.sleep(random.uniform(0.05, 0.2))
        
        # Add custom attributes to the span
        span.set_attribute("products.count", len(products))
        
        # Randomly add error
        if random.random() < 0.1:  # 10% chance of error
            span.set_attribute("error", True)
            span.set_attribute("error.type", "database_timeout")
            return jsonify({"error": "Database timeout"}), 500
            
        return jsonify(products)

@app.route('/products/<int:product_id>')
def get_product(product_id):
    with tracer.start_as_current_span("get_product_by_id") as span:
        span.set_attribute("product.id", product_id)
        
        # Simulate database query latency
        time.sleep(random.uniform(0.01, 0.1))
        
        product = next((p for p in products if p["id"] == product_id), None)
        if product:
            return jsonify(product)
        else:
            span.set_attribute("error", True)
            span.set_attribute("error.type", "product_not_found")
            return jsonify({"error": f"Product {product_id} not found"}), 404

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8080)
```

#### Order Service (Java with OpenTelemetry)

```java
// OrderController.java
package com.example.orderservice.controller;

import com.example.orderservice.model.Order;
import com.example.orderservice.model.OrderRequest;
import com.example.orderservice.service.OrderService;
import io.opentelemetry.api.trace.Span;
import io.opentelemetry.api.trace.Tracer;
import io.opentelemetry.extension.annotations.WithSpan;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

import java.util.List;
import java.util.UUID;

@RestController
public class OrderController {
    
    private final OrderService orderService;
    private final Tracer tracer;
    
    @Autowired
    public OrderController(OrderService orderService, Tracer tracer) {
        this.orderService = orderService;
        this.tracer = tracer;
    }
    
    @GetMapping("/")
    public String home() {
        return "Order Service";
    }
    
    @GetMapping("/orders")
    public List<Order> getAllOrders() {
        return orderService.getAllOrders();
    }
    
    @PostMapping("/orders")
    @WithSpan  // OpenTelemetry annotation for automatic span creation
    public Order createOrder(@RequestBody OrderRequest orderRequest) {
        Span span = tracer.spanBuilder("process-order").startSpan();
        try {
            span.setAttribute("order.items", orderRequest.getProducts().size());
            span.setAttribute("order.total", orderRequest.getTotal());
            
            Order order = new Order();
            order.setId(UUID.randomUUID().toString());
            order.setProducts(orderRequest.getProducts());
            order.setTotal(orderRequest.getTotal());
            order.setStatus("CREATED");
            
            // Simulate processing time
            Thread.sleep((long) (Math.random() * 300));
            
            // Process payment
            processPayment(order);
            
            return orderService.saveOrder(order);
        } catch (Exception e) {
            span.recordException(e);
            span.setAttribute("error", true);
            throw new RuntimeException("Order processing failed", e);
        } finally {
            span.end();
        }
    }
    
    @WithSpan
    private void processPayment(Order order) throws InterruptedException {
        // Simulate payment processing
        Thread.sleep((long) (Math.random() * 500));
        order.setStatus("PAID");
    }
}
```

## Testing the Tracing Setup

### Generate Some Traffic

```bash
# Port forward to the frontend service
kubectl port-forward -n e-commerce svc/frontend 8080:8080

# Make requests to the application
curl http://localhost:8080/products
curl -X POST http://localhost:8080/checkout
```

### View Traces in Jaeger

1. Access the Jaeger UI at `http://localhost:16686`
2. Select a service from the dropdown (e.g., "frontend")
3. Click "Find Traces"
4. Examine the traces to see the request flow across services

## Advanced Tracing Configurations

### Sampling Configuration

For production, you might want to configure sampling to reduce the volume of traces:

```yaml
# otel-collector-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: otel-collector-conf
  namespace: observability
data:
  otel-collector-config: |
    receivers:
      otlp:
        protocols:
          grpc:
          http:
    
    processors:
      batch:
      
      tail_sampling:
        decision_wait: 10s
        num_traces: 100
        expected_new_traces_per_sec: 10
        policies:
          - name: error-policy
            type: status_code
            status_code:
              status_codes: [ERROR]
          - name: probability-policy
            type: probabilistic
            probabilistic:
              sampling_percentage: 10
    
    exporters:
      jaeger:
        endpoint: jaeger-collector.observability:14250
        tls:
          insecure: true
    
    service:
      pipelines:
        traces:
          receivers: [otlp]
          processors: [batch, tail_sampling]
          exporters: [jaeger]
```

Apply the ConfigMap and update the OpenTelemetry Collector:

```bash
kubectl apply -f otel-collector-config.yaml
helm upgrade opentelemetry-collector open-telemetry/opentelemetry-collector \
  --namespace observability \
  --set mode=daemonset \
  --set configMap.create=false \
  --set configMap.name=otel-collector-conf
```

### Context Propagation

Ensure proper context propagation between services:

#### HTTP Headers (Example in Node.js)

```javascript
// When making HTTP requests
const headers = {};
// Inject current trace context into headers
const currentSpan = trace.getSpan(context.active());
if (currentSpan) {
  propagator.inject(context.active(), headers);
}

// Make HTTP request with propagated context
axios.get('http://another-service/endpoint', { headers });
```

### Correlating Traces with Logs

To correlate traces with logs, include trace and span IDs in your log entries:

```javascript
// Node.js example
const span = trace.getSpan(context.active());
if (span) {
  const { traceId, spanId } = span.spanContext();
  logger.info('Processing request', { traceId, spanId, userId: '123' });
}
```

## Service Mesh Integration

If you're using a service mesh like Istio, you can leverage its built-in tracing capabilities:

### Istio Example

```yaml
# istio-tracing.yaml
apiVersion: install.istio.io/v1alpha1
kind: IstioOperator
metadata:
  namespace: istio-system
spec:
  meshConfig:
    enableTracing: true
    defaultConfig:
      tracing:
        sampling: 100
        zipkin:
          address: jaeger-collector.observability:9411
```

Apply the configuration:

```bash
istioctl apply -f istio-tracing.yaml
```

## Best Practices for Distributed Tracing

1. **Use consistent service naming**: Adopt a convention for service names
2. **Propagate context correctly**: Ensure trace context is passed between services
3. **Add meaningful attributes**: Include business-relevant data in spans
4. **Implement proper sampling**: Use tailored sampling strategies for production
5. **Be mindful of sensitive data**: Don't include sensitive information in span attributes
6. **Correlate with logs and metrics**: Use trace IDs in logs for correlation
7. **Monitor tracing infrastructure**: Ensure your tracing backend is healthy
8. **Use automated instrumentation**: Leverage libraries that instrument frameworks automatically
9. **Add custom spans judiciously**: Create spans for important operations, but avoid over-instrumentation
10. **Document your tracing practices**: Maintain documentation on how tracing is implemented

## Troubleshooting

### Common Issues

1. **Traces not appearing in Jaeger**:
   - Verify OpenTelemetry Collector is running
   - Check collector logs for export errors
   - Ensure context propagation is working

2. **Incomplete traces**:
   - Check that all services are properly instrumented
   - Verify context propagation between services
   - Look for timeout issues in span processors

3. **High resource usage**:
   - Adjust sampling rates
   - Optimize batch processing settings
   - Scale Jaeger components horizontally

### Debugging Commands

```bash
# Check OpenTelemetry Collector logs
kubectl logs -n observability -l app=opentelemetry-collector

# Check Jaeger Collector metrics
kubectl port-forward -n observability svc/jaeger-collector 14269:14269
curl http://localhost:14269/metrics

# Check if spans are being received
kubectl exec -n observability deploy/jaeger-collector -- curl http://localhost:14269/metrics | grep jaeger_collector_spans_received
```

## Conclusion

Distributed tracing provides crucial insights into the behavior of microservices applications. By implementing OpenTelemetry instrumentation and using Jaeger for visualization, you can:

- Track requests across service boundaries
- Identify performance bottlenecks
- Debug errors in complex request flows
- Understand service dependencies

This example provides a foundation that you can build upon to implement comprehensive tracing across your entire Kubernetes environment. 