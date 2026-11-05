# **AWS Fundamentals**

[← Back to main readme](../readme.md)

> How AWS is laid out, how its networking is split between public and private, and the basics of the core services. One file per service; general concepts are grouped together.

### Contents

| # | Notes | What's inside |
|---|---|---|
| 1 | [**Global Infrastructure & Networking Zones**](global-infrastructure.md) | Public vs private services, regions, AZs, edge locations, resilience levels |
| 2 | [**VPC (Virtual Private Cloud)**](vpc.md) | What a VPC is, CIDR, subnets, default VPC facts |
| 3 | [**EC2 (Elastic Compute Cloud)**](ec2.md) | Key facts, instance lifecycle, AMIs, connecting via SSH/RDP |
| 4 | [**S3 (Simple Storage Service)**](s3.md) | Objects, buckets, naming rules and limits, patterns and anti-patterns |
| 5 | [**CloudFormation**](cloudformation.md) | Templates, template sections, logical vs physical resources, stacks |
| 6 | [**CloudWatch**](cloudwatch.md) | Metrics, logs, events, namespaces, datapoints, dimensions, alarms |
| 7 | [**Shared Responsibility Model**](shared-responsibility-model.md) | Security OF vs IN the cloud, how it shifts per service, shared controls |
| 8 | [**HA vs Fault Tolerance vs Disaster Recovery**](ha-ft-dr.md) | Definitions, uptime "nines", RPO/RTO, AWS examples, exam keywords |
| 9 | [**Route 53**](route53.md) | Domain registration, delegation, hosted zones (public/private) |
| 10 | [**DNS Record Types**](dns-record-types.md) | NS, A/AAAA, CNAME, MX, TXT, SOA, TTL and caching |

Each file has its own contents list at the top, plus a **Django/FastAPI map** and **Exam notes** at the bottom.
