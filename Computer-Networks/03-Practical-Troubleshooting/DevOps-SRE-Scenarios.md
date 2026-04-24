# DevOps/SRE Real-World Troubleshooting Scenarios

Real-world scenarios and how to debug them using networking knowledge.

---

## Scenario 1: Kubernetes Pod Cannot Reach Database

**Symptom**: Pod crash with "Connection refused" error

```
Pod logs: FATAL ERROR: Cannot connect to database at db.prod:5432
         Connection refused (111)
```

**Troubleshooting Flowchart**:

```
Pod cannot reach DB
  │
  ├─ L3 (Network): Can pod reach DB server?
  │   └─ kubectl exec -it pod -- ping db.prod
  │       └─ If FAIL → Network policy issue
  │       └─ If OK → Continue
  │
  ├─ L4 (Transport): Is port open?
  │   └─ kubectl exec -it pod -- nc -zv db.prod 5432
  │       └─ If FAIL → Firewall/port issue
  │       └─ If OK → Continue
  │
  ├─ L7 (Application): Is service listening?
  │   └─ From pod: psql -h db.prod -U user
  │       └─ If FAIL → DB service not running
  │       └─ If OK → Check credentials/DB
  │
  └─ Solution depends on where problem found
```

**Step-by-Step Diagnosis**:

```bash
# Step 1: Verify pod is running
kubectl get pods -n default
kubectl describe pod myapp

# Step 2: Check pod logs
kubectl logs myapp
kubectl logs myapp --previous          # If crashed

# Step 3: Test connectivity from pod
kubectl exec -it myapp -- /bin/bash

# Inside pod:
# Step 4: Test DNS (L3)
nslookup db.prod
dig db.prod
host db.prod

# Step 5: Ping database server (L3)
ping -c 3 db.prod                      # ICMP

# Step 6: Test TCP connection (L4)
nc -zv db.prod 5432                    # Port test
telnet db.prod 5432                    # Alternative

# Step 7: Check if DB service is responding (L7)
psql -h db.prod -U myuser -d mydb -c "SELECT 1;"
mysql -h db.prod -u myuser -p -e "SELECT 1;"

# Step 8: Check service endpoints (Kubernetes)
exit                                    # Exit pod shell

# From host:
kubectl get endpoints db
kubectl describe svc db
kubectl get pods -o wide | grep db     # See pod IPs
```

**Possible Solutions**:

| Error | Likely Cause | Solution |
|-------|--------------|----------|
| `Destination Host Unreachable` | Pod in different subnet | Check NetworkPolicy, routing |
| `Connection refused` | Port closed | DB service not running, firewall |
| `Operation timed out` | Firewall dropping packets | Check NetworkPolicy, security group |
| `Unable to resolve host` | DNS issue | Check kube-dns, /etc/resolv.conf |
| `Authentication failed` | Wrong credentials | Check secrets, username/password |

---

## Scenario 2: Microservice API Endpoints Returning 502 Bad Gateway

**Symptom**: Load balancer shows all backend services as down

```
Client: GET /api/users → 502 Bad Gateway
LB Logs: upstream server failed
Service: Actually running, but no one can reach it
```

**Debugging Steps**:

```bash
# Step 1: Verify service is actually running
kubectl get pods -n api
kubectl get svc -n api

# Step 2: Check service endpoints
kubectl get endpoints api-service -n api
# Should show IPs and ports

# Step 3: Check if pods are healthy
kubectl describe pod api-service-xxxxx

# Step 4: Check health check status
kubectl get endpoints api-service -o wide

# Step 5: Inside pod, check service is responding
kubectl exec -it api-service-xxxxx -n api -- /bin/bash

# Inside pod:
curl -v http://localhost:8080/health
# Should return: HTTP 200 OK

# Step 6: Check service port mapping
kubectl describe svc api-service -n api
# Verify: targetPort, port, service IP

# Step 7: From pod, test connectivity to service
curl -v http://api-service.api.svc.cluster.local:8080/health

# Step 8: Check load balancer backend configuration
kubectl describe ingress api-ingress
# Check: backend service names and ports
```

**Common Causes & Solutions**:

```
Cause 1: Health check failing
─────────────────────────────
liveness check says pod is dead
kubectl logs api-service-xxxxx
# See what's failing
Potential: App crashed, memory limit hit, dependency missing

Cause 2: Wrong port in service definition
─────────────────────────────
Service points to wrong port
kubectl get svc api-service -o yaml
# Check: targetPort must match app's listening port
# curl -v http://localhost:8080 should work

Cause 3: Network Policy blocking traffic
─────────────────────────────
kubectl get networkpolicies
kubectl describe networkpolicy api-policy
# Check if policy allows traffic from ingress/LB

Cause 4: Resource limits
─────────────────────────────
kubectl top pod api-service-xxxxx
# Check if pod is OOMKilled or CPU-throttled
kubectl describe pod | grep "Last State"
# Look for: OOMKilled, CrashLoopBackOff

Solution:
1. Check pod logs: kubectl logs pod-name
2. Verify health endpoint: curl localhost:8080/health
3. Ensure port matches: kubectl get svc -o yaml
4. Check NetworkPolicy: kubectl get networkpolicies
5. Monitor resources: kubectl top pod
```

---

## Scenario 3: High Network Latency in Cluster

**Symptom**: APIs that normally respond in 100ms now take 500ms+

```
Client: Request takes 500ms
Expected: <100ms
Issue: Network layer slow?
```

**Investigation Path**:

```bash
# Step 1: Identify which path is slow
kubectl exec -it client-pod -- /bin/bash

# Inside pod:
# Test DNS (Layer 3)
time curl -w "%{time_namelookup}s\n" \
  http://api-service.default.svc.cluster.local -I

# If DNS is slow (>100ms):
nslookup api-service
dig api-service
# Check kube-dns
kubectl logs -n kube-system -l k8s-app=kube-dns

# Test TCP connection (Layer 4)
time nc -zv api-service.default.svc 8080

# Test HTTP (Layer 7)
time curl -w "@curl-format.txt" \
  http://api-service.default.svc/api/endpoint

# Breakdown:
# time_namelookup: DNS (L3)
# time_connect: TCP (L4)
# time_appconnect: TLS (L6)
# time_pretransfer: ~TCP RTT
# time_starttransfer: Server processing
```

**Root Cause Identification**:

```bash
# L1-L3: Network path
# Check for packet loss
ping -c 100 service-ip | tail -1
# If loss > 0% → Packet loss present

# L4: TCP connection
# Check RTT
ss -opt | grep service-ip
# RTT should be <10ms in cluster

# L7: Application
# Check if app itself is slow
kubectl exec -it app-pod -- time curl localhost:8080/health
# Direct connection should be fast

# Check node networking
# Get pod's node
kubectl get pods -o wide | grep pod-name
NODE=$(kubectl get pods pod-name -o jsonpath='{.spec.nodeName}')

# Check node network stats
kubectl debug node/$NODE -it --image=ubuntu

# Inside node container:
ip link show                          # Interface stats
ethtool -S eth0                       # Extended stats
```

**Possible Causes & Fixes**:

```
Cause 1: DNS caching layer
─────────────────────────
kube-dns overloaded with queries
Fix: Check CoreDNS replicas
kubectl scale deployment coredns -n kube-system --replicas=5

Cause 2: Network plugin (CNI) overhead
─────────────────────────
Flannel/Weave/Calico slow
Fix: Check CNI logs, consider MTU tuning

Cause 3: ethtool IRQ handling
─────────────────────────
Network card not handling interrupts efficiently
Fix: Check ethtool settings, adjust RxTx rings

Cause 4: Pod on different node
─────────────────────────
Pod scheduled on far-away node (rack-aware scheduling)
Fix: Use pod affinity, node affinity rules
```

---

## Scenario 4: Docker Container Cannot Reach External URL

**Symptom**: Container cannot reach api.external.com

```
Container: curl https://api.external.com
Error: Could not resolve host: api.external.com
       Or: Connection timed out
```

**Troubleshooting**:

```bash
# Step 1: Check Docker networking
docker network ls
docker inspect container-name | grep Networks

# Step 2: Check DNS inside container
docker exec container-name nslookup api.external.com
docker exec container-name cat /etc/resolv.conf

# Step 3: Test DNS resolution
docker exec container-name ping -c 1 1.1.1.1    # Cloudflare DNS
docker exec container-name ping -c 1 8.8.8.8    # Google DNS

# Step 4: Test routing
docker exec container-name ip route

# Step 5: Test connectivity
docker exec container-name curl -v http://8.8.8.8/

# Step 6: Check if DNS server is reachable from host
docker run --rm busybox nslookup api.external.com 8.8.8.8

# Step 7: Test if it's DNS or connectivity
# Use IP directly
docker exec container-name curl https://1.1.1.1/

# Step 8: Check docker daemon logs
sudo journalctl -u docker -n 50
docker logs container-name
```

**Solutions**:

```bash
# Fix 1: Update container DNS
docker run --dns 8.8.8.8 image-name

# For running container, change docker config:
sudo vim /etc/docker/daemon.json
# Add:
{
  "dns": ["8.8.8.8", "8.8.4.4"]
}
sudo systemctl restart docker

# Fix 2: Use host network
docker run --network host image-name

# Fix 3: Check firewall
sudo iptables -L -n | grep -i forward
# Allow DNS
sudo iptables -A FORWARD -p tcp --dport 53 -j ACCEPT
sudo iptables -A FORWARD -p udp --dport 53 -j ACCEPT
```

---

## Scenario 5: Service-to-Service Communication Failing

**Symptom**: Service A cannot call Service B

```
Service A: curl http://service-b
Error: Connection refused
Both services supposedly running
```

**Diagnosis**:

```bash
# Step 1: Verify both services are running
kubectl get svc service-a service-b
kubectl get pods -l app=service-b

# Step 2: Check service DNS
kubectl exec -it service-a-pod -- nslookup service-b
# Should return service-b's ClusterIP

# Step 3: Check if ClusterIP is correct
kubectl get svc service-b -o jsonpath='{.spec.clusterIP}'

# Step 4: From service-a pod, try connecting
kubectl exec -it service-a-pod -- curl -v http://service-b:8080

# Step 5: Try ClusterIP directly
CLUSTER_IP=$(kubectl get svc service-b -o jsonpath='{.spec.clusterIP}')
kubectl exec -it service-a-pod -- curl -v http://$CLUSTER_IP:8080

# Step 6: Check service endpoints
kubectl get endpoints service-b
# Should show actual pod IPs

# Step 7: Check NetworkPolicy
kubectl get networkpolicies
# Does it allow service-a → service-b traffic?

# Step 8: Check ingress/egress rules
kubectl get networkpolicies -o yaml | grep -A 10 service-b

# Step 9: Check if backend pod is healthy
kubectl logs service-b-pod-name | tail -20

# Step 10: Try port-forward to bypass network
kubectl port-forward svc/service-b 8080:8080
# Then from service-a-pod:
curl http://localhost:8080
```

**Common Issues**:

```
Issue 1: NetworkPolicy blocks traffic
Fix: kubectl edit networkpolicy
    Add ingress rule for service-a

Issue 2: Port mismatch in service definition
Fix: kubectl get svc service-b -o yaml
    Verify port: 8080 matches targetPort

Issue 3: Service selector wrong
Fix: kubectl get svc service-b -o yaml
    Check selector matches pod labels
    kubectl get pods --show-labels

Issue 4: DNS not resolving
Fix: Check kube-dns/CoreDNS
    kubectl get pods -n kube-system -l k8s-app=kube-dns

Issue 5: Pod is CrashLooping
Fix: kubectl logs service-b-pod-name --previous
    Check why pod keeps crashing
```

---

## Scenario 6: Database Connection Pool Exhaustion

**Symptom**: "Too many connections" error, application unresponsive

```
MySQL: ERROR 1040 (HY000): Too many connections
App: Connection pool exhausted
```

**Investigation**:

```bash
# Step 1: Count active connections
netstat -an | grep :3306 | grep -c ESTABLISHED
ss -ant | grep :3306 | grep -c ESTABLISHED

# Step 2: Check connection states
netstat -an | grep :3306 | awk '{print $6}' | sort | uniq -c

# Step 3: Check for TIME_WAIT accumulation
netstat -an | grep :3306 | grep TIME_WAIT | wc -l

# Step 4: Check max_connections in database
# MySQL
mysql -u root -p -e "SHOW VARIABLES LIKE 'max_connections';"
mysql -u root -p -e "SHOW PROCESSLIST;" | grep -c Query

# PostgreSQL
psql -U postgres -c "SHOW max_connections;"
psql -U postgres -c "SELECT datname, count(*) FROM pg_stat_activity GROUP BY datname;"

# Step 5: Check from application perspective
kubectl exec -it app-pod -- netstat -an | grep :3306 | grep -c ESTABLISHED

# Step 6: Monitor new connections
watch -n 1 'ss -ant | grep :3306 | wc -l'

# Step 7: Check if connections are being closed
ss -ant | grep :3306 | grep CLOSE_WAIT | wc -l
# High CLOSE_WAIT = Connections not being returned to pool
```

**Solutions**:

```bash
# Solution 1: Increase database max_connections
# MySQL
mysql -u root -p
SET GLOBAL max_connections = 1000;
# Permanent: add to /etc/mysql/mysql.conf.d/mysqld.cnf

# Solution 2: Kill idle connections
# MySQL
mysql -u root -p
SELECT * FROM information_schema.processlist WHERE time > 300;
KILL CONNECTION id;

# Solution 3: Configure connection pooling
# Application level
# Set min_pool_size, max_pool_size
# Enable connection reuse

# Solution 4: Implement circuit breaker
# Reject new requests if connections nearly exhausted

# Solution 5: Scale database
# Add read replicas
# Use connection pool proxy (PgBouncer for PostgreSQL, ProxySQL for MySQL)
```

---

## Scenario 7: Intermittent Packet Loss / Random Failures

**Symptom**: Occasional request timeouts, not consistent

```
95% of requests succeed
5% get timeout or connection reset
```

**Diagnosis**:

```bash
# Step 1: Monitor packet loss
ping -i 0.2 target-host | grep "loss"
mtr -r -c 100 target-host | tail -5

# Step 2: Look for spikes in latency
iperf3 -c target-host -i 1 -t 60
# Watch for drops in throughput

# Step 3: Check for TX errors
ethtool -S eth0 | grep -i tx_error
ip -s link show

# Step 4: Monitor drops
watch -n 1 'ip -s link show eth0'

# Step 5: Check kernel logs for drops
dmesg | grep -i "dropped"
journalctl -x | tail -50

# Step 6: Check for MTU issues
# If packets > MTU, they're fragmented
ping -M do -s 1472 target-host
# Try different sizes: 1500, 1400, 1300, etc.

# Step 7: Check network interface errors
ifconfig eth0
# Look for: RX errors, TX errors, dropped

# Step 8: Load test to trigger issue
ab -n 10000 -c 100 http://service
# Watch for connection errors
```

**Possible Causes & Fixes**:

```
Cause 1: MTU mismatch
─────────────────────
Packet size > path MTU, gets fragmented
Fix: ip link set dev eth0 mtu 1500
     Check if issue is network path (customer ISP)

Cause 2: Network card firmware issues
─────────────────────
Driver or firmware bug
Fix: Update network driver
     ethtool -S shows errors

Cause 3: Switch/Router buffer overflow
─────────────────────
Too much traffic causes packets to drop
Fix: Monitor switch buffers
     Rate limit traffic
     Add more network capacity

Cause 4: Timeout too short
─────────────────────
RTO (Retransmission Timeout) too low for latency
Fix: sysctl net.ipv4.tcp_retries2
     Increase if latency is variable
```

---

## Quick Decision Tree

**"Things are broken, where do I start?"**

```
┌─────────────────────────────────────────┐
│ Internet access works (ping 8.8.8.8 OK) │
└──────────────────────┬──────────────────┘
                       │
         ┌─────────────┴─────────────┐
         ▼                           ▼
   Can reach DNS?          Can reach Service?
   (dig google.com)        (curl http://service)
   
   NO → Fix DNS            NO → Check:
        Check resolv.conf     - Port open?
        Restart systemd       - Service running?
        Add nameserver        - Firewall rule?
        
   YES ↓                  YES ↓
   
   Services responding?    Response is Error?
   (curl service)          (5xx, 4xx)
   
   NO → Check Service      YES ↓
        App running?         Check Logs
        Port open?           Fix Application
        Connectivity?        
        
   YES ↓
   
   Performance Issue?
   (High latency)
   
   - Check RTT
   - Profile application
   - Check resources
```

---

## Interview Prep Checklist

**Be ready to discuss:**

- ✓ How to diagnose connectivity issues (L3 → L4 → L7)
- ✓ Understanding TCP connection states
- ✓ DNS resolution troubleshooting
- ✓ Port and firewall issues
- ✓ Performance debugging (where latency comes from)
- ✓ Container and Kubernetes networking basics
- ✓ Load balancing and health checks
- ✓ How to read tcpdump/wireshark
- ✓ Common DevOps networking problems
- ✓ Quick diagnosis with one-liners

---

**Practice**: Work through these scenarios with a test application and network simulator.
