---
title: "scapy Cheatsheet"
description: "Quick reference guide for scapy Python packet manipulation library"
tool: scapy
category: network-sniffing
difficulty: advanced
requires_root: true
---

# scapy Cheatsheet

## Quick Start

```bash
# Start scapy interactive mode
sudo scapy

# Run command
sudo scapy -c "ls(IP)"
```

## Python API

```python
# Import all
from scapy.all import *

# Import specific layers
from scapy.layers.inet import IP, TCP, UDP, ICMP
from scapy.layers.l2 import Ether, ARP
from scapy.layers.dns import DNS, DNSQR
```

## Packet Construction

```python
# IP packet
IP(dst="192.168.1.100")

# TCP packet
TCP(dport=80, flags="S")

# UDP packet
UDP(dport=53)

# ICMP packet
ICMP()

# Layered packet
IP(dst="192.168.1.100")/TCP(dport=80, flags="S")

# With payload
IP(dst="192.168.1.100")/TCP(dport=80)/Raw(load="GET / HTTP/1.1\r\n\r\n")
```

## Sending Functions

```python
# Layer 3 send
send(IP(dst="192.168.1.100")/ICMP())

# Layer 2 send
sendp(Ether(dst="ff:ff:ff:ff:ff:ff")/ARP(pdst="192.168.1.1"))

# Send and receive (L3)
ans, unans = sr(IP(dst="192.168.1.1")/ICMP())

# Send and receive one (L3)
response = sr1(IP(dst="192.168.1.1")/ICMP())

# Send and receive (L2)
ans, unans = srp(Ether(dst="ff:ff:ff:ff:ff:ff")/ARP(pdst="192.168.1.1"))
```

## Sniffing Functions

```python
# Basic sniffing
packets = sniff(count=10)

# With filter
packets = sniff(filter="tcp port 80", count=10)

# Specific interface
packets = sniff(iface="eth0", count=10)

# With timeout
packets = sniff(timeout=10)

# With callback
packets = sniff(prn=lambda x: x.summary())

# Store packets
packets = sniff(count=10, store=True)
```

## Pcap Functions

```python
# Read pcap
packets = rdpcap("capture.pcap")

# Write pcap
wrpcap("output.pcap", packets)

# Read specific packet
packet = packets[0]
```

## Display Functions

```python
# Summary
packet.summary()

# Verbose
packet.show()

# Hex dump
hexdump(packet)

# Raw bytes
bytes(packet)

# Packet list summary
packets.summary()
```

## Common Flags

| Function | Description | Example |
|----------|-------------|---------|
| `send()` | Send L3 packet | `send(IP(dst="192.168.1.1")/ICMP())` |
| `sendp()` | Send L2 packet | `sendp(Ether()/ARP())` |
| `sr()` | Send/receive L3 | `sr(IP(dst="192.168.1.1")/ICMP())` |
| `sr1()` | Send/receive one L3 | `sr1(IP(dst="192.168.1.1")/ICMP())` |
| `srp()` | Send/receive L2 | `srp(Ether()/ARP())` |
| `sniff()` | Capture packets | `sniff(count=10)` |
| `rdpcap()` | Read pcap | `rdpcap("file.pcap")` |
| `wrpcap()` | Write pcap | `wrpcap("file.pcap", pkts)` |

## Examples

### Basic Operations
```python
# ICMP ping
send(IP(dst="192.168.1.100")/ICMP())

# TCP SYN
sr1(IP(dst="192.168.1.100")/TCP(dport=80, flags="S"))

# ARP request
srp(Ether(dst="ff:ff:ff:ff:ff:ff")/ARP(pdst="192.168.1.1"))

# DNS query
sr1(IP(dst="8.8.8.8")/UDP(dport=53)/DNS(rd=1, qd=DNSQR(qname="example.com")))
```

### Scanning
```python
# Ping sweep
ans, unans = sr(IP(dst="192.168.1.0/24")/ICMP(), timeout=2)
ans.summary()

# TCP SYN scan
ans, unans = sr(IP(dst="192.168.1.100")/TCP(dport=(1,1024), flags="S"))

# Port scan with flags
ans, unans = sr(IP(dst="192.168.1.100")/TCP(dport=[22,80,443], flags="S"))
```

### Packet Manipulation
```python
# Clone packet
new_pkt = pkt.copy()

# Modify fields
new_pkt[IP].dst = "192.168.1.200"

# Delete fields
del new_pkt[TCP].chksum

# Add payload
new_pkt = new_pkt/Raw(load="data")
```

## Chaining with Other Tools

```python
# Read pcap from tcpdump
packets = rdpcap("capture.pcap")

# Filter packets
http_pkts = [p for p in packets if p.haslayer(TCP) and p[TCP].dport == 80]

# Write filtered
wrpcap("http_only.pcap", http_pkts)

# Process with tshark
import subprocess
result = subprocess.run(["tshark", "-r", "capture.pcap"], capture_output=True, text=True)
```

## Quick Reference

### Target IPs
- Gateway: `192.168.1.1`
- Target: `192.168.1.100`
- Subnet: `192.168.1.0/24`

### TCP Flags
- `S` - SYN
- `A` - ACK
- `F` - FIN
- `R` - RST
- `P` - PSH
- `SA` - SYN-ACK
- `FA` - FIN-ACK
- `RA` - RST-ACK

### Common Protocols
- `IP` - Internet Protocol
- `TCP` - Transmission Control Protocol
- `UDP` - User Datagram Protocol
- `ICMP` - Internet Control Message Protocol
- `ARP` - Address Resolution Protocol
- `DNS` - Domain Name System
- `HTTP` - Hypertext Transfer Protocol

### Layer Classes
- `Ether` - Ethernet
- `Dot1Q` - VLAN
- `IP` - IPv4
- `IPv6` - IPv6
- `TCP` - TCP
- `UDP` - UDP
- `ICMP` - ICMP
- `ARP` - ARP
- `DNS` - DNS
- `Raw` - Raw payload
