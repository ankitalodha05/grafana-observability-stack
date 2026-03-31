
## Objective

This document explains how to set up OpenTelemetry Collector for distributed tracing with Tempo.
It also includes a brief overview, setup steps, and current status.

---

## Overview

OpenTelemetry Collector acts as a central layer between applications and Tempo.

Instead of sending traces directly to Tempo, applications send traces to the Collector, and the Collector forwards them to Tempo.

### Flow

Application → OpenTelemetry Collector → Tempo → Grafana

This setup helps in:

* centralizing trace ingestion
* managing multiple applications
* improving reliability with batching and retries
* simplifying configuration across servers

---

## Components

### OpenTelemetry Collector

* receives traces from applications
* processes and batches trace data
* forwards traces to Tempo

### Tempo

* stores trace data
* provides trace data to Grafana

### Grafana

* visualizes and searches traces

---

## Prerequisites

* Linux VM for Collector
* Docker installed
* Tempo already configured and running
* Grafana already connected to Tempo
* Network connectivity between application, Collector, and Tempo

### Required Ports

| Port | Purpose                       |
| ---- | ----------------------------- |
| 4317 | OTLP gRPC receive (Collector) |
| 4318 | OTLP HTTP receive (Collector) |
| 4317 | OTLP gRPC forward (Tempo)     |
| 3200 | Tempo API                     |

---

## Collector Setup

### Step 1: Create Configuration File

```bash
sudo nano otel-collector-config.yaml
```

Paste the configuration:

```yaml
receivers:
  otlp:
    protocols:
      grpc:
        endpoint: 0.0.0.0:4317
      http:
        endpoint: 0.0.0.0:4318

processors:
  batch:

exporters:
  otlp:
    endpoint: 103.150.136.119:4317
    tls:
      insecure: true

service:
  pipelines:
    traces:
      receivers: [otlp]
      processors: [batch]
      exporters: [otlp]
```

Replace:

```text
103.150.136.119
```

with your Tempo server IP if different.

---

### Step 2: Run Collector Container

```bash
docker run -d \
  --name otel-collector \
  -p 4317:4317 \
  -p 4318:4318 \
  -v $(pwd)/otel-collector-config.yaml:/etc/otelcol-contrib/config.yaml \
  otel/opentelemetry-collector-contrib:latest
```

---

### Step 3: Verify Collector

```bash
docker ps
docker logs -f otel-collector
```

Ensure there are no startup errors.

---

## Network Configuration

Ensure following ports are open:

```bash
ufw allow 4317/tcp
ufw allow 4318/tcp
ufw reload
```

---

## Application Configuration

Applications must send traces to Collector.

### gRPC

```bash
OTEL_EXPORTER_OTLP_ENDPOINT=http://<COLLECTOR_IP>:4317
OTEL_EXPORTER_OTLP_PROTOCOL=grpc
OTEL_SERVICE_NAME=my-app
```

---

### HTTP

```bash
OTEL_EXPORTER_OTLP_ENDPOINT=http://<COLLECTOR_IP>:4318
OTEL_EXPORTER_OTLP_PROTOCOL=http/protobuf
OTEL_SERVICE_NAME=my-app
```

---

## What Has Been Completed

* Tempo setup completed
* Grafana Tempo datasource configured and working
* Required ports opened
* Collector configuration prepared
* Collector deployment approach defined

---

## What Is Pending

* Collector deployment on server
* Identification of applications running on servers
* Application-level OpenTelemetry instrumentation
* Configuration of applications to send traces to Collector
* Validation of traces in Grafana

---

## Important Note

OpenTelemetry Collector does not generate traces.

Tracing requires:

* application instrumentation
* OpenTelemetry SDK or auto-instrumentation
* developer involvement

Without this, no trace data will be visible in Grafana.

---

## Next Steps

* deploy Collector in environment
* coordinate with application owners
* enable OpenTelemetry in applications
* validate traces in Grafana Explore

---

## Expected Result

After completing pending steps:

* applications send traces to Collector
* Collector forwards traces to Tempo
* Tempo stores traces
* Grafana displays traces

---

## Conclusion

OpenTelemetry Collector setup provides a centralized and scalable approach for trace collection.

Infrastructure for tracing is ready.
Further progress depends on application-level integration.

---
