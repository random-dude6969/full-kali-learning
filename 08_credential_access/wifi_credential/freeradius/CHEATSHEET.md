# freeradius CHEATSHEET

## Quick Reference

### Start/Stop
```bash
freeradius -X                    # Debug mode
sudo systemctl start freeradius  # Start service
sudo systemctl stop freeradius   # Stop service
freeradius -XC                    # Check config
```

### radtest
```bash
radtest username password 127.0.0.1 0 testing123
```

### radclient
```bash
echo "User-Name = test" | radclient 127.0.0.1 auth testing123
radclient 127.0.0.1 status testing123
```

### radsniff
```bash
radsniff -i eth0 -p 1812           # Capture
radsniff -i eth0 -w capture.pcap   # Write to file
```

### radwho
```bash
radwho    # Show online users
```

### Configuration Files
```bash
/etc/freeradius/3.0/radiusd.conf     # Main config
/etc/freeradius/3.0/clients.conf      # Client definitions
/etc/freeradius/3.0/users             # User database
/etc/freeradius/3.0/mods-enabled/eap  # EAP config
```

### Common Options
```bash
-C              # Check configuration
-f              # Foreground mode
-s              # Single process
-t              # Disable threads
-X              # Full debug
-x              # Additional debug
```

---

## Target: 192.168.1.x

```bash
# Test RADIUS server
freeradius -X &
radtest testuser testpass 192.168.1.1 0 testing123
```
