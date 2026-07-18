---
title: "nbtscan - NetBIOS Name Service Scanner"
description: "Scans networks for NetBIOS name information via NBNS (UDP 137) to identify hosts, users, shares, and OS details"
mitre_tactic: discovery
mitre_tactic_id: TA0007
mitre_subcategory: network_share_discovery
categories:
  - Network Enumeration
  - NetBIOS Enumeration
  - Host Discovery
  - SMB Enumeration
tags:
  - netbios
  - nbt
  - nbns
  - windows
  - samba
  - discovery
install_command: "sudo apt install nbtscan"
version: "1.7.2"
github_repo: "https://github.com/rescador/nbtscan"
kali_tools_page: "https://www.kali.org/tools/nbtscan/"
difficulty_level: beginner
requires_root: false
gui_available: false
related_tools:
  - smbclient
  - enum4linux
  - nmblookup
  - smbmap
---

# nbtscan - NetBIOS Name Service Scanner

## 1. Introduction

### What is nbtscan?

nbtscan is a command-line tool that scans networks for NetBIOS name information using the NetBIOS Name Service (NBNS) protocol over UDP port 137. The tool resolves NetBIOS names to IP addresses and extracts valuable information including hostnames, MAC addresses, logged-in users, operating systems, and shared resources.

### History

nbtscan was originally developed by Sergey Gladysh in the late 1990s as a lightweight NetBIOS scanner. The tool became widely adopted in security assessments due to its speed and efficiency in discovering Windows systems on networks. The current maintained version is available on GitHub, with improvements for modern network environments.

### When to Use nbtscan

- **Network Discovery**: Rapidly identify Windows/Samba systems on a subnet
- **Host Enumeration**: Map network topology with NetBIOS information
- **Share Discovery**: Identify systems with accessible SMB shares
- **User Tracking**: Discover logged-in users via NetBIOS sessions
- **Pre-Engagement Reconnaissance**: Quick network survey before deeper enumeration
- **Asset Management**: Inventory Windows systems on corporate networks

### Key Concepts

| Concept | Description |
|---------|-------------|
| **NBNS** | NetBIOS Name Service, resolves NetBIOS names to IP addresses |
| **UDP 137** | Standard port for NetBIOS name queries |
| **NetBIOS Name** | 15-character identifier for network resources |
| **MAC Address** | Hardware address included in NetBIOS responses |
| **Workgroup/Domain** | Network grouping information from NetBIOS |
| **Session User** | Currently logged-in user information |

## 2. Installation

### System Requirements

- **OS**: Kali Linux, Parrot OS, or any Linux distribution
- **Network**: Access to target UDP port 137
- **Privileges**: No root required for basic operation
- **Dependencies**: None (standalone binary)

### Installation Methods

**Method 1: Package Manager (Recommended)**

```bash
sudo apt update
sudo apt install nbtscan
```

**Method 2: From Source**

```bash
git clone https://github.com/rescador/nbtscan.git
cd nbtscan
make
sudo cp nbtscan /usr/local/bin/
```

**Method 3: Kali Linux Default**

```bash
# Pre-installed on Kali Linux
which nbtscan
nbtscan --help
```

### Configuration

nbtscan requires no configuration. The tool operates directly from command-line arguments.

### Verify Installation

```bash
nbtscan --help
nbtscan -v 192.168.1.100
```

## 3. Core Concepts

### How nbtscan Works

nbtscan sends NetBIOS name queries to UDP port 137 on target hosts and parses the responses to extract information:

1. **Query Construction**: Builds NetBIOS Status Query packets
2. **UDP Transmission**: Sends queries to target UDP 137
3. **Response Parsing**: Extracts data from NetBIOS responses
4. **Data Presentation**: Formats and displays results

### NetBIOS Name Service Protocol

```
┌─────────────────────────────────────────────────────┐
│              NBNS Query/Response Flow                │
├─────────────────────────────────────────────────────┤
│  Client                    Server                    │
│    │                         │                       │
│    │──── Query Request ─────>│  (UDP 137)            │
│    │                         │                       │
│    │<──── Query Response ────│  (Name + IP + MAC)    │
│    │                         │                       │
└─────────────────────────────────────────────────────┘
```

### NetBIOS Name Table

NetBIOS names are 15 characters with a 16th character indicating the service type:

| Suffix | Service | Description |
|--------|---------|-------------|
| `\x00` | Workstation | Client service |
| `\x20` | Server | File server service |
| `\x03` | Messenger | Messaging service |
| `\x1B` | Domain Controller | Primary domain controller |
| `\1C` | Domain Group | Domain controller group |
| `\1E` | Browser Service | Network browser |

### Response Data Fields

| Field | Description | Example |
|-------|-------------|---------|
| IP Address | Target host IP | `192.168.1.100` |
| NetBIOS Name | 15-char identifier | `SERVER01` |
| Server Status | Service availability | `<Server>` |
| MAC Address | Hardware address | `00:1A:2B:3C:4D:5E` |
| User | Logged-in user | `ADMINISTRATOR` |
| Workgroup | Domain/workgroup | `WORKGROUP` |
| OS | Operating system | `Windows 10` |

## 4. Command Reference

### Basic Syntax

```bash
nbtscan [options] <target>
```

### Target Specification

| Option | Description | Example |
|--------|-------------|---------|
| `<ip>` | Single IP | `nbtscan 192.168.1.100` |
| `<ip>/<mask>` | CIDR notation | `nbtscan 192.168.1.0/24` |
| `<start>-<end>` | IP range | `nbtscan 192.168.1.1-192.168.1.254` |
| `-f <file>` | File with targets | `nbtscan -f targets.txt` |

### Scanning Options

| Flag | Description | Example |
|------|-------------|---------|
| `-r <range>` | IP range to scan | `-r 192.168.1.0/24` |
| `-f <file>` | Read targets from file | `-f hosts.txt` |
| `-v` | Verbose output | `-v` |
| `-q` | Quiet mode | `-q` |
| `-n` | No resolution | `-n` |
| `-l` | Local mode | `-l` |

### Output Options

| Flag | Description | Example |
|------|-------------|---------|
| `-o <file>` | Output file | `-o results.txt` |
| `-p` | Parse output | `-p` |
| `-d` | Debug mode | `-d` |

### Common Flag Combinations

```bash
# Scan entire subnet
nbtscan 192.168.1.0/24

# Scan IP range
nbtscan 192.168.1.1-192.168.1.254

# Verbose scan
nbtscan -v 192.168.1.0/24

# Save results
nbtscan -o results.txt 192.168.1.0/24
```

## 5. Usage Examples

### Basic Examples

**Example 1: Scan Single Host**

```bash
nbtscan 192.168.1.100
```

**Example 2: Scan Subnet**

```bash
nbtscan 192.168.1.0/24
```

**Example 3: Scan IP Range**

```bash
nbtscan 192.168.1.1-192.168.1.50
```

**Example 4: Verbose Scan**

```bash
nbtscan -v 192.168.1.0/24
```

### Intermediate Examples

**Example 5: Scan from File**

```bash
nbtscan -f targets.txt
```

**Example 6: Save Results to File**

```bash
nbtscan -o results.txt 192.168.1.0/24
```

**Example 7: Quiet Mode**

```bash
nbtscan -q 192.168.1.0/24
```

**Example 8: No DNS Resolution**

```bash
nbtscan -n 192.168.1.0/24
```

### Advanced Examples

**Example 9: Find Domain Controllers**

```bash
nbtscan -v 192.168.1.0/24 | grep "Domain Controller"
```

**Example 10: Find Active Users**

```bash
nbtscan -v 192.168.1.0/24 | grep -E "User:|<Server>"
```

**Example 11: Extract IP Addresses**

```bash
nbtscan 192.168.1.0/24 | awk '{print $1}' | grep -v "IP"
```

**Example 12: Scan Multiple Subnets**

```bash
for subnet in 192.168.{1,2,3}.0/24; do
  nbtscan $subnet >> all_hosts.txt
done
```

### Real-World Examples

**Example 13: Quick Network Survey**

```bash
# Discover all Windows systems
nbtscan -v 192.168.1.0/24 | tee network_survey.txt

# Count discovered hosts
nbtscan 192.168.1.0/24 | wc -l
```

**Example 14: Pre-Engagement Reconnaissance**

```bash
# Step 1: Quick discovery
nbtscan 192.168.1.0/24 > discovery.txt

# Step 2: Extract IPs
nbtscan 192.168.1.0/24 | awk '{print $1}' | grep -v "^IP" > hosts.txt

# Step 3: Deeper enumeration
while read host; do
  enum4linux -a $host > "${host}_enum.txt"
done < hosts.txt
```

**Example 15: Asset Inventory Script**

```bash
#!/bin/bash
SUBNET=$1
OUTPUT="inventory_$(date +%Y%m%d).csv"

echo "IP,Hostname,MAC,User,Workgroup" > $OUTPUT

nbtscan -v $SUBNET | while read line; do
  # Parse nbtscan output
  ip=$(echo $line | awk '{print $1}')
  hostname=$(echo $line | awk '{print $2}')
  mac=$(echo $line | awk '{print $NF}')
  echo "$ip,$hostname,$mac,," >> $OUTPUT
done

echo "Inventory saved to $OUTPUT"
```

**Example 16: Find Specific Hostnames**

```bash
nbtscan 192.168.1.0/24 | grep -i "server"
nbtscan 192.168.1.0/24 | grep -i "dc"
nbtscan 192.168.1.0/24 | grep -i "sql"
```

## 6. Advanced Techniques

### Scripting with nbtscan

**Python Script for Network Mapping**

```python
#!/usr/bin/env python3
import subprocess
import re

def nbtscan_scan(subnet):
    """Scan subnet with nbtscan"""
    cmd = f"nbtscan -v {subnet}"
    result = subprocess.run(cmd, shell=True, capture_output=True, text=True)
    return result.stdout

def parse_nbtscan(output):
    """Parse nbtscan output"""
    hosts = []
    for line in output.split('\n'):
        if re.match(r'\d+\.\d+\.\d+\.\d+', line):
            parts = line.split()
            hosts.append({
                'ip': parts[0],
                'hostname': parts[1] if len(parts) > 1 else '',
                'mac': parts[-1] if len(parts) > 3 else ''
            })
    return hosts

# Usage
output = nbtscan_scan("192.168.1.0/24")
hosts = parse_nbtscan(output)
print(f"Found {len(hosts)} hosts")
for host in hosts:
    print(f"{host['ip']} - {host['hostname']}")
```

**Bash Script for Automated Discovery**

```bash
#!/bin/bash
# nbtscan_discovery.sh

SUBNET=$1
OUTPUT_DIR="nbtscan_$(date +%Y%m%d)"

mkdir -p $OUTPUT_DIR

# Run nbtscan
nbtscan -v $SUBNET > "$OUTPUT_DIR/full_scan.txt"

# Extract hosts
nbtscan $SUBNET | awk '{print $1}' | grep -v "^IP" > "$OUTPUT_DIR/hosts.txt"

# Count results
HOST_COUNT=$(wc -l < "$OUTPUT_DIR/hosts.txt")
echo "[+] Discovered $HOST_COUNT hosts"

# Categorize by workgroup
nbtscan -v $SUBNET | awk '{print $5}' | sort | uniq -c | sort -rn > "$OUTPUT_DIR/workgroups.txt"
```

### Chaining with Other Tools

**Discovery Pipeline**

```bash
# Step 1: Quick NetBIOS discovery
nbtscan 192.168.1.0/24 > nbtscan_results.txt

# Step 2: Extract IP addresses
nbtscan 192.168.1.0/24 | awk '{print $1}' | grep -v "^IP" > discovered_hosts.txt

# Step 3: Port scan
nmap -p 445,139 -iL discovered_hosts.txt -oN port_scan.txt

# Step 4: Enumerate discovered hosts
while read host; do
  enum4linux -a $host > "${host}_enum.txt"
done < discovered_hosts.txt
```

**Integration with smbclient**

```bash
# Find hosts with shares
nbtscan 192.168.1.0/24 | while read line; do
  ip=$(echo $line | awk '{print $1}')
  if echo $line | grep -q "Server"; then
    echo "=== $ip ==="
    smbclient -L $ip -N
  fi
done
```

### Configuration Tuning

**Timeout Optimization**

```bash
# Faster scanning with reduced timeout
nbtscan -v 192.168.1.0/24 2>/dev/null
```

**Parallel Scanning**

```bash
# Split subnet and scan in parallel
nbtscan 192.168.1.0/25 &
nbtscan 192.168.1.128/25 &
wait
```

### Automation Patterns

**Cron Job for Network Monitoring**

```bash
# Add to crontab
0 */6 * * * nbtscan 192.168.1.0/24 >> /var/log/nbtscan.log
```

**Integration with SIEM**

```bash
# Output JSON for SIEM ingestion
nbtscan -v 192.168.1.0/24 | while read line; do
  ip=$(echo $line | awk '{print $1}')
  hostname=$(echo $line | awk '{print $2}')
  echo "{\"ip\":\"$ip\",\"hostname\":\"$hostname\",\"tool\":\"nbtscan\"}"
done | jq -s '.'
```

## 7. Detection & Defense

### Detection Signatures

**Network-Based Detection**

```
# Snort rule for nbtscan activity
alert udp any any -> $HOME_NET 137 (
  msg:"NETBIOS nbtscan enumeration";
  content:"|00 01 00 00|";
  sid:1000003; rev:1;
)
```

**Windows Event Log Analysis**

```powershell
# Event ID 4624 - Successful logon
# Look for NetBIOS name queries
Get-WinEvent -FilterHashtable @{
    LogName='Security'
    Id=4624
} | Select TimeCreated, Message
```

**Linux Firewall Logs**

```bash
# Log NetBIOS queries
iptables -A INPUT -p udp --dport 137 -j LOG --log-prefix "NBNS Query: "
```

### Defense Strategies

**1. Block NetBIOS Queries**

```bash
# Firewall rules
iptables -A INPUT -p udp --dport 137 -s 192.168.1.0/24 -j ACCEPT
iptables -A INPUT -p udp --dport 137 -j DROP
```

**2. Disable NetBIOS**

```bash
# Windows Registry
reg add "HKLM\SYSTEM\CurrentControlSet\Services\NetBT\Parameters\Interfaces\Tcpip_{GUID}" /v NetbiosOptions /t REG_DWORD /d 2 /f

# Samba Configuration
[global]
   disable netbios = yes
```

**3. Implement NetBIOS Filtering**

```bash
# Disable NetBIOS over TCP/IP
# Network Adapter Settings -> IPv4 -> Advanced -> WINS -> Disable NetBIOS
```

**4. Monitor NetBIOS Activity**

```bash
# Real-time monitoring
tcpdump -i eth0 udp port 137 -n | tee /var/log/nbtscan_monitor.log
```

### IDS/IPS Signatures

```bash
# Suricata rule
alert udp any any -> any 137 (
  msg:"NetBIOS Name Query flood";
  threshold:type both, track by_src, count 50, seconds 60;
  sid:2000003; rev:1;
)
```

### Log Analysis

```bash
# Parse firewall logs for NetBIOS queries
grep "NBNS Query" /var/log/messages | \
  awk '{print $5}' | sort | uniq -c | sort -rn
```

## 8. Troubleshooting & FAQ

### Common Issues

**Issue: "No response from target"**

```bash
# Cause: Firewall blocking UDP 137 or host offline
# Solution: Verify connectivity
ping 192.168.1.100
nmap -sU -p 137 192.168.1.100
```

**Issue: "Permission denied"**

```bash
# Cause: Insufficient privileges for raw sockets
# Solution: Use sudo for some operations
sudo nbtscan 192.168.1.0/24
```

**Issue: No results returned**

```bash
# Cause: NetBIOS disabled on target or wrong subnet
# Solution: Verify target configuration
nbtscan -v 192.168.1.100
```

**Issue: Slow scanning**

```bash
# Cause: Large subnet or network latency
# Solution: Scan smaller ranges
nbtscan 192.168.1.1-192.168.1.50
```

**Issue: "Host not found"**

```bash
# Cause: DNS resolution failure
# Solution: Use IP addresses directly
nbtscan 192.168.1.100
```

### Frequently Asked Questions

**Q: Is nbtscan legal to use?**
A: Only on networks you own or have explicit written authorization to test.

**Q: Does nbtscan work on Linux/Unix targets?**
A: Yes, if Samba is installed and NetBIOS is enabled.

**Q: Can nbtscan discover domain controllers?**
A: Yes, look for `\x1B` suffix in NetBIOS names.

**Q: What is the difference between nbtscan and nmblookup?**
A: nbtscan scans entire subnets; nmblookup queries individual names.

**Q: Does nbtscan support SMB2/3?**
A: nbtscan operates at NetBIOS layer, independent of SMB version.

### Performance Optimization

```bash
# Parallel scanning
nbtscan 192.168.1.0/25 &
nbtscan 192.168.1.128/25 &
wait

# Timeout control
timeout 30 nbtscan 192.168.1.0/24
```

### Exit Codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | Error in arguments |
| 2 | Network error |
| 3 | No hosts found |

## Resources

### Official Documentation

- [nbtscan GitHub](https://github.com/rescador/nbtscan)
- [Kali Linux Tools](https://www.kali.org/tools/nbtscan/)
- [NetBIOS Over TCP/IP (RFC 1001/1002)](https://tools.ietf.org/html/rfc1001)

### Related Tools

- **nmblookup**: Individual NetBIOS name queries
- **smbclient**: Samba SMB client for share access
- **enum4linux**: Comprehensive SMB enumeration
- **smbmap**: SMB share enumeration and file operations

### Further Reading

- [NetBIOS Name Service Protocol](https://docs.microsoft.com/en-us/windows/win32/netbios/netbios-name-service)
- [SMB Protocol Specification](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-smb2)
- [Network Scanner Comparison](https://www.kali.org/tools/)
