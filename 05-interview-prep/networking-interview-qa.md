# 🎤 Interview Prep — Networking for DevOps

## Core Concept Questions

### DNS

**Q: Explain the full flow of a browser making a request to google.com**

> 1. Browser checks local DNS cache
> 2. Asks OS resolver (checks hosts file, then configured DNS)
> 3. Recursive resolver checks its own cache
> 4. Queries root nameserver `(.)` → returns `.com` TLD NS
> 5. Queries `.com` TLD NS → returns Google's authoritative NS
> 6. Queries Google's authoritative NS → returns `142.250.x.x`
> 7. Browser connects to that IP on port 443 via TCP + TLS

**Q: What's the difference between CNAME and A record?**

> - **A record**: Maps a domain directly to an IPv4 address. Faster, one lookup.
> - **CNAME**: Alias that points to another domain name (requires one more lookup).
> Use CNAME for flexibility (CDN, LBs); A record for direct IP mappings.

---

### TCP/UDP

**Q: Why is TCP slower than UDP? When would you use UDP?**

> TCP adds reliability overhead: 3-way handshake, sequence numbers, ACKs, retransmissions.
> Use UDP for: DNS lookups, video streaming, VoIP, gaming — anything where speed matters more than guaranteed delivery.

**Q: What happens during a TCP 3-way handshake?**

> 1. Client sends **SYN** (seq=x)
> 2. Server replies with **SYN-ACK** (seq=y, ack=x+1)
> 3. Client sends **ACK** (ack=y+1)
> Connection is established, data can flow.

---

### Firewalls

**Q: What's the difference between Security Groups and NACLs in AWS?**

> | | Security Group | NACL |
> |---|---|---|
> | Level | Instance | Subnet |
> | Stateful | Yes | No |
> | Allow/Deny | Allow only | Both |
> | Rule order | All evaluated | Numbered order |

**Q: What is iptables and how does it work?**

> `iptables` is a Linux firewall using tables and chains. The `filter` table has INPUT, OUTPUT, FORWARD chains. Rules match packets by protocol, port, IP, state and apply actions (ACCEPT, DROP, REJECT). Rules are evaluated top-to-bottom, first match wins.

---

### Subnetting

**Q: How many usable hosts does a /27 subnet have?**

> 2^(32-27) - 2 = 2^5 - 2 = 32 - 2 = **30 usable hosts**

**Q: You need to allocate a subnet for a point-to-point VPN link. What CIDR would you use?**

> `/30` — gives 4 IPs (2 usable), perfect for two endpoints.

---

### Load Balancing

**Q: What's the difference between L4 and L7 load balancers?**

> - **L4**: Operates at TCP/UDP. Routes based on IP/port. Very fast, doesn't inspect HTTP content.
> - **L7**: Operates at HTTP. Can route by URL path, headers, cookies. Supports TLS termination, sticky sessions, smarter routing.

**Q: How does a load balancer handle a backend server going down?**

> Health checks (HTTP or TCP) periodically probe each server. If a check fails N times consecutively, the server is removed from the pool. Traffic is routed to healthy servers. When the server recovers, it's added back.

---

### Cloud Networking

**Q: What is a VPC and what components make up a typical VPC?**

> A VPC (Virtual Private Cloud) is an isolated virtual network in cloud. Typical components:
> - Subnets (public and private)
> - Internet Gateway (IGW) for public internet
> - NAT Gateway for private subnet outbound access
> - Route Tables to control traffic flow
> - Security Groups and NACLs for firewall rules
> - VPC Endpoints for private access to cloud services

**Q: What's the difference between NAT Gateway and Internet Gateway in AWS?**

> - **IGW**: Bidirectional gateway — allows both inbound and outbound internet traffic. Used by public subnets.
> - **NAT Gateway**: Outbound-only — allows private subnet resources to initiate outbound connections but blocks all inbound initiations.

---

## Scenario-Based Questions

**Q: Your microservice can't connect to the database. How do you debug?**

> 1. Check DNS — can the service resolve the DB hostname? (`nslookup db-hostname`)
> 2. Check network connectivity — is the port reachable? (`nc -zv db-host 5432`)
> 3. Check firewall — are security groups / NACLs allowing TCP/5432?
> 4. Check if DB is actually listening (`ss -tlnp | grep 5432` on DB host)
> 5. Check DB user permissions
> 6. Review app logs and DB logs

**Q: Traffic to your app is suddenly slow. How do you investigate?**

> 1. Check latency: `mtr api.example.com` (identify where loss/latency starts)
> 2. Check DNS: `time dig api.example.com` (is DNS slow?)
> 3. Check timeouts: `curl -w "connect:%{time_connect} TTFB:%{time_starttransfer}\n"`
> 4. Check LB metrics — is one backend slow?
> 5. Check CPU/memory on backend servers
> 6. Check for packet loss or network saturation

**Q: How would you design a highly available 3-tier web application VPC?**

> ```
> VPC: 10.0.0.0/16, 3 AZs
>
> Public subnets (one per AZ):
>   - ALB / NLB
>   - NAT Gateways (one per AZ for HA)
>
> Private subnets - App tier (one per AZ):
>   - EC2 in Auto Scaling Group
>   - Private only, outbound via NAT
>
> Private subnets - DB tier (one per AZ):
>   - RDS Multi-AZ
>   - Security Group: only allow app SG
> ```

---

## Quick-Fire Q&A

| Question | Answer |
|---|---|
| Default VPC CIDR in AWS? | `172.31.0.0/16` |
| What port does HTTPS use? | 443 |
| What does TTL stand for in DNS? | Time to Live |
| What tool do you use to see ARP cache? | `ip neigh show` or `arp -n` |
| How many bits in IPv6? | 128 |
| What is mTLS? | Both client AND server present certificates |
| What is a CDN? | Content Delivery Network — caches assets close to users |
| What is BGP? | Border Gateway Protocol — how internet routing works between ASes |
| What is the loopback address? | 127.0.0.1 (IPv4) / ::1 (IPv6) |
| What OSI layer does a router operate at? | Layer 3 |
