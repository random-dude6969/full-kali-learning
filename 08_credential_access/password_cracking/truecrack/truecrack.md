---
tool_name: "truecrack"
display_name: "TrueCrack"
mitre_tactic: "credential_access"
mitre_tactic_id: "TA0006"
mitre_subcategory: "password_cracking"
install_command: "sudo apt install truecrack"
version: "3.6"
github_repo: ""
kali_tools_page: "https://www.kali.org/tools/truecrack/"
difficulty_level: "intermediate"
requires_root: false
gui_available: false
tags: ["password", "cracking", "truecrypt", "volume", "encryption"]
related_tools: ["john", "hashcat", "hashcat"]
---

# TrueCrack

## Chapter 1: Introduction & Overview

### What is TrueCrack?
TrueCrack is a tool for cracking TrueCrypt volume passwords. It uses GPU acceleration to efficiently brute-force or dictionary-attack TrueCrypt encrypted volumes.

### History & Background
- **Created:** 2010 by TrueCrack Project
- **Maintained:** TrueCrack Project
- **License:** GNU General Public License v3
- **Purpose:** TrueCrypt password recovery

TrueCrack is specialized for TrueCrypt volume password recovery.

### When to Use This Tool
- **Forensics:** Recover TrueCrypt passwords
- **Penetration testing:** Test volume security
- **CTF challenges:** Solve encryption challenges
- **Incident response:** Access encrypted evidence

### Key Concepts
- **TrueCrypt:** Disk encryption software
- **Volume:** Encrypted container
- **Header:** Volume metadata
- **Keyfile:** Additional authentication factor

---

## Chapter 2: Installation & Setup

### System Requirements
- **GPU:** NVIDIA (CUDA) recommended
- **Disk Space:** ~10 MB
- **Privileges:** User-level

### Installation Methods

#### Method 1: APT (Recommended)
```bash
sudo apt update
sudo apt install truecrack
```

### Verification
```bash
truecrack --help
```

---

## Chapter 3: Basic Usage

### First Crack

#### Dictionary Attack
```bash
truecrack -t volume.tc -w wordlist.txt
```
**Output:** Attempts to crack TrueCrypt password using wordlist.

#### Brute Force Attack
```bash
truecrack -t volume.tc -b
```
**Output:** Tries all password combinations.

### Essential Commands

#### Dictionary Attack
```bash
truecrack -t volume.tc -w /usr/share/wordlists/rockyou.txt
```
**Explanation:** Tries words from wordlist against TrueCrypt volume.

#### Brute Force
```bash
truecrack -t volume.tc -b -l 8
```
**Explanation:** Tries all 8-character passwords.

### Complete Command Reference

#### Attack Options
| Flag | Description | Example |
|------|-------------|---------|
| `-t` | Target volume | `truecrack -t volume.tc` |
| `-w` | Wordlist file | `truecrack -t volume.tc -w wordlist.txt` |
| `-b` | Brute force | `truecrack -t volume.tc -b` |
| `-l` | Password length | `truecrack -t volume.tc -b -l 8` |

#### Output Options
| Flag | Description | Example |
|------|-------------|---------|
| `-v` | Verbose | `truecrack -v volume.tc` |
| `-s` | Speed | `truecrack -s volume.tc` |

---

## Chapter 4: Intermediate Usage

### Advanced Cracking Techniques

#### With Keyfiles
```bash
# Crack with keyfile
truecrack -t volume.tc -w wordlist.txt -k keyfile.key
```

#### GPU Acceleration
```bash
# Use GPU
truecrack -t volume.tc -w wordlist.txt -g
```

### Scripting & Automation

#### Bash Scripting
```bash
#!/bin/bash
# Automated TrueCrypt cracking
VOLUME="volume.tc"
WORDLIST="/usr/share/wordlists/rockyou.txt"

truecrack -t $VOLUME -w $WORDLIST
```

### Combining with Other Tools

#### With Hashcat
```bash
# Extract TrueCrypt hash
# Then crack with hashcat
hashcat -m 6211 hash.txt wordlist.txt
```

---

## Chapter 5: Advanced Usage

### Advanced Cracking Techniques

#### Custom Character Sets
```bash
# Brute force with specific charset
truecrack -t volume.tc -b -l 8 -c aA1
```

### Integration in Pentest Workflow

#### Phase 1: Discovery
```bash
# Find TrueCrypt volumes
find / -name "*.tc" 2>/dev/null
```

#### Phase 2: Cracking
```bash
# Crack volume password
truecrack -t volume.tc -w wordlist.txt
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: Forensic Investigation
**Objective:** Access TrueCrypt volume

**Steps:**
```bash
# 1. Identify TrueCrypt volume
file volume.tc

# 2. Crack password
truecrack -t volume.tc -w rockyou.txt

# 3. Mount volume with recovered password
```

---

## Chapter 7: Detection & Defense

### How Defenders Detect TrueCrack

#### System Indicators
```
# High GPU usage
# TrueCrypt volume access
```

### Mitigation Strategies

#### Strong Passwords
```bash
# Use complex passwords (20+ characters)
# Use keyfiles
# Use hidden volumes
```

### Blue Team Response
1. **Use strong passwords** - 20+ characters
2. **Use keyfiles** - Additional authentication
3. **Use hidden volumes** - Plausible deniability
4. **Monitor access** - Detect cracking attempts

---

## Chapter 8: Troubleshooting

### Common Errors

#### Error: "No GPU found"
**Cause:** No compatible GPU
**Solution:**
```bash
# Use CPU mode (slower)
truecrack -t volume.tc -w wordlist.txt
```

### FAQ

**Q: Is truecrack legal to use?**
A: Yes, truecrack is a legitimate security testing tool. However, using it without authorization is illegal.

**Q: Can truecrack crack VeraCrypt?**
A: No, truecrack is for TrueCrypt only. Use hashcat for VeraCrypt.

**Q: How do I improve speed?**
A: Use GPU acceleration and shorter passwords.

---

## Resources

### Official Documentation
- [TrueCrack Documentation](https://github.com/0x64746b/truecrack)

### Related Tools
- **hashcat:** Advanced password recovery
- **john:** John the Ripper
- **veracrypt:** TrueCrypt successor

### Practice Resources
- [HackTheBox](https://www.hackthebox.com/)
- [TryHackMe](https://tryhackme.com/)
