# 09 — Load Balancing

## What is Load Balancing?

**Load balancing** distributes incoming traffic across multiple servers to ensure no single server is overwhelmed.

```
                    ┌──► Server 1 (healthy)
Client ──► [LB] ───┼──► Server 2 (healthy)
                    └──► Server 3 (healthy)
```

Benefits:
- **High availability** — if one server fails, traffic continues
- **Scalability** — add servers without changing client config
- **Performance** — distribute CPU/memory load

---

## L4 vs L7 Load Balancing

| | L4 (Transport) | L7 (Application) |
|---|---|---|
| Works at | TCP/UDP level | HTTP/HTTPS level |
| Routing based on | IP, Port | URL path, headers, cookies |
| TLS termination | No (passthrough) | Yes |
| Session persistence | IP-based | Cookie-based |
| Speed | Faster | Slightly slower |
| AWS equivalent | NLB | ALB |

---

## Load Balancing Algorithms

| Algorithm | Description | Best for |
|---|---|---|
| **Round Robin** | Sends to next server in sequence | Stateless, equal servers |
| **Least Connections** | Sends to server with fewest active connections | Long-lived connections |
| **IP Hash** | Routes based on client IP hash | Session persistence |
| **Weighted Round Robin** | Servers get % of traffic based on weight | Mixed server sizes |
| **Random** | Random selection | Simple, low-traffic |
| **Least Response Time** | Routes to fastest-responding server | Performance-sensitive |

---

## Health Checks

Load balancers use **health checks** to remove unhealthy servers from rotation.

```
LB: "GET /health HTTP/1.1 → Server1"
Server1: "200 OK" → HEALTHY ✓
Server2: "503 Service Unavailable" → UNHEALTHY ✗ (removed from pool)
```

### Health Check Types
- **HTTP/HTTPS**: Check for 200 OK on `/health` endpoint
- **TCP**: Check if port is open
- **Custom**: Check response body content

---

## AWS Elastic Load Balancers

### ALB (Application Load Balancer) — Layer 7
- Routes based on **URL path**, **host header**, **query strings**, **HTTP methods**
- Supports **gRPC**, **WebSockets**
- **Target groups**: EC2 instances, containers (ECS/EKS), Lambda, IPs

```
Rule: IF path = /api/* → Target Group: api-servers
Rule: IF path = /static/* → Target Group: cdn-origin
Rule: DEFAULT → Target Group: web-servers
```

### NLB (Network Load Balancer) — Layer 4
- Ultra-low latency, millions of requests per second
- Static IP support, Elastic IP support
- Good for: TCP/UDP traffic, gaming, IoT, databases

### CLB (Classic Load Balancer) — Legacy
- Old, avoid for new projects

---

## Nginx as a Load Balancer

```nginx
upstream backend {
    least_conn;                    # Algorithm
    server 10.0.1.10:8080;
    server 10.0.1.11:8080;
    server 10.0.1.12:8080 backup; # Only used if others fail
}

server {
    listen 80;

    location / {
        proxy_pass http://backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    location /health {
        return 200 "OK";
    }
}
```

---

## HAProxy as a Load Balancer

```haproxy
frontend http_front
    bind *:80
    default_backend http_back

backend http_back
    balance roundrobin
    option httpchk GET /health
    server server1 10.0.1.10:8080 check
    server server2 10.0.1.11:8080 check
    server server3 10.0.1.12:8080 check
```

---

## Sticky Sessions (Session Persistence)

Some apps require a client to always hit the **same server** (e.g., if session is stored in memory).

- **Cookie-based**: LB inserts a cookie with server ID
- **IP-based**: Same client IP always goes to same server
- ⚠️ Reduces load distribution effectiveness — prefer **stateless apps** or **shared session stores (Redis)**

---

## DevOps Best Practices

- Use **health check endpoints** in every service (`/health`, `/ready`, `/live`)
- Prefer **stateless services** to avoid sticky session issues
- Use **ALB** for HTTP apps, **NLB** for TCP/UDP
- Enable **access logs** on ALBs for debugging
- Use **connection draining** — let in-flight requests finish before removing a server

```bash
# Test load balancer is distributing (watch server IDs change)
for i in {1..10}; do curl -s https://api.example.com/health; echo; done
```
