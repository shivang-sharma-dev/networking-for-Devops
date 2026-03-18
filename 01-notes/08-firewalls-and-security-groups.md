# 08 — Firewalls & Security Groups

## What is a Firewall?

A **firewall** controls incoming and outgoing network traffic based on **predefined rules**.

```
Internet ──► [Firewall] ──► Internal Network
                │
                └─ ALLOW: port 443 from 0.0.0.0/0
                   DENY: port 22 from 0.0.0.0/0
                   ALLOW: port 22 from 10.0.0.0/8
```

---

## Stateful vs Stateless Firewalls

| | Stateful | Stateless |
|---|---|---|
| Tracks connections? | Yes | No |
| Return traffic | Automatically allowed | Must create rule for both directions |
| Performance | Slightly slower | Faster |
| Examples | AWS Security Groups, iptables conntrack, UFW | AWS NACLs, ACLs on routers |

---

## iptables (Linux Firewall)

`iptables` is the classic Linux firewall tool, uses **chains** and **tables**.

### Tables & Chains
```
filter table (most common):
  INPUT   → incoming packets to this host
  OUTPUT  → outgoing packets from this host
  FORWARD → packets routed through this host
```

### Basic Commands

```bash
# View current rules
sudo iptables -L -v -n

# Allow incoming SSH
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# Allow incoming HTTP and HTTPS
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# Allow established connections
sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Drop everything else
sudo iptables -A INPUT -j DROP

# Delete a rule
sudo iptables -D INPUT -p tcp --dport 22 -j ACCEPT

# Save rules
sudo iptables-save > /etc/iptables/rules.v4
```

---

## UFW (Uncomplicated Firewall)

UFW is a user-friendly frontend for iptables, common on Ubuntu.

```bash
# Enable UFW
sudo ufw enable

# Allow SSH
sudo ufw allow 22/tcp

# Allow HTTP/HTTPS
sudo ufw allow http
sudo ufw allow https

# Allow from specific IP
sudo ufw allow from 10.0.0.0/8 to any port 22

# Deny a port
sudo ufw deny 23

# View status
sudo ufw status verbose

# Delete a rule
sudo ufw delete allow 22/tcp
```

---

## nftables (Modern iptables replacement)

```bash
# List all rules
sudo nft list ruleset

# Add a rule (allow HTTPS)
sudo nft add rule inet filter input tcp dport 443 accept
```

---

## AWS Security Groups

Security Groups are **stateful** virtual firewalls for EC2, RDS, Lambda, etc.

### Key Points
- Rules only **allow** traffic (no explicit deny)
- **Inbound rules**: What can come in
- **Outbound rules**: What can go out (default: allow all)
- Rules specify: **Protocol, Port range, Source (IP or SG)**

### Example Security Group Rules

| Type | Protocol | Port | Source | Purpose |
|---|---|---|---|---|
| SSH | TCP | 22 | 10.0.0.0/8 | Admin access from VPN only |
| HTTP | TCP | 80 | 0.0.0.0/0 | Public web traffic |
| HTTPS | TCP | 443 | 0.0.0.0/0 | Public secure web traffic |
| MySQL | TCP | 3306 | sg-app-server | Only app servers can reach DB |
| All traffic | All | All | sg-same-group | Allow internal communication |

---

## AWS NACLs (Network ACLs)

**Stateless** firewall applied at the **subnet level**.

- Rules evaluated in order (lowest number first)
- Must create rules for **both directions** (inbound + outbound)
- Can **explicitly deny** traffic (unlike Security Groups)

| Rule # | Protocol | Port | Source | Action |
|---|---|---|---|---|
| 100 | TCP | 443 | 0.0.0.0/0 | ALLOW |
| 200 | TCP | 22 | 10.0.0.0/8 | ALLOW |
| * | All | All | 0.0.0.0/0 | DENY |

---

## Security Group vs NACL

| Feature | Security Group | NACL |
|---|---|---|
| Level | Instance / ENI | Subnet |
| Stateful? | Yes | No |
| Allow only | Yes | Allow + Deny |
| Rule order | All rules evaluated | Numbered order |
| Default | Deny all inbound | Allow all |

---

## DevOps Best Practices

- **Least privilege**: Only open the ports that are truly needed
- **Never open 0.0.0.0/0 to SSH** — use a VPN or bastion host
- **Use SG references** instead of IP ranges when possible (auto-scales)
- **Log denied traffic** for security auditing
- **Terraform**: Manage security groups as code — avoid console drift

```hcl
resource "aws_security_group" "app" {
  name   = "app-sg"
  vpc_id = aws_vpc.main.id

  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```
