# **VPC (Virtual Private Cloud) Basics**

[← Back to AWS Fundamentals](readme.md)

> Your private network inside AWS: what a VPC is, how CIDR ranges and subnets work, and why the default VPC isn't for real workloads.

### Contents

- [What a VPC is](#what-a-vpc-is)
- [VPC CIDR](#vpc-cidr)
- [Subnets](#subnets)
- [Default VPC facts](#default-vpc-facts)
- [Django/FastAPI map](#djangofastapi-map)
- [Exam notes](#exam-notes)

---

### What a VPC is

A **VPC** is a private network inside **one AWS account and one region**. It spans every AZ in that region, so it's **regionally resilient**.

- Private and isolated by default. Unless you configure it, a VPC can't talk to the internet, your on-prem network or even other VPCs.
- There are two types:
  - **Default VPC**: at most **one per region**, set up by AWS.
  - **Custom VPC**: as many as you want per region. You configure everything end to end, and they're fully private by default.

---

### VPC CIDR

The IP range the VPC owns, from start address to end address. Everything inside the VPC gets its IP from this range.

- A custom VPC can have **multiple** CIDR ranges.
- The default VPC has exactly **one**, always `172.31.0.0/16`.

---

### Subnets

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

- **VPC**: your private network. The API servers can sit in a public-facing setup, but Postgres belongs in a private subnet that the internet can't reach. Your `DATABASES['default']['HOST']` points at a private address.
- **Default VPC**: fine for a quick test. For anything real, build a custom VPC so you control what's public.

---

### Exam notes

- A VPC is **regional**; a subnet is **AZ-scoped**. Surviving an AZ failure means running resources in subnets in **2+ AZs**.
- Default VPC: `172.31.0.0/16`, one `/20` per AZ, public IPs on by default.
- If an answer relies on the default VPC for a production workload, it's probably a distractor.
