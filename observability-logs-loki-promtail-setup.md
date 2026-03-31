
## 📘 Document

````markdown
# Loki and Promtail Setup on VM (Beginner Guide)

## Objective

This document explains how to configure Promtail on a VM to collect system logs and send them to Loki. Grafana dashboard is assumed to be already available.

---

## Architecture

VM Logs → Promtail → Loki → Grafana

---

## Prerequisites

- Linux VM (Ubuntu)
- Loki server running and accessible
- Grafana already configured
- Network access to Loki (port 3100)

---

## Step 1: Install Promtail

Download and prepare Promtail binary:

```bash
wget https://github.com/grafana/loki/releases/download/v2.9.0/promtail-linux-amd64.zip
unzip promtail-linux-amd64.zip
chmod +x promtail-linux-amd64
sudo mv promtail-linux-amd64 /tmp/
````

---

## Step 2: Create Promtail Configuration

Create config file:

```bash
sudo nano /etc/promtail/config.yml
```

Add the following configuration:

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
          host: <VM_IP>
          __path__: /var/log/syslog

  - job_name: auth-log
    static_configs:
      - targets:
          - localhost
        labels:
          job: auth-log
          host: <VM_IP>
          __path__: /var/log/auth.log
```

Replace:

* `<LOKI_IP>` with Loki server IP
* `<VM_IP>` with current VM IP

---

## Step 3: Start Promtail

Run Promtail:

```bash
sudo /tmp/promtail-linux-amd64 -config.file=/etc/promtail/config.yml
```

Promtail will start reading log files and sending data to Loki.

---

## Step 4: Generate Test Log

Create a test log entry:

```bash
logger "promtail test log"
```

---

## Step 5: Verify in Grafana

Open Grafana → Explore → Select Loki datasource

Run query:

```logql
{job="syslog"}
```

To filter test logs:

```logql
{job="syslog"} |= "promtail test"
```

---

## Important Notes

* Promtail reads logs from the end of the file, so only new logs are visible.
* Always generate a fresh log after starting Promtail.
* Logs should be viewed using Explore or Table format in Grafana.
* Time range should be small (last 5 or 15 minutes) during testing.

---

## Troubleshooting

### Check Loki connectivity

```bash
curl http://<LOKI_IP>:3100/ready
```

Expected output:

```
ready
```

---

### Check Promtail process

```bash
ps -ef | grep promtail
```

---

### Check log file

```bash
tail -n 20 /var/log/syslog
```

---

### Fix permission issues

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

Promtail collects logs from the VM and sends them to Loki. Logs can be queried and visualized in Grafana using LogQL queries.

```
