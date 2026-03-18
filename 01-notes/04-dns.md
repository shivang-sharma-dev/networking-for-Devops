# 04 — DNS (Domain Name System)

## What is DNS?

**DNS** translates human-readable domain names into IP addresses that computers use to communicate.

```
Browser asks: "Where is google.com?"
DNS answers:  "142.250.80.46"
```

Think of DNS as the **phone book of the internet**.

---

## DNS Resolution Flow

```
Browser
  │
  ├─► Checks local cache
  │     └─► If cached: done ✓
  │
  ├─► Asks Recursive Resolver (your ISP or 8.8.8.8)
  │     └─► If cached: returns IP ✓
  │
  ├─► Resolver asks Root Name Server (.)
  │     └─► Returns address of TLD nameserver
  │
  ├─► Resolver asks TLD Nameserver (.com)
  │     └─► Returns address of Authoritative Nameserver
  │
  └─► Resolver asks Authoritative Nameserver (google.com)
        └─► Returns final IP: 142.250.80.46 ✓
```

---

## DNS Record Types

| Record | Purpose | Example |
|---|---|---|
| **A** | Maps domain → IPv4 | `app.example.com → 1.2.3.4` |
| **AAAA** | Maps domain → IPv6 | `app.example.com → 2001:db8::1` |
| **CNAME** | Alias to another domain | `www → app.example.com` |
| **MX** | Mail server | `example.com → mail.example.com` |
| **TXT** | Text info (SPF, DKIM, verification) | `"v=spf1 include:..."` |
| **NS** | Nameservers for domain | `example.com → ns1.cloudflare.com` |
| **PTR** | Reverse DNS (IP → domain) | `1.2.3.4 → server1.example.com` |
| **SOA** | Start of Authority (zone metadata) | Admin info, serial number |
| **SRV** | Service location | `_http._tcp.example.com` |

---

## TTL — Time to Live

- TTL (in seconds) tells resolvers how long to **cache** a record
- **Low TTL** (60–300s): Changes propagate fast, more DNS queries
- **High TTL** (3600–86400s): Fewer queries, slower propagation

> ⚡ Before migrations or IP changes, lower TTL days in advance!

---

## DNS Commands

```bash
# Basic lookup
dig google.com
nslookup google.com

# Look up specific record type
dig google.com MX
dig google.com TXT
dig google.com AAAA

# Reverse lookup
dig -x 8.8.8.8

# Trace the full resolution path
dig +trace google.com

# Use a specific DNS server
dig @8.8.8.8 google.com
dig @1.1.1.1 google.com

# Short answer only
dig +short google.com

# Check DNS propagation (from host tool)
host google.com
host -t MX google.com
```

---

## Common DNS Servers

| Server | IP | Provider |
|---|---|---|
| Google | 8.8.8.8 / 8.8.4.4 | Google |
| Cloudflare | 1.1.1.1 / 1.0.0.1 | Cloudflare |
| OpenDNS | 208.67.222.222 | Cisco |
| Quad9 | 9.9.9.9 | Quad9 |

---

## DNS in DevOps — Key Concepts

### Split-Horizon DNS
- Return **different IPs** for the same domain based on where the query comes from
- Internal clients → private IP; External clients → public IP

### DNS Load Balancing
- Return **multiple A records** for the same domain
- Round-robin between them

### CNAME vs A Record
- Use **CNAME** for flexibility (CDN endpoints, load balancers) — e.g. `app.com → d1234.cloudfront.net`
- Use **A record** when you have a direct IP

### Route 53 (AWS DNS)
- Supports **health checks**, **weighted routing**, **geolocation routing**, **failover**
- Can route to ALB, EC2, CloudFront, S3

---

## Practical Tips

```bash
# Test if DNS is the issue (use IP directly)
curl http://1.2.3.4 -H "Host: app.example.com"

# Check DNS from inside a pod (Kubernetes)
kubectl exec -it <pod> -- nslookup kubernetes.default

# Flush DNS cache (Linux)
sudo systemd-resolve --flush-caches

# See current DNS resolver
cat /etc/resolv.conf
```
