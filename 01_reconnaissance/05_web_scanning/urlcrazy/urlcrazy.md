---
tool_name: "urlcrazy"
display_name: "URLCrazy"
mitre_tactic: "reconnaissance"
mitre_tactic_id: "TA0043"
mitre_subcategory: "web_scanning"
install_command: "sudo apt install urlcrazy"
version: "0.7"
github_repo: "https://github.com/urbanadventurer/URLCrazy"
kali_tools_page: "https://www.kali.org/tools/urlcrazy/"
difficulty_level: "beginner"
requires_root: false
gui_available: false
tags: ["domain", "typo", "phishing", "subdomain", "brand-protection", "dns"]
related_tools: ["dnstwist", "dnsrecon", "subfinder", "amass"]
---

# URLCrazy

## Chapter 1: Introduction & Overview

### What is URLCrazy?
URLCrazy is a domain name permutation generator and tester. It generates potential misspellings, typos, and permutations of a domain name using 15 different algorithms (bit-squatting, homoglyphs, hyphenation, insertion, omission, etc.), then resolves DNS to find which permutations are registered and checks their HTTP status. It's used for phishing detection, brand protection, and threat intelligence.

### History & Background
- **Created:** 2015 by Andrew Horton (urbanadventurer)
- **License:** GNU General Public License v3 (GPLv3)
- **Language:** Ruby
- **Purpose:** Domain permutation generation and phishing domain detection
- **Algorithms:** 15 domain mutation techniques

### When to Use This Tool
- **Phishing detection:** Find domains impersonating your brand
- **Typosquatting discovery:** Identify accidental misspellings registered by attackers
- **Brand protection:** Monitor for domain abuse
- **Threat intelligence:** Discover related malicious domains
- **Bug bounty:** Find subdomain takeovers and domain-related vulnerabilities
- **Corporate security:** Audit domain portfolio for gaps

### Legal and Ethical Considerations
- Generates domain permutations (passive technique)
- DNS resolution is legal and non-intrusive
- Use for defensive security and threat intelligence
- Monitor your own brand for abuse
- Do not use generated domains for phishing or social engineering

### Key Concepts
- **Typosquatting:** Registering misspelled versions of popular domains
- **Bit-squatting:** Flipping individual bits in domain name encoding
- **Homoglyphs:** Using visually similar characters (e.g., rn vs m)
- **Domain permutation:** Any variation of a domain name
- **Popularity check:** Alexa/Tranco ranking to find actively used domains

---

## Chapter 2: Installation & Setup

### System Requirements
- Ruby 2.5+
- DNS resolution capability
- Internet access
- No special permissions needed

### APT Installation (Kali Linux)
```bash
sudo apt update
sudo apt install urlcrazy
```

**Sample output:**
```
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following packages were automatically installed and are no longer required:
  libruby3.1 ruby3.1
Use 'sudo apt autoremove' to remove them.
The following NEW packages will be installed:
  urlcrazy
0 upgraded, 1 newly installed, 0 to remove and 45 not upgraded.
Need to get 65.2 kB of archives.
After this operation, 324 kB of additional disk space will be used.
Get:1 http://kali.download/kali kali-rolling/main amd64 urlcrazy all 0.7-0kali1 [65.2 kB]
Fetched 65.2 kB in 1s (65.2 kB/s)
Selecting previously unselected package urlcrazy.
(Reading database ... 312456 files and directories currently installed.)
Preparing to unpack .../urlcrazy_0.7-0kali1_amd64.deb ...
Unpacking urlcrazy (0.7-0kali1) ...
Setting up urlcrazy (0.7-0kali1) ...
```

### From Source
```bash
git clone https://github.com/urbanadventurer/URLCrazy.git
cd URLCrazy
chmod +x urlcrazy
./urlcrazy --version
```

### Docker Installation
```bash
docker run --rm urbanadventurer/urlcrazy target.com
```

### Verification
```bash
urlcrazy --version
```
**Output:**
```
URLCrazy v0.7
```

---

## Chapter 3: Basic Usage

### First Run
```bash
urlcrazy target.com
```
**Sample output:**
```
 URLCrazy - Domain Permutation Generator
 ========================================
 Target: target.com
 Input:  target.com
 Result: 160 permutations

 Use pop [y/N]? 

Domain                  Type               IP              CC   AFD
---------------------------------------------------------------
target.com              Original           93.184.216.34   US   -
tarjet.com              Typosquat          198.51.100.50   US   -
targest.com             Typosquat          -               -    -
tagret.com              Typosquat          198.51.100.51   US   -
targ3t.com              Typosquat          198.51.100.52   US   -
target.com              Homograph          93.184.216.34   US   -
target.com              Homograph          -               -    -
target-com.com          Hyphenation        198.51.100.53   US   -
target.com              Insertion          -               -    -
targeet.com             Omission           198.51.100.54   US   -
tarhget.com             Transposition      -               -    -
target.com              Vowel-Swap         198.51.100.55   US   -
target.com              Bit-Squatting      -               -    -
target.com              Singular           -               -    -
target.com              Plural             -               -    -

Summary:
  Total: 160 permutations
  Registered: 23
  Active (HTTP): 8
  Suspicious: 3
```

### Essential Commands

#### Generate Permutations
```bash
urlcrazy target.com
```
**Explanation:** Generates domain permutations and checks DNS resolution.

#### CSV Output
```bash
urlcrazy -f csv target.com
```
**Output:**
```
domain,type,ip,cc,afd
target.com,Original,93.184.216.34,US,
tarjet.com,Typosquat,198.51.100.50,US,
targest.com,Typosquat,-,-,
tagret.com,Typosquat,198.51.100.51,US,
targ3t.com,Typosquat,198.51.100.52,US,
```

#### Limit Permutations
```bash
urlcrazy -n 50 target.com
```
**Explanation:** Generate only 50 permutations (faster).

#### Popularity Check
```bash
urlcrazy -p target.com
```
**Explanation:** Check which permutation domains are popular (potentially malicious).

### Complete Flags Reference

#### Target Specification
| Flag | Description | Example |
|------|-------------|---------|
| positional | Target domain | `urlcrazy target.com` |

#### Output Options
| Flag | Description | Example |
|------|-------------|---------|
| `-f`, `--format` | Output format | `urlcrazy -f csv target.com` |
| `-o`, `--output` | Output file | `urlcrazy -o results.csv target.com` |
| `-r`, `--result` | Result format | `urlcrazy -r csv target.com` |
| `-n`, `--count` | Number of permutations | `urlcrazy -n 100 target.com` |

#### Scan Options
| Flag | Description | Example |
|------|-------------|---------|
| `-p`, `--pop` | Check popularity (Alexa/Tranco) | `urlcrazy -p target.com` |
| `--no-dns` | Skip DNS resolution | `urlcrazy --no-dns target.com` |
| `--no-http` | Skip HTTP checks | `urlcrazy --no-http target.com` |

#### Network Options
| Flag | Description | Example |
|------|-------------|---------|
| `--nameserver` | Custom DNS server | `urlcrazy --nameserver 8.8.8.8 target.com` |
| `--timeout` | DNS timeout | `urlcrazy --timeout 5 target.com` |

---

## Chapter 4: Intermediate Usage

### Bash Automation
```bash
#!/bin/bash
# Brand monitoring with URLCrazy
BRANDS=("company.com" "brand.org" "product.net")
REPORT_DIR="urlcrazy_$(date +%Y%m%d)"
mkdir -p "$REPORT_DIR"

for brand in "${BRANDS[@]}"; do
    echo "[*] Checking: $brand"
    
    # Generate permutations with popularity check
    urlcrazy -p -f csv -o "${REPORT_DIR}/${brand}.csv" "$brand" 2>/dev/null
    
    # Find registered domains
    REGISTERED=$(grep -v "\-,$" "${REPORT_DIR}/${brand}.csv" | grep -v "Original" | wc -l)
    echo "[+] Found $REGISTERED registered permutations for $brand"
    
    # Find suspicious domains (active HTTP)
    echo "=== Active Domains ===" > "${REPORT_DIR}/${brand}_suspicious.txt"
    awk -F',' '$3 != "-" && $1 != "Original" {print $1, $3, $4}' "${REPORT_DIR}/${brand}.csv" >> "${REPORT_DIR}/${brand}_suspicious.txt"
    
    echo ""
done

echo "[+] Report complete. Results in: $REPORT_DIR"
```

### Tool Chaining
```bash
# URLCrazy → httpx for live domain check
urlcrazy -f csv target.com | awk -F',' '{print $2}' | \
    grep -v "Original" | \
    httpx -silent -status-code | grep "\[200\]" | \
    tee live_phishing.txt

# URLCrazy → Nuclei for vulnerability scanning
urlcrazy target.com | awk -F',' '{print $2}' | \
    grep -v "Original" | \
    httpx -silent | \
    xargs -I{} nuclei -u {} -t takeovers/

# URLCrazy → Masscan for port scanning
urlcrazy target.com | awk -F',' '{print $3}' | \
    grep -v "\-" | sort -u | \
    xargs -I{} masscan {} -p80,443 --rate=100
```

### Python Integration
```python
import subprocess
import csv
import io

def urlcrazy_scan(domain, count=100, popularity=False):
    cmd = ["urlcrazy", "-n", str(count), "-f", "csv", domain]
    if popularity:
        cmd.append("-p")
    
    result = subprocess.run(cmd, capture_output=True, text=True)
    return result.stdout

def parse_results(csv_output):
    reader = csv.DictReader(io.StringIO(csv_output))
    domains = []
    for row in reader:
        domains.append({
            "domain": row.get("domain", ""),
            "type": row.get("type", ""),
            "ip": row.get("ip", "-"),
            "country": row.get("cc", "-"),
            "active": row.get("afd", "-") != "-"
        })
    return domains

# Scan and analyze
output = urlcrazy_scan("target.com", count=200, popularity=True)
domains = parse_results(output)

active = [d for d in domains if d["active"]]
suspicious = [d for d in domains if d["type"] == "Typosquat" and d["active"]]

print(f"[+] Total permutations: {len(domains)}")
print(f"[+] Active domains: {len(active)}")
print(f"[+] Suspicious typosquats: {len(suspicious)}")
for d in suspicious:
    print(f"  - {d['domain']} ({d['ip']})")
```

### CSV Analysis
```bash
# Parse CSV output for analysis
urlcrazy -f csv -o results.csv target.com

# Find all registered domains
awk -F',' '$3 != "-" && NR > 1 {print $1, $2, $3}' results.csv

# Find typosquats specifically
awk -F',' '$2 == "Typosquat" && $3 != "-" {print $1, $3}' results.csv

# Find domains in specific countries
awk -F',' '$4 == "CN" || $4 == "RU" {print $1, $3, $4}' results.csv

# Sort by IP to find clustering
awk -F',' '$3 != "-" && NR > 1 {print $3}' results.csv | sort | uniq -c | sort -rn
```

---

## Chapter 5: Advanced Usage

### Custom Permutation Patterns
```bash
# Generate permutations without DNS check (faster)
urlcrazy --no-dns target.com | head -30

# Generate specific count
urlcrazy -n 500 --no-dns target.com
```

### Batch Domain Monitoring
```bash
#!/bin/bash
# Daily brand monitoring script
MONITOR_FILE="monitored_domains.txt"
OUTPUT_DIR="urlcrazy_monitor"
DATE=$(date +%Y%m%d)
mkdir -p "$OUTPUT_DIR"

while IFS= read -r domain; do
    echo "[*] Monitoring: $domain"
    
    # Run URLCrazy
    urlcrazy -p -f csv \
        -o "${OUTPUT_DIR}/${domain}_${DATE}.csv" \
        "$domain" 2>/dev/null
    
    # Compare with previous scan
    PREV=$(ls "${OUTPUT_DIR}/${domain}"_*.csv 2>/dev/null | sort -r | head -2 | tail -1)
    if [[ -n "$PREV" ]]; then
        NEW=$(awk -F',' 'NR>1 && $3 != "-" {print $1}' "${OUTPUT_DIR}/${domain}_${DATE}.csv" | sort)
        OLD=$(awk -F',' 'NR>1 && $3 != "-" {print $1}' "$PREV" | sort)
        DIFF=$(comm -13 <(echo "$OLD") <(echo "$NEW"))
        
        if [[ -n "$DIFF" ]]; then
            echo "[!] NEW registered domains:"
            echo "$DIFF"
            echo "$DIFF" >> "${OUTPUT_DIR}/alerts_${DATE}.txt"
        fi
    fi
    
done < "$MONITOR_FILE"
```

### Evasion Techniques
```bash
# Use custom DNS server
urlcrazy --nameserver 8.8.8.8 target.com

# Skip HTTP checks for faster scanning
urlcrazy --no-http -n 100 target.com

# Generate without DNS (pure permutation generation)
urlcrazy --no-dns --no-http target.com
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: Brand Protection — Phishing Domain Detection
**Objective:** Find phishing domains impersonating a company brand.

```bash
# Step 1: Generate permutations for the company domain
urlcrazy -p -f csv -o brand_check.csv mycompany.com

# Step 2: Find active (registered) domains
echo "[+] Active phishing candidates:"
awk -F',' 'NR>1 && $3 != "-" && $1 != "mycompany.com" {print $1, $3, $4}' brand_check.csv

# Step 3: Check HTTP status of active domains
ACTIVE=$(awk -F',' 'NR>1 && $3 != "-" && $1 != "mycompany.com" {print $2}' brand_check.csv)
for domain in $ACTIVE; do
    HTTP_CODE=$(curl -s -o /dev/null -w "%{http_code}" -m 5 "http://${domain}" 2>/dev/null)
    echo "${domain}: HTTP ${HTTP_CODE}"
done

# Step 4: Screenshot suspicious domains
for domain in $ACTIVE; do
    HTTP_CODE=$(curl -s -o /dev/null -w "%{http_code}" -m 5 "http://${domain}" 2>/dev/null)
    if [[ "$HTTP_CODE" == "200" ]]; then
        echo "[!] ACTIVE: ${domain} — taking screenshot"
        # Use cutycapt or similar
        cutycapt --url="http://${domain}" --out="screenshots/${domain}.png"
    fi
done
```

### Scenario 2: CTF Challenge — Find the Phishing Domain
**Objective:** Identify which domain permutation is being used for a phishing attack.

```bash
# Step 1: The target is company.com
urlcrazy company.com

# Step 2: Look for active domains in the output
# Output shows:
# company.com     Original     93.184.216.34     US
# c0mpany.com     Typosquat    198.51.100.50     CN  ← Suspicious!
# comparny.com    Typosquat    -                  -
# company.com     Homograph    93.184.216.34     US

# Step 3: Investigate the suspicious domain
curl -s http://c0mpany.com | head -20
# Reveals: Login page mimicking company.com

# Step 4: Find the flag
curl -s http://c0mpany.com/flag.php
# FLAG{typosquatting_phishing_detected}
```

### Scenario 3: Corporate Domain Portfolio Audit
**Objective:** Audit all brand-related domains for gaps and abuse.

```bash
#!/bin/bash
# Corporate domain audit
BRAND="bigcorp"
TLDs=("com" "net" "org" "io" "co" "info" "biz")
AUDIT_DIR="domain_audit_$(date +%Y%m%d)"
mkdir -p "$AUDIT_DIR"

for tld in "${TLDs[@]}"; do
    DOMAIN="${BRAND}.${tld}"
    echo "[*] Checking: $DOMAIN"
    
    # Check if main domain exists
    HTTP_CODE=$(curl -s -o /dev/null -w "%{http_code}" -m 5 "http://${DOMAIN}" 2>/dev/null)
    
    if [[ "$HTTP_CODE" != "000" ]]; then
        echo "[+] ${DOMAIN} exists (HTTP ${HTTP_CODE})"
        
        # Run URLCrazy for permutations
        urlcrazy -p -f csv -o "${AUDIT_DIR}/${DOMAIN}.csv" "$DOMAIN" 2>/dev/null
        
        # Find registered variations
        REGISTERED=$(awk -F',' 'NR>1 && $3 != "-" {print $1, $3, $4}' "${AUDIT_DIR}/${DOMAIN}.csv")
        if [[ -n "$REGISTERED" ]]; then
            echo "[!] Registered variations:"
            echo "$REGISTERED"
            echo "$REGISTERED" >> "${AUDIT_DIR}/registered_variations.txt"
        fi
    else
        echo "[!] ${DOMAIN} not found — potential gap"
        echo "${DOMAIN}" >> "${AUDIT_DIR}/unregistered.txt"
    fi
    echo ""
done

echo "[+] Audit complete. Findings:"
echo "=== Registered Variations ==="
cat "${AUDIT_DIR}/registered_variations.txt" 2>/dev/null
echo ""
echo "=== Unregistered Domains ==="
cat "${AUDIT_DIR}/unregistered.txt" 2>/dev/null
```

---

## Chapter 7: Detection & Defense

### How Defenders Detect URLCrazy

#### DNS Query Pattern Detection
URLCrazy generates many DNS queries for domain permutations:
```
A tarjet.com → 198.51.100.50
A targest.com → NXDOMAIN
A tagret.com → 198.51.100.51
A targ3t.com → 198.51.100.52
... (100+ queries)
```

#### Log Analysis
```bash
# Find DNS query bursts for similar domains
# On DNS server:
grep "query:" /var/log/named/queries.log | \
    awk '{print $NF}' | sort | uniq -c | sort -rn | head -20

# On web server - find access from similar domains
awk '{print $1}' access.log | sort | uniq -c | sort -rn | head -10
```

### Mitigation Strategies

#### 1. Register Common Misspellings
```bash
# Generate and register common typosquats
urlcrazy --no-dns -n 50 mycompany.com | \
    awk 'NR>2 {print $1}' | \
    while read domain; do
        echo "Register: ${domain}"
    done
```

#### 2. Domain Monitoring Services
- **DomainTools:** Commercial brand protection
- **MarkMonitor:** Enterprise domain management
- **DNSTwist:** Open-source domain monitoring
- **URLCrazy:** Open-source permutation generation

#### 3. Implement DMARC/DKIM/SPF
```dns
; DNS TXT records for email authentication
_dmarc.mycompany.com. IN TXT "v=DMARC1; p=reject; rua=mailto:dmarc@mycompany.com"
mycompany.com. IN TXT "v=spf1 mx a -all"
```

#### 4. Certificate Transparency Monitoring
```bash
# Monitor CT logs for your domains
curl -s "https://crt.sh/?q=%.mycompany.com&output=json" | \
    jq -r '.[].name_value' | sort -u
```

#### 5. Browser Protection
- Enable Safe Browsing (Google, Microsoft, Apple)
- Use DNS-over-HTTPS with malicious domain blocking
- Implement HSTS preloading

### Blue Team Response Playbook
1. **Alert triggered:** Phishing domain detected via monitoring
2. **Verify:** Confirm domain is impersonating your brand
3. **Document:** Screenshot the phishing site, capture DNS records
4. **Takedown:** Report to registrar, hosting provider, and abuse contacts
5. **Block:** Add domain to internal blocklists and email filters
6. **Notify:** Alert affected users and security team
7. **Report:** File abuse reports with relevant authorities
8. **Monitor:** Continue monitoring for new registrations

---

## Chapter 8: Troubleshooting

### Common Errors

#### "DNS resolution failed"
**Cause:** DNS server unreachable or domain doesn't exist.
**Solution:**
```bash
# Test DNS connectivity
nslookup target.com
# Use custom DNS server
urlcrazy --nameserver 8.8.8.8 target.com
```

#### "Too many results"
**Cause:** Generated too many permutations.
**Solution:**
```bash
# Limit permutations
urlcrazy -n 50 target.com
# Skip DNS for faster generation
urlcrazy --no-dns target.com
```

#### "Slow scanning"
**Cause:** DNS resolution is slow for many permutations.
**Solution:**
```bash
# Skip HTTP checks
urlcrazy --no-http target.com
# Reduce permutation count
urlcrazy -n 100 target.com
```

#### "Connection timeout"
**Cause:** DNS server or HTTP targets unreachable.
**Solution:**
```bash
urlcrazy --timeout 10 --no-http target.com
```

### Performance Optimization
```bash
# Fast permutation generation (no DNS/HTTP)
urlcrazy --no-dns --no-http target.com

# Moderate scan (DNS only, no HTTP)
urlcrazy --no-http target.com

# Full scan (DNS + HTTP + popularity)
urlcrazy -p target.com
```

### FAQ

**Q: How many permutations does URLCrazy generate?**
A: Typically 100-200 per domain, depending on domain length and TLD.

**Q: What are the 15 mutation algorithms?**
A: Bit-squatting, homoglyphs, homograph, hyphenation, insertion, omission, transposition, vowel-swap, singular/plural, repetition, replacement, subdomain, and more.

**Q: Does URLCrazy work with any TLD?**
A: Yes, it generates permutations for any TLD.

**Q: Can I use URLCrazy for domain investing?**
A: Yes, it can help find available domain variations.

---

## Resources

- [URLCrazy GitHub](https://github.com/urbanadventurer/URLCrazy)
- [MITRE ATT&CK TA0043](https://attack.mitre.org/tactics/TA0043/)
- [OWASP Phishing](https://owasp.org/www-community/attacks/Phishing)
- [ICANN Domain Abuse](https://www.icann.org/resources/pages/abuse-2012-02-01-en)
- [DNSTwist](https://github.com/elceef/dnstwist)
