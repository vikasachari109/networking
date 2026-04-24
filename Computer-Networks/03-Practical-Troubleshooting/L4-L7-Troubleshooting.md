# L4 & L7 Transport & Application Layer Troubleshooting

This guide covers practical troubleshooting for Layer 4 (Transport) and Layer 7 (Application) - the most critical layers for DevOps/SRE engineers.

---

## Layer 4 (Transport) Troubleshooting

### Understanding the Problem

Layer 4 problems involve:
- Port connectivity issues
- TCP connection states
- UDP delivery problems
- Flow control and congestion

### Common L4 Problems

#### 1. "Connection Refused"

**What it means:**
```
Client tried to connect to server:port
Server is NOT listening on that port
```

**Troubleshooting:**

```bash
# Step 1: Check if service is running
ps aux | grep -i service-name

# Step 2: Check if port is listening
netstat -tlnp | grep :8080
ss -tlnp | grep :8080          # Modern alternative

# Step 3: Verify service listening on correct port
cat /etc/service/config.yml    # Check config
systemctl status service-name  # Check status

# Step 4: Check if firewall is blocking
sudo iptables -L -n | grep 8080
sudo firewall-cmd --list-all

# Step 5: Test from different machine
ssh user@remote-server
telnet localhost 8080

# Step 6: Try connecting
curl -v telnet://localhost:8080
```

**Solutions:**
```bash
# Start service
systemctl start service-name

# Change port in config if needed
vim /etc/service/config.yml
systemctl restart service-name

# Open firewall port
sudo iptables -A INPUT -p tcp --dport 8080 -j ACCEPT
sudo firewall-cmd --add-port=8080/tcp

# Verify
netstat -tlnp | grep :8080
```

#### 2. "Connection Timeout"

**What it means:**
```
Client tried to connect but got no response
Service might be down, slow, or unreachable
```

**Troubleshooting:**

```bash
# Step 1: Ping to check basic connectivity
ping server-ip              # If ping works, network path OK

# Step 2: Check if port is open
timeout 5 bash -c 'echo > /dev/tcp/server-ip/8080' 2>&1
# If: Connection refused → Service not listening
# If: Timeout → Network/firewall issue

# Step 3: Traceroute to check path
traceroute server-ip
mtr -c 10 server-ip        # Continuous monitoring

# Step 4: Check service status on server
ssh user@server-ip
systemctl status service-name
ps aux | grep service-name

# Step 5: Check service logs
journalctl -u service-name -n 50  # Last 50 lines
tail -f /var/log/service.log

# Step 6: Check if service is responding
curl -v --connect-timeout 5 http://server-ip:8080
timeout 5 curl -v http://server-ip:8080

# Step 7: Check network latency
ping -c 10 server-ip | grep "avg"
```

**Solutions:**
```bash
# Restart service
systemctl restart service-name

# Check if service crashed (out of memory, CPU limit)
top
free -h
df -h

# Increase timeout in client
curl --connect-timeout 30 http://server-ip:8080

# Check firewall rules
sudo firewall-cmd --list-all
sudo ufw status

# Check routing
ip route show
route -n
```

#### 3. "Connection Reset" / "Connection Aborted"

**What it means:**
```
Connection was established but then closed abruptly
Server rejected or application crashed
```

**Troubleshooting:**

```bash
# Step 1: Check service logs
journalctl -u service-name -n 100 | grep -i error

# Step 2: Check if service crashes
systemctl status service-name
systemctl is-active service-name

# Step 3: Check system resources
top -bn1 | head -20             # CPU/Memory
free -h                          # RAM availability
df -h                            # Disk space

# Step 4: Check TCP backlog
ss -tln | grep LISTEN
cat /proc/sys/net/ipv4/tcp_max_syn_backlog

# Step 5: Monitor in real-time
watch -n 1 'ss -ant | grep ESTABLISHED | wc -l'

# Step 6: Check for port exhaustion
ss -ant | grep -c ESTABLISHED    # Number of connections
ss -ant | grep -c TIME_WAIT      # Number waiting to close
```

**Solutions:**
```bash
# Check application logs
docker logs container-name
kubectl logs pod-name

# Increase limits if hitting ceiling
ulimit -n                        # Current limit
ulimit -n 65536                 # Increase to 65536

# Lower TCP_FIN_TIMEOUT if too many TIME_WAIT
sudo sysctl -w net.ipv4.tcp_fin_timeout=30

# Restart service
systemctl restart service-name

# Scale up if too many connections
kubectl scale deployment myapp --replicas=5
docker-compose up -d --scale service=3
```

#### 4. Asymmetric Routing (Request OK, Response Lost)

**What it means:**
```
Request reaches server (outbound path OK)
Response packet takes different route and gets lost
```

**Troubleshooting:**

```bash
# Step 1: Trace both directions
traceroute server-ip        # Client → Server
ssh user@server-ip
traceroute client-ip        # Server → Client

# Step 2: Use tcpdump to see packets
sudo tcpdump -i eth0 'host server-ip' -n
# Look for: Requests arriving but responses missing

# Step 3: Check source IP
hostname -I
ip addr show

# Step 4: Monitor gateway routes
ip route show
route -n
```

**Solutions:**
```bash
# Check routing tables on both sides
ip route show

# Add explicit route if needed
sudo ip route add server-ip/32 via gateway-ip

# Check firewall RETURN traffic rules
sudo iptables -L -n | grep ESTABLISHED
sudo firewall-cmd --query-rich-rule="rule family='ipv4' 
direction='in' state name='RELATED' action='accept'"
```

#### 5. Port Conflicts / Port Already in Use

**What it means:**
```
Trying to start service on port that's already in use
```

**Troubleshooting:**

```bash
# Step 1: Find what's using the port
lsof -i :8080                   # What's on port 8080?
netstat -tlnp | grep :8080
ss -tlnp | grep :8080

# Step 2: Check full details
ps aux | grep PID               # Get more details on process

# Step 3: Check if it's in TIME_WAIT
netstat -an | grep :8080 | grep TIME_WAIT
ss -an | grep :8080 | grep TIME_WAIT
```

**Solutions:**
```bash
# Option 1: Kill the process using the port
kill PID
kill -9 PID                     # Force kill

# Option 2: Use different port
# Edit config and use port 8081
netstat -tlnp | grep :8081     # Make sure 8081 free

# Option 3: Wait for TIME_WAIT to expire
# TIME_WAIT default: 60 seconds on Linux

# Option 4: Enable port reuse
# Add to application (socket option): SO_REUSEADDR
# Or kernel setting:
sudo sysctl -w net.ipv4.tcp_tw_reuse=1
```

---

## Layer 7 (Application) Troubleshooting

### Understanding the Problem

Layer 7 problems involve:
- HTTP status codes
- DNS resolution
- API response times
- Service availability

### Common L7 Problems

#### 1. High Response Time / Slow API

**What it means:**
```
Request reaches server quickly, but processing is slow
Response takes >1 second (depending on SLA)
```

**Troubleshooting:**

```bash
# Step 1: Check response time
curl -w "Total: %{time_total}s\n" http://example.com

# Breakdown:
# time_namelookup: DNS lookup time
# time_connect: TCP connection time
# time_appconnect: TLS handshake time
# time_pretransfer: Server processing time
# time_starttransfer: First byte received time
# time_total: Total time

curl -w "@curl-format.txt" -o /dev/null -s http://example.com

# Step 2: Identify bottleneck
# If time_namelookup high → DNS issue (L3)
# If time_connect high → Network/TCP issue (L4)
# If time_appconnect high → TLS issue (L6)
# If time_starttransfer high → Server slow (L7)

# Step 3: Check server resources
ssh server
top
vmstat 1 5
iostat -x 1 5

# Step 4: Check database performance
mysql -u user -p
SHOW PROCESSLIST;              # Long-running queries
SHOW ENGINE INNODB STATUS;     # Performance issues

# Step 5: Check application logs
journalctl -u app -f
tail -f /var/log/app/error.log

# Step 6: Check external API calls
strace -p PID                  # Trace system calls
# Look for 'connect' syscalls taking time

# Step 7: Load test
ab -n 100 -c 10 http://example.com/
# Check if response time increases with concurrency
```

**Solutions:**
```bash
# Optimize application
# - Profile code (Python: cProfile, Node: clinic.js)
# - Look for N+1 query problems
# - Add database indexes
# - Cache results

# Scale horizontally
kubectl scale deployment app --replicas=5
docker-compose up -d --scale service=3

# Optimize database
# ANALYZE tables
# Create indexes
# Tune query

# Enable caching
# Redis, Memcached, HTTP caching headers
```

#### 2. 404 Not Found / 500 Internal Server Error

**What it means:**
```
404: Resource doesn't exist
500: Server error (crash, exception, config error)
```

**Troubleshooting:**

```bash
# Step 1: Check what the server is saying
curl -v http://example.com/api/users
# Look at: HTTP response code

# Step 2: Check server logs
tail -f /var/log/nginx/error.log      # Web server errors
tail -f /var/log/app/app.log          # Application errors
docker logs container-name

# Step 3: Check configuration
nginx -t                              # Test nginx config
apache2ctl configtest                # Test Apache
systemctl status app-name

# Step 4: Check application health
curl http://localhost:8080/health
curl http://localhost:8080/health/readiness

# Step 5: Check routes are configured
grep -r "api/users" /etc/nginx/
grep -r "api/users" /etc/apache2/

# Step 6: Check database connectivity
mysql -u user -p -h host -e "SELECT 1;"
psql -U user -h host -c "SELECT 1;"
redis-cli ping
```

**Solutions:**
```bash
# For 404:
# - Check URL spelling
# - Check route configuration
# - Verify resource exists

# For 500:
# - Restart service
systemctl restart app-name

# - Check dependencies (database, cache)
# - Check resource limits
top -p PID
# - Check application config

# - Deploy fix if code error
# - Rollback if recent change
```

#### 3. DNS Resolution Issues

**What it means:**
```
Can't resolve domain name to IP address
```

**Troubleshooting:**

```bash
# Step 1: Basic DNS test
nslookup example.com
dig example.com
host example.com

# Step 2: Check with different DNS server
nslookup example.com 8.8.8.8           # Google DNS
dig @1.1.1.1 example.com               # Cloudflare DNS

# Step 3: Check DNS servers configured
cat /etc/resolv.conf

# Step 4: Trace DNS resolution
dig +trace example.com
# Shows: Root NS → TLD NS → Authoritative NS

# Step 5: Check specific record types
dig example.com A                       # IPv4
dig example.com AAAA                    # IPv6
dig example.com MX                      # Mail server
dig example.com NS                      # Name servers
dig example.com CNAME                   # Alias

# Step 6: Check from application perspective
curl -v http://example.com             # Shows DNS lookup time
# Look for: time_namelookup

# Step 7: Flush DNS cache
sudo systemctl restart systemd-resolved # Linux
sudo dscacheutil -flushcache           # macOS
ipconfig /flushdns                      # Windows
```

**Solutions:**
```bash
# Update /etc/resolv.conf
sudo nano /etc/resolv.conf
# Add: nameserver 8.8.8.8
# Add: nameserver 1.1.1.1

# Restart DNS service
sudo systemctl restart systemd-resolved

# Check DNS server status
sudo systemctl status systemd-resolved

# For Kubernetes
kubectl edit configmap coredns -n kube-system

# Wait for DNS propagation (24-48 hours for changes)
```

#### 4. TLS/SSL Certificate Issues

**What it means:**
```
HTTPS connection fails, certificate problem, or expired
```

**Troubleshooting:**

```bash
# Step 1: Check certificate details
openssl s_client -connect example.com:443 -showcerts

# Step 2: Check expiration date
echo | openssl s_client -servername example.com \
  -connect example.com:443 2>/dev/null \
  | openssl x509 -noout -dates

# Step 3: Check certificate chain
openssl s_client -connect example.com:443 -showcerts

# Step 4: Verify certificate validity
openssl verify /path/to/cert.pem

# Step 5: Check certificate matches domain
openssl x509 -in /path/to/cert.pem -noout -text | grep -A1 "Subject:"

# Step 6: Monitor certificate expiration
# In Kubernetes:
kubectl get certificate
kubectl describe certificate my-cert

# Step 7: Test with different TLS version
openssl s_client -connect example.com:443 -tls1_2
openssl s_client -connect example.com:443 -tls1_3
```

**Solutions:**
```bash
# Renew certificate
# Using Let's Encrypt:
certbot renew --force-renewal

# Manual renewal:
sudo certbot certonly --standalone -d example.com

# For Kubernetes (cert-manager):
kubectl apply -f certificate.yaml
kubectl describe certificate my-cert

# Update nginx/apache config
# Restart services
systemctl restart nginx
systemctl restart apache2

# Monitor renewal
# Set calendar reminder for 30 days before expiration
```

#### 5. High HTTP Error Rate (4xx, 5xx)

**What it means:**
```
Many requests returning error status codes
```

**Troubleshooting:**

```bash
# Step 1: Check error rate
curl -I http://example.com                # Check response code

# Step 2: Load test to reproduce
ab -n 1000 -c 50 http://example.com/

# Step 3: Check server logs for patterns
tail -100 /var/log/nginx/access.log
# Look for: Status code patterns, 500 errors

# Step 4: Check specific error
grep "500" /var/log/nginx/error.log
grep "5xx" /var/log/app.log

# Step 5: Check load and resources
top
free -h
disk usage
netstat -an | grep -c ESTABLISHED

# Step 6: Check recent changes
git log --oneline -10
docker images | head

# Step 7: Check dependencies
curl http://database-server:3306
curl http://cache-server:6379
curl http://external-api.com
```

**Solutions:**
```bash
# Restart service
systemctl restart app-name

# Scale up if under load
kubectl scale deployment app --replicas=10

# Rollback recent changes
git revert COMMIT
docker pull image:previous-tag
kubectl set image deployment/app app=image:previous-tag

# Fix root cause
# - Check logs for exception
# - Fix application code
# - Optimize queries
# - Increase resources
```

---

## Common Patterns & Quick Checks

### Quick Diagnosis Matrix

```
Problem Type        L4 Check            L7 Check
──────────────────────────────────────────────────
Connectivity       telnet host:port    curl http://host
Response Time      ping                curl -w @fmt.txt
Error Response     netstat -an         curl -v
Resource Issue     ss -ant             top
DNS Problem        nslookup            dig +trace
Certificate        openssl s_client    curl -v https://
```

### One-Liner Diagnostics

```bash
# Check if service is up and responding
curl -s http://localhost:8080/health | jq .

# Check all listening ports
ss -tlnp

# Count active connections
ss -ant | grep ESTAB | wc -l

# Check for connection leaks
watch -n 2 'ss -ant | grep TIME_WAIT | wc -l'

# Monitor DNS
while true; do dig example.com @8.8.8.8 | grep -i "Query time"; sleep 5; done

# Test endpoint repeatedly
watch -n 1 'curl -s -o /dev/null -w "%{http_code}\n" http://example.com'

# Check TLS certificate expiration
echo | openssl s_client -servername example.com -connect example.com:443 2>/dev/null | openssl x509 -noout -dates
```

---

## Next Steps

- Review [Common Network Commands](Common-Network-Commands.md)
- Study [DevOps/SRE Scenarios](DevOps-SRE-Scenarios.md)
- Practice with sample exercises
