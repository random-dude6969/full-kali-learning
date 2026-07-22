# tcpflow — TCP Flow Recorder

## Table of Contents

1. [Overview](#overview)
2. [Installation](#installation)
3. [Command Syntax](#command-syntax)
4. [Options Reference](#options-reference)
5. [Understanding Output Files](#understanding-output-files)
6. [Capturing Live Traffic](#capturing-live-traffic)
7. [Analyzing Pcap Files](#analyzing-pcap-files)
8. [HTTP Content Extraction](#http-content-extraction)
9. [Practical Lab Exercises](#practical-lab-exercises)
10. [Integration with Other Tools](#integration-with-other-tools)
11. [Common Forensic Scenarios](#common-forensic-scenarios)
12. [Troubleshooting](#troubleshooting)
13. [Best Practices](#best-practices)
14. [Summary](#summary)

---

## Overview

`tcpflow` is a TCP/IP packet demultiplexer that **captures data transmitted as part of TCP connections** and stores the data in a way that is convenient for protocol analysis and debugging. Developed by Simson Garfinkel, tcpflow reconstructs complete TCP streams from captured packets.

Unlike Wireshark or tcpdump which show individual packets, `tcpflow` **reassembles the actual data streams** and stores each flow in separate files. This makes it invaluable for:

- Reconstructing web sessions (HTTP content)
- Recovering transferred files
- Extracting email content
- Analyzing malware command-and-control traffic
- Reconstructing chat sessions
- Network forensics investigations

### tcpflow vs Other Network Tools

| Tool | Purpose | Output |
|------|---------|--------|
| tcpdump | Packet capture | Raw packets / pcap |
| Wireshark | Packet analysis | Interactive GUI |
| tcpflow | Stream reassembly | Reconstructed data files |
| ngrep | Pattern matching in packets | Matching packets |

---

## Installation

### On Kali Linux

```bash
sudo apt update
sudo apt install tcpflow
```

### Verify Installation

```bash
tcpflow -V
```

### Building from Source

```bash
git clone --recursive https://github.com/simsong/tcpflow.git
cd tcpflow
bash bootstrap.sh
./configure
make
sudo make install
```

---

## Command Syntax

```bash
tcpflow [options] [bpf-filter]
```

### Basic Usage

```bash
# Capture all TCP traffic
sudo tcpflow -i eth0

# Capture on specific port
sudo tcpflow -i eth0 port 80

# Read from pcap file
tcpflow -r capture.pcap

# Read and write to specific directory
tcpflow -r capture.pcap -o /evidence/flows/
```

---

## Options Reference

### Core Options

| Option | Description |
|--------|-------------|
| `-i interface` | Capture on specified network interface |
| `-r pcap_file` | Read from pcap file instead of live capture |
| `-o output_dir` | Write output to specified directory |
| `-b` | Process in breadth-first mode |
| `-c` | Console mode |
| `-v` | Verbose mode |

### Output Control

| Option | Description |
|--------|-------------|
| `-T template` | Template for output filenames |
| `-F prefix` | Add prefix to output filenames |
| `-P pcap_file` | Write captured packets to pcap file |
| `-A` | Auto-interpret HTTP responses |
| `-a` | Post-process HTTP |

### Advanced Options

| Option | Description |
|--------|-------------|
| `-d` | Debug mode |
| `-f` | Forensic mode |
| `-S snaplen` | Set snapshot length |
| `-B` | Burst mode for high-speed networks |
| `-x` | Do not purge old flows |
| `-w` | Write packet data to pcap |
| `-X file` | Write XML (DFXML) output |

### BPF Filter Expressions

```bash
# Capture HTTP traffic
tcpflow -i eth0 'port 80'

# Capture traffic from specific host
tcpflow -i eth0 'host 192.168.1.100'

# Capture SMTP traffic
tcpflow -i eth0 'port 25 or port 587'

# Capture DNS traffic
tcpflow -i eth0 'port 53'

# Capture all traffic except SSH
tcpflow -i eth0 'not port 22'
```

---

## Understanding Output Files

### Standard Flow Files

Each TCP flow creates two files (one per direction):

```
192.168.1.100.45678-93.184.216.34.00080    # Client to Server
93.184.216.34.00080-192.168.1.100.45678    # Server to Client
```

### HTTP Post-Processing Files

With the `-a` flag, tcpflow creates additional files:

```
93.184.216.34.00080-192.168.1.100.45678         # Raw stream
93.184.216.34.00080-192.168.1.100.45678-HTTP    # HTTP headers + body
93.184.216.34.00080-192.168.1.100.45678-HTTPBODY # Just the body
```

---

## Capturing Live Traffic

### Basic Capture

```bash
# Capture all TCP traffic on eth0
sudo tcpflow -i eth0

# Capture HTTP traffic
sudo tcpflow -i eth0 port 80

# Capture with output directory
sudo tcpflow -i eth0 -o /evidence/capture/
```

### Capture with Filtering

```bash
# Capture traffic to/from specific host
sudo tcpflow -i eth0 'host 192.168.1.100'

# Capture SMTP traffic
sudo tcpflow -i eth0 'port 25 or port 587 or port 465'

# Capture FTP traffic
sudo tcpflow -i eth0 'port 21 or port 20'
```

### Capture with Time Limit

```bash
# Use timeout to limit capture duration
sudo timeout 300 tcpflow -i eth0 -o /evidence/capture/

# Or use the -c flag for packet count
sudo tcpflow -i eth0 -c 10000
```

---

## Analyzing Pcap Files

### Reading Existing Captures

```bash
# Analyze a pcap file
tcpflow -r capture.pcap

# Analyze with output directory
tcpflow -r capture.pcap -o /analysis/

# Analyze with HTTP post-processing
tcpflow -a -r capture.pcap -o /analysis/

# Analyze large pcap file
tcpflow -b -r large_capture.pcap -o /analysis/
```

### Processing Multiple Pcap Files

```bash
# Process all pcap files in a directory
for pcap in /evidence/*.pcap; do
    echo "Processing: $pcap"
    tcpflow -r "$pcap" -o "/analysis/$(basename $pcap .pcap)/"
done
```

---

## HTTP Content Extraction

### Automatic HTTP Processing

```bash
# Enable HTTP auto-interpretation
tcpflow -a -r capture.pcap -o /analysis/

# This creates:
# - Flow files (raw streams)
# - -HTTP files (HTTP headers + body)
# - -HTTPBODY files (body only)
# - -HTTPBODY-GZIP files (decompressed)
```

### Extracting Web Pages

```bash
# Capture and extract web content
sudo tcpflow -a -i eth0 port 80 -o /evidence/web/

# View extracted HTML
cat /evidence/web/*-HTTP | less

# Find specific content
grep -l "search_term" /evidence/web/*-HTTPBODY
```

### Analyzing HTTPS (TLS)

Note: `tcpflow` cannot decrypt HTTPS traffic without the session keys. For HTTPS analysis:

1. Use Wireshark with SSLKEYLOGFILE
2. Use `sslkeylog` to extract keys
3. Decrypt pcap first, then process with tcpflow

---

## Practical Lab Exercises

### Lab 1: Basic HTTP Capture

```bash
# Start a simple web server on target
python3 -m http.server 8080 &

# Capture HTTP traffic
sudo tcpflow -i lo port 8080 -o /tmp/flows/

# Make a request
curl http://localhost:8080/index.html

# Examine captured flows
ls -la /tmp/flows/
cat /tmp/flows/*-HTTP
```

### Lab 2: Analyzing a Pcap File

```bash
# Download a sample pcap
wget https://wiki.wireshark.org/uploads/__moin_import__/attachments/SampleCaptures/http.cap

# Analyze with tcpflow
tcpflow -a -r http.cap -o /tmp/analysis/

# View results
ls -la /tmp/analysis/
cat /tmp/analysis/*-HTTP
```

### Lab 3: File Recovery from Network Capture

```bash
# Capture FTP file transfer
sudo tcpflow -i eth0 port 21 or port 20 -o /tmp/ftp/

# Or analyze existing pcap
tcpflow -r ftp_transfer.pcap -o /tmp/ftp/

# Recover transferred files
for f in /tmp/ftp/*; do
    file "$f"
    strings "$f" | head -20
done
```

### Lab 4: Email Reconstruction

```bash
# Capture SMTP traffic
sudo tcpflow -i eth0 'port 25 or port 587' -o /tmp/email/

# Analyze captured email
cat /tmp/email/* | grep -i "from:\|to:\|subject:\|content-type:"

# Extract attachments (look for MIME boundaries)
cat /tmp/email/* | grep -A 50 "boundary="
```

---

## Integration with Other Tools

### tcpflow + Wireshark

```bash
# Capture flows and pcap simultaneously
sudo tcpflow -i eth0 -P /evidence/capture.pcap -o /evidence/flows/

# Open pcap in Wireshark for packet-level analysis
wireshark /evidence/capture.pcap
```

### tcpflow + tcpdump

```bash
# Use tcpdump for capture, tcpflow for analysis
sudo tcpdump -i eth0 -w /evidence/capture.pcap 'port 80'

# Then analyze with tcpflow
tcpflow -a -r /evidence/capture.pcap -o /analysis/
```

### tcpflow + foremost/scalpel

```bash
# Extract files from network flows
tcpflow -r capture.pcap -o /tmp/flows/

# Carve files from flow data
for f in /tmp/flows/*; do
    foremost -i "$f" -o "/tmp/carved/$(basename $f)/"
done
```

### tcpflow + bulk_extractor

```bash
# Extract email addresses, URLs, etc. from flows
tcpflow -r capture.pcap -o /tmp/flows/

for f in /tmp/flows/*; do
    bulk_extractor "$f" -o "/tmp/be/$(basename $f)/"
done
```

---

## Common Forensic Scenarios

### Scenario 1: Data Exfiltration

```bash
# Capture outbound traffic
sudo tcpflow -i eth0 'not port 22 and not port 53' -o /evidence/exfil/

# Look for large transfers
for f in /evidence/exfil/*; do
    size=$(wc -c < "$f")
    echo "$size $f"
done | sort -rn | head -20
```

### Scenario 2: Credential Theft

```bash
# Capture authentication traffic
sudo tcpflow -i eth0 'port 80 or port 443 or port 21 or port 25' -o /evidence/auth/

# Search for credentials
grep -r -i "password\|passwd\|login\|user=" /evidence/auth/
grep -r -i "Authorization:" /evidence/auth/
```

### Scenario 3: Web Activity Analysis

```bash
# Capture web traffic
sudo tcpflow -a -i eth0 'port 80 or port 443' -o /evidence/web/

# List all accessed URLs
grep -h "GET \|POST " /evidence/web/*-HTTP | awk '{print $2}' | sort | uniq -c | sort -rn

# Find downloaded files
ls -la /evidence/web/*-HTTPBODY | sort -k5 -rn
```

---

## Troubleshooting

### "No interfaces available"

```bash
ip link show
sudo tcpflow -i eth0
```

### "Permission denied"

```bash
sudo tcpflow -i eth0
# Or add capabilities
sudo setcap cap_net_raw,cap_net_admin=eip /usr/bin/tcpflow
```

### No Output Files

```bash
sudo tcpflow -i eth0 -c 10  # Capture 10 packets
sudo tcpdump -i eth0 -c 10  # Verify traffic exists
```

### Cannot Read HTTPS

```bash
# tcpflow cannot decrypt TLS
# Use Wireshark with SSL keys instead
```

---

## Best Practices

1. **Always capture to pcap AND flows** — `-P` saves packets for later analysis
2. **Use filters to reduce noise** — focus on relevant traffic
3. **Use `-a` for HTTP analysis** — automatic content extraction
4. **Use `-b` for large captures** — prevents memory issues
5. **Document your capture parameters** — interface, filter, duration
6. **Verify capture integrity** — check file sizes and flow counts
7. **Combine with Wireshark** — tcpflow for streams, Wireshark for packets
8. **Search flow content** — use `strings`, `grep`, `srch_strings`
9. **Save DFXML output** — provides forensic documentation
10. **Work with copies** — never analyze on the live system

---

## Summary

`tcpflow` is essential for TCP stream reassembly and network forensics. Key capabilities:

- Reassembles TCP streams from packets
- Extracts HTTP content automatically
- Generates DFXML forensic reports
- Works with live capture and pcap files
- Handles retransmissions and out-of-order delivery

### Quick Reference

```bash
# Live capture
sudo tcpflow -i eth0 -o /evidence/

# Analyze pcap
tcpflow -a -r capture.pcap -o /analysis/

# HTTP analysis
tcpflow -a -i eth0 port 80 -o /web/

# Save pcap too
sudo tcpflow -i eth0 -P capture.pcap -o /flows/

# DFXML output
tcpflow -r capture.pcap -X output.xml
```

---

## References

- tcpflow GitHub: https://github.com/simsong/tcpflow
- Simson Garfinkel, "Network Forensics with tcpflow"
- NIST Computer Forensics Tool Testing Program
- SANS Network Forensics: https://www.sans.org/network-forensics/
