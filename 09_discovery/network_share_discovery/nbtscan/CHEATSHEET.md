---
title: "nbtscan Cheatsheet"
description: "Quick reference for nbtscan NetBIOS name service scanning commands"
tool: nbtscan
category: NetBIOS Enumeration
---

# nbtscan Cheatsheet

## Quick Start

```bash
# Scan single host
nbtscan 192.168.1.100

# Scan subnet
nbtscan 192.168.1.0/24

# Scan IP range
nbtscan 192.168.1.1-192.168.1.254

# Verbose scan
nbtscan -v 192.168.1.0/24
```

## Common Flags

| Flag | Description | Example |
|------|-------------|---------|
| `-r <range>` | IP range to scan | `-r 192.168.1.0/24` |
| `-f <file>` | Read targets from file | `-f hosts.txt` |
| `-v` | Verbose output | `-v` |
| `-q` | Quiet mode | `-q` |
| `-n` | No DNS resolution | `-n` |
| `-l` | Local mode | `-l` |
| `-o <file>` | Output file | `-o results.txt` |
| `-d` | Debug mode | `-d` |

## Examples

### Basic Scanning

```bash
# Single host
nbtscan 192.168.1.100

# Subnet
nbtscan 192.168.1.0/24

# IP range
nbtscan 192.168.1.1-192.168.1.50

# From file
nbtscan -f targets.txt
```

### Output Options

```bash
# Verbose output
nbtscan -v 192.168.1.0/24

# Save to file
nbtscan -o results.txt 192.168.1.0/24

# Quiet mode
nbtscan -q 192.168.1.0/24

# No DNS resolution
nbtscan -n 192.168.1.0/24
```

### Advanced Usage

```bash
# Find domain controllers
nbtscan -v 192.168.1.0/24 | grep "Domain Controller"

# Find active users
nbtscan -v 192.168.1.0/24 | grep -E "User:|<Server>"

# Extract IP addresses
nbtscan 192.168.1.0/24 | awk '{print $1}' | grep -v "^IP"

# Scan multiple subnets
for subnet in 192.168.{1,2,3}.0/24; do
  nbtscan $subnet >> all_hosts.txt
done
```

## Chaining

```bash
# Discover hosts, then enumerate
nbtscan 192.168.1.0/24 | awk '{print $1}' | grep -v "^IP" > hosts.txt

while read host; do
  echo "=== $host ==="
  enum4linux -a $host
done < hosts.txt

# Find servers and list shares
nbtscan -v 192.168.1.0/24 | grep "Server" | awk '{print $1}' | while read ip; do
  smbclient -L $ip -N
done

# Port scan discovered hosts
nbtscan 192.168.1.0/24 | awk '{print $1}' | grep -v "^IP" | \
  xargs -I {} nmap -p 445,139 {}
```

## Troubleshooting

| Issue | Solution |
|-------|----------|
| No response | Check firewall: `nmap -sU -p 137 TARGET` |
| Permission denied | Use sudo: `sudo nbtscan TARGET` |
| No results | Verify NetBIOS enabled on target |
| Slow scanning | Scan smaller ranges |
| Host not found | Use IP directly, not hostname |
