# 12 — Networking in the Cloud

## Cloud Networking Overview

Cloud providers virtualize networking using **Software Defined Networking (SDN)**. You define networks, subnets, routes, and firewalls in code — no physical hardware.

---

## AWS Networking

### VPC (Virtual Private Cloud)
A **VPC** is your isolated virtual network in AWS.

```
VPC: 10.0.0.0/16
├── Public Subnet: 10.0.1.0/24  (has route to Internet Gateway)
│   ├── EC2 (web server with public IP)
│   └── NAT Gateway
└── Private Subnet: 10.0.2.0/24 (no direct internet access)
    ├── EC2 (app servers)
    └── RDS (database)
```

### Key AWS Networking Components

| Component | Purpose |
|---|---|
| **VPC** | Isolated virtual network |
| **Subnet** | Segment of a VPC (public or private) |
| **Internet Gateway (IGW)** | Allows public internet traffic in/out |
| **NAT Gateway** | Lets private subnets access internet (outbound only) |
| **Route Table** | Controls where traffic goes |
| **Security Group** | Stateful instance-level firewall |
| **NACL** | Stateless subnet-level firewall |
| **Elastic IP** | Static public IP address |
| **VPC Peering** | Private connection between two VPCs |
| **Transit Gateway** | Hub to connect many VPCs + on-prem |
| **VPN Gateway** | Site-to-site VPN endpoint |
| **Direct Connect** | Dedicated private circuit to AWS |
| **Endpoint (VPC)** | Private access to AWS services (no internet) |

### Public vs Private Subnet

| | Public Subnet | Private Subnet |
|---|---|---|
| Internet Gateway route | ✅ Yes | ❌ No |
| Instances get public IP | Optional | No |
| Use cases | Load balancers, bastion hosts, NAT GW | App servers, databases |
| Internet access | Both in + out | Outbound only (via NAT) |

### Terraform: Basic VPC

```hcl
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
  tags = { Name = "main-vpc" }
}

resource "aws_subnet" "public" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.1.0/24"
  availability_zone       = "us-east-1a"
  map_public_ip_on_launch = true
}

resource "aws_internet_gateway" "igw" {
  vpc_id = aws_vpc.main.id
}

resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id
  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.igw.id
  }
}

resource "aws_route_table_association" "public" {
  subnet_id      = aws_subnet.public.id
  route_table_id = aws_route_table.public.id
}
```

---

## GCP Networking

### VPC Differences from AWS
- GCP VPCs are **global** (not region-specific like AWS)
- Subnets are regional but all belong to one VPC
- **No need for peering** between subnets in the same VPC

| Component | GCP | AWS equivalent |
|---|---|---|
| VPC | VPC Network | VPC |
| Firewall rules | Cloud Firewall | Security Groups + NACLs |
| Load Balancer | Cloud Load Balancing | ALB / NLB |
| Private Google Access | Cloud Private Google Access | VPC Endpoint |
| VPC Peering | VPC Network Peering | VPC Peering |
| Cloud Interconnect | Cloud Interconnect | Direct Connect |
| Cloud DNS | Cloud DNS | Route 53 |

---

## Azure Networking

| Component | Azure | AWS equivalent |
|---|---|---|
| Virtual Network | VNet | VPC |
| Subnet | Subnet | Subnet |
| NSG | Network Security Group | Security Group |
| VNet Peering | VNet Peering | VPC Peering |
| Azure DNS | Azure DNS | Route 53 |
| Application Gateway | App Gateway | ALB |
| Azure Load Balancer | Load Balancer | NLB |
| ExpressRoute | ExpressRoute | Direct Connect |

---

## Kubernetes Networking in Cloud

### Pod Networking
- Every pod gets a **unique IP**
- Pods can communicate with any other pod **without NAT**
- Nodes can communicate with any pod **without NAT**

### Cloud CNI Plugins
| Cloud | Default CNI |
|---|---|
| AWS EKS | Amazon VPC CNI (pods get real VPC IPs) |
| GCP GKE | Calico / Dataplane V2 (eBPF) |
| Azure AKS | Azure CNI |

### AWS EKS Networking
```bash
# Each pod gets a VPC IP from node's subnet
# Nodes need enough free IPs for max pods
# Formula: max_pods = (ENIs per instance × IPs per ENI) - 1
```

---

## Cloud Networking Best Practices

1. **Use private subnets** for databases and backend services
2. **Enable VPC Flow Logs** for security auditing and debugging
3. **Use VPC Endpoints** to access AWS services without internet
4. **Separate VPCs per environment** (dev, staging, prod) + use Transit Gateway
5. **Use multi-AZ** deployments for high availability
6. **Tag everything** — VPCs, subnets, security groups
7. **Restrict Security Groups** — avoid `0.0.0.0/0` for admin ports
8. **Use Private DNS** for internal service discovery

---

## Useful AWS CLI Commands

```bash
# List VPCs
aws ec2 describe-vpcs --query 'Vpcs[*].[VpcId,CidrBlock,Tags]'

# List subnets
aws ec2 describe-subnets

# List security groups
aws ec2 describe-security-groups

# List route tables
aws ec2 describe-route-tables

# Check VPC Flow Logs
aws ec2 describe-flow-logs

# Test connectivity with Reachability Analyzer
aws ec2 create-network-insights-path \
  --source <instance-id> \
  --destination <instance-id> \
  --protocol tcp
```
