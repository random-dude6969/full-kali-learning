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

## Chapter 9: Configuration File

### sslsplit.conf Format

sslsplit supports configuration files to avoid long command lines:

```
# /etc/sslsplit/sslsplit.conf

# SSL/TLS keys and certificates
ca /etc/sslsplit/ca.pem
cert /etc/sslsplit/ca.key

# Logging
l /var/log/sslsplit/connections.log
L /var/log/sslsplit/content.log
S /var/log/sslsplit/content/
M /var/log/sslsplit/masterkeys.log

# PCAP output
X /var/log/sslsplit/pcap/

# Security options
O           # Deny OCSP requests
P           # Pass-through unsplittable connections
j /tmp/sslsplit/  # Chroot jail

# Drop privileges
u nobody
g nogroup

# Proxy specifications
# syntax: type listen_addr listen_port [dest_addr dest_port]
https 0.0.0.0 8443
http 0.0.0.0 8080
tcp 0.0.0.0 10025
```

### Using Configuration File

```bash
# Start with configuration file
sslsplit -F /etc/sslsplit/sslsplit.conf

# Override specific options
sslsplit -F /etc/sslsplit/sslsplit.conf -D  # Add debug mode
```

### nftables Rules

```bash
#!/usr/sbin/nft -f
# /etc/nftables.d/sslsplit.conf

# Flush existing nat rules
flush ruleset

table ip sslsplit_nat {
    chain prerouting {
        type nat hook prerouting priority dstnat; policy accept;

        # Redirect HTTP to sslsplit
        iifname "eth0" tcp dport 80 redirect to :8080

        # Redirect HTTPS to sslsplit
        iifname "eth0" tcp dport 443 redirect to :8443

        # Redirect SMTP STARTTLS
        iifname "eth0" tcp dport 25 redirect to :8465
    }

    chain postrouting {
        type nat hook postrouting priority srcnat; policy accept;

        # NAT for internet sharing
        oifname "eth0" masquerade
    }
}
```

### IPv6 Interception

```bash
# Enable IPv6 forwarding
sudo sysctl -w net.ipv6.conf.all.forwarding=1

# Redirect IPv6 HTTPS traffic
sudo ip6tables -t nat -A PREROUTING -i eth0 -p tcp --dport 443 \
  -j REDIRECT --to-ports 8443

# Start sslsplit with IPv6 support
sslsplit -k ca.key -c ca.pem \
  https 0.0.0.0 8443 \
  https [::] 8443 \
  http 0.0.0.0 8080 \
  http [::] 8080
```

### Certificate Generation Best Practices

```bash
# Generate a strong CA key and certificate
# Use ECDSA for better performance
openssl ecparam -genkey -name prime256v1 -out ca.key
openssl req -new -x509 -key ca.key -out ca.pem -days 3650 \
  -subj "/CN=sslsplit CA" \
  -addext "basicConstraints=critical,CA:TRUE" \
  -addext "keyUsage=critical,keyCertSign,cRLSign"

# For RSA (slower but broader compatibility)
openssl genrsa -out ca_rsa.key 4096
openssl req -new -x509 -key ca_rsa.key -out ca_rsa.pem -days 3650 \
  -subj "/CN=sslsplit CA RSA"

# Verify the generated certificate
openssl x509 -in ca.pem -text -noout
```

---

## Chapter 10: Advanced STARTTLS Interception

### SMTP STARTTLS

```bash
# Intercept SMTP with STARTTLS upgrade
sslsplit -D -l /var/log/sslsplit/smtp.log \
  -S /var/log/smtp_content/ \
  -k ca.key -c ca.pem \
  autossl 0.0.0.0 8465 192.168.1.10 25

# Redirect SMTP traffic
sudo iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 25 \
  -j REDIRECT --to-ports 8465
```

### IMAP/POP3 STARTTLS

```bash
# Intercept IMAP STARTTLS
sslsplit -D -l /var/log/sslsplit/imap.log \
  -S /var/log/imap_content/ \
  -k ca.key -c ca.pem \
  autossl 0.0.0.0 8993 192.168.1.10 143

# Intercept POP3 STARTTLS
sslsplit -D -l /var/log/sslsplit/pop3.log \
  -S /var/log/pop3_content/ \
  -k ca.key -c ca.pem \
  autossl 0.0.0.0 8995 192.168.1.10 110

# Redirect mail traffic
sudo iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 143 \
  -j REDIRECT --to-ports 8993
sudo iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 110 \
  -j REDIRECT --to-ports 8995
```

### MySQL/PostgreSQL SSL

```bash
# Intercept MySQL SSL connections
sslsplit -D -l /var/log/sslsplit/mysql.log \
  -k ca.key -c ca.pem \
  ssl 0.0.0.0 3307 192.168.1.10 3306

# Intercept PostgreSQL SSL
sslsplit -D -l /var/log/sslsplit/pg.log \
  -k ca.key -c ca.pem \
  ssl 0.0.0.0 5433 192.168.1.10 5432
```

---

## Chapter 11: Log Management and Analysis

### Log Rotation

```bash
# /etc/logrotate.d/sslsplit
/var/log/sslsplit/*.log {
    daily
    missingok
    rotate 30
    compress
    delaycompress
    notifempty
    create 0640 nobody nogroup
    sharedscripts
    postrotate
        systemctl reload sslsplit > /dev/null 2>&1 || true
    endscript
}
```

### Content Log Analysis

```bash
# Search for credentials in content logs
grep -rl "password\|passwd\|Authorization\|Bearer" /var/log/sslsplit/content/

# Extract HTTP POST data
find /var/log/sslsplit/content/ -type f -exec grep -l "POST" {} \; | \
  xargs grep -A5 "POST"

# Count connections per IP
cat /var/log/sslsplit/connections.log | awk '{print $1}' | sort | uniq -c | sort -rn

# Find largest sessions
ls -lhS /var/log/sslsplit/content/ | head -20
```

### PCAP Analysis

```bash
# Open PCAP output in Wireshark
wireshark /var/log/sslsplit/pcap/

# Analyze with tshark
tshark -r /var/log/sslsplit/pcap/session.pcap -Y "http" -T fields \
  -e http.host -e http.request.uri -e http.request.method

# Extract files from PCAP
tshark -r session.pcap --export-objects "http,/tmp/extracted_files/"
```

### Master Key Logging for Wireshark

```bash
# Start sslsplit with master key logging
sslsplit -M /var/log/sslsplit/masterkeys.log \
  -k ca.key -c ca.pem \
  https 0.0.0.0 8443

# In Wireshark, load the master keys:
# Edit → Preferences → Protocols → TLS
# (Pre)-Master-Secret log filename: /var/log/sslsplit/masterkeys.log
```

---

## Chapter 12: Enterprise Deployment

### High-Throughput Tuning

```bash
# Increase file descriptor limits
ulimit -n 65536
echo "sslsplit soft nofile 65536" >> /etc/security/limits.conf
echo "sslsplit hard nofile 65536" >> /etc/security/limits.conf

# Use multiple instances for load distribution
# Instance 1: HTTP
sslsplit -D -k ca.key -c ca.pem http 0.0.0.0 8080 &
# Instance 2: HTTPS
sslsplit -D -k ca.key -c ca.pem https 0.0.0.0 8443 &

# Optimize kernel network buffers
sudo sysctl -w net.core.rmem_max=16777216
sudo sysctl -w net.core.wmem_max=16777216
sudo sysctl -w net.ipv4.tcp_rmem="4096 87380 16777216"
```

### Systemd Service File

```ini
# /etc/systemd/system/sslsplit.service
[Unit]
Description=SSLsplit Transparent SSL/TLS Interception
After=network.target

[Service]
Type=simple
ExecStart=/usr/bin/sslsplit -F /etc/sslsplit/sslsplit.conf
Restart=always
RestartSec=5
User=nobody
Group=nogroup
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
```

```bash
# Enable and start
sudo systemctl daemon-reload
sudo systemctl enable sslsplit
sudo systemctl start sslsplit
sudo systemctl status sslsplit
```

### Network Tap Hardware

For passive interception on production networks:

```
Recommended hardware taps:
- Network TAP: Garland Technology GTPtap series
- Fiber tap: Keysight/IXIA network brokers
- Copper tap: Dualcomm ETAP series

Deployment considerations:
- Passive taps are undetectable (no electrical signature)
- Support full-duplex traffic at line rate
- No latency addition
- SPAN/mirror port alternative (may drop packets under load)
```

---

## Chapter 13: Comparison with Other Tools

### sslsplit vs mitmproxy

| Feature | sslsplit | mitmproxy |
|---------|----------|-----------|
| Operation Mode | Transparent (NAT) | Explicit proxy |
| Protocol Support | Any TCP/SSL | HTTP/HTTPS focused |
| Configuration | CLI / config file | CLI / scripts |
| Scripting | None | Python addons |
| Performance | High (C) | Moderate (Python) |
| Deployment | Network-level | Application-level |
| Stealth | Very high | Moderate |

### sslsplit vs sslsniff

| Feature | sslsplit | sslsniff |
|---------|----------|----------|
| Architecture | Transparent proxy | Inline proxy |
| Certificate Handling | Dynamic on-the-fly | Dynamic + targeted |
| OCSP Handling | Denial (-O flag) | Denial (-d flag) |
| HPKP Bypass | Automatic | Manual |
| STARTTLS | Yes (autossl) | Limited |
| PCAP Output | Yes | No |
| Master Key Logging | Yes | No |

### sslsplit vs Squid + SSL bump

| Feature | sslsplit | Squid + SSL bump |
|---------|----------|-----------------|
| Complexity | Simple | Complex |
| Filtering | Minimal | Full URL filtering |
| Authentication | None | Built-in |
| Caching | No | Yes |
| Logging | Basic | Full access.log |
| Resource Usage | Low | High |

---

## Resources

- **GitHub:** https://github.com/droe/sslsplit
- **Website:** https://www.roe.ch/SSLsplit
- **Kali Documentation:** https://www.kali.org/tools/sslsplit/
- **Manual Pages:** `man sslsplit` and `man sslsplit.conf`
- **OpenSSL Documentation:** https://www.openssl.org/docs/
- **Wireshark TLS:** https://www.wireshark.org/docs/
- **RFC 5246 (TLS 1.2):** https://tools.ietf.org/html/rfc5246
- **RFC 8446 (TLS 1.3):** https://tools.ietf.org/html/rfc8446
- **nftables Wiki:** https://wiki.nftables.org/
- **Network TAP Guide:** https://www.garlandtechnology.com/resources/blog/what-is-a-network-tap
