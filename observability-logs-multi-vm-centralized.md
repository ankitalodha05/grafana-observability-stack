
# Multi-VM Centralized Logging with Loki and Promtail

## Objective

This document explains how to configure multiple VMs to send logs to a single Loki server and visualize them in Grafana.

---

## Architecture

Multiple VMs → Promtail (on each VM) → Central Loki → Grafana

---

## Prerequisites

- Multiple Linux VMs
- One central Loki server
- Grafana already configured with Loki datasource
- Network connectivity from all VMs to Loki (port 3100 open)

---

## Step 1: Install Promtail on Each VM

On every VM, install Promtail:

```bash
wget https://github.com/grafana/loki/releases/download/v2.9.0/promtail-linux-amd64.zip
unzip promtail-linux-amd64.zip
chmod +x promtail-linux-amd64
sudo mv promtail-linux-amd64 /tmp/
````

---

## Step 2: Configure Promtail on Each VM

Create configuration file:

```bash
sudo nano /etc/promtail/config.yml
```

Use the following configuration:

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

* `<LOKI_IP>` with central Loki server IP
* `<VM_NAME>` with unique identifier for each VM (example: vm-1, vm-2, prod-app-1)

---

## Step 3: Start Promtail on Each VM

```bash
sudo /tmp/promtail-linux-amd64 -config.file=/etc/promtail/config.yml
```

---

## Step 4: Generate Test Logs

On each VM:

```bash
logger "multi vm test log"
```

---

## Step 5: Verify in Grafana

Open Grafana → Explore → Select Loki datasource

### View all logs

```logql
{job="syslog"}
```

### Filter by specific VM

```logql
{job="syslog", host="vm-1"}
```

### Search specific logs

```logql
{job="syslog"} |= "multi vm test"
```

---

## Step 6: Create Host-Based Filtering (Dashboard)

To dynamically filter logs by VM:

Use query:

```logql
label_values({job="syslog"}, host)
```

This will list all VM names sending logs.

---

## Important Notes

* Each VM must have a unique `host` label.
* Promtail reads only new logs after startup.
* Ensure time range is small during testing (last 5–15 minutes).
* Use Explore or Table view for logs.

---

## Troubleshooting

### Check Loki connectivity from each VM

```bash
curl http://<LOKI_IP>:3100/ready
```

---

### Check Promtail running

```bash
ps -ef | grep promtail
```

---

### Check logs available on VM

```bash
tail -n 20 /var/log/syslog
```

---

### Restart Promtail if needed

```bash
sudo pkill promtail
sudo rm -f /tmp/positions.yaml
sudo /tmp/promtail-linux-amd64 -config.file=/etc/promtail/config.yml
```

---

## Best Practices

* Use meaningful VM names in `host` label
* Keep same config structure across all VMs
* Separate logs by job (syslog, auth-log, application logs)
* Use centralized Loki for all environments

---

## Conclusion

Multiple VMs can send logs to a single Loki instance using Promtail. Logs can be filtered by host in Grafana, enabling centralized monitoring across all systems.

```
