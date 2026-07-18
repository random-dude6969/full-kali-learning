---
title: "smbclient - Samba SMB/CIFS Client"
description: "FTP-like client for accessing SMB/CIFS shared resources on Windows and Samba servers"
mitre_tactic: discovery
mitre_tactic_id: TA0007
mitre_subcategory: network_share_discovery
categories:
  - Network Enumeration
  - SMB Enumeration
  - File Transfer
  - Share Access
tags:
  - smb
  - cifs
  - samba
  - file_transfer
  - shares
  - windows
install_command: "sudo apt install smbclient"
version: "4.17+"
github_repo: "https://www.samba.org/samba/"
kali_tools_page: "https://www.kali.org/tools/smbclient/"
difficulty_level: beginner
requires_root: false
gui_available: false
related_tools:
  - smbmap
  - enum4linux
  - net
  - rpcclient
---

# smbclient - Samba SMB/CIFS Client

## 1. Introduction

### What is smbclient?

smbclient is a command-line client program that allows users to access shared resources on SMB/CIFS servers. Part of the Samba suite, it provides an FTP-like interface for interacting with Windows and Samba file shares, enabling users to list directories, transfer files, execute commands, and manage remote file systems.

### History

smbclient has been part of the Samba suite since its inception in the early 1990s. Developed as an open-source alternative to Microsoft's networking utilities, it enables interoperability between Linux/Unix systems and Windows networks. The tool has evolved through multiple SMB protocol versions while maintaining backward compatibility.

### When to Use smbclient

- **Share Enumeration**: List available shares on target systems
- **File Transfer**: Upload/download files to/from SMB shares
- **Share Access Testing**: Verify access permissions and credentials
- **Forensic Investigation**: Access remote file systems during incident response
- **Backup Operations**: Copy files from network shares
- **CTF Challenges**: Solve SMB-related challenges requiring share access

### Key Concepts

| Concept | Description |
|---------|-------------|
| **SMB** | Server Message Block protocol for network file sharing |
| **CIFS** | Common Internet File System, predecessor to SMB2/3 |
| **Share** | A directory or resource shared over the network |
| **Null Session** | Unauthenticated connection attempt |
| **Guest Access** | Access with guest/anonymous account |
| **Session** | Authenticated connection to an SMB server |

## 2. Installation

### System Requirements

- **OS**: Kali Linux, Parrot OS, or any Linux distribution
- **Dependencies**: Samba suite (libsmbclient, samba-common)
- **Network**: Access to target SMB ports (TCP 445, UDP 137-139)
- **Privileges**: No root required for basic operation

### Installation Methods

**Method 1: Package Manager (Recommended)**

```bash
sudo apt update
sudo apt install smbclient
```

**Method 2: Full Samba Suite**

```bash
sudo apt install samba
# Includes smbclient and other utilities
```

**Method 3: Kali Linux Default**

```bash
# Pre-installed on Kali Linux
which smbclient
smbclient --help
```

### Configuration

smbclient can use system-wide Samba configuration or command-line options:

```bash
# Check Samba configuration
cat /etc/samba/smb.conf

# Client configuration
cat /etc/samba/smbclient.conf
```

**Optional: Create Credentials File**

```bash
# ~/.smbcredentials
username=admin
password=secret
domain=WORKGROUP

# Use with: smbclient -A ~/.smbcredentials //host/share
```

### Verify Installation

```bash
smbclient --help
smbclient -V
```

## 3. Core Concepts

### How smbclient Works

smbclient establishes an SMB/CIFS connection to a target server and provides an interactive shell for file operations:

1. **Connection Setup**: Negotiates SMB protocol version with server
2. **Authentication**: Presents credentials (or attempts null session)
3. **Share Access**: Connects to specified share or lists available shares
4. **Interactive Shell**: Provides FTP-like command interface
5. **File Operations**: Executes requested file operations

### SMB Protocol Versions

```
┌─────────────────────────────────────────────────────┐
│                 SMB Protocol Evolution               │
├─────────────────────────────────────────────────────┤
│  SMB1 (CIFS)  │ Legacy, insecure, disabled by default │
│  SMB2         │ Improved performance, Vista+           │
│  SMB2.1       │ Large MTU support, Windows 7+         │
│  SMB3         │ Encryption, caching, Windows 8+       │
│  SMB3.1.1     │ Pre-authentication integrity, Win10+  │
└─────────────────────────────────────────────────────┘
```

### Connection States

| State | Description |
|-------|-------------|
| **Connected** | Active session with authentication |
| **Guest** | Anonymous/guest access |
| **Null** | Unauthenticated attempt |
| **Failed** | Authentication or connection failure |

### Share Types

| Type | Description |
|------|-------------|
| **Disk** | File system share |
| **Printer** | Shared printer |
| **Device** | Named pipe or device |
| **IPC** | Inter-process communication |

## 4. Command Reference

### Basic Syntax

```bash
smbclient [options] //server/share
```

### Connection Options

| Flag | Description | Example |
|------|-------------|---------|
| `-L` | List shares | `-L //192.168.1.100` |
| `-U <user>` | Username | `-U admin` |
| `-p <port>` | Port number | `-p 445` |
| `-I <ip>` | Target IP | `-I 192.168.1.100` |
| `-N` | No password | `-N` |
| `-D <dir>` | Starting directory | `-D /share` |
| `-m <max>` | Max protocol | `-m SMB3` |
| `-e` | Encrypt | `-e` |
| `-A <file>` | Credentials file | `-A creds.txt` |

### Interactive Commands

| Command | Description | Example |
|---------|-------------|---------|
| `ls` | List files | `ls` |
| `cd <dir>` | Change directory | `cd /docs` |
| `get <file>` | Download file | `get report.pdf` |
| `put <file>` | Upload file | `put data.csv` |
| `mget <mask>` | Download multiple | `mget *.txt` |
| `mput <mask>` | Upload multiple | `mput *.log` |
| `del <file>` | Delete file | `del old.txt` |
| `mkdir <dir>` | Create directory | `mkdir backup` |
| `rmdir <dir>` | Remove directory | `rmdir temp` |
| `rename <old> <new>` | Rename file | `rename a.txt b.txt` |
| `tar` | Archive operations | `tar c backup.tar /data` |
| `queue` | Show command queue | `queue` |
| `quit` | Exit smbclient | `quit` |

### Common Flag Combinations

```bash
# List shares with null session
smbclient -L //192.168.1.100 -N

# Connect with credentials
smbclient //192.168.1.100/share -U admin -p password

# Force SMB3 protocol
smbclient -m SMB3 //192.168.1.100/share -U admin

# Encrypted connection
smbclient -e //192.168.1.100/share -U admin
```

## 5. Usage Examples

### Basic Examples

**Example 1: List Shares**

```bash
smbclient -L //192.168.1.100 -N
```

**Example 2: Connect to Share**

```bash
smbclient //192.168.1.100/shared -U administrator
```

**Example 3: List Directory Contents**

```bash
smbclient //192.168.1.100/share -U admin -c "ls"
```

**Example 4: Download File**

```bash
smbclient //192.168.1.100/share -U admin -c "get file.txt"
```

### Intermediate Examples

**Example 5: Upload File**

```bash
smbclient //192.168.1.100/share -U admin -c "put localfile.txt"
```

**Example 6: Multiple File Download**

```bash
smbclient //192.168.1.100/share -U admin -c "mget *.log"
```

**Example 7: Create Directory**

```bash
smbclient //192.168.1.100/share -U admin -c "mkdir newdir"
```

**Example 8: Recursive Tar Archive**

```bash
smbclient //192.168.1.100/share -U admin -c "tar c backup.tar /docs"
```

### Advanced Examples

**Example 9: Null Session Enumeration**

```bash
smbclient -L //192.168.1.100 -N
# List shares without authentication
```

**Example 10: Force Specific SMB Version**

```bash
smbclient -m SMB3 //192.168.1.100/share -U admin
# Use SMB3 for encrypted connections
```

**Example 11: Execute Multiple Commands**

```bash
smbclient //192.168.1.100/share -U admin -c "ls;cd /logs;ls"
```

**Example 12: Download Entire Share**

```bash
smbclient //192.168.1.100/share -U admin -c "lcd /local/dir;recurse;prompt;mget *"
```

### Real-World Examples

**Example 13: Quick Share Discovery**

```bash
# List all shares
smbclient -L //192.168.1.100 -N 2>/dev/null | grep "Disk"

# Test access to each share
for share in $(smbclient -L //192.168.1.100 -N 2>/dev/null | awk '/Disk/ {print $1}'); do
  echo "=== Testing $share ==="
  smbclient //192.168.1.100/$share -N -c "ls" 2>/dev/null
done
```

**Example 14: Forensic Evidence Collection**

```bash
#!/bin/bash
TARGET=192.168.1.100
SHARE=forensics
USER=investigator
PASS=SecurePass123
OUTPUT="evidence_$(date +%Y%m%d)"

mkdir -p $OUTPUT

# Connect and download
smbclient //$TARGET/$SHARE -U $USER%$PASS -c "
  lcd $OUTPUT;
  cd /case_files;
  recurse;
  prompt;
  mget *
"
```

**Example 15: Automated Backup Script**

```bash
#!/bin/bash
# Backup share to local directory

TARGET=//192.168.1.100/backup
USER=admin
PASS=backuppass
LOCAL=/mnt/backup/$(date +%Y%m%d)

mkdir -p $LOCAL

smbclient $TARGET -U $USER%$PASS -c "
  lcd $LOCAL;
  recurse;
  prompt;
  mget *
"

echo "Backup completed: $LOCAL"
```

**Example 16: Access Testing Script**

```bash
#!/bin/bash
# Test access to multiple shares

for host in $(cat hosts.txt); do
  echo "=== $host ==="
  
  # Null session test
  smbclient -L //$host -N 2>/dev/null | grep -i "share" && \
    echo "  Null session: SUCCESS" || \
    echo "  Null session: DENIED"
  
  # Guest test
  smbclient -L //$host -U guest% 2>/dev/null | grep -i "share" && \
    echo "  Guest access: SUCCESS" || \
    echo "  Guest access: DENIED"
done
```

## 6. Advanced Techniques

### Scripting with smbclient

**Python Script for Share Enumeration**

```python
#!/usr/bin/env python3
import subprocess
import re

def list_shares(target):
    """List shares on target"""
    cmd = f"smbclient -L //{target} -N"
    result = subprocess.run(cmd, shell=True, capture_output=True, text=True)
    return result.stdout

def parse_shares(output):
    """Parse smbclient output"""
    shares = []
    for line in output.split('\n'):
        if 'Disk' in line or 'Print' in line:
            parts = line.split()
            if len(parts) >= 2:
                shares.append(parts[0])
    return shares

def test_access(target, share, username="", password=""):
    """Test access to share"""
    auth = f"-U {username}%{password}" if username else "-N"
    cmd = f"smbclient //{target}/{share} {auth} -c 'ls'"
    result = subprocess.run(cmd, shell=True, capture_output=True, text=True)
    return result.returncode == 0

# Usage
target = "192.168.1.100"
shares = list_shares(target)
for share in parse_shares(shares):
    if test_access(target, share):
        print(f"[+] {share}: Accessible")
    else:
        print(f"[-] {share}: Denied")
```

**Bash Script for Batch Download**

```bash
#!/bin/bash
# batch_download.sh

TARGET=$1
SHARE=$2
USER=$3
PASS=$4
OUTPUT_DIR="downloads_$(date +%Y%m%d)"

mkdir -p $OUTPUT_DIR

# List and download files
smbclient //$TARGET/$SHARE -U $USER%$PASS -c "
  lcd $OUTPUT_DIR;
  ls;
  quit
"

echo "Files downloaded to $OUTPUT_DIR"
```

### Chaining with Other Tools

**Reconnaissance Pipeline**

```bash
# Step 1: Discover SMB hosts
nbtscan 192.168.1.0/24 | awk '{print $1}' | grep -v "^IP" > hosts.txt

# Step 2: Enumerate shares
while read host; do
  echo "=== $host ==="
  smbclient -L //$host -N > "${host}_shares.txt"
done < hosts.txt

# Step 3: Access discovered shares
while read host; do
  grep "Disk" "${host}_shares.txt" | awk '{print $1}' | while read share; do
    echo "Testing $host/$share"
    smbclient //$host/$share -N -c "ls" > "${host}_${share}_access.txt" 2>&1
  done
done < hosts.txt
```

**Integration with enum4linux**

```bash
# Enumerate and access
enum4linux -S 192.168.1.100 | grep "Sharename" | awk '{print $2}' | while read share; do
  echo "=== $share ==="
  smbclient //192.168.1.100/$share -U admin -c "ls"
done
```

### Configuration Tuning

**Timeout Optimization**

```bash
# Reduce timeout for faster operations
smbclient -L //192.168.1.100 -N -t 10
```

**Protocol Selection**

```bash
# Force specific SMB version
smbclient -m SMB3 //192.168.1.100/share -U admin

# Maximum protocol
smbclient -m SMB3 //192.168.1.100/share -U admin
```

### Automation Patterns

**Cron Job for Backup**

```bash
# Add to crontab
0 2 * * * smbclient //server/backup -U user%pass -c "lcd /backup;recurse;prompt;mget *" >> /var/log/smb_backup.log
```

**Integration with Ansible**

```yaml
- name: Download files from SMB share
  hosts: localhost
  tasks:
    - name: Download configuration files
      command: >
        smbclient //192.168.1.100/config -U admin%pass
        -c "lcd /local/config;get config.xml"
```

## 7. Detection & Defense

### Detection Signatures

**Network-Based Detection**

```
# Snort rule for smbclient activity
alert tcp any any -> $HOME_NET 445 (
  msg:"SMB smbclient connection";
  content:"|FF|SMB"; depth:4;
  content:"|73 00 00 00|"; offset:36; depth:4;
  sid:1000004; rev:1;
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
   full_audit:success = connect open read write
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

**2. Implement Share Permissions**

```bash
# Windows NTFS permissions
icacls C:\share /grant:r "DOMAIN\Users:(RX)"

# Samba ACL
[share]
   read list = @users
   write list = @admins
```

**3. Disable Null Sessions**

```bash
# Windows Registry
reg add HKLM\SYSTEM\CurrentControlSet\Services\LanManServer\Parameters /v RestrictNullSessAccess /t REG_DWORD /d 1 /f

# Samba Configuration
[global]
   restrict anonymous = 2
```

**4. Enable SMB Signing**

```bash
# Samba Configuration
[global]
   client signing = mandatory
   server signing = mandatory
```

**5. Monitor Share Access**

```bash
# Real-time monitoring
tail -f /var/log/samba/log.* | grep -i "session setup"
```

### IDS/IPS Signatures

```bash
# Suricata rule
alert tcp any any -> any 445 (
  msg:"SMB share enumeration detected";
  flow:to_server,established;
  content:"SMB";
  content:"|FF|SMB";
  sid:2000004; rev:1;
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

**Issue: "Connection to 192.168.1.100 failed"**

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
smbclient //192.168.1.100/share -U 'admin:P@ssw0rd'
```

**Issue: "NT_STATUS_ACCESS_DENIED"**

```bash
# Cause: Insufficient permissions
# Solution: Try different credentials or check share permissions
smbclient -L //192.168.1.100 -N
```

**Issue: "Protocol negotiation failed"**

```bash
# Cause: SMB version mismatch
# Solution: Force SMB version
smbclient -m SMB3 //192.168.1.100/share -U admin
```

**Issue: No shares listed**

```bash
# Cause: Null sessions restricted or no shares
# Solution: Try authenticated enumeration
smbclient -L //192.168.1.100 -U admin -p password
```

### Frequently Asked Questions

**Q: Is smbclient legal to use?**
A: Only on systems you own or have explicit written authorization to test.

**Q: Does smbclient support SMB2/3?**
A: Yes, use `-m SMB3` flag to force SMB3.

**Q: Can smbclient access Active Directory shares?**
A: Yes, with valid domain credentials.

**Q: How does smbclient differ from smbmap?**
A: smbclient provides interactive file operations; smbmap focuses on enumeration.

**Q: Can smbclient transfer large files?**
A: Yes, but performance may vary. Use tar for bulk transfers.

### Performance Optimization

```bash
# Use tar for bulk transfers
smbclient //server/share -U admin -c "tar c /local/backup.tar /remote/dir"

# Parallel downloads
for file in $(smbclient //server/share -U admin -c "ls" | awk '{print $1}'); do
  smbclient //server/share -U admin -c "get $file" &
done
wait
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

- [Samba Documentation](https://www.samba.org/samba/docs/)
- [smbclient Man Page](https://www.samba.org/samba/docs/man/manpages-3/smbclient.1.html)
- [Kali Linux Tools](https://www.kali.org/tools/smbclient/)

### Related Tools

- **smbmap**: SMB share enumeration and file operations
- **enum4linux**: Comprehensive SMB enumeration
- **net**: Windows/Samba network command
- **rpcclient**: Low-level RPC enumeration

### Further Reading

- [SMB Protocol Specification (MS-SMB2)](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-smb2)
- [CIFS Protocol Reference](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-cifs)
- [Samba Security Configuration](https://www.samba.org/samba/docs/man/smb.conf/)
