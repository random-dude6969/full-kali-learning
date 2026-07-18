---
title: "enum4linux Cheatsheet"
description: "Quick reference for enum4linux SMB enumeration commands"
tool: enum4linux
category: SMB Enumeration
---

# enum4linux Cheatsheet

## Quick Start

```bash
# Basic enumeration
enum4linux 192.168.1.100

# Full enumeration
enum4linux -a 192.168.1.100

# Authenticated enumeration
enum4linux -u admin -p password 192.168.1.100
```

## Common Flags

| Flag | Description | Example |
|------|-------------|---------|
| `-a` | All enumeration options | `enum4linux -a TARGET` |
| `-U` | User enumeration | `enum4linux -U TARGET` |
| `-S` | Share enumeration | `enum4linux -S TARGET` |
| `-G` | Group enumeration | `enum4linux -G TARGET` |
| `-P` | Password policy | `enum4linux -P TARGET` |
| `-O` | OS information | `enum4linux -O TARGET` |
| `-L` | NetBIOS/LM info | `enum4linux -L TARGET` |
| `-u <user>` | Username | `-u administrator` |
| `-p <pass>` | Password | `-p P@ssw0rd` |
| `-d <domain>` | Domain name | `-d WORKGROUP` |
| `-r` | RID cycling | `-r` |
| `-o <file>` | Output file | `-o results.txt` |
| `-v` | Verbose output | `-v` |
| `-n` | No DNS resolution | `-n` |

## Examples

### Basic Enumeration

```bash
# Simple scan
enum4linux 192.168.1.100

# List shares
enum4linux -S 192.168.1.100

# Enumerate users
enum4linux -U 192.168.1.100

# Get password policy
enum4linux -P 192.168.1.100
```

### Authenticated Scans

```bash
# With credentials
enum4linux -u admin -p password 192.168.1.100

# Domain authentication
enum4linux -d CORP -u john -p pass123 192.168.1.100

# Guest access
enum4linux -u guest -p "" 192.168.1.100
```

### Advanced Usage

```bash
# Full enumeration with verbose
enum4linux -a -v 192.168.1.100

# Save to file
enum4linux -a -o output.txt 192.168.1.100

# RID cycling attack
enum4linux -r -u admin -p pass 192.168.1.100

# Execute smbclient commands
enum4linux -c "ls;quit" 192.168.1.100
```

## Chaining

```bash
# Scan network for SMB hosts, then enumerate
nmap -p 445 --open 192.168.1.0/24 | grep "report" | awk '{print $NF}' > hosts.txt

while read host; do
  echo "=== $host ==="
  enum4linux -a $host > "${host}_enum.txt"
done < hosts.txt

# Combine with smbmap
enum4linux -a 192.168.1.100 && smbmap -H 192.168.1.100
```

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Connection refused | Verify SMB service: `nmap -p 445 TARGET` |
| Logon failure | Check credentials: `-u user -p 'pass'` |
| Access denied | Try authenticated scan |
| No users found | Try RID cycling: `-r` |
| Slow output | Redirect stderr: `2>/dev/null` |
