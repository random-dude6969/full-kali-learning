---
title: bloodhound
category: discovery
subcategory: active_directory
phase: 09
mitre_attack: TA0007
tags:
  - active_directory
  - enumeration
  - graph
  - attack_paths
  - neo4j
  -SharpHound
difficulty: advanced
os: linux
install: sudo apt install bloodhound
github: https://github.com/BloodHoundAD/BloodHound
related:
  - bloodhound-python
  - sharphound
  - azurehound
  - ldeep
---

# bloodhound

## Chapter 1: Overview

**BloodHound** is a single-page JavaScript web application that uses graph theory to reveal hidden relationships in Active Directory environments. It identifies attack paths to Domain Admin and other high-value targets.

### Key Features

- **Graph visualization**: Visual attack path mapping
- **Path analysis**: Find shortest paths to Domain Admin
- **Collection methods**: SharpHound (Windows), bloodhound-python (Linux)
- **Database backend**: Neo4j graph database
- **Pre-built queries**: Common attack scenarios
- **Custom analysis**: Cypher query language

### Attack Path Types

| Path Type | Description |
|-----------|-------------|
| Shortest Path to DA | Fastest route to Domain Admin |
| Find All Domain Admins | Identify all DA accounts |
| Kerberoastable Users | Users with SPNs |
| AS-REP Roastable | Users without preauth |
| Constrained Delegation | Delegation attack paths |
| ACL Abuse | Permission abuse paths |

---

## Chapter 2: Installation & Setup

```bash
sudo apt update && sudo apt install bloodhound
```

**Install size**: 281.25 MB

### Initial Setup

```bash
sudo bloodhound-setup
```

This will:
1. Start PostgreSQL service
2. Create database
3. Start Neo4j
4. Prompt for new Neo4j password

### Default Credentials

```
Neo4j: neo4j/neo4j (change immediately)
BloodHound: admin/admin (change on first login)
```

### Update Neo4j Password

```bash
sudo vim /etc/bhapi/bhapi.json
# Update "secret" field with new Neo4j password
```

---

## Chapter 3: Starting Services

### Start BloodHound

```bash
sudo bloodhound-start
```

### Access Web Interface

Open browser: `http://localhost:7474` (Neo4j) or `http://localhost:8080` (BloodHound)

### Stop Services

```bash
sudo bloodhound-stop
```

### Reset Admin Password

```bash
sudo env bhe_recreate_default_admin=true bloodhound
```

---

## Chapter 4: Data Collection

### SharpHound (Windows)

```powershell
# From Kali, transfer SharpHound.exe
# Run on Windows domain-joined machine
.\SharpHound.exe -c All

# Output: zip file with collected data
```

### SharpHound Options

```powershell
# Collect all data
.\SharpHound.exe -c All

# Collect specific data
.\SharpHound.exe -c Users,Groups,ACLs

# Stealth collection
.\SharpHound.exe -c All --stealth

# Loop collection
.\SharpHound.exe -c All --loop --loopduration 02:00:00
```

### bloodhound-python (Linux)

```bash
# Basic collection
bloodhound-python -u user -p password -d domain.local -ns 192.168.1.1 -c All

# With hash
bloodhound-python -u user -H aad3b435b51404eeaad3b435b51404ee:da76f...
```

### bloodhound-python Options

```bash
# Collect specific data
bloodhound-python -u user -p pass -d domain.local -ns 192.168.1.1 -c Users,Groups

# Stealth mode
bloodhound-python -u user -p pass -d domain.local -ns 192.168.1.1 -c All --stealth
```

---

## Chapter 5: Ingesting Data

### Upload to BloodHound

1. Open BloodHound GUI
2. Click "Upload Data" (or drag and drop)
3. Select zip file from SharpHound/bloodhound-python
4. Wait for processing

### Verify Ingestion

```bash
# In Neo4j browser
MATCH (n) RETURN labels(n), count(n)
```

---

## Chapter 6: Analysis Queries

### Find Shortest Path to Domain Admin

```cypher
MATCH p=shortestPath((n:User {owned:true})-[*1..]->(g:Group {name:"DOMAIN ADMINS@"}))
RETURN p
```

### Find All Domain Admins

```cypher
MATCH (g:Group {name:"DOMAIN ADMINS@"})<-[:MemberOf]-(u:User) RETURN u
```

### Kerberoastable Users

```cypher
MATCH (u:User {hasspn:true}) RETURN u
```

### AS-REP Roastable Users

```cypher
MATCH (u:User {dontreqpreauth:true}) RETURN u
```

### Find Principals with DCSync Rights

```cyphered
MATCH p=(n)-[:GetChangesAll|:GetChanges]->(d:Domain) RETURN p
```

### Find Computers with Unconstrained Delegation

```cypher
MATCH (c:Computer {unconstraineddelegation:true}) RETURN c
```

---

## Chapter 7: Real-World Scenarios

### Scenario 1: Initial AD Assessment

```bash
# On Windows machine
.\SharpHound.exe -c All

# Transfer zip to Kali
# Upload to BloodHound
# Run pre-built queries
```

### Scenario 2: Linux-based Collection

```bash
# Enumerate from Linux
bloodhound-python -u user@domain.local -p password -d domain.local -ns 192.168.1.1 -c All

# Upload output to BloodHound
```

### Scenario 3: Stealth Assessment

```bash
# Windows
.\SharpHound.exe -c All --stealth --loopduration 02:00:00

# Linux
bloodhound-python -u user -p pass -d domain.local -ns 192.168.1.1 -c All --stealth
```

---

## Chapter 8: Mitigation & Defense

**For defenders:**

- **Monitor SharpHound**: Detect LDAP enumeration
- **Audit ACLs**: Review permission grants
- **Least privilege**: Minimize group memberships
- **Protect DA accounts**: MFA, tiered administration
- **Monitor delegation**: Alert on unconstrained delegation
- **Regular audits**: Review attack paths
- **Honey tokens**: Detect enumeration
- **Network segmentation**: Limit LDAP access

### Detection Queries

```cypher
// Find all users with SPNs
MATCH (u:User {hasspn:true}) RETURN u.name, u.serviceprincipalnames

// Find users with DCSync rights
MATCH p=(n)-[:GetChangesAll]->(d:Domain) RETURN p
```

---

## Resources

- [BloodHound Documentation](https://bloodhound.readthedocs.io/)
- [BloodHound GitHub](https://github.com/BloodHoundAD/BloodHound)
- [SharpHound GitHub](https://github.com/BloodHoundAD/SharpHound)
- [OffSec PEN-200 Course](https://www.offsec.com/courses/pen-200/)
- [OffSec PEN-300 Course](https://www.offsec.com/courses/pen-300/)
- [AD Security](https://adsecurity.org/)

---

*Generated for educational purposes in authorized security testing environments only.*
