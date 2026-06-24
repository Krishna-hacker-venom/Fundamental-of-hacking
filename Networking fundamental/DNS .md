# DNS — TryHackMe Notes

---

## What is a Domain?

A **domain** is a human-readable name used to identify a location on the internet.

Instead of remembering `142.250.190.46`, you just type `google.com`.

**Real-world analogy:**
> A domain is like a shop name — "McDonald's" is easier to remember than GPS coordinates `28.6139° N, 77.2090° E`.

---

## What is DNS?

**DNS (Domain Name System)** is the internet's phonebook.

It translates human-friendly domain names into machine-readable IP addresses.

**Real-world analogy:**
> You search a contact name "Mom" in your phone → your phone finds her number `+91-98765-43210`.
> DNS does the same: `google.com` → `142.250.190.46`

---

## What is a Framework (in DNS context)?

A **framework** here means the **set of rules and structure** that defines how DNS works — how names are organized, how queries flow, and how records are stored.

Think of it as the **rulebook** the internet follows so every device can find any other device.

---

## What is an IP Address?

An **IP address** is a unique numerical label assigned to every device on a network.

| Type | Example | Use |
|------|---------|-----|
| IPv4 | `192.168.1.1` | 32-bit, most common |
| IPv6 | `2001:0db8::1` | 128-bit, newer standard |

**Real-world analogy:**
> An IP address is like your home's postal address — unique to your location, needed to deliver anything to you.

---

## Relation Between IP Address and DNS

```
User types:  google.com
     ↓
DNS resolves it to:  142.250.190.46
     ↓
Browser connects to that IP
```

- DNS = the translator
- IP = the actual destination
- Without DNS, you'd have to memorize IPs for every site

---

## Domain Hierarchy

DNS follows a tree-like structure, read **right to left**.

```
www  .  tryhackme  .  com  .
 │          │          │    │
Sub       Second      TLD  Root
domain    Level            (invisible dot)
          Domain
```

> The root is the invisible `.` at the very end. Your browser adds it silently — `google.com` is actually `google.com.` internally.

---

### Root Level Domain

- The **top of the entire DNS tree** — sits above all TLDs
- Represented as a single **dot** `.`
- Managed by **ICANN** and operated by 13 root server clusters worldwide (labeled `a.root-servers.net` through `m.root-servers.net`)
- Every DNS query starts here if no cached answer exists

**How it fits into a full lookup:**

```
You type: tryhackme.com

1. Query hits Root (.)         → "I don't know, ask .com TLD servers"
2. Query hits TLD (.com)       → "I don't know, ask tryhackme's nameservers"
3. Query hits NS (tryhackme)   → "Here's the IP: 104.21.57.152"
4. Browser connects to IP
```

**Real-world analogy:**
> The root is like the **central directory of all post offices on Earth**. You don't interact with it directly — but every letter has to be routed through its system first.

---

### Top Level Domain (TLD)

The **rightmost** part of a domain.

| TLD Type | Example | Meaning |
|----------|---------|---------|
| Generic (gTLD) | `.com`, `.org`, `.net`, `.edu` | General purpose |
| Country Code (ccTLD) | `.in`, `.uk`, `.us`, `.jp` | Country-specific |

---

### Generic Top Level Domain (gTLD)

Originally meant to tell users the **purpose** of a website:

| gTLD | Intended for |
|------|--------------|
| `.com` | Commercial |
| `.org` | Organisations |
| `.edu` | Education |
| `.gov` | Government |
| `.net` | Networks |

Over time, `.com` became used for almost everything.

---

### Country Code Top Level Domain (ccTLD)

Used for **country-specific** websites:

| ccTLD | Country |
|-------|---------|
| `.in` | India |
| `.uk` | United Kingdom |
| `.de` | Germany |
| `.jp` | Japan |

---

### Second Level Domain (SLD)

The part **directly to the left of the TLD**.

- In `tryhackme.com` → `tryhackme` is the SLD
- In `google.co.in` → `google` is the SLD

**Creation Restrictions:**
- Only letters `a-z`, digits `0-9`, and hyphens `-`
- **Cannot start or end with a hyphen**
- Max **63 characters** long

| Valid | Invalid |
|-------|---------|
| `tryhackme` | `-tryhackme` |
| `try-hackme` | `tryhackme-` |
| `hack101` | `hack_me` (no underscores) |

---

## What is a Subdomain?

A **subdomain** sits to the **left of the Second Level Domain**, separated by a `.` (period).

```
blog  .  tryhackme  .  com
 │           │           │
Sub        Second       TLD
domain     Level
```

**Examples:**

| Full Domain | Subdomain | Purpose |
|-------------|-----------|---------|
| `mail.google.com` | `mail` | Gmail |
| `maps.google.com` | `maps` | Google Maps |
| `admin.tryhackme.com` | `admin` | Admin panel |

**Rules for subdomains:**
- Same restrictions as SLD: `a-z`, `0-9`, hyphens only
- Cannot start or end with a hyphen
- Max **63 characters** per subdomain label
- You can chain **multiple subdomains**: `dev.api.tryhackme.com`

---

## Maximum Domain Length

The **total length** of a fully qualified domain name (FQDN) cannot exceed **253 characters**.

```
[subdomain.][subdomain.]second-level.tld
|___________________________________|
          max 253 characters
```

---

## What are DNS Record Types?

A **DNS record** tells the DNS server **what to do** with a domain name — like an instruction tag.

**Simple analogy:**
> You walk into a post office and ask for "John Smith".
> The record type tells the staff: *"Give him a letter" (A record), "Forward to another address" (CNAME), or "Send to his email server" (MX record).*

---

## DNS Record Types

### A Record

- Maps a domain to an **IPv4 address**
- "A" = Address

```
tryhackme.com  →  A  →  104.21.57.152
```

---

### AAAA Record

- Maps a domain to an **IPv6 address**
- Four A's = IPv6 (4x longer than IPv4)

```
tryhackme.com  →  AAAA  →  2606:4700:3035::ac43:b9e8
```

---

### CNAME Record (Canonical Name)

- Maps a domain to **another domain name** (an alias)
- Useful when multiple names point to the same place

```
shop.tryhackme.com  →  CNAME  →  stores.tryhackme.com
stores.tryhackme.com  →  A  →  104.21.57.152
```

**Real-world analogy:**
> "Call me at my office" → which then has its own phone number.

---

### MX Record (Mail Exchange)

- Tells where to send **emails** for a domain
- Has a **priority number** — lower = higher priority

```
tryhackme.com  MX  10  mail1.tryhackme.com
tryhackme.com  MX  20  mail2.tryhackme.com   ← backup
```

**Real-world analogy:**
> "Send all letters to Post Office #1. If it's closed, use Post Office #2."

---

### NS Record (Name Server)

- Tells which **DNS servers** are authoritative for a domain
- These servers hold the actual DNS records

```
tryhackme.com  →  NS  →  ns1.cloudflare.com
tryhackme.com  →  NS  →  ns2.cloudflare.com
```

**Real-world analogy:**
> "To find info about this company, go ask the office at Building A."

---

### TXT Record

- Stores **free-form text** in DNS
- Used for domain verification, anti-spam (SPF, DKIM), and security configs

```
tryhackme.com  TXT  "v=spf1 include:mailgun.org ~all"
tryhackme.com  TXT  "google-site-verification=abc123"
```

**Hacker note:** TXT records often leak internal info — always check them during recon.

---

### SOA Record (Start of Authority)

- Stores **administrative info** about a DNS zone
- Contains: primary nameserver, admin email, serial number, refresh intervals

```
tryhackme.com  SOA  ns1.cloudflare.com  admin.tryhackme.com  (
    2024010101  ; Serial
    3600        ; Refresh
    900         ; Retry
    604800      ; Expire
    300 )       ; Minimum TTL
```

**Real-world analogy:**
> The SOA is the "owner's manual" of a DNS zone — who's in charge, when to update, when records expire.

---

### PTR Record (Pointer Record)

- The **reverse** of an A record — maps an **IP back to a domain name**
- Used in reverse DNS lookups

```
104.21.57.152  →  PTR  →  tryhackme.com
```

**Real-world analogy:**
> You have a phone number `+91-98765-43210` → you reverse-lookup → "Mom"

**Hacker use:** PTR records reveal hostnames from IPs during network recon.

---

## Quick Reference Table

| Record | Full Name | Maps | Example Use |
|--------|-----------|------|-------------|
| `A` | Address | Domain → IPv4 | Website IP |
| `AAAA` | IPv6 Address | Domain → IPv6 | IPv6 website |
| `CNAME` | Canonical Name | Domain → Domain | Alias/redirect |
| `MX` | Mail Exchange | Domain → Mail server | Email routing |
| `NS` | Name Server | Domain → DNS server | Zone authority |
| `TXT` | Text | Domain → Text string | SPF, verification |
| `SOA` | Start of Authority | Zone metadata | Admin info |
| `PTR` | Pointer | IP → Domain | Reverse lookup |

---

