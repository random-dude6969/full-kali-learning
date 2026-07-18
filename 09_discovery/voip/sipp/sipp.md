---
title: sipp
description: SIP protocol test tool and traffic generator
mitre_tactic: discovery
mitre_tactic_id: TA0007
mitre_subcategory: voip
categories:
  - voip
  - protocol-testing
  - load-testing
  - discovery
tags:
  - sip
  - voip
  - load-testing
  - protocol-test
  - traffic-generator
install_command: sudo apt install sipp
version: "3.7"
github_repo: https://github.com/SIPp/sipp
kali_tools_page: https://www.kali.org/tools/sipp/
difficulty_level: advanced
requires_root: false
gui_available: false
related_tools:
  - sipsak
  - sipvicious
  - protos-sip
---

# sipp

## 1. Introduction

### What is sipp?

sipp is a comprehensive SIP protocol test tool and traffic generator used for testing SIP infrastructure. It supports custom XML scenarios, load testing, and protocol compliance verification. sipp can act as a SIP User Agent Client (UAC), User Agent Server (UAS), or Third Party Call Controller (3PCC), making it versatile for VoIP testing.

### History

sipp was originally developed by Hewlett-Packard in 2001 to test SIP proxy servers. It has since become the industry standard for SIP load testing and protocol validation. The tool is actively maintained by the open-source community and supports modern SIP features including TLS, SRTP, and IPv6.

### When to Use

- **SIP server load testing** and capacity planning
- **Protocol compliance testing** against RFC standards
- **Performance benchmarking** of VoIP infrastructure
- **Stress testing** of SIP proxies and registrars
- **Regression testing** after configuration changes
- **VoIP security assessments** with custom scenarios

### VoIP Concepts

**SIP Protocol Testing:**

sipp generates SIP traffic based on XML scenarios that define:

- Message sequences (INVITE, ACK, BYE, etc.)
- Timing and pauses
- Header values and substitutions
- Response handling

**Testing Modes:**

| Mode | Description | Use Case |
|------|-------------|----------|
| UAC | Initiates calls | Load testing, client simulation |
| UAS | Receives calls | Server simulation, response testing |
| 3PCC | Third-party call control | Call transfer, conference testing |

**SIP Call Flow:**

```
UAC (sipp)                    UAS (Server)
    |                              |
    |-------- INVITE ------------->|
    |<------- 100 Trying ----------|
    |<------- 180 Ringing ---------|
    |<------- 200 OK --------------|
    |-------- ACK ---------------->|
    |                              |
    |        [Call Active]         |
    |                              |
    |-------- BYE ---------------->|
    |<------- 200 OK --------------|
```

---

## 2. Installation

### Kali Linux (Default)

```bash
sudo apt update
sudo apt install sipp
```

### Verify Installation

```bash
sipp -v
```

### Manual Installation (Latest)

```bash
git clone https://github.com/SIPp/sipp.git
cd sipp
mkdir build && cd build
cmake ..
make
sudo cp sipp /usr/local/bin/
```

### Dependencies

```bash
sudo apt install build-essential cmake libssl-dev libncurses-dev
```

---

## 3. Core Concepts

### sipp Architecture

```
+-----------+    SIP Scenarios   +------------------+
| sipp      | <---------------> |  SIP Server      |
+-----------+                   +------------------+
     |                                |
     |    Generate Load               |
     +--------------------------------+
     |
     v
+-----------+    Statistics      +------------------+
| XML       | ---------------> | Reports          |
| Scenarios |                  |                  |
+-----------+                  +------------------+
```

**XML Scenario Structure:**

```xml
<?xml version="1.0" encoding="ISO-8859-1" ?>
<scenario name="Basic SIP Call">
  <!-- Send INVITE -->
  <send retrans="500">
    <![CDATA[
      INVITE sip:[service]@[remote_ip]:[remote_port] SIP/2.0
      Via: SIP/2.0/[transport] [local_ip]:[local_port];branch=[branch]
      From: sipp <sip:[service]@[local_ip]:[local_port]>;tag=[call_number]
      To: <sip:[service]@[remote_ip]:[remote_port]>
      Call-ID: [call_id]
      CSeq: 1 INVITE
      Contact: sip:[service]@[local_ip]:[local_port]
      Max-Forwards: 70
      Content-Type: application/sdp
      Content-Length: [len]
      
      v=0
      o=user1 53655765 2353687637 IN IP[local_ip_type] [local_ip]
      s=-
      c=IN IP[media_ip_type] [media_ip]
      t=0 0
      m=audio [media_port] RTP/AVP 0 8
      a=rtpmap:0 PCMU/8000
      a=rtpmap:8 PCMA/8000
    ]]>
  </send>

  <!-- Expect 100 Trying -->
  <recv response="100" optional="true">
  </recv>

  <!-- Expect 180 Ringing -->
  <recv response="180" optional="true">
  </recv>

  <!-- Expect 200 OK -->
  <recv response="200">
  </recv>

  <!-- Send ACK -->
  <send>
    <![CDATA[
      ACK sip:[service]@[remote_ip]:[remote_port] SIP/2.0
      Via: SIP/2.0/[transport] [local_ip]:[local_port];branch=[branch]
      From: sipp <sip:[service]@[local_ip]:[local_port]>;tag=[call_number]
      To: <sip:[service]@[remote_ip]:[remote_port]>[peer_tag_param]
      Call-ID: [call_id]
      CSeq: 1 ACK
      Contact: sip:[service]@[local_ip]:[local_port]
      Max-Forwards: 70
      Content-Length: 0
    ]]>
  </send>

  <!-- Pause for call duration -->
  <pause milliseconds="[call_duration]"/>

  <!-- Send BYE -->
  <send retrans="500">
    <![CDATA[
      BYE sip:[service]@[remote_ip]:[remote_port] SIP/2.0
      Via: SIP/2.0/[transport] [local_ip]:[local_port];branch=[branch]
      From: sipp <sip:[service]@[local_ip]:[local_port]>;tag=[call_number]
      To: <sip:[service]@[remote_ip]:[remote_port]>[peer_tag_param]
      Call-ID: [call_id]
      CSeq: 2 BYE
      Contact: sip:[service]@[local_ip]:[local_port]
      Max-Forwards: 70
      Content-Length: 0
    ]]>
  </send>

  <!-- Expect 200 OK for BYE -->
  <recv response="200">
  </recv>
</scenario>
```

---

## 4. Command Reference

### Basic Syntax

```
sipp [options] <target>
```

### Command-Line Options

| Flag | Long Form | Description |
|------|-----------|-------------|
| `-sf` | `--scenario-file` | Load custom XML scenario |
| `-r` | `--rate` | Call rate (calls/second) |
| `-m` | `--max-calls` | Maximum total calls |
| `-l` | `--limit` | Maximum simultaneous calls |
| `-p` | `--port` | Local port (default: 5060) |
| `-nr` | `--no-reconnect` | Disable reconnection |
| `-trace_stat` | `--trace-statistics` | Write statistics to file |
| `-trace_screen` | `--trace-screen` | Write screen output to file |
| `-bg` | `--background` | Run in background mode |
| `-f` | `--from` | From SIP URI |
| `-t` | `--transport` | Transport protocol (udp/tcp/tls) |
| `-tls_port` | `--tls-port` | TLS port |
| `-auth` | `--authentication` | Enable authentication |

### Built-in Scenarios

```bash
# List available scenarios
sipp -sipp_help

# Use built-in UAC scenario
sipp 192.168.1.100

# Use built-in UAS mode
sipp -bg -p 5061
```

---

## 5. Examples

### Example 1: Basic Load Test

Generate 100 calls at 10 calls/second:

```bash
sipp 192.168.1.100 -r 10 -m 100
```

### Example 2: Custom Scenario

Load a custom XML scenario:

```bash
sipp 192.168.1.100 -sf scenario.xml -r 5 -m 50
```

### Example 3: Rate-Limited Testing

Gradual load increase:

```bash
sipp 192.168.1.100 -r 1 -m 1000 -l 10 -p 5060
```

### Example 4: Background Mode

Run test in background:

```bash
sipp 192.168.1.100 -bg -r 20 -m 500 -trace_stat
```

### Example 5: UAS Mode

Start a SIP server simulator:

```bash
sipp -bg -p 5061 -trace_stat
```

### Example 6: 3PCC Testing

Third-party call control:

```bash
sipp 192.168.1.100 -sf 3pcc-2.xml -bg
```

### Example 7: TLS Testing

Test with TLS transport:

```bash
sipp 192.168.1.100 -t tls -tls_port 5061 -sf scenario.xml
```

### Example 8: Authentication

Test with SIP authentication:

```bash
sipp 192.168.1.100 -auth admin:password -sf scenario.xml
```

---

## 6. Advanced Usage

### Custom Scenario Creation

```bash
# Create scenario directory
mkdir -p ~/sipp_scenarios

# Create registration scenario
cat > ~/sipp_scenarios/register.xml << 'EOF'
<?xml version="1.0" encoding="ISO-8859-1" ?>
<scenario name="SIP Registration">
  <send retrans="500">
    <![CDATA[
      REGISTER sip:[remote_ip]:[remote_port] SIP/2.0
      Via: SIP/2.0/[transport] [local_ip]:[local_port];branch=[branch]
      From: <sip:[service]@[local_ip]:[local_port]>;tag=[call_number]
      To: <sip:[service]@[remote_ip]:[remote_port]>
      Call-ID: [call_id]
      CSeq: 1 REGISTER
      Contact: sip:[service]@[local_ip]:[local_port]
      Expires: 3600
      Max-Forwards: 70
      Content-Length: 0
    ]]>
  </send>
  <recv response="200">
  </recv>
</scenario>
EOF

# Use the scenario
sipp 192.168.1.100 -sf ~/sipp_scenarios/register.xml -r 1 -m 10
```

### Performance Tuning

```bash
# Increase concurrent calls
sipp 192.168.1.100 -r 50 -m 1000 -l 100

# Use multiple threads
sipp 192.168.1.100 -r 100 -m 5000 -l 200

# Optimize packet timing
sipp 192.168.1.100 -r 200 -m 10000 -l 500
```

### Statistics and Reporting

```bash
# Enable statistics logging
sipp 192.168.1.100 -trace_stat -stat_file stats.csv

# Enable screen recording
sipp 192.168.1.100 -trace_screen

# Real-time statistics
sipp 192.168.1.100 -trace_stat -screen_file stats.txt
```

### Automation Scripts

```bash
#!/bin/bash
# Load test script
TARGET="192.168.1.100"
RATES="1 5 10 20 50"
MAX_CALLS=1000

for rate in $RATES; do
    echo "Testing rate: $rate calls/sec"
    sipp $TARGET -r $rate -m $MAX_CALLS -trace_stat -stat_file "stats_${rate}cps.csv"
    sleep 10
done
```

### Integration with Monitoring

```bash
# Terminal 1: Run sipp load test
sipp 192.168.1.100 -r 50 -m 1000 -trace_stat

# Terminal 2: Monitor server resources
watch -n 1 'top -b | head -20'

# Terminal 3: Monitor network traffic
sudo tcpdump -i eth0 host 192.168.1.100 and port 5060 -n
```

---

## 7. Detection & Defense

### Detection Methods

**SIP Traffic Analysis:**

```bash
# Detect high call rates
sudo tcpdump -i eth0 "udp port 5060 and src <attacker_ip>" -n -c 100 | grep "INVITE" | wc -l

# Monitor for unusual patterns
sudo tcpdump -i eth0 "udp port 5060" -v | grep -i "INVITE\|REGISTER"
```

**IDS Signatures:**

```
alert udp any any -> 192.168.1.0/24 5060 (msg:"SIP Load Test Detected"; content:"INVITE"; threshold:type both, track by_src, count 50, seconds 10; sid:1000008; rev:1;)
```

### Defense Strategies

**Rate Limiting:**

```bash
# Limit SIP requests per source
sudo iptables -A INPUT -p udp --dport 5060 -m limit --limit 10/minute --limit-burst 20 -j ACCEPT
sudo iptables -A INPUT -p udp --dport 5060 -j DROP

# Limit by IP address
sudo iptables -A INPUT -p udp --dport 5060 -m connlimit --connlimit-above 10 -j DROP
```

**SIP Server Hardening:**

```bash
# Asterisk configuration
; /etc/asterisk/sip.conf
[general]
maxcallbitrate = 384
maxcallnumbers = 200
calltimeout = 30
```

**Fail2ban Protection:**

```bash
# /etc/fail2ban/jail.local
[sip-loadtest]
enabled = true
port = 5060
filter = sip-loadtest
logpath = /var/log/asterisk/messages
maxretry = 5
bantime = 3600
```

### Load Testing Best Practices

- Get written authorization before load testing
- Test in staging environments first
- Monitor system resources during tests
- Use gradual load increases
- Document all test scenarios

---

## 8. Troubleshooting

### Common Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| Connection refused | Server not running | Verify SIP server is active |
| Timeout | Network issues | Check connectivity |
| Call failures | Authentication | Add `-auth` flag |
| High latency | Server overload | Reduce call rate |
| Memory exhaustion | Too many calls | Limit concurrent calls |

### Debug Commands

```bash
# Verbose output
sipp 192.168.1.100 -v -v -sf scenario.xml

# Capture SIP traffic
sudo tcpdump -i eth0 "udp port 5060" -w debug.pcap

# Check scenario validity
sipp -check_sc scenario.xml
```

### Performance Issues

```bash
# Reduce call rate
sipp 192.168.1.100 -r 1 -m 100

# Limit concurrent calls
sipp 192.168.1.100 -l 10 -m 100

# Use shorter pauses
sipp 192.168.1.100 -sf fast_scenario.xml
```

### Scenario Validation

```bash
# Check XML syntax
xmllint scenario.xml

# Validate scenario
sipp -check_sc scenario.xml

# Test scenario locally
sipp -p 5061 -bg -sf scenario.xml
sipp 127.0.0.1 -sf scenario.xml -m 1
```

---

## Resources

- [sipp Official Documentation](https://sipp.readthedocs.io/)
- [sipp GitHub Repository](https://github.com/SIPp/sipp)
- [SIP RFC 3261](https://tools.ietf.org/html/rfc3261)
- [VoIP Load Testing Guide](https://www.sans.org/white-papers/voip-security/)
- [SIP Scenarios Library](https://sipp.readthedocs.io/en/latest/scenarios/)
