---
title: "bruteshark - Network Packet Analyzer and Credential Extractor"
description: "Extracts credentials, hashes, and sensitive data from captured network traffic"
mitre_tactic: discovery
mitre_tactic_id: TA0007
mitre_subcategory: network_sniffing
categories:
  - Credential Extraction
  - Packet Analysis
  - Traffic Forensics
  - Password Recovery
tags:
  - credentials
  - passwords
  - hashes
  - packet-analysis
  - forensics
  - http
  - ftp
  - smb
  - telnet
install_command: "sudo apt install bruteshark"
version: "1.6"
github_repo: "https://github.com/odedshimon/BruteShark"
kali_tools_page: "https://www.kali.org/tools/bruteshark/"
difficulty_level: beginner
requires_root: false
gui_available: true
related_tools:
  - wireshark
  - ngrep
  - NetworkMiner
  - tcpdump
  - mitmproxy
---

# bruteshark — Network Packet Analyzer and Credential Extractor

## Chapter 1: Introduction

### What is bruteshark

**BruteShark** is a network forensic analysis tool that automatically extracts credentials, passwords, hashes, and sensitive data from captured network traffic. It processes pcap files or live captures to identify authentication attempts across multiple protocols including HTTP, FTP, SMB, Telnet, and Kerberos.

Unlike general-purpose packet analyzers, BruteShark is purpose-built for credential extraction. It performs deep packet inspection on authentication-related traffic, reconstructs TCP sessions, and presents extracted credentials in an organized, searchable interface. The tool supports both GUI (Windows/.NET) and CLI modes.

### Historical Context

Credential extraction from network traffic has been a core forensic technique since the early days of network security. Tools like ngrep and NetworkMiner provided manual extraction capabilities, but required significant analyst effort. BruteShark automated this process by combining protocol-specific parsers with session reconstruction engines.

The tool was created by Oded Shimon as a response to the need for a faster, more automated approach to extracting credentials during incident response engagements. It fills the gap between raw packet analysis (Wireshark) and specialized credential-hunting tools.

### When to Use BruteShark

- **Incident response**: Extract credentials from captured traffic during an active investigation
- **Penetration testing**: Validate credential harvesting attacks against network services
- **Forensic analysis**: Recover plaintext passwords from historical pcap files
- **Compliance auditing**: Verify that protocols transmit credentials securely
- **Red team operations**: Assess exposure of credentials in network traffic
- **Network monitoring**: Continuously extract credentials from live traffic

### Core Concepts

- **Protocol dissection**: Parses authentication fields from HTTP, FTP, SMB, Telnet
- **Session reconstruction**: Reassembles TCP streams to extract complete credentials
- **Hash extraction**: Captures NTLM, Kerberos, and other authentication hashes
- **File extraction**: Recovers files transferred over FTP, SMB, HTTP
- **Live and offline modes**: Works with pcap files or real-time captures

---

## Chapter 2: Installation

### System Requirements

| Requirement | Minimum | Recommended |
|------------|---------|-------------|
| OS | Kali Linux 2020+ | Kali Linux 2024+ |
| RAM | 256MB | 1GB+ |
| CPU | Single core | Multi-core |
| Disk | 50MB | 500MB+ for captures |
| Display | CLI only | X11 for GUI |
| .NET Runtime | 6.0+ | 8.0+ |

### Installation Methods

**Method 1: Kali Package Manager (Recommended)**

```bash
sudo apt update
sudo apt install bruteshark
```

**Method 2: From GitHub Release**

```bash
# Download latest release
wget https://github.com/odedshimon/BruteShark/releases/latest/download/BruteSharkCli-linux-x64.zip
unzip BruteSharkCli-linux-x64.zip
chmod +x BruteSharkCli
sudo mv BruteSharkCli /usr/local/bin/bruteshark
```

**Method 3: Build from Source**

```bash
sudo apt install git dotnet-sdk-8.0
git clone https://github.com/odedshimon/BruteShark.git
cd BruteShark
dotnet build -c Release
sudo cp BruteSharkCli/bin/Release/net8.0/BruteSharkCli /usr/local/bin/bruteshark
```

### Post-Installation Verification

```bash
# Verify CLI installation
bruteshark --help

# Check version
bruteshark --version

# Test with sample pcap
bruteshark -p sample.pcap
```

### Permission Setup

```bash
# For live capture, ensure libpcap permissions
sudo setcap cap_net_raw,cap_net_admin=eip /usr/local/bin/bruteshark

# Or run with sudo for live captures
sudo bruteshark -i eth0
```

---

## Chapter 3: Core Concepts

### How BruteShark Works

BruteShark operates through a multi-stage pipeline:

```
[Pcap Input] → [Packet Parser] → [Protocol Dissector] → [Session Reassembler] → [Credential Extractor] → [Output]
```

1. **Packet Parser**: Reads raw packets from pcap files or live capture
2. **Protocol Dissector**: Identifies and parses application-layer protocols
3. **Session Reassembler**: Reconstructs TCP/UDP streams from fragments
4. **Credential Extractor**: Extracts authentication fields from reconstructed sessions
5. **Output Module**: Presents results in CLI or GUI format

### Protocol Support Matrix

| Protocol | Credentials | Hashes | Files | Notes |
|----------|-------------|--------|-------|-------|
| HTTP | Yes (Basic, Form) | No | Yes | GET/POST parameters |
| FTP | Yes (USER/PASS) | No | Yes | Control channel |
| Telnet | Yes (Login) | No | No | Interactive sessions |
| SMB | Yes (NTLM) | NTLMv1/v2 | Yes | Full SMB support |
| Kerberos | No | AS-REQ/TGS | No | Hash extraction |
| SMTP | Yes (AUTH) | No | No | Email credentials |
| POP3 | Yes (PASS) | No | No | Email credentials |
| IMAP | Yes (LOGIN) | No | No | Email credentials |

### Architecture

```
┌─────────────────────────────────────────────┐
│              BruteShark                      │
├─────────────┬─────────────┬─────────────────┤
│  Input      │  Analysis   │  Output         │
├─────────────┼─────────────┼─────────────────┤
│ Pcap file   │ Protocol    │ CLI table       │
│ Live capture│ dissectors  │ JSON            │
│ Directory   │ TCP recon   │ Extracted files │
│             │ Credential  │ Credentials     │
│             │ extraction  │ database        │
└─────────────┴─────────────┴─────────────────┘
```

### Data Model

BruteShark organizes extracted data into:

- **Credentials**: Username/password pairs from authentication protocols
- **Hashes**: NTLM, Kerberos, and other hash types
- **Files**: Files transferred over FTP, SMB, HTTP
- **Sessions**: Reconstructed TCP/UDP sessions with metadata
- **DNS**: DNS queries and responses
- **Network map**: Host-to-host communication patterns

---

## Chapter 4: Command Reference

### Basic Syntax

```bash
bruteshark [OPTIONS] -p <pcap-file>
bruteshark [OPTIONS] -i <interface>
```

### Core Flags

| Flag | Long | Description | Example |
|------|------|-------------|---------|
| `-p` | `--pcap` | Pcap file to analyze | `-p capture.pcap` |
| `-i` | `--interface` | Live capture interface | `-i eth0` |
| `-d` | `--directory` | Directory of pcap files | `-d /tmp/captures/` |
| `-o` | `--output` | Output directory | `-o /tmp/bruteshark_output/` |
| `-j` | `--json` | Output in JSON format | `-j` |
| `-v` | `--verbose` | Verbose output | `-v` |
| `-q` | `--quiet` | Quiet mode | `-q` |

### Analysis Flags

| Flag | Description | Example |
|------|-------------|---------|
| `--extract-credentials` | Extract passwords (default: on) | `--extract-credentials` |
| `--extract-hashes` | Extract NTLM hashes | `--extract-hashes` |
| `--extract-files` | Extract transferred files | `--extract-files` |
| `--extract-dns` | Extract DNS queries | `--extract-dns` |
| `--extract-voip` | Extract VoIP calls | `--extract-voip` |
| `--no-credentials` | Skip credential extraction | `--no-credentials` |

### Filter Flags

| Flag | Description | Example |
|------|-------------|---------|
| `--filter` | BPF filter for live capture | `--filter "port 80"` |
| `--protocols` | Limit to specific protocols | `--protocols http,ftp` |
| `--ip-filter` | Filter by IP address | `--ip-filter 192.168.1.100` |

### Output Flags

| Flag | Description | Example |
|------|-------------|---------|
| `-o` | Output directory for files | `-o /tmp/output/` |
| `--csv` | Export credentials to CSV | `--csv credentials.csv` |
| `--sqlite` | Export to SQLite database | `--sqlite results.db` |

---

## Chapter 5: Examples

### Basic Examples

**Analyze a pcap file:**

```bash
bruteshark -p /tmp/capture.pcap
```

**Live capture on interface:**

```bash
sudo bruteshark -i eth0
```

**Analyze entire directory of pcaps:**

```bash
bruteshark -d /tmp/captures/
```

### Intermediate Examples

**Extract only credentials:**

```bash
bruteshark -p capture.pcap --extract-credentials --no-credentials false
```

**Extract NTLM hashes:**

```bash
bruteshark -p capture.pcap --extract-hashes -v
```

**Export to JSON:**

```bash
bruteshark -p capture.pcap -j -o /tmp/bruteshark_output/
```

**Filter by protocol:**

```bash
bruteshark -p capture.pcap --protocols http,ftp,telnet
```

### Advanced Examples

**Full forensic analysis pipeline:**

```bash
#!/bin/bash
# forensic_analysis.sh

PCAP="$1"
OUTPUT_DIR="/tmp/bruteshark_$(date +%Y%m%d_%H%M%S)"

# Create output directory
mkdir -p "$OUTPUT_DIR"

# Run BruteShark with all extraction enabled
bruteshark -p "$PCAP" \
    --extract-credentials \
    --extract-hashes \
    --extract-files \
    --extract-dns \
    -j \
    -o "$OUTPUT_DIR" \
    -v

# Generate summary report
echo "=== BruteShark Analysis Report ===" | tee "$OUTPUT_DIR/report.txt"
echo "Pcap file: $PCAP" | tee -a "$OUTPUT_DIR/report.txt"
echo "Date: $(date)" | tee -a "$OUTPUT_DIR/report.txt"
echo "" | tee -a "$OUTPUT_DIR/report.txt"

if [ -f "$OUTPUT_DIR/credentials.json" ]; then
    echo "=== Credentials Found ===" | tee -a "$OUTPUT_DIR/report.txt"
    jq '.[] | "\(.protocol) \(.username):\(.password)"' "$OUTPUT_DIR/credentials.json" | tee -a "$OUTPUT_DIR/report.txt"
fi

if [ -f "$OUTPUT_DIR/hashes.json" ]; then
    echo "" | tee -a "$OUTPUT_DIR/report.txt"
    echo "=== Hashes Found ===" | tee -a "$OUTPUT_DIR/report.txt"
    jq '.[] | "\(.type) \(.hash)"' "$OUTPUT_DIR/hashes.json" | tee -a "$OUTPUT_DIR/report.txt"
fi

echo "Report saved to: $OUTPUT_DIR/report.txt"
```

**Live capture with filtering:**

```bash
# Capture only HTTP traffic and extract credentials
sudo bruteshark -i eth0 \
    --filter "port 80 or port 8080" \
    --protocols http \
    --extract-credentials \
    -j \
    -o /tmp/live_credentials/
```

**Batch analysis of multiple pcaps:**

```bash
#!/bin/bash
# batch_analysis.sh

CAPTURE_DIR="/evidence/captures"
OUTPUT_DIR="/evidence/analysis"
mkdir -p "$OUTPUT_DIR"

for pcap in "$CAPTURE_DIR"/*.pcap; do
    echo "Analyzing: $pcap"
    bruteshark -p "$pcap" \
        --extract-credentials \
        --extract-hashes \
        --extract-files \
        -j \
        -o "$OUTPUT_DIR/$(basename "$pcap" .pcap)/"
done

# Aggregate results
echo "=== Aggregate Credentials ===" > "$OUTPUT_DIR/aggregate.txt"
for json in "$OUTPUT_DIR"/*/credentials.json; do
    [ -f "$json" ] && jq -r '.[] | "\(.protocol)\t\(.username)\t\(.password)"' "$json"
done >> "$OUTPUT_DIR/aggregate.txt"
```

### Real-World Scenarios

**Incident response — extract credentials from captured traffic:**

```bash
# During IR engagement, analyze captured pcap
bruteshark -p /evidence/$(date +%Y%m%d)_capture.pcap \
    --extract-credentials \
    --extract-hashes \
    --extract-files \
    -j \
    -o /evidence/bruteshark_results/ \
    -v

# Review results
cat /evidence/bruteshark_results/credentials.json | jq '.'
```

**Penetration test — validate credential exposure:**

```bash
# After running bettercap, extract captured credentials
bruteshark -p /tmp/bettercap_capture.pcap \
    --extract-credentials \
    --extract-hashes \
    --csv /tmp/pt_credentials.csv

# Check for cleartext passwords
grep -i "password" /tmp/pt_credentials.csv
```

**Compliance audit — verify protocol security:**

```bash
# Analyze traffic for cleartext credential transmission
bruteshark -p /tmp/network_traffic.pcap \
    --extract-credentials \
    --extract-hashes \
    -j \
    -o /tmp/compliance_results/

# Flag any cleartext protocols found
jq -r '.[] | select(.protocol == "ftp" or .protocol == "telnet" or .protocol == "http") |
    "WARNING: Cleartext \(.protocol) credentials found: \(.username):\(.password)"' \
    /tmp/compliance_results/credentials.json
```

---

## Chapter 6: Advanced Usage

### Scripting with BruteShark

**Automated analysis pipeline:**

```bash
#!/bin/bash
# auto_analyze.sh — Automated credential extraction and reporting

INPUT="$1"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
OUTPUT="/var/lib/bruteshark/$TIMESTAMP"

# Create output directory
mkdir -p "$OUTPUT"

# Run BruteShark
bruteshark -p "$INPUT" \
    --extract-credentials \
    --extract-hashes \
    --extract-files \
    --extract-dns \
    -j \
    -o "$OUTPUT"

# Process results
CRED_COUNT=$(jq '. | length' "$OUTPUT/credentials.json" 2>/dev/null || echo 0)
HASH_COUNT=$(jq '. | length' "$OUTPUT/hashes.json" 2>/dev/null || echo 0)

# Generate alert if credentials found
if [ "$CRED_COUNT" -gt 0 ]; then
    logger -t bruteshark "ALERT: $CRED_COUNT credentials found in $INPUT"
    jq -r '.[] | "\(.protocol) - \(.username):\(.password)"' "$OUTPUT/credentials.json" >> \
        /var/log/bruteshark/credentials.log
fi

if [ "$HASH_COUNT" -gt 0 ]; then
    logger -t bruteshark "ALERT: $HASH_COUNT hashes found in $INPUT"
fi
```

### Chaining with Other Tools

**BruteShark + tcpdump for capture and analysis:**

```bash
# Capture traffic
sudo tcpdump -i eth0 -w /tmp/capture.pcap -s 0 &
TCPDUMP_PID=$!

# Wait for capture
sleep 300
kill $TCPDUMP_PID

# Analyze with BruteShark
bruteshark -p /tmp/capture.pcap \
    --extract-credentials \
    --extract-hashes \
    -j -o /tmp/analysis/
```

**BruteShark + hashcat for password cracking:**

```bash
# Extract NTLM hashes
bruteshark -p capture.pcap --extract-hashes -j -o /tmp/hashes/

# Convert to hashcat format
jq -r '.[] | select(.type == "NTLM") | .hash' /tmp/hashes/hashes.json > /tmp/ntlm_hashes.txt

# Crack with hashcat
hashcat -m 5600 /tmp/ntlm_hashes.txt wordlist.txt
```

**BruteShark + Python for custom processing:**

```python
#!/usr/bin/env python3
"""process_bruteshark.py - Process BruteShark JSON output"""

import json
import sys

def process_credentials(json_file):
    with open(json_file) as f:
        credentials = json.load(f)

    # Group by protocol
    by_protocol = {}
    for cred in credentials:
        proto = cred.get('protocol', 'unknown')
        if proto not in by_protocol:
            by_protocol[proto] = []
        by_protocol[proto].append(cred)

    # Report
    for proto, creds in by_protocol.items():
        print(f"\n=== {proto.upper()} Credentials ({len(creds)}) ===")
        for c in creds:
            print(f"  {c.get('username', 'N/A')}:{c.get('password', 'N/A')}")

if __name__ == '__main__':
    process_credentials(sys.argv[1])
```

### Configuration Tuning

**Optimizing for large pcap files:**

```bash
# Process large pcaps with memory limits
bruteshark -p /tmp/large_capture.pcap \
    --extract-credentials \
    -j \
    -o /tmp/output/ \
    --quiet  # Reduce output overhead
```

**Tuning extraction depth:**

```bash
# Extract everything
bruteshark -p capture.pcap \
    --extract-credentials \
    --extract-hashes \
    --extract-files \
    --extract-dns \
    --extract-voip

# Minimal extraction (faster)
bruteshark -p capture.pcap \
    --extract-credentials \
    --no-credentials false
```

---

## Chapter 7: Detection and Defense

### How BruteShark Appears in Logs

BruteShark is an analysis tool — it doesn't generate network traffic. However, its use indicates:

**System artifacts:**

```bash
# Check for BruteShark execution
ps aux | grep bruteshark
history | grep bruteshark

# Check for output files
find /tmp -name "*.json" -newer /tmp/timestamp 2>/dev/null
ls -la /tmp/bruteshark*/
```

### Defensive Monitoring

**Detect credential exposure in your network:**

```bash
#!/bin/bash
# monitor_credentials.sh — Detect cleartext credential transmission

INTERFACE="eth0"
ALERT_LOG="/var/log/credential_monitor.log"

# Monitor for cleartext authentication protocols
sudo tcpdump -i "$INTERFACE" -l -A | \
while IFS= read -r line; do
    # Detect FTP credentials
    if echo "$line" | grep -qEi "USER |PASS "; then
        echo "$(date) [ALERT] FTP credentials detected" >> "$ALERT_LOG"
    fi

    # Detect HTTP Basic Auth
    if echo "$line" | grep -qEi "Authorization: Basic"; then
        echo "$(date) [ALERT] HTTP Basic Auth detected" >> "$ALERT_LOG"
    fi

    # Detect Telnet login
    if echo "$line" | grep -qEi "login:|Password:"; then
        echo "$(date) [ALERT] Telnet credentials detected" >> "$ALERT_LOG"
    fi
done
```

### Evasion Considerations

Credentials are exposed through:

- **Cleartext protocols**: FTP, Telnet, HTTP, SMTP transmit passwords in plaintext
- **Weak hashing**: NTLMv1 hashes can be cracked quickly
- **Metadata leakage**: DNS queries, HTTP headers may contain credentials
- **Session tokens**: Cookies and tokens may be captured

**Defensive countermeasures:**

```bash
# Enforce encrypted protocols
# Disable FTP, Telnet on network devices
# Use SSH instead of Telnet
# Use SFTP instead of FTP
# Enforce HTTPS with HSTS
# Use NTLMv2 or Kerberos instead of NTLMv1
```

---

## Chapter 8: Troubleshooting and FAQ

### Common Issues

**"No credentials found" despite cleartext traffic:**

```bash
# Problem: Protocol not recognized or traffic encrypted
# Diagnosis
bruteshark -p capture.pcap -v  # Check verbose output

# Solution: Verify traffic contains cleartext credentials
tcpdump -A -r capture.pcap | grep -iE "USER|PASS|Authorization"
```

**BruteShark crashes on large pcap:**

```bash
# Problem: Memory exhaustion
# Solution: Split large pcap into chunks
editcap -c 100000 large.pcap chunk_%d.pcap
for f in chunk_*.pcap; do
    bruteshark -p "$f" -j -o /tmp/output/
done
```

**GUI not displaying:**

```bash
# Problem: X11 forwarding or display issues
# Solution: Use CLI mode or ensure X11 is configured
export DISPLAY=:0
bruteshark-gui
```

**Missing .NET runtime:**

```bash
# Problem: .NET not installed
# Solution: Install .NET runtime
sudo apt install dotnet-runtime-8.0
```

### FAQ

**Q: Does BruteShark work with encrypted traffic?**
A: No. BruteShark extracts cleartext credentials from unencrypted protocols (HTTP, FTP, Telnet, SMB). For encrypted traffic, you need to decrypt it first using session keys or other methods.

**Q: Can BruteShark crack passwords?**
A: No. BruteShark extracts credentials from network traffic. It does not perform password cracking. Extracted hashes can be processed with hashcat or John the Ripper.

**Q: Does BruteShark support wireless captures?**
A: Yes, as long as the capture contains TCP/UDP traffic. BruteShark works with any standard pcap file regardless of the capture medium.

**Q: What protocols does BruteShark support?**
A: HTTP (Basic Auth, Form-based), FTP, Telnet, SMB (NTLM), Kerberos, SMTP, POP3, IMAP, and DNS.

**Q: Is BruteShark passive or active?**
A: BruteShark is a passive analysis tool. It reads existing pcap files or captures traffic but does not generate or inject packets.

---

## Resources

- **GitHub Repository**: https://github.com/odedshimon/BruteShark
- **Kali Tools Page**: https://www.kali.org/tools/bruteshark/
- **Related Tools**: wireshark, ngrep, NetworkMiner, tcpdump, mitmproxy
- **Password Cracking**: hashcat, John the Ripper
- **MITRE ATT&CK**: [TA0007 - Discovery](https://attack.mitre.org/tactics/TA0007/)
- **Protocol Security**: RFC 2616 (HTTP), RFC 959 (FTP), RFC 854 (Telnet)
