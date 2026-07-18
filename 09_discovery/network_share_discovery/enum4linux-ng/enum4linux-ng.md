---
title: "enum4linux-ng - Modern SMB Enumeration Tool"
description: "Python-based rewrite of enum4linux with JSON/XML output support for comprehensive SMB/Samba enumeration"
mitre_tactic: discovery
mitre_tactic_id: TA0007
mitre_subcategory: network_share_discovery
categories:
  - Network Enumeration
  - SMB Enumeration
  - Windows Enumeration
  - Samba Enumeration
tags:
  - smb
  - netbios
  - windows
  - samba
  - enumeration
  - python
  - json
install_command: "sudo apt install enum4linux-ng"
version: "1.0+"
github_repo: "https://github.com/cddmp/enum4linux-ng"
kali_tools_page: "https://www.kali.org/tools/enum4linux-ng/"
difficulty_level: beginner
requires_root: false
gui_available: false
related_tools:
  - enum4linux
  - smbclient
  - smbmap
  - rpcclient
---

# enum4linux-ng - Modern SMB Enumeration Tool

## 1. Introduction

### What is enum4linux-ng?

enum4linux-ng is a modern, Python-based rewrite of the classic enum4linux tool, designed for enumerating information from Windows and Samba systems using the SMB protocol. The tool preserves the core functionality of the original while adding contemporary features like JSON and XML output formats, improved error handling, and better performance through asynchronous operations.

### History

Developed as a successor to the aging bash-based enum4linux, enum4linux-ng was created to address the limitations of the original tool while maintaining compatibility. The project began in 2020 as a community effort to modernize SMB enumeration for current security assessments, incorporating lessons learned from years of penetration testing and security auditing.

### When to Use enum4linux-ng

- **Modern Penetration Testing**: Enumerating contemporary Windows and Samba environments
- **Automated Security Assessments**: JSON output enables easy integration with automation frameworks
- **Compliance Auditing**: Structured output for report generation
- **CI/CD Security Testing**: Integration into continuous security pipelines
- **CTF Challenges**: Solving modern SMB enumeration puzzles
- **Red Team Operations**: Reconnaissance with machine-readable output

### Key Concepts

| Concept | Description |
|---------|-------------|
| **JSON Output** | Machine-readable format for automation and parsing |
| **XML Output** | Structured data for enterprise tool integration |
| **Asynchronous Operations** | Parallel enumeration for improved performance |
| **Dependency Check** | Automatic verification of required utilities |
| **Colorized Output** | Enhanced terminal readability |
| **Modular Design** | Extensible architecture for custom enumeration |

## 2. Installation

### System Requirements

- **OS**: Kali Linux, Parrot OS, or any Linux distribution with Python 3.6+
- **Python**: Python 3.6 or higher
- **Dependencies**: smbclient, nmblookup, net, rpcclient
- **Network**: Access to target SMB ports (TCP 445, UDP 137-139)
- **Privileges**: No root required for basic operation

### Installation Methods

**Method 1: Package Manager (Recommended)**

```bash
sudo apt update
sudo apt install enum4linux-ng
```

**Method 2: From Source**

```bash
git clone https://github.com/cddmp/enum4linux-ng.git
cd enum4linux-ng
pip install -r requirements.txt
chmod +x enum4linux-ng.py
sudo cp enum4linux-ng.py /usr/local/bin/enum4linux-ng
```

**Method 3: pip Installation**

```bash
pip install enum4linux-ng
```

**Method 4: Kali Linux Default**

```bash
# Pre-installed on recent Kali Linux versions
which enum4linux-ng
enum4linux-ng --help
```

### Configuration

enum4linux-ng reads configuration from command-line arguments. No configuration file is required.

**Verify Dependencies**

```bash
# Check Python version
python3 --version

# Verify SMB utilities
which smbclient nmblookup rpcclient

# Check enum4linux-ng installation
enum4linux-ng --check-deps
```

### Verify Installation

```bash
enum4linux-ng --help
enum4linux-ng --version
```

## 3. Core Concepts

### How enum4linux-ng Works

enum4linux-ng operates by invoking multiple SMB utilities in parallel, parsing their output, and presenting consolidated results in configurable formats. The tool leverages Python's asynchronous capabilities for improved performance:

1. **Dependency Check**: Validates required utilities are installed
2. **Target Resolution**: Resolves hostname via DNS or NetBIOS
3. **SMB Negotiation**: Establishes SMB connection to target
4. **Parallel Enumeration**: Queries SAMR, LSA, and SMB services simultaneously
5. **Data Aggregation**: Combines results from multiple sources
6. **Output Generation**: Formats results as text, JSON, or XML

### SMB Protocol Stack

```
┌─────────────────────────────────────────────────────┐
│                 enum4linux-ng Architecture           │
├─────────────────────────────────────────────────────┤
│  Output Layer:  Text | JSON | XML                   │
├─────────────────────────────────────────────────────┤
│  Parser Layer:  Output normalization and validation │
├─────────────────────────────────────────────────────┤
│  Enumeration:   Users | Groups | Shares | Policies  │
├─────────────────────────────────────────────────────┤
│  SMB Layer:     smbclient | rpcclient | net         │
├─────────────────────────────────────────────────────┤
│  Network:       TCP 445 | UDP 137-139               │
└─────────────────────────────────────────────────────┘
```

### JSON Output Structure

```json
{
  "target": "192.168.1.100",
  "hostname": "SERVER01",
  "os": "Windows Server 2019",
  "shares": [
    {
      "name": "C$",
      "type": "Disk",
      "comment": "Default share"
    }
  ],
  "users": [
    {
      "name": "Administrator",
      "rid": 500,
      "flags": "U"
    }
  ],
  "groups": [...],
  "policies": {...}
}
```

### XML Output Structure

```xml
<?xml version="1.0" encoding="UTF-8"?>
<enum4linux-ng>
  <target>192.168.1.100</target>
  <hostname>SERVER01</hostname>
  <shares>
    <share>
      <name>C$</name>
      <type>Disk</type>
    </share>
  </shares>
</enum4linux-ng>
```

## 4. Command Reference

### Basic Syntax

```bash
enum4linux-ng [options] <target>
```

### Target Specification

| Option | Description | Example |
|--------|-------------|---------|
| `<ip>` | Direct IP address | `enum4linux-ng 192.168.1.100` |
| `<hostname>` | Resolvable hostname | `enum4linux-ng dc01.local` |
| `<url>` | SMB URL | `enum4linux-ng smb://192.168.1.100` |

### Enumeration Flags

| Flag | Description | Output |
|------|-------------|--------|
| `-A` | All enumeration options | Complete enumeration |
| `-U` | User enumeration | User account list |
| `-S` | Share enumeration | SMB share list |
| `-G` | Group enumeration | Group memberships |
| `-P` | Password policy | Password complexity rules |
| `-O` | OS information | Operating system details |
| `-L` | NetBIOS/LM info | NetBIOS name table |
| `-I` | NBT info | NetBIOS over TCP/IP data |
| `-d` | Domain info | Domain controller details |
| `-r` | RID cycling | Brute-force user RIDs |
| `-R` | Remote SAM | Remote SAM database query |
| `-N` | No NetBIOS | Skip NetBIOS enumeration |
| `-F` | Fingerprint | OS fingerprinting only |

### Authentication Options

| Flag | Description | Example |
|------|-------------|---------|
| `-u <user>` | Username | `-u administrator` |
| `-p <pass>` | Password | `-p Password1` |
| `-d <domain>` | Domain name | `-d WORKGROUP` |
| `-k` | Kerberos authentication | Use Kerberos tickets |

### Output Options

| Flag | Description | Example |
|------|-------------|---------|
| `-o <file>` | Output file | `-o results.json` |
| `-O <format>` | Output format | `-O json` (text/json/xml) |
| `-C` | Colorized output | `-C` (default: enabled) |
| `-c` | No color | `-c` |
| `-q` | Quiet mode | `-q` |
| `-v` | Verbose output | `-v` |

### Common Flag Combinations

```bash
# Full enumeration with JSON output
enum4linux-ng -A -O json 192.168.1.100

# Authenticated enumeration
enum4linux-ng -u admin -p password 192.168.1.100

# Save results to XML
enum4linux-ng -A -o results.xml -O xml 192.168.1.100

# Domain enumeration
enum4linux-ng -d CORP -u user -p pass 192.168.1.100
```

## 5. Usage Examples

### Basic Examples

**Example 1: Simple Enumeration**

```bash
enum4linux-ng 192.168.1.100
```

**Example 2: List Shares Only**

```bash
enum4linux-ng -S 192.168.1.100
```

**Example 3: Enumerate Users**

```bash
enum4linux-ng -U 192.168.1.100
```

**Example 4: Get Password Policy**

```bash
enum4linux-ng -P 192.168.1.100
```

### Intermediate Examples

**Example 5: Authenticated Enumeration**

```bash
enum4linux-ng -u administrator -p P@ssw0rd 192.168.1.100
```

**Example 6: Domain Enumeration**

```bash
enum4linux-ng -d CORP -u john.doe -p Secret123 192.168.1.100
```

**Example 7: Full Enumeration with JSON Output**

```bash
enum4linux-ng -A -O json 192.168.1.100
```

**Example 8: Save Results to File**

```bash
enum4linux-ng -A -o enum_results.json -O json 192.168.1.100
```

### Advanced Examples

**Example 9: NULL Session Enumeration**

```bash
enum4linux-ng -U 192.168.1.100
# NULL session attempts to enumerate users without authentication
```

**Example 10: RID Cycling Attack**

```bash
enum4linux-ng -r -u administrator -p password 192.168.1.100
# Enumerates users by cycling through RIDs
```

**Example 11: Skip NetBIOS Enumeration**

```bash
enum4linux-ng -N -A 192.168.1.100
# Faster enumeration, skips NetBIOS queries
```

**Example 12: Network Scan with JSON Processing**

```bash
for ip in $(seq 1 254); do
  enum4linux-ng -A -O json 192.168.1.$ip > "enum_${ip}.json" 2>/dev/null
done
```

### Real-World Examples

**Example 13: Pre-Engagement Reconnaissance**

```bash
# Map the network
nmap -p 445,139 --open 192.168.1.0/24 -oG smb_hosts.txt

# Enumerate discovered hosts with JSON output
cat smb_hosts.txt | grep "445/open" | awk '{print $2}' | while read host; do
  echo "=== Enumerating $host ==="
  enum4linux-ng -A -O json "${host}_enum.json"
done
```

**Example 14: Active Directory Enumeration**

```bash
enum4linux-ng -d domain.local -u user -p password -A 192.168.1.100
```

**Example 15: Guest Access Testing**

```bash
enum4linux-ng -u guest -p "" 192.168.1.100
```

**Example 16: Compliance Audit Script**

```bash
#!/bin/bash
for host in $(cat targets.txt); do
  echo "=== $host ===" | tee -a audit.log
  enum4linux-ng -P -O json "$host" | tee -a audit.log
done
```

## 6. Advanced Techniques

### Scripting with enum4linux-ng

**Python Script for Automated Enumeration**

```python
#!/usr/bin/env python3
import subprocess
import json
from pathlib import Path

def enum4linux_scan(target, output_dir="enum_results"):
    """Scan target with enum4linux-ng"""
    Path(output_dir).mkdir(exist_ok=True)
    
    cmd = [
        "enum4linux-ng",
        "-A", "-O", "json",
        "-o", f"{output_dir}/{target}.json",
        target
    ]
    
    result = subprocess.run(cmd, capture_output=True, text=True)
    return result.returncode

def parse_results(json_file):
    """Parse enum4linux-ng JSON output"""
    with open(json_file) as f:
        return json.load(f)

# Usage
for ip in range(1, 255):
    target = f"192.168.1.{ip}"
    enum4linux_scan(target)
    results = parse_results(f"enum_results/{target}.json")
    print(f"{target}: {len(results.get('shares', []))} shares")
```

**Bash Script for Batch Processing**

```bash
#!/bin/bash
# batch_enum.sh

TARGETS=$1
OUTPUT_DIR="enum_$(date +%Y%m%d)"
FORMAT="json"

mkdir -p $OUTPUT_DIR

while read target; do
  echo "[*] Enumerating $target"
  
  # Full enumeration
  enum4linux-ng -A -O $FORMAT -o "$OUTPUT_DIR/${target}.$FORMAT" $target
  
  # Extract summary
  echo "=== $target ===" >> "$OUTPUT_DIR/summary.txt"
  enum4linux-ng -A $target | grep -E "(Hostname|OS|Shares)" >> "$OUTPUT_DIR/summary.txt"
  
  echo "[+] $target enumeration complete"
done < $TARGETS
```

### Chaining with Other Tools

**Reconnaissance Pipeline**

```bash
# Step 1: Discover SMB hosts
nmap -p 445 --open 192.168.1.0/24 | grep "report" | awk '{print $NF}' > smb_hosts.txt

# Step 2: Enumerate with enum4linux-ng
while read host; do
  enum4linux-ng -A -O json $host > "${host}_enum4linux-ng.json"
done < smb_hosts.txt

# Step 3: Deep enumeration with smbmap
while read host; do
  smbmap -H $host -R --json > "${host}_smbmap.json"
done < smb_hosts.txt

# Step 4: Parse JSON results
python3 -c "
import json, glob
for f in glob.glob('*_enum4linux-ng.json'):
    with open(f) as fh:
        data = json.load(fh)
        print(f\"{data.get('target')}: {len(data.get('users', []))} users\")
"
```

**Output Parsing Pipeline**

```bash
enum4linux-ng -A -O json 192.168.1.100 | \
  jq '.shares[].name' | \
  tee shares.json
```

### Configuration Tuning

**Timeout Optimization**

```bash
# Reduce timeout for faster scanning
enum4linux-ng -A 192.168.1.100 2>/dev/null | tee output.txt
```

**Performance Tuning**

```bash
# Skip NetBIOS for faster enumeration
enum4linux-ng -N -A 192.168.1.100

# Quiet mode for automation
enum4linux-ng -A -q 192.168.1.100
```

### Automation Patterns

**Integration with Metasploit**

```bash
# Export results for Metasploit
enum4linux-ng -A -O json 192.168.1.100 | \
  msfconsole -q -x "db_import -; hosts; exit"
```

**Integration with Ansible**

```yaml
- name: Enumerate SMB hosts
  hosts: smb_targets
  tasks:
    - name: Run enum4linux-ng
      command: enum4linux-ng -A -O json {{ inventory_hostname }}
      register: enum_results
    
    - name: Save results
      copy:
        content: "{{ enum_results.stdout }}"
        dest: "/tmp/{{ inventory_hostname }}_enum.json"
```

## 7. Detection & Defense

### Detection Signatures

**Network-Based Detection**

```
# Snort rule for enum4linux-ng activity
alert tcp any any -> $HOME_NET 445 (
  msg:"NETBIOS SMB enum4linux-ng enumeration";
  content:"|FF|SMB"; depth:4;
  content:"|73 00 00 00|"; offset:36; depth:4;
  sid:1000002; rev:1;
)
```

**Windows Event Log Analysis**

```powershell
# Event ID 4624 - Successful logon
# Look for NULL session attempts
Get-WinEvent -FilterHashtable @{
    LogName='Security'
    Id=4624
    Data='NT AUTHORITY\ANONYMOUS LOGON'
} | Select TimeCreated, Message
```

**SMB Audit Logging**

```bash
# Enable SMB audit logging on Linux
# /etc/samba/smb.conf
[global]
   vfs objects = full_audit
   full_audit:prefix = %u|%I|%S
   full_audit:success = connect opendir
   full_audit:failure = none
   full_audit:facility = local5
   full_audit:priority = notice
```

### Defense Strategies

**1. Disable NULL Sessions**

```bash
# Windows Registry
reg add HKLM\SYSTEM\CurrentControlSet\Services\LanManServer\Parameters /v RestrictNullSessAccess /t REG_DWORD /d 1 /f

# Samba Configuration
[global]
   restrict anonymous = 2
```

**2. Implement SMB Signing**

```bash
# Samba Configuration
[global]
   client signing = mandatory
   server signing = mandatory
```

**3. Restrict SMB Access**

```bash
# Firewall rules
iptables -A INPUT -p tcp --dport 445 -s 192.168.1.0/24 -j ACCEPT
iptables -A INPUT -p tcp --dport 445 -j DROP
```

**4. Monitor Enumeration Activity**

```bash
# Real-time monitoring
tail -f /var/log/samba/log.* | grep -i "session setup"
```

### IDS/IPS Signatures

```bash
# Suricata rule
alert smb any any -> any any (
  msg:"SMB Enumeration detected";
  flow:to_server,established;
  content:"SMB";
  content:"|FF|SMB";
  sid:2000002; rev:1;
)
```

### Log Analysis

```bash
# Parse Samba logs for enumeration attempts
grep -E "(connect from|session setup)" /var/log/samba/log.* | \
  awk '{print $1, $4}' | sort | uniq -c | sort -rn
```

## 8. Troubleshooting & FAQ

### Common Issues

**Issue: "Python version not supported"**

```bash
# Cause: Python 2.x or old Python 3.x
# Solution: Install Python 3.6+
python3 --version
sudo apt install python3
```

**Issue: "ModuleNotFoundError: No module named '...'**

```bash
# Cause: Missing Python dependencies
# Solution: Install requirements
pip3 install -r requirements.txt
```

**Issue: "Connection refused"**

```bash
# Cause: SMB service not running or blocked
# Solution: Verify SMB service is active
systemctl status smbd
nmap -p 445 192.168.1.100
```

**Issue: "NT_STATUS_LOGON_FAILURE"**

```bash
# Cause: Invalid credentials
# Solution: Verify username and password
enum4linux-ng -u administrator -p 'P@ssw0rd' 192.168.1.100
```

**Issue: JSON parsing errors**

```bash
# Cause: Invalid JSON output
# Solution: Validate JSON
enum4linux-ng -A -O json 192.168.1.100 | python3 -m json.tool
```

### Frequently Asked Questions

**Q: Is enum4linux-ng legal to use?**
A: Only on systems you own or have explicit written authorization to test.

**Q: Does enum4linux-ng support SMB2/3?**
A: Yes, with improved compatibility compared to the original.

**Q: Can enum4linux-ng enumerate Active Directory?**
A: Yes, with valid domain credentials. Use `-d` flag for domain specification.

**Q: How does enum4linux-ng differ from enum4linux?**
A: enum4linux-ng adds JSON/XML output, better error handling, and Python-based architecture.

**Q: Can I use Kerberos authentication?**
A: Yes, use the `-k` flag with valid Kerberos tickets.

### Performance Optimization

```bash
# Parallel enumeration
parallel enum4linux-ng -A -O json {} ::: 192.168.1.{1..254}

# Timeout control
timeout 30 enum4linux-ng -A 192.168.1.100
```

### Exit Codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | Error in arguments |
| 2 | Connection failed |
| 3 | Authentication failed |
| 4 | Enumeration partial |

## Resources

### Official Documentation

- [enum4linux-ng GitHub](https://github.com/cddmp/enum4linux-ng)
- [Kali Linux Tools](https://www.kali.org/tools/enum4linux-ng/)
- [Samba Documentation](https://www.samba.org/samba/docs/)

### Related Tools

- **enum4linux**: Classic bash-based SMB enumeration
- **smbclient**: Samba SMB client for direct share access
- **smbmap**: SMB share enumeration and file operations
- **rpcclient**: Low-level RPC enumeration

### Further Reading

- [SMB Protocol Specification (MS-SMB2)](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-smb2)
- [NetBIOS Over TCP/IP (RFC 1001/1002)](https://tools.ietf.org/html/rfc1001)
- [Python Subprocess Documentation](https://docs.python.org/3/library/subprocess.html)
