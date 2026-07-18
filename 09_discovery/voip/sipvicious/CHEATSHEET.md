---
title: "sipvicious Cheatsheet"
description: "Quick reference for sipvicious SIP security audit toolkit"
mitre_tactic: discovery
mitre_tactic_id: TA0007
mitre_subcategory: voip
categories:
  - cheatsheet
  - voip
  - security-audit
tags:
  - sipvicious
  - sip
  - voip
  - quick-reference
---

# sipvicious Cheatsheet

## Quick Start

```bash
# Network scan
svmap 192.168.1.0/24

# Extension enumeration
svwar -e 1000-9999 192.168.1.100

# Password crack
svcrack -u 1000 -d passwords.txt 192.168.1.100

# View results
svreport list

# Stop scans
svcrash --fromip=192.168.1.100
```

## Module Reference

### svmap - Network Scanner

```bash
# Basic scan
svmap 192.168.1.0/24

# Scan ports
svmap -p 5060-5061 192.168.1.0/24

# Identify devices
svmap --identify 192.168.1.0/24

# Stealth scan
svmap -S -r 10 192.168.1.0/24

# Save results
svmap 192.168.1.0/24 -o scan.txt
```

**Flags:**

| Flag | Description |
|------|-------------|
| `-p` | Port range |
| `-s` | Source port |
| `-l` | Local interface |
| `--identify` | Fingerprint |
| `-e` | Extension range |
| `-m` | Method |
| `-S` | Stealth mode |
| `-r` | Rate (pkt/s) |
| `-o` | Output file |

### svwar - Extension War Dialer

```bash
# Numeric range
svwar -e 1000-9999 192.168.1.100

# Dictionary
svwar -d extensions.txt 192.168.1.100

# REGISTER method
svwar -e 1000-9999 -m REGISTER 192.168.1.100

# INVITE method
svwar -e 1000-9999 -m INVITE 192.168.1.100

# Resume scan
svwar --resume 192.168.1.100

# Fast scan
svwar -e 1000-9999 -c 20 -r 100 192.168.1.100
```

**Flags:**

| Flag | Description |
|------|-------------|
| `-e` | Extension range |
| `-d` | Dictionary file |
| `-m` | Method (REGISTER/INVITE/OPTIONS) |
| `-r` | Rate |
| `-c` | Concurrency |
| `-p` | Port |
| `--resume` | Resume scan |

### svcrack - Credential Cracker

```bash
# Dictionary attack
svcrack -u 1000 -d passwords.txt 192.168.1.100

# Password range
svcrack -u 1000 -c 1000-9999 192.168.1.100

# Multiple extensions
svcrack -e 1000-1100 -d passwords.txt 192.168.1.100

# Stealthy crack
svcrack -u 1000 -d passwords.txt -r 10 -S 192.168.1.100

# Resume attack
svcrack --resume 192.168.1.100
```

**Flags:**

| Flag | Description |
|------|-------------|
| `-u` | Extension |
| `-d` | Dictionary |
| `-e` | Extension range |
| `-c` | Password range |
| `-r` | Rate |
| `-S` | Stealth mode |
| `--resume` | Resume attack |

### svreport - Reporter

```bash
# List results
svreport list

# Export CSV
svreport export -f csv -o results.csv

# Export PDF
svreport export -f pdf -o report.pdf

# Search
svreport search --query "extension:1000"

# Purge old
svreport purge --older-than 30
```

### svcrash - Controller

```bash
# Stop by IP
svcrash --fromip=192.168.1.100

# Stop by port
svcrash --fromport=5060 --toport=5080

# Stop by server
svcrash --server=192.168.1.100

# Force stop
svcrash --proto=udp --fromip=192.168.1.100
```

## Examples

### Full Assessment

```bash
# Phase 1: Discovery
svmap 192.168.1.0/24 -o discovery.txt

# Phase 2: Enumeration
svwar -e 1000-9999 -m REGISTER 192.168.1.100

# Phase 3: Cracking
svcrack -e 1000-1100 -d rockyou.txt 192.168.1.100

# Phase 4: Report
svreport export -f pdf -o audit.pdf
```

### Stealthy Assessment

```bash
# Slow discovery
svmap -S -r 5 192.168.1.0/24

# Careful enumeration
svwar -e 1000-9999 -r 10 -c 5 192.168.1.100

# Patient cracking
svcrack -u 1000 -d passwords.txt -r 5 -S 192.168.1.100
```

### Automated Script

```bash
#!/bin/bash
TARGET=$1
svmap $TARGET -o scan.txt
svwar -e 1000-9999 $TARGET -o ext.txt
svcrack -e 1000-1100 -d rockyou.txt $TARGET
svreport export -f pdf -o report.pdf
```

## Chaining

### With Nmap

```bash
# Find SIP hosts first
nmap -sU -p 5060 192.168.1.0/24 -oG - | \
    grep "5060/open" | awk '{print $2}' > sip_hosts.txt

# Then scan with svmap
while read host; do svmap $host; done < sip_hosts.txt
```

### With sipsak

```bash
# Quick probe
sipsak -s sip:user@192.168.1.100

# Then detailed scan
svmap 192.168.1.100
```

### With Hydra

```bash
# Use hydra for additional brute-force
hydra -l 1000 -P passwords.txt 192.168.1.100 sip
```

### Database Queries

```bash
# Direct SQLite queries
sqlite3 ~/.sipvicious/sipvicious.db \
    "SELECT * FROM devices WHERE user_agent LIKE '%Asterisk%'"

sqlite3 ~/.sipvicious/sipvicious.db \
    "SELECT extension, password FROM credentials"
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

### Database Issues

```bash
# Check database
ls -la ~/.sipvicious/

# Reset database
rm ~/.sipvicious/sipvicious.db

# View database
sqlite3 ~/.sipvicious/sipvicious.db ".tables"
```

### Scan Stuck

```bash
# Find stuck process
ps aux | grep sv

# Kill stuck scan
pkill -f svwar

# Reset and resume
svwar --resume --force 192.168.1.100
```

### Rate Issues

| Symptom | Solution |
|---------|----------|
| Too slow | Increase `-r` |
| Too fast | Decrease `-r`, add `-S` |
| Packet loss | Add `-S`, reduce rate |
| High CPU | Reduce `-c` concurrency |

### Verbose Debug

```bash
# Enable verbose
svmap -v 192.168.1.100

# Extra verbose
svmap -vv 192.168.1.100

# Check logs
ls ~/.sipvicious/*.log
```

---

**Last Updated**: 2026
**Tool Version**: sipvicious 0.3.4
**Category**: VoIP Security Audit
