P

# 🎯 1. Basic Project Questions (MUST PREPARE)

## ❓ Q1: Explain your observability project

**Answer:**

I implemented an end-to-end observability solution across multiple VMs using:

* Prometheus for metrics
* Loki + Promtail for centralized logging
* Tempo + OpenTelemetry for tracing
* Grafana for visualization

Logs, metrics, and traces are collected from multiple servers and visualized in a centralized Grafana dashboard, enabling real-time monitoring and troubleshooting.

---

## ❓ Q2: What is the difference between logs, metrics, and traces?

**Answer:**

* **Metrics** → numerical data (CPU, memory)
* **Logs** → event-based records (errors, auth logs)
* **Traces** → request flow across services

Together, they provide complete observability.

---

## ❓ Q3: What problem does Loki solve?

**Answer:**

Loki provides centralized log aggregation with **low storage cost** because it stores only labels as indexes instead of full log content.

---

# 🔥 2. Loki & Promtail Deep Questions

## ❓ Q4: How does Promtail work?

**Answer:**

Promtail:

* Reads log files from system (like `/var/log/syslog`)
* Adds labels (job, host)
* Pushes logs to Loki via HTTP API

---

## ❓ Q5: Why labels are important in Loki?

**Answer:**

Labels are used for:

* indexing logs
* filtering queries
* building dashboards

Example:

```logql
{job="syslog", host="20.6.128.221"}
```

---

## ❓ Q6: What happens if labels are wrong?

**Answer:**

* Logs may not be searchable
* Filtering will fail
* Multi-VM dashboard will break

---

## ❓ Q7: Why logs were not visible initially in your setup?

**Answer (real scenario):**

* Promtail was reading from file end (not old logs)
* Needed to generate fresh logs
* Also corrected host label mismatch

---

## ❓ Q8: Difference between Loki and ELK stack?

**Answer:**

| Loki                    | ELK                |
| ----------------------- | ------------------ |
| Lightweight             | Heavy              |
| Label-based indexing    | Full-text indexing |
| Cheap storage           | Expensive          |
| Works well with Grafana | Uses Kibana        |

---

# 🚀 3. Grafana Questions

## ❓ Q9: How did you implement multi-VM filtering?

**Answer:**

Using Grafana variables:

```logql
label_values({job="syslog"}, host)
```

Then used:

```logql
{job="syslog", host=~"$host"}
```

---

## ❓ Q10: Why use `=~` instead of `=`?

**Answer:**

* `=` → exact match
* `=~` → regex match (supports multi-select & all option)

---

## ❓ Q11: Why logs were not visible in panel but visible in Explore?

**Answer:**

Because:

* Panel was using **Time series**
* Logs require **Table or Explore**

---

## ❓ Q12: What kind of dashboards did you build?

**Answer:**

* Centralized multi-VM logs dashboard
* Error logs panel
* Authentication logs panel
* Failed login tracking

---

# 🔥 4. Real Troubleshooting Questions (VERY IMPORTANT)

## ❓ Q13: How did you debug "No logs in Grafana"?

**Answer:**

Step-by-step:

1. Checked Loki health (`/ready`)
2. Verified Promtail running
3. Checked syslog file content
4. Verified config (`__path__`)
5. Generated test logs
6. Checked labels mismatch
7. Switched to Explore view

---

## ❓ Q14: What are common issues in Promtail setup?

**Answer:**

* Wrong log path
* Permission issues
* Wrong labels
* Loki unreachable
* Positions file causing old offset

---

## ❓ Q15: What is positions file in Promtail?

**Answer:**

It tracks:

* last read position in log file
* prevents re-reading old logs

---

# 🔥 5. Tempo & Tracing Questions

## ❓ Q16: What is Tempo?

**Answer:**

Tempo is a distributed tracing backend used to store and query traces.

---

## ❓ Q17: How does tracing help?

**Answer:**

* Tracks request flow across services
* Helps debug latency issues
* Identifies bottlenecks

---

## ❓ Q18: What is OpenTelemetry?

**Answer:**

OpenTelemetry is a standard for collecting traces, metrics, and logs from applications.

---

# 🔥 6. Scenario-Based Questions (🔥 Interview Favorite)

## ❓ Q19: CPU high but logs normal — what will you check?

**Answer:**

* Metrics (Prometheus)
* Correlate with logs
* Check traces for slow requests

---

## ❓ Q20: Logs coming but not visible in Grafana?

**Answer:**

* Check labels
* Check query
* Check time range
* Check visualization type

---

## ❓ Q21: Multiple servers — how do you identify issue in one?

**Answer:**

Use host filter:

```logql
{host="20.6.128.221"}
```

---

# 🔥 7. Advanced Questions (Bonus)

## ❓ Q22: How will you scale this setup?

**Answer:**

* Use Loki distributed mode
* Store logs in object storage (S3/Azure Blob)
* Deploy on Kubernetes

---

## ❓ Q23: How will you secure logs?

**Answer:**

* Restrict access via Grafana roles
* Use HTTPS
* Mask sensitive data

---

## ❓ Q24: How will you implement alerting?

**Answer:**

* Use Grafana alert rules
* Trigger on:

  * error logs
  * high CPU
* Send notifications via Slack/email

---

# 🎯 Final Tip (VERY IMPORTANT)

If interviewer asks:

👉 "What did YOU do in this project?"

Say confidently:

* Installed and configured Promtail on multiple VMs
* Set up Loki for centralized logging
* Built Grafana dashboards with dynamic filtering
* Debugged real issues (logs not showing, label mismatch)
* Integrated tracing with Tempo

---

# 🏆 One-line interview summary

👉
**"I built a centralized observability platform using Grafana, Loki, Prometheus, and Tempo to monitor logs, metrics, and traces across multiple servers."**

---

