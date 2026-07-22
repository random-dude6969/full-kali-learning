# Xplico — Network Forensics Analysis Tool

## Table of Contents

1. [Overview](#overview)
2. [Installation](#installation)
3. [Command Syntax](#command-syntax)
4. [Web Interface Usage](#web-interface-usage)
5. [Decoding Protocols](#decoding-protocols)
6. [Analyzing Captured Data](#analyzing-captured-data)
7. [Practical Lab Exercises](#practical-lab-exercises)
8. [Integration with Other Tools](#integration-with-other-tools)
9. [Common Forensic Scenarios](#common-forensic-scenarios)
10. [Troubleshooting](#troubleshooting)
11. [Best Practices](#best-practices)
12. [Summary](#summary)

---

## Overview

Xplico is a **network forensics analysis tool** that decodes network captures (pcap files) and extracts application data. It parses pcap files and reconstructs web pages, emails, files, images, and other data transferred over the network.

### Key Features

- Decodes HTTP, SMTP, POP3, IMAP, FTP, DNS, and other protocols
- Reconstructs web pages with images and resources
- Extracts email content and attachments
- Recovers transferred files
- Provides both CLI and web interface
- Supports multiple pcap file formats

### When to Use Xplico

| Scenario | Why Xplico |
|----------|------------|
| Web traffic reconstruction | Rebuilds complete web pages |
| Email recovery | Extracts emails with attachments |
| File recovery | Recovers transferred files |
| Protocol analysis | Deep protocol decoding |
| Forensic investigation | Comprehensive traffic analysis |

---

## Installation

### On Kali Linux

```bash
sudo apt update
sudo apt install xplico
```

### Docker Installation

```bash
docker pull xplico/xplico
docker run -it -p 8080:8080 xplico/xplico
```

### Web Interface Setup

```bash
# Start the web interface
sudo systemctl start apache2
sudo systemctl start mysql

# Access at http://localhost/xplico
```

---

## Command Syntax

### Basic Usage

```bash
# Decode a pcap file
xplico -f capture.pcap

# Decode with output directory
xplico -f capture.pcap -o /output/

# Decode multiple files
xplico -d /pcaps/ -o /output/
```

### Command Line Options

| Option | Description |
|--------|-------------|
| `-f file` | Input pcap file |
| `-o dir` | Output directory |
| `-d dir` | Directory of pcap files |
| `-v` | Verbose mode |
| `-q` | Quiet mode |

---

## Web Interface Usage

### Accessing the Interface

```bash
# Start services
sudo systemctl start apache2 mysql xplico

# Open browser
# http://localhost/xplico
```

### Creating a Case

1. Click "New Case"
2. Enter case name and description
3. Add pcap files to the case
4. Start decoding
5. Browse decoded content

### Viewing Results

The web interface shows:
- **Web pages**: Reconstructed HTML with images
- **Emails**: Full email content with attachments
- **Files**: Downloaded files organized by type
- **Images**: Extracted images from traffic
- **Statistics**: Protocol breakdown and traffic summary

---

## Decoding Protocols

### Supported Protocols

| Protocol | Description | What's Extracted |
|----------|-------------|------------------|
| HTTP | Web traffic | Pages, images, scripts, cookies |
| SMTP | Email sending | Emails, attachments, headers |
| POP3 | Email retrieval | Emails, attachments |
| IMAP | Email access | Emails, folders, attachments |
| FTP | File transfer | Transferred files |
| DNS | Name resolution | Queries, responses |
| SSL/TLS | Encrypted traffic | Metadata (when keys available) |

### HTTP Decoding

```bash
# Xplico automatically decodes HTTP traffic
# Output includes:
# - HTML pages
# - CSS/JavaScript files
# - Images (JPEG, PNG, GIF)
# - Cookies and headers
# - POST data
```

### Email Decoding

```bash
# SMTP/POP3/IMAP emails are decoded with:
# - Sender/recipient addresses
# - Subject lines
# - Email body (text and HTML)
# - Attachments (saved as files)
# - Headers for tracing
```

---

## Analyzing Captured Data

### Web Page Reconstruction

```bash
# After decoding, web pages are in:
# /output/web/
# - index.html files for each page
# - Linked resources (images, CSS, JS)
# - Form data and cookies
```

### File Recovery

```bash
# Downloaded files are in:
# /output/files/
# - Organized by file type
# - Original filenames preserved when possible
```

### Email Analysis

```bash
# Emails are in:
# /output/email/
# - Individual email files
# - Attachments in subdirectories
# - Header information preserved
```

---

## Practical Lab Exercises

### Lab 1: Basic Pcap Decoding

```bash
# Download sample pcap
wget https://wiki.wireshark.org/uploads/__moin_import__/attachments/SampleCaptures/http.cap

# Decode with Xplico CLI
xplico -f http.cap -o /tmp/xplico_output/

# View results
ls -la /tmp/xplico_output/
```

### Lab 2: Web Traffic Reconstruction

```bash
# Capture web traffic
sudo tcpdump -i eth0 port 80 -w web_traffic.pcap

# Decode with Xplico
xplico -f web_traffic.pcap -o /tmp/web_reconstruction/

# Browse reconstructed pages
# Open HTML files in browser
```

### Lab 3: Email Recovery

```bash
# Capture email traffic
sudo tcpdump -i eth0 'port 25 or port 110 or port 143' -w email.pcap

# Decode emails
xplico -f email.pcap -o /tmp/email_recovery/

# Review extracted emails and attachments
find /tmp/email_recovery/ -name "*.eml" -o -name "*.msg"
```

---

## Integration with Other Tools

### Xplico + tcpdump

```bash
# Capture traffic with tcpdump
sudo tcpdump -i eth0 -w capture.pcap

# Decode with Xplico
xplico -f capture.pcap -o /analysis/
```

### Xplico + Wireshark

```bash
# Use Wireshark for initial analysis
# Export interesting sessions
# Then decode full content with Xplico
```

### Xplico + tcpflow

```bash
# Use tcpflow for stream reassembly
# Use Xplico for protocol decoding and content extraction
tcpflow -r capture.pcap -o /tmp/flows/
xplico -f capture.pcap -o /tmp/xplico/
```

---

## Common Forensic Scenarios

### Scenario 1: Web Attack Analysis

```bash
# Capture web traffic during attack
sudo tcpdump -i eth0 'port 80 or port 443' -w attack.pcap

# Decode with Xplico
xplico -f attack.pcap -o /evidence/web_attack/

# Review:
# - Injected content in pages
# - Stolen form data
# - Malicious scripts
```

### Scenario 2: Data Breach Investigation

```bash
# Capture all outbound traffic
sudo tcpdump -i eth0 -w breach.pcap

# Decode and analyze
xplico -f breach.pcap -o /evidence/breach/

# Look for:
# - Large file transfers
# - Encoded/encrypted data
# - Unusual protocols
```

### Scenario 3: Phishing Analysis

```bash
# Capture email traffic
sudo tcpdump -i eth0 'port 25 or port 587' -w phishing.pcap

# Decode emails
xplico -f phishing.pcap -o /evidence/phishing/

# Extract:
# - Phishing email content
# - Malicious attachments
# - Redirect URLs
```

---

## Troubleshooting

### "No pcap files found"

- Verify file exists: `ls -la capture.pcap`
- Check file format: `tcpdump -r capture.pcap -c 5`
- Ensure file permissions are correct

### "Web interface not loading"

```bash
# Check services
sudo systemctl status apache2
sudo systemctl status mysql

# Restart services
sudo systemctl restart apache2 mysql xplico
```

### "Decoding errors"

- Check pcap file integrity
- Try with different pcap files
- Check disk space in output directory
- Review Xplico logs: `/var/log/xplico/`

### "Missing content"

- Some encrypted traffic (HTTPS) may not be fully decoded
- Check if SSL keys are available for decryption
- Verify pcap contains the expected protocols

---

## Best Practices

1. **Always work with copies** — don't modify original pcap files
2. **Use proper output directories** — keep decoded content organized
3. **Check disk space** — decoded content can be large
4. **Combine with other tools** — use tcpdump for capture, Xplico for decoding
5. **Review all protocols** — don't just focus on HTTP
6. **Document your analysis** — save notes and screenshots
7. **Verify findings** — cross-reference with other tools
8. **Keep original pcaps** — they may be needed for court
9. **Use web interface** — easier for visual analysis
10. **Check for encrypted traffic** — may need SSL keys for full decoding

---

## Summary

Xplico excels at decoding network captures and reconstructing application data. Key strengths:

- Comprehensive protocol decoding
- Web page reconstruction with resources
- Email extraction with attachments
- File recovery from network traffic
- Both CLI and web interface

### Quick Reference

```bash
# Decode pcap file
xplico -f capture.pcap -o /output/

# Decode directory of pcaps
xplico -d /pcaps/ -o /output/

# Start web interface
sudo systemctl start apache2 mysql xplico
# Access: http://localhost/xplico
```

---

## References

- Xplico: http://www.xplico.org/
- Xplico GitHub: https://github.com/gallypette/xplico
- Network Forensics with Xplico
- SANS Network Forensics: https://www.sans.org/network-forensics/
