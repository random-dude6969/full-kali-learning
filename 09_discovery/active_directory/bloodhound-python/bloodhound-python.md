---
title: bloodhound-python
category: discovery
subcategory: active_directory
phase: 09
mitre_attack: TA0007
tags:
  - active_directory
  - enumeration
  - python
  - linux
  - ldap
difficulty: advanced
os: linux
install: sudo apt install bloodhound-python
github: https://github.com/fox-it/BloodHound.py
related:
  - bloodhound
  - sharphound
  - ldapsearch
---

# bloodhound-python

## Chapter 1: Overview

**bloodhound-python** is a Python-based data collector for BloodHound that runs on Linux systems. It performs Active Directory enumeration via LDAP and exports data in BloodHound-compatible format.

### Key Features

- **Linux support**: Run from Kali without Windows
- **LDAP collection**: Users, Groups, ACLs, Trusts
- **Multiple auth methods**: Password, hash, Kerberos
- **Stealth options**: Reduce detection
- **Output format**: BloodHound-compatible JSON

### Collection Types

| Type | Description |
|------|-------------|
| Users | User accounts and properties |
| Groups | Group memberships |
| ACLs | Access Control Lists |
| Computers | Computer objects |
| Trusts | Domain trust relationships |
| Sessions | Active sessions |

---

## Chapter 2: Installation & Setup

```bash
sudo apt update && sudo apt install bloodhound-python
```

### Dependencies

```bash
sudo pip3 install bloodhound
```

### Verify Installation

```bash
bloodhound-python --help
```

---

## Chapter 3: Basic Usage

### Password Authentication

```bash
bloodhound-python -u user -p 'P@ssw0rd' -d domain.local -ns 192.168.1.1 -c All
```

### Hash Authentication

```bash
bloodhound-python -u user -H aad3b435b51404eeaad3b435b51404ee:da76f...
```

### Kerberos Authentication

```bash
bloodhound-python -u user -k -d domain.local -ns 192.168.1.1 -c All
```

---

## Chapter 4: Collection Options

### Collect All Data

```bash
bloodhound-python -u user -p pass -d domain.local -ns 192.168.1.1 -c All
```

### Collect Specific Data

```bash
bloodhound-python -u user -p pass -d domain.local -ns 192.168.1.1 -c Users,Groups
```

### Available Collections

```bash
-c Users
-c Groups
-c Computers
-c ACLs
-c Trusts
-c Sessions
-c All
```

---

## Chapter 5: Connection Options

### Specify Domain Controller

```bash
bloodhound-python -u user -p pass -d domain.local -ns 192.168.1.1 -dc 192.168.1.1
```

### Use DNS Resolution

```bash
bloodhound-python -u user -p pass -d domain.local -ns 192.168.1.1 --dns-tcp
```

### Disable SSL Verification

```bash
bloodhound-python -u user -p pass -d domain.local -ns 192.168.1.1 --disable-prefill
```

---

## Chapter 6: Stealth Options

### Reduce LDAP Queries

```bash
bloodhound-python -u user -p pass -d domain.local -ns 192.168.1.1 -c All --stealth
```

### Loop Collection

```bash
# Collect in intervals
while true; do
  bloodhound-python -u user -p pass -d domain.local -ns 192.168.1.1 -c All --loopduration 02:00:00
  sleep 3600
done
```

### Randomize Timing

```bash
bloodhound-python -u user -p pass -d domain.local -ns 192.168.1.1 -c All --random
```

---

## Chapter 7: Real-World Scenarios

### Scenario 1: Quick AD Enumeration

```bash
bloodhound-python -u user@domain.local -p 'P@ssw0rd' -d domain.local -ns 192.168.1.1 -c All
```

### Scenario 2: From Linux without Windows

```bash
# If you have domain credentials
bloodhound-python -u administrator -p 'Password123' -d corp.local -ns 10.0.0.1 -c All

# Upload output to BloodHound GUI
```

### Scenario 3: Stealth Assessment

```bash
bloodhound-python -u lowpriv -p 'Password123' -d corp.local -ns 10.0.0.1 -c Users,Groups --stealth
```

---

## Chapter 8: Mitigation & Defense

**For defenders:**

- **Monitor LDAP queries**: Alert on unusual patterns
- **Rate limiting**: Block excessive enumeration
- **Service account protection**: Limit permissions
- **Network segmentation**: Restrict LDAP access
- **Audit logging**: Track enumeration attempts
- **Honey tokens**: Detect BloodHound collection

### Detection Queries (Splunk)

```spl
index=main EventCode=4662 | grep "LDAP"
```

### Detection Queries (Sigma)

```yaml
title: BloodHound LDAP Enumeration
status: experimental
logsource:
  category: directory_service
detection:
  selection:
    EventID: 4662
  condition: selection
```

---

## Resources

- [BloodHound.py GitHub](https://github.com/fox-it/BloodHound.py)
- [BloodHound Documentation](https://bloodhound.readthedocs.io/)
- [OffSec PEN-200 Course](https://www.offsec.com/courses/pen-200/)
- [AD Security](https://adsecurity.org/)

---

*Generated for educational purposes in authorized security testing environments only.*
