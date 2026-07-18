---
tool_name: "bopscrk"
display_name: "Bopscrk"
mitre_tactic: "credential_access"
mitre_tactic_id: "TA0006"
mitre_subcategory: "password_wordlists"
install_command: "pip3 install bopscrk"
version: "1.0"
github_repo: "https://github.com/RaikiaUuh/bopscrk"
kali_tools_page: "https://www.kali.org/tools/bopscrk/"
difficulty_level: "beginner-to-intermediate"
requires_root: false
gui_available: false
tags: ["wordlist", "generation", "personalized", "custom"]
related_tools: ["crunch", "cewl", "rsmangler"]
---

# Bopscrk

## Chapter 1: Introduction & Overview

### What is Bopscrk?
Bopscrk (Before Offensive Passwords) is a tool for generating personalized password wordlists based on information about a target. It creates targeted wordlists by combining personal details like names, birthdays, and other information.

### History & Background
- **Created:** 2019 by RaikiaUuh
- **Maintained:** RaikiaUuh
- **License:** GNU General Public License v3
- **Purpose:** Personalized wordlist generation

Bopscrk creates highly targeted wordlists for social engineering attacks.

### When to Use This Tool
- **Penetration testing:** Create target-specific wordlists
- **Social engineering:** Generate personalized passwords
- **CTF challenges:** Solve password challenges
- **Security assessments:** Test password policies

### Key Concepts
- **Personalization:** Using target information
- **Wordlist:** List of potential passwords
- **Social engineering:** Gathering target info
- **Targeted attack:** Focused password guessing

---

## Chapter 2: Installation & Setup

### System Requirements
- **Python:** 3.x
- **Disk Space:** ~5 MB
- **Privileges:** User-level

### Installation Methods

#### Method 1: Pip (Recommended)
```bash
pip3 install bopscrk
```

#### Method 2: From Source
```bash
git clone https://github.com/RaikiaUuh/bopscrk.git
cd bopscrk
pip3 install -r requirements.txt
```

### Verification
```bash
bopscrk --help
```

---

## Chapter 3: Basic Usage

### First Wordlist

#### Interactive Mode
```bash
bopscrk -i
```
**Output:** Opens interactive mode to input target information.

#### Generate Wordlist
```bash
bopscrk -n "John Doe" -b "1990" -o wordlist.txt
```
**Output:** Generates personalized wordlist.

### Essential Commands

#### Interactive Mode
```bash
bopscrk -i
```
**Explanation:** Opens interactive mode to gather target information.

#### Command Line
```bash
bopscrk -n "John Doe" -b "1990-05-15" -o wordlist.txt
```
**Explanation:** Generates wordlist from command line arguments.

### Complete Command Reference

#### Target Information
| Flag | Description | Example |
|------|-------------|---------|
| `-n` | Name | `bopscrk -n "John Doe"` |
| `-b` | Birthday | `bopscrk -b "1990-05-15"` |
| `-p` | Partner name | `bopscrk -p "Jane"` |
| `-c` | Company | `bopscrk -c "Acme"` |
| `-k` | Keywords | `bopscrk -k "admin,root"` |

#### Output Options
| Flag | Description | Example |
|------|-------------|---------|
| `-o` | Output file | `bopscrk -o wordlist.txt` |
| `-i` | Interactive mode | `bopscrk -i` |

---

## Chapter 4: Intermediate Usage

### Advanced Generation Techniques

#### Multiple Information Sources
```bash
# Combine multiple sources
bopscrk -n "John Doe" -b "1990" -p "Jane" -c "Acme" -o wordlist.txt
```

#### Custom Keywords
```bash
# Add custom keywords
bopscrk -k "admin,password,root,123456" -o wordlist.txt
```

### Scripting & Automation

#### Bash Scripting
```bash
#!/bin/bash
# Automated wordlist generation
TARGET_NAME="John Doe"
BIRTHDAY="1990"
COMPANY="Acme"

bopscrk -n "$TARGET_NAME" -b "$BIRTHDAY" -c "$COMPANY" -o wordlist.txt
```

#### Python Scripting
```python
#!/usr/bin/env python3
import subprocess

def generate_wordlist(name, birthday, company=None, output="wordlist.txt"):
    """Generate personalized wordlist"""
    cmd = f"bopscrk -n '{name}' -b '{birthday}' -o {output}"
    if company:
        cmd += f" -c '{company}'"
    result = subprocess.run(cmd, shell=True, capture_output=True, text=True)
    return result.stdout

# Generate wordlist
generate_wordlist("John Doe", "1990", "Acme")
```

### Combining with Other Tools

#### With Hydra
```bash
# Generate wordlist
bopscrk -n "John Doe" -b "1990" -o wordlist.txt

# Use with hydra
hydra -l john -P wordlist.txt ssh://192.168.1.100
```

#### With Crunch
```bash
# Generate base wordlist
bopscrk -n "John Doe" -o base.txt

# Generate variations
crunch 8 8 -t @@@@%%%% -o variations.txt

# Combine
cat base.txt variations.txt | sort -u > final.txt
```

---

## Chapter 5: Advanced Usage

### Advanced Generation Techniques

#### Social Engineering Integration
```bash
# Gather target info
# Then generate wordlist
bopscrk -i
```

### Integration in Pentest Workflow

#### Phase 1: Reconnaissance
```bash
# Gather target information
# Social media, company website, etc.
```

#### Phase 2: Wordlist Generation
```bash
# Generate targeted wordlist
bopscrk -n "Target Name" -b "1985" -c "TargetCompany" -o target_wordlist.txt
```

#### Phase 3: Password Testing
```bash
# Use wordlist
hydra -l target -P target_wordlist.txt ssh://192.168.1.100
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: CTF Challenge
**Objective:** Guess user password

**Steps:**
```bash
# 1. Gather target info from challenge
# 2. Generate wordlist
bopscrk -n "Alice" -b "1992" -o alice_wordlist.txt

# 3. Use with hydra
hydra -l alice -P alice_wordlist.txt ssh://192.168.1.100
```

### Scenario 2: Penetration Test
**Objective:** Test employee password strength

**Steps:**
```bash
# 1. Gather employee info
# 2. Generate wordlist
bopscrk -i

# 3. Test passwords
hydra -l employee -P company_wordlist.txt ssh://192.168.1.100
```

---

## Chapter 7: Detection & Defense

### How Defenders Detect Bopscrk

#### System Indicators
```
# Personalized wordlist generation
# Social engineering activity
```

### Mitigation Strategies

#### Strong Password Policies
```bash
# Enforce minimum 12+ characters
# Require complexity
# Ban personal information in passwords
```

### Blue Team Response
1. **Enforce strong passwords** - 12+ characters
2. **Ban personal info** - No names, birthdays
3. **Use password filters** - Block common patterns
4. **Educate users** - Avoid personal info in passwords

---

## Chapter 8: Troubleshooting

### Common Errors

#### Error: "Module not found"
**Cause:** Missing Python dependencies
**Solution:**
```bash
pip3 install -r requirements.txt
```

### FAQ

**Q: Is bopscrk legal to use?**
A: Yes, bopscrk is a legitimate security testing tool. However, using it without authorization is illegal.

**Q: How do I update bopscrk?**
A: `pip3 install --upgrade bopscrk`

**Q: What information can I use?**
A: Names, birthdays, partner names, company names, keywords.

---

## Resources

### Official Documentation
- [Bopscrk GitHub](https://github.com/RaikiaUuh/bopscrk)

### Related Tools
- **crunch:** Wordlist generator
- **cewl:** Custom wordlist from websites
- **rsmangler:** Wordlist mangler

### Practice Resources
- [HackTheBox](https://www.hackthebox.com/)
- [TryHackMe](https://tryhackme.com/)
