---
tool_name: "libnfc"
display_name: "libnfc"
mitre_tactic: "credential_access"
mitre_tactic_id: "TA0006"
mitre_subcategory: "nfc_tools"
install_command: "sudo apt install libnfc-utils"
version: "1.8.0"
github_repo: "https://github.com/nfc-tools/libnfc"
kali_tools_page: "https://www.kali.org/tools/libnfc/"
difficulty_level: "intermediate"
requires_root: false
gui_available: false
tags: ["nfc", "rfid", "library", "reader", "mifare"]
related_tools: ["mfcuk", "mfoc", "mfterm"]
---

# libnfc

## Chapter 1: Introduction & Overview

### What is libnfc?
libnfc is a library for NFC (Near Field Communication) devices. It provides utilities for interacting with NFC readers and tags.

### History & Background
- **Created:** 2009 by nfc-tools
- **Maintained:** nfc-tools
- **License:** GNU Lesser General Public License v2
- **Purpose:** NFC device interaction

libnfc is the standard NFC library for Linux.

### When to Use This Tool
- **RFID security:** Interact with NFC devices
- **Access control:** Test card systems
- **CTF challenges:** Solve RFID challenges
- **Development:** Build NFC applications

### Key Concepts
- **NFC:** Near Field Communication
- **Reader:** NFC device
- **Tag:** NFC card/chip
- **Mifare:** NFC card type

### Available Tools
| Tool | Description |
|------|-------------|
| `nfc-list` | List NFC devices |
| `nfc-mfclassic` | Mifare Classic operations |
| `nfc-mfultralight` | Mifare Ultralight operations |
| `nfc-emulate` | Emulate NFC tag |

---

## Chapter 2: Installation & Setup

### System Requirements
- **NFC Reader:** ACR122U or compatible
- **Disk Space:** ~10 MB
- **Privileges:** User-level

### Installation Methods

#### Method 1: APT (Recommended)
```bash
sudo apt update
sudo apt install libnfc-utils libnfc-bin
```

### Verification
```bash
nfc-list
```

---

## Chapter 3: Basic Usage

### First Use

#### List NFC Devices
```bash
nfc-list
```
**Output:** Shows connected NFC readers.

#### Read Mifare Card
```bash
nfc-mfclassic r b card.dump
```
**Output:** Reads card to dump file.

### Essential Commands

#### List Devices
```bash
nfc-list
```
**Explanation:** Shows NFC readers and tags.

#### Read Card
```bash
nfc-mfclassic r b card.dump
```
**Explanation:** Reads Mifare Classic card.

#### Write Card
```bash
nfc-mfclassic w a card.dump
```
**Explanation:** Writes to Mifare Classic card.

### Complete Command Reference

#### nfc-list
| Command | Description | Example |
|---------|-------------|---------|
| `nfc-list` | List devices | `nfc-list` |
| `nfc-list -v` | Verbose | `nfc-list -v` |

#### nfc-mfclassic
| Command | Description | Example |
|---------|-------------|---------|
| `r` | Read card | `nfc-mfclassic r b dump` |
| `w` | Write card | `nfc-mfclassic w a dump` |
| `R` | Read with key A | `nfc-mfclassic R b dump` |
| `W` | Write with key A | `nfc-mfclassic W a dump` |
| `b` | Use key B | `nfc-mfclassic r b dump` |
| `a` | Use key A | `nfc-mfclassic r a dump` |

#### nfc-mfultralight
| Command | Description | Example |
|---------|-------------|---------|
| `r` | Read tag | `nfc-mfultralight r dump` |
| `w` | Write tag | `nfc-mfultralight w dump` |
| `dump` | Dump tag | `nfc-mfultralight dump` |

---

## Chapter 4: Intermediate Usage

### Advanced NFC Operations

#### Mifare Classic Operations
```bash
# Read with key A
nfc-mfclassic r a card.dump

# Read with key B
nfc-mfclassic r b card.dump

# Write with key A
nfc-mfclassic w a card.dump
```

#### Custom Keys
```bash
# Read with custom key
nfc-mfclassic R b card.dump -k keys.txt
```

### Scripting & Automation

#### Bash Scripting
```bash
#!/bin/bash
# Automated card operations
TIMESTAMP=$(date +%Y%m%d_%H%M%S)

echo "[*] Listing NFC devices..."
nfc-list

echo "[*] Reading card..."
nfc-mfclassic r b "card_${TIMESTAMP}.dump"

echo "[+] Card saved"
```

### Combining with Other Tools

#### With MFCUK
```bash
# Recover keys
mfcuk -C -R -v 3 -o keys.txt

# Read card
nfc-mfclassic R b card.dump
```

#### With MFOC
```bash
# Dump card
mfoc -r card.dump

# Analyze
nfc-list
```

---

## Chapter 5: Advanced Usage

### Advanced NFC Operations

#### Emulation
```bash
# Emulate tag
nfc-emulate -t 1 -u 0x04 card.dump
```

### Integration in Pentest Workflow

#### Phase 1: Discovery
```bash
# List devices
nfc-list
```

#### Phase 2: Reading
```bash
# Read card
nfc-mfclassic r b card.dump
```

#### Phase 3: Analysis
```bash
# Analyze dump
strings card.dump
hexdump -C card.dump
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: CTF Challenge
**Objective:** Read NFC card

**Steps:**
```bash
# 1. List devices
nfc-list

# 2. Read card
nfc-mfclassic r b card.dump

# 3. Analyze
strings card.dump
```

---

## Chapter 7: Detection & Defense

### How Defenders Detect libnfc

#### System Indicators
```
# NFC reader activity
# Card read/write attempts
```

### Mitigation Strategies

#### Strong Keys
```bash
# Use unique keys
# Avoid default keys
```

### Blue Team Response
1. **Use unique keys** - Per-card keys
2. **Monitor NFC readers** - Detect attacks
3. **Upgrade cards** - Use Mifare DESFire

---

## Chapter 8: Troubleshooting

### Common Errors

#### Error: "No NFC reader found"
**Cause:** Reader not connected
**Solution:**
```bash
# Check reader
lsusb
pcscd status
```

### FAQ

**Q: Is libnfc legal to use?**
A: Yes, libnfc is a legitimate NFC library. However, using it on cards you don't own may be illegal.

**Q: How do I update libnfc?**
A: `sudo apt update && sudo apt upgrade libnfc-utils`

---

## Resources

### Official Documentation
- [libnfc Homepage](https://nfc-tools.github.io/libnfc/)
- [GitHub Repository](https://github.com/nfc-tools/libnfc)
- [libnfc Documentation](https://nfc-tools.github.io/libnfc/documentation.html)

### Related Tools
- **mfcuk:** Key recovery
- **mfoc:** Card dump
- **mfterm:** Mifare terminal

### Practice Resources
- [HackTheBox](https://www.hackthebox.com/)
- [TryHackMe](https://tryhackme.com/)
