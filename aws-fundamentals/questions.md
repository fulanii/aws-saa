# **AWS Fundamentals: Practice Questions**

[← Back to AWS Fundamentals](readme.md)

> Test yourself on the AWS Fundamentals section. Pick an answer for each question, then click **Show answer** to check it. Questions marked *(Choose 2)* or *(Choose 3)* have more than one correct option.

### Contents

- [Cloud models & global infrastructure](#cloud-models--global-infrastructure) (Q1 to Q8)
- [VPC](#vpc) (Q9 to Q13)
- [EC2](#ec2) (Q14 to Q19)
- [S3](#s3) (Q20 to Q25)
- [CloudFormation](#cloudformation) (Q26 to Q31)
- [CloudWatch](#cloudwatch) (Q32 to Q35)
- [Shared Responsibility Model](#shared-responsibility-model) (Q36 to Q37)
- [HA vs Fault Tolerance vs Disaster Recovery](#ha-vs-fault-tolerance-vs-disaster-recovery) (Q38 to Q41)
- [DNS](#dns) (Q42 to Q51)
- [Route 53](#route-53) (Q52 to Q54)

---

### Cloud models & global infrastructure

**1. EC2 is an example of which service model?**

- A. PaaS
- B. SaaS
- C. IaaS
- D. DBaaS
- E. FaaS

<details>
<summary>Show answer</summary>

<br>

**C. IaaS**

- AWS manages the hardware and virtualization; you manage everything from the operating system up.
- PaaS is a full platform (e.g. Heroku), DBaaS is a managed database (e.g. RDS), FaaS is functions (e.g. Lambda), SaaS is finished software (e.g. Gmail).

[Notes: Cloud computing models](models.md)
</details>

**2. Which service model does AWS Lambda belong to?**

- A. IaaS
- B. FaaS
- C. SaaS
- D. DBaaS

<details>
<summary>Show answer</summary>

<br>

**B. FaaS**

- You only provide the function code and its triggers. AWS manages the servers, OS and runtime, and runs your code when an event fires.

[Notes: Cloud computing models](models.md)
</details>

**3. What is true of an AWS Public Service?** *(Choose 2)*

- A. Located in the public internet zone
- B. Publicly accessible by anyone
- C. Located in the AWS Public Zone
- D. Located in a VPC
- E. Anyone can connect, but permissions are required to access the service

<details>
<summary>Show answer</summary>

<br>

**C and E**

- Public services (e.g. S3) live in the **AWS Public Zone**, a network attached to the internet, not in the public internet zone itself (A) or a VPC (D).
- "Public" is about **network reachability**, not permissions: anyone can reach the endpoint, but nothing is accessible without permission (so B is wrong).

[Notes: Public vs private services](global-infrastructure.md#aws-public-vs-private-services)
</details>

**4. What is true of an AWS Private Service?** *(Choose 3)*

- A. Accessible from any other VPC
- B. Located in a VPC
- C. Located on the public internet
- D. Accessible from other VPCs or on-premises networks as long as private networking is configured
- E. Located in the AWS Public Zone
- F. Accessible from the VPC it is located in

<details>
<summary>Show answer</summary>

<br>

**B, D and F**

- Private services (e.g. EC2) run **inside a VPC** and are reachable from that VPC.
- VPCs are isolated by default, so other VPCs or on-prem networks can only reach them once private networking (peering, VPN, ...) is configured. That's why A is wrong and D is right.

[Notes: Public vs private services](global-infrastructure.md#aws-public-vs-private-services)
</details>

**5. Which benefits does using multiple AWS Regions provide?** *(Choose 3)*

- A. Geographic separation
- B. Unlimited capacity in each region
- C. Geopolitical separation
- D. Automatic replication of all your data between regions
- E. Location control

<details>
<summary>Show answer</summary>

<br>

**A, C and E**

- **Geographic separation**: a disaster in one region doesn't affect another.
- **Geopolitical separation**: each region follows local laws.
- **Location control**: put systems close to customers.
- Data does **not** move between regions unless you configure it (D).

[Notes: AWS global infrastructure](global-infrastructure.md#aws-global-infrastructure)
</details>

**6. What is the resilience level of a single EC2 instance?**

- A. Globally resilient
- B. Regionally resilient
- C. AZ resilient
- D. Edge resilient

<details>
<summary>Show answer</summary>

<br>

**C. AZ resilient**

- An instance runs on one host in one AZ. If that AZ fails, the instance fails with it.

[Notes: Resilience levels](global-infrastructure.md#resilience-levels)
</details>

**7. Which of these services is globally resilient?**

- A. S3
- B. VPC
- C. IAM
- D. EBS

<details>
<summary>Show answer</summary>

<br>

**C. IAM**

- IAM data is replicated across all regions. S3 and VPC are **regionally** resilient; EBS is **AZ** resilient.

[Notes: Resilience levels](global-infrastructure.md#resilience-levels)
</details>

**8. What are AWS edge locations mainly used for?**

- A. Running full workloads such as RDS databases
- B. Content distribution and some edge computing
- C. Hosting VPCs closer to customers
- D. Storing disaster recovery backups

<details>
<summary>Show answer</summary>

<br>

**B. Content distribution and some edge computing**

- Edge locations are small sites (much smaller than regions) used mainly by CloudFront to cache content close to users.

[Notes: AWS global infrastructure](global-infrastructure.md#aws-global-infrastructure)
</details>

---

### VPC

**9. How many subnets are in a default VPC?**

- A. 2
- B. Equal to the number of AZs in the region the VPC is located in
- C. 3
- D. 10

<details>
<summary>Show answer</summary>

<br>

**B. Equal to the number of AZs in the region**

- The default VPC always has one `/20` subnet in every AZ of its region, so the count depends on the region.

[Notes: Default VPC facts](vpc.md#default-vpc-facts)
</details>

**10. What is the IP CIDR of a default VPC?**

- A. It depends on the region
- B. `10.0.0.0/16`
- C. `172.31.0.0/16`
- D. Random, based on the AWS account
- E. You can configure an IP range suitable for your network

<details>
<summary>Show answer</summary>

<br>

**C. `172.31.0.0/16`**

- Every default VPC, in every region and account, uses `172.31.0.0/16`. You can only choose the CIDR for **custom** VPCs (E).

[Notes: VPC CIDR](vpc.md#vpc-cidr)
</details>

**11. How many default VPCs can a region have?**

- A. Unlimited
- B. Up to 5
- C. At most one
- D. One per AZ

<details>
<summary>Show answer</summary>

<br>

**C. At most one**

- One default VPC per region. It can be deleted and recreated. Custom VPCs are the ones you can have many of.

[Notes: What a VPC is](vpc.md#what-a-vpc-is)
</details>

**12. Where is a VPC subnet located?**

- A. Across all AZs in the region
- B. In exactly one AZ, which can be changed later
- C. In exactly one AZ, chosen at creation and never changed
- D. Across multiple regions

<details>
<summary>Show answer</summary>

<br>

**C. In exactly one AZ, chosen at creation and never changed**

- The VPC spans the region, but each subnet is tied to one AZ. That's why high availability means using subnets in **2+ AZs**.

[Notes: Subnets](vpc.md#subnets)
</details>

**13. Why should the default VPC be avoided for production workloads?**

- A. It's slower than a custom VPC
- B. It costs more than a custom VPC
- C. Its subnets assign public IPv4 addresses by default and its CIDR can't be chosen
- D. EC2 instances can't be launched into it

<details>
<summary>Show answer</summary>

<br>

**C**

- Anything launched there gets a public IP and is internet-exposed by default, and you're stuck with `172.31.0.0/16`. A custom VPC lets you design public and private subnets and pick a CIDR that won't clash with other networks.

[Notes: Default VPC facts](vpc.md#default-vpc-facts)
</details>

---

### EC2

**14. What permission options does an AMI have?**

- A. Public Access, Owner only, Specific IAM users
- B. Public Access, Owner only, Specific AWS Accounts
- C. Public Access, Owner only, Specific Regions
- D. Public Access, Specific AWS Accounts, Specific IAM users

<details>
<summary>Show answer</summary>

<br>

**B. Public Access, Owner only, Specific AWS Accounts**

- **Public**: any AWS account can launch it. **Owner**: only the account that created it. **Explicit**: specific other AWS accounts.
- AMIs are shared with **accounts**, not individual IAM users, which rules out A and D.
- AMIs are **regional**: to use one in another region you copy it there. There's no "share with a region" permission, which rules out C.

[Notes: AMI permissions](ec2.md#ami-amazon-machine-image)
</details>

**15. What is NOT stored in an AMI?** *(Choose 2)*

- A. Boot volume
- B. Instance settings
- C. AMI permissions
- D. Block device mapping
- E. Network settings
- F. Data volumes

<details>
<summary>Show answer</summary>

<br>

**B and E**

- An AMI contains the **boot volume** and **data volumes** (as snapshots), its **permissions**, and the **block device mapping**.
- Instance settings (e.g. instance type) and network settings (subnet, security group, IP) are chosen **when you launch** an instance, not stored in the image.

[Notes: AMI](ec2.md#ami-amazon-machine-image)
</details>

**16. What do you pay for while an EC2 instance is stopped?**

- A. Nothing
- B. Compute only
- C. EBS storage only
- D. The same as when it's running

<details>
<summary>Show answer</summary>

<br>

**C. EBS storage only**

- Compute charges stop, but the EBS volumes still exist and still cost money. Only **terminating** stops all charges.

[Notes: Instance lifecycle](ec2.md#instance-lifecycle)
</details>

**17. What happens to data on an instance store volume when the instance is stopped?**

- A. It persists until the instance is terminated
- B. It's lost
- C. It's automatically moved to EBS
- D. It's automatically snapshotted to S3

<details>
<summary>Show answer</summary>

<br>

**B. It's lost**

- Instance store is temporary disk on the physical host. Data that must survive belongs on **EBS** (or S3/EFS).

[Notes: EC2 key facts](ec2.md#key-facts)
</details>

**18. You have a custom AMI in `us-east-1` and need to launch it in `eu-west-1`. What do you do?**

- A. Share the AMI with the `eu-west-1` region
- B. Copy the AMI to `eu-west-1`
- C. Nothing: AMIs are global, so launch it directly
- D. It's not possible; rebuild the server by hand

<details>
<summary>Show answer</summary>

<br>

**B. Copy the AMI to `eu-west-1`**

- AMIs are **regional**. Copying creates a new AMI in the target region. (Key pairs are regional too.)

[Notes: AMI](ec2.md#ami-amazon-machine-image)
</details>

**19. How do you connect to a Linux EC2 instance by default?**

- A. RDP on port 3389
- B. SSH on port 22
- C. HTTPS on port 443
- D. SSH on port 3389

<details>
<summary>Show answer</summary>

<br>

**B. SSH on port 22**

- Linux uses **SSH (22)** with a key pair; Windows uses **RDP (3389)**. The security group must allow the port from your IP.

[Notes: Connecting to an instance](ec2.md#connecting-to-an-instance)
</details>

---

### S3

**20. What is true of Simple Storage Service (S3)?** *(Choose 3)*

- A. S3 is an AWS Public Service
- B. S3 is a private service
- C. S3 is a web scale block storage system
- D. S3 is an object storage system
- E. Buckets can store a limit of 100 TB of data
- F. Buckets can store an unlimited amount of data

<details>
<summary>Show answer</summary>

<br>

**A, D and F**

- S3 is a **public service** (public endpoint, private by default) and an **object** store, not block storage (that's EBS).
- Buckets hold unlimited data and objects, each object 0 bytes to 5 TB.

[Notes: S3 key facts](s3.md#key-facts)
</details>

**21. What is the maximum size of a single S3 object?**

- A. 5 GB
- B. 5 TB
- C. 100 TB
- D. Unlimited

<details>
<summary>Show answer</summary>

<br>

**B. 5 TB**

- Don't mix it up with **5 GB**, the maximum for a single PUT upload.

[Notes: Bucket naming rules and limits](s3.md#bucket-naming-rules-and-limits)
</details>

**22. You need to upload a 20 GB file to S3. How?**

- A. A single PUT request
- B. Multipart upload
- C. You can't, the file is too large
- D. Split it across several buckets

<details>
<summary>Show answer</summary>

<br>

**B. Multipart upload**

- A single PUT tops out at 5 GB. Multipart upload is required above that, and recommended above about 100 MB.

[Notes: Bucket naming rules and limits](s3.md#bucket-naming-rules-and-limits)
</details>

**23. S3 bucket names must be unique...**

- A. Within your AWS account
- B. Within the region
- C. Globally, across all AWS accounts
- D. Within your AWS Organization

<details>
<summary>Show answer</summary>

<br>

**C. Globally, across all AWS accounts**

- The bucket's data is regional, but its **name** is global.

[Notes: Buckets](s3.md#buckets)
</details>

**24. In a bucket, what is `old/koala1.jpg`?**

- A. A file called `koala1.jpg` inside a folder called `old`
- B. An object whose key starts with the prefix `old/`
- C. An object in a bucket called `old`
- D. A link to `koala1.jpg`

<details>
<summary>Show answer</summary>

<br>

**B. An object whose key starts with the prefix `old/`**

- S3 is flat: there are no real folders. The console just displays shared prefixes as folders.

[Notes: Buckets](s3.md#buckets)
</details>

**25. Several EC2 instances need a shared file system they can all mount. Which service fits?**

- A. S3
- B. EBS
- C. EFS
- D. Instance store

<details>
<summary>Show answer</summary>

<br>

**C. EFS**

- S3 is object storage and can't be mounted as a file system. EBS and instance store are block storage, normally attached to a single instance.

[Notes: Patterns and anti-patterns](s3.md#patterns-and-anti-patterns)
</details>

---

### CloudFormation

**26. What is a CloudFormation logical resource?**

- A. A resource in a stack which hasn't been created yet
- B. A resource defined in a CloudFormation template
- C. A resource created in an AWS account by CloudFormation
- D. A name given to a resource created with best practice configuration

<details>
<summary>Show answer</summary>

<br>

**B. A resource defined in a CloudFormation template**

- e.g. `Instance` with `Type: AWS::EC2::Instance`. The real thing it creates in your account is the **physical** resource (C).

[Notes: Stacks, logical and physical resources](cloudformation.md#stacks-logical-and-physical-resources)
</details>

**27. What is a CloudFormation physical resource?**

- A. A resource defined in a CloudFormation template, e.g. `EC2Instance`
- B. A product in AWS which is a physical piece of hardware, e.g. a router
- C. A resource created in your account by creating a CloudFormation stack
- D. None of the above

<details>
<summary>Show answer</summary>

<br>

**C. A resource created by creating a CloudFormation stack**

- Each logical resource in the stack gets a matching physical resource (e.g. a real instance `i-0abc123...`), and CloudFormation keeps them in sync.

[Notes: Stacks, logical and physical resources](cloudformation.md#stacks-logical-and-physical-resources)
</details>

**28. Which is the only mandatory section of a CloudFormation template?**

- A. Parameters
- B. Resources
- C. Description
- D. Outputs

<details>
<summary>Show answer</summary>

<br>

**B. Resources**

[Notes: Template anatomy](cloudformation.md#template-anatomy)
</details>

**29. Which template section prompts the user for input when creating a stack?**

- A. Mappings
- B. Conditions
- C. Parameters
- D. Metadata

<details>
<summary>Show answer</summary>

<br>

**C. Parameters**

- Mappings are lookup tables, Conditions decide whether resources are created, and Metadata controls how the console displays things.

[Notes: Template anatomy](cloudformation.md#template-anatomy)
</details>

**30. If a template uses both, which section must directly follow `AWSTemplateFormatVersion`?**

- A. Resources
- B. Description
- C. Parameters
- D. Metadata

<details>
<summary>Show answer</summary>

<br>

**B. Description**

[Notes: Template anatomy](cloudformation.md#template-anatomy)
</details>

**31. What happens when you delete a CloudFormation stack?**

- A. Only the stack is removed; its resources stay in the account
- B. The stack's physical resources are deleted too
- C. The template file is deleted
- D. The resources are stopped but kept

<details>
<summary>Show answer</summary>

<br>

**B. The stack's physical resources are deleted too**

[Notes: Stacks, logical and physical resources](cloudformation.md#stacks-logical-and-physical-resources)
</details>

---

### CloudWatch

**32. You need to monitor memory utilization on an EC2 instance. What do you do?**

- A. Enable detailed monitoring; memory is a built-in metric
- B. Install the CloudWatch agent on the instance
- C. Enable CloudTrail
- D. Enable VPC Flow Logs

<details>
<summary>Show answer</summary>

<br>

**B. Install the CloudWatch agent**

- EC2 reports CPU, network and disk I/O by default, but **memory and disk space usage** need the agent running inside the OS.

[Notes: How it fits together](cloudwatch.md#how-it-fits-together)
</details>

**33. What state does a new CloudWatch alarm start in?**

- A. OK
- B. ALARM
- C. INSUFFICIENT_DATA
- D. PENDING

<details>
<summary>Show answer</summary>

<br>

**C. INSUFFICIENT_DATA**

- It moves to OK or ALARM once there's enough data to evaluate.

[Notes: Alarms](cloudwatch.md#alarms)
</details>

**34. `CPUUtilization` in `AWS/EC2` receives data from every instance. What lets you view the CPU of one specific instance?**

- A. Namespace
- B. Dimension
- C. Datapoint
- D. Alarm

<details>
<summary>Show answer</summary>

<br>

**B. Dimension**

- e.g. `InstanceId=i-0abc123`. Dimensions are name/value pairs that split a metric into separate views.

[Notes: Datapoints and dimensions](cloudwatch.md#datapoints-and-dimensions)
</details>

**35. What does a CloudWatch datapoint consist of?**

- A. A namespace and a metric
- B. A timestamp and a value
- C. A dimension and an alarm
- D. A metric and a threshold

<details>
<summary>Show answer</summary>

<br>

**B. A timestamp and a value**

- e.g. `2026-11-05T14:30:00Z` and `98.3`.

[Notes: Datapoints and dimensions](cloudwatch.md#datapoints-and-dimensions)
</details>

---

### Shared Responsibility Model

**36. Who is responsible for patching the guest operating system on an EC2 instance?**

- A. AWS
- B. The customer
- C. Both equally
- D. Nobody, it's patched automatically

<details>
<summary>Show answer</summary>

<br>

**B. The customer**

- EC2 is IaaS, so everything from the OS up is yours. On managed services like RDS or Lambda, AWS patches the OS.

[Notes: It shifts with the service type](shared-responsibility-model.md#it-shifts-with-the-service-type)
</details>

**37. Which are AWS's responsibilities under the shared responsibility model?** *(Choose 2)*

- A. Physical security of data centers
- B. IAM user permissions
- C. The hypervisor that runs EC2 instances
- D. Encrypting customer data
- E. Security group configuration

<details>
<summary>Show answer</summary>

<br>

**A and C**

- AWS secures the cloud itself (security **OF** the cloud). IAM, encryption choices and security groups are things you configure, so they're yours (security **IN** the cloud).

[Notes: Who does what](shared-responsibility-model.md#who-does-what)
</details>

---

### HA vs Fault Tolerance vs Disaster Recovery

**38. What is a simple and correct definition of High Availability?**

- A. A system which maximises uptime
- B. A system which is highly performing
- C. A system which can operate through failure
- D. A system which has regular backups and restore processes

<details>
<summary>Show answer</summary>

<br>

**A. A system which maximises uptime**

- Operating through failure (C) is **fault tolerance**. Backups and restore (D) are **disaster recovery**.

[Notes: High Availability](ha-ft-dr.md#high-availability-ha)
</details>

**39. Which of the following is a correct definition of a fault-tolerant system?**

- A. A system which uses automation to return a service to operational status with little user disruption
- B. A system which has 99.999% uptime
- C. A system which allows failure, and can continue operating without disruption
- D. A system which has regular and reliable system backups and restore processes

<details>
<summary>Show answer</summary>

<br>

**C. A system which allows failure, and can continue operating without disruption**

- A describes **high availability** (fast recovery, *some* disruption). D describes **disaster recovery**.

[Notes: Fault Tolerance](ha-ft-dr.md#fault-tolerance-ft)
</details>

**40. RDS Multi-AZ fails over to its standby with an interruption of a minute or two. This is an example of:**

- A. Fault tolerance
- B. High availability
- C. Disaster recovery
- D. None of the above

<details>
<summary>Show answer</summary>

<br>

**B. High availability**

- There's a brief outage, then automatic recovery. Fault tolerance would mean no interruption at all.

[Notes: High Availability](ha-ft-dr.md#high-availability-ha)
</details>

**41. What does RPO (Recovery Point Objective) measure?**

- A. The maximum acceptable downtime
- B. The maximum acceptable data loss, measured in time
- C. The cost of recovery
- D. The number of regions used for backups

<details>
<summary>Show answer</summary>

<br>

**B. The maximum acceptable data loss**

- e.g. "we can lose at most the last hour of data". Maximum downtime (A) is **RTO**.

[Notes: Disaster Recovery](ha-ft-dr.md#disaster-recovery-dr)
</details>

---

### DNS

**42. How many DNS root servers exist?**

- A. 7
- B. 12
- C. 13
- D. 100

<details>
<summary>Show answer</summary>

<br>

**C. 13**

- `a.root-servers.net` to `m.root-servers.net`. Each is really many physical servers sharing one IP (anycast).

[Notes: Root servers vs the root zone](dns-basics.md#root-servers-vs-the-root-zone)
</details>

**43. Who manages the DNS root servers?**

- A. IANA
- B. Google
- C. 12 large organisations
- D. The IANA Root Server Board

<details>
<summary>Show answer</summary>

<br>

**C. 12 large organisations**

- 13 servers but 12 operators, because Verisign runs two. They host the root zone but don't control its content.

[Notes: Root servers vs the root zone](dns-basics.md#root-servers-vs-the-root-zone)
</details>

**44. Who manages the DNS root zone?**

- A. 12 large organisations
- B. IANA
- C. The IANA Root Server Board
- D. Microsoft
- E. Nobody, it's managed via the root hints file

<details>
<summary>Show answer</summary>

<br>

**B. IANA**

- The 12 organisations (A) operate the root **servers**. The root hints file (E) only tells resolvers where those servers are.

[Notes: Root servers vs the root zone](dns-basics.md#root-servers-vs-the-root-zone)
</details>

**45. Which DNS record type converts a host into an IPv4 address?**

- A. AAAA
- B. CNAME
- C. A
- D. MX
- E. TXT
- F. NS

<details>
<summary>Show answer</summary>

<br>

**C. A**

- **AAAA** is the IPv6 equivalent. **CNAME** maps a name to another name, not to an IP.

[Notes: A and AAAA](dns-record-types.md#a-and-aaaa)
</details>

**46. Which DNS record type is how the root zone delegates control of `.org` to the `.org` registry?**

- A. A
- B. TXT
- C. NS
- D. CNAME
- E. MX
- F. AAAA

<details>
<summary>Show answer</summary>

<br>

**C. NS**

- Every level of delegation uses NS records: root → `.org` → `example.org`.

[Notes: The DNS hierarchy](dns-basics.md#the-dns-hierarchy)
</details>

**47. Which type of organisation maintains the zones for a TLD (e.g. `.org`)?**

- A. Registrar
- B. IANA
- C. Registry
- D. None of the above

<details>
<summary>Show answer</summary>

<br>

**C. Registry**

- e.g. `.org` is maintained by Public Interest Registry, `.com` by Verisign.

[Notes: Registry vs registrar](dns-basics.md#registry-vs-registrar)
</details>

**48. Which type of organisation has a relationship with the `.org` TLD zone manager, allowing domain registration?**

- A. Registry
- B. Registrar
- C. IANA
- D. None of the above

<details>
<summary>Show answer</summary>

<br>

**B. Registrar**

- e.g. Route 53, Namecheap, GoDaddy. The registrar sells you the domain and passes the registration to the registry.

[Notes: Registry vs registrar](dns-basics.md#registry-vs-registrar)
</details>

**49. In Route 53, you want the bare domain `example.com` to point at an Application Load Balancer. Which record should you use?**

- A. CNAME
- B. An A record with the load balancer's IP address
- C. An alias record
- D. MX

<details>
<summary>Show answer</summary>

<br>

**C. An alias record**

- A CNAME can't be used at the zone apex (A). Load balancer IPs change, so hard-coding one (B) breaks. An alias can sit at the apex and points at the AWS resource itself.

[Notes: Records in Route 53](route53.md#records-in-route-53)
</details>

**50. A domain has two MX records with priorities 10 and 20. Which mail server is tried first?**

- A. Priority 20
- B. Priority 10
- C. One at random
- D. Both at the same time

<details>
<summary>Show answer</summary>

<br>

**B. Priority 10**

- A lower number means higher preference. Priority 20 is the backup.

[Notes: MX](dns-record-types.md#mx-mail-exchange)
</details>

**51. A record has a 24-hour TTL and you'll change its IP next week. How do you make sure users move to the new IP quickly?**

- A. Change the IP, then lower the TTL
- B. Lower the TTL well before the change, then change the IP
- C. Delete the record, then recreate it
- D. Nothing, TTL doesn't affect this

<details>
<summary>Show answer</summary>

<br>

**B. Lower the TTL well before the change**

- Resolvers cache the record for the old TTL. Lowering it at least one old TTL (24 hours) ahead means caches expire quickly when you make the switch.

[Notes: TTL](dns-record-types.md#ttl-time-to-live)
</details>

---

### Route 53

**52. How many managed nameservers host a Route 53 hosted zone?**

- A. 1
- B. 2
- C. 4
- D. 13

<details>
<summary>Show answer</summary>

<br>

**C. 4**

- 13 (D) is the number of DNS **root** servers.

[Notes: Hosted zones](route53.md#hosted-zones)
</details>

**53. Where can a Route 53 private hosted zone be resolved from?**

- A. The public internet
- B. Any VPC in the same account
- C. Only the VPCs associated with it
- D. On-premises networks, automatically

<details>
<summary>Show answer</summary>

<br>

**C. Only the VPCs associated with it**

[Notes: Hosted zones](route53.md#hosted-zones)
</details>

**54. How is Route 53 deployed?**

- A. Regional service, AZ resilient
- B. Global service, globally resilient
- C. Regional service, regionally resilient
- D. Global service, AZ resilient

<details>
<summary>Show answer</summary>

<br>

**B. Global service, globally resilient**

- One global database, replicated worldwide. You don't pick a region for it.

[Notes: Route 53 overview](route53.md#what-it-is)
</details>
