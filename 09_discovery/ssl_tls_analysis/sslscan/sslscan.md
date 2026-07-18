---
title: "sslscan - SSL/TLS Scanner"
description: "Comprehensive SSL/TLS scanner that detects ciphers, protocols, certificates, and known vulnerabilities including Heartbleed, POODLE, ROBOT, BEAST, and CRIME"
mitre_tactic: "discovery"
mitre_tactic_id: "TA0007"
mitre_subcategory: "ssl_tls_analysis"
categories:
  - "ssl"
  - "tls"
  - "network-scanning"
  - "vulnerability-scanning"
tags:
  - "ssl"
  - "tls"
  - "scanner"
  - "ciphers"
  - "certificates"
  - "heartbleed"
  - "poodle"
install_command: "sudo apt install sslscan"
version: "2.0.15"
github_repo: "https://github.com/rbsec/sslscan"
kali_tools_page: "https://www.kali.org/tools/sslscan/"
difficulty_level: "beginner"
requires_root: false
gui_available: false
related_tools:
  - "sslyze"
  - "testssl.sh"
  - "nmap"
  - "openssl"
---

# sslscan - SSL/TLS Scanner

## 1. Introduction

**sslscan** is a comprehensive SSL/TLS scanner designed to test SSL/TLS configurations, enumerate supported ciphers, detect known vulnerabilities, and examine certificates. The tool provides detailed information about the SSL/TLS security posture of servers.

### Key Capabilities

**Protocol Detection**: Identifies all SSL/TLS protocol versions supported by the target, including deprecated versions.

**Cipher Enumeration**: Lists all supported cipher suites with strength ratings and compatibility information.

**Certificate Analysis**: Extracts certificate details including issuer, subject, validity, and chain information.

**Vulnerability Detection**: Tests for known SSL/TLS vulnerabilities including Heartbleed, POODLE, ROBOT, BEAST, and CRIME.

**Multiple Output Formats**: Supports text, XML, JSON, and HTML output for integration with other tools and reporting.

### Use Cases

- **Penetration Testing**: Identify weak SSL/TLS configurations during security assessments.
- **Compliance Auditing**: Verify SSL/TLS configurations meet industry standards (PCI DSS, HIPAA).
- **Security Monitoring**: Track SSL/TLS configuration changes over time.
- **Incident Response**: Investigate SSL/TLS-related security incidents.
- **Certificate Management**: Monitor certificate expiry and chain validity.

### How sslscan Works

sslscan connects to target servers and initiates SSL/TLS handshakes with various protocol versions and cipher suites. It analyzes the server's responses to determine supported configurations and test for known vulnerabilities. The tool also retrieves and analyzes the server's certificate chain.

## 2. Installation

### Standard Installation

```bash
sudo apt update
sudo apt install sslscan
```

### Verify Installation

```bash
sslscan --version
```

### Manual Installation from Source

```bash
git clone https://github.com/rbsec/sslscan.git
cd sslscan
make
sudo cp sslscan /usr/local/bin/
```

### Dependencies

**OpenSSL**: Required for SSL/TLS operations.

```bash
sudo apt install libssl-dev
```

## 3. Core Concepts

### SSL/TLS Protocols

**SSL 2.0**: Deprecated in 2011. Multiple critical vulnerabilities. Never use.

**SSL 3.0**: Deprecated in 2015. Vulnerable to POODLE attack. Never use.

**TLS 1.0**: Deprecated in 2020. Vulnerable to BEAST attack. Not recommended.

**TLS 1.1**: Deprecated in 2021. Still supported by some legacy systems.

**TLS 1.2**: Current standard. Widely supported. Recommended minimum.

**TLS 1.3**: Latest version. Improved security and performance. Preferred.

### Cipher Suites

**Key Exchange**: RSA, DHE, ECDHE - Determines how keys are exchanged.

**Authentication**: RSA, DSS, ECDSA - Verifies server identity.

**Encryption**: AES, DES, 3DES, RC4 - Encrypts data in transit.

**MAC**: SHA256, SHA384, MD5 - Ensures data integrity.

### Certificate Components

**Subject**: Entity the certificate identifies (domain name, organization).

**Issuer**: Certificate Authority that issued the certificate.

**Validity**: Not Before and Not After dates.

**Public Key**: Server's public key for key exchange.

**Signature**: CA's digital signature verifying the certificate.

### Known Vulnerabilities

**Heartbleed (CVE-2014-0160)**: Memory disclosure vulnerability in OpenSSL.

**POODLE (CVE-2014-3566)**: Padding oracle attack on SSL 3.0.

**ROBOT (Return Of Bleichenbacher's Oracle Threat)**: RSA padding oracle attack.

**BEAST (CVE-2011-3389)**: CBC cipher attack on TLS 1.0.

**CRIME (CVE-2012-4929)**: Compression-based information leakage.

## 4. Command Reference

### Basic Syntax

```bash
sslscan [OPTIONS] TARGET[:PORT]
```

### Protocol Flags

**--ssl-2**: Test for SSL 2.0 support.

**--ssl-3**: Test for SSL 3.0 support.

**--tls10**: Test for TLS 1.0 support.

**--tls11**: Test for TLS 1.1 support.

**--tls12**: Test for TLS 1.2 support.

**--tls13**: Test for TLS 1.3 support.

**--no-fallback**: Don't test fallback from TLS 1.3 to 1.2.

### Vulnerability Flags

**--heartbleed**: Test for Heartbleed vulnerability.

**--poodle**: Test for POODLE vulnerability.

**--robot**: Test for ROBOT vulnerability.

**--beast**: Test for BEAST vulnerability.

**--crime**: Test for CRIME vulnerability.

**--bugs**: Enable all bug tests (heartbleed, poodle, robot, beast, crime).

### Certificate Flags

**--show-certificate**: Display full certificate details.

**--no-colour**: Disable colored output.

**--ocsp**: Request OCSP stapling status.

**--ocsp-stapling-v2**: Use OCSP stapling v2.

### Output Flags

**--xml FILE**: Output results in XML format.

**--json FILE**: Output results in JSON format.

**--html FILE**: Output results in HTML format.

**--targets FILE**: Read targets from file.

### STARTTLS Flags

**--starttls-ftp**: Use STARTTLS for FTP.

**--starttls-imap**: Use STARTTLS for IMAP.

**--starttls-pop3**: Use STARTTLS for POP3.

**--starttls-smtp**: Use STARTTLS for SMTP.

**--starttls-xmpp**: Use STARTTLS for XMPP.

### Advanced Flags

**--no-colour**: Disable colored output.

**--show-certificate**: Display certificate details.

**--verbose**: Verbose output.

## 5. Examples

### Basic Scan

```bash
sslscan 192.168.1.100:443
```

**Result**: Scans target for SSL/TLS configuration and vulnerabilities.

### Custom Port

```bash
sslscan 192.168.1.100:8443
```

**Result**: Scans non-standard HTTPS port.

### Test Specific Protocols

```bash
sslscan --tls12 --tls13 192.168.1.100:443
```

**Result**: Tests only TLS 1.2 and 1.3 support.

### Test All Vulnerabilities

```bash
sslscan --bugs 192.168.1.100:443
```

**Result**: Tests for Heartbleed, POODLE, ROBOT, BEAST, and CRIME.

### Certificate Details

```bash
sslscan --show-certificate 192.168.1.100:443
```

**Result**: Displays full certificate information.

### XML Output

```bash
sslscan --xml results.xml 192.168.1.100:443
```

**Result**: Saves results in XML format for parsing.

### STARTTLS (SMTP)

```bash
sslscan --starttls-smtp 192.168.1.100:25
```

**Result**: Scans SMTP server with STARTTLS upgrade.

### Target List

```bash
sslscan --targets targets.txt
```

**Result**: Scans all targets listed in file.

### JSON Output

```bash
sslscan --json results.json 192.168.1.100:443
```

**Result**: Saves results in JSON format.

### No Color Output

```bash
sslscan --no-colour 192.168.1.100:443
```

**Result**: Plain text output without color codes.

## 6. Advanced Techniques

### Comprehensive Scan

```bash
sslscan --bugs --show-certificate --ocsp 192.168.1.100:443
```

**Result**: Full vulnerability testing with certificate and OCSP details.

### Network-Wide Scan

```bash
for i in $(seq 1 100); do
  echo "=== Scanning 192.168.1.$i ==="
  sslscan 192.168.1.$i:443 > "ssl_192.168.1.$i.txt" 2>/dev/null
done
```

### Automation Script

```bash
#!/bin/bash
TARGETS="targets.txt"
OUTPUT_DIR="ssl_results_$(date +%Y%m%d_%H%M%S)"
mkdir -p "$OUTPUT_DIR"

while IFS= read -r target; do
  sslscan --json "$OUTPUT_DIR/$target.json" "$target" 2>/dev/null
done < "$TARGETS"
```

### Combining with Other Tools

```bash
# Find HTTPS hosts with nmap, then scan with sslscan
nmap -p 443 192.168.1.0/24 -oG - | grep "443/open" | awk '{print $2}' | while read host; do
  sslscan "$host:443"
done
```

### Certificate Monitoring

```bash
sslscan --show-certificate 192.168.1.100:443 | grep -E "(Not Before|Not After|Subject:)"
```

### Protocol Downgrade Testing

```bash
# Test if server allows downgrade to weak protocols
sslscan --ssl-3 --tls10 --tls11 192.168.1.100:443
```

## 7. Detection & Defense

### Network Detection

**SSL/TLS Handshake Monitoring**: Detect unusual SSL/TLS handshake patterns.

**Cipher Enumeration Alerts**: Monitor for scanning of cipher suites.

**Connection Frequency**: Track hosts performing repeated SSL/TLS connections.

### System Detection

**SSL/TLS Logs**: Review server logs for unusual connection patterns.

**Certificate Transparency**: Monitor Certificate Transparency logs for unauthorized certificates.

### Defense Strategies

**Disable Weak Protocols**: Disable SSL 2.0, SSL 3.0, TLS 1.0, and TLS 1.1.

**Remove Weak Ciphers**: Disable RC4, DES, 3DES, and export cipher suites.

**Enable TLS 1.3**: Prefer TLS 1.3 for improved security and performance.

**Certificate Management**: Use valid certificates from trusted CAs.

**HSTS Implementation**: Enable HTTP Strict Transport Security.

**OCSP Stapling**: Enable OCSP stapling for faster certificate validation.

## 8. Troubleshooting

### Connection Refused

**Port Verification**: Confirm target port is open.

```bash
nmap -p 443 192.168.1.100
```

**Firewall Issues**: Check for firewall blocking SSL/TLS connections.

### No Protocols Detected

**Server Configuration**: Target may not support SSL/TLS.

**Network Issues**: Verify network connectivity.

```bash
openssl s_client -connect 192.168.1.100:443
```

### Partial Results

**Protocol Filtering**: Some servers may reject certain protocol tests.

**Load Balancers**: Multiple servers behind load balancer may have different configs.

### Timeout Issues

**Network Latency**: Increase timeout with --timeout flag.

**Server Load**: Target server may be overloaded.

### Certificate Errors

**Self-Signed Certificates**: Use --show-certificate to view details.

**Expired Certificates**: Check certificate validity dates.

**Chain Issues**: Verify certificate chain completeness.

## Resources

- **sslscan GitHub**: https://github.com/rbsec/sslscan
- **SSL/TLS Best Practices**: https://wiki.mozilla.org/Security/Server_Side_TLS
- **Mozilla SSL Configuration Generator**: https://ssl-config.mozilla.org/
- **Kali Tools Documentation**: https://www.kali.org/tools/sslscan/

## Related Tools

**sslyze**: Python-based SSL/TLS analyzer with JSON output and comprehensive testing.

**testssl.sh**: Comprehensive SSL/TLS testing script with extensive vulnerability checks.

**nmap**: Network scanner with SSL/TLS enumeration scripts.

**openssl**: Swiss army knife for SSL/TLS operations and manual testing.
