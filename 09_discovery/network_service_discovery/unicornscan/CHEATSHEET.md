---
title: "unicornscan Cheatsheet"
tool: "unicornscan"
---

# unicornscan Cheatsheet

## Quick Start

**Basic TCP scan**:
```bash
sudo unicornscan -pT80 192.168.1.1
```

**Scan subnet**:
```bash
sudo unicornscan -pT80,443 192.168.1.0/24
```

**Scan with OS fingerprint**:
```bash
sudo unicornscan -pT80 -I 192.168.1.1
```

## Scan Types

| Mode | Flag | Use Case |
|------|------|----------|
| TCP Connect | `-pT` | Standard port scan |
| TCP SYN | `-pS` | Stealth scan |
| TCP ACK | `-pA` | Firewall mapping |
| TCP FIN | `-pF` | Stealth scan |
| TCP Xmas | `-pX` | Stealth scan |
| UDP | `-pU` | UDP services |
| ICMP | `-pI` | Host discovery |

## Common Flags

| Flag | Description | Example |
|------|-------------|---------|
| `-pT<ports>` | TCP connect scan | `unicornscan -pT80 192.168.1.1` |
| `-pS<ports>` | TCP SYN scan | `unicornscan -pS80 192.168.1.1` |
| `-pU<ports>` | UDP scan | `unicornscan -pU53 192.168.1.1` |
| `-pA<ports>` | TCP ACK scan | `unicornscan -pA80 192.168.1.1` |
| `-pF<ports>` | TCP FIN scan | `unicornscan -pF80 192.168.1.1` |
| `-pX<ports>` | TCP Xmas scan | `unicornscan -pX80 192.168.1.1` |
| `-I` | OS fingerprint | `unicornscan -pT80 -I 192.168.1.1` |
| `-r <pps>` | Rate (packets/sec) | `unicornscan -pT80 -r 1000 192.168.1.1` |
| `-T <ms>` | Timeout | `unicornscan -pT80 -T 5000 192.168.1.1` |
| `-R <n>` | Retries | `unicornscan -pT80 -R 5 192.168.1.1` |
| `-s <ip>` | Source IP | `unicornscan -pT80 -s 192.168.1.100 192.168.1.1` |
| `-v` | Verbose output | `unicornscan -pT80 -v 192.168.1.1` |
| `-l` | Log to file | `unicornscan -pT80 -l results.txt 192.168.1.1` |

## Output Options

| Flag | Description |
|------|-------------|
| `-l` | Log output to file |
| `-v` | Verbose output |
| `-V` | Very verbose output |
| `-q` | Quiet mode |
| `-I` | OS fingerprint |

## Examples

**Scan all TCP ports**:
```bash
sudo unicornscan -pT1-65535 192.168.1.1
```

**UDP scan**:
```bash
sudo unicornscan -pU53,123,161 192.168.1.0/24
```

**Stealth FIN scan**:
```bash
sudo unicornscan -pF1-1024 192.168.1.1
```

**Xmas scan**:
```bash
sudo unicornscan -pX1-1024 192.168.1.1
```

**High-speed scan**:
```bash
sudo unicornscan -pT80,443 -r 10000 192.168.1.0/24
```

**OS fingerprinting**:
```bash
sudo unicornscan -pT80 -I -v 192.168.1.1
```

**Scan from target file**:
```bash
sudo unicornscan -pT80,443 -iL targets.txt -l results.txt
```

**Custom source IP**:
```bash
sudo unicornscan -pT80 -s 192.168.1.100 192.168.1.1
```

**Firewall mapping**:
```bash
sudo unicornscan -pA1-1024 192.168.1.1
```

## Chaining

**With Nmap for deep analysis**:
```bash
# Phase 1: Fast discovery
sudo unicornscan -pT80,443,22 -r 1000 192.168.1.0/24 > discovery.txt

# Phase 2: Detailed scan
grep -oP '\d+\.\d+\.\d+\.\d+' discovery.txt | sort -u | \
    xargs -I {} nmap -sV -sC -p80,443,22 {} -oN deep_{}.txt
```

**With masscan for comparison**:
```bash
sudo unicornscan -pT80 192.168.1.0/24 > unicorn.txt
sudo masscan 192.168.1.0/24 -p80 -oG masscan.txt
```

**With grep for filtering**:
```bash
sudo unicornscan -pT80 192.168.1.0/24 | grep "open"
```

## Troubleshooting

**Permission errors**:
```bash
# Run with sudo
sudo unicornscan -pT80 192.168.1.1

# Set capabilities
sudo setcap cap_net_raw+ep /usr/bin/unicornscan
```

**Low scan rates**:
```bash
# Increase rate
sudo unicornscan -pT80 -r 10000 192.168.1.1

# Check network interface
sudo unicornscan -pT80 -i eth0 192.168.1.1
```

**Missing results**:
```bash
# Verify target specification
sudo unicornscan -pT80 -v 192.168.1.1

# Check timeout settings
sudo unicornscan -pT80 -T 10000 192.168.1.1
```

**Firewall issues**:
```bash
# Check iptables rules
sudo iptables -L -n | grep -E "50000|60000"

# Add rules for unicornscan
sudo iptables -A OUTPUT -p tcp --sport 40000:60000 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 80,443 -j ACCEPT
```
