# 03 — IP Addressing & Subnetting

## IPv4 Addressing

An **IPv4 address** is a 32-bit number written as four decimal octets:

```
192   .  168   .  1     .  100
11000000.10101000.00000001.01100100
```

### Address Classes (Legacy)

| Class | Range | Default Mask | Use |
|---|---|---|---|
| A | 1.0.0.0 – 126.255.255.255 | /8 | Large orgs |
| B | 128.0.0.0 – 191.255.255.255 | /16 | Medium orgs |
| C | 192.0.0.0 – 223.255.255.255 | /24 | Small orgs |

---

## Private IP Ranges (RFC 1918)

These IPs are **not routable on the internet** — used inside VPCs, LANs, containers.

| Range | CIDR | Example use |
|---|---|---|
| 10.0.0.0 – 10.255.255.255 | 10.0.0.0/8 | AWS VPCs, large orgs |
| 172.16.0.0 – 172.31.255.255 | 172.16.0.0/12 | Docker default bridge |
| 192.168.0.0 – 192.168.255.255 | 192.168.0.0/16 | Home routers |

---

## CIDR Notation

**Classless Inter-Domain Routing (CIDR)** lets us define network sizes flexibly.

```
192.168.1.0/24
           └── 24 bits for network, 8 bits for hosts
```

### Quick CIDR Reference

| CIDR | Subnet Mask | # Hosts | Typical Use |
|---|---|---|---|
| /8 | 255.0.0.0 | 16,777,214 | Large VPC |
| /16 | 255.255.0.0 | 65,534 | Medium VPC |
| /24 | 255.255.255.0 | 254 | Small subnet |
| /28 | 255.255.255.240 | 14 | Small subnet in AWS |
| /30 | 255.255.255.252 | 2 | Point-to-point links |
| /32 | 255.255.255.255 | 1 | Single host / Security group rule |

### Formula
```
Number of hosts = 2^(32 - prefix) - 2
```
> Subtract 2 for the **network address** and **broadcast address**.

---

## Subnetting Example

**Scenario**: Split `192.168.1.0/24` into 4 equal subnets.

We need 4 subnets → borrow **2 bits** → /26 (because 2² = 4)

| Subnet | Range | Broadcast |
|---|---|---|
| 192.168.1.0/26 | .1 – .62 | .63 |
| 192.168.1.64/26 | .65 – .126 | .127 |
| 192.168.1.128/26 | .129 – .190 | .191 |
| 192.168.1.192/26 | .193 – .254 | .255 |

Each subnet has **62 usable hosts**.

---

## Special Addresses

| Address | Meaning |
|---|---|
| `0.0.0.0` | Default route / all IPs on the host |
| `127.0.0.1` | Loopback (localhost) |
| `255.255.255.255` | Limited broadcast |
| `169.254.x.x` | Link-local (APIPA — no DHCP found) |

---

## IPv6 Basics

IPv6 is a **128-bit** address in hexadecimal groups:

```
2001:0db8:85a3:0000:0000:8a2e:0370:7334
```

### Key differences from IPv4
- No broadcast, uses **multicast**
- **No NAT** needed (address space is huge: 3.4 × 10³⁸)
- Built-in **IPsec** support
- Loopback: `::1`
- Link-local: `fe80::/10`

---

## NAT — Network Address Translation

**NAT** allows multiple devices using private IPs to share a single public IP.

```
[10.0.1.5]──────┐
[10.0.1.6]──────┤──► [NAT Router] ──► [Public IP: 52.10.0.1] ──► Internet
[10.0.1.7]──────┘
```

- **SNAT (Source NAT)**: Changes source IP — outbound traffic
- **DNAT (Destination NAT)**: Changes destination IP — inbound traffic (port forwarding)

---

## DevOps Relevance

- **AWS VPCs**: You choose CIDR blocks like `10.0.0.0/16`
- **Security groups**: Use `/32` to whitelist a single IP
- **Kubernetes pods**: Each pod gets a unique IP from the pod CIDR
- **Terraform**: You define `cidr_block` in `aws_subnet`
- **Docker**: Uses `172.17.0.0/16` for the default bridge network
