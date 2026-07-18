---
tool_name: "dnsmap"
display_name: "DNSmap"
mitre_tactic: "reconnaissance"
mitre_tactic_id: "TA0043"
mitre_subcategory: "dns_reconnaissance"
install_command: "sudo apt install dnsmap"
version: "0.36"
github_repo: "https://github.com/galkan/dnsmap"
kali_tools_page: "https://www.kali.org/tools/dnsmap/"
difficulty_level: "beginner"
requires_root: false
gui_available: false
tags: ["dns", "brute-force", "subdomain", "reconnaissance", "passive"]
related_tools: ["subfinder", "amass", "dnsenum", "fierce"]
---

# DNSmap

## Chapter 1: Introduction and Overview

### What is DNSmap?

DNSmap is a DNS brute-force reconnaissance tool that discovers subdomains by querying DNS servers with a predefined wordlist. It sends DNS A record queries for potential subdomain names (e.g., mail.target.com, www.target.com, ftp.target.com) and reports which ones resolve to valid IP addresses.

Unlike passive DNS tools that query public databases, DNSmap actively performs DNS lookups, making it a lightweight and effective subdomain enumeration tool for penetration testing.

### History and Background

- **Author:** Sergio Abraham (galkan)
- **Language:** Perl
- **License:** GNU General Public License v3.0
- **Repository:** https://github.com/galkan/dnsmap
- **Kali Package:** Included in Kali Linux repositories

DNSmap was originally written in Perl and has been a staple in Kali Linux for years. It's valued for its simplicity, reliability, and ability to work without API keys.

### When to Use This Tool

- **Subdomain enumeration:** Discovering hidden subdomains not listed in public sources
- **Attack surface mapping:** Identifying additional entry points for a target
- **DNS infrastructure analysis:** Understanding DNS configuration
- **Penetration testing:** Pre-engagement reconnaissance phase
- **Bug bounty:** Expanding the scope of discoverable assets
- **Red team operations:** Mapping target infrastructure

### When NOT to Use This Tool

- Without proper authorization from the target organization
- Against production systems without careful rate limiting
- When passive methods should be used first (DNSmap is active)
- For large-scale scanning without permission (may trigger rate limits)

### Legal and Ethical Considerations

- **Active Tool:** DNSmap performs active DNS queries — unlike passive OSINT tools
- **Rate Limiting:** Respect DNS server rate limits to avoid service disruption
- **Authorization:** Required before scanning third-party domains
- **Data Handling:** Discovered subdomains may contain sensitive information
- **Logging:** DNS queries may be logged by authoritative servers

### Key Concepts

- **DNS Brute-Force:** Systematically guessing subdomain names
- **Wordlist-Based:** Uses predefined lists of common subdomain names
- **A Record Query:** Checks if a subdomain resolves to an IP address
- **Subdomain:** A prefix added to the main domain (e.g., "mail" in mail.example.com)
- **DNS Resolver:** Server that translates domain names to IP addresses
- **NXDOMAIN:** DNS response indicating the queried domain does not exist

---

## Chapter 2: Installation and Setup

### System Requirements

- **Operating System:** Linux, macOS, Windows (with Perl)
- **Perl Version:** 5.10 or higher
- **RAM:** 64 MB minimum
- **Disk Space:** 10 MB
- **Network:** Internet connection for DNS queries
- **Privileges:** Non-root sufficient

### Installation Methods

#### Method 1: APT (Recommended)

```bash
sudo apt update
sudo apt install dnsmap
```

#### Method 2: From Source

```bash
git clone https://github.com/galkan/dnsmap.git
cd dnsmap
chmod +x dnsmap.pl
```

#### Method 3: Manual Perl Module Installation

```bash
# Install required Perl modules
cpan install Net::DNS
cpan install IO::Socket

# Download and run
wget https://raw.githubusercontent.com/galkan/dnsmap/master/dnsmap.pl
chmod +x dnsmap.pl
./dnsmap.pl example.com
```

### Configuration

DNSmap uses built-in wordlists. To use custom wordlists:

```bash
# Use custom wordlist
dnsmap example.com -w custom_wordlist.txt

# Use built-in wordlist (default)
dnsmap example.com
```

### Verification

```bash
dnsmap --version
# Output: dnsmap 0.36

dnsmap --help
# Displays all available options
```

---

## Chapter 3: Basic Usage

### First Scan

```bash
dnsmap example.com
```

**Output:**
```
dnsmap 0.36 - DNS Network Mapper
galkan@nullsecurity.net

[+] searching (sub)domains for example.com using a built-in wordlist
[+] using maximum 30 threads
[+]进度: 30 threads

[+] found: admin.example.com (IP: 192.168.1.10)
[+] found: mail.example.com (IP: 192.168.1.20)
[+] found: www.example.com (IP: 192.168.1.30)
[+] found: ftp.example.com (IP: 192.168.1.40)

[+] 4 subdomains found in 12 seconds
[+] results saved to example.com.csv
```

### Essential Commands

#### Basic Subdomain Scan

```bash
dnsmap example.com
```

**Explanation:** Scans for common subdomains using the built-in wordlist.

#### Use Custom Wordlist

```bash
dnsmap example.com -w /usr/share/wordlists/subdomains.txt
```

**Explanation:** Uses a custom wordlist for more targeted enumeration.

#### Scan with IP Filter

```bash
dnsmap example.com -i 192.168.1.0/24
```

**Explanation:** Only shows results within the specified IP range.

#### Limit Thread Count

```bash
dnsmap example.com -t 10
```

**Explanation:** Uses only 10 concurrent threads (reduces network load).

#### Save Results to File

```bash
dnsmap example.com -r results.txt
```

**Explanation:** Saves detailed results to a text file.

### Complete Flags Reference

#### Target Specification

| Flag | Description | Example |
|------|-------------|---------|
| (positional) | Target domain | `dnsmap example.com` |

#### Wordlist Options

| Flag | Description | Example |
|------|-------------|---------|
| `-w` | Custom wordlist file | `dnsmap example.com -w wordlist.txt` |
| `-l` | Maximum wordlist length | `dnsmap example.com -l 1000` |

#### Threading Options

| Flag | Description | Example |
|------|-------------|---------|
| `-t` | Number of threads | `dnsmap example.com -t 20` |

#### Output Options

| Flag | Description | Example |
|------|-------------|---------|
| `-r` | Results file | `dnsmap example.com -r results.txt` |
| `-c` | CSV output | `dnsmap example.com -c results.csv` |

#### Filtering Options

| Flag | Description | Example |
|------|-------------|---------|
| `-i` | IP range filter | `dnsmap example.com -i 192.168.1.0/24` |

#### Miscellaneous

| Flag | Description | Example |
|------|-------------|---------|
| `-h` | Help | `dnsmap -h` |
| `--help` | Extended help | `dnsmap --help` |

---

## Chapter 4: Intermediate Usage

### Bash Scripting with DNSmap

#### Multi-Domain Subdomain Enumeration

```bash
#!/bin/bash
# multi_domain_dnsmap.sh - Enumerate subdomains for multiple domains

DOMAINS_FILE=$1
OUTPUT_DIR="dnsmap_results_$(date +%Y%m%d)"

mkdir -p "$OUTPUT_DIR"

while IFS= read -r domain; do
    echo "[*] Scanning: $domain"
    dnsmap "$domain" -r "$OUTPUT_DIR/${domain}_results.txt" -c "$OUTPUT_DIR/${domain}.csv" 2>/dev/null
    sleep 5  # Rate limiting
done < "$DOMAINS_FILE"

echo "[+] All results saved to $OUTPUT_DIR/"
```

#### Automated Subdomain Discovery Pipeline

```bash
#!/bin/bash
# subdomain_pipeline.sh - Complete subdomain discovery workflow

TARGET=$1
WORDLIST=${2:-"/usr/share/wordlists/subdomains.txt"}
OUTPUT_DIR="subdomain recon_${TARGET}"

mkdir -p "$OUTPUT_DIR"

echo "[+] Starting subdomain enumeration for: $TARGET"

# Phase 1: DNSmap brute-force
echo "[*] Phase 1: DNSmap brute-force..."
dnsmap "$TARGET" -w "$WORDLIST" -r "$OUTPUT_DIR/dnsmap_results.txt" -c "$OUTPUT_DIR/dnsmap.csv" 2>/dev/null

# Phase 2: Extract subdomains from results
echo "[*] Phase 2: Extracting subdomains..."
grep -oE "[a-zA-Z0-9.-]+\.$TARGET" "$OUTPUT_DIR/dnsmap_results.txt" | \
    sort -u > "$OUTPUT_DIR/discovered_subdomains.txt"

# Phase 3: Resolve to IPs
echo "[*] Phase 3: Resolving subdomains..."
while read subdomain; do
    ip=$(dig +short "$subdomain" | head -1)
    if [ -n "$ip" ]; then
        echo "$subdomain,$ip" >> "$OUTPUT_DIR/resolved.csv"
    fi
done < "$OUTPUT_DIR/discovered_subdomains.txt"

echo "[+] Enumeration complete!"
echo "[+] Subdomains found: $(wc -l < "$OUTPUT_DIR/discovered_subdomains.txt")"
echo "[+] Results: $OUTPUT_DIR/"
```

### Tool Chaining

#### DNSmap + Nmap Pipeline

```bash
# Step 1: Enumerate subdomains
dnsmap example.com -r dnsmap_results.txt 2>/dev/null

# Step 2: Extract and resolve
grep -oE "[a-zA-Z0-9.-]+\.example\.com" dnsmap_results.txt | \
    sort -u | while read sub; do
        dig +short "$sub" | head -1
    done | sort -u > resolved_ips.txt

# Step 3: Port scan
nmap -sV -sC -iL resolved_ips.txt -oN nmap_results.txt
```

#### DNSmap + Subfinder Pipeline

```bash
# Combine multiple tools for comprehensive enumeration
dnsmap example.com -r dnsmap_results.txt 2>/dev/null
subfinder -d example.com -silent > subfinder_results.txt

# Merge and deduplicate
cat dnsmap_results.txt subfinder_results.txt | \
    grep -oE "[a-zA-Z0-9.-]+\.example\.com" | \
    sort -u > all_subdomains.txt
```

### Output Processing

#### CSV Analysis

```bash
# Parse DNSmap CSV output
dnsmap example.com -c results.csv 2>/dev/null

# Analyze results
awk -F',' '{print $2}' results.csv | sort -u > unique_ips.txt
echo "[*] Unique IPs: $(wc -l < unique_ips.txt)"
```

#### JSON Conversion

```bash
# Convert results to JSON
dnsmap example.com -r results.txt 2>/dev/null

# Parse and convert
echo "[" > results.json
grep "found:" results.txt | \
    sed 's/\[+\] found: /{/' | \
    sed 's/ (IP: /","ip":"/' | \
    sed 's/)/"}/' >> results.json
echo "]" >> results.json
```

---

## Chapter 5: Advanced Usage

### Custom Wordlist Creation

#### Creating Targeted Wordlists

```bash
# Create wordlist from company information
cat > company_wordlist.txt << EOF
admin
webmail
mail
email
ftp
sftp
scp
vpn
remote
gateway
proxy
proxy01
proxy02
ns1
ns2
ns3
dns
dns1
dns2
mx
mx1
mx2
smtp
pop
imap
web
www
www2
www3
blog
portal
intranet
extranet
dev
dev01
dev02
development
staging
stage
stage01
test
testing
test01
qa
uat
demo
sandbox
pre
preview
beta
alpha
release
canary
api
api01
api02
rest
graphql
ws
websocket
cdn
static
media
assets
images
img
css
js
app
application
mobile
m
wap
wap01
shop
store
ecommerce
cart
checkout
payment
pay
billing
account
accounts
login
signin
sso
auth
oauth
register
signup
help
support
docs
documentation
wiki
kb
knowledge
faq
forum
community
social
blog
news
press
media
pressroom
events
calendar
jobs
careers
recruit
hr
training
learn
lms
crm
erp
sap
oracle
database
db
db01
db02
mysql
postgres
mssql
oracle
redis
memcache
elastic
search
solr
ldap
ad
dc
domain
kdc
ca
cert
pki
ssl
tls
owa
exchange
owa01
owa02
autodiscover
lync
skype
teams
voip
pbx
phone
tel
fax
print
printer
scan
copier
backup
bak
restore
archive
log
logs
syslog
monitor
nagios
zabbix
grafana
prometheus
splunk
siem
ids
ips
firewall
fw
fw01
fw02
utm
waf
dlp
noc
soc
ops
ops01
admin01
admin02
root
sysadmin
netadmin
security
audit
compliance
EOF
```

#### Industry-Specific Wordlists

```bash
# Financial industry wordlist
cat > finance_wordlist.txt << EOF
online
banking
webbank
ebank
mbank
ibank
atm
card
credit
debit
wire
transfer
swift
ach
sepa
forex
trading
invest
portfolio
wealth
asset
fund
etf
stock
bond
crypto
wallet
exchange
broker
advisor
financial
finance
tax
irs
revenue
accounting
payroll
invoice
billing
pdf
document
secure
login
portal
self-service
myaccount
customer
client
partner
employee
internal
extranet
vpn
remote
gateway
proxy
firewall
dmz
lan
wan
wifi
wireless
iot
scada
industrial
manufacturing
warehouse
logistics
supply
chain
inventory
erp
sap
oracle
database
db
backup
dr
disaster
recovery
business-continuity
EOF
```

### Rate Limiting Evasion

```bash
# Use slower thread count
dnsmap example.com -t 5

# Add delays between queries
dnsmap example.com -t 3 -l 500

# Use VPN rotation
for i in {1..10}; do
    dnsmap example.com -t 5 -r "results_${i}.txt" 2>/dev/null
    # Switch VPN
    killall openvpn
    sleep 5
    openvpn /path/to/config${i}.ovpn &
    sleep 10
done
```

### Integration with Pentest Frameworks

#### Custom DNS Enumeration Module

```python
#!/usr/bin/env python3
# dnsmap_advanced.py - Advanced DNS enumeration wrapper

import subprocess
import json
import csv
from datetime import datetime

def run_dnsmap(domain, wordlist=None, threads=10):
    """Run DNSmap and return structured results."""
    cmd = ["dnsmap", domain, "-t", str(threads), "-c", f"/tmp/{domain}.csv"]
    if wordlist:
        cmd.extend(["-w", wordlist])

    result = subprocess.run(cmd, capture_output=True, text=True)
    return parse_results(domain)

def parse_results(domain):
    """Parse DNSmap CSV output."""
    results = {"domain": domain, "subdomains": [], "timestamp": str(datetime.now())}
    csv_file = f"/tmp/{domain}.csv"

    try:
        with open(csv_file, 'r') as f:
            reader = csv.reader(f)
            for row in reader:
                if len(row) >= 2:
                    results["subdomains"].append({
                        "subdomain": row[0],
                        "ip": row[1] if len(row) > 1 else None
                    })
    except FileNotFoundError:
        pass

    return results

def main():
    domain = "example.com"
    results = run_dnsmap(domain)
    print(json.dumps(results, indent=2))

if __name__ == "__main__":
    main()
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: Corporate Subdomain Discovery

**Objective:** Discover all subdomains of a corporate target during authorized penetration test.

```bash
#!/bin/bash
# corporate_subdomain_enum.sh

TARGET="targetcorp.com"
OUTPUT_DIR="subdomain_enum_$(date +%Y%m%d)"

mkdir -p "$OUTPUT_DIR"

echo "[+] Corporate subdomain enumeration: $TARGET"

# Phase 1: DNSmap with built-in wordlist
echo "[*] Phase 1: Built-in wordlist scan..."
dnsmap "$TARGET" -r "$OUTPUT_DIR/phase1_builtin.txt" -c "$OUTPUT_DIR/phase1.csv" 2>/dev/null

# Phase 2: DNSmap with comprehensive wordlist
echo "[*] Phase 2: Comprehensive wordlist scan..."
dnsmap "$TARGET" -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt \
    -r "$OUTPUT_DIR/phase2_comprehensive.txt" -c "$OUTPUT_DIR/phase2.csv" 2>/dev/null

# Phase 3: DNSmap with company-specific wordlist
echo "[*] Phase 3: Company-specific wordlist..."
cat > "$OUTPUT_DIR/company_wordlist.txt" << EOF
targetcorp
mail
webmail
owa
autodiscover
vpn
remote
gateway
proxy
admin
portal
dev
staging
test
api
cdn
static
media
blog
wiki
docs
help
support
jobs
careers
hr
crm
erp
sap
oracle
db
database
backup
ns1
ns2
ns3
mx
mx1
mx2
smtp
pop
imap
ftp
sftp
EOF

dnsmap "$TARGET" -w "$OUTPUT_DIR/company_wordlist.txt" \
    -r "$OUTPUT_DIR/phase3_custom.txt" -c "$OUTPUT_DIR/phase3.csv" 2>/dev/null

# Phase 4: Compile all results
echo "[*] Phase 4: Compiling results..."
cat "$OUTPUT_DIR"/phase*.txt | \
    grep -oE "[a-zA-Z0-9.-]+\.$TARGET" | \
    sort -u > "$OUTPUT_DIR/all_subdomains.txt"

echo "[+] Enumeration complete!"
echo "[+] Total unique subdomains: $(wc -l < "$OUTPUT_DIR/all_subdomains.txt")"
```

### Scenario 2: CTF Challenge Subdomain Discovery

**Objective:** Find hidden subdomains for a CTF challenge.

```bash
# Quick subdomain scan
dnsmap target.ctf -r ctf_results.txt 2>/dev/null

# Extract subdomains
grep "found:" ctf_results.txt | awk '{print $3}' > discovered.txt

# Check each subdomain
while read sub; do
    echo "[*] Checking: $sub"
    curl -s -o /dev/null -w "%{http_code}" "http://$sub" 2>/dev/null
done < discovered.txt
```

### Scenario 3: Bug Bounty Subdomain Expansion

**Objective:** Expand attack surface for a bug bounty program.

```bash
#!/bin/bash
# bugbounty_subdomains.sh

TARGET=$1

echo "[+] Bug bounty subdomain enumeration: $TARGET"

# DNSmap scan
dnsmap "$TARGET" -r "dnsmap_${TARGET}.txt" -c "dnsmap_${TARGET}.csv" 2>/dev/null

# Extract and verify subdomains
grep -oE "[a-zA-Z0-9.-]+\.$TARGET" "dnsmap_${TARGET}.txt" | \
    sort -u | while read sub; do
        # Quick HTTP check
        status=$(curl -s -o /dev/null -w "%{http_code}" "http://$sub" 2>/dev/null)
        if [ "$status" != "000" ]; then
            echo "$sub (HTTP $status)" >> "live_subdomains_${TARGET}.txt"
        fi
    done

echo "[+] Results: dnsmap_${TARGET}.txt"
echo "[+] Live subdomains: live_subdomains_${TARGET}.txt"
```

---

## Chapter 7: Detection and Defense

### Detection Signatures

#### DNS Query Monitoring

```bash
# Monitor for brute-force patterns
tcpdump -i eth0 port 53 -n | \
    awk '/A?/ {print $NF}' | \
    sort | uniq -c | sort -rn | head -20

# Detect high-frequency DNS queries
dnstop -l 5 eth0
```

#### Log Analysis

```bash
# Analyze DNS logs for brute-force patterns
awk '{print $1}' /var/log/named/queries.log | \
    sort | uniq -c | sort -rn | \
    awk '$1 > 100 {print "Possible brute-force from:", $2}'
```

#### IDS Signatures

```bash
# Suricata rule for DNS brute-force
alert udp $HOME_NET any -> $HOME_NET 53 (
    msg:"OSINT DNS brute-force detected";
    dns.query;
    threshold:type both, track by_src, count 50, seconds 10;
    sid:1000010;
    rev:1;
)
```

### Mitigation Strategies

#### Rate Limiting

```bash
# Bind rate limiting configuration
options {
    rate-limit {
        responses-per-second 10;
        window 5;
        slip 2;
        qps-scale 256;
    };
};
```

#### DNS Response Policy

```bash
# Return NXDOMAIN for unregistered subdomains
zone "example.com" {
    type master;
    file "/etc/bind/example.com.zone";
    allow-query { any; };
    allow-transfer { none; };
};
```

#### Monitoring and Alerting

```bash
# Monitor DNS queries with Prometheus
# Alert on abnormal query rates
- alert: DNSBruteForceDetected
  expr: rate(dns_queries_total[1m]) > 100
  for: 5m
  labels:
    severity: warning
  annotations:
    summary: "Possible DNS brute-force detected"
```

---

## Chapter 8: Troubleshooting

### Common Errors

#### Error: "No such domain"

**Cause:** Target domain doesn't exist or DNS resolution failing.

**Solution:**
```bash
# Test DNS resolution
nslookup example.com
dig example.com

# Check DNS server
cat /etc/resolv.conf

# Try different DNS server
dig @8.8.8.8 example.com
```

#### Error: "Connection timed out"

**Cause:** Network issues or DNS server unreachable.

**Solution:**
```bash
# Test DNS connectivity
dig @8.8.8.8 example.com

# Check network
ping -c 3 8.8.8.8

# Reduce thread count
dnsmap example.com -t 5
```

#### Error: "Too many open files"

**Cause:** System limit on open file descriptors.

**Solution:**
```bash
# Increase file descriptor limit
ulimit -n 65536

# Or reduce thread count
dnsmap example.com -t 10
```

### Performance Optimization

- **Use appropriate thread count:** Start with 10, increase if network allows
- **Filter by IP range:** Use `-i` to focus on relevant IP ranges
- **Use targeted wordlists:** Custom wordlists are faster than comprehensive ones
- **Rate limiting:** Add delays to avoid detection and rate limits

### FAQ

**Q: Is DNSmap passive or active?**
A: DNSmap is active — it performs DNS queries directly to DNS servers. Use passive tools first if stealth is required.

**Q: How does DNSmap differ from Subfinder?**
A: DNSmap uses brute-force with wordlists, while Subfinder queries multiple passive sources. Using both provides more comprehensive results.

**Q: Can I use DNSmap without a wordlist?**
A: No, DNSmap requires a wordlist. The built-in wordlist is used by default if no custom wordlist is specified.

**Q: How do I avoid detection?**
A: Use low thread counts (`-t 5`), add delays, and use VPN rotation. However, detection is likely for active DNS queries.

**Q: What's the best wordlist for DNSmap?**
A: For general use, Seclists' subdomain wordlists are excellent. For targeted engagements, create custom wordlists based on company information.

---

## Resources

### Official Documentation

- **GitHub Repository:** https://github.com/galkan/dnsmap
- **Kali Linux Tools Page:** https://www.kali.org/tools/dnsmap/
- **Man Pages:** `man dnsmap`

### Books

- *Penetration Testing* by Georgia Weidman
- *The Hacker Playbook 3* by Peter Kim
- *Web Application Hacker's Handbook* by Dafydd Stuttard

### Video Tutorials

- **DNSmap Tutorial** by HackerSploit
- **Subdomain Enumeration** by John Hammond
- **OSINT Reconnaissance** by NetworkChuck

### Practice Labs

- **HackTheBox:** Subdomain discovery challenges
- **TryHackMe:** DNS enumeration rooms
- **VulnHub:** Practice targets
- **BugbountyTraining.com:** Bug bounty practice
