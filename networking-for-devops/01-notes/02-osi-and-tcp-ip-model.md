# 02 — OSI and TCP/IP Model

> The OSI model is the most important framework in networking.
> It tells you exactly WHERE a problem is happening — which makes
> debugging 10x faster.

---

## Why Models Exist

Networking is incredibly complex. To manage that complexity,
engineers created layered models. Each layer has one job and
only talks to the layers directly above and below it.

```
ANALOGY: Sending a letter through a corporation

  You (the sender)
    ↓  writes the letter           ← Application layer
  Secretary
    ↓  puts it in an envelope      ← Presentation layer
  Mail room
    ↓  adds tracking number        ← Session layer
  Courier company
    ↓  guarantees delivery         ← Transport layer
  Post office
    ↓  adds routing info           ← Network layer
  Sorting facility
    ↓  decides which truck         ← Data Link layer
  Truck driver
    ↓  physically drives it        ← Physical layer
  Recipient
```

Each "department" only knows its own job. The courier doesn't
care what's in the letter. The mail room doesn't care which
truck carries it. This separation is what makes networking modular.

---

## The OSI Model — 7 Layers

```
┌─────┬──────────────┬────────────────────────────────────────────┐
│  7  │ Application  │ What the user/app interacts with           │
│     │              │ HTTP, FTP, DNS, SMTP, SSH                  │
├─────┼──────────────┼────────────────────────────────────────────┤
│  6  │ Presentation │ Data format, encryption, compression       │
│     │              │ TLS/SSL, JPEG, MP4, ASCII, encryption      │
├─────┼──────────────┼────────────────────────────────────────────┤
│  5  │ Session      │ Manages connections (open, maintain, close)│
│     │              │ NetBIOS, RPC, SQL sessions                 │
├─────┼──────────────┼────────────────────────────────────────────┤
│  4  │ Transport    │ Reliable delivery, ports, flow control     │
│     │              │ TCP, UDP                                   │
├─────┼──────────────┼────────────────────────────────────────────┤
│  3  │ Network      │ Logical addressing, routing between nets   │
│     │              │ IP, ICMP, routing protocols (OSPF, BGP)    │
├─────┼──────────────┼────────────────────────────────────────────┤
│  2  │ Data Link    │ Physical addressing, node-to-node delivery │
│     │              │ Ethernet, WiFi, MAC addresses, ARP         │
├─────┼──────────────┼────────────────────────────────────────────┤
│  1  │ Physical     │ Raw bits over a physical medium            │
│     │              │ Cables, fibre, radio waves, voltages       │
└─────┴──────────────┴────────────────────────────────────────────┘

Mnemonic (top to bottom): All People Seem To Need Data Processing
Mnemonic (bottom to top): Please Do Not Throw Sausage Pizza Away
```

---

## What Each Layer Actually Does

### Layer 7 — Application
The layer you interact with as a developer. HTTP, DNS, SMTP, SSH,
FTP all live here. This is where your application code produces
and consumes data.

```
Your browser sends:
GET /index.html HTTP/1.1
Host: google.com
```

### Layer 6 — Presentation
Handles data formatting so both sides understand each other.
Encryption (TLS) happens here. So does compression and encoding.

```
Plain text: "hello"
After TLS:  ╔▓▒░█▓▒╗  (encrypted bytes)
```

In practice, layers 5, 6, and 7 are often handled together by
the application. The OSI model was theoretical — TCP/IP collapsed
these in practice.

### Layer 5 — Session
Manages the lifecycle of a connection — establishing, maintaining,
and terminating sessions. Think of it as the "phone call manager."

### Layer 4 — Transport
**The most important layer for DevOps.** Handles end-to-end delivery.

```
Two main protocols:
  TCP  → Reliable, ordered, connection-based  (like registered mail)
  UDP  → Fast, unreliable, connectionless     (like a postcard)

Adds PORT numbers to identify which application gets the data.
  Source port:      52341  (random, chosen by client)
  Destination port: 80     (HTTP server)
```

### Layer 3 — Network
Handles logical addressing (IP addresses) and routing.
Routers operate at this layer.

```
Adds IP addresses to packets:
  Source IP:      192.168.1.5   (your laptop)
  Destination IP: 142.250.80.46 (google.com)

Routers read Layer 3 headers to decide where to forward packets.
```

### Layer 2 — Data Link
Handles node-to-node delivery within a single network segment.
Switches operate at this layer.

```
Adds MAC addresses:
  Source MAC:      00:1A:2B:3C:4D:5E  (your laptop's network card)
  Destination MAC: FF:EE:DD:CC:BB:AA  (your router's network card)

Important: MAC addresses only matter within a LAN.
They change at every router hop (IP stays the same).
```

### Layer 1 — Physical
Raw bits transmitted over a physical medium.

```
Bit 1 = high voltage (or light pulse, or radio wave)
Bit 0 = low voltage (or no pulse, or no wave)

Media types:
  Copper cable (Cat5e, Cat6)
  Fibre optic
  WiFi (radio waves)
  Bluetooth
```

---

## Data Encapsulation — How Layers Wrap Data

Each layer adds its own header as data travels down the stack.
At the destination, each layer strips its header going back up.

```
SENDER                              RECEIVER

Application   [  DATA  ]            [  DATA  ]   Application
              ↓ add HTTP header      ↑ strip HTTP header

Transport   [TCP|  DATA  ]          [TCP|  DATA  ]   Transport
              ↓ add TCP header       ↑ strip TCP header

Network   [IP|TCP|  DATA  ]         [IP|TCP|  DATA  ]   Network
              ↓ add IP header        ↑ strip IP header

Data Link [ETH|IP|TCP|DATA|FCS]     [ETH|IP|TCP|DATA|FCS]   Data Link
              ↓ add Ethernet header  ↑ strip Ethernet header

Physical  10101011010101010101010    10101011010101010101010   Physical
              ↓ convert to bits      ↑ convert from bits


Each wrapper is called:
  Transport layer  → Segment
  Network layer    → Packet
  Data Link layer  → Frame
  Physical layer   → Bits
```

---

## TCP/IP Model — The Real-World Version

The OSI model is conceptual. The TCP/IP model is what the
internet actually uses. It collapses 7 layers into 4:

```
OSI MODEL          TCP/IP MODEL       PROTOCOLS
─────────          ────────────       ─────────
Application   ┐
Presentation  ├──► Application    HTTP, HTTPS, DNS, SSH, FTP, SMTP
Session       ┘
Transport     ────► Transport     TCP, UDP
Network       ────► Internet      IP, ICMP, ARP
Data Link     ┐
Physical      ├──► Network Access Ethernet, WiFi, MAC
              ┘

The TCP/IP model is what engineers actually use day-to-day.
The OSI model is used for understanding and troubleshooting.
```

---

## Using the OSI Model to Troubleshoot

The real power of the OSI model: when something breaks, you
work from Layer 1 upward until you find the problem.

```
TROUBLESHOOTING FLOWCHART

Can't reach a server?
        │
        ▼
Layer 1: Is the cable plugged in? Is WiFi on?
        │ No → fix physical connection
        │ Yes ↓
Layer 2: Can you ping the default gateway? (ARP working?)
        │ No → switch/VLAN issue
        │ Yes ↓
Layer 3: Can you ping the destination IP?
        │ No → routing issue, check ip route
        │ Yes ↓
Layer 4: Can you connect to the port? (nc -zv host port)
        │ No → firewall blocking, service not running
        │ Yes ↓
Layer 7: Does the application respond correctly?
        │ No → app config issue, check logs
        │ Yes → problem is solved!
```

```bash
# Layer 1/2 check
ip link show                    # is interface UP?
ping 192.168.1.1               # can I reach my gateway?

# Layer 3 check
ping 8.8.8.8                   # can I reach internet by IP?
ip route                       # routing table correct?
traceroute 8.8.8.8             # where does it stop?

# Layer 4 check
nc -zv google.com 443          # is port 443 reachable?
ss -tulnp                      # is the service listening?

# Layer 7 check
curl -v https://google.com     # does the app respond?
```

---

## Where Common Problems Occur

| Problem | Layer | What to check |
|---|---|---|
| No network at all | 1 | Cable, WiFi, NIC |
| Can ping gateway, not internet | 3 | Routing table, default gateway |
| Can ping IP, not hostname | 7 (DNS) | DNS config, /etc/resolv.conf |
| Can reach host, port refused | 4 | Service running? Firewall? |
| Connection times out | 3 or 4 | Firewall dropping packets |
| HTTPS cert error | 6 | TLS certificate invalid/expired |
| Slow connection | 1, 3, or 4 | Bandwidth, packet loss, congestion |

---

## Quick Layer Reference

| Layer | Number | Device | Protocol | Unit |
|---|---|---|---|---|
| Application | 7 | — | HTTP, DNS, SSH | Data |
| Presentation | 6 | — | TLS, JPEG | Data |
| Session | 5 | — | NetBIOS | Data |
| Transport | 4 | — | TCP, UDP | Segment |
| Network | 3 | Router | IP, ICMP | Packet |
| Data Link | 2 | Switch | Ethernet, WiFi | Frame |
| Physical | 1 | Hub, cable | — | Bits |
