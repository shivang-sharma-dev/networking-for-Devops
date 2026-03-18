# 10 — VPN & Tunneling

## What is a VPN?

A **VPN (Virtual Private Network)** creates an encrypted tunnel over a public network (internet) to securely connect two endpoints.

```
[Office Laptop] ──encrypted tunnel──► [VPN Server] ──► [Private Data Center]
```

Use cases:
- Remote employees accessing internal resources
- Connecting cloud VPCs to on-premises
- Securing traffic on public Wi-Fi

---

## VPN Types

| Type | Description | Use Case |
|---|---|---|
| **Site-to-Site** | Connects two networks permanently | Office ↔ Cloud VPC |
| **Remote Access (Client VPN)** | Individual users connect to a network | Remote workers |
| **Split Tunnel** | Only route specific traffic through VPN | Bandwidth optimization |
| **Full Tunnel** | All traffic goes through VPN | Maximum security |

---

## VPN Protocols

| Protocol | Port | Encrypt | Speed | Notes |
|---|---|---|---|---|
| **WireGuard** | UDP 51820 | ChaCha20 | Very fast | Modern, simple, recommended |
| **OpenVPN** | UDP 1194 / TCP 443 | AES | Fast | Battle-tested, complex |
| **IPsec/IKEv2** | UDP 500, 4500 | AES | Fast | Common in enterprise, mobile |
| **L2TP/IPsec** | UDP 1701 | AES | Moderate | Older, two protocols combined |
| **PPTP** | TCP 1723 | Weak | Fast | **Deprecated — insecure** |

---

## WireGuard Setup

WireGuard is the **recommended modern VPN** — simple config, high performance.

### Server Setup

```bash
# Install WireGuard
sudo apt install wireguard

# Generate server keys
umask 077
wg genkey | tee server_private.key | wg pubkey > server_public.key

# /etc/wireguard/wg0.conf
[Interface]
Address = 10.8.0.1/24
ListenPort = 51820
PrivateKey = <server_private_key>
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE

[Peer]
PublicKey = <client_public_key>
AllowedIPs = 10.8.0.2/32

# Start WireGuard
sudo wg-quick up wg0
sudo systemctl enable wg-quick@wg0
```

### Client Setup

```bash
# Generate client keys
wg genkey | tee client_private.key | wg pubkey > client_public.key

# /etc/wireguard/wg0.conf
[Interface]
Address = 10.8.0.2/24
PrivateKey = <client_private_key>
DNS = 1.1.1.1

[Peer]
PublicKey = <server_public_key>
Endpoint = <server_ip>:51820
AllowedIPs = 0.0.0.0/0  # Full tunnel
# or AllowedIPs = 10.8.0.0/24  # Split tunnel — only route private IPs

# Connect
sudo wg-quick up wg0
```

---

## SSH Tunneling

SSH can create encrypted tunnels without a dedicated VPN solution.

### Local Port Forwarding
```bash
# Access remote DB as if it were local
ssh -L 5432:db.private:5432 user@bastion.server.com

# Then connect locally:
psql -h localhost -p 5432 -U myuser mydb
```

### Remote Port Forwarding
```bash
# Expose local port to the remote server
ssh -R 8080:localhost:3000 user@remote-server.com
```

### Dynamic Port Forwarding (SOCKS Proxy)
```bash
# Create a SOCKS5 proxy through a server
ssh -D 1080 user@remote-server.com

# Configure browser/apps to use SOCKS5 proxy: localhost:1080
```

### Jump Host (Bastion)
```bash
# Connect to internal host through a bastion
ssh -J user@bastion.example.com user@internal-server.private

# Or in ~/.ssh/config:
Host internal
  HostName 10.0.1.50
  ProxyJump bastion.example.com
  User ubuntu
```

---

## IPsec

**IPsec** is a suite of protocols for securing IP communications.

- **AH (Authentication Header)**: Integrity and authentication (no encryption)
- **ESP (Encapsulating Security Payload)**: Encryption + authentication
- **IKE (Internet Key Exchange)**: Negotiates keys and SAs

Modes:
- **Transport mode**: Encrypts only the payload (between two hosts)
- **Tunnel mode**: Encrypts the entire packet (between two gateways, site-to-site VPN)

---

## AWS VPN Options

| Option | Use Case |
|---|---|
| **AWS Site-to-Site VPN** | Connect on-prem to VPC over internet (IPsec) |
| **AWS Client VPN** | Remote access for users to VPC |
| **AWS Direct Connect** | Dedicated private connection (not VPN, no internet) |

```bash
# Test VPN connectivity
ping 10.0.0.1       # Ping resource inside VPN
traceroute 10.0.0.1 # Trace path
```

---

## DevOps Tips

- **Never SSH to production** over the public internet — use a VPN or bastion
- Use **WireGuard** for modern VPN setups (simpler than OpenVPN)
- SSH tunneling is great for **quick, temporary access** to internal services
- Use `~/.ssh/config` to define all your SSH connection profiles
- Monitor VPN **handshake failures** and **data transfer** as health metrics
