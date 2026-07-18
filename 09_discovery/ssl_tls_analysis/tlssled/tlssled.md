---
title: "tlssled - SSL/TLS Security Assessment Wrapper"
description: "Wrapper tool combining sslscan and sslyze for quick SSL/TLS security assessment with consolidated output"
mitre_tactic: "discovery"
mitre_tactic_id: "TA0007"
mitre_subcategory: "ssl_tls_analysis"
categories:
  - "ssl"
  - "tls"
  - "network-scanning"
  - "security-assessment"
tags:
  - "ssl"
  - "tls"
  - "wrapper"
  - "sslscan"
  - "sslyze"
  - "quick-assessment"
install_command: "sudo apt install tlssled"
version: "1.3"
github_repo: ""
kali_tools_page: "https://www.kali.org/tools/tlssled/"
difficulty_level: "beginner"
requires_root: false
gui_available: false
related_tools:
  - "sslscan"
  - "sslyze"
  - "testssl.sh"
  - "nmap"
---

# tlssled - SSL/TLS Security Assessment Wrapper

## 1. Introduction

**tlssled** is a wrapper tool that combines the functionality of **sslscan** and **sslyze** to provide a quick SSL/TLS security assessment. The tool simplifies the process of evaluating SSL/TLS configurations by running both scanners and presenting consolidated results.

### Key Capabilities

**Simplified Interface**: Single command to run both sslscan and sslyze.

**Consolidated Output**: Results from both tools presented together for easy analysis.

**Quick Assessment**: Rapid evaluation of SSL/TLS security posture.

**Comprehensive Coverage**: Leverages sslscan's vulnerability detection and sslyze's detailed analysis.

### Use Cases

- **Quick Security Assessments**: Fast evaluation of SSL/TLS configurations.
- **Penetration Testing**: Initial reconnaissance of SSL/TLS services.
- **Compliance Checks**: Verify SSL/TLS configurations meet standards.
- **Security Auditing**: Identify weak configurations and vulnerabilities.
- **Educational Use**: Learn about SSL/TLS security by examining tool outputs.

### How tlssled Works

tlssled accepts a target hostname or IP address and optional port, then executes both sslscan and sslyze against the target. Results from both tools are displayed in sequence, providing a comprehensive view of the SSL/TLS configuration.

## 2. Installation

### Standard Installation

```bash
sudo apt update
sudo apt install tlssled
```

### Verify Installation

```bash
tlssled --help
```

### Dependencies

tlssled requires sslscan and sslyze to be installed:

```bash
sudo apt install sslscan sslyze
```

## 3. Core Concepts

### SSL/TLS Assessment Components

**Protocol Analysis**: Determine which SSL/TLS protocol versions are supported.

**Cipher Suite Enumeration**: List all supported cipher suites.

**Certificate Analysis**: Examine certificate details and chain.

**Vulnerability Testing**: Check for known SSL/TLS vulnerabilities.

### Tool Integration

**sslscan**: Provides colored output, vulnerability detection, and quick cipher enumeration.

**sslyze**: Offers detailed analysis, JSON output, and comprehensive testing.

**tlssled**: Combines both tools for complete coverage.

### Assessment Output

**Protocol Support**: Shows which SSL/TLS versions are accepted.

**Cipher Suites**: Lists supported ciphers with strength ratings.

**Certificate Details**: Displays certificate information and chain.

**Vulnerability Status**: Reports findings from both tools.

## 4. Command Reference

### Basic Syntax

```bash
tlssled [OPTIONS] HOSTNAME PORT
```

### Core Flags

**-p PORT**: Specify target port (default: 443).

**-v**: Verbose output. Show additional details during scanning.

### Optional Flags

**--no-color**: Disable colored output.

**--quiet**: Suppress informational messages.

**--help**: Display help information.

**--version**: Display version information.

## 5. Examples

### Basic Scan

```bash
tlssled 192.168.1.100 443
```

**Result**: Runs both sslscan and sslyze against target.

### Custom Port

```bash
tlssled 192.168.1.100 8443
```

**Result**: Scans non-standard HTTPS port.

### Verbose Mode

```bash
tlssled -v 192.168.1.100 443
```

**Result**: Shows additional details during scanning.

### Default Port

```bash
tlssled 192.168.1.100
```

**Result**: Uses default port 443.

### Quick Assessment

```bash
tlssled 192.168.1.100 443 2>/dev/null
```

**Result**: Suppresses errors for cleaner output.

### Multiple Hosts

```bash
for host in web1 web2 web3; do
  echo "=== $host ==="
  tlssled "$host" 443
done
```

**Result**: Scans multiple hosts sequentially.

### Save Output

```bash
tlssled 192.168.1.100 443 2>&1 | tee results.txt
```

**Result**: Saves combined output to file.

### Educational Review

```bash
tlssled 192.168.1.100 443 | less
```

**Result**: Paginated output for detailed examination.

## 6. Advanced Techniques

### Batch Assessment Script

```bash
#!/bin/bash
TARGETS="targets.txt"
OUTPUT_DIR="ssl_assessment_$(date +%Y%m%d_%H%M%S)"
mkdir -p "$OUTPUT_DIR"

while IFS= read -r target; do
  echo "Assessing $target..."
  tlssled "$target" 443 > "$OUTPUT_DIR/$target.txt" 2>&1
done < "$TARGETS"

echo "Results saved to $OUTPUT_DIR"
```

### Network-Wide Assessment

```bash
for i in $(seq 1 100); do
  echo "=== 192.168.1.$i ==="
  tlssled 192.168.1.$i 443 2>/dev/null
done
```

### Combining with Other Tools

```bash
# Find HTTPS hosts, then assess
nmap -p 443 192.168.1.0/24 -oG - | grep "443/open" | awk '{print $2}' | while read host; do
  tlssled "$host" 443
done
```

### Custom Assessment

```bash
# Run individual tools with custom options
sslscan --bugs 192.168.1.100:443
sslyze --regular --json results.json 192.168.1.100:443
```

### Comparison Assessment

```bash
tlssled server1 443 > server1.txt
tlssled server2 443 > server2.txt
diff server1.txt server2.txt
```

### Automation Integration

```bash
# CI/CD integration
if tlssled "$DEPLOY_URL" 443 | grep -q "VULNERABLE"; then
  echo "SSL/TLS vulnerabilities detected!"
  exit 1
fi
```

## 7. Detection & Defense

### Network Detection

**SSL/TLS Handshake Monitoring**: Detect rapid SSL/TLS connections indicating scanning.

**Tool Fingerprinting**: Identify sslscan and sslyze patterns in network traffic.

**Connection Frequency**: Track hosts performing repeated SSL/TLS assessments.

### System Detection

**SSL/TLS Logs**: Review server logs for unusual connection patterns.

**Access Monitoring**: Track assessment attempts against SSL/TLS services.

### Defense Strategies

**Disable Weak Protocols**: Remove SSL 2.0, SSL 3.0, TLS 1.0, and TLS 1.1 support.

**Remove Weak Ciphers**: Disable RC4, DES, 3DES, and export cipher suites.

**Enable TLS 1.3**: Prefer TLS 1.3 for improved security and performance.

**Certificate Management**: Use valid certificates from trusted CAs.

**HSTS Implementation**: Enable HTTP Strict Transport Security.

**Regular Assessments**: Schedule periodic SSL/TLS configuration audits.

## 8. Troubleshooting

### Tool Not Found

**Install Dependencies**: Ensure sslscan and sslyze are installed.

```bash
sudo apt install sslscan sslyze
```

### Connection Refused

**Port Verification**: Confirm target port is open.

```bash
nmap -p 443 192.168.1.100
```

**Firewall Issues**: Check for firewall blocking SSL/TLS connections.

### Partial Results

**Server Configuration**: Target may not support all protocols.

**Network Issues**: Verify network connectivity.

### Timeout Issues

**Network Latency**: Check network connectivity.

**Server Load**: Target server may be overloaded.

### Inconsistent Results

**Load Balancers**: Multiple servers behind load balancer may have different configs.

**Dynamic Configurations**: Server may change configurations between tool runs.

### No Output

**Command Syntax**: Verify correct hostname and port format.

**Tool Execution**: Run individual tools to diagnose issues.

```bash
sslscan 192.168.1.100:443
sslyze 192.168.1.100:443
```

## Resources

- **tlssled Kali Documentation**: https://www.kali.org/tools/tlssled/
- **sslscan GitHub**: https://github.com/rbsec/sslscan
- **sslyze GitHub**: https://github.com/nabla-c0d3/sslyze
- **SSL/TLS Best Practices**: https://wiki.mozilla.org/Security/Server_Side_TLS

## Related Tools

**sslscan**: Standalone SSL/TLS scanner with colored output and vulnerability detection.

**sslyze**: Python-based SSL/TLS analyzer with JSON output and comprehensive testing.

**testssl.sh**: Comprehensive SSL/TLS testing script with extensive checks.

**nmap**: Network scanner with SSL/TLS enumeration scripts.
