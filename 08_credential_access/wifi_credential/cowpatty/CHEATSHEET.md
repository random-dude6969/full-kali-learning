# cowpatty CHEATSHEET

## Quick Reference

### Dictionary Attack
```bash
cowpatty -c -r capture.cap -s "SSID" -f wordlist.txt
```

### Generate PMK Database
```bash
genpmk -f wordlist.txt -s "SSID" -d pmk_database
```

### Use PMK Database
```bash
cowpatty -c -r capture.cap -s "SSID" -d pmk_database -2
```

### Multi-core
```bash
cowpatty -c -r capture.cap -s "SSID" -f wordlist.txt -2 -p 4
```

### Session
```bash
cowpatty -c -r capture.cap -s "SSID" -f wordlist.txt -S session1  # Save
cowpatty -c -r capture.cap -s "SSID" -f wordlist.txt -R session1  # Resume
```

### Options
```bash
-c          # Check for valid handshake
-v          # Verbose
-b          # Bruteforce mode
-2          # Use PMK database
```

---

## Target: 192.168.1.x

```bash
cowpatty -c -r capture-01.cap -s "TargetNetwork" -f /usr/share/wordlists/rockyou.txt
```
