---
tool_name: "sublist3r"
display_name: "Sublist3r"
mitre_tactic: "reconnaissance"
mitre_tactic_id: "TA0043"
mitre_subcategory: "web_scanning"
install_command: "pip3 install sublist3r"
version: "1.2"
github_repo: "https://github.com/aboul3la/Sublist3r"
kali_tools_page: "https://www.kali.org/tools/sublist3r/"
difficulty_level: "beginner"
requires_root: false
gui_available: false
tags: ["subdomain", "enumeration", "osint", "passive"]
related_tools: ["subfinder", "amass", "assetfinder"]
---

# Sublist3r

## Chapter 1: Introduction & Overview

### What is Sublist3r?

Sublist3r is a penetration testing and bug bounty tool that enumerates subdomains of websites using search engines (Google, Bing, Yahoo, Baidu), certificate transparency logs (crt.sh, Censys), and various other online services and DNS databases. Written in Python, it provides a simple command-line interface and also supports a basic GUI mode for users who prefer visual interfaces.

Sublist3r was one of the pioneering tools in passive subdomain enumeration and remains widely used due to its simplicity and reliability. While newer tools like Subfinder and Amass offer more features, Sublist3r's straightforward approach makes it an excellent choice for quick reconnaissance and for users who are new to subdomain enumeration.

### History & Background

- **Created:** 2015 by Ahmed Aboul-Ela (aboul3la)
- **Maintained:** Community contributors (less active than Subfinder)
- **License:** LGPLv3
- **Language:** Python 2/3
- **Latest version:** 1.2
- **Upstream repository:** https://github.com/aboul3la/Sublist3r

Sublist3r was one of the first widely-adopted passive subdomain enumeration tools. It popularized the technique of aggregating results from multiple search engines and online services. While its development has slowed compared to newer alternatives, it remains included in Kali Linux and is still actively used in the security community.

### When to Use This Tool

- **Passive subdomain enumeration:** Quick discovery without touching the target
- **Bug bounty reconnaissance:** Initial scope discovery for bug bounty programs
- **Penetration testing:** Build subdomain lists for further testing
- **OSINT investigations:** Map web infrastructure from public sources
- **CTF challenges:** Find hidden subdomains in capture-the-flag competitions
- **Learning:** Understanding subdomain enumeration concepts

### Legal and Ethical Considerations

- Passive technique using public data sources
- Does not directly interact with the target domain
- Uses search engines and public databases
- Results may be cached or outdated
- Always verify subdomains are within authorized scope
- Use for authorized security assessments only
- Respect search engine terms of service

### Key Concepts

| Concept | Description |
|---------|-------------|
| **Passive Enumeration** | Finding subdomains through third-party sources |
| **Search Engine Dorking** | Using advanced search operators to find subdomains |
| **Certificate Transparency** | Public logs of SSL/TLS certificates revealing subdomains |
| **Brute-force Mode** | Active subdomain guessing using wordlists (optional) |
| **DNS Resolution** | Verifying discovered subdomains have active DNS records |
| **Engine** | A data source that Sublist3r queries |

---

## Chapter 2: Installation & Setup

### System Requirements

- Operating system: Linux, macOS, or Windows
- RAM: 256 MB minimum
- Disk space: < 20 MB
- Python: 2.7+ or 3.x
- Network: Internet access required
- Privileges: Non-root is sufficient

### Installation via pip

```bash
pip3 install sublist3r
```

Expected output:
```
Collecting sublist3r
  Downloading Sublist3r-1.2.tar.gz (3.2 kB)
  Preparing metadata (setup.py) ... done
Collecting dnspython
  Downloading dnspython-2.3.0-py3-none-any.whl (612 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 612.1/612.1 kB 2.1 MB/s eta 0:00:00
Collecting requests
  Downloading requests-2.28.2-py3-none-any.whl (62 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 62.6/62.6 kB 1.8 MB/s eta 0:00:00
Building wheels for collected packages: sublist3r
  Building wheel for sublist3r (setup.py) ... done
  Created wheel for sublist3r: filename=Sublist3r-1.2-py3-none-any.whl size=4587 sha256=...
Successfully installed Sublist3r-1.2 dnspython-2.3.0 requests-2.28.2
```

### Installation from Source

```bash
# Clone repository
git clone https://github.com/aboul3la/Sublist3r.git
cd Sublist3r

# Install dependencies
pip3 install -r requirements.txt

# Make executable
chmod +x sublist3r.py

# Run directly
python3 sublist3r.py -d target.com
```

Expected output:
```
Collecting dnspython
  Using cached dnspython-2.3.0-py3-none-any.whl (612 kB)
Collecting requests
  Using cached requests-2.28.2-py3-none-any.whl (62 kB)
Installing collected packages: dnspython, requests
Successfully installed dnspython-2.3.0 requests-2.28.2
```

### APT Installation (Kali/Debian)

```bash
sudo apt update
sudo apt install sublist3r
```

Expected output:
```
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following NEW packages will be installed:
  sublist3r
0 upgraded, 1 newly installed, 0 to remove and 0 not upgraded.
Need to get 12.3 kB of archives.
After this operation, 67.8 kB of additional disk space will be used.
Get:1 http://http.kali.org/kali kali-rolling/main amd64 sublist3r 1.2-1kali1 [12.3 kB]
Fetched 12.3 kB in 0s (61.5 kB/s)
Selecting previously unselected package sublist3r.
(Reading database ... 1234567 files and directories currently installed.)
Preparing to unpack .../sublist3r_1.2-1kali1_all.deb ...
Unpacking sublist3r (1.2-1kali1) ...
Setting up sublist3r (1.2-1kali1) ...
```

### Docker Installation

```bash
docker pull aboul3la/sublist3r
docker run --rm aboul3la/sublist3r -d target.com
```

### Verification

```bash
sublist3r --version
```

Expected output:
```
Sublist3r v1.2
```

Or verify with:

```bash
sublist3r --help
```

---

## Chapter 3: Basic Usage

### First Run

```bash
sublist3r -d example.com
```

Sample output:
```
**11 subdomains found for example.com**
blog.example.com
docs.example.com
ftp.example.com
mail.example.com
mx.example.com
ns1.example.com
ns2.example.com
shop.example.com
staging.example.com
webmail.example.com
www.example.com
```

### Essential Commands

#### Basic Enumeration

```bash
sublist3r -d example.com
```

**Explanation:** Enumerates subdomains of example.com using all available search engines and data sources.

#### Verbose Output

```bash
sublist3r -v -d example.com
```

**Explanation:** Shows detailed output including which sources are being queried and their status.

Expected output:
```
[-] Searching in Baidu: 5 results
[-] Searching in Google: 12 results
[-] Searching in Bing: 8 results
[-] Searching in Yahoo: 3 results
[-] Searching in Ask: 2 results
[-] Searching in crt.sh: 15 results
[-] Searching in DNSdumpster: 7 results
[-] Searching in Netcraft: 4 results
[-] Searching in Threatminer: 6 results

**34 subdomains found for example.com**
```

#### Output to File

```bash
sublist3r -d example.com -o subdomains.txt
```

**Explanation:** Saves discovered subdomains to a file.

#### Silent Mode

```bash
sublist3r -d example.com -s
```

**Explanation:** Outputs only subdomains without banner or informational messages.

#### Brute-force Mode

```bash
sublist3r -b -d example.com
```

**Explanation:** Enables brute-force mode — actively guesses subdomains using a wordlist. This is NOT passive and will touch the target.

#### Scan Specific Ports

```bash
sublist3r -p 80,443 -d example.com
```

**Explanation:** After discovering subdomains, scans specified ports to verify which are active.

#### Specify Threads

```bash
sublist3r -t 10 -d example.com
```

**Explanation:** Uses 10 concurrent threads (default is 10).

#### Use Specific Engines

```bash
sublist3r -e google,bing,crtsh -d example.com
```

**Explanation:** Only uses specified search engines instead of all available ones.

### Complete Flags Reference

| Flag | Long Form | Description | Default | Example |
|------|-----------|-------------|---------|---------|
| `-d` | `--domain` | Target domain | none (required) | `sublist3r -d example.com` |
| `-b` | `--bruteforce` | Enable brute-force mode | off | `sublist3r -b -d example.com` |
| `-p` | `--ports` | Scan specific ports | none | `sublist3r -p 80,443 -d example.com` |
| `-o` | `--output` | Output file | stdout | `sublist3r -o out.txt -d example.com` |
| `-v` | `--verbose` | Verbose mode | off | `sublist3r -v -d example.com` |
| `-t` | `--threads` | Thread count | 10 | `sublist3r -t 10 -d example.com` |
| `-e` | `--engines` | Specify engines | all | `sublist3r -e google,bing -d example.com` |
| `-s` | `--silent` | Silent mode | off | `sublist3r -s -d example.com` |
| `--recursive` | `--recursive` | Recursive enumeration | off | `sublist3r --recursive -d example.com` |

### Available Engines/Sources

| Engine | Type | Description |
|--------|------|-------------|
| baidu | Search engine | Chinese search engine |
| bing | Search engine | Microsoft search engine |
| google | Search engine | Google search engine |
| yahoo | Search engine | Yahoo search engine |
| ask | Search engine | Ask.com search engine |
| threatminer | DNS database | Threat intelligence DNS data |
| netcraft | DNS database | Netcraft DNS survey |
| crtsh | Certificate transparency | crt.sh certificate search |
| dnsdumpster | DNS recon | DNS recon and research tool |
| virustotal | Threat intelligence | VirusTotal DNS data |
| bufferover | Certificate transparency | TLS certificate transparency |

---

## Chapter 4: Intermediate Usage

### Bash Automation Scripts

#### Multi-Engine Enumeration

```bash
#!/bin/bash
# multi_engine_enum.sh — Enumerate with all engines individually for comparison

DOMAIN="${1:?Usage: $0 <domain>}"
ENGINES=("google" "bing" "yahoo" "baidu" "ask" "crtsh" "threatminer" "netcraft" "dnsdumpster")
OUTPUT_DIR="sublist3r_results/$(date +%Y%m%d)"
mkdir -p "$OUTPUT_DIR"

echo "=== Multi-Engine Enumeration: $DOMAIN ==="

for engine in "${ENGINES[@]}"; do
    echo -n "[*] $engine: "
    sublist3r -d "$DOMAIN" -e "$engine" -s > "$OUTPUT_DIR/${engine}.txt" 2>/dev/null
    COUNT=$(wc -l < "$OUTPUT_DIR/${engine}.txt")
    echo "$COUNT subdomains"
done

# Combine all results
cat "$OUTPUT_DIR"/*.txt | sort -u > "$OUTPUT_DIR/combined.txt"
echo ""
echo "[+] Total unique subdomains: $(wc -l < "$OUTPUT_DIR/combined.txt")"
```

#### Filtered Enumeration

```bash
#!/bin/bash
# filtered_enum.sh — Enumerate and filter for interesting subdomains

DOMAIN="${1:?Usage: $0 <domain>}"

echo "[*] Running Sublist3r..."
sublist3r -d "$DOMAIN" -s > raw_subs.txt

echo "[*] Filtering for interesting subdomains..."
grep -iE "(admin|api|dev|staging|test|internal|portal|backup|vpn|mail|webmail|ftp)" raw_subs.txt > interesting_subs.txt

echo "[*] Filtering for active subdomains..."
# Simple HTTP check
while IFS= read -r sub; do
    STATUS=$(curl -s -o /dev/null -w "%{http_code}" --max-time 3 "http://$sub" 2>/dev/null)
    if [ "$STATUS" != "000" ]; then
        echo "$sub (HTTP $STATUS)"
    fi
done < raw_subs.txt > active_subs.txt

echo ""
echo "=== Results ==="
echo "Total subdomains: $(wc -l < raw_subs.txt)"
echo "Interesting subdomains: $(wc -l < interesting_subs.txt)"
echo "Active subdomains: $(wc -l < active_subs.txt)"
```

### Tool Chaining

#### Sublist3r → Subfinder → MassDNS Pipeline

```bash
#!/bin/bash
# combined_enum.sh — Maximize subdomain discovery

TARGET="${1:?Usage: $0 <domain>}"

echo "[*] Running Sublist3r..."
sublist3r -d "$TARGET" -s > sublist3r_raw.txt

echo "[*] Running Subfinder..."
subfinder -d "$TARGET" -silent > subfinder_raw.txt

echo "[*] Running Amass (passive)..."
amass enum -passive -d "$TARGET" > amass_raw.txt 2>/dev/null

echo "[*] Combining results..."
cat sublist3r_raw.txt subfinder_raw.txt amass_raw.txt | sort -u > all_subs.txt
echo "    Total unique: $(wc -l < all_subs.txt)"

echo "[*] Resolving..."
massdns -r resolvers.txt -t A -o S all_subs.txt > resolved.txt 2>/dev/null
grep " A " resolved.txt | awk '{print $1}' | sed 's/\.$//' | sort -u > active.txt
echo "    Active: $(wc -l < active.txt)"
```

#### Sublist3r → httpx → Nuclei Pipeline

```bash
#!/bin/bash
# vuln_pipeline.sh — Enumerate and scan for vulnerabilities

TARGET="${1:?Usage: $0 <domain>}"

# Enumerate
sublist3r -d "$TARGET" -s > subs.txt

# Probe
cat subs.txt | httpx -silent > webservers.txt

# Scan
nuclei -l webservers.txt -o results.txt
```

### Python Integration

```python
#!/usr/bin/env python3
"""
sublist3r_automation.py — Programmatic Sublist3r integration
"""

import subprocess
import json
from typing import List, Dict

def run_sublist3r(domain: str, engines: List[str] = None, 
                  brute_force: bool = False, verbose: bool = True) -> List[str]:
    """Run Sublist3r and return discovered subdomains."""
    cmd = ["sublist3r", "-d", domain, "-s"]
    
    if engines:
        cmd.extend(["-e", ",".join(engines)])
    if brute_force:
        cmd.append("-b")
    if verbose:
        cmd.insert(1, "-v")
    
    result = subprocess.run(cmd, capture_output=True, text=True, timeout=300)
    
    subdomains = []
    for line in result.stdout.splitlines():
        line = line.strip()
        if line and not line.startswith(('*', '-', '[', '=')):
            subdomains.append(line)
    
    return list(set(subdomains))

def batch_enumerate(domains: List[str]) -> Dict[str, List[str]]:
    """Enumerate subdomains for multiple domains."""
    results = {}
    for domain in domains:
        print(f"[*] Enumerating: {domain}")
        subs = run_sublist3r(domain)
        results[domain] = subs
        print(f"    Found: {len(subs)} subdomains")
    return results

# Example usage
domains = ["example.com", "google.com", "github.com"]
results = batch_enumerate(domains)

for domain, subs in results.items():
    print(f"\n{domain}:")
    for sub in subs[:10]:
        print(f"  {sub}")
    if len(subs) > 10:
        print(f"  ... and {len(subs) - 10} more")
```

---

## Chapter 5: Advanced Usage

### Custom Wordlist for Brute-force Mode

```bash
# Create custom wordlist
cat > custom_wordlist.txt << 'EOF'
admin
administrator
api
backup
beta
blog
cdn
cpanel
dashboard
db
dev
devops
docs
email
ftp
git
help
internal
lab
mail
media
mobile
monitor
news
ns1
ns2
portal
preprod
private
proxy
sandbox
shop
stage
staging
status
support
test
upload
vpn
webdisk
webmail
wiki
www
EOF

# Use custom wordlist with brute-force
sublist3r -b -d target.com -w custom_wordlist.txt
```

### Recursive Enumeration

```bash
# Enumerate subdomains of discovered subdomains
sublist3r --recursive -d target.com -o recursive_subs.txt
```

**Note:** Recursive enumeration can significantly increase the number of results but also increases time and potential false positives.

### Engine-Specific Searches

```bash
# Only certificate transparency (most reliable passive source)
sublist3r -e crtsh -d target.com

# Only search engines
sublist3r -e google,bing,yahoo -d target.com

# Only DNS databases
sublist3r -e threatminer,netcraft,dnsdumpster -d target.com
```

### Integration with Other Tools

```bash
# Combine with other enumeration tools
(sublist3r -d target.com -s; subfinder -d target.com -silent; amass enum -passive -d target.com) | sort -u > all_subs.txt

# Feed to mass resolution
sublist3r -d target.com -s | massdns -r resolvers.txt -t A -o S > resolved.txt

# Feed to port scanner
sublist3r -d target.com -s | httpx -silent | nmap -iL - -p 80,443
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: Quick Bug Bounty Recon

**Objective:** Fast subdomain discovery for a bug bounty program.

**Step 1: Run Sublist3r with All Sources**
```bash
sublist3r -d target.com -o sublist3r_results.txt
echo "[+] Sublist3r found $(wc -l < sublist3r_results.txt) subdomains"
```

**Step 2: Enrich with Other Tools**
```bash
subfinder -d target.com -silent > subfinder_results.txt
cat sublist3r_results.txt subfinder_results.txt | sort -u > combined.txt
echo "[+] Combined: $(wc -l < combined.txt) unique subdomains"
```

**Step 3: Quick Port Check**
```bash
sublist3r -d target.com -p 80,443,8080,8443 -o port_checked.txt
```

**Step 4: Generate Report**
```bash
cat > bugbounty_recon.txt << EOF
=== Bug Bounty Recon: target.com ===
Date: $(date)

Sublist3r results: $(wc -l < sublist3r_results.txt)
Subfinder results: $(wc -l < subfinder_results.txt)
Combined unique: $(wc -l < combined.txt)

Subdomains:
$(cat combined.txt)
EOF
```

### Scenario 2: CTF Challenge — "Hidden Subdomains"

**Objective:** Find hidden subdomains containing flags.

**Step 1: Passive Enumeration**
```bash
sublist3r -v -d ctf-target.com 2>&1 | tee ctf_subs.txt
```

**Step 2: Analyze Results**
```bash
# Look for unusual subdomains
grep -iE "(flag|secret|hidden|internal|debug)" ctf_subs.txt

# Check for non-standard ports
sublist3r -p 80,443,8080,3000,5000,8000,8888,9000 -d ctf-target.com
```

**Step 3: Brute-force if Needed**
```bash
# Create custom wordlist based on challenge hints
cat > ctf_wordlist.txt << 'EOF'
flag
admin
secret
hidden
root
internal
debug
test
EOF

sublist3r -b -d ctf-target.com -w ctf_wordlist.txt
```

**Step 4: Verify Findings**
```bash
while IFS= read -r sub; do
    echo -n "$sub: "
    curl -s -o /dev/null -w "%{http_code}" "http://$sub" 2>/dev/null
    echo ""
done < ctf_subs.txt
```

### Scenario 3: Corporate Subdomain Audit

**Objective:** Audit all subdomains of a corporate domain for security.

**Step 1: Comprehensive Enumeration**
```bash
# Run all tools
sublist3r -d company.com -o sublist3r.txt
subfinder -d company.com -all -silent > subfinder.txt
amass enum -passive -d company.com > amass.txt

# Combine
cat sublist3r.txt subfinder.txt amass.txt | sort -u > all_subs.txt
```

**Step 2: Resolve and Verify**
```bash
massdns -r resolvers.txt -t A -o S all_subs.txt > resolved.txt
grep " A " resolved.txt | awk '{print $1}' | sed 's/\.$//' | sort -u > active_subs.txt
```

**Step 3: Categorize Findings**
```bash
# Categorize by prefix
echo "=== Subdomain Categories ==="
echo "Admin panels:"
grep -i "admin" active_subs.txt || echo "  None"

echo "Development/Staging:"
grep -iE "(dev|staging|test|beta|preprod)" active_subs.txt || echo "  None"

echo "Internal services:"
grep -iE "(internal|intranet|vpn|remote)" active_subs.txt || echo "  None"

echo "Email services:"
grep -iE "(mail|webmail|smtp|imap)" active_subs.txt || echo "  None"

echo "Total active: $(wc -l < active_subs.txt)"
```

---

## Chapter 7: Detection & Defense

### How Sublist3r Activity Is Detected

**Search Engine Query Patterns:**
- Sublist3r queries Google, Bing, Yahoo with site: operator
- High volume of site: queries may be flagged by search engines
- Search engines may return CAPTCHAs for automated queries

**No Direct Target Impact:**
- Sublist3r is passive — it does NOT directly contact the target
- The target cannot detect Sublist3r reconnaissance
- Only search engines and data sources see the queries

**Data Source Logging:**
- Search engines log all queries
- Certificate transparency logs are public
- DNS databases log API requests

### Blue Team Defensive Measures

**1. Monitor Certificate Transparency Logs:**
```bash
# Subscribe to CT log alerts for your domain
# Check crt.sh periodically
curl -s "https://crt.sh/?q=%.company.com&output=json" | \
    jq -r '.[] | .name_value' | sort -u > current_certs.txt
```

**2. Limit Subdomain Information:**
```bash
# Avoid exposing internal naming conventions
# Bad: dev-john.internal.company.com
# Good: app-01.company.com

# Minimize DNS records
# Remove stale/unnecessary subdomain records
```

**3. Search Engine Monitoring:**
```bash
# Monitor what search engines know about your domains
# Google: site:company.com
# Check for unexpected subdomains appearing in search results
```

**4. DNS Monitoring:**
```bash
# Monitor DNS query patterns for enumeration
grep "company.com" /var/log/named/query.log | \
    awk '{print $5}' | sort | uniq -c | sort -rn | head -20
```

**5. Proactive Security:**
```bash
# Regularly audit your own subdomains
# Use Sublist3r yourself to see what's exposed
sublist3r -d company.com -o self_audit.txt
cat self_audit.txt  # Review for unexpected subdomains
```

---

## Chapter 8: Troubleshooting

### Common Errors

#### "No results found"

**Cause:** Domain has very few subdomains, or search engines are blocking requests.

**Solutions:**
```bash
# Try with different engines
sublist3r -e crtsh,threatminer -d target.com

# Enable brute-force mode
sublist3r -b -d target.com

# Combine with other tools
subfinder -d target.com -silent > subs.txt
amass enum -passive -d target.com >> subs.txt
cat subs.txt | sort -u > combined.txt
```

#### "Import error" or "ModuleNotFoundError"

**Cause:** Missing Python dependencies.

**Solutions:**
```bash
# Reinstall dependencies
pip3 install -r requirements.txt

# Or install individually
pip3 install dnspython requests

# Check Python version
python3 --version  # Need 2.7+ or 3.x
```

#### "Connection reset" or "Timeout"

**Cause:** Search engines blocking automated queries.

**Solutions:**
```bash
# Reduce threads
sublist3r -t 5 -d target.com

# Use fewer engines
sublist3r -e crtsh -d target.com

# Wait and retry later
sleep 60 && sublist3r -d target.com
```

#### "SSL: CERTIFICATE_VERIFY_FAILED"

**Cause:** SSL certificate verification failure.

**Solutions:**
```bash
# Update certificates
sudo apt install ca-certificates

# Or use Python to update
pip3 install --upgrade certifi

# If using proxy, add certificate
export REQUESTS_CA_BUNDLE=/path/to/cert.pem
```

#### "Command not found"

**Cause:** Sublist3r not installed or not in PATH.

**Solutions:**
```bash
# Check installation
pip3 list | grep sublist3r

# Reinstall
pip3 install sublist3r

# Or install from source
git clone https://github.com/aboul3la/Sublist3r.git
cd Sublist3r
pip3 install -r requirements.txt
python3 sublist3r.py -d target.com
```

### Performance Optimization

```bash
# For faster results, use specific engines
sublist3r -e crtsh,google,bing -d target.com

# Reduce threads if hitting rate limits
sublist3r -t 5 -d target.com

# Run in background for long operations
sublist3r -d target.com -o results.txt &
```

### FAQ

**Q: Sublist3r vs Subfinder — which is better?**
A: Subfinder is more actively maintained, faster, and supports more sources. Sublist3r is simpler and good for quick scans. For serious recon, prefer Subfinder.

**Q: Can Sublist3r find all subdomains?**
A: No tool can find all subdomains. Sublist3r only finds those indexed by its data sources. Combining multiple tools maximizes coverage.

**Q: Does brute-force mode work well?**
A: Brute-force mode is basic. For serious brute-forcing, use specialized tools like Subfinder with a large wordlist or Gobuster.

**Q: Why are some results duplicates?**
A: Multiple engines may return the same subdomains. Sublist3r deduplicates within its output, but when combining with other tools, use `sort -u`.

**Q: Can Sublist3r handle large domains?**
A: Yes, but it may take longer for domains with many subdomains. Consider using Subfinder for large-scale enumeration.

**Q: How do I update Sublist3r?**
A: `pip3 install --upgrade sublist3r` or `git pull` in the source directory.

---

## Resources

- [Sublist3r GitHub Repository](https://github.com/aboul3la/Sublist3r)
- [Sublist3r Documentation](https://github.com/aboul3la/Sublist3r#readme)
- [Subfinder — Recommended Alternative](https://github.com/projectdiscovery/subfinder)
- [Amass — OWASP Subdomain Enumeration](https://github.com/owasp-amass/amass)
- [Certificate Transparency — Wikipedia](https://en.wikipedia.org/wiki/Certificate_Transparency)
- [OWASP Subdomain Enumeration](https://cheatsheetseries.owasp.org/cheatsheets/SubdomainEnumerationCheatSheet.html)
- [crt.sh — Certificate Search](https://crt.sh)
- [DNSdumpster — DNS Recon](https://dnsdumpster.com)
- [Kali Linux Sublist3r Package](https://www.kali.org/tools/sublist3r/)
- [Bug Bounty Methodology — KathanP19](https://github.com/KathanP19/HowToRecon)
