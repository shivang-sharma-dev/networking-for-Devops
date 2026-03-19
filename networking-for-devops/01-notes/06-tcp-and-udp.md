# 06 — TCP and UDP

> TCP and UDP are the two workhorses of the transport layer.
> Choosing between them (or understanding why a protocol chose one)
> is a question that comes up in every DevOps and backend interview.

---

## The Core Difference — One Analogy

```
TCP = Registered Mail
  ────────────────────
  You get a receipt when you send it.
  The post office confirms delivery.
  If the letter is lost, it is re-sent.
  Letters arrive in order.
  Slower. More overhead.
  Use when: the content MUST arrive correctly.


UDP = Dropping leaflets from a plane
  ──────────────────────────────────
  You throw them out and hope they land.
  No confirmation of delivery.
  No re-sending if they are lost.
  They may land in random order.
  Faster. No overhead.
  Use when: speed matters more than guaranteed delivery.
```

---

## TCP — Transmission Control Protocol

TCP guarantees three things:
1. **Delivery** — lost packets are retransmitted
2. **Order** — packets are reassembled in the correct sequence
3. **Error checking** — corrupted data is detected and rejected

### The TCP 3-Way Handshake

Before any data is sent, TCP establishes a connection.

```
CLIENT                              SERVER
──────                              ──────

1. SYN ─────────────────────────►
   "I want to connect.
    My sequence starts at 1000"

2.              ◄─────────────────── SYN-ACK
                                     "OK, I'm ready.
                                      My sequence starts at 5000.
                                      I got your 1000 (ACK=1001)"

3. ACK ─────────────────────────►
   "Got it. You start at 5000.
    (ACK=5001)"

Connection established. Data flows now.
```

```bash
# Watch the handshake happen in real time
sudo tcpdump -i eth0 host google.com -n
# You will see: SYN, SYN-ACK, ACK, data, data, FIN, ACK
```

### TCP Connection Teardown (4-Way)

```
CLIENT                              SERVER
──────                              ──────
FIN ────────────────────────────►
◄───────────────────────────────── ACK
◄───────────────────────────────── FIN
ACK ────────────────────────────►

Four steps because each direction closes independently.
```

### TCP Connection States

```bash
ss -tn           # see TCP connection states

# States you will see:
LISTEN        Service is waiting for connections
ESTABLISHED   Active connection, data flowing
TIME_WAIT     Connection closed, waiting for late packets (60s)
CLOSE_WAIT    Remote end closed, local not yet
SYN_SENT      We sent SYN, waiting for SYN-ACK
FIN_WAIT_1    We sent FIN, waiting for ACK

# Too many TIME_WAIT = high connection turnover (normal for web servers)
# Too many CLOSE_WAIT = your app is not closing connections properly
ss -s           # summary of all connection states
```

---

## UDP — User Datagram Protocol

UDP just sends packets. No handshake. No confirmation. No ordering.

```
CLIENT                              SERVER
──────                              ──────

Packet 1 ───────────────────────►
Packet 2 ───────────────────────►
Packet 3 ───────────────────────►  (arrives as Packet 3, 1, 2)

No connection setup.
No "did you get it?"
No re-sending.
Server receives whatever arrives.
```

---

## When to Use Each

```
USE TCP WHEN:                       USE UDP WHEN:
─────────────                       ─────────────
Data must arrive correctly          Speed is more important
Order matters                       Some loss is acceptable
Web browsing (HTTP/HTTPS)           Video streaming (Netflix)
File transfer (FTP, SCP)            Online gaming
Email (SMTP)                        VoIP calls
Database queries                    DNS queries (small, fast)
SSH                                 DHCP
API calls                           Live metrics/telemetry
                                    IoT sensor data

WHY DNS USES UDP:
  DNS query is tiny (a few bytes).
  If the response is lost, just ask again.
  No need for TCP overhead of a 3-way handshake.
  Result: DNS over UDP is ~10x faster.
  (DNS falls back to TCP for large responses > 512 bytes)

WHY VIDEO STREAMING USES UDP:
  A dropped frame is better than a frozen video.
  Retransmitting old frames would make the video worse.
  The app handles recovery at the application layer.
```

---

## TCP vs UDP Comparison

```
┌────────────────────┬─────────────────────┬──────────────────────┐
│ Feature            │ TCP                 │ UDP                  │
├────────────────────┼─────────────────────┼──────────────────────┤
│ Connection         │ Connection-oriented │ Connectionless       │
│ Reliability        │ Guaranteed delivery │ Best effort only     │
│ Ordering           │ Ordered             │ Unordered            │
│ Error checking     │ Yes (retransmit)    │ Checksum only        │
│ Speed              │ Slower              │ Faster               │
│ Header size        │ 20-60 bytes         │ 8 bytes              │
│ Flow control       │ Yes                 │ No                   │
│ Congestion control │ Yes                 │ No                   │
│ Use case           │ Accuracy needed     │ Speed needed         │
└────────────────────┴─────────────────────┴──────────────────────┘
```

---

## Ports in TCP/UDP

Both TCP and UDP use port numbers to identify services.

```
A connection is uniquely identified by 5 values:
  Protocol + Source IP + Source Port + Dest IP + Dest Port

  TCP + 192.168.1.5:52341 + 142.250.80.46:443
  TCP + 192.168.1.5:52342 + 142.250.80.46:443
  TCP + 192.168.1.5:52343 + 74.125.200.100:80

Your browser can open many tabs simultaneously because
each connection has a different source port.
```

---

## TCP Flow Control and Congestion Control

```
FLOW CONTROL — don't overwhelm the receiver
  Receiver advertises a "window size" (how much it can buffer).
  Sender only sends that much before waiting for ACK.

  Slow receiver → small window → sender slows down
  Fast receiver → large window → sender speeds up

CONGESTION CONTROL — don't overwhelm the network
  Starts slow (slow start) and increases speed.
  If packet loss is detected, backs off immediately.
  Algorithms: Reno, Cubic, BBR (used by Google/Linux)

This is why TCP connections "warm up" — they start slow
and accelerate. Short-lived connections never reach full speed.
HTTP/2 persistent connections exploit this — keep one fast connection
instead of opening many slow ones.
```

---

## Practical Commands

```bash
# See all TCP/UDP listening ports
ss -tulnp

# See established TCP connections
ss -tnp state established

# Count connections by state
ss -tan | awk '{print $1}' | sort | uniq -c

# See UDP activity
ss -unp

# Monitor specific port (watch connections come and go)
watch -n 1 'ss -tn dst :443 | wc -l'

# Capture TCP handshakes
sudo tcpdump -i eth0 'tcp[tcpflags] & (tcp-syn) != 0'

# Test TCP connection to a port
nc -zv google.com 443       # TCP
nc -zuv 8.8.8.8 53          # UDP

# Simulate packet loss (for testing)
sudo tc qdisc add dev eth0 root netem loss 10%   # add 10% packet loss
sudo tc qdisc del dev eth0 root                  # remove
```

---

## Common TCP Issues in Production

```
Too many TIME_WAIT connections
  → High-traffic web server — normal
  → Tune: net.ipv4.tcp_tw_reuse=1 in sysctl
  → Or use HTTP keep-alive to reuse connections

Connection refused (port closed)
  → Service not running
  → Wrong port
  → Check: ss -tulnp | grep <port>

Connection timeout (no response)
  → Firewall dropping packets silently
  → Server overloaded
  → Wrong IP/hostname
  → Check: traceroute to the host

SYN flood / DDoS
  → Attacker sends many SYN packets without completing handshake
  → Server fills up SYN backlog
  → Fix: enable SYN cookies
  → sysctl net.ipv4.tcp_syncookies=1
```
