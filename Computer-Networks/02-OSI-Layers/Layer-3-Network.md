# Layer 3: Network Layer

## Definition

The **Network Layer** (Layer 3) is responsible for **routing** data across different networks. It uses **IP addresses** to deliver packets from source to destination across multiple hops (routers).

### Core Concept
```
Layer 2: "How to reach the next device on this segment?"
         Uses: MAC address

Layer 3: "How to reach the destination across the Internet?"
         Uses: IP address
         Device: Router
```

---

## Key Functions

### 1. **Routing**
- Determining best path to destination
- Using routing protocols and routing tables
- Example: OSPF, BGP, RIP

### 2. **Logical Addressing (IP)**
- IPv4: 32-bit address (e.g., 192.168.1.1)
- IPv6: 128-bit address (e.g., 2001:0db8::1)
- Hierarchical structure (Network + Host)

### 3. **IP Packet Forwarding**
- Forward packet from one network to another
- Decrement TTL (Time To Live)
- Perform NAT if configured

### 4. **Network Address Translation (NAT)**
- Translates private IPs to public IPs
- Enables multiple devices on one public IP
- Common in home networks and corporate LANs

---

## IP Addressing (IPv4)

### Address Structure

```
192.168.1.100

192      .     168      .     1       .     100
┌─────────────────────────┐  ┌──────────────┐
    Network Portion          Host Portion
    (Class C)                (256 possibilities)
```

### IP Address Classes (Classful)

| Class | Range | First Octet | Networks | Hosts/Network | Use |
|-------|-------|-------------|----------|--------------|-----|
| **A** | 1-126.x.x.x | 0xxxxxxx | 126 | 16.7M | Large |
| **B** | 128-191.x.x | 10xxxxxx | 16,384 | 65,534 | Medium |
| **C** | 192-223.x.x | 110xxxxx | 2.1M | 254 | Small |
| **D** | 224-239.x.x | 1110xxxx | - | - | Multicast |
| **E** | 240-255.x.x | 1111xxxx | - | - | Reserved |

### Classless Addressing (CIDR - Modern)

```
192.168.1.0/24

192.168.1.0 = Network address
/24 = Subnet mask (first 24 bits are network)

Benefits:
- More flexible than classful
- Reduces waste
- Better utilization
- Used everywhere today
```

### CIDR Notation Examples

```
/32 = 255.255.255.255 (Single host) - 1 IP
/30 = 255.255.255.252 (Router links) - 4 IPs (2 usable)
/24 = 255.255.255.0 (Subnet) - 256 IPs (254 usable)
/16 = 255.255.0.0 (Large network) - 65,536 IPs
/8 = 255.0.0.0 (Class A) - 16.7M IPs
```

### Subnet Calculation Example

```
Network: 192.168.1.0/24

Network Address: 192.168.1.0
First Host: 192.168.1.1
Last Host: 192.168.1.254
Broadcast: 192.168.1.255

Total IPs: 256 (2^8)
Usable IPs: 254 (256 - 2 for network and broadcast)
```

---

## Private vs Public IP Addresses

### Private IP Ranges (RFC 1918)

```
Class A: 10.0.0.0 to 10.255.255.255 (10.0.0.0/8)
Class B: 172.16.0.0 to 172.31.255.255 (172.16.0.0/12)
Class C: 192.168.0.0 to 192.168.255.255 (192.168.0.0/16)

Used for: Internal networks, not routable on internet
```

### Public IP Ranges

```
All other addresses are public
Routable on the internet
Assigned by ISP or IANA
Cost: Monthly fee
```

### Special Addresses

```
127.0.0.1 = Loopback (localhost)
0.0.0.0 = Default route
169.254.0.0/16 = Link-local (APIPA)
224.0.0.0/4 = Multicast
255.255.255.255 = Broadcast (local)
```

---

## IPv4 vs IPv6

### Comparison

| Aspect | IPv4 | IPv6 |
|--------|------|------|
| **Address Length** | 32 bits | 128 bits |
| **Format** | Decimal (192.168.1.1) | Hexadecimal (2001:db8::1) |
| **Addresses** | 4.3 billion | 340 undecillion |
| **Header Size** | 20 bytes | 40 bytes |
| **Checksum** | Yes | No |
| **Fragmentation** | Yes | Limited |
| **Status** | Dominant (still) | Growing |

### IPv6 Address Format

```
2001:0db8:0000:0000:0000:0000:0000:0001

Can be simplified:
2001:db8::1 (leading zeros removed, consecutive zeros = ::)

Prefix notation:
2001:db8::/32 (first 32 bits are network)
```

---

## Routing Concepts

### Routing Table Example

```
Destination    | Next Hop      | Interface | Metric
──────────────────────────────────────────────────
0.0.0.0/0      | 192.168.1.1  | eth0      | 1
192.168.0.0/24 | Direct       | eth0      | 0
192.168.1.0/24 | Direct       | eth1      | 0
10.0.0.0/8     | 192.168.1.2  | eth0      | 2
```

Router decision:
```
Packet arrives for 10.0.1.5
↓
Check routing table
↓
Match found: 10.0.0.0/8 → Next hop 192.168.1.2
↓
Forward to 192.168.1.2
```

### Static vs Dynamic Routing

| Type | Configuration | Updates | Best For |
|------|---------------|---------|----------|
| **Static** | Manual routes | Manual | Small networks, known paths |
| **Dynamic** | Automatic routes | Automatic | Large networks, failover |

### Routing Protocols

```
Distance Vector Routing:
- RIP (Routing Information Protocol)
- EIGRP (Enhanced IGRP)
- Looks at distance (hops) to destination

Link State Routing:
- OSPF (Open Shortest Path First)
- IS-IS
- Looks at link costs

Path Vector Routing:
- BGP (Border Gateway Protocol)
- Considers AS (Autonomous System) path
```

---

## Important Layer 3 Protocols

### ICMP (Internet Control Message Protocol)

```
Uses: Ping, Traceroute, error reporting
Format: Echo Request, Echo Reply, Destination Unreachable

Ping Example:
Host A → ICMP Echo Request → Host B
Host A ← ICMP Echo Reply    ← Host B

Traceroute Example:
Host A → TTL=1 → Router 1 → TTL expired → ICMP Time Exceeded
Host A → TTL=2 → Router 2 → TTL expired → ICMP Time Exceeded
...continues until destination
```

### ARP (Address Resolution Protocol)

```
Purpose: Map IP address to MAC address
Process:
1. "Who has 192.168.1.5?" (Broadcast)
2. Device with 192.168.1.5 replies with its MAC
3. Requester learns: IP 192.168.1.5 = MAC 00:1A:2B:3C:4D:5E

ARP Cache: Stores these mappings locally
ARP Table (switch): Helps track devices
```

### IGMP (Internet Group Management Protocol)

```
Purpose: Multicast group membership
Use: Streaming, video distribution
Host joins/leaves multicast groups
```

---

## NAT (Network Address Translation)

### How NAT Works

```
Internal Network (Private)        External Network (Public)
192.168.1.0/24                    1.2.3.4

Host A (192.168.1.100)
Request to google.com
↓
NAT Router translates
192.168.1.100:port1234 → 1.2.3.4:port5678
↓
Google sees request from 1.2.3.4
Google responds to 1.2.3.4:port5678
↓
NAT Router translates back
1.2.3.4:port5678 → 192.168.1.100:port1234
↓
Host A receives response
```

### Types of NAT

| Type | Details | Use |
|------|---------|-----|
| **Static NAT** | 1 private = 1 public | Web servers |
| **Dynamic NAT** | Many private = pool of public | Office networks |
| **PAT (Overload)** | Many private = 1 public | Home networks |

---

## Advantages of Network Layer

✅ **Inter-network Communication**: Enables communication across different networks  
✅ **Routing**: Optimal path selection  
✅ **Scalability**: IP hierarchical addressing allows growth  
✅ **Logical Addressing**: IP independent of physical medium  
✅ **Flexibility**: Supports both IPv4 and IPv6  
✅ **NAT**: Extends IPv4 life by address reuse  

---

## Disadvantages of Network Layer

❌ **Complexity**: Routing algorithms can be complex  
❌ **Overhead**: IP header adds 20 bytes per packet  
❌ **Unreliable**: Doesn't guarantee packet delivery (TCP does)  
❌ **No QoS**: Can't guarantee quality of service  
❌ **Routing Loops**: Potential for infinite loops  
❌ **Convergence Time**: Takes time for networks to adapt to changes  

---

## DevOps & SRE Relevance

### IP Planning for Infrastructure

```
Data Center Network Design:
Management Network: 10.0.0.0/24
Container Network: 10.1.0.0/16
Storage Network: 10.2.0.0/16
LB Network: 10.3.0.0/24
```

### Kubernetes & Docker Networking

```
Pod CIDR: 10.244.0.0/16 (all pods)
Service CIDR: 10.96.0.0/12 (services)
Node Network: 172.31.0.0/16 (nodes)
```

### Common L3 Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| **No Connectivity** | Route missing | Add route, check gateway |
| **Slow Internet** | Bad routing | Check RTT, use ping/traceroute |
| **Asymmetric Routing** | Different paths in/out | Check return routes |
| **IP Exhaustion** | Too many devices | Subnet redesign |

### Troubleshooting Commands

```bash
# Check IP configuration
ip addr show                    # View IP addresses
ip route show                   # View routing table
route -n                        # Numeric routing table

# Ping test (ICMP)
ping -c 4 8.8.8.8              # Test reachability

# Traceroute (shows hops)
traceroute google.com          # Windows: tracert

# Check specific route
ip route get 8.8.8.8           # Which route used?

# See ARP table (IP to MAC mapping)
arp -a                         # View ARP cache
ip neigh show                  # Modern syntax

# TTL test
ping -m 1 8.8.8.8              # Force TTL=1 (first hop)

# Check MTU
ip link show | grep mtu        # View MTU on interfaces

# DNS to IP (Layer 3 uses IP)
nslookup google.com            # Lookup IP from domain
dig google.com                 # Detailed DNS info
```

---

## Quick Reference Diagram

```
┌──────────────────────────────────────┐
│   NETWORK LAYER (L3)                 │
├──────────────────────────────────────┤
│ ADDRESSING: IP (192.168.1.1)         │
│ DEVICES: Routers, Layer 3 Switches   │
│ UNITS: PACKETS                       │
│ PROTOCOLS: IP, ICMP, ARP, IGMP       │
│ FUNCTIONS: Routing, addressing,      │
│            forwarding, NAT           │
│ SCOPE: Inter-network (end-to-end)    │
└──────────────────────────────────────┘
```

---

## Important Points for Interview

1. **Layer 3 is end-to-end** (unlike L2 which is hop-to-hop)
2. **IP address doesn't change** as packet travels (unlike MAC)
3. **Router decrements TTL** by 1 on each hop
4. **TTL=0** → Packet dropped (prevents infinite loops)
5. **Subnet mask** determines network vs host portion
6. **Default route (0.0.0.0/0)** is "everywhere else"
7. **NAT hides internal IPs** from external network
8. **ICMP** is used for diagnostics (ping, traceroute)

---

## Quick Summary

| Aspect | Details |
|--------|---------|
| **Main Job** | Routing packets across networks |
| **Addressing** | IP address (32-bit IPv4 or 128-bit IPv6) |
| **Data Unit** | Packet |
| **Scope** | End-to-end (across multiple networks) |
| **Key Functions** | Routing, IP addressing, forwarding, NAT |
| **Devices** | Routers, Layer 3 Switches, Gateways |
| **Error Handling** | ICMP messages for errors |
| **Performance Metric** | Latency, Packet loss, Throughput |

---

**Next Layer**: [Layer 4 - Transport Layer](Layer-4-Transport.md)
