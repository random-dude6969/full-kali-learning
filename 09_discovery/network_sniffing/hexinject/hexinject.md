---
title: "hexinject - Raw Packet Injector and Sniffer"
description: "Low-level packet injection and sniffing tool for crafting and injecting hex data directly into network interfaces"
mitre_tactic: discovery
mitre_tactic_id: TA0007
mitre_subcategory: network_sniffing
categories:
  - packet-injection
  - network-security
  - raw-packets
  - protocol-testing
  - security-research
tags:
  - hexinject
  - packet-injection
  - raw-packets
  - hex-data
  - network-debugging
  - protocol-testing
install_command: "sudo apt install hexinject"
version: "1.0"
github_repo: "https://github.com/raoulh/hexinject"
kali_tools_page: "https://www.kali.org/tools/hexinject/"
difficulty_level: advanced
requires_root: true
gui_available: false
related_tools:
  - scapy
  - tcpdump
  - hping3
  - nemesis
  - packit
---

# hexinject - Raw Packet Injector and Sniffer

## 1. Introduction

hexinject is a low-level packet injection and sniffing tool that allows direct manipulation of raw network packets using hexadecimal data. It provides the ability to craft custom packets, inject them into network interfaces, and capture traffic with precise control over packet content.

**Primary Use Cases:**
- Raw packet injection into network interfaces
- Packet crafting using hexadecimal data
- Network protocol testing and debugging
- Security research and vulnerability analysis
- Custom packet generation for fuzzing
- Packet capture with BPF filters
- MAC address spoofing
- Protocol implementation testing

**Key Features:**
- **Raw Packet Injection**: Send custom packets directly
- **Hex Data Input**: Craft packets using hex strings
- **Stdin Support**: Pipe hex data from other tools
- **BPF Filters**: Advanced packet filtering
- **MAC Spoofing**: Change source MAC addresses
- **Pcap Support**: Read/write pcap files
- **Color Output**: Terminal color coding
- **Verbose Mode**: Detailed packet information

**Use Cases in Security Testing:**
- Testing firewall rules with custom packets
- Exploiting protocol vulnerabilities
- Network stack testing
- Custom attack payload delivery
- Protocol implementation validation
- Fuzzing network services

## 2. Installation

### Standard Installation

```bash
# Install hexinject
sudo apt update
sudo apt install hexinject

# Verify installation
hexinject -h
```

### Installation from Source

```bash
# Install dependencies
sudo apt install build-essential libpcap-dev

# Clone repository
git clone https://github.com/raoulh/hexinject.git
cd hexinject

# Build and install
make
sudo make install
```

### Verify Installation

```bash
# Check installation
which hexinject

# Test basic functionality
sudo hexinject -h

# Check version
hexinject -V
```

## 3. Core Concepts

### Raw Packet Injection

Raw packet injection allows sending custom-crafted packets directly to a network interface. This bypasses higher-level network protocols and provides direct control over packet content.

**Injection Process:**
1. Craft packet in hexadecimal format
2. Specify target interface
3. Inject packet into network
4. Monitor response

### Hexadecimal Packet Format

Packets are specified in hexadecimal format:
- **Ethernet Header**: 14 bytes (destination MAC, source MAC, type)
- **IP Header**: 20 bytes minimum (version, length, flags, etc.)
- **Transport Header**: TCP/UDP header (ports, flags, etc.)
- **Payload**: Application data

**Example Packet Structure:**
```
FF:FF:FF:FF:FF:FF  # Destination MAC
00:11:22:33:44:55  # Source MAC
08:00              # EtherType (IPv4)
45:00:00:28...     # IP header
00:00:00:00...     # TCP/UDP header
48:45:4C:4C:4F     # Payload "HELLO"
```

### BPF Filter Expressions

Berkeley Packet Filter (BPF) syntax for precise packet filtering:
- **Protocol**: `tcp`, `udp`, `icmp`, `arp`
- **Host**: `host 192.168.1.1`
- **Port**: `port 80`, `src port 22`
- **Direction**: `src`, `dst`
- **Logical**: `and`, `or`, `not`

### MAC Address Spoofing

hexinject can modify the source MAC address:
- Bypass MAC-based filtering
- Test MAC authentication
- Anonymize traffic source
- Test network access control

## 4. Command Reference

### General Options

```bash
# Basic syntax
hexinject [options]

# Interface selection
-i <interface>          # Specify network interface

# Mode selection
-s                      # Sniff mode (capture packets)
-x <hex>                # Inject hex data
-X                      # Inject from stdin
```

### Injection Options

```bash
# Inject hex data
-x <hex_string>         # Inject hex packet
-X                      # Read hex from stdin

# Example injection
sudo hexinject -i eth0 -x "FF:FF:FF:FF:FF:FF:00:11:22:33:44:55:08:00:45000028..."
```

### Sniffing Options

```bash
# Sniff mode
-s                      # Enable sniffing

# BPF filter
-f <filter>             # Apply BPF filter

# Verbose output
-v                      # Verbose mode

# Color output
-c                      # Enable color
```

### Pcap Options

```bash
# Read from pcap
-r <pcapfile>           # Read pcap file

# Write to pcap
-w <pcapfile>           # Write to pcap file
```

### MAC Spoofing

```bash
# Spoof source MAC
-S <mac_address>        # Set source MAC address

# Example
sudo hexinject -i eth0 -S 00:11:22:33:44:55
```

### Output Options

```bash
# Verbose output
-v                      # Detailed packet info

# Color output
-c                      # Terminal colors

# Quiet mode
-q                      # Minimal output
```

## 5. Examples

### Basic Examples

```bash
# Sniff packets on interface
sudo hexinject -i eth0 -s

# Sniff with BPF filter
sudo hexinject -i eth0 -s -f "tcp port 80"

# Inject simple packet
sudo hexinject -i eth0 -x "FF:FF:FF:FF:FF:FF:00:11:22:33:44:55:08:00"

# Inject from stdin
echo "FF:FF:FF:FF:FF:FF:00:11:22:33:44:55:08:00" | sudo hexinject -i eth0 -X

# Read from pcap
sudo hexinject -r capture.pcap -s
```

### Intermediate Examples

```bash
# ARP request injection
sudo hexinject -i eth0 -x "FF:FF:FF:FF:FF:FF:00:11:22:33:44:55:08:06:0001:0800:0604:0001:00:11:22:33:44:55:C0:A8:01:01:00:00:00:00:00:00:C0:A8:01:64"

# ICMP ping injection
sudo hexinject -i eth0 -x "00:11:22:33:44:55:AA:BB:CC:DD:EE:FF:08:00:4500:003C:1C46:4000:4001:B766:C0A8:0164:C0A8:0101:0800:EC29:0001:0001:0000:0000:0000:0000:0000"

# MAC spoofing with injection
sudo hexinject -i eth0 -S 00:11:22:33:44:55 -x "00:11:22:33:44:55:AA:BB:CC:DD:EE:FF:08:00"

# Capture with color output
sudo hexinject -i eth0 -s -c -v

# Filter and capture specific traffic
sudo hexinject -i eth0 -s -f "host 192.168.1.100 and tcp port 443"
```

### Advanced Examples

```bash
# Custom TCP SYN packet
sudo hexinject -i eth0 -x "FF:FF:FF:FF:FF:FF:00:11:22:33:44:55:08:00:45000028:00010000:4006:0000:C0A8:0101:C0A8:0164:0014:0050:00000000:00000000:5002:0000:0000"

# Bulk injection from file
cat packets.hex | while read line; do sudo hexinject -i eth0 -x "$line"; done

# Automated packet generation
for i in $(seq 1 100); do
  sudo hexinject -i eth0 -x "FF:FF:FF:FF:FF:FF:00:11:22:33:44:55:08:00:45000028"
done

# Read and re-inject packets
sudo hexinject -r input.pcap -s -w output.pcap

# Protocol fuzzing
for payload in $(seq 1 255); do
  sudo hexinject -i eth0 -x "FF:FF:FF:FF:FF:FF:00:11:22:33:44:55:08:00:45000028:$(printf '%02x' $payload)"
done
```

### Real-World Scenarios

```bash
# Firewall testing
sudo hexinject -i eth0 -x "FF:FF:FF:FF:FF:FF:00:11:22:33:44:55:08:00:45000028:00010000:4006:0000:C0A8:0101:C0A8:0164:0014:0050:00000000:00000000:5002:0000:0000"

# Custom protocol testing
sudo hexinject -i eth0 -x "00:11:22:33:44:55:AA:BB:CC:DD:EE:FF:08:00:45000028:00010000:4011:0000:C0A8:0101:C0A8:0164:0035:0035:0008:0000"

# Network discovery
for ip in $(seq 1 254); do
  sudo hexinject -i eth0 -x "FF:FF:FF:FF:FF:FF:00:11:22:33:44:55:08:06:0001:0800:0604:0001:00:11:22:33:44:55:C0:A8:01:01:00:00:00:00:00:00:C0:A8:01:$(printf '%02x' $ip)"
done

# MAC address scanning
for mac in $(seq 1 100); do
  sudo hexinject -i eth0 -S "$(printf '00:11:22:33:44:%02x' $mac)" -x "FF:FF:FF:FF:FF:FF:00:11:22:33:44:$(printf '%02x' $mac):08:00"
done
```

## 6. Advanced Usage

### Custom Packet Crafting

```bash
# Create packet template
cat > packet.txt << 'EOF'
FF:FF:FF:FF:FF:FF
00:11:22:33:44:55
08:00
45 00 00 28 00 01 00 00 40 06 00 00
C0 A8 01 01 C0 A8 01 64
00 14 00 50 00 00 00 00 00 00 00 00
50 02 00 00 00 00
EOF

# Inject packet
cat packet.txt | sudo hexinject -i eth0 -X
```

### Scripting and Automation

```bash
#!/bin/bash
# hexinject automation script

INTERFACE="eth0"
TARGET="192.168.1.100"

# Function to create ARP packet
create_arp() {
  local target_ip=$1
  echo "FF:FF:FF:FF:FF:FF:00:11:22:33:44:55:08:06:0001:0800:0604:0001:00:11:22:33:44:55:C0:A8:01:01:00:00:00:00:00:00:C0:A8:$(echo $target_ip | cut -d'.' -f3-4 | tr '.' ':')"
}

# Send ARP requests
for i in $(seq 1 10); do
  sudo hexinject -i $INTERFACE -x "$(create_arp $TARGET)"
  sleep 0.1
done
```

### Integration with Other Tools

```bash
# Capture with tcpdump, inject with hexinject
tcpdump -i eth0 -w capture.pcap -c 100 &
sleep 10
sudo hexinject -i eth0 -r capture.pcap -s

# Process pcap and re-inject
tcpdump -r input.pcap -w output.pcap 'tcp port 80'
sudo hexinject -i eth0 -r output.pcap -s

# Use with nmap
nmap -sP 192.168.1.0/24
sudo hexinject -i eth0 -s -f "host 192.168.1.100"

# Pipe to analysis
sudo hexinject -i eth0 -s -v | grep -i "ARP"
```

## 7. Detection & Defense

### Detection Methods

**Packet Analysis:**
```bash
# Monitor for unusual packets
sudo tcpdump -i eth0 -n -v

# Detect MAC spoofing
arp-scan -l

# Monitor ARP traffic
sudo tcpdump -i eth0 arp -n

# Use Wireshark for analysis
wireshark -i eth0 -f "arp or icmp"
```

**IDS Signatures:**
- Custom Snort rules for hexinject patterns
- Suricata rules for raw packet injection
- Zeek scripts for protocol anomalies

### Defense Strategies

**Network Security:**
```bash
# Enable port security on switches
# switch(config-if)# switchport port-security
# switch(config-if)# switchport port-security maximum 1

# Implement 802.1X authentication
# switch(config)# dot1x system-auth-control

# Enable Dynamic ARP Inspection
# switch(config)# ip arp inspection vlan 10
```

**Host-Based Protection:**
```bash
# Monitor for ARP anomalies
sudo arpwatch -i eth0

# Install intrusion detection
sudo apt install snort

# Enable firewalls
sudo ufw enable
sudo ufw deny from 192.168.1.0/24
```

**Packet Filtering:**
```bash
# Filter suspicious packets
sudo iptables -A INPUT -m conntrack --ctstate INVALID -j DROP

# Block specific MAC addresses
sudo ebtables -A INPUT -s 00:11:22:33:44:55 -j DROP

# Enable rate limiting
sudo iptables -A INPUT -p tcp --dport 80 -m limit --limit 100/min -j ACCEPT
```

## 8. Troubleshooting & FAQ

### Common Issues

**"No such device" Error:**
```bash
# List interfaces
ip link show

# Check interface status
sudo ip link set eth0 up

# Enable promiscuous mode
sudo ip link set eth0 promisc on
```

**"Permission Denied" Error:**
```bash
# Run with sudo
sudo hexinject -i eth0 -s

# Check capabilities
sudo setcap cap_net_raw,cap_net_admin=eip /usr/bin/hexinject
```

**Packet Injection Fails:**
```bash
# Verify interface supports injection
sudo ip link show eth0

# Check for driver issues
dmesg | grep -i eth0

# Test with different interface
sudo hexinject -i eth1 -x "FF:FF:FF:FF:FF:FF:00:11:22:33:44:55:08:00"
```

**BPF Filter Not Working:**
```bash
# Test filter syntax
sudo hexinject -i eth0 -s -f "tcp port 80"

# Check filter with tcpdump
sudo tcpdump -i eth0 -f "tcp port 80"

# Simplify filter
sudo hexinject -i eth0 -s -f "tcp"
```

### Performance Issues

**High CPU Usage:**
```bash
# Use more specific filters
sudo hexinject -i eth0 -s -f "host 192.168.1.100"

# Limit capture count
sudo hexinject -i eth0 -s -c 1000

# Reduce verbose output
sudo hexinject -i eth0 -s -q
```

**Packet Loss:**
```bash
# Increase buffer size
sudo hexinject -i eth0 -s -B 4096

# Use faster interface
sudo hexinject -i eth1 -s

# Reduce system load
sudo systemctl stop [unnecessary-services]
```

### FAQ

**Q: What is hexinject used for?**
A: hexinject is used for raw packet injection and sniffing, allowing direct manipulation of network packets using hexadecimal data.

**Q: Does hexinject work on wireless networks?**
A: Yes, but the wireless interface must be in monitor mode and support packet injection.

**Q: Can hexinject capture HTTPS traffic?**
A: Yes, it can capture encrypted packets, but cannot decrypt them without additional tools.

**Q: Is hexinject legal to use?**
A: Only on networks you own or have explicit authorization to test. Unauthorized use is illegal.

**Q: How does hexinject differ from scapy?**
A: hexinject is a focused tool for raw packet injection, while scapy is a comprehensive Python library for packet manipulation.

**Q: Can hexinject generate packets automatically?**
A: Yes, through scripting and automation with bash or Python.

## Resources

### Official Documentation
- [hexinject GitHub Repository](https://github.com/raoulh/hexinject)
- [hexinject Manual Page](https://linux.die.net/man/8/hexinject)
- [Kali Linux hexinject Documentation](https://www.kali.org/tools/hexinject/)

### Tutorials and Guides
- [Packet Injection with hexinject](https://www.youtube.com/watch?v=QaJjJmMmKqQ)
- [Raw Packet Crafting Tutorial](https://www.tutorialspoint.com/hexinject/index.htm)
- [Network Security Testing](https://www.offensive-security.com)

### Related Tools
- **scapy**: Python packet manipulation library
- **tcpdump**: Command-line packet analyzer
- **hping3**: Network packet generator
- **nemesis**: Packet injection tool
- **packit**: Linux packet crafting tool

### Books and Resources
- *Network Security Assessment* by Chris McNab
- *Hacking: The Art of Exploitation* by Jon Erickson
- *TCP/IP Illustrated* by W. Richard Stevens

### Community
- [hexinject GitHub Issues](https://github.com/raoulh/hexinject/issues)
- [Kali Linux Forums](https://forums.kali.org/)
- [Reddit r/netsec](https://www.reddit.com/r/netsec/)
