# Study Guide & Interview Preparation

Comprehensive study guide for mastering computer networks as a DevOps/SRE engineer.

---

## Learning Path

### Phase 1: Foundations (2-3 hours)

**Goal**: Understand the big picture of networking

**Topics**:
1. Read: [OSI Model Overview](../01-Fundamentals/OSI-Model-Overview.md)
2. Watch: YouTube videos on OSI model (optional)
3. Review: Quick summary of each layer
4. Quiz yourself: What is each layer's responsibility?

**Key Takeaways**:
- ✓ 7 layers from physical to application
- ✓ Each layer adds headers (encapsulation)
- ✓ Top-down (sending) vs bottom-up (receiving)
- ✓ Lower layers enable higher layers

---

### Phase 2: Layer Understanding (5-7 hours)

**Goal**: Deep dive into each layer

**Sequence**:
```
Layer 1-3: Foundation (Network fundamentals)
├─ Layer 1: Physical (Cables, signals)
├─ Layer 2: Data Link (MAC, switches, frames)
└─ Layer 3: Network (IP, routing, ICMP)

Layer 4: Critical for DevOps
└─ Layer 4: Transport (TCP/UDP, ports, connections)

Layer 5-7: Application focus
├─ Layer 5: Session (State management)
├─ Layer 6: Presentation (Encryption, compression)
└─ Layer 7: Application (HTTP, DNS, SSH)
```

**Time Allocation**:
- Layer 1: 30 mins (overview, not critical for DevOps)
- Layer 2: 45 mins (switches, VLANs matter)
- Layer 3: 1.5 hours (IP, routing, crucial)
- Layer 4: 2 hours (Most critical, most complex)
- Layer 5: 20 mins (Less critical)
- Layer 6: 30 mins (TLS important)
- Layer 7: 1.5 hours (Application layer, very important)

**For Each Layer**:
1. Read the full file
2. Understand the function
3. Know the protocols
4. Learn DevOps relevance
5. Remember troubleshooting commands

---

### Phase 3: Practical Skills (4-5 hours)

**Goal**: Know how to troubleshoot in real world

**Resources**:
- [Common Network Commands](../03-Practical-Troubleshooting/Common-Network-Commands.md)
- [L4-L7 Troubleshooting](../03-Practical-Troubleshooting/L4-L7-Troubleshooting.md)
- [DevOps/SRE Scenarios](../03-Practical-Troubleshooting/DevOps-SRE-Scenarios.md)

**Practice**:
1. **Commands Lab**:
   - Set up test environment
   - Run each troubleshooting command
   - Understand the output
   - Create your personal cheat sheet

2. **Scenario Practice**:
   - Work through each scenario
   - Predict what's wrong before reading solution
   - Practice diagnostic thinking

3. **Hands-on**:
   - Set up services (web server, database)
   - Intentionally break them
   - Debug using networking knowledge
   - Document your process

---

### Phase 4: Advanced Topics (2-3 hours)

**Goal**: Understand complex networking patterns

**Topics**:
- [Core Networking Concepts](../04-Core-Concepts/Networking-Concepts.md)
- IP addressing deep dive
- Routing protocols
- QoS and traffic management
- Kubernetes networking (if relevant)
- Container networking

**Optional** (depends on role):
- Network security (firewalls, VPNs)
- Load balancing strategies
- Service meshes (Istio, Linkerd)
- Modern networking (5G, edge computing)

---

## Quick Review Checklist

**After each layer, verify you know**:

### Layer 1 (Physical)
- [ ] Data units: Bits
- [ ] Cable types and speeds
- [ ] Network devices: Hubs, repeaters
- [ ] Topologies: Star, bus, ring, mesh
- [ ] Transmission modes: Simplex, half-duplex, full-duplex

### Layer 2 (Data Link)
- [ ] Data units: Frames
- [ ] Addressing: MAC addresses (00:1A:2B:3C:4D:5E)
- [ ] Key device: Switches
- [ ] Error detection: CRC checksum
- [ ] Key protocol: Ethernet
- [ ] Technologies: VLANs, LAG

### Layer 3 (Network)
- [ ] Data units: Packets
- [ ] Addressing: IP (IPv4, IPv6)
- [ ] Key function: Routing
- [ ] Key protocols: IP, ICMP (ping), ARP
- [ ] Concepts: Subnetting, NAT, routing tables
- [ ] Troubleshooting: ping, traceroute, arp

### Layer 4 (Transport)
- [ ] Data units: Segments (TCP), Datagrams (UDP)
- [ ] Addressing: Ports (0-65535)
- [ ] TCP: Reliable, ordered, connection-oriented
- [ ] UDP: Fast, best-effort, connectionless
- [ ] Key concepts: 3-way handshake, flow control, congestion control
- [ ] Troubleshooting: netstat, ss, lsof

### Layer 5 (Session)
- [ ] Purpose: Maintain conversations
- [ ] Protocols: RPC, PPTP
- [ ] DevOps concern: Session timeout, sticky sessions

### Layer 6 (Presentation)
- [ ] Functions: Encryption, compression, formatting
- [ ] Key protocol: TLS/SSL
- [ ] DevOps focus: Certificate management, HTTPS

### Layer 7 (Application)
- [ ] Where users interact
- [ ] Key protocols: HTTP/HTTPS, DNS, SSH, SMTP, FTP
- [ ] Common ports: 80, 443, 22, 53, 25
- [ ] DevOps concern: Web server config, API monitoring

---

## Interview Preparation

### Common Interview Questions

**Easy (Warm-up)**

1. **"What is the OSI model?"**
   - 7 layers framework for network communication
   - Each layer independent, adds headers (encapsulation)
   - Physical to Application
   
2. **"What's the difference between TCP and UDP?"**
   - TCP: Reliable, ordered, connection-oriented
   - UDP: Best-effort, unordered, connectionless
   - TCP: Email, HTTP, SSH / UDP: DNS, Video, Gaming

3. **"What is a MAC address?"**
   - Layer 2 address, 48 bits (00:1A:2B:3C:4D:5E)
   - Scope: Local network only
   - Used by switches to forward frames

**Medium (Concepts)**

4. **"Explain the DNS resolution process"**
   - Client → Recursive resolver → Root nameserver
   - → TLD nameserver → Authoritative nameserver
   - Returns IP address to client
   - Caching at each step improves performance

5. **"How does a web request work from browser to server?"**
   - DNS: Resolve domain to IP
   - TCP: 3-way handshake (SYN, SYN-ACK, ACK)
   - TLS: Certificate handshake (if HTTPS)
   - HTTP: Send request, receive response
   - Render page

6. **"What's NAT and why is it used?"**
   - Translates private IPs to public IPs
   - Enables multiple devices to share one public IP
   - Common in homes and offices
   - Types: Static, Dynamic, PAT (masquerading)

**Hard (Troubleshooting)**

7. **"Service is not reachable, how do you debug?"**
   - Check if service is running: `systemctl status service`
   - Check if port is listening: `netstat -tlnp | grep :port`
   - Test connectivity: `telnet host port`
   - Check firewall: `iptables -L`
   - Verify network: `ping host`, `traceroute host`

8. **"High response time, where do you look?"**
   - Break down with curl: `curl -w "@fmt" http://service`
   - Time_namelookup → DNS issue (L3)
   - Time_connect → TCP issue (L4)
   - Time_appconnect → TLS issue (L6)
   - Time_starttransfer → Server processing (L7)

9. **"How do you monitor network health?"**
   - Connection tracking: `ss -ant | grep ESTABLISHED`
   - DNS queries: `dig`
   - Latency: `ping`, `mtr`
   - Packet capture: `tcpdump`
   - Throughput: `iperf3`

---

## Must-Know Commands

### Diagnosis

```bash
# Quick checks
ping 8.8.8.8                    # Basic connectivity
curl -v http://example.com      # Full request trace
nslookup example.com            # DNS check
ss -ant                         # All connections
netstat -tlnp | grep :8080      # Port listener
```

### Network Information

```bash
ip addr show                    # IP configuration
ip route show                   # Routing table
arp -a                          # ARP table
ss -tln                         # Listening ports
```

### Troubleshooting

```bash
curl -w "@curl-format" http://service  # Performance
tcpdump -i eth0 -n port 80             # Packet capture
traceroute example.com                  # Path trace
dig @8.8.8.8 example.com               # DNS query
```

### Performance

```bash
mtr example.com                 # Latency + loss
iperf3 -c host                  # Bandwidth test
ss -opt | grep ESTAB            # RTT per connection
watch -n 1 'ss -ant | wc -l'    # Connection count
```

---

## DevOps/SRE Specific Focus

**What's most important for your role?**

### Container/Kubernetes DevOps

- [ ] Container networking (bridge, overlay)
- [ ] Kubernetes service discovery (DNS)
- [ ] Network policies for security
- [ ] Ingress controllers
- [ ] Service mesh basics (Istio)

**Study**: Container networking deep dive, Kubernetes networking

### Infrastructure/Platform DevOps

- [ ] VPC/VNet configuration
- [ ] Subnetting and CIDR planning
- [ ] NAT and security groups
- [ ] Load balancer configuration
- [ ] VPN and site-to-site connectivity

**Study**: IP addressing, routing, NAT, firewalls

### SRE/Observability Focus

- [ ] Network monitoring and alerting
- [ ] Latency tracking and optimization
- [ ] Packet loss detection
- [ ] DNS health checks
- [ ] Connection pool management

**Study**: Troubleshooting guides, monitoring metrics

### Security/Compliance Focus

- [ ] Firewall rules
- [ ] Encryption (TLS)
- [ ] Network segmentation (VLANs)
- [ ] DDoS mitigation
- [ ] Compliance requirements

**Study**: L6-L7, security protocols, firewalls

---

## Practice Exercises

### Exercise 1: Network Troubleshooting

**Scenario**: Application can't reach database

```bash
# Step 1: Check connectivity
ping database-ip

# Step 2: Check port
telnet database-ip 3306

# Step 3: Check DNS
nslookup database-hostname

# Step 4: Check from different location
ssh app-server
curl database-ip:3306

# Step 5: Check application logs
kubectl logs app-pod

# Document findings and solution
```

### Exercise 2: Performance Analysis

**Scenario**: API responding slow

```bash
# Measure where time is spent
curl -w "DNS: %{time_namelookup}
Connect: %{time_connect}
Total: %{time_total}
" http://api

# If DNS slow:
dig @8.8.8.8 api-domain  # Use different DNS

# If connect slow:
mtr api-host             # Check path latency

# If total slow:
curl -w "@curl-format" http://api  # Detailed breakdown
```

### Exercise 3: Connection Debugging

**Scenario**: High connection count, application sluggish

```bash
# Count connections
ss -ant | grep ESTABLISHED | wc -l

# Check states
ss -ant | awk 'NR>1 {print $1}' | sort | uniq -c

# Find leaks
ss -ant | grep TIME_WAIT | wc -l

# Identify slow consumers
ss -opt | tail -20
```

---

## Memorization Tips

### Layer Mnemonic

**"Please Do Not Throw Sausage Pizza Away"**

```
P - Physical (Layer 1)
D - Data Link (Layer 2)
N - Network (Layer 3)
T - Transport (Layer 4)
S - Session (Layer 5)
P - Presentation (Layer 6)
A - Application (Layer 7)
```

### Key Port Numbers

```
22: SSH (Secure Shell)
53: DNS
80: HTTP
443: HTTPS
3306: MySQL
5432: PostgreSQL
6379: Redis
8080: HTTP Alternate
```

### TCP Handshake

```
SYN → "Can we talk?"
SYN-ACK ← "Yes, let's talk"
ACK → "Great, let's start"
```

### Error Code Ranges

```
1xx: Info (continue)
2xx: Success (got it!)
3xx: Redirect (go there)
4xx: Client error (your fault)
5xx: Server error (my fault)
```

---

## Pre-Interview Checklist

**1 Week Before**

- [ ] Review all 7 OSI layers (quick summary)
- [ ] Know TCP vs UDP differences
- [ ] Understand 5 troubleshooting scenarios
- [ ] Practice 10 common commands
- [ ] Review IP addressing basics

**2-3 Days Before**

- [ ] Do practice scenarios
- [ ] Time yourself on answers
- [ ] Prepare examples from your work
- [ ] Review interview questions
- [ ] Test all commands on actual system

**Day Before**

- [ ] Light review (don't overload)
- [ ] Get good sleep
- [ ] Prepare stories about real issues solved
- [ ] Review your DevOps/SRE specific topics

**During Interview**

- [ ] Listen carefully to the problem
- [ ] Ask clarifying questions
- [ ] Think about layers bottom-up (L1 → L7)
- [ ] Share your thinking process
- [ ] Don't pretend to know if unsure

---

## Resources for Deeper Learning

### Official & Technical

- RFC 793 (TCP specification)
- RFC 791 (IP specification)
- IEEE 802.3 (Ethernet)
- IANA Service Name Registry

### Books

- "Computer Networking: A Top-Down Approach" by Kurose & Ross
- "TCP/IP Illustrated" by Stevens
- "UNIX Network Programming" by Stevens & Fenner

### Websites

- GeeksforGeeks: Computer Networks
- Cisco Learning Network
- CloudFlare Blog: DNS, CDN articles
- Dev.to: DevOps networking posts

### Practice Platforms

- HackTheBox: Networking challenges
- PicoCTF: Network packet analysis
- Coursera: Networking courses
- YouTube: Network tutorials

---

## Test Yourself

### Quick Quiz

```
Q1: What layer is IP?
Q2: What layer is HTTP?
Q3: TCP needs 3-way handshake, UDP needs?
Q4: What's the subnet mask for /24?
Q5: How many usable IPs in 192.168.1.0/24?
Q6: What's the scope of MAC address?
Q7: What protocol does 'ping' use?
Q8: What's the default route 0.0.0.0/0 mean?
Q9: In TCP handshake, what's second message?
Q10: What layer is TLS/SSL?
```

### Answers (No Cheating!)

```
A1: Layer 3 (Network)
A2: Layer 7 (Application)
A3: None (connectionless)
A4: 255.255.255.0
A5: 254 (256 - 2 for network/broadcast)
A6: Local network (not routable)
A7: ICMP
A8: Default gateway (everything else)
A9: SYN-ACK
A10: Layer 6 (Presentation)
```

**Scoring**:
- 10/10: Excellent, interview ready
- 8-9/10: Good, review weak areas
- 6-7/10: Study more before interview
- <6/10: Need focused study time

---

## Final Tips

1. **Don't memorize**, understand concepts
2. **Practice troubleshooting**, not just theory
3. **Know your tools**: netstat, curl, dig, tcpdump
4. **Test on real systems**, not just theory
5. **Work through scenarios**, discuss your thinking
6. **Learn from failures**, they're learning opportunities
7. **Keep a notebook** of issues you've solved
8. **Stay curious**, networking keeps evolving
9. **Focus on your role**, DevOps/SRE priorities
10. **Have patience**, mastery takes time

---

**Remember**: 
> "Networking knowledge makes you invaluable as a DevOps/SRE engineer. Invest the time to learn it well."

Good luck with your interview! 🚀
