# Networking for DevOps

> A structured networking repo built specifically for DevOps engineers.
> Concepts · Diagrams · Analogies · Labs · Interview Prep — all in one place.

---

## Why Networking for DevOps?

Every time you deploy an app, configure a cloud environment, debug a
production issue, or set up a CI/CD pipeline — you are doing networking.
Containers talk to each other over networks. Load balancers distribute
traffic over networks. DNS resolves your domain over a network. VPNs
connect your office to your cloud over a network.

> "You can be a great coder and still get completely lost when your
> app can't reach the database. Networking knowledge is what turns
> a developer into a DevOps engineer."

---

## The Networking Mental Model for DevOps

```
THE POSTAL SYSTEM ANALOGY
══════════════════════════════════════════════════════════════════

  Networking concept        Real world equivalent
  ─────────────────         ──────────────────────────────────
  IP Address                Your house's street address
  Port                      The specific room in the house
  DNS                       The phonebook (name → address)
  Router                    The post office sorting facility
  Firewall                  The security guard at the gate
  Load Balancer             A receptionist who routes calls
  VPN                       A private underground tunnel
  Packet                    A single envelope in the mail
  Protocol                  The rules for writing addresses


THE FOUR QUESTIONS YOU ASK WHEN NETWORKING BREAKS
══════════════════════════════════════════════════════════════════

  1. Can I reach it?          → ping, traceroute
  2. Is the port open?        → nc, curl, telnet
  3. Is DNS resolving?        → dig, nslookup
  4. Is traffic being blocked?→ firewall rules, security groups
```

---

## What This Repo Covers

| # | Topic | Why It Matters for DevOps |
|---|---|---|
| 01 | Networking Fundamentals | Vocabulary every engineer must know |
| 02 | OSI and TCP/IP Model | Understand where problems occur |
| 03 | IP Addressing and Subnetting | Design cloud VPCs and networks |
| 04 | DNS | How your domain actually works |
| 05 | HTTP and HTTPS | Every web app runs on this |
| 06 | TCP and UDP | Why some protocols are reliable and others aren't |
| 07 | Routing and Switching | How packets find their destination |
| 08 | Firewalls and Security Groups | Control what traffic gets through |
| 09 | Load Balancing | Scale apps and eliminate single points of failure |
| 10 | VPN and Tunneling | Connect environments securely |
| 11 | Network Troubleshooting | Diagnose and fix network problems fast |
| 12 | Networking in the Cloud | VPCs, subnets, security groups in AWS/GCP/Azure |

---

## Repo Structure

```
networking-for-devops/
│
├── 01-notes/               ← 12 topic files with diagrams and analogies
├── 02-cheatsheet/          ← all commands and concepts one-page reference
├── 03-hands-on-labs/
│   ├── online-platforms.md ← practice without any setup
│   └── local-tasks/        ← hands-on exercises on your machine
├── 04-projects/
│   ├── beginner/           ← DNS resolver, port scanner
│   └── intermediate/       ← network monitor, load balancer setup
├── 05-interview-prep/      ← Q&A by topic
├── 06-resources/           ← books, courses, tools, YouTube
└── 07-troubleshooting/     ← real scenarios with fixes
```

---

## How to Use This Repo

**If you are new to networking:**
Start at `01-notes/01-networking-fundamentals.md` and go in order.
Every note file has analogies and ASCII diagrams — read them carefully,
they are designed to build intuition not just facts.

**If you know basics but want DevOps-specific knowledge:**
Jump to notes 08–12 — firewalls, load balancing, VPNs, cloud networking.

**If you are debugging a network issue right now:**
Go directly to `07-troubleshooting/` — it has step-by-step guides.

**If you are preparing for interviews:**
`05-interview-prep/` is organised by topic with answers.

---

## Prerequisites

- Completed `linux-for-devops` (or comfortable with Linux terminal)
- Access to a Linux machine (local, WSL2, or cloud VM)
- Basic comfort running commands in a terminal

---

## Progress Tracker

- [ ] 01 Networking Fundamentals
- [ ] 02 OSI and TCP/IP Model
- [ ] 03 IP Addressing and Subnetting
- [ ] 04 DNS
- [ ] 05 HTTP and HTTPS
- [ ] 06 TCP and UDP
- [ ] 07 Routing and Switching
- [ ] 08 Firewalls and Security Groups
- [ ] 09 Load Balancing
- [ ] 10 VPN and Tunneling
- [ ] 11 Network Troubleshooting
- [ ] 12 Networking in the Cloud
- [ ] All lab tasks completed
- [ ] At least one project built
- [ ] Interview prep reviewed

---

## Contributing

See [CONTRIBUTING.md](./CONTRIBUTING.md) — corrections, new labs,
and resource links are all welcome.

---

*Part of the [DevOps Mastery](https://github.com/yourusername) learning series.*
