---
title: "wireshark - THE Network Protocol Analyzer"
description: "Industry-standard network protocol analyzer with GUI for deep packet inspection, 3000+ protocol support, and comprehensive analysis features"
mitre_tactic: discovery
mitre_tactic_id: TA0007
mitre_subcategory: network_sniffing
categories:
  - packet-analysis
  - protocol-analyzer
  - network-forensics
  - deep-inspection
  - security-analysis
tags:
  - wireshark
  - tshark
  - protocol-analyzer
  - packet-capture
  - network-forensics
  - deep-inspection
install_command: "sudo apt install wireshark"
version: "4.2.0"
github_repo: "https://github.com/wireshark/wireshark"
kali_tools_page: "https://www.kali.org/tools/wireshark/"
difficulty_level: beginner
requires_root: true
gui_available: true
related_tools:
  - tcpdump
  - tcpflow
  - tshark
  - netsniff-ng
  - ngrep
---

# wireshark - THE Network Protocol Analyzer

## 1. Introduction

Wireshark is the world's foremost network protocol analyzer. It lets you capture and interactively browse the traffic running on a computer network. With support for 3000+ protocols, deep packet inspection, and comprehensive analysis features, Wireshark is the industry standard for network analysis.

**Primary Use Cases:**
- Network troubleshooting and analysis
- Protocol development and debugging
- Security forensics and incident response
- Network performance monitoring
- Packet capture and recording
- Deep packet inspection
- Educational protocol learning

**Key Features:**
- **3000+ Protocols**: Deep inspection of virtually any protocol
- **Live Capture**: Real-time packet capture
- **Offline Analysis**: Analyze saved capture files
- **Color Coding**: Visual packet categorization
- **Stream Following**: Reconstruct TCP/UDP/SSL streams
- **IO Graphs**: Traffic visualization
- **Expert Info**: Protocol anomalies and warnings
- **Cross-Platform**: Windows, macOS, Linux

**Wireshark vs tshark:**
- **Wireshark**: Full-featured GUI application
- **tshark**: Command-line version for scripting and automation

## 2. Installation

### Standard Installation

```bash
# Install Wireshark (GUI)
sudo apt update
sudo apt install wireshark

# Add user to wireshark group
sudo usermod -aG wireshark $USER

# Verify installation
wireshark --version
tshark --version
```

### Installation from Source

```bash
# Install dependencies
sudo apt install cmake build-essential libgtk-3-dev libpcap-dev \
  libgnutls28-dev qtbase5-dev qttools5-dev libqt5svg5-dev

# Clone repository
git clone https://github.com/wireshark/wireshark.git
cd wireshark

# Build and install
mkdir build && cd build
cmake -DCMAKE_INSTALL_PREFIX=/usr ..
make
sudo make install
```

### Post-Installation Setup

```bash
# Enable dumpcap for non-root capture
sudo dpkg-reconfigure wireshark-common

# Add user to wireshark group
sudo usermod -aG wireshark $USER

# Log out and back in for group changes
```

## 3. Core Concepts

### Capture vs Display Filters

**Capture Filters (BPF):**
- Applied during capture
- Reduces amount of data captured
- Uses BPF syntax
- Example: `tcp port 80`

**Display Filters:**
- Applied after capture
- Hides/shows packets in display
- Uses Wireshark-specific syntax
- Example: `tcp.port == 80`

### Packet Dissection

Wireshark decodes protocol layers:
- **Link Layer**: Ethernet, WiFi, PPP
- **Network Layer**: IP, IPv6, ICMP, ARP
- **Transport Layer**: TCP, UDP, SCTP
- **Application Layer**: HTTP, DNS, DHCP, etc.

### Color Coding

Wireshark colors packets by type:
- **Green**: TCP traffic
- **Blue**: DNS traffic
- **Light Purple**: HTTP traffic
- **Black**: Errors and retransmissions

### Stream Following

Reconstruct complete conversations:
- **TCP Streams**: Full TCP conversation
- **UDP Streams**: UDP data flows
- **SSL/TLS Streams**: Encrypted data (with keys)
- **HTTP Streams**: Web traffic

## 4. Command Reference

### Wireshark (GUI)

```bash
# Basic syntax
wireshark [options]

# Interface selection
-i <interface>          # Specify network interface

# Capture options
-f <filter>             # Capture filter
-c <count>              # Packet count
-s <snaplen>            # Snapshot length

# File options
-r <pcapfile>           # Read pcap file
-w <pcapfile>           # Write to pcap file

# Display options
-Y <filter>             # Display filter
```

### tshark (Command-Line)

```bash
# Basic syntax
tshark [options]

# Interface selection
-i <interface>          # Specify network interface
-i any                  # Capture on all interfaces

# Capture options
-f <filter>             # Capture filter (BPF)
-c <count>              # Packet count
-a <autostop>           # Auto-stop condition

# Display options
-Y <filter>             # Display filter
-V                      # Verbose output
-x                      # Hex dump

# Output options
-w <pcapfile>           # Write to pcap file
-r <pcapfile>           # Read from pcap file
-T fields               # Field output format
-e <field>              # Extract field

# Statistics options
-z io,stat,1            # IO statistics
-z conv,tcp             # TCP conversations
-z http,stat            # HTTP statistics
```

### Common Display Filters

```bash
# Protocol filters
http
tcp
udp
dns
arp
icmp

# Host filters
ip.addr == 192.168.1.100
ip.src == 192.168.1.100
ip.dst == 192.168.1.100

# Port filters
tcp.port == 80
tcp.dstport == 443
udp.port == 53

# Combined filters
ip.addr == 192.168.1.100 and tcp.port == 80
http or dns
tcp.flags.syn == 1
```

## 5. Examples

### Basic Examples

```bash
# Start Wireshark with interface
wireshark -i eth0

# Start tshark capture
sudo tshark -i eth0

# Capture with filter
sudo tshark -i eth0 -f "tcp port 80"

# Capture 100 packets
sudo tshark -i eth0 -c 100

# Capture to file
sudo tshark -i eth0 -w capture.pcap

# Read from file
tshark -r capture.pcap
```

### Intermediate Examples

```bash
# Verbose output
sudo tshark -i eth0 -V -c 10

# Hex dump
sudo tshark -i eth0 -x -c 10

# Extract specific fields
sudo tshark -i eth0 -T fields -e ip.src -e ip.dst -e tcp.port

# Filter HTTP traffic
sudo tshark -i eth0 -Y "http" -c 100

# Filter by host
sudo tshark -i eth0 -Y "ip.addr == 192.168.1.100"

# IO statistics
sudo tshark -i eth0 -z io,stat,1
```

### Advanced Examples

```bash
# Capture with auto-stop
sudo tshark -i eth0 -a duration:300 -w capture.pcap

# Capture with file rotation
sudo tshark -i eth0 -b filesize:10000 -b files:5 -w capture.pcap

# Extract HTTP requests
sudo tshark -i eth0 -Y "http.request" -T fields -e http.request.method -e http.request.uri

# Extract DNS queries
sudo tshark -i eth0 -Y "dns" -T fields -e dns.qry.name

# Extract TLS certificates
sudo tshark -i eth0 -Y "tls.handshake.type == 11" -T fields -e x509sat.utf8String

# Generate statistics
sudo tshark -i eth0 -z conv,tcp -z http,stat -z dns,stat
```

### Real-World Scenarios

```bash
# Network troubleshooting
wireshark -i eth0 -Y "tcp.analysis.retransmission"

# Security monitoring
sudo tshark -i eth0 -Y "arp" -T fields -e arp.src.hw_mac -e arp.src.proto_ipv4

# HTTP analysis
sudo tshark -i eth0 -Y "http" -T fields -e http.host -e http.request.uri -e http.response.code

# DNS monitoring
sudo tshark -i eth0 -Y "dns" -T fields -e dns.qry.name -e dns.flags.response

# VoIP analysis
wireshark -i eth0 -Y "sip or rtp"
```

## 6. Advanced Usage

### Custom Display Filters

```bash
# Complex filters
ip.addr == 192.168.1.100 and (tcp.port == 80 or tcp.port == 443)
http and !(ip.dst == 192.168.1.1)
tcp.flags.syn == 1 and tcp.flags.ack == 0

# Filter by protocol fields
http.request.method == "POST"
dns.qry.name contains "example.com"
tcp.analysis.retransmission

# Filter by packet size
frame.len > 1000
frame.len < 100

# Filter by time
frame.time >= "2024-01-01 00:00:00"
```

### Field Extraction

```bash
# Extract IP addresses
sudo tshark -i eth0 -T fields -e ip.src -e ip.dst

# Extract TCP ports
sudo tshark -i eth0 -T fields -e tcp.srcport -e tcp.dstport

# Extract HTTP data
sudo tshark -i eth0 -T fields -e http.host -e http.request.uri

# Extract DNS queries
sudo tshark -i eth0 -T fields -e dns.qry.name

# Extract TLS info
sudo tshark -i eth0 -T fields -e tls.handshake.type -e tls.handshake.ciphersuite
```

### Scripting and Automation

```bash
#!/bin/bash
# Automated Wireshark analysis

INTERFACE="eth0"
DURATION=300
OUTPUT_DIR="/var/log/wireshark"

mkdir -p $OUTPUT_DIR

# Capture traffic
sudo tshark -i $INTERFACE \
  -w $OUTPUT_DIR/capture_$(date +%Y%m%d_%H%M%S).pcap \
  -a duration:$DURATION &

TSHARK_PID=$!

# Wait for capture
wait $TSHARK_PID

# Analyze capture
for file in $OUTPUT_DIR/*.pcap; do
  echo "Analyzing $file"
  
  # Basic stats
  tshark -r $file -z conv,tcp > $file.stats.txt
  
  # Extract HTTP
  tshark -r $file -Y "http" -T fields -e http.host > $file.http.txt
  
  # Extract DNS
  tshark -r $file -Y "dns" -T fields -e dns.qry.name > $file.dns.txt
done
```

### Integration with Other Tools

```bash
# Capture with tcpdump, analyze with Wireshark
tcpdump -i eth0 -w capture.pcap -c 1000
wireshark -r capture.pcap

# Pipe to Wireshark
sudo tshark -i eth0 -w - | wireshark -k -

# Use with tcpflow
tcpflow -r capture.pcap -o /tmp/flows
wireshark -r capture.pcap -Y "tcp.stream eq 1"

# Process with tshark
sudo tshark -i eth0 -w - -c 1000 | tshark -r - -Y "http"
```

## 7. Detection & Defense

### Detection Methods

**Monitor for Capture:**
```bash
# Check for Wireshark processes
ps aux | grep -i "wireshark\|tshark"

# Monitor network interfaces
ifconfig eth0 | grep -i "RX\|TX"

# Check for dumpcap
ps aux | grep dumpcap

# Monitor disk usage
df -h /var/log
```

**IDS Signatures:**
- Snort rules for packet capture detection
- Suricata rules for Wireshark patterns
- Zeek scripts for monitoring anomalies

### Defense Strategies

**Network Security:**
```bash
# Enable encryption
# Use HTTPS, SSH, SFTP instead of HTTP, Telnet, FTP

# Implement VPN
# Route sensitive traffic through encrypted tunnels

# Monitor for exfiltration
# Use DLP solutions to detect data leaving network
```

**Host-Based Protection:**
```bash
# Monitor for capture
sudo auditctl -w /usr/bin/wireshark -p rwxa
sudo auditctl -w /usr/bin/tshark -p rwxa

# Enable logging
sudo logger -t security "Wireshark detected"

# Check capabilities
getcap /usr/bin/dumpcap
```

**Encrypted Communications:**
- Use end-to-end encryption
- Implement certificate pinning
- Enable TLS 1.3
- Use VPN for sensitive traffic

## 8. Troubleshooting & FAQ

### Common Issues

**"No such device" Error:**
```bash
# List interfaces
wireshark -D
tshark -D

# Check interface status
ip link show eth0

# Enable interface
sudo ip link set eth0 up
```

**"Permission Denied" Error:**
```bash
# Run with sudo (not recommended for GUI)
sudo wireshark

# Better: Add user to wireshark group
sudo usermod -aG wireshark $USER
newgrp wireshark

# Check capabilities
getcap /usr/bin/dumpcap
```

**No Packets Captured:**
```bash
# Verify interface is up
ip link show eth0

# Check capture filter
sudo tshark -i eth0 -f "tcp port 80" -v

# Test with simple filter
sudo tshark -i eth0 -c 10
```

**Display Filter Not Working:**
```bash
# Check filter syntax
tshark -r capture.pcap -Y "http"

# Use autocomplete
# In Wireshark: Ctrl+Space

# Check field names
tshark -G fields | grep -i "http"
```

### Performance Issues

**High CPU Usage:**
```bash
# Reduce capture scope
sudo tshark -i eth0 -c 1000

# Use specific filters
sudo tshark -i eth0 -f "tcp port 80"

# Reduce verbose output
sudo tshark -i eth0 -q
```

**Large Capture Files:**
```bash
# Limit capture length
sudo tshark -i eth0 -s 96 -w capture.pcap

# Rotate files
sudo tshark -i eth0 -b filesize:10000 -b files:5 -w capture.pcap

# Compress old files
gzip capture_*.pcap
```

### FAQ

**Q: What is Wireshark used for?**
A: Wireshark is a network protocol analyzer for capturing, displaying, and analyzing network traffic.

**Q: Does Wireshark require root privileges?**
A: For live capture, yes (or proper capabilities). For reading pcap files, no.

**Q: Can Wireshark capture HTTPS traffic?**
A: Yes, but the data is encrypted. You need the decryption keys to see the content.

**Q: Is Wireshark legal to use?**
A: Only on networks you own or have explicit authorization to test. Unauthorized use is illegal.

**Q: How does Wireshark differ from tcpdump?**
A: Wireshark has a GUI with advanced analysis features, while tcpdump is command-line only.

**Q: What is tshark?**
A: tshark is the command-line version of Wireshark for scripting and automation.

## Resources

### Official Documentation
- [Wireshark User's Guide](https://www.wireshark.org/docs/)
- [Wireshark Display Filter Reference](https://www.wireshark.org/docs/wsug_html_chunked/ChWorkBuildDisplayFilterSection.html)
- [tshark Manual Page](https://www.wireshark.org/docs/man-pages/tshark.html)

### Tutorials and Guides
- [Wireshark Tutorial](https://www.youtube.com/watch?v=QaJjJmMmKqQ)
- [Network Analysis with Wireshark](https://www.tutorialspoint.com/wireshark/index.htm)
- [Wireshark for Security](https://www.offensive-security.com)

### Related Tools
- **tcpdump**: Command-line packet analyzer
- **tcpflow**: TCP flow recorder
- **tshark**: Command-line Wireshark
- **netsniff-ng**: High-performance toolkit
- **ngrep**: Network grep

### Books and Resources
- *Wireshark & Ethical Hacking Network Analysis* by Laura Chappell
- *Network Security Monitoring* by Richard Bejtlich
- *TCP/IP Illustrated* by W. Richard Stevens

### Community
- [Wireshark Q&A](https://ask.wireshark.org/)
- [Wireshark Mailing Lists](https://www.wireshark.org/mailman/listinfo)
- [Kali Linux Forums](https://forums.kali.org/)
- [Reddit r/wireshark](https://www.reddit.com/r/wireshark/)
