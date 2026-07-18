---
title: "enum4linux-ng Cheatsheet"
description: "Quick reference for enum4linux-ng SMB enumeration commands"
tool: enum4linux-ng
category: SMB Enumeration
---

# enum4linux-ng Cheatsheet

## Quick Start

```bash
# Basic enumeration
enum4linux-ng 192.168.1.100

# Full enumeration
enum4linux-ng -A 192.168.1.100

# Authenticated enumeration
enum4linux-ng -u admin -p password 192.168.1.100

# JSON output
enum4linux-ng -A -O json 192.168.1.100
```

## Common Flags

| Flag | Description | Example |
|------|-------------|---------|
| `-A` | All enumeration options | `enum4linux-ng -A TARGET` |
| `-U` | User enumeration | `enum4linux-ng -U TARGET` |
| `-S` | Share enumeration | `enum4linux-ng -S TARGET` |
| `-G` | Group enumeration | `enum4linux-ng -G TARGET` |
| `-P` | Password policy | `enum4linux-ng -P TARGET` |
| `-O` | OS information | `enum4linux-ng -O TARGET` |
| `-L` | NetBIOS/LM info | `enum4linux-ng -L TARGET` |
| `-N` | Skip NetBIOS | `enum4linux-ng -N TARGET` |
| `-r` | RID cycling | `enum4linux-ng -r TARGET` |
| `-R` | Remote SAM | `enum4linux-ng -R TARGET` |
| `-u <user>` | Username | `-u administrator` |
| `-p <pass>` | Password | `-p P@ssw0rd` |
| `-d <domain>` | Domain name | `-d WORKGROUP` |
| `-o <file>` | Output file | `-o results.json` |
| `-O json` | JSON format | `-O json` |
| `-O xml` | XML format | `-O xml` |
| `-q` | Quiet mode | `-q` |
| `-v` | Verbose output | `-v` |
| `-c` | No color | `-c` |

## Examples

### Basic Enumeration

```bash
# Simple scan
enum4linux-ng 192.168.1.100

# List shares
enum4linux-ng -S 192.168.1.100

# Enumerate users
enum4linux-ng -U 192.168.1.100

# Get password policy
enum4linux-ng -P 192.168.1.100
```

### Authenticated Scans

```bash
# With credentials
enum4linux-ng -u admin -p password 192.168.1.100

# Domain authentication
enum4linux-ng -d CORP -u john -p pass123 192.168.1.100

# Guest access
enum4linux-ng -u guest -p "" 192.168.1.100
```

### Output Formats

```bash
# JSON output
enum4linux-ng -A -O json 192.168.1.100

# XML output
enum4linux-ng -A -O xml 192.168.1.100

# Save to file
enum4linux-ng -A -o results.json -O json 192.168.1.100

# Quiet JSON for parsing
enum4linux-ng -A -O json -q 192.168.1.100
```

### Advanced Usage

```bash
# Full enumeration with verbose
enum4linux-ng -A -v 192.168.1.100

# Skip NetBIOS for speed
enum4linux-ng -N -A 192.168.1.100

# RID cycling attack
enum4linux-ng -r -u admin -p pass 192.168.1.100

# Remote SAM query
enum4linux-ng -R -u admin -p pass 192.168.1.100
```

## Chaining

```bash
# Scan network for SMB hosts, then enumerate
nmap -p 445 --open 192.168.1.0/24 | grep "report" | awk '{print $NF}' > hosts.txt

while read host; do
  echo "=== $host ==="
  enum4linux-ng -A -O json "$host" > "${host}_enum.json"
done < hosts.txt

# Parse JSON with jq
cat results.json | jq '.users[].name'

# Combine with smbmap
enum4linux-ng -A 192.168.1.100 && smbmap -H 192.168.1.100
```

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Python version error | Install Python 3.6+: `python3 --version` |
| Module not found | Install deps: `pip3 install -r requirements.txt` |
| Connection refused | Verify SMB: `nmap -p 445 TARGET` |
| Logon failure | Check creds: `-u user -p 'pass'` |
| JSON parse error | Validate: `enum4linux-ng ... \| python3 -m json.tool` |
| Slow output | Skip NetBIOS: `-N` |
