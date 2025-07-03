# Monitoring and Observability Architecture

This document outlines the monitoring, logging, and alerting infrastructure implemented across the microservices ecosystem. The architecture leverages Prometheus, Grafana, Node Exporter, and Blackbox Exporter to provide comprehensive system observability with minimal operational overhead.

## Monitoring Infrastructure

### Core Components

| Component | Purpose | Deployment Context |
|-----------|---------|-------------------|
| Prometheus | Time-series metrics collection | Dedicated EC2 instance |
| Grafana | Metrics visualization | Co-located with Prometheus |
| Node Exporter | System metrics collection | All service instances |
| Blackbox Exporter | External endpoint monitoring | Co-located with Prometheus |
| CloudWatch | Optional AWS service metrics | AWS managed service |

### System Architecture

1. Dedicated Monitoring Instance:
   - Hosts Prometheus, Grafana, and Blackbox Exporter
   - Configured with persistent EBS volume for data retention
   - Accessible via internal VPC endpoints

2. Metrics Collection Pipeline:
   - Node Exporter exposes system metrics on :9100/metrics
   - Blackbox Exporter performs HTTP probes on :9115/probe
   - Prometheus scrapes targets on configurable intervals

## Prometheus Configuration

### Target Configuration

```yaml
scrape_configs:
  - job_name: 'node'
    static_configs:
      - targets: ['localhost:9100']
        labels:
          instance: 'monitoring-host'

  - job_name: 'blackbox'
    metrics_path: /probe
    params:
      module: [http_2xx]
    static_configs:
      - targets:
        - http://test.domain.com/health
        - http://stage.domain.com/health
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
```

## Grafana Dashboards

### Pre-configured Dashboards

1. EC2 Node Metrics:
   - CPU, Memory, Disk utilization
   - Network I/O metrics
   - System load averages

2. API Availability:
   - Endpoint response times
   - Success rates
   - Error rates by status code

3. Auto Scaling Metrics:
   - Instance count by ASG
   - Scale-in/out events
   - CPU utilization triggers

4. AI Proxy Load:
   - Request queue depth
   - Processing latency
   - Token consumption rate

5. Gmail Service:
   - API quota utilization
   - Email processing rates
   - Error tracking

## Alert Configuration

### Prometheus Alert Rules

```yaml
groups:
- name: instance_alerts
  rules:
  - alert: HighCPUUsage
    expr: instance:node_cpu_usage:rate5m > 0.85
    for: 5m
    labels:
      severity: warning
    annotations:
      description: CPU usage above 85%

  - alert: APIHighLatency
    expr: probe_duration_seconds > 1
    for: 5m
    labels:
      severity: critical
```

### Alertmanager Integration

Alertmanager handles alert routing and aggregation:
- Email notifications for non-critical alerts
- Slack integration for immediate attention
- PagerDuty escalation for critical issues

## Logging Strategy

### Implementation Options

1. Local Disk Storage:
   - Rotated log files on EC2 instances
   - Retention policy based on disk space

2. CloudWatch Logs:
   - Centralized log aggregation
   - Log insights for analysis

3. Optional Loki Integration:
   - Log aggregation with label-based querying
   - Grafana visualization integration

### Structured Logging

JSON format for machine-readable logs:
```json
{
  "timestamp": "2023-11-01T10:00:00Z",
  "level": "error",
  "service": "ai-proxy",
  "message": "Rate limit exceeded",
  "request_id": "abc-123"
}
```
