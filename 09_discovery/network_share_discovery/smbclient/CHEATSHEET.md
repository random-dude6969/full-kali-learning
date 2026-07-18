---
title: "smbclient Cheatsheet"
description: "Quick reference for smbclient SMB/CIFS client commands"
tool: smbclient
category: SMB Enumeration
---

# smbclient Cheatsheet

## Quick Start

```bash
# List shares (null session)
smbclient -L //192.168.1.100 -N

# Connect to share
smbclient //192.168.1.100/share -U admin

# List files
smbclient //192.168.1.100/share -U admin -c "ls"

# Download file
smbclient //192.168.1.100/share -U admin -c "get file.txt"
```

## Common Flags

| Flag | Description | Example |
|------|-------------|---------|
| `-L` | List shares | `-L //192.168.1.100` |
| `-U <user>` | Username | `-U admin` |
| `-p <port>` | Port | `-p 445` |
| `-I <ip>` | Target IP | `-I 192.168.1.100` |
| `-N` | No password | `-N` |
| `-D <dir>` | Start directory | `-D /share` |
| `-m <max>` | Max protocol | `-m SMB3` |
| `-e` | Encrypt | `-e` |
| `-A <file>` | Credentials file | `-A creds.txt` |
| `-c <cmd>` | Execute command | `-c "ls"` |

## Interactive Commands

| Command | Description | Example |
|---------|-------------|---------|
| `ls` | List files | `ls` |
| `cd <dir>` | Change directory | `cd /docs` |
| `get <file>` | Download | `get file.txt` |
| `put <file>` | Upload | `put file.txt` |
| `mget <mask>` | Multiple download | `mget *.txt` |
| `mput <mask>` | Multiple upload | `mput *.log` |
| `del <file>` | Delete | `del file.txt` |
| `mkdir <dir>` | Create directory | `mkdir backup` |
| `rmdir <dir>` | Remove directory | `rmdir temp` |
| `tar` | Archive | `tar c backup.tar /data` |
| `quit` | Exit | `quit` |

## Examples

### Basic Operations

```bash
# List shares
smbclient -L //192.168.1.100 -N

# Connect with password
smbclient //192.168.1.100/share -U admin%password

# List files
smbclient //192.168.1.100/share -U admin -c "ls"

# Download file
smbclient //192.168.1.100/share -U admin -c "get file.txt"

# Upload file
smbclient //192.168.1.100/share -U admin -c "put local.txt"
```

### Advanced Usage

```bash
# Force SMB3
smbclient -m SMB3 //192.168.1.100/share -U admin

# Multiple commands
smbclient //192.168.1.100/share -U admin -c "ls;cd /logs;ls"

# Download all files
smbclient //192.168.1.100/share -U admin -c "lcd /local;recurse;prompt;mget *"

# Create directory
smbclient //192.168.1.100/share -U admin -c "mkdir newdir"

# Tar archive
smbclient //192.168.1.100/share -U admin -c "tar c backup.tar /docs"
```

### Null Session Access

```bash
# List shares
smbclient -L //192.168.1.100 -N

# Test share access
smbclient //192.168.1.100/share -N -c "ls"

# Guest access
smbclient -L //192.168.1.100 -U guest%
```

## Chaining

```bash
# Enumerate shares, then access
enum4linux -S 192.168.1.100 | grep "Sharename" | awk '{print $2}' | while read share; do
  echo "=== $share ==="
  smbclient //192.168.1.100/$share -U admin -c "ls"
done

# Download all files from share
smbclient //192.168.1.100/share -U admin -c "
  lcd /local/dir;
  recurse;
  prompt;
  mget *
"

# Test access from nbtscan results
nbtscan 192.168.1.0/24 | awk '{print $1}' | grep -v "^IP" | while read ip; do
  smbclient -L //$ip -N 2>/dev/null | grep "Disk" && echo "$ip has shares"
done
```

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Connection refused | Check SMB: `nmap -p 445 TARGET` |
| Logon failure | Verify creds: `-U user%pass` |
| Access denied | Try null session: `-N` |
| Protocol failed | Force SMB: `-m SMB3` |
| No shares listed | Try authenticated: `-U user -p pass` |
| Timeout | Reduce: `-t 10` |
