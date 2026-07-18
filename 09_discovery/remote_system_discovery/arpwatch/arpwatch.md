---
title: "arpwatch — ARP Network Monitoring and Alerting"
description: "Monitor ARP traffic, detect IP/MAC changes, and alert on network anomalies for intrusion detection."
mitre_tactic: discovery
mitre_tactic_id: TA0007
mitre_subcategory: remote_system_discovery
categories:
  - network-monitoring
  - arp
  - intrusion-detection
tags:
  - arp
  - monitoring
  - alerts
  - detection
  - network
install_command: "sudo apt install arpwatch"
version: "3.1-1"
github_repo: ""
kali_tools_page: "https://www.kali.org/tools/arpwatch/"
difficulty_level: intermediate
requires_root: true
gui_available: false
related_tools:
  - arping
  - arp-scan
  - p0f
---

# arpwatch — ARP Network Monitoring and Alerting

## 1. Introduction

**arpwatch** is a daemon that monitors ARP traffic on a network interface and maintains a database of IP-to-MAC address mappings. It generates alerts when it detects suspicious changes such as new hosts, moved hosts, or duplicate IP addresses.

**Use cases:**
- Detect ARP spoofing attacks in real time
- Monitor for unauthorized devices joining the network
- Track IP/MAC address changes (host migration, virtualization moves)
- Log all ARP activity for forensic analysis
- Alert on duplicate IP address conflicts

**Key advantage:** arpwatch runs as a background daemon, continuously monitoring without manual intervention. It logs to syslog and can send email alerts.

---

## 2. Installation

**Install via apt:**
```bash
sudo apt install arpwatch
```

**Verify installation:**
```bash
arpwatch -V
```

**Start the service:**
```bash
sudo systemctl start arpwatch
sudo systemctl enable arpwatch
```

**Permissions:** arpwatch requires root to capture raw ARP packets.

---

## 3. Core Concepts

### 3.1 ARP Monitoring Process

1. arpwatch listens on the specified interface for ARP packets
2. It builds and maintains a database of known IP/MAC pairs
3. On each ARP event, it checks against the database
4. New hosts, changed mappings, and duplicates trigger alerts

### 3.2 Alert Types

| Alert | Description |
|-------|-------------|
| **new** | New IP/MAC pair seen for the first time |
| **flip flop** | Host moved to a different MAC address |
| **changed** | Same IP, different MAC (potential spoof) |
| **unchanged** | Same IP/MAC seen again (normal) |
| **temporary** | Short-lived entry seen once |
| **deletion** | Entry removed from database |

### 3.3 arpwatch Database

**Default location:** `/var/lib/arpwatch/arp.dat`

The database is a binary file containing all known IP/MAC pairs with timestamps. It persists across restarts.

### 3.4 Syslog Integration

arpwatch logs to syslog (typically `/var/log/syslog` or `/var/log/messages`) with the `arpwatch` facility. Alerts appear as:

```
arpwatch: flip flop 192.168.1.100 (old: aa:bb:cc:dd:ee:ff new: 11:22:33:44:55:66)
arpwatch: new station 192.168.1.200 aa:bb:cc:dd:ee:ff
```

---

## 4. Command Reference

### Basic Usage

```bash
# Monitor on default interface
sudo arpwatch

# Monitor on specific interface
sudo arpwatch -i eth0

# Run in foreground (non-daemon)
sudo arpwatch -d -i eth0
```

### Flags

| Flag | Description |
|------|-------------|
| `-i IFACE` | Network interface to monitor |
| `-f FILE` | Path to database file (default: /var/lib/arpwatch/arp.dat) |
| `-d` | Debug mode — run in foreground |
| `-e EMAIL` | Send alerts to email address |
| `-s EMAIL` | Sender email address for alerts |
| `-P` | Promiscuous mode — capture all ARP traffic |
| `-r FILE` | Read pcap file instead of live traffic |
| `-N` | No hostname resolution |
| `-m EMAIL` | Mail alerts to address |
| `-p` | Do not put interface in promiscuous mode |
| `-a` | Alert on new stations only |
| `-f FILE` | Read existing database from FILE |

---

## 5. Examples

### Basic

**Start monitoring on eth0:**
```bash
sudo arpwatch -i eth0
```

**Run in foreground with debug output:**
```bash
sudo arpwatch -d -i eth0
```
Output:
```
Listening on eth0, packet size 65535
eth0: 192.168.1.1 at 08:00:27:a1:b2:c3 (old: 08:00:27:a1:b2:c3)
eth0: new station 192.168.1.100 aa:bb:cc:dd:ee:ff
eth0: flip flop 192.168.1.50 (old: 00:11:22:33:44:55 new: 66:77:88:99:aa:bb)
```

### Intermediate

**Monitor with email alerts:**
```bash
sudo arpwatch -i eth0 -e admin@company.com -s arpwatch@company.com
```

**Use custom database file:**
```bash
sudo arpwatch -i eth0 -f /home/admin/arp_database.dat
```

**Monitor from pcap file:**
```bash
sudo arpwatch -r captured_arp.pcap -d
```

**Enable promiscuous mode:**
```bash
sudo arpwatch -i eth0 -P
```
Captures all ARP traffic on the network segment, not just traffic addressed to this host.

### Advanced

**Run as systemd service with custom config:**
```bash
# Edit service file
sudo systemctl edit arpwatch

# Add overrides
[Service]
ExecStart=
ExecStart=/usr/sbin/arpwatch -i eth0 -f /etc/arpwatch/arp.dat -e admin@company.com
```

**Monitor multiple interfaces:**
```bash
sudo arpwatch -i eth0 -f /var/lib/arpwatch/arp_eth0.dat &
sudo arpwatch -i eth1 -f /var/lib/arpwatch/arp_eth1.dat &
```

**Parse arpwatch logs:**
```bash
# Find all flip-flop alerts
grep "flip flop" /var/log/syslog | awk '{print $NF}' | sort -u

# Count alerts by type
grep "arpwatch:" /var/log/syslog | awk '{print $3}' | sort | uniq -c | sort -rn
```

### Real-World

**Deploy arpwatch for network security:**
```bash
#!/bin/bash
# Install and configure arpwatch
sudo apt install -y arpwatch

# Create custom database directory
sudo mkdir -p /etc/arpwatch
sudo cp /var/lib/arpwatch/arp.dat /etc/arpwatch/

# Start with email alerts
sudo arpwatch -i eth0 \
    -f /etc/arpwatch/arp.dat \
    -e security@company.com \
    -s arpwatch@company.com \
    -P

echo "arpwatch running on $(hostname) - interface eth0"
```

**Monitor for ARP spoofing:**
```bash
sudo arpwatch -d -i eth0 2>&1 | while read line; do
    if echo "$line" | grep -q "flip flop\|changed"; then
        echo "ALERT: $line" | logger -t arpwatch-alert
        # Send notification
        echo "$line" | mail -s "ARP Alert on $(hostname)" admin@company.com
    fi
done
```

---

## 6. Advanced

### 6.1 Database Management

**Export database to readable format:**
```bash
sudo cp /var/lib/arpwatch/arp.dat /tmp/arp_backup.dat
strings /var/lib/arpwatch/arp.dat | head -50
```

**Reset database:**
```bash
sudo systemctl stop arpwatch
sudo rm /var/lib/arpwatch/arp.dat
sudo systemctl start arpwatch
```

### 6.2 Log Analysis

**Monitor real-time alerts:**
```bash
sudo tail -f /var/log/syslog | grep arpwatch
```

**Weekly report script:**
```bash
#!/bin/bash
WEEK=$(date -d "7 days ago" +%Y-%m-%d)
echo "ARP Events since $WEEK:"
grep "arpwatch:" /var/log/syslog | awk -v d="$WEEK" '$0 >= d' | \
    awk '{print $3}' | sort | uniq -c | sort -rn
```

### 6.3 Integration with SIEM

**Forward arpwatch logs to syslog server:**
```bash
# In /etc/rsyslog.conf
if $programname == 'arpwatch' then @siem-server:514
```

**Parse for SIEM ingestion:**
```bash
grep "arpwatch:" /var/log/syslog | \
    awk '{gsub(/:/, ":", $0); print}'
```

### 6.4 Multiple Interface Monitoring

**Dual-homed monitoring:**
```bash
# Interface 1
sudo arpwatch -i eth0 -f /var/lib/arpwatch/arp_eth0.dat -d &

# Interface 2
sudo arpwatch -i eth1 -f /var/lib/arpwatch/arp_eth1.dat -d &

# Merge databases periodically
sudo arpwatch -i eth0 -r /var/lib/arpwatch/arp_eth0.dat &
```

---

## 7. Detection and Defense

### 7.1 Detection

**arpwatch alerts indicate:**
- **flip flop**: Host changed MAC (possible spoofing or VM migration)
- **new station**: Unknown device on network
- **changed**: IP reassigned to different MAC (DHCP issue or attack)

**Monitor for alerts:**
```bash
sudo tail -f /var/log/syslog | grep -E "flip flop|new station|changed"
```

### 7.2 Defense

**Response to flip-flop alerts:**
1. Verify if change is legitimate (VM migration, new NIC)
2. Check physical connection if unexpected
3. Block suspicious MAC at switch port

**Static ARP for critical servers:**
```bash
sudo arp -s 192.168.1.1 08:00:27:a1:b2:c3
```

**Switch port security:**
Configure port security to limit MAC addresses per port and restrict to known MACs.

**DHCP snooping:**
Enable DHCP snooping to prevent unauthorized DHCP servers and ARP spoofing.

---

## 8. Troubleshooting

### 8.1 Common Issues

**"No such device":**
```bash
ip -br link show
sudo arpwatch -i eth0
```
Verify the interface name matches an existing interface.

**arpwatch not starting:**
```bash
sudo systemctl status arpwatch
sudo journalctl -u arpwatch
```

**No alerts appearing:**
- Check if interface is receiving ARP: `sudo tcpdump -i eth0 arp`
- Verify database file permissions: `ls -la /var/lib/arpwatch/`
- Run in debug mode: `sudo arpwatch -d -i eth0`

**Database corruption:**
```bash
sudo systemctl stop arpwatch
sudo rm /var/lib/arpwatch/arp.dat
sudo systemctl start arpwatch
```
Database rebuilds automatically from scratch.

### 8.2 Performance Issues

**High CPU usage:**
```bash
sudo arpwatch -i eth0 -p  # Disable promiscuous mode
```
Promiscuous mode captures more traffic, increasing CPU.

**Missing alerts:**
```bash
sudo arpwatch -i eth0 -P  # Enable promiscuous mode
```
Ensure promiscuous mode is on for accurate monitoring.

### 8.3 Log Management

**Rotate arpwatch logs:**
```bash
# In /etc/logrotate.d/arpwatch
/var/log/syslog {
    daily
    rotate 30
    compress
    postrotate
        systemctl reload arpwatch
    endscript
}
```

---

## Resources

- [arpwatch man page](https://linux.die.net/man/8/arpwatch)
- [arpwatch source (original)](https://ee.lbl.gov/)
- [Kali arpwatch tool page](https://www.kali.org/tools/arpwatch/)
- [ARP Protocol (RFC 826)](https://tools.ietf.org/html/rfc826)
- [arpwatch configuration guide](https://www.oreilly.com/)
