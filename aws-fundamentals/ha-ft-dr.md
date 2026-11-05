# **High Availability vs Fault Tolerance vs Disaster Recovery**

[← Back to AWS Fundamentals](readme.md)

> Three terms that sound alike but mean different things, and the exam picks answers based on exactly which one a question is asking for.

### Contents

- [**High Availability vs Fault Tolerance vs Disaster Recovery**](#high-availability-vs-fault-tolerance-vs-disaster-recovery)
    - [Contents](#contents)
    - [The short version](#the-short-version)
    - [High Availability (HA)](#high-availability-ha)
    - [Fault Tolerance (FT)](#fault-tolerance-ft)
    - [Disaster Recovery (DR)](#disaster-recovery-dr)
    - [Side by side](#side-by-side)
    - [Django/FastAPI map](#djangofastapi-map)
    - [Exam notes](#exam-notes)

---

### The short version

| Term | Goal | Analogy |
|---|---|---|
| **High Availability** | **Minimize** outages and recover **fast** | A car with a spare tire: you stop briefly, swap it, drive on |
| **Fault Tolerance** | **Keep operating through** a failure, with no outage | A plane with several engines: one fails, it keeps flying |
| **Disaster Recovery** | **Recover** when HA and FT weren't enough | The plan, backups and ejector seat for when everything else fails |

---

### High Availability (HA)

HA aims to **keep a system up for a higher than normal share of the time**, to an agreed level (usually uptime).

- It's about **maximizing uptime**, not preventing every failure. A short disruption is acceptable as long as recovery is **fast** and ideally **automatic**.
- Users may notice a brief blip (e.g. reconnecting, logging in again).
- Achieved with **redundancy and automatic replacement**: spare capacity in other AZs, health checks, automatic failover.

**Uptime is measured in "nines":**

| Uptime | Downtime per year |
|---|---|
| 99.9% ("three nines") | ~8.77 hours |
| 99.99% ("four nines") | ~52.6 minutes |
| 99.999% ("five nines") | ~5.26 minutes |

**AWS examples:**

- An **Auto Scaling group across multiple AZs** behind a load balancer: if an instance or AZ dies, it's replaced automatically.
- **RDS Multi-AZ**: if the primary database fails, AWS fails over to the standby. There's a short interruption (typically a minute or two), so it's HA, not FT.

---

### Fault Tolerance (FT)

FT means the system **continues operating properly even when some of its components fail**: no outage, no disruption.

- HA says "fail, then recover quickly". FT says "**operate through the failure**".
- Needed where even a short outage is unacceptable: medical systems, aircraft, payment processing.
- Requires **active redundancy** with enough spare capacity to absorb a failure instantly, so it's **more expensive and more complex** than HA.

**AWS example:**

- Your API needs **4 instances** to handle its load. With HA you might run 4 across two AZs and replace any that fail. With **FT** you run **4 in AZ-a and 4 in AZ-b**, so if an entire AZ fails, the other AZ can still carry the full load with no slowdown.

> **HA is not FT.** A highly available system can still have brief outages. A fault-tolerant system is built not to.

---

### Disaster Recovery (DR)

DR is the **set of policies, tools and procedures** used to **recover or continue** vital systems after a disaster, natural or human-made: a region outage, a fire, deleted data, ransomware.

- It's what you fall back on **when HA and FT fail** or don't cover the problem.
- Two parts:
  - **Pre-planning**: backups stored somewhere else (another region or account), standby infrastructure, documented runbooks.
  - **The DR process**: actually restoring and switching over, ideally **tested and automated**.
- It's measured with two numbers (covered in depth later in the course):
  - **RPO (Recovery Point Objective)**: how much **data** you can afford to lose (e.g. the last 1 hour).
  - **RTO (Recovery Time Objective)**: how much **time** you can afford to be down (e.g. 4 hours).

**AWS examples:**

- Backups and snapshots copied to **another region** (AWS Backup, EBS/RDS snapshot copies).
- **S3 Cross-Region Replication**.
- A scaled-down copy of your environment in another region, ready to scale up. The four DR strategies (backup & restore, pilot light, warm standby, multi-site active/active) come later in the course.

---

### Side by side

| | High Availability | Fault Tolerance | Disaster Recovery |
|---|---|---|---|
| **Question it answers** | How do we keep downtime low? | How do we never go down? | How do we come back after a disaster? |
| **Outage during failure?** | Brief, recovers fast | None | Yes, until recovery completes |
| **Typical scope** | Instance or AZ failure | Instance or AZ failure | Region failure, data loss, major incidents |
| **Cost / complexity** | Medium | High | Varies with RPO/RTO targets |
| **AWS building blocks** | Multi-AZ ASG + ALB, RDS Multi-AZ | Active-active, over-provisioned across AZs | Cross-region backups and replication, standby environment |

---

### Django/FastAPI map

- **HA**: run Gunicorn/Uvicorn on 2+ servers in different AZs behind a load balancer, with health checks hitting `/health`. If one server dies, the load balancer stops sending it traffic and a new one boots. Users might see a failed request or two.
- **FT**: enough servers in each AZ that losing a whole AZ doesn't slow anything down. Stateless app servers (sessions in Redis or the database, files in S3) make this possible, because any server can handle any request.
- **DR**: your Postgres backups copied to another region, plus a tested way (ideally a CloudFormation template) to bring the whole stack up there. A backup you've never restored is only a hope.

---

### Exam notes

- Match the **keywords** in the question:
  - "minimize downtime", "recover quickly", "highly available" → **HA** (Multi-AZ, Auto Scaling, load balancer)
  - "no interruption", "continue operating during failure", "zero downtime" → **FT** (active-active, extra capacity)
  - "region outage", "disaster", "restore", "RPO/RTO" → **DR** (cross-region backups and replication)
- **HA ≠ FT**: HA allows brief outages, FT doesn't.
- **RDS Multi-AZ** failover involves a short outage, so it counts as **HA**, not FT.
- FT is the **most expensive** option. If a question stresses cost-effectiveness and allows brief downtime, HA is usually the answer.
- **RPO = data loss tolerance**, **RTO = downtime tolerance**.
