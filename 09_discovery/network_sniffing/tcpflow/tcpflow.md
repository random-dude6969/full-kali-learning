---
title: "tcpflow - TCP Flow Recorder"
description: "Network forensics tool that records TCP flows in separate files, reconstructing complete TCP streams for analysis"
mitre_tactic: discovery
mitre_tactic_id: TA0007
mitre_subcategory: network_sniffing
categories:
  - network-forensics
  - tcp-analysis
  - packet-capture
  - stream-reconstruction
  - security-analysis
tags:
  - tcpflow
  - tcp-streams
  - network-forensics
  - flow-recording
  - pcap-analysis
  - stream-reconstruction
install_command: "sudo apt install tcpflow"
version: "1.6.1"
github_repo: "https://github.com/simsong/tcpflow"
kali_tools_page: "https://www.kali.org/tools/tcpflow/"
difficulty_level: beginner
requires_root: true
gui_available: false
related_tools:
  - tcpdump
  - wireshark
  - chaosreader
  - ngrep
  - tshark
---

# tcpflow - TCP Flow Recorder

## 1. Introduction

tcpflow is a network forensics tool that records TCP flows in separate files, reconstructing complete TCP streams. Unlike tcpdump which captures individual packets, tcpflow reassembles the entire TCP conversation into readable files.

**Primary Use Cases:**
- Network forensics and incident response
- TCP stream reconstruction and analysis
- File extraction from network traffic
- HTTP, FTP, SMTP, and other protocol analysis
- Web traffic reconstruction
- Email capture and analysis
- Data exfiltration detection

**Key Features:**
- **Flow Recording**: Each TCP conversation saved separately
- **Stream Reconstruction**: Reassembles out-of-order packets
- **File Extraction**: Extracts transferred files automatically
- **Protocol Awareness**: Understands HTTP, FTP, SMTP, etc.
- **PCap Support**: Read from pcap files
- **XML Output**: Machine-readable output
- **Compression**: Optional gzip compression
- **IPv4/IPv6**: Full dual-stack support

**How tcpflow Works:**
1. Captures packets on network interface
2. Identifies TCP streams by 4-tuple (src/dst IP/port)
3. Reassembles packets in order
4. Writes each stream to separate file
5. Extracts files from streams (HTTP, FTP, etc.)

**Output Format:**
- Files named: `<src_ip>.<src_port>-<dst_ip>.<dst_port>`
- Example: `192.168.1.100.44362-192.168.1.1.80`
- Directory: Default `./output/`

## 2. Installation

### Standard Installation

```bash
# Install tcpflow
sudo apt update
sudo apt install tcpflow

# Verify installation
tcpflow --version

# Check available options
tcpflow -h
```

### Installation from Source

```bash
# Install dependencies
sudo apt install build-essential libpcap-dev libboost-all-dev

# Clone repository
git clone https://github.com/simsong/tcpflow.git
cd tcpflow

# Build and install
./bootstrap
./configure
make
sudo make install
```

### Verify Installation

```bash
# Check installation
which tcpflow

# Test basic functionality
tcpflow -h

# Check version
tcpflow --version
```

## 3. Core Concepts

### TCP Flow Recording

tcpflow captures and records complete TCP conversations:
- **4-Tuple Identification**: src_ip, src_port, dst_ip, dst_port
- **Bidirectional**: Both directions in same or separate files
- **Reassembly**: Handles out-of-order packets
- **Gap Detection**: Marks missing data with `<gap>` markers

**Flow File Naming:**
```
<src_ip>.<src_port>-<dst_ip>.<dst_port>
```

**Example:**
```
192.168.1.100.44362-192.168.1.1.80    # HTTP request/response
192.168.1.100.51234-192.168.1.1.443   # HTTPS connection
```

### Stream Reconstruction

tcpflow reassembles TCP streams:
- Handles sequence numbers
- Detects gaps in data
- Preserves order
- Handles retransmissions

**Gap Handling:**
- Missing data marked with `<gap>` or `<dropped>`
- Preserves approximate positions
- Allows partial reconstruction

### File Extraction

tcpflow automatically extracts files from streams:
- **HTTP**: Downloads, uploads, embedded objects
- **FTP**: Transferred files
- **SMTP**: Email attachments
- **IMAP**: Email content
- **POP3**: Email content

**Extraction Directory:**
```
<output_dir>/<flow_file>/
  ├── index.html
  ├── style.css
  ├── script.js
  └── image.png
```

### Protocol Awareness

tcpflow understands common protocols:
- **HTTP**: Request/response parsing
- **FTP**: Command/response analysis
- **SMTP**: Email protocol handling
- **DNS**: Query/response parsing
- **TLS/SSL**: Connection detection (encrypted data preserved)

## 4. Command Reference

### Basic Options

```bash
# Basic syntax
tcpflow [options] [filter expression]

# Interface selection
-i <interface>          # Specify network interface

# Output directory
-o <directory>          # Output directory
--output <directory>    # Long form

# Read from file
-r <pcapfile>           # Read from pcap file
--read <pcapfile>       # Long form
```

### Capture Options

```bash
# BPF filter
-f <filter>             # BPF filter expression
--filter <filter>       # Long form

# Verbose output
-v                      # Verbose mode
-vv                     # More verbose

# Quiet mode
-q                      # Quiet (no output)

# Packet count
-c <count>              # Stop after count packets
--count <count>         # Long form
```

### Output Options

```bash
# Output directory
-o <directory>          # Set output directory

# XML output
-X                      # Generate XML output
--xml                   # Long form

# Gzip compression
-z                      # Compress output files
--gzip                  # Long form

# No filenames
-n                      # Don't add filenames
--no-filenames          # Long form

# Max file size
-s <bytes>              # Maximum file size
--max-bytes <bytes>     # Long form
```

### Advanced Options

```bash
# Dont strip headers
-H                      # Don't strip HTTP headers

# Dont write flows
-S                      # Don't write flow files

# Dont extract files
-E                      # Don't extract files

# Chunk size
-C <bytes>              # Chunk size for writing
--chunk-size <bytes>    # Long form

# Memory limit
-m <bytes>              # Memory limit
--memory-limit <bytes>  # Long form
```

### Reporting Options

```bash
# Report file
-R <file>              # Report file
--report <file>         # Long form

# Report format
--report-format <fmt>  # Report format (text, xml, json)
```

## 5. Examples

### Basic Examples

```bash
# Capture on interface
sudo tcpflow -i eth0

# Capture with output directory
sudo tcpflow -i eth0 -o /tmp/flows

# Capture specific count
sudo tcpflow -i eth0 -c 1000

# Capture with BPF filter
sudo tcpflow -i eth0 'tcp port 80'

# Read from pcap
tcpflow -r capture.pcap

# Read from pcap with output
tcpflow -r capture.pcap -o /tmp/flows
```

### Intermediate Examples

```bash
# Capture HTTP traffic
sudo tcpflow -i eth0 'tcp port 80' -o /tmp/http_flows

# Capture FTP traffic
sudo tcpflow -i eth0 'tcp port 21' -o /tmp/ftp_flows

# Capture SMTP traffic
sudo tcpflow -i eth0 'tcp port 25 or tcp port 587' -o /tmp/mail_flows

# Capture with XML output
sudo tcpflow -i eth0 -X -o /tmp/flows

# Capture with compression
sudo tcpflow -i eth0 -z -o /tmp/flows

# Read and analyze
tcpflow -r capture.pcap -o /tmp/flows
ls -la /tmp/flows/
```

### Advanced Examples

```bash
# Capture and extract files
sudo tcpflow -i eth0 -o /tmp/extracted 'tcp port 80'

# Extract files from pcap
tcpflow -r capture.pcap -o /tmp/extracted

# Capture with verbose output
sudo tcpflow -i eth0 -vv -o /tmp/flows 'host 192.168.1.100'

# Capture with max file size
sudo tcpflow -i eth0 -s 1048576 -o /tmp/flows

# Capture with memory limit
sudo tcpflow -i eth0 -m 268435456 -o /tmp/flows

# Generate report
sudo tcpflow -i eth0 -R /tmp/report.txt -o /tmp/flows
```

### Real-World Scenarios

```bash
# Network forensics
tcpflow -r evidence.pcap -o /tmp/forensics

# Web traffic analysis
sudo tcpflow -i eth0 'tcp port 80' -o /tmp/web_traffic

# Email capture
sudo tcpflow -i eth0 'tcp port 25 or tcp port 110 or tcp port 143' -o /tmp/emails

# File transfer analysis
sudo tcpflow -i eth0 'tcp port 21 or tcp port 22' -o /tmp/transfers

# Incident response
tcpflow -r incident.pcap -o /tmp/incident_analysis

# Continuous monitoring
sudo tcpflow -i eth0 -o /tmp/monitoring -z
```

## 6. Advanced Usage

### Custom BPF Filters

```bash
# Filter by host and port
sudo tcpflow -i eth0 'host 192.168.1.100 and tcp port 80'

# Filter by subnet
sudo tcpflow -i eth0 'net 192.168.1.0/24'

# Filter by protocol
sudo tcpflow -i eth0 'tcp'

# Complex filters
sudo tcpflow -i eth0 '(tcp port 80 or tcp port 443) and host 192.168.1.0/24'
```

### Scripting and Automation

```bash
#!/bin/bash
# Automated flow capture script

INTERFACE="eth0"
OUTPUT_DIR="/var/log/flows"
DURATION=3600

mkdir -p $OUTPUT_DIR

# Start capture
sudo tcpflow -i $INTERFACE \
  -o $OUTPUT_DIR/flows_$(date +%Y%m%d_%H%M%S) \
  -z \
  -R /tmp/report_$(date +%Y%m%d_%H%M%S).txt &

TCPFLOW_PID=$!

# Wait for duration
sleep $DURATION

# Stop capture
kill $TCPFLOW_PID

# Analyze extracted files
find $OUTPUT_DIR -type f -name "*.html" -o -name "*.pdf" -o -name "*.zip" | head -20
```

### Integration with Other Tools

```bash
# Capture with tcpdump, analyze with tcpflow
tcpdump -i eth0 -w capture.pcap -c 1000
tcpflow -r capture.pcap -o /tmp/flows

# Process extracted files
find /tmp/flows -type f -exec file {} \;

# Analyze HTTP content
cat /tmp/flows/*:80 | head -50

# Extract URLs
grep -h "GET\|POST" /tmp/flows/*:80 | awk '{print $2}' | sort -u

# Use with chaosreader
chaosreader capture.pcap
```

## 7. Detection & Defense

### Detection Methods

**Monitor for Flow Recording:**
```bash
# Check for tcpflow processes
ps aux | grep tcpflow

# Monitor disk usage
df -h /var/log

# Check for flow files
find /tmp -name "*.*-*.*" -type f

# Monitor network interfaces
ifconfig eth0 | grep -i "RX\|TX"
```

**IDS Signatures:**
- Snort rules for tcpflow patterns
- Suricata rules for flow recording
- Zeek scripts for stream analysis

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
sudo auditctl -w /usr/bin/tcpflow -p rwxa

# Enable logging
sudo logger -t security "tcpflow detected"

# Check capabilities
getcap /usr/bin/tcpflow
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
tcpflow -D

# Check interface status
ip link show eth0

# Enable interface
sudo ip link set eth0 up
```

**"Permission Denied" Error:**
```bash
# Run with sudo
sudo tcpflow -i eth0

# Check capabilities
sudo setcap cap_net_raw,cap_net_admin=eip /usr/bin/tcpflow
```

**No Flows Captured:**
```bash
# Verify BPF filter
sudo tcpflow -i eth0 -v 'tcp port 80'

# Check output directory
ls -la /tmp/flows/

# Test with simple filter
sudo tcpflow -i eth0 -c 10
```

**Missing Data in Flows:**
```bash
# Check for gaps
grep -l "<gap>" /tmp/flows/*

# Increase buffer size
sudo tcpflow -i eth0 -B 4096

# Use faster interface
sudo tcpflow -i eth1
```

### Performance Issues

**High Disk Usage:**
```bash
# Limit file size
sudo tcpflow -i eth0 -s 1048576

# Use compression
sudo tcpflow -i eth0 -z

# Rotate output directory
sudo tcpflow -i eth0 -o /tmp/flows_$(date +%Y%m%d)

# Clean old flows
find /tmp/flows* -mtime +7 -delete
```

**High CPU Usage:**
```bash
# Use specific filters
sudo tcpflow -i eth0 'host 192.168.1.100'

# Limit packet count
sudo tcpflow -i eth0 -c 1000

# Reduce verbose output
sudo tcpflow -i eth0 -q
```

### FAQ

**Q: What is tcpflow used for?**
A: tcpflow is a network forensics tool that records TCP flows in separate files, reconstructing complete TCP streams for analysis.

**Q: Does tcpflow capture UDP traffic?**
A: No, tcpflow only captures TCP traffic. Use tcpdump for UDP capture.

**Q: Can tcpflow extract files from HTTP?**
A: Yes, tcpflow automatically extracts files from HTTP streams (downloads, uploads, embedded objects).

**Q: Is tcpflow legal to use?**
A: Only on networks you own or have explicit authorization to test. Unauthorized use is illegal.

**Q: How does tcpflow differ from tcpdump?**
A: tcpflow reassembles TCP streams into files, while tcpdump captures individual packets.

**Q: Can tcpflow capture HTTPS traffic?**
A: Yes, but the data will be encrypted. tcpflow can capture the encrypted stream but cannot decrypt it.

## Resources

### Official Documentation
- [tcpflow Manual Page](https://linux.die.net/man/8/tcpflow)
- [tcpflow GitHub Repository](https://github.com/simsong/tcpflow)
- [tcpflow Documentation](https://github.com/simsong/tcpflow/wiki)

### Tutorials and Guides
- [Network Forensics with tcpflow](https://www.youtube.com/watch?v=QaJjJmMmKqQ)
- [TCP Stream Analysis](https://www.tutorialspoint.com/tcpflow/index.htm)
- [Network Security Monitoring](https://www.offensive-security.com)

### Related Tools
- **tcpdump**: Command-line packet analyzer
- **wireshark**: GUI protocol analyzer
- **chaosreader**: Flow analysis tool
- **ngrep**: Network grep
- **tshark**: Command-line Wireshark

### Books and Resources
- *Network Security Monitoring* by Richard Bejtlich
- *Network Forensics* by Ric Messier
- *TCP/IP Illustrated* by W. Richard Stevens

### Community
- [tcpflow GitHub Issues](https://github.com/simsong/tcpflow/issues)
- [Kali Linux Forums](https://forums.kali.org/)
- [Reddit r/netsec](https://www.reddit.com/r/netsec/)
