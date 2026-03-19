# 🟢 Beginner Project: Diagnose a Network Issue

## Project Overview

**Goal**: Use CLI tools to diagnose and document a real-world network connectivity problem.

**Skills practiced**: ping, traceroute, dig, curl, nc, ss

**Time estimate**: 1–2 hours

---

## Scenario

Your web application at `http://localhost:8080` is not responding. Users are reporting errors. You need to systematically diagnose the issue, identify the root cause, and fix it.

---

## Setup

```bash
# Create a failing app scenario
docker run -d --name broken-app \
  -p 8080:9999 \    # Wrong port mapping (app listens on 80, not 9999)
  nginx

# Now try to access it:
curl http://localhost:8080
```

---

## Tasks

### Task 1: Layer 3 Check
```bash
# Is the host reachable at all?
ping localhost

# Document what you find:
```

### Task 2: Layer 4 Check
```bash
# Is the port open?
nc -zv localhost 8080

# What ports are actually listening?
ss -tlnp

# Document what you find:
```

### Task 3: Application Layer Check
```bash
# What error does curl give?
curl -v http://localhost:8080

# Document what you find:
```

### Task 4: Find the Root Cause
```bash
# What port is the nginx container actually exposing?
docker ps
docker inspect broken-app | grep -i port

# Fix the port mapping:
docker stop broken-app && docker rm broken-app
docker run -d --name fixed-app -p 8080:80 nginx

# Verify fix:
curl http://localhost:8080
```

### Task 5: DNS Check (Bonus)
```bash
# What does your system resolve google.com to?
dig google.com +short

# What nameserver is being used?
cat /etc/resolv.conf

# Test alternate DNS vs your default
time dig @8.8.8.8 google.com
time dig @1.1.1.1 google.com
# Which is faster?
```

---

## Deliverable

Write a short report (in this directory as `report.md`) documenting:
1. What you found at each layer
2. What the root cause was
3. How you fixed it
4. What you would monitor to detect this earlier

---

## Bonus Challenges

- Set up UFW rules to block and unblock the port, validating with `nc`
- Capture the traffic on the working endpoint with `tcpdump`
- Write a simple bash health check script that alerts if the port is down
