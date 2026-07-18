# Bluetooth Reconnaissance Tools — Quick Reference Cheatsheet

## Overview

| Tool | Purpose | Difficulty | Root | GUI |
|------|---------|------------|------|-----|
| **BetterCap** | Network attack framework (BLE, WiFi, ARP, DNS) | Intermediate-Advanced | Yes | No |
| **BlueLog** | Bluetooth scanner/logger | Beginner | Yes | No |
| **BlueRanger** | Bluetooth signal strength/distance | Beginner | Yes | No |
| **Bluesnarfer** | Bluetooth information extraction | Intermediate | Yes | No |
| **BTScanner** | Interactive Bluetooth scanner | Beginner | Yes | Yes |
| **RedFang** | Bluetooth brute-force & sniffing | Intermediate | Yes | No |
| **Spooftooph** | Bluetooth address/name spoofing | Intermediate | Yes | No |
| **Ubertooth Util** | SDR-based Bluetooth analysis | Advanced | Yes | Yes |

---

## Installation

```bash
# Install all Bluetooth tools at once
sudo apt install bettercap bluelog blueranger bluesnarfer btscanner redfang spooftooph ubertooth

# Individual installation
sudo apt install bettercap      # Network attack framework
sudo apt install bluelog         # Bluetooth scanner/logger
sudo apt install blueranger      # Signal strength tool
sudo apt install bluesnarfer     # Info extraction
sudo apt install btscanner       # Interactive scanner
sudo apt install redfang         # Brute-force/sniffing
sudo apt install spooftooph      # Address spoofing
sudo apt install ubertooth       # SDR analysis
```

---

## Quick Commands

### BetterCap
```bash
# Network discovery
sudo bettercap -eval "net.probe on; net.show"

# BLE scanning
sudo bettercap -eval "ble.recon on; ble.show"

# ARP spoofing
sudo bettercap -eval "set arp.spoof.targets 192.168.1.100; arp.spoof on"

# WiFi reconnaissance
sudo bettercap -eval "wifi.recon on; wifi.show"
```

### BlueLog
```bash
# Basic scan (runs until Ctrl+C)
sudo bluelog

# Timed scan (300 seconds)
sudo bluelog -t 300

# Verbose scan to file
sudo bluelog -v -t 3600 -o bluetooth_log.txt

# Background monitoring
sudo bluelog -t 86400 -o /var/log/bt_daily.txt &
```

### BlueRanger
```bash
# Scan all devices
sudo blueranger

# Track specific device
sudo blueranger -t AA:BB:CC:DD:EE:FF

# Continuous monitoring
sudo blueranger -t AA:BB:CC:DD:EE:FF -c
```

### Bluesnarfer
```bash
# Get device info
bluesnarfer -r AA:BB:CC:DD:EE:FF -i

# Get phonebook
bluesnarfer -r AA:BB:CC:DD:EE:FF -b

# Get calendar
bluesnarfer -r AA:BB:CC:DD:EE:FF -c

# Full enumeration
bluesnarfer -r AA:BB:CC:DD:EE:FF -b -c -i
```

### BTScanner
```bash
# Start interactive scanner
sudo btscanner

# Key commands in interactive mode:
# i - Start scan
# Enter - View device details
# q - Quit
# / - Search

# Command-line scan
btscanner --scan --output results.txt
```

### RedFang
```bash
# Brute-force PIN
sudo redfang -r AA:BB:CC:DD:EE:FF

# Sniff traffic
sudo redfang -s AA:BB:CC:DD:EE:FF

# Save capture
sudo redfang -i hci0 -w capture.pcap
```

### Spooftooph
```bash
# Random address
sudo spooftooph -i hci0 -r

# Spoof specific address
sudo spooftooph -i hci0 -s AA:BB:CC:DD:EE:FF -n "Spoofed"

# Restore original
sudo spooftooph -i hci0 -o
```

### Ubertooth Util
```bash
# Check device
ubertooth-util -v

# BLE capture
ubertooth-ble -f

# Spectrum analysis
ubertooth-specan -f 2400,2480

# Save capture
ubertooth-ble -f -c capture.cfifo
```

---

## Bluetooth Device Classes

| Class | Type | Example |
|-------|------|---------|
| `0x000404` | Peripheral (Keyboard/Mouse) | Logitech K380 |
| `0x040404` | Computer/Laptop | Dell XPS |
| `0x080404` | Audio Device | JBL Flip 5 |
| `0x200400` | Phone/Mobile | iPhone, Galaxy |
| `0x200800` | Medical Device | Patient Monitor |

---

## Common Workflow

```bash
# 1. Discover devices
sudo bluelog -t 300 -o devices.txt

# 2. Detailed scan
sudo btscanner

# 3. Extract info (authorized only)
bluesnarfer -r TARGET -b -c -i

# 4. Test authentication
sudo redfang -r TARGET

# 5. Track signal
sudo blueranger -t TARGET -c

# 6. Advanced analysis (requires Ubertooth)
ubertooth-ble -f
```

---

## MITRE ATT&CK Mapping

| Technique | ID | Tools |
|-----------|-----|-------|
| Active Scanning | T1595 | BlueLog, BTScanner, BetterCap |
| Search Open Websites | T1593 | Bluesnarfer |
| Remote Services | T1021 | Bluesnarfer, RedFang |
| Exploitation for Client Execution | T1203 | BetterCap, RedFang |
| Masquerading | T1036 | Spooftooph |

---

## Legal Reminder

**Always obtain written authorization before testing Bluetooth security.**

- Only scan/test devices you own or have permission to test
- Document all activities
- Follow responsible disclosure
- Local laws vary — consult legal counsel
