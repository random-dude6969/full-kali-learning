---
tool_name: "ophcrack"
display_name: "Ophcrack"
mitre_tactic: "credential_access"
mitre_tactic_id: "TA0006"
mitre_subcategory: "password_cracking"
install_command: "sudo apt install ophcrack"
version: "3.8.0"
github_repo: "https://github.com/ophcrack/ophcrack"
kali_tools_page: "https://www.kali.org/tools/ophcrack/"
difficulty_level: "beginner-to-intermediate"
requires_root: false
gui_available: true
tags: ["password", "cracking", "windows", "rainbow-tables", "ntlm"]
related_tools: ["john", "hashcat", "chntpw"]
---

# Ophcrack

## Chapter 1: Introduction & Overview

### What is Ophcrack?
Ophcrack is a Windows password cracker based on rainbow tables. It's designed to recover Windows passwords quickly using precomputed hash tables.

### History & Background
- **Created:** 2003 by ophcrack.sourceforge.net
- **Maintained:** Ophcrack Project
- **License:** GNU General Public License v2
- **Purpose:** Windows password cracking

Ophcrack is one of the fastest Windows password crackers.

### When to Use This Tool
- **Penetration testing:** Recover Windows passwords
- **Forensics:** Analyze SAM databases
- **Incident response:** Recover lost passwords
- **CTF challenges:** Solve password challenges

### Key Concepts
- **Rainbow Table:** Precomputed hash lookup
- **SAM Database:** Windows password hashes
- **NTLM:** Windows authentication protocol
- **LM:** Legacy Windows authentication

---

## Chapter 2: Installation & Setup

### System Requirements
- **RAM:** 4 GB minimum
- **Disk Space:** 500 MB (for rainbow tables)
- **Privileges:** User-level

### Installation Methods

#### Method 1: APT (Recommended)
```bash
sudo apt update
sudo apt install ophcrack
```

#### Method 2: From Source
```bash
git clone https://github.com/ophcrack/ophcrack.git
cd ophcrack
cmake .
make
sudo make install
```

### Rainbow Tables

#### Download Tables
```bash
# Download free tables
wget http://ophcrack.sourceforge.net/tables.php

# Or use built-in tables
ophcrack -d /usr/share/ophcrack/tables
```

### Verification
```bash
ophcrack --version
```

---

## Chapter 3: Basic Usage

### First Crack

#### Start Ophcrack
```bash
ophcrack
```
**Explanation:** Opens GUI interface.

#### Command Line Crack
```bash
ophcrack -f sam_file -t rainbow_table_dir
```
**Explanation:** Cracks hashes with rainbow tables.

### Essential Commands

#### GUI Mode
```bash
ophcrack
```
**Explanation:** Opens interactive GUI.

#### Command Line Mode
```bash
ophcrack -f sam_file -t table_dir
```
**Explanation:** Cracks hashes from command line.

#### Live CD
```bash
# Boot from Ophcrack Live CD
# Automatically cracks Windows passwords
```

### Complete Flags Reference

#### Input Options
| Flag | Description | Example |
|------|-------------|---------|
| `-f` | SAM file | `ophcrack -f sam_file` |
| `-g` | Group file | `ophcrack -g group_file` |
| `-s` | System file | `ophcrack -s system_file` |
| `-d` | Table directory | `ophcrack -d /path/to/tables` |

#### Table Options
| Flag | Description | Example |
|------|-------------|---------|
| `-t` | Table directory | `ophcrack -t tables/` |
| `-D` | Dump tables | `ophcrack -D` |
| `-e` | Table info | `ophcrack -e table_name` |

#### Output Options
| Flag | Description | Example |
|------|-------------|---------|
| `-o` | Output file | `ophcrack -o results.txt` |
| `-v` | Verbose | `ophcrack -v` |
| `-n` | No GUI | `ophcrack -n` |
| `-c` | Console | `ophcrack -c` |

#### Advanced Options
| Flag | Description | Example |
|------|-------------|---------|
| `-a` | Audit mode | `ophcrack -a sam_file` |
| `-b` | Benchmark | `ophcrack -b` |
| `-l` | List users | `ophcrack -l sam_file` |

---

## Chapter 4: Intermediate Usage

### Advanced Cracking Techniques

#### Using Rainbow Tables
```bash
# List available tables
ophcrack -l tables/

# Crack with specific table
ophcrack -f sam_file -t tables/XP_pro_fast
```

#### Audit Mode
```bash
# Audit password strength
ophcrack -a sam_file -t tables/
```

### Scripting & Automation

#### Bash Scripting
```bash
#!/bin/bash
# Automated cracking
SAM_FILE="sam_file"
TABLE_DIR="/usr/share/ophcrack/tables"

ophcrack -f $SAM_FILE -t $TABLE_DIR -o results.txt
```

#### Python Scripting
```python
#!/usr/bin/env python3
import subprocess

def ophcrack_crack(sam_file, table_dir, output=None):
    """Run ophcrack"""
    cmd = f"ophcrack -f {sam_file} -t {table_dir}"
    if output:
        cmd += f" -o {output}"
    result = subprocess.run(cmd, shell=True, capture_output=True, text=True)
    return result.stdout

# Crack SAM file
ophcrack_crack("sam_file", "/usr/share/ophcrack/tables")
```

### Combining with Other Tools

#### With Samdump2
```bash
# Extract SAM hashes
samdump2 SYSTEM SAM > hashes.txt

# Crack with ophcrack
ophcrack -f hashes.txt -t tables/
```

#### With John
```bash
# Crack with john
john --format=nt hashes.txt

# Or use ophcrack
ophcrack -f hashes.txt -t tables/
```

### Advanced Configurations

#### Custom Tables
```bash
# Generate custom tables
# Download from ophcrack.sourceforge.net

# Use custom tables
ophcrack -f sam_file -t custom_tables/
```

### Performance Optimization

#### Faster Cracking
```bash
# Use faster tables
ophcrack -f sam_file -t tables/XP_pro_fast

# Use larger tables
ophcrack -f sam_file -t tables/XP_pro_big
```

---

## Chapter 5: Advanced Usage

### Advanced Cracking Techniques

#### SAM File Extraction
```bash
# Windows
reg save HKLM\SAM sam_file
reg save HKLM\SYSTEM system_file

# Linux (from mounted Windows)
chntpw -r /mnt/windows/WINDOWS/system32/config/SAM
```

#### Live CD Method
```bash
# Download Ophcrack Live CD
# Boot from USB/CD
# Automatically cracks Windows passwords
```

### Integration in Pentest Workflow

#### Phase 1: Hash Collection
```bash
# Extract SAM on Windows
reg save HKLM\SAM sam_file
reg save HKLM\SYSTEM system_file

# Or use mimikatz
mimikatz.exe "privilege::debug" "lsadump::sam"
```

#### Phase 2: Cracking
```bash
# Crack with ophcrack
ophcrack -f sam_file -t tables/

# Or use john
john --format=nt sam_hashes.txt
```

### Advanced Configurations

#### Table Selection
```bash
# Fast tables (less coverage)
ophcrack -f sam_file -t tables/XP_pro_fast

# Large tables (more coverage)
ophcrack -f sam_file -t tables/XP_pro_big
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: CTF Challenge
**Objective:** Crack Windows password

**Steps:**
```bash
# 1. Get SAM file
# 2. Crack with ophcrack
ophcrack -f sam_file -t tables/

# 3. Use password
```

### Scenario 2: Penetration Test
**Objective:** Recover Windows passwords

**Steps:**
```bash
# 1. Extract SAM on target
reg save HKLM\SAM sam_file
reg save HKLM\SYSTEM system_file

# 2. Transfer to attacker

# 3. Crack with ophcrack
ophcrack -f sam_file -t tables/

# 4. Use credentials
```

---

## Chapter 7: Detection & Defense

### How Defenders Detect Ophcrack

#### System Indicators
```
# SAM file access
# Rainbow table usage
# Password cracking activity
```

#### Log Analysis
```bash
# Check for SAM access
# Monitor for cracking tools
```

### Mitigation Strategies

#### Strong Passwords
```bash
# Use complex passwords (12+ characters)
# Use uppercase, lowercase, numbers, symbols
# Avoid common passwords
```

#### Password Policies
```bash
# Enforce minimum length
# Require complexity
# Regular password changes
```

### Blue Team Response
1. **Use strong passwords** - Resist rainbow tables
2. **Enable NTLMv2** - Stronger than LM
3. **Disable LM hashes** - Prevent legacy attacks
4. **Monitor SAM access** - Detect extraction attempts

---

## Chapter 8: Troubleshooting

### Common Errors

#### Error: "SAM file not found"
**Cause:** Wrong path
**Solution:**
```bash
# Check path
ls -la sam_file

# Use correct path
ophcrack -f /path/to/sam_file -t tables/
```

#### Error: "Table not found"
**Cause:** Tables not installed
**Solution:**
```bash
# Download tables
wget http://ophcrack.sourceforge.net/tables.php

# Use correct path
ophcrack -f sam_file -t /path/to/tables/
```

### Performance Optimization

#### Slow Cracking
```bash
# Use faster tables
ophcrack -f sam_file -t tables/XP_pro_fast

# Use larger tables
ophcrack -f sam_file -t tables/XP_pro_big
```

### FAQ

**Q: Is ophcrack legal to use?**
A: Yes, ophcrack is a legitimate security testing tool. However, using it without authorization is illegal. Always get written permission before testing.

**Q: How do I update ophcrack?**
A: `sudo apt update && sudo apt upgrade ophcrack`

**Q: Can ophcrack crack Linux passwords?**
A: No, ophcrack is designed for Windows NTLM/LM hashes.

**Q: How do I get SAM files?**
A: Extract from Windows using `reg save` or from mounted drives.

**Q: What tables should I use?**
A: Use XP_pro_fast for speed, XP_pro_big for coverage.

---

## Resources

### Official Documentation
- [Ophcrack Homepage](https://ophcrack.sourceforge.net/)
- [GitHub Repository](https://github.com/ophcrack/ophcrack)
- [Rainbow Tables](https://ophcrack.sourceforge.net/tables.php)

### Related Tools
- **john:** John the Ripper
- **hashcat:** Advanced password recovery
- **chntpw:** Windows password change

### Practice Resources
- [HackTheBox](https://www.hackthebox.com/)
- [TryHackMe](https://tryhackme.com/)
- [VulnHub](https://www.vulnhub.com/)
