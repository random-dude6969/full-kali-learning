---
tool_name: "dnsrecon"
display_name: "DNSRecon"
mitre_tactic: "reconnaissance"
mitre_tactic_id: "TA0043"
mitre_subcategory: "dns_reconnaissance"
install_command: "sudo apt install dnsrecon"
version: "1.2.0"
github_repo: "https://github.com/darkoperator/dnsrecon"
kali_tools_page: "https://www.kali.org/tools/dnsrecon/"
difficulty_level: "beginner-to-intermediate"
requires_root: false
gui_available: false
tags: ["dns", "enumeration", "zone-transfer", "brute-force", "reverse-lookup", "reconnaissance"]
related_tools: ["dnsenum", "dnsmap", "massdns", "dnstracer", "dnswalk"]
---

# DNSRecon - DNS Reconnaissance Tool

## Chapter 1: Introduction & Overview

### What is DNSRecon?

DNSRecon is a comprehensive DNS reconnaissance tool written in Python that performs multiple types of DNS enumeration in a single tool. It supports zone transfers, subdomain brute-forcing, reverse DNS lookups, Google/Bing enumeration, cache snooping, and passive reconnaissance from multiple data sources. DNSRecon produces output in multiple formats including JSON, XML, and CSV, making it highly scriptable for automated workflows.

Unlike dnsenum (Perl) or dnsmap (C), DNSRecon is written in Python, making it easier to modify and extend. It combines the features of several DNS tools into one cohesive package.

### History & Background

- **Created:** Developed by Carlos Perez (darkoperator)
- **Language:** Python 3 (with dnspython library)
- **License:** BSD License
- **Maintained:** Active development on GitHub
- **Repository:** https://github.com/darkoperator/dnsrecon

DNSRecon was designed to be a one-stop tool for DNS reconnaissance during penetration tests, replacing the need to run multiple separate tools.

### When to Use DNSRecon

- **Penetration testing** - Comprehensive DNS reconnaissance
- **Security audits** - Test DNS configuration and security
- **Bug bounty programs** - Discover attack surface
- **CTF competitions** - Solve DNS challenges
- **Red team operations** - Gather infrastructure intelligence
- **Incident response** - Investigate DNS-related incidents

### When NOT to Use This Tool

- Without explicit written authorization
- Against critical infrastructure without coordination
- For unauthorized reconnaissance
- When only simple DNS lookup is needed (use dig/nslookup)

### Legal and Ethical Considerations

- **Written authorization required** before scanning any domain
- **DNS queries are logged** - your source IP will be visible
- **Zone transfer attempts trigger alerts** on monitored DNS servers
- **Brute-forcing generates high query volume** that may be detected
- **Keep records** of all authorized testing activities
- **Follow responsible disclosure** for discovered vulnerabilities

### Key Concepts

- **Zone Transfer (AXFR):** DNS replication mechanism that, if misconfigured, allows full zone data download
- **Brute-force Enumeration:** Systematic subdomain discovery using wordlists
- **Reverse DNS (PTR):** Resolving IP addresses back to hostnames
- **Google/Bing Enumeration:** Searching engines for indexed subdomains
- **Cache Snooping:** Querying DNS cache to discover visited domains
- **Passive Reconnaissance:** Gathering data from public sources without direct queries
- **Wildcards:** DNS configurations where non-existent subdomains resolve to specific IPs

---

## Chapter 2: Installation & Setup

### System Requirements

- **Minimum RAM:** 256 MB (512 MB recommended)
- **Disk Space:** 50 MB
- **Python Version:** 3.6+
- **Dependencies:** dnspython, netaddr, beautifulsoup4, requests
- **Privileges:** Not required for basic operations

### Installation Methods

#### Method 1: APT (Recommended)

```bash
sudo apt update
sudo apt install dnsrecon
```

**Output:**
```
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following NEW packages will be installed:
  dnsrecon
0 upgraded, 1 newly installed, 0 to remove and 142 not upgraded.
Get:1 http://kali.download/kali kali-rolling/main amd64 dnsrecon all 1.2.0-1kali1 [45.8 kB]
Fetched 45.8 kB in 1s (62.1 kB/s)
Selecting previously unselected package dnsrecon.
Setting up dnsrecon (1.2.0-1kali1) ...
```

#### Method 2: From Source

```bash
git clone https://github.com/darkoperator/dnsrecon.git
cd dnsrecon

# Install Python dependencies
pip3 install -r requirements.txt

# Install globally
sudo python3 setup.py install
```

#### Method 3: pip Installation

```bash
pip3 install dnsrecon
```

#### Method 4: Docker

```bash
docker run -it --rm dnsrecon
```

### Configuration

DNSRecon uses a default configuration but supports customization:

```bash
# Default wordlist location
/usr/share/wordlists/dnsrecon.txt

# Create custom config
cat > ~/.dnsrecon/config.json << 'EOF'
{
  "default_threads": 10,
  "timeout": 3,
  "wordlist": "/usr/share/wordlists/subdomains-top1mil-5000.txt"
}
EOF
```

### Verification

```bash
dnsrecon -h
# Shows full help with all available options

dnsrecon --version
# Output: dnsrecon 1.2.0
```

---

## Chapter 3: Basic Usage

### First Run

```bash
dnsrecon -d example.com
```

**Output:**
```
[*] Performing General Enumeration of Domain:example.com
[*] DNS Servers for example.com:
    ns1.example.com [8.8.8.8]
    ns2.example.com [8.8.4.4]
[*] Host Address records for example.com:
    example.com. 3600 IN A 93.184.216.34
    example.com. 3600 IN AAAA 2606:2800:220:1:248:1893:25c8:1946
[*] Nameservers for example.com:
    ns1.example.com. 3600 IN A 8.8.8.8
    ns2.example.com. 3600 IN A 8.8.4.4
[*] Mail Servers for example.com:
    mail.example.com. 3600 IN A 93.184.216.36
[*] SOA Record for example.com:
    example.com. 3600 IN SOA ns1.example.com. admin.example.com. 2024010101 3600 900 604800 86400
```

### Essential Commands

#### Basic DNS Enumeration
```bash
dnsrecon -d target.com
```

#### Zone Transfer Test
```bash
dnsrecon -d target.com -t zt
```

**Output:**
```
[*] Checking for Zone Transfer for example.com using ns1.example.com
[-] Zone Transfer Failed!
    Reason: REFUSED
[*] Checking for Zone Transfer for example.com using ns2.example.com
[-] Zone Transfer Failed!
    Reason: REFUSED
```

#### Subdomain Brute-force
```bash
dnsrecon -d target.com -t brt -D /usr/share/wordlists/subdomains-top1mil-5000.txt
```

**Output:**
```
[*] Performing brute force enumeration for target.com
[*] Using wordlist: /usr/share/wordlists/subdomains-top1mil-5000.txt
[*] Brute forcing subdomains for target.com
[*] www.target.com -> 93.184.216.34
[*] mail.target.com -> 93.184.216.36
[*] ftp.target.com -> 93.184.216.38
[*] api.target.com -> 93.184.216.39
[*] dev.target.com -> 93.184.216.40
[*] Found 5 subdomains
```

#### Reverse DNS Lookup
```bash
dnsrecon -d target.com -t rvl -r 192.168.1.0/24
```

#### Google Enumeration
```bash
dnsrecon -d target.com -t goo
```

#### Standard Enumeration (All Methods)
```bash
dnsrecon -d target.com -t std
```

#### JSON Output
```bash
dnsrecon -d target.com -j output.json
```

#### XML Output
```bash
dnsrecon -d target.com --xml output.xml
```

### Complete Flags Reference

#### Target Specification
| Flag | Description | Example |
|------|-------------|---------|
| `-d` | Target domain | `dnsrecon -d example.com` |
| `-r` | IP range for reverse lookup | `dnsrecon -d example.com -r 192.168.1.0/24` |

#### Enumeration Type
| Flag | Description | Example |
|------|-------------|---------|
| `-t` | Type of enumeration | `dnsrecon -t brt` |
| `std` | Standard enumeration | `dnsrecon -t std -d example.com` |
| `brt` | Brute-force enumeration | `dnsrecon -t brt -d example.com -D wordlist.txt` |
| `rvl` | Reverse lookup | `dnsrecon -t rvl -r 192.168.1.0/24` |
| `zt` | Zone transfer | `dnsrecon -t zt -d example.com` |
| `goo` | Google enumeration | `dnsrecon -t goo -d example.com` |
| `bing` | Bing enumeration | `dnsrecon -t bing -d example.com` |
| `crt` | Certificate transparency | `dnsrecon -t crt -d example.com` |
| `snoop` | Cache snooping | `dnsrecon -t snoop -n nameserver.txt -d example.com` |
| `tld` | TLD expansion | `dnsrecon -t tld -d example.com` |

#### Brute-force Flags
| Flag | Description | Example |
|------|-------------|---------|
| `-D` | Wordlist file | `dnsrecon -D wordlist.txt` |
| `-Z` | Request TLD expansion | `dnsrecon -t brt -Z` |

#### Output Flags
| Flag | Description | Example |
|------|-------------|---------|
| `-j` | JSON output file | `dnsrecon -j output.json` |
| `--xml` | XML output file | `dnsrecon --xml output.xml` |
| `-c` | CSV output file | `dnsrecon -c output.csv` |
| `-S` | Save to file (grepable) | `dnsrecon -S output.txt` |

#### Performance Flags
| Flag | Description | Example |
|------|-------------|---------|
| `--threads` | Number of threads | `dnsrecon --threads 20` |
| `--timeout` | Query timeout | `dnsrecon --timeout 10` |
| `--lifetime` | Overall lifetime | `dnsrecon --lifetime 60` |

#### DNS Server Flags
| Flag | Description | Example |
|------|-------------|---------|
| `--dns-servers` | Specific DNS servers | `dnsrecon --dns-servers 8.8.8.8` |
| `--tcp` | Use TCP for queries | `dnsrecon --tcp` |

#### Miscellaneous Flags
| Flag | Description | Example |
|------|-------------|---------|
| `-a` | Enable A record brute-force | `dnsrecon -a` |
| `-s` | Search for SPF records | `dnsrecon -s` |
| `-k` | Perform cache snooping | `dnsrecon -k` |
| `-p` | Perform reverse lookup on ranges | `dnsrecon -p` |
| `-x` | Perform XML RPC brute-force | `dnsrecon -x` |
| `-u` | Check for zone updates | `dnsrecon -u` |
| `-f` | Filter results by wildcard IP | `dnsrecon -f 93.184.216.34` |
| `-b` | Bypass wildcard filtering | `dnsrecon -b` |

---

## Chapter 4: Intermediate Usage

### Scripting & Automation

#### Bash: Batch Domain Enumeration

```bash
#!/bin/bash
# dnsrecon_batch.sh - Enumerate multiple domains

DOMAINS_FILE="domains.txt"
OUTPUT_DIR="./dnsrecon_results"
WORDLIST="/usr/share/wordlists/subdomains-top1mil-5000.txt"
THREADS=10

mkdir -p "$OUTPUT_DIR"

while IFS= read -r domain; do
    echo "[*] Enumerating: $domain"
    
    # Standard enumeration
    dnsrecon -d "$domain" -t std \
        --threads "$THREADS" \
        --xml "$OUTPUT_DIR/${domain}_std.xml" \
        -j "$OUTPUT_DIR/${domain}_std.json" \
        | tee "$OUTPUT_DIR/${domain}_std.txt"
    
    # Brute-force enumeration
    dnsrecon -d "$domain" -t brt -D "$WORDLIST" \
        --threads "$THREADS" \
        --xml "$OUTPUT_DIR/${domain}_brt.xml" \
        -j "$OUTPUT_DIR/${domain}_brt.json" \
        | tee "$OUTPUT_DIR/${domain}_brt.txt"
    
    echo "[+] Completed: $domain"
    echo "---"
done < "$DOMAINS_FILE"

echo "[*] All domains enumerated. Results in: $OUTPUT_DIR"
```

#### Bash: Zone Transfer Scanner

```bash
#!/bin/bash
# zone_transfer_scan.sh - Test zone transfers across multiple domains

DOMAINS_FILE="targets.txt"
LOG_FILE="zone_transfer_results.log"
NAMESERVERS="8.8.8.8 1.1.1.1 208.67.222.222"

echo "Zone Transfer Scan - $(date)" > "$LOG_FILE"
echo "========================================" >> "$LOG_FILE"

while IFS= read -r domain; do
    echo "[*] Testing zone transfer: $domain"
    
    for ns in $NAMESERVERS; do
        RESULT=$(dnsrecon -d "$domain" -t zt --dns-servers "$ns" 2>&1)
        
        if echo "$RESULT" | grep -q "Zone Transfer was successful"; then
            echo "[!] VULNERABLE: $domain via $ns" | tee -a "$LOG_FILE"
            echo "$RESULT" >> "$LOG_FILE"
        fi
    done
    echo "---" >> "$LOG_FILE"
done < "$DOMAINS_FILE"

echo "[+] Check complete. Results: $LOG_FILE"
```

#### Bash: Comprehensive DNS Audit

```bash
#!/bin/bash
# dnsrecon_audit.sh - Full DNS audit with multiple methods

TARGET="$1"
OUTPUT_DIR="./dnsrecon_audit_${TARGET}"

if [ -z "$TARGET" ]; then
    echo "Usage: $0 <domain>"
    exit 1
fi

mkdir -p "$OUTPUT_DIR"

echo "[*] Starting comprehensive DNS audit for: $TARGET"

# 1. Standard enumeration
echo "[1/6] Standard enumeration..."
dnsrecon -d "$TARGET" -t std \
    --xml "$OUTPUT_DIR/01_standard.xml" \
    -j "$OUTPUT_DIR/01_standard.json" \
    > "$OUTPUT_DIR/01_standard.txt" 2>&1

# 2. Zone transfer test
echo "[2/6] Zone transfer test..."
dnsrecon -d "$TARGET" -t zt \
    --xml "$OUTPUT_DIR/02_zonetransfer.xml" \
    -j "$OUTPUT_DIR/02_zonetransfer.json" \
    > "$OUTPUT_DIR/02_zonetransfer.txt" 2>&1

# 3. Subdomain brute-force
echo "[3/6] Subdomain brute-force..."
dnsrecon -d "$TARGET" -t brt \
    -D /usr/share/wordlists/subdomains-top1mil-5000.txt \
    --threads 15 \
    --xml "$OUTPUT_DIR/03_bruteforce.xml" \
    -j "$OUTPUT_DIR/03_bruteforce.json" \
    > "$OUTPUT_DIR/03_bruteforce.txt" 2>&1

# 4. Google enumeration
echo "[4/6] Google enumeration..."
dnsrecon -d "$TARGET" -t goo \
    --xml "$OUTPUT_DIR/04_google.xml" \
    -j "$OUTPUT_DIR/04_google.json" \
    > "$OUTPUT_DIR/04_google.txt" 2>&1

# 5. Certificate transparency
echo "[5/6] Certificate transparency..."
dnsrecon -d "$TARGET" -t crt \
    --xml "$OUTPUT_DIR/05_crt.xml" \
    -j "$OUTPUT_DIR/05_crt.json" \
    > "$OUTPUT_DIR/05_crt.txt" 2>&1

# 6. Reverse DNS (if IP range known)
echo "[6/6] Reverse DNS lookup..."
# Extract IPs from previous results
grep -oE '([0-9]{1,3}\.){3}[0-9]{1,3}' "$OUTPUT_DIR"/01_standard.txt | \
    sort -u > "$OUTPUT_DIR/ips.txt"

if [ -s "$OUTPUT_DIR/ips.txt" ]; then
    CIDR=$(ipcalc -n $(head -1 "$OUTPUT_DIR/ips.txt")/24 2>/dev/null | grep Network | awk '{print $2}')
    if [ -n "$CIDR" ]; then
        dnsrecon -d "$TARGET" -t rvl -r "$CIDR" \
            --xml "$OUTPUT_DIR/06_reverse.xml" \
            -j "$OUTPUT_DIR/06_reverse.json" \
            > "$OUTPUT_DIR/06_reverse.txt" 2>&1
    fi
fi

echo "[+] Audit complete. Results in: $OUTPUT_DIR"

# Generate summary
echo "=== DNS Audit Summary ===" > "$OUTPUT_DIR/summary.txt"
echo "Target: $TARGET" >> "$OUTPUT_DIR/summary.txt"
echo "Date: $(date)" >> "$OUTPUT_DIR/summary.txt"
echo "" >> "$OUTPUT_DIR/summary.txt"

echo "--- Zone Transfer Results ---" >> "$OUTPUT_DIR/summary.txt"
if grep -q "Zone Transfer was successful" "$OUTPUT_DIR"/02_zonetransfer.txt 2>/dev/null; then
    echo "VULNERABLE: Zone transfer succeeded" >> "$OUTPUT_DIR/summary.txt"
else
    echo "SECURE: Zone transfer blocked" >> "$OUTPUT_DIR/summary.txt"
fi

echo "" >> "$OUTPUT_DIR/summary.txt"
echo "--- Subdomains Found ---" >> "$OUTPUT_DIR/summary.txt"
grep -E '^\[\*\].*->' "$OUTPUT_DIR"/03_bruteforce.txt 2>/dev/null | wc -l >> "$OUTPUT_DIR/summary.txt"
```

### Python Automation

#### Python: Structured DNS Enumeration

```python
#!/usr/bin/env python3
"""
dnsrecon_wrapper.py - Python wrapper for dnsrecon with structured output
"""

import subprocess
import json
import re
import sys
from datetime import datetime
from typing import Dict, List, Optional
import argparse


class DNSReconRunner:
    def __init__(self, threads: int = 10, timeout: int = 3):
        self.threads = threads
        self.timeout = timeout
    
    def run_standard(self, domain: str) -> Dict:
        """Run standard enumeration."""
        cmd = [
            "dnsrecon", "-d", domain, "-t", "std",
            "--threads", str(self.threads),
            "-j", f"/tmp/dnsrecon_{domain}_std.json"
        ]
        
        try:
            result = subprocess.run(cmd, capture_output=True, text=True, timeout=300)
            return self._parse_json_output(f"/tmp/dnsrecon_{domain}_std.json")
        except Exception as e:
            return {"error": str(e)}
    
    def run_bruteforce(self, domain: str, wordlist: str) -> Dict:
        """Run brute-force enumeration."""
        cmd = [
            "dnsrecon", "-d", domain, "-t", "brt",
            "-D", wordlist,
            "--threads", str(self.threads),
            "-j", f"/tmp/dnsrecon_{domain}_brt.json"
        ]
        
        try:
            result = subprocess.run(cmd, capture_output=True, text=True, timeout=600)
            return self._parse_json_output(f"/tmp/dnsrecon_{domain}_brt.json")
        except Exception as e:
            return {"error": str(e)}
    
    def run_zone_transfer(self, domain: str, nameservers: List[str]) -> Dict:
        """Test zone transfer against multiple nameservers."""
        results = {}
        for ns in nameservers:
            cmd = [
                "dnsrecon", "-d", domain, "-t", "zt",
                "--dns-servers", ns,
                "-j", f"/tmp/dnsrecon_{domain}_zt_{ns}.json"
            ]
            
            try:
                subprocess.run(cmd, capture_output=True, text=True, timeout=30)
                results[ns] = self._parse_json_output(f"/tmp/dnsrecon_{domain}_zt_{ns}.json")
            except Exception as e:
                results[ns] = {"error": str(e)}
        
        return results
    
    def _parse_json_output(self, json_file: str) -> Dict:
        """Parse dnsrecon JSON output."""
        try:
            with open(json_file, 'r') as f:
                return json.load(f)
        except FileNotFoundError:
            return {"error": f"Output file not found: {json_file}"}
        except json.JSONDecodeError:
            return {"error": "Invalid JSON output"}
    
    def generate_report(self, results: Dict) -> str:
        """Generate human-readable report."""
        report = []
        report.append(f"DNSRecon Report")
        report.append(f"Generated: {datetime.now().isoformat()}")
        report.append("=" * 50)
        
        if "error" in results:
            report.append(f"Error: {results['error']}")
            return '\n'.join(report)
        
        # Parse standard results
        if "standard_enum" in results:
            for record in results["standard_enum"]:
                report.append(f"Type: {record.get('type', 'unknown')}")
                report.append(f"  Name: {record.get('name', 'N/A')}")
                report.append(f"  Address: {record.get('address', 'N/A')}")
                report.append("")
        
        return '\n'.join(report)


def main():
    parser = argparse.ArgumentParser(description="DNSRecon API Wrapper")
    parser.add_argument("domain", help="Target domain")
    parser.add_argument("-t", "--type", choices=["std", "brt", "zt"], default="std")
    parser.add_argument("-w", "--wordlist", help="Wordlist for brute-force")
    parser.add_argument("--threads", type=int, default=10)
    parser.add_argument("-o", "--output", help="Output JSON file")
    
    args = parser.parse_args()
    
    runner = DNSReconRunner(threads=args.threads)
    
    if args.type == "std":
        results = runner.run_standard(args.domain)
    elif args.type == "brt":
        if not args.wordlist:
            args.wordlist = "/usr/share/wordlists/subdomains-top1mil-5000.txt"
        results = runner.run_bruteforce(args.domain, args.wordlist)
    elif args.type == "zt":
        # Get nameservers first
        import subprocess
        ns_result = subprocess.run(
            ["dig", "+short", "NS", args.domain],
            capture_output=True, text=True
        )
        nameservers = ns_result.stdout.strip().split('\n')
        results = runner.run_zone_transfer(args.domain, nameservers)
    
    if args.output:
        with open(args.output, 'w') as f:
            json.dump(results, f, indent=2)
        print(f"[+] Results saved to: {args.output}")
    
    print(runner.generate_report(results))


if __name__ == "__main__":
    main()
```

### Output Processing

#### Extract Subdomains
```bash
dnsrecon -d example.com -t brt -D wordlist.txt | \
    grep -E '^\[\*\].*->' | \
    sed 's/\[\*\] //' | \
    awk -F' -> ' '{print $1}' | \
    sort -u > subdomains.txt
```

#### Extract IP Addresses
```bash
dnsrecon -d example.com | \
    grep -oE '([0-9]{1,3}\.){3}[0-9]{1,3}' | \
    sort -u > ips.txt
```

### Tool Chaining

#### With Nmap
```bash
dnsrecon -d example.com -t brt -D wordlist.txt | \
    grep -oE '([0-9]{1,3}\.){3}[0-9]{1,3}' | \
    sort -u > /tmp/target_ips.txt

nmap -sV -sC -iL /tmp/target_ips.txt -oA nmap_dnsrecon_results
```

#### With Amass
```bash
# Combine dnsrecon with amass for comprehensive subdomain discovery
dnsrecon -d example.com -t brt -D wordlist.txt -j dnsrecon.json &
amass enum -passive -d example.txt -o amass_results.txt &
wait

# Merge results
cat dnsrecon.json | jq -r '.[].address' | sort -u > all_ips.txt
```

---

## Chapter 5: Advanced Usage

### Evasion Techniques

#### Rate Limiting Evasion
```bash
# Slow queries to avoid detection
dnsrecon -d example.com -t brt \
    --threads 2 \
    --timeout 10 \
    --lifetime 600 \
    -D wordlist.txt

# Use multiple DNS servers
dnsrecon -d example.com -t brt \
    --dns-servers 8.8.8.8,1.1.1.1,208.67.222.222 \
    -D wordlist.txt
```

#### DNS over HTTPS (DoH) Proxy
```bash
# Route through local DoH proxy
dnsrecon -d example.com --dns-servers 127.0.0.1 -t brt -D wordlist.txt
```

### Advanced Script Development

```python
#!/usr/bin/env python3
"""
advanced_dnsrecon.py - Multi-method DNS enumeration with evasion
"""

import subprocess
import asyncio
import aiohttp
import json
import random
from datetime import datetime
from pathlib import Path
from typing import Dict, Set, Optional


class AdvancedDNSRecon:
    def __init__(self, domain: str, config: Dict):
        self.domain = domain
        self.config = config
        self.output_dir = Path(config.get('output_dir', './results'))
        self.output_dir.mkdir(exist_ok=True)
        
        self.discovered: Dict[str, Set[str]] = {
            'subdomains': set(),
            'ips': set(),
            'nameservers': set(),
            'mail_servers': set()
        }
    
    async def run_comprehensive(self) -> Dict:
        """Run all enumeration methods."""
        
        methods = [
            self._run_standard_enum(),
            self._run_bruteforce(),
            self._query_ct_logs(),
            self._query_dns_over_https()
        ]
        
        await asyncio.gather(*methods, return_exceptions=True)
        
        return self._generate_report()
    
    async def _run_standard_enum(self):
        cmd = [
            "dnsrecon", "-d", self.domain, "-t", "std",
            "--threads", str(self.config.get('threads', 10)),
            "-j", str(self.output_dir / "standard.json")
        ]
        
        process = await asyncio.create_subprocess_exec(
            *cmd, stdout=asyncio.subprocess.PIPE, stderr=asyncio.subprocess.PIPE
        )
        await process.communicate()
        
        # Parse results
        try:
            with open(self.output_dir / "standard.json") as f:
                data = json.load(f)
                for record in data:
                    if 'address' in record:
                        self.discovered['ips'].add(record['address'])
                    if 'name' in record:
                        self.discovered['subdomains'].add(record['name'])
        except Exception:
            pass
    
    async def _run_bruteforce(self):
        wordlist = self.config.get('wordlist', '/usr/share/wordlists/subdomains-top1mil-5000.txt')
        cmd = [
            "dnsrecon", "-d", self.domain, "-t", "brt",
            "-D", wordlist,
            "--threads", str(self.config.get('threads', 10)),
            "-j", str(self.output_dir / "bruteforce.json")
        ]
        
        process = await asyncio.create_subprocess_exec(
            *cmd, stdout=asyncio.subprocess.PIPE, stderr=asyncio.subprocess.PIPE
        )
        await process.communicate()
        
        try:
            with open(self.output_dir / "bruteforce.json") as f:
                data = json.load(f)
                for record in data:
                    if 'address' in record:
                        self.discovered['ips'].add(record['address'])
                    if 'name' in record:
                        self.discovered['subdomains'].add(record['name'])
        except Exception:
            pass
    
    async def _query_ct_logs(self):
        """Query Certificate Transparency logs."""
        url = f"https://crt.sh/?q=%.{self.domain}&output=json"
        
        async with aiohttp.ClientSession() as session:
            try:
                async with session.get(url, timeout=aiohttp.ClientTimeout(total=30)) as resp:
                    if resp.status == 200:
                        data = await resp.json()
                        for entry in data:
                            name = entry.get('name_value', '')
                            if name:
                                for subdomain in name.split('\n'):
                                    if subdomain.endswith(self.domain):
                                        self.discovered['subdomains'].add(subdomain.strip())
            except Exception:
                pass
    
    async def _query_dns_over_https(self):
        """Query DNS over HTTPS for additional records."""
        async with aiohttp.ClientSession() as session:
            for sub in ['www', 'mail', 'ftp', 'api', 'dev', 'staging']:
                query = f"{sub}.{self.domain}"
                url = f"https://dns.google/resolve?name={query}&type=A"
                
                try:
                    async with session.get(url) as resp:
                        if resp.status == 200:
                            data = await resp.json()
                            if data.get('Answer'):
                                self.discovered['subdomains'].add(query)
                                for answer in data['Answer']:
                                    if answer.get('data'):
                                        self.discovered['ips'].add(answer['data'])
                except Exception:
                    pass
                
                if self.config.get('evasion'):
                    await asyncio.sleep(random.uniform(0.5, 2.0))
    
    def _generate_report(self) -> Dict:
        report = {
            "domain": self.domain,
            "timestamp": datetime.now().isoformat(),
            "summary": {
                "total_subdomains": len(self.discovered['subdomains']),
                "total_ips": len(self.discovered['ips']),
                "total_nameservers": len(self.discovered['nameservers']),
                "total_mail_servers": len(self.discovered['mail_servers'])
            },
            "subdomains": sorted(list(self.discovered['subdomains'])),
            "ip_addresses": sorted(list(self.discovered['ips'])),
            "nameservers": sorted(list(self.discovered['nameservers'])),
            "mail_servers": sorted(list(self.discovered['mail_servers']))
        }
        
        report_file = self.output_dir / f"{self.domain}_report.json"
        with open(report_file, 'w') as f:
            json.dump(report, f, indent=2)
        
        return report
```

### Scaling Techniques

```bash
#!/bin/bash
# scale_dnsrecon.sh - Enumerate many domains efficiently

DOMAINS_FILE="domains.txt"
WORDLIST="/usr/share/wordlists/subdomains-top1mil-5000.txt"
OUTPUT_BASE="./scaled_results"
MAX_JOBS=10

mkdir -p "$OUTPUT_BASE"

scan_domain() {
    local domain=$1
    local output_dir="$OUTPUT_BASE/$domain"
    mkdir -p "$output_dir"
    
    dnsrecon -d "$domain" -t std -t brt -D "$WORDLIST" \
        --threads 5 \
        --xml "$output_dir/dnsrecon.xml" \
        -j "$output_dir/dnsrecon.json" \
        > "$output_dir/dnsrecon.txt" 2>&1
    
    echo "[+] Completed: $domain"
}

export -f scan_domain
export WORDLIST OUTPUT_BASE

cat "$DOMAINS_FILE" | xargs -P "$MAX_JOBS" -I {} bash -c 'scan_domain "$@"' _ {}
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: Network Audit - Corporate DNS Security Assessment

**Objective:** Comprehensive DNS security assessment for a corporate client.

#### Step 1: Initial Reconnaissance
```bash
# Standard enumeration
dnsrecon -d megacorp.com -t std \
    --xml megacorp_standard.xml \
    -j megacorp_standard.json \
    | tee megacorp_recon.txt
```

**Output:**
```
[*] Performing General Enumeration of Domain:megacorp.com
[*] DNS Servers for megacorp.com:
    ns1.megacorp.com [10.0.0.1]
    ns2.megacorp.com [10.0.0.2]
[*] Host Address records for megacorp.com:
    megacorp.com. 3600 IN A 203.0.113.10
    megacorp.com. 3600 IN AAAA 2001:db8::1
[*] Nameservers for megacorp.com:
    ns1.megacorp.com. 3600 IN A 10.0.0.1
    ns2.megacorp.com. 3600 IN A 10.0.0.2
[*] Mail Servers for megacorp.com:
    mail.megacorp.com. 3600 IN A 203.0.113.11
[*] SOA Record for megacorp.com:
    megacorp.com. 3600 IN SOA ns1.megacorp.com. admin.megacorp.com. 2024010101 3600 900 604800 86400
```

#### Step 2: Zone Transfer Testing
```bash
# Test zone transfer
dnsrecon -d megacorp.com -t zt \
    --xml megacorp_zonetransfer.xml \
    -j megacorp_zonetransfer.json \
    | tee megacorp_zt_results.txt
```

**Finding:** Zone transfer successful on ns2.megacorp.com - reveals internal hostnames including `jenkins.internal.megacorp.com`, `gitlab.internal.megacorp.com`, and `db.internal.megacorp.com`.

#### Step 3: Subdomain Brute-Force
```bash
# Comprehensive subdomain discovery
dnsrecon -d megacorp.com -t brt \
    -D /usr/share/wordlists/subdomains-top1mil-5000.txt \
    --threads 15 \
    --xml megacorp_bruteforce.xml \
    -j megacorp_bruteforce.json \
    | tee megacorp_subdomains.txt
```

#### Step 4: Service Discovery
```bash
# Extract IPs and scan
grep -oE '([0-9]{1,3}\.){3}[0-9]{1,3}' megacorp_subdomains.txt | \
    sort -u > megacorp_ips.txt

nmap -sV -sC -iL megacorp_ips.txt -oA megacorp_services
```

### Scenario 2: CTF Challenge - DNS Detective

**Objective:** Find the flag hidden in DNS records.

```bash
# Step 1: Comprehensive enumeration
dnsrecon -d ctf.example.com -t std -t brt \
    -D /usr/share/wordlists/dnsrecon.txt

# Step 2: Zone transfer check
dnsrecon -d ctf.example.com -t zt

# Output shows zone transfer successful:
# [*] Zone Transfer was successful for ns1.ctf.example.com
# flag.ctf.example.com.    3600    IN    TXT    "FLAG{dns_zone_transfer_is_dangerous}"
# hidden.ctf.example.com.  3600    IN    A      10.10.10.50

# Step 3: Extract flag
dig TXT flag.ctf.example.com

# Step 4: Check hidden subdomain
curl http://hidden.ctf.example.com/
# Returns: "Welcome to the hidden service. Flag: CTF{subdomain_discovery_101}"
```

### Scenario 3: Bug Bounty - Subdomain Discovery

**Objective:** Find hidden subdomains for bug bounty program.

#### Phase 1: Multi-Method Enumeration
```bash
# Standard enumeration
dnsrecon -d program.example.com -t std \
    -j program_std.json

# Brute-force
dnsrecon -d program.example.com -t brt \
    -D /usr/share/wordlists/subdomains-top1mil-5000.txt \
    --threads 20 \
    -j program_brt.json

# Google enumeration
dnsrecon -d program.example.com -t goo \
    -j program_goo.json

# Certificate transparency
dnsrecon -d program.example.com -t crt \
    -j program_crt.json
```

#### Phase 2: Merge and Validate
```bash
# Merge all results
for f in program_*.json; do
    jq -r '.[].name' "$f"
done | sort -u > all_subdomains.txt

# Validate subdomains
while IFS= read -r subdomain; do
    ip=$(dig +short "$subdomain" | head -1)
    if [ -n "$ip" ]; then
        echo "[+] $subdomain -> $ip"
        echo "$subdomain,$ip" >> validated_subdomains.csv
    fi
done < all_subdomains.txt
```

#### Phase 3: High-Value Target Scan
```bash
# Scan high-value subdomains
nmap -sV -p 80,443,8080,8443 \
    $(grep -E '(admin|api|dev|staging)' validated_subdomains.csv | \
      cut -d',' -f2 | sort -u | tr '\n' ' ') \
    -oA bugbounty_highvalue
```

---

## Chapter 7: Detection & Defense

### How Defenders Detect DNSRecon

#### DNS Query Pattern Analysis

DNSRecon generates distinctive patterns:

- **High query rate** from single source
- **Sequential subdomain queries** (admin., www., mail., ftp., etc.)
- **AXFR queries** for zone transfer attempts
- **Multiple query types** in rapid succession

#### IDS/IPS Rules

```bash
# Suricata rules for DNSRecon detection
alert dns any any -> any any (msg:"DNS Subdomain Brute Force - DNSRecon"; \
    dns.query; threshold:type both, track by_src, count 100, seconds 60; \
    classtype:attempted-recon; sid:1000001; rev:1;)

alert dns any any -> any any (msg:"DNS Zone Transfer AXFR Attempt"; \
    dns.opcode: 0; content:"|00 FC|"; offset:2; depth:2; \
    classtype:attempted-recon; sid:1000002; rev:1;)

alert dns any any -> any any (msg:"DNS Sequential Subdomain Enumeration"; \
    dns.query; pcre:"/^(admin|www|mail|ftp|smtp|ns1|ns2|dev|staging|test)\./i"; \
    threshold:type both, track by_src, count 50, seconds 30; \
    classtype:attempted-recon; sid:1000003; rev:1;)
```

#### Log Analysis
```bash
#!/bin/bash
# dnsrecon_detection.sh

DNS_LOG="/var/log/bind/queries.log"
ALERT_THRESHOLD=100

echo "=== DNS Query Analysis ==="
awk '{print $1}' "$DNS_LOG" | sort | uniq -c | sort -rn | head -20

echo ""
echo "=== Potential Brute-Force Activity ==="
awk '{print $1}' "$DNS_LOG" | sort | uniq -c | \
    awk -v threshold="$ALERT_THRESHOLD" '$1 > threshold {print "[!] Suspicious: "$2" ("$1" queries)"}'
```

### DNS Server Hardening

```bash
# BIND9 configuration to prevent zone transfers
cat > /etc/bind/named.conf.options << 'EOF'
options {
    directory "/var/cache/bind";
    
    // Restrict zone transfers
    allow-transfer { none; };
    
    // Only allow queries from trusted networks
    allow-query { localhost; 192.168.1.0/24; };
    
    // Disable recursion for authoritative servers
    recursion no;
    
    // Rate limiting
    rate-limit {
        responses-per-second 10;
        window 5;
        log-only no;
    };
};
EOF

systemctl restart bind9
```

### Network-Level Defenses

```bash
# iptables rules to rate-limit DNS queries
iptables -A INPUT -p udp --dport 53 -m hashlimit \
    --hashlimit-name dns \
    --hashlimit-mode srcip \
    --hashlimit-upto 10/second \
    --hashlimit-burst 20 \
    --hashlimit-htable-expire 30000 \
    -j ACCEPT

iptables -A INPUT -p udp --dport 53 -j DROP
```

### Blue Team Response

1. **Monitor for reconnaissance** - Watch for DNS query rate anomalies
2. **Analyze scan patterns** - Identify sequential subdomain queries
3. **Check for zone transfer attempts** - Alert on AXFR queries
4. **Block scanner IPs** - Update firewall rules
5. **Document the incident** - Record all findings
6. **Report to management** - Notify stakeholders

---

## Chapter 8: Troubleshooting

### Common Errors

#### Error: "No module named 'dns'"
**Solution:**
```bash
pip3 install dnspython
# or
sudo apt install python3-dnspython
```

#### Error: "dnsrecon: command not found"
**Solution:**
```bash
which dnsrecon
# If installed but not in PATH
export PATH=$PATH:/usr/share/dnsrecon
```

#### Error: "Timeout during zone transfer"
**Solution:**
```bash
# Increase timeout
dnsrecon -d example.com -t zt --timeout 30

# Check connectivity
dig @ns1.example.com example.com AXFR
```

#### Error: "No reverse records found"
**Solution:**
```bash
# Verify IP range is correct
dnsrecon -d example.com -t rvl -r 192.168.1.0/24 --lifetime 60
```

#### Error: "Brute-force produces no results"
**Solution:**
```bash
# Check wordlist
wc -l /usr/share/wordlists/subdomains-top1mil-5000.txt

# Try different wordlist
dnsrecon -d example.com -t brt -D /usr/share/wordlists/dnsrecon.txt
```

### Performance Optimization

#### Slow Enumeration
```bash
# Increase threads
dnsrecon -d example.com --threads 20

# Use faster DNS servers
dnsrecon -d example.com --dns-servers 8.8.8.8,1.1.1.1

# Reduce timeout
dnsrecon -d example.com --timeout 2
```

#### High Memory Usage
```bash
# Reduce thread count
dnsrecon -d example.com --threads 5

# Use smaller wordlist
dnsrecon -d example.com -t brt -D small_wordlist.txt
```

### FAQ

**Q: Does dnsrecon work with DNS over HTTPS (DoH)?**
A: Not directly. Set up a local DoH proxy and point dnsrecon to it: `dnsrecon --dns-servers 127.0.0.1 example.com`

**Q: Can dnsrecon enumerate IPv6 addresses?**
A: Yes, DNSRecon fetches AAAA records by default.

**Q: How do I use dnsrecon with Tor?**
```bash
sudo systemctl start tor
proxychains dnsrecon -d example.com
```

**Q: Is dnsrecon legal to use?**
A: dnsrecon itself is legal. Using it against systems without authorization is illegal. Always get written permission.

**Q: What's the difference between dnsrecon and dnsenum?**
A: dnsrecon is more comprehensive with multiple enumeration methods (Google, Bing, cache snooping), richer output formats (JSON, XML, CSV), and Python extensibility. dnsenum focuses on zone transfers and brute-force with search engine integration.

**Q: Can I integrate dnsrecon with Metasploit?**
A: Yes. DNSRecon output can be imported into Metasploit using XML format: `msfconsole -x "db_import dnsrecon.xml; hosts; services"`

---

## Resources

### Official Documentation
- [DNSRecon GitHub Repository](https://github.com/darkoperator/dnsrecon)
- [DNSRecon Wiki](https://github.com/darkoperator/dnsrecon/wiki)

### Wordlists
- [SecLists DNS Subdomains](https://github.com/danielmiessler/SecLists/tree/master/Discovery/DNS)
- [DNSRecon Default Wordlist](https://github.com/darkoperator/dnsrecon/blob/master/dnsrecon.txt)

### Related Tools
- **dnsenum:** Perl-based DNS enumeration
- **dnsmap:** Fast C-based subdomain brute-forcing
- **massdns:** High-speed bulk DNS resolution
- **sublist3r:** Multi-source subdomain enumeration

### Learning Resources
- [DNS Enumeration Techniques](https://www.offensive-security.com/metasploit-unleashed/dns-enumeration/)
- [Zone Transfer Testing](https://book.hacktricks.xyz/network-services-pentesting/pentesting-dns)
- [Bug Bounty DNS Recon](https://github.com/arkadiyt/bounty-targets-data)
