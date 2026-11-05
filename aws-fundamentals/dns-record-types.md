# **DNS Record Types**

[← Back to AWS Fundamentals](readme.md)

> The records that live inside a zone file (or a Route 53 hosted zone), what each one does, and how TTL controls caching.

### Contents

- [Quick reference](#quick-reference)
- [NS (Nameserver)](#ns-nameserver)
- [A and AAAA](#a-and-aaaa)
- [CNAME (Canonical Name)](#cname-canonical-name)
- [MX (Mail Exchange)](#mx-mail-exchange)
- [TXT (Text)](#txt-text)
- [SOA (Start of Authority)](#soa-start-of-authority)
- [Others worth recognizing](#others-worth-recognizing)
- [TTL (Time To Live)](#ttl-time-to-live)
- [Django/FastAPI map](#djangofastapi-map)
- [Exam notes](#exam-notes)

---

### Quick reference

| Type | Maps | Example |
|---|---|---|
| **NS** | Zone → its authoritative nameservers | `example.com` → `ns-123.awsdns-15.com` |
| **A** | Name → **IPv4** address | `www` → `203.0.113.10` |
| **AAAA** | Name → **IPv6** address | `www` → `2001:db8::10` |
| **CNAME** | Name → **another name** | `ftp` → `www.example.com` |
| **MX** | Domain → mail servers, with priority | `10 mail.example.com` |
| **TXT** | Name → arbitrary text | `"v=spf1 include:amazonses.com ~all"` |
| **SOA** | Zone → admin info (one per zone) | primary nameserver, serial, timers |

---

### NS (Nameserver)

NS records are what make **delegation** work: they say which nameservers are **authoritative** for a zone.

- The `.com` zone holds NS records for `example.com` pointing at its nameservers (e.g. Route 53's four).
- Without them, nobody on the internet can find your zone. This is how the DNS hierarchy is stitched together: root → `.com` → `example.com`.

---

### A and AAAA

Both map a **hostname to an IP address**. The only difference is the IP version:

- **A** → **IPv4**, e.g. `www.example.com` → `203.0.113.10`
- **AAAA** ("quad A") → **IPv6**, e.g. `www.example.com` → `2001:db8::10`

It's common to have **both** for the same name, so clients on either IPv4 or IPv6 can connect. The client picks which to use.

---

### CNAME (Canonical Name)

A CNAME maps **one name to another name**, so several names can share a single A/AAAA record.

```
server.example.com   A      203.0.113.10
ftp.example.com      CNAME  server.example.com
mail.example.com     CNAME  server.example.com
www.example.com      CNAME  server.example.com
```

Change the IP once on `server`, and all three names follow.

- A CNAME **points only to a name, never directly to an IP address**.
- A CNAME **can't be used at the zone apex** (the bare `example.com`), and a name with a CNAME can't have any other records.
- Route 53 has its own workaround, **alias records**, covered in the [Route 53 notes](route53.md).

---

### MX (Mail Exchange)

MX records tell other mail servers **where to deliver email** for your domain. Each one has:

- **Priority**: a number. **Lower = tried first**. Servers with higher numbers are backups.
- **Value**: the mail server's hostname.

```
example.com   MX   10 mail.example.com        ← tried first
example.com   MX   20 mail-backup.example.com ← used if 10 fails
```

When someone emails `you@example.com`, their mail server looks up the MX records for `example.com`, picks the lowest priority, then resolves that hostname with an A/AAAA lookup.

- **Hostname format**: a value without a trailing dot (e.g. `mail`) is treated as **relative to the zone**, i.e. `mail.example.com`. A fully qualified value ends in a dot (e.g. `mail.other.com.`) and points **outside** the zone.

---

### TXT (Text)

TXT records hold **arbitrary text** attached to a name. Common uses:

- **Proving domain ownership**: a provider (Google, an email service, a certificate authority) asks you to add a specific TXT value. If they can see it, you control the domain.
- **Email security**: SPF (which servers may send mail for your domain), DKIM keys, DMARC policy. These fight spam and spoofing.

---

### SOA (Start of Authority)

Every zone has **exactly one** SOA record. It holds admin info about the zone: the primary nameserver, the admin contact, a serial number that increases on every change, and timers for how secondary nameservers refresh. Route 53 creates and manages it for you.

---

### Others worth recognizing

You won't need much detail on these, but you should know what they are:

- **CAA**: which certificate authorities are allowed to issue SSL/TLS certificates for your domain.
- **PTR**: reverse DNS, IP → name (the opposite of A).
- **SRV**: location (host and port) of a specific service, used by some protocols.

---

### TTL (Time To Live)

Every record has a **TTL** in seconds: how long resolvers may **cache** the answer before asking the authoritative nameserver again.

```
 You ──> Resolver (ISP / 8.8.8.8) ──> root ──> .com ──> example.com nameserver
              │
              └── caches the answer for TTL seconds; later queries get the cached
                  copy (a "non-authoritative" answer) without walking the tree again
```

- **Authoritative answer**: straight from the zone's own nameservers.
- **Non-authoritative answer**: from a resolver's cache, possibly a little out of date.
- **High TTL** (e.g. 86400 = 1 day): fewer queries, lower load and cost, but changes take up to a day to reach everyone.
- **Low TTL** (e.g. 60 seconds): changes spread fast, but there are more queries.
- **Migration tip**: **lower the TTL well before** a planned change (at least one old TTL ahead), make the change, then raise it again.

---

### Django/FastAPI map

- **A / AAAA**: point `api.example.com` at your server's IP. On AWS you'll usually use a Route 53 alias to a load balancer instead, because load balancer IPs change.
- **CNAME**: what ACM asks you to add to validate an HTTPS certificate, and how you point `docs.example.com` at a hosted docs platform.
- **MX**: send company email for your domain to Google Workspace or Microsoft 365.
- **TXT**: verify your domain with SES/SendGrid so transactional emails (password resets, receipts) from Django don't land in spam (SPF/DKIM).
- **TTL**: drop it to 60 a day before moving your API from a VPS to AWS, so the cutover takes minutes instead of hours.

---

### Exam notes

- **A = IPv4, AAAA = IPv6**, both name → IP.
- **CNAME = name → name**. It can't point at an IP and **can't be used at the zone apex**. In Route 53 the answer is usually an **alias** record.
- **MX**: **lower priority number = higher preference**.
- **TXT**: domain ownership verification and email security (SPF/DKIM).
- **NS** records enable **delegation** from a parent zone (e.g. `.com`) to your zone.
- **TTL** = cache duration. Low TTL means faster changes and more queries. Lower it **before** a migration.
