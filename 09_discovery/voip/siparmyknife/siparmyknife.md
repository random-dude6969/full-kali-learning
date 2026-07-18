---
title: siparmyknife
description: SIP vulnerability scanner and user enumeration tool
mitre_tactic: discovery
mitre_tactic_id: TA0007
mitre_subcategory: voip
categories:
  - voip
  - vulnerability-scanning
  - enumeration
  - discovery
tags:
  - sip
  - voip
  - vulnerability
  - enumeration
  - scanner
install_command: sudo apt install siparmyknife
version: "1.0"
github_repo: https://github.com/0x90/siparmyknife
kali_tools_page: https://www.kali.org/tools/siparmyknife/
difficulty_level: intermediate
requires_root: false
gui_available: false
related_tools:
  - svmap
  - sipvicious
  - enumiax
---

# siparmyknife

## 1. Introduction

### What is siparmyknife?

siparmyknife is a SIP (Session Initiation Protocol) vulnerability scanner and user enumeration tool. It tests SIP servers for common misconfigurations, weak authentication, and known vulnerabilities. The tool can enumerate valid usernames, test default credentials, and identify security weaknesses in VoIP deployments.

### History

siparmyknife was developed to provide a comprehensive SIP security assessment tool that combines user enumeration with vulnerability scanning. It addresses the need for a single tool that can perform both reconnaissance and vulnerability testing against SIP infrastructure.

### When to Use

- **VoIP penetration testing** against SIP servers
- **User enumeration** on SIP-enabled infrastructure
- **Credential testing** for default and weak passwords
- **Vulnerability scanning** for SIP misconfigurations
- **Security assessments** of VoIP deployments

### VoIP Concepts

**SIP Authentication:**

SIP servers typically use digest authentication (MD5-based) or basic authentication. Common vulnerabilities include:

- Default credentials (admin:admin, user:user)
- Weak passwords
- Missing authentication on critical methods
- Exposed administrative interfaces

**SIP Methods for Enumeration:**

| Method | Purpose | Risk |
|--------|---------|------|
| REGISTER | Register endpoint | Username enumeration |
| OPTIONS | Query capabilities | Server fingerprinting |
| INVITE | Initiate call | User existence verification |
| SUBSCRIBE | Subscribe to events | Event enumeration |

---

## 2. Installation

### Kali Linux (Default)

```bash
sudo apt update
sudo apt install siparmyknife
```

### Verify Installation

```bash
siparmyknife --help
```

### Manual Installation

```bash
git clone https://github.com/0x90/siparmyknife.git
cd siparmyknife
chmod +x siparmyknife
sudo cp siparmyknife /usr/local/bin/
```

---

## 3. Core Concepts

### SIP Vulnerability Scanning

```
+-----------+    SIP Messages   +------------------+
| siparmy   | <---------------> |  SIP Server      |
+-----------+                   +------------------+
     |                                |
     |    Enumerate / Test            |
     +--------------------------------+
     |
     v
+-----------+    Results        +------------------+
| Valid     | ---------------> | Reports          |
| Users     |                  |                  |
+-----------+                  +------------------+
```

**Scanning Techniques:**

1. **User Enumeration**: Send REGISTER requests to identify valid extensions
2. **Default Credential Testing**: Test common username/password combinations
3. **Method Probing**: Check which SIP methods are enabled
4. **Header Analysis**: Identify server version and configuration

**Attack Vectors:**

| Vector | Description | Impact |
|--------|-------------|--------|
| User Enumeration | Identify valid extensions | Account compromise |
| Default Credentials | Test common passwords | Unauthorized access |
| Method Abuse | Exploit enabled methods | Service disruption |
| Information Leak | Server fingerprinting | Targeted attacks |

---

## 4. Command Reference

### Basic Syntax

```
siparmyknife [options] <target>
```

### Command-Line Options

| Flag | Long Form | Description |
|------|-----------|-------------|
| `-t` | `--target` | Target SIP server IP |
| `-p` | `--port` | Target SIP port (default: 5060) |
| `-d` | `--dict` | Dictionary file for enumeration |
| `-u` | `--user` | Specific username to test |
| `-v` | `--verbose` | Verbose output |
| `-T` | `--timeout` | Response timeout in seconds |
| `-f` | `--from-domain` | From domain for SIP messages |
| `-m` | `--method` | SIP method to use |

---

## 5. Examples

### Example 1: Basic Server Scan

```bash
siparmyknife -t 192.168.1.100 -v
```

### Example 2: User Enumeration

Enumerate usernames using a dictionary:

```bash
siparmyknife -t 192.168.1.100 -d /usr/share/wordlists/sip_users.txt -v
```

### Example 3: Default Credential Testing

Test common default credentials:

```bash
siparmyknife -t 192.168.1.100 -d /usr/share/wordlists/sip_defaults.txt -v
```

### Example 4: Single User Test

Test a specific username:

```bash
siparmyknife -t 192.168.1.100 -u admin -v
```

### Example 5: Custom Port

Test on non-standard port:

```bash
siparmyknife -t 192.168.1.100 -p 5080 -v
```

### Example 6: Network Scan

Scan entire subnet:

```bash
for ip in $(seq 1 254); do
    siparmyknife -t 192.168.1.$ip -v 2>/dev/null
done
```

---

## 6. Advanced Usage

### Custom SIP Messages

```bash
# Test with custom From domain
siparmyknife -t 192.168.1.100 -f "evil.com" -u admin -v

# Test different SIP methods
siparmyknife -t 192.168.1.100 -m OPTIONS -v
```

### Integration with Other Tools

```bash
# Combine with nmap for SIP discovery
nmap -sU -p 5060 192.168.1.0/24 -oG - | grep "5060/open" | awk '{print $2}' > sip_targets.txt

# Enumerate all discovered SIP servers
for ip in $(cat sip_targets.txt); do
    siparmyknife -t $ip -d users.txt -v >> sip_results.txt
done
```

### Automation Scripts

```bash
#!/bin/bash
TARGETS="192.168.1.100 192.168.1.101 192.168.1.102"
DICT="/usr/share/wordlists/sip_users.txt"

for target in $TARGETS; do
    echo "Scanning: $target"
    siparmyknife -t $target -d $DICT -v > "sip_${target}.txt"
done
```

### Result Processing

```bash
# Extract valid users
grep "Valid" sip_results.txt | awk '{print $2}' > valid_users.txt

# Count discovered users
grep -c "Valid" sip_results.txt
```

---

## 7. Detection & Defense

### Detection Methods

**Network Monitoring:**

```bash
# Detect SIP enumeration attempts
sudo tcpdump -i eth0 "udp port 5060" -n -c 100

# Monitor REGISTER requests
sudo tcpdump -i eth0 "udp port 5060 and src host <attacker_ip>" -n
```

**IDS Signatures:**

```
alert udp any any -> 192.168.1.0/24 5060 (msg:"SIP User Enumeration Detected"; content:"REGISTER"; content:"Authorization"; threshold:type both, track by_src, count 20, seconds 60; sid:1000007; rev:1;)
```

### Defense Strategies

**SIP Server Hardening:**

```bash
# Rate limit REGISTER requests
# Asterisk: /etc/asterisk/sip.conf
[general]
max REGISTER attempts = 3
REGISTER timeout = 10

# Fail2ban configuration
# /etc/fail2ban/jail.local
[sip]
enabled = true
port = 5060
filter = sip
logpath = /var/log/asterisk/messages
maxretry = 3
bantime = 3600
```

**Authentication Strengthening:**

- Use strong, unique passwords
- Implement certificate-based authentication
- Enable TLS for SIP signaling
- Restrict access by IP address

**Network Security:**

```bash
# Block unauthorized SIP sources
sudo iptables -A INPUT -p udp --dport 5060 -s 192.168.1.0/24 -j ACCEPT
sudo iptables -A INPUT -p udp --dport 5060 -j DROP

# Rate limit SIP traffic
sudo iptables -A INPUT -p udp --dport 5060 -m limit --limit 10/minute -j ACCEPT
```

---

## 8. Troubleshooting

### Common Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| No response | Port blocked | Check firewall rules |
| Timeout | Server slow | Increase `-T` timeout |
| False positives | Network issues | Verify with manual testing |
| Permission denied | Privileged port | No root needed; check network |

### Debug Commands

```bash
# Verbose mode
siparmyknife -t 192.168.1.100 -v -v

# Capture traffic
sudo tcpdump -i eth0 "udp port 5060 and host 192.168.1.100" -w debug.pcap

# Test connectivity
nc -zvu 192.168.1.100 5060
```

### Result Analysis

```bash
# Parse results
grep -E "Valid|Invalid|Error" sip_results.txt

# Sort by status
sort -k2 sip_results.txt | uniq
```

---

## Resources

- [SIP RFC 3261](https://tools.ietf.org/html/rfc3261)
- [SIP Security Best Practices](https://www.owasp.org/index.php/SIP_Security)
- [VoIP Security Testing](https://www.sans.org/white-papers/voip-security/)
- [Asterisk Security](https://docs.asterisk.org/)
