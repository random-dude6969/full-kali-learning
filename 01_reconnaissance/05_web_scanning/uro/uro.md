---
tool_name: "uro"
display_name: "Uro"
mitre_tactic: "reconnaissance"
mitre_tactic_id: "TA0043"
mitre_subcategory: "web_scanning"
install_command: "pip3 install uro"
version: "0.4.2"
github_repo: "https://github.com/s0md3v/uro"
kali_tools_page: "https://www.kali.org/tools/uro/"
difficulty_level: "beginner"
requires_root: false
gui_available: false
tags: ["url", "filtering", "deduplication", "web", "osint"]
related_tools: ["gau", "waybackurls", "gospider", "httpx"]
---

# Uro

## Chapter 1: Introduction & Overview

### What is Uro?
Uro is a URL filtering tool designed to declutter the output of URL discovery tools like gau, waybackurls, and gospider. It removes duplicate URLs and filters out URLs that belong to the same directory, producing a clean, unique list of endpoints for further testing.

### History & Background
- **Created:** 2020 by s0md3v (Sandeep Singh)
- **Maintained:** s0md3v
- **License:** MIT
- **Language:** Python 3
- **Purpose:** URL deduplication and filtering for pentesting

### When to Use This Tool
- **URL cleanup:** Clean output from gau, waybackurls, gospider
- **Pentest workflows:** Prepare URL lists for fuzzing/scanning
- **Bug bounty:** Filter discovered endpoints before testing
- **CTF:** Clean URL lists for endpoint discovery

### When NOT to Use This Tool
- On raw, unfiltered URL lists (use gau/waybackurls first)
- For active scanning (pair with wfuzz/ffuf/nuclei)

### Legal and Ethical Considerations
- Uro is a local filtering tool — no network activity
- Only process URLs from authorized targets
- Use filtered output for legitimate testing only

### Key Concepts
- **Deduplication:** Removing identical URLs
- **Directory grouping:** URLs in the same path are considered similar
- **Similarity threshold:** How aggressively to filter similar URLs
- **Pipeline:** Chain tools together using Unix pipes

---

## Chapter 2: Installation & Setup

### System Requirements
- **Python:** 3.6+
- **RAM:** 512 MB
- **Disk Space:** 10 MB
- **Network:** Not required (local tool)

### Installation Methods

#### Method 1: pip (Recommended)
```bash
pip3 install uro
```

#### Method 2: From Source
```bash
git clone https://github.com/s0md3v/uro.git
cd uro
pip3 install -r requirements.txt
python3 setup.py install
```

### Verification
```bash
uro --help
```
**Output:**
```
usage: uro [-h] [-t THRESHOLD] [-o OUTPUT] [--verbose] [input]

URL filter for web pentesters

positional arguments:
  input                 Input file (default: stdin)

optional arguments:
  -h, --help            show this help message and exit
  -t THRESHOLD, --threshold THRESHOLD
                        Similarity threshold (0.0-1.0, default: 0.6)
  -o OUTPUT, --output OUTPUT
                        Output file (default: stdout)
  --verbose             Show filtering details
```

---

## Chapter 3: Basic Usage

### First Run
```bash
echo -e "https://target.com/page1\nhttps://target.com/page2\nhttps://target.com/page1" | uro
```
**Output:**
```
https://target.com/page1
https://target.com/page2
```

### Essential Commands

#### Filter URLs from stdin
```bash
cat urls.txt | uro
```
**Explanation:** Reads URLs from file, removes duplicates, outputs unique URLs.

#### Filter with gau
```bash
gau target.com | uro > filtered.txt
```
**Explanation:** Collects all URLs from gau, then filters to unique endpoints.

#### Save to file
```bash
cat urls.txt | uro -o clean_urls.txt
```
**Explanation:** Outputs filtered URLs to a file instead of stdout.

#### Custom threshold
```bash
cat urls.txt | uro -t 0.8
```
**Explanation:** More aggressive filtering — removes URLs that are 80% similar.

#### Verbose mode
```bash
cat urls.txt | uro --verbose
```
**Explanation:** Shows which URLs were filtered and why.

### Complete Flags Reference

| Flag | Description | Example |
|------|-------------|---------|
| `-t`, `--threshold` | Similarity threshold (0.0-1.0) | `cat urls.txt \| uro -t 0.8` |
| `-o`, `--output` | Output file | `cat urls.txt \| uro -o clean.txt` |
| `--verbose` | Show filtering details | `cat urls.txt \| uro --verbose` |
| `-h`, `--help` | Show help | `uro --help` |

### How Uro Works Internally
1. Reads URLs line by line from stdin or file
2. Groups URLs by directory path
3. Compares URLs within each group using similarity ratio
4. Removes URLs below the threshold (too similar)
5. Outputs unique, diverse URLs

### Similarity Threshold Guide
| Threshold | Behavior | Use Case |
|-----------|----------|----------|
| 0.3 | Very aggressive | Quick cleanup, many URLs removed |
| 0.5 | Moderate | Default, good balance |
| 0.6 | Default | Standard deduplication |
| 0.8 | Conservative | Keep more variation |
| 1.0 | Exact match only | Only remove exact duplicates |

---

## Chapter 4: Intermediate Usage

### Bash Automation

#### Multi-Source URL Collection
```bash
#!/bin/bash
# Collect URLs from multiple sources and filter
TARGET="target.com"
echo "[*] Collecting URLs from gau..."
gau "$TARGET" > /tmp/gau.txt

echo "[*] Collecting URLs from waybackurls..."
echo "$TARGET" | waybackurls > /tmp/wayback.txt

echo "[*] Collecting URLs from gospider..."
gospider -s "https://$TARGET" -d 2 --plain-source | \
    grep -oP 'https?://\S+' > /tmp/gospider.txt

echo "[*] Merging and filtering..."
cat /tmp/gau.txt /tmp/wayback.txt /tmp/gospider.txt | \
    sort -u | uro > "${TARGET}_urls.txt"

echo "[+] $(wc -l < "${TARGET}_urls.txt") unique URLs found"
rm /tmp/gau.txt /tmp/wayback.txt /tmp/gospider.txt
```

#### Pipeline with httpx
```bash
# Discover URLs → Filter → Check alive
gau target.com | uro | httpx -silent -mc 200,301,302 > alive_urls.txt
```

#### Pipeline with wfuzz
```bash
# Collect → Filter → Fuzz directories
gau target.com | uro | wfuzz -c -z file,/dev/stdin http://target.com/FUZZ
```

### Python Scripting

#### Programmatic Usage
```python
import subprocess

def filter_urls(input_file, threshold=0.6):
    """Filter URLs using uro."""
    cmd = f"cat {input_file} | uro -t {threshold}"
    result = subprocess.run(cmd, shell=True, capture_output=True, text=True)
    return result.stdout.strip().split('\n')

# Example
urls = filter_urls("raw_urls.txt", threshold=0.7)
print(f"Filtered to {len(urls)} unique URLs")
```

#### Custom Filtering Script
```python
#!/usr/bin/env python3
import subprocess
import sys

def pipeline_filter(target):
    """Full URL discovery pipeline with uro."""
    commands = [
        f"gau {target}",
        f"waybackurls < /dev/null",
        f"gospider -s https://{target} -d 1 --plain-source | grep -oP 'https?://\\S+'",
    ]
    
    all_urls = []
    for cmd in commands:
        try:
            result = subprocess.run(cmd, shell=True, capture_output=True, text=True, timeout=60)
            all_urls.extend(result.stdout.strip().split('\n'))
        except subprocess.TimeoutExpired:
            continue
    
    # Write to temp file and filter with uro
    with open('/tmp/all_urls.txt', 'w') as f:
        f.write('\n'.join(all_urls))
    
    filtered = subprocess.run('cat /tmp/all_urls.txt | uro', shell=True, capture_output=True, text=True)
    return filtered.stdout.strip().split('\n')

if __name__ == "__main__":
    target = sys.argv[1]
    urls = pipeline_filter(target)
    for url in urls:
        if url:
            print(url)
```

### Tool Chaining Examples

#### Complete Recon Pipeline
```bash
gau target.com | uro | httpx -silent | tee alive_endpoints.txt | \
    nuclei -t exposures/ -t misconfigurations/
```

#### Endpoint Discovery for Bug Bounty
```bash
# Step 1: Collect
(subfinder -d target.com -silent | gau; echo "https://target.com") | \
    sort -u | uro | tee endpoints.txt

# Step 2: Filter by extension
grep -E "\.(php|asp|aspx|jsp|json|xml)$" endpoints.txt > dynamic_endpoints.txt

# Step 3: Scan
cat dynamic_endpoints.txt | nuclei -t vulns/
```

---

## Chapter 5: Advanced Usage

### Custom Threshold Tuning
```bash
# Exact duplicates only
cat urls.txt | uro -t 1.0

# Very aggressive (keep only very different URLs)
cat urls.txt | uro -t 0.3

# Test different thresholds
for t in 0.3 0.5 0.6 0.8 1.0; do
    count=$(cat urls.txt | uro -t $t | wc -l)
    echo "Threshold $t: $count URLs"
done
```

### Combining with Other Filters
```bash
# Filter by extension, then deduplicate
cat urls.txt | grep -E "\.(php|js|json)$" | uro > filtered_by_ext.txt

# Filter by status code, then deduplicate
cat urls.txt | httpx -silent -mc 200 | uro > alive_200.txt

# Filter by length, then deduplicate
cat urls.txt | httpx -silent -cl | awk '$NF > 1000' | uro > large_pages.txt
```

### Regex Filtering
```bash
# Keep only API endpoints
cat urls.txt | uro | grep -iE "(api|v[0-9]|graphql|rest)" > api_endpoints.txt

# Keep only admin panels
cat urls.txt | uro | grep -iE "(admin|panel|dashboard|login)" > admin_panels.txt

# Remove static assets
cat urls.txt | uro | grep -v -E "\.(css|js|png|jpg|gif|svg|ico|woff|ttf)$" > no_static.txt
```

### Bulk Processing with Parallel Execution
```bash
#!/bin/bash
# bulk_uro.sh — Process multiple domains in parallel

DOMAINS_FILE="${1:?Usage: $0 <domains_file>}"
MAX_PARALLEL=10

echo "=== Bulk URL Processing ==="
echo "Domains: $(wc -l < "$DOMAINS_FILE")"
echo "Max parallel: $MAX_PARALLEL"

cat "$DOMAINS_FILE" | xargs -P $MAX_PARALLEL -I {} bash -c '
    DOMAIN="{}"
    OUTPUT="filtered_${DOMAIN}.txt"
    
    # Collect URLs
    (gau "$DOMAIN" 2>/dev/null; echo "$DOMAIN" | waybackurls 2>/dev/null) | \
        sort -u | uro > "$OUTPUT"
    
    echo "[+] $DOMAIN: $(wc -l < "$OUTPUT") filtered URLs"
'

# Combine all results
cat filtered_*.txt | sort -u > all_filtered.txt
echo ""
echo "[+] Total unique URLs across all domains: $(wc -l < all_filtered.txt)"
```

### Uro in CI/CD Pipelines
```bash
#!/bin/bash
# ci_url_check.sh — Automated URL monitoring in CI/CD

TARGET_DOMAIN="${1:?Usage: $0 <domain>}"
SNAPSHOT_FILE=".url_snapshot.txt"

# Collect current URLs
gau "$TARGET_DOMAIN" | sort -u | uro > current_urls.txt

# Compare with previous snapshot
if [ -f "$SNAPSHOT_FILE" ]; then
    NEW_URLS=$(comm -13 "$SNAPSHOT_FILE" current_urls.txt)
    REMOVED_URLS=$(comm -23 "$SNAPSHOT_FILE" current_urls.txt)
    
    if [ -n "$NEW_URLS" ]; then
        echo "[!] New URLs detected:"
        echo "$NEW_URLS"
    fi
    
    if [ -n "$REMOVED_URLS" ]; then
        echo "[!] Removed URLs:"
        echo "$REMOVED_URLS"
    fi
else
    echo "[*] No previous snapshot found. Creating baseline."
fi

# Save current as snapshot
mv current_urls.txt "$SNAPSHOT_FILE"
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: Bug Bounty Recon Pipeline
```bash
#!/bin/bash
# Complete bug bounty URL pipeline
TARGET="$1"
REPORT="bounty_$(date +%Y%m%d)"

echo "[*] Phase 1: URL Collection"
gau "$TARGET" > "/tmp/${REPORT}_gau.txt"
echo "$TARGET" | waybackurls > "/tmp/${REPORT}_wayback.txt"
gospider -s "https://$TARGET" -d 2 --plain-source 2>/dev/null | \
    grep -oP 'https?://\S+' > "/tmp/${REPORT}_gospider.txt"

echo "[*] Phase 2: Merge & Filter"
cat /tmp/${REPORT}_*.txt | sort -u | uro > "${REPORT}_filtered.txt"

echo "[*] Phase 3: Alive Check"
cat "${REPORT}_filtered.txt" | httpx -silent -mc 200,301,302,403 > "${REPORT}_alive.txt"

echo "[*] Phase 4: Vulnerability Scan"
cat "${REPORT}_alive.txt" | nuclei -severity critical,high -o "${REPORT}_vulns.txt"

echo "[+] Complete!"
echo "  Filtered: $(wc -l < ${REPORT}_filtered.txt) URLs"
echo "  Alive: $(wc -l < ${REPORT}_alive.txt) endpoints"
echo "  Vulnerabilities: $(wc -l < ${REPORT}_vulns.txt) found"
```

### Scenario 2: CTF Endpoint Discovery
```bash
# CTF: Find hidden API endpoints
gau ctf-target.com | uro | grep -iE "(api|flag|secret|hidden|admin)" | \
    httpx -silent -mc 200,301 | while read url; do
        echo "[*] Testing: $url"
        curl -s "$url" | head -5
        echo "---"
    done
```

### Scenario 3: Corporate Asset Discovery
```bash
# Find all unique endpoints across subdomains
subfinder -d company.com -silent | while read sub; do
    gau "https://$sub" 2>/dev/null
done | uro | httpx -silent | tee company_endpoints.txt

# Analyze by subdomain
cut -d'/' -f3 company_endpoints.txt | sort | uniq -c | sort -rn | head -20
```

---

## Chapter 7: Detection & Defense

### Detection
- Uro is a local filtering tool with **zero network activity**
- Cannot be detected by target systems
- Only processes data locally

### How to Defend Against Uro-Filtered Scans
Since uro is used offline, defenders should:

1. **Monitor for tool patterns:** gau, waybackurls, gospider User-Agents
2. **Rate limit endpoints:** Slow down mass scanning
3. **Honeypots:** Deploy fake endpoints to catch automated scanners
4. **WAF rules:** Block common scan patterns in URLs

### Blue Team Integration
```bash
# Monitor access logs for patterns typical of uro-filtered scans
grep -E "(gau|waybackurls|gospider|httpx)" access.log | \
    awk '{print $1}' | sort | uniq -c | sort -rn | head -10
```

---

## Chapter 8: Troubleshooting

### Common Errors

#### "No output"
**Cause:** Input format incorrect or all URLs filtered out.
**Solution:** Ensure input is one URL per line. Try lower threshold: `uro -t 0.3`

#### "ModuleNotFoundError"
**Cause:** uro not installed properly.
**Solution:** `pip3 install uro --force-reinstall`

#### "Too many results"
**Cause:** Threshold too high (not filtering enough).
**Solution:** Lower threshold: `uro -t 0.4`

#### "Too few results"
**Cause:** Threshold too low (filtering too aggressively).
**Solution:** Raise threshold: `uro -t 0.8`

### Performance Tips
- For large files (100K+ URLs), use `sort -u` before uro
- Pipe through `head` for testing: `cat urls.txt | head -1000 | uro`
- Use `--verbose` to understand filtering decisions

### FAQ

**Q: uro vs sort -u?**
A: `sort -u` only removes exact duplicates. uro also removes similar URLs (e.g., `/page?id=1` and `/page?id=2`).

**Q: What threshold should I use?**
A: Start with 0.6 (default). Use 0.8 for conservative, 0.3 for aggressive filtering.

**Q: Can uro filter by extension?**
A: No — use grep before uro: `cat urls.txt | grep "\.php" | uro`

---

## Resources

### Official Documentation
- [Uro GitHub](https://github.com/s0md3v/uro)
- [s0md3v's Tools](https://github.com/s0md3v)

### Related Tools
- **gau** — Fetch known URLs from AlienVault OTX, Wayback Machine, Common Crawl
- **waybackurls** — Fetch URLs from Wayback Machine
- **gospider** — Fast web spider
- **httpx** — Fast HTTP toolkit for probing URLs
- **gau + uro + httpx** — Classic recon pipeline

### Practice
- Use with gau on bug bounty targets
- Practice filtering on large URL lists
- Experiment with different thresholds
