---
tool_name: "hashcat"
display_name: "Hashcat"
mitre_tactic: "credential_access"
mitre_tactic_id: "TA0006"
mitre_subcategory: "password_cracking"
install_command: "sudo apt install hashcat"
version: "6.2.6"
github_repo: "https://github.com/hashcat/hashcat"
kali_tools_page: "https://www.kali.org/tools/hashcat/"
difficulty_level: "intermediate-to-advanced"
requires_root: false
gui_available: false
tags: ["password", "cracking", "gpu", "hash", "offline"]
related_tools: ["john", "hcxdumptool", "hashid"]
---

# Hashcat

## Chapter 1: Introduction & Overview

### What is Hashcat?
Hashcat is the world's fastest and most advanced password recovery utility, supporting five unique modes of attack for over 300 highly-optimized hashing algorithms. It's designed to leverage GPU acceleration for maximum performance.

### History & Background
- **Created:** 2009 by Jens "atom" Steube
- **Maintained:** Hashcat Project
- **License:** MIT License
- **Purpose:** Advanced password recovery using GPU acceleration

Hashcat is the industry standard for offline password cracking due to its unmatched speed and support for numerous hash types.

### When to Use This Tool
- **Password auditing:** Test password strength
- **Penetration testing:** Crack captured hashes
- **Forensics:** Recover passwords from systems
- **CTF challenges:** Solve password cracking challenges

### Key Concepts
- **Hash Mode (-m):** Algorithm identifier
- **Attack Mode (-a):** Attack strategy (straight, combination, brute-force, etc.)
- **Rules:** Mutations applied to wordlist words
- **Masks:** Pattern-based password generation
- **Charsets:** Custom character sets

---

## Chapter 2: Installation & Setup

### System Requirements
- **GPU:** NVIDIA (CUDA) or AMD (OpenCL) recommended
- **RAM:** 4 GB minimum (8 GB+ recommended)
- **Disk Space:** ~50 MB

### Installation Methods

#### Method 1: APT (Recommended)
```bash
sudo apt update
sudo apt install hashcat
```

#### Method 2: From Source
```bash
git clone https://github.com/hashcat/hashcat.git
cd hashcat
make
sudo make install
```

### Verification
```bash
hashcat --version
# Output: v6.2.6
```

---

## Chapter 3: Basic Usage

### First Crack
```bash
hashcat -m 0 hashes.txt rockyou.txt
```
**Output:** Attempts to crack MD5 hashes using rockyou.txt wordlist.

### Essential Commands

#### MD5 Cracking
```bash
hashcat -m 0 hashes.txt rockyou.txt
```
**Explanation:** Cracks MD5 hashes with straight (dictionary) attack.

#### Show Results
```bash
hashcat -m 0 hashes.txt rockyou.txt --show
```
**Explanation:** Displays cracked passwords.

#### Identify Hash
```bash
hashid '$1$salt$hash'
# or
hash-identifier
```

### Complete Flags Reference

#### Hash Mode (-m)
| Mode | Hash Type | Example |
|------|-----------|---------|
| 0 | MD5 | `hashcat -m 0 hashes.txt rockyou.txt` |
| 100 | SHA1 | `hashcat -m 100 hashes.txt rockyou.txt` |
| 1400 | SHA2-256 | `hashcat -m 1400 hashes.txt rockyou.txt` |
| 1700 | SHA2-512 | `hashcat -m 1700 hashes.txt rockyou.txt` |
| 3200 | bcrypt | `hashcat -m 3200 hashes.txt rockyou.txt` |
| 1000 | NTLM | `hashcat -m 1000 hashes.txt rockyou.txt` |
| 5600 | NetNTLMv2 | `hashcat -m 5600 hashes.txt rockyou.txt` |
| 1800 | sha512crypt | `hashcat -m 1800 hashes.txt rockyou.txt` |
| 7400 | sha256crypt | `hashcat -m 7400 hashes.txt rockyou.txt` |
| 500 | md5crypt | `hashcat -m 500 hashes.txt rockyou.txt` |

#### Attack Mode (-a)
| Mode | Name | Description | Example |
|------|------|-------------|---------|
| 0 | Straight | Dictionary attack | `hashcat -a 0 hashes.txt rockyou.txt` |
| 1 | Combination | Combine words | `hashcat -a 1 hashes.txt word1.txt word2.txt` |
| 3 | Brute-force | Try all combinations | `hashcat -a 3 hashes.txt ?a?a?a?a?a?a` |
| 6 | Hybrid Wordlist+Mask | Wordlist + mask | `hashcat -a 6 hashes.txt rockyou.txt ?d?d?d` |
| 7 | Hybrid Mask+Wordlist | Mask + wordlist | `hashcat -a 7 hashes.txt ?d?d?d rockyou.txt` |
| 8 | PIN | PIN attack | `hashcat -a 3 hashes.txt ?d?d?d?d` |

#### Input/Output Options
| Flag | Description | Example |
|------|-------------|---------|
| `-o` | Output file | `hashcat -o cracked.txt hashes.txt rockyou.txt` |
| `--show` | Show cracked | `hashcat -m 0 hashes.txt --show` |
| `--left` | Show uncracked | `hashcat -m 0 hashes.txt --left` |
| `--username` | Ignore usernames | `hashcat --username hashes.txt rockyou.txt` |
| `--remove` | Remove cracked | `hashcat --remove hashes.txt rockyou.txt` |

#### Session Options
| Flag | Description | Example |
|------|-------------|---------|
| `-s` | Skip | `hashcat -s 1000 hashes.txt rockyou.txt` |
| `-l` | Limit | `hashcat -l 1000 hashes.txt rockyou.txt` |
| `--session` | Session name | `hashcat --session=mycrack hashes.txt rockyou.txt` |
| `--restore` | Restore session | `hashcat --restore=mycrack` |
| `--status` | Session status | `hashcat --status=mycrack` |

#### Optimization Options
| Flag | Description | Example |
|------|-------------|---------|
| `-d` | Device | `hashcat -d 1 hashes.txt rockyou.txt` |
| `-n` | Threads | `hashcat -n 32 hashes.txt rockyou.txt` |
| `-u` | Workload | `hashcat -u 1024 hashes.txt rockyou.txt` |
| `--force` | Ignore warnings | `hashcat --force hashes.txt rockyou.txt` |

#### Rule Options
| Flag | Description | Example |
|------|-------------|---------|
| `-r` | Rule file | `hashcat -r rules/best64.rule hashes.txt rockyou.txt` |
| `-g` | Generate rules | `hashcat -g 4 hashes.txt rockyou.txt` |
| `-j` | Rule (left) | `hashcat -j "l" hashes.txt rockyou.txt` |
| `-k` | Rule (right) | `hashcat -k "l" hashes.txt rockyou.txt` |

#### Mask Options
| Flag | Description | Example |
|------|-------------|---------|
| `?l` | Lowercase | `hashcat -a 3 hashes.txt ?l?l?l?l` |
| `?u` | Uppercase | `hashcat -a 3 hashes.txt ?u?u?u?u` |
| `?d` | Digits | `hashcat -a 3 hashes.txt ?d?d?d?d` |
| `?a` | All printable | `hashcat -a 3 hashes.txt ?a?a?a?a` |
| `?b` | Binary | `hashcat -a 3 hashes.txt ?b?b?b` |
| `--custom-charset` | Custom charset | `hashcat --custom-charset=1=abc hashes.txt ?1?1?1` |

---

## Chapter 4: Intermediate Usage

### Advanced Attack Techniques

#### Combination Attack
```bash
# Combine two wordlists
hashcat -a 1 hashes.txt wordlist1.txt wordlist2.txt

# With rules
hashcat -a 1 -r rules/best64.rule hashes.txt wordlist1.txt wordlist2.txt
```

#### Hybrid Attacks
```bash
# Wordlist + mask (append numbers)
hashcat -a 6 hashes.txt rockyou.txt ?d?d?d

# Mask + wordlist (prepend numbers)
hashcat -a 7 hashes.txt ?d?d?d rockyou.txt
```

#### Mask Attacks
```bash
# 6-character lowercase
hashcat -a 3 hashes.txt ?l?l?l?l?l?l

# 8-character alphanumeric
hashcat -a 3 hashes.txt ?a?a?a?a?a?a?a?a

# Custom charset
hashcat --custom-charset=1=abc123 -a 3 hashes.txt ?1?1?1?1
```

### Scripting & Automation

#### Bash Scripting
```bash
#!/bin/bash
# Batch crack multiple hash types
declare -A HASH_TYPES
HASH_TYPES[0]="MD5"
HASH_TYPES[100]="SHA1"
HASH_TYPES[1000]="NTLM"

for hashfile in hashes_*.txt; do
    for mode in "${!HASH_TYPES[@]}"; do
        echo "[*] Cracking ${HASH_TYPES[$mode]} ($hashfile)"
        hashcat -m $mode $hashfile rockyou.txt -o cracked_${hashfile} --remove
    done
done
```

#### Python Scripting
```python
#!/usr/bin/env python3
import subprocess

def crack_hash(hashfile, mode=0, wordlist="rockyou.txt"):
    """Run hashcat to crack hashes"""
    cmd = f"hashcat -m {mode} {hashfile} {wordlist} --show"
    result = subprocess.run(cmd, shell=True, capture_output=True, text=True)
    return result.stdout

# Crack MD5 hashes
output = crack_hash("hashes.txt", mode=0)
print(output)
```

### Combining with Other Tools

#### With John the Ripper
```bash
# Convert john pot to hashcat format
john2hashcat hashes.txt > hashcat_format.txt

# Use with hashcat
hashcat -m 0 hashcat_format.txt rockyou.txt
```

#### With hcxdumptool
```bash
# Capture PMKID/handshakes
hcxdumptool -i wlan0mon --pmkid -o capture.pcapng

# Convert to hashcat format
hcxpcapngtool -o hash.hc22000 capture.pcapng

# Crack WPA/WPA2
hashcat -m 22000 hash.hc22000 rockyou.txt
```

### Advanced Configurations

#### Custom Rules
```bash
# Create custom rule
cat > my_rules.rule << 'EOF'
# Append numbers 0-9
: 0
: 1
: 2
: 3
: 4
: 5
: 6
: 7
: 8
: 9
EOF

# Use custom rules
hashcat -a 0 -r my_rules.rule hashes.txt rockyou.txt
```

#### Custom Charsets
```bash
# Define custom charset
hashcat --custom-charset=1=abc123 -a 3 hashes.txt ?1?1?1?1

# Multiple custom charsets
hashcat --custom-charset=1=abc --custom-charset=2=123 -a 3 hashes.txt ?1?2?1?2
```

### Performance Optimization

#### GPU Settings
```bash
# Check GPU support
hashcat -I

# Force specific GPU
hashcat -d 1 hashes.txt rockyou.txt

# Optimize for GPU
hashcat -O hashes.txt rockyou.txt
```

#### Workload Tuning
```bash
# Low workload (less power, slower)
hashcat --workload-profile=1 hashes.txt rockyou.txt

# High workload (more power, faster)
hashcat --workload-profile=3 hashes.txt rockyou.txt
```

---

## Chapter 5: Advanced Usage

### Advanced Attack Techniques

#### Rule-Based Attacks
```bash
# Best 64 rules
hashcat -a 0 -r rules/best64.rule hashes.txt rockyou.txt

# One rule per line
hashcat -a 0 -r rules/d3ad0ne.rule hashes.txt rockyou.txt

# Generate rules on the fly
hashcat -a 0 -g 4 hashes.txt rockyou.txt
```

### Integration in Pentest Workflow

#### Phase 1: Hash Collection
```bash
# From Windows (via SMB)
secretsdump.py user:pass@target > windows_hashes.txt

# From Linux
unshadow /etc/passwd /etc/shadow > linux_hashes.txt

# From Kerberos
GetUserSPNs.py domain/user:pass -request > kerberos_hashes.txt
```

#### Phase 2: Hash Cracking
```bash
# Crack NTLM
hashcat -m 1000 windows_hashes.txt rockyou.txt -r rules/best64.rule

# Crack MD5
hashcat -m 0 linux_hashes.txt rockyou.txt

# Crack Kerberos TGS
hashcat -m 13100 kerberos_hashes.txt rockyou.txt
```

### Advanced Configurations

#### Session Management
```bash
# Start session
hashcat --session=mycrack -m 1000 hashes.txt rockyou.txt

# Check status
hashcat --status=mycrack

# Restore session
hashcat --restore=mycrack
```

### Performance Optimization

#### Large Hash Lists
```bash
# Split hash file
split -l 100000 hashes.txt hash_chunk_

# Run in parallel
for f in hash_chunk_*; do
    hashcat -m 0 $f rockyou.txt -o cracked_$f &
done
wait

# Merge results
cat cracked_hash_chunk_* > cracked_all.txt
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: CTF Challenge
**Objective:** Crack password hashes to gain access

**Steps:**
```bash
# 1. Identify hash type
hashid '$6$salt$hash'

# 2. Crack with wordlist
hashcat -m 1800 hashes.txt rockyou.txt

# 3. Apply rules
hashcat -m 1800 hashes.txt rockyou.txt -r rules/best64.rule

# 4. Try brute-force if needed
hashcat -m 1800 -a 3 hashes.txt ?a?a?a?a?a?a

# 5. Show results
hashcat -m 1800 hashes.txt --show
```

### Scenario 2: Penetration Test
**Objective:** Audit password security

**Steps:**
```bash
# 1. Extract hashes
secretsdump.py user:pass@target > hashes.txt

# 2. Identify hash type
hashcat hashes.txt --example-hashes | head -50

# 3. Crack with common passwords
hashcat -m 1000 hashes.txt rockyou.txt

# 4. Apply rules
hashcat -m 1000 hashes.txt rockyou.txt -r rules/d3ad0ne.rule

# 5. Report weak passwords
hashcat -m 1000 hashes.txt --show | awk -F: '{print $1}'
```

### Scenario 3: WiFi Cracking
**Objective:** Test WiFi password strength

**Steps:**
```bash
# 1. Capture handshake
hcxdumptool -i wlan0mon --pmkid -o capture.pcapng

# 2. Convert to hashcat format
hcxpcapngtool -o hash.hc22000 capture.pcapng

# 3. Crack WPA2
hashcat -m 22000 hash.hc22000 rockyou.txt

# 4. Show result
hashcat -m 22000 hash.hc22000 --show
```

---

## Chapter 7: Detection & Defense

### How Defenders Detect Hashcat

#### System Indicators
```
# High GPU usage
# High CPU usage
# Large wordlist files on disk
# Hashcat pot file modifications
```

#### Log Analysis
```bash
# Check for hashcat activity
ps aux | grep hashcat
find / -name "hashcat.potfile" 2>/dev/null
ls -la ~/.local/share/hashcat/
```

### Mitigation Strategies

#### Strong Password Policies
```bash
# Enforce minimum 12+ characters
# Require complexity (upper, lower, digits, symbols)
# Ban common passwords
# Use password managers
```

#### Use Strong Hashing
```bash
# Use bcrypt (slow hashing)
# Use Argon2 (memory-hard)
# Implement proper salting
# Use high iteration counts
```

### Blue Team Response
1. **Monitor GPU usage** - Detect cracking attempts
2. **Use slow hashing** - bcrypt, Argon2
3. **Enforce password policies** - Long, complex passwords
4. **Implement account lockout** - Prevent brute force
5. **Monitor hash access** - Track extraction attempts

---

## Chapter 8: Troubleshooting

### Common Errors

#### Error: "No hashes loaded"
**Cause:** Invalid hash format or file
**Solution:**
```bash
# Check hash format
hashcat hashes.txt --example-hashes | head -50

# Specify format manually
hashcat -m 0 hashes.txt rockyou.txt
```

#### Error: "GPU not found"
**Cause:** No compatible GPU or driver issues
**Solution:**
```bash
# Check GPU support
hashcat -I

# Update drivers
sudo apt update && sudo apt upgrade

# Use CPU mode (slower)
hashcat --backend-devices=1 hashes.txt rockyou.txt
```

#### Error: "Kernel build error"
**Cause:** GPU driver issues
**Solution:**
```bash
# Rebuild kernels
hashcat --backend-devices=1 --force hashes.txt rockyou.txt

# Update drivers
sudo apt install nvidia-driver
```

### Performance Optimization

#### Slow Cracking
```bash
# Use GPU
hashcat -d 1 hashes.txt rockyou.txt

# Optimize workload
hashcat --workload-profile=3 hashes.txt rockyou.txt

# Use faster rules
hashcat -r rules/best64.rule hashes.txt rockyou.txt
```

### FAQ

**Q: Is hashcat legal to use?**
A: Yes, hashcat is a legitimate security testing tool. However, using it without authorization is illegal. Always get written permission before cracking passwords.

**Q: How do I update hashcat?**
A: `sudo apt update && sudo apt upgrade hashcat` or rebuild from source.

**Q: What's the difference between john and hashcat?**
A: john is more flexible and supports more hash types. hashcat is faster for GPU cracking. Use both for best results.

**Q: How do I crack NTLM hashes?**
A: Use `-m 1000`: `hashcat -m 1000 hashes.txt rockyou.txt`

**Q: Can hashcat crack bcrypt?**
A: Yes, but it's very slow. Use `-m 3200`: `hashcat -m 3200 hashes.txt rockyou.txt`

**Q: How do I use rules?**
A: Use `-r` flag: `hashcat -r rules/best64.rule hashes.txt rockyou.txt`

---

## Resources

### Official Documentation
- [Hashcat Homepage](https://hashcat.net/hashcat/)
- [GitHub Repository](https://github.com/hashcat/hashcat)
- [Hashcat Wiki](https://hashcat.net/wiki/)
- [Example Hashes](https://hashcat.net/wiki/doku.php?id=example_hashes)

### Tutorials
- [Hashcat Tutorial - HackTricks](https://book.hacktricks.xyz/generic-methodologies-and-resources/tips-and-tricks-john-the-ripper)
- [Hashcat Cheat Sheet](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Cracking.md)

### Related Tools
- **john:** John the Ripper (alternative)
- **hcxdumptool:** WiFi handshake capture
- **hcxpcapngtool:** Convert pcap to hashcat format
- **hashid:** Hash identifier
- **hash-identifier:** Hash identifier

### Practice Resources
- [CrackMe Archive](https://crackmes.one/)
- [HackTheBox](https://www.hackthebox.com/)
- [TryHackMe](https://tryhackme.com/)
- [VulnHub](https://www.vulnhub.com/)
