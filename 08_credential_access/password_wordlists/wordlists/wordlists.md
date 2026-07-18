---
tool_name: "wordlists"
display_name: "Wordlists"
mitre_tactic: "credential_access"
mitre_tactic_id: "TA0006"
mitre_subcategory: "password_wordlists"
install_command: "sudo apt install wordlists"
version: "1.0"
github_repo: ""
kali_tools_page: "https://www.kali.org/tools/wordlists/"
difficulty_level: "beginner"
requires_root: false
gui_available: false
tags: ["wordlist", "dictionary", "password", "common"]
related_tools: ["crunch", "cewl", "seclists"]
---

# Wordlists

## Chapter 1: Introduction & Overview

### What is Wordlists?
Wordlists is a collection of common wordlists used for password cracking, directory brute-forcing, and other security testing tasks.

### History & Background
- **Maintained:** Kali Linux
- **License:** Various
- **Purpose:** Provide common wordlists

Wordlists are essential for dictionary attacks.

### When to Use This Tool
- **Password cracking:** Dictionary attacks
- **Directory brute-forcing:** Find hidden paths
- **Subdomain discovery:** Find subdomains
- **Security assessments:** Test password policies

### Key Concepts
- **Wordlist:** List of words/passwords
- **Dictionary attack:** Try all words in list
- **Common passwords:** Frequently used passwords
- **Target-specific:** Customized lists

### Available Wordlists
| Wordlist | Location | Description |
|----------|----------|-------------|
| rockyou.txt | /usr/share/wordlists/ | Large password list |
| common.txt | /usr/share/wordlists/dirb/ | Common directories |
| big.txt | /usr/share/wordlists/dirb/ | Large directory list |
| SecLists | /usr/share/seclists/ | Comprehensive collection |

---

## Chapter 2: Installation & Setup

### System Requirements
- **Disk Space:** ~100 MB
- **Privileges:** User-level

### Installation Methods

#### Method 1: APT (Recommended)
```bash
sudo apt update
sudo apt install wordlists
```

#### Method 2: Download SecLists
```bash
git clone https://github.com/danielmiessler/SecLists.git
```

### Verification
```bash
ls /usr/share/wordlists/
```

---

## Chapter 3: Basic Usage

### First Use

#### List Available Wordlists
```bash
ls /usr/share/wordlists/
ls /usr/share/seclists/
```
**Output:** Shows available wordlists.

#### Use Wordlist
```bash
hydra -l admin -P /usr/share/wordlists/rockyou.txt ssh://target
```
**Explanation:** Uses rockyou.txt for brute-force.

### Essential Commands

#### List Wordlists
```bash
ls /usr/share/wordlists/
ls /usr/share/seclists/
```
**Explanation:** Shows available wordlists.

#### Extract Wordlist
```bash
gunzip /usr/share/wordlists/rockyou.txt.gz
```
**Explanation:** Extracts compressed wordlist.

### Common Wordlists

#### Password Lists
| Wordlist | Description | Size |
|----------|-------------|------|
| rockyou.txt | Common passwords | 14M |
| common.txt | Common words | 4K |
| password.txt | Password list | 1K |

#### Directory Lists
| Wordlist | Description | Size |
|----------|-------------|------|
| common.txt | Common directories | 4K |
| big.txt | Large directory list | 20K |
| medium.txt | Medium directory list | 10K |

#### Subdomain Lists
| Wordlist | Description | Size |
|----------|-------------|------|
| subdomains-top1million-5000.txt | Top 5K subdomains | 5K |
| subdomains-top1million-20000.txt | Top 20K subdomains | 20K |

---

## Chapter 4: Intermediate Usage

### Advanced Wordlist Techniques

#### Combining Wordlists
```bash
# Combine multiple wordlists
cat wordlist1.txt wordlist2.txt | sort -u > combined.txt
```

#### Filtering Wordlists
```bash
# Filter by length
awk 'length >= 8 && length <= 12' wordlist.txt > filtered.txt

# Filter by pattern
grep -E '^[a-zA-Z0-9]{8}$' wordlist.txt > pattern.txt
```

### Scripting & Automation

#### Bash Scripting
```bash
#!/bin/bash
# Generate target-specific wordlist
TARGET="target.com"
WORDLIST="/usr/share/wordlists/rockyou.txt"

# Filter wordlist
grep -i "$TARGET" $WORDLIST > "${TARGET}_wordlist.txt"

# Add common variations
echo "admin" >> "${TARGET}_wordlist.txt"
echo "password" >> "${TARGET}_wordlist.txt"
```

### Combining with Other Tools

#### With Hydra
```bash
# Use rockyou.txt
hydra -l admin -P /usr/share/wordlists/rockyou.txt ssh://target

# Use custom wordlist
hydra -l admin -P custom_wordlist.txt ssh://target
```

#### With Gobuster
```bash
# Use common directories
gobuster dir -u http://target.com -w /usr/share/wordlists/dirb/common.txt

# Use big directory list
gobuster dir -u http://target.com -w /usr/share/wordlists/dirb/big.txt
```

### Advanced Configurations

#### Custom Wordlists
```bash
# Create custom wordlist
cat > custom.txt << EOF
admin
password
123456
root
toor
EOF
```

### Performance Optimization

#### Reduce Wordlist Size
```bash
# Sort and unique
sort wordlist.txt | uniq > unique.txt

# Filter by length
awk 'length >= 8' wordlist.txt > filtered.txt
```

---

## Chapter 5: Advanced Usage

### Advanced Wordlist Techniques

#### Target-Specific Generation
```bash
# Generate from website
cewl http://target.com -w target_wordlist.txt

# Generate with crunch
crunch 8 8 -t @@@@%%%% -o custom_wordlist.txt
```

### Integration in Pentest Workflow

#### Phase 1: Wordlist Selection
```bash
# Choose appropriate wordlist
# rockyou.txt for passwords
# common.txt for directories
# subdomains.txt for subdomains
```

#### Phase 2: Attack
```bash
# Use wordlist with tool
hydra -l admin -P /usr/share/wordlists/rockyou.txt ssh://target
gobuster dir -u http://target.com -w /usr/share/wordlists/dirb/common.txt
```

### Advanced Configurations

#### Wordlist Optimization
```bash
# Remove duplicates
sort wordlist.txt | uniq > unique.txt

# Sort by length
awk '{print length, $0}' wordlist.txt | sort -n | cut -d' ' -f2- > sorted.txt
```

### Performance Optimization

#### Large Wordlists
```bash
# Use head for testing
head -1000 wordlist.txt > test.txt

# Split wordlist
split -l 100000 wordlist.txt chunk_
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: CTF Challenge
**Objective:** Find password

**Steps:**
```bash
# 1. Use common wordlist
hydra -l admin -P /usr/share/wordlists/rockyou.txt ssh://target

# 2. If not found, try larger list
hydra -l admin -P /usr/share/seclists/Passwords/Common-Credentials/10k-most-common.txt ssh://target
```

### Scenario 2: Directory Discovery
**Objective:** Find hidden directories

**Steps:**
```bash
# 1. Use common directories
gobuster dir -u http://target.com -w /usr/share/wordlists/dirb/common.txt

# 2. Use larger list
gobuster dir -u http://target.com -w /usr/share/wordlists/dirb/big.txt
```

### Scenario 3: Subdomain Discovery
**Objective:** Find subdomains

**Steps:**
```bash
# 1. Use subdomain list
gobuster dns -d target.com -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
```

---

## Chapter 7: Detection & Defense

### How Defenders Detect Wordlist Attacks

#### Network Indicators
```
# Multiple failed login attempts
# Sequential path requests
# Dictionary-based attacks
```

#### Log Analysis
```bash
# Check authentication logs
grep "Failed password" /var/log/auth.log | awk '{print $11}' | sort | uniq -c
```

### Mitigation Strategies

#### Strong Passwords
```bash
# Enforce minimum 12+ characters
# Require complexity
# Ban common passwords
```

#### Rate Limiting
```bash
# Configure fail2ban
apt install fail2ban
systemctl enable fail2ban
```

### Blue Team Response
1. **Enforce strong passwords** - 12+ characters
2. **Implement rate limiting** - Block brute force
3. **Monitor logs** - Detect dictionary attacks
4. **Use password filters** - Ban common passwords

---

## Chapter 8: Troubleshooting

### Common Errors

#### Error: "No such file"
**Cause:** Wordlist not found
**Solution:**
```bash
# Check location
ls /usr/share/wordlists/
ls /usr/share/seclists/

# Extract if needed
gunzip /usr/share/wordlists/rockyou.txt.gz
```

### Performance Optimization

#### Slow Attacks
```bash
# Use smaller wordlist
head -1000 /usr/share/wordlists/rockyou.txt > fast.txt

# Use targeted wordlist
grep -i "admin\|password\|123" /usr/share/wordlists/rockyou.txt > targeted.txt
```

### FAQ

**Q: Where are wordlists located?**
A: `/usr/share/wordlists/` and `/usr/share/seclists/`

**Q: How do I extract rockyou.txt?**
A: `gunzip /usr/share/wordlists/rockyou.txt.gz`

**Q: Can I create custom wordlists?**
A: Yes, use crunch or cewl to generate custom wordlists.

---

## Resources

### Official Documentation
- [SecLists GitHub](https://github.com/danielmiessler/SecLists)
- [Kali Wordlists](https://www.kali.org/tools/wordlists/)

### Related Tools
- **crunch:** Wordlist generator
- **cewl:** Custom wordlist from websites
- **seclists:** Comprehensive wordlist collection

### Practice Resources
- [HackTheBox](https://www.hackthebox.com/)
- [TryHackMe](https://tryhackme.com/)
