---
tool_name: "swaks"
display_name: "Swiss Army Knife for SMTP"
mitre_tactic: "discovery"
mitre_tactic_id: "TA0007"
mitre_subcategory: "account_discovery"
install_command: "sudo apt install swaks"
version: "20240102"
github_repo: "https://github.com/fullstackarchive/swaks"
kali_tools_page: "https://www.kali.org/tools/swaks/"
difficulty_level: "beginner"
requires_root: false
gui_available: false
tags: ["smtp", "email", "testing", "sending", "diagnostic"]
related_tools: ["smtp-user-enum", "nmap", "telnet"]
---

# Swaks

## Chapter 1: Introduction & Overview

### What is Swaks?
Swaks (Swiss Army Knife for SMTP) is a featureful SMTP test tool. It can send emails, test SMTP servers, and verify email configurations.

### History & Background
- **Created:** 2001 by John Googas
- **Maintained:** John Googas
- **License:** GNU General Public License v2
- **Purpose:** SMTP testing

Swaks tests SMTP servers.

### When to Use This Tool
- **Email testing:** Test SMTP servers
- **Penetration testing:** Test email security
- **Security assessments:** Verify email config
- **CTF challenges:** Email challenges

### Key Concepts
- **SMTP:** Simple Mail Transfer Protocol
- **Email:** Electronic mail
- **Authentication:** Login verification
- **TLS:** Transport Layer Security

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
sudo apt install swaks
```

### Verification
```bash
swaks --version
```

---

## Chapter 3: Basic Usage

### First Use

#### Basic Test
```bash
swaks --to user@target.com --from test@localhost --server 192.168.1.1
```
**Output:** Sends test email.

### Essential Commands

#### Basic Test
```bash
swaks --to user@target.com --from test@localhost --server 192.168.1.1
```
**Explanation:** Sends test email.

#### With Authentication
```bash
swaks --to user@target.com --from test@localhost --server 192.168.1.1 --auth PLAIN --auth-user admin --auth-password pass
```
**Explanation:** Sends authenticated email.

### Complete Flags Reference

#### Target Options
| Flag | Description | Example |
|------|-------------|---------|
| `--server` | SMTP server | `swaks --server target` |
| `--to` | Recipient | `swaks --to user@target.com` |
| `--from` | Sender | `swaks --from test@localhost` |
| `--port` | Port | `swaks --port 587` |

#### Authentication Options
| Flag | Description | Example |
|------|-------------|---------|
| `--auth` | Auth type | `swaks --auth PLAIN` |
| `--auth-user` | Username | `swaks --auth-user admin` |
| `--auth-password` | Password | `swaks --auth-password pass` |

#### Message Options
| Flag | Description | Example |
|------|-------------|---------|
| `--header` | Header | `swaks --header "Subject: Test"` |
| `--body` | Body | `swaks --body "Test message"` |
| `--attach` | Attachment | `swaks --attach file.txt` |

---

## Chapter 4: Intermediate Usage

### Advanced Testing

#### TLS Testing
```bash
# Test with TLS
swaks --to user@target.com --from test@localhost --server 192.168.1.1 --tls
```

#### Custom Headers
```bash
# Custom headers
swaks --to user@target.com --from test@localhost --server 192.168.1.1 --header "X-Custom: value"
```

### Scripting & Automation

#### Bash Scripting
```bash
#!/bin/bash
# SMTP testing
swaks --to user@target.com --from test@localhost --server 192.168.1.1
```

---

## Chapter 5: Advanced Usage

### Advanced Techniques

#### Multiple Tests
```bash
# Test multiple recipients
for user in admin user1 user2; do
    swaks --to $user@target.com --from test@localhost --server 192.168.1.1
done
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: Email Audit
**Objective:** Test SMTP server

**Steps:**
```bash
# 1. Test basic SMTP
swaks --to test@target.com --from test@localhost --server 192.168.1.1

# 2. Test authentication
swaks --to test@target.com --from test@localhost --server 192.168.1.1 --auth PLAIN --auth-user admin --auth-password pass

# 3. Test TLS
swaks --to test@target.com --from test@localhost --server 192.168.1.1 --tls
```

---

## Chapter 7: Detection & Defense

### Blue Team Response
1. **Enable authentication** - Require login
2. **Use TLS** - Encrypt connections
3. **Rate limit** - Prevent abuse
4. **Monitor logs** - Detect testing

---

## Chapter 8: Troubleshooting

### FAQ

**Q: Is swaks legal to use?**
A: Yes, swaks is a legitimate security testing tool. However, sending without authorization is illegal.

**Q: How do I update swaks?**
A: `sudo apt update && sudo apt upgrade swaks`

---

## Resources

### Related Tools
- **smtp-user-enum:** SMTP user enumeration
- **nmap:** Network scanner
- **telnet:** Telnet client

### Practice Resources
- [HackTheBox](https://www.hackthebox.com/)
- [TryHackMe](https://tryhackme.com/)
