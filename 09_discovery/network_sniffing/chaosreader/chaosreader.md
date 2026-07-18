---
title: "chaosreader - TCP Session Reconstruction and Analysis"
description: "Perl-based tool that parses tcpdump output to extract and reconstruct TCP/UDP sessions into readable HTML reports"
mitre_tactic: discovery
mitre_tactic_id: TA0007
mitre_subcategory: network_sniffing
categories:
  - Session Reconstruction
  - Traffic Forensics
  - Packet Analysis
  - Report Generation
tags:
  - tcpdump
  - session-reconstruction
  - perl
  - html-reports
  - tcp-streams
  - forensics
install_command: "sudo apt install chaosreader"
version: "0.94"
kali_tools_page: "https://www.kali.org/tools/chaosreader/"
difficulty_level: beginner
requires_root: false
gui_available: false
related_tools:
  - tcpflow
  - tcpdump
  - wireshark
  - NetworkMiner
  - ngrep
---

# chaosreader — TCP Session Reconstruction and Analysis

## Chapter 1: Introduction

### What is chaosreader

**chaosreader** is a Perl script that parses tcpdump output (or tcpdump-format pcap files) to extract and reconstruct TCP and UDP sessions. It produces organized HTML reports that display each session's contents in a readable format, including HTTP transactions, FTP commands, Telnet sessions, SMTP emails, and other protocol exchanges.

chaosreader bridges the gap between raw packet capture and human-readable session analysis. Rather than manually following TCP streams in Wireshark, chaosreader automatically reassembles every session and presents them in a browsable directory structure with HTML index pages.

### Historical Context

chaosreader was written by Brendan Dolan-Gavitt (then at Victoria University of Wellington) as part of his research into network forensics. It was released in 2006 and quickly became a staple tool in the Kali Linux distribution.

The tool emerged from the need to quickly triage large pcap files without the overhead of GUI tools. In incident response scenarios, analysts need to rapidly identify interesting sessions from gigabytes of captured traffic. chaosreader's automated session extraction and HTML reporting made this possible from the command line.

The tool was later incorporated into the Bro/Zeek ecosystem and influenced the development of similar session-reconstruction tools. Despite its age, it remains relevant due to its simplicity and effectiveness.

### When to Use chaosreader

- **Incident response**: Quickly triage captured traffic to identify interesting sessions
- **Forensic analysis**: Extract and review HTTP, FTP, Telnet, SMTP sessions from pcaps
- **Malware analysis**: Examine C2 communication patterns in captured traffic
- **Penetration testing**: Review captured sessions after MITM attacks
- **Network debugging**: Analyze connection issues by reviewing session data
- **Education**: Learn protocol behavior by reading reconstructed sessions

### Core Concepts

- **Session reconstruction**: Reassembles TCP streams from individual packets
- **Protocol identification**: Automatically detects application-layer protocols
- **HTML reporting**: Generates browsable reports with session contents
- **Directory structure**: Organizes output by protocol and session
- **Non-interactive**: Runs from command line without user intervention

---

## Chapter 2: Installation

### System Requirements

| Requirement | Minimum | Recommended |
|------------|---------|-------------|
| OS | Any Linux | Kali Linux 2024+ |
| RAM | 128MB | 512MB+ |
| CPU | Single core | Single core |
| Disk | 10MB | 1GB+ for large captures |
| Perl | 5.10+ | 5.30+ |
| tcpdump | Any version | Latest |

### Installation Methods

**Method 1: Kali Package Manager (Recommended)**

```bash
sudo apt update
sudo apt install chaosreader
```

**Method 2: From Source**

```bash
wget https://www.vitonica.com/chaosreader/chaosreader0.94
chmod +x chaosreader0.94
sudo mv chaosreader0.94 /usr/local/bin/chaosreader
```

**Method 3: From SVN**

```bash
sudo apt install subversion perl
svn checkout https://github.com/brendangr/chaosreader/trunk chaosreader
cd chaosreader
chmod +x chaosreader.pl
sudo cp chaosreader.pl /usr/local/bin/chaosreader
```

### Post-Installation Verification

```bash
# Verify installation
chaosreader --version

# Check Perl availability
perl --version

# Test with sample pcap
chaosreader --help
```

### Dependency Check

```bash
# Verify required tools
which tcpdump
which tshark  # Optional but helpful
perl -v

# Install dependencies if needed
sudo apt install tcpdump perl
```

---

## Chapter 3: Core Concepts

### How chaosreader Works

chaosreader operates in a multi-stage pipeline:

```
[Pcap Input] → [tcpdump/tshark] → [Packet Parser] → [Session Reassembler] → [Protocol Identifier] → [HTML Generator]
```

1. **Input handling**: Accepts pcap files or tcpdump-format output
2. **Packet parsing**: Extracts packet headers and payloads using tcpdump or tshark
3. **Session tracking**: Groups packets by TCP/UDP flow (5-tuple)
4. **Stream reassembly**: Orders packets by sequence number within each session
5. **Protocol detection**: Identifies application-layer protocols from port numbers and payloads
6. **Report generation**: Creates HTML files with session contents

### Session Model

chaosreader reconstructs sessions based on the 5-tuple:

```
(Source IP, Source Port, Destination IP, Destination Port, Protocol)
```

Each unique combination creates a separate session entry in the output.

### Protocol Detection

chaosreader identifies protocols through:

| Method | Protocols Detected |
|--------|-------------------|
| Port number | HTTP(80), FTP(21), Telnet(23), SMTP(25), DNS(53) |
| Payload signatures | HTTP methods, FTP commands, SMTP headers |
| TCP characteristics | Handshake patterns, data flow direction |

### Architecture

```
┌─────────────────────────────────────────────┐
│              chaosreader                     │
├─────────────┬─────────────┬─────────────────┤
│  Input      │  Processing │  Output         │
├─────────────┼-------------┼─────────────────┤
│ pcap file   │ Packet      │ HTML reports    │
│ tcpdump     │ parsing     │ Session files   │
│ output      │ Session     │ Directory       │
│             │ reassembly  │ structure       │
│             │ Protocol    │ Index pages     │
│             │ detection   │                 │
└─────────────┴─────────────┴─────────────────┘
```

### Output Structure

```
output/
├── index.html              # Main index page
├── sessions.html           # All sessions listing
├── http_192.168.1.100_80/  # HTTP sessions directory
│   ├── session_001.html
│   └── session_002.html
├── ftp_192.168.1.100_21/   # FTP sessions directory
├── telnet_192.168.1.100_23/
├── smtp_192.168.1.100_25/
└── dns_192.168.1.100_53/
```

---

## Chapter 4: Command Reference

### Basic Syntax

```bash
chaosreader [OPTIONS] <pcap-file>
chaosreader [OPTIONS] <tcpdump-output-file>
```

### Core Flags

| Flag | Description | Example |
|------|-------------|---------|
| `--help` | Show help message | `chaosreader --help` |
| `--verbose` | Verbose output | `chaosreader --verbose capture.pcap` |
| `--no-open` | Don't open browser | `chaosreader --no-open capture.pcap` |
| `--output-dir` | Output directory | `chaosreader --output-dir /tmp/report capture.pcap` |
| `--prefix` | Output file prefix | `chaosreader --prefix analysis capture.pcap` |
| `--split-sessions` | Split each session into separate file | `chaosreader --split-sessions capture.pcap` |
| `--no-html` | Don't generate HTML | `chaosreader --no-html capture.pcap` |

### Input Methods

```bash
# From pcap file
chaosreader capture.pcap

# From tcpdump output
tcpdump -r capture.pcap -w - | chaosreader -

# From live tcpdump (pipe)
sudo tcpdump -i eth0 -w - | chaosreader -

# From tshark output
tshark -r capture.pcap -w - | chaosreader -
```

### Output Control

```bash
# Custom output directory
chaosreader --output-dir /var/www/html/analysis capture.pcap

# Split sessions into separate files
chaosreader --split-sessions capture.pcap

# Generate only session files (no HTML)
chaosreader --no-html capture.pcap
```

---

## Chapter 5: Examples

### Basic Examples

**Analyze a pcap file:**

```bash
chaosreader capture.pcap
```

**Analyze with custom output directory:**

```bash
chaosreader --output-dir /tmp/chaosreport capture.pcap
```

**Analyze tcpdump output:**

```bash
tcpdump -r capture.pcap -w - | chaosreader -
```

### Intermediate Examples

**Live capture and analysis:**

```bash
# Capture traffic
sudo tcpdump -i eth0 -w /tmp/live_capture.pcap -c 10000

# Analyze
chaosreader --output-dir /tmp/report /tmp/live_capture.pcap
```

**Filter before analysis:**

```bash
# Extract only HTTP traffic
tcpdump -r capture.pcap "port 80" -w /tmp/http_only.pcap
chaosreader /tmp/http_only.pcap
```

**Split sessions for individual review:**

```bash
chaosreader --split-sessions --output-dir /tmp/sessions capture.pcap
```

### Advanced Examples

**Full forensic analysis pipeline:**

```bash
#!/bin/bash
# forensic_session_analysis.sh

PCAP="$1"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
OUTPUT="/var/forensics/chaosreader_$TIMESTAMP"

# Create output directory
mkdir -p "$OUTPUT"

# Run chaosreader with full options
chaosreader \
    --output-dir "$OUTPUT" \
    --split-sessions \
    --verbose \
    "$PCAP"

# Generate summary
echo "=== ChaosReader Session Analysis ===" > "$OUTPUT/summary.txt"
echo "Pcap: $PCAP" >> "$OUTPUT/summary.txt"
echo "Date: $(date)" >> "$OUTPUT/summary.txt"
echo "" >> "$OUTPUT/summary.txt"

# Count sessions by protocol
echo "=== Session Counts ===" >> "$OUTPUT/summary.txt"
for dir in "$OUTPUT"/*/; do
    if [ -d "$dir" ]; then
        proto=$(basename "$dir")
        count=$(ls "$dir"/*.html 2>/dev/null | wc -l)
        echo "$proto: $count sessions" >> "$OUTPUT/summary.txt"
    fi
done

# Open report
echo "Report: $OUTPUT/index.html"
xdg-open "$OUTPUT/index.html" 2>/dev/null
```

**Extract specific protocol sessions:**

```bash
# Extract only FTP sessions
tcpdump -r capture.pcap "port 21" -w /tmp/ftp_only.pcap
chaosreader --output-dir /tmp/ftp_sessions /tmp/ftp_only.pcap

# Extract only SMTP sessions
tcpdump -r capture.pcap "port 25" -w /tmp/smtp_only.pcap
chaosreader --output-dir /tmp/smtp_sessions /tmp/smtp_only.pcap
```

**Batch analysis of multiple pcaps:**

```bash
#!/bin/bash
# batch_chaosreader.sh

CAPTURE_DIR="/evidence/captures"
OUTPUT_BASE="/evidence/chaosreader"
mkdir -p "$OUTPUT_BASE"

for pcap in "$CAPTURE_DIR"/*.pcap; do
    name=$(basename "$pcap" .pcap)
    echo "Processing: $name"
    chaosreader --output-dir "$OUTPUT_BASE/$name" "$pcap"
done

# Generate master index
echo "<html><body><h1>ChaosReader Batch Analysis</h1><ul>" > "$OUTPUT_BASE/index.html"
for dir in "$OUTPUT_BASE"/*/; do
    if [ -f "$dir/index.html" ]; then
        name=$(basename "$dir")
        echo "<li><a href='$name/index.html'>$name</a></li>" >> "$OUTPUT_BASE/index.html"
    fi
done
echo "</ul></body></html>" >> "$OUTPUT_BASE/index.html"
```

### Real-World Scenarios

**Incident response — triage captured traffic:**

```bash
# During IR, quickly identify interesting sessions
chaosreader --output-dir /tmp/ir_triage /evidence/capture_$(date +%Y%m%d).pcap

# Review HTTP sessions
ls -la /tmp/ir_triage/http_*/

# Check for credential transmission
grep -rl "Authorization:" /tmp/ir_triage/http_*/
grep -rl "Password:" /tmp/ir_triage/telnet_*/
```

**Malware analysis — examine C2 traffic:**

```bash
# Extract suspicious sessions from malware capture
chaosreader --split-sessions --output-dir /tmp/malware_sessions suspicious_capture.pcap

# Look for unusual protocols or ports
ls /tmp/malware_sessions/ | grep -vE "http|ftp|dns|smtp"
```

**Penetration test — review MITM capture:**

```bash
# After bettercap capture, analyze sessions
chaosreader --output-dir /tmp/pt_sessions bettercap_capture.pcap

# Find HTTP credentials
grep -r "POST" /tmp/pt_sessions/http_*/ | grep -i "pass"
```

---

## Chapter 6: Advanced Usage

### Scripting with chaosreader

**Automated analysis with post-processing:**

```bash
#!/bin/bash
# auto_analyze.sh — Automated chaosreader with keyword extraction

PCAP="$1"
OUTPUT="/tmp/chaos_analysis_$(date +%Y%m%d_%H%M%S)"

# Run chaosreader
chaosreader --output-dir "$OUTPUT" --split-sessions "$PCAP"

# Extract interesting keywords
echo "=== Potential Credentials ===" > "$OUTPUT/keywords.txt"
grep -rliE "password|passwd|secret|token|auth" "$OUTPUT"/*/ 2>/dev/null >> "$OUTPUT/keywords.txt"

echo "=== Potential Commands ===" >> "$OUTPUT/keywords.txt"
grep -rliE "exec|run|shell|cmd|powershell" "$OUTPUT"/*/ 2>/dev/null >> "$OUTPUT/keywords.txt"

echo "=== URLs ===" >> "$OUTPUT/keywords.txt"
grep -rohE "https?://[^ \"']*" "$OUTPUT"/*/ 2>/dev/null | sort -u >> "$OUTPUT/keywords.txt"
```

### Chaining with Other Tools

**chaosreader + tcpdump for targeted analysis:**

```bash
# Capture specific traffic and analyze
sudo tcpdump -i eth0 "port 80 or port 443" -w /tmp/web_traffic.pcap -c 50000
chaosreader --output-dir /tmp/web_sessions /tmp/web_traffic.pcap
```

**chaosreader + tshark for enhanced parsing:**

```bash
# Use tshark for better protocol dissection
tshark -r capture.pcap -Y "http.request" -T fields \
    -e frame.time -e ip.src -e http.host -e http.request.uri > /tmp/http_requests.txt

# Then use chaosreader for full session reconstruction
chaosreader --output-dir /tmp/sessions capture.pcap
```

**chaosreader + grep for credential hunting:**

```bash
# Run chaosreader
chaosreader --output-dir /tmp/report capture.pcap

# Hunt for credentials across all sessions
echo "=== FTP Credentials ==="
grep -rE "USER |PASS " /tmp/report/ftp_*/ 2>/dev/null

echo "=== HTTP Basic Auth ==="
grep -rE "Authorization: Basic" /tmp/report/http_*/ 2>/dev/null

echo "=== Telnet Credentials ==="
grep -rE "login:|Password:" /tmp/report/telnet_*/ 2>/dev/null
```

### Configuration Tuning

**Optimizing for large pcap files:**

```bash
# Filter to reduce processing time
tcpdump -r large.pcap "port 80" -w /tmp/filtered.pcap
chaosreader /tmp/filtered.pcap

# Split sessions for memory efficiency
chaosreader --split-sessions --output-dir /tmp/output large.pcap
```

**Custom report format:**

```bash
# Generate session files only (for custom processing)
chaosreader --no-html --split-sessions --output-dir /tmp/raw capture.pcap

# Process with custom tools
for f in /tmp/raw/*.txt; do
    # Custom processing here
    echo "Processing: $f"
done
```

---

## Chapter 7: Detection and Defense

### How chaosreader Appears in Logs

chaosreader is an analysis tool — it reads pcap files but doesn't generate network traffic. However, its use can be detected through:

**System artifacts:**

```bash
# Check for chaosreader execution
ps aux | grep chaosreader
history | grep chaosreader

# Check for output files
find / -name "index.html" -path "*/chaosreader*" 2>/dev/null
find / -name "*.html" -path "*/sessions*" 2>/dev/null

# Check for Perl execution
ps aux | grep perl | grep -i chaos
```

### Defensive Monitoring

**Monitor for session reconstruction tools:**

```bash
#!/bin/bash
# detect_session_reconstruction.sh

# Watch for chaosreader binary execution
auditctl -w /usr/bin/chaosreader -p x -k chaosreader_exec

# Watch for HTML report generation
auditctl -w /tmp/ -p wa -k session_reports

# Monitor for pcap file access
auditctl -w /var/lib/tcpdump/ -p r -k pcap_access
```

### Evasion Considerations

Session reconstruction can be evaded through:

- **Encryption**: TLS/SSL prevents plaintext session reconstruction
- **Traffic fragmentation**: Splitting sessions across multiple connections
- **Protocol obfuscation**: Using non-standard ports or protocols
- **Timing attacks**: Sending data in patterns that disrupt reassembly

**Defensive countermeasures:**

```bash
# Use encrypted protocols
# Enforce TLS 1.3
# Monitor for unusual session patterns
# Deploy network encryption (IPsec, MACsec)
```

---

## Chapter 8: Troubleshooting and FAQ

### Common Issues

**"No sessions found" error:**

```bash
# Problem: pcap contains no TCP/UDP traffic or is corrupted
# Diagnosis
tcpdump -r capture.pcap -c 10 | head

# Solution: Verify pcap contains valid traffic
tcpdump -r capture.pcap -c 100 | wc -l
```

**Perl errors:**

```bash
# Problem: Missing Perl modules
# Solution: Install required modules
sudo apt install libnet-pcap-perl

# Or install specific modules
cpan Net::Pcap
```

**HTML files not generated:**

```bash
# Problem: Output directory permissions
# Diagnosis
ls -la /tmp/

# Solution: Ensure writable directory
mkdir -p /tmp/chaosreport
chmod 755 /tmp/chaosreport
chaosreader --output-dir /tmp/chaosreport capture.pcap
```

**Browser doesn't open:**

```bash
# Problem: No X11 display or browser configured
# Solution: Use --no-open flag and open manually
chaosreader --no-open --output-dir /tmp/report capture.pcap
xdg-open /tmp/report/index.html
```

### FAQ

**Q: How does chaosreader differ from Wireshark?**
A: chaosreader automatically extracts and organizes all sessions into browsable HTML reports. Wireshark provides interactive packet-level analysis. chaosreader is better for quick triage; Wireshark is better for detailed analysis.

**Q: Can chaosreader handle encrypted traffic?**
A: No. chaosreader reconstructs plaintext sessions. Encrypted traffic (TLS/SSL) will appear as encrypted data.

**Q: What protocols does chaosreader support?**
A: HTTP, FTP, Telnet, SMTP, POP3, IMAP, DNS, and other protocols identifiable by port number or payload signatures.

**Q: Can chaosreader process live traffic?**
A: Yes, by piping tcpdump output: `sudo tcpdump -i eth0 -w - | chaosreader -`

**Q: Is chaosreader still maintained?**
A: chaosreader is mature and stable. While not actively developed, it continues to work with modern pcap files and Perl versions.

---

## Resources

- **Kali Tools Page**: https://www.kali.org/tools/chaosreader/
- **Original Documentation**: Brendan Dolan-Gavitt's original chaosreader page
- **Related Tools**: tcpflow, tcpdump, wireshark, NetworkMiner, ngrep
- **Perl Documentation**: perldoc chaosreader
- **MITRE ATT&CK**: [TA0007 - Discovery](https://attack.mitre.org/tactics/TA0007/)
- **Network Forensics**: SANS Network Forensics poster
