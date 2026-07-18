---
tool_name: "wfuzz"
display_name: "Wfuzz"
mitre_tactic: "reconnaissance"
mitre_tactic_id: "TA0043"
mitre_subcategory: "web_scanning"
install_command: "sudo apt install wfuzz"
version: "3.1"
github_repo: "https://github.com/xmendez/wfuzz"
kali_tools_page: "https://www.kali.org/tools/wfuzz/"
difficulty_level: "advanced"
requires_root: false
gui_available: false
tags: ["fuzzer", "web", "brute-force", "parameter"]
related_tools: ["ffuf", "gobuster", "fuzzdb"]
---

# Wfuzz

## Chapter 1: Introduction & Overview

### What is Wfuzz?

Wfuzz is a highly customizable web application fuzzing tool designed to brute-force web applications. It can fuzz GET/POST parameters, authentication fields, HTTP headers, directory and file names, virtual hosts, and virtually any part of an HTTP request. The tool supports multiple payload types (wordlists, ranges, lists, scripts), advanced filtering based on response codes, word count, line count, and byte size, and recursive fuzzing capabilities.

Unlike simpler directory brute-force tools, Wfuzz provides a powerful payload system with built-in encoders, multiple payload positions (FUZZ, FUZ2Z, etc.), and conditional payloads. This makes it suitable for everything from simple directory scanning to complex parameter fuzzing and credential testing.

### History & Background

- **Created:** 2011 by Xavier Mendez (xmendez)
- **Maintained:** Active community development
- **License:** GPL-2.0
- **Language:** Python 3
- **Latest version:** 3.1
- **Upstream repository:** https://github.com/xmendez/wfuzz

Wfuzz evolved from earlier fuzzing tools and was designed to be the most flexible web fuzzer available. Its modular payload and encoder system allows it to handle complex fuzzing scenarios that other tools cannot.

### When to Use This Tool

- **Directory and file discovery:** Find hidden directories, files, and endpoints
- **Parameter fuzzing:** Discover hidden GET/POST parameters
- **Virtual host discovery:** Find hosted domains on a web server
- **Credential testing:** Brute-force login forms and authentication
- **HTTP header fuzzing:** Test for header injection and manipulation
- **API endpoint discovery:** Fuzz REST API paths
- **CTF challenges:** Solve web-based capture-the-flag challenges
- **Bug bounty:** Discover hidden functionality in web applications

### Legal and Ethical Considerations

- Fuzzing generates high volumes of HTTP traffic
- Always obtain written authorization before fuzzing target applications
- Be aware of WAF rate limits and blocking thresholds
- Use filtering to reduce unnecessary traffic
- Some fuzzing patterns can trigger security alerts
- Respect robots.txt and scope limitations
- Document all findings for remediation reports

### Key Concepts

| Concept | Description |
|---------|-------------|
| **FUZZ** | Keyword marking the payload position in a URL or parameter |
| **Payload** | The data injected at FUZZ positions (wordlist, range, list, script) |
| **Encoder** | Transforms payloads before sending (base64, md5, url-encode) |
| **Filter** | Hides responses matching criteria (status code, word count, line count) |
| **Recursive** | Automatically fuzz discovered directories |
| **Clusterbomb** | Multiple payloads tried in all combinations |
| **Pitchfork** | Multiple payloads matched position-by-position |

---

## Chapter 2: Installation & Setup

### System Requirements

- Operating system: Linux, macOS, or Windows
- RAM: 512 MB minimum
- Disk space: < 20 MB
- Python: 3.7+
- Network: HTTP/HTTPS access to target
- Privileges: Non-root is sufficient (root needed for low-port binding)

### Installation via APT (Kali/Debian/Ubuntu)

```bash
sudo apt update
sudo apt install wfuzz
```

Expected output:
```
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following NEW packages will be installed:
  wfuzz
0 upgraded, 1 newly installed, 0 to remove and 0 not upgraded.
Need to get 156 kB of archives.
After this operation, 789 kB of additional disk space will be used.
Get:1 http://http.kali.org/kali kali-rolling/main amd64 wfuzz 3.1.0-2kali1 [156 kB]
Fetched 156 kB in 0s (780 kB/s)
Selecting previously unselected package wfuzz.
(Reading database ... 1234567 files and directories currently installed.)
Preparing to unpack .../wfuzz_3.1.0-2kali1_all.deb ...
Unpacking wfuzz (3.1.0-2kali1) ...
Setting up wfuzz (3.1.0-2kali1) ...
```

### Installation via pip

```bash
pip3 install wfuzz
```

### Installation from Source

```bash
git clone https://github.com/xmendez/wfuzz.git
cd wfuzz
pip3 install -r requirements.txt
python3 setup.py install
```

### Docker Installation

```bash
docker pull ghcr.io/xmendez/wfuzz:master
docker run --rm -v $(pwd)/wordlists:/wordlists ghcr.io/xmendez/wfuzz:master -z file,/wordlists/common.txt http://target.com/FUZZ
```

### Wordlist Preparation

```bash
# Create common wordlists directory
mkdir -p /usr/share/wordlists/wfuzz

# Link Kali default wordlists
ln -s /usr/share/wordlists/dirb/common.txt /usr/share/wordlists/wfuzz/
ln -s /usr/share/wordlists/dirb/big.txt /usr/share/wordlists/wfuzz/
ln -s /usr/share/seclists/Discovery/Web-Content/common.txt /usr/share/wordlists/wfuzz/seclists_common.txt
ln -s /usr/share/seclists/Discovery/Web-Content/raft-large-directories.txt /usr/share/wordlists/wfuzz/raft_large.txt
```

### Verification

```bash
wfuzz --version
```

Expected output:
```
Wfuzz 3.1.0 - A tool designed to bruteforce web applications
```

Or:

```bash
wfuzz --help 2>&1 | head -5
```

---

## Chapter 3: Basic Usage

### First Run

```bash
wfuzz -c -z file,/usr/share/wordlists/dirb/common.txt http://target.com/FUZZ
```

Sample output:
```
********************************************************
* Wfuzz 3.1.0 - A tool designed to bruteforce web applications *
********************************************************

Target: http://target.com/FUZZ
Total requests: 4614

==================================================================
ID           Response   Lines    Word     Chars       Payload
==================================================================

000000001    200        123      456      3456       admin
000000023    301        0        0        0          backup
000000045    200        89       234      1890       config
000000089    403        0        0        0          data
000000123    200        234      678      5432       images
000000167    200        156      345      2345       login
000000234    302        0        0        0          portal
000000312    200        67       189      1234       uploads
==================================================================

Total time: 45.234567
Requests/sec: 102.01
```

### Essential Commands

#### Directory Brute-force

```bash
wfuzz -c -z file,/usr/share/wordlists/dirb/common.txt --hc 404 http://target.com/FUZZ
```

**Explanation:** Brute-forces directories, hiding 404 (Not Found) responses to show only valid pages.

#### Hide by Word Count

```bash
wfuzz -c -z file,/usr/share/wordlists/dirb/common.txt --hw 50 http://target.com/FUZZ
```

**Explanation:** Hides responses with exactly 50 words (the default "not found" page word count).

#### Parameter Fuzzing

```bash
wfuzz -c -z file,params.txt http://target.com/page?FUZZ=test
```

**Explanation:** Fuzzes GET parameters to discover hidden ones.

#### POST Parameter Fuzzing

```bash
wfuzz -c -z file,params.txt -d "FUZZ=value" http://target.com/login
```

**Explanation:** Fuzzes POST parameters by injecting payloads into POST data.

#### Multiple Payloads (Clusterbomb)

```bash
wfuzz -c -z file,users.txt -z file,pass.txt -d "user=FUZZ&pass=FUZ2Z" http://target.com/login
```

**Explanation:** Tries every combination of usernames and passwords.

#### Virtual Host Fuzzing

```bash
wfuzz -c -z file,subdomains.txt -H "Host: FUZZ.target.com" http://target.com
```

**Explanation:** Discovers virtual hosts by varying the Host header.

#### Custom Header

```bash
wfuzz -c -z file,wordlist.txt -H "Authorization: Bearer FUZZ" http://target.com/api
```

**Explanation:** Fuzzes the Authorization header with different tokens.

#### Recursive Fuzzing

```bash
wfuzz -c -z file,wordlist.txt --recursion --recursion-depth 2 http://target.com/FUZZ
```

**Explanation:** Recursively fuzzes discovered directories up to depth 2.

### Complete Flags Reference

| Flag | Long Form | Description | Default | Example |
|------|-----------|-------------|---------|---------|
| `-c` | `--color` | Colored output | off | `wfuzz -c ...` |
| `-z` | `--payload` | Payload (type,options) | none (required) | `wfuzz -z file,wordlist.txt ...` |
| `-H` | `--header` | Custom header | none | `wfuzz -H "Auth: Bearer FUZZ" ...` |
| `-d` | `--data` | POST data | none | `wfuzz -d "user=FUZZ" ...` |
| `-X` | `--method` | HTTP method | GET | `wfuzz -X POST ...` |
| `-b` | `--cookie` | Cookie header | none | `wfuzz -b "session=FUZZ" ...` |
| `--hc` | `--hide-code` | Hide responses by code | none | `wfuzz --hc 404 ...` |
| `--hl` | `--hide-lines` | Hide by line count | none | `wfuzz --hl 0 ...` |
| `--hw` | `--hide-words` | Hide by word count | none | `wfuzz --hw 0 ...` |
| `--hh` | `--hide-chars` | Hide by char count | none | `wfuzz --hh 0 ...` |
| `-s` | `--sleep` | Delay between requests (sec) | 0 | `wfuzz -s 0.5 ...` |
| `-t` | `--threads` | Concurrent threads | 10 | `wfuzz -t 50 ...` |
| `-p` | `--proxy` | Proxy server | none | `wfuzz -p http://127.0.0.1:8080 ...` |
| `-f` | `--output-file` | Output file | none | `wfuzz -f output.txt ...` |
| `-R` | `--recursion` | Recursive mode | off | `wfuzz -R ...` |
| `-D` | `--recursion-depth` | Max recursion depth | 1 | `wfuzz -D 3 ...` |
| `-L` | `--follow` | Follow redirects | off | `wfuzz -L ...` |
| `--req-delay` | `--req-delay` | Min time between requests | 0 | `wfuzz --req-delay 0.1 ...` |
| `--conn-delay` | `--conn-delay` | Connection delay | 0 | `wfuzz --conn-delay 1 ...` |
| `-u` | `--url` | Target URL | none | `wfuzz -u http://target.com ...` |
| `-m` | `--method` | Fuzz method (clusterbomb, pitchfork, zipper) | clusterbomb | `wfuzz -m pitchfork ...` |
| `-e` | `--encoder` | Payload encoder | none | `wfuzz -e md5 ...` |
| `--script` | `--script` | Lua script | none | `wfuzz --script script.lua ...` |
| `-o` | `--output-format` | Output format (cii, json, csv) | cii | `wfuzz -o json ...` |
| `-v` | `--verbose` | Verbose output | off | `wfuzz -v ...` |
| `-q` | `--quiet` | Quiet mode | off | `wfuzz -q ...` |
| `-h` | `--help` | Show help | — | `wfuzz -h` |

### Payload Types

| Type | Syntax | Description | Example |
|------|--------|-------------|---------|
| file | `file,PATH` | Read payloads from file | `-z file,/usr/share/wordlists/dirb/common.txt` |
| range | `range,START-END` | Numeric range | `-z range,1-1000` |
| list | `list,"val1,val2,..."` | Comma-separated values | `-z list,"admin,root,test"` |
| stdin | `stdin` | Read from stdin | `cat wordlist.txt \| wfuzz -z stdin ...` |
| script | `script,NAME` | Lua script payload | `-z script,scenarios/usage` |
| binary | `binary,FILE` | Read binary file | `-z binary,payloads.bin` |
| buffer | `buffer,SIZE` | Buffer of specified size | `-z buffer,1024` |

### Payload Position Keywords

| Keyword | Description | Example |
|---------|-------------|---------|
| `FUZZ` | Primary payload position | `http://target.com/FUZZ` |
| `FUZ2Z` | Secondary payload position | `user=FUZZ&pass=FUZ2Z` |
| `FUZnZ` | nth payload position | Supports up to 1000 positions |

---

## Chapter 4: Intermediate Usage

### Advanced Filtering

```bash
# Hide multiple status codes
wfuzz -c -z file,wordlist.txt --hc 404,403,500 http://target.com/FUZZ

# Hide responses with specific word count range
wfuzz -c -z file,wordlist.txt --hw 0,50 http://target.com/FUZZ

# Only show responses with specific code
wfuzz -c -z file,wordlist.txt --sc 200 http://target.com/FUZZ

# Hide based on character count
wfuzz -c -z file,wordlist.txt --hh 0 http://target.com/FUZZ
```

### Multi-Payload Fuzzing

```bash
# Clusterbomb: try all combinations
wfuzz -c -z file,users.txt -z file,pass.txt \
    -d "user=FUZZ&pass=FUZ2Z" http://target.com/login

# Pitchfork: match position-by-position
wfuzz -c -z file,users.txt -z file,pass.txt -m pitchfork \
    -d "user=FUZZ&pass=FUZ2Z" http://target.com/login

# Three payloads
wfuzz -c -z file,users.txt -z file,pass.txt -z list,"admin,root" \
    -d "user=FUZZ&pass=FUZ2Z&role=FUZ3Z" http://target.com/register
```

### Encoders

```bash
# URL encoding
wfuzz -z file,wordlist.txt --encoder url http://target.com/FUZZ

# Base64 encoding
wfuzz -z file,wordlist.txt --encoder md5 http://target.com/FUZZ

# Multiple encoders
wfuzz -z file,wordlist.txt --encoder md5,base64 http://target.com/FUZZ
```

### Bash Automation

```bash
#!/bin/bash
# wfuzz_pipeline.sh — Comprehensive web fuzzing pipeline

TARGET="${1:?Usage: $0 <url>}"
WORDLIST="/usr/share/wordlists/dirb/common.txt"
OUTPUT_DIR="wfuzz_$(date +%Y%m%d)"
mkdir -p "$OUTPUT_DIR"

echo "=== Wfuzz Pipeline: $TARGET ==="

# Step 1: Directory Fuzzing
echo "[1/4] Directory fuzzing..."
wfuzz -c -z file,"$WORDLIST" --hc 404 -t 30 \
    "$TARGET/FUZZ" 2>/dev/null | tee "$OUTPUT_DIR/directories.txt"

# Step 2: File Extension Fuzzing
echo "[2/4] File extension fuzzing..."
wfuzz -c -z list,"php,txt,html,bak,old,sql,log" \
    "$TARGET/FUZZ.FUZ2Z" 2>/dev/null | tee "$OUTPUT_DIR/extensions.txt"

# Step 3: Parameter Discovery
echo "[3/4] Parameter fuzzing..."
wfuzz -c -z file,/usr/share/wordlists/dirb/common.txt \
    "$TARGET/page?FUZZ=1" 2>/dev/null | tee "$OUTPUT_DIR/parameters.txt"

# Step 4: Backup File Discovery
echo "[4/4] Backup file fuzzing..."
wfuzz -c -z list,"backup,bak,old,orig,save,tmp" \
    --hc 404 "$TARGET/FUZZ" 2>/dev/null | tee "$OUTPUT_DIR/backups.txt"

echo "[+] Pipeline complete. Results in $OUTPUT_DIR/"
```

### Python Integration

```python
#!/usr/bin/env python3
"""
wfuzz_automation.py — Programmatic Wfuzz integration
"""

import subprocess
import json
import re
from typing import List, Dict

def run_wfuzz(url: str, wordlist: str, method: str = "GET",
              filters: Dict = None, threads: int = 10) -> List[Dict]:
    """Run wfuzz and parse results."""
    cmd = ["wfuzz", "-c", "-z", f"file,{wordlist}", "-t", str(threads)]
    
    if filters:
        if "code" in filters:
            cmd.extend(["--hc", filters["code"]])
        if "words" in filters:
            cmd.extend(["--hw", str(filters["words"])])
    
    cmd.append(url)
    
    result = subprocess.run(cmd, capture_output=True, text=True, timeout=600)
    
    results = []
    for line in result.stdout.splitlines():
        match = re.match(r'\s*(\d+)\s+(\d+)\s+(\d+)\s+(\d+)\s+(\d+)\s+(.*)', line)
        if match:
            results.append({
                "id": int(match.group(1)),
                "status": int(match.group(2)),
                "lines": int(match.group(3)),
                "words": int(match.group(4)),
                "chars": int(match.group(5)),
                "payload": match.group(6).strip()
            })
    
    return results

# Example usage
results = run_wfuzz(
    url="http://target.com/FUZZ",
    wordlist="/usr/share/wordlists/dirb/common.txt",
    filters={"code": "404"}
)

print(f"Found {len(results)} valid paths:")
for r in results:
    print(f"  [{r['status']}] {r['payload']} ({r['words']} words)")
```

---

## Chapter 5: Advanced Usage

### Custom Wordlists

```bash
# Generate custom wordlist from target
cat > custom_wordlist.txt << 'EOF'
admin
administrator
login
portal
dashboard
api
backup
config
debug
test
dev
staging
internal
hidden
secret
private
upload
uploads
files
documents
images
img
css
js
static
assets
media
content
blog
news
forum
community
help
support
docs
documentation
wiki
status
health
monitor
EOF

wfuzz -c -z file,custom_wordlist.txt --hc 404 http://target.com/FUZZ
```

### Advanced Payload Manipulation

```bash
# Prefix and suffix payloads
wfuzz -z file,wordlist.txt --prefix "admin_" --suffix ".txt" http://target.com/FUZZ

# Replace in payload
wfuzz -z file,wordlist.txt --replace ".txt",".php" http://target.com/FUZZ

# Regex filter on payloads
wfuzz -z file,wordlist.txt --filter "payload~admin" http://target.com/FUZZ
```

### Rate Limiting and Stealth

```bash
# Slow down requests
wfuzz -c -z file,wordlist.txt -s 0.5 --hc 404 http://target.com/FUZZ

# Use proxy
wfuzz -c -z file,wordlist.txt -p http://127.0.0.1:8080 --hc 404 http://target.com/FUZZ

# Tor proxy
wfuzz -c -z file,wordlist.txt -p socks5://127.0.0.1:9050 --hc 404 http://target.com/FUZZ
```

### Complex Fuzzing Scenarios

```bash
# Cookie-based fuzzing
wfuzz -c -z file,wordlist.txt -b "session=FUZZ" http://target.com/dashboard

# Custom User-Agent
wfuzz -c -z file,wordlist.txt -H "User-Agent: FUZZ" http://target.com/FUZZ

# JSON body fuzzing
wfuzz -c -z file,wordlist.txt -H "Content-Type: application/json" \
    -d '{"username":"admin","password":"FUZZ"}' http://target.com/api/login
```

### Lua Script Payloads

```lua
-- custom_payload.lua
function main(context)
    -- Generate payloads dynamically
    for i = 1, 1000 do
        context:add("payload_" .. i)
    end
end
```

```bash
wfuzz -z script,custom_payload.lua http://target.com/FUZZ
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: Complete Web Application Audit

**Objective:** Perform comprehensive fuzzing of a web application.

**Step 1: Directory Discovery**
```bash
wfuzz -c -z file,/usr/share/wordlists/dirb/big.txt --hc 404 \
    -t 30 http://target.com/FUZZ 2>&1 | tee directories.txt
```

**Step 2: File Extension Discovery**
```bash
wfuzz -c -z list,"php,asp,aspx,jsp,html,txt,bak,old,sql,log,conf,xml,json" \
    http://target.com/FUZZ.FUZ2Z 2>&1 | tee extensions.txt
```

**Step 3: Parameter Discovery**
```bash
wfuzz -c -z file,/usr/share/wordlists/dirb/common.txt \
    http://target.com/page?FUZZ=test 2>&1 | tee parameters.txt
```

**Step 4: Credential Testing**
```bash
wfuzz -c -z file,/usr/share/seclists/Passwords/Default-Credentials/admin.txt \
    -z list,"password,admin,123456,letmein,welcome" \
    -d "username=FUZZ&password=FUZ2Z" \
    --hc 200 http://target.com/login 2>&1 | tee credentials.txt
```

### Scenario 2: CTF Challenge — "Find the Hidden Admin Panel"

**Objective:** Discover a hidden admin panel in a CTF web application.

**Step 1: Directory Enumeration**
```bash
wfuzz -c -z file,/usr/share/wordlists/dirb/big.txt --hc 404 \
    http://ctf-target.com/FUZZ 2>&1 | grep -v "Fuse:"
```

**Step 2: Check for Interesting Directories**
```bash
# Filter for directories with different response sizes
wfuzz -c -z file,/usr/share/wordlists/dirb/big.txt --hw 0 \
    http://ctf-target.com/FUZZ 2>&1 | tee ctf_results.txt
```

**Step 3: Fuzz Discovered Directories**
```bash
# Fuzz the admin directory
wfuzz -c -z file,/usr/share/wordlists/dirb/common.txt --hc 404 \
    http://ctf-target.com/admin/FUZZ 2>&1
```

**Step 4: Find Hidden Parameters**
```bash
wfuzz -c -z list,"id,user,cmd,exec,run,page,action,debug,admin,secret" \
    http://ctf-target.com/admin/dashboard?FUZZ=1 2>&1
```

### Scenario 3: Bug Bounty — Virtual Host Discovery

**Objective:** Find hidden virtual hosts hosted on the target server.

**Step 1: Prepare Subdomain Wordlist**
```bash
cat > vhosts.txt << 'EOF'
admin
api
beta
dev
 staging
internal
mail
portal
test
vpn
webmail
cpanel
whm
ftp
ns1
ns2
mx1
mx2
EOF
```

**Step 2: Virtual Host Fuzzing**
```bash
wfuzz -c -z file,vhosts.txt -H "Host: FUZZ.target.com" \
    --hc 404 http://target.com 2>&1 | tee vhosts.txt
```

**Step 3: Refine Results**
```bash
# Filter by response size
wfuzz -c -z file,vhosts.txt -H "Host: FUZZ.target.com" \
    --hw 0 http://target.com 2>&1 | tee vhosts_active.txt
```

**Step 4: Validate Discovered Hosts**
```bash
while read line; do
    vhost=$(echo "$line" | awk '{print $NF}')
    echo -n "$vhost: "
    curl -s -o /dev/null -w "%{http_code}" -H "Host: $vhost.target.com" http://target.com
    echo ""
done < vhosts_active.txt
```

---

## Chapter 7: Detection & Defense

### How Wfuzz Activity Is Detected

**HTTP Log Analysis:**
- High volume of 404 responses from same source
- Sequential or patterned request paths
- Unusual User-Agent strings
- Rapid request rates
- Requests to non-existent paths

**WAF Detection Signatures:**
```
# Common WAF rules that detect fuzzing
# ModSecurity rule example
SecRule REQUEST_URI "@rx FUZZ" "id:1001,phase:1,deny,status:403,msg:'Wfuzz detected'"

# Detection by request pattern
# Many requests to different paths in short time = directory brute-force
# Requests with common parameter names = parameter fuzzing
```

**Network Signatures:**
```
# Suricata rule for detecting web fuzzing
alert http any any -> $HOME_NET any (msg:"Potential Web Directory Fuzzing"; \
  threshold:type both, track by_src, count 50, seconds 10; \
  sid:1000020; rev:1;)
```

### Blue Team Defensive Measures

**1. Rate Limiting:**
```nginx
# Nginx rate limiting
limit_req_zone $binary_remote_addr zone=fuzz:10m rate=10r/s;

server {
    location / {
        limit_req zone=fuzz burst=20 nodelay;
    }
}
```

**2. WAF Rules:**
```apache
# Apache ModSecurity rules
SecRule REQUEST_URI "@detectSQLi" "id:1001,phase:1,deny,status:403"
SecRule REQUEST_URI "@detectXSS" "id:1002,phase:1,deny,status:403"

# Block common fuzzing patterns
SecRule REQUEST_URI "@rx (admin|backup|config|debug|test)" \
    "id:1003,phase:1,deny,status:403,msg:'Fuzzing pattern detected'"
```

**3. Honeypot Endpoints:**
```bash
# Create fake admin panels that trigger alerts
# Add to web server config:
# location /admin-panel-xyz123 {
#     return 200 "Nothing here";
#     access_log /var/log/honeypot.log;
# }
```

**4. User-Agent Filtering:**
```nginx
# Block known fuzzing tool User-Agents
if ($http_user_agent ~* (wfuzz|gobuster|dirbuster|nikto|nmap)) {
    return 403;
}
```

**5. Log Monitoring:**
```bash
#!/bin/bash
# monitor_fuzzing.sh — Detect web fuzzing in real-time

LOG="/var/log/nginx/access.log"

tail -f "$LOG" | while read line; do
    IP=$(echo "$line" | awk '{print $1}')
    STATUS=$(echo "$line" | awk '{print $9}')
    
    # Count 404s per IP
    if [ "$STATUS" = "404" ]; then
        COUNT=$(grep "^$IP" "$LOG" | grep " 404 " | wc -l)
        if [ "$COUNT" -gt 50 ]; then
            echo "ALERT: Potential fuzzing from $IP ($COUNT 404s)"
            logger -p security.warning "Web fuzzing detected from $IP"
        fi
    fi
done
```

---

## Chapter 8: Troubleshooting

### Common Errors

#### "Too many results"

**Cause:** Word count filter not set correctly, or default "not found" page not filtered.

**Solutions:**
```bash
# First, find the default page's word count
wfuzz -c -z file,wordlist.txt http://target.com/FUZZ
# Look at 404 responses and note word/line/char counts

# Then filter by those values
wfuzz -c -z file,wordlist.txt --hw 156 --hc 404 http://target.com/FUZZ

# Or use --hh for character count
wfuzz -c -z file,wordlist.txt --hh 5432 http://target.com/FUZZ
```

#### "Connection timeout"

**Cause:** Target is slow or blocking requests.

**Solutions:**
```bash
# Reduce threads
wfuzz -c -z file,wordlist.txt -t 5 http://target.com/FUZZ

# Add delay
wfuzz -c -z file,wordlist.txt -s 1 http://target.com/FUZZ

# Check target availability
curl -I http://target.com
```

#### "Memory error" or "MemoryError"

**Cause:** Wordlist too large for available memory.

**Solutions:**
```bash
# Use smaller wordlist
wc -l wordlist.txt  # Check size
split -l 10000 wordlist.txt chunk_  # Split large files

# Process in batches
for chunk in chunk_*; do
    wfuzz -c -z file,$chunk http://target.com/FUZZ >> results.txt
done
```

#### "SSL error" or "certificate verify failed"

**Cause:** SSL certificate issues with target.

**Solutions:**
```bash
# Disable SSL verification
wfuzz -c -z file,wordlist.txt --no-check-certificates https://target.com/FUZZ

# Or use http instead of https
wfuzz -c -z file,wordlist.txt http://target.com/FUZZ
```

#### "Connection refused"

**Cause:** Target server is down or blocking connections.

**Solutions:**
```bash
# Verify target is accessible
curl -I http://target.com

# Check if using wrong port
wfuzz -c -z file,wordlist.txt http://target.com:8080/FUZZ

# Try with different method
wfuzz -c -z file,wordlist.txt -X HEAD http://target.com/FUZZ
```

### Performance Optimization

```bash
# Optimal thread count depends on target
# Start low and increase:
wfuzz -t 10 ...   # Conservative
wfuzz -t 30 ...   # Moderate
wfuzz -t 100 ...  # Aggressive (may trigger WAF)

# Use proxy for distributed fuzzing
wfuzz -p http://proxy1:8080 ...  # Proxy 1
wfuzz -p http://proxy2:8080 ...  # Proxy 2
```

### FAQ

**Q: Wfuzz vs ffuf — which is better?**
A: ffuf is generally faster and more actively maintained. Wfuzz has more advanced payload features (encoders, scripts) and is better for complex fuzzing scenarios.

**Q: Can Wfuzz handle HTTPS?**
A: Yes, but you may need to disable certificate verification for self-signed certificates.

**Q: How do I find the right filter values?**
A: Run without filters first, identify the "not found" page's word/line/char count, then filter by those values.

**Q: Can Wfuzz fuzz JSON APIs?**
A: Yes, use `-H "Content-Type: application/json"` and `-d '{"key":"FUZZ"}'`.

**Q: How do I use Wfuzz with Tor?**
A: Use `-p socks5://127.0.0.1:9050` after starting Tor service.

**Q: Can Wfuzz handle POST data?**
A: Yes, use `-d "param=FUZZ"` for POST data, `-X POST` to set the method.

---

## Resources

- [Wfuzz GitHub Repository](https://github.com/xmendez/wfuzz)
- [Wfuzz Documentation](https://wfuzz.readthedocs.io)
- [Wfuzz Payload Documentation](https://wfuzz.readthedocs.io/en/latest/payloads.html)
- [Wfuzz Encoder Documentation](https://wfuzz.readthedocs.io/en/latest/encoders.html)
- [OWASP Testing Guide — Fuzzing](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/07-Input_Validation_Testing/05-Testing_for_HTTP_Fuzzing)
- [SecLists Wordlists](https://github.com/danielmiessler/SecLists)
- [FuzzDB — Fuzzing Payloads](https://github.com/fuzzdb-project/fuzzdb)
- [Kali Linux Wfuzz Package](https://www.kali.org/tools/wfuzz/)
- [HTTP Fuzzing Techniques — PortSwigger](https://portswigger.net/web-security/fuzzing)
