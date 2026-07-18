---
title: "Firewalk"
description: "Active reconnaissance tool that tests firewall rules and ACLs by sending TTL-limited probes through a gateway to determine which ports are open on downstream hosts."
categories: ["Network Security Appliances"]
tags: ["firewall", "acl", "reconnaissance", "ttl", "traceroute", "port-scanning"]
mitre_tactic: "discovery"
mitre_tactic_id: "TA0007"
mitre_subcategory: "network_security_appliances"
install_command: "sudo apt install firewalk"
version: "5.0"
github_repo: ""
kali_tools_page: "https://www.kali.org/tools/firewalk/"
difficulty_level: "intermediate"
requires_root: true
gui_available: false
related_tools: ["hping3", "nmap", "0trace"]
---

# Firewalk

## Chapter 1: Introduction & Overview

### What is Firewalk?

Firewalk is an active network reconnaissance tool designed to map firewall rules and access control lists (ACLs) by analyzing how a gateway handles TTL-limited probes. It exploits the fundamental behavior of IP routing: packets with a TTL of exactly one hop beyond the gateway will either be forwarded (if allowed) or dropped (if denied). By systematically probing ports and interpreting ICMP TTL Exceeded responses versus silence, Firewalk reconstructs which ports the firewall permits or blocks.

Firewalk operates at Layer 3/4 — it crafts raw IP packets and relies on network-level responses, not application-layer behavior. This makes it protocol-agnostic for TCP, UDP, and ICMP probing, though TCP is the most common use case.

### History & Background

Firewalk was originally developed by Mike D. Schiffman and Dustin D. Trammell, first released in the late 1990s during the peak era of firewall enumeration research. The tool was inspired by the TTL manipulation technique described in the seminal paper "Firewalking" by the same authors.

**Key historical milestones:**
- **1998**: Original concept and proof-of-concept implementation
- **2000s**: Version 5.0 released with improved stability, libdumbnet dependency
- **2010s–present**: Maintained in Kali Linux repositories; no major version changes but consistent updates to dependencies

The tool's name is a portmanteau of "firewall" and "traceroute," reflecting its operational similarity to traceroute but with explicit firewall-focused probing logic.

### When to Use This Tool

Firewalk is appropriate in the following scenarios:

- **Penetration testing engagements** where mapping firewall rules is an explicit objective
- **Network auditing** to verify that firewall ACLs match organizational policy
- **Red team operations** requiring knowledge of which ports survive firewall filtering before launching application-layer attacks
- **Compliance verification** confirming that only authorized ports are open through perimeter defenses
- **Blue team exercises** to validate firewall configuration from an external attacker's perspective

Firewalk is **not** suitable for:
- Internal network scanning behind the firewall (use nmap)
- Application-layer vulnerability scanning
- Situations where stealth is critical (Firewalk generates predictable ICMP traffic)

### Key Concepts

**TTL-Based Probing**: The core technique. Firewalk sets the TTL of outgoing packets to exactly the number of hops between the attacker and the gateway plus one. If the gateway forwards the packet, it reaches the next hop, decrements TTL to zero, and the target responds with ICMP Time Exceeded. If the gateway blocks it, no response is generated.

**Gateway Discovery**: Firewalk must first identify the gateway (default router) between the attacker and the target. This is typically done via `traceroute` or by examining the routing table.

**Probe Packets**: Firewalk sends TCP SYN, UDP, or ICMP packets to the target on specific ports. The response (or lack thereof) determines whether the port is open, closed, or filtered.

**Strict RFC Compliance**: The `-r` flag forces Firewalk to adhere strictly to RFC specifications for TTL handling, which can improve accuracy against some firewall implementations but may fail against others.

**Expire Vector**: The number of hops from the attacker to the gateway. Firewalk automatically determines this, but it can be overridden with `-x`.

## Chapter 2: Installation & Setup

### System Requirements

**Minimum requirements:**
- **OS**: Kali Linux or any Debian-based distribution
- **RAM**: 128 MB
- **Disk space**: 5 MB
- **Network**: Ethernet or Wi-Fi interface with raw socket capability
- **Privileges**: Root access required (raw packet construction)

**Dependencies:**
- `libdumbnet` — network utility library for low-level packet manipulation
- `libnet` — portable interface for packet construction
- `libpcap` — packet capture library
- A C compiler (gcc) for building from source

### Installation Methods

**Method 1: Kali Linux repositories (recommended)**

```bash
sudo apt update
sudo apt install firewalk
```

**Method 2: Build from source**

```bash
# Install build dependencies
sudo apt install build-essential libdumbnet-dev libpcap-dev libnet1-dev

# Clone or download source
git clone https://github.com/lightnobody/firewalk.git
cd firewalk

# Build and install
./configure
make
sudo make install
```

**Method 3: From original upstream**

```bash
# Download from original source (if available)
wget https://packetstormsecurity.com/files/download/31948/firewalk-5.0.tar.gz
tar -xzf firewalk-5.0.tar.gz
cd firewalk-5.0
./configure
make
sudo make install
```

### Configuration

Firewalk does not require a configuration file. All parameters are specified via command-line flags. However, consider these preparation steps:

**Verify interface is available:**
```bash
ip link show
# Identify your active interface (e.g., eth0, wlan0)
```

**Verify gateway:**
```bash
ip route show default
# Note the default gateway IP
```

**Enable IP forwarding (if needed for advanced routing):**
```bash
sudo sysctl net.ipv4.ip_forward=1
```

### Verifying Installation

```bash
# Check version
firewalk --version

# Check help output
firewalk --help

# Quick test (sends minimal probes)
firewalk -S80 -n -pTCP 192.168.1.1 192.168.0.1
```

Expected output should show the gateway IP, target IP, and probe results for port 80.

## Chapter 3: Core Concepts & Architecture

### How It Works

Firewalk's operation follows a precise sequence:

1. **Gateway Discovery**: The tool determines the hop count between the attacker's machine and the gateway (typically the default router). This is done via a TTL-limited traceroute.

2. **TTL Calculation**: Firewalk calculates the exact TTL value needed so that packets expire one hop past the gateway. For example, if the gateway is 3 hops away, the TTL is set to 4.

3. **Probe Construction**: For each port in the specified range, Firewalk constructs a packet (TCP SYN, UDP, or ICMP) with the calculated TTL.

4. **Response Analysis**:
   - **ICMP Time Exceeded received**: The gateway forwarded the packet, meaning the port is likely open or at least not filtered by the firewall
   - **No response**: The gateway dropped the packet, meaning the port is filtered/blocked
   - **TCP RST received**: The target host responded with a reset (port closed but not filtered)
   - **TCP SYN-ACK received**: The port is open

5. **Result Compilation**: Firewalk aggregates results and displays which ports are open, closed, or filtered through the gateway.

### Protocols & Standards

**IP TTL Behavior (RFC 791)**:
- Each router decrements the TTL by at least 1
- When TTL reaches 0, the router discards the packet and sends ICMP Time Exceeded (Type 11)
- Firewalk exploits this by setting TTL to gateway_hop_count + 1

**TCP Probing (RFC 793)**:
- TCP SYN packets are sent to target ports
- SYN-ACK indicates open port (and TTL allows forwarding)
- RST indicates closed port (and TTL allows forwarding)
- No response indicates filtered port

**ICMP Probing**:
- ICMP Echo Request packets sent with specific TTL
- Used to verify reachability of non-TCP services

### Network Architecture

```
[Attacker] ──── [Gateway/Firewall] ──── [Target Network]
     │                    │                      │
     │  TTL=4             │  TTL=3→2→1→0        │
     │  ───────────────>  │  ──────────────>     │
     │                    │  (expires here)      │
     │                    │  ICMP Time Exceeded  │
     │  <───────────────  │                      │
     │                    │                      │
     │  TTL=4             │  DROPPED             │
     │  ───────────────>  │  (no response)       │
     │  (silence = filtered)                     │
```

The attacker's machine, gateway, and target must be in a linear routing path for Firewalk to work correctly. If the gateway is not the last hop before the target, results will be unreliable.

## Chapter 4: Command Reference

### All Flags/Options

| Flag | Long Form | Description |
|------|-----------|-------------|
| `-S <ports>` | `--source-ports` | Port range to scan (e.g., `-S80,443` or `-S1-1024`) |
| `-p <proto>` | `--proto` | Protocol to use: `TCP`, `UDP`, or `ICMP` |
| `-i <iface>` | `--interface` | Network interface to use (e.g., `eth0`) |
| `-n` | `--no-dns` | Disable DNS resolution for IPs |
| `-s <port>` | `--source-port` | Source port for probes (default: random) |
| `-T <ms>` | `--timeout` | Timeout in milliseconds for responses |
| `-t <ttl>` | `--ttl` | Set TTL manually (bypasses auto-detection) |
| `-d <port>` | `--destination-port` | Default destination port for initial probes |
| `-r` | `--strict-rfc` | Strict RFC compliance for TTL handling |
| `-x <hops>` | `--expire-vector` | Number of hops to gateway (bypasses auto-detection) |
| `-v` | `--verbose` | Verbose output |
| `-h` | `--help` | Display help |

### Output Formats

Firewalk outputs results to stdout in a human-readable table format:

```
Firewalk: closing down...
TCP open: 192.168.0.1:80    [open]
TCP open: 192.168.0.1:443   [open]
TCP open: 192.168.0.1:22    [closed]
TCP open: 192.168.0.1:8080  [filtered]
```

**Result states:**
- **open**: Gateway forwarded the probe, target responded positively
- **closed**: Gateway forwarded the probe, target responded with RST
- **filtered**: Gateway dropped the probe (no response)
- **unreachable**: ICMP Unreachable received

## Chapter 5: Usage Examples

### Basic Usage

**Scan common web ports through a gateway:**

```bash
sudo firewalk -S80,443,8080,8443 -n -pTCP 192.168.1.1 192.168.0.1
```

This scans ports 80, 443, 8080, and 8443 on target `192.168.0.1`, with gateway `192.168.1.1`, disabling DNS resolution.

**Scan a port range:**

```bash
sudo firewalk -S1-1024 -n -pTCP 192.168.1.1 192.168.0.1
```

Scans the well-known ports (1–1024) through the gateway.

**UDP probing:**

```bash
sudo firewalk -S53,123,161 -n -pUDP 192.168.1.1 192.168.0.1
```

Tests DNS (53), NTP (123), and SNMP (161) UDP services.

### Intermediate Usage

**Specify interface and timeout:**

```bash
sudo firewalk -S80,443 -i eth1 -n -pTCP -T5000 192.168.1.1 192.168.0.1
```

Uses `eth1` as the outgoing interface with a 5-second timeout per probe.

**Override TTL and source port:**

```bash
sudo firewalk -S80 -n -pTCP -t 64 -s 54321 192.168.1.1 192.168.0.1
```

Sets TTL to 64 explicitly and uses source port 54321 for probes.

**Strict RFC mode:**

```bash
sudo firewalk -S80,443,8080 -n -pTCP -r 192.168.1.1 192.168.0.1
```

Enables strict RFC compliance, useful against firewalls that implement TTL checking differently.

### Advanced Usage

**Scan specific services with verbose output:**

```bash
sudo firewalk -S21,22,23,25,53,80,110,143,443,993,995 -n -pTCP -v 192.168.1.1 192.168.0.1
```

Scans common service ports with detailed output showing TTL values and response types.

**Override expire vector for non-standard topology:**

```bash
sudo firewalk -S80 -n -pTCP -x 5 192.168.1.1 192.168.0.1
```

Manually sets the gateway distance to 5 hops, bypassing auto-detection. Useful when the attacker is behind multiple hops before the gateway.

**Combine with traceroute for gateway discovery:**

```bash
# First, discover the gateway
traceroute -n 192.168.0.1

# Then use Firewalk with manual TTL
sudo firewalk -S80 -n -pTCP -t 3 192.168.1.1 192.168.0.1
```

### Real-World Scenarios

**Scenario 1: External penetration test — map web-facing firewall rules**

```bash
# Scan HTTP/HTTPS through the perimeter firewall
sudo firewalk -S80,443,8080,8443 -n -pTCP 10.0.0.1 192.168.0.100

# Then scan additional services
sudo firewalk -S22,3389,5432,3306 -n -pTCP 10.0.0.1 192.168.0.100
```

**Scenario 2: Verify firewall policy compliance**

```bash
# Check if only allowed ports are open
sudo firewalk -S1-65535 -n -pTCP 192.168.1.1 10.0.0.50

# Compare results against the firewall ACL documentation
```

**Scenario 3: Pre-attack reconnaissance for red team**

```bash
# Identify open ports before launching targeted exploits
sudo firewalk -S21,22,23,25,53,80,110,135,139,443,445,993,995,1433,3306,3389,5432,8080 -n -pTCP 192.168.1.1 192.168.0.50
```

## Chapter 6: Advanced Techniques

### Automation & Scripting

**Batch scanning multiple targets:**

```bash
#!/bin/bash
# scan_firewalls.sh — enumerate firewall rules for multiple gateways
GATEWAYS=("192.168.1.1" "10.0.0.1" "172.16.0.1")
TARGETS=("192.168.0.10" "10.0.1.20" "172.16.1.30")
PORTS="22,80,443,3389,8080"

for i in "${!GATEWAYS[@]}"; do
    echo "=== Scanning target ${TARGETS[$i]} via gateway ${GATEWAYS[$i]} ==="
    sudo firewalk -S${PORTS} -n -pTCP "${GATEWAYS[$i]}" "${TARGETS[$i]}" > "result_${i}.txt" 2>&1
    echo "Results saved to result_${i}.txt"
done
```

**Automated gateway discovery and scanning:**

```bash
#!/bin/bash
TARGET="192.168.0.1"
# Discover gateway hop count
HOPS=$(traceroute -n -m 10 $TARGET | tail -n +2 | head -1 | awk '{print $1}')
echo "Gateway is approximately $HOPS hops away"

# Scan with discovered TTL
sudo firewalk -S80,443 -n -pTCP -x $HOPS 192.168.1.1 $TARGET
```

### Chaining with Other Tools

**Firewalk + Nmap for deep inspection:**

```bash
# Step 1: Identify open ports through firewall
sudo firewalk -S1-1024 -n -pTCP 192.168.1.1 192.168.0.50 > open_ports.txt

# Step 2: Deep scan identified open ports with Nmap
nmap -sV -sC -p 80,443 192.168.0.50
```

**Firewalk + hping3 for advanced probing:**

```bash
# Use hping3 for SYN scan on discovered open ports
sudo hping3 -S -p 80 -c 3 192.168.0.50
sudo hping3 -S -p 443 -c 3 192.168.0.50
```

**Firewalk + tcpdump for packet analysis:**

```bash
# Capture Firewalk probes in real-time
sudo tcpdump -i eth0 -nn host 192.168.0.50 &
FIREWALK_PID=$!

# Run Firewalk
sudo firewalk -S80,443 -n -pTCP 192.168.1.1 192.168.0.50

# Stop capture
kill $FIREWALK_PID
```

### Custom Configurations

**Non-standard network topologies:**

```bash
# Multi-hop gateway scenario
# Attacker → Router1 → Router2 (gateway) → Target
# If Router2 is 4 hops from attacker:
sudo firewalk -S80 -n -pTCP -x 4 192.168.2.1 192.168.3.50
```

**IPv6 probing (limited support):**

```bash
# Note: Firewalk has limited IPv6 support; prefer nmap for IPv6
# For IPv6, use nmap with traceroute:
nmap -6 --traceroute -p 80,443 target_ipv6_address
```

### Performance Tuning

**Reduce timeout for faster scans:**

```bash
sudo firewalk -S1-1024 -n -pTCP -T2000 192.168.1.1 192.168.0.50
```

**Scan specific ports only:**

```bash
# Instead of scanning all 65535 ports, target known services
sudo firewalk -S21,22,23,25,53,80,110,143,443,993,995,1433,3306,3389,5432,8080,8443 -n -pTCP 192.168.1.1 192.168.0.50
```

## Chapter 7: Detection & Defense

### How to Detect

Firewalk generates detectable network traffic patterns:

1. **TTL Anomalies**: Packets arriving with TTL values exactly one hop past the gateway are suspicious. Normal traffic does not have such precisely crafted TTL values.

2. **Sequential Port Probing**: Firewalk sends probes to many ports in rapid succession from the same source. This pattern is detectable by flow analysis.

3. **ICMP Time Exceeded Flood**: If the target ports are open, each probe generates an ICMP Time Exceeded response. A spike in these responses is a detection signal.

4. **Source Port Patterns**: If a fixed source port is used (via `-s`), it creates a fingerprintable pattern.

### Defensive Measures

**Firewall TTL checking:**
```
# On iptables-based firewalls, reject packets with suspicious TTL values
iptables -A INPUT -m ttl --ttl-eq 1 -j DROP
iptables -A INPUT -m ttl --ttl-lt 2 -j LOG --log-prefix "FIREWALK: "
```

**Rate limiting ICMP responses:**
```
# Limit ICMP Time Exceeded responses to reduce information leakage
iptables -A OUTPUT -p icmp --icmp-type 11 -m limit --limit 1/s -j ACCEPT
iptables -A OUTPUT -p icmp --icmp-type 11 -j DROP
```

**Disable ICMP TTL Exceeded:**
```
# Prevent the firewall from sending TTL Exceeded messages
# This breaks traceroute but prevents Firewalk
echo 1 > /proc/sys/net/ipv4/icmp_errors_use_inbound_ifaddr
```

### IDS/IPS Signatures

**Snort/Suricata rule examples:**

```
# Detect Firewalk-like TTL manipulation
alert icmp any any -> any any (msg:"Possible Firewalk TTL Probe"; \
  ttl:1; itype:8; sid:1000001; rev:1;)

# Detect sequential port scanning pattern
alert tcp any any -> any any (msg:"Sequential Port Scan"; \
  flags:S; threshold:type both, track by_src, count 20, seconds 60; \
  sid:1000002; rev:1;)
```

### Log Analysis

**Firewall log monitoring:**
```bash
# Monitor iptables logs for Firewalk patterns
sudo grep "FIREWALK" /var/log/syslog

# Analyze connection attempts by source
sudo netstat -an | grep SYN_SENT | awk '{print $5}' | sort | uniq -c | sort -rn

# Check for TTL anomalies in tcpdump
sudo tcpdump -i eth0 'ip[8] <= 2' -nn
```

## Chapter 8: Trouleshooting & FAQ

### Common Issues

**Problem: "No route to host" error**

Cause: The gateway IP or target IP is incorrect, or the network interface is not configured.

Solution:
```bash
# Verify routing table
ip route show

# Test basic connectivity
ping -c 3 192.168.1.1
ping -c 3 192.168.0.1

# Verify interface is up
ip link show eth0
```

**Problem: All ports show as "filtered"**

Cause: The firewall is dropping all packets regardless of TTL, or the gateway is not forwarding TTL-limited packets.

Solution:
```bash
# Verify TTL calculation
traceroute -n 192.168.0.1

# Manually override TTL
sudo firewalk -S80 -n -pTCP -t 3 192.168.1.1 192.168.0.1

# Check if gateway even responds to TTL Expired
sudo hping3 -T -c 1 192.168.0.1
```

**Problem: "Permission denied" when running**

Cause: Firewalk requires root privileges for raw packet construction.

Solution:
```bash
# Run with sudo
sudo firewalk -S80 -n -pTCP 192.168.1.1 192.168.0.1

# Or run as root
su -c "firewalk -S80 -n -pTCP 192.168.1.1 192.168.0.1"
```

**Problem: Results are inconsistent**

Cause: Network asymmetry — the return path differs from the forward path, or load balancing distributes probes differently.

Solution:
```bash
# Increase timeout for more reliable results
sudo firewalk -S80 -n -pTCP -T10000 192.168.1.1 192.168.0.1

# Run multiple times and compare
for i in 1 2 3; do
    echo "Run $i:"
    sudo firewalk -S80 -n -pTCP 192.168.1.1 192.168.0.1
done
```

### FAQ

**Q: Can Firewalk scan through NAT?**
A: Firewalk works through NAT as long as the NAT device is the gateway. However, the TTL calculation must account for the NAT hop.

**Q: Does Firewalk work with IPv6?**
A: Limited support. IPv6 Firewalk is experimental and unreliable. Use `nmap --traceroute -6` for IPv6 environments.

**Q: Is Firewalk detectable?**
A: Yes. IDS/IPS systems can detect the TTL manipulation pattern. Use rate limiting and multiple source IPs to reduce detectability.

**Q: Can Firewalk bypass stateful firewalls?**
A: Firewalk does not bypass firewalls — it enumerates them. If a firewall drops packets based on TTL, Firewalk will report them as filtered.

**Q: What's the difference between Firewalk and Nmap traceroute?**
A: Firewalk is specifically designed for firewall rule enumeration with TTL manipulation. Nmap's traceroute (`--traceroute`) is a general-purpose path discovery tool that also does port scanning.

**Q: How accurate is Firewalk?**
A: Accuracy depends on network topology. Firewalk works best with symmetric routing and single-gateway topologies. Results should be verified with additional tools like Nmap.

## Resources

### Official Documentation
- **Firewalk man page**: `man firewalk`
- **Original paper**: "Firewalking" by Mike D. Schiffman and Dustin D. Trammell
- **Kali tools page**: https://www.kali.org/tools/firewalk/

### Online Resources
- **Nmap Project — Firewalk**: https://nmap.org/book/firewalk.html
- **SANS — Firewalk Analysis**: https://www.sans.org/
- **Packet Storm Security**: https://packetstormsecurity.com/files/view/31948/firewalk-5.0.tar.gz

### Related Tools
- **hping3** — Packet crafting and manipulation tool
- **nmap** — Comprehensive port scanner with traceroute
- **0trace** — Traceroute through stateful firewalls
- **tcpdump** — Packet capture for analysis
- **Scapy** — Python packet manipulation library
