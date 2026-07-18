---
title: "ohrwurm Cheatsheet"
description: Quick reference for ohrwurm SIP fuzzing tool
tool: ohrwurm
category: voip
difficulty: intermediate
---

# ohrwurm Cheatsheet

## Quick Start

```bash
# Basic SIP fuzzing
sudo ohrwurm 192.168.1.100

# Verbose with packet count
sudo ohrwurm -n 500 -v 192.168.1.100

# Randomized fuzzing
sudo ohrwurm -n 1000 -r 192.168.1.100
```

## SIP Methods

| Method | Purpose |
|--------|---------|
| INVITE | Initiate call |
| REGISTER | Register endpoint |
| OPTIONS | Query capabilities |
| BYE | Terminate call |
| CANCEL | Cancel pending call |
| ACK | Acknowledge response |

## Common Flags

| Flag | Description | Example |
|------|-------------|---------|
| `-p` | SIP port | `-p 5060` |
| `-n` | Packet count | `-n 1000` |
| `-t` | Timeout | `-t 5` |
| `-v` | Verbose | `-v` |
| `-r` | Randomize | `-r` |
| `-d` | Delay (ms) | `-d 100` |

## Examples

### Basic Fuzzing
```bash
sudo ohrwurm -p 5060 -n 500 -v 192.168.1.100
```

### Stealth Mode
```bash
sudo ohrwurm -n 20 -d 2000 192.168.1.100
```

### Fuzz Specific Method
```bash
sudo ohrwurm --method INVITE -n 300 192.168.1.100
```

## Chaining

```bash
# Find SIP hosts, then fuzz
nmap -sU -p 5060 192.168.1.0/24 | grep "open" | awk '{print $2}' | while read ip; do
    sudo ohrwurm -n 200 $ip
done
```

## Troubleshooting

| Problem | Fix |
|---------|-----|
| Permission denied | Run with `sudo` |
| No response | Check firewall on UDP 5060 |
| Server crashes | May indicate successful fuzz |
| Tool not found | `sudo apt install ohrwurm` |

```bash
# Verify SIP connectivity
nc -zvu 192.168.1.100 5060

# Capture for analysis
sudo tcpdump -i eth0 host 192.168.1.100 and port 5060 -n
```
