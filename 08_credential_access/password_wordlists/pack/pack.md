---
tool_name: "pack"
display_name: "PACK"
mitre_tactic: "credential_access"
mitre_tactic_id: "TA0006"
mitre_subcategory: "password_wordlists"
install_command: "sudo apt install pack"
version: "0.0.4"
github_repo: "https://github.com/iphelix/pack"
kali_tools_page: "https://www.kali.org/tools/pack/"
difficulty_level: "intermediate"
requires_root: false
gui_available: false
tags: ["wordlist", "analysis", "password", "statistics", "hashcat"]
related_tools: ["crunch", "hashcat", "john"]
---

# PACK

## Chapter 1: Introduction & Overview

### What is PACK?
PACK (Password Analysis and Cracking Kit) is a tool for analyzing password hashes and generating statistics about password patterns. It helps understand password behavior and optimize cracking strategies.

### History & Background
- **Created:** 2012 by iphelix
- **Maintained:** iphelix
- **License:** GNU General Public License v2
- **Purpose:** Password analysis and statistics

PACK provides insights into password patterns for better cracking strategies.

### When to Use This Tool
- **Password analysis:** Understand password patterns
- **Cracking optimization:** Choose best attack methods
- **Security assessments:** Evaluate password policies
- **Forensics:** Analyze captured hashes

### Key Concepts
- **Mask:** Password pattern
- **Charset:** Character set
- **Statistics:** Password behavior analysis
- **Optimization:** Improve cracking efficiency

---

## Chapter 2: Installation & Setup

### System Requirements
- **Python:** 2.7 or 3.x
- **Disk Space:** ~5 MB
- **Privileges:** User-level

### Installation Methods

#### Method 1: APT (Recommended)
```bash
sudo apt update
sudo apt install pack
```

#### Method 2: From Source
```bash
git clone https://github.com/iphelix/pack.git
cd pack
python3 setup.py install
```

### Verification
```bash
pack --help
```

---

## Chapter 3: Basic Usage

### First Analysis

#### Analyze Hashes
```bash
pack -i hashes.txt
```
**Output:** Shows password statistics and patterns.

### Essential Commands

#### Basic Analysis
```bash
pack -i hashes.txt
```
**Explanation:** Analyzes hashes and shows statistics.

#### Generate Mask
```bash
pack -i hashes.txt -o masks.txt
```
**Explanation:** Generates mask file for hashcat.

### Complete Command Reference

#### Input Options
| Flag | Description | Example |
|------|-------------|---------|
| `-i` | Input file | `pack -i hashes.txt` |

#### Output Options
| Flag | Description | Example |
|------|-------------|---------|
| `-o` | Output file | `pack -i hashes.txt -o masks.txt` |
| `-t` | Top masks | `pack -i hashes.txt -t 10` |

---

## Chapter 4: Intermediate Usage

### Advanced Analysis

#### Generate Hashcat Masks
```bash
# Generate top 10 masks
pack -i hashes.txt -t 10 -o masks.txt

# Use with hashcat
hashcat -a 3 -m 0 hashes.txt masks.txt
```

#### Statistics Analysis
```bash
# Detailed statistics
pack -i hashes.txt -v
```

### Scripting & Automation

#### Bash Scripting
```bash
#!/bin/bash
# Analyze hashes
HASH_FILE="hashes.txt"

echo "[*] Analyzing hashes..."
pack -i $HASH_FILE -o masks.txt

echo "[*] Top masks:"
pack -i $HASH_FILE -t 10
```

### Combining with Other Tools

#### With Hashcat
```bash
# Generate masks
pack -i hashes.txt -o masks.txt

# Use with hashcat
hashcat -a 3 -m 0 hashes.txt masks.txt
```

---

## Chapter 5: Advanced Usage

### Advanced Analysis

#### Custom Analysis
```bash
# Analyze specific hash type
pack -i hashes.txt -m 0
```

### Integration in Pentest Workflow

#### Phase 1: Hash Collection
```bash
# Collect hashes
secretsdump.py user:pass@target > hashes.txt
```

#### Phase 2: Analysis
```bash
# Analyze patterns
pack -i hashes.txt -o masks.txt
```

#### Phase 3: Cracking
```bash
# Use generated masks
hashcat -a 3 -m 0 hashes.txt masks.txt
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: Password Audit
**Objective:** Analyze password strength

**Steps:**
```bash
# 1. Collect password hashes
unshadow /etc/passwd /etc/shadow > hashes.txt

# 2. Analyze patterns
pack -i hashes.txt -o masks.txt

# 3. Use insights for better cracking
```

---

## Chapter 7: Detection & Defense

### How Defenders Detect PACK

#### System Indicators
```
# Hash analysis activity
# Password pattern analysis
```

### Mitigation Strategies

#### Strong Passwords
```bash
# Enforce complexity
# Ban common patterns
# Use password filters
```

### Blue Team Response
1. **Enforce strong passwords** - 12+ characters
2. **Ban common patterns** - Use password filters
3. **Monitor analysis** - Detect hash access

---

## Chapter 8: Troubleshooting

### Common Errors

#### Error: "No hashes loaded"
**Cause:** Invalid hash format
**Solution:**
```bash
# Check hash format
head -1 hashes.txt

# Specify hash type
pack -i hashes.txt -m 0
```

### FAQ

**Q: Is pack legal to use?**
A: Yes, pack is a legitimate security testing tool. However, using it without authorization is illegal.

**Q: What hash types are supported?**
A: Most common hash types including MD5, SHA1, NTLM.

**Q: How do I use generated masks?**
A: Use with hashcat: `hashcat -a 3 -m 0 hashes.txt masks.txt`

---

## Resources

### Official Documentation
- [PACK GitHub](https://github.com/iphelix/pack)

### Related Tools
- **hashcat:** Advanced password recovery
- **john:** John the Ripper
- **crunch:** Wordlist generator

### Practice Resources
- [HackTheBox](https://www.hackthebox.com/)
- [TryHackMe](https://tryhackme.com/)
