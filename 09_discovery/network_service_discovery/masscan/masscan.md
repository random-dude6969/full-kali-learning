---
title: "masscan"
description: "TCP port scanner capable of scanning the entire Internet in under 5 minutes at 10 million packets per second"
categories: ["Network Service Discovery"]
tags: ["port-scanning", "tcp", "high-speed", "banner-grabbing", "async", "ipv4", "ipv6"]
mitre_tactic: "discovery"
mitre_tactic_id: "TA0007"
mitre_subcategory: "network_service_discovery"
install_command: "sudo apt install masscan"
version: "1.3.2"
github_repo: "https://github.com/robertdavidgraham/masscan"
kali_tools_page: "https://www.kali.org/tools/masscan/"
difficulty_level: "intermediate"
requires_root: true
gui_available: false
related_tools: ["nmap", "zmap", "unicornscan", "rustscan"]
---

# masscan

## Chapter 1: Introduction & Overview

### What is masscan?

masscan is a TCP port scanner that transmits packets at the theoretical maximum rate of Ethernet, achieving speeds of 10 million packets per second. It can scan the entire IPv4 address space in approximately 5 minutes at maximum rate. Unlike traditional scanners that rely on the operating system's TCP/IP stack, masscan implements its own TCP/IP stack, enabling it to bypass kernel limitations and achieve unprecedented scanning speeds.

The tool produces output in Nmap-compatible formats, making it a drop-in replacement for initial host discovery phases. masscan is designed for large-scale reconnaissance where speed is critical, such as mapping internet-facing services across entire autonomous systems or conducting rapid network audits.

### History & Background

masscan was created by Robert David Graham and released in 2013. It emerged from the need to scan large IP ranges quickly for the Mass Internet Scanning project. The tool solved the fundamental problem that OS TCP/IP stacks cannot handle the packet rates needed for full internet scans.

Key milestones:
- **2013**: Initial release, demonstrated 10M pps capability
- **2014**: Added banner grabbing for common protocols
- **2015**: IPv6 support added
- **2016**: PF_RING integration for >2M pps on standard hardware
- **2018**: Nmap output format compatibility
- **2020+**: Continued optimization, Windows support improvements

### When to Use This Tool

- **Large-scale reconnaissance**: Scan entire /8 or /16 ranges for specific services
- **Internet-wide surveys**: Map all instances of a service across the internet
- **Rapid host discovery**: Identify live hosts in massive IP ranges
- **Banner grabbing at scale**: Collect service banners from millions of hosts
- **Security research**: Large-scale vulnerability scanning infrastructure
- **Compliance auditing**: Verify service exposure across enterprise networks

### Key Concepts

**Asynchronous Transmission**: masscan sends packets without waiting for responses. This asynchronous approach maximizes throughput by eliminating the wait state between probes and responses.

**Custom TCP/IP Stack**: The tool implements its own TCP/IP stack in userspace, bypassing kernel networking entirely. This enables raw packet construction and transmission at wire speed.

**Adaptive Rate Control**: masscan automatically adjusts transmission rate based on packet loss, network congestion, and response patterns. The `--max-rate` option provides manual control.

**Banner Grabbing**: Beyond port detection, masscan captures service banners during the TCP handshake. Supported protocols include HTTP, SSH, FTP, SMTP, POP3, IMAP, SMB, RDP, VNC, and SSL/TLS.

**Seed-Based Randomization**: Scan order is determined by a seed value, enabling reproducible scans and distributed scanning across multiple machines that can later merge results.

---

## Chapter 2: Installation & Setup

### System Requirements

**Operating System**: Linux (primary), macOS, Windows (via MinGW/WSL)

**Hardware**:
- **Minimum**: Any modern x86_64 system with 1 Gbps NIC
- **Recommended**: Multi-core CPU, 10 Gbps NIC for high-speed scanning
- **For >2M pps**: PF_RING or DPDK compatible NIC

**Dependencies**:
- libpcap (packet capture)
- Root or CAP_NET_RAW capability
- Raw socket access (required for custom TCP/IP stack)

**Network**: masscan uses its own TCP/IP stack, so firewall rules may need adjustment. The tool requires raw socket access.

### Installation Methods

**From Kali repositories**:
```bash
sudo apt update
sudo apt install masscan
```

**From source** (latest version):
```bash
git clone https://github.com/robertdavidgraham/masscan.git
cd masscan
make
sudo make install
```

**Binary distribution**:
```bash
# Download latest release
wget https://github.com/robertdavidgraham/masscan/releases/download/1.3.2/masscan-1.3.2.tar.gz
tar -xzf masscan-1.3.2.tar.gz
cd masscan-1.3.2
make
sudo make install
```

### Configuration

masscan uses a configuration file at `/etc/masscan/masscan.conf` (if installed via apt). Configuration can also be passed via command-line or config file with `-c`.

**Default configuration**:
- Source IP: Randomized (requires firewall configuration)
- Source port: Randomized (requires firewall configuration)
- Rate: 100 packets/second (conservative default)
- Adapter: First available network interface

**Firewall configuration** (critical for proper operation):
```bash
# masscan uses its own TCP/IP stack
# Configure firewall to allow masscan traffic
sudo iptables -A OUTPUT -p tcp --sport 40000:60000 -j ACCEPT
sudo iptables -A INPUT -p tcp --sport 80,443,22 -j ACCEPT
```

### Verifying Installation

```bash
# Check version
masscan --version

# Verify binary location
which masscan

# Test configuration file
masscan --echo > /tmp/masscan_test.conf

# Verify network interface
masscan --list
```

---

## Chapter 3: Core Concepts & Architecture

### How It Works

masscan operates by constructing raw TCP packets and transmitting them at maximum rate without waiting for responses. The tool maintains its own TCP state machine, handling SYN, SYN-ACK, and RST packets independently of the kernel.

**Scanning Process**:

1. **Initialization**: masscan configures the network interface and loads target specifications
2. **Packet Generation**: TCP SYN packets are constructed for specified ports
3. **Transmission**: Packets are sent at configured rate (default 100 pps)
4. **Response Collection**: SYN-ACK responses indicate open ports
5. **Banner Grabbing**: Additional packets capture service banners
6. **Result Output**: Findings are written in configured format

**TCP Handshake Completion**:

When masscan receives a SYN-ACK (port open), it completes the handshake by sending an ACK, then a RST to close the connection cleanly. This enables banner grabbing while maintaining scan speed.

### Protocols & Standards

| Protocol | Usage | Notes |
|----------|-------|-------|
| TCP | Primary scan protocol | Custom implementation |
| IPv4 | Host addressing | Full /8 to /32 range support |
| IPv6 | Extended addressing | Dual-stack scanning |
| ARP | Host discovery | Optional, requires local network |

**Banner Grabbing Protocols**:
- HTTP (80, 8080, 443)
- SSH (22)
- FTP (21)
- SMTP (25, 587)
- POP3 (110, 995)
- IMAP4 (143, 993)
- SMB (445)
- RDP (3389)
- VNC (5900)
- SSL/TLS (any port)

### Network Architecture

```
┌─────────────┐         Raw TCP/IP         ┌─────────────┐
│   masscan   │ ─────────────────────────►  │   Target    │
│  (Scanner)  │ ◄─────────────────────────  │   Host      │
└─────────────┘     SYN → SYN-ACK → ACK    └─────────────┘
       │
       ▼
┌─────────────┐
│   Kernel    │
│   (Bypassed)│
└─────────────┘
```

**Architecture Notes**:
- masscan bypasses kernel TCP/IP entirely
- Requires raw socket access (root/CAP_NET_RAW)
- Network interface must be configured (IP address, gateway)
- Firewall rules must allow masscan traffic patterns

---

## Chapter 4: Command Reference

### Command Syntax

```
masscan [options] <targets> [ports]
```

Targets: IP addresses, CIDR ranges, file lists, or ranges (e.g., 192.168.1.1-192.168.1.254)

Ports: -p followed by port list (e.g., -p80,443 or -p1-65535)

### Core Options

| Flag | Description | Default |
|------|-------------|---------|
| `-p <ports>` | Ports to scan | All ports |
| `--rate=<pps>` | Packets per second | 100 |
| `--max-rate=<pps>` | Maximum rate | Unlimited |
| `-oX <file>` | XML output (Nmap format) | None |
| `-oG <file>` | Grepable output | None |
| `-oJ <file>` | JSON output | None |
| `-oL <file>` | List output | None |
| `--banners` | Capture banners | Disabled |
| `--echo` | Show config and exit | None |

### Target Specification

| Flag | Description | Example |
|------|-------------|---------|
| `192.168.1.1` | Single IP | `masscan 192.168.1.1 -p80` |
| `192.168.1.0/24` | CIDR range | `masscan 192.168.1.0/24 -p80` |
| `192.168.1.1-254` | IP range | `masscan 192.168.1.1-254 -p80` |
| `-iL <file>` | Target file | `masscan -iL targets.txt -p80` |
| `--excludefile <f>` | Exclude file | `masscan --excludefile exclude.txt 0.0.0.0/0` |

### Output Options

| Flag | Description | Format |
|------|-------------|--------|
| `-oX <file>` | XML output | Nmap-compatible XML |
| `-oG <file>` | Grepable output | Grep-friendly format |
| `-oJ <file>` | JSON output | JSON array |
| `-oL <file>` | List output | Simple text list |
| `--output-filename <f>` | Alternative output | Any format |

### Network Options

| Flag | Description | Default |
|------|-------------|---------|
| `--source-ip <ip>` | Source IP address | Random |
| `--source-port <port>` | Source port | Random |
| `--adapter <iface>` | Network interface | Auto-detect |
| `--adapter-ip <ip>` | Interface IP | Auto-detect |
| `--router-ip <ip>` | Gateway IP | Auto-detect |

### Performance Options

| Flag | Description | Default |
|------|-------------|---------|
| `--retries <n>` | Retry count | 0 |
| `--shards <n/m>` | Shard configuration | 1/1 |
| `--seed <n>` | Random seed | Time-based |
| `--open` | Show only open ports | Disabled |
| `--offline` | Offline mode | Disabled |

---

## Chapter 5: Usage Examples

### Basic Usage

**Scan single host for HTTP**:
```bash
masscan 192.168.1.1 -p80
```

**Scan subnet for common ports**:
```bash
masscan 192.168.1.0/24 -p80,443,22,21
```

**Scan with banner grabbing**:
```bash
masscan 192.168.1.0/24 -p80 --banners
```

**Scan all ports**:
```bash
masscan 192.168.1.0/24 -p1-65535 --rate=1000
```

### Intermediate Usage

**XML output for Nmap parsing**:
```bash
masscan 192.168.1.0/24 -p80,443 -oX results.xml
```

**Grepable output**:
```bash
masscan 192.168.1.0/24 -p80 -oG results.grep
```

**JSON output**:
```bash
masscan 192.168.1.0/24 -p80 -oJ results.json
```

**Scan with exclusion list**:
```bash
masscan 192.168.1.0/24 -p80 --excludefile exclude.txt
```

**Controlled rate scanning**:
```bash
masscan 192.168.1.0/24 -p80 --rate=5000
```

### Advanced Usage

**Full internet scan simulation**:
```bash
masscan 0.0.0.0/0 -p80 --rate=10000000 --banners -oX internet_scan.xml
```

**Distributed scanning with shards**:
```bash
# Machine 1 of 4
masscan 192.168.1.0/24 -p80 --shards=1/4 --seed=12345

# Machine 2 of 4
masscan 192.168.1.0/24 -p80 --shards=2/4 --seed=12345
```

**Scan specific port range**:
```bash
masscan 192.168.1.0/24 -p1000-2000 --banners
```

**Custom source IP** (requires network support):
```bash
masscan 192.168.1.0/24 -p80 --source-ip=192.168.1.100
```

### Real-World Scenarios

**Enterprise service audit**:
```bash
# Scan corporate network for exposed services
masscan 10.0.0.0/8 -p80,443,3389,22,21 --rate=100000 -oX corp_audit.xml

# Parse results for critical services
grep -E "port=\"(3389|22)\"" corp_audit.xml
```

**Cloud infrastructure mapping**:
```bash
# Scan cloud provider IP ranges
masscan 13.0.0.0/8 -p80,443 --banners --rate=1000000 -oX cloud_scan.xml
```

---

## Chapter 6: Advanced Techniques

### Automation & Scripting

**Batch scanning script**:
```bash
#!/bin/bash
# Automated masscan with rate limiting

TARGETS="192.168.1.0/24 192.168.2.0/24 10.0.0.0/16"
PORTS="80,443,22,21,3389"
RATE=50000
OUTPUT="scan_$(date +%Y%m%d_%H%M%S).xml"

for target in $TARGETS; do
    echo "[*] Scanning $target..."
    masscan "$target" -p "$PORTS" --rate="$RATE" -oX "$OUTPUT" --open
    sleep 10
done

echo "[+] Scan complete: $OUTPUT"
```

**Result parsing script**:
```bash
#!/bin/bash
# Extract open ports from masscan XML output

INPUT="$1"
echo "=== Open Ports Summary ==="
grep -oP 'port="\K[^"]+' "$INPUT" | sort -n | uniq -c | sort -rn
echo ""
echo "=== Hosts with Port 80 ==="
grep -B1 'port="80"' "$INPUT" | grep -oP 'addr="\K[^"]+'
```

### Chaining with Other Tools

**Nmap deep scan from masscan results**:
```bash
# Phase 1: Fast discovery with masscan
masscan 192.168.1.0/24 -p80,443,22 -oX fast_discovery.xml --open

# Phase 2: Detailed scan with Nmap
grep -oP 'addr="\K[^"]+' fast_discovery.xml | sort -u | \
    xargs -I {} nmap -sV -sC -p80,443,22 {} -oN deep_scan_{}.txt
```

**With Python for result processing**:
```python
#!/usr/bin/env python3
import xml.etree.ElementTree as ET
import sys

tree = ET.parse(sys.argv[1])
for host in tree.findall('.//host'):
    addr = host.find('address').get('addr')
    for port in host.findall('.//port'):
        portid = port.get('portid')
        state = port.find('state').get('state')
        if state == 'open':
            print(f"{addr}:{portid}")
```

### Custom Configurations

**Configuration file** (`masscan.conf`):
```bash
rate = 100000
output-format = xml
output-filename = scan_results.xml
ports = 80,443,22
source-ip = 192.168.1.100
source-port = 40000-60000
```

**Advanced configuration**:
```bash
masscan -c config.conf 192.168.1.0/24
```

### Performance Tuning

**Maximize throughput**:
```bash
masscan 192.168.1.0/24 -p80 --rate=10000000 --retries=0
```

**Conservative scanning** (avoid IDS detection):
```bash
masscan 192.168.1.0/24 -p80 --rate=100 --retries=2
```

**Optimize for banner grabbing**:
```bash
masscan 192.168.1.0/24 -p80,443 --banners --rate=10000
```

**PF_RING for maximum performance**:
```bash
# Requires PF_RING kernel module
masscan 192.168.1.0/24 -p80 --rate=2000000 --adapter=pfring0
```

---

## Chapter 7: Detection & Defense

### How to Detect

masscan generates distinctive network patterns:

**High-volume SYN traffic**:
- Massive SYN packet rates from single source
- SYN packets to random ports
- Lack of corresponding ACK/FIN packets (asynchronous)

**Behavioral indicators**:
- Sequential IP scanning patterns
- Unusual source port ranges
- High packet rates exceeding normal application traffic
- SYN packets to non-standard ports

**Signature-based detection**:
- Fixed TCP window size in SYN packets
- Specific TCP options ordering
- Unusual IP ID field patterns

### Defensive Measures

**Network-level defenses**:
- Rate limiting on SYN packets from untrusted sources
- Connection state tracking and validation
- SYN flood protection mechanisms
- Network segmentation and access controls

**Host-level defenses**:
- SYN cookies for connection tracking
- Rate limiting on new connections
- Port knocking for stealth services
- Honeypots for detection and deception

**Firewall rules**:
```bash
# Block high-rate SYN scanning
iptables -A INPUT -p tcp --syn -m limit --limit 10/second --limit-burst 20 -j ACCEPT
iptables -A INPUT -p tcp --syn -j DROP
```

### IDS/IPS Signatures

**Snort rule for masscan detection**:
```
alert tcp any any -> any any (msg:"masscan SYN Scan Detected"; 
    flags:S; 
    threshold:type both, track by_src, count 100, seconds 1; 
    sid:1000002; rev:1;)
```

**Suricata rule**:
```
alert tcp any any -> any any (msg:"High Rate SYN Scan"; 
    flags:S; 
    detection_filter:track by_src, count 50, seconds 1; 
    sid:2000002; rev:1;)
```

### Log Analysis

**Monitoring masscan traffic**:
```bash
# Capture SYN packets
tcpdump -i eth0 'tcp[tcpflags] & tcp-syn != 0' -n -c 1000

# Analyze scan patterns
tcpdump -r scan.pcap -n | awk '{print $3}' | cut -d. -f1-4 | sort | uniq -c
```

**Firewall log analysis**:
```bash
# Monitor blocked connections
iptables -L -n -v | grep -E "DROP|REJECT"

# Analyze connection attempts
netstat -an | grep -E "SYN_SENT|SYN_RECV" | awk '{print $5}' | cut -d: -f1 | sort | uniq -c
```

---

## Chapter 8: Troubleshooting & FAQ

### Common Issues

**"No network interface found"**:
- Verify network interface is up: `ip link show`
- Specify interface manually: `--adapter=eth0`
- Check if interface has IP address assigned

**"Permission denied" error**:
- Run with sudo or as root
- Verify CAP_NET_RAW capability: `sudo setcap cap_net_raw+ep /usr/bin/masscan`
- Check libpcap installation: `apt list --installed | grep libpcap`

**"Invalid port specification"**:
- Verify port format: `-p80,443` or `-p1-1024`
- Check for syntax errors in port list
- Use `--echo` to verify configuration

**Low scan rates**:
- Increase `--rate` parameter
- Check network interface speed
- Verify no packet loss: `masscan --echo | grep rate`
- Check firewall rules blocking traffic

**Missing results**:
- Verify target specification is correct
- Check if targets are reachable
- Use `--open` flag to filter results
- Review exclusion lists

### FAQ

**Q: Can masscan scan UDP ports?**
A: No, masscan is TCP-only. Use nmap for UDP scanning: `nmap -sU -p53,123 target`

**Q: Is masscan undetectable?**
A: No, high-rate scanning is easily detected. Use `--rate=100` for stealthier scanning, but still expect detection by IDS/IPS.

**Q: Does masscan work with VPNs?**
A: Limited. masscan's custom TCP/IP stack may not route through VPN tunnels properly. Scan from inside the VPN network instead.

**Q: How accurate is banner grabbing?**
A: Accurate for standard protocols (HTTP, SSH, FTP). May miss banners on non-standard implementations. Use `--retries=1` for better banner capture.

**Q: Can I scan IPv6 with masscan?**
A: Yes, masscan supports IPv6 scanning. Use standard IPv6 notation: `masscan 2001:db8::/32 -p80`

**Q: What's the difference between masscan and nmap?**
A: masscan prioritizes speed (10M pps vs ~1000 pps for nmap). nmap provides detailed service detection and OS fingerprinting. Use masscan for discovery, nmap for deep analysis.

---

## Resources

### Official Documentation
- [masscan GitHub](https://github.com/robertdavidgraham/masscan)
- [masscan Wiki](https://github.com/robertdavidgraham/masscan/wiki)
- [masscan README](https://raw.githubusercontent.com/robertdavidgraham/masscan/master/README.md)

### Online Resources
- [Internet-Wide Scanning](https://zmap.io/)
- [masscan Performance Tuning](https://github.com/robertdavidgraham/masscan/wiki/Performance)
- [Network Scanning Best Practices](https://attack.mitre.org/techniques/T1046/)

### Related Tools
- **nmap**: Feature-rich network scanner (slower, more detailed)
- **zmap**: Internet-wide ICMP/DNS scanner
- **unicornscan**: Asynchronous scanner (predecessor to masscan)
- **rustscan**: Modern port scanner written in Rust
