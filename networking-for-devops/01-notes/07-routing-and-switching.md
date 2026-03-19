# 07 — Routing and Switching

> Every packet you send crosses multiple routers to reach its destination.
> Understanding how this works makes cloud networking, VPCs, and
> debugging "can't reach the server" problems much clearer.

---

## Switching — Within a Network

A switch connects devices within the same network (LAN).
It learns which device is on which port by reading MAC addresses.

```
SWITCH (Layer 2 device)

  Port 1 ── PC-A  (MAC: AA:AA:AA:AA:AA:AA)
  Port 2 ── PC-B  (MAC: BB:BB:BB:BB:BB:BB)
  Port 3 ── PC-C  (MAC: CC:CC:CC:CC:CC:CC)
  Port 4 ── Router

MAC Address Table (the switch learns this automatically):
  Port 1 → AA:AA:AA:AA:AA:AA
  Port 2 → BB:BB:BB:BB:BB:BB
  Port 3 → CC:CC:CC:CC:CC:CC

When PC-A sends data to PC-B:
  Switch checks table → PC-B is on Port 2
  Sends ONLY to Port 2 (not broadcast to all)
```

---

## Routing — Between Networks

A router connects different networks together. It reads IP addresses
(Layer 3) to decide where to forward packets.

```
ANALOGY: The post office highway system

  Your street (192.168.1.0/24)
       ↓ your local router (gateway)
  City highway (your ISP network)
       ↓ ISP router
  National highway (internet backbone)
       ↓ multiple router hops
  Destination city (142.250.80.0/24)
       ↓ Google's router
  Google's server (142.250.80.46)

Each router only knows the NEXT HOP.
It does not know the full path — just "which direction to go".
```

---

## Routing Table

Every router (and your Linux machine) has a routing table.
It maps destination networks to next hops.

```bash
ip route
# default via 192.168.1.1 dev eth0          ← default route (0.0.0.0/0)
# 192.168.1.0/24 dev eth0 proto kernel      ← directly connected network
# 10.0.0.0/8 via 192.168.1.254 dev eth0    ← specific route via gateway

# Reading a routing table entry:
# DESTINATION     VIA (next hop)    INTERFACE
# 192.168.1.0/24  directly          eth0      (no gateway — on same network)
# default         192.168.1.1       eth0      (everything else → router)
# 10.0.0.0/8      192.168.1.254     eth0      (VPN route → VPN gateway)
```

### How a Router Makes a Decision

```
Packet arrives: destination 142.250.80.46

Router checks routing table (longest prefix match):
  10.0.0.0/8     → no match
  192.168.1.0/24 → no match
  default (0.0.0.0/0) → MATCH → forward to ISP router

Longest prefix match = most specific route wins:
  Packet to 10.0.1.5
  10.0.0.0/8   → matches (8-bit prefix)
  10.0.1.0/24  → matches (24-bit prefix) ← MORE SPECIFIC — use this one
```

---

## Default Gateway

The default gateway is where your machine sends packets when it
doesn't know a more specific route. It is your "exit door".

```
Your laptop (192.168.1.10)
    │
    │ doesn't know how to reach 142.250.80.46
    │ routing table says: default → 192.168.1.1
    ▼
Your router (192.168.1.1)
    │
    │ forwards to ISP
    ▼
Internet...
```

```bash
# See default gateway
ip route | grep default
# default via 192.168.1.1 dev eth0

# Add a route manually
sudo ip route add 10.0.0.0/8 via 192.168.1.254
sudo ip route add default via 192.168.1.1

# Delete a route
sudo ip route del 10.0.0.0/8
```

---

## ARP — How IP Maps to MAC

When your machine wants to send to an IP on the same network,
it needs the MAC address. ARP (Address Resolution Protocol) finds it.

```
Your laptop wants to send to 192.168.1.1 (your router)

Step 1: Broadcast to everyone on the LAN:
  "WHO HAS 192.168.1.1? TELL 192.168.1.10"
  (sent to FF:FF:FF:FF:FF:FF — broadcast MAC)

Step 2: The router (192.168.1.1) responds:
  "192.168.1.1 is at AA:BB:CC:DD:EE:FF"

Step 3: Your laptop caches this in its ARP table.
  Now it can send frames directly to AA:BB:CC:DD:EE:FF.
```

```bash
# View ARP table (IP → MAC mappings)
arp -n
ip neigh show

# Clear ARP cache
sudo ip neigh flush all

# ARP only works within a subnet.
# Cross-network communication always goes through the router.
```

---

## Traceroute — Seeing the Path

```bash
traceroute google.com
# traceroute to google.com (142.250.80.46)
#  1  192.168.1.1      2.1 ms    ← your home router
#  2  10.20.0.1        8.3 ms    ← ISP router
#  3  72.14.198.65    12.4 ms    ← ISP backbone
#  4  142.250.80.46   14.2 ms    ← Google's server

# Each line = one router hop
# The number = round trip time (latency)
# * * * = router doesn't respond to ICMP (firewall) — not necessarily broken

mtr google.com      # live traceroute with packet loss stats
```

---

## VLANs — Virtual LANs

A VLAN logically divides one physical switch into multiple
separate networks — without needing separate hardware.

```
PHYSICAL: One switch with 24 ports

LOGICAL (with VLANs):
  VLAN 10 (Engineering): Ports 1-8   → 10.0.10.0/24
  VLAN 20 (Marketing):   Ports 9-16  → 10.0.20.0/24
  VLAN 30 (Servers):     Ports 17-24 → 10.0.30.0/24

  Engineering devices CANNOT communicate with Marketing
  without going through a router (Layer 3).
  This is network segmentation without buying extra switches.
```

---

## BGP — How the Internet Routes

BGP (Border Gateway Protocol) is the protocol that connects
the world's networks (called Autonomous Systems) together.

```
INTERNET = thousands of independent networks (AS)
  AS1 (AT&T)     AS2 (Comcast)    AS3 (Google)
       │               │               │
       └───────────────┴───────────────┘
                  BGP connections

Each AS advertises which IP ranges it owns.
BGP routers choose the best path between ASes.
This is what makes the internet "self-healing" —
if one path breaks, BGP finds another route.
```

---

## Routing in Cloud (AWS Example)

```
VPC 10.0.0.0/16
  │
  ├── Public Subnet 10.0.1.0/24
  │     Route Table:
  │       10.0.0.0/16 → local      (within VPC)
  │       0.0.0.0/0   → igw-xxx    (internet gateway)
  │
  └── Private Subnet 10.0.2.0/24
        Route Table:
          10.0.0.0/16 → local      (within VPC)
          0.0.0.0/0   → nat-xxx    (NAT gateway — outbound only)

Public subnet  → can be reached from internet
Private subnet → can reach internet (via NAT) but not reachable from outside
```

---

## Key Concepts to Remember

| Concept | What it does |
|---|---|
| Switch | Connects devices in the SAME network using MAC addresses |
| Router | Connects DIFFERENT networks using IP addresses |
| Routing table | The router's map of where to send packets |
| Default gateway | Where to send packets with no specific route |
| ARP | Resolves IP address to MAC address within a LAN |
| Traceroute | Shows every router hop to a destination |
| VLAN | Logically separates one switch into multiple networks |
| BGP | The protocol that routes traffic across the internet |
