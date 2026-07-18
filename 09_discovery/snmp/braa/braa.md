---
title: "braa - Multi-threaded SNMP Community String Scanner"
description: "High-performance SNMP community string brute-force scanner supporting SNMPv1/v2c with multi-threaded scanning capabilities for network discovery and enumeration"
mitre_tactic: "discovery"
mitre_tactic_id: "TA0007"
mitre_subcategory: "snmp"
categories:
  - "snmp"
  - "network-scanning"
  - "enumeration"
  - "brute-force"
tags:
  - "snmp"
  - "community-strings"
  - "scanner"
  - "multi-threaded"
  - "v1"
  - "v2c"
install_command: "sudo apt install braa"
version: "0.3"
github_repo: ""
kali_tools_page: "https://www.kali.org/tools/braa/"
difficulty_level: "intermediate"
requires_root: true
gui_available: false
related_tools:
  - "onesixtyone"
  - "snmpwalk"
  - "snmp-check"
  - "snmpget"
---

# braa - Multi-threaded SNMP Community String Scanner

## 1. Introduction

**braa** is a high-performance, multi-threaded SNMP community string scanner designed for network discovery and enumeration. The tool brute-forces SNMP community strings across network ranges, identifying valid credentials that can be used to extract sensitive system information.

### Key Capabilities

**Multi-threaded Scanning**: braa leverages multiple threads to scan network ranges concurrently, dramatically reducing scan times compared to single-threaded alternatives.

**SNMP Protocol Support**: Full support for SNMPv1 and SNMPv2c protocols, covering the most commonly deployed SNMP versions in enterprise environments.

**Network Range Scanning**: Ability to scan entire /24 and other CIDR subnets, not just individual hosts, enabling comprehensive network-wide discovery.

**High Performance**: Optimized for speed with efficient packet handling and minimal overhead, making it suitable for large-scale network assessments.

### Use Cases

- **Penetration Testing**: Identify SNMP-enabled devices with weak or default community strings during security assessments.
- **Network Auditing**: Catalog all SNMP-accessible devices across an organization's network infrastructure.
- **Compliance Verification**: Verify that SNMP community strings meet organizational security policies.
- **Asset Discovery**: Locate previously unknown SNMP-enabled devices on network segments.

### How braa Works

braa operates by sending SNMP GetRequest packets with candidate community strings to target hosts. When a valid community string is used, the target responds with system information. braa captures these responses and reports successful authentications.

The tool uses a wordlist-based approach, testing each community string against each target host. Multi-threading allows concurrent testing of multiple hosts and strings, significantly accelerating the discovery process.

## 2. Installation

### Standard Installation

```bash
sudo apt update
sudo apt install braa
```

### Verify Installation

```bash
braa --version
```

### Manual Installation from Source

```bash
git clone https://github.com/mteg/braa.git
cd braa
make
sudo cp braa /usr/local/bin/
```

### Installation Dependencies

**libnet**: Required for raw packet crafting and network communication.

**pcre**: Used for regular expression pattern matching in community string files.

```bash
sudo apt install libnet1-dev libpcre3-dev
```

## 3. Core Concepts

### SNMP Community Strings

**Community Strings** function as passwords for SNMP access. Common default strings include:

**public**: Read-only access, most commonly deployed default string.

**private**: Read-write access, provides higher privilege level.

**manager**: Used by SNMP management stations for read access.

### SNMP Versions

**SNMPv1**: Original SNMP protocol with plaintext community strings. Authentication is simple string matching.

**SNMPv2c**: Enhanced version with improved error handling and bulk request support. Still uses plaintext community strings.

**SNMPv3**: Latest version with encryption and authentication. braa does not support SNMPv3 due to its cryptographic requirements.

### Scanning Architecture

**Thread Pool**: braa creates a pool of worker threads that process target/string combinations in parallel.

**Packet Queue**: Incoming responses are queued and processed asynchronously to prevent blocking.

**Rate Limiting**: Adjustable throttling to avoid overwhelming target networks or triggering intrusion detection systems.

### CIDR Notation

braa accepts CIDR notation for network range specification:

- **/24**: 256 hosts (e.g., 192.168.1.0/24)
- **/16**: 65,536 hosts (e.g., 192.168.0.0/16)
- **/32**: Single host (e.g., 192.168.1.100/32)

## 4. Command Reference

### Basic Syntax

```bash
braa [OPTIONS] COMMUNITY@TARGET
```

### Core Flags

**-t NUM**: Set number of concurrent threads (default: 50). Higher values increase speed but consume more resources.

**-p PORT**: Specify SNMP port (default: 161). Change for non-standard deployments.

**-q**: Quiet mode. Suppress verbose output, showing only successful findings.

**-v VERSION**: Specify SNMP version (1 or 2c). Default is version 1.

**-w**: Wait mode. Increase timeout for slow-responding devices.

### Advanced Flags

**-s SOURCE_IP**: Set source IP address for outgoing packets. Useful for spoofing or binding to specific interfaces.

**-d**: Debug mode. Display raw packet data for troubleshooting.

**-m MAX_HOSTS**: Limit maximum number of hosts to scan simultaneously.

**-r RETRIES**: Set number of retries for unresponsive hosts (default: 1).

### Output Options

**-o FILE**: Write results to specified output file.

**-f FORMAT**: Output format (text, csv, xml).

## 5. Examples

### Basic Community String Test

```bash
braa public@192.168.1.100
```

**Result**: Tests "public" community string against single host.

### Network Range Scan

```bash
braa public@192.168.1.0/24
```

**Result**: Tests "public" against all 256 hosts in subnet.

### Multiple Community Strings

```bash
braa community.txt@192.168.1.0/24
```

**Result**: Tests each string in community.txt against all hosts.

### High-Speed Scan

```bash
braa -t 100 public@192.168.1.0/24
```

**Result**: Uses 100 threads for maximum scanning speed.

### SNMPv2c Scanning

```bash
braa -v 2c public@192.168.1.0/24
```

**Result**: Scans using SNMPv2c protocol instead of v1.

### Custom Port

```bash
braa -p 1161 public@192.168.1.100
```

**Result**: Scans non-standard SNMP port 1161.

### File Output

```bash
braa -o results.txt public@192.168.1.0/24
```

**Result**: Saves successful findings to results.txt.

### Quiet Mode Scan

```bash
braa -q -o results.txt public@192.168.1.0/24
```

**Result**: Minimal output, only results saved to file.

## 6. Advanced Techniques

### Custom Community String Files

Create comprehensive wordlists for thorough scanning:

```bash
echo -e "public\nprivate\nmanager\nadmin\nsnmp\ncommunity\nsecret\nreadonly\nreadwrite" > custom_strings.txt
braa custom_strings.txt@192.168.1.0/24
```

### Optimized Large Network Scans

For /16 networks, adjust thread count and add delays:

```bash
braa -t 200 -r 2 public@192.168.0.0/16
```

### Combining with Other Tools

Pipe successful community strings to snmpwalk for enumeration:

```bash
braa public@192.168.1.0/24 2>/dev/null | while read host string; do
  snmpwalk -c "$string" -v1 "$host" 1.3.6.1.2.1.1
done
```

### Rate-Limited Scanning

Avoid IDS detection with conservative settings:

```bash
braa -t 10 -r 1 -w public@192.168.1.0/24
```

### Automation Scripts

```bash
#!/bin/bash
NETWORK="192.168.1.0/24"
STRINGS="community_strings.txt"

braa -q -o scan_results.txt "$STRINGS@$NETWORK"

if [ -s scan_results.txt ]; then
  echo "Found valid community strings:"
  cat scan_results.txt
fi
```

## 7. Detection & Defense

### Network Detection

**SNMP Flood Monitoring**: Detect rapid SNMP requests indicating brute-force attempts.

**Anomaly Detection**: Monitor for unusual SNMP traffic patterns or source IPs.

**Rate Analysis**: Identify scanning patterns based on request frequency.

### System Detection

**SNMP Log Analysis**: Review SNMP service logs for authentication failures.

**Connection Monitoring**: Track unique source IPs connecting to SNMP ports.

### Defense Strategies

**Community String Hardening**: Replace default community strings with complex, unique values.

**SNMP Filtering**: Restrict SNMP access using firewall rules to authorized management stations only.

**SNMPv3 Migration**: Upgrade to SNMPv3 with authentication and encryption.

**Disable Unused SNMP**: Disable SNMP on devices where network management is not required.

**Access Control Lists**: Implement SNMP community string access restrictions on network devices.

## 8. Troubleshooting

### No Results Found

**Verify Target Availability**: Confirm target hosts are online and responding to ICMP.

```bash
ping -c 2 192.168.1.100
```

**Check SNMP Service**: Ensure SNMP is running on target.

```bash
snmpwalk -c public -v1 192.168.1.100 1.3.6.1.2.1.1
```

### Slow Performance

**Reduce Thread Count**: Lower -t value to reduce network congestion.

```bash
braa -t 20 public@192.168.1.0/24
```

**Increase Timeout**: Use -w for slower networks.

```bash
braa -w public@192.168.1.0/24
```

### Permission Denied

**Run as Root**: braa requires root privileges for raw packet operations.

```bash
sudo braa public@192.168.1.100
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

- **SNMP Protocol RFC**: https://tools.ietf.org/html/rfc1157
- **Kali Tools Documentation**: https://www.kali.org/tools/braa/
- **SNMP Security Best Practices**: https://tools.ietf.org/html/rfc3584
- **Network Enumeration Guide**: Various security training resources
- **braa Source Code**: Available in Kali repositories

## Related Tools

**onesixtyone**: Broadcast-based SNMP community string scanner, faster for single-subnet scans.

**snmpwalk**: SNMP tree walker for enumerating device information after finding valid strings.

**snmp-check**: Comprehensive SNMP enumeration tool for detailed device information extraction.

**snmpget**: Direct SNMP object retrieval for targeted information gathering.
