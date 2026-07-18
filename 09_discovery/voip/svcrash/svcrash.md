---
title: "svcrash - sipvicious Scan Controller"
description: "Utility to stop and manage running sipvicious scans across multiple targets and sessions"
mitre_tactic: discovery
mitre_tactic_id: TA0007
mitre_subcategory: voip
categories:
  - discovery
  - voip
  - utility
  - scan-management
tags:
  - sip
  - voip
  - sipvicious
  - scan-control
  - utility
install_command: "sudo apt install sipvicious"
version: "0.3.4"
github_repo: "https://github.com/EnableSecurity/sipvicious"
kali_tools_page: "https://tools.kali.org/information-gathering-tools/sipvicious"
difficulty_level: beginner
requires_root: false
gui_available: false
related_tools:
  - sipvicious
  - svmap
  - svwar
---

# svcrash - sipvicious Scan Controller

svcrash is a utility tool within the sipvicious suite designed to stop and manage running SIP scans. It provides precise control over active svmap, svwar, and svcrack sessions, allowing operators to halt scans gracefully without leaving orphaned processes or incomplete database entries.

## 1. Introduction

In large-scale VoIP security assessments, multiple scans often run simultaneously across different network segments. svcrash addresses the operational need to manage these concurrent scans by sending specially crafted SIP packets that trigger scan termination in target sipvicious instances.

This tool is essential for:
- **Red Team Operations**: Stopping scans when detected
- **Blue Team Defense**: Terminating unauthorized sipvicious scans
- **Assessment Management**: Controlling scan lifecycle
- **Emergency Response**: Immediate scan shutdown

### Use Cases

| Scenario | Description |
|----------|-------------|
| Emergency Stop | Halt scans immediately when detected |
| Scheduled Termination | End scans at specific times |
| Multi-Session Control | Manage multiple concurrent scans |
| Cleanup | Ensure clean scan termination |
| Defense | Block adversarial SIP scanning |

## 2. Installation

### Kali Linux

```bash
sudo apt update
sudo apt install sipvicious
```

### Verify Installation

```bash
svcrash --version
which svcrash
```

## 3. Core Concepts

### Scan State Management

sipvicious stores scan state in SQLite databases:

```
~/.sipvicious/
├── svmap_192.168.1.0_24.db    # svmap state
├── svwar_192.168.1.100.db     # svwar state
├── svcrack_192.168.1.100.db   # svcrack state
└── scan_log.db                # Global scan log
```

### Crash Mechanism

svcrash terminates scans by:

1. **Database Lock**: Locking the scan database
2. **State Injection**: Writing termination flag
3. **Signal Delivery**: Sending interrupt signal
4. **Cleanup**: Removing temporary files

### Process Architecture

```
┌─────────────────────────────────────┐
│         svcrash Command             │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│      Scan Control Layer             │
│  ┌───────────┬───────────┐         │
│  │  Signal   │  Database │         │
│  │  Delivery │  Locking  │         │
│  └───────────┴───────────┘         │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│      Target Scan Processes          │
│  ┌─────┐ ┌─────┐ ┌─────┐          │
│  │svmap│ │svwar│ │svcrk│          │
│  └─────┘ └─────┘ └─────┘          │
└─────────────────────────────────────┘
```

## 4. Command Reference

### Basic Syntax

```
svcrash [options]
```

### Target Selection

| Flag | Description | Example |
|------|-------------|---------|
| `--fromip` | Target IP address | `--fromip=192.168.1.100` |
| `--fromport` | Start port range | `--fromport=5060` |
| `--toport` | End port range | `--toport=5080` |
| `--server` | SIP server address | `--server=192.168.1.100` |

### Protocol Options

| Flag | Description | Example |
|------|-------------|---------|
| `--proto` | Transport protocol | `--proto=udp` |
| `--proto=tcp` | TCP protocol | `--proto=tcp` |

### Control Options

| Flag | Description |
|------|-------------|
| `--force` | Force termination |
| `--verbose` | Verbose output |
| `--debug` | Debug output |

### Return Codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | Error |
| 2 | No matching scans |

## 5. Examples

### Basic Scan Termination

```bash
# Stop all scans on target
svcrash --fromip=192.168.1.100

# Stop scans on specific port
svcrash --fromport=5060 --toport=5060

# Stop scans by server
svcrash --server=192.168.1.100
```

### Port Range Control

```bash
# Stop scans on SIP range
svcrash --fromport=5060 --toport=5061

# Stop scans on non-standard ports
svcrash --fromport=5080 --toport=5090

# Stop all UDP scans
svcrash --proto=udp --fromip=192.168.1.100
```

### Force Termination

```bash
# Force stop all scans
svcrash --fromip=192.168.1.100 --force

# Force stop by server
svcrash --server=192.168.1.100 --force

# Force stop on port range
svcrash --fromport=5060 --toport=5080 --force
```

### Verbose Output

```bash
# Show termination details
svcrash --verbose --fromip=192.168.1.100

# Debug output
svcrash --debug --server=192.168.1.100

# Combined options
svcrash --verbose --force --fromip=192.168.1.100
```

### Advanced Scenarios

```bash
# Stop all SIP scans on network
svcrash --fromip=192.168.1.0 --fromport=5060 --toport=5061

# Stop scans on multiple ports
svcrash --fromport=5060 --toport=5080 --proto=udp

# Emergency stop all
svcrash --fromip=0.0.0.0 --force
```

## 6. Advanced Usage

### Automation Scripts

```bash
#!/bin/bash
# Emergency stop script
TARGET="$1"
echo "[!] Emergency stop initiated for $TARGET"
svcrash --fromip=$TARGET --force --verbose
echo "[+] All scans stopped"
```

### Monitoring Active Scans

```bash
# List running sipvicious processes
ps aux | grep -E "sv(map|war|crack)"

# Check scan state files
ls -la ~/.sipvicious/*.db

# View scan database
sqlite3 ~/.sipvicious/scan_log.db "SELECT * FROM scans;"
```

### Defensive Deployment

```bash
#!/bin/bash
# Monitor and block sipvicious scans
while true; do
    # Check for sipvicious User-Agent
    tcpdump -i eth0 -l -c 1 port 5060 2>/dev/null | \
        grep -q "sipvicious" && \
        svcrash --fromip=$(tcpdump -r - port 5060 | \
        awk '{print $5}' | cut -d. -f1-4 | head -1) --force
    sleep 10
done
```

### Integration with IDS

```bash
# Snort rule to detect sipvicious
alert udp any any -> $HOME_NET 5060 (
    msg:"SIP sipvicious Scan Detected";
    content:"sipvicious";
    sid:1000010; rev:1;
    flowbits:set,sipvicious;
)

# Response rule
alert udp $HOME_NET 5060 -> any any (
    msg:"svcrash Response Sent";
    content:"sipvicious";
    flowbits:isset,sipvicious;
    sid:1000011; rev:1;
    rev:1;
)
```

## 7. Detection and Defense

### Detecting svcrash Usage

| Indicator | Description |
|-----------|-------------|
| Database Lock | SQLite database locked |
| State Files | Scan state modified |
| Process Kill | Scan process terminated |
| Log Entries | Termination logged |

### Defensive Measures

**Database Protection:**

```bash
# Set database permissions
chmod 600 ~/.sipvicious/*.db

# Use file locking
flock -n ~/.sipvicious/scan.db svmap 192.168.1.0/24
```

**Process Monitoring:**

```bash
# Monitor for scan termination
inotifywait -m ~/.sipvicious/ -e modify,delete

# Alert on database changes
auditctl -w ~/.sipvicious/ -p wa -k sipvicious
```

**Network Defense:**

```bash
# Block svcrash packets
iptables -A INPUT -p udp --dport 5060 -m string \
    --string "sipvicious" -j DROP

# Rate limit SIP responses
iptables -A OUTPUT -p udp --sport 5060 \
    -m limit --limit 10/second -j ACCEPT
```

### Countermeasures for Operators

- Use `--force` for guaranteed termination
- Verify termination with `ps aux | grep sv`
- Clear state files after forced termination
- Document all scan stops in audit logs

## 8. Troubleshooting

### Common Issues

#### No Matching Scans

```bash
# Check for running scans
ps aux | grep -E "sv(map|war|crack)"

# List scan state files
ls -la ~/.sipvicious/*.db

# View scan database
sqlite3 ~/.sipvicious/scan_log.db ".tables"
```

#### Database Locked

```bash
# Find processes using database
lsof ~/.sipvicious/*.db

# Kill processes
pkill -f svmap
pkill -f svwar
pkill -f svcrack

# Remove lock files
rm ~/.sipvicious/*.db-journal
```

#### Scan Still Running

```bash
# Force kill processes
pkill -9 -f svmap
pkill -9 -f svwar
pkill -9 -f svcrack

# Verify stopped
ps aux | grep -E "sv(map|war|crack)" | grep -v grep
```

### Debug Commands

```bash
# Verbose output
svcrash --verbose --fromip=192.168.1.100

# Debug mode
svcrash --debug --server=192.168.1.100

# Check database state
sqlite3 ~/.sipvicious/scan_log.db "SELECT * FROM scans;"

# View scan logs
cat ~/.sipvicious/*.log
```

### Performance Issues

| Symptom | Solution |
|---------|----------|
| Slow termination | Use `--force` |
| Database errors | Remove lock files |
| Process hanging | Kill with `pkill -9` |
| State corruption | Reset database |

### Cleanup Procedures

```bash
# Remove all scan state
rm ~/.sipvicious/*.db

# Remove lock files
rm ~/.sipvicious/*.db-journal

# Reset database
rm ~/.sipvicious/sipvicious.db
sqlite3 ~/.sipvicious/sipvicious.db ".tables"
```

## Resources

### Official Documentation
- [svcrash GitHub](https://github.com/EnableSecurity/sipvicious)
- [sipvicious Wiki](https://github.com/EnableSecurity/sipvicious/wiki)

### Related Tools
- **svmap**: SIP scanner
- **svwar**: Extension war dialer
- **svcrack**: Credential cracker
- **svreport**: Report generator

### System Administration
- [flock - File Locking](https://linux.die.net/man/1/flock)
- [inotifywait - File Monitoring](https://linux.die.net/man/1/inotifywait)
- [auditctl - Audit Control](https://linux.die.net/man/8/auditctl)

### Related MITRE ATT&CK Techniques
- **T1562.001 - Disable or Modify Tools**: Stopping security tools
- **T1562.002 - Disable Windows Event Logging**: Scan evasion
- **T1070.004 - File Deletion**: Removing scan artifacts
