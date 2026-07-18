---
tool_name: "dirsearch"
display_name: "DirSearch"
mitre_tactic: "reconnaissance"
mitre_tactic_id: "TA0043"
mitre_subcategory: "web_scanning"
install_command: "sudo apt install dirsearch"
version: "0.4.3"
github_repo: "https://github.com/maurosoria/dirsearch"
kali_tools_page: "https://www.kali.org/tools/dirsearch/"
difficulty_level: "beginner-to-intermediate"
requires_root: false
gui_available: false
tags: ["web", "directory", "brute-force", "path-scanner"]
related_tools: ["gobuster", "dirb", "feroxbuster", "ffuf"]
---

# DirSearch

## Chapter 1: Introduction and Overview

### What is DirSearch?

DirSearch is a web path brute-forcing tool written in Python. It's designed to discover hidden directories, files, and endpoints on web servers by systematically testing a wordlist against the target. DirSearch is known for its rich feature set, including recursive scanning, multiple HTTP method support, advanced filtering, and extensibility.

### History and Background

- **Created:** 2019 by Mauro Soria (maurosoria)
- **Maintained:** maurosoria on GitHub
- **License:** Apache 2.0
- **Language:** Python 3
- **Purpose:** Web path brute-forcing and content discovery

DirSearch was created to address limitations in existing directory brute-forcing tools. It offers advanced features like recursive scanning with depth control, custom HTTP methods, response filtering by regex, and extensibility through plugins. It has become one of the most popular web path scanners in the security community.

### When to Use This Tool

- **Penetration testing:** Discover hidden endpoints and directories
- **Security audits:** Map the complete web application structure
- **Bug bounty hunting:** Find hidden API endpoints and admin panels
- **CTF challenges:** Discover hidden flags and paths
- **Web application assessments:** Identify backup files, configuration files, and sensitive resources
- **API testing:** Discover undocumented API endpoints

### When NOT to Use It

- Against targets without written authorization
- When you need extreme speed (use feroxbuster for that)
- On very old Python versions (requires Python 3.6+)
- When you need GUI-based scanning (use dirbuster)

### Legal and Ethical Considerations

- Always obtain written permission before scanning targets you do not own
- DirSearch generates significant HTTP traffic and may trigger IDS/WAF alerts
- Be aware that directory brute-forcing may cause log noise
- Respect rate limits and scope boundaries defined in your engagement
- Document all findings and report them responsibly

### Key Concepts and Terminology

- **Wordlist:** A file containing potential directory and file names
- **Extension:** File extensions to test (e.g., .php, .html, .txt)
- **Recursive Scanning:** Scanning discovered directories for additional content
- **Filter:** Exclude results based on response codes, sizes, or regex patterns
- **HTTP Method:** GET, POST, PUT, DELETE, etc.
- **Plugin:** Extensibility mechanism for custom functionality
- **Thread:** Concurrent HTTP request

---

## Chapter 2: Installation and Setup

### System Requirements

- **Python:** 3.6 or higher
- **RAM:** 256 MB minimum (512 MB recommended)
- **Disk Space:** ~30 MB for DirSearch + wordlists
- **Network:** Internet connection to reach target
- **Privileges:** User-level (no root required)
- **Dependencies:** requests, urllib3, certifi

### Installation Methods

#### Method 1: APT (Recommended)
```bash
sudo apt update
sudo apt install dirsearch
```

#### Method 2: Pip
```bash
pip3 install dirsearch
```

#### Method 3: From Source
```bash
git clone https://github.com/maurosoria/dirsearch.git
cd dirsearch
pip3 install -r requirements.txt
python3 dirsearch.py
```

#### Method 4: Docker
```bash
docker run --rm -it ghcr.io/maurosoria/dirsearch:latest -u https://target.com
```

### Configuration Files and Environment Variables

DirSearch uses a configuration file for persistent settings:

- **Default config:** `~/.config/dirsearch/config.ini`
- **Custom config:** Specify with `-c` flag

```ini
# ~/.config/dirsearch/config.ini
[general]
threads = 30
extensions = php,html,txt
wordlist = /usr/share/wordlists/dirb/common.txt
```

### Verification Commands

```bash
# Verify installation
dirsearch --version
# Expected output: dirsearch v0.4.3

# Quick test
dirsearch -u https://example.com
```

---

## Chapter 3: Basic Usage

### First Scan

```bash
dirsearch -u https://target.com
```

**Sample Output:**
```
 _|. _ _  _  _  _ _|_    v0.4.3
(_||| _) (/_(_|| (_| )

Extensions: php, html, txt, js | HTTP method: GET | Threads: 30 | Wordlist size: 4612

Target: https://target.com

[10:00:00] Starting: https://target.com/
[10:00:01] 200 -   512B  - /admin
[10:00:01] 301 -     0B  - /backup -> https://target.com/backup/
[10:00:01] 200 -   256B  - /config
[10:00:02] 403 -   277B  - /dashboard
[10:00:02] 200 -  1024B  - /login
[10:00:02] 200 -   128B  - /robots.txt

Task Completed
```

### Essential Commands

#### 1. Basic Scan
```bash
dirsearch -u https://target.com
```
**Explanation:** Scans the target using default settings and wordlist.

#### 2. Custom Wordlist
```bash
dirsearch -u https://target.com -w /usr/share/wordlists/dirb/big.txt
```
**Explanation:** Uses a specific wordlist for scanning.

#### 3. Custom Extensions
```bash
dirsearch -u https://target.com -e php,html,txt,bak
```
**Explanation:** Tests for files with specific extensions.

#### 4. Exclude Extensions
```bash
dirsearch -u https://target.com -X pdf,png,jpg
```
**Explanation:** Excludes specific extensions from testing.

#### 5. Custom HTTP Method
```bash
dirsearch -u https://target.com -m POST -d "test=test"
```
**Explanation:** Uses POST method with data.

#### 6. Custom Headers
```bash
dirsearch -u https://target.com -H "Authorization: Bearer token123"
```
**Explanation:** Adds custom headers to all requests.

#### 7. Custom Cookies
```bash
dirsearch -u https://target.com --cookie "session=abc123; user=admin"
```
**Explanation:** Includes cookies in all requests.

#### 8. Exclude Status Codes
```bash
dirsearch -u https://target.com --exclude-status 404,403
```
**Explanation:** Excludes specific HTTP status codes from results.

#### 9. Recursive Scanning
```bash
dirsearch -u https://target.com -r -R 2
```
**Explanation:** Recursively scans found directories up to depth 2.

#### 10. Output to File
```bash
dirsearch -u https://target.com -o results.txt
```
**Explanation:** Saves results to a text file.

#### 11. Multiple URLs
```bash
dirsearch -l urls.txt
```
**Explanation:** Scans multiple URLs from a file.

#### 12. Verbose Mode
```bash
dirsearch -u https://target.com -v
```
**Explanation:** Shows detailed output including all tested URLs.

### COMPLETE Flag Reference

#### Target Options
| Flag | Description | Example |
|------|-------------|---------|
| `-u` | Single target URL | `dirsearch -u https://target.com` |
| `-l` | URLs file | `dirsearch -l urls.txt` |
| `-e` | Extensions | `dirsearch -u URL -e php,html,txt` |
| `-X` | Exclude extensions | `dirsearch -u URL -X pdf,png` |
| `-m` | HTTP method | `dirsearch -u URL -m POST` |
| `-d` | POST data | `dirsearch -u URL -d "key=value"` |
| `--data-file` | POST data file | `dirsearch -u URL --data-file data.txt` |

#### Header and Cookie Options
| Flag | Description | Example |
|------|-------------|---------|
| `-H` | Custom header | `dirsearch -u URL -H "Auth: token"` |
| `--header-file` | Headers file | `dirsearch -u URL --header-file headers.txt` |
| `--cookie` | Custom cookies | `dirsearch -u URL --cookie "session=abc"` |
| `--cookie-file` | Cookies file | `dirsearch -u URL --cookie-file cookies.txt` |
| `--user-agent` | User-Agent | `dirsearch -u URL --user-agent "Mozilla/5.0"` |
| `--random-agent` | Random User-Agent | `dirsearch -u URL --random-agent` |

#### Scan Options
| Flag | Description | Example |
|------|-------------|---------|
| `-w` | Wordlist | `dirsearch -u URL -w wordlist.txt` |
| `-t` | Thread count | `dirsearch -u URL -t 50` |
| `--timeout` | Request timeout | `dirsearch -u URL --timeout 10` |
| `--delay` | Delay between requests | `dirsearch -u URL --delay 1` |
| `--rate-limit` | Rate limit (req/s) | `dirsearch -u URL --rate-limit 100` |
| `-r` | Recursive scan | `dirsearch -u URL -r` |
| `-R` | Recursive depth | `dirsearch -u URL -r -R 3` |
| `--force-extensions` | Force extensions | `dirsearch -u URL --force-extensions` |
| `--no-extension` | No extensions | `dirsearch -u URL --no-extension` |
| `--lowercase` | Lowercase only | `dirsearch -u URL --lowercase` |
| `--uppercase` | Uppercase only | `dirsearch -u URL --uppercase` |

#### Filter Options
| Flag | Description | Example |
|------|-------------|---------|
| `--exclude-status` | Exclude status codes | `dirsearch -u URL --exclude-status 404,403` |
| `--include-status` | Include only these codes | `dirsearch -u URL --include-status 200,301` |
| `--exclude-sizes` | Exclude response sizes | `dirsearch -u URL --exclude-sizes 0,1234` |
| `--exclude-text` | Exclude text in response | `dirsearch -u URL --exclude-text "not found"` |
| `--exclude-regex` | Exclude regex match | `dirsearch -u URL --exclude-regex "error.*"` |
| `--minimal` | Min response size | `dirsearch -u URL --minimal 100` |
| `--maximal` | Max response size | `dirsearch -u URL --maximal 5000` |
| `-F` | Follow redirects | `dirsearch -u URL -F` |

#### Network Options
| Flag | Description | Example |
|------|-------------|---------|
| `--proxy` | HTTP proxy | `dirsearch -u URL --proxy http://127.0.0.1:8080` |
| `--proxy-file` | Proxy list file | `dirsearch -u URL --proxy-file proxies.txt` |
| `--tor` | Use Tor | `dirsearch -u URL --tor` |
| `--socks5` | SOCKS5 proxy | `dirsearch -u URL --socks5 socks5://127.0.0.1:9050` |

#### Output Options
| Flag | Description | Example |
|------|-------------|---------|
| `-o` | Output file | `dirsearch -u URL -o results.txt` |
| `--output-format` | Output format | `dirsearch -u URL --output-format json` |
| `-q` | Quiet mode | `dirsearch -u URL -q` |
| `-v` | Verbose mode | `dirsearch -u URL -v` |
| `--plain-text` | Plain text output | `dirsearch -u URL --plain-text results.txt` |
| `--json` | JSON output | `dirsearch -u URL --json results.json` |
| `--csv` | CSV output | `dirsearch -u URL --csv results.csv` |

#### Miscellaneous
| Flag | Description | Example |
|------|-------------|---------|
| `-c` | Config file | `dirsearch -u URL -c config.ini` |
| `--full-url` | Full URLs in output | `dirsearch -u URL --full-url` |
| `-h` / `--help` | Show help | `dirsearch -h` |
| `--version` | Show version | `dirsearch --version` |

### Output Format Explanation

DirSearch outputs results with detailed information:

```
[10:00:01] 200 -   512B  - /admin
[10:00:01] 301 -     0B  - /backup -> https://target.com/backup/
[10:00:02] 403 -   277B  - /dashboard
```

- **Timestamp:** When the result was found
- **Status Code:** HTTP response code
- **Size:** Response body size
- **URL:** Discovered path
- **Redirect:** Redirect target (for 301/302)

---

## Chapter 4: Intermediate Usage

### Bash Scripting and Automation

#### Batch Scanning
```bash
#!/bin/bash
# dirsearch_batch.sh

URLS_FILE="urls.txt"
OUTPUT_DIR="dirsearch_results"
mkdir -p $OUTPUT_DIR

while IFS= read -r url; do
    echo "[*] Scanning $url"
    dirsearch -u "$url" \
        -e php,html,txt,bak \
        -t 30 \
        --random-agent \
        -o "$OUTPUT_DIR/$(echo $url | tr '/:' '__').txt" \
        --output-format json
done < "$URLS_FILE"
```

#### Extension Fuzzing
```bash
#!/bin/bash
# ext_fuzz.sh

TARGET="https://target.com"
EXTENSIONS=(
    "php,html,txt"
    "bak,old,backup"
    "log,conf,config"
    "sql,db,sqlite"
    "xml,json,yaml"
)

for ext in "${EXTENSIONS[@]}"; do
    echo "[*] Testing: $ext"
    dirsearch -u "$TARGET" -e "$ext" -o "dirsearch_${ext//,/}.txt"
done
```

### Python Scripting Examples

#### DirSearch Wrapper
```python
#!/usr/bin/env python3
import subprocess
import json
import sys

def run_dirsearch(url, wordlist=None, extensions="php,html,txt",
                  threads=30, recursive=False, depth=1):
    """Run DirSearch and return results."""
    cmd = [
        "dirsearch",
        "-u", url,
        "-e", extensions,
        "-t", str(threads),
        "--json", "-o", "/dev/stdout"
    ]

    if wordlist:
        cmd.extend(["-w", wordlist])

    if recursive:
        cmd.extend(["-r", "-R", str(depth)])

    result = subprocess.run(cmd, capture_output=True, text=True)

    try:
        return json.loads(result.stdout)
    except json.JSONDecodeError:
        return {"error": result.stderr}

def main():
    url = sys.argv[1] if len(sys.argv) > 1 else "https://target.com"

    results = run_dirsearch(
        url,
        extensions="php,html,txt,bak",
        threads=50,
        recursive=True,
        depth=2
    )

    if "results" in results:
        for r in results["results"]:
            print(f"[{r['status']}] {r['url']} ({r['size']} bytes)")
    else:
        print("No results found")

if __name__ == "__main__":
    main()
```

### Tool Chaining

#### DirSearch + sqlmap Pipeline
```bash
# Find directories, then test for SQL injection
dirsearch -u https://target.com -e php -o dirs.txt

# Extract URLs and test with sqlmap
grep "^  " dirs.txt | awk '{print $4}' | while read path; do
    sqlmap -u "https://target.com${path}?id=1" --batch --random-agent
done
```

#### DirSearch + nuclei
```bash
# Find directories, then scan with nuclei
dirsearch -u https://target.com -e php,html,txt -o dirs.txt

# Extract URLs and scan
grep "^  " dirs.txt | awk '{print $4}' | \
    sed 's|^|https://target.com|' | \
    nuclei -l - -t nuclei-templates/
```

### Output Processing and Filtering

#### Filter by Status Code
```bash
# Show only 200 OK
dirsearch -u https://target.com --include-status 200

# Exclude 404 and 403
dirsearch -u https://target.com --exclude-status 404,403
```

#### Filter by Response Size
```bash
# Only responses between 100 and 5000 bytes
dirsearch -u https://target.com --minimal 100 --maximal 5000
```

#### Export to Different Formats
```bash
# JSON output
dirsearch -u https://target.com --json results.json

# CSV output
dirsearch -u https://target.com --csv results.csv

# Plain text
dirsearch -u https://target.com --plain-text results.txt
```

---

## Chapter 5: Advanced Usage

### Custom Wordlists and Configurations

#### Build Technology-Specific Wordlist
```bash
# WordPress directories
cat > wordpress_wordlist.txt << 'EOF'
wp-admin
wp-content
wp-includes
wp-json
wp-login.php
wp-register.php
xmlrpc.php
wp-config.php
wp-cron.php
wp-load.php
readme.html
license.txt
EOF

dirsearch -u https://target.com -w wordpress_wordlist.txt -e php
```

#### Configuration File
```ini
# my_config.ini
[general]
threads = 50
extensions = php,html,txt,bak
wordlist = /usr/share/wordlists/dirb/common.txt
timeout = 10
delay = 0

[headers]
Authorization = Bearer mytoken123
X-Forwarded-For = 127.0.0.1

[filters]
exclude_status = 404,403
minimal_size = 10
maximal_size = 10000
```

```bash
# Use custom config
dirsearch -u https://target.com -c my_config.ini
```

### Evasion Techniques

#### Random User-Agent
```bash
dirsearch -u https://target.com --random-agent
```

#### Proxy Chaining
```bash
# HTTP proxy
dirsearch -u https://target.com --proxy http://127.0.0.1:8080

# SOCKS5 proxy
dirsearch -u https://target.com --socks5 socks5://127.0.0.1:9050

# Tor
dirsearch -u https://target.com --tor

# Rotating proxies
dirsearch -u https://target.com --proxy-file proxies.txt
```

#### Rate Limiting
```bash
# Slow down requests
dirsearch -u https://target.com --delay 1 --rate-limit 50
```

### Integration with Pentest Workflows

#### Full Web Recon Workflow
```bash
#!/bin/bash
# full_dirsearch_recon.sh

TARGET="https://target.com"
WORDLIST="/usr/share/wordlists/dirb/common.txt"

echo "[Phase 1] Quick discovery..."
dirsearch -u "$TARGET" -w "$WORDLIST" -e php,html,txt \
    --random-agent -o phase1.txt

echo "[Phase 2] Extension testing..."
dirsearch -u "$TARGET" -w "$WORDLIST" -e bak,old,backup,log,conf \
    --random-agent -o phase2.txt

echo "[Phase 3] Recursive scan..."
dirsearch -u "$TARGET" -w "$WORDLIST" -e php -r -R 2 \
    --random-agent -o phase3.txt

echo "[Phase 4] Combine results..."
cat phase*.txt | grep "^  " | awk '{print $4}' | \
    sed 's|^|'"$TARGET"'|' | sort -u > all_dirs.txt

echo "[+] Found $(wc -l < all_dirs.txt) unique paths"

echo "[Phase 5] Vulnerability testing..."
while IFS= read -r url; do
    sqlmap -u "${url}?id=1" --batch --random-agent --level 1 2>/dev/null
done < all_dirs.txt
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: Web Application Security Audit

**Objective:** Discover all hidden paths and endpoints.

```bash
# Step 1: Quick discovery
dirsearch -u https://target.com -e php,html,txt -o quick.txt

# Step 2: Extended wordlist
dirsearch -u https://target.com -w /usr/share/wordlists/dirb/big.txt -o extended.txt

# Step 3: Backup files
dirsearch -u https://target.com -e bak,old,backup,save -o backups.txt

# Step 4: Recursive scanning
dirsearch -u https://target.com -r -R 2 -o recursive.txt

# Step 5: Combine results
cat quick.txt extended.txt backups.txt recursive.txt | \
  grep "^  " | awk '{print $4}' | sort -u > all_paths.txt

# Step 6: Analyze findings
echo "[+] Total unique paths: $(wc -l < all_paths.txt)"
grep "(200)" all_paths.txt  # Accessible paths
grep "(403)" all_paths.txt  # Forbidden paths (interesting)
```

### Scenario 2: CTF Challenge Walkthrough

**Objective:** Find the hidden flag.

```bash
# Step 1: Quick scan
dirsearch -u http://target.com:8080 -e php,html,txt

# Output:
# [200] /admin
# [301] /secret
# [200] /robots.txt

# Step 2: Check robots.txt
curl http://target.com:8080/robots.txt
# Reveals: /hidden/flag.txt

# Step 3: Find the flag
curl http://target.com:8080/hidden/flag.txt
# CTF{dirsearch_found_it}
```

### Scenario 3: Bug Bounty Methodology

**Objective:** Maximize endpoint discovery.

```bash
# Step 1: Multi-wordlist approach
for wordlist in /usr/share/wordlists/dirb/common.txt \
                /usr/share/wordlists/dirb/big.txt \
                /usr/share/seclists/Discovery/Web-Content/common.txt; do
    dirsearch -u https://target.com -w "$wordlist" \
        -e php,html,txt,js --random-agent \
        -o "ds_$(basename $wordlist).txt"
done

# Step 2: API endpoint discovery
dirsearch -u https://target.com/api/ -e php,html,json \
    --random-agent -o api_endpoints.txt

# Step 3: Combine and analyze
cat ds_*.txt api_endpoints.txt | \
  grep "^  " | awk '{print $4}' | sort -u > all_endpoints.txt

echo "[+] Found $(wc -l < all_endpoints.txt) unique endpoints"
```

---

## Chapter 7: Detection and Defense

### How Defenders Detect DirSearch

#### Network Indicators
```
# DirSearch request patterns:
# - Python requests User-Agent
# - High volume of 404 responses
# - Sequential directory name patterns
# - Consistent request timing
# - Multiple requests from same IP
```

#### Log Analysis
```bash
# Detect DirSearch patterns
grep -i "dirsearch" /var/log/nginx/access.log

# Check for Python User-Agent
grep "python-requests" /var/log/nginx/access.log

# High 404 rate detection
awk '{print $7}' access.log | sort | uniq -c | sort -rn | head -20
```

### IDS/IPS Signatures

```
# Snort/Suricata rules
alert http any any -> $HOME_NET any (msg:"DirSearch Detected";
  content:"User-Agent|3a| python-requests"; http_header;
  threshold:type both,track by_src,count 50,seconds 60;
  sid:2000001; rev:1;)

alert http any any -> $HOME_NET any (msg:"Web Path Brute-Force";
  threshold:type both,track by_src,count 100,seconds 60;
  sid:2000002; rev:1;)
```

### Mitigation Strategies

#### Rate Limiting (nginx)
```nginx
limit_req_zone $binary_remote_addr zone=dirsearch:10m rate=10r/s;

location / {
    limit_req zone=dirsearch burst=20 nodelay;
    limit_req_status 429;
}
```

#### Fail2Ban Configuration
```ini
[dirsearch]
enabled = true
port = http,https
filter = dirsearch
logpath = /var/log/nginx/access.log
maxretry = 100
findtime = 60
bantime = 3600
```

### Blue Team Response Playbook

1. **Detect** - Monitor for Python User-Agent and high 404 rates
2. **Analyze** - Check logs for sequential directory requests
3. **Block** - Apply rate limiting or block scanner IP
4. **Review** - Check if discovered content has vulnerabilities
5. **Harden** - Remove or restrict access to sensitive files
6. **Report** - Document the incident

---

## Chapter 8: Troubleshooting

### Common Errors and Solutions

#### Error: "ModuleNotFoundError: No module named 'requests'"
**Cause:** Missing Python dependencies.
```bash
# Install dependencies
pip3 install -r requirements.txt

# Or install manually
pip3 install requests urllib3 certifi
```

#### Error: "Connection refused"
**Cause:** Target is not accessible.
```bash
# Verify target
curl -I https://target.com

# Check DNS
nslookup target.com
```

#### Error: "SSL: CERTIFICATE_VERIFY_FAILED"
**Cause:** SSL certificate verification failure.
```bash
# Disable SSL verification
dirsearch -u https://target.com --no-check-certificate

# Or install certificates
pip3 install certifi
```

#### Error: "Timeout"
**Cause:** Target is slow to respond.
```bash
# Increase timeout
dirsearch -u https://target.com --timeout 30
```

### Performance Optimization

```bash
# Fast scan
dirsearch -u https://target.com -t 50 --timeout 5

# Thorough scan
dirsearch -u https://target.com -t 10 --timeout 30 -r -R 2

# Rate-limited scan
dirsearch -u https://target.com --rate-limit 100 --delay 0.5
```

### FAQ

**Q: Is DirSearch legal to use?**
A: Yes, DirSearch is a legitimate security testing tool. However, using it without authorization is illegal.

**Q: How do I update DirSearch?**
A: `sudo apt update && sudo apt upgrade dirsearch` or `pip3 install --upgrade dirsearch`

**Q: What's the difference between DirSearch and gobuster?**
A: DirSearch has more features (recursive scanning, filtering, plugins) but is slower. Gobuster is faster and simpler.

**Q: Can DirSearch bypass WAFs?**
A: DirSearch has proxy support and random User-Agent features. For serious WAF bypass, use it with Burp Suite.

**Q: Why does DirSearch find false positives?**
A: Use `--exclude-status` and `--minimal`/`--maximal` to filter results.

---

## Resources

### Official Documentation
- [DirSearch GitHub](https://github.com/maurosoria/dirsearch)
- [DirSearch Wiki](https://github.com/maurosoria/dirsearch/wiki)

### Related Tools
- **gobuster:** Faster directory brute-forcing
- **feroxbuster:** Recursive content discovery
- **dirb:** Simple directory brute-forcing
- **ffuf:** Web fuzzer

### Practice Labs
- [DVWA](https://github.com/digininja/DVWA)
- [HackTheBox](https://www.hackthebox.com/)
- [TryHackMe](https://tryhackme.com/)
