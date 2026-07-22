# ChaosReader — TCP/UDP Session Analyzer

## Table of Contents

1. [Overview](#overview)
2. [Installation](#installation)
3. [Command Syntax](#command-syntax)
4. [Options Reference](#options-reference)
5. [Working with Pcap Files](#working-with-pcap-files)
6. [Session Analysis](#session-analysis)
7. [HTTP Session Extraction](#http-session-extraction)
8. [FTP Session Analysis](#ftp-session-analysis)
9. [SMTP and Email Analysis](#smtp-and-email-analysis)
10. [DNS Session Analysis](#dns-session-analysis)
11. [Practical Lab Exercises](#practical-lab-exercises)
12. [Integration with Other Tools](#integration-with-other-tools)
13. [Common Forensic Scenarios](#common-forensic-scenarios)
14. [Troubleshooting](#troubleshooting)
15. [Best Practices](#best-practices)
16. [Summary](#summary)

---

## Overview

ChaosReader is a Perl script that **extracts TCP/UDP sessions from tcpdump or snoop output** and generates HTML reports for each session. It provides a high-level overview of network traffic, making it ideal for quickly identifying interesting sessions in large pcap files.

### Key Features

- Extracts TCP and UDP sessions from pcap files
- Generates HTML reports for each session
- Identifies HTTP, FTP, SMTP, DNS, and other protocols
- Creates session summaries with timestamps, ports, and byte counts
- Supports both tcpdump and snoop output formats
- Color-coded output for easy identification

### When to Use ChaosReader

| Scenario | Why ChaosReader |
|----------|-----------------|
| Initial triage of large pcap | Quick overview of all sessions |
| Identifying interesting traffic | Session summaries highlight anomalies |
| HTTP content review | Extracts and displays HTTP content |
| Protocol identification | Automatically identifies common protocols |
| Forensic documentation | HTML reports for court/presentation |

---

## Installation

### On Kali Linux

```bash
# ChaosReader may not be in default repos
# Download directly
wget http://www.snoopy.nl/files/chaosreaderv0.94
chmod +x chaosreaderv0.94
sudo mv chaosreaderv0.94 /usr/local/bin/chaosreader
```

### Verify Installation

```bash
chaosreader --version
# or
perl chaosreaderv0.94 --help
```

### Dependencies

```bash
# Ensure required tools are installed
sudo apt install tcpdump tshark perl
```

---

## Command Syntax

```bash
chaosreader [options] <pcap_file>
```

### Basic Usage

```bash
# Analyze a pcap file
chaosreader capture.pcap

# Analyze with specific output directory
chaosreader -output /analysis/ capture.pcap

# Analyze with time filter
chaosreader -time "2024-01-15 10:00:00" capture.pcap
```

---

## Options Reference

| Option | Description |
|--------|-------------|
| `-output DIR` | Set output directory for reports |
| `-time TIME` | Filter sessions by time |
| `-nosort` | Don't sort sessions by time |
| `-nogreen` | Disable green color coding |
| `-nopointer` | Don't create pointer HTML files |
| `-nolocal` | Don't create local HTML files |
| `-nofile` | Don't create individual session files |
| `-pcapsuffix SUFFIX` | Set pcap file suffix |
| `-mactime` | Generate mactime-compatible output |

---

## Working with Pcap Files

### Basic Analysis

```bash
# Analyze a pcap file
chaosreader capture.pcap

# This creates:
# - A main HTML index page
# - Individual HTML files for each session
# - Raw data files for each session
# - Summary statistics
```

### Output Structure

```
capture/
├── index.html          # Main overview page
├── session_00001.html  # Individual session report
├── session_00001.raw   # Raw session data
├── session_00002.html
├── session_00002.raw
└── ...
```

### Processing Large Files

```bash
# For large pcaps, use time filtering
chaosreader -time "2024-01-15 10:00:00-11:00:00" large_capture.pcap

# Or use tcpdump to filter first
tcpdump -r large.pcap 'port 80' -w http_only.pcap
chaosreader http_only.pcap
```

---

## Session Analysis

### Understanding Session Reports

Each session HTML report contains:

1. **Session Header**: Source/dest IPs, ports, protocol, timestamps
2. **Packet List**: All packets in the session with details
3. **Packet Detail**: Hex dump of each packet
4. **Follow Stream**: Reassembled data stream
5. **Statistics**: Byte counts, timing, retransmissions

### Session Types

| Type | Description | Indicators |
|------|-------------|------------|
| HTTP | Web traffic | Port 80/443, GET/POST requests |
| FTP | File transfer | Port 20/21, USER/PASS commands |
| SMTP | Email sending | Port 25/587, MAIL FROM/RCPT TO |
| DNS | Name resolution | Port 53, query/response pairs |
| SSH | Secure shell | Port 22, encrypted traffic |
| Telnet | Remote login | Port 23, plaintext credentials |

---

## HTTP Session Extraction

### Viewing HTTP Sessions

```bash
# After running chaosreader, open the HTML index
# Click on HTTP sessions to view:
# - Request headers
# - Response headers
# - Response body (HTML, images, etc.)
# - Cookies and authentication data
```

### Extracting HTTP Content

```bash
# Find all HTTP sessions in the output
grep -l "HTTP" capture/session_*.html

# Extract URLs from sessions
grep -oP 'GET \K[^ ]+' capture/session_*.raw | sort -u

# Extract POST data
grep -A 10 "POST" capture/session_*.raw
```

---

## FTP Session Analysis

### Identifying FTP Sessions

```bash
# FTP sessions typically show:
# - USER and PASS commands
# - RETR (download) and STOR (upload) commands
# - Directory listings (LIST)

# Find FTP sessions in chaosreader output
grep -l "USER\|PASS\|RETR\|STOR" capture/session_*.raw
```

### Extracting Transferred Files

```bash
# After identifying FTP data channels
# Files are in the raw session files
# Use foremost or scalpel to carve files
for f in capture/ftp_*.raw; do
    foremost -i "$f" -o "carved_$(basename $f)/"
done
```

---

## SMTP and Email Analysis

### Analyzing Email Traffic

```bash
# SMTP sessions contain:
# - MAIL FROM: (sender)
# - RCPT TO: (recipients)
# - DATA (email content)
# - Attachments (MIME encoded)

# Find email sessions
grep -l "MAIL FROM\|RCPT TO" capture/session_*.raw

# Extract email content
grep -A 100 "DATA" capture/session_*.raw | head -100
```

### Extracting Attachments

```bash
# Look for MIME boundaries in email data
grep -B 5 -A 50 "boundary=" capture/session_*.raw

# Save attachments
# (Manual extraction from MIME-encoded content)
```

---

## DNS Session Analysis

### Identifying DNS Queries

```bash
# DNS sessions show query/response pairs
# Look for domain names in DNS traffic

# Find DNS sessions
grep -l "domain\|query\|response" capture/session_*.raw

# Extract queried domains
grep -oP '[a-z0-9]+\.[a-z]{2,}' capture/dns_*.raw | sort -u
```

---

## Practical Lab Exercises

### Lab 1: Basic Pcap Analysis

```bash
# Download sample pcap
wget https://wiki.wireshark.org/uploads/__moin_import__/attachments/SampleCaptures/http.cap

# Run chaosreader
chaosreader http.cap

# Open index.html in browser
# Review each session
```

### Lab 2: Web Traffic Analysis

```bash
# Capture web traffic
sudo tcpdump -i eth0 port 80 -w web_traffic.pcap

# Analyze with chaosreader
chaosreader web_traffic.pcap

# Review all HTTP sessions
# Extract URLs, POST data, cookies
```

### Lab 3: Credential Extraction

```bash
# Capture login traffic
sudo tcpdump -i eth0 'port 80 or port 21 or port 23' -w auth.pcap

# Analyze with chaosreader
chaosreader auth.pcap

# Search for credentials
grep -i "password\|passwd\|login" capture/session_*.raw
```

---

## Integration with Other Tools

### ChaosReader + tcpflow

```bash
# Use chaosreader for overview, tcpflow for deep analysis
chaosreader capture.pcap
tcpflow -a -r capture.pcap -o tcpflow_output/
```

### ChaosReader + Wireshark

```bash
# Use chaosreader to identify interesting sessions
# Then open specific packets in Wireshark for detailed analysis
chaosreader capture.pcap
# Note session timestamps and use in Wireshark filters
```

### ChaosReader + tcpdump

```bash
# Create filtered pcap first
tcpdump -r large.pcap 'port 80 or port 443' -w web_only.pcap
chaosreader web_only.pcap
```

---

## Common Forensic Scenarios

### Scenario 1: Data Exfiltration Detection

```bash
# Analyze all traffic
chaosreader capture.pcap

# Look for large outbound transfers
grep -l "Content-Length" capture/session_*.raw | while read f; do
    size=$(wc -c < "$(echo $f | sed 's/\.raw/-HTTPBODY/')")
    echo "$size $f"
done | sort -rn | head -10
```

### Scenario 2: Web Application Attack

```bash
# Find SQL injection attempts
grep -l "union\|select\|insert\|update\|delete" capture/session_*.raw

# Find XSS attempts
grep -l "<script\|javascript:" capture/session_*.raw

# Review those sessions in detail
```

### Scenario 3: Malware C2 Traffic

```bash
# Identify beaconing (regular interval connections)
chaosreader capture.pcap

# Look for unusual patterns in session timing
grep "Duration:" capture/session_*.html | sort -t: -k2 -n
```

---

## Troubleshooting

### "No sessions found"

- Verify pcap file is valid: `tcpdump -r capture.pcap -c 10`
- Check file permissions
- Ensure pcap contains TCP/UDP traffic (not just ICMP)

### "HTML files not displaying"

- Open index.html in a web browser
- Check that all session files are in the same directory
- Verify JavaScript is enabled in browser

### "Large output directory"

- Use time filtering: `chaosreader -time "start-end" capture.pcap`
- Filter pcap first: `tcpdump -r large.pcap 'port 80' -w filtered.pcap`

---

## Best Practices

1. **Use as first step** — chaosreader gives quick overview before deep analysis
2. **Filter pcap first** — reduce noise with tcpdump filters
3. **Combine with tcpflow** — chaosreader for overview, tcpflow for streams
4. **Export results** — HTML reports are great for documentation
5. **Check timestamps** — use time filtering for large captures
6. **Review all protocols** — don't just focus on HTTP
7. **Save session data** — raw files may contain evidence
8. **Document findings** — screenshot important sessions
9. **Cross-reference** — compare with other tools' findings
10. **Archive pcaps** — keep original evidence intact

---

## Summary

ChaosReader excels at providing a high-level overview of network traffic through HTML session reports. Key strengths:

- Quick triage of large pcap files
- Automatic protocol identification
- Session-level analysis and reporting
- Easy-to-read HTML output
- Integration with other forensic tools

### Quick Reference

```bash
# Basic analysis
chaosreader capture.pcap

# With output directory
chaosreader -output /analysis/ capture.pcap

# Time-filtered
chaosreader -time "2024-01-15 10:00:00" capture.pcap

# Process filtered pcap
tcpdump -r large.pcap 'port 80' -w filtered.pcap
chaosreader filtered.pcap
```

---

## References

- ChaosReader: http://www.snoopy.nl/
- Network Forensics with ChaosReader
- SANS Network Forensics: https://www.sans.org/network-forensics/
- tcpdump Documentation: https://www.tcpdump.org/
