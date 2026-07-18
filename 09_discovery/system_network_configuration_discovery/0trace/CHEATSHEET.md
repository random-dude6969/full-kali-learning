---
title: "0trace Quick Reference"
description: "Quick reference guide for 0trace firewall traversal traceroute"
categories:
  - "network"
  - "cheatsheet"
tags:
  - "quick-reference"
  - "traceroute"
  - "firewall"
---

# 0trace CHEATSHEET

## Quick Start

**Basic trace**: `sudo 0trace eth0 192.168.1.100`

**Custom port**: `sudo 0trace -p 443 eth0 192.168.1.100`

**Numeric mode**: `sudo 0trace -n eth0 192.168.1.100`

**Verbose**: `sudo 0trace -v eth0 192.168.1.100`

**Save output**: `sudo 0trace eth0 192.168.1.100 | tee trace.txt`

## Common Flags

| Flag | Description | Example |
|------|-------------|---------|
| `-i INTERFACE` | Network interface | `sudo 0trace -i eth0 192.168.1.100` |
| `-t TIMEOUT` | Timeout (sec) | `sudo 0trace -t 5 eth0 192.168.1.100` |
| `-l LENGTH` | Packet length | `sudo 0trace -l 1400 eth0 192.168.1.100` |
| `-n` | Numeric mode | `sudo 0trace -n eth0 192.168.1.100` |
| `-s PORT` | Source port | `sudo 0trace -s 54321 eth0 192.168.1.100` |
| `-p PORT` | Dest port | `sudo 0trace -p 443 eth0 192.168.1.100` |
| `-r NUM` | Retries | `sudo 0trace -r 3 eth0 192.168.1.100` |
| `-v` | Verbose | `sudo 0trace -v eth0 192.168.1.100` |

## Examples

### Basic Trace
```bash
sudo 0trace eth0 192.168.1.100
```

### Custom Port
```bash
sudo 0trace -p 443 eth0 192.168.1.100
```

### Numeric Mode
```bash
sudo 0trace -n eth0 192.168.1.100
```

### Extended Timeout
```bash
sudo 0trace -t 5 eth0 192.168.1.100
```

### Verbose Output
```bash
sudo 0trace -v eth0 192.168.1.100
```

### Save Results
```bash
sudo 0trace eth0 192.168.1.100 2>&1 | tee trace.txt
```

### Multiple Targets
```bash
for target in 192.168.1.1 192.168.1.100 192.168.1.200; do
  echo "=== $target ==="
  sudo 0trace eth0 "$target"
done
```

### Retry Config
```bash
sudo 0trace -r 3 -t 3 eth0 192.168.1.100
```

### Custom Packet Length
```bash
sudo 0trace -l 1400 eth0 192.168.1.100
```

## Chaining

### Chain with nmap
```bash
# Find open ports, then trace
nmap -p 80,443 192.168.1.0/24 -oG - | grep "open" | awk '{print $2}' | while read host; do
  echo "=== $host ==="
  sudo 0trace eth0 "$host"
done
```

### Chain with traceroute
```bash
# Try traditional traceroute first, fall back to 0trace
traceroute -n 192.168.1.100 || sudo 0trace -n eth0 192.168.1.100
```

### Network Mapping
```bash
#!/bin/bash
for i in $(seq 1 254); do
  TARGET="192.168.1.$i"
  RESULT=$(sudo 0trace -n eth0 "$TARGET" 2>/dev/null)
  if [ -n "$RESULT" ]; then
    echo "$TARGET: $(echo "$RESULT" | tail -1)"
  fi
done
```

## Troubleshooting

### No results
- Check interface: `ip link show`
- Verify target: `ping -c 2 192.168.1.100`
- Check port: `nmap -p 80,443 192.168.1.100`

### Incomplete trace
- Increase timeout: `sudo 0trace -t 5 eth0 192.168.1.100`
- Check firewall blocking ICMP

### Permission denied
- Run as root: `sudo 0trace eth0 192.168.1.100`

### Interface errors
- List interfaces: `ip addr show`
- Use correct interface name

### Slow performance
- Reduce timeout: `sudo 0trace -t 1 eth0 192.168.1.100`
- Limit retries: `sudo 0trace -r 1 eth0 192.168.1.100`
