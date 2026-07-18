---
title: "ike-scan"
description: "IKE scanner that detects and fingerprints VPN concentrators by sending IKE Phase 1 proposals"
categories: ["Network Service Discovery"]
tags: ["ike", "vpn", "ipsec", "phase1", "aggressive-mode", "main-mode", "fingerprinting"]
mitre_tactic: "discovery"
mitre_tactic_id: "TA0007"
mitre_subcategory: "network_service_discovery"
install_command: "sudo apt install ike-scan"
version: "1.9"
github_repo: "https://github.com/royhills/ike-scan"
kali_tools_page: "https://www.kali.org/tools/ike-scan/"
difficulty_level: "intermediate"
requires_root: true
gui_available: false
related_tools: ["ipsec-scan", "ikeforce", "nmap", "strongswan"]
---

# ike-scan

## Chapter 1: Introduction & Overview

### What is ike-scan?

ike-scan is a command-line tool that discovers and fingerprints IKE (Internet Key Exchange) hosts on a network. It sends IKE Phase 1 proposals to target hosts and analyzes the responses to identify VPN products and configurations. The tool is the standard utility for auditing IPSec VPN deployments, testing for misconfigured VPN endpoints, and enumerating VPN infrastructure during penetration tests.

IKE is the key exchange protocol used in IPSec VPNs. It establishes a secure channel between two peers before the actual IPSec tunnel is created. ike-scan exploits this by sending crafted IKE_SA_INIT or IKE_SA packets and analyzing the response payloads to determine the VPN software, supported transforms, and authentication methods.

### History & Background

ike-scan was developed by Roy Hills and released in 2003 as the first open-source IKE scanning tool. Before ike-scan, identifying VPN endpoints required manual IKE protocol interaction or expensive commercial scanners. The tool popularized IKE fingerprinting and established the methodology for passive and active VPN detection.

Key milestones:
- **2003**: Initial release, supported IKEv1 Main and Aggressive modes
- **2006**: Added IKEv2 support and fingerprint database
- **2010**: PSK cracking mode added
- **2015**: Active development on GitHub, community fingerprint contributions
- **2020+**: Modern updates, improved IKEv2 handling, better error reporting

### When to Use This Tool

- **VPN endpoint discovery**: Find all IKE-enabled hosts on a network segment
- **VPN product fingerprinting**: Identify vendor and version of VPN concentrators
- **Security auditing**: Detect weak IKE configurations or deprecated transforms
- **Penetration testing**: Enumerate VPN infrastructure before attacking IPSec tunnels
- **PSK weakness testing**: Test for pre-shared key vulnerabilities in aggressive mode
- **Compliance checks**: Verify VPN deployments meet organizational security standards

### Key Concepts

**IKE Phases**: IKE operates in two phases. Phase 1 establishes a secure management channel (ISAKMP SA). Phase 2 negotiates the actual IPSec tunnel (IPSec SA). ike-scan focuses on Phase 1 discovery and fingerprinting.

**Aggressive vs Main Mode**: Aggressive mode exchanges fewer packets (3 vs 6) and transmits identity information in cleartext, making it easier to fingerprint but less secure. Main mode protects identity but requires more round trips. ike-scan supports both.

**IKEv1 vs IKEv2**: IKEv1 uses ISAKMP and IKEv2 is a simplified, more secure protocol. ike-scan supports both versions with different packet construction and parsing.

**Transform Attributes**: These define the cryptographic algorithms (encryption, hash, DH group, authentication) proposed during Phase 1 negotiation. The combination of transforms reveals the VPN product and its configuration.

**Fingerprinting**: By analyzing which transforms are accepted or rejected, ike-scan identifies the VPN product. The built-in fingerprint database maps transform combinations to known VPN vendors.

---

## Chapter 2: Installation & Setup

### System Requirements

**Operating System**: Kali Linux (primary target), Debian/Ubuntu, or any Linux distribution with libpcap

**Dependencies**:
- libpcap (packet capture library)
- Standard C library and POSIX headers
- Root or CAP_NET_RAW capability for raw socket access

**Hardware**: No special requirements. Network interface capable of sending/receiving IKE traffic (UDP ports 500 and 4500).

### Installation Methods

**From Kali repositories** (default):
```bash
sudo apt update
sudo apt install ike-scan
```

**From source** (latest version):
```bash
git clone https://github.com/royhills/ike-scan.git
cd ike-scan
./configure
make
sudo make install
```

**Verify installation**:
```bash
ike-scan --version
```

### Configuration

ike-scan uses command-line options exclusively. No configuration files are required. The tool operates with sensible defaults:

- **Default ports**: UDP 500 (IKE) and UDP 4500 (NAT-T)
- **Default timeout**: 500 milliseconds per host
- **Default retries**: 3 attempts per host
- **Default mode**: Main mode (unless -A is specified)

### Verifying Installation

```bash
# Check version
ike-scan --version

# Verify binary location
which ike-scan

# Test with localhost (safe, no IKE service)
ike-scan 127.0.0.1

# Verify raw socket capability
sudo ike-scan --verbose 192.168.1.1
```

---

## Chapter 3: Core Concepts & Architecture

### How It Works

ike-scan constructs IKE Phase 1 packets and sends them to target hosts via UDP. The tool listens for responses and parses IKE payloads to extract transform information, vendor IDs, and other IKE attributes.

**Discovery Process**:

1. **Packet Construction**: ike-scan builds an IKE_SA_INIT (IKEv2) or ISAKMP SA (IKEv1) packet with configurable transforms
2. **Transmission**: Packets are sent to target IP on UDP 500 (or custom port)
3. **Response Collection**: The tool captures responses using libpcap
4. **Payload Parsing**: Response payloads are decoded to extract transforms, vendor IDs, and attributes
5. **Fingerprint Matching**: Extracted data is compared against the built-in fingerprint database
6. **Result Reporting**: Matches are displayed with VPN product identification

**Aggressive Mode Workflow**:

In aggressive mode, ike-scan sends an IKE_SA proposal. If the target responds with a valid transform, ike-scan extracts the DH public value and computes the SKEYID for authentication testing. This enables PSK brute-force attempts.

### Protocols & Standards

| Protocol | RFC | Purpose |
|----------|-----|---------|
| IKEv1 | RFC 2409 | Original IKE key exchange |
| IKEv2 | RFC 7296 | Simplified IKE with improved security |
| ISAKMP | RFC 2408 | Framework for key exchange |
| IPSec | RFC 4301 | Security architecture for IP |

### Network Architecture

```
┌─────────────┐         UDP 500/4500         ┌─────────────┐
│  ike-scan   │ ──────────────────────────►   │ VPN Gateway │
│  (Attacker) │ ◄──────────────────────────   │ (Target)    │
└─────────────┘     IKE_SA_INIT / ISAKMP      └─────────────┘
                          │
                          ▼
                    ┌─────────────┐
                    │  Analysis   │
                    │  Fingerprint│
                    │  Database   │
                    └─────────────┘
```

---

## Chapter 4: Command Reference

### Command Syntax

```
ike-scan [options] [targets...]
```

Targets can be individual IPs, CIDR ranges, or file-based target lists.

### Core Options

| Flag | Description | Default |
|------|-------------|---------|
| `-A` | Use Aggressive Mode | Main Mode |
| `-M` | Use Main Mode | Yes (default) |
| `-P` | Attempt PSK crack using Aggressive Mode | Disabled |
| `-d <port>` | Destination port | 500 |
| `-s <port>` | Source port | Random |
| `--retry=<n>` | Number of retries | 3 |
| `--timeout=<ms>` | Response timeout in ms | 500 |
| `--cookie=<hex>` | Set ISAKMP cookie | Random |
| `--id=<string>` | Set identification payload | None |
| `-f` | Use IKE fragmentation | Disabled |
| `--showbackoff` | Show backoff timing analysis | Disabled |

### Target Specification

| Flag | Description |
|------|-------------|
| `192.168.1.1` | Single host |
| `192.168.1.0/24` | CIDR range |
| `-l <file>` | Read targets from file |
| `--interval=<ms>` | Time between packets |

### Output Options

| Flag | Description |
|------|-------------|
| `-o <file>` | Write output to file |
| `-v` | Verbose output |
| `-vv` | Very verbose output |
| `--format=<fmt>` | Output format |

### IKEv2 Options

| Flag | Description |
|------|-------------|
| `--ikev2` | Force IKEv2 mode |
| `--ikev2method=<n>` | IKEv2 method number |
| `--auth=<method>` | Authentication method |

---

## Chapter 5: Usage Examples

### Basic Usage

**Scan a single host**:
```bash
sudo ike-scan 192.168.1.1
```

**Scan a subnet**:
```bash
sudo ike-scan 192.168.1.0/24
```

**Scan with aggressive mode** (faster, reveals more info):
```bash
sudo ike-scan -A 192.168.1.0/24
```

**Scan specific port**:
```bash
sudo ike-scan -d 4500 192.168.1.1
```

### Intermediate Usage

**Verbose output for debugging**:
```bash
sudo ike-scan -v 192.168.1.1
```

**PSK crack attempt** (aggressive mode with hash extraction):
```bash
sudo ike-scan -P -A 192.168.1.1
```

**Custom source port** (bypass some firewalls):
```bash
sudo ike-scan -s 500 -d 500 192.168.1.1
```

**Set identification payload**:
```bash
sudo ike-scan -A --id="vpnclient" 192.168.1.1
```

### Advanced Usage

**IKEv2 scanning**:
```bash
sudo ike-scan --ikev2 192.168.1.0/24
```

**Fragmented IKE packets** (bypass some IDS):
```bash
sudo ike-scan -f -A 192.168.1.1
```

**Backoff timing analysis** (detect rate limiting):
```bash
sudo ike-scan --showbackoff 192.168.1.0/24
```

**Scan from file with custom timeout**:
```bash
sudo ike-scan -l targets.txt --timeout=1000 --retry=5
```

**Batch scan with output file**:
```bash
sudo ike-scan -A -o results.txt 10.0.0.0/16
```

### Real-World Scenarios

**Internal network VPN audit**:
```bash
# Discover all VPN endpoints in the DMZ
sudo ike-scan -v -A 10.0.1.0/24 > vpn_audit.txt

# Test for weak PSK on discovered endpoints
sudo ike-scan -P -A --id="testuser" 10.0.1.10
```

**Remote VPN assessment**:
```bash
# Scan external VPN concentrator
sudo ike-scan -A -d 500 203.0.113.1

# Check if NAT-T is enabled
sudo ike-scan -d 4500 203.0.113.1
```

---

## Chapter 6: Advanced Techniques

### Automation & Scripting

**Automated VPN discovery script**:
```bash
#!/bin/bash
# Discover VPN endpoints across multiple subnets

SUBNETS="10.0.1.0/24 10.0.2.0/24 192.168.1.0/24"
OUTPUT="vpn_discovery_$(date +%Y%m%d).txt"

for subnet in $SUBNETS; do
    echo "[*] Scanning $subnet..."
    ike-scan -A -o /tmp/ike_$subnet.txt "$subnet"
done

cat /tmp/ike_*.txt > "$OUTPUT"
echo "[+] Results saved to $OUTPUT"
```

**Pipeline with awk for filtering**:
```bash
sudo ike-scan -A 192.168.1.0/24 | grep "返回" | awk '{print $1}' > active_vpn_hosts.txt
```

### Chaining with Other Tools

**Nmap integration**:
```bash
# Discover VPN hosts, then fingerprint with Nmap
sudo ike-scan -A 192.168.1.0/24 | grep -oP '\d+\.\d+\.\d+\.\d+' | \
    xargs -I {} nmap -sU -p 500,4500 --script ike-version {}
```

**With ipsec-scan**:
```bash
# Use ike-scan for discovery, ipsec-scan for deeper analysis
sudo ike-scan -A 192.168.1.0/24 | grep -oP '\d+\.\d+\.\d+\.\d+' | \
    xargs -I {} ipsec-scan {}
```

### Custom Configurations

**Forced transforms** (test specific cipher suites):
```bash
sudo ike-scan --trans=1 --trans=3 --trans=5 192.168.1.1
```

**Vendor ID filtering**:
```bash
sudo ike-scan --vendorid="Cisco" 192.168.1.0/24
```

### Performance Tuning

**Reduce scan time**:
```bash
sudo ike-scan --retry=1 --timeout=200 192.168.1.0/24
```

**Increase reliability**:
```bash
sudo ike-scan --retry=5 --timeout=2000 192.168.1.1
```

---

## Chapter 7: Detection & Defense

### How to Detect

IKE scanning generates detectable network patterns:

**Signature-based detection**:
- Multiple IKE_SA_INIT packets from single source
- Unusual source port usage
- High volume of IKE traffic to multiple hosts
- IKE packets with unusual transform combinations

**Behavioral detection**:
- Sequential scanning patterns across subnets
- Traffic to known IKE ports from non-VPN clients
- Response anomalies indicating fingerprinting

### Defensive Measures

**Network-level defenses**:
- Restrict IKE access to authorized VPN clients only
- Use firewall rules to filter UDP 500/4500 from untrusted sources
- Implement network segmentation for VPN infrastructure
- Deploy IDS/IPS rules for IKE scanning detection

**Host-level defenses**:
- Use Main Mode instead of Aggressive Mode
- Implement strong pre-shared keys (20+ characters, random)
- Disable unused IKE transforms
- Enable IKE logs for monitoring

### IDS/IPS Signatures

**Snort rule example**:
```
alert udp any any -> any 500 (msg:"IKE Scan Detected"; 
    content:"|00 00 00 00|"; depth:4; 
    threshold:type both, track by_src, count 10, seconds 60; 
    sid:1000001; rev:1;)
```

**Suricata rule example**:
```
alert udp any any -> any any (msg:"Potential IKE Scan"; 
    udp.port:500; 
    detection_filter:track by_src, count 20, seconds 60; 
    sid:2000001; rev:1;)
```

### Log Analysis

**IKE server logs** show scanning patterns:
- Multiple failed IKE negotiations from same source
- Rapid connection attempts
- Unusual identification payloads
- Failed transform negotiations

**Log monitoring commands**:
```bash
# Monitor IKE connections in real-time
tcpdump -i eth0 udp port 500 -n

# Analyze collected pcap for IKE scanning
tshark -r capture.pcap -Y "udp.port==500" -T fields -e ip.src -e ip.dst
```

---

## Chapter 8: Troubleshooting & FAQ

### Common Issues

**"No IKE responses received"**:
- Target may not be running IKE on scanned port
- Firewall blocking IKE traffic
- Try different port (-d 4500 for NAT-T)
- Increase timeout (--timeout=2000)
- Verify target is reachable (ping, traceroute)

**"Permission denied" error**:
- Run with sudo or as root
- Check libpcap permissions
- Verify network interface is up

**"Invalid transform" warnings**:
- Target may not support proposed transforms
- Try default transforms without forcing specific values
- Use verbose mode to see rejected transforms

**Inconsistent results**:
- Network congestion causing packet loss
- Rate limiting on VPN gateway
- Try --showbackoff to analyze timing

### FAQ

**Q: Can ike-scan detect VPN behind NAT?**
A: Yes, scan port 4500 for NAT-T enabled VPNs. Use `-d 4500` flag.

**Q: Does ike-scan work against IKEv2-only servers?**
A: Yes, use `--ikev2` flag to force IKEv2 packet construction.

**Q: How accurate is VPN fingerprinting?**
A: Highly accurate for known products. Fingerprint database covers most major VPN vendors. Custom or obscure VPNs may show as "Unknown".

**Q: Is PSK cracking practical with ike-scan?**
A: Only for weak PSKs. Strong keys (>16 characters) are infeasible to crack with this tool alone. Use `-P` to extract hash, then crack with hashcat.

**Q: Can ike-scan scan through VPN tunnels?**
A: Limited. IKE inside IPSec may work, but performance is degraded. Direct scanning of VPN endpoints is recommended.

---

## Resources

### Official Documentation
- [ike-scan GitHub](https://github.com/royhills/ike-scan)
- [ike-scan Manual Page](https://www.systutorials.com/docs/linux/man/1-ike-scan/)
- [IKE RFC 2409](https://tools.ietf.org/html/rfc2409) (IKEv1)
- [IKEv2 RFC 7296](https://tools.ietf.org/html/rfc7296) (IKEv2)

### Online Resources
- [IKE Scanning Techniques](https://github.com/royhills/ike-scan/wiki)
- [VPN Security Assessment](https://www.harmj0y.net/blog/)
- [IPSec Vulnerability Analysis](https://attack.mitre.org/techniques/T1572/)

### Related Tools
- **ipsec-scan**: IPSec endpoint scanner
- **ikeforce**: IKE brute-force tool
- **nmap**: Network scanner with IKE scripts
- **strongSwan**: Open-source IPSec implementation
