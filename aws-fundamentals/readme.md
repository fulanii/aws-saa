# **AWS Fundamentals**

> How AWS is laid out (regions, AZs, edge locations), how its networking is split between public and private, and the basics of a VPC.

### Contents

- [AWS public vs private services](#aws-public-vs-private-services)
- [AWS global infrastructure](#aws-global-infrastructure)
- [Resilience levels](#resilience-levels)
- [VPC (Virtual Private Cloud) basics](#vpc-virtual-private-cloud-basics)
- [Default VPC facts](#default-vpc-facts)
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

### VPC (Virtual Private Cloud) basics

A **VPC** is a private network inside **one AWS account and one region**. It spans every AZ in that region, so it's **regionally resilient**.

- Private and isolated by default. Unless you configure it, a VPC can't talk to the internet, your on-prem network or even other VPCs.
- There are two types:
  - **Default VPC**: at most **one per region**, set up by AWS.
  - **Custom VPC**: as many as you want per region. You configure everything end to end, and they're fully private by default.

**VPC CIDR**: the IP range the VPC owns, from start address to end address. Everything inside the VPC gets its IP from this range.

- A custom VPC can have **multiple** CIDR ranges.
- The default VPC has exactly **one**, always `172.31.0.0/16`.

**Subnets** (subnetworks) are how a VPC gets resilience:

- Each subnet is a slice of the VPC's CIDR range.
- Each subnet lives in **exactly one AZ**. You choose it when you create the subnet and it can never change.
- Subnet ranges can't overlap with any other subnet in the same VPC.
- A resource in a subnet gets its IP from that subnet's range.
- Put subnets (and your resources) in several AZs, and an AZ failure only takes out part of your system.

Example: the default VPC in `us-east-2` (`172.31.0.0/16`), one `/20` subnet per AZ:

| AZ | Subnet CIDR | Address range | Addresses |
|---|---|---|---|
| `us-east-2a` | `172.31.0.0/20` | `172.31.0.0` to `172.31.15.255` | 4,096 |
| `us-east-2b` | `172.31.16.0/20` | `172.31.16.0` to `172.31.31.255` | 4,096 |
| `us-east-2c` | `172.31.32.0/20` | `172.31.32.0` to `172.31.47.255` | 4,096 |

---

### Default VPC facts

- One per region. It can be deleted and recreated.
- CIDR is always `172.31.0.0/16`.
- One `/20` subnet in every AZ in the region.
- Comes preconfigured with an **Internet Gateway (IGW)**, a **Security Group (SG)** and a **NACL**.
- Its subnets assign **public IPv4 addresses** automatically, so anything launched there is exposed to the internet by default.
- **Best practice: don't use the default VPC for real workloads.** Build a custom VPC instead. Some services expect a default VPC to exist, so it's usually left in place rather than deleted.

---

### Django/FastAPI map

- **Region**: where your app is hosted. Pick one close to your users, or one that meets your data-residency requirements.
- **Multiple AZs**: like running your Django/FastAPI servers in two or more data centers behind a load balancer, so one going down doesn't take the site with it.
- **VPC**: your private network. The API servers can sit in a public-facing setup, but Postgres belongs in a private subnet that the internet can't reach. Your `DATABASES['default']['HOST']` points at a private address.
- **Default VPC**: fine for a quick test. For anything real, build a custom VPC so you control what's public.

---

### Exam notes

- "Public vs private service" is about **network reachability**, not permissions.
- Know the resilience level of common services: IAM (global), S3 (regional), EC2 (AZ).
- Subnets are AZ-scoped. Surviving an AZ failure means running resources in subnets in **2+ AZs**.
- Default VPC: `172.31.0.0/16`, one `/20` per AZ, public IPs on by default.
- If an answer relies on the default VPC for a production workload, it's probably a distractor.
