---
title: "smbmap - SMB Share Enumeration and File Listing"
description: "Enumerate SMB share drives, list directories, and download/upload files with recursive scanning capabilities"
mitre_tactic: discovery
mitre_tactic_id: TA0007
mitre_subcategory: network_share_discovery
categories:
  - Network Enumeration
  - SMB Enumeration
  - Share Enumeration
  - File Operations
tags:
  - smb
  - shares
  - enumeration
  - file_transfer
  - recursive
  - samba
install_command: "sudo apt install smbmap"
version: "1.10+"
github_repo: "https://github.com/ShawnDEvans/smbmap"
kali_tools_page: "https://www.kali.org/tools/smbmap/"
difficulty_level: beginner
requires_root: false
gui_available: false
related_tools:
  - smbclient
  - enum4linux
  - rpcclient
---

# smbmap - SMB Share Enumeration and File Listing

## 1. Introduction

### What is smbmap?

smbmap is a Python-based tool for enumerating SMB share drives and listing directory contents across Windows and Samba networks. It provides capabilities for discovering accessible shares, listing files recursively, downloading/uploading files, and executing commands on remote systems—all with minimal authentication requirements.

### History

Developed by Shawn Evans, smbmap emerged as a modern alternative to older SMB enumeration tools, focusing on share enumeration and file operations. The tool gained popularity in penetration testing communities for its ability to quickly identify accessible shares and extract sensitive files. It continues active development on GitHub with regular updates.

### When to Use smbmap

- **Share Discovery**: Identify accessible SMB shares on the network
- **File Enumeration**: List contents of shares recursively
- **Data Exfiltration Testing**: Test file download/upload capabilities
- **Permission Assessment**: Verify access controls on shares
- **Forensic Collection**: Gather evidence from remote file systems
- **CTF Challenges**: Solve SMB-related file enumeration challenges

### Key Concepts

| Concept | Description |
|---------|-------------|
| **Share Enumeration** | Discovering available SMB shares on a target |
| **Recursive Listing** | Listing all files and directories hierarchically |
| **READ/WRITE Access** | Permission levels for share operations |
| **Null Session** | Unauthenticated connection to enumerate shares |
| **RID Cycling** | Brute-force user enumeration via RIDs |
| **Command Execution** | Running commands via SMB named pipes |

## 2. Installation

### System Requirements

- **OS**: Kali Linux, Parrot OS, or any Linux distribution with Python
- **Python**: Python 3.6+
- **Dependencies**: Python SMB library (pysmb or impacket)
- **Network**: Access to target SMB ports (TCP 445, UDP 137-139)
- **Privileges**: No root required for basic operation

### Installation Methods

**Method 1: Package Manager (Recommended)**

```bash
sudo apt update
sudo apt install smbmap
```

**Method 2: From Source**

```bash
git clone https://github.com/ShawnDEvans/smbmap.git
cd smbmap
pip install -r requirements.txt
chmod +x smbmap.py
sudo cp smbmap.py /usr/local/bin/smbmap
```

**Method 3: pip Installation**

```bash
pip install smbmap
```

**Method 4: Kali Linux Default**

```bash
# Pre-installed on Kali Linux
which smbmap
smbmap --help
```

### Configuration

smbmap requires no configuration file. Dependencies must be installed:

```bash
# Check Python version
python3 --version

# Install dependencies if needed
pip3 install pysmb impacket
```

### Verify Installation

```bash
smbmap --help
smbmap --version
```

## 3. Core Concepts

### How smbmap Works

smbmap connects to SMB servers, enumerates available shares, and performs file operations based on user credentials:

1. **Target Resolution**: Resolves target hostname or IP
2. **Authentication**: Attempts connection with provided credentials
3. **Share Enumeration**: Lists available shares and permissions
4. **File Listing**: Recursively lists directory contents
5. **Operations**: Performs requested file operations (download/upload/execute)

### SMB Permission Levels

```
┌─────────────────────────────────────────────────────┐
│              SMB Permission Levels                   │
├─────────────────────────────────────────────────────┤
│  NO ACCESS  │ Cannot connect or list shares          │
│  READ       │ Can view/download files                │
│  WRITE      │ Can upload/modify/delete files         │
│  ADMIN      │ Full control, including permissions    │
└─────────────────────────────────────────────────────┘
```

### Share Discovery Flow

```
┌─────────────────────────────────────────────────────┐
│              smbmap Enumeration Flow                 │
├─────────────────────────────────────────────────────┤
│  Target ──> Authentication ──> Share List            │
│                                    │                 │
│                                    ▼                 │
│                            Permission Check          │
│                                    │                 │
│                    ┌───────────────┼───────────────┐ │
│                    ▼               ▼               ▼ │
│                  READ            WRITE           DENY│
│                    │               │               │ │
│                    ▼               ▼               │ │
│               File List      File Operations      │ │
└─────────────────────────────────────────────────────┘
```

### Output Formats

| Format | Description | Use Case |
|--------|-------------|----------|
| **Text** | Human-readable tables | Manual review |
| **CSV** | Comma-separated values | Spreadsheet import |
| **JSON** | Machine-readable | Automation |

## 4. Command Reference

### Basic Syntax

```bash
smbmap [options]
```

### Target Options

| Flag | Description | Example |
|------|-------------|---------|
| `-H <host>` | Target host | `-H 192.168.1.100` |
| `-u <user>` | Username | `-u admin` |
| `-p <pass>` | Password | `-p password` |
| `-P <port>` | Port number | `-P 445` |
| `-d <domain>` | Domain name | `-d WORKGROUP` |

### Enumeration Options

| Flag | Description | Example |
|------|-------------|---------|
| `-L` | List shares | `-L -H 192.168.1.100` |
| `-s <share>` | Specify share | `-s sharename` |
| `-R` | Recursive listing | `-R` |
| `-r <path>` | Specify path | `-r /docs` |

### Operation Options

| Flag | Description | Example |
|------|-------------|---------|
| `--download <path>` | Download file | `--download /file.txt` |
| `--upload <file>` | Upload file | `--upload local.txt` |
| `-x <cmd>` | Execute command | `-x "whoami"` |
| `--delete <path>` | Delete file | `--delete /file.txt` |

### Output Options

| Flag | Description | Example |
|------|-------------|---------|
| `--csv` | CSV output | `--csv` |
| `--json` | JSON output | `--json` |
| `--exclude <share>` | Exclude shares | `--exclude print$` |

### Common Flag Combinations

```bash
# List all shares
smbmap -H 192.168.1.100

# List shares with credentials
smbmap -H 192.168.1.100 -u admin -p password

# Recursive listing of share
smbmap -H 192.168.1.100 -s share -R

# Download file
smbmap -H 192.168.1.100 -u admin -p pass --download /file.txt

# Execute command
smbmap -H 192.168.1.100 -u admin -p pass -x "whoami"
```

## 5. Usage Examples

### Basic Examples

**Example 1: List Shares**

```bash
smbmap -H 192.168.1.100
```

**Example 2: List Shares with Credentials**

```bash
smbmap -H 192.168.1.100 -u administrator -p P@ssw0rd
```

**Example 3: List Share Contents**

```bash
smbmap -H 192.168.1.100 -s shared -r
```

**Example 4: Recursive Directory Listing**

```bash
smbmap -H 192.168.1.100 -s share -R
```

### Intermediate Examples

**Example 5: Download File**

```bash
smbmap -H 192.168.1.100 -u admin -p pass --download /docs/report.pdf
```

**Example 6: Upload File**

```bash
smbmap -H 192.168.1.100 -u admin -p pass --upload localfile.txt -r /uploads
```

**Example 7: Execute Command**

```bash
smbmap -H 192.168.1.100 -u admin -p pass -x "whoami"
```

**Example 8: CSV Output**

```bash
smbmap -H 192.168.1.100 -u admin -p pass --csv > shares.csv
```

### Advanced Examples

**Example 9: NULL Session Enumeration**

```bash
smbmap -H 192.168.1.100
# Without credentials, attempts NULL session
```

**Example 10: Domain Enumeration**

```bash
smbmap -H 192.168.1.100 -d CORP -u john -p password
```

**Example 11: Recursive with Exclusions**

```bash
smbmap -H 192.168.1.100 -s share -R --exclude "print$"
```

**Example 12: JSON Output for Parsing**

```bash
smbmap -H 192.168.1.100 -u admin -p pass --json > results.json
```

### Real-World Examples

**Example 13: Quick Share Discovery**

```bash
# Scan multiple hosts
for ip in $(seq 1 100); do
  echo "=== 192.168.1.$ip ==="
  smbmap -H 192.168.1.$ip 2>/dev/null | grep "READ"
done
```

**Example 14: Find Sensitive Files**

```bash
# Search for password files
smbmap -H 192.168.1.100 -s share -R | grep -i "password\|secret\|config"
```

**Example 15: Data Exfiltration Test**

```bash
#!/bin/bash
TARGET=192.168.1.100
SHARE=confidential
USER=admin
PASS=pass123

# List all files
smbmap -H $TARGET -s $SHARE -u $USER -p $PASS -R --csv > file_list.csv

# Download all files
smbmap -H $TARGET -s $SHARE -u $USER -p $PASS --download /
```

**Example 16: Compliance Audit Script**

```bash
#!/bin/bash
for host in $(cat targets.txt); do
  echo "=== $host ==="
  smbmap -H $host -u admin -p pass --csv > "${host}_shares.csv"
  
  # Check for open shares
  OPEN=$(smbmap -H $host 2>/dev/null | grep "READ" | wc -l)
  echo "$host has $OPEN readable shares"
done
```

## 6. Advanced Techniques

### Scripting with smbmap

**Python Script for Automated Enumeration**

```python
#!/usr/bin/env python3
import subprocess
import json
from pathlib import Path

def smbmap_scan(target, username="", password=""):
    """Scan target with smbmap"""
    cmd = ["smbmap", "-H", target, "--json"]
    
    if username:
        cmd.extend(["-u", username, "-p", password])
    
    result = subprocess.run(cmd, capture_output=True, text=True)
    return json.loads(result.stdout) if result.stdout else {}

def find_sensitive_files(json_file):
    """Find potentially sensitive files"""
    sensitive_patterns = ["password", "secret", "config", "backup", "key"]
    
    with open(json_file) as f:
        data = json.load(f)
    
    findings = []
    for share in data.get("shares", []):
        for file in share.get("files", []):
            for pattern in sensitive_patterns:
                if pattern.lower() in file.lower():
                    findings.append({"share": share["name"], "file": file})
    
    return findings

# Usage
target = "192.168.1.100"
results = smbmap_scan(target, "admin", "password")
print(json.dumps(results, indent=2))
```

**Bash Script for Batch Processing**

```bash
#!/bin/bash
# smbmap_batch.sh

TARGETS=$1
USER=$2
PASS=$3
OUTPUT_DIR="smbmap_$(date +%Y%m%d)"

mkdir -p $OUTPUT_DIR

while read target; do
  echo "[*] Enumerating $target"
  
  # List shares
  smbmap -H $target -u $USER -p $PASS > "$OUTPUT_DIR/${target}_shares.txt"
  
  # Recursive listing of readable shares
  smbmap -H $target -u $USER -p $PASS -R --csv > "$OUTPUT_DIR/${target}_files.csv"
  
  echo "[+] $target complete"
done < $TARGETS
```

### Chaining with Other Tools

**Reconnaissance Pipeline**

```bash
# Step 1: Discover SMB hosts
nmap -p 445 --open 192.168.1.0/24 | grep "report" | awk '{print $NF}' > smb_hosts.txt

# Step 2: Enumerate with smbmap
while read host; do
  echo "=== $host ==="
  smbmap -H $host > "${host}_smbmap.txt"
done < smb_hosts.txt

# Step 3: Access readable shares
while read host; do
  READABLE=$(smbmap -H $host 2>/dev/null | grep "READ" | awk '{print $1}')
  for share in $READABLE; do
    echo "Downloading from $host/$share"
    smbmap -H $host -s $share -R > "${host}_${share}_list.txt"
  done
done < smb_hosts.txt
```

**Integration with enum4linux**

```bash
# Enumerate and access
enum4linux -S 192.168.1.100 | grep "Sharename" | awk '{print $2}' | while read share; do
  echo "=== $share ==="
  smbmap -H 192.168.1.100 -s $share -R
done
```

### Configuration Tuning

**Timeout Optimization**

```bash
# Faster scanning
smbmap -H 192.168.1.100 2>/dev/null
```

**Protocol Selection**

```bash
# Force SMB2/3
smbmap -H 192.168.1.100 --no-smb2 2>/dev/null
```

### Automation Patterns

**Cron Job for Share Monitoring**

```bash
# Add to crontab
0 */12 * * * smbmap -H 192.168.1.100 --csv >> /var/log/smbmap_monitor.csv
```

**Integration with SIEM**

```bash
# Output JSON for SIEM ingestion
smbmap -H 192.168.1.100 --json | jq -c '.shares[] | {host: .host, share: .name}'
```

## 7. Detection & Defense

### Detection Signatures

**Network-Based Detection**

```
# Snort rule for smbmap activity
alert tcp any any -> $HOME_NET 445 (
  msg:"NETBIOS SMB smbmap enumeration";
  content:"|FF|SMB"; depth:4;
  content:"|73 00 00 00|"; offset:36; depth:4;
  sid:1000005; rev:1;
)
```

**Windows Event Log Analysis**

```powershell
# Event ID 5140 - Share Access
Get-WinEvent -FilterHashtable @{
    LogName='Security'
    Id=5140
} | Select TimeCreated, Message
```

**SMB Audit Logging**

```bash
# Enable SMB audit logging on Linux
# /etc/samba/smb.conf
[global]
   vfs objects = full_audit
   full_audit:prefix = %u|%I|%S
   full_audit:success = connect opendir read
   full_audit:failure = none
   full_audit:facility = local5
   full_audit:priority = notice
```

### Defense Strategies

**1. Restrict Share Access**

```bash
# Samba Configuration
[share]
   valid users = @trusted_group
   hosts allow = 192.168.1.0/24
   hosts deny = ALL
```

**2. Implement Least Privilege**

```bash
# Windows NTFS permissions
icacls C:\share /grant:r "DOMAIN\Users:(RX)"
icacls C:\share /grant:r "DOMAIN\Admins:(F)"
```

**3. Disable Unnecessary Shares**

```bash
# Windows Registry
reg add "HKLM\SYSTEM\CurrentControlSet\Services\LanManServer\Parameters" /v AutoShareServer /t REG_DWORD /d 0 /f
```

**4. Monitor Share Access**

```bash
# Real-time monitoring
tail -f /var/log/samba/log.* | grep -i "connect from"
```

**5. Implement Share Auditing**

```bash
# Windows auditing
auditpol /set /subcategory:"File Share" /success:enable /failure:enable
```

### IDS/IPS Signatures

```bash
# Suricata rule
alert tcp any any -> any 445 (
  msg:"SMB share enumeration detected";
  flow:to_server,established;
  content:"SMB";
  content:"|FF|SMB";
  sid:2000005; rev:1;
)
```

### Log Analysis

```bash
# Parse Samba logs for access attempts
grep -E "(connect from|session setup)" /var/log/samba/log.* | \
  awk '{print $1, $4}' | sort | uniq -c | sort -rn
```

## 8. Troubleshooting & FAQ

### Common Issues

**Issue: "Connection refused"**

```bash
# Cause: SMB service not running or blocked
# Solution: Verify SMB service is active
nmap -p 445 192.168.1.100
systemctl status smbd
```

**Issue: "NT_STATUS_LOGON_FAILURE"**

```bash
# Cause: Invalid credentials
# Solution: Verify username and password
smbmap -H 192.168.1.100 -u 'admin' -p 'P@ssw0rd'
```

**Issue: "NT_STATUS_ACCESS_DENIED"**

```bash
# Cause: Insufficient permissions
# Solution: Try NULL session or different credentials
smbmap -H 192.168.1.100
```

**Issue: No shares listed**

```bash
# Cause: NULL sessions restricted
# Solution: Provide valid credentials
smbmap -H 192.168.1.100 -u admin -p password
```

**Issue: Recursive listing fails**

```bash
# Cause: Insufficient permissions or large share
# Solution: Specify path or limit depth
smbmap -H 192.168.1.100 -s share -r /specific/path
```

### Frequently Asked Questions

**Q: Is smbmap legal to use?**
A: Only on systems you own or have explicit written authorization to test.

**Q: Does smbmap support SMB2/3?**
A: Yes, smbmap supports modern SMB protocols.

**Q: Can smbmap execute commands?**
A: Yes, use `-x` flag, requires admin privileges.

**Q: How does smbmap differ from smbclient?**
A: smbmap focuses on enumeration and file operations; smbclient provides interactive shell.

**Q: Can smbmap bypass authentication?**
A: smbmap attempts NULL sessions but cannot bypass proper authentication.

### Performance Optimization

```bash
# Parallel enumeration
parallel smbmap -H {} ::: 192.168.1.{1..254}

# Timeout control
timeout 30 smbmap -H 192.168.1.100
```

### Exit Codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | Error in arguments |
| 2 | Connection failed |
| 3 | Authentication failed |
| 4 | File operation failed |

## Resources

### Official Documentation

- [smbmap GitHub](https://github.com/ShawnDEvans/smbmap)
- [Kali Linux Tools](https://www.kali.org/tools/smbmap/)
- [Samba Documentation](https://www.samba.org/samba/docs/)

### Related Tools

- **smbclient**: Samba SMB client for interactive operations
- **enum4linux**: Comprehensive SMB enumeration
- **rpcclient**: Low-level RPC enumeration
- **smbget**: wget-like SMB client

### Further Reading

- [SMB Protocol Specification (MS-SMB2)](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-smb2)
- [Samba Security Configuration](https://www.samba.org/samba/docs/man/smb.conf/)
- [Python SMB Library Documentation](https://pypi.org/project/pysmb/)
