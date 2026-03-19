# 04 — DNS (Domain Name System)

> DNS is the phonebook of the internet. Without it you would have
> to memorise IP addresses for every website. Understanding DNS
> is critical — misconfigured DNS is one of the most common causes
> of production outages.

---

## The Problem DNS Solves

```
WITHOUT DNS                         WITH DNS

curl 142.250.80.46                  curl google.com
curl 104.26.10.78                   curl cloudflare.com
curl 151.101.1.69                   curl reddit.com

Nobody can remember IP addresses.
DNS translates human-readable names into IP addresses.
```

---

## The Phonebook Analogy

```
DNS is exactly like a phonebook — but for the internet.

  You want to call "Alice Smith"
  You look up "Alice Smith" in the phonebook
  You find her number: 555-1234
  You dial 555-1234

  You want to visit "google.com"
  Your computer asks DNS: "what is the IP for google.com?"
  DNS responds: "142.250.80.46"
  Your computer connects to 142.250.80.46
```

---

## How DNS Resolution Works — Step by Step

```
You type: google.com in your browser

Step 1: Check local cache
  └─ Your OS checks if it already knows the IP
     (from a recent lookup — saves time)
     If found → done. Use cached IP.
     If not → continue

Step 2: Check /etc/hosts
  └─ Your OS checks the local hosts file
     (manual overrides live here)
     If found → done.
     If not → continue

Step 3: Ask your Recursive Resolver
  └─ Your OS asks your configured DNS server
     (usually your ISP's or 8.8.8.8 or 1.1.1.1)
     This resolver does the hard work FOR you

Step 4: Recursive Resolver asks Root Nameserver
  └─ "Who handles .com domains?"
     Root: "Ask the .com TLD server at 192.5.6.30"

Step 5: Ask .com TLD Nameserver
  └─ "Who handles google.com?"
     TLD: "Ask Google's nameserver at ns1.google.com"

Step 6: Ask Google's Authoritative Nameserver
  └─ "What is the IP for google.com?"
     Google NS: "142.250.80.46"

Step 7: Response returned + cached
  └─ Resolver tells your OS: 142.250.80.46
     OS caches it for the TTL duration
     Browser connects to 142.250.80.46
```

```
YOUR COMPUTER
     │
     │ 1. "What is google.com?"
     ▼
RECURSIVE RESOLVER (8.8.8.8)
     │
     │ 2. "Who handles .com?"
     ▼
ROOT NAMESERVER (13 worldwide)
     │
     │ "Ask .com TLD server"
     ▼
.COM TLD NAMESERVER
     │
     │ "Ask Google's nameserver"
     ▼
GOOGLE'S AUTHORITATIVE NAMESERVER
     │
     │ "142.250.80.46"
     ▼
RECURSIVE RESOLVER caches + returns to your computer
     │
     ▼
YOUR COMPUTER connects to 142.250.80.46
```

---

## DNS Record Types

```
┌────────┬──────────────────────────────────────────────────────────┐
│   A    │ Maps hostname → IPv4 address                             │
│        │ google.com → 142.250.80.46                              │
├────────┼──────────────────────────────────────────────────────────┤
│  AAAA  │ Maps hostname → IPv6 address                             │
│        │ google.com → 2607:f8b0:4004:c09::6a                     │
├────────┼──────────────────────────────────────────────────────────┤
│  CNAME │ Alias — maps one name to another name                    │
│        │ www.google.com → google.com                              │
│        │ (CNAME points to a name, A points to an IP)              │
├────────┼──────────────────────────────────────────────────────────┤
│   MX   │ Mail exchange — where to send email for a domain         │
│        │ google.com → aspmx.l.google.com (priority 10)           │
├────────┼──────────────────────────────────────────────────────────┤
│   TXT  │ Text record — arbitrary text, used for verification      │
│        │ SPF, DKIM, domain ownership verification                 │
├────────┼──────────────────────────────────────────────────────────┤
│   NS   │ Nameserver — which servers are authoritative for domain  │
│        │ google.com NS → ns1.google.com, ns2.google.com          │
├────────┼──────────────────────────────────────────────────────────┤
│  PTR   │ Reverse DNS — maps IP → hostname                         │
│        │ 142.250.80.46 → lga34s32-in-f14.1e100.net               │
├────────┼──────────────────────────────────────────────────────────┤
│  SOA   │ Start of Authority — info about the zone itself          │
│        │ Primary NS, admin email, serial number, TTL values       │
├────────┼──────────────────────────────────────────────────────────┤
│  SRV   │ Service record — location of specific services           │
│        │ _http._tcp.example.com → server + port                  │
└────────┴──────────────────────────────────────────────────────────┘
```

---

## TTL — Time to Live

TTL tells resolvers how long to cache a DNS record (in seconds).

```
google.com A record TTL = 300 (5 minutes)

This means:
  After a lookup, resolvers cache the result for 5 minutes.
  For 5 minutes, no new DNS queries are needed for google.com.
  After 5 minutes, the cache expires and a fresh lookup happens.

Short TTL (60–300s)   → changes propagate fast, more DNS queries
Long TTL (86400s=1d)  → changes take longer to propagate, fewer queries

Best practices:
  Normal operation     → 3600  (1 hour)
  Before a migration   → 300   (5 min) — set this 24hrs before change
  After migration done → 3600  (back to normal)
```

---

## DNS Commands

```bash
# Basic lookup
nslookup google.com               # simple A record lookup
nslookup google.com 8.8.8.8      # query specific DNS server
nslookup -type=MX google.com      # MX records

# dig — detailed DNS queries (best tool)
dig google.com                    # full A record response
dig google.com A                  # A records only
dig google.com MX                 # mail records
dig google.com NS                 # nameservers
dig google.com TXT                # text records
dig google.com ANY                # all records
dig +short google.com             # just the IP
dig @8.8.8.8 google.com          # query specific server (Google DNS)
dig @1.1.1.1 google.com          # query Cloudflare DNS
dig +trace google.com             # trace full resolution path

# Reverse DNS
dig -x 142.250.80.46              # IP → hostname
host 142.250.80.46                # simpler reverse lookup

# Check DNS propagation
dig @8.8.8.8 example.com A       # check from Google DNS
dig @1.1.1.1 example.com A       # check from Cloudflare DNS
dig @208.67.222.222 example.com A # check from OpenDNS

# Local DNS config
cat /etc/resolv.conf              # your configured DNS servers
cat /etc/hosts                    # local overrides
resolvectl status                 # systemd-resolved status
systemd-resolve --status          # detailed resolver info
```

---

## Reading dig Output

```bash
dig google.com

# ;; QUESTION SECTION:
# ;google.com.    IN  A         ← querying A record for google.com

# ;; ANSWER SECTION:
# google.com.  300  IN  A  142.250.80.46
# ─────────────────────────────────────
# domain  TTL  class type  value

# ;; Query time: 12 msec          ← how long the query took
# ;; SERVER: 127.0.0.53#53        ← which DNS server answered
# ;; MSG SIZE  rcvd: 55            ← response size in bytes
```

---

## /etc/hosts — Local DNS Override

```bash
cat /etc/hosts
# 127.0.0.1   localhost
# 127.0.1.1   myhostname
# 192.168.1.10  myserver.local
# 10.0.0.5    db.internal staging-db

# Add an entry (as root)
echo "192.168.1.20  dev.myapp.local" >> /etc/hosts

# Now this resolves without DNS:
curl http://dev.myapp.local

# Use case: override DNS for testing before going live
# Point prod domain to staging server in /etc/hosts
echo "10.0.1.50  myapp.com" >> /etc/hosts
```

---

## Common DNS Issues and Fixes

```
Issue: "Name or service not known" / "Could not resolve host"

  Check 1: DNS server reachable?
    dig @8.8.8.8 google.com       # bypass local DNS

  Check 2: /etc/resolv.conf correct?
    cat /etc/resolv.conf
    nameserver 8.8.8.8            # should have a valid nameserver

  Check 3: Network connected?
    ping 8.8.8.8                  # can I reach internet by IP?

  Check 4: Local resolver running?
    systemctl status systemd-resolved
    resolvectl status

Issue: DNS resolves wrong IP (propagation delay)

  Check different DNS servers:
    dig @8.8.8.8 yourdomain.com
    dig @1.1.1.1 yourdomain.com

  Check TTL:
    dig yourdomain.com | grep TTL

  Force flush DNS cache:
    sudo systemd-resolve --flush-caches   # Linux
    ipconfig /flushdns                    # Windows

Issue: Slow DNS (every request takes 2+ seconds)

  Test DNS response time:
    time dig google.com
  
  Try a faster DNS:
    nameserver 1.1.1.1    (Cloudflare — fastest)
    nameserver 8.8.8.8    (Google)
    nameserver 9.9.9.9    (Quad9 — privacy focused)
```

---

## Popular DNS Servers

| Provider | Primary | Secondary | Notes |
|---|---|---|---|
| Google | 8.8.8.8 | 8.8.4.4 | Fast, widely used |
| Cloudflare | 1.1.1.1 | 1.0.0.1 | Fastest, privacy-focused |
| OpenDNS | 208.67.222.222 | 208.67.220.220 | Filtering options |
| Quad9 | 9.9.9.9 | 149.112.112.112 | Blocks malware domains |
| Your ISP | varies | varies | Default, often slower |

---

## DNS in DevOps — Daily Use

```bash
# Check if your app's domain resolves correctly after deploy
dig myapp.com +short

# Verify a new DNS record has propagated
dig @8.8.8.8 api.myapp.com A

# Check SSL certificate matches DNS (debugging HTTPS)
openssl s_client -connect myapp.com:443 -servername myapp.com 2>/dev/null | openssl x509 -noout -subject

# Test internal DNS in Kubernetes
kubectl run dnstest --image=busybox --rm -it --restart=Never -- nslookup kubernetes.default

# DNS round-robin load balancing check (multiple IPs for one name)
for i in {1..5}; do dig +short myapp.com; done
```
