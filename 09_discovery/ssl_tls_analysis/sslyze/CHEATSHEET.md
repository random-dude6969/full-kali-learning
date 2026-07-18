---
title: "sslyze Quick Reference"
description: "Quick reference guide for sslyze SSL/TLS analyzer"
categories:
  - "ssl"
  - "tls"
  - "cheatsheet"
tags:
  - "quick-reference"
  - "ssl"
  - "tls"
  - "analyzer"
---

# sslyze CHEATSHEET

## Quick Start

**Basic scan**: `sslyze 192.168.1.100:443`

**Regular scan**: `sslyze --regular 192.168.1.100:443`

**JSON output**: `sslyze --json results.json 192.168.1.100:443`

**Certificate info**: `sslyze --cert-info 192.168.1.100:443`

**Vulnerability test**: `sslyze --heartbleed --robot 192.168.1.100:443`

## Common Flags

| Flag | Description | Example |
|------|-------------|---------|
| `--regular` | Regular scan | `sslyze --regular 192.168.1.100:443` |
| `--json FILE` | JSON output | `sslyze --json out.json 192.168.1.100:443` |
| `--xml FILE` | XML output | `sslyze --xml out.xml 192.168.1.100:443` |
| `--cert-info` | Certificate info | `sslyze --cert-info 192.168.1.100:443` |
| `--chain` | Certificate chain | `sslyze --chain 192.168.1.100:443` |
| `--ocsp-stapling` | OCSP stapling | `sslyze --ocsp-stapling 192.168.1.100:443` |
| `--heartbleed` | Heartbleed test | `sslyze --heartbleed 192.168.1.100:443` |
| `--robot` | ROBOT test | `sslyze --robot 192.168.1.100:443` |
| `--compression` | Compression test | `sslyze --compression 192.168.1.100:443` |
| `--file FILE` | Target list | `sslyze --file targets.txt` |
| `--quiet` | Quiet mode | `sslyze --quiet 192.168.1.100:443` |
| `--timeout NUM` | Timeout | `sslyze --timeout 10 192.168.1.100:443` |

### Protocol Flags

| Flag | Description | Example |
|------|-------------|---------|
| `--ssl2` | SSL 2.0 | `sslyze --ssl2 192.168.1.100:443` |
| `--ssl3` | SSL 3.0 | `sslyze --ssl3 192.168.1.100:443` |
| `--tls10` | TLS 1.0 | `sslyze --tls10 192.168.1.100:443` |
| `--tls11` | TLS 1.1 | `sslyze --tls11 192.168.1.100:443` |
| `--tls12` | TLS 1.2 | `sslyze --tls12 192.168.1.100:443` |
| `--tls13` | TLS 1.3 | `sslyze --tls13 192.168.1.100:443` |

### STARTTLS Flags

| Flag | Protocol | Example |
|------|----------|---------|
| `--starttls ftp` | FTP | `sslyze --starttls ftp 192.168.1.100:21` |
| `--starttls imap` | IMAP | `sslyze --starttls imap 192.168.1.100:143` |
| `--starttls pop3` | POP3 | `sslyze --starttls pop3 192.168.1.100:110` |
| `--starttls smtp` | SMTP | `sslyze --starttls smtp 192.168.1.100:25` |
| `--starttls xmpp` | XMPP | `sslyze --starttls xmpp 192.168.1.100:5222` |
| `--starttls ldap` | LDAP | `sslyze --starttls ldap 192.168.1.100:389` |
| `--starttls rdp` | RDP | `sslyze --starttls rdp 192.168.1.100:3389` |

## Examples

### Basic Scan
```bash
sslyze 192.168.1.100:443
```

### Regular Scan
```bash
sslyze --regular 192.168.1.100:443
```

### JSON Output
```bash
sslyze --json results.json 192.168.1.100:443
```

### Certificate Info
```bash
sslyze --cert-info --chain 192.168.1.100:443
```

### Vulnerability Test
```bash
sslyze --heartbleed --robot --compression 192.168.1.100:443
```

### STARTTLS SMTP
```bash
sslyze --starttls smtp 192.168.1.100:25
```

### Target List
```bash
sslyze --file targets.txt
```

### Quiet JSON
```bash
sslyze --quiet --json results.json 192.168.1.100:443
```

### OCSP Stapling
```bash
sslyze --ocsp-stapling 192.168.1.100:443
```

### Specific Protocols
```bash
sslyze --tls12 --tls13 192.168.1.100:443
```

## Chaining

### Chain with sslscan
```bash
# Quick scan with sslscan, detailed with sslyze
sslscan 192.168.1.100:443 > quick.txt
sslyze --regular --json detailed.json 192.168.1.100:443
```

### Chain with nmap
```bash
# Find HTTPS hosts, then analyze
nmap -p 443 192.168.1.0/24 -oG - | grep "443/open" | awk '{print $2}' | while read host; do
  sslyze --json "$host.json" "$host:443"
done
```

### Parse JSON
```bash
sslyze --json results.json 192.168.1.100:443
jq '.scan_result.protocols_support' results.json
```

### Batch Processing
```bash
sslyze --file targets.txt --json all_results.json
```

## Troubleshooting

### Connection refused
- Verify port: `nmap -p 443 192.168.1.100`
- Check firewall: `iptables -L -n | grep 443`

### No results
- Server may not support SSL/TLS
- Try: `openssl s_client -connect 192.168.1.100:443`

### Timeout issues
- Increase timeout: `sslyze --timeout 10 192.168.1.100:443`
- Check network: `ping -c 2 192.168.1.100`

### JSON parse errors
- Corrupted output: re-run with `--quiet`
- Version compatibility: check Python version

### Certificate errors
- Self-signed: normal for internal services
- Expired: check with `--cert-info`
- Chain issues: verify with `--chain`
