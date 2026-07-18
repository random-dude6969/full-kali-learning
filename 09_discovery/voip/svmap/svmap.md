---
title: "svmap - SIP Network Scanner"
description: "High-performance SIP network scanner for discovering and fingerprinting VoIP devices across network segments"
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
  - network-scanning
  - device-discovery
  - fingerprinting
install_command: "sudo apt install sipvicious"
version: "0.3.4"
github_repo: "https://github.com/EnableSecurity/sipvicious"
kali_tools_page: "https://tools.kali.org/information-gathering-tools/sipvicious"
difficulty_level: intermediate
requires_root: false
gui_available: false
related_tools:
  - sipvicious
  - nmap
  - svwar
  - svreport
---

# svmap - SIP Network Scanner

svmap is a high-performance SIP network scanner designed for discovering VoIP devices and fingerprinting SIP software implementations. As part of the sipvicious suite, it provides rapid network reconnaissance capabilities essential for VoIP security assessments.

## 1. Introduction

svmap addresses the fundamental need to identify SIP-enabled devices across network segments. Unlike generic network scanners, svmap understands SIP protocol specifics, enabling accurate device identification, software fingerprinting, and capability assessment.

The tool sends SIP OPTIONS or INVITE requests to target ranges and analyzes responses to identify active SIP endpoints. It extracts valuable information including User-Agent headers, supported methods, and device capabilities, providing a comprehensive view of the VoIP infrastructure.

### Key Capabilities

| Feature | Description |
|---------|-------------|
| Network Scanning | CIDR notation, IP ranges, single targets |
| Device Fingerprinting | Software identification via User-Agent |
| Port Scanning | Configurable SIP port ranges |
| Method Selection | OPTIONS, INVITE, REGISTER probes |
| Rate Control | Bandwidth limiting for stealth |
| Stealth Mode | Reduced detection profile |
| Resume Support | Continue interrupted scans |
| Result Storage | SQLite database for analysis |

### Scan Types

| Type | Description | Use Case |
|------|-------------|----------|
| Quick Scan | Single OPTIONS request | Fast host check |
| Full Scan | Multi-method probes | Comprehensive assessment |
| Stealth Scan | Rate-limited, slow | Evasion-focused |
| Targeted Scan | Specific port/extension | Precision testing |

## 2. Installation

### Kali Linux

```bash
sudo apt update
sudo apt install sipvicious
```

### Verify Installation

```bash
svmap --version
which svmap
```

### Dependencies

| Package | Purpose |
|---------|---------|
| python3 | Runtime |
| python3-pysip | SIP library |
| sqlite3 | Result storage |

## 3. Core Concepts

### SIP Discovery Model

```
┌─────────────────────────────────────────────────────┐
│                   Network Segment                    │
├─────────────────────────────────────────────────────┤
│                                                      │
│  svmap sends OPTIONS requests to all IPs            │
│                                                      │
│  ┌─────┐    OPTIONS    ┌─────┐                      │
│  │svmap│──────────────▶│ PBX │                      │
│  │     │◀──────────────│     │  ← 200 OK            │
│  └─────┘    Response   └─────┘                      │
│                                                      │
│  ┌─────┐    OPTIONS    ┌─────┐                      │
│  │svmap│──────────────▶│Phone│                      │
│  │     │◀──────────────│     │  ← 200 OK            │
│  └─────┘    Response   └─────┘                      │
│                                                      │
│  ┌─────┐    OPTIONS    ┌─────┐                      │
│  │svmap│──────────────▶│ FW  │                      │
│  │     │    No Response│     │  ← No SIP            │
│  └─────┘               └─────┘                      │
│                                                      │
└─────────────────────────────────────────────────────┘
```

### Response Analysis

| Response | Meaning | Action |
|----------|---------|--------|
| 200 OK | SIP device active | Record device info |
| 401 Unauthorized | Auth required | Note protected device |
| 403 Forbidden | Access denied | Note restricted device |
| 404 Not Found | Extension unknown | Skip |
| Timeout | No response | Not a SIP device |
| ICMP Unreachable | Host down | Skip |

### Fingerprinting Methods

1. **User-Agent Extraction**: Identify software from header
2. **Method Support**: Test OPTIONS, INVITE, REGISTER
3. **Header Analysis**: Examine Via, Contact, Record-Route
4. **Behavioral Analysis**: Response patterns and timing

## 4. Command Reference

### Basic Syntax

```
svmap [options] <target>
```

### Target Specification

| Syntax | Description | Example |
|--------|-------------|---------|
| IP Address | Single host | `192.168.1.100` |
| CIDR | Network range | `192.168.1.0/24` |
| Range | IP range | `192.168.1.1-192.168.1.254` |
| Hostname | DNS name | `sip.example.com` |

### Scan Options

| Flag | Description | Example |
|------|-------------|---------|
| `-p` | Port range | `-p 5060-5061` |
| `-s` | Source port | `-s 5060` |
| `-l` | Local interface | `-l eth0` |
| `-m` | SIP method | `-m OPTIONS` |
| `-e` | Extension range | `-e 1000-9999` |
| `--identify` | Fingerprint devices | `--identify` |

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
| 2 | No devices found |

## 5. Examples

### Basic Network Scan

```bash
# Scan /24 subnet
svmap 192.168.1.0/24

# Scan single host
svmap 192.168.1.100

# Scan IP range
svmap 192.168.1.1-192.168.1.50
```

### Port Scanning

```bash
# Scan standard SIP ports
svmap -p 5060-5061 192.168.1.0/24

# Scan non-standard ports
svmap -p 5080-5090 192.168.1.0/24

# Single port
svmap -p 5060 192.168.1.0/24
```

### Device Fingerprinting

```bash
# Identify devices
svmap --identify 192.168.1.0/24

# Extract User-Agent
svmap --identify -v 192.168.1.0/24

# Custom fingerprinting
svmap -m INVITE --identify 192.168.1.0/24
```

### Rate Control

```bash
# Fast scan (LAN)
svmap -r 200 192.168.1.0/24

# Balanced scan
svmap -r 100 192.168.1.0/24

# Slow, stealthy scan
svmap -r 10 -S 192.168.1.0/24

# Very slow (WAN)
svmap -r 5 -S 192.168.1.0/24
```

### Stealth Scanning

```bash
# Stealth mode
svmap -S 192.168.1.0/24

# Custom User-Agent
svmap --user-agent "Polycom/5.0" 192.168.1.0/24

# Through proxy
svmap -P 192.168.1.1 192.168.1.0/24

# Source port spoofing
svmap -s 5060 192.168.1.0/24
```

### Extension Enumeration

```bash
# Enumerate extensions
svmap -e 1000-9999 192.168.1.100

# Fast enumeration
svmap -e 1000-9999 -r 200 192.168.1.100

# Stealthy enumeration
svmap -e 1000-9999 -r 10 -S 192.168.1.100
```

### Advanced Scenarios

```bash
# Scan multiple subnets
svmap 192.168.1.0/24 192.168.2.0/24

# Scan with source interface
svmap -l eth1 192.168.1.0/24

# Save results
svmap 192.168.1.0/24 -o scan_results.txt

# Resume interrupted scan
svmap --resume 192.168.1.0/24
```

## 6. Advanced Usage

### Automation Scripts

```bash
#!/bin/bash
# Comprehensive SIP network scan
TARGET="$1"
OUTPUT="/tmp/svmap_$(date +%Y%m%d_%H%M%S)"

echo "[*] Starting SIP scan of $TARGET"
svmap --identify -r 100 -o ${OUTPUT}.txt $TARGET

echo "[*] Results saved to ${OUTPUT}.txt"
echo "[*] Device count:"
grep -c "200 OK" ${OUTPUT}.txt

echo "[*] User-Agents found:"
grep -oP "User-Agent: \K.*" ${OUTPUT}.txt | sort -u
```

### Performance Tuning

| Scenario | Rate | Concurrency | Timeout | Notes |
|----------|------|-------------|---------|-------|
| Local LAN | 200 | 20 | 3s | Fast, reliable |
| Remote WAN | 25 | 10 | 10s | Account for latency |
| Stealth | 10 | 5 | 5s | Low profile |
| Cloud/VPN | 50 | 15 | 8s | Moderate speed |

### Integration with Other Tools

```bash
# Combine with Nmap
nmap -sU -p 5060 192.168.1.0/24 -oG - | \
    grep "5060/open" | awk '{print $2}' > sip_hosts.txt
while read host; do svmap $host; done < sip_hosts.txt

# Export to CSV
svmap 192.168.1.0/24 -o results.txt
awk '/200 OK/ {print $1}' results.txt > active_hosts.txt

# Feed to svwar
svmap 192.168.1.0/24 -e 1000-9999 | \
    grep -oP '\d+\.\d+\.\d+\.\d+' > targets.txt
```

## 7. Detection and Defense

### Detection Indicators

| Indicator | Description |
|-----------|-------------|
| User-Agent | sipvicious/svmap |
| Pattern | OPTIONS flood |
| Rate | Consistent packet timing |
| Source | Single IP scanning |

### Defensive Measures

**Network:**

```bash
# Rate limiting
iptables -A INPUT -p udp --dport 5060 \
    -m hashlimit --hashlimit-above 20/sec \
    --hashlimit-burst 50 \
    --hashlimit-mode srcip \
    -j DROP

# Detect sipvicious
iptables -A INPUT -p udp --dport 5060 -m string \
    --string "sipvicious" -j LOG --log-prefix "SVMAP: "
```

**SIP Server:**

```
# FreeSWITCH - fail2ban
# /etc/fail2ban/jail.d/sip.conf
[svmap]
enabled = true
filter = svmap
action = iptables-allports[name=svmap]
logpath = /var/log/freeswitch/freeswitch.log
maxretry = 10
bantime = 3600
```

### Countermeasures for Testers

- Use `-S` stealth mode
- Reduce rate with `-r 10`
- Spoof User-Agent with `--user-agent`
- Use proxy routing with `-P`
- Vary scan timing

## 8. Troubleshooting

### Common Issues

#### No Devices Found

```bash
# Verify target
ping 192.168.1.100
nc -zvu 192.168.1.100 5060

# Test with sipsak
sipsak -s sip:user@192.168.1.100 -v

# Check firewall
sudo iptables -L -n | grep 5060
```

#### Slow Scans

```bash
# Increase rate
svmap -r 200 192.168.1.0/24

# Reduce timeout
svmap -t 3 192.168.1.0/24

# Skip unreachable hosts
svmap --skip-unreachable 192.168.1.0/24
```

#### Scan Stuck

```bash
# Kill process
pkill -f svmap

# Reset state
rm ~/.sipvicious/svmap_*

# Resume scan
svmap --resume 192.168.1.0/24
```

#### Database Errors

```bash
# Check database
ls -la ~/.sipvicious/*.db

# Reset database
rm ~/.sipvicious/svmap_*.db

# View database
sqlite3 ~/.sipvicious/svmap_*.db "SELECT * FROM devices;"
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
svmap -v 192.168.1.100

# Debug mode
svmap --debug 192.168.1.100

# Check state files
ls -la ~/.sipvicious/

# View database
sqlite3 ~/.sipvicious/svmap_*.db ".tables"
```

## Resources

### Official Documentation
- [svmap GitHub](https://github.com/EnableSecurity/sipvicious)
- [sipvicious Wiki](https://github.com/EnableSecurity/sipvicious/wiki)

### Related Tools
- **nmap**: General network scanner
- **svwar**: Extension war dialer
- **sipsak**: Quick SIP requests
- **svreport**: Report generator

### Learning Resources
- [SIP Protocol](https://www.rfc-editor.org/rfc/rfc3261)
- [VoIP Security](https://www.owasp.org/index.php/VoIP)
- [Network Scanning](https://www.sans.org/white-papers/network-scanning/)

### Related MITRE ATT&CK Techniques
- **T1046 - Network Service Discovery**: SIP scanning
- **T1018 - Remote System Discovery**: Device enumeration
- **T1592 - Gather Victim Host Information**: Fingerprinting
