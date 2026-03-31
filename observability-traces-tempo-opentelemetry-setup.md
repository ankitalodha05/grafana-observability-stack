
# Tempo and OpenTelemetry Setup for Distributed Tracing

## Objective

This document explains how to set up distributed tracing using Tempo and OpenTelemetry. Grafana is assumed to be already configured.

---

## Overview

Tempo and OpenTelemetry together provide tracing in the observability stack.

OpenTelemetry is used to generate and send traces from applications. It collects trace data such as request flow, service calls, and response timings.

Tempo is a tracing backend that receives, stores, and makes trace data available for querying in Grafana.

In this setup:

- OpenTelemetry acts as the trace generator and exporter
- Tempo acts as the centralized trace storage
- Grafana is used to search and visualize traces

This helps in understanding how a request flows across services and where delays or failures are happening.

---

## Architecture

Application → OpenTelemetry → Tempo → Grafana

---

## Prerequisites

- Linux VM for Tempo server
- Docker installed on Tempo server
- Grafana already available
- Network connectivity between application, Tempo, and Grafana

Important:
Port 3200 must be open for Tempo UI/API access.
Port 4317 must be open for OTLP gRPC ingestion if applications send traces using gRPC.
Port 4318 must be open for OTLP HTTP ingestion if applications send traces using HTTP.

---

## Part 1: Tempo Server Setup

### Step 1: Create Tempo Configuration

Create a file:

```bash
sudo nano tempo.yaml
````

Add the following configuration:

```yaml
server:
  http_listen_port: 3200

distributor:
  receivers:
    otlp:
      protocols:
        grpc:
        http:

storage:
  trace:
    backend: local
    local:
      path: /tmp/tempo/traces

compactor:
  compaction:
    block_retention: 24h
```

---

### Step 2: Run Tempo using Docker

```bash
docker pull grafana/tempo:2.4.1
```

```bash
docker run -d \
  --name tempo \
  -p 3200:3200 \
  -p 4317:4317 \
  -p 4318:4318 \
  -v $(pwd)/tempo.yaml:/etc/tempo.yaml \
  grafana/tempo:2.4.1 \
  -config.file=/etc/tempo.yaml
```

---

### Step 3: Verify Tempo Container

```bash
docker ps
```

Check Tempo logs if needed:

```bash
docker logs tempo
```

---

## Part 2: Open Required Ports

Ensure the following ports are open on the Tempo server:

* 3200
* 4317
* 4318

If using UFW:

```bash
sudo ufw allow 3200
sudo ufw allow 4317
sudo ufw allow 4318
```

Also ensure cloud security rules allow inbound traffic on these ports.

---

## Part 3: Verify Tempo Connectivity

Run from Tempo server:

```bash
curl http://localhost:3200/ready
```

Expected output:

```text
ready
```

From another VM:

```bash
curl http://<TEMPO_IP>:3200/ready
```

---

## Part 4: Configure Grafana with Tempo Datasource

### Step 1: Open Grafana

Open Grafana in browser and log in.

---

### Step 2: Add Tempo Datasource

* Go to Settings
* Click Data Sources
* Click Add data source
* Select Tempo

---

### Step 3: Configure Datasource

Set the following:

Name:

```text
Tempo
```

URL:

```text
http://<TEMPO_IP>:3200
```

Access:

```text
Server
```

Click Save & Test.

Expected result:

```text
Data source connected successfully
```

---

## Part 5: Application Trace Export Configuration

Applications need to send traces to Tempo using OpenTelemetry.

Two common export options are:

### OTLP gRPC

```text
http://<TEMPO_IP>:4317
```

### OTLP HTTP

```text
http://<TEMPO_IP>:4318
```

Use the endpoint supported by your application setup.

---

## Part 6: Example OpenTelemetry Environment Variables

Below is a simple example of environment variables that an application can use:

```bash
OTEL_SERVICE_NAME=my-app
OTEL_EXPORTER_OTLP_ENDPOINT=http://<TEMPO_IP>:4318
OTEL_EXPORTER_OTLP_PROTOCOL=http/protobuf
OTEL_TRACES_EXPORTER=otlp
```

This will send traces from the application to Tempo.

---

## Part 7: Verify Traces in Grafana

Open Grafana and go to Explore.

Select Tempo datasource.

Search traces using:

* Service name
* Trace ID
* Time range

If traces are flowing correctly, they will be visible in Grafana.

---

## Important Notes

* Tempo stores traces, not logs
* OpenTelemetry must be configured in the application or collector
* Use Tempo datasource in Grafana for traces
* Use a small time range during testing
* Ensure application and Tempo server can communicate over required ports

---

## Troubleshooting

### Tempo not running

```bash
docker ps
```

---

### Tempo logs

```bash
docker logs tempo
```

---

### Tempo not reachable

```bash
curl http://<TEMPO_IP>:3200/ready
```

---

### Port issue

Check listening ports:

```bash
ss -ltnp | grep -E '3200|4317|4318'
```

---

### No traces in Grafana

* Check application OpenTelemetry configuration
* Check Tempo datasource in Grafana
* Check correct Tempo endpoint
* Check time range
* Check container logs

---

## Conclusion

OpenTelemetry generates and exports traces from applications, and Tempo stores them centrally. Grafana can then be used to search and visualize traces, completing the observability stack with tracing support.
