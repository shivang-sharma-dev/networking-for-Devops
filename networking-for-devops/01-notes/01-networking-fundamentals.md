# 01 — Networking Fundamentals

> Before touching any command or protocol, build the right mental model.
> Every networking concept in this repo builds on what's in this file.

---

## What is a Network?

A network is simply two or more devices connected together so they can
share data. That's it. Everything else — protocols, IP addresses, DNS,
firewalls — is just solving the problems that come with making that
sharing reliable, fast, and secure.

```
SIMPLEST POSSIBLE NETWORK

  Computer A ────────────── Computer B
             (cable or WiFi)

  Problem: works for 2 devices.
  What about 1000 devices?
  What about devices in different cities?
  What about sending data reliably?
  → These problems created all of networking.
```

---

## Types of Networks

```
┌─────────────────────────────────────────────────────────────┐
│                         WAN                                  │
│            (Wide Area Network — the Internet)                │
│                                                              │
│   ┌──────────────────┐        ┌──────────────────┐          │
│   │       LAN        │        │       LAN        │          │
│   │  (your office)   │        │  (AWS datacenter)│          │
│   │                  │        │                  │          │
│   │  PC ── Switch    │        │  Server cluster  │          │
│   │  PC ──┘          │        │                  │          │
│   │  Printer         │        │                  │          │
│   └────────┬─────────┘        └────────┬─────────┘          │
│            │                           │                     │
│         Router                      Router                   │
│            └───────────── WAN ────────┘                     │
└─────────────────────────────────────────────────────────────┘
```

| Type | Stands for | Coverage | Example |
|---|---|---|---|
| LAN | Local Area Network | One building / office | Your home WiFi |
| WAN | Wide Area Network | Cities / countries | The Internet |
| MAN | Metropolitan Area Network | A city | City-wide fibre |
| VPN | Virtual Private Network | Anywhere | Secure tunnel over internet |
| VLAN | Virtual LAN | Logical, within a switch | Isolating teams in one office |

---

## The Postal System Analogy

This analogy maps every networking concept to something familiar.
Come back to this whenever a concept feels abstract.

```
NETWORKING          POSTAL SYSTEM
══════════          ═════════════
IP Address    ←→   Street address of a house
Port          ←→   Specific room/department in that house
              │     (port 80 = front door, port 22 = back door)
MAC Address   ←→   The unique serial number stamped on the house
Packet        ←→   One envelope in the mail
Protocol      ←→   Rules for how to write the address on the envelope
Router        ←→   Post office sorting facility
Switch        ←→   The mailroom inside one building
Firewall      ←→   Security guard who checks IDs at the gate
DNS           ←→   The phonebook (google.com → 142.250.80.46)
ISP           ←→   The postal service company
Gateway       ←→   The exit point from your neighbourhood to the highway
```

---

## Key Networking Terms

### Bandwidth vs Latency

These are two completely different things that people often confuse:

```
BANDWIDTH = how wide the pipe is (how much data per second)
LATENCY   = how long a trip takes (time from A to B)

Think of a highway:
  Bandwidth = number of lanes (10 lanes = more cars at once)
  Latency   = speed limit     (100 km/h = faster trip)

A wide highway with a low speed limit = high bandwidth, high latency
A narrow road with a high speed limit = low bandwidth, low latency

For video streaming  → bandwidth matters (lots of data)
For online gaming    → latency matters  (fast response)
For a database query → latency matters  (fast response)
```

### Packet

Data on a network is never sent as one big chunk. It is broken into
small pieces called **packets** (typically 1500 bytes each).

```
You send a 10MB file:

  ┌─────────────────────────────────┐
  │           10MB file             │
  └─────────────────────────────────┘
                  │
                  ▼  broken into packets
  ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐
  │ Pkt1 │ │ Pkt2 │ │ Pkt3 │ │ Pkt4 │ │ Pkt5 │  ... (6666 packets)
  └──────┘ └──────┘ └──────┘ └──────┘ └──────┘
      │        │        │        │        │
      └────────┴────────┴────────┴────────┘
                  different routes
                  reassembled at destination

Why packets?
  - If one gets lost, only that packet is resent (not the whole file)
  - Multiple conversations can share the same wire
  - Traffic can take different routes around failures
```

Each packet has a **header** (metadata: source IP, destination IP,
packet number) and a **payload** (the actual data).

### Protocol

A protocol is an agreed set of rules for communication. Both sides
must speak the same protocol or they cannot understand each other.

```
Analogy: language

  If you speak English and I speak French, we cannot communicate.
  We need to agree on a common language first.

  HTTP  = rules for web communication
  TCP   = rules for reliable delivery
  DNS   = rules for name resolution
  SSH   = rules for secure remote access
  SMTP  = rules for email
```

### Client and Server

```
CLIENT                          SERVER
──────                          ──────
Makes requests              Responds to requests
Your browser                nginx, Apache
curl command                Your Flask app
Your phone                  Instagram's servers
MySQL client                MySQL database

  Client ──── "GET /index.html" ────► Server
  Client ◄──── "200 OK + HTML" ────── Server

The same machine can be both:
  A web server is a CLIENT when it queries a database
  A database server is the SERVER in that exchange
```

### Port

An IP address identifies a machine. A port identifies a specific
service running on that machine.

```
YOUR SERVER (IP: 192.168.1.10)
┌─────────────────────────────────────┐
│  Port 22   → SSH server             │
│  Port 80   → nginx (HTTP)           │
│  Port 443  → nginx (HTTPS)          │
│  Port 3306 → MySQL                  │
│  Port 5432 → PostgreSQL             │
│  Port 6379 → Redis                  │
│  Port 8080 → your app               │
└─────────────────────────────────────┘

Full address = IP + Port
192.168.1.10:80   → the nginx web server on this machine
192.168.1.10:3306 → the MySQL database on this machine
```

### Well-known ports

| Port | Protocol | Service |
|---|---|---|
| 22 | TCP | SSH |
| 25 | TCP | SMTP (email sending) |
| 53 | TCP/UDP | DNS |
| 80 | TCP | HTTP |
| 443 | TCP | HTTPS |
| 3306 | TCP | MySQL |
| 5432 | TCP | PostgreSQL |
| 6379 | TCP | Redis |
| 27017 | TCP | MongoDB |
| 2181 | TCP | Zookeeper |
| 9092 | TCP | Kafka |

Ports 0–1023 = **well-known ports** (require root to bind)
Ports 1024–49151 = **registered ports** (apps use these)
Ports 49152–65535 = **ephemeral ports** (clients use these temporarily)

---

## Network Devices

```
┌───────────────────────────────────────────────────────────┐
│                                                           │
│   HUB         Broadcasts every packet to ALL devices.    │
│   (old)       Dumb. No longer used.                       │
│                                                           │
│   SWITCH      Sends packets only to the correct device.  │
│   (smart hub) Learns MAC addresses. Used in LANs.         │
│                                                           │
│   ROUTER      Connects different networks together.       │
│               Decides the best path for packets.          │
│               Your home router = switch + router + WiFi   │
│                                                           │
│   FIREWALL    Filters traffic based on rules.             │
│               Allows or blocks based on IP, port,         │
│               protocol, direction.                        │
│                                                           │
│   LOAD        Distributes incoming traffic across         │
│   BALANCER    multiple servers.                           │
│                                                           │
│   GATEWAY     The exit point from one network to another. │
│               Your home router is your default gateway.   │
│                                                           │
└───────────────────────────────────────────────────────────┘
```

---

## MAC Address vs IP Address

```
MAC ADDRESS                     IP ADDRESS
───────────────                 ──────────────────
48-bit hardware address         32-bit (IPv4) logical address
Burned into network card        Assigned by network/admin
Never changes                   Changes when you move networks
Used within a LAN               Used across the internet
Example: 00:1A:2B:3C:4D:5E      Example: 192.168.1.10

Analogy:
  MAC = your national ID number (permanent, unique to you)
  IP  = your current home address (changes when you move)
```

---

## IPv4 vs IPv6

```
IPv4                            IPv6
────────────────                ────────────────────────────────
32-bit address                  128-bit address
4.3 billion addresses           340 undecillion addresses
192.168.1.10                    2001:0db8:85a3::8a2e:0370:7334
Running out                     Designed to never run out
Still dominant                  Adoption growing
```

We are running out of IPv4 addresses. Solutions:
- **NAT** (Network Address Translation) — many devices share one public IP
- **IPv6** — long-term solution, massively more addresses

---

## Unicast, Broadcast, Multicast

```
UNICAST     One sender → One specific receiver
            (most traffic: HTTP, SSH, DNS responses)

BROADCAST   One sender → ALL devices on the network
            (ARP requests: "who has IP 192.168.1.10?")

MULTICAST   One sender → A specific GROUP of receivers
            (video streaming to subscribers, routing protocols)
```

---

## Key Takeaways

| Concept | Remember it as |
|---|---|
| IP Address | The address of a house |
| Port | The room inside the house |
| Packet | One envelope in the mail |
| Protocol | The language both sides agree to speak |
| Router | The post office sorting facility |
| Switch | The mailroom inside one building |
| Bandwidth | Width of the pipe (data per second) |
| Latency | Speed of the trip (time to get there) |
| MAC Address | Permanent hardware ID of a device |
| Gateway | The exit door from your network |
