
# Centralized Multi-VM Logs Dashboard (Grafana + Loki + Promtail)

## Overview

This document explains how to create a **centralized multi-VM logging dashboard** in Grafana using Loki and Promtail.

The goal is to:
- Collect logs from multiple VMs
- Store logs in Loki
- Visualize logs in Grafana
- Filter logs dynamically using a **host dropdown**

---

## Architecture

```

Multiple VMs → Promtail → Loki → Grafana Dashboard

````

Each VM sends logs to Loki with a unique `host` label.

---

## Prerequisites

- Loki server is running and accessible
- Grafana is configured with Loki datasource
- Promtail is installed on all VMs
- Logs are already visible in Grafana Explore

---

## Step 1: Configure Promtail on Each VM

Each VM must have a **unique host label**

### Example (VM 1)

```yaml
scrape_configs:
  - job_name: syslog
    static_configs:
      - targets:
          - localhost
        labels:
          job: syslog
          host: 20.6.128.221
          __path__: /var/log/syslog

  - job_name: auth-log
    static_configs:
      - targets:
          - localhost
        labels:
          job: auth-log
          host: 20.6.128.221
          __path__: /var/log/auth.log
````

### Example (VM 2)

```yaml
labels:
  job: syslog
  host: 4.247.131.121
```

### Important Notes

* `job` can be same across VMs
* `host` must be unique
* Label naming must be consistent

---

## Step 2: Restart Promtail

```bash
sudo pkill promtail
sudo /tmp/promtail-linux-amd64 -config.file=/etc/promtail/config.yml
```

---

## Step 3: Generate Test Logs

Run on each VM:

```bash
logger "test log from this vm"
```

---

## Step 4: Verify Logs in Grafana

Go to:

**Grafana → Explore → Loki**

Run:

```logql
{job="syslog"}
```

You should see logs from all VMs.

---

## Step 5: Create New Dashboard

1. Go to **Dashboards**
2. Click **New Dashboard**
3. Click **Add Visualization**
4. Select **Loki datasource**

---

## Step 6: Create Host Dropdown Variable

Go to:

**Dashboard Settings → Variables → Add Variable**

### Configuration

| Field       | Value |
| ----------- | ----- |
| Name        | host  |
| Label       | Host  |
| Type        | Query |
| Data Source | Loki  |

### Query

```logql
label_values({job="syslog"}, host)
```

### Options

* Enable **Multi-value**
* Enable **Include All option**

Click **Save**

### Result

A dropdown will appear at the top of dashboard:

```
Host ▼
```

---

## Step 7: Create Panels

---

### Panel 1: System Logs

```logql
{job="syslog", host=~"$host"}
```

Title:

```
System Logs
```

---

### Panel 2: Authentication Logs

```logql
{job="auth-log", host=~"$host"}
```

Title:

```
Authentication Logs
```

---

### Panel 3: Error Logs

```logql
{job="syslog", host=~"$host"} |= "error"
```

Title:

```
Error Logs
```

---

### Panel 4: Failed Login Attempts

```logql
{job="auth-log", host=~"$host"} |= "Failed password"
```

Title:

```
Failed Login Attempts
```

---

## Step 8: Save Dashboard

Click **Save Dashboard**

Suggested Name:

```
Centralized Multi-VM Logs Dashboard
```

---

## Query Patterns

### All Logs

```logql
{job="syslog", host=~"$host"}
```

### Specific Host

```logql
{job="syslog", host="20.6.128.221"}
```

### Filter Logs

```logql
{job="syslog"} |= "error"
```

---

## Important Concepts

### 1. host=~"$host"

* Supports dropdown
* Supports multi-select
* Supports "All" option

### 2. Logs Visualization

Use:

* Explore (best)
* Table panel

Avoid:

* Time series (not suitable for logs)

---

## Common Issues

### No data

* Check Promtail running
* Generate fresh logs
* Check time range

### Dropdown empty

* Loki has no data yet
* Run test logs

### Wrong filtering

Use:

```logql
host=~"$host"
```

Not:

```logql
host="$host"
```

---

## Best Practices

Add more labels for better filtering:

```yaml
labels:
  job: syslog
  host: 20.6.128.221
  vm: azuresr
  env: prod
  cloud: azure
```

---

## Advanced Improvements

You can create additional dropdowns:

### VM dropdown

```logql
label_values({job="syslog"}, vm)
```

### Environment dropdown

```logql
label_values({job="syslog"}, env)
```

---

## Summary

* Each VM sends logs with unique host label
* Grafana uses `host` variable for filtering
* Dashboard provides centralized log visibility
* Logs can be filtered dynamically across multiple VMs

---

## One-line Conclusion

A single Grafana dashboard can monitor logs from multiple VMs using a dynamic host-based dropdown filter.

