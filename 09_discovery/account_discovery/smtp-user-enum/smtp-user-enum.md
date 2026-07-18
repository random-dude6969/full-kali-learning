---
title: smtp-user-enum
category: discovery
subcategory: account_discovery
phase: 09
mitre_attack: TA0007
tags:
  - enumeration
  - smtp
  - users
  - email
  - brute_force
difficulty: beginner
os: linux
install: sudo apt install smtp-user-enum
github: https://pentestmonkey.net/audit/scripts/smtp-user-enum
related:
  - apache-users
  - smtp-vrfy
  - nmap
---

# smtp-user-enum

## Chapter 1: Overview

**smtp-user-enum** enumerates valid usernames on SMTP servers using the EXPN, VRFY, and RCPT TO commands. These commands can reveal valid email addresses and user accounts on mail servers.

### Key Features

- **Multiple methods**: EXPN, VRFY, RCPT TO
- **User enumeration**: Discover valid email accounts
- **Wordlist support**: Custom username lists
- **Parallel scanning**: Multi-threaded requests
- **Output formats**: Text, CSV

### SMTP Commands

| Command | Purpose | Status |
|---------|---------|--------|
| VRFY | Verify username | Often disabled |
| EXPN | Expand mailing list | Usually disabled |
| RCPT TO | Recipient test | Often enabled |

---

## Chapter 2: Installation & Setup

```bash
sudo apt update && sudo apt install smtp-user-enum
```

### Verify Installation

```bash
smtp-user-enum -h
```

---

## Chapter 3: Basic Usage

### VRFY Enumeration

```bash
smtp-user-enum -V VRFY -u admin -t 192.168.1.1
```

### EXPN Enumeration

```bash
smtp-user-enum -V EXPN -u admin -t 192.168.1.1
```

### RCPT TO Enumeration

```bash
smtp-user-enum -V RCPT -u admin -t 192.168.1.1
```

---

## Chapter 4: Wordlist Attack

### Specify Wordlist

```bash
smtp-user-enum -V RCPT -U users.txt -t 192.168.1.1
```

### Common Wordlist

```bash
smtp-user-enum -V RCPT -U /usr/share/wordlists/metasploit/unix_users.txt -t 192.168.1.1
```

### Custom Wordlist

```bash
echo -e "admin\nroot\ntest\nuser\nsupport" > custom.txt
smtp-user-enum -V RCPT -U custom.txt -t 192.168.1.1
```

---

## Chapter 5: Connection Options

### Specify Port

```bash
smtp-user-enum -V RCPT -U users.txt -t 192.168.1.1 -p 587
```

### Use SSL/TLS

```bash
smtp-user-enum -V RCPT -U users.txt -t 192.168.1.1 -S
```

### Username/Password Auth

```bash
smtp-user-enum -V RCPT -U users.txt -t 192.168.1.1 -M AUTH -u admin -p password
```

### From Address

```bash
smtp-user-enum -V RCPT -U users.txt -t 192.168.1.1 -f admin@target.com
```

---

## Chapter 6: Output Options

### Save Results

```bash
smtp-user-enum -V RCPT -U users.txt -t 192.168.1.1 -o results.txt
```

### Verbose Output

```bash
smtp-user-enum -V RCPT -U users.txt -t 192.168.1.1 -v
```

### Max Results

```bash
smtp-user-enum -V RCPT -U users.txt -t 192.168.1.1 -m 100
```

---

## Chapter 7: Real-World Scenarios

### Scenario 1: Basic SMTP Enumeration

```bash
smtp-user-enum -V RCPT -U /usr/share/wordlists/unix_users.txt -t 192.168.1.1
```

### Scenario 2: Full Mail Server Audit

```bash
# Try all methods
for method in VRFY EXPN RCPT; do
  echo "Testing $method..."
  smtp-user-enum -V $method -U users.txt -t 192.168.1.1 -o "${method}_results.txt"
done
```

### Scenario 3: Network Mail Enumeration

```bash
# Find SMTP servers
nmap -p 25,587,465 192.168.1.0/24

# Enumerate each
for ip in 192.168.1.{1..10}; do
  smtp-user-enum -V RCPT -U users.txt -t $ip -o "${ip}_smtp_users.txt"
done
```

---

## Chapter 8: Mitigation & Defense

**For administrators:**

- **Disable VRFY/EXPN**: Most common fix
- **Rate limiting**: Block enumeration attempts
- **Connection limits**: Limit per-IP connections
- **Logging**: Monitor for scanning
- **Firewall**: Restrict SMTP access

### Postfix Configuration

```postfix
# Disable VRFY
disable_vrfy_command = yes
```

### Sendmail Configuration

```sendmail
# Disable EXPN
PrivacyOptions=noexpn
```

### Exim Configuration

```exim
# Disable VRFY
smtp_disable_vrfy_command = true
```

---

## Resources

- [smtp-user-enum](https://pentestmonkey.net/audit/scripts/smtp-user-enum)
- [RFC 5321 - SMTP](https://tools.ietf.org/html/rfc5321)
- [OWASP Testing Guide](https://owasp.org/www-project-web-security-testing-guide/)
- [OffSec PEN-200 Course](https://www.offsec.com/courses/pen-200/)

---

*Generated for educational purposes in authorized security testing environments only.*
