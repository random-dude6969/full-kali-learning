---
title: "darkstat - Network Traffic Analyzer with Embedded Web Server"
description: "Passive network traffic analyzer that captures packets and serves bandwidth usage statistics via an embedded HTTP server"
mitre_tactic: discovery
mitre_tactic_id: TA0007
mitre_subcategory: network_sniffing
categories:
  - Network Traffic Analysis
  - Bandwidth Monitoring
  - Host Profiling
  - Passive Monitoring
tags:
  - traffic-analysis
  - bandwidth
  - monitoring
  - web-interface
  - passive
  - host-tracking
install_command: "sudo apt install darkstat"
version: "3.0.717"
github_repo: "https://github.com/nicmcd/darkstat"
kali_tools_page: "https://www.kali.org/tools/darkstat/"
difficulty_level: beginner
requires_root: true
gui_available: true
related_tools:
  - nethogs
  - iftop
  - nload
  - bmon
  - vnstat
---

# darkstat — Network Traffic Analyzer with Embedded Web Server

## Chapter 1: Introduction

### What is darkstat

**darkstat** is a passive network traffic analyzer that captures packets from a network interface and provides real-time bandwidth usage statistics through an embedded HTTP server. It tracks per-host and per-port traffic, maintains historical data, and presents results through a web-based interface accessible from any browser.

Unlike active tools that probe the network, darkstat operates passively — it silently captures and analyzes all traffic flowing through the monitored interface without sending any packets. This makes it ideal for network monitoring in production environments where active probing would be disruptive or detected.

### Historical Context

darkstat was written by Evan Anderson and released in 2001. It was one of the first lightweight, daemon-based network traffic analyzers for Linux. The tool was designed to fill the gap between simple bandwidth monitors (like iftop) and complex network management systems (like MRTG/Cacti).

The name "darkstat" is a play on "dark" (passive, stealthy) and "stat" (statistics). The tool's minimal footprint and embedded web server made it popular for headless servers and embedded systems where running a full monitoring stack was impractical.

darkstat's simplicity and reliability have kept it relevant for over two decades. It remains a go-to tool for quick network visibility without the overhead of more complex solutions.

### When to Use darkstat

- **Bandwidth monitoring**: Track per-host and per-port bandwidth usage
- **Network discovery**: Identify all hosts communicating on a network segment
- **Traffic baselining**: Establish normal traffic patterns for anomaly detection
- **Quick triage**: Get instant visibility into network activity without complex setup
- **Headless servers**: Monitor traffic on servers without a desktop environment
- **Historical analysis**: Review past traffic patterns from stored data

### Core Concepts

- **Passive capture**: Analyzes traffic without generating any packets
- **Per-host tracking**: Maintains statistics for each source/destination IP
- **Per-port analysis**: Tracks traffic by TCP/UDP port number
- **Bandwidth calculation**: Computes bits/second, packets/second, and total bytes
- **Embedded HTTP server**: Serves real-time statistics on a configurable port
- **Reverse DNS**: Optionally resolves IP addresses to hostnames
- **Data persistence**: Stores historical data for trend analysis

---

## Chapter 2: Installation

### System Requirements

| Requirement | Minimum | Recommended |
|------------|---------|-------------|
| OS | Kali Linux 2020+ | Kali Linux 2024+ |
| RAM | 32MB | 128MB+ |
| CPU | Single core | Single core |
| Disk | 5MB | 100MB+ for data storage |
| Network | Any NIC | Promiscuous-capable NIC |
| Root | Required | Required |

### Installation Methods

**Method 1: Kali Package Manager (Recommended)**

```bash
sudo apt update
sudo apt install darkstat
```

**Method 2: From Source**

```bash
sudo apt install git build-essential libpcap-dev
git clone https://github.com/nicmcd/darkstat.git
cd darkstat
./configure
make
sudo make install
```

**Method 3: Compile from Tarball**

```bash
wget https://mfries.org/darkstat/darkstat-3.0.717.tar.gz
tar xzf darkstat-3.0.717.tar.gz
cd darkstat-3.0.717
./configure
make
sudo make install
```

### Post-Installation Verification

```bash
# Verify installation
darkstat --version

# Check available interfaces
ip link show

# Test web interface
darkstat -i lo -p 6666 &
sleep 2
curl http://localhost:6666
```

### Initial Configuration

```bash
# Start darkstat on interface eth0
sudo darkstat -i eth0 -p 6666

# Access web interface
# Open browser to http://localhost:6666
```

---

## Chapter 3: Core Concepts

### How darkstat Works

darkstat operates through a simple pipeline:

```
[Network Interface] → [Packet Capture (libpcap)] → [Traffic Counter] → [HTTP Server] → [Web Browser]
```

1. **Packet capture**: Uses libpcap to capture all packets on the interface
2. **Traffic counting**: Increments per-host and per-port counters for each packet
3. **Bandwidth calculation**: Computes rates from packet sizes and timestamps
4. **DNS resolution**: Optionally resolves IPs to hostnames
5. **HTTP server**: Serves statistics on the configured port
6. **Data persistence**: Stores data for historical analysis

### Traffic Tracking Model

darkstat tracks traffic using a dual-axis model:

**Per-host tracking:**

| Metric | Description |
|--------|-------------|
| Bytes in | Total bytes received from host |
| Bytes out | Total bytes sent to host |
| Packets in | Total packets received from host |
| Packets out | Total packets sent to host |
| First seen | Timestamp of first packet |
| Last seen | Timestamp of most recent packet |

**Per-port tracking:**

| Metric | Description |
|--------|-------------|
| Bytes | Total bytes on port |
| Packets | Total packets on port |
| Hosts | Number of unique hosts using port |
| Protocol | TCP or UDP |

### Architecture

```
┌─────────────────────────────────────────────┐
│                darkstat                      │
├─────────────┬─────────────┬─────────────────┤
│  Capture    │  Analysis   │  Web Server     │
│  Engine     │  Engine     │  Engine         │
├─────────────┼─────────────┼─────────────────┤
│ libpcap     │ Per-host    │ Embedded HTTP   │
│ Promiscuous │ counters    │ HTML pages      │
│ mode        │ Per-port    │ CSS/JS          │
│             │ counters    │ JSON API        │
│             │ Bandwidth   │                 │
│             │ calculation │                 │
└─────────────┴─────────────┴─────────────────┘
```

### Web Interface Pages

| Page | Description |
|------|-------------|
| `/` | Dashboard with traffic summary |
| `/host/<ip>` | Per-host traffic details |
| `/port/<port>` | Per-port traffic details |
| `/graph` | Traffic graphs |
| `/export` | Export data in various formats |

---

## Chapter 4: Command Reference

### Basic Syntax

```bash
darkstat [OPTIONS] -i <interface>
```

### Core Flags

| Flag | Long | Description | Example |
|------|------|-------------|---------|
| `-i` | `--interface` | Network interface to monitor | `-i eth0` |
| `-p` | `--port` | HTTP server port | `-p 6666` |
| `-b` | `--bind` | HTTP server bind address | `-b 192.168.1.1` |
| `-f` | `--filter` | BPF capture filter | `-f "not port 22"` |
| `-d` | `--chdir` | Change to directory | `-d /var/lib/darkstat` |
| `-l` | `--logfile` | Log file path | `-l /var/log/darkstat.log` |
| `-n` | `--no-promisc` | Don't enable promiscuous mode | `-n` |
| `-v` | `--verbose` | Verbose logging | `-v` |

### Web Server Flags

| Flag | Description | Example |
|------|-------------|---------|
| `-p` | HTTP server port | `-p 6666` |
| `-b` | Bind address | `-b 0.0.0.0` |
| `--no-dns` | Disable reverse DNS | `--no-dns` |
| `--no-mac` | Don't show MAC addresses | `--no-mac` |

### Capture Flags

| Flag | Description | Example |
|------|-------------|---------|
| `-i` | Interface to capture | `-i eth0` |
| `-f` | BPF filter | `-f "net 192.168.1.0/24"` |
| `-n` | No promiscuous mode | `-n` |
| `-m` | Maximum packet length | `-m 65535` |

### Persistence Flags

| Flag | Description | Example |
|------|-------------|---------|
| `-d` | Data directory | `-d /var/lib/darkstat` |
| `--day-log` | Daily log file | `--day-log /var/log/darkstat/daily.log` |
| `--month-log` | Monthly log file | `--month-log /var/log/darkstat/monthly.log` |

### Output Flags

| Flag | Description | Example |
|------|-------------|---------|
| `-l` | Log file | `-l /var/log/darkstat.log` |
| `--verbose` | Verbose output | `-v` |

---

## Chapter 5: Examples

### Basic Examples

**Start darkstat on an interface:**

```bash
sudo darkstat -i eth0 -p 6666
```

**Access web interface:**

```bash
# Open browser to http://localhost:6666
xdg-open http://localhost:6666
```

**Monitor with custom port:**

```bash
sudo darkstat -i eth0 -p 8080
```

### Intermediate Examples

**Bind to specific address:**

```bash
# Only accessible from localhost
sudo darkstat -i eth0 -p 6666 -b 127.0.0.1

# Accessible from network
sudo darkstat -i eth0 -p 6666 -b 192.168.1.1
```

**Filter specific traffic:**

```bash
# Monitor only HTTP/HTTPS
sudo darkstat -i eth0 -p 6666 -f "port 80 or port 443"

# Exclude management traffic
sudo darkstat -i eth0 -p 6666 -f "not port 22 and not port 3389"

# Monitor specific subnet
sudo darkstat -i eth0 -p 6666 -f "net 192.168.1.0/24"
```

**Disable reverse DNS:**

```bash
# Faster operation without DNS lookups
sudo darkstat -i eth0 -p 6666 --no-dns
```

### Advanced Examples

**Full monitoring setup:**

```bash
#!/bin/bash
# darkstat_monitor.sh — Production monitoring setup

INTERFACE="eth0"
PORT="6666"
LOGDIR="/var/log/darkstat"
DATADIR="/var/lib/darkstat"

# Create directories
mkdir -p "$LOGDIR" "$DATADIR"

# Start darkstat with full options
sudo darkstat \
    -i "$INTERFACE" \
    -p "$PORT" \
    -b 0.0.0.0 \
    -d "$DATADIR" \
    -l "$LOGDIR/darkstat.log" \
    --day-log "$LOGDIR/daily.log" \
    --month-log "$LOGDIR/monthly.log" \
    -f "not port 22" \
    --verbose

echo "darkstat running on http://$(hostname -I | awk '{print $1}'):$PORT"
```

**Multi-interface monitoring:**

```bash
# Monitor internal network
sudo darkstat -i eth0 -p 6666 -b 192.168.1.1 -f "net 192.168.1.0/24" &

# Monitor DMZ
sudo darkstat -i eth1 -p 6667 -b 10.0.0.1 -f "net 10.0.0.0/24" &

wait
```

**Headless server monitoring:**

```bash
# Start darkstat as a daemon
sudo darkstat -i eth0 -p 6666 -b 0.0.0.0 --no-dns -d /var/lib/darkstat

# Access from remote machine
# http://server-ip:6666
```

### Real-World Scenarios

**Bandwidth monitoring for capacity planning:**

```bash
# Monitor for 24 hours to establish baseline
sudo darkstat -i eth0 -p 6666 -d /var/lib/darkstat/baseline

# Review after 24 hours
# Access web interface and note top talkers
# Export data for analysis
curl http://localhost:6666/export > /tmp/darkstat_export.json
```

**Incident response — identify suspicious traffic:**

```bash
# Quick triage of network activity
sudo darkstat -i eth0 -p 6666 --no-dns -f "not port 22"

# Check for unusual hosts
# Look for high bandwidth to external IPs
# Review port usage patterns
```

**Compliance monitoring — track data exfiltration:**

```bash
# Monitor outbound traffic
sudo darkstat -i eth0 -p 6666 \
    -b 192.168.1.1 \
    -f "src net 192.168.1.0/24 and dst not net 192.168.1.0/24" \
    -d /var/lib/darkstat/compliance

# Review external connections
# Alert on unusual data volumes
```

---

## Chapter 6: Advanced Usage

### Scripting with darkstat

**Automated monitoring with alerting:**

```bash
#!/bin/bash
# monitor_bandwidth.sh — Alert on high bandwidth usage

INTERFACE="eth0"
THRESHOLD_MBPS=100
CHECK_INTERVAL=60

while true; do
    # Get current bandwidth from darkstat
    BW=$(curl -s http://localhost:6666/api/traffic 2>/dev/null | jq -r '.total_bytes_per_second // 0')
    BW_MBPS=$(echo "scale=2; $BW * 8 / 1000000" | bc 2>/dev/null || echo "0")

    if (( $(echo "$BW_MBPS > $THRESHOLD_MBPS" | bc -l 2>/dev/null || echo 0) )); then
        logger -t darkstat_monitor "HIGH BANDWIDTH: ${BW_MBPS} Mbps exceeds ${THRESHOLD_MBPS} Mbps threshold"
    fi

    sleep "$CHECK_INTERVAL"
done
```

### Chaining with Other Tools

**darkstat + tcpdump for detailed capture:**

```bash
# darkstat for overview
sudo darkstat -i eth0 -p 6666 &

# tcpdump for detailed capture when suspicious activity detected
sudo tcpdump -i eth0 -w /tmp/suspicious_$(date +%Y%m%d_%H%M%S).pcap -c 10000 &

wait
```

**darkstat + grep for log analysis:**

```bash
# Analyze darkstat logs for unusual patterns
grep "new host" /var/log/darkstat.log | tail -20

# Find high-bandwidth connections
grep "bytes" /var/log/darkstat.log | sort -t' ' -k5 -n -r | head -10
```

**darkstat + cron for periodic reporting:**

```bash
# Add to crontab
0 */6 * * * /usr/local/bin/darkstat_report.sh

# Report script
#!/bin/bash
curl -s http://localhost:6666/export > /tmp/darkstat_$(date +%Y%m%d_%H).json
```

### Configuration Tuning

**Performance optimization for busy networks:**

```bash
# Increase kernel buffers
sudo sysctl -w net.core.rmem_max=26214400
sudo sysctl -w net.core.netdev_max_backlog=5000

# Start darkstat with optimized settings
sudo darkstat -i eth0 -p 6666 -n --no-dns -m 65535
```

**Reducing resource usage:**

```bash
# Disable DNS resolution (reduces CPU usage)
sudo darkstat -i eth0 -p 6666 --no-dns

# Don't show MAC addresses (reduces memory)
sudo darkstat -i eth0 -p 6666 --no-mac

# Filter to reduce capture volume
sudo darkstat -i eth0 -p 6666 -f "not port 53"
```

---

## Chapter 7: Detection and Defense

### How darkstat Appears in Logs

darkstat is a passive monitoring tool — it doesn't generate network traffic. However, its presence can be detected:

**Process artifacts:**

```bash
# Detect darkstat running
ps aux | grep darkstat
pgrep -a darkstat

# Check listening ports
ss -tlnp | grep darkstat
netstat -tlnp | grep darkstat
```

**Network artifacts:**

```bash
# darkstat listens on configured port
curl http://target-ip:6666

# Check for HTTP server
nmap -sV -p 6666 target-ip
```

**Log artifacts:**

```bash
# Check for darkstat logs
find /var/log -name "darkstat*" 2>/dev/null
find /var/lib -name "darkstat*" 2>/dev/null
```

### Defensive Monitoring

**Monitor for darkstat deployment:**

```bash
#!/bin/bash
# detect_darkstat.sh

# Watch for darkstat binary
auditctl -w /usr/bin/darkstat -p x -k darkstat_exec

# Watch for darkstat configuration
auditctl -w /etc/darkstat/ -p wa -k darkstat_config

# Monitor for HTTP server on common darkstat ports
ss -tlnp | grep -E ":6666|:6667|:8080"
```

### Evasion Considerations

darkstat's passive nature makes it difficult to detect, but:

- **Encrypted traffic**: TLS/SSL prevents payload inspection
- **High-volume networks**: May miss packets under heavy load
- **VLAN segmentation**: Only captures traffic on the monitored segment
- **Promiscuous mode**: Can be detected through ARP anomaly checks

**Defensive countermeasures:**

```bash
# Monitor for promiscuous mode interfaces
cat /sys/class/net/*/flags | while read flags; do
    if [ $((flags & 1)) -ne 0 ]; then
        echo "Promiscuous mode detected"
    fi
done

# Use network segmentation to limit monitoring scope
# Deploy IDS/IPS to detect monitoring tools
```

---

## Chapter 8: Troubleshooting and FAQ

### Common Issues

**"No such device" error:**

```bash
# Problem: Interface doesn't exist
# Diagnosis
ip link show

# Solution: Use correct interface name
sudo darkstat -i eth0 -p 6666
```

**Port already in use:**

```bash
# Problem: Port 6666 is occupied
# Diagnosis
ss -tlnp | grep 6666

# Solution: Use different port
sudo darkstat -i eth0 -p 6667
```

**Web interface not accessible:**

```bash
# Problem: Binding issue or firewall
# Diagnosis
curl http://localhost:6666
iptables -L -n

# Solution: Bind to correct address
sudo darkstat -i eth0 -p 6666 -b 0.0.0.0
```

**High CPU usage:**

```bash
# Problem: Processing too many packets
# Solution: Apply BPF filter
sudo darkstat -i eth0 -p 6666 -f "net 192.168.1.0/24"

# Alternative: Disable DNS
sudo darkstat -i eth0 -p 6666 --no-dns
```

### FAQ

**Q: Does darkstat capture packets?**
A: darkstat captures packet headers for analysis but doesn't store full packet contents by default. It focuses on traffic statistics rather than packet-level forensics.

**Q: Can darkstat monitor multiple interfaces?**
A: Each darkstat instance monitors one interface. Run multiple instances with different ports for multi-interface monitoring.

**Q: Is darkstat secure?**
A: darkstat's web interface has no authentication. Bind to localhost (`-b 127.0.0.1`) or use firewall rules to restrict access.

**Q: How much bandwidth does darkstat add?**
A: darkstat is passive and generates no network traffic. It only reads packets from the monitored interface.

**Q: Can darkstat export data?**
A: Yes. Use the web interface export function or curl the API endpoints for JSON data.

---

## Resources

- **GitHub Repository**: https://github.com/nicmcd/darkstat
- **Kali Tools Page**: https://www.kali.org/tools/darkstat/
- **Related Tools**: nethogs, iftop, nload, bmon, vnstat
- **Network Monitoring**: MRTG, Cacti, Zabbix, Prometheus
- **MITRE ATT&CK**: [TA0007 - Discovery](https://attack.mitre.org/tactics/TA0007/)
- **Packet Capture**: tcpdump, libpcap documentation
