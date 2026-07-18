---
tool_name: "bluelog"
display_name: "BlueLog"
mitre_tactic: "reconnaissance"
mitre_tactic_id: "TA0043"
mitre_subcategory: "bluetooth"
install_command: "sudo apt install bluelog"
version: "1.0"
github_repo: "https://github.com/MS3FGX/Bluelog"
kali_tools_page: "https://www.kali.org/tools/bluelog/"
difficulty_level: "beginner"
requires_root: true
gui_available: false
tags: ["bluetooth", "logging", "reconnaissance", "scanner", "monitoring"]
related_tools: ["bluetoothctl", "btscanner", "blueranger", "bluesnarfer"]
---

# BlueLog — Bluetooth Scanner & Logger

## What This Tool Does (in 30 seconds)

BlueLog is a simple, lightweight Bluetooth scanner that logs discovered devices to a file. It runs quietly in the background and records all discoverable Bluetooth devices, including their MAC addresses and device names. Ideal for long-term monitoring and device inventory.

---

## Chapter 1: Introduction & Overview

### What is BlueLog?

BlueLog is a Bluetooth reconnaissance tool that scans for and logs nearby Bluetooth devices. It's designed for long-term monitoring, running quietly in the background while recording all discoverable devices. Unlike more complex tools, BlueLog focuses on one job: discovering and logging Bluetooth devices.

### Key Features

- Passive Bluetooth scanning
- Device logging with timestamps
- MAC address and device name capture
- Background operation mode
- Simple command-line interface
- Minimal resource usage
- No configuration required

### History & Background

- **Created:** By MS3FGX (securestate)
- **License:** GPL-3.0
- **Purpose:** Bluetooth device discovery and logging for security assessments
- **Part of:** Kali Linux Bluetooth toolkit

### When to Use BlueLog

- **Physical security assessments** — Map Bluetooth devices in a facility
- **Long-term monitoring** — Track device presence over time
- **Device inventory** — Catalog all Bluetooth devices in range
- **Security audits** — Identify unauthorized or rogue devices
- **Compliance testing** — Verify Bluetooth security policies
- **Red team operations** — Passive reconnaissance before active attacks

### When NOT to Use This Tool

- Without explicit written authorization
- On networks/systems you don't own or have permission to test
- For tracking individuals without consent
- In environments where scanning could cause interference

### Legal and Ethical Considerations

- Only scan areas you have authorization to test
- Respect privacy laws regarding device tracking
- Do not attempt to connect to or exploit discovered devices without permission
- Document all scanning activities
- Some jurisdictions restrict Bluetooth scanning

### Key Concepts

- **Bluetooth Inquiry Scan:** Active discovery of nearby Bluetooth devices
- **BD_ADDR:** Bluetooth Device Address — unique MAC address for each Bluetooth adapter
- **Device Class:** 24-bit identifier indicating device type (phone, computer, audio, etc.)
- **RSSI:** Received Signal Strength Indicator — measures signal strength
- **Discoverable Mode:** When a Bluetooth device responds to inquiry scans

---

## Chapter 2: Installation & Setup

### System Requirements

- **Minimum RAM:** 256 MB
- **Disk Space:** 10 MB
- **Bluetooth:** Any Bluetooth adapter (built-in or USB)
- **Privileges:** Root required for raw Bluetooth access

### Installation Methods

#### Method 1: APT (Recommended)

```bash
sudo apt update
sudo apt install bluelog
```

#### Method 2: From Source

```bash
git clone https://github.com/MS3FGX/Bluelog.git
cd Bluelog
make
sudo make install
```

### Configuration

BlueLog requires minimal configuration:
- Bluetooth adapter must be up and running
- BlueZ stack must be installed (`sudo apt install bluez`)

### Verification

```bash
bluelog -h
# Output: BlueLog v1.0 - Bluetooth scanner/logger
```

### Adapter Setup

```bash
# Check Bluetooth adapter
hciconfig -a

# Bring adapter up
sudo hciconfig hci0 up

# Verify adapter is ready
hciconfig hci0
```

---

## Chapter 3: Basic Usage

### First Run

```bash
# Start Bluetooth scanning (runs until interrupted with Ctrl+C)
sudo bluelog
```

### Essential Commands

```bash
# Scan and log devices (interactive, runs until Ctrl+C)
sudo bluelog

# Scan for specific duration (seconds)
sudo bluelog -t 300

# Verbose output (shows more details)
sudo bluelog -v

# Log to specific file
sudo bluelog -o /var/log/bluetooth_devices.txt

# Use specific Bluetooth interface
sudo bluelog -i hci0
```

### Command-Line Options

| Flag | Description | Example |
|------|-------------|---------|
| `-t` | Scan duration (seconds) | `-t 300` |
| `-v` | Verbose output | `-v` |
| `-o` | Output file | `-o log.txt` |
| `-i` | Interface | `-i hci0` |
| `-h` | Help | `-h` |

### Quick Reference

```bash
# Quick scan (60 seconds)
sudo bluelog -t 60

# Background monitoring (1 hour)
sudo bluelog -t 3600 -o /var/log/bt_monitor.log &

# Verbose scan to file
sudo bluelog -v -t 300 -o scan_results.txt
```

---

## Chapter 4: Intermediate Usage

### Batch Processing

```bash
#!/bin/bash
# monitor_bluetooth.sh - Automated Bluetooth monitoring

LOGFILE="/var/log/bluetooth_monitor.log"
DURATION=3600  # 1 hour

echo "Starting Bluetooth monitoring..."
sudo bluelog -t $DURATION -o "$LOGFILE" -v

echo "Scan complete. Results:"
cat "$LOGFILE"
```

### Integration with Other Tools

#### BlueLog + btscanner

```bash
# Step 1: Quick discovery with BlueLog
sudo bluelog -t 300 -o devices.txt

# Step 2: Detailed scan with btscanner
sudo btscanner
```

#### BlueLog + Wireshark

```bash
# Capture Bluetooth traffic for analysis
sudo bluelog -v | tee bluelog.txt

# Analyze with Wireshark (if Bluetooth capture supported)
```

### Python Automation

```python
#!/usr/bin/env python3
"""Analyze BlueLog output for device statistics"""

import re
from collections import Counter

def analyze_bluelog(logfile):
    """Parse BlueLog output and count device sightings"""
    devices = []
    
    with open(logfile) as f:
        for line in f:
            # Extract MAC addresses
            macs = re.findall(r'([0-9A-Fa-f]{2}[:-]){5}([0-9A-Fa-f]{2})', line)
            devices.extend([m[0]+m[1] for m in macs])
    
    return Counter(devices)

if __name__ == '__main__':
    import sys
    if len(sys.argv) != 2:
        print(f"Usage: {sys.argv[0]} <logfile>")
        sys.exit(1)
    
    devices = analyze_bluelog(sys.argv[1])
    print(f"Unique devices: {len(devices)}")
    for mac, count in devices.most_common():
        print(f"  {mac}: {count} sightings")
```

---

## Chapter 5: Advanced Usage

### Custom Continuous Monitoring

```bash
#!/bin/bash
# custom_bluetooth_monitor.sh - Continuous monitoring with alerts

LOGFILE="/var/log/bluetooth_$(date +%Y%m%d).log"
INTERFACE="hci0"
INTERVAL=300
ALERT_EMAIL="admin@company.com"

while true; do
    echo "[$(date)] Starting scan..."
    sudo bluelog -i $INTERFACE -t $INTERVAL -o "$LOGFILE" -v
    
    # Analyze results
    NEW_DEVICES=$(grep -c "New device" "$LOGFILE" 2>/dev/null || echo 0)
    echo "[$(date)] Found $NEW_DEVICES new devices"
    
    # Alert on suspicious devices
    if [ "$NEW_DEVICES" -gt 0 ]; then
        grep "New device" "$LOGFILE" | mail -s "New BT devices detected" "$ALERT_EMAIL"
    fi
    
    sleep 60
done
```

### Multi-Adapter Monitoring

```bash
#!/bin/bash
# multi_adapter_monitor.sh - Monitor with multiple Bluetooth adapters

for i in 0 1 2; do
    if hciconfig hci$i &>/dev/null; then
        sudo bluelog -i hci$i -t 3600 -o "log_hci$i.txt" &
        echo "[*] Started monitoring on hci$i"
    fi
done

wait
echo "[*] All adapters complete"
```

### Advanced Analysis

```python
#!/usr/bin/env python3
"""Advanced Bluetooth device analysis and tracking"""

import json
from datetime import datetime

class BluetoothAnalyzer:
    def __init__(self):
        self.devices = {}
    
    def add_device(self, mac, name=None, timestamp=None):
        """Add discovered device"""
        if mac not in self.devices:
            self.devices[mac] = {
                'first_seen': timestamp or datetime.now().isoformat(),
                'last_seen': timestamp or datetime.now().isoformat(),
                'name': name,
                'sightings': 1
            }
        else:
            self.devices[mac]['last_seen'] = timestamp or datetime.now().isoformat()
            self.devices[mac]['sightings'] += 1
    
    def get_statistics(self):
        """Get device statistics"""
        return {
            'total_devices': len(self.devices),
            'frequent_devices': sorted(
                self.devices.items(),
                key=lambda x: x[1]['sightings'],
                reverse=True
            )[:10]
        }

# Usage
analyzer = BluetoothAnalyzer()
# Parse bluelog output and feed to analyzer
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: Physical Security Assessment

```bash
# Monitor Bluetooth devices in facility
sudo bluelog -t 3600 -o facility_scan.txt -v

# Analyze findings
echo "=== Bluetooth Devices Found ==="
cat facility_scan.txt | grep -E "Device|MAC" | sort | uniq

# Count devices by type
echo "=== Device Summary ==="
wc -l facility_scan.txt
```

### Scenario 2: Device Inventory

```bash
# Create comprehensive device inventory
sudo bluelog -t 1800 -o inventory.txt -v

# Generate report
python3 << 'EOF'
import re
from collections import Counter

with open('inventory.txt') as f:
    content = f.read()

macs = re.findall(r'([0-9A-Fa-f]{2}[:-]){5}[0-9A-Fa-f]{2}', content)
unique_macs = list(set(macs))

print(f"Total unique devices: {len(unique_macs)}")
print("\nDevice list:")
for mac in unique_macs:
    print(f"  {mac}")
EOF
```

### Scenario 3: Long-term Monitoring

```bash
# Set up continuous monitoring (24 hours)
nohup sudo bluelog -t 86400 -o /var/log/bluetooth_daily.txt -v &

# Daily analysis with crontab
crontab -e
# Add: 0 0 * * * /usr/local/bin/analyze_bluetooth.sh
```

### Scenario 4: Network Segmentation Verification

```bash
#!/bin/bash
# verify_segmentation.sh - Verify Bluetooth segmentation

echo "=== Bluetooth Segmentation Verification ==="

# Scan from different network segments
LOCATIONS=("office" "guest_wifi" "iot_network" "management")

for location in "${LOCATIONS[@]}"; do
    echo "[*] Scanning $location segment..."
    
    # Set interface based on segment
    case $location in
        "office") IFACE="hci0" ;;
        "guest_wifi") IFACE="hci1" ;;
        "iot_network") IFACE="hci2" ;;
        "management") IFACE="hci3" ;;
    esac
    
    sudo bluelog -i $IFACE -t 300 -o "scan_${location}.txt" -v
    
    DEVICE_COUNT=$(grep -c "Device" "scan_${location}.txt" 2>/dev/null || echo "0")
    echo "[+] $location: $DEVICE_COUNT devices found"
done

echo "[+] Segmentation verification complete"
```

### Scenario 5: Incident Response Scanning

```bash
#!/bin/bash
# incident_bt_scan.sh - Quick BT scan during security incident

echo "=== Incident Response BT Scan ==="
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
OUTPUT_DIR="/tmp/incident_bt_${TIMESTAMP}"
mkdir -p "$OUTPUT_DIR"

# Quick scan for suspicious devices
echo "[*] Quick scan (60 seconds)..."
sudo bluelog -t 60 -o "$OUTPUT_DIR/quick_scan.txt" -v

# Full scan if time permits
echo "[*] Full scan (5 minutes)..."
sudo bluelog -t 300 -o "$OUTPUT_DIR/full_scan.txt" -v

# Compare results
echo "[*] Analyzing results..."
QUICK_COUNT=$(grep -c "Device" "$OUTPUT_DIR/quick_scan.txt" 2>/dev/null || echo "0")
FULL_COUNT=$(grep -c "Device" "$OUTPUT_DIR/full_scan.txt" 2>/dev/null || echo "0")

echo "Quick scan: $QUICK_COUNT devices"
echo "Full scan: $FULL_COUNT devices"

# Generate report
cat > "$OUTPUT_DIR/incident_report.txt" << EOF
# Incident BT Scan Report
Date: $(date)
Quick scan devices: $QUICK_COUNT
Full scan devices: $FULL_COUNT

## Quick Scan Results
$(cat "$OUTPUT_DIR/quick_scan.txt")

## Full Scan Results
$(cat "$OUTPUT_DIR/full_scan.txt")
EOF

echo "[+] Report: $OUTPUT_DIR/incident_report.txt"
```

### Scenario 6: Compliance Audit

```bash
#!/bin/bash
# compliance_audit.sh - Bluetooth compliance audit

echo "=== Bluetooth Compliance Audit ==="

# Define compliance checks
CHECKS=(
    "unauthorized_devices:Devices not in whitelist"
    "discoverable_devices:Devices in discoverable mode"
    "legacy_devices:Devices using legacy pairing"
    "weak_encryption:Devices with weak encryption"
)

# Scan for devices
sudo bluelog -t 600 -o compliance_scan.txt -v

# Analyze results
echo "[*] Running compliance checks..."

# Check 1: Unauthorized devices
echo "[Check 1] Unauthorized devices..."
WHITELIST="/etc/bt_whitelist.txt"
if [ -f "$WHITELIST" ]; then
    grep -oE "([0-9A-Fa-f]{2}:){5}[0-9A-Fa-f]{2}" compliance_scan.txt | while read mac; do
        if ! grep -qi "$mac" "$WHITELIST"; then
            echo "  [FAIL] Unauthorized: $mac"
        fi
    done
else
    echo "  [WARN] No whitelist found"
fi

# Check 2: Scan duration compliance
echo "[Check 2] Scan duration compliance..."
SCAN_DURATION=$(grep -c "Device" compliance_scan.txt)
if [ "$SCAN_DURATION" -gt 100 ]; then
    echo "  [INFO] High device count: $SCAN_DURATION devices"
fi

echo "[+] Compliance audit complete"
```

---

## Chapter 7: Detection & Defense

### Detection Methods

BlueLog generates Bluetooth inquiry scans that can be detected:

1. **Bluetooth monitoring** — Detect inquiry scan patterns
2. **Device logging** — Monitor for scanning devices
3. **Signal analysis** — Identify scanning patterns
4. **Process monitoring** — Detect bluelog process

### Mitigation Strategies

#### 1. Make Devices Non-Discoverable

```bash
# Disable discoverable mode on Bluetooth devices
bluetoothctl discoverable off

# Make device invisible to scans
bluetoothctl alias ""
```

#### 2. Bluetooth Monitoring

```bash
# Monitor for scanning activity
btmon -t | grep -i "inquiry"

# Detect repeated scans
journalctl -f | grep -i bluetooth
```

#### 3. Physical Security

```bash
# Disable Bluetooth when not needed
bluetoothctl power off

# Use Faraday cages for sensitive areas
# Implement Bluetooth access controls
```

#### 4. Policy Enforcement

```bash
# Implement BYOD policies
# Restrict Bluetooth in sensitive areas
# Regular security audits
# Device whitelisting
```

---

## Chapter 8: Troubleshooting & Resources

### Common Errors

#### Error: "No Bluetooth adapter found"

**Cause:** Bluetooth adapter not detected.

**Solution:**
```bash
# Check Bluetooth adapter
hciconfig -a

# Restart Bluetooth service
sudo systemctl restart bluetooth

# Install drivers
sudo apt install bluez

# Load kernel module
sudo modprobe btusb
```

#### Error: "Permission denied"

**Cause:** Not running as root.

**Solution:**
```bash
sudo bluelog
```

#### Error: "No devices found"

**Cause:** No discoverable Bluetooth devices nearby.

**Solution:**
```bash
# Check adapter is enabled
sudo hciconfig hci0 up

# Scan manually first
sudo hcitool scan

# Ensure adapter is in scan mode
sudo hciconfig hci0 piscan
```

#### Error: "Device busy"

**Cause:** Another process is using the Bluetooth adapter.

**Solution:**
```bash
# Find and kill other BT processes
ps aux | grep -i bluetooth
sudo pkill -f bluetooth

# Reset adapter
sudo hciconfig hci0 down
sudo hciconfig hci0 up
```

### Resources

- [BlueLog GitHub](https://github.com/MS3FGX/Bluelog)
- [Kali Bluetooth Tools](https://www.kali.org/tools/bluelog/)
- [Bluetooth Security Guide](https://www.bluetooth.com/)
- [BlueZ Documentation](http://www.bluez.org/)
- [MITRE ATT&CK](https://attack.mitre.org/techniques/T1557/)

---

## Resources

| Resource | URL |
|----------|-----|
| Official Repository | https://github.com/MS3FGX/Bluelog |
| Kali Tools Page | https://www.kali.org/tools/bluelog/ |
| BlueZ Documentation | http://www.bluez.org/ |
| Bluetooth SIG | https://www.bluetooth.com/ |
| MITRE ATT&CK | https://attack.mitre.org/techniques/T1557/ |
