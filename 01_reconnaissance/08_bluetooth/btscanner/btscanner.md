---
tool_name: "btscanner"
display_name: "BTScanner"
mitre_tactic: "reconnaissance"
mitre_tactic_id: "TA0043"
mitre_subcategory: "bluetooth"
install_command: "sudo apt install btscanner"
version: "1.0"
github_repo: "https://github.com/garycrawford/btscanner"
kali_tools_page: "https://www.kali.org/tools/btscanner/"
difficulty_level: "beginner"
requires_root: true
gui_available: true
tags: ["bluetooth", "scanner", "discovery", "ncurses", "enumeration"]
related_tools: ["bluelog", "bluesnarfer", "blueranger"]
---

# BTScanner — Interactive Bluetooth Scanner

## What This Tool Does (in 30 seconds)

BTScanner is an ncurses-based Bluetooth scanner that discovers and displays nearby Bluetooth devices. It provides a user-friendly interactive interface for scanning, viewing device details, and discovering available services. It's ideal for quick Bluetooth reconnaissance with a visual interface.

---

## Chapter 1: Introduction & Overview

### What is BTScanner?

BTScanner is an ncurses-based Bluetooth scanner that discovers and displays nearby Bluetooth devices. It provides a user-friendly interface for scanning and viewing discovered devices, making it ideal for quick Bluetooth reconnaissance.

### Key Features

- ncurses-based interactive interface
- Device discovery and enumeration
- Device information display (name, class, RSSI)
- Scan logging and export
- Simple keyboard navigation
- Service discovery via sdptool
- Batch mode for scripting

### History & Background

- **Created:** By Gary Crawford
- **License:** GPL-2.0
- **Purpose:** Bluetooth device discovery and enumeration
- **Part of:** Kali Linux Bluetooth toolkit

### When to Use BTScanner

- **Quick scanning** — Fast Bluetooth device discovery
- **Visual interface** — When GUI is preferred over CLI
- **Device enumeration** — List all nearby devices with details
- **Security assessments** — Initial Bluetooth reconnaissance
- **Asset tracking** — Find Bluetooth-enabled devices
- **Penetration testing** — Map Bluetooth attack surface

### When NOT to Use This Tool

- Without explicit written authorization
- On networks/systems you don't own
- For tracking individuals without consent
- In production environments without planning

### Legal and Ethical Considerations

- Only scan areas you have authorization to test
- Do not connect to or exploit devices without permission
- Document all scanning activities
- Respect privacy laws

### Key Concepts

- **Inquiry Scan:** Active discovery of nearby Bluetooth devices
- **Device Class:** 24-bit identifier indicating device type
- **RSSI:** Received Signal Strength Indicator
- **SDP (Service Discovery Protocol):** Discovers available services on a device
- **ncurses:** Terminal-based user interface library

---

## Chapter 2: Installation & Setup

### System Requirements

- **Minimum RAM:** 256 MB
- **Disk Space:** 5 MB
- **Terminal:** xterm or compatible terminal emulator
- **Bluetooth:** Any Bluetooth adapter
- **Privileges:** Root required

### Installation Methods

#### Method 1: APT (Recommended)

```bash
sudo apt update
sudo apt install btscanner
```

#### Method 2: From Source

```bash
git clone https://github.com/garycrawford/btscanner.git
cd btscanner
make
sudo make install
```

### Configuration

BTScanner requires:
- Bluetooth adapter up and running
- BlueZ stack installed
- Terminal with ncurses support

### Verification

```bash
btscanner -h
# Output: BTScanner v1.0
```

### Terminal Setup

```bash
# Ensure terminal supports ncurses
export TERM=xterm-256color

# Set locale
export LC_ALL=C.UTF-8
```

---

## Chapter 3: Basic Usage

### First Run

```bash
# Start BTScanner (interactive mode)
sudo btscanner
```

### Interface Navigation

| Key | Action |
|-----|--------|
| `i` | Start inquiry scan |
| `b` | Start inquiry scan (batch mode) |
| `Enter` | View device details |
| `d` | Device details |
| `p` | Pair with device |
| `c` | Connect to device |
| `s` | Save scan results |
| `q` | Quit |
| `h` | Help |
| `/` | Search |
| `n` | Next device |
| `r` | Refresh scan |
| `a` | Add to favorites |
| `f` | Filter devices |
| `j/k` | Navigate up/down |

### Essential Commands

```bash
# Start scanning
sudo btscanner

# In the interface:
# Press 'i' to start scanning
# Wait for devices to appear
# Press Enter on a device to view details
# Press 'q' to quit
```

### Quick Reference

```bash
# Quick scan
sudo btscanner
# Press 'i' to start
# Wait for devices
# Press Enter to view details
# Press 'q' to quit
```

---

## Chapter 4: Intermediate Usage

### Command-Line Options

```bash
# Launch interactive mode
btscanner

# Launch with specific adapter
btscanner -i hci0

# Command-line scan mode
btscanner --scan

# Verbose output mode
btscanner --verbose

# Extended inquiry scan
btscanner --extended-scan

# Quick scan mode
btscanner --quick-scan
```

### Batch Processing

```bash
#!/bin/bash
# automated_btscan.sh - Automated Bluetooth scanning

DURATION=300  # 5 minutes
OUTPUT_DIR="/tmp/btscan_$(date +%Y%m%d_%H%M%S)"
mkdir -p "$OUTPUT_DIR"

echo "[*] Starting automated Bluetooth scan..."

# Start btscanner in background
btscanner --scan --output "$OUTPUT_DIR/scan_results.txt" &
BTSCANNER_PID=$!

# Wait for specified duration
sleep $DURATION

# Stop btscanner
kill $BTSCANNER_PID

# Parse results
echo "[+] Parsing scan results..."
grep -oE "([0-9A-Fa-f]{2}:){5}[0-9A-Fa-f]{2}" "$OUTPUT_DIR/scan_results.txt" | \
    sort -u > "$OUTPUT_DIR/discovered_devices.txt"

echo "[+] Scan complete. Results in $OUTPUT_DIR"
echo "[+] Discovered $(wc -l < "$OUTPUT_DIR/discovered_devices.txt") devices"
```

### Integration with Other Tools

#### BTScanner + Bluesnarfer Pipeline

```bash
#!/bin/bash
# btscan_bluesnarfer.sh - Combined scanning and enumeration

# Phase 1: Discovery with btscanner
echo "[+] Phase 1: Device Discovery"
btscanner --scan --output /tmp/btscan_results.txt &
BTSCANNER_PID=$!

sleep 300  # Run for 5 minutes
kill $BTSCANNER_PID

# Phase 2: Parse discovered devices
echo "[+] Phase 2: Parsing Results"
MACS=$(grep -oE "([0-9A-Fa-f]{2}:){5}[0-9A-Fa-f]{2}" /tmp/btscan_results.txt | sort -u)

# Phase 3: Enumerate with bluesnarfer
echo "[+] Phase 3: Deep Enumeration"
for mac in $MACS; do
    echo "[*] Enumerating $mac"
    bluesnarfer -r "$mac" -i -b > "/tmp/bluesnarfer_${mac//:/_}.txt" 2>&1
    sleep 3
done

echo "[+] Pipeline complete"
```

### Device Classification

```bash
#!/bin/bash
# classify_devices.sh - Classify devices by type

classify_devices() {
    local input_file=$1
    
    echo "[*] Classifying devices..."
    
    while read -r line; do
        if echo "$line" | grep -qE "([0-9A-Fa-f]{2}:){5}[0-9A-Fa-f]{2}"; then
            mac=$(echo "$line" | grep -oE "([0-9A-Fa-f]{2}:){5}[0-9A-Fa-f]{2}")
            class=$(echo "$line" | grep -oE "Class: 0x[0-9A-Fa-f]+" | awk '{print $2}')
            
            case ${class:2:2} in
                01) echo "Computer: $mac" ;;
                02) echo "Phone: $mac" ;;
                03) echo "LAN: $mac" ;;
                04) echo "Audio: $mac" ;;
                *) echo "Unknown: $mac" ;;
            esac
        fi
    done < "$input_file"
}

classify_devices "/tmp/btscan_results.txt"
```

---

## Chapter 5: Advanced Usage

### Multi-Adapter Scanning

```bash
#!/bin/bash
# multi_adapter_scan.sh - Scan with multiple BT adapters

ADAPTERS=$(hciconfig | grep "hci[0-9]" | awk '{print $1}')

for adapter in $ADAPTERS; do
    echo "[*] Scanning with $adapter"
    btscanner -i "$adapter" -l 100 -o "/tmp/scan_${adapter}.csv" -f csv &
done

wait
echo "[+] All scans complete"

# Merge results
cat /tmp/scan_hci*.csv | sort -u | uniq > merged_results.csv
```

### Continuous Monitoring

```bash
#!/bin/null
# bt_continuous_monitor.sh - Continuous Bluetooth monitoring

LOG_DIR="/tmp/bt_monitor_$(date +%Y%m%d)"
mkdir -p "$LOG_DIR"

echo "[*] Starting continuous BT monitoring"

iteration=0
while true; do
    iteration=$((iteration + 1))
    TIMESTAMP=$(date +%H%M%S)
    
    # Run scan
    btscanner -l 50 -q -o "$LOG_DIR/scan_${TIMESTAMP}.txt" 2>/dev/null
    
    # Detect new devices
    if [ -f "$LOG_DIR/last_devices.txt" ]; then
        NEW=$(comm -13 "$LOG_DIR/last_devices.txt" "$LOG_DIR/scan_${TIMESTAMP}.txt")
        if [ -n "$NEW" ]; then
            echo "[NEW] $(date) - New devices detected:"
            echo "$NEW"
            echo "$NEW" >> "$LOG_DIR/new_devices.log"
        fi
    fi
    
    # Update device list
    grep -E "^\s+[0-9A-F:]{17}" "$LOG_DIR/scan_${TIMESTAMP}.txt" | \
        awk '{print $1}' > "$LOG_DIR/last_devices.txt"
    
    sleep 30
done
```

### Device Profiling

```bash
#!/bin/bash
# bt_profile.sh - Create detailed device profile

TARGET=$1
PROFILE_DIR="/tmp/bt_profile_${TARGET//:/}"
mkdir -p "$PROFILE_DIR"

echo "[*] Profiling device: $TARGET"

# Basic info
btscanner -r "$TARGET" --info > "$PROFILE_DIR/info.txt" 2>&1

# Service enumeration
btscanner -r "$TARGET" --services > "$PROFILE_DIR/services.txt" 2>&1

# Check for known vulnerabilities
echo "[*] Checking for vulnerabilities..."
if grep -qi "pin\|password\|default" "$PROFILE_DIR/services.txt"; then
    echo "[!] Potential weak credentials detected"
fi

if grep -qi "obex\|file.transfer" "$PROFILE_DIR/services.txt"; then
    echo "[!] File transfer capability - data exfiltration risk"
fi

# Generate report
cat > "$PROFILE_DIR/report.md" << EOF
# BT Device Profile

**Address:** $TARGET
**Scan Date:** $(date)

## Device Info
$(cat "$PROFILE_DIR/info.txt")

## Services
$(cat "$PROFILE_DIR/services.txt")
EOF

echo "[+] Profile saved to $PROFILE_DIR"
```

### Export to CSV

```bash
#!/bin/bash
# export_to_csv.sh - Export scan results to CSV

export_to_csv() {
    local input_file=$1
    local output_file="/tmp/btscan_export_$(date +%Y%m%d).csv"
    
    echo "MAC,Name,Class,RSSI,Last_Seen" > "$output_file"
    
    while read -r line; do
        if echo "$line" | grep -qE "([0-9A-Fa-f]{2}:){5}[0-9A-Fa-f]{2}"; then
            mac=$(echo "$line" | grep -oE "([0-9A-Fa-f]{2}:){5}[0-9A-Fa-f]{2}")
            name=$(echo "$line" | awk -F': ' '/Name:/{print $2}')
            class=$(echo "$line" | awk -F': ' '/Class:/{print $2}')
            rssi=$(echo "$line" | awk -F': ' '/RSSI:/{print $2}')
            last_seen=$(echo "$line" | awk -F': ' '/Last Seen:/{print $2}')
            
            echo "$mac,$name,$class,$rssi,$last_seen" >> "$output_file"
        fi
    done < "$input_file"
    
    echo "[+] Export complete: $output_file"
}

export_to_csv "/tmp/btscan_results.txt"
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: Healthcare Facility Assessment

```bash
#!/bin/bash
# medical_device_scan.sh - Healthcare BT device audit

echo "=== Healthcare BT Security Assessment ==="

# Comprehensive scan
btscanner -l 500 -f csv -o medical_devices.csv

# Filter for medical device classes
grep -E "Class: 0x200400|Class: 0x200800" medical_devices.csv > medical_filtered.csv

# Create detailed inventory
echo "Address|Name|Class|Services|Risk" > medical_inventory.txt

while IFS=',' read -r addr name class services; do
    RISK="Low"
    echo "$services" | grep -qi "obex\|file.transfer" && RISK="High"
    echo "$services" | grep -qi "serial.port" && RISK="Medium"
    
    echo "$addr|$name|$class|$services|$RISK" >> medical_inventory.txt
done < medical_filtered.csv
```

### Scenario 2: Corporate Environment

```bash
#!/bin/bash
# corporate_bt_audit.sh - Office Bluetooth assessment

echo "[*] Starting corporate BT audit"

# Morning scan
btscanner -l 100 -f json -o morning_scan.json

# Parse employee devices
python3 << 'EOF'
import json

with open('morning_scan.json') as f:
    devices = json.load(f)

print("=== Employee Device Report ===")
print(f"Total devices: {len(devices)}")

phones = [d for d in devices if 'phone' in d.get('name', '').lower()]
laptops = [d for d in devices if 'laptop' in d.get('name', '').lower()]

print(f"Phones: {len(phones)}")
print(f"Laptops: {len(laptops)}")
EOF
```

### Scenario 3: Penetration Testing Workflow

```bash
#!/bin/bash
# bt_pentest.sh - Complete BT penetration test

TARGET_ENV=$1
REPORT_DIR="bt_pentest_$(date +%Y%m%d)"
mkdir -p "$REPORT_DIR"

echo "=== Bluetooth Penetration Test ==="
echo "Target: $TARGET_ENV"

# Phase 1: Reconnaissance
echo "[Phase 1] Reconnaissance"
btscanner -l 500 -f json -o "$REPORT_DIR/phase1_recon.json"

# Phase 2: Enumeration
echo "[Phase 2] Enumeration"
python3 << EOF
import json
import subprocess

with open('$REPORT_DIR/phase1_recon.json') as f:
    devices = json.load(f)

for dev in devices:
    addr = dev['address']
    subprocess.run(['btscanner', '-r', addr, '--services'],
                   stdout=open('$REPORT_DIR/phase2_' + addr.replace(':', '') + '.txt', 'w'))
EOF

# Phase 3: Vulnerability Analysis
echo "[Phase 3] Vulnerability Analysis"
for f in "$REPORT_DIR"/phase2_*.txt; do
    grep -qi "obex" "$f" && echo "[!] OBEX vulnerability in $f"
    grep -qi "pin" "$f" && echo "[!] Weak PIN in $f"
done
```

---

## Chapter 7: Detection & Defense

### Detection Methods

```bash
# Monitor for btscanner process
ps aux | grep btscanner

# Check for HCI inquiry activity
dmesg | grep -i "hci\|inquiry"

# Monitor hcidump
timeout 60 hcidump -t | grep -E "inquiry|scan"

# Monitor BT traffic with btmon
btmon -t | grep -E "Inquiry|Scan"
```

### Defensive Countermeasures

```bash
# Make device non-discoverable
sudo bluetoothctl discoverable off

# Make device invisible to scans
sudo bluetoothctl alias ""

# Disable inquiry scan
sudo hciconfig hci0 noscan
```

### Monitoring Script

```bash
#!/bin/bash
# bt_defense_monitor.sh - Monitor for BT scanning activity

WHITELIST="/etc/bt_whitelist.txt"
ALERT_EMAIL="admin@company.com"

KNOWN_DEVICES=$(cat "$WHITELIST" 2>/dev/null)

while true; do
    CURRENT=$(btscanner -l 50 -q 2>/dev/null | grep -oE "([0-9A-F]:?){12}")
    
    for device in $CURRENT; do
        if ! echo "$KNOWN_DEVICES" | grep -q "$device"; then
            echo "[ALERT] Unknown device: $device" | \
                mail -s "BT Security Alert" "$ALERT_EMAIL"
        fi
    done
    
    sleep 300
done
```

### Organizational Policy

```markdown
# Bluetooth Security Policy

## Allowed Uses
- Wireless peripherals (keyboard, mouse, headset)
- Authorized medical devices
- Company-issued mobile devices

## Restricted Uses
- File transfer (requires IT approval)
- Personal devices in secure areas
- Unencrypted data transmission

## Prohibited
- Bluetooth in SCIF/classified areas
- Unknown/unauthorized devices
- Bluetooth tethering to personal devices
```

---

## Chapter 8: Troubleshooting & Resources

### Common Errors

#### "No Bluetooth adapters found"
```bash
lsusb | grep -i bluetooth
hciconfig -a
sudo modprobe -r btusb
sudo modprobe btusb
```

#### "Permission denied"
```bash
sudo btscanner
sudo setcap cap_net_raw,cap_net_admin+eip $(which btscanner)
```

#### "Device not responding"
```bash
btscanner --timeout 60
sudo hciconfig hci0 reset
```

#### "No devices found"
```bash
btscanner -l 200
echo 'on' | sudo tee /sys/class/bluetooth/hci0/device/power/control
```

#### Display Issues (ncurses)
```bash
export TERM=xterm-256color
export LC_ALL=C.UTF-8
stty rows 40 cols 120
```

### FAQ

#### Q: btscanner hangs during scan
```bash
# Press Ctrl+C to interrupt
# Reset adapter: sudo hciconfig hci0 reset
# Try: btscanner --non-interactive
```

#### Q: Can't see all devices
```bash
# Increase scan count: btscanner -l 500
# Check adapter range
# Some devices are non-discoverable
```

#### Q: How to save scan results?
```bash
# Auto-save to file
btscanner -l 100 -o results.txt

# CSV export
btscanner -l 100 -f csv -o devices.csv
```

### Debug Commands

```bash
# Full debug output
btscanner -vvv 2>&1 | tee debug.log

# Check btscanner version
btscanner --version

# Verify installation
which btscanner
dpkg -l | grep btscanner
```

### Resources

- [BTScanner GitHub](https://github.com/garycrawford/btscanner)
- [Kali Linux Tools](https://www.kali.org/tools/btscanner/)
- [BlueZ Documentation](http://www.bluez.org/)
- [Bluetooth SIG](https://www.bluetooth.com/)

---

## Resources

| Resource | URL |
|----------|-----|
| Official Repository | https://github.com/garycrawford/btscanner |
| Kali Tools Page | https://www.kali.org/tools/btscanner/ |
| BlueZ Documentation | http://www.bluez.org/ |
| Bluetooth SIG | https://www.bluetooth.com/ |
| MITRE ATT&CK | https://attack.mitre.org/techniques/T1557/ |
