---
tool_name: "redfang"
display_name: "RedFang"
mitre_tactic: "reconnaissance"
mitre_tactic_id: "TA0043"
mitre_subcategory: "bluetooth"
install_command: "sudo apt install redfang"
version: "2.0"
github_repo: "https://github.com/hchoices/redfang"
kali_tools_page: "https://www.kali.org/tools/redfang/"
difficulty_level: "intermediate"
requires_root: true
gui_available: false
tags: ["bluetooth", "brute-force", "sniffing", "pin", "authentication"]
related_tools: ["bluesnarfer", "btscanner", "bluelog"]
---

# RedFang — Bluetooth Brute-Force & Sniffing Tool

## What This Tool Does (in 30 seconds)

RedFang is a Bluetooth security assessment tool that performs brute-force attacks on Bluetooth PINs and can sniff unencrypted Bluetooth traffic. It's used to test the strength of Bluetooth authentication mechanisms and capture data transmitted over Bluetooth connections.

---

## Chapter 1: Introduction & Overview

### What is RedFang?

RedFang is a Bluetooth security assessment tool that performs brute-force attacks on Bluetooth PINs and can sniff unencrypted traffic. It's used to test the strength of Bluetooth authentication mechanisms and identify weak PIN configurations.

### Key Features

- PIN brute-force attacks
- Traffic sniffing
- Pairing attack support
- Multiple attack modes
- RFCOMM channel scanning
- Connection-based attacks

### History & Background

- **Created:** By H-Choice (hchoices)
- **License:** GPL-2.0
- **Purpose:** Bluetooth authentication testing and traffic analysis
- **Part of:** Kali Linux Bluetooth toolkit

### When to Use RedFang

- **Security assessments** — Test Bluetooth authentication strength
- **Penetration testing** — Evaluate pairing security
- **Compliance testing** — Verify Bluetooth security policies
- **Research** — Study Bluetooth vulnerabilities
- **Forensics** — Capture Bluetooth traffic for analysis

### When NOT to Use This Tool

- Without explicit written authorization
- On devices you don't own or have permission to test
- For unauthorized access to Bluetooth devices
- For intercepting private communications

### Legal and Ethical Considerations

- Only test devices you own or have authorization to access
- Unauthorized Bluetooth access is illegal in most jurisdictions
- Obtain written permission before testing
- Document all activities for accountability
- Traffic sniffing may violate wiretapping laws

### Key Concepts

- **PIN (Personal Identification Number):** 4-16 digit code used for Bluetooth pairing
- **Brute Force:** Systematically trying all possible PIN combinations
- **RFCOMM:** Protocol providing serial port emulation over Bluetooth
- **Sniffing:** Capturing unencrypted Bluetooth traffic
- **Pairing:** Process of establishing a trusted relationship between devices

---

## Chapter 2: Installation & Setup

### System Requirements

- **Minimum RAM:** 256 MB
- **Disk Space:** 5 MB
- **Bluetooth:** Any Bluetooth adapter
- **Privileges:** Root required
- **Target:** Device must be in pairing mode

### Installation Methods

#### Method 1: APT (Recommended)

```bash
sudo apt update
sudo apt install redfang
```

#### Method 2: From Source

```bash
git clone https://github.com/hchoices/redfang.git
cd redfang
make
sudo make install
```

### Configuration

RedFang requires:
- Bluetooth adapter up and running
- BlueZ stack installed
- Target device in range and discoverable

### Verification

```bash
redfang -h
# Output: RedFang v2.0
```

---

## Chapter 3: Basic Usage

### Essential Commands

```bash
# Brute-force PIN
sudo redfang -r TARGET_MAC

# Sniff traffic
sudo redfang -s TARGET_MAC

# Use specific attack type
sudo redfang -r TARGET_MAC -a 1

# Use specific interface
sudo redfang -r TARGET_MAC -i hci0
```

### Command-Line Options

| Flag | Description | Example |
|------|-------------|---------|
| `-r` | Target MAC address | `-r AA:BB:CC:DD:EE:FF` |
| `-s` | Sniff mode | `-s` |
| `-a` | Attack type (1-4) | `-a 1` |
| `-i` | Interface | `-i hci0` |
| `-v` | Verbose output | `-v` |
| `-h` | Help | `-h` |

### Quick Reference

```bash
# Basic PIN attack
sudo redfang -r AA:BB:CC:DD:EE:FF

# Sniff unencrypted traffic
sudo redfang -s AA:BB:CC:DD:EE:FF

# Attack with specific method
sudo redfang -r AA:BB:CC:DD:EE:FF -a 2
```

---

## Chapter 4: Intermediate Usage

### Capture Filters

```bash
# Capture only inquiry results
redfang -i hci0 -f "inquiry"

# Capture only ACL data
redfang -i hci0 -f "acl"

# Capture only SCO data
redfang -i hci0 -f "sco"

# Combine filters
redfang -i hci0 -f "inquiry || acl"
```

### Protocol-Specific Capture

```bash
# Capture L2CAP traffic
redfang -i hci0 -f "l2cap"

# Capture RFCOMM connections
redfang -i hci0 -f "rfcomm"

# Capture OBEX transfers
redfang -i hci0 -f "obex"
```

### Output Options

```bash
# Save capture to file
redfang -i hci0 -w capture.pcap

# Verbose output
redfang -i hci0 -v

# Hex dump
redfang -i hci0 -X
```

### Integration with Wireshark

```bash
# Capture for Wireshark analysis
redfang -i hci0 -w bt_capture.pcap

# Open in Wireshark
wireshark bt_capture.pcap
```

### Python Analysis

```python
#!/usr/bin/env python3
"""RedFang capture analyzer"""

import subprocess
import struct

def parse_capture(filename):
    """Parse redfang capture"""
    with open(filename, 'rb') as f:
        data = f.read()
    
    offset = 0
    packets = []
    while offset < len(data):
        if offset + 4 <= len(data):
            pkt_type = data[offset]
            pkt_len = struct.unpack('<H', data[offset+2:offset+4])[0]
            packets.append({
                'type': pkt_type,
                'length': pkt_len,
                'data': data[offset:offset+4+pkt_len]
            })
            offset += 4 + pkt_len
    
    return packets

packets = parse_capture('capture.pcap')
print(f"Parsed {len(packets)} packets")
```

---

## Chapter 5: Advanced Usage

### Protocol Analysis

```bash
# Analyze HCI commands
tshark -r capture.pcap -Y "bthci_cmd" -T fields -e bthci_cmd.opcode

# Analyze L2CAP
tshark -r capture.pcap -Y "btl2cap" -T fields -e btl2cap.psm

# Analyze RFCOMM
tshark -r capture.pcap -Y "btrfcomm" -T fields -e btrfcomm.channel
```

### Automated Capture and Analysis

```bash
#!/bin/bash
# bt_capture_analysis.sh - Automated capture and analysis

DURATION=300  # 5 minutes
OUTPUT="/tmp/bt_capture_$(date +%Y%m%d_%H%M%S).pcap"

echo "[*] Starting Bluetooth capture for ${DURATION}s"
timeout $DURATION redfang -i hci0 -w "$OUTPUT" 2>/dev/null

echo "[*] Analyzing capture..."
echo "=== Protocol Statistics ==="
tshark -r "$OUTPUT" -q -z io,phs 2>/dev/null

echo ""
echo "=== Device Communication Patterns ==="
tshark -r "$OUTPUT" -q -z conv,bluetooth 2>/dev/null
```

### Custom Analysis Scripts

```python
#!/usr/bin/env python3
"""bt_traffic_analyzer.py - Bluetooth traffic analysis"""

from scapy.all import *
from scapy.layers.bluetooth import *

def analyze_capture(pcap_file):
    """Analyze Bluetooth capture file"""
    packets = rdpcap(pcap_file)
    
    stats = {
        'total': 0,
        'hci_cmd': 0,
        'hci_acl': 0,
        'hci_sco': 0,
        'l2cap': 0,
        'rfcomm': 0
    }
    
    for pkt in packets:
        stats['total'] += 1
        
        if HCI_Cmd in pkt:
            stats['hci_cmd'] += 1
        elif HCI_ACL_Hdr in pkt:
            stats['hci_acl'] += 1
            if L2CAP_Hdr in pkt:
                stats['l2cap'] += 1
                if RFCOMM_Hdr in pkt:
                    stats['rfcomm'] += 1
        elif HCISCO_Hdr in pkt:
            stats['hci_sco'] += 1
    
    return stats

# Usage
stats = analyze_capture('capture.pcap')
print(f"Protocol Distribution:")
for proto, count in stats.items():
    print(f"  {proto}: {count}")
```

### Performance Optimization

```bash
# Increase buffer size
redfang -i hci0 -B 2M -w capture.pcap

# Disable capture filters (raw mode)
redfang -i hci0 -p -w raw_capture.pcap

# Memory-efficient mode
redfang -i hci0 -m -w lowmem.pcap
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: Corporate Traffic Monitoring

```bash
#!/bin/bash
# corporate_bt_monitor.sh - Monitor BT traffic in office

echo "[*] Starting corporate BT traffic monitoring"

# Capture BT traffic
timeout 3600 redfang -i hci0 -w /tmp/corporate_bt.pcap 2>/dev/null &

MONITOR_PID=$!

# Real-time alerts
while kill -0 $MONITOR_PID 2>/dev/null; do
    if [ -f /tmp/bt_current.pcap ]; then
        PACKETS=$(tshark -r /tmp/bt_current.pcap 2>/dev/null | wc -l)
        echo "[$(date +%H:%M:%S)] Packets captured: $PACKETS"
    fi
    sleep 60
done

echo "[+] Capture complete. Analyzing..."
tshark -r /tmp/corporate_bt.pcap -q -z io,phs
```

### Scenario 2: Bluetooth MITM Assessment

```bash
#!/bin/bash
# bt_mitm_test.sh - BT MITM assessment

TARGET_ADDR=$1

echo "=== BT MITM Assessment ==="
echo "Target: $TARGET_ADDR"

# Step 1: Baseline capture
echo "[1] Capturing baseline traffic..."
timeout 30 redfang -i hci0 -f "bd_addr == $TARGET_ADDR" -w baseline.pcap

# Step 2: Monitor connection attempts
echo "[2] Monitoring connection attempts..."
redfang -i hci0 -f "hci_evt.code == 0x03" -w connections.pcap &
MONITOR_PID=$!

sleep 60
kill $MONITOR_PID 2>/dev/null

# Step 3: Analyze
echo "[3] Analysis:"
echo "  Baseline packets: $(tshark -r baseline.pcap 2>/dev/null | wc -l)"
echo "  Connection events: $(tshark -r connections.pcap 2>/dev/null | wc -l)"
```

### Scenario 3: IoT Security Audit

```bash
#!/bin/bash
# iot_bt_audit.sh - IoT Bluetooth traffic analysis

echo "=== IoT BT Traffic Analysis ==="

# Capture IoT traffic
timeout 300 redfang -i hci0 -w iot_bt.pcap 2>/dev/null

# Analyze by device
echo "[*] Devices communicating:"
tshark -r iot_bt.pcap -T fields -e bluetooth.address 2>/dev/null | sort | uniq -c | sort -rn

# Check for insecure protocols
echo ""
echo "[*] Potential security issues:"
tshark -r iot_bt.pcap -Y "btl2cap.psm == 3 || btrfcomm" 2>/dev/null
```

---

## Chapter 7: Detection & Defense

### Detection Methods

```bash
# Check for redfang process
ps aux | grep redfang

# Monitor system calls
sudo strace -p $(pgrep redfang) -e trace=network

# Check for packet capture
lsof | grep -i "redfang\|pcap"

# Monitor for BT capture activity
btmon -t | grep -E "inquiry|scan|capture"
```

### Defensive Countermeasures

```bash
# Completely disable BT
sudo bluetoothctl power off
sudo systemctl stop bluetooth

# Blacklist kernel module
echo "blacklist btusb" | sudo tee /etc/modprobe.d/bluetooth-blacklist.conf

# Make non-discoverable
sudo bluetoothctl discoverable off

# Disable inquiry scan
sudo hciconfig hci0 noscan
```

### Monitoring Script

```bash
#!/bin/bash
# bt_capture_detector.sh - Detect BT capture attempts

LOG="/var/log/bt_capture_alerts.log"

while true; do
    # Check for new pcap files
    NEW_PCAPP=$(find /tmp -name "*.pcap" -mmin -5 2>/dev/null)
    
    if [ -n "$NEW_PCAPP" ]; then
        echo "[ALERT] $(date) - New pcap file detected: $NEW_PCAPP" >> "$LOG"
        
        for f in $NEW_PCAPP; do
            PID=$(lsof "$f" 2>/dev/null | awk 'NR>1 {print $2}')
            if [ -n "$PID" ]; then
                echo "  Process: $(ps -p $PID -o comm=)" >> "$LOG"
            fi
        done
    fi
    
    # Check for active captures
    if pgrep -x "redfang" > /dev/null; then
        echo "[ALERT] $(date) - redfang process detected" >> "$LOG"
    fi
    
    sleep 30
done
```

### Device Hardening

```bash
#!/bin/null
# bt_hardening.sh - Bluetooth security hardening

# 1. Non-discoverable mode
sudo bluetoothctl discoverable off

# 2. Disable pairable
sudo bluetoothctl pairable off

# 3. Set secure connections only
sudo btmgmt ssp on

# 4. Disable legacy pairing
sudo hcitool cmd 0x3F 0x001A 0x00

# 5. Enable encryption
sudo btmgmt ssp on
```

---

## Chapter 8: Troubleshooting & Resources

### Common Errors

#### "No Bluetooth adapter found"
```bash
hciconfig -a
sudo modprobe -r btusb
sudo modprobe btusb
```

#### "Permission denied"
```bash
sudo redfang -i hci0
sudo setcap cap_net_raw,cap_net_admin+eip $(which redfang)
```

#### "Device not ready"
```bash
sleep 5
sudo hciconfig hci0 reset
dmesg | grep -i bluetooth
```

#### "Capture failed"
```bash
df -h /tmp
redfang -i hci0 -B 128K -w capture.pcap
redfang -i hci1 -w capture.pcap
```

### Performance Issues

#### Low Capture Rate
```bash
redfang -i hci0 -B 2M -w capture.pcap
redfang -i hci0 -p -w raw.pcap
redfang -i hci0 -w /dev/shm/capture.pcap
```

#### High CPU Usage
```bash
redfang -i hci0 -q -w capture.pcap
timeout 60 redfang -i hci0 -w capture.pcap
```

### FAQ

#### Q: Can't capture any packets
```bash
sudo hciconfig hci0
sudo hciconfig hci0 up
# Try different adapter
```

#### Q: Capture file is empty
```bash
# Ensure BT devices are active nearby
# Check adapter range
# Verify filter syntax
```

#### Q: How to capture BLE traffic?
```bash
redfang -i hci0 -f "type == 0x04" -w ble_capture.pcap
```

#### Q: Can't read capture in Wireshark
```bash
editcap -F pcap capture.pcap fixed.pcap
# Ensure Wireshark has BT plugins
```

### Resources

- [RedFang GitHub](https://github.com/hchoices/redfang)
- [Kali Linux Tools](https://www.kali.org/tools/redfang/)
- [BlueZ Documentation](http://www.bluez.org/)
- [Bluetooth SIG](https://www.bluetooth.com/)
- [Wireshark BT Dissection](https://www.wireshark.org/tools/)

---

## Resources

| Resource | URL |
|----------|-----|
| Official Repository | https://github.com/hchoices/redfang |
| Kali Tools Page | https://www.kali.org/tools/redfang/ |
| BlueZ Documentation | http://www.bluez.org/ |
| Bluetooth SIG | https://www.bluetooth.com/ |
| Wireshark | https://www.wireshark.org/ |
| MITRE ATT&CK | https://attack.mitre.org/techniques/T1557/ |
