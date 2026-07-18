---
title: "tcpdump - The Classic Command-Line Packet Analyzer"
description: "Powerful command-line packet capture and analysis tool with BPF filter support for network debugging and security analysis"
mitre_tactic: discovery
mitre_tactic_id: TA0007
mitre_subcategory: network_sniffing
categories:
  - packet-capture
  - network-analysis
  - command-line
  - debugging
  - security-analysis
tags:
  - tcpdump
  - packet-capture
  - bpf-filters
  - pcap
  - command-line
  - network-analysis
install_command: "sudo apt install tcpdump"
version: "4.99.4"
github_repo: ""
kali_tools_page: "https://www.kali.org/tools/tcpdump/"
difficulty_level: beginner
requires_root: true
gui_available: false
related_tools:
  - wireshark
  - tcpflow
  - netsniff-ng
  - tshark
  - ngrep
---

# tcpdump - The Classic Command-Line Packet Analyzer

## 1. Introduction

tcpdump is the classic and most widely-used command-line packet analyzer. It allows users to capture and display TCP/IP and other packets being transmitted or received over a network. tcpdump is a powerful tool for network debugging, analysis, and security auditing.

**Primary Use Cases:**
- Network traffic capture and analysis
- Packet filtering with BPF expressions
- Network debugging and troubleshooting
- Security auditing and forensics
- Protocol analysis and debugging
- Traffic monitoring and statistics
- Pcap file creation and analysis

**Key Features:**
- **BPF Filters**: Powerful packet filtering syntax
- **Pcap Support**: Read and write pcap files
- **Verbose Output**: Multiple verbosity levels
- **Protocol Decoding**: Supports 300+ protocols
- **Cross-Platform**: Linux, macOS, BSD, Windows
- **Scripting**: Pipe output to other tools
- **Remote Capture**: Capture on remote systems

**Why tcpdump Matters:**
- Default tool on most Unix systems
- Lightweight and efficient
- Scriptable and automatable
- Foundation for other tools (Wireshark reads tcpdump files)
- Essential for network troubleshooting

## 2. Installation

### Standard Installation

```bash
# Install tcpdump
sudo apt update
sudo apt install tcpdump

# Verify installation
tcpdump --version

# Check capabilities
which tcpdump
```

### Installation from Source

```bash
# Install dependencies
sudo apt install build-essential libpcap-dev

# Download source
wget https://www.tcpdump.org/release/tcpdump-4.99.4.tar.gz
tar -xzf tcpdump-4.99.4.tar.gz
cd tcpdump-4.99.4

# Build and install
./configure
make
sudo make install
```

### Verify Installation

```bash
# Check version
tcpdump --version

# List interfaces
tcpdump -D

# Test capture
sudo tcpdump -i eth0 -c 5
```

## 3. Core Concepts

### BPF Filter Expressions

Berkeley Packet Filter (BPF) is the foundation of tcpdump's filtering:
- **Primitives**: Basic filter elements
- **Operators**: Combine primitives
- **Qualifiers**: Modify primitives

**Filter Primitives:**
- `host <ip>` - Filter by host
- `net <subnet>` - Filter by network
- `port <port>` - Filter by port
- `src` / `dst` - Direction qualifier
- `tcp` / `udp` / `icmp` - Protocol qualifier

**Filter Operators:**
- `and` / `&&` - Logical AND
- `or` / `||` - Logical OR
- `not` / `!` - Logical NOT
- `()` - Grouping

### Pcap Files

tcpdump uses the pcap (packet capture) file format:
- **Standard Format**: Used by Wireshark, tcpdump, and many tools
- **Headers**: Global header + packet headers
- **Timestamps**: Microsecond precision
- **Metadata**: Capture length, original length

### Packet Decoding

tcpdump decodes protocol headers:
- **Link Layer**: Ethernet, PPP, WiFi
- **Network Layer**: IP, IPv6, ICMP, ARP
- **Transport Layer**: TCP, UDP, SCTP
- **Application Layer**: HTTP, DNS, DHCP (limited)

### Output Formats

- **Default**: Human-readable ASCII
- **Verbose**: Detailed packet info
- **Hex/ASCII**: Raw packet data
- **Fields**: Extract specific fields (with -T)

## 4. Command Reference

### Basic Options

```bash
# Basic syntax
tcpdump [options] [filter expression]

# Interface selection
-i <interface>          # Specify network interface
-i any                  # Capture on all interfaces

# Count limit
-c <count>              # Stop after count packets

# Name resolution
-n                      # Don't resolve hostnames
-nn                     # Don't resolve hostnames or port names
```

### Output Options

```bash
# Write to file
-w <filename>           # Write packets to pcap file

# Read from file
-r <filename>           # Read packets from pcap file

# Verbose output
-v                      # Verbose
-vv                     # More verbose
-vvv                    # Most verbose

# Print options
-A                      # Print ASCII
-X                      # Print hex and ASCII
-e                      # Print link-layer header
-q                      # Quiet/less verbose
```

### Capture Options

```bash
# Snap length
-s <length>             # Snapshot length (bytes)
-s 0                    # Full packet (no truncation)

# Buffer size
-B <buffer_size>        # Kernel buffer size (bytes)

# File rotation
-C <file_size>          # Rotate file at size (MB)
-W <count>              # Limit number of files
-G <seconds>            # Rotate file every N seconds
```

### Time Options

```bash
# Timestamps
-t                      # No timestamps
-tttt                   # Human-readable timestamps
-ttttt                  # Seconds since epoch

# Time precision
--time-precision=us     # Microsecond precision
--time-precision=ns     # Nanosecond precision
```

### Display Options

```bash
# Print packet count
-q                      # Quiet (less output)

# Print packets
-e                      # Print link-layer header
-x                      # Print hex
-xx                     # Print hex with link-layer
-X                      # Print hex and ASCII
-XX                     # Print hex/ASCII with link-layer

# Protocol specific
-tcp                    # Decode TCP
-udp                     # Decode UDP
-icmp                    # Decode ICMP
```

## 5. Examples

### Basic Examples

```bash
# Capture on default interface
sudo tcpdump -i eth0

# Capture 100 packets
sudo tcpdump -i eth0 -c 100

# Capture without name resolution
sudo tcpdump -i eth0 -nn

# Capture all interfaces
sudo tcpdump -i any

# Capture and write to file
sudo tcpdump -i eth0 -w capture.pcap

# Read from file
tcpdump -r capture.pcap
```

### Intermediate Examples

```bash
# Filter by host
sudo tcpdump -i eth0 -nn host 192.168.1.100

# Filter by port
sudo tcpdump -i eth0 -nn port 80

# Filter by protocol
sudo tcpdump -i eth0 -nn tcp
sudo tcpdump -i eth0 -nn udp
sudo tcpdump -i eth0 -nn icmp

# Complex filters
sudo tcpdump -i eth0 -nn 'host 192.168.1.100 and port 80'
sudo tcpdump -i eth0 -nn 'tcp port 80 or tcp port 443'
sudo tcpdump -i eth0 -nn 'not port 22'

# Verbose output
sudo tcpdump -i eth0 -nn -vv
```

### Advanced Examples

```bash
# Capture with hex dump
sudo tcpdump -i eth0 -nn -X -c 10

# Capture with ASCII
sudo tcpdump -i eth0 -nn -A -c 10 'tcp port 80'

# Capture with timestamps
sudo tcpdump -i eth0 -nn -tttt -c 10

# Rotate files by size
sudo tcpdump -i eth0 -w capture.pcap -C 100 -W 10

# Rotate files by time
sudo tcpdump -i eth0 -w capture.pcap -G 3600 -W 24

# Capture with full packets
sudo tcpdump -i eth0 -w capture.pcap -s 0

# Remote capture via SSH
ssh user@remote 'tcpdump -i eth0 -w -' | tcpdump -r -
```

### Real-World Scenarios

```bash
# Network debugging
sudo tcpdump -i eth0 -nn -vv 'host 192.168.1.100'

# HTTP traffic analysis
sudo tcpdump -i eth0 -nn -A 'tcp port 80' | grep -i "http"

# DNS monitoring
sudo tcpdump -i eth0 -nn -vv 'udp port 53'

# DHCP monitoring
sudo tcpdump -i eth0 -nn -vv 'udp port 67 or udp port 68'

# ARP monitoring
sudo tcpdump -i eth0 -nn -e 'arp'

# Security audit
sudo tcpdump -i eth0 -nn -w security.pcap 'not arp and not icmp'
```

## 6. Advanced Usage

### Complex BPF Filters

```bash
# Filter by subnet
sudo tcpdump -i eth0 -nn 'net 192.168.1.0/24'

# Filter by port range
sudo tcpdump -i eth0 -nn 'portrange 8000-9000'

# Filter by TCP flags
sudo tcpdump -i eth0 -nn 'tcp[tcpflags] & (tcp-syn) != 0'
sudo tcpdump -i eth0 -nn 'tcp[tcpflags] & (tcp-rst) != 0'

# Filter by packet size
sudo tcpdump -i eth0 -nn 'greater 1000'
sudo tcpdump -i eth0 -nn 'less 100'

# Complex expressions
sudo tcpdump -i eth0 -nn '(tcp port 80 or tcp port 443) and host 192.168.1.100'
```

### Scripting and Automation

```bash
#!/bin/bash
# Automated capture script

INTERFACE="eth0"
OUTPUT_DIR="/var/log/captures"
DURATION=3600

mkdir -p $OUTPUT_DIR

# Start capture with rotation
sudo tcpdump -i $INTERFACE \
  -w $OUTPUT_DIR/capture_%Y%m%d_%H%M%S.pcap \
  -G 3600 \
  -W 24 \
  -s 0 \
  -nn &

TCPDUMP_PID=$!

# Wait for duration
sleep $DURATION

# Stop capture
kill $TCPDUMP_PID

# Analyze captures
for file in $OUTPUT_DIR/*.pcap; do
  echo "Analyzing $file"
  tcpdump -r $file -nn -c 10
done
```

### Integration with Other Tools

```bash
# Pipe to grep
sudo tcpdump -i eth0 -nn -A 'tcp port 80' | grep -i "password"

# Pipe to Wireshark
sudo tcpdump -i eth0 -w - | wireshark -k -

# Process with tshark
sudo tcpdump -i eth0 -w - -c 1000 | tshark -r - -T fields -e ip.src -e ip.dst

# Use in scripts
sudo tcpdump -i eth0 -nn -c 10 -w capture.pcap
if [ $? -eq 0 ]; then
  echo "Capture successful"
fi
```

## 7. Detection & Defense

### Detection Methods

**Monitor for Capture:**
```bash
# Check for promiscuous mode
cat /sys/class/net/eth0/flags

# Monitor interface statistics
ifconfig eth0 | grep -i "RX\|TX"

# Check for tcpdump processes
ps aux | grep tcpdump

# Monitor network load
iftop -i eth0
```

**IDS Signatures:**
- Snort rules for packet capture detection
- Suricata rules for tcpdump patterns
- Zeek scripts for monitoring anomalies

### Defense Strategies

**Network Security:**
```bash
# Enable port security
# switch(config-if)# switchport port-security

# Implement VLANs
# switch(config)# vlan 10

# Enable 802.1X
# switch(config)# dot1x system-auth-control
```

**Host-Based Protection:**
```bash
# Monitor for capture
sudo auditctl -w /usr/sbin/tcpdump -p rwxa

# Enable logging
sudo logger -t security "tcpdump detected"

# Check capabilities
getcap /usr/sbin/tcpdump
```

**Encrypted Communications:**
- Use SSH instead of Telnet
- Use HTTPS instead of HTTP
- Implement VPN for sensitive traffic
- Use encrypted protocols

## 8. Troubleshooting & FAQ

### Common Issues

**"No such device" Error:**
```bash
# List interfaces
tcpdump -D

# Check interface status
ip link show eth0

# Enable interface
sudo ip link set eth0 up
```

**"Permission Denied" Error:**
```bash
# Run with sudo
sudo tcpdump -i eth0

# Check capabilities
sudo setcap cap_net_raw,cap_net_admin=eip /usr/sbin/tcpdump
```

**No Packets Captured:**
```bash
# Verify interface is up
ip link show eth0

# Check BPF filter
sudo tcpdump -i eth0 -nn -v

# Test with simple filter
sudo tcpdump -i eth0 -nn -c 10
```

**Packet Loss:**
```bash
# Increase buffer size
sudo tcpdump -i eth0 -B 4096

# Use faster interface
sudo tcpdump -i eth1

# Reduce system load
sudo systemctl stop [unnecessary-services]
```

### Performance Issues

**High CPU Usage:**
```bash
# Use specific filters
sudo tcpdump -i eth0 -nn 'host 192.168.1.100'

# Limit packet count
sudo tcpdump -i eth0 -c 1000

# Reduce verbose output
sudo tcpdump -i eth0 -q
```

**File Size Issues:**
```bash
# Rotate files
sudo tcpdump -i eth0 -w capture.pcap -C 100 -W 10

# Limit capture length
sudo tcpdump -i eth0 -w capture.pcap -s 96

# Use compression
sudo tcpdump -i eth0 -w - | gzip > capture.pcap.gz
```

### FAQ

**Q: What is tcpdump used for?**
A: tcpdump is a command-line packet analyzer for capturing, displaying, and analyzing network traffic.

**Q: Does tcpdump require root privileges?**
A: Yes, for packet capture. Use sudo or run as root.

**Q: Can tcpdump capture HTTPS traffic?**
A: Yes, it can capture encrypted packets but cannot decrypt them.

**Q: Is tcpdump legal to use?**
A: Only on networks you own or have explicit authorization to test. Unauthorized use is illegal.

**Q: How does tcpdump differ from Wireshark?**
A: tcpdump is a command-line tool, while Wireshark is a GUI tool. Both read pcap files.

**Q: Can tcpdump capture on remote systems?**
A: Yes, via SSH or remote capture capabilities.

## Resources

### Official Documentation
- [tcpdump Manual Page](https://linux.die.net/man/8/tcpdump)
- [tcpdump Website](https://www.tcpdump.org/)
- [BPF Documentation](https://www.tcpdump.org/manpages/pcap-filter.7.html)

### Tutorials and Guides
- [tcpdump Tutorial](https://www.youtube.com/watch?v=QaJjJmMmKqQ)
- [BPF Filter Examples](https://www.tutorialspoint.com/tcpdump/index.htm)
- [Network Analysis with tcpdump](https://www.offensive-security.com)

### Related Tools
- **wireshark**: GUI protocol analyzer
- **tcpflow**: TCP flow recorder
- **tshark**: Command-line Wireshark
- **netsniff-ng**: High-performance toolkit
- **ngrep**: Network grep

### Books and Resources
- *TCP/IP Illustrated* by W. Richard Stevens
- *Network Security Assessment* by Chris McNab
- *The Practice of Network Security Monitoring* by Richard Bejtlich

### Community
- [tcpdump Mailing List](https://www.tcpdump.org/)
- [Kali Linux Forums](https://forums.kali.org/)
- [Reddit r/netsec](https://www.reddit.com/r/netsec/)
