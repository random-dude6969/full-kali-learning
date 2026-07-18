---
title: "arp-scan Cheatsheet"
description: "Quick reference for arp-scan ARP network scanning and OUI fingerprinting"
mitre_tactic: discovery
mitre_tactic_id: TA0007
mitre_subcategory: remote_system_discovery
tags:
  - arp
  - scanner
  - cheatsheet
  - fingerprinting
---

# arp-scan Cheatsheet

## Quick Start

```bash
# Scan local network (auto-detect)
sudo arp-scan -l

# Scan specific subnet
sudo arp-scan -I eth0 192.168.1.0/24

# Scan single host
sudo arp-scan -I eth0 192.168.1.1
```

## Common Flags

| Flag | Description | Example |
|------|-------------|---------|
| `-I IFACE` | Interface | `arp-scan -I eth0 192.168.1.0/24` |
| `-l` | Auto-detect local network | `arp-scan -l` |
| `-N` | No reverse DNS | `arp-scan -N -I eth0 192.168.1.0/24` |
| `-r RATE` | Packets per second | `arp-scan -r 100 192.168.1.0/24` |
| `-t MS` | Timeout (ms) | `arp-scan -t 500 192.168.1.0/24` |
| `-d MS` | Delay (ms) | `arp-scan -d 10 192.168.1.0/24` |
| `-b RATE` | Bandwidth limit | `arp-scan -b 1000000 192.168.1.0/24` |
| `-o FILE` | OUI file | `arp-scan -o oui.txt 192.168.1.0/24` |
| `--arpspa IP` | Source IP | `arp-scan --arpspa 10.0.0.1 192.168.1.0/24` |
| `--retry COUNT` | Retries | `arp-scan --retry 3 192.168.1.0/24` |
| `--ignoredups` | Skip duplicate MACs | `arp-scan --ignoredups 192.168.1.0/24` |

## Host Discovery

```bash
# Scan local subnet
sudo arp-scan -l

# Scan with no DNS
sudo arp-scan -I eth0 -N 192.168.1.0/24

# Multiple ranges
sudo arp-scan -I eth0 192.168.1.0/24 192.168.2.0/24

# Find specific vendor
sudo arp-scan -I eth0 192.168.1.0/24 | grep -i "vmware"
```

## Examples

**Basic scan:**
```bash
sudo arp-scan -I eth0 192.168.1.0/24
```

**Fast scan, no DNS:**
```bash
sudo arp-scan -I eth0 -N -r 200 192.168.1.0/24
```

**Export to file:**
```bash
sudo arp-scan -I eth0 -N 192.168.1.0/24 > scan_results.txt
```

**CSV format:**
```bash
sudo arp-scan -I eth0 -N 192.168.1.0/24 | awk 'NR>2{print $1","$2","$3}'
```

## Chaining

```bash
# arp-scan + nmap for deeper scan
sudo arp-scan -I eth0 -N 192.168.1.0/24 | awk '{print $1}' | xargs nmap -sV -T4

# arp-scan + arping verification
sudo arp-scan -I eth0 -N 192.168.1.0/24 | awk 'NR>2{print $1}' | while read ip; do
    arping -c 1 -f "$ip" && echo "$ip confirmed"
done

# Compare scans
diff <(sudo arp-scan -I eth0 -N 192.168.1.0/24 | awk '{print $2}' | sort) \
     <(sudo arp-scan -I eth0 -N 192.168.1.0/24 | awk '{print $2}' | sort)
```

## Troubleshooting

| Problem | Fix |
|---------|-----|
| No interface specified | Use `-I eth0` or `-l` for auto-detect |
| Permission denied | Run with `sudo` |
| No responses | Check interface: `ip link show eth0` |
| OUI shows Unknown | Update OUI database from Wireshark repo |
| Slow scan | Increase `-r` rate and `-t` timeout |
| Packet loss | Use `--retry 3` and lower `-r` rate |
