# CIDR — Prerequisites & Fundamentals

---

## TABLE OF CONTENTS

1. [Prerequisites — What You Must Know First](#1-prerequisites)
2. [The Problem CIDR Solves — Why It Exists](#2-the-problem-cidr-solves)
3. [What is CIDR?](#3-what-is-cidr)
4. [IP Address Deep Dive](#4-ip-address-deep-dive)
5. [Classful Networking (the old way)](#5-classful-networking)
6. [How CIDR Works — The Mechanics](#6-how-cidr-works)
7. [CIDR Notation Explained](#7-cidr-notation-explained)
8. [Binary Math You Need](#8-binary-math-you-need)
9. [CIDR Block Sizes — Quick Reference Table](#9-cidr-block-sizes)

---

## 1. Prerequisites

Before you can truly understand CIDR, you need a solid grasp of the following foundational concepts. Skipping these will make CIDR feel like magic — in a bad way.

---

### 1.1 What is an IP Address?

An **IP Address (Internet Protocol Address)** is a unique numerical label assigned to every device connected to a network that uses the Internet Protocol for communication.

Think of it like a **home mailing address** — it tells the network exactly where to deliver data.

**IPv4 Address:**
- 32 bits long
- Written as 4 groups of decimal numbers separated by dots (called **octets**)
- Each octet = 8 bits = values from 0 to 255

```
Example:
  192   .  168  .   1   .  10
 11000000.10101000.00000001.00001010
```

**IPv6 Address:**
- 128 bits long
- Written in hexadecimal (we will focus on IPv4 for CIDR learning)

---

### 1.2 Binary Number System

Computers think in **binary** (base-2): only 0s and 1s.

| Decimal | Binary    |
|---------|-----------|
| 0       | 00000000  |
| 1       | 00000001  |
| 128     | 10000000  |
| 192     | 11000000  |
| 255     | 11111111  |

**How to convert Decimal → Binary:**
Divide by 2 repeatedly and read remainders bottom-up.

```
192 ÷ 2 = 96  remainder 0
96  ÷ 2 = 48  remainder 0
48  ÷ 2 = 24  remainder 0
24  ÷ 2 = 12  remainder 0
12  ÷ 2 = 6   remainder 0
6   ÷ 2 = 3   remainder 0
3   ÷ 2 = 1   remainder 1
1   ÷ 2 = 0   remainder 1
Read bottom-up → 11000000 = 192 ✓
```

**Powers of 2 — memorize these:**

| Power | Value |
|-------|-------|
| 2^0   | 1     |
| 2^1   | 2     |
| 2^2   | 4     |
| 2^3   | 8     |
| 2^4   | 16    |
| 2^5   | 32    |
| 2^6   | 64    |
| 2^7   | 128   |
| 2^8   | 256   |
| 2^16  | 65,536 |
| 2^24  | 16,777,216 |

---

### 1.3 Network vs. Host Bits

An IP address has two parts:

```
| NETWORK PART  |  HOST PART  |
|     identifies |  identifies |
|   the network  |  the device |
```

- **Network bits** — identify WHICH network (like a city/street name)
- **Host bits** — identify WHICH device on that network (like a house number)

The **subnet mask** tells us where the boundary is.

```
IP Address :  192.168.1.10   →  11000000.10101000.00000001.00001010
Subnet Mask:  255.255.255.0  →  11111111.11111111.11111111.00000000
                                 ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^  ^^^^^^^^
                                      NETWORK PART (24 bits)    HOST (8 bits)
```

---

### 1.4 Subnet Mask

A **subnet mask** is a 32-bit number that "masks" the IP address to extract the network portion.

| Subnet Mask     | Binary Representation                     | CIDR |
|-----------------|-------------------------------------------|------|
| 255.0.0.0       | 11111111.00000000.00000000.00000000       | /8   |
| 255.255.0.0     | 11111111.11111111.00000000.00000000       | /16  |
| 255.255.255.0   | 11111111.11111111.11111111.00000000       | /24  |
| 255.255.255.128 | 11111111.11111111.11111111.10000000       | /25  |
| 255.255.255.192 | 11111111.11111111.11111111.11000000       | /26  |
| 255.255.255.224 | 11111111.11111111.11111111.11100000       | /27  |
| 255.255.255.240 | 11111111.11111111.11111111.11110000       | /28  |
| 255.255.255.248 | 11111111.11111111.11111111.11111000       | /29  |
| 255.255.255.252 | 11111111.11111111.11111111.11111100       | /30  |

**Rule:** A subnet mask is ALWAYS a contiguous block of 1s followed by 0s. No gaps allowed.

---

### 1.5 AND Operation — How Networks Are Calculated

To find the **Network Address** from an IP + Subnet Mask, you perform a **bitwise AND**:

```
1 AND 1 = 1
1 AND 0 = 0
0 AND 1 = 0
0 AND 0 = 0
```

Example:
```
IP:   192.168.1.10  →  11000000.10101000.00000001.00001010
Mask: 255.255.255.0 →  11111111.11111111.11111111.00000000
                       ─────────────────────────────────────
AND:  192.168.1.0   →  11000000.10101000.00000001.00000000
                                                  ↑ HOST bits zeroed out
```

The result **192.168.1.0** is the **Network Address**.

---

### 1.6 Key Network Addresses to Know

For any given network block:

| Address             | Meaning                                      |
|---------------------|----------------------------------------------|
| First address       | **Network Address** — identifies the network (not assignable to hosts) |
| Last address        | **Broadcast Address** — sends to ALL hosts on the network (not assignable) |
| In between          | **Usable Host Addresses**                    |

Example for `192.168.1.0/24`:
```
Network Address  : 192.168.1.0     ← not assignable
First usable host: 192.168.1.1
Last usable host : 192.168.1.254
Broadcast Address: 192.168.1.255   ← not assignable
Total usable hosts: 254  (= 2^8 - 2)
```

---

### 1.7 Private IP Address Ranges

These ranges are reserved for private (internal) use — they are NOT routable on the public Internet:

| Range                          | CIDR Block     | Common Use                     |
|-------------------------------|----------------|-------------------------------|
| 10.0.0.0 – 10.255.255.255     | 10.0.0.0/8     | Large enterprises, cloud VPCs  |
| 172.16.0.0 – 172.31.255.255   | 172.16.0.0/12  | Medium-sized networks          |
| 192.168.0.0 – 192.168.255.255 | 192.168.0.0/16 | Home/small office networks     |

> Your home router assigns 192.168.x.x addresses. These never reach the public Internet directly — NAT handles the translation.

---

## 2. The Problem CIDR Solves

### 2.1 Context: The Internet Was Running Out of IP Addresses

In the early days of the Internet (1980s), IP addresses were assigned using a **Classful** system. This system was rigid and extremely wasteful.

**The problem:**
- A large company might need 300 devices addressed
- The classful system forced you to assign a **Class B** block = **65,534 usable addresses**
- Result: **65,234 addresses wasted** in a single assignment
- The Internet's address space was being consumed at an alarming rate
- By 1992, experts predicted IPv4 address exhaustion within a few years

### 2.2 The Routing Table Explosion

Every network needed its own **entry in global routing tables** (the giant lookup tables that routers use to decide where to send packets).

- Classful assignments created thousands of small, fragmented network entries
- Router memory was expensive and limited
- Routing tables were growing out of control

**CIDR solved BOTH problems simultaneously.**

---

## 3. What is CIDR?

**CIDR** = **Classless Inter-Domain Routing**

- Pronounced: **"cider"** (like the apple drink)
- Introduced in: **1993** (RFC 1518 and RFC 1519)
- Replaced: The old Classful addressing system

### Definition

CIDR is a method for allocating IP addresses and routing IP packets. It replaced the fixed Classful (A/B/C) system with a **flexible, variable-length prefix** system that allows network sizes of any power of 2.

### The Core Idea

Instead of rigid class boundaries, CIDR lets you say:

> "I want a block of exactly 512 addresses" — and you get `/23`
> "I want a block of exactly 16 addresses" — and you get `/28`

The slash notation (e.g., `/24`) tells you how many bits are "fixed" (the network part).

---

## 4. IP Address Deep Dive

### 4.1 Structure of an IPv4 Address

```
 Octet 1   Octet 2   Octet 3   Octet 4
┌────────┬────────┬────────┬────────┐
│  8 bits│  8 bits│  8 bits│  8 bits│  = 32 bits total
└────────┴────────┴────────┴────────┘

Range per octet: 0 – 255
Total possible IPv4 addresses: 2^32 = 4,294,967,296 (~4.3 billion)
```

### 4.2 Why 4.3 Billion Wasn't Enough

When IPv4 was designed in 1981, 4.3 billion addresses seemed astronomical. But:
- Every phone, laptop, IoT device, smart TV needs an IP
- The world population is 8 billion people
- Companies, cloud providers, and datacenters consume millions of addresses
- Today IPv4 exhaustion is real — IANA allocated the last blocks in 2011

### 4.3 Special Address Ranges

| Address Block   | Purpose                                    |
|-----------------|--------------------------------------------|
| 0.0.0.0/8       | "This" network (source only)               |
| 127.0.0.0/8     | Loopback (localhost — 127.0.0.1)           |
| 169.254.0.0/16  | Link-local / APIPA (auto-assigned)         |
| 224.0.0.0/4     | Multicast                                  |
| 240.0.0.0/4     | Reserved for future use                    |
| 255.255.255.255 | Limited broadcast                          |
| 10.0.0.0/8      | Private (RFC 1918)                         |
| 172.16.0.0/12   | Private (RFC 1918)                         |
| 192.168.0.0/16  | Private (RFC 1918)                         |

---

## 5. Classful Networking

### 5.1 What Was Classful Networking?

Before CIDR, IP addresses were divided into fixed classes based on the first few bits of the address.

```
Class A: 0xxxxxxx.xxxxxxxx.xxxxxxxx.xxxxxxxx
         └─ starts with 0
         Network: 8 bits  | Host: 24 bits
         Range: 0.0.0.0 – 127.255.255.255
         Networks: 128    | Hosts per network: 16,777,214

Class B: 10xxxxxx.xxxxxxxx.xxxxxxxx.xxxxxxxx
         └─ starts with 10
         Network: 16 bits | Host: 16 bits
         Range: 128.0.0.0 – 191.255.255.255
         Networks: 16,384 | Hosts per network: 65,534

Class C: 110xxxxx.xxxxxxxx.xxxxxxxx.xxxxxxxx
         └─ starts with 110
         Network: 24 bits | Host: 8 bits
         Range: 192.0.0.0 – 223.255.255.255
         Networks: 2,097,152 | Hosts per network: 254

Class D: 1110xxxx... → Multicast (224.0.0.0 – 239.255.255.255)
Class E: 1111xxxx... → Reserved  (240.0.0.0 – 255.255.255.255)
```

### 5.2 The Classful Problem — A Real Example

A company (say, a medium-sized bank) needs **1,000 IP addresses**:

- Class C gives **254** hosts — too small
- Class B gives **65,534** hosts — way too many
- There was **nothing in between**

So they'd receive a **Class B** block. That wastes **64,534 addresses**.

Multiply this across thousands of companies... you see the problem.

### 5.3 Summary: Classful vs. Classless

| Feature             | Classful              | Classless (CIDR)         |
|---------------------|-----------------------|--------------------------|
| Block sizes         | Fixed (8/16/24 bits)  | Any prefix length        |
| Flexibility         | Very low              | Very high                |
| Address waste       | Extreme               | Minimal                  |
| Routing table size  | Large and fragmented  | Compact (route aggregation)|
| Still in use?       | No (obsolete since 1993) | Yes — universal today  |

---

## 6. How CIDR Works — The Mechanics

### 6.1 The Prefix Length

CIDR uses a **prefix length** (or prefix bits) to define the network boundary.

```
192.168.10.0/24
               ↑
           Prefix length = 24
           Means: first 24 bits are the NETWORK part
                  remaining 8 bits are the HOST part
```

### 6.2 Calculating the Block Size

```
Block size = 2^(32 - prefix_length)

/24  → 2^(32-24) = 2^8  = 256  addresses
/25  → 2^(32-25) = 2^7  = 128  addresses
/26  → 2^(32-26) = 2^6  = 64   addresses
/28  → 2^(32-28) = 2^4  = 16   addresses
/30  → 2^(32-30) = 2^2  = 4    addresses
/32  → 2^(32-32) = 2^0  = 1    address (single host)
```

### 6.3 Usable Hosts

```
Usable hosts = 2^(32 - prefix_length) - 2
```

We subtract 2 because:
- The **first address** = Network Address (not assignable)
- The **last address** = Broadcast Address (not assignable)

> Exception: In cloud environments (Azure, GCP, AWS), additional addresses may be reserved by the provider.

### 6.4 Finding the Network Address

**Step 1:** Write IP and mask in binary  
**Step 2:** AND them together  
**Step 3:** The result is the Network Address

Example: `10.0.5.130/26`

```
Prefix = 26 → mask = 255.255.255.192

IP  : 10.0.5.130  →  00001010.00000000.00000101.10000010
Mask: 255.255.255.192 → 11111111.11111111.11111111.11000000
AND :                  00001010.00000000.00000101.10000000
                     = 10.0.5.128

Network Address = 10.0.5.128
Block size = 2^6 = 64
Range: 10.0.5.128 – 10.0.5.191
Broadcast: 10.0.5.191
Usable: 10.0.5.129 – 10.0.5.190 (62 hosts)
```

### 6.5 CIDR Aggregation / Supernetting

CIDR also allows **route aggregation** — combining multiple small network blocks into one routing entry.

```
Before aggregation (4 separate routes in routing table):
  192.168.0.0/24
  192.168.1.0/24
  192.168.2.0/24
  192.168.3.0/24

After aggregation (1 route = supernet):
  192.168.0.0/22

This single /22 covers all 4 /24 blocks:
  192.168.0.0 – 192.168.3.255 (1024 addresses)
```

This dramatically **reduces routing table size** — one of CIDR's greatest contributions to Internet scalability.

---

## 7. CIDR Notation Explained

### 7.1 The Format

```
<IP Address> / <Prefix Length>

Examples:
  10.0.0.0/8
  172.16.0.0/12
  192.168.1.0/24
  10.10.5.64/26
```

### 7.2 What Does the Prefix Tell You?

| Prefix | Network Bits | Host Bits | Total Addresses | Usable Hosts  |
|--------|-------------|-----------|-----------------|---------------|
| /8     | 8           | 24        | 16,777,216      | 16,777,214    |
| /16    | 16          | 16        | 65,536          | 65,534        |
| /20    | 20          | 12        | 4,096           | 4,094         |
| /22    | 22          | 10        | 1,024           | 1,022         |
| /24    | 24          | 8         | 256             | 254           |
| /25    | 25          | 7         | 128             | 126           |
| /26    | 26          | 6         | 64              | 62            |
| /27    | 27          | 5         | 32              | 30            |
| /28    | 28          | 4         | 16              | 14            |
| /29    | 29          | 3         | 8               | 6             |
| /30    | 30          | 2         | 4               | 2             |
| /31    | 31          | 1         | 2               | 2* (point-to-point)|
| /32    | 32          | 0         | 1               | 1 (single host/loopback)|

---

## 8. Binary Math You Need

### 8.1 Quick Conversion Reference

| Octet Value | Binary    |
|-------------|-----------|
| 0           | 00000000  |
| 128         | 10000000  |
| 192         | 11000000  |
| 224         | 11100000  |
| 240         | 11110000  |
| 248         | 11111000  |
| 252         | 11111100  |
| 254         | 11111110  |
| 255         | 11111111  |

### 8.2 The "Magic Number" Trick

To quickly find the block boundary without full binary conversion:

```
Magic Number = 256 - subnet_mask_last_octet

Example: /26 → mask = 255.255.255.192
  Magic Number = 256 - 192 = 64

This means network blocks start at multiples of 64:
  0, 64, 128, 192

So valid /26 networks in 10.0.0.x are:
  10.0.0.0/26    (0–63)
  10.0.0.64/26   (64–127)
  10.0.0.128/26  (128–191)
  10.0.0.192/26  (192–255)
```

### 8.3 Summary Steps for Any CIDR Block

Given an IP and prefix (e.g., `172.20.100.75/20`):

1. **Identify mask:** /20 → 255.255.240.0
2. **Magic number:** 256 - 240 = 16 (applies to third octet)
3. **Find block:** 100 ÷ 16 = 6 (integer) → block starts at 6×16 = 96
4. **Network address:** 172.20.96.0
5. **Last address:** 172.20.111.255 (96 + 16 - 1 = 111)
6. **Broadcast:** 172.20.111.255
7. **Usable range:** 172.20.96.1 – 172.20.111.254
8. **Total hosts:** 2^12 = 4096, usable = 4094

---

## 9. CIDR Block Sizes — Quick Reference Table

| CIDR | Subnet Mask       | Total Addresses | Usable Hosts | Cloud Use Case              |
|------|-------------------|-----------------|--------------|------------------------------|
| /8   | 255.0.0.0         | 16,777,216      | 16,777,214   | Entire organization VPC      |
| /10  | 255.192.0.0       | 4,194,304       | 4,194,302    | Very large enterprise        |
| /12  | 255.240.0.0       | 1,048,576       | 1,048,574    | Large enterprise             |
| /16  | 255.255.0.0       | 65,536          | 65,534       | VNet/VPC in Azure/GCP        |
| /18  | 255.255.192.0     | 16,384          | 16,382       | Regional subnet              |
| /20  | 255.255.240.0     | 4,096           | 4,094        | Large subnet                 |
| /22  | 255.255.252.0     | 1,024           | 1,022        | Medium subnet                |
| /24  | 255.255.255.0     | 256             | 254          | Standard subnet              |
| /25  | 255.255.255.128   | 128             | 126          | Small subnet                 |
| /26  | 255.255.255.192   | 64              | 62           | Micro subnet                 |
| /27  | 255.255.255.224   | 32              | 30           | DMZ / Management             |
| /28  | 255.255.255.240   | 16              | 14           | Azure Gateway Subnet         |
| /29  | 255.255.255.248   | 8               | 6            | Point-to-point link          |
| /30  | 255.255.255.252   | 4               | 2            | WAN links, router interfaces |
| /32  | 255.255.255.255   | 1               | 1            | Host route, loopback         |

---

> **Next:** [02-Subnets-Deep-Dive.md](./02-Subnets-Deep-Dive.md) — Learn what subnets are, why they exist, and how to design them.
