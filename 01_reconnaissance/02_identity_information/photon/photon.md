---
tool_name: "photon"
display_name: "Photon"
mitre_tactic: "reconnaissance"
mitre_tactic_id: "TA0043"
mitre_subcategory: "identity_information"
install_command: "pip3 install photon"
version: "0.8"
github_repo: "https://github.com/s0md3v/Photon"
kali_tools_page: "https://www.kali.org/tools/photon/"
difficulty_level: "beginner"
requires_root: false
gui_available: false
tags: ["crawler", "osint", "web", "scraping"]
related_tools: ["gospider", "hakrawler", "gowitness"]
---

# Photon

## Chapter 1: Introduction & Overview

### What is Photon?
Photon is an incredibly fast web crawler designed for OSINT (Open Source Intelligence) gathering. It recursively crawls a target website and extracts URLs, emails, social media accounts, files, and other useful information for penetration testing and security assessments.

### History & Background
- **Created:** 2017 by s0md3v
- **Maintained:** Community
- **License:** MIT
- **Language:** Python 3
- **Purpose:** Fast web crawling for OSINT and information gathering
- **Architecture:** Multi-threaded async crawler
- **Features:** URL, email, file, and social media extraction

### When to Use This Tool
- **Web reconnaissance:** Map target website structure
- **OSINT gathering:** Extract emails, social profiles, files
- **Penetration testing:** Find hidden endpoints and assets
- **Bug bounty:** Discover subdomains and endpoints
- **Content discovery:** Map all pages and resources
- **CTF challenges:** Find hidden web pages and files

### When NOT to Use This Tool
- Without authorization on target websites
- On production sites without rate limiting
- For scraping copyrighted content
- For any illegal activity

### Legal and Ethical Considerations
- Always get written authorization before crawling
- Respect robots.txt directives
- Implement rate limiting to avoid DoS
- Document scope and authorization
- Stay within the defined scope of engagement
- Consider the impact on target infrastructure

### Key Concepts
- **Web crawler:** Automated program that browses web pages
- **OSINT:** Open Source Intelligence from public web sources
- **Depth:** How many levels deep to crawl
- **Spider:** Another term for web crawler
- **Endpoint:** A specific URL or API path
- **Asset:** Any digital resource (file, image, script)

---

## Chapter 2: Installation & Setup

### System Requirements
- **Python:** 3.6+
- **RAM:** 512 MB
- **Network:** Internet access
- **Privileges:** Normal user (no root required)

### Installation

#### Method 1: pip
```bash
pip3 install photon
```
**Expected output:**
```
Collecting photon
  Downloading photon-0.8-py3-none-any.whl (12 kB)
Installing collected packages: photon
Successfully installed photon-0.8
```

#### Method 2: From Source
```bash
git clone https://github.com/s0md3v/Photon.git
cd Photon
pip3 install -r requirements.txt
```
**Expected output:**
```
Cloning into 'Photon'...
remote: Enumerating objects: 2345, done.
remote: Counting objects: 100% (234/234), done.
remote: Compressing objects: 100% (89/89), done.
remote: Total 2345 (delta 156), reused 200 (delta 130)
Receiving objects: 100% (2345/2345), 890.12 KiB | 1.56 MiB/s, done.
Resolving deltas: 100% (156/156), done.
Collecting requests>=2.25.0
  Downloading requests-2.28.1-py3-none-any.whl (62 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 62.3/62.3 kB 2.1 MB/s eta 0:00:00
Installing collected packages: requests, urllib3, certifi, idna, chardet
Successfully installed requests-2.28.1 urllib3-1.26.12 certifi-2022.6.15 idna-3.4 chardet-5.0.0
```

### Verification
```bash
photon --help
```
**Expected output:**
```
Photon v0.8 - Web Crawler for OSINT

Usage: photon [options]

Options:
  -u, --url         Target URL (required)
  -l, --level       Crawl depth (default: 2)
  -t, --threads     Number of threads (default: 10)
  -o, --output      Output directory
  --keys            Find API keys
  --only-urls       Only extract URLs
  --no-external     Don't follow external links
  --random-agents   Use random User-Agents
  --proxy           Use proxy
  --timeout         Request timeout
  -v, --verbose     Verbose output
  -h, --help        Show this help message
```

### Default Output Structure
After running Photon, the output directory contains:
```
photon_output/
├── urls.txt          # All discovered URLs
├── emails.txt        # Extracted email addresses
├── files.txt         # Discovered files (PDF, DOC, etc.)
├── social.txt        # Social media profile links
└── dirs.txt          # Directory structure
```

---

## Chapter 3: Basic Usage

### First Run
```bash
photon -u https://example.com
```
**Output:**
```
[*] Crawling https://example.com
[+] URLs found: 145
[+] Emails found: 12
[+] Files found: 23
[+] Social media: 8
[*] Results saved to photon_output/
```

### Essential Commands

#### Basic Crawl
```bash
photon -u https://target.com
```
**Explanation:** Crawls target website with default settings (depth 2, 10 threads).

#### Set Crawl Depth
```bash
photon -u https://target.com -l 3
```
**Explanation:** Crawls up to 3 levels deep from the starting URL.

#### Extract Only URLs
```bash
photon -u https://target.com --only-urls
```
**Explanation:** Focuses on URL extraction only, ignoring emails and files.

#### Output Format
```bash
photon -u https://target.com -o output_dir
```
**Explanation:** Saves results to specified directory.

#### API Key Discovery
```bash
photon -u https://target.com --keys
```
**Explanation:** Searches for exposed API keys in JavaScript files.

#### Use Random User-Agents
```bash
photon -u https://target.com --random-agents
```
**Explanation:** Randomizes User-Agent strings to avoid detection.

### Complete Flags Reference

| Flag | Description | Example |
|------|-------------|---------|
| `-u`, `--url` | Target URL | `photon -u https://target.com` |
| `-l`, `--level` | Crawl depth | `photon -u URL -l 3` |
| `-t`, `--threads` | Number of threads | `photon -u URL -t 20` |
| `-o`, `--output` | Output directory | `photon -u URL -o results/` |
| `--keys` | Find API keys | `photon -u URL --keys` |
| `--only-urls` | Only extract URLs | `photon -u URL --only-urls` |
| `--no-external` | Don't follow external links | `photon -u URL --no-external` |
| `--random-agents` | Use random User-Agents | `photon -u URL --random-agents` |
| `--proxy` | Use proxy | `photon -u URL --proxy socks5://127.0.0.1:9050` |
| `--timeout` | Request timeout | `photon -u URL --timeout 10` |
| `-v`, `--verbose` | Verbose output | `photon -u URL -v` |
| `--help` | Show help | `photon --help` |

---

## Chapter 4: Intermediate Usage

### Bash Automation
```bash
#!/bin/bash
# Crawl multiple targets
TARGETS=("https://target1.com" "https://target2.com")
for target in "${TARGETS[@]}"; do
    domain=$(echo "$target" | sed 's|https\?://||;s|/.*||')
    photon -u "$target" -o "results/$domain" -l 3 -t 10
done
```

### Python Integration
```python
import subprocess
import os

def crawl_target(url, depth=2):
    """Crawl a target and return results."""
    output_dir = f"crawl_{url.split('//')[1].replace('/', '_')}"
    cmd = ["photon", "-u", url, "-l", str(depth), "-o", output_dir]
    subprocess.run(cmd, capture_output=True)
    return output_dir

results = crawl_target("https://target.com")
```

### Tool Chaining
```bash
# Crawl → Extract emails → Verify
photon -u https://target.com -o crawl_results
cat crawl_results/emails.csv | sort -u > emails_clean.txt
```

### Advanced Python Script
```python
import subprocess
import json
import csv
from datetime import datetime

class PhotonCrawler:
    def __init__(self):
        self.results = {}
    
    def crawl(self, url, depth=2, threads=10):
        cmd = ["photon", "-u", url, "-l", str(depth), "-t", str(threads)]
        result = subprocess.run(cmd, capture_output=True, text=True)
        self.results[url] = {
            "output": result.stdout,
            "errors": result.stderr,
            "timestamp": datetime.now().isoformat()
        }
        return result.stdout
    
    def extract_emails(self, output_dir):
        emails = set()
        try:
            with open(f"{output_dir}/emails.txt") as f:
                for line in f:
                    emails.add(line.strip())
        except FileNotFoundError:
            pass
        return emails
    
    def extract_urls(self, output_dir):
        urls = set()
        try:
            with open(f"{output_dir}/urls.txt") as f:
                for line in f:
                    urls.add(line.strip())
        except FileNotFoundError:
            pass
        return urls
    
    def generate_report(self, filename):
        with open(filename, 'w') as f:
            f.write("# Photon Crawl Report\n\n")
            f.write(f"Generated: {datetime.now().isoformat()}\n\n")
            for url, data in self.results.items():
                f.write(f"## Target: {url}\n\n")
                f.write(f"Timestamp: {data['timestamp']}\n\n")

# Usage
crawler = PhotonCrawler()
crawler.crawl("https://target.com", depth=3)
crawler.generate_report("crawl_report.md")
```

---

## Chapter 5: Advanced Usage

### API Key Discovery
```bash
# Search for exposed API keys
photon -u https://target.com --keys
```

### Custom Headers
```bash
photon -u https://target.com -H "Authorization: Bearer token123"
```

### Recursive Subdomain Crawl
```bash
# Crawl all discovered subdomains
photon -u https://target.com -l 5 --no-external -o full_crawl
```

### Docker Deployment
```bash
# Run Photon in Docker
docker run -it --rm \
  -v $(pwd)/results:/results \
  photon:latest -u https://target.com -o /results
```

### Custom User-Agent Rotation
```bash
# Create custom User-Agent list
cat > user_agents.txt << EOF
Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36
Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36
Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36
EOF

# Use with Photon
photon -u https://target.com --random-agents
```

### Advanced Crawling Techniques
```bash
# Crawl with specific file type focus
photon -u https://target.com --only-urls | grep -E "\.(pdf|doc|xls)"

# Extract JavaScript files
photon -u https://target.com | grep "\.js$" > js_files.txt

# Find API endpoints
photon -u https://target.com | grep -i "api" > api_endpoints.txt
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: Bug Bounty Recon
```bash
# Step 1: Crawl target
photon -u https://target.com -l 3 -o bounty_results

# Step 2: Extract endpoints
cat bounty_results/urls.txt | grep -E "\.(php|asp|jsp)" > dynamic_pages.txt

# Step 3: Check for API keys
photon -u https://target.com --keys -o api_results

# Step 4: Generate report
echo "=== Bug Bounty Recon Report ===" > bounty_report.md
echo "Target: https://target.com" >> bounty_report.md
echo "Date: $(date)" >> bounty_report.md
echo "" >> bounty_report.md
echo "Dynamic Pages Found:" >> bounty_report.md
wc -l dynamic_pages.txt >> bounty_report.md
```

### Scenario 2: CTF Web Recon
```bash
# CTF: Find hidden pages
photon -u http://ctf-target.com -l 4 --no-external -o ctf_crawl

# Check results for flags or hints
grep -r "flag" ctf_crawl/
grep -r "CTF" ctf_crawl/
```

### Scenario 3: Corporate Web Assessment
```bash
# Step 1: Map entire website
photon -u https://corporate.com -l 5 -o corporate_map

# Step 2: Extract all emails
cat corporate_map/emails.csv > employee_emails.txt

# Step 3: Find social media links
cat corporate_map/social.csv > social_profiles.txt

# Step 4: Identify third-party integrations
grep -E "(api|sdk|analytics)" corporate_map/urls.txt > third_party.txt
```

---

## Chapter 7: Detection & Defense

### Detection
- Rapid sequential page requests
- Missing/automated User-Agent
- Unusual crawl patterns (depth-first)
- High request volume
- Regular timing intervals
- Requests for unusual file types

### IDS/IPS Signatures
```
# Snort rule for Photon crawler
alert http any any -> any any (msg:"Photon Web Crawler"; \
  http.user_agent; content:"Photon"; \
  classtype:web-application-attack; sid:1000001; rev:1;)

# Suricata rule for rapid requests
alert http any any -> any any (msg:"Rapid Web Crawl Detected"; \
  threshold:type both, track by_src, count 100, seconds 60; \
  classtype:web-application-attack; sid:1000002; rev:1;)
```

### Mitigation
- Implement rate limiting per IP
- Use WAF to detect automated crawlers
- Monitor for unusual access patterns
- Block known crawler User-Agents
- Deploy honeypots to detect crawlers
- Set up alerts for high request volumes

### Blue Team Response
1. Monitor for rapid sequential page access
2. Log and alert on automated crawler activity
3. Implement progressive delays for repeated requests
4. Use WAF to block known crawler signatures
5. Deploy canary tokens to detect reconnaissance
6. Monitor for unusual file access patterns

---

## Chapter 8: Troubleshooting

### Common Errors

#### "Connection timeout"
**Solution:** Increase timeout with `--timeout 30`

#### "Too many redirects"
**Solution:** Limit depth with `-l 2`

#### "Rate limited"
**Solution:** Reduce threads with `-t 5` and add delays

#### "SSL certificate error"
**Solution:** Use `--no-verify-ssl` or update certificates

### Performance Tips
- Use `--no-external` to stay on target
- Limit depth with `-l` for faster results
- Use `--random-agents` to avoid detection
- Reduce threads on slow networks
- Use `--only-urls` for faster URL extraction
- Cache results to avoid repeat crawling

### FAQ
**Q: How is Photon different from other crawlers?**
A: Photon is optimized for speed with multi-threaded async crawling and focuses on OSINT extraction.

**Q: Can Photon crawl password-protected sites?**
A: No, Photon only crawls publicly accessible pages.

**Q: How do I avoid being blocked?**
A: Use `--random-agents`, `--proxy`, and reduce thread count.

**Q: What output formats does Photon support?**
A: Photon saves results as text files (URLs, emails, files, social media).

---

## Resources

### Official Documentation
- [Photon GitHub](https://github.com/s0md3v/Photon)
- [OSINT Framework](https://osintframework.com)

### Related Tools
- **Gospider** — Fast web spider written in Go
- **Hakrawler** — Simple web crawler for discovery
- **GoWitness** — Screenshot utility for web reconnaissance

### Practice
- Use with authorized web applications
- Practice on intentionally vulnerable apps
- Join bug bounty programs for legal testing
