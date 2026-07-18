---
tool_name: "blueranger"
display_name: "BlueRanger"
mitre_tactic: "reconnaissance"
mitre_tactic_id: "TA0043"
mitre_subcategory: "bluetooth"
install_command: "sudo apt install blueranger"
version: "1.0"
github_repo: "https://github.com/securestate/blueranger"
kali_tools_page: "https://www.kali.org/tools/blueranger/"
difficulty_level: "beginner"
requires_root: true
gui_available: false
tags: ["bluetooth", "signal", "distance", "rssi", "tracking", "physical-security"]
related_tools: ["bluelog", "ubertooth-util", "btscanner"]
---

# BlueRanger — Bluetooth Signal Strength Tool

## What This Tool Does (in 30 seconds)

BlueRanger measures Bluetooth signal strength (RSSI) to estimate the distance between your device and nearby Bluetooth devices. It's useful for physical security assessments, device tracking, and locating Bluetooth-enabled assets.

---

## Chapter 1: Introduction & Overview

### What is BlueRanger?

BlueRanger is a Bluetooth reconnaissance tool that measures signal strength (RSSI) to estimate the distance between your device and nearby Bluetooth devices. By monitoring how signal strength changes over time, you can track device movement and estimate proximity.

### Key Features

- RSSI-based distance estimation
- Real-time signal monitoring
- Device tracking capability
- Simple command-line interface
- Continuous monitoring mode
- Signal strength logging

### History & Background

- **Created:** By SecureState
- **License:** GPL-3.0
- **Purpose:** Physical security assessments and device tracking
- **Part of:** Kali Linux Bluetooth toolkit

### When to Use BlueRanger

- **Physical security** — Locate Bluetooth devices in a facility
- **Device tracking** — Follow device movement patterns
- **Security assessments** — Test Bluetooth visibility and tracking risks
- **Asset management** — Find lost or misplaced Bluetooth devices
- **Penetration testing** — Physical security component of assessments
- **Red team operations** — Track target movements

### When NOT to Use This Tool

- Without explicit written authorization
- To track individuals without consent
- In areas where scanning could cause interference
- For stalking or harassment purposes

### Legal and Ethical Considerations

- Only track devices you own or have authorization to locate
- Respect privacy laws regarding device tracking
- Document all tracking activities
- Some jurisdictions restrict Bluetooth tracking

### Key Concepts

- **RSSI:** Received Signal Strength Indicator — measured in dBm (typically -30 to -100)
- **Distance Estimation:** Using RSSI and known TX power to estimate distance
- **Path Loss Model:** The relationship between signal strength and distance
- **Signal Variability:** RSSI fluctuates due to multipath, interference, and obstacles

---

## Chapter 2: Installation & Setup

### System Requirements

- **Minimum RAM:** 256 MB
- **Disk Space:** 5 MB
- **Bluetooth:** Any Bluetooth adapter
- **Privileges:** Root required

### Installation Methods

#### Method 1: APT (Recommended)

```bash
sudo apt update
sudo apt install blueranger
```

#### Method 2: From Source

```bash
git clone https://github.com/securestate/blueranger.git
cd blueranger
make
sudo make install
```

### Configuration

BlueRanger requires minimal configuration:
- Bluetooth adapter must be up
- BlueZ stack installed

### Verification

```bash
blueranger -h
# Output: BlueRanger v1.0
```

### Adapter Setup

```bash
# Check adapter
hciconfig -a

# Bring up adapter
sudo hciconfig hci0 up
```

---

## Chapter 3: Basic Usage

### Essential Commands

```bash
# Scan for devices and show signal strength
sudo blueranger

# Monitor specific device
sudo blueranger -t XX:XX:XX:XX:XX:XX

# Continuous monitoring
sudo blueranger -t XX:XX:XX:XX:XX:XX -c

# Use specific interface
sudo blueranger -i hci0
```

### Command-Line Options

| Flag | Description | Example |
|------|-------------|---------|
| `-t` | Target MAC address | `-t AA:BB:CC:DD:EE:FF` |
| `-c` | Continuous mode | `-c` |
| `-i` | Interface | `-i hci0` |
| `-h` | Help | `-h` |

### Quick Reference

```bash
# Quick signal check
sudo blueranger -t AA:BB:CC:DD:EE:FF

# Continuous monitoring with logging
sudo blueranger -t AA:BB:CC:DD:EE:FF -c > signal_log.txt

# Scan all nearby devices
sudo blueranger
```

---

## Chapter 4: Intermediate Usage

### Python Automation

```python
#!/usr/bin/env python3
"""Track Bluetooth device signal strength over time"""

import subprocess
import time
import json

def monitor_device(mac_address, duration=60):
    """Monitor device signal strength"""
    cmd = ['sudo', 'blueranger', '-t', mac_address, '-c']
    readings = []
    
    start = time.time()
    proc = subprocess.Popen(cmd, stdout=subprocess.PIPE, text=True)
    
    try:
        while time.time() - start < duration:
            line = proc.stdout.readline()
            if line:
                readings.append({
                    'timestamp': time.time(),
                    'rssi': line.strip()
                })
    finally:
        proc.terminate()
    
    return readings

if __name__ == '__main__':
    import sys
    if len(sys.argv) != 2:
        print(f"Usage: {sys.argv[0]} <MAC_ADDRESS>")
        sys.exit(1)
    
    readings = monitor_device(sys.argv[1])
    print(json.dumps(readings, indent=2))
```

### Integration with BlueLog

```bash
# Step 1: Discover devices
sudo bluelog -t 300 -o devices.txt

# Step 2: Track signal strength of each device
while read mac; do
    echo "Tracking: $mac"
    sudo blueranger -t "$mac" -c
done < <(grep -oE '([0-9A-Fa-f]{2}:){5}[0-9A-Fa-f]{2}' devices.txt)
```

---

## Chapter 5: Advanced Usage

### Custom Signal Analysis

```python
#!/usr/bin/env python3
"""Analyze Bluetooth signal patterns for distance estimation"""

import statistics
from typing import List

def analyze_signals(readings: List[float]) -> dict:
    """Analyze signal strength readings"""
    return {
        'mean': statistics.mean(readings),
        'stdev': statistics.stdev(readings) if len(readings) > 1 else 0,
        'min': min(readings),
        'max': max(readings),
        'samples': len(readings)
    }

def estimate_distance(rssi: float, tx_power: float = -59) -> float:
    """Estimate distance from RSSI using log-distance path loss model"""
    # n = path loss exponent (2-4 depending on environment)
    n = 2.0
    return 10 ** ((tx_power - rssi) / (10 * n))

# Usage example
readings = [-45, -47, -46, -48, -45, -44, -46, -47, -45, -44]
stats = analyze_signals(readings)
distance = estimate_distance(stats['mean'])
print(f"Average RSSI: {stats['mean']:.1f} dBm")
print(f"Estimated distance: {distance:.2f} meters")
```

### Multi-Device Monitoring

```bash
#!/bin/bash
# monitor_multiple_devices.sh

DEVICES=("AA:BB:CC:DD:EE:FF" "11:22:33:44:55:66")

for mac in "${DEVICES[@]}"; do
    echo "Monitoring: $mac"
    sudo blueranger -t "$mac" -c > "signal_$mac.log" &
done

# Wait for all background processes
wait
echo "All monitoring complete"
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: Physical Security Assessment

```bash
# Find Bluetooth devices in facility
sudo blueranger

# Track signal strength as you move through the building
sudo blueranger -t TARGET_MAC -c

# Log for later analysis
sudo blueranger -t TARGET_MAC -c > signal_log.txt
```

### Scenario 2: Device Location

```bash
# Track device movement
sudo blueranger -t DEVICE_MAC -c > movement.txt

# Analyze movement patterns
python3 << 'EOF'
import re

with open('movement.txt') as f:
    signals = [float(line.strip()) for line in f if line.strip()]

# Stronger signal = closer
# Weaker signal = farther
print(f"Average signal: {sum(signals)/len(signals):.2f}")
print(f"Signal range: {max(signals) - min(signals):.2f}")
print(f"Signal samples: {len(signals)}")
EOF
```

### Scenario 3: Asset Tracking

```bash
#!/bin/bash
# track_assets.sh - Track multiple Bluetooth assets

ASSETS=("AA:BB:CC:DD:EE:FF:Printer" "11:22:33:44:55:66:Laptop")

for asset in "${ASSETS[@]}"; do
    mac=$(echo "$asset" | cut -d: -f1-6)
    name=$(echo "$asset" | cut -d: -f7)
    
    echo "[$name] Tracking..."
    sudo blueranger -t "$mac" -c > "asset_$name.log" &
done

wait
echo "All assets tracked"
```

### Scenario 4: Proximity-Based Access Control Testing

```bash
#!/bin/bash
# test_proximity_access.sh - Test proximity-based Bluetooth access control

echo "=== Proximity Access Control Test ==="

TARGET="AA:BB:CC:DD:EE:FF"
TEST_DISTANCE="5 meters"

echo "Target: $TARGET"
echo "Test distance: $TEST_DISTANCE"
echo "Start monitoring..."

# Capture signal strength at specific distance
sudo blueranger -t "$TARGET" -c -o proximity_test.log &
MONITOR_PID=$!

echo "[*] Move to test distance and press Enter when ready"
read -p "Press Enter to start 30-second capture..."

sleep 30
kill $MONITOR_PID 2>/dev/null

# Analyze results
echo "[*] Analyzing signal strength..."
python3 << 'EOF'
import statistics

with open('proximity_test.log') as f:
    signals = [float(line.strip()) for line in f if line.strip()]

if signals:
    print(f"Samples: {len(signals)}")
    print(f"Average RSSI: {statistics.mean(signals):.2f} dBm")
    print(f"Std Dev: {statistics.stdev(signals):.2f}")
    print(f"Min RSSI: {min(signals):.2f} dBm")
    print(f"Max RSSI: {max(signals):.2f} dBm")
else
    print("No signal data collected")
EOF
```

### Scenario 5: Rogue Device Hunting

```bash
#!/bin/bash
# hunt_rogue_devices.sh - Hunt for rogue Bluetooth devices

echo "=== Rogue Device Hunt ==="

# Baseline: known devices
KNOWN_DEVICES="/etc/bt_known_devices.txt"
SCAN_DURATION=120

echo "[*] Scanning for $SCAN_DURATION seconds..."

# Scan for devices
sudo blueranger > rogue_scan.txt 2>/dev/null &
SCAN_PID=$!

sleep $SCAN_DURATION
kill $SCAN_PID 2>/dev/null

# Extract MAC addresses
echo "[*] Extracting device addresses..."
grep -oE "([0-9A-Fa-f]{2}:){5}[0-9A-Fa-f]{2}" rogue_scan.txt | sort -u > found_devices.txt

# Compare with known devices
echo "[*] Comparing with known devices..."
if [ -f "$KNOWN_DEVICES" ]; then
    ROGUES=$(comm -23 <(sort found_devices.txt) <(sort "$KNOWN_DEVICES"))
    
    if [ -n "$ROGUES" ]; then
        echo "[!] Rogue devices found:"
        echo "$ROGUES"
        
        # Track signal strength of rogues
        echo "[*] Tracking rogue devices..."
        for mac in $ROGUES; do
            echo "Tracking: $mac"
            sudo blueranger -t "$mac" -c > "rogue_$mac.log" &
        done
        wait
    else
        echo "[+] No rogue devices found"
    fi
else
    echo "[!] No known devices list found"
    echo "[*] Creating baseline..."
    cp found_devices.txt "$KNOWN_DEVICES"
    echo "[+] Baseline created with $(wc -l < "$KNOWN_DEVICES") devices"
fi
```

### Scenario 6: Multi-Room Signal Mapping

```bash
#!/bin/bash
# signal_map.sh - Map Bluetooth signal across rooms

echo "=== Bluetooth Signal Mapping ==="

TARGET="AA:BB:CC:DD:EE:FF"
ROOMS=("entrance" "office1" "office2" "conference" "server_room")
DURATION=30

for room in "${ROOMS[@]}"; do
    echo "[*] Scanning room: $room"
    echo "    Move to $room and press Enter"
    read -p "    Press Enter when ready..."
    
    # Capture signal strength
    sudo blueranger -t "$TARGET" -c > "signal_${room}.log" &
    MAP_PID=$!
    
    sleep $DURATION
    kill $MAP_PID 2>/dev/null
    
    # Calculate average signal
    AVG=$(awk '{sum+=$1; count++} END {print sum/count}' "signal_${room}.log" 2>/dev/null || echo "N/A")
    echo "    Average RSSI in $room: $AVG dBm"
done

echo "[+] Signal mapping complete"
echo "[*] Results:"
for room in "${ROOMS[@]}"; do
    AVG=$(awk '{sum+=$1; count++} END {print sum/count}' "signal_${room}.log" 2>/dev/null || echo "N/A")
    echo "    $room: $AVG dBm"
done
```

---

## Chapter 7: Detection & Defense

### Detection Methods

BlueRanger generates Bluetooth inquiry scans that can be detected:

1. **Bluetooth monitoring** — Detect scanning patterns
2. **Signal analysis** — Identify tracking attempts
3. **Device logging** — Monitor for repeated scans
4. **Process monitoring** — Detect blueranger process

### Mitigation Strategies

#### 1. Disable Discoverable Mode

```bash
# Make device non-discoverable
bluetoothctl discoverable off

# Remove device name
bluetoothctl alias ""
```

#### 2. Use Bluetooth Low Energy (BLE)

```bash
# BLE has limited range and is harder to track
# Use BLE instead of Classic Bluetooth where possible
```

#### 3. Physical Security

```bash
# Disable Bluetooth in secure areas
bluetoothctl power off

# Use Faraday shielding for sensitive devices
# Implement access controls
```

#### 4. Monitoring

```bash
# Monitor for Bluetooth scanning activity
btmon -t | grep -i "inquiry"

# Log all Bluetooth connections
journalctl -f | grep -i bluetooth
```

---

## Chapter 8: Troubleshooting & Resources

### Common Errors

#### Error: "No Bluetooth adapter found"

**Solution:**
```bash
# Check adapter
sudo hciconfig -a

# Restart Bluetooth service
sudo systemctl restart bluetooth

# Load kernel module
sudo modprobe btusb
```

#### Error: "Device not found"

**Solution:**
```bash
# Ensure device is in range
# Check device is discoverable
sudo hcitool scan

# Check signal strength
sudo hcitool rssi AA:BB:CC:DD:EE:FF
```

#### Error: "Permission denied"

**Solution:**
```bash
sudo blueranger
```

#### Error: "No signal data"

**Solution:**
```bash
# Move closer to target
# Check adapter is working
hciconfig hci0

# Reset adapter
sudo hciconfig hci0 reset
```

### Resources

- [BlueRanger GitHub](https://github.com/securestate/blueranger)
- [Bluetooth Security Guide](https://www.bluetooth.com/)
- [Kali Linux Tools](https://www.kali.org/tools/blueranger/)
- [BlueZ Documentation](http://www.bluez.org/)

### Tips and Best Practices

- Move slowly when tracking to get accurate signal readings
- Multiple readings provide better distance estimates
- Environmental factors (walls, people) affect RSSI accuracy
- Use consistent positioning for repeatable measurements

---

## Resources

| Resource | URL |
|----------|-----|
| Official Repository | https://github.com/securestate/blueranger |
| Kali Tools Page | https://www.kali.org/tools/blueranger/ |
| BlueZ Documentation | http://www.bluez.org/ |
| Bluetooth SIG | https://www.bluetooth.com/ |
| MITRE ATT&CK | https://attack.mitre.org/techniques/T1557/ |
