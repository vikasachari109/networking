# Layer 4: Transport Layer (CRITICAL FOR DevOps & SRE)

## Definition

The **Transport Layer** (Layer 4) is responsible for **reliable** (or unreliable) **end-to-end communication** between applications on different hosts. It uses **ports** to identify specific applications.

### Core Concept
```
Application wants to send data reliably
↓
Transport Layer ensures:
- Correct ordering (sequence numbers)
- No duplicates (acknowledgments)
- No loss (retransmissions)
↓
Actually delivered to correct application
```

---

## Key Functions

### 1. **Segmentation & Reassembly**
- Break large data into segments (TCP/UDP)
- Reassemble at destination

### 2. **Reliability (if needed)**
- TCP: Ensures delivery, ordering, no duplicates
- UDP: No guarantees (speed over reliability)

### 3. **Port Management**
- Identifies application (HTTP=80, HTTPS=443, SSH=22)
- Allows multiple applications on same host

### 4. **Flow Control**
- Prevent sender from overwhelming receiver
- TCP: Window-based (Sliding Window)
- UDP: No flow control

### 5. **Congestion Control**
- Monitor network congestion
- Adjust transmission rate
- TCP: Slow Start, Congestion Avoidance

---

## Port Numbers

### Port Ranges

```
Well-Known Ports: 0-1023 (Standard services)
Registered Ports: 1024-49151 (Applications)
Dynamic Ports: 49152-65535 (Temporary, client-side)

Port = 16-bit number (65,536 total)
```

### Common Ports

| Port | Protocol | Service |
|------|----------|---------|
| 20-21 | TCP | FTP |
| 22 | TCP | SSH |
| 23 | TCP | Telnet |
| 25 | TCP | SMTP |
| 53 | TCP/UDP | DNS |
| 80 | TCP | HTTP |
| 110 | TCP | POP3 |
| 143 | TCP | IMAP |
| 443 | TCP | HTTPS |
| 3306 | TCP | MySQL |
| 5432 | TCP | PostgreSQL |
| 6379 | TCP | Redis |
| 8080 | TCP | HTTP (alt) |
| 8443 | TCP | HTTPS (alt) |

### DevOps Port Usage

```
Web Services: 80, 443
Kubernetes API: 6443
Docker: 2375, 2376
Prometheus: 9090
Grafana: 3000
Consul: 8500
Elasticsearch: 9200
Kafka: 9092
MySQL: 3306
PostgreSQL: 5432
```

---

## TCP vs UDP - Detailed Comparison

### TCP (Transmission Control Protocol)

**Connection-oriented, Reliable, Ordered**

```
3-Way Handshake (Connection Setup):
─────────────────────────────────────

Host A                                    Host B
│                                         │
├─ SYN (seq=100) ────────────────────────>│
│                                         │
│                 <────────── SYN-ACK (seq=300, ack=101)
│                                         │
├─ ACK (seq=101, ack=301) ───────────────>│
│                                         │

Connection Established! ✓

Data Transfer:
├─ Data 1 → (ack 1)
├─ Data 2 → (ack 2)
└─ Data 3 → (ack 3)

Connection Termination (4-Way):
├─ FIN
├─ ACK
├─ FIN
└─ ACK
```

**Segment Structure:**
```
┌────────────┬──────────┬──────────┬──────────┬──────────┐
│Source Port │Dest Port │Sequence# │Acknowledgement│
└────────────┴──────────┴──────────┴──────────┴──────────┘
┌────────────┬──────────┬──────────┬──────────┬──────────┐
│Flags│Window│Checksum  │Pointer   │ Options  │
└────────────┴──────────┴──────────┴──────────┴──────────┘
```

**TCP Flags:**
```
SYN: Synchronization (connection start)
ACK: Acknowledgement (confirm data received)
FIN: Finish (connection end)
RST: Reset (abrupt close)
PSH: Push (send immediately)
URG: Urgent (urgent data)
```

### UDP (User Datagram Protocol)

**Connectionless, Unreliable, Fast**

```
Host A                    Host B
│                         │
├─ Data ────────────────> │ (No handshake)
│                         │
├─ Data ────────────────> │ (No order guarantee)
│                         │
└─ Data ────────────────> │ (No loss notification)
```

**Datagram Structure:**
```
┌────────────┬──────────┬──────────┬──────────┐
│Source Port │Dest Port │ Length   │Checksum  │
└────────────┴──────────┴──────────┴──────────┘
```

### Detailed Comparison Table

| Feature | TCP | UDP |
|---------|-----|-----|
| **Connection** | Required | Not needed |
| **Reliability** | Guaranteed delivery | No guarantee |
| **Ordering** | Ordered | Out of order possible |
| **Speed** | Slower (checks) | Faster (no checks) |
| **Header Size** | 20 bytes minimum | 8 bytes |
| **Error Checking** | Extensive | Basic (checksum) |
| **Flow Control** | Yes | No |
| **Congestion Control** | Yes | No |
| **Use Cases** | HTTP, SSH, Email | DNS, Video, Gaming |
| **DevOps Examples** | SSH, APIs | DNS queries, Logs |

---

## TCP Connection States (Important for Troubleshooting)

```
CLOSED
  │
  ├─ LISTEN (server waiting)
  │   ├─ SYN_RCVD (received SYN)
  │   └─ ESTABLISHED (ready for data)
  │       │
  │       ├─ CLOSE_WAIT (closing)
  │       └─ LAST_ACK (waiting for ACK)
  │
  ├─ SYN_SENT (client initiating)
  │   └─ ESTABLISHED (ready for data)
  │
  └─ CLOSED (connection ends)

TIME_WAIT: Waiting before fully closing (prevents confusion)
FIN_WAIT_1, FIN_WAIT_2: Closing states
```

### What Each State Means

| State | Meaning |
|-------|---------|
| **LISTEN** | Waiting for incoming connections |
| **ESTABLISHED** | Active connection |
| **TIME_WAIT** | Waiting after close (2-4 minutes default) |
| **CLOSE_WAIT** | Waiting for application to close |
| **SYN_SENT** | Trying to connect |
| **CLOSED** | No connection |

---

## TCP Features for Reliability

### Sequence & Acknowledgement

```
Sender: Sends "HELLO" (sequence=1000)
         │
         ├─ H (seq 1000)
         ├─ E (seq 1001)
         ├─ L (seq 1002)
         ├─ L (seq 1003)
         └─ O (seq 1004)
                  │
Receiver:     ACK (ack=1005) - "I got up to 1004"
```

### Retransmission on Loss

```
Sender: Send Data (seq 100)
        Wait for ACK (RTO - Retransmission Timeout)
        │
        ├─ If ACK received → Continue
        │
        └─ If timeout → Retransmit Data
                        Increase RTO (exponential backoff)
```

### Sliding Window Flow Control

```
Window Size = 5

Sender can send 5 segments before waiting:
[1] [2] [3] [4] [5] (wait for ACK)
         │
Receiver: ACK 1 received
[2] [3] [4] [5] [6] (window slides)
         │
Sender: Send segment 6
```

---

## Congestion Control (TCP)

### Slow Start

```
Start: cwnd = 1 (1 segment)
       │
       └─ ACK → cwnd = 2
                  │
                  └─ ACK → cwnd = 4
                             │
                             └─ ACK → cwnd = 8
                                        (Exponential growth)

Until: Reaches ssthresh (slow start threshold)
       or packet loss detected
```

### Congestion Avoidance

```
After Slow Start:
cwnd increases by 1 per RTT (Linear growth)

Faster in short networks, slower in congested
```

---

## Advantages of Layer 4

### TCP Advantages
✅ **Reliability**: Guaranteed delivery  
✅ **Ordering**: Packets arrive in order  
✅ **No Duplicates**: Each packet received once  
✅ **Flow Control**: Adapts to receiver speed  
✅ **Congestion Control**: Adapts to network conditions  
✅ **Connection Tracking**: Knows active connections  

### UDP Advantages
✅ **Speed**: No connection overhead  
✅ **Low Latency**: Minimal delay  
✅ **Multicast/Broadcast**: Can send to many  
✅ **Stateless**: Server doesn't track connections  
✅ **Low CPU**: Simpler processing  

---

## Disadvantages of Layer 4

### TCP Disadvantages
❌ **Slow**: Handshake overhead (3 RTTs to send data)  
❌ **High CPU**: Tracking connections expensive  
❌ **Buffering**: Retransmission buffer uses memory  
❌ **Head of Line Blocking**: One lost packet delays all  

### UDP Disadvantages
❌ **Unreliable**: Packets can be lost  
❌ **Unordered**: May arrive out of order  
❌ **Duplicates**: Same packet twice possible  
❌ **No Flow Control**: Can overwhelm receiver  

---

## DevOps & SRE Critical Topics

### Connection Issues

```bash
# Too many connections in TIME_WAIT
netstat -an | grep TIME_WAIT | wc -l
# Solution: Lower tcp_fin_timeout or use SO_REUSEADDR

# Connections stuck in CLOSE_WAIT
netstat -an | grep CLOSE_WAIT | wc -l
# Solution: Fix application to close sockets

# Active connections
netstat -an | grep ESTABLISHED | wc -l
# Solution: Implement connection pooling if too high
```

### Network Tuning Parameters

```bash
# TCP buffer sizes
sysctl net.core.rmem_default          # Receive buffer
sysctl net.core.wmem_default          # Write buffer

# Connection limits
ulimit -n                             # File descriptors
sysctl net.ipv4.tcp_max_syn_backlog   # SYN queue

# Timeouts
sysctl net.ipv4.tcp_fin_timeout       # TIME_WAIT duration

# Optimize for high throughput
sysctl net.ipv4.tcp_tw_reuse=1        # Reuse TIME_WAIT
sysctl net.ipv4.tcp_tw_recycle=1      # Quick close

# Window scaling (jumbo frames benefit)
sysctl net.ipv4.tcp_window_scaling=1

# TCP keepalive (detect dead connections)
sysctl net.ipv4.tcp_keepalive_time=600
```

### Port Management Issues

```
Port already in use:
─ Kill process using port
─ Change port number
─ Increase TIME_WAIT timeout (if many closed connections)

Port exhaustion (high load):
─ Number of connections too high
─ Not enough IPs/ports available
─ Solution: More servers, connection pooling, NAT
```

### Load Balancing at Layer 4

```
L4 (TCP) Load Balancing:

Client 1 ──┐
Client 2 ──┼─> Load Balancer ──┬─> Server 1
Client 3 ──┘                    ├─> Server 2
                               └─> Server 3

Distributes traffic by:
- Round Robin
- Least connections
- IP hash
- Port hash
```

---

## Common L4 Troubleshooting Scenarios

### Scenario 1: "Connection Refused"

```
Error: Connection refused
Cause: No service listening on port
Check: Is service running?
       netstat -tlnp | grep :8080

Fix: Start service or check configuration
```

### Scenario 2: "Connection Timeout"

```
Error: Connection timeout
Cause: Firewall blocking, service down, network unreachable
Check: Can you ping the server?
       Can you reach the port? (timeout ≠ refused)

Fix: Check firewall, network route, service health
```

### Scenario 3: "Connection Reset"

```
Error: Connection reset by peer
Cause: Service rejected connection, firewall reset, buffer full
Check: Check service logs
       Check firewall rules
       Check TCP backlog

Fix: Investigate why service rejected
     Increase backlog if server overwhelmed
```

---

## Troubleshooting Commands (L4 Focus)

```bash
# List listening ports
netstat -tlnp                         # All listening
netstat -tlnp | grep :8080            # Specific port

# Check connection status
netstat -an | grep :8080              # All states
netstat -an | grep ESTABLISHED        # Active
netstat -an | grep TIME_WAIT          # Waiting to close

# Modern alternative (ss = socket statistics)
ss -tlnp                              # Listening ports
ss -ant                               # All TCP sockets
ss -ant | grep ESTAB | wc -l          # Count established

# UDP sockets
netstat -ump                          # UDP listeners
ss -ulnp                              # Modern syntax

# TCP statistics
netstat -s | grep -i tcp              # TCP stats
cat /proc/net/tcp | head              # Raw TCP table

# Per-socket options
ss -opt                               # TCP options (RTT, RTO)
ss -opt | grep ESTAB                  # Active connections with RTT

# Port process mapping
lsof -i :8080                         # What's on port 8080?
lsof -i tcp                           # All TCP
lsof -p PID                           # Specific process

# Packet captures (Layer 4 analysis)
tcpdump -i eth0 -n port 8080          # Traffic on 8080
tcpdump -i eth0 tcp flags=S           # SYN packets

# Connection state distribution
netstat -an | awk '/tcp/ {print $6}' | sort | uniq -c
# Shows: ESTABLISHED, TIME_WAIT, LISTEN counts

# Check RTO and retransmissions
ss -info                              # Detailed socket info
```

---

## Quick Reference Diagram

```
┌──────────────────────────────────────┐
│   TRANSPORT LAYER (L4)               │
├──────────────────────────────────────┤
│ ADDRESSING: Port (16-bit, 0-65535)   │
│ DEVICES: Firewalls, L4 LB            │
│ UNITS: SEGMENTS (TCP), DATAGRAMS(UDP)│
│ PROTOCOLS: TCP, UDP, SCTP            │
│ FUNCTIONS: Segmentation,             │
│            reliability, ports,       │
│            flow control              │
│ SCOPE: End-to-end applications       │
└──────────────────────────────────────┘
```

---

## Important Points for Interview

1. **TCP requires handshake**, UDP doesn't
2. **TCP ensures order & delivery**, UDP doesn't
3. **Ports are Layer 4** (not Layer 3)
4. **Connection state tracking** is TCP-specific
5. **TIME_WAIT prevents packet confusion**
6. **RTO doubles on timeout** (exponential backoff)
7. **Window size controls flow** in TCP
8. **UDP used for speed** (DNS, video, gaming)
9. **Connection reset** means service rejected
10. **Timeout** means no response (firewall, unreachable)

---

## Quick Summary

| Aspect | Details |
|--------|---------|
| **Main Job** | Reliable/fast delivery to application |
| **Addressing** | Port (16-bit: 0-65535) |
| **Data Unit** | Segment (TCP), Datagram (UDP) |
| **Scope** | End-to-end (application to application) |
| **Key Functions** | Segmentation, reliability, ports, flow control |
| **Protocols** | TCP, UDP, SCTP |
| **Reliability** | TCP yes, UDP no |
| **Use Cases** | TCP: Web, SSH; UDP: DNS, Video |

---

**Next Layer**: [Layer 5 - Session Layer](Layer-5-Session.md)

---

## Special Note for DevOps/SRE

**Layer 4 is crucial for DevOps/SRE engineers because:**
1. Most infrastructure issues are at L3 or L4
2. Service connectivity problems = L4 diagnosis
3. Performance tuning requires L4 understanding
4. Load balancing operates at L4
5. Monitoring connections = tracking L4 states
6. Container networking uses L4 concepts
7. Service mesh (Istio) operates between L4-L7

**Make sure to master:**
- TCP/UDP differences
- Port management
- Connection states
- Troubleshooting commands
- Network tuning parameters
