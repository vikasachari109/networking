# Exercises — CIDR & Subnetting Practice

**30 exercises across 3 levels. Work each one yourself before checking the answer.**

---

## TABLE OF CONTENTS

1. [Easy Exercises (1–10)](#easy-exercises)
2. [Medium Exercises (11–20)](#medium-exercises)
3. [Hard Exercises (21–30)](#hard-exercises)
4. [Answer Key](#answer-key)

---

## EASY EXERCISES

*Goal: Build fluency with basic CIDR notation, block sizes, and address ranges.*

---

### Exercise 1
**What is the total number of IP addresses in the `192.168.10.0/24` block?**

A) 254  
B) 256  
C) 512  
D) 128  

---

### Exercise 2
**How many usable host addresses are available in a `/26` subnet?**

A) 64  
B) 62  
C) 30  
D) 126  

---

### Exercise 3
**Which of the following is a valid private IP address per RFC 1918?**

A) 172.32.0.1  
B) 192.169.1.1  
C) 10.255.255.1  
D) 203.0.113.5  

---

### Exercise 4
**Convert the decimal `192` to binary.**

Write your answer.

---

### Exercise 5
**What is the subnet mask for `/20`?**

A) 255.255.240.0  
B) 255.255.255.0  
C) 255.255.0.0  
D) 255.255.248.0  

---

### Exercise 6
**What is the network address for the IP `10.0.5.200/24`?**

A) 10.0.5.1  
B) 10.0.5.0  
C) 10.0.0.0  
D) 10.0.5.200  

---

### Exercise 7
**A `/30` subnet is often used for what type of connection?**

A) Large datacenter segments  
B) Point-to-point router links  
C) Enterprise VLANs  
D) Internet-facing web servers  

---

### Exercise 8
**You need to communicate with all devices on your subnet at once. Which address do you use?**

A) Default gateway  
B) Network address  
C) Broadcast address  
D) Loopback address  

---

### Exercise 9
**Which of the following correctly represents the CIDR notation for subnet mask `255.255.255.128`?**

A) /25  
B) /26  
C) /27  
D) /24  

---

### Exercise 10
**IP address `127.0.0.1` is known as what?**

A) Default gateway  
B) Multicast address  
C) Loopback / localhost  
D) Link-local address  

---

## MEDIUM EXERCISES

*Goal: Work through subnet calculations, VLSM design, and real scenario analysis.*

---

### Exercise 11
**Given: `172.16.50.0/20`**  
Calculate:
- Network address
- Broadcast address
- Usable host range
- Total usable hosts

---

### Exercise 12
**You are given `10.10.0.0/23`.  
How many `/26` subnets can you create from it?**

Show your working.

---

### Exercise 13
**Which subnet does the IP address `10.0.7.130` belong to if the network is subnetted as `/25`?**

The parent network is `10.0.7.0/24`. Identify which `/25` block this IP falls into and give the:
- Network address
- Broadcast address
- Usable range

---

### Exercise 14
**A company needs:**
- 1 subnet for Servers: 50 hosts
- 1 subnet for Users: 100 hosts
- 1 subnet for Printers: 10 hosts
- 1 subnet for Management: 5 hosts

They have `192.168.5.0/24` available.  
Using VLSM, assign appropriate subnets to each group. Show network address, prefix, and usable range.

---

### Exercise 15
**True or False — and explain why:**  
"In Azure, a `/28` subnet gives you 14 usable IP addresses for VMs."

---

### Exercise 16
**A router has the following routes in its table:**
```
10.0.0.0/8     → Interface A
10.10.0.0/16   → Interface B
10.10.5.0/24   → Interface C
```
Where will a packet destined for `10.10.5.200` be forwarded?  
Explain the rule that determines this.

---

### Exercise 17
**Identify all that are TRUE about GCP VPCs compared to Azure VNets:**

A) GCP VPCs are regional, Azure VNets are global  
B) GCP VPCs are global, Azure VNets are regional  
C) GCP reserves 4 IPs per subnet, Azure reserves 5  
D) Azure reserves 4 IPs per subnet, GCP reserves 5  
E) GCP firewall rules apply per subnet, like Azure NSGs  
F) GCP firewall rules apply VPC-wide and target resources by tag  

---

### Exercise 18
**You are designing an Azure VNet for a 3-tier application.  
The following subnets are required:**

| Subnet | Purpose | Required hosts |
|--------|---------|---------------|
| GatewaySubnet | VPN Gateway | — (use min size) |
| AzureFirewallSubnet | Azure Firewall | — (use min size) |
| web-subnet | Web servers | 40 VMs |
| app-subnet | App servers | 80 VMs |
| data-subnet | Databases | 15 VMs |

Assign appropriate CIDR blocks.  
Parent VNet: `10.5.0.0/16`

---

### Exercise 19
**Supernetting / Aggregation:**  
Can the following networks be summarized into a single CIDR block?
```
172.16.8.0/24
172.16.9.0/24
172.16.10.0/24
172.16.11.0/24
```
If yes, what is the supernet?

---

### Exercise 20
**You are setting up a GKE cluster in GCP.  
The node subnet is `10.20.0.0/22`.  
You need a secondary range for pods that supports up to 10,000 pods, and a secondary range for services that supports 500 services.**

Assign appropriate CIDR blocks for the pod range and service range.  
Justify your prefix lengths.

---

## HARD EXERCISES

*Goal: Multi-step design problems, real architecture decisions, troubleshooting.*

---

### Exercise 21
**Full Network Design:**

Your company has been assigned `10.50.0.0/16` for their Azure environment.  
Design subnets for the following requirements:

- Hub VNet containing: GatewaySubnet, AzureFirewallSubnet, AzureBastionSubnet, mgmt-subnet
- Spoke 1 (Production): web (150 VMs), app (200 VMs), data (40 VMs)
- Spoke 2 (Dev/Test): dev (50 VMs), test (30 VMs)
- All VNets must not overlap
- Leave 20% of the total space unallocated for future growth

Produce a subnet table with: VNet, Subnet Name, CIDR, Usable Hosts, Purpose.

---

### Exercise 22
**Troubleshooting — find the error:**

A network engineer has designed the following subnets for `192.168.100.0/23`:

| Subnet | CIDR |
|--------|------|
| Sales | 192.168.100.0/25 |
| Engineering | 192.168.100.128/25 |
| Finance | 192.168.101.0/25 |
| HR | 192.168.101.128/24 |

Find and explain every error in this design.

---

### Exercise 23
**CIDR Aggregation — Complex:**

You are a network engineer at an ISP. You have the following customer assignments:
```
Customer A: 203.0.113.0/26
Customer B: 203.0.113.64/26
Customer C: 203.0.113.128/26
Customer D: 203.0.113.192/26
```

1. Can all four be summarized? What is the supernet?
2. If you lose Customer C, can the remaining three be summarized into one announcement? If not, what is the minimum number of routes needed?
3. If you add Customer E at `203.0.114.0/25`, can you now create an even larger supernet covering Customers A, B, D, and E?

---

### Exercise 24
**Multi-Cloud Connectivity:**

You are designing a hybrid multi-cloud network.  
- On-premises network: `192.168.0.0/16`
- Azure VNet: You choose the CIDR
- GCP VPC: You choose the CIDR
- All three must communicate via VPN

Rules:
- No two networks can have overlapping CIDR blocks
- Azure VNet must be large enough for 3 subnets of ~500 VMs each
- GCP VPC must support 2 regions, 3 subnets per region, ~200 VMs per subnet
- Plan for future expansion of 30%

Design the full IP plan and explain your choices.

---

### Exercise 25
**Longest Prefix Match Routing:**

A router has the following routing table:
```
0.0.0.0/0         → Default (ISP)
10.0.0.0/8        → VPN Tunnel A
10.10.0.0/16      → VPN Tunnel B
10.10.5.0/24      → Local subnet (eth0)
10.10.5.128/25    → Local subnet (eth1)
192.168.0.0/16    → Local LAN
```

For EACH destination, state which route is used and why:
1. `10.10.5.200`
2. `10.10.5.10`
3. `10.99.0.1`
4. `8.8.8.8`
5. `192.168.1.50`

---

### Exercise 26
**Azure Reserved IP Impact:**

You are provisioning Azure VMs across multiple subnets.  
Given these subnets:

| Subnet | CIDR |
|--------|------|
| web-subnet | 10.0.1.0/27 |
| app-subnet | 10.0.2.0/26 |
| db-subnet | 10.0.3.0/29 |
| mgmt-subnet | 10.0.4.0/28 |

For each:
1. Total IPs in the block
2. Azure-reserved IPs (list them)
3. Actual usable IPs for VMs

Are any of these subnet sizes problematic? If so, why and what would you change?

---

### Exercise 27
**GCP Shared VPC Design:**

Your organization has:
- Host project: `network-host`
- Service project 1: `frontend-team`
- Service project 2: `backend-team`
- Service project 3: `data-team`

Requirements:
- Frontend: 2 subnets of 100 VMs in us-central1 and europe-west1
- Backend: 1 GKE cluster per region (us-central1), nodes: 50, pods: up to 2000, services: up to 256
- Data: 1 subnet of 20 VMs in us-central1, private Google access enabled
- No team subnet should be able to reach another team's data subnet directly (firewall enforcement by tag)

Design the full VPC, subnets (with secondary ranges), firewall rules, and project allocations.

---

### Exercise 28
**Subnet Exhaustion Scenario:**

Production environment:
```
app-subnet: 10.0.1.0/26 (Azure, 62 usable - 5 reserved = 57 effective)
Currently assigned: 55 VMs
```

You urgently need to add 20 more VMs to this subnet.  
The adjacent space `10.0.1.64/26` is also allocated to another team.

Propose two different solutions with pros/cons for each.  
Which would you implement in production and why?

---

### Exercise 29
**Security Zone Design:**

A financial services company must comply with PCI-DSS.  
They need to separate:
- Cardholder Data Environment (CDE) — payment systems
- Connected Systems — systems that communicate with CDE
- Out-of-scope systems — everything else
- Management — IT admin systems

Design a subnet architecture in Azure (`10.100.0.0/16`) that:
1. Enforces PCI-DSS network segmentation
2. Defines NSG rules between zones
3. Uses a separate subnet for Azure Firewall and Bastion
4. Leaves room for a future zone

---

### Exercise 30
**CIDR Math Challenge:**

Answer all of the following:

1. How many `/27` subnets fit inside a `/22`?
2. What is the 7th subnet of `10.0.0.0/24` when divided into `/27` blocks?
3. You have `10.0.0.0/22`. You want to carve out blocks for: one `/24`, two `/25`s, four `/26`s, and eight `/28`s. How many addresses remain unallocated?
4. A host has IP `172.31.200.148` and mask `255.255.255.240`. What are the network address, broadcast, and usable range?
5. Which is more specific: `10.0.0.0/12` or `10.10.0.0/20`? How many more host bits does the less specific route have?

---

## ANSWER KEY

---

### Easy Answers

**Exercise 1:** B — 256 (2^8)

**Exercise 2:** B — 62 (2^6 - 2 = 64 - 2 = 62)

**Exercise 3:** C — `10.255.255.1` is in the 10.0.0.0/8 range (RFC 1918). 172.32.x is outside 172.16-31 range.

**Exercise 4:** `11000000`
```
192 = 128 + 64 = 10000000 + 01000000 = 11000000
```

**Exercise 5:** A — `255.255.240.0`  (32-20=12 host bits → 4th octet all 0, 3rd octet: 256-16=240)

**Exercise 6:** B — `10.0.5.0` (AND with 255.255.255.0 zeros the last octet)

**Exercise 7:** B — Point-to-point router links (only 2 usable IPs)

**Exercise 8:** C — Broadcast address

**Exercise 9:** A — `/25` (255.255.255.128 → 11111111.11111111.11111111.10000000 → 25 ones)

**Exercise 10:** C — Loopback / localhost

---

### Medium Answers

**Exercise 11:**
```
172.16.50.0/20
Mask: 255.255.240.0
Magic number: 256 - 240 = 16 (applies to 3rd octet)
50 / 16 = 3 (integer) → block start = 3 × 16 = 48

Network address:   172.16.48.0
Broadcast address: 172.16.63.255  (48 + 16 - 1 = 63 in 3rd octet, last octet 255)
Usable host range: 172.16.48.1 – 172.16.63.254
Total usable hosts: 2^12 - 2 = 4094
```

**Exercise 12:**
```
10.10.0.0/23 → 2^(32-23) = 2^9 = 512 addresses
Each /26 = 2^(32-26) = 64 addresses
512 ÷ 64 = 8 subnets

Or use formula: 2^(26-23) = 2^3 = 8 subnets ✓
```

**Exercise 13:**
```
10.0.7.0/24 divided into /25:
  Subnet A: 10.0.7.0/25   → 10.0.7.0   – 10.0.7.127
  Subnet B: 10.0.7.128/25 → 10.0.7.128 – 10.0.7.255

IP 10.0.7.130:
  130 > 128 → falls in Subnet B

Network:   10.0.7.128
Broadcast: 10.0.7.255
Usable:    10.0.7.129 – 10.0.7.254
```

**Exercise 14:**
```
Sort largest first: 100, 50, 10, 5

100 hosts → /25 (126 usable)
  Network: 192.168.5.0/25     Range: 192.168.5.0 – 192.168.5.127

50 hosts → /26 (62 usable)
  Network: 192.168.5.128/26   Range: 192.168.5.128 – 192.168.5.191

10 hosts → /28 (14 usable)
  Network: 192.168.5.192/28   Range: 192.168.5.192 – 192.168.5.207

5 hosts → /29 (6 usable)
  Network: 192.168.5.208/29   Range: 192.168.5.208 – 192.168.5.215

Remaining: 192.168.5.216 – 192.168.5.255 (unallocated)
```

**Exercise 15:**
```
FALSE.

In Azure, every subnet has 5 reserved IPs (not 2 like on-prem):
  .0  → Network address
  .1  → Default gateway (Azure reserved)
  .2  → DNS (Azure reserved)
  .3  → Future use (Azure reserved)
  .x  → Broadcast

A /28 has 16 total addresses.
16 - 5 Azure reserved = 11 usable IPs for VMs (not 14).
```

**Exercise 16:**
```
Destination: 10.10.5.200

Matches:
  10.0.0.0/8     → yes (10.10.5.200 starts with 10)
  10.10.0.0/16   → yes (10.10.x.x matches)
  10.10.5.0/24   → yes (10.10.5.x matches) ← LONGEST prefix (/24)

Packet forwarded to Interface C.

Rule: Longest Prefix Match — routers always choose the most specific
(longest) matching route.
```

**Exercise 17:** B, C, F
```
B: Correct — GCP VPCs are global, Azure VNets are regional
C: Correct — GCP reserves 4, Azure reserves 5
F: Correct — GCP firewall rules apply to the whole VPC and use network tags or service accounts to target specific VMs

A: Wrong (reversed)
D: Wrong (numbers reversed)
E: Wrong — GCP firewall rules are not per-subnet, they're VPC-wide
```

**Exercise 18:**
```
Azure reserved IPs = 5 per subnet → add 5 to required hosts

GatewaySubnet: minimum /27 (Azure requirement)
  10.5.0.0/27   (32 - 5 = 27 usable, min required by Azure)

AzureFirewallSubnet: minimum /26 (Azure requirement)
  10.5.0.32/26  (64 - 5 = 59 usable, Azure requires exactly /26)

web-subnet: 40 VMs + 5 reserved = 45 → /26 (62-5=57 remaining, fits)
  10.5.0.96/26

app-subnet: 80 VMs + 5 reserved = 85 → /25 (128-5=123 remaining, fits)
  10.5.1.0/25

data-subnet: 15 VMs + 5 reserved = 20 → /27 (32-5=27 remaining, fits)
  10.5.1.128/27
```

**Exercise 19:**
```
172.16.8.0/24  → binary 3rd octet: 00001000
172.16.9.0/24  → binary 3rd octet: 00001001
172.16.10.0/24 → binary 3rd octet: 00001010
172.16.11.0/24 → binary 3rd octet: 00001011

Common prefix of 3rd octet: 000010xx → first 6 bits match
Total prefix: 16 (first two octets) + 6 = 22 bits → /22

Supernet: 172.16.8.0/22 ✓
Range: 172.16.8.0 – 172.16.11.255 (covers all 4 networks)
```

**Exercise 20:**
```
Nodes: 10.20.0.0/22 → 2^10 = 1024 addresses (1019 usable after GCP reserves)

Pods: Need 10,000 pods.
  2^h ≥ 10,000 → h = 14 → 2^14 = 16,384 IPs
  Use /18 (16,384) or /17 (32,768) for headroom
  Assign: 10.200.0.0/18  (16,384 addresses for pods)

Services: Need 500 services.
  2^h ≥ 500 → h = 9 → 2^9 = 512 IPs
  Use /23 (512) for exact fit or /22 (1024) for headroom
  Assign: 10.201.0.0/23  (512 addresses for ClusterIP services)

Note: Use non-overlapping ranges that don't conflict with node or VPC subnets.
```

---

### Hard Answers

**Exercise 21:**
```
Parent block: 10.50.0.0/16 (65,536 addresses)
20% reserved for future = keep ~13,000+ addresses free

Hub VNet: 10.50.0.0/20 (4,096)
  GatewaySubnet:        10.50.0.0/27    (32 - use /27 min)
  AzureFirewallSubnet:  10.50.0.64/26   (64 - required /26)
  AzureBastionSubnet:   10.50.1.0/26    (64 - required /26)
  mgmt-subnet:          10.50.2.0/27    (32, 27 usable)

Spoke 1 — Production: 10.50.16.0/20 (4,096)
  web-subnet:   10.50.16.0/24   (256 → 251 usable, fits 150+azure)
  app-subnet:   10.50.17.0/24   (256 → 251 usable, fits 200+azure)
  data-subnet:  10.50.18.0/26   (64 → 59 usable, fits 40+azure)

Spoke 2 — Dev/Test: 10.50.32.0/20 (4,096)
  dev-subnet:   10.50.32.0/26   (64 → 59 usable)
  test-subnet:  10.50.32.64/27  (32 → 27 usable)

Unallocated: 10.50.48.0 – 10.50.255.255 (~56,000 addresses = 85%+ free)
```

**Exercise 22:**
```
Errors found:

Error 1 — HR subnet CIDR is wrong:
  192.168.101.128/24 is invalid.
  A /24 must start on a boundary divisible by 256 (i.e., x.x.x.0).
  192.168.101.128/24 would claim 192.168.101.0–192.168.101.255,
  which OVERLAPS with Finance (192.168.101.0/25).
  Fix: Use 192.168.101.128/25 instead.

Error 2 — The parent block is 192.168.100.0/23:
  Valid range: 192.168.100.0 – 192.168.101.255
  All 4 subnets are within this range → no out-of-range error, but overlap still exists.

Summary of errors:
  1. HR uses /24 instead of /25 — creates OVERLAP with Finance subnet
```

**Exercise 23:**
```
1. All four subnets → Supernet?
   203.0.113.0/26   (0–63)
   203.0.113.64/26  (64–127)
   203.0.113.128/26 (128–191)
   203.0.113.192/26 (192–255)
   Together: 203.0.113.0 – 203.0.113.255 = 203.0.113.0/24 ✓

2. Without Customer C (203.0.113.128/26):
   Remaining: 203.0.113.0/26, 203.0.113.64/26, 203.0.113.192/26
   
   203.0.113.0/26 + 203.0.113.64/26 → 203.0.113.0/25 (contiguous) ✓
   203.0.113.192/26 → stands alone (not contiguous with 0/25 — missing 128/26)
   
   Minimum routes needed: 2
     203.0.113.0/25 and 203.0.113.192/26

3. Adding Customer E at 203.0.114.0/25:
   203.0.113.0/25 covers .0–.127
   203.0.113.192/26 covers .192–.255
   203.0.114.0/25 covers 203.0.114.0–.127
   
   These are non-contiguous (gap at 203.0.113.128/26 and 203.0.113.128–191).
   Cannot form a single supernet cleanly.
```

**Exercise 24:**
```
Rule: No overlaps. On-prem = 192.168.0.0/16.

Azure VNet:
  Need 3 subnets × ~500 VMs + 5 Azure reserved = ~505 per subnet
  Each subnet needs /23 (510 usable) or /22 (1018 usable)
  +30% growth → ~655 per subnet → /22 is safer
  3 × /22 = 3 × 1024 = 3072 minimum
  Use a /20 parent (4096) for the VNet with room
  Choose: 10.1.0.0/16 for Azure (non-overlapping with 192.168.0.0/16)
    web-subnet:  10.1.0.0/22
    app-subnet:  10.1.4.0/22
    data-subnet: 10.1.8.0/22

GCP VPC:
  2 regions × 3 subnets × ~200 VMs + growth
  200 × 1.3 = 260, so /24 (254-4=250 usable, close) → use /23 per subnet
  6 subnets × /23 = 6 × 512 = 3072 minimum
  Use another range, e.g., 10.2.0.0/16
    us-central1-web:    10.2.0.0/23
    us-central1-app:    10.2.2.0/23
    us-central1-data:   10.2.4.0/23
    europe-west1-web:   10.2.8.0/23
    europe-west1-app:   10.2.10.0/23
    europe-west1-data:  10.2.12.0/23

VPN Connectivity:
  On-prem (192.168.0.0/16) ↔ Azure VPN Gateway ↔ Azure (10.1.0.0/16)
  On-prem (192.168.0.0/16) ↔ GCP Cloud VPN ↔ GCP (10.2.0.0/16)
  Azure ↔ GCP: separate VPN or via on-prem hub

No overlaps:
  192.168.0.0/16 ≠ 10.1.0.0/16 ≠ 10.2.0.0/16 ✓
```

**Exercise 25:**
```
1. 10.10.5.200:
   Matches /8, /16, /24, /25 (200 > 128, so in 10.10.5.128/25)
   Longest match → 10.10.5.128/25 → eth1

2. 10.10.5.10:
   Matches /8, /16, /24 (10 < 128, not in .128/25)
   Longest match → 10.10.5.0/24 → eth0

3. 10.99.0.1:
   Matches 10.0.0.0/8 only
   → VPN Tunnel A

4. 8.8.8.8:
   No specific match → 0.0.0.0/0 (default route)
   → Default (ISP)

5. 192.168.1.50:
   Matches 192.168.0.0/16
   → Local LAN
```

**Exercise 26:**
```
web-subnet: 10.0.1.0/27
  Total: 32
  Reserved: .0, .1, .2, .3, .31 → 5 IPs
  Usable: 32 - 5 = 27 VMs ✓

app-subnet: 10.0.2.0/26
  Total: 64
  Reserved: .0, .1, .2, .3, .63 → 5 IPs
  Usable: 64 - 5 = 59 VMs ✓

db-subnet: 10.0.3.0/29
  Total: 8
  Reserved: .0, .1, .2, .3, .7 → 5 IPs
  Usable: 8 - 5 = 3 VMs ← PROBLEMATIC
  
  This is too small for any real DB workload.
  Azure SQL Managed Instance requires /27 minimum!
  Fix: Use 10.0.3.0/27 or larger.

mgmt-subnet: 10.0.4.0/28
  Total: 16
  Reserved: .0, .1, .2, .3, .15 → 5 IPs
  Usable: 16 - 5 = 11 VMs ✓ (acceptable for mgmt)
```

**Exercise 27:**
```
Shared VPC: prod-shared-vpc (custom mode, global)
Host project: network-host

Subnets:
  frontend-us-central1:   10.0.0.0/24   → allocated to: frontend-team
    Primary: 10.0.0.0/24 (nodes)
    
  frontend-europe-west1:  10.0.1.0/24   → allocated to: frontend-team

  backend-gke-us:         10.0.4.0/26   → allocated to: backend-team
    Primary (nodes):  10.0.4.0/26   (59 usable, fits 50 nodes)
    Secondary (pods): 10.100.0.0/19  (8192 — fits 2000 pods)
    Secondary (svcs): 10.101.0.0/24  (256 — fits 256 services)

  data-us-central1:       10.0.2.0/27   → allocated to: data-team
    private_ip_google_access: true

Firewall Rules:
  allow-https-to-frontend:    0.0.0.0/0 → tag:frontend, tcp:443, ALLOW, p:1000
  allow-backend-from-frontend: tag:frontend → tag:backend, tcp:8080, ALLOW, p:1000
  deny-data-from-frontend:    tag:frontend → tag:database, all, DENY, p:900
  deny-data-from-backend:     tag:backend → tag:database, all, DENY, p:900
  allow-data-from-backend-specific: tag:backend → tag:database, tcp:5432, ALLOW, p:800
  deny-all-ingress:           * → all, all, DENY, p:65534

Tags applied:
  Frontend VMs: frontend
  Backend VMs/GKE nodes: backend
  Data VMs: database
```

**Exercise 28:**
```
Problem: 10.0.1.0/26 (Azure, 62-5=57 usable) has 55 VMs, need 20 more = 75 needed.
Adjacent 10.0.1.64/26 is allocated to another team.

Solution 1: Resize the subnet (expand to /25)
  Pros:
    - Clean solution, single contiguous subnet
    - All VMs keep their IPs
    - No network changes on VMs
  Cons:
    - In Azure, you CANNOT resize a subnet while it has resources in it
    - Must deallocate all VMs in the subnet first (downtime!)
    - Adjacent 10.0.1.64/26 is taken — /25 would need 10.0.1.0–127,
      requiring the adjacent subnet to be cleared first too
  Verdict: Very disruptive for production.

Solution 2: Create a new subnet and split workloads
  New subnet: find available range in the VNet, e.g., 10.0.10.0/25 (123 usable)
  
  Pros:
    - No downtime on existing VMs
    - New 20 VMs go into new subnet
    - Can be done immediately
  Cons:
    - Two subnets for the "same" tier → needs NSG rules between them
    - Routing/app config must allow both subnets to function
    - Slightly more network complexity
  
  Verdict: RECOMMENDED for production. Zero downtime. Move traffic gradually.

Best practice: Avoid /26 for subnets that might grow. Always provision /24 or
/23 for app subnets to leave room for scaling.
```

**Exercise 29:**
```
PCI-DSS Azure Architecture: 10.100.0.0/16

Special subnets:
  AzureFirewallSubnet:  10.100.0.0/26    ← Central inspection point
  AzureBastionSubnet:   10.100.0.64/26   ← Admin access only

Security Zones:
  CDE (Cardholder Data):    10.100.1.0/24    ← Most restricted
  Connected Systems:        10.100.2.0/23    ← Can talk to CDE on specific ports
  Out-of-scope:             10.100.4.0/22    ← No CDE access allowed
  Management (PCI admin):   10.100.8.0/27    ← Restricted access to CDE read-only

Future Zone:              10.100.9.0/24    ← Reserved

NSG Rules:

CDE NSG (on 10.100.1.0/24):
  INBOUND:
    Allow TCP:443,8443 from Connected Systems (10.100.2.0/23) — p:100
    Allow TCP:22 from Bastion (10.100.0.64/26) — p:110
    Deny ALL from Out-of-scope (10.100.4.0/22) — p:200
    Deny ALL inbound — p:65500
  OUTBOUND:
    Allow TCP:443 to Azure Monitor Service Tag (logging) — p:100
    Deny ALL to Internet — p:200
    Deny ALL — p:65500

Connected Systems NSG (on 10.100.2.0/23):
  INBOUND:
    Allow HTTPS from Out-of-scope — p:100
    Deny ALL from Out-of-scope to DB ports — p:90
  OUTBOUND:
    Allow TCP:443 to CDE — p:100
    Deny ALL to CDE on other ports — p:200

All zones: UDR → 0.0.0.0/0 → Azure Firewall for traffic inspection.
```

**Exercise 30:**
```
1. How many /27 subnets in a /22?
   2^(27-22) = 2^5 = 32 subnets ✓

2. 7th subnet of 10.0.0.0/24 in /27 blocks:
   /27 block size = 32
   Subnets: .0, .32, .64, .96, .128, .160, .192 ← 7th
   Answer: 10.0.0.192/27

3. 10.0.0.0/22 = 4096 addresses
   Allocations:
     1 × /24 = 256
     2 × /25 = 2 × 128 = 256
     4 × /26 = 4 × 64  = 256
     8 × /28 = 8 × 16  = 128
   Total used: 256 + 256 + 256 + 128 = 896
   Remaining: 4096 - 896 = 3200 addresses unallocated ✓

4. 172.31.200.148 with mask 255.255.255.240 (/28)
   Magic number: 256 - 240 = 16
   148 / 16 = 9 (integer) → block starts at 9 × 16 = 144
   Network:   172.31.200.144
   Broadcast: 172.31.200.159
   Usable:    172.31.200.145 – 172.31.200.158

5. 10.0.0.0/12  vs  10.10.0.0/20
   /20 is more specific (longer prefix = smaller block = more specific)
   10.0.0.0/12 has (32-12) = 20 host bits
   10.10.0.0/20 has (32-20) = 12 host bits
   Less specific is /12, which has 20-12 = 8 MORE host bits than /20.
   (2^8 = 256 times as many addresses in the /12 block)
```

---

> **Start Over:** [01-Prerequisites-and-CIDR-Fundamentals.md](./01-Prerequisites-and-CIDR-Fundamentals.md) to review concepts before re-attempting hard exercises.
