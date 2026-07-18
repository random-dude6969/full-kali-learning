---
title: protos-sip
description: SIP protocol test suite from the PROTOS project
mitre_tactic: discovery
mitre_tactic_id: TA0007
mitre_subcategory: voip
categories:
  - voip
  - protocol-testing
  - fuzzing
  - discovery
tags:
  - sip
  - voip
  - protos
  - protocol-test
  - fuzzing
install_command: sudo apt install protos-sip
version: "1.0"
github_repo: https://www.ee.oulu.fi/research/ouspg/protos/
kali_tools_page: https://www.kali.org/tools/protos-sip/
difficulty_level: advanced
requires_root: true
gui_available: false
related_tools:
  - ohrwurm
  - sipp
  - siparmyknife
---

# protos-sip

## 1. Introduction

### What is protos-sip?

protos-sip is a comprehensive SIP protocol test suite developed by the PROTOS (Protocol Testing and Security) project at the University of Oulu, Finland. It provides an extensive collection of malformed SIP messages designed to test the robustness of SIP implementations against protocol violations, boundary conditions, and error handling vulnerabilities.

### History

The PROTOS project was initiated to address the growing concern about VoIP security in the early 2000s. The SIP test suite became a standard reference for SIP security testing, with its test cases widely adopted by VoIP vendors and security researchers. The project identified numerous vulnerabilities in commercial SIP implementations.

### When to Use

- **SIP implementation certification** testing
- **Security audits** of VoIP systems
- **Vulnerability research** in SIP stacks
- **Regression testing** after patches
- **Compliance verification** against SIP RFCs

### VoIP Concepts

**SIP Protocol Compliance:**

SIP (RFC 3261) defines strict rules for message construction, header formats, and state machine behavior. Non-compliant implementations may crash, leak memory, or expose security vulnerabilities when handling malformed messages.

**Test Case Categories:**

| Category | RFC Reference | Tests |
|----------|---------------|-------|
| Message Syntax | RFC 3261 §7 | Header parsing, URI formats |
| Transaction Layer | RFC 3261 §17 | State machine, timers |
| Transport Layer | RFC 3261 §18 | UDP/TCP handling |
| Session Layer | RFC 3261 §13 | Dialog management |
| Security | RFC 3261 §26 | Authentication, TLS |

---

## 2. Installation

### Kali Linux (Default)

```bash
sudo apt update
sudo apt install protos-sip
```

### Verify Installation

```bash
protos-sip --help
```

### Manual Installation

```bash
# Download from PROTOS project
wget https://www.ee.oulu.fi/research/ouspg/protos/sip/c07-sip-r200.tar.gz
tar -xzf c07-sip-r200.tar.gz
cd c07-sip-r200
sudo cp -r testcases /usr/local/share/protos-sip/
```

---

## 3. Core Concepts

### PROTOS Testing Architecture

```
+-----------+    Malformed SIP    +------------------+
| protos-sip| ---- Test Cases ---> |  SIP Server      |
+-----------+                     +------------------+
     |                                    |
     |    Log Results / Crash Reports     |
     +<-----------------------------------+
```

**Test Case Structure:**

Each test case consists of:
- A malformed SIP message (binary or text)
- Expected behavior (crash, error, success)
- Risk level (low, medium, high, critical)

**Testing Methodology:**

1. **Baseline**: Establish normal behavior with valid messages
2. **Fuzzing**: Send malformed messages systematically
3. **Analysis**: Document responses and crashes
4. **Verification**: Confirm vulnerabilities with proof-of-concept

---

## 4. Command Reference

### Basic Syntax

```
protos-sip [options] <target>
```

### Command-Line Options

| Flag | Long Form | Description |
|------|-----------|-------------|
| `-host` | `--target-host` | Target SIP server IP |
| `-port` | `--target-port` | Target SIP port (default: 5060) |
| `-timeout` | `--response-timeout` | Response timeout (ms) |
| `-verbose` | `--verbose-mode` | Detailed output |
| `-testcase` | `--test-case` | Specific test case to run |
| `-category` | `--test-category` | Test category filter |
| `-output` | `--output-file` | Save results to file |
| `-rate` | `--send-rate` | Packet send rate |

---

## 5. Examples

### Example 1: Full Test Suite

```bash
sudo protos-sip -host 192.168.1.100 -port 5060 -verbose
```

### Example 2: Category-Specific Testing

```bash
sudo protos-sip -host 192.168.1.100 -category "header-syntax" -verbose
```

### Example 3: Single Test Case

```bash
sudo protos-sip -host 192.168.1.100 -testcase TC-001 -verbose
```

### Example 4: Rate-Limited Testing

```bash
sudo protos-sip -host 192.168.1.100 -rate 10 -timeout 5000 -verbose
```

### Example 5: Save Results

```bash
sudo protos-sip -host 192.168.1.100 -output protos_results.txt -verbose
```

---

## 6. Advanced Usage

### Custom Test Case Creation

```bash
# Create custom malformed SIP message
cat > custom_test.raw << 'EOF'
INVITE sip:user@192.168.1.100 SIP/2.0
Via: SIP/2.0/UDP invalid-header-value
From: <sip:>;
To: <sip:>;
Call-ID: @
CSeq: 0 INVITE
Content-Length: -1
EOF

# Send custom test
cat custom_test.raw | nc -u 192.168.1.100 5060
```

### Batch Testing

```bash
#!/bin/bash
CATEGORIES="header-syntax transaction-layer transport-layer session-layer"
for cat in $CATEGORIES; do
    echo "Testing category: $cat"
    sudo protos-sip -host 192.168.1.100 -category $cat -output "protos_${cat}.txt"
done
```

### Integration with Monitoring

```bash
# Terminal 1: Run protos-sip
sudo protos-sip -host 192.168.1.100 -rate 50 -verbose

# Terminal 2: Monitor for crashes
sudo tcpdump -i eth0 host 192.168.1.100 and port 5060 -w protos_capture.pcap
```

---

## 7. Detection & Defense

### Detection Methods

**IDS Signatures:**

```
alert udp any any -> 192.168.1.0/24 5060 (msg:"PROTOS SIP Test Detected"; content:"SIP/2.0"; dsize:>2000; threshold:type both, track by_src, count 20, seconds 60; sid:1000003; rev:1;)
```

**Log Analysis:**

```bash
# Monitor SIP server logs for anomalies
tail -f /var/log/asterisk/messages | grep -i "error\|malformed\|invalid"
```

### Defense Strategies

**SIP Server Hardening:**

- Enable message size limits
- Implement input validation
- Use SIP over TLS (SIPS)
- Deploy SIP-aware firewalls
- Regular security patching

**Network-Level Defense:**

```bash
# Rate limit SIP traffic
sudo iptables -A INPUT -p udp --dport 5060 -m limit --limit 50/minute -j ACCEPT
sudo iptables -A INPUT -p udp --dport 5060 -j DROP

# Block suspicious sources
sudo iptables -A INPUT -s <attacker_ip> -p udp --dport 5060 -j DROP
```

---

## 8. Troubleshooting

### Common Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| No test cases found | Missing test suite | Reinstall or download manually |
| Connection refused | Port blocked | Check firewall rules |
| Tool hangs | High timeout | Reduce timeout value |
| False positives | Normal SIP errors | Cross-reference with logs |

### Debug Commands

```bash
# Verify test cases exist
ls /usr/local/share/protos-sip/testcases/

# Test connectivity
nc -zvu 192.168.1.100 5060

# Capture for analysis
sudo tcpdump -i eth0 host 192.168.1.100 and port 5060 -w debug.pcap
```

---

## Resources

- [PROTOS Project](https://www.ee.oulu.fi/research/ouspg/protos/)
- [SIP RFC 3261](https://tools.ietf.org/html/rfc3261)
- [VoIP Security Testing](https://www.sans.org/white-papers/voip-security/)
- [NIST VoIP Security](https://csrc.nist.gov/)
