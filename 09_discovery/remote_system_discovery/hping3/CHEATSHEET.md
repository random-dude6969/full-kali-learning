---
title: "hping3 Cheatsheet"
description: "Quick reference for hping3 packet crafting and network testing"
mitre_tactic: discovery
mitre_tactic_id: TA0007
mitre_subcategory: remote_system_discovery
tags:
  - tcp
  - packet-crafting
  - cheatsheet
  - firewall-testing
---

# hping3 Cheatsheet

## Quick Start

```bash
# ICMP ping
sudo hping3 -1 192.168.1.1

# SYN scan port 80
sudo hping3 -S 192.168.1.1 -p 80

# Traceroute on port 80
sudo hping3 -z -p 80 192.168.1.1

# Send 10 packets
sudo hping3 -S 192.168.1.1 -p 80 -c 10
```

## Common Flags

| Flag | Description | Example |
|------|-------------|---------|
| `-c COUNT` | Packet count | `hping3 -c 10 192.168.1.1` |
| `-i U/M` | Interval | `hping3 -i u100 192.168.1.1` |
| `-n` | No DNS | `hping3 -n 192.168.1.1` |
| `-q` | Quiet output | `hping3 -q -S 192.168.1.1 -p 80` |
| `-V` | Verbose | `hping3 -V -S 192.168.1.1 -p 80` |
| `-p PORT` | Port | `hping3 -S 192.168.1.1 -p 80` |
| `-S` | SYN flag | `hping3 -S 192.168.1.1 -p 80` |
| `-A` | ACK flag | `hping3 -A 192.168.1.1 -p 80` |
| `-F` | FIN flag | `hping3 -F 192.168.1.1 -p 80` |
| `-R` | RST flag | `hping3 -R 192.168.1.1 -p 80` |
| `-P` | PSH flag | `hping3 -P 192.168.1.1 -p 80` |
| `-U` | URG flag | `hping3 -U 192.168.1.1 -p 80` |
| `-W SIZE` | Window size | `hping3 -S -W 1024 192.168.1.1 -p 80` |
| `-s SIZE` | Packet size | `hping3 -S -s 1000 192.168.1.1 -p 80` |
| `-a IP` | Spoof source | `hping3 -S -a 10.0.0.1 192.168.1.1 -p 80` |
| `-z` | Traceroute | `hping3 -z -p 80 192.168.1.1` |
| `--flood` | Flood mode | `hping3 -S --flood 192.168.1.1 -p 80` |
| `--rand-source` | Random source | `hping3 -S --rand-source 192.168.1.1 -p 80` |
| `-1` | ICMP mode | `hping3 -1 192.168.1.1` |
| `-2` | UDP mode | `hping3 -2 -p 53 192.168.1.1` |

## Host Discovery

```bash
# ICMP ping
sudo hping3 -1 192.168.1.1

# TCP ping on port 80
sudo hping3 -S 192.168.1.1 -p 80 -c 1

# TCP ping on port 443
sudo hping3 -S 192.168.1.1 -p 443 -c 1

# Scan common ports
for port in 22 80 443; do
    sudo hping3 -S 192.168.1.1 -p $port -c 1 -n 2>&1 | grep -q "flags=SA" && echo "Port $port OPEN"
done
```

## Examples

**ICMP ping:**
```bash
sudo hping3 -1 192.168.1.1
```

**SYN scan:**
```bash
sudo hping3 -S 192.168.1.1 -p 80 -c 3
```

**ACK scan (firewall mapping):**
```bash
sudo hping3 -A 192.168.1.1 -p 80 -c 3
```

**FIN scan:**
```bash
sudo hping3 -F 192.168.1.1 -p 80 -c 3
```

**Traceroute:**
```bash
sudo hping3 -z -p 80 192.168.1.1
```

**Spoofed source:**
```bash
sudo hping3 -S -a 10.0.0.100 192.168.1.1 -p 80
```

## Chaining

```bash
# hping3 + tcpdump
sudo tcpdump -i eth0 host 192.168.1.1 &
sudo hping3 -S 192.168.1.1 -p 80 -c 10

# hping3 + nmap
sudo hping3 -S 192.168.1.1 -p 80 -c 1
nmap -sV -T4 192.168.1.1

# hping3 firewall test
for port in 21 22 23 25 53 80 443; do
    sudo hping3 -S 192.168.1.1 -p $port -c 1 -n 2>&1 | grep -q "flags=SA" && echo "$port OPEN" || echo "$port FILTERED"
done
```

## Troubleshooting

| Problem | Fix |
|---------|-----|
| "Raw socket access needed" | Use `sudo` |
| "No route to host" | Check routing: `ip route get 192.168.1.1` |
| No responses | Try different scan type, check firewall |
| Interface error | Specify with `-I eth0` |
| Slow scanning | Use `-i u100` for microsecond intervals |
| Packet loss | Increase interval: `-i m100` |
