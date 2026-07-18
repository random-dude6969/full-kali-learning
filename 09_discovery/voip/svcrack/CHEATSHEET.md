---
title: "svcrack Cheatsheet"
description: "Quick reference for svcrack SIP credential brute-force tool"
mitre_tactic: discovery
mitre_tactic_id: TA0007
mitre_subcategory: voip
categories:
  - cheatsheet
  - voip
  - brute-force
tags:
  - svcrack
  - sip
  - brute-force
  - quick-reference
---

# svcrack Cheatsheet

## Quick Start

```bash
# Dictionary attack
svcrack -u 1000 -d passwords.txt 192.168.1.100

# Numeric PIN range
svcrack -u 1000 -c 1000-9999 192.168.1.100

# Multiple extensions
svcrack -e 1000-1100 -d passwords.txt 192.168.1.100

# Stealthy attack
svcrack -u 1000 -d passwords.txt -r 10 -S 192.168.1.100

# Resume interrupted attack
svcrack --resume 192.168.1.100
```

## Common Flags

### Target

| Flag | Description | Example |
|------|-------------|---------|
| `-u` | Single extension | `-u 1000` |
| `-e` | Extension range | `-e 1000-9999` |
| `-p` | SIP port | `-p 5060` |

### Password

| Flag | Description | Example |
|------|-------------|---------|
| `-d` | Dictionary file | `-d passwords.txt` |
| `-c` | Password range | `-c 1000-9999` |
| `-x` | Password prefix | `-x admin` |

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

### Control

| Flag | Description |
|------|-------------|
| `--resume` | Resume attack |
| `--force` | Force resume |
| `-v` | Verbose |
| `--debug` | Debug mode |

## Examples

### Single Extension

```bash
# Dictionary attack
svcrack -u 1000 -d /usr/share/wordlists/rockyou.txt 192.168.1.100

# 4-digit PINs
svcrack -u 1000 -c 1000-9999 192.168.1.100

# 6-digit PINs
svcrack -u 1000 -c 100000-999999 192.168.1.100
```

### Multiple Extensions

```bash
# Range
svcrack -e 1000-1100 -d passwords.txt 192.168.1.100

# From file
cat extensions.txt | while read ext; do
    svcrack -u $ext -d passwords.txt 192.168.1.100
done
```

### Rate Control

```bash
# Slow (stealthy)
svcrack -u 1000 -d passwords.txt -r 10 -S 192.168.1.100

# Balanced
svcrack -u 1000 -d passwords.txt -r 50 192.168.1.100

# Fast
svcrack -u 1000 -d passwords.txt -r 200 -c 30 192.168.1.100
```

### Advanced

```bash
# Through proxy
svcrack -u 1000 -d passwords.txt -P 10.0.0.1 192.168.1.100

# Custom User-Agent
svcrack -u 1000 -d passwords.txt --user-agent "Polycom/5.0" 192.168.1.100

# Non-standard port
svcrack -u 1000 -d passwords.txt -p 5061 192.168.1.100
```

## Chaining

### With svmap

```bash
# Discover extensions first
svmap 192.168.1.0/24 -e 1000-9999

# Then crack found extensions
svcrack -u 1000 -d passwords.txt 192.168.1.100
```

### With svwar

```bash
# Enumerate valid extensions
svwar -e 1000-9999 192.168.1.100

# Crack each extension
for ext in $(svwar -e 1000-9999 192.168.1.100 | grep -oP '\d+'); do
    svcrack -u $ext -d passwords.txt 192.168.1.100
done
```

### With Hydra

```bash
# Use hydra for comparison
hydra -l 1000 -P passwords.txt 192.168.1.100 sip
```

### Verification

```bash
# Verify found password
sipsak -s sip:1000@192.168.1.100 -l 1000 -p FOUND_PASSWORD -v
```

## Troubleshooting

### No Results

```bash
# Verify extension exists
svmap 192.168.1.100 -e 1000-1000

# Test manually
sipsak -s sip:1000@192.168.1.100 -l 1000 -p test

# Check verbose output
svcrack -v -u 1000 -c 1000-1001 192.168.1.100
```

### Connection Issues

```bash
# Verify target
ping 192.168.1.100
nc -zvu 192.168.1.100 5060

# Increase timeout
svcrack -u 1000 -d passwords.txt --timeout 30 192.168.1.100

# Try TCP
svcrack -u 1000 -d passwords.txt -t tcp 192.168.1.100
```

### Scan Stuck

```bash
# Kill process
pkill -f svcrack

# Reset state
rm ~/.sipvicious/svcrack_*

# Resume
svcrack --resume 192.168.1.100
```

### Return Codes

| Code | Meaning |
|------|---------|
| 0 | Password found |
| 1 | No password found |
| 2 | Connection error |
| 3 | Auth error |
| 4 | Timeout |

### Debug Commands

```bash
# Verbose
svcrack -v -u 1000 -d passwords.txt 192.168.1.100

# Debug
svcrack --debug -u 1000 -c 1000-1001 192.168.1.100

# Check state
ls -la ~/.sipvicious/

# View database
sqlite3 ~/.sipvicious/sipvicious.db "SELECT * FROM credentials;"
```

---

**Last Updated**: 2026
**Tool Version**: sipvicious 0.3.4
**Category**: SIP Credential Testing
