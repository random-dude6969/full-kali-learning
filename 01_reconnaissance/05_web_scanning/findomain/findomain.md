---
tool_name: "findomain"
display_name: "Findomain"
mitre_tactic: "reconnaissance"
mitre_tactic_id: "TA0043"
mitre_subcategory: "web_scanning"
install_command: "sudo apt install findomain"
version: "8.2.1"
github_repo: "https://github.com/Findomain/Findomain"
kali_tools_page: "https://www.kali.org/tools/findomain/"
difficulty_level: "beginner"
requires_root: false
gui_available: false
tags: ["subdomain", "reconnaissance", "fast", "multi-source", "dns"]
related_tools: ["subfinder", "amass", "assetfinder"]
---

# Findomain — Fast Cross-Platform Subdomain Enumerator

## Chapter 1: Introduction & Overview

### What is Findomain?
Findomain is a fast, cross-platform subdomain enumeration tool that uses multiple sources including certificate transparency logs, DNS databases, and APIs to discover subdomains. Written in Rust, it is extremely fast and efficient, capable of finding thousands of subdomains in seconds. Findomain is one of the fastest subdomain enumeration tools available and supports both passive and active enumeration techniques.

### History & Background
- **Created:** 2019 by Eduard Tolosa (Findomain project)
- **Language:** Rust
- **License:** GNU General Public License v3.0
- **Current Version:** 8.2.1 (as of 2024)
- **Purpose:** Fast subdomain discovery using multiple sources

Findomain was created to address the need for a fast, reliable subdomain enumeration tool that aggregates results from multiple sources. It quickly gained popularity in the bug bounty and penetration testing communities due to its speed and accuracy. The tool is actively maintained and regularly updated with new data sources.

### When to Use This Tool
- **Subdomain discovery:** Find all subdomains of a target domain
- **Attack surface mapping:** Discover the complete attack surface of a target
- **Bug bounty hunting:** Find hidden subdomains for bug bounty programs
- **Penetration testing:** Phase 1 reconnaissance for web applications
- **Asset discovery:** Inventory all web assets for an organization
- **Certificate transparency monitoring:** Track SSL certificate issuance

### When NOT to Use This Tool
- Without explicit written authorization from the target owner
- Against production systems without prior coordination
- When you need deep vulnerability scanning (use specialized tools instead)
- For denial-of-service testing
- Against systems you do not own or have permission to test

### Legal and Ethical Considerations
- **Always obtain written authorization** before testing any system you do not own
- **Define scope clearly** — know exactly which domains are in scope
- **Document everything** — keep records of authorization, scope, and findings
- **Follow responsible disclosure** — report vulnerabilities through proper channels
- **Understand local laws** — unauthorized computer access is illegal in most jurisdictions

### Key Concepts
- **Subdomain:** A domain that is part of a larger domain (e.g., sub.example.com)
- **Certificate Transparency (CT):** Public logs of SSL/TLS certificate issuance
- **DNS Brute-Forcing:** Guessing subdomain names through DNS queries
- **Passive Enumeration:** Gathering information from publicly available sources without direct interaction
- **Active Enumeration:** Directly querying DNS servers to discover subdomains
- **API Integration:** Using external services to discover subdomains

---

## Chapter 2: Installation & Setup

### System Requirements
- **Operating System:** Linux, macOS, Windows, or any Rust-supported platform
- **RAM:** Minimum 256 MB (512 MB recommended)
- **Disk Space:** 20 MB for installation
- **CPU:** Multi-core recommended for maximum performance
- **Network:** Stable internet connection
- **Rust:** Required for building from source (Rust 1.70+)

### Installation Methods

#### Method 1: APT (Recommended for Kali Linux)
```bash
sudo apt update
sudo apt install findomain
```

#### Method 2: Cargo Install
```bash
cargo install findomain
```

#### Method 3: From Source
```bash
git clone https://github.com/Findomain/Findomain.git
cd Findomain
cargo build --release
sudo cp target/release/findomain /usr/local/bin/
```

#### Method 4: Download Release Binary
```bash
# Visit https://github.com/Findomain/Findomain/releases
# Download the appropriate binary for your platform
chmod +x findomain
sudo mv findomain /usr/local/bin/
```

#### Method 5: Docker
```bash
docker run -it --rm findomain/findomain -t target.example.com
```

### Configuration
Findomain requires API keys for some data sources. Create a configuration file:

```bash
# Create configuration file
cat > ~/.config/findomain/config.toml << 'EOF'
[config]
# Shodan API key (optional)
shodan_api_key = "YOUR_SHODAN_API_KEY"

# Censys API credentials (optional)
censys_api_id = "YOUR_CENSYS_API_ID"
censys_api_secret = "YOUR_CENSYS_API_SECRET"

# VirusTotal API key (optional)
virustotal_api_key = "YOUR_VIRUSTOTAL_API_KEY"

# SecurityTrails API key (optional)
securitytrails_api_key = "YOUR_SECURITYTRAILS_API_KEY"
EOF
```

### Verification
```bash
findomain --version
# Output: findomain 8.2.1
```

### Updating
```bash
# APT
sudo apt update && sudo apt install --only-upgrade findomain

# Cargo
cargo install findomain --force
```

---

## Chapter 3: Basic Usage

### First Run
```bash
findomain -t target.example.com
```
**Output:**
```
Findomain 8.2.1
Target: target.example.com

[*] Searching subdomains in Certificate Transparency logs...
[*] Searching subdomains in DNS databases...
[*] Searching subdomains using brute-force...

[+] Subdomains found: 15

www.target.example.com
api.target.example.com
mail.target.example.com
ftp.target.example.com
dev.target.example.com
staging.target.example.com
test.target.example.com
admin.target.example.com
blog.target.example.com
shop.target.example.com
cdn.target.example.com
ns1.target.example.com
ns2.target.example.com
mx1.target.example.com
mx2.target.example.com
```

### Essential Commands

#### Basic Subdomain Discovery
```bash
findomain -t target.example.com
```
**Explanation:** Discovers subdomains using all available sources.

#### With API Keys
```bash
findomain -t target.example.com \
    --shodan-key YOUR_SHODAN_KEY \
    --censys-ids YOUR_CENSYS_ID:YOUR_CENSYS_SECRET \
    --virustotal-key YOUR_VT_KEY
```
**Explanation:** Uses API keys for additional data sources.

#### DNS Brute-Forcing
```bash
findomain -t target.example.com -b
```
**Explanation:** Enables DNS brute-forcing with default wordlist.

#### Custom Wordlist
```bash
findomain -t target.example.com -b -w /path/to/wordlist.txt
```
**Explanation:** Uses a custom wordlist for DNS brute-forcing.

#### Verbose Output
```bash
findomain -t target.example.com -v
```
**Explanation:** Shows detailed output including source information.

#### Save Output to File
```bash
findomain -t target.example.com -o subdomains.txt
```
**Explanation:** Saves discovered subdomains to a file.

#### Multiple Targets
```bash
findomain -f targets.txt
```
**Explanation:** Processes multiple targets from a file.

#### Filter by Status
```bash
findomain -t target.example.com --subdomains-output live_subdomains.txt --check
```
**Explanation:** Checks which subdomains are live and saves them.

#### Custom DNS Resolver
```bash
findomain -t target.example.com -r 8.8.8.8,8.8.4.4
```
**Explanation:** Uses custom DNS resolvers for queries.

#### JSON Output
```bash
findomain -t target.example.com --json
```
**Explanation:** Outputs results in JSON format.

#### Disable Specific Sources
```bash
findomain -t target.example.com --no-ct --no-dns
```
**Explanation:** Disables Certificate Transparency and DNS database sources.

#### Enable Only Specific Sources
```bash
findomain -t target.example.com --ct-only
```
**Explanation:** Only uses Certificate Transparency logs.

#### Timeout Configuration
```bash
findomain -t target.example.com --timeout 60
```
**Explanation:** Sets 60-second timeout for connections.

#### Thread Control
```bash
findomain -t target.example.com --threads 100
```
**Explanation:** Uses 100 concurrent threads.

#### IP Resolution
```bash
findomain -t target.example.com --resolve
```
**Explanation:** Resolves discovered subdomains to IP addresses.

#### Rate Limiting
```bash
findomain -t target.example.com --rate-limit 10
```
**Explanation:** Limits to 10 requests per second.

### Complete Flags Reference

#### Target Options
| Flag | Description | Example |
|------|-------------|---------|
| `-t` | Target domain | `-t target.example.com` |
| `-f` | Targets file | `-f targets.txt` |
| `--subdomains-only` | Only show subdomains | `--subdomains-only` |
| `--ips-only` | Only show IPs | `--ips-only` |
| `--resolved` | Show resolved IPs | `--resolved` |

#### Source Options
| Flag | Description | Example |
|------|-------------|---------|
| `--ct` | Use Certificate Transparency | `--ct` |
| `--dns` | Use DNS databases | `--dns` |
| `--bruteforce` | Enable DNS brute-forcing | `--bruteforce` or `-b` |
| `--wordlist` | Custom wordlist for brute-forcing | `-w /path/to/wordlist.txt` |
| `--shodan-key` | Shodan API key | `--shodan-key YOUR_KEY` |
| `--censys-ids` | Censys API credentials | `--censys-ids ID:SECRET` |
| `--virustotal-key` | VirusTotal API key | `--virustotal-key YOUR_KEY` |
| `--securitytrails-key` | SecurityTrails API key | `--securitytrails-key YOUR_KEY` |
| `--facebook` | Use Facebook CT | `--facebook` |
| `--hackertarget` | Use HackerTarget | `--hackertarget` |
| `--rapiddns` | Use RapidDNS | `--rapiddns` |
| `--sitedossier` | Use SiteDossier | `--sitedossier` |
| `--no-ct` | Disable CT logs | `--no-ct` |
| `--no-dns` | Disable DNS databases | `--no-dns` |
| `--ct-only` | Only use CT logs | `--ct-only` |
| `--dns-only` | Only use DNS databases | `--dns-only` |

#### DNS Options
| Flag | Description | Example |
|------|-------------|---------|
| `-r` | Custom DNS resolvers | `-r 8.8.8.8,8.8.4.4` |
| `--dns-resolver` | Single DNS resolver | `--dns-resolver 8.8.8.8` |
| `--dns-timeout` | DNS query timeout | `--dns-timeout 10` |
| `--dns-retries` | DNS query retries | `--dns-retries 3` |

#### Output Options
| Flag | Description | Example |
|------|-------------|---------|
| `-o` | Output file | `-o subdomains.txt` |
| `--json` | JSON output | `--json` |
| `--csv` | CSV output | `--csv` |
| `--a` | Show all results | `--a` |
| `--q` | Quiet mode | `--q` |
| `-v` | Verbose output | `-v` |

#### Performance Options
| Flag | Description | Example |
|------|-------------|---------|
| `--threads` | Number of threads | `--threads 100` |
| `--timeout` | Connection timeout | `--timeout 30` |
| `--rate-limit` | Rate limit (req/sec) | `--rate-limit 10` |
| `--connections-limit` | Max connections | `--connections-limit 100` |

#### Check Options
| Flag | Description | Example |
|------|-------------|---------|
| `--check` | Check if subdomains are live | `--check` |
| `--subdomains-output` | Output live subdomains | `--subdomains-output live.txt` |
| `--ips-output` | Output resolved IPs | `--ips-output ips.txt` |

---

## Chapter 4: Intermediate Usage

### Bash Scripting and Automation

#### Automated Subdomain Discovery Script
```bash
#!/bin/bash
# auto_findomain.sh - Automated subdomain discovery

DOMAIN=$1
OUTPUT_DIR="findomain_$(date +%Y%m%d_%H%M%S)"

mkdir -p "$OUTPUT_DIR"

echo "[*] Starting subdomain discovery on $DOMAIN"

# Basic scan
echo "[*] Phase 1: Basic discovery"
findomain -t "$DOMAIN" \
    -o "$OUTPUT_DIR/basic_subs.txt"

# Scan with brute-forcing
echo "[*] Phase 2: DNS brute-forcing"
findomain -t "$DOMAIN" \
    -b \
    -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt \
    -o "$OUTPUT_DIR/bruteforce_subs.txt"

# Check live subdomains
echo "[*] Phase 3: Checking live subdomains"
findomain -t "$DOMAIN" \
    --check \
    --subdomains-output "$OUTPUT_DIR/live_subs.txt"

echo "[*] Results saved to $OUTPUT_DIR"
```

#### Multi-Domain Scanning Script
```bash
#!/bin/bash
# multi_domain_findomain.sh

DOMAINS_FILE=$1
OUTPUT_BASE="findomain_multi"

while IFS= read -r domain; do
    echo "[*] Scanning: $domain"
    OUTPUT_DIR="$OUTPUT_BASE/$(echo $domain | tr '.' '_')"
    mkdir -p "$OUTPUT_DIR"
    
    findomain -t "$domain" \
        -b \
        --check \
        -o "$OUTPUT_DIR/subdomains.txt" \
        --subdomains-output "$OUTPUT_DIR/live_subs.txt"
done < "$DOMAINS_FILE"
```

#### Recon Pipeline with Other Tools
```bash
#!/bin/bash
# recon_pipeline.sh

DOMAIN=$1

echo "[+] Phase 1: Findomain - Subdomain Discovery"
findomain -t "$DOMAIN" -b -o findomain_subs.txt

echo "[+] Phase 2: Cross-reference with other tools"
subfinder -d "$DOMAIN" -o subfinder_subs.txt
amass enum -passive -d "$DOMAIN" -o amass_subs.txt
assetfinder --subs-only "$DOMAIN" > assetfinder_subs.txt

echo "[+] Phase 3: Combine and deduplicate"
cat findomain_subs.txt subfinder_subs.txt amass_subs.txt assetfinder_subs.txt | \
    sort -u > all_subs.txt

echo "[+] Phase 4: Check live hosts"
findomain -f all_subs.txt --check --subdomains-output live_subs.txt

echo "[+] Phase 5: Port scanning live hosts"
nmap -iL live_subs.txt -p 80,443,8080,8443 --open -oN ports.txt

echo "[+] Reconnaissance complete"
```

### Python Scripting

#### Python Wrapper for Findomain
```python
#!/usr/bin/env python3
"""Python wrapper for Findomain subdomain discovery."""

import subprocess
import json
import sys

def run_findomain(domain, options=None):
    """
    Run Findomain scan and return results.
    
    Args:
        domain: Target domain
        options: Dict of additional options
    
    Returns:
        list: Discovered subdomains
    """
    cmd = ['findomain', '-t', domain]
    
    if options:
        if 'bruteforce' in options and options['bruteforce']:
            cmd.append('-b')
        if 'wordlist' in options:
            cmd.extend(['-w', options['wordlist']])
        if 'check' in options and options['check']:
            cmd.append('--check')
        if 'output' in options:
            cmd.extend(['-o', options['output']])
        if 'threads' in options:
            cmd.extend(['--threads', str(options['threads'])])
        if 'json' in options and options['json']:
            cmd.append('--json')
    
    try:
        result = subprocess.run(cmd, capture_output=True, text=True, timeout=300)
        
        if options and 'json' in options and options['json']:
            return json.loads(result.stdout)
        
        return result.stdout.strip().split('\n')
    except subprocess.TimeoutExpired:
        print(f"[-] Scan timed out for {domain}")
        return []
    except json.JSONDecodeError:
        return result.stdout.strip().split('\n')

def main():
    if len(sys.argv) < 2:
        print(f"Usage: {sys.argv[0]} <domain> [options]")
        print(f"Options: bruteforce, check, threads=N")
        sys.exit(1)
    
    domain = sys.argv[1]
    options = {}
    
    for arg in sys.argv[2:]:
        if arg == 'bruteforce':
            options['bruteforce'] = True
        elif arg == 'check':
            options['check'] = True
        elif arg.startswith('threads='):
            options['threads'] = int(arg.split('=')[1])
    
    subdomains = run_findomain(domain, options)
    
    print(f"\n[+] Found {len(subdomains)} subdomains for {domain}:")
    for sub in subdomains:
        if sub.strip():
            print(f"  {sub}")

if __name__ == '__main__':
    main()
```

### Tool Chaining

#### Findomain with Subfinder
```bash
# Findomain for initial discovery
findomain -t target.example.com -b -o findomain_subs.txt

# Subfinder for additional sources
subfinder -d target.example.com -o subfinder_subs.txt

# Combine results
cat findomain_subs.txt subfinder_subs.txt | sort -u > all_subs.txt
```

#### Findomain with Amass
```bash
# Findomain for fast discovery
findomain -t target.example.com -o findomain_subs.txt

# Amass for deep enumeration
amass enum -passive -d target.example.com -o amass_subs.txt

# Combine and deduplicate
cat findomain_subs.txt amass_subs.txt | sort -u > all_subs.txt
```

#### Findomain with Nmap
```bash
# Discover subdomains
findomain -t target.example.com --check --subdomains-output live_subs.txt

# Port scan live hosts
nmap -iL live_subs.txt -p 80,443,8080,8443 --open -oN ports.txt
```

### Output Processing and Filtering

#### Extracting Only Live Subdomains
```bash
findomain -t target.example.com --check --subdomains-output live_subs.txt
wc -l live_subs.txt
```

#### Converting to Other Formats
```bash
# JSON output
findomain -t target.example.com --json > subs.json

# CSV output
findomain -t target.example.com --csv > subs.csv
```

#### Filtering by Status
```bash
# Check which subdomains are alive
findomain -t target.example.com --check --subdomains-output live_subs.txt

# Get only subdomains with specific keywords
grep -E "^(admin|api|dev|staging)" live_subs.txt
```

---

## Chapter 5: Advanced Usage

### Custom Configuration

#### API Key Configuration
```bash
# Set environment variables
export SHODAN_API_KEY="your_key"
export CENSYS_API_ID="your_id"
export CENSYS_API_SECRET="your_secret"
export VIRUSTOTAL_API_KEY="your_key"
export SECURITYTRAILS_API_KEY="your_key"

# Use with Findomain
findomain -t target.example.com
```

#### Custom Wordlists for Brute-Forcing
```bash
# Create custom wordlist
cat > custom_subdomains.txt << 'EOF'
www
api
mail
ftp
dev
staging
test
admin
blog
shop
cdn
ns1
ns2
mx1
mx2
 vpn
webmail
portal
app
mobile
EOF
```

### Evasion and Rate Limiting

#### Slow Scanning
```bash
findomain -t target.example.com \
    --rate-limit 5 \
    --threads 10 \
    --timeout 60
```

#### Using Proxies
```bash
# Findomain doesn't directly support proxies, but you can use proxychains
proxychains findomain -t target.example.com
```

### Integration with Pentest Workflows

#### Comprehensive Subdomain Enumeration
```bash
#!/bin/bash
# comprehensive_subdomain_enum.sh

DOMAIN=$1
OUTPUT_DIR="subenum_$(date +%Y%m%d)"

mkdir -p "$OUTPUT_DIR"

echo "[+] Phase 1: Findomain - Fast Discovery"
findomain -t "$DOMAIN" \
    -b \
    -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-110000.txt \
    -o "$OUTPUT_DIR/findomain_subs.txt"

echo "[+] Phase 2: Subfinder - Multiple Sources"
subfinder -d "$DOMAIN" \
    -o "$OUTPUT_DIR/subfinder_subs.txt" \
    -all

echo "[+] Phase 3: Amass - Deep Enumeration"
amass enum -passive -d "$DOMAIN" \
    -o "$OUTPUT_DIR/amass_subs.txt"

echo "[+] Phase 4: Assetfinder - Additional Sources"
assetfinder --subs-only "$DOMAIN" \
    > "$OUTPUT_DIR/assetfinder_subs.txt"

echo "[+] Phase 5: Combine and Deduplicate"
cat "$OUTPUT_DIR"/*.txt | sort -u > "$OUTPUT_DIR/all_subs.txt"

echo "[+] Phase 6: Check Live Hosts"
findomain -f "$OUTPUT_DIR/all_subs.txt" \
    --check \
    --subdomains-output "$OUTPUT_DIR/live_subs.txt"

echo "[+] Phase 7: DNS Resolution"
cat "$OUTPUT_DIR/live_subs.txt" | while read subdomain; do
    dig +short "$subdomain" | head -1
done > "$OUTPUT_DIR/resolved_ips.txt"

echo "[+] Subdomain enumeration complete. Results in $OUTPUT_DIR"
```

#### Bug Bounty Workflow
```bash
#!/bin/bash
# bug_bounty_subdomain_enum.sh

DOMAIN=$1

echo "[+] Starting bug bounty subdomain enumeration for $DOMAIN"

# Phase 1: Fast discovery
echo "[*] Phase 1: Fast discovery"
findomain -t "$DOMAIN" -o findomain_subs.txt

# Phase 2: Additional sources
echo "[*] Phase 2: Additional sources"
subfinder -d "$DOMAIN" -o subfinder_subs.txt
amass enum -passive -d "$DOMAIN" -o amass_subs.txt

# Phase 3: Combine results
echo "[*] Phase 3: Combining results"
cat findomain_subs.txt subfinder_subs.txt amass_subs.txt | sort -u > all_subs.txt
echo "[*] Total unique subdomains: $(wc -l < all_subs.txt)"

# Phase 4: Check live hosts
echo "[*] Phase 4: Checking live hosts"
findomain -f all_subs.txt --check --subdomains-output live_subs.txt
echo "[*] Live subdomains: $(wc -l < live_subs.txt)"

# Phase 5: Port scanning
echo "[*] Phase 5: Port scanning"
nmap -iL live_subs.txt -p 80,443,8080,8443 --open -oN ports.txt

echo "[+] Subdomain enumeration complete"
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: Web Application Audit

**Objective:** Discover all subdomains for a corporate web application.

**Step 1: Initial Discovery**
```bash
findomain -t corporate.example.com \
    -b \
    -o subdomains.txt
```

**Step 2: Cross-Reference with Other Tools**
```bash
subfinder -d corporate.example.com -o subfinder_subs.txt
amass enum -passive -d corporate.example.com -o amass_subs.txt
cat subdomains.txt subfinder_subs.txt amass_subs.txt | sort -u > all_subdomains.txt
```

**Step 3: Check Live Hosts**
```bash
findomain -f all_subdomains.txt \
    --check \
    --subdomains-output live_subdomains.txt
```

**Step 4: Port Scanning**
```bash
nmap -iL live_subdomains.txt \
    -p 80,443,8080,8443 \
    --open \
    -oN port_scan.txt
```

### Scenario 2: CTF Challenge

**Objective:** Find hidden subdomains in a CTF challenge.

**Step 1: Initial Discovery**
```bash
findomain -t ctf.example.com \
    -o subdomains.txt
```

**Step 2: Deep Brute-Forcing**
```bash
findomain -t ctf.example.com \
    -b \
    -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-110000.txt \
    -o bruteforce_subs.txt
```

**Step 3: Check for Hidden Services**
```bash
findomain -t ctf.example.com \
    --check \
    --subdomains-output live_subs.txt

# Probe live subdomains
httpx -l live_subs.txt -o live_hosts.txt -silent
```

### Scenario 3: Bug Bounty

**Objective:** Comprehensive subdomain discovery for a bug bounty program.

**Step 1: Fast Discovery**
```bash
findomain -t target.com \
    -b \
    -o findomain_subs.txt
```

**Step 2: Additional Sources**
```bash
subfinder -d target.com -o subfinder_subs.txt
amass enum -passive -d target.com -o amass_subs.txt
assetfinder --subs-only target.com > assetfinder_subs.txt
```

**Step 3: Combine and Analyze**
```bash
cat findomain_subs.txt subfinder_subs.txt amass_subs.txt assetfinder_subs.txt | \
    sort -u > all_subs.txt

echo "[*] Total unique subdomains: $(wc -l < all_subs.txt)"
```

**Step 4: Live Host Detection**
```bash
findomain -f all_subs.txt \
    --check \
    --subdomains-output live_subs.txt

httpx -l live_subs.txt -o final_live.txt -silent
echo "[*] Live hosts: $(wc -l < final_live.txt)"
```

---

## Chapter 7: Detection & Defense

### IDS/IPS Detection

**Signature-Based Detection:**
- Findomain generates distinctive User-Agent strings by default
- Rapid DNS queries trigger anomaly detection
- High-volume requests from single IP addresses create detectable patterns
- Specific query patterns are signature-matchable

**Behavioral Detection:**
- Unusual DNS query patterns (sequential subdomain enumeration)
- High request rates from single source
- Queries to non-existent subdomains generating NXDOMAIN responses
- Time-based patterns (consistent intervals between queries)

### Log Indicators

**DNS Server Log Patterns:**
```
# Sequential subdomain queries
query[A] admin.target.example.com from 192.168.1.100
query[A] admin-api.target.example.com from 192.168.1.100
query[A] admin-panel.target.example.com from 192.168.1.100

# High query rate
query[A] api.target.example.com from 192.168.1.100
query[A] api-v1.target.example.com from 192.168.1.100
query[A] api-v2.target.example.com from 192.168.1.100
```

### Mitigation Strategies

**Rate Limiting:**
```bind
# BIND DNS rate limiting
options {
    rate-limit {
        responses-per-second 10;
        window 5;
        slip 2;
    };
};
```

**DNS Firewall:**
```bash
# Using dnsrpz (DNS Response Policy Zones)
# Block suspicious subdomain queries
```

**Monitoring:**
```bash
# Watch for enumeration in real-time
tail -f /var/log/named/queries.log | grep "NXDOMAIN"

# Check for high query rates
grep "query" /var/log/named/queries.log | \
    awk '{print $1}' | sort | uniq -c | sort -rn | head -20
```

### Blue Team Response

**Incident Response Steps:**
1. **Identify:** Analyze DNS logs for enumeration patterns
2. **Contain:** Implement rate limiting or block IPs
3. **Eradicate:** Review exposed subdomains
4. **Recover:** Secure or remove exposed content
5. **Lessons Learned:** Implement preventive controls

**Monitoring Commands:**
```bash
# Watch for enumeration in real-time
tail -f /var/log/named/queries.log | grep -E "(admin|api|dev|staging)"

# Check for high query rates
awk '{print $1}' /var/log/named/queries.log | sort | uniq -c | sort -rn | head -20
```

---

## Chapter 8: Troubleshooting & Resources

### Common Errors

**Error: "Connection timeout"**
```bash
# Solution: Increase timeout
findomain -t target.example.com --timeout 60

# Check network connectivity
ping target.example.com
```

**Error: "DNS resolution failed"**
```bash
# Solution: Use custom DNS resolver
findomain -t target.example.com -r 8.8.8.8,8.8.4.4

# Check DNS configuration
cat /etc/resolv.conf
```

**Error: "API key not found"**
```bash
# Solution: Configure API keys
export SHODAN_API_KEY="your_key"
export CENSYS_API_ID="your_id"
export CENSYS_API_SECRET="your_secret"
```

**Error: "Too many connections"**
```bash
# Solution: Reduce threads
findomain -t target.example.com --threads 10

# Or reduce connections limit
findomain -t target.example.com --connections-limit 50
```

### Performance Optimization

**Maximizing Speed:**
```bash
# Use multiple threads
findomain -t target.example.com --threads 100

# Use fast DNS resolvers
findomain -t target.example.com -r 8.8.8.8,1.1.1.1

# Disable unnecessary sources
findomain -t target.example.com --ct-only
```

**Reducing Memory Usage:**
```bash
# Process domains one at a time
findomain -t domain1.com -o subs1.txt
findomain -t domain2.com -o subs2.txt
```

### FAQ

**Q: How is Findomain different from Subfinder?**
A: Findomain is faster due to its Rust implementation, while Subfinder supports more data sources. Use both for comprehensive coverage.

**Q: Do I need API keys for basic functionality?**
A: No, basic subdomain discovery works without API keys. API keys add additional data sources.

**Q: Can Findomain resolve subdomains to IPs?**
A: Yes, use the `--resolve` flag to resolve discovered subdomains.

**Q: How do I avoid getting blocked?**
A: Use rate limiting (`--rate-limit`), reduce threads, and consider using custom DNS resolvers.

**Q: Can Findomain enumerate subdomains for multiple domains?**
A: Yes, use the `-f` flag with a file containing one domain per line.

---

## Resources

### Official Documentation
- [Findomain GitHub Repository](https://github.com/Findomain/Findomain)
- [Findomain Wiki](https://github.com/Findomain/Findomain/wiki)
- [Findomain Usage Guide](https://github.com/Findomain/Findomain/blob/master/README.md)

### Recommended Wordlists
- [SecLists DNS Subdomains](https://github.com/danielmiessler/SecLists/tree/master/Discovery/DNS)
- [subdomains-top1million-5000.txt](https://github.com/danielmiessler/SecLists/blob/master/Discovery/DNS/subdomains-top1million-5000.txt)
- [subdomains-top1million-20000.txt](https://github.com/danielmiessler/SecLists/blob/master/Discovery/DNS/subdomains-top1million-20000.txt)

### Books
- "Web Application Hacker's Handbook" by Dafydd Stuttard
- "The Hacker Playbook 3" by Peter Kim
- "Penetration Testing" by Georgia Weidman

### Practice Labs
- [HackTheBox](https://www.hackthebox.com/)
- [TryHackMe](https://tryhackme.com/)
- [BugBountyHunter](https://www.bugbountyhunter.com/)

### Related Tools
- [Subfinder](https://github.com/projectdiscovery/subfinder) - Fast passive subdomain enumeration tool
- [Amass](https://github.com/owasp-amass/amass) - In-depth attack surface mapping
- [Assetfinder](https://github.com/tomnomnom/assetfinder) - Find domains and subdomains related to a given domain
