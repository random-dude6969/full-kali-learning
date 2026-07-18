# wifi-honey CHEATSHEET

## Quick Reference

### Launch
```bash
sudo wifi-honey                    # Default
sudo wifi-honey -i wlan0           # Specify interface
sudo wifi-honey -c 6               # Fix channel
```

### Configuration
```bash
/etc/wifi-honey/wifi-honey.conf    # Config file
```

### Monitor Logs
```bash
tail -f /var/log/wifi-honey.log
```

### Capture Traffic
```bash
airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF -w capture wlan0mon
```

### Analysis
```bash
# Extract probe requests
tshark -r capture.pcap -Y "wlan.fc.type_subtype == 0x04" -T fields -e wlan.sa -e wlan_mgt.ssid

# Monitor deauth
tshark -r capture.pcap -Y "wlan.fc.type_subtype == 0x0c"
```

### Detection
```bash
grep -i "aircrack" /var/log/wifi-honey.log
grep -i "wps" /var/log/wifi-honey.log
```

---

## Target: 192.168.1.x

```bash
# Deploy honeypot
sudo wifi-honey -i wlan0

# Monitor
tail -f /var/log/wifi-honey.log
```
