---
title: "sslscan Quick Reference"
description: "Quick reference guide for sslscan SSL/TLS scanner"
categories:
  - "ssl"
  - "tls"
  - "cheatsheet"
tags:
  - "quick-reference"
  - "ssl"
  - "tls"
  - "scanner"
---

# sslscan CHEATSHEET

## Quick Start

**Basic scan**: `sslscan 192.168.1.100:443`

**Custom port**: `sslscan 192.168.1.100:8443`

**All vulnerabilities**: `sslscan --bugs 192.168.1.100:443`

**Certificate details**: `sslscan --show-certificate 192.168.1.100:443`

**JSON output**: `sslscan --json results.json 192.168.1.100:443`

## Common Flags

| Flag | Description | Example |
|------|-------------|---------|
| `--ssl-2` | Test SSL 2.0 | `sslscan --ssl-2 192.168.1.100:443` |
| `--ssl-3` | Test SSL 3.0 | `sslscan --ssl-3 192.168.1.100:443` |
| `--tls10` | Test TLS 1.0 | `sslscan --tls10 192.168.1.100:443` |
| `--tls11` | Test TLS 1.1 | `sslscan --tls11 192.168.1.100:443` |
| `--tls12` | Test TLS 1.2 | `sslscan --tls12 192.168.1.100:443` |
| `--tls13` | Test TLS 1.3 | `sslscan --tls13 192.168.1.100:443` |
| `--bugs` | All bug tests | `sslscan --bugs 192.168.1.100:443` |
| `--heartbleed` | Test Heartbleed | `sslscan --heartbleed 192.168.1.100:443` |
| `--poodle` | Test POODLE | `sslscan --poodle 192.168.1.100:443` |
| `--robot` | Test ROBOT | `sslscan --robot 192.168.1.100:443` |
| `--show-certificate` | Show cert details | `sslscan --show-certificate 192.168.1.100:443` |
| `--xml FILE` | XML output | `sslscan --xml results.xml 192.168.1.100:443` |
| `--json FILE` | JSON output | `sslscan --json results.json 192.168.1.100:443` |
| `--html FILE` | HTML output | `sslscan --html results.html 192.168.1.100:443` |
| `--no-colour` | No color | `sslscan --no-colour 192.168.1.100:443` |
| `--targets FILE` | Target list | `sslscan --targets hosts.txt` |

### STARTTLS Flags

| Flag | Protocol | Example |
|------|----------|---------|
| `--starttls-ftp` | FTP | `sslscan --starttls-ftp 192.168.1.100:21` |
| `--starttls-imap` | IMAP | `sslscan --starttls-imap 192.168.1.100:143` |
| `--starttls-pop3` | POP3 | `sslscan --starttls-pop3 192.168.1.100:110` |
| `--starttls-smtp` | SMTP | `sslscan --starttls-smtp 192.168.1.100:25` |
| `--starttls-xmpp` | XMPP | `sslscan --starttls-xmpp 192.168.1.100:5222` |

## Examples

### Basic HTTPS Scan
```bash
sslscan 192.168.1.100:443
```

### All Vulnerabilities
```bash
sslscan --bugs 192.168.1.100:443
```

### Certificate Details
```bash
sslscan --show-certificate 192.168.1.100:443
```

### Specific Protocol Test
```bash
sslscan --tls13 192.168.1.100:443
```

### STARTTLS SMTP
```bash
sslscan --starttls-smtp 192.168.1.100:25
```

### XML Output
```bash
sslscan --xml results.xml 192.168.1.100:443
```

### JSON Output
```bash
sslscan --json results.json 192.168.1.100:443
```

### Target List
```bash
sslscan --targets hosts.txt
```

### No Color
```bash
sslscan --no-colour 192.168.1.100:443
```

## Chaining

### Chain with nmap
```bash
# Find HTTPS hosts, then scan SSL/TLS
nmap -p 443 192.168.1.0/24 -oG - | grep "443/open" | awk '{print $2}' | while read host; do
  echo "=== $host ==="
  sslscan "$host:443"
done
```

### Chain with sslyze
```bash
# Quick scan with sslscan, detailed with sslyze
sslscan 192.168.1.100:443 > quick.txt
sslyze --regular 192.168.1.100:443 > detailed.txt
```

### Batch Processing
```bash
for i in $(seq 1 100); do
  sslscan 192.168.1.$i:443 > "ssl_192.168.1.$i.txt"
done
```

## Troubleshooting

### Connection refused
- Verify port: `nmap -p 443 192.168.1.100`
- Check firewall: `iptables -L -n | grep 443`

### No protocols detected
- Server may not support SSL/TLS
- Try: `openssl s_client -connect 192.168.1.100:443`

### Partial results
- Load balancers may have different configs
- Try different protocols individually

### Timeout issues
- Check network: `ping -c 2 192.168.1.100`
- Server may be overloaded

### Certificate errors
- Self-signed: normal for internal services
- Expired: check with `--show-certificate`
- Chain issues: verify with `openssl s_client`
