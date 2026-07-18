---
title: "Phase 06: Privilege Escalation"
description: "Kali Linux privilege escalation tools — automated enumeration and path discovery for escalating from low-privilege access to root/system on Linux, Windows, and Unix systems."
category: "privilege-escalation"
phase: "06"
mitre_tactic: "privilege_escalation"
mitre_tactic_id: "TA0004"
---

# Phase 06: Privilege Escalation

Privilege escalation is the act of exploiting a misconfiguration, vulnerable software, or design flaw to gain higher-level access on a system. After obtaining initial access (Phase 05), the attacker's next objective is to escalate to root on Linux or SYSTEM on Windows. This phase covers the tools, techniques, and checklists for identifying and exploiting local privilege escalation vectors.

## Why Privilege Escalation Matters

Initial access gives you a foothold. Root/SYSTEM gives you the system. Without privilege escalation:

- You cannot read `/etc/shadow` or `SAM` hashes
- You cannot install persistent backdoors
- You cannot pivot to other machines as a domain admin
- You cannot fully exfiltrate data
- You cannot complete a penetration test or CTF

Every penetration test requires escalation. Every CTF box requires it. The difference between a junior and senior pentester is the speed and thoroughness of privilege escalation enumeration.

## MITRE ATT&CK Mapping

| Tactic | ID | Description |
|---|---|---|
| **Privilege Escalation** | TA0004 | Adversaries attempt to gain higher-level permissions |

### Relevant Techniques

| Technique | ID | Tool Coverage |
|---|---|---|
| Exploitation for Privilege Escalation | T1068 | LinPEAS kernel exploit detection |
| Process Injection | T1055 | winPEAS token impersonation |
| Abuse Elevation Control Mechanism | T1548 | winPEAS AlwaysInstallElevated |
| Domain Groups | T1069.002 | winPEAS domain enumeration |
| Scheduled Task/Job | T1053 | Both tools check cron/Task Scheduler |
| Boot or Logon Autostart | T1547 | winPEAS autorun checks |
| Hijack Execution Flow | T1574 | winPEAS DLL hijacking, unquoted paths |
| Credentials from Password Stores | T1555 | winPEAS credential extraction |

## Tool Comparison Matrix

| Feature | LinPEAS | unix-privesc-check | winPEAS |
|---|---|---|---|
| **Target OS** | Linux/macOS/BSD | Linux/Solaris/HP-UX/FreeBSD | Windows |
| **Language** | Shell/Go | Shell | C#/PowerShell/Batch |
| **Speed** | 4-10 min | 5-30 sec | 2-5 min |
| **Dependencies** | sh or none (binary) | POSIX sh | .NET 4.5.2+ or PowerShell |
| **Check Depth** | 300+ checks | ~50 checks | 200+ checks |
| **Color Output** | Yes | No | Yes |
| **AV Bypass** | AES/Base64/Go binary | Minimal (small script) | Memory exec/Base64 |
| **Password Brute-force** | Yes | No | No (extracts stored) |
| **Network Recon** | Yes | No | Limited |
| **Container Checks** | Yes | Limited | N/A |
| **Kernel Exploits** | Yes | No | Yes |
| **Active Development** | Yes (2026) | Infrequent | Yes (2026) |

## When to Use Each Tool

### Use LinPEAS When

- Target is Linux, macOS, or BSD
- You want comprehensive enumeration
- You have time for a 4-10 minute scan
- You need password brute-force via `su`
- You need process monitoring for transient cron jobs
- You want API key/secret scanning
- You need network discovery from the compromised host

### Use unix-privesc-check When

- System is minimal (no Python, no Go, no curl)
- You need a fast scan (<30 seconds)
- Target is Solaris, HP-UX, or FreeBSD
- You are in an OSCP exam environment
- You want a simple, auditable check
- The system has restricted tooling

### Use winPEAS When

- Target is Windows (any version)
- You need to find service misconfigurations
- You need token impersonation checks
- You need stored credential extraction
- You need kernel exploit detection for Windows
- You need DLL hijacking path analysis

## Linux Privilege Escalation Checklist

Run through this checklist after obtaining a low-privilege shell on any Linux system.

### 1. System Enumeration

```bash
# Kernel and OS
uname -a
cat /etc/os-release
cat /proc/version

# Users and groups
id
cat /etc/passwd
cat /etc/group
groups
getent group sudo
getent group docker

# Sudo permissions
sudo -l

# Environment
env
echo $PATH
```

### 2. File Permission Checks

```bash
# SUID/SGID binaries
find / -perm -4000 -type f 2>/dev/null
find / -perm -g=s -type f 2>/dev/null

# World-writable files
find / -writable -type f 2>/dev/null
find / -writable -type d 2>/dev/null

# Writable /etc/passwd or /etc/shadow
ls -la /etc/passwd /etc/shadow

# NFS exports
cat /etc/exports

# Capabilities
getcap -r / 2>/dev/null
```

### 3. Cron and Scheduled Tasks

```bash
# System crontab
cat /etc/crontab
ls -la /etc/cron*
cat /etc/anacrontab

# User crontabs
crontab -l
ls -la /var/spool/cron/crontabs/

# Systemd timers
systemctl list-timers --all

# Writable cron scripts
find /etc/cron* -writable 2>/dev/null
```

### 4. Services and Processes

```bash
# Running processes
ps aux
ps -ef

# Service binaries
systemctl list-units --type=service

# Process binary permissions
ls -la $(which <service_binary>)
```

### 5. Kernel Exploits

```bash
# Check kernel version
uname -r
cat /proc/version

# Search for exploits
searchsploit linux kernel <version>
```

### 6. Credentials

```bash
# Search for passwords
grep -ri "password" /home/ /var/www/ /etc/ 2>/dev/null

# SSH keys
find / -name "id_rsa" -o -name "authorized_keys" 2>/dev/null

# Bash history
cat /home/*/.bash_history

# Configuration files
find / -name "*.conf" -o -name "*.config" -o -name "*.cfg" 2>/dev/null
```

## Windows Privilege Escalation Checklist

Run through this checklist after obtaining a low-privilege shell on any Windows system.

### 1. System Enumeration

```cmd
:: OS and architecture
systeminfo
wmic os get caption,version,buildnumber,osarchitecture

:: Hotfixes
wmic qfe list brief

:: Users and groups
net user
net localgroup administrators
net localgroup "Remote Desktop Users"

:: Environment
echo %PATH%
set
```

### 2. Service Misconfigurations

```cmd
:: Unquoted service paths
wmic service get name,displayname,pathname,startmode | findstr /i "auto" | findstr /i /v "c:\windows" | findstr /i /v """

:: Service binary permissions
sc query state= all

:: Writable service directories
accesschk /uwcq "Authenticated Users" * /accepteula
```

### 3. Registry Checks

```cmd
:: AlwaysInstallElevated
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated

:: Autorun programs
reg query HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
reg query HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Run

:: Autologon credentials
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v DefaultPassword
```

### 4. Stored Credentials

```cmd
:: WiFi passwords
netsh wlan show profiles
netsh wlan show profile name="NetworkName" key=clear

:: Credential Manager
cmdkey /list

:: Unattend.xml
dir /s /b C:\*unattend.xml
dir /s /b C:\*sysprep.xml

:: Vault files
dir /s /b C:\*vault*
```

### 5. Token Impersonation

```cmd
:: Check privileges
whoami /priv

:: If SeImpersonatePrivilege or SeAssignPrimaryTokenPrivilege:
:: Use Potato attacks (PrintSpoofer, GodPotato, JuicyPotato)
```

### 6. Kernel Exploits

```cmd
:: Check OS version
systeminfo | findstr /B /C:"OS Name" /C:"OS Version"

:: Compare with known exploits
:: BlueKeep, PrintNightmare, HIVENightmare, etc.
```

## Exploitation Decision Tree

```
After running LinPEAS/winPEAS:

1. Red/Yellow findings → Exploit immediately
   ├── SUID binary → GTFOBins
   ├── Writable cron → Append reverse shell
   ├── Sudo NOPASSWD → GTFOBins
   ├── Capabilities → abuse cap_setuid
   ├── Unquoted service → Place binary in path
   ├── AlwaysInstallElevated → Malicious MSI
   ├── Token impersonation → Potato attack
   └── Kernel exploit → Compile and run

2. Red findings → Investigate manually
   ├── Writable files → Check for exploitation path
   ├── Service binary perms → Check for DLL hijacking
   └── Stored credentials → Crack or reuse

3. Green findings → Low priority
   └── Only if all other paths exhausted

4. No findings → Manual deeper enumeration
   ├── Check /etc/crontab for custom scripts
   ├── Check /opt, /usr/local for custom software
   ├── Check for Docker socket
   ├── Check for NFS mounts
   └── Check for LXC/LXD group membership
```

## Common Exploitation Techniques

### Linux

| Vector | Tool to Check | Exploitation Reference |
|---|---|---|
| SUID binaries | LinPEAS, unix-privesc-check | GTFOBins |
| Capabilities | LinPEAS | GTFOBins capabilities |
| Sudo misconfig | LinPEAS, unix-privesc-check | GTFOBins sudo |
| Writable cron | LinPEAS, unix-privesc-check | Manual exploitation |
| NFS no_root_squash | LinPEAS, unix-privesc-check | Manual mount + SUID |
| Writable /etc/passwd | LinPEAS, unix-privesc-check | Add root user |
| Docker escape | LinPEAS | Docker run -v /:/mnt |
| Kernel exploit | LinPEAS | searchsploit |

### Windows

| Vector | Tool to Check | Exploitation Reference |
|---|---|---|
| Unquoted service path | winPEAS | Manual binary placement |
| AlwaysInstallElevated | winPEAS | Malicious MSI |
| DLL hijacking | winPEAS | Malicious DLL |
| Token impersonation | winPEAS | Potato attacks |
| Stored credentials | winPEAS | Reuse / crack |
| Kernel exploit | winPEAS | MSF modules |
| Autorun writable | winPEAS | Replace binary |
| Print Spooler | winPEAS | PrintNightmare |

## Practical Workflow

```
Phase 05: Initial Access
  ↓
Phase 06: Privilege Escalation
  ├── Step 1: Run LinPEAS (Linux) or winPEAS (Windows)
  ├── Step 2: Filter Red/Yellow findings
  ├── Step 3: Check GTFOBins/LOLBAS for exploitation
  ├── Step 4: Exploit highest-confidence vector
  ├── Step 5: Verify → id (Linux) / whoami (Windows)
  └── Step 6: If failed → next finding → repeat
  ↓
Phase 07: Post-Exploitation
```

## Resources

- **HackTricks Linux PE:** https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html
- **HackTricks Windows PE:** https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html
- **GTFOBins:** https://gtfobins.github.io/
- **LOLBAS:** https://lolbas-project.github.io/
- **PEASS-ng Repository:** https://github.com/peass-ng/PEASS-ng
- **HackTricks Training Course:** https://hacktricks-training.com/courses/lhe/
- **Linux Exploit Suggester:** https://github.com/The-Z-Labs/linux-exploit-suggester
- **Windows Exploit Suggester:** https://github.com/AonCyberLabs/Windows-Exploit-Suggester
- **LinPEAS Builder:** https://github.com/peass-ng/PEASS-ng/tree/master/linPEAS/builder
