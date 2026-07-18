---
title: "sipvicious - SIP Security Audit Toolkit"
description: "Comprehensive SIP security assessment suite for scanning, enumerating, and cracking SIP devices and services"
mitre_tactic: discovery
mitre_tactic_id: TA0007
mitre_subcategory: voip
categories:
  - discovery
  - voip
  - security-audit
  - penetration-testing
tags:
  - sip
  - voip
  - security-audit
  - brute-force
  - enumeration
install_command: "sudo apt install sipvicious"
version: "0.3.4"
github_repo: "https://github.com/EnableSecurity/sipvicious"
kali_tools_page: "https://tools.kali.org/information-gathering-tools/sipvicious"
difficulty_level: intermediate
requires_root: false
gui_available: false
related_tools:
  - svmap
  - svwar
  - svcrack
  - svreport
  - svcrash
---

# sipvicious - SIP Security Audit Toolkit

sipvicious is a comprehensive SIP security assessment suite comprising multiple specialized tools for VoIP penetration testing. The toolkit provides capabilities for network scanning, extension enumeration, credential brute-forcing, and result reporting, making it the de facto standard for SIP security audits.

## 1. Introduction

sipvicious revolutionizes VoIP security testing by providing a unified framework of interconnected tools that work together seamlessly. Each module addresses a specific phase of SIP security assessment, from initial reconnaissance through credential testing to final reporting.

The suite's modular design allows security professionals to use individual tools independently or chain them together for comprehensive assessments. All modules share common libraries for SIP packet construction, authentication handling, and result storage, ensuring consistency and reliability across the entire assessment workflow.

### Suite Components

| Module | Purpose | Primary Function |
|--------|---------|------------------|
| svmap | Network Scanner | Discover SIP devices on network |
| svwar | War Dialer | Enumerate valid extensions |
| svcrack | Credential Cracker | Brute-force SIP passwords |
| svreport | Reporter | Generate assessment reports |
| svcrash | Controller | Stop running scans |

### Architecture

```
┌─────────────────────────────────────────────────────┐
│                    sipvicious                        │
├─────────────┬─────────────┬─────────────┬───────────┤
│    svmap    │    svwar    │   svcrack   │  svreport │
│  (Scanner)  │ (Enum)      │  (Cracker)  │ (Reports) │
├─────────────┴─────────────┴─────────────┴───────────┤
│              Shared SIP Library                      │
│  ┌──────────┬──────────┬──────────┬──────────┐      │
│  │ Packet   │ Auth     │ Storage  │ Database │      │
│  │ Builder  │ Handler  │ Manager  │ Layer    │      │
│  └──────────┴──────────┴──────────┴──────────┘      │
├─────────────────────────────────────────────────────┤
│                    SQLite DB                         │
└─────────────────────────────────────────────────────┘
```

### Use Cases

| Scenario | Module(s) | Description |
|----------|-----------|-------------|
| Network Discovery | svmap | Map SIP infrastructure |
| Extension Enumeration | svwar | Find valid SIP extensions |
| Password Audit | svcrack | Test extension passwords |
| Full Assessment | All modules | Complete SIP security audit |
| Compliance Testing | svreport | Generate compliance reports |

## 2. Installation

### Kali Linux

```bash
sudo apt update
sudo apt install sipvicious
```

### From Source (Latest)

```bash
git clone https://github.com/EnableSecurity/sipvicious.git
cd sipvicious
sudo python3 setup.py install
```

### Verify Installation

```bash
svmap --version
svwar --version
svcrack --version
```

### Package Details

| Property | Value |
|----------|-------|
| Package Name | sipvicious |
| Default Install | Yes (Kali) |
| Dependencies | python3, python3-pysip |
| Architecture | all |
| License | GPL |

### Module Paths

```bash
# Find installed modules
which svmap
which svwar
which svcrack
which svreport
which svcrash
```

## 3. Core Concepts

### SIP Network Model

```
┌─────────────────────────────────────────────────────┐
│                   SIP Network                        │
├─────────────────────────────────────────────────────┤
│                                                      │
│  ┌──────────┐     ┌──────────┐     ┌──────────┐    │
│  │   SIP    │────▶│   SIP    │────▶│   SIP    │    │
│  │  Proxy   │     │  Server  │     │  Phones  │    │
│  └──────────┘     └──────────┘     └──────────┘    │
│       │                │                │            │
│       └────────────────┴────────────────┘            │
│                    │                                 │
│             ┌──────┴──────┐                          │
│             │   Network   │                          │
│             │   Segment   │                          │
│             └─────────────┘                          │
│                                                      │
└─────────────────────────────────────────────────────┘
```

### Assessment Phases

1. **Reconnaissance (svmap)**: Identify active SIP endpoints
2. **Enumeration (svwar)**: Discover valid extensions/users
3. **Exploitation (svcrack)**: Test extension passwords
4. **Reporting (svreport)**: Document findings

### SIP Registration Flow

```
Phone                              Server
  |                                   |
  |--- REGISTER ------------------->  |
  |<-- 401 Unauthorized ------------  |
  |--- REGISTER + Credentials ----->  |
  |<-- 200 OK ---------------------  |
  |                                   |
```

### Database Schema

sipvicious stores results in SQLite databases:

```sql
-- Devices discovered
CREATE TABLE devices (
    ip TEXT,
    port INTEGER,
    user_agent TEXT,
    timestamp DATETIME
);

-- Extensions found
CREATE TABLE extensions (
    extension TEXT,
    status TEXT,
    method TEXT,
    timestamp DATETIME
);

-- Credentials cracked
CREATE TABLE credentials (
    extension TEXT,
    password TEXT,
    timestamp DATETIME
);
```

## 4. Command Reference

### svmap - Network Scanner

```bash
# Basic scan
svmap 192.168.1.0/24

# Scan specific ports
svmap -p 5060-5061 192.168.1.0/24

# Identify devices
svmap --identify 192.168.1.0/24

# Rate limiting
svmap -r 50 192.168.1.0/24
```

**Key Flags:**

| Flag | Description | Example |
|------|-------------|---------|
| `-p` | Port range | `-p 5060-5061` |
| `-s` | Source port | `-s 5060` |
| `-l` | Local interface | `-l eth0` |
| `--identify` | Fingerprint devices | `--identify` |
| `-e` | Extension range | `-e 1000-9999` |
| `-m` | Method | `-m OPTIONS` |
| `-S` | Stealth mode | `-S` |
| `-r` | Rate (pkt/s) | `-r 100` |

### svwar - Extension War Dialer

```bash
# Numeric range enumeration
svwar -e 1000-9999 192.168.1.100

# Dictionary enumeration
svwar -d extensions.txt 192.168.1.100

# Method selection
svwar -e 1000-9999 -m REGISTER 192.168.1.100

# Resume scan
svwar --resume 192.168.1.100
```

**Key Flags:**

| Flag | Description | Example |
|------|-------------|---------|
| `-e` | Extension range | `-e 1000-9999` |
| `-d` | Dictionary file | `-d ext.txt` |
| `-m` | Method (REGISTER/INVITE/OPTIONS) | `-m REGISTER` |
| `-r` | Rate | `-r 50` |
| `-c` | Concurrency | `-c 10` |
| `-p` | Port | `-p 5060` |
| `--resume` | Resume interrupted scan | `--resume` |

### svcrack - Credential Cracker

```bash
# Dictionary attack
svcrack -u 1000 -d passwords.txt 192.168.1.100

# Numeric range
svcrack -u 1000 -c 1000-9999 192.168.1.100

# Multiple extensions
svcrack -e 1000-1100 -d passwords.txt 192.168.1.100

# Resume attack
svcrack --resume 192.168.1.100
```

**Key Flags:**

| Flag | Description | Example |
|------|-------------|---------|
| `-u` | Extension | `-u 1000` |
| `-d` | Dictionary | `-d passwords.txt` |
| `-e` | Extension range | `-e 1000-9999` |
| `-c` | Password range | `-c 1000-9999` |
| `-p` | Port | `-p 5060` |
| `-r` | Rate | `-r 50` |
| `-S` | Stealth mode | `-S` |
| `--resume` | Resume attack | `--resume` |

### svreport - Report Generator

```bash
# List stored results
svreport list

# Export to CSV
svreport export -f csv -o results.csv

# Export to PDF
svreport export -f pdf -o report.pdf

# Search results
svreport search --query "extension:1000"

# Purge old results
svreport purge --older-than 30
```

**Commands:**

| Command | Description |
|---------|-------------|
| `list` | List all stored results |
| `export` | Export results to file |
| `search` | Search stored results |
| `delete` | Delete specific results |
| `purge` | Remove old results |

### svcrash - Scan Controller

```bash
# Stop all scans on target
svcrash --fromip=192.168.1.100

# Stop scans on specific port
svcrash --fromport=5060 --toport=5080

# Stop specific scan by server
svcrash --server=192.168.1.100

# Force stop
svcrash --proto=udp --fromip=192.168.1.100
```

**Key Flags:**

| Flag | Description | Example |
|------|-------------|---------|
| `--fromip` | Target IP | `--fromip=192.168.1.100` |
| `--fromport` | Start port | `--fromport=5060` |
| `--toport` | End port | `--toport=5080` |
| `--server` | SIP server | `--server=192.168.1.100` |
| `--proto` | Protocol | `--proto=udp` |

## 5. Examples

### Complete Assessment Workflow

```bash
# Phase 1: Network Discovery
svmap 192.168.1.0/24 -o scan_results.txt

# Phase 2: Extension Enumeration
svwar -e 1000-9999 -m REGISTER 192.168.1.100

# Phase 3: Password Testing
svcrack -e 1000-1100 -d /usr/share/wordlists/rockyou.txt 192.168.1.100

# Phase 4: Report Generation
svreport export -f pdf -o voip_audit_report.pdf
```

### Stealthy Network Scan

```bash
# Slow, stealthy scan
svmap -S -r 10 -p 5060 192.168.1.0/24

# Through proxy
svmap -P 192.168.1.1 -p 5060 192.168.1.0/24

# Custom User-Agent
svmap --user-agent "Polycom/5.0" 192.168.1.0/24
```

### Extension Enumeration

```bash
# Fast enumeration
svwar -e 1000-9999 -c 20 192.168.1.100

# Using INVITE method (more reliable)
svwar -e 1000-9999 -m INVITE 192.168.1.100

# Dictionary-based
svwar -d /usr/share/seclists/Discovery/VoIP/extensions.txt 192.168.1.100

# Resume interrupted scan
svwar --resume 192.168.1.100
```

### Credential Brute-Force

```bash
# Dictionary attack on single extension
svcrack -u 1000 -d /usr/share/wordlists/rockyou.txt 192.168.1.100

# Range of extensions
svcrack -e 1000-1100 -d passwords.txt 192.168.1.100

# Numeric PIN range
svcrack -u 1000 -c 1000-9999 192.168.1.100

# Stealthy brute-force
svcrack -u 1000 -d passwords.txt -r 10 -S 192.168.1.100
```

### Advanced Scenarios

```bash
# Scan multiple subnets
svmap 192.168.1.0/24 192.168.2.0/24

# Scan with specific source interface
svmap -l eth1 192.168.1.0/24

# Export and analyze
svreport export -f csv -o results.csv
cut -d',' -f1 results.csv | sort -u > extensions.txt
```

## 6. Advanced Usage

### Automation Scripts

```bash
#!/bin/bash
# Full SIP audit script
TARGET="$1"
OUTPUT="/tmp/sip_audit_$(date +%Y%m%d)"

mkdir -p $OUTPUT

echo "[*] Phase 1: Network Discovery"
svmap $TARGET -o $OUTPUT/scan.txt

echo "[*] Phase 2: Extension Enumeration"
svwar -e 1000-9999 -m REGISTER $TARGET -o $OUTPUT/extensions.txt

echo "[*] Phase 3: Password Testing"
cat $OUTPUT/extensions.txt | while read ext; do
    svcrack -u $ext -d /usr/share/wordlists/rockyou.txt $TARGET
done

echo "[*] Phase 4: Report Generation"
svreport export -f pdf -o $OUTPUT/audit_report.pdf

echo "[+] Audit complete: $OUTPUT"
```

### Performance Optimization

| Parameter | Value | Use Case |
|-----------|-------|----------|
| `-r 100` | 100 pkt/s | Fast LAN scan |
| `-r 10` | 10 pkt/s | Stealthy scan |
| `-c 20` | 20 threads | Aggressive enum |
| `-c 5` | 5 threads | Low-profile enum |

### Integration with Other Tools

```bash
# Combine with Nmap
nmap -sU -p 5060 192.168.1.0/24 -oG - | \
    grep "5060/open" | \
    awk '{print $2}' | \
    while read ip; do svmap $ip; done

# Pipe to Metasploit
svmap 192.168.1.0/24 | grep -oP '\d+\.\d+\.\d+\.\d+' | \
    msfconsole -x "use auxiliary/scanner/sip/sipderegister; set RHOSTS file:/dev/stdin"
```

## 7. Detection and Defense

### Detection Indicators

| Indicator | svmap | svwar | svcrack |
|-----------|-------|-------|---------|
| User-Agent | sipvicious | sipvicious | sipvicious |
| Pattern | OPTIONS flood | REGISTER attempts | REGISTER failures |
| Rate | Variable | High | High |
| Duration | Short | Medium | Long |

### Defensive Measures

**Network:**

```bash
# Rate limiting
iptables -A INPUT -p udp --dport 5060 \
    -m hashlimit --hashlimit-above 20/sec \
    --hashlimit-burst 50 \
    --hashlimit-mode srcip \
    -j DROP

# Detect sipvicious User-Agent
iptables -A INPUT -p udp --dport 5060 -m string \
    --string "sipvicious" -j LOG --log-prefix "SIPVICIOUS: "
```

**SIP Server:**

```
# FreeSWITCH - fail2ban integration
# /etc/fail2ban/jail.d/sip.conf
[sipvicious]
enabled = true
filter = sipvicious
action = iptables-allports[name=sipvicious]
logpath = /var/log/freeswitch/freeswitch.log
maxretry = 5
bantime = 3600
```

### Countermeasures for Testers

- Use `-S` stealth mode
- Reduce rate with `-r`
- Spoof User-Agent with `--user-agent`
- Use `-P` proxy routing
- Break scans into smaller ranges

## 8. Troubleshooting

### Common Issues

#### Database Errors

```bash
# Reset database
rm ~/.sipvicious/sipvicious.db

# Check database
sqlite3 ~/.sipvicious/sipvicious.db ".tables"
```

#### Scan Not Resuming

```bash
# Check scan state
ls ~/.sipvicious/

# Force resume
svwar --resume --force 192.168.1.100
```

#### No Results

```bash
# Verify target
ping 192.168.1.100
nc -zvu 192.168.1.100 5060

# Check with sipsak
sipsak -s sip:user@192.168.1.100 -v
```

### Performance Issues

| Symptom | Solution |
|---------|----------|
| Slow scans | Increase `-r` rate |
| High CPU | Decrease `-c` concurrency |
| Packet loss | Reduce rate, add `-S` |
| Memory issues | Clear old results with `svreport purge` |

### Debug Commands

```bash
# Verbose output
svmap -v 192.168.1.100

# Check running scans
ps aux | grep sv

# View database
sqlite3 ~/.sipvicious/sipvicious.db "SELECT * FROM devices;"
```

## Resources

### Official Documentation
- [sipvicious GitHub](https://github.com/EnableSecurity/sipvicious)
- [sipvicious Wiki](https://github.com/EnableSecurity/sipvicious/wiki)
- [Original Paper](https://www.sensepost.com/publications/l01-lon20-voip/)

### Related Tools
- **sipsak**: Quick SIP requests
- **siparmyknife**: Alternative SIP tester
- **hydra**: General brute-force tool
- **nmap**: Network scanner

### Learning Resources
- [VoIP Hacking](https://www.hackingexposedvoip.com/)
- [SIP Security Assessment](https://www.owasp.org/index.php/VoIP)
- [SANS VoIP Security](https://www.sans.org/white-papers/voip-security/)

### Related MITRE ATT&CK Techniques
- **T1046 - Network Service Discovery**: SIP scanning
- **T1110.001 - Password Guessing**: Extension brute-force
- **T1018 - Remote System Discovery**: Device enumeration
- **T1083 - File and Directory Discovery**: Database extraction
