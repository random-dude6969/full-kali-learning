---
title: enumiax
description: IAX2 protocol enumerator for VoIP user discovery and brute-force attacks
mitre_tactic: discovery
mitre_tactic_id: TA0007
mitre_subcategory: voip
categories:
  - voip
  - enumeration
  - brute-force
  - discovery
tags:
  - iax2
  - voip
  - enumeration
  - brute-force
  - protocol
install_command: sudo apt install enumiax
version: "0.1"
github_repo: https://github.com/0x90/enumiax
kali_tools_page: https://www.kali.org/tools/enumiax/
difficulty_level: intermediate
requires_root: false
gui_available: false
related_tools:
  - sipvicious
  - svmap
  - siparmyknife
---

# enumiax

## 1. Introduction

### What is enumiax?

enumiax is a VoIP enumeration and brute-force tool targeting the IAX2 (Inter-Asterisk eXchange version 2) protocol. It discovers valid usernames, extensions, and accounts on IAX2 servers by sending crafted registration requests and analyzing responses. The tool supports dictionary-based and sequential enumeration attacks against IAX2-enabled PBX systems.

### History

IAX2 was developed by Digium as an open protocol for VoIP communication between Asterisk servers and endpoints. Unlike SIP, IAX2 uses a single UDP port (4569) for both signaling and media, simplifying NAT traversal. enumiax was created to address the need for IAX2-specific enumeration tools in the penetration testing community.

### When to Use

- **VoIP penetration testing** against Asterisk PBX systems
- **User enumeration** on IAX2-enabled infrastructure
- **Credential brute-forcing** for weak IAX2 passwords
- **Security assessments** of VoIP deployments using IAX2
- **Compliance audits** requiring VoIP protocol coverage

### VoIP Concepts

**IAX2 Protocol Fundamentals:**

IAX2 operates over UDP port 4569 and uses a binary protocol format. It handles both control and media traffic, making it efficient for NAT traversal. IAX2 supports authentication via MD5 or plaintext methods.

**Key IAX2 Message Types:**

| Message | Purpose |
|---------|---------|
| NEW | Initiate a new call |
| ACCEPT | Accept an incoming call |
| REJECT | Reject an incoming call |
| ACK | Acknowledge receipt |
| HANGUP | Terminate a call |
| REGREQ | Registration request |
| REGACK | Registration acknowledgement |
| REGREJ | Registration rejection |

**IAX2 vs SIP:**

IAX2 combines signaling and media on a single port (4569), while SIP uses separate ports for signaling (5060) and media (RTP range). IAX2's binary format is more compact than SIP's text-based protocol.

---

## 2. Installation

### Kali Linux (Default)

enumiax is included in Kali Linux repositories:

```bash
sudo apt update
sudo apt install enumiax
```

### Verify Installation

```bash
enumiax --help
```

Expected output shows usage syntax and available options.

### Manual Installation

```bash
git clone https://github.com/0x90/enumiax.git
cd enumiax
chmod +x enumiax
sudo cp enumiax /usr/local/bin/
```

---

## 3. Core Concepts

### IAX2 Architecture

```
+-----------+         UDP/4569         +------------------+
|  enumiax  | <---------------------> |  IAX2 PBX Server |
+-----------+   REGREQ/REGACK/REGREJ  +------------------+
```

**Protocol Flow:**

1. enumiax sends REGREQ messages with username/password combinations
2. Server responds with REGACK (valid credentials) or REGREJ (invalid)
3. Tool aggregates valid usernames and passwords from responses

**Authentication Methods:**

- **MD5 Authentication**: Server sends challenge, client responds with MD5 hash
- **Plaintext Authentication**: Username/password sent in cleartext (insecure)

### Enumeration Strategies

| Strategy | Description | Speed |
|----------|-------------|-------|
| Sequential | Iterate extension range (100-999) | Slow |
| Dictionary | Use wordlist of common extensions | Medium |
| Combined | Dictionary + sequential fallback | Medium |

---

## 4. Command Reference

### Basic Syntax

```
enumiax [options] <target>
```

### Command-Line Options

| Flag | Long Form | Description |
|------|-----------|-------------|
| `-u` | `--user` | Username or extension to enumerate |
| `-p` | `--password` | Password to test |
| `-d` | `--dict` | Dictionary file for brute-force |
| `-n` | `--range` | Extension range to scan |
| `-v` | `--verbose` | Verbose output |
| `-m` | `--max` | Maximum concurrent requests |
| `-t` | `--timeout` | Response timeout in seconds |
| `-b` | `--brute` | Enable brute-force mode |
| `-f` | `--format` | Output format (text, csv) |

### Target Specification

```bash
# Single target
enumiax 192.168.1.100

# Target with port
enumiax 192.168.1.100:4569

# CIDR range
enumiax 192.168.1.0/24
```

---

## 5. Examples

### Example 1: Basic User Enumeration

Enumerate extensions 100-200 on a target IAX2 server:

```bash
enumiax -n 100-200 -v 192.168.1.100
```

Output shows discovered extensions and authentication status.

### Example 2: Dictionary Attack

Brute-force usernames using a wordlist:

```bash
enumiax -d /usr/share/wordlists/rockyou.txt -v 192.168.1.100
```

### Example 3: Single User Test

Test a specific username with a password:

```bash
enumiax -u admin -p password123 192.168.1.100
```

### Example 4: Range Scan with Output

Scan extension range and save results:

```bash
enumiax -n 1000-9999 -m 10 -t 5 -v 192.168.1.100 2>&1 | tee enumiax_results.txt
```

### Example 5: Full Network Scan

Enumerate IAX2 across an entire subnet:

```bash
for ip in $(seq 1 254); do
    enumiax -n 100-200 -t 3 192.168.1.$ip 2>/dev/null
done
```

---

## 6. Advanced Usage

### Performance Tuning

**Concurrent Requests:**

```bash
enumiax -m 20 -t 2 -n 100-999 192.168.1.100
```

**Rate Limiting (Avoid Detection):**

```bash
enumiax -m 1 -t 10 -n 100-200 192.168.1.100
```

### Custom Authentication Testing

```bash
# Test MD5 authentication
enumiax -u admin -p pass123 --auth md5 192.168.1.100

# Test plaintext authentication
enumiax -u admin -p pass123 --auth plain 192.168.1.100
```

### Integration with Other Tools

```bash
# Pipe valid users to sipvicious
enumiax -n 100-999 192.168.1.100 | grep "VALID" | cut -d: -f1 > valid_users.txt

# Combine with nmap for host discovery
nmap -sU -p 4569 192.168.1.0/24 -oG - | grep "4569/open" | awk '{print $2}' > iax2_hosts.txt
```

### Automation Scripts

```bash
#!/bin/bash
TARGETS=$(nmap -sU -p 4569 192.168.1.0/24 | grep "open" | awk '{print $2}')
for target in $TARGETS; do
    enumiax -n 100-999 -m 5 -t 3 -v $target >> iax2_enum_results.txt
done
```

---

## 7. Detection & Defense

### Detection Methods

**Network-Level Detection:**

```bash
# Monitor IAX2 traffic on port 4569
sudo tcpdump -i eth0 udp port 4569 -v -w iax2_capture.pcap

# Detect enumeration patterns (high REGREQ rate)
sudo tcpdump -i eth0 udp port 4569 -c 100 | awk '{print $3}' | cut -d. -f4 | sort | uniq -c | sort -rn
```

**Firewall Rules:**

```bash
# Rate limit IAX2 traffic
sudo iptables -A INPUT -p udp --dport 4569 -m recent --set --name IAX2
sudo iptables -A INPUT -p udp --dport 4569 -m recent --update --seconds 60 --hitcount 10 --name IAX2 -j DROP
```

**IDS Signatures:**

```
alert udp any any -> 192.168.1.0/24 4569 (msg:"IAX2 Enum Enumeration Detected"; threshold:type both, track by_src, count 50, seconds 60; sid:1000001; rev:1;)
```

### Defense Strategies

**SRTP and Encryption:**

- Enable MD5 authentication on IAX2 connections
- Implement IAX2 encryption (AES-128)
- Use VPN for IAX2 traffic between sites

**Access Control:**

- Restrict IAX2 access by IP address
- Implement fail2ban for IAX2 brute-force protection
- Monitor registration logs for suspicious patterns

**Asterisk Configuration:**

```ini
; /etc/asterisk/iax.conf
[general]
maxcallnumbers = 200
maxcallnumbers_nonvalidated = 100
calltokenoptional = all
```

---

## 8. Troubleshooting

### Common Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| No response | UDP port blocked | Check firewall rules on port 4569 |
| Timeout errors | Server too slow to respond | Increase `-t` timeout value |
| False positives | Network instability | Reduce `-m` concurrency |
| Permission denied | Non-privileged port | Use source port with `-sp` flag |

### Debug Mode

```bash
enumiax -v -v -n 100-105 192.168.1.100
```

### Network Verification

```bash
# Verify IAX2 port is reachable
nc -zvu 192.168.1.100 4569

# Check IAX2 registration attempts
sudo tcpdump -i eth0 udp port 4569 -n -c 10
```

### Packet Analysis

```bash
# Capture IAX2 traffic for analysis
sudo tcpdump -i eth0 udp port 4569 -w iax2_debug.pcap

# Open in Wireshark for deep inspection
wireshark iax2_debug.pcap
```

---

## Resources

- [Asterisk IAX2 Documentation](https://docs.asterisk.org/Configuration/Protocols/IAX2/)
- [RFC 5456 - IAX2 Protocol](https://tools.ietf.org/html/rfc5456)
- [SANS VoIP Security](https://www.sans.org/white-papers/voip-security/)
- [OWASP VoIP Security](https://owasp.org/www-project-web-security-testing-guide/)
- [Kali Linux VoIP Tools](https://www.kali.org/docs/)
