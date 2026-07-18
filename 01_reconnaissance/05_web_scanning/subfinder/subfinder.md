---
tool_name: "subfinder"
display_name: "Subfinder"
mitre_tactic: "reconnaissance"
mitre_tactic_id: "TA0043"
mitre_subcategory: "web_scanning"
install_command: "sudo apt install subfinder"
version: "2.6.3"
github_repo: "https://github.com/projectdiscovery/subfinder"
kali_tools_page: "https://www.kali.org/tools/subfinder/"
difficulty_level: "beginner"
requires_root: false
gui_available: false
tags: ["subdomain", "enumeration", "osint", "passive"]
related_tools: ["amass", "assetfinder", "findomain"]
---

# Subfinder

## Chapter 1: Introduction & Overview

### What is Subfinder?

Subfinder is a passive subdomain discovery tool developed by ProjectDiscovery. It uses a wide array of online sources — including search engines, certificate transparency logs, DNS databases, passive DNS repositories, and web archives — to discover subdomains without ever directly connecting to the target. This passive-only approach makes it extremely stealthy and safe for reconnaissance during authorized penetration tests and bug bounty programs.

Unlike active subdomain brute-force tools that generate DNS queries against the target, Subfinder queries third-party data sources that have already indexed the target's subdomains. This means the target never sees any direct traffic from Subfinder, making it ideal for early-stage reconnaissance where stealth is important.

### History & Background

- **Created:** 2017 by Ahmed Aboul-Ela (bSys)
- **Maintained:** ProjectDiscovery (https://github.com/projectdiscovery)
- **License:** MIT
- **Language:** Go
- **Latest version:** 2.6.3
- **Upstream repository:** https://github.com/projectdiscovery/subfinder

Subfinder was originally created as a standalone tool and later adopted by ProjectDiscovery as part of their suite of reconnaissance and vulnerability scanning tools. It has become one of the most popular subdomain enumeration tools due to its speed, accuracy, and ease of use. The tool is actively maintained with regular updates to add new sources and improve performance.

### When to Use This Tool

- **Passive reconnaissance:** Find subdomains without directly touching the target
- **Bug bounty programs:** Discover subdomains within scope for testing
- **Penetration testing:** Build comprehensive subdomain lists as part of the information-gathering phase
- **OSINT investigations:** Map an organization's web infrastructure using publicly available data
- **Attack surface mapping:** Identify all web properties owned by a target organization
- **CTF challenges:** Discover hidden subdomains in capture-the-flag competitions
- **Mergers & acquisitions:** Assess web infrastructure of companies being acquired

### Legal and Ethical Considerations

- Subfinder is entirely passive — it never sends queries to the target domain
- All data comes from third-party public sources
- Passive reconnaissance is generally legal in most jurisdictions
- Always verify that discovered subdomains are within the authorized scope
- Use findings only for authorized security assessments
- Respect the terms of service of the data sources used
- Some sources (Shodan, Censys) require API keys and have usage terms

### Key Concepts

| Concept | Description |
|---------|-------------|
| **Passive Enumeration** | Finding subdomains through third-party sources without directly querying the target |
| **Certificate Transparency (CT)** | Public logs of SSL/TLS certificates that reveal subdomains |
| **Passive DNS** | Databases that store historical DNS resolution data |
| **OSINT** | Open Source Intelligence — information gathered from publicly available sources |
| **API Keys** | Authentication tokens for accessing premium data sources |
| **Recursive Enumeration** | Finding subdomains of discovered subdomains |
| **Source** | An online service or database that Subfinder queries for subdomain data |

---

## Chapter 2: Installation & Setup

### System Requirements

- Operating system: Linux, macOS, or Windows (including WSL)
- RAM: 256 MB minimum
- Disk space: < 50 MB
- Network: Internet access required
- Go: 1.19+ (for installation from source)
- Privileges: Non-root is sufficient

### Installation via APT (Kali/Debian/Ubuntu)

```bash
sudo apt update
sudo apt install subfinder
```

Expected output:
```
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following NEW packages will be installed:
  subfinder
0 upgraded, 1 newly installed, 0 to remove and 0 not upgraded.
Need to get 7.8 MB of archives.
After this operation, 31.2 MB of additional disk space will be used.
Get:1 http://http.kali.org/kali kali-rolling/main amd64 subfinder 2.6.3+ds1-1kali1 [7.8 MB]
Fetched 7.8 MB in 1s (7.8 MB/s)
Selecting previously unselected package subfinder.
(Reading database ... 1234567 files and directories currently installed.)
Preparing to unpack .../subfinder_2.6.3+ds1-1kali1_amd64.deb ...
Unpacking subfinder (2.6.3+ds1-1kali1) ...
Setting up subfinder (2.6.3+ds1-1kali1) ...
```

### Installation from Source (Go)

```bash
# Install Go (if not present)
sudo apt install golang-go

# Install subfinder
go install -v github.com/projectdiscovery/subfinder/v2/cmd/subfinder@latest
```

Expected output:
```
github.com/projectdiscovery/subfinder/v2 (download)
github.com/projectdiscovery/subfinder/v2/pkg/runner
github.com/projectdiscovery/subfinder/v2/pkg/subscraping
github.com/projectdiscovery/subfinder/v2/cmd/subfinder
```

The binary will be installed to `~/go/bin/subfinder`.

### Installation via GoReleaser

```bash
# Download latest release from GitHub
curl -sL https://github.com/projectdiscovery/subfinder/releases/latest/download/subfinder_$(uname -s)_$(uname -m).zip -o subfinder.zip
unzip subfinder.zip -d /usr/local/bin/
chmod +x /usr/local/bin/subfinder
```

### Docker Installation

```bash
# Pull the official image
docker pull projectdiscovery/subfinder:latest

# Run with output to stdout
docker run --rm projectdiscovery/subfinder:latest -d example.com

# Run with volume mount for output
docker run --rm -v $(pwd):/output projectdiscovery/subfinder:latest -d example.com -o /output/subs.txt
```

### API Key Configuration

Subfinder uses API keys to access premium data sources. Configure them in:

```bash
# Create config directory
mkdir -p ~/.config/subfinder

# Create provider config file
cat > ~/.config/subfinder/provider-config.yaml << 'EOF'
# Shodan (https://account.shodan.io)
shodan:
  - YOUR_SHODAN_API_KEY

# Censys (https://censys.io)
censys:
  - YOUR_CENSYS_API_ID:YOUR_CENSYS_API_SECRET

# VirusTotal (https://www.virustotal.com)
virustotal:
  - YOUR_VT_API_KEY

# SecurityTrails (https://securitytrails.com)
securitytrails:
  - YOUR_SECURITYTRAILS_KEY

# Shodan (alternative key format)
shodan:
  - YOUR_SHODAN_KEY

# BinaryEdge (https://binaryedge.io)
binaryedge:
  - YOUR_BINARYEDGE_KEY

# GitHub (https://github.com/settings/tokens)
github:
  - YOUR_GITHUB_TOKEN

# FullHunt (https://fullhunt.io)
fullhunt:
  - YOUR_FULLHUNT_KEY

# Passivedns (https://www.passivedns.com)
passivedns:
  - YOUR_PASSIVEDNS_KEY
EOF
```

### Verification

```bash
subfinder -version
```

Expected output:
```
subfinder version v2.6.3
```

Or verify with:

```bash
which subfinder
subfinder -h 2>&1 | head -3
```

---

## Chapter 3: Basic Usage

### First Run

```bash
subfinder -d example.com
```

Sample output:
```
                     __    __
    ____  __  ______/ /_  / /_  ___  __  ________
   / __ \/ / / / ___/ __ \/ __ \/ _ \/ / / / ___/
  / /_/ / /_/ / /__/ / / / /_/ /  __/ /_/ (__  )
 / .___/\_,_/\___/_/ /_/\____/\___/\__,_/____/
 /_/_                                 v2.6.3

[INF] Enumerating subdomains for example.com
blog.example.com
docs.example.com
ftp.example.com
mail.example.com
shop.example.com
staging.example.com
www.example.com
[INF] Found 7 subdomains for example.com
```

### Essential Commands

#### Basic Enumeration

```bash
subfinder -d example.com
```

**Explanation:** Discovers subdomains of example.com using all available passive sources. Results are printed to stdout.

#### Silent Output

```bash
subfinder -d example.com -silent
```

**Explanation:** Outputs only discovered subdomains without the banner or informational messages. Ideal for piping to other tools.

Expected output:
```
blog.example.com
docs.example.com
ftp.example.com
mail.example.com
shop.example.com
staging.example.com
www.example.com
```

#### Output to File

```bash
subfinder -d example.com -o subdomains.txt
```

**Explanation:** Saves discovered subdomains to a file.

#### Multiple Domains from File

```bash
subfinder -dL domains.txt -o all_subdomains.txt
```

**Explanation:** Reads domains from `domains.txt` (one per line) and discovers subdomains for each.

#### Use All Sources

```bash
subfinder -d example.com -all
```

**Explanation:** Uses all available sources including those that require API keys (if configured). Without `-all`, only free sources are used.

#### Recursive Enumeration

```bash
subfinder -d example.com -recursive
```

**Explanation:** After finding subdomains, recursively enumerates subdomains of those subdomains. This can significantly increase results but takes longer.

#### Specify Threads

```bash
subfinder -d example.com -t 30
```

**Explanation:** Uses 30 concurrent threads for faster enumeration (default is 30).

#### Set Timeout

```bash
subfinder -d example.com -timeout 60
```

**Explanation:** Sets a 60-second timeout per source (default is 30 seconds).

#### Use Specific Sources

```bash
subfinder -d example.com -sources crtsh,threatminer,virustotal
```

**Explanation:** Only queries specific sources instead of all available ones.

#### Set Source Timeout

```bash
subfinder -d example.com -timeout 30
```

**Explanation:** Sets timeout for each individual source to 30 seconds.

#### Custom Provider Config

```bash
subfinder -d example.com -provider-config /path/to/config.yaml
```

**Explanation:** Uses a custom provider configuration file for API keys.

### Complete Flags Reference

| Flag | Long Form | Description | Default | Example |
|------|-----------|-------------|---------|---------|
| `-d` | `--domain` | Target domain | none (required) | `subfinder -d example.com` |
| `-dL` | `--domain-list` | File with domains (one per line) | none | `subfinder -dL domains.txt` |
| `-o` | `--output` | Output file path | stdout | `subfinder -d example.com -o out.txt` |
| `-oJ` | `--json` | Output in JSON format | off | `subfinder -d example.com -oJ` |
| `-silent` | `--silent` | Suppress banner and info | off | `subfinder -d example.com -silent` |
| `-all` | `--all` | Use all sources (including API keys) | off | `subfinder -d example.com -all` |
| `-recursive` | `--recursive` | Enable recursive enumeration | off | `subfinder -d example.com -recursive` |
| `-t` | `--threads` | Number of concurrent threads | 30 | `subfinder -d example.com -t 50` |
| `-timeout` | `--timeout` | Timeout per source in seconds | 30 | `subfinder -d example.com -timeout 60` |
| `-sources` | `--sources` | Comma-separated list of sources | all | `subfinder -d example.com -sources crtsh` |
| `-exclude-sources` | `--exclude-sources` | Exclude specific sources | none | `subfinder -d example.com -exclude-sources shodan` |
| `-provider-config` | `--provider-config` | API keys config file | `~/.config/subfinder/provider-config.yaml` | `subfinder -provider-config keys.yaml` |
| `-pc` | `--provider-config` | Alias for --provider-config | — | `subfinder -pc keys.yaml` |
| `-max-time` | `--max-time` | Maximum enumeration time | 0 (unlimited) | `subfinder -d example.com -max-time 300` |
| `-rate-limit` | `--rate-limit` | Rate limit (requests per minute) | 0 (unlimited) | `subfinder -d example.com -rate-limit 100` |
| `-h` | `--help` | Show help message | — | `subfinder -h` |
| `-v` | `--verbose` | Verbose output | off | `subfinder -d example.com -v` |

### Available Sources

Subfinder supports 40+ passive sources:

| Source | Requires API Key | Type |
|--------|-----------------|------|
| anubis | No | Certificate transparency |
| beVigil | Yes | Mobile app intelligence |
| binaryedge | Yes | Internet scanning |
| bufferover | No | Certificate transparency |
| c99 | Yes | DNS enumeration |
| certspotter | No | Certificate transparency |
| censys | Yes | Certificate intelligence |
| chaos | Yes | ProjectDiscovery DB |
| crtsh | No | Certificate transparency |
| digitorus | No | DNS enumeration |
| dnsdumpster | No | DNS reconnaissance |
| facebook | Yes | Facebook threat exchange |
| github | Yes | GitHub code search |
| gitlab | Yes | GitLab code search |
| hackertarget | No | DNS enumeration |
| hunter | Yes | Email intelligence |
| intelx | Yes | Intelligence X |
| passivedns | Yes | Passive DNS |
| rapiddns | No | DNS enumeration |
| shodan | Yes | Internet scanning |
| sitedossier | No | DNS enumeration |
| threatminer | Yes | Threat intelligence |
| urlscan | Yes | URL scanning |
| virustotal | Yes | Threat intelligence |
| waybackarchive | No | Web archive |
| yahoo | No | Search engine |

---

## Chapter 4: Intermediate Usage

### Bash Automation Scripts

#### Multi-Domain Enumeration Pipeline

```bash
#!/bin/bash
# multi_domain_enum.sh — Enumerate subdomains for multiple domains

DOMAINS_FILE="${1:?Usage: $0 <domains_file>}"
OUTPUT_DIR="subfinder_$(date +%Y%m%d_%H%M%S)"
mkdir -p "$OUTPUT_DIR"

echo "=== Subfinder Multi-Domain Enumeration ==="
echo "Started: $(date)"
echo "Domains file: $DOMAINS_FILE"
echo ""

while IFS= read -r domain; do
    [ -z "$domain" ] && continue
    echo "[*] Enumerating: $domain"
    
    # Run subfinder with silent output
    subfinder -d "$domain" -silent -all -o "$OUTPUT_DIR/${domain}.txt" 2>/dev/null
    
    # Count results
    COUNT=$(wc -l < "$OUTPUT_DIR/${domain}.txt" 2>/dev/null || echo "0")
    echo "    Found: $COUNT subdomains"
    
done < "$DOMAINS_FILE"

# Combine all results
cat "$OUTPUT_DIR"/*.txt | sort -u > "$OUTPUT_DIR/all_subdomains.txt"
TOTAL=$(wc -l < "$OUTPUT_DIR/all_subdomains.txt")
echo ""
echo "=== Enumeration Complete ==="
echo "Total unique subdomains: $TOTAL"
echo "Results in: $OUTPUT_DIR/"
```

#### Filtered Enumeration with Validation

```bash
#!/bin/bash
# validated_enum.sh — Enumerate and validate subdomains

TARGET="${1:?Usage: $0 <domain>}"

echo "[*] Running subfinder..."
subfinder -d "$TARGET" -all -silent > raw_subs.txt

echo "[*] Running amass (passive)..."
amass enum -passive -d "$TARGET" > amass_subs.txt

echo "[*] Combining results..."
cat raw_subs.txt amass_subs.txt | sort -u > all_subs.txt
echo "    Raw subdomains: $(wc -l < all_subs.txt)"

echo "[*] Resolving subdomains..."
massdns -r resolvers.txt -t A -o S all_subs.txt > resolved.txt

echo "[*] Filtering active subdomains..."
grep " A " resolved.txt | awk '{print $1}' | sed 's/\.$//' | sort -u > active_subs.txt
echo "    Active subdomains: $(wc -l < active_subs.txt)"
```

### Tool Chaining

#### Subfinder → MassDNS → httpx Pipeline

```bash
#!/bin/bash
# full_recon_pipeline.sh — Complete subdomain recon pipeline

TARGET="${1:?Usage: $0 <domain>}"
WORKDIR="recon_${TARGET}"
mkdir -p "$WORKDIR"

echo "=== Full Recon Pipeline: $TARGET ==="

# Phase 1: Subdomain Enumeration
echo "[Phase 1/5] Enumerating subdomains..."
subfinder -d "$TARGET" -all -silent -o "$WORKDIR/subfinder_raw.txt"

# Combine with other tools
amass enum -passive -d "$TARGET" > "$WORKDIR/amass_raw.txt" 2>/dev/null
cat "$WORKDIR/subfinder_raw.txt" "$WORKDIR/amass_raw.txt" | sort -u > "$WORKDIR/all_subs.txt"
echo "  Found $(wc -l < "$WORKDIR/all_subs.txt") unique subdomains"

# Phase 2: Resolution
echo "[Phase 2/5] Resolving subdomains..."
massdns -r resolvers.txt -t A -o S -w "$WORKDIR/resolved.txt" "$WORKDIR/all_subs.txt"
grep " A " "$WORKDIR/resolved.txt" | awk '{print $1}' | sed 's/\.$//' | sort -u > "$WORKDIR/active_subs.txt"
echo "  $(wc -l < "$WORKDIR/active_subs.txt") active subdomains"

# Phase 3: Web Probing
echo "[Phase 3/5] Probing for web servers..."
cat "$WORKDIR/active_subs.txt" | httpx -silent -title -tech-detect -status-code -o "$WORKDIR/webservers.txt"
echo "  $(wc -l < "$WORKDIR/webservers.txt") web servers found"

# Phase 4: URL Discovery
echo "[Phase 4/5] Discovering URLs..."
while IFS= read -r sub; do
    gau "$sub" 2>/dev/null
done < "$WORKDIR/active_subs.txt" | uro > "$WORKDIR/urls.txt"
echo "  $(wc -l < "$WORKDIR/urls.txt") unique URLs"

# Phase 5: Vulnerability Scanning
echo "[Phase 5/5] Running nuclei templates..."
nuclei -l "$WORKDIR/webservers.txt" -o "$WORKDIR/nuclei_results.txt" 2>/dev/null

echo ""
echo "=== Pipeline Complete ==="
echo "Results in: $WORKDIR/"
```

#### Subfinder → GOWitness Screenshots

```bash
#!/bin/bash
# screenshot_pipeline.sh — Enumerate and screenshot subdomains

TARGET="${1:?Usage: $0 <domain>}"

# Enumerate
subfinder -d "$TARGET" -all -silent > subs.txt

# Resolve
massdns -r resolvers.txt -t A -o S subs.txt | grep " A " | \
    awk '{print $1}' | sed 's/\.$//' | sort -u > active.txt

# Probe
cat active.txt | httpx -silent > webservers.txt

# Screenshot
gowitness file -f webservers.txt -P screenshots/ --disable-redirects
```

### Python Integration

```python
#!/usr/bin/env python3
"""
subfinder_automation.py — Programmatic Subfinder integration
"""

import subprocess
import json
import os
from typing import List, Dict
from datetime import datetime

def run_subfinder(domain: str, use_all_sources: bool = False, 
                  silent: bool = True, threads: int = 30) -> List[str]:
    """Run subfinder and return discovered subdomains."""
    cmd = ["subfinder", "-d", domain, "-silent"]
    
    if use_all_sources:
        cmd.append("-all")
    if threads:
        cmd.extend(["-t", str(threads)])
    
    result = subprocess.run(cmd, capture_output=True, text=True, timeout=300)
    
    if result.returncode != 0:
        print(f"[!] Error running subfinder: {result.stderr}")
        return []
    
    subdomains = [line.strip() for line in result.stdout.splitlines() if line.strip()]
    return subdomains

def run_subfinder_json(domain: str) -> List[Dict]:
    """Run subfinder and return results as JSON."""
    cmd = ["subfinder", "-d", domain, "-oJ"]
    result = subprocess.run(cmd, capture_output=True, text=True, timeout=300)
    
    results = []
    for line in result.stdout.splitlines():
        if line.strip():
            try:
                results.append(json.loads(line))
            except json.JSONDecodeError:
                continue
    return results

def batch_enumerate(domains: List[str], output_dir: str = "subfinder_results"):
    """Enumerate subdomains for multiple domains."""
    os.makedirs(output_dir, exist_ok=True)
    all_results = {}
    
    for domain in domains:
        print(f"[*] Enumerating: {domain}")
        subdomains = run_subfinder(domain, use_all_sources=True)
        all_results[domain] = subdomains
        
        # Save individual results
        output_file = os.path.join(output_dir, f"{domain}.txt")
        with open(output_file, 'w') as f:
            f.write('\n'.join(subdomains) + '\n')
        
        print(f"    Found: {len(subdomains)} subdomains")
    
    # Save combined results
    combined = set()
    for subs in all_results.values():
        combined.update(subs)
    
    combined_file = os.path.join(output_dir, "all_subdomains.txt")
    with open(combined_file, 'w') as f:
        f.write('\n'.join(sorted(combined)) + '\n')
    
    print(f"\n[+] Total unique subdomains: {len(combined)}")
    return all_results

# Example usage
if __name__ == "__main__":
    domains = ["example.com", "google.com", "github.com"]
    results = batch_enumerate(domains)
    
    for domain, subs in results.items():
        print(f"{domain}: {len(subs)} subdomains")
```

Expected output:
```
[*] Enumerating: example.com
    Found: 12 subdomains
[*] Enumerating: google.com
    Found: 847 subdomains
[*] Enumerating: github.com
    Found: 234 subdomains

[+] Total unique subdomains: 1093
example.com: 12 subdomains
google.com: 847 subdomains
github.com: 234 subdomains
```

---

## Chapter 5: Advanced Usage

### API Key Optimization

```yaml
# ~/.config/subfinder/provider-config.yaml — Optimized configuration
# Order sources by reliability and speed

# Free sources (no API key needed, always available)
crtsh: []           # Certificate transparency — excellent coverage
hackertarget: []    # DNS enumeration — good for active domains
rapiddns: []        # DNS enumeration — fast
sitedossier: []     # DNS enumeration — moderate coverage

# Paid sources (much higher coverage)
shodan:
  - YOUR_SHODAN_KEY
censys:
  - YOUR_CENSYS_ID:YOUR_CENSYS_SECRET
virustotal:
  - YOUR_VT_KEY
securitytrails:
  - YOUR_ST_KEY
fullhunt:
  - YOUR_FH_KEY
binaryedge:
  - YOUR_BE_KEY
```

### Recursive Enumeration

```bash
# Deep recursive enumeration (may take longer but finds more)
subfinder -d target.com -recursive -all -timeout 60 -o deep_subs.txt

# Recursive with specific sources
subfinder -d target.com -recursive -sources crtsh,hackertarget -o rec_subs.txt
```

**Note:** Recursive enumeration can significantly increase result count but also increases time and may produce false positives.

### Rate Limiting

```bash
# Respect source rate limits
subfinder -d target.com -rate-limit 100 -t 10

# For large-scale operations, use lower rate limits
subfinder -dL domains.txt -rate-limit 50 -t 5
```

### Excluding Sources

```bash
# Exclude slow or unreliable sources
subfinder -d target.com -exclude-sources yahoo,bing

# Exclude sources that require API keys you don't have
subfinder -d target.com -exclude-sources shodan,censys,virustotal
```

### Maximum Time Limit

```bash
# Stop after 5 minutes regardless of remaining sources
subfinder -d target.com -all -max-time 300
```

### JSON Output Processing

```bash
# Get JSON output and parse with jq
subfinder -d target.com -oJ | jq -r '.host' | sort -u > subs.txt

# Extract source information
subfinder -d target.com -oJ | jq -r '[.host, .source] | @tsv' > subs_with_sources.txt

# Count results per source
subfinder -d target.com -oJ | jq -r '.source' | sort | uniq -c | sort -rn
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: Comprehensive Bug Bounty Recon

**Objective:** Build a complete subdomain list for a bug bounty program.

**Step 1: Run Subfinder with All Sources**
```bash
echo "[*] Phase 1: Subfinder enumeration..."
subfinder -d target.com -all -o subfinder_full.txt 2>/dev/null
echo "    Subfinder found: $(wc -l < subfinder_full.txt) subdomains"
```

**Step 2: Combine with Other Tools**
```bash
echo "[*] Phase 2: Combining with other tools..."
amass enum -passive -d target.com > amass_subs.txt 2>/dev/null
assetfinder --subs-only target.com > assetfinder_subs.txt 2>/dev/null

# Combine all results
cat subfinder_full.txt amass_subs.txt assetfinder_subs.txt | sort -u > all_subs.txt
echo "    Combined unique subdomains: $(wc -l < all_subs.txt)"
```

**Step 3: Resolve Active Subdomains**
```bash
echo "[*] Phase 3: Resolving subdomains..."
massdns -r resolvers.txt -t A -o S all_subs.txt > resolved.txt 2>/dev/null
grep " A " resolved.txt | awk '{print $1}' | sed 's/\.$//' | sort -u > active_subs.txt
echo "    Active subdomains: $(wc -l < active_subs.txt)"
```

**Step 4: Probe for Web Servers**
```bash
echo "[*] Phase 4: Probing web servers..."
cat active_subs.txt | httpx -silent -title -tech-detect -status-code > webservers.txt
echo "    Web servers: $(wc -l < webservers.txt)"
```

**Step 5: Screenshot and Report**
```bash
echo "[*] Phase 5: Taking screenshots..."
gowitness file -f webservers.txt -P screenshots/ 2>/dev/null

# Generate report
echo "=== Bug Bounty Recon Report: target.com ===" > report.md
echo "**Date:** $(date)" >> report.md
echo "**Total subdomains:** $(wc -l < all_subs.txt)" >> report.md
echo "**Active subdomains:** $(wc -l < active_subs.txt)" >> report.md
echo "**Web servers:** $(wc -l < webservers.txt)" >> report.md
```

### Scenario 2: CTF Challenge — "Hidden Subdomains"

**Objective:** Find a hidden subdomain that contains the flag.

**Step 1: Initial Enumeration**
```bash
subfinder -d ctf-target.com -all -silent | tee ctf_subs.txt
```

**Step 2: Check for Unusual Subdomains**
```bash
# Look for interesting patterns
grep -iE "(admin|flag|secret|hidden|internal|debug|test)" ctf_subs.txt

# Check for very long or unusual subdomains
awk 'length > 30' ctf_subs.txt
```

**Step 3: Probe Discovered Subdomains**
```bash
cat ctf_subs.txt | httpx -silent -title -status-code | grep -v "404"
```

**Step 4: Check for Hidden Content**
```bash
while IFS= read -r sub; do
    echo "--- $sub ---"
    curl -s -o /dev/null -w "%{http_code} %{redirect_url}" "http://$sub"
    echo ""
done < ctf_subs.txt
```

### Scenario 3: Corporate Asset Discovery

**Objective:** Map all web properties of a large corporation.

**Step 1: Enumerate Primary Domains**
```bash
# Start with known domains
echo "company.com" > primary_domains.txt
echo "company.co.uk" >> primary_domains.txt
echo "company.de" >> primary_domains.txt
```

**Step 2: Deep Enumeration**
```bash
while IFS= read -r domain; do
    echo "[*] Enumerating: $domain"
    subfinder -d "$domain" -all -recursive -o "subs_${domain}.txt"
done < primary_domains.txt

cat subs_*.txt | sort -u > corporate_subs.txt
```

**Step 3: Identify Patterns**
```bash
# Extract subdomain prefixes
cat corporate_subs.txt | sed "s/\.company\.com//g" | sort | uniq -c | sort -rn | head -20

# Find common patterns
grep -oP '^[a-z]+' corporate_subs.txt | sort | uniq -c | sort -rn | head -20
```

**Step 4: Generate Infrastructure Map**
```bash
# Resolve all subdomains
massdns -r resolvers.txt -t A -o S corporate_subs.txt > resolved.txt

# Group by IP
grep " A " resolved.txt | awk '{print $3, $1}' | sort > ip_mapping.txt

# Find shared hosting
awk '{print $1}' ip_mapping.txt | sort | uniq -c | sort -rn | head -20
```

---

## Chapter 7: Detection & Defense

### How Subfinder Activity Is Detected

**Important:** Subfinder is entirely passive and does NOT directly contact the target. Detection focuses on the data sources, not the target itself.

**Data Source Monitoring:**
- Certificate transparency log queries are logged by CT monitors
- Search engine queries may be logged by the search providers
- DNS database queries (SecurityTrails, etc.) are logged by those services
- API usage is tracked by source providers

**No Direct Target Impact:**
- Subfinder does not send any packets to the target domain
- The target cannot detect Subfinder reconnaissance directly
- Only the data sources know you're querying them

### Blue Team Defensive Measures

**1. Monitor Certificate Transparency Logs:**
```bash
# Subscribe to CT log notifications for your domains
# Use crt.sh to monitor: https://crt.sh/?q=%.example.com

# Automated CT monitoring script
#!/bin/bash
# ct_monitor.sh — Monitor certificate transparency for domain

DOMAIN="${1:?Usage: $0 <domain>}"

while true; do
    # Check for new certificates
    curl -s "https://crt.sh/?q=%.${DOMAIN}&output=json" | \
        jq -r '.[] | .name_value' | sort -u > current_certs.txt
    
    # Compare with previous scan
    if [ -f previous_certs.txt ]; then
        NEW=$(comm -13 previous_certs.txt current_certs.txt)
        if [ -n "$NEW" ]; then
            echo "[!] New certificates detected:"
            echo "$NEW"
            # Send alert
            logger -p security.info "CT: New certificate for ${DOMAIN}: ${NEW}"
        fi
    fi
    
    mv current_certs.txt previous_certs.txt
    sleep 3600  # Check hourly
done
```

**2. Limit DNS Record Information:**
```bash
# Minimize information in DNS records
# Avoid: internal hostnames, employee names, environment indicators
# Example of bad records:
# dev-john.internal.company.com
# staging-db.company.com
# vpn-backend.company.com

# Better:
# app-01.company.com
# db-01.company.com
# vpn.company.com
```

**3. Use Wildcard DNS Cautiously:**
```bash
# Wildcard DNS can hide subdomains but also creates noise
# Instead of wildcard, explicitly define needed records
# * IN A 1.2.3.4  # Bad — reveals wildcard
# www IN A 1.2.3.4  # Good — explicit
```

**4. Implement DNS Monitoring:**
```bash
#!/bin/bash
# dns_monitor.sh — Monitor for DNS enumeration attempts

# Monitor DNS query logs for enumeration patterns
tail -f /var/log/named/query.log | while read line; do
    # Detect sequential queries from same source
    SRC=$(echo "$line" | awk '{print $5}')
    DOMAIN=$(echo "$line" | awk '{print $7}')
    
    # Log potential enumeration
    echo "$(date) $SRC $DOMAIN" >> /var/log/dns_enum_watch.log
done

# Analyze for patterns
awk '{print $1}' /var/log/dns_enum_watch.log | sort | uniq -c | sort -rn | head -20
```

**5. Deploy Honeypot Subdomains:**
```bash
# Create fake subdomains that trigger alerts when accessed
# Add to DNS:
# canary.company.com IN A 10.0.0.99

# Monitor access to canary subdomains
tcpdump -i eth0 dst host 10.0.0.99 -l | while read line; do
    echo "ALERT: Canary subdomain accessed: $line"
    logger -p security.alert "Honeypot triggered: $line"
done
```

---

## Chapter 8: Troubleshooting

### Common Errors

#### "No results found"

**Cause:** No subdomains discovered, possibly due to limited sources or API keys.

**Solutions:**
```bash
# Try with all sources enabled
subfinder -d target.com -all

# Check API key configuration
cat ~/.config/subfinder/provider-config.yaml

# Try with specific reliable sources
subfinder -d target.com -sources crtsh,hackertarget,rapiddns

# Combine with other tools
amass enum -passive -d target.com > amass.txt
cat subfinder.txt amass.txt | sort -u > combined.txt
```

#### "Timeout" or "context deadline exceeded"

**Cause:** Source is slow to respond or network issues.

**Solutions:**
```bash
# Increase timeout
subfinder -d target.com -timeout 60

# Use faster sources only
subfinder -d target.com -sources crtsh,hackertarget

# Reduce threads to avoid overwhelming sources
subfinder -d target.com -t 10
```

#### "Rate limited" or "429 Too Many Requests"

**Cause:** API rate limits exceeded.

**Solutions:**
```bash
# Reduce request rate
subfinder -d target.com -rate-limit 50 -t 5

# Use different API keys
# Rotate between multiple Shodan/Censys accounts

# Use free sources that don't have rate limits
subfinder -d target.com -sources crtsh,hackertarget
```

#### "Permission denied" or "access denied"

**Cause:** API key issues or invalid credentials.

**Solutions:**
```bash
# Verify API keys
cat ~/.config/subfinder/provider-config.yaml

# Test API keys manually
curl -H "apikey: YOUR_KEY" "https://api.shodan.io/dns/domain/target.com"

# Reset API keys if needed
# Revoke and regenerate from provider dashboards
```

#### "Executable file not found" or "command not found"

**Cause:** Subfinder not installed or not in PATH.

**Solutions:**
```bash
# Check if installed
which subfinder
ls ~/go/bin/subfinder

# Add Go bin to PATH
export PATH=$PATH:~/go/bin

# Or install via APT
sudo apt install subfinder
```

#### "Too many open files"

**Cause:** System file descriptor limit reached with high thread counts.

**Solutions:**
```bash
# Reduce threads
subfinder -d target.com -t 10

# Increase file descriptor limit
ulimit -n 65535
```

### Performance Optimization

```bash
# Benchmark different configurations
#!/bin/bash
# benchmark_subfinder.sh — Find optimal settings

TARGET="example.com"
echo "=== Subfinder Benchmark ==="

for threads in 10 20 30 50; do
    echo -n "Threads=$threads: "
    START=$(date +%s%N)
    subfinder -d "$TARGET" -silent -t $threads > /dev/null 2>&1
    END=$(date +%s%N)
    ELAPSED=$(( (END - START) / 1000000 ))
    echo "${ELAPSED}ms"
done
```

### FAQ

**Q: How does Subfinder differ from Amass?**
A: Subfinder is faster and simpler, focused on passive enumeration. Amass offers both passive and active enumeration (including DNS brute-forcing and network mapping) but is slower.

**Q: Can Subfinder find all subdomains?**
A: No tool can find all subdomains — Subfinder only finds those indexed by its data sources. Combining multiple tools maximizes coverage.

**Q: Does Subfinder work with wildcard DNS?**
A: Subfinder can discover subdomains from wildcard DNS via certificate transparency and other sources, but it doesn't perform DNS brute-forcing.

**Q: Why are some subdomains not resolving?**
A: Subfinder finds historical subdomains that may no longer exist. Always validate with DNS resolution (massdns, httpx).

**Q: Can I use Subfinder without API keys?**
A: Yes, but with limited sources. Free sources like crtsh, hackertarget, and rapiddns provide good baseline coverage.

**Q: How do I update Subfinder?**
A: `go install -v github.com/projectdiscovery/subfinder/v2/cmd/subfinder@latest` or `sudo apt update && sudo apt install subfinder`

---

## Resources

- [Subfinder GitHub Repository](https://github.com/projectdiscovery/subfinder)
- [ProjectDiscovery Documentation](https://docs.projectdiscovery.io)
- [Subfinder Usage Guide](https://github.com/projectdiscovery/subfinder#usage)
- [Certificate Transparency — Wikipedia](https://en.wikipedia.org/wiki/Certificate_Transparency)
- [RFC 6962 — Certificate Transparency](https://datatracker.ietf.org/doc/html/rfc6962)
- [OWASP Subdomain Enumeration Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/SubdomainEnumerationCheatSheet.html)
- [Subdomain Enumeration Techniques — Bug Bounty Methodology](https://github.com/KathanP19/HowToRecon)
- [crt.sh — Certificate Search](https://crt.sh)
- [Kali Linux Subfinder Package](https://www.kali.org/tools/subfinder/)
- [ProjectDiscovery Blog](https://blog.projectdiscovery.io)
