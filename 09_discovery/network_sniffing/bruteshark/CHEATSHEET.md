---
title: "bruteshark - Quick Reference Cheatsheet"
description: "Quick reference card for bruteshark credential extractor"
mitre_tactic: discovery
mitre_tactic_id: TA0007
mitre_subcategory: network_sniffing
tags: [bruteshark, credentials, cheatsheet, packet-analysis]
---

# bruteshark — Cheatsheet

## Quick Start

```bash
# Install
sudo apt install bruteshark

# Analyze pcap file
bruteshark -p capture.pcap

# Live capture
sudo bruteshark -i eth0

# Analyze directory of pcaps
bruteshark -d /tmp/captures/
```

## Core Commands

| Command | Description |
|---------|-------------|
| `bruteshark -p file.pcap` | Analyze pcap file |
| `bruteshark -i eth0` | Live capture |
| `bruteshark -d /path/` | Analyze pcap directory |
| `bruteshark -p file.pcap -j -o /out/` | JSON output to directory |
| `bruteshark --help` | Show help |

## Common Flags

| Flag | Description |
|------|-------------|
| `-p` | Pcap file to analyze |
| `-i` | Live capture interface |
| `-d` | Directory of pcap files |
| `-o` | Output directory |
| `-j` | JSON output format |
| `-v` | Verbose output |
| `-q` | Quiet mode |
| `--extract-credentials` | Extract passwords |
| `--extract-hashes` | Extract NTLM hashes |
| `--extract-files` | Extract transferred files |
| `--extract-dns` | Extract DNS queries |
| `--protocols` | Limit protocol analysis |
| `--filter` | BPF filter (live capture) |

## Protocol Support

| Protocol | Credentials | Files | Hashes |
|----------|-------------|-------|--------|
| HTTP | Yes (Basic/Form) | Yes | No |
| FTP | Yes (USER/PASS) | Yes | No |
| Telnet | Yes (Login) | No | No |
| SMB | Yes (NTLM) | Yes | NTLMv1/v2 |
| Kerberos | No | No | AS-REQ/TGS |
| SMTP | Yes (AUTH) | No | No |
| POP3 | Yes (PASS) | No | No |
| IMAP | Yes (LOGIN) | No | No |

## Examples

```bash
# Basic analysis
bruteshark -p capture.pcap

# Extract only credentials
bruteshark -p capture.pcap --extract-credentials

# Extract NTLM hashes
bruteshark -p capture.pcap --extract-hashes

# Full extraction with JSON output
bruteshark -p capture.pcap --extract-credentials --extract-hashes --extract-files -j -o /tmp/output/

# Limit to HTTP/FTP
bruteshark -p capture.pcap --protocols http,ftp

# Live capture with filter
sudo bruteshark -i eth0 --filter "port 80" --protocols http

# Batch process
bruteshark -d /tmp/captures/ -j -o /tmp/results/
```

## Chaining

```bash
# Capture + Extract
sudo tcpdump -i eth0 -w /tmp/capture.pcap -c 10000 &
wait
bruteshark -p /tmp/capture.pcap --extract-credentials -j -o /tmp/output/

# Extract hashes + Crack
bruteshark -p capture.pcap --extract-hashes -j -o /tmp/hashes/
jq -r '.[] | select(.type == "NTLM") | .hash' /tmp/hashes/hashes.txt > ntlm.txt
hashcat -m 5600 ntlm.txt wordlist.txt

# Bettercap + BruteShark
sudo bettercap -iface eth0 -eval "arp.spoof on; net.sniff on; sleep 60" -pcap /tmp/attack.pcap
bruteshark -p /tmp/attack.pcap --extract-credentials -j -o /tmp/results/
```

## Output Format

```json
{
  "protocol": "http",
  "src_ip": "192.168.1.100",
  "dst_ip": "192.168.1.1",
  "username": "admin",
  "password": "password123",
  "timestamp": "2024-01-15T10:30:00Z"
}
```

## Troubleshooting

| Problem | Solution |
|---------|----------|
| No credentials found | Verify cleartext: `tcpdump -A -r file.pcap \| grep -iE "USER\|PASS"` |
| Crashes on large pcap | Split file: `editcap -c 100000 large.pcap chunk_%d.pcap` |
| GUI not working | Use CLI mode: `bruteshark -p file.pcap` |
| .NET errors | Install runtime: `sudo apt install dotnet-runtime-8.0` |
| Permission denied | Run with sudo for live capture |
| No output files | Check output directory exists: `-o /tmp/output/` |
