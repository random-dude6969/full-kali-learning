---
title: "snmp-check Quick Reference"
description: "Quick reference guide for snmp-check SNMP enumeration tool"
categories:
  - "snmp"
  - "cheatsheet"
tags:
  - "quick-reference"
  - "snmp"
  - "enumeration"
---

# snmp-check CHEATSHEET

## Quick Start

**Basic enumeration**: `snmp-check -c public 192.168.1.100`

**SNMPv2c**: `snmp-check -c public -v 2c 192.168.1.100`

**SNMPv3**: `snmp-check -v 3 -u admin -l authPriv -a SHA -A pass -x AES -X pass 192.168.1.100`

**File output**: `snmp-check -c public -o results.txt 192.168.1.100`

**Compact mode**: `snmp-check -C -c public 192.168.1.100`

## Common Flags

| Flag | Description | Example |
|------|-------------|---------|
| `-c COMMUNITY` | Community string | `snmp-check -c public 192.168.1.100` |
| `-v VERSION` | SNMP version (1/2c/3) | `snmp-check -c public -v 2c 192.168.1.100` |
| `-p PORT` | SNMP port | `snmp-check -c public -p 1161 192.168.1.100` |
| `-t TIMEOUT` | Timeout (sec) | `snmp-check -c public -t 10 192.168.1.100` |
| `-d` | Debug mode | `snmp-check -d -c public 192.168.1.100` |
| `-C` | Compact output | `snmp-check -C -c public 192.168.1.100` |
| `--all` | All OID subtrees | `snmp-check --all -c public 192.168.1.100` |
| `-o FILE` | Output file | `snmp-check -c public -o out.txt 192.168.1.100` |
| `-w COMMUNITY` | Write community | `snmp-check -c public -w private 192.168.1.100` |

### SNMPv3 Flags

| Flag | Description | Example |
|------|-------------|---------|
| `-u USERNAME` | SNMPv3 username | `-u admin` |
| `-l LEVEL` | Security level | `-l authPriv` |
| `-a PROTO` | Auth protocol | `-a SHA` |
| `-A PASS` | Auth passphrase | `-A secret123` |
| `-x PROTO` | Privacy protocol | `-x AES` |
| `-X PASS` | Privacy passphrase | `-X private123` |

## Examples

### Basic Enumeration
```bash
snmp-check -c public 192.168.1.100
```

### SNMPv2c
```bash
snmp-check -c public -v 2c 192.168.1.100
```

### SNMPv3 AuthPriv
```bash
snmp-check -v 3 -u admin -l authPriv -a SHA -A mypass -x AES -X mypass 192.168.1.100
```

### Custom Port
```bash
snmp-check -c public -p 1161 192.168.1.100
```

### Extended Timeout
```bash
snmp-check -c public -t 15 192.168.1.100
```

### Save to File
```bash
snmp-check -c public -o device.txt 192.168.1.100
```

### Compact Output
```bash
snmp-check -C -c public 192.168.1.100
```

### All OIDs
```bash
snmp-check --all -c public 192.168.1.100
```

### Filter Output
```bash
snmp-check -c public 192.168.1.100 | grep -E "(System|Interfaces|IP)"
```

## Chaining

### Chain with braa/onesixtyone
```bash
# Find valid strings, then enumerate
onesixtyone -c community.txt 192.168.1.0/24 | grep -E "^[0-9]" | while read host string; do
  echo "=== $host ==="
  snmp-check -c "$string" "$host"
done
```

### Chain with snmpwalk
```bash
# Enumerate, then query specific OID
snmp-check -c public 192.168.1.100 > full_enum.txt
snmpwalk -c public -v1 192.168.1.100 1.3.6.1.2.1.25.4.2.1.2
```

### Batch Processing
```bash
for i in $(seq 1 10); do
  echo "=== 192.168.1.$i ==="
  snmp-check -c public 192.168.1.$i > "192.168.1.$i.txt"
done
```

## Troubleshooting

### No results
- Verify community: `snmpwalk -c public -v1 192.168.1.100 1.3.6.1.2.1.1`
- Check service: `systemctl status snmpd`
- Try different version: `snmp-check -c public -v 2c 192.168.1.100`

### Partial results
- Target may restrict OIDs
- Try `--all` flag
- Different SNMP versions

### Connection timeout
- Increase timeout: `snmp-check -c public -t 15 192.168.1.100`
- Check connectivity: `ping -c 2 192.168.1.100`
- Verify port: `nmap -sU -p 161 192.168.1.100`

### Permission denied
- Check community string access level
- For SNMPv3, verify credentials
- Try write community: `snmp-check -c public -w private 192.168.1.100`

### Garbled output
- Encoding issues
- Try different SNMP version
- Check device documentation for OID specifics
