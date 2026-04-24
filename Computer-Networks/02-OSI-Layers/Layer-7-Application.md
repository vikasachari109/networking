# Layer 7: Application Layer (CRITICAL FOR DevOps & SRE)

## Definition

The **Application Layer** (Layer 7) is the **topmost layer** where users and applications interact with the network. It provides network services directly to user applications.

### Core Concept
```
User/Application ──> Application Layer ──> Network
                     (HTTP, HTTPS, DNS, 
                      SSH, FTP, Email, etc)

HTTP Request → Layer 7 → Layer 6 (encrypt) → Layer 5 (session)
            → Layer 4 (TCP) → Layer 3 (routing) → ...
```

---

## Key Functions

### 1. **User Interface & Services**
- Provide services to end-users
- Interface for applications
- User-friendly interaction

### 2. **Resource Sharing**
- File sharing (FTP, SMB)
- Printer sharing (IPP)
- Document collaboration

### 3. **Network Services**
- DNS: Domain name resolution
- DHCP: IP address assignment
- NTP: Time synchronization

### 4. **Email Services**
- SMTP: Sending emails
- POP3/IMAP: Receiving emails
- S/MIME: Secure email

### 5. **Web Services**
- HTTP/HTTPS: Web browsing
- APIs: Application programming interfaces
- Web sockets: Real-time communication

### 6. **Remote Access**
- SSH: Secure shell
- RDP: Remote desktop
- VNC: Virtual network computing

---

## Common Layer 7 Protocols

### Web & Internet Protocols

| Protocol | Port | Function | Use |
|----------|------|----------|-----|
| **HTTP** | 80 | Web pages | Websites |
| **HTTPS** | 443 | Secure web | Secure websites |
| **DNS** | 53 | Name resolution | Domain → IP |
| **DHCP** | 67/68 | IP assignment | Auto config |
| **NTP** | 123 | Time sync | Clock accuracy |
| **TFTP** | 69 | Simple file transfer | Bootup |
| **FTP** | 20/21 | File transfer | Large files |
| **SFTP** | 22 | Secure FTP | Secure files |
| **SSH** | 22 | Secure shell | Remote access |

### Email Protocols

| Protocol | Port | Function |
|----------|------|----------|
| **SMTP** | 25/587 | Send mail |
| **POP3** | 110 | Receive mail |
| **IMAP** | 143 | Advanced mail |
| **S/MIME** | - | Secure email |

### Directory & Auth Protocols

| Protocol | Port | Function |
|----------|------|----------|
| **LDAP** | 389 | Directory service |
| **LDAPS** | 636 | Secure LDAP |
| **Kerberos** | 88 | Authentication |
| **Radius** | 1812 | Access control |

### Management Protocols

| Protocol | Port | Function |
|----------|------|----------|
| **SNMP** | 161 | Network monitoring |
| **Syslog** | 514 | System logs |
| **Telnet** | 23 | Remote shell (insecure) |
| **SSH** | 22 | Secure shell |

---

## HTTP/HTTPS in Detail (Most Common for DevOps)

### HTTP Protocol

```
Request/Response Model:

Client                              Server
  │                                  │
  ├─ HTTP Request ────────────────> │
  │  GET /index.html HTTP/1.1       │
  │  Host: example.com              │
  │  User-Agent: ...                │
  │  [headers]                      │
  │  [optional body]                │
  │                                 │
  │ <────── HTTP Response ──────────┤
  │  HTTP/1.1 200 OK                │
  │  Content-Type: text/html        │
  │  Content-Length: 1234           │
  │  [headers]                      │
  │  [body: HTML content]           │
  │                                  │
  └─ (Connection close or keep-alive)
```

### HTTP Status Codes (Important for DevOps)

```
1xx: Informational
  100: Continue
  101: Switching protocols

2xx: Success ✓
  200: OK (success)
  201: Created
  204: No Content
  206: Partial Content

3xx: Redirection
  301: Moved Permanently
  302: Found (temporary)
  304: Not Modified (cached)
  307: Temporary Redirect

4xx: Client Error
  400: Bad Request
  401: Unauthorized
  403: Forbidden
  404: Not Found
  429: Too Many Requests

5xx: Server Error
  500: Internal Server Error
  502: Bad Gateway
  503: Service Unavailable
  504: Gateway Timeout
```

### HTTP Methods

| Method | Purpose | Safe | Idempotent |
|--------|---------|------|-----------|
| **GET** | Retrieve | Yes | Yes |
| **POST** | Create | No | No |
| **PUT** | Update | No | Yes |
| **DELETE** | Delete | No | Yes |
| **PATCH** | Partial update | No | No |
| **HEAD** | Get headers only | Yes | Yes |
| **OPTIONS** | Get allowed methods | Yes | Yes |

### HTTP Versions

```
HTTP/1.0: Simple, one request per connection
HTTP/1.1: Persistent connection, pipelining
HTTP/2: Multiplexing, header compression, server push
HTTP/3: QUIC protocol, faster, more reliable
```

---

## DNS (Domain Name System)

### How DNS Works

```
User: "Go to google.com"
         │
         ├─ Resolver: "Who knows google.com?"
         │
         ├─ Root NS: "Ask TLD server for .com"
         │
         ├─ TLD NS: "Ask Authoritative NS for google"
         │
         ├─ Authoritative NS: "google.com = 142.251.41.14"
         │
         ├─ Resolver caches it
         │
         └─ Returns 142.251.41.14 to user

Browser: "Connect to 142.251.41.14"
```

### DNS Record Types

| Type | Purpose | Example |
|------|---------|---------|
| **A** | IPv4 address | example.com → 93.184.216.34 |
| **AAAA** | IPv6 address | example.com → 2606:2800::1 |
| **CNAME** | Alias | www → example.com |
| **MX** | Mail server | example.com → mail.example.com |
| **TXT** | Text (SPF, DKIM) | For email security |
| **NS** | Name server | Points to DNS server |
| **SOA** | Start of Authority | Zone info |
| **SRV** | Service | Kubernetes, SIP |

### DNS Troubleshooting

```bash
# Query DNS
nslookup google.com
dig google.com
host google.com

# Detailed DNS lookup
dig +trace google.com         # Show all steps
dig @8.8.8.8 google.com       # Use specific resolver

# Check DNS resolution
getent hosts google.com       # System's DNS view
cat /etc/resolv.conf          # DNS servers used

# Reverse DNS (IP → domain)
nslookup 8.8.8.8
dig -x 8.8.8.8
```

---

## DHCP (Dynamic Host Configuration Protocol)

### DHCP Process (DORA)

```
Device                        DHCP Server
  │                              │
  ├─ Discover (broadcast) ──────>│ "Need an IP"
  │                              │
  │ <─────── Offer ──────────────┤ "Here's 192.168.1.100"
  │                              │
  ├─ Request ────────────────────>│ "I'll take it"
  │                              │
  │ <────── Acknowledge ─────────┤ "Confirmed, lease 24h"
  │                              │
  └─ Now has IP 192.168.1.100
```

### Lease Renewal

```
At 50% lease time (12 hours):
├─ Renewal request sent
├─ Server grants extension or new IP
└─ Lease refreshed

At 87.5% lease time (21 hours):
├─ Rebinding starts (broadcast)
├─ Any DHCP server responds
└─ Lease extended
```

---

## Advantages of Application Layer

✅ **User-Friendly**: Direct interaction with users  
✅ **Service Variety**: Many protocols for different needs  
✅ **Standardization**: Well-defined protocols (HTTP, DNS, FTP)  
✅ **Flexibility**: Can add new protocols easily  
✅ **Interoperability**: Different vendors, same protocols  
✅ **High-Level Functions**: Authentication, authorization, etc.  

---

## Disadvantages of Application Layer

❌ **Complexity**: Many protocols to learn  
❌ **Performance**: Additional processing overhead  
❌ **Debugging**: Difficult to troubleshoot issues  
❌ **Security Exposure**: Direct user interaction increases risk  
❌ **Variability**: Different apps use L7 differently  
❌ **Bandwidth Usage**: Unoptimized applications waste bandwidth  

---

## DevOps & SRE Critical Topics at L7

### Web Server Configuration

```
Common servers:
- Nginx: Lightweight, high performance
- Apache: Feature-rich, modular
- IIS: Windows-based
- Caddy: Modern, automatic HTTPS

Key configs:
- Gzip compression
- Keep-alive connections
- Request timeouts
- Max connections
- Log levels
```

### API & Web Service Monitoring

```
Monitor at L7:
- Response time (latency)
- Error rates (4xx, 5xx)
- Request rate (throughput)
- Payload size
- Cache hit ratio

Tools:
- Prometheus
- Datadog
- New Relic
- ELK Stack
```

### Health Checks

```
Service health checks:

Simple (L4):
- TCP port open?

Better (L7):
- GET /health → 200 OK?
- GET /health → Response time < 1s?
- Database query works?

Example Kubernetes:
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 10
  periodSeconds: 10
```

### Load Balancing at L7

```
L7 Load Balancing:

Can route based on:
- URL path (/api → api-server, /static → cdn)
- Hostname (api.example.com → api-lb, www.example.com → web-lb)
- HTTP method (POST → heavy-worker, GET → cache)
- Cookie value (special-user → premium-server)
- Request size (large → special-server)

Examples:
- Nginx
- HAProxy
- AWS ALB
- Kubernetes Ingress
```

### Common L7 Troubleshooting

```
Problem: High response time
Debug:
1. Check server CPU/memory
2. Check database queries
3. Check external API calls
4. Check network latency (L3/L4)

Problem: High error rate
Debug:
1. Check application logs
2. Check database connectivity
3. Check service dependencies
4. Check resource limits
```

### DevOps Commands for L7

```bash
# Test HTTP connectivity
curl -v http://example.com         # Verbose output
curl -I http://example.com         # Headers only
curl -w '%{http_code}' ...         # Just status code

# Check DNS resolution
curl -v http://example.com         # Shows DNS lookup time

# Test with specific headers
curl -H "User-Agent: Mozilla/5.0" http://example.com
curl -H "X-Forwarded-For: 1.2.3.4" http://example.com

# Follow redirects
curl -L http://example.com         # Follow 301/302

# Test specific port
curl http://example.com:8080

# HTTP/2 test
curl -I --http2 https://example.com

# Measure performance
curl -w "@curl-format.txt" -o /dev/null -s http://example.com

# Apache bench (load test)
ab -n 1000 -c 10 http://example.com/

# Test with authentication
curl -u username:password http://example.com

# Check DNS specifically
dig @8.8.8.8 +short example.com
nslookup example.com 8.8.8.8

# Test HTTPS certificate
openssl s_client -connect example.com:443
```

---

## DevOps Scenarios at L7

### Scenario 1: Website Returns 502 (Bad Gateway)

```
Symptom: Users see "502 Bad Gateway"

Troubleshooting Steps:
1. Check backend servers are running
   curl http://backend-ip:8080
   ssh backend-server; systemctl status app

2. Check load balancer connectivity
   telnet backend-ip 8080
   Check load balancer logs: /var/log/nginx/error.log

3. Check health checks
   Is health check endpoint returning 200?
   curl http://backend:8080/health

4. Check backend capacity
   Top, htop - CPU/Memory usage?
   Connection pool size reached?

5. Check logs
   Application logs, access logs, error logs
   kubectl logs pod-name

Solution:
- Restart service: systemctl restart app
- Scale up: kubectl scale deployment app --replicas=5
- Check dependencies: Database, cache, external APIs
```

### Scenario 2: Slow API Response

```
Symptom: API responses taking 5+ seconds (normal: <1s)

Troubleshooting:
1. Check network (L3/L4)
   ping backend
   traceroute backend
   Latency should be <100ms

2. Check server performance
   top, htop - CPU, memory, disk usage
   iostat - disk I/O issues
   vmstat - context switches

3. Check application logs
   Error messages?
   Debug mode info?
   Slow query logs?

4. Check external dependencies
   Database response time
   API call times
   Cache hits/misses

5. Check network service (L7)
   Response body size? (1GB response = slow)
   Gzip compression enabled?
   Connection pooling?
   Keep-alive enabled?

Solution:
- Optimize queries
- Add caching
- Scale horizontally
- Compress responses
- Profile application
```

### Scenario 3: DNS Not Resolving

```
Symptom: Can't reach example.com

Troubleshooting:
1. Check local DNS
   nslookup example.com
   dig example.com

2. Check with public DNS
   nslookup example.com 8.8.8.8
   dig @8.8.8.8 example.com

3. Check DNS server is reachable
   telnet ns.example.com 53
   ping ns.example.com

4. Check DNS records
   dig example.com +trace (all DNS steps)
   dig ns example.com (name servers)
   dig a example.com (A records)

5. Check /etc/resolv.conf
   Should point to correct DNS servers
   
Solution:
- Add nameserver to /etc/resolv.conf
- Restart DNS service: systemctl restart systemd-resolved
- Check DNS registrar settings
- Wait for propagation (24-48 hours)
```

---

## Quick Reference Diagram

```
┌──────────────────────────────────────┐
│   APPLICATION LAYER (L7)             │
├──────────────────────────────────────┤
│ SERVICES: Web, Email, DNS, SSH, FTP │
│ PROTOCOLS: HTTP, HTTPS, SMTP, POP3  │
│ UNITS: MESSAGES, STREAMS             │
│ SCOPE: User applications             │
│ DEVICES: Web servers, Email servers  │
│ KEY: User interaction, Services      │
└──────────────────────────────────────┘
```

---

## Important Points for Interview

1. **L7 is where users interact**
2. **HTTP/HTTPS most common protocols**
3. **DNS maps domain → IP address**
4. **Each service uses different protocol**
5. **L7 monitoring crucial for user experience**
6. **Health checks at L7 most accurate**
7. **Load balancing at L7 most flexible**
8. **API Gateway operates at L7**
9. **Web server configuration = L7 tuning**
10. **Log analysis = understanding L7 issues**

---

## Quick Summary

| Aspect | Details |
|--------|---------|
| **Main Job** | User services & applications |
| **Protocols** | HTTP, HTTPS, DNS, SSH, SMTP, FTP |
| **Services** | Web, Email, Remote access, File transfer |
| **Data Unit** | Messages, Streams |
| **Scope** | User applications |
| **DevOps Focus** | Web servers, APIs, health checks |
| **Key Concern** | Performance, availability, user experience |

---

**Advanced Topics**: [Networking Concepts](../04-Core-Concepts/Networking-Concepts.md)
