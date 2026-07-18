---
tool_name: "john"
display_name: "John the Ripper"
mitre_tactic: "credential_access"
mitre_tactic_id: "TA0006"
mitre_subcategory: "password_cracking"
install_command: "sudo apt install john"
version: "1.9.0"
github_repo: "https://github.com/magnumripper/JohnTheRipper"
kali_tools_page: "https://www.kali.org/tools/john/"
difficulty_level: "beginner-to-advanced"
requires_root: false
gui_available: false
tags: ["password", "cracking", "hash", "offline", "john"]
related_tools: ["hashcat", "ophcrack", "johnny"]
---

# John the Ripper

## Chapter 1: Introduction & Overview

### What is John the Ripper?
John the Ripper (john) is a free and open-source password security auditing and password recovery tool. It's designed to detect weak Unix passwords and supports hundreds of hash types including MD5, SHA-1, SHA-2, bcrypt, and many more.

### History & Background
- **Created:** 1996 by Solar Designer
- **Maintained:** Openwall Project
- **License:** GNU General Public License v2
- **Purpose:** Password cracking and security auditing

John the Ripper is one of the most popular password cracking tools due to its versatility, speed, and support for numerous hash formats.

### When to Use This Tool
- **Password auditing:** Test password strength
- **Penetration testing:** Crack captured hashes
- **Forensics:** Recover passwords from systems
- **CTF challenges:** Solve password cracking challenges
- **Security assessments:** Evaluate password policies

### When NOT to Use This Tool
- Without explicit written authorization
- Against systems you don't own
- For malicious purposes

### Legal and Ethical Considerations
- **Always get written permission** before cracking passwords
- **Only crack hashes you own** or have authorization to test
- **Document everything** - keep records of your activities
- **Follow responsible disclosure** - report weak passwords properly

### Key Concepts
- **Hash:** A one-way cryptographic function
- **Salt:** Random data added to password before hashing
- **Rainbow table:** Precomputed hash lookup table
- **Brute-force:** Try all possible combinations
- **Dictionary attack:** Try words from a wordlist
- **Rule-based attack:** Apply mutations to wordlist words

### Hash Types Supported
| Type | Format | Example |
|------|--------|---------|
| MD5 | $1$ | $1$salt$hash |
| SHA-256 | $5$ | $5$salt$hash |
| SHA-512 | $6$ | $6$salt$hash |
| bcrypt | $2a$/$2b$ | $2a$10$hash |
| NTLM | N/A | aad3b435b51404eeaad3b435b51404ee |
| Kerberos | $krb5$ | $krb5$19$... |
| NT | N/A | aad3b435b51404eeaad3b435b51404ee |

---

## Chapter 2: Installation & Setup

### System Requirements
- **Disk Space:** ~50 MB
- **Privileges:** User-level (no root required)

### Installation Methods

#### Method 1: APT (Recommended)
```bash
sudo apt update
sudo apt install john
```

#### Method 2: From Source
```bash
git clone https://github.com/magnumripper/JohnTheRipper.git
cd JohnTheRipper/src
./configure
make
```

### Verification
```bash
john --version
# Output: John the Ripper 1.9.0-jumbo-1
```

---

## Chapter 3: Basic Usage

### First Crack
```bash
john --wordlist=/usr/share/wordlists/rockyou.txt hashes.txt
```
**Output:** Attempts to crack hashes using rockyou.txt wordlist.

### Essential Commands

#### Wordlist Attack
```bash
john --wordlist=rockyou.txt hashes.txt
```
**Explanation:** Tries all words in rockyou.txt against the hashes.

#### Default Attack (Auto-Detect)
```bash
john hashes.txt
```
**Explanation:** Uses built-in wordlist and rules automatically.

#### Show Cracked Passwords
```bash
john --show hashes.txt
```
**Explanation:** Displays all cracked passwords.

#### List Format
```bash
john --list=formats
```
**Explanation:** Shows all supported hash formats.

### Complete Flags Reference

#### Input Options
| Flag | Description | Example |
|------|-------------|---------|
| `--wordlist` | Wordlist file | `john --wordlist=rockyou.txt hashes.txt` |
| `--stdin` | Read from stdin | `echo pass123 \| john --stdin hashes.txt` |
| `--pipe` | Read from pipe | `cat passwords \| john --pipe hashes.txt` |
| `--format` | Hash format | `john --format=raw-md5 hashes.txt` |

#### Attack Options
| Flag | Description | Example |
|------|-------------|---------|
| `--single` | Single crack mode | `john --single hashes.txt` |
| `--wordlist` | Wordlist mode | `john --wordlist=rockyou.txt hashes.txt` |
| `--incremental` | Brute-force | `john --incremental hashes.txt` |
| `--external` | External mode | `john --external=filter hashes.txt` |
| `--fork` | Fork processes | `john --fork=4 hashes.txt` |

#### Rules Options
| Flag | Description | Example |
|------|-------------|---------|
| `--rules` | Enable rules | `john --rules hashes.txt` |
| `--rules=single` | Single rules | `john --rules=single hashes.txt` |
| `--rules=jumbo` | Jumbo rules | `john --rules=jumbo hashes.txt` |
| `--rule` | Specific rule | `john --rule=l3 hashes.txt` |
| `--list-rules` | List all rules | `john --list-rules` |

#### Session Options
| Flag | Description | Example |
|------|-------------|---------|
| `--session` | Session name | `john --session=mycrack hashes.txt` |
| `--restore` | Restore session | `john --restore=mycrack` |
| `--status` | Session status | `john --status=mycrack` |
| `--pot` | Pot file | `john --pot=john.pot hashes.txt` |

#### Output Options
| Flag | Description | Example |
|------|-------------|---------|
| `--show` | Show cracked | `john --show hashes.txt` |
| `--list` | List info | `john --list=formats` |
| `--format` | Output format | `john --show --format=raw-md5 hashes.txt` |

#### Cracking Options
| Flag | Description | Example |
|------|-------------|---------|
| `--min-length` | Min password len | `john --min-length=8 hashes.txt` |
| `--max-length` | Max password len | `john --max-length=12 hashes.txt` |
| `--min` | Min password | `john --min=80000000 hashes.txt` |
| `--max` | Max password | `john --max=99999999 hashes.txt` |
| `--mask` | Mask attack | `john --mask=?a?a?a?a?a?a hashes.txt` |
| `--charset` | Charset | `john --charset=abc123 hashes.txt` |

#### Incremental Options
| Flag | Description | Example |
|------|-------------|---------|
| `--incremental` | Default incremental | `john --incremental hashes.txt` |
| `--incremental=digits` | Digits only | `john --incremental=digits hashes.txt` |
| `--incremental=lowercase` | Lowercase | `john --incremental=lowercase hashes.txt` |

---

## Chapter 4: Intermediate Usage

### Advanced Cracking Techniques

#### Multiple Attack Modes
```bash
# Single crack mode (uses username as password)
john --single hashes.txt

# Wordlist with rules
john --wordlist=rockyou.txt --rules hashes.txt

# Brute-force (incremental)
john --incremental hashes.txt

# Mask attack
john --mask="?a?a?a?a?a?a" hashes.txt
```

### Scripting & Automation

#### Bash Scripting
```bash
#!/bin/bash
# Batch crack multiple hash files
HASHFILES_DIR="hashfiles"
WORDLIST="/usr/share/wordlists/rockyou.txt"

for hashfile in $HASHFILES_DIR/*.txt; do
    echo "[*] Cracking $hashfile"
    john --wordlist=$WORDLIST --rules --fork=4 $hashfile
done

# Show all results
for hashfile in $HASHFILES_DIR/*.txt; do
    echo "=== $hashfile ==="
    john --show $hashfile
done
```

#### Python Scripting
```python
#!/usr/bin/env python3
import subprocess
import os

def crack_hashes(hashfile, wordlist="rockyou.txt"):
    """Run john to crack hashes"""
    cmd = f"john --wordlist={wordlist} --rules {hashfile}"
    result = subprocess.run(cmd, shell=True, capture_output=True, text=True)
    return result.stdout

def show_results(hashfile):
    """Show cracked passwords"""
    cmd = f"john --show {hashfile}"
    result = subprocess.run(cmd, shell=True, capture_output=True, text=True)
    return result.stdout

# Usage
output = crack_hashes("hashes.txt")
print(output)
results = show_results("hashes.txt")
print(results)
```

### Hash Extraction

#### From /etc/shadow
```bash
# Extract hashes from shadow file
unshadow /etc/passwd /etc/shadow > hashes.txt

# Crack extracted hashes
john --wordlist=rockyou.txt hashes.txt
```

#### From SAM Database
```bash
# Extract NTLM hashes
secretsdump.py -sam SAM -system SYSTEM LOCAL

# Crack with john
john --format=nt hashes.txt
```

#### From Kerberos Tickets
```bash
# Extract Kerberos hashes
ticketExtractor.py ticket.kirbi

# Crack with john
john --format=krb5 hashes.txt
```

### Combining with Other Tools

#### With Hashcat
```bash
# Convert john pot to hashcat format
john2hashcat hashes.txt > hashcat_hashes.txt

# Use with hashcat
hashcat -m 0 hashcat_hashes.txt rockyou.txt
```

#### With Mimikatz
```bash
# Extract NTLM hashes with mimikatz
mimikatz.exe "privilege::debug" "sekurlsa::logonpasswords" > mimikatz_output.txt

# Parse and crack
john --format=nt mimikatz_hashes.txt
```

### Advanced Configurations

#### Custom Rules
```bash
# Create custom rule file
cat > my_rules.rule << EOF
# Rule: append numbers
[Regex: ^.*$]
, 0
, 1
, 2
, 3
, 4
, 5
, 6
, 7
, 8
, 9
EOF

# Use custom rules
john --wordlist=rockyou.txt --rules=my_rules hashes.txt
```

#### Custom External Filters
```bash
# Create custom filter
cat > my_filter.c << 'EOF'
#include <ctype.h>

int filter(int index)
{
    // Only try passwords with at least one digit
    int i, has_digit = 0;
    for (i = 0; word[i]; i++)
        if (isdigit(word[i])) { has_digit = 1; break; }
    return has_digit;
}
EOF

# Compile and use
john --external=my_filter hashes.txt
```

### Performance Optimization

#### GPU Acceleration (with john-jumbo)
```bash
# Check GPU support
john --list=formats | grep -i "gpu"

# Use GPU for specific format
john --format=raw-md5-opencl hashes.txt
```

#### Parallel Processing
```bash
# Use multiple cores
john --fork=8 --wordlist=rockyou.txt hashes.txt
```

---

## Chapter 5: Advanced Usage

### Advanced Cracking Techniques

#### Mask Attacks
```bash
# 6-character lowercase
john --mask="[a-z][a-z][a-z][a-z][a-z][a-z]" hashes.txt

# 8-character alphanumeric
john --mask="?a?a?a?a?a?a?a?a" hashes.txt

# Phone number pattern
john --mask="???-???-????" hashes.txt

# Custom charset
john --mask="[abc123][abc123][abc123][abc123]" hashes.txt
```

#### Hybrid Attacks
```bash
# Wordlist + mask
john --wordlist=rockyou.txt --mask="?d?d?d" hashes.txt

# Mask + wordlist
john --mask="?d?d?d" --wordlist=base_words.txt hashes.txt
```

### Integration in Pentest Workflow

#### Phase 1: Hash Collection
```bash
# From Linux systems
unshadow /etc/passwd /etc/shadow > linux_hashes.txt

# From Windows (via SMB)
secretsdump.py user:pass@target > windows_hashes.txt

# From Kerberos
GetUserSPNs.py domain/user:pass -dc-ip DC_IP -request > kerberos_hashes.txt
```

#### Phase 2: Hash Cracking
```bash
# Crack Linux hashes
john --wordlist=rockyou.txt --rules linux_hashes.txt

# Crack NTLM hashes
john --format=nt --wordlist=rockyou.txt windows_hashes.txt

# Crack Kerberos tickets
john --format=krb5-tgs --wordlist=rockyou.txt kerberos_hashes.txt
```

#### Phase 3: Credential Reuse
```bash
# Test cracked credentials
crackmapexec smb target -u admin -p password123
```

### Advanced Configurations

#### Session Management
```bash
# Start named session
john --session=crack1 --wordlist=rockyou.txt hashes.txt

# Check status
john --status=crack1

# Restore session
john --restore=crack1
```

#### Pot File Management
```bash
# View pot file
cat ~/.john/john.pot

# Reset pot file
rm ~/.john/john.pot

# Use custom pot file
john --pot=/tmp/custom.pot hashes.txt
```

### Performance Optimization

#### Rule Optimization
```bash
# Fast rules
john --rules=best64 --wordlist=rockyou.txt hashes.txt

# All rules (slow)
john --rules=jumbo --wordlist=rockyou.txt hashes.txt
```

#### Wordlist Optimization
```bash
# Remove duplicates
sort -u rockyou.txt > unique_rockyou.txt

# Use faster storage
john --wordlist=/dev/shm/rockyou.txt hashes.txt
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: CTF Challenge
**Objective:** Crack password hash to gain access

**Steps:**
```bash
# 1. Identify hash type
hashid '$1$salt$hash'

# 2. Try single crack mode
john --single hashes.txt

# 3. Try wordlist
john --wordlist=rockyou.txt hashes.txt

# 4. Try rules
john --wordlist=rockyou.txt --rules hashes.txt

# 5. Try brute-force
john --incremental hashes.txt

# 6. Show results
john --show hashes.txt
```

### Scenario 2: Penetration Test
**Objective:** Audit password security

**Steps:**
```bash
# 1. Extract hashes from system
unshadow /etc/passwd /etc/shadow > hashes.txt

# 2. Crack with common passwords
john --wordlist=rockyou.txt hashes.txt

# 3. Apply rules
john --wordlist=rockyou.txt --rules hashes.txt

# 4. Try brute-force for remaining
john --incremental hashes.txt

# 5. Report weak passwords
john --show hashes.txt | awk -F: '{print $1}'
```

### Scenario 3: Forensics Investigation
**Objective:** Recover passwords from captured evidence

**Steps:**
```bash
# 1. Extract hashes from memory dump
volatility -f memory.dump --profile=Win7SP1x64 hashdump > hashes.txt

# 2. Crack NTLM hashes
john --format=nt --wordlist=rockyou.txt hashes.txt

# 3. Document findings
john --show hashes.txt > cracked_passwords.txt
```

---

## Chapter 7: Detection & Defense

### How Defenders Detect John

#### System Indicators
```
# High CPU usage during cracking
# Large wordlist files on disk
# John pot file modifications
# Unusual process activity
```

#### Log Analysis
```bash
# Check for suspicious activity
ps aux | grep john
ls -la ~/.john/
find / -name "john.pot" 2>/dev/null
```

### Mitigation Strategies

#### Strong Password Policies
```bash
# Enforce minimum length (12+ characters)
# Require complexity (upper, lower, digits, symbols)
# Ban common passwords
# Use password managers
```

#### Use Strong Hashing
```bash
# Use bcrypt instead of MD5
# Use SHA-512 with high work factor
# Implement salting
```

#### Monitor for Cracking
```bash
# Monitor CPU usage
top -c | grep john

# Monitor file access
auditctl -w ~/.john/ -p rwa -k john_monitor
```

### Blue Team Response
1. **Monitor for high CPU** - Detect cracking attempts
2. **Use strong hashing** - bcrypt, Argon2
3. **Enforce password policies** - Long, complex passwords
4. **Implement account lockout** - Prevent brute force
5. **Enable auditing** - Track hash access
6. **Rotate hashes** - Regular password changes

---

## Chapter 8: Troubleshooting

### Common Errors

#### Error: "No password hashes loaded"
**Cause:** Invalid hash format or file
**Solution:**
```bash
# Check hash format
john --list=formats | grep md5

# Specify format manually
john --format=raw-md5 hashes.txt

# Verify hash file
head -1 hashes.txt
```

#### Error: "Unknown ciphertext format"
**Cause:** Hash format not recognized
**Solution:**
```bash
# List all formats
john --list=formats

# Try automatic detection
john hashes.txt

# Use hashid to identify
hashid '$1$salt$hash'
```

#### Error: "Wordlist not found"
**Cause:** Wordlist file missing
**Solution:**
```bash
# Install wordlists
sudo apt install wordlists

# Use alternative wordlist
john --wordlist=/usr/share/wordlists/rockyou.txt hashes.txt
```

### Performance Optimization

#### Slow Cracking
```bash
# Use GPU (if supported)
john --format=raw-md5-opencl hashes.txt

# Use more cores
john --fork=8 hashes.txt

# Use faster rules
john --rules=best64 hashes.txt
```

### FAQ

**Q: Is john legal to use?**
A: Yes, john is a legitimate security testing tool. However, using it without authorization is illegal. Always get written permission before cracking passwords.

**Q: How do I update john?**
A: `sudo apt update && sudo apt upgrade john` or rebuild from source.

**Q: What's the difference between john and hashcat?**
A: john is more flexible and supports more hash types out of the box. hashcat is faster for GPU cracking. Use both for best results.

**Q: How do I crack NTLM hashes?**
A: Use `--format=nt` flag: `john --format=nt hashes.txt`

**Q: Can john crack bcrypt?**
A: Yes, but it's very slow. Use `--format=bcrypt` flag.

**Q: How do I use rules?**
A: Use `--rules` flag: `john --wordlist=rockyou.txt --rules hashes.txt`

---

## Resources

### Official Documentation
- [John the Ripper Homepage](https://www.openwall.com/john/)
- [GitHub Repository](https://github.com/magnumripper/JohnTheRipper)
- [John Documentation](https://www.openwall.com/john/doc/)
- [John Wiki](https://github.com/magnumripper/JohnTheRipper/wiki)

### Tutorials
- [John the Ripper Tutorial - HackTricks](https://book.hacktricks.xyz/generic-methodologies-and-resources/tips-and-tricks-john-the-ripper)
- [John Cheat Sheet](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Cracking.md)

### Related Tools
- **hashcat:** Advanced password recovery (GPU)
- **ophcrack:** Windows password cracker (rainbow tables)
- **johnny:** GUI for John the Ripper
- **truecrack:** TrueCrypt password cracker
- **bruteforce-luks:** LUKS partition cracker

### Practice Resources
- [CrackMe Archive](https://crackmes.one/)
- [HackTheBox](https://www.hackthebox.com/)
- [TryHackMe](https://tryhackme.com/)
- [VulnHub](https://www.vulnhub.com/)
