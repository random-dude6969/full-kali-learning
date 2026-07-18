---
tool_name: "mfoc"
display_name: "MFOC"
mitre_tactic: "credential_access"
mitre_tactic_id: "TA0006"
mitre_subcategory: "nfc_tools"
install_command: "sudo apt install mfoc"
version: "0.3.1"
github_repo: "https://github.com/nfc-tools/mfoc"
kali_tools_page: "https://www.kali.org/tools/mfoc/"
difficulty_level: "intermediate"
requires_root: false
gui_available: false
tags: ["nfc", "mifare", "rfid", "dump", "recovery"]
related_tools: ["mfcuk", "libnfc", "mfterm"]
---

# MFOC

## Chapter 1: Introduction & Overview

### What is MFOC?
MFOC is a tool for dumping Mifare Classic RFID cards. It uses recovered keys to read card contents.

### History & Background
- **Created:** 2007 by nfc-tools
- **Maintained:** nfc-tools
- **License:** GNU General Public License v2
- **Purpose:** Mifare Classic card dumping

MFOC reads Mifare Classic card contents.

### When to Use This Tool
- **RFID security:** Read Mifare cards
- **Access control:** Audit card systems
- **CTF challenges:** Solve RFID challenges
- **Security assessments:** Test card security

### Key Concepts
- **Mifare Classic:** RFID card type
- **Sector:** Memory division
- **Block:** Data unit
- **Key:** Cryptographic key
- **Dump:** Card data copy

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
sudo apt install mfoc
```

### Verification
```bash
mfoc --help
```

---

## Chapter 3: Basic Usage

### First Dump

#### Dump Card
```bash
mfoc -r card.dump
```
**Output:** Creates card dump file.

### Essential Commands

#### Dump Card
```bash
mfoc -r card.dump
```
**Explanation:** Dumps card to file.

#### With Known Keys
```bash
mfoc -k keys.txt -r card.dump
```
**Explanation:** Uses provided keys.

### Complete Flags Reference

#### Operation Options
| Flag | Description | Example |
|------|-------------|---------|
| `-r` | Output file | `mfoc -r card.dump` |
| `-k` | Key file | `mfoc -k keys.txt` |
| `-O` | Override | `mfoc -O card.dump` |
| `-P` | Partial dump | `mfoc -P card.dump` |

#### Target Options
| Flag | Description | Example |
|------|-------------|---------|
| `-t` | Target key | `mfoc -t 0xFFFFFFFFFFFF` |
| `-b` | Block range | `mfoc -b 0-63` |
| `-f` | Force | `mfoc -f` |

#### Debug Options
| Flag | Description | Example |
|------|-------------|---------|
| `-v` | Verbose | `mfoc -v` |
| `-d` | Debug | `mfoc -d` |

---

## Chapter 4: Intermediate Usage

### Advanced Dumping Techniques

#### Use Recovered Keys
```bash
# Dump with recovered keys
mfoc -k recovered_keys.txt -r card.dump
```

#### Partial Dump
```bash
# Dump specific blocks
mfoc -P -b 0-15 card.dump
```

### Scripting & Automation

#### Bash Scripting
```bash
#!/bin/bash
# Automated card dump
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
OUTPUT="card_${TIMESTAMP}.dump"

echo "[*] Dumping card..."
mfoc -r $OUTPUT

echo "[+] Card saved to $OUTPUT"
```

### Combining with Other Tools

#### With MFCUK
```bash
# Recover keys
mfcuk -C -R -v 3 -o keys.txt

# Dump card
mfoc -k keys.txt -r card.dump
```

#### With NFC-Utils
```bash
# List card
nfc-list

# Dump card
mfoc -r card.dump
```

---

## Chapter 5: Advanced Usage

### Advanced Dumping Techniques

#### Authentication Methods
```bash
# Default keys
mfoc -t 0xFFFFFFFFFFFF -r card.dump

# Specific keys
mfoc -k keys.txt -r card.dump
```

### Integration in Pentest Workflow

#### Phase 1: Card Reading
```bash
# List card
nfc-list

# Check card info
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
strings card.dump
```

---

## Chapter 7: Detection & Defense

### How Defenders Detect MFOC

#### System Indicators
```
# Unusual NFC reader activity
# Card dump attempts
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

#### Error: "Wrong key"
**Cause:** Incorrect key
**Solution:**
```bash
# Recover keys first
mfcuk -C -R -v 3 -o keys.txt

# Use recovered keys
mfoc -k keys.txt -r card.dump
```

### FAQ

**Q: Is mfoc legal to use?**
A: Yes, mfoc is a legitimate security testing tool. However, using it on cards you don't own is illegal.

**Q: How do I update mfoc?**
A: `sudo apt update && sudo apt upgrade mfoc`

---

## Resources

### Official Documentation
- [MFOC GitHub](https://github.com/nfc-tools/mfoc)
- [MFOC Wiki](https://github.com/nfc-tools/mfoc/wiki)

### Related Tools
- **mfcuk:** Key recovery
- **libnfc:** NFC library
- **mfterm:** Mifare terminal

### Practice Resources
- [HackTheBox](https://www.hackthebox.com/)
- [TryHackMe](https://tryhackme.com/)
