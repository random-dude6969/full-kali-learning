---
title: "netdiscover Cheatsheet"
description: "Quick reference for netdiscover ARP host discovery"
mitre_tactic: discovery
mitre_tactic_id: TA0007
mitre_subcategory: remote_system_discovery
tags:
  - arp
  - scanner
  - cheatsheet
  - passive
---

# netdiscover Cheatsheet

## Quick Start

```bash
# Active scan
sudo netdiscover -i eth0 -r 192.168.1.0/24

# Passive monitoring
sudo netdiscover -i eth0 -p

# Interactive mode
sudo netdiscover -i eth0 --browse
```

## Common Flags

| Flag | Description | Example |
|------|-------------|---------|
| `-i IFACE` | Interface | `netdiscover -i eth0 -r 192.168.1.0/24` |
| `-r RANGE` | IP range | `netdiscover -r 192.168.1.0/24` |
| `-p` | Passive mode | `netdiscover -i eth0 -p` |
| `-s MAC` | Source MAC | `netdiscover -s 00:11:22:33:44:55 -r 192.168.1.0/24` |
| `-c COUNT` | Requests/host | `netdiscover -c 3 -r 192.168.1.0/24` |
| `-P` | Promiscuous | `netdiscover -P -i eth0 -r 192.168.1.0/24` |
| `-N` | No DNS | `netdiscover -N -i eth0 -r 192.168.1.0/24` |
| `-d MS` | Delay | `netdiscover -d 1000 -r 192.168.1.0/24` |
| `--write FILE` | Save results | `netdiscover --write out.txt -r 192.168.1.0/24` |
| `--browse` | Interactive UI | `netdiscover --browse -i eth0` |
| `--semi passive` | Semi-passive | `netdiscover --semi passive -r 192.168.1.0/24` |

## Host Discovery

```bash
# Active scan
sudo netdiscover -i eth0 -r 192.168.1.0/24

# Passive monitoring
sudo netdiscover -i eth0 -p

# Export to file
sudo netdiscover -i eth0 -r 192.168.1.0/24 --write hosts.txt

# Parse IPs only
sudo netdiscover -i eth0 -r 192.168.1.0/24 -N | awk 'NR>4{print $1}'
```

## Examples

**Basic scan:**
```bash
sudo netdiscover -i eth0 -r 192.168.1.0/24
```

**Passive monitoring:**
```bash
sudo netdiscover -i eth0 -p
```

**Stealthy scan (slow):**
```bash
sudo netdiscover -i eth0 -r 192.168.1.0/24 -d 2000 -c 1
```

**Interactive mode:**
```bash
sudo netdiscover -i eth0 --browse
```

## Chaining

```bash
# netdiscover + nmap
sudo netdiscover -i eth0 -r 192.168.1.0/24 -N | awk 'NR>4{print $1}' | \
    xargs -I {} nmap -sV -T4 {}

# netdiscover + arping
sudo netdiscover -i eth0 -r 192.168.1.0/24 -N | awk 'NR>4{print $1}' | \
    while read ip; do arping -c 1 -f "$ip" && echo "$ip verified"; done

# netdiscover + arp-scan
sudo netdiscover -i eth0 -r 192.168.1.0/24 -N > passive.txt
sudo arp-scan -I eth0 192.168.1.0/24 > active.txt
diff <(sort passive.txt) <(sort active.txt)
```

## Troubleshooting

| Problem | Fix |
|---------|-----|
| No interface specified | Use `-i eth0` |
| Permission denied | Run with `sudo` |
| No hosts found | Check interface up, try passive mode |
| Interface in use | Check: `sudo fuser eth0` |
| Slow discovery | Reduce `-d` delay, increase `-c` count |
| Missing hosts | Increase capture time, try active mode |
