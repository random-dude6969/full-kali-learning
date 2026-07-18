# Unicornscan — Cheatsheet

## Quick Start

```bash
# Basic SYN scan
sudo unicornscan 192.168.1.1

# Scan specific ports
sudo unicornscan 192.168.1.1:80,443,22

# High-speed full port scan
sudo unicornscan -r 5000 192.168.1.1:1-65535
```

## Essential Commands

| Command | Description |
|---------|-------------|
| `sudo unicornscan target` | Default SYN scan |
| `sudo unicornscan target:80,443` | Specific ports |
| `sudo unicornscan target:1-1024` | Port range |
| `sudo unicornscan target:*` | All ports |
| `sudo unicornscan -U target:53,161` | UDP scan |
| `sudo unicornscan -r 5000 target` | High-speed scan |
| `sudo unicornscan -mT target:80` | TCP module |
| `sudo unicornscan -mU target:161` | UDP module |

## Scan Types

| Flag | Type | Use Case |
|------|------|----------|
| `-mT` | TCP SYN | Default, fast |
| `-mU` | UDP | DNS, SNMP, etc. |
| `-sT` | TCP Connect | No root needed |
| `-sA` | ACK | Firewall mapping |
| `-sI` | Idle | Stealth scan |

## Flag Reference

| Flag | Description |
|------|-------------|
| `-r` | Packets per second |
| `-w` | Wait time (ms) |
| `-d` | Delay between probes |
| `-m` | Scan module (T/U) |
| `-U` | UDP scan |
| `-p` | Port specification |
| `-v` | Verbose output |
| `-vv` | Very verbose |
| `-l` | Log to file |
| `-L` | Log level (0-5) |
| `-e` | Network interface |
| `-f` | Fragment packets |
| `-n` | No DNS resolution |
| `-D` | Debug mode |
| `-h` | Help |
| `-V` | Version |

## Port Formats

| Format | Example |
|--------|---------|
| Single port | `target:80` |
| Multiple | `target:80,443,22` |
| Range | `target:1-1024` |
| All | `target:*` or `target:all` |
| UDP | `-mU target:53,161` |

## Speed Profiles

```bash
# Conservative (stealthy)
sudo unicornscan -r 10 -d 100 target:1-1024

# Normal
sudo unicornscan -r 1000 target:1-1024

# Fast
sudo unicornscan -r 5000 target:1-1024

# Aggressive
sudo unicornscan -r 10000 target:*
```

## Automation

```bash
# Network sweep
sudo unicornscan -r 5000 -mT "192.168.1.0/24:80,443" 2>/dev/null | \
    grep "open" | awk '{print $1}' | sort -u > live_hosts.txt

# Full port scan pipeline
sudo unicornscan -r 10000 -mT "target:*" 2>/dev/null | \
    grep "open" | sort -t: -k2 -n > open_ports.txt

# Scan and pipe to Nmap
sudo unicornscan -r 5000 -mT "target:*" 2>/dev/null | \
    grep "open" | awk '{print $1}' | sort -u | \
    xargs -I{} nmap -sV -p- {}
```

## Workflow Integration

```bash
# Unicornscan → Nmap pipeline
sudo unicornscan -r 3000 -mT "10.0.0.0/24:*" 2>/dev/null | \
    grep "open" | awk '{print $1}' | sort -u > hosts.txt && \
    nmap -sV -sC -iL hosts.txt -oN detailed_scan.txt

# Quick recon script
TARGET=$1
sudo unicornscan -r 5000 -mT "$TARGET:*" 2>/dev/null | \
    grep "open" | tee "unicorn_$TARGET.txt"
nmap -sV $(grep -oE "([0-9]{1,3}\.){3}[0-9]{1,3}" "unicorn_$TARGET.txt" | \
    sort -u | head -1) -p $(grep -oE ":[0-9]+" "unicorn_$TARGET.txt" | \
    tr -d ':' | sort -n | tr '\n' ',' | sed 's/,$//')
```
