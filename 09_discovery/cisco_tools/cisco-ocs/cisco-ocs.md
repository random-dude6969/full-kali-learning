---
title: cisco-ocs
category: discovery
subcategory: cisco_tools
phase: 09
mitre_attack: TA0007
tags:
  - cisco
  - scanner
  - vulnerability
  - network
difficulty: beginner
os: linux
install: sudo apt install cisco-ocs
github: https://github.com/ndyck/cisco-ocs
related:
  - cisco-torch
  - cisco-global-exploiter
  - nmap
---

# cisco-ocs

## Chapter 1: Overview

**cisco-ocs** (Cisco Scanner) scans Cisco devices for default credentials and known vulnerabilities. It's a quick assessment tool for Cisco network devices.

### Key Features

- **Default credential testing**: Common username/password combinations
- **SNMP community string testing**: Default and common strings
- **Vulnerability scanning**: Known Cisco vulnerabilities
- **Batch scanning**: Multiple targets
- **Output reporting**: Text and CSV formats

### Scan Capabilities

| Scan Type | Description |
|-----------|-------------|
| HTTP | Web interface default creds |
| Telnet | Telnet default creds |
| SSH | SSH default creds |
| SNMP | Community strings |

---

## Chapter 2: Installation & Setup

```bash
sudo apt update && sudo apt install cisco-ocs
```

### Verify Installation

```bash
cisco-ocs -h
```

---

## Chapter 3: Basic Usage

### Scan Single Target

```bash
cisco-ocs 192.168.1.1
```

### Scan with Options

```bash
cisco-ocs -t 192.168.1.1 -p 80
```

### Verbose Output

```bash
cisco-ocs -t 192.168.1.1 -v
```

### Save Results

```bash
cisco-ocs -t 192.168.1.1 -o results.txt
```

---

## Chapter 4: Scan Options

### Specify Port

```bash
cisco-ocs -t 192.168.1.1 -p 8080
```

### Enable All Scans

```bash
cisco-ocs -t 192.168.1.1 -a
```

### Timeout

```bash
cisco-ocs -t 192.168.1.1 --timeout 10
```

---

## Chapter 5: Network Scanning

### Scan Subnet

```bash
for ip in 192.168.1.{1..254}; do
  cisco-ocs -t $ip -o "${ip}_results.txt"
done
```

### Scan from File

```bash
while read ip; do
  cisco-ocs -t $ip
done < targets.txt
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: Quick Cisco Audit

```bash
cisco-ocs 192.168.1.1
```

### Scenario 2: Network Assessment

```bash
# Find Cisco devices
nmap -p 80,443,23,22 192.168.1.0/24 -oG - | grep "Open" > cisco_hosts.txt

# Scan each
while read host; do
  cisco-ocs -t $host
done < cisco_hosts.txt
```

---

## Chapter 7: Mitigation & Defense

**For network administrators:**

- Change all default credentials
- Disable unused services (Telnet, HTTP)
- Use SSH with key-based auth
- Implement SNMPv3
- Regular firmware updates
- Network segmentation
- Access control lists

---

## Resources

- [cisco-ocs Documentation](https://github.com/ndyck/cisco-ocs)
- [Cisco Hardening Guide](https://www.cisco.com/c/en/us/support/docs/ip/access-lists/13608-5.html)
- [OffSec PEN-200 Course](https://www.offsec.com/courses/pen-200/)

---

*Generated for educational purposes in authorized security testing environments only.*
