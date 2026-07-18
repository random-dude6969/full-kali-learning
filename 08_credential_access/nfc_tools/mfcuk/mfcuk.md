---
tool_name: "mfcuk"
display_name: "MFCUK"
mitre_tactic: "credential_access"
mitre_tactic_id: "TA0006"
mitre_subcategory: "nfc_tools"
install_command: "sudo apt install mfcuk"
version: "0.3.1"
github_repo: "https://github.com/nfc-tools/mfcuk"
kali_tools_page: "https://www.kali.org/tools/mfcuk/"
difficulty_level: "intermediate"
requires_root: false
gui_available: false
tags: ["nfc", "mifare", "rfid", "recovery", "crypto"]
related_tools: ["mfoc", "libnfc", "mfterm"]
---

# MFCUK

## Chapter 1: Introduction & Overview

### What is MFCUK?
MFCUK is a tool for recovering cryptographic keys from Mifare Classic RFID cards. It exploits vulnerabilities in the Crypto-1 cipher.

### History & Background
- **Created:** 2009 by Anders Sundell
- **Maintained:** nfc-tools
- **License:** GNU General Public License v2
- **Purpose:** Mifare Classic key recovery

MFCUK exploits weak Mifare Classic encryption.

### When to Use This Tool
- **RFID security:** Test Mifare Classic
- **Access control:** Audit card systems
- **CTF challenges:** Solve RFID challenges
- **Security assessments:** Test card security

### Key Concepts
- **Mifare Classic:** RFID card type
- **Crypto-1:** Encryption algorithm
- **Sector:** Memory division
- **Block:** Data unit
- **Key:** Cryptographic key

---

## Chapter 2: Installation & Setup

### System Requirements
- **NFC Reader:** ACR122U or compatible
- **Disk Space:** ~5 MB
- **Privileges:** User-level

### Installation Methods

#### Method 1: APT (Recommended)
```bash
sudo apt update
sudo apt install mfcuk
```

### Verification
```bash
mfcuk --help
```

---

## Chapter 3: Basic Usage

### First Recovery

#### Recover Keys
```bash
mfcuk -C -R -v 3 -f 0xFFFFFFFFFFFF
```
**Output:** Attempts to recover keys.

### Essential Commands

#### Key Recovery
```bash
mfcuk -C -R -v 3 -f 0xFFFFFFFFFFFF
```
**Explanation:** Recovers Mifare Classic keys.

#### Specific Card
```bash
mfcuk -C -R -v 3 -f 0xFFFFFFFFFFFF -o keys.txt
```
**Explanation:** Saves recovered keys.

### Complete Flags Reference

#### Operation Options
| Flag | Description | Example |
|------|-------------|---------|
| `-C` | Check for card | `mfcuk -C` |
| `-R` | Recover keys | `mfcuk -R` |
| `-W` | Write keys | `mfcuk -W` |

#### Target Options
| Flag | Description | Example |
|------|-------------|---------|
| `-f` | Target key | `mfcuk -f 0xFFFFFFFFFFFF` |
| `-o` | Output file | `mfcuk -o keys.txt` |
| `-i` | Input file | `mfcuk -i keys.txt` |

#### Debug Options
| Flag | Description | Example |
|------|-------------|---------|
| `-v` | Verbose | `mfcuk -v 3` |
| `-d` | Debug | `mfcuk -d` |

---

## Chapter 4: Intermediate Usage

### Advanced Recovery Techniques

#### Recover All Keys
```bash
# Recover all sector keys
mfcuk -C -R -v 3 -f 0xFFFFFFFFFFFF -o recovered_keys.txt
```

#### Use Recovered Keys
```bash
# Use keys with mfoc
mfoc -k recovered_keys.txt -r card.dump
```

### Scripting & Automation

#### Bash Scripting
```bash
#!/bin/bash
# Automated key recovery
OUTPUT="keys_$(date +%Y%m%d).txt"

echo "[*] Starting MFCUK recovery..."
mfcuk -C -R -v 3 -f 0xFFFFFFFFFFFF -o $OUTPUT

echo "[+] Keys saved to $OUTPUT"
```

### Combining with Other Tools

#### With MFOC
```bash
# Recover keys
mfcuk -C -R -v 3 -o keys.txt

# Dump card with keys
mfoc -k keys.txt -r card.dump
```

#### With NFC-List
```bash
# List card info
nfc-list

# Recover keys
mfcuk -C -R -v 3 -o keys.txt
```

---

## Chapter 5: Advanced Usage

### Advanced Recovery Techniques

#### Brute Force Keys
```bash
# Brute force specific key
mfcuk -C -R -v 3 -f 0xA0A1A2A3A4A5

# Brute force all keys
mfcuk -C -R -v 3 -f 0xFFFFFFFFFFFF
```

### Integration in Pentest Workflow

#### Phase 1: Card Reading
```bash
# List card
nfc-list

# Check card type
```

#### Phase 2: Key Recovery
```bash
# Recover keys
mfcuk -C -R -v 3 -o keys.txt
```

#### Phase 3: Card Dump
```bash
# Dump card
mfoc -k keys.txt -r card.dump
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: CTF Challenge
**Objective:** Read Mifare card

**Steps:**
```bash
# 1. List card
nfc-list

# 2. Recover keys
mfcuk -C -R -v 3 -o keys.txt

# 3. Dump card
mfoc -k keys.txt -r card.dump

# 4. Analyze dump
```

---

## Chapter 7: Detection & Defense

### How Defenders Detect MFCUK

#### System Indicators
```
# Unusual NFC reader activity
# Key recovery attempts
```

### Mitigation Strategies

#### Strong Keys
```bash
# Use unique keys per card
# Avoid default keys
# Implement key diversification
```

### Blue Team Response
1. **Use unique keys** - Per-card keys
2. **Avoid default keys** - Change defaults
3. **Monitor NFC readers** - Detect attacks
4. **Upgrade cards** - Use Mifare DESFire

---

## Chapter 8: Troubleshooting

### Common Errors

#### Error: "No reader found"
**Cause:** NFC reader not connected
**Solution:**
```bash
# Check reader
pcscd status
nfc-list
```

### FAQ

**Q: Is mfcuk legal to use?**
A: Yes, mfcuk is a legitimate security testing tool. However, using it on cards you don't own is illegal.

**Q: How do I update mfcuk?**
A: `sudo apt update && sudo apt upgrade mfcuk`

---

## Resources

### Official Documentation
- [MFCUK GitHub](https://github.com/nfc-tools/mfcuk)
- [MFCUK Wiki](https://github.com/nfc-tools/mfcuk/wiki)

### Related Tools
- **mfoc:** Mifare Classic dump tool
- **libnfc:** NFC library
- **mfterm:** Mifare terminal

### Practice Resources
- [HackTheBox](https://www.hackthebox.com/)
- [TryHackMe](https://tryhackme.com/)
