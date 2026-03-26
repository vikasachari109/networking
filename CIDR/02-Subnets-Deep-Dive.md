# Subnets — Deep Dive

---

## TABLE OF CONTENTS

1. [What is a Subnet?](#1-what-is-a-subnet)
2. [Why Do We Need Subnets?](#2-why-do-we-need-subnets)
3. [How Subnetting Works — The Process](#3-how-subnetting-works)
4. [Fixed-Length Subnet Masking (FLSM)](#4-fixed-length-subnet-masking)
5. [Variable-Length Subnet Masking (VLSM)](#5-variable-length-subnet-masking)
6. [Subnetting an IPv4 Network — Step-by-Step](#6-step-by-step-subnetting)
7. [Key Subnet Terms Cheat Sheet](#7-key-subnet-terms)
8. [Designing Subnets for a Real Network](#8-designing-subnets)
9. [Common Subnetting Mistakes](#9-common-mistakes)
10. [Subnet Cheat Sheet — All /prefix Values](#10-subnet-cheat-sheet)

---

## 1. What is a Subnet?

### Definition

A **subnet (sub-network)** is a logically divided segment of a larger IP network. It is created by borrowing bits from the **host portion** of an IP address to create a **new network identifier**.

### Analogy

Think of a city:
```
City          = Your entire IP network (e.g., 10.0.0.0/8)
   ├── District A = Subnet 1 (e.g., 10.1.0.0/16)  ← Finance department
   ├── District B = Subnet 2 (e.g., 10.2.0.0/16)  ← Engineering
   └── District C = Subnet 3 (e.g., 10.3.0.0/16)  ← HR / Admin
```

Each district has its own streets (IP ranges), and you need a bridge (router) to travel between districts, just like data needs a router to move between subnets.

### The Mechanism

Subnetting works by **extending the network prefix** — taking bits that used to identify hosts and using them to identify sub-networks instead.

```
Original:    10.0.0.0 /8
              ^^^^^^^^  8 network bits, 24 host bits

After subnetting to /16:
             10.0.0.0 /16
              ^^^^^^^^^^  16 network bits, 16 host bits
              ^^^^^^^^ ^^^^^^^^
             original  borrowed
             network   (new subnet bits)
             part
```

The more bits you "borrow", the more subnets you get, but each subnet has fewer hosts.

**The trade-off:**
```
More subnets  ←────────────────────→  More hosts per subnet
    /28 (16 subnets of 14 hosts)        /25 (2 subnets of 126 hosts)
```

---

## 2. Why Do We Need Subnets?

### 2.1 Security Isolation

Without subnets, all devices on your network can communicate directly. This is a nightmare for security.

```
Without subnets:
[HR Laptop] ─────── [Finance Server] ─── [Dev Server]
                All on same flat network
                ANY device can reach ANY other device

With subnets:
[HR Subnet: 10.1.0.0/24]         [Finance Subnet: 10.2.0.0/24]
[HR Laptop] [HR Printer]         [Finance App] [Finance DB]
         │                                │
         └────────── ROUTER/FIREWALL ─────┘
                   (traffic controlled here)
```

**Security rules are enforced at the router/firewall between subnets.** HR staff physically cannot reach the Finance database without going through a controlled gateway.

### 2.2 Reduced Broadcast Traffic

Every network has a **broadcast domain** — when one device sends a broadcast packet (like an ARP request), EVERY device on the same subnet receives it, processes it, and has to respond or ignore it.

```
Without subnets (1 flat /16 network → 65,534 devices):
  1 broadcast → 65,534 devices interrupted. Every. Single. Time.

With subnets (divided into /24 → 254 devices per subnet):
  1 broadcast → only 254 devices interrupted → 99.6% reduction
```

This dramatically improves network performance, especially in large environments.

### 2.3 Efficient IP Address Management

Without subnetting you'd allocate large blocks "just in case." With VLSM (Variable Length Subnet Masking), you can allocate exactly what you need:

```
Department needs:
  Engineering: 60 hosts  → /26 (62 usable) ✓  no waste
  HR:          20 hosts  → /27 (30 usable) ✓
  IoT sensors: 10 hosts  → /28 (14 usable) ✓
  DB link:      2 hosts  → /30 (2 usable)  ✓
```

### 2.4 Network Organization & Troubleshooting

Subnets make it easy to:
- **Identify which team/function** an IP belongs to
- **Isolate problems** — if 10.3.x.x is down, it's the Finance subnet
- **Apply QoS policies** — prioritize VoIP traffic in its own subnet
- **Simplify routing tables** — summarize multiple hosts into one route

### 2.5 Compliance & Regulatory Requirements

PCI-DSS, HIPAA, and SOC2 all require **network segmentation** to protect sensitive data. Subnets are the technical implementation of this requirement.

```
PCI Zone: Payment processing servers in dedicated subnet
  e.g., 10.50.0.0/24  (credit card processing only)
  
Only specific IP addresses are allowed to communicate with this subnet
All other traffic is blocked at the firewall
```

---

## 3. How Subnetting Works

### 3.1 The Core Concept: Borrowing Bits

When you subnet, you "borrow" bits from the host portion to create subnet identifiers.

```
Original Class C: 192.168.1.0/24
  Network: 24 bits   Host: 8 bits

Borrow 2 bits → /26
  Network: 26 bits   Host: 6 bits
  
Number of subnets = 2^(borrowed bits) = 2^2 = 4
Hosts per subnet  = 2^(remaining host bits) - 2 = 2^6 - 2 = 62
```

### 3.2 Visualizing Bit Borrowing

```
Original /24 host octet:
  [h][h][h][h][h][h][h][h]   8 host bits = 256 addresses

Borrow 1 bit → /25:
  [s][h][h][h][h][h][h][h]   1 subnet bit, 7 host bits
  Creates 2 subnets of 128 addresses each

Borrow 2 bits → /26:
  [s][s][h][h][h][h][h][h]   2 subnet bits, 6 host bits
  Creates 4 subnets of 64 addresses each

Borrow 3 bits → /27:
  [s][s][s][h][h][h][h][h]   3 subnet bits, 5 host bits
  Creates 8 subnets of 32 addresses each

Borrow 4 bits → /28:
  [s][s][s][s][h][h][h][h]   4 subnet bits, 4 host bits
  Creates 16 subnets of 16 addresses each
```

### 3.3 The Subnet Formula

```
Number of subnets  = 2^n         where n = number of borrowed bits
Addresses per subnet = 2^h       where h = remaining host bits
Usable hosts per subnet = 2^h - 2
```

---

## 4. Fixed-Length Subnet Masking (FLSM)

### What is FLSM?

**FLSM** divides a network into subnets of **equal size**. Every subnet uses the same prefix length.

### When to Use

- Simple environments
- When all segments need the same number of hosts
- Cisco exam practice (CCNA)

### Example: Divide 192.168.10.0/24 into 4 equal subnets

```
Borrow 2 bits  →  /26
Block size: 64 (256 ÷ 4 = 64)

Subnet 1: 192.168.10.0/26
  Network:   192.168.10.0
  Usable:    192.168.10.1  – 192.168.10.62
  Broadcast: 192.168.10.63

Subnet 2: 192.168.10.64/26
  Network:   192.168.10.64
  Usable:    192.168.10.65 – 192.168.10.126
  Broadcast: 192.168.10.127

Subnet 3: 192.168.10.128/26
  Network:   192.168.10.128
  Usable:    192.168.10.129 – 192.168.10.190
  Broadcast: 192.168.10.191

Subnet 4: 192.168.10.192/26
  Network:   192.168.10.192
  Usable:    192.168.10.193 – 192.168.10.254
  Broadcast: 192.168.10.255
```

---

## 5. Variable-Length Subnet Masking (VLSM)

### What is VLSM?

**VLSM** allows different subnets within the same network to have **different prefix lengths** (different sizes). This is what modern networks and cloud platforms use.

### Why VLSM is Better

```
Network given: 192.168.1.0/24  (256 addresses)

Requirements:
  Subnet A: 100 hosts  (Sales team)
  Subnet B: 50 hosts   (Dev team)
  Subnet C: 20 hosts   (Ops team)
  Subnet D: 2 hosts    (Router-to-Router link)
```

**FLSM approach (wasteful):**
- 4 subnets of /26 (64 addresses each) = 256 total
- Subnet A needs 100 but gets 62 usable → **DOESN'T EVEN FIT!**
- Even /25 only gives 126, wasting 26 for Sales and leaving 126 unused in the other /25

**VLSM approach (efficient):**
```
Step 1: Sort requirements largest to smallest:
  100 hosts, 50 hosts, 20 hosts, 2 hosts

Step 2: Assign smallest fitting block to each:

Subnet A (100 hosts): need /25 → 126 usable
  192.168.1.0/25  → 192.168.1.0   – 192.168.1.127

Subnet B (50 hosts): need /26 → 62 usable
  192.168.1.128/26 → 192.168.1.128 – 192.168.1.191

Subnet C (20 hosts): need /27 → 30 usable
  192.168.1.192/27 → 192.168.1.192 – 192.168.1.223

Subnet D (2 hosts): need /30 → 2 usable
  192.168.1.224/30 → 192.168.1.224 – 192.168.1.227

Remaining (unallocated): 192.168.1.228 – 192.168.1.255
  (available for future use)
```

VLSM wastes almost nothing. This is real-world subnetting.

---

## 6. Step-by-Step Subnetting

### Problem: 
You have been given `10.10.0.0/20`. You need to create subnets for:
- 3 departments of ~200 hosts each
- 1 management subnet of ~20 hosts
- 2 point-to-point router links

### Step 1: Understand What You Have

```
10.10.0.0/20
Block size: 2^(32-20) = 2^12 = 4096 addresses
Range: 10.10.0.0 – 10.10.15.255
```

### Step 2: Sort by size (largest to smallest)

```
1. Dept A: 200 hosts
2. Dept B: 200 hosts
3. Dept C: 200 hosts
4. Management: 20 hosts
5. Link 1: 2 hosts
6. Link 2: 2 hosts
```

### Step 3: Find the right prefix for each

```
200 hosts → 2^h - 2 ≥ 200 → 2^h ≥ 202 → h=8 → /24 (254 usable) ✓
 20 hosts → 2^h - 2 ≥ 20  → 2^h ≥ 22  → h=5 → /27 (30 usable)  ✓
  2 hosts → 2^h - 2 ≥ 2   → 2^h ≥ 4   → h=2 → /30 (2 usable)   ✓
```

### Step 4: Assign sequentially, no overlapping

```
Subnet 1 (Dept A):    10.10.0.0/24   → 10.10.0.0   – 10.10.0.255
Subnet 2 (Dept B):    10.10.1.0/24   → 10.10.1.0   – 10.10.1.255
Subnet 3 (Dept C):    10.10.2.0/24   → 10.10.2.0   – 10.10.2.255
Subnet 4 (Mgmt):      10.10.3.0/27   → 10.10.3.0   – 10.10.3.31
Subnet 5 (Link 1):    10.10.3.32/30  → 10.10.3.32  – 10.10.3.35
Subnet 6 (Link 2):    10.10.3.36/30  → 10.10.3.36  – 10.10.3.39

Remaining available:  10.10.3.40 – 10.10.15.255
```

### Step 5: Verify — no overlaps, all within parent range

```
Parent:          10.10.0.0 – 10.10.15.255  ✓
All subnets fit within parent range         ✓
No address ranges overlap                   ✓
```

---

## 7. Key Subnet Terms Cheat Sheet

| Term                | Definition                                                                 |
|---------------------|----------------------------------------------------------------------------|
| **Network Address** | First IP of a subnet. Identifies the subnet itself. Not assignable.       |
| **Broadcast Address**| Last IP of a subnet. Sends to all hosts on subnet. Not assignable.       |
| **Host Range**      | All IPs between network and broadcast addresses. Assignable to devices.   |
| **Subnet Mask**     | 32-bit number showing which bits are network vs host.                      |
| **Prefix Length**   | Count of consecutive 1-bits in subnet mask (the /XX number).              |
| **Default Gateway** | Router interface IP on each subnet. The exit point for inter-subnet traffic.|
| **VLSM**            | Variable Length Subnet Masking — different sized subnets in same network. |
| **FLSM**            | Fixed Length Subnet Masking — all subnets same size.                      |
| **Supernet**        | Combining multiple subnets into a larger block for routing efficiency.     |
| **Broadcast Domain**| All devices that receive each other's broadcast packets (= 1 subnet).    |
| **Collision Domain** | (Layer 2) segment where data collisions can occur. Each switch port = 1. |

---

## 8. Designing Subnets for a Real Network

### 8.1 Design Principles

1. **Start with the largest subnet first** when using VLSM
2. **Plan for growth** — add 20–50% buffer to host counts
3. **Use consistent addressing conventions** across environments
4. **Document everything** — subnet purpose, range, VLAN ID, responsible team
5. **Align with security zones** — one subnet per trust level
6. **Don't cross router boundaries** when designing one subnet

### 8.2 Typical Enterprise Subnet Design

```
Given: 10.0.0.0/8 (entire private range)

Tier 1 — Core/DMZ:      10.0.0.0/16
  Web servers (DMZ):    10.0.1.0/24
  Load balancers:       10.0.2.0/24
  API Gateway:          10.0.3.0/24

Tier 2 — App Layer:     10.1.0.0/16
  Microservices:        10.1.0.0/22   (1022 hosts)
  Background workers:   10.1.4.0/24

Tier 3 — Data Layer:    10.2.0.0/16
  Primary DBs:          10.2.0.0/24
  Read replicas:        10.2.1.0/24
  Cache layer:          10.2.2.0/24

Management:             10.255.0.0/16
  Bastion hosts:        10.255.0.0/27
  Monitoring agents:    10.255.1.0/24
  VPN gateway:          10.255.2.0/28
```

### 8.3 The Subnet Planning Worksheet

Before writing a single IP, answer:

1. How many subnets do I need? (departments, security zones, tiers)
2. How many hosts per subnet? (current + 20% growth buffer)
3. Which parent CIDR block do I have?
4. Should I use VLSM or FLSM?
5. What are the security boundaries?
6. Which subnets need Internet access?
7. Which subnets need to talk to each other?

---

## 9. Common Subnetting Mistakes

| Mistake | Problem | Fix |
|---------|---------|-----|
| Overlapping subnet ranges | Packets route to wrong destination | Always verify ranges don't overlap before assigning |
| Using network or broadcast address as a host IP | Device can't communicate | Always skip first and last address |
| Not leaving room for growth | Subnet fills up, major re-addressing needed | Add 20–30% buffer when sizing |
| All departments on same subnet | Zero security isolation | Always separate by security zone |
| Choosing the wrong parent CIDR | Not enough addresses for all subnets | Calculate total required addresses first |
| Ignoring cloud-reserved IPs | Azure/GCP/AWS reserve 3–5 IPs per subnet | Account for this in sizing |
| /30 for network links in cloud | Azure doesn't support /30 for some services | Use /29 or larger in Azure subnets |

---

## 10. Subnet Cheat Sheet — All /prefix Values

```
Prefix  Subnet Mask         Block  Usable  # Subnets from /24
/24     255.255.255.0        256    254         1
/25     255.255.255.128      128    126         2
/26     255.255.255.192       64     62         4
/27     255.255.255.224       32     30         8
/28     255.255.255.240       16     14        16
/29     255.255.255.248        8      6        32
/30     255.255.255.252        4      2        64
/31     255.255.255.254        2      2*       128  ← point-to-point only
/32     255.255.255.255        1      1        256  ← single host
```

### "How many /26s fit in a /24?" Formula:
```
2^(target_prefix - parent_prefix) = 2^(26-24) = 2^2 = 4 subnets
```

Any prefix difference works:
```
/28 subnets in a /22 = 2^(28-22) = 2^6 = 64 subnets
/27 subnets in a /20 = 2^(27-20) = 2^7 = 128 subnets
```

---

> **Next:** [03-Real-World-Scenarios-Azure-GCP.md](./03-Real-World-Scenarios-Azure-GCP.md)
