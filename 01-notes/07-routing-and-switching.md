# 07 — Routing & Switching

## Switching (Layer 2)

A **switch** forwards data by **MAC address** within the same network segment.

```
Device A ──► [Switch] ──► Device B
              │
              └─ Learns: "Device A is on port 3, MAC = AA:BB:CC:DD:EE:FF"
```

### MAC Address Table (CAM Table)
- Switch maintains a table: `MAC → Port`
- Learned dynamically by observing source MACs
- **Unknown unicast**: Flooded to all ports until MAC is learned

### VLANs (Virtual LANs)
- Logically segments a switch into multiple isolated networks
- Devices on VLAN 10 cannot talk to VLAN 20 without a **router**
- Tagged with `802.1Q` standard

```bash
# View VLAN config on a Linux bridge
bridge vlan show
```

---

## Routing (Layer 3)

A **router** forwards packets by **IP address** across different networks using a **routing table**.

```bash
# View routing table on Linux
ip route show
route -n
```

Example routing table:
```
Destination     Gateway         Iface
0.0.0.0/0       192.168.1.1     eth0    ← Default route (internet)
10.0.0.0/8      10.10.0.1       eth1    ← Internal network
192.168.1.0/24  0.0.0.0         eth0    ← Directly connected
```

### How a Router Forwards a Packet
1. Look up **destination IP** in routing table
2. Match the **longest prefix** (most specific route wins)
3. Forward to the **next hop** (gateway)

---

## Types of Routes

| Type | Description |
|---|---|
| **Connected** | Directly attached networks |
| **Static** | Manually configured |
| **Dynamic** | Learned via routing protocols |
| **Default** | 0.0.0.0/0 — used when no match found |

---

## Routing Protocols

### Interior Gateway Protocols (within an organization)

| Protocol | Type | Metric | Use |
|---|---|---|---|
| **OSPF** | Link-state | Cost | Enterprise networks |
| **EIGRP** | Hybrid | Composite | Cisco-only |
| **RIP** | Distance-vector | Hop count | Legacy, small networks |

### Exterior Gateway Protocol (between organizations)

| Protocol | Type | Use |
|---|---|---|
| **BGP** | Path-vector | Internet routing, cloud |

### BGP (Border Gateway Protocol)
- The protocol of the internet — how ISPs and cloud providers exchange routes
- In cloud: used in **AWS Direct Connect**, **VPN BGP sessions**, **GCP Interconnect**

---

## ARP — Address Resolution Protocol

ARP maps **IP address → MAC address** on a local network.

```
"Who has 192.168.1.5? Tell 192.168.1.1"
→ Device with .1.5 replies: "I'm AA:BB:CC:DD:EE:FF"
```

```bash
# View ARP cache
arp -n
ip neigh show

# Send ARP request
arping -I eth0 192.168.1.1
```

---

## Static Routes in Linux

```bash
# Add a static route
sudo ip route add 10.10.0.0/24 via 192.168.1.1

# Add default route
sudo ip route add default via 192.168.1.1

# Delete a route
sudo ip route del 10.10.0.0/24

# Make persistent (add to /etc/network/interfaces or Netplan)
```

---

## DevOps Relevance

```bash
# Trace the route a packet takes
traceroute google.com
mtr google.com          # live traceroute

# Check if routing table has a route to a destination
ip route get 8.8.8.8

# AWS VPC: Route tables define where traffic goes
# - 0.0.0.0/0 → Internet Gateway (public subnet)
# - 0.0.0.0/0 → NAT Gateway (private subnet)
# - 10.0.0.0/8 → Transit Gateway (multi-VPC routing)
```
