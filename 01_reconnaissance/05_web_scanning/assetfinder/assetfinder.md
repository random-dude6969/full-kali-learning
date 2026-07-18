---
tool_name: "assetfinder"
display_name: "Assetfinder"
mitre_tactic: "reconnaissance"
mitre_tactic_id: "TA0043"
mitre_subcategory: "web_scanning"
install_command: "sudo apt install assetfinder"
version: "0.4.1"
github_repo: "https://github.com/tomnomnom/assetfinder"
kali_tools_page: "https://www.kali.org/tools/assetfinder/"
difficulty_level: "beginner"
requires_root: false
gui_available: false
tags: ["web", "subdomain-discovery", "asset-discovery", "recon"]
related_tools: ["subfinder", "findomain", "amass", "sublist3r"]
---

# Assetfinder

## Chapter 1: Introduction and Overview

### What is Assetfinder?

Assetfinder is a fast, lightweight command-line tool for discovering assets (subdomains, IPs, related domains) associated with a given domain. It aggregates results from multiple passive and active data sources to provide comprehensive asset discovery with minimal configuration. Written in Go, it's designed for speed and simplicity.

### History and Background

- **Created:** 2017 by Tom Hudson (tomnomnom)
- **Maintained:** tomnomnom on GitHub
- **License:** MIT License
- **Language:** Go
- **Purpose:** Passive and active subdomain and asset discovery

Assetfinder was created as a simpler, faster alternative to existing subdomain enumeration tools. Tom nomnom (known for his contributions to the bug bounty community and tools like httpx, waybackurls, and gau) designed assetfinder to be a quick, no-nonsense tool that can be easily integrated into recon pipelines.

### When to Use This Tool

- **Bug bounty reconnaissance:** Discover all subdomains of a target organization
- **Penetration testing:** Map the complete attack surface by finding all assets
- **Asset inventory:** Maintain an inventory of all domains and subdomains
- **Third-party risk assessment:** Discover assets belonging to a target organization
- **Brand protection:** Find unauthorized subdomains or related domains
- **Red team operations:** Identify external-facing assets

### When NOT to Use It

- When you need DNS resolution (use subfinder or amass for that)
- When you need detailed host information (use amass or subfinder)
- Against targets without written authorization
- When you need active DNS brute-forcing (use amass for that)

### Legal and Ethical Considerations

- Always obtain written authorization before enumerating assets
- Passive reconnaissance is generally legal but verify local laws
- Be aware that some data sources may log your queries
- Document all findings and report responsibly
- Respect scope boundaries defined in your engagement

### Key Concepts and Terminology

- **Subdomain:** A domain that is part of a larger domain (e.g., mail.example.com is a subdomain of example.com)
- **Asset:** Any internet-facing resource associated with a target (domains, subdomains, IPs, certificates)
- **Passive Enumeration:** Gathering information without directly interacting with the target
- **Certificate Transparency (CT):** Public logs of SSL/TLS certificates that reveal subdomains
- **DNS Zone Transfer:** A misconfigured DNS server that reveals all records
- **TLD (Top-Level Domain):** The highest level in the DNS hierarchy (e.g., .com, .org, .net)

---

## Chapter 2: Installation and Setup

### System Requirements

- **Go:** 1.16 or higher (if building from source)
- **RAM:** 128 MB minimum
- **Disk Space:** ~10 MB
- **Network:** Internet connection for API queries
- **Privileges:** User-level (no root required)

### Installation Methods

#### Method 1: APT (Recommended)
```bash
sudo apt update
sudo apt install assetfinder
```

#### Method 2: Go Install
```bash
go install github.com/tomnomnom/assetfinder@latest
```

#### Method 3: From Source
```bash
git clone https://github.com/tomnomnom/assetfinder.git
cd assetfinder
go build -o assetfinder
sudo cp assetfinder /usr/local/bin/
```

#### Method 4: Binary Download
```bash
# Download latest release
wget https://github.com/tomnomnom/assetfinder/releases/latest/download/assetfinder_linux_amd64.tar.gz
tar -xzf assetfinder_linux_amd64.tar.gz
chmod +x assetfinder
sudo mv assetfinder /usr/local/bin/
```

### Verification Commands

```bash
# Verify installation
assetfinder --version
# Expected output: assetfinder v0.4.1

# Quick test
assetfinder example.com
# Should return results (or empty if no subdomains found)
```

---

## Chapter 3: Basic Usage

### First Scan

```bash
assetfinder example.com
```

**Sample Output:**
```
www.example.com
mail.example.com
ftp.example.com
dev.example.com
staging.example.com
api.example.com
```

### Essential Commands

#### 1. Basic Subdomain Discovery
```bash
assetfinder example.com
```
**Explanation:** Discovers all subdomains using passive data sources.

#### 2. Only Subdomains (Exclude Related)
```bash
assetfinder --subs-only example.com
```
**Explanation:** Returns only subdomains of the target, not related domains.

#### 3. Include Related Domains
```bash
assetfinder example.com
```
**Explanation:** Returns both subdomains and related domains (default behavior).

#### 4. Pipe Output to File
```bash
assetfinder example.com > subdomains.txt
```
**Explanation:** Saves results to a file for further processing.

#### 5. Count Results
```bash
assetfinder example.com | wc -l
```
**Explanation:** Shows how many subdomains were found.

#### 6. Sort and Deduplicate
```bash
assetfinder example.com | sort -u > unique_subdomains.txt
```
**Explanation:** Removes duplicates and sorts alphabetically.

#### 7. Filter by Keyword
```bash
assetfinder example.com | grep -i "api"
```
**Explanation:** Filters results containing "api".

#### 8. Check Live Subdomains
```bash
assetfinder example.com | httpx -silent
```
**Explanation:** Filters to only live subdomains using httpx.

#### 9. Resolve to IP Addresses
```bash
assetfinder example.com | xargs -I {} dig +short {}
```
**Explanation:** Resolves each subdomain to its IP address.

#### 10. Combine with Waybackurls
```bash
assetfinder example.com | waybackurls | sort -u
```
**Explanation:** Gets historical URLs for all discovered subdomains.

### COMPLETE Flag Reference

| Flag | Description | Example |
|------|-------------|---------|
| `--subs-only` | Show only subdomains (exclude related) | `assetfinder --subs-only example.com` |
| `--include-related` | Include related domains (default) | `assetfinder --include-related example.com` |
| `-h` / `--help` | Show help message | `assetfinder -h` |

**Note:** Assetfinder is deliberately simple with minimal flags. Most filtering is done via standard Unix tools (grep, sort, uniq, etc.).

### Output Format Explanation

Assetfinder outputs one domain per line:

```
www.example.com
mail.example.com
dev.example.com
```

- **Subdomains:** Direct children of the target domain (e.g., `www.example.com`)
- **Related Domains:** Domains that share infrastructure or certificates with the target (e.g., `example-api.com`)

---

## Chapter 4: Intermediate Usage

### Bash Scripting and Automation

#### Multi-Domain Recon Script
```bash
#!/bin/bash
# multi_domain_recon.sh - Discover subdomains for multiple domains

DOMAINS_FILE="domains.txt"
OUTPUT_DIR="recon_results"
mkdir -p $OUTPUT_DIR

while IFS= read -r domain; do
    echo "[*] Scanning $domain"

    # Assetfinder
    assetfinder "$domain" > "$OUTPUT_DIR/${domain}_assetfinder.txt"

    # Count results
    count=$(wc -l < "$OUTPUT_DIR/${domain}_assetfinder.txt")
    echo "[+] Found $count assets for $domain"

done < "$DOMAINS_FILE"

echo "[+] All scans complete"
```

#### Automated Recon Pipeline
```bash
#!/bin/bash
# full_recon.sh - Complete reconnaissance pipeline

TARGET=$1

echo "=========================================="
echo "[*] Reconnaissance Pipeline for: $TARGET"
echo "=========================================="

# Phase 1: Subdomain Discovery
echo ""
echo "[Phase 1] Subdomain Discovery"
echo "-------------------------------------------"
assetfinder "$TARGET" > assetfinder_results.txt
echo "[+] Assetfinder: $(wc -l < assetfinder_results.txt) results"

# Phase 2: Filter Live Hosts
echo ""
echo "[Phase 2] Live Host Detection"
echo "-------------------------------------------"
cat assetfinder_results.txt | httpx -silent -o live_hosts.txt
echo "[+] Live hosts: $(wc -l < live_hosts.txt) results"

# Phase 3: Port Scanning
echo ""
echo "[Phase 3] Port Scanning"
echo "-------------------------------------------"
nmap -iL live_hosts.txt -T4 --min-rate 1000 -oG nmap_results.txt

# Phase 4: Directory Discovery
echo ""
echo "[Phase 4] Directory Discovery"
echo "-------------------------------------------"
while IFS= read -r url; do
    feroxbuster -u "$url" -q -o "dirs_${url##*/}.txt"
done < live_hosts.txt

echo ""
echo "=========================================="
echo "[+] Reconnaissance complete for $TARGET"
echo "=========================================="
```

### Python Scripting Examples

#### Assetfinder Wrapper with JSON Output
```python
#!/usr/bin/env python3
import subprocess
import json
import sys

def assetfinder_scan(domain, subs_only=False):
    """Run assetfinder and return results as list."""
    cmd = ["assetfinder"]
    if subs_only:
        cmd.append("--subs-only")
    cmd.append(domain)

    result = subprocess.run(cmd, capture_output=True, text=True)

    if result.returncode != 0:
        print(f"Error: {result.stderr}", file=sys.stderr)
        return []

    return sorted(set(result.stdout.strip().split('\n')))

def main():
    if len(sys.argv) < 2:
        print(f"Usage: {sys.argv[0]} <domain> [--subs-only]")
        sys.exit(1)

    domain = sys.argv[1]
    subs_only = "--subs-only" in sys.argv

    results = assetfinder_scan(domain, subs_only)

    output = {
        "domain": domain,
        "subdomains_only": subs_only,
        "count": len(results),
        "results": results
    }

    print(json.dumps(output, indent=2))

    # Save to file
    with open(f"{domain}_assets.json", "w") as f:
        json.dump(output, f, indent=2)

if __name__ == "__main__":
    main()
```

#### Comprehensive Recon Script
```python
#!/usr/bin/env python3
import subprocess
import json
import concurrent.futures

def run_assetfinder(domain):
    """Run assetfinder on a domain."""
    result = subprocess.run(
        ["assetfinder", "--subs-only", domain],
        capture_output=True, text=True
    )
    return {
        "domain": domain,
        "subdomains": result.stdout.strip().split('\n') if result.stdout else []
    }

def main():
    domains = ["example.com", "example.org", "example.net"]

    with concurrent.futures.ThreadPoolExecutor(max_workers=3) as executor:
        futures = {executor.submit(run_assetfinder, d): d for d in domains}
        results = []

        for future in concurrent.futures.as_completed(futures):
            results.append(future.result())

    # Combine all results
    all_subdomains = []
    for r in results:
        all_subdomains.extend(r["subdomains"])
        print(f"[+] {r['domain']}: {len(r['subdomains'])} subdomains")

    # Deduplicate and save
    unique = sorted(set(all_subdomains))
    with open("all_subdomains.txt", "w") as f:
        f.write('\n'.join(unique))

    print(f"\n[+] Total unique subdomains: {len(unique)}")

if __name__ == "__main__":
    main()
```

### Tool Chaining

#### Assetfinder + httpx Pipeline
```bash
# Find subdomains and check which are live
assetfinder example.com | httpx -silent -status-code -title -o live_results.txt
```

#### Assetfinder + nmap Integration
```bash
# Discover subdomains, then port scan
assetfinder example.com | httpx -silent | nmap -iL - -T4 -oA recon_results
```

#### Assetfinder + feroxbuster
```bash
# Find live subdomains, then brute-force directories
assetfinder example.com | httpx -silent | while IFS= read -r url; do
    feroxbuster -u "$url" -q -o "dirs_${url##*/}.txt"
done
```

### Output Processing and Filtering

#### Extract Only Unique Domains
```bash
assetfinder example.com | sort -u > unique.txt
```

#### Filter by TLD
```bash
assetfinder example.com | grep '\.com$' > com_domains.txt
```

#### Remove Wildcard Subdomains
```bash
assetfinder example.com | grep -v '^\*' | sort -u
```

#### Count by Subdomain Level
```bash
assetfinder example.com | awk -F. '{print NF-2}' | sort | uniq -c
```

---

## Chapter 5: Advanced Usage

### Custom Wordlists and Configurations

#### Combine with Other Tools
```bash
# Merge results from multiple tools
assetfinder example.com > assetfinder.txt
subfinder -d example.com -silent > subfinder.txt
amass enum -passive -d example.com > amass.txt

# Combine and deduplicate
cat assetfinder.txt subfinder.txt amass.txt | sort -u > all_subdomains.txt
echo "[+] Total unique subdomains: $(wc -l < all_subdomains.txt)"
```

#### Certificate Transparency Logs
```bash
# Use crt.sh directly (assetfinder uses this internally)
curl -s "https://crt.sh/?q=%.example.com&output=json" | \
  python3 -c "
import sys, json
data = json.load(sys.stdin)
for entry in data:
    print(entry['name_value'])
" | sort -u > ct_results.txt
```

### Evasion Techniques

#### Rate Limiting and Delays
```bash
# Use with delay to avoid detection
assetfinder example.com | while IFS= read -r sub; do
    echo "$sub"
    sleep 0.1  # 100ms delay between lookups
done
```

#### Proxy Routing
```bash
# Route through proxy (for API-based lookups)
export HTTP_PROXY=http://127.0.0.1:8080
assetfinder example.com
```

### Integration with Pentest Workflows

#### Full Recon Workflow
```bash
#!/bin/bash
# complete_recon.sh - Full reconnaissance workflow

TARGET=$1
mkdir -p "$TARGET/recon"

echo "[Phase 1] Subdomain Discovery..."
assetfinder "$TARGET" > "$TARGET/recon/assetfinder_raw.txt"
subfinder -d "$TARGET" -silent > "$TARGET/recon/subfinder_raw.txt"

echo "[Phase 2] Deduplication..."
cat "$TARGET/recon/assetfinder_raw.txt" "$TARGET/recon/subfinder_raw.txt" | \
  sort -u > "$TARGET/recon/all_subdomains.txt"
echo "[+] $(wc -l < "$TARGET/recon/all_subdomains.txt") unique subdomains"

echo "[Phase 3] Live Host Detection..."
httpx -l "$TARGET/recon/all_subdomains.txt" -silent -o "$TARGET/recon/live_hosts.txt"
echo "[+] $(wc -l < "$TARGET/recon/live_hosts.txt") live hosts"

echo "[Phase 4] Port Scanning..."
nmap -iL "$TARGET/recon/live_hosts.txt" -T4 --min-rate 1000 \
  -p 80,443,8080,8443 -oG "$TARGET/recon/nmap_quick.txt"

echo "[+] Reconnaissance complete for $TARGET"
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: Web Application Security Audit

**Objective:** Discover all subdomains and assets of a target organization.

```bash
# Step 1: Subdomain discovery
assetfinder target.com > assetfinder_raw.txt

# Step 2: Merge with other tools
subfinder -d target.com -silent > subfinder_raw.txt
amass enum -passive -d target.com > amass_raw.txt

# Step 3: Deduplicate
cat assetfinder_raw.txt subfinder_raw.txt amass_raw.txt | sort -u > all_subdomains.txt

# Step 4: Verify live hosts
httpx -l all_subdomains.txt -silent -o live_hosts.txt

# Step 5: Port scan
nmap -iL live_hosts.txt -T4 --min-rate 1000 -oA nmap_results

echo "[+] Found $(wc -l < all_subdomains.txt) total subdomains"
echo "[+] Found $(wc -l < live_hosts.txt) live hosts"
```

### Scenario 2: CTF Challenge Walkthrough

**Objective:** Find hidden subdomains that host the flag.

```bash
# Step 1: Run assetfinder
assetfinder ctf-target.com

# Output:
# admin.ctf-target.com
# staging.ctf-target.com
# dev.ctf-target.com

# Step 2: Check each subdomain
for sub in admin staging dev; do
    echo "[*] Checking ${sub}.ctf-target.com"
    curl -s -o /dev/null -w "%{http_code}" "http://${sub}.ctf-target.com"
    echo ""
done

# Step 3: The staging subdomain might have the flag
curl http://staging.ctf-target.com/flag.txt
```

### Scenario 3: Bug Bounty Methodology

**Objective:** Maximize attack surface discovery for a bug bounty program.

```bash
# Step 1: Passive subdomain discovery
assetfinder --subs-only target.com | tee assetfinder.txt

# Step 2: Check certificate transparency
curl -s "https://crt.sh/?q=%.target.com&output=json" | \
  python3 -c "import sys,json; [print(e['name_value']) for e in json.load(sys.stdin)]" | \
  sort -u > crtsh.txt

# Step 3: Combine results
cat assetfinder.txt crtsh.txt | sort -u > all_subs.txt

# Step 4: Filter for live hosts
httpx -l all_subs.txt -silent -o live.txt

# Step 5: Sort by subdomain depth
awk -F. '{print NF-2, $0}' live.txt | sort -n | awk '{$1=""; print}' | sed 's/^ //'

# Step 6: Report
echo "=========================================="
echo "Bug Bounty Recon Summary for target.com"
echo "=========================================="
echo "Total subdomains found: $(wc -l < all_subs.txt)"
echo "Live hosts: $(wc -l < live.txt)"
```

---

## Chapter 7: Detection and Defense

### How Defenders Detect This Tool

#### Network Indicators
```
# Assetfinder primarily uses passive sources:
# - crt.sh API queries
# - DNS-related API lookups
# - Certificate transparency log queries

# Detection points:
# - Unusual API queries to certificate transparency services
# - DNS-related reconnaissance patterns
# - Rapid subdomain enumeration
```

#### Log Analysis
```bash
# Monitor for certificate transparency queries
grep "crt.sh" /var/log/nginx/access.log

# Check for DNS enumeration patterns
grep "SOA\|NS\|TXT" /var/log/dnsmasq.log 2>/dev/null

# Look for subdomain enumeration
awk '{print $1}' /var/log/nginx/access.log | sort | uniq -c | sort -rn | head -20
```

### Mitigation Strategies

#### DNS Hardening
```bash
# Disable zone transfers
# In BIND named.conf:
options {
    allow-transfer { none; };
};

# Limit DNS information disclosure
# Remove unnecessary TXT/MX/SPF records
```

#### Rate Limiting
```bash
# nginx rate limiting for certificate transparency
limit_req_zone $binary_remote_addr zone=ct:10m rate=5r/s;

# Monitor for enumeration patterns
fail2ban-client set nginx-auth jail.maxretry 5
```

### Blue Team Response Playbook

1. **Monitor** - Watch for certificate transparency log queries
2. **Detect** - Identify unusual subdomain enumeration patterns
3. **Audit** - Check DNS configurations for zone transfer vulnerabilities
4. **Harden** - Implement DNS security best practices
5. **Document** - Record all findings and response actions
6. **Review** - Assess if any discovered subdomains have vulnerabilities

---

## Chapter 8: Troubleshooting

### Common Errors and Solutions

#### Error: "command not found"
**Cause:** Assetfinder is not installed or not in PATH.
```bash
# Install via APT
sudo apt install assetfinder

# Or verify installation
which assetfinder
```

#### Error: "no results found"
**Cause:** The target domain has no discoverable subdomains, or the API is unavailable.
```bash
# Verify with manual crt.sh query
curl -s "https://crt.sh/?q=%.example.com&output=json" | python3 -c "import sys,json; print(len(json.load(sys.stdin)))"

# Try with other tools
subfinder -d example.com -silent
```

#### Error: "connection refused" or timeout
**Cause:** Network connectivity issue or API rate limiting.
```bash
# Check internet connection
curl -I https://crt.sh

# Try with proxy
export HTTP_PROXY=http://127.0.0.1:8080
assetfinder example.com
```

### Performance Optimization

```bash
# Pipe to file for faster processing
assetfinder example.com > results.txt

# Use sort -u to deduplicate in one pass
assetfinder example.com | sort -u

# Limit output to specific patterns
assetfinder example.com | grep '\.api\.' > api_subdomains.txt
```

### FAQ

**Q: Is assetfinder legal to use?**
A: Yes, assetfinder performs passive reconnaissance using publicly available data sources. However, always get authorization before testing discovered assets.

**Q: How do I update assetfinder?**
A: `sudo apt update && sudo apt upgrade assetfinder` or `go install github.com/tomnomnom/assetfinder@latest`

**Q: What's the difference between assetfinder and subfinder?**
A: Assetfinder is simpler and faster for quick subdomain discovery. Subfinder supports more data sources and provides more detailed output. For comprehensive recon, use both.

**Q: Why does assetfinder find fewer subdomains than other tools?**
A: Assetfinder focuses on speed and simplicity. Use it in combination with other tools (subfinder, amass) for comprehensive coverage.

**Q: Can assetfinder discover non-subdomain assets?**
A: By default, assetfinder also finds related domains (not just subdomains). Use `--subs-only` to limit to just subdomains.

---

## Resources

### Official Documentation
- [Assetfinder GitHub](https://github.com/tomnomnom/assetfinder)
- [tomnomnom's GitHub](https://github.com/tomnomnom)

### Tutorials
- [Assetfinder Usage Guide](https://github.com/tomnomnom/assetfinder#readme)
- [Subdomain Enumeration Techniques](https://book.hacktricks.xyz/)
- [Bug Bounty Recon Methodology](https://www.hackerone.com/)

### Related Tools
- **subfinder:** More comprehensive subdomain discovery
- **amass:** In-depth attack surface mapping
- **findomain:** Cross-platform subdomain discovery
- **sublist3r:** Subdomain enumeration tool
- **httpx:** HTTP probing for live hosts
- **dnsx:** DNS toolkit

### Practice Labs
- [HackTheBox](https://www.hackthebox.com/) - CTF platform
- [TryHackMe](https://tryhackme.com/) - Learning platform
- [BugBounty Hunter](https://www.bugbountyhunter.com/) - Bug bounty practice
