---
tool_name: "mfterm"
display_name: "Mfterm"
mitre_tactic: "credential_access"
mitre_tactic_id: "TA0006"
mitre_subcategory: "nfc_tools"
install_command: "sudo apt install mfterm"
version: "0.1"
github_repo: "https://github.com/jbtronics/mfterm"
kali_tools_page: "https://www.kali.org/tools/mfterm/"
difficulty_level: "beginner-to-intermediate"
requires_root: false
gui_available: false
tags: ["nfc", "mifare", "rfid", "terminal", "interactive"]
related_tools: ["mfcuk", "mfoc", "libnfc"]
---

# Mfterm

## Chapter 1: Introduction & Overview

### What is Mfterm?
Mfterm is a terminal-based tool for interacting with Mifare Classic RFID cards. It provides an interactive shell for reading, writing, and manipulating card data.

### History & Background
- **Created:** 2012 by jbtronics
- **Maintained:** jbtronics
- **License:** GNU General Public License v2
- **Purpose:** Mifare Classic interaction

Mfterm provides interactive card manipulation.

### When to Use This Tool
- **RFID security:** Interact with Mifare cards
- **Access control:** Test card systems
- **CTF challenges:** Solve RFID challenges
- **Security assessments:** Test card security

### Key Concepts
- **Mifare Classic:** RFID card type
- **Interactive shell:** Command-line interface
- **Sector:** Memory division
- **Block:** Data unit

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
sudo apt install mfterm
```

### Verification
```bash
mfterm --help
```

---

## Chapter 3: Basic Usage

### First Session

#### Start Mfterm
```bash
mfterm
```
**Output:** Opens interactive shell.

### Essential Commands

#### Interactive Mode
```bash
mfterm
```
**Explanation:** Opens interactive shell.

#### Read Card
```bash
mfterm -r
```
**Explanation:** Reads card data.

### Command Reference

#### Card Operations
| Command | Description | Example |
|---------|-------------|---------|
| `read` | Read card | `read` |
| `write` | Write card | `write` |
| `dump` | Dump card | `dump card.dump` |
| `load` | Load dump | `load card.dump` |

#### Key Operations
| Command | Description | Example |
|---------|-------------|---------|
| `keys` | Show keys | `keys` |
| `setkey` | Set key | `setkey A 0xFFFFFFFFFFFF` |
| `loadkeys` | Load keys | `loadkeys keys.txt` |

#### Navigation
| Command | Description | Example |
|---------|-------------|---------|
| `cd` | Change sector | `cd 1` |
| `ls` | List blocks | `ls` |
| `cat` | Read block | `cat 0` |
| `echo` | Write block | `echo "data" > 0` |

#### Utility
| Command | Description | Example |
|---------|-------------|---------|
| `help` | Show help | `help` |
| `exit` | Exit shell | `exit` |

---

## Chapter 4: Intermediate Usage

### Advanced Card Manipulation

#### Custom Keys
```bash
# Set custom key
setkey A 0xA0A1A2A3A4A5

# Load keys from file
loadkeys custom_keys.txt
```

#### Sector Navigation
```bash
# Change sector
cd 1

# List blocks
ls

# Read specific block
cat 0
```

### Scripting & Automation

#### Bash Scripting
```bash
#!/bin/bash
# Automated card reading
mfterm -r -o card_dump.txt
```

### Combining with Other Tools

#### With MFOC
```bash
# Dump card
mfoc -r card.dump

# Analyze with mfterm
mfterm
load card.dump
```

---

## Chapter 5: Advanced Usage

### Advanced Card Manipulation

#### Write Data
```bash
# Write to block
echo "Hello" > 4

# Write hex data
echo "48656C6C6F" > 4
```

### Integration in Pentest Workflow

#### Phase 1: Read Card
```bash
mfterm
read
```

#### Phase 2: Analyze
```bash
keys
ls
cat 0
```

#### Phase 3: Dump
```bash
dump card.dump
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: CTF Challenge
**Objective:** Read Mifare card

**Steps:**
```bash
# 1. Start mfterm
mfterm

# 2. Read card
read

# 3. Check keys
keys

# 4. Dump card
dump card.dump

# 5. Analyze
cat 4
```

---

## Chapter 7: Detection & Defense

### How Defenders Detect Mfterm

#### System Indicators
```
# Unusual NFC reader activity
# Card manipulation attempts
```

### Mitigation Strategies

#### Strong Keys
```bash
# Use unique keys per card
# Avoid default keys
```

### Blue Team Response
1. **Use unique keys** - Per-card keys
2. **Monitor NFC readers** - Detect attacks
3. **Upgrade cards** - Use Mifare DESFire

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

**Q: Is mfterm legal to use?**
A: Yes, mfterm is a legitimate security testing tool. However, using it on cards you don't own is illegal.

**Q: How do I update mfterm?**
A: `sudo apt update && sudo apt upgrade mfterm`

---

## Resources

### Official Documentation
- [Mfterm GitHub](https://github.com/jbtronics/mfterm)
- [Mfterm Wiki](https://github.com/jbtronics/mfterm/wiki)

### Related Tools
- **mfcuk:** Key recovery
- **mfoc:** Card dump
- **libnfc:** NFC library

### Practice Resources
- [HackTheBox](https://www.hackthebox.com/)
- [TryHackMe](https://tryhackme.com/)
