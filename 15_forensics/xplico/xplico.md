# xplico — Network Forensics Analysis Tool

## Table of Contents

1. [Overview](#overview)
2. [How xplico Works](#how-xplico-works)
3. [Installation](#installation)
4. [Command Syntax](#command-syntax)
5. [Options Reference](#options-reference)
6. [Supported Protocols](#supported-protocols)
7. [Using the Web Interface](#using-the-web-interface)
8. [Using the Command Line](#using-the-command-line)
9. [Practical Lab Exercises](#practical-lab-exercises)
10. [Integration with Other Tools](#integration-with-other-tools)
11. [Common Forensic Scenarios](#common-forensic-scenarios)
12. [Troubleshooting](#troubleshooting)
13. [Best Practices](#best-practices)
14. [Summary](#summary)

---

## Overview

**Xplico** is a Network Forensic Analysis Tool (NFAT) that extracts application data from network traffic captures. Unlike protocol analyzers that show packets, Xplico extracts the actual content — emails, web pages, files, VoIP calls, and more.

Xplico is NOT a network protocol analyzer. It is an **application data extractor**. From a pcap file, Xplico extracts:

- **Emails** (POP, IMAP, SMTP)
- **Web content** (HTTP, HTTPS metadata)
- **VoIP calls** (SIP, MGCP, H.323)
- **FTP file transfers**
- **Instant messages** (IRC, MSN, Yahoo)
- **Images and files**
- **DNS queries**
- **And more**

### Key Features

- Web-based interface for easy browsing
- Command-line interface for automation
- Supports numerous protocols
- Automatically extracts files and content
- Provides timeline views
- Supports both live capture and pcap file analysis

---

## How xplico Works

### Architecture

Xplico uses a modular architecture with:

- **Decoders** — Protocol-specific modules that parse traffic
- **Manipulators** — Components that handle different capture types
- **Dema** — The core engine that orchestrates decoding
- **Web Interface** — PHP-based UI for browsing results

### Processing Pipeline

1. Input: pcap file or live capture
2. Protocol identification and separation
3. Protocol-specific decoding
4. Content extraction (files, emails, etc.)
5. Storage in SQLite database
6. Web interface for browsing

---

## Installation

### On Kali Linux

```bash
sudo apt update
sudo apt install xplico
```

### Start the Service

```bash
# Start Xplico web interface
sudo xplico-webui-start

# Access at http://127.0.0.1:9876
# Default credentials: xplico / xplico
```

### Verify Installation

```bash
xplico -v
```

---

## Command Syntax

### Command-Line Mode

```bash
xplico [options] -m <capture_module>
```

### Web Interface Mode

```bash
# Start the web UI
sudo xplico-webui-start

# Access via browser
# http://127.0.0.1:9876
```

---

## Options Reference

### xplico Command Options

| Option | Description |
|--------|-------------|
| `-v` | Display version |
| `-c config_file` | Specify configuration file |
| `-h` | Help |
| `-s` | Silent mode |
| `-g` | Display protocol graph tree |
| `-l` | Print all log to screen |
| `-i prot` | Show info about protocol |
| `-m module` | Capture module to use |

### Capture Modules

| Module | Description |
|--------|-------------|
| `rltm` | Real-time capture from interface |
| `pcapf` | Read from pcap file |
| `pcapd` | Pcap daemon mode |

### Manipulator Commands

| Command | Description |
|---------|-------------|
| `mfbc` | File-based capture manipulator |
| `mfile` | File manipulator |
| `mwmail` | Web mail manipulator |
| `mpaltalk` | Paltalk manipulator |

---

## Supported Protocols

### Application Protocols

| Protocol | Data Extracted |
|----------|----------------|
| HTTP | Web pages, images, files, cookies |
| POP3/POP3S | Emails, attachments |
| IMAP/IMAPS | Emails, attachments |
| SMTP/SMTPS | Sent emails |
| FTP/FTPS | Transferred files |
| DNS | Domain queries |
| SIP/VoIP | Voice calls |
| IRC | Chat messages |
| MSN/MSN | Instant messages |
| Yahoo Messenger | Instant messages |

### Transport Protocols

| Protocol | Support |
|----------|---------|
| TCP | Full support |
| UDP | Full support |
| ICMP | Basic support |

---

## Using the Web Interface

### Starting the Web UI

```bash
# Start Xplico service
sudo xplico-webui-start

# Access in browser
# URL: http://127.0.0.1:9876
# Username: xplico
# Password: xplico
```

### Creating a Case

1. Open the web interface
2. Click "New Case"
3. Enter case name and description
4. Click "Create"

### Importing Pcap Files

1. Select your case
2. Click "Add Pcap"
3. Browse to your pcap file
4. Select the appropriate decoder
5. Click "Import"

### Browsing Results

- **Sessions** — List of all decoded sessions
- **HTTP** — Web pages and content
- **Email** — Extracted emails
- **FTP** — Transferred files
- **VoIP** — Call recordings
- **Images** — Extracted images
- **Files** — All extracted files

### Exporting Data

1. Navigate to the desired section
2. Select items to export
3. Click "Export"
4. Choose format (files, zip, etc.)

---

## Using the Command Line

### Processing a Pcap File

```bash
# Create a configuration file
cat > xplico_cli.cfg << 'EOF'
[main]
path_pcap = /tmp/xplico
path_db = /tmp/xplico/db

[decoders]
pcapf = 1
EOF

# Process pcap file
xplico -m pcapf -i /evidence/capture.pcap -c xplico_cli.cfg
```

### Real-Time Capture

```bash
# Capture from interface
xplico -m rltm -i eth0 -c xplico_cli.cfg
```

### Using Manipulator Commands

```bash
# File-based capture
mfbc -p 9877 -c xplico_cli.cfg

# File manipulator
mfile -p 9878 -c xplico_cli.cfg
```

---

## Practical Lab Exercises

### Lab 1: Basic Pcap Analysis

```bash
# Start Xplico
sudo xplico-webui-start

# Access web interface
# Create new case
# Import sample pcap file
# Browse extracted content
```

### Lab 2: HTTP Content Extraction

```bash
# Capture HTTP traffic
sudo tcpdump -i eth0 port 80 -w http_traffic.pcap

# Start Xplico
sudo xplico-webui-start

# Import http_traffic.pcap
# Browse extracted web pages and images
```

### Lab 3: Email Extraction

```bash
# Capture email traffic
sudo tcpdump -i eth0 'port 25 or port 110 or port 143' -w email_traffic.pcap

# Import into Xplico
# Browse extracted emails
# Download attachments
```

### Lab 4: VoIP Analysis

```bash
# Capture SIP/RTP traffic
sudo tcpdump -i eth0 'port 5060 or portrange 10000-20000' -w voip_traffic.pcap

# Import into Xplico
# Browse call sessions
# Play extracted audio
```

### Lab 5: File Recovery from FTP

```bash
# Capture FTP traffic
sudo tcpdump -i eth0 'port 21 or port 20' -w ftp_traffic.pcap

# Import into Xplico
# Browse transferred files
# Download recovered files
```

---

## Integration with Other Tools

### Xplico + tcpdump

```bash
# Capture with tcpdump
sudo tcpdump -i eth0 -w capture.pcap

# Analyze with Xplico
sudo xplico-webui-start
# Import capture.pcap via web interface
```

### Xplico + tcpflow

```bash
# Use Xplico for comprehensive extraction
sudo xplico-webui-start
# Import capture.pcap

# Use tcpflow for specific stream analysis
tcpflow -r capture.pcap -o tcpflow_output/
```

### Xplico + Wireshark

```bash
# Xplico for content extraction
sudo xplico-webui-start
# Import capture.pcap

# Wireshark for packet details
wireshark capture.pcap
```

### Xplico + NetworkMiner

```bash
# Xplico for protocol-specific extraction
sudo xplico-webui-start
# Import capture.pcap

# NetworkMiner for file and host analysis
networkminer -r capture.pcap -o nm_output/
```

---

## Common Forensic Scenarios

### Scenario 1: Data Exfiltration

```bash
# Capture outbound traffic
sudo tcpdump -i eth0 'not port 22' -w exfil.pcap

# Import into Xplico
# Look for:
# - Large file transfers (FTP, HTTP uploads)
# - Email attachments
# - HTTP POST requests with data
```

### Scenario 2: Credential Theft

```bash
# Capture authentication traffic
sudo tcpdump -i eth0 'port 80 or port 443 or port 21' -w auth.pcap

# Import into Xplico
# Look for:
# - HTTP form submissions
# - FTP credentials
# - Email login attempts
```

### Scenario 3: Malware Communication

```bash
# Capture suspicious traffic
sudo tcpdump -i eth0 'not port 22 and not port 53' -w malware.pcap

# Import into Xplico
# Look for:
# - HTTP C2 communication
# - DNS tunneling
# - Unusual protocols
```

### Scenario 4: Web Activity Analysis

```bash
# Capture web traffic
sudo tcpdump -i eth0 'port 80 or port 443' -w web.pcap

# Import into Xplico
# Browse:
# - Visited websites
# - Downloaded files
# - Form submissions
```

---

## Troubleshooting

### "Service not starting"

```bash
# Check service status
sudo systemctl status xplico

# Check logs
sudo journalctl -u xplico

# Restart service
sudo xplico-webui-stop
sudo xplico-webui-start
```

### "Cannot access web interface"

```bash
# Verify service is running
sudo systemctl status xplico

# Check port
netstat -tlnp | grep 9876

# Try restarting
sudo xplico-webui-stop
sudo xplico-webui-start
```

### "No sessions decoded"

```bash
# Check pcap file
tcpdump -r capture.pcap -c 10

# Verify decoder selection
# Try different decoder in web interface
```

### "Database errors"

```bash
# Check SQLite database
ls -la /tmp/xplico/db/

# Remove and recreate
sudo rm -rf /tmp/xplico/db/*
sudo xplico-webui-stop
sudo xplico-webui-start
```

---

## Best Practices

1. **Use the web interface** — easiest way to browse results
2. **Create cases** — organize your analysis
3. **Check all protocol sections** — emails, HTTP, FTP, etc.
4. **Export important files** — download extracted content
5. **Combine with other tools** — Xplico for extraction, Wireshark for packets
6. **Document findings** — note session numbers and timestamps
7. **Check for hidden content** — DNS, ICMP, unusual ports
8. **Verify extracted files** — check file types and integrity
9. **Use command-line for automation** — script your analysis
10. **Save pcap files** — always keep the original capture

---

## Summary

Xplico is a powerful network forensic tool for extracting application data. Key capabilities:

- Extracts emails, web content, files, VoIP calls
- Web-based interface for easy browsing
- Supports numerous protocols
- Automatic content extraction
- Timeline views

### Quick Reference

```bash
# Start web interface
sudo xplico-webui-start

# Access
# http://127.0.0.1:9876
# xplico / xplico

# Stop service
sudo xplico-webui-stop

# Command-line processing
xplico -m pcapf -i capture.pcap -c config.cfg
```

---

## References

- Xplico Homepage: https://www.xplico.org
- Kali Linux Xplico: https://www.kali.org/tools/xplico/
- Network Forensics with Xplico
