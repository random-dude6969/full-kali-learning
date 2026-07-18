---
tool_name: "gospider"
display_name: "GoSpider"
mitre_tactic: "reconnaissance"
mitre_tactic_id: "TA0043"
mitre_subcategory: "web_scanning"
install_command: "go install github.com/jaeles-project/gospider@latest"
version: "1.1.6"
github_repo: "https://github.com/jaeles-project/gospider"
kali_tools_page: "https://www.kali.org/tools/gospider/"
difficulty_level: "beginner"
requires_root: false
gui_available: false
tags: ["spider", "crawler", "web", "reconnaissance", "link-discovery"]
related_tools: ["hakrawler", "katana", "gau", "waybackurls"]
---

# GoSpider — Fast Web Crawler

## Chapter 1: Introduction & Overview

### What is GoSpider?
GoSpider is a fast, efficient web spider written in Go that automatically discovers links, subdomains, and endpoints on websites. It crawls web applications to map out their structure, finding hidden links, JavaScript files, endpoints, and other resources. GoSpider is designed for web reconnaissance and is widely used in penetration testing and bug bounty hunting to discover attack surfaces.

### History & Background
- **Created:** 2020 by jaeles-project (GitHub user)
- **Language:** Go (Golang)
- **License:** MIT License
- **Current Version:** 1.1.6 (as of 2024)
- **Purpose:** Web crawling, link discovery, endpoint mapping

GoSpider was created to provide a fast, multi-threaded web crawler that can efficiently discover links and endpoints on web applications. It supports multiple crawling modes including JavaScript rendering, recursive crawling, and source code analysis. The tool is particularly effective for discovering hidden endpoints and mapping web application structure.

### When to Use This Tool
- **Link discovery:** Find all links and endpoints on a website
- **JavaScript analysis:** Extract endpoints from JavaScript files
- **Subdomain discovery:** Discover subdomains through web crawling
- **API endpoint discovery:** Find undocumented API endpoints
- **Content mapping:** Map out the complete structure of a web application
- **Penetration testing:** Phase 1 reconnaissance for web applications
- **Bug bounty hunting:** Discover hidden endpoints for bug bounty programs

### When NOT to Use This Tool
- Without explicit written authorization from the target owner
- Against production systems without prior coordination
- When you need deep vulnerability scanning (use specialized tools instead)
- For denial-of-service testing (crawling can generate significant traffic)
- Against systems you do not own or have permission to test

### Legal and Ethical Considerations
- **Always obtain written authorization** before testing any system you do not own
- **Define scope clearly** — know exactly which hosts and paths are in scope
- **Use rate limiting** to avoid overwhelming targets or causing service degradation
- **Document everything** — keep records of authorization, scope, and findings
- **Follow responsible disclosure** — report vulnerabilities through proper channels
- **Understand local laws** — unauthorized computer access is illegal in most jurisdictions

### Key Concepts
- **Web Crawler/Spider:** Automated program that browses the web following links
- **Endpoint:** A specific URL path or API route on a web server
- **JavaScript Analysis:** Extracting information from JavaScript files
- **Recursive Crawling:** Automatically following discovered links to crawl deeper
- **Source Code Analysis:** Analyzing HTML/JavaScript source code for hidden links
- **Link Depth:** How many levels deep the crawler follows links
- **User-Agent:** The identifier sent with HTTP requests to identify the crawler

---

## Chapter 2: Installation & Setup

### System Requirements
- **Operating System:** Linux, macOS, Windows, or any Go-supported platform
- **RAM:** Minimum 256 MB (512 MB recommended for large crawls)
- **Disk Space:** 50 MB for installation + space for output
- **CPU:** Multi-core recommended for maximum performance
- **Network:** Stable internet connection
- **Go:** Required for building from source (Go 1.18+)

### Installation Methods

#### Method 1: Go Install (Recommended)
```bash
go install github.com/jaeles-project/gospider@latest
```

#### Method 2: From Source
```bash
git clone https://github.com/jaeles-project/gospider.git
cd gospider
go build -o gospider
sudo cp gospider /usr/local/bin/
```

#### Method 3: Download Release Binary
```bash
# Visit https://github.com/jaeles-project/gospider/releases
# Download the appropriate binary for your platform
chmod +x gospider
sudo mv gospider /usr/local/bin/
```

#### Method 4: Docker
```bash
docker run -it --rm jaeles/gospider -s http://target.example.com
```

### Configuration
GoSpider does not require a configuration file for basic operation. All options are passed via command-line arguments. For persistent configurations, consider using shell aliases or wrapper scripts.

### Verification
```bash
gospider --version
# Output: gospider version v1.1.6
```

### Updating
```bash
# Go
go install github.com/jaeles-project/gospider@latest

# From source
cd gospider
git pull
go build -o gospider
sudo cp gospider /usr/local/bin/
```

---

## Chapter 3: Basic Usage

### First Run
```bash
gospider -s http://target.example.com
```
**Output:**
```
GoSpider v1.1.6
[Spider] Starting: http://target.example.com
[Spider] Found link: http://target.example.com/about
[Spider] Found link: http://target.example.com/contact
[Spider] Found link: http://target.example.com/login
[Spider] Found link: http://target.example.com/api/users
[Spider] Found link: http://target.example.com/static/js/app.js
[Spider] Found link: http://target.example.com/robots.txt
[Spider] Found link: http://target.example.com/sitemap.xml
[Spider] Found javascript: http://target.example.com/static/js/app.js
[Spider] Crawled: 7 URLs
[Spider] Found: 12 links
[Spider] Completed in 15.32s
```

### Essential Commands

#### Basic Web Crawling
```bash
gospider -s http://target.example.com
```
**Explanation:** Crawls the target website and discovers links.

#### Crawling with Depth Limit
```bash
gospider -s http://target.example.com -d 3
```
**Explanation:** Crawls up to 3 levels deep from the starting URL.

#### Crawling Multiple URLs
```bash
gospider -s http://target.example.com -S urls.txt
```
**Explanation:** Crawls multiple URLs from a file (one per line).

#### Thread Control
```bash
gospider -s http://target.example.com -t 20
```
**Explanation:** Uses 20 concurrent threads for faster crawling.

#### JavaScript Analysis
```bash
gospider -s http://target.example.com --js
```
**Explanation:** Analyzes JavaScript files to extract endpoints.

#### Output to File
```bash
gospider -s http://target.example.com -o results.txt
```
**Explanation:** Saves discovered links to a file.

#### Verbose Output
```bash
gospider -s http://target.example.com -v
```
**Explanation:** Shows detailed output during crawling.

#### Custom User-Agent
```bash
gospider -s http://target.example.com -a "Mozilla/5.0 (Windows NT 10.0; Win64; x64)"
```
**Explanation:** Sets a custom User-Agent string.

#### Include Subdomains
```bash
gospider -s http://target.example.com --include-subdomains
```
**Explanation:** Crawls links on subdomains as well.

#### Exclude Patterns
```bash
gospider -s http://target.example.com --exclude "logout,admin"
```
**Explanation:** Excludes URLs containing specified patterns.

#### Blacklist Keywords
```bash
gospider -s http://target.example.com --blacklist "logout,admin,test"
```
**Explanation:** Skips URLs containing blacklisted keywords.

#### Whitelist Keywords
```bash
gospider -s http://target.example.com --whitelist "api,v1,v2"
```
**Explanation:** Only follows URLs containing whitelisted keywords.

#### Regex Filtering
```bash
gospider -s http://target.example.com --regex "api/.*"
```
**Explanation:** Only follows URLs matching the specified regex pattern.

#### Source Code Analysis
```bash
gospider -s http://target.example.com --source
```
**Explanation:** Analyzes HTML source code for hidden links.

#### Sitemap Crawling
```bash
gospider -s http://target.example.com --sitemap
```
**Explanation:** Discovers and follows sitemap.xml.

#### Robots.txt Crawling
```bash
gospider -s http://target.example.com --robots
```
**Explanation:** Discovers and follows robots.txt.

#### Cookie Support
```bash
gospider -s http://target.example.com --cookie "session=abc123"
```
**Explanation:** Sends cookies with each request for authenticated crawling.

#### Proxy Support
```bash
gospider -s http://target.example.com --proxy http://proxy:8080
```
**Explanation:** Routes all requests through a proxy server.

#### Timeout Configuration
```bash
gospider -s http://target.example.com --timeout 30
```
**Explanation:** Sets 30-second timeout for connections.

#### Retry Configuration
```bash
gospider -s http://target.example.com --retry 3
```
**Explanation:** Retries failed requests up to 3 times.

### Complete Flags Reference

#### Target Options
| Flag | Description | Example |
|------|-------------|---------|
| `-s` | Starting URL | `-s http://target.com` |
| `-S` | URLs file | `-S urls.txt` |
| `--sitemap` | Follow sitemap.xml | `--sitemap` |
| `--robots` | Follow robots.txt | `--robots` |

#### Crawling Options
| Flag | Description | Example |
|------|-------------|---------|
| `-d` | Crawl depth | `-d 3` |
| `-t` | Number of threads | `-t 20` |
| `--include-subdomains` | Include subdomains | `--include-subdomains` |
| `--js` | Analyze JavaScript | `--js` |
| `--source` | Analyze source code | `--source` |
| `--preload` | Preload all pages | `--preload` |
| `--no-redirect` | Don't follow redirects | `--no-redirect` |
| `--no-head` | Don't use HEAD requests | `--no-head` |
| `--no-ext` | Don't extract external links | `--no-ext` |

#### Filtering Options
| Flag | Description | Example |
|------|-------------|---------|
| `--blacklist` | Blacklist keywords | `--blacklist "admin,logout"` |
| `--whitelist` | Whitelist keywords | `--whitelist "api,v1"` |
| `--exclude` | Exclude URL patterns | `--exclude "logout"` |
| `--regex` | Regex filter | `--regex "api/.*"` |
| `--filter-extensions` | Filter file extensions | `--filter-extensions "jpg,png,gif"` |

#### Network Options
| Flag | Description | Example |
|------|-------------|---------|
| `-a` | User-Agent string | `-a "Mozilla/5.0..."` |
| `--random-agent` | Random User-Agent | `--random-agent` |
| `--proxy` | HTTP proxy | `--proxy http://proxy:8080` |
| `--cookie` | Cookie string | `--cookie "session=abc"` |
| `--timeout` | Connection timeout | `--timeout 30` |
| `--retry` | Retry count | `--retry 3` |
| `--delay` | Delay between requests | `--delay 1` |
| `--rate-limit` | Rate limit (req/sec) | `--rate-limit 10` |

#### Output Options
| Flag | Description | Example |
|------|-------------|---------|
| `-o` | Output file | `-o results.txt` |
| `-j` | JSON output | `-j` |
| `-c` | CSV output | `-c` |
| `-v` | Verbose output | `-v` |
| `-q` | Quiet mode | `-q` |
| `--no-color` | Disable colors | `--no-color` |

#### Authentication Options
| Flag | Description | Example |
|------|-------------|---------|
| `-u` | Basic auth username | `-u admin` |
| `-p` | Basic auth password | `-p password` |
| `-- bearer` | Bearer token | `--bearer "token123"` |

---

## Chapter 4: Intermediate Usage

### Bash Scripting and Automation

#### Automated Web Crawling Script
```bash
#!/bin/bash
# auto_gospider.sh - Automated web crawling

TARGET=$1
OUTPUT_DIR="gospider_$(date +%Y%m%d_%H%M%S)"

mkdir -p "$OUTPUT_DIR"

echo "[*] Starting web crawling on $TARGET"

# Basic crawl
echo "[*] Phase 1: Basic crawl"
gospider -s "$TARGET" \
    -o "$OUTPUT_DIR/basic_links.txt" \
    -t 20

# JavaScript analysis
echo "[*] Phase 2: JavaScript analysis"
gospider -s "$TARGET" \
    --js \
    -o "$OUTPUT_DIR/js_links.txt" \
    -t 10

# Deep crawl
echo "[*] Phase 3: Deep crawl"
gospider -s "$TARGET" \
    -d 5 \
    --include-subdomains \
    -o "$OUTPUT_DIR/deep_links.txt" \
    -t 10

echo "[*] Results saved to $OUTPUT_DIR"
```

#### Multi-Target Crawling Script
```bash
#!/bin/bash
# multi_target_gospider.sh

TARGETS_FILE=$1
OUTPUT_BASE="gospider_multi"

while IFS= read -r target; do
    echo "[*] Crawling: $target"
    OUTPUT_DIR="$OUTPUT_BASE/$(echo $target | sed 's|https\?://||' | tr '/' '_')"
    mkdir -p "$OUTPUT_DIR"
    
    gospider -s "$target" \
        -o "$OUTPUT_DIR/links.txt" \
        --js \
        -t 20 \
        -v
done < "$TARGETS_FILE"
```

#### Recon Pipeline with Other Tools
```bash
#!/bin/bash
# recon_pipeline.sh

TARGET=$1

echo "[+] Phase 1: GoSpider - Link Discovery"
gospider -s "$TARGET" \
    --js \
    --source \
    -o gospider_links.txt \
    -t 20

echo "[+] Phase 2: Extract Endpoints"
grep -oP 'https?://[^\s"]+' gospider_links.txt | sort -u > endpoints.txt

echo "[+] Phase 3: Find Subdomains"
grep -oP 'https?://([a-zA-Z0-9-]+\.)+[a-zA-Z]{2,}' gospider_links.txt | \
    sed 's|https\?://||' | sort -u > subdomains.txt

echo "[+] Phase 4: Directory Discovery"
cat endpoints.txt | while read url; do
    path=$(echo "$url" | sed 's|https\?://[^/]*||')
    echo "$path"
done | sort -u > paths.txt

echo "[+] Phase 5: Parameter Discovery"
grep -oP '[?&][a-zA-Z0-9_]+=' gospider_links.txt | \
    sed 's/[?&]=//' | sort -u > parameters.txt

echo "[+] Reconnaissance complete"
```

### Python Scripting

#### Python Wrapper for GoSpider
```python
#!/usr/bin/env python3
"""Python wrapper for GoSpider web crawling."""

import subprocess
import json
import sys

def run_gospider(url, options=None):
    """
    Run GoSpider crawl and return results.
    
    Args:
        url: Target URL
        options: Dict of additional options
    
    Returns:
        list: Discovered links
    """
    cmd = ['gospider', '-s', url]
    
    if options:
        if 'depth' in options:
            cmd.extend(['-d', str(options['depth'])])
        if 'threads' in options:
            cmd.extend(['-t', str(options['threads'])])
        if 'javascript' in options and options['javascript']:
            cmd.append('--js')
        if 'output' in options:
            cmd.extend(['-o', options['output']])
        if 'json' in options and options['json']:
            cmd.append('-j')
        if 'proxy' in options:
            cmd.extend(['--proxy', options['proxy']])
        if 'timeout' in options:
            cmd.extend(['--timeout', str(options['timeout'])])
    
    try:
        result = subprocess.run(cmd, capture_output=True, text=True, timeout=600)
        
        if options and 'json' in options and options['json']:
            return json.loads(result.stdout)
        
        return result.stdout.strip().split('\n')
    except subprocess.TimeoutExpired:
        print(f"[-] Crawl timed out for {url}")
        return []
    except json.JSONDecodeError:
        return result.stdout.strip().split('\n')

def parse_gospider_output(output):
    """Parse GoSpider output and extract links."""
    links = {
        'urls': [],
        'javascript': [],
        'forms': [],
        'subdomains': []
    }
    
    for line in output:
        if line.startswith('http'):
            links['urls'].append(line)
        if '.js' in line:
            links['javascript'].append(line)
        if 'form' in line.lower():
            links['forms'].append(line)
    
    return links

def main():
    if len(sys.argv) < 2:
        print(f"Usage: {sys.argv[0]} <url> [options]")
        print(f"Options: depth=N, threads=N, javascript, proxy=URL")
        sys.exit(1)
    
    url = sys.argv[1]
    options = {}
    
    for arg in sys.argv[2:]:
        if arg.startswith('depth='):
            options['depth'] = int(arg.split('=')[1])
        elif arg.startswith('threads='):
            options['threads'] = int(arg.split('=')[1])
        elif arg == 'javascript':
            options['javascript'] = True
        elif arg.startswith('proxy='):
            options['proxy'] = arg.split('=')[1]
    
    links = run_gospider(url, options)
    parsed = parse_gospider_output(links)
    
    print(f"\n[+] Results for {url}:")
    print(f"    URLs: {len(parsed['urls'])}")
    print(f"    JavaScript files: {len(parsed['javascript'])}")
    print(f"    Forms: {len(parsed['forms'])}")

if __name__ == '__main__':
    main()
```

### Tool Chaining

#### GoSpider with FFuF
```bash
# Discover endpoints with GoSpider
gospider -s http://target.example.com --js -o endpoints.txt

# Fuzz discovered endpoints
cat endpoints.txt | while read url; do
    path=$(echo "$url" | sed 's|http://target.example.com||')
    ffuf -u "http://target.example.com${path}/FUZZ" \
        -w /usr/share/wordlists/dirb/common.txt \
        -fc 404 \
        -o "ffuf_$path.json" \
        -of json
done
```

#### GoSpider with Gobuster
```bash
# Discover links with GoSpider
gospider -s http://target.example.com -o links.txt

# Extract paths for Gobuster
grep -oP 'https?://target\.example\.com\K/[^"?]+' links.txt | sort -u > paths.txt

# Use paths as wordlist for Gobuster
gobuster dir -u http://target.example.com -w paths.txt
```

#### GoSpider with Nuclei
```bash
# Discover endpoints
gospider -s http://target.example.com --js -o endpoints.txt

# Scan endpoints with Nuclei
nuclei -l endpoints.txt -t cves/
```

### Output Processing and Filtering

#### Extracting Only JavaScript Files
```bash
gospider -s http://target.example.com --js -o links.txt
grep '\.js' links.txt | sort -u > js_files.txt
```

#### Extracting API Endpoints
```bash
gospider -s http://target.example.com --js -o links.txt
grep -oP '/api/[^"?]+' links.txt | sort -u > api_endpoints.txt
```

#### Filtering by Status Code
```bash
# Use httpx to check which endpoints are live
gospider -s http://target.example.com -o links.txt
httpx -l links.txt -mc 200 -o live_links.txt -silent
```

---

## Chapter 5: Advanced Usage

### Custom Configuration

#### Custom User-Agent Strings
```bash
# Rotate User-Agents
USER_AGENTS=(
    "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36"
    "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36"
    "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36"
)

for ua in "${USER_AGENTS[@]}"; do
    gospider -s http://target.example.com -a "$ua" -o "links_$ua.txt"
done
```

#### Custom Headers
```bash
# Using proxy with custom headers
gospider -s http://target.example.com \
    --proxy http://proxy:8080 \
    --cookie "session=abc123"
```

### Evasion and Rate Limiting

#### Slow Crawling
```bash
gospider -s http://target.example.com \
    --delay 2 \
    --rate-limit 5 \
    -t 5 \
    --timeout 30
```

#### Using Proxies
```bash
gospider -s http://target.example.com \
    --proxy http://proxy:8080 \
    --random-agent
```

### Integration with Pentest Workflows

#### Comprehensive Web Assessment
```bash
#!/bin/bash
# comprehensive_web_assessment.sh

TARGET=$1
OUTPUT_DIR="web_assessment_$(date +%Y%m%d)"

mkdir -p "$OUTPUT_DIR"

echo "[+] Phase 1: GoSpider - Link Discovery"
gospider -s "$TARGET" \
    --js \
    --source \
    --include-subdomains \
    -o "$OUTPUT_DIR/all_links.txt" \
    -t 20

echo "[+] Phase 2: Extract Endpoints"
grep -oP 'https?://[^\s"]+' "$OUTPUT_DIR/all_links.txt" | sort -u > "$OUTPUT_DIR/endpoints.txt"

echo "[+] Phase 3: Find Subdomains"
grep -oP 'https?://([a-zA-Z0-9-]+\.)+[a-zA-Z]{2,}' "$OUTPUT_DIR/all_links.txt" | \
    sed 's|https\?://||' | sort -u > "$OUTPUT_DIR/subdomains.txt"

echo "[+] Phase 4: Check Live Hosts"
httpx -l "$OUTPUT_DIR/subdomains.txt" -o "$OUTPUT_DIR/live_hosts.txt" -silent

echo "[+] Phase 5: Directory Discovery"
while read host; do
    ffuf -u "http://$host/FUZZ" \
        -w /usr/share/wordlists/dirb/common.txt \
        -fc 404 \
        -o "$OUTPUT_DIR/ffuf_$host.json" \
        -of json \
        -s &
done < "$OUTPUT_DIR/live_hosts.txt"
wait

echo "[+] Phase 6: Parameter Discovery"
grep -oP '[?&][a-zA-Z0-9_]+=' "$OUTPUT_DIR/all_links.txt" | \
    sed 's/[?&]=//' | sort -u > "$OUTPUT_DIR/parameters.txt"

echo "[+] Assessment complete. Results in $OUTPUT_DIR"
```

#### Bug Bounty Workflow
```bash
#!/bin/bash
# bug_bounty_gospider.sh

TARGET=$1

echo "[+] Starting bug bounty web crawling for $TARGET"

# Phase 1: Basic crawl
echo "[*] Phase 1: Basic crawl"
gospider -s "$TARGET" \
    -o basic_links.txt \
    -t 20

# Phase 2: JavaScript analysis
echo "[*] Phase 2: JavaScript analysis"
gospider -s "$TARGET" \
    --js \
    -o js_links.txt \
    -t 10

# Phase 3: Deep crawl
echo "[*] Phase 3: Deep crawl"
gospider -s "$TARGET" \
    -d 5 \
    --include-subdomains \
    -o deep_links.txt \
    -t 10

# Phase 4: Combine results
echo "[*] Phase 4: Combining results"
cat basic_links.txt js_links.txt deep_links.txt | sort -u > all_links.txt
echo "[*] Total unique links: $(wc -l < all_links.txt)"

# Phase 5: Extract endpoints
echo "[*] Phase 5: Extracting endpoints"
grep -oP 'https?://[^\s"]+' all_links.txt | sort -u > endpoints.txt

# Phase 6: Find subdomains
echo "[*] Phase 6: Finding subdomains"
grep -oP 'https?://([a-zA-Z0-9-]+\.)+[a-zA-Z]{2,}' all_links.txt | \
    sed 's|https\?://||' | sort -u > subdomains.txt

echo "[+] Web crawling complete"
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: Web Application Audit

**Objective:** Discover all endpoints and links on a corporate web application.

**Step 1: Initial Crawl**
```bash
gospider -s http://corporate.example.com \
    --js \
    --source \
    -o links.txt \
    -t 20
```

**Step 2: Extract Endpoints**
```bash
grep -oP 'https?://[^\s"]+' links.txt | sort -u > endpoints.txt

# Extract API endpoints
grep -oP '/api/[^"?]+' links.txt | sort -u > api_endpoints.txt

# Extract JavaScript files
grep '\.js' links.txt | sort -u > js_files.txt
```

**Step 3: Analyze JavaScript**
```bash
# Download and analyze JavaScript files
while read js_file; do
    curl -s "$js_file" | grep -oP '/[a-zA-Z0-9_/]+(?:\?|")' >> js_endpoints.txt
done < js_files.txt

sort -u js_endpoints.txt > unique_js_endpoints.txt
```

**Step 4: Directory Discovery**
```bash
# Use discovered paths for directory discovery
cat endpoints.txt | while read url; do
    path=$(echo "$url" | sed 's|http://corporate.example.com||')
    echo "$path"
done | sort -u > paths.txt

gobuster dir -u http://corporate.example.com -w paths.txt -o directories.txt
```

### Scenario 2: CTF Challenge

**Objective:** Find hidden endpoints in a CTF web challenge.

**Step 1: Initial Crawl**
```bash
gospider -s http://ctf.example.com \
    --js \
    --source \
    -o links.txt
```

**Step 2: Deep Crawl**
```bash
gospider -s http://ctf.example.com \
    -d 5 \
    --include-subdomains \
    -o deep_links.txt
```

**Step 3: Analyze JavaScript**
```bash
# Extract endpoints from JavaScript
grep '\.js' links.txt | while read js_file; do
    curl -s "$js_file" | grep -oP '"/[^"]*"' >> hidden_endpoints.txt
done

sort -u hidden_endpoints.txt > unique_endpoints.txt
```

**Step 4: Look for Flags**
```bash
# Search for flag patterns
grep -oP 'flag\{[^}]*\}' links.txt deep_links.txt hidden_endpoints.txt
```

### Scenario 3: Bug Bounty

**Objective:** Discover hidden endpoints for a bug bounty program.

**Step 1: Initial Crawl**
```bash
gospider -s http://target.com \
    --js \
    --source \
    -o basic_links.txt \
    -t 20
```

**Step 2: JavaScript Analysis**
```bash
gospider -s http://target.com \
    --js \
    -o js_links.txt \
    -t 10

# Extract API endpoints from JavaScript
grep '\.js' js_links.txt | while read js_file; do
    curl -s "$js_file" | grep -oP '/api/[^"?]+' >> api_endpoints.txt
done

sort -u api_endpoints.txt > unique_api_endpoints.txt
```

**Step 3: Subdomain Discovery**
```bash
gospider -s http://target.com \
    --include-subdomains \
    -o subdomain_links.txt

grep -oP 'https?://([a-zA-Z0-9-]+\.)+[a-zA-Z]{2,}' subdomain_links.txt | \
    sed 's|https\?://||' | sort -u > subdomains.txt
```

**Step 4: Parameter Discovery**
```bash
grep -oP '[?&][a-zA-Z0-9_]+=' basic_links.txt js_links.txt | \
    sed 's/[?&]=//' | sort -u > parameters.txt
```

---

## Chapter 7: Detection & Defense

### IDS/IPS Detection

**Signature-Based Detection:**
- GoSpider generates distinctive User-Agent strings by default
- Rapid sequential requests to similar paths trigger anomaly detection
- High-volume requests from single IP addresses create detectable patterns
- Specific request patterns (sequential link following) are signature-matchable

**Behavioral Detection:**
- Unusual request patterns (sequential link following)
- High request rates from single source
- Requests to unusual paths or endpoints
- Time-based patterns (consistent intervals between requests)

### Log Indicators

**Apache/Nginx Log Patterns:**
```
# Sequential path requests
[18/Jul/2024:10:15:23 +0000] "GET /about HTTP/1.1" 200
[18/Jul/2024:10:15:23 +0000] "GET /contact HTTP/1.1" 200
[18/Jul/2024:10:15:23 +0000] "GET /login HTTP/1.1" 200

# JavaScript file requests
[18/Jul/2024:10:15:23 +0000] "GET /static/js/app.js HTTP/1.1" 200
[18/Jul/2024:10:15:23 +0000] "GET /static/js/main.js HTTP/1.1" 200

# High request rate
[18/Jul/2024:10:15:23 +0000] "GET /api/users HTTP/1.1" 200
[18/Jul/2024:10:15:23 +0000] "GET /api/users/1 HTTP/1.1" 200
[18/Jul/2024:10:15:23 +0000] "GET /api/users/2 HTTP/1.1" 200
```

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

**User-Agent Blocking:**
```apache
# ModSecurity rule to block GoSpider
SecRule REQUEST_HEADERS:User-Agent "@contains GoSpider" \
    "id:10001,phase:1,deny,status:403,log,msg:'GoSpider detected'"
```

**Fail2Ban Integration:**
```ini
# /etc/fail2ban/filter.d/gospider.conf
[Definition]
failregex = ^<HOST> -.*GoSpider.*$
ignoreregex =
```

```ini
# /etc/fail2ban/jail.d/gospider.conf
[gospider]
enabled = true
filter = gospider
logpath = /var/log/apache2/access.log
maxretry = 10
findtime = 60
bantime = 3600
```

### Blue Team Response

**Incident Response Steps:**
1. **Identify:** Analyze logs for crawling patterns
2. **Contain:** Implement rate limiting or block IPs
3. **Eradicate:** Review exposed sensitive files
4. **Recover:** Secure or remove exposed content
5. **Lessons Learned:** Implement preventive controls

**Monitoring Commands:**
```bash
# Watch for crawling in real-time
tail -f /var/log/apache2/access.log | grep -E "(\.js|\.json|api/)"

# Check for high request rates
awk '{print $1}' /var/log/apache2/access.log | sort | uniq -c | sort -rn | head -20

# Identify crawling patterns
grep "GoSpider" /var/log/apache2/access.log | awk '{print $1}' | sort | uniq -c
```

---

## Chapter 8: Troubleshooting & Resources

### Common Errors

**Error: "Connection timeout"**
```bash
# Solution: Increase timeout
gospider -s http://target.example.com --timeout 60

# Check network connectivity
ping target.example.com
```

**Error: "Too many open files"**
```bash
# Solution: Increase file descriptor limit
ulimit -n 65535

# Or reduce threads
gospider -s http://target.example.com -t 5
```

**Error: "SSL certificate error"**
```bash
# Solution: Skip SSL verification (if available)
gospider -s https://target.example.com --insecure
```

**Error: "Rate limited"**
```bash
# Solution: Add delay between requests
gospider -s http://target.example.com --delay 2 --rate-limit 5
```

### Performance Optimization

**Maximizing Speed:**
```bash
# Use multiple threads
gospider -s http://target.example.com -t 100

# Disable JavaScript analysis if not needed
gospider -s http://target.example.com --no-js

# Limit crawl depth
gospider -s http://target.example.com -d 2
```

**Reducing Memory Usage:**
```bash
# Process targets one at a time
gospider -s http://target1.com -o links1.txt
gospider -s http://target2.com -o links2.txt
```

### FAQ

**Q: How is GoSpider different from Hakrawler?**
A: GoSpider is generally faster due to its Go implementation and supports more features like JavaScript analysis. Hakrawler is simpler and more lightweight.

**Q: Can GoSpider crawl authenticated pages?**
A: Yes, use the `--cookie` flag to pass session cookies for authenticated crawling.

**Q: How do I avoid getting blocked?**
A: Use `--delay`, `--rate-limit`, `--random-agent`, and consider using `--proxy`.

**Q: Can GoSpider extract API endpoints?**
A: Yes, use the `--js` flag to analyze JavaScript files and extract API endpoints.

**Q: How do I crawl multiple URLs?**
A: Use the `-S` flag with a file containing one URL per line.

---

## Resources

### Official Documentation
- [GoSpider GitHub Repository](https://github.com/jaeles-project/gospider)
- [GoSpider Wiki](https://github.com/jaeles-project/gospider/wiki)
- [GoSpider Usage Guide](https://github.com/jaeles-project/gospider/blob/master/README.md)

### Recommended Wordlists
- [SecLists](https://github.com/danielmiessler/SecLists)
- [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings)

### Books
- "Web Application Hacker's Handbook" by Dafydd Stuttard
- "The Hacker Playbook 3" by Peter Kim
- "Penetration Testing" by Georgia Weidman

### Practice Labs
- [HackTheBox](https://www.hackthebox.com/)
- [TryHackMe](https://tryhackme.com/)
- [PortSwigger Web Security Academy](https://portswigger.net/web-security)

### Related Tools
- [Hakrawler](https://github.com/hakluke/hakrawler) - Simple, fast web crawler
- [Katana](https://github.com/projectdiscovery/katana) - Next-generation crawling and spidering framework
- [GAU](https://github.com/lc/gau) - Fetch known URLs from AlienVault OTX, Wayback Machine, and Common Crawl
- [Waybackurls](https://github.com/tomnomnom/waybackurls) - Fetch wayback machine URLs for a domain
