# Real-World Scenarios — CIDR, Subnets, Azure VNet & GCP VPC

---

## TABLE OF CONTENTS

1. [Real-World CIDR Scenarios](#1-real-world-cidr-scenarios)
2. [Real-World Subnet Scenarios](#2-real-world-subnet-scenarios)
3. [Azure Virtual Network (VNet) — Deep Dive](#3-azure-vnet-deep-dive)
4. [GCP Virtual Private Cloud (VPC) — Deep Dive](#4-gcp-vpc-deep-dive)
5. [Azure vs. GCP — Side-by-Side Comparison](#5-azure-vs-gcp-comparison)
6. [Real-World Cloud Architecture Examples](#6-real-world-cloud-architecture)

---

## 1. Real-World CIDR Scenarios

### Scenario 1: ISP Allocating IP Space to Customers

An Internet Service Provider (ISP) receives `203.0.113.0/24` from ARIN (the IP registry). They need to allocate addresses to business customers:

```
ISP Address Pool: 203.0.113.0/24  (256 addresses)

Customer A — Small business (20 devices):
  Assigned: 203.0.113.0/27  → 30 usable hosts
  Range: 203.0.113.0 – 203.0.113.31

Customer B — Medium office (60 devices):
  Assigned: 203.0.113.32/26 → 62 usable hosts
  Range: 203.0.113.32 – 203.0.113.95

Customer C — Large office (100 devices):
  Assigned: 203.0.113.96/25 → 126 usable hosts
  Range: 203.0.113.96 – 203.0.113.223

ISP Management:
  Available: 203.0.113.224/27 → 203.0.113.224 – 203.0.113.255
```

Without CIDR, the ISP would have to give each customer a separate Class C network — wasting entire /24 blocks. CIDR allows efficient carving.

---

### Scenario 2: Route Aggregation at an Internet Exchange Point

A datacenter operator has multiple customers with the following networks:

```
192.168.0.0/24
192.168.1.0/24
192.168.2.0/24
192.168.3.0/24
```

Without aggregation: 4 separate routing announcements to the entire Internet.

With CIDR aggregation: These share the same first 22 bits → announce ONE supernet:
```
192.168.0.0/22  ← covers all 4 networks in a single routing entry
```

This is how Tier-1 ISPs manage the global routing table (BGP). Without CIDR aggregation, the global BGP routing table would have millions more entries.

---

### Scenario 3: Home Network Behind a Router

Your home ISP gives you ONE public IP address (say `203.0.113.50`).

Inside your home:
```
Router assigns private CIDR block: 192.168.1.0/24
  Your laptop:  192.168.1.100
  Your phone:   192.168.1.101
  Smart TV:     192.168.1.102
  Router:       192.168.1.1 (gateway)
```

**NAT (Network Address Translation)** converts:
- Outbound: `192.168.1.100:54321` → `203.0.113.50:54321` (public IP)
- Inbound: `203.0.113.50:54321` → `192.168.1.100:54321` (back to device)

CIDR enables you to run 254 private devices behind 1 public IP — critical for IPv4 conservation.

---

### Scenario 4: Datacenter Micro-Segmentation

A datacenter hosts services for multiple tenants. They allocate:

```
Tenant A (E-commerce): 10.10.0.0/20   (4096 addresses)
  Web tier:   10.10.0.0/24
  App tier:   10.10.1.0/24
  DB tier:    10.10.2.0/24
  Cache tier: 10.10.3.0/24

Tenant B (FinTech):    10.10.16.0/20  (4096 addresses)
  Web tier:   10.10.16.0/24
  App tier:   10.10.17.0/24
  PCI zone:   10.10.18.0/26   ← tighter control on payment systems

Management plane:      10.255.0.0/16  (shared infra)
  Monitoring:   10.255.0.0/24
  Bastion:      10.255.1.0/27
```

Tenants are completely isolated at the IP level — a firewall rule simply blocks `10.10.x.x ↔ 10.10.16.x` traffic.

---

### Scenario 5: CIDR in BGP — How the Internet Routes

When you visit `google.com`, your ISP looks up the IP in a BGP routing table. Google announces:

```
8.8.8.0/24  ← where Google's DNS lives
```

Your ISP has a route that says: "Any traffic to 8.8.8.0/24 → send to Google's upstream router."

**Longest prefix match rule:** If a router has two matching routes, it always prefers the more specific (longer prefix) one:

```
Routes in table:
  10.0.0.0/8    → Router A
  10.10.0.0/16  → Router B
  10.10.5.0/24  → Router C

Traffic to 10.10.5.50:
  Matches /8   ✓
  Matches /16  ✓
  Matches /24  ✓ ← LONGEST match → Router C wins
```

This is the **longest prefix match** — fundamental to how ALL routers make forwarding decisions.

---

## 2. Real-World Subnet Scenarios

### Scenario 1: Hospital Network

A hospital needs strict separation between:
- Medical devices (patient monitors, infusion pumps)
- Staff workstations
- Guest WiFi
- Administrative systems
- Clinical imaging (PACS — huge bandwidth)

```
Hospital Network: 172.16.0.0/12

Medical Devices:   172.16.0.0/16
  ICU devices:     172.16.1.0/24   ← most restricted
  Ward devices:    172.16.2.0/24
  Lab equipment:   172.16.3.0/24

Staff Network:     172.17.0.0/16
  Doctors:         172.17.0.0/22   (1000+ staff)
  Nurses:          172.17.4.0/22
  Admin:           172.17.8.0/23

Guest WiFi:        172.18.0.0/20   (Internet only, no internal access)

Imaging (PACS):    172.19.0.0/20   (high bandwidth, isolated)

Firewall rules:
  Guest → Staff:          DENY ALL
  Guest → Medical:        DENY ALL
  Staff → Medical:        ALLOW specific ports only
  Medical → Internet:     DENY ALL (security requirement)
  Imaging → Internet:     DENY ALL
```

Life-critical devices can NEVER reach the internet. This is enforced purely by subnet-level firewall rules.

---

### Scenario 2: E-Commerce Platform (3-Tier Architecture)

```
Architecture Goal: Separate presentation, logic, and data tiers

Network Block: 10.0.0.0/16

DMZ (Public-facing):   10.0.0.0/24
  Load Balancers:      10.0.0.0/27   (30 usable — enough for LBs)
  WAF appliances:      10.0.0.32/27
  CDN origin servers:  10.0.0.64/26

Application Tier:      10.0.1.0/24
  Web servers:         10.0.1.0/25   (126 hosts)
  API servers:         10.0.1.128/25

Database Tier:         10.0.2.0/24
  Primary DB:          10.0.2.0/28   (14 hosts — small, controlled)
  Read replicas:       10.0.2.16/28
  Redis cache:         10.0.2.32/28

Traffic Flow:
  Internet → Load Balancer (DMZ)
           → Web Servers (App Tier)
           → API Servers (App Tier)
           → Database (DB Tier)

Security:
  DB Tier has NO outbound internet access
  DB Tier accepts connections ONLY from App Tier subnet
  DMZ has NO access to DB Tier directly
```

---

### Scenario 3: Remote Branch Office Connected via VPN

```
HQ Network:    10.1.0.0/16   (London)
Branch A:      10.2.0.0/16   (New York)
Branch B:      10.3.0.0/16   (Singapore)
VPN Tunnels:   10.255.0.0/29 per tunnel (point-to-point /30 links)

Routing:
  HQ router knows:  10.2.0.0/16 → VPN Tunnel to New York
                    10.3.0.0/16 → VPN Tunnel to Singapore
  
  NY router knows:  10.1.0.0/16 → VPN Tunnel to HQ
                    10.3.0.0/16 → VPN Tunnel through HQ

All non-RFC-1918 traffic routed to Internet through HQ (hub-and-spoke)
or directly to Internet per-branch (split tunneling)
```

---

## 3. Azure VNet Deep Dive

### 3.1 What is an Azure VNet?

An Azure **Virtual Network (VNet)** is the fundamental networking building block in Azure. It is your **logically isolated private network** in the Azure cloud.

```
Azure VNet ≈ your own private datacenter network in the cloud

You define:
  - The IP address space (CIDR block)
  - Subnets
  - Security rules (NSGs)
  - Routing behavior (UDRs)
  - DNS settings
  - Peering to other VNets
```

### 3.2 VNet Key Properties

| Property | Details |
|----------|---------|
| Address Space | One or more CIDR blocks (e.g., 10.0.0.0/16, can add more) |
| Region-scoped | A VNet exists in ONE Azure region |
| Subnet | Must be within VNet address space; cannot overlap |
| No Internet by default | VMs have no inbound internet unless you assign a Public IP or use a Load Balancer |
| Free service | VNet itself is free; data transfer and gateways cost money |

### 3.3 Azure Reserved IPs Per Subnet

**Critical fact:** Azure reserves **5 IP addresses** in every subnet you create:

```
Subnet: 10.0.1.0/24  (256 total addresses)

10.0.1.0   → Network address (reserved)
10.0.1.1   → Reserved by Azure (default gateway)
10.0.1.2   → Reserved by Azure (DNS mapping)
10.0.1.3   → Reserved by Azure (future use)
10.0.1.255 → Broadcast (reserved)

Usable addresses: 256 - 5 = 251  (NOT 254 like on-prem!)

For a /29 subnet (8 total):
Usable: 8 - 5 = 3 hosts only!
```

**This means you should NEVER use /29 or smaller for VM workloads in Azure.**  
Minimum practical subnet size: `/28` (16 addresses - 5 reserved = 11 usable).

### 3.4 Azure Subnet Types

| Subnet Type | Example Name | Use Case | Special Requirements |
|-------------|-------------|----------|---------------------|
| General-purpose | `web-subnet`, `app-subnet` | VMs, containers | None |
| GatewaySubnet | `GatewaySubnet` (exact name required) | VPN Gateway, ExpressRoute | Min /27, recommend /26 or /27 |
| AzureFirewallSubnet | `AzureFirewallSubnet` (exact name required) | Azure Firewall | Min /26 |
| AzureBastionSubnet | `AzureBastionSubnet` (exact name required) | Bastion host (secure RDP/SSH) | Min /26 |
| RouteServerSubnet | `RouteServerSubnet` (exact name required) | Azure Route Server | Min /27 |

> If you misname a special subnet, the service deployment will fail.

### 3.5 Network Security Groups (NSGs)

NSGs are Azure's subnet/NIC-level firewall. They are **stateful** (you only need to define inbound OR outbound, not both).

```
NSG Rules have:
  - Priority:    100–4096 (lower number = higher priority)
  - Direction:   Inbound or Outbound
  - Protocol:    TCP, UDP, ICMP, * (any)
  - Source/Dest: IP, CIDR, Service Tag, Application Security Group
  - Action:      Allow or Deny

Default rules (cannot delete, only override):
  AllowVNetInBound    (priority 65000) — Allow all traffic within VNet
  AllowAzureLoadBalancerInBound (65001) — Allow Azure health probes
  DenyAllInBound      (priority 65500) — Deny everything else
```

Example NSG for an App Subnet:
```
INBOUND RULES:
Priority  Name                   Source          Dest     Port  Action
100       Allow-HTTPS-from-LB    10.0.0.0/27     *        443   Allow
200       Allow-SSH-from-Bastion 10.0.3.0/27     *        22    Allow
300       Allow-Health-Probe     AzureLoadBalancer *      *     Allow
65500     DenyAllInbound         *               *        *     Deny

OUTBOUND RULES:
100       Allow-DB-Access        *               10.0.2.0/24 5432 Allow
200       Allow-Redis            *               10.0.2.32/28 6379 Allow
300       Deny-Internet          *               Internet  *    Deny
65001     Allow-VNet             *               VirtualNetwork * Allow
```

### 3.6 User-Defined Routes (UDR)

By default, Azure routes traffic between subnets automatically. UDRs override this:

```
Scenario: Force all outbound internet traffic through Azure Firewall

Route Table (applied to AppSubnet):
  Destination     Next Hop Type     Next Hop IP
  0.0.0.0/0       Virtual Appliance  10.0.0.132   ← Azure Firewall IP
  10.0.0.0/8      VNet Local         (default)
```

### 3.7 VNet Peering

VNets in different regions or subscriptions can communicate via **VNet Peering** (no gateway required, low latency, uses Microsoft backbone):

```
VNet A: 10.0.0.0/16  (East US)
VNet B: 10.1.0.0/16  (West Europe)

Peering connects them — 10.0.x.x can reach 10.1.x.x directly

Requirements:
  - Address spaces MUST NOT overlap
  - Peering is non-transitive (A↔B + B↔C ≠ A↔C, unless using hub-spoke with NVA)
```

### 3.8 Real-World Azure Architecture: Production 3-Tier App

```
VNet: myapp-prod-vnet  (10.0.0.0/16)
Region: East US

Subnets:
┌─────────────────────────────────────────────────────────┐
│ GatewaySubnet        10.0.0.0/27   — VPN/ER Gateway     │
│ AzureFirewallSubnet  10.0.1.0/26   — Azure Firewall      │
│ AzureBastionSubnet   10.0.2.0/26   — Bastion (SSH/RDP)   │
│ web-subnet           10.0.3.0/24   — App Service, VMs    │
│ app-subnet           10.0.4.0/24   — Backend APIs        │
│ data-subnet          10.0.5.0/24   — Azure SQL, Cosmos DB│
│ mgmt-subnet          10.0.6.0/27   — Monitoring, Jump VMs│
└─────────────────────────────────────────────────────────┘

Traffic Flow:
Internet → Azure Front Door → Azure Firewall (10.0.1.0/26)
        → Web Subnet (10.0.3.0/24) via UDR
        → App Subnet (10.0.4.0/24) internal routing
        → Data Subnet (10.0.5.0/24) private endpoint only

Remote Admin → Azure Bastion (10.0.2.0/26)
             → mgmt-subnet (10.0.6.0/27) VMs

VPN Gateway (10.0.0.0/27) connects on-prem → VNet peered to hub
```

---

## 4. GCP VPC Deep Dive

### 4.1 What is a GCP VPC?

A GCP **Virtual Private Cloud (VPC)** is fundamentally different from Azure VNet in one critical way:

> **GCP VPCs are GLOBAL.** A single VPC spans ALL GCP regions.

```
Azure VNet:     Regional (1 VNet = 1 Region)
GCP VPC:        GLOBAL   (1 VPC = All Regions)

Azure subnets:  Regional scope (in a VNet)
GCP subnets:    Regional scope (in a VPC, but VPC is global)
```

### 4.2 GCP VPC Key Concepts

| Concept | Description |
|---------|-------------|
| VPC | Global networking container. Does NOT have a single CIDR — subnets define ranges. |
| Subnet | Region-scoped. Has a primary CIDR range. |
| Secondary Range | Additional CIDR ranges on a subnet (used for GKE pods/services) |
| Alias IP | Multiple IPs on a single VM NIC (from secondary ranges) |
| Shared VPC | One VPC shared across multiple GCP projects |
| VPC Peering | Connect two VPCs (can be in different projects/organizations) |
| Cloud Router | Manages dynamic routing (BGP) for VPN and Interconnect |

### 4.3 GCP Reserved IPs Per Subnet

GCP reserves **4 IP addresses** per subnet (one fewer than Azure):

```
Subnet: 10.128.0.0/20 (default-us-central1)

10.128.0.0   → Network address (reserved)
10.128.0.1   → Default gateway (reserved by GCP)
10.128.0.2   → Reserved by GCP
10.128.15.255 → Broadcast (reserved)

Usable: 4096 - 4 = 4092 addresses
```

### 4.4 GCP Default VPC

Every new GCP project gets a **default VPC** with pre-created subnets in every region:

```
Default VPC subnets (auto mode — 1 per region):
  us-central1:            10.128.0.0/20
  us-east1:               10.142.0.0/20
  us-west1:               10.138.0.0/20
  europe-west1:           10.132.0.0/20
  asia-east1:             10.140.0.0/20
  ... (one /20 per region)
```

**Auto mode vs. Custom mode:**

| Mode | Description | Recommended For |
|------|-------------|-----------------|
| Auto mode | Automatically creates one /20 subnet per region | Dev/test, quick prototyping |
| Custom mode | You define every subnet | Production — full control |

**Best practice:** Always use **Custom mode** for production.

### 4.5 GCP Firewall Rules

GCP Firewalls are **VPC-wide** (not subnet-level like Azure NSG). They are also **stateful**.

```
GCP Firewall Rule components:
  - Direction: INGRESS or EGRESS
  - Priority: 0–65535 (lower = higher priority)
  - Target: All instances, network tag, service account
  - Source/Dest: CIDR, network tag, service account
  - Protocol/Port: tcp:80, udp:53, icmp, etc.
  - Action: ALLOW or DENY

Default rules:
  default-allow-internal (priority 65534) — Allow all traffic within VPC
  default-allow-ssh      (priority 65534) — Allow SSH from anywhere ← DANGEROUS in prod!
  default-allow-rdp      (priority 65534) — Allow RDP from anywhere ← DANGEROUS!
  default-allow-icmp     (priority 65534) — Allow ping from anywhere
```

**Critical:** Disable `default-allow-ssh` and `default-allow-rdp` in production. Use **IAP (Identity-Aware Proxy)** tunneling for secure SSH/RDP instead.

GCP Firewall Example for production:
```
Rule: allow-web-ingress
  Direction: INGRESS
  Priority: 1000
  Target tags: web-server
  Source: 0.0.0.0/0
  Ports: tcp:443
  Action: ALLOW

Rule: allow-app-from-web
  Direction: INGRESS
  Priority: 1000
  Target tags: app-server
  Source tags: web-server
  Ports: tcp:8080
  Action: ALLOW

Rule: allow-db-from-app
  Direction: INGRESS
  Priority: 1000
  Target tags: database
  Source tags: app-server
  Ports: tcp:5432
  Action: ALLOW

Rule: deny-all-ingress
  Direction: INGRESS
  Priority: 65534
  Target: All instances
  Source: 0.0.0.0/0
  Ports: all
  Action: DENY
```

### 4.6 GCP Secondary IP Ranges — GKE Use Case

When running **Google Kubernetes Engine (GKE)** on GCP, pods and services need their own IP ranges. This is where secondary ranges come in:

```
Subnet: k8s-subnet
  Primary range:        10.0.0.0/24     ← Node IPs (VMs)
  Secondary range 1:    10.10.0.0/16    ← Pod IPs  (alias IPs on node)
  Secondary range 2:    10.20.0.0/20    ← ClusterIP services

GKE cluster config:
  nodeIpv4CidrBlock:    10.0.0.0/24     (from primary)
  podIpv4CidrBlock:     10.10.0.0/16    (from secondary #1)
  servicesIpv4CidrBlock: 10.20.0.0/20  (from secondary #2)
```

This ensures Kubernetes pod IPs are routable within the VPC — GCP native networking.

### 4.7 GCP Private Google Access

A subnet's VMs can reach Google APIs without a public IP:

```
Normal access:   VM → Internet → Google API (e.g., Cloud Storage)
Private access:  VM → Google's private network → Google API
                       (no Internet exposure, faster, more secure)

Enable per-subnet:
  private_ip_google_access = true
```

### 4.8 Real-World GCP Architecture: Multi-Region Production

```
VPC: prod-vpc  (custom mode, global)

Subnets:
┌──────────────────────────────────────────────────────────────────────┐
│ us-central1-web     10.0.0.0/24      Web tier (us-central1)          │
│ us-central1-app     10.0.1.0/24      App tier (us-central1)          │
│ us-central1-data    10.0.2.0/24      Data tier (us-central1)         │
│ us-central1-gke     10.0.4.0/22      GKE nodes (us-central1)         │
│   └─ secondary pods 10.100.0.0/14   GKE pods                         │
│   └─ secondary svcs 10.104.0.0/20   GKE services                     │
│                                                                        │
│ europe-west1-web    10.1.0.0/24      Web tier (europe-west1)          │
│ europe-west1-app    10.1.1.0/24      App tier (europe-west1)          │
│ europe-west1-data   10.1.2.0/24      Data tier (europe-west1)         │
└──────────────────────────────────────────────────────────────────────┘

Cloud Armor (WAF) → Global HTTP LB → us-central1-web + europe-west1-web
                                           ↓
                                   App tier (same region)
                                           ↓
                                   Cloud SQL Private IP (data subnet)

Cloud VPN / Interconnect → Cloud Router → BGP → on-prem network

IAP Tunnel → Jump VMs in each region (no public SSH exposure)
```

---

## 5. Azure vs. GCP Comparison

| Feature | Azure VNet | GCP VPC |
|---------|-----------|---------|
| Scope | Regional | **Global** |
| Subnet scope | Regional (within VNet) | Regional (VPC is global) |
| Reserved IPs per subnet | **5** | **4** |
| Default gateway | x.x.x.1 | x.x.x.1 |
| Firewall granularity | Subnet or NIC (NSG) | VPC-wide (firewall rules) |
| Firewall identity | IP / CIDR / Service Tag | IP / CIDR / **Network Tag** / Service Account |
| Inter-region connectivity | VNet Peering across regions | Automatic within same VPC |
| Cross-project networks | VNet in each subscription | **Shared VPC** across projects |
| Special subnets | GatewaySubnet, AzureFirewallSubnet, etc. | No special naming required |
| Kubernetes networking | Azure CNI (subnet IPs) | **Alias IP** with secondary ranges |
| Private service access | Private Endpoint | Private Service Connect |
| Minimum subnet size | /29 (not recommended) | /29 (not recommended) |
| Practical min for workloads | /28 (11 usable after reservations) | /29 (4 usable after reservations) |
| Route tables | Route Table resource (UDR) | Routes on VPC directly |
| NAT | NAT Gateway (separate resource) | Cloud NAT (separate resource) |
| Load Balancer placement | In subnet (private LB) | Not in subnet (external/internal) |

---

## 6. Real-World Cloud Architecture Examples

### Architecture 1: Azure Hub-and-Spoke for Enterprise

```
This is the most common Azure enterprise pattern.

HUB VNet (Shared Services): 10.0.0.0/16
  GatewaySubnet:           10.0.0.0/27    ← ExpressRoute/VPN to on-prem
  AzureFirewallSubnet:     10.0.1.0/26    ← Azure Firewall (central)
  AzureBastionSubnet:      10.0.2.0/26    ← Bastion for all spokes
  mgmt-subnet:             10.0.3.0/27    ← Monitoring, NTP, DNS

SPOKE 1 — Production (10.1.0.0/16) [Peered to Hub]
  web-subnet:    10.1.0.0/24
  app-subnet:    10.1.1.0/24
  data-subnet:   10.1.2.0/24

SPOKE 2 — Dev/Test (10.2.0.0/16) [Peered to Hub]
  dev-subnet:    10.2.0.0/24
  test-subnet:   10.2.1.0/24

SPOKE 3 — DMZ / Partners (10.3.0.0/16) [Peered to Hub]
  partner-subnet: 10.3.0.0/24            ← External partners only

On-premises: 192.168.0.0/16
  Connected via ExpressRoute → GatewaySubnet in Hub
  Route: 192.168.0.0/16 → On-prem via ER Gateway

ALL traffic between spokes goes THROUGH the Hub firewall.
No direct spoke-to-spoke communication (prevents lateral movement).

UDR on all spoke subnets:
  0.0.0.0/0 → 10.0.1.4 (Azure Firewall private IP)
```

---

### Architecture 2: GCP Shared VPC for Multi-Team Organization

```
Organization: mycompany.com
  Host Project:    shared-network-project  (owns the VPC)
  Service Projects: frontend-project, backend-project, data-project

shared-prod-vpc (custom mode, global):

Subnets (with secondary ranges for GKE):
  frontend-subnet   10.0.0.0/24  (us-central1)
    ← allocated to: frontend-project
    ← Network tag: frontend
    
  backend-subnet    10.0.1.0/24  (us-central1)
    ← allocated to: backend-project
    ← Network tag: backend
    
  data-subnet       10.0.2.0/24  (us-central1)
    ← allocated to: data-project
    ← Network tag: database

  gke-subnet        10.0.4.0/22  (us-central1)
    ← Nodes: 10.0.4.0/22
    ← Pods (secondary): 10.10.0.0/14
    ← Services (secondary): 10.104.0.0/20
    ← allocated to: backend-project (GKE cluster)

Firewall Rules (in host project, apply to all):
  allow-https-to-frontend: 0.0.0.0/0 → tag:frontend, tcp:443
  allow-backend-from-frontend: tag:frontend → tag:backend, tcp:8080-8090
  allow-db-from-backend: tag:backend → tag:database, tcp:5432,6379
  deny-ingress-default: 0.0.0.0/0 → all, all → DENY (priority 65534)

Cloud NAT (for private instances to reach Internet):
  NAT gateway in us-central1 → apply to all subnets
  (Private VMs have no public IP, but can initiate outbound connections)

Cloud VPN:
  On-prem: 192.168.0.0/16 → via Cloud VPN → Cloud Router (BGP) → prod-vpc
```

---

### Architecture 3: Disaster Recovery — Azure Primary + GCP Secondary

```
This is an advanced multi-cloud scenario.

Azure (Primary):
  VNet: 10.0.0.0/16
  Production runs here normally

GCP (Secondary / DR):
  VPC: 10.1.0.0/16
  Mirror of production, kept warm (scale to zero between DR tests)

Connectivity:
  Azure VPN Gateway ←IPSec VPN→ GCP Cloud VPN
  BGP sharing routes bidirectionally
  10.0.0.0/16 reachable from GCP
  10.1.0.0/16 reachable from Azure

DNS:
  Azure DNS zones + GCP Cloud DNS in sync
  Global traffic manager points *.app.com to Azure (primary)
  On failover: TTL-60 record flipped to GCP IP

Key CIDR rules:
  ✓  Azure VNet and GCP VPC MUST NOT overlap (10.0.x vs 10.1.x)
  ✓  On-prem (192.168.0.0/16) also must not overlap either cloud
  ✓  Kubernetes pod CIDRs (10.10.x and 10.20.x) also non-overlapping
```

---

> **Next:** [04-Exercises.md](./04-Exercises.md) — Practice what you've learned with 30 exercises across three difficulty levels.
