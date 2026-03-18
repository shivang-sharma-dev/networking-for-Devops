# 🗺️ Learning Roadmap — Networking for DevOps

A structured, milestone-based path to go from networking beginner to cloud-native networking confident.

---

## 🎯 Learning Goals

By the end of this roadmap you should be able to:
- Explain any networking concept in an interview
- Design and troubleshoot network architectures in AWS/GCP/Azure
- Configure Kubernetes networking and service meshes
- Use networking tools fluently (curl, dig, tcpdump, nmap, etc.)

---

## 📅 Phase 1 — Foundations (Week 1–2)

> **Goal**: Build a solid mental model of how networks work.

- [ ] Networking Fundamentals
- [ ] OSI Model & TCP/IP Stack
- [ ] IP Addressing & Subnetting (CIDR)
- [ ] DNS — How domain resolution works
- [ ] HTTP vs HTTPS — TLS, certificates
- [ ] TCP vs UDP — When to use which

**Milestone**: Explain the full journey of a browser request (DNS → TCP → HTTP → Response)

---

## 📅 Phase 2 — Infrastructure Networking (Week 3–4)

> **Goal**: Understand how traffic flows inside and between systems.

- [ ] Routing & Switching
- [ ] Firewalls & Security Groups
- [ ] Load Balancing (L4 vs L7)
- [ ] VPN & Tunneling (WireGuard, IPsec, SSH tunnels)
- [ ] Network troubleshooting tools

**Milestone**: Diagnose a broken network path using only CLI tools

---

## 📅 Phase 3 — Cloud Networking (Week 5–6)

> **Goal**: Apply networking concepts in major cloud providers.

- [ ] AWS VPC — Subnets, Route Tables, IGW, NAT, Security Groups, NACLs
- [ ] GCP VPC — VPC Peering, Cloud DNS, Cloud Load Balancing
- [ ] Azure VNet — NSG, Application Gateway
- [ ] DNS in Cloud — Route 53, Cloud DNS
- [ ] Multi-region & Hybrid networking

**Milestone**: Build a multi-tier VPC from scratch using Terraform

---

## 📅 Phase 4 — Container & Kubernetes Networking (Week 7–8)

> **Goal**: Master networking in containerized environments.

- [ ] Docker Networking (bridge, host, overlay)
- [ ] Kubernetes Networking Model
- [ ] CNI Plugins — Calico, Flannel, Cilium
- [ ] Services — ClusterIP, NodePort, LoadBalancer
- [ ] Ingress Controllers — Nginx, Traefik
- [ ] Network Policies
- [ ] Service Mesh — Istio / Linkerd

**Milestone**: Deploy an app with proper Ingress + TLS + Network Policies

---

## 📅 Phase 5 — Advanced & Interview Prep (Week 9–10)

> **Goal**: Get job-ready and interview-confident.

- [ ] Complete interview Q&A
- [ ] System design problems (design a CDN, global load balancer)
- [ ] Mock troubleshooting scenarios
- [ ] Complete at least 2 projects

**Milestone**: Ace a DevOps networking interview round

---

## 🏅 Recommended Certifications

| Certification | Provider | Relevance |
|---|---|---|
| CCNA | Cisco | Networking fundamentals |
| AWS ANS-C01 | Amazon | Cloud networking deep dive |
| GCP Professional Network Engineer | Google | GCP networking |
| CKA (Certified Kubernetes Administrator) | CNCF | K8s networking |
