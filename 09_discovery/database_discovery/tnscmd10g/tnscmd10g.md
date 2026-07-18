---
title: tnscmd10g
category: discovery
subcategory: database_discovery
phase: 09
mitre_attack: TA0007
tags:
  - database
  - oracle
  - tns
  - listener
  - enumeration
difficulty: intermediate
os: linux
install: sudo apt install tnscmd10g
github: https://github.com/0x90/tnscmd10g
related:
  - oscanner
  - sidguess
  - nmap
---

# tnscmd10g

## Chapter 1: Overview

**tnscmd10g** is a tool for sending TNS (Transparent Network Substrate) commands to Oracle listeners. It's used for enumeration, fingerprinting, and testing Oracle TNS listener vulnerabilities.

### Key Features

- **TNS command sending**: Send raw TNS packets
- **Listener enumeration**: Discover listener information
- **Version detection**: Identify Oracle versions
- **Vulnerability testing**: Test for known CVEs
- **DoS testing**: Test listener resilience

### TNS Commands

| Command | Description |
|---------|-------------|
| STATUS | Get listener status |
| VERSION | Get version info |
| SERVICES | List services |
| RELOAD | Reload configuration |
| STOP | Stop listener |
| START | Start listener |

---

## Chapter 2: Installation & Setup

```bash
sudo apt update && sudo apt install tnscmd10g
```

### Verify Installation

```bash
tnscmd10g --help
```

---

## Chapter 3: Basic Usage

### Send STATUS Command

```bash
tnscmd10g status -h 192.168.1.1
```

### Send VERSION Command

```bash
tnscmd10g version -h 192.168.1.1
```

### Send SERVICES Command

```bash
tnscmd10g services -h 192.168.1.1
```

---

## Chapter 4: Connection Options

### Specify Port

```bash
tnscmd10g status -h 192.168.1.1 -p 1521
```

### Set Timeout

```bash
tnscmd10g status -h 192.168.1.1 -t 10
```

### Verbose Output

```bash
tnscmd10g status -h 192.168.1.1 -v
```

---

## Chapter 5: Advanced Commands

### Reload Listener

```bash
tnscmd10g reload -h 192.168.1.1
```

### Stop Listener (Dangerous)

```bash
tnscmd10g stop -h 192.168.1.1
```

### Start Listener

```bash
tnscmd10g start -h 192.168.1.1
```

### Custom Command

```bash
tnscmd10g -h 192.168.1.1 -c "STATUS"
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: Listener Enumeration

```bash
# Get listener status
tnscmd10g status -h 192.168.1.1

# Get version
tnscmd10g version -h 192.168.1.1

# List services
tnscmd10g services -h 192.168.1.1
```

### Scenario 2: Network Oracle Scan

```bash
# Find Oracle listeners
nmap -p 1521 192.168.1.0/24

# Enumerate each
for ip in $(grep "1521/open" hosts.txt | awk '{print $2}'); do
  tnscmd10g status -h $ip
done
```

### Scenario 3: Vulnerability Assessment

```bash
# Test for TNS poisoning
tnscmd10g status -h 192.168.1.1 -v

# Check listener security
tnscmd10g services -h 192.168.1.1
```

---

## Chapter 7: Mitigation & Defense

**For administrators:**

- Restrict TNS listener access
- Enable listener authentication
- Use non-standard ports
- Implement network segmentation
- Enable listener logging
- Regular security updates

### Oracle Listener Security

```bash
# Enable listener security
lsnrctl set parameter SECURITY_FEATURES

# Restrict access
lsnrctl set parameter VALID_NODE_CHECKING

# Enable logging
lsnrctl set parameter LOG_FILE
```

---

## Resources

- [Oracle TNS Documentation](https://docs.oracle.com/en/database/oracle/oracle-database/19/netrf/)
- [tnscmd10g Usage](https://www pentest-tools.com)
- [Oracle Security Guide](https://docs.oracle.com/en/database/oracle/oracle-database/19/seguj/)
- [OffSec PEN-200 Course](https://www.offsec.com/courses/pen-200/)

---

*Generated for educational purposes in authorized security testing environments only.*
