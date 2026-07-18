---
title: fern-wifi-cracker
category: credential_access
subcategory: wifi_credential
phase: 08
mitre_attack: TA0006
tags:
  - wireless
  - gui
  - automated
  - wpa
  - wep
  - wps
difficulty: beginner
os: linux
install: sudo apt install fern-wifi-cracker
github: https://github.com/savio-code/fern-wifi-cracker
related:
  - wifite
  - aircrack-ng
  - reaver
---

# fern-wifi-cracker

## Chapter 1: Overview

**fern-wifi-cracker** is a GUI-based automated wireless auditing tool that provides a graphical interface for WEP, WPA/WPA2, and WPS attacks. It integrates aircrack-ng suite tools into an easy-to-use interface.

### Key Features

- **GUI interface**: Visual wireless auditing
- **Automated attacks**: One-click WEP/WPA/WPS attacks
- **Real-time monitoring**: Live attack progress
- **Session management**: Save and resume audits
- **Dependency management**: Automatic tool installation

### Supported Attacks

| Attack Type | Method | Tool |
|-------------|--------|------|
| WEP | PTW, FMS | aircrack-ng |
| WPA/WPA2 | Dictionary | aircrack-ng |
| WPS | PIN brute force | reaver |
| Evil Twin | Rogue AP | airbase-ng |

---

## Chapter 2: Installation & Setup

```bash
sudo apt update && sudo apt install fern-wifi-cracker
```

### Dependencies

```bash
sudo apt install aircrack-ng reaver tshark xterm
```

### Launch

```bash
fern-wifi-cracker
```

---

## Chapter 3: Basic Usage

### Launch Application

```bash
fern-wifi-cracker
```

### Select Interface

1. Click "Select Interface"
2. Choose wireless adapter
3. Enable monitor mode

### Scan Networks

1. Click "Scan for APs"
2. Wait for results
3. Select target network

### Attack Target

1. Select attack type (WEP/WPA/WPS)
2. Select wordlist (for WPA)
3. Click "Attack"

---

## Chapter 4: WEP Attacks

### Capture IVs

1. Select WEP target
2. Click "Start WEP Attack"
3. Wait for IV collection
4. Crack when enough IVs collected

### Configure Attack

- **Channel**: Auto or manual
- **Attack mode**: PTW or FMS
- **IV threshold**: Minimum for cracking

---

## Chapter 5: WPA/WPA2 Attacks

### Dictionary Attack

1. Select WPA target
2. Choose wordlist file
3. Click "Start WPA Attack"
4. Wait for handshake capture
5. Automated dictionary attack

### Options

- **Deauth**: Force handshake capture
- **Wordlist**: Default or custom
- **Threads**: Multi-core cracking

---

## Chapter 6: WPS Attacks

### PIN Brute Force

1. Select WPS-enabled target
2. Click "Start WPS Attack"
3. Select attack tool (reaver)
4. Monitor progress

### Pixie Dust

1. Select WPS target
2. Enable Pixie Dust mode
3. Fast offline computation

---

## Chapter 7: Real-World Scenarios

### Scenario 1: Quick WPA Test

```bash
fern-wifi-cracker
# 1. Select interface
# 2. Scan networks
# 3. Select WPA target
# 4. Choose wordlist
# 5. Start attack
```

### Scenario 2: WEP Audit

```bash
fern-wifi-cracker
# 1. Select interface
# 2. Scan for WEP networks
# 3. Select target
# 4. Start WEP attack
# 5. Wait for IVs
```

---

## Chapter 8: Mitigation & Defense

**For network administrators:**

- Disable WPS
- Use WPA3-SAE
- Strong passphrases
- Monitor for rogue APs
- Enterprise authentication
- Regular security audits

---

## Resources

- [fern-wifi-cracker GitHub](https://github.com/savio-code/fern-wifi-cracker)
- [Kali Linux Tools](https://www.kali.org/tools/fern-wifi-cracker/)
- [WiFi Security](https://www.wi-fi.org/discover-wi-fi/security)
- [OffSec PEN-210 Course](https://www.offsec.com/courses/pen-210/)

---

*Generated for educational purposes in authorized security testing environments only.*
