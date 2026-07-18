---
title: "chaosreader - Quick Reference Cheatsheet"
description: "Quick reference card for chaosreader session reconstruction tool"
mitre_tactic: discovery
mitre_tactic_id: TA0007
mitre_subcategory: network_sniffing
tags: [chaosreader, cheatsheet, session-reconstruction, tcpdump, html-reports]
---

# chaosreader — Cheatsheet

## Quick Start

```bash
# Install
sudo apt install chaosreader

# Analyze pcap file
chaosreader capture.pcap

# Analyze with custom output
chaosreader --output-dir /tmp/report capture.pcap

# Pipe from tcpdump
tcpdump -r capture.pcap -w - | chaosreader -
```

## Core Commands

| Command | Description |
|---------|-------------|
| `chaosreader file.pcap` | Analyze pcap file |
| `chaosreader --output-dir /path/ file.pcap` | Custom output directory |
| `chaosreader --split-sessions file.pcap` | Split sessions into files |
| `chaosreader --no-open file.pcap` | Don't open browser |
| `chaosreader --verbose file.pcap` | Verbose output |
| `tcpdump -r file.pcap -w - \| chaosreader -` | Analyze tcpdump output |

## Common Flags

| Flag | Description |
|------|-------------|
| `--help` | Show help |
| `--verbose` | Verbose output |
| `--no-open` | Don't open browser |
| `--output-dir` | Output directory |
| `--prefix` | Output file prefix |
| `--split-sessions` | Split sessions into files |
| `--no-html` | Don't generate HTML |

## Protocol Support

| Protocol | Port | Session Type |
|----------|------|--------------|
| HTTP | 80, 8080 | Request/Response pairs |
| FTP | 21 | Commands + data transfers |
| Telnet | 23 | Interactive sessions |
| SMTP | 25 | Email transactions |
| POP3 | 110 | Email retrieval |
| IMAP | 143 | Email access |
| DNS | 53 | Query/Response pairs |

## Examples

```bash
# Basic analysis
chaosreader capture.pcap

# HTTP-only analysis
tcpdump -r capture.pcap "port 80" -w /tmp/http.pcap
chaosreader /tmp/http.pcap

# Split sessions for review
chaosreader --split-sessions --output-dir /tmp/sessions capture.pcap

# Live capture analysis
sudo tcpdump -i eth0 -w - | chaosreader - --output-dir /tmp/live/

# Batch analysis
for f in *.pcap; do
    mkdir -p "output/$(basename $f .pcap)"
    chaosreader --output-dir "output/$(basename $f .pcap)" "$f"
done
```

## Chaining

```bash
# Capture + Analyze
sudo tcpdump -i eth0 -w /tmp/capture.pcap -c 10000 &
wait
chaosreader --output-dir /tmp/report /tmp/capture.pcap

# chaosreader + grep for credentials
chaosreader --output-dir /tmp/report capture.pcap
grep -rE "USER |PASS |Password:" /tmp/report/ftp_*/ /tmp/report/telnet_*/ 2>/dev/null

# chaosreader + tcpdump filter
tcpdump -r capture.pcap "port 21 or port 23" -w /tmp/sensitive.pcap
chaosreader --split-sessions /tmp/sensitive.pcap
```

## Output Structure

```
output/
├── index.html          # Main index
├── sessions.html       # All sessions
├── http_ip_port/       # HTTP sessions
│   ├── session_001.html
│   └── session_002.html
├── ftp_ip_port/        # FTP sessions
├── telnet_ip_port/     # Telnet sessions
└── smtp_ip_port/       # SMTP sessions
```

## Troubleshooting

| Problem | Solution |
|---------|----------|
| No sessions found | Verify traffic: `tcpdump -r file.pcap -c 10` |
| Perl errors | Install modules: `sudo apt install libnet-pcap-perl` |
| HTML not generated | Check permissions: `chmod 755 /tmp/output` |
| Browser won't open | Use `--no-open` and open manually |
| Large file slow | Filter first: `tcpdump -r file.pcap "port 80" -w filtered.pcap` |
| Missing protocols | Check port numbers in pcap |
