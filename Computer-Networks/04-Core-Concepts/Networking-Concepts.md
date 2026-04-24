# Core Networking Concepts for DevOps/SRE

Essential concepts and topics that appear across multiple OSI layers.

---

## TCP/IP Model vs OSI Model

### Overview

```
OSI (Theoretical)          TCP/IP (Practical - Used Today)
─────────────────         ──────────────────────────────
Layer 7: Application       │
Layer 6: Presentation      ├─→ Application Layer
Layer 5: Session           │
                           │
Layer 4: Transport         ─→ Transport Layer (TCP, UDP)
                           │
Layer 3: Network           ─→ Internet Layer (IP)
                           │
Layer 2: Data Link         ├─→ Link Layer
Layer 1: Physical          │
```

### Why TCP/IP Dominates

- **Simpler**: 4-5 layers vs 7 layers
- **Practical**: Real-world implementation
- **Current Standard**: Internet uses TCP/IP
- **De facto**: Industry standard

### Mapping

| Layer | TCP/IP | OSI | Examples |
|-------|--------|-----|----------|
| 4-5 | Application | L7, L6, L5 | HTTP, DNS, SSH |
| 3 | Transport | L4 | TCP, UDP |
| 2 | Internet | L3 | IP, ICMP |
| 1 | Link | L2, L1 | Ethernet, WiFi |

---

## Network Addressing Deep Dive

### IPv4 Address Space

```
Total: 4.3 billion addresses (2^32)
Distributed as:
- Class A: ~16.7 million (16%)
- Class B: ~1 million (23%)
- Class C: ~2.1 million (61%)

Private (Non-routable):
- 10.0.0.0/8 (Class A): 16.7 million
- 172.16.0.0/12 (Part of Class B): 1 million
- 192.168.0.0/16 (Class C): 65,536

Special:
- 127.0.0.0/8: Loopback (localhost)
- 0.0.0.0/8: Current network
- 255.255.255.255/32: Broadcast
- 169.254.0.0/16: Link-local (APIPA)
```

### Subnetting Examples

```
Network: 192.168.1.0/24

Bits:    | Network    | Host   |
─────────┼────────────┼────────┤
Mask:    | 11111111 | 11111111 | 11111111 | 00000000
         | 255      | 255      | 255      | 0
IP:      | 11000000 | 10101000 | 00000001 | 00000000
         | 192      | 168      | 1        | 0

Network Address: 192.168.1.0
Broadcast: 192.168.1.255
Usable IPs: 192.168.1.1 - 192.168.1.254 (254 total)

Second subnet (if split):
192.168.1.0/25:
- Network: 192.168.1.0
- Broadcast: 192.168.1.127
- Usable: 192.168.1.1 - 192.168.1.126 (126 total)

192.168.1.128/25:
- Network: 192.168.1.128
- Broadcast: 192.168.1.255
- Usable: 192.168.1.129 - 192.168.1.254 (126 total)
```

### CIDR Calculation

```
/32 = 255.255.255.255 (Host) - 1 address
/31 = 255.255.255.254 (Link) - 2 addresses
/30 = 255.255.255.252 (Router) - 4 addresses (2 usable)
/29 = 255.255.255.248 - 8 addresses
/28 = 255.255.255.240 - 16 addresses
/27 = 255.255.255.224 - 32 addresses
/26 = 255.255.255.192 - 64 addresses
/25 = 255.255.255.128 - 128 addresses
/24 = 255.255.255.0 - 256 addresses (254 usable) - Common subnet
/23 = 255.255.254.0 - 512 addresses
/22 = 255.255.252.0 - 1024 addresses
/16 = 255.255.0.0 - 65,536 addresses (Class B)
/8 = 255.0.0.0 - 16.7M addresses (Class A)
```

---

## Routing Fundamentals

### Static Routing

```
Manual routing table entries
Used in small networks with known paths

Example:
Destination: 10.0.0.0/8     → Via 192.168.1.1
Destination: 172.16.0.0/12  → Via 192.168.1.2
Destination: 0.0.0.0/0      → Via 192.168.1.1 (default)

Advantage: Simple, no overhead
Disadvantage: Doesn't adapt to changes
```

### Dynamic Routing Protocols

#### RIP (Routing Information Protocol)

```
Metric: Hop count (max 15 hops)
Updates: Every 30 seconds (broadcast)
Convergence: Slow
Use: Small networks, legacy

Problems:
- Slow convergence on failures
- High bandwidth overhead
- Not scalable
```

#### OSPF (Open Shortest Path First)

```
Metric: Cost based on link speed/bandwidth
Updates: On changes (multicast)
Convergence: Fast
Use: Medium to large networks

Characteristics:
- Hierarchical (areas)
- Fast failure detection
- Complex configuration
- Better than RIP
```

#### BGP (Border Gateway Protocol)

```
Metric: Path attributes, policies
Updates: On changes (TCP connection)
Convergence: Slow but reliable
Use: Internet, ASN to ASN routing

Characteristics:
- Autonomous Systems (AS)
- Policy-based routing
- Used by ISPs
- Most complex
```

### Routing Decision Process

```
Packet arrives → Check Destination IP
                     ↓
              Longest prefix match?
                     ↓
              Match found? Send via interface
              No match?   Send to default gateway (0.0.0.0/0)
                     ↓
              Gateway forwards to next hop
                     ↓
              Repeat until destination reached
```

---

## NAT and Port Mapping

### Network Address Translation Types

#### Static NAT (1-to-1)

```
Private IP: 192.168.1.100
Public IP: 203.0.113.10

Always: 192.168.1.100 ↔→ 203.0.113.10

Use: Web server access, Static mapping
```

#### Dynamic NAT (Many-to-Many)

```
Private IPs: 192.168.1.0/24 (254 devices)
Public IPs: 203.0.113.0/25 (126 IPs)

First request: 192.168.1.1 → uses 203.0.113.1
              192.168.1.2 → uses 203.0.113.2
              ...
              192.168.1.127 → uses 203.0.113.126
              192.168.1.128 → waits (no public IP available)

Use: Corporate networks, ISP customers
```

#### PAT/Masquerading (Many-to-One)

```
Private IPs: 192.168.1.0/24 (254 devices)
Public IP: 203.0.113.10 (1 IP)

All requests → 203.0.113.10
Router tracks connections by:
- Source IP + Port → Destination IP + Port
- Maps back on response

Example:
192.168.1.100:52341 → 8.8.8.8:53   →  203.0.113.10:52341 → 8.8.8.8:53
8.8.8.8:53 → 203.0.113.10:52341  →  8.8.8.8:53 → 192.168.1.100:52341

Use: Home networks, Most common
```

---

## Quality of Service (QoS)

### Why QoS Needed

```
Network has limited bandwidth
Multiple applications competing
Fairness vs Priority

Without QoS:
- Video streaming might hog all bandwidth
- VoIP calls drop (needs low latency)
- File downloads succeed

With QoS:
- Video: Best effort (lower priority)
- VoIP: High priority (low latency/loss)
- File transfer: Bulk (fills unused bandwidth)
```

### QoS Mechanisms

#### Traffic Classification

```
Identify different traffic types:
- Source/dest IP
- Port numbers
- Protocol type
- Application signature
```

#### Prioritization

```
Queue different traffic separately:
Priority Queue: VoIP, video conferencing
Bulk Queue: File transfers, backups
Best effort: General traffic
```

#### Traffic Shaping

```
Limit traffic rate to match SLA
Token Bucket algorithm:
- Token = permission to send data
- Tokens generated at rate R
- Bucket capacity = Burst
```

### QoS in Cloud/Containers

```
Kubernetes: Network Policies define QoS
Istio: Rate limiting, circuit breaking
Cloud providers: Bandwidth guarantees (paid)

Example:
VoIP app: Max 100 Mbps
Database: Max 1 Gbps
Default: Fair share
```

---

## Security at Network Level

### Firewalls

#### Stateless Firewall

```
Checks each packet independently
Rule: If source=192.168.1.1 and port=80 → ACCEPT

Problem: Can't distinguish between:
- Legitimate return traffic
- Spoofed packets
```

#### Stateful Firewall

```
Tracks connection state
Rule: Allow traffic if part of established connection

More secure, slightly slower
Knows: Request out → Response back = legitimate
```

#### Application Layer Firewall (WAF)

```
Inspects application content
Can block SQL injection, XSS attacks
Example: ModSecurity, AWS WAF

vs Network firewall:
- Network: Blocks by IP/port
- Application: Blocks by content
```

### VPN (Virtual Private Network)

```
Encrypted tunnel over public network

Device 1 → [Encrypted] → VPN Gateway → [Encrypted] → Device 2
           ↑            ↑              ↑            ↑
        Private       Internet      Internet    Private
```

#### VPN Protocols

| Protocol | Encryption | Speed | Use |
|----------|-----------|-------|-----|
| **IPSec** | Strong | Good | Site-to-site |
| **TLS** | Strong | Slower | Client-to-site |
| **WireGuard** | Strong | Very fast | Modern, containers |
| **OpenVPN** | Strong | Good | Universal |

---

## Bandwidth & Throughput Optimization

### Theory vs Practice

```
Ethernet 1 Gbps = 1,000,000,000 bits/second
             = 125,000,000 bytes/second
             = ~119 MB/second (in practice)

Why the difference?
- Frame overhead (headers)
- Interframe gaps
- TCP/IP headers (40 bytes per packet)
- Collisions (half-duplex)

Real-world:
- 1Gbps link: 900-950 Mbps actual
- 10Gbps link: 9-9.5 Gbps actual
- WiFi 5: Advertised 1.3Gbps, actual 300-500Mbps
```

### Optimization Techniques

```
1. Large Frame Sizes
   Standard: 1500 bytes (Ethernet)
   Jumbo: Up to 9000 bytes
   Benefit: More payload, fewer packets, lower overhead

2. TCP Window Scaling
   Larger send/receive windows
   Better utilize high-latency links
   sysctl net.ipv4.tcp_window_scaling=1

3. Connection Aggregation
   Multiple TCP streams
   Use parallelism
   Example: 4 connections × 250 Mbps = 1 Gbps utilization

4. Compression
   gzip, brotli reduce payload size
   Trade: CPU for bandwidth

5. Protocol Choice
   QUIC (UDP): Better than TCP for lossy networks
   HTTP/2: Multiplexing reduces latency
   HTTP/3: Uses QUIC, even faster
```

---

## Network Monitoring Metrics

### Key Metrics to Monitor

```
Metric             | Warning | Critical | Tool
────────────────────────────────────────────────────
Latency (RTT)      | >100ms  | >500ms   | ping, mtr
Packet Loss        | >1%     | >5%      | mtr, iperf3
Jitter             | >50ms   | >200ms   | mtr
Throughput         | <80%BW  | <50%BW   | iperf3
DNS Resolution     | >100ms  | >500ms   | dig, nslookup
Connection Time    | >1s     | >5s      | curl -w
Error Rate         | >0.1%   | >1%      | TCP/IP stats
CPU (network)      | >60%    | >85%     | top, htop
```

### DevOps Monitoring Tools

```
Real-time:
- iftop: Bandwidth per connection
- nethogs: Bandwidth per process
- nettop: Network top
- tcpdump: Packet capture

Periodic:
- Prometheus: Metrics collection
- Grafana: Visualization
- Datadog: Full monitoring
- New Relic: APM + networking

Application:
- tcpdump + Wireshark: Deep inspection
- OpenTelemetry: Distributed tracing
- Envoy/Istio: Service mesh metrics
```

---

## Container & Kubernetes Networking

### Docker Networking Modes

```
Bridge (default):
- Container gets IP on docker0 bridge
- Can talk to host and other containers
- Port mapping via iptables

Host:
- Container uses host's network
- No isolation
- Performance benefit

Overlay:
- Across multiple hosts
- Service mesh like Swarm

None:
- No network
```

### Kubernetes Networking

```
Cluster Network:
- Every pod gets IP
- Any pod can reach any pod
- No NAT needed

Service DNS:
- service-name.namespace.svc.cluster.local
- DNS returns virtual IP (ClusterIP)
- kube-proxy handles routing

CNI (Container Network Interface):
- Flannel: Overlay networking
- Weave: Mesh networking
- Calico: Policy-based routing
```

---

## Common DevOps Networking Patterns

### Service Mesh Pattern

```
Instead of:          With Service Mesh:
App → App     →      App → (Proxy) → App
Direct calls         Intercepted calls
Limited visibility   Full observability
Difficult policies   Easy rate limiting
```

### API Gateway Pattern

```
External Traffic → API Gateway → Internal Services
                     ↑
                  - Rate limiting
                  - Authentication
                  - Load balancing
                  - Request routing
```

### Load Balancer Pattern

```
Clients → Load Balancer → Server Pool
            ↑
          - Health checks
          - Session affinity
          - SSL termination
```

---

## Interview Preparation

### Must Know Concepts

1. **OSI Model**: All 7 layers, responsibilities, examples
2. **IP Addressing**: CIDR, subnetting, private vs public
3. **TCP/UDP**: Differences, use cases, port numbers
4. **DNS**: Resolution process, record types, troubleshooting
5. **Routing**: Static vs dynamic, routing protocols
6. **NAT**: Types and use cases
7. **Firewalls**: Stateless vs stateful
8. **Load Balancing**: L4 vs L7, algorithms
9. **VLANs**: Logical segmentation
10. **Kubernetes Networking**: Services, CNI, DNS

### Common Interview Questions

```
1. "Explain the difference between TCP and UDP"
   → Reliable vs best-effort, ordered vs unordered
   
2. "How does DNS work?"
   → Recursive query, nameservers, caching
   
3. "What happens when you type 'google.com' in browser?"
   → DNS, TCP handshake, HTTP request, TLS
   
4. "How do you debug a 'connection refused' error?"
   → Is service running? Is port open? Firewall?
   
5. "Explain NAT"
   → Maps private IPs to public IPs, maintains connection state
   
6. "What's the difference between L4 and L7 load balancing?"
   → L4: TCP/port based, L7: HTTP/hostname based
```

---

**Next**: Review all OSI layer files and Troubleshooting guides for comprehensive understanding.
