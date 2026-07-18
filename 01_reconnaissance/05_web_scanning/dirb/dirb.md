---
tool_name: "dirb"
display_name: "DIRB"
mitre_tactic: "reconnaissance"
mitre_tactic_id: "TA0043"
mitre_subcategory: "web_scanning"
install_command: "sudo apt install dirb"
version: "2.22"
github_repo: "https://github.com/mzfr/dirb"
kali_tools_page: "https://www.kali.org/tools/dirb/"
difficulty_level: "beginner"
requires_root: false
gui_available: false
tags: ["web", "directory", "brute-force", "content-discovery"]
related_tools: ["gobuster", "feroxbuster", "dirsearch", "ffuf"]
---

# DIRB (Directory Brute-Forcer)

## Chapter 1: Introduction and Overview

### What is DIRB?

DIRB is a web content scanner that brute-forces directories and files on web servers. It launches a dictionary-based attack against web servers and analyzes the responses to find hidden content such as directories, files, scripts, and other resources that are not directly linked from the application's main pages.

### History and Background

- **Created:** 2004 by TheDarkRider
- **Maintained:** Community contributors
- **License:** GPL v2
- **Language:** C
- **Purpose:** Web content discovery and directory brute-forcing

DIRB has been one of the most widely used directory brute-forcing tools in Kali Linux for over a decade. It was one of the first dedicated web content scanners and remains a go-to tool for many penetration testers due to its simplicity, reliability, and extensive wordlist collection.

### When to Use This Tool

- **Penetration testing:** Discover hidden directories and files on web servers
- **Security audits:** Map the complete web application structure
- **Bug bounty hunting:** Find hidden endpoints, admin panels, and backup files
- **CTF challenges:** Discover hidden flags and paths
- **Web application assessments:** Identify configuration files, backup files, and sensitive resources
- **Compliance testing:** Verify that sensitive files are not publicly accessible

### When NOT to Use It

- When you need speed (use feroxbuster or gobuster for faster scans)
- Against targets without written authorization
- On very large targets without rate limiting (DIRB can be aggressive)
- When you need recursive scanning with depth control (use feroxbuster)

### Legal and Ethical Considerations

- Always obtain written permission before scanning targets you do not own
- DIRB generates significant traffic and may trigger IDS/WAF alerts
- Be aware that directory brute-forcing may cause log noise
- Respect rate limits and scope boundaries defined in your engagement
- Some organizations consider directory brute-forcing as hostile activity

### Key Concepts and Terminology

- **Wordlist:** A file containing potential directory and file names
- **Dictionary Attack:** Systematically testing each word in a wordlist
- **HTTP Response Code:** Server responses (200 OK, 403 Forbidden, 404 Not Found, etc.)
- **Extension:** File extensions to test (e.g., .php, .html, .txt)
- **Recursive Scanning:** Scanning discovered directories for additional content
- **Base URL:** The target URL being scanned

---

## Chapter 2: Installation and Setup

### System Requirements

- **RAM:** 128 MB minimum
- **Disk Space:** ~20 MB for DIRB + wordlists
- **Network:** Internet connection to reach target
- **Privileges:** User-level (no root required)
- **Dependencies:** None (statically compiled)

### Installation Methods

#### Method 1: APT (Recommended)
```bash
sudo apt update
sudo apt install dirb
```

#### Method 2: From Source
```bash
git clone https://github.com/mzfr/dirb.git
cd dirb
./configure
make
sudo make install
```

#### Method 3: Manual Download
```bash
wget https://downloads.sourceforge.net/project/dirb/dirb222.tar.gz
tar -xzf dirb222.tar.gz
cd dirb222
./configure
make
sudo cp dirb /usr/local/bin/
```

### Configuration Files and Environment Variables

DIRB uses several configuration files:

- **Wordlists directory:** `/usr/share/wordlists/dirb/`
- **Default wordlist:** `/usr/share/wordlists/dirb/common.txt`
- **Big wordlist:** `/usr/share/wordlists/dirb/big.txt`
- **DIRB config:** `/etc/dirb/dirb.conf` (if configured)

### Verification Commands

```bash
# Verify installation
dirb
# Should display the help menu and banner

# Check version
dirb --help | head -5
# Expected: DIRB v2.22

# List available wordlists
ls -la /usr/share/wordlists/dirb/
```

---

## Chapter 3: Basic Usage

### First Scan

```bash
dirb https://target.com
```

**Sample Output:**
```
-----------------
DIRB v2.22
The Dark Raver
-----------------

START_TIME: Mon Jul 18 10:00:00 2026
URL_BASE: https://target.com/
WORDLIST_FILES: /usr/share/wordlists/dirb/common.txt
-----------------

GENERATED WORDS: 4612

---- Scanning URL: https://target.com/ - ----
+ https://target.com/admin (CODE:403|SIZE:277)
+ https://target.com/backup (CODE:200|SIZE:1024)
+ https://target.com/config (CODE:301|SIZE:310)
+ https://target.com/index.html (CODE:200|SIZE:5120)
+ https://target.com/login (CODE:200|SIZE:2048)
+ https://target.com/robots.txt (CODE:200|SIZE:128)

-----------------
END_TIME: Mon Jul 18 10:05:32 2026
URLS_FOUND: 6
-----------------
```

### Essential Commands

#### 1. Basic Scan with Default Wordlist
```bash
dirb https://target.com
```
**Explanation:** Scans the target using the default common.txt wordlist.

#### 2. Scan with Custom Wordlist
```bash
dirb https://target.com /usr/share/wordlists/dirb/big.txt
```
**Explanation:** Uses the larger wordlist for more thorough scanning.

#### 3. Scan with Custom Extensions
```bash
dirb https://target.com -X .php,.html,.txt
```
**Explanation:** Tests for files with specific extensions.

#### 4. Recursive Scanning
```bash
dirb https://target.com -r
```
**Explanation:** Recursively scans discovered directories.

#### 5. Verbose Mode
```bash
dirb https://target.com -v
```
**Explanation:** Shows detailed output including all tested URLs.

#### 6. Custom User-Agent
```bash
dirb https://target.com -a "Mozilla/5.0 (Windows NT 10.0; Win64; x64)"
```
**Explanation:** Uses a custom User-Agent string.

#### 7. Custom Headers
```bash
dirb https://target.com -H "Authorization: Bearer token123"
```
**Explanation:** Adds custom headers to all requests.

#### 8. Using Cookies
```bash
dirb https://target.com -z "session=abc123; user=admin"
```
**Explanation:** Includes cookies in all requests.

#### 9. Force Extensions
```bash
dirb https://target.com -Z
```
**Explanation:** Forces DIRB to try extensions on all URLs.

#### 10. Custom Timeouts
```bash
dirb https://target.com -t 30
```
**Explanation:** Sets a 30-second timeout for each request.

### COMPLETE Flag Reference

#### Target Options
| Flag | Description | Example |
|------|-------------|---------|
| `<URL>` | Target URL (required) | `dirb https://target.com` |
| `-l` | Scan using lowercase only | `dirb https://target.com -l` |
| `-r` | Recursive scanning | `dirb https://target.com -r` |
| `-R` | Recursive limit (depth) | `dirb https://target.com -R 3` |

#### Wordlist Options
| Flag | Description | Example |
|------|-------------|---------|
| (none) | Default wordlist | `dirb https://target.com` |
| (path) | Custom wordlist | `dirb https://target.com /path/to/wordlist.txt` |
| `-S` | Case-sensitive mode | `dirb https://target.com -S` |

#### Extension Options
| Flag | Description | Example |
|------|-------------|---------|
| `-X` | Custom extensions | `dirb https://target.com -X .php,.txt` |
| `-Z` | Force extensions | `dirb https://target.com -Z` |
| `-N` | Ignore response codes | `dirb https://target.com -N 404,403` |

#### Network Options
| Flag | Description | Example |
|------|-------------|---------|
| `-t` | Timeout (seconds) | `dirb https://target.com -t 30` |
| `-p` | Use proxy | `dirb https://target.com -p http://127.0.0.1:8080` |
| `-i` | Ignore case | `dirb https://target.com -i` |
| `-s` | Stop on redirection | `dirb https://target.com -s` |

#### Authentication Options
| Flag | Description | Example |
|------|-------------|---------|
| `-a` | Custom User-Agent | `dirb https://target.com -a "Mozilla/5.0"` |
| `-H` | Custom header | `dirb https://target.com -H "Auth: token"` |
| `-z` | Custom cookies | `dirb https://target.com -z "session=abc"` |

#### Output Options
| Flag | Description | Example |
|------|-------------|---------|
| `-o` | Output file | `dirb https://target.com -o results.txt` |
| `-v` | Verbose output | `dirb https://target.com -v` |
| `-S` | Silent mode | `dirb https://target.com -S` |

#### Miscellaneous
| Flag | Description | Example |
|------|-------------|---------|
| `-c` | Custom config file | `dirb https://target.com -c config.txt` |
| `-u` | Check for 302 redirects | `dirb https://target.com -u` |
| `-w` | Don't wait between requests | `dirb https://target.com -w` |
| `-h` | Show help | `dirb -h` |

### Output Format Explanation

DIRB outputs results with HTTP response codes and sizes:

```
+ https://target.com/admin (CODE:403|SIZE:277)
+ https://target.com/backup (CODE:200|SIZE:1024)
+ https://target.com/config (CODE:301|SIZE:310)
```

- **CODE:200** - Found (accessible)
- **CODE:301/302** - Redirect
- **CODE:403** - Forbidden (exists but restricted)
- **SIZE:** Response body size in bytes

---

## Chapter 4: Intermediate Usage

### Bash Scripting and Automation

#### Batch Directory Scanning
```bash
#!/bin/bash
# dirb_batch.sh - Scan multiple targets

TARGETS_FILE="targets.txt"
OUTPUT_DIR="dirb_results"
mkdir -p $OUTPUT_DIR

while IFS= read -r url; do
    echo "[*] Scanning $url"
    dirb "$url" -o "$OUTPUT_DIR/$(echo $url | tr '/:' '__').txt" -w
done < "$TARGETS_FILE"

echo "[+] All scans complete"
```

#### Extension Fuzzing
```bash
#!/bin/bash
# ext_fuzz.sh - Test multiple extension sets

TARGET="https://target.com"
EXTENSIONS=(
    ".php,.html,.txt"
    ".bak,.old,.backup"
    ".log,.conf,.config"
    ".sql,.db,.sqlite"
    ".xml,.json,.yaml"
)

for ext in "${EXTENSIONS[@]}"; do
    echo "[*] Testing extensions: $ext"
    dirb "$TARGET" -X "$ext" -o "dirb_${ext//,/}.txt"
done
```

### Python Scripting Examples

#### DIRB Wrapper
```python
#!/usr/bin/env python3
import subprocess
import json
import re

def run_dirb(url, wordlist=None, extensions=None):
    """Run DIRB and return parsed results."""
    cmd = ["dirb", url, "-w"]  # -w suppresses waiting message

    if wordlist:
        cmd.append(wordlist)

    if extensions:
        cmd.extend(["-X", extensions])

    result = subprocess.run(cmd, capture_output=True, text=True)

    # Parse output
    findings = []
    for line in result.stdout.split('\n'):
        if line.startswith('+ '):
            match = re.search(r'\+ (.+) \(CODE:(\d+)\|SIZE:(\d+)\)', line)
            if match:
                findings.append({
                    "url": match.group(1),
                    "status": int(match.group(2)),
                    "size": int(match.group(3))
                })

    return findings

def main():
    url = "https://target.com"
    results = run_dirb(url, extensions=".php,.html,.txt")

    for r in results:
        print(f"[{r['status']}] {r['url']} ({r['size']} bytes)")

if __name__ == "__main__":
    main()
```

### Tool Chaining

#### DIRB + sqlmap Pipeline
```bash
# Find directories, then test for SQL injection
dirb https://target.com -o dirs.txt

# Extract URLs and test with sqlmap
grep "^+ " dirs.txt | awk '{print $2}' | while read url; do
    sqlmap -u "$url?id=1" --batch --random-agent --level 2
done
```

#### DIRB + Searchsploit
```bash
# Find directories, then search for exploits
dirb https://target.com -o dirs.txt

# Search for known vulnerabilities
cat dirs.txt | grep "^+ " | awk '{print $2}' | while read url; do
    # Extract technology from URL
    searchsploit $(basename "$url" | cut -d. -f2)
done
```

### Output Processing and Filtering

#### Filter by Status Code
```bash
# Show only 200 OK responses
dirb https://target.com -o results.txt
grep "(CODE:200" results.txt

# Show only 403 Forbidden (interesting but restricted)
grep "(CODE:403" results.txt

# Exclude 404
grep -v "(CODE:404" results.txt
```

#### Sort by Size
```bash
# Sort findings by response size
dirb https://target.com -o results.txt
grep "^+ " results.txt | awk -F'SIZE:' '{print $2, $1}' | sort -n
```

---

## Chapter 5: Advanced Usage

### Custom Wordlists

#### Build a Custom Wordlist
```bash
# Create a custom wordlist from discovered content
cat > custom_wordlist.txt << 'EOF'
admin
login
api
config
backup
test
debug
internal
portal
dashboard
upload
files
docs
api/v1
api/v2
wp-admin
wp-content
phpmyadmin
.git
.env
.htaccess
web.config
EOF

# Use the custom wordlist
dirb https://target.com custom_wordlist.txt
```

#### Combine Multiple Wordlists
```bash
cat /usr/share/wordlists/dirb/common.txt \
    /usr/share/wordlists/dirb/big.txt \
    custom_wordlist.txt | sort -u > combined_wordlist.txt

dirb https://target.com combined_wordlist.txt
```

### Evasion Techniques

#### Rotating User-Agents
```bash
#!/bin/bash
AGENTS=(
    "Mozilla/5.0 (Windows NT 10.0; Win64; x64)"
    "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7)"
    "Mozilla/5.0 (X11; Linux x86_64)"
)

for agent in "${AGENTS[@]}"; do
    dirb https://target.com -a "$agent" -w -o "dirb_${agent// /_}.txt"
done
```

#### Proxy Chaining
```bash
# Route through Burp Suite
dirb https://target.com -p http://127.0.0.1:8080

# Route through Tor
dirb https://target.com -p socks5://127.0.0.1:9050
```

### Integration with Pentest Workflows

#### Full Web Recon Workflow
```bash
#!/bin/bash
# full_web_recon.sh

TARGET="https://target.com"
WORDLIST="/usr/share/wordlists/dirb/common.txt"

echo "[Phase 1] Initial directory discovery..."
dirb "$TARGET" "$WORDLIST" -o "phase1_dirs.txt"

echo "[Phase 2] Extension testing..."
dirb "$TARGET" "$WORDLIST" -X .php,.html,.txt,.bak -o "phase2_ext.txt"

echo "[Phase 3] Recursive scanning..."
dirb "$TARGET" "$WORDLIST" -r -R 3 -o "phase3_recursive.txt"

echo "[Phase 4] Combining results..."
cat phase*.txt | grep "^+ " | awk '{print $2}' | sort -u > all_dirs.txt
echo "[+] Found $(wc -l < all_dirs.txt) unique directories/files"

echo "[Phase 5] Vulnerability testing..."
cat all_dirs.txt | while read url; do
    sqlmap -u "$url?id=1" --batch --random-agent --level 1 2>/dev/null
done
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: Web Application Security Audit

**Objective:** Discover all hidden content on a web application.

```bash
# Step 1: Initial discovery
dirb https://target.com /usr/share/wordlists/dirb/common.txt -o initial.txt

# Step 2: Extended wordlist
dirb https://target.com /usr/share/wordlists/dirb/big.txt -o extended.txt

# Step 3: Extension testing
dirb https://target.com /usr/share/wordlists/dirb/common.txt -X .php,.html,.txt,.bak,.old -o extensions.txt

# Step 4: Recursive scanning of discovered directories
dirb https://target.com /usr/share/wordlists/dirb/common.txt -r -R 2 -o recursive.txt

# Step 5: Combine and analyze
cat initial.txt extended.txt extensions.txt recursive.txt | \
  grep "^+ " | awk '{print $2}' | sort -u > all_findings.txt

echo "[+] Total unique findings: $(wc -l < all_findings.txt)"

# Step 6: Check for sensitive files
for file in robots.txt .htaccess .env config.php backup.sql; do
    curl -s -o /dev/null -w "%{http_code}" "https://target.com/$file"
    echo " $file"
done
```

### Scenario 2: CTF Challenge Walkthrough

**Objective:** Find the hidden flag.

```bash
# Step 1: Quick scan
dirb http://target.com:8080

# Output shows:
# + http://target.com:8080/admin (CODE:403|SIZE:277)
# + http://target.com:8080/secret (CODE:301|SIZE:310)
# + http://target.com:8080/robots.txt (CODE:200|SIZE:128)

# Step 2: Check robots.txt
curl http://target.com:8080/robots.txt
# Reveals: /hidden_flag_location

# Step 3: Follow the trail
dirb http://target.com:8080/hidden_flag_location -X .txt,.html

# Step 4: Find the flag
curl http://target.com:8080/hidden_flag_location/flag.txt
```

### Scenario 3: Bug Bounty Methodology

**Objective:** Maximize hidden content discovery.

```bash
# Step 1: Multi-wordlist approach
for wordlist in /usr/share/wordlists/dirb/common.txt \
                /usr/share/wordlists/dirb/big.txt \
                /usr/share/seclists/Discovery/Web-Content/common.txt; do
    dirb https://target.com "$wordlist" -w -o "dirb_$(basename $wordlist).txt"
done

# Step 2: Extension fuzzing
dirb https://target.com /usr/share/wordlists/dirb/common.txt \
    -X .php,.html,.txt,.bak,.old,.backup,.log,.conf,.config \
    -o ext_fuzz.txt

# Step 3: Combine all results
cat dirb_*.txt ext_fuzz.txt | grep "^+ " | awk '{print $2}' | sort -u > all_findings.txt

# Step 4: Verify findings
while IFS= read -r url; do
    status=$(curl -s -o /dev/null -w "%{http_code}" "$url")
    echo "[$status] $url"
done < all_findings.txt
```

---

## Chapter 7: Detection and Defense

### How Defenders Detect DIRB

#### Network Indicators
```
# DIRB request patterns:
# - High volume of 404 responses
# - Sequential requests to common directory names
# - Consistent request timing
# - Default User-Agent string
# - Multiple requests from same IP in rapid succession
```

#### Log Analysis
```bash
# Detect directory brute-forcing
awk '{print $7}' access.log | sort | uniq -c | sort -rn | head -30

# Check for 404 flood
grep " 404 " access.log | awk '{print $1}' | sort | uniq -c | sort -rn | head -10

# Look for common wordlist entries
grep -E "(admin|login|config|backup|\.php|\.bak)" access.log | wc -l
```

### IDS/IPS Signatures

```
# Snort/Suricata rules
alert http any any -> $HOME_NET any (msg:"Directory Brute-Force Detected";
  threshold:type both,track by_src,count 100,seconds 60;
  sid:2000001; rev:1;)

alert http any any -> $HOME_NET any (msg:"DIRB User-Agent Detected";
  content:"User-Agent|3a| DIRB"; http_header;
  sid:2000002; rev:1;)
```

### Mitigation Strategies

#### Rate Limiting (nginx)
```nginx
limit_req_zone $binary_remote_addr zone=dirb:10m rate=10r/s;

location / {
    limit_req zone=dirb burst=20 nodelay;
    limit_req_status 429;
}
```

#### Fail2Ban Configuration
```ini
# /etc/fail2ban/jail.d/dirb.conf
[dirb]
enabled = true
port = http,https
filter = dirb
logpath = /var/log/nginx/access.log
maxretry = 100
findtime = 60
bantime = 3600
```

### Blue Team Response Playbook

1. **Detect** - Monitor for high 404 rates and sequential directory requests
2. **Analyze** - Check logs for wordlist patterns and scanner signatures
3. **Block** - Apply rate limiting or block scanner IP
4. **Review** - Check if any discovered content has vulnerabilities
5. **Remediate** - Remove or restrict access to sensitive files
6. **Report** - Document the incident and response

---

## Chapter 8: Troubleshooting

### Common Errors and Solutions

#### Error: "Can't connect to target"
**Cause:** Network connectivity issue or target is down.
```bash
# Verify target is accessible
curl -I https://target.com

# Check DNS resolution
nslookup target.com
```

#### Error: "SSL certificate problem"
**Cause:** SSL certificate verification failure.
```bash
# Disable SSL verification
dirb https://target.com --no-check-certificate

# Or use HTTP instead
dirb http://target.com
```

#### Error: "Too many redirects"
**Cause:** Target has a redirect loop.
```bash
# Limit redirects
dirb https://target.com -u
```

### Performance Optimization

```bash
# Fast scan (less thorough)
dirb https://target.com /usr/share/wordlists/dirb/common.txt -w

# Thorough scan (slower)
dirb https://target.com /usr/share/wordlists/dirb/big.txt

# Custom timeout
dirb https://target.com -t 10
```

### FAQ

**Q: Is DIRB legal to use?**
A: Yes, DIRB is a legitimate security testing tool. However, using it without authorization against systems you don't own is illegal.

**Q: How do I update DIRB?**
A: `sudo apt update && sudo apt upgrade dirb`

**Q: What's the difference between DIRB and gobuster?**
A: DIRB is simpler and has been around longer. Gobuster is faster and supports multiple modes (DNS, VHost, etc.). For modern pentesting, gobuster or feroxbuster are often preferred.

**Q: Can DIRB bypass WAFs?**
A: DIRB has limited WAF bypass capabilities. Use it with a proxy (like Burp Suite) for better evasion.

**Q: Why does DIRB find so many false positives?**
A: Some web servers return 200 OK for any path. Use `-N` to ignore certain response codes.

---

## Resources

### Official Documentation
- [DIRB Man Page](https://www.kali.org/tools/dirb/)
- [DIRB Usage Guide](https://github.com/mzfr/dirb)

### Tutorials
- [DIRB Tutorial - HackTricks](https://book.hacktricks.xyz/)
- [Web Content Discovery - PortSwigger](https://portswigger.net/web-security)

### Related Tools
- **gobuster:** Faster directory brute-forcing
- **feroxbuster:** Recursive content discovery
- **dirsearch:** Web path brute-forcing
- **ffuf:** Web fuzzer
- **wfuzz:** Web application fuzzer

### Practice Labs
- [DVWA](https://github.com/digininja/DVWA)
- [HackTheBox](https://www.hackthebox.com/)
- [TryHackMe](https://tryhackme.com/)
