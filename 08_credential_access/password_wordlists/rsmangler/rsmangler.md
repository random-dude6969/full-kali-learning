---
tool_name: "rsmangler"
display_name: "Rsmangler"
mitre_tactic: "credential_access"
mitre_tactic_id: "TA0006"
mitre_subcategory: "password_wordlists"
install_command: "sudo apt install rsmangler"
version: "1.0"
github_repo: "https://github.com/evilsocket/rsmangler"
kali_tools_page: "https://www.kali.org/tools/rsmangler/"
difficulty_level: "beginner"
requires_root: false
gui_available: false
tags: ["wordlist", "generation", "mangling", "custom"]
related_tools: ["crunch", "cewl", "wordlists"]
---

# Rsmangler

## Chapter 1: Introduction & Overview

### What is Rsmangler?
Rsmangler is a tool that takes a wordlist and mangles it to produce a new wordlist. It applies various transformations to generate password variations.

### History & Background
- **Created:** 2011 by evilsocket
- **Maintained:** evilsocket
- **License:** GNU General Public License v2
- **Purpose:** Wordlist mangling

Rsmangler creates password variations from existing wordlists.

### When to Use This Tool
- **Penetration testing:** Generate password variations
- **Password auditing:** Test password policies
- **CTF challenges:** Generate brute-force lists
- **Security assessments:** Expand wordlists

### Key Concepts
- **Mangling:** Applying transformations
- **Wordlist:** Input word list
- **Transformation:** How to modify words
- **Output:** New wordlist

### Available Transformations
| Transformation | Description |
|----------------|-------------|
| `--first-upper` | First letter uppercase |
| `--first-lower` | First letter lowercase |
| `--all-upper` | All uppercase |
| `--all-lower` | All lowercase |
| `--capitalize` | Capitalize first letter |
| `--duplicate` | Duplicate words |
| `--toggle-case` | Toggle case |
| `--add-suffix` | Add suffix |
| `--add-prefix` | Add prefix |
| `--replace` | Replace characters |

---

## Chapter 2: Installation & Setup

### System Requirements
- **Disk Space:** ~1 MB
- **Privileges:** User-level

### Installation Methods

#### Method 1: APT (Recommended)
```bash
sudo apt update
sudo apt install rsmangler
```

#### Method 2: From Source
```bash
git clone https://github.com/evilsocket/rsmangler.git
cd rsmangler
```

### Verification
```bash
rsmangler --help
```

---

## Chapter 3: Basic Usage

### First Mangling

#### Basic Mangling
```bash
rsmangler -f wordlist.txt -o mangled.txt
```
**Output:** Creates mangled wordlist.

#### With Transformations
```bash
rsmangler -f wordlist.txt --first-upper --all-lower -o mangled.txt
```
**Output:** Applies transformations.

### Essential Commands

#### Basic Usage
```bash
rsmangler -f wordlist.txt -o mangled.txt
```
**Explanation:** Mangles wordlist with default options.

#### With Transformations
```bash
rsmangler -f wordlist.txt --first-upper --capitalize -o mangled.txt
```
**Explanation:** Applies specific transformations.

### Complete Flags Reference

#### Input Options
| Flag | Description | Example |
|------|-------------|---------|
| `-f` | Input file | `rsmangler -f wordlist.txt` |
| `--file` | Input file | `rsmangler --file wordlist.txt` |

#### Transformation Options
| Flag | Description | Example |
|------|-------------|---------|
| `--first-upper` | First letter uppercase | `rsmangler --first-upper` |
| `--first-lower` | First letter lowercase | `rsmangler --first-lower` |
| `--all-upper` | All uppercase | `rsmangler --all-upper` |
| `--all-lower` | All lowercase | `rsmangler --all-lower` |
| `--capitalize` | Capitalize first letter | `rsmangler --capitalize` |
| `--duplicate` | Duplicate words | `rsmangler --duplicate` |
| `--toggle-case` | Toggle case | `rsmangler --toggle-case` |

#### Suffix/Prefix Options
| Flag | Description | Example |
|------|-------------|---------|
| `--add-suffix` | Add suffix | `rsmangler --add-suffix "123"` |
| `--add-prefix` | Add prefix | `rsmangler --add-prefix "admin"` |
| `--suffix-file` | Suffix file | `rsmangler --suffix-file suffixes.txt` |
| `--prefix-file` | Prefix file | `rsmangler --prefix-file prefixes.txt` |

#### Replacement Options
| Flag | Description | Example |
|------|-------------|---------|
| `--replace` | Replace chars | `rsmangler --replace "a:4,e:3"` |
| `--replace-file` | Replace file | `rsmangler --replace-file replacements.txt` |

#### Output Options
| Flag | Description | Example |
|------|-------------|---------|
| `-o` | Output file | `rsmangler -o mangled.txt` |
| `--output` | Output file | `rsmangler --output mangled.txt` |

---

## Chapter 4: Intermediate Usage

### Advanced Mangling Techniques

#### Multiple Transformations
```bash
# Apply multiple transformations
rsmangler -f wordlist.txt --first-upper --capitalize --add-suffix "123" -o mangled.txt
```

#### Custom Replacements
```bash
# Replace characters
rsmangler -f wordlist.txt --replace "a:4,e:3,i:1,o:0" -o mangled.txt
```

### Scripting & Automation

#### Bash Scripting
```bash
#!/bin/bash
# Generate multiple variations
WORDLIST="wordlist.txt"

echo "[*] Basic mangling..."
rsmangler -f $WORDLIST -o mangled_basic.txt

echo "[*] With suffix..."
rsmangler -f $WORDLIST --add-suffix "123" -o mangled_suffix.txt

echo "[*] With replacements..."
rsmangler -f $WORDLIST --replace "a:4,e:3" -o mangled_replace.txt

# Combine all
cat mangled_basic.txt mangled_suffix.txt mangled_replace.txt | sort -u > final.txt
```

### Combining with Other Tools

#### With Crunch
```bash
# Generate base wordlist
crunch 6 6 -t @@@%%% -o base.txt

# Mangle it
rsmangler -f base.txt --first-upper --capitalize -o mangled.txt
```

#### With Hydra
```bash
# Generate wordlist
rsmangler -f wordlist.txt --first-upper -o mangled.txt

# Use with hydra
hydra -l admin -P mangled.txt ssh://target
```

### Advanced Configurations

#### Suffix/Prefix Files
```bash
# Create suffix file
cat > suffixes.txt << EOF
123
!
?
@#
$

# Use suffix file
rsmangler -f wordlist.txt --suffix-file suffixes.txt -o mangled.txt
```

### Performance Optimization

#### Reduce Output
```bash
# Limit transformations
rsmangler -f wordlist.txt --first-upper -o small.txt

# Sort and unique
rsmangler -f wordlist.txt -o mangled.txt
sort mangled.txt | uniq > unique.txt
```

---

## Chapter 5: Advanced Usage

### Advanced Mangling Techniques

#### Complex Replacements
```bash
# Multiple replacements
rsmangler -f wordlist.txt --replace "a:4,e:3,i:1,o:0,s:5" -o mangled.txt
```

### Integration in Pentest Workflow

#### Phase 1: Wordlist Generation
```bash
# Generate base wordlist
crunch 6 6 -t @@@%%% -o base.txt

# Mangle it
rsmangler -f base.txt --first-upper --capitalize -o final.txt
```

#### Phase 2: Password Testing
```bash
# Use with hydra
hydra -l admin -P final.txt ssh://target
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: CTF Challenge
**Objective:** Generate password variations

**Steps:**
```bash
# 1. Start with common words
echo -e "admin\npassword\ntoor\nroot" > base.txt

# 2. Mangle it
rsmangler -f base.txt --first-upper --capitalize --add-suffix "123" -o variations.txt

# 3. Use with hydra
hydra -l admin -P variations.txt ssh://target
```

### Scenario 2: Penetration Test
**Objective:** Test password policies

**Steps:**
```bash
# 1. Generate base wordlist
echo -e "company\nadmin\npassword\nwelcome" > base.txt

# 2. Mangle with variations
rsmangler -f base.txt --first-upper --capitalize --add-suffix "2024" -o company_wordlist.txt

# 3. Test passwords
hydra -l admin -P company_wordlist.txt ssh://target
```

---

## Chapter 7: Detection & Defense

### How Defenders Detect Rsmangler

#### System Indicators
```
# Wordlist generation activity
# Multiple password attempts
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

---

## Chapter 8: Troubleshooting

### Common Errors

#### Error: "No input file"
**Cause:** Wrong path
**Solution:**
```bash
ls -la wordlist.txt
rsmangler -f wordlist.txt -o mangled.txt
```

### Performance Optimization

#### Large Wordlists
```bash
# Use smaller input
head -1000 wordlist.txt > small.txt
rsmangler -f small.txt -o mangled.txt
```

### FAQ

**Q: Is rsmangler legal to use?**
A: Yes, rsmangler is a legitimate security testing tool. However, using generated wordlists for unauthorized access is illegal.

**Q: How do I update rsmangler?**
A: `sudo apt update && sudo apt upgrade rsmangler`

**Q: What transformations are available?**
A: first-upper, first-lower, all-upper, all-lower, capitalize, duplicate, toggle-case, add-suffix, add-prefix, replace.

---

## Resources

### Official Documentation
- [Rsmangler GitHub](https://github.com/evilsocket/rsmangler)
- [Rsmangler Wiki](https://github.com/evilsocket/rsmangler/wiki)

### Related Tools
- **crunch:** Wordlist generator
- **cewl:** Custom wordlist from websites
- **wordlists:** Common wordlists

### Practice Resources
- [HackTheBox](https://www.hackthebox.com/)
- [TryHackMe](https://tryhackme.com/)
