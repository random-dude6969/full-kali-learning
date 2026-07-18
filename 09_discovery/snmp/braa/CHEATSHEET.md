---
title: "braa Quick Reference"
description: "Quick reference guide for braa SNMP community string scanner"
categories:
  - "snmp"
  - "cheatsheet"
tags:
  - "quick-reference"
  - "snmp"
  - "scanner"
---

# braa CHEATSHEET

## Quick Start

**Basic scan**: `braa public@192.168.1.0/24`

**Multi-string scan**: `braa strings.txt@192.168.1.0/24`

**High-speed scan**: `braa -t 100 public@192.168.1.0/24`

**With output**: `braa -o results.txt public@192.168.1.0/24`

**SNMPv2c**: `braa -v 2c public@192.168.1.0/24`

## Common Flags

| Flag | Description | Example |
|------|-------------|---------|
| `-t NUM` | Thread count | `braa -t 100 public@192.168.1.0/24` |
| `-p PORT` | SNMP port | `braa -p 1161 public@192.168.1.100` |
| `-q` | Quiet mode | `braa -q public@192.168.1.0/24` |
| `-v VER` | SNMP version | `braa -v 2c public@192.168.1.100` |
| `-w` | Wait/timeout | `braa -w public@192.168.1.0/24` |
| `-o FILE` | Output file | `braa -o out.txt public@192.168.1.0/24` |
| `-d` | Debug mode | `braa -d public@192.168.1.100` |
| `-r NUM` | Retries | `braa -r 3 public@192.168.1.0/24` |
| `-s IP` | Source IP | `braa -s 192.168.1.50 public@192.168.1.100` |

## Examples

### Single Host Scan
```bash
braa public@192.168.1.100
```

### Network Range Scan
```bash
braa public@192.168.1.0/24
```

### Multiple Community Strings
```bash
cat > strings.txt << 'EOF'
public
private
manager
admin
secret
EOF
braa strings.txt@192.168.1.0/24
```

### High Performance Scan
```bash
braa -t 200 -r 2 public@192.168.0.0/16
```

### Custom Port Scan
```bash
braa -p 1161 public@192.168.1.100
```

### SNMPv2c Scan
```bash
braa -v 2c public@192.168.1.0/24
```

### Quiet Mode with Output
```bash
braa -q -o results.txt public@192.168.1.0/24
```

### Debug Mode
```bash
braa -d public@192.168.1.100
```

## Chaining

### Chain with snmpwalk
```bash
# Find valid strings, then enumerate
braa -q public@192.168.1.0/24 | while read host string; do
  echo "Enumerating $host with $string"
  snmpwalk -c "$string" -v1 "$host" 1.3.6.1.2.1.1
done
```

### Chain with snmp-check
```bash
# Discover then enumerate
braa -q public@192.168.1.0/24 | while read host string; do
  snmp-check -c "$string" "$host"
done
```

### Pipe to Report
```bash
braa -q public@192.168.1.0/24 | tee discovered.txt | wc -l
```

## Troubleshooting

### No results
- Verify targets are online: `ping -c 2 192.168.1.100`
- Check SNMP running: `systemctl status snmpd`
- Verify port: `nmap -sU -p 161 192.168.1.100`

### Slow performance
- Reduce threads: `braa -t 20 public@192.168.1.0/24`
- Increase timeout: `braa -w public@192.168.1.0/24`

### Permission denied
- Run as root: `sudo braa public@192.168.1.100`

### Connection refused
- Check firewall: `iptables -L -n | grep 161`
- Check SNMP service: `snmpwalk -c public -v1 192.168.1.100 1.3.6.1.2.1.1`

### Packet loss
- Reduce threads: `braa -t 10 public@192.168.1.0/24`
- Increase retries: `braa -r 3 public@192.168.1.0/24`
