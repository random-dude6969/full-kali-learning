---
title: "p0f Cheatsheet"
description: "Quick reference for p0f passive OS fingerprinting"
mitre_tactic: discovery
mitre_tactic_id: TA0007
mitre_subcategory: remote_system_discovery
tags:
  - os-detection
  - passive
  - cheatsheet
  - fingerprinting
---

# p0f Cheatsheet

## Quick Start

```bash
# Listen on interface
sudo p0f -i eth0

# Read from pcap
sudo p0f -r capture.pcap

# SYN only, specific port
sudo p0f -i eth0 -S -P 443
```

## Common Flags

| Flag | Description | Example |
|------|-------------|---------|
| `-i IFACE` | Interface | `p0f -i eth0` |
| `-f FILE` | Custom fingerprint DB | `p0f -f custom.fp -i eth0` |
| `-r FILE` | Read pcap | `p0f -r capture.pcap` |
| `-o FILE` | Output file | `p0f -i eth0 -o log.txt` |
| `-p` | Promiscuous mode | `p0f -p -i eth0` |
| `-S` | SYN packets only | `p0f -i eth0 -S` |
| `-P PORT` | Filter by port | `p0f -i eth0 -P 443` |
| `-Q` | Quiet mode | `p0f -i eth0 -Q` |
| `-v` | Verbose | `p0f -i eth0 -v` |
| `-d` | Daemon mode | `p0f -i eth0 -d` |
| `-l` | Log to stdout | `p0f -i eth0 -l` |
| `--lookup` | Look up signature | `p0f --lookup '8:64:0:*:mss*20,10:0'` |

## Host Discovery

```bash
# Passive OS detection
sudo p0f -i eth0 -v

# SYN-only detection
sudo p0f -i eth0 -S -v

# Analyze pcap for OSes
sudo p0f -r capture.pcap -v

# Extract unique OS fingerprints
sudo p0f -r capture.pcap -Q 2>&1 | grep -oP 'OS: \K.*' | sort -u
```

## Examples

**Live monitoring:**
```bash
sudo p0f -i eth0 -v
```

**Analyze pcap:**
```bash
sudo p0f -r traffic.pcap -v
```

**Monitor HTTPS visitors:**
```bash
sudo p0f -i eth0 -S -P 443 -v
```

**Daemon with logging:**
```bash
sudo p0f -i eth0 -d -o /var/log/p0f.log
```

## Chaining

```bash
# p0f + tcpdump
sudo tcpdump -i eth0 -w capture.pcap -c 10000
sudo p0f -r capture.pcap -v

# p0f + nmap comparison
sudo p0f -i eth0 -S -P 443 -Q > passive.txt
nmap -O -sV 192.168.1.50 > active.txt

# p0f + Wireshark
sudo tcpdump -i eth0 -w full.pcap
# Open full.pcap in Wireshark
# Run p0f for OS identification
sudo p0f -r full.pcap -v
```

## Troubleshooting

| Problem | Fix |
|---------|-----|
| "No suitable interface" | Check: `ip -br link show` |
| "Permission denied" | Use `sudo` |
| No fingerprints | Check traffic: `sudo tcpdump -i eth0 -c 10` |
| High CPU | Use `-Q` quiet mode |
| Missed packets | Enable `-p` promiscuous mode |
| Wrong OS detection | Compare with nmap -O, check NAT |
