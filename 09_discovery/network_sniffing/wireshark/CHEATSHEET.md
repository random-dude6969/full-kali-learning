---
title: "Wireshark Cheatsheet"
description: "Quick reference guide for Wireshark network protocol analyzer and tshark"
tool: wireshark
category: network-analysis
difficulty: beginner
requires_root: true
---

# Wireshark Cheatsheet

## Quick Start

```bash
# Install Wireshark
sudo apt install wireshark

# Start Wireshark GUI
wireshark

# Start tshark capture
sudo tshark -i eth0

# Capture to file
sudo tshark -i eth0 -w capture.pcap

# Read from file
tshark -r capture.pcap
```

## Capture Filters (BPF)

```bash
# Filter by host
sudo tshark -i eth0 -f "host 192.168.1.100"

# Filter by port
sudo tshark -i eth0 -f "tcp port 80"
sudo tshark -i eth0 -f "udp port 53"

# Filter by protocol
sudo tshark -i eth0 -f "tcp"
sudo tshark -i eth0 -f "udp"
sudo tshark -i eth0 -f "icmp"
sudo tshark -i eth0 -f "arp"

# Complex filters
sudo tshark -i eth0 -f "host 192.168.1.100 and tcp port 80"
sudo tshark -i eth0 -f "not port 22"
```

## Display Filters

```bash
# Protocol filters
http
tcp
udp
dns
arp
icmp

# Host filters
ip.addr == 192.168.1.100
ip.src == 192.168.1.100
ip.dst == 192.168.1.100

# Port filters
tcp.port == 80
tcp.dstport == 443
udp.port == 53

# Combined filters
ip.addr == 192.168.1.100 and tcp.port == 80
http or dns
tcp.flags.syn == 1
```

## Analysis

```bash
# Verbose output
sudo tshark -i eth0 -V

# Hex dump
sudo tshark -i eth0 -x

# Quiet mode
sudo tshark -i eth0 -q

# Field extraction
sudo tshark -i eth0 -T fields -e ip.src -e ip.dst

# Statistics
sudo tshark -i eth0 -z io,stat,1
sudo tshark -i eth0 -z conv,tcp
```

## Common Flags

| Flag | Description | Example |
|------|-------------|---------|
| `-i` | Interface | `tshark -i eth0` |
| `-f` | Capture filter | `tshark -f "tcp port 80"` |
| `-Y` | Display filter | `tshark -Y "http"` |
| `-c` | Packet count | `tshark -c 100` |
| `-w` | Write pcap | `tshark -w capture.pcap` |
| `-r` | Read pcap | `tshark -r capture.pcap` |
| `-V` | Verbose | `tshark -V` |
| `-x` | Hex dump | `tshark -x` |
| `-q` | Quiet | `tshark -q` |
| `-T` | Output format | `tshark -T fields` |
| `-e` | Field | `tshark -e ip.src` |
| `-a` | Auto-stop | `tshark -a duration:300` |
| `-b` | Ring buffer | `tshark -b filesize:10000` |
| `-z` | Statistics | `tshark -z io,stat,1` |

## Display Filter Syntax

### Comparison Operators
```bash
==          # Equal
!=          # Not equal
>           # Greater than
<           # Less than
>=          # Greater than or equal
<=          # Less than or equal
contains    # Contains
matches     # Regex match
```

### Logical Operators
```bash
and (&&)    # Logical AND
or (||)     # Logical OR
not (!)     # Logical NOT
xor         # Logical XOR
```

### Common Fields
```bash
ip.addr        # IP address
ip.src         # Source IP
ip.dst         # Destination IP
tcp.port       # TCP port
tcp.srcport    # Source port
tcp.dstport    # Destination port
tcp.flags      # TCP flags
udp.port       # UDP port
dns.qry.name   # DNS query name
http.host      # HTTP host
http.request.method  # HTTP method
http.response.code   # HTTP response code
```

## Examples

### Basic Capture
```bash
# Capture on interface
sudo tshark -i eth0

# Capture 100 packets
sudo tshark -i eth0 -c 100

# Capture to file
sudo tshark -i eth0 -w capture.pcap
```

### Filtering Examples
```bash
# HTTP traffic
sudo tshark -i eth0 -Y "http"

# DNS traffic
sudo tshark -i eth0 -Y "dns"

# Specific host
sudo tshark -i eth0 -Y "ip.addr == 192.168.1.100"

# Specific port
sudo tshark -i eth0 -Y "tcp.port == 80"

# Combined
sudo tshark -i eth0 -Y "ip.addr == 192.168.1.100 and tcp.port == 80"
```

### Field Extraction
```bash
# Extract IP addresses
sudo tshark -i eth0 -T fields -e ip.src -e ip.dst

# Extract TCP ports
sudo tshark -i eth0 -T fields -e tcp.srcport -e tcp.dstport

# Extract HTTP data
sudo tshark -i eth0 -T fields -e http.host -e http.request.uri

# Extract DNS queries
sudo tshark -i eth0 -T fields -e dns.qry.name

# Extract TLS info
sudo tshark -i eth0 -T fields -e tls.handshake.type
```

### Statistics
```bash
# IO statistics
sudo tshark -i eth0 -z io,stat,1

# TCP conversations
sudo tshark -i eth0 -z conv,tcp

# HTTP statistics
sudo tshark -i eth0 -z http,stat

# DNS statistics
sudo tshark -i eth0 -z dns,stat

# Protocol hierarchy
sudo tshark -i eth0 -z io,phs
```

## Chaining with Other Tools

```bash
# Capture with tcpdump, analyze with Wireshark
tcpdump -i eth0 -w capture.pcap -c 1000
tshark -r capture.pcap -Y "http"

# Pipe to Wireshark
sudo tshark -i eth0 -w - | wireshark -k -

# Process with tcpflow
tcpflow -r capture.pcap -o /tmp/flows
tshark -r capture.pcap -Y "tcp.stream eq 1"

# Export to CSV
sudo tshark -i eth0 -T fields -e frame.time -e ip.src -e ip.dst -e tcp.port > output.csv
```

## Quick Reference

### Target IPs
- Gateway: `192.168.1.1`
- Target: `192.168.1.100`
- Subnet: `192.168.1.0/24`

### Common Ports
- FTP: 21
- SSH: 22
- Telnet: 23
- HTTP: 80
- HTTPS: 443
- DNS: 53
- DHCP: 67/68
- MySQL: 3306
- RDP: 3389

### TCP Flags
- `SYN` - Synchronize
- `ACK` - Acknowledgment
- `FIN` - Finish
- `RST` - Reset
- `PSH` - Push
- `URG` - Urgent

### Display Filter Examples
```bash
# HTTP errors
http.response.code >= 400

# DNS failures
dns.flags.rcode != 0

# TCP retransmissions
tcp.analysis.retransmission

# Large packets
frame.len > 1000

# Specific time range
frame.time >= "2024-01-01 00:00:00"
```

### Statistics Options
```bash
-z io,stat,1          # IO stats per second
-z conv,tcp           # TCP conversations
-z conv,udp           # UDP conversations
-z http,stat          # HTTP statistics
-z dns,stat           # DNS statistics
-z io,phs             # Protocol hierarchy
-z expert             # Expert info
```
