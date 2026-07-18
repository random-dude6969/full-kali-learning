---
title: "unicornscan"
description: "Asynchronous TCP/UDP scanner with custom TCP/IP stack for high-speed port scanning and OS fingerprinting"
categories: ["Network Service Discovery"]
tags: ["port-scanning", "async", "tcp", "udp", "os-fingerprint", "high-speed", "custom-stack"]
mitre_tactic: "discovery"
mitre_tactic_id: "TA0007"
mitre_subcategory: "network_service_discovery"
install_command: "sudo apt install unicornscan"
version: "0.4.7"
github_repo: "https://github.com/netsniff-ng/unicornscan"
kali_tools_page: "https://www.kali.org/tools/unicornscan/"
difficulty_level: "intermediate"
requires_root: true
gui_available: false
related_tools: ["masscan", "nmap", "zmap", "rustscan"]
---

# unicornscan

## Chapter 1: Introduction & Overview

### What is unicornscan?

unicornscan is an asynchronous TCP/UDP port scanner that implements its own TCP/IP stack in userspace, bypassing the operating system kernel. This architecture enables high-speed scanning with accurate results. The tool supports TCP connect scanning, SYN scanning, UDP scanning, and ICMP-based OS fingerprinting. unicornscan is the predecessor to modern high-speed scanners like masscan and zmap.

Unlike traditional scanners that rely on the kernel's TCP/IP stack, unicornscan constructs and transmits raw packets directly, achieving scanning speeds far exceeding nmap while maintaining accuracy. The tool is designed for network reconnaissance where both speed and reliability are required.

### History & Background

unicornscan was developed by Marc Beier and released in 2006. It pioneered the concept of userspace TCP/IP stacks for scanning, influencing the development of later tools like masscan.

Key milestones:
- **2006**: Initial release with asynchronous scanning
- **2008**: Added UDP scanning and OS fingerprinting
- **2010**: Integration with Kali Linux (BackTrack)
- **2012**: Improved banner grabbing capabilities
- **2015**: Migration to netsniff-ng organization
- **2020+**: Maintenance updates, compatibility improvements

### When to Use This Tool

- **High-speed port scanning**: Scan large networks quickly
- **UDP service discovery**: Find UDP services that TCP scanners miss
- **OS fingerprinting**: Identify operating systems via TCP/IP stack behavior
- **Banner grabbing**: Collect service banners during scanning
- **Network auditing**: Comprehensive service discovery across subnets
- **Penetration testing**: Initial reconnaissance phase scanning

### Key Concepts

**Asynchronous Scanning**: unicornscan sends probes without waiting for responses, then processes all responses asynchronously. This maximizes throughput by eliminating the wait state.

**Userspace TCP/IP Stack**: The tool implements its own TCP/IP stack, bypassing kernel limitations. This enables raw packet construction and transmission at wire speed.

**TCP Scanning Modes**: unicornscan supports multiple TCP scanning techniques:
- `-pT`: TCP connect scan (full handshake)
- `-pS`: TCP SYN scan (half-open)
- `-pA`: TCP ACK scan
- `-pF`: TCP FIN scan
- `-pX`: TCP Xmas scan

**UDP Scanning**: UDP scanning (`-pU`) is supported without requiring kernel UDP stack interaction, making it faster than traditional UDP scanners.

**OS Fingerprinting**: The `-I` flag enables OS detection based on TCP/IP stack behavior, including TCP window size, TTL, and TCP options.

---

## Chapter 2: Installation & Setup

### System Requirements

**Operating System**: Kali Linux (primary), Debian/Ubuntu, or any Linux distribution

**Dependencies**:
- libpcap (packet capture)
- libnet (packet construction)
- Root or CAP_NET_RAW capability

**Hardware**: Standard network interface. No special requirements.

### Installation Methods

**From Kali repositories**:
```bash
sudo apt update
sudo apt install unicornscan
```

**From source**:
```bash
git clone https://github.com/netsniff-ng/unicornscan.git
cd unicornscan
./configure
make
sudo make install
```

**Binary distribution**:
```bash
# Download and install manually
wget http://unicornscan.org/downloads/unicornscan-0.4.7.tar.bz2
tar -xjf unicornscan-0.4.7.tar.bz2
cd unicornscan-0.4.7
./configure && make && sudo make install
```

### Configuration

unicornscan uses command-line options exclusively. No configuration files are required. The tool operates with sensible defaults:

- **Default mode**: TCP connect scan
- **Default ports**: Common ports (if not specified)
- **Default timeout**: 7000 milliseconds
- **Default retries**: 3 attempts
- **Default rate**: 150 packets/second

### Verifying Installation

```bash
# Check version
unicornscan --version

# Verify binary location
which unicornscan

# Test with localhost (safe)
unicornscan 127.0.0.1

# Display help
unicornscan --help
```

---

## Chapter 3: Core Concepts & Architecture

### How It Works

unicornscan constructs raw packets and transmits them asynchronously. The tool maintains its own TCP state machine, handling handshake completion independently of the kernel.

**Scanning Process**:

1. **Interface Configuration**: unicornscan identifies the network interface and configures raw socket access
2. **Target Processing**: Target specifications are parsed and queued for scanning
3. **Probe Generation**: TCP/UDP/ICMP probes are constructed with configurable parameters
4. **Asynchronous Transmission**: Probes are sent at configured rate without waiting for responses
5. **Response Collection**: Responses are captured and processed asynchronously
6. **Result Analysis**: Open ports, services, and OS fingerprints are identified
7. **Result Reporting**: Findings are displayed in configured format

**TCP SYN Scan Workflow**:

```
┌─────────────┐         TCP SYN           ┌─────────────┐
│ unicornscan │ ────────────────────────►  │   Target    │
│  (Scanner)  │ ◄────────────────────────  │   Host      │
└─────────────┘     SYN-ACK → RST         └─────────────┘
       │
       ▼
┌─────────────┐
│   Analysis  │
│   OS Detect │
│   Banner    │
└─────────────┘
```

### Protocols & Standards

| Protocol | Usage | Scanning Mode |
|----------|-------|---------------|
| TCP | Primary scan protocol | -pT, -pS, -pA, -pF, -pX |
| UDP | UDP service discovery | -pU |
| ICMP | OS fingerprinting | -I |
| ARP | Local network discovery | -m |

**TCP Scan Types**:

| Flag | Type | Description |
|------|------|-------------|
| `-pT` | Connect | Full TCP handshake |
| `-pS` | SYN | Half-open scan |
| `-pA` | ACK | Firewall mapping |
| `-pF` | FIN | Stealth scan |
| `-pX` | Xmas | FIN/PSH/URG flags |

### Network Architecture

**Asynchronous Architecture**:
```
┌─────────────┐     ┌─────────────────┐     ┌─────────────┐
│  unicornscan│ ──► │  Userspace TCP   │ ──► │  Network    │
│  (Scanner)  │     │ /IP Stack        │     │  Interface  │
└─────────────┘     └─────────────────┘     └─────────────┘
       │                    │                       │
       │                    │                       │
       ▼                    ▼                       ▼
┌─────────────┐     ┌─────────────────┐     ┌─────────────┐
│  Result     │ ◄── │  Response       │ ◄── │  Target     │
│  Analysis   │     │  Collection     │     │  Host       │
└─────────────┘     └─────────────────┘     └─────────────┘
```

---

## Chapter 4: Command Reference

### Command Syntax

```
unicornscan [options] <targets> [ports]
```

Targets: IP addresses, CIDR ranges, or file-based lists

Ports: -p followed by protocol prefix and port list

### Core Options

| Flag | Description | Default |
|------|-------------|---------|
| `-pT<ports>` | TCP connect scan | Default mode |
| `-pS<ports>` | TCP SYN scan | None |
| `-pU<ports>` | UDP scan | None |
| `-pA<ports>` | TCP ACK scan | None |
| `-pF<ports>` | TCP FIN scan | None |
| `-pX<ports>` | TCP Xmas scan | None |
| `-I` | OS fingerprinting | Disabled |
| `-m <mode>` | Scan mode | Default |

### Scan Mode Options

| Flag | Description |
|------|-------------|
| `-mT` | TCP mode (default) |
| `-mU` | UDP mode |
| `-mI` | ICMP mode |
| `-mA` | ARP mode |

### Port Specification

| Syntax | Description | Example |
|--------|-------------|---------|
| `-pT80` | Single TCP port | `unicornscan -pT80 192.168.1.1` |
| `-pT80,443` | Multiple ports | `unicornscan -pT80,443 192.168.1.1` |
| `-pT1-1024` | Port range | `unicornscan -pT1-1024 192.168.1.1` |
| `-pU53,123` | UDP ports | `unicornscan -pU53,123 192.168.1.1` |

### Performance Options

| Flag | Description | Default |
|------|-------------|---------|
| `-r <pps>` | Rate (packets/sec) | 150 |
| `-T <ms>` | Timeout | 7000 |
| `-R <n>` | Retries | 3 |

### Output Options

| Flag | Description |
|------|-------------|
| `-l` | Log output to file |
| `-v` | Verbose output |
| `-V` | Very verbose output |
| `-q` | Quiet mode |
| `-I` | OS fingerprint |

### Advanced Options

| Flag | Description |
|------|-------------|
| `-s <ip>` | Source IP address |
| `-S <port>` | Source port |
| `-d <delay>` | Delay between probes |
| `-w <wait>` | Wait time for responses |

---

## Chapter 5: Usage Examples

### Basic Usage

**Scan single host for HTTP**:
```bash
sudo unicornscan -pT80 192.168.1.1
```

**Scan subnet for common ports**:
```bash
sudo unicornscan -pT80,443,22,21 192.168.1.0/24
```

**Scan with OS fingerprinting**:
```bash
sudo unicornscan -pT80 -I 192.168.1.1
```

**Scan all TCP ports**:
```bash
sudo unicornscan -pT1-65535 192.168.1.1
```

### Intermediate Usage

**UDP scan**:
```bash
sudo unicornscan -pU53,123,161 192.168.1.0/24
```

**SYN scan**:
```bash
sudo unicornscan -pS80,443 192.168.1.1
```

**ACK scan** (firewall mapping):
```bash
sudo unicornscan -pA80,443 192.168.1.1
```

**FIN scan**:
```bash
sudo unicornscan -pF80,443 192.168.1.1
```

**Xmas scan**:
```bash
sudo unicornscan -pX80,443 192.168.1.1
```

### Advanced Usage

**High-speed scanning**:
```bash
sudo unicornscan -pT1-65535 -r 10000 192.168.1.0/24
```

**Stealth scanning**:
```bash
sudo unicornscan -pS80 -r 10 -T 10000 192.168.1.1
```

**OS fingerprinting with banner grab**:
```bash
sudo unicornscan -pT80,443 -I -v 192.168.1.1
```

**Scan from file with logging**:
```bash
sudo unicornscan -pT80,443 -l results.log -iL targets.txt
```

**Custom source IP**:
```bash
sudo unicornscan -pT80 -s 192.168.1.100 192.168.1.1
```

### Real-World Scenarios

**Network service audit**:
```bash
# Comprehensive TCP scan
sudo unicornscan -pT1-1024 -I -r 500 192.168.1.0/24 > audit.txt

# UDP service discovery
sudo unicornscan -pU53,67,68,123,161,162,500,514 192.168.1.0/24 >> audit.txt
```

**Firewall testing**:
```bash
# Map firewall rules with ACK scan
sudo unicornscan -pA1-1024 192.168.1.1

# Test stealth with FIN scan
sudo unicornscan -pF1-1024 192.168.1.1
```

**OS detection**:
```bash
# Identify operating systems on network
sudo unicornscan -pT80 -I -r 100 192.168.1.0/24
```

---

## Chapter 6: Advanced Techniques

### Automation & Scripting

**Automated network audit script**:
```bash
#!/bin/bash
# Comprehensive network audit with unicornscan

TARGETS="192.168.1.0/24 192.168.2.0/24"
TCP_PORTS="1-1024"
UDP_PORTS="53,67,68,123,161,162,500,514"
OUTPUT="audit_$(date +%Y%m%d).txt"

for target in $TARGETS; do
    echo "[*] TCP scan: $target"
    unicornscan -pT"$TCP_PORTS" -I -r 500 "$target" >> "$OUTPUT"
    
    echo "[*] UDP scan: $target"
    unicornscan -pU"$UDP_PORTS" -r 100 "$target" >> "$OUTPUT"
done

echo "[+] Audit complete: $OUTPUT"
```

**Result parsing script**:
```bash
#!/bin/bash
# Parse unicornscan output for services

INPUT="$1"
echo "=== Open Ports Summary ==="
grep -oP 'tcp:\K\d+' "$INPUT" | sort -n | uniq -c | sort -rn
echo ""
echo "=== OS Fingerprints ==="
grep -i "OS" "$INPUT"
```

### Chaining with Other Tools

**Nmap integration**:
```bash
# Phase 1: Fast discovery with unicornscan
sudo unicornscan -pT80,443,22 -r 1000 192.168.1.0/24 > discovery.txt

# Phase 2: Deep scan with nmap
grep -oP '\d+\.\d+\.\d+\.\d+' discovery.txt | sort -u | \
    xargs -I {} nmap -sV -sC -p80,443,22 {} -oN deep_{}.txt
```

**With masscan for comparison**:
```bash
# Compare results from different scanners
sudo unicornscan -pT80 192.168.1.0/24 > unicorn_results.txt
sudo masscan 192.168.1.0/24 -p80 -oG masscan_results.txt
diff <(sort unicorn_results.txt) <(sort masscan_results.txt)
```

### Custom Configurations

**Custom rate and timeout**:
```bash
sudo unicornscan -pT80 -r 100 -T 10000 192.168.1.1
```

**Custom source configuration**:
```bash
sudo unicornscan -pT80 -s 192.168.1.100 -S 40000 192.168.1.1
```

### Performance Tuning

**Maximize throughput**:
```bash
sudo unicornscan -pT1-65535 -r 10000 -T 5000 192.168.1.0/24
```

**Conservative scanning**:
```bash
sudo unicornscan -pT80,443 -r 50 -T 10000 192.168.1.0/24
```

**Optimize for UDP**:
```bash
sudo unicornscan -pU53,123,161 -r 100 -T 5000 192.168.1.0/24
```

---

## Chapter 7: Detection & Defense

### How to Detect

unicornscan generates detectable network patterns:

**Signature-based detection**:
- Multiple TCP/UDP probes from single source
- Unusual TCP flag combinations (FIN, Xmas scans)
- High packet rates exceeding normal traffic
- Specific TCP window size and TTL values

**Behavioral detection**:
- Sequential port scanning patterns
- Rapid connection attempts to multiple hosts
- Unusual source port usage
- ICMP responses indicating OS fingerprinting

### Defensive Measures

**Network-level defenses**:
- Rate limiting on new connections
- Connection state tracking and validation
- SYN flood protection mechanisms
- Network segmentation and access controls

**Host-level defenses**:
- SYN cookies for connection tracking
- Rate limiting on new connections
- Port knocking for stealth services
- Honeypots for detection and deception

### IDS/IPS Signatures

**Snort rule for unicornscan detection**:
```
alert tcp any any -> any any (msg:"unicornscan SYN Scan Detected"; 
    flags:S; 
    threshold:type both, track by_src, count 50, seconds 1; 
    sid:1000004; rev:1;)
```

**Suricata rule**:
```
alert tcp any any -> any any (msg:"Stealth Scan Detected"; 
    flags:F,12; 
    threshold:type both, track by_src, count 10, seconds 60; 
    sid:2000004; rev:1;)
```

### Log Analysis

**Monitoring scanning traffic**:
```bash
# Capture scanning packets
tcpdump -i eth0 'tcp[tcpflags] & (tcp-syn|tcp-fin|tcp-urg) != 0' -n

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
- Specify interface manually: `-i eth0`
- Check if interface has IP address assigned

**"Permission denied" error**:
- Run with sudo or as root
- Verify CAP_NET_RAW capability: `sudo setcap cap_net_raw+ep /usr/bin/unicornscan`
- Check libpcap installation: `apt list --installed | grep libpcap`

**"Invalid port specification"**:
- Verify port format: `-pT80,443` or `-pT1-1024`
- Check for syntax errors in port list
- Use `-pT` prefix for TCP, `-pU` for UDP

**Low scan rates**:
- Increase `-r` parameter
- Check network interface speed
- Verify no packet loss
- Check firewall rules blocking traffic

**Missing results**:
- Verify target specification is correct
- Check if targets are reachable
- Use `-v` flag for verbose output
- Review timeout settings

### FAQ

**Q: Is unicornscan faster than nmap?**
A: Yes, unicornscan is significantly faster due to its asynchronous architecture and userspace TCP/IP stack. However, nmap provides more detailed service detection and OS fingerprinting.

**Q: Does unicornscan work with UDP?**
A: Yes, unicornscan supports UDP scanning with `-pU` flag. UDP scanning is faster than nmap's UDP scanning due to the userspace stack.

**Q: Can unicornscan do OS fingerprinting?**
A: Yes, use `-I` flag for OS detection based on TCP/IP stack behavior. Results are accurate for common operating systems.

**Q: Is unicornscan detectable?**
A: Yes, high-rate scanning is easily detected. Use `-r 10` for stealthier scanning, but expect detection by IDS/IPS systems.

**Q: What's the difference between unicornscan and masscan?**
A: Both use userspace TCP/IP stacks. masscan is newer and faster (10M pps vs 1M pps). unicornscan has more scanning modes (FIN, Xmas, ACK) and better OS fingerprinting.

**Q: Does unicornscan support IPv6?**
A: Limited IPv6 support. For comprehensive IPv6 scanning, use nmap or masscan.

---

## Resources

### Official Documentation
- [unicornscan GitHub](https://github.com/netsniff-ng/unicornscan)
- [unicornscan Documentation](https://github.com/netsniff-ng/unicornscan/blob/master/README)
- [netsniff-ng Project](https://netsniff-ng.org/)

### Online Resources
- [Asynchronous Scanning Techniques](https://www.sans.org/white-papers/asynchronous-scanning/)
- [Port Scanning Best Practices](https://attack.mitre.org/techniques/T1046/)
- [Network Reconnaissance](https://attack.mitre.org/tactics/TA0043/)

### Related Tools
- **masscan**: High-speed scanner (faster than unicornscan)
- **nmap**: Feature-rich scanner (slower, more detailed)
- **zmap**: Internet-wide ICMP/DNS scanner
- **rustscan**: Modern scanner written in Rust
