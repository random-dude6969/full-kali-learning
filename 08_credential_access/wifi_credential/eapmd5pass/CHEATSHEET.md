# eapmd5pass CHEATSHEET

## Quick Reference

### Dictionary Attack
```bash
eapmd5pass -r eap_capture.pcap -w wordlist.txt
```

### Multiple Captures
```bash
eapmd5pass -r capture1.pcap -r capture2.pcap -w wordlist.txt
```

### Hashcat Conversion
```bash
eapmd5pass -r eap.pcap -o hash.txt -J
```

### Options
```bash
-r file.pcap      # Input capture file
-w wordlist.txt   # Wordlist file
-i interface      # Network interface
-v                # Verbose
-o output.txt     # Output file
-J                # Convert to hashcat format
```

### Capture EAP Traffic
```bash
airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF -w eap wlan0mon
tshark -i wlan0mon -f "ether proto 0x888e" -w eap.pcap
```

---

## Target: 192.168.1.x

```bash
# Capture and crack
airodump-ng -c 6 --bssid 192.168.1.1 -w eap wlan0mon
eapmd5pass -r eap-01.cap -w /usr/share/wordlists/rockyou.txt
```
