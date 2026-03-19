# 12 — Networking in the Cloud

> Cloud networking is on-premises networking, but virtual and API-driven.
> Everything you learned in notes 01-11 applies here.
> This note shows how those concepts map to real cloud resources.

---

## The Big Picture — AWS VPC

A VPC (Virtual Private Cloud) is your own private section of AWS.
Think of it as your own data centre inside AWS.

```
AWS REGION (e.g., us-east-1)
┌───────────────────────────────────────────────────────────────┐
│                                                               │
│  YOUR VPC (10.0.0.0/16)                                       │
│  ┌─────────────────────────────────────────────────────────┐  │
│  │                                                         │  │
│  │  Availability Zone A          Availability Zone B       │  │
│  │  ┌─────────────────────┐  ┌─────────────────────┐      │  │
│  │  │ Public Subnet       │  │ Public Subnet       │      │  │
│  │  │ 10.0.1.0/24         │  │ 10.0.2.0/24         │      │  │
│  │  │ [Web Server]        │  │ [Web Server]        │      │  │
│  │  └─────────────────────┘  └─────────────────────┘      │  │
│  │  ┌─────────────────────┐  ┌─────────────────────┐      │  │
│  │  │ Private Subnet      │  │ Private Subnet      │      │  │
│  │  │ 10.0.3.0/24         │  │ 10.0.4.0/24         │      │  │
│  │  │ [App Server]        │  │ [App Server]        │      │  │
│  │  └─────────────────────┘  └─────────────────────┘      │  │
│  │  ┌─────────────────────┐  ┌─────────────────────┐      │  │
│  │  │ DB Subnet           │  │ DB Subnet           │      │  │
│  │  │ 10.0.5.0/24         │  │ 10.0.6.0/24         │      │  │
│  │  │ [RDS Database]      │  │ [RDS Standby]       │      │  │
│  │  └─────────────────────┘  └─────────────────────┘      │  │
│  │                                                         │  │
│  └─────────────────────────────────────────────────────────┘  │
│                                                               │
└───────────────────────────────────────────────────────────────┘
```

---

## VPC Components

### Internet Gateway (IGW)
Allows resources in your VPC to communicate with the internet.

```
Internet ◄──► Internet Gateway ◄──► Public Subnet resources

Rules:
  1. Subnet must have a route: 0.0.0.0/0 → igw-xxxx
  2. Instance must have a public IP or Elastic IP
  3. Security group must allow the traffic
```

### NAT Gateway
Allows private subnet resources to reach the internet
but prevents internet from initiating connections to them.

```
Private Subnet resource
       │
       ▼
NAT Gateway (in public subnet)
       │
       ▼
Internet Gateway
       │
       ▼
Internet

Direction: outbound only.
Your private app server can download packages, call APIs.
But nobody on the internet can reach your private server.
```

### Route Tables

```
PUBLIC SUBNET ROUTE TABLE:
  Destination     Target
  10.0.0.0/16    local          ← all VPC traffic stays local
  0.0.0.0/0      igw-xxxxxx    ← internet traffic → internet gateway

PRIVATE SUBNET ROUTE TABLE:
  Destination     Target
  10.0.0.0/16    local          ← all VPC traffic stays local
  0.0.0.0/0      nat-xxxxxx    ← internet traffic → NAT gateway
```

---

## Public vs Private Subnets

```
PUBLIC SUBNET                   PRIVATE SUBNET
─────────────                   ──────────────
Route to internet gateway       Route to NAT gateway
Resources have public IPs       Resources have private IPs only
Reachable from internet         NOT reachable from internet
Use for: Load balancers,        Use for: App servers,
  Bastion hosts, NAT gateways     Databases, Internal services
```

---

## Security Groups vs NACLs

```
SECURITY GROUPS (instance-level firewall)
  ┌─────────────────────────┐
  │  EC2 Instance           │
  │  ┌─────────────────┐    │
  │  │ Security Group  │    │ ← attached to instance
  │  │ Inbound rules   │    │
  │  │ Outbound rules  │    │
  │  └─────────────────┘    │
  └─────────────────────────┘
  Stateful: return traffic auto-allowed
  Allow rules only

NETWORK ACLs (subnet-level firewall)
  ┌─────────────────────────────────────┐
  │  Subnet                             │
  │  ┌─────────────────┐                │ ← attached to subnet
  │  │ NACL            │                │
  │  │ Inbound rules   │                │
  │  │ Outbound rules  │  EC2 Instance  │
  │  └─────────────────┘                │
  └─────────────────────────────────────┘
  Stateless: need both inbound AND outbound rules
  Allow AND Deny rules
  Rules evaluated in number order (lowest first)
```

---

## VPC Peering

Connect two VPCs so they can communicate as if on the same network.

```
VPC A (10.0.0.0/16)  ◄──── Peering Connection ────►  VPC B (172.16.0.0/16)

Resources in VPC A can reach VPC B using private IPs.
Traffic stays within AWS (no internet).
Non-transitive: A↔B and B↔C does NOT mean A can reach C.

Use cases:
  Connect dev VPC to prod VPC (carefully)
  Connect to a shared services VPC (monitoring, logging)
  Multi-account setups
```

---

## Elastic IP (EIP)

A static public IP address that you own (not lost when instance stops).

```
Without EIP:
  Instance stops → public IP released → app URL breaks

With EIP:
  Instance stops → EIP persists → re-attach to new instance
  IP address never changes

Use for:
  Any server that needs a fixed public IP
  IP whitelisting (tell partners "always allow 52.1.2.3")
  Failover (move EIP from failed instance to replacement)
```

---

## DNS in the Cloud

### AWS Route 53

```
Record types you use daily:

A record:     myapp.com → 52.1.2.3   (EC2 public IP or ELB IP)
CNAME:        www.myapp.com → myapp.com
ALIAS:        myapp.com → alb-xxx.us-east-1.elb.amazonaws.com
              (ALIAS is like CNAME but works for root domain)

Routing policies:
  Simple       → one answer always
  Weighted     → 90% to v1, 10% to v2 (gradual rollout)
  Latency      → route to nearest region
  Failover     → primary + secondary (health check based)
  Geolocation  → EU users → EU servers, US users → US servers
```

### Private Hosted Zones

```
Private DNS within your VPC:
  db.internal        → 10.0.5.10  (only resolvable inside VPC)
  redis.internal     → 10.0.3.20
  api.internal       → 10.0.3.15

No public DNS for internal services.
Use service names in your app config instead of IPs.
```

---

## Production VPC Design

```
10.0.0.0/16  (65,534 IPs for your entire environment)

Public subnets (one per AZ):
  10.0.1.0/24  AZ-a  — Load balancers, NAT gateway
  10.0.2.0/24  AZ-b  — Load balancers, NAT gateway

App subnets (one per AZ):
  10.0.10.0/24 AZ-a  — Application servers, ECS tasks
  10.0.11.0/24 AZ-b  — Application servers, ECS tasks

Data subnets (one per AZ):
  10.0.20.0/24 AZ-a  — RDS, ElastiCache, Elasticsearch
  10.0.21.0/24 AZ-b  — RDS standby, replicas

Management subnet:
  10.0.30.0/24       — Bastion host, monitoring, VPN endpoint
```

---

## Connecting to Private Resources Safely

```
OPTION 1: Bastion Host (Jump Box)
  Internet → Bastion (public subnet, port 22 open to your IP)
           → Private server (port 22 open to bastion SG only)

  ssh -J ec2-user@bastion-ip ubuntu@10.0.10.5

OPTION 2: AWS Session Manager (better — no open SSH port needed)
  aws ssm start-session --target i-1234567890

OPTION 3: VPN
  All developers VPN into the VPC.
  Access private resources directly.
  No public SSH ports needed.

OPTION 4: Port forwarding via SSM
  aws ssm start-session --target i-xxx \
    --document-name AWS-StartPortForwardingSession \
    --parameters "localPortNumber=5432,portNumber=5432"
```

---

## Networking Concepts Mapped to Cloud

| Concept | On-premises | AWS | GCP | Azure |
|---|---|---|---|---|
| Virtual network | LAN | VPC | VPC | VNet |
| Subnet | VLAN | Subnet | Subnet | Subnet |
| Router | Physical router | Route Table | Route | Route Table |
| Firewall (instance) | Host firewall | Security Group | Firewall Rules | NSG |
| Firewall (subnet) | ACL | Network ACL | — | — |
| Public IP | ISP IP | Elastic IP | Static IP | Public IP |
| Private DNS | Internal DNS | Route 53 Private | Cloud DNS | Azure DNS |
| VPN | IPSec device | Site-to-Site VPN | Cloud VPN | VPN Gateway |
| Network monitoring | Wireshark/SNMP | VPC Flow Logs | Flow Logs | NSG Flow Logs |
