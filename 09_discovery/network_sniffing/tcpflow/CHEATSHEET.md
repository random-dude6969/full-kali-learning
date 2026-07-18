---
title: "tcpflow Cheatsheet"
description: "Quick reference guide for tcpflow TCP flow recorder"
tool: tcpflow
category: network-forensics
difficulty: beginner
requires_root: true
---

# tcpflow Cheatsheet

## Quick Start

```bash
# Install tcpflow
sudo apt install tcpflow

# Capture flows
sudo tcpflow -i eth0

# Capture to directory
sudo tcpflow -i eth0 -o /tmp/flows

# Read from pcap
tcpflow -r capture.pcap -o /tmp/flows

# Capture HTTP traffic
sudo tcpflow -i eth0 'tcp port 80' -o /tmp/http
```

## Capture Filters

```bash
# Filter by host
sudo tcpflow -i eth0 'host 192.168.1.100'

# Filter by port
sudo tcpflow -i eth0 'tcp port 80'
sudo tcpflow -i eth0 'tcp port 443'

# Filter by subnet
sudo tcpflow -i eth0 'net 192.168.1.0/24'

# Complex filters
sudo tcpflow -i eth0 'host 192.168.1.100 and tcp port 80'
sudo tcpflow -i eth0 'tcp port 80 or tcp port 443'
```

## Analysis

```bash
# Verbose output
sudo tcpflow -i eth0 -v

# More verbose
sudo tcpflow -i eth0 -vv

# Quiet mode
sudo tcpflow -i eth0 -q

# List captured flows
ls -la /tmp/flows/

# View flow content
cat /tmp/flows/*:80
```

## Common Flags

| Flag | Description | Example |
|------|-------------|---------|
| `-i` | Interface | `tcpflow -i eth0` |
| `-o` | Output dir | `tcpflow -o /tmp/flows` |
| `-r` | Read pcap | `tcpflow -r capture.pcap` |
| `-c` | Packet count | `tcpflow -c 1000` |
| `-v` | Verbose | `tcpflow -v` |
| `-vv` | More verbose | `tcpflow -vv` |
| `-q` | Quiet | `tcpflow -q` |
| `-X` | XML output | `tcpflow -X` |
| `-z` | Gzip compress | `tcpflow -z` |
| `-s` | Max file size | `tcpflow -s 1048576` |
| `-S` | No flow files | `tcpflow -S` |
| `-E` | No extract | `tcpflow -E` |
| `-R` | Report file | `tcpflow -R report.txt` |
| `-B` | Buffer size | `tcpflow -B 4096` |
| `-m` | Memory limit | `tcpflow -m 268435456` |

## Output Structure

```
<output_dir>/
├── 192.168.1.100.44362-192.168.1.1.80    # HTTP flow
├── 192.168.1.100.51234-192.168.1.1.443   # HTTPS flow
└── 192.168.1.100.12345-192.168.1.1.21    # FTP flow
    ├── file1.txt
    ├── file2.pdf
    └── ...
```

## Examples

### Basic Capture
```bash
# Capture all flows
sudo tcpflow -i eth0 -o /tmp/flows

# Capture specific count
sudo tcpflow -i eth0 -c 1000 -o /tmp/flows

# Capture with compression
sudo tcpflow -i eth0 -z -o /tmp/flows
```

### Protocol-Specific
```bash
# HTTP traffic
sudo tcpflow -i eth0 'tcp port 80' -o /tmp/http

# FTP traffic
sudo tcpflow -i eth0 'tcp port 21' -o /tmp/ftp

# SMTP traffic
sudo tcpflow -i eth0 'tcp port 25' -o /tmp/mail

# SSH traffic
sudo tcpflow -i eth0 'tcp port 22' -o /tmp/ssh
```

### Forensics
```bash
# Read from evidence pcap
tcpflow -r evidence.pcap -o /tmp/forensics

# Generate XML report
tcpflow -r capture.pcap -X -o /tmp/analysis

# Extract files only
tcpflow -r capture.pcap -S -E -o /tmp/files
```

## Chaining with Other Tools

```bash
# Capture with tcpdump, analyze with tcpflow
tcpdump -i eth0 -w capture.pcap -c 1000
tcpflow -r capture.pcap -o /tmp/flows

# Analyze extracted files
find /tmp/flows -type f -exec file {} \;

# Extract HTTP content
grep -h "GET\|POST" /tmp/flows/*:80 | awk '{print $2}' | sort -u

# Use with chaosreader
chaosreader capture.pcap

# Process with tshark
tshark -r capture.pcap -T fields -e ip.src -e ip.dst -e tcp.stream
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
- SMTP: 25
- POP3: 110
- IMAP: 143

### Flow File Naming
```
<src_ip>.<src_port>-<dst_ip>.<dst_port>
```

### Output Directories
- Default: `./output/`
- Custom: `/tmp/flows/`
- Forensics: `/var/forensics/`

### File Sizes
- Small: `1048576` (1MB)
- Medium: `10485760` (10MB)
- Large: `104857600` (100MB)
- Unlimited: `0`
