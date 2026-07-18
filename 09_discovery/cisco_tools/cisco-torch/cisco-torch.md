---
title: cisco-torch
category: discovery
subcategory: cisco_tools
phase: 09
mitre_attack: TA0007
tags:
  - cisco
  - scanner
  - enumeration
  - vulnerability
  - network
difficulty: intermediate
os: linux
install: sudo apt install cisco-torch
github: https://github.com/jeelabs/torch
related:
  - cisco-ocs
  - cisco-global-exploiter
  - nmap
---

# cisco-torch

## Chapter 1: Overview

**cisco-torch** is a Cisco device scanner and exploitation tool that performs comprehensive enumeration and vulnerability testing against Cisco network equipment.

### Key Features

- **Device fingerprinting**: Identify Cisco IOS version
- **Vulnerability scanning**: Test for known CVEs
- **Configuration extraction**: Download device configs
- **Credential testing**: Default and custom credentials
- **Batch processing**: Multiple target support

### Scan Types

| Scan | Description |
|------|-------------|
| Fingerprint | Identify IOS version |
| Vulnerability | Test for CVEs |
| Config | Download configuration |
| Credential | Test default creds |
| SNMP | Enumerate via SNMP |

---

## Chapter 2: Installation & Setup

```bash
sudo apt update && sudo apt install cisco-torch
```

### Verify Installation

```bash
cisco-torch -h
```

---

## Chapter 3: Basic Usage

### Fingerprint Device

```bash
cisco-torch -F 192.168.1.1
```

### Scan for Vulnerabilities

```bash
cisco-torch -V 192.168.1.1
```

### Download Configuration

```bash
cisco-torch -C 192.168.1.1
```

### Test Credentials

```bash
cisco-torch -T 192.168.1.1
```

---

## Chapter 4: Advanced Options

### Specify Port

```bash
cisco-torch -F 192.168.1.1 -p 8080
```

### Set Timeout

```bash
cisco-torch -F 192.168.1.1 --timeout 10
```

### Verbose Output

```bash
cisco-torch -F 192.168.1.1 -v
```

### Output File

```bash
cisco-torch -F 192.168.1.1 -o results.txt
```

---

## Chapter 5: Batch Scanning

### Scan Multiple Targets

```bash
cisco-torch -F -i targets.txt
```

### Scan Subnet

```bash
cisco-torch -F 192.168.1.0/24
```

### Output Format

```bash
cisco-torch -F 192.168.1.1 -o results.csv
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: Quick Assessment

```bash
cisco-torch -F 192.168.1.1
cisco-torch -V 192.168.1.1
```

### Scenario 2: Configuration Audit

```bash
# Download configs from all devices
for ip in 192.168.1.{1..10}; do
  cisco-torch -C $ip -o "${ip}_config.txt"
done
```

### Scenario 3: Vulnerability Scan

```bash
# Scan for known vulnerabilities
cisco-torch -V 192.168.1.0/24 -o vuln_report.txt
```

---

## Chapter 7: Mitigation & Defense

**For network administrators:**

- Update IOS to latest stable version
- Disable unused services
- Implement strong access controls
- Use SNMPv3 with authentication
- Regular configuration backups
- Network segmentation
- Monitor for scanning activity

### Cisco Hardening Commands

```cisco
! Disable HTTP
no ip http server

! Secure SNMP
snmp-server community secret RO
snmp-server enable traps

! Enable SSH
ip ssh version 2
crypto key generate rsa modulus 2048
```

---

## Resources

- [cisco-torch Documentation](https://www pentest-tools.com)
- [Cisco Security Advisories](https://tools.cisco.com/security/center/publicationsOverview.x)
- [OffSec PEN-200 Course](https://www.offsec.com/courses/pen-200/)

---

*Generated for educational purposes in authorized security testing environments only.*
