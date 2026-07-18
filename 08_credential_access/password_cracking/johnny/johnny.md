---
tool_name: "johnny"
display_name: "Johnny"
mitre_tactic: "credential_access"
mitre_tactic_id: "TA0006"
mitre_subcategory: "password_cracking"
install_command: "sudo apt install johnny"
version: "2.2"
github_repo: ""
kali_tools_page: "https://www.kali.org/tools/johnny/"
difficulty_level: "beginner"
requires_root: false
gui_available: true
tags: ["password", "cracking", "gui", "john", "hash"]
related_tools: ["john", "hashcat", "ophcrack"]
---

# Johnny

## Chapter 1: Introduction & Overview

### What is Johnny?
Johnny is a GUI (Graphical User Interface) frontend for John the Ripper. It provides a user-friendly interface for password cracking operations, making John's powerful features accessible to beginners.

### History & Background
- **Created:** 2007 by Openwall
- **Maintained:** Openwall Project
- **License:** GNU General Public License v2
- **Purpose:** GUI for John the Ripper

Johnny simplifies password cracking with a visual interface.

### When to Use This Tool
- **Beginners:** Learn password cracking
- **Password auditing:** Test password strength
- **Forensics:** Recover passwords
- **CTF challenges:** Solve password challenges

### Key Concepts
- **GUI:** Graphical User Interface
- **John the Ripper:** Backend password cracker
- **Hash:** Cryptographic one-way function
- **Wordlist:** List of potential passwords

---

## Chapter 2: Installation & Setup

### System Requirements
- **OS:** Linux with GUI
- **Disk Space:** ~10 MB
- **Privileges:** User-level

### Installation Methods

#### Method 1: APT (Recommended)
```bash
sudo apt update
sudo apt install johnny
```

### Dependencies
```bash
# John the Ripper must be installed
sudo apt install john
```

### Verification
```bash
johnny --version
```

---

## Chapter 3: Basic Usage

### First Session

#### Start Johnny
```bash
johnny
```
**Output:** Opens GUI interface.

### Essential Commands

#### GUI Mode
```bash
johnny
```
**Explanation:** Opens interactive GUI.

### Interface Components

#### Main Window
| Component | Description |
|-----------|-------------|
| Hash Input | Enter or paste hashes |
| Wordlist | Select wordlist file |
| Rules | Enable/disable rules |
| Start | Begin cracking |
| Stop | Stop cracking |
| Show | Display results |

---

## Chapter 4: Intermediate Usage

### Advanced Features

#### Hash Types
```bash
# Johnny auto-detects hash types
# Or select manually from dropdown
```

#### Session Management
```bash
# Save session
# Load session
# Resume cracking
```

### Scripting & Automation

#### Command Line
```bash
# Johnny can be scripted via command line
johnny --help
```

### Combining with Other Tools

#### With John the Ripper
```bash
# Johnny uses John as backend
# Results are compatible
john --show hashes.txt
```

---

## Chapter 5: Advanced Usage

### Advanced Features

#### Custom Rules
```bash
# Enable rules in GUI
# Select rule set from dropdown
```

#### Performance Tuning
```bash
# Adjust thread count in GUI
# Enable GPU acceleration if available
```

### Integration in Pentest Workflow

#### Phase 1: Hash Collection
```bash
# Collect hashes
# Paste into Johnny
```

#### Phase 2: Cracking
```bash
# Select wordlist
# Enable rules
# Start cracking
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: CTF Challenge
**Objective:** Crack password hash

**Steps:**
```bash
# 1. Launch Johnny
johnny

# 2. Paste hash
# 3. Select wordlist
# 4. Start cracking
# 5. View results
```

---

## Chapter 7: Detection & Defense

### How Defenders Detect Johnny

#### System Indicators
```
# GUI password cracking activity
# John the Ripper process
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
2. **Monitor cracking tools** - Detect activity
3. **Use account lockout** - Prevent brute force

---

## Chapter 8: Troubleshooting

### Common Errors

#### Error: "John not found"
**Cause:** John the Ripper not installed
**Solution:**
```bash
sudo apt install john
```

#### Error: "No hashes loaded"
**Cause:** Invalid hash format
**Solution:**
```bash
# Check hash format
# Use hashid to identify
hashid 'hash_string'
```

### FAQ

**Q: Is johnny legal to use?**
A: Yes, johnny is a legitimate security testing tool. However, using it without authorization is illegal.

**Q: How do I update johnny?**
A: `sudo apt update && sudo apt upgrade johnny`

**Q: What's the difference between johnny and john?**
A: Johnny is a GUI frontend for John. John is the backend cracker.

---

## Resources

### Official Documentation
- [Johnny Documentation](https://openwall.com/john/)
- [John the Ripper](https://www.openwall.com/john/)

### Related Tools
- **john:** John the Ripper
- **hashcat:** Advanced password recovery
- **ophcrack:** Windows password cracker

### Practice Resources
- [HackTheBox](https://www.hackthebox.com/)
- [TryHackMe](https://tryhackme.com/)
