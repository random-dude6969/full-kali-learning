---
title: "svmap Cheatsheet"
description: "Quick reference for svmap SIP network scanner"
mitre_tactic: discovery
mitre_tactic_id: TA0007
mitre_subcategory: voip
categories:
  - cheatsheet
  - voip
  - network-scanning
tags:
  - svmap
  - sip
  - voip
  - quick-reference
---

# svmap Cheatsheet

## Quick Start

```bash
# Scan subnet
svmap 192.168.1.0/24

# Scan single host
svmap 192.168.1.100

# Scan with identification
svmap --identify 192.168.1.0/24

# Stealth scan
svmap -S -r 10 192.168.1.0/24
```

## Common Flags

### Target

| Syntax | Description |
|--------|-------------|
| `192.168.1.100` | Single host |
| `192.168.1.0/24` | CIDR range |
| `192.168.1.1-50` | IP range |
| `sip.example.com` | Hostname |

### Ports

| Flag | Description | Example |
|------|-------------|---------|
| `-p` | Port range | `-p 5060-5061` |
| `-s` | Source port | `-s 5060` |

### Scan Options

| Flag | Description | Example |
|------|-------------|---------|
| `-m` | SIP method | `-m OPTIONS` |
| `-e` | Extension range | `-e 1000-9999` |
| `--identify` | Fingerprint | `--identify` |
| `-l` | Interface | `-l eth0` |

### Performance

| Flag | Description | Default |
|------|-------------|---------|
| `-r` | Rate (pkt/s) | 100 |
| `-c` | Concurrency | 10 |
| `-t` | Timeout (s) | 5 |

### Stealth

| Flag | Description |
|------|-------------|
| `-S` | Stealth mode |
| `--user-agent` | Custom User-Agent |
| `-P` | Proxy address |

### Output

| Flag | Description |
|------|-------------|
| `-o` | Output file |
| `--resume` | Resume scan |
| `-v` | Verbose |

## Examples

### Basic Scanning

```bash
# /24 subnet
svmap 192.168.1.0/24

# Single host
svmap 192.168.1.100

# IP range
svmap 192.168.1.1-192.168.1.50

# Multiple subnets
svmap 192.168.1.0/24 192.168.2.0/24
```

### Port Scanning

```bash
# Standard ports
svmap -p 5060-5061 192.168.1.0/24

# Non-standard ports
svmap -p 5080-5090 192.168.1.0/24

# Single port
svmap -p 5060 192.168.1.0/24
```

### Device Identification

```bash
# Fingerprint devices
svmap --identify 192.168.1.0/24

# Verbose identification
svmap --identify -v 192.168.1.0/24

# INVITE method
svmap -m INVITE --identify 192.168.1.0/24
```

### Rate Control

```bash
# Fast (LAN)
svmap -r 200 192.168.1.0/24

# Balanced
svmap -r 100 192.168.1.0/24

# Slow (WAN)
svmap -r 25 192.168.1.0/24

# Stealthy
svmap -r 10 -S 192.168.1.0/24
```

### Stealth Scanning

```bash
# Stealth mode
svmap -S 192.168.1.0/24

# Custom User-Agent
svmap --user-agent "Polycom/5.0" 192.168.1.0/24

# Through proxy
svmap -P 10.0.0.1 192.168.1.0/24

# Custom source port
svmap -s 5060 192.168.1.0/24
```

### Extension Enumeration

```bash
# Range
svmap -e 1000-9999 192.168.1.100

# Fast enum
svmap -e 1000-9999 -r 200 192.168.1.100

# Stealthy enum
svmap -e 1000-9999 -r 10 -S 192.168.1.100
```

### Output and Resume

```bash
# Save results
svmap 192.168.1.0/24 -o scan.txt

# Resume scan
svmap --resume 192.168.1.0/24

# Verbose output
svmap -v 192.168.1.0/24
```

## Chaining

### With Other SIP Tools

```bash
# svmap → svwar
svmap 192.168.1.0/24          # Find SIP hosts
svwar -e 1000-9999 192.168.1.100  # Enumerate

# svmap → svcrack
svmap --identify 192.168.1.0/24  # Find devices
svcrack -u 1000 -d passwords.txt 192.168.1.100  # Crack

# svmap → sipsak
svmap 192.168.1.0/24          # Quick scan
sipsak -s sip:user@192.168.1.100  # Detailed probe
```

### With Network Tools

```bash
# Nmap + svmap
nmap -sU -p 5060 192.168.1.0/24 -oG - | \
    grep "5060/open" | awk '{print $2}' > sip_hosts.txt
while read host; do svmap $host; done < sip_hosts.txt

# Export for analysis
svmap 192.168.1.0/24 -o results.txt
awk '/200 OK/ {print $1}' results.txt > active.txt
```

### With Reporting

```bash
# Scan → Report
svmap 192.168.1.0/24 -o scan.txt
svreport export -f csv -o results.csv
svreport export -f pdf -o report.pdf
```

## Troubleshooting

### No Results

```bash
# Verify target
ping 192.168.1.100
nc -zvu 192.168.1.100 5060

# Test with sipsak
sipsak -s sip:user@192.168.1.100 -v

# Check firewall
sudo iptables -L -n | grep 5060
```

### Slow Scans

```bash
# Increase rate
svmap -r 200 192.168.1.0/24

# Reduce timeout
svmap -t 3 192.168.1.0/24

# Skip unreachable
svmap --skip-unreachable 192.168.1.0/24
```

### Scan Stuck

```bash
# Kill process
pkill -f svmap

# Reset state
rm ~/.sipvicious/svmap_*

# Resume
svmap --resume 192.168.1.0/24
```

### Return Codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | Error |
| 2 | No devices found |

### Debug Commands

```bash
# Verbose
svmap -v 192.168.1.100

# Debug
svmap --debug 192.168.1.100

# Check state
ls -la ~/.sipvicious/

# View database
sqlite3 ~/.sipvicious/svmap_*.db "SELECT * FROM devices;"
```

---

**Last Updated**: 2026
**Tool Version**: sipvicious 0.3.4
**Category**: SIP Network Scanning
