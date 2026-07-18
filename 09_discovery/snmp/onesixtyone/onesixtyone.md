---
title: "onesixtyone - Fast SNMP Community String Scanner"
description: "Broadcast-based SNMP community string scanner that rapidly identifies valid community strings across network ranges using efficient packet techniques"
mitre_tactic: "discovery"
mitre_tactic_id: "TA0007"
mitre_subcategory: "snmp"
categories:
  - "snmp"
  - "network-scanning"
  - "enumeration"
tags:
  - "snmp"
  - "community-strings"
  - "scanner"
  - "broadcast"
  - "fast"
install_command: "sudo apt install onesixtyone"
version: "0.12"
github_repo: ""
kali_tools_page: "https://www.kali.org/tools/onesixtyone/"
difficulty_level: "beginner"
requires_root: true
gui_available: false
related_tools:
  - "braa"
  - "snmpwalk"
  - "snmp-check"
  - "snmpget"
---

# onesixtyone - Fast SNMP Community String Scanner

## 1. Introduction

**onesixtyone** is a fast, broadcast-based SNMP community string scanner designed to quickly identify valid community strings on network targets. The tool leverages SNMP's UDP-based nature to efficiently scan multiple hosts simultaneously.

### Key Capabilities

**Broadcast Scanning**: Uses broadcast packets to efficiently discover SNMP-enabled devices across network segments.

**High Speed**: Optimized for rapid scanning with minimal network overhead, capable of testing thousands of combinations quickly.

**Default String Detection**: Includes built-in default community strings commonly found on SNMP devices.

**CSV Output**: Generates structured output for easy integration with other security tools and reporting systems.

### Use Cases

- **Network Discovery**: Rapidly identify all SNMP-enabled devices on a network segment.
- **Security Auditing**: Detect devices using default or weak community strings.
- **Penetration Testing**: Gather initial intelligence about SNMP services during security assessments.
- **Asset Management**: Catalog network devices accessible via SNMP.

### How onesixtyone Works

onesixtyone sends SNMP GetRequest packets with various community strings to target hosts. When a valid community string is used, the target responds with system information. The tool captures these responses and reports successful authentications.

The broadcast-based approach allows onesixtyone to efficiently scan entire network segments by sending packets to broadcast addresses, reducing the number of individual packets required.

## 2. Installation

### Standard Installation

```bash
sudo apt update
sudo apt install onesixtyone
```

### Verify Installation

```bash
onesixtyone -h
```

### Manual Installation from Source

```bash
git clone https://github.com/trailofbits/onesixtyone.git
cd onesixtyone
make
sudo cp onesixtyone /usr/local/bin/
```

### Installation Notes

onesixtyone has minimal dependencies, primarily requiring standard C libraries and network development headers.

## 3. Core Concepts

### SNMP Protocol Basics

**SNMP (Simple Network Management Protocol)**: Application-layer protocol for network device management. Uses UDP ports 161 (agent) and 162 (manager).

**Community Strings**: Act as passwords for SNMP access. Most devices ship with default strings like "public" or "private".

**OIDs (Object Identifiers)**: Hierarchical addressing system for SNMP objects. The system MIB tree starts at 1.3.6.1.2.1.1.

### Scanning Methodology

**Broadcast Discovery**: onesixtyone sends packets to network broadcast addresses to efficiently discover SNMP-enabled devices.

**Response Analysis**: Valid community strings produce SNMP responses containing system information like sysDescr and sysName.

**Rate Limiting**: Built-in throttling prevents network congestion and IDS detection.

### Community String Categories

**Default Strings**: public, private, manager - commonly found on out-of-box devices.

**Manufacturer Defaults**: Some vendors use specific default strings (e.g., "cisco", "hp", "dell").

**Custom Strings**: Organization-specific strings that may be weaker than expected.

## 4. Command Reference

### Basic Syntax

```bash
onesixtyone [OPTIONS] COMMUNITY_FILE TARGET
```

### Core Flags

**-c FILE**: Community string file containing one string per line.

**-d**: Debug mode. Display raw SNMP responses for troubleshooting.

**-w NUM**: Wait time in milliseconds between packets (default: 10).

**-i FILE**: Input file containing target hosts (one per line).

**-o FILE**: Output file for results in CSV format.

**-p PORT**: Specify SNMP port (default: 161).

**-s**: Silent mode. Suppress non-essential output.

### Advanced Flags

**-n NUM**: Maximum number of hosts to scan simultaneously.

**-t NUM**: Timeout in seconds for host responses.

**-r NUM**: Number of retries for unresponsive hosts.

## 5. Examples

### Basic Scan with Default Strings

```bash
onesixtyone -c /usr/share/onesixtyone/community.txt 192.168.1.0/24
```

**Result**: Scans network using built-in community string list.

### Custom Community String File

```bash
onesixtyone -c custom_strings.txt 192.168.1.100
```

**Result**: Tests custom strings against single target.

### Single String Test

```bash
echo "public" > single.txt
onesixtyone -c single.txt 192.168.1.100
```

**Result**: Tests "public" against target host.

### Debug Mode

```bash
onesixtyone -d -c community.txt 192.168.1.100
```

**Result**: Displays detailed SNMP response data.

### CSV Output

```bash
onesixtyone -o results.csv -c community.txt 192.168.1.0/24
```

**Result**: Saves findings in CSV format for analysis.

### Network Range Scan

```bash
onesixtyone -c community.txt 192.168.1.0/24
```

**Result**: Scans all 256 hosts in subnet.

### Silent Mode with Output

```bash
onesixtyone -s -o results.csv -c community.txt 192.168.1.0/24
```

**Result**: Minimal output, structured CSV results.

### Custom Port

```bash
onesixtyone -p 1161 -c community.txt 192.168.1.100
```

**Result**: Scans non-standard SNMP port.

## 6. Advanced Techniques

### Comprehensive Community String List

```bash
cat > comprehensive.txt << 'EOF'
public
private
manager
admin
secret
snmp
community
readonly
readwrite
cisco
hp
dell
ibm
oracle
sun
linux
windows
net-snmp
test
demo
EOF
onesixtyone -c comprehensive.txt 192.168.1.0/24
```

### Target List File

```bash
cat > targets.txt << 'EOF'
192.168.1.1
192.168.1.10
192.168.1.20
192.168.1.30
EOF
onesixtyone -i targets.txt -c community.txt
```

### High-Speed Scanning

```bash
onesixtyone -w 5 -c community.txt 192.168.1.0/24
```

**Result**: Reduced wait time for faster scanning.

### Automation Script

```bash
#!/bin/bash
NETWORK="192.168.1.0/24"
STRINGS="/usr/share/onesixtyone/community.txt"
OUTPUT="scan_$(date +%Y%m%d_%H%M%S).csv"

onesixtyone -o "$OUTPUT" -c "$STRINGS" "$NETWORK"

if [ -s "$OUTPUT" ]; then
  echo "Results saved to $OUTPUT"
  cat "$OUTPUT"
fi
```

### Combining with snmpwalk

```bash
onesixtyone -c community.txt 192.168.1.0/24 | grep -E "^[0-9]" | while read host string; do
  echo "=== Enumerating $host ==="
  snmpwalk -c "$string" -v1 "$host" 1.3.6.1.2.1.1
done
```

## 7. Detection & Defense

### Network Detection

**SNMP Broadcast Monitoring**: Detect broadcast SNMP requests indicating scanning activity.

**Rate Analysis**: Identify rapid SNMP requests exceeding normal management patterns.

**Source IP Tracking**: Monitor for unusual source IPs performing SNMP scans.

### System Detection

**SNMP Logs**: Review SNMP service logs for authentication failures or unusual access patterns.

**Connection Monitoring**: Track unique source IPs connecting to SNMP ports.

### Defense Strategies

**Community String Hardening**: Replace default strings with complex, unique values per device.

**SNMP Filtering**: Implement firewall rules restricting SNMP access to authorized management stations.

**Disable Unused SNMP**: Disable SNMP on devices where network management is not required.

**SNMPv3 Migration**: Upgrade to SNMPv3 with authentication and encryption.

**Network Segmentation**: Isolate management networks from user networks.

## 8. Troubleshooting

### No Results

**Verify Network Connectivity**: Confirm target hosts are reachable.

```bash
ping -c 2 192.168.1.100
```

**Check SNMP Service**: Ensure SNMP is running on target.

```bash
snmpwalk -c public -v1 192.168.1.100 1.3.6.1.2.1.1
```

**Verify Community Strings**: Test with known valid strings.

```bash
echo "public" > test.txt
onesixtyone -c test.txt 192.168.1.100
```

### Slow Performance

**Reduce Wait Time**: Lower -w value for faster scanning.

```bash
onesixtyone -w 5 -c community.txt 192.168.1.0/24
```

**Limit Network Range**: Scan smaller subnets for quicker results.

```bash
onesixtyone -c community.txt 192.168.1.0/24
```

### Permission Denied

**Run as Root**: onesixtyone requires root privileges for raw packet operations.

```bash
sudo onesixtyone -c community.txt 192.168.1.100
```

### Connection Refused

**Firewall Issues**: Check for firewall blocking SNMP traffic.

```bash
iptables -L -n | grep 161
```

**Port Configuration**: Verify target SNMP port with nmap.

```bash
nmap -sU -p 161 192.168.1.100
```

## Resources

- **onesixtyone GitHub**: https://github.com/trailofbits/onesixtyone
- **SNMP Protocol RFC**: https://tools.ietf.org/html/rfc1157
- **Kali Tools Documentation**: https://www.kali.org/tools/onesixtyone/
- **SNMP Security Guide**: https://tools.ietf.org/html/rfc3584

## Related Tools

**braa**: Multi-threaded SNMP scanner with brute-force capabilities for large-scale scanning.

**snmpwalk**: SNMP tree walker for enumerating device information after finding valid strings.

**snmp-check**: Comprehensive SNMP enumeration tool for detailed device information extraction.

**snmpget**: Direct SNMP object retrieval for targeted information gathering.
