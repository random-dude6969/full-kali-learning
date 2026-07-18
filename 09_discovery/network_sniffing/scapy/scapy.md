---
title: "scapy - Powerful Python Packet Manipulation Library"
description: "Comprehensive Python library for crafting, sending, receiving, and analyzing network packets with support for hundreds of protocols"
mitre_tactic: discovery
mitre_tactic_id: TA0007
mitre_subcategory: network_sniffing
categories:
  - packet-manipulation
  - python-library
  - network-security
  - protocol-analysis
  - security-research
tags:
  - scapy
  - python
  - packet-crafting
  - packet-injection
  - protocol-analysis
  - network-security
install_command: "pip3 install scapy"
version: "2.5.0"
github_repo: "https://github.com/secdev/scapy"
kali_tools_page: "https://www.kali.org/tools/scapy/"
difficulty_level: advanced
requires_root: true
gui_available: false
related_tools:
  - tcpdump
  - wireshark
  - hping3
  - netcat
  - nmap
---

# scapy - Powerful Python Packet Manipulation Library

## 1. Introduction

scapy is a powerful Python-based interactive packet manipulation library and tool. It can forge or decode packets, send them on the wire, capture them, and match requests and replies. scapy supports a wide range of protocols and can handle tasks like scanning, tracerouting, probing, unit tests, attacks, and network discovery.

**Primary Use Cases:**
- Packet crafting and injection
- Network scanning and enumeration
- Protocol fuzzing and testing
- Man-in-the-middle attacks
- Packet sniffing and analysis
- Network discovery and mapping
- Custom protocol implementation
- Security research and education

**Key Features:**
- **Protocol Support**: 200+ protocol layers
- **Packet Crafting**: Build any packet from scratch
- **Packet Sniffing**: Capture and analyze traffic
- **Interactive Mode**: Python shell for packet manipulation
- **Scripting**: Full Python integration
- **pcap Support**: Read/write pcap files
- **Scapy Shell**: Interactive packet exploration
- **Cross-Platform**: Linux, macOS, Windows

**Protocol Layers:**
- **Link Layer**: Ethernet, WiFi, PPP, LLC
- **Network Layer**: IP, IPv6, ICMP, ARP, RARP
- **Transport Layer**: TCP, UDP, SCTP
- **Application Layer**: HTTP, DNS, DHCP, FTP, SMTP, SSH, and more

## 2. Installation

### Standard Installation

```bash
# Install scapy via pip
pip3 install scapy

# Or install from Kali repository
sudo apt update
sudo apt install python3-scapy

# Verify installation
python3 -c "import scapy; print(scapy.__version__)"
```

### Installation from Source

```bash
# Clone repository
git clone https://github.com/secdev/scapy.git
cd scapy

# Install in development mode
pip3 install -e .

# Or install globally
sudo python3 setup.py install
```

### Verify Installation

```bash
# Check scapy is working
sudo scapy -c "version"

# Test basic functionality
sudo scapy -c "ls()"

# Check available protocols
sudo scapy -c "ls(IP)"
```

## 3. Core Concepts

### Packet Layers

scapy uses a layered approach to packet construction:
- **Stacking Layers**: Combine protocol layers using `/`
- **Field Access**: Access fields using dot notation
- **Automatic Fields**: Many fields are automatically calculated

**Example:**
```python
# Build an IP/TCP packet
packet = IP(dst="192.168.1.100") / TCP(dport=80, flags="S")

# Access fields
packet[IP].src
packet[TCP].dport
```

### Packet Operations

**Sending Packets:**
- `send()`: Send at Layer 3 (IP)
- `sendp()`: Send at Layer 2 (Ethernet)

**Receiving Packets:**
- `sr()`: Send and receive (Layer 3)
- `sr1()`: Send and receive one response
- `srp()`: Send and receive (Layer 2)

**Sniffing:**
- `sniff()`: Capture packets
- `rdpcap()`: Read pcap file
- `wrpcap()`: Write pcap file

### BPF Filters

scapy supports Berkeley Packet Filter expressions:
```python
sniff(filter="tcp port 80", count=10)
sniff(filter="host 192.168.1.100 and icmp")
```

### Packet Manipulation

**Modifying Packets:**
```python
# Clone and modify
new_packet = packet.copy()
new_packet[IP].dst = "192.168.1.200"

# Delete fields
del new_packet[TCP].chksum
```

## 4. Command Reference

### Interactive Mode

```bash
# Start scapy shell
sudo scapy

# Start with specific interface
sudo scapy -I eth0

# Run command and exit
sudo scapy -c "ls(IP)"

# Verbose mode
sudo scapy -V
```

### Python API

```python
# Import all
from scapy.all import *

# Or import specific modules
from scapy.layers.inet import IP, TCP, UDP, ICMP
from scapy.layers.l2 import Ether, ARP
from scapy.layers.dns import DNS, DNSQR
```

### Sending Functions

```python
# Send at Layer 3
send(IP(dst="192.168.1.100")/ICMP())

# Send at Layer 2
sendp(Ether(dst="ff:ff:ff:ff:ff:ff")/ARP(pdst="192.168.1.1"))

# Send and receive (Layer 3)
ans, unans = sr(IP(dst="192.168.1.1")/ICMP())

# Send and receive one (Layer 3)
response = sr1(IP(dst="192.168.1.1")/ICMP())

# Send and receive (Layer 2)
ans, unans = srp(Ether(dst="ff:ff:ff:ff:ff:ff")/ARP(pdst="192.168.1.1"))
```

### Sniffing Functions

```python
# Basic sniffing
packets = sniff(count=10)

# Sniff with filter
packets = sniff(filter="tcp port 80", count=10)

# Sniff on specific interface
packets = sniff(iface="eth0", count=10)

# Sniff with timeout
packets = sniff(timeout=10)

# Sniff with prn callback
packets = sniff(prn=lambda x: x.summary())
```

### Pcap Functions

```python
# Read pcap file
packets = rdpcap("capture.pcap")

# Write pcap file
wrpcap("output.pcap", packets)

# Read specific packet
packet = packets[0]
```

### Display Functions

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

## 5. Examples

### Basic Examples

```bash
# Start interactive mode
sudo scapy

# Simple ICMP ping
>>> send(IP(dst="192.168.1.1")/ICMP())

# TCP SYN scan
>>> sr1(IP(dst="192.168.1.1")/TCP(dport=80, flags="S"))

# ARP request
>>> srp(Ether(dst="ff:ff:ff:ff:ff:ff")/ARP(pdst="192.168.1.1"))

# DNS query
>>> sr1(IP(dst="8.8.8.8")/UDP(dport=53)/DNS(rd=1, qd=DNSQR(qname="example.com")))
```

### Intermediate Examples

```bash
# Ping sweep
>>> ans, unans = sr(IP(dst="192.168.1.0/24")/ICMP())
>>> ans.summary()

# TCP SYN scan on port range
>>> ans, unans = sr(IP(dst="192.168.1.100")/TCP(dport=(1,1024), flags="S"))

# OS fingerprinting
>>> ans, unans = sr(IP(dst="192.168.1.100")/TCP(dport=80, flags="S"))

# DHCP discover
>>> sendp(Ether(dst="ff:ff:ff:ff:ff:ff")/IP(src="0.0.0.0", dst="255.255.255.255")/UDP(sport=68, dport=67)/BOOTP(op=1)/DHCP(options=[("message-type","discover"), "end"]))

# VLAN hopping
>>> sendp(Ether(dst="ff:ff:ff:ff:ff:ff")/Dot1Q(vlan=100)/IP(dst="192.168.1.1")/ICMP())
```

### Advanced Examples

```bash
# Custom packet with specific fields
>>> packet = IP(dst="192.168.1.100", ttl=64, id=12345)/TCP(sport=12345, dport=80, flags="S", seq=1000)/Raw(load="GET / HTTP/1.1\r\nHost: example.com\r\n\r\n")
>>> send(packet)

# Packet manipulation script
>>> def process_packet(pkt):
...     if pkt.haslayer(TCP) and pkt[TCP].dport == 80:
...         print(f"HTTP packet from {pkt[IP].src}")
>>> sniff(filter="tcp port 80", prn=process_packet, count=100)

# VLAN tag injection
>>> packet = Ether(dst="ff:ff:ff:ff:ff:ff")/Dot1Q(vlan=100)/IP(dst="192.168.1.1")/ICMP()
>>> sendp(packet)

# Custom DNS spoofing
>>> def dns_spoof(pkt):
...     if pkt.haslayer(DNS) and pkt[DNS].qr == 0:
...         spoofed = IP(dst=pkt[IP].src, src=pkt[IP].dst)/UDP(dport=pkt[UDP].sport, sport=53)/DNS(id=pkt[DNS].id, qr=1, an=DNSRR(rrname=pkt[DNSQR].qname, rdata="192.168.1.100"))
...         send(spoofed, verbose=0)
>>> sniff(filter="udp port 53", prn=dns_spoof)
```

### Real-World Scenarios

```bash
# Network discovery
>>> ans, unans = srp(Ether(dst="ff:ff:ff:ff:ff:ff")/ARP(pdst="192.168.1.0/24"), timeout=2)
>>> for sent, received in ans:
...     print(f"{received.hwsrc} - {received.psrc}")

# Port scanning with version detection
>>> ans, unans = sr(IP(dst="192.168.1.100")/TCP(dport=[22,80,443], flags="S"))

# ARP cache poisoning
>>> send(ARP(op="who-has", pdst="192.168.1.100", psrc="192.168.1.1", hwdst="ff:ff:ff:ff:ff:ff"), count=5)

# Packet capture and analysis
>>> packets = sniff(filter="tcp port 80", count=100, timeout=30)
>>> wrpcap("http_capture.pcap", packets)
```

## 6. Advanced Usage

### Custom Protocol Layers

```python
# Define custom protocol
class CustomProtocol(Packet):
    name = "CustomProtocol"
    fields_desc = [
        ShortEnumField("type", 1, {1: "Request", 2: "Response"}),
        ShortField("length", None),
        StrLenField("data", "", length_of="length"),
    ]

# Use custom protocol
packet = IP(dst="192.168.1.100")/CustomProtocol(type=1, data="Hello")
```

### Scripting and Automation

```python
#!/usr/bin/env python3
# network_scanner.py

from scapy.all import *

def scan_network(network):
    """Scan network for live hosts"""
    ans, unans = srp(Ether(dst="ff:ff:ff:ff:ff:ff")/ARP(pdst=network), timeout=2, verbose=0)
    hosts = []
    for sent, received in ans:
        hosts.append({"mac": received.hwsrc, "ip": received.psrc})
    return hosts

def port_scan(target, ports):
    """Scan ports on target"""
    ans, unans = sr(IP(dst=target)/TCP(dport=ports, flags="S"), timeout=1, verbose=0)
    open_ports = []
    for sent, received in ans:
        if received[TCP].flags == 0x12:  # SYN-ACK
            open_ports.append(sent[TCP].dport)
    return open_ports

# Usage
if __name__ == "__main__":
    hosts = scan_network("192.168.1.0/24")
    print("Live hosts:")
    for host in hosts:
        print(f"  {host['ip']} ({host['mac']})")
```

### Integration with Other Tools

```python
# Read pcap from tcpdump
packets = rdpcap("capture.pcap")

# Filter packets
http_packets = [p for p in packets if p.haslayer(TCP) and p[TCP].dport == 80]

# Analyze
for pkt in http_packets:
    print(pkt.summary())

# Write filtered to pcap
wrpcap("http_only.pcap", http_packets)

# Process with tshark
import subprocess
result = subprocess.run(["tshark", "-r", "capture.pcap", "-T", "fields", "-e", "ip.src"], capture_output=True, text=True)
```

## 7. Detection & Defense

### Detection Methods

**Packet Analysis:**
```python
# Detect ARP spoofing
def detect_arp_spoof(pkt):
    if pkt.haslayer(ARP) and pkt[ARP].op == 2:
        # Check if MAC is known
        if pkt[ARP].hwsrc in known_macs:
            print(f"Possible ARP spoofing: {pkt[ARP].psrc} is {pkt[ARP].hwsrc}")

sniff(filter="arp", prn=detect_arp_spoof)

# Detect unusual packets
def detect_anomaly(pkt):
    if pkt.haslayer(TCP) and pkt[TCP].flags == 0x29:  # FIN+PSH+URG
        print(f"Anomalous packet from {pkt[IP].src}")

sniff(prn=detect_anomaly)
```

**IDS Integration:**
- Custom Scapy-based IDS
- Integration with Snort/Suricata
- Zeek scripts for protocol analysis

### Defense Strategies

**Network Security:**
```python
# Monitor for packet injection
def monitor_injection(pkt):
    if pkt.haslayer(Raw):
        if b"<script>" in pkt[Raw].load:
            print(f"Possible XSS attempt from {pkt[IP].src}")

sniff(filter="tcp port 80", prn=monitor_injection)
```

**Host-Based Protection:**
```python
# Detect port scanning
def detect_scan(pkt):
    if pkt.haslayer(TCP) and pkt[TCP].flags == 2:  # SYN
        scan_counts[pkt[IP].src] = scan_counts.get(pkt[IP].src, 0) + 1
        if scan_counts[pkt[IP].src] > 100:
            print(f"Port scan detected from {pkt[IP].src}")

scan_counts = {}
sniff(filter="tcp[tcpflags] == tcp-syn", prn=detect_scan)
```

## 8. Troubleshooting & FAQ

### Common Issues

**"No such device" Error:**
```python
# List interfaces
>>> get_if_list()

# Check interface status
>>> get_if_addr("eth0")

# Use specific interface
>>> sniff(iface="eth0", count=10)
```

**"Permission Denied" Error:**
```bash
# Run with sudo
sudo scapy

# Or in Python script
sudo python3 script.py
```

**Packets Not Captured:**
```python
# Verify BPF filter
>>> sniff(filter="tcp port 80", count=10, verbose=2)

# Check interface
>>> conf.iface

# Use specific interface
>>> sniff(iface="eth0", count=10)
```

**Slow Performance:**
```python
# Reduce packet count
>>> sniff(count=100)

# Use shorter timeout
>>> sniff(timeout=10)

# Filter specific traffic
>>> sniff(filter="tcp port 80", count=100)
```

### Performance Issues

**High CPU Usage:**
```python
# Use specific filters
>>> sniff(filter="host 192.168.1.100", count=100)

# Reduce verbose output
>>> sniff(count=100, verbose=0)

# Use timeout
>>> sniff(timeout=10)
```

**Memory Issues:**
```python
# Process packets in batches
>>> packets = sniff(count=1000)
>>> process_packets(packets)
>>> del packets

# Use callback function
>>> def process(pkt):
...     # Process each packet
...     pass
>>> sniff(prn=process, count=1000)
```

### FAQ

**Q: What is scapy used for?**
A: scapy is a Python library for packet manipulation, allowing crafting, sending, receiving, and analyzing network packets.

**Q: Does scapy require root privileges?**
A: Yes, for packet injection and sniffing. Use sudo or run as root.

**Q: Can scapy capture HTTPS traffic?**
A: Yes, it can capture encrypted packets but cannot decrypt them without additional tools.

**Q: Is scapy legal to use?**
A: Only on networks you own or have explicit authorization to test. Unauthorized use is illegal.

**Q: How does scapy differ from tcpdump?**
A: scapy is a Python library for packet manipulation, while tcpdump is a command-line tool for packet capture.

**Q: Can scapy be used for network scanning?**
A: Yes, scapy can perform network scanning, port scanning, and OS fingerprinting.

## Resources

### Official Documentation
- [scapy Documentation](https://scapy.readthedocs.io/)
- [scapy GitHub Repository](https://github.com/secdev/scapy)
- [scapy Interactive Mode](https://scapy.readthedocs.io/en/latest/usage.html#interactive-tutorial)

### Tutorials and Guides
- [scapy Tutorial](https://www.youtube.com/watch?v=QaJjJmMmKqQ)
- [Packet Manipulation with scapy](https://www.tutorialspoint.com/scapy/index.htm)
- [Network Security Testing](https://www.offensive-security.com)

### Related Tools
- **tcpdump**: Command-line packet analyzer
- **wireshark**: GUI protocol analyzer
- **hping3**: Network packet generator
- **netcat**: Network utility
- **nmap**: Network scanner

### Books and Resources
- *Violent Python* by TJ O'Connor
- *Black Hat Python* by Justin Seitz
- *Python Packet Programming* by Patrick Farley

### Community
- [scapy GitHub Issues](https://github.com/secdev/scapy/issues)
- [Kali Linux Forums](https://forums.kali.org/)
- [Reddit r/netsec](https://www.reddit.com/r/netsec/)
