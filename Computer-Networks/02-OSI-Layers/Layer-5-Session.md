# Layer 5: Session Layer

## Definition
The **Session Layer** (Layer 5) is responsible for establishing, maintaining, and terminating communication sessions between applications.

### Key Responsibility
Manage the conversation (session) between two applications. Handle login, logout, session timeouts, and conversation rules.

---

## Session Layer Concepts

### 1. **What is a Session?**

A session is a logical connection between two applications.

```
LOGIN → ACTIVE SESSION → LOGOUT
│           │              │
Start   Exchange data   Close
```

#### Example: Web Session
```
1. User clicks login
2. Browser sends credentials to server
3. Server creates session
4. Browser gets session ID (cookie)
5. Browser sends session ID with each request
6. Server recognizes user
7. User logs out
8. Server terminates session
```

### 2. **Session Tokens/IDs**

```
Server: "Login successful! Your session ID is: abc123xyz"
Client: Stores this ID in cookie
Client: With every request, sends cookie containing session ID
Server: Validates session ID, remembers who you are
```

### 3. **Session Management**

| Aspect | Description |
|--------|-------------|
| **Creation** | User login, authentication |
| **Maintenance** | Keep session alive (heartbeat) |
| **Validation** | Check session ID with each request |
| **Termination** | User logout or timeout |
| **Security** | Prevent session hijacking |

---

## Layer 5 Protocols

### **RPC** (Remote Procedure Call)
```
Call a function on another computer as if local:
result = remote_server.calculate(x, y)
```

### **SMB** (Server Message Block)
- File sharing (Windows)
- Printer sharing
- Network browsing

### **NFS** (Network File System)
- UNIX/Linux file sharing
- Mount remote directories

### **SSH** (Secure Shell)
- Secure remote login
- Manages session state
- Encryption at presentation layer

### **PPTP** (Point-to-Point Tunneling Protocol)
- VPN protocol
- Creates virtual session

### **LDAP** (Lightweight Directory Access Protocol)
- User authentication
- Directory services
- Session management for enterprise

### **H.323** (VoIP Protocol)
- Video conferencing
- Session setup for calls

---

## Session vs Cookie Confusion

### **Session (L5)**
```
Server maintains state about user
Example:
  In-memory: user_sessions = {
    "abc123": {"user": "john", "login_time": 1234567}
  }
```

### **Cookie (L7)**
```
Client stores small data file
Example:
  Browser cookie: session_id=abc123; path=/; secure
```

### **Relationship**
```
Session ID → Stored in Cookie → Sent with each request
Cookie is transport, session is state
```

---

## Important L5 Concepts

### **Session Establishment**

```
CLIENT                    SERVER
  │                         │
  ├─ Request Session ───────→
  │                         │
  │ ←── Session Token ──────┤
  │     (e.g., abc123)      │
  │                         │
  └─ Use Token ─────────────→
    with each request
```

### **Session Timeout**

```
User logs in → Session created
User inactive for 30 minutes → Session times out
User tries to access → "Session expired, login again"
```

### **Concurrent Sessions**

```
User logs in on Phone → Session A
Same user logs in on Laptop → Session B
Two separate sessions (or replace old with new)
```

### **Session Hijacking** (Security Issue)

```
Attacker steals session token
Attacker impersonates user
Solution: Use HTTPS, regenerate session IDs
```

---

## Stateful vs Stateless Architectures

### **Stateful (Traditional)**
```
Server remembers every request
Server stores sessions in memory/database
Example: Traditional web apps, SSH

Advantages:
- Can enforce strict order
- Can maintain context
- Better for complex interactions

Disadvantages:
- Doesn't scale well (sticky sessions needed)
- Server must remember everything
- Memory overhead
```

### **Stateless (Modern)**
```
Server doesn't store session state
Client sends all needed info with each request
Example: REST APIs, JWT tokens

Advantages:
- Scales horizontally
- No server memory needed
- Easy to distribute load
- Simpler infrastructure

Disadvantages:
- Client must send all info
- Harder to invalidate sessions
- Larger request size
```

---

## JWT (JSON Web Tokens) - Modern Session Management

```
Traditional Session:
User → Server → Server stores session → Server checks session

JWT:
User → Server → Server creates signed token → User keeps token
User sends token with request → Server verifies signature → No storage needed
```

### JWT Structure
```
Header.Payload.Signature
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.
eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIn0.
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

---

## DevOps Relevance: LOW-MEDIUM (⭐⭐)

**When DevOps handles L5:**
- API session management
- VPN configuration (PPTP, OpenVPN)
- Load balancer session persistence (sticky sessions)
- SSH session management
- Container health checks (session validation)

**Common L5 Issues:**
- "Session timeout"
- "Session already exists"
- "Session hijacking/security"
- "Sticky session not working"
- "User logged out unexpectedly"

---

## Troubleshooting Layer 5

```bash
# Check SSH session
ssh user@host
# Monitor active sessions
who
w

# View network sessions
netstat -anp | grep ESTABLISHED

# Check if service maintaining sessions
curl -b cookies.txt -c cookies.txt http://example.com

# View session files (some apps)
ls /tmp/sessions/
```

---

## Session in Container/Cloud

```
Kubernetes Pod:
Multiple replicas of same app
Each has own memory
Session stored in Pod A
User request goes to Pod B
Session not found → "login again"

Solution:
- Store sessions in shared cache (Redis)
- Use stateless architecture (JWT)
- Use session affinity (sticky sessions)
```

---

## Summary Table

| Aspect | Details |
|--------|---------|
| **Function** | Establish/manage conversations |
| **Data Unit** | Message/Data |
| **Key Concept** | Session tokens, cookies |
| **Protocols** | RPC, SMB, SSH, LDAP, VoIP |
| **State** | Stateful (traditional) or Stateless (modern) |
| **DevOps Focus** | Medium (relevant for APIs, VPN) |

---

## Quick Reference

**Session = Conversation between apps**
- Created at login
- Maintained during activity
- Terminated at logout
- Identified by session ID/token
- Stored in memory or database

---

## Related Layers

- **Previous Layer**: [Layer 4: Transport](Layer-4-Transport.md) - Connection management
- **Next Layer**: [Layer 6: Presentation](Layer-6-Presentation.md) - Encryption, compression
- **Layer 7**: [Layer 7: Application](Layer-7-Application.md) - Uses sessions
- **Overview**: [OSI Model Overview](../01-Fundamentals/OSI-Model-Overview.md)

**Most DevOps focus is on Layer 3, 4, and 7. Layer 5 is less critical unless handling authentication/VPN.**
