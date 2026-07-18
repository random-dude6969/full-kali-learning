---
tool_name: "dnsenum"
display_name: "DNSenum"
mitre_tactic: "reconnaissance"
mitre_tactic_id: "TA0043"
mitre_subcategory: "dns_reconnaissance"
install_command: "sudo apt install dnsenum"
version: "1.2.6"
github_repo: "https://github.com/fwaeyt/dnsenum"
kali_tools_page: "https://www.kali.org/tools/dnsenum/"
difficulty_level: "beginner"
requires_root: false
gui_available: false
tags: ["dns", "enumeration", "zone-transfer", "brute-force", "subdomains", "reconnaissance"]
related_tools: ["dnsrecon", "dnsmap", "massdns", "dnstracer", "dnswalk"]
---

# DNSenum - DNS Enumeration Tool

## Chapter 1: Introduction & Overview

### What is DNSenum?

DNSenum is a comprehensive DNS enumeration tool written in Perl that automates the process of discovering DNS information about a target domain. It performs zone transfer testing, subdomain brute-forcing, email address enumeration, reverse DNS lookups, and Google/Bing search engine queries to build a complete picture of a domain's DNS infrastructure.

Unlike simpler DNS tools, dnsenum combines multiple reconnaissance techniques into a single automated workflow, making it one of the most thorough DNS enumeration tools available for penetration testers and security auditors.

### History & Background

- **Created:** dnsenum was originally developed by Filip Waeytens
- **Language:** Perl (requires Net::DNS, Net::IP, and XML::Writer modules)
- **License:** GPLv2
- **Maintained:** Active community on GitHub

DNSenum evolved from the need to automate multiple DNS reconnaissance steps that were previously done manually with tools like `dig`, `nslookup`, and `host`. It became a standard tool in Kali Linux's DNS reconnaissance toolkit.

### When to Use DNSenum

- **Penetration testing engagements** - Map DNS infrastructure before exploitation
- **Security audits** - Test for zone transfer vulnerabilities
- **Bug bounty programs** - Discover hidden subdomains and attack surfaces
- **Red team operations** - Gather intelligence for social engineering
- **CTF competitions** - Solve DNS-related challenges
- **Network administration** - Audit your own DNS configuration

### When NOT to Use This Tool

- Without explicit written authorization from the domain owner
- Against critical infrastructure without proper coordination
- For malicious reconnaissance or attack preparation
- When simpler tools (dig, nslookup) suffice for the task

### Legal and Ethical Considerations

- **Always obtain written authorization** before scanning domains you don't own
- **DNS queries are logged** - your activities will be visible in DNS server logs
- **Zone transfer attempts may trigger alerts** on properly monitored DNS servers
- **Brute-forcing can be detected** as reconnaissance activity
- **Document everything** - keep records of your authorization and findings
- **Follow responsible disclosure** - report vulnerabilities properly

### Key Concepts

- **Zone Transfer (AXFR):** A DNS replication mechanism that, if misconfigured, allows anyone to download the entire DNS zone file
- **Subdomain Brute-force:** Systematically trying common subdomain prefixes (www, mail, ftp, etc.) against a domain
- **Reverse DNS:** Resolving IP addresses back to hostnames
- **DNS Records:** A (IPv4), AAAA (IPv6), MX (mail), NS (nameserver), TXT (text), CNAME (alias), SOA (authority)
- **Wildcard DNS:** A DNS configuration where any non-existent subdomain resolves to a specific IP
- **Recursive Query:** Following the full DNS resolution chain from root servers to authoritative servers

---

## Chapter 2: Installation & Setup

### System Requirements

- **Minimum RAM:** 256 MB
- **Disk Space:** 50 MB for installation
- **Perl Version:** 5.10+ with modules (Net::DNS, Net::IP, Net::Netmask, XML::Writer)
- **Network:** Any network interface with DNS access
- **Privileges:** Not required for basic enumeration; useful for raw socket operations

### Installation Methods

#### Method 1: APT (Recommended)

```bash
sudo apt update
sudo apt install dnsenum
```

**Output:**
```
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following NEW packages will be installed:
  dnsenum
0 upgraded, 1 newly installed, 0 to remove and 142 not upgraded.
Need to get 31.2 kB of archives.
After this operation, 155 kB of additional disk space will be used.
Get:1 http://kali.download/kali kali-rolling/main amd64 dnsenum all 1.2.6-1kali1 [31.2 kB]
Fetched 31.2 kB in 1s (42.3 kB/s)
Selecting previously unselected package dnsenum.
Setting up dnsenum (1.2.6-1kali1) ...
Processing triggers for man-db (2.10.2-1) ...
```

#### Method 2: From Source

```bash
git clone https://github.com/fwaeyt/dnsenum.git
cd dnsenum
sudo cp dnsenum.pl /usr/local/bin/dnsenum
chmod +x /usr/local/bin/dnsenum
```

#### Method 3: Manual Perl Module Installation

```bash
# Install required Perl modules
sudo apt install libnet-dns-perl libnet-ip-perl libnet-netmask-perl libxml-writer-perl

# Verify modules are available
perl -e 'use Net::DNS; use Net::IP; use Net::Netmask; use XML::Writer; print "All modules OK\n"'
```

### Configuration

DNSenum stores its default wordlist at:
- `/usr/share/wordlists/dnsenum.txt` (built-in)
- Custom wordlists can be specified with `-f` flag

### Verification

```bash
dnsenum --version
# Output: dnsenum 1.2.6

dnsenum -h
# Shows full help with all available options
```

---

## Chapter 3: Basic Usage

### First Run

The simplest way to start is with a basic enumeration:

```bash
dnsenum example.com
```

**Output:**
```
dnsenum 1.2.6

Information Security Solutions & Services (BullGuard - dissec.com)

[-] Warning: Can't determine nameservers for example.com. Trying the bad way!
[-] No Nameservers found for example.com

[-] No zone transfer possible for example.com
[-] Could not enumerate Nameservers for example.com
[-] Could not resolve Host names
[-] Could not enumerate Mail Servers for example.com
```

### Essential Commands

#### Basic DNS Enumeration
```bash
dnsenum example.com
```

#### Full Enumeration with Brute-Force
```bash
dnsenum --enum example.com
```

**Output:**
```
dnsenum 1.2.6

Information Security Solutions & Services (BullGuard - dissec.com)

-----example.com-----

Host address:   93.184.216.34
Host address:   93.184.216.35

Name servers:
  ns1.example.com.       (8.8.8.8)
  ns2.example.com.       (8.8.4.4)

Mail servers:
  mail.example.com.      (93.184.216.36)
  mail2.example.com.     (93.184.216.37)

Brute-forcing example.com using /usr/share/wordlists/dnsenum.txt:
www.example.com.          3600    IN      A       93.184.216.34
mail.example.com.         3600    IN      A       93.184.216.36
ftp.example.com.          3600    IN      A       93.184.216.38
api.example.com.          3600    IN      A       93.184.216.39
dev.example.com.          3600    IN      A       93.184.216.40
staging.example.com.      3600    IN      A       93.184.216.41

[+] 6 subdomains found
```

#### Custom Wordlist
```bash
dnsenum --enum -f /usr/share/wordlists/subdomains-top1mil-5000.txt example.com
```

#### Zone Transfer Only
```bash
dnsenum --zone --noreverse example.com
```

**Output:**
```
[-] Zone Transfer failed for 8.8.8.8: REFUSED
[-] Zone Transfer failed for 8.8.4.4: REFUSED
[-] No Zone Transfers found for example.com
```

#### Recursive Enumeration
```bash
dnsenum --enum --recursion yes example.com
```

#### Limit Threads
```bash
dnsenum --threads 5 --timeout 10 example.com
```

#### XML Output
```bash
dnsenum --enum --xmlout results.xml example.com
```

### Complete Flags Reference

#### Enumeration Flags
| Flag | Description | Example |
|------|-------------|---------|
| `--enum` | Enable brute-force enumeration | `dnsenum --enum example.com` |
| `--zone` | Perform zone transfer test | `dnsenum --zone example.com` |
| `--noreverse` | Skip reverse lookups | `dnsenum --noreverse example.com` |
| `--recursion yes` | Enable recursive enumeration | `dnsenum --recursion yes example.com` |
| `-f` | Specify wordlist file | `dnsenum -f wordlist.txt example.com` |

#### Output Flags
| Flag | Description | Example |
|------|-------------|---------|
| `--xmlout` | Output results to XML file | `dnsenum --xmlout results.xml example.com` |
| `--csvout` | Output results to CSV format | `dnsenum --csvout example.com` |

#### Performance Flags
| Flag | Description | Example |
|------|-------------|---------|
| `--threads` | Number of threads (default: 5) | `dnsenum --threads 10 example.com` |
| `--timeout` | Timeout in seconds (default: 3) | `dnsenum --timeout 10 example.com` |
| `--retry` | Number of retries (default: 3) | `dnsenum --retry 5 example.com` |

#### DNS Server Flags
| Flag | Description | Example |
|------|-------------|---------|
| `--dns-servers` | Specify DNS servers | `dnsenum --dns-servers 8.8.8.8,1.1.1.1 example.com` |
| `--source-ip` | Spoof source IP | `dnsenum --source-ip 10.0.0.1 example.com` |
| `--rskip` | Skip specific nameserver | `dnsenum --rskip ns1.example.com example.com` |

#### Brute-Force Flags
| Flag | Description | Example |
|------|-------------|---------|
| `-f` | Custom wordlist file | `dnsenum -f custom.txt example.com` |
| `--limit` | Limit results | `dnsenum --limit 100 example.com` |

#### Search Engine Flags
| Flag | Description | Example |
|------|-------------|---------|
| `--google-exchange` | Google search depth | `dnsenum --google-exchange 100 example.com` |
| `--bing-exchange` | Bing search depth | `dnsenum --bing-exchange 100 example.com` |

#### Miscellaneous Flags
| Flag | Description | Example |
|------|-------------|---------|
| `-h` | Show help | `dnsenum -h` |
| `--version` | Show version | `dnsenum --version` |

---

## Chapter 4: Intermediate Usage

### Scripting & Automation

#### Bash: Batch Domain Enumeration

```bash
#!/bin/bash
# dnsenum_batch.sh - Enumerate multiple domains from a file

DOMAINS_FILE="domains.txt"
OUTPUT_DIR="./dnsenum_results"
WORDLIST="/usr/share/wordlists/subdomains-top1mil-5000.txt"
THREADS=10
TIMEOUT=5

mkdir -p "$OUTPUT_DIR"

while IFS= read -r domain; do
    echo "[*] Enumerating: $domain"
    dnsenum \
        --enum \
        --threads "$THREADS" \
        --timeout "$TIMEOUT" \
        -f "$WORDLIST" \
        --xmlout "$OUTPUT_DIR/${domain}_dnsenum.xml" \
        "$domain" | tee "$OUTPUT_DIR/${domain}_dnsenum.txt"
    
    echo "[+] Completed: $domain"
    echo "---"
done < "$DOMAINS_FILE"

echo "[*] All domains enumerated. Results in: $OUTPUT_DIR"
```

#### Bash: Zone Transfer Checker

```bash
#!/bin/bash
# zone_transfer_check.sh - Check zone transfer across multiple domains

DOMAINS_FILE="targets.txt"
LOG_FILE="zone_transfer_results.log"

echo "Zone Transfer Check - $(date)" > "$LOG_FILE"
echo "========================================" >> "$LOG_FILE"

while IFS= read -r domain; do
    echo "[*] Testing zone transfer: $domain"
    RESULT=$(dnsenum --zone --noreverse "$domain" 2>&1)
    
    if echo "$RESULT" | grep -q "Zone Transfer successful"; then
        echo "[!] VULNERABLE: $domain allows zone transfer" | tee -a "$LOG_FILE"
        echo "$RESULT" >> "$LOG_FILE"
    else
        echo "[-] Secure: $domain" | tee -a "$LOG_FILE"
    fi
    echo "---" >> "$LOG_FILE"
done < "$DOMAINS_FILE"

echo "[+] Check complete. Results: $LOG_FILE"
```

#### Bash: Parallel Multi-Domain Scanner

```bash
#!/bin/bash
# parallel_dnsenum.sh - Scan multiple domains in parallel

DOMAINS_FILE="domains.txt"
MAX_PARALLEL=5
OUTPUT_DIR="./parallel_results"
WORDLIST="/usr/share/wordlists/subdomains-top1mil-5000.txt"

mkdir -p "$OUTPUT_DIR"

scan_domain() {
    local domain=$1
    local output_file="$OUTPUT_DIR/${domain//\./_}_results.txt"
    
    dnsenum --enum --threads 5 --timeout 5 \
        -f "$WORDLIST" \
        "$domain" > "$output_file" 2>&1
    
    echo "[+] Finished: $domain -> $output_file"
}

export -f scan_domain
export WORDLIST

cat "$DOMAINS_FILE" | xargs -P "$MAX_PARALLEL" -I {} bash -c 'scan_domain "$@"' _ {}

echo "[+] All scans complete"
```

### Python Automation

#### Python: DNS Enumeration Wrapper

```python
#!/usr/bin/env python3
"""
dnsenum_api.py - Python wrapper for dnsenum with structured output
"""

import subprocess
import json
import re
import sys
from datetime import datetime
from pathlib import Path
from typing import Dict, List, Optional
import argparse


class DNSenumRunner:
    def __init__(self, wordlist: str = "/usr/share/wordlists/subdomains-top1mil-5000.txt",
                 threads: int = 10, timeout: int = 5):
        self.wordlist = wordlist
        self.threads = threads
        self.timeout = timeout
        self.results = {}
    
    def run_enum(self, domain: str, zone_transfer: bool = True,
                 brute_force: bool = True, recursive: bool = False) -> Dict:
        """Run dnsenum and parse results into structured format."""
        
        cmd = ["dnsenum"]
        cmd.extend(["--threads", str(self.threads)])
        cmd.extend(["--timeout", str(self.timeout)])
        
        if zone_transfer:
            cmd.append("--zone")
        
        if brute_force and self.wordlist:
            cmd.extend(["--enum", "-f", self.wordlist])
        
        if recursive:
            cmd.append("--recursion")
            cmd.append("yes")
        
        cmd.append(domain)
        
        print(f"[*] Running: {' '.join(cmd)}")
        
        try:
            result = subprocess.run(
                cmd,
                capture_output=True,
                text=True,
                timeout=300
            )
            output = result.stdout
        except subprocess.TimeoutExpired:
            print(f"[!] Timeout enumerating {domain}")
            return {"error": "timeout"}
        except FileNotFoundError:
            print("[!] dnsenum not found. Install with: sudo apt install dnsenum")
            return {"error": "dnsenum not installed"}
        
        return self._parse_output(domain, output)
    
    def _parse_output(self, domain: str, output: str) -> Dict:
        """Parse dnsenum output into structured format."""
        
        parsed = {
            "domain": domain,
            "timestamp": datetime.now().isoformat(),
            "records": {},
            "nameservers": [],
            "mail_servers": [],
            "subdomains": [],
            "zone_transfer": None,
            "wildcard": None
        }
        
        lines = output.split('\n')
        current_section = None
        
        for line in lines:
            line = line.strip()
            
            if "Zone Transfer successful" in line:
                parsed["zone_transfer"] = True
            elif "Zone Transfer failed" in line:
                parsed["zone_transfer"] = False
            
            if "Wildcard detection" in line:
                parsed["wildcard"] = "enabled" if "enabled" in line else "disabled"
            
            if "Host address" in line:
                current_section = "addresses"
            elif "Name servers" in line:
                current_section = "nameservers"
            elif "Mail servers" in line:
                current_section = "mail_servers"
            
            if current_section == "addresses":
                ip_match = re.search(r'(\d+\.\d+\.\d+\.\d+)', line)
                if ip_match:
                    parsed["records"].setdefault("A", []).append(ip_match.group(1))
            
            elif current_section == "nameservers":
                ns_match = re.search(r'(\S+\.\S+)\s+\((\S+)\)', line)
                if ns_match:
                    parsed["nameservers"].append({
                        "hostname": ns_match.group(1),
                        "ip": ns_match.group(2)
                    })
            
            elif current_section == "mail_servers":
                mx_match = re.search(r'(\S+\.\S+)\s+\((\S+)\)', line)
                if mx_match:
                    parsed["mail_servers"].append({
                        "hostname": mx_match.group(1),
                        "ip": mx_match.group(2)
                    })
            
            if domain in line and line and not line.startswith("dnsenum"):
                subdomain = line.split()[0] if line.split() else None
                if subdomain and subdomain.endswith(domain):
                    parsed["subdomains"].append(subdomain)
        
        parsed["stats"] = {
            "total_addresses": len(parsed["records"].get("A", [])),
            "total_nameservers": len(parsed["nameservers"]),
            "total_mail_servers": len(parsed["mail_servers"]),
            "total_subdomains": len(parsed["subdomains"])
        }
        
        return parsed
    
    def save_json(self, results: Dict, output_file: str):
        """Save results to JSON file."""
        with open(output_file, 'w') as f:
            json.dump(results, f, indent=2)
        print(f"[+] Results saved to: {output_file}")
    
    def generate_report(self, results: Dict) -> str:
        """Generate human-readable report from results."""
        report = []
        report.append(f"DNS Enumeration Report: {results['domain']}")
        report.append(f"Generated: {results['timestamp']}")
        report.append("=" * 50)
        
        if results.get("zone_transfer"):
            report.append("[!] ZONE TRANSFER VULNERABLE")
        else:
            report.append("[-] Zone transfer not possible")
        
        report.append(f"\nNameservers ({results['stats']['total_nameservers']}):")
        for ns in results["nameservers"]:
            report.append(f"  - {ns['hostname']} ({ns['ip']})")
        
        report.append(f"\nMail Servers ({results['stats']['total_mail_servers']}):")
        for mx in results["mail_servers"]:
            report.append(f"  - {mx['hostname']} ({mx['ip']})")
        
        report.append(f"\nIP Addresses ({results['stats']['total_addresses']}):")
        for ip in results["records"].get("A", []):
            report.append(f"  - {ip}")
        
        return '\n'.join(report)


def main():
    parser = argparse.ArgumentParser(description="DNSenum API Wrapper")
    parser.add_argument("domain", help="Target domain")
    parser.add_argument("-f", "--wordlist", help="Wordlist file")
    parser.add_argument("-t", "--threads", type=int, default=10)
    parser.add_argument("-o", "--output", help="Output JSON file")
    parser.add_argument("--no-zone", action="store_true", help="Skip zone transfer")
    parser.add_argument("--no-brute", action="store_true", help="Skip brute-force")
    parser.add_argument("--recursive", action="store_true", help="Enable recursive enumeration")
    
    args = parser.parse_args()
    
    runner = DNSenumRunner(
        wordlist=args.wordlist or "/usr/share/wordlists/subdomains-top1mil-5000.txt",
        threads=args.threads
    )
    
    results = runner.run_enum(
        domain=args.domain,
        zone_transfer=not args.no_zone,
        brute_force=not args.no_brute,
        recursive=args.recursive
    )
    
    if args.output:
        runner.save_json(results, args.output)
    
    print(runner.generate_report(results))


if __name__ == "__main__":
    main()
```

### Output Processing

#### Extract Subdomains from Output
```bash
dnsenum --enum example.com | grep -E '^\s+\S+\.\S+\s' | awk '{print $1}' | sort -u > subdomains.txt
```

#### Extract IP Addresses
```bash
dnsenum example.com | grep -oE '([0-9]{1,3}\.){3}[0-9]{1,3}' | sort -u > ips.txt
```

### Tool Chaining

#### With Nmap
```bash
dnsenum --enum -f wordlist.txt example.com | \
    grep -oE '([0-9]{1,3}\.){3}[0-9]{1,3}' | \
    sort -u > /tmp/target_ips.txt

nmap -sV -sC -iL /tmp/target_ips.txt -oA nmap_dnsenum_results
```

#### With Masscan
```bash
dnsenum --enum example.com | \
    grep -oE '([0-9]{1,3}\.){3}[0-9]{1,3}' | \
    sort -u > /tmp/ips.txt

masscan -p1-65535 -iL /tmp/ips.txt --rate=1000 -oJ masscan_results.json
```

---

## Chapter 5: Advanced Usage

### Evasion Techniques

#### Rate Limiting Evasion
```bash
# Slow down requests to avoid detection
dnsenum --threads 2 --timeout 10 --retry 5 example.com

# Use multiple source IPs (requires multiple NICs or VPN)
for ip in 192.168.1.10 192.168.1.11 192.168.1.12; do
    dnsenum --source-ip "$ip" --threads 3 --timeout 15 example.com &
done
wait
```

#### DNS over HTTPS (DoH) Proxy
```bash
# Route DNS queries through DoH to avoid network monitoring
# Configure a local DoH proxy first
dnsenum --dns-servers 127.0.0.1 --timeout 5 example.com
```

#### Distributed Enumeration
```bash
#!/bin/bash
# distributed_dnsenum.sh - Distribute enumeration across multiple servers

TARGET="example.com"
SERVERS=("server1" "server2" "server3")
WORDLIST="/usr/share/wordlists/subdomains-top1mil-5000.txt"

# Split wordlist
split -l $(( $(wc -l < "$WORDLIST") / ${#SERVERS[@]} )) "$WORDLIST" /tmp/wordlist_chunk_

# Distribute across servers
i=0
for server in "${SERVERS[@]}"; do
    i=$((i + 1))
    scp /tmp/wordlist_chunk_* "$server:/tmp/"
    ssh "$server" "dnsenum --enum -f /tmp/wordlist_chunk_aa $TARGET > /tmp/results_$i.txt" &
done
wait

# Merge results
cat /tmp/results_*.txt | sort -u > merged_results.txt
```

### Advanced Script Development

```python
#!/usr/bin/env python3
"""
advanced_dnsenum.py - Custom DNS enumeration with evasion and scaling
"""

import subprocess
import asyncio
import aiohttp
import json
import random
from datetime import datetime
from pathlib import Path
from typing import List, Dict, Set, Optional


class AdvancedDNSEnum:
    def __init__(self, config: Dict):
        self.config = config
        self.domain = config['domain']
        self.wordlist = config.get('wordlist', '/usr/share/wordlists/subdomains-top1mil-5000.txt')
        self.threads = config.get('threads', 5)
        self.timeout = config.get('timeout', 10)
        self.evasion = config.get('evasion', False)
        self.output_dir = Path(config.get('output_dir', './results'))
        self.output_dir.mkdir(exist_ok=True)
        
        self.discovered_subdomains: Set[str] = set()
        self.discovered_ips: Set[str] = set()
        self.zone_transfers: Dict[str, bool] = {}
    
    async def _throttled_dns_query(self, session, subdomain: str) -> Optional[Dict]:
        if self.evasion:
            await asyncio.sleep(random.uniform(0.5, 2.0))
        try:
            url = f"https://dns.google/resolve?name={subdomain}&type=A"
            async with session.get(url, timeout=aiohttp.ClientTimeout(total=self.timeout)) as resp:
                if resp.status == 200:
                    return await resp.json()
        except Exception:
            pass
        return None
    
    async def enumerate_subdomains(self, wordlist: List[str]) -> Set[str]:
        print(f"[*] Starting enumeration for {self.domain}")
        
        await self._run_dnsenum_bruteforce(wordlist)
        await self._ct_log_enum()
        await self._doh_bruteforce(wordlist)
        
        print(f"[+] Total unique subdomains: {len(self.discovered_subdomains)}")
        return self.discovered_subdomains
    
    async def _run_dnsenum_bruteforce(self, wordlist: List[str]):
        chunk_file = self.output_dir / "wordlist_chunk.txt"
        with open(chunk_file, 'w') as f:
            f.write('\n'.join(wordlist))
        
        cmd = [
            "dnsenum", "--enum",
            "--threads", str(self.threads),
            "--timeout", str(self.timeout),
            "-f", str(chunk_file),
            "--xmlout", str(self.output_dir / "dnsenum_results.xml"),
            self.domain
        ]
        
        process = await asyncio.create_subprocess_exec(
            *cmd, stdout=asyncio.subprocess.PIPE, stderr=asyncio.subprocess.PIPE
        )
        stdout, stderr = await process.communicate()
        output = stdout.decode()
        
        for line in output.split('\n'):
            if self.domain in line and line.strip():
                parts = line.split()
                if parts and parts[0].endswith(self.domain):
                    self.discovered_subdomains.add(parts[0])
    
    async def _ct_log_enum(self):
        url = f"https://crt.sh/?q=%.{self.domain}&output=json"
        async with aiohttp.ClientSession() as session:
            try:
                async with session.get(url) as resp:
                    if resp.status == 200:
                        data = await resp.json()
                        for entry in data:
                            name = entry.get('name_value', '')
                            if name:
                                for subdomain in name.split('\n'):
                                    if subdomain.endswith(self.domain):
                                        self.discovered_subdomains.add(subdomain.strip())
            except Exception:
                pass
    
    async def _doh_bruteforce(self, wordlist: List[str]):
        async with aiohttp.ClientSession() as session:
            semaphore = asyncio.Semaphore(50)
            async def query_subdomain(subdomain: str):
                async with semaphore:
                    result = await self._throttled_dns_query(session, subdomain)
                    if result and result.get('Answer'):
                        self.discovered_subdomains.add(subdomain)
                        for answer in result['Answer']:
                            if answer.get('data'):
                                self.discovered_ips.add(answer['data'])
            
            tasks = [query_subdomain(f"{word}.{self.domain}") for word in wordlist]
            await asyncio.gather(*tasks, return_exceptions=True)
    
    async def test_zone_transfers(self):
        print("[*] Testing zone transfers...")
        cmd = ["dig", "+short", "NS", self.domain]
        process = await asyncio.create_subprocess_exec(
            *cmd, stdout=asyncio.subprocess.PIPE, stderr=asyncio.subprocess.PIPE
        )
        stdout, _ = await process.communicate()
        nameservers = stdout.decode().strip().split('\n')
        
        for ns in nameservers:
            ns = ns.strip()
            if not ns:
                continue
            print(f"  Testing {ns}...")
            cmd = ["dnsenum", "--zone", "--rskip", ns, self.domain]
            process = await asyncio.create_subprocess_exec(
                *cmd, stdout=asyncio.subprocess.PIPE, stderr=asyncio.subprocess.PIPE
            )
            stdout, _ = await process.communicate()
            output = stdout.decode()
            self.zone_transfers[ns] = "Zone Transfer successful" in output
    
    def generate_report(self) -> Dict:
        report = {
            "domain": self.domain,
            "timestamp": datetime.now().isoformat(),
            "summary": {
                "total_subdomains": len(self.discovered_subdomains),
                "total_ips": len(self.discovered_ips),
                "zone_transfer_vulnerable": any(self.zone_transfers.values()),
                "vulnerable_nameservers": [ns for ns, vuln in self.zone_transfers.items() if vuln]
            },
            "subdomains": sorted(list(self.discovered_subdomains)),
            "ip_addresses": sorted(list(self.discovered_ips)),
            "zone_transfers": self.zone_transfers
        }
        
        report_file = self.output_dir / f"{self.domain}_report.json"
        with open(report_file, 'w') as f:
            json.dump(report, f, indent=2)
        
        print(f"[+] Report saved to: {report_file}")
        return report
```

### Custom Configuration File

```bash
# Create a scan profile for repeated use
cat > ~/.dnsenum/profiles/aggressive.conf << EOF
--enum
--threads 15
--timeout 5
--retry 5
-f /usr/share/wordlists/subdomains-top1mil-5000.txt
--xmlout
EOF
```

### Scaling Techniques

#### Large-Scale Domain Enumeration
```bash
#!/bin/bash
# scale_dnsenum.sh - Enumerate many domains efficiently

DOMAINS_FILE="domains.txt"
WORDLIST="/usr/share/wordlists/subdomains-top1mil-5000.txt"
OUTPUT_BASE="./scaled_results"
MAX_JOBS=20

mkdir -p "$OUTPUT_BASE"

scan_domain() {
    local domain=$1
    local output_dir="$OUTPUT_BASE/$domain"
    mkdir -p "$output_dir"
    
    dnsenum --enum --threads 5 -f "$WORDLIST" \
        --xmlout "$output_dir/dnsenum.xml" "$domain" > "$output_dir/dnsenum.txt" 2>&1 &
    dnsrecon -d "$domain" -D "$WORDLIST" \
        --xml "$output_dir/dnsrecon.xml" -j "$output_dir/dnsrecon.json" > "$output_dir/dnsrecon.txt" 2>&1 &
    
    wait
    cat "$output_dir"/*.txt | sort -u > "$output_dir/merged.txt"
    echo "[+] Completed: $domain"
}

export -f scan_domain
export WORDLIST OUTPUT_BASE

cat "$DOMAINS_FILE" | xargs -P "$MAX_JOBS" -I {} bash -c 'scan_domain "$@"' _ {}
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: Network Audit - Corporate DNS Security Assessment

**Objective:** Assess DNS security posture for a corporate client.

#### Step 1: Initial DNS Footprint
```bash
# Quick DNS enumeration
dnsenum --threads 5 --timeout 3 megacorp.com

# Document initial findings
echo "=== Initial DNS Scan ===" > megacorp_recon.md
dnsenum megacorp.com >> megacorp_recon.md
```

#### Step 2: Zone Transfer Testing
```bash
# Test zone transfer against all nameservers
dnsenum --zone --noreverse megacorp.com

# If zone transfer succeeds, save full zone data
dnsenum --zone megacorp.com | tee megacorp_zone_transfer.txt
```

**Finding:** Zone transfer successful on `ns2.megacorp.com` - reveals entire DNS namespace including internal hostnames like `jenkins.internal.megacorp.com` and `gitlab.internal.megacorp.com`.

#### Step 3: Subdomain Brute-Forcing
```bash
# Use comprehensive wordlist
dnsenum --enum \
  --threads 10 \
  --timeout 5 \
  -f /usr/share/wordlists/subdomains-top1mil-5000.txt \
  megacorp.com | tee megacorp_subdomains.txt

# Extract discovered subdomains
grep -E '^\s+\S+\.\S+\s' megacorp_subdomains.txt | \
  awk '{print $1}' | sort -u > megacorp_subdomains_clean.txt
```

#### Step 4: Service Discovery on Subdomains
```bash
# Extract IPs and scan with Nmap
cat megacorp_subdomains_clean.txt | \
  dig +short -f - | grep -E '^[0-9]+\.' | sort -u > megacorp_ips.txt

# Quick service scan
nmap -sV -sC -iL megacorp_ips.txt -oA megacorp_services

# Target high-value subdomains
nmap -sV -p 80,443,8080,8443 \
  $(dig +short jenkins.internal.megacorp.com) \
  -oA jenkins_scan
```

### Scenario 2: CTF Challenge - DNS Detective

**Objective:** Find the flag hidden in DNS records.

```bash
# Step 1: Basic enumeration
dnsenum --enum ctf.example.com

# Step 2: Check for zone transfer
dnsenum --zone ctf.example.com

# Output shows zone transfer successful with records:
# flag.ctf.example.com.    3600    IN    TXT    "FLAG{dns_zone_transfer_is_dangerous}"
# hidden.ctf.example.com.  3600    IN    A      10.10.10.50

# Step 3: Extract flag from TXT record
dig TXT flag.ctf.example.com

# Step 4: Check hidden subdomain
curl http://hidden.ctf.example.com/
```

### Scenario 3: Bug Bounty - Subdomain Discovery

**Objective:** Find hidden subdomains for bug bounty program.

#### Phase 1: Passive Enumeration
```bash
dnsenum --google-exchange 100 --bing-exchange 100 program.example.com | \
  tee bugbounty_passive.txt
```

#### Phase 2: Active Brute-Force
```bash
for wordlist in \
  /usr/share/wordlists/subdomains-top1mil-5000.txt \
  /usr/share/wordlists/dnsenum.txt; do
    dnsenum --enum -f "$wordlist" program.example.com
done | sort -u > bugbounty_active.txt
```

#### Phase 3: Certificate Transparency
```bash
curl -s "https://crt.sh/?q=%.program.example.com&output=json" | \
  jq -r '.[].name_value' | sort -u > ct_subdomains.txt

cat bugbounty_active.txt ct_subdomains.txt | sort -u > all_subdomains.txt
```

#### Phase 4: Validation
```bash
while IFS= read -r subdomain; do
    ip=$(dig +short "$subdomain" | head -1)
    if [ -n "$ip" ]; then
        echo "[+] $subdomain -> $ip"
        echo "$subdomain,$ip" >> validated_subdomains.csv
    fi
done < all_subdomains.txt
```

---

## Chapter 7: Detection & Defense

### How Defenders Detect DNSenum

#### DNS Query Pattern Analysis

dnsenum generates distinctive DNS query patterns:

- **High query rate** from single source IP
- **Sequential subdomain queries** (admin., www., mail., ftp., etc.)
- **AXFR queries** for zone transfer attempts
- **Consistent timing patterns** between queries

#### Zeek/Suricata Detection Rules

```bash
# Suricata rule for dnsenum detection
alert dns any any -> any any (msg:"DNS Subdomain Brute Force Detected"; \
    dns.query; threshold:type both, track by_src, count 100, seconds 60; \
    classtype:attempted-recon; sid:1000001; rev:1;)

# Detect AXFR zone transfer attempts
alert dns any any -> any any (msg:"DNS Zone Transfer AXFR Attempt"; \
    dns.opcode: 0; content:"|00 FC|"; offset:2; depth:2; \
    classtype:attempted-recon; sid:1000002; rev:1;)

# Detect sequential subdomain queries
alert dns any any -> any any (msg:"DNS Sequential Subdomain Enumeration"; \
    dns.query; pcre:"/^(admin|www|mail|ftp|smtp|ns1|ns2|dev|staging|test)\./i"; \
    threshold:type both, track by_src, count 50, seconds 30; \
    classtype:attempted-recon; sid:1000003; rev:1;)
```

#### Log Analysis for Detection
```bash
#!/bin/bash
# dnsenum_detection.sh

DNS_LOG="/var/log/bind/queries.log"
ALERT_THRESHOLD=100

# Count queries per source IP
echo "=== DNS Query Analysis ==="
echo "Queries per source IP:"
awk '{print $1}' "$DNS_LOG" | sort | uniq -c | sort -rn | head -20

# Detect potential brute-force
echo ""
echo "=== Potential Brute-Force Activity ==="
awk '{print $1}' "$DNS_LOG" | sort | uniq -c | \
    awk -v threshold="$ALERT_THRESHOLD" '$1 > threshold {print "[!] Suspicious: "$2" ("$1" queries)"}'

# Check for AXFR attempts
echo ""
echo "=== AXFR Zone Transfer Attempts ==="
grep -i "axfr" "$DNS_LOG" | tail -20
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
    
    // Enable DNSSEC
    dnssec-validation auto;
    
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

# Block AXFR from external sources
iptables -A INPUT -p udp --dport 53 -m string --string "AXFR" \
    --algo bm -j DROP
iptables -A INPUT -p tcp --dport 53 -m string --string "AXFR" \
    --algo bm -j DROP
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

### Common Errors and Solutions

#### Error: "Can't locate Net/DNS.pm in @INC"

**Cause:** Missing Perl Net::DNS module.

**Solution:**
```bash
# Debian/Ubuntu
sudo apt install libnet-dns-perl

# CentOS/RHEL
sudo yum install perl-Net-DNS

# Verify installation
perl -e 'use Net::DNS; print "OK\n"'
```

#### Error: "Can't locate Net/IP.pm"

**Cause:** Missing Perl Net::IP module.

**Solution:**
```bash
sudo apt install libnet-ip-perl
# or
cpanm Net::IP
```

#### Error: "Can't locate Net/Netmask.pm"

**Cause:** Missing Perl Net::Netmask module.

**Solution:**
```bash
sudo apt install libnet-netmask-perl
```

#### Error: "dnsenum: command not found"

**Cause:** dnsenum not installed or not in PATH.

**Solution:**
```bash
# Check if installed
which dnsenum

# If installed but not in PATH
export PATH=$PATH:/opt/dnsenum
echo 'export PATH=$PATH:/opt/dnsenum' >> ~/.bashrc

# Create symlink
sudo ln -s /opt/dnsenum/dnsenum.pl /usr/local/bin/dnsenum
```

#### Error: "Zone Transfer failed" (Unexpected)

**Cause:** Zone transfer legitimately blocked or network issue.

**Solutions:**
```bash
# Test connectivity to nameserver
dig @ns1.example.com example.com AXFR

# Increase timeout
dnsenum --timeout 10 --retry 5 example.com

# Try different nameserver
dnsenum --rskip ns1.example.com example.com
```

#### Error: Timeout During Brute-Force

**Cause:** Too many queries overwhelming network or DNS server.

**Solutions:**
```bash
# Reduce threads
dnsenum --threads 2 --timeout 10 example.com

# Use smaller wordlist
dnsenum -f small_wordlist.txt example.com
```

#### Error: "XML::Writer not found"

**Solution:**
```bash
sudo apt install libxml-writer-perl
```

### Performance Optimization

#### Slow Enumeration
```bash
# Use faster DNS server
dnsenum --dns-servers 8.8.8.8,8.8.4.4 example.com

# Reduce timeout
dnsenum --timeout 3 example.com

# Increase threads (if network allows)
dnsenum --threads 20 example.com
```

#### High Memory Usage
```bash
# Limit subdomain depth
dnsenum --limit 1000 example.com

# Avoid recursive mode
dnsenum --recursion no example.com
```

### FAQ

**Q: Does dnsenum work with DNS over HTTPS (DoH)?**
A: Not directly. Set up a local DoH proxy (e.g., dnscrypt-proxy) and point dnsenum to it: `dnsenum --dns-servers 127.0.0.1 example.com`

**Q: Can dnsenum enumerate IPv6 addresses?**
A: Yes, dnsenum fetches AAAA records by default.

**Q: How do I use dnsenum with Tor?**
```bash
sudo systemctl start tor
proxychains dnsenum example.com
```

**Q: Is dnsenum legal to use?**
A: dnsenum itself is legal. Using it against systems without authorization is illegal. Always get written permission.

**Q: What's the difference between dnsenum and dnsrecon?**
A: dnsenum focuses on zone transfers and brute-force with search engine integration. dnsrecon is more comprehensive with multiple enumeration methods and richer output formats.

---

## Resources

### Official Documentation
- [dnsenum GitHub Repository](https://github.com/fwaeyt/dnsenum)
- [Perl Net::DNS Documentation](https://metacpan.org/pod/Net::DNS)

### Wordlists
- [SecLists DNS Subdomains](https://github.com/danielmiessler/SecLists/tree/master/Discovery/DNS)
- [dnsenum Default Wordlist](https://github.com/fwaeyt/dnsenum/blob/master/dnsenum.txt)

### Related Tools
- **dnsrecon:** More comprehensive DNS reconnaissance
- **dnsmap:** Fast subdomain brute-forcing
- **massdns:** High-speed bulk DNS resolution
- **sublist3r:** Subdomain enumeration via multiple sources

### Learning Resources
- [DNS Enumeration Techniques](https://www.offensive-security.com/metasploit-unleashed/dns-enumeration/)
- [Zone Transfer Testing](https://book.hacktricks.xyz/network-services-pentesting/pentesting-dns)
- [Bug Bounty DNS Recon](https://github.com/arkadiyt/bounty-targets-data)
