# Layer 2: Data Link Layer

## Definition

The **Data Link Layer** (Layer 2) is responsible for reliable data transfer between **directly connected** devices (node-to-node). It operates using **MAC addresses** instead of IP addresses.

### Core Concept
```
Frame = Data + MAC Header + Checksum

MAC Address (Layer 2):  48-bit address (e.g., 00:1A:2B:3C:4D:5E)
IP Address (Layer 3):   32-bit address (e.g., 192.168.1.1)

Layer 2: Only cares about getting frame to next device (MAC)
Layer 3: Cares about getting packet to final destination (IP)
```

---

## Key Functions

### 1. **Framing**
- Breaks data into frames
- Adds header (MAC addresses) and trailer (checksum)
- Removes frames at receiving end

### 2. **Physical Addressing (MAC)**
- Uses MAC addresses for local communication
- MAC Format: `00:1A:2B:3C:4D:5E` (48 bits)
- OUI (Organizationally Unique Identifier): First 24 bits (vendor)

### 3. **Error Detection & Correction**
- CRC (Cyclic Redundancy Check): Detects errors
- Some protocols can correct errors

### 4. **Flow Control**
- Prevents faster sender from overwhelming slower receiver
- Uses buffering and ACK mechanisms

### 5. **Media Access Control (MAC)**
- Determines who can transmit on shared medium
- Examples: CSMA/CD, Token Ring

---

## Data Link Layer Sublayers

```
┌───────────────────────────────┐
│    Data Link Layer (L2)       │
├───────────────────────────────┤
│  Logical Link Control (LLC)   │ ← Error checking, flow control
├───────────────────────────────┤
│  Media Access Control (MAC)   │ ← MAC addresses, medium access
├───────────────────────────────┤
│     Physical Layer (L1)       │
└───────────────────────────────┘
```

### LLC (Logical Link Control)
- Error detection (CRC)
- Flow control
- Sequencing

### MAC (Media Access Control)
- MAC addressing
- Channel access
- Frame formatting

---

## MAC Addressing

### MAC Address Structure

```
00:1A:2B:3C:4D:5E
├─ OUI (Vendor ID) ─┤  ├─ Device ID ─┤
00:1A:2B             3C:4D:5E

OUI Examples:
00:50:F2 = Microsoft
08:00:07 = Apple
00:0A:95 = Cisco
```

### MAC Address Types

| Type | Format | Example | Use |
|------|--------|---------|-----|
| **Unicast** | Specific device | 00:1A:2B:3C:4D:5E | Single device |
| **Broadcast** | All devices | FF:FF:FF:FF:FF:FF | All in segment |
| **Multicast** | Group devices | 01:00:5E:xx:xx:xx | Group communication |

### Important:
```
MAC Address Scope: LOCAL network only (not routable)
When you leave your network (pass through router):
- MAC address changes (gets rewritten)
- IP address remains same
```

---

## Frame Structure

### Ethernet Frame Format

```
┌────────┬──────────┬──────────┬──────────────┬───────┬─────┐
│ Preamble│Dest MAC │Source MAC│ Type  │Payload│ CRC  │
│ 8 bytes │ 6 bytes │ 6 bytes  │2 bytes│46-1500│4 bytes│
└────────┴──────────┴──────────┴──────────────┴───────┴─────┘

Preamble: Synchronization (not part of frame for processing)
Destination MAC: Where frame goes
Source MAC: Where frame comes from
Type: Protocol type (IP=0x0800, ARP=0x0806)
Payload: Actual data (46-1500 bytes)
FCS (CRC): Error detection checksum
```

### Frame Size (MTU - Maximum Transmission Unit)

```
Standard Ethernet:
- Max payload: 1500 bytes
- Total frame: 1518 bytes (with headers)
- Min payload: 46 bytes
- Jumbo frames: Up to 9000 bytes (special networks)
```

---

## Common Layer 2 Protocols

### Ethernet (Most Common)

| Standard | Speed | Cable | Year |
|----------|-------|-------|------|
| 10BASE-T | 10 Mbps | Twisted pair | 1985 |
| 100BASE-TX | 100 Mbps | Twisted pair | 1995 |
| 1000BASE-T | 1 Gbps | Twisted pair | 1999 |
| 10GBASE-T | 10 Gbps | Twisted pair | 2006 |

### Frame Types

```
802.3 Ethernet (Standard)
├─ 802.3u (Fast Ethernet - 100 Mbps)
├─ 802.3z (Gigabit - Fiber)
└─ 802.3ab (Gigabit - UTP)

802.11 (WiFi)
├─ 802.11a (5 GHz, 54 Mbps)
├─ 802.11b (2.4 GHz, 11 Mbps)
├─ 802.11g (2.4 GHz, 54 Mbps)
├─ 802.11n (2.4/5 GHz, 600 Mbps)
├─ 802.11ac (5 GHz, 1.3 Gbps)
└─ 802.11ax (2.4/5 GHz, 9.6 Gbps)
```

### PPP (Point-to-Point Protocol)
- Used for serial connections (modems, DSL)
- Simpler than Ethernet
- Used in dial-up and some WAN links

### Frame Relay (Legacy WAN)
- Used for WAN links (obsolete)
- Virtual circuits

---

## Error Detection & Correction

### Cyclic Redundancy Check (CRC)

```
Data: 1010101
Polynomial: x³ + x + 1 (binary: 1011)

Process:
- Sender: Divides data by polynomial, appends remainder
- Receiver: Divides received data by same polynomial
- If remainder = 0: No errors
- If remainder ≠ 0: Error detected
```

### Error Detection Methods

| Method | Detection | Correction | Overhead |
|--------|-----------|-----------|----------|
| **CRC** | Yes | No | 32 bits |
| **Hamming Code** | Yes | Yes | Few bits |
| **Parity Check** | Limited | No | 1 bit |

### What Layer 2 Does When Error Detected

```
Errors Found:
├─ Drop frame (Ethernet, WiFi)
└─ Request retransmission (some protocols)

Note: TCP at Layer 4 ensures reliability if L2 fails
```

---

## Flow Control Mechanisms

### Stop-and-Wait ARQ

```
Sender: Send Frame 1
        ↓ (Wait)
Receiver: Process Frame 1 → Send ACK
        ↓ (Wait)
Sender: Receive ACK → Send Frame 2
        ...
```
- **Advantage**: Simple
- **Disadvantage**: Slow, inefficient

### Sliding Window Protocol

```
Sender can send multiple frames before waiting for ACK
Window size = 3

Sender: [Frame 1] [Frame 2] [Frame 3]
                  ↓ (Wait for ACK, can send Frame 4)
Receiver: ACK 1, then ACK 2, then ACK 3
```
- **Advantage**: Efficient, full pipe utilization
- **Disadvantage**: More complex

---

## Layer 2 Devices & Technologies

### Switches

```
Hub (Old - Broadcasts)       Switch (New - Intelligent)
────────────────────────────────────────────────────
Device 1 ─┐                  Device 1 ═┐
Device 2 ─┼─ Hub (repeats)   Device 2 ═╬─ Switch (learns MAC)
Device 3 ─┘ all frames       Device 3 ═┘ sends to correct port
Device 4   to all ports      Device 4

Switch learns MAC addresses and builds MAC table:
MAC Address    | Port
───────────────────────
00:1A:2B:3C:4D:5E | Port 1
00:2B:3C:4D:5E:6F | Port 2
```

### VLANs (Virtual LANs)

```
Physical: One switch
Logical: Multiple networks

VLAN 10 (Finance):  Device A, Device B
VLAN 20 (HR):       Device C, Device D
VLAN 30 (IT):       Device E, Device F

Devices in different VLAN can't talk without router
```

### Link Aggregation (LAG / EtherChannel)

```
Multiple physical links = One logical link

Port 1: 1 Gbps ─┐
Port 2: 1 Gbps ─┼─ = 4 Gbps combined bandwidth
Port 3: 1 Gbps ─┤
Port 4: 1 Gbps ─┘
```

---

## Advantages of Data Link Layer

✅ **Reliability**: Error detection ensures frame integrity  
✅ **Efficiency**: Flow control prevents buffer overflow  
✅ **Local Communication**: MAC addressing for local networks  
✅ **Switching**: Switches reduce collisions (vs hubs)  
✅ **Speed**: Direct link negotiation for optimal speed  
✅ **Organization**: Frames organize data logically  

---

## Disadvantages of Data Link Layer

❌ **Limited Scope**: Only works for directly connected devices  
❌ **Error Recovery**: Drops frames on error (TCP retransmits)  
❌ **No Routing**: Can't reach beyond local network  
❌ **No Security**: Raw MAC address visible (fixable with encryption)  
❌ **Flooding**: Broadcast storms can occur in large networks  
❌ **Complexity**: Multiple sublayers and technologies  

---

## DevOps & SRE Relevance

### Switch Configuration

1. **VLAN Management**
   ```bash
   # Assign port to VLAN
   switch# interface fa0/1
   switch# switchport mode access
   switch# switchport access vlan 10
   ```

2. **LAG/EtherChannel for redundancy**
   ```bash
   switch# interface range fa0/1-4
   switch# channel-group 1 mode active
   ```

3. **MAC Address Filtering (Security)**
   ```bash
   switch# mac-address-table secure
   switch# port-security
   ```

### Container Networking

```
Docker/Kubernetes uses L2 bridges internally:
Container A ─┐
Container B ─┼─ Virtual Switch (L2 learning)
Container C ─┘
```

### Network Monitoring

```bash
# View MAC address table
show mac-address-table
show arp  # Indirect: maps IP to MAC (L3 to L2)

# Check interface errors
show interface fa0/1
# Look for: CRC errors, collision, runts
```

---

## Common DevOps Problems at L2

| Problem | Cause | Symptom | Solution |
|---------|-------|---------|----------|
| **MAC Flap** | Device moves frequently | Constant MAC changes | Check for loops |
| **Broadcast Storm** | Loop in VLAN | Network slowdown | Enable STP |
| **Unknown Unicast Flood** | Dest MAC unknown | Inefficient forwarding | Check VLAN config |
| **Port Flapping** | Unstable link | Interface up/down | Check cable, duplex |

### Troubleshooting Commands

```bash
# View MAC address table
show mac-address-table dynamic

# Check switch port status
show interface status

# See protocol negotiation details
show interface fa0/1

# Verify VLAN assignment
show vlan
show interface fa0/1 switchport

# Check spanning tree (loop prevention)
show spanning-tree brief
```

---

## Quick Reference Diagram

```
┌──────────────────────────────────────┐
│  DATA LINK LAYER (L2)                │
├──────────────────────────────────────┤
│ ADDRESSING: MAC (00:1A:2B:3C:4D:5E) │
│ DEVICES: Switches, Bridges, WAPs     │
│ UNITS: FRAMES                        │
│ PROTOCOLS: Ethernet, PPP, Frame Relay│
│ FUNCTIONS: Framing, MAC addressing,  │
│            Error detection, Flow ctrl│
│ SCOPE: Local network (one segment)   │
└──────────────────────────────────────┘
```

---

## Important Points for Interview

1. **Layer 2 is "hop-to-hop"** - not end-to-end
2. **MAC address changes at each hop** (after router)
3. **Switches learn MAC addresses dynamically**
4. **Collisions occur in shared medium** (use switches to avoid)
5. **Error detection, not correction** (in most protocols)
6. **Flow control prevents buffer overflow**
7. **VLAN isolates traffic logically**
8. **STP prevents loops in switched networks**

---

## Quick Summary

| Aspect | Details |
|--------|---------|
| **Main Job** | Reliable delivery between directly connected devices |
| **Addressing** | MAC address (48-bit, 00:1A:2B:3C:4D:5E) |
| **Data Unit** | Frame |
| **Scope** | Local network segment (not routable) |
| **Key Functions** | Framing, MAC addressing, error detection, flow control |
| **Devices** | Switches, Bridges, WAPs |
| **Error Handling** | Detection (CRC), drops on error |
| **Performance Metric** | Frame loss, Latency, Throughput |

---

**Next Layer**: [Layer 3 - Network Layer](Layer-3-Network.md)
