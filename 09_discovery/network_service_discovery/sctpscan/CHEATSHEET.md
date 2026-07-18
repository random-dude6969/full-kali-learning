---
title: "sctpscan Cheatsheet"
tool: "sctpscan"
---

# sctpscan Cheatsheet

## Quick Start

**Basic SCTP scan**:
```bash
sudo sctpscan 192.168.1.1
```

**Scan telecom ports**:
```bash
sudo sctpscan -p2904,2905 192.168.1.0/24
```

**Scan Diameter ports**:
```bash
sudo sctpscan -p3868 192.168.1.0/24
```

## Scan Types

| Mode | Command | Use Case |
|------|---------|----------|
| Full Scan | `sctpscan target` | All ports |
| SS7/SIGUA | `sctpscan -p2904,2905 target` | Telecom signaling |
| Diameter | `sctpscan -p3868 target` | Authentication |
| H.248 | `sctpscan -p2944 target` | Media gateway |

## Common Flags

| Flag | Description | Example |
|------|-------------|---------|
| `-p <ports>` | Ports to scan | `sctpscan -p2904 192.168.1.1` |
| `-S <ip>` | Source IP address | `sctpscan -S 192.168.1.100 192.168.1.1` |
| `-T <ms>` | Timeout (ms) | `sctpscan -T 5000 192.168.1.1` |
| `-r <pps>` | Rate (packets/sec) | `sctpscan -r 100 192.168.1.0/24` |
| `-v` | Verbose output | `sctpscan -v 192.168.1.1` |
| `-vv` | Very verbose | `sctpscan -vv 192.168.1.1` |
| `--vtag=<hex>` | Verification tag | `sctpscan --vtag=0x12345678 192.168.1.1` |
| `--a-rwnd=<n>` | Receiver window | `sctpscan --a-rwnd=131072 192.168.1.1` |
| `-o <file>` | Output file | `sctpscan -o results.txt 192.168.1.0/24` |

## Output Options

| Flag | Description |
|------|-------------|
| `-o <file>` | Write results to file |
| `-v` | Verbose mode |
| `-vv` | Very verbose mode |
| `-q` | Quiet mode |

## Examples

**Scan SS7/SIGUA infrastructure**:
```bash
sudo sctpscan -p2904,2905 -v 10.0.1.0/24
```

**Scan with custom source**:
```bash
sudo sctpscan -S 10.0.1.100 -p2904 192.168.1.0/24
```

**Multihoming detection**:
```bash
sudo sctpscan -p2904 -vv 192.168.1.0/24
```

**Scan from target file**:
```bash
sudo sctpscan -l telecom_hosts.txt -p2904,3868
```

**Rate-limited scan**:
```bash
sudo sctpscan -p2904 -r 50 10.0.0.0/8
```

**Scan all telecom ports**:
```bash
sudo sctpscan -p2904,2905,3868,2944 -v 192.168.1.0/24
```

**Scan with output file**:
```bash
sudo sctpscan -p2904 -o sctp_audit.txt 172.16.0.0/12
```

## Chaining

**With Nmap for deep analysis**:
```bash
# Phase 1: Discover SCTP hosts
sudo sctpscan -p2904 192.168.1.0/24 | grep -oP '\d+\.\d+\.\d+\.\d+' > sctp_hosts.txt

# Phase 2: Nmap SCTP scripts
nmap -sY -p2904,2905,3868 -iL sctp_hosts.txt --script sctp-* -oN deep_sctp.txt
```

**With hping3 for verification**:
```bash
# Verify SCTP service found by sctpscan
sudo hping3 -S -p 2904 192.168.1.1
```

**With tcpdump for capture**:
```bash
# Capture SCTP traffic during scan
sudo tcpdump -i eth0 proto sctp -w sctp.pcap &
sudo sctpscan -p2904 192.168.1.0/24
```

## Troubleshooting

**SCTP kernel module not loaded**:
```bash
# Load SCTP module
sudo modprobe sctp

# Verify loading
lsmod | grep sctp

# Install SCTP tools
sudo apt install lksctp-tools
```

**Permission errors**:
```bash
# Run with sudo
sudo sctpscan 192.168.1.1 -p2904

# Set capabilities
sudo setcap cap_net_raw+ep /usr/bin/sctpscan
```

**No responses received**:
```bash
# Check if target is reachable
ping 192.168.1.1

# Try different ports
sudo sctpscan -p2904,3868,2944 192.168.1.1

# Increase timeout
sudo sctpscan -T 5000 -p2904 192.168.1.1

# Verify SCTP support
sudo sctpscan -v 192.168.1.1
```

**Firewall issues**:
```bash
# Check iptables rules
sudo iptables -L -n | grep -E "139|sctp"

# Add SCTP rules
sudo iptables -A INPUT -p sctp --dport 2904 -j ACCEPT
sudo iptables -A OUTPUT -p sctp --sport 2904 -j ACCEPT
```
