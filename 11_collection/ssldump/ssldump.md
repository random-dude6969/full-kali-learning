---
mitre_tactic: collection
mitre_tactic_id: TA0009
tool_name: ssldump
tool_category: SSL/TLS Protocol Analyzer
github: https://github.com/adulau/ssldump
version: 1.9
license: BSD
language: C
---

# ssldump - SSL/TLS Network Protocol Analyzer

## Chapter 1: Overview

ssldump is a network protocol analyzer that decrypts SSL/TLS traffic when provided with the appropriate keying material. Based on tcpdump, it captures and decodes SSLv3/TLS network traffic, displaying the application data in human-readable format.

**Core Function:** Decode and analyze SSL/TLS encrypted network traffic

**Primary Use Cases:**
- Network forensics and traffic analysis
- SSL/TLS protocol debugging
- Encrypted traffic inspection
- Credential capture from decrypted sessions
- Application behavior analysis over encrypted connections

**Key Capabilities:**
- SSL/TLS protocol decoding
- Decryption with RSA private keys
- Session key logging via SSLKEYLOGFILE
- PCAP file analysis
- Real-time network capture
- Application data extraction

**Protocol Support:**
- SSL 3.0
- TLS 1.0, 1.1, 1.2
- HTTP over SSL
- Generic TCP over SSL

---

## Chapter 2: Installation

### Kali Linux (Default)

```bash
sudo apt install ssldump
```

### From Source

```bash
git clone https://github.com/adulau/ssldump.git
cd ssldump
./configure
make
sudo make install
```

### Dependencies

- libpcap (packet capture)
- OpenSSL (TLS decryption)
- libnet (packet construction)

### Verify Installation

```bash
ssldump --help
```

---

## Chapter 3: Architecture and Operation

### Decryption Methods

```
┌─────────────────────────────────────────────────────────────────┐
│                   ssldump Decryption Methods                     │
├─────────────────────────────────────────────────────────────────┤
│  RSA Private Key    │ Decrypt sessions using server's RSA key   │
│  SSLKEYLOGFILE      │ Use pre-master secret log file            │
│  Forward Secrecy    │ Limited (requires key exchange capture)   │
└─────────────────────────────────────────────────────────────────┘
```

### MITRE ATT&CK Mapping

| Technique | ID | Description |
|-----------|-----|-------------|
| Network Sniffing | T1040 | Capture network traffic |
| Man-in-the-Middle | T1557 | Intercept communications |
| Decrypt Application Data | T1573.002 | Decrypt encrypted data |

### SSL/TLS Handshake Analysis

```
1. ClientHello     → Cipher suites, version, random
2. ServerHello     → Selected cipher, certificate
3. Certificate     → Server identity verification
4. Key Exchange    → Pre-master secret exchange
5. Finished        → Handshake verification
6. Application     → Encrypted data transfer
```

---

## Chapter 4: Command Reference

### Basic Capture

```bash
ssldump -i eth0
```

**Explanation:** Capture all SSL/TLS traffic on interface eth0.

### Filter by Host

```bash
ssldump -i eth0 -h target.local
```

**Explanation:** Only capture traffic to/from target.local.

### Filter by Port

```bash
ssldump -i eth0 -p 443
```

**Explanation:** Only capture traffic on port 443.

### Decryption with RSA Key

```bash
ssldump -i eth0 -k server.key -p 443
```

**Explanation:** Decrypt SSL/TLS traffic using the server's RSA private key.

### Read from PCAP File

```bash
ssldump -r captured.pcap -k server.key
```

**Explanation:** Analyze previously captured traffic from PCAP file.

### Verbose Output

```bash
ssldump -i eth0 -v -p 443
```

**Explanation:** Show detailed protocol information.

### Show Application Data

```bash
ssldump -i eth0 -A -p 443
```

**Explanation:** Print application data in ASCII format.

### Hex Dump Mode

```bash
ssldump -i eth0 -X -p 443
```

**Explanation:** Print application data in hexadecimal format.

### Print All Packets

```bash
ssldump -i eth0 -a -p 443
```

**Explanation:** Print all packets including TCP handshake.

### Quiet Mode

```bash
ssldump -i eth0 -q -p 443
```

**Explanation:** Minimal output, only decrypted application data.

### Time Display

```bash
ssldump -i eth0 -t -p 443
```

**Explanation:** Print timestamp for each packet.

### Output to File

```bash
ssldump -i eth0 -w output.pcap -k server.key
```

**Explanation:** Write decrypted traffic to new PCAP file.

### Network Interface

```bash
ssldump -i eth0 -l -k server.key
```

**Explanation:** Log SSL key material to file.

### Multiple Filters

```bash
ssldump -i eth0 -h target.local -p 443 -k server.key -v -A
```

**Explanation:** Combine multiple filters for targeted analysis.

---

## Chapter 5: Advanced Attack Scenarios

### Scenario 1: Web Server Traffic Analysis

**Setup:**

```bash
# Capture HTTPS traffic
ssldump -i eth0 -p 443 -k /etc/ssl/private/server.key -A 2>&1 | tee web_traffic.txt
```

**Explanation:** Decrypt and analyze all HTTPS traffic to the web server.

### Scenario 2: Credential Extraction

**Setup:**

```bash
# Capture and analyze authentication traffic
ssldump -i eth0 -p 443 -k server.key -A | grep -i "authorization\|password\|token\|cookie"
```

**Explanation:** Extract authentication credentials from decrypted HTTP headers.

### Scenario 3: API Analysis

**Setup:**

```bash
# Capture API traffic
ssldump -i eth0 -h api.target.local -p 443 -k server.key -v -A 2>&1 | tee api_analysis.txt
```

**Explanation:** Analyze API request/response patterns over encrypted connections.

### Scenario 4: PCAP Forensics

**Setup:**

```bash
# Analyze captured PCAP
sslsplit -r evidence.pcap -k suspect_server.key -A 2>&1 | tee forensic_analysis.txt

# Extract specific connections
ssldump -r evidence.pcap -k server.key -h target.local -A > extracted.txt
```

**Explanation:** Perform forensic analysis on previously captured encrypted traffic.

---

## Chapter 6: Detection and Evasion

### Detection Methods

- **Encrypted Traffic Analysis:** Monitor for SSL/TLS without decryption
- **Certificate Transparency:** Track certificate issuances
- **Network Monitoring:** Detect capture attempts on network interfaces
- **Host-Based:** Monitor for packet capture processes

### Evasion Techniques

1. **Passive Capture:** Use passive mode to avoid detection
2. **Interface Selection:** Capture onSPAN/mirror ports
3. **PCAP Analysis:** Analyze offline captures
4. **Limited Scope:** Filter to specific hosts/ports

### Stealth Configuration

```bash
# Passive capture onSPAN port
ssldump -i eth1 -k server.key -q -A

# Minimal output
ssldump -i eth0 -k server.key -q
```

---

## Chapter 7: Integration with Other Tools

### Wireshark Integration

```bash
# Export to PCAP for Wireshark
ssldump -i eth0 -k server.key -w decrypted.pcap

# Open in Wireshark
wireshark decrypted.pcap
```

### tcpdump Integration

```bash
# Capture with tcpdump, analyze with ssldump
tcpdump -i eth0 -w ssl_capture.pcap port 443
ssldump -r ssl_capture.pcap -k server.key
```

### SSLKEYLOGFILE Integration

```bash
# Capture pre-master secrets from browser
# Set environment variable: SSLKEYLOGFILE=/path/to/keylog.txt

# Analyze with ssldump
ssldump -i eth0 -l keylog.txt
```

### Metasploit Integration

```bash
# Capture traffic during exploitation
ssldump -i eth0 -k server.key -w exploitation.pcap
```

---

## Chapter 8: Troubleshooting

### Common Issues

**Issue:** No decryption occurring

```bash
# Verify private key matches server certificate
openssl x509 -noout -modulus -in server.crt | openssl md5
openssl rsa -noout -modulus -in server.key | openssl md5

# Check key is not encrypted
openssl rsa -in server.key -check
```

**Issue:** No traffic captured

```bash
# Verify interface has traffic
tcpdump -i eth0 -c 10

# Check interface is up
ip link show eth0

# Use correct interface
ssldump -i any  # Capture on all interfaces
```

**Issue:** Forward secrecy not decrypting

```bash
# Note: Forward secrecy (DHE/ECDHE) requires key exchange capture
# Use SSLKEYLOGFILE method instead
# Set in browser: SSLKEYLOGFILE=/tmp/keylog.txt
ssldump -i eth0 -l /tmp/keylog.txt
```

**Issue:** Partial decryption

```bash
# Some sessions may use different keys
# Try with verbose output to identify
ssldump -i eth0 -k server.key -v

# Check if multiple certificates are used
ssldump -i eth0 -k server.key -v | grep "Certificate"
```

### Debug Mode

```bash
ssldump -i eth0 -k server.key -vvv 2>&1 | tee debug.log
```

### Performance Tuning

```bash
# Reduce capture scope
ssldump -i eth0 -h target.local -p 443

# Use BPF filters
ssldump -i eth0 -f "host target.local and port 443"
```

---

## Resources

- **GitHub:** https://github.com/adulau/ssldump
- **Kali Documentation:** https://www.kali.org/tools/ssldump/
- **SSL/TLS Protocol:** https://tools.ietf.org/html/rfc8446
- **Wireshark:** https://www.wireshark.org/
- **tcpdump:** https://www.tcpdump.org/
