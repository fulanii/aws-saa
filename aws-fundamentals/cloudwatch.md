# **CloudWatch Basics**

[← Back to AWS Fundamentals](readme.md)

> AWS's monitoring service: it collects metrics, logs and events from AWS services, your apps and on-prem servers, and can raise alarms that trigger actions.

### Contents

- [What it is](#what-it-is)
- [How it fits together](#how-it-fits-together)
- [Namespaces and metrics](#namespaces-and-metrics)
- [Datapoints and dimensions](#datapoints-and-dimensions)
- [Alarms](#alarms)
- [Django/FastAPI map](#djangofastapi-map)
- [Exam notes](#exam-notes)

---

### What it is

CloudWatch is a **core supporting service** that collects and manages **operational data**. Many other AWS services use it for health and performance monitoring, log management and **event-driven** (serverless) architectures.

It has three main parts:

| Part | Collects | From | Examples |
|---|---|---|---|
| **CloudWatch Metrics** | Numbers over time | AWS products, your apps, on-prem servers | CPU utilization, network in/out, request count |
| **CloudWatch Logs** | Log lines | AWS products, your apps, on-prem servers | Web server logs, Linux system logs, Lambda logs |
| **CloudWatch Events** | "Something happened" signals | AWS services, schedules | An EC2 instance stopped; run something every day at 02:00 |

> CloudWatch Events has since become **Amazon EventBridge**, which does the same job and more. Exam questions may use either name.

---

### How it fits together

```
 Data sources                 CloudWatch                       Actions
 ────────────                 ──────────                       ───────
 AWS services     ──┐
 Your apps        ──┼──>  Metrics / Logs  ──>  Alarm  ──>  Auto Scaling (add EC2 instances)
 On-prem servers  ──┘          │                       └─>  SNS (email / SMS notification)
                               v
                     Console, CLI, API (view & query)
```

- You view and query the data through the **console, CLI or API**.
- **Alarms** watch a metric and trigger **actions**, e.g. scale out an Auto Scaling group or send a notification.
- Some data isn't collected by default. For EC2, **memory and disk space usage are not built-in metrics**: they need the **CloudWatch agent** installed on the instance. CPU, network and disk I/O come out of the box.

---

### Namespaces and metrics

**Namespace**: a **container** that keeps metrics separate.

- AWS services use namespaces named **`AWS/<service>`**, e.g. `AWS/EC2` holds all EC2 metric data.
- The `AWS/` prefix is **reserved** for AWS. Your own custom metrics go in your own namespace, e.g. `MyApp/API`.

**Metric**: a **time-ordered set of datapoints**, e.g. `CPUUtilization`, `NetworkIn`, `NetworkOut`, `DiskReadOps`.

- A metric isn't tied to one specific server. `CPUUtilization` in `AWS/EC2` receives data from **every** EC2 instance. Dimensions tell them apart (see below).

---

### Datapoints and dimensions

**Datapoint**: one measurement. Every time a server measures its CPU and sends it to CloudWatch, that's a new datapoint in the `CPUUtilization` metric. It has two parts:

- **Timestamp**: when it was measured, e.g. `2026-11-05T14:30:00Z`
- **Value**: the measurement, e.g. `98.3` (percent)

**Dimensions**: **name/value pairs** that split a metric into separate views, e.g.:

- `InstanceId=i-0abc123` → the CPU of one specific instance
- `InstanceType=t3.micro` → the CPU of all `t3.micro` instances
- `AutoScalingGroupName=web-asg` → the CPU across one Auto Scaling group

```
 Namespace  AWS/EC2
   └── Metric  CPUUtilization
         ├── Dimension InstanceId=i-0abc123   → [14:30, 98.3] [14:35, 72.0] ...
         └── Dimension InstanceId=i-0def456   → [14:30, 12.1] [14:35, 15.4] ...
```

---

### Alarms

An alarm is **linked to one metric** (and its dimensions) and compares it against a threshold, e.g. "CPU > 80% for 5 minutes".

| State | Meaning |
|---|---|
| **INSUFFICIENT_DATA** | Not enough data yet to decide. **Every alarm starts here.** |
| **OK** | The metric is within the threshold. All good. |
| **ALARM** | The threshold was breached. This is where you attach **actions** (scale out, send an SNS notification, ...). |

---

### Django/FastAPI map

- **Metrics**: like Prometheus/Grafana dashboards. Your app can push custom metrics (e.g. signups per minute, queue depth) with `boto3`'s `put_metric_data`.
- **Logs**: where your Gunicorn/Uvicorn and Django logs end up when they're shipped from EC2 (by the CloudWatch agent) or written automatically from Lambda/ECS, so you don't have to SSH into each server to tail logs.
- **Alarms**: like PagerDuty or Sentry alerts: "p95 latency too high" or "5xx errors spiking" → notify you, or add more servers.
- **Events / EventBridge schedules**: like cron or Celery beat, but managed by AWS.

---

### Exam notes

- CloudWatch = **metrics, logs and events**. Events is now **EventBridge**.
- **Namespace** = container (`AWS/EC2`). **Metric** = time-ordered datapoints. **Datapoint** = timestamp + value. **Dimension** = name/value pair that splits a metric.
- EC2 **memory and disk space usage need the CloudWatch agent**. A question about monitoring memory on EC2 is pointing at the agent.
- Alarm states: **OK**, **ALARM**, **INSUFFICIENT_DATA** (the starting state).
- Alarm actions commonly trigger **Auto Scaling** or **SNS** notifications.
