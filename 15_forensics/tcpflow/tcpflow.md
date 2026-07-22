# tcpflow — TCP Flow Recorder

## Table of Contents

1. [Overview](#overview)
2. [How tcpflow Works](#how-tcpflow-works)
3. [Installation](#installation)
4. [Command Syntax](#command-syntax)
5. [Options Reference](#options-reference)
6. [Understanding Output Files](#understanding-output-files)
7. [Capturing Live Traffic](#capturing-live-traffic)
8. [Analyzing Pcap Files](#analyzing-pcap-files)
9. [HTTP Content Extraction](#http-content-extraction)
10. [DFXML Reports](#dfxml-reports)
11. [Practical Lab Exercises](#practical-lab-exercises)
12. [Integration with Other Tools](#integration-with-other-tools)
13. [Common Forensic Scenarios](#common-forensic-scenarios)
14. [Troubleshooting](#troubleshooting)
15. [Best Practices](#best-practices)
16. [Summary](#summary)

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

## How tcpflow Works

### Packet Capture and Reassembly

1. `tcpflow` reads packets from a network interface or pcap file
2. It identifies TCP flows by the 5-tuple: (src_ip, src_port, dst_ip, dst_port, protocol)
3. It reconstructs each flow using TCP sequence numbers
4. It handles retransmissions and out-of-order delivery
5. Each direction of each flow is stored in a separate file

### Output File Naming Convention

```
[timestampT]sourceip.sourceport-destip.destport[--VLAN][cNNNN]
```

| Component | Description |
|-----------|-------------|
| `timestampT` | Optional timestamp when first packet was seen |
| `sourceip` | Source IP address |
| `sourceport` | Source port number |
| `destip` | Destination IP address |
| `destport` | Destination port number |
| `--VLAN` | VLAN tag (if present) |
| `cNNNN` | Connection counter (for multiple connections) |

### Example Filenames

```
# Standard HTTP connection
128.129.130.131.00080-192.168.001.064.37314

# With timestamp
1325542703T128.129.130.131.02345-010.011.012.013.45103

# With VLAN tag
128.129.130.131.02345-010.011.012.013.45103--3

# Multiple connections with same endpoints
128.129.130.131.02345-010.011.012.013.45103c0005
```

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

The `bpf-filter` is a Berkeley Packet Filter expression (same as tcpdump).

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
| `-b` | Process in breadth-first mode (for memory efficiency) |
| `-c` | Console mode (print flow info to stdout) |
| `-e` | Print extra info about each packet |
| `-v` | Verbose mode |

### Output Control

| Option | Description |
|--------|-------------|
| `-T template` | Template for output filenames |
| `-F prefix` | Add prefix to output filenames |
| `-P pcap_file` | Write captured packets to pcap file |
| `-A` | Auto-interpret HTTP responses |
| `-a` | Post-process HTTP (create -HTTP files) |

### Advanced Options

| Option | Description |
|--------|-------------|
| `-d` | Debug mode |
| `-f` | Forensic mode (use filesystem timestamps) |
| `-S snaplen` | Set snapshot length |
| `-B` | Burst mode for high-speed networks |
| `-C config_file` | Use configuration file |
| `-x` | Do not purge old flows |
| `-w` | Write packet data to pcap |
| `-X file` | Write XML (DFXML) output |

### BPF Filter Expressions

```bash
# Capture HTTP traffic
tcpflow -i eth0 'port 80'

# Capture traffic from specific host
tcpflow -i eth0 'host 192.168.1.100'

# Capture traffic between two hosts
tcpflow -i eth0 'host 192.168.1.100 and host 10.0.0.1'

# Capture SMTP traffic
tcpflow -i eth0 'port 25 or port 587'

# Capture DNS traffic
tcpflow -i eth0 'port 53'

# Capture all traffic (except SSH)
tcpflow -i eth0 'not port 22'
```

---

## Understanding Output Files

### Standard Flow Files

Each TCP flow creates two files (one per direction):

```
192.168.1.100.45678-93.184.216.34.00080    # Client → Server
93.184.216.34.00080-192.168.1.100.45678    # Server → Client
```

The first file contains data sent FROM the client TO the server.
The second file contains data sent FROM the server TO the client.

### HTTP Post-Processing Files

With the `-a` flag, tcpflow creates additional files for HTTP connections:

```
93.184.216.34.00080-192.168.1.100.45678         # Raw stream
93.184.216.34.00080-192.168.1.100.45678-HTTP    # HTTP headers + body
93.184.216.34.00080-192.168.1.100.45678-HTTPBODY # Just the body
93.184.216.34.00080-192.168.1.100.45678-HTTPBODY-GZIP  # Decompressed body
```

### File Content Examples

```bash
# View raw HTTP request
cat 192.168.1.100.45678-93.184.216.34.00080

# Output:
# GET /index.html HTTP/1.1
# Host: www.example.com
# User-Agent: Mozilla/5.0
# Accept: text/html
# Connection: keep-alive

# View HTTP response
cat 93.184.216.34.00080-192.168.1.100.45678-HTTP

# Output:
# HTTP/1.1 200 OK
# Content-Type: text/html
# Content-Length: 1234
# 
# <html>
# <body>
# <h1>Hello World</h1>
# ...
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

# Capture SSH traffic
sudo tcpflow -i eth0 'port 22'
```

### Capture with Time Limit

```bash
# Use timeout to limit capture duration
sudo timeout 300 tcpflow -i eth0 -o /evidence/capture/

# Or use the -c flag for packet count
sudo tcpflow -i eth0 -c 10000
```

### Capture to Pcap and Flows

```bash
# Capture flows AND save pcap
sudo tcpflow -i eth0 -P /evidence/capture.pcap -o /evidence/flows/
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

### Merging Results

```bash
# Combine results from multiple captures
for dir in /analysis/*/; do
    cat "$dir"* > "/analysis/combined_$(basename $dir).txt"
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

### Extracting Files from HTTP

```bash
# Files transferred via HTTP are in the -HTTPBODY files
# Rename based on content type
for f in /evidence/web/*-HTTPBODY; do
    content_type=$(head -1 "$f" | grep -oP 'Content-Type: \K.*')
    case $content_type in
        *pdf*) cp "$f" "${f}.pdf" ;;
        *html*) cp "$f" "${f}.html" ;;
        *javascript*) cp "$f" "${f}.js" ;;
        *image*) cp "$f" "${f}.img" ;;
    esac
done
```

### Analyzing HTTPS (TLS)

Note: `tcpflow` cannot decrypt HTTPS traffic without the session keys. For HTTPS analysis:

1. Use Wireshark with SSLKEYLOGFILE
2. Use `sslkeylog` to extract keys
3. Decrypt pcap first, then process with tcpflow

---

## DFXML Reports

### Generating DFXML Output

```bash
# Generate XML (DFXML) output
tcpflow -r capture.pcap -X /evidence/flows.xml

# DFXML includes:
# - System information
# - All flow details
# - Source/destination IPs
# - Byte counts
# - MD5 hashes
# - Timestamps
```

### DFXML Structure

```xml
<dfxml xmloutputversion="1.0">
  <configuration>
    <tool name="tcpflow" version="1.5.0"/>
    <build>...</build>
  </configuration>
  <source>
    <image filename="capture.pcap"/>
  </source>
  <fileobject>
    <filename>192.168.1.100.45678-93.184.216.34.00080</filename>
    <filesize>12345</filesize>
    <hashdigest type="MD5">abc123...</hashdigest>
  </fileobject>
</dfxml>
```

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

### Lab 5: Malware C2 Traffic Analysis

```bash
# Capture suspicious traffic
sudo tcpflow -i eth0 'not port 22' -o /tmp/c2/

# Look for encoded/encrypted traffic
for f in /tmp/c2/*; do
    echo "=== $f ==="
    strings "$f" | head -20
    hexdump -C "$f" | head -20
done
```

---

## Integration with Other Tools

### tcpflow + Wireshark

```bash
# Capture flows and pcap simultaneously
sudo tcpflow -i eth0 -P /evidence/capture.pcap -o /evidence/flows/

# Open pcap in Wireshark for packet-level analysis
wireshark /evidence/capture.pcap

# Use tcpflow for stream reassembly
# Use Wireshark for packet details
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

### tcpflow + NetworkMiner

```bash
# NetworkMiner can parse tcpflow output
# Or use both tools independently
tcpflow -r capture.pcap -o /tmp/tcpflow/
networkminer -r capture.pcap -o /tmp/networkminer/
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

# Check for encoded data
strings /evidence/exfil/* | grep -E "base64|encode|compress"
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

### Scenario 4: Malware Communication

```bash
# Capture all traffic from suspicious host
sudo tcpflow -i eth0 'host 192.168.1.100' -o /evidence/malware/

# Look for beaconing patterns
for f in /evidence/malware/*; do
    echo "=== $f ==="
    strings "$f" | head -5
done

# Check for DNS tunneling
grep -r "dns" /evidence/malware/
```

### Scenario 5: VoIP Analysis

```bash
# Capture SIP/RTP traffic
sudo tcpflow -i eth0 'port 5060 or portrange 10000-20000' -o /evidence/voip/

# Use with VoIP analysis tools
# (SIPp, Wireshark VoIP analysis, etc.)
```

---

## Troubleshooting

### "No interfaces available"

```bash
# Check available interfaces
ip link show

# Try with explicit interface
sudo tcpflow -i eth0

# Or use "any" for all interfaces
sudo tcpflow -i any
```

### "Permission denied"

```bash
# tcpflow requires root for live capture
sudo tcpflow -i eth0

# Or add capabilities
sudo setcap cap_net_raw,cap_net_admin=eip /usr/bin/tcpflow
```

### No Output Files

```bash
# Check if traffic matches the filter
sudo tcpflow -i eth0 -c 10  # Capture 10 packets

# Verify with tcpdump
sudo tcpdump -i eth0 -c 10

# Check output directory permissions
ls -la /output/dir/
```

### Large Files, Slow Processing

```bash
# Use breadth-first mode for memory efficiency
tcpflow -b -r large.pcap

# Use filter to reduce traffic
tcpflow -r capture.pcap 'port 80'
```

### Cannot Read HTTPS

```bash
# tcpflow cannot decrypt TLS
# Use Wireshark with SSL keys:
# 1. Set SSLKEYLOGFILE environment variable
# 2. Capture traffic
# 3. Open in Wireshark with SSL key file
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
- Generates DXML forensic reports
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

# DXML output
tcpflow -r capture.pcap -X output.xml
```

---

## References

- tcpflow GitHub: https://github.com/simsong/tcpflow
- Simson Garfinkel, "Network Forensics with tcpflow"
- NIST Computer Forensics Tool Testing Program
- SANS Network Forensics: https://www.sans.org/network-forensics/
