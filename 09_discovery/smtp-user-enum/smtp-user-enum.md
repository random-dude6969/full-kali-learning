---
tool_name: "smtp-user-enum"
display_name: "SMTP User Enum"
mitre_tactic: "discovery"
mitre_tactic_id: "TA0007"
mitre_subcategory: "smtp"
install_command: "sudo apt install smtp-user-enum"
version: "1.3"
github_repo: ""
kali_tools_page: "https://www.kali.org/tools/smtp-user-enum/"
difficulty_level: "beginner"
requires_root: false
gui_available: false
tags: ["smtp", "email", "enumeration", "users", "accounts"]
related_tools: ["swaks", "nmap", "smtp-scan"]
---

# SMTP User Enum

## Chapter 1: Introduction & Overview

### What is SMTP User Enum?
SMTP User Enum is a tool for enumerating valid user accounts on SMTP servers. It tests for existing users via VRFY, EXPN, or RCPT commands.

### History & Background
- **Created:** 2006 by Pentest Monkey
- **Maintained:** Pentest Monkey
- **License:** GNU General Public License v2
- **Purpose:** SMTP user enumeration

SMTP User Enum discovers valid users.

### When to Use This Tool
- **Email testing:** Enumerate users
- **Penetration testing:** Find accounts
- **Security assessments:** Test SMTP security
- **CTF challenges:** Email challenges

### Key Concepts
- **SMTP:** Simple Mail Transfer Protocol
- **VRFY:** Verify user command
- **EXPN:** Expand mailing list
- **RCPT TO:** Recipient command

---

## Chapter 2: Installation & Setup

### System Requirements
- **Perl:** Required
- **Disk Space:** ~1 MB
- **Privileges:** User-level

### Installation Methods

#### Method 1: APT (Recommended)
```bash
sudo apt update
sudo apt install smtp-user-enum
```

### Verification
```bash
smtp-user-enum -h
```

---

## Chapter 3: Basic Usage

### First Use

#### Basic Enumeration
```bash
smtp-user-enum -M VRFY -U users.txt -t 192.168.1.1
```
**Output:** Lists valid users.

### Essential Commands

#### VRFY Enumeration
```bash
smtp-user-enum -M VRFY -U users.txt -t 192.168.1.1
```
**Explanation:** Enumerates users via VRFY.

### Complete Flags Reference

#### Target Options
| Flag | Description | Example |
|------|-------------|---------|
| `-t` | Target | `smtp-user-enum -t target` |
| `-p` | Port | `smtp-user-enum -p 25 target` |
| `-M` | Mode | `smtp-user-enum -M VRFY target` |

#### User Options
| Flag | Description | Example |
|------|-------------|---------|
| `-U` | User list | `smtp-user-enum -U users.txt` |
| `-u` | Single user | `smtp-user-enum -u admin` |

---

## Chapter 4: Intermediate Usage

### Advanced Enumeration

#### Multiple Modes
```bash
# Try different modes
smtp-user-enum -M VRFY -U users.txt -t target
smtp-user-enum -M EXPN -U users.txt -t target
smtp-user-enum -M RCPT -U users.txt -t target
```

---

## Chapter 5: Advanced Usage

### Advanced Techniques

#### Custom Port
```bash
# Use non-standard port
smtp-user-enum -M VRFY -U users.txt -t target -p 587
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: Email Audit
**Objective:** Find valid users

**Steps:**
```bash
# 1. Find SMTP service
nmap -p 25,587 target

# 2. Enumerate users
smtp-user-enum -M VRFY -U users.txt -t target
```

---

## Chapter 7: Detection & Defense

### Blue Team Response
1. **Disable VRFY** - Prevent enumeration
2. **Rate limit** - Block brute force
3. **Monitor logs** - Detect enumeration

---

## Chapter 8: Troubleshooting

### FAQ

**Q: Is smtp-user-enum legal to use?**
A: Yes, smtp-user-enum is a legitimate security testing tool. However, enumerating without authorization is illegal.

**Q: How do I update smtp-user-enum?**
A: `sudo apt update && sudo apt upgrade smtp-user-enum`

---

## Resources

### Related Tools
- **swaks:** Swiss Army Knife for SMTP
- **nmap:** Network scanner

### Practice Resources
- [HackTheBox](https://www.hackthebox.com/)
- [TryHackMe](https://tryhackme.com/)
