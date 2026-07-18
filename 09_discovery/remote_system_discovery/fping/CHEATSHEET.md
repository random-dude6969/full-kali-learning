---
title: "fping Cheatsheet"
description: "Quick reference for fping parallel ping and host discovery"
mitre_tactic: discovery
mitre_tactic_id: TA0007
mitre_subcategory: remote_system_discovery
tags:
  - ping
  - parallel
  - cheatsheet
  - host-discovery
---

# fping Cheatsheet

## Quick Start

```bash
# Ping subnet
fping -a -g 192.168.1.0/24

# Ping from file
fping -a -f targets.txt

# Ping single host
fping 192.168.1.1
```

## Common Flags

| Flag | Description | Example |
|------|-------------|---------|
| `-g RANGE` | Generate range | `fping -g 192.168.1.0/24` |
| `-f FILE` | Read from file | `fping -f targets.txt` |
| `-a` | Alive hosts only | `fping -a -g 192.168.1.0/24` |
| `-u` | Unreachable only | `fping -u -g 192.168.1.0/24` |
| `-c COUNT` | Pings per target | `fping -c 3 -g 192.168.1.0/24` |
| `-t MS` | Timeout | `fping -t 2000 -g 192.168.1.0/24` |
| `-p MS` | Period | `fping -p 100 -g 192.168.1.0/24` |
| `-s` | Statistics | `fping -a -s -g 192.168.1.0/24` |
| `-S IP` | Source IP | `fping -S 10.0.0.1 -g 192.168.1.0/24` |
| `-n` | No DNS | `fping -n -a -g 192.168.1.0/24` |
| `-r COUNT` | Retries | `fping -r 2 -g 192.168.1.0/24` |
| `-i MS` | Interval | `fping -i 100 -g 192.168.1.0/24` |
| `-v` | Verbose | `fping -v -g 192.168.1.0/24` |

## Host Discovery

```bash
# Find alive hosts
fping -a -g 192.168.1.0/24

# Find unreachable hosts
fping -u -g 192.168.1.0/24

# Ping range format
fping -g 192.168.1.1 192.168.1.254

# With statistics
fping -a -s -g 192.168.1.0/24
```

## Examples

**Basic sweep:**
```bash
fping -a -g 192.168.1.0/24
```

**With retries and timeout:**
```bash
fping -a -c 3 -t 2000 -r 2 -g 192.168.1.0/24
```

**Pipe to nmap:**
```bash
fping -a -g 192.168.1.0/24 | xargs nmap -sV -T4
```

**Continuous monitoring:**
```bash
while true; do fping -a -g 192.168.1.0/24; sleep 60; done
```

## Chaining

```bash
# fping + nmap
fping -a -g 192.168.1.0/24 | xargs -P 10 -I {} nmap -sV -T4 {}

# fping + arping (dual layer)
fping -a -g 192.168.1.0/24 > icmp.txt
for ip in $(seq 1 254); do arping -c 1 -f 192.168.1.$ip 2>/dev/null && echo "192.168.1.$ip" >> arp.txt; done
comm -23 arp.txt icmp.txt  # ARP-only (ICMP blocked)

# fping + httpx
fping -a -g 192.168.1.0/24 | httpx -silent
```

## Troubleshooting

| Problem | Fix |
|---------|-----|
| "Operation not permitted" | Use `sudo` for broadcast/source IP |
| All unreachable | Check network: `ping -c 1 192.168.1.1` |
| No output | Verify target IPs, try `-v` verbose |
| Scan too slow | Reduce `-t` timeout, `-r` retries |
| Missing hosts | Increase `-t` timeout, add retries |
