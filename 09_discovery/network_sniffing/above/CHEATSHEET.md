---
title: "above - Quick Reference Cheatsheet"
description: "Quick reference card for above network security monitoring tool"
mitre_tactic: discovery
mitre_tactic_id: TA0007
mitre_subcategory: network_sniffing
tags: [above, network-monitoring, cheatsheet, nsm]
---

# above — Cheatsheet

## Quick Start

```bash
# Install
sudo apt install above

# Basic monitoring
sudo above -i eth0

# Verbose JSON output
sudo above -i eth0 -v -j

# Read from pcap file
sudo above -r capture.pcap -v
```

## Core Commands

| Command | Description |
|---------|-------------|
| `sudo above -i eth0` | Monitor interface eth0 |
| `sudo above -i eth0 -v` | Verbose monitoring |
| `sudo above -r file.pcap` | Analyze existing pcap |
| `sudo above -i eth0 -j -l alerts.json` | JSON alerts to file |
| `sudo above -i eth0 -f "port 80"` | Filter HTTP traffic only |
| `sudo above --list-interfaces` | List available interfaces |

## Capture Filters (BPF)

```bash
# By host
sudo above -i eth0 -f "host 192.168.1.100"

# By subnet
sudo above -i eth0 -f "net 192.168.1.0/24"

# By port
sudo above -i eth0 -f "port 53"

# Exclude traffic
sudo above -i eth0 -f "not port 22"

# Combined
sudo above -i eth0 -f "net 192.168.1.0/24 and not port 22"
```

## Analysis Modes

```bash
# ARP analysis
sudo above -i eth0 --analyze-arp

# DNS analysis
sudo above -i eth0 --analyze-dns

# HTTP analysis
sudo above -i eth0 --analyze-http

# Full analysis
sudo above -i eth0 --analyze-arp --analyze-dns --analyze-http

# Baseline learning (5 minutes)
sudo above -i eth0 --baseline-time 300

# Adjusted anomaly threshold
sudo above -i eth0 --anomaly-threshold 0.8
```

## Common Flags

| Flag | Description |
|------|-------------|
| `-i` | Interface to monitor |
| `-r` | Read from pcap file |
| `-c` | Write captures to pcap |
| `-l` | Log file path |
| `-v` | Verbose output |
| `-q` | Quiet mode |
| `-j` | JSON output |
| `-f` | BPF filter expression |
| `--analyze-arp` | Enable ARP analysis |
| `--analyze-dns` | Enable DNS analysis |
| `--analyze-http` | Enable HTTP analysis |
| `--baseline-time` | Baseline learning duration (seconds) |
| `--anomaly-threshold` | Sensitivity 0.0-1.0 |
| `--pcap-ring` | Ring buffer size (MB) |
| `--stats-interval` | Stats output interval (seconds) |
| `--alerts-file` | Alerts output file |

## Output Formats

```bash
# JSON (for SIEM)
sudo above -i eth0 -j --alerts-file alerts.json

# Console (human-readable)
sudo above -i eth0 -v

# CSV
sudo above -i eth0 --csv --output results.csv
```

## Examples

```bash
# Monitor only internal traffic
sudo above -i eth0 -f "net 192.168.1.0/24"

# Capture with ring buffer
sudo above -i eth0 -c /tmp/capture.pcap --pcap-ring 500

# Full monitoring pipeline
sudo above -i eth0 -v -j -c /tmp/capture.pcap \
    --alerts-file /var/log/above/alerts.json

# Multi-interface monitoring
sudo above -i eth0 --alerts-file internal.json &
sudo above -i eth1 --alerts-file dmz.json &
wait
```

## Chaining

```bash
# above + jq for alert filtering
sudo above -i eth0 -j --alerts-file /dev/stdout | \
    jq 'select(.severity == "critical")'

# above + tcpdump for parallel capture
sudo above -i eth0 -j --alerts-file alerts.json &
sudo tcpdump -i eth0 -w full.pcap &
wait

# above + fail2ban for auto-blocking
sudo above -i eth0 -j --alerts-file /dev/stdout | \
    jq -r 'select(.severity == "critical") | .src_ip' | \
    while read ip; do fail2ban-client set above-ban banip "$ip"; done
```

## Troubleshooting

| Problem | Solution |
|---------|----------|
| Permission denied | `sudo above -i eth0` or `setcap cap_net_raw+eip /usr/bin/above` |
| No packets captured | Check interface: `ip link show eth0; tcpdump -i eth0 -c 5` |
| High CPU usage | Add BPF filter: `-f "net 192.168.1.0/24"` |
| Disk full | Monitor `/var/lib/above/` space; use `--pcap-ring` for rotation |
| No alerts | Lower threshold: `--anomaly-threshold 0.5` |
| Interface not found | List interfaces: `above --list-interfaces` |
