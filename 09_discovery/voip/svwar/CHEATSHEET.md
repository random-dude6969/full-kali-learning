---
title: "svwar Cheatsheet"
description: "Quick reference for svwar SIP extension war dialer"
mitre_tactic: discovery
mitre_tactic_id: TA0007
mitre_subcategory: voip
categories:
  - cheatsheet
  - voip
  - enumeration
tags:
  - svwar
  - sip
  - war-dialing
  - quick-reference
---

# svwar Cheatsheet

## Quick Start

```bash
# Basic range enumeration
svwar -e 1000-9999 192.168.1.100

# Dictionary enumeration
svwar -d extensions.txt 192.168.1.100

# Stealth enumeration
svwar -e 1000-9999 -r 10 -S 192.168.1.100

# Resume scan
svwar --resume 192.168.1.100
```

## Common Flags

### Target

| Flag | Description | Example |
|------|-------------|---------|
| `-e` | Extension range | `-e 1000-9999` |
| `-d` | Dictionary file | `-d extensions.txt` |
| `-p` | SIP port | `-p 5060` |

### Method

| Flag | Description | Notes |
|------|-------------|-------|
| `-m REGISTER` | Registration | Default, reliable |
| `-m INVITE` | Call initiation | Most reliable |
| `-m OPTIONS` | Query | Stealthy |

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
| `-s` | Source port |

### Output

| Flag | Description |
|------|-------------|
| `-o` | Output file |
| `--resume` | Resume scan |
| `-v` | Verbose |

## Examples

### Range Enumeration

```bash
# Standard range
svwar -e 1000-9999 192.168.1.100

# Small range
svwar -e 1000-1099 192.168.1.100

# Large range
svwar -e 1000-99999 192.168.1.100
```

### Dictionary Enumeration

```bash
# From file
svwar -d extensions.txt 192.168.1.100

# Common extensions
echo -e "1000\n1001\n1002\n1003" > common.txt
svwar -d common.txt 192.168.1.100
```

### Method Selection

```bash
# REGISTER (default)
svwar -e 1000-9999 -m REGISTER 192.168.1.100

# INVITE (reliable)
svwar -e 1000-9999 -m INVITE 192.168.1.100

# OPTIONS (stealthy)
svwar -e 1000-9999 -m OPTIONS 192.168.1.100
```

### Rate Control

```bash
# Fast
svwar -e 1000-9999 -r 200 192.168.1.100

# Balanced
svwar -e 1000-9999 -r 100 192.168.1.100

# Slow
svwar -e 1000-9999 -r 10 192.168.1.100

# Stealthy
svwar -e 1000-9999 -r 10 -S 192.168.1.100
```

### Stealth Enumeration

```bash
# Stealth mode
svwar -e 1000-9999 -S 192.168.1.100

# Custom User-Agent
svwar -e 1000-9999 --user-agent "Polycom/5.0" 192.168.1.100

# Through proxy
svwar -e 1000-9999 -P 10.0.0.1 192.168.1.100

# Custom source port
svwar -e 1000-9999 -s 5060 192.168.1.100
```

### Resume

```bash
# Resume interrupted scan
svwar --resume 192.168.1.100

# Force resume
svwar --resume --force 192.168.1.100
```

## Chaining

### With svmap

```bash
# Find SIP hosts first
svmap 192.168.1.0/24

# Then enumerate extensions
svwar -e 1000-9999 192.168.1.100
```

### With svcrack

```bash
# Enumerate extensions
svwar -e 1000-9999 192.168.1.100

# Crack found extensions
svcrack -e 1000-1100 -d passwords.txt 192.168.1.100
```

### With sipsak

```bash
# Quick extension test
sipsak -s sip:1000@192.168.1.100

# Then detailed enumeration
svwar -e 1000-9999 192.168.1.100
```

### With svreport

```bash
# Enumerate
svwar -e 1000-9999 192.168.1.100

# Generate report
svreport export -f pdf -o extensions.pdf
```

## Troubleshooting

### No Extensions Found

```bash
# Verify target
ping 192.168.1.100
nc -zvu 192.168.1.100 5060

# Test manually
sipsak -s sip:1000@192.168.1.100 -v

# Try INVITE method
svwar -e 1000-9999 -m INVITE 192.168.1.100
```

### Slow Enumeration

```bash
# Increase rate
svwar -e 1000-9999 -r 200 192.168.1.100

# Reduce timeout
svwar -e 1000-9999 -t 3 192.168.1.100
```

### Scan Stuck

```bash
# Kill process
pkill -f svwar

# Reset state
rm ~/.sipvicious/svwar_*

# Resume
svwar --resume 192.168.1.100
```

### Return Codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | Error |
| 2 | No extensions found |

### Debug Commands

```bash
# Verbose
svwar -v -e 1000-9999 192.168.1.100

# Debug
svwar --debug -e 1000-9999 192.168.1.100

# Check state
ls -la ~/.sipvicious/

# View database
sqlite3 ~/.sipvicious/svwar_*.db "SELECT * FROM extensions;"
```

---

**Last Updated**: 2026
**Tool Version**: sipvicious 0.3.4
**Category**: SIP Extension Enumeration
