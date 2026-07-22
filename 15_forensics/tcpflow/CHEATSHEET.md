# tcpflow Cheatsheet — TCP Flow Recorder

## Quick Command

```bash
tcpflow [options] [bpf-filter]
```

## Installation

```bash
sudo apt install tcpflow
```

## Most Common Commands

```bash
# Live capture on interface
sudo tcpflow -i eth0 -o /evidence/

# Capture HTTP traffic
sudo tcpflow -i eth0 port 80 -o /evidence/web/

# Analyze pcap file
tcpflow -a -r capture.pcap -o /analysis/

# Capture with pcap output
sudo tcpflow -i eth0 -P capture.pcap -o /evidence/

# Generate DXML report
tcpflow -r capture.pcap -X report.xml
```

## Options Reference

| Option | Description |
|--------|-------------|
| `-i interface` | Capture on this interface |
| `-r pcap_file` | Read from pcap file |
| `-o dir` | Output directory |
| `-P pcap_file` | Write captured packets to pcap |
| `-a` | Auto-interpret HTTP responses |
| `-b` | Breadth-first (memory efficient) |
| `-c` | Console mode |
| `-e` | Extra packet info |
| `-v` | Verbose mode |
| `-T template` | Filename template |
| `-X file` | DXML output |
| `-V` | Print version |

## BPF Filter Examples

```bash
# HTTP only
'port 80'

# Specific host
'host 192.168.1.100'

# SMTP
'port 25 or port 587'

# All except SSH
'not port 22'

# Traffic between two hosts
'host 192.168.1.100 and host 10.0.0.1'

# Port range
'portrange 1000-2000'
```

## Output File Format

```
[source_ip].[source_port]-[dest_ip].[dest_port]
```

With HTTP post-processing (`-a`):
```
flow_file              # Raw stream
flow_file-HTTP         # HTTP headers + body
flow_file-HTTPBODY     # Body only
flow_file-HTTPBODY-GZIP # Decompressed body
```

## Quick Workflow

```bash
# 1. Capture or read pcap
sudo tcpflow -a -i eth0 port 80 -o /analysis/

# 2. List flows
ls -la /analysis/

# 3. View HTTP content
cat /analysis/*-HTTP

# 4. Search for content
grep -r "search_term" /analysis/

# 5. Extract files
cat /analysis/*-HTTPBODY > extracted.bin
```

## Integration

```bash
# tcpflow + Wireshark (capture + packets)
sudo tcpflow -i eth0 -P capture.pcap -o /flows/

# tcpflow + foremost (file carving)
tcpflow -r capture.pcap -o /tmp/flows/
for f in /tmp/flows/*; do foremost -i "$f" -o "/tmp/carved/$(basename $f)/"; done

# tcpflow + bulk_extractor
tcpflow -r capture.pcap -o /tmp/flows/
for f in /tmp/flows/*; do bulk_extractor "$f" -o "/tmp/be/$(basename $f)/"; done
```

## Troubleshooting

| Problem | Fix |
|---------|-----|
| No interfaces | Use `ip link show` to find interface |
| Permission denied | Use `sudo` for live capture |
| No output | Check filter matches traffic |
| Slow processing | Use `-b` for large files |
| Can't read HTTPS | Use Wireshark with SSL keys |
