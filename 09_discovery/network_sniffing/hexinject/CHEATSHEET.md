---
title: "hexinject Cheatsheet"
description: "Quick reference guide for hexinject raw packet injection and sniffing"
tool: hexinject
category: network-sniffing
difficulty: advanced
requires_root: true
---

# hexinject Cheatsheet

## Quick Start

```bash
# Install hexinject
sudo apt install hexinject

# Sniff packets
sudo hexinject -i eth0 -s

# Inject hex data
sudo hexinject -i eth0 -x "FF:FF:FF:FF:FF:FF:00:11:22:33:44:55:08:00"

# Inject from stdin
echo "FF:FF:FF:FF:FF:FF:00:11:22:33:44:55:08:00" | sudo hexinject -i eth0 -X
```

## Capture Filters

```bash
# Sniff all traffic
sudo hexinject -i eth0 -s

# Filter by host
sudo hexinject -i eth0 -s -f "host 192.168.1.100"

# Filter by port
sudo hexinject -i eth0 -s -f "tcp port 80"
sudo hexinject -i eth0 -s -f "udp port 53"

# Filter by protocol
sudo hexinject -i eth0 -s -f "arp"
sudo hexinject -i eth0 -s -f "icmp"

# Complex filters
sudo hexinject -i eth0 -s -f "host 192.168.1.100 and tcp port 443"
sudo hexinject -i eth0 -s -f "not arp and not icmp"
```

## Analysis

```bash
# Verbose output
sudo hexinject -i eth0 -s -v

# Color output
sudo hexinject -i eth0 -s -c

# Read from pcap
sudo hexinject -r capture.pcap -s

# Write to pcap
sudo hexinject -i eth0 -s -w output.pcap
```

## Common Flags

| Flag | Description | Example |
|------|-------------|---------|
| `-i` | Network interface | `hexinject -i eth0` |
| `-s` | Sniff mode | `hexinject -i eth0 -s` |
| `-x` | Inject hex | `hexinject -i eth0 -x "FF:FF:..."` |
| `-X` | Inject from stdin | `echo "..." \| hexinject -X` |
| `-f` | BPF filter | `hexinject -i eth0 -s -f "tcp port 80"` |
| `-r` | Read pcap | `hexinject -r capture.pcap` |
| `-w` | Write pcap | `hexinject -i eth0 -s -w out.pcap` |
| `-S` | Spoof MAC | `hexinject -i eth0 -S 00:11:22:33:44:55` |
| `-c` | Color output | `hexinject -i eth0 -s -c` |
| `-v` | Verbose | `hexinject -i eth0 -s -v` |
| `-q` | Quiet | `hexinject -i eth0 -s -q` |

## Packet Formats

### Ethernet Header (14 bytes)
```
FF:FF:FF:FF:FF:FF  # Destination MAC
00:11:22:33:44:55  # Source MAC
08:00              # EtherType (IPv4)
```

### IP Header (20 bytes)
```
45 00 00 28  # Version, Length, ID
00 01 00 00  # Flags, TTL, Protocol
40 06 00 00  # Checksum
C0 A8 01 01  # Source IP
C0 A8 01 64  # Destination IP
```

### TCP Header (20 bytes)
```
00 14  # Source Port
00 50  # Destination Port
00 00 00 00  # Sequence Number
00 00 00 00  # Acknowledgment
50 02 00 00  # Flags, Window
00 00 00 00  # Checksum, Urgent
```

### UDP Header (8 bytes)
```
00 35  # Source Port
00 35  # Destination Port
00 08  # Length
00 00  # Checksum
```

## Examples

### Basic Packet Injection
```bash
# ARP broadcast
sudo hexinject -i eth0 -x "FF:FF:FF:FF:FF:FF:00:11:22:33:44:55:08:06:0001:0800:0604:0001:00:11:22:33:44:55:C0:A8:01:01:00:00:00:00:00:00:C0:A8:01:64"

# ICMP ping
sudo hexinject -i eth0 -x "00:11:22:33:44:55:AA:BB:CC:DD:EE:FF:08:00:4500:003C:1C46:4000:4001:B766:C0A8:0164:C0A8:0101:0800:EC29:0001:0001:0000:0000:0000:0000:0000"

# TCP SYN
sudo hexinject -i eth0 -x "FF:FF:FF:FF:FF:FF:00:11:22:33:44:55:08:00:45000028:00010000:4006:0000:C0A8:0101:C0A8:0164:0014:0050:00000000:00000000:5002:0000:0000"
```

### Sniffing Examples
```bash
# Capture HTTP traffic
sudo hexinject -i eth0 -s -f "tcp port 80"

# Capture DNS
sudo hexinject -i eth0 -s -f "udp port 53"

# Capture ARP
sudo hexinject -i eth0 -s -f "arp"

# Capture with color
sudo hexinject -i eth0 -s -c -v
```

### MAC Spoofing
```bash
# Spoof source MAC
sudo hexinject -i eth0 -S 00:11:22:33:44:55 -x "00:11:22:33:44:55:FF:FF:FF:FF:FF:FF:08:00"

# Broadcast with spoofed MAC
sudo hexinject -i eth0 -S 00:11:22:33:44:55 -x "FF:FF:FF:FF:FF:FF:00:11:22:33:44:55:08:06:0001:0800:0604:0001:00:11:22:33:44:55:C0:A8:01:01:00:00:00:00:00:00:C0:A8:01:64"
```

## Chaining with Other Tools

```bash
# Capture with tcpdump, inject with hexinject
tcpdump -i eth0 -w capture.pcap -c 100 &
sleep 10
sudo hexinject -i eth0 -r capture.pcap -s

# Process pcap
tcpdump -r input.pcap -w filtered.pcap 'tcp port 80'
sudo hexinject -i eth0 -r filtered.pcap -s

# Use in automation
sudo hexinject -i eth0 -s -v | grep -i "ARP" > arp_log.txt

# Pipe to analysis
sudo hexinject -i eth0 -s -w - | wireshark -k -
```

## Quick Reference

### Target IPs
- Gateway: `192.168.1.1`
- Target: `192.168.1.100`
- Subnet: `192.168.1.0/24`

### Common EtherTypes
- `08:00` - IPv4
- `08:06` - ARP
- `86DD` - IPv6
- `8100` - VLAN

### Common IP Protocols
- `01` - ICMP
- `06` - TCP
- `11` - UDP
- `02` - IGMP

### TCP Flags
- `02` - SYN
- `10` - ACK
- `01` - FIN
- `04` - RST
- `18` - PSH+ACK
- `12` - SYN+ACK
