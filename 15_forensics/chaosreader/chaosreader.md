# chaosreader — TCP/UDP Session Analyzer

## Table of Contents

1. [Overview](#overview)
2. [How chaosreader Works](#how-chaosreader-works)
3. [Installation](#installation)
4. [Command Syntax](#command-syntax)
5. [Options Reference](#options-reference)
6. [Understanding the Output](#understanding-the-output)
7. [Analyzing Sessions](#analyzing-sessions)
8. [Practical Lab Exercises](#practical-lab-exercises)
9. [Integration with Other Tools](#integration-with-other-tools)
10. [Common Forensic Scenarios](#common-forensic-scenarios)
11. [Troubleshooting](#troubleshooting)
12. [Best Practices](#best-practices)
13. [Summary](#summary)

---

## Overview

`chaosreader` is a Perl script that extracts TCP and UDP sessions from **tcpdump** or **snoop** packet capture files. It produces HTML reports for each session, making it easy to browse and analyze network communications.

Unlike tcpflow which focuses on TCP stream reassembly, `chaosreader` provides a more comprehensive view of network sessions including:

- TCP and UDP sessions
- DNS queries and responses
- HTTP requests and responses
- SMTP, POP3, IMAP email sessions
- FTP control and data sessions
- Telnet sessions
- Various other protocols

### Key Features

- Generates HTML reports for easy browsing
- Extracts session data to separate files
- Creates timeline views of network activity
- Works with tcpdump and snoop capture formats
- Perl-based (no compilation required)

---

## How chaosreader Works

### Processing Pipeline

1. Reads tcpdump/snoop output files
2. Identifies TCP and UDP sessions
3. Extracts session data
4. Generates HTML reports
5. Creates separate data files for each session

### Session Detection

- **TCP Sessions**: Identified by SYN, SYN-ACK, FIN, RST packets
- **UDP Sessions**: Identified by source/destination IP and port pairs
- **DNS**: Parsed for query/response pairs
- **HTTP**: Parsed for request/response pairs

---

## Installation

### On Kali Linux

```bash
sudo apt update
sudo apt install chaosreader
```

### Manual Installation

```bash
# Download chaosreader
wget http://www.netometer.com/script/chaosreader.pl

# Make executable
chmod +x chaosreader.pl

# Move to PATH
sudo mv chaosreader.pl /usr/local/bin/chaosreader
```

### Dependencies

```bash
# Ensure Perl is installed
sudo apt install perl

# Optional: Install additional Perl modules
sudo apt install libnet-pcap-perl
```

---

## Command Syntax

```bash
chaosreader [options] <tcpdump_output_file>
```

### Basic Usage

```bash
# Process a tcpdump output file
chaosreader capture.out

# Process with specific options
chaosreader -n 5 capture.out

# Process all files in a directory
chaosreader *.out
```

### Creating Input Files

```bash
# Create tcpdump output
sudo tcpdump -i eth0 -w capture.pcap

# Convert to tcpdump text output
tcpdump -r capture.pcap > capture.out

# Or use tcpdump directly
sudo tcpdump -i eth0 > capture.out
```

---

## Options Reference

| Option | Description |
|--------|-------------|
| `-n <num>` | Number of packets to process |
| `-s` | Sort sessions by start time |
| `-t` | Print session table only |
| `-v` | Verbose output |
| `-h` | Help |

---

## Understanding the Output

### Directory Structure

```
chaosreader_output/
├── index.html              # Main session listing
├── session_0001.html       # Individual session report
├── session_0001.dat        # Raw session data
├── session_0002.html
├── session_0002.dat
├── tcp_session_0001.html   # TCP session details
├── tcp_session_0001.dat
├── udp_session_0001.html   # UDP session details
├── udp_session_0001.dat
├── dns_session_0001.html   # DNS session details
├── http_session_0001.html  # HTTP session details
└── ...
```

### Session Reports

Each HTML report contains:

- Session summary (IPs, ports, timestamps)
- Packet list with timing
- Hex dump of session data
- ASCII representation
- Protocol-specific parsing (for HTTP, DNS, etc.)

### Session Data Files

The `.dat` files contain the raw session data in hex dump format, which can be parsed with other tools.

---

## Analyzing Sessions

### Viewing the Index

```bash
# Open the main index in a browser
xdg-open chaosreader_output/index.html

# Or list sessions from command line
ls -la chaosreader_output/session_*.dat
```

### Analyzing HTTP Sessions

```bash
# Find HTTP sessions
grep -l "HTTP" chaosreader_output/session_*.html

# Extract HTTP data
cat chaosreader_output/http_session_*.dat | strings

# Search for specific content
grep -r "GET\|POST" chaosreader_output/
```

### Analyzing DNS Sessions

```bash
# Find DNS sessions
ls -la chaosreader_output/dns_session_*.dat

# Extract DNS queries
cat chaosreader_output/dns_session_*.dat | strings | grep -i "query\|response"
```

### Analyzing Email Sessions

```bash
# Find SMTP/POP3/IMAP sessions
grep -l "SMTP\|POP3\|IMAP" chaosreader_output/session_*.html

# Extract email content
cat chaosreader_output/session_*.dat | strings | grep -i "from:\|to:\|subject:"
```

---

## Practical Lab Exercises

### Lab 1: Basic Session Analysis

```bash
# Capture traffic
sudo tcpdump -i eth0 -c 1000 -w capture.pcap

# Convert to text format
tcpdump -r capture.pcap > capture.out

# Process with chaosreader
chaosreader capture.out

# View results
xdg-open chaosreader_output/index.html
```

### Lab 2: HTTP Session Analysis

```bash
# Capture HTTP traffic
sudo tcpdump -i eth0 port 80 -w http_capture.pcap

# Process
tcpdump -r http_capture.pcap > http_capture.out
chaosreader http_capture.out

# Review HTTP sessions
for f in chaosreader_output/http_session_*.html; do
    echo "=== $f ==="
    head -50 "$f"
done
```

### Lab 3: DNS Analysis

```bash
# Capture DNS traffic
sudo tcpdump -i eth0 port 53 -w dns_capture.pcap

# Process
tcpdump -r dns_capture.pcap > dns_capture.out
chaosreader dns_capture.out

# List all DNS queries
grep -h "Query\|Response" chaosreader_output/dns_session_*.html
```

### Lab 4: Email Forensics

```bash
# Capture email traffic
sudo tcpdump -i eth0 'port 25 or port 110 or port 143' -w email_capture.pcap

# Process
tcpdump -r email_capture.pcap > email_capture.out
chaosreader email_capture.out

# Extract email content
cat chaosreader_output/session_*.dat | strings | grep -B2 -A10 "Subject:\|From:\|To:"
```

### Lab 5: Full Network Analysis

```bash
# Capture all traffic
sudo tcpdump -i eth0 -w full_capture.pcap -c 10000

# Process
tcpdump -r full_capture.pcap > full_capture.out
chaosreader full_capture.out

# Review all sessions
ls -la chaosreader_output/
cat chaosreader_output/index.html
```

---

## Integration with Other Tools

### chaosreader + tcpdump

```bash
# Create capture
sudo tcpdump -i eth0 -w capture.pcap

# Convert and process
tcpdump -r capture.pcap > capture.out
chaosreader capture.out
```

### chaosreader + tcpflow

```bash
# Use chaosreader for session overview
chaosreader capture.out

# Use tcpflow for detailed stream reassembly
tcpflow -r capture.pcap -o tcpflow_output/
```

### chaosreader + Wireshark

```bash
# chaosreader for quick session overview
chaosreader capture.out

# Wireshark for detailed packet analysis
wireshark capture.pcap
```

### chaosreader + NetworkMiner

```bash
# chaosreader for session extraction
chaosreader capture.out

# NetworkMiner for file extraction and host analysis
networkminer -r capture.pcap -o networkminer_output/
```

---

## Common Forensic Scenarios

### Scenario 1: Identifying Suspicious Connections

```bash
# Process capture
chaosreader capture.out

# Look for unusual ports
grep -h "port" chaosreader_output/session_*.html | sort | uniq -c | sort -rn

# Check for long-duration sessions
ls -la chaosreader_output/session_*.dat | awk '{print $5, $9}' | sort -rn | head
```

### Scenario 2: Credential Extraction

```bash
# Process capture
chaosreader capture.out

# Search for credentials in sessions
grep -r -i "password\|passwd\|login\|user=" chaosreader_output/
grep -r "Authorization:" chaosreader_output/
```

### Scenario 3: Data Exfiltration Detection

```bash
# Process capture
chaosreader capture.out

# Find large sessions
ls -la chaosreader_output/session_*.dat | sort -k5 -rn | head

# Check for encoded data
cat chaosreader_output/session_*.dat | strings | grep -i "base64\|encode"
```

### Scenario 4: Malware C2 Analysis

```bash
# Process capture
chaosreader capture.out

# Look for beaconing patterns
ls -la chaosreader_output/session_*.dat

# Check for encoded commands
cat chaosreader_output/session_*.dat | strings | head -50
```

---

## Troubleshooting

### "Command not found"

```bash
# Install from repository
sudo apt install chaosreader

# Or download manually
wget http://www.netometer.com/script/chaosreader.pl
chmod +x chaosreader.pl
sudo mv chaosreader.pl /usr/local/bin/
```

### "Cannot read capture file"

```bash
# Ensure file is in tcpdump text format
tcpdump -r capture.pcap > capture.out

# Or use snoop format
snoop -r capture.snoop > capture.out
```

### No Sessions Found

```bash
# Check capture file has content
wc -l capture.out

# Verify traffic type
tcpdump -r capture.pcap -c 10

# Try with fewer packets
chaosreader -n 100 capture.out
```

### HTML Reports Not Opening

```bash
# Check if xdg-open is available
which xdg-open

# Open manually
xdg-open chaosreader_output/index.html
# Or
firefox chaosreader_output/index.html
```

---

## Best Practices

1. **Use tcpdump for capture** — creates compatible output files
2. **Process the full capture** — don't use `-n` unless necessary
3. **Review the index first** — get an overview of all sessions
4. **Check HTTP sessions** — most application data flows through HTTP
5. **Look for DNS sessions** — reveals C2 communication
6. **Extract email content** — often contains evidence
7. **Combine with tcpflow** — chaosreader for overview, tcpflow for detail
8. **Save HTML reports** — provides forensic documentation
9. **Search session data** — use grep/strings on .dat files
10. **Document findings** — note session numbers and timestamps

---

## Summary

`chaosreader` provides a comprehensive view of network sessions. Key capabilities:

- Extracts TCP and UDP sessions from captures
- Generates HTML reports for easy browsing
- Parses HTTP, DNS, SMTP, and other protocols
- Creates separate data files for each session
- Works with tcpdump and snoop formats

### Quick Reference

```bash
# Process capture
chaosreader capture.out

# With packet limit
chaosreader -n 1000 capture.out

# Sort by time
chaosreader -s capture.out

# View results
xdg-open chaosreader_output/index.html
```

---

## References

- chaosreader Documentation: http://www.netometer.com/script/chaosreader.html
- tcpdump Documentation: https://www.tcpdump.org/
- SANS Network Forensics: https://www.sans.org/network-forensics/
