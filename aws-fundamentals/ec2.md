# **EC2 (Elastic Compute Cloud) Basics**

[← Back to AWS Fundamentals](readme.md)

> Virtual machines on AWS: key facts, the instance lifecycle and what each state costs, AMIs, and how to connect to an instance.

### Contents

- [Key facts](#key-facts)
- [Instance lifecycle](#instance-lifecycle)
- [AMI (Amazon Machine Image)](#ami-amazon-machine-image)
- [Connecting to an instance](#connecting-to-an-instance)
- [Django/FastAPI map](#djangofastapi-map)
- [Exam notes](#exam-notes)

---

### Key facts

EC2 gives you **virtual machines**, called **instances**. It's **IaaS** (Infrastructure as a Service): AWS runs the physical hardware and the virtualization, and you manage everything from the operating system up.

- **Private service by default**: an instance is launched into a subnet in a [VPC](vpc.md) and uses VPC networking. It's only reachable from the internet if that VPC and subnet allow it.
- **AZ resilient**: an instance runs on one physical host in one AZ. If that AZ fails, the instance fails with it.
- **Many sizes and capabilities**: instance types trade off CPU, memory, storage and networking (e.g. `t3.micro` for small and cheap, larger types for heavy workloads).
- **On-demand billing, per second**: most Linux and Windows instances are billed per second, with a 60-second minimum.
- **Two kinds of storage**:
  - **Instance store**: local disk on the physical host. Fast, but **temporary**: the data is lost when the instance stops or the host fails.
  - **EBS (Elastic Block Store)**: network-attached volumes that persist independently of the instance. Most root volumes are EBS.

---

### Instance lifecycle

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

---

### AMI (Amazon Machine Image)

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

---

### Connecting to an instance

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

### Django/FastAPI map

- **EC2 instance**: the Linux box that runs Gunicorn/Uvicorn, the same thing as a VPS (DigitalOcean droplet, Linode), but inside your VPC.
- **AMI**: a golden server image with Python, your dependencies and your systemd service already installed, so new instances boot ready to serve instead of being set up by hand each time.
- **Stopped vs terminated**: stopping is like shutting down a server you'll restart later (disk kept, still billed for it). Terminating is deleting the server.

---

### Exam notes

- EC2 is **IaaS** and **AZ resilient**. One instance in one AZ is never "highly available".
- A stopped instance still costs money for its **EBS storage**. Only terminating stops all charges.
- **Instance store** data is lost on stop or host failure. Persistent data belongs on **EBS** (or S3, EFS).
- AMIs and key pairs are **regional**. Launching in another region means copying the AMI and importing or creating a key pair there.
- AMI permissions: **public**, **owner** (implicit) or **explicit** (specific accounts).
