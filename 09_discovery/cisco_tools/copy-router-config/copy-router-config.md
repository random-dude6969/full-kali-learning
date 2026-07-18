---
title: copy-router-config
category: discovery
subcategory: cisco_tools
phase: 09
mitre_attack: TA0007
tags:
  - cisco
  - configuration
  - backup
  - tftp
  - network
difficulty: intermediate
os: linux
install: sudo apt install copy-router-config
github: https://github.com/jeelabs/torch
related:
  - cisco-torch
  - merge-router-config
  - tftp
---

# copy-router-config

## Chapter 1: Overview

**copy-router-config** copies Cisco router and switch configurations via TFTP, SNMP, or other methods. It's used for configuration backup, auditing, and extraction during penetration tests.

### Key Features

- **Configuration backup**: Save running/startup configs
- **Multiple methods**: TFTP, SCP, SNMP
- **Batch operations**: Multiple device support
- **Output formatting**: Text and structured formats
- **Authentication support**: Various credential types

### Transfer Methods

| Method | Description |
|--------|-------------|
| TFTP | Trivial File Transfer Protocol |
| SCP | Secure Copy Protocol |
| SNMP | Configuration via MIB |
| HTTP | Web interface download |

---

## Chapter 2: Installation & Setup

```bash
sudo apt update && sudo apt install copy-router-config
```

### Verify Installation

```bash
copy-router-config --help
```

### Setup TFTP Server

```bash
sudo apt install atftpd
sudo systemctl start atftpd
```

---

## Chapter 3: Basic Usage

### Copy via TFTP

```bash
copy-router-config -t 192.168.1.1 -m tftp -s 192.168.1.100
```

### Copy via SCP

```bash
copy-router-config -t 192.168.1.1 -m scp -u admin -p password
```

### Copy via SNMP

```bash
copy-router-config -t 192.168.1.1 -m snmp -c public
```

---

## Chapter 4: Configuration Options

### Specify Config Type

```bash
# Running config
copy-router-config -t 192.168.1.1 -c running

# Startup config
copy-router-config -t 192.168.1.1 -c startup
```

### Output Directory

```bash
copy-router-config -t 192.168.1.1 -o /backup/configs/
```

### Verbose Output

```bash
copy-router-config -t 192.168.1.1 -v
```

---

## Chapter 5: Batch Operations

### Multiple Devices

```bash
while read ip; do
  copy-router-config -t $ip -o "/backup/$ip/"
done < routers.txt
```

### Scan and Backup

```bash
nmap -p 23,22 192.168.1.0/24 -oG - | grep "Open" | awk '{print $2}' | while read ip; do
  copy-router-config -t $ip -o "/backup/$ip/"
done
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: Configuration Backup

```bash
# Backup all routers
for ip in 192.168.1.{1..10}; do
  copy-router-config -t $ip -m tftp -s 192.168.1.100
done
```

### Scenario 2: Configuration Audit

```bash
# Download configs for review
copy-router-config -t 192.168.1.1 -m scp -u admin -p pass -o /audit/

# Search for sensitive information
grep -r "password" /audit/
grep -r "enable secret" /audit/
```

### Scenario 3: Incident Response

```bash
# Backup configs during incident
copy-router-config -t 192.168.1.1 -m scp -u admin -p pass -o /forensics/
```

---

## Chapter 7: Mitigation & Defense

**For network administrators:**

- Restrict TFTP/SCP access
- Use encrypted transfer methods
- Implement access controls
- Monitor configuration changes
- Regular configuration audits
- Configuration version control

### Cisco Security Commands

```cisco
! Restrict TFTP
ip tftp source-interface Loopback0

! Enable SCP
ip scp server enable

! Secure SNMP
snmp-server community secret RO
snmp-server host 192.168.1.100 version 2c secret
```

---

## Resources

- [Cisco Configuration Guide](https://www.cisco.com/c/en/us/td/docs/routers/access/wireless/software/guide/SW_Assembly.html)
- [copy-router-config Documentation](https://www pentest-tools.com)
- [OffSec PEN-200 Course](https://www.offsec.com/courses/pen-200/)

---

*Generated for educational purposes in authorized security testing environments only.*
