# **DNS Basics: Root Servers, Registries & Registrars**

[← Back to AWS Fundamentals](readme.md)

> Who runs the top of DNS, how control is handed down from the root to your domain, and the difference between a registry and a registrar. For the step-by-step lookup and DNSSEC, see [Cloud, Networking and Technical Fundamentals](../cloud-networking-and-technical-fundamentals/readme.md#dns-and-dnssec).

### Contents

- [The DNS hierarchy](#the-dns-hierarchy)
- [Root servers vs the root zone](#root-servers-vs-the-root-zone)
- [Root hints](#root-hints)
- [Registry vs registrar](#registry-vs-registrar)
- [Authoritative nameservers](#authoritative-nameservers)
- [Django/FastAPI map](#djangofastapi-map)
- [Exam notes](#exam-notes)

---

### The DNS hierarchy

DNS is a tree of **zones**. Each level only knows about the level directly below it and **delegates** control downward with **NS records**.

```
                  Root zone  "."          managed by IANA, hosted on the 13 root servers
                      │  NS records for every TLD
          ┌───────────┼───────────┐
        .com        .org        .net       TLD zones, each run by a registry
          │  NS records for every domain in .com
     example.com                           your zone, hosted on your nameservers
          │
     www.example.com  A  203.0.113.10      the actual record
```

- The **root zone** delegates `.org` to the `.org` registry with **NS records**.
- The `.org` zone delegates `example.org` to that domain's nameservers, again with **NS records**.
- Nobody controls the whole tree. Each zone only answers for its own level and points to the next one down.

---

### Root servers vs the root zone

These are two different things, managed by different people:

| | Root zone | Root servers |
|---|---|---|
| **What it is** | The **data**: the zone file at the top of DNS, listing every TLD and its nameservers | The **servers** that host and answer queries for the root zone |
| **Who manages it** | **IANA** (Internet Assigned Numbers Authority) | **12 large organisations** (e.g. Verisign, NASA, the US Army, universities) |
| **How many** | One | **13** root server identities, named `a.root-servers.net` to `m.root-servers.net` |

- 13 identities but 12 operators, because **Verisign runs two** (A and J).
- Each of the 13 isn't one machine: it's hundreds of physical servers around the world, all sharing one IP address (**anycast**).
- The root server operators **host** the root zone but **don't control its content**. Only IANA changes what's in it.

---

### Root hints

How does a resolver find the root servers in the first place? A **root hints file** comes with every DNS resolver. It lists the names and IP addresses of the 13 root servers, so the resolver always knows where to start a lookup.

---

### Registry vs registrar

| | Registry | Registrar |
|---|---|---|
| **What it does** | **Maintains the zone for a TLD** (e.g. `.org`) and runs its nameservers | **Sells domains to you**, and has a relationship with the registry that lets it register domains in that TLD |
| **Examples** | `.com` → Verisign, `.org` → Public Interest Registry | Route 53, Namecheap, GoDaddy |
| **You deal with it** | Never directly | Whenever you buy or renew a domain |

When you register `example.org` through a registrar:

1. The **registrar** sends your registration to the **`.org` registry**.
2. The **registry** adds **NS records** for `example.org` to the `.org` zone, pointing at your nameservers (e.g. Route 53's four).
3. Your domain is now **delegated**: queries for it are sent to your nameservers.

> Route 53 can be **both** your registrar (selling you the domain) and your DNS host (running the hosted zone). See [Route 53](route53.md#registering-a-domain).

---

### Authoritative nameservers

- The nameservers named in a zone's NS records are **authoritative** for it: their answers are the source of truth.
- Answers served from a resolver's cache are **non-authoritative**. They may be a little out of date until the [TTL](dns-record-types.md#ttl-time-to-live) expires.

---

### Django/FastAPI map

- **Registrar**: where you bought your domain and where you **change the nameservers** when moving DNS hosting to Route 53.
- **Registry**: invisible to you, but it's why a nameserver change at your registrar can take time to apply: the TLD zone has to be updated, and resolvers cache the old NS records.
- **Delegation**: you can delegate a subdomain like `api.example.com` to a separate hosted zone (e.g. one per environment or team) by adding NS records for it in the parent zone.

---

### Exam notes

- **13** root servers (a to m), operated by **12 large organisations**.
- **IANA** manages the **root zone**. The root server operators only host it.
- The root zone delegates TLDs (e.g. `.org`) with **NS** records. Every level of delegation uses NS records.
- **Registry** = maintains a TLD's zone. **Registrar** = has a relationship with the registry and lets you register domains.
- The **root hints** file tells resolvers where the root servers are.
