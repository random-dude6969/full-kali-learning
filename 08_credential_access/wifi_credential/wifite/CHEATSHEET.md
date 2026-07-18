# wifite CHEATSHEET

## Quick Reference

### Basic Usage
```bash
sudo wifite                    # Auto attack all
sudo wifite -i wlan0           # Specify interface
sudo wifite --kill             # Kill conflicting processes
```

### Target Specific Networks
```bash
sudo wifite --wpa              # WPA only
sudo wifite --wpa2             # WPA2 only
sudo wifite --wpa3             # WPA3 only
sudo wifite --wep              # WEP only
sudo wifite --wps              # WPS only
```

### Signal Filtering
```bash
sudo wifite -pow 50            # Min 50dB signal
sudo wifite -pow 40            # Min 40dB signal
```

### Custom Wordlist
```bash
sudo wifite --dict /path/to/wordlist.txt
sudo wifite --wpa --dict rockyou.txt
```

### WPS Attacks
```bash
sudo wifite --wps-only         # WPS PIN attacks only
sudo wifite --wps --bully      # Use bully
sudo wifite --wps --reaver     # Use reaver
sudo wifite --ignore-locks     # Ignore WPS locks
```

### PMKID Attack
```bash
sudo wifite --pmkid            # PMKID only
sudo wifite --no-pmkid         # Skip PMKID
```

### Advanced Options
```bash
sudo wifite --random-mac       # Randomize MAC
sudo wifite --infinite         # Continuous mode
sudo wifite --clients-only     # Only targets with clients
sudo wifite --first 5          # Attack first 5 targets
sudo wifite --skip-crack       # Don't crack, just capture
sudo wifite --ignore-cracked   # Skip previously cracked
```

### Output Options
```bash
wifite --cracked               # Show cracked APs
wifite --ignored               # Show ignored APs
wifite --check capture.cap     # Check for handshake
```

---

## Target: 192.168.1.x

```bash
# Quick WPA audit
sudo wifite --wpa --kill --pow 40 --dict rockyou.txt

# WPS assessment
sudo wifite --wps --kill --pow 50

# Comprehensive
sudo wifite -i wlan0 --random-mac --kill --pow 30
```
