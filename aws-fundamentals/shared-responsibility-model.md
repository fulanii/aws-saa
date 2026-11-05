# **Shared Responsibility Model**

[← Back to AWS Fundamentals](readme.md)

> Who is responsible for what when it comes to security on AWS: AWS secures the cloud itself, you secure what you put in it.

### Contents

- [The core idea](#the-core-idea)
- [Who does what](#who-does-what)
- [It shifts with the service type](#it-shifts-with-the-service-type)
- [Inherited, shared and customer-only controls](#inherited-shared-and-customer-only-controls)
- [Django/FastAPI map](#djangofastapi-map)
- [Exam notes](#exam-notes)

---

### The core idea

Security on AWS is split between AWS and you:

- **AWS** is responsible for security **OF** the cloud: the hardware, buildings and software that run AWS.
- **You** are responsible for security **IN** the cloud: everything you build, configure and store on top of it.

The line between the two moves depending on how much of the stack AWS manages for a given service. The more managed the service, the less you're responsible for.

---

### Who does what

```
 ┌──────────────────────────────────────────────────────────────┐
 │ CUSTOMER: security IN the cloud                              │
 │  Customer data                                               │
 │  Platform, applications, identity & access management (IAM)  │
 │  Operating system, network & firewall configuration          │
 │  Client-side encryption & data integrity                     │
 │  Server-side encryption (file system and/or data)            │
 │  Network traffic protection (encryption, integrity, identity)│
 ├──────────────────────────────────────────────────────────────┤
 │ AWS: security OF the cloud                                   │
 │  Software: compute, storage, database, networking            │
 │  Hardware / AWS global infrastructure                        │
 │  Regions, Availability Zones, edge locations                 │
 └──────────────────────────────────────────────────────────────┘
```

| AWS is responsible for | You are responsible for |
|---|---|
| Physical security of data centers | Your data, and who can access it |
| Hardware, networking gear, cabling | IAM users, roles, policies, MFA |
| The hypervisor that runs EC2 instances | The guest OS on EC2: patching, hardening |
| Regions, AZs and edge locations | Security groups, NACLs, network design |
| The software behind managed services | Encryption choices (at rest and in transit) |
| Decommissioning old disks securely | Your application code and its vulnerabilities |

---

### It shifts with the service type

| Service | Model | AWS manages | You manage |
|---|---|---|---|
| **EC2** | IaaS | Hardware, hypervisor, physical network | Guest OS and its patches, installed software, security groups, data |
| **RDS** | Managed database | Hardware, OS, database engine patching, backups infrastructure | Database users and permissions, security groups, enabling encryption, your data |
| **Lambda** | Serverless | Hardware, OS, runtime, scaling | Your function code, its IAM role, its data |
| **S3** | Managed storage | Infrastructure, durability, replication across AZs | Bucket policies, access control, encryption settings, what you upload |

The pattern: on **EC2 you patch the OS**; on **RDS and Lambda, AWS does**. On every service, **your data and IAM are always your job**.

---

### Inherited, shared and customer-only controls

- **Inherited controls** (fully AWS): physical and environmental controls. You inherit them just by using AWS.
- **Shared controls** (both, each in their own layer):
  - **Patch management**: AWS patches the infrastructure; you patch your guest OS and applications.
  - **Configuration management**: AWS configures its infrastructure; you configure your OS, databases and apps.
  - **Awareness and training**: AWS trains its staff; you train yours.
- **Customer-specific controls** (fully you): e.g. how you protect and route data within your own environments.

---

### Django/FastAPI map

- **On EC2**, running Ubuntu with Gunicorn and Nginx: OS updates, Python version upgrades, Nginx config, firewall rules (security groups) and `DEBUG=False` in production are **all on you**.
- **On RDS Postgres**: AWS patches Postgres and the OS. You still choose a strong password, keep the database in a private subnet, enable encryption and restrict the security group to your app servers.
- **On S3**: AWS keeps the files safe from hardware failure. If you make the bucket public and leak user uploads, that's **your** responsibility.
- **Always yours**: secrets in your code (`SECRET_KEY`, API keys), SQL injection and auth bugs in your app, and IAM permissions that are too broad.

---

### Exam notes

- **AWS = security OF the cloud. Customer = security IN the cloud.**
- **Guest OS patching on EC2 is the customer's job.** On RDS, Lambda and other managed services, AWS patches the OS and runtime.
- **Customer data, IAM and encryption choices are always the customer's**, on every service.
- Physical security, hardware and the hypervisor are always **AWS**'s.
- **Shared controls**: patch management, configuration management, awareness and training.
- A typical question: "Who is responsible for X?" Ask whether X is the physical/managed layer (AWS) or something you configure or put in (you).
