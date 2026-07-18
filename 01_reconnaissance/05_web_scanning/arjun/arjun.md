---
tool_name: "arjun"
display_name: "Arjun"
mitre_tactic: "reconnaissance"
mitre_tactic_id: "TA0043"
mitre_subcategory: "web_scanning"
install_command: "sudo apt install arjun"
version: "1.7.5"
github_repo: "https://github.com/s0md3v/Arjun"
kali_tools_page: "https://www.kali.org/tools/arjun/"
difficulty_level: "beginner-to-intermediate"
requires_root: false
gui_available: false
tags: ["web", "parameter-discovery", "fuzzing", "http"]
related_tools: ["parameth", "x8", "ffuf", "wfuzz"]
---

# Arjun

## Chapter 1: Introduction and Overview

### What is Arjun?

Arjun is an HTTP parameter discovery suite that helps you find hidden GET, POST, JSON, XML, and other parameters in web applications. It automates the process of discovering hidden inputs that developers may have left in their code — parameters that are not linked from the application's UI or source code but still accept input and can be manipulated by an attacker.

### History and Background

- **Created:** 2018 by Sudeep Singh (s0md3v)
- **Maintained:** s0md3v on GitHub
- **License:** Apache 2.0
- **Language:** Python 3
- **Purpose:** HTTP parameter discovery for security testing

Arjun was created to fill a gap in the security tooling landscape — while tools like dirb and gobuster find hidden directories, and tools like sqlmap exploit SQL injection, there was no dedicated tool for discovering the hidden HTTP parameters that serve as the entry points for many web vulnerabilities. Arjun uses various techniques including wordlist-based fuzzing, comparison analysis, and heuristic methods to uncover these parameters.

### When to Use This Tool

- **Penetration testing:** Discover hidden parameters before testing for injection vulnerabilities
- **Bug bounty hunting:** Find undocumented API parameters that may expose sensitive functionality
- **Security audits:** Map the complete attack surface of a web application
- **Web application assessments:** Find debug parameters, hidden form fields, and API endpoints
- **API security testing:** Discover undocumented query parameters and body fields

### When NOT to Use It

- On targets without explicit written authorization
- Against production systems with rate-limiting that may block your IP
- When you need to fuzz parameter values (use ffuf or wfuzz for that)
- For non-HTTP targets (Arjun is HTTP-specific)

### Legal and Ethical Considerations

- Always obtain written permission before scanning targets you do not own
- Be aware that parameter discovery generates significant traffic and may trigger WAF/IDS alerts
- Document all findings and report them responsibly
- Respect rate limits and scope boundaries defined in your engagement
- Some jurisdictions consider unauthorized web scanning a criminal offense

### Key Concepts and Terminology

- **Parameter:** A variable sent to the server via GET query string, POST body, JSON body, XML body, or headers
- **Wordlist:** A file containing potential parameter names used for fuzzing
- **Comparison Mode:** Arjun compares responses to identify parameters by detecting changes in response behavior
- **Injection Point:** A location where user input enters the application (URL, headers, body)
- **Content-Type:** The media type of the request body (application/json, application/xml, etc.)
- **Heuristics:** Arjun uses statistical analysis of responses to detect parameters without wordlists

---

## Chapter 2: Installation and Setup

### System Requirements

- **Python:** 3.6 or higher
- **RAM:** 256 MB minimum (512 MB recommended)
- **Disk Space:** ~20 MB for Arjun + wordlists
- **Network:** Internet connection to reach target
- **Privileges:** User-level (no root required)

### Installation Methods

#### Method 1: APT (Recommended)
```bash
sudo apt update
sudo apt install arjun
```

#### Method 2: Pip
```bash
pip3 install arjun
```

#### Method 3: From Source
```bash
git clone https://github.com/s0md3v/Arjun.git
cd Arjun
pip3 install -r requirements.txt
python3 arjun.py
```

#### Method 4: Docker
```bash
docker run --rm -it ghcr.io/s0md3v/arjun:latest arjun -u https://target.com
```

### Configuration Files and Environment Variables

Arjun does not use a configuration file by default. All options are passed via command-line arguments. However, Arjun stores its wordlists in its installation directory:

- **Default wordlist:** `arjun/db/params.txt` (included with Arjun)
- **Large wordlist:** `arjun/db/params-large.txt` (extended parameter list)

### Verification Commands

```bash
# Verify installation
arjun --version
# Expected output: Arjun v1.7.5

# Verify it runs
arjun -h
# Should display the help menu

# Verify wordlists exist
ls -la $(which arjun | xargs dirname)/../share/arjun/db/
# Or from source:
ls -la db/
```

---

## Chapter 3: Basic Usage

### First Scan

```bash
arjun -u https://target.com
```

**Sample Output:**
```
Arjun v1.7.5

[i] Heuristics have been loaded
[i] Starting heuristic scan on https://target.com
[i] Found 3 parameters using heuristics: debug, lang, theme
[i] Starting wordlist-based scan on https://target.com
[+] Found 5 parameters:
    debug
    lang
    theme
    redirect
    callback
```

### Essential Commands

#### 1. Basic Parameter Discovery
```bash
arjun -u https://target.com
```
**Explanation:** Discovers hidden parameters using heuristics and the default wordlist.

#### 2. GET Parameter Discovery
```bash
arjun -u https://target.com -m GET
```
**Explanation:** Specifically discovers GET (query string) parameters.

#### 3. POST Parameter Discovery
```bash
arjun -u https://target.com -m POST -d "test=test"
```
**Explanation:** Discovers POST body parameters. The `-d` flag provides sample POST data.

#### 4. JSON Parameter Discovery
```bash
arjun -u https://target.com -m JSON -d '{"test":"test"}'
```
**Explanation:** Discovers JSON body parameters.

#### 5. Custom Wordlist
```bash
arjun -u https://target.com -w /usr/share/wordlists/custom_params.txt
```
**Explanation:** Uses a custom wordlist for parameter name fuzzing.

#### 6. Custom Headers
```bash
arjun -u https://target.com -H "Authorization: Bearer token123" -H "X-Api-Key: key456"
```
**Explanation:** Adds custom headers to all requests.

#### 7. Custom Cookies
```bash
arjun -u https://target.com --cookies "session=abc123; user=admin"
```
**Explanation:** Includes cookies in every request.

#### 8. Output to JSON File
```bash
arjun -u https://target.com -o results.json
```
**Explanation:** Saves discovered parameters to a JSON file.

#### 9. Scan Multiple URLs
```bash
arjun -i urls.txt
```
**Explanation:** Reads URLs from a file and scans each one.

#### 10. Verbose Output
```bash
arjun -u https://target.com -v
```
**Explanation:** Shows detailed output including request/response details.

#### 11. Thread Control
```bash
arjun -u https://target.com -t 20
```
**Explanation:** Sets the number of concurrent threads to 20.

#### 12. Timeout Configuration
```bash
arjun -u https://target.com --timeout 10
```
**Explanation:** Sets a 10-second timeout for each request.

### COMPLETE Flag Reference

#### Target Options
| Flag | Description | Example |
|------|-------------|---------|
| `-u` | Single target URL | `arjun -u https://target.com` |
| `-i` | Input file (list of URLs) | `arjun -i urls.txt` |
| `-m` | HTTP method (GET/POST/JSON/XML) | `arjun -u URL -m POST` |
| `-d` | POST/JSON/XML data | `arjun -u URL -d "key=value"` |
| `-w` | Custom wordlist | `arjun -u URL -w wordlist.txt` |

#### Header and Cookie Options
| Flag | Description | Example |
|------|-------------|---------|
| `-H` | Custom header | `arjun -u URL -H "Auth: token"` |
| `-b` | Custom cookies | `arjun -u URL -b "session=abc"` |
| `--cookies` | Cookies (alternate) | `arjun -u URL --cookies "k=v"` |

#### Scan Control Options
| Flag | Description | Example |
|------|-------------|---------|
| `-t` | Thread count | `arjun -u URL -t 30` |
| `--timeout` | Request timeout (seconds) | `arjun -u URL --timeout 15` |
| `--delay` | Delay between requests (ms) | `arjun -u URL --delay 1000` |
| `-c` | Case sensitivity | `arjun -u URL -c` |
| `--skip` | Skip parameters (comma-separated) | `arjun -u URL --skip "id,user"` |
| `--stable` | Prefer stability over speed | `arjun -u URL --stable` |

#### Comparison Mode Options
| Flag | Description | Example |
|------|-------------|---------|
| `--compare` | Enable comparison mode | `arjun -u URL --compare` |
| `--include` | Include parameters in comparison | `arjun -u URL --compare --include "lang"` |

#### Output Options
| Flag | Description | Example |
|------|-------------|---------|
| `-o` | Output file (JSON) | `arjun -u URL -o results.json` |
| `--json` | Output as JSON | `arjun -u URL --json` |
| `-v` | Verbose output | `arjun -u URL -v` |
| `--no-logging` | Disable logging | `arjun -u URL --no-logging` |

#### Network Options
| Flag | Description | Example |
|------|-------------|---------|
| `--proxy` | Use HTTP proxy | `arjun -u URL --proxy http://127.0.0.1:8080` |
| `--tor` | Route through Tor | `arjun -u URL --tor` |

#### Miscellaneous
| Flag | Description | Example |
|------|-------------|---------|
| `-h` / `--help` | Show help | `arjun -h` |
| `--version` | Show version | `arjun --version` |

### Output Format Explanation

Arjun outputs results in two formats:

**Terminal Output:**
```
[+] Found 5 parameters:
    debug
    lang
    theme
    redirect
    callback
```

**JSON Output (`-o results.json`):**
```json
{
  "https://target.com": {
    "GET": ["debug", "lang", "theme", "redirect", "callback"]
  }
}
```

---

## Chapter 4: Intermediate Usage

### Bash Scripting and Automation

#### Batch URL Scanning
```bash
#!/bin/bash
# arjun_batch.sh - Scan multiple targets with different methods

TARGETS_FILE="targets.txt"
OUTPUT_DIR="arjun_results"
mkdir -p $OUTPUT_DIR

while IFS= read -r url; do
    echo "[*] Scanning $url"

    # GET parameter discovery
    arjun -u "$url" -m GET -o "$OUTPUT_DIR/$(echo $url | tr '/:' '__')_get.json" 2>/dev/null

    # POST parameter discovery
    arjun -u "$url" -m POST -d "test=test" -o "$OUTPUT_DIR/$(echo $url | tr '/:' '__')_post.json" 2>/dev/null

    # JSON parameter discovery
    arjun -u "$url" -m JSON -d '{"test":"test"}' -o "$OUTPUT_DIR/$(echo $url | tr '/:' '__')_json.json" 2>/dev/null

done < "$TARGETS_FILE"

echo "[+] All scans complete. Results in $OUTPUT_DIR"
```

#### Automated Parameter Discovery Pipeline
```bash
#!/bin/bash
# Auto-discover parameters across multiple endpoints

ENDPOINTS=(
    "https://target.com/"
    "https://target.com/api"
    "https://target.com/search"
    "https://target.com/login"
)

for endpoint in "${ENDPOINTS[@]}"; do
    echo "==========================================="
    echo "[*] Scanning: $endpoint"
    echo "==========================================="

    arjun -u "$endpoint" --stable -t 10 --timeout 10 -v

    echo ""
done
```

### Python Scripting Examples

#### Custom Arjun Wrapper
```python
#!/usr/bin/env python3
import subprocess
import json
import sys

def run_arjun(url, method="GET", wordlist=None, threads=10):
    """Run arjun and return discovered parameters."""
    cmd = [
        "arjun", "-u", url,
        "-m", method,
        "-t", str(threads),
        "-v"
    ]

    if wordlist:
        cmd.extend(["-w", wordlist])

    result = subprocess.run(cmd, capture_output=True, text=True)

    # Parse JSON output if available
    try:
        return json.loads(result.stdout)
    except json.JSONDecodeError:
        return {"error": result.stderr}

def parameter_audit(url_list):
    """Audit multiple URLs for hidden parameters."""
    all_params = {}

    for url in url_list:
        print(f"[*] Scanning {url}")
        result = run_arjun(url, threads=20)

        if url in result:
            params = result[url]
            all_params[url] = params

            for method, param_list in params.items():
                print(f"  [+] {method}: {', '.join(param_list)}")

    return all_params

if __name__ == "__main__":
    urls = [
        "https://target.com/",
        "https://target.com/api/v1/users",
        "https://target.com/search"
    ]

    results = parameter_audit(urls)

    # Save results
    with open("arjun_audit.json", "w") as f:
        json.dump(results, f, indent=2)

    print(f"\n[+] Results saved to arjun_audit.json")
```

#### Response Comparison Script
```python
#!/usr/bin/env python3
import requests
import json

def compare_responses(url, param_name, param_value="test123"):
    """Compare response with and without a parameter to verify it's real."""
    headers = {"User-Agent": "Mozilla/5.0 (compatible; Arjun Test)"}

    # Without parameter
    r1 = requests.get(url, headers=headers, timeout=10)

    # With parameter
    params = {param_name: param_value}
    r2 = requests.get(url, params=params, headers=headers, timeout=10)

    return {
        "status_changed": r1.status_code != r2.status_code,
        "length_changed": len(r1.text) != len(r2.text),
        "status_without": r1.status_code,
        "status_with": r2.status_code,
        "length_without": len(r1.text),
        "length_with": len(r2.text)
    }
```

### Tool Chaining

#### Arjun + sqlmap Pipeline
```bash
# Step 1: Discover parameters
arjun -u https://target.com/page -o params.json

# Step 2: Extract parameters and test for SQL injection
cat params.json | python3 -c "
import sys, json
data = json.load(sys.stdin)
for url, methods in data.items():
    for method, params in methods.items():
        for param in params:
            print(f'{method} {url} {param}')
" | while read method url param; do
    echo "[*] Testing $param for SQLi on $url"
    sqlmap -u "${url}?${param}=test" --batch --random-agent --level 2
done
```

#### Arjun + nuclei Integration
```bash
# Discover parameters, then scan endpoints with nuclei
arjun -u https://target.com -o params.json

# Extract URLs and scan with nuclei
cat params.json | python3 -c "
import sys, json
data = json.load(sys.stdin)
for url, methods in data.items():
    for method, params in methods.items():
        for param in params:
            print(f'{url}?{param}=FUZZ')
" > endpoints.txt

nuclei -l endpoints.txt -t nuclei-templates/ -o nuclei_results.txt
```

### Output Processing and Filtering

#### Extract Parameters for Burp Suite
```bash
# Convert Arjun output to Burp-compatible format
arjun -u https://target.com -o results.json

python3 -c "
import json
with open('results.json') as f:
    data = json.load(f)
for url, methods in data.items():
    for method, params in methods.items():
        for param in params:
            print(f'{method} {url} {param}')
" > burp_params.txt

# Import into Burp Suite Intruder
```

#### Merge Multiple Arjun Results
```bash
# Merge multiple JSON files
python3 -c "
import json, glob
merged = {}
for f in glob.glob('*.json'):
    with open(f) as fh:
        data = json.load(fh)
        for url, methods in data.items():
            if url not in merged:
                merged[url] = {}
            for method, params in methods.items():
                if method not in merged[url]:
                    merged[url][method] = []
                merged[url][method].extend(params)
                merged[url][method] = list(set(merged[url][method]))
with open('merged_results.json', 'w') as f:
    json.dump(merged, f, indent=2)
print('Merged results saved to merged_results.json')
"
```

---

## Chapter 5: Advanced Usage

### Custom Wordlists and Configurations

#### Building a Custom Parameter Wordlist
```bash
# Extract parameter names from JavaScript files
curl -s https://target.com/static/app.js | \
  grep -oP '(?<=\?|&|data\[["\x27])\w+(?=["\x27])' | \
  sort -u > js_params.txt

# Combine with default wordlist
cat /usr/share/wordlists/arjun/params.txt js_params.txt | sort -u > custom_wordlist.txt

# Use the custom wordlist
arjun -u https://target.com -w custom_wordlist.txt
```

#### Technology-Specific Wordlists
```bash
# WordPress parameters
cat > wordpress_params.txt << 'EOF'
action
author
cat
comment
content
date
email
orderby
paged
page_id
post
s
search
tag
type
wpnonce
EOF

arjun -u https://target.com/ -w wordpress_params.txt
```

### Evasion Techniques

#### Rotating User-Agents
```bash
#!/bin/bash
# Rotate user agents to avoid fingerprinting

AGENTS=(
    "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36"
    "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36"
    "Mozilla/5.0 (X11; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/115.0"
    "Mozilla/5.0 (iPhone; CPU iPhone OS 17_0 like Mac OS X)"
)

for agent in "${AGENTS[@]}"; do
    echo "[*] Using agent: $agent"
    arjun -u https://target.com -H "User-Agent: $agent" --delay 500
done
```

#### Proxy Chaining
```bash
# Route through Burp Suite
arjun -u https://target.com --proxy http://127.0.0.1:8080

# Route through Tor
arjun -u https://target.com --tor

# Route through custom proxy
arjun -u https://target.com --proxy http://proxy.corp.com:3128
```

### Integration with Pentest Workflows

#### Full Recon Workflow
```bash
#!/bin/bash
# Full web application parameter discovery workflow

TARGET="https://target.com"

echo "[Phase 1] JavaScript parameter extraction..."
curl -s "$TARGET" | grep -oP 'src="[^"]*\.js"' | sed 's/src="//;s/"//' | \
  while read js; do
    curl -s "${TARGET}${js}" 2>/dev/null
  done | grep -oP '\w+(?=\s*[=:])' | sort -u > js_discovered.txt

echo "[Phase 2] Arjun parameter fuzzing..."
arjun -u "$TARGET" -m GET -t 20 --stable -o arjun_get.json
arjun -u "$TARGET" -m POST -d "test=test" -t 20 --stable -o arjun_post.json
arjun -u "$TARGET" -m JSON -d '{"test":"test"}' -t 20 --stable -o arjun_json.json

echo "[Phase 3] Combining results..."
cat js_discovered.txt | while read param; do
    echo "[*] Testing $param"
done

echo "[Phase 4] Validation..."
# Merge and deduplicate all findings
python3 -c "
import json, glob
all_params = set()
for f in glob.glob('arjun_*.json'):
    with open(f) as fh:
        data = json.load(fh)
    for url, methods in data.items():
        for method, params in methods.items():
            all_params.update(params)
with open('all_params.txt', 'w') as f:
    for p in sorted(all_params):
        f.write(p + '\n')
print(f'Found {len(all_params)} unique parameters')
"

echo "[+] Workflow complete"
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: Web Application Security Audit

**Objective:** Discover all hidden parameters in a web application before testing for injection vulnerabilities.

**Step-by-Step:**

```bash
# Step 1: Initial reconnaissance
echo "[*] Phase 1: Application mapping"
nmap -sV -p 80,443 target.com

# Step 2: Spider the application
gospider -s https://target.com -d 2 -c 10 -t 5 -q | grep -oP 'https?://[^\s]+' | sort -u > urls.txt

# Step 3: Parameter discovery on main pages
echo "[*] Phase 2: Parameter discovery"
while IFS= read -r url; do
    echo "[*] Scanning $url"
    arjun -u "$url" --stable -t 10 -o "arjun_$(echo $url | md5sum | cut -d' ' -f1).json" 2>/dev/null
done < urls.txt

# Step 4: Merge results
echo "[*] Phase 3: Merging results"
python3 -c "
import json, glob
all_params = {}
for f in glob.glob('arjun_*.json'):
    with open(f) as fh:
        data = json.load(fh)
    all_params.update(data)
with open('all_parameters.json', 'w') as f:
    json.dump(all_params, f, indent=2)
total = sum(len(p) for m in all_params.values() for p in m.values())
print(f'Discovered {total} parameters across {len(all_params)} URLs')
"

# Step 5: Test for injection vulnerabilities
echo "[*] Phase 4: Vulnerability testing"
cat all_parameters.json | python3 -c "
import sys, json
data = json.load(sys.stdin)
for url, methods in data.items():
    for method, params in methods.items():
        for param in params:
            print(f'{url}?{param}=test')
" > test_urls.txt

# Test with sqlmap
while IFS= read -r url; do
    sqlmap -u "$url" --batch --random-agent --level 2 --risk 1 2>/dev/null
done < test_urls.txt
```

### Scenario 2: CTF Challenge Walkthrough

**Objective:** Find the hidden parameter that gives the flag.

```bash
# Step 1: Run Arjun on the target
arjun -u http://target.com:8080/ -v

# Output might show:
# [+] Found 3 parameters: id, page, debug

# Step 2: Test each parameter
curl "http://target.com:8080/?id=1"
curl "http://target.com:8080/?page=home"
curl "http://target.com:8080/?debug=1"

# Step 3: The debug parameter might reveal something
curl "http://target.com:8080/?debug=true"
# Returns: {"flag": "CTF{hidden_param_discovered}"}
```

### Scenario 3: Bug Bounty Methodology

**Objective:** Maximize findings through systematic parameter discovery.

```bash
# Step 1: Passive parameter discovery
echo "[*] Checking JavaScript for parameters"
curl -s https://target.com | grep -oP 'src="[^"]*\.js"' | \
  while read js; do
    curl -s "https://target.com${js}" 2>/dev/null
  done | grep -oP '(?<=\?|&|["\x27])\w+(?=["\x27]\s*[=:])' | sort -u > js_params.txt

# Step 2: Active discovery with Arjun
arjun -u https://target.com/ -m GET -t 30 -o get_params.json
arjun -u https://target.com/ -m POST -d "test=test" -o post_params.json
arjun -u https://target.com/api/ -m JSON -d '{"test":"test"}' -o json_params.json

# Step 3: Compare mode for verification
arjun -u https://target.com/ --compare -o verified_params.json

# Step 4: Create a parameter wordlist from findings
cat js_params.txt | while read param; do
    echo "$param"
done > custom_wordlist.txt

# Step 5: Re-scan with custom wordlist for deeper discovery
arjun -u https://target.com/ -w custom_wordlist.txt -o final_params.json

# Step 6: Report
echo "[+] Parameter Discovery Summary"
echo "=============================="
for f in *.json; do
    echo "File: $f"
    cat "$f" | python3 -c "import sys,json; d=json.load(sys.stdin); print(json.dumps(d,indent=2))"
done
```

---

## Chapter 7: Detection and Defense

### How Defenders Detect This Tool

#### Network Indicators
```
# Arjun request patterns:
# - High volume of requests to same endpoint
# - Parameter names from common wordlists appearing in query strings
# - Consistent request timing
# - Requests with minimal response size variation
# - User-Agent may include "Arjun" or default Python requests UA
```

#### Log Analysis
```bash
# Check for parameter fuzzing patterns
awk '{print $7}' access.log | grep '?' | \
  awk -F'?' '{print $1}' | sort | uniq -c | sort -rn | head -20

# Detect high request rates from single IP
awk '{print $1}' access.log | sort | uniq -c | sort -rn | head -10

# Look for common parameter names appearing frequently
grep -oP '\?.*' access.log | \
  grep -oP '\w+(?==)' | sort | uniq -c | sort -rn | head -30
```

### IDS/IPS Signatures

```
# Snort/Suricata rules
alert http any any -> $HOME_NET any (msg:"Parameter Fuzzing Detected";
  content:"?"; http_uri; threshold:type both,track by_src,count 50,seconds 10;
  sid:2000001; rev:1;)

alert http any any -> $HOME_NET any (msg:"Arjun User-Agent Detected";
  content:"User-Agent|3a| Arjun"; http_header;
  sid:2000002; rev:1;)
```

### Mitigation Strategies

#### Rate Limiting (nginx)
```nginx
# Limit requests per IP
limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;

location /api/ {
    limit_req zone=api burst=20 nodelay;
    limit_req_status 429;
}
```

#### WAF Rules
```bash
# ModSecurity rule to block parameter fuzzing
SecRule ARGS_NAMES "@rx \w{20,}" \
    "id:1001,phase:1,block,msg:'Possible parameter fuzzing detected'"

# Block known scanner User-Agents
SecRule REQUEST_HEADERS:User-Agent "@rx Arjun|parameth" \
    "id:1002,phase:1,block,msg:'Scanner User-Agent detected'"
```

### Blue Team Response Playbook

1. **Detect** - Monitor for high-volume requests with varying parameters
2. **Analyze** - Check logs for fuzzing patterns (many 400/404 responses)
3. **Block** - Apply rate limiting or block scanner IP at firewall
4. **Review** - Check if any discovered parameters lead to actual vulnerabilities
5. **Remediate** - Remove or secure any vulnerable parameters found
6. **Report** - Document the incident and response actions

---

## Chapter 8: Troubleshooting

### Common Errors and Solutions

#### Error: "Connection refused"
**Cause:** Target is not responding or port is closed.
```bash
# Verify the target is accessible
curl -I https://target.com
# If using non-standard port
arjun -u https://target.com:8443/
```

#### Error: "SSLError: CERTIFICATE_VERIFY_FAILED"
**Cause:** SSL certificate verification failure.
```bash
# Use --verify flag or disable SSL verification
arjun -u https://target.com --verify no
# Or install certificates
pip3 install certifi
```

#### Error: "Too many requests"
**Cause:** Rate limiting by the target server.
```bash
# Reduce threads and add delay
arjun -u https://target.com -t 5 --delay 2000
```

#### Error: "Timeout"
**Cause:** Target is slow to respond.
```bash
# Increase timeout
arjun -u https://target.com --timeout 30
```

### Performance Optimization

```bash
# Fast scan (less thorough)
arjun -u https://target.com -t 50 --timeout 5

# Thorough scan (slower)
arjun -u https://target.com -t 10 --timeout 30 --stable

# Balanced
arjun -u https://target.com -t 20 --timeout 10 --stable
```

### FAQ

**Q: Is Arjun legal to use?**
A: Yes, Arjun is a legitimate security testing tool. However, using it without authorization against systems you don't own is illegal in most jurisdictions.

**Q: How do I update Arjun?**
A: `sudo apt update && sudo apt upgrade arjun` or `pip3 install --upgrade arjun`

**Q: What's the difference between Arjun and parameth?**
A: Arjun is actively maintained and supports more injection types (JSON, XML). parameth is older and focuses primarily on GET/POST parameters.

**Q: Can Arjun bypass WAFs?**
A: Arjun includes some evasion capabilities, but for serious WAF bypass, use it in conjunction with a proxy (like Burp Suite) and custom headers.

**Q: Why does Arjun find parameters that don't exist?**
A: False positives can occur. Use `--compare` mode to verify findings by comparing responses with and without the parameter.

---

## Resources

### Official Documentation
- [Arjun GitHub Repository](https://github.com/s0md3v/Arjun)
- [Arjun Wiki](https://github.com/s0md3v/Arjun/wiki)
- [Arjun Paper/Writeups](https://github.com/s0md3v/Arjun#readme)

### Tutorials
- [Arjun Usage Guide - GitHub](https://github.com/s0md3v/Arjun#usage)
- [Parameter Discovery in Bug Bounty - HackerOne](https://www.hackerone.com/)
- [Hidden Parameters - HackTricks](https://book.hacktricks.xyz/)

### Books
- "The Web Application Hacker's Handbook" by Dafydd Stuttard
- "Bug Bounty Bootcamp" by Vickie Li
- "Real-World Bug Hunting" by Peter Yaworski

### Practice Labs
- [DVWA](https://github.com/digininja/DVWA) - Damn Vulnerable Web Application
- [HackTheBox](https://www.hackthebox.com/) - CTF platform
- [TryHackMe](https://tryhackme.com/) - Learning platform
- [PortSwigger Web Security Academy](https://portswigger.net/web-security) - Free labs

### Related Tools
- **parameth:** Parameter discovery tool (similar purpose)
- **x8:** Hidden parameters discovery suite
- **ffuf:** Web fuzzer for parameter values
- **wfuzz:** Web application fuzzer
- **subfinder:** Subdomain discovery (for broader recon)
