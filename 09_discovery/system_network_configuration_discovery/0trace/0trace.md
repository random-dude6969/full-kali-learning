---
title: "0trace - Firewall Traversal Traceroute"
description: "Traceroute tool that bypasses firewalls using existing TCP connections, enabling network path discovery through stateful inspection devices"
mitre_tactic: "discovery"
mitre_tactic_id: "TA0007"
mitre_subcategory: "system_network_configuration_discovery"
categories:
  - "network"
  - "traceroute"
  - "firewall-bypass"
  - "network-discovery"
tags:
  - "traceroute"
  - "firewall"
  - "tcp"
  - "network-path"
  - "stateful"
install_command: "sudo apt install 0trace"
version: "0.1"
github_repo: ""
kali_tools_page: "https://www.kali.org/tools/0trace/"
difficulty_level: "intermediate"
requires_root: true
gui_available: false
related_tools:
  - "traceroute"
  - "hping3"
  - "firewalk"
  - "intrace"
---

# 0trace - Firewall Traversal Traceroute

## 1. Introduction

**0trace** is a traceroute tool designed to discover network paths through firewalls by leveraging existing TCP connections. Unlike traditional traceroute tools that rely on ICMP or UDP, 0trace uses established TCP connections to bypass stateful inspection firewalls.

### Key Capabilities

**Firewall Traversal**: Bypasses stateful firewalls by using existing TCP connections rather than generating new ICMP/UDP traffic.

**TCP-Based Tracing**: Uses TTL manipulation on established TCP connections to map network hops.

**Stealth Operation**: Appears as normal TCP traffic rather than traceroute probes.

**Network Path Discovery**: Identifies intermediate routers and network infrastructure.

### Use Cases

- **Penetration Testing**: Map network topology through restrictive firewalls.
- **Network Debugging**: Troubleshoot connectivity issues through firewalls.
- **Security Assessments**: Identify network infrastructure behind perimeter defenses.
- **Incident Response**: Trace attack paths through network segments.

### How 0trace Works

0trace establishes a TCP connection to the target, then manipulates TTL values on outgoing packets to elicit ICMP Time Exceeded messages from intermediate routers. This approach bypasses firewalls that block ICMP/UDP traceroute but allow TCP traffic.

## 2. Installation

### Standard Installation

```bash
sudo apt update
sudo apt install 0trace
```

### Verify Installation

```bash
0trace --help
```

### Dependencies

0trace requires root privileges for raw packet manipulation and uses libnet for packet crafting.

## 3. Core Concepts

### Traceroute Fundamentals

**TTL (Time to Live)**: IP packet field that decrements at each router hop. When TTL reaches zero, router sends ICMP Time Exceeded message.

**ICMP Time Exceeded**: Error message sent by routers when TTL expires, revealing router IP addresses.

**Hop Count**: Number of routers between source and destination.

### Firewall Challenges

**Stateful Inspection**: Firewalls track connection state and may block ICMP/UDP probes.

**Firewall Rules**: Many networks block traceroute protocols but allow established TCP connections.

**TCP Passthrough**: Firewalls typically allow established TCP connections, making TCP-based tracing effective.

### 0trace Methodology

**Connection Establishment**: 0trace first establishes a TCP connection to the target.

**TTL Manipulation**: The tool sends packets with incrementing TTL values through the established connection.

**Response Analysis**: ICMP Time Exceeded messages from intermediate routers reveal network path.

## 4. Command Reference

### Basic Syntax

```bash
sudo 0trace [OPTIONS] INTERFACE TARGET
```

### Core Flags

**-i INTERFACE**: Specify network interface for packet capture.

**-t TIMEOUT**: Set timeout in seconds for responses (default: 1).

**-l LENGTH**: Set packet length.

**-n**: Numeric mode. Don't resolve IP addresses to hostnames.

**-s PORT**: Specify source port for outgoing packets.

### Advanced Flags

**-p PORT**: Specify destination port (default: 80).

**-r NUM**: Set number of retries for unresponsive hops.

**-v**: Verbose output. Show additional details.

## 5. Examples

### Basic Trace

```bash
sudo 0trace eth0 192.168.1.100
```

**Result**: Traces network path to target using eth0 interface.

### Custom Port

```bash
sudo 0trace -p 443 eth0 192.168.1.100
```

**Result**: Traces path using port 443 instead of default 80.

### Numeric Mode

```bash
sudo 0trace -n eth0 192.168.1.100
```

**Result**: Displays IP addresses without hostname resolution.

### Extended Timeout

```bash
sudo 0trace -t 5 eth0 192.168.1.100
```

**Result**: Uses 5-second timeout for slow-responding networks.

### Verbose Output

```bash
sudo 0trace -v eth0 192.168.1.100
```

**Result**: Shows detailed packet information.

### Custom Source Port

```bash
sudo 0trace -s 54321 eth0 192.168.1.100
```

**Result**: Uses source port 54321 for outgoing packets.

### Save Output

```bash
sudo 0trace eth0 192.168.1.100 2>&1 | tee trace.txt
```

**Result**: Saves trace results to file.

### Multiple Targets

```bash
for target in 192.168.1.1 192.168.1.100 192.168.1.200; do
  echo "=== Tracing $target ==="
  sudo 0trace eth0 "$target"
done
```

**Result**: Traces paths to multiple targets.

## 6. Advanced Techniques

### Firewall Bypass Strategy

```bash
# First establish connection
echo "Establishing connection..."
nc -zv 192.168.1.100 443

# Then trace with 0trace
sudo 0trace -p 443 eth0 192.168.1.100
```

### Network Mapping

```bash
#!/bin/bash
INTERFACE="eth0"
TARGETS="targets.txt"

while IFS= read -r target; do
  echo "=== $target ==="
  sudo 0trace "$INTERFACE" "$target" | tee -a network_map.txt
done < "$TARGETS"
```

### Combining with Other Tools

```bash
# Use nmap to find open ports, then trace
nmap -p 80,443 192.168.1.0/24 -oG - | grep "open" | while read line; do
  target=$(echo "$line" | awk '{print $2}')
  sudo 0trace eth0 "$target"
done
```

### Custom Packet Length

```bash
sudo 0trace -l 1400 eth0 192.168.1.100
```

**Result**: Uses larger packets to test path MTU.

### Retry Configuration

```bash
sudo 0trace -r 3 -t 3 eth0 192.168.1.100
```

**Result**: Retries unresponsive hops up to 3 times with 3-second timeout.

### Automation Script

```bash
#!/bin/bash
INTERFACE="eth0"
NETWORK="192.168.1.0/24"

for i in $(seq 1 254); do
  TARGET="192.168.1.$i"
  RESULT=$(sudo 0trace -n "$INTERFACE" "$TARGET" 2>/dev/null)
  if [ -n "$RESULT" ]; then
    echo "$TARGET: $(echo "$RESULT" | tail -1)"
  fi
done
```

## 7. Detection & Defense

### Network Detection

**TTL Anomaly Detection**: Monitor for unusual TTL values indicating traceroute activity.

**ICMP Time Exceeded Monitoring**: Track ICMP Time Exceeded messages for scanning patterns.

**Connection Analysis**: Identify sequential TCP connections indicating 0trace usage.

### System Detection

**Firewall Logs**: Review firewall logs for TTL-related events.

**Network Monitoring**: Track unusual packet patterns on network interfaces.

### Defense Strategies

**Rate Limiting**: Implement rate limiting for ICMP Time Exceeded responses.

**TTL Filtering**: Filter packets with suspicious TTL values.

**Connection Monitoring**: Monitor for sequential TCP connections from single sources.

**Network Segmentation**: Isolate sensitive network segments.

**Firewall Rules**: Implement strict egress filtering.

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
sudo 0trace -t 5 eth0 192.168.1.100
```

### Permission Denied

**Root Required**: 0trace requires root privileges.

```bash
sudo 0trace eth0 192.168.1.100
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
sudo 0trace -t 1 eth0 192.168.1.100
```

**Limit Retries**: Reduce retry count.

```bash
sudo 0trace -r 1 eth0 192.168.1.100
```

## Resources

- **0trace Kali Documentation**: https://www.kali.org/tools/0trace/
- **Traceroute RFC**: https://tools.ietf.org/html/rfc1393
- **Network Path Analysis**: Various security training resources
- **Firewall Bypass Techniques**: Security research documentation

## Related Tools

**traceroute**: Traditional ICMP/UDP traceroute tool for networks without firewalls.

**hping3**: Advanced packet crafting tool with traceroute capabilities.

**firewalk**: Firewalking tool for mapping firewall rules through TTL manipulation.

**intrace**: TCP-based traceroute using established connections.
