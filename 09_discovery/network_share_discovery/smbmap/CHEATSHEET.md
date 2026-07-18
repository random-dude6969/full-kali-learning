---
title: "smbmap Cheatsheet"
description: "Quick reference for smbmap SMB share enumeration and file operations"
tool: smbmap
category: SMB Enumeration
---

# smbmap Cheatsheet

## Quick Start

```bash
# List shares
smbmap -H 192.168.1.100

# List shares with credentials
smbmap -H 192.168.1.100 -u admin -p password

# Recursive listing
smbmap -H 192.168.1.100 -s share -R

# Download file
smbmap -H 192.168.1.100 -u admin -p pass --download /file.txt
```

## Common Flags

| Flag | Description | Example |
|------|-------------|---------|
| `-H <host>` | Target host | `-H 192.168.1.100` |
| `-u <user>` | Username | `-u admin` |
| `-p <pass>` | Password | `-p password` |
| `-P <port>` | Port | `-P 445` |
| `-d <domain>` | Domain | `-d WORKGROUP` |
| `-L` | List shares | `-L -H TARGET` |
| `-s <share>` | Specify share | `-s sharename` |
| `-R` | Recursive | `-R` |
| `-r <path>` | Path | `-r /docs` |
| `-x <cmd>` | Execute command | `-x "whoami"` |
| `--download <path>` | Download | `--download /file.txt` |
| `--upload <file>` | Upload | `--upload local.txt` |
| `--delete <path>` | Delete | `--delete /file.txt` |
| `--csv` | CSV output | `--csv` |
| `--json` | JSON output | `--json` |

## Examples

### Basic Enumeration

```bash
# List shares (null session)
smbmap -H 192.168.1.100

# List shares with credentials
smbmap -H 192.168.1.100 -u admin -p password

# List share contents
smbmap -H 192.168.1.100 -s share -r

# Recursive listing
smbmap -H 192.168.1.100 -s share -R
```

### File Operations

```bash
# Download file
smbmap -H 192.168.1.100 -u admin -p pass --download /docs/file.txt

# Upload file
smbmap -H 192.168.1.100 -u admin -p pass --upload local.txt -r /uploads

# Delete file
smbmap -H 192.168.1.100 -u admin -p pass --delete /temp/old.txt

# Execute command (requires admin)
smbmap -H 192.168.1.100 -u admin -p pass -x "whoami"
```

### Advanced Usage

```bash
# Domain authentication
smbmap -H 192.168.1.100 -d CORP -u john -p pass123

# CSV output
smbmap -H 192.168.1.100 -u admin -p pass --csv > shares.csv

# JSON output
smbmap -H 192.168.1.100 -u admin -p pass --json > results.json

# Exclude shares
smbmap -H 192.168.1.100 -s share -R --exclude "print$"

# Recursive download
smbmap -H 192.168.1.100 -s share -u admin -p pass --download /
```

## Chaining

```bash
# Enumerate shares, then list contents
smbmap -H 192.168.1.100 | grep "READ" | awk '{print $1}' | while read share; do
  echo "=== $share ==="
  smbmap -H 192.168.1.100 -s $share -R
done

# Find sensitive files
smbmap -H 192.168.1.100 -s share -R | grep -i "password\|secret\|config"

# Combine with enum4linux
enum4linux -S 192.168.1.100 | grep "Sharename" | awk '{print $2}' | while read share; do
  smbmap -H 192.168.1.100 -s $share -R
done

# Batch download from multiple hosts
for ip in $(cat hosts.txt); do
  smbmap -H $ip -u admin -p pass -s share --download / /tmp/$ip/
done
```

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Connection refused | Check SMB: `nmap -p 445 TARGET` |
| Logon failure | Verify creds: `-u user -p pass` |
| Access denied | Try null session: no `-u` flag |
| No shares listed | Provide credentials: `-u admin -p pass` |
| Recursive fails | Specify path: `-r /specific/path` |
| Command fails | Needs admin: verify privileges |
