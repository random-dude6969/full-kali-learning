---
tool_name: "crackle"
display_name: "Crackle"
mitre_tactic: "credential_access"
mitre_tactic_id: "TA0006"
mitre_subcategory: "password_cracking"
install_command: "sudo apt install crackle"
version: "1.0"
github_repo: "https://github.com/evilsocket/crackle"
kali_tools_page: "https://www.kali.org/tools/crackle/"
difficulty_level: "intermediate"
requires_root: false
gui_available: false
tags: ["password", "cracking", "encryption", "zip", "rar"]
related_tools: ["fcrackzip", "john", "hashcat"]
---

# Crackle

## Chapter 1: Introduction & Overview

### What is Crackle?
Crackle is a tool for cracking password-protected files including ZIP, RAR, and 7z archives. It uses a combination of dictionary and brute-force attacks to recover passwords from encrypted archives.

### History & Background
- **Created:** 2018 by evilsocket
- **Maintained:** evilsocket
- **License:** MIT License
- **Purpose:** Archive password recovery

Crackle provides an efficient way to crack passwords from various archive formats.

### When to Use This Tool
- **Forensics:** Recover passwords from encrypted archives
- **Penetration testing:** Test archive security
- **CTF challenges:** Solve password challenges
- **Incident response:** Access encrypted evidence

### Key Concepts
- **Archive:** Compressed file format
- **Encryption:** Password protection
- **Dictionary Attack:** Try words from wordlist
- **Brute-force:** Try all combinations

---

## Chapter 2: Installation & Setup

### System Requirements
- **Disk Space:** ~5 MB
- **Privileges:** User-level

### Installation Methods

#### Method 1: APT (Recommended)
```bash
sudo apt update
sudo apt install crackle
```

#### Method 2: From Source
```bash
git clone https://github.com/evilsocket/crackle.git
cd crackle
go build
```

### Verification
```bash
crackle --help
```

---

## Chapter 3: Basic Usage

### First Crack

#### Dictionary Attack
```bash
crackle -d wordlist.txt archive.zip
```
**Output:** Attempts to crack ZIP password using wordlist.

#### Brute Force Attack
```bash
crackle -b archive.zip
```
**Output:** Tries all password combinations.

### Essential Commands

#### Dictionary Attack
```bash
crackle -d /usr/share/wordlists/rockyou.txt archive.zip
```
**Explanation:** Tries words from wordlist against archive.

#### Brute Force
```bash
crackle -b archive.zip
```
**Explanation:** Tries all password combinations.

### Complete Command Reference

#### Attack Options
| Flag | Description | Example |
|------|-------------|---------|
| `-d` | Dictionary file | `crackle -d wordlist.txt archive.zip` |
| `-b` | Brute force | `crackle -b archive.zip` |
| `-r` | RAR file | `crackle -r archive.rar` |

#### Output Options
| Flag | Description | Example |
|------|-------------|---------|
| `-v` | Verbose | `crackle -v archive.zip` |
| `-o` | Output file | `crackle -o results.txt archive.zip` |

---

## Chapter 4: Intermediate Usage

### Advanced Cracking Techniques

#### Multiple Attack Types
```bash
# Try dictionary first
crackle -d wordlist.txt archive.zip

# Then brute force
crackle -b archive.zip
```

#### Specific Archive Types
```bash
# ZIP
crackle archive.zip

# RAR
crackle archive.rar

# 7z
crackle archive.7z
```

### Scripting & Automation

#### Bash Scripting
```bash
#!/bin/bash
# Automated cracking
ARCHIVE="archive.zip"
WORDLIST="/usr/share/wordlists/rockyou.txt"

echo "[*] Dictionary attack..."
crackle -d $WORDLIST $ARCHIVE

echo "[*] Brute force attack..."
crackle -b $ARCHIVE
```

### Combining with Other Tools

#### With Fcrackzip
```bash
# Try with crackle first
crackle -d wordlist.txt archive.zip

# If not found, try fcrackzip
fcrackzip -u -D -p wordlist.txt archive.zip
```

---

## Chapter 5: Advanced Usage

### Advanced Cracking Techniques

#### Large Archives
```bash
# For large files, use dictionary first
crackle -d wordlist.txt large_archive.zip
```

### Integration in Pentest Workflow

#### Phase 1: Discovery
```bash
# Find encrypted archives
find / -name "*.zip" -o -name "*.rar" -o -name "*.7z" 2>/dev/null
```

#### Phase 2: Cracking
```bash
# Crack archives
crackle -d wordlist.txt archive.zip
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: CTF Challenge
**Objective:** Open password-protected archive

**Steps:**
```bash
# 1. Check archive type
file archive.zip

# 2. Try dictionary attack
crackle -d rockyou.txt archive.zip

# 3. Try brute force if needed
crackle -b archive.zip
```

---

## Chapter 7: Detection & Defense

### How Defenders Detect Crackle

#### System Indicators
```
# Archive access attempts
# Password cracking activity
```

### Mitigation Strategies

#### Strong Passwords
```bash
# Use complex passwords (12+ characters)
# Use strong encryption (AES-256)
```

### Blue Team Response
1. **Use strong passwords** - Resist brute force
2. **Use strong encryption** - AES-256
3. **Monitor archive access** - Detect cracking attempts

---

## Chapter 8: Troubleshooting

### Common Errors

#### Error: "No password found"
**Cause:** Password not in wordlist or too complex
**Solution:**
```bash
# Try larger wordlist
crackle -d /usr/share/seclists/Passwords/Common-Credentials/10k-most-common.txt archive.zip

# Try brute force
crackle -b archive.zip
```

### FAQ

**Q: Is crackle legal to use?**
A: Yes, crackle is a legitimate security testing tool. However, using it without authorization is illegal.

**Q: What archive types are supported?**
A: ZIP, RAR, and 7z archives.

**Q: Can crackle handle large files?**
A: Yes, but performance may vary depending on file size and encryption strength.

---

## Resources

### Official Documentation
- [Crackle GitHub](https://github.com/evilsocket/crackle)

### Related Tools
- **fcrackzip:** ZIP password cracker
- **john:** John the Ripper
- **hashcat:** Advanced password recovery

### Practice Resources
- [HackTheBox](https://www.hackthebox.com/)
- [TryHackMe](https://tryhackme.com/)
