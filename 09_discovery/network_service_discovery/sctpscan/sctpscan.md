---
title: "sctpscan"
description: "SCTP port scanner for discovering telecom protocols and multihomed hosts"
categories: ["Network Service Discovery"]
tags: ["sctp", "telecom", "ss7", "sigua", "diameter", "m3ua", "multihoming"]
mitre_tactic: "discovery"
mitre_tactic_id: "TA0007"
mitre_subcategory: "network_service_discovery"
install_command: "sudo apt install sctpscan"
version: "0.4"
github_repo: ""
kali_tools_page: "https://www.kali.org/tools/sctpscan/"
difficulty_level: "intermediate"
requires_root: true
gui_available: false
related_tools: ["nmap", "hping3", "sctp_client"]
---

# sctpscan

## Chapter 1: Introduction & Overview

### What is sctpscan?

sctpscan is a command-line tool that discovers SCTP (Stream Control Transmission Protocol) services on networks. It sends SCTP INIT packets to target hosts and analyzes responses to identify open SCTP ports, supported protocols, and multihomed network configurations. The tool is essential for auditing telecommunications infrastructure where SCTP is the primary transport protocol.

SCTP is a transport-layer protocol designed for signaling applications in telecommunications networks. Unlike TCP and UDP, SCTP supports multihoming (multiple IP addresses per association) and multi-streaming (multiple independent message streams within a single association). These features make SCTP critical for SS7/SIGUA, Diameter, M3UA, and other telecom protocols.

### History & Background

sctpscan was developed to address the need for SCTP-aware scanning tools in telecom security assessments. Traditional port scanners like nmap have limited SCTP support, making specialized tools necessary for comprehensive telecom infrastructure audits.

Key milestones:
- **2005**: Initial SCTP implementation in Linux kernel (LKSCTP project)
- **2008**: First SCTP scanning tools developed for security research
- **2012**: sctpscan released as standalone utility
- **2015**: Integration with Kali Linux for telecom pentesting
- **2020+**: Improved multihoming detection and protocol fingerprinting

### When to Use This Tool

- **Telecom infrastructure auditing**: Discover SCTP services in SS7/SIGUA networks
- **Protocol gateway scanning**: Identify Diameter, M3UA, and other SCTP-based services
- **Multihoming detection**: Find hosts with multiple IP addresses configured for SCTP
- **Security assessments**: Test SCTP service exposure and misconfigurations
- **Compliance checks**: Verify telecom network security configurations
- **Penetration testing**: Enumerate SCTP endpoints for signaling attacks

### Key Concepts

**SCTP Protocol**: SCTP provides reliable, ordered delivery of data like TCP, but with built-in support for multihoming and multi-streaming. It operates over IP protocol 139 (or IP protocol 132 for some implementations).

**INIT Packet**: The SCTP equivalent of a TCP SYN packet. Initiates an SCTP association and triggers an INIT-ACK response from the target.

**Multihoming**: SCTP allows a single association to use multiple IP addresses. This provides fault tolerance and load balancing. sctpscan detects multihomed hosts by analyzing INIT-ACK responses.

**Telecom Protocols**: SCTP is the transport for SS7/SIGUA (signaling), Diameter (authentication), M3UA (MTP3 user adaptation), and H.248 (media gateway control). sctpscan identifies these protocols through port analysis.

---

## Chapter 2: Installation & Setup

### System Requirements

**Operating System**: Kali Linux (primary), Debian/Ubuntu, or any Linux distribution with LKSCTP support

**Dependencies**:
- Linux kernel with SCTP support (lksctp-tools)
- libpcap (packet capture)
- Root or CAP_NET_RAW capability

**Hardware**: Network interface supporting SCTP traffic. No special requirements.

**Kernel SCTP Support**:
```bash
# Verify SCTP kernel module is loaded
lsmod | grep sctp

# Load SCTP module if needed
sudo modprobe sctp
```

### Installation Methods

**From Kali repositories**:
```bash
sudo apt update
sudo apt install sctpscan
```

**Install SCTP tools first**:
```bash
sudo apt install lksctp-tools
sudo apt install sctpscan
```

**From source**:
```bash
git clone https://github.com/sctp/sctpscan.git
cd sctpscan
./configure
make
sudo make install
```

### Configuration

sctpscan uses command-line options exclusively. No configuration files are required. The tool operates with sensible defaults:

- **Default port**: Any (if not specified)
- **Default timeout**: 3000 milliseconds
- **Default source IP**: Auto-detect from network interface
- **Default protocol**: SCTP over IPv4

### Verifying Installation

```bash
# Check version
sctpscan --version

# Verify binary location
which sctpscan

# Test SCTP kernel support
lsmod | grep sctp

# Test with localhost (safe)
sctpscan 127.0.0.1
```

---

## Chapter 3: Core Concepts & Architecture

### How It Works

sctpscan constructs SCTP INIT packets and sends them to target hosts. The tool listens for INIT-ACK responses to identify open SCTP ports and analyze the target's SCTP configuration.

**Scanning Process**:

1. **Packet Construction**: sctpscan builds an SCTP INIT packet with configurable verification tag and initialization parameters
2. **Transmission**: Packets are sent to target IP on specified SCTP port
3. **Response Collection**: The tool captures INIT-ACK responses using libpcap
4. **Payload Analysis**: INIT-ACK payloads are decoded to extract supported parameters, addresses, and capabilities
5. **Multihoming Detection**: Multiple IP addresses in INIT-ACK indicate multihomed configuration
6. **Result Reporting**: Findings include open ports, protocol identification, and multihoming status

**SCTP Association Setup**:

```
┌─────────────┐         SCTP INIT          ┌─────────────┐
│  sctpscan   │ ──────────────────────────► │   Target    │
│  (Scanner)  │ ◄─────────────────────────  │   Host      │
└─────────────┘     INIT-ACK Response       └─────────────┘
       │
       ▼
┌─────────────┐
│   Analysis  │
│   Protocol  │
│   Detection │
└─────────────┘
```

### Protocols & Standards

| Protocol | RFC | Usage | Port |
|----------|-----|-------|------|
| SCTP | RFC 4960 | Transport protocol | IP 139 |
| SS7/SIGUA | Q.711-Q.716 | Signaling | 2904, 2905 |
| Diameter | RFC 6733 | Authentication | 3868 |
| M3UA | RFC 4666 | MTP3 adaptation | 2904 |
| H.248 | ITU-T H.248 | Media gateway | 2944 |

**SCTP over UDP (encapsulated)**:
Some implementations use SCTP over UDP for NAT traversal. sctpscan can target these with UDP encapsulation.

### Network Architecture

**Telecom Signaling Network**:
```
┌─────────────┐      SCTP/SS7       ┌─────────────┐
│   STP       │ ◄─────────────────► │   STP       │
│ (Signaling) │                     │ (Signaling) │
└─────────────┘                     └─────────────┘
       │                                   │
       │ SCTP/Diameter                     │ SCTP/M3UA
       ▼                                   ▼
┌─────────────┐                     ┌─────────────┐
│   HLR/HSS   │                     │   MSC       │
│ (Subscriber)│                     │ (Switch)    │
└─────────────┘                     └─────────────┘
```

---

## Chapter 4: Command Reference

### Command Syntax

```
sctpscan [options] <targets> [ports]
```

Targets: IP addresses or CIDR ranges

Ports: -p followed by port list (e.g., -p2904,3868)

### Core Options

| Flag | Description | Default |
|------|-------------|---------|
| `-p <ports>` | Ports to scan | All ports |
| `-S <ip>` | Source IP address | Auto-detect |
| `-T <ms>` | Timeout in ms | 3000 |
| `-r <pps>` | Rate (packets/sec) | 10 |
| `-v` | Verbose output | Disabled |
| `-vv` | Very verbose | Disabled |

### Target Specification

| Flag | Description | Example |
|------|-------------|---------|
| `192.168.1.1` | Single IP | `sctpscan 192.168.1.1 -p2904` |
| `192.168.1.0/24` | CIDR range | `sctpscan 192.168.1.0/24 -p2904` |
| `-l <file>` | Target file | `sctpscan -l targets.txt -p2904` |

### SCTP Options

| Flag | Description | Default |
|------|-------------|---------|
| `--vtag=<hex>` | Verification tag | Random |
| `--init-tsn=<n>` | Initial TSN | Random |
| `--a-rwnd=<n>` | Receiver window | 65535 |
| `--num-outbound=<n>` | Outbound streams | 1 |
| `--num-inbound=<n>` | Inbound streams | 1 |

### Output Options

| Flag | Description |
|------|-------------|
| `-o <file>` | Write output to file |
| `-q` | Quiet mode |

---

## Chapter 5: Usage Examples

### Basic Usage

**Scan single host for SCTP**:
```bash
sudo sctpscan 192.168.1.1
```

**Scan specific SCTP port**:
```bash
sudo sctpscan -p2904 192.168.1.1
```

**Scan subnet for SS7 ports**:
```bash
sudo sctpscan -p2904,2905 192.168.1.0/24
```

**Scan with verbose output**:
```bash
sudo sctpscan -v 192.168.1.0/24
```

### Intermediate Usage

**Scan Diameter ports**:
```bash
sudo sctpscan -p3868 192.168.1.0/24
```

**Scan with custom source IP**:
```bash
sudo sctpscan -S 192.168.1.100 -p2904 192.168.1.0/24
```

**Scan with increased timeout**:
```bash
sudo sctpscan -T 5000 -p2904 192.168.1.1
```

**Scan from target file**:
```bash
sudo sctpscan -l telecom_hosts.txt -p2904,3868
```

### Advanced Usage

**Scan M3UA ports**:
```bash
sudo sctpscan -p2904 -v 10.0.0.0/8
```

**Multihoming detection**:
```bash
sudo sctpscan -p2904 -vv 192.168.1.0/24
```

**Scan H.248 media gateway ports**:
```bash
sudo sctpscan -p2944 192.168.1.0/24
```

**Batch scan with output**:
```bash
sudo sctpscan -p2904,3868 -o sctp_results.txt 10.0.0.0/16
```

### Real-World Scenarios

**Telecom network audit**:
```bash
# Discover all SCTP services in core network
sudo sctpscan -p2904,2905,3868,2944 -v 10.0.1.0/24 > telecom_audit.txt

# Verify no unauthorized SCTP exposure
sudo sctpscan -p2904 192.168.0.0/16
```

**SS7/SIGUA assessment**:
```bash
# Map SS7 signaling infrastructure
sudo sctpscan -p2904,2905 -vv 10.0.0.0/8

# Identify multihomed signaling points
sudo sctpscan -p2904 -vv 172.16.0.0/12
```

---

## Chapter 6: Advanced Techniques

### Automation & Scripting

**Telecom network discovery script**:
```bash
#!/bin/bash
# Discover SCTP services across telecom infrastructure

SUBNETS="10.0.1.0/24 10.0.2.0/24 172.16.1.0/24"
PORTS="2904,2905,3868,2944"
OUTPUT="telecom_$(date +%Y%m%d).txt"

for subnet in $SUBNETS; do
    echo "[*] Scanning $subnet for SCTP services..."
    sctpscan -p "$PORTS" -v "$subnet" >> "$OUTPUT"
done

echo "[+] Results saved to $OUTPUT"
```

**Protocol-specific scanning**:
```bash
#!/bin/bash
# Scan for specific telecom protocols

# SS7/SIGUA ports
echo "[*] Scanning for SS7/SIGUA..."
sctpscan -p2904,2905 10.0.0.0/8

# Diameter ports
echo "[*] Scanning for Diameter..."
sctpscan -p3868 10.0.0.0/8

# H.248 ports
echo "[*] Scanning for H.248..."
sctpscan -p2944 10.0.0.0/8
```

### Chaining with Other Tools

**Nmap SCTP integration**:
```bash
# Discover SCTP hosts with sctpscan
sudo sctpscan -p2904 192.168.1.0/24 | grep -oP '\d+\.\d+\.\d+\.\d+' | \
    xargs -I {} nmap -sY -p2904,2905,3868 {} -oN nmap_sctp_{}.txt
```

**With hping3 for SCTP**:
```bash
# Verify SCTP services found by sctpscan
sudo hping3 -S -p 2904 192.168.1.1
```

**With tcpdump for analysis**:
```bash
# Capture SCTP traffic during scan
sudo tcpdump -i eth0 proto sctp -w sctp_capture.pcap &
sudo sctpscan -p2904 192.168.1.0/24
```

### Custom Configurations

**Custom verification tag**:
```bash
sudo sctpscan --vtag=0x12345678 -p2904 192.168.1.1
```

**Custom initialization parameters**:
```bash
sudo sctpscan --a-rwnd=131072 --num-outbound=2 --num-inbound=2 -p2904 192.168.1.1
```

### Performance Tuning

**Rate limiting for large networks**:
```bash
sudo sctpscan -p2904 -r 100 10.0.0.0/8
```

**Increased timeout for slow networks**:
```bash
sudo sctpscan -p2904 -T 5000 -r 5 192.168.1.0/24
```

---

## Chapter 7: Detection & Defense

### How to Detect

SCTP scanning generates detectable network patterns:

**Signature-based detection**:
- Multiple SCTP INIT packets from single source
- High volume of INIT-ACK responses
- SCTP traffic to non-standard ports
- Unusual verification tag patterns

**Behavioral detection**:
- Sequential scanning across subnets
- Traffic to known telecom ports from non-telecom sources
- INIT packets with unusual initialization parameters

### Defensive Measures

**Network-level defenses**:
- Restrict SCTP access to authorized telecom systems
- Use firewall rules to filter SCTP from untrusted sources
- Implement network segmentation for signaling infrastructure
- Deploy IDS/IPS rules for SCTP scanning detection

**Host-level defenses**:
- Disable SCTP if not needed
- Configure SCTP firewalls (e.g., iptables rules for IP protocol 139)
- Monitor SCTP connection attempts
- Implement access control lists for signaling ports

### IDS/IPS Signatures

**iptables rule for SCTP**:
```bash
# Block unauthorized SCTP traffic
sudo iptables -A INPUT -p sctp -j DROP
sudo iptables -A INPUT -p sctp --dport 2904 -s 10.0.0.0/8 -j ACCEPT
```

**Snort rule example**:
```
alert sctp any any -> any any (msg:"SCTP Scan Detected"; 
    content:"|01 00 00 00|"; 
    threshold:type both, track by_src, count 10, seconds 60; 
    sid:1000003; rev:1;)
```

### Log Analysis

**Monitoring SCTP connections**:
```bash
# Monitor SCTP traffic
tcpdump -i eth0 proto sctp -n

# Check SCTP connections
ss -a | grep sctp

# Analyze SCTP logs
grep -i sctp /var/log/syslog
```

**Firewall log analysis**:
```bash
# Log SCTP connection attempts
iptables -A INPUT -p sctp -j LOG --log-prefix "SCTP: "
tail -f /var/log/kern.log | grep SCTP
```

---

## Chapter 8: Troubleshooting & FAQ

### Common Issues

**"No SCTP kernel support"**:
- Install lksctp-tools: `sudo apt install lksctp-tools`
- Load SCTP module: `sudo modprobe sctp`
- Verify with: `lsmod | grep sctp`

**"Permission denied" error**:
- Run with sudo or as root
- Verify CAP_NET_RAW capability
- Check libpcap installation

**"No responses received"**:
- Target may not be running SCTP services
- Firewall blocking SCTP traffic
- Try different ports (2904, 3868, 2944)
- Increase timeout: `-T 5000`

**"Invalid SCTP packet"**:
- Network translation issues
- SCTP over UDP encapsulation
- Try different source IP: `-S 192.168.1.100`

**Inconsistent results**:
- Network congestion
- SCTP load balancing
- Try reduced rate: `-r 5`

### FAQ

**Q: Does sctpscan work over UDP encapsulation?**
A: Limited support. Some SCTP implementations use UDP encapsulation for NAT traversal. sctpscan primarily targets native SCTP over IP protocol 139.

**Q: Can sctpscan identify specific telecom protocols?**
A: sctpscan identifies SCTP services. Protocol identification (SS7, Diameter, M3UA) requires deeper analysis of application-layer payloads, which may require specialized tools.

**Q: How accurate is multihoming detection?**
A: Highly accurate for hosts responding with multiple IP addresses in INIT-ACK. Some implementations may not expose all configured addresses.

**Q: Does sctpscan work against SCTP over DTLS?**
A: No. sctpscan targets native SCTP. SCTP over DTLS requires different tools.

**Q: Is SCTP common outside telecom?**
A: SCTP is primarily used in telecom signaling. Some webRTC implementations use SCTP over DTLS, but traditional SCTP scanning is telecom-specific.

---

## Resources

### Official Documentation
- [sctpscan GitHub](https://github.com/sctp/sctpscan)
- [Linux SCTP Project (LKSCTP)](https://sourceforge.net/projects/lksctp/)
- [SCTP RFC 4960](https://tools.ietf.org/html/rfc4960)

### Online Resources
- [Telecom Security Assessment](https://www.coresecurity.com/technologies/telecom-security)
- [SS7 Security Analysis](https://www.accellion.com/blog/ss7-security/)
- [SCTP Protocol Analysis](https://en.wikipedia.org/wiki/Stream_Control_Transmission_Protocol)

### Related Tools
- **nmap**: Network scanner with SCTP scripts
- **hping3**: Packet crafter with SCTP support
- **sctp_client**: SCTP client for testing
- **sctp_test**: SCTP testing utility
