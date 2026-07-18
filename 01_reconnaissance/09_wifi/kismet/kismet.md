---
tool_name: "kismet"
display_name: "Kismet"
mitre_tactic: "reconnaissance"
mitre_tactic_id: "TA0043"
mitre_subcategory: "wifi"
install_command: "sudo apt install kismet"
version: "2024.09.R1"
github_repo: "https://github.com/kismetwireless/kismet"
kali_tools_page: "https://www.kali.org/tools/kismet/"
difficulty_level: "intermediate"
requires_root: true
gui_available: true
tags: ["wifi", "sniffer", "ids", "wireless", "bluetooth", "drone-detection", "packet-capture", "channel-hopping", "gps"]
related_tools: ["airodump-ng", "wireshark", "bettercap", "hcxdumptool"]
---

# Kismet — Wireless Network Detector, Sniffer, and IDS

## What This Tool Does (in 30 seconds)

Kismet is a comprehensive wireless network detector, sniffer, and intrusion detection system (IDS). It passively detects networks and clients, captures wireless traffic, and identifies potential security threats. Kismet works with any wireless card that supports raw monitoring mode and can detect 802.11 WiFi, Bluetooth, Zigbee, and other wireless protocols via its extensible datasource plugin system.

---

## Chapter 1: Introduction & Overview

### What is Kismet?

Kismet is the gold-standard open-source wireless network detection, sniffing, and intrusion detection framework. Unlike active tools that transmit probe requests, Kismet operates primarily in passive mode — it listens for wireless frames without transmitting, making it stealthier and more reliable for comprehensive network discovery.

Kismet's architecture has evolved significantly. Originally a passive 802.11 sniffer, it now functions as a complete wireless IDS/IPS framework supporting multiple protocols, GPS-based network mapping, and a modern client-server architecture with a web-based UI.

### History & Development

| Year | Milestone |
|------|-----------|
| 2001 | Kismet created by Mike Kershaw (dragorn) |
| 2005 | Plugin system introduced |
| 2010 | BLE (Bluetooth Low Energy) support added |
| 2013 | Drone detection capabilities added |
| 2018 | Major rewrite — Python 2 removed, server/client architecture |
| 2020 | Web-based UI replaces ncurses |
| 2022 | SQLite database backend, kismet_db analysis tool |
| 2024 | Current stable release with improved SDR support |

### MITRE ATT&CK Mapping

| Tactic | Technique | ID | Description |
|--------|-----------|----|----|
| Reconnaissance | Network Service Discovery | T1046 | Active scanning of wireless networks |
| Reconnaissance | Wireless Network Discovery | T1590.004 | Identifying wireless infrastructure |
| Collection | Network Sniffing | T1040 | Capturing wireless traffic |
| Credential Access | Network Sniffing | T1557.001 | Capturing authentication data |

### Key Concepts

#### Passive vs Active Detection

Kismet operates primarily in **passive mode**:

- **Passive Detection**: Listens for 802.11 beacon frames, probe responses, data frames, and management frames without transmitting. This makes the detector invisible to the network.
- **Active Detection**: Sends probe requests to discover networks (configurable). Active detection reveals the presence of the scanner.
- **Benefits**: Passive mode is stealthier, captures more complete data, and detects hidden networks via client probe requests.
- **Limitations**: May miss networks with no clients sending probe requests if beacon suppression is enabled.

#### Network Types Detected

| Protocol | Description | Detection Method |
|----------|-------------|-----------------|
| 802.11 WiFi | Standard wireless networks | Beacon/probe/data frames |
| Bluetooth Classic | BR/EDR connections | Inquiry/scan responses |
| Bluetooth LE (BLE) | IoT devices, beacons | Advertisement packets |
| Zigbee | Smart home networks | Frame analysis (with SDR) |
| Software Defined Radio | Custom protocols | SoapySDR integration |

#### Data Sources (Datasources)

Kismet uses a modular datasource plugin system:

| Datasource | Description | Requirements |
|------------|-------------|-------------|
| `linuxwifi` | nl80211-based WiFi capture | Linux, monitor mode capable driver |
| `linuxbluetooth` | BlueZ-based Bluetooth capture | BlueZ stack, hci device |
| `android` | Remote capture via Android app | Kismet Android client |
| `rtl433` | RTL-SDR for 433MHz ISM band | RTL-SDR dongle |
| `rtlsdr` | General SDR capture | RTL-SDR dongle |
| `hackrf` | HackRF SDR capture | HackRF device |
| `nrf24l01` | NRF24L01+ based capture | NRF24L01 hardware |

### Client-Server Architecture

```
┌──────────────────┐     ┌──────────────────┐     ┌──────────────────┐
│  Capture Sources │────▶│   Kismet Server   │────▶│  Kismet Client   │
│  (linuxwifi,     │     │                   │     │  (Web Browser)   │
│   bluetooth,     │     │  - Packet decode  │     │                  │
│   SDR plugins)   │     │  - Device tracking│     │  - Real-time map │
└──────────────────┘     │  - Alert engine   │     │  - Network list  │
                         │  - Logging        │     │  - Signal graphs │
                         │  - REST API       │     │  - Device details│
                         └──────────────────┘     └──────────────────┘
```

The server runs as a daemon, processing and logging all wireless data. Clients connect via HTTPS REST API to view and interact with the data. Multiple clients can connect simultaneously.

---

## Chapter 2: Installation & Setup

### System Requirements

| Component | Minimum | Recommended |
|-----------|---------|-------------|
| CPU | Dual-core 1GHz | Quad-core 2GHz+ |
| RAM | 1 GB | 4 GB+ |
| Disk | 100 MB | 10 GB+ (logs grow fast) |
| Network | Monitor-mode WiFi adapter | Multiple adapters |
| OS | Linux (kernel 3.x+) | Kali Linux / Ubuntu 22.04+ |

### Installation Methods

#### Method 1: APT (Recommended)

```bash
# Update package lists
sudo apt update

# Install Kismet
sudo apt install kismet

# Verify installation
kismet --version
# Output: Kismet 2024.09.R1
```

#### Method 2: From Source (Latest Features)

```bash
# Install build dependencies
sudo apt install build-essential cmake libwebsockets-dev libprotobuf-c-dev \
  protobuf-c-compiler libnl-3-dev libnl-genl-3-dev libsqlite3-dev \
  libgps-dev libpcap-dev libusb-1.0-0-dev libavahi-client-dev \
  libpcre2-dev libnm-dev libdw-dev libpulse-dev

# Clone repository
git clone https://github.com/kismetwireless/kismet.git
cd kismet

# Build
./configure
make -j$(nproc)
sudo make install

# Create systemd service
sudo make install-systemd
sudo systemctl daemon-reload
sudo systemctl enable kismet
sudo systemctl start kismet
```

#### Method 3: Docker (Remote Capture)

```bash
# Pull Kismet Docker image
docker pull kismetremote/kismet

# Run Kismet in container
docker run -it --rm --net=host --privileged \
  -v /dev:/dev \
  -v ~/.kismet:/root/.kismet \
  kismetremote/kismet
```

### Configuration

Kismet stores its configuration in `/etc/kismet/kismet.conf`:

```bash
# Key configuration parameters
cat /etc/kismet/kismet.conf | grep -v "^#" | grep -v "^$"

# Server bind address and ports
detection_server_host=127.0.0.1
detection_server_port=2501

# TLS configuration (auto-generated on first run)
tls_cert=/etc/kismet/kismet_httpd-ssl.pem
tls_key=/etc/kismet/kismet_httpd-ssl.key

# Logging
log_types=kismet,pcap,netxml,netjson,gps
log_prefix=/var/log/kismet
log_template=%p-%n-%D-%t-%i

# Source configuration (can also be set via UI)
# source=wlan0:name=MyWiFi
```

### Post-Installation Setup

```bash
# Add user to kismet group for non-root capture
sudo usermod -a -G kismet $USER

# Create log directory
sudo mkdir -p /var/log/kismet
sudo chown root:kismet /var/log/kismet
sudo chmod 775 /var/log/kismet

# Start monitor mode on your adapter
sudo ip link set wlan0 down
sudo iw wlan0 set type monitor
sudo ip link set wlan0 up

# Start Kismet
sudo kismet -c wlan0
```

### Verification

```bash
# Check Kismet is running
sudo systemctl status kismet

# Check web UI is accessible
curl -s -o /dev/null -w "%{http_code}" http://127.0.0.1:2501
# Output: 200

# Check datasources
kismet_db --list-sources 2>/dev/null || echo "Check via web UI"
```

---

## Chapter 3: Basic Usage

### First Scan

```bash
# Start Kismet with a specific interface
sudo kismet -c wlan0

# Open web browser to interact
# Navigate to https://localhost:2501
```

### Command-Line Flags Reference

#### Core Operation Flags

| Flag | Description | Example |
|------|-------------|---------|
| `-c` | Specify capture source(s) | `kismet -c wlan0` |
| `-c iface:name=X` | Named capture source | `kismet -c wlan0:name=OfficeWiFi` |
| `-c iface:channel=X` | Lock to specific channel | `kismet -c wlan0:channel=6` |
| `-c iface:hop=X` | Channel hop rate (/sec) | `kismet -c wlan0:hop=5` |
| `-c iface:uuid=X` | Assign UUID to source | `kismet -c wlan0:uuid=abc123` |
| `-w` | Write logs to prefix | `kismet -w capture_prefix` |
| `--no-ui` / `--no-gui` | Run without web UI | `kismet --no-gui` |
| `-f` | Use config file | `kismet -f /etc/kismet/kismet.conf` |
| `--daemonize` | Run as background daemon | `kismet --daemonize` |
| `--pid-file` | PID file path | `kismet --pid-file /var/run/kismet.pid` |
| `-t` | Log type override | `kismet -t pcap` |
| `--log-types` | Comma-separated log types | `kismet --log-types kismet,pcap` |
| `--log-prefix` | Override log directory | `kismet --log-prefix /tmp/kismet` |
| `--no-logging` | Disable all logging | `kismet --no-logging` |
| `-h` / `--help` | Show help | `kismet -h` |
| `-v` / `--version` | Show version | `kismet -v` |

#### Network & Server Flags

| Flag | Description | Example |
|------|-------------|---------|
| `--server-host` | Server bind address | `kismet --server-host 0.0.0.0` |
| `--http-port` | HTTP port | `kismet --http-port 8080` |
| `--tls-port` | HTTPS port | `kismet --tls-port 8443` |
| `--tcp-port` | Legacy TCP port | `kismet --tcp-port 2501` |
| `--server-name` | Server display name | `kismet --server-name "HQ-WIDS"` |
| `--disable-api` | Disable REST API | `kismet --disable-api` |

#### GPS Flags

| Flag | Description | Example |
|------|-------------|---------|
| `--use-gpsd` | Enable GPSD integration | `kismet --use-gpsd` |
| `--gpsd-host` | GPSD host address | `kismet --gpsd-host 127.0.0.1` |
| `--gpsd-port` | GPSD port | `kismet --gpsd-port 2947` |
| `--use-gps` | Legacy GPS support | `kismet --use-gps` |

#### Filtering Flags

| Flag | Description | Example |
|------|-------------|---------|
| `--filter-tracked` | Filter tracked devices | `kismet --filter-tracked WPA2` |
| `--filter-phyname` | Filter by PHY type | `kismet --filter-phyname IEEE802.11` |
| `--filter-source` | Filter by datasource | `kismet --filter-source wlan0` |

#### Alert Flags

| Flag | Description | Example |
|------|-------------|---------|
| `--alert` | Add alert rule | `kismet --alert DEAUTHFLOOD,5/min` |
| `--no-alerts` | Disable alerts | `kismet --no-alerts` |

#### Advanced Flags

| Flag | Description | Example |
|------|-------------|---------|
| `--track-plugins` | Load tracking plugins | `kismet --track-plugins /usr/lib/kismet` |
| `--no-plugins` | Disable plugins | `kismet --no-plugins` |
| `--conf` | Override config key | `kismet --conf "key=value"` |
| `--no-dns` | Don't resolve hostnames | `kismet --no-dns` |
| `--ring-warn` | Log ring buffer warning | `kismet --ring-warn 10000` |
| `--ring-fail` | Log ring buffer failure | `kismet --ring-fail 100000` |

### Sample Command Output

```bash
sudo kismet -c wlan0 --no-gui --daemonize

# Console output:
[KISMET] Starting Kismet 2024.09.R1
[KISMET] Registering datasources...
[KISMET] Registered linuxwifi source: wlan0
[KISMET] HTTP server: http://127.0.0.1:2501/
[KISMET] HTTPS server: https://127.0.0.1:2502/
[KISMET] GPSD: Enabled on 127.0.0.1:2947
[KISMET] Logging to /var/log/kismet/Kismet-20240718-10-30-00-1/
[KISMET] Rest API available
[KISMET] Waiting for clients...
```

### Common First Commands

```bash
# Quick scan with default settings
sudo kismet -c wlan0

# Scan specific channels only
sudo kismet -c wlan0:channels=1,6,11

# Scan with fast hopping
sudo kismet -c wlan0:hop=10

# Scan and capture all packets
sudo kismet -c wlan0 -w capture_2024

# Scan with GPS
sudo kismet -c wlan0 --use-gpsd

# Remote access (bind to all interfaces)
sudo kismet --server-host 0.0.0.0 -c wlan0
```

---

## Chapter 4: Intermediate Usage

### Channel Hopping Configuration

Channel hopping is how Kismet scans across different WiFi frequencies. Configure it per-datasource:

```bash
# Lock to channel 6 (no hopping)
sudo kismet -c wlan0:channel=6

# Scan only non-overlapping 2.4GHz channels
sudo kismet -c wlan0:channels=1,6,11

# Fast hopping (10 channels/sec)
sudo kismet -c wlan0:hop=10

# Slow hopping (2 channels/sec, better capture)
sudo kismet -c wlan0:hop=2

# 5GHz only channels
sudo kismet -c wlan0:channels=36,40,44,48,149,153,157,161
```

### GPS Integration with GPSD

```bash
# Start GPSD first
sudo gpsd /dev/ttyUSB0 -F /var/run/gpsd.sock

# Test GPS connection
gpspipe -w -n 10

# Start Kismet with GPS
sudo kismet -c wlan0 --use-gpsd

# GPS data in logs shows coordinates for each network
# Networks can be mapped using kismet_db or the web UI map view
```

### Multi-Source Capture

```bash
# Multiple WiFi adapters on different bands
sudo kismet -c wlan0:name=2.4GHz -c wlan1:name=5GHz

# WiFi + Bluetooth
sudo kismet -c wlan0:name=WiFi -c hci0:name=Bluetooth

# WiFi + SDR
sudo kismet -c wlan0:name=WiFi -c rtl433:name=ISM_Band

# Channel-locked sources
sudo kismet -c wlan0:channel=6:name=Ch6_Monitor -c wlan1:channel=36:name=Ch36_5G
```

### Web UI Navigation

After starting Kismet, open `https://localhost:2501` in your browser:

| UI Section | Description |
|------------|-------------|
| **Devices** | List of all detected wireless devices (APs, clients, probes) |
| **APS** | Access points sorted by signal, channel, encryption |
| **Clients** | Detected client devices and their associations |
| **Probes** | SSID probe requests from clients |
| **Data** | Packet data and protocol breakdown |
| **Alerts** | Security alerts and notifications |
| **Map** | GPS-based network map (requires GPS) |
| **Messages** | Kismet server log messages |

### Bash Scripting

```bash
#!/bin/bash
# kismet_monitor.sh - Automated Kismet monitoring

INTERFACE="wlan0"
LOG_DIR="/var/log/kismet"
ALERT_THRESHOLD=10

# Start Kismet in daemon mode
echo "[*] Starting Kismet on $INTERFACE..."
sudo kismet -c "$INTERFACE" --daemonize \
    --log-prefix "$LOG_DIR" \
    --log-types kismet,pcap

# Wait for startup
sleep 5

echo "[*] Kismet running. Monitoring for alerts..."
echo "[*] Web UI: https://localhost:2501"

# Monitor for new alerts
while true; do
    ALERT_COUNT=$(kismet_db --count-alerts 2>/dev/null || echo "0")
    
    if [ "$ALERT_COUNT" -gt "$ALERT_THRESHOLD" ]; then
        echo "[!] ALERT: $ALERT_COUNT alerts detected!"
        # Send notification
        notify-send "Kismet Alert" "$ALERT_COUNT security alerts detected"
    fi
    
    sleep 60
done
```

### Python Scripting

```python
#!/usr/bin/env python3
"""
kismet_wrapper.py - Programmatic Kismet control via REST API
"""

import requests
import json
import time
import sys
import subprocess
from datetime import datetime

class KismetWrapper:
    """Wrapper for Kismet REST API"""
    
    def __init__(self, host="127.0.0.1", port=2501):
        self.base_url = f"https://{host}:{port}"
        self.session = requests.Session()
        self.session.verify = False  # Self-signed certs
    
    def get_devices(self):
        """Fetch all detected devices"""
        response = self.session.get(f"{self.base_url}/devices/views/all/devices.json")
        return response.json()
    
    def get_alerts(self):
        """Fetch all alerts"""
        response = self.session.get(f"{self.base_url}/alerts/all_alerts.json")
        return response.json()
    
    def get_networks(self):
        """Fetch all access points"""
        response = self.session.get(f"{self.base_url}/devices/views/accesspoints/devices.json")
        return response.json()
    
    def get_statistics(self):
        """Get server statistics"""
        response = self.session.get(f"{self.base_url}/system/status.json")
        return response.json()
    
    def export_pcap(self, filename):
        """Export capture to PCAP"""
        response = self.session.get(f"{self.base_url}/packets/bytestream/pcap")
        with open(filename, 'wb') as f:
            f.write(response.content)
        return filename
    
    def summary(self):
        """Print a summary of detected networks"""
        devices = self.get_devices()
        networks = self.get_networks()
        alerts = self.get_alerts()
        
        print(f"\n{'='*60}")
        print(f"  KISMET Network Summary - {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}")
        print(f"{'='*60}")
        print(f"  Total Devices:  {len(devices)}")
        print(f"  Access Points:  {len(networks)}")
        print(f"  Alerts:         {len(alerts)}")
        print(f"{'='*60}")
        
        # Top networks by signal
        sorted_networks = sorted(
            networks.values(),
            key=lambda x: x.get('kismet.device.base.signal', {}).get('kismet.device.base.signal.signal', -100),
            reverse=True
        )[:10]
        
        print(f"\n  Top 10 Networks by Signal:")
        print(f"  {'SSID':<25} {'BSSID':<18} {'Ch':<4} {'Signal':<8} {'Enc'}")
        print(f"  {'-'*25} {'-'*18} {'-'*4} {'-'*8} {'-'*10}")
        
        for net in sorted_networks:
            ssid = net.get('kismet.device.base.name', '<hidden>')
            bssid = net.get('kismet.device.base.macaddr', '??')
            channel = net.get('kismet.device.base.channel', '??')
            signal = net.get('kismet.device.base.signal', {}).get(
                'kismet.device.base.signal.signal', -100)
            enc = net.get('kismet.device.base.crypt', 'Unknown')
            print(f"  {ssid:<25} {bssid:<18} {channel:<4} {signal:<8} {enc}")


if __name__ == "__main__":
    # Start Kismet
    print("[*] Starting Kismet...")
    subprocess.Popen(
        ['sudo', 'kismet', '-c', 'wlan0', '--daemonize'],
        stdout=subprocess.PIPE,
        stderr=subprocess.PIPE
    )
    time.sleep(5)
    
    # Connect and query
    wrapper = KismetWrapper()
    try:
        wrapper.summary()
    except requests.exceptions.ConnectionError:
        print("[!] Could not connect to Kismet. Is it running?")
        sys.exit(1)
```

---

## Chapter 5: Advanced Usage

### Drone Detection (UAV/Drone Tracking)

Kismet can detect drone wireless signatures including WiFi-based controllers:

```bash
# Drone detection is built into Kismet's tracking system
# Drones appear as devices with specific characteristics:
# - Moving rapidly between access points
# - Using known drone controller MACs (DJI, Parrot, etc.)
# - Operating on specific frequencies

# Configure drone-specific alerts in kismet.conf
cat >> /etc/kismet/kismet.conf << 'EOF'
# Drone detection alerts
alert=DOT11_DEAUTHDOT11,1/min,2/sec
alert=DOT11_DISCON,5/min,1/sec
alert=PROBENORESP,10/min,2/sec
EOF

# View drone-suspect devices in web UI
# Look for devices with:
# - High mobility
# - Unknown manufacturer
# - Specific data rate patterns
```

### Bluetooth & Zigbee Detection

```bash
# Enable Bluetooth datasource
sudo kismet -c wlan0:name=WiFi -c hci0:name=Bluetooth

# Bluetooth devices appear alongside WiFi in the device list
# BLE beacons are tracked automatically
# Classic Bluetooth connections show device names when discoverable

# View Bluetooth devices
# In web UI: filter by PHY type = Bluetooth
```

### Server/Client Remote Capture

```bash
# On capture machine (running as remote sensor)
sudo kismet -c wlan0 --daemonize \
    --server-host 0.0.0.0 \
    --http-port 2501 \
    --tls-port 2502

# On monitoring machine (connecting to remote sensor)
# Add remote source in web UI or via config:
# source=remote+tls://sensor-ip:2502:name=RemoteSensor

# Or configure in kismet.conf
echo 'source=remote+tls://192.168.1.100:2502:name=RemoteOffice' >> /etc/kismet/kismet.conf
```

### kismet_db Log Analysis

```bash
# List all devices in a capture
kismet_db -r /var/log/kismet/Kismet-*/Kismet-*.kismetdb -l devices

# Export devices to CSV
kismet_db -r /var/log/kismet/Kismet-*/Kismet-*.kismetdb -l devices -o devices.csv

# Count devices by encryption type
kismet_db -r capture.kismetdb -l devices | \
    awk -F'|' '{print $5}' | sort | uniq -c | sort -nr

# Get alerts from capture
kismet_db -r capture.kismetdb -l alerts

# Export to PCAP for Wireshark
kismet_db -r capture.kismetdb -p output.pcap

# Query specific fields
kismet_db -r capture.kismetdb -l devices -o - | \
    grep "WPA2" | awk -F'|' '{print $1, $3, $4}'

# List data sources used in capture
kismet_db -r capture.kismetdb -l datasources

# Count packets per device
kismet_db -r capture.kismetdb -l packets | \
    awk -F'|' '{print $2}' | sort | uniq -c | sort -nr | head -20
```

### Custom Alert Rules

```bash
# Edit /etc/kismet/kismet.conf for alert configuration

# Alert types and thresholds
alert=DOT11_DEAUTHDOT11,5/min,2/sec     # Deauth floods
alert=DOT11_DISCON,10/min,2/sec         # Disconnection floods
alert=DOT11_EAPOL,5/min,1/sec           # EAPOL attacks
alert=PROBENORESP,20/min,2/sec          # Probe storms
alert=BSSTIMESTAMP,1/min,1/sec          # BSSID timestamp manipulation
alert=NETWRAP,1/min,1/sec               # Network wrap attacks
alert=DEVEL,10/min,5/sec                # Development alerts

# Rate limiting: alerts/per_interval,max_burst_per_sec
# This prevents alert flooding during attacks
```

### Plugin System

```bash
# List installed plugins
ls /usr/lib/kismet/

# Kismet plugins extend functionality:
# - Custom alert generators
# - Additional datasources
# - Protocol parsers
# - Logging modules

# Load custom plugin
sudo kismet -c wlan0 --track-plugins /usr/lib/kismet/custom_plugin.so
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: Enterprise Wireless Security Audit

**Objective**: Comprehensive wireless security assessment of a corporate campus.

```bash
#!/bin/bash
# enterprise_wireless_audit.sh

echo "=== Enterprise Wireless Security Audit ==="

# Phase 1: Deploy capture sensors
# Setup on multiple floors/locations
LOCATIONS=("lobby" "floor1" "floor2" "exec_floor" "parking_lot")

for loc in "${LOCATIONS[@]}"; do
    echo "[*] Starting sensor: $loc"
    sudo kismet -c "wlan0:name=$loc" \
        --daemonize \
        --log-prefix "/var/log/kismet/$loc" \
        --log-types kismet,pcap,netjson
    sleep 3
done

echo "[*] All sensors running for 24 hours..."
echo "[*] Web UI: https://localhost:2501"
echo "[*] Access individual sensors via remote+tls://"

# After 24 hours, analyze results
sleep 86400

# Phase 2: Consolidate results
echo "[*] Phase 2: Consolidating results..."
for loc in "${LOCATIONS[@]}"; do
    echo "--- $loc ---"
    kismet_db -r /var/log/kismet/$loc/*.kismetdb -l devices | \
        awk -F'|' '{print $3, $4, $5}' | sort | uniq -c | sort -nr
done

# Phase 3: Generate report
cat > enterprise_audit_report.md << 'REPORT'
# Enterprise Wireless Security Audit Report

## Executive Summary
- Deployed 5 Kismet sensors across campus
- Monitored for 24 hours
- [Insert findings here]

## Rogue AP Detection
- Unauthorized networks found: [count]
- Networks with weak encryption: [count]
- Hidden networks detected: [count]

## Client Security
- Clients connecting to open networks: [count]
- Clients probing for SSIDs: [list]

## Recommendations
1. Disable unused wireless interfaces
2. Enforce WPA3-Enterprise for corporate networks
3. Deploy wireless IDS on all floors
4. Implement MAC filtering for IoT devices
REPORT
```

### Scenario 2: Hospital Wireless Security Assessment (HIPAA)

```bash
#!/bin/bash
# hipaa_wireless_assessment.sh

echo "=== HIPAA Wireless Security Assessment ==="

# Healthcare-specific monitoring
# Focus on medical device networks and patient data flows

# Start Kismet with medical device tracking
sudo kismet -c wlan0:name=HospitalWiFi --daemonize \
    --log-prefix /var/log/kismet/hipaa \
    --log-types kismet,pcap,netjson

echo "[*] HIPAA Assessment running..."

# Monitor for specific concerns:
# 1. Medical device traffic on wireless
# 2. Patient data transmission
# 3. Unauthorized access points
# 4. Bluetooth medical devices

# After assessment period, generate HIPAA report
cat > hipaa_report.md << 'REPORT'
# HIPAA Wireless Security Assessment

## Scope
- All wireless networks in healthcare facility
- Medical device wireless connections
- Guest/patient WiFi networks

## Findings
### Critical
- [WPA2-Enterprise not enforced on clinical networks]
- [Medical devices on same VLAN as guest network]

### High
- [WPS enabled on multiple access points]
- [Bluetooth medical devices unencrypted]

### Medium
- [Guest network has access to internal resources]
- [SSID broadcasting clinical department names]

## HIPAA Compliance Status
- Encryption Required: WPA2-AES minimum
- Network Segmentation: Clinical vs Guest
- Access Controls: 802.1X recommended
- Audit Logging: Kismet captures for forensic review
REPORT
```

### Scenario 3: Campus Wireless Site Survey

```bash
#!/bin/bash
# campus_site_survey.sh

echo "=== Campus Wireless Site Survey ==="

# GPS-based site survey
# Requires GPS receiver connected

# Start Kismet with GPS
sudo kismet -c wlan0:name=SiteSurvey --daemonize \
    --use-gpsd \
    --log-prefix /var/log/kismet/survey \
    --log-types kismet,pcap,gps

echo "[*] Site survey running with GPS..."
echo "[*] Walk campus routes to map coverage"
echo "[*] View live map at https://localhost:2501/map"

# After survey, export GPS data
sleep 14400  # 4 hours of walking

# Generate coverage report
kismet_db -r /var/log/kismet/survey/*.kismetdb -l devices -o survey_devices.csv

echo "[*] Export KML for Google Earth:"
echo "    Import GPS data from logs into Google Earth"
echo "[*] Analyze signal strength patterns"
```

---

## Chapter 7: Detection & Defense

### Detecting Kismet Activity

Since Kismet operates passively, detection requires monitoring for:

```bash
# Method 1: Check for monitor mode interfaces
iwconfig 2>/dev/null | grep -i "mode:monitor"

# Method 2: Check for Kismet processes
ps aux | grep -E "kismet|kismet_cap"

# Method 3: Check for Kismet network connections
netstat -tulpn | grep -E "2501|2502"

# Method 4: Check for Kismet log files
find /var/log/kismet -name "*.kismetdb" 2>/dev/null

# Method 5: Wireless Intrusion Detection System (WIDS)
# Deploy dedicated WIDS to detect passive monitoring
```

### Defense Against Unauthorized Monitoring

```bash
# 1. Deploy WIDS sensors
# Use Kismet itself or commercial WIDS to detect rogue monitoring

# 2. Physical security
# Restrict physical access to prevent adapter insertion

# 3. Network segmentation
# Isolate sensitive networks

# 4. Disable unused ports
# Administratively disable unused switch ports

# 5. Monitor for USB wireless adapters
udevadm monitor --property | grep -i "usb"
```

### Incident Response

```bash
# If unauthorized monitoring is detected:

# 1. Identify the source
ps aux | grep kismet
iwconfig

# 2. Capture evidence
sudo tcpdump -i wlan0mon -w evidence.pcap &
sleep 300
sudo killall tcpdump

# 3. Document chain of custody
echo "Evidence collected: $(date)" >> incident_log.txt
sha256sum evidence.pcap >> incident_log.txt

# 4. Remove unauthorized tools
sudo killall kismet 2>/dev/null
sudo ip link set wlan0mon down 2>/dev/null
```

---

## Chapter 8: Troubleshooting & Resources

### Common Errors

| Error | Cause | Solution |
|-------|-------|----------|
| `No suitable datasources` | Interface not in monitor mode | `sudo ip link set wlan0 down && sudo iw wlan0 set type monitor && sudo ip link set wlan0 up` |
| `Could not connect to GPSD` | GPSD not running | `sudo gpsd /dev/ttyUSB0 -F /var/run/gpsd.sock` |
| `Permission denied` | Not running as root | `sudo kismet -c wlan0` |
| `SSL certificate error` | Missing/invalid TLS certs | Delete `/etc/kismet/kismet_httpd-ssl.*` and restart |
| `Web UI blank` | Browser caching | Clear cache or use incognito mode |
| `No packets captured` | Wrong interface | Verify with `iwconfig` and check datasource name |
| `High CPU usage` | Too many networks | Reduce channel hop rate: `-c wlan0:hop=2` |
| `Log directory error` | Permission issue | `sudo chown -R root:kismet /var/log/kismet` |

### Performance Optimization

```bash
# Reduce channel hop rate for better capture
sudo kismet -c wlan0:hop=2

# Limit logging to specific types
sudo kismet --log-types kismet,pcap

# Use faster storage for logs
sudo kismet --log-prefix /tmp/fast_ssd/logs

# Reduce memory usage
# Edit /etc/kismet/kismet.conf:
# ring_log_size=10000
# max_devices=5000
```

### Resources

| Resource | URL |
|----------|-----|
| Official Website | https://www.kismetwireless.net/ |
| GitHub Repository | https://github.com/kismetwireless/kismet |
| Documentation | https://www.kismetwireless.net/documentation/ |
| Kali Linux Tool Page | https://www.kali.org/tools/kismet/ |
| Mailing List | https://www.kismetwireless.net/lists/ |
| IRC | #kismet on OFTC |

---

## Resources

### Official Documentation

- **Kismet Website**: https://www.kismetwireless.net/
- **Kismet Wiki**: https://www.kismetwireless.net/documentation/
- **GitHub**: https://github.com/kismetwireless/kismet
- **REST API Docs**: https://www.kismetwireless.net/docs/apis/

### Related Tools

| Tool | Purpose | Relationship |
|------|---------|-------------|
| **airodump-ng** | Active WiFi scanning | Complementary — active vs passive |
| **Wireshark** | Packet analysis | Import Kismet PCAPs |
| **bettercap** | Network attack framework | Post-discovery exploitation |
| **hcxdumptool** | Handshake capture | Targeted capture after discovery |
| **gpsd** | GPS daemon | Kismet GPS integration |
| **Wireshark/tshark** | Deep packet analysis | Analyze Kismet exports |

### Books & Courses

- "Hacking Exposed: Wireless" by Johnny Cache
- "Metasploit: The Penetration Tester's Guide"
- SANS SEC575: Mobile Device Security
- Offensive Security Wireless Attacks (WiFu)

### Community

- **Kismet Forum**: https://www.kismetwireless.net/forum/
- **GitHub Issues**: https://github.com/kismetwireless/kismet/issues
- **Reddit**: r/netsec, r/Kalilinux
- **Kali Linux Forums**: https://forums.kali.org/

---

*Always ensure you have proper authorization before monitoring wireless networks. Unauthorized monitoring may violate wiretapping and computer crime laws.*
