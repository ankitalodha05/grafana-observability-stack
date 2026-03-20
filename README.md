# Prometheus + Grafana + Alertmanager + Node Exporter Setup Runbook

## Objective

This document captures the end-to-end process that was successfully implemented for VM monitoring using:

- Prometheus
- Grafana
- Alertmanager
- Node Exporter

This setup was validated successfully for:

- 1 monitoring server
- 1 target VM

The full monitoring flow, dashboard visibility, and alert validation were completed successfully.

---

## 1. Scope

### Phase 1

For the initial setup, monitoring was configured only for VMs:

- Utho monitoring server
- One Azure target VM

### Components used

- **Prometheus** → metrics collection and storage
- **Grafana** → dashboard visualization
- **Alertmanager** → alert handling
- **Node Exporter** → server metrics exposure

### Monitoring flow

```text
Target VM → Node Exporter → Prometheus → Grafana
Prometheus → Alertmanager

# 2. Server Details

## Monitoring Server

* Used as central monitoring server
* Hosted on Utho
* Public IP: `103.150.136.119`

Installed on this server:

* Prometheus
* Grafana
* Alertmanager
* Node Exporter

## Target VM

* Azure VM
* Target IP used in setup: `20.6.128.221`

Installed on this VM:

* Node Exporter only

---

# 3. Initial Validation Before Setup

## 3.1 Check server configuration

Run on monitoring server:

```bash
lscpu
free -h
df -h
uptime
top
ps -eo pid,user,cmd,%mem,%cpu,rss --sort=-%mem | head -15
```

## 3.2 Observation

The monitoring server already had SonarQube running, so resource usage was checked carefully before installation.

## 3.3 Port validation

Checked whether required ports were free:

```bash
ss -tulnp | grep -E ':(9090|3000|9093|9100)\s'
```

If command returns nothing, ports are free.

Required ports:

* `9090` → Prometheus
* `3000` → Grafana
* `9093` → Alertmanager
* `9100` → Node Exporter

---

# 4. Firewall / Security Group Rules

## 4.1 Monitoring server (Utho)

The following ports were required on the monitoring server:

* `3000` → Grafana
* `9090` → Prometheus
* `9093` → Alertmanager

These were opened in the **Utho security group**.

### Example rules

* Service: `CUSTOM`
* Protocol: `TCP`
* Port Range: `3000`
* Source: `ALL` (used for testing; can be restricted later)

Similarly added:

* `9090`
* `9093`

## 4.2 Target VM (Azure)

For the target Azure VM, port `9100` was allowed in NSG.

### Rule details

* Source: monitoring server IP `103.150.136.119`
* Protocol: `TCP`
* Destination port: `9100`
* Action: `Allow`

This allowed Prometheus on the monitoring server to scrape Node Exporter metrics from the target VM.

---

# 5. Monitoring Server Installation

## 5.1 Update packages

Run on monitoring server:

```bash
apt update && apt upgrade -y
apt install -y curl wget tar gnupg2 software-properties-common apt-transport-https
```

---

## 5.2 Install Prometheus

### Create user and directories

```bash
useradd --no-create-home --shell /bin/false prometheus || true
mkdir -p /etc/prometheus /var/lib/prometheus
chown prometheus:prometheus /etc/prometheus /var/lib/prometheus
```

### Download and copy binaries

```bash
cd /tmp
curl -LO https://github.com/prometheus/prometheus/releases/download/v2.55.1/prometheus-2.55.1.linux-amd64.tar.gz
tar -xvf prometheus-2.55.1.linux-amd64.tar.gz
cd prometheus-2.55.1.linux-amd64

cp prometheus promtool /usr/local/bin/
chown prometheus:prometheus /usr/local/bin/prometheus /usr/local/bin/promtool

cp -r consoles console_libraries /etc/prometheus/
cp prometheus.yml /etc/prometheus/prometheus.yml
chown -R prometheus:prometheus /etc/prometheus
```

### Create Prometheus service

```bash
cat >/etc/systemd/system/prometheus.service <<'EOF'
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/var/lib/prometheus/ \
  --web.console.templates=/etc/prometheus/consoles \
  --web.console.libraries=/etc/prometheus/console_libraries \
  --storage.tsdb.retention.time=15d
Restart=always

[Install]
WantedBy=multi-user.target
EOF
```

### Enable service

```bash
systemctl daemon-reload
systemctl enable prometheus
```

---

## 5.3 Install Alertmanager

### Create user and directories

```bash
useradd --no-create-home --shell /bin/false alertmanager || true
mkdir -p /etc/alertmanager /var/lib/alertmanager
chown -R alertmanager:alertmanager /etc/alertmanager /var/lib/alertmanager
```

### Download and copy binaries

```bash
cd /tmp
curl -LO https://github.com/prometheus/alertmanager/releases/download/v0.27.0/alertmanager-0.27.0.linux-amd64.tar.gz
tar -xvf alertmanager-0.27.0.linux-amd64.tar.gz
cd alertmanager-0.27.0.linux-amd64

cp alertmanager amtool /usr/local/bin/
chown alertmanager:alertmanager /usr/local/bin/alertmanager /usr/local/bin/amtool
```

### Create Alertmanager config

```bash
cat >/etc/alertmanager/alertmanager.yml <<'EOF'
route:
  receiver: default-receiver
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 1h

receivers:
  - name: default-receiver
EOF
```

```bash
chown alertmanager:alertmanager /etc/alertmanager/alertmanager.yml
```

### Create Alertmanager service

```bash
cat >/etc/systemd/system/alertmanager.service <<'EOF'
[Unit]
Description=Alertmanager
Wants=network-online.target
After=network-online.target

[Service]
User=alertmanager
Group=alertmanager
Type=simple
ExecStart=/usr/local/bin/alertmanager \
  --config.file=/etc/alertmanager/alertmanager.yml \
  --storage.path=/var/lib/alertmanager
Restart=always

[Install]
WantedBy=multi-user.target
EOF
```

### Enable service

```bash
systemctl daemon-reload
systemctl enable alertmanager
```

---

## 5.4 Install Grafana

```bash
mkdir -p /etc/apt/keyrings
curl -fsSL https://packages.grafana.com/gpg.key | gpg --dearmor -o /etc/apt/keyrings/grafana.gpg

echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://packages.grafana.com/oss/deb stable main" | tee /etc/apt/sources.list.d/grafana.list

apt update
apt install -y grafana
systemctl enable grafana-server
```

---

## 5.5 Install Node Exporter on Monitoring Server

```bash
useradd --no-create-home --shell /bin/false node_exporter || true

cd /tmp
curl -LO https://github.com/prometheus/node_exporter/releases/download/v1.8.2/node_exporter-1.8.2.linux-amd64.tar.gz
tar -xvf node_exporter-1.8.2.linux-amd64.tar.gz
cd node_exporter-1.8.2.linux-amd64

cp node_exporter /usr/local/bin/
chown node_exporter:node_exporter /usr/local/bin/node_exporter
```

### Create service

```bash
cat >/etc/systemd/system/node_exporter.service <<'EOF'
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter
Restart=always

[Install]
WantedBy=multi-user.target
EOF
```

### Enable service

```bash
systemctl daemon-reload
systemctl enable node_exporter
```

---

# 6. Prometheus Configuration

## 6.1 Prometheus main config

File: `/etc/prometheus/prometheus.yml`

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

alerting:
  alertmanagers:
    - static_configs:
        - targets: ["localhost:9093"]

rule_files:
  - "alert.rules.yml"

scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]

  - job_name: "monitoring-server-node"
    static_configs:
      - targets: ["localhost:9100"]

  - job_name: "target-vms"
    static_configs:
      - targets:
          - "20.6.128.221:9100"
```

Set permissions:

```bash
chown prometheus:prometheus /etc/prometheus/prometheus.yml
```

---

## 6.2 Alert rules

File: `/etc/prometheus/alert.rules.yml`

```yaml
groups:
  - name: vm-alerts
    rules:
      - alert: InstanceDown
        expr: up == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Instance {{ $labels.instance }} is down"
          description: "Prometheus target {{ $labels.instance }} has been unreachable for more than 1 minute."

      - alert: HighCPUUsage
        expr: 100 - (avg by(instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
        for: 2m
        labels:
          severity: warning
        annotations:
          summary: "High CPU usage on {{ $labels.instance }}"
          description: "CPU usage is above 80% for more than 2 minutes."

      - alert: HighDiskUsage
        expr: (1 - (node_filesystem_avail_bytes{fstype!~"tmpfs|overlay"} / node_filesystem_size_bytes{fstype!~"tmpfs|overlay"})) * 100 > 85
        for: 2m
        labels:
          severity: warning
        annotations:
          summary: "High disk usage on {{ $labels.instance }}"
          description: "Disk usage is above 85% for more than 2 minutes."
```

Set permissions:

```bash
chown prometheus:prometheus /etc/prometheus/alert.rules.yml
```

---

## 6.3 Validate Prometheus config

```bash
promtool check config /etc/prometheus/prometheus.yml
promtool check rules /etc/prometheus/alert.rules.yml
```

---

# 7. Start Monitoring Server Services

```bash
systemctl start alertmanager
systemctl start node_exporter
systemctl start prometheus
systemctl start grafana-server
```

## Check status

```bash
systemctl status alertmanager --no-pager
systemctl status node_exporter --no-pager
systemctl status prometheus --no-pager
systemctl status grafana-server --no-pager
```

## Check listening ports

```bash
ss -tulnp | grep -E ':(9090|3000|9093|9100)\s'
```

Expected:

* 3000 → Grafana
* 9090 → Prometheus
* 9093 → Alertmanager
* 9100 → Node Exporter

---

# 8. Target VM Setup

## 8.1 SSH into target VM

Logged into target VM using user account.

Since installation needed admin privileges, `sudo` access was verified:

```bash
sudo -l
```

Result confirmed full sudo access.

## 8.2 Switch to root

```bash
sudo -i
```

## 8.3 Install Node Exporter on target VM

```bash
apt update && apt upgrade -y
apt install -y curl wget tar

useradd --no-create-home --shell /bin/false node_exporter || true

cd /tmp
rm -rf node_exporter-1.8.2.linux-amd64*
curl -LO https://github.com/prometheus/node_exporter/releases/download/v1.8.2/node_exporter-1.8.2.linux-amd64.tar.gz
tar -xvf node_exporter-1.8.2.linux-amd64.tar.gz
cd node_exporter-1.8.2.linux-amd64

cp node_exporter /usr/local/bin/
chown node_exporter:node_exporter /usr/local/bin/node_exporter
```

## 8.4 Create service on target VM

```bash
cat >/etc/systemd/system/node_exporter.service <<'EOF'
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter
Restart=always

[Install]
WantedBy=multi-user.target
EOF
```

## 8.5 Enable and start service

```bash
systemctl daemon-reload
systemctl enable node_exporter
systemctl start node_exporter
systemctl status node_exporter --no-pager
```

## 8.6 Verify locally on target VM

```bash
curl http://localhost:9100/metrics | head
ss -tulnp | grep :9100
```

---

# 9. Test Connectivity from Monitoring Server

After Node Exporter was installed on target VM and NSG allowed `9100`, connectivity was tested from the monitoring server:

```bash
curl -s http://20.6.128.221:9100/metrics | head
```

This confirmed:

* target VM Node Exporter is working
* monitoring server can reach the target VM
* firewall / NSG rule is correct

---

# 10. Prometheus Validation

## 10.1 Open Prometheus UI

```text
http://103.150.136.119:9090
```

## 10.2 Open Targets page

```text
http://103.150.136.119:9090/targets
```

Verified targets:

* `prometheus` → `localhost:9090`
* `monitoring-server-node` → `localhost:9100`
* `target-vms` → `20.6.128.221:9100`

All targets were shown as **UP**.

---

# 11. Grafana Validation

## 11.1 Open Grafana UI

```text
http://103.150.136.119:3000
```

Default login:

* Username: `*****`
* Password: `*****`

## 11.2 Add Prometheus datasource

In Grafana:

* Go to **Connections**
* Open **Data sources**
* Add **Prometheus**

Prometheus URL used:

```text
http://localhost:9090
```

Clicked **Save & test** successfully.

## 11.3 Import dashboard

Dashboard imported using Grafana ID:

```text
1860
```

Dashboard name:

* **Node Exporter Full**

## 11.4 View metrics

Dashboard was validated for:

* monitoring server
* target VM

Filters used:

* Job = `monitoring-server-node`
* Instance = `localhost:9100`

Then changed to:

* Job = `target-vms`
* Instance = `20.6.128.221:9100`

Metrics visible:

* CPU
* Memory
* Disk
* Filesystem
* Load
* Network

---

# 12. Alertmanager Validation

## 12.1 Open Alertmanager UI

```text
http://103.150.136.119:9093
```

Alertmanager UI was accessible successfully.

Initially it showed:

* No alert groups found

This was normal because no alert was firing at that time.

---

# 13. End-to-End Alert Test

## 13.1 Alert tested

Tested alert:

* `InstanceDown`

## 13.2 How test was done

On target VM, Node Exporter was stopped temporarily:

```bash
sudo systemctl stop node_exporter
```

Because alert rule had:

```yaml
for: 1m
```

after around 1 minute:

* Prometheus showed `InstanceDown` alert as **firing**
* Alertmanager also showed the alert

This validated the end-to-end alerting flow.

## 13.3 Recovery

Node Exporter was started again:

```bash
sudo systemctl start node_exporter
```

Then target returned to **UP** state.

---

# 14. Final Validation Status

The following were validated successfully:

## Monitoring server

* Prometheus installed and accessible
* Grafana installed and accessible
* Alertmanager installed and accessible
* Node Exporter installed and working

## Target VM

* Node Exporter installed and working
* Metrics accessible from monitoring server
* Prometheus scraping successfully

## Dashboards

* Grafana showing metrics for both:

  * monitoring server
  * target VM

## Alerting

* `InstanceDown` alert fired successfully
* Alert visible in both Prometheus and Alertmanager

---

# 15. Outcome

Initial monitoring setup was completed successfully.

Working components:

* Prometheus
* Grafana
* Alertmanager
* Node Exporter

Validated successfully for:

* 1 monitoring server
* 1 Azure target VM

End-to-end process is working fine.

---

# 16. Next Steps

## Immediate next steps

* Onboard additional Azure and Utho VMs
* Repeat Node Exporter installation on each VM
* Add each VM IP under `target-vms` in `prometheus.yml`
* Restart Prometheus after updating config

## Future work

* Review monitoring method for **Azure WebApps**
* WebApps monitoring flow will be different because it is a **platform-managed service**

---

# 17. Useful Commands Summary

## Service status

```bash
systemctl status prometheus --no-pager
systemctl status grafana-server --no-pager
systemctl status alertmanager --no-pager
systemctl status node_exporter --no-pager
```

## Port check

```bash
ss -tulnp | grep -E ':(9090|3000|9093|9100)\s'
```

## Prometheus config validation

```bash
promtool check config /etc/prometheus/prometheus.yml
promtool check rules /etc/prometheus/alert.rules.yml
```

## Test target metrics from monitoring server

```bash
curl -s http://20.6.128.221:9100/metrics | head
```

---

# 18. Key Notes

* For VM monitoring, the same Prometheus + Node Exporter approach works well across:

  * Azure VM
  * Utho VM
  * AWS EC2
* Difference mainly comes in:

  * firewall / NSG rules
  * access method
  * cloud-specific security controls
* For Azure WebApps, monitoring will need a different review because it is a managed platform service.

---

# 19. Success Statement

Initial monitoring setup has been completed successfully. Prometheus, Grafana, Node Exporter, and Alertmanager are working fine. Metrics for the monitoring server and one target VM are visible in Grafana, and the alert flow has also been tested successfully using the InstanceDown alert. The end-to-end process is working fine.

---

```
```
