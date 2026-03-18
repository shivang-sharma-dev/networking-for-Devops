# 11 — Network Troubleshooting

## Troubleshooting Mental Model

Always work **bottom-up** (OSI layers):
1. Is the physical/network connectivity OK? (`ping`, `ip link`)
2. Is DNS resolving? (`dig`, `nslookup`)
3. Is the port open/reachable? (`nc`, `telnet`, `curl`)
4. Is the app responding correctly? (`curl -v`, logs)

---

## Essential Troubleshooting Tools

### ping — Test Basic Connectivity
```bash
ping google.com           # ICMP echo test
ping -c 5 8.8.8.8         # 5 packets only
ping -i 0.2 8.8.8.8       # Faster ping (0.2s interval)
ping6 2001:db8::1         # IPv6 ping

# Interpret results:
# packet loss % → network issues
# TTL value → estimated distance (hops)
# time ms → latency
```

### traceroute / tracepath — Trace Network Path
```bash
traceroute google.com
traceroute -n google.com   # No DNS reverse lookup (faster)
tracepath google.com       # Similar, no root required

# * * * means: hop not responding to ICMP (firewall dropping it)
```

### mtr — Real-time Traceroute
```bash
mtr google.com
mtr --report google.com    # Generate report
mtr -n google.com          # No DNS resolution
```

### dig / nslookup — DNS Troubleshooting
```bash
dig google.com
dig @8.8.8.8 google.com    # Use specific DNS server
dig +trace google.com      # Full resolution trace
dig -x 8.8.8.8             # Reverse lookup
nslookup google.com
```

### curl — HTTP Troubleshooting
```bash
curl -v https://api.example.com         # Verbose output
curl -I https://api.example.com         # Headers only
curl -L https://example.com            # Follow redirects
curl --max-time 5 https://example.com   # Timeout after 5s
curl -w "DNS: %{time_namelookup}s, Connect: %{time_connect}s, Total: %{time_total}s\n" \
     -o /dev/null -s https://example.com
```

### nc (netcat) — Port Testing
```bash
nc -zv google.com 443        # Test TCP port
nc -zvu 8.8.8.8 53           # Test UDP port
nc -l 9000                    # Listen on port 9000
echo "hello" | nc host 9000   # Send data to port
```

### ss / netstat — Socket & Connection Info
```bash
ss -tulnp                    # All listening ports with process
ss -tnp state established    # Established connections
ss -s                        # Summary statistics
netstat -tulnp               # Same as ss (older)
netstat -rn                  # Routing table
```

### tcpdump — Packet Capture
```bash
# Capture all traffic on interface
sudo tcpdump -i eth0

# Filter by host
sudo tcpdump -i eth0 host 8.8.8.8

# Filter by port
sudo tcpdump -i eth0 port 80

# Capture to file for Wireshark
sudo tcpdump -i eth0 -w capture.pcap

# Read a capture file
tcpdump -r capture.pcap

# Show HTTP traffic content
sudo tcpdump -i eth0 -A port 80
```

### nmap — Port Scanning
```bash
nmap 192.168.1.1             # Scan common ports
nmap -p 1-65535 192.168.1.1  # Full port scan
nmap -sU 192.168.1.1         # UDP scan
nmap -sV 192.168.1.1         # Version detection
nmap 192.168.1.0/24          # Scan entire subnet
```

### ip — Interface & Route Management
```bash
ip addr show                 # All interfaces + IPs
ip link show                 # Interface states (UP/DOWN)
ip route show                # Routing table
ip neigh show                # ARP table
ip -s link                   # Interface statistics
```

---

## Common Troubleshooting Scenarios

### Scenario 1: Can't reach a website
```bash
# Step 1: Basic connectivity
ping 8.8.8.8
# If fails: network/routing issue

# Step 2: DNS resolution
dig google.com
# If fails: DNS issue (/etc/resolv.conf, DNS server)

# Step 3: TCP connection
nc -zv google.com 443
# If fails: firewall/security group blocking

# Step 4: Application layer
curl -v https://google.com
# If fails: TLS or app issue
```

### Scenario 2: Application can't connect to database
```bash
# Check if port is reachable from app server
nc -zv db-server 5432

# Check if DB is listening
ss -tlnp | grep 5432

# Check firewall rules
sudo iptables -L -n
sudo ufw status

# Check routing
ip route get <db-ip>
```

### Scenario 3: Slow network performance
```bash
# Check for packet loss
mtr --report google.com

# Check interface errors
ip -s link show eth0

# Check bandwidth
iperf3 -c server-ip         # Client
iperf3 -s                    # Server

# Check DNS resolution time
time dig google.com
```

### Scenario 4: DNS not resolving
```bash
# Check /etc/resolv.conf
cat /etc/resolv.conf

# Try alternative DNS
dig @1.1.1.1 google.com

# Check if DNS port is reachable
nc -zvu 8.8.8.8 53

# Flush DNS cache
sudo systemd-resolve --flush-caches
sudo killall -HUP dnsmasq
```

---

## Kubernetes Network Troubleshooting

```bash
# Check pod DNS
kubectl exec -it <pod> -- nslookup kubernetes.default

# Check pod-to-pod connectivity
kubectl exec -it <pod-a> -- curl http://<pod-b-ip>:8080

# Check service endpoints
kubectl get endpoints <service-name>

# Debug with a network toolbox pod
kubectl run tmp-shell --rm -i --tty --image nicolaka/netshoot -- /bin/bash

# Check network policies
kubectl get networkpolicies
kubectl describe networkpolicy <name>
```
