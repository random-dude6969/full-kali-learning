---
title: "enum4linux - SMB/Samba Enumeration Tool"
description: "Comprehensive Windows/Samba system enumeration via SMB protocol for discovering users, groups, shares, policies, and OS information"
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
  - discovery
install_command: "sudo apt install enum4linux"
version: "0.9.9"
github_repo: "https://github.com/cddmp/enum4linux"
kali_tools_page: "https://www.kali.org/tools/enum4linux/"
difficulty_level: beginner
requires_root: false
gui_available: false
related_tools:
  - enum4linux-ng
  - smbclient
  - smbmap
  - rpcclient
  - nmblookup
---

# enum4linux - SMB/Samba Enumeration Tool

## 1. Introduction

### What is enum4linux?

enum4linux is a command-line tool designed for enumerating information from Windows and Samba systems using the SMB (Server Message Block) protocol. The tool orchestrates multiple underlying utilities—`net`, `smbclient`, `rpcclient`, and `nmblookup`—to extract comprehensive data about target systems, including user accounts, group memberships, share listings, password policies, OS details, and security configurations.

### History

Developed as a bash script, enum4linux became the de facto standard for SMB enumeration during the early 2000s penetration testing era. The tool was created to simplify the complex process of enumerating Windows systems by wrapping multiple SMB utilities into a single, cohesive interface. While its original development has slowed, it remains widely deployed in security distributions like Kali Linux and Parrot OS.

### When to Use enum4linux

- **Penetration Testing**: Initial enumeration phase to map attack surface
- **Security Auditing**: Verify SMB configurations and access controls
- **Red Team Operations**: Reconnaissance for Active Directory environments
- **CTF Challenges**: Solving SMB-related enumeration puzzles
- **Incident Response**: Identifying compromised SMB configurations

### Key Concepts

| Concept | Description |
|---------|-------------|
| **SMB Protocol** | Application-layer protocol for sharing resources between networked computers |
| **NetBIOS** | Session-layer service enabling communication between applications on different computers |
| **SAMR** | Security Account Manager Remote interface for user/group enumeration |
| **LSA** | Local Security Authority interface for policy and account information |
| **NULL Session** | Unauthenticated connection exploiting default permissions |
| **Guest Account** | Low-privilege account accessible without password in many configurations |

## 2. Installation

### System Requirements

- **OS**: Kali Linux, Parrot OS, or any Debian-based distribution
- **Dependencies**: smbclient, nmblookup, net (samba-common-bin), rpcclient
- **Network**: Access to target SMB ports (TCP 445, UDP 137-139)
- **Privileges**: No root required for basic operation; root may be needed for raw NetBIOS queries

### Installation Methods

**Method 1: Package Manager (Recommended)**

```bash
sudo apt update
sudo apt install enum4linux
```

**Method 2: From Source**

```bash
git clone https://github.com/cddmp/enum4linux.git
cd enum4linux
chmod +x enum4linux
sudo cp enum4linux /usr/local/bin/
```

**Method 3: Kali Linux Default**

```bash
# Pre-installed on Kali Linux
which enum4linux
enum4linux --version
```

### Configuration

enum4linux requires no configuration file. Dependencies must be installed:

```bash
# Verify all dependencies
which smbclient nmblookup rpcclient
dpkg -l | grep samba
```

### Verify Installation

```bash
enum4linux --help
enum4linux -v
```

## 3. Core Concepts

### How enum4linux Works

enum4linux operates by invoking multiple SMB utilities in sequence, parsing their output, and presenting consolidated results. The tool follows a systematic enumeration workflow:

1. **NetBIOS Discovery**: Resolves target hostname via UDP 137 (nmblookup)
2. **OS Fingerprinting**: Identifies operating system version via SMB negotiation
3. **Share Enumeration**: Lists available SMB shares (smbclient)
4. **User Enumeration**: Extracts user accounts via SAMR (rpcclient)
5. **Group Enumeration**: Retrieves group memberships via LSA (rpcclient)
6. **Policy Enumeration**: Dumps password policies and security settings

### SMB Protocol Fundamentals

```
┌─────────────────────────────────────────────────────┐
│                    SMB Protocol Stack                │
├─────────────────────────────────────────────────────┤
│  Application Layer:  SMB Commands (Negotiate, etc.) │
├─────────────────────────────────────────────────────┤
│  Session Layer:      NetBIOS Session Service        │
├─────────────────────────────────────────────────────┤
│  Transport Layer:    TCP (445) / UDP (137-139)      │
├─────────────────────────────────────────────────────┤
│  Network Layer:      IP                              │
└─────────────────────────────────────────────────────┘
```

### NetBIOS Name Resolution

NetBIOS names are 15-character identifiers with a 16th character indicating the service type:

| Suffix | Service | Description |
|--------|---------|-------------|
| `\x00` | Workstation | Client service |
| `\x20` | Server | File server service |
| `\x03` | Messenger | Messaging service |
| `\x1B` | Domain Controller | Primary domain controller |
| `\x1C` | Domain Group | Domain controller group |

### SAMR (Security Account Manager Remote)

SAMR provides remote access to the Security Account Manager database. enum4linux queries SAMR to:

- Enumerate user accounts
- Retrieve group memberships
- Extract user attributes (RID, flags, description)
- Identify disabled/enabled accounts

### LSA (Local Security Authority)

LSA manages local security policy. enum4linux queries LSA for:

- Account rights and privileges
- Audit policies
- Trust relationships
- Domain information

## 4. Command Reference

### Basic Syntax

```bash
enum4linux [options] <target>
```

### Target Specification

| Option | Description | Example |
|--------|-------------|---------|
| `<ip>` | Direct IP address | `enum4linux 192.168.1.100` |
| `<hostname>` | Resolvable hostname | `enum4linux dc01.local` |
| `<ip>/<mask>` | CIDR notation | `enum4linux 192.168.1.0/24` (not supported, use for loop) |

### Enumeration Flags

| Flag | Description | Output |
|------|-------------|--------|
| `-a` | All enumeration options | Complete enumeration |
| `-U` | User enumeration | User account list |
| `-S` | Share enumeration | SMB share list |
| `-G` | Group enumeration | Group memberships |
| `-P` | Password policy | Password complexity rules |
| `-O` | OS information | Operating system details |
| `-L` | NetBIOS/LM info | NetBIOS name table |
| `-l` | License information | Windows license details |
| `-I` | NBT info | NetBIOS over TCP/IP data |
| `-d` | Domain info | Domain controller details |
| `-r` | RID cycling | Brute-force user RIDs |
| `-c` | Command to execute | Custom smbclient commands |
| `-k` | Kerberos authentication | Use Kerberos tickets |

### Authentication Options

| Flag | Description | Example |
|------|-------------|---------|
| `-u <user>` | Username | `-u administrator` |
| `-p <pass>` | Password | `-p Password1` |
| `-U <user:pass>` | Combined credentials | `-U admin:password` |
| `-d <domain>` | Domain name | `-d WORKGROUP` |

### Output Control

| Flag | Description | Example |
|------|-------------|---------|
| `-o` | Output file | `-o results.txt` |
| `-n` | No DNS resolution | `-n` |
| `-v` | Verbose output | `-v` |

### Common Flag Combinations

```bash
# Full enumeration
enum4linux -a 192.168.1.100

# Authenticated enumeration
enum4linux -u admin -p password 192.168.1.100

# Share-specific enumeration
enum4linux -S -u admin -p password 192.168.1.100

# Domain enumeration
enum4linux -d WORKGROUP -u admin -p password 192.168.1.100
```

## 5. Usage Examples

### Basic Examples

**Example 1: Simple Enumeration**

```bash
enum4linux 192.168.1.100
```

**Example 2: List Shares Only**

```bash
enum4linux -S 192.168.1.100
```

**Example 3: Enumerate Users**

```bash
enum4linux -U 192.168.1.100
```

**Example 4: Get Password Policy**

```bash
enum4linux -P 192.168.1.100
```

### Intermediate Examples

**Example 5: Authenticated Enumeration**

```bash
enum4linux -u administrator -p P@ssw0rd 192.168.1.100
```

**Example 6: Domain Enumeration**

```bash
enum4linux -d CORP -u john.doe -p Secret123 192.168.1.100
```

**Example 7: Full Enumeration with Verbose Output**

```bash
enum4linux -a -v 192.168.1.100
```

**Example 8: Save Results to File**

```bash
enum4linux -a -o enum_results.txt 192.168.1.100
```

### Advanced Examples

**Example 9: NULL Session Enumeration**

```bash
enum4linux -U 192.168.1.100
# NULL session attempts to enumerate users without authentication
```

**Example 10: RID Cycling Attack**

```bash
enum4linux -r -u administrator -p password 192.168.1.100
# Enumerates users by cycling through RIDs
```

**Example 11: Execute smbclient Commands**

```bash
enum4linux -c "ls;quit" -u admin -p password 192.168.1.100
```

**Example 12: Network Scan for SMB Hosts**

```bash
for ip in $(seq 1 254); do
  enum4linux -S 192.168.1.$ip 2>/dev/null | grep "Sharename" && echo "192.168.1.$ip has shares"
done
```

### Real-World Examples

**Example 13: Pre-Engagement Reconnaissance**

```bash
# Map the network
nmap -p 445,139 --open 192.168.1.0/24 -oG smb_hosts.txt

# Enumerate discovered hosts
cat smb_hosts.txt | grep "445/open" | awk '{print $2}' | while read host; do
  echo "=== Enumerating $host ==="
  enum4linux -a $host > "${host}_enum.txt"
done
```

**Example 14: Active Directory Enumeration**

```bash
enum4linux -d domain.local -u user -p password -a 192.168.1.100
```

**Example 15: Guest Access Testing**

```bash
enum4linux -u guest -p "" 192.168.1.100
```

**Example 16: Compliance Audit Script**

```bash
#!/bin/bash
for host in $(cat targets.txt); do
  echo "=== $host ===" | tee -a audit.log
  enum4linux -P $host | tee -a audit.log
done
```

## 6. Advanced Techniques

### Scripting with enum4linux

**Bash Script for Automated Enumeration**

```bash
#!/bin/bash
# smb_enum_script.sh

TARGETS=$1
OUTPUT_DIR="enum_$(date +%Y%m%d)"

mkdir -p $OUTPUT_DIR

for target in $(cat $TARGETS); do
  echo "[*] Enumerating $target"
  
  # Basic enumeration
  enum4linux -a $target > "$OUTPUT_DIR/${target}_full.txt" 2>&1
  
  # Extract users
  grep "User:" "$OUTPUT_DIR/${target}_full.txt" > "$OUTPUT_DIR/${target}_users.txt"
  
  # Extract shares
  grep "Sharename" "$OUTPUT_DIR/${target}_full.txt" > "$OUTPUT_DIR/${target}_shares.txt"
  
  echo "[+] $target enumeration complete"
done
```

### Chaining with Other Tools

**Reconnaissance Pipeline**

```bash
# Step 1: Discover SMB hosts
nmap -p 445 --open 192.168.1.0/24 | grep "report" | awk '{print $NF}' > smb_hosts.txt

# Step 2: Enumerate with enum4linux
while read host; do
  enum4linux -a $host > "${host}_enum4linux.txt"
done < smb_hosts.txt

# Step 3: Deep enumeration with smbmap
while read host; do
  smbmap -H $host -R > "${host}_smbmap.txt"
done < smb_hosts.txt

# Step 4: Connect to shares with smbclient
while read host; do
  smbclient -L $host -N > "${host}_smbclient.txt"
done < smb_hosts.txt
```

**Output Parsing Pipeline**

```bash
enum4linux -a 192.168.1.100 | \
  grep -E "(User:|Sharename:|Password)" | \
  tee parsed_output.txt
```

### Configuration Tuning

**Timeout Optimization**

```bash
# Reduce timeout for faster scanning
enum4linux -a 192.168.1.100 2>/dev/null | tee output.txt
```

**SMB Version Targeting**

```bash
# Force SMB1 compatibility
enum4linux -a --port=445 192.168.1.100
```

### Automation Patterns

**Python Integration**

```python
import subprocess

def enum4linux_scan(target):
    cmd = f"enum4linux -a {target}"
    result = subprocess.run(cmd, shell=True, capture_output=True, text=True)
    return result.stdout

def extract_users(output):
    users = []
    for line in output.split('\n'):
        if 'User:' in line:
            users.append(line.split(':')[1].strip())
    return users

# Usage
output = enum4linux_scan("192.168.1.100")
users = extract_users(output)
print(f"Found users: {users}")
```

## 7. Detection & Defense

### Detection Signatures

**Network-Based Detection**

```
# Snort rule for enum4linux activity
alert tcp any any -> $HOME_NET 445 (
  msg:"NETBIOS SMB enum4linux enumeration";
  content:"|FF|SMB"; depth:4;
  content:"|73 00 00 00|"; offset:36; depth:4;
  sid:1000001; rev:1;
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

**2. Rename Guest Account**

```bash
# Samba Configuration
[global]
   guest account = nobody
   map to guest = never
```

**3. Implement SMB Signing**

```bash
# Windows Group Policy
# Computer Configuration -> Windows Settings -> Security Settings
# -> Local Policies -> Security Options
# Microsoft network server: Digitally sign communications (always)

# Samba Configuration
[global]
   client signing = mandatory
   server signing = mandatory
```

**4. Restrict SMB Access**

```bash
# Firewall rules
iptables -A INPUT -p tcp --dport 445 -s 192.168.1.0/24 -j ACCEPT
iptables -A INPUT -p tcp --dport 445 -j DROP
```

**5. Monitoring and Alerting**

```bash
# Real-time monitoring with inotifywait
inotifywait -m -r /etc/samba | while read path action file; do
  echo "$(date): $path$file modified" >> /var/log/smb_changes.log
done
```

### IDS/IPS Signatures

```bash
# Suricata rule
alert smb any any -> any any (
  msg:"SMB Enumeration detected";
  flow:to_server,established;
  content:"SMB";
  content:"|FF|SMB";
  sid:2000001; rev:1;
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
enum4linux -u administrator -p 'P@ssw0rd' 192.168.1.100
# Note: Use single quotes for special characters
```

**Issue: "NT_STATUS_ACCESS_DENIED"**

```bash
# Cause: Insufficient permissions or restricted enumeration
# Solution: Try authenticated enumeration
enum4linux -u domain_user -p password 192.168.1.100
```

**Issue: No users enumerated**

```bash
# Cause: Guest access restricted or SMB1 disabled
# Solution: Force SMB1 or try authenticated enumeration
enum4linux -S -u guest -p "" 192.168.1.100
```

**Issue: Slow enumeration**

```bash
# Cause: Network latency or firewall delays
# Solution: Increase timeout or reduce scope
enum4linux -a 192.168.1.100 2>/dev/null &
```

### Frequently Asked Questions

**Q: Is enum4linux legal to use?**
A: Only on systems you own or have explicit written authorization to test.

**Q: Does enum4linux detect SMB2/3?**
A: Yes, but enumeration capabilities vary by SMB version. SMB1 provides more enumeration data.

**Q: Can enum4linux enumerate Active Directory?**
A: Yes, with valid domain credentials. Use `-d` flag for domain specification.

**Q: How does enum4linux differ from smbmap?**
A: enum4linux focuses on enumeration metadata; smbmap focuses on share access and file operations.

**Q: What if NULL sessions are blocked?**
A: Provide valid credentials with `-u` and `-p` flags.

### Performance Optimization

```bash
# Parallel enumeration of multiple hosts
parallel enum4linux -a {} ::: 192.168.1.{1..254}

# Timeout control
timeout 30 enum4linux -a 192.168.1.100
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

- [enum4linux GitHub](https://github.com/cddmp/enum4linux)
- [Kali Linux Tools](https://www.kali.org/tools/enum4linux/)
- [Samba Documentation](https://www.samba.org/samba/docs/)

### Related Tools

- **enum4linux-ng**: Modern Python rewrite with JSON output
- **smbclient**: Samba SMB client for direct share access
- **smbmap**: SMB share enumeration and file operations
- **rpcclient**: Low-level RPC enumeration
- **nmblookup**: NetBIOS name resolution

### Further Reading

- [SMB Protocol Specification (MS-SMB2)](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-smb2)
- [NetBIOS Over TCP/IP (RFC 1001/1002)](https://tools.ietf.org/html/rfc1001)
- [Samba Security Configuration](https://www.samba.org/samba/docs/man/smb.conf/)
