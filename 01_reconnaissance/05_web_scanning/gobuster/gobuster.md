---
tool_name: "gobuster"
display_name: "Gobuster"
mitre_tactic: "reconnaissance"
mitre_tactic_id: "TA0043"
mitre_subcategory: "web_scanning"
install_command: "sudo apt install gobuster"
version: "3.6"
github_repo: "https://github.com/OJ/gobuster"
kali_tools_page: "https://www.kali.org/tools/gobuster/"
difficulty_level: "beginner-to-advanced"
requires_root: false
gui_available: false
tags: ["directory", "dns", "vhost", "brute-force", "fuzzing", "web", "subdomain"]
related_tools: ["ffuf", "dirb", "dirsearch", "feroxbuster", "wfuzz"]
---

# Gobuster — Directory, DNS, and VHost Brute-Forcer

## Chapter 1: Introduction & Overview

### What is Gobuster?
Gobuster is a fast, flexible directory/file, DNS subdomain, and virtual host brute-forcing tool written in Go. It is used to discover hidden files, directories, subdomains, and virtual hosts on web servers by systematically guessing paths and names from wordlists. Gobuster is one of the most widely used web content discovery tools in the penetration testing and bug bounty communities.

### History & Background
- **Created:** 2016 by OJ Reeves (TheColonial)
- **Language:** Go (Golang)
- **License:** Apache License 2.0
- **Current Version:** 3.6 (as of 2024)
- **Purpose:** Directory discovery, DNS enumeration, virtual host discovery, S3 bucket discovery

Gobuster was created as a modern, fast alternative to tools like DirBuster and DirB. Written in Go, it offers excellent performance with minimal resource usage, making it ideal for large-scale reconnaissance operations. The tool supports multiple operation modes, making it a versatile choice for various discovery tasks.

### When to Use This Tool
- **Directory and file discovery:** Find hidden directories and files on web servers
- **DNS subdomain enumeration:** Discover subdomains through DNS brute-forcing
- **Virtual host discovery:** Find virtual hosts hosted on a web server
- **S3 bucket discovery:** Discover exposed Amazon S3 buckets
- **Web application reconnaissance:** Map out web application structure
- **Penetration testing:** Phase 1 reconnaissance for web applications
- **Bug bounty hunting:** Discover hidden endpoints and content

### When NOT to Use This Tool
- Without explicit written authorization from the target owner
- Against production systems without prior coordination
- When the target has strict rate limiting that could trigger account lockouts
- For denial-of-service testing (Gobuster can generate significant traffic)
- Against systems you do not own or have permission to test

### Legal and Ethical Considerations
- **Always obtain written authorization** before testing any system you do not own
- **Define scope clearly** — know exactly which hosts, ports, and paths are in scope
- **Use rate limiting** to avoid overwhelming targets or causing service degradation
- **Document everything** — keep records of authorization, scope, and findings
- **Follow responsible disclosure** — report vulnerabilities through proper channels
- **Understand local laws** — unauthorized computer access is illegal in most jurisdictions

### Key Concepts
- **Brute-Forcing:** Systematically guessing paths, names, or values from a wordlist
- **Wordlist:** A file containing potential paths, names, or values to test
- **Status Code:** HTTP response code indicating request result (200, 301, 403, 404, etc.)
- **Virtual Host (VHost):** Multiple websites hosted on the same IP, differentiated by Host header
- **DNS Brute-Forcing:** Guessing subdomain names through DNS queries
- **S3 Bucket:** Amazon Web Services cloud storage containers accessible via HTTP
- **Recursion:** Automatically discovering and fuzzing subdirectories

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
sudo apt install gobuster
```

#### Method 2: Go Install
```bash
go install github.com/OJ/gobuster/v3@latest
```

#### Method 3: From Source
```bash
git clone https://github.com/OJ/gobuster.git
cd gobuster
go build -o gobuster
sudo cp gobuster /usr/local/bin/
```

#### Method 4: Docker
```bash
docker run -it --rm oj/gobuster -h
```

#### Method 5: Download Release Binary
```bash
# Visit https://github.com/OJ/gobuster/releases
# Download the appropriate binary for your platform
chmod +x gobuster
sudo mv gobuster /usr/local/bin/
```

### Configuration
Gobuster does not require a configuration file. All options are passed via command-line arguments. For persistent configurations, consider using shell aliases or wrapper scripts.

### Verification
```bash
gobuster version
# Output: gobuster 3.6
```

### Updating
```bash
# APT
sudo apt update && sudo apt install --only-upgrade gobuster

# Go
go install github.com/OJ/gobuster/v3@latest
```

---

## Chapter 3: Basic Usage

### First Run
```bash
gobuster dir -u http://target.example.com -w /usr/share/wordlists/dirb/common.txt
```
**Output:**
```
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://target.example.com
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Status codes:            200,204,301,302,307,401,403
[+] User-Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/admin                 (Status: 301) [Size: 312] [--> /admin/]
/backup                (Status: 403) [Size: 278]
/config                (Status: 403) [Size: 278]
/index.html            (Status: 200) [Size: 4521]
/login                 (Status: 200) [Size: 4521]
/robots.txt            (Status: 200) [Size: 125]
/server-status         (Status: 403) [Size: 278]
===============================================================
Finished
===============================================================
```

### Essential Commands

#### Basic Directory Discovery
```bash
gobuster dir -u http://target.example.com -w /usr/share/wordlists/dirb/common.txt
```
**Explanation:** Scans the target for common directories using the default wordlist.

#### Directory Discovery with Extensions
```bash
gobuster dir -u http://target.example.com -w /usr/share/wordlists/dirb/common.txt -x php,html,txt
```
**Explanation:** Appends .php, .html, and .txt extensions to each word in the wordlist.

#### DNS Subdomain Enumeration
```bash
gobuster dns -d target.example.com -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt
```
**Explanation:** Brute-forces subdomains using DNS queries.

#### Virtual Host Discovery
```bash
gobuster vhost -u http://target.example.com -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt
```
**Explanation:** Discovers virtual hosts by fuzzing the Host header.

#### Filter by Status Code
```bash
gobuster dir -u http://target.example.com -w /usr/share/wordlists/dirb/common.txt -s 200,301,302
```
**Explanation:** Only shows results with status codes 200, 301, or 302.

#### Exclude Status Codes
```bash
gobuster dir -u http://target.example.com -w /usr/share/wordlists/dirb/common.txt -b 404
```
**Explanation:** Excludes responses with 404 status code.

#### Recursive Discovery
```bash
gobuster dir -u http://target.example.com -w /usr/share/wordlists/dirb/common.txt -r
```
**Explanation:** Automatically follows redirects and discovers subdirectories.

#### Custom Headers
```bash
gobuster dir -u http://target.example.com -w /usr/share/wordlists/dirb/common.txt -H "Authorization: Bearer token123"
```
**Explanation:** Adds custom headers to every request.

#### Verbose Output
```bash
gobuster dir -u http://target.example.com -w /usr/share/wordlists/dirb/common.txt -v
```
**Explanation:** Shows detailed output including URLs and status codes.

#### Save Output to File
```bash
gobuster dir -u http://target.example.com -w /usr/share/wordlists/dirb/common.txt -o results.txt
```
**Explanation:** Saves results to a text file.

#### Thread Control
```bash
gobuster dir -u http://target.example.com -w /usr/share/wordlists/dirb/common.txt -t 5
```
**Explanation:** Uses 5 concurrent threads instead of the default 10.

#### Rate Limiting
```bash
gobuster dir -u http://target.example.com -w /usr/share/wordlists/dirb/common.txt --delay 100ms
```
**Explanation:** Adds 100ms delay between requests.

#### Cookie Authentication
```bash
gobuster dir -u http://target.example.com -w /usr/share/wordlists/dirb/common.txt -c "session=abc123"
```
**Explanation:** Sends cookies with each request for authenticated scanning.

#### Basic Authentication
```bash
gobuster dir -u http://target.example.com -w /usr/share/wordlists/dirb/common.txt -U admin -P password
```
**Explanation:** Uses basic authentication for authenticated scanning.

#### User-Agent Customization
```bash
gobuster dir -u http://target.example.com -w /usr/share/wordlists/dirb/common.txt -a "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36"
```
**Explanation:** Sets a custom User-Agent string.

#### Wildcard Detection
```bash
gobuster dir -u http://target.example.com -w /usr/share/wordlists/dirb/common.txt --wildcard
```
**Explanation:** Forces scanning even if wildcard DNS is detected.

#### No Error Checking
```bash
gobuster dir -u http://target.example.com -w /usr/share/wordlists/dirb/common.txt --no-error
```
**Explanation:** Suppresses error messages during scanning.

#### URL Path
```bash
gobuster dir -u http://target.example.com -w /usr/share/wordlists/dirb/common.txt -p /api/
```
**Explanation:** Prepends /api/ to each path in the wordlist.

### Complete Flags Reference

#### Global Flags
| Flag | Description | Example |
|------|-------------|---------|
| `-h` | Show help | `gobuster -h` |
| `--delay` | Delay between requests | `--delay 100ms` |
| `-z` | Don't print status codes | `-z` |
| `--no-error` | Suppress errors | `--no-error` |
| `--version` | Show version | `--version` |
| `-t` | Number of threads | `-t 20` |
| `--timeout` | HTTP request timeout | `--timeout 30s` |
| `-p` | URL path prefix | `-p /api/` |
| `--proxy` | HTTP proxy | `--proxy http://proxy:8080` |
| `-a` | User-Agent string | `-a "Mozilla/5.0..."` |
| `--ua` | Random User-Agent | `--ua` |
| `-c` | Cookie data | `-c "session=abc123"` |
| `--cookie` | Cookie file | `--cookie cookies.txt` |
| `-U` | Basic auth username | `-U admin` |
| `-P` | Basic auth password | `-P password` |
| `--basic-auth` | Basic auth string | `--basic-auth user:pass` |
| `-k` | Skip TLS verification | `-k` |
| `--tls` | Use TLS | `--tls` |
| `--ca-cert` | CA certificate | `--ca-cert /path/to/ca.crt` |
| `-x` | File extensions | `-x php,html,txt` |
| `-r` | Follow redirects | `-r` |
| `-s` | Status codes to include | `-s 200,301,302` |
| `-b` | Status codes to exclude | `-b 404,403` |
| `--wildcard` | Force wildcard mode | `--wildcard` |
| `-o` | Output file | `-o results.txt` |
| `-v` | Verbose output | `-v` |
| `-n` | Don't print progress | `-n` |
| `-e` | Print full URLs | `-e` |
| `--ssl` | Use SSL | `--ssl` |

#### Dir Mode Flags
| Flag | Description | Example |
|------|-------------|---------|
| `dir` | Directory mode | `gobuster dir -u URL -w WORDLIST` |
| `-u` | Target URL | `-u http://target.com` |
| `-w` | Wordlist path | `-w /path/to/wordlist.txt` |
| `-x` | File extensions | `-x php,html,txt` |
| `-r` | Follow redirects | `-r` |
| `-k` | Skip TLS verification | `-k` |
| `--wildcard` | Force wildcard mode | `--wildcard` |
| `-f` | Append / to every request | `-f` |
| `-l` | Include response length | `-l` |
| `--expanded` | Show full URLs | `--expanded` |
| `--no-tls-validation` | Skip TLS validation | `--no-tls-validation` |
| `--threads` | Number of threads | `--threads 10` |
| `--timeout` | Request timeout | `--timeout 10s` |
| `--delay` | Delay between requests | `--delay 100ms` |
| `--wordlist` | Wordlist path (alias) | `--wordlist /path/to/wordlist.txt` |
| `--url` | Target URL (alias) | `--url http://target.com` |
| `--statuscodes` | Status codes to include (alias) | `--statuscodes 200,301` |
| `--insecure` | Skip TLS verification (alias) | `--insecure` |
| `--proxy` | HTTP proxy | `--proxy http://proxy:8080` |
| `--useragent` | User-Agent string | `--useragent "Mozilla/5.0..."` |
| `--basic-auth` | Basic auth credentials | `--basic-auth user:pass` |
| `--cookie` | Cookie string | `--cookie "session=abc"` |
| `--exclude-length` | Exclude response length | `--exclude-length 0` |
| `-o` | Output file | `-o results.txt` |
| `-v` | Verbose output | `-v` |
| `--no-error` | Suppress errors | `--no-error` |
| `-n` | Don't print progress | `-n` |
| `-e` | Print full URLs | `-e` |

#### DNS Mode Flags
| Flag | Description | Example |
|------|-------------|---------|
| `dns` | DNS mode | `gobuster dns -d DOMAIN -w WORDLIST` |
| `-d` | Target domain | `-d target.example.com` |
| `-w` | Wordlist path | `-w /path/to/wordlist.txt` |
| `-r` | Use custom DNS resolver | `-r 8.8.8.8` |
| `--resolver` | Custom DNS resolver | `--resolver 8.8.8.8` |
| `--wildcard` | Force wildcard mode | `--wildcard` |
| `--threads` | Number of threads | `--threads 10` |
| `--timeout` | Request timeout | `--timeout 10s` |
| `--delay` | Delay between requests | `--delay 100ms` |
| `--wordlist` | Wordlist path (alias) | `--wordlist /path/to/wordlist.txt` |
| `--domain` | Target domain (alias) | `--domain target.example.com` |
| `-o` | Output file | `-o results.txt` |
| `-v` | Verbose output | `-v` |
| `--no-error` | Suppress errors | `--no-error` |
| `-n` | Don't print progress | `-n` |
| `-c` | Show CNAME records | `-c` |
| `--show-ips` | Show IP addresses | `--show-ips` |

#### Vhost Mode Flags
| Flag | Description | Example |
|------|-------------|---------|
| `vhost` | Virtual host mode | `gobuster vhost -u URL -w WORDLIST` |
| `-u` | Target URL | `-u http://target.com` |
| `-w` | Wordlist path | `-w /path/to/wordlist.txt` |
| `-k` | Skip TLS verification | `-k` |
| `--wildcard` | Force wildcard mode | `--wildcard` |
| `--append-domain` | Append domain to words | `--append-domain` |
| `--exclude-length` | Exclude response length | `--exclude-length 0` |
| `--threads` | Number of threads | `--threads 10` |
| `--timeout` | Request timeout | `--timeout 10s` |
| `--delay` | Delay between requests | `--delay 100ms` |
| `--wordlist` | Wordlist path (alias) | `--wordlist /path/to/wordlist.txt` |
| `--url` | Target URL (alias) | `--url http://target.com` |
| `--proxy` | HTTP proxy | `--proxy http://proxy:8080` |
| `--useragent` | User-Agent string | `--useragent "Mozilla/5.0..."` |
| `--basic-auth` | Basic auth credentials | `--basic-auth user:pass` |
| `--cookie` | Cookie string | `--cookie "session=abc"` |
| `-o` | Output file | `-o results.txt` |
| `-v` | Verbose output | `-v` |
| `--no-error` | Suppress errors | `--no-error` |
| `-n` | Don't print progress | `-n` |

---

## Chapter 4: Intermediate Usage

### Bash Scripting and Automation

#### Automated Directory Discovery Script
```bash
#!/bin/bash
# auto_gobuster.sh - Automated directory discovery

TARGET=$1
WORDLIST=$2
OUTPUT_DIR="gobuster_$(date +%Y%m%d_%H%M%S)"

mkdir -p "$OUTPUT_DIR"

echo "[*] Starting directory discovery on $TARGET"

# Phase 1: Quick scan
echo "[*] Phase 1: Quick scan"
gobuster dir -u "$TARGET" \
    -w /usr/share/wordlists/dirb/common.txt \
    -b 404 \
    -o "$OUTPUT_DIR/quick_scan.txt" \
    -q

# Phase 2: Deep scan with extensions
echo "[*] Phase 2: Deep scan with extensions"
gobuster dir -u "$TARGET" \
    -w /usr/share/wordlists/dirb/big.txt \
    -x php,html,txt,bak,old,config \
    -b 404 \
    -o "$OUTPUT_DIR/deep_scan.txt" \
    -q

echo "[*] Results saved to $OUTPUT_DIR"
```

#### Multi-Target Scanning Script
```bash
#!/bin/bash
# multi_target_gobuster.sh

TARGETS_FILE=$1
WORDLIST=$2

while IFS= read -r target; do
    echo "[*] Scanning: $target"
    gobuster dir -u "$target" \
        -w "$WORDLIST" \
        -b 404 \
        -o "gobuster_${target//\//_}.txt" \
        -t 20 \
        -q
done < "$TARGETS_FILE"
```

#### Full Recon Pipeline
```bash
#!/bin/bash
# full_recon.sh

DOMAIN=$1

echo "[+] Starting full reconnaissance for $DOMAIN"

# Phase 1: Subdomain enumeration
echo "[*] Phase 1: Subdomain enumeration"
gobuster dns -d "$DOMAIN" \
    -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt \
    -o "subdomains.txt" \
    -q

# Phase 2: Virtual host discovery
echo "[*] Phase 2: Virtual host discovery"
gobuster vhost -u "http://$DOMAIN" \
    -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt \
    -o "vhosts.txt" \
    -q

# Phase 3: Directory discovery
echo "[*] Phase 3: Directory discovery"
gobuster dir -u "http://$DOMAIN" \
    -w /usr/share/wordlists/dirb/big.txt \
    -b 404 \
    -o "directories.txt" \
    -q

echo "[+] Reconnaissance complete"
```

### Python Scripting

#### Python Wrapper for Gobuster
```python
#!/usr/bin/env python3
"""Python wrapper for Gobuster scanning."""

import subprocess
import sys

def run_gobuster(mode, target, wordlist, options=None):
    """
    Run Gobuster scan and return results.
    
    Args:
        mode: 'dir', 'dns', or 'vhost'
        target: Target URL or domain
        wordlist: Path to wordlist file
        options: Dict of additional options
    
    Returns:
        str: Gobuster output
    """
    cmd = ['gobuster', mode]
    
    if mode == 'dir' or mode == 'vhost':
        cmd.extend(['-u', target])
    elif mode == 'dns':
        cmd.extend(['-d', target])
    
    cmd.extend(['-w', wordlist])
    
    if options:
        if 'threads' in options:
            cmd.extend(['-t', str(options['threads'])])
        if 'extensions' in options:
            cmd.extend(['-x', options['extensions']])
        if 'status_codes' in options:
            cmd.extend(['-s', options['status_codes']])
        if 'exclude' in options:
            cmd.extend(['-b', options['exclude']])
        if 'output' in options:
            cmd.extend(['-o', options['output']])
        if 'timeout' in options:
            cmd.extend(['--timeout', options['timeout']])
    
    try:
        result = subprocess.run(cmd, capture_output=True, text=True, timeout=600)
        return result.stdout
    except subprocess.TimeoutExpired:
        print(f"[-] Scan timed out for {target}")
        return ""

def parse_gobuster_output(output):
    """Parse Gobuster output and extract results."""
    results = []
    for line in output.split('\n'):
        if line.startswith('/') or 'Status:' in line:
            parts = line.split()
            if len(parts) >= 1:
                path = parts[0]
                status = None
                size = None
                for i, part in enumerate(parts):
                    if part == '(Status:':
                        status = parts[i+1].rstrip(')')
                    if part == '[Size:':
                        size = parts[i+1].rstrip(']')
                results.append({
                    'path': path,
                    'status': status,
                    'size': size
                })
    return results

def main():
    if len(sys.argv) < 3:
        print(f"Usage: {sys.argv[0]} <mode> <target> [wordlist]")
        print(f"Modes: dir, dns, vhost")
        sys.exit(1)
    
    mode = sys.argv[1]
    target = sys.argv[2]
    wordlist = sys.argv[3] if len(sys.argv) > 3 else '/usr/share/wordlists/dirb/common.txt'
    
    output = run_gobuster(mode, target, wordlist, {
        'threads': 20,
        'exclude': '404'
    })
    
    results = parse_gobuster_output(output)
    print(f"\n[+] Found {len(results)} results:")
    for r in results:
        print(f"  {r['path']} - Status: {r['status']} Size: {r['size']}")

if __name__ == '__main__':
    main()
```

### Tool Chaining

#### Gobuster with FFuF
```bash
# Phase 1: Quick scan with Gobuster
gobuster dir -u http://target.example.com \
    -w /usr/share/wordlists/dirb/common.txt \
    -b 404 \
    -o quick_results.txt

# Phase 2: Detailed fuzzing with FFuF
ffuf -u http://target.example.com/FUZZ \
    -w discovered_paths.txt \
    -mc 200,301,302
```

#### Gobuster with Nmap
```bash
# First, discover services
nmap -sV -p 80,443,8080,8443 target.example.com

# Then fuzz each discovered service
gobuster dir -u http://target.example.com:8080 \
    -w /usr/share/wordlists/dirb/common.txt \
    -b 404
```

#### Gobuster with Nuclei
```bash
# Discover directories
gobuster dir -u http://target.example.com \
    -w /usr/share/wordlists/dirb/common.txt \
    -b 404 \
    -o directories.txt

# Scan discovered paths with Nuclei
nuclei -l directories.txt -t cves/
```

#### Gobuster with Sublist3r
```bash
# Find subdomains
sublist3r -d target.example.com -o subdomains.txt

# Fuzz each subdomain
while read subdomain; do
    gobuster dir -u "http://$subdomain" \
        -w /usr/share/wordlists/dirb/common.txt \
        -b 404 \
        -o "gobuster_$subdomain.txt"
done < subdomains.txt
```

### Output Processing and Filtering

#### Extracting Only 200 Status Codes
```bash
gobuster dir -u http://target.example.com -w wordlist.txt -s 200
```

#### Sorting Results by Size
```bash
gobuster dir -u http://target.example.com -w wordlist.txt -o results.txt
sort -t'[' -k2 -n results.txt
```

#### Counting Results by Status Code
```bash
gobuster dir -u http://target.example.com -w wordlist.txt | \
    grep -oP 'Status: \K[0-9]+' | sort | uniq -c
```

---

## Chapter 5: Advanced Usage

### Custom Wordlists

#### Combining Wordlists
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

### Evasion and Rate Limiting

#### Request Throttling
```bash
# Slow scan to avoid detection
gobuster dir -u http://target.example.com \
    -w wordlist.txt \
    --delay 500ms \
    -t 5
```

#### Using Proxies
```bash
# Route through proxy
gobuster dir -u http://target.example.com \
    -w wordlist.txt \
    --proxy http://proxy:8080
```

#### Random User-Agent
```bash
# Use random User-Agent
gobuster dir -u http://target.example.com \
    -w wordlist.txt \
    --random-agent
```

#### Request Delay Between Threads
```bash
gobuster dir -u http://target.example.com \
    -w wordlist.txt \
    --delay 200ms
```

### Integration with Pentest Workflows

#### Full Web Assessment Pipeline
```bash
#!/bin/bash
# full_web_assessment.sh

TARGET=$1

echo "[+] Phase 1: Subdomain discovery"
gobuster dns -d "$TARGET" \
    -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt \
    -o subdomains.txt \
    -q

echo "[+] Phase 2: Port scanning"
nmap -iL subdomains.txt -p 80,443,8080,8443 --open -oN ports.txt

echo "[+] Phase 3: Directory discovery"
while read host; do
    gobuster dir -u "http://$host" \
        -w /usr/share/wordlists/dirb/big.txt \
        -b 404 \
        -o "gobuster_$host.txt" \
        -q
done < subdomains.txt

echo "[+] Phase 4: Parameter discovery"
# Use discovered endpoints for parameter fuzzing
find . -name "gobuster_*.txt" -exec grep -oP '/\K[^ ]+' {} \; | \
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
# bug_bounty_gobuster.sh

DOMAIN=$1

echo "[+] Starting bug bounty reconnaissance for $DOMAIN"

# Subdomain enumeration
echo "[*] Phase 1: Subdomain discovery"
amass enum -passive -d "$DOMAIN" -o amass_subs.txt
gobuster dns -d "$DOMAIN" \
    -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt \
    -o gobuster_subs.txt \
    -q
cat amass_subs.txt gobuster_subs.txt | sort -u > all_subs.txt

# Probe for live hosts
echo "[*] Phase 2: Probing live hosts"
httpx -l all_subs.txt -o live_hosts.txt -silent

# Directory discovery
echo "[*] Phase 3: Directory discovery"
while read host; do
    gobuster dir -u "http://$host" \
        -w /usr/share/wordlists/dirb/common.txt \
        -b 404 \
        -o "gobuster_${host}.txt" \
        -t 20 \
        -q &
done < live_hosts.txt
wait

echo "[+] Reconnaissance complete"
```

### Advanced DNS Enumeration

#### Using Custom DNS Resolver
```bash
# Use specific DNS resolver
gobuster dns -d target.example.com \
    -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt \
    -r 8.8.8.8 \
    --show-ips
```

#### Wildcard Detection
```bash
# Force wildcard mode
gobuster dns -d target.example.com \
    -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt \
    --wildcard
```

### Advanced Virtual Host Discovery

#### VHost with Response Filtering
```bash
# Filter by response size
gobuster vhost -u http://target.example.com \
    -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt \
    --exclude-length 0
```

#### VHost with Domain Appending
```bash
# Append domain to wordlist entries
gobuster vhost -u http://target.example.com \
    -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt \
    --append-domain
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: Web Application Audit

**Objective:** Perform comprehensive directory and subdomain discovery on a corporate web application.

**Step 1: Subdomain Discovery**
```bash
# Quick DNS enumeration
gobuster dns -d corporate.example.com \
    -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt \
    -o subdomains.txt \
    --show-ips
```

**Step 2: Directory Discovery**
```bash
# Quick scan with common wordlist
gobuster dir -u http://app.corporate.example.com \
    -w /usr/share/wordlists/dirb/common.txt \
    -b 404 \
    -o quick_scan.txt

# Deep scan with extended wordlist
gobuster dir -u http://app.corporate.example.com \
    -w /usr/share/wordlists/dirb/big.txt \
    -x php,html,js,txt,bak,old,config \
    -b 404 \
    -o deep_scan.txt
```

**Step 3: Virtual Host Discovery**
```bash
# Discover virtual hosts
gobuster vhost -u http://app.corporate.example.com \
    -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt \
    -o vhosts.txt
```

**Step 4: Analysis and Reporting**
```bash
# Combine and analyze results
cat quick_scan.txt deep_scan.txt vhosts.txt | sort -u > findings.txt
```

### Scenario 2: CTF Challenge

**Objective:** Find hidden flags in a CTF web challenge.

**Step 1: Initial Discovery**
```bash
# Basic directory scan
gobuster dir -u http://ctf.example.com \
    -w /usr/share/wordlists/dirb/common.txt

# Check robots.txt and sitemap
curl -s http://ctf.example.com/robots.txt
curl -s http://ctf.example.com/sitemap.xml
```

**Step 2: Deep Content Discovery**
```bash
# Scan with extensions
gobuster dir -u http://ctf.example.com \
    -w /usr/share/wordlists/dirb/common.txt \
    -x php,html,txt,bak,old,swp,git

# Look for backup files
gobuster dir -u http://ctf.example.com \
    -w /usr/share/wordlists/seclists/Discovery/Web-Content/backup.txt \
    -b 404
```

**Step 3: DNS Enumeration**
```bash
# Find subdomains
gobuster dns -d ctf.example.com \
    -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt
```

### Scenario 3: Bug Bounty

**Objective:** Comprehensive web reconnaissance for a bug bounty program.

**Step 1: Scope Discovery**
```bash
# Get all subdomains
gobuster dns -d target.com \
    -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt \
    -o gobuster_subs.txt \
    --show-ips

# Combine with other tools
amass enum -passive -d target.com -o amass_subs.txt
cat gobuster_subs.txt amass_subs.txt | sort -u > all_subdomains.txt
```

**Step 2: Directory Discovery**
```bash
# Find hidden directories
gobuster dir -u http://target.com \
    -w /usr/share/wordlists/dirb/big.txt \
    -b 404 \
    -o directories.txt

# Find hidden files
gobuster dir -u http://target.com \
    -w /usr/share/wordlists/dirb/common.txt \
    -x php,html,js,txt,bak,old,config \
    -o files.txt
```

**Step 3: Virtual Host Discovery**
```bash
# Find virtual hosts
gobuster vhost -u http://target.com \
    -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt \
    -o vhosts.txt
```

---

## Chapter 7: Detection & Defense

### IDS/IPS Detection

**Signature-Based Detection:**
- Gobuster generates distinctive User-Agent strings by default
- Rapid sequential requests to similar paths trigger anomaly detection
- High-volume requests from single IP addresses create detectable patterns
- Specific request patterns (sequential directory enumeration) are signature-matchable

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
# /etc/fail2ban/filter.d/gobuster.conf
[Definition]
failregex = ^<HOST> -.*"(GET|POST) /(admin|backup|config).*" (404|403)$
ignoreregex =
```

```ini
# /etc/fail2ban/jail.d/gobuster.conf
[gobuster]
enabled = true
filter = gobuster
logpath = /var/log/apache2/access.log
maxretry = 10
findtime = 60
bantime = 3600
```

**Web Application Firewall Rules:**
```apache
# ModSecurity rule to detect Gobuster
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
gobuster dir -u http://target.example.com -w wordlist.txt -v
```

**Error: "Connection refused"**
```bash
# Solution: Verify target is running
nmap -p 80,443 target.example.com

# Check if port is different
gobuster dir -u http://target.example.com:8080 -w wordlist.txt
```

**Error: "Too many open files"**
```bash
# Solution: Increase file descriptor limit
ulimit -n 65535

# Or reduce threads
gobuster dir -u http://target.example.com -w wordlist.txt -t 5
```

**Error: "Connection timeout"**
```bash
# Solution: Increase timeout
gobuster dir -u http://target.example.com -w wordlist.txt --timeout 30s

# Check network connectivity
ping target.example.com
```

**Error: "Wildcard found"**
```bash
# Solution: Use --wildcard flag
gobuster dir -u http://target.example.com -w wordlist.txt --wildcard

# Or filter by response size
gobuster dir -u http://target.example.com -w wordlist.txt --exclude-length 0
```

### Performance Optimization

**Maximizing Speed:**
```bash
# Use multiple threads
gobuster dir -u http://target.example.com -w wordlist.txt -t 100

# Use efficient wordlists
# Smaller, focused wordlists are faster than large generic ones

# Disable progress output
gobuster dir -u http://target.example.com -w wordlist.txt -n
```

**Reducing Memory Usage:**
```bash
# Process large wordlists in chunks
split -l 10000 big_wordlist.txt chunk_
for chunk in chunk_*; do
    gobuster dir -u http://target.example.com -w "$chunk" -b 404
done
```

**Network Optimization:**
```bash
# Use connection pooling
gobuster dir -u http://target.example.com -w wordlist.txt --timeout 10s

# Use custom DNS resolver
gobuster dns -d target.example.com -w wordlist.txt -r 8.8.8.8
```

### FAQ

**Q: What's the difference between dir, dns, and vhost modes?**
A: `dir` discovers web directories/files, `dns` brute-forces subdomains via DNS queries, and `vhost` discovers virtual hosts via Host header manipulation.

**Q: How do I handle wildcard DNS?**
A: Use the `--wildcard` flag to force scanning, or `--exclude-length` to filter responses by size.

**Q: Can Gobuster test POST parameters?**
A: Not directly. For POST parameter fuzzing, use FFuF or WFuzz instead.

**Q: How do I avoid getting blocked?**
A: Use `--delay`, reduce threads with `-t`, and consider using `--proxy`.

**Q: Can Gobuster discover S3 buckets?**
A: Yes, use the `s3` mode: `gobuster s3 -w wordlist.txt`

---

## Resources

### Official Documentation
- [Gobuster GitHub Repository](https://github.com/OJ/gobuster)
- [Gobuster Wiki](https://github.com/OJ/gobuster/wiki)
- [Gobuster Usage Guide](https://github.com/OJ/gobuster/blob/master/README.md)

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
- [Gobuster Crash Course](https://www.youtube.com/watch?v=example)
- [Directory Brute-Forcing with Gobuster](https://www.youtube.com/watch?v=example)
- [Bug Bounty Recon with Gobuster](https://www.youtube.com/watch?v=example)

### Related Tools
- [FFuF](https://github.com/ffuf/ffuf) - Fast web fuzzer
- [DirB](https://github.com/v0re/dirb) - Web content scanner
- [DirSearch](https://github.com/maurosoria/dirsearch) - Web path scanner
- [Feroxbuster](https://github.com/epi052/feroxbuster) - Fast, simple, recursive content discovery tool
