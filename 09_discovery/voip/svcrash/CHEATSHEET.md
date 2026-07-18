---
title: "svcrash Cheatsheet"
description: "Quick reference for svcrash sipvicious scan controller"
mitre_tactic: discovery
mitre_tactic_id: TA0007
mitre_subcategory: voip
categories:
  - cheatsheet
  - voip
  - utility
tags:
  - svcrash
  - sipvicious
  - scan-control
  - quick-reference
---

# svcrash Cheatsheet

## Quick Start

```bash
# Stop all scans on target
svcrash --fromip=192.168.1.100

# Stop scans on port range
svcrash --fromport=5060 --toport=5080

# Force stop all scans
svcrash --fromip=192.168.1.100 --force
```

## Common Flags

### Target Selection

| Flag | Description | Example |
|------|-------------|---------|
| `--fromip` | Target IP | `--fromip=192.168.1.100` |
| `--fromport` | Start port | `--fromport=5060` |
| `--toport` | End port | `--toport=5080` |
| `--server` | SIP server | `--server=192.168.1.100` |

### Protocol

| Flag | Description |
|------|-------------|
| `--proto=udp` | UDP protocol |
| `--proto=tcp` | TCP protocol |

### Control

| Flag | Description |
|------|-------------|
| `--force` | Force termination |
| `--verbose` | Verbose output |
| `--debug` | Debug output |

## Examples

### Basic Termination

```bash
# Stop by IP
svcrash --fromip=192.168.1.100

# Stop by server
svcrash --server=192.168.1.100

# Stop by port
svcrash --fromport=5060 --toport=5060
```

### Port Ranges

```bash
# SIP standard ports
svcrash --fromport=5060 --toport=5061

# Extended range
svcrash --fromport=5060 --toport=5080

# Non-standard ports
svcrash --fromport=5080 --toport=5090
```

### Force Stop

```bash
# Force by IP
svcrash --fromip=192.168.1.100 --force

# Force by server
svcrash --server=192.168.1.100 --force

# Force all
svcrash --fromip=0.0.0.0 --force
```

### Verbose/Debug

```bash
# Verbose
svcrash --verbose --fromip=192.168.1.100

# Debug
svcrash --debug --server=192.168.1.100

# Combined
svcrash --verbose --force --fromip=192.168.1.100
```

## Chaining

### With Scan Tools

```bash
# Stop svmap
svcrash --fromip=192.168.1.100 --force

# Stop svwar
svcrash --server=192.168.1.100 --force

# Stop svcrack
svcrash --fromport=5060 --toport=5060 --force
```

### With Monitoring

```bash
# Check before stopping
ps aux | grep -E "sv(map|war|crack)"

# Stop if running
ps aux | grep -q svmap && svcrash --fromip=192.168.1.100
```

### Defensive Script

```bash
#!/bin/bash
# Auto-block sipvicious scans
while true; do
    if tcpdump -l -c 1 port 5060 2>/dev/null | grep -q "sipvicious"; then
        svcrash --fromip=$(getLastIP) --force
    fi
    sleep 5
done
```

### Database Cleanup

```bash
# Check scan state
ls -la ~/.sipvicious/*.db

# View scan log
sqlite3 ~/.sipvicious/scan_log.db "SELECT * FROM scans;"

# Reset state
rm ~/.sipvicious/*.db
```

## Troubleshooting

### No Matching Scans

```bash
# Check running processes
ps aux | grep -E "sv(map|war|crack)"

# List state files
ls -la ~/.sipvicious/*.db

# View database
sqlite3 ~/.sipvicious/scan_log.db ".tables"
```

### Database Locked

```bash
# Find processes using DB
lsof ~/.sipvicious/*.db

# Kill processes
pkill -f svmap
pkill -f svwar
pkill -f svcrack

# Remove lock files
rm ~/.sipvicious/*.db-journal
```

### Scan Still Running

```bash
# Force kill
pkill -9 -f svmap
pkill -9 -f svwar
pkill -9 -f svcrack

# Verify stopped
ps aux | grep -E "sv(map|war|crack)" | grep -v grep
```

### Return Codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | Error |
| 2 | No matching scans |

### Debug Commands

```bash
# Verbose
svcrash --verbose --fromip=192.168.1.100

# Debug
svcrash --debug --server=192.168.1.100

# Check database
sqlite3 ~/.sipvicious/scan_log.db "SELECT * FROM scans;"

# View logs
cat ~/.sipvicious/*.log
```

### Cleanup

```bash
# Remove all state
rm ~/.sipvicious/*.db

# Remove locks
rm ~/.sipvicious/*.db-journal

# Full reset
rm -rf ~/.sipvicious/
```

---

**Last Updated**: 2026
**Tool Version**: sipvicious 0.3.4
**Category**: SIP Scan Control
