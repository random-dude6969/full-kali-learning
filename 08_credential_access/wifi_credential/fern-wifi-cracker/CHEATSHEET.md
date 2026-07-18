# fern-wifi-cracker CHEATSHEET

## Quick Reference

### Launch
```bash
fern-wifi-cracker     # GUI mode
```

### Usage Steps
1. Select wireless interface
2. Enable monitor mode
3. Scan for access points
4. Select target
5. Choose attack type:
   - WEP: PTW/FMS attack
   - WPA: Dictionary attack
   - WPS: PIN brute force

### Options
```bash
--interface wlan0     # Specify interface
--channel 6           # Fix channel
--wordlist path       # Custom wordlist
```

---

## Target: 192.168.1.x

```bash
# Launch GUI
fern-wifi-cracker

# Select interface -> Scan -> Select target -> Attack
```
