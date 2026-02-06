# Monitoring & Observability Cheatsheet

## Quick Reference Guide for Platform Engineers

### Prometheus Queries (PromQL)

```promql
# Basic Queries
up                                          # All targets up/down status
up{job="node-exporter"}                     # Filter by label
up{job="node-exporter", instance="localhost:9100"}  # Multiple labels

# Rate and Increase
rate(http_requests_total[5m])               # Per-second rate over 5 minutes
irate(http_requests_total[5m])              # Instant rate (last 2 points)
increase(http_requests_total[1h])           # Total increase over 1 hour

# Aggregation Operators
sum(rate(http_requests_total[5m]))          # Sum across all series
avg(rate(http_requests_total[5m]))          # Average
max(rate(http_requests_total[5m]))          # Maximum
min(rate(http_requests_total[5m]))          # Minimum
count(up == 1)                              # Count of up instances

# Grouping
sum by (job) (rate(http_requests_total[5m]))           # Sum grouped by job
sum without (instance) (rate(http_requests_total[5m])) # Sum without instance dimension

# Comparison Operators
http_requests_total > 1000                  # Greater than
http_requests_total < 1000                  # Less than
http_requests_total != 1000                 # Not equal
http_requests_total == 1000                 # Equal
up == 0                                     # Down instances

# Logical Operators
up == 1 and on(instance) http_requests_total > 100     # AND
up == 1 or up == 0                          # OR
up == 1 unless on(instance) http_errors_total > 0     # UNLESS

# Time-based Functions
# Time ranges: s (seconds), m (minutes), h (hours), d (days), w (weeks), y (years)
rate(http_requests_total[1h])               # 1 hour window
rate(http_requests_total[30m])              # 30 minute window
rate(http_requests_total[1d])               # 1 day window

# Offset
http_requests_total offset 1h               # Data from 1 hour ago
rate(http_requests_total[5m] offset 1d)     # Rate from 1 day ago

# Mathematical Operations
node_memory_MemFree_bytes / 1024 / 1024 / 1024          # Convert to GB
(1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100  # Memory usage %

# Useful Metrics
# CPU Usage
100 - (avg by (instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

# Memory Usage Percentage
(1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100

# Disk Usage Percentage
(1 - (node_filesystem_avail_bytes / node_filesystem_size_bytes)) * 100

# Network Traffic (bytes/sec)
rate(node_network_receive_bytes_total[5m])
rate(node_network_transmit_bytes_total[5m])

# HTTP Error Rate
sum(rate(http_requests_total{status=~"5.."}[5m])) / sum(rate(http_requests_total[5m])) * 100

# HTTP Request Duration (95th percentile)
histogram_quantile(0.95, sum(rate(http_request_duration_seconds_bucket[5m])) by (le))

# Pod CPU Usage (Kubernetes)
sum(rate(container_cpu_usage_seconds_total{pod=~"myapp.*"}[5m])) by (pod)

# Pod Memory Usage (Kubernetes)
sum(container_memory_usage_bytes{pod=~"myapp.*"}) by (pod) / 1024 / 1024  # MB

# Pod Restart Count (Kubernetes)
sum(increase(kube_pod_container_status_restarts_total[1h])) by (pod)

# Predicting Future Values
predict_linear(node_filesystem_free_bytes[1h], 4 * 3600)  # Predict 4 hours ahead

# Top-K Results
topk(5, sum by (instance) (rate(http_requests_total[5m])))    # Top 5 instances
bottomk(3, node_filesystem_avail_bytes)                       # Bottom 3 by disk space

# Sorting
sort(node_filesystem_avail_bytes)           # Ascending
sort_desc(node_filesystem_avail_bytes)      # Descending

# Time and Date Functions
time()                                      # Current Unix timestamp
day_of_month()                             # Day of month (1-31)
day_of_week()                              # Day of week (0-6, 0=Sunday)
hour()                                     # Hour of day (0-23)
```

### Prometheus Configuration

```yaml
# prometheus.yml
global:
  scrape_interval: 15s                      # Default scrape interval
  evaluation_interval: 15s                  # Evaluate rules interval
  external_labels:
    cluster: 'production'
    region: 'us-east-1'

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
            - alertmanager:9093

# Rule files
rule_files:
  - 'alerts/*.yml'
  - 'recording_rules/*.yml'

# Scrape configurations
scrape_configs:
  # Prometheus itself
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
  
  # Node exporter
  - job_name: 'node-exporter'
    static_configs:
      - targets:
          - 'node1:9100'
          - 'node2:9100'
        labels:
          environment: 'production'
  
  # Kubernetes service discovery
  - job_name: 'kubernetes-pods'
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
        action: replace
        target_label: __metrics_path__
        regex: (.+)
      - source_labels: [__address__, __meta_kubernetes_pod_annotation_prometheus_io_port]
        action: replace
        regex: ([^:]+)(?::\d+)?;(\d+)
        replacement: $1:$2
        target_label: __address__
  
  # Blackbox exporter (endpoint monitoring)
  - job_name: 'blackbox'
    metrics_path: /probe
    params:
      module: [http_2xx]
    static_configs:
      - targets:
          - https://example.com
          - https://api.example.com
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: blackbox-exporter:9115

# Remote write (long-term storage)
remote_write:
  - url: "http://thanos-receiver:19291/api/v1/receive"
    queue_config:
      capacity: 10000
      max_shards: 5
```

### Prometheus Alert Rules

```yaml
# alerts/alerts.yml
groups:
  - name: instance_alerts
    interval: 30s
    rules:
      # Instance down
      - alert: InstanceDown
        expr: up == 0
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "Instance {{ $labels.instance }} down"
          description: "{{ $labels.instance }} of job {{ $labels.job }} has been down for more than 5 minutes."
      
      # High CPU usage
      - alert: HighCPUUsage
        expr: 100 - (avg by(instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "High CPU usage on {{ $labels.instance }}"
          description: "CPU usage is above 80% (current value: {{ $value }}%)"
      
      # High Memory usage
      - alert: HighMemoryUsage
        expr: (1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100 > 90
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High memory usage on {{ $labels.instance }}"
          description: "Memory usage is above 90% (current value: {{ $value }}%)"
      
      # Disk space low
      - alert: DiskSpaceLow
        expr: (1 - (node_filesystem_avail_bytes{fstype!~"tmpfs|overlay"} / node_filesystem_size_bytes{fstype!~"tmpfs|overlay"})) * 100 > 85
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "Disk space low on {{ $labels.instance }}"
          description: "Disk usage on {{ $labels.mountpoint }} is above 85% (current value: {{ $value }}%)"
      
      # High HTTP error rate
      - alert: HighHTTPErrorRate
        expr: sum(rate(http_requests_total{status=~"5.."}[5m])) / sum(rate(http_requests_total[5m])) > 0.05
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "High HTTP error rate"
          description: "HTTP 5xx error rate is above 5% (current value: {{ $value }}%)"
      
      # Pod restart
      - alert: PodRestartingTooOften
        expr: increase(kube_pod_container_status_restarts_total[1h]) > 5
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Pod {{ $labels.pod }} restarting frequently"
          description: "Pod has restarted {{ $value }} times in the last hour"

  - name: recording_rules
    interval: 30s
    rules:
      # Pre-calculate expensive queries
      - record: job:http_requests:rate5m
        expr: sum by (job) (rate(http_requests_total[5m]))
      
      - record: instance:node_cpu:avg_rate5m
        expr: avg by (instance) (irate(node_cpu_seconds_total{mode!="idle"}[5m]))
```

### Grafana Dashboards

```json
// Grafana Dashboard JSON (simplified)
{
  "dashboard": {
    "title": "System Overview",
    "panels": [
      {
        "title": "CPU Usage",
        "targets": [
          {
            "expr": "100 - (avg by(instance) (irate(node_cpu_seconds_total{mode=\"idle\"}[5m])) * 100)"
          }
        ],
        "type": "graph"
      },
      {
        "title": "Memory Usage",
        "targets": [
          {
            "expr": "(1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100"
          }
        ],
        "type": "gauge"
      },
      {
        "title": "Disk I/O",
        "targets": [
          {
            "expr": "rate(node_disk_read_bytes_total[5m])",
            "legendFormat": "Read - {{ device }}"
          },
          {
            "expr": "rate(node_disk_written_bytes_total[5m])",
            "legendFormat": "Write - {{ device }}"
          }
        ],
        "type": "graph"
      }
    ]
  }
}
```

### Grafana Provisioning

```yaml
# grafana/provisioning/datasources/prometheus.yml
apiVersion: 1

datasources:
  - name: Prometheus
    type: prometheus
    access: proxy
    url: http://prometheus:9090
    isDefault: true
    editable: true
    jsonData:
      timeInterval: "15s"

# grafana/provisioning/dashboards/dashboard.yml
apiVersion: 1

providers:
  - name: 'default'
    orgId: 1
    folder: ''
    type: file
    disableDeletion: false
    updateIntervalSeconds: 10
    allowUiUpdates: true
    options:
      path: /var/lib/grafana/dashboards
```

### ELK Stack Basics

```bash
# Elasticsearch
# Index Operations
curl -X GET "localhost:9200/_cat/indices?v"                    # List indices
curl -X GET "localhost:9200/myindex/_search?pretty"            # Search index
curl -X DELETE "localhost:9200/myindex"                        # Delete index
curl -X PUT "localhost:9200/myindex"                          # Create index

# Search Queries
curl -X GET "localhost:9200/myindex/_search" -H 'Content-Type: application/json' -d'
{
  "query": {
    "match": {
      "message": "error"
    }
  }
}
'

# Aggregations
curl -X GET "localhost:9200/logs-*/_search" -H 'Content-Type: application/json' -d'
{
  "size": 0,
  "aggs": {
    "error_count": {
      "terms": {
        "field": "level.keyword",
        "size": 10
      }
    }
  }
}
'

# Cluster Health
curl -X GET "localhost:9200/_cluster/health?pretty"
curl -X GET "localhost:9200/_nodes/stats?pretty"

# Logstash Configuration
# logstash.conf
input {
  beats {
    port => 5044
  }
  
  file {
    path => "/var/log/*.log"
    start_position => "beginning"
  }
  
  syslog {
    port => 514
  }
}

filter {
  # Grok pattern for parsing logs
  grok {
    match => { "message" => "%{COMBINEDAPACHELOG}" }
  }
  
  # Date parsing
  date {
    match => [ "timestamp", "dd/MMM/yyyy:HH:mm:ss Z" ]
    target => "@timestamp"
  }
  
  # Add fields
  mutate {
    add_field => { "environment" => "production" }
    remove_field => [ "temp_field" ]
  }
  
  # Conditionals
  if [level] == "ERROR" {
    mutate {
      add_tag => [ "error" ]
    }
  }
}

output {
  elasticsearch {
    hosts => ["localhost:9200"]
    index => "logs-%{+YYYY.MM.dd}"
  }
  
  stdout {
    codec => rubydebug
  }
  
  # Only errors to separate index
  if "error" in [tags] {
    elasticsearch {
      hosts => ["localhost:9200"]
      index => "errors-%{+YYYY.MM.dd}"
    }
  }
}

# Filebeat Configuration
# filebeat.yml
filebeat.inputs:
  - type: log
    enabled: true
    paths:
      - /var/log/*.log
    fields:
      environment: production
    multiline.pattern: '^[0-9]{4}-[0-9]{2}-[0-9]{2}'
    multiline.negate: true
    multiline.match: after

output.logstash:
  hosts: ["localhost:5044"]

# OR output directly to Elasticsearch
output.elasticsearch:
  hosts: ["localhost:9200"]
  index: "filebeat-%{+yyyy.MM.dd}"

# Kibana Query Language (KQL)
# Simple search
message: "error"
status: 500
level: "ERROR"

# AND/OR operators
message: "error" AND status: 500
level: "ERROR" OR level: "FATAL"

# NOT operator
NOT status: 200

# Wildcards
message: err*
host: web*.example.com

# Ranges
status >= 400
response_time > 1000
@timestamp >= "2024-01-01" AND @timestamp < "2024-02-01"

# Exists
_exists_: user_id

# Nested fields
user.name: "john"
metadata.region: "us-east-1"
```

### Alerting with Alertmanager

```yaml
# alertmanager.yml
global:
  resolve_timeout: 5m
  slack_api_url: 'https://hooks.slack.com/services/XXX/YYY/ZZZ'
  smtp_from: 'alerts@example.com'
  smtp_smarthost: 'smtp.gmail.com:587'
  smtp_auth_username: 'alerts@example.com'
  smtp_auth_password: 'password'

# Templates
templates:
  - '/etc/alertmanager/templates/*.tmpl'

# Route tree
route:
  group_by: ['alertname', 'cluster', 'service']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 12h
  receiver: 'default'
  
  routes:
    # Critical alerts
    - match:
        severity: critical
      receiver: 'pagerduty'
      continue: true
    
    # Database alerts
    - match_re:
        service: ^(postgres|mysql|mongodb)$
      receiver: 'database-team'
      group_by: ['alertname', 'instance']
    
    # Warning alerts
    - match:
        severity: warning
      receiver: 'slack'

# Inhibition rules (suppress alerts)
inhibit_rules:
  - source_match:
      severity: 'critical'
    target_match:
      severity: 'warning'
    equal: ['alertname', 'instance']

# Receivers
receivers:
  - name: 'default'
    email_configs:
      - to: 'team@example.com'
  
  - name: 'slack'
    slack_configs:
      - channel: '#alerts'
        title: '{{ .GroupLabels.alertname }}'
        text: '{{ range .Alerts }}{{ .Annotations.description }}{{ end }}'
  
  - name: 'pagerduty'
    pagerduty_configs:
      - service_key: 'YOUR_PAGERDUTY_KEY'
        description: '{{ .GroupLabels.alertname }}'
  
  - name: 'database-team'
    email_configs:
      - to: 'db-team@example.com'
    slack_configs:
      - channel: '#database-alerts'
```

### Distributed Tracing (Jaeger/Zipkin)

```yaml
# Jaeger All-in-One (Docker Compose)
version: '3'
services:
  jaeger:
    image: jaegertracing/all-in-one:latest
    ports:
      - "5775:5775/udp"    # accept zipkin.thrift over compact thrift protocol
      - "6831:6831/udp"    # accept jaeger.thrift over compact thrift protocol
      - "6832:6832/udp"    # accept jaeger.thrift over binary thrift protocol
      - "5778:5778"        # serve configs
      - "16686:16686"      # serve frontend
      - "14268:14268"      # accept jaeger.thrift directly from clients
      - "14250:14250"      # accept model.proto
      - "9411:9411"        # Zipkin compatible endpoint
    environment:
      COLLECTOR_ZIPKIN_HTTP_PORT: 9411
```

```javascript
// Jaeger Instrumentation (Node.js)
const { initTracer } = require('jaeger-client');

const config = {
  serviceName: 'my-service',
  sampler: {
    type: 'const',
    param: 1,
  },
  reporter: {
    logSpans: true,
    agentHost: 'localhost',
    agentPort: 6831,
  },
};

const tracer = initTracer(config);

// Create span
const span = tracer.startSpan('http_request');
span.setTag('http.method', 'GET');
span.setTag('http.url', '/api/users');
span.log({ event: 'request_received' });

// Finish span
span.finish();
```

```python
# OpenTelemetry (Python)
from opentelemetry import trace
from opentelemetry.exporter.jaeger.thrift import JaegerExporter
from opentelemetry.sdk.resources import Resource
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor

# Configure tracer
resource = Resource.create({"service.name": "my-python-service"})
trace.set_tracer_provider(TracerProvider(resource=resource))
tracer = trace.get_tracer(__name__)

# Configure Jaeger exporter
jaeger_exporter = JaegerExporter(
    agent_host_name="localhost",
    agent_port=6831,
)

# Add span processor
trace.get_tracer_provider().add_span_processor(
    BatchSpanProcessor(jaeger_exporter)
)

# Create span
with tracer.start_as_current_span("operation_name") as span:
    span.set_attribute("http.method", "GET")
    span.set_attribute("http.url", "/api/data")
    # Your code here
    span.add_event("Processing request")
```

### Common Metrics to Monitor

```yaml
# Infrastructure Metrics
- CPU usage (%)
- Memory usage (%)
- Disk usage (%)
- Disk I/O (read/write MB/s)
- Network traffic (bytes in/out)
- Network errors/drops
- Load average (1m, 5m, 15m)
- Open file descriptors
- Process count

# Application Metrics
- Request rate (req/s)
- Error rate (%)
- Response time (p50, p95, p99)
- Active connections
- Queue length
- Cache hit ratio
- Database connection pool usage
- GC pause time
- Thread count
- Heap usage

# Business Metrics
- Active users
- Transactions per second
- Revenue per minute
- Conversion rate
- API calls per endpoint
- Feature usage
- User sessions

# Kubernetes Metrics
- Pod CPU/Memory usage
- Pod restart count
- Container status
- Node status
- Deployment status
- PersistentVolume usage
- Service availability
- Ingress request rate
- HPA (Horizontal Pod Autoscaler) status

# Database Metrics
- Query execution time
- Connections (active/idle)
- Slow queries
- Lock wait time
- Cache hit ratio
- Replication lag
- Table size
- Index usage

# Web Server Metrics (Nginx/Apache)
- Request rate
- Response codes (2xx, 3xx, 4xx, 5xx)
- Request duration
- Active connections
- Waiting/Reading/Writing connections
- Upstream response time
```

### Monitoring Best Practices

```yaml
1. The Four Golden Signals:
   - Latency: Time to serve a request
   - Traffic: Amount of demand on your system
   - Errors: Rate of failed requests
   - Saturation: How "full" your service is

2. RED Method (for services):
   - Rate: Requests per second
   - Errors: Number of failed requests
   - Duration: Time taken per request

3. USE Method (for resources):
   - Utilization: % time resource is busy
   - Saturation: Amount of work resource cannot service (queue)
   - Errors: Count of error events

4. Monitoring Principles:
   - Monitor symptoms, not causes
   - Build dashboards for humans, not machines
   - Set meaningful alert thresholds
   - Avoid alert fatigue
   - Document runbooks for alerts
   - Test your monitoring system
   - Monitor the monitors
```

### Logging Best Practices

```json
// Structured Logging (JSON)
{
  "timestamp": "2024-01-15T10:30:00Z",
  "level": "ERROR",
  "service": "api-server",
  "environment": "production",
  "message": "Failed to connect to database",
  "error": {
    "type": "ConnectionError",
    "message": "Connection timeout",
    "stack": "..."
  },
  "context": {
    "user_id": "12345",
    "request_id": "abc-123",
    "ip": "192.168.1.1"
  },
  "duration_ms": 5000
}
```

```bash
# Log Levels
TRACE   # Very detailed diagnostic information
DEBUG   # Detailed information for debugging
INFO    # General informational messages
WARN    # Warning messages, potential issues
ERROR   # Error messages, failures
FATAL   # Critical errors, application cannot continue

# What to Log
- Application start/stop
- Configuration changes
- Authentication attempts (success/failure)
- Authorization failures
- Data validation errors
- External service calls
- Performance metrics
- Business events
- Exceptions and errors

# What NOT to Log
- Passwords or secrets
- Credit card numbers
- Personal identifiable information (PII)
- Session tokens
- API keys
```

## Quick Tips

1. **Use labels wisely** - Don't create too many time series
2. **Set retention policies** - Balance storage costs vs data needs
3. **Use recording rules** - Pre-calculate expensive queries
4. **Test alerts in non-prod** - Verify alert rules work correctly
5. **Document dashboards** - Add descriptions and context
6. **Use template variables** - Make dashboards reusable
7. **Monitor your monitoring** - Ensure observability stack is healthy
8. **Implement SLOs/SLIs** - Define and track service level objectives
9. **Use distributed tracing** - Understand complex request flows
10. **Correlate metrics, logs, and traces** - Get complete picture of issues
