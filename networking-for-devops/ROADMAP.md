# Networking for DevOps — Learning Roadmap

> Follow this roadmap in order. Each phase builds the mental model
> before adding more complexity on top.
> Estimated total time: 3–4 weeks at 1–2 hours per day.

---

## The Learning Pyramid

```
                        ┌─────────────────┐
                        │  Cloud Networks  │  ← Phase 4
                        │  VPC, Subnets    │
                   ┌────┴─────────────────┴────┐
                   │  Firewalls · Load Balancers│  ← Phase 3
                   │  VPN · Troubleshooting     │
              ┌────┴────────────────────────────┴────┐
              │   DNS · HTTP/HTTPS · TCP · UDP        │  ← Phase 2
              │   How the internet actually works     │
         ┌────┴──────────────────────────────────────┴────┐
         │   Fundamentals · OSI Model · IP · Subnetting    │  ← Phase 1
         │   The vocabulary and mental model               │
         └──────────────────────────────────────────────────┘

    You cannot skip a layer. Each one is built on the one below.
```

---

## Phase 1 — Build the Foundation (Week 1)

**Goal:** Understand the vocabulary and mental model of networking.
Without this, everything else is just memorising commands with no context.

| Step | File | Time | What you will be able to do |
|---|---|---|---|
| 1 | `01-notes/01-networking-fundamentals.md` | 1.5 hr | Explain what a network is, types of networks, key terms |
| 2 | `01-notes/02-osi-and-tcp-ip-model.md` | 2 hr | Say which layer a problem is at |
| 3 | `01-notes/03-ip-addressing-and-subnetting.md` | 3 hr | Calculate subnets, design a VPC CIDR |

**Phase 1 checkpoint:**
- You can explain what happens when you type `google.com` in a browser (at a high level)
- You can calculate how many hosts fit in a `/24` subnet
- You can explain the difference between a private and public IP

---

## Phase 2 — How the Internet Works (Week 2)

**Goal:** Understand the protocols that power every app you deploy.

| Step | File | Time | What you will be able to do |
|---|---|---|---|
| 4 | `01-notes/04-dns.md` | 2 hr | Trace a DNS query, configure records, debug DNS issues |
| 5 | `01-notes/05-http-and-https.md` | 2 hr | Read HTTP headers, understand TLS handshake, debug 4xx/5xx |
| 6 | `01-notes/06-tcp-and-udp.md` | 1.5 hr | Explain TCP 3-way handshake, when to use UDP |
| 7 | `01-notes/07-routing-and-switching.md` | 1.5 hr | Explain how a packet finds its destination |

**Phase 2 checkpoint:**
- You can fully answer "what happens when you type google.com in a browser"
- You can read and interpret HTTP response headers
- You can explain what a TCP SYN flood is and why it works

---

## Phase 3 — DevOps-Specific Networking (Week 3)

**Goal:** Apply networking knowledge to real DevOps tasks.

| Step | File | Time | What you will be able to do |
|---|---|---|---|
| 8 | `01-notes/08-firewalls-and-security-groups.md` | 2 hr | Write firewall rules, configure cloud security groups |
| 9 | `01-notes/09-load-balancing.md` | 1.5 hr | Set up and explain L4 vs L7 load balancing |
| 10 | `01-notes/10-vpn-and-tunneling.md` | 1.5 hr | Explain VPN types, set up a basic tunnel |
| 11 | `01-notes/11-network-troubleshooting.md` | 2 hr | Diagnose any network problem systematically |

**Phase 3 checkpoint:**
- You can configure AWS security groups for a 3-tier app
- You can explain the difference between L4 and L7 load balancing
- You can diagnose a "can't connect to server" issue in under 5 minutes

---

## Phase 4 — Cloud Networking (Week 4)

**Goal:** Design and manage networks in cloud environments.

| Step | File | Time | What you will be able to do |
|---|---|---|---|
| 12 | `01-notes/12-networking-in-cloud.md` | 3 hr | Design a VPC, set up subnets, NAT gateways, peering |

**Phase 4 checkpoint:**
- You can design a production-ready VPC with public and private subnets
- You can explain the difference between a NAT gateway and an internet gateway
- You can set up VPC peering between two networks

---

## Topic Dependency Map

```
networking-fundamentals
        │
        ├──► osi-and-tcp-ip-model
        │           │
        │           ├──► ip-addressing-and-subnetting ──► networking-in-cloud
        │           │
        │           ├──► tcp-and-udp ──► load-balancing
        │           │
        │           └──► routing-and-switching ──► vpn-and-tunneling
        │
        ├──► dns ──────────────────────────────────► network-troubleshooting
        │
        └──► http-and-https ──► firewalls-and-security-groups
```

---

## The "What Happens When You Type google.com" Answer

This is the most famous interview question in networking.
By the end of this repo you should be able to answer it at every layer:

```
Phase 1 finish:  "DNS resolves the name, TCP connects, HTTP fetches the page"
Phase 2 finish:  Full answer with DNS recursion, TCP handshake, TLS, HTTP/2
Phase 3 finish:  Add: firewall rules checked, load balancer routes the request
Phase 4 finish:  Add: VPC routing, security groups, NAT gateway for private subnets
```

---

## Time Estimates

| Your situation | Time to complete |
|---|---|
| No networking background | 5–6 weeks |
| Studied networking before | 2–3 weeks |
| Refreshing for interviews | 3–5 days |
| Cloud-specific only (notes 8–12) | 1 week |
