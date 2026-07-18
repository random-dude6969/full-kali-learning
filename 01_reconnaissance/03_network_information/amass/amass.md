---
tool_name: "amass"
display_name: "Amass"
mitre_tactic: "reconnaissance"
mitre_tactic_id: "TA0043"
mitre_subcategory: "network_information"
install_command: "sudo apt install amass"
version: "4.2.1"
github_repo: "https://github.com/owasp-amass/amass"
kali_tools_page: "https://www.kali.org/tools/amass/"
difficulty_level: "intermediate-to-advanced"
requires_root: false
gui_available: false
tags: ["dns", "reconnaissance", "osint", "subdomain-enumeration", "attack-surface", "network-mapping"]
related_tools: ["subfinder", "recon-ng", "theHarvester", "dnsrecon", "sublist3r"]
---

# Amass

## Chapter 1: Introduction & Overview

### What is Amass?

Amass is a powerful open-source reconnaissance tool developed under the OWASP Foundation. It specializes in network mapping and attack surface discovery through DNS enumeration, OSINT data collection, and infrastructure analysis. The tool creates comprehensive maps of target environments by combining passive intelligence gathering with active probing techniques.

### History & Background

- **Created:** OWASP Foundation (Open Web Application Security Project)
- **Maintained:** OWASP Amass Project
- **License:** Apache 2.0
- **Language:** Go
- **Purpose:** DNS enumeration, subdomain discovery, network mapping, attack surface management

Amass is one of the most comprehensive reconnaissance tools available, aggregating data from over 50 passive sources and performing active DNS enumeration to build detailed maps of target infrastructure.

### Why Use Amass?

#### Comprehensive Asset Discovery
Traditional reconnaissance often misses shadow IT assets, forgotten subdomains, and legacy infrastructure. Amass systematically discovers these hidden assets through multiple enumeration techniques:

- **Passive sources:** Certificate transparency logs, WHOIS databases, web archives
- **Active enumeration:** DNS brute-forcing, zone transfer attempts, reverse DNS
- **API integrations:** Shodan, Censys, SecurityTrails, VirusTotal, and more

```bash
# Single domain
amass enum -d example.com

# Multiple domains from file
amass enum -dL domains.txt

# Organization-wide enumeration
amass intel -org "Target Company"
```

#### Scalability
Amass efficiently handles large-scale reconnaissance engagements through parallel processing, distributed execution, and efficient database management.

#### Relationship Mapping
Unlike simple enumeration tools, Amass builds relationship graphs that reveal:

- How domains connect through shared infrastructure
- Which ASN ranges the target controls
- Certificate chains and SSL/TLS relationships
- Historical changes in DNS records

### When to Use This Tool

- **Penetration Testing:** Initial reconnaissance, scope expansion, dependency mapping
- **Bug Bounty Hunting:** Asset discovery, hidden services, subdomain takeover identification
- **Security Assessments:** Attack surface management, shadow IT detection, M&A due diligence
- **Incident Response:** Threat hunting, scope definition, recovery validation

### When NOT to Use This Tool

- Without explicit written authorization from the network owner
- Against production networks without proper planning
- For malicious purposes or unauthorized access
- When a simple DNS lookup would suffice

### Legal and Ethical Considerations

- **Always obtain written permission** before scanning networks you don't own
- **Understand your scope** - know exactly what you're authorized to enumerate
- **Be careful with timing** - aggressive enumeration can cause disruptions
- **Document everything** - keep records of your authorization and activities
- **Follow responsible disclosure** - report vulnerabilities properly

### Key Concepts

- **Subdomain:** A domain that is part of a larger domain (e.g., mail.example.com)
- **DNS Enumeration:** The process of discovering DNS records and subdomains
- **Passive Reconnaissance:** Gathering information without directly querying the target
- **Active Reconnaissance:** Directly querying target DNS servers and infrastructure
- **Certificate Transparency (CT):** Public logs of SSL/TLS certificate issuance
- **ASN (Autonomous System Number):** Unique identifier for a network operator
- **Zone Transfer (AXFR):** DNS replication protocol that can reveal all records

---

## Chapter 2: Installation & Setup

### System Requirements

- **Operating System:** Linux (recommended), macOS, Windows
- **Memory:** 4GB+ RAM recommended for large-scale enumeration
- **Storage:** 1GB+ free space for database and output files
- **Network:** Reliable internet connection for API queries
- **Go:** 1.18 or later (for building from source)

### Installation Methods

#### Method 1: Go Install (Recommended)

```bash
# Install Go first (if not installed)
# Visit https://go.dev/doc/install for your platform

# Install Amass
go install github.com/owasp-amass/amass/v4/...@master

# Verify installation
amass -version
```

#### Method 2: Binary Release

```bash
# Check your architecture
uname -m

# Download latest release (check https://github.com/owasp-amass/amass/releases)
# Linux AMD64
wget https://github.com/owasp-amass/amass/releases/latest/download/amass_Linux_amd64.zip

# Extract
unzip amass_Linux_amd64.zip

# Move to PATH
sudo mv amass_Linux_amd64/amass /usr/local/bin/

# Verify
amass -version
```

#### Method 3: Package Managers

```bash
# Debian/Ubuntu (may be outdated)
sudo apt update
sudo apt install amass

# Homebrew (macOS)
brew install amass

# Arch Linux
sudo pacman -S amass

# Snap
sudo snap install amass
```

#### Method 4: Build from Source

```bash
# Clone the repository
git clone https://github.com/owasp-amass/amass.git

# Navigate to directory
cd amass

# Build
go build -o amass ./cmd/amass/

# Install
sudo cp amass /usr/local/bin/
```

### API Configuration

#### Setting Up API Keys

Create configuration file:

```bash
# Create config directory
mkdir -p ~/.config/amass

# Create and edit config
nano ~/.config/amass/config.ini
```

#### Config File Format

```ini
[api_keys]
# Shodan - https://shodan.io
shodan = YOUR_SHODAN_API_KEY

# Censys - https://censys.io
censys_username = YOUR_EMAIL
censys_api_key = YOUR_CENSYS_KEY

# SecurityTrails - https://securitytrails.com
securitytrails = YOUR_SECURITYTRAILS_KEY

# VirusTotal - https://virustotal.com
virustotal = YOUR_VIRUSTOTAL_KEY

# GitHub - https://github.com/settings/tokens
github = YOUR_GITHUB_TOKEN

# BinaryEdge - https://binaryedge.io
binaryedge = YOUR_BINARYEDGE_KEY

# FullHunt - https://fullhunt.io
fullhunt = YOUR_FULLHUNT_KEY

# HackedDB - https://hackeddb.org
hackeddb = YOUR_HACKEDDB_KEY

# IntelX - https://intelx.io
intelx = YOUR_INTELX_KEY
```

#### Environment Variables Alternative

```bash
# Add to ~/.bashrc or ~/.zshrc
export AMASS_SHODAN_API_KEY="your_key_here"
export AMASS_CENSYS_API_KEY="your_key_here"
export AMASS_CENSYS_API_SECRET="your_secret_here"
export AMASS_SECURITYTRAILS_API_KEY="your_key_here"
export AMASS_VIRUSTOTAL_API_KEY="your_key_here"
export AMASS_GITHUB_API_KEY="your_token_here"

# Reload shell
source ~/.bashrc
```

### Directory Structure

```
~/.config/amass/
├── config.ini              # API keys and configuration
└── graph/                  # Database storage directory
    └── amass.db           # SQLite database (created automatically)

~/amass/                    # Default output directory
├── output.txt             # Enumeration results
└── *.json                 # JSON output files
```

### Database Management

```bash
# Create fresh database
amass db -init

# Specify custom database location
amass db -d /path/to/custom/db -init

# List domains in database
amass db -show

# Show specific domain
amass db -show -d example.com

# Export data
amass db -show -d example.com -o output.json -format json

# Remove and recreate
rm -rf ~/.config/amass/graph/
amass db -init
```

### Network Configuration

#### Proxy Support

```bash
# Use SOCKS5 proxy
amass enum -d example.com -proxy socks5://127.0.0.1:9050

# Use HTTP proxy
amass enum -d example.com -proxy http://proxy.example.com:8080

# With authentication
amass enum -d example.com -proxy user:pass@proxy.example.com:8080
```

#### DNS Configuration

```bash
# Use custom DNS resolvers
amass enum -d example.com -r 8.8.8.8,8.8.4.4

# Use resolver file
amass enum -d example.com -rf resolvers.txt
```

Create resolver file:

```bash
# resolvers.txt
8.8.8.8
8.8.4.4
1.1.1.1
1.0.0.1
9.9.9.9
208.67.222.222
```

### Verification

```bash
# Check version
amass -version

# Verify configuration
amass enum -passive -d owasp.org -o /dev/stdout | head

# Test API keys
amass intel -org "OWASP" -d /dev/stdout | head
```

### Health Check Script

```bash
#!/bin/bash
# amass-health.sh - Verify Amass installation

echo "=== Amass Health Check ==="

# Check binary
if command -v amass &> /dev/null; then
    echo "[✓] Amass installed: $(amass -version)"
else
    echo "[✗] Amass not found in PATH"
    exit 1
fi

# Check config directory
if [ -d "$HOME/.config/amass" ]; then
    echo "[✓] Config directory exists"
else
    echo "[!] Config directory missing - run: mkdir -p ~/.config/amass"
fi

# Check config file
if [ -f "$HOME/.config/amass/config.ini" ]; then
    KEY_COUNT=$(grep -c "=" "$HOME/.config/amass/config.ini")
    echo "[✓] Config file found ($KEY_COUNT keys configured)"
else
    echo "[!] No config file found"
fi

# Test basic enumeration
echo ""
echo "Testing passive enumeration..."
if timeout 10 amass enum -passive -d example.com -o /dev/null 2>/dev/null; then
    echo "[✓] Passive enumeration working"
else
    echo "[!] Passive enumeration may have issues"
fi
```

---

## Chapter 3: Basic Usage

### First Run

```bash
# Basic passive enumeration
amass enum -passive -d example.com

# Basic active enumeration
amass enum -d example.com

# Save results to file
amass enum -d example.com -o results.txt
```

### Command Structure

```
amass [subcommand] [options] -d <domain>
```

### Core Subcommands

#### amass intel - Passive Reconnaissance

Gather intelligence without directly querying the target:

```bash
# Find domains belonging to an organization
amass intel -org "Example Corp"

# Discover infrastructure by ASN
amass intel -asn 13335

# Find domains by IP range
amass intel -addr 192.168.1.0/24

# Reverse WHOIS lookup
amass intel -whois -d example.com

# From a list of domains
amass intel -dL domains.txt
```

#### amass enum - Active Enumeration

Actively enumerate subdomains and infrastructure:

```bash
# Passive only (no direct queries)
amass enum -passive -d example.com

# Active enumeration with all sources
amass enum -d example.com

# Specify data sources
amass enum -src -d example.com

# Verbose output
amass enum -v -d example.com
```

#### amass db - Database Operations

Manage the Amass graph database:

```bash
# Show all collected data
amass db -show

# Filter by domain
amass db -show -d example.com

# Export to JSON
amass db -show -d example.com -o data.json -format json

# Export to GraphML
amass db -show -d example.com -o graph.graphml -format graphml

# Initialize fresh database
amass db -init
```

#### amass viz - Visualization

Generate visualizations of collected data:

```bash
# Generate D3.js visualization
amass viz -d3 -d example.com -o visualization.html

# Generate Gephi graph
amass viz -gephi -d example.com -o network.gexf

# Generate GraphML
amass viz -graphml -d example.com -o graph.graphml
```

#### amass track - Change Detection

Monitor changes in target's attack surface:

```bash
# Track changes over time
amass track -d example.com

# Compare against previous results
amass track -d example.com -last 30

# Track with specific output
amass track -d example.com -o changes.txt
```

### Complete Flags Reference

#### Target Specification

| Flag | Description | Example |
|------|-------------|---------|
| `-d` | Target domain | `-d example.com` |
| `-dL` | File with domains | `-dL domains.txt` |
| `-org` | Organization name | `-org "Corp Name"` |
| `-asn` | ASN number | `-asn 13335` |
| `-cidr` | CIDR range | `-cidr 192.168.0.0/16` |
| `-addr` | IP address/CIDR | `-addr 10.0.0.0/8` |

#### Enumeration Options

| Flag | Description | Example |
|------|-------------|---------|
| `-passive` | Passive only | `-passive` |
| `-active` | Active enumeration | `-active` |
| `-src` | Show data sources | `-src` |
| `-v` | Verbose output | `-v` |
| `-timeout` | Query timeout (sec) | `-timeout 30` |
| `-max-dns-queries` | Rate limit | `-max-dns-queries 100` |

#### Output Options

| Flag | Description | Example |
|------|-------------|---------|
| `-o` | Output file | `-o results.txt` |
| `-format` | Output format | `-format json` |
| `-nolocal` | Exclude local IPs | `-nolocal` |
| `-noa` | Exclude ASN data | `-noa` |

#### Data Source Options

| Flag | Description | Example |
|------|-------------|---------|
| `-src` | Show which source found it | `-src` |
| `-include` | Specific sources | `-include shodan` |
| `-exclude` | Skip sources | `-exclude archiveit` |
| `-list` | List all sources | `-list` |

### Practical Examples

#### Example 1: Basic Subdomain Enumeration

```bash
# Passive enumeration (fast, no direct queries)
amass enum -passive -d company.com -o passive_results.txt

# Review results
wc -l passive_results.txt
cat passive_results.txt
```

#### Example 2: Comprehensive Enumeration

```bash
# Full enumeration with all sources
amass enum -d company.com \
    -src \
    -v \
    -o comprehensive_results.txt \
    -format json

# Time the operation
time amass enum -d company.com -o /dev/null
```

#### Example 3: Organization-Wide Discovery

```bash
# Find all assets for an organization
amass intel -org "Target Organization" -d org_assets.txt

# Enumerate discovered domains
while read domain; do
    amass enum -passive -d "$domain" -o "${domain}_subs.txt"
done < org_assets.txt
```

#### Example 4: Multiple Domains

```bash
# From file
cat domains.txt
# company.com
# subsidiary.com
# partner.org

amass enum -dL domains.txt -o all_results.txt

# Pipe from stdin
echo -e "company.com\nsubsidiary.com" | amass enum -dL /dev/stdin
```

### Output Formats

#### Text Format (Default)

```bash
# Simple list of subdomains
amass enum -d company.com -o results.txt

# Output example:
# mail.company.com
# web.company.com
# api.company.com
# vpn.company.com
```

#### JSON Format

```bash
# Detailed JSON output
amass enum -d company.com -o results.json -format json

# JSON structure:
# {
#   "name": "subdomain.company.com",
#   "domain": "company.com",
#   "addresses": [{"ip": "1.2.3.4", "asn": 12345}],
#   "sources": ["shodan", "censys"],
#   "tag": "subdomain"
# }
```

#### GraphML Format

```bash
# For network visualization tools
amass enum -d company.com -o graph.graphml -format graphml
```

#### CSV Format

```bash
# Convert JSON to CSV
amass enum -d company.com -o results.json -format json
jq -r '.[] | [.name, .domain, (.addresses[].ip // ""), (.sources | join("|"))] | @csv' results.json > results.csv
```

### Processing Results

#### Quick Analysis

```bash
# Count discovered subdomains
amass enum -d company.com | wc -l

# Find unique IP addresses
amass enum -d company.com -o results.json -format json
jq -r '.[].addresses[].ip' results.json | sort -u

# Find most common sources
amass enum -d company.com -o results.json -format json
jq -r '.[].sources[]' results.json | sort | uniq -c | sort -rn
```

#### Pipeline with Other Tools

```bash
# Feed to httpx for HTTP probing
amass enum -d company.com | httpx -o live.txt

# Feed to nmap for port scanning
amass enum -d company.com | nmap -iL - -oA amass_scan

# Feed to subjack for subdomain takeover
amass enum -d company.com -o subs.txt
subjack -w subs.txt -t 100 -timeout 30 -ssl -c fingerprints.json
```

---

## Chapter 4: Intermediate Usage

### Automation with Bash

#### Automated Recon Script

```bash
#!/bin/bash
# amass-auto-recon.sh - Automated reconnaissance workflow

TARGET=$1
OUTPUT_DIR="recon_$(date +%Y%m%d_%H%M%S)"

if [ -z "$TARGET" ]; then
    echo "Usage: $0 <domain>"
    exit 1
fi

mkdir -p "$OUTPUT_DIR"
echo "[*] Starting reconnaissance against $TARGET"
echo "[*] Output directory: $OUTPUT_DIR"

# Phase 1: Passive Enumeration
echo "[+] Phase 1: Passive enumeration..."
amass enum -passive -d "$TARGET" \
    -o "$OUTPUT_DIR/passive_subs.txt" \
    -timeout 300

PASSIVE_COUNT=$(wc -l < "$OUTPUT_DIR/passive_subs.txt")
echo "[+] Found $PASSIVE_COUNT subdomains passively"

# Phase 2: Active Enumeration
echo "[+] Phase 2: Active enumeration..."
amass enum -d "$TARGET" \
    -src \
    -o "$OUTPUT_DIR/active_subs.txt" \
    -timeout 600

ACTIVE_COUNT=$(wc -l < "$OUTPUT_DIR/active_subs.txt")
echo "[+] Found $ACTIVE_COUNT subdomains actively"

# Phase 3: Merge and deduplicate
echo "[+] Phase 3: Merging results..."
cat "$OUTPUT_DIR/passive_subs.txt" "$OUTPUT_DIR/active_subs.txt" | \
    sort -u > "$OUTPUT_DIR/all_subs.txt"

TOTAL_COUNT=$(wc -l < "$OUTPUT_DIR/all_subs.txt")
echo "[+] Total unique subdomains: $TOTAL_COUNT"

# Phase 4: Quick probe for live hosts
echo "[+] Phase 4: Probing for live hosts..."
cat "$OUTPUT_DIR/all_subs.txt" | \
    httpx -silent -o "$OUTPUT_DIR/live_hosts.txt"

LIVE_COUNT=$(wc -l < "$OUTPUT_DIR/live_hosts.txt")
echo "[+] Live hosts found: $LIVE_COUNT"

# Phase 5: Port scan live hosts
echo "[+] Phase 5: Port scanning..."
nmap -iL "$OUTPUT_DIR/live_hosts.txt" \
    -T4 -Pn -sV \
    -oA "$OUTPUT_DIR/nmap_scan" \
    --max-retries 2

echo "[+] Reconnaissance complete!"
echo "[*] Results saved to: $OUTPUT_DIR/"
ls -la "$OUTPUT_DIR/"
```

#### Multi-Target Automation

```bash
#!/bin/bash
# amass-multi-target.sh - Enumerate multiple organizations

TARGETS_FILE=$1
RESULTS_DIR="multi_recon_$(date +%Y%m%d)"

if [ -z "$TARGETS_FILE" ]; then
    echo "Usage: $0 <targets.txt>"
    echo "Format: domain.com"
    exit 1
fi

mkdir -p "$RESULTS_DIR"

while IFS= read -r domain; do
    echo ""
    echo "========================================="
    echo "Enumerating: $domain"
    echo "========================================="
    
    # Create domain-specific directory
    DOMAIN_DIR="$RESULTS_DIR/$domain"
    mkdir -p "$DOMAIN_DIR"
    
    # Run amass with timeout
    timeout 3600 amass enum -d "$domain" \
        -src \
        -o "$DOMAIN_DIR/subdomains.txt" \
        2>&1 | tee "$DOMAIN_DIR/log.txt"
    
    # Quick statistics
    COUNT=$(wc -l < "$DOMAIN_DIR/subdomains.txt")
    echo "[+] $domain: Found $COUNT subdomains"
    
    # Sleep between targets to avoid rate limiting
    sleep 30
    
done < "$TARGETS_FILE"

echo "[+] All targets processed!"
```

### Automation with Python

#### Python Amass Wrapper

```python
#!/usr/bin/env python3
"""
amass_wrapper.py - Python wrapper for Amass automation
"""

import subprocess
import json
import os
import time
from datetime import datetime
from pathlib import Path
from typing import List, Dict, Optional
import argparse


class AmassScanner:
    """Wrapper class for Amass reconnaissance"""
    
    def __init__(self, output_dir: str = "amass_results"):
        self.output_dir = Path(output_dir)
        self.output_dir.mkdir(parents=True, exist_ok=True)
        self.results = {}
        
    def passive_enum(self, domain: str) -> List[str]:
        """Run passive enumeration"""
        print(f"[*] Running passive enumeration for {domain}")
        
        output_file = self.output_dir / f"{domain}_passive.txt"
        
        cmd = [
            "amass", "enum",
            "-passive",
            "-d", domain,
            "-o", str(output_file)
        ]
        
        try:
            subprocess.run(cmd, check=True, capture_output=True, text=True)
            with open(output_file, 'r') as f:
                subdomains = [line.strip() for line in f if line.strip()]
            
            print(f"[+] Found {len(subdomains)} subdomains (passive)")
            return subdomains
            
        except subprocess.CalledProcessError as e:
            print(f"[!] Error: {e.stderr}")
            return []
    
    def active_enum(self, domain: str, timeout: int = 600) -> List[str]:
        """Run active enumeration with sources"""
        print(f"[*] Running active enumeration for {domain}")
        
        output_file = self.output_dir / f"{domain}_active.json"
        
        cmd = [
            "amass", "enum",
            "-d", domain,
            "-src",
            "-o", str(output_file),
            "-timeout", str(timeout)
        ]
        
        try:
            subprocess.run(cmd, check=True, capture_output=True, text=True)
            
            # Parse JSON output
            with open(output_file, 'r') as f:
                data = json.load(f)
            
            subdomains = [item['name'] for item in data if 'name' in item]
            print(f"[+] Found {len(subdomains)} subdomains (active)")
            
            return subdomains
            
        except (subprocess.CalledProcessError, json.JSONDecodeError) as e:
            print(f"[!] Error: {e}")
            return []
    
    def comprehensive_scan(self, domain: str) -> Dict:
        """Run comprehensive scan combining passive and active"""
        print(f"[*] Starting comprehensive scan for {domain}")
        
        start_time = time.time()
        
        # Phase 1: Passive
        passive = self.passive_enum(domain)
        
        # Phase 2: Active
        active = self.active_enum(domain)
        
        # Merge and deduplicate
        all_subs = list(set(passive + active))
        
        elapsed = time.time() - start_time
        
        results = {
            "domain": domain,
            "timestamp": datetime.now().isoformat(),
            "passive_count": len(passive),
            "active_count": len(active),
            "total_unique": len(all_subs),
            "elapsed_seconds": round(elapsed, 2),
            "subdomains": sorted(all_subs)
        }
        
        # Save results
        output_file = self.output_dir / f"{domain}_comprehensive.json"
        with open(output_file, 'w') as f:
            json.dump(results, f, indent=2)
        
        print(f"[+] Comprehensive scan complete: {len(all_subs)} unique subdomains")
        return results
    
    def export_for_tool(self, domain: str, tool: str = "httpx") -> str:
        """Export results formatted for other tools"""
        results_file = self.output_dir / f"{domain}_comprehensive.json"
        
        if not results_file.exists():
            print("[!] No results found. Run scan first.")
            return ""
        
        with open(results_file, 'r') as f:
            data = json.load(f)
        
        output_file = self.output_dir / f"{domain}_{tool}_input.txt"
        
        if tool == "httpx":
            with open(output_file, 'w') as f:
                for sub in data['subdomains']:
                    f.write(f"{sub}\n")
        elif tool == "nuclei":
            with open(output_file, 'w') as f:
                for sub in data['subdomains']:
                    f.write(f"https://{sub}\n")
        
        print(f"[+] Exported {len(data['subdomains'])} entries for {tool}")
        return str(output_file)


def main():
    parser = argparse.ArgumentParser(description="Amass Automation Wrapper")
    parser.add_argument("-d", "--domain", help="Target domain")
    parser.add_argument("-o", "--org", help="Target organization")
    parser.add_argument("-t", "--type", 
                       choices=["passive", "active", "comprehensive"],
                       default="comprehensive",
                       help="Scan type")
    parser.add_argument("-out", "--output", default="amass_results",
                       help="Output directory")
    
    args = parser.parse_args()
    
    scanner = AmassScanner(args.output)
    
    if args.org:
        domains = scanner.organization_intel(args.org)
        for domain in domains:
            scanner.comprehensive_scan(domain)
    elif args.domain:
        if args.type == "passive":
            scanner.passive_enum(args.domain)
        elif args.type == "active":
            scanner.active_enum(args.domain)
        else:
            scanner.comprehensive_scan(args.domain)
    else:
        parser.print_help()


if __name__ == "__main__":
    main()
```

### Integration with Other Tools

#### Amass + httpx + nuclei Pipeline

```bash
#!/bin/bash
# amass-pipeline.sh - Complete recon pipeline

DOMAIN=$1
OUTPUT="pipeline_$(date +%Y%m%d)"

mkdir -p "$OUTPUT"

echo "[*] Starting pipeline for $DOMAIN"

# Phase 1: Amass enumeration
echo "[+] Phase 1: Amass enumeration..."
amass enum -d "$DOMAIN" -src -o "$OUTPUT/amass.json" -format json

# Parse amass output
jq -r '.[].name' "$OUTPUT/amass.json" | sort -u > "$OUTPUT/subdomains.txt"

# Phase 2: HTTP probing
echo "[+] Phase 2: HTTP probing..."
cat "$OUTPUT/subdomains.txt" | httpx -silent -status-code -o "$OUTPUT/live.txt"

# Phase 3: Extract live hosts
awk '{print $1}' "$OUTPUT/live.txt" > "$OUTPUT/hosts.txt"

# Phase 4: Port scanning
echo "[+] Phase 4: Port scanning..."
nmap -iL "$OUTPUT/hosts.txt" -T4 -Pn -sV -oA "$OUTPUT/nmap"

# Phase 5: Vulnerability scanning
echo "[+] Phase 5: Nuclei scanning..."
nuclei -l "$OUTPUT/hosts.txt" -o "$OUTPUT/vulns.txt"

echo "[+] Pipeline complete!"
```

### Performance Optimization

```bash
# Use multiple resolvers for faster enumeration
amass enum -d company.com \
    -rf resolvers.txt \
    -max-dns-queries 200 \
    -timeout 30

# Parallel execution with GNU parallel
cat domains.txt | parallel -j 4 \
    amass enum -passive -d {} -o {.}_subs.txt
```

### Data Enrichment

```bash
# Enrich with Shodan data
amass enum -d company.com -o amass.json -format json
jq -r '.[].name' amass.json | while read host; do
    ip=$(dig +short "$host" | head -1)
    if [ -n "$ip" ]; then
        shodan host "$ip" >> shodan_data.txt
    fi
done
```

---

## Chapter 5: Advanced Usage

### Evasion Techniques

#### Rate Limiting and Timing

```bash
# Slow enumeration to avoid detection
amass enum -d company.com \
    -max-dns-queries 50 \
    -timeout 30 \
    -passive

# Use multiple resolvers to distribute queries
amass enum -d company.com \
    -rf resolvers_large.txt \
    -max-dns-queries 100

# Random delays between queries
amass enum -d company.com \
    -max-dns-queries 30 \
    -timeout 45
```

#### Proxy Chains

```bash
# Route through Tor
amass enum -d company.com \
    -proxy socks5://127.0.0.1:9050

# Use multiple proxies
for proxy in $(cat proxies.txt); do
    amass enum -d company.com \
        -proxy "$proxy" \
        -o "output_${proxy//[:\/]/_}.txt" &
done
wait

# Rotate through proxy list
cat proxies.txt | while read proxy; do
    amass enum -passive -d company.com \
        -proxy "$proxy" \
        -o /dev/null
    sleep 60
done
```

#### Source Manipulation

```bash
# Use only passive sources (minimal footprint)
amass enum -passive -d company.com \
    -exclude archiveit,commoncrawl

# Select specific trusted sources
amass enum -d company.com \
    -include shodan,censys,virustotal

# Exclude noisy sources
amass enum -d company.com \
    -exclude baidu,yahoo,bing
```

### Scaling for Large Engagements

#### Enterprise-Scale Enumeration

```bash
#!/bin/bash
# enterprise_amass.sh - Scale amass for large organizations

ORG=$1
PARALLEL=10

echo "[*] Starting enterprise enumeration for: $ORG"

# Phase 1: Organization discovery
echo "[+] Discovering organization domains..."
amass intel -org "$ORG" -o org_domains.txt

# Phase 2: Parallel domain enumeration
echo "[+] Enumerating domains in parallel ($PARALLEL workers)..."
cat org_domains.txt | parallel -j $PARALLEL \
    amass enum -passive -d {} -o {.}_subs.txt

# Phase 3: Merge results
echo "[+] Merging results..."
cat *_subs.txt | sort -u > all_subdomains.txt

# Phase 4: Deduplication and cleanup
echo "[+] Cleaning results..."
awk '!seen[$0]++' all_subdomains.txt > final_subdomains.txt

TOTAL=$(wc -l < final_subdomains.txt)
echo "[+] Total unique subdomains: $TOTAL"
```

### Custom Data Collection

```python
#!/usr/bin/env python3
"""
amass_custom.py - Extended amass with custom collection
"""

import subprocess
import requests
import json
from typing import List, Set
import dns.resolver


class CustomAmass:
    """Extended enumeration beyond standard amass"""
    
    def __init__(self, domain: str):
        self.domain = domain
        self.discovered: Set[str] = set()
    
    def run_standard_amass(self) -> List[str]:
        """Run standard amass enumeration"""
        cmd = [
            "amass", "enum",
            "-d", self.domain,
            "-passive",
            "-o", "/dev/stdout"
        ]
        
        result = subprocess.run(cmd, capture_output=True, text=True)
        subdomains = [s.strip() for s in result.stdout.split('\n') if s.strip()]
        self.discovered.update(subdomains)
        
        return subdomains
    
    def check_certificate_transparency(self) -> List[str]:
        """Query certificate transparency logs"""
        url = f"https://crt.sh/?q=%.{self.domain}&output=json"
        
        try:
            response = requests.get(url, timeout=30)
            data = response.json()
            
            subdomains = set()
            for entry in data:
                name = entry.get('name_value', '')
                for sub in name.split('\n'):
                    if sub.endswith(self.domain):
                        subdomains.add(sub)
            
            self.discovered.update(subdomains)
            return list(subdomains)
            
        except Exception as e:
            print(f"[!] CT log error: {e}")
            return []
    
    def bruteforce_subdomains(self, wordlist: List[str] = None) -> List[str]:
        """DNS bruteforce additional subdomains"""
        if not wordlist:
            wordlist = [
                'www', 'mail', 'ftp', 'smtp', 'pop', 'ns1', 'ns2',
                'dns', 'mx', 'webmail', 'cpanel', 'whm', 'api',
                'dev', 'test', 'staging', 'admin', 'portal', 'vpn',
                'remote', 'gateway', 'proxy', 'cdn', 'static',
                'media', 'assets', 'img', 'images', 'css', 'js',
                'app', 'mobile', 'm', 'wap', 'beta', 'alpha'
            ]
        
        found = []
        
        for sub in wordlist:
            domain = f"{sub}.{self.domain}"
            try:
                answers = dns.resolver.resolve(domain, 'A')
                if answers:
                    found.append(domain)
                    self.discovered.add(domain)
            except:
                continue
        
        return found
    
    def get_full_results(self) -> dict:
        """Get all discovered subdomains"""
        return {
            "domain": self.domain,
            "total": len(self.discovered),
            "subdomains": sorted(self.discovered)
        }
```

### Advanced Database Queries

#### Direct SQLite Queries

```bash
# Query the Amass database directly
sqlite3 ~/.config/amass/graph/amass.sql

# List all domains
SELECT DISTINCT domain FROM domains;

# Count subdomains per domain
SELECT domain, COUNT(*) as count 
FROM names 
GROUP BY domain 
ORDER BY count DESC;

# Find recently added entries
SELECT name, domain, created_at 
FROM names 
WHERE created_at > datetime('now', '-7 days');

# Export to CSV
.mode csv
.headers on
.output amass_export.csv
SELECT * FROM names;
```

### Graph Analysis

```python
#!/usr/bin/env python3
"""
amass_graph.py - Build and analyze relationship graphs
"""

import json
from collections import defaultdict
from typing import Dict, List, Set


class GraphAnalyzer:
    """Analyze relationships in amass data"""
    
    def __init__(self, amass_json: str):
        with open(amass_json) as f:
            self.data = json.load(f)
        
        self.ip_to_domains = defaultdict(set)
        self.domain_to_ips = defaultdict(set)
        self.source_graph = defaultdict(set)
    
    def build_graph(self):
        """Build relationship graph from amass data"""
        for entry in self.data:
            domain = entry['name']
            sources = entry.get('sources', [])
            
            for source in sources:
                self.source_graph[source].add(domain)
            
            for addr in entry.get('addresses', []):
                ip = addr.get('ip')
                if ip:
                    self.domain_to_ips[domain].add(ip)
                    self.ip_to_domains[ip].add(domain)
    
    def find_shared_infrastructure(self) -> List[Dict]:
        """Find domains sharing infrastructure"""
        shared = []
        
        for ip, domains in self.ip_to_domains.items():
            if len(domains) > 1:
                shared.append({
                    "ip": ip,
                    "domains": list(domains),
                    "count": len(domains)
                })
        
        return sorted(shared, key=lambda x: x['count'], reverse=True)
    
    def print_summary(self):
        """Print analysis summary"""
        print("\n" + "=" * 60)
        print("GRAPH ANALYSIS SUMMARY")
        print("=" * 60)
        
        print(f"\nTotal domains: {len(self.domain_to_ips)}")
        print(f"Total IPs: {len(self.ip_to_domains)}")
        print(f"Total sources: {len(self.source_graph)}")
        
        print("\nSHARED INFRASTRUCTURE:")
        for infra in self.find_shared_infrastructure()[:5]:
            print(f"  IP {infra['ip']}: {infra['count']} domains")
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: Penetration Testing Engagement

**Objective:** Corporate external assessment with systematic reconnaissance

```bash
#!/bin/bash
# pentest_amass.sh - Penetration testing reconnaissance

CLIENT="financialcorp.com"
ENGAGEMENT="pentest_$(date +%Y%m%d)"
OUTPUT_DIR="/pentest/$ENGAGEMENT"

mkdir -p "$OUTPUT_DIR"

echo "=== External Reconnaissance ==="
echo "Target: $CLIENT"

# Phase 1: Organization intelligence
echo "[Phase 1] Organization Intelligence"
amass intel -org "Financial Corp" \
    -o "$OUTPUT_DIR/org_domains.txt"

# Phase 2: Primary domain enumeration
echo "[Phase 2] Primary Domain Enumeration"
amass enum -d "$CLIENT" \
    -src \
    -o "$OUTPUT_DIR/primary_enum.json" \
    -format json

# Phase 3: Extended enumeration with API sources
echo "[Phase 3] Extended Enumeration"
amass enum -d "$CLIENT" \
    -include shodan,censys,securitytrails \
    -o "$OUTPUT_DIR/extended_enum.json" \
    -format json

# Phase 4: Merge results
echo "[Phase 4] Merging Results"
cat "$OUTPUT_DIR"/*.json | \
    jq -r 'if type == "array" then .[].name else .name end' | \
    sort -u > "$OUTPUT_DIR/all_subdomains.txt"

TOTAL=$(wc -l < "$OUTPUT_DIR/all_subdomains.txt")
echo "Total subdomains discovered: $TOTAL"

# Phase 5: Live host check
echo "[Phase 5] Live Host Discovery"
cat "$OUTPUT_DIR/all_subdomains.txt" | \
    httpx -silent -status-code -o "$OUTPUT_DIR/live_hosts.txt"

# Phase 6: Port scanning
echo "[Phase 6] Port Scanning"
nmap -iL "$OUTPUT_DIR/live_hosts.txt" \
    -T4 -Pn -sV --version-intensity 5 \
    -oA "$OUTPUT_DIR/port_scan"

echo "=== Reconnaissance Complete ==="
```

### Scenario 2: Bug Bounty Hunting

**Objective:** Find subdomain takeover and information disclosure vulnerabilities

```python
#!/usr/bin/env python3
"""
bugbounty_amass.py - Bug bounty reconnaissance automation
"""

import subprocess
import json
from pathlib import Path
from typing import List, Dict
import requests
import time


class BugBountyRecon:
    def __init__(self, program: str, domains: List[str]):
        self.program = program
        self.domains = domains
        self.output_dir = Path(f"bugbounty_{program}")
        self.output_dir.mkdir(exist_ok=True)
        
    def run_amass(self, domain: str) -> List[str]:
        """Run amass enumeration"""
        output_file = self.output_dir / f"{domain.replace('.', '_')}.txt"
        
        cmd = [
            "amass", "enum",
            "-passive",
            "-d", domain,
            "-o", str(output_file)
        ]
        
        subprocess.run(cmd, capture_output=True)
        
        with open(output_file) as f:
            return [line.strip() for line in f if line.strip()]
    
    def check_takeover(self, subdomains: List[str]) -> List[Dict]:
        """Check for subdomain takeover opportunities"""
        vulnerable = []
        
        fingerprints = {
            "amazonaws.com": ["NoSuchBucket", "No Such Bucket"],
            "herokuapp.com": ["No such app"],
            "github.io": ["There isn't a GitHub Pages site here."],
            "shopify.com": ["Sorry, this shop is currently unavailable."],
            "fastly.net": ["Fastly error: unknown domain"],
            "pantheon.io": ["404 error unknown site"],
        }
        
        for subdomain in subdomains:
            url = f"https://{subdomain}"
            try:
                response = requests.get(url, timeout=10, verify=False)
                
                for service, pattern in fingerprints.items():
                    if pattern[0] in response.text:
                        vulnerable.append({
                            "subdomain": subdomain,
                            "service": service,
                            "evidence": pattern[0],
                            "status": response.status_code
                        })
                        print(f"[!] POTENTIAL TAKEOVER: {subdomain}")
                        break
                        
            except requests.RequestException:
                continue
            
            time.sleep(0.5)
        
        return vulnerable
    
    def run(self):
        """Execute full bug bounty reconnaissance"""
        all_subdomains = []
        
        print(f"[*] Starting reconnaissance for {self.program}")
        
        for domain in self.domains:
            print(f"[*] Enumerating {domain}...")
            subs = self.run_amass(domain)
            all_subdomains.extend(subs)
            print(f"[+] Found {len(subs)} subdomains")
        
        all_subdomains = list(set(all_subdomains))
        print(f"\n[*] Total unique subdomains: {len(all_subdomains)}")
        
        print("\n[*] Checking for subdomain takeover...")
        takeovers = self.check_takeover(all_subdomains)
        
        print(f"\n[+] Potential takeovers: {len(takeovers)}")


def main():
    recon = BugBountyRecon(
        program="target_startup",
        domains=["target.com", "target.io"]
    )
    recon.run()


if __name__ == "__main__":
    main()
```

### Scenario 3: CTF Competition

**Objective:** Fast reconnaissance for time-limited challenges

```bash
#!/bin/bash
# ctf_amass.sh - CTF reconnaissance script

CHALLENGE_DOMAIN=$1

echo "=== CTF Reconnaissance ==="
echo "Target: $CHALLENGE_DOMAIN"

# Quick enumeration (time is critical in CTF)
echo "[*] Running quick passive enumeration..."
timeout 60 amass enum -passive -d "$CHALLENGE_DOMAIN" \
    -o ctf_subs.txt

# Check for common CTF patterns
echo "[*] Looking for CTF-specific subdomains..."
while IFS= read -r sub; do
    response=$(curl -s -o /dev/null -w "%{http_code}" "https://$sub" 2>/dev/null)
    
    if [ "$response" = "200" ]; then
        echo "[+] $sub - HTTP $response"
        curl -s "https://$sub" | grep -i "flag\|ctf\|key" | head -5
    fi
    
    # Check common CTF paths
    for path in "/flag" "/flag.txt" "/robots.txt" "/.well-known/" "/admin/"; do
        status=$(curl -s -o /dev/null -w "%{http_code}" "https://$sub$path" 2>/dev/null)
        if [ "$status" = "200" ]; then
            echo "[!] FOUND: https://$sub$path"
            curl -s "https://$sub$path"
        fi
    done
done < ctf_subs.txt
```

### Scenario 4: Red Team Operations

**Objective:** Map target infrastructure with operational security

```python
#!/usr/bin/env python3
"""
redteam_amass.py - Red team reconnaissance with OPSEC
"""

import subprocess
import json
import random
import time
from typing import List, Dict
import requests


class RedTeamRecon:
    def __init__(self, target: str):
        self.target = target
        self.results = {
            "target": target,
            "infrastructure": [],
            "subdomains": []
        }
    
    def passive_recon(self) -> Dict:
        """Passive-only reconnaissance"""
        print(f"[*] Passive recon for {self.target}")
        
        cmd = [
            "amass", "enum",
            "-passive",
            "-d", self.target,
            "-include", "shodan,censys,virustotal",
            "-o", f"/tmp/amass_{self.target}.json",
            "-format", "json"
        ]
        
        # Add random delay to avoid patterns
        delay = random.uniform(5, 15)
        time.sleep(delay)
        
        subprocess.run(cmd, capture_output=True)
        
        try:
            with open(f"/tmp/amass_{self.target}.json") as f:
                data = json.load(f)
                self.results["subdomains"] = [e['name'] for e in data]
        except:
            pass
        
        return self.results
    
    def generate_report(self) -> str:
        """Generate operational report"""
        report = f"""
=== Red Team Reconnaissance Report ===

Target: {self.target}

Subdomains Discovered: {len(self.results['subdomains'])}

Key Findings:
"""
        for sub in self.results['subdomains'][:10]:
            report += f"  - {sub}\n"
        
        return report


def main():
    import sys
    
    if len(sys.argv) < 2:
        print("Usage: python redteam_amass.py <target>")
        sys.exit(1)
    
    recon = RedTeamRecon(sys.argv[1])
    recon.passive_recon()
    print(recon.generate_report())


if __name__ == "__main__":
    main()
```

---

## Chapter 7: Detection & Defense

### How Amass is Detected

#### Network-Level Detection

##### DNS Query Analysis

Amass generates distinctive DNS query patterns that can be detected:

```bash
# Detection patterns in DNS logs

# 1. Rapid sequential queries
# Amass queries many subdomains quickly
# Normal: ~10-20 queries/minute
# Amass: 100+ queries/minute

# 2. Pattern recognition
# Common Amass patterns:
# - Sequential subdomain enumeration
# - TXT record queries for email discovery
# - NS record enumeration
# - AXFR zone transfer attempts

# Example detection query (Splunk)
index=dns src_ip="AMASS_IP" | 
    stats count as query_count by src_ip | 
    where query_count > 100
```

##### Query Signature Detection

```python
#!/usr/bin/env python3
"""
detect_amass.py - Detect Amass activity in DNS logs
"""

import re
from collections import Counter, defaultdict
from typing import List, Dict


class AmassDetector:
    """Detect Amass enumeration patterns"""
    
    AMASS_PATTERNS = [
        r'^(www|mail|ftp|smtp|pop|ns[12]|dns|webmail)\.',
        r'^(admin|portal|vpn|remote|gateway)\.',
        r'^(api|dev|test|staging|beta)\.',
        r'^(amass|recon|scan|enum)\.',
    ]
    
    NORMAL_RATE = 20
    AMASS_RATE = 100
    
    def __init__(self):
        self.query_history = defaultdict(list)
    
    def analyze_query(self, src_ip: str, query: str, timestamp: float) -> Dict:
        alerts = []
        
        for pattern in self.AMASS_PATTERNS:
            if re.match(pattern, query, re.IGNORECASE):
                alerts.append({
                    "type": "pattern_match",
                    "query": query,
                    "confidence": 0.6
                })
        
        self.query_history[src_ip].append(timestamp)
        
        recent = [t for t in self.query_history[src_ip] 
                  if timestamp - t < 60]
        
        if len(recent) > self.AMASS_RATE:
            alerts.append({
                "type": "high_query_rate",
                "rate": len(recent),
                "confidence": 0.8
            })
        
        return {
            "src_ip": src_ip,
            "query": query,
            "alerts": alerts,
            "risk_level": "high" if any(a.get("confidence", 0) >= 0.8 for a in alerts) else "medium" if alerts else "low"
        }
```

#### SIEM Query Examples

```sql
-- Splunk: Detect Amass enumeration
index=dns
| stats count as query_count by src_ip
| where query_count > 100
| join type=lookup asset_inventory src_ip
| table src_ip, query_count, asset_type

-- Microsoft Sentinel: Amass detection
DNSEvents
| where TimeGenerated > ago(1h)
| summarize QueryCount = count() by ClientIP
| where QueryCount > 100
| join kind=inner (
    DNSEvents
    | where TimeGenerated > ago(1h)
    | where QueryType == "TXT"
    | summarize TXTCount = count() by ClientIP
) on ClientIP
| where TXTCount > 10
```

### Defense Strategies

#### DNS Security Measures

##### Rate Limiting

```bash
# Configure DNS rate limiting (BIND)
# named.conf
options {
    rate-limit {
        responses-per-second 10;
        window 5;
        log-only no;
    };
};
```

##### DNS Firewall (RPZ)

```bash
# Response Policy Zone to block enumeration
# rpz.db
; Block known Amass scanner IPs
100.0.0.127.zen.rpz.drop   CNAME .
```

##### Query Logging and Monitoring

```bash
# Enable DNS query logging (BIND)
logging {
    channel query_log {
        file "/var/log/named/queries.log" versions 5 size 50m;
        severity info;
        print-time yes;
        print-category yes;
    };
    category queries { query_log; };
};
```

#### Network-Level Defenses

```bash
#!/bin/bash
# dns_defense.sh - DNS security hardening

echo "=== DNS Defense Configuration ==="

# 1. Enable response rate limiting
cat >> /etc/bind/named.conf << 'EOF'
rate-limit {
    responses-per-second 5;
    window 10;
    log-only no;
};
EOF

# 2. Block zone transfers
cat >> /etc/bind/named.conf << 'EOF'
zone "example.com" {
    type master;
    file "/etc/bind/db.example.com";
    allow-transfer { none; };
    allow-query { internal-networks; };
};
EOF

# 3. Configure firewall rules
iptables -A INPUT -p udp --dport 53 -m limit --limit 100/min -j ACCEPT
iptables -A INPUT -p udp --dport 53 -j DROP
iptables -A INPUT -p tcp --dport 53 -m limit --limit 100/min -j ACCEPT
iptables -A INPUT -p tcp --dport 53 -j DROP

echo "[+] DNS defense configured"
```

### Blue Team Response

1. **Monitor for reconnaissance** - Watch for DNS query rate spikes
2. **Analyze scan patterns** - Identify scanner IP and methodology
3. **Check for follow-up attacks** - Look for exploitation attempts
4. **Block scanner IPs** - Update firewall and DNS RPZ rules
5. **Document the incident** - Record all findings
6. **Report to management** - Notify stakeholders

---

## Chapter 8: Troubleshooting

### Common Errors

#### Error: "command not found"

```bash
# Problem
$ amass -version
-bash: amass: command not found

# Solution 1: Check installation method
which amass
echo $PATH

# Solution 2: Reinstall with Go
go install github.com/owasp-amass/amass/v4/...@master

# Solution 3: Add to PATH (for go install)
export PATH=$PATH:$(go env GOPATH)/bin
echo 'export PATH=$PATH:$(go env GOPATH)/bin' >> ~/.bashrc

# Solution 4: Use full path
~/go/bin/amass -version
```

#### Error: "permission denied"

```bash
# Problem
$ amass enum -d example.com
permission denied: /usr/local/bin/amass

# Solution 1: Install to user directory
go install -v ./...
# Installs to ~/go/bin/amass

# Solution 2: Fix permissions
sudo chmod +x /usr/local/bin/amass
```

#### Error: API key issues

```bash
# Problem
[!] Could not retrieve API data: Unauthorized

# Solution 1: Check config file
cat ~/.config/amass/config.ini

# Solution 2: Verify API keys
curl -s "https://api.shodan.io/api-info?key=YOUR_SHODAN_KEY"

# Solution 3: Use environment variables
export AMASS_SHODAN_API_KEY="your_key_here"
```

#### Error: "no such host"

```bash
# Problem
[!] DNS lookup failed for example.com

# Solution 1: Check DNS configuration
cat /etc/resolv.conf
nslookup example.com
dig example.com

# Solution 2: Use custom resolvers
amass enum -d example.com -rf resolvers.txt

# Solution 3: Test with known resolvers
amass enum -d example.com -r 8.8.8.8,1.1.1.1
```

#### Error: "context deadline exceeded"

```bash
# Problem
[!] Timeout reached for query

# Solution 1: Increase timeout
amass enum -d example.com -timeout 60

# Solution 2: Reduce concurrent queries
amass enum -d example.com -max-dns-queries 50

# Solution 3: Check network connectivity
ping 8.8.8.8
traceroute 8.8.8.8
```

#### Error: "database is locked"

```bash
# Problem
[!] Database is locked by another process

# Solution 1: Kill other amass processes
pkill -f amass

# Solution 2: Remove lock file
rm -f ~/.config/amass/graph/*.lock

# Solution 3: Reinitialize database
amass db -init
```

#### Error: "out of memory"

```bash
# Problem
fatal error: runtime: out of memory

# Solution 1: Reduce memory usage
amass enum -d example.com -passive

# Solution 2: Process in smaller batches
amass enum -d sub1.example.com
amass enum -d sub2.example.com

# Solution 3: Increase swap
sudo fallocate -l 4G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```

### Performance Issues

#### Slow Enumeration

```bash
# Problem: Enumeration taking too long

# Solution 1: Use passive mode
amass enum -passive -d example.com

# Solution 2: Limit sources
amass enum -d example.com \
    -include shodan,censys \
    -exclude archiveit,commoncrawl

# Solution 3: Increase parallelism
amass enum -d example.com \
    -max-dns-queries 200 \
    -rf resolvers.txt
```

#### High Memory Usage

```bash
# Problem: Amass consuming too much memory

# Monitor memory usage
top -p $(pgrep amass)

# Solution 1: Process smaller targets
amass enum -d sub.example.com

# Solution 2: Use passive only
amass enum -passive -d example.com

# Solution 3: Clear database
amass db -init
```

### FAQ

**Q: Why am I getting fewer results than expected?**
A: API keys significantly enhance results. Without keys: ~50-100 subdomains. With keys: ~500-1000+ subdomains. Check your API configuration and verify keys are working.

**Q: How do I rate limit Amass?**
A: Use `-max-dns-queries 50` to limit DNS queries, `-timeout 30` to add delays, or `-passive` mode for minimal footprint.

**Q: How do I export results for other tools?**
A: Use `-o results.txt` for simple text lists, `-o results.json -format json` for JSON output, or pipe directly to httpx/nmap/nuclei.

**Q: How do I speed up enumeration?**
A: Use passive mode (fastest), limit to high-quality sources (`-include shodan,censys`), increase parallelism (`-max-dns-queries 200`), or use fast resolvers.

**Q: How do I handle large-scale engagements?**
A: Use GNU parallel for parallel execution, split domains across multiple machines, and manage the database with `amass db` commands.

### Debugging Techniques

```bash
# Enable verbose logging
amass enum -d example.com -v

# Save and analyze logs
amass enum -d example.com -v 2>&1 | tee amass.log

# Check for errors
grep -i error amass.log
grep -i warning amass.log

# Test DNS resolution
dig example.com A
dig example.com ANY

# Test API endpoints
curl -s "https://api.shodan.io/api-info?key=YOUR_KEY"

# Monitor network traffic
tcpdump -i eth0 port 53
```

### Troubleshooting Script

```bash
#!/bin/bash
# amass-troubleshoot.sh - Automated troubleshooting

echo "=== Amass Troubleshooting ==="

# Check installation
echo ""
echo "[1] Checking installation..."
if command -v amass &> /dev/null; then
    echo "  [✓] Amass found: $(which amass)"
    echo "  [✓] Version: $(amass -version)"
else
    echo "  [✗] Amass not found in PATH"
fi

# Check configuration
echo ""
echo "[2] Checking configuration..."
CONFIG_DIR="$HOME/.config/amass"
CONFIG_FILE="$CONFIG_DIR/config.ini"

if [ -d "$CONFIG_DIR" ]; then
    echo "  [✓] Config directory exists"
else
    echo "  [!] Config directory missing"
fi

if [ -f "$CONFIG_FILE" ]; then
    KEY_COUNT=$(grep -c "=" "$CONFIG_FILE" 2>/dev/null || echo 0)
    echo "  [✓] Config file found ($KEY_COUNT entries)"
else
    echo "  [!] No config file found"
fi

# Check network
echo ""
echo "[3] Checking network..."
if ping -c 1 8.8.8.8 &> /dev/null; then
    echo "  [✓] Network connectivity OK"
else
    echo "  [✗] Network connectivity failed"
fi

# Quick test
echo ""
echo "[4] Quick test..."
if timeout 10 amass enum -passive -d example.com -o /dev/null 2>/dev/null; then
    echo "  [✓] Passive enumeration working"
else
    echo "  [!] Passive enumeration failed"
fi

echo ""
echo "=== Troubleshooting Complete ==="
```

---

## Resources

### Official Documentation

- [OWASP Amass Documentation](https://github.com/owasp-amass/amass)
- [Amass Usage Guide](https://github.com/owasp-amass/amass/blob/master/doc/doc.md)
- [OWASP Project Page](https://owasp.org/projects/project-amass/)
- [GitHub Repository](https://github.com/owasp-amass/amass)

### API Services

- [Shodan](https://shodan.io) - Service discovery
- [Censys](https://censys.io) - Certificate and host data
- [SecurityTrails](https://securitytrails.com) - DNS history
- [VirusTotal](https://virustotal.com) - Malware intelligence
- [BinaryEdge](https://binaryedge.io) - Internet-wide scanning
- [FullHunt](https://fullhunt.io) - Attack surface discovery

### Related Tools

- **subfinder:** Fast passive subdomain enumeration
- **recon-ng:** Full-featured web reconnaissance framework
- **theHarvester:** Email and subdomain harvesting
- **dnsrecon:** DNS enumeration tool
- **sublist3r:** Subdomain bruteforcer

### Practice Resources

- [HackTheBox](https://www.hackthebox.com/) - CTF platform
- [TryHackMe](https://tryhackme.com/) - Learning platform
- [VulnHub](https://www.vulnhub.com/) - Vulnerable VMs

### Certifications That Use Amass

- OSCP (Offensive Security Certified Professional)
- CEH (Certified Ethical Hacker)
- GPEN (GIAC Penetration Tester)
- PNPT (Practical Network Penetration Tester)
- CompTIA Security+
