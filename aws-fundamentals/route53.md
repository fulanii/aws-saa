# **Route 53 Basics**

[← Back to AWS Fundamentals](readme.md)

> AWS's DNS service: it registers domains and hosts the DNS records for them, on a global, globally resilient platform. For the record types themselves, see [DNS Record Types](dns-record-types.md).

### Contents

- [What it is](#what-it-is)
- [Registering a domain](#registering-a-domain)
- [Hosted zones](#hosted-zones)
- [Records in Route 53](#records-in-route-53)
- [Django/FastAPI map](#djangofastapi-map)
- [Exam notes](#exam-notes)

---

### What it is

Route 53 does two separate jobs:

1. **Domain registration**: buy and renew domain names (like a registrar such as Namecheap or GoDaddy).
2. **DNS hosting**: host the **zone files** (all the DNS records) for a domain on **managed nameservers**.

You can use either job without the other, e.g. register a domain elsewhere and host its DNS in Route 53, or the reverse.

- **Global service** with a **single database**: you don't pick a region for it.
- **Globally resilient**: its data is replicated around the world, so it survives a whole region failing.

---

### Registering a domain

When you register `example.com` with Route 53, it:

1. **Checks the TLD registry** (the organization that runs `.com`) that the domain is available.
2. **Creates a zone file** for the domain, as a **public hosted zone**.
3. **Puts that zone on four managed nameservers**, spread across different TLDs for resilience (e.g. `ns-123.awsdns-15.com`, `ns-456.awsdns-57.net`, ...).
4. **Asks the TLD registry to add NS records** for `example.com` in the `.com` zone, pointing at those four nameservers.

```
 .com TLD zone                      Route 53 managed nameservers (x4)
 ─────────────                      ─────────────────────────────────
 example.com  NS  ns-123.awsdns...  ──>  hosted zone for example.com
              NS  ns-456.awsdns...         www      A      203.0.113.10
              NS  ...                      @        MX     10 mail.example.com
                                           ...
```

Step 4 is **delegation**: the `.com` servers now say "for anything about `example.com`, ask these four nameservers". That's what makes the records in your hosted zone answer real queries on the internet.

---

### Hosted zones

A **hosted zone** is Route 53's version of a **zone file**: a database holding all the DNS records for one domain.

- Hosted on **four managed nameservers**.
- Stores **records**, which AWS calls **record sets**.
- Created automatically when you register a domain in Route 53, or created by hand for a domain registered elsewhere (then you point that registrar's NS records at the four Route 53 nameservers).
- Costs a small monthly fee per hosted zone, plus a charge per DNS query.

**Two types:**

| | Public hosted zone | Private hosted zone |
|---|---|---|
| **Resolvable from** | The whole internet | Only the **VPCs it's associated with** |
| **Use it for** | Your public website and API (`api.example.com`) | Internal names (`db.internal.example.com`) that should never be visible publicly |

- A public and a private hosted zone can exist for the **same domain name** (**split-horizon DNS**). Queries from inside the VPC get the private answers, and everyone else gets the public ones.

---

### Records in Route 53

A hosted zone holds standard DNS records (A, AAAA, CNAME, MX, TXT, NS, SOA), each with a TTL. The full breakdown of every type is in [DNS Record Types](dns-record-types.md).

Route 53-specific points:

- Every hosted zone starts with an **NS** record (its four nameservers) and an **SOA** record, created for you.
- A **CNAME can't be used at the zone apex** (the bare `example.com`). Route 53 adds its own **alias** record to get around this. An alias can sit at the apex and point at AWS resources like a load balancer, CloudFront or an S3 website. Alias records and Route 53 **routing policies** come later in the course.

---

### Django/FastAPI map

- **Hosted zone**: where you add `api.example.com` pointing at your load balancer (an alias record), instead of configuring DNS at your registrar.
- **TXT records**: proving domain ownership for email providers (SES, SendGrid), Google Workspace, or SSL certificates.
- **CNAME records**: the records **ACM** (AWS Certificate Manager) asks you to add to validate an HTTPS certificate for your API.
- **MX records**: route company email for your domain to Google Workspace or another provider.
- **Private hosted zone**: give internal services friendly names (`db.internal`, `redis.internal`) for your `settings.py`, resolvable only inside your VPC.
- **Low TTL before a cutover**: when moving your app from a VPS to AWS, drop the TTL first so traffic shifts within minutes.

---

### Exam notes

- Route 53 = **domain registration + DNS hosting**. The two can be used separately.
- **Global service**, **globally resilient**, single database: no region to choose.
- A hosted zone is served by **four managed nameservers**.
- **Public hosted zone** = internet-resolvable. **Private hosted zone** = only from **associated VPCs**.
- Registering a domain → Route 53 creates the hosted zone and the **TLD registry gets NS records** pointing at Route 53's nameservers (**delegation**).
- **CNAME can't be at the apex**; Route 53 **alias** records can.
- Record type and TTL exam notes: see [DNS Record Types](dns-record-types.md#exam-notes).
