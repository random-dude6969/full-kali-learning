# Ferret-Sidejack Cheatsheet

## Quick Start

```bash
# Live capture on interface
sudo ferret-sidejack -i eth0

# Offline pcap analysis
ferret-sidejack -r capture.pcap

# Multiple pcap files
ferret-sidejack -r *.pcap
```

## Basic Commands

| Command | Description |
|---------|-------------|
| `ferret-sidejack -i <iface>` | Live capture on interface |
| `ferret-sidejack -r <file.pcap>` | Analyze pcap file offline |
| `ferret-sidejack -r *.pcap` | Analyze multiple pcaps |
| `ferret-sidejack -i <iface> -c <config>` | Custom configuration file |

## Capture Modes

### Live Capture

```bash
# Ethernet
sudo ferret-sidejack -i eth0

# Wireless (monitor mode required)
sudo ferret-sidejack -i wlan0mon

# With logging
sudo ferret-sidejack -i eth0 2>&1 | tee ferret_output.log
```

### Offline Analysis

```bash
# Single file
ferret-sidejack -r capture.pcap

# Multiple files
ferret-sidejack -r capture1.pcap capture2.pcap

# Wildcard
ferret-sidejack -r *.pcap
```

## Network Interface Setup

```bash
# List interfaces
ip link show

# Enable monitor mode (wireless)
sudo ip link set wlan0 down
sudo iwconfig wlan0 mode monitor
sudo ip link set wlan0 up

# Or using airmon-ng
sudo airmon-ng start wlan0
```

## Output and Logging

```bash
# Console output with logging
sudo ferret-sidejack -i eth0 2>&1 | tee ferret_log.txt

# Combined with tcpdump
sudo tcpdump -i eth0 -w full_capture.pcap &
sudo ferret-sidejack -i eth0
```

## What Ferret Captures

| Data Type | Description |
|-----------|-------------|
| HTTP Cookies | Session identifiers, auth tokens |
| Authorization Headers | Basic auth, Bearer tokens |
| POST Data | Form submissions (usernames, passwords) |
| Referer URLs | Navigation history |
| User-Agent | Browser fingerprinting |

## Limitations

| Limitation | Description |
|-----------|-------------|
| **HTTP Only** | Cannot capture HTTPS/TLS cookies |
| **Passive Only** | Does not inject or modify traffic |
| **No Decryption** | Encrypted traffic is opaque |
| **Plaintext Required** | Only unencrypted protocols yield data |

## Integration with Hamster

```bash
# Terminal 1: Ferret captures cookies
sudo ferret-sidejack -i wlan0

# Terminal 2: Hamster uses captured cookies
sudo hamster-sidejack
# Configure browser proxy to http://127.0.0.1:1234
```

## Practical Patterns

### Internal Network Monitoring

```bash
# Monitor internal HTTP traffic
sudo ferret-sidejack -i eth0 2>&1 | tee internal_capture.log

# Analyze results
grep -i "cookie" internal_capture.log
grep -i "authorization" internal_capture.log
```

### Wireless Capture

```bash
# Enable monitor mode
sudo airmon-ng start wlan0

# Capture
sudo ferret-sidejack -i wlan0mon 2>&1 | tee wireless_cookies.log

# Feed to Hamster
sudo hamster-sidejack
```

### Pcap Forensics

```bash
# Analyze incident capture
ferret-sidejack -r incident.pcap 2>&1 > incident_cookies.txt

# Search for specific sessions
grep -i "session_id" incident_cookies.txt

# Extract unique cookies
sort -u incident_cookies.txt > unique_cookies.txt
```

## Configuration File Format

```bash
# Example ferret.conf
# Filter by IP
filter src 192.168.1.0/24

# Filter by port
filter port 80

# Output format
output text
```

## Troubleshooting

```bash
# Permission denied
sudo ferret-sidejack -i eth0

# Interface not found
ip link show
sudo airmon-ng start wlan0

# No cookies captured
# Check if HTTP (non-HTTPS) traffic exists
sudo tcpdump -i eth0 port 80 -c 20

# Low capture rate
sudo tcpdump -i eth0 -w http_only.pcap 'tcp port 80' &
ferret-sidejack -r http_only.pcap
```

## Detection Indicators

| Indicator | Description |
|-----------|-------------|
| Promiscuous mode | Network adapter in promiscuous mode |
| Port 80 traffic analysis | Unusual HTTP parsing patterns |
| ARP anomalies | ARP spoofing for positioning |
| Console output | Ferret output to terminal/logs |

## File Locations

```bash
# Default output locations
ls ~/hamster/cookies.txt
ls /tmp/ferret*.log

# Hamster reads from
cat ~/hamster/cookies.txt
```

## Related Tools

| Tool | Purpose |
|------|---------|
| `hamster-sidejack` | Session replay proxy |
| `wireshark` | Full packet analysis |
| `tcpdump` | Packet capture |
| `bettercap` | ARP spoofing + MITM |
| `dsniff` | Network credential sniffer |
