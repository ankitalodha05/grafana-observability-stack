
# End-to-End Logging Setup using Loki and Promtail

## Objective

This document explains how to set up centralized logging using Loki and Promtail across multiple VMs. Grafana is assumed to be already configured.

---

## Overview

Loki and Promtail together provide a centralized logging solution.

Promtail runs on each VM and collects logs from local files such as /var/log/syslog and /var/log/auth.log. It continuously monitors these files and sends new log entries to Loki.

Loki is a log aggregation system that receives logs from multiple Promtail agents and stores them in a structured way. It indexes logs based on labels such as job and host, which makes it easy to search and filter logs.

In this setup:

- Promtail acts as the log collector and forwarder
- Loki acts as the centralized log storage
- Grafana is used to query and visualize the logs

This allows logs from multiple VMs to be collected in one place and analyzed efficiently.

---

## Architecture

Multiple VMs → Promtail → Loki → Grafana

---

## Prerequisites

- Loki server VM
- Multiple application VMs
- Grafana already configured with Loki datasource
- Docker installed on Loki server
- Network connectivity between VMs and Loki

Important:
Port 3100 must be open on Loki server and accessible from all VMs.

---

## Part 1: Loki Server Setup

### Step 1: Open Port 3100

Allow inbound traffic on Loki server:

```bash
sudo ufw allow 3100
````

Also ensure cloud security rules allow port 3100.

---

### Step 2: Run Loki using Docker

```bash
docker pull grafana/loki:2.9.0

docker run -d \
  --name loki \
  -p 3100:3100 \
  grafana/loki:2.9.0
```

---

### Step 3: Verify Loki

```bash
curl http://localhost:3100/ready
```

Expected output:

```
ready
```

From another VM:

```bash
curl http://<LOKI_IP>:3100/ready
```

---

## Part 2: Promtail Setup on Each VM

### Step 1: Install Promtail

```bash
wget https://github.com/grafana/loki/releases/download/v2.9.0/promtail-linux-amd64.zip
unzip promtail-linux-amd64.zip
chmod +x promtail-linux-amd64
sudo mv promtail-linux-amd64 /tmp/
```

---

### Step 2: Create Promtail Config

```bash
sudo nano /etc/promtail/config.yml
```

```yaml
server:
  http_listen_port: 9080

positions:
  filename: /tmp/positions.yaml

clients:
  - url: http://<LOKI_IP>:3100/loki/api/v1/push

scrape_configs:
  - job_name: syslog
    static_configs:
      - targets:
          - localhost
        labels:
          job: syslog
          host: <VM_NAME>
          __path__: /var/log/syslog

  - job_name: auth-log
    static_configs:
      - targets:
          - localhost
        labels:
          job: auth-log
          host: <VM_NAME>
          __path__: /var/log/auth.log
```

Replace:

* <LOKI_IP> with Loki server IP
* <VM_NAME> with unique VM identifier

---

### Step 3: Start Promtail

```bash
sudo /tmp/promtail-linux-amd64 -config.file=/etc/promtail/config.yml
```

---

### Step 4: Generate Test Log

```bash
logger "loki promtail test log"
```

---

## Part 3: Verification in Grafana

Open Grafana → Explore → Select Loki datasource

### View all logs

```logql
{job="syslog"}
```

### Filter by VM

```logql
{job="syslog", host="<VM_NAME>"}
```

### Search specific log

```logql
{job="syslog"} |= "test log"
```

---

## Important Notes

* Promtail reads logs from the end of the file, so only new logs are visible
* Always generate fresh logs after starting Promtail
* Use small time range (last 5–15 minutes) during testing
* Logs should be viewed in Explore or Table format

---

## Troubleshooting

### Loki not reachable

```bash
curl http://<LOKI_IP>:3100/ready
```

---

### Promtail not running

```bash
ps -ef | grep promtail
```

---

### Log file check

```bash
tail -n 20 /var/log/syslog
```

---

### Permission fix

```bash
sudo chmod 644 /var/log/syslog
sudo chmod 644 /var/log/auth.log
```

---

### Restart Promtail

```bash
sudo pkill promtail
sudo rm -f /tmp/positions.yaml
sudo /tmp/promtail-linux-amd64 -config.file=/etc/promtail/config.yml
```

---

## Conclusion

Logs from multiple VMs are collected using Promtail and sent to a central Loki server. Grafana is used to query and visualize logs using LogQL.

```
