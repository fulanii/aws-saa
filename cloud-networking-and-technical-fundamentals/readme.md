# **Cloud, Networking and Technical Fundamentals**

### Networking, Security, and Cloud Fundamentals: Plain-English Overview
****
Each topic is covered as **What**, **Backend dev**, and **AWS**.

---

## OSI 7-Layer Model

**OSI Model Introduction**

- **What:** A reference model that splits networking into 7 layers (Physical, Data Link, Network, Transport, Session, Presentation, Application). Each layer does one job and relies on the one below it.
- **Backend dev:** It gives you a vocabulary for debugging ("is this an L3 routing problem or an L7 app problem?").
- **AWS:** Services map to layers: ENI (L2), VPC routing and NACLs (L3), Security Groups and NLB (L4), ALB and WAF (L7).

**Layer 1: Physical**

- **What:** Raw bits moving as electrical, light, or radio signals over cables or wireless. No addressing or logic, just signal transmission.
- **Backend dev:** You rarely touch it, but it sets the floor for latency and bandwidth.
- **AWS:** Hidden from you, except Direct Connect, where you literally get a physical fiber cross-connect to AWS.

**Layer 2: Data Link**

- **What:** Delivers frames between devices on the *same local network* using MAC addresses. Switches live here and learn which MAC is on which port. It also covers collisions and how shared media were managed (hubs vs switches). ARP bridges L3 IPs to L2 MACs.
- **Backend dev:** Mostly invisible, but it shows up in Docker bridge networks, Kubernetes CNI plugins, and "same subnet vs different subnet" behavior.
- **AWS:** Each ENI has a MAC address, but VPCs don't give you a real L2 network (no true broadcast, and you can't sniff other instances' traffic).

**Layer 3: Network** 

- **What:** IP addressing and routing between different networks. Covers IP packets, subnet masks, routers and route tables, ARP, TTL, ICMP (ping/traceroute), and fragmentation.
- **Backend dev:** IP allowlists, "why can't service A reach service B", ping/traceroute debugging, and CIDR planning.
- **AWS:** The heart of VPC: route tables, Internet Gateway, NAT Gateway, peering, and Transit Gateway. NACLs and Security Groups also filter at this layer.

**Layer 4 and 5: Transport and Session**

- **What:** **TCP** (reliable, ordered, handshake-based) and **UDP** (fast, no guarantees) using **ports** to reach specific apps. TCP builds a stateful "session" between two endpoints. L5 is about managing those ongoing conversations.
- **Backend dev:** Sockets, connection pooling, timeouts, keepalives, and port numbers (5432, 443, etc.). Most "connection reset/timeout" bugs live here.
- **AWS:** Network Load Balancer (L4), Security Group port rules, NACL ephemeral ports, and stateful connection tracking.

---

## Other Networking

**Network Address Translation (NAT)** 

- **What:** Rewrites IP addresses in packets. Static NAT is 1:1, dynamic NAT uses a pool, and PAT (many-to-one, using ports) lets many private hosts share one public IP.
- **Backend dev:** Why private servers can call external APIs but can't be reached from outside, and why you sometimes see NAT port-exhaustion or idle-connection drops.
- **AWS:** The Internet Gateway does static 1:1 NAT for instances with public IPv4. NAT Gateway does PAT so private-subnet instances can reach the internet outbound only.

**IP Address Space and Subnetting**

- **What:** How the IP address range is divided: public vs private ranges, CIDR notation (`/16`, `/24`), and carving networks into smaller subnets.
- **Backend dev:** Designing network layouts, allowlisting ranges, and avoiding overlapping CIDRs between environments.
- **AWS:** Every VPC and subnet is defined with a CIDR (VPC /16 to /28). AWS reserves 5 IPs per subnet. Overlapping CIDRs break peering and VPN connectivity.

**DDoS Attacks**

- **What:** Flooding a target with traffic or requests to make it unavailable. Types: volumetric (saturate bandwidth), protocol (SYN floods), and application-layer (expensive HTTP requests). Amplification and reflection attacks (like DNS/NTP) multiply attacker traffic using third parties.
- **Backend dev:** Rate limiting, caching, autoscaling, and not exposing origin servers directly.
- **AWS:** **Shield** (Standard is free and automatic), WAF (L7 rules and rate limits), CloudFront and Route 53 (absorb traffic at the edge), Global Accelerator, and Auto Scaling.

**VLANs, Trunks and QinQ**

- **What:** VLANs split one physical switch network into multiple isolated logical L2 networks via 802.1Q tags. Trunks carry multiple VLANs over one link. QinQ stacks two tags so providers can carry customers' VLANs inside their own.
- **Backend dev:** Rarely directly, but it's how data centers isolate traffic between teams and environments.
- **AWS:** Direct Connect uses VLANs under the hood (each virtual interface is a VLAN). Inside AWS, subnets and VPCs are the equivalent isolation.

**Decimal to Binary Conversion (IP Addressing)**

- **What:** IPv4 addresses are 32 bits shown as four decimal octets. Converting to binary is how subnet masks and CIDR actually work.
- **Backend dev:** Helps you reason about whether an IP falls in a CIDR range.
- **AWS:** Needed for subnet planning and exam-style "how many hosts / which subnet contains this IP" questions.

**SSL and TLS**

- **What:** Protocols that encrypt traffic and authenticate servers using certificates. Handshake negotiates ciphers and exchanges keys (asymmetric to set up, symmetric for the actual data). SSL is the old name, and TLS is the modern version.
- **Backend dev:** HTTPS everywhere, cert expiry and chain issues, mTLS between services, and TLS termination at proxies.
- **AWS:** **ACM** issues and renews certs for ALB, NLB, CloudFront, and API Gateway. LBs can terminate TLS or pass it through.

**Border Gateway Protocol (BGP) 101**

- **What:** The routing protocol of the internet. Networks (Autonomous Systems) announce which IP ranges they can reach, and BGP picks paths between them. It's trust-based, so bad announcements can cause outages.
- **Backend dev:** Explains big internet outages and why traffic takes the paths it does.
- **AWS:** Used by Direct Connect and Site-to-Site VPN for dynamic route exchange with on-prem. Also underpins anycast for CloudFront and Global Accelerator.

**Stateful vs Stateless Firewalls**

- **What:** Stateless firewalls judge each packet alone, so you must allow both directions. Stateful firewalls track connections and automatically allow return traffic.
- **Backend dev:** Why a rule works for requests but responses get blocked (or not).
- **AWS:** Security Groups = stateful, NACLs = stateless. This is a heavily tested exam topic.

**Jumbo Frames**

- **What:** Larger packets (up to ~9000 bytes) than the standard 1500-byte MTU, reducing per-packet overhead for bulk data.
- **Backend dev:** Matters for high-throughput transfers. MTU mismatches cause "small requests work, large ones hang" bugs.
- **AWS:** Supported inside a VPC (9001 MTU), but traffic leaving via an Internet Gateway or VPN is limited to 1500. Useful with cluster placement groups.

**Layer 7 Firewalls**

- **What:** Firewalls that inspect actual application content (URLs, headers, payloads), not just IPs and ports. Can block SQL injection, XSS, bad bots, etc.
- **Backend dev:** Protecting APIs and web apps from application-level attacks.
- **AWS:** AWS WAF (on CloudFront, ALB, API Gateway), Network Firewall (deep packet inspection), and Gateway Load Balancer for third-party appliances.

**IPsec VPN Fundamentals**

- **What:** Encrypted tunnels between two networks over the public internet. Set up in phases: first the peers authenticate and agree on keys (IKE), then data tunnels are established.
- **Backend dev:** Connecting a cloud environment to an office or data center securely.
- **AWS:** **Site-to-Site VPN** (two tunnels for redundancy, to a Virtual Private Gateway or Transit Gateway). It's quick and cheap versus Direct Connect.

**Fibre Optic Cable 101**

- **What:** Cables that carry data as light. Single-mode goes long distances, multi-mode is cheaper for short runs. Very high bandwidth, low interference.
- **Backend dev:** Background knowledge, as it sets the speed and latency limits of the internet.
- **AWS:** Direct Connect uses fiber (single-mode) and AWS's global backbone is fiber, which is why Global Accelerator can beat the public internet.

---

## Security

**Encryption 101 (Parts 1 and 2)**

- **What:** Scrambling data so only key holders can read it. Symmetric uses one shared key (fast), asymmetric uses a public/private key pair (solves key exchange). Plaintext → ciphertext. "At rest" vs "in transit".
- **Backend dev:** HTTPS, encrypted DB columns and disks, secrets handling, and signing tokens. Note: password hashing is not encryption.
- **AWS:** KMS manages keys, and encryption is available for S3, EBS, RDS, DynamoDB, and more. TLS covers in-transit.

**Envelope Encryption**

- **What:** Encrypt your data with a data key, then encrypt that data key with a master key. Store the encrypted data key next to the data. This avoids sending big data to a key service and keeps the master key protected.
- **Backend dev:** The standard pattern for encrypting files or records at scale.
- **AWS:** **KMS** `GenerateDataKey` does exactly this. S3 SSE-KMS, EBS, and RDS all use it. KMS can only directly encrypt up to 4 KB, hence the envelope approach.

**Hardware Security Modules (HSMs)**

- **What:** Tamper-resistant hardware devices that generate and store keys, and perform crypto operations without ever exposing the key.
- **Backend dev:** Used where compliance or high security demands keys never exist in plain software.
- **AWS:** KMS uses HSMs behind the scenes (multi-tenant, managed). CloudHSM gives you your own dedicated, single-tenant HSM where *you* control the keys.

**Hash Functions and Hashing**

- **What:** One-way functions that turn any data into a fixed-length fingerprint. Same input gives the same output, and you can't reverse it. Used for integrity checks and password storage.
- **Backend dev:** Password hashing (bcrypt/argon2 with salt), file checksums, ETags, Git commits, and deduplication.
- **AWS:** S3 checksums/ETags, CloudTrail log file integrity validation, and request signing (SigV4 uses HMAC-SHA256).

**Digital Signatures**

- **What:** Sign a hash with a private key, and anyone can verify it with the public key. Proves who sent it (authenticity), that it wasn't changed (integrity), and the signer can't deny it (non-repudiation).
- **Backend dev:** JWTs (RS256), webhook signatures, signed commits, and TLS certificates.
- **AWS:** KMS asymmetric keys can sign/verify, CloudTrail digest files are signed, Lambda code signing, and ACM certificates rely on signatures.

---

## DNS and DNSSEC

**DNS #1: What does DNS do**

- **What:** The internet's phonebook, translating names (`example.com`) into IP addresses.
- **Backend dev:** Every service call by hostname starts with a DNS lookup. Service discovery often uses DNS.
- **AWS:** Route 53 is AWS's DNS service. The VPC also has a built-in resolver.

**DNS #2: Why does DNS need a complex architecture**

- **What:** A single database couldn't handle the scale or trust model, so DNS is hierarchical and distributed (root → TLD → domain), with caching to cut load.
- **Backend dev:** Explains why DNS changes take time (TTLs/caching) and why it's reliable.
- **AWS:** Route 53 acts as the authoritative layer, with caching happening in resolvers everywhere else.

**DNS #3: How DNS works, walking the tree**

- **What:** The step-by-step lookup: resolver asks root, then the TLD (`.com`), then the domain's authoritative nameserver, and caches the answer.
- **Backend dev:** Debugging with `dig +trace`, and understanding why one machine sees a different answer than another.
- **AWS:** Route 53 hosted zones (public/private) are the authoritative servers. **Route 53 Resolver** handles lookups inside VPCs and hybrid setups.

**DNS #4: What happens when a domain is registered**

- **What:** A registrar records your domain with the registry (like `.com`'s operator), which adds NS records pointing to your nameservers, delegating authority to you.
- **Backend dev:** Why you update nameservers at the registrar after moving DNS providers.
- **AWS:** Route 53 can be both registrar and DNS host. If domain is registered elsewhere, point its NS records to your Route 53 hosted zone.

**DNSSEC #1: Why we need it**

- **What:** Plain DNS can be spoofed (poisoned) so you're sent to a fake site. DNSSEC adds cryptographic proof that answers are authentic.
- **Backend dev:** Protects users from DNS hijacking. It guarantees authenticity, not privacy.
- **AWS:** Route 53 supports DNSSEC signing for public hosted zones.

**DNSSEC #2: How it works within a zone**

- **What:** Records get signed (RRSIG) using a zone's keys (DNSKEY, typically ZSK and KSK). Resolvers verify signatures against the published public keys.
- **Backend dev:** Mostly set-and-forget, but misconfigured keys can take a domain offline for validating resolvers.
- **AWS:** Route 53 uses a **KMS asymmetric key** (in us-east-1) as the signing key.

**DNSSEC #3: Chain of trust**

- **What:** Each level vouches for the one below via **DS records** (a hash of the child's key stored in the parent). Trust flows from the root down to your domain.
- **Backend dev:** Why you must publish a DS record at your registrar after enabling DNSSEC.
- **AWS:** Route 53 gives you the DS details to add at the registrar (or does it automatically if Route 53 is the registrar).

**DNSSEC #4: Root signing ceremony**

- **What:** A highly controlled, witnessed ritual where the root zone's keys are used and protected (HSMs, multiple trusted key holders), establishing the ultimate anchor of trust.
- **Backend dev:** Background knowledge on how trust is bootstrapped.
- **AWS:** Not AWS-specific. It's the foundation that makes Route 53 DNSSEC validation meaningful.

---

## Containers and Virtualization

**Kubernetes 101**

- **What:** An orchestration platform that runs and manages containers across a cluster: scheduling, scaling, self-healing, rolling updates, and service discovery. Core pieces: pods, deployments, services, ingress.
- **Backend dev:** How modern microservices are deployed, scaled, and exposed.
- **AWS:** EKS is managed Kubernetes. Alternatives are ECS (AWS's own orchestrator) and Fargate (serverless container compute).

---

## Backups and DR

**RPO and RTO**

- **What:** **RPO** (Recovery Point Objective) is how much data loss you can tolerate (time since last good backup). **RTO** (Recovery Time Objective) is how long you can be down. Lower values cost more.
- **Backend dev:** Drives your backup frequency, replication design, and failover plans.
- **AWS:** DR strategies from cheapest to fastest: backup and restore → pilot light → warm standby → multi-site active/active. Tools: AWS Backup, RDS snapshots and cross-region replicas, S3 replication, Route 53 failover, Elastic Disaster Recovery.

---

## Data and Config Formats

**YAML 101**

- **What:** A human-readable, indentation-based format for configuration. Supports maps, lists, and scalars.
- **Backend dev:** Kubernetes manifests, docker-compose, CI/CD pipelines (GitHub Actions), and OpenAPI specs. Watch indentation errors.
- **AWS:** CloudFormation and SAM templates, CodeBuild buildspecs.

**JSON 101**

- **What:** A lightweight, universal text format for structured data (objects, arrays, strings, numbers, booleans, null).
- **Backend dev:** The default for REST API request/response bodies and config.
- **AWS:** IAM policies, CloudFormation (alt format), Lambda event payloads, API Gateway, and CLI output are all JSON.

---

## Cloud Computing 101

**What is Cloud Computing?**

- **What:** On-demand IT resources over the internet with pay-as-you-go pricing. Key traits: on-demand self-service, broad network access, resource pooling, rapid elasticity, measured service.
- **Backend dev:** You provision infrastructure with code or API calls instead of buying servers, and design for scaling.
- **AWS:** The whole platform. Core ideas: elasticity, global regions/AZs, and the shared responsibility model.

**Public vs Private vs Multi vs Hybrid Cloud**

- **What:** Public is shared provider infrastructure (AWS, Azure, GCP). Private is dedicated to one organization. Hybrid mixes public with on-prem. Multi-cloud uses more than one public provider.
- **Backend dev:** Affects architecture choices, compliance, and portability.
- **AWS:** Hybrid via Direct Connect, Site-to-Site VPN, and Outposts (AWS hardware on-prem).

**Cloud Service Models (IaaS, PaaS, SaaS)**

- **What:** How much the provider manages for you. IaaS gives raw infrastructure (you manage OS and up), PaaS gives a platform (you manage just your code and data), SaaS is a finished application.
- **Backend dev:** Choose the level of control vs convenience for each component.
- **AWS:** IaaS is EC2/EBS/VPC, PaaS is Elastic Beanstalk, RDS, and Lambda (roughly), and SaaS is things like Amazon Chime or WorkMail.