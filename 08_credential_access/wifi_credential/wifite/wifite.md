---
title: wifite
category: credential_access
subcategory: wifi_credential
phase: 08
mitre_attack: TA0006
tags:
  - wireless
  - automation
  - wpa
  - wep
  - wps
  - cracking
  - automated
difficulty: beginner
os: linux
install: sudo apt install wifite
github: https://github.com/kimocoder/wifite2
related:
  - aircrack-ng
  - reaver
  - bully
  - pixiewps
---

# wifite

## Chapter 1: Overview

**wifite** (wifite2) is an automated wireless auditing tool that wraps aircrack-ng, reaver, bully, pixiewps, and hashcat into a single automated attack tool. It performs all attacks without user intervention after initial configuration.

### Key Features

- **Fully automated**: Scan, attack, crack without manual steps
- **Multiple attack types**: WEP, WPA/WPA2, WPS, PMKID
- **Smart targeting**: Prioritizes targets by signal strength and clients
- **Persistent cracking**: Continues cracking in background
- **MAC randomization**: Spoofs MAC address during attacks
- **Infinite mode**: Continuous scanning and attacking

### Attack Matrix

| Encryption | Attack Method | Tool Used |
|------------|---------------|-----------|
| WEP | PTW, FMS, ChopChop | aircrack-ng |
| WPA/WPA2 | Handshake capture + dict | aircrack-ng/hashcat |
| WPA3 | SAE capture | hashcat |
| WPS | PIN brute force | reaver/bully |
| WPS | Pixie Dust | pixiewps |
| PMKID | Clientless capture | hcxdumptool |

---

## Chapter 2: Installation & Setup

```bash
sudo apt update && sudo apt install wifite
```

**Install size**: 8.01 MB

**Dependencies**: aircrack-ng, reaver, tshark, python3

### Verify Dependencies

```bash
wifite --check
```

---

## Chapter 3: Basic Usage

### Quick Start

```bash
sudo wifite
```

### Target by Signal Strength

```bash
sudo wifite -pow 50
```

### Target WPS Networks Only

```bash
sudo wifite --wps
```

### Target WPA Networks Only

```bash
sudo wifite --wpa
```

### Target WPA3 Networks Only

```bash
sudo wifite --wpa3
```

---

## Chapter 4: Attack Configuration

### Specify Interface

```bash
sudo wifite -i wlan0
```

### Set Channel

```bash
sudo wifite -c 6
```

### Set Custom Wordlist

```bash
sudo wifite --dict /path/to/wordlist.txt
```

### Enable MAC Randomization

```bash
sudo wifite --random-mac
```

### Kill Conflicting Processes

```bash
sudo wifite --kill
```

---

## Chapter 5: WPA/WPA2 Attacks

### Capture Handshake

```bash
# Default behavior
sudo wifite --wpa

# Specify wordlist
sudo wifite --wpa --dict rockyou.txt

# Skip cracking (capture only)
sudo wifite --wpa --skip-crack
```

### PMKID-Only Mode

```bash
sudo wifite --pmkid
```

### Ignore Previously Cracked

```bash
sudo wifite --wpa --ignore-cracked
```

### Check for Handshakes

```bash
wifite --check capture.cap
wifite --check hs/
```

---

## Chapter 6: WPS Attacks

### WPS-Only Attacks

```bash
sudo wifite --wps-only
```

### Choose Attack Tool

```bash
# Use reaver (default)
sudo wifite --wps --reaver

# Use bully
sudo wifite --wps --bully
```

### Ignore WPS Lockouts

```bash
sudo wifite --wps --ignore-locks
```

---

## Chapter 7: Advanced Features

### Infinite Mode

```bash
sudo wifite --infinite
```

### Only Targets with Clients

```bash
sudo wifite --clients-only
```

### First N Targets

```bash
sudo wifite --first 5
```

### Passive Mode (No Deauth)

```bash
sudo wifite --nodeauths
```

### View Previously Cracked

```bash
wifite --cracked
```

---

## Chapter 8: Real-World Scenarios

### Scenario 1: Quick Network Audit

```bash
sudo wifite -pow 40 --kill
# Review results
wifite --cracked
```

### Scenario 2: WPS Vulnerability Assessment

```bash
sudo wifite --wps --ignore-locks --pow 50
```

### Scenario 3: Comprehensive Assessment

```bash
sudo wifite -i wlan0 --random-mac --kill --pow 30 --dict rockyou.txt
```

### Scenario 4: Continuous Monitoring

```bash
sudo wifite --infinite --pow 50 --wpa
```

---

## Mitigation & Defense

**For network administrators:**

- Disable WPS on all access points
- Use WPA3-SAE where possible
- Implement strong passphrases (20+ characters)
- Enable client isolation
- Deploy wireless IDS/IPS
- Regular firmware updates
- Monitor for rogue devices

---

## Resources

- [wifite2 GitHub](https://github.com/kimocoder/wifite2)
- [Original wifite](https://github.com/derv82/wifite2)
- [Kali Linux Documentation](https://www.kali.org/tools/wifite/)
- [OffSec PEN-210 Course](https://www.offsec.com/courses/pen-210/)
- [Wireless Security Guide](https://www.cisa.gov/news-events/cybersecurity-advisories)

---

*Generated for educational purposes in authorized security testing environments only.*
