---
tool_name: "feroxbuster"
display_name: "Feroxbuster"
mitre_tactic: "reconnaissance"
mitre_tactic_id: "TA0043"
mitre_subcategory: "web_scanning"
install_command: "sudo apt install feroxbuster"
version: "2.11.0"
github_repo: "https://github.com/epi052/feroxbuster"
kali_tools_page: "https://www.kali.org/tools/feroxbuster/"
difficulty_level: "beginner-to-intermediate"
requires_root: false
gui_available: false
tags: ["web", "directory", "brute-force", "recursive", "fast"]
related_tools: ["gobuster", "ffuf", "dirsearch", "dirb"]
---

# Feroxbuster

## Chapter 1: Introduction & Overview

### What is Feroxbuster?

Feroxbuster is a fast, simple, recursive content discovery tool written in Rust. It's designed for speed and ease of use, making it one of the most popular directory brute-forcing tools in the modern penetration testing and bug bounty community. Feroxbuster combines high performance with advanced features like auto-calibration, recursive scanning, and extensive filtering options.

### History & Background

- **Created:** 2019 by epi052
- **Maintained:** epi052 on GitHub
- **License:** MIT License
- **Language:** Rust
- **Purpose:** Recursive content discovery and directory brute-forcing

Feroxbuster was created to address the need for a fast, reliable, and feature-rich content discovery tool. Written in Rust, it achieves near-native performance while maintaining a clean, user-friendly interface. It quickly gained popularity in the bug bounty and pentesting communities due to its speed, auto-calibration feature, and built-in recursion support.

### When to Use This Tool

- **Web reconnaissance:** Find hidden directories and files on web servers
- **Penetration testing:** Discover files and folders that aren't linked from the application
- **Bug bounty hunting:** Find hidden endpoints, API routes, and admin panels
- **Security audits:** Map the complete web application structure
- **CTF challenges:** Discover hidden flags and paths
- **API testing:** Discover undocumented API endpoints and routes

### When NOT to Use It

- Against targets without written authorization
- When you need parameter discovery (use arjun for that)
- When you need subdomain discovery (use subfinder for that)
- On extremely slow networks (reduce thread count significantly)

### Legal and Ethical Considerations

- Always obtain written permission before scanning targets you do not own
- Feroxbuster generates significant HTTP traffic and may trigger IDS/WAF alerts
- Be aware that directory brute-forcing may cause log noise
- Respect rate limits and scope boundaries defined in your engagement
- Document all findings and report them responsibly
- Some organizations consider directory brute-forcing as hostile activity

### Key Concepts and Terminology

- **Recursion:** Automatically scanning found directories for additional content
- **Auto-calibrate:** Automatically detecting and filtering common responses (like 404 pages)
- **Wordlist:** A file containing potential directory and file names
- **Extension:** File extensions to test (e.g., .php, .html, .txt)
- **Thread:** Concurrent HTTP request
- **Rate Limit:** Maximum number of requests per second
- **Filter:** Exclude results based on various criteria (status codes, size, lines, words)
- **Scan Limit:** Maximum number of recursive scans to run simultaneously

---

## Chapter 2: Installation & Setup

### System Requirements

- **OS:** Linux, macOS, or Windows
- **RAM:** 128 MB minimum (512 MB recommended)
- **Disk Space:** ~15 MB for Feroxbuster binary
- **Network:** Internet connection to reach target
- **Privileges:** User-level (no root required)

### Installation Methods

#### Method 1: APT (Recommended)
```bash
sudo apt update
sudo apt install feroxbuster
```

#### Method 2: Cargo (Rust Package Manager)
```bash
cargo install feroxbuster
```

#### Method 3: Binary Download
```bash
# Download latest release
wget https://github.com/epi052/feroxbuster/releases/latest/download/x86_64-linux-feroxbuster.tar.gz
tar -xzf x86_64-linux-feroxbuster.tar.gz
chmod +x feroxbuster
sudo mv feroxbuster /usr/local/bin/
```

#### Method 4: Docker
```bash
docker run --rm -it ghcr.io/epi052/feroxbuster:latest -u https://target.com
```

#### Method 5: From Source (requires Rust)
```bash
git clone https://github.com/epi052/feroxbuster.git
cd feroxbuster
cargo build --release
sudo cp target/release/feroxbuster /usr/local/bin/
```

### Configuration Files and Environment Variables

Feroxbuster uses a TOML configuration file:

- **Default config:** `~/.config/feroxbuster/ferox-config.toml`
- **Custom config:** Specify with `--config` flag

```toml
# ~/.config/feroxbuster/ferox-config.toml
threads = 50
wordlist = "/usr/share/wordlists/dirb/common.txt"
timeout = 10
depth = 3
scan-limit = 5
parallel-scan = 5
rate-limit = 100
```

### Verification Commands

```bash
# Verify installation
feroxbuster --version
# Expected output: feroxbuster 2.11.0

# Quick test
feroxbuster -u https://example.com -w /usr/share/wordlists/dirb/common.txt
```

---

## Chapter 3: Basic Usage

### First Scan

```bash
feroxbuster -u https://target.com
```

**Sample Output:**
```
 ___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___

by epi052

____________________________________________________________
|        Code       |      URL      | Redirected? |          |
|___________________|_______________|_____________|__________|
|      200          | https://target.com/admin     |     No   |
|      301          | https://target.com/backup    |    Yes   |
|      200          | https://target.com/config    |     No   |
|      403          | https://target.com/dashboard |     No   |
|      200          | https://target.com/login     |     No   |
|      200          | https://target.com/robots.txt|     No   |
|___________________|_______________|_____________|__________|

[+] Found 6 URLs
[+] Tests completed in 45.23 seconds
[+] Requests: 4612 | Errors: 0 | Expected time: 45.23s
```

### Essential Commands

#### 1. Basic Scan with Wordlist
```bash
feroxbuster -u https://target.com -w /usr/share/wordlists/dirb/common.txt
```
**Explanation:** Scans the target using the common wordlist.

#### 2. Recursive Scan
```bash
feroxbuster -u https://target.com -w /usr/share/wordlists/dirb/common.txt -r
```
**Explanation:** Recursively scans found directories for additional content.

#### 3. Recursive with Depth Limit
```bash
feroxbuster -u https://target.com -w wordlist.txt -r --depth 3
```
**Explanation:** Recursively scans up to 3 levels deep.

#### 4. Custom Extensions
```bash
feroxbuster -u https://target.com -w wordlist.txt -x php,html,txt
```
**Explanation:** Tests for files with specific extensions.

#### 5. Exclude Extensions
```bash
feroxbuster -u https://target.com -w wordlist.txt -X pdf,png,jpg
```
**Explanation:** Excludes specific extensions from testing.

#### 6. Custom Headers
```bash
feroxbuster -u https://target.com -H "Authorization: Bearer token123"
```
**Explanation:** Adds custom headers to all requests.

#### 7. Custom Cookies
```bash
feroxbuster -u https://target.com -b "session=abc123; user=admin"
```
**Explanation:** Includes cookies in all requests.

#### 8. Auto-Calibrate
```bash
feroxbuster -u https://target.com -C
```
**Explanation:** Automatically detects and filters common responses like custom 404 pages.

#### 9. Rate Limiting
```bash
feroxbuster -u https://target.com --rate-limit 100
```
**Explanation:** Limits to 100 requests per second.

#### 10. Output to File
```bash
feroxbuster -u https://target.com -o results.txt
```
**Explanation:** Saves results to a text file.

#### 11. JSON Output
```bash
feroxbuster -u https://target.com -O results.json
```
**Explanation:** Saves results in JSON format.

#### 12. Silent Mode
```bash
feroxbuster -u https://target.com --silent
```
**Explanation:** Outputs only discovered URLs, one per line.

### COMPLETE Flag Reference

#### Target Options
| Flag | Description | Example |
|------|-------------|---------|
| `-u` | Target URL | `feroxbuster -u https://target.com` |
| `-w` | Wordlist | `feroxbuster -u URL -w wordlist.txt` |
| `-x` | Extensions | `feroxbuster -u URL -x php,html,txt` |
| `-X` | Exclude extensions | `feroxbuster -u URL -X pdf,png` |
| `-H` | Custom header | `feroxbuster -u URL -H "Auth: token"` |
| `-b` | Cookie | `feroxbuster -u URL -b "session=abc"` |
| `-d` | POST data | `feroxbuster -u URL -d "FUZZ=test"` |
| `-m` | HTTP method | `feroxbuster -u URL -m POST` |
| `-k` | No TLS verify | `feroxbuster -u URL -k` |
| `--url` | Alternate URL flag | `feroxbuster --url https://target.com` |

#### Scan Options
| Flag | Description | Example |
|------|-------------|---------|
| `-t` | Thread count | `feroxbuster -u URL -t 50` |
| `--rate-limit` | Rate limit (req/s) | `feroxbuster -u URL --rate-limit 100` |
| `--timeout` | Request timeout (s) | `feroxbuster -u URL --timeout 10` |
| `--delay` | Delay between requests | `feroxbuster -u URL --delay 100ms` |
| `-c` | Concurrency (per scan) | `feroxbuster -u URL -c 10` |
| `--scan-limit` | Max concurrent scans | `feroxbuster -u URL -r --scan-limit 5` |
| `--parallel-scan` | Parallel scans | `feroxbuster -u URL -r --parallel-scan 10` |
| `--no-recursion` | Disable recursion | `feroxbuster -u URL --no-recursion` |
| `--force-recursion` | Force recursion | `feroxbuster -u URL --force-recursion` |
| `--crawl` | Crawl for links | `feroxbuster -u URL --crawl` |

#### Recursion Options
| Flag | Description | Example |
|------|-------------|---------|
| `-r` | Enable recursion | `feroxbuster -u URL -r` |
| `--depth` | Recursion depth | `feroxbuster -u URL -r --depth 3` |
| `--scan-limit` | Max concurrent scans | `feroxbuster -u URL -r --scan-limit 5` |
| `--parallel-scan` | Parallel scans | `feroxbuster -u URL -r --parallel-scan 10` |

#### Filter Options
| Flag | Description | Example |
|------|-------------|---------|
| `-s` | Status codes (include) | `feroxbuster -u URL -s 200,301,302` |
| `-S` | Status codes (exclude) | `feroxbuster -u URL -S 404` |
| `-W` | Word count filter | `feroxbuster -u URL -W 42` |
| `-L` | Line count filter | `feroxbuster -u URL -L 10-100` |
| `--size` | Response size filter | `feroxbuster -u URL --size 500` |
| `-C` | Auto-calibrate | `feroxbuster -u URL -C` |
| `--auto-tune` | Auto-tune filters | `feroxbuster -u URL --auto-tune` |
| `--smart-filter` | Smart filter | `feroxbuster -u URL --smart-filter` |

#### Output Options
| Flag | Description | Example |
|------|-------------|---------|
| `-o` | Output file (text) | `feroxbuster -u URL -o results.txt` |
| `-O` | Output file (JSON) | `feroxbuster -u URL -O results.json` |
| `--csv` | CSV output | `feroxbuster -u URL --csv -o results.csv` |
| `--silent` | Silent mode | `feroxbuster -u URL --silent` |
| `--quiet` | Quiet mode | `feroxbuster -u URL --quiet` |
| `-v` | Verbose | `feroxbuster -u URL -v` |
| `-vv` | Very verbose | `feroxbuster -u URL -vv` |
| `--debug` | Debug mode | `feroxbuster -u URL --debug` |
| `--json` | JSON output to stdout | `feroxbuster -u URL --json` |

#### Request Options
| Flag | Description | Example |
|------|-------------|---------|
| `--proxy` | HTTP/SOCKS5 proxy | `feroxbuster -u URL --proxy http://127.0.0.1:8080` |
| `--user-agent` | Custom User-Agent | `feroxbuster -u URL --user-agent "Mozilla/5.0"` |
| `--random-agent` | Random User-Agent | `feroxbuster -u URL --random-agent` |
| `--http` | Force HTTP/1.1 | `feroxbuster -u URL --http` |
| `--insecure` | Skip TLS verify | `feroxbuster -u URL --insecure` |

#### Miscellaneous
| Flag | Description | Example |
|------|-------------|---------|
| `--config` | Config file | `feroxbuster -u URL --config config.toml` |
| `--resume` | Resume from state file | `feroxbuster --resume state.json` |
| `--json` | JSON output | `feroxbuster -u URL --json` |
| `-h` / `--help` | Show help | `feroxbuster -h` |
| `--version` | Show version | `feroxbuster --version` |

### Output Format Explanation

Feroxbuster provides multiple output formats:

**Text Output (default):**
```
|      200          | https://target.com/admin     |     No   |
|      301          | https://target.com/backup    |    Yes   |
```

**JSON Output (`-O results.json`):**
```json
{
  "url": "https://target.com/admin",
  "status": 200,
  "content_length": 512,
  "content_type": "text/html",
  "redirect_url": null,
  "wildcard": false
}
```

**CSV Output (`--csv -o results.csv`):**
```
url,status,content_length,content_type,redirect_url
https://target.com/admin,200,512,text/html,
https://target.com/backup,301,0,,https://target.com/backup/
```

---

## Chapter 4: Intermediate Usage

### Bash Scripting and Automation

#### Batch URL Scanning
```bash
#!/bin/bash
# ferox_batch.sh - Scan multiple targets

TARGETS_FILE="targets.txt"
OUTPUT_DIR="ferox_results"
mkdir -p $OUTPUT_DIR

while IFS= read -r url; do
    echo "[*] Scanning $url"
    feroxbuster -u "$url" \
        -w /usr/share/wordlists/dirb/common.txt \
        -x php,html,txt \
        -t 30 \
        --random-agent \
        -o "$OUTPUT_DIR/$(echo $url | tr '/:' '__').txt"
done < "$TARGETS_FILE"

echo "[+] All scans complete"
```

#### Extension Fuzzing Script
```bash
#!/bin/bash
# ext_fuzz.sh - Test multiple extension sets

TARGET="https://target.com"
EXTENSIONS=(
    "php,html,txt"
    "bak,old,backup"
    "log,conf,config"
    "sql,db,sqlite"
    "xml,json,yaml"
    "js,ts,jsx"
)

for ext in "${EXTENSIONS[@]}"; do
    echo "[*] Testing: $ext"
    feroxbuster -u "$TARGET" \
        -x "$ext" \
        -o "ferox_${ext//,/}.txt" \
        --silent
done

# Combine results
cat ferox_*.txt | sort -u > all_extensions.txt
echo "[+] Found $(wc -l < all_extensions.txt) unique paths"
```

#### Recursive Deep Scan Script
```bash
#!/bin/bash
# deep_scan.sh - Deep recursive scanning

TARGET=$1
WORDLIST="/usr/share/wordlists/dirb/common.txt"

echo "=========================================="
echo "[*] Deep Recursive Scan: $TARGET"
echo "=========================================="

# Phase 1: Quick scan
echo "[Phase 1] Quick scan..."
feroxbuster -u "$TARGET" -w "$WORDLIST" -t 50 --silent -o phase1.txt

# Phase 2: Recursive scan
echo "[Phase 2] Recursive scan (depth 3)..."
feroxbuster -u "$TARGET" -w "$WORDLIST" -r --depth 3 -t 30 --silent -o phase2.txt

# Phase 3: Extension testing
echo "[Phase 3] Extension testing..."
feroxbuster -u "$TARGET" -w "$WORDLIST" -x php,html,txt,bak -t 30 --silent -o phase3.txt

# Phase 4: Combine results
echo "[Phase 4] Combining results..."
cat phase*.txt | sort -u > all_results.txt
echo "[+] Total unique paths: $(wc -l < all_results.txt)"

# Phase 5: Analyze
echo ""
echo "=========================================="
echo "[*] Results Summary"
echo "=========================================="
echo "Phase 1 (Quick): $(wc -l < phase1.txt) paths"
echo "Phase 2 (Recursive): $(wc -l < phase2.txt) paths"
echo "Phase 3 (Extensions): $(wc -l < phase3.txt) paths"
echo "Total unique: $(wc -l < all_results.txt) paths"
echo "=========================================="
```

### Python Scripting Examples

#### Feroxbuster Wrapper
```python
#!/usr/bin/env python3
import subprocess
import json
import sys

def run_feroxbuster(url, wordlist=None, extensions=None, recursion=True,
                     threads=30, depth=3, rate_limit=None):
    """Run feroxbuster and return parsed results."""
    cmd = [
        "feroxbuster",
        "-u", url,
        "-t", str(threads),
        "--json"
    ]

    if wordlist:
        cmd.extend(["-w", wordlist])

    if extensions:
        cmd.extend(["-x", extensions])

    if recursion:
        cmd.extend(["-r", "--depth", str(depth)])

    if rate_limit:
        cmd.extend(["--rate-limit", str(rate_limit)])

    result = subprocess.run(cmd, capture_output=True, text=True)

    try:
        return json.loads(result.stdout)
    except json.JSONDecodeError:
        return {"error": result.stderr}

def main():
    url = sys.argv[1] if len(sys.argv) > 1 else "https://target.com"

    results = run_feroxbuster(
        url,
        extensions="php,html,txt",
        recursion=True,
        depth=2,
        threads=50
    )

    if "results" in results:
        for r in results["results"]:
            print(f"[{r['status']}] {r['url']} ({r['content_length']} bytes)")
    else:
        print("No results found or error occurred")

if __name__ == "__main__":
    main()
```

#### Automated Audit Script
```python
#!/usr/bin/env python3
import subprocess
import json
import os

class FeroxAudit:
    def __init__(self, target, output_dir="ferox_audit"):
        self.target = target
        self.output_dir = output_dir
        os.makedirs(output_dir, exist_ok=True)

    def scan(self, name, **kwargs):
        """Run a feroxbuster scan with given parameters."""
        cmd = ["feroxbuster", "-u", self.target, "--json", "-q"]

        for key, value in kwargs.items():
            if key == "wordlist":
                cmd.extend(["-w", value])
            elif key == "extensions":
                cmd.extend(["-x", value])
            elif key == "threads":
                cmd.extend(["-t", str(value)])
            elif key == "depth":
                cmd.extend(["-r", "--depth", str(value)])
            elif key == "rate_limit":
                cmd.extend(["--rate-limit", str(value)])

        output_file = os.path.join(self.output_dir, f"{name}.json")
        cmd.extend(["-O", output_file])

        print(f"[*] Running scan: {name}")
        subprocess.run(cmd, capture_output=True, text=True)
        return output_file

    def run_all(self):
        """Run all audit scans."""
        wordlist = "/usr/share/wordlists/dirb/common.txt"

        # Quick scan
        self.scan("quick", wordlist=wordlist, threads=50)

        # Recursive scan
        self.scan("recursive", wordlist=wordlist, depth=3, threads=30)

        # Extension scan
        self.scan("extensions", wordlist=wordlist, extensions="php,html,txt,bak", threads=30)

        # Combine results
        self.combine_results()

    def combine_results(self):
        """Combine all scan results."""
        all_urls = set()

        for filename in os.listdir(self.output_dir):
            if filename.endswith(".json"):
                filepath = os.path.join(self.output_dir, filename)
                with open(filepath) as f:
                    try:
                        data = json.load(f)
                        if "results" in data:
                            for r in data["results"]:
                                all_urls.add(r["url"])
                    except json.JSONDecodeError:
                        pass

        # Save combined results
        combined_file = os.path.join(self.output_dir, "all_urls.txt")
        with open(combined_file, "w") as f:
            for url in sorted(all_urls):
                f.write(url + "\n")

        print(f"[+] Total unique URLs: {len(all_urls)}")

if __name__ == "__main__":
    audit = FeroxAudit("https://target.com")
    audit.run_all()
```

### Tool Chaining

#### Feroxbuster + httpx Pipeline
```bash
# Find live hosts, then scan
httpx -l urls.txt -o live.txt

while IFS= read -r url; do
    feroxbuster -u "$url" -o "dirs_${url##*/}.txt"
done < live.txt
```

#### Feroxbuster + nuclei Integration
```bash
# Scan, then test findings with nuclei
feroxbuster -u http://target.com -o dirs.txt

# Extract URLs and scan with nuclei
grep "^" dirs.txt | grep -oP 'https?://[^\s]+' | \
    nuclei -l - -t nuclei-templates/ -o nuclei_results.txt
```

#### Feroxbuster + sqlmap Pipeline
```bash
# Find directories, then test for SQL injection
feroxbuster -u http://target.com -e php -o dirs.txt

# Extract URLs and test with sqlmap
grep "^" dirs.txt | grep -oP 'https?://[^\s]+' | while read url; do
    sqlmap -u "${url}?id=1" --batch --random-agent --level 2
done
```

### Output Processing and Filtering

#### Filter by Status Code
```bash
# Show only 200 OK
feroxbuster -u http://target.com -s 200

# Exclude 404
feroxbuster -u http://target.com -S 404

# Show only redirects
feroxbuster -u http://target.com -s 301,302
```

#### Filter by Response Size
```bash
# Only responses between 100 and 5000 bytes
feroxbuster -u http://target.com --size 100-5000

# Exclude empty responses
feroxbuster -u http://target.com -S 0
```

#### Export to Different Formats
```bash
# JSON output
feroxbuster -u http://target.com -O results.json

# CSV output
feroxbuster -u http://target.com --csv -o results.csv

# Plain text
feroxbuster -u http://target.com -o results.txt
```

---

## Chapter 5: Advanced Usage

### Custom Wordlists

#### Build a Custom Wordlist
```bash
# Create a custom wordlist from JavaScript files
curl -s https://target.com/static/app.js | \
  grep -oP '(?<=[/"'\''])\w+(?=[/"'\''])' | \
  sort -u > js_paths.txt

# Combine with standard wordlists
cat /usr/share/wordlists/dirb/common.txt js_paths.txt | sort -u > custom_wordlist.txt

# Use the custom wordlist
feroxbuster -u https://target.com -w custom_wordlist.txt
```

#### Technology-Specific Wordlists
```bash
# WordPress paths
cat > wordpress_paths.txt << 'EOF'
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

feroxbuster -u https://target.com -w wordpress_paths.txt -e php
```

### Evasion Techniques

#### User-Agent Rotation
```bash
#!/bin/bash
AGENTS=(
    "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36"
    "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36"
    "Mozilla/5.0 (X11; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/115.0"
    "Mozilla/5.0 (iPhone; CPU iPhone OS 17_0 like Mac OS X)"
)

for agent in "${AGENTS[@]}"; do
    echo "[*] Using agent: $agent"
    feroxbuster -u https://target.com --user-agent "$agent" --delay 500ms
done
```

#### Proxy Chaining
```bash
# Route through Burp Suite
feroxbuster -u https://target.com --proxy http://127.0.0.1:8080

# Route through Tor
feroxbuster -u https://target.com --proxy socks5://127.0.0.1:9050

# Route through custom proxy
feroxbuster -u https://target.com --proxy http://proxy.corp.com:3128
```

#### Rate Limiting for Stealth
```bash
# Very slow scan
feroxbuster -u https://target.com --rate-limit 10 --delay 500ms

# Moderate scan
feroxbuster -u https://target.com --rate-limit 50

# Fast scan (be careful!)
feroxbuster -u https://target.com --rate-limit 500
```

### Integration with Pentest Workflows

#### Full Recon Workflow
```bash
#!/bin/bash
# full_recon_workflow.sh

TARGET="https://target.com"
WORDLIST="/usr/share/wordlists/dirb/common.txt"

echo "=========================================="
echo "[*] Full Recon Workflow: $TARGET"
echo "=========================================="

# Phase 1: Quick discovery
echo ""
echo "[Phase 1] Quick Discovery"
feroxbuster -u "$TARGET" -w "$WORDLIST" -t 50 --silent -o quick.txt

# Phase 2: Recursive scan
echo ""
echo "[Phase 2] Recursive Discovery (depth 3)"
feroxbuster -u "$TARGET" -w "$WORDLIST" -r --depth 3 -t 30 --silent -o recursive.txt

# Phase 3: Extension testing
echo ""
echo "[Phase 3] Extension Testing"
feroxbuster -u "$TARGET" -w "$WORDLIST" -x php,html,txt,bak,old -t 30 --silent -o extensions.txt

# Phase 4: Auto-calibrate
echo ""
echo "[Phase 4] Auto-calibrated Scan"
feroxbuster -u "$TARGET" -w "$WORDLIST" -C -t 30 --silent -o calibrated.txt

# Phase 5: Combine results
echo ""
echo "[Phase 5] Combining Results"
cat quick.txt recursive.txt extensions.txt calibrated.txt | \
    grep -oP 'https?://[^\s]+' | sort -u > all_urls.txt

echo "[+] Total unique URLs: $(wc -l < all_urls.txt)"

# Phase 6: Verify findings
echo ""
echo "[Phase 6] Verifying Findings"
while IFS= read -r url; do
    status=$(curl -s -o /dev/null -w "%{http_code}" "$url")
    echo "[$status] $url"
done < all_urls.txt > verified_urls.txt

echo "[+] Verified URLs: $(wc -l < verified_urls.txt)"
echo "=========================================="
echo "[+] Recon Workflow Complete"
echo "=========================================="
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: CTF Challenge Walkthrough

**Objective:** Find hidden admin panel and flag.

```bash
# Step 1: Quick discovery
feroxbuster -u http://target.com:8080 -w /usr/share/wordlists/dirb/common.txt

# Output shows:
# /admin (200)
# /secret (301)
# /robots.txt (200)

# Step 2: Check robots.txt
curl http://target.com:8080/robots.txt
# Reveals: /secret/flag.txt

# Step 3: Recursive scan of /secret
feroxbuster -u http://target.com:8080/secret -r --depth 2

# Step 4: Find the flag
curl http://target.com:8080/secret/flag.txt
# CTF{feroxbuster_found_it}
```

### Scenario 2: Bug Bounty Methodology

**Objective:** Maximize endpoint discovery for a bug bounty program.

```bash
# Step 1: Directory discovery
feroxbuster -u https://target.com \
    -w /usr/share/wordlists/dirb/common.txt \
    -x php,html,txt,js \
    -o dirs.txt

# Step 2: Recursive scan
feroxbuster -u https://target.com \
    -w /usr/share/wordlists/dirb/common.txt \
    -r --depth 2 \
    -o deep_dirs.txt

# Step 3: Combine and deduplicate
cat dirs.txt deep_dirs.txt | grep -oP 'https?://[^\s]+' | sort -u > all_dirs.txt

# Step 4: Filter for interesting paths
grep -E "(admin|api|debug|internal|backup|config)" all_dirs.txt > interesting.txt

# Step 5: Report
echo "=========================================="
echo "Bug Bounty Recon Summary"
echo "=========================================="
echo "Total directories found: $(wc -l < all_dirs.txt)"
echo "Interesting paths: $(wc -l < interesting.txt)"
echo "=========================================="
```

### Scenario 3: Penetration Test

**Objective:** Comprehensive web application assessment.

```bash
# Step 1: Full scan with auto-calibrate
feroxbuster -u https://target.com \
    -w /usr/share/wordlists/dirb/big.txt \
    -C \
    -x php,html,txt,bak,old,conf,config,log \
    -r --depth 3 \
    -t 50 \
    -o full_results.txt

# Step 2: JSON output for analysis
feroxbuster -u https://target.com \
    -w /usr/share/wordlists/dirb/common.txt \
    -O results.json

# Step 3: Parse and analyze
python3 -c "
import json
with open('results.json') as f:
    data = json.load(f)

print('=== Feroxbuster Results ===')
for r in data.get('results', []):
    print(f\"[{r['status']}] {r['url']} ({r['content_length']} bytes)\")
"

# Step 4: Test found endpoints
while IFS= read -r url; do
    echo "[*] Testing: $url"
    # Add your vulnerability testing here
done < full_results.txt
```

---

## Chapter 7: Detection & Defense

### How Defenders Detect Feroxbuster

#### Network Indicators
```
# Feroxbuster request patterns:
# - High volume of HTTP requests
# - Sequential path requests
# - Recursive scan patterns
# - Consistent request timing
# - Multiple 404 responses
# - Default User-Agent string
```

#### Log Analysis
```bash
# Check for scanning patterns
awk '{print $7}' access.log | sort | uniq -c | sort -rn | head -20

# Look for recursive requests
grep "403" access.log | awk '{print $7}' | sort | uniq -c | sort -rn

# Detect high request rates
awk '{print $1}' access.log | sort | uniq -c | sort -rn | head -10

# Look for common wordlist entries
grep -E "(admin|login|config|backup|\.php|\.bak)" access.log | wc -l
```

### IDS/IPS Signatures

```
# Snort/Suricata rules
alert http any any -> $HOME_NET any (msg:"Feroxbuster Detected";
  content:"User-Agent|3a| feroxbuster"; http_header;
  sid:2000001; rev:1;)

alert http any any -> $HOME_NET any (msg:"Web Content Brute-Force";
  threshold:type both,track by_src,count 100,seconds 60;
  sid:2000002; rev:1;)
```

### Mitigation Strategies

#### Rate Limiting (nginx)
```nginx
limit_req_zone $binary_remote_addr zone=ferox:10m rate=10r/s;

location / {
    limit_req zone=ferox burst=20 nodelay;
    limit_req_status 429;
}
```

#### WAF Rules
```bash
# ModSecurity rule
SecRule REQUEST_HEADERS:User-Agent "@rx feroxbuster" \
    "id:1001,phase:1,block,msg:'Feroxbuster detected'"

# Block high request rates
SecRule IP:REQUEST_RATE "@gt 100" \
    "id:1002,phase:1,block,msg:'High request rate detected'"
```

#### Fail2Ban Configuration
```ini
[feroxbuster]
enabled = true
port = http,https
filter = feroxbuster
logpath = /var/log/nginx/access.log
maxretry = 100
findtime = 60
bantime = 3600
```

### Blue Team Response

1. **Monitor request rates** - Detect scanning patterns
2. **Implement rate limiting** - Block excessive requests
3. **Use WAF** - Detect known scanner signatures
4. **Hide sensitive paths** - Use non-standard names
5. **Log analysis** - Regular review of access logs
6. **Block scanner IPs** - Update firewall rules
7. **Document findings** - Record all incidents

---

## Chapter 8: Troubleshooting

### Common Errors

#### Error: "Connection refused"
**Cause:** Target unreachable or not responding.
```bash
# Verify target
curl -I http://target.com

# Increase timeout
feroxbuster -u http://target.com --timeout 10
```

#### Error: "Too many redirects"
**Cause:** Redirect loop on target.
```bash
# Limit redirects
feroxbuster -u http://target.com --max-redirs 5
```

#### Error: "SSL: CERTIFICATE_VERIFY_FAILED"
**Cause:** SSL certificate issue.
```bash
# Skip TLS verification
feroxbuster -u https://target.com -k
```

#### Error: "Connection timed out"
**Cause:** Target is slow or unreachable.
```bash
# Increase timeout
feroxbuster -u http://target.com --timeout 30

# Reduce threads
feroxbuster -u http://target.com -t 10
```

### Performance Optimization

#### Fast Scanning
```bash
# High thread count
feroxbuster -u http://target.com -t 100

# Rate limit for speed
feroxbuster -u http://target.com --rate-limit 500

# Reduce timeout
feroxbuster -u http://target.com --timeout 5
```

#### Thorough Scanning
```bash
# Lower thread count for stability
feroxbuster -u http://target.com -t 20

# Recursive with depth
feroxbuster -u http://target.com -r --depth 3

# Auto-calibrate
feroxbuster -u http://target.com -C
```

#### Memory Optimization
```bash
# Limit concurrent scans
feroxbuster -u http://target.com -r --scan-limit 5

# Use smaller wordlist
feroxbuster -u http://target.com -w /usr/share/wordlists/dirb/common.txt
```

### FAQ

**Q: Is feroxbuster legal to use?**
A: Yes, feroxbuster is a legitimate security testing tool. However, using it without authorization against systems you don't own is illegal in most jurisdictions.

**Q: How do I update feroxbuster?**
A: `sudo apt update && sudo apt upgrade feroxbuster` or `cargo install feroxbuster --force`

**Q: What's the difference between feroxbuster and gobuster?**
A: Feroxbuster is generally faster, has built-in recursion, auto-calibration, and more filtering options. Gobuster is simpler and supports DNS/VHost modes.

**Q: How do I enable recursion?**
A: Use the `-r` flag: `feroxbuster -u http://target.com -r`

**Q: Can I use feroxbuster with a proxy?**
A: Yes, use `--proxy` flag: `feroxbuster -u http://target.com --proxy http://127.0.0.1:8080`

**Q: How do I resume a interrupted scan?**
A: Use the `--resume` flag with the state file: `feroxbuster --resume ferox-state.json`

**Q: What is auto-calibrate?**
A: Auto-calibrate (-C) automatically detects custom 404 pages and filters them from results, reducing false positives.

**Q: How do I filter results?**
A: Use `-S` to exclude status codes, `-W` for word count, `-L` for line count, or `--size` for response size.

---

## Resources

### Official Documentation
- [Feroxbuster GitHub](https://github.com/epi052/feroxbuster)
- [Feroxbuster Wiki](https://github.com/epi052/feroxbuster/wiki)
- [Feroxbuster Releases](https://github.com/epi052/feroxbuster/releases)

### Tutorials
- [Feroxbuster Tutorial - HackTricks](https://book.hacktricks.xyz/pentesting-web/content-discovery)
- [Content Discovery Guide - PortSwigger](https://portswigger.net/web-security)
- [Directory Brute-Forcing - OWASP](https://owasp.org/)

### Books
- "The Web Application Hacker's Handbook" by Dafydd Stuttard
- "Bug Bounty Bootcamp" by Vickie Li
- "Real-World Bug Hunting" by Peter Yaworski

### Related Tools
- **gobuster:** Directory brute-forcing tool
- **ffuf:** Web fuzzer
- **dirsearch:** Directory scanner
- **dirb:** Directory brute-forcer

### Practice Labs
- [DVWA](https://github.com/digininja/DVWA)
- [HackTheBox](https://www.hackthebox.com/)
- [TryHackMe](https://tryhackme.com/)
- [PortSwigger Web Security Academy](https://portswigger.net/web-security)
