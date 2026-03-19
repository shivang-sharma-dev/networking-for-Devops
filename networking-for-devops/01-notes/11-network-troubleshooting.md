# 11 — Network Troubleshooting

> When the network breaks (and it will), you need a systematic approach.
> Guessing wastes hours. This methodology finds the problem in minutes.

---

## The Golden Rule

```
ALWAYS work bottom-up through the OSI layers.

Start at Layer 1 (physical).
If that works → go to Layer 2.
Keep going until you find where it breaks.

Layer 1: Can I reach the gateway? (physical/link)
Layer 3: Can I reach the IP?      (routing)
Layer 4: Can I reach the port?    (firewall/service)
Layer 7: Does the app respond?    (application)
```

---

## The Troubleshooting Flowchart

```
"I can't reach the server"
           │
           ▼
┌─────────────────────────────────────────────────────────┐
│ Step 1: Am I on the network?                            │
│   ip a                    ← do I have an IP address?   │
│   ip route                ← do I have a default route? │
│   ping 192.168.1.1        ← can I reach my gateway?    │
│                                                         │
│   FAIL → interface down, DHCP failed, cable unplugged  │
└────────────────────────┬────────────────────────────────┘
                         │ pass
                         ▼
┌─────────────────────────────────────────────────────────┐
│ Step 2: Can I reach the internet by IP?                 │
│   ping 8.8.8.8            ← can I reach Google by IP?  │
│                                                         │
│   FAIL → routing issue, ISP/cloud firewall, NAT broken │
└────────────────────────┬────────────────────────────────┘
                         │ pass
                         ▼
┌─────────────────────────────────────────────────────────┐
│ Step 3: Does DNS resolve?                               │
│   ping google.com         ← does the name resolve?     │
│   dig google.com          ← detailed DNS check         │
│                                                         │
│   FAIL → DNS server unreachable, /etc/resolv.conf bad  │
└────────────────────────┬────────────────────────────────┘
                         │ pass
                         ▼
┌─────────────────────────────────────────────────────────┐
│ Step 4: Can I reach the specific port?                  │
│   nc -zv TARGET_HOST PORT ← is the port open?          │
│   curl -v http://host:port← test HTTP specifically     │
│                                                         │
│   FAIL → firewall blocking, service not running,       │
│           wrong port, security group                   │
└────────────────────────┬────────────────────────────────┘
                         │ pass
                         ▼
┌─────────────────────────────────────────────────────────┐
│ Step 5: Is the app responding correctly?                │
│   curl -v https://host    ← full HTTP check            │
│   Check app logs          ← journalctl -u myapp        │
│                                                         │
│   FAIL → application error, wrong config, crash        │
└─────────────────────────────────────────────────────────┘
```

---

## The Full Toolkit

### Connectivity
```bash
ping 8.8.8.8                    # basic reachability (ICMP)
ping -c 4 google.com            # 4 packets, also tests DNS
ping -f 192.168.1.1             # flood ping (test packet loss, root only)

traceroute google.com           # trace every hop
traceroute -T google.com        # use TCP instead of ICMP (bypasses some firewalls)
mtr google.com                  # live traceroute with packet loss stats
mtr --report google.com         # one-shot report
```

### DNS
```bash
dig google.com                  # full DNS response
dig +short google.com           # just the IP
dig @8.8.8.8 google.com         # test with specific DNS server
dig +trace google.com           # full resolution chain
nslookup google.com             # simple lookup
cat /etc/resolv.conf            # what DNS servers am I using?
resolvectl status               # systemd-resolved status
```

### Port and Service Testing
```bash
nc -zv host 443                 # is TCP port open?
nc -zuv host 53                 # is UDP port open?
curl -v http://host:8080        # test HTTP
curl -Iv https://host           # test HTTPS (follow redirects)
telnet host 80                  # old way (less useful)
ss -tulnp                       # what is listening locally?
ss -tnp state established       # active connections
```

### Routing
```bash
ip route                        # routing table
ip route get 8.8.8.8            # which route for this destination?
route -n                        # old way
traceroute google.com           # see actual path taken
```

### Packet Capture
```bash
sudo tcpdump -i eth0                        # capture all traffic
sudo tcpdump -i eth0 host 8.8.8.8          # filter by host
sudo tcpdump -i eth0 port 80               # filter by port
sudo tcpdump -i eth0 'tcp port 443'        # HTTPS only
sudo tcpdump -i eth0 -w capture.pcap       # save to file (open with Wireshark)
sudo tcpdump -i any -n port 53             # DNS queries on any interface
sudo tcpdump -i eth0 'tcp[tcpflags] & (tcp-syn) != 0'  # only SYN packets
```

### Interface and Stats
```bash
ip a                            # interfaces and IPs
ip link show                    # interface state (UP/DOWN)
ip -s link show eth0            # interface statistics (errors, drops)
ethtool eth0                    # physical link info (speed, duplex)
netstat -s                      # protocol statistics
cat /proc/net/dev               # raw interface counters
```

---

## Common Scenarios and Fixes

### "ping works but curl doesn't"
```bash
# DNS resolves but HTTP fails → port 80 blocked
nc -zv host 80              # is port 80 reachable?
curl -v http://host         # verbose curl — see exactly where it fails
sudo iptables -L -n | grep 80
sudo ufw status
# → Check firewall rules on both client and server
```

### "connection refused"
```bash
# Port is blocked or service is not running
ss -tulnp | grep 8080       # is service actually listening?
systemctl status myapp      # is the service running?
journalctl -u myapp -n 50   # check service logs for errors
# If service is down → start it
# If running but not listening → check app config (PORT env var?)
```

### "connection timed out"
```bash
# Firewall is silently dropping packets (no RST, just silence)
nc -zv -w 5 host 8080       # timeout after 5 seconds
traceroute host             # where does it stop?
# On AWS → check security group inbound rules
# On Linux → check iptables / ufw
```

### "DNS resolution failed"
```bash
ping 8.8.8.8                # does IP connectivity work?
dig @8.8.8.8 google.com     # bypass local DNS — does this work?
cat /etc/resolv.conf        # is a nameserver configured?
systemctl status systemd-resolved
resolvectl flush-caches     # clear DNS cache
# If dig @8.8.8.8 works but dig google.com doesn't → local resolver issue
```

### "intermittent packet loss"
```bash
mtr --report -c 100 host    # send 100 pings, show packet loss per hop
# Loss at every hop from point X → problem at X
# Loss only at one hop but not after → that router limits ICMP (not a real problem)
ping -f -c 1000 host 2>&1 | tail -3  # flood ping statistics
```

### "SSL certificate error"
```bash
openssl s_client -connect host:443 -servername host
# Check:
#   Verify return code: 0 (ok) = certificate is valid
#   Expiry date in the output
#   Subject Alternative Names include your domain

# Quick expiry check
echo | openssl s_client -connect host:443 2>/dev/null | openssl x509 -noout -dates
```

### "slow network"
```bash
# Find the slow hop
mtr host                    # look for high latency at specific hops

# Bandwidth test
iperf3 -s                   # on server
iperf3 -c SERVER_IP         # on client

# Check for packet loss (causes TCP retransmission = slowness)
ping -c 100 host | tail -3

# Check interface errors
ip -s link show eth0
# errors/dropped non-zero = hardware problem
```

---

## Quick Diagnostic One-Liners

```bash
# Full network status snapshot
echo "=== IP ===" && ip a
echo "=== Routes ===" && ip route
echo "=== DNS ===" && cat /etc/resolv.conf
echo "=== Listening ===" && ss -tulnp
echo "=== Ping GW ===" && ping -c 2 $(ip route | grep default | awk '{print $3}')
echo "=== Ping DNS ===" && ping -c 2 8.8.8.8
echo "=== DNS resolve ===" && dig +short google.com

# Who is connecting to my server right now
ss -tn state established | awk '{print $5}' | cut -d: -f1 | sort | uniq -c | sort -rn

# Top talkers (install iftop or nethogs)
sudo nethogs eth0

# Check if specific IP is blocked by firewall
sudo iptables -L INPUT -n -v | grep 1.2.3.4
```
