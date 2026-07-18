---
title: "svcrack - SIP Credential Brute-Force Tool"
description: "Specialized SIP brute-force tool for testing extension passwords using dictionary and numeric range attacks"
mitre_tactic: discovery
mitre_tactic_id: TA0007
mitre_subcategory: voip
categories:
  - discovery
  - voip
  - brute-force
  - credential-testing
tags:
  - sip
  - voip
  - brute-force
  - password-cracking
  - credential-testing
install_command: "sudo apt install sipvicious"
version: "0.3.4"
github_repo: "https://github.com/EnableSecurity/sipvicious"
kali_tools_page: "https://tools.kali.org/information-gathering-tools/sipvicious"
difficulty_level: intermediate
requires_root: false
gui_available: false
related_tools:
  - sipvicious
  - hydra
  - medusa
  - ncrack
---

# svcrack - SIP Credential Brute-Force Tool

svcrack is a specialized brute-force tool designed for testing SIP extension passwords. As part of the sipvicious suite, it provides efficient dictionary attacks, numeric range testing, and resume capabilities for comprehensive SIP credential audits.

## 1. Introduction

svcrack addresses the critical security need of validating SIP extension password strength. Weak or default passwords on VoIP extensions can lead to toll fraud, eavesdropping, and unauthorized access to corporate communications. svcrack enables security professionals to identify these vulnerabilities before malicious actors exploit them.

The tool leverages SIP REGISTER authentication to test credentials against target extensions. It supports multiple attack vectors including dictionary files, numeric ranges, and custom password lists, with built-in rate limiting and resume capabilities for large-scale assessments.

### Attack Vectors

| Method | Description | Use Case |
|--------|-------------|----------|
| Dictionary Attack | Wordlist-based passwords | Common password testing |
| Numeric Range | Sequential PIN testing | Default PIN discovery |
| Custom List | Targeted password list | Organization-specific testing |
| Hybrid | Combined approaches | Comprehensive auditing |

### Why SIP Passwords Matter

| Risk | Impact |
|------|--------|
| Toll Fraud | Unauthorized international calls |
| Eavesdropping | Call interception and recording |
| Identity Theft | Caller ID spoofing |
| Service Abuse | Network resource consumption |
| Data Breach | Access to voicemail, contacts |

## 2. Installation

### Kali Linux

```bash
sudo apt update
sudo apt install sipvicious
```

### Verify Installation

```bash
svcrack --version
which svcrack
```

### Dependencies

| Package | Purpose |
|---------|---------|
| python3 | Runtime |
| python3-pysip | SIP library |
| sqlite3 | Result storage |

## 3. Core Concepts

### SIP Authentication Flow

```
Client                              Server
  |                                   |
  |--- REGISTER ------------------->  |
  |<-- 401 Unauthorized ------------  |
  |    (nonce, realm, qop)           |
  |                                   |
  |--- REGISTER + Digest Auth ----->  |
  |    HA1 = MD5(user:realm:pass)    |
  |    HA2 = MD5(method:digestURI)   |
  |    response = MD5(HA1:nonce:HA2) |
  |                                   |
  |<-- 200 OK ---------------------  | (valid credentials)
  |<-- 403 Forbidden ---------------  | (invalid credentials)
```

### Attack Modes

#### Dictionary Attack

```
Password List:          Attack Flow:
┌──────────────┐       ┌─────────────┐
│ password1    │──────▶│ Test pass1   │
│ password2    │       │ Test pass2   │
│ password3    │       │ Test pass3   │
│ ...          │       │ ...          │
│ letmein      │       │ Test letmein │
└──────────────┘       └─────────────┘
```

#### Numeric Range Attack

```
Range 1000-1010:        Attack Flow:
┌──────────────┐       ┌─────────────┐
│ 1000         │──────▶│ Test 1000   │
│ 1001         │       │ Test 1001   │
│ 1002         │       │ Test 1002   │
│ ...          │       │ ...         │
│ 1010         │       │ Test 1010   │
└──────────────┘       └─────────────┘
```

### Password Strength Analysis

| Complexity | Example | Crack Time |
|------------|---------|------------|
| 4-digit PIN | 1234 | Seconds |
| 6-digit PIN | 123456 | Minutes |
| 8-char lowercase | password | Minutes |
| 8-char mixed | P@ssw0rd | Hours |
| 12+ char complex | C0mpl3x!Pass | Years |

## 4. Command Reference

### Basic Syntax

```
svcrack [options] <target>
```

### Attack Options

#### Target Specification

| Flag | Description | Example |
|------|-------------|---------|
| `-u` | Single extension | `-u 1000` |
| `-e` | Extension range | `-e 1000-9999` |
| `-p` | SIP port | `-p 5060` |
| `<target>` | Target IP/hostname | `192.168.1.100` |

#### Password Options

| Flag | Description | Example |
|------|-------------|---------|
| `-d` | Dictionary file | `-d passwords.txt` |
| `-c` | Password range | `-c 1000-9999` |
| `-x` | Password prefix | `-x admin` |

#### Performance Options

| Flag | Description | Default |
|------|-------------|---------|
| `-r` | Rate (pkt/s) | 100 |
| `-c` | Concurrency | 10 |
| `-t` | Timeout (s) | 5 |
| `--timeout` | Connection timeout | 30 |

#### Stealth Options

| Flag | Description |
|------|-------------|
| `-S` | Stealth mode |
| `--user-agent` | Custom User-Agent |
| `-P` | Proxy address |

#### Control Options

| Flag | Description |
|------|-------------|
| `--resume` | Resume interrupted attack |
| `--force` | Force resume |
| `--verbose` | Verbose output |
| `--debug` | Debug output |

### Return Codes

| Code | Meaning |
|------|---------|
| 0 | Success (password found) |
| 1 | No valid password found |
| 2 | Connection error |
| 3 | Authentication error |
| 4 | Timeout |

## 5. Examples

### Basic Dictionary Attack

```bash
# Attack single extension
svcrack -u 1000 -d /usr/share/wordlists/rockyou.txt 192.168.1.100

# Attack with custom wordlist
svcrack -u 1000 -d custom_passwords.txt 192.168.1.100

# Verbose output
svcrack -v -u 1000 -d passwords.txt 192.168.1.100
```

### Numeric Range Attack

```bash
# Test 4-digit PINs
svcrack -u 1000 -c 1000-9999 192.168.1.100

# Test 6-digit PINs
svcrack -u 1000 -c 100000-999999 192.168.1.100

# Common PINs only
svcrack -u 1000 -c 0000-9999 192.168.1.100
```

### Multiple Extensions

```bash
# Range of extensions
svcrack -e 1000-1100 -d passwords.txt 192.168.1.100

# Fast multi-extension attack
svcrack -e 1000-1100 -d passwords.txt -c 20 192.168.1.100

# From file
cat extensions.txt | while read ext; do
    svcrack -u $ext -d passwords.txt 192.168.1.100
done
```

### Rate Control

```bash
# Slow, stealthy attack
svcrack -u 1000 -d passwords.txt -r 10 -S 192.168.1.100

# Balanced rate
svcrack -u 1000 -d passwords.txt -r 50 192.168.1.100

# Aggressive attack
svcrack -u 1000 -d passwords.txt -r 200 -c 30 192.168.1.100
```

### Resume Capability

```bash
# Start attack
svcrack -u 1000 -d large_wordlist.txt 192.168.1.100

# Interrupt with Ctrl+C

# Resume attack
svcrack --resume 192.168.1.100

# Force resume (ignore saved state)
svcrack --resume --force 192.168.1.100
```

### Advanced Scenarios

```bash
# Attack through proxy
svcrack -u 1000 -d passwords.txt -P 192.168.1.1 192.168.1.100

# Custom User-Agent
svcrack -u 1000 -d passwords.txt --user-agent "Polycom/5.0" 192.168.1.100

# Non-standard port
svcrack -u 1000 -d passwords.txt -p 5061 192.168.1.100

# IPv6 target
svcrack -u 1000 -d passwords.txt -6 ::1
```

## 6. Advanced Usage

### Automation Scripts

```bash
#!/bin/bash
# Full SIP password audit
TARGET="$1"
EXTENSIONS="1000 1001 1002 1003 1004 1005"
DICTIONARY="/usr/share/wordlists/rockyou.txt"
OUTPUT="/tmp/sip_audit_$(date +%Y%m%d).txt"

echo "[*] SIP Password Audit - $TARGET" | tee $OUTPUT

for ext in $EXTENSIONS; do
    echo "[*] Testing extension $ext..." | tee -a $OUTPUT
    svcrack -u $ext -d $DICTIONARY -r 50 $TARGET 2>&1 | tee -a $OUTPUT
done

echo "[+] Audit complete" | tee -a $OUTPUT
```

### Performance Tuning

| Scenario | Rate | Concurrency | Notes |
|----------|------|-------------|-------|
| Local LAN | 200 | 20 | Fast, reliable |
| Remote WAN | 50 | 10 | Balanced |
| Stealth | 10 | 5 | Low profile |
| Cloud/VPN | 25 | 8 | Account for latency |

### Wordlist Optimization

```bash
# Extract passwords from existing breaches
cat breached_passwords.txt | sort -u > unique_passwords.txt

# Generate organization-specific list
echo -e "company\n$(date +%Y)\n$(hostname)" > org_passwords.txt

# Filter by length
awk 'length >= 6 && length <= 12' passwords.txt > filtered.txt
```

## 7. Detection and Defense

### Detection Indicators

| Indicator | Description |
|-----------|-------------|
| User-Agent | sipvicious/svcrack |
| Pattern | Rapid REGISTER attempts |
| Source | Single or distributed IPs |
| Timing | Consistent intervals |

### Defensive Measures

**Fail2Ban Configuration:**

```ini
# /etc/fail2ban/jail.d/sip.conf
[sip-auth]
enabled = true
filter = sip-auth
action = iptables-allports[name=sip-auth]
logpath = /var/log/asterisk/messages
maxretry = 5
findtime = 600
bantime = 3600
```

**Rate Limiting:**

```bash
# Limit authentication attempts
iptables -A INPUT -p udp --dport 5060 -m string \
    --string "REGISTER sip:" --stringfrom 0 12 \
    -m hashlimit --hashlimit-above 5/min \
    --hashlimit-mode srcip \
    -j DROP
```

**SIP Server Hardening:**

```
# Asterisk - sip.conf
[general]
maxauthfail = 3
authlimit = 5
authtimeout = 10

# FreeSWITCH - vars.xml
<X-PRE-PROCESS cmd="set" data="max_failures=3"/>
```

### Countermeasures for Testers

- Use `-S` stealth mode
- Reduce rate with `-r 10`
- Vary timing with random delays
- Use proxy routing with `-P`
- Rotate source IPs if possible

## 8. Troubleshooting

### Common Issues

#### No Password Found

```bash
# Verify extension exists
svmap 192.168.1.100 -e 1000-1000

# Test manually with sipsak
sipsak -s sip:1000@192.168.1.100 -l 1000 -p test

# Check if extension requires auth
svcrack -v -u 1000 -c 1000-1001 192.168.1.100
```

#### Connection Errors

```bash
# Verify target
ping 192.168.1.100
nc -zvu 192.168.1.100 5060

# Increase timeout
svcrack -u 1000 -d passwords.txt --timeout 30 192.168.1.100

# Try TCP
svcrack -u 1000 -d passwords.txt -t tcp 192.168.1.100
```

#### Scan Stuck/Hanging

```bash
# Kill stuck process
pkill -f svcrack

# Reset state
rm ~/.sipvicious/svcrack_*

# Start fresh
svcrack -u 1000 -d passwords.txt 192.168.1.100
```

#### High False Positive Rate

```bash
# Verify with sipsak
sipsak -s sip:1000@192.168.1.100 -l 1000 -p found_password -v

# Check server logs
tail -f /var/log/asterisk/messages
```

### Performance Issues

| Symptom | Solution |
|---------|----------|
| Slow speed | Increase `-r` rate |
| Timeouts | Increase `--timeout` |
| Packet loss | Reduce rate, add `-S` |
| Memory usage | Clear old results |

### Debug Commands

```bash
# Verbose output
svcrack -v -u 1000 -d passwords.txt 192.168.1.100

# Debug mode
svcrack --debug -u 1000 -c 1000-1001 192.168.1.100

# Check state files
ls -la ~/.sipvicious/

# View SQLite database
sqlite3 ~/.sipvicious/sipvicious.db "SELECT * FROM credentials;"
```

## Resources

### Official Documentation
- [svcrack GitHub](https://github.com/EnableSecurity/sipvicious)
- [sipvicious Wiki](https://github.com/EnableSecurity/sipvicious/wiki)

### Related Tools
- **hydra**: General-purpose brute-force
- **medusa**: Network brute-force
- **ncrack**: Network authentication cracking
- **siparmyknife**: SIP testing tool

### Wordlists
- [SecLists](https://github.com/danielmiessler/SecLists)
- [CrackStation](https://crackstation.net/crackstation-wordlists/)
- [SecLists VoIP](https://github.com/danielmiessler/SecLists/tree/master/Discovery/VoIP)

### Related MITRE ATT&CK Techniques
- **T1110.001 - Password Guessing**: Dictionary attacks
- **T1110.003 - Password Spraying**: Testing common passwords
- **T1078 - Valid Accounts**: Using discovered credentials
