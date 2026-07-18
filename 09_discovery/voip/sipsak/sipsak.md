---
title: "sipsak - SIP Swiss Army Knife"
description: "Lightweight SIP utility for sending quick SIP requests, testing SIP infrastructure, and diagnosing VoIP connectivity issues"
mitre_tactic: discovery
mitre_tactic_id: TA0007
mitre_subcategory: voip
categories:
  - discovery
  - voip
  - network
  - reconnaissance
tags:
  - sip
  - voip
  - protocol-testing
  - network-discovery
install_command: "sudo apt install sipsak"
version: "0.9.7"
github_repo: "https://github.com/atheme/sipsak"
kali_tools_page: "https://tools.kali.org/information-gathering-tools/sipsak"
difficulty_level: intermediate
requires_root: false
gui_available: false
related_tools:
  - svmap
  - siparmyknife
  - sipvicious
  - sipsniff
---

# sipsak - SIP Swiss Army Knife

sipsak is a lightweight command-line utility designed for sending quick SIP requests to test and diagnose SIP infrastructure. Known as the "SIP Swiss Army Knife," it supports multiple SIP methods including OPTIONS, REGISTER, SUBSCRIBE, NOTIFY, and MESSAGE, making it indispensable for VoIP discovery and protocol testing.

## 1. Introduction

sipsak fills a critical niche in VoIP reconnaissance by providing a fast, scriptable way to probe SIP endpoints without the overhead of full-featured SIP user agents. Security professionals use it to verify SIP server availability, test authentication mechanisms, measure response times, and enumerate SIP services across target networks.

The tool operates at the SIP protocol level, constructing raw SIP messages and sending them over UDP or TCP. Unlike heavier alternatives, sipsak focuses on simplicity and speed, making it ideal for batch scanning, automation scripts, and quick diagnostic checks during penetration tests.

### Key Capabilities

- **Multi-method support**: OPTIONS, REGISTER, SUBSCRIBE, NOTIFY, MESSAGE, INVITE, and custom methods
- **Authentication testing**: Digest authentication with configurable credentials
- **Bandwidth control**: Rate limiting for large-scale scans
- **Proxy support**: Route requests through SIP proxies
- **Timeout configuration**: Fine-grained control over retransmission and timeout behavior
- **User-Agent spoofing**: Custom User-Agent headers for fingerprinting evasion

### Use Cases

| Scenario | Description |
|----------|-------------|
| SIP Server Discovery | Probe networks for active SIP endpoints |
| Service Enumeration | Identify SIP software versions and capabilities |
| Authentication Testing | Verify SIP registration credentials |
| Connectivity Verification | Confirm SIP connectivity between endpoints |
| Firewall Rule Testing | Validate SIP traversal through firewalls |
| Proxy Chain Analysis | Map SIP routing paths through proxies |

### Protocol Support

sipsak implements RFC 3261 (SIP) with support for:

- UDP and TCP transport
- Digest authentication (MD5, MD5-sess)
- Via, Contact, and Record-Route headers
- Content-Type negotiation
- Proxy-Require and Proxy-Authorization
- Max-Forwards hop limiting

## 2. Installation

### Kali Linux

```bash
sudo apt update
sudo apt install sipsak
```

### From Source (Latest)

```bash
git clone https://github.com/atheme/sipsak.git
cd sipsak
autoreconf -fi
./configure
make
sudo make install
```

### Verify Installation

```bash
sipsak --version
which sipsak
```

### Package Details

| Property | Value |
|----------|-------|
| Package Name | sipsak |
| Default Install | No |
| Dependencies | libssl-dev, libsctp-dev |
| Architecture | amd64, i386 |
| License | GPLv2 |

## 3. Core Concepts

### SIP Protocol Basics

Session Initiation Protocol (SIP) is a text-based signaling protocol used to establish, modify, and terminate multimedia sessions. Understanding SIP fundamentals is essential for effective sipsak usage.

#### SIP Message Structure

```
|-------------------------------|
| Request Line / Status Line    |
|-------------------------------|
| Header Fields (one per line)  |
|-------------------------------|
| Empty Line                    |
|-------------------------------|
| Message Body (optional)       |
|-------------------------------|
```

#### SIP Request Methods

| Method | Purpose | sipsak Flag |
|--------|---------|-------------|
| OPTIONS | Capability query | `-m OPTIONS` (default) |
| REGISTER | Endpoint registration | `-m REGISTER` |
| SUBSCRIBE | Event subscription | `-m SUBSCRIBE` |
| NOTIFY | Event notification | `-m NOTIFY` |
| MESSAGE | Instant messaging | `-m MESSAGE` |
| INVITE | Session initiation | `-m INVITE` |

#### SIP Response Codes

| Code | Meaning | Relevance |
|------|---------|-----------|
| 200 | OK | Endpoint active, method supported |
| 401 | Unauthorized | Authentication required |
| 403 | Forbidden | Access denied |
| 404 | Not Found | Endpoint unavailable |
| 407 | Proxy Authentication Required | Proxy authentication needed |
| 408 | Request Timeout | Endpoint unreachable |
| 503 | Service Unavailable | Server overloaded |

### SIP URI Format

```
sip:user@host:port;parameters
```

Components:
- **user**: SIP endpoint identifier (extension, phone number, username)
- **host**: IP address or hostname of SIP server
- **port**: SIP signaling port (default: 5060)
- **parameters**: URI parameters (transport, user, method)

### Authentication Mechanism

sipsak supports HTTP Digest Authentication as defined in RFC 2617:

1. Client sends request without credentials
2. Server responds with 401 Unauthorized and challenge nonce
3. Client computes hash of username:realm:password and sends credentials
4. Server validates and responds with 200 OK

The authentication flow:

```
Client                          Server
  |--- OPTIONS ----------------->|
  |<-- 401 Unauthorized ---------|
  |--- OPTIONS + Auth ---------->|
  |<-- 200 OK ------------------|
```

## 4. Command Reference

### Basic Syntax

```
sipsak [options] -s <sip-uri>
```

### Option Categories

#### Target Specification

| Flag | Description | Example |
|------|-------------|---------|
| `-s <uri>` | SIP URI to test (required) | `-s sip:user@192.168.1.100` |
| `-H <host>` | Override host in SIP URI | `-H 192.168.1.100` |
| `-O <outbound>` | Outbound proxy address | `-O 192.168.1.1:5060` |
| `-P <proxy>` | Proxy address for routing | `-P proxy.example.com` |

#### Authentication

| Flag | Description | Example |
|------|-------------|---------|
| `-l <login>` | Authentication username | `-l admin` |
| `-p <password>` | Authentication password | `-p secret123` |
| `-a <auth>` | Authentication realm | `-a pbx.local` |

#### Request Control

| Flag | Description | Example |
|------|-------------|---------|
| `-m <method>` | SIP method to send | `-m REGISTER` |
| `-I <max-forwards>` | Max-Forwards header value | `-I 70` |
| `-C <content-type>` | Content-Type for body | `-C "application/sdp"` |
| `-U <user-agent>` | Custom User-Agent header | `-U "Custom UA/1.0"` |

#### Timing and Retransmission

| Flag | Description | Example |
|------|-------------|---------|
| `-T <timeout>` | Connection timeout (ms) | `-T 5000` |
| `-M <max-time>` | Maximum scan duration (s) | `-M 60` |
| `-R <retrans>` | Number of retransmissions | `-R 3` |
| `-B <bandwidth>` | Bandwidth limit (packets/s) | `-B 100` |
| `-Q <expires>` | Expires header value | `-Q 3600` |

#### Output Control

| Flag | Description | Example |
|------|-------------|---------|
| `-v` | Verbose output | `-v` |
| `-VV` | Extra verbose output | `-VV` |
| `-q` | Quiet mode | `-q` |
| `-x <count>` | Number of requests to send | `-x 10` |

#### Transport

| Flag | Description | Example |
|------|-------------|---------|
| `-6` | Use IPv6 | `-6` |
| `-t <transport>` | Transport protocol | `-t tcp` |
| `-s <uri>` | Target URI with port | `-s sip:user@192.168.1.100:5061` |

### Return Codes

| Code | Meaning |
|------|---------|
| 0 | Success (200 OK received) |
| 1 | Error (no response) |
| 2 | Error (non-200 response) |
| 3 | Error (connection failed) |
| 4 | Error (DNS resolution failed) |
| 5 | Error (timeout) |
| 6 | Error (bandwidth exceeded) |

## 5. Examples

### Basic SIP OPTIONS Probe

```bash
# Send OPTIONS request to SIP endpoint
sipsak -s sip:user@192.168.1.100

# Verbose output for detailed response
sipsak -v -s sip:user@192.168.1.100

# Test SIP service on non-standard port
sipsak -s sip:user@192.168.1.100:5061
```

### REGISTER Method Testing

```bash
# Test registration with credentials
sipsak -s sip:1000@192.168.1.100 -l 1000 -p extension1000

# Registration with custom expires
sipsak -s sip:1000@192.168.1.100 -l 1000 -p ext1000 -Q 120

# Registration through proxy
sipsak -s sip:1000@192.168.1.100 -P 192.168.1.1 -l 1000 -p ext1000
```

### Authentication Testing

```bash
# Test with wrong credentials (expect 401/403)
sipsak -s sip:1000@192.168.1.100 -l admin -p wrongpass

# Test with valid credentials (expect 200)
sipsak -s sip:1000@192.168.1.100 -l 1000 -p correctpass

# Test authentication realm
sipsak -v -s sip:1000@192.168.1.100 -l 1000 -p ext1000 -a pbx.local
```

### Rate-Limited Scanning

```bash
# Bandwidth-limited scan for enumeration
sipsak -s sip:1000@192.168.1.100 -B 50 -l 1000 -p ext1000

# Limited number of requests
sipsak -s sip:user@192.168.1.100 -x 5 -v

# Scan with timeout control
sipsak -s sip:user@192.168.1.100 -T 2000 -R 1
```

### Proxy and Outbound Testing

```bash
# Route through outbound proxy
sipsak -s sip:user@192.168.1.100 -O 192.168.1.1:5060

# Test via SIP proxy
sipsak -s sip:user@pbx.example.com -P proxy.local

# Test SIP over TCP
sipsak -s sip:user@192.168.1.100 -t tcp
```

### Custom Headers and Methods

```bash
# Send SUBSCRIBE request
sipsak -s sip:user@192.168.1.100 -m SUBSCRIBE

# Custom User-Agent for fingerprinting
sipsak -s sip:user@192.168.1.100 -U "CustomTest/1.0"

# MESSAGE method for text delivery
sipsak -s sip:user@192.168.1.100 -m MESSAGE -C "text/plain" -d "Test message"
```

### Automated Scanning

```bash
# Quick scan with minimal output
sipsak -q -s sip:user@192.168.1.100

# Scan with maximum verbosity
sipsak -VV -s sip:user@192.168.1.100

# Timeout-based scan
sipsak -M 30 -s sip:user@192.168.1.100
```

## 6. Advanced Usage

### Scripting Integration

sipsak integrates with shell scripts for automated VoIP discovery:

```bash
#!/bin/bash
# SIP endpoint discovery script
TARGETS="192.168.1.1 192.168.1.10 192.168.1.100"
for target in $TARGETS; do
    result=$(sipsak -q -s sip:user@$target 2>&1)
    if [ $? -eq 0 ]; then
        echo "[+] $target - SIP Active"
    else
        echo "[-] $target - No Response"
    fi
done
```

### Batch Testing with Input

```bash
# Test multiple SIP URIs from file
while read uri; do
    sipsak -s "$uri" -q 2>/dev/null && echo "Active: $uri"
done < sip_targets.txt
```

### Authentication Brute-Force Script

```bash
#!/bin/bash
# Credential testing script
EXTENSION="1000"
TARGET="192.168.1.100"
DICTIONARY="passwords.txt"

while read password; do
    result=$(sipsak -s sip:${EXTENSION}@${TARGET} \
             -l ${EXTENSION} -p "${password}" -q 2>&1)
    if [ $? -eq 0 ]; then
        echo "[+] Found: ${EXTENSION}:${password}"
        break
    fi
done < "$DICTIONARY"
```

### Performance Tuning

| Parameter | Value | Purpose |
|-----------|-------|---------|
| `-B 100` | 100 packets/sec | Balanced rate for LAN |
| `-T 3000` | 3000ms timeout | Accounts for network latency |
| `-R 2` | 2 retransmissions | Reduces packet loss impact |
| `-Q 3600` | 1 hour expires | Standard registration duration |

### Integration with Other Tools

```bash
# Pipe results to analysis
sipsak -v -s sip:user@192.168.1.100 2>&1 | grep -i "user-agent"

# Capture traffic during test
sipsak -s sip:user@192.168.1.100 &
tcpdump -i eth0 -w sip_traffic.pcap port 5060 &
wait

# Combine with DNS enumeration
for host in $(dig +short sip.example.com); do
    sipsak -s sip:user@$host -q
done
```

## 7. Detection and Defense

### Detection Indicators

SIP OPTIONS probes generate recognizable patterns:

| Indicator | Value |
|-----------|-------|
| Method | OPTIONS |
| User-Agent | sipsak/[version] |
| Max-Forwards | 70 (default) |
| Transport | UDP |
| Frequency | Burst pattern |

### Defensive Measures

**Network-Level:**

```bash
# iptables rules to detect/restrict sipsak traffic
iptables -A INPUT -p udp --dport 5060 -m string \
    --string "sipsak" -j DROP

# Rate limiting for SIP traffic
iptables -A INPUT -p udp --dport 5060 \
    -m limit --limit 10/second -j ACCEPT
iptables -A INPUT -p udp --dport 5060 -j DROP
```

**SIP Server Configuration:**

```
# FreeSWITCH - Disable OPTIONS response to unknown sources
# In autoload_conf/sofia.conf.xml
<param name="apply-inbound-acl" value="rtp.conf"/>

# Asterisk - Restrict by IP
# In sip.conf
[general]
dynamic_peers=no
```

**Monitoring:**

```bash
# Snort/Suricata rule for sipsak detection
alert udp any any -> $HOME_NET 5060 (
    msg:"SIP sipsak Probe Detected";
    content:"OPTIONS sip:";
    content:"sipsak";
    sid:1000001; rev:1;
)
```

### Countermeasures for Penetration Testers

- Use `-U` to spoof User-Agent headers
- Adjust `-B` for slower, stealthier scans
- Route through proxies with `-P`
- Use `-T` for realistic timeout behavior
- Consider TCP transport with `-t tcp`

## 8. Troubleshooting

### Common Issues

#### No Response Received

```bash
# Verify target is reachable
ping 192.168.1.100

# Check SIP port is open
nc -zvu 192.168.1.100 5060

# Increase timeout
sipsak -T 10000 -s sip:user@192.168.1.100

# Try TCP transport
sipsak -t tcp -s sip:user@192.168.1.100
```

#### Authentication Failures

```bash
# Verify credentials
sipsak -v -s sip:1000@192.168.1.100 -l 1000 -p test

# Check realm
sipsak -VV -s sip:1000@192.168.1.100 -l 1000 -p test

# Use correct realm
sipsak -s sip:1000@192.168.1.100 -l 1000 -p test -a pbx.local
```

#### DNS Resolution Issues

```bash
# Use IP address directly
sipsak -s sip:user@192.168.1.100

# Verify DNS resolution
nslookup sip.example.com

# Use -H to override
sipsak -s sip:user@domain.com -H 192.168.1.100
```

#### Firewall Blocking

```bash
# Check local firewall
sudo iptables -L -n | grep 5060

# Verify network path
traceroute 192.168.1.100

# Test with lower rate
sipsak -B 10 -s sip:user@192.168.1.100
```

### Debug Mode

```bash
# Maximum verbosity for troubleshooting
sipsak -VV -s sip:user@192.168.1.100

# Capture raw packets
sipsak -s sip:user@192.168.1.100 &
tcpdump -i eth0 -n port 5060 -A &
wait
```

### Performance Issues

| Symptom | Solution |
|---------|----------|
| High latency | Increase `-T` timeout value |
| Packet loss | Decrease `-B` bandwidth limit |
| Incomplete scans | Check `-M` max-time setting |
| DNS delays | Use IP addresses with `-H` |

## Resources

### Official Documentation
- [sipsak GitHub Repository](https://github.com/atheme/sipsak)
- [RFC 3261 - SIP Protocol](https://www.rfc-editor.org/rfc/rfc3261)
- [sipsak Man Page](https://linux.die.net/man/1/sipsak)

### Related Tools
- **svmap**: SIP network scanner from sipvicious suite
- **siparmyknife**: Alternative SIP testing tool
- **sipsniff**: SIP traffic sniffer
- **sngrep**: SIP call flow visualizer

### Learning Resources
- [SIP Protocol Overview](https://www.oreilly.com/library/view/voip-hacks/0596009526/)
- [VoIP Security Assessment](https://www.sans.org/white-papers/voip-security/)
- [SIP Security Testing](https://owasp.org/www-community/attacks/VoIP)

### Related MITRE ATT&CK Techniques
- **T1046 - Network Service Discovery**: Using SIP probes to discover services
- **T1018 - Remote System Discovery**: Identifying SIP endpoints
- **T1040 - Network Sniffing**: Capturing SIP traffic for analysis
