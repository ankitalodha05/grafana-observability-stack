
## Objective

This document explains the setup of distributed tracing using Tempo and OpenTelemetry.
It also includes a brief overview, completed work, and pending actions.

---

## Overview

Distributed tracing helps track how a request flows across different services and systems.

In this setup:

* **OpenTelemetry** is used to generate and send traces from applications
* **Tempo** is used as a centralized tracing backend
* **Grafana** is used to visualize and analyze traces

### Flow

Application → OpenTelemetry → Tempo → Grafana

This setup helps in:

* Understanding request flow across services
* Identifying latency and bottlenecks
* Debugging failures across distributed systems

---

## Components

### Tempo

* Stores trace data
* Receives traces using OTLP (gRPC/HTTP)
* Provides data to Grafana for visualization

### OpenTelemetry

* Collects traces from applications
* Sends trace data to Tempo
* Requires application-level integration

### Grafana

* Used to search and visualize traces
* Connected to Tempo as a datasource

---

## Prerequisites

* Linux VM for Tempo server
* Docker installed
* Grafana already configured
* Network connectivity between systems

### Required Ports

| Port | Purpose                        |
| ---- | ------------------------------ |
| 3200 | Tempo API / Grafana connection |
| 4317 | OTLP gRPC ingestion            |
| 4318 | OTLP HTTP ingestion            |

---

## Tempo Setup

### Step 1: Create Configuration File

```bash
sudo nano tempo.yaml
```

Paste configuration:

```yaml
server:
  http_listen_port: 3200

distributor:
  receivers:
    otlp:
      protocols:
        grpc:
          endpoint: "0.0.0.0:4317"
        http:
          endpoint: "0.0.0.0:4318"

ingester:
  max_block_duration: 5m

compactor:
  compaction:
    block_retention: 24h

storage:
  trace:
    backend: local
    local:
      path: /tmp/tempo/traces

querier:
  frontend_worker:
    frontend_address: 127.0.0.1:9095

query_frontend:
  search:
    duration_slo: 5s
    throughput_bytes_slo: 1073741824
  trace_by_id:
    duration_slo: 5s

overrides:
  defaults:
    metrics_generator:
      processors: []
```

---

### Step 2: Create Data Directory

```bash
mkdir -p tempo-data
chmod -R 777 tempo-data
```

---

### Step 3: Run Tempo Container

```bash
docker run -d \
  --name tempo \
  -p 3200:3200 \
  -p 4317:4317 \
  -p 4318:4318 \
  -v $(pwd)/tempo.yaml:/etc/tempo.yaml \
  -v $(pwd)/tempo-data:/tmp/tempo \
  grafana/tempo:2.9.0 \
  -config.file=/etc/tempo.yaml
```

---

### Step 4: Verify Tempo

```bash
docker ps
docker logs -f tempo
curl http://localhost:3200/ready
```

Expected output:

```bash
ready
```

---

## Grafana Configuration

### Add Tempo Datasource

* Go to: **Connections → Data Sources**
* Select: **Tempo**
* URL:

```text
http://<TEMPO_SERVER_IP>:3200
```

* Click **Save & Test**

---

## OpenTelemetry Integration (Concept)

OpenTelemetry is responsible for generating traces.

To enable tracing:

* Applications must be instrumented using OpenTelemetry SDK
* Or use OpenTelemetry Collector

### Example Endpoint

```bash
http://<TEMPO_SERVER_IP>:4317   (gRPC)
http://<TEMPO_SERVER_IP>:4318   (HTTP)
```

---

## What Has Been Completed

* Tempo server setup completed
* Tempo configuration verified
* Docker container running successfully
* Required ports opened (3200, 4317, 4318)
* Grafana integrated with Tempo datasource
* Tempo readiness and connectivity verified

---

## What Is Pending

* Application-level OpenTelemetry instrumentation
* Identification of running applications on servers
* Integration of OpenTelemetry SDK or agents in applications
* Trace generation from applications
* End-to-end trace visibility validation in Grafana

---

## Important Note

Tracing cannot work without application instrumentation.

Since current applications are not instrumented:

* No trace data will be generated
* Grafana will not show traces

This step requires coordination with the development team.

---

## Next Steps

* Identify application owners
* Decide instrumentation approach (SDK / Collector)
* Enable tracing in applications
* Validate traces in Grafana Explore

---

## Expected Result

After completing pending steps:

* Applications will generate traces
* Tempo will store trace data
* Grafana will display traces
* Full request flow visibility will be available

---

## Conclusion

Tempo and OpenTelemetry setup for tracing is successfully completed at infrastructure level.

The system is ready to receive traces.
Further progress depends on application-level integration.

---
