# 06 — TCP & UDP

## Overview

| | TCP | UDP |
|---|---|---|
| Full name | Transmission Control Protocol | User Datagram Protocol |
| Connection | Connection-oriented | Connectionless |
| Reliability | Guaranteed delivery | Best-effort (no guarantee) |
| Order | Ordered | No ordering |
| Error checking | Yes | Minimal (checksum only) |
| Speed | Slower | Faster |
| Use cases | HTTP, SSH, FTP, databases | DNS, video streaming, gaming, VoIP |

---

## TCP — Deep Dive

### 3-Way Handshake (Connection Setup)

```
Client              Server
  │── SYN ──────────►│   "I want to connect, seq=100"
  │◄─ SYN-ACK ────── │   "OK, seq=300, ack=101"
  │── ACK ──────────►│   "Acknowledged, ack=301"
  │                  │
  │══ Data flows ══ ││
```

### 4-Way Handshake (Connection Teardown)

```
Client              Server
  │── FIN ──────────►│
  │◄─ ACK ─────────  │
  │◄─ FIN ─────────  │
  │── ACK ──────────►│
```

---

## TCP Features

### Sequence Numbers
- Every byte is numbered so the receiver can **reorder** out-of-sequence packets

### Flow Control (Sliding Window)
- Receiver tells sender how much buffer space it has (`window size`)
- Prevents the sender from overwhelming the receiver

### Congestion Control
- TCP detects network congestion and **slows down** transmission
- Algorithms: Slow Start, AIMD, CUBIC, BBR

### Retransmission
- If ACK not received within timeout → packet is **retransmitted**

---

## TCP Flags

| Flag | Meaning |
|---|---|
| **SYN** | Synchronize, start connection |
| **ACK** | Acknowledge received data |
| **FIN** | Finish, close connection gracefully |
| **RST** | Reset, forcefully close |
| **PSH** | Push data immediately to app |
| **URG** | Urgent data |

---

## UDP — Deep Dive

UDP sends packets with **no ordering, no acknowledgment, no retransmission**.

```
Client                 Server
  │── Datagram1 ──────►│
  │── Datagram2 ──────►│   (may arrive out of order or be lost)
  │── Datagram3 ──────►│
```

### Why UDP is Useful
- **Low latency** — no round trips for handshake/ACKs
- **Real-time** — in video calls, a dropped frame is better than a delayed one
- **Application-level control** — apps like QUIC build their own reliability on top of UDP

---

## Common Use Cases

### Use TCP when:
- Data must arrive **completely and in order**
- HTTP/HTTPS, SSH, FTP, SMTP, database connections

### Use UDP when:
- Speed > reliability
- DNS queries, NTP, VoIP, video streaming, online gaming, TFTP

---

## Port States

```bash
# Check listening ports
ss -tulnp
netstat -tulnp

# Check established connections
ss -tnp state established

# Count connections per state
ss -s
```

### Port States Explained
| State | Meaning |
|---|---|
| `LISTEN` | Waiting for incoming connections |
| `ESTABLISHED` | Active connection |
| `TIME_WAIT` | Waiting after close (prevents stale packets) |
| `CLOSE_WAIT` | Remote side closed, local closing |
| `SYN_SENT` | Local sent SYN, waiting for SYN-ACK |

---

## QUIC (HTTP/3)

**QUIC** is a transport protocol built on UDP, developed by Google, now the basis for HTTP/3.

- Combines TLS handshake + transport handshake in **1 round trip**
- Solves **head-of-line blocking** (a TCP problem)
- Better performance on **mobile and lossy networks**

---

## DevOps Relevance

```bash
# Test TCP connectivity to a port
nc -zv google.com 443
curl -v telnet://google.com:443

# Test UDP connectivity
nc -zvu 8.8.8.8 53

# Capture TCP traffic on port 80
sudo tcpdump -i eth0 tcp port 80

# See TIME_WAIT connections (important for high-traffic servers)
ss -tan | grep TIME-WAIT | wc -l
```

> **Key DevOps scenario**: If your service has too many `TIME_WAIT` connections, you may be opening a new TCP connection per request. Use **connection pooling** or **keep-alive** to reuse connections.
