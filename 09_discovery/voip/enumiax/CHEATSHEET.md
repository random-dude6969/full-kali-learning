---
title: "enumiax Cheatsheet"
description: Quick reference for enumiax IAX2 enumeration tool
tool: enumiax
category: voip
difficulty: intermediate
---

# enumiax Cheatsheet

## Quick Start

```bash
# Basic enumeration
enumiax 192.168.1.100

# Verbose scan
enumiax -v -n 100-200 192.168.1.100

# Dictionary attack
enumiax -d /usr/share/wordlists/rockyou.txt 192.168.1.100
```

## SIP Methods

| Method | Description |
|--------|-------------|
| REGREQ | Registration request |
| REGACK | Registration acknowledgement |
| REGREJ | Registration rejection |
| NEW | Initiate new call |
| ACCEPT | Accept incoming call |
| REJECT | Reject incoming call |
| HANGUP | Terminate call |

## Common Flags

| Flag | Description | Example |
|------|-------------|---------|
| `-u` | Username/extension | `-u admin` |
| `-p` | Password | `-p pass123` |
| `-d` | Dictionary file | `-d wordlist.txt` |
| `-n` | Extension range | `-n 100-999` |
| `-v` | Verbose output | `-v` |
| `-m` | Max concurrent | `-m 10` |
| `-t` | Timeout (seconds) | `-t 5` |
| `-b` | Brute-force mode | `-b` |

## Examples

### Single User Test
```bash
enumiax -u admin -p password123 192.168.1.100
```

### Range Enumeration
```bash
enumiax -n 1000-2000 -m 10 -t 3 -v 192.168.1.100
```

### Dictionary Attack
```bash
enumiax -d /usr/share/wordlists/rockyou.txt -m 5 -v 192.168.1.100
```

### Save Results
```bash
enumiax -n 100-999 192.168.1.100 2>&1 | tee results.txt
```

## Chaining

```bash
# Discover IAX2 hosts first
nmap -sU -p 4569 192.168.1.0/24 -oG - | grep "4569/open" | awk '{print $2}' > targets.txt

# Enumerate all discovered hosts
for ip in $(cat targets.txt); do
    enumiax -n 100-999 -m 5 -t 3 $ip >> enumiax_full.txt
done
```

## Troubleshooting

| Problem | Fix |
|---------|-----|
| No response | Check firewall on UDP 4569 |
| Timeout | Increase `-t` value |
| Too many false positives | Reduce `-m` concurrency |
| Permission denied | No root needed; check network |

```bash
# Verify connectivity
nc -zvu 192.168.1.100 4569

# Capture for debugging
sudo tcpdump -i eth0 udp port 4569 -n -c 20
```
