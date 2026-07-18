---
tool_name: "wash"
display_name: "Wash"
mitre_tactic: "reconnaissance"
mitre_tactic_id: "TA0043"
mitre_subcategory: "wifi"
install_command: "sudo apt install reaver"
version: "1.6.6"
github_repo: "https://github.com/t6x/reaver-wps-fork"
kali_tools_page: "https://www.kali.org/tools/reaver/"
difficulty_level: "beginner"
requires_root: true
gui_available: false
tags: ["wifi", "wps", "scanning", "reaver", "reconnaissance", "wireless"]
related_tools: ["reaver", "bully", "pixiewps", "airodump-ng", "wifite"]
---

# Wash — WiFi Protected Setup Scanning Tool

## Chapter 1: Introduction & Overview

### What is Wash?

Wash is a WiFi Protected Setup (WPS) scanning tool that detects WPS-enabled wireless networks. Part of the Reaver suite, it passively monitors wireless traffic and identifies WPS information elements in beacon frames and probe responses. When it detects a WPS-enabled network, it displays BSSID, channel, signal strength, WPS version, and lock status.

### Historical Context

- **2006**: WPS standard introduced by Wi-Fi Alliance
- **2008**: First WPS vulnerabilities discovered
- **2011**: Reaver tool released with WPS attack capabilities
- **2012**: Wash developed as part of Reaver suite by Craig Heffner
- **2014**: WPS 2.0 released with improvements
- **2016**: Pixie Dust attack discovered
- **2018+**: Continued community development under T6x fork

### Core Capabilities

1. **WPS Detection** — Identifies WPS-enabled networks in real-time
2. **WPS Version Detection** — Distinguishes WPS 1.0 from 2.0, identifies lock feature
3. **Network Information** — BSSID, channel, RSSI, vendor, ESSID
4. **Output Options** — Terminal, file, CSV, daemon mode
5. **Passive Operation** — Doesn't transmit, only listens
6. **Low Resource Usage** — Minimal CPU and memory footprint

### MITRE ATT&CK Mapping

| Tactic | Technique | ID |
|--------|-----------|-----|
| Reconnaissance | Network Service Discovery | T1046 |
| Reconnaissance | Wireless Network Discovery | T1590.004 |
| Initial Access | Exploit Public-Facing Application | T1190 |

### Use Cases

- **Security Assessments**: Identify WPS-enabled networks for vulnerability evaluation
- **Penetration Testing**: Reconnaissance for WPS attack vectors
- **Network Administration**: Verify WPS configurations, compliance checking
- **Research**: Study WPS implementation vulnerabilities

### WPS Protocol Details

WPS uses specific information elements in WiFi frames: WPS Version, WPS State, WPS Methods, WPS PIN, and WPS UUID. Configuration methods include PIN (8-digit), Push Button, NFC, USB, Ethernet, and Label.

### WPS Security Concerns

1. **PIN Brute Force** — 8-digit PIN can be cracked in hours
2. **Default PINs** — Many devices use predictable manufacturer PINs
3. **Pixie Dust** — Exploits weak random number generation in WPS 1.0
4. **Online Attacks** — Multiple authentication attempts against locked APs
5. **Offline Attacks** — Capture and analyze WPS handshakes

### Architecture

```
┌─────────────────────────────────────────┐
│              Wash Scanner               │
├─────────────────────────────────────────┤
│  WiFi Capture  │  WPS Parser  │ Output  │
│  (libpcap)     │  (Detection) │(Format) │
└─────────────────────────────────────────┘
```

**Key Components:**
1. WiFi Capture Module — Uses libpcap, operates in monitor mode, captures beacons and probe responses
2. WPS Parser — Identifies WPS information elements, extracts version and configuration
3. Output Formatter — Handles terminal, file, CSV, and daemon output modes

### Comparison with Similar Tools

| Aspect | Wash | Reaver | Bully | Wifite |
|--------|------|--------|-------|--------|
| Primary Use | Scanning | Attack | Attack | All-in-one |
| WPS Detection | Yes | No | No | Integrated |
| Attack Capability | No | Yes | Yes | Multiple |
| Speed | Fast | Slow | Moderate | High |
| Risk Level | Low | High | High | Variable |
| Ease of Use | High | Moderate | Moderate | High |

## Chapter 2: Installation & Setup

### System Requirements

- **OS**: Linux (Kali Linux recommended)
- **Hardware**: Wireless adapter supporting monitor mode
- **Root Access**: Required for wireless capture
- **Dependencies**: libpcap, libsqlite3, libnl
- **Storage**: ~50 MB

### Method 1: Package Manager (Recommended)

```bash
# Kali Linux / Debian
sudo apt update
sudo apt install reaver
wash --help

# Ubuntu / Mint
echo "deb http://http.kali.org/kali kali-rolling main non-free contrib" | sudo tee /etc/apt/sources.list.d/kali.list
wget -q -O - https://archive.kali.org/archive-key.asc | sudo apt-key add -
sudo apt update && sudo apt install reaver

# Arch Linux
sudo pacman -S reaver
# Or from AUR
yay -S reaver-wps-fork
```

### Method 2: Docker Installation

```bash
docker pull securitytube/reaver
docker run -it --rm --net=host --privileged \
  -v /dev:/dev \
  securitytube/reaver wash -i wlan0mon
```

### Method 3: Build from Source

```bash
# Install build dependencies
sudo apt install build-essential libpcap-dev libsqlite3-dev \
  libnl-3-dev libnl-genl-3-dev git

# Clone and build
git clone https://github.com/t6x/reaver-wps-fork.git
cd reaver-wps-fork/src
./configure
make
sudo make install
wash --help
```

### Post-Installation Setup

```bash
# Verify installation
wash --version
ldd $(which wash)

# Start monitor mode
sudo airmon-ng start wlan0
iwconfig wlan0mon

# Initialize log directory
sudo mkdir -p /var/log/reaver
```

### Configuration

```bash
# Basic configuration
cat > /etc/reaver/wash.conf << 'EOF'
interface=wlan0mon
channel=0
timeout=60
output_file=/var/log/wash/scan.log
EOF
```

### Systemd Service

```bash
cat > /etc/systemd/system/wash-scan.service << 'EOF'
[Unit]
Description=Wash WPS Scanner
After=network.target

[Service]
Type=simple
User=root
ExecStart=/usr/bin/wash -i wlan0mon -o /var/log/wash/scan.log
Restart=on-failure
RestartSec=10

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable wash-scan
sudo systemctl start wash-scan
```

### Uninstallation

```bash
sudo apt remove reaver
sudo apt purge reaver
sudo apt autoremove
sudo rm -rf /etc/reaver /var/log/reaver
```

## Chapter 3: Basic Usage

### Command Line Syntax

```bash
wash [options] [arguments]
```

### Common Options

| Option | Description | Example |
|--------|-------------|---------|
| `-i` | Wireless interface | `-i wlan0mon` |
| `-c` | Channel to scan | `-c 6` |
| `-o` | Timeout in seconds | `-o 60` |
| `-f` | Output format | `-f csv` |
| `-s` | Scan mode | `-s` |
| `-v` | Verbose output | `-v` |
| `-q` | Quiet mode | `-q` |

### Basic WPS Scan

```bash
# Start basic scan
sudo wash -i wlan0mon

# Scan with timeout
sudo wash -i wlan0mon -o 60

# Scan specific channel
sudo wash -i wlan0mon -c 6
```

### Scan with Output

```bash
# Save to text file
sudo wash -i wlan0mon > /tmp/wps_scan.txt

# Save to CSV
sudo wash -i wlan0mon -f csv > wps_networks.csv

# Save with timestamp
sudo wash -i wlan0mon > "scan_$(date +%Y%m%d_%H%M%S).txt"
```

### Continuous Scanning

```bash
# Continuous scan (Ctrl+C to stop)
sudo wash -i wlan0mon -s

# Background scan
nohup sudo wash -i wlan0mon -s > /var/log/wash_continuous.log &
```

### Output Interpretation

```
BSSID               Channel  RSSI  WPS Version  Locked  ESSID
00:11:22:33:44:55    6       -45   2.0          No      CorporateWiFi
AA:BB:CC:DD:EE:FF    11      -62   1.0          Yes     GuestNetwork
```

**Signal Strength Guide:**
- -30 to -50 dBm: Excellent
- -50 to -67 dBm: Good
- -67 to -80 dBm: Fair
- -80 to -90 dBm: Poor
- Below -90 dBm: Very poor

## Chapter 4: Intermediate Usage

### Filtering and Selection

```bash
# Filter by WPS version
sudo wash -i wlan0mon | grep "1.0"
sudo wash -i wlan0mon | grep "2.0"

# Filter by signal strength
sudo wash -i wlan0mon | awk '$3 > -60'

# Count by channel
sudo wash -i wlan0mon | awk '{print $2}' | sort | uniq -c
```

### Data Analysis

```bash
# Count total networks
sudo wash -i wlan0mon | wc -l

# Average signal strength
sudo wash -i wlan0mon | awk '{sum+=$3; count++} END {print "Average:", sum/count}'

# Export to CSV for analysis
sudo wash -i wlan0mon -f csv > wps_networks.csv
awk -F',' '{print $1, $3, $5}' wps_networks.csv
```

### Integration with Reaver

```bash
# Scan and attack workflow
sudo wash -i wlan0mon > wps_networks.txt
TARGET=$(awk 'NR>1 {print $1}' wps_networks.txt | head -1)
sudo reaver -i wlan0mon -b $TARGET -v
```

### Integration with Bully

```bash
sudo wash -i wlan0mon > wps_networks.txt
TARGET=$(awk 'NR>1 {print $1}' wps_networks.txt | head -1)
CHANNEL=$(awk 'NR>1 {print $2}' wps_networks.txt | head -1)
sudo bully -b $TARGET -c $CHANNEL -w -v 3
```

### Automated Scanning Script

```bash
#!/bin/bash
# automated_wps_scan.sh

set -euo pipefail

INTERFACE="wlan0mon"
OUTPUT_DIR="/var/log/wps_scans"
DURATION=60

mkdir -p "$OUTPUT_DIR"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
OUTPUT_FILE="$OUTPUT_DIR/scan_$TIMESTAMP.txt"

echo "Starting WPS scan..."
sudo wash -i "$INTERFACE" -o "$DURATION" > "$OUTPUT_FILE"

NETWORKS=$(wc -l < "$OUTPUT_FILE")
echo "Scan complete: $NETWORKS networks found"

cat > "$OUTPUT_DIR/report_$TIMESTAMP.md" << EOF
# WPS Scan Report

## Scan Details
- Date: $(date)
- Interface: $INTERFACE
- Duration: $DURATION seconds
- Networks found: $NETWORKS

## Results
$(cat "$OUTPUT_FILE")
EOF

echo "Report saved to: $OUTPUT_DIR/report_$TIMESTAMP.md"
```

## Chapter 5: Advanced Usage

### Python Integration

```python
#!/usr/bin/env python3
"""wash_advanced.py - Advanced Wash wrapper"""

import subprocess
import os
import json
import logging
from datetime import datetime
from pathlib import Path
from typing import Optional, Dict, List
import argparse
import time

logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(levelname)s - %(message)s',
    handlers=[
        logging.FileHandler('wash_advanced.log'),
        logging.StreamHandler()
    ]
)
logger = logging.getLogger(__name__)

class WashWrapper:
    def __init__(self, interface: str, output_dir: str = "/var/log/wash"):
        self.interface = interface
        self.output_dir = Path(output_dir)
        self.output_dir.mkdir(parents=True, exist_ok=True)

    def start_scan(self, duration: int = 60, output_file: Optional[str] = None) -> str:
        if output_file is None:
            timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
            output_file = str(self.output_dir / f"wash_{timestamp}.txt")

        cmd = ['sudo', 'wash', '-i', self.interface, '-o', str(duration), '-v']
        logger.info(f"Starting Wash scan: {output_file}")

        with open(output_file, 'w') as f:
            process = subprocess.Popen(cmd, stdout=f, stderr=subprocess.PIPE)
        process.wait()
        return output_file

    def analyze_scan(self, scan_file: str) -> Dict:
        analysis = {'total_networks': 0, 'wps_versions': {}, 'channels': {}, 'signal_strengths': []}

        with open(scan_file, 'r') as f:
            for line in f.readlines()[1:]:
                parts = line.split()
                if len(parts) >= 6:
                    analysis['total_networks'] += 1
                    version = parts[3]
                    analysis['wps_versions'][version] = analysis['wps_versions'].get(version, 0) + 1
                    channel = parts[1]
                    analysis['channels'][channel] = analysis['channels'].get(channel, 0) + 1
                    analysis['signal_strengths'].append(int(parts[2]))

        if analysis['signal_strengths']:
            analysis['avg_signal'] = sum(analysis['signal_strengths']) / len(analysis['signal_strengths'])
        return analysis

    def get_strong_signals(self, scan_file: str, threshold: int = -60) -> List[Dict]:
        networks = []
        with open(scan_file, 'r') as f:
            for line in f.readlines()[1:]:
                parts = line.split()
                if len(parts) >= 6 and int(parts[2]) > threshold:
                    networks.append({
                        'bssid': parts[0], 'channel': parts[1],
                        'signal': int(parts[2]), 'wps_version': parts[3],
                        'locked': parts[4], 'ssid': parts[5] if len(parts) > 5 else ''
                    })
        return networks

if __name__ == '__main__':
    parser = argparse.ArgumentParser(description='Advanced Wash wrapper')
    parser.add_argument('interface', help='Wireless interface')
    parser.add_argument('--duration', type=int, default=60)
    parser.add_argument('--analyze', action='store_true')
    parser.add_argument('--threshold', type=int, default=-60)
    args = parser.parse_args()

    wrapper = WashWrapper(args.interface)
    scan_file = wrapper.start_scan(args.duration)
    print(f"Scan saved to: {scan_file}")

    if args.analyze:
        analysis = wrapper.analyze_scan(scan_file)
        print(json.dumps(analysis, indent=2))

    networks = wrapper.get_strong_signals(scan_file, args.threshold)
    print(f"\nNetworks with signal > {args.threshold} dBm: {len(networks)}")
    for net in networks:
        print(f"  {net['bssid']} - {net['signal']} dBm - {net['ssid']}")
```

### Advanced Automation Script

```bash
#!/bin/bash
# wash_advanced.sh - Advanced Wash automation

set -euo pipefail

WASH_LOG="/var/log/wash"
INTERFACE="wlan0mon"

log() { echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a "$WASH_LOG/wash_$(date +%Y%m%d).log"; }

advanced_scan() {
    local duration=$1 output_file=$2
    log "Starting advanced WPS scan for $duration seconds..."
    sudo wash -i "$INTERFACE" -o "$duration" -v > "$output_file" 2>&1

    awk '{print $4}' "$output_file" | sort | uniq -c > "${output_file}.versions"
    awk '{print $2}' "$output_file" | sort | uniq -c > "${output_file}.channels"
    awk '{print $3}' "$output_file" | sort -n > "${output_file}.signals"
    log "Analysis complete"
}

generate_report() {
    local scan_dir=$1 report_file=$2
    {
        echo "=== Wash Comprehensive Report ==="
        echo "Date: $(date)"
        echo "Interface: $INTERFACE"
        echo ""
        echo "=== WPS Version Distribution ==="
        cat "$scan_dir"/*.versions 2>/dev/null | sort -k2 -nr
        echo ""
        echo "=== Channel Distribution ==="
        cat "$scan_dir"/*.channels 2>/dev/null | sort -k2 -nr
    } > "$report_file"
    log "Report generated: $report_file"
}

DURATION=${1:-3600}
OUTPUT_DIR="$WASH_LOG/$(date +%Y%m%d_%H%M%S)"
mkdir -p "$OUTPUT_DIR"
advanced_scan "$DURATION" "$OUTPUT_DIR/scan.txt"
generate_report "$OUTPUT_DIR" "$OUTPUT_DIR/report.txt"
log "Advanced Wash automation completed"
```

### SIEM Integration

```bash
# Splunk integration
tail -f /var/log/wash/*.txt | while read line; do
    curl -s -k "https://splunk.example.com:8088/services/collector/event" \
        -H "Authorization: Splunk YOUR_TOKEN" \
        -d "{\"event\": \"$line\"}"
done

# Elasticsearch integration
tail -f /var/log/wash/*.txt | while read line; do
    curl -s -X POST "http://localhost:9200/wash-scans/_doc" \
        -H "Content-Type: application/json" \
        -d "$line"
done
```

### Signal Strength Analysis

```bash
# Extract and analyze signal strengths
awk '{print $3}' wash_scan.txt | sort -n > signals.txt

echo "=== Signal Strength Statistics ==="
echo "Total networks: $(wc -l < signals.txt)"
echo "Strongest: $(tail -1 signals.txt) dBm"
echo "Weakest: $(head -1 signals.txt) dBm"
echo "Average: $(awk '{sum+=$1; count++} END {print sum/count}' signals.txt) dBm"
```

## Chapter 6: Real-World Scenarios

### Enterprise WPS Assessment

```bash
#!/bin/bash
# Enterprise WPS assessment workflow

echo "=== Enterprise WPS Assessment ==="

# Phase 1: Extended WPS discovery
echo "Phase 1: WPS Discovery"
sudo wash -i wlan0mon -o 3600 > enterprise_wps.txt

# Phase 2: Analyze results
echo "Phase 2: Analysis"
TOTAL=$(wc -l < enterprise_wps.txt)
UNLOCKED=$(awk '$5 == "No"' enterprise_wps.txt | wc -l)

# Phase 3: Generate report
cat > enterprise_wps_report.md << EOF
# Enterprise WPS Assessment Report

## Summary
- Total WPS networks: $TOTAL
- Unlocked (vulnerable): $UNLOCKED
- Date: $(date)

## Findings
$(awk '{printf "- %s (Ch %s, Signal %s, WPS %s, Locked: %s)\n", $1, $2, $3, $4, $5}' enterprise_wps.txt)

## Recommendations
1. Disable WPS on all access points
2. Enable WPS lock feature where WPS is required
3. Implement WPA2-Enterprise for critical networks
4. Regular security assessments
EOF
```

### Small Business Assessment

```bash
#!/bin/bash
# Quick WPS assessment for small business

echo "=== Small Business WPS Assessment ==="
sudo wash -i wlan0mon -o 60 > small_business_wps.txt

VULNERABLE=$(awk '$5 == "No"' small_business_wps.txt | wc -l)
echo "WPS networks found: $(wc -l < small_business_wps.txt)"
echo "Vulnerable (unlocked): $VULNERABLE"

cat > small_business_report.md << EOF
# Small Business WPS Report

## Summary
- Total WPS networks: $(wc -l < small_business_wps.txt)
- Vulnerable networks: $VULNERABLE

## Recommendations
1. Disable WPS immediately
2. Use WPA2 with strong passwords
3. Enable WPS lock feature
4. Regular security monitoring
EOF
```

### Incident Response Investigation

```bash
#!/bin/bash
# WPS forensic evidence collection

EVIDENCE_DIR="evidence/$(date +%Y%m%d_%H%M%S)"
mkdir -p "$EVIDENCE_DIR"

# Start WPS scan
sudo wash -i wlan0mon -o 300 > "$EVIDENCE_DIR/wps_evidence.txt" &
sleep 600
sudo killall wash

# Create forensic copy
cp "$EVIDENCE_DIR/wps_evidence.txt" "$EVIDENCE_DIR/wps_backup.txt"

# Generate integrity checksums
sha256sum "$EVIDENCE_DIR"/* > "$EVIDENCE_DIR/checksums.txt"

# Document chain of custody
cat > "$EVIDENCE_DIR/chain_of_custody.md" << EOF
# Chain of Custody

## Evidence Details
- Date: $(date)
- Collector: $(whoami)
- Description: WPS forensic evidence

## Integrity Checksums
$(cat "$EVIDENCE_DIR/checksums.txt")
EOF
```

### Compliance Testing (PCI DSS)

```bash
#!/bin/bash
# PCI DSS WPS compliance test

sudo wash -i wlan0mon -o 120 > pci_wps_scan.txt

WPS_ENABLED=$(wc -l < pci_wps_scan.txt)
WPS_LOCKED=$(awk '$5 == "Yes"' pci_wps_scan.txt | wc -l)

cat > pci_compliance_report.md << EOF
# PCI DSS WPS Compliance Report

## Test Results
- WPS-enabled networks: $WPS_ENABLED
- WPS-locked networks: $WPS_LOCKED

## Compliance Status
- [ ] Compliant (WPS disabled)
- [ ] Non-compliant (WPS enabled and unlocked)
- [ ] Partially compliant (WPS enabled but locked)

## Recommendations
1. Disable WPS on all payment card network segments
2. If WPS required, enable lock feature
3. Regular security assessments per PCI DSS Requirement 11
EOF
```

## Chapter 7: Detection & Defense

### Detecting WPS Scanning Activity

```bash
# Check for monitor mode interfaces
iwconfig | grep -i "mode:monitor"

# Monitor for Wash processes
ps aux | grep wash | grep -v grep

# Check for Wash log files
find /var/log/wash -name "*.txt" 2>/dev/null
```

### Wireless Intrusion Detection

```bash
#!/bin/bash
# WPS scanning detection script

while true; do
    MONITOR_IFACES=$(iwconfig 2>/dev/null | grep -c "Mode:Monitor")
    WASH_PROCS=$(ps aux | grep -c "wash")

    if [ "$MONITOR_IFACES" -gt 0 ]; then
        echo "[$(date)] ALERT: Monitor mode interface detected"
    fi

    if [ "$WASH_PROCS" -gt 0 ]; then
        echo "[$(date)] ALERT: Wash process detected"
    fi

    sleep 30
done
```

### Defense Strategies

**1. Disable WPS (Most Effective)**

```bash
# OpenWrt
uci set wireless.@wifi-iface[0].wps_pushbutton='0'
uci commit wireless && wifi reload

# DD-WRT
nvram set wps_enabled=0 && nvram commit && reboot
```

**2. Enable WPS Lock**

```bash
# Prevent brute force after failed attempts
# Configure via AP web interface
# Lock triggers after 3-5 failed PIN attempts
```

**3. Network Segmentation**

```bash
# Isolate WPS-enabled networks via VLAN
sudo ip link add link eth0 name eth0.100 type vlan id 100
sudo ip addr add 192.168.100.1/24 dev eth0.100
sudo ip link set eth0.100 up
sudo iptables -A FORWARD -i eth0.100 -o eth0 -j DROP
```

**4. Use WPA3**

```bash
cat > /etc/hostapd/hostapd.conf << 'EOF'
interface=wlan0
driver=nl80211
ssid=SecureNetwork
hw_mode=g
channel=6
ieee80211w=2
wpa=3
wpa_key_mgmt=SAE
wpa_pairwise=CCMP
rsn_pairwise=CCMP
EOF
```

### Incident Response Procedures

1. **Detection**: Monitor for unauthorized WPS scanning
2. **Analysis**: Capture forensic evidence with Wash
3. **Containment**: Disable WPS on affected networks
4. **Eradication**: Remove WPS from all devices, update firmware
5. **Recovery**: Restore normal operations with enhanced security
6. **Lessons Learned**: Document findings and improve procedures

## Chapter 8: Troubleshooting & Resources

### Common Errors

**"No wireless interface found"**
```bash
iwconfig
sudo airmon-ng start wlan0
ip link show wlan0mon
```

**"Interface not in monitor mode"**
```bash
iwconfig wlan0
sudo airmon-ng start wlan0
sudo airmon-ng check kill
```

**"Permission denied"**
```bash
sudo wash -i wlan0mon
sudo usermod -a -G netdev $USER
sudo setcap cap_net_raw,cap_net_admin=eip /usr/bin/wash
```

**"No WPS networks found"**
```bash
sudo wash -i wlan0mon -o 300
sudo wash -i wlan0mon -c 1,6,11
iwconfig wlan0mon
```

### Performance Optimization

```bash
# CPU pinning
taskset -c 0-3 sudo wash -i wlan0mon

# Set CPU governor
sudo cpupower frequency-set -g performance

# Reduce scan duration
sudo wash -i wlan0mon -o 30
```

### FAQ

**Q: What's the difference between Wash and Reaver?**
A: Wash scans for WPS-enabled networks (reconnaissance), while Reaver exploits WPS vulnerabilities (attack).

**Q: Can Wash detect hidden networks?**
A: Yes, Wash can detect WPS information even if the SSID is hidden, as long as WPS is enabled in beacon frames.

**Q: Is Wash legal?**
A: Wash is a legitimate security testing tool. Use only on networks you own or have authorization to test.

**Q: How accurate is Wash?**
A: Generally accurate for detecting WPS-enabled networks. Some networks may have WPS disabled or use non-standard implementations.

### Resources

- **Reaver Documentation**: https://github.com/t6x/reaver-wps-fork
- **Wash Man Page**: `man wash`
- **Bully**: https://github.com/aanarchyy/bully
- **Pixiewps**: https://github.com/suiwiz/pixiewps
- **Aircrack-ng**: https://www.aircrack-ng.org/
- **Kali Linux Forums**: https://forums.kali.org/
- **Reddit /r/netsec**: https://www.reddit.com/r/netsec/

### Legal Considerations

Always ensure proper authorization before scanning wireless networks. Unauthorized network scanning may violate computer crime laws (CFAA, ECPA, and equivalents in other jurisdictions). Document all authorized testing activities.
