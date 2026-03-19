# 10 — VPN and Tunneling

> VPNs connect isolated networks securely over the internet.
> In DevOps you use them to access private cloud resources,
> connect offices to cloud, and secure remote developer access.

---

## What is a VPN?

A VPN (Virtual Private Network) creates an encrypted tunnel
over a public network (the internet), making it behave like
a private network.

```
ANALOGY: A private underground tunnel

WITHOUT VPN:
  Your laptop ───public internet──► AWS private server
              (anyone can intercept traffic)

WITH VPN:
  Your laptop ══private tunnel══► VPN server ──► AWS private server
              (encrypted, can't be intercepted)

Your laptop appears to be "inside" the private network.
It gets a private IP from that network.
It can reach resources that are not exposed to the internet.
```

---

## Types of VPN

```
SITE-TO-SITE VPN
  Connects two entire networks permanently.
  
  Office Network (192.168.1.0/24) ══VPN══ AWS VPC (10.0.0.0/16)
  
  All office machines can reach all AWS resources.
  VPN runs 24/7 between two gateways (routers).
  Use case: Connect on-premises datacenter to cloud.

CLIENT VPN (Remote Access VPN)
  Connects individual users to a network.
  
  Developer laptop ══VPN══ Company network
  
  Developer's laptop gets a VPN IP.
  Can access internal resources (databases, admin panels).
  Use case: Remote developers accessing private resources.

SPLIT TUNNEL VPN
  Only specific traffic goes through VPN.
  Company traffic → VPN tunnel.
  Personal traffic (YouTube, etc.) → normal internet.
  
  More efficient. Requires careful routing config.
```

---

## How a VPN Tunnel Works

```
ENCAPSULATION — packet inside a packet

Normal packet:
  [IP Header: src=your_IP, dst=db_IP | Data]

VPN packet:
  [IP Header: src=your_IP, dst=vpn_server | Encrypted[IP Header + Data]]
                                            └─ original packet is encrypted
                                               and wrapped in a new one

On the VPN server side:
  1. Receives outer packet
  2. Decrypts inner packet
  3. Routes original packet to destination (db_IP)
  4. Sends response back through tunnel
```

---

## OpenVPN

OpenVPN is the most widely deployed open-source VPN.

```bash
# Server setup (basic)
sudo apt install openvpn easy-rsa

# Generate certificates
make-cadir /etc/openvpn/easy-rsa
cd /etc/openvpn/easy-rsa
./easyrsa init-pki
./easyrsa build-ca
./easyrsa gen-req server nopass
./easyrsa sign-req server server
./easyrsa gen-dh

# Client config (~/.config/openvpn/client.conf)
# client
# dev tun
# proto udp
# remote vpn.example.com 1194
# ca ca.crt
# cert client.crt
# key client.key

# Connect
sudo openvpn --config client.conf
```

---

## WireGuard — Modern VPN

WireGuard is newer, simpler, and much faster than OpenVPN.
It is now built into the Linux kernel (5.6+).

```bash
# Install
sudo apt install wireguard

# Generate key pair
wg genkey | tee privatekey | wg pubkey > publickey

# Server config (/etc/wireguard/wg0.conf)
# [Interface]
# PrivateKey = SERVER_PRIVATE_KEY
# Address = 10.0.0.1/24
# ListenPort = 51820
#
# [Peer]
# PublicKey = CLIENT_PUBLIC_KEY
# AllowedIPs = 10.0.0.2/32

# Client config
# [Interface]
# PrivateKey = CLIENT_PRIVATE_KEY
# Address = 10.0.0.2/24
#
# [Peer]
# PublicKey = SERVER_PUBLIC_KEY
# Endpoint = vpn.example.com:51820
# AllowedIPs = 0.0.0.0/0  (all traffic through VPN)
# or: AllowedIPs = 10.0.0.0/8  (split tunnel — only internal traffic)

# Start VPN
sudo wg-quick up wg0
sudo systemctl enable wg-quick@wg0   # start on boot

# Status
sudo wg show
```

---

## SSH Tunneling

SSH can create tunnels without a full VPN setup.
Great for quick, one-off access to private services.

```bash
# Local port forwarding — access remote service locally
# Access remote MySQL (port 3306) via localhost:3307
ssh -L 3307:localhost:3306 user@bastion-host -N
# Now: mysql -h 127.0.0.1 -P 3307  connects to remote MySQL

# Access private web app through bastion
ssh -L 8080:10.0.1.50:80 user@bastion -N
# Now: curl localhost:8080  reaches the private web app

# Remote port forwarding — expose local service to remote server
ssh -R 8080:localhost:3000 user@remote -N
# Remote server can now access your local port 3000 via its localhost:8080

# Dynamic port forwarding — SOCKS5 proxy
ssh -D 1080 user@remote -N
# Configure browser to use SOCKS5 proxy at localhost:1080
# All browser traffic goes through the remote server

# Keep SSH tunnel alive
ssh -L 3307:db:3306 user@bastion -N -o ServerAliveInterval=60
```

---

## AWS VPN Options

```
AWS CLIENT VPN
  Individual developers connect to AWS VPC.
  Each dev gets a VPN certificate.
  Access private resources (RDS, ElasticSearch, etc.)
  Cost: per connection-hour.

AWS SITE-TO-SITE VPN
  Connect on-premises office to AWS VPC.
  Uses IPSec over internet.
  Two tunnels for redundancy.
  ~$0.05/hour per VPN connection.

AWS DIRECT CONNECT
  Physical dedicated connection from your DC to AWS.
  Not over the internet — private fibre.
  Consistent low latency, high bandwidth.
  Expensive but worth it for high-traffic production.

VPC PEERING
  Connect two AWS VPCs together.
  Not technically a VPN but serves similar purpose.
  Traffic stays within AWS network.
  No bandwidth charge within same region.
```

---

## Tunneling Protocols Quick Reference

| Protocol | Port | Use |
|---|---|---|
| OpenVPN | UDP 1194 | Widely compatible, any platform |
| WireGuard | UDP 51820 | Fast, modern, simple config |
| IPSec/IKEv2 | UDP 500, 4500 | AWS Site-to-Site VPN, iOS/Android |
| SSH tunnels | TCP 22 | Quick ad-hoc tunnels |
| SSTP | TCP 443 | Windows, uses HTTPS port |
