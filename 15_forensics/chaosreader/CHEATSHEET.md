# chaosreader Cheatsheet — TCP/UDP Session Analyzer

## Quick Command

```bash
chaosreader [options] <tcpdump_output_file>
```

## Installation

```bash
sudo apt install chaosreader
```

## Most Common Commands

```bash
# Process tcpdump output
chaosreader capture.out

# With packet limit
chaosreader -n 1000 capture.out

# Sort by time
chaosreader -s capture.out

# Table view only
chaosreader -t capture.out
```

## Creating Input Files

```bash
# Capture and convert
sudo tcpdump -i eth0 -w capture.pcap
tcpdump -r capture.pcap > capture.out

# Or capture directly to text
sudo tcpdump -i eth0 > capture.out
```

## Options Reference

| Option | Description |
|--------|-------------|
| `-n <num>` | Limit packets to process |
| `-s` | Sort sessions by start time |
| `-t` | Print session table only |
| `-v` | Verbose output |
| `-h` | Help |

## Output Structure

```
chaosreader_output/
├── index.html              # Main session listing
├── session_0001.html       # Session report
├── session_0001.dat        # Raw session data
├── tcp_session_0001.html   # TCP details
├── http_session_0001.html  # HTTP details
├── dns_session_0001.html   # DNS details
└── ...
```

## Quick Workflow

```bash
# 1. Capture traffic
sudo tcpdump -i eth0 -w capture.pcap

# 2. Convert to text
tcpdump -r capture.pcap > capture.out

# 3. Process
chaosreader capture.out

# 4. View results
xdg-open chaosreader_output/index.html
```

## Session Analysis

```bash
# Find HTTP sessions
ls chaosreader_output/http_session_*.dat

# Find DNS sessions
ls chaosreader_output/dns_session_*.dat

# Search session data
cat chaosreader_output/session_*.dat | strings

# Find large sessions
ls -la chaosreader_output/session_*.dat | sort -k5 -rn
```

## Integration

```bash
# chaosreader + tcpflow
chaosreader capture.out
tcpflow -r capture.pcap -o tcpflow_output/

# chaosreader + Wireshark
chaosreader capture.out  # overview
wireshark capture.pcap   # detail

# chaosreader + NetworkMiner
chaosreader capture.out  # sessions
networkminer -r capture.pcap -o nm/  # files/hosts
```

## Troubleshooting

| Problem | Fix |
|---------|-----|
| Command not found | `sudo apt install chaosreader` |
| Cannot read file | Convert with `tcpdump -r file.pcap > file.out` |
| No sessions found | Check capture has content |
| HTML not opening | Use `firefox chaosreader_output/index.html` |
