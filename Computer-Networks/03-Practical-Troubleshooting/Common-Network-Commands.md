# Common Network Commands - DevOps/SRE Reference

Quick reference for essential networking commands used in troubleshooting and monitoring.

---

## Network Information & Diagnostics

### IP Configuration

```bash
# View IP addresses
ip addr show
ip addr show eth0
hostname -I

# Old style (still works)
ifconfig
ifconfig eth0

# View detailed info
ip link show
ip -s link show                    # With statistics
```

### Network Interface Management

```bash
# Bring interface up/down
sudo ip link set eth0 up
sudo ip link set eth0 down

# Change IP temporarily
sudo ip addr add 192.168.1.100/24 dev eth0
sudo ip addr del 192.168.1.100/24 dev eth0

# Change default gateway
sudo ip route add default via 192.168.1.1
sudo ip route del default

# Make permanent (edit config)
# /etc/netplan/01-netcfg.yaml (Ubuntu)
# /etc/sysconfig/network-scripts/ifcfg-eth0 (CentOS)
```

### Routing

```bash
# View routing table
ip route show
route -n                           # Numeric format
netstat -r                         # Alternative

# Add route
sudo ip route add 10.0.0.0/24 via 192.168.1.1

# Add default gateway
sudo ip route add default via 192.168.1.1

# Delete route
sudo ip route del 10.0.0.0/24

# Check specific route
ip route get 8.8.8.8               # Which route used?

# Policy routing (advanced)
sudo ip rule add from 192.168.1.1 table 100
sudo ip route add default via 192.168.1.1 table 100
```

### Connectivity Testing

```bash
# Ping (ICMP, Layer 3)
ping -c 4 8.8.8.8                  # 4 packets
ping -i 0.5 8.8.8.8                # 0.5s interval
ping -t 64 8.8.8.8                 # TTL=64

# Ping statistics only
ping -c 100 8.8.8.8| tail -1      # Show summary

# Traceroute (Layer 3)
traceroute google.com
traceroute -m 10 google.com        # Max 10 hops
tracert google.com                 # Windows

# Modern alternative (TCP instead of UDP)
mtr google.com                     # Like traceroute but continuous
mtr -c 10 google.com               # 10 cycles
mtr -r -c 100 google.com           # Report mode

# Test port reachability (Layer 4)
timeout 5 bash -c 'echo > /dev/tcp/8.8.8.8/53' && echo OK || echo FAIL

# Telnet (interactive, Layer 4)
telnet google.com 80
# Ctrl+] then quit

# SSH port check
ssh -v -p 22 user@host 2>&1 | grep -i "Connection"
```

---

## DNS Troubleshooting

### DNS Queries

```bash
# Simple DNS lookup
nslookup google.com
nslookup google.com 8.8.8.8        # Use specific DNS server

# More detailed (dig is better)
dig google.com
dig @8.8.8.8 google.com            # Specific DNS server
dig google.com +short              # Minimal output
dig google.com +noall +answer      # Answer only

# Trace DNS resolution path
dig +trace google.com
dig +trace @8.8.8.8 google.com

# Query specific record type
dig google.com A                    # IPv4
dig google.com AAAA                # IPv6
dig google.com MX                  # Mail server
dig google.com NS                  # Name servers
dig google.com TXT                 # Text records
dig google.com SOA                 # Start of Authority
dig google.com CNAME               # Alias

# Reverse DNS (IP to domain)
dig -x 8.8.8.8
nslookup 8.8.8.8

# Host command (simple)
host google.com
host 8.8.8.8

# Check DNS server configuration
cat /etc/resolv.conf
systemctl status systemd-resolved

# Flush DNS cache
sudo systemd-resolve --flush-caches
sudo systemd-resolve --statistics   # Show cache status
```

### DNS Debugging

```bash
# Check all DNS servers configured
resolvectl
systemctl status systemd-resolved

# Test resolution with different servers
for ns in 8.8.8.8 1.1.1.1 9.9.9.9; do
  echo "Testing $ns:"
  dig @$ns google.com +short
done

# Monitor DNS queries
sudo tcpdump -i eth0 -n 'port 53'

# Check DNS propagation worldwide
# Use online tools: whatsmydns.net
```

---

## Network Connectivity & Ports

### Viewing Connections

```bash
# List all listening ports
ss -tlnp                           # Modern (socket statistics)
netstat -tlnp                      # Traditional (net statistics)
lsof -i -P -n | grep LISTEN       # Using lsof

# List all established connections
ss -ant | grep ESTABLISHED
ss -ant | head -20

# Count connections by state
ss -ant | awk 'NR>1 {print $1}' | sort | uniq -c

# Find connections to specific port
ss -ant | grep :8080
netstat -an | grep :8080

# Find what process is using a port
lsof -i :8080                      # What's on 8080?
lsof -i tcp:8080
ss -tlnp | grep :8080

# Watch connection states in real-time
watch -n 1 'ss -ant | tail -20'
watch -n 1 'ss -ant | grep -c ESTABLISHED'
```

### Connection Details

```bash
# All TCP connections with details
ss -ant
netstat -an

# Connection states
ss -ant | awk 'NR>1 {print $1}' | sort | uniq -c

# Connections in TIME_WAIT state
ss -ant | grep TIME_WAIT | wc -l
netstat -an | grep TIME_WAIT | wc -l

# Connections to specific host
ss -ant | grep 192.168.1.1
netstat -an | grep 192.168.1.1

# SSL/TLS connection info
ss -opt                            # TCP options including RTT
ss -opt | grep ESTABLISHED | head -5
```

### Port/Process Management

```bash
# Kill process on specific port
pid=$(lsof -ti :8080)
kill $pid
kill -9 $pid                       # Force kill

# Or use fuser
fuser -k 8080/tcp                  # Kill process on port 8080

# Change service port
# Edit configuration and restart
sudo vim /etc/service/config.conf
sudo systemctl restart service-name

# Find available ports
# Check which ports are listening
ss -tlnp | awk '{print $4}' | grep -o ':[0-9]*' | sort -u
```

---

## Packet Inspection & Capture

### tcpdump (Most Common)

```bash
# Basic capture
sudo tcpdump -i eth0               # All traffic on eth0

# Limit by protocol
sudo tcpdump -i eth0 tcp           # TCP only
sudo tcpdump -i eth0 udp           # UDP only
sudo tcpdump -i eth0 icmp          # ICMP (ping) only

# Limit by host
sudo tcpdump -i eth0 host 8.8.8.8
sudo tcpdump -i eth0 src 192.168.1.1
sudo tcpdump -i eth0 dst 8.8.8.8

# Limit by port
sudo tcpdump -i eth0 port 80
sudo tcpdump -i eth0 port 443
sudo tcpdump -i eth0 'tcp port 8080'

# Combined filters
sudo tcpdump -i eth0 'tcp port 80 and host 8.8.8.8'
sudo tcpdump -i eth0 'udp port 53'   # DNS queries

# Save to file
sudo tcpdump -i eth0 -w traffic.pcap port 80
# Analyze later
tcpdump -r traffic.pcap | head -20

# Live display
sudo tcpdump -i eth0 -A port 80    # Show payload
sudo tcpdump -i eth0 -X port 80    # Hex and ASCII

# Verbosity
sudo tcpdump -i eth0 -v             # Verbose
sudo tcpdump -i eth0 -vv            # Very verbose

# Count packets
sudo tcpdump -i eth0 -c 100         # Capture 100 packets
```

### Wireshark (GUI)

```bash
# Launch Wireshark
wireshark &

# Capture from command line
tshark -i eth0 -f 'port 80' -w capture.pcap

# Analyze pcap file
tshark -r traffic.pcap
tshark -r traffic.pcap -Y 'http.request'
```

---

## HTTP/HTTPS Testing

### curl (Most Useful)

```bash
# Simple GET
curl http://example.com

# GET with verbose output
curl -v http://example.com
curl -V http://example.com         # More verbose

# Show only response headers
curl -I http://example.com

# Show only status code
curl -s -o /dev/null -w "%{http_code}" http://example.com

# Custom headers
curl -H "X-Custom-Header: value" http://example.com
curl -H "Authorization: Bearer token" http://example.com

# POST request
curl -X POST -d "param=value" http://example.com
curl -X POST -H "Content-Type: application/json" \
  -d '{"key":"value"}' http://example.com

# Save response to file
curl http://example.com -o output.html

# Follow redirects
curl -L http://example.com

# Timeout
curl --max-time 5 http://example.com

# User authentication
curl -u username:password http://example.com

# Measure performance
curl -w "DNS: %{time_namelookup}s\nConnect: %{time_connect}s\nTotal: %{time_total}s\n" \
  -o /dev/null -s http://example.com

# HTTPS certificate
curl -I https://example.com        # Basic
curl -v https://example.com 2>&1 | grep "subject"
```

### wget (Similar to curl)

```bash
# Simple GET
wget http://example.com -O output.html

# Verbose
wget -v http://example.com

# With timeout
wget --timeout=5 http://example.com

# Follow redirects (default)
wget -L http://example.com
```

### ab (Apache Bench - Load Testing)

```bash
# Basic load test
ab -n 1000 -c 10 http://example.com/
# -n 1000: Total requests
# -c 10: Concurrent requests

# With different concurrency
ab -n 1000 -c 50 http://example.com/

# Save results
ab -n 1000 -c 10 http://example.com/ > results.txt
```

---

## Network Monitoring & Statistics

### netstat (Now ss - Modern)

```bash
# All protocols
ss -a
netstat -a

# TCP only
ss -at
netstat -at

# Listening ports
ss -tl
netstat -tl

# Numeric format (faster)
ss -tlnp
netstat -tlnp

# UDP sockets
ss -ul
netstat -ul

# Statistics
ss -s                              # Summary statistics
netstat -s                         # Detailed statistics
netstat -s | grep -i tcp
netstat -s | grep -i udp
```

### Interface Statistics

```bash
# View interface stats
ip -s link
ip -s addr

# Detailed stats
ethtool eth0
ethtool -S eth0                    # Extended statistics

# Watch bandwidth usage
watch -n 1 'ip -s link show eth0'

# iftop (Top for network interfaces - if installed)
iftop -i eth0
```

### ARP (Address Resolution Protocol)

```bash
# View ARP table
arp -a                             # Traditional
ip neigh                           # Modern
ip neigh show

# ARP to specific host
arp -a 192.168.1.1
ip neigh show to 192.168.1.1

# Add ARP entry (static)
sudo arp -s 192.168.1.100 00:11:22:33:44:55
sudo ip neigh add to 192.168.1.100 lladdr 00:11:22:33:44:55 dev eth0

# Delete ARP entry
sudo arp -d 192.168.1.1
sudo ip neigh del to 192.168.1.1 dev eth0

# Flush ARP table
sudo ip neigh flush dev eth0
```

---

## Firewall & iptables

### iptables (Linux)

```bash
# View current rules
sudo iptables -L
sudo iptables -L -v
sudo iptables -L -n                # Numeric
sudo iptables -L -n -v             # Numeric + verbose

# View specific chain
sudo iptables -L INPUT
sudo iptables -L OUTPUT
sudo iptables -L FORWARD

# View with line numbers
sudo iptables -L -n --line-numbers

# Allow port
sudo iptables -A INPUT -p tcp --dport 8080 -j ACCEPT
sudo iptables -A OUTPUT -p tcp --sport 8080 -j ACCEPT

# Block IP
sudo iptables -A INPUT -s 192.168.1.1 -j DROP

# Allow from IP
sudo iptables -A INPUT -s 192.168.1.1 -j ACCEPT

# Save rules (persist)
# Ubuntu/Debian
sudo apt-get install iptables-persistent
sudo netfilter-persistent save

# CentOS/RHEL
sudo service iptables save
```

### firewalld (Modern)

```bash
# View status
sudo firewall-cmd --state
sudo firewall-cmd --list-all

# Add port
sudo firewall-cmd --add-port=8080/tcp
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --reload

# Remove port
sudo firewall-cmd --remove-port=8080/tcp

# Add service
sudo firewall-cmd --add-service=http
sudo firewall-cmd --permanent --add-service=https

# List services
sudo firewall-cmd --get-services

# View zones
sudo firewall-cmd --list-all-zones
```

### ufw (Ubuntu Firewall)

```bash
# Status
sudo ufw status
sudo ufw status verbose

# Enable/Disable
sudo ufw enable
sudo ufw disable

# Allow port
sudo ufw allow 8080
sudo ufw allow 8080/tcp
sudo ufw allow from 192.168.1.1 to any port 8080

# Deny port
sudo ufw deny 8080

# View rules
sudo ufw show added
```

---

## Network Configuration Files

### Linux Network Config Locations

```bash
# Modern (Netplan - Ubuntu 18.04+)
/etc/netplan/01-netcfg.yaml

# Traditional (CentOS/RHEL)
/etc/sysconfig/network-scripts/ifcfg-eth0

# Debian/Ubuntu (older)
/etc/network/interfaces

# DNS configuration
/etc/resolv.conf                   # DNS servers (auto-generated)
/etc/systemd/resolved.conf         # systemd-resolved config

# Hosts file
/etc/hosts                         # Local IP to hostname mapping

# Network services
/etc/services                      # Port number to service name mapping
/etc/protocols                     # Protocol numbers
```

### systemd-networkd (Modern)

```bash
# View status
systemctl status systemd-networkd
systemctl status systemd-resolved

# Restart networking
sudo systemctl restart systemd-networkd

# View network info
networkctl
networkctl status eth0
```

---

## Performance Tuning

### TCP/IP Tuning

```bash
# View current settings
sysctl net.ipv4.tcp_window_scaling
sysctl net.core.rmem_default
sysctl net.core.wmem_default

# Temporarily change
sudo sysctl -w net.ipv4.tcp_tw_reuse=1

# Permanently change
sudo vim /etc/sysctl.conf
# Add: net.ipv4.tcp_tw_reuse = 1
sudo sysctl -p                     # Reload

# View all network settings
sudo sysctl -a | grep net
```

### Connection Limits

```bash
# View file descriptor limits
ulimit -n

# Increase file descriptors
ulimit -n 65536

# Permanent (per user)
echo "username soft nofile 65536" | sudo tee -a /etc/security/limits.conf
echo "username hard nofile 65536" | sudo tee -a /etc/security/limits.conf

# System-wide
sudo vim /etc/sysctl.conf
# Add: fs.file-max = 2097152
sudo sysctl -p
```

---

## Kubernetes-Specific Commands

```bash
# Check service DNS
nslookup service-name.namespace.svc.cluster.local
dig service-name.namespace.svc.cluster.local

# Pod connectivity
kubectl exec -it pod-name -- bash
# Inside pod: curl service-name

# Port forwarding
kubectl port-forward svc/service-name 8080:80

# Check endpoints
kubectl get endpoints service-name
kubectl describe endpoints service-name

# View network policies
kubectl get networkpolicies
kubectl describe networkpolicy policy-name

# Check DNS service
kubectl logs -n kube-system -l k8s-app=kube-dns

# Check service discovery
kubectl get svc
kubectl describe svc service-name
```

---

## Container-Specific Commands

```bash
# Docker network
docker network ls
docker network inspect bridge
docker inspect container-name | grep Networks

# Container connectivity
docker exec -it container-name bash
# Inside: curl http://other-container-ip

# Port mapping
docker port container-name

# Network logs
docker logs -f container-name
```

---

## Quick Diagnosis Commands

```bash
# Everything is slow
ping 8.8.8.8                       # Check latency
traceroute 8.8.8.8                 # Check path
curl -w "@fmt" http://example.com  # Check response time

# Can't reach service
curl -v http://service:port        # Check L7
telnet service port                # Check L4
ping service                       # Check L3

# DNS not working
dig example.com                    # Check DNS
nslookup example.com               # Alternative
cat /etc/resolv.conf               # Check config

# Port conflicts
lsof -i :8080                      # What's on port?
ss -tlnp | grep 8080               # Connection info
kill pid                           # Kill process

# Too many connections
ss -ant | grep -c ESTABLISHED      # Count connections
ss -ant | grep TIME_WAIT | wc -l   # TIME_WAIT count
sysctl net.ipv4.tcp_max_syn_backlog # Check backlog
```

---

## Useful One-Liners

```bash
# Monitor network in real-time
watch -n 1 'ss -ant | tail -20'

# Count connections by state
ss -ant | awk 'NR>1 {print $1}' | sort | uniq -c

# Find and kill process on port
kill $(lsof -ti :8080)

# Monitor bandwidth
while true; do clear; ss -ant | grep -c ESTABLISHED; sleep 1; done

# Check which DNS server is being used
systemctl status systemd-resolved | grep -i dns

# Test DNS from multiple servers
for ns in 8.8.8.8 1.1.1.1; do echo "$ns:"; dig @$ns example.com +short; done

# Monitor certificate expiration
for domain in $(cat domains.txt); do 
  echo -n "$domain: "
  echo | openssl s_client -servername $domain -connect $domain:443 2>/dev/null | openssl x509 -noout -dates
done
```

---

**Note**: Replace `eth0` with your actual interface name. Check with `ip link show`.

**Pro Tip**: Create shell aliases for frequently used commands:
```bash
alias netstat='ss'
alias getport='lsof -i'
alias curltime='curl -w "Total: %{time_total}s\n"'
```
