---
tool_name: "hashid"
display_name: "HashID"
mitre_tactic: "credential_access"
mitre_tactic_id: "TA0006"
mitre_subcategory: "hash_identification"
install_command: "sudo apt install hashid"
version: "3.1.3"
github_repo: "https://github.com/psypanda/hashid"
kali_tools_page: "https://www.kali.org/tools/hashid/"
difficulty_level: "beginner"
requires_root: false
gui_available: false
tags: ["hash", "identification", "forensics", "analysis"]
related_tools: ["hash-identifier", "hashcat", "john"]
---

# HashID

## Chapter 1: Introduction & Overview

### What is HashID?
HashID is a tool to identify the type of hash and the algorithm used to generate it. It supports over 220 hash types.

### History & Background
- **Created:** 2012 by psypanda
- **Maintained:** psypanda
- **License**: MIT License
- **Purpose:** Hash type identification

HashID is essential for determining how to crack hashes.

### When to Use This Tool
- **Forensics:** Identify hash types
- **Penetration testing:** Determine cracking methods
- **CTF challenges:** Solve hash puzzles
- **Incident response:** Analyze captured hashes

### Key Concepts
- **Hash:** Cryptographic one-way function
- **Algorithm:** Hash function used (MD5, SHA1, etc.)
- **Salt:** Random data added to hash
- **Mode:** Hashcat mode number

### Supported Hash Types
| Hash Type | Example |
|-----------|---------|
| MD5 | 5d41402abc4b2a76b9719d911017c592 |
| SHA-1 | aaf4c61ddcc5e8a2dabede0f3b482cd9aea9434d |
| SHA-256 | 2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c... |
| SHA-512 | cf83e1357eefb8bdf1542850d66d8007d620e405... |
| NTLM | aad3b435b51404eeaad3b435b51404ee |
| bcrypt | $2a$10$... |

---

## Chapter 2: Installation & Setup

### System Requirements
- **Disk Space:** ~1 MB
- **Privileges:** User-level

### Installation Methods

#### Method 1: APT (Recommended)
```bash
sudo apt update
sudo apt install hashid
```

#### Method 2: From Source
```bash
git clone https://github.com/psypanda/hashid.git
cd hashid
python setup.py install
```

### Verification
```bash
hashid --version
```

---

## Chapter 3: Basic Usage

### First Identification

#### Identify Hash
```bash
hashid '5d41402abc4b2a76b9719d911017c592'
```
**Output:** Shows possible hash types.

### Essential Commands

#### Basic Identification
```bash
hashid 'hash_string'
```
**Explanation:** Identifies hash type.

#### Show Hashcat Modes
```bash
hashid -m 'hash_string'
```
**Explanation:** Shows hashcat mode numbers.

#### Show John Formats
```bash
hashid -j 'hash_string'
```
**Explanation:** Shows John the Ripper format names.

### Complete Flags Reference

#### Output Options
| Flag | Description | Example |
|------|-------------|---------|
| `-m` | Hashcat modes | `hashid -m 'hash'` |
| `-j` | John formats | `hashid -j 'hash'` |
| `-f` | Force identification | `hashid -f 'hash'` |
| `-e` | Extended output | `hashid -e 'hash'` |
| `-o` | Output file | `hashid -o results.txt 'hash'` |
| `-r` | Raw output | `hashid -r 'hash'` |
| `-d` | Dictionary | `hashid -d 'hash'` |
| `-p` | Prototype | `hashid -p 'hash'` |
| `-c` | Color | `hashid -c 'hash'` |
| `-n` | No color | `hashid -n 'hash'` |

---

## Chapter 4: Intermediate Usage

### Advanced Identification

#### Multiple Hashes
```bash
# From file
hashid -m -f hashes.txt

# Multiple arguments
hashid -m 'hash1' 'hash2' 'hash3'
```

#### Specific Format
```bash
# Only hashcat modes
hashid -m 'hash_string'

# Only John formats
hashid -j 'hash_string'
```

### Scripting & Automation

#### Bash Scripting
```bash
#!/bin/bash
# Identify hashes from file
HASHFILE="hashes.txt"

while IFS= read -r hash; do
    echo "[*] Hash: $hash"
    hashid -m "$hash"
    echo ""
done < "$HASHFILE"
```

#### Python Scripting
```python
#!/usr/bin/env python3
import subprocess

def identify_hash(hash_string):
    """Identify hash type"""
    cmd = f"hashid -m '{hash_string}'"
    result = subprocess.run(cmd, shell=True, capture_output=True, text=True)
    return result.stdout

# Identify hash
output = identify_hash("5d41402abc4b2a76b9719d911017c592")
print(output)
```

### Combining with Other Tools

#### With Hashcat
```bash
# Identify hash
hashid -m 'hash_string'

# Use identified mode
hashcat -m 0 'hash_string' rockyou.txt
```

#### With John
```bash
# Identify hash
hashid -j 'hash_string'

# Use identified format
john --format=raw-md5 hash.txt
```

### Advanced Configurations

#### Output Formats
```bash
# Save to file
hashid -m -o results.txt 'hash_string'

# Raw output (no colors)
hashid -r -m 'hash_string'
```

---

## Chapter 5: Advanced Usage

### Advanced Identification Techniques

#### Hash Analysis
```bash
# Analyze hash properties
hashid -e -m 'hash_string'

# Check for salt
hashid -e 'hash_string'
```

### Integration in Pentest Workflow

#### Phase 1: Collection
```bash
# Collect hashes from target
# SAM database, /etc/shadow, etc.
```

#### Phase 2: Identification
```bash
# Identify hash types
hashid -m -f collected_hashes.txt
```

#### Phase 3: Cracking
```bash
# Use identified modes
hashcat -m [mode] hash.txt rockyou.txt
john --format=[format] hash.txt
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: CTF Challenge
**Objective:** Identify and crack hash

**Steps:**
```bash
# 1. Identify hash
hashid -m '5d41402abc4b2a76b9719d911017c592'
# Output: MD5 [HC: 0]

# 2. Crack with hashcat
hashcat -m 0 '5d41402abc4b2a76b9719d911017c592' rockyou.txt

# 3. Or with john
john --format=raw-md5 hash.txt
```

### Scenario 2: Penetration Test
**Objective:** Analyze captured hashes

**Steps:**
```bash
# 1. Collect hashes
cat /etc/shadow > hashes.txt

# 2. Identify types
hashid -m -f hashes.txt

# 3. Crack each type
hashcat -m [mode] hash.txt rockyou.txt
```

---

## Chapter 7: Detection & Defense

### How Defenders Detect HashID

#### System Indicators
```
# Hash analysis activity
# Password file access
```

#### Log Analysis
```bash
# Monitor file access
# Check for hash tools
```

### Mitigation Strategies

#### Strong Hashing
```bash
# Use bcrypt, Argon2
# Use high work factors
# Implement proper salting
```

### Blue Team Response
1. **Use strong hashing** - bcrypt, Argon2
2. **Implement proper salting** - Unique salts per password
3. **Monitor hash access** - Detect analysis attempts

---

## Chapter 8: Troubleshooting

### Common Errors

#### Error: "Unknown hash"
**Cause:** Hash not recognized
**Solution:**
```bash
# Try extended mode
hashid -e 'hash_string'

# Try force mode
hashid -f 'hash_string'
```

### FAQ

**Q: Is hashid legal to use?**
A: Yes, hashid is a legitimate security testing tool. However, using it for unauthorized purposes is illegal.

**Q: How do I update hashid?**
A: `sudo apt update && sudo apt upgrade hashid`

**Q: How many hash types does hashid support?**
A: Over 220 hash types.

---

## Resources

### Official Documentation
- [HashID GitHub](https://github.com/psypanda/hashid)
- [HashID Wiki](https://github.com/psypanda/hashid/wiki)

### Related Tools
- **hash-identifier:** Hash identifier
- **hashcat:** Password cracker
- **john:** John the Ripper

### Practice Resources
- [CrackMe Archive](https://crackmes.one/)
- [HackTheBox](https://www.hackthebox.com/)
