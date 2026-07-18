---
title: "netsniff-ng - High-Performance Network Toolkit"
description: "Modern network toolkit with zero-copy packet capture, ring buffers, and high-speed packet generation capabilities"
mitre_tactic: discovery
mitre_tactic_id: TA0007
mitre_subcategory: network_sniffing
categories:
  - network-analysis
  - packet-capture
  - high-performance
  - traffic-generation
  - network-testing
tags:
  - netsniff-ng
  - trafgen
  - mausezahn
  - flowtop
  - bpfc
  - packet-capture
  - ring-buffer
  - zero-copy
install_command: "sudo apt install netsniff-ng"
version: "0.6.7"
github_repo: "https://github.com/netsniff-ng/netsniff-ng"
kali_tools_page: "https://www.kali.org/tools/netsniff-ng/"
difficulty_level: intermediate
requires_root: true
gui_available: false
related_tools:
  - tcpdump
  - wireshark
  - scapy
  - hping3
  - trafgen
---

# netsniff-ng - High-Performance Network Toolkit

## 1. Introduction

netsniff-ng is a high-performance network toolkit designed for modern high-speed networks. It utilizes zero-copy packet capture, ring buffers, and CPU-optimized techniques to achieve maximum throughput. The toolkit includes multiple subtools for packet capture, traffic generation, flow monitoring, and BPF compilation.

**Primary Use Cases:**
- High-speed packet capture and analysis
- Traffic generation and packet crafting
- Network flow monitoring and statistics
- BPF filter compilation and testing
- Protocol analysis and debugging
- Network performance testing
- Forensic packet analysis

**Tool Suite Components:**
- **netsniff-ng**: Main packet capture and analysis tool
- **trafgen**: High-speed traffic generator
- **mausezahn**: Packet crafting tool (like hping3 on steroids)
- **flowtop**: Real-time flow monitoring (like top for network)
- **bpfc**: BPF filter compiler and validator

**Key Features:**
- **Zero-Copy Capture**: Direct memory access for maximum speed
- **Ring Buffers**: Kernel-level packet buffering
- **Multi-Core Support**: CPU affinity and load balancing
- **PCap Compatibility**: Works with standard pcap files
- **BPF Support**: Berkeley Packet Filter expressions
- **IPv4/IPv6**: Full dual-stack support
- **Low Latency**: Optimized for real-time analysis

**Performance Capabilities:**
- Line-rate capture on 10Gbps+ networks
- Millions of packets per second
- Minimal CPU overhead
- Efficient memory usage

## 2. Installation

### Standard Installation

```bash
# Install netsniff-ng toolkit
sudo apt update
sudo apt install netsniff-ng

# Verify installation
netsniff-ng --help
trafgen --help
mausezahn --help
flowtop --help
bpfc --help
```

### Installation from Source

```bash
# Install dependencies
sudo apt install cmake build-essential libpcap-dev libnl-3-dev libnl-genl-3-dev \
  libncurses5-dev zlib1g-dev

# Clone repository
git clone https://github.com/netsniff-ng/netsniff-ng.git
cd netsniff-ng

# Build and install
mkdir build && cd build
cmake -DCMAKE_INSTALL_PREFIX=/usr ..
make
sudo make install
```

### Verify Installation

```bash
# Check all tools are installed
which netsniff-ng trafgen mausezahn flowtop bpfc

# Check version
netsniff-ng --version

# List available interfaces
netsniff-ng --list
```

## 3. Core Concepts

### Zero-Copy Packet Capture

Traditional packet capture involves multiple memory copies between kernel and user space. Zero-copy capture eliminates these copies:
- Kernel writes directly to user-space memory
- Minimal CPU overhead
- Maximum throughput on high-speed networks

### Ring Buffers

Ring buffers provide kernel-level packet buffering:
- Circular buffer in kernel memory
- Prevents packet loss during high traffic
- Configurable buffer sizes
- Asynchronous packet delivery

### CPU Affinity and Load Balancing

netsniff-ng can distribute capture across multiple CPU cores:
- Pin capture to specific cores
- Load balance across available CPUs
- Optimize cache utilization
- Maximize parallel processing

### BPF Filters

Berkeley Packet Filter expressions for precise packet selection:
- Compile to BPF bytecode
- Execute in kernel space
- Minimal overhead
- Complex filtering logic

## 4. Command Reference

### netsniff-ng (Main Capture Tool)

```bash
# Basic syntax
netsniff-ng [options]

# Interface selection
-i <interface>          # Specify network interface
--in <interface>        # Long form

# Output options
-o <filename>           # Output file (pcap)
--out <filename>        # Long form

# Filter options
-f <filter>             # BPF filter expression
--bpffile <file>        # Read BPF from file

# Capture options
-c <count>              # Number of packets to capture
-s <snaplen>            # Snap length (bytes)
--silent                # Suppress statistics output

# Performance options
--ring-size <size>      # Ring buffer size (pages)
--bundle <num>          # Number of bundles
--fanout                # Enable fanout mode
--cpu <core>            # Pin to specific CPU core
```

### trafgen (Traffic Generator)

```bash
# Basic syntax
trafgen [options]

# Interface selection
-o <interface>          # Output interface

# Input options
--in <file>             # Read packet from file
--cfg <file>            # Read configuration from file

# Rate limiting
-p <pps>                # Packets per second
-B <bandwidth>          # Bandwidth limit (e.g., 100mbit)

# Packet options
-d <destination>        # Destination MAC
-s <source>             # Source MAC
--ip-dst <ip>           # Destination IP
--ip-src <ip>           # Source IP
```

### mausezahn (Packet Crafter)

```bash
# Basic syntax
mausezahn [options]

# Interface selection
-i <interface>          # Specify network interface

# Packet options
-a <source>             # Source MAC/IP
-b <destination>        # Destination MAC/IP
-t <type>               # Packet type (icmp, udp, tcp, arp)

# Rate options
-r <pps>                # Packets per second
-u <burst>              # Burst mode

# Payload options
-p <payload>            # Custom payload
-d <data>               # Hex data
```

### flowtop (Flow Monitor)

```bash
# Basic syntax
flowtop [options]

# Interface selection
-i <interface>          # Specify network interface

# Display options
-n                      # Don't resolve hostnames
-t <seconds>            # Update interval
-c                      # Color output

# Filter options
-f <filter>             # BPF filter expression
```

### bpfc (BPF Compiler)

```bash
# Basic syntax
bpfc [options]

# Input options
-f <filter>             # BPF filter expression
-i <file>               # Read from file

# Output options
-o <file>               # Write to file
-c                      # Compile and display bytecode
-p                      # Print filter in C format
```

## 5. Examples

### Basic Examples

```bash
# Capture packets to file
sudo netsniff-ng -i eth0 -o capture.pcap

# Capture with filter
sudo netsniff-ng -i eth0 -o capture.pcap -f "tcp port 80"

# Capture specific number of packets
sudo netsniff-ng -i eth0 -o capture.pcap -c 1000

# Capture with ring buffer
sudo netsniff-ng -i eth0 -o capture.pcap --ring-size 4096

# Read pcap file
netsniff-ng -i capture.pcap
```

### Intermediate Examples

```bash
# High-speed capture with multiple cores
sudo netsniff-ng -i eth0 -o capture.pcap --fanout --bundle 4

# Capture with custom snap length
sudo netsniff-ng -i eth0 -o capture.pcap -s 96

# Monitor flows in real-time
sudo flowtop -i eth0

# Generate traffic with trafgen
sudo trafgen -o eth0 --cfg traffic.cfg -p 10000

# Create ARP flood with mausezahn
sudo mausezahn -i eth0 -t arp "a ff:ff:ff:ff:ff:ff" -r 1000
```

### Advanced Examples

```bash
# Multi-interface capture
sudo netsniff-ng -i eth0 -o eth0.pcap --fanout &
sudo netsniff-ng -i eth1 -o eth1.pcap --fanout &

# Capture with CPU affinity
sudo netsniff-ng -i eth0 -o capture.pcap --cpu 0 --ring-size 8192

# Traffic generation script
#!/bin/bash
# Generate various traffic types

# HTTP traffic
sudo trafgen -o eth0 -p 1000 --ip-dst 192.168.1.100 \
  --ip-src 192.168.1.1 -t tcp --dport 80

# DNS queries
sudo trafgen -o eth0 -p 100 --ip-dst 8.8.8.8 \
  -t udp --dport 53

# ICMP echo
sudo trafgen -o eth0 -p 1000 --ip-dst 192.168.1.100 \
  -t icmp --icmp-type 8
```

### Real-World Scenarios

```bash
# Network performance testing
sudo trafgen -o eth0 -p 100000 --ip-dst 192.168.1.100 \
  --bandwidth 1gbit

# Forensic capture
sudo netsniff-ng -i eth0 -o forensic_$(date +%Y%m%d_%H%M%S).pcap \
  --silent --ring-size 16384

# VoIP monitoring
sudo flowtop -i eth0 -f "udp port 5060 or udp portrange 10000-20000"

# Security monitoring
sudo netsniff-ng -i eth0 -o security.pcap \
  -f "not arp and not icmp and host 192.168.1.0/24"
```

## 6. Advanced Usage

### Custom BPF Filters

```bash
# Compile BPF filter
bpfc -f "tcp port 80 and host 192.168.1.100"

# Save BPF to file
bpfc -f "tcp port 80" -o filter.bpf

# Use BPF file
netsniff-ng -i eth0 -o capture.pcap --bpffile filter.bpf

# Complex filters
bpfc -f "(tcp port 80 or tcp port 443) and not arp"
```

### Performance Tuning

```bash
# Optimize ring buffer size
# Calculate: ring_size = (bandwidth * latency) / (packet_size * 8)

# For 10Gbps with 1500 byte packets:
sudo netsniff-ng -i eth0 -o capture.pcap --ring-size 32768

# Enable CPU pinning
sudo netsniff-ng -i eth0 -o capture.pcap --cpu 0

# Enable fanout for multi-core
sudo netsniff-ng -i eth0 -o capture.pcap --fanout --bundle 8
```

### Scripting and Automation

```bash
#!/bin/bash
# Automated capture script

INTERFACE="eth0"
OUTPUT_DIR="/var/log/captures"
DURATION=3600

mkdir -p $OUTPUT_DIR

# Start capture
sudo netsniff-ng -i $INTERFACE \
  -o $OUTPUT_DIR/capture_$(date +%Y%m%d_%H%M%S).pcap \
  --silent \
  --ring-size 8192 \
  --fanout &

CAPTURE_PID=$!

# Wait for duration
sleep $DURATION

# Stop capture
kill $CAPTURE_PID

# Analyze capture
tcpdump -r $OUTPUT_DIR/capture_*.pcap -c 100
```

### Integration with Other Tools

```bash
# Pipe to Wireshark
sudo netsniff-ng -i eth0 -o - | wireshark -k -

# Analyze with tcpdump
sudo netsniff-ng -i eth0 -o capture.pcap -c 1000
tcpdump -r capture.pcap -v

# Process with tshark
sudo netsniff-ng -i eth0 -o capture.pcap -c 1000
tshark -r capture.pcap -T fields -e ip.src -e ip.dst

# Use with flowtop
sudo flowtop -i eth0 -t 5
```

## 7. Detection & Defense

### Detection Methods

**Network Monitoring:**
```bash
# Monitor for high-speed capture
sudo flowtop -i eth0

# Detect unusual traffic patterns
sudo netsniff-ng -i eth0 -o monitor.pcap -f "not arp"

# Monitor interface statistics
ifconfig eth0 | grep -i "RX\|TX"

# Check for promiscuous mode
cat /sys/class/net/eth0/flags
```

**IDS Integration:**
- Snort rules for high-speed capture detection
- Suricata rules for traffic generation
- Zeek scripts for flow anomalies

### Defense Strategies

**Network Security:**
```bash
# Enable port security
# switch(config-if)# switchport port-security

# Implement rate limiting
sudo iptables -A INPUT -m limit --limit 1000/min -j ACCEPT

# Enable traffic monitoring
sudo apt install iftop nethogs
```

**Host-Based Protection:**
```bash
# Monitor network interfaces
watch -n 2 'ifconfig eth0'

# Check for unusual processes
sudo netstat -tulpn | grep -i "netsniff\|trafgen"

# Enable logging
sudo auditctl -w /usr/bin/netsniff-ng -p rwxa
```

**Performance Monitoring:**
```bash
# Monitor CPU usage
top -p $(pgrep -d, netsniff-ng)

# Monitor memory usage
ps aux | grep netsniff-ng

# Check ring buffer stats
cat /proc/net/pf_ring/stats
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
sudo netsniff-ng -i eth0

# Check capabilities
sudo setcap cap_net_raw,cap_net_admin=eip /usr/bin/netsniff-ng
```

**Ring Buffer Errors:**
```bash
# Check system limits
cat /proc/sys/net/core/rmem_max
cat /proc/sys/net/core/wmem_max

# Increase limits
sudo sysctl -w net.core.rmem_max=26214400
sudo sysctl -w net.core.wmem_max=26214400

# Check available memory
free -m
```

**High CPU Usage:**
```bash
# Reduce capture scope
sudo netsniff-ng -i eth0 -o capture.pcap -c 1000

# Use BPF filter
sudo netsniff-ng -i eth0 -o capture.pcap -f "tcp port 80"

# Pin to specific CPU
sudo netsniff-ng -i eth0 -o capture.pcap --cpu 0
```

### Performance Issues

**Packet Loss:**
```bash
# Increase ring buffer
sudo netsniff-ng -i eth0 -o capture.pcap --ring-size 16384

# Enable fanout
sudo netsniff-ng -i eth0 -o capture.pcap --fanout

# Use faster interface
sudo netsniff-ng -i eth1 -o capture.pcap

# Reduce system load
sudo systemctl stop [unnecessary-services]
```

**Capture Too Slow:**
```bash
# Check CPU frequency
cat /proc/cpuinfo | grep "cpu MHz"

# Set performance mode
sudo cpupower frequency-set -g performance

# Check for thermal throttling
sensors

# Optimize kernel parameters
sudo sysctl -w net.core.netdev_max_backlog=5000
```

### FAQ

**Q: What is netsniff-ng used for?**
A: netsniff-ng is a high-performance network toolkit for packet capture, traffic generation, and flow monitoring on modern high-speed networks.

**Q: How does netsniff-ng differ from tcpdump?**
A: netsniff-ng uses zero-copy capture and ring buffers for maximum throughput, while tcpdump is more portable but slower on high-speed networks.

**Q: Can netsniff-ng capture on 10Gbps+ networks?**
A: Yes, netsniff-ng is designed for line-rate capture on high-speed networks with proper hardware and configuration.

**Q: Is netsniff-ng legal to use?**
A: Only on networks you own or have explicit authorization to test. Unauthorized use is illegal.

**Q: What is trafgen used for?**
A: trafgen is a high-speed traffic generator for network testing, performance benchmarking, and protocol validation.

**Q: Does netsniff-ng support IPv6?**
A: Yes, netsniff-ng has full IPv4/IPv6 support.

## Resources

### Official Documentation
- [netsniff-ng Manual Page](https://linux.die.net/man/8/netsniff-ng)
- [netsniff-ng GitHub Repository](https://github.com/netsniff-ng/netsniff-ng)
- [netsniff-ng Documentation](https://netsniff-ng.readthedocs.io/)

### Tutorials and Guides
- [High-Speed Packet Capture with netsniff-ng](https://www.youtube.com/watch?v=QaJjJmMmKqQ)
- [Network Performance Testing](https://www.tutorialspoint.com/netsniff-ng/index.htm)
- [Traffic Generation Tutorial](https://www.offensive-security.com)

### Related Tools
- **tcpdump**: Command-line packet analyzer
- **wireshark**: GUI protocol analyzer
- **scapy**: Python packet manipulation
- **hping3**: Network packet generator
- **iperf**: Network performance testing

### Books and Resources
- *TCP/IP Illustrated* by W. Richard Stevens
- *Network Security Assessment* by Chris McNab
- *High Performance Browser Networking* by Ilya Grigorik

### Community
- [netsniff-ng GitHub Issues](https://github.com/netsniff-ng/netsniff-ng/issues)
- [Kali Linux Forums](https://forums.kali.org/)
- [Reddit r/netsec](https://www.reddit.com/r/netsec/)
