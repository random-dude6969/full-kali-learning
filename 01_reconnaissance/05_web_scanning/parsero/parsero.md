---
tool_name: "parsero"
display_name: "Parsero"
mitre_tactic: "reconnaissance"
mitre_tactic_id: "TA0043"
mitre_subcategory: "web_scanning"
install_command: "sudo apt install parsero"
version: "0.9.2"
github_repo: "https://github.com/orsorio/parsero"
kali_tools_page: "https://www.kali.org/tools/parsero/"
difficulty_level: "beginner"
requires_root: false
gui_available: false
tags: ["web", "robots-txt", "recon", "discovery", "directories", "hidden-files"]
related_tools: ["dirb", "gobuster", "dirsearch", "feroxbuster"]
---

# Parsero

## Chapter 1: Introduction & Overview

### What is Parsero?
Parsero is a web content analysis tool that fetches and parses robots.txt files from web servers. It extracts all directories and files listed in Disallow directives, then checks their HTTP status codes to identify accessible (200 OK), redirected (301/302), or forbidden (403) paths. It also detects hidden files, backup files, and misconfigurations revealed through robots.txt.

### History & Background
- **Created:** 2013 by Jonas R. Orsorio
- **License:** GNU General Public License v3 (GPLv3)
- **Language:** Python
- **Purpose:** Analyze robots.txt for hidden content and misconfigurations
- **GitHub Stars:** 600+

### When to Use This Tool
- **Content discovery:** Find hidden files and directories listed in robots.txt
- **Reconnaissance:** Analyze robots.txt as part of web reconnaissance
- **Penetration testing:** Identify backup files, admin panels, and sensitive paths
- **Compliance audits:** Check what paths are being hidden from search engines
- **Bug bounty:** Find sensitive files accidentally exposed through robots.txt

### Legal and Ethical Considerations
- robots.txt is a publicly accessible file — fetching it is legal
- Following Disallow directives is voluntary (search engine convention)
- Use only on authorized targets
- Accessing discovered paths may be intrusive — verify authorization

### Key Concepts
- **robots.txt:** File at site root telling search engine crawlers which paths to avoid
- **Disallow:** Directive in robots.txt specifying paths to block from indexing
- **Sitemap:** XML file listing all URLs on a site, often referenced in robots.txt
- **HTTP Status Codes:** 200 (accessible), 301/302 (redirect), 403 (forbidden), 404 (not found)
- **Hidden Content:** Files/directories listed in robots.txt because they're sensitive

---

## Chapter 2: Installation & Setup

### System Requirements
- Python 3.6+
- Internet access
- No special permissions needed

### APT Installation (Kali Linux)
```bash
sudo apt update
sudo apt install parsero
```

**Sample output:**
```
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following packages were automatically installed and are no longer required:
  libpython3.11 python3.11
Use 'sudo apt autoremove' to remove them.
The following NEW packages will be installed:
  parsero
0 upgraded, 1 newly installed, 0 to remove and 45 not upgraded.
Need to get 87.4 kB of archives.
After this operation, 456 kB of additional disk space will be used.
Get:1 http://kali.download/kali kali-rolling/main amd64 parsero all 0.9.2-0kali1 [87.4 kB]
Fetched 87.4 kB in 1s (87.4 kB/s)
Selecting previously unselected package parsero.
(Reading database ... 312456 files and directories currently installed.)
Preparing to unpack .../parsero_0.9.2-0kali1_amd64.deb ...
Unpacking parsero (0.9.2-0kali1) ...
Setting up parsero (0.9.2-0kali1) ...
```

### From Source
```bash
git clone https://github.com/orsorio/parsero.git
cd parsero
pip3 install -r requirements.txt
```

### Verification
```bash
parsero --help
```
**Output:**
```
Parsero v0.9.2 - Robots.txt analyzer

usage: parsero.py [-h] [-u URL] [-r] [-s] [-d] [-t TIMEOUT] [-v] [-o OUTPUT]

optional arguments:
  -h, --help            show this help message and exit
  -u URL, --url URL     Target URL
  -r, --recursive       Recursive check of Disallow directories
  -s, --status          Show HTTP status codes
  -d, --disallow        Show only Disallow entries
  -t TIMEOUT, --timeout TIMEOUT
                        Request timeout in seconds
  -v, --verbose         Verbose output
  -o OUTPUT, --output OUTPUT
                        Output file
```

---

## Chapter 3: Basic Usage

### First Run
```bash
parsero -u https://target.com
```
**Sample output:**
```
[+] Fetching robots.txt from: https://target.com/robots.txt
[+] User-agent: *
Disallow: /admin/
Disallow: /backup/
Disallow: /config/
Disallow: /database/
Disallow: /private/
Disallow: /tmp/
Disallow: /wp-admin/
Disallow: /wp-config.php
Disallow: /wp-content/debug.log
Disallow: /xmlrpc.php

[+] Checking HTTP status codes...
[+] /admin/                    -> 403 Forbidden
[+] /backup/                   -> 200 OK
[+] /config/                   -> 403 Forbidden
[+] /database/                 -> 200 OK
[+] /private/                  -> 403 Forbidden
[+] /tmp/                      -> 403 Forbidden
[+] /wp-admin/                 -> 302 Redirect -> /wp-login.php
[+] /wp-config.php             -> 200 OK
[+] /wp-content/debug.log      -> 200 OK
[+] /xmlrpc.php                -> 200 OK

[+] Found 10 Disallow entries
[+] Accessible paths: 4
[+] Forbidden paths: 4
[+] Redirect paths: 2
```

### Verbose Mode
```bash
parsero -v -u https://target.com
```
**Output:**
```
[*] Target: https://target.com
[*] Fetching: https://target.com/robots.txt
[+] robots.txt found (342 bytes)
[+] Parsing Disallow directives...

User-agent: *
Disallow: /admin/
Disallow: /backup/
Disallow: /config/
Disallow: /private/
Disallow: /wp-admin/
Disallow: /wp-config.php
Disallow: /wp-content/debug.log

Sitemap: https://target.com/sitemap.xml

[+] Checking HTTP status codes...
[+] GET /admin/ -> 403 Forbidden (127 ms)
[+] GET /backup/ -> 200 OK (89 ms)
[+] GET /config/ -> 403 Forbidden (134 ms)
[+] GET /private/ -> 403 Forbidden (98 ms)
[+] GET /wp-admin/ -> 302 Redirect (112 ms)
[+] GET /wp-config.php -> 200 OK (145 ms)
[+] GET /wp-content/debug.log -> 200 OK (167 ms)

[+] Sitemap URL found: https://target.com/sitemap.xml
[+] Fetching sitemap.xml...
[+] Sitemap contains 156 URLs
```

### Essential Commands

#### Basic robots.txt Analysis
```bash
parsero -u https://target.com
```
**Explanation:** Fetches and parses robots.txt, showing all Disallow entries and their HTTP status.

#### Show HTTP Status Codes
```bash
parsero -s -u https://target.com
```
**Explanation:** Displays HTTP status codes for each discovered path.

#### Recursive Directory Check
```bash
parsero -r -u https://target.com
```
**Explanation:** Recursively checks subdirectories found in robots.txt.

#### Show Only Disallow Entries
```bash
parsero -d -u https://target.com
```
**Explanation:** Displays only Disallow directives without checking HTTP status.

### Complete Flags Reference

#### Target Specification
| Flag | Description | Example |
|------|-------------|---------|
| `-u`, `--url` | Target URL (required) | `parsero -u https://target.com` |

#### Scan Options
| Flag | Description | Example |
|------|-------------|---------|
| `-r`, `--recursive` | Recursive directory check | `parsero -r -u https://target.com` |
| `-s`, `--status` | Show HTTP status codes | `parsero -s -u https://target.com` |
| `-d`, `--disallow` | Show only Disallow entries | `parsero -d -u https://target.com` |
| `-t`, `--timeout` | Request timeout (seconds) | `parsero -t 30 -u https://target.com` |
| `-v`, `--verbose` | Verbose output | `parsero -v -u https://target.com` |
| `-o`, `--output` | Output file | `parsero -o results.txt -u https://target.com` |

---

## Chapter 4: Intermediate Usage

### Bash Automation
```bash
#!/bin/bash
# Multi-site robots.txt analysis
SITES=(
    "https://target1.com"
    "https://target2.com"
    "https://target3.com"
)
REPORT_DIR="parsero_$(date +%Y%m%d)"
mkdir -p "$REPORT_DIR"

for site in "${SITES[@]}"; do
    domain=$(echo "$site" | sed 's|https\?://||;s|/.*||')
    echo "=== $site ==="
    
    # Fetch and analyze robots.txt
    parsero -s -v -u "$site" 2>&1 | tee "${REPORT_DIR}/${domain}.txt"
    
    # Extract accessible paths
    grep "200 OK" "${REPORT_DIR}/${domain}.txt" > "${REPORT_DIR}/${domain}_accessible.txt" 2>/dev/null
    
    echo ""
done

echo "[+] Analysis complete. Reports in: $REPORT_DIR"
echo "[+] Summary of accessible paths:"
cat "${REPORT_DIR}"/*_accessible.txt 2>/dev/null
```

### Tool Chaining
```bash
# Parsero → Dirb for discovered paths
parsero -s -u https://target.com | grep "200 OK" | \
    awk '{print $2}' | \
    while read -r path; do
        dirb "https://target.com${path}" /usr/share/wordlists/dirb/common.txt
    done

# Parsero → Gobuster for recursive discovery
PATHS=$(parsero -s -u https://target.com | grep "200 OK" | awk '{print $2}')
for path in $PATHS; do
    echo "[*] Brute-forcing: $path"
    gobuster dir -u "https://target.com${path}" \
        -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt \
        -t 20
done

# Parsero → Nuclei for vulnerability scanning
parsero -s -u https://target.com | grep "200 OK" | \
    awk '{print $2}' | \
    xargs -I{} nuclei -u "https://target.com{}" -t exposures/
```

### Python Integration
```python
import subprocess
import json
import re

def parsero_scan(url, recursive=False, status=True):
    cmd = ["parsero", "-u", url]
    if recursive:
        cmd.append("-r")
    if status:
        cmd.append("-s")
    
    result = subprocess.run(cmd, capture_output=True, text=True)
    return result.stdout

def parse_accessible_paths(output):
    paths = []
    for line in output.split('\n'):
        if '200 OK' in line:
            match = re.search(r'\+\s+(/\S+)\s+->\s+200 OK', line)
            if match:
                paths.append(match.group(1))
    return paths

def parse_forbidden_paths(output):
    paths = []
    for line in output.split('\n'):
        if '403 Forbidden' in line:
            match = re.search(r'\+\s+(/\S+)\s+->\s+403 Forbidden', line)
            if match:
                paths.append(match.group(1))
    return paths

# Scan and analyze
output = parsero_scan("https://target.com", recursive=True)
accessible = parse_accessible_paths(output)
forbidden = parse_forbidden_paths(output)

print(f"[+] Accessible paths ({len(accessible)}):")
for path in accessible:
    print(f"  - {path}")

print(f"\n[+] Forbidden paths ({len(forbidden)}):")
for path in forbidden:
    print(f"  - {path}")
```

### Batch Processing
```bash
#!/bin/bash
# Parse robots.txt from multiple targets
INPUT_FILE="targets.txt"
OUTPUT_DIR="parsero_results"
mkdir -p "$OUTPUT_DIR"

while IFS= read -r target; do
    domain=$(echo "$target" | sed 's|https\?://||;s|/.*||')
    echo "[*] Processing: $target"
    
    # Fetch robots.txt directly
    curl -s -o "${OUTPUT_DIR}/${domain}_robots.txt" "${target}/robots.txt" 2>/dev/null
    
    # Run parsero
    parsero -s -u "$target" > "${OUTPUT_DIR}/${domain}_parsero.txt" 2>/dev/null
    
    # Extract interesting findings
    echo "=== ${domain} ===" >> "${OUTPUT_DIR}/summary.txt"
    grep -E "(200 OK|403 Forbidden)" "${OUTPUT_DIR}/${domain}_parsero.txt" >> "${OUTPUT_DIR}/summary.txt" 2>/dev/null
    echo "" >> "${OUTPUT_DIR}/summary.txt"
    
done < "$INPUT_FILE"

echo "[+] Batch processing complete. Summary:"
cat "${OUTPUT_DIR}/summary.txt"
```

---

## Chapter 5: Advanced Usage

### Recursive Scanning Deep Dive
```bash
# Recursive scan with verbose output
parsero -r -v -s -u https://target.com
```
**Output:**
```
[*] Target: https://target.com
[*] Fetching: https://target.com/robots.txt
[+] robots.txt found
[+] Disallow entries found: 8

[+] Checking: /admin/
[+] /admin/ -> 403 Forbidden
[+] Checking subdirectories of /admin/...
[+] /admin/config/ -> 200 OK
[+] /admin/users/ -> 200 OK
[+] /admin/logs/ -> 403 Forbidden

[+] Checking: /backup/
[+] /backup/ -> 200 OK
[+] Checking subdirectories of /backup/...
[+] /backup/db/ -> 200 OK
[+] /backup/site/ -> 200 OK

[+] Checking: /wp-admin/
[+] /wp-admin/ -> 302 Redirect
```

### Hidden Files Detection
```bash
# Check for common hidden files referenced in robots.txt
parsero -s -u https://target.com | grep -E "\.(bak|old|orig|save|sql|tar|gz|zip|conf|config|log|txt)$"
```

### Custom Wordlist Generation
```bash
# Generate wordlist from robots.txt Disallow entries
parsero -d -u https://target.com | awk '{print $2}' | \
    sed 's|/$||' | \
    sort -u > robots_wordlist.txt

# Add common suffixes
cat robots_wordlist.txt | while read -r path; do
    echo "$path"
    echo "$path/"
    echo "${path}.bak"
    echo "${path}.old"
    echo "${path}.txt"
    echo "${path}.php"
    echo "${path}.html"
done | sort -u > expanded_wordlist.txt

# Use with Gobuster
gobuster dir -u https://target.com -w expanded_wordlist.txt -t 20
```

### Sitemap Analysis
```bash
# Extract and analyze sitemap from robots.txt
SITEMAP_URL=$(curl -s https://target.com/robots.txt | grep -i "Sitemap:" | awk '{print $2}')
if [[ -n "$SITEMAP_URL" ]]; then
    echo "[+] Found sitemap: $SITEMAP_URL"
    curl -s "$SITEMAP_URL" | grep -oP '<loc>\K[^<]+' | head -50
fi
```

### Evasion Techniques
```bash
# Use custom User-Agent
curl -s -H "User-Agent: Googlebot/2.1 (+http://www.google.com/bot.html)" \
    https://target.com/robots.txt

# Slow requests to avoid rate limiting
parsero -s -t 10 -u https://target.com

# Route through proxy
curl -s --proxy http://proxy:8080 https://target.com/robots.txt
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: Web Reconnaissance — Finding Sensitive Files
**Objective:** Discover sensitive files exposed through robots.txt.

```bash
# Step 1: Fetch and analyze robots.txt
parsero -s -v -u https://target.com
# Output reveals: /wp-config.php (200 OK), /debug.log (200 OK), /backup/ (200 OK)

# Step 2: Check for WordPress config backup
curl -s https://target.com/wp-config.php.bak | head -20
# Output reveals database credentials!

# Step 3: Check debug log
curl -s https://target.com/wp-content/debug.log | head -50
# Output reveals PHP errors and file paths

# Step 4: Explore backup directory
curl -s https://target.com/backup/
# Lists database dumps and site backups

# Step 5: Download sensitive files
curl -O https://target.com/backup/database.sql
curl -O https://target.com/backup/site.tar.gz
```

### Scenario 2: CTF Challenge — Robots.txt Clue
**Objective:** Find the flag hidden behind robots.txt restrictions.

```bash
# Step 1: Check robots.txt
curl -s http://ctf-target.com/robots.txt
# Output:
# User-agent: *
# Disallow: /this-is-the-flag-location/

# Step 2: Access the hidden path
curl -s http://ctf-target.com/this-is-the-flag-location/
# Output: <html>... FLAG{robots_txt_is_not_security} ...</html>

# Step 3: Full parsero scan for more clues
parsero -s -v -u http://ctf-target.com
# Reveals additional hidden paths

# Step 4: Check each discovered path
for path in $(parsero -s -u http://ctf-target.com | grep "200 OK" | awk '{print $2}'); do
    echo "=== $path ==="
    curl -s "http://ctf-target.com${path}" | head -20
done
```

### Scenario 3: Corporate Security Audit — robots.txt Review
**Objective:** Audit robots.txt across all corporate web properties for security issues.

```bash
#!/bin/bash
# Corporate robots.txt audit
DOMAINS=("corp.com" "staging.corp.com" "dev.corp.com" "api.corp.com")
AUDIT_DIR="robots_audit_$(date +%Y%m%d)"
mkdir -p "$AUDIT_DIR"

for domain in "${DOMAINS[@]}"; do
    echo "[*] Auditing: $domain"
    
    # Check if robots.txt exists
    HTTP_CODE=$(curl -s -o "${AUDIT_DIR}/${domain}_robots.txt" -w "%{http_code}" "https://${domain}/robots.txt")
    
    if [[ "$HTTP_CODE" == "200" ]]; then
        echo "[+] robots.txt found"
        
        # Run parsero
        parsero -s -u "https://${domain}" > "${AUDIT_DIR}/${domain}_analysis.txt" 2>/dev/null
        
        # Check for sensitive paths
        SENSITIVE=$(grep -E "(config|backup|database|debug|password|secret|private|\.sql|\.bak)" \
            "${AUDIT_DIR}/${domain}_analysis.txt" 2>/dev/null)
        
        if [[ -n "$SENSITIVE" ]]; then
            echo "[!] WARNING: Sensitive paths found:"
            echo "$SENSITIVE"
            echo "$SENSITIVE" >> "${AUDIT_DIR}/findings.txt"
        else
            echo "[+] No sensitive paths found"
        fi
    else
        echo "[!] robots.txt not found (HTTP $HTTP_CODE)"
    fi
    echo ""
done

echo "[+] Audit complete. Findings:"
cat "${AUDIT_DIR}/findings.txt" 2>/dev/null || echo "No findings."
```

---

## Chapter 7: Detection & Defense

### How Defenders Detect Parsero

#### robots.txt Access Logging
Parsero fetches robots.txt and then probes each Disallow path — this creates a predictable log pattern:
```
GET /robots.txt 200
GET /admin/ 403
GET /backup/ 200
GET /config/ 403
```

#### Request Pattern Detection
```bash
# Find robots.txt access followed by multiple path probes
awk '$7 == "/robots.txt" {ip=$1; time=$4; next} $1 == ip && NR - prev < 100' access.log

# Detect sequential probing
awk '{print $1}' access.log | sort | uniq -c | sort -rn | head -10
```

#### Log Analysis
```bash
# Find all robots.txt access
grep "robots.txt" access.log

# Find access to Disallow paths
grep -E "(admin|backup|config|private|\.bak|\.sql)" access.log | head -20
```

### IDS/IPS Signatures

#### Snort Rule
```
alert http any any -> any any (msg:"Parsero robots.txt Analysis"; \
  content:"/robots.txt"; http_uri; \
  content:"GET"; http_method; \
  flow:to_server,established; \
  threshold:type both, track by_src, count 5, seconds 60; \
  classtype:web-application-attack; \
  sid:1000020; rev:1;)
```

#### Suricata Rule
```
alert http $HOME_NET any -> $EXTERNAL_NET any (msg:"Sequential robots.txt Probing"; \
  http.method; content:"GET"; \
  http.uri; content:"/robots.txt"; \
  flow:to_server,established; \
  threshold:type both, track by_src, count 3, seconds 30; \
  classtype:web-application-attack; \
  sid:2000020; rev:1;)
```

### Mitigation Strategies

#### 1. Don't List Sensitive Paths in robots.txt
robots.txt is public — listing sensitive paths tells attackers exactly where to look.

**Bad:**
```
User-agent: *
Disallow: /admin/
Disallow: /backup/
Disallow: /wp-config.php
```

**Better:**
```
User-agent: *
Disallow: /wp-admin/
```

#### 2. Use Authentication Instead of Disallow
```apache
# Apache .htaccess - protect sensitive directories
<Directory "/var/www/html/admin">
    Require ip 192.168.1.0/24
</Directory>

<Directory "/var/www/html/backup">
    Require all denied
</Directory>
```

#### 3. Rate Limiting
```nginx
# Nginx rate limiting
limit_req_zone $binary_remote_addr zone=robots:10m rate=2r/s;

location /robots.txt {
    limit_req zone=robots burst=5 nodelay;
}
```

#### 4. WAF Rules
```apache
# Block probing of Disallow paths
SecRule REQUEST_URI "@rx ^/(admin|backup|config|private)/" \
  "id:1000021,phase:1,deny,status:403,log,msg:'Disallow Path Access Blocked'"
```

#### 5. Remove Sensitive Files
```bash
# Don't leave backup files in web root
rm -f /var/www/html/wp-config.php.bak
rm -f /var/www/html/database.sql
rm -f /var/www/html/debug.log
```

### Blue Team Response Playbook
1. **Alert triggered:** Multiple path probes following robots.txt access
2. **Verify:** Check if scanning is authorized
3. **Document:** Log source IP, timestamp, paths accessed
4. **Block:** Add IP to firewall blocklist if unauthorized
5. **Review:** Check if accessed paths contain sensitive data
6. **Harden:** Remove sensitive paths from robots.txt, use auth instead
7. **Report:** Document incident and response actions

---

## Chapter 8: Troubleshooting

### Common Errors

#### "No robots.txt found"
**Cause:** Site doesn't have robots.txt or returns 404.
**Solution:**
```bash
# Check manually
curl -I https://target.com/robots.txt
# If 404, the site doesn't have robots.txt
```

#### "Connection timeout"
**Cause:** Target is slow or blocking requests.
**Solution:**
```bash
parsero -t 30 -s -u https://target.com
```

#### "SSL certificate problem"
**Cause:** Self-signed or invalid SSL certificate.
**Solution:**
```bash
curl -k https://target.com/robots.txt
# Or use --insecure flag with curl
```

#### "Empty output"
**Cause:** robots.txt exists but has no Disallow entries.
**Solution:**
```bash
curl -s https://target.com/robots.txt
# Verify robots.txt content
```

### Performance Optimization
```bash
# Quick scan (no HTTP status checks)
parsero -d -u https://target.com

# Full scan (with status checks and recursion)
parsero -r -s -v -u https://target.com
```

### FAQ

**Q: Does Parsero work with non-standard robots.txt paths?**
A: No, it only checks `/robots.txt` at the site root.

**Q: Can Parsero handle multiple targets?**
A: Not natively. Use a bash loop to iterate over targets.

**Q: What if robots.txt blocks Parsero?**
A: robots.txt is advisory — Parsero ignores Crawl-delay and User-agent rules.

**Q: How do I handle rate limiting?**
A: Use `-t` to increase timeout between requests.

---

## Resources

- [Parsero GitHub](https://github.com/orsorio/parsero)
- [robots.txt Specification](https://www.robotstxt.org/robotstxt.html)
- [Google robots.txt Documentation](https://developers.google.com/search/docs/crawling-indexing/robots/intro)
- [MITRE ATT&CK TA0043](https://attack.mitre.org/tactics/TA0043/)
- [OWASP Content Discovery](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/02-Configuration_and_Deployment_Management_Testing/05-Enumerate_Infrastructure_and_Application_Administrators)
