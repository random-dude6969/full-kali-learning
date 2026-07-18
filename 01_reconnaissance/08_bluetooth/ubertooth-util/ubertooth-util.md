---
tool_name: "ubertooth-util"
display_name: "Ubertooth Util"
mitre_tactic: "reconnaissance"
mitre_tactic_id: "TA0043"
mitre_subcategory: "bluetooth"
install_command: "sudo apt install ubertooth"
version: "2020.12.R1"
github_repo: "https://github.com/greatscottgadgets/ubertooth"
kali_tools_page: "https://www.kali.org/tools/ubertooth/"
difficulty_level: "advanced"
requires_root: true
gui_available: true
tags: ["bluetooth", "ble", "sdr", "hardware", "sniffing", "spectrum-analysis", "2.4ghz"]
related_tools: ["bettercap", "btscanner", "wireshark"]
---

# Ubertooth Util — Bluetooth SDR Tool

## What This Tool Does (in 30 seconds)

Ubertooth Util is the command-line interface for the Ubertooth One, an open-source 2.4 GHz Bluetooth development tool. It enables advanced Bluetooth sniffing, analysis, and testing using software-defined radio (SDR). Unlike software-only tools, Ubertooth One provides raw access to the 2.4 GHz spectrum, allowing deep protocol analysis and custom firmware development.

---

## Chapter 1: Introduction & Overview

### What is Ubertooth Util?

Ubertooth Util is the command-line interface for the Ubertooth One hardware platform. It provides advanced Bluetooth and BLE sniffing, analysis, and testing capabilities using software-defined radio. The Ubertooth One is an open-source 2.4 GHz wireless development tool that can capture Bluetooth traffic at the physical layer.

### Key Features

- Bluetooth and BLE sniffing at physical layer
- Channel hopping and tracking
- Raw packet capture
- Spectrum analysis (2.4 GHz band)
- Real-time analysis
- Custom firmware support
- IQ data capture
- Protocol research capabilities

### History & Background

- **Created:** By Great Scott Gadgets (Michael Ossmann)
- **License:** GPL-2.0
- **Hardware:** Ubertooth One ($100-150)
- **Purpose:** Bluetooth security research and protocol analysis
- **Part of:** Kali Linux Bluetooth toolkit

### When to Use Ubertooth Util

- **Advanced Bluetooth research** — Deep protocol analysis
- **Security assessments** — Capture Bluetooth traffic at physical layer
- **BLE analysis** — Analyze Bluetooth Low Energy devices
- **Protocol development** — Test Bluetooth implementations
- **Forensics** — Capture and analyze Bluetooth communications
- **Spectrum analysis** — Monitor 2.4 GHz band activity

### When NOT to Use This Tool

- Without proper authorization
- For intercepting private communications
- In environments where RF interference could cause issues
- Without understanding of RF fundamentals

### Legal and Ethical Considerations

- Only capture traffic on networks you own or have authorization to test
- RF interception laws vary by jurisdiction
- Physical layer capture is more invasive than software-only tools
- Document all capture activities
- Some captures may require warrants depending on jurisdiction

### Key Concepts

- **SDR (Software-Defined Radio):** Radio implemented in software rather than hardware
- **Physical Layer:** The lowest layer of the Bluetooth protocol stack
- **IQ Data:** In-phase and Quadrature data representing raw RF signals
- **Channel Hopping:** Bluetooth's frequency hopping spread spectrum technique
- **BLE (Bluetooth Low Energy):** Modern, low-power Bluetooth protocol
- **2.4 GHz ISM Band:** The frequency band used by Bluetooth (2400-2483.5 MHz)

---

## Chapter 2: Installation & Setup

### System Requirements

- **Minimum RAM:** 2 GB
- **Disk Space:** 200 MB
- **Hardware:** Ubertooth One ($100-150)
- **USB:** USB 2.0 port (USB 3.0 may have issues)
- **Privileges:** Root required

### Installation Methods

#### Method 1: APT (Recommended)

```bash
sudo apt update
sudo apt install ubertooth
```

#### Method 2: From Source

```bash
# Clone Ubertooth repository
git clone https://github.com/greatscottgadgets/ubertooth.git
cd ubertooth/host

# Build
mkdir build && cd build
cmake ..
make
sudo make install
```

### Firmware Update

```bash
# Check current firmware version
ubertooth-util -v

# Update firmware (if needed)
ubertooth-dfu -d bluetooth_rxtx.dfu
```

### Configuration

Ubertooth requires:
- Ubertooth One hardware connected via USB
- USB permissions configured
- Firmware up to date

### Verification

```bash
# Check Ubertooth connection
ubertooth-util -v

# Should show firmware version and hardware info
# Example output: Firmware version: 2020.12.R1
```

### USB Setup

```bash
# Check USB connection
lsusb | grep -i "1d50\|ubertooth"

# Set USB permissions (if needed)
echo 'SUBSYSTEM=="usb", ATTR{idVendor}=="1d50", MODE="0666"' | \
    sudo tee /etc/udev/rules.d/99-ubertooth.rules
sudo udevadm control --reload-rules
```

---

## Chapter 3: Basic Usage

### Essential Commands

```bash
# Check Ubertooth status
ubertooth-util -v

# Get firmware version
ubertooth-util -V

# Start Bluetooth sniffing
ubertooth-btbb

# Capture BLE packets
ubertooth-ble -f

# Spectrum analysis
ubertooth-specan

# General receiver
ubertooth-rx
```

### Command Reference

| Command | Description |
|---------|-------------|
| `ubertooth-util` | Utility functions (status, reset, etc.) |
| `ubertooth-btbb` | Bluetooth baseband sniffer |
| `ubertooth-ble` | BLE sniffer |
| `ubertooth-specan` | Spectrum analyzer |
| `ubertooth-rx` | General receiver |
| `ubertooth-dfu` | Device Firmware Update |

### Quick Reference

```bash
# Check device
ubertooth-util -v

# Capture BLE advertising
ubertooth-ble -f

# Save capture
ubertooth-ble -f -c capture.cfifo

# Spectrum scan
ubertooth-specan -f 2400,2480
```

---

## Chapter 4: Intermediate Usage

### Spectrum Analysis

```bash
# Start spectrum analysis
ubertooth-util -s

# Specify frequency range (MHz)
ubertooth-util -s -f 2400,2480

# Continuous scan mode
ubertooth-util -s -c

# Save spectrum data
ubertooth-util -s -o spectrum.csv
```

### Channel Hopping

```bash
# Enable channel hopping
ubertooth-util -h

# Specify channels
ubertooth-util -h -C 0,1,2,3,4,5,6

# Set hop rate
ubertooth-util -h -r 50  # 50 hops/second
```

### BLE Operations

```bash
# Sniff BLE advertisements
ubertooth-util -b

# Sniff specific channels
ubertooth-util -b -c 37,38,39

# Follow connections
ubertooth-util -b -f AA:BB:CC:DD:EE:FF

# Capture BLE packets
ubertooth-util -p

# Filter by PDU type
ubertooth-util -p --type ADV_IND

# Save to pcap
ubertooth-util -p -w ble_capture.pcap
```

### Configuration

```bash
# Set TX power
ubertooth-util -t 4  # Max power

# Set modulation
ubertooth-util -m GFSK  # Gaussian FSK

# Set data rate
ubertooth-util -d 1M  # 1 Mbps

# LED control
ubertooth-util -l 1  # LED 1 on
ubertooth-util -l 0  # All LEDs off
ubertooth-util -l b  # Blink mode
```

### Integration with Wireshark

```bash
# Capture for Wireshark
ubertooth-util -p -w bt_ble.pcap

# Open in Wireshark
wireshark bt_ble.pcap
```

### Python Integration

```python
#!/usr/bin/env python3
"""ubertooth_capture.py - Ubertooth BLE capture"""

import subprocess
import struct

def capture_ble(duration=10):
    """Capture BLE packets with Ubertooth"""
    cmd = ["ubertooth-util", "-b", "-p"]
    result = subprocess.run(cmd, timeout=duration, capture_output=True)
    return result.stdout

def parse_capture(data):
    """Parse captured BLE packets"""
    packets = []
    # Parse pcap format
    return packets

# Usage
print("[*] Capturing BLE for 10 seconds...")
data = capture_ble(10)
packets = parse_capture(data)
print(f"[+] Captured {len(packets)} packets")
```

---

## Chapter 5: Advanced Usage

### Custom Firmware

```bash
# Clone Ubertooth repository
git clone https://github.com/greatscottgadgets/ubertooth.git
cd ubertooth/firmware

# Build firmware
make

# Flash firmware
ubertooth-util -D ubertooth-one-firmware.bin
```

### Advanced BLE Analysis

```bash
# Follow BLE connection
ubertooth-util -b -f 11:22:33:44:55:66

# Monitor connection parameters
ubertooth-util -b --follow --params

# Decode encrypted connections (with key)
ubertooth-util -b --decrypt --key 00112233445566778899aabbccddeeff

# Capture and decode ATT
ubertooth-util -b -p --decode ATT

# Analyze GATT
ubertooth-util -b --gatt-dump

# Track RSSI
ubertooth-util -b --rssi --rssi-log rssi.csv
```

### Spectrum Analysis Advanced

```bash
# High-resolution spectrum
ubertooth-util -s -r 1000000  # 1 Msps

# IQ data capture
ubertooth-util -q -w iq_data.raw

# Analyze with custom scripts
python3 analyze_spectrum.py iq_data.raw

# Scan specific frequencies
ubertooth-util -s -f 2426,2428,2430  # BLE channels

# Wideband scan
ubertooth-util -s -f 2400,2500 -r 100000
```

### Automated BLE Monitoring

```bash
#!/bin/null
# ubertooth_monitor.sh - Continuous BLE monitoring

LOG_DIR="/tmp/ubertooth_$(date +%Y%m%d)"
mkdir -p "$LOG_DIR"

echo "[*] Starting Ubertooth BLE monitoring"

while true; do
    TIMESTAMP=$(date +%H%M%S)
    
    # 5-minute capture
    timeout 300 ubertooth-util -b -p -w "$LOG_DIR/capture_${TIMESTAMP}.pcap" 2>/dev/null
    
    # Quick analysis
    PACKETS=$(tshark -r "$LOG_DIR/capture_${TIMESTAMP}.pcap" 2>/dev/null | wc -l)
    DEVICES=$(tshark -r "$LOG_DIR/capture_${TIMESTAMP}.pcap" -T fields -e bluetooth.address 2>/dev/null | sort -u | wc -l)
    
    echo "[$(date +%H:%M:%S)] Packets: $PACKETS, Devices: $DEVICES"
done
```

### Custom Protocol Decoder

```python
#!/usr/bin/env python3
"""ble_protocol_decoder.py - Custom BLE protocol decoder"""

import struct

class BLEDecoder:
    def __init__(self):
        self.pdu_types = {
            0x00: "ADV_IND",
            0x01: "ADV_DIRECT_IND",
            0x02: "ADV_NONCONN_IND",
            0x03: "SCAN_REQ",
            0x04: "SCAN_RSP",
            0x05: "CONNECT_IND",
            0x06: "ADV_SCAN_IND"
        }
    
    def decode_pdu(self, data):
        if len(data) < 1:
            return None
        
        pdu_type = data[0] & 0x0F
        length = data[1] if len(data) > 1 else 0
        
        return {
            "type": self.pdu_types.get(pdu_type, f"Unknown ({pdu_type})"),
            "length": length,
            "data": data[2:length+2].hex() if len(data) > length+1 else ""
        }

# Usage
decoder = BLEDecoder()
# Parse captured data
```

### Integration with gnuradio

```bash
# Capture IQ data for gnuradio
ubertooth-util -q -w iq_data.raw

# Process in gnuradio
gnuradio-companion ble_analysis.grc
```

### Integration with Scapy

```python
#!/usr/bin/env python3
"""ubertooth_scapy.py - Ubertooth with Scapy"""

from scapy.all import *
from scapy.layers.bluetooth import *

def capture_with_ubertooth():
    """Use Ubertooth with Scapy"""
    import subprocess
    subprocess.run(["ubertooth-util", "-b", "-p", "-w", "ble.pcap"])
    
    packets = rdpcap("ble.pcap")
    
    for pkt in packets:
        if BLE_ADV_IND in pkt:
            print(f"Advertisement from {pkt[BLE_ADV_IND].src}")

capture_with_ubertooth()
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: IoT Security Testing

```bash
#!/bin/bash
# iot_bt_audit.sh - IoT Bluetooth security audit

echo "=== IoT Bluetooth Security Audit ==="

# Step 1: Spectrum analysis
echo "[1] Spectrum Analysis..."
ubertooth-util -s -f 2400,2480 -o /tmp/iot_spectrum.csv

# Step 2: BLE device discovery
echo "[2] BLE Device Discovery..."
timeout 60 ubertooth-util -b -p -w /tmp/iot_ble.pcap

# Step 3: Analyze advertisements
echo "[3] Analyzing Advertisements..."
tshark -r /tmp/iot_ble.pcap -Y "btle" -T fields \
    -e bluetooth.address -e btle.advertising_header.pdu_type 2>/dev/null | \
    sort | uniq -c | sort -rn

# Step 4: Track device patterns
echo "[4] Device Communication Patterns..."
tshark -r /tmp/iot_ble.pcap -q -z conv,btle 2>/dev/null
```

### Scenario 2: Medical Device Assessment

```bash
#!/bin/bash
# medical_device_bt_audit.sh - Medical device BT assessment

echo "=== Medical Device BT Security Assessment ==="

# Capture medical device BLE traffic
echo "[*] Capturing medical device BLE..."
timeout 300 ubertooth-util -b -p -w medical_bt.pcap

# Analyze for vulnerabilities
echo "[*] Analyzing for vulnerabilities..."

# Check for unencrypted data
tshark -r medical_bt.pcap -Y "btle.data" 2>/dev/null | head -20

# Check for pairing attempts
tshark -r medical_bt.pcap -Y "btle.data_header.opcode == 0x05" 2>/dev/null
```

### Scenario 3: Corporate Device Tracking

```bash
#!/bin/null
# corporate_bt_tracking.sh - Track corporate Bluetooth devices

echo "=== Corporate BT Device Tracking ==="

while true; do
    TIMESTAMP=$(date +%H%M%S)
    
    # Capture BLE advertisements
    timeout 60 ubertooth-util -b -p -w "corporate_bt_${TIMESTAMP}.pcap" 2>/dev/null
    
    # Extract unique devices
    DEVICES=$(tshark -r "corporate_bt_${TIMESTAMP}.pcap" -T fields -e bluetooth.address 2>/dev/null | sort -u)
    
    # Log new devices
    for dev in $DEVICES; do
        if ! grep -q "$dev" /tmp/known_bt_devices.txt 2>/dev/null; then
            echo "[NEW] $(date) - $dev" >> /tmp/new_bt_devices.log
            echo "$dev" >> /tmp/known_bt_devices.txt
        fi
    done
    
    sleep 300
done
```

### Scenario 4: BT Forensics

```bash
#!/bin/null
# bt_forensics.sh - Bluetooth forensics investigation

echo "=== BT Forensics Investigation ==="

EVIDENCE_DIR="/tmp/bt_forensics_$(date +%Y%m%d_%H%M%S)"
mkdir -p "$EVIDENCE_DIR"

# Step 1: Capture evidence
echo "[1] Capturing BT evidence..."
timeout 120 ubertooth-util -b -p -w "$EVIDENCE_DIR/evidence.pcap"

# Step 2: Spectrum analysis
echo "[2] Spectrum analysis..."
ubertooth-util -s -o "$EVIDENCE_DIR/spectrum.csv"

# Step 3: Device inventory
echo "[3] Device inventory..."
tshark -r "$EVIDENCE_DIR/evidence.pcap" -T fields -e bluetooth.address 2>/dev/null | \
    sort -u > "$EVIDENCE_DIR/devices.txt"

# Step 4: Timeline
echo "[4] Creating timeline..."
tshark -r "$EVIDENCE_DIR/evidence.pcap" -T fields -e frame.time -e bluetooth.address 2>/dev/null > "$EVIDENCE_DIR/timeline.txt"

echo "[+] Evidence collected in $EVIDENCE_DIR"
```

---

## Chapter 7: Detection & Defense

### Detection Methods

```bash
# Detect Ubertooth processes
ps aux | grep ubertooth

# Check USB device
lsusb | grep 1d50:6002  # Ubertooth One VID/PID

# Monitor USB activity
sudo usbmon | grep -i ubertooth

# Check system logs
dmesg | grep -i "ubertooth\|1d50"

# Monitor HCI events
sudo btmon -t | grep -i "ubertooth"
```

### Detection Script

```bash
#!/bin/null
# detect_ubertooth.sh - Detect Ubertooth devices

echo "=== Ubertooth Detection ==="

# Check USB
echo "[1] USB Devices:"
lsusb | grep -i "1d50\|ubertooth"

# Check running processes
echo "[2] Running Processes:"
ps aux | grep ubertooth | grep -v grep

# Check kernel modules
echo "[3] Kernel Modules:"
lsmod | grep -i "ubertooth\|btusb"

# Monitor for new USB devices
echo "[4] USB Monitor (Ctrl+C to stop):"
udevadm monitor --subsystem-match=usb | grep -i "1d50"
```

### Defensive Countermeasures

```bash
# Completely disable BT
sudo bluetoothctl power off
sudo systemctl stop bluetooth

# Blacklist modules
echo "blacklist btusb" | sudo tee /etc/modprobe.d/bluetooth-blacklist.conf

# Physical security
# - Use USB port locks
# - Disable unused USB ports in BIOS
# - Monitor USB device connections
```

### RF Shielding

```bash
# Physical RF shielding
# - Use Faraday cages for sensitive areas
# - Deploy 2.4GHz jammers (authorized only)
# - Use frequency-selective surfaces

# Software-based detection
while true; do
    ubertooth-util -s -f 2400,2480 2>/dev/null | grep -i "anomaly\|custom"
    sleep 10
done
```

### Monitoring Script

```bash
#!/bin/null
# ubertooth_detection_monitor.sh - Real-time Ubertooth detection

LOG="/var/log/ubertooth_alerts.log"

while true; do
    # Check for Ubertooth USB
    if lsusb | grep -q "1d50:6002"; then
        echo "[ALERT] $(date) - Ubertooth device detected via USB" >> "$LOG"
    fi
    
    # Check for Ubertooth processes
    if pgrep -f "ubertooth" > /dev/null; then
        echo "[ALERT] $(date) - Ubertooth process detected" >> "$LOG"
        ps aux | grep ubertooth >> "$LOG"
    fi
    
    # Check for unusual BT activity
    BT_TRAFFIC=$(btmon -t 2>/dev/null | wc -l)
    if [ "$BT_TRAFFIC" -gt 1000 ]; then
        echo "[ALERT] $(date) - High BT traffic detected ($BT_TRAFFIC events)" >> "$LOG"
    fi
    
    sleep 30
done
```

---

## Chapter 8: Troubleshooting & Resources

### Common Errors

#### "No Ubertooth device found"
```bash
# Check USB connection
lsusb | grep -i "1d50\|ubertooth"

# Reset USB device
sudo usbreset 1d50:6002

# Reload kernel module
sudo modprobe -r btusb
sudo modprobe btusb
```

#### "Permission denied"
```bash
sudo ubertooth-util -v

# Set USB permissions
sudo chmod 666 /dev/bus/usb/XXX/YYY

# Add udev rule
echo 'SUBSYSTEM=="usb", ATTR{idVendor}=="1d50", MODE="0666"' | \
    sudo tee /etc/udev/rules.d/99-ubertooth.rules
sudo udevadm control --reload-rules
```

#### "Firmware version mismatch"
```bash
# Check current firmware
ubertooth-util -v

# Update firmware
git clone https://github.com/greatscottgadgets/ubertooth.git
cd ubertooth/firmware
make
sudo ubertooth-util -D ubertooth-one-firmware.bin
```

#### "USB transfer error"
```bash
# Try different USB port
# Use USB 2.0 port (not 3.0)
# Add USB quirks
echo "options usb-storage quirk=1d50:6002:u" | sudo tee /etc/modprobe.d/usb-quirks.conf
```

#### "Device busy"
```bash
# Kill other Ubertooth processes
sudo pkill -f ubertooth

# Reset device
sudo ubertooth-util -r

# Try again
ubertooth-util -v
```

### Firmware Issues

```bash
# Enter DFU mode
sudo ubertooth-util -D

# Flash firmware
dfu-util -D firmware/ubertooth-one-firmware.bin

# Verify flash
sudo ubertooth-util -v
```

### Performance Issues

```bash
# Use USB 2.0 port
lsusb -t  # Check USB tree

# Reduce capture rate
ubertooth-util -b -p -r 1000000

# Disable power management
echo 'on' | sudo tee /sys/bus/usb/devices/*/power/control

# Increase USB buffer
echo 0 | sudo tee /sys/module/usbcore/parameters/usbfs_memory_mb
```

### FAQ

#### Q: Ubertooth not recognized
```bash
# Check USB connection
# Try different USB port
sudo udevadm control --reload-rules && sudo udevadm trigger
dmesg | tail -20
```

#### Q: Can't capture BLE
```bash
# Ensure proper mode
ubertooth-util -b

# Check channel
ubertooth-util -b -c 37,38,39

# Verify firmware
ubertooth-util -v
```

#### Q: Firmware update failed
```bash
# Enter DFU mode: sudo ubertooth-util -D
# Flash: dfu-util -D firmware.bin
# If still fails, use JTAG programmer
```

#### Q: Range issues
```bash
# Check antenna connection
# Remove obstructions
# Use external antenna (if supported)
```

### Resources

- [Ubertooth GitHub](https://github.com/greatscottgadgets/ubertooth)
- [Ubertooth Wiki](https://github.com/greatscottgadgets/ubertooth/wiki)
- [Kali Linux Tools](https://www.kali.org/tools/ubertooth/)
- [Bluetooth SIG](https://www.bluetooth.com/)
- [OSSmann Training](https://greatscottgadgets.com/)

---

## Resources

| Resource | URL |
|----------|-----|
| Official Repository | https://github.com/greatscottgadgets/ubertooth |
| Ubertooth Wiki | https://github.com/greatscottgadgets/ubertooth/wiki |
| Kali Tools Page | https://www.kali.org/tools/ubertooth/ |
| Great Scott Gadgets | https://greatscottgadgets.com/ |
| Bluetooth SIG | https://www.bluetooth.com/ |
| MITRE ATT&CK | https://attack.mitre.org/techniques/T1557/ |
