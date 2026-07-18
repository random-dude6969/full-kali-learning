---
title: "onesixtyone Quick Reference"
description: "Quick reference guide for onesixtyone SNMP community string scanner"
categories:
  - "snmp"
  - "cheatsheet"
tags:
  - "quick-reference"
  - "snmp"
  - "scanner"
---

# onesixtyone CHEATSHEET

## Quick Start

**Basic scan**: `onesixtyone -c community.txt 192.168.1.0/24`

**Single host**: `onesixtyone -c community.txt 192.168.1.100`

**Debug mode**: `onesixtyone -d -c community.txt 192.168.1.100`

**CSV output**: `onesixtyone -o results.csv -c community.txt 192.168.1.0/24`

**Silent mode**: `onesixtyone -s -c community.txt 192.168.1.0/24`

## Common Flags

| Flag | Description | Example |
|------|-------------|---------|
| `-c FILE` | Community string file | `onesixtyone -c strings.txt 192.168.1.100` |
| `-d` | Debug mode | `onesixtyone -d -c community.txt 192.168.1.100` |
| `-w NUM` | Wait time (ms) | `onesixtyone -w 5 -c community.txt 192.168.1.100` |
| `-i FILE` | Target list file | `onesixtyone -i targets.txt -c community.txt` |
| `-o FILE` | Output CSV file | `onesixtyone -o results.csv -c community.txt 192.168.1.100` |
| `-p PORT` | SNMP port | `onesixtyone -p 1161 -c community.txt 192.168.1.100` |
| `-s` | Silent mode | `onesixtyone -s -c community.txt 192.168.1.100` |
| `-n NUM` | Max hosts | `onesixtyone -n 50 -c community.txt 192.168.1.0/24` |
| `-t NUM` | Timeout (sec) | `onesixtyone -t 5 -c community.txt 192.168.1.100` |

## Examples

### Single Host
```bash
onesixtyone -c community.txt 192.168.1.100
```

### Network Range
```bash
onesixtyone -c community.txt 192.168.1.0/24
```

### Custom Community Strings
```bash
cat > strings.txt << 'EOF'
public
private
manager
admin
secret
EOF
onesixtyone -c strings.txt 192.168.1.100
```

### Target List
```bash
cat > targets.txt << 'EOF'
192.168.1.1
192.168.1.10
192.168.1.20
EOF
onesixtyone -i targets.txt -c community.txt
```

### Debug Output
```bash
onesixtyone -d -c community.txt 192.168.1.100
```

### Fast Scan
```bash
onesixtyone -w 5 -c community.txt 192.168.1.0/24
```

### Custom Port
```bash
onesixtyone -p 1161 -c community.txt 192.168.1.100
```

### Silent with CSV
```bash
onesixtyone -s -o results.csv -c community.txt 192.168.1.0/24
```

## Chaining

### Chain with snmpwalk
```bash
onesixtyone -c community.txt 192.168.1.0/24 | grep -E "^[0-9]" | while read host string; do
  echo "=== $host ==="
  snmpwalk -c "$string" -v1 "$host" 1.3.6.1.2.1.1
done
```

### Chain with snmp-check
```bash
onesixtyone -c community.txt 192.168.1.0/24 | grep -E "^[0-9]" | while read host string; do
  snmp-check -c "$string" "$host"
done
```

### Pipe to Report
```bash
onesixtyone -s -c community.txt 192.168.1.0/24 | tee found.txt | wc -l
```

## Troubleshooting

### No results
- Verify targets: `ping -c 2 192.168.1.100`
- Check SNMP: `snmpwalk -c public -v1 192.168.1.100 1.3.6.1.2.1.1`
- Test with echo: `echo "public" > test.txt && onesixtyone -c test.txt 192.168.1.100`

### Slow scanning
- Reduce wait: `onesixtyone -w 5 -c community.txt 192.168.1.100`
- Scan smaller range: `onesixtyone -c community.txt 192.168.1.0/24`

### Permission denied
- Run as root: `sudo onesixtyone -c community.txt 192.168.1.100`

### Connection refused
- Check firewall: `iptables -L -n | grep 161`
- Verify port: `nmap -sU -p 161 192.168.1.100`

### No output file
- Check write permissions: `ls -la /path/to/output/`
- Use absolute path: `onesixtyone -o /tmp/results.csv -c community.txt 192.168.1.100`
