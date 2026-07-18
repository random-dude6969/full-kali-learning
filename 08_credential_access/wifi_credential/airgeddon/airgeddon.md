---
title: airgeddon
category: credential_access
subcategory: wifi_credential
phase: 08
mitre_attack: TA0006
tags:
  - wireless
  - automation
  - wpa
  - wps
  - evil_twin
  - phishing
  - menu_driven
difficulty: beginner
os: linux
install: sudo apt install airgeddon
github: https://github.com/v1s1t0r1sh3r3/airgeddon
related:
  - wifite
  - aircrack-ng
  - reaver
---

# airgeddon

## Chapter 1: Overview

**airgeddon** is a multi-use bash script for Linux systems to audit wireless networks. It provides a menu-driven interface that orchestrates multiple wireless attack tools under a single unified interface. The script automates complex attack sequences that would otherwise require manual configuration of individual tools.

### Key Features

- **Menu-driven interface**: Easy navigation for complex attacks
- **Multiple attack types**: WPA/WPA2, WPS, Evil Twin, Captive Portal
- **Tool integration**: Wraps aircrack-ng, reaver, bully, hashcat, and more
- **Dependency management**: Checks and installs required tools
- **Multilingual support**: Interface available in multiple languages
- **Persistent sessions**: Resume interrupted attacks

### Supported Attack Vectors

| Category | Attacks |
|----------|---------|
| **WPA/WPA2** | Dictionary, Handshake capture, PMKID |
| **WPS** | PIN brute force, Pixie Dust, Bully/Reaver |
| **Evil Twin** | Rogue AP, Captive Portal, DNS spoofing |
| **Enterprise** | Evil Twin with WPA-Enterprise |
| **DoS** | Deauthentication, Authentication flooding |

---

## Chapter 2: Installation & Setup

### Install Dependencies

```bash
sudo apt update
sudo apt install -y aircrack-ng tmux xterm reaver bully pixiewps hashcat
```

### Install airgeddon

```bash
# Via package manager
sudo apt install airgeddon

# Or from source
git clone https://github.com/v1s1t0r1sh3r3/airgeddon.git
cd airgeddon
sudo bash airgeddon.sh
```

### First Run

```bash
sudo airgeddon
```

The script will:
1. Check all dependencies
2. Update tools if needed
3. Present main menu

---

## Chapter 3: Basic Usage

### Launch airgeddon

```bash
sudo airgeddon
```

### Main Menu Navigation

```
[0] Exit
[1] Interface selection
[2] Explore/capture networks
[3] Attack options
[4] Useful tools
[5] Framework updates
```

### Select Interface

```
Menu [1] -> Select your wireless interface
```

### Scan Networks

```
Menu [2] -> Discover networks
```

### Select Target

After scanning, select the target network from the displayed list.

---

## Chapter 4: WPA/WPA2 Attacks

### Capture Handshake

```
Menu [3] -> WPA/WPA2 attacks -> Handshake capture
```

**Process**:
1. Select target network
2. Wait for handshake capture
3. Attack will capture 4-way handshake
4. Save handshake for offline cracking

### Crack Handshake

```
Menu [3] -> WPA/WPA2 attacks -> Handshake cracking
```

**Options**:
- Use default wordlist or specify custom
- Choose between aircrack-ng and hashcat
- Set thread count for faster cracking

### PMKID Attack

```
Menu [3] -> WPA/WPA2 attacks -> PMKID
```

**Benefits**: No client required, passive capture

---

## Chapter 5: WPS Attacks

### WPS PIN Attack

```
Menu [3] -> WPS attacks -> PIN brute force
```

**Tools available**:
- Reaver (default)
- Bully (alternative)
- Pixie Dust (offline)

### Pixie Dust Attack

```
Menu [3] -> WPS attacks -> Pixie Dust
```

**Process**:
1. Capture WPS M1-M7 messages
2. Offline PIN computation
3. Works against vulnerable routers
4. Usually completes in seconds

### Configure Attack Parameters

```
[1] Set PIN delay (default: 5 seconds)
[2] Set timeout (default: 10 seconds)
[3] Enable/disable lock ignore
[4] Set max attempts
```

---

## Chapter 6: Evil Twin Attacks

### Basic Evil Twin

```
Menu [3] -> Evil Twin attacks -> Basic
```

**Process**:
1. Deauthenticate clients from real AP
2. Create rogue AP with same ESSID
3. Clients connect to rogue AP
4. Capture credentials

### Evil Twin with Captive Portal

```
Menu [3] -> Evil Twin attacks -> Captive Portal
```

**Templates available**:
- firmware_upgrade
- router_login
- free_wifi
- custom HTML

### Evil Twin + DNS Spoofing

```
Menu [3] -> Evil Twin attacks -> DNS spoof
```

**Use cases**:
- Redirect to phishing pages
- Custom DNS responses
- Man-in-the-middle positioning

---

## Chapter 7: Advanced Features

### Enterprise WPA Attacks

```
Menu [3] -> Enterprise attacks
```

**Requirements**:
- Hostapd-mana
- freeradius
- Custom certificate generation

### DoS Attacks

```
Menu [4] -> DoS attack menu
```

**Attack types**:
- Deauthentication
- Authentication flooding
- Association flooding
- Power saving DoS

### Useful Tools

```
Menu [4] -> Useful tools
```

**Available tools**:
- MAC changer
- Network scanner
- Packet analysis
- Wordlist management

---

## Chapter 8: Real-World Scenarios

### Scenario 1: Quick WPA Audit

```bash
sudo airgeddon
# [1] Select interface
# [2] Scan networks
# [3] Select target
# [4] WPA attacks -> Handshake capture
# [5] WPA attacks -> Crack with wordlist
```

### Scenario 2: WPS Vulnerability Test

```bash
sudo airgeddon
# [1] Select interface
# [2] Scan networks
# [3] Filter WPS-enabled
# [4] WPS attacks -> Pixie Dust
# [5] Review results
```

### Scenario 3: Rogue AP Assessment

```bash
sudo airgeddon
# [1] Select interface
# [2] Scan networks
# [3] Evil Twin -> Captive Portal
# [4] Select template
# [5] Monitor connections
```

---

## Mitigation & Defense

**For network administrators:**

- Disable WPS immediately
- Use WPA3 where possible
- Implement RADIUS authentication
- Monitor for rogue APs
- Enable wireless IDS/IPS
- Regular security audits
- Employee security awareness training

---

## Resources

- [airgeddon GitHub](https://github.com/v1s1t0r1sh3r3/airgeddon)
- [Official Wiki](https://github.com/v1s1t0r1sh3r3/airgeddon/wiki)
- [Attack Documentation](https://github.com/v1s1t0r1sh3r3/airgeddon/wiki/Attacks)
- [OffSec PEN-210 Course](https://www.offsec.com/courses/pen-210/)
- [WiFi Security Best Practices](https://www.cisa.gov/news-events/cybersecurity-advisories)

---

*Generated for educational purposes in authorized security testing environments only.*
