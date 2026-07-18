---
tool_name: "sparrow-wifi"
display_name: "Sparrow WiFi"
mitre_tactic: "reconnaissance"
mitre_tactic_id: "TA0043"
mitre_subcategory: "wifi"
install_command: "sudo apt install sparrow-wifi"
version: "2024.09"
github_repo: "https://github.com/ghostop14/sparrow-wifi"
kali_tools_page: "https://www.kali.org/tools/sparrow-wifi/"
difficulty_level: "intermediate"
requires_root: true
gui_available: true
tags: ["wifi", "bluetooth", "analyzer", "spectrum", "gps", "sdr", "pyqt5", "visualization"]
related_tools: ["kismet", "wireshark", "gqrx", "bettercap"]
---

# Sparrow WiFi — WiFi and Bluetooth Wireless Network Analyzer

## What This Tool Does (in 30 seconds)

Sparrow WiFi is a next-generation graphical WiFi and Bluetooth analyzer built with Python 3 and PyQt5. It provides comprehensive wireless network analysis with GPS integration, SDR spectrum analysis, and real-time visualization — combining the functionality of multiple wireless tools into a single modern desktop interface.

---

## Chapter 1: Introduction & Overview

### What is Sparrow WiFi?

Sparrow WiFi is a modern, graphical wireless network analyzer that provides comprehensive analysis of WiFi and Bluetooth networks with advanced features like GPS integration, spectrum analysis, and real-time visualization. Developed by ghostop14, it represents a new generation of wireless analysis tools designed for security professionals and network administrators.

Unlike traditional command-line tools, Sparrow WiFi offers an intuitive PyQt5 graphical interface that makes wireless analysis accessible while maintaining the advanced features needed for professional security assessments.

### Development Timeline

| Year | Milestone |
|------|-----------|
| 2017 | Initial release as WiFi analyzer |
| 2018 | Bluetooth support added |
| 2019 | GPS integration introduced |
| 2020 | Spectrum analysis via SoapySDR |
| 2021 | SDR integration with waterfall display |
| 2022 | Enhanced real-time visualization |
| 2024 | Current version with improved performance |

### MITRE ATT&CK Mapping

| Tactic | Technique | ID | Description |
|--------|-----------|----|----|
| Reconnaissance | Network Service Discovery | T1046 | Active scanning of wireless networks |
| Reconnaissance | Wireless Network Discovery | T1590.004 | Identifying wireless infrastructure |
| Collection | Network Sniffing | T1040 | Capturing wireless traffic |

### Technical Architecture

```
┌─────────────────────────────────────────────────────┐
│                  User Interface                      │
│              (PyQt5 + Qt Charts)                     │
├─────────────────────────────────────────────────────┤
│                  Core Engine                         │
│              (Python 3 + asyncio)                    │
├───────────────┬──────────┬───────┬─────────────────┤
│  WiFi Scanner │ BT Scanner│  GPS  │  SDR Spectrum   │
│  (scapy/nl80211)│ (bluepy) │(gpsd) │  (SoapySDR)    │
└───────────────┴──────────┴───────┴─────────────────┘
```

| Component | Technology | Purpose |
|-----------|-----------|---------|
| GUI Layer | PyQt5 + Qt Charts | Cross-platform interface with real-time graphs |
| Core Engine | Python 3 + asyncio | Asynchronous packet processing |
| WiFi Scanner | scapy / nl80211 | 802.11 frame capture and analysis |
| Bluetooth Scanner | bluepy | BLE and Classic Bluetooth discovery |
| GPS Module | gpsd integration | Geographic network mapping |
| SDR Module | SoapySDR | Spectrum analysis with RTL-SDR/HackRF |

### Protocol Support

| Protocol | Standard | Max Speed | Detection |
|----------|----------|-----------|-----------|
| 802.11a | WiFi 1 | 54 Mbps | 5GHz |
| 802.11b | WiFi 2 | 11 Mbps | 2.4GHz |
| 802.11g | WiFi 3 | 54 Mbps | 2.4GHz |
| 802.11n | WiFi 4 | 600 Mbps | 2.4/5GHz |
| 802.11ac | WiFi 5 | 6.9 Gbps | 5GHz |
| 802.11ax | WiFi 6 | 9.6 Gbps | 2.4/5/6GHz |
| Bluetooth Classic | BR/EDR | 3 Mbps | Inquiry/scan |
| Bluetooth LE | BLE 4.0+ | 2 Mbps | Advertisement |

### Comparison with Similar Tools

| Feature | Sparrow WiFi | Kismet | Wireshark | Airodump-ng |
|---------|--------------|--------|-----------|-------------|
| Interface | PyQt5 Desktop GUI | Web-based | Qt-based | Terminal |
| GPS | Native integration | Plugin required | Limited | Limited |
| Spectrum Analysis | Built-in SDR | No | No | No |
| Bluetooth | Full support | Yes | Limited | No |
| Real-time Graphs | Yes | Limited | Yes | No |
| Ease of Use | High | Moderate | Low | Moderate |
| Command-line | Limited | Full | Full | Full |

---

## Chapter 2: Installation & Setup

### System Requirements

| Component | Minimum | Recommended |
|-----------|---------|-------------|
| CPU | Dual-core 1GHz | Quad-core 2GHz+ |
| RAM | 4 GB | 8 GB+ |
| Storage | 500 MB | 2 GB+ |
| Display | 1280x720 | 1920x1080 |
| WiFi Adapter | Monitor mode capable | Dual-band adapter |
| OS | Linux (Kali/Ubuntu) | Kali Linux 2024+ |

### Installation Methods

#### Method 1: APT (Recommended)

```bash
# Update package lists
sudo apt update

# Install Sparrow WiFi
sudo apt install sparrow-wifi

# Start Sparrow WiFi
sudo sparrow-wifi
```

#### Method 2: From Source

```bash
# Install dependencies
sudo apt install python3-pyqt5 python3-scapy python3-bluepy \
    python3-pyqtgraph python3-qtmultimedia gpsd gpsd-clients \
    libgps-dev libbluetooth-dev

# Clone and build
git clone https://github.com/ghostop14/sparrow-wifi.git
cd sparrow-wifi
pip3 install -r requirements.txt
chmod +x sparrow-wifi
sudo ./sparrow-wifi
```

#### Method 3: Docker

```bash
docker pull ghostop14/sparrow-wifi
docker run -it --rm --net=host --privileged \
    -v /dev:/dev \
    -v /tmp/.X11-unix:/tmp/.X11-unix \
    -e DISPLAY=$DISPLAY \
    ghostop14/sparrow-wifi
```

### Configuration

Sparrow WiFi stores configuration in `~/.sparrow-wifi/config.ini`:

```ini
[General]
theme=dark
language=en
auto_save=true

[WiFi]
interface=wlan0mon
channel_hop=true
channel_list=1,6,11

[Bluetooth]
enabled=true
scan_interval=5

[GPS]
enabled=false
device=/dev/ttyUSB0
baud_rate=9600

[Display]
refresh_rate=1000
max_networks=1000
```

### Post-Installation Setup

```bash
# Start monitor mode
sudo airmon-ng start wlan0

# Verify
iwconfig wlan0mon

# Launch Sparrow WiFi
sudo sparrow-wifi -i wlan0mon
```

---

## Chapter 3: Basic Usage

### Command-Line Flags Reference

#### Core Flags

| Flag | Description | Example |
|------|-------------|---------|
| `-i` | Specify wireless interface | `sparrow-wifi -i wlan0mon` |
| `--gps` | Enable GPS integration | `sparrow-wifi --gps` |
| `--bluetooth` | Enable Bluetooth scanning | `sparrow-wifi --bluetooth` |
| `--sdr` | Enable SDR spectrum analysis | `sparrow-wifi --sdr` |
| `--export` | Export data to file | `sparrow-wifi --export networks.csv` |
| `--channel` | Lock to specific channel | `sparrow-wifi --channel 6` |
| `--channels` | Scan specific channels | `sparrow-wifi --channels 1,6,11` |
| `--export-pcap` | Export to PCAP format | `sparrow-wifi --export-pcap capture.pcap` |
| `--export-kml` | Export to KML format | `sparrow-wifi --export-kml networks.kml` |
| `--export-geojson` | Export to GeoJSON | `sparrow-wifi --export-geojson networks.geojson` |
| `-h` / `--help` | Show help | `sparrow-wifi -h` |

#### WiFi Flags

| Flag | Description | Example |
|------|-------------|---------|
| `--log-signals` | Log signal strength data | `sparrow-wifi --log-signals` |
| `--signal-threshold` | Set signal filter (dBm) | `sparrow-wifi --signal-threshold -70` |
| `--filter-ssid` | Filter by SSID pattern | `sparrow-wifi --filter-ssid "Corp"` |
| `--filter-encryption` | Filter by encryption type | `sparrow-wifi --filter-encryption WPA2` |
| `--filter-signal` | Filter by signal strength | `sparrow-wifi --filter-signal -50` |
| `--max-networks` | Limit tracked networks | `sparrow-wifi --max-networks 500` |

#### Bluetooth Flags

| Flag | Description | Example |
|------|-------------|---------|
| `--ble` | Enable BLE scanning | `sparrow-wifi --bluetooth --ble` |
| `--ble-interval` | BLE scan interval (ms) | `sparrow-wifi --ble-interval 1000` |
| `--track` | Track BT device movement | `sparrow-wifi --bluetooth --track` |
| `--track-interval` | Tracking interval (sec) | `sparrow-wifi --track-interval 300` |
| `--export-ble` | Export BLE data | `sparrow-wifi --export-ble ble.csv` |
| `--export-tracking` | Export tracking data | `sparrow-wifi --export-tracking track.csv` |

#### GPS Flags

| Flag | Description | Example |
|------|-------------|---------|
| `--gps-device` | GPS serial device | `sparrow-wifi --gps-device /dev/ttyUSB0` |
| `--gps-baud` | GPS baud rate | `sparrow-wifi --gps-baud 9600` |
| `--gps-refresh` | GPS refresh rate (ms) | `sparrow-wifi --gps-refresh 1000` |
| `--export-gps` | Export GPS data | `sparrow-wifi --export-gps gps.csv` |

#### SDR Flags

| Flag | Description | Example |
|------|-------------|---------|
| `--sdr-device` | SDR device type | `sparrow-wifi --sdr-device rtlsdr` |
| `--freq-min` | Minimum frequency (MHz) | `sparrow-wifi --freq-min 2400` |
| `--freq-max` | Maximum frequency (MHz) | `sparrow-wifi --freq-max 2500` |
| `--sample-rate` | SDR sample rate | `sparrow-wifi --sample-rate 2.4e6` |
| `--fft-size` | FFT size for spectrum | `sparrow-wifi --fft-size 1024` |
| `--waterfall` | Enable waterfall display | `sparrow-wifi --waterfall` |
| `--average` | Spectrum averaging | `sparrow-wifi --average 10` |
| `--export-spectrum` | Export spectrum data | `sparrow-wifi --export-spectrum spec.csv` |

#### Performance Flags

| Flag | Description | Example |
|------|-------------|---------|
| `--refresh` | UI refresh rate (ms) | `sparrow-wifi --refresh 2000` |
| `--no-animations` | Disable animations | `sparrow-wifi --no-animations` |
| `--hw-accel` | Hardware acceleration | `sparrow-wifi --hw-accel` |
| `--max-memory` | Limit memory usage | `sparrow-wifi --max-memory 2G` |
| `--max-devices` | Limit tracked devices | `sparrow-wifi --max-devices 1000` |
| `--packet-buffer` | Set packet buffer size | `sparrow-wifi --packet-buffer 1000` |
| `--compress` | Compress output data | `sparrow-wifi --compress` |
| `--threads` | Thread count | `sparrow-wifi --threads 4` |
| `--debug` | Enable debug output | `sparrow-wifi --debug` |

### Common Launch Commands

```bash
# Basic WiFi analysis
sudo sparrow-wifi -i wlan0mon

# WiFi + Bluetooth
sudo sparrow-wifi -i wlan0mon --bluetooth

# WiFi + GPS
sudo sparrow-wifi -i wlan0mon --gps

# WiFi + SDR spectrum
sudo sparrow-wifi -i wlan0mon --sdr

# Full analysis
sudo sparrow-wifi -i wlan0mon --bluetooth --gps --sdr

# Background capture
nohup sudo sparrow-wifi -i wlan0mon --export capture.csv &

# Timed capture
timeout 300 sudo sparrow-wifi -i wlan0mon --export data.csv
```

### Sample Output

```
Sparrow WiFi 2024.09

Starting Sparrow WiFi...
Using interface: wlan0mon
GPS: Disabled
Bluetooth: Enabled
SDR: Disabled

Loading GUI...
Detected networks: 23
Detected devices: 47
BLE devices: 12
```

---

## Chapter 4: Intermediate Usage

### Channel Hopping Control

```bash
# Scan specific channels only
sudo sparrow-wifi -i wlan0mon --channels 1,6,11

# Full 2.4GHz scan
sudo sparrow-wifi -i wlan0mon --channels 1,2,3,4,5,6,7,8,9,10,11

# 5GHz channels
sudo sparrow-wifi -i wlan0mon --channels 36,40,44,48,149,153,157,161

# Lock to single channel
sudo sparrow-wifi -i wlan0mon --channel 6
```

### Bash Scripting

```bash
#!/bin/bash
# sparrow_automated.sh - Automated Sparrow WiFi monitoring

INTERFACE="wlan0mon"
OUTPUT_DIR="$HOME/sparrow-captures/$(date +%Y%m%d_%H%M%S)"
DURATION=3600  # 1 hour

mkdir -p "$OUTPUT_DIR"

echo "[*] Starting Sparrow WiFi capture..."
echo "[*] Interface: $INTERFACE"
echo "[*] Duration: $DURATION seconds"
echo "[*] Output: $OUTPUT_DIR"

# Start Sparrow WiFi with export
sudo sparrow-wifi -i "$INTERFACE" \
    --export "$OUTPUT_DIR/networks.csv" \
    --export-pcap "$OUTPUT_DIR/capture.pcap" &

SPARROW_PID=$!

# Wait for capture duration
sleep "$DURATION"

# Stop Sparrow WiFi
sudo kill $SPARROW_PID 2>/dev/null

echo "[*] Capture complete"
echo "[*] Networks: $OUTPUT_DIR/networks.csv"
echo "[*] Packets: $OUTPUT_DIR/capture.pcap"

# Analyze results
echo "[*] Network count: $(wc -l < "$OUTPUT_DIR/networks.csv")"
echo "[*] Top channels:"
awk -F',' '{print $4}' "$OUTPUT_DIR/networks.csv" | sort | uniq -c | sort -nr
```

### Python Scripting

```python
#!/usr/bin/env python3
"""
sparrow_wrapper.py - Programmatic Sparrow WiFi control
"""

import subprocess
import json
import csv
import time
import signal
import sys
from pathlib import Path
from datetime import datetime
from typing import Optional, List, Dict


class SparrowWiFi:
    """Wrapper for Sparrow WiFi CLI operations"""
    
    def __init__(self, interface: str, output_dir: str = "~/.sparrow-wifi/logs"):
        self.interface = interface
        self.output_dir = Path(output_dir).expanduser()
        self.output_dir.mkdir(parents=True, exist_ok=True)
        self.process: Optional[subprocess.Popen] = None
        
        if not self._check_interface():
            raise ValueError(f"Interface {interface} not found or not in monitor mode")
    
    def _check_interface(self) -> bool:
        """Verify interface exists and is in monitor mode"""
        try:
            result = subprocess.run(
                ['iwconfig', self.interface],
                capture_output=True, text=True
            )
            return result.returncode == 0 and 'Mode:Monitor' in result.stdout
        except FileNotFoundError:
            return False
    
    def start_capture(self, duration: int = 300, 
                      capture_file: Optional[str] = None,
                      enable_gps: bool = False,
                      enable_bluetooth: bool = False) -> str:
        """Start Sparrow WiFi capture"""
        if capture_file is None:
            timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
            capture_file = str(self.output_dir / f"sparrow_{timestamp}.csv")
        
        cmd = ['sudo', 'sparrow-wifi', '-i', self.interface, '--export', capture_file]
        
        if enable_gps:
            cmd.append('--gps')
        if enable_bluetooth:
            cmd.append('--bluetooth')
        
        print(f"[*] Starting capture: {capture_file}")
        self.process = subprocess.Popen(
            cmd, stdout=subprocess.PIPE, stderr=subprocess.PIPE
        )
        
        time.sleep(duration)
        self.stop_capture()
        return capture_file
    
    def stop_capture(self):
        """Stop Sparrow WiFi capture"""
        if self.process:
            self.process.send_signal(signal.SIGTERM)
            try:
                self.process.wait(timeout=10)
            except subprocess.TimeoutExpired:
                self.process.kill()
            self.process = None
    
    def analyze_csv(self, csv_file: str) -> Dict:
        """Analyze exported CSV data"""
        networks = []
        with open(csv_file, 'r') as f:
            reader = csv.DictReader(f)
            for row in reader:
                networks.append(row)
        
        analysis = {
            'total_networks': len(networks),
            'encryption_types': {},
            'channels': {},
            'avg_signal': 0,
        }
        
        for net in networks:
            enc = net.get('encryption', 'Unknown')
            analysis['encryption_types'][enc] = analysis['encryption_types'].get(enc, 0) + 1
            
            ch = net.get('channel', 'Unknown')
            analysis['channels'][ch] = analysis['channels'].get(ch, 0) + 1
        
        return analysis
    
    def export_to_kml(self, csv_file: str, kml_file: str) -> bool:
        """Convert CSV data to KML for Google Earth"""
        # KML export handled by Sparrow WiFi CLI
        cmd = ['sudo', 'sparrow-wifi', '-i', self.interface, '--export-kml', kml_file]
        try:
            subprocess.run(cmd, check=True, capture_output=True)
            return True
        except subprocess.CalledProcessError:
            return False


# Main usage example
if __name__ == "__main__":
    sparrow = SparrowWiFi("wlan0mon")
    
    # 5-minute capture with GPS
    capture = sparrow.start_capture(
        duration=300,
        enable_gps=True,
        enable_bluetooth=True
    )
    
    # Analyze results
    analysis = sparrow.analyze_csv(capture)
    print(json.dumps(analysis, indent=2))
```

---

## Chapter 5: Advanced Usage

### Custom Themes

```bash
# Create custom dark theme
mkdir -p ~/.sparrow-wifi/themes
cat > ~/.sparrow-wifi/themes/custom_dark.ini << 'EOF'
[Colors]
background=#0d1117
foreground=#c9d1d9
accent=#58a6ff
border=#30363d

[Fonts]
family=Monospace
size=10
bold=false
EOF
```

### SIEM Integration

```bash
# Splunk integration - forward Sparrow WiFi logs
tail -f ~/.sparrow-wifi/logs/*.csv | while read line; do
    curl -s -k "https://splunk.example.com:8088/services/collector/event" \
        -H "Authorization: Splunk YOUR_TOKEN" \
        -d "{\"event\": \"$line\"}"
done
```

### Grafana Dashboard

Create dashboards by piping Sparrow WiFi CSV exports into Prometheus or Elasticsearch for visualization.

### Performance Optimization

```bash
# Reduce refresh rate for lower CPU
sudo sparrow-wifi -i wlan0mon --refresh 2000

# Limit tracked networks
sudo sparrow-wifi -i wlan0mon --max-networks 500

# CPU pinning for dedicated capture
taskset -c 0-3 sudo sparrow-wifi -i wlan0mon

# Disable animations for performance
sudo sparrow-wifi -i wlan0mon --no-animations
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: Enterprise Wireless Assessment

```bash
#!/bin/bash
# enterprise_sparrow_assessment.sh

echo "=== Enterprise Wireless Assessment ==="

# Phase 1: WiFi network discovery
sudo sparrow-wifi -i wlan0mon \
    --export enterprise_wifi.csv \
    --channels 1,6,11,36,40,44,48,149,153,157,161 &

SPARROW_PID=$!
sleep 1800  # 30 minutes
sudo kill $SPARROW_PID

# Phase 2: Bluetooth discovery
sudo sparrow-wifi --bluetooth --ble \
    --export-bt enterprise_bt.csv &

BT_PID=$!
sleep 600  # 10 minutes
sudo kill $BT_PID

# Phase 3: GPS mapping
sudo sparrow-wifi -i wlan0mon --gps \
    --export-gps enterprise_gps.csv \
    --export-kml enterprise_map.kml &

GPS_PID=$!
sleep 3600  # 1 hour of walking survey
sudo kill $GPS_PID

echo "[*] Assessment complete. Analyzing results..."

# Analyze WiFi
echo "WiFi Networks: $(wc -l < enterprise_wifi.csv)"
echo "Encryption distribution:"
awk -F',' '{print $5}' enterprise_wifi.csv | sort | uniq -c | sort -nr

echo "Bluetooth devices: $(wc -l < enterprise_bt.csv)"
echo "Networks mapped to KML: enterprise_map.kml"
```

### Scenario 2: Hospital HIPAA Assessment

```bash
#!/bin/bash
# hipaa_wireless_assessment.sh

echo "=== HIPAA Wireless Assessment ==="

# Healthcare-specific WiFi analysis
# Focus: Medical device networks, PHI transmission, segmentation

# Start comprehensive monitoring
sudo sparrow-wifi -i wlan0mon --bluetooth --gps \
    --export hipaa_capture.csv \
    --export-pcap hipaa_traffic.pcap \
    --export-kml hipaa_map.kml &

PID=$!

# Monitor for 4 hours
echo "[*] Monitoring for 4 hours..."
sleep 14400

sudo kill $PID

# Generate HIPAA report
cat > hipaa_report.md << 'REPORT'
# HIPAA Wireless Security Assessment Report

## Assessment Scope
- All wireless networks in healthcare facility
- Medical device wireless connections
- Guest/patient WiFi networks
- Bluetooth medical devices

## Findings
### Critical
- Open/unsencrypted networks detected on clinical floor
- Medical devices sharing VLAN with guest traffic

### Recommendations
1. Enforce WPA3-Enterprise for all clinical networks
2. Segment medical device traffic via dedicated VLAN
3. Disable WPS on all access points
4. Implement wireless IDS monitoring
REPORT
```

### Scenario 3: Campus Site Survey

```bash
#!/bin/bash
# campus_site_survey.sh

echo "=== Campus Site Survey ==="

# GPS-based wireless coverage survey
# Walk campus routes to map signal strength

sudo sparrow-wifi -i wlan0mon --gps \
    --export campus_survey.csv \
    --export-gps campus_gps.csv \
    --export-kml campus_map.kml \
    --log-signals &

PID=$!
echo "[*] Site survey running. Walk campus routes."
echo "[*] View live map in Sparrow WiFi Map tab"
echo "[*] Press Ctrl+C when done"

# Wait for user interrupt
trap "sudo kill $PID 2>/dev/null; exit" INT TERM
wait $PID

echo "[*] Survey complete"
echo "[*] Networks mapped: campus_map.kml"
echo "[*] Signal data: campus_survey.csv"
```

---

## Chapter 7: Detection & Defense

### Detecting Sparrow WiFi Activity

Sparrow WiFi is passive but its presence can be detected:

```bash
# Check for monitor mode interfaces
iwconfig 2>/dev/null | grep -i "mode:monitor"

# Check for Sparrow WiFi processes
ps aux | grep sparrow-wifi

# Check for capture log files
find ~/.sparrow-wifi -name "*.csv" 2>/dev/null

# Network-based detection: look for unusual monitor mode
cat > /usr/local/bin/detect_sparrow.sh << 'EOF'
#!/bin/bash
MONITOR_COUNT=$(iwconfig 2>/dev/null | grep -c "Mode:Monitor")
if [ "$MONITOR_COUNT" -gt 0 ]; then
    echo "[$(date)] ALERT: Monitor mode interface detected"
fi
SPARROW_PROCS=$(pgrep -c sparrow-wifi 2>/dev/null)
if [ "$SPARROW_PROCS" -gt 0 ]; then
    echo "[$(date)] ALERT: Sparrow WiFi process detected"
fi
EOF
chmod +x /usr/local/bin/detect_sparrow.sh
```

### Defense Strategies

```bash
# 1. Deploy Wireless Intrusion Detection (WIDS)
# Use Kismet or commercial WIDS for continuous monitoring

# 2. Network Segmentation
sudo ip link add link eth0 name eth0.100 type vlan id 100  # Corporate
sudo ip link add link eth0 name eth0.200 type vlan id 200  # Guest
sudo ip link add link eth0 name eth0.300 type vlan id 300  # IoT

# 3. 802.1X Authentication
sudo apt install freeradius
# Configure RADIUS for WPA2-Enterprise

# 4. Use WPA3 for strongest protection
cat > /etc/hostapd/hostapd.conf << 'EOF'
interface=wlan0
driver=nl80211
ssid=SecureNetwork
wpa=3
wpa_key_mgmt=SAE
wpa_pairwise=CCMP
ieee80211w=2
EOF
```

---

## Chapter 8: Troubleshooting & Resources

### Common Errors

| Error | Cause | Solution |
|-------|-------|----------|
| `Cannot connect to display` | Missing X11 | `export DISPLAY=:0 && sudo -E sparrow-wifi` |
| `Qt plugin error` | Missing Qt libs | `sudo apt install libqt5gui5` |
| `No wireless interface found` | Interface missing | `sudo airmon-ng start wlan0` |
| `Interface not in monitor mode` | Wrong mode | `sudo airmon-ng check kill && sudo airmon-ng start wlan0` |
| `GPS device not found` | GPSD not running | `sudo gpsd /dev/ttyUSB0 -F /var/run/gpsd.sock` |
| `Bluetooth adapter not found` | BT not enabled | `sudo hciconfig hci0 up` |
| `High CPU usage` | Too many networks | `sparrow-wifi --refresh 2000 --max-networks 500` |

### Performance Optimization

```bash
# Monitor resources
top -p $(pgrep sparrow-wifi)

# Reduce CPU usage
sudo sparrow-wifi -i wlan0mon --refresh 2000 --no-animations

# Increase swap for memory issues
sudo fallocate -l 2G /swapfile && sudo chmod 600 /swapfile
sudo mkswap /swapfile && sudo swapon /swapfile
```

### Resources

| Resource | URL |
|----------|-----|
| GitHub | https://github.com/ghostop14/sparrow-wifi |
| Documentation | https://github.com/ghostop14/sparrow-wifi/wiki |
| Kali Tools | https://www.kali.org/tools/sparrow-wifi/ |
| Related: Kismet | https://www.kismetwireless.net/ |
| Related: Wireshark | https://www.wireshark.org/ |

---

*Always ensure you have proper authorization before analyzing wireless networks. Unauthorized monitoring may violate wiretapping laws.*
