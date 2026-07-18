---
title: sidguess
category: discovery
subcategory: database_discovery
phase: 09
mitre_attack: TA0007
tags:
  - database
  - oracle
  - sid
  - enumeration
  - brute_force
difficulty: intermediate
os: linux
install: sudo apt install sidguess
github: https://github.com/c边车/sidguess
related:
  - oscanner
  - tnscmd10g
  - nmap
---

# sidguess

## Chapter 1: Overview

**sidguess** is a tool for guessing Oracle Database SIDs (System Identifiers). It performs brute-force attacks against Oracle TNS listeners to discover valid database instances.

### Key Features

- **SID brute force**: Guess database identifiers
- **Fast scanning**: Multi-threaded attacks
- **Custom wordlists**: Use your own SID lists
- **Parallel attacks**: Multiple concurrent connections
- **Output logging**: Save discovered SIDs

### Why SID Matters

```
Oracle SID = Database Identifier
- Required for connection
- Often default or guessable
- Enables further enumeration
- First step in Oracle attacks
```

---

## Chapter 2: Installation & Setup

```bash
sudo apt update && sudo apt install sidguess
```

### Verify Installation

```bash
sidguess --help
```

---

## Chapter 3: Basic Usage

### Guess SID

```bash
sidguess 192.168.1.1
```

### Specify Port

```bash
sidguess 192.168.1.1 -p 1521
```

### Verbose Output

```bash
sidguess 192.168.1.1 -v
```

### Save Results

```bash
sidguess 192.168.1.1 -o found_sids.txt
```

---

## Chapter 4: Wordlist Options

### Default Wordlist

```bash
sidguess 192.168.1.1
```

### Custom Wordlist

```bash
sidguess 192.168.1.1 -w custom_sids.txt
```

### Common SIDs

```
ORCL
XE
PROD
TEST
DEV
DB01
ORACLE
SID
DB
DATABASE
```

---

## Chapter 5: Attack Options

### Thread Count

```bash
sidguess 192.168.1.1 -t 10
```

### Timeout

```bash
sidguess 192.168.1.1 -T 5
```

### Delay Between Attempts

```bash
sidguess 192.168.1.1 -d 1
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: Quick SID Discovery

```bash
sidguess 192.168.1.1
```

### Scenario 2: Network Oracle Scan

```bash
# Find Oracle listeners
nmap -p 1521 192.168.1.0/24 -oG - | grep "Open"

# Guess SIDs on each
for ip in $(grep "1521/open" hosts.txt | awk '{print $2}'); do
  sidguess $ip -o "${ip}_sids.txt"
done
```

### Scenario 3: Full Oracle Assessment

```bash
# Discover SIDs
sidguess 192.168.1.1 -v

# Test default credentials with discovered SID
oscanner -h 192.168.1.1 -s ORCL -D
```

---

## Chapter 7: Mitigation & Defense

**For administrators:**

- Change default SIDs
- Restrict TNS listener access
- Implement access controls
- Enable listener logging
- Use non-standard ports
- Network segmentation

### Oracle Security

```bash
# Change default SID during installation
# Restrict listener access
lsnrctl set parameter VALID_NODE_CHECKING

# Enable logging
lsnrctl set parameter LOG_FILE
```

---

## Resources

- [Oracle Security Guide](https://docs.oracle.com/en/database/oracle/oracle-database/19/seguj/)
- [SID Brute Force](https://www pentest-tools.com)
- [OffSec PEN-200 Course](https://www.offsec.com/courses/pen-200/)

---

*Generated for educational purposes in authorized security testing environments only.*
