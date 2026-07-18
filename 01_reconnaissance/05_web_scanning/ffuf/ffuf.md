---
tool_name: "ffuf"
display_name: "FFuF"
mitre_tactic: "reconnaissance"
mitre_tactic_id: "TA0043"
mitre_subcategory: "web_scanning"
install_command: "sudo apt install ffuf"
version: "2.1.0"
github_repo: "https://github.com/ffuf/ffuf"
kali_tools_page: "https://www.kali.org/tools/ffuf/"
difficulty_level: "beginner-to-advanced"
requires_root: false
gui_available: false
tags: ["fuzzing", "directory", "web", "brute-force", "fast", "parameter-fuzzing", "vhost"]
related_tools: ["gobuster", "wfuzz", "dirsearch", "dirb", "feroxbuster"]
---

# FFuF — Fast Web Fuzzer

## Chapter 1: Introduction & Overview

### What is FFuF?
FFuF (Fuzz Faster U Fool) is a fast, flexible, and open-source web fuzzing tool written in Go. It is designed for web application security testing, including directory and file discovery, virtual host enumeration, parameter fuzzing, and custom payload injection. FFuF achieves exceptional speed through its asynchronous architecture and efficient request handling, making it one of the fastest web fuzzers available today.

### History & Background
- **Created:** 2019 by Jooni Väyrynen (github.com/ffuf)
- **Language:** Go (Golang)
- **License:** MIT License
- **Current Version:** 2.1.0 (as of 2024)
- **Purpose:** Web fuzzing, directory discovery, parameter testing, vhost enumeration

FFuF was created as a modern alternative to tools like WFuzz and DirBuster, focusing on speed and simplicity. It quickly became one of the most popular web fuzzing tools in the Kali Linux toolkit due to its performance and ease of use.

### When to Use This Tool
- **Directory and file discovery:** Find hidden directories, files, and endpoints on web servers
- **Parameter fuzzing:** Test GET/POST parameters for injection points, hidden values, or vulnerabilities
- **Virtual host enumeration:** Discover virtual hosts hosted on a web server
- **Subdomain discovery:** Brute-force subdomains using DNS resolution
- **API endpoint discovery:** Find undocumented or hidden API endpoints
- **Content discovery:** Uncover backup files, configuration files, and sensitive data
- **Authentication brute-forcing:** Test login forms with credential lists
- **Web application fingerprinting:** Identify technologies and versions through response analysis

### When NOT to Use This Tool
- Without explicit written authorization from the target owner
- On production systems without prior coordination and testing windows
- Against systems you do not own or have permission to test
- When the target has strict rate limiting that could trigger account lockouts
- For denial-of-service testing (FFuF can generate enormous traffic volumes)

### Legal and Ethical Considerations
- **Always obtain written authorization** before testing any system you do not own
- **Define scope clearly** — know exactly which hosts, ports, and paths are in scope
- **Use rate limiting** to avoid overwhelming targets or causing service degradation
- **Document everything** — keep records of authorization, scope, and findings
- **Follow responsible disclosure** — report vulnerabilities through proper channels
- **Understand local laws** — unauthorized computer access is illegal in most jurisdictions
- **Bug bounty programs** — only test within the defined rules of engagement

### Key Concepts
- **Fuzzing:** Sending a large number of requests with varying payloads to discover hidden content or vulnerabilities
- **Wordlist:** A file containing potential paths, parameters, or values to test
- **Filter:** Rules to exclude responses based on status code, size, words, or lines
- **Matcher:** Rules to include only responses matching specific criteria
- **Recursion:** Automatically discover and fuzz subdirectories found during scanning
- **Rate Limiting:** Controlling request speed to avoid detection or service disruption
- **Virtual Host (VHost):** Multiple websites hosted on the same IP address, differentiated by the Host header
- **Injection Point:** The position in a URL or request where payloads are inserted (FUZZ keyword)

---

## Chapter 2: Installation & Setup

### System Requirements
- **Operating System:** Linux, macOS, Windows, or any Go-supported platform
- **RAM:** Minimum 256 MB (512 MB recommended for large wordlists)
- **Disk Space:** 50 MB for installation + space for wordlists and output
- **CPU:** Multi-core recommended for maximum performance
- **Network:** Stable internet connection for remote targets
- **Go:** Required for building from source (Go 1.18+)

### Installation Methods

#### Method 1: APT (Recommended for Kali Linux)
```bash
sudo apt update
sudo apt install ffuf
```

#### Method 2: Go Install
```bash
go install github.com/ffuf/ffuf/v2@latest
```

#### Method 3: From Source
```bash
git clone https://github.com/ffuf/ffuf.git
cd ffuf
go build -o ffuf
sudo cp ffuf /usr/local/bin/
```

#### Method 4: Docker
```bash
docker run -it --rm ffuf/ffuf -u http://target -w /path/to/wordlist
```

#### Method 5: Download Release Binary
```bash
# Visit https://github.com/ffuf/ffuf/releases
# Download the appropriate binary for your platform
chmod +x ffuf
sudo mv ffuf /usr/local/bin/
```

### Configuration
FFuF does not require a configuration file for basic operation. All options are passed via command-line arguments. For persistent configurations, consider using shell aliases or wrapper scripts.

### Verification
```bash
ffuf -V
# Output: ffuf version 2.1.0
```

### Updating
```bash
# APT
sudo apt update && sudo apt install --only-upgrade ffuf

# Go
go install github.com/ffuf/ffuf/v2@latest
```

---

## Chapter 3: Basic Usage

### First Run
```bash
ffuf -u http://target.example.com/FUZZ -w /usr/share/wordlists/dirb/common.txt
```
**Output:**
```
        /'___\  /'___\           /'___\
       /\ \__/ /\ \__/  __  __  /\ \__/
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/
         \ \_\   \ \_\  \ \____/  \ \_\
          \/_/    \/_/   \/___/    \/_/

       v2.1.0
________________________________________________

 :: Method           : GET
 :: URL              : http://target.example.com/FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/dirb/common.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,303,307,401,403,405
________________________________________________

admin                   [Status: 301, Size: 312, Words: 20, Lines: 10]
backup                  [Status: 403, Size: 278, Words: 18, Lines: 8]
config                  [Status: 403, Size: 278, Words: 18, Lines: 8]
login                   [Status: 200, Size: 4521, Words: 312, Lines: 89]
robots.txt              [Status: 200, Size: 125, Words: 8, Lines: 6]
server-status           [Status: 403, Size: 278, Words: 18, Lines: 8]
```

### Essential Commands

#### Basic Directory Discovery
```bash
ffuf -u http://target.example.com/FUZZ -w /usr/share/wordlists/dirb/common.txt
```
**Explanation:** Fuzzes the target URL replacing FUZZ with words from the common wordlist.

#### Directory Discovery with Extensions
```bash
ffuf -u http://target.example.com/FUZZ -w /usr/share/wordlists/dirb/common.txt -e .php,.html,.txt
```
**Explanation:** Appends .php, .html, and .txt extensions to each word in the wordlist.

#### Filter by Status Code
```bash
ffuf -u http://target.example.com/FUZZ -w /usr/share/wordlists/dirb/common.txt -fc 404
```
**Explanation:** Filters out responses with 404 status code from results.

#### Filter by Response Size
```bash
ffuf -u http://target.example.com/FUZZ -w /usr/share/wordlists/dirb/common.txt -fs 4242
```
**Explanation:** Filters out responses with exactly 4242 bytes (common default page size).

#### Parameter Fuzzing
```bash
ffuf -u "http://target.example.com/page?id=FUZZ" -w /usr/share/wordlists/seclists/Fuzzing/dentions.txt
```
**Explanation:** Fuzzes the id parameter with values from the wordlist.

#### POST Data Fuzzing
```bash
ffuf -u http://target.example.com/login -X POST -d "user=admin&pass=FUZZ" -w /usr/share/wordlists/rockyou.txt -fc 401
```
**Explanation:** Brute-forces the password field using POST request.

#### Virtual Host Discovery
```bash
ffuf -u http://192.168.1.1 -H "Host: FUZZ.target.example.com" -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt -fs 0
```
**Explanation:** Discovers virtual hosts by fuzzing the Host header.

#### Recursive Directory Discovery
```bash
ffuf -u http://target.example.com/FUZZ -w /usr/share/wordlists/dirb/common.txt -recursion -recursion-depth 2
```
**Explanation:** Automatically fuzzes discovered subdirectories up to 2 levels deep.

#### Multiple Wordlists
```bash
ffuf -u http://target.example.com/FUZZ -w /usr/share/wordlists/dirb/common.txt:/usr/share/wordlists/dirb/big.txt
```
**Explanation:** Uses multiple wordlists separated by colons.

#### Save Output to File
```bash
ffuf -u http://target.example.com/FUZZ -w /usr/share/wordlists/dirb/common.txt -o results.json -of json
```
**Explanation:** Saves results in JSON format to results.json.

#### Verbose Output
```bash
ffuf -u http://target.example.com/FUZZ -w /usr/share/wordlists/dirb/common.txt -v
```
**Explanation:** Shows full request and response information for each finding.

#### Custom Headers
```bash
ffuf -u http://target.example.com/FUZZ -w /usr/share/wordlists/dirb/common.txt -H "Authorization: Bearer token123" -H "X-Custom: value"
```
**Explanation:** Adds custom headers to every request.

#### Rate Limiting
```bash
ffuf -u http://target.example.com/FUZZ -w /usr/share/wordlists/dirb/common.txt -rate 100
```
**Explanation:** Limits requests to 100 per second to avoid detection.

#### Thread Control
```bash
ffuf -u http://target.example.com/FUZZ -w /usr/share/wordlists/dirb/common.txt -t 10
```
**Explanation:** Uses 10 concurrent threads instead of the default 40.

### Complete Flags Reference

#### Input Options
| Flag | Description | Example |
|------|-------------|---------|
| `-w` | Wordlist path(s) | `-w /path/to/wordlist.txt` |
| `-recursion` | Enable recursion | `-recursion` |
| `-recursion-depth` | Max recursion depth | `-recursion-depth 2` |
| `-e` | Extensions to append | `-e .php,.html,.txt` |
| `-enc` | Encode payloads | `-enc url,html,base64` |
| `-request` | Request file | `-request /path/to/request.txt` |
| `-request-proto` | Request protocol | `-request-proto https` |
| `-ac` | Auto-calibrate | `-ac` |
| `-ac-url` | Auto-calibrate URL | `-ac-url http://target/404` |
| `-ic` | Ignore comments | `-ic` |
| `-mode` | Operation mode | `-mode sniper` |

#### Target Options
| Flag | Description | Example |
|------|-------------|---------|
| `-u` | Target URL | `-u http://target/FUZZ` |
| `-H` | Custom header | `-H "Host: FUZZ.target.com"` |
| `-X` | HTTP method | `-X POST` |
| `-d` | POST data | `-d "user=admin&pass=FUZZ"` |
| `-b` | Cookie data | `-b "session=abc123"` |
| `-r` | Follow redirects | `-r` |
| `-auth` | Basic auth | `-auth user:pass` |
| `-timeout` | Request timeout | `-timeout 10` |
| `-json` | JSON output | `-json` |
| `-timeout` | Request timeout (seconds) | `-timeout 30` |

#### General Options
| Flag | Description | Example |
|------|-------------|---------|
| `-rate` | Requests per second | `-rate 100` |
| `-t` | Number of threads | `-t 10` |
| `-p` | Delay between requests | `-p 1` |
| `-c` | Colorize output | `-c` |
| `-v` | Verbose output | `-v` |
| `-s` | Silent mode | `-s` |
| `-S` | Delay per thread | `-S 1` |
| `-se` | Stop on spool error | `-se` |
| `-sf` | Stop when >95% matched | `-sf` |
| `-p` | Delay in seconds between each thread | `-p 0.1` |
| `-timeout` | HTTP request timeout in seconds | `-timeout 10` |

#### Matcher Options
| Flag | Description | Example |
|------|-------------|---------|
| `-mc` | Match HTTP status codes | `-mc 200,301,302` |
| `-ms` | Match response size | `-ms 1024` |
| `-mw` | Match word count | `-mw 100` |
| `-ml` | Match line count | `-ml 50` |
| `-mr` | Match regex | `-mr "flag\{.*\}"` |
| `-mt` | Match time (ms) | `-mt 1000` |

#### Filter Options
| Flag | Description | Example |
|------|-------------|---------|
| `-fc` | Filter HTTP status codes | `-fc 404,403` |
| `-fs` | Filter response size | `-fs 0` |
| `-fw` | Filter word count | `-fw 200` |
| `-fl` | Filter line count | `-fl 5` |
| `-fr` | Filter regex | `-fr "error"` |
| `-ft` | Filter time (ms) | `-ft 200` |

#### Output Options
| Flag | Description | Example |
|------|-------------|---------|
| `-o` | Output file | `-o results.json` |
| `-of` | Output format | `-of json` |
| `-od` | Output directory | `-od /tmp/ffuf/` |
| `-or` | Output to stdout | `-or` |

#### Supported Output Formats
| Format | Description |
|--------|-------------|
| `json` | JSON format |
| `ejson` | Enhanced JSON with full request/response |
| `html` | HTML report |
| `md` | Markdown report |
| `csv` | CSV format |
| `all` | All formats at once |

---

## Chapter 4: Intermediate Usage

### Bash Scripting and Automation

#### Automated Directory Discovery Script
```bash
#!/bin/bash
# auto_ffuf.sh - Automated directory discovery

TARGET=$1
WORDLIST=$2
OUTPUT_DIR="ffuf_$(date +%Y%m%d_%H%M%S)"

mkdir -p "$OUTPUT_DIR"

echo "[*] Starting directory discovery on $TARGET"
ffuf -u "$TARGET/FUZZ" \
     -w "$WORDLIST" \
     -mc 200,301,302,303,307,401,403,405 \
     -fc 404 \
     -o "$OUTPUT_DIR/directories.json" \
     -of json \
     -s

echo "[*] Results saved to $OUTPUT_DIR"
```

#### Multi-Target Scanning Script
```bash
#!/bin/bash
# multi_target_ffuf.sh

TARGETS_FILE=$1
WORDLIST=$2

while IFS= read -r target; do
    echo "[*] Scanning: $target"
    ffuf -u "$target/FUZZ" \
         -w "$WORDLIST" \
         -fc 404 \
         -o "ffuf_${target//\//_}.json" \
         -of json \
         -rate 50 \
         -t 20
done < "$TARGETS_FILE"
```

#### Filter Out Default Pages
```bash
#!/bin/bash
# Smart filtering - auto-calibrate against default page
TARGET=$1

# First request to get default page size
DEFAULT_SIZE=$(ffuf -u "$TARGET/FUZZ" -w /usr/share/wordlists/dirb/common.txt -mc 200 -s -o /dev/stdout -of json 2>/dev/null | jq '.results[0].length // 0')

# Now scan with filtering
ffuf -u "$TARGET/FUZZ" \
     -w /usr/share/wordlists/dirb/common.txt \
     -fs "$DEFAULT_SIZE" \
     -fc 404
```

### Python Scripting

#### Python Wrapper for FFuF
```python
#!/usr/bin/env python3
"""Python wrapper for FFuF scanning."""

import subprocess
import json
import sys

def run_ffuf(target, wordlist, extensions=None, filters=None):
    """
    Run FFuF scan and return results.
    
    Args:
        target: Target URL with FUZZ keyword
        wordlist: Path to wordlist file
        extensions: List of extensions to append
        filters: Dict of filter options
    
    Returns:
        List of discovered paths
    """
    cmd = [
        'ffuf',
        '-u', target,
        '-w', wordlist,
        '-o', '/dev/stdout',
        '-of', 'json',
        '-s'  # Silent mode
    ]
    
    if extensions:
        cmd.extend(['-e', ','.join(extensions)])
    
    if filters:
        if 'status' in filters:
            cmd.extend(['-fc', filters['status']])
        if 'size' in filters:
            cmd.extend(['-fs', str(filters['size'])])
    
    try:
        result = subprocess.run(cmd, capture_output=True, text=True, timeout=300)
        data = json.loads(result.stdout)
        return data.get('results', [])
    except subprocess.TimeoutExpired:
        print(f"[-] Scan timed out for {target}")
        return []
    except json.JSONDecodeError:
        print(f"[-] Failed to parse FFuF output")
        return []

def main():
    if len(sys.argv) < 3:
        print(f"Usage: {sys.argv[0]} <target_url> <wordlist>")
        sys.exit(1)
    
    target = sys.argv[1]
    wordlist = sys.argv[2]
    
    # Run scan
    results = run_ffuf(
        target=target,
        wordlist=wordlist,
        extensions=['.php', '.html', '.txt'],
        filters={'status': '404'}
    )
    
    # Print results
    print(f"\n[+] Found {len(results)} results:")
    for r in results:
        print(f"  {r['input']['FUZZ']} - Status: {r['status']} Size: {r['length']}")

if __name__ == '__main__':
    main()
```

#### Processing FFuF JSON Output
```python
#!/usr/bin/env python3
"""Process FFuF JSON output for analysis."""

import json
import sys

def analyze_ffuf_results(json_file):
    """Analyze FFuF results and generate summary."""
    with open(json_file, 'r') as f:
        data = json.load(f)
    
    results = data.get('results', [])
    
    # Group by status code
    status_groups = {}
    for r in results:
        status = r['status']
        if status not in status_groups:
            status_groups[status] = []
        status_groups[status].append(r)
    
    print(f"[*] Total results: {len(results)}")
    print(f"[*] Status code breakdown:")
    for status, items in sorted(status_groups.items()):
        print(f"    {status}: {len(items)} results")
    
    # Find interesting results
    interesting = [r for r in results if r['status'] in [200, 301, 302]]
    if interesting:
        print(f"\n[+] Interesting findings:")
        for r in interesting:
            print(f"    {r['input']['FUZZ']} - {r['status']} ({r['length']} bytes)")

if __name__ == '__main__':
    if len(sys.argv) < 2:
        print(f"Usage: {sys.argv[0]} <ffuf_results.json>")
        sys.exit(1)
    analyze_ffuf_results(sys.argv[1])
```

### Tool Chaining

#### FFuF with Gobuster
```bash
# Phase 1: Quick scan with Gobuster
gobuster dir -u http://target.example.com -w /usr/share/wordlists/dirb/common.txt -o quick_results.txt

# Phase 2: Detailed fuzzing with FFuF based on Gobuster results
ffuf -u http://target.example.com/FUZZ -w discovered_paths.txt -mc 200,301,302
```

#### FFuF with Nmap
```bash
# First, discover services
nmap -sV -p 80,443,8080,8443 target.example.com

# Then fuzz each discovered service
ffuf -u http://target.example.com:8080/FUZZ -w /usr/share/wordlists/dirb/common.txt
```

#### FFuF with Sublist3r
```bash
# Find subdomains
sublist3r -d target.example.com -o subdomains.txt

# Fuzz each subdomain
while read subdomain; do
    ffuf -u "http://$subdomain/FUZZ" -w /usr/share/wordlists/dirb/common.txt -fc 404 -o "ffuf_$subdomain.json" -of json -s
done < subdomains.txt
```

#### FFuF with Nuclei
```bash
# Discover directories
ffuf -u http://target.example.com/FUZZ -w /usr/share/wordlists/dirb/common.txt -fc 404 -o directories.txt

# Scan discovered paths with Nuclei
nuclei -l directories.txt -t cves/
```

### Output Processing and Filtering

#### Extracting Only 200 Status Codes
```bash
ffuf -u http://target.example.com/FUZZ -w wordlist.txt -mc 200 | grep "Status: 200"
```

#### Converting JSON to CSV
```bash
# Using jq
ffuf -u http://target.example.com/FUZZ -w wordlist.txt -o results.json -of json
cat results.json | jq -r '.results[] | [.input.FUZZ, .status, .length, .words] | @csv' > results.csv
```

#### Sorting Results by Size
```bash
ffuf -u http://target.example.com/FUZZ -w wordlist.txt -o results.json -of json
cat results.json | jq '.results | sort_by(.length) | reverse'
```

---

## Chapter 5: Advanced Usage

### Custom Wordlists

#### Creating Targeted Wordlists
```bash
# Combine multiple wordlists
cat /usr/share/wordlists/dirb/common.txt \
    /usr/share/wordlists/dirb/big.txt \
    /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt > combined.txt

# Remove duplicates
sort -u combined.txt > unique_wordlist.txt
```

#### Custom Wordlist Generation
```bash
# Generate wordlist from target's robots.txt
curl -s http://target.example.com/robots.txt | grep -E "^Disallow:" | cut -d' ' -f2 > custom_wordlist.txt

# Generate wordlist from target's sitemap
curl -s http://target.example.com/sitemap.xml | grep -oP '(?<=<loc>).*?(?=</loc>)' > sitemap_wordlist.txt
```

#### Password and Username Lists
```bash
# For brute-force testing
ffuf -u http://target.example.com/login \
     -X POST \
     -d "username=admin&password=FUZZ" \
     -w /usr/share/wordlists/rockyou.txt \
     -fc 401

# Common usernames
echo -e "admin\nadministrator\nroot\nuser\ntest\nbackup" > usernames.txt
```

### Evasion and Rate Limiting

#### Request Throttling
```bash
# Slow scan to avoid detection
ffuf -u http://target.example.com/FUZZ \
     -w wordlist.txt \
     -rate 10 \
     -p 0.5 \
     -t 5
```

#### Using Proxies
```bash
# Route through proxy
ffuf -u http://target.example.com/FUZZ \
     -w wordlist.txt \
     -x http://proxy:8080
```

#### Random User-Agent Rotation
```bash
# Using proxychains with random user agents
proxychains ffuf -u http://target.example.com/FUZZ -w wordlist.txt
```

#### Request Delay Between Threads
```bash
ffuf -u http://target.example.com/FUZZ \
     -w wordlist.txt \
     -S \
     -p 2
```

### Integration with Pentest Workflows

#### Full Web Assessment Pipeline
```bash
#!/bin/bash
# full_web_assessment.sh

TARGET=$1

echo "[+] Phase 1: Subdomain discovery"
subfinder -d "$TARGET" -o subdomains.txt

echo "[+] Phase 2: Port scanning"
nmap -iL subdomains.txt -p 80,443,8080,8443 --open -oN ports.txt

echo "[+] Phase 3: Directory discovery"
while read host; do
    ffuf -u "http://$host/FUZZ" \
         -w /usr/share/wordlists/dirb/big.txt \
         -fc 404 \
         -o "ffuf_$host.json" \
         -of json \
         -rate 100 \
         -s
done < subdomains.txt

echo "[+] Phase 4: Parameter fuzzing"
# Use discovered endpoints for parameter fuzzing
find . -name "ffuf_*.json" -exec jq -r '.results[].input.FUZZ' {} \; | \
    while read path; do
        ffuf -u "http://$TARGET/$path?param=FUZZ" \
             -w /usr/share/wordlists/seclists/Fuzzing/special-chars.txt \
             -fc 404 \
             -o "param_fuzz_$path.json" \
             -of json
    done
```

#### Bug Bounty Workflow
```bash
#!/bin/bash
# bug_bounty_ffuf.sh

DOMAIN=$1

echo "[+] Starting bug bounty reconnaissance for $DOMAIN"

# Subdomain enumeration
echo "[*] Phase 1: Subdomain discovery"
amass enum -passive -d "$DOMAIN" -o amass_subs.txt
subfinder -d "$DOMAIN" -o subfinder_subs.txt
cat amass_subs.txt subfinder_subs.txt | sort -u > all_subs.txt

# Probe for live hosts
echo "[*] Phase 2: Probing live hosts"
httpx -l all_subs.txt -o live_hosts.txt -silent

# Directory discovery
echo "[*] Phase 3: Directory discovery"
while read host; do
    ffuf -u "http://$host/FUZZ" \
         -w /usr/share/wordlists/dirb/common.txt \
         -fc 404 \
         -o "ffuf_${host}.json" \
         -of json \
         -rate 50 \
         -s &
done < live_hosts.txt
wait

echo "[+] Reconnaissance complete"
```

### Custom Extensions and Payloads

#### SQL Injection Testing
```bash
ffuf -u "http://target.example.com/page?id=FUZZ" \
     -w /usr/share/wordlists/seclists/Fuzzing/SQL/SQLite_Special.txt \
     -fc 404 \
     -mc 200,500
```

#### XSS Payload Testing
```bash
ffuf -u "http://target.example.com/search?q=FUZZ" \
     -w /usr/share/wordlists/seclists/Fuzzing/XSS/XSS-BruteViolin.txt \
     -fc 404 \
     -mc 200
```

#### Path Traversal Testing
```bash
ffuf -u "http://target.example.com/download?file=FUZZ" \
     -w /usr/share/wordlists/seclists/Fuzzing/Traversal/../../etc/passwd.txt \
     -fc 404 \
     -mr "root:x:0:0"
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: Web Application Audit

**Objective:** Perform comprehensive directory and parameter discovery on a corporate web application.

**Step 1: Initial Reconnaissance**
```bash
# Discover subdomains
subfinder -d corporate.example.com -o subdomains.txt

# Check for live hosts
httpx -l subdomains.txt -o live_hosts.txt -silent
```

**Step 2: Directory Discovery**
```bash
# Quick scan with common wordlist
ffuf -u http://app.corporate.example.com/FUZZ \
     -w /usr/share/wordlists/dirb/common.txt \
     -fc 404 \
     -o quick_scan.json \
     -of json \
     -rate 100

# Deep scan with extended wordlist
ffuf -u http://app.corporate.example.com/FUZZ \
     -w /usr/share/wordlists/dirb/big.txt \
     -fc 404 \
     -e .php,.html,.js,.txt,.bak,.old,.config \
     -o deep_scan.json \
     -of json \
     -rate 100
```

**Step 3: Parameter Discovery**
```bash
# Fuzz discovered endpoints for parameters
ffuf -u "http://app.corporate.example.com/api/endpoint?FUZZ=test" \
     -w /usr/share/wordlists/seclists/Discovery/Web-Content/burp-parameter-names.txt \
     -fc 404 \
     -mc 200 \
     -o param_discovery.json \
     -of json
```

**Step 4: Analysis and Reporting**
```bash
# Extract and analyze results
cat quick_scan.json deep_scan.json param_discovery.json | \
    jq '.results[] | {path: .input.FUZZ, status: .status, size: .length}' | \
    sort -u > findings.json
```

### Scenario 2: CTF Challenge

**Objective:** Find hidden flags in a CTF web challenge.

**Step 1: Initial Discovery**
```bash
# Basic directory scan
ffuf -u http://ctf.example.com/FUZZ \
     -w /usr/share/wordlists/dirb/common.txt

# Check robots.txt and sitemap
curl -s http://ctf.example.com/robots.txt
curl -s http://ctf.example.com/sitemap.xml
```

**Step 2: Deep Content Discovery**
```bash
# Scan with extensions
ffuf -u http://ctf.example.com/FUZZ \
     -w /usr/share/wordlists/dirb/common.txt \
     -e .php,.html,.txt,.bak,.old,.swp,.git

# Look for backup files
ffuf -u http://ctf.example.com/FUZZ \
     -w /usr/share/wordlists/seclists/Discovery/Web-Content/backup.txt \
     -fc 404
```

**Step 3: Parameter Fuzzing**
```bash
# Fuzz parameters on discovered pages
ffuf -u "http://ctf.example.com/page?id=FUZZ" \
     -w /usr/share/wordlists/seclists/Fuzzing/dentions.txt \
     -mc 200 \
     -mr "flag\{.*\}"
```

**Step 4: Flag Extraction**
```bash
# Search for flag patterns
ffuf -u http://ctf.example.com/FUZZ \
     -w /usr/share/wordlists/dirb/common.txt \
     -mc 200 \
     -mr "flag\{.*\}"
```

### Scenario 3: Bug Bounty

**Objective:** Comprehensive web reconnaissance for a bug bounty program.

**Step 1: Scope Discovery**
```bash
# Get all subdomains
amass enum -passive -d target.com -o amass.txt
subfinder -d target.com -o subfinder.txt
assetfinder --subs-only target.com > assetfinder.txt
cat amass.txt subfinder.txt assetfinder.txt | sort -u > all_subdomains.txt

# Check which are alive
httpx -l all_subdomains.txt -o alive.txt -silent
```

**Step 2: Technology Fingerprinting**
```bash
# Identify technologies
whatweb -i alive.txt > technologies.txt

# Check for common technologies
ffuf -u http://target.com/FUZZ \
     -w /usr/share/wordlists/dirb/common.txt \
     -fc 404
```

**Step 3: Endpoint Discovery**
```bash
# Find hidden endpoints
ffuf -u http://target.com/api/FUZZ \
     -w /usr/share/wordlists/seclists/Discovery/Web-Content/api/api-endpoints.txt \
     -fc 404 \
     -o api_endpoints.json \
     -of json

# Discover hidden directories
ffuf -u http://target.com/FUZZ \
     -w /usr/share/wordlists/dirb/big.txt \
     -fc 404 \
     -e .php,.html,.js \
     -o hidden_dirs.json \
     -of json
```

**Step 4: Parameter Discovery**
```bash
# Fuzz parameters
ffuf -u "http://target.com/page?FUZZ=1" \
     -w /usr/share/wordlists/seclists/Discovery/Web-Content/burp-parameter-names.txt \
     -fc 404 \
     -mc 200 \
     -o parameters.json \
     -of json
```

---

## Chapter 7: Detection & Defense

### IDS/IPS Detection

**Signature-Based Detection:**
- FFuF generates distinctive User-Agent strings by default
- Rapid sequential requests to similar paths trigger anomaly detection
- High-volume requests from single IP addresses create detectable patterns
- Specific request patterns (sequential fuzzing) are signature-matchable

**Behavioral Detection:**
- Unusual request patterns (sequential directory enumeration)
- High request rates from single source
- Requests to non-existent paths generating 404/403 responses
- Time-based patterns (consistent intervals between requests)

### Log Indicators

**Apache/Nginx Log Patterns:**
```
# Sequential path requests
[18/Jul/2024:10:15:23 +0000] "GET /admin HTTP/1.1" 404
[18/Jul/2024:10:15:23 +0000] "GET /admin.php HTTP/1.1" 404
[18/Jul/2024:10:15:23 +0000] "GET /administrator HTTP/1.1" 404

# High request rate
[18/Jul/2024:10:15:23 +0000] "GET /backup HTTP/1.1" 403
[18/Jul/2024:10:15:23 +0000] "GET /backup.zip HTTP/1.1" 404
[18/Jul/2024:10:15:23 +0000] "GET /backup.tar.gz HTTP/1.1" 404
```

**WAF Log Indicators:**
- Multiple 403/404 responses in rapid succession
- Requests to common fuzzing paths (/admin, /backup, /config)
- Unusual User-Agent strings
- High request volume from single IP

### Mitigation Strategies

**Rate Limiting:**
```nginx
# Nginx rate limiting
limit_req_zone $binary_remote_addr zone=one:10m rate=10r/s;

server {
    location / {
        limit_req zone=one burst=20 nodelay;
    }
}
```

**Fail2Ban Integration:**
```ini
# /etc/fail2ban/filter.d/ffuf.conf
[Definition]
failregex = ^<HOST> -.*"(GET|POST) /(admin|backup|config).*" (404|403)$
ignoreregex =
```

```ini
# /etc/fail2ban/jail.d/ffuf.conf
[ffuf]
enabled = true
filter = ffuf
logpath = /var/log/apache2/access.log
maxretry = 10
findtime = 60
bantime = 3600
```

**Web Application Firewall Rules:**
```apache
# ModSecurity rule to detect FFuF
SecRule REQUEST_URI "@contains /admin" \
    "id:10001,phase:1,deny,status:403,log,msg:'Potential directory fuzzing detected'"
```

### Blue Team Response

**Incident Response Steps:**
1. **Identify:** Analyze logs for fuzzing patterns
2. **Contain:** Implement rate limiting or block IPs
3. **Eradicate:** Review exposed sensitive files
4. **Recover:** Secure or remove exposed content
5. **Lessons Learned:** Implement preventive controls

**Monitoring Commands:**
```bash
# Watch for fuzzing in real-time
tail -f /var/log/apache2/access.log | grep -E "(admin|backup|config|\.bak)"

# Check for high request rates
awk '{print $1}' /var/log/apache2/access.log | sort | uniq -c | sort -rn | head -20
```

---

## Chapter 8: Troubleshooting & Resources

### Common Errors

**Error: "No output"**
```bash
# Solution: Check if target is reachable
curl -v http://target.example.com

# Check if wordlist exists
ls -la /usr/share/wordlists/dirb/common.txt

# Try with verbose mode
ffuf -u http://target.example.com/FUZZ -w wordlist.txt -v
```

**Error: "Connection refused"**
```bash
# Solution: Verify target is running
nmap -p 80,443 target.example.com

# Check if port is different
ffuf -u http://target.example.com:8080/FUZZ -w wordlist.txt
```

**Error: "Too many open files"**
```bash
# Solution: Increase file descriptor limit
ulimit -n 65535

# Or reduce threads
ffuf -u http://target.example.com/FUZZ -w wordlist.txt -t 10
```

**Error: "Connection timeout"**
```bash
# Solution: Increase timeout
ffuf -u http://target.example.com/FUZZ -w wordlist.txt -timeout 30

# Check network connectivity
ping target.example.com
```

### Performance Optimization

**Maximizing Speed:**
```bash
# Use multiple threads
ffuf -u http://target.example.com/FUZZ -w wordlist.txt -t 100

# Disable recursion if not needed
ffuf -u http://target.example.com/FUZZ -w wordlist.txt -recursion -recursion-depth 1

# Use efficient wordlists
# Smaller, focused wordlists are faster than large generic ones
```

**Reducing Memory Usage:**
```bash
# Process large wordlists in chunks
split -l 10000 big_wordlist.txt chunk_
for chunk in chunk_*; do
    ffuf -u http://target.example.com/FUZZ -w "$chunk" -fc 404
done
```

**Network Optimization:**
```bash
# Use connection pooling
ffuf -u http://target.example.com/FUZZ -w wordlist.txt -timeout 10

# Disable keep-alive for short scans
# (FFuF uses HTTP/1.1 by default)
```

### FAQ

**Q: How is FFuF different from WFuzz?**
A: FFuF is significantly faster due to its Go-based asynchronous architecture. WFuzz offers more advanced payload processing but is slower.

**Q: Can FFuF test POST parameters?**
A: Yes, use `-X POST -d "param=FUZZ"` to fuzz POST data.

**Q: How do I handle SSL certificates?**
A: FFuF follows redirects by default. For self-signed certs, use `-k` (if supported) or configure your system to trust the certificate.

**Q: Can FFuF fuzz subdomains?**
A: Yes, use the vhost mode: `ffuf -u http://IP -H "Host: FUZZ.target.com" -w subdomains.txt`

**Q: How do I avoid getting blocked?**
A: Use rate limiting (`-rate`), request delays (`-p`), and consider using proxies.

---

## Resources

### Official Documentation
- [FFuF GitHub Repository](https://github.com/ffuf/ffuf)
- [FFuF Wiki](https://github.com/ffuf/ffuf/wiki)
- [FFuF Usage Guide](https://github.com/ffuf/ffuf/blob/master/README.md)

### Recommended Wordlists
- [SecLists](https://github.com/danielmiessler/SecLists)
- [DirBuster Wordlists](https://sourceforge.net/projects/dirbuster/)
- [FuzzDB](https://github.com/fuzzdb-project/fuzzdb)
- [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings)

### Books
- "Web Application Hacker's Handbook" by Dafydd Stuttard
- "The Hacker Playbook 3" by Peter Kim
- "Penetration Testing" by Georgia Weidman

### Practice Labs
- [HackTheBox](https://www.hackthebox.com/)
- [TryHackMe](https://tryhackme.com/)
- [PortSwigger Web Security Academy](https://portswigger.net/web-security)
- [DVWA](https://github.com/digininja/DVWA)

### Video Tutorials
- [FFuF Crash Course](https://www.youtube.com/watch?v=example)
- [Web Fuzzing with FFuF](https://www.youtube.com/watch?v=example)
- [Bug Bounty Recon with FFuF](https://www.youtube.com/watch?v=example)

### Related Tools
- [Gobuster](https://github.com/OJ/gobuster) - Directory/DNS/VHost brute-forcer
- [WFuzz](https://github.com/xmendez/wfuzz) - Web application fuzzer
- [DirSearch](https://github.com/maurosoria/dirsearch) - Web path scanner
- [Feroxbuster](https://github.com/epi052/feroxbuster) - Fast, simple, recursive content discovery tool
