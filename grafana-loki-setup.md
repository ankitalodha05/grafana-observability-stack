
# Grafana Loki Datasource Configuration

## Objective

This document explains how to configure Loki as a datasource in Grafana so that logs stored in Loki can be queried and visualized.

---

## Prerequisites

- Grafana installed and accessible
- Loki server running
- Network connectivity between Grafana and Loki

Important:
Loki must be reachable on port 3100 from the Grafana server.

---

## Step 1: Verify Loki Connectivity

Run the following from Grafana server or any VM:

```bash
curl http://<LOKI_IP>:3100/ready
````

Expected output:

```
ready
```

---

## Step 2: Open Grafana

Open Grafana in browser:

```
http://<GRAFANA_IP>:3000
```

Login with valid credentials.

---

## Step 3: Navigate to Data Sources

* Click on Settings (gear icon)
* Click on Data Sources

---

## Step 4: Add Loki Datasource

* Click Add data source
* Select Loki

---

## Step 5: Configure Datasource

Fill the following details:

### Name

```
Loki
```

### URL

```
http://<LOKI_IP>:3100
```

### Access

```
Server (default)
```

---

## Step 6: Save and Test

Click Save & Test

Expected result:

```
Data source connected successfully
```

---

## Step 7: Verify Logs

Go to Explore section in Grafana.

Select Loki datasource and run:

```logql
{job="syslog"}
```

If logs are available in Loki, they will be displayed.

---

## Troubleshooting

### Loki not reachable

```bash
curl http://<LOKI_IP>:3100/ready
```

* Check Loki is running
* Check firewall rules
* Check port 3100 is open

---

### Check Loki container

```bash
docker ps
```

---

### No logs in Grafana

* Promtail may not be running
* Logs not pushed to Loki
* Wrong query used
* Time range too large

---

## Important Notes

* Use Loki server IP, not localhost (if Grafana is on different VM)
* Ensure port 3100 is open
* Ensure Loki is accessible from Grafana server

---

## Conclusion

Grafana is configured with Loki as a datasource. Logs stored in Loki can now be queried and visualized using LogQL.

