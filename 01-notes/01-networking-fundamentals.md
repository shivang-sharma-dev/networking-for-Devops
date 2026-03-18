# 01 — Networking Fundamentals

## What is a Network?

A **network** is a collection of devices (computers, servers, phones, routers) connected together to share data and resources.

---

## Key Concepts

### Nodes & Hosts
- **Node**: Any device connected to a network
- **Host**: A node that sends or receives data (servers, clients)

### Bandwidth vs Throughput vs Latency

| Term | Definition | Example |
|---|---|---|
| **Bandwidth** | Maximum capacity of a link | 1 Gbps fiber |
| **Throughput** | Actual data transferred per second | 750 Mbps under load |
| **Latency** | Time for a packet to travel A → B | 20ms ping |
| **Jitter** | Variation in latency | ±5ms |

---

## Types of Networks

| Type | Full Name | Scope |
|---|---|---|
| LAN | Local Area Network | Office / Home |
| WAN | Wide Area Network | Country / World |
| MAN | Metropolitan Area Network | City |
| VPN | Virtual Private Network | Tunnel over internet |
| SDN | Software Defined Network | Virtualized control plane |

---

## Network Topologies

```
Star:          Ring:         Mesh:
  [S]          A—B           A—B
 / | \         |  |         /|  |\
A  B  C        D—C         D-+--+-C
                           |      |
                           E——————F
```

- **Star** — all devices connect to a central switch/hub (most common in LAN)
- **Ring** — each device connects to two neighbors (older, used in some WANs)
- **Mesh** — every device connects to every other (used in SD-WAN, IoT)
- **Bus** — all devices share a single cable (legacy)

---

## Network Devices

| Device | Layer | Function |
|---|---|---|
| **Hub** | L1 | Broadcasts to all ports (dumb) |
| **Switch** | L2 | Forwards by MAC address |
| **Router** | L3 | Forwards by IP address |
| **Firewall** | L3/L4 | Filters traffic by rules |
| **Load Balancer** | L4/L7 | Distributes traffic across servers |
| **Proxy** | L7 | Sits between client and server |

---

## The Client-Server Model

```
Client                    Server
  │──── Request (HTTP) ────►│
  │◄─── Response (200 OK) ──│
```

- **Client**: Initiates the request (browser, curl, mobile app)
- **Server**: Listens and responds (web server, API, database)

---

## Ports & Sockets

- A **port** is a 16-bit number (0–65535) that identifies a specific process on a host
- A **socket** is the combination of `IP:Port` (e.g., `192.168.1.10:443`)

### Common Ports to Know

| Port | Protocol | Service |
|---|---|---|
| 22 | TCP | SSH |
| 25 | TCP | SMTP |
| 53 | UDP/TCP | DNS |
| 80 | TCP | HTTP |
| 443 | TCP | HTTPS |
| 3306 | TCP | MySQL |
| 5432 | TCP | PostgreSQL |
| 6379 | TCP | Redis |
| 8080 | TCP | HTTP alt / dev server |
| 27017 | TCP | MongoDB |

---

## Protocols

A **protocol** is a set of rules that define how data is formatted, transmitted, and received.

- **HTTP** — web communication
- **TCP** — reliable transport
- **UDP** — fast, unreliable transport
- **IP** — addressing & routing
- **DNS** — domain name resolution
- **TLS** — encryption

---

## DevOps Relevance

- You need to understand ports to write correct **firewall rules** and **security groups**
- Latency awareness is critical for **microservices** and distributed systems
- Understanding the **client-server model** helps you debug application connectivity issues
