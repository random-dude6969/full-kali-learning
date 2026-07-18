---
title: "darkstat - Quick Reference Cheatsheet"
description: "Quick reference card for darkstat network traffic analyzer"
mitre_tactic: discovery
mitre_tactic_id: TA0007
mitre_subcategory: network_sniffing
tags: [darkstat, traffic-analysis, cheatsheet, bandwidth-monitoring, web-interface]
---

# darkstat — Cheatsheet

## Quick Start

```bash
# Install
sudo apt install darkstat

# Start monitoring
sudo darkstat -i eth0 -p 6666

# Access web interface
xdg-open http://localhost:6666
```

## Core Commands

| Command | Description |
|---------|-------------|
| `sudo darkstat -i eth0` | Monitor interface eth0 |
| `sudo darkstat -i eth0 -p 6666` | Custom HTTP port |
| `sudo darkstat -i eth0 -b 192.168.1.1` | Bind to specific IP |
| `sudo darkstat -i eth0 --no-dns` | Disable reverse DNS |
| `sudo darkstat -i eth0 -n` | No promiscuous mode |

## Common Flags

| Flag | Description |
|------|-------------|
| `-i` | Network interface |
| `-p` | HTTP server port |
| `-b` | Bind address |
| `-f` | BPF capture filter |
| `-d` | Data directory |
| `-l` | Log file path |
| `-n` | No promiscuous mode |
| `-v` | Verbose logging |
| `--no-dns` | Disable reverse DNS |
| `--no-mac` | Don't show MACs |
| `--day-log` | Daily log file |
| `--month-log` | Monthly log file |

## Capture Filters (BPF)

```bash
# By host
sudo darkstat -i eth0 -f "host 192.168.1.100"

# By subnet
sudo darkstat -i eth0 -f "net 192.168.1.0/24"

# By port
sudo darkstat -i eth0 -f "port 80"

# Exclude traffic
sudo darkstat -i eth0 -f "not port 22"

# Outbound only
sudo darkstat -i eth0 -f "src net 192.168.1.0/24 and dst not net 192.168.1.0/24"
```

## Web Interface Pages

| Page | Description |
|------|-------------|
| `http://host:port/` | Dashboard |
| `http://host:port/host/IP` | Per-host details |
| `http://host:port/port/PORT` | Per-port details |
| `http://host:port/graph` | Traffic graphs |
| `http://host:port/export` | Export data |

## Examples

```bash
# Basic monitoring
sudo darkstat -i eth0 -p 6666

# Monitor only HTTP
sudo darkstat -i eth0 -p 6666 -f "port 80 or port 443"

# Headless server (accessible remotely)
sudo darkstat -i eth0 -p 6666 -b 0.0.0.0

# Localhost only (secure)
sudo darkstat -i eth0 -p 6666 -b 127.0.0.1

# With logging
sudo darkstat -i eth0 -p 6666 -l /var/log/darkstat.log -d /var/lib/darkstat

# Multi-interface
sudo darkstat -i eth0 -p 6666 -b 192.168.1.1 &
sudo darkstat -i eth1 -p 6667 -b 10.0.0.1 &
wait

# Performance optimized
sudo darkstat -i eth0 -p 6666 --no-dns -n
```

## Chaining

```bash
# darkstat + tcpdump for detailed capture
sudo darkstat -i eth0 -p 6666 &
sudo tcpdump -i eth0 -w full_capture.pcap &
wait

# darkstat + curl for data export
curl -s http://localhost:6666/export > /tmp/darkstat_data.json

# darkstat + cron for periodic reports
echo "0 */6 * * * curl -s http://localhost:6666/export > /tmp/darkstat_\$(date +\%Y\%m\%d_\%H).json" | crontab -
```

## Configuration

```bash
# Create systemd service
cat > /etc/systemd/system/darkstat.service << 'EOF'
[Unit]
Description=darkstat Network Traffic Analyzer
After=network.target

[Service]
Type=simple
ExecStart=/usr/bin/darkstat -i eth0 -p 6666 -b 0.0.0.0 -d /var/lib/darkstat
Restart=always

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl enable darkstat
sudo systemctl start darkstat
```

## Troubleshooting

| Problem | Solution |
|---------|----------|
| No such device | Check interfaces: `ip link show` |
| Port in use | Use different port: `-p 6667` |
| Web UI not accessible | Bind to all: `-b 0.0.0.0` |
| High CPU | Add filter: `-f "net 192.168.1.0/24"` |
| No DNS names | Disable DNS: `--no-dns` |
| Permission denied | Run with sudo |
| No data showing | Check interface has traffic: `tcpdump -i eth0 -c 5` |
