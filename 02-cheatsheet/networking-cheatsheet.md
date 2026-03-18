# 🗒️ Networking Cheatsheet for DevOps

## IP & Subnetting

| CIDR | Hosts | Mask |
|---|---|---|
| /8 | 16,777,214 | 255.0.0.0 |
| /16 | 65,534 | 255.255.0.0 |
| /24 | 254 | 255.255.255.0 |
| /28 | 14 | 255.255.255.240 |
| /30 | 2 | 255.255.255.252 |
| /32 | 1 (host) | 255.255.255.255 |

Private ranges: `10.0.0.0/8` · `172.16.0.0/12` · `192.168.0.0/16`

---

## Common Ports

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
| 27017 | TCP | MongoDB |
| 2379 | TCP | etcd |
| 6443 | TCP | Kubernetes API |

---

## DNS Records

| Record | Use |
|---|---|
| A | domain → IPv4 |
| AAAA | domain → IPv6 |
| CNAME | alias → domain |
| MX | mail server |
| TXT | SPF, DKIM, verification |
| NS | nameservers |
| PTR | IP → domain (reverse) |

---

## HTTP Status Codes

| Code | Meaning |
|---|---|
| 200 | OK |
| 201 | Created |
| 301 | Moved Permanently |
| 400 | Bad Request |
| 401 | Unauthorized |
| 403 | Forbidden |
| 404 | Not Found |
| 429 | Too Many Requests |
| 500 | Internal Server Error |
| 502 | Bad Gateway |
| 503 | Service Unavailable |
| 504 | Gateway Timeout |

---

## CLI Commands

### ping / traceroute
```bash
ping -c 4 8.8.8.8
traceroute google.com
mtr google.com
```

### DNS
```bash
dig google.com
dig @1.1.1.1 example.com
dig +trace google.com
dig -x 8.8.8.8            # reverse
nslookup google.com
```

### Port Testing
```bash
nc -zv host 443           # TCP
nc -zvu host 53            # UDP
curl -v https://host
```

### Sockets & Ports
```bash
ss -tulnp                 # listening
ss -tnp state established
lsof -i :8080             # process on port
```

### Packet Capture
```bash
sudo tcpdump -i eth0 host 8.8.8.8
sudo tcpdump -i eth0 port 443
sudo tcpdump -i eth0 -w out.pcap
```

### Interface & Routes
```bash
ip addr show
ip route show
ip neigh show             # ARP table
ip route get 8.8.8.8
```

### Firewall (UFW)
```bash
sudo ufw status
sudo ufw allow 443/tcp
sudo ufw deny 23
sudo ufw allow from 10.0.0.0/8 to any port 22
```

### Firewall (iptables)
```bash
sudo iptables -L -v -n
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT
sudo iptables -A INPUT -j DROP
```

### curl Timing
```bash
curl -w "DNS:%{time_namelookup} Connect:%{time_connect} Total:%{time_total}\n" \
     -o /dev/null -s https://example.com
```

---

## SSH

```bash
# Basic connection
ssh user@host

# With key
ssh -i ~/.ssh/key.pem user@host

# Port forward (local)
ssh -L 5432:db.private:5432 user@bastion

# Jump host
ssh -J user@bastion user@internal

# Tunnel SOCKS proxy
ssh -D 1080 user@host
```

---

## WireGuard

```bash
wg-quick up wg0           # Connect
wg-quick down wg0         # Disconnect
wg show                    # Status
sudo systemctl enable wg-quick@wg0
```

---

## Kubernetes Networking

```bash
# DNS test
kubectl exec -it <pod> -- nslookup kubernetes.default

# Connectivity test
kubectl exec -it <pod> -- curl http://<svc>

# Service endpoints
kubectl get endpoints <svc>

# Network policies
kubectl get networkpolicies

# Debug pod (netshoot)
kubectl run tmp --rm -i --tty --image nicolaka/netshoot -- bash
```
