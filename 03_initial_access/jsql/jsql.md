---
title: jSQL Injection
category: Initial Access
tags:
  - sql-injection
  - database
  - java
  - gui
  - blind-sql
  - kali-linux
  - pentest
mitre:
  tactic: TA0001
  technique: Initial Access
  id: T1190
difficulty: beginner
platforms:
  - linux
  - windows
  - macos
languages:
  - java
dependencies:
  - java-jdk-21-or-higher
created: "2010"
author: ron190
organization: ron190 (community)
license: GPL-2.0
status: active
github: https://github.com/ron190/jsql-injection
---

# jSQL Injection

## 1. Introduction & Overview

jSQL Injection is a lightweight, cross-platform Java application for automatic SQL database injection. It automates the discovery and exploitation of SQL injection vulnerabilities across multiple database engines, extracting database schemas, table data, and providing file read/write and reverse shell capabilities.

As one of the most mature GUI-based SQL injection tools, jSQL Injection ships with Kali Linux and is included in Parrot Security OS, ArchStrike, BlackArch Linux, and Pentest Box. It supports injection against MySQL, PostgreSQL, Microsoft SQL Server, Oracle, SQLite, and Access databases through a variety of injection strategies.

**Key characteristics:**

- GUI-based (Java Swing) — no command-line memorization required
- Automatic injection strategy identification
- Supports blind, error-based, UNION-based, and time-based injection
- Works with GET, POST, cookie, and header injection vectors
- Batch scanning for multiple targets
- Built-in hash cracker and encoding tools
- File read/write via database engine (when permissions allow)
- Reverse shell capability on certain configurations
- Requires Java 21 through Java 25

**MITRE ATT&CK Mapping:**

| Tactic | Technique | ID |
|--------|-----------|-----|
| Initial Access | Exploit Public-Facing Application | T1190 |
| Collection | Data from Information Repositories | T1213 |
| Credential Access | Unsecured Credentials | T1552 |

**When to use this tool:**

- During penetration tests to quickly identify and enumerate SQL injection points
- When a GUI interface speeds up analysis of injection vectors
- For batch scanning of multiple URLs for injection vulnerabilities
- To extract database schemas and data from confirmed injection points
- For file system access through database-specific functions

**When NOT to use this tool:**

- Against targets without explicit authorization
- When command-line tools like `sqlmap` are preferred for scripting/automation
- Against databases not in the supported list (though some generic strategies may work)

---

## 2. Installation & Setup

### Prerequisites

jSQL Injection requires Java 21 or newer (up to Java 25). Check your Java version:

```bash
java -version
```

If Java is not installed:

```bash
# Kali Linux / Debian / Ubuntu
sudo apt install default-jdk

# Or install Java 21+ from adoptium.net
# Download from https://jdk.java.net
```

### Installation Methods

**Method 1: Kali Linux (recommended for Kali users)**

```bash
sudo apt update && sudo apt install jsql
```

**Method 2: Direct download from GitHub**

```bash
# Download the latest release
wget https://github.com/ron190/jsql-injection/releases/download/v0.115/jsql-injection-v0.115.jar

# Run the application
java -jar jsql-injection-v0.115.jar
```

**Method 3: Build from source**

```bash
git clone https://github.com/ron190/jsql-injection.git
cd jsql-injection
mvn clean package
java -jar target/jsql-injection-*.jar
```

### Verify Installation

```bash
# Check Java version
java -version

# Check jSQL is accessible (Kali)
which jsql
jsql --help 2>/dev/null || echo "GUI application - launch with: java -jar jsql-injection-*.jar"
```

### Configuration

jSQL Injection stores configuration in its GUI preferences. Key settings accessible through the Preferences tab:

- **Connection timeout:** Adjust for slow targets (default: 10 seconds)
- **Thread count:** Parallel requests for faster injection (default: 4)
- **Tamper scripts:** Apply transformations to bypass WAF filters
- **Proxy settings:** Route through Burp Suite or other intercepting proxies

---

## 3. Basic Usage

### First Run

```bash
# Launch jSQL Injection
java -jar jsql-injection-v0.115.jar

# Or on Kali
jsql
```

**Explanation:** The GUI application launches with the Database tab active. Enter the vulnerable URL in the address bar and click the injection button (syringe icon) to begin automatic injection.

### Essential Commands

**Basic injection workflow:**

1. Enter the vulnerable URL in the address bar (e.g., `http://192.168.1.100/page.php?id=1`)
2. Click the **Inject** button (syringe icon) to start automatic identification
3. jSQL will test multiple injection strategies (UNION, blind, error-based, etc.)
4. Once a strategy is identified, the database tree populates on the left
5. Expand the tree to select databases → tables → columns
6. Right-click a table and select **Load** to extract rows

**Manual strategy selection:**

If automatic identification fails, manually select the engine and strategy:

1. In the address bar, append the injection marker: `http://192.168.1.100/page.php?id=1'__SQL_INJECT__`
2. Select the database engine from the dropdown (MySQL, MSSQL, PostgreSQL, Oracle, etc.)
3. Select the injection strategy (UNION, BLIND, ERROR, TIME, etc.)
4. Click Inject

**Using the address bar syntax:**

```
http://target.example.com/page.php?id=1' [INJECT HERE]
```

jSQL replaces the injection point marker with its payloads automatically.

### Flags Reference

jSQL Injection is a GUI application and does not use command-line flags. All operations are performed through the interface. The JAR file accepts no command-line arguments:

| Action | Method |
|--------|--------|
| Launch GUI | `java -jar jsql-injection-v0.115.jar` |
| Display help | No CLI help available |
| Version | Check Help → About in GUI |

---

## 4. Intermediate Usage

### Tab Overview

jSQL Injection organizes functionality into distinct tabs:

| Tab | Function | Description |
|-----|----------|-------------|
| **Database** | Injection | Main injection interface — extract schemas, tables, columns, rows |
| **Admin Page** | Search | Scan for admin login pages on the target |
| **Read File** | Injection | Read files from the filesystem via database functions |
| **Exploit** | Injection | Remote command execution, file upload, reverse shell |
| **Brute Force** | Processing | Hash cracking for extracted password hashes |
| **Encoding** | Text | Encode/decode text (Base64, hex, URL, etc.) |
| **Batch Scan** | Injection | Test multiple URLs for injection simultaneously |

### Tamper Scripts for WAF Bypass

jSQL supports tamper scripts to bypass Web Application Firewalls. Access via Preferences → Tamper:

| Tamper | Effect |
|--------|--------|
| `space2comment` | Replaces spaces with `/**/` |
| `randomcase` | Randomizes character casing |
| `encodeurl` | URL-encodes payload characters |
| `charencode` | Char-based encoding |
| `between` | Replaces `>` with `NOT BETWEEN 0 AND` |

### Proxy Configuration

Route jSQL traffic through Burp Suite for detailed request inspection:

1. Go to Preferences
2. Set proxy host: `127.0.0.1`
3. Set proxy port: `8080`
4. Enable proxy

All jSQL requests now pass through Burp, allowing you to inspect and modify payloads.

### Batch Scanning

The Batch Scan tab tests multiple URLs for injection:

1. Switch to the **Batch Scan** tab
2. Add target URLs to the left panel
3. Click **Start** to begin identification
4. Each target is tagged with its injection status (vulnerable/not vulnerable)
5. Export results for further analysis

**Example target list format:**

```
http://192.168.1.10/page.php?id=1
http://192.168.1.11/page.php?id=1
http://192.168.1.12/page.php?id=1
```

### Database Extraction Workflow

Complete workflow for extracting data:

1. **Inject** the target URL to identify the injection strategy
2. Expand the database tree on the left panel
3. Select the target database (e.g., `production_db`)
4. Expand to see all tables
5. Select columns of interest
6. Right-click the table → **Load** to extract all rows
7. Results appear in a tab on the right — sort, search, and export as needed

---

## 5. Advanced Usage

### File Read/Write Operations

When the database user has sufficient privileges, jSQL can read and write files on the target:

**Reading files:**

1. Switch to the **Read File** tab
2. Inject the target first (required for connection context)
3. Enter the file path on the left panel (e.g., `/etc/passwd` or `C:\Windows\System32\drivers\etc\hosts`)
4. Click **Read**
5. File contents appear in a tab on the right

**Supported file read operations by database:**

| Database | Method | Privilege Required |
|----------|--------|-------------------|
| MySQL | `LOAD_FILE()` | `FILE` privilege |
| PostgreSQL | `pg_read_file()` or `COPY` | Superuser or `pg_read_server_files` |
| MSSQL | `OPENROWSET` or ` xp_cmdshell type` | Sysadmin |
| Oracle | `UTL_FILE.FGETL` | `UTL_FILE` privilege |

**Writing files (exploit tab):**

1. Switch to the **Exploit** tab
2. Select the exploit type (write file, web shell, reverse shell)
3. Configure the payload
4. Click **Create** to execute

### Reverse Shell via Database

When file write is available, jSQL can deploy reverse shells:

1. Go to the **Exploit** tab
2. Select the appropriate exploit type
3. Configure the listener IP and port
4. Create the payload
5. Start a listener on the attacker machine

**Listener setup:**

```bash
# Netcat listener
nc -lvnp 4444

# Metasploit handler
msfconsole -q -x "use exploit/multi/handler; set PAYLOAD linux/x86/shell_reverse_tcp; set LPORT 4444; run"
```

### Hash Cracking

The Brute Force tab cracks hashes extracted from databases:

1. Switch to the **Brute Force** tab
2. Paste the hash to crack
3. Select the hash type (MD5, SHA1, SHA256, etc.)
4. Configure character range and length
5. Click **Start**

This is useful for cracking password hashes extracted from user tables.

### Custom SQL Engine Configuration

For non-standard injection patterns:

1. Enter the URL with a custom injection marker
2. Manually select the database engine from the SQL Engine dropdown
3. Override the strategy if automatic identification fails
4. Use the Parameter tab to fine-tune injection parameters

### Export and Reporting

Extracted data can be exported in multiple formats:

- **CSV:** Right-click the results tab → Export → CSV
- **XML:** Right-click → Export → XML
- **Clipboard:** Select cells → Copy

---

## 6. Real-World Scenarios

### Scenario 1: Penetration Test — E-Commerce Application

**Target:** Online store running PHP/MySQL with product search functionality.

**Steps:**

```bash
# 1. Identify injection point via manual testing
# URL: http://shop.target.example.com/products.php?category=electronics
# Single quote causes SQL error

# 2. Launch jSQL
java -jar jsql-injection-v0.115.jar

# 3. Enter URL in address bar
# http://shop.target.example.com/products.php?category=electronics'

# 4. Click Inject
# jSQL identifies MySQL with UNION-based injection

# 5. Extract database tree
# Production → users table → columns: username, password, email

# 6. Load user table
# 15,000 user records extracted including password hashes

# 7. Export to CSV for offline cracking
```

**Outcome:** Full user database extracted, password hashes cracked offline, credential reuse test against other services.

### Scenario 2: Internal Network Assessment — MSSQL Server

**Target:** Internal web application backed by Microsoft SQL Server.

**Steps:**

```bash
# 1. Application found during network scan
# http://intranet.company.local/app/detail.aspx?id=5

# 2. Manual test confirms blind SQL injection

# 3. jSQL injection with MSSQL engine selected
# Strategy: BLIND with WAITFOR DELAY

# 4. Extract sa password hash from sys.sql_logins

# 5. Use Brute Force tab to crack the hash
# Hash: 0x0200... (MSSQL format)
# Cracked: "Password123"

# 6. Use Exploit tab to execute commands
# exec xp_cmdshell 'whoami'
# Returns: nt service\mssqlserver

# 7. Write web shell for persistent access
```

**Outcome:** Database administrator access achieved, file system access via `xp_cmdshell`, persistent web shell deployed.

### Scenario 3: Bug Bounty — Government Web Portal

**Target:** Public-facing government portal with user registration.

**Steps:**

```bash
# 1. Registration form has "organization" parameter
# http://portal.gov.example.com/register?org=education

# 2. jSQL identifies error-based injection in org parameter

# 3. Extract information_schema
# databases: portal_db, information_schema, mysql

# 4. Enumerate portal_db
# tables: users, organizations, documents
# users columns: id, username, password_hash, role, email

# 5. Extract admin user
# username: admin@portal.gov.example.com
# role: superadmin

# 6. Hash extracted, cracked: "GovAdmin2024!"

# 7. Document for responsible disclosure
```

**Outcome:** Critical vulnerability documented with proof-of-concept extraction, reported through responsible disclosure program.

---

## 7. Detection & Defense

### Indicators of Compromise (IOCs)

**Network-level:**

- Unusual SQL keywords in HTTP parameters (`UNION SELECT`, `WAITFOR DELAY`, `ORDER BY`)
- High volume of 500/SQL error responses from a single source
- Requests with encoded payloads (URL-encoded, hex-encoded SQL)
- Rapid sequential requests to database-heavy pages

**Application-level:**

- MySQL/MSSQL error messages in HTTP responses
- Abnormal database query patterns in application logs
- Unusual queries against `information_schema` or `sys.objects`
- `LOAD_FILE()` or `UTL_FILE` calls from web application users

### Detection Rules

**ModSecurity WAF rule:**

```apache
# Block common SQL injection patterns
SecRule ARGS|QUERY_STRING|REQUEST_URI \
    "(?i)(union\s+(all\s+)?select|waitfor\s+delay|information_schema|load_file|into\s+(out|dump)file)" \
    "id:1010,phase:1,deny,status:403,log,msg:'SQL Injection Detected'"
```

**Log analysis (Splunk/ELK query):**

```
index=web sourcetype=access_combined
| regex _raw="(?i)(union|select|insert|update|delete|drop|waitfor|information_schema)"
| stats count by src_ip, uri
| where count > 10
```

### Hardening Against SQL Injection

1. **Use parameterized queries (prepared statements)** — never concatenate user input into SQL
2. **Input validation:** Whitelist acceptable characters and values
3. **Least privilege:** Database users should have minimal required permissions
4. **WAF deployment:** Deploy and tune rules for SQL injection patterns
5. **Error handling:** Never expose raw database errors to users
6. **Stored procedures:** Use parameterized stored procedures where possible
7. **ORM frameworks:** Use established ORMs (Hibernate, SQLAlchemy, Eloquent) with proper escaping
8. **Regular scanning:** Run automated SQL injection scans against your applications

### Remediation Code Examples

**Vulnerable (parameter concatenation):**

```php
// DANGEROUS - Never do this
$query = "SELECT * FROM users WHERE id = " . $_GET['id'];
$result = mysqli_query($conn, $query);
```

**Secure (parameterized query):**

```php
// SAFE - Use prepared statements
$stmt = $conn->prepare("SELECT * FROM users WHERE id = ?");
$stmt->bind_param("i", $_GET['id']);
$stmt->execute();
```

---

## 8. Troubleshooting

### Common Issues and Solutions

**Problem:** Application fails to start with Java version error

```bash
# Check Java version
java -version

# jSQL requires Java 21-25
# Install correct version:
sudo apt install openjdk-21-jdk

# Or specify Java path
export JAVA_HOME=/usr/lib/jvm/java-21-openjdk-amd64
java -jar jsql-injection-v0.115.jar
```

**Problem:** GUI doesn't display correctly (blank/garbled)

```bash
# Java Swing rendering issue — try different Look and Feel
# In Preferences, switch theme between Light/Dark

# Or force a specific rendering mode
java -Dsun.java2d.opengl=true -jar jsql-injection-v0.115.jar

# Alternative: set display environment
export DISPLAY=:0
```

**Problem:** Injection identification fails / "No strategy found"

```bash
# 1. Verify the URL is reachable
curl -v "http://target.example.com/page.php?id=1'"

# 2. Try manual strategy selection
# - Select engine (MySQL, MSSQL, etc.) from dropdown
# - Select strategy (UNION, BLIND, ERROR, TIME)

# 3. Ensure the injection marker is placed correctly
# URL should be: http://target.example.com/page.php?id=1'
# (The single quote closes the original query)

# 4. Increase timeout in Preferences if target is slow
```

**Problem:** Connection timeout / no response

```bash
# Test basic connectivity
ping target.example.com
curl -I http://target.example.com/page.php?id=1

# Check proxy settings in Preferences
# If using Burp, ensure it's running on 127.0.0.1:8080

# Increase timeout in Preferences → Connection
```

**Problem:** Extracted data is incomplete or garbled

```bash
# 1. Check encoding in Preferences
# 2. Try different strategies (TIME-based is slowest but most reliable)
# 3. Verify injection point is correct
# 4. Check for WAF interference (use tamper scripts)
# 5. Reduce thread count in Preferences to avoid detection
```

**Problem:** File read fails

```bash
# Check database user privileges:
# MySQL: SHOW GRANTS FOR 'current_user'
# MSSQL: SELECT * FROM fn_my_permissions(NULL, 'DATABASE')

# Common issues:
# - MySQL FILE privilege not granted
# - secure_file_priv is set (restricts LOAD_FILE paths)
# - File doesn't exist or user lacks read permission
```

### Performance Optimization

```bash
# For large databases:
# 1. Reduce thread count to 1-2 to avoid overwhelming the target
# 2. Use TIME-based strategy only when others fail
# 3. Extract specific tables, not entire databases
# 4. Use Batch Scan with minimal target lists
```

### Debugging

```bash
# Enable verbose logging
java -Djsql.debug=true -jar jsql-injection-v0.115.jar

# Monitor network traffic
tcpdump -i eth0 host target.example.com -w jsql_traffic.pcap

# Check Java logs
find ~/.jsql -name "*.log" 2>/dev/null
```

---

## Resources

- **GitHub Repository:** https://github.com/ron190/jsql-injection
- **Wiki Documentation:** https://github.com/ron190/jsql-injection/wiki
- **Latest Release:** https://github.com/ron190/jsql-injection/releases
- **Kali Linux Package:** `apt info jsql`
- **OWASP SQL Injection:** https://owasp.org/www-community/attacks/SQL_Injection
- **MITRE ATT&CK - Exploit Public-Facing Application:** https://attack.mitre.org/techniques/T1190/
- **SonarCloud Analysis:** https://sonarcloud.io/dashboard?id=ron190%3Ajsql-injection
- **Maven Reports:** https://ron190.github.io/jsql-injection/

---

**See Also:**
- [jboss-autopwn](../jboss-autopwn/jboss-autopwn.md) - JBoss exploitation tool
- [sqlninja](../sqlninja/sqlninja.md) - SQL injection tool specialized for MSSQL
