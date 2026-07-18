---
tool_name: "spooftooph"
display_name: "Spooftooph"
mitre_tactic: "reconnaissance"
mitre_tactic_id: "TA0043"
mitre_subcategory: "bluetooth"
install_command: "sudo apt install spooftooph"
version: "1.0"
github_repo: "https://github.com/securestate/spooftooph"
kali_tools_page: "https://www.kali.org/tools/spooftooph/"
difficulty_level: "intermediate"
requires_root: true
gui_available: false
tags: ["bluetooth", "spoofing", "bluetooth-address", "identity", "mac-spoofing"]
related_tools: ["bluesnarfer", "redfang", "btscanner"]
---

# Spooftooph — Bluetooth Address Spoofer

## What This Tool Does (in 30 seconds)

Spooftooph spoofs Bluetooth device addresses (BD_ADDR) and names. It can impersonate other Bluetooth devices to test security mechanisms and access controls. Useful for testing whether Bluetooth access controls rely solely on device identity.

---

## Chapter 1: Introduction & Overview

### What is Spooftooph?

Spooftooph spoofs Bluetooth device addresses (BD_ADDR) and names. It can impersonate other Bluetooth devices to test security mechanisms and access controls. By changing your Bluetooth identity, you can test whether systems trust devices based solely on their MAC address or name.

### Key Features

- Bluetooth address spoofing
- Device name spoofing
- Device class spoofing
- Multiple spoofing modes
- Random address generation
- EIR (Extended Inquiry Response) manipulation
- Restore original identity

### History & Background

- **Created:** By SecureState
- **License:** GPL-3.0
- **Purpose:** Bluetooth security testing and identity manipulation
- **Part of:** Kali Linux Bluetooth toolkit

### When to Use Spooftooph

- **Security testing** — Test Bluetooth access controls
- **Penetration testing** — Impersonate trusted devices
- **Research** — Study Bluetooth security mechanisms
- **Compliance testing** — Verify security policies
- **Red team operations** — Social engineering via Bluetooth
- **Privacy protection** — Evade Bluetooth tracking

### When NOT to Use This Tool

- Without explicit written authorization
- To impersonate devices you don't own
- For unauthorized access to systems or data
- For stalking or harassment

### Legal and Ethical Considerations

- Only spoof addresses on devices you own or have authorization
- Unauthorized impersonation is illegal in most jurisdictions
- Use for legitimate security testing only
- Document all spoofing activities
- Some jurisdictions have specific laws against identity spoofing

### Key Concepts

- **BD_ADDR:** Bluetooth Device Address — unique MAC address for each adapter
- **EIR (Extended Inquiry Response):** Additional data broadcast during inquiry
- **Device Class:** 24-bit identifier indicating device type
- **Identity Spoofing:** Changing your Bluetooth identity to impersonate another device
- **MAC Spoofing:** Changing the hardware address of a network interface

---

## Chapter 2: Installation & Setup

### System Requirements

- **Minimum RAM:** 256 MB
- **Disk Space:** 5 MB
- **Bluetooth:** Adapter that supports address spoofing
- **Privileges:** Root required

### Installation Methods

#### Method 1: APT (Recommended)

```bash
sudo apt update
sudo apt install spooftooph
```

#### Method 2: From Source

```bash
git clone https://github.com/securestate/spooftooph.git
cd spooftooph
make
sudo make install
```

### Configuration

Spooftooph requires:
- Bluetooth adapter up and running
- Adapter must support address spoofing (most do)
- BlueZ stack installed

### Verification

```bash
spooftooph -h
# Output: Spooftooph v1.0
```

### Adapter Compatibility Check

```bash
# Check if adapter supports spoofing
hciconfig hci0 features | grep -i "features"

# Check current adapter state
hciconfig hci0
```

---

## Chapter 3: Basic Usage

### Essential Commands

```bash
# Spoof to random address
sudo spooftooph -i hci0 -r

# Spoof to specific address
sudo spooftooph -i hci0 -s TARGET_MAC

# Spoof with custom name
sudo spooftooph -i hci0 -s TARGET_MAC -n "New Name"

# Restore original address
sudo spooftooph -i hci0 -o
```

### Command-Line Options

| Flag | Description | Example |
|------|-------------|---------|
| `-i` | Interface | `-i hci0` |
| `-s` | Target MAC address | `-s AA:BB:CC:DD:EE:FF` |
| `-n` | Device name | `-n "Test Device"` |
| `-r` | Random address | `-r` |
| `-o` | Restore original | `-o` |
| `-c` | Device class | `-c 0x040404` |
| `-h` | Help | `-h` |

### Quick Reference

```bash
# Spoof to random address
sudo spooftooph -i hci0 -r

# Spoof to specific device
sudo spooftooph -i hci0 -s AA:BB:CC:DD:EE:FF -n "Spoofed"

# Restore original
sudo spooftooph -i hci0 -o
```

---

## Chapter 4: Intermediate Usage

### Identity Manipulation

```bash
# Change device name
sudo spooftooph -i hci0 -n "MyDevice"

# Change device address (BD_ADDR)
sudo spooftooph -i hci0 -a AA:BB:CC:DD:EE:FF

# Change device class
sudo spooftooph -i hci0 -c 0x200400  # Phone class
sudo spooftooph -i hci0 -c 0x040404  # Computer class

# Change all at once
sudo spooftooph -i hci0 -n "OfficePC" -a 11:22:33:44:55:66 -c 0x040404
```

### EIR Data Spoofing

```bash
# Set manufacturer-specific EIR data
sudo spooftooph -i hci0 -e "FF01020304"

# Set service UUID in EIR
sudo spooftooph -i hci0 --uuid 0x1101  # SPP

# Clear all EIR data
sudo spooftooph -i hci0 --clear-eir
```

### Random Identity Generation

```bash
# Generate random MAC
RANDOM_MAC=$(printf "AA:%02X:%02X:%02X:%02X:%02X" $((RANDOM%256)) $((RANDOM%256)) $((RANDOM%256)) $((RANDOM%256)) $((RANDOM%256)))
sudo spooftooph -i hci0 -a "$RANDOM_MAC"

# Random name
RANDOM_NAME="Device_$(cat /dev/urandom | tr -dc 'a-zA-Z0-9' | head -c 8)"
sudo spooftooph -i hci0 -n "$RANDOM_NAME"
```

### Batch Operations

```bash
#!/bin/bash
# bt_profile_switcher.sh - Switch between spoofed identities

PROFILES=(
    "OfficeLaptop|AA:BB:CC:DD:EE:FF|0x040404"
    "iPhone12|11:22:33:44:55:66|0x200400"
    "BTSpeaker|22:33:44:55:66:77|0x080404"
)

for profile in "${PROFILES[@]}"; do
    IFS='|' read -r name mac class <<< "$profile"
    echo "[*] Switching to profile: $name"
    sudo spooftooph -i hci0 -n "$name" -a "$mac" -c "$class"
    sleep 5
done
```

### Verification

```bash
# Check current identity
hciconfig hci0

# Scan to verify
btscanner -l 10

# Check EIR data
sudo hcitool cmd 0x01 0x0005
```

---

## Chapter 5: Advanced Usage

### Identity Management Framework

```python
#!/usr/bin/env python3
"""bt_identity_manager.py - Bluetooth identity management"""

import subprocess
import random
import json
import os

class BTIdentityManager:
    def __init__(self, adapter="hci0"):
        self.adapter = adapter
        self.profiles_file = os.path.expanduser("~/.bt_profiles.json")
        self.load_profiles()
    
    def load_profiles(self):
        if os.path.exists(self.profiles_file):
            with open(self.profiles_file) as f:
                self.profiles = json.load(f)
        else:
            self.profiles = {}
    
    def save_profiles(self):
        with open(self.profiles_file, 'w') as f:
            json.dump(self.profiles, f, indent=2)
    
    def generate_random_mac(self):
        return "AA:{:02X}:{:02X}:{:02X}:{:02X}:{:02X}".format(
            random.randint(0, 255), random.randint(0, 255),
            random.randint(0, 255), random.randint(0, 255),
            random.randint(0, 255)
        )
    
    def create_profile(self, name, mac=None, device_class="0x040404"):
        if mac is None:
            mac = self.generate_random_mac()
        
        self.profiles[name] = {
            "mac": mac,
            "class": device_class,
            "created": str(__import__('datetime').datetime.now())
        }
        self.save_profiles()
        return self.profiles[name]
    
    def apply_profile(self, name):
        if name not in self.profiles:
            raise ValueError(f"Profile '{name}' not found")
        
        profile = self.profiles[name]
        cmd = [
            "sudo", "spooftooph",
            "-i", self.adapter,
            "-a", profile["mac"],
            "-c", profile["class"]
        ]
        
        subprocess.run(cmd, check=True)
        return True
    
    def list_profiles(self):
        return list(self.profiles.keys())

# Usage
mgr = BTIdentityManager()
mgr.create_profile("office", device_class="0x040404")
mgr.create_profile("phone", device_class="0x200400")
mgr.apply_profile("office")
```

### Automated Identity Rotation

```bash
#!/bin/null
# bt_identity_rotator.sh - Rotate Bluetooth identity

while true; do
    # Generate random identity
    RANDOM_MAC=$(printf "AA:%02X:%02X:%02X:%02X:%02X" $((RANDOM%256)) $((RANDOM%256)) $((RANDOM%256)) $((RANDOM%256)) $((RANDOM%256)))
    RANDOM_NAME="Dev_$(cat /dev/urandom | tr -dc 'a-zA-Z0-9' | head -c 6)"
    
    sudo spooftooph -i hci0 -a "$RANDOM_MAC" -n "$RANDOM_NAME"
    echo "[$(date +%H:%M:%S)] Identity: $RANDOM_NAME ($RANDOM_MAC)"
    
    sleep 300  # Rotate every 5 minutes
done
```

### Multi-Adapter Spoofing

```bash
#!/bin/bash
# multi_adapter_spoof.sh - Spoof all Bluetooth adapters

ADAPTERS=$(hciconfig | grep "hci[0-9]" | awk '{print $1}' | tr '\n' ' ')

for adapter in $ADAPTERS; do
    MAC=$(printf "AA:%02X:%02X:%02X:%02X:%02X" \
        $((RANDOM%256)) $((RANDOM%256)) $((RANDOM%256)) \
        $((RANDOM%256)) $((RANDOM%256)))
    
    sudo spooftooph -i "$adapter" -a "$MAC" -n "Device_$adapter"
    echo "[+] $adapter spoofed to $MAC"
done
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: Privacy Protection

```bash
#!/bin/null
# bt_privacy.sh - Evade BT location tracking

rotate_identity() {
    # Random phone-like identity
    NAMES=("iPhone" "Galaxy" "Pixel" "OnePlus")
    NAME=${NAMES[$RANDOM % ${#NAMES[@]}]}
    MODEL=$((RANDOM % 99 + 1))
    
    MAC=$(printf "%02X:%02X:%02X:%02X:%02X:%02X" \
        $((RANDOM%256)) $((RANDOM%256)) $((RANDOM%256)) \
        $((RANDOM%256)) $((RANDOM%256)) $((RANDOM%256)))
    
    sudo spooftooph -i hci0 -n "${NAME}${MODEL}" -a "$MAC" -c 0x200400
    echo "[$(date +%T)] Identity: ${NAME}${MODEL}"
}

# Rotate every 5 minutes
while true; do
    rotate_identity
    sleep 300
done
```

### Scenario 2: Red Team BT Operations

```bash
#!/bin/bash
# redteam_bt_spoof.sh - Corporate BT red team operation

echo "=== BT Red Team Operation ==="

# Step 1: Reconnaissance
echo "[1] Discovering target devices..."
btscanner -l 100 -q > /tmp/bt_recon.txt

# Step 2: Identify high-value targets
echo "[2] Identifying targets..."
grep -i "ceo\|finance\|hr\|admin" /tmp/bt_recon.txt > /tmp/hvt_targets.txt

# Step 3: Clone target identity
echo "[3] Cloning device identity..."
TARGET=$(head -1 /tmp/hvt_targets.txt | grep -oE "([0-9A-F]:?){12}")

if [ -n "$TARGET" ]; then
    bluesnarfer -r "$TARGET" -s > /tmp/target_info.txt
    NAME=$(grep "Name:" /tmp/target_info.txt | awk '{print $2}')
    
    sudo spooftooph -i hci0 -n "$NAME"
    echo "[+] Spoofed as: $NAME"
fi
```

### Scenario 3: Social Engineering

```bash
#!/bin/bash
# fake_device_attack.sh - Create fake Bluetooth device

DEVICE_TYPE=$1

case $DEVICE_TYPE in
    "printer")
        sudo spooftooph -i hci0 -n "HP_LaserJet_Pro" -c 0x040404
        ;;
    "speaker")
        sudo spooftooph -i hci0 -n "JBL Flip 5" -c 0x080404
        ;;
    "keyboard")
        sudo spooftooph -i hci0 -n "Logitech_K380" -c 0x000404
        ;;
    *)
        sudo spooftooph -i hci0 -n "BT_Device" -c 0x040404
        ;;
esac

echo "[+] Spoofed as $DEVICE_TYPE"
```

---

## Chapter 7: Detection & Defense

### Detection Methods

```bash
#!/bin/null
# bt_spoof_detector.sh - Detect Bluetooth spoofing

# Check for duplicate MACs
check_duplicates() {
    btscanner -l 100 -q 2>/dev/null | \
        grep -oE "([0-9A-F]:?){12}" | \
        sort | uniq -d
}

# Check for name/MAC mismatch
check_mismatch() {
    btscanner -l 100 -q 2>/dev/null | \
        awk '/BD_ADDR:/{addr=$2} /Name:/{if($2!=prev_name && prev_name!="") print addr, $2; prev_name=$2}'
}

# Check for random MAC patterns
check_random_mac() {
    btscanner -l 100 -q 2>/dev/null | \
        grep -oE "([0-9A-F]:?){12}" | \
        grep -E "^(AA|BB|CC|DD|EE|FF):" | \
        head -10
}

echo "[*] Checking for duplicates..."
DUPES=$(check_duplicates)
if [ -n "$DUPES" ]; then
    echo "[!] Duplicate MACs found:"
    echo "$DUPES"
fi
```

### Defensive Measures

```bash
#!/bin/null
# bt_anti_spoof.sh - Prevent Bluetooth spoofing

# Enable authentication
sudo btmgmt ssp on
sudo btmgmt io-capability 0x01

# Disable legacy pairing
sudo hcitool cmd 0x3F 0x001A 0x00

# Enable secure connections only
sudo btmgmt sc on
```

### Device Whitelisting

```bash
#!/bin/null
# bt_whitelist.sh - Monitor for unauthorized devices

WHITELIST="/etc/bt_whitelist.txt"

is_authorized() {
    local mac=$1
    grep -qi "$mac" "$WHITELIST" 2>/dev/null
}

while true; do
    btscanner -l 50 -q 2>/dev/null | grep -oE "([0-9A-F]:?){12}" | while read -r mac; do
        if ! is_authorized "$mac"; then
            echo "[ALERT] Unauthorized device: $mac"
            logger -t bt_security "Unauthorized BT device: $mac"
        fi
    done
    sleep 60
done
```

### Security Hardening

```bash
# Disable BT when not needed
sudo bluetoothctl power off

# Set non-discoverable
sudo bluetoothctl discoverable off

# Disable pairable mode
sudo bluetoothctl pairable off

# Enable secure connections only
sudo btmgmt sc on
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
sudo spooftooph -i hci0 -n "MyDevice"
sudo setcap cap_net_raw,cap_net_admin+eip $(which spooftooph)
```

#### "Invalid BD_ADDR format"
```bash
# Correct format (colon-separated)
spooftooph -i hci0 -a AA:BB:CC:DD:EE:FF

# Not: AA-BB-CC-DD-EE-FF
# Not: AABBCCDDEEFF
```

#### "Operation not permitted"
```bash
sudo fuser /dev/hci0
sudo pkill -f bluetooth
sudo pkill -f btscanner
sudo spooftooph -i hci0 -n "Test"
```

#### "Invalid device class"
```bash
# Valid classes (hex format)
# 0x040404 - Computer
# 0x200400 - Phone
# 0x080404 - Audio
# 0x000404 - Peripheral
spooftooph -i hci0 -c 0x040404
```

### Verification

```bash
# Check current identity
hciconfig hci0

# Scan to verify
btscanner -l 10 -q

# Check EIR data
sudo hcitool cmd 0x01 0x0005

# Save original before spoofing
hciconfig hci0 | grep -E "Name|BD Address" > /tmp/bt_original.txt

# Restore later
NAME=$(grep "Name" /tmp/bt_original.txt | awk '{print $2}')
ADDR=$(grep "BD Address" /tmp/bt_original.txt | awk '{print $3}')
sudo spooftooph -i hci0 -n "$NAME" -a "$ADDR"
```

### FAQ

#### Q: Spoofing doesn't show up in scan
```bash
# Wait 30 seconds for EIR update
# Check adapter is up: hciconfig hci0
# Try: sudo hcitool cmd 0x01 0x0005 to refresh
```

#### Q: Can't spoof MAC address
```bash
# Some adapters block MAC changes
# Try: sudo hciconfig hci0 down && sudo hciconfig hci0 up
# Check adapter capabilities: hciconfig hci0 features
```

#### Q: How to verify spoofing?
```bash
# Before spoofing
hciconfig hci0 | tee /tmp/before.txt

# Spoof
sudo spooftooph -i hci0 -n "NewName" -a 11:22:33:44:55:66

# After
hciconfig hci0 | tee /tmp/after.txt

# Compare
diff /tmp/before.txt /tmp/after.txt
```

### Resources

- [Spooftooph GitHub](https://github.com/securestate/spooftooph)
- [Kali Linux Tools](https://www.kali.org/tools/spooftooph/)
- [BlueZ Documentation](http://www.bluez.org/)
- [Bluetooth SIG](https://www.bluetooth.com/)

---

## Resources

| Resource | URL |
|----------|-----|
| Official Repository | https://github.com/securestate/spooftooph |
| Kali Tools Page | https://www.kali.org/tools/spooftooph/ |
| BlueZ Documentation | http://www.bluez.org/ |
| Bluetooth SIG | https://www.bluetooth.com/ |
| MITRE ATT&CK | https://attack.mitre.org/techniques/T1557/ |
