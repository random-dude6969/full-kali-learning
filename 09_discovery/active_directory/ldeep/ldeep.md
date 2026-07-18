---
title: ldeep
category: discovery
subcategory: active_directory
phase: 09
mitre_attack: TA0007
tags:
  - active_directory
  - ldap
  - enumeration
  - linux
  - python
difficulty: intermediate
os: linux
install: sudo apt install ldeep
github: https://github.com/franc-music/ldeep
related:
  - bloodhound-python
  - ldapsearch
  - windapsearch
---

# ldeep

## Chapter 1: Overview

**ldeep** is a Linux tool for LDAP enumeration and Active Directory information gathering. It provides various LDAP queries to extract users, groups, computers, and policies from Active Directory.

### Key Features

- **LDAP enumeration**: Query AD objects
- **Multiple output formats**: Text, JSON, CSV
- **Stealth options**: Reduce detection
- **Built-in queries**: Common AD enumeration tasks
- **Cache support**: Store results locally

### Enumeration Capabilities

| Query | Description |
|-------|-------------|
| users | List all domain users |
| groups | List all domain groups |
| computers | List all domain computers |
| policies | Extract GPO policies |
| trusts | Enumerate domain trusts |
| user | Specific user details |

---

## Chapter 2: Installation & Setup

```bash
sudo apt update && sudo apt install ldeep
```

### Dependencies

```bash
sudo pip3 install ldap3
```

### Verify Installation

```bash
ldeep --help
```

---

## Chapter 3: Basic Usage

### LDAP Authentication

```bash
ldeep ldap -u user -p 'P@ssw0rd' -d domain.local -s ldap://192.168.1.1
```

### List Users

```bash
ldeep ldap -u user -p 'P@ssw0rd' -d domain.local -s ldap://192.168.1.1 users
```

### List Groups

```bash
ldeep ldap -u user -p 'P@ssw0rd' -d domain.local -s ldap://192.168.1.1 groups
```

### List Computers

```bash
ldeep ldap -u user -p 'P@ssw0rd' -d domain.local -s ldap://192.168.1.1 computers
```

---

## Chapter 4: User Enumeration

### All Users

```bash
ldeep ldap -u user -p 'P@ssw0rd' -d domain.local -s ldap://192.168.1.1 users
```

### Specific User

```bash
ldeep ldap -u user -p 'P@ssw0rd' -d domain.local -s ldap://192.168.1.1 user admin
```

### Users with SPN (Kerberoastable)

```bash
ldeep ldap -u user -p 'P@ssw0rd' -d domain.local -s ldap://192.168.1.1 kerberoastable
```

### Users with AdminCount

```bash
ldeep ldap -u user -p 'P@ssw0rd' -d domain.local -s ldap://192.168.1.1 privileged
```

---

## Chapter 5: Group Enumeration

### All Groups

```bash
ldeep ldap -u user -p 'P@ssw0rd' -d domain.local -s ldap://192.168.1.1 groups
```

### Group Members

```bash
ldeep ldap -u user -p 'P@ssw0rd' -d domain.local -s ldap://192.168.1.1 group_members "Domain Admins"
```

### Nested Groups

```bash
ldeep ldap -u user -p 'P@ssw0rd' -d domain.local -s ldap://192.168.1.1 nested_groups "Domain Admins"
```

---

## Chapter 6: Advanced Enumeration

### Computer Enumeration

```bash
ldeep ldap -u user -p 'P@ssw0rd' -d domain.local -s ldap://192.168.1.1 computers
```

### Policies

```bash
ldeep ldap -u user -p 'P@ssw0rd' -d domain.local -s ldap://192.168.1.1 policies
```

### Domain Trusts

```bash
ldeep ldap -u user -p 'P@ssw0rd' -d domain.local -s ldap://192.168.1.1 trusts
```

### LDAP Search

```bash
ldeep ldap -u user -p 'P@ssw0rd' -d domain.local -s ldap://192.168.1.1 search "(objectClass=user)"
```

---

## Chapter 7: Real-World Scenarios

### Scenario 1: Quick AD Enumeration

```bash
ldeep ldap -u user -p 'P@ssw0rd' -d domain.local -s ldap://192.168.1.1 users > users.txt
ldeep ldap -u user -p 'P@ssw0rd' -d domain.local -s ldap://192.168.1.1 groups > groups.txt
ldeep ldap -u user -p 'P@ssw0rd' -d domain.local -s ldap://192.168.1.1 computers > computers.txt
```

### Scenario 2: Kerberoasting Preparation

```bash
# Find Kerberoastable users
ldeep ldap -u user -p 'P@ssw0rd' -d domain.local -s ldap://192.168.1.1 kerberoastable > kerberoastable.txt
```

### Scenario 3: Privileged Access Audit

```bash
# Find privileged users
ldeep ldap -u user -p 'P@ssw0rd' -d domain.local -s ldap://192.168.1.1 privileged > privileged.txt

# Find DA members
ldeep ldap -u user -p 'P@ssw0rd' -d domain.local -s ldap://192.168.1.1 group_members "Domain Admins" > da_members.txt
```

---

## Chapter 8: Mitigation & Defense

**For defenders:**

- **Monitor LDAP queries**: Detect enumeration attempts
- **Rate limiting**: Block excessive queries
- **Least privilege**: Limit LDAP read access
- **Network segmentation**: Restrict LDAP access
- **Audit logging**: Track enumeration
- **Honey tokens**: Detect ldeep queries

### Detection Queries

```spl
# Splunk LDAP monitoring
index=main EventCode=4662 ObjectType="*user*"
```

---

## Resources

- [ldeep GitHub](https://github.com/franc-music/ldeep)
- [LDAP Security](https://docs.microsoft.com/en-us/windows/security/identity-protection/credential-guard/credential-guard)
- [OffSec PEN-200 Course](https://www.offsec.com/courses/pen-200/)
- [AD Security](https://adsecurity.org/)

---

*Generated for educational purposes in authorized security testing environments only.*
