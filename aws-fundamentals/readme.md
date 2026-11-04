# **AWS Fundamentals**

> How AWS is laid out (regions, AZs, edge locations), how its networking is split between public and private, and the basics of VPC, EC2 and S3.

### Contents

- [AWS public vs private services](#aws-public-vs-private-services)
- [AWS global infrastructure](#aws-global-infrastructure)
- [Resilience levels](#resilience-levels)
- [VPC (Virtual Private Cloud) basics](#vpc-virtual-private-cloud-basics)
- [Default VPC facts](#default-vpc-facts)
- [EC2 (Elastic Compute Cloud) basics](#ec2-elastic-compute-cloud-basics)
  - [Instance lifecycle](#instance-lifecycle)
  - [AMI (Amazon Machine Image)](#ami-amazon-machine-image)
  - [Connecting to an instance](#connecting-to-an-instance)
- [S3 (Simple Storage Service) basics](#s3-simple-storage-service-basics)
  - [Objects](#objects)
  - [Buckets](#buckets)
  - [Bucket naming rules and limits](#bucket-naming-rules-and-limits)
  - [Patterns and anti-patterns](#patterns-and-anti-patterns)
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

### EC2 (Elastic Compute Cloud) basics

EC2 gives you **virtual machines**, called **instances**. It's **IaaS** (Infrastructure as a Service): AWS runs the physical hardware and the virtualization, and you manage everything from the operating system up.

**Key facts:**

- **Private service by default**: an instance is launched into a subnet in a VPC and uses VPC networking. It's only reachable from the internet if that VPC and subnet allow it.
- **AZ resilient**: an instance runs on one physical host in one AZ. If that AZ fails, the instance fails with it.
- **Many sizes and capabilities**: instance types trade off CPU, memory, storage and networking (e.g. `t3.micro` for small and cheap, larger types for heavy workloads).
- **On-demand billing, per second**: most Linux and Windows instances are billed per second, with a 60-second minimum.
- **Two kinds of storage**:
  - **Instance store**: local disk on the physical host. Fast, but **temporary**: the data is lost when the instance stops or the host fails.
  - **EBS (Elastic Block Store)**: network-attached volumes that persist independently of the instance. Most root volumes are EBS.

#### Instance lifecycle

The three states that matter most are **running**, **stopped** and **terminated**. Between them are short transition states (`pending`, `stopping`, `shutting-down`).

```
 pending ──> running ──> stopping ──> stopped
                ^                        │
                └────────── start ───────┘

 running or stopped ──> shutting-down ──> terminated   (one way, no going back)
```

| State | What you pay for | Notes |
|---|---|---|
| **Running** | CPU, memory, storage, networking | The instance is up and doing work. |
| **Stopped** | **EBS storage only** | No compute charges, but the EBS volumes (OS, app, data) still exist and still cost money. Instance store data is lost. |
| **Terminated** | Nothing | The instance is deleted. **Not reversible.** By default the root EBS volume is deleted too. |

- Running ⇄ stopped can go back and forth as many times as you like.
- Terminated is final. You can't start a terminated instance.

#### AMI (Amazon Machine Image)

An AMI is a **template for an instance**, like a server image or a Docker image for a whole VM.

- You **launch** EC2 instances **from** an AMI.
- You can **create** an AMI **from** an existing EC2 instance (bake a configured server into a reusable image).
- AMIs exist for different operating systems: Amazon Linux, Ubuntu, Red Hat, Windows, macOS and more.
- AMIs are **regional**. To use one in another region, copy it there.

**AMI permissions** (who can launch from it):

- **Public**: any AWS account can use it.
- **Owner**: the account that created it. Owners have implicit full access.
- **Explicit**: the owner grants access to specific other AWS accounts.

**What's inside an AMI:**

- **Root volume**: the volume the instance boots from, holding the operating system. For EBS-backed AMIs this is a reference to an EBS snapshot, which becomes the new instance's root volume.
- **Block device mapping**: the list of volumes to attach at launch and the device name each one gets (e.g. root on `/dev/xvda`, data on `/dev/sdf`). It's how the instance knows which volume is the boot volume and which are data volumes.

#### Connecting to an instance

| OS | Protocol | Port | How you authenticate |
|---|---|---|---|
| Linux | SSH | 22 | **Key pair**: AWS stores the public key, you keep the private key (`.pem`) |
| Windows | RDP | 3389 | The key pair's private key is used to decrypt the local Administrator password |

- **Key pairs are regional**, like AMIs. You can import your own public key (e.g. `~/.ssh/id_ed25519.pub`) instead of letting AWS generate one.
- If AWS generates the key, the private key is downloaded **once**. Lose it and you can't download it again.
- SSH refuses keys with loose permissions: run `chmod 400 my-key.pem` first.
- The instance's security group must allow inbound traffic on port 22 (or 3389) from your IP.

```bash
ssh -i ~/.ssh/my-key.pem ec2-user@<public-ip>   # Amazon Linux; Ubuntu uses "ubuntu@"
```

---

### S3 (Simple Storage Service) basics

S3 is AWS's **object storage**: you put files in, you get files out, over HTTP. It's the default answer for "where do I store this data" on AWS.

**Key facts:**

- **Global namespace, regional storage**: S3 is available from every region, and bucket names are unique across all of AWS. But each bucket, and the data in it, lives in **one region** and never leaves that region unless you configure it to (e.g. replication).
- **Public service**: reached over a public endpoint, from anywhere on the internet (see [public vs private services](#aws-public-vs-private-services)). Reachable doesn't mean open: buckets are **private by default**.
- **Regionally resilient**: data is replicated across multiple AZs in the bucket's region, so losing an AZ loses nothing.
- **Unlimited storage**, shared by many users.
- Holds any kind of data: movies, audio, photos, text, large datasets.
- **Cheap**, and accessed via the console, CLI, SDKs/API or plain HTTP.
- Its two main building blocks are **buckets** and **objects**.

#### Objects

An object is basically a **file**. It has two main parts:

- **Key**: the object's name, e.g. `pic.png` or `old/koala1.jpg`. The key identifies the object within its bucket.
- **Value**: the content (the actual bytes of the image). Anywhere from **0 bytes to 5 TB**.

Objects also carry a **version ID**, **metadata**, **access control** and **subresources**.

#### Buckets

A bucket is a **container for objects**.

- Created in a **specific region**, which is the data's home. Data stays there unless you configure otherwise.
- Its name must be **globally unique**: across every AWS account in the world, not just yours.
- Holds an **unlimited** number of objects.
- **Flat structure**: there are no real folders. Everything is stored at the same level.

```
koala1.jpg
koala2.jpg
old/koala1.jpg     <- not a file inside an "old" folder
old/koala2.jpg        just an object whose key starts with "old/"
old/koala3.jpg
```

The console *shows* `old/` as a folder for convenience, but it's only a shared **prefix** in the key names.

#### Bucket naming rules and limits

- Globally unique.
- **3 to 63 characters**, all **lowercase**, **no underscores** (letters, numbers, hyphens and dots).
- Must **start and end with a lowercase letter or number**.
- Can't look like an IP address (e.g. `1.1.1.1`).
- **Buckets per account**: the course (and older exam material) teaches **100 soft limit, 1,000 hard limit**. AWS raised the default quota to 10,000 in late 2024, but expect the exam to use the 100 / 1,000 numbers.
- **Objects per bucket**: unlimited, each from 0 bytes to 5 TB.
- A single upload (PUT) maxes out at **5 GB**. Anything bigger needs **multipart upload**, which AWS recommends for anything over about 100 MB.

#### Patterns and anti-patterns

S3 is an **object store**: not a file system and not block storage.

| Use S3 for | Don't use S3 for |
|---|---|
| Large-scale data storage, distribution and upload | A disk to **mount** on an instance (use **EBS** for block storage) |
| **Offloading** static files and user uploads from your servers | A shared file system mounted by many servers (use **EFS**) |
| Input to and output from many AWS services (Lambda, Athena, CloudFront, ...) | Booting an OS or running a database on it |

---

### Django/FastAPI map

- **Region**: where your app is hosted. Pick one close to your users, or one that meets your data-residency requirements.
- **Multiple AZs**: like running your Django/FastAPI servers in two or more data centers behind a load balancer, so one going down doesn't take the site with it.
- **VPC**: your private network. The API servers can sit in a public-facing setup, but Postgres belongs in a private subnet that the internet can't reach. Your `DATABASES['default']['HOST']` points at a private address.
- **Default VPC**: fine for a quick test. For anything real, build a custom VPC so you control what's public.
- **EC2 instance**: the Linux box that runs Gunicorn/Uvicorn, the same thing as a VPS (DigitalOcean droplet, Linode), but inside your VPC.
- **AMI**: a golden server image with Python, your dependencies and your systemd service already installed, so new instances boot ready to serve instead of being set up by hand each time.
- **Stopped vs terminated**: stopping is like shutting down a server you'll restart later (disk kept, still billed for it). Terminating is deleting the server.
- **S3**: where `MEDIA_ROOT` and `STATIC_ROOT` go in production, via `django-storages` (`S3Boto3Storage`) or `boto3` in FastAPI. Your app servers stay stateless, so you can run several behind a load balancer.
- **Presigned URLs**: let users upload to and download from a private bucket directly, without the bytes going through your API.

---

### Exam notes

- "Public vs private service" is about **network reachability**, not permissions.
- Know the resilience level of common services: IAM (global), S3 (regional), EC2 (AZ).
- Subnets are AZ-scoped. Surviving an AZ failure means running resources in subnets in **2+ AZs**.
- Default VPC: `172.31.0.0/16`, one `/20` per AZ, public IPs on by default.
- If an answer relies on the default VPC for a production workload, it's probably a distractor.
- EC2 is **IaaS** and **AZ resilient**. One instance in one AZ is never "highly available".
- A stopped instance still costs money for its **EBS storage**. Only terminating stops all charges.
- **Instance store** data is lost on stop or host failure. Persistent data belongs on **EBS** (or S3, EFS).
- AMIs and key pairs are **regional**. Launching in another region means copying the AMI and importing or creating a key pair there.
- AMI permissions: **public**, **owner** (implicit) or **explicit** (specific accounts).
- S3 bucket names are **global**, but the data is **regional** and **regionally resilient**.
- Objects: **0 bytes to 5 TB**. Single PUT: **5 GB max**, so anything larger means **multipart upload**.
- S3 has **no real folders**: "folders" are just key **prefixes**.
- "Needs to be mounted as a drive" or "needs a file system" means **EBS** or **EFS**, not S3.
- Buckets are **private by default**. Public access has to be granted deliberately.
