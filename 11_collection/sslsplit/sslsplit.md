---
mitre_tactic: collection
mitre_tactic_id: TA0009
tool_name: sslsplit
tool_category: Transparent SSL/TLS Interception
github: https://github.com/droe/sslsplit
version: 0.5.5
license: BSD-2-Clause
language: C
---

# sslsplit - Transparent SSL/TLS Interception

## Chapter 1: Overview

sslsplit is a transparent man-in-the-middle proxy for SSL/TLS encrypted network connections. It terminates incoming SSL/TLS connections and initiates new connections to the original destination, logging all transmitted data. SSLsplit operates via network address translation (NAT) engines.

**Core Function:** Transparent SSL/TLS interception and logging via NAT

**Primary Use Cases:**
- Network forensics and traffic capture
- Penetration testing of encrypted services
- Application security analysis
- Credential extraction from SSL/TLS sessions
- Large-scale encrypted traffic monitoring
- SSL/TLS protocol analysis

**Key Capabilities:**
- Transparent proxy operation via NAT
- Dynamic certificate generation on-the-fly
- SNI-based routing
- STARTTLS support (SMTP, etc.)
- IPv4 and IPv6 support
- Multiple logging formats (text, PCAP, binary)
- OCSP request denial
- HPKP/HSTS bypass
- Certificate Transparency bypass

**Supported Protocols:**
- Plain TCP
- Plain SSL
- HTTP/HTTPS
- SMTP STARTTLS
- Any TLS-capable protocol

---

## Chapter 2: Installation

### Kali Linux (Default)

```bash
sudo apt install sslsplit
```

### From Source

```bash
git clone https://github.com/droe/sslsplit.git
cd sslsplit
make
sudo make install
```

### Dependencies

- OpenSSL (SSL/TLS operations)
- libevent (event-driven networking)
- libpcap (packet capture)
- libnet (packet construction)

### Verify Installation

```bash
sslsplit -V
```

---

## Chapter 3: Architecture and Operation

### Proxy Modes

```
┌─────────────────────────────────────────────────────────────────┐
│                  sslsplit Proxy Specifications                   │
├─────────────────────────────────────────────────────────────────┤
│  http   │ HTTP proxy with optional static destination          │
│  https  │ HTTPS proxy with optional static destination         │
│  tcp    │ Generic TCP proxy (no SSL termination)               │
│  ssl    │ SSL proxy (terminate SSL, forward as TCP)            │
│  autossl│ Auto-detect STARTTLS and upgrade                      │
└─────────────────────────────────────────────────────────────────┘
```

### NAT Engine Support

| OS | NAT Engines |
|----|-------------|
| Linux | netfilter REDIRECT, TPROXY |
| FreeBSD | pf rdr, divert-to, ipfw fwd, ipfilter rdr |
| OpenBSD | pf rdr-to, divert-to |
| macOS | pf rdr, ipfw fwd |

### MITRE ATT&CK Mapping

| Technique | ID | Description |
|-----------|-----|-------------|
| Man-in-the-Middle | T1557 | Intercept communications |
| Network Sniffing | T1040 | Capture network traffic |
| Decrypt Application Data | T1573.002 | Decrypt encrypted data |

### Connection Flow

```
1. Client → NAT → sslsplit (intercepted via iptables)
2. sslsplit: Terminate SSL/TLS connection
3. sslsplit: Generate forged certificate
4. sslsplit → Original Server: New SSL/TLS connection
5. Client ← sslsplit: Forged certificate
6. Client → sslsplit: Encrypted request
7. sslsplit: Decrypt, log, re-encrypt
8. sslsplit → Server: Forward request
9. Server → sslsplit: Response
10. sslsplit: Decrypt, log, re-encrypt
11. sslsplit → Client: Forward response
```

---

## Chapter 4: Command Reference

### Basic Operation

```bash
sslsplit -k ca.key -c ca.pem https 0.0.0.0 8443
```

**Explanation:** Start sslsplit with CA certificate, intercept HTTPS on port 8443.

### Debug Mode

```bash
sslsplit -D -k ca.key -c ca.pem https 0.0.0.0 8443
```

**Explanation:** Run in debug mode with verbose output.

### Connection Logging

```bash
sslsplit -l connections.log -k ca.key -c ca.pem https 0.0.0.0 8443
```

**Explanation:** Log one line per connection to connections.log.

### Content Logging

```bash
# Log to single file
sslsplit -L content.log -k ca.key -c ca.pem https 0.0.0.0 8443

# Log to separate files per connection
sslsplit -S /var/log/sslsplit/ -k ca.key -c ca.pem https 0.0.0.0 8443

# Log with path substitution
sslsplit -F "/var/log/sslsplit/%T-%s-%d.log" -k ca.key -c ca.pem https 0.0.0.0 8443
```

**Explanation:** Log decrypted content in various formats.

### PCAP Logging

```bash
# Single PCAP file
sslsplit -X output.pcap -k ca.key -c ca.pem https 0.0.0.0 8443

# Separate PCAP files per connection
sslsplit -Y /var/log/pcap/ -k ca.key -c ca.pem https 0.0.0.0 8443
```

**Explanation:** Capture decrypted traffic as PCAP files.

### Master Key Logging

```bash
sslsplit -M masterkeys.log -k ca.key -c ca.pem https 0.0.0.0 8443
```

**Explanation:** Log SSL master keys for later decryption in Wireshark.

### Chroot Jail

```bash
sslsplit -j /tmp/sslsplit/ -k ca.key -c ca.pem https 0.0.0.0 8443
```

**Explanation:** Run in chroot jail for security.

### Deny OCSP Requests

```bash
sslsplit -O -k ca.key -c ca.pem https 0.0.0.0 8443
```

**Explanation:** Deny all OCSP requests to prevent certificate revocation checking.

### Pass-Through Mode

```bash
sslsplit -P -k ca.key -c ca.pem https 0.0.0.0 8443
```

**Explanation:** Pass through SSL connections that cannot be split (client cert auth, no matching cert).

### Certificate from Directory

```bash
sslsplit -t /path/to/certs/ -k ca.key -c ca.pem https 0.0.0.0 8443
```

**Explanation:** Use pre-generated certificates from directory for targeted domains.

### SNI-Based Routing

```bash
sslsplit -k ca.key -c ca.pem https 127.0.0.1 8443 sni 443
```

**Explanation:** Route connections based on SNI hostname to original destination.

### Multiple Proxy Specs

```bash
sslsplit -k ca.key -c ca.pem \
  https 0.0.0.0 8443 192.168.1.100 443 \
  http 0.0.0.0 8080 192.168.1.100 80 \
  tcp 0.0.0.0 10025
```

**Explanation:** Run multiple proxy specifications simultaneously.

### Daemon Mode

```bash
sslsplit -d -k ca.key -c ca.pem https 0.0.0.0 8443
```

**Explanation:** Run in daemon mode (background).

### Drop Privileges

```bash
sslsplit -u nobody -g nogroup -k ca.key -c ca.pem https 0.0.0.0 8443
```

**Explanation:** Drop privileges to specified user/group after binding.

---

## Chapter 5: Advanced Attack Scenarios

### Scenario 1: Full Network Interception

**Setup:**

```bash
# Enable IP forwarding
sudo sysctl -w net.ipv4.ip_forward=1

# Configure iptables
sudo iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 80 -j REDIRECT --to-ports 8080
sudo iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 443 -j REDIRECT --to-ports 8443

# Start sslsplit
sslsplit -D -l /var/log/sslsplit/connections.log \
  -S /var/log/sslsplit/content/ \
  -k ca.key -c ca.pem \
  https 0.0.0.0 8443 \
  http 0.0.0.0 8080
```

**Explanation:** Intercept all HTTP/HTTPS traffic on the network.

### Scenario 2: SMTP STARTTLS Interception

**Setup:**

```bash
# Configure sslsplit for STARTTLS
sslsplit -D -l smtp.log -S /var/log/smtp/ \
  -k ca.key -c ca.pem \
  autossl 0.0.0.0 8465 192.168.1.10 25

# Redirect SMTP traffic
sudo iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 25 -j REDIRECT --to-ports 8465
```

**Explanation:** Intercept SMTP traffic with STARTTLS upgrade.

### Scenario 3: Large-Scale Forensics

**Setup:**

```bash
# Generate certificates on-the-fly, log everything
sslsplit -D -l /forensics/connections.log \
  -L /forensics/content.log \
  -X /forensics/pcap/ \
  -M /forensics/masterkeys.log \
  -k ca.key -c ca.pem \
  https 0.0.0.0 8443 \
  http 0.0.0.0 8080

# Use with network tap
sudo iptables -t nat -A PREROUTING -i eth0 -j REDIRECT
```

**Explanation:** Comprehensive logging for forensic analysis.

### Scenario 4: Targeted Domain Interception

**Setup:**

```bash
# Prepare certificates
mkdir -p /etc/sslsplit/certs/
cp target.com.crt /etc/sslsplit/certs/
cp target.com.key /etc/sslsplit/certs/

# Start with targeted certificates
sslsplit -t /etc/sslsplit/certs/ \
  -k ca.key -c ca.pem \
  https 0.0.0.0 8443
```

**Explanation:** Use pre-generated certificates for specific target domains.

---

## Chapter 6: Detection and Evasion

### Detection Methods

- **Certificate Transparency:** Monitor for unexpected certificate issuances
- **OCSP Stapling:** Check for valid OCSP responses
- **Certificate Pinning:** Verify against pinned certificates
- **Network Analysis:** Detect NAT redirection
- **HSTS Preloading:** Check HSTS preload lists

### Evasion Techniques

1. **OCSP Denial:** Use `-O` flag to block revocation checking
2. **HPKP Bypass:** sslsplit mangles HPKP headers automatically
3. **HSTS Bypass:** sslsplit removes HSTS headers
4. **CT Bypass:** sslsplit removes Expect-CT headers

### Stealth Configuration

```bash
# Deny OCSP, pass through when can't split
sslsplit -O -P -k ca.key -c ca.pem https 0.0.0.0 8443

# Use pre-generated certificates
sslsplit -t /etc/sslsplit/certs/ -k ca.key -c ca.pem https 0.0.0.0 8443
```

---

## Chapter 7: Integration with Other Tools

### Wireshark Analysis

```bash
# Log master keys for Wireshark
sslsplit -M masterkeys.log -k ca.key -c ca.pem https 0.0.0.0 8443

# In Wireshark:
# Edit > Preferences > Protocols > TLS > (Pre)-Master-Secret log filename
# Select masterkeys.log
```

### tcpdump Integration

```bash
# Capture before sslsplit
tcpdump -i eth0 -w before_sslsplit.pcap port 443

# Capture after sslsplit (decrypted)
tcpdump -i lo -w after_sslsplit.pcap port 8443
```

### iptables/nftables Integration

```bash
# iptables REDIRECT
sudo iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 443 -j REDIRECT --to-port 8443

# nftables
sudo nft add rule nat prerouting iif eth0 tcp dport 443 redirect to :8443
```

### Forensic Tools Integration

```bash
# Analyze with NetworkMiner
# Export PCAP from sslsplit
# Open in NetworkMiner for credential extraction
```

---

## Chapter 8: Troubleshooting

### Common Issues

**Issue:** No connections being intercepted

```bash
# Verify iptables rules
sudo iptables -t nat -L -n

# Check IP forwarding
sysctl net.ipv4.ip_forward

# Test with direct connection
curl -x http://localhost:8443 https://example.com
```

**Issue:** Certificate errors

```bash
# Verify CA certificate
openssl x509 -in ca.pem -text -noout

# Check certificate matches key
openssl x509 -noout -modulus -in ca.pem | openssl md5
openssl rsa -noout -modulus -in ca.key | openssl md5
```

**Issue:** Connections timing out

```bash
# Check sslsplit is running
ps aux | grep sslsplit

# Check listening ports
netstat -tlnp | grep 8443

# Verify destination is reachable
nc -zv 192.168.1.10 443
```

**Issue:** Partial decryption

```bash
# Some connections use client certificates
# Use -P to pass through unsplittable connections
sslsplit -P -k ca.key -c ca.pem https 0.0.0.0 8443

# Check for forward secrecy (requires master key logging)
sslsplit -M keys.log -k ca.key -c ca.pem https 0.0.0.0 8443
```

### Debug Mode

```bash
sslsplit -D -l /var/log/sslsplit/debug.log -k ca.key -c ca.pem https 0.0.0.0 8443
```

### Performance Tuning

```bash
# Increase file descriptor limits
ulimit -n 65536

# Use multiple threads (default: number of CPU cores)
# sslsplit automatically uses multiple threads

# Reduce logging for high-throughput
sslsplit -l /dev/null -k ca.key -c ca.pem https 0.0.0.0 8443
```

---

## Resources

- **GitHub:** https://github.com/droe/sslsplit
- **Website:** https://www.roe.ch/SSLsplit
- **Kali Documentation:** https://www.kali.org/tools/sslsplit/
- **Manual Pages:** `man sslsplit` and `man sslsplit.conf`
- **OpenSSL Documentation:** https://www.openssl.org/docs/
- **Wireshark TLS:** https://www.wireshark.org/docs/
