---
title: "intrace - TCP Connection Traceroute"
description: "Traceroute tool that uses established TCP connections for network path discovery, working through stateless firewalls via TTL analysis"
mitre_tactic: "discovery"
mitre_tactic_id: "TA0007"
mitre_subcategory: "system_network_configuration_discovery"
categories:
  - "network"
  - "traceroute"
  - "tcp"
  - "network-discovery"
tags:
  - "traceroute"
  - "tcp"
  - "ttl"
  - "firewall"
  - "network-path"
install_command: "sudo apt install intrace"
version: "1.0"
github_repo: ""
kali_tools_page: "https://www.kali.org/tools/intrace/"
difficulty_level: "intermediate"
requires_root: true
gui_available: false
related_tools:
  - "0trace"
  - "traceroute"
  - "hping3"
  - "tcptraceroute"
---

# intrace - TCP Connection Traceroute

## 1. Introduction

**intrace** is a traceroute tool that leverages established TCP connections to discover network paths. By manipulating TTL values on packets within existing TCP sessions, intrace can trace routes through stateless firewalls that block traditional ICMP/UDP probes.

### Key Capabilities

**TCP-Based Tracing**: Uses established TCP connections for path discovery.

**Firewall Bypass**: Works through stateless firewalls that allow TCP traffic.

**TTL Analysis**: Manipulates TTL values to identify intermediate routers.

**Session Awareness**: Operates within existing TCP sessions for stealth.

### Use Cases

- **Penetration Testing**: Map network topology through restrictive firewalls.
- **Network Debugging**: Troubleshoot connectivity issues with TCP-based tracing.
- **Security Assessments**: Identify network infrastructure behind firewalls.
- **Incident Response**: Trace attack paths through network segments.

### How intrace Works

intrace attaches to an existing TCP connection and sends packets with incrementing TTL values. When TTL expires at intermediate routers, ICMP Time Exceeded messages reveal router IP addresses. This approach bypasses firewalls that block ICMP/UDP but allow TCP traffic.

## 2. Installation

### Standard Installation

```bash
sudo apt update
sudo apt install intrace
```

### Verify Installation

```bash
intrace --help
```

### Dependencies

intrace requires root privileges for packet manipulation and raw socket access.

## 3. Core Concepts

### TCP Traceroute Mechanics

**TCP Packets**: Uses TCP SYN or ACK packets with manipulated TTL values.

**TTL Expiration**: Intermediate routers decrement TTL and send ICMP Time Exceeded when TTL reaches zero.

**Session Hijacking**: intrace attaches to existing TCP connections for packet injection.

### Firewall Considerations

**Stateless Firewalls**: Allow established TCP connections regardless of packet content.

**Stateful Firewalls**: Track connection state and may allow injected packets within established sessions.

**Packet Filtering**: Firewalls that inspect packet content may detect TTL manipulation.

### Network Path Analysis

**Hop Identification**: Each router in the path is identified by its IP address.

**Response Time**: Round-trip time to each hop indicates network latency.

**Path Mapping**: Complete network path from source to destination.

## 4. Command Reference

### Basic Syntax

```bash
sudo intrace [OPTIONS] TARGET[:PORT]
```

### Core Flags

**-i INTERFACE**: Specify network interface.

**-s SOURCE_IP**: Set source IP address.

**-p PORT**: Specify destination port (default: 80).

**-t TIMEOUT**: Set timeout in seconds for responses.

**-v**: Verbose output. Show additional details.

### Advanced Flags

**-n**: Numeric mode. Don't resolve hostnames.

**-r NUM**: Set number of retries.

**-w WAIT**: Set wait time between probes.

**-x**: Extended mode. Show additional information.

## 5. Examples

### Basic Trace

```bash
sudo intrace 192.168.1.100
```

**Result**: Traces network path to target using default port.

### Custom Port

```bash
sudo intrace -p 443 192.168.1.100
```

**Result**: Traces path using port 443.

### Verbose Mode

```bash
sudo intrace -v 192.168.1.100
```

**Result**: Shows detailed packet information.

### Extended Mode

```bash
sudo intrace -x 192.168.1.100
```

**Result**: Displays additional trace information.

### Custom Interface

```bash
sudo intrace -i eth1 192.168.1.100
```

**Result**: Uses specific network interface.

### Source IP Spoofing

```bash
sudo intrace -s 192.168.1.50 192.168.1.100
```

**Result**: Uses spoofed source IP address.

### Save Output

```bash
sudo intrace 192.168.1.100 2>&1 | tee trace.txt
```

**Result**: Saves trace results to file.

### Multiple Targets

```bash
for target in 192.168.1.1 192.168.1.100 192.168.1.200; do
  echo "=== $target ==="
  sudo intrace "$target"
done
```

**Result**: Traces paths to multiple targets.

## 6. Advanced Techniques

### Comprehensive Network Discovery

```bash
#!/bin/bash
TARGETS="192.168.1.0/24"
OUTPUT_DIR="network_paths_$(date +%Y%m%d_%H%M%S)"
mkdir -p "$OUTPUT_DIR"

for i in $(seq 1 254); do
  TARGET="192.168.1.$i"
  RESULT=$(sudo intrace -v "$TARGET" 2>/dev/null)
  if [ -n "$RESULT" ]; then
    echo "$TARGET" > "$OUTPUT_DIR/$TARGET.txt"
    echo "$RESULT" >> "$OUTPUT_DIR/$TARGET.txt"
  fi
done

echo "Results saved to $OUTPUT_DIR"
```

### Combining with 0trace

```bash
# Compare results from both tools
echo "=== 0trace Results ==="
sudo 0trace eth0 192.168.1.100

echo "=== intrace Results ==="
sudo intrace 192.168.1.100
```

### Custom Probe Configuration

```bash
sudo intrace -w 1 -r 3 -t 5 192.168.1.100
```

**Result**: Custom timing with 1-second wait, 3 retries, 5-second timeout.

### Network Mapping Script

```bash
#!/bin/bash
INTERFACE="eth0"
NETWORK="192.168.1"

for i in $(seq 1 254); do
  TARGET="$NETWORK.$i"
  HOPS=$(sudo intrace -n "$TARGET" 2>/dev/null | grep -c "^[0-9]")
  if [ "$HOPS" -gt 0 ]; then
    echo "$TARGET: $HOPS hops"
  fi
done
```

### Path Comparison

```bash
# Compare paths to different destinations
sudo intrace 192.168.1.100 > path1.txt
sudo intrace 192.168.1.200 > path2.txt
diff path1.txt path2.txt
```

### Automation with Logging

```bash
#!/bin/bash
LOGFILE="trace_$(date +%Y%m%d_%H%M%S).log"
echo "Trace started at $(date)" > "$LOGFILE"

while read -r target; do
  echo "Tracing $target..." | tee -a "$LOGFILE"
  sudo intrace "$target" >> "$LOGFILE" 2>&1
done < targets.txt
```

## 7. Detection & Defense

### Network Detection

**TTL Anomaly Monitoring**: Detect unusual TTL values indicating traceroute activity.

**ICMP Response Analysis**: Monitor ICMP Time Exceeded messages for scanning patterns.

**TCP Connection Analysis**: Identify sequential TCP connections indicating intrace usage.

### System Detection

**Firewall Logs**: Review firewall logs for TTL-related events.

**Network Monitoring**: Track unusual packet patterns on network interfaces.

### Defense Strategies

**Rate Limiting**: Implement rate limiting for ICMP Time Exceeded responses.

**TTL Filtering**: Filter packets with suspicious TTL values.

**Connection Monitoring**: Monitor for sequential TCP connections from single sources.

**Network Segmentation**: Isolate sensitive network segments.

**Deep Packet Inspection**: Deploy DPI to detect TTL manipulation.

## 8. Troubleshooting

### No Results

**Interface Verification**: Confirm correct network interface.

```bash
ip link show
```

**Target Reachability**: Verify target is reachable.

```bash
ping -c 2 192.168.1.100
```

**Port Availability**: Ensure target port is open.

```bash
nmap -p 80,443 192.168.1.100
```

### Incomplete Trace

**Firewall Blocking**: Some firewalls may block TTL exceeded messages.

**Network Issues**: Intermediate routers may not generate ICMP responses.

**Timeout**: Increase timeout with -t flag.

```bash
sudo intrace -t 5 192.168.1.100
```

### Permission Denied

**Root Required**: intrace requires root privileges.

```bash
sudo intrace 192.168.1.100
```

### Interface Errors

**Interface Selection**: Use correct interface name.

```bash
ip addr show
```

**Promiscuous Mode**: Ensure interface supports promiscuous mode.

### Slow Performance

**Reduce Timeout**: Lower timeout for faster scanning.

```bash
sudo intrace -t 1 192.168.1.100
```

**Limit Retries**: Reduce retry count.

```bash
sudo intrace -r 1 192.168.1.100
```

### Connection Errors

**TCP State**: Ensure TCP connection is established.

**Session Hijacking**: Some systems may detect session manipulation.

## Resources

- **intrace Kali Documentation**: https://www.kali.org/tools/intrace/
- **Traceroute RFC**: https://tools.ietf.org/html/rfc1393
- **TCP/IP Analysis**: Network protocol documentation
- **Firewall Bypass Techniques**: Security research documentation

## Related Tools

**0trace**: TCP-based traceroute using existing connections for firewall traversal.

**traceroute**: Traditional ICMP/UDP traceroute tool.

**hping3**: Advanced packet crafting tool with traceroute capabilities.

**tcptraceroute**: TCP-based traceroute using SYN/ACK packets.
