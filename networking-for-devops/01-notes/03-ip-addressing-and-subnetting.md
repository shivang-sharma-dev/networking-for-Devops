# 03 — IP Addressing and Subnetting

> Subnetting is the most math-heavy topic in networking.
> This file breaks it down with analogies and a repeatable method
> so you never have to guess.

---

## What is an IP Address?

An IPv4 address is a 32-bit number written as four decimal
numbers separated by dots. Each number is called an **octet**
(8 bits, range 0–255).

```
192    .    168    .    1    .    10
 │           │         │         │
 └─octet 1   └─octet 2 └─octet 3 └─octet 4

In binary:
11000000 . 10101000 . 00000001 . 00001010

Every IP address = 32 bits = 4 octets = 4 numbers (0-255)
```

---

## Network Part vs Host Part

Every IP address has two parts:

```
ANALOGY: A street address

  "221B Baker Street, London"
   ─────────────────  ──────
   House number       City/Street (network)

  IP Address: 192.168.1.10
               ─────────── ──
               Network part  Host part
               (the street)  (the house number)

The subnet mask tells you WHERE the split is.
```

---

## Subnet Mask

A subnet mask defines which part of the IP is the network
and which part is the host.

```
IP Address:   192.168.1.10
Subnet Mask:  255.255.255.0

In binary:
IP:    11000000.10101000.00000001.00001010
Mask:  11111111.11111111.11111111.00000000
       ├────────────────────────┤├────────┤
              Network part        Host part

Where mask = 1 → network part
Where mask = 0 → host part
```

---

## CIDR Notation — The Shorthand

Writing out `255.255.255.0` is verbose. CIDR notation counts
the number of 1-bits in the subnet mask.

```
255.255.255.0   =  /24  (24 ones in the mask)
255.255.0.0     =  /16  (16 ones)
255.0.0.0       =  /8   (8 ones)
255.255.255.128 =  /25  (25 ones)

192.168.1.0/24 means:
  IP address:  192.168.1.0
  Subnet mask: 255.255.255.0 (24 ones)
  Network:     192.168.1.x   (first 24 bits fixed)
  Hosts:       192.168.1.1 to 192.168.1.254
```

---

## Subnetting — The Key Formula

```
Given a /N subnet:

  Number of host bits   = 32 - N
  Total addresses       = 2^(32-N)
  Usable hosts          = 2^(32-N) - 2
                          (subtract network + broadcast addresses)

  Network address       = first address (all host bits = 0)
  Broadcast address     = last address  (all host bits = 1)
  Usable host range     = everything between network and broadcast
```

### Common subnet sizes — memorise these

```
┌──────┬──────────────────┬──────────────┬───────────────┬───────────────────┐
│ CIDR │   Subnet Mask    │  Total IPs   │ Usable Hosts  │   Common Use      │
├──────┼──────────────────┼──────────────┼───────────────┼───────────────────┤
│  /8  │ 255.0.0.0        │  16,777,216  │  16,777,214   │ Large corp/ISP    │
│ /16  │ 255.255.0.0      │  65,536      │  65,534       │ Large office/VPC  │
│ /24  │ 255.255.255.0    │  256         │  254          │ Typical LAN       │
│ /25  │ 255.255.255.128  │  128         │  126          │ Split a /24       │
│ /26  │ 255.255.255.192  │  64          │  62           │ Small subnet      │
│ /27  │ 255.255.255.224  │  32          │  30           │ Small team        │
│ /28  │ 255.255.255.240  │  16          │  14           │ Very small        │
│ /29  │ 255.255.255.248  │  8           │  6            │ Point-to-point    │
│ /30  │ 255.255.255.252  │  4           │  2            │ Router links      │
│ /32  │ 255.255.255.255  │  1           │  0 (one host) │ Specific host     │
└──────┴──────────────────┴──────────────┴───────────────┴───────────────────┘
```

---

## Worked Example — Calculating a Subnet

**Given: 192.168.10.0/26 — find everything**

```
Step 1: How many host bits?
  32 - 26 = 6 host bits

Step 2: Total addresses?
  2^6 = 64

Step 3: Usable hosts?
  64 - 2 = 62

Step 4: Network address?
  192.168.10.0 (all host bits = 0)

Step 5: Broadcast address?
  192.168.10.63 (all host bits = 1)
  63 = 64 - 1 (last address in the block)

Step 6: Usable host range?
  192.168.10.1  to  192.168.10.62

Answer:
  Network:    192.168.10.0
  Broadcast:  192.168.10.63
  Hosts:      192.168.10.1 – 192.168.10.62
  Count:      62 usable hosts
```

---

## Private IP Ranges — Must Memorise

These IP ranges are reserved for private networks (not routable
on the internet). NAT translates them to public IPs.

```
┌─────────────────┬────────────────────────────────┬───────────────┐
│   Range         │   CIDR                         │   Hosts       │
├─────────────────┼────────────────────────────────┼───────────────┤
│ 10.0.0.0 to     │ 10.0.0.0/8                     │ 16 million    │
│ 10.255.255.255  │                                │               │
├─────────────────┼────────────────────────────────┼───────────────┤
│ 172.16.0.0 to   │ 172.16.0.0/12                  │ 1 million     │
│ 172.31.255.255  │                                │               │
├─────────────────┼────────────────────────────────┼───────────────┤
│ 192.168.0.0 to  │ 192.168.0.0/16                 │ 65,536        │
│ 192.168.255.255 │                                │               │
└─────────────────┴────────────────────────────────┴───────────────┘

10.0.0.0/8      → AWS VPCs, large corporate networks
172.16.0.0/12   → Docker default bridge network
192.168.0.0/16  → Home routers (192.168.0.x or 192.168.1.x)
```

### Other special addresses

```
127.0.0.0/8       Loopback (127.0.0.1 = localhost)
169.254.0.0/16    Link-local (APIPA — no DHCP server found)
0.0.0.0           Non-specific / all interfaces
255.255.255.255   Limited broadcast
```

---

## NAT — Network Address Translation

NAT is why 8 billion devices can use 4.3 billion IPv4 addresses.
Multiple private IPs share ONE public IP.

```
YOUR HOME NETWORK                         INTERNET

192.168.1.10 (your laptop) ──┐
192.168.1.11 (your phone)  ──┤──► Router ──► 203.0.113.1 ──► google.com
192.168.1.12 (smart TV)    ──┘    (NAT)       (one public IP)

From Google's perspective, all traffic comes from 203.0.113.1.
The router keeps a translation table:

  Internal              External               Destination
  192.168.1.10:52341 ←→ 203.0.113.1:52341 ←→ 142.250.80.46:443
  192.168.1.11:54210 ←→ 203.0.113.1:54210 ←→ 142.250.80.46:443
```

---

## Subnetting a Network — Real World Example

**Scenario:** You have `10.0.0.0/16` for your AWS VPC.
You need to divide it into subnets for different purposes.

```
10.0.0.0/16  →  65,534 usable hosts  (your VPC)

Split into:
  10.0.1.0/24   Public subnet  AZ-1   (254 hosts)   ← web servers
  10.0.2.0/24   Public subnet  AZ-2   (254 hosts)   ← web servers
  10.0.3.0/24   Private subnet AZ-1   (254 hosts)   ← app servers
  10.0.4.0/24   Private subnet AZ-2   (254 hosts)   ← app servers
  10.0.5.0/24   DB subnet      AZ-1   (254 hosts)   ← databases
  10.0.6.0/24   DB subnet      AZ-2   (254 hosts)   ← databases

Each /24 = one subnet. All fit within the 10.0.0.0/16 space.
```

---

## CIDR Quick-Check Method

For any /N, the block size = `2^(32-N)`.
The network address must be a multiple of the block size.

```
/24 → block = 256  → networks: 0, 256 (doesn't apply, it's the 3rd octet)
      → 192.168.0.0, 192.168.1.0, 192.168.2.0 ... each 256 apart

/25 → block = 128  → 192.168.1.0, 192.168.1.128 (two /25s in a /24)

/26 → block = 64   → 192.168.1.0, .64, .128, .192 (four /26s in a /24)

/27 → block = 32   → .0, .32, .64, .96, .128, .160, .192, .224

/28 → block = 16   → .0, .16, .32, .48 ... .240
```

---

## Useful Commands

```bash
# See your IP address and subnet
ip a
ip addr show eth0

# See routing table (and default gateway)
ip route
# default via 192.168.1.1 dev eth0  ← your gateway

# Calculate subnet info from command line
ipcalc 192.168.10.0/26      # install: apt install ipcalc
sipcalc 192.168.10.0/26     # install: apt install sipcalc

# Check if an IP is in a subnet
python3 -c "import ipaddress; print(ipaddress.ip_address('10.0.1.5') in ipaddress.ip_network('10.0.0.0/16'))"
# True

# List all IPs in a subnet
nmap -sL 192.168.1.0/24 | grep "Nmap scan report"
```

---

## IPv6 — Quick Overview

```
Format: 8 groups of 4 hex digits
  2001:0db8:85a3:0000:0000:8a2e:0370:7334

Shortening rules:
  Leading zeros dropped:  0db8 → db8
  Consecutive zero groups replaced with ::
  2001:0db8:0000:0000:0000:0000:0370:7334
  → 2001:db8::370:7334

Special addresses:
  ::1           Loopback (like 127.0.0.1)
  fe80::/10     Link-local
  fc00::/7      Unique local (like private IPv4)
  2000::/3      Global unicast (public internet)
```

---

## Subnetting Cheat Sheet

```
/24  = 256  IPs, 254 hosts  → most common LAN size
/25  = 128  IPs, 126 hosts  → split a /24 in half
/26  = 64   IPs, 62  hosts  → quarter of a /24
/27  = 32   IPs, 30  hosts  → eighth of a /24
/28  = 16   IPs, 14  hosts  → small subnet
/29  = 8    IPs, 6   hosts  → tiny subnet
/30  = 4    IPs, 2   hosts  → point-to-point links
/32  = 1    IP,  0   hosts  → single host route

Private ranges:
  10.0.0.0/8       → large (16M hosts)
  172.16.0.0/12    → medium (1M hosts)
  192.168.0.0/16   → small (65K hosts)
```
