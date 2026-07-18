---
title: "masscan Cheatsheet"
tool: "masscan"
---

# masscan Cheatsheet

## Quick Start

**Basic port scan**:
```bash
sudo masscan 192.168.1.1 -p80
```

**Scan subnet for common ports**:
```bash
sudo masscan 192.168.1.0/24 -p80,443,22
```

**High-speed scan with banners**:
```bash
sudo masscan 192.168.1.0/24 -p80 --banners --rate=10000
```

## Scan Types

| Mode | Command | Use Case |
|------|---------|----------|
| Basic | `masscan target -p80` | Quick port check |
| Banner Grab | `masscan target -p80 --banners` | Service identification |
| Full Range | `masscan target -p1-65535` | Complete port scan |
| Rate Limited | `masscan target -p80 --rate=100` | Stealthy scanning |

## Common Flags

| Flag | Description | Example |
|------|-------------|---------|
| `-p <ports>` | Ports to scan | `masscan -p80,443 192.168.1.0/24` |
| `--rate=<pps>` | Packets per second | `masscan --rate=100000 192.168.1.0/24` |
| `--banners` | Capture banners | `masscan -p80 --banners 192.168.1.1` |
| `-oX <file>` | XML output | `masscan -oX results.xml 192.168.1.0/24` |
| `-oG <file>` | Grepable output | `masscan -oG results.grep 192.168.1.0/24` |
| `-oJ <file>` | JSON output | `masscan -oJ results.json 192.168.1.0/24` |
| `--open` | Show only open ports | `masscan --open 192.168.1.0/24` |
| `--retries <n>` | Retry count | `masscan --retries=2 192.168.1.0/24` |
| `--excludefile <f>` | Exclude hosts | `masscan --excludefile exclude.txt 0.0.0.0/0` |
| `--source-ip <ip>` | Source IP | `masscan --source-ip=192.168.1.100 192.168.1.0/24` |
| `--adapter <iface>` | Network interface | `masscan --adapter=eth0 192.168.1.0/24` |

## Output Options

| Flag | Format | Use Case |
|------|--------|----------|
| `-oX` | XML | Nmap compatibility, parsing |
| `-oG` | Grepable | Quick text searching |
| `-oJ` | JSON | Programmatic processing |
| `-oL` | List | Simple host/port list |
| `--echo` | Config | Generate sample configuration |

## Examples

**Scan entire /16 network**:
```bash
sudo masscan 10.0.0.0/16 -p80,443 --rate=100000 -oX large_scan.xml
```

**Scan with rate limiting** (avoid IDS):
```bash
sudo masscan 192.168.1.0/24 -p80 --rate=100
```

**Distributed scanning**:
```bash
# Shard 1 of 4
sudo masscan 192.168.1.0/24 -p80 --shards=1/4 --seed=12345

# Shard 2 of 4
sudo masscan 192.168.1.0/24 -p80 --shards=2/4 --seed=12345
```

**Scan from target file**:
```bash
sudo masscan -iL targets.txt -p80,443 --banners
```

**Exclude specific hosts**:
```bash
sudo masscan 192.168.1.0/24 -p80 --excludefile exclude.txt
```

**Full internet scan** (use with caution):
```bash
sudo masscan 0.0.0.0/0 -p80 --rate=10000000 -oX internet.xml
```

## Chaining

**With Nmap for deep analysis**:
```bash
# Phase 1: Fast discovery
sudo masscan 192.168.1.0/24 -p80,443,22 -oX discovery.xml --open

# Phase 2: Detailed scan
grep -oP 'addr="\K[^"]+' discovery.xml | sort -u | \
    xargs -I {} nmap -sV -sC -p80,443,22 {} -oN deep_{}.txt
```

**With grep for filtering**:
```bash
sudo masscan 192.168.1.0/24 -p80 -oG results.grep
grep "80/open" results.grep
```

**With Python for processing**:
```python
#!/usr/bin/env python3
import xml.etree.ElementTree as ET
tree = ET.parse('results.xml')
for host in tree.findall('.//host'):
    for port in host.findall('.//port'):
        if port.find('state').get('state') == 'open':
            print(f"{host.find('address').get('addr')}:{port.get('portid')}")
```

## Troubleshooting

**Low scan rates**:
```bash
# Increase rate
sudo masscan --rate=100000 192.168.1.0/24

# Check network interface
sudo masscan --adapter=eth0 --rate=100000 192.168.1.0/24
```

**Missing results**:
```bash
# Verify target specification
sudo masscan --echo > config.txt
cat config.txt

# Check exclusion lists
sudo masscan 192.168.1.0/24 -p80 --excludefile="" 
```

**Permission errors**:
```bash
# Run with sudo
sudo masscan 192.168.1.0/24 -p80

# Set capabilities
sudo setcap cap_net_raw+ep /usr/bin/masscan
```

**Firewall issues**:
```bash
# Check iptables rules
sudo iptables -L -n | grep -E "50000|60000"

# Add rules for masscan
sudo iptables -A OUTPUT -p tcp --sport 40000:60000 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 80,443 -j ACCEPT
```
