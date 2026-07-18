---
tool_name: "fcrackzip"
display_name: "Fcrackzip"
mitre_tactic: "credential_access"
mitre_tactic_id: "TA0006"
mitre_subcategory: "password_cracking"
install_command: "sudo apt install fcrackzip"
version: "1.0"
github_repo: ""
kali_tools_page: "https://www.kali.org/tools/fcrackzip/"
difficulty_level: "beginner"
requires_root: false
gui_available: false
tags: ["password", "cracking", "zip", "archive", "brute-force"]
related_tools: ["john", "hashcat", "unzip"]
---

# Fcrackzip

## Chapter 1: Introduction & Overview

### What is Fcrackzip?
Fcrackzip is a password cracker for ZIP archives. It uses brute-force or dictionary attacks to find the password for encrypted ZIP files.

### History & Background
- **Created:** 2005 by Hiroshi Iwatani
- **Maintained:** Hiroshi Iwatani
- **License:** BSD License
- **Purpose:** ZIP password cracking

Fcrackzip is a simple and effective ZIP password cracker.

### When to Use This Tool
- **Forensics:** Recover ZIP passwords
- **Penetration testing:** Test archive security
- **CTF challenges:** Solve password challenges
- **Incident response:** Access encrypted archives

### Key Concepts
- **Brute-force:** Try all possible combinations
- **Dictionary attack:** Try words from a wordlist
- **ZIP Archive:** Compressed file format
- **Encryption:** Password protection

---

## Chapter 2: Installation & Setup

### System Requirements
- **Disk Space:** ~1 MB
- **Privileges:** User-level

### Installation Methods

#### Method 1: APT (Recommended)
```bash
sudo apt update
sudo apt install fcrackzip
```

### Verification
```bash
fcrackzip --version
```

---

## Chapter 3: Basic Usage

### First Crack

#### Dictionary Attack
```bash
fcrackzip -u -D -p /usr/share/wordlists/rockyou.txt archive.zip
```
**Output:** Shows password if found.

#### Brute Force Attack
```bash
fcrackzip -u -b -c a -l 1-6 archive.zip
```
**Output:** Tries all combinations.

### Essential Commands

#### Dictionary Attack
```bash
fcrackzip -u -D -p wordlist.txt archive.zip
```
**Explanation:** Tries words from wordlist.

#### Brute Force (Letters)
```bash
fcrackzip -u -b -c a -l 1-6 archive.zip
```
**Explanation:** Tries lowercase letters, 1-6 chars.

#### Brute Force (Numbers)
```bash
fcrackzip -u -b -c 1 -l 1-6 archive.zip
```
**Explanation:** Tries numbers, 1-6 chars.

### Complete Flags Reference

#### Attack Options
| Flag | Description | Example |
|------|-------------|---------|
| `-b` | Brute force | `fcrackzip -b archive.zip` |
| `-D` | Dictionary | `fcrackzip -D -p wordlist.txt archive.zip` |
| `-c` | Charset | `fcrackzip -c a archive.zip` |
| `-l` | Length range | `fcrackzip -l 1-6 archive.zip` |

#### Output Options
| Flag | Description | Example |
|------|-------------|---------|
| `-u` | Unzip on success | `fcrackzip -u archive.zip` |
| `-v` | Verbose | `fcrackzip -v archive.zip` |
| `-p` | Password file | `fcrackzip -p wordlist.txt archive.zip` |

#### Charset Options
| Charset | Description | Example |
|---------|-------------|---------|
| `a` | Lowercase | `fcrackzip -c a archive.zip` |
| `A` | Uppercase | `fcrackzip -c A archive.zip` |
| `1` | Numbers | `fcrackzip -c 1 archive.zip` |
| `!` | Symbols | `fcrackzip -c ! archive.zip` |
| `aA1!` | All | `fcrackzip -c aA1! archive.zip` |

---

## Chapter 4: Intermediate Usage

### Advanced Cracking Techniques

#### Custom Length
```bash
# Try 4-8 character passwords
fcrackzip -u -b -c aA1 -l 4-8 archive.zip
```

#### Combined Attack
```bash
# Try dictionary first, then brute force
fcrackzip -u -D -p wordlist.txt archive.zip
fcrackzip -u -b -c aA1 -l 1-6 archive.zip
```

### Scripting & Automation

#### Bash Scripting
```bash
#!/bin/bash
# Automated cracking
ARCHIVE="archive.zip"
WORDLIST="/usr/share/wordlists/rockyou.txt"

echo "[*] Dictionary attack..."
fcrackzip -u -D -p $WORDLIST $ARCHIVE

echo "[*] Brute force attack..."
fcrackzip -u -b -c aA1 -l 1-6 $ARCHIVE
```

### Combining with Other Tools

#### With John
```bash
# Extract hash
zip2john archive.zip > hash.txt

# Crack with john
john hash.txt
```

#### With Hashcat
```bash
# Extract hash
zip2john archive.zip > hash.txt

# Convert to hashcat format
# Crack with hashcat
hashcat -m 17200 hash.txt rockyou.txt
```

### Advanced Configurations

#### Custom Charset
```bash
# Try specific characters
fcrackzip -u -b -c "abc123" -l 1-6 archive.zip
```

---

## Chapter 5: Advanced Usage

### Advanced Cracking Techniques

#### Multiple Archives
```bash
# Crack multiple files
for archive in *.zip; do
    fcrackzip -u -D -p wordlist.txt $archive
done
```

### Integration in Pentest Workflow

#### Phase 1: Discovery
```bash
# Find ZIP files
find / -name "*.zip" 2>/dev/null
```

#### Phase 2: Cracking
```bash
# Crack ZIP files
fcrackzip -u -D -p wordlist.txt archive.zip
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: CTF Challenge
**Objective:** Open password-protected ZIP

**Steps:**
```bash
# 1. Try dictionary attack
fcrackzip -u -D -p rockyou.txt archive.zip

# 2. Try brute force
fcrackzip -u -b -c aA1 -l 1-6 archive.zip

# 3. Extract files
unzip archive.zip
```

### Scenario 2: Forensics
**Objective:** Access encrypted archive

**Steps:**
```bash
# 1. Find ZIP files
find / -name "*.zip" 2>/dev/null

# 2. Crack password
fcrackzip -u -D -p rockyou.txt archive.zip

# 3. Extract evidence
unzip archive.zip
```

---

## Chapter 7: Detection & Defense

### How Defenders Detect Fcrackzip

#### System Indicators
```
# ZIP file access
# Password cracking attempts
```

#### Log Analysis
```bash
# Monitor file access
# Check for cracking tools
```

### Mitigation Strategies

#### Strong Passwords
```bash
# Use complex passwords (12+ characters)
# Use uppercase, lowercase, numbers, symbols
```

#### Encryption
```bash
# Use AES-256 encryption
# Avoid weak encryption methods
```

### Blue Team Response
1. **Use strong passwords** - Resist brute force
2. **Use strong encryption** - AES-256
3. **Monitor file access** - Detect cracking attempts

---

## Chapter 8: Troubleshooting

### Common Errors

#### Error: "File not found"
**Cause:** Wrong path
**Solution:**
```bash
ls -la archive.zip
fcrackzip -u -D -p wordlist.txt archive.zip
```

### Performance Optimization

#### Slow Cracking
```bash
# Use faster charset
fcrackzip -u -b -c a -l 1-4 archive.zip

# Use dictionary
fcrackzip -u -D -p wordlist.txt archive.zip
```

### FAQ

**Q: Is fcrackzip legal to use?**
A: Yes, fcrackzip is a legitimate security testing tool. However, using it without authorization is illegal. Always get written permission before testing.

**Q: How do I update fcrackzip?**
A: `sudo apt update && sudo apt upgrade fcrackzip`

**Q: Can fcrackzip crack RAR files?**
A: No, fcrackzip only works with ZIP archives.

---

## Resources

### Official Documentation
- [Fcrackzip Man Page](https://linux.die.net/man/1/fcrackzip)

### Related Tools
- **john:** John the Ripper
- **hashcat:** Advanced password recovery
- **zip2john:** ZIP hash extractor

### Practice Resources
- [HackTheBox](https://www.hackthebox.com/)
- [TryHackMe](https://tryhackme.com/)
