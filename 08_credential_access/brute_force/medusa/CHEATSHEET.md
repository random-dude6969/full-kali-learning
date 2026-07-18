# Medusa Cheatsheet

## Quick Reference

### SSH Brute Force

```bash
# Single user, password list
medusa -h 192.168.1.100 -u root -P /usr/share/wordlists/rockyou.txt -M ssh

# User list, password list
medusa -h 192.168.1.100 -U usernames.txt -P passwords.txt -M ssh

# Multiple hosts
medusa -H targets.txt -u root -P passwords.txt -M ssh

# Custom port
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

```bash
# Basic authentication
medusa -h 192.168.1.100 -u admin -P passwords.txt -M http -m DIR:/

# Form-based
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

```bash
# Create combo file (host:user:pass)
echo "192.168.1.100:admin:password123" > combo.txt
echo "192.168.1.101:root:toor" >> combo.txt

# Attack with combo file
medusa -C combo.txt -M ssh
```

### Parallel Attacks

```bash
# Multiple hosts, multiple users
medusa -H targets.txt -U usernames.txt -P passwords.txt -M ssh -T 10 -t 5

# Parallel login testing
medusa -h 192.168.1.100 -U usernames.txt -P passwords.txt -M ssh -L
```

### Resume Interrupted Scan

```bash
medusa -Z "192.168.1.100:ssh:user:pass" -h 192.168.1.100 -U usernames.txt -P passwords.txt -M ssh
```

### SSL/TLS Attacks

```bash
# HTTPS
medusa -h 192.168.1.100 -u admin -P passwords.txt -M http -s -m DIR:/

# FTPS
medusa -h 192.168.1.100 -u admin -P passwords.txt -M ftp -s
```

## Command Options

| Option | Description |
|--------|-------------|
| `-h HOST` | Single target hostname or IP |
| `-H FILE` | Target list file |
| `-u USER` | Single username |
| `-U FILE` | Username list file |
| `-p PASS` | Single password |
| `-P FILE` | Password list file |
| `-C FILE` | Combo file (host:user:pass) |
| `-M MODULE` | Module to execute |
| `-m PARAM` | Module parameter |
| `-d` | Dump all modules |
| `-n PORT` | Non-default TCP port |
| `-s` | Enable SSL |
| `-g SECS` | Give up after SECS seconds (default: 3) |
| `-r SECS` | Sleep SECS seconds between retries (default: 3) |
| `-R NUM` | Attempt NUM retries |
| `-c USECS` | Time to wait to verify socket |
| `-t NUM` | Concurrent logins |
| `-T NUM` | Concurrent hosts |
| `-L` | Parallelize logins |
| `-f` | Stop host after first valid credential |
| `-F` | Stop audit after first valid credential |
| `-b` | Suppress startup banner |
| `-q` | Display module usage |
| `-v NUM` | Verbose level (0-6) |
| `-w NUM` | Error debug level (0-10) |
| `-V` | Display version |
| `-Z TEXT` | Resume scan |

## Supported Modules

| Module | Description | Default Port |
|--------|-------------|--------------|
| `ssh` | SSH | 22 |
| `rdp` | RDP | 3389 |
| `ftp` | FTP | 21 |
| `telnet` | Telnet | 23 |
| `http` | HTTP | 80 |
| `https` | HTTPS | 443 |
| `mysql` | MySQL | 3306 |
| `mssql` | MSSQL | 1433 |
| `postgres` | PostgreSQL | 5432 |
| `oracle` | Oracle | 1521 |
| `smb` | SMB | 445 |
| `ldap` | LDAP | 389 |
| `vnc` | VNC | 5900 |
| `snmp` | SNMP | 161 |
| `smtp` | SMTP | 25 |
| `pop3` | POP3 | 110 |
| `imap` | IMAP | 143 |

## Common Workflows

### Discover and Attack

```bash
# 1. Find services
nmap -p 22,80,445,3306,3389 --open 192.168.1.0/24 -oG - > services.txt

# 2. Extract targets per service
grep "22/open" services.txt | awk '{print $2}' > ssh_hosts.txt
grep "3389/open" services.txt | awk '{print $2}' > rdp_hosts.txt

# 3. Brute force
medusa -H ssh_hosts.txt -u root -P passwords.txt -M ssh
medusa -H rdp_hosts.txt -u administrator -P passwords.txt -M rdp
```

### Combo File Attack

```bash
# Create combo file
echo "192.168.1.100:admin:password123" > combo.txt
echo "192.168.1.100:root:toor" >> combo.txt
echo "192.168.1.101:user:pass" >> combo.txt

# Attack with combo file
medusa -C combo.txt -M ssh
```

### Multi-Host Attack

```bash
# Attack entire subnet
medusa -H targets.txt -u root -P passwords.txt -M ssh -T 10 -t 5
```

## Troubleshooting

### Common Errors

**"Module not found"**:
```bash
medusa -d
ls /usr/share/medusa/modules/
```

**"Connection refused"**:
```bash
nmap -p 22 192.168.1.100
ssh root@192.168.1.100
```

**"Too many connections"**:
```bash
medusa -h 192.168.1.100 -u root -P passwords.txt -M ssh -t 2 -r 5
```

**"SSL handshake failed"**:
```bash
medusa -h 192.168.1.100 -u admin -P passwords.txt -M http -s
```

## OpSec Tips

1. **Reduce concurrent connections**: Use `-t 1` to avoid detection
2. **Add delay**: Use `-r 10` to slow down attempts
3. **Use verbose mode**: Use `-v 6` for detailed logging
4. **Log to file**: Use `-O` for audit trail

## Resources

- **GitHub**: https://github.com/jmk-foofus/medusa
- **Kali Docs**: https://www.kali.org/tools/medusa/
- **Documentation**: https://jmk-foofus.github.io/medusa/medusa.html
