---
tool_name: "sucrack"
display_name: "Sucrack"
mitre_tactic: "credential_access"
mitre_tactic_id: "TA0006"
mitre_subcategory: "password_cracking"
install_command: "sudo apt install sucrack"
version: "1.0"
github_repo: ""
kali_tools_page: "https://www.kali.org/tools/sucrack/"
difficulty_level: "beginner"
requires_root: true
gui_available: false
tags: ["password", "cracking", "su", "brute-force", "local"]
related_tools: ["john", "hashcat", "hydra"]
---

# Sucrack

## Chapter 1: Introduction & Overview

### What is Sucrack?
Sucrack is a tool designed to brute-force the `su` command on Unix-like systems. It attempts to guess the root password by trying multiple password combinations through the `su` utility.

### History & Background
- **Created:** 2010 by George Chatzisofroniou
- **Maintained:** Various
- **License:** GNU General Public License v2
- **Purpose:** Local privilege escalation via su brute-force

Sucrack is useful for testing local password security.

### When to Use This Tool
- **Penetration testing:** Test local password security
- **Privilege escalation:** Attempt root access
- **CTF challenges:** Solve local privilege escalation
- **Security assessments:** Evaluate su security

### Key Concepts
- **su:** Substitute user command
- **Brute-force:** Try all password combinations
- **Root:** Superuser account
- **Local escalation:** Gaining root access

---

## Chapter 2: Installation & Setup

### System Requirements
- **Privileges:** Root (required for su)
- **Disk Space:** ~5 MB

### Installation Methods

#### Method 1: APT (Recommended)
```bash
sudo apt update
sudo apt install sucrack
```

### Verification
```bash
sucrack --help
```

---

## Chapter 3: Basic Usage

### First Crack

#### Brute Force Root
```bash
sudo sucrack -w wordlist.txt
```
**Output:** Attempts to brute-force root password.

### Essential Commands

#### Dictionary Attack
```bash
sudo sucrack -w /usr/share/wordlists/rockyou.txt
```
**Explanation:** Tries words from wordlist against root.

### Complete Command Reference

#### Attack Options
| Flag | Description | Example |
|------|-------------|---------|
| `-w` | Wordlist file | `sucrack -w wordlist.txt` |
| `-u` | Username | `sucrack -u admin -w wordlist.txt` |
| `-s` | Sleep time | `sucrack -s 1 -w wordlist.txt` |

#### Output Options
| Flag | Description | Example |
|------|-------------|---------|
| `-v` | Verbose | `sucrack -v wordlist.txt` |

---

## Chapter 4: Intermediate Usage

### Advanced Cracking Techniques

#### Target Specific User
```bash
# Brute force specific user
sudo sucrack -u admin -w wordlist.txt
```

#### Sleep Between Attempts
```bash
# Add delay between attempts
sudo sucrack -s 1 -w wordlist.txt
```

### Scripting & Automation

#### Bash Scripting
```bash
#!/bin/bash
# Automated su cracking
WORDLIST="/usr/share/wordlists/rockyou.txt"

sudo sucrack -w $WORDLIST
```

### Combining with Other Tools

#### With Password Lists
```bash
# Use rockyou.txt
sudo sucrack -w /usr/share/wordlists/rockyou.txt

# Use custom list
sudo sucrack -w custom_passwords.txt
```

---

## Chapter 5: Advanced Usage

### Advanced Cracking Techniques

#### Custom Wordlist
```bash
# Generate custom wordlist
crunch 6 6 -t @@@%%% -o custom.txt

# Use with sucrack
sudo sucrack -w custom.txt
```

### Integration in Pentest Workflow

#### Phase 1: Initial Access
```bash
# Gain low-privilege access
ssh user@192.168.1.100
```

#### Phase 2: Privilege Escalation
```bash
# Attempt su brute-force
sudo sucrack -w /usr/share/wordlists/rockyou.txt
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: CTF Challenge
**Objective:** Escalate to root

**Steps:**
```bash
# 1. Check user access
id

# 2. Attempt su brute-force
sudo sucrack -w rockyou.txt

# 3. If successful, become root
su
```

---

## Chapter 7: Detection & Defense

### How Defenders Detect Sucrack

#### System Indicators
```
# Multiple su attempts
# Authentication failures
```

#### Log Analysis
```bash
# Check authentication logs
grep "su:" /var/log/auth.log
```

### Mitigation Strategies

#### Strong Root Password
```bash
# Use complex root password
# Implement account lockout
# Use PAM modules
```

### Blue Team Response
1. **Use strong root password** - Complex and long
2. **Implement account lockout** - Limit failed attempts
3. **Use PAM** - Pluggable authentication modules
4. **Monitor logs** - Detect brute force attempts

---

## Chapter 8: Troubleshooting

### Common Errors

#### Error: "Permission denied"
**Cause:** Not running as root
**Solution:**
```bash
# Run with sudo
sudo sucrack -w wordlist.txt
```

### FAQ

**Q: Is sucrack legal to use?**
A: Yes, sucrack is a legitimate security testing tool. However, using it without authorization is illegal.

**Q: Can sucrack crack any user?**
A: Yes, use `-u` flag to specify username.

**Q: How do I slow down attempts?**
A: Use `-s` flag to add sleep time between attempts.

---

## Resources

### Official Documentation
- [Sucrack Documentation](https://github.com/EdgeSecurity/sucrack)

### Related Tools
- **john:** John the Ripper
- **hashcat:** Advanced password recovery
- **hydra:** Network login cracker

### Practice Resources
- [HackTheBox](https://www.hackthebox.com/)
- [TryHackMe](https://tryhackme.com/)
