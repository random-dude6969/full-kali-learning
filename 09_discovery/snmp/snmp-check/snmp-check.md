---
title: "snmp-check - SNMP Enumeration Tool"
description: "Comprehensive SNMP enumeration tool that walks common OIDs to extract detailed system information including interfaces, routes, processes, and software"
mitre_tactic: "discovery"
mitre_tactic_id: "TA0007"
mitre_subcategory: "snmp"
categories:
  - "snmp"
  - "enumeration"
  - "information-gathering"
tags:
  - "snmp"
  - "enumeration"
  - "oids"
  - "system-info"
  - "detailed"
install_command: "sudo apt install snmp-check"
version: "1.8"
github_repo: ""
kali_tools_page: "https://www.kali.org/tools/snmp-check/"
difficulty_level: "beginner"
requires_root: false
gui_available: false
related_tools:
  - "snmpwalk"
  - "braa"
  - "onesixtyone"
  - "snmpget"
---

# snmp-check - SNMP Enumeration Tool

## 1. Introduction

**snmp-check** is a comprehensive SNMP enumeration tool designed to extract detailed information from SNMP-enabled devices. The tool walks common OID trees to gather system data, network interfaces, routing tables, ARP tables, running processes, and installed software.

### Key Capabilities

**Comprehensive Enumeration**: Walks multiple OID trees to extract a complete profile of SNMP-accessible devices.

**Human-Readable Output**: Presents information in organized, easy-to-read format with clear section headers.

**Multiple SNMP Versions**: Supports SNMPv1, SNMPv2c, and SNMPv3 protocols.

**Extensive OID Coverage**: Queries system, interfaces, routing, ARP, TCP/UDP, processes, storage, and software information.

### Use Cases

- **Penetration Testing**: Extract sensitive system information during security assessments.
- **Network Auditing**: Catalog device configurations and installed software across the network.
- **Asset Management**: Gather detailed inventory of network devices and their capabilities.
- **Compliance Verification**: Verify device configurations meet security standards.
- **Incident Response**: Collect forensic information from compromised or suspect devices.

### How snmp-check Works

snmp-check connects to target SNMP agents and performs a series of GetNext requests to walk specific OID subtrees. Each subtree corresponds to a category of information (system, interfaces, routing, etc.). The tool then formats and displays the collected data in an organized report.

## 2. Installation

### Standard Installation

```bash
sudo apt update
sudo apt install snmp-check
```

### Verify Installation

```bash
snmp-check --version
```

### Dependencies

snmp-check requires the Net::SNMP Perl module:

```bash
sudo apt install libnet-snmp-perl
```

## 3. Core Concepts

### SNMP OIDs

**Object Identifiers (OIDs)**: Hierarchical addressing system for SNMP objects. Each device exposes information through a tree of OIDs.

**System MIB (1.3.6.1.2.1.1)**: Contains system description, uptime, contact, name, and location.

**Interfaces MIB (1.3.6.1.2.1.2)**: Network interface information including names, speeds, and statistics.

**IP MIB (1.3.6.1.2.1.4)**: IP routing table and interface statistics.

**TCP/UDP MIBs**: Active connections and listening services.

### SNMP Versions

**SNMPv1**: Basic authentication with plaintext community strings. Most compatible but least secure.

**SNMPv2c**: Enhanced error handling and bulk operations. Still uses plaintext community strings.

**SNMPv3**: Strongest security with authentication and encryption. Requires additional configuration.

### Enumeration Categories

**System Information**: Device description, uptime, contact, name, location, services.

**Network Interfaces**: Interface names, types, speeds, MAC addresses, IP addresses, statistics.

**Routing Table**: IP routes, destinations, gateways, metrics, interfaces.

**ARP Table**: IP-to-MAC address mappings for local network.

**TCP Connections**: Active TCP connections and listening ports.

**UDP Connections**: Active UDP listeners and connections.

**Running Processes**: Process IDs, names, and paths (if exposed).

**Storage Information**: Disk usage, file systems, mount points.

**Installed Software**: Software versions and installed packages.

## 4. Command Reference

### Basic Syntax

```bash
snmp-check [OPTIONS] TARGET
```

### Core Flags

**-c COMMUNITY**: Specify SNMP community string (required for v1/v2c).

**-v VERSION**: SNMP version (1, 2c, or 3). Default: 1.

**-p PORT**: Specify SNMP port (default: 161).

**-w WRITE_COMMUNITY**: Community string for write access (if needed).

**-t TIMEOUT**: Connection timeout in seconds (default: 5).

**-d**: Debug mode. Display SNMP operations and responses.

### SNMPv3 Flags

**-u USERNAME**: SNMPv3 username (required for v3).

**-l LEVEL**: Security level (noAuthNoPriv, authNoPriv, authPriv).

**-a PROTOCOL**: Authentication protocol (MD5, SHA).

**-A PASSPHRASE**: Authentication passphrase.

**-x PROTOCOL**: Privacy protocol (DES, AES).

**-X PASSPHRASE**: Privacy passphrase.

### Output Flags

**-C**: Compact output format. Reduce verbosity.

**--all**: Query all available OID subtrees.

**-o FILE**: Write output to file.

## 5. Examples

### Basic Enumeration

```bash
snmp-check -c public 192.168.1.100
```

**Result**: Enumerates target using "public" community string.

### SNMPv2c Enumeration

```bash
snmp-check -c public -v 2c 192.168.1.100
```

**Result**: Uses SNMPv2c protocol for enumeration.

### SNMPv3 Enumeration

```bash
snmp-check -v 3 -u admin -l authPriv -a SHA -A secret123 -x AES -X private123 192.168.1.100
```

**Result**: Authenticates and encrypts SNMPv3 session.

### Compact Output

```bash
snmp-check -C -c public 192.168.1.100
```

**Result**: Reduced verbosity output.

### File Output

```bash
snmp-check -c public -o device_info.txt 192.168.1.100
```

**Result**: Saves enumeration results to file.

### Custom Port

```bash
snmp-check -c public -p 1161 192.168.1.100
```

**Result**: Scans non-standard SNMP port.

### Extended Timeout

```bash
snmp-check -c public -t 10 192.168.1.100
```

**Result**: Uses 10-second timeout for slow devices.

### Debug Mode

```bash
snmp-check -d -c public 192.168.1.100
```

**Result**: Displays detailed SNMP operations.

## 6. Advanced Techniques

### Batch Enumeration Script

```bash
#!/bin/bash
COMMUNITY="public"
OUTPUT_DIR="enumeration_$(date +%Y%m%d_%H%M%S)"
mkdir -p "$OUTPUT_DIR"

for host in $(seq 1 254); do
  IP="192.168.1.$host"
  echo "Enumerating $IP..."
  snmp-check -c "$COMMUNITY" "$IP" > "$OUTPUT_DIR/$IP.txt" 2>/dev/null
done

echo "Results saved to $OUTPUT_DIR"
```

### Comprehensive OID Walk

```bash
snmp-check --all -c public 192.168.1.100
```

**Result**: Queries all available OID subtrees.

### Network-Wide Enumeration

```bash
for i in $(seq 1 254); do
  echo "=== 192.168.1.$i ==="
  timeout 10 snmp-check -c public 192.168.1.$i 2>/dev/null
done
```

### Custom OID Query with snmpwalk

```bash
# Query specific OID after snmp-check finds valid string
snmpwalk -c public -v1 192.168.1.100 1.3.6.1.2.1.25.4.2.1.2
```

### Comparison Report

```bash
# Compare two devices
diff <(snmp-check -c public 192.168.1.100) <(snmp-check -c public 192.168.1.101)
```

### Automation with Filtering

```bash
snmp-check -c public 192.168.1.100 | grep -E "(System|Interfaces|IP Address|Running)"
```

## 7. Detection & Defense

### Network Detection

**SNMP Enumeration Monitoring**: Detect repeated SNMP walks indicating enumeration activity.

**OID Tree Access Patterns**: Monitor for access to sensitive OID subtrees.

**Connection Frequency**: Track hosts performing extensive SNMP queries.

### System Detection

**SNMP Logs**: Review SNMP agent logs for unusual query patterns.

**Access Control Logs**: Monitor SNMP authentication attempts and failures.

### Defense Strategies

**Community String Hardening**: Use complex, unique community strings per device.

**SNMP Filtering**: Restrict SNMP access to authorized management stations.

**Disable Unnecessary OIDs**: Configure SNMP agents to restrict accessible OID subtrees.

**SNMPv3 Migration**: Upgrade to SNMPv3 with authentication and encryption.

**Access Control Lists**: Implement SNMP access restrictions on network devices.

**Network Segmentation**: Isolate management networks from user networks.

## 8. Troubleshooting

### No Results

**Verify Community String**: Test with snmpwalk.

```bash
snmpwalk -c public -v1 192.168.1.100 1.3.6.1.2.1.1
```

**Check SNMP Service**: Ensure SNMP is running on target.

```bash
systemctl status snmpd
```

### Partial Results

**OID Restrictions**: Target may restrict access to certain OID subtrees.

**SNMP Version Mismatch**: Try different SNMP versions.

```bash
snmp-check -c public -v 2c 192.168.1.100
```

### Connection Timeout

**Network Issues**: Verify network connectivity.

```bash
ping -c 2 192.168.1.100
```

**Firewall Blocking**: Check for firewall blocking SNMP traffic.

```bash
nmap -sU -p 161 192.168.1.100
```

**Increase Timeout**: Use -t flag for slower networks.

```bash
snmp-check -c public -t 15 192.168.1.100
```

### Permission Denied

**Community String Access**: Verify community string has appropriate access level.

**SNMPv3 Authentication**: Check credentials and security level.

```bash
snmp-check -v 3 -u admin -l authPriv -a MD5 -A password -x DES -X password 192.168.1.100
```

### Garbled Output

**Encoding Issues**: Some devices use non-standard encodings.

**OID Compatibility**: Try different SNMP versions or community strings.

## Resources

- **snmp-check Documentation**: Included with Kali Linux installation
- **SNMP Protocol RFC**: https://tools.ietf.org/html/rfc1157
- **MIB Tree Reference**: https://www.oid-info.com/
- **Kali Tools Documentation**: https://www.kali.org/tools/snmp-check/

## Related Tools

**snmpwalk**: Low-level SNMP tree walker for custom OID queries.

**braa**: Multi-threaded SNMP community string scanner for finding valid strings.

**onesixtyone**: Fast broadcast-based SNMP community string scanner.

**snmpget**: Direct SNMP object retrieval for specific OID values.
