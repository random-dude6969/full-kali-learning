---
title: apache-users
category: discovery
subcategory: account_discovery
phase: 09
mitre_attack: TA0007
tags:
  - enumeration
  - apache
  - users
  - web
  - brute_force
difficulty: beginner
os: linux
install: sudo apt install apache-users
github: https://github.com/kozmer/sherlock-py
related:
  - smtp-user-enum
  - dirb
  - gobuster
---

# apache-users

## Chapter 1: Overview

**apache-users** is a tool designed to enumerate valid usernames on Apache HTTP servers using the UserDir module. It exploits the common misconfiguration where Apache allows user directories (e.g., `~username`) to discover valid user accounts.

### Key Features

- **Username enumeration**: Discover valid user accounts
- **UserDir module detection**: Identifies Apache user directories
- **Multiple wordlists**: Use custom or default username lists
- **Response analysis**: Distinguishes valid vs invalid users
- **Fast scanning**: Multi-threaded requests

### Attack Principle

```
Apache UserDir Module:
1. Access http://target/~username/
2. Valid user: 200 OK (or 403 Forbidden)
3. Invalid user: 404 Not Found
4. Enumerate all valid usernames
```

---

## Chapter 2: Installation & Setup

```bash
sudo apt update && sudo apt install apache-users
```

### Dependencies

```bash
sudo apt install libwww-perl
```

### Verify Installation

```bash
apache-users --help
```

---

## Chapter 3: Basic Usage

### Scan with Default Wordlist

```bash
apache-users -h 192.168.1.1
```

### Specify Wordlist

```bash
apache-users -h 192.168.1.1 -w /usr/share/wordlists/common_users.txt
```

### Set Thread Count

```bash
apache-users -h 192.168.1.1 -t 10
```

### Verbose Output

```bash
apache-users -h 192.168.1.1 -v
```

---

## Chapter 4: Scan Options

### Specify Port

```bash
apache-users -h 192.168.1.1 -p 8080
```

### HTTPS Scan

```bash
apache-users -h 192.168.1.1 -s
```

### Custom UserDir

```bash
apache-users -h 192.168.1.1 -d "public_html"
```

### Filter by Status Code

```bash
apache-users -h 192.168.1.1 --filter 403
```

---

## Chapter 5: Wordlists

### Default Wordlist Location

```bash
ls /usr/share/wordlists/
```

### Custom Wordlist

```bash
# Create custom list
echo -e "admin\nroot\ntest\nuser" > custom_users.txt

# Use custom list
apache-users -h 192.168.1.1 -w custom_users.txt
```

### Common Usernames

```
admin
administrator
root
test
user
guest
info
support
```

---

## Chapter 6: Advanced Features

### Multiple Targets

```bash
for ip in 192.168.1.{1..10}; do
  apache-users -h $ip -w users.txt
done
```

### Output to File

```bash
apache-users -h 192.168.1.1 -o found_users.txt
```

### Proxy Support

```bash
apache-users -h 192.168.1.1 --proxy http://127.0.0.1:8080
```

---

## Chapter 7: Real-World Scenarios

### Scenario 1: Quick User Enumeration

```bash
apache-users -h 192.168.1.1
```

### Scenario 2: Full Assessment

```bash
# Scan for UserDir
apache-users -h 192.168.1.1 -v

# Try additional wordlists
apache-users -h 192.168.1.1 -w /usr/share/wordlists/rockyou.txt
```

### Scenario 3: Network Audit

```bash
# Scan entire subnet
nmap -p 80 192.168.1.0/24 -oG - | awk '/Up$/{print $2}' > hosts.txt

while read host; do
  apache-users -h $host -o "${host}_users.txt"
done < hosts.txt
```

---

## Chapter 8: Mitigation & Defense

**For administrators:**

- **Disable UserDir module**: Most secure option
- **Restrict UserDir**: Limit to specific users
- **Custom error pages**: Prevent information leakage
- **Rate limiting**: Block enumeration attempts
- **Log monitoring**: Detect scanning activity
- **Apache configuration**:

```apache
# Disable UserDir
UserDir disabled

# Or restrict to specific users
UserDir enabled username1 username2
```

---

## Resources

- [Apache UserDir Documentation](https://httpd.apache.org/docs/2.4/mod/mod_userdir.html)
- [OWASP Testing Guide](https://owasp.org/www-project-web-security-testing-guide/)
- [CWE-200: Information Exposure](https://cwe.mitre.org/data/definitions/200.html)
- [OffSec PEN-200 Course](https://www.offsec.com/courses/pen-200/)

---

*Generated for educational purposes in authorized security testing environments only.*
