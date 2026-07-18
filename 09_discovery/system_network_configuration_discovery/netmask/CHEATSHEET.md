---
title: "netmask Quick Reference"
description: "Quick reference guide for netmask subnet calculator"
categories:
  - "network"
  - "cheatsheet"
tags:
  - "quick-reference"
  - "subnet"
  - "calculator"
---

# netmask CHEATSHEET

## Quick Start

**Basic calculation**: `netmask 192.168.1.0/24`

**Decimal mask**: `netmask -d 192.168.1.0/24`

**Wildcard mask**: `netmask -w 192.168.1.0/24`

**Hex mask**: `netmask -h 192.168.1.0/24`

**Network range**: `netmask -r 192.168.1.0/24`

**All formats**: `netmask -l 192.168.1.0/24`

## Common Flags

| Flag | Description | Example |
|------|-------------|---------|
| `-d` | Decimal format | `netmask -d 192.168.1.0/24` |
| `-c` | CIDR notation | `netmask -c 192.168.1.0/24` |
| `-w` | Wildcard mask | `netmask -w 192.168.1.0/24` |
| `-h` | Hexadecimal | `netmask -h 192.168.1.0/24` |
| `-r` | Range format | `netmask -r 192.168.1.0/24` |
| `-s` | Short output | `netmask -s 192.168.1.0/24` |
| `-l` | Long output | `netmask -l 192.168.1.0/24` |
| `-i` | Interactive mode | `netmask -i` |
| `-e` | Exact match | `netmask -e 192.168.1.0/24` |

## Examples

### Basic CIDR
```bash
netmask 192.168.1.0/24
```

### Decimal Mask
```bash
netmask -d 192.168.1.0/24
# Output: 255.255.255.0
```

### Wildcard Mask
```bash
netmask -w 192.168.1.0/24
# Output: 0.0.0.255
```

### Hex Mask
```bash
netmask -h 192.168.1.0/24
# Output: 0xFFFFFF00
```

### Network Range
```bash
netmask -r 192.168.1.0/24
# Output: 192.168.1.0 - 192.168.1.255
```

### /26 Subnet
```bash
netmask 192.168.1.0/26
```

### /30 Subnet
```bash
netmask 192.168.1.0/30
```

### Large Network
```bash
netmask 10.0.0.0/8
```

### Multiple Formats
```bash
netmask -d -w -h 192.168.1.0/24
```

### All Formats
```bash
netmask -l 192.168.1.0/24
```

## Chaining

### ACL Calculation
```bash
# Calculate wildcard for ACL
netmask -w 192.168.1.0/24
# Use: access-list 10 permit 192.168.1.0 0.0.0.255
```

### Subnet Planning
```bash
#!/bin/bash
for i in $(seq 0 7); do
  SUBNET="192.168.$i.0/24"
  echo "=== $SUBNET ==="
  netmask -r "$SUBNET"
done
```

### Network Documentation
```bash
netmask -l 192.168.1.0/24 > network_doc.txt
```

### Batch Processing
```bash
while read network; do
  echo "=== $network ==="
  netmask -d -w "$network"
done < networks.txt
```

### Compare Subnets
```bash
diff <(netmask -l 192.168.1.0/24) <(netmask -l 192.168.2.0/24)
```

## Troubleshooting

### Invalid input
- Use format: `IP/PREFIX` (e.g., 192.168.1.0/24)
- No spaces or special characters

### No output
- Check input: `netmask 192.168.1.0/24`
- Try long format: `netmask -l 192.168.1.0/24`

### Unexpected results
- Verify CIDR prefix
- Check network class

### Format confusion
- Use `-l` for all formats
- Review documentation

### Permission issues
- netmask typically doesn't require root
- Try without sudo
