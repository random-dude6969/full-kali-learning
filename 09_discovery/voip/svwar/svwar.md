---
title: "svwar - SIP Extension War Dialer"
description: "SIP extension enumeration tool for discovering valid extensions via REGISTER and INVITE methods"
mitre_tactic: discovery
mitre_tactic_id: TA0007
mitre_subcategory: voip
categories:
  - discovery
  - voip
  - enumeration
  - war-dialing
tags:
  - sip
  - voip
  - enumeration
  - war-dialing
  - extension-discovery
install_command: "sudo apt install sipvicious"
version: "0.3.4"
github_repo: "https://github.com/EnableSecurity/sipvicious"
kali_tools_page: "https://tools.kali.org/information-gathering-tools/sipvicious"
difficulty_level: intermediate
requires_root: false
gui_available: false
related_tools:
  - sipvicious
  - svmap
  - svcrack
  - enumiax
---

# svwar - SIP Extension War Dialer

svwar is a SIP extension enumeration tool that systematically discovers valid extensions on VoIP systems. As part of the sipvicious suite, it provides war dialing capabilities using REGISTER, INVITE, and OPTIONS methods to identify active SIP endpoints.

## 1. Introduction

svwar addresses the critical reconnaissance phase of VoIP security assessments by identifying valid extensions. Valid extensions represent potential attack vectors for credential brute-forcing, toll fraud, and unauthorized access.

The tool works by sending SIP requests to a range of extensions and analyzing responses to determine which extensions are valid, protected, or non-existent. This information is essential for targeted security testing and network inventory.

### Attack Methods

| Method | Description | Reliability |
|--------|-------------|-------------|
| REGISTER | Registration attempt | High |
| INVITE | Call initiation | Very High |
| OPTIONS | Capability query | Medium |

### Response Analysis

| Response | Meaning | Action |
|----------|---------|--------|
| 200 OK | Extension active, no auth | Record as vulnerable |
| 401 Unauthorized | Auth required | Record as valid |
| 403 Forbidden | Access denied | Record as restricted |
| 404 Not Found | Extension invalid | Skip |
| 408 Timeout | No response | Unknown |

## 2. Installation

### Kali Linux

```bash
sudo apt update
sudo apt install sipvicious
```

### Verify Installation

```bash
svwar --version
which svwar
```

### Dependencies

| Package | Purpose |
|---------|---------|
| python3 | Runtime |
| python3-pysip | SIP library |
| sqlite3 | Result storage |

## 3. Core Concepts

### War Dialing Model

```
┌─────────────────────────────────────────────────────┐
│                  svwar Operation                     │
├─────────────────────────────────────────────────────┤
│                                                      │
│  Target Range: 1000-1010                             │
│                                                      │
│  ┌─────────┐    REGISTER     ┌─────────┐            │
│  │  svwar  │──── ext 1000 ──▶│   PBX   │            │
│  │         │◀─── 401 Unauthorized ─────│            │
│  │         │──── ext 1001 ──▶│         │            │
│  │         │◀─── 401 Unauthorized ─────│            │
│  │         │──── ext 1002 ──▶│         │            │
│  │         │◀─── 404 Not Found ────────│            │
│  │         │──── ext 1003 ──▶│         │            │
│  │         │◀─── 200 OK ───────────────│            │
│  └─────────┘                 └─────────┘            │
│                                                      │
│  Results:                                            │
│  - 1000: Valid (401)                                 │
│  - 1001: Valid (401)                                 │
│  - 1002: Invalid (404)                               │
│  - 1003: Active, No Auth (200)                       │
│                                                      │
└─────────────────────────────────────────────────────┘
```

### Method Comparison

| Aspect | REGISTER | INVITE | OPTIONS |
|--------|----------|--------|---------|
| Stealth | Medium | Low | High |
| Reliability | High | Very High | Medium |
| Detection Risk | Medium | High | Low |
| Information | Extension validity | Extension + media | Basic capability |

### Extension Patterns

| Pattern | Example | Description |
|---------|---------|-------------|
| Sequential | 1000-1099 | Standard extension range |
| Departmental | 2000-2099 | Sales, Support, etc. |
| DID | 5551234-5551299 | Direct inward dialing |
| Random | Custom list | Non-sequential |

## 4. Command Reference

### Basic Syntax

```
svwar [options] <target>
```

### Target Specification

| Flag | Description | Example |
|------|-------------|---------|
| `-e` | Extension range | `-e 1000-9999` |
| `-d` | Dictionary file | `-d extensions.txt` |
| `-p` | SIP port | `-p 5060` |
| `<target>` | Target IP/hostname | `192.168.1.100` |

### Method Options

| Flag | Description | Example |
|------|-------------|---------|
| `-m` | SIP method | `-m REGISTER` |
| `-m REGISTER` | Registration method | Default |
| `-m INVITE` | Call method | `-m INVITE` |
| `-m OPTIONS` | Query method | `-m OPTIONS` |

### Performance Options

| Flag | Description | Default |
|------|-------------|---------|
| `-r` | Rate (pkt/s) | 100 |
| `-c` | Concurrency | 10 |
| `-t` | Timeout (s) | 5 |

### Stealth Options

| Flag | Description |
|------|-------------|
| `-S` | Stealth mode |
| `--user-agent` | Custom User-Agent |
| `-P` | Proxy address |

### Output Options

| Flag | Description |
|------|-------------|
| `-o` | Output file |
| `--resume` | Resume scan |
| `-v` | Verbose output |
| `--debug` | Debug output |

### Return Codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | Error |
| 2 | No extensions found |

## 5. Examples

### Basic Extension Enumeration

```bash
# Numeric range
svwar -e 1000-9999 192.168.1.100

# Small range
svwar -e 1000-1099 192.168.1.100

# Single extension test
svwar -e 1000-1000 192.168.1.100
```

### Method Selection

```bash
# REGISTER method (default)
svwar -e 1000-9999 -m REGISTER 192.168.1.100

# INVITE method (more reliable)
svwar -e 1000-9999 -m INVITE 192.168.1.100

# OPTIONS method (stealthy)
svwar -e 1000-9999 -m OPTIONS 192.168.1.100
```

### Dictionary Enumeration

```bash
# Custom extension list
svwar -d extensions.txt 192.168.1.100

# Common extensions
echo -e "1000\n1001\n1002\n1003" > common.txt
svwar -d common.txt 192.168.1.100

# Departmental extensions
svwar -d sales_extensions.txt 192.168.1.100
```

### Rate Control

```bash
# Fast scan (LAN)
svwar -e 1000-9999 -r 200 192.168.1.100

# Balanced
svwar -e 1000-9999 -r 100 192.168.1.100

# Slow, stealthy
svwar -e 1000-9999 -r 10 -S 192.168.1.100

# Very slow (WAN)
svwar -e 1000-9999 -r 5 -S 192.168.1.100
```

### Stealth Enumeration

```bash
# Stealth mode
svwar -e 1000-9999 -S 192.168.1.100

# Custom User-Agent
svwar -e 1000-9999 --user-agent "Polycom/5.0" 192.168.1.100

# Through proxy
svwar -e 1000-9999 -P 192.168.1.1 192.168.1.100

# Source port spoofing
svwar -e 1000-9999 -s 5060 192.168.1.100
```

### Resume Capability

```bash
# Start enumeration
svwar -e 1000-9999 192.168.1.100

# Interrupt with Ctrl+C

# Resume scan
svwar --resume 192.168.1.100

# Force resume
svwar --resume --force 192.168.1.100
```

### Advanced Scenarios

```bash
# Multiple targets
svwar -e 1000-9999 192.168.1.100 192.168.1.101

# Non-standard port
svwar -e 1000-9999 -p 5061 192.168.1.100

# IPv6 target
svwar -e 1000-9999 -6 ::1

# Save results
svwar -e 1000-9999 -o extensions.txt 192.168.1.100
```

## 6. Advanced Usage

### Automation Scripts

```bash
#!/bin/bash
# Multi-target extension enumeration
TARGETS="192.168.1.100 192.168.1.101 192.168.1.102"
for target in $TARGETS; do
    echo "[*] Enumerating $target"
    svwar -e 1000-9999 -r 100 -o "ext_${target}.txt" $target
done
```

### Performance Tuning

| Scenario | Rate | Concurrency | Method | Notes |
|----------|------|-------------|--------|-------|
| Local LAN | 200 | 20 | REGISTER | Fast, reliable |
| Remote WAN | 25 | 10 | REGISTER | Account for latency |
| Stealth | 10 | 5 | OPTIONS | Low profile |
| High-security | 5 | 3 | INVITE | Maximum stealth |

### Integration with Other Tools

```bash
# svmap → svwar
svmap 192.168.1.0/24           # Find SIP hosts
svwar -e 1000-9999 192.168.1.100  # Enumerate extensions

# svwar → svcrack
svwar -e 1000-9999 192.168.1.100  # Find extensions
svcrack -e 1000-1100 -d passwords.txt 192.168.1.100  # Crack passwords

# svwar → svreport
svwar -e 1000-9999 192.168.1.100  # Enumerate
svreport export -f pdf -o extensions.pdf  # Generate report
```

## 7. Detection and Defense

### Detection Indicators

| Indicator | Description |
|-----------|-------------|
| User-Agent | sipvicious/svwar |
| Pattern | Sequential REGISTER attempts |
| Rate | Consistent timing |
| Source | Single IP scanning |

### Defensive Measures

**Network:**

```bash
# Rate limiting
iptables -A INPUT -p udp --dport 5060 \
    -m hashlimit --hashlimit-above 10/sec \
    --hashlimit-burst 30 \
    --hashlimit-mode srcip \
    -j DROP

# Detect sipvicious
iptables -A INPUT -p udp --dport 5060 -m string \
    --string "sipvicious" -j LOG --log-prefix "SVWAR: "
```

**SIP Server:**

```
# FreeSWITCH - fail2ban
# /etc/fail2ban/jail.d/sip.conf
[svwar]
enabled = true
filter = svwar
action = iptables-allports[name=svwar]
logpath = /var/log/freeswitch/freeswitch.log
maxretry = 5
bantime = 3600

# Asterisk - sip.conf
[general]
maxauthfail = 3
authlimit = 5
```

### Countermeasures for Testers

- Use `-S` stealth mode
- Reduce rate with `-r 10`
- Spoof User-Agent with `--user-agent`
- Use proxy routing with `-P`
- Vary scan timing

## 8. Troubleshooting

### Common Issues

#### No Extensions Found

```bash
# Verify target
ping 192.168.1.100
nc -zvu 192.168.1.100 5060

# Test with sipsak
sipsak -s sip:1000@192.168.1.100 -v

# Try different method
svwar -e 1000-9999 -m INVITE 192.168.1.100
```

#### Slow Enumeration

```bash
# Increase rate
svwar -e 1000-9999 -r 200 192.168.1.100

# Reduce timeout
svwar -e 1000-9999 -t 3 192.168.1.100

# Skip unreachable
svwar -e 1000-9999 --skip-unreachable 192.168.1.100
```

#### Scan Stuck

```bash
# Kill process
pkill -f svwar

# Reset state
rm ~/.sipvicious/svwar_*

# Resume scan
svwar --resume 192.168.1.100
```

#### Database Errors

```bash
# Check database
ls -la ~/.sipvicious/*.db

# Reset database
rm ~/.sipvicious/svwar_*.db

# View database
sqlite3 ~/.sipvicious/svwar_*.db "SELECT * FROM extensions;"
```

### Performance Issues

| Symptom | Solution |
|---------|----------|
| Too slow | Increase `-r` rate |
| High CPU | Reduce `-c` concurrency |
| Packet loss | Add `-S`, reduce rate |
| Memory issues | Clear old results |

### Debug Commands

```bash
# Verbose output
svwar -v -e 1000-9999 192.168.1.100

# Debug mode
svwar --debug -e 1000-9999 192.168.1.100

# Check state files
ls -la ~/.sipvicious/

# View database
sqlite3 ~/.sipvicious/svwar_*.db ".tables"
```

## Resources

### Official Documentation
- [svwar GitHub](https://github.com/EnableSecurity/sipvicious)
- [sipvicious Wiki](https://github.com/EnableSecurity/sipvicious/wiki)

### Related Tools
- **enumiax**: IAX extension enumeration
- **siparmyknife**: SIP enumeration
- **svmap**: SIP network scanner
- **svcrack**: SIP credential cracker

### Learning Resources
- [SIP Protocol](https://www.rfc-editor.org/rfc/rfc3261)
- [VoIP Security](https://www.owasp.org/index.php/VoIP)
- [War Dialing](https://en.wikipedia.org/wiki/War_dialing)

### Related MITRE ATT&CK Techniques
- **T1046 - Network Service Discovery**: SIP scanning
- **T1018 - Remote System Discovery**: Extension enumeration
- **T1087 - Account Discovery**: Valid extension discovery
