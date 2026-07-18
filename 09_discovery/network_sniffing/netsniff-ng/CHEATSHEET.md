---
title: "netsniff-ng Cheatsheet"
description: "Quick reference guide for netsniff-ng high-performance network toolkit"
tool: netsniff-ng
category: network-sniffing
difficulty: intermediate
requires_root: true
---

# netsniff-ng Cheatsheet

## Quick Start

```bash
# Install netsniff-ng
sudo apt install netsniff-ng

# Capture packets
sudo netsniff-ng -i eth0 -o capture.pcap

# Capture with filter
sudo netsniff-ng -i eth0 -o capture.pcap -f "tcp port 80"

# Monitor flows
sudo flowtop -i eth0

# Generate traffic
sudo trafgen -o eth0 --cfg traffic.cfg -p 1000
```

## Capture Filters

```bash
# Basic capture
sudo netsniff-ng -i eth0 -o capture.pcap

# Filter by host
sudo netsniff-ng -i eth0 -o capture.pcap -f "host 192.168.1.100"

# Filter by port
sudo netsniff-ng -i eth0 -o capture.pcap -f "tcp port 80"
sudo netsniff-ng -i eth0 -o capture.pcap -f "udp port 53"

# Filter by protocol
sudo netsniff-ng -i eth0 -o capture.pcap -f "arp"
sudo netsniff-ng -i eth0 -o capture.pcap -f "icmp"

# Complex filters
sudo netsniff-ng -i eth0 -o capture.pcap -f "(tcp port 80 or tcp port 443) and host 192.168.1.100"
```

## Analysis

```bash
# Read pcap file
netsniff-ng -i capture.pcap

# Capture specific count
sudo netsniff-ng -i eth0 -o capture.pcap -c 1000

# Custom snap length
sudo netsniff-ng -i eth0 -o capture.pcap -s 96

# Silent mode
sudo netsniff-ng -i eth0 -o capture.pcap --silent
```

## Common Flags

| Flag | Description | Example |
|------|-------------|---------|
| `-i` | Interface | `netsniff-ng -i eth0` |
| `-o` | Output file | `netsniff-ng -o capture.pcap` |
| `-f` | BPF filter | `netsniff-ng -f "tcp port 80"` |
| `-c` | Packet count | `netsniff-ng -c 1000` |
| `-s` | Snap length | `netsniff-ng -s 96` |
| `--ring-size` | Ring buffer | `netsniff-ng --ring-size 4096` |
| `--fanout` | Multi-core | `netsniff-ng --fanout` |
| `--bundle` | Core count | `netsniff-ng --bundle 4` |
| `--cpu` | CPU affinity | `netsniff-ng --cpu 0` |
| `--silent` | No stats | `netsniff-ng --silent` |

## Tool-Specific Commands

### netsniff-ng (Capture)
```bash
sudo netsniff-ng -i eth0 -o capture.pcap
sudo netsniff-ng -i eth0 -o capture.pcap -f "tcp port 80"
```

### trafgen (Traffic Generator)
```bash
sudo trafgen -o eth0 --cfg traffic.cfg
sudo trafgen -o eth0 -p 1000 --ip-dst 192.168.1.100
```

### mausezahn (Packet Crafter)
```bash
sudo mausezahn -i eth0 -t arp "a ff:ff:ff:ff:ff:ff"
sudo mausezahn -i eth0 -t icmp -p "data"
```

### flowtop (Flow Monitor)
```bash
sudo flowtop -i eth0
sudo flowtop -i eth0 -f "tcp port 80"
```

### bpfc (BPF Compiler)
```bash
bpfc -f "tcp port 80"
bpfc -f "tcp port 80" -o filter.bpf
```

## Examples

### Basic Capture
```bash
sudo netsniff-ng -i eth0 -o capture.pcap
```

### High-Speed Capture
```bash
sudo netsniff-ng -i eth0 -o capture.pcap --ring-size 8192 --fanout --bundle 4
```

### Traffic Generation
```bash
sudo trafgen -o eth0 --cfg traffic.cfg -p 10000
```

### Flow Monitoring
```bash
sudo flowtop -i eth0 -t 5
```

## Chaining with Other Tools

```bash
# Capture and analyze
sudo netsniff-ng -i eth0 -o capture.pcap -c 1000
tcpdump -r capture.pcap -v

# Pipe to Wireshark
sudo netsniff-ng -i eth0 -o - | wireshark -k -

# Use with tshark
sudo netsniff-ng -i eth0 -o capture.pcap -c 1000
tshark -r capture.pcap -T fields -e ip.src -e ip.dst

# Process with flowtop
sudo flowtop -i eth0 -f "not arp" | grep -i "tcp"
```

## Quick Reference

### Target IPs
- Gateway: `192.168.1.1`
- Target: `192.168.1.100`
- Subnet: `192.168.1.0/24`

### Common Ports
- FTP: 21
- SSH: 22
- HTTP: 80
- HTTPS: 443
- DNS: 53
- MySQL: 3306

### BPF Keywords
- `host` - Filter by IP
- `net` - Filter by subnet
- `port` - Filter by port
- `src` - Source address
- `dst` - Destination address
- `tcp` - TCP protocol
- `udp` - UDP protocol
- `icmp` - ICMP protocol
- `arp` - ARP protocol

### Ring Buffer Sizes
- Small: 1024 pages (4MB)
- Medium: 4096 pages (16MB)
- Large: 8192 pages (32MB)
- Very Large: 16384 pages (64MB)
