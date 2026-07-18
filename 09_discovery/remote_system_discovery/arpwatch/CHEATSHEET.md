---
title: "arpwatch Cheatsheet"
description: "Quick reference for arpwatch ARP monitoring and alerting commands"
mitre_tactic: discovery
mitre_tactic_id: TA0007
mitre_subcategory: remote_system_discovery
tags:
  - arp
  - monitoring
  - cheatsheet
  - alerts
---

# arpwatch Cheatsheet

## Quick Start

```bash
# Monitor on interface
sudo arpwatch -i eth0

# Debug mode (foreground)
sudo arpwatch -d -i eth0

# With email alerts
sudo arpwatch -i eth0 -e admin@company.com
```

## Common Flags

| Flag | Description | Example |
|------|-------------|---------|
| `-i IFACE` | Interface to monitor | `arpwatch -i eth0` |
| `-f FILE` | Database file | `arpwatch -f /var/lib/arpwatch/arp.dat` |
| `-d` | Debug (foreground) | `arpwatch -d -i eth0` |
| `-e EMAIL` | Alert email | `arpwatch -e admin@co.com` |
| `-s EMAIL` | Sender email | `arpwatch -s arpwatch@co.com` |
| `-P` | Promiscuous mode | `arpwatch -P -i eth0` |
| `-r FILE` | Read pcap file | `arpwatch -r capture.pcap` |
| `-N` | No hostname resolution | `arpwatch -N -i eth0` |
| `-p` | No promiscuous mode | `arpwatch -p -i eth0` |

## Alert Types

| Alert | Meaning | Action |
|-------|---------|--------|
| `new station` | New IP/MAC pair | Verify device is authorized |
| `flip flop` | Host changed MAC | Check for spoofing or VM migration |
| `changed` | IP reassigned | Investigate DHCP or attack |
| `unchanged` | Known host | Normal activity |

## Host Discovery

```bash
# View database
sudo cat /var/lib/arpwatch/arp.dat | strings

# Monitor for new hosts
sudo arpwatch -d -i eth0 2>&1 | grep "new station"

# Count events by type
grep "arpwatch:" /var/log/syslog | awk '{print $3}' | sort | uniq -c
```

## Examples

**Basic monitoring:**
```bash
sudo arpwatch -i eth0
```

**Debug with email:**
```bash
sudo arpwatch -d -i eth0 -e admin@company.com
```

**Monitor from pcap:**
```bash
sudo arpwatch -r captured.pcap -d
```

**Multi-interface:**
```bash
sudo arpwatch -i eth0 -f /var/lib/arpwatch/arp0.dat &
sudo arpwatch -i eth1 -f /var/lib/arpwatch/arp1.dat &
```

## Chaining

```bash
# arpwatch + tcpdump for analysis
sudo arpwatch -d -i eth0 &
sudo tcpdump -i eth0 arp -w arp_debug.pcap

# Parse logs for alerts
grep -E "flip flop|new station|changed" /var/log/syslog

# Alert on flip-flop events
sudo arpwatch -d -i eth0 2>&1 | while read line; do
    echo "$line" | grep -q "flip flop" && echo "$line" | mail -s "ARP Alert" admin@co.com
done
```

## Troubleshooting

| Problem | Fix |
|---------|-----|
| "No such device" | Check interface: `ip -br link show` |
| Not starting | Check service: `sudo systemctl status arpwatch` |
| No alerts | Check ARP traffic: `sudo tcpdump -i eth0 arp` |
| High CPU | Disable promiscuous: `arpwatch -p -i eth0` |
| Database corruption | Remove and restart: `rm /var/lib/arpwatch/arp.dat` |
