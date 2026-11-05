# **AWS Global Infrastructure & Networking Zones**

[← Back to AWS Fundamentals](readme.md)

> How AWS is laid out physically (regions, AZs, edge locations), how its networking is split between public and private, and how much failure a service can survive.

### Contents

- [AWS public vs private services](#aws-public-vs-private-services)
- [AWS global infrastructure](#aws-global-infrastructure)
- [Resilience levels](#resilience-levels)
- [Django/FastAPI map](#djangofastapi-map)
- [Exam notes](#exam-notes)

---

### AWS public vs private services

"Public" and "private" here describe **networking only**: where a service can be reached from. They say nothing about permissions. A public service like S3 is still locked down by default. Anyone can reach its endpoint, but nobody gets in without permission.

AWS networking is easiest to picture as three zones:

- **Public internet zone**: the internet, where your users are.
- **AWS public zone**: an AWS network attached to the public internet. Public services with public endpoints live here, e.g. S3.
- **AWS private zone**: where VPCs live. Isolated by default.

```
 Public internet  <---->  AWS public zone  <---->  AWS private zone
   (your users)            (S3, ...)                (VPCs: EC2, RDS, ...)
                                            only if you configure it
                                            (e.g. an internet gateway)
```

- A **public service** is reachable through a public endpoint (e.g. S3).
- A **private service** runs inside a VPC (e.g. an EC2 instance) and can only be reached the ways that VPC allows.
- VPCs are isolated unless you configure them otherwise.

---

### AWS global infrastructure

**Region**: an area of the world with a full deployment of AWS infrastructure (compute, storage, databases, networking). Each region has a code, e.g. `us-east-1` (N. Virginia) or `ap-southeast-2` (Sydney).

Regions give you three things:

- **Geographic separation**: fault isolation. A natural disaster, outage or war in one region doesn't affect another.
- **Geopolitical separation**: each region follows the laws of where it is. Your data stays in that region unless you move it.
- **Location control**: run your systems close to your customers for better performance, or spread them across regions for resilience.

Resources in different regions are separate. An S3 bucket created in Virginia is not the same as one in Sydney. Most services are **regional** (EC2, VPC, RDS). A few are **global** (IAM, Route 53, CloudFront).

**Availability Zone (AZ)**: every region has several AZs, each with its own isolated compute, storage, networking, power and facilities. AZ names are the region code plus a letter: `us-east-2a`, `us-east-2b`, ... If one AZ fails (fire, power loss), the other AZs in that region keep running.

**Edge location**: much smaller than a region, but there are many more of them. They mostly do content delivery (CloudFront) and some edge computing. The point is low latency, because there's one close to almost every user.

---

### Resilience levels

How much failure a service can survive. This shows up constantly in exam questions.

| Level | Survives | Examples |
|---|---|---|
| **Globally resilient** | a whole region failing (data is replicated across regions) | IAM, Route 53 |
| **Regionally resilient** | an AZ failing (data is replicated across AZs in one region) | S3, VPC |
| **AZ resilient** | nothing beyond its AZ: if the AZ fails, it fails | EC2 instance, EBS volume, subnet |

---

### Django/FastAPI map

- **Region**: where your app is hosted. Pick one close to your users, or one that meets your data-residency requirements.
- **Multiple AZs**: like running your Django/FastAPI servers in two or more data centers behind a load balancer, so one going down doesn't take the site with it.

---

### Exam notes

- "Public vs private service" is about **network reachability**, not permissions.
- Know the resilience level of common services: IAM (global), S3 (regional), EC2 (AZ).
- "Survive an AZ failure" means **2+ AZs**. "Survive a region failure" means **multi-region**.
