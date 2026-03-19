# 08 — Firewalls and Security Groups

> Firewalls are the gatekeepers of your network. Every cloud
> engineer configures security groups daily. Getting this wrong
> means either your app is unreachable or your servers are exposed.

---

## What is a Firewall?

A firewall filters network traffic based on rules.
It sits between your network and the outside world,
deciding what gets through and what gets blocked.

```
ANALOGY: A nightclub bouncer

  Rules:
    ALLOW: VIP list (known good IPs)
    ALLOW: Guests with valid ID (authenticated traffic)
    DENY:  Anyone on the banned list (known bad IPs)
    DENY:  Anyone without ID (unauthenticated)

  Traffic:
    Request arrives → Firewall checks rules → Allow or Deny
```

---

## Stateful vs Stateless Firewalls

```
STATELESS FIREWALL
  Checks each packet independently.
  Does not remember previous packets.
  You must write rules for BOTH directions.

  Rule: Allow TCP port 80 inbound
  Rule: Allow TCP port 80 outbound  ← also needed for responses

STATEFUL FIREWALL (modern — what you almost always use)
  Tracks connection state.
  If you allow inbound traffic, return traffic is
  automatically allowed.

  Rule: Allow TCP port 80 inbound  ← response traffic auto-allowed
  
  AWS Security Groups = stateful
  AWS Network ACLs    = stateless (you need both inbound + outbound rules)
```

---

## iptables — Linux Firewall

iptables is the classic Linux firewall (still widely used).

```
iptables works with CHAINS and TABLES:

TABLES:
  filter  → the default, handles packet filtering
  nat     → network address translation
  mangle  → modify packet headers

CHAINS (in the filter table):
  INPUT   → packets coming INTO this machine
  OUTPUT  → packets going OUT from this machine
  FORWARD → packets passing THROUGH this machine (router)

Each chain has an ordered list of RULES.
Packet matches first matching rule → action taken.
```

```bash
# View current rules
sudo iptables -L -n -v
sudo iptables -L INPUT -n -v --line-numbers

# Allow SSH
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# Allow HTTP and HTTPS
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# Allow from specific IP only
sudo iptables -A INPUT -s 192.168.1.0/24 -p tcp --dport 22 -j ACCEPT

# Block an IP
sudo iptables -A INPUT -s 1.2.3.4 -j DROP

# Allow established connections (stateful — critical rule)
sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
sudo iptables -A INPUT -i lo -j ACCEPT    # always allow loopback

# Set default policy to DROP (block everything not explicitly allowed)
sudo iptables -P INPUT DROP
sudo iptables -P FORWARD DROP
sudo iptables -P OUTPUT ACCEPT            # usually allow all outbound

# Delete a rule by line number
sudo iptables -D INPUT 3

# Save rules permanently (Ubuntu)
sudo apt install iptables-persistent
sudo netfilter-persistent save
```

---

## ufw — Simple Firewall (Ubuntu)

ufw is a frontend to iptables that's much easier to use.

```bash
sudo ufw status verbose
sudo ufw enable
sudo ufw disable
sudo ufw reset                      # clear all rules

# Basic rules
sudo ufw allow 22/tcp               # SSH
sudo ufw allow 80/tcp               # HTTP
sudo ufw allow 443/tcp              # HTTPS
sudo ufw allow 8080                 # specific port

# From specific source
sudo ufw allow from 192.168.1.0/24 to any port 22
sudo ufw allow from 10.0.0.0/8

# Deny
sudo ufw deny 8080
sudo ufw deny from 1.2.3.4

# Rate limiting (protect SSH from brute force)
sudo ufw limit 22/tcp               # max 6 connections per 30 seconds

# Delete rules
sudo ufw status numbered            # see rule numbers
sudo ufw delete 3                   # delete rule 3
sudo ufw delete allow 80/tcp        # delete by specification

# Logging
sudo ufw logging on
sudo ufw logging high
```

---

## AWS Security Groups

Security Groups are virtual firewalls for EC2 instances in AWS.
They are stateful — you only need inbound rules (return traffic auto-allowed).

```
SECURITY GROUP RULES FORMAT:
  Type | Protocol | Port Range | Source/Destination

INBOUND RULES (what can come in):
  SSH        TCP   22          My IP (203.0.113.5/32)    ← only your IP
  HTTP       TCP   80          0.0.0.0/0                 ← anyone
  HTTPS      TCP   443         0.0.0.0/0                 ← anyone
  Custom     TCP   5432        sg-app-servers             ← only app servers SG

OUTBOUND RULES (what can go out):
  All traffic  All   All   0.0.0.0/0    ← allow all outbound (default)
```

### 3-Tier Architecture Security Groups

```
INTERNET
   │
   ▼
┌──────────────────────────────────────────────────┐
│ sg-load-balancer                                  │
│ Inbound:  80, 443 from 0.0.0.0/0                 │
│ Outbound: 8080 to sg-app-servers                 │
└──────────────────────────────────────────────────┘
   │
   ▼
┌──────────────────────────────────────────────────┐
│ sg-app-servers                                    │
│ Inbound:  8080 from sg-load-balancer             │
│ Outbound: 5432 to sg-databases                   │
└──────────────────────────────────────────────────┘
   │
   ▼
┌──────────────────────────────────────────────────┐
│ sg-databases                                      │
│ Inbound:  5432 from sg-app-servers               │
│ Outbound: nothing (or all)                        │
└──────────────────────────────────────────────────┘

Each layer only accepts traffic from the layer above it.
Databases are never exposed to the internet.
```

---

## Network ACLs vs Security Groups (AWS)

```
┌──────────────────┬──────────────────┬──────────────────────┐
│ Feature          │ Security Group   │ Network ACL          │
├──────────────────┼──────────────────┼──────────────────────┤
│ Level            │ Instance         │ Subnet               │
│ Stateful?        │ Yes              │ No (need both rules) │
│ Allow/Deny       │ Allow only       │ Both allow and deny  │
│ Rule evaluation  │ All rules        │ Rules in order       │
│ Default          │ Deny all inbound │ Allow all            │
└──────────────────┴──────────────────┴──────────────────────┘

Use security groups for most things.
Add Network ACLs as an extra layer for subnet-level blocking
(e.g., block a specific IP range at the subnet level).
```

---

## Firewall Best Practices

```
1. Default deny
   Block everything. Explicitly allow only what is needed.
   sudo ufw default deny incoming
   sudo ufw default allow outgoing

2. Principle of least privilege
   Open the minimum ports necessary.
   Never open 0.0.0.0/0 for SSH in production.

3. Restrict SSH to known IPs
   Allow 22 only from YOUR IP or a bastion host.
   Never from 0.0.0.0/0.

4. Use security groups per layer
   Load balancer SG, app SG, database SG.
   Each only accepts from the layer above.

5. No inbound rules on databases from internet
   Databases should ONLY accept from app servers.
   Never 0.0.0.0/0 on port 3306/5432.

6. Log dropped traffic
   sudo ufw logging on
   Monitor /var/log/ufw.log for suspicious patterns.

7. Regularly audit rules
   Remove rules you no longer need.
   Check for overly permissive rules (0.0.0.0/0 on non-HTTP ports).
```

---

## Debugging Firewall Issues

```bash
# Is the service running and listening?
ss -tulnp | grep 8080

# Can I connect from the same machine?
curl localhost:8080

# Can I connect from another machine on the same network?
curl 192.168.1.10:8080

# If local works but remote doesn't → likely a firewall issue
# Check ufw rules
sudo ufw status verbose

# Check iptables rules
sudo iptables -L -n -v

# Test if port is reachable from outside
nc -zv REMOTE_IP PORT

# Check if traffic is being dropped (watch for DROP in iptables)
sudo iptables -L INPUT -n -v | grep DROP

# On AWS: check security group inbound rules
# On GCP: check VPC firewall rules
# On Azure: check Network Security Group rules
```
