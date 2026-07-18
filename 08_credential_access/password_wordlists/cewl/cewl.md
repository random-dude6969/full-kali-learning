---
tool_name: "cewl"
display_name: "CeWL"
mitre_tactic: "credential_access"
mitre_tactic_id: "TA0006"
mitre_subcategory: "password_wordlists"
install_command: "sudo apt install cewl"
version: "5.5.5"
github_repo: "https://github.com/digininja/CeWL"
kali_tools_page: "https://www.kali.org/tools/cewl/"
difficulty_level: "beginner-to-intermediate"
requires_root: false
gui_available: false
tags: ["wordlist", "generation", "web", "scraping", "custom"]
related_tools: ["crunch", "wordlists", "twofi"]
---

# CeWL

## Chapter 1: Introduction & Overview

### What is CeWL?
CeWL (Custom Word List generator) is a Ruby program that spiders a target URL to a specified depth and returns a list of words which can be used for password cracking.

### History & Background
- **Created:** 2009 by DigIninja
- **Maintained:** DigIninja
- **License:** GNU General Public License v3
- **Purpose:** Custom wordlist generation

CeWL creates targeted wordlists from websites.

### When to Use This Tool
- **Penetration testing:** Create custom wordlists
- **Password auditing:** Test password policies
- **CTF challenges:** Generate relevant passwords
- **Security assessments:** Custom dictionary attacks

### Key Concepts
- **Spidering:** Crawling websites
- **Wordlist:** List of potential passwords
- **Depth:** How deep to crawl
- **Lowercase:** Convert to lowercase

---

## Chapter 2: Installation & Setup

### System Requirements
- **Disk Space:** ~10 MB
- **Privileges:** User-level

### Installation Methods

#### Method 1: APT (Recommended)
```bash
sudo apt update
sudo apt install cewl
```

#### Method 2: From Source
```bash
git clone https://github.com/digininja/CeWL.git
cd CeWL
gem install -r cewl
```

### Dependencies
```bash
# Ruby
apt install ruby

# Gems
gem install mechanize
gem install ruby-readability
```

### Verification
```bash
cewl --version
```

---

## Chapter 3: Basic Usage

### First Wordlist

#### Generate Wordlist
```bash
cewl http://target.com -w wordlist.txt
```
**Output:** Creates wordlist from website.

#### Generate Lowercase
```bash
cewl -l http://target.com -w wordlist.txt
```
**Output:** Creates lowercase wordlist.

### Essential Commands

#### Basic Wordlist
```bash
cewl http://target.com -w wordlist.txt
```
**Explanation:** Crawls website and creates wordlist.

#### With Depth
```bash
cewl -d 3 http://target.com -w wordlist.txt
```
**Explanation:** Crawls 3 levels deep.

#### Lowercase
```bash
cewl -l http://target.com -w wordlist.txt
```
**Explanation:** Converts all words to lowercase.

### Complete Flags Reference

#### Spider Options
| Flag | Description | Example |
|------|-------------|---------|
| `-d` | Depth | `cewl -d 3 http://target.com` |
| `-m` | Min word length | `cewl -m 5 http://target.com` |
| `-M` | Max word length | `cewl -M 10 http://target.com` |
| `-w` | Output file | `cewl -w wordlist.txt http://target.com` |

#### Processing Options
| Flag | Description | Example |
|------|-------------|---------|
| `-l` | Lowercase | `cewl -l http://target.com` |
| `--lowercase` | Lowercase all | `cewl --lowercase http://target.com` |
| `--ua` | User-Agent | `cewl --ua "Mozilla/5.0" http://target.com` |
| `--meta` | Meta data | `cewl --meta http://target.com` |
| `--meta_file` | Meta file | `cewl --meta_file meta.txt http://target.com` |
| `--email` | Email addresses | `cewl --email http://target.com` |
| `--email_file` | Email file | `cewl --email_file emails.txt http://target.com` |

#### Authentication Options
| Flag | Description | Example |
|------|-------------|---------|
| `--auth_type` | Auth type | `cewl --auth_type basic http://target.com` |
| `--auth_user` | Username | `cewl --auth_user admin http://target.com` |
| `--auth_pass` | Password | `cewl --auth_pass pass http://target.com` |

#### Connection Options
| Flag | Description | Example |
|------|-------------|---------|
| `-a` | User-Agent | `cewl -a "Mozilla/5.0" http://target.com` |
| `--proxy` | Proxy | `cewl --proxy http://127.0.0.1:8080 http://target.com` |
| `--timeout` | Timeout | `cewl --timeout 10 http://target.com` |
| `--threads` | Threads | `cewl --threads 10 http://target.com` |
| `--no-words` | No words | `cewl --no-words http://target.com` |
| `--with-words` | With words | `cewl --with-words wordlist.txt http://target.com` |

#### Output Options
| Flag | Description | Example |
|------|-------------|---------|
| `-w` | Output file | `cewl -w wordlist.txt http://target.com` |
| `-v` | Verbose | `cewl -v http://target.com` |

---

## Chapter 4: Intermediate Usage

### Advanced Wordlist Techniques

#### Multiple Depths
```bash
# Shallow crawl
cewl -d 1 http://target.com -w shallow.txt

# Deep crawl
cewl -d 5 http://target.com -w deep.txt
```

#### Word Length Filtering
```bash
# Only 8+ character words
cewl -m 8 http://target.com -w long_words.txt

# Only 4-8 character words
cewl -m 4 -M 8 http://target.com -w medium_words.txt
```

### Scripting & Automation

#### Bash Scripting
```bash
#!/bin/bash
# Generate wordlists from multiple sites
SITES="http://target1.com http://target2.com http://target3.com"

for site in $SITES; do
    echo "[*] Crawling $site"
    cewl -d 3 -m 5 -w "${site##*/}_wordlist.txt" $site
done
```

#### Python Scripting
```python
#!/usr/bin/env python3
import subprocess

def cewl_generate(url, depth=3, min_length=5, output=None):
    """Generate wordlist with CeWL"""
    cmd = f"cewl -d {depth} -m {min_length}"
    if output:
        cmd += f" -w {output}"
    cmd += f" {url}"
    result = subprocess.run(cmd, shell=True, capture_output=True, text=True)
    return result.stdout

# Generate wordlist
cewl_generate("http://target.com", depth=3, output="wordlist.txt")
```

### Combining with Other Tools

#### With Crunch
```bash
# Generate base wordlist
cewl http://target.com -w base_wordlist.txt

# Generate variations
crunch 8 8 -t @@@@%%%% -o custom.txt
cat base_wordlist.txt custom.txt | sort -u > final_wordlist.txt
```

#### With Hydra
```bash
# Generate wordlist
cewl http://target.com -w wordlist.txt

# Use with hydra
hydra -l admin -P wordlist.txt ssh://target
```

### Advanced Configurations

#### Custom User-Agent
```bash
cewl -a "Mozilla/5.0 (Windows NT 10.0; Win64; x64)" http://target.com -w wordlist.txt
```

#### Proxy Support
```bash
cewl --proxy http://127.0.0.1:8080 http://target.com -w wordlist.txt
```

### Performance Optimization

#### Increase Threads
```bash
cewl --threads 10 http://target.com -w wordlist.txt
```

---

## Chapter 5: Advanced Usage

### Advanced Wordlist Techniques

#### Email Extraction
```bash
# Extract emails
cewl --email http://target.com -w emails.txt

# Extract emails with wordlist
cewl --email --email_file emails.txt http://target.com -w wordlist.txt
```

#### Meta Data Extraction
```bash
# Extract meta data
cewl --meta http://target.com -w meta_words.txt
```

### Integration in Pentest Workflow

#### Phase 1: Reconnaissance
```bash
# Generate custom wordlist
cewl -d 3 -m 5 http://target.com -w target_wordlist.txt
```

#### Phase 2: Password Testing
```bash
# Use wordlist with hydra
hydra -l admin -P target_wordlist.txt ssh://target

# Use with hashcat
hashcat -m 0 hash.txt target_wordlist.txt
```

### Advanced Configurations

#### Authentication
```bash
# Basic auth
cewl --auth_type basic --auth_user admin --auth_pass pass http://target.com -w wordlist.txt
```

### Performance Optimization

#### Large Sites
```bash
# Limit depth
cewl -d 2 http://target.com -w wordlist.txt

# Limit word length
cewl -m 5 http://target.com -w wordlist.txt
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: CTF Challenge
**Objective:** Generate password for target

**Steps:**
```bash
# 1. Generate wordlist from target
cewl -d 3 -m 5 http://target.com -w target_wordlist.txt

# 2. Use with hydra
hydra -l admin -P target_wordlist.txt ssh://target

# 3. Login with found password
ssh admin@target
```

### Scenario 2: Penetration Test
**Objective:** Test password policies

**Steps:**
```bash
# 1. Generate company wordlist
cewl -d 2 http://company.com -w company_wordlist.txt

# 2. Add common variations
cat company_wordlist.txt | sed 's/.*/\L&/' > lower.txt
cat company_wordlist.txt | sed 's/.*/\U&/' > upper.txt
cat lower.txt upper.txt | sort -u > final_wordlist.txt

# 3. Test passwords
hydra -l admin -P final_wordlist.txt ssh://target
```

---

## Chapter 7: Detection & Defense

### How Defenders Detect CeWL

#### Network Indicators
```
# Web crawling patterns
# Multiple page requests
# Unusual User-Agent
```

#### Log Analysis
```bash
# Check web server logs
grep "GET" access.log | awk '{print $1}' | sort | uniq -c | sort -rn

# Monitor for crawling
tail -f access.log | grep -i "cewl"
```

### Mitigation Strategies

#### Rate Limiting
```bash
# nginx rate limiting
limit_req_zone $binary_remote_addr zone=cewl:10m rate=10r/s;

location / {
    limit_req zone=cewl burst=20 nodelay;
}
```

#### Robots.txt
```bash
# Block crawling
User-agent: *
Disallow: /sensitive/
```

### Blue Team Response
1. **Implement rate limiting** - Block rapid crawling
2. **Use robots.txt** - Block sensitive areas
3. **Monitor logs** - Detect crawling patterns
4. **Filter User-Agents** - Block known tools

---

## Chapter 8: Troubleshooting

### Common Errors

#### Error: "Connection refused"
**Cause:** Target unreachable
**Solution:**
```bash
curl -I http://target.com
cewl -d 3 http://target.com -w wordlist.txt
```

#### Error: "No words found"
**Cause:** Site has no text content
**Solution:**
```bash
# Try different depth
cewl -d 5 http://target.com -w wordlist.txt

# Try different site
```

### Performance Optimization

#### Slow Crawling
```bash
# Reduce depth
cewl -d 1 http://target.com -w wordlist.txt

# Increase threads
cewl --threads 10 http://target.com -w wordlist.txt
```

### FAQ

**Q: Is cewl legal to use?**
A: Yes, cewl is a legitimate security testing tool. However, crawling without authorization may be illegal. Always get written permission before testing.

**Q: How do I update cewl?**
A: `sudo apt update && sudo apt upgrade cewl`

**Q: Can cewl crawl password protected sites?**
A: Yes, use `--auth_type`, `--auth_user`, and `--auth_pass` options.

**Q: How do I extract emails?**
A: Use `--email` flag: `cewl --email http://target.com -w emails.txt`

---

## Resources

### Official Documentation
- [CeWL GitHub](https://github.com/digininja/CeWL)
- [CeWL Wiki](https://github.com/digininja/CeWL/wiki)

### Related Tools
- **crunch:** Wordlist generator
- **wordlists:** Common wordlists
- **twofi:** Two-word frequency list

### Practice Resources
- [HackTheBox](https://www.hackthebox.com/)
- [TryHackMe](https://tryhackme.com/)
