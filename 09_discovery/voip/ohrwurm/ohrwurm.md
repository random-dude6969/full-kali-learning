---
title: ohrwurm
description: SIP protocol fuzzing and torture test tool
mitre_tactic: discovery
mitre_tactic_id: TA0007
mitre_subcategory: voip
categories:
  - voip
  - fuzzing
  - protocol-testing
  - discovery
tags:
  - sip
  - voip
  - fuzzing
  - torture-test
  - protocol
install_command: sudo apt install ohrwurm
version: "0.1"
github_repo: https://github.com/Naoril/ohrwurm
kali_tools_page: https://www.kali.org/tools/ohrwurm/
difficulty_level: intermediate
requires_root: true
gui_available: false
related_tools:
  - sipp
  - protos-sip
  - voiphopper
---

# ohrwurm

## 1. Introduction

### What is ohrwurm?

ohrwurm is a SIP protocol fuzzing tool that sends malformed SIP messages to test the robustness of SIP implementations. It generates and transmits crafted packets with invalid headers, boundary values, and protocol violations to identify vulnerabilities in SIP servers, proxies, and endpoints.

### History

ohrwurm was developed as part of VoIP security research to automate the process of SIP protocol fuzzing. It complements tools like protos-sip by providing a more focused, real-time fuzzing approach that can be used during active penetration testing engagements.

### When to Use

- **VoIP security assessments** requiring protocol fuzzing
- **SIP implementation testing** for robustness
- **Vulnerability discovery** in SIP servers and proxies
- **Compliance testing** against malformed input handling
- **Penetration testing** of VoIP infrastructure

### VoIP Concepts

**SIP Protocol Basics:**

SIP (Session Initiation Protocol) is a text-based signaling protocol used for initiating, maintaining, and terminating real-time communication sessions. SIP messages consist of a start line, headers, and an optional body.

**SIP Message Structure:**

```
METHOD sip:user@domain SIP/2.0
Via: SIP/2.0/UDP 192.168.1.100:5060
From: <sip:caller@domain>;tag=abc123
To: <sip:callee@domain>
Call-ID: unique-id@domain
CSeq: 1 INVITE
Content-Type: application/sdp
Content-Length: 0
```

**Fuzzing Attack Vectors:**

| Vector | Description |
|--------|-------------|
| Header overflow | Exceed maximum header length |
| Invalid methods | Send undefined SIP methods |
| Boundary values | Integer overflow, negative values |
| Encoding errors | Invalid character encoding |
| Malformed SDP | Corrupted session descriptions |

---

## 2. Installation

### Kali Linux (Default)

```bash
sudo apt update
sudo apt install ohrwurm
```

### Verify Installation

```bash
ohrwurm --help
```

### Manual Installation

```bash
git clone https://github.com/Naoril/ohrwurm.git
cd ohrwurm
make
sudo cp ohrwurm /usr/local/bin/
```

---

## 3. Core Concepts

### SIP Fuzzing Architecture

```
+-----------+    Malformed SIP    +------------------+
| ohrwurm   | ---- Messages ----> |  SIP Server      |
+-----------+                     +------------------+
     |                                    |
     |    Crash / Unexpected Behavior     |
     +<-----------------------------------+
```

**Fuzzing Techniques:**

1. **Mutation-Based**: Modifies valid SIP messages with random variations
2. **Generation-Based**: Creates messages from protocol grammar rules
3. **Protocol-Aware**: Understands SIP structure and targets specific fields

**Test Categories:**

| Category | Tests |
|----------|-------|
| Syntax | Invalid characters, encoding errors |
| Semantics | Invalid header values, contradictions |
| State | Out-of-order messages, invalid transitions |
| Resource | Memory exhaustion, connection limits |

---

## 4. Command Reference

### Basic Syntax

```
ohrwurm [options] <target>
```

### Command-Line Options

| Flag | Long Form | Description |
|------|-----------|-------------|
| `-p` | `--port` | Target SIP port (default: 5060) |
| `-n` | `--number` | Number of test packets to send |
| `-t` | `--timeout` | Response timeout in seconds |
| `-v` | `--verbose` | Verbose output |
| `-i` | `--interface` | Network interface |
| `-s` | `--source` | Source IP address |
| `-r` | `--random` | Randomize test packets |
| `-d` | `--delay` | Delay between packets (ms) |

---

## 5. Examples

### Example 1: Basic SIP Fuzzing

```bash
sudo ohrwurm -p 5060 192.168.1.100
```

### Example 2: Intensive Fuzzing

```bash
sudo ohrwurm -n 1000 -v -p 5060 192.168.1.100
```

### Example 3: Targeted Header Fuzzing

```bash
sudo ohrwurm -n 500 --fuzz-headers -p 5060 192.168.1.100
```

### Example 4: Randomized Testing

```bash
sudo ohrwurm -n 2000 -r -d 100 -p 5060 192.168.1.100
```

### Example 5: Stealth Fuzzing

```bash
sudo ohrwurm -n 50 -d 500 -v -p 5060 192.168.1.100
```

---

## 6. Advanced Usage

### Custom Fuzzing Patterns

```bash
# Fuzz specific SIP methods
sudo ohrwurm --method INVITE -n 500 192.168.1.100
sudo ohrwurm --method REGISTER -n 500 192.168.1.100
sudo ohrwurm --method OPTIONS -n 500 192.168.1.100
```

### Combining with Monitoring

```bash
# Terminal 1: Run fuzzing
sudo ohrwurm -n 1000 -v 192.168.1.100

# Terminal 2: Monitor responses
sudo tcpdump -i eth0 host 192.168.1.100 and port 5060 -n
```

### Automation Scripts

```bash
#!/bin/bash
METHODS="INVITE REGISTER OPTIONS BYE CANCEL ACK"
for method in $METHODS; do
    echo "Fuzzing method: $method"
    sudo ohrwurm --method $method -n 200 192.168.1.100
    sleep 10
done
```

### Integration with Wireshark

```bash
# Capture fuzzing traffic for analysis
sudo tcpdump -i eth0 host 192.168.1.100 and port 5060 -w ohrwurm_capture.pcap &

# Run fuzzing
sudo ohrwurm -n 100 -v 192.168.1.100

# Stop capture
kill %1

# Analyze in Wireshark
wireshark ohrwurm_capture.pcap
```

### Custom Payload Generation

```bash
# Create custom malformed SIP message
cat > malformed_invite.txt << 'EOF'
INVITE sip:user@192.168.1.100 SIP/2.0
Via: SIP/2.0/UDP 192.168.1.100:5060;branch=z9hG4bK776asdhds
From: Test <sip:test@192.168.1.100>;tag=12345
To: <sip:user@192.168.1.100>
Call-ID: a84b4c76e66710@192.168.1.100
CSeq: 1 INVITE
Contact: <sip:test@192.168.1.100>
Content-Type: application/sdp
Content-Length: 142

v=0
o=user1 53655765 2353687637 IN IP4 192.168.1.100
s=-
c=IN IP4 192.168.1.100
t=0 0
m=audio 49170 RTP/AVP 0
a=rtpmap:0 PCMU/8000
EOF

# Send malformed message
cat malformed_invite.txt | nc -u 192.168.1.100 5060
```

### Fuzzing Methodology

```
Phase 1: Reconnaissance
  - Identify SIP server software and version
  - Map enabled SIP methods
  - Document normal behavior baseline

Phase 2: Fuzzing
  - Start with low-intensity fuzzing
  - Gradually increase intensity
  - Monitor for crashes and anomalies
  - Document all findings

Phase 3: Analysis
  - Analyze crash dumps
  - Identify vulnerable code paths
  - Develop proof-of-concept exploits
  - Create remediation recommendations

Phase 4: Reporting
  - Document all vulnerabilities found
  - Include severity ratings
  - Provide proof-of-concept code
  - Recommend fixes and mitigations
```

### Common Fuzzing Targets

| SIP Component | Common Vulnerabilities |
|---------------|----------------------|
| SIP Proxy | Header injection, buffer overflow |
| SIP Registrar | Authentication bypass, enumeration |
| SIP Gateway | Call hijacking, media interception |
| SIP Client | Remote code execution, DoS |

### Performance Optimization

```bash
# Run with maximum speed (caution: may cause issues)
sudo ohrwurm -n 5000 -d 0 -p 5060 192.168.1.100

# Run with moderate speed
sudo ohrwurm -n 1000 -d 50 -p 5060 192.168.1.100

# Run slowly for stealth
sudo ohrwurm -n 100 -d 2000 -p 5060 192.168.1.100
```

### Batch Processing

```bash
#!/bin/bash
TARGETS="192.168.1.100 192.168.1.101 192.168.1.102"
OUTPUT_DIR="./fuzzing_results"
mkdir -p $OUTPUT_DIR

for target in $TARGETS; do
    echo "Fuzzing: $target"
    sudo ohrwurm -n 500 -v -p 5060 $target > "$OUTPUT_DIR/${target}_results.txt" 2>&1
done
```

---

## 7. Detection & Defense

### Detection Methods

**Signature-Based Detection:**

```
alert udp any any -> 192.168.1.0/24 5060 (msg:"SIP Fuzzing Detected"; content:"SIP/2.0"; depth:8; dsize:>1000; threshold:type both, track by_src, count 10, seconds 60; sid:1000002; rev:1;)
```

**Behavioral Detection:**

- Monitor for abnormal SIP message rates
- Detect malformed SIP headers in logs
- Track crash reports on SIP servers

### Defense Strategies

**SIP Firewall Rules:**

```bash
# Rate limit SIP traffic
sudo iptables -A INPUT -p udp --dport 5060 -m limit --limit 10/minute -j ACCEPT
sudo iptables -A INPUT -p udp --dport 5060 -j DROP

# Block known fuzzing sources
sudo iptables -A INPUT -s <attacker_ip> -j DROP

# Restrict SIP to known hosts only
sudo iptables -A INPUT -p udp --dport 5060 -s 192.168.1.0/24 -j ACCEPT
sudo iptables -A INPUT -p udp --dport 5060 -j DROP
```

**SIP Server Hardening:**

- Enable SIP over TLS (SIPS)
- Implement SIP authentication
- Configure maximum message size limits
- Disable unused SIP methods
- Enable input validation on all SIP headers
- Deploy SIP-aware intrusion detection systems

**Fail2ban Configuration:**

```bash
# /etc/fail2ban/jail.local
[sip-fuzzing]
enabled = true
port = 5060
filter = sip-fuzzing
logpath = /var/log/asterisk/messages
maxretry = 5
bantime = 3600
findtime = 600
```

**Asterisk Security Settings:**

```ini
; /etc/asterisk/sip.conf
[general]
maxcallnumbers = 200
maxcallnumbers_nonvalidated = 100
calltokenoptional = all
; Enable authentication for all methods
alwaysauthreject = yes
; Limit registration attempts
max_reg_attempts = 3
```

**Monitoring and Alerting:**

```bash
# Monitor SIP server logs
tail -f /var/log/asterisk/messages | grep -i "error\|malformed\|invalid\|crash"

# Set up automated alerts
grep -c "malformed" /var/log/asterisk/messages | while read count; do
    if [ "$count" -gt 10 ]; then
        echo "Alert: High malformed SIP message count detected" | mail -s "SIP Security Alert" admin@example.com
    fi
done
```

---

## 8. Troubleshooting

### Common Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| Permission denied | Root required | Use `sudo` |
| No response | Port blocked | Check firewall rules |
| Tool crashes | Bug in fuzzer | Update to latest version |
| False positives | Normal SIP errors | Analyze responses carefully |

### Debug Commands

```bash
# Verbose mode
sudo ohrwurm -v -n 10 192.168.1.100

# Capture traffic for analysis
sudo tcpdump -i eth0 host 192.168.1.100 and port 5060 -w ohrwurm_debug.pcap

# Verify network connectivity
nc -zvu 192.168.1.100 5060

# Check if SIP server is running
nmap -sU -p 5060 192.168.1.100
```

### Analyzing Results

```bash
# Parse captured SIP responses
tcpdump -r ohrwurm_debug.pcap -n | grep "SIP/2.0"

# Look for crash indicators
grep -i "segmentation fault\|core dumped\|error" /var/log/syslog

# Check SIP server status
systemctl status asterisk
```

### Performance Issues

```bash
# Reduce packet rate if server is unstable
sudo ohrwurm -n 50 -d 500 -p 5060 192.168.1.100

# Increase timeout for slow servers
sudo ohrwurm -n 100 -t 10 -p 5060 192.168.1.100

# Monitor system resources during fuzzing
top -d 1 | grep -i "ohrwurm\|sip"
```

### Network Issues

```bash
# Check if firewall is blocking traffic
sudo iptables -L -n | grep 5060

# Verify routing
traceroute 192.168.1.100

# Test basic SIP communication
echo -e "OPTIONS sip:ping@192.168.1.100 SIP/2.0\r\nVia: SIP/2.0/UDP 192.168.1.100:5060\r\nFrom: test <sip:test@192.168.1.100>\r\nTo: ping <sip:ping@192.168.1.100>\r\nCall-ID: ping-test\r\nCSeq: 1 OPTIONS\r\nContent-Length: 0\r\n\r\n" | nc -u 192.168.1.100 5060
```

---

## Resources

- [ohrwurm GitHub](https://github.com/Naoril/ohrwurm)
- [SIP Security Best Practices](https://www.owasp.org/index.php/SIP_Security)
- [SIP Protocol RFC 3261](https://tools.ietf.org/html/rfc3261)
- [VoIP Security Testing Guide](https://www.sans.org/white-papers/voip-security/)
