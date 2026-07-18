---
title: "sslyze - SSL/TLS Configuration Analyzer"
description: "Python-based SSL/TLS configuration analyzer with JSON output, Heartbleed/ROBOT/compression/OCSP checks, and comprehensive protocol testing"
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
  - "analyzer"
  - "python"
  - "json"
  - "heartbleed"
  - "robot"
install_command: "sudo apt install sslyze"
version: "5.0.6"
github_repo: "https://github.com/nabla-c0d3/sslyze"
kali_tools_page: "https://www.kali.org/tools/sslyze/"
difficulty_level: "intermediate"
requires_root: false
gui_available: false
related_tools:
  - "sslscan"
  - "testssl.sh"
  - "nmap"
  - "openssl"
---

# sslyze - SSL/TLS Configuration Analyzer

## 1. Introduction

**sslyze** is a comprehensive SSL/TLS configuration analyzer written in Python. The tool provides deep analysis of SSL/TLS configurations, including protocol support, cipher suites, certificate chains, and known vulnerabilities. Its JSON output makes it ideal for automation and integration with other security tools.

### Key Capabilities

**JSON Output**: Machine-readable JSON output for easy parsing and integration with security pipelines.

**Comprehensive Testing**: Tests for Heartbleed, ROBOT, compression, OCSP stapling, and protocol support.

**Certificate Chain Analysis**: Detailed certificate chain verification and analysis.

**Cipher Suite Enumeration**: Complete listing of supported cipher suites with strength ratings.

**Scriptable Interface**: Python API for programmatic SSL/TLS analysis.

### Use Cases

- **Automated Security Scanning**: Integrate SSL/TLS testing into CI/CD pipelines.
- **Compliance Auditing**: Verify SSL/TLS configurations meet PCI DSS, HIPAA, and other standards.
- **Certificate Monitoring**: Track certificate expiry, chain validity, and OCSP status.
- **Vulnerability Assessment**: Identify weak configurations and known vulnerabilities.
- **Configuration Management**: Compare SSL/TLS configurations across multiple servers.

### How sslyze Works

sslyze connects to target servers and performs a series of SSL/TLS handshakes to determine supported configurations. It analyzes protocol versions, cipher suites, certificate chains, and tests for specific vulnerabilities. Results are presented in structured format, with JSON output available for automation.

## 2. Installation

### Standard Installation

```bash
sudo apt update
sudo apt install sslyze
```

### Verify Installation

```bash
sslyze --version
```

### Manual Installation (pip)

```bash
pip install sslyze
```

### Installation Notes

sslyze requires Python 3.6+ and the following dependencies:

```bash
pip install cryptography pyOpenSSL nassl
```

## 3. Core Concepts

### SSL/TLS Protocol Analysis

**Protocol Support Detection**: sslyze determines which SSL/TLS protocol versions a server supports by attempting handshakes with each version.

**Cipher Suite Analysis**: The tool enumerates all supported cipher suites, categorizing them by key exchange, authentication, encryption, and MAC algorithms.

**Certificate Chain Validation**: sslyze verifies the certificate chain against trusted root CAs and checks for common issues.

### Vulnerability Detection

**Heartbleed (CVE-2014-0160)**: Tests for OpenSSL memory disclosure vulnerability.

**ROBOT**: Tests for RSA padding oracle vulnerability.

**Compression**: Tests for CRIME vulnerability via TLS compression.

**OCSP Stapling**: Checks if server supports OCSP stapling for certificate validation.

### JSON Output Structure

sslyze JSON output contains structured data organized by scan type:

**protocols_support**: Supported protocol versions.

**cipher_suites**: List of supported cipher suites.

**certificate**: Certificate chain information.

**heartbleed**: Heartbleed vulnerability status.

**robot**: ROBOT vulnerability status.

## 4. Command Reference

### Basic Syntax

```bash
sslyze [OPTIONS] TARGET[:PORT]
```

### Core Flags

**--regular**: Perform regular scan (protocols, ciphers, certificate, heartbleed, compression).

**--json FILE**: Output results in JSON format.

**--file FILE**: Read targets from file.

**--quiet**: Suppress output, only errors displayed.

**--timeout NUM**: Connection timeout in seconds (default: 5).

### Protocol Flags

**--ssl2**: Test for SSL 2.0 support.

**--ssl3**: Test for SSL 3.0 support.

**--tls10**: Test for TLS 1.0 support.

**--tls11**: Test for TLS 1.1 support.

**--tls12**: Test for TLS 1.2 support.

**--tls13**: Test for TLS 1.3 support.

### Vulnerability Flags

**--heartbleed**: Test for Heartbleed vulnerability.

**--robot**: Test for ROBOT vulnerability.

**--compression**: Test for TLS compression (CRIME).

**--fallback**: Test for TLS fallback downgrade.

### Certificate Flags

**--cert-info**: Display certificate information.

**--ocsp-stapling**: Check OCSP stapling status.

**--chain**: Display certificate chain.

**--pinning**: Check for HPKP (HTTP Public Key Pinning).

### STARTTLS Flags

**--starttls FTP/IMAP/POP3/SMTP/XMPP/LDAP/RDP**: Use STARTTLS for specified protocol.

**--sni HOSTNAME**: Set Server Name Indication.

### Advanced Flags

**--http_get**: Send HTTP GET request after TLS handshake.

**--xml FILE**: Output results in XML format.

**--targets-file FILE**: Read multiple targets from file.

## 5. Examples

### Basic Scan

```bash
sslyze 192.168.1.100:443
```

**Result**: Performs regular SSL/TLS analysis on target.

### JSON Output

```bash
sslyze --json results.json 192.168.1.100:443
```

**Result**: Saves structured JSON results for parsing.

### Certificate Analysis

```bash
sslyze --cert-info 192.168.1.100:443
```

**Result**: Displays detailed certificate information.

### Vulnerability Testing

```bash
sslyze --heartbleed --robot 192.168.1.100:443
```

**Result**: Tests for specific vulnerabilities.

### Protocol Testing

```bash
sslyze --tls12 --tls13 192.168.1.100:443
```

**Result**: Tests TLS 1.2 and 1.3 support.

### STARTTLS (SMTP)

```bash
sslyze --starttls smtp 192.168.1.100:25
```

**Result**: Scans SMTP server with STARTTLS upgrade.

### Target List

```bash
sslyze --file targets.txt
```

**Result**: Scans all targets listed in file.

### Quiet Mode with JSON

```bash
sslyze --quiet --json results.json 192.168.1.100:443
```

**Result**: Minimal output, JSON results saved.

### Certificate Chain

```bash
sslyze --cert-info --chain 192.168.1.100:443
```

**Result**: Displays full certificate chain details.

### OCSP Stapling Check

```bash
sslyze --ocsp-stapling 192.168.1.100:443
```

**Result**: Checks OCSP stapling support.

## 6. Advanced Techniques

### Comprehensive Analysis

```bash
sslyze --regular --cert-info --ocsp-stapling --json full_analysis.json 192.168.1.100:443
```

**Result**: Complete SSL/TLS analysis with certificate and OCSP details.

### Network-Wide Scanning

```bash
for i in $(seq 1 254); do
  sslyze --json "ssl_192.168.1.$i.json" 192.168.1.$i:443 2>/dev/null
done
```

### Python API Usage

```python
from sslyze import Scanner, ServerScanRequest, ScanCommand

scanner = Scanner()
server_scan = ServerScanRequest(
    server_location=ServerLocation(hostname="192.168.1.100", port=443),
    scan_commands={ScanCommand.REGULAR}
)
scanner.queue_scan(server_scan)
for result in scanner.get_results():
    print(result.scan_result)
```

### Comparing Configurations

```bash
sslyze --json server1.json server1:443
sslyze --json server2.json server2:443
diff <(jq -S . server1.json) <(jq -S . server2.json)
```

### Automated Reporting

```bash
#!/bin/bash
sslyze --json results.json "$1:443"
python3 << 'EOF'
import json
with open("results.json") as f:
    data = json.load(f)
    print(f"Server: {data['server']}")
    print(f"Protocols: {data['scan_result']['protocols_support']}")
EOF
```

### Certificate Expiry Monitoring

```bash
sslyze --cert-info --json results.json 192.168.1.100:443 | jq '.scan_result.certificate.as_json.not_after'
```

## 7. Detection & Defense

### Network Detection

**SSL/TLS Handshake Monitoring**: Detect unusual SSL/TLS handshake patterns.

**Scanning Detection**: Monitor for rapid SSL/TLS connections indicating scanning activity.

**Cipher Enumeration Alerts**: Track hosts requesting multiple cipher suites.

### System Detection

**SSL/TLS Logs**: Review server logs for unusual connection patterns.

**Certificate Transparency**: Monitor Certificate Transparency logs.

### Defense Strategies

**Disable Weak Protocols**: Remove SSL 2.0, SSL 3.0, TLS 1.0, and TLS 1.1 support.

**Remove Weak Ciphers**: Disable RC4, DES, 3DES, and export cipher suites.

**Enable TLS 1.3**: Prefer TLS 1.3 for improved security and performance.

**Certificate Management**: Use valid certificates from trusted CAs.

**HSTS Implementation**: Enable HTTP Strict Transport Security.

**OCSP Stapling**: Enable OCSP stapling for faster certificate validation.

**Regular Scanning**: Schedule regular SSL/TLS configuration audits.

## 8. Troubleshooting

### Connection Refused

**Port Verification**: Confirm target port is open.

```bash
nmap -p 443 192.168.1.100
```

**Firewall Issues**: Check for firewall blocking SSL/TLS connections.

### No Results

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

### JSON Parse Errors

**Corrupted Output**: Re-run scan with --quiet flag.

**Version Compatibility**: Ensure sslyze version matches Python version.

### Certificate Errors

**Self-Signed Certificates**: Normal for internal services.

**Expired Certificates**: Check certificate validity dates.

**Chain Issues**: Verify certificate chain completeness.

## Resources

- **sslyze GitHub**: https://github.com/nabla-c0d3/sslyze
- **SSL/TLS Best Practices**: https://wiki.mozilla.org/Security/Server_Side_TLS
- **Mozilla SSL Configuration Generator**: https://ssl-config.mozilla.org/
- **Kali Tools Documentation**: https://www.kali.org/tools/sslyze/

## Related Tools

**sslscan**: Command-line SSL/TLS scanner with colored output and vulnerability testing.

**testssl.sh**: Comprehensive SSL/TLS testing script with extensive checks.

**nmap**: Network scanner with SSL/TLS enumeration scripts.

**openssl**: Swiss army knife for SSL/TLS operations and manual testing.
