---
title: "intrace Quick Reference"
description: "Quick reference guide for intrace TCP traceroute"
categories:
  - "network"
  - "cheatsheet"
tags:
  - "quick-reference"
  - "traceroute"
  - "tcp"
---

# intrace CHEATSHEET

## Quick Start

**Basic trace**: `sudo intrace 192.168.1.100`

**Custom port**: `sudo intrace -p 443 192.168.1.100`

**Verbose mode**: `sudo intrace -v 192.168.1.100`

**Extended mode**: `sudo intrace -x 192.168.1.100`

**Save output**: `sudo intrace 192.168.1.100 | tee trace.txt`

## Common Flags

| Flag | Description | Example |
|------|-------------|---------|
| `-i INTERFACE` | Network interface | `sudo intrace -i eth1 192.168.1.100` |
| `-s SOURCE_IP` | Source IP | `sudo intrace -s 192.168.1.50 192.168.1.100` |
| `-p PORT` | Dest port | `sudo intrace -p 443 192.168.1.100` |
| `-t TIMEOUT` | Timeout (sec) | `sudo intrace -t 5 192.168.1.100` |
| `-v` | Verbose | `sudo intrace -v 192.168.1.100` |
| `-n` | Numeric mode | `sudo intrace -n 192.168.1.100` |
| `-r NUM` | Retries | `sudo intrace -r 3 192.168.1.100` |
| `-w WAIT` | Wait time | `sudo intrace -w 1 192.168.1.100` |
| `-x` | Extended mode | `sudo intrace -x 192.168.1.100` |

## Examples

### Basic Trace
```bash
sudo intrace 192.168.1.100
```

### Custom Port
```bash
sudo intrace -p 443 192.168.1.100
```

### Verbose Output
```bash
sudo intrace -v 192.168.1.100
```

### Extended Mode
```bash
sudo intrace -x 192.168.1.100
```

### Custom Interface
```bash
sudo intrace -i eth1 192.168.1.100
```

### Save Results
```bash
sudo intrace 192.168.1.100 2>&1 | tee trace.txt
```

### Multiple Targets
```bash
for target in 192.168.1.1 192.168.1.100 192.168.1.200; do
  echo "=== $target ==="
  sudo intrace "$target"
done
```

### Custom Timing
```bash
sudo intrace -w 1 -r 3 -t 5 192.168.1.100
```

### Source IP Spoofing
```bash
sudo intrace -s 192.168.1.50 192.168.1.100
```

## Chaining

### Chain with 0trace
```bash
# Compare results from both tools
echo "=== 0trace ==="
sudo 0trace eth0 192.168.1.100

echo "=== intrace ==="
sudo intrace 192.168.1.100
```

### Chain with traceroute
```bash
# Try traditional traceroute first, fall back to intrace
traceroute -n 192.168.1.100 || sudo intrace -n 192.168.1.100
```

### Network Mapping
```bash
#!/bin/bash
for i in $(seq 1 254); do
  TARGET="192.168.1.$i"
  RESULT=$(sudo intrace -n "$TARGET" 2>/dev/null)
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
- Increase timeout: `sudo intrace -t 5 192.168.1.100`
- Check firewall blocking ICMP

### Permission denied
- Run as root: `sudo intrace 192.168.1.100`

### Interface errors
- List interfaces: `ip addr show`
- Use correct interface name

### Slow performance
- Reduce timeout: `sudo intrace -t 1 192.168.1.100`
- Limit retries: `sudo intrace -r 1 192.168.1.100`

### Connection errors
- Ensure TCP connection established
- Some systems detect session manipulation
