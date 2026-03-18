# 02 — OSI Model & TCP/IP Model

## The OSI Model

The **Open Systems Interconnection (OSI)** model is a conceptual framework that standardizes network communication into **7 layers**.

```
┌───────────────────────────────────────┐
│  7 - Application   (HTTP, DNS, SMTP)  │
├───────────────────────────────────────┤
│  6 - Presentation  (TLS, JPEG, ASCII) │
├───────────────────────────────────────┤
│  5 - Session       (NetBIOS, RPC)     │
├───────────────────────────────────────┤
│  4 - Transport     (TCP, UDP)         │
├───────────────────────────────────────┤
│  3 - Network       (IP, ICMP, BGP)   │
├───────────────────────────────────────┤
│  2 - Data Link     (Ethernet, MAC)    │
├───────────────────────────────────────┤
│  1 - Physical      (Cables, Wi-Fi)    │
└───────────────────────────────────────┘
```

### Memory Trick (Bottom to Top)
> **P**lease **D**o **N**ot **T**hrow **S**ausage **P**izza **A**way

---

## Layer-by-Layer Breakdown

### Layer 1 — Physical
- Transmits raw bits over a physical medium
- Cables (Ethernet, fiber), Wi-Fi radio signals
- **Units**: Bits

### Layer 2 — Data Link
- Node-to-node data transfer (on the same network segment)
- Uses **MAC addresses**
- Ethernet frames, VLANs, switches operate here
- **Units**: Frames

### Layer 3 — Network
- Logical addressing and routing between different networks
- Uses **IP addresses**
- Routers operate here
- **Units**: Packets

### Layer 4 — Transport
- End-to-end communication, segmentation, flow control
- **TCP** (reliable) and **UDP** (fast) live here
- Uses **ports**
- **Units**: Segments (TCP) / Datagrams (UDP)

### Layer 5 — Session
- Establishes, manages, and terminates sessions
- Authentication, reconnection

### Layer 6 — Presentation
- Data translation, encryption, compression
- TLS/SSL encryption happens here

### Layer 7 — Application
- Closest layer to the user
- HTTP, HTTPS, FTP, DNS, SMTP, gRPC

---

## TCP/IP Model (Practical Model)

The **TCP/IP model** is what real networks actually use. It condenses OSI into **4 layers**.

```
┌──────────────────────────────────────────────┐
│  Application   →  OSI Layers 5, 6, 7         │
│  (HTTP, DNS, TLS, FTP, SMTP)                 │
├──────────────────────────────────────────────┤
│  Transport     →  OSI Layer 4                │
│  (TCP, UDP)                                  │
├──────────────────────────────────────────────┤
│  Internet      →  OSI Layer 3                │
│  (IP, ICMP, ARP)                            │
├──────────────────────────────────────────────┤
│  Network Access → OSI Layers 1 & 2           │
│  (Ethernet, Wi-Fi, MAC)                      │
└──────────────────────────────────────────────┘
```

---

## OSI vs TCP/IP

| OSI Layer | TCP/IP Layer | Example Protocols |
|---|---|---|
| Application (7) | Application | HTTP, HTTPS, DNS, SMTP |
| Presentation (6) | Application | TLS, JPEG, ASCII |
| Session (5) | Application | NetBIOS, RPC |
| Transport (4) | Transport | TCP, UDP |
| Network (3) | Internet | IP, ICMP, ARP |
| Data Link (2) | Network Access | Ethernet, 802.11 |
| Physical (1) | Network Access | Cables, radio waves |

---

## Data Encapsulation

As data flows **down** the stack, each layer wraps (encapsulates) the data with its own header.

```
Application:     [Data]
Transport:       [TCP Header | Data]
Network:         [IP Header | TCP Header | Data]
Data Link:       [Eth Header | IP Header | TCP Header | Data | Eth Trailer]
Physical:        01001010110100...
```

On the receiving end, each layer **strips** its header and passes the payload up — this is **de-encapsulation**.

---

## DevOps Relevance

| Scenario | OSI Layer |
|---|---|
| Debugging a broken HTTP API | Layer 7 |
| Diagnosing TLS certificate errors | Layer 6 |
| Checking if a port is open | Layer 4 |
| Tracing packet routing issues | Layer 3 |
| VLAN or MAC address issues | Layer 2 |
| Cable-related outages | Layer 1 |

> When troubleshooting, always start at **Layer 1** and work your way up.
