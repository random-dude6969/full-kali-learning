---
title: "sara Quick Reference"
description: "Quick reference guide for sara Security Auditor's Research Assistant"
categories:
  - "network"
  - "cheatsheet"
tags:
  - "quick-reference"
  - "reconnaissance"
  - "security-audit"
---

# sara CHEATSHEET

## Quick Start

**Basic scan**: `sara -r 192.168.1.0/24`

**Port scan**: `sara -r 192.168.1.100 -p 1-1000`

**OS detection**: `sara -r 192.168.1.100 -O`

**Service detection**: `sara -r 192.168.1.100 -sV`

**Comprehensive**: `sara -r 192.168.1.100 -A`

## Common Flags

| Flag | Description | Example |
|------|-------------|---------|
| `-r RANGE` | Target range | `sara -r 192.168.1.0/24` |
| `-p PORTS` | Port range | `sara -r 192.168.1.100 -p 80,443` |
| `-v` | Verbose | `sara -r 192.168.1.100 -v` |
| `-o FILE` | Output file | `sara -r 192.168.1.100 -o results.txt` |
| `-a` | All ports | `sara -r 192.168.1.100 -a` |
| `-sS` | SYN scan | `sara -r 192.168.1.100 -sS` |
| `-sT` | Connect scan | `sara -r 192.168.1.100 -sT` |
| `-sU` | UDP scan | `sara -r 192.168.1.100 -sU` |
| `-O` | OS detection | `sara -r 192.168.1.100 -O` |
| `-sV` | Version detect | `sara -r 192.168.1.100 -sV` |
| `-A` | Aggressive scan | `sara -r 192.168.1.100 -A` |
| `-T timing` | Timing template | `sara -r 192.168.1.100 -T4` |
| `-Pn` | Skip host discovery | `sara -r 192.168.1.100 -Pn` |
| `-oX FILE` | XML output | `sara -r 192.168.1.100 -oX results.xml` |
| `-oG FILE` | Grepable output | `sara -r 192.168.1.100 -oG results.txt` |

## Examples

### Basic Network Scan
```bash
sara -r 192.168.1.0/24
```

### Specific Ports
```bash
sara -r 192.168.1.100 -p 21,22,23,25,53,80,443
```

### SYN Scan
```bash
sara -r 192.168.1.100 -sS
```

### OS Detection
```bash
sara -r 192.168.1.100 -O --osscan-guess
```

### Service Detection
```bash
sara -r 192.168.1.100 -sV --version-intensity 9
```

### Comprehensive Scan
```bash
sara -r 192.168.1.100 -A -v
```

### All Ports
```bash
sara -r 192.168.1.100 -p-
```

### XML Output
```bash
sara -r 192.168.1.0/24 -oX results.xml
```

### Fast Scan
```bash
sara -r 192.168.1.0/24 -T4 -p 1-1000
```

### Vulnerability Scan
```bash
sara -r 192.168.1.100 --script vuln
```

## Chaining

### Chain with nmap
```bash
# Find hosts, then scan ports
sara -r 192.168.1.0/24 -sn -oG hosts.txt
grep "Up" hosts.txt | awk '{print $2}' | while read host; do
  sara -r "$host" -sV -O
done
```

### Chain with nikto
```bash
# Find web servers, then scan
sara -r 192.168.1.0/24 -p 80,443 -oG web_hosts.txt
grep "open" web_hosts.txt | awk '{print $2}' | while read host; do
  nikto -h "$host"
done
```

### Chain with hydra
```bash
# Find services, then brute force
sara -r 192.168.1.100 -sV -p 21,22,23 -oG services.txt
```

### Batch Processing
```bash
for i in $(seq 1 10); do
  echo "=== 192.168.1.$i ==="
  sara -r "192.168.1.$i" -sV -O > "192.168.1.$i.txt"
done
```

## Troubleshooting

### No results
- Verify hosts: `sara -r 192.168.1.100 -sn`
- Check firewall: `sara -r 192.168.1.100 -Pn`
- Verify connectivity: `ping -c 2 192.168.1.100`

### Slow scanning
- Use faster timing: `sara -r 192.168.1.100 -T4`
- Reduce ports: `sara -r 192.168.1.100 -p 80,443`
- Scan smaller range

### Permission denied
- Run as root: `sudo sara -r 192.168.1.100`

### Incomplete results
- Skip host discovery: `sara -r 192.168.1.100 -Pn`
- Increase timeout
- Check network connectivity

### OS detection failures
- Use aggressive: `sara -r 192.168.1.100 -O --osscan-guess`
- Multiple probes may be needed

### Service detection issues
- Increase intensity: `sara -r 192.168.1.100 -sV --version-intensity 9`
- Check service availability
