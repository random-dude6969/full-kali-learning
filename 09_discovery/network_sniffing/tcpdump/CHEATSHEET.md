---
title: "tcpdump Cheatsheet"
description: "Quick reference guide for tcpdump command-line packet analyzer"
tool: tcpdump
category: network-sniffing
difficulty: beginner
requires_root: true
---

# tcpdump Cheatsheet

## Quick Start

```bash
# Install tcpdump
sudo apt install tcpdump

# Capture on interface
sudo tcpdump -i eth0

# Capture with count
sudo tcpdump -i eth0 -c 100

# Capture to file
sudo tcpdump -i eth0 -w capture.pcap

# Read from file
tcpdump -r capture.pcap
```

## Capture Filters

```bash
# Filter by host
sudo tcpdump -i eth0 'host 192.168.1.100'

# Filter by port
sudo tcpdump -i eth0 'port 80'
sudo tcpdump -i eth0 'port 443'

# Filter by protocol
sudo tcpdump -i eth0 'tcp'
sudo tcpdump -i eth0 'udp'
sudo tcpdump -i eth0 'icmp'
sudo tcpdump -i eth0 'arp'

# Filter by subnet
sudo tcpdump -i eth0 'net 192.168.1.0/24'

# Complex filters
sudo tcpdump -i eth0 'host 192.168.1.100 and port 80'
sudo tcpdump -i eth0 'tcp port 80 or tcp port 443'
sudo tcpdump -i eth0 'not port 22'
sudo tcpdump -i eth0 '(tcp port 80 or tcp port 443) and host 192.168.1.100'
```

## Analysis

```bash
# Verbose output
sudo tcpdump -i eth0 -v

# More verbose
sudo tcpdump -i eth0 -vv

# Most verbose
sudo tcpdump -i eth0 -vvv

# Hex dump
sudo tcpdump -i eth0 -X

# ASCII output
sudo tcpdump -i eth0 -A

# Hex and ASCII
sudo tcpdump -i eth0 -XX

# No name resolution
sudo tcpdump -i eth0 -nn

# With timestamps
sudo tcpdump -i eth0 -tttt
```

## Common Flags

| Flag | Description | Example |
|------|-------------|---------|
| `-i` | Interface | `tcpdump -i eth0` |
| `-c` | Packet count | `tcpdump -c 100` |
| `-n` | No DNS | `tcpdump -n` |
| `-nn` | No DNS/port | `tcpdump -nn` |
| `-w` | Write pcap | `tcpdump -w capture.pcap` |
| `-r` | Read pcap | `tcpdump -r capture.pcap` |
| `-v` | Verbose | `tcpdump -v` |
| `-vv` | More verbose | `tcpdump -vv` |
| `-vvv` | Most verbose | `tcpdump -vvv` |
| `-X` | Hex+ASCII | `tcpdump -X` |
| `-A` | ASCII only | `tcpdump -A` |
| `-s` | Snap length | `tcpdump -s 0` |
| `-e` | Link header | `tcpdump -e` |
| `-q` | Quiet | `tcpdump -q` |
| `-B` | Buffer size | `tcpdump -B 4096` |
| `-D` | List interfaces | `tcpdump -D` |

## BPF Filter Syntax

### Primitives
```bash
# Host filter
host 192.168.1.100
src host 192.168.1.100
dst host 192.168.1.100

# Network filter
net 192.168.1.0/24
src net 192.168.1.0/24
dst net 192.168.1.0/24

# Port filter
port 80
src port 22
dst port 443
portrange 8000-9000

# Protocol filter
tcp
udp
icmp
arp

# Direction
src
dst
src or dst

# Size
greater 1000
less 100
```

### Operators
```bash
# Logical AND
and (&&)

# Logical OR
or (||)

# Logical NOT
not (!)

# Grouping
()
```

### TCP Flags
```bash
# SYN flag
tcp[tcpflags] & (tcp-syn) != 0

# ACK flag
tcp[tcpflags] & (tcp-ack) != 0

# FIN flag
tcp[tcpflags] & (tcp-fin) != 0

# RST flag
tcp[tcpflags] & (tcp-rst) != 0

# PSH flag
tcp[tcpflags] & (tcp-push) != 0

# URG flag
tcp[tcpflags] & (tcp-urg) != 0
```

## Examples

### Basic Capture
```bash
# Capture on interface
sudo tcpdump -i eth0

# Capture 100 packets
sudo tcpdump -i eth0 -c 100

# Capture to file
sudo tcpdump -i eth0 -w capture.pcap
```

### Filtering Examples
```bash
# HTTP traffic
sudo tcpdump -i eth0 'tcp port 80'

# DNS traffic
sudo tcpdump -i eth0 'udp port 53'

# SSH traffic
sudo tcpdump -i eth0 'tcp port 22'

# All traffic from host
sudo tcpdump -i eth0 'src host 192.168.1.100'

# All traffic to host
sudo tcpdump -i eth0 'dst host 192.168.1.100'
```

### Analysis Examples
```bash
# Hex dump of HTTP
sudo tcpdump -i eth0 -X 'tcp port 80'

# ASCII output
sudo tcpdump -i eth0 -A 'tcp port 80'

# With timestamps
sudo tcpdump -i eth0 -tttt -c 10

# Verbose output
sudo tcpdump -i eth0 -vv 'host 192.168.1.100'
```

## Chaining with Other Tools

```bash
# Pipe to grep
sudo tcpdump -i eth0 -A 'tcp port 80' | grep -i "password"

# Pipe to Wireshark
sudo tcpdump -i eth0 -w - | wireshark -k -

# Pipe to tshark
sudo tcpdump -i eth0 -w - -c 1000 | tshark -r -

# Save and analyze
sudo tcpdump -i eth0 -w capture.pcap -c 1000
tcpdump -r capture.pcap -nn -c 10

# Use in scripts
sudo tcpdump -i eth0 -c 10 -w capture.pcap
if [ $? -eq 0 ]; then echo "Success"; fi
```

## Quick Reference

### Target IPs
- Gateway: `192.168.1.1`
- Target: `192.168.1.100`
- Subnet: `192.168.1.0/24`

### Common Ports
- FTP: 21
- SSH: 22
- Telnet: 23
- HTTP: 80
- HTTPS: 443
- DNS: 53
- DHCP: 67/68
- MySQL: 3306
- RDP: 3389

### Protocol Keywords
- `tcp` - TCP protocol
- `udp` - UDP protocol
- `icmp` - ICMP protocol
- `arp` - ARP protocol
- `host` - Filter by IP
- `net` - Filter by subnet
- `port` - Filter by port
- `src` - Source
- `dst` - Destination

### Snap Lengths
- `96` - Headers only
- `128` - Headers + small payload
- `256` - Headers + medium payload
- `512` - Headers + large payload
- `0` - Full packet (no truncation)
