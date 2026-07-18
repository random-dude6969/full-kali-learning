---
title: wifi-honey
category: credential_access
subcategory: wifi_credential
phase: 08
mitre_attack: TA0006
tags:
  - wireless
  - honeypot
  - monitoring
  - logging
  - detection
difficulty: intermediate
os: linux
install: sudo apt install wifi-honey
github: https://github.com/41undHund/wifi-honey
related:
  - aircrack-ng
  - kismet
  - hostapd
---

# wifi-honey

## Chapter 1: Overview

**wifi-honey** creates a wireless honeypot to monitor and log wireless attack activity. It captures authentication attempts, probes, and other wireless frames for analysis and detection of attack tools.

### Key Features

- **Honeypot AP**: Attracts wireless probes and connections
- **Traffic logging**: Captures all wireless activity
- **Attack detection**: Identifies common attack tools
- **Passive monitoring**: No active engagement
- **Forensic evidence**: Logs for analysis

### Detection Capabilities

| Attack | Detection Method |
|--------|------------------|
| Aircrack-ng | Probe patterns |
| Reaver | WPS enumeration |
| MDK3/MDK4 | Deauth floods |
| Karma | SSID impersonation |
| Evil Twin | Duplicate SSID |

---

## Chapter 2: Installation & Setup

```bash
sudo apt update && sudo apt install wifi-honey
```

### Dependencies

```bash
sudo apt install hostapd dnsmasq aircrack-ng
```

### Check Wireless Card

```bash
iwconfig
```

---

## Chapter 3: Basic Usage

### Launch wifi-honey

```bash
sudo wifi-honey
```

### Select Interface

```bash
# Menu-driven interface selection
sudo wifi-honey -i wlan0
```

### Specify Channel

```bash
sudo wifi-honey -c 6
```

---

## Chapter 4: Configuration

### Configuration File

```bash
/etc/wifi-honey/wifi-honey.conf
```

### Key Settings

```bash
# SSID to broadcast
SSID="FreeWiFi"

# Channel
CHANNEL=6

# Logging
LOGFILE="/var/log/wifi-honey.log"

# Interface
INTERFACE=wlan0
```

### Custom SSIDs

```bash
# Multiple SSIDs
SSID="FreeWiFi, Linksys, NETGEAR"
```

---

## Chapter 5: Monitoring

### View Logs

```bash
tail -f /var/log/wifi-honey.log
```

### Capture Traffic

```bash
airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF -w honey_capture wlan0mon
```

### Analyze Probes

```bash
# Extract probe requests
tshark -r capture.pcap -Y "wlan.fc.type_subtype == 0x04" -T fields -e wlan.sa -e wlan_mgt.ssid
```

---

## Chapter 6: Attack Detection

### Detect Aircrack-ng

```bash
# Look for injection patterns
grep -i "aircrack" /var/log/wifi-honey.log
```

### Detect Reaver

```bash
# Look for WPS enumeration
grep -i "wps" /var/log/wifi-honey.log
```

### Detect Deauth Attacks

```bash
# Monitor deauth frames
tshark -r capture.pcap -Y "wlan.fc.type_subtype == 0x0c"
```

---

## Chapter 7: Real-World Scenarios

### Scenario 1: Basic Monitoring

```bash
sudo wifi-honey -i wlan0
# Monitor logs for attack activity
tail -f /var/log/wifi-honey.log
```

### Scenario 2: Corporate Monitoring

```bash
# Deploy as detection sensor
sudo wifi-honey -i wlan0 -c 1 -c 6 -c 11
# Centralized log collection
```

### Scenario 3: Forensic Capture

```bash
sudo wifi-honey -i wlan0
# In parallel, capture all traffic
airodump-ng -w forensic_capture wlan0mon
```

---

## Chapter 8: Mitigation & Defense

**For security teams:**

- Deploy honeypots for early warning
- Monitor for attack tool signatures
- Correlate with IDS/IPS alerts
- Implement wireless anomaly detection
- Regular threat hunting
- Incident response procedures

---

## Resources

- [wifi-honey Documentation](https://github.com/41undHund/wifi-honey)
- [Kali Linux Tools](https://www.kali.org/tools/wifi-honey/)
- [Wireless IDS](https://www.wi-fi.org/discover-wi-fi/security)
- [Honeypot Concepts](https://www honeypots.net)

---

*Generated for educational purposes in authorized security testing environments only.*
