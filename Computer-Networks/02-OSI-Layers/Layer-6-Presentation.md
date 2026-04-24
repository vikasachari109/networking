# Layer 6: Presentation Layer

## Definition
The **Presentation Layer** (Layer 6) is responsible for data formatting, encryption, and compression. It prepares data for the application layer.

### Key Responsibility
Translate, encrypt, compress, and format data so the application can understand it.

---

## Presentation Layer Concepts

### 1. **Data Translation**

```
Different systems represent data differently:

Character Encoding:
ASCII:   'A' = 01000001
EBCDIC:  'A' = 11000001 (IBM mainframes)

Layer 6 converts between formats
```

### 2. **Encryption** (Security)

```
Plain Data → Encrypt → Encrypted Data → Network
Encrypted Data → Decrypt → Plain Data → Application

Example: HTTPS
HTTP (unencrypted) → Encrypt at L6 → Transport → Network
Receive at L6 → Decrypt → HTTP → Application
```

### 3. **Compression**

```
Large File (10 MB) → Compress to 2 MB → Transmit → Decompress → Original

Benefits:
- Faster transmission
- Less bandwidth used
- Faster downloads
```

---

## Layer 6 Protocols & Technologies

### **SSL/TLS** (Encryption)

```
HTTPS = HTTP + SSL/TLS
Secures web traffic
Connection: HTTP → TCP → SSL/TLS → Encrypted data
```

#### TLS Versions
| Version | Status | Security |
|---------|--------|----------|
| SSL 2.0 | Obsolete | Insecure |
| SSL 3.0 | Deprecated | Weak |
| TLS 1.0 | Deprecated | Weak |
| TLS 1.1 | Deprecated | Weak |
| TLS 1.2 | Current | Strong |
| TLS 1.3 | Latest | Strongest |

### **JPEG, PNG, GIF** (Image Compression)

```
Original image: 5 MB (uncompressed)
JPEG compressed: 200 KB
PNG compressed: 800 KB
GIF compressed: 400 KB
```

### **MPEG, H.264** (Video Compression)

```
Raw video: 100 GB per hour
H.264 compressed: 1 GB per hour
Massive reduction for streaming
```

### **GZIP** (Data Compression)

```
text/html file: 50 KB
gzipped: 10 KB (80% reduction)
```

### **MIME Types** (Data Format Identification)

```
text/html       → HTML webpage
application/json → JSON data
image/png       → PNG image
video/mp4       → MP4 video
application/pdf → PDF document
```

---

## Encryption Deep Dive

### **Symmetric Encryption**
```
Same key to encrypt and decrypt

Sender: "Hello" + key → "XYZABC" (encrypted)
Network: "XYZABC"
Receiver: "XYZABC" + key → "Hello"

Fast but key sharing is problem
```

### **Asymmetric Encryption**
```
Different key to encrypt and decrypt
Public key for encryption
Private key for decryption

Sender: "Hello" + public_key → "XYZABC"
Receiver: "XYZABC" + private_key → "Hello"

Slower but safer key distribution
```

### **Hashing** (One-way)
```
Data → Hash → Fixed-size string
Cannot reverse

Example: password hashing
password "secret123" → hash "abc123def456"
```

---

## Certificate-Based Security (HTTPS)

### **SSL/TLS Handshake**

```
CLIENT                              SERVER
  │                                   │
  ├─────── ClientHello ──────────────→
  │  "I support TLS 1.2, these ciphers"
  │                                   │
  │ ←────── ServerHello ──────────────┤
  │ ←── ServerCertificate ────────────┤
  │  (contains public key)            │
  │                                   │
  ├────── ClientKeyExchange ────────→
  │  (encrypted with public key)      │
  │                                   │
  ├────── ChangeCipherSpec ──────────→
  │ ←── ChangeCipherSpec ─────────────┤
  │                                   │
  └──── Encrypted Communication ─────┘
```

### **Certificate Components**

```
Certificate contains:
- Server's public key
- Server's domain name
- Certificate authority signature
- Validity dates
- Encryption algorithms
```

---

## Important L6 Concepts

### **Lossy vs Lossless Compression**

| Type | Data Loss | Use Case |
|------|-----------|----------|
| **Lossless** | None | ZIP, GZIP, PNG (all data recoverable) |
| **Lossy** | Some | JPEG, MP3 (minor details lost, imperceptible) |

### **Character Encoding**

```
ASCII:     7-bit, English only
UTF-8:     Variable-length, all Unicode
UTF-16:    2-3 bytes per character
ISO-8859:  8-bit, European languages
```

### **Data Format Standards**

```
JSON:  Lightweight, human-readable
XML:   Verbose, structured
YAML:  Human-readable, minimal syntax
Protocol Buffers: Compact, fast
```

---

## Presentation Layer Issues

### **Certificate Errors**

```
"Certificate expired"
"Certificate not trusted"
"Certificate mismatch" (domain doesn't match)
→ Solution: Update certificate, renew, or fix domain
```

### **Encryption Failures**

```
"Decryption failed"
"Unable to verify signature"
→ Solution: Check key, verify certificate
```

### **Compression Issues**

```
"Decompression error"
"Unsupported compression"
→ Solution: Use supported algorithm
```

---

## DevOps Relevance: MEDIUM (⭐⭐)

**When DevOps handles L6:**
- HTTPS/TLS configuration
- Certificate management
- Certificate renewal (Let's Encrypt)
- Compression tuning
- Encryption key management
- Certificate pinning in apps
- Supporting compression (gzip, brotli)

**Common L6 Issues:**
- "SSL certificate expired"
- "Certificate not trusted"
- "Certificate mismatch error"
- "TLS handshake failure"
- "Weak cipher suite"

---

## Troubleshooting Layer 6

```bash
# Check SSL/TLS certificate
openssl s_client -connect example.com:443

# View certificate expiry
openssl x509 -in cert.pem -noout -dates

# Check certificate validity
openssl verify -CAfile ca.pem cert.pem

# Generate self-signed certificate
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365

# Test TLS version
curl -I --tlsv1.2 https://example.com

# Check supported ciphers
openssl ciphers -v
```

---

## HTTPS Configuration Example

```nginx
server {
    listen 443 ssl http2;
    server_name example.com;
    
    # Certificate files
    ssl_certificate /etc/ssl/certs/cert.pem;
    ssl_certificate_key /etc/ssl/private/key.pem;
    
    # TLS version
    ssl_protocols TLSv1.2 TLSv1.3;
    
    # Strong ciphers only
    ssl_ciphers HIGH:!aNULL:!MD5;
    
    # Compression (use with caution for CRIME attacks)
    gzip on;
    gzip_types text/plain text/css application/json;
}
```

---

## Summary Table

| Aspect | Details |
|--------|---------|
| **Function** | Format, encrypt, compress data |
| **Data Unit** | Data |
| **Key Technologies** | SSL/TLS, Gzip, JPEG, UTF-8 |
| **Responsibilities** | Encryption, compression, translation |
| **Protocols** | SSL/TLS, SSH, HTTPS |
| **DevOps Focus** | Medium (cert management) |

---

## Quick Reference

**Layer 6 = Security & Formatting**
- Encryption (SSL/TLS)
- Compression (gzip, brotli)
- Translation (character encoding)
- Format conversion (JSON, XML)

---

## Related Layers

- **Previous Layer**: [Layer 5: Session](Layer-5-Session.md) - Session management
- **Next Layer**: [Layer 7: Application](Layer-7-Application.md) - User applications
- **Overview**: [OSI Model Overview](../01-Fundamentals/OSI-Model-Overview.md)

**For DevOps, focus on SSL/TLS and certificate management. Then see [Layer 7: Application](Layer-7-Application.md).**
