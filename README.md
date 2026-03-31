
# 🚀 Observability Stack DevOps

## 📌 Overview

This project demonstrates a **complete end-to-end observability setup** using modern DevOps tools across multiple virtual machines.

It includes:

* 📊 Metrics Monitoring (Prometheus + Grafana)
* 📜 Centralized Logging (Loki + Promtail)
* 🔍 Distributed Tracing (Tempo + OpenTelemetry)
* 🚨 Alerting setup for proactive monitoring

The solution provides **centralized visibility** into infrastructure and application behavior.

---

## 🏗️ Architecture

```
VMs (Azure / Utho)
        │
        ├── Node Exporter (Metrics)
        ├── Promtail (Logs)
        ├── OpenTelemetry (Traces)
        │
        ▼
Monitoring Server
        │
        ├── Prometheus (Metrics Collection)
        ├── Loki (Log Aggregation)
        ├── Tempo (Tracing Backend)
        │
        ▼
Grafana Dashboard
        │
        ├── Metrics Visualization
        ├── Logs Exploration
        ├── Traces Analysis
        └── Alerting
```

---

## 🛠️ Tools & Technologies

* **Grafana** – Visualization & dashboards
* **Prometheus** – Metrics monitoring
* **Loki** – Log aggregation
* **Promtail** – Log collection agent
* **Tempo** – Distributed tracing backend
* **OpenTelemetry** – Tracing instrumentation
* **Node Exporter / Windows Exporter** – System metrics
* **Azure / Utho VMs** – Infrastructure

---

## 📂 Repository Structure

```
.
├── grafana-loki-setup.md
├── observability-logs-end-to-end-loki-promtail.md
├── observability-logs-multi-vm-dashboard.md
├── observability-tracing-tempo-opentelemetry.md
├── observability-tracing-otel-collector-tempo.md
└── vm-monitoring.md
```

---

## 🔹 Features Implemented

### ✅ Centralized Logging

* Logs collected from multiple VMs
* Stored in Loki
* Visualized in Grafana
* Multi-VM filtering using dropdown

---

### ✅ Metrics Monitoring

* CPU, Memory, Disk, Network metrics
* Prometheus-based scraping
* Grafana dashboards for visualization

---

### ✅ Distributed Tracing

* Tempo integration with Grafana
* OpenTelemetry setup for tracing
* End-to-end request visibility

---

### ✅ Multi-VM Dashboard

* Single dashboard for all servers
* Dynamic host-based filtering
* Error and auth log tracking

---

## 📊 Example Use Cases

* Monitor CPU spikes across VMs
* Identify failed login attempts
* Debug application latency using traces
* Centralize logs for troubleshooting
* Track system health in real-time

---

## 🔍 Sample Queries

### View all logs

```
{job="syslog"}
```

### Filter by host

```
{job="syslog", host="20.6.128.221"}
```

### Error logs

```
{job="syslog"} |= "error"
```

### Failed login attempts

```
{job="auth-log"} |= "Failed password"
```

---

## ⚙️ Key Learnings

* Importance of **label-based filtering in Loki**
* Difference between **metrics, logs, and traces**
* Real-time observability pipeline design
* Debugging distributed systems using traces
* Visualization best practices in Grafana

---

## 🚀 Future Enhancements

* Alerting using Grafana Alert Manager
* Slack / Email notification integration
* Kubernetes (AKS/EKS) observability
* Application-level logging (Node/Java apps)
* JSON structured logs parsing

---

## 📌 Conclusion

This project demonstrates a **production-style observability implementation** combining logs, metrics, and traces into a single unified platform.

It provides a strong foundation for **DevOps monitoring, debugging, and performance analysis**.

---

## 👩‍💻 Author

**Ankita Lodha**
DevOps Engineer
---
