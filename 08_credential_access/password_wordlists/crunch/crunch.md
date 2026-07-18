---
tool_name: "crunch"
display_name: "Crunch"
mitre_tactic: "credential_access"
mitre_tactic_id: "TA0006"
mitre_subcategory: "password_wordlists"
install_command: "sudo apt install crunch"
version: "3.6"
github_repo: "https://github.com/crunchsec/crunch"
kali_tools_page: "https://www.kali.org/tools/crunch/"
difficulty_level: "beginner"
requires_root: false
gui_available: false
tags: ["wordlist", "generation", "brute-force", "custom"]
related_tools: ["cewl", "wordlists", "maskprocessor"]
---

# Crunch

## Chapter 1: Introduction & Overview

### What is Crunch?
Crunch is a wordlist generator where you can specify a standard character set or a character set you specify. Crunch can generate all possible combinations and permutations.

### History & Background
- **Created:** 2009 by bofh28
- **Maintained:** bofh28
- **License:** GNU General Public License v2
- **Purpose:** Wordlist generation

Crunch generates custom wordlists for brute-force attacks.

### When to Use This Tool
- **Penetration testing:** Generate custom wordlists
- **Password auditing:** Test password policies
- **CTF challenges:** Generate brute-force lists
- **Wireless testing:** Generate WiFi passwords

### Key Concepts
- **Charset:** Characters to use
- **Min/Max length:** Password length range
- **Pattern:** Custom patterns with placeholders
- **Start/Stop:** Resume generation

### Placeholders
| Placeholder | Description | Example |
|-------------|-------------|---------|
| `@` | Lowercase letters | `@@@` = "abc" |
| `,` | Uppercase letters | `,,` = "ABC" |
| `%` | Numbers | `%%%` = "012" |
| `^` | Symbols | `^^` = "!@" |

---

## Chapter 2: Installation & Setup

### System Requirements
- **Disk Space:** ~5 MB
- **Privileges:** User-level

### Installation Methods

#### Method 1: APT (Recommended)
```bash
sudo apt update
sudo apt install crunch
```

### Verification
```bash
crunch --version
```

---

## Chapter 3: Basic Usage

### First Wordlist

#### Generate Simple Wordlist
```bash
crunch 8 8 -t @@@@%%%% -o wordlist.txt
```
**Output:** Generates 8-char passwords with 4 letters + 4 numbers.

#### Generate With Min/Max
```bash
crunch 4 6 abc -o wordlist.txt
```
**Output:** Generates 4-6 char passwords with abc.

### Essential Commands

#### Basic Generation
```bash
crunch 8 8 -t @@@@%%%% -o wordlist.txt
```
**Explanation:** Generates 8-char passwords.

#### Character Set
```bash
crunch 4 4 0123456789 -o wordlist.txt
```
**Explanation:** Generates 4-digit numbers.

#### With Pattern
```bash
crunch 8 8 -t @@@@%%%% -o wordlist.txt
```
**Explanation:** Pattern: 4 letters + 4 numbers.

### Complete Flags Reference

#### Length Options
| Flag | Description | Example |
|------|-------------|---------|
| `min-len` | Minimum length | `crunch 8 8 ...` |
| `max-len` | Maximum length | `crunch 4 8 ...` |

#### Character Options
| Flag | Description | Example |
|------|-------------|---------|
| `-t` | Pattern | `crunch 8 8 -t @@@@%%%%` |
| `-c` | Numbers per line | `crunch 8 8 -c 1000` |
| `-d` | Duplicate limit | `crunch 8 8 -d 2` |

#### Output Options
| Flag | Description | Example |
|------|-------------|---------|
| `-o` | Output file | `crunch 8 8 -o wordlist.txt` |
| `-b` | Start byte | `crunch 8 8 -b 1GB` |
| `-e` | End byte | `crunch 8 8 -e 2GB` |
| `-f` | Charset file | `crunch 8 8 -f charset.lst mixalpha-numeric` |
| `-s` | Start string | `crunch 8 8 -s aaaa0000` |

#### Misc Options
| Flag | Description | Example |
|------|-------------|---------|
| `-p` | Permutation | `crunch 4 4 -p abc def` |
| `-u` | Unique words | `crunch 8 8 -u` |
| `-z` | Gzip | `crunch 8 8 -z output.gz` |

---

## Chapter 4: Intermediate Usage

### Advanced Generation Techniques

#### Pattern Generation
```bash
# Pattern: 4 letters + 4 numbers
crunch 8 8 -t @@@@%%%% -o wordlist.txt

# Pattern: 2 uppercase + 4 numbers
crunch 6 6 -t %%%%,, -o wordlist.txt

# Pattern: word + 2 numbers
crunch 6 6 -t @@@%%% -o wordlist.txt
```

#### Character Sets
```bash
# Lowercase letters
crunch 8 8 -t @@@@@@@@ -o wordlist.txt

# Uppercase letters
crunch 8 8 -t ,,,,,,, -o wordlist.txt

# Numbers
crunch 8 8 -t %%%%%%%% -o wordlist.txt

# Symbols
crunch 8 8 -t ^^^^^^^^ -o wordlist.txt

# Mixed
crunch 8 8 -t @@@@%%%% -o wordlist.txt
```

### Scripting & Automation

#### Bash Scripting
```bash
#!/bin/bash
# Generate wordlists for different patterns
PATTERNS=(
    "@@@@%%%%"  # 4 letters + 4 numbers
    "%%%%@@@@"  # 4 numbers + 4 letters
    "@@@%%%@"   # 3 letters + 3 numbers + 1 letter
)

for pattern in "${PATTERNS[@]}"; do
    echo "[*] Generating pattern: $pattern"
    crunch 8 8 -t $pattern -o "wordlist_${pattern}.txt"
done
```

#### Python Scripting
```python
#!/usr/bin/env python3
import subprocess

def crunch_generate(min_len, max_len, pattern=None, charset=None, output=None):
    """Generate wordlist with crunch"""
    cmd = f"crunch {min_len} {max_len}"
    if pattern:
        cmd += f" -t {pattern}"
    if charset:
        cmd += f" {charset}"
    if output:
        cmd += f" -o {output}"
    result = subprocess.run(cmd, shell=True, capture_output=True, text=True)
    return result.stdout

# Generate wordlist
crunch_generate(8, 8, pattern="@@%%@@%%", output="wordlist.txt")
```

### Combining with Other Tools

#### With Hydra
```bash
# Generate wordlist
crunch 8 8 -t @@@@%%%% -o wordlist.txt

# Use with hydra
hydra -l admin -P wordlist.txt ssh://target
```

#### With Hashcat
```bash
# Generate wordlist
crunch 8 8 -t @@@@%%%% -o wordlist.txt

# Use with hashcat
hashcat -m 0 hash.txt wordlist.txt
```

### Advanced Configurations

#### Resuming Generation
```bash
# Start from specific string
crunch 8 8 -t @@@@%%%% -s aaaa0000 -o wordlist.txt
```

#### Limiting Output
```bash
# Limit to 1000 words per file
crunch 8 8 -t @@@@%%%% -c 1000 -o wordlist.txt

# Limit by size
crunch 8 8 -t @@@@%%%% -b 1GB -o wordlist.txt
```

### Performance Optimization

#### Reduce File Size
```bash
# Compress output
crunch 8 8 -t @@@@%%%% -z wordlist.txt.gz

# Limit words
crunch 8 8 -t @@@@%%%% -c 100000 -o wordlist.txt
```

---

## Chapter 5: Advanced Usage

### Advanced Generation Techniques

#### Custom Character Sets
```bash
# Use charset file
crunch 8 8 -f charset.lst mixalpha-numeric -o wordlist.txt

# Create charset file
cat > charset.lst << EOF
mixalpha-numeric = abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789
EOF
```

#### Permutation Generation
```bash
# Generate permutations
crunch 3 3 -p abc def ghi -o permutations.txt
```

### Integration in Pentest Workflow

#### Phase 1: Wordlist Generation
```bash
# Generate target-specific wordlist
crunch 8 8 -t @@@@%%%% -o target_wordlist.txt

# Generate wireless wordlist
crunch 8 8 -t %%%%@@@@ -o wifi_wordlist.txt
```

#### Phase 2: Password Testing
```bash
# Use with hydra
hydra -l admin -P target_wordlist.txt ssh://target

# Use with hashcat
hashcat -m 0 hash.txt target_wordlist.txt
```

### Advanced Configurations

#### Multiple Patterns
```bash
# Generate multiple patterns
crunch 8 8 -t @@@@%%%% -o wordlist1.txt
crunch 8 8 -t %%%%@@@@ -o wordlist2.txt
crunch 8 8 -t @@%%@@%% -o wordlist3.txt
```

### Performance Optimization

#### Large Generation
```bash
# Use pipes
crunch 8 8 -t @@@@%%%% | head -10000 > wordlist.txt

# Compress
crunch 8 8 -t @@@@%%%% -z wordlist.txt.gz
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: CTF Challenge
**Objective:** Brute-force password

**Steps:**
```bash
# 1. Generate wordlist
crunch 8 8 -t @@@@%%%% -o wordlist.txt

# 2. Use with hydra
hydra -l admin -P wordlist.txt ssh://target

# 3. Login with found password
ssh admin@target
```

### Scenario 2: WiFi Testing
**Objective:** Generate WiFi passwords

**Steps:**
```bash
# 1. Generate 8-digit PINs
crunch 8 8 0123456789 -o pins.txt

# 2. Use with reaver
reaver -i wlan0mon -b BSSID -p pins.txt
```

### Scenario 3: Penetration Test
**Objective:** Test password policies

**Steps:**
```bash
# 1. Generate common passwords
crunch 6 6 -t %%%%@@ -o common.txt
crunch 6 6 -t @@@%%% -o common2.txt

# 2. Combine wordlists
cat common.txt common2.txt | sort -u > final.txt

# 3. Test passwords
hydra -l admin -P final.txt ssh://target
```

---

## Chapter 7: Detection & Defense

### How Defenders Detect Crunch

#### System Indicators
```
# Wordlist generation activity
# Large file creation
```

#### Log Analysis
```bash
# Monitor file creation
inotifywait -r /path/to/

# Check for large files
find / -size +100M -name "*.txt"
```

### Mitigation Strategies

#### Strong Passwords
```bash
# Enforce minimum 12+ characters
# Require complexity
# Ban common passwords
```

### Blue Team Response
1. **Enforce strong passwords** - 12+ characters
2. **Require complexity** - Mix character types
3. **Ban common passwords** - Use password filters
4. **Monitor generation** - Detect wordlist creation

---

## Chapter 8: Troubleshooting

### Common Errors

#### Error: "Invalid pattern"
**Cause:** Wrong placeholder syntax
**Solution:**
```bash
# Use correct placeholders
# @ = lowercase
# , = uppercase
# % = numbers
# ^ = symbols

crunch 8 8 -t @@@@%%%% -o wordlist.txt
```

### Performance Optimization

#### Slow Generation
```bash
# Reduce length
crunch 4 4 -t @@@@ -o wordlist.txt

# Limit output
crunch 8 8 -t @@@@%%%% -c 100000 -o wordlist.txt
```

### FAQ

**Q: Is crunch legal to use?**
A: Yes, crunch is a legitimate security testing tool. However, using generated wordlists for unauthorized access is illegal.

**Q: How do I update crunch?**
A: `sudo apt update && sudo apt upgrade crunch`

**Q: What placeholders are available?**
A: `@` = lowercase, `,` = uppercase, `%` = numbers, `^` = symbols

**Q: How do I resume generation?**
A: Use `-s` flag: `crunch 8 8 -t @@@@%%%% -s aaaa0000`

---

## Resources

### Official Documentation
- [Crunch GitHub](https://github.com/crunchsec/crunch)
- [Crunch Man Page](https://linux.die.net/man/1/crunch)

### Related Tools
- **cewl:** Custom wordlist from websites
- **wordlists:** Common wordlists
- **maskprocessor:** Mask-based generation

### Practice Resources
- [HackTheBox](https://www.hackthebox.com/)
- [TryHackMe](https://tryhackme.com/)
