---
title: "arping Cheatsheet"
description: "Quick reference for arping ARP ping commands and flags"
mitre_tactic: discovery
mitre_tactic_id: TA0007
mitre_subcategory: remote_system_discovery
tags:
  - arp
  - ping
  - cheatsheet
---

# arping Cheatsheet

## Quick Start

```bash
# Basic ARP ping
arping -c 4 192.168.1.1

# Specify interface
arping -c 4 -i eth0 192.168.1.1

# Quit on first reply
arping -f -c 1 192.168.1.1
```

## Common Flags

| Flag | Description | Example |
|------|-------------|---------|
| `-c COUNT` | Number of requests | `arping -c 10 192.168.1.1` |
| `-i IFACE` | Interface | `arping -i wlan0 192.168.1.1` |
| `-s IP` | Source IP (no ARP update) | `arping -s 10.0.0.1 192.168.1.1` |
| `-S IP` | Source IP (updates target ARP) | `arping -S 10.0.0.1 192.168.1.1` |
| `-f` | Quit on first reply | `arping -f 192.168.1.1` |
| `-D` | Duplicate detection | `arping -D -c 3 192.168.1.1` |
| `-q` | Quiet output | `arping -q -c 4 192.168.1.1` |
| `-b` | Broadcast mode | `arping -b -c 4 192.168.1.1` |
| `-W SECS` | Deadline (seconds) | `arping -W 5 192.168.1.1` |
| `-w USECS` | Wait between probes | `arping -w 200000 -c 4 192.168.1.1` |

## Host Discovery

```bash
# Scan subnet
for i in $(seq 1 254); do
    arping -c 1 -f -W 1 192.168.1.$i 2>/dev/null && echo "192.168.1.$i alive"
done

# Quick check
arping -c 1 -f 192.168.1.1 && echo "UP" || echo "DOWN"
```

## Examples

**Basic ping:**
```bash
arping -c 4 192.168.1.1
```

**Duplicate IP detection:**
```bash
arping -D -c 3 192.168.1.1
```

**Spoofed source (no ARP update):**
```bash
arping -s 192.168.1.100 -c 4 192.168.1.1
```

**Spoofed source (forces ARP update):**
```bash
arping -S 192.168.1.100 -c 4 192.168.1.1
```

## Chaining

```bash
# arping + arp-scan for full ARP discovery
arping -c 1 -f 192.168.1.1 && sudo arp-scan -I eth0 192.168.1.0/24

# arping + tcpdump for analysis
arping -c 4 192.168.1.1 &
sudo tcpdump -i eth0 arp -c 10
```

## Troubleshooting

| Problem | Fix |
|---------|-----|
| No responses | Check interface is up: `ip link show` |
| "Network unreachable" | Verify route: `ip route get 192.168.1.1` |
| Permission denied | Use `sudo` for spoofed source IPs |
| All timeouts | Check cable, try different interface |
| Slow responses | Increase `-W` timeout value |

## Quick Reference

**ARP vs ICMP:**
- ARP = Layer 2, local subnet only, bypasses ICMP blocks
- ICMP = Layer 3, cross-subnet, standard ping

**Source IP flags:**
- `-s IP`: Spoofed source, target does NOT update ARP cache
- `-S IP`: Spoofed source, target DOES update ARP cache

**Common patterns:**
```bash
# Quick existence check
arping -c 1 -f 192.168.1.1 && echo "UP" || echo "DOWN"

# Subnet sweep
for i in $(seq 1 254); do arping -c 1 -f 192.168.1.$i 2>/dev/null && echo "192.168.1.$i alive"; done

# Duplicate detection
arping -D -c 3 192.168.1.1
```
