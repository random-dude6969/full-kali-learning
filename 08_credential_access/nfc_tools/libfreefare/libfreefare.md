---
tool_name: "libfreefare"
display_name: "libfreefare"
mitre_tactic: "credential_access"
mitre_tactic_id: "TA0006"
mitre_subcategory: "nfc_tools"
install_command: "sudo apt install libfreefare-bin"
version: "0.4.0"
github_repo: "https://github.com/nfc-tools/libfreefare"
kali_tools_page: "https://www.kali.org/tools/libfreefare/"
difficulty_level: "intermediate"
requires_root: false
gui_available: false
tags: ["nfc", "mifare", "rfid", "desfire", "ultralight"]
related_tools: ["libnfc", "mfcuk", "mfoc"]
---

# libfreefare

## Chapter 1: Introduction & Overview

### What is libfreefare?
libfreefare is a library for working with Mifare RFID tags. It provides tools for Mifare Classic, Ultralight, and DESFire cards.

### History & Background
- **Created:** 2009 by nfc-tools
- **Maintained:** nfc-tools
- **License:** GNU Lesser General Public License v2
- **Purpose:** Mifare tag operations

libfreefare supports multiple Mifare card types.

### When to Use This Tool
- **RFID security:** Work with Mifare cards
- **Access control:** Test card systems
- **CTF challenges:** Solve RFID challenges
- **Development:** Build NFC applications

### Key Concepts
- **Mifare Classic:** Legacy card type
- **Mifare Ultralight:** Low-cost card
- **Mifare DESFire:** Secure card
- **NDEF:** NFC Data Exchange Format

### Available Tools
| Tool | Description |
|------|-------------|
| `mifare-classic-format` | Format Classic cards |
| `mifare-desfire-info` | DESFire card info |
| `mifare-desfire-create-ndef` | Create NDEF data |

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
sudo apt install libfreefare-bin
```

### Verification
```bash
mifare-classic-format --help
```

---

## Chapter 3: Basic Usage

### First Use

#### Format Mifare Classic
```bash
mifare-classic-format
```
**Output:** Formats card to factory defaults.

#### DESFire Info
```bash
mifare-desfire-info
```
**Output:** Shows DESFire card information.

### Essential Commands

#### Format Card
```bash
mifare-classic-format
```
**Explanation:** Formats Mifare Classic card.

#### DESFire Info
```bash
mifare-desfire-info
```
**Explanation:** Reads DESFire card info.

### Complete Command Reference

#### mifare-classic-format
| Command | Description | Example |
|---------|-------------|---------|
| Default | Format card | `mifare-classic-format` |

#### mifare-desfire-info
| Command | Description | Example |
|---------|-------------|---------|
| Default | Card info | `mifare-desfire-info` |

#### mifare-desfire-create-ndef
| Command | Description | Example |
|---------|-------------|---------|
| `-t` | Text record | `mifare-desfire-create-ndef -t "Hello"` |
| `-u` | URL record | `mifare-desfire-create-ndef -u "http://example.com"` |

---

## Chapter 4: Intermediate Usage

### Advanced Operations

#### Format Multiple Cards
```bash
# Format card
mifare-classic-format

# Confirm
echo "y" | mifare-classic-format
```

#### Create NDEF Data
```bash
# Create text NDEF
mifare-desfire-create-ndef -t "Hello World"

# Create URL NDEF
mifare-desfire-create-ndef -u "http://example.com"
```

### Scripting & Automation

#### Bash Scripting
```bash
#!/bin/bash
# Format card
echo "[*] Formatting Mifare Classic card..."
mifare-classic-format

# Read DESFire info
echo "[*] Reading DESFire card..."
mifare-desfire-info
```

### Combining with Other Tools

#### With libnfc
```bash
# List cards
nfc-list

# Format
mifare-classic-format

# Read
nfc-mfclassic r b card.dump
```

---

## Chapter 5: Advanced Usage

### Advanced Operations

#### NDEF Programming
```bash
# Create custom NDEF
mifare-desfire-create-ndef -t "Custom Data"

# Create multi-record NDEF
mifare-desfire-create-ndef -t "Text" -u "http://example.com"
```

### Integration in Pentest Workflow

#### Phase 1: Discovery
```bash
# List cards
nfc-list

# Check card type
mifare-desfire-info
```

#### Phase 2: Manipulation
```bash
# Format card
mifare-classic-format

# Create NDEF
mifare-desfire-create-ndef -t "payload"
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: CTF Challenge
**Objective:** Program NFC card

**Steps:**
```bash
# 1. Check card
nfc-list
mifare-desfire-info

# 2. Format if needed
mifare-classic-format

# 3. Write NDEF
mifare-desfire-create-ndef -t "flag{xxx}"
```

---

## Chapter 7: Detection & Defense

### How Defenders Detect libfreefare

#### System Indicators
```
# Card formatting attempts
# NDEF programming
```

### Mitigation Strategies

#### Secure Cards
```bash
# Use DESFire EV1/EV2
# Implement proper access controls
```

### Blue Team Response
1. **Use secure cards** - DESFire
2. **Implement access controls**
3. **Monitor card operations**

---

## Chapter 8: Troubleshooting

### Common Errors

#### Error: "No card found"
**Cause:** No card on reader
**Solution:**
```bash
# Place card on reader
nfc-list
```

### FAQ

**Q: Is libfreefare legal to use?**
A: Yes, libfreefare is a legitimate NFC library. However, using it on cards you don't own may be illegal.

**Q: How do I update libfreefare?**
A: `sudo apt update && sudo apt upgrade libfreefare-bin`

---

## Resources

### Official Documentation
- [libfreefare GitHub](https://github.com/nfc-tools/libfreefare)
- [libfreefare Wiki](https://github.com/nfc-tools/libfreefare/wiki)

### Related Tools
- **libnfc:** NFC library
- **mfcuk:** Key recovery
- **mfoc:** Card dump

### Practice Resources
- [HackTheBox](https://www.hackthebox.com/)
- [TryHackMe](https://tryhackme.com/)
