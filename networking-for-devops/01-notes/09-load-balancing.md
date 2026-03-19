# 09 — Load Balancing

> Load balancers are the key to scalability. Without one, your app
> has a single point of failure. With one, you can scale to millions
> of users and survive server failures without downtime.

---

## The Problem Load Balancing Solves

```
WITHOUT A LOAD BALANCER:

  1000 users ──────────────────► Single server
                                  (overloaded, single point of failure)

  Problems:
    One server handles ALL traffic
    Server goes down → app goes down
    Can't scale horizontally

WITH A LOAD BALANCER:

                          ┌──► Server 1 (handles ~333 users)
  1000 users ──► LB ──────┼──► Server 2 (handles ~333 users)
                          └──► Server 3 (handles ~334 users)

  Benefits:
    Traffic distributed evenly
    Server dies → LB routes to healthy servers
    Add more servers → LB sends them traffic
```

---

## Layer 4 vs Layer 7 Load Balancing

This is the most common interview question about load balancers.

```
LAYER 4 (Transport Layer — TCP/UDP)
  ────────────────────────────────
  Works at the TCP level.
  Sees: source IP, destination IP, ports.
  Does NOT look inside the packet.

  Analogy: A post office that sorts by ZIP code only.
           It doesn't open envelopes.

  Pros: Very fast, low overhead
  Cons: Cannot make routing decisions based on content

  Examples:
    AWS Network Load Balancer (NLB)
    HAProxy (TCP mode)


LAYER 7 (Application Layer — HTTP)
  ─────────────────────────────────
  Works at the HTTP level.
  Sees: HTTP method, URL path, headers, cookies.
  Can route based on content.

  Analogy: A smart receptionist who reads your request
           and sends you to the right department.

  Pros: Smart routing, SSL termination, can inspect traffic
  Cons: More overhead than L4

  Examples:
    AWS Application Load Balancer (ALB)
    nginx
    HAProxy (HTTP mode)
    Traefik
```

---

## Load Balancing Algorithms

```
ROUND ROBIN
  Requests go to each server in rotation: 1, 2, 3, 1, 2, 3...
  Simple. Good when all servers are equal capacity.

LEAST CONNECTIONS
  New request goes to server with fewest active connections.
  Better when requests have varying processing times.

IP HASH
  Client's IP is hashed → always same server for same client.
  Good when you need session persistence (sticky sessions).

WEIGHTED ROUND ROBIN
  Assign weights: Server1=3, Server2=1.
  Server1 gets 3 requests for every 1 to Server2.
  Use when servers have different capacities.

RANDOM
  Randomly pick a server.
  Surprisingly effective at scale.

LEAST RESPONSE TIME
  Send to server with fastest recent response time.
  Best for performance-critical applications.
```

---

## Health Checks

A load balancer constantly checks if backends are healthy.
If one fails, it stops sending traffic there.

```
Load Balancer
    │
    ├── every 30s → GET /health → Server 1
    │               200 OK ← healthy
    ├── every 30s → GET /health → Server 2
    │               200 OK ← healthy
    └── every 30s → GET /health → Server 3
                    Connection refused ← UNHEALTHY

LB removes Server 3 from rotation.
Traffic goes to Server 1 and Server 2 only.
When Server 3 recovers → LB adds it back automatically.
```

```
Common health check types:
  HTTP  → GET /health → expect 200 OK
  TCP   → can we connect to the port?
  HTTPS → GET /health with TLS verification

Your app MUST expose a health check endpoint.
It should return 200 if healthy, 5xx if unhealthy.

Good /health endpoint checks:
  → Can I connect to the database?
  → Can I connect to Redis/cache?
  → Am I running out of memory?
  → Return 200 only if all dependencies are OK
```

---

## SSL/TLS Termination

The load balancer handles TLS encryption/decryption.
Backend servers receive plain HTTP — simpler and faster.

```
Client ──HTTPS──► Load Balancer ──HTTP──► Backend Servers
         (encrypted)  (decrypt here)  (plain text, fast)

Benefits:
  Certificates managed in one place (not on every server)
  Backends don't spend CPU on encryption
  Easier certificate renewal

Alternative: End-to-end encryption (HTTPS to backend too)
  Use when compliance requires it (PCI-DSS, HIPAA)
```

---

## Sticky Sessions

Some apps store session data in memory. The same user must
always hit the same server, or they lose their session.

```
WITHOUT sticky sessions:
  User logs in → Server 1 (stores session)
  Next request → Load balancer → Server 2
  User appears logged out! (Server 2 has no session)

WITH sticky sessions (session persistence):
  User logs in → Server 1
  All future requests → always Server 1 (LB uses cookie/IP hash)

Better solution: Store sessions in Redis (shared cache)
  Then any server can serve any user.
  No sticky sessions needed.
  Much more reliable.
```

---

## nginx as a Load Balancer

```nginx
# /etc/nginx/nginx.conf

upstream myapp {
    # Load balancing algorithm (default is round robin)
    least_conn;                         # or: ip_hash; or: random;

    server 10.0.1.10:8080 weight=3;    # gets 3x traffic
    server 10.0.1.11:8080 weight=1;
    server 10.0.1.12:8080;
    server 10.0.1.13:8080 backup;      # only used if others fail
    server 10.0.1.14:8080 down;        # manually disabled
}

server {
    listen 80;

    location / {
        proxy_pass http://myapp;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

        # Health check (nginx Plus only — else use upstream_check module)
        health_check interval=10 fails=3 passes=2;
    }
}
```

---

## AWS Load Balancers

```
ALB (Application Load Balancer) — Layer 7
  Use for: HTTP/HTTPS web apps, microservices, containers
  Can route based on: URL path, hostname, HTTP headers, query strings
  Supports: WebSockets, HTTP/2, gRPC
  
  Path-based routing:
    /api/*     → API server target group
    /static/*  → Static file server target group
    /*         → Main app target group

  Host-based routing:
    api.example.com   → API target group
    www.example.com   → Web target group

NLB (Network Load Balancer) — Layer 4
  Use for: Ultra-high performance, TCP/UDP, static IP needed
  Handles millions of requests per second
  Preserves client IP address
  
CLB (Classic Load Balancer) — Legacy
  Avoid for new deployments. Use ALB or NLB.
```

---

## Global Load Balancing

```
Users in multiple regions need low latency:

  User in India     ──► AWS Mumbai (ap-south-1)
  User in US        ──► AWS Virginia (us-east-1)
  User in Europe    ──► AWS Frankfurt (eu-central-1)

Solutions:
  AWS Route 53 Latency-based routing
  AWS Global Accelerator
  Cloudflare Load Balancing
  
Traffic flows to the nearest healthy region automatically.
If one region goes down, traffic routes to next nearest.
```
