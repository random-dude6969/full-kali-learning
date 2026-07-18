---
title: oscanner
category: discovery
subcategory: database_discovery
phase: 09
mitre_attack: TA0007
tags:
  - database
  - oracle
  - enumeration
  - scanner
  - tns
difficulty: intermediate
os: linux
install: sudo apt install oscanner
github: https://github.com/0x90/oscanner
related:
  - tnscmd10g
  - sidguess
  - mysql
---

# oscanner

## Chapter 1: Overview

**oscanner** is an Oracle database scanner that enumerates Oracle TNS listeners, discovers databases, and tests for common vulnerabilities and default credentials.

### Key Features

- **TNS listener scanning**: Discover Oracle listeners
- **Database enumeration**: Find Oracle instances
- **Default credential testing**: Test common passwords
- **Version detection**: Identify Oracle versions
- **SID guessing**: Discover database SIDs

### Scan Capabilities

| Scan Type | Description |
|-----------|-------------|
| Listener | Scan TNS listeners |
| SID | Guess database SIDs |
| Default creds | Test default passwords |
| Version | Detect Oracle version |
| Info | Gather listener info |

---

## Chapter 2: Installation & Setup

```bash
sudo apt update && sudo apt install oscanner
```

### Verify Installation

```bash
oscanner --help
```

---

## Chapter 3: Basic Usage

### Scan Listener

```bash
oscanner -h 192.168.1.1
```

### Scan with Port

```bash
oscanner -h 192.168.1.1 -P 1521
```

### Guess SID

```bash
oscanner -h 192.168.1.1 -S
```

### Test Default Credentials

```bash
oscanner -h 192.168.1.1 -D
```

---

## Chapter 4: Scan Options

### Specify Service Name

```bash
oscanner -h 192.168.1.1 -s ORCL
```

### Timeout

```bash
oscanner -h 192.168.1.1 -t 10
```

### Verbose Output

```bash
oscanner -h 192.168.1.1 -v
```

### Output File

```bash
oscanner -h 192.168.1.1 -o results.txt
```

---

## Chapter 5: SID Guessing

### Default SID List

```bash
oscanner -h 192.168.1.1 -S
```

### Custom SID List

```bash
oscanner -h 192.168.1.1 -S -F sids.txt
```

### Common SIDs

```
ORCL
XE
PROD
TEST
DEV
DB01
```

---

## Chapter 6: Credential Testing

### Default Credentials

```bash
oscanner -h 192.168.1.1 -D
```

### Custom Credentials

```bash
oscanner -h 192.168.1.1 -U users.txt -P passwords.txt
```

### Common Default Credentials

| Username | Password |
|----------|----------|
| sys | change_on_install |
| system | manager |
| scott | tiger |
| hr | hr |

---

## Chapter 7: Real-World Scenarios

### Scenario 1: Oracle Discovery

```bash
# Scan for Oracle listeners
nmap -p 1521 192.168.1.0/24

# Scan discovered hosts
oscanner -h 192.168.1.1 -v
```

### Scenario 2: SID Enumeration

```bash
# Guess SIDs
oscanner -h 192.168.1.1 -S

# Use discovered SID
oscanner -h 192.168.1.1 -s ORCL -D
```

### Scenario 3: Full Assessment

```bash
# Complete Oracle audit
oscanner -h 192.168.1.1 -S -D -v -o oracle_audit.txt
```

---

## Chapter 8: Mitigation & Defense

**For administrators:**

- Restrict TNS listener access
- Change default passwords
- Implement access controls
- Enable listener logging
- Use Oracle Advanced Security
- Network segmentation
- Regular security updates

### Oracle Security Commands

```bash
# Stop unused listeners
lsnrctl stop

# Change default passwords
ALTER USER sys IDENTIFIED BY new_password;
ALTER USER system IDENTIFIED BY new_password;

# Enable listener logging
lsnrctl set parameter LOG_FILE
```

---

## Resources

- [Oracle Security Guide](https://docs.oracle.com/en/database/oracle/oracle-database/19/seguj/)
- [Oracle TNS Documentation](https://docs.oracle.com/en/database/oracle/oracle-database/19/netrf/)
- [OffSec PEN-200 Course](https://www.offsec.com/courses/pen-200/)

---

*Generated for educational purposes in authorized security testing environments only.*
