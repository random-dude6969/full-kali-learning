---
tool_name: "massdns"
display_name: "MassDNS"
mitre_tactic: "reconnaissance"
mitre_tactic_id: "TA0043"
mitre_subcategory: "dns_reconnaissance"
install_command: "sudo apt install massdns"
version: "0.2"
github_repo: "https://github.com/blechschmidt/massdns"
kali_tools_page: "https://www.kali.org/tools/massdns/"
difficulty_level: "advanced"
requires_root: false
gui_available: false
tags: ["dns", "resolver", "high-performance", "subdomain"]
related_tools: ["dnsenum", "dnsrecon", "amass"]
---

# MassDNS

## Chapter 1: Introduction & Overview

### What is MassDNS?

MassDNS is a high-performance DNS stub resolver designed for massive DNS enumeration tasks. Unlike traditional DNS tools that resolve one domain at a time, MassDNS can resolve millions of domains per second by leveraging high-performance asynchronous I/O and configurable parallelism. It's built specifically for large-scale reconnaissance tasks like subdomain brute-forcing, domain validation, and bulk DNS resolution.

MassDNS operates by sending DNS queries to multiple resolvers in parallel, processing responses as they arrive. It supports multiple output formats (simple, JSON, text, CSV) and can handle millions of input domains efficiently. The tool is written in C for maximum performance and includes built-in retry logic, timeout handling, and resolver rotation.

### History & Background

- **Created:** 2018 by Benjamin Schimichel (TU Munich)
- **Paper:** "MassDNS: A Practical Monolithic DNS Resolver with Scalability" (ACM CODASPY 2018)
- **Maintained:** Active community development
- **License:** MIT
- **Language:** C
- **Latest version:** 0.2
- **Upstream repository:** https://github.com/blechschmidt/massdns

The academic paper behind MassDNS demonstrated that it could resolve over 100 million domains per day using commodity hardware, making it orders of magnitude faster than traditional DNS resolution tools. This capability made it the go-to tool for large-scale subdomain enumeration and domain validation tasks.

### When to Use This Tool

- **Mass subdomain enumeration:** Resolve thousands to millions of subdomains quickly
- **Large-scale domain validation:** Check which domains from a wordlist are alive
- **Subdomain brute-forcing:** Generate and resolve subdomains at scale
- **Domain monitoring:** Continuously validate DNS records for large zones
- **Reconnaissance pipelines:** Feed results to web scanners, screenshot tools, and vulnerability scanners
- **CTF competitions:** Quickly enumerate DNS records for large targets
- **Threat intelligence:** Bulk check indicators of compromise (IOCs) against DNS

### Legal and Ethical Considerations

- High-volume DNS queries may trigger abuse complaints from resolvers
- Some resolvers rate-limit or block sources generating excessive queries
- Use authorized resolvers or your own DNS infrastructure
- Document all scanning activities for compliance purposes
- Large-scale subdomain brute-forcing may be detected by DNS monitoring systems
- Always obtain written authorization before scanning target domains
- Consider using your own DNS resolvers to avoid impacting public infrastructure

### Key Concepts

| Concept | Description |
|---------|-------------|
| **Stub Resolver** | A DNS resolver that forwards queries to recursive resolvers rather than resolving directly |
| **Async I/O** | Non-blocking input/output that allows handling thousands of concurrent connections |
| **Resolver Pool** | A list of DNS resolvers that MassDNS rotates through to distribute load |
| **Batch Size** | Number of queries sent before waiting for responses |
| **Socket Count** | Number of concurrent UDP sockets used for sending queries |
| **NXDOMAIN** | Non-Existent Domain — DNS response indicating the domain doesn't exist |
| **CNAME Chain** | A series of CNAME records pointing to other domains |

---

## Chapter 2: Installation & Setup

### System Requirements

- Operating system: Linux (recommended), macOS, or Windows (WSL)
- RAM: 256 MB minimum, 1 GB+ recommended for large-scale tasks
- Disk space: < 10 MB for the tool, additional for output files
- Network: Fast internet connection recommended
- CPU: Multi-core recommended for high performance
- Privileges: Non-root is sufficient (root needed only for source compilation without sudo)

### Installation via APT (Kali/Debian/Ubuntu)

```bash
sudo apt update
sudo apt install massdns
```

Expected output:
```
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following NEW packages will be installed:
  massdns
0 upgraded, 1 newly installed, 0 to remove and 0 not upgraded.
Need to get 42.1 kB of archives.
After this operation, 172 kB of additional disk space will be used.
Get:1 http://http.kali.org/kali kali-rolling/main amd64 massdns 0.2-3kali1 [42.1 kB]
Fetched 42.1 kB in 0s (211 kB/s)
Selecting previously unselected package massdns.
(Reading database ... 1234567 files and directories currently installed.)
Preparing to unpack .../massdns_0.2-3kali1_amd64.deb ...
Unpacking massdns (0.2-3kali1) ...
Setting up massdns (0.2-3kali1) ...
```

### Installation from Source

```bash
# Install build dependencies
sudo apt install build-essential libldns-dev libpcap-dev

# Clone repository
git clone https://github.com/blechschmidt/massdns.git
cd massdns

# Build
make

# Install to /usr/local/bin
sudo cp bin/massdns /usr/local/bin/
sudo chmod +x /usr/local/bin/massdns
```

Expected output:
```
gcc -O3 -Wall -I/usr/include -Ilib/includes -c src/massdns.c -o build/massdns.o
gcc -O3 -Wall -I/usr/include -Ilib/includes -c src/net.c -o build/net.o
gcc -O3 -Wall -I/usr/include -Ilib/includes -c src/dns.c -o build/dns.o
gcc -O3 -Wall -I/usr/include -Ilib/includes -c src/random.c -o build/random.o
gcc -O3 -Wall -I/usr/include -Ilib/includes -c src/configuration.c -o build/configuration.o
gcc -O3 -Wall -I/usr/include -Ilib/includes -c src/main.c -o build/main.o
gcc -O3 -Wall -I/usr/include -Ilib/includes -c src/resolvers.c -o build/resolvers.o
gcc build/*.o -o bin/massdns -lldns -lpthread -lpcap
```

### Docker Installation

```bash
# Build image
docker build -t massdns .

# Run with volume mounts
docker run --rm -v $(pwd):/data massdns -r /data/resolvers.txt -t A -o S /data/domains.txt
```

### Resolver File Preparation

MassDNS requires a list of DNS resolvers. Create a resolver file:

```bash
# Create resolvers.txt with public DNS resolvers
cat > resolvers.txt << 'EOF'
8.8.8.8
8.8.4.4
1.1.1.1
1.0.0.1
9.9.9.9
149.112.112.112
208.67.222.222
208.67.220.220
64.6.64.6
64.6.65.6
185.228.168.9
185.228.169.9
76.76.19.19
76.223.122.150
94.140.14.14
94.140.15.15
EOF
```

**Important:** Use reliable, fast resolvers. Avoid using only one resolver — MassDNS distributes queries across all listed resolvers.

### Verification

```bash
massdns --version
```

Expected output:
```
massdns version 0.2
```

Or verify with:

```bash
massdns --help 2>&1 | head -5
which massdns
```

---

## Chapter 3: Basic Usage

### First Run

```bash
echo "example.com" | massdns -r resolvers.txt -t A -o S
```

Sample output:
```
example.com. A 93.184.216.34
```

### Essential Commands

#### Basic Resolution from File

```bash
massdns -r resolvers.txt -t A -o S domains.txt
```

**Explanation:** Resolves all domains in `domains.txt` using resolvers from `resolvers.txt`, outputting A records in simple format.

Expected output:
```
google.com. A 142.250.80.46
github.com. A 140.82.121.3
example.com. A 93.184.216.34
cloudflare.com. A 104.16.132.229
```

#### Output to File

```bash
massdns -r resolvers.txt -t A -o S domains.txt -w results.txt
```

**Explanation:** Same as above but writes output to `results.txt` instead of stdout.

#### JSON Output

```bash
massdns -r resolvers.txt -t A -o J domains.txt -w results.json
```

**Explanation:** Outputs results in JSON format for easy parsing by scripts and tools.

Expected output:
```json
{"type": "answer", "name": "example.com.", "class": "IN", "status": "NOERROR", "data": {"type": "A", "ttl": 86400, "rdata": "93.184.216.34"}}
```

#### Resolve AAAA Records

```bash
massdns -r resolvers.txt -t AAAA -o S domains.txt
```

**Explanation:** Resolves IPv6 (AAAA) records instead of IPv4 (A) records.

Expected output:
```
google.com. AAAA 2607:f8b0:4004:800::200e
github.com. AAAA 2001:41d0:1:c804::1
```

#### Resolve MX Records

```bash
massdns -r resolvers.txt -t MX -o S domains.txt
```

Expected output:
```
google.com. MX 10 smtp.google.com.
example.com. MX 10 mail.example.com.
```

#### Custom Timeout

```bash
massdns -r resolvers.txt -t A -o S --timeout 10 domains.txt
```

**Explanation:** Sets a 10-second timeout for each query (default is 2 seconds).

#### High Socket Count

```bash
massdns -r resolvers.txt -t A -o S -s 1000 domains.txt
```

**Explanation:** Uses 1000 concurrent UDP sockets for maximum parallelism.

### Complete Flags Reference

| Flag | Long Form | Description | Default | Example |
|------|-----------|-------------|---------|---------|
| `-r` | `--resolvers` | Path to resolver file | none (required) | `massdns -r resolvers.txt` |
| `-t` | `--type` | DNS record type | A | `massdns -t AAAA domains.txt` |
| `-o` | `--output` | Output format (S/J/T/F) | S | `massdns -o J domains.txt` |
| `-w` | `--write` | Write output to file | stdout | `massdns -w results.txt domains.txt` |
| `-s` | `--socket-nums` | Number of concurrent sockets | 500 | `massdns -s 1000 domains.txt` |
| `-b` | `--batch` | Batch size (queries per round) | 1000 | `massdns -b 5000 domains.txt` |
| `-c` | `--retry` | Number of retries per query | 3 | `massdns -c 5 domains.txt` |
| `-t` | `--timeout` | Query timeout in seconds | 2 | `massdns -t 10 domains.txt` |
| `--snaplen` | — | Max response packet size | 1024 | `massdns --snaplen 2048 domains.txt` |
| `--flush` | — | Flush output after each batch | off | `massdns --flush domains.txt` |
| `--norecurse` | — | Don't set Recursion Desired flag | off | `massdns --norecurse domains.txt` |
| `--verify-ip` | — | Verify resolver IPs via UDP | off | `massdns --verify-ip domains.txt` |
| `-q` | `--quiet` | Quiet mode | off | `massdns -q domains.txt` |
| `-h` | `--help` | Show help | — | `massdns -h` |

### Output Formats

| Format | Flag | Description | Example Output |
|--------|------|-------------|----------------|
| Simple | `-o S` | `domain. TYPE data` | `example.com. A 93.184.216.34` |
| JSON | `-o J` | JSON objects | `{"type":"answer","name":"example.com.",...}` |
| Text | `-o T` | Detailed text | Multi-line with record details |
| CSV | `-o F` | CSV format | `example.com,A,93.184.216.34` |

---

## Chapter 4: Intermediate Usage

### Resolver List Preparation

A good resolver list is critical for MassDNS performance. Here's how to prepare one:

```bash
#!/bin/bash
# prepare_resolvers.sh — Create and validate a resolver list

echo "[*] Fetching public DNS resolver lists..."

# Method 1: Use known public resolvers
cat > resolvers_raw.txt << 'EOF'
8.8.8.8
8.8.4.4
1.1.1.1
1.0.0.1
9.9.9.9
149.112.112.112
208.67.222.222
208.67.220.220
64.6.64.6
64.6.65.6
185.228.168.9
185.228.169.9
76.76.19.19
76.223.122.150
94.140.14.14
94.140.15.15
8.26.56.26
8.20.247.20
88.80.248.7
216.146.35.35
EOF

# Method 2: Download from public lists
curl -s "https://raw.githubusercontent.com/trickest/resolvers/main/resolvers.txt" >> resolvers_raw.txt

# Method 3: Extract from local DNS config
grep "nameserver" /etc/resolv.conf | awk '{print $2}' >> resolvers_raw.txt

# Deduplicate and validate
sort -u resolvers_raw.txt | while read resolver; do
    # Quick validation: try a simple query
    if timeout 3 dig @"$resolver" example.com +short > /dev/null 2>&1; then
        echo "$resolver"
    fi
done > resolvers.txt

echo "[+] $(wc -l < resolvers.txt) validated resolvers"
rm -f resolvers_raw.txt
```

### Batch Subdomain Resolution

```bash
#!/bin/bash
# batch_resolve.sh — Resolve large subdomain lists efficiently

SUBDOMAIN_LIST="${1:?Usage: $0 <subdomain_list>}"
RESOLVERS="resolvers.txt"
OUTPUT="resolved_$(date +%Y%m%d_%H%M%S).txt"
ACTIVE_OUTPUT="active_$(date +%Y%m%d_%H%M%S).txt"

echo "[*] Starting batch resolution..."
echo "    Input: $SUBDOMAIN_LIST"
echo "    Domains: $(wc -l < "$SUBDOMAIN_LIST")"
echo "    Resolvers: $(wc -l < "$RESOLVERS")"

# Run MassDNS
massdns -r "$RESOLVERS" -t A -o S -w "$OUTPUT" "$SUBDOMAIN_LIST"

# Filter active domains (those that resolved)
grep " A " "$OUTPUT" | awk '{print $1, $3}' > "$ACTIVE_OUTPUT"

echo "[+] Resolution complete"
echo "    Total results: $(wc -l < "$OUTPUT")"
echo "    Active domains: $(wc -l < "$ACTIVE_OUTPUT")"
```

### Tool Chaining

#### Subfinder → MassDNS Pipeline

```bash
#!/bin/bash
# subfinder_massdns.sh — Subdomain enumeration pipeline

TARGET="${1:?Usage: $0 <domain>}"
OUTPUT_DIR="recon_$(date +%Y%m%d)"
mkdir -p "$OUTPUT_DIR"

# Step 1: Enumerate subdomains with subfinder
echo "[1/4] Enumerating subdomains..."
subfinder -d "$TARGET" -silent -o "$OUTPUT_DIR/subfinder_raw.txt"

# Step 2: Generate additional subdomains with wordlist
echo "[2/4] Generating subdomains from wordlist..."
# Extract subdomain prefixes from existing results
cat "$OUTPUT_DIR/subfinder_raw.txt" | sed "s/\.${TARGET}//g" > /tmp/prefixes.txt

# Step 3: Resolve with MassDNS
echo "[3/4] Resolving subdomains..."
massdns -r resolvers.txt -t A -o S \
    -w "$OUTPUT_DIR/massdns_raw.txt" \
    "$OUTPUT_DIR/subfinder_raw.txt"

# Step 4: Filter and deduplicate
echo "[4/4] Filtering results..."
grep " A " "$OUTPUT_DIR/massdns_raw.txt" | \
    awk '{print $1}' | \
    sed 's/\.$//' | \
    sort -u > "$OUTPUT_DIR/active_subs.txt"

echo "[+] Pipeline complete"
echo "    Total subdomains found: $(wc -l < "$OUTPUT_DIR/subfinder_raw.txt")"
echo "    Active subdomains: $(wc -l < "$OUTPUT_DIR/active_subs.txt")"
```

#### MassDNS → httpx Pipeline

```bash
#!/bin/bash
# massdns_httpx.sh — Resolve and probe web servers

TARGET="${1:?Usage: $0 <domain>}"

# Resolve
subfinder -d "$TARGET" -silent | \
    massdns -r resolvers.txt -t A -o S | \
    grep " A " | awk '{print $3}' | sort -u | \
    httpx -silent -title -tech-detect -status-code > web_results.txt
```

#### MassDNS → Nuclei Pipeline

```bash
#!/bin/bash
# massdns_nuclei.sh — Full recon pipeline

TARGET="${1:?Usage: $0 <domain>}"

# Step 1: Subdomain enum
subfinder -d "$TARGET" -silent > subs.txt

# Step 2: Resolve
massdns -r resolvers.txt -t A -o S -w resolved.txt subs.txt

# Step 3: Extract active subdomains
grep " A " resolved.txt | awk '{print $1}' | sed 's/\.$//' | sort -u > active.txt

# Step 4: Probe for web servers
cat active.txt | httpx -silent -o webservers.txt

# Step 5: Run nuclei templates
nuclei -l webservers.txt -o nuclei_results.txt
```

### Python Integration

```python
#!/usr/bin/env python3
"""
massdns_pipeline.py — Automated MassDNS integration with Python
"""

import subprocess
import json
import os
from datetime import datetime

class MassDNSPipeline:
    def __init__(self, resolvers_file="resolvers.txt"):
        self.resolvers = resolvers_file
        self.output_dir = f"massdns_results_{datetime.now().strftime('%Y%m%d_%H%M%S')}"
        os.makedirs(self.output_dir, exist_ok=True)
    
    def resolve(self, input_file, record_type="A", output_format="S", 
                sockets=500, timeout=2, retries=3):
        """Run MassDNS on input file."""
        output_file = os.path.join(self.output_dir, f"resolved_{record_type}.txt")
        
        cmd = [
            "massdns",
            "-r", self.resolvers,
            "-t", record_type,
            "-o", output_format,
            "-s", str(sockets),
            "-w", output_file,
            input_file
        ]
        
        print(f"[*] Running MassDNS: {' '.join(cmd)}")
        result = subprocess.run(cmd, capture_output=True, text=True, timeout=7200)
        
        if result.returncode != 0:
            print(f"[!] MassDNS error: {result.stderr}")
            return None
        
        return output_file
    
    def parse_results(self, output_file, record_type="A"):
        """Parse MassDNS output into structured data."""
        results = []
        with open(output_file, 'r') as f:
            for line in f:
                parts = line.strip().split()
                if len(parts) >= 3 and parts[1] == record_type:
                    results.append({
                        "domain": parts[0].rstrip('.'),
                        "type": parts[1],
                        "value": parts[2]
                    })
        return results
    
    def get_active_domains(self, results):
        """Filter to only domains that resolved."""
        return [r for r in results if r["value"] not in ("0.0.0.0", "::", "")]
    
    def generate_report(self, input_file, results):
        """Generate a summary report."""
        report = os.path.join(self.output_dir, "report.json")
        data = {
            "timestamp": datetime.now().isoformat(),
            "input_file": input_file,
            "total_input": sum(1 for _ in open(input_file)),
            "total_resolved": len(results),
            "active_domains": len(self.get_active_domains(results)),
            "results": results[:100]  # First 100 for report
        }
        with open(report, 'w') as f:
            json.dump(data, f, indent=2)
        return report

# Example usage
pipeline = MassDNSPipeline()
output = pipeline.resolve("subdomains.txt")
if output:
    results = pipeline.parse_results(output)
    active = pipeline.get_active_domains(results)
    print(f"[+] Resolved {len(results)} domains, {len(active)} active")
    pipeline.generate_report("subdomains.txt", results)
```

---

## Chapter 5: Advanced Usage

### High-Performance Configuration

```bash
# Maximum performance settings
massdns -r resolvers.txt \
    -t A \
    -o S \
    -s 10000 \
    -b 50000 \
    -c 3 \
    -t 5 \
    --flush \
    -w high_perf_results.txt \
    large_subdomain_list.txt
```

**Performance tuning guidelines:**
- **Socket count (`-s`):** Start with 500, increase to 10000+ for fast networks
- **Batch size (`-b`):** Set to 10x socket count for optimal throughput
- **Timeout (`-t`):** Lower (1-2s) for fast networks, higher (5-10s) for slow ones
- **Resolvers:** More resolvers = better distribution = higher throughput

### DNS-over-HTTPS (DoH) Support

```bash
# Create DoH resolver list
cat > doh_resolvers.txt << 'EOF'
8.8.8.8
1.1.1.1
9.9.9.9
EOF

# MassDNS supports DoH when resolvers support it
massdns -r doh_resolvers.txt -t A -o J domains.txt
```

### Custom Output Processing

```bash
# Resolve and extract domain-IP pairs
massdns -r resolvers.txt -t A -o S domains.txt | \
    awk '/ A / {gsub(/\.$/, "", $1); print $1, $3}' > domain_ip_pairs.txt

# Resolve and generate hosts file format
massdns -r resolvers.txt -t A -o S domains.txt | \
    awk '/ A / {print $3, $1}' | sed 's/\.$//' > /tmp/hosts_format.txt

# Count results by status
massdns -r resolvers.txt -t A -o S domains.txt | \
    awk '{print $2}' | sort | uniq -c | sort -rn
```

### Pipeline with Chaining

```bash
#!/bin/bash
# full_recon_pipeline.sh — Complete recon pipeline with MassDNS

TARGET="${1:?Usage: $0 <domain>}"
WORKDIR="recon_${TARGET}_$(date +%Y%m%d)"
mkdir -p "$WORKDIR"

echo "=== Full Recon Pipeline: $TARGET ==="

# Phase 1: Subdomain Enumeration
echo "[Phase 1] Subdomain enumeration..."
subfinder -d "$TARGET" -silent > "$WORKDIR/subfinder.txt"
amass enum -passive -d "$TARGET" > "$WORKDIR/amass.txt"
cat "$WORKDIR/subfinder.txt" "$WORKDIR/amass.txt" | sort -u > "$WORKDIR/all_subs.txt"
echo "  Found $(wc -l < "$WORKDIR/all_subs.txt") unique subdomains"

# Phase 2: Resolution
echo "[Phase 2] Resolving subdomains..."
massdns -r resolvers.txt -t A -o S \
    -s 1000 -w "$WORKDIR/resolved.txt" \
    "$WORKDIR/all_subs.txt"
ACTIVE=$(grep " A " "$WORKDIR/resolved.txt" | wc -l)
echo "  $ACTIVE active subdomains"

# Phase 3: Web Probing
echo "[Phase 3] Probing for web servers..."
grep " A " "$WORKDIR/resolved.txt" | awk '{print $1}' | \
    sed 's/\.$//' | sort -u | \
    httpx -silent -title -tech-detect -status-code \
    -o "$WORKDIR/webservers.txt"
echo "  $(wc -l < "$WORKDIR/webservers.txt") web servers found"

# Phase 4: Screenshot
echo "[Phase 4] Taking screenshots..."
gowitness file -f "$WORKDIR/webservers.txt" -P "$WORKDIR/screenshots" \
    --disable-redirects 2>/dev/null

# Phase 5: Vulnerability Scanning
echo "[Phase 5] Running nuclei..."
nuclei -l "$WORKDIR/webservers.txt" -severity critical,high \
    -o "$WORKDIR/nuclei_results.txt" 2>/dev/null

echo ""
echo "=== Pipeline Complete ==="
echo "Results in: $WORKDIR/"
```

### Monitoring and Metrics

```bash
#!/bin/bash
# massdns_monitor.sh — Monitor MassDNS progress

LOGFILE="${1:?Usage: $0 <massdns_log>}"

echo "=== MassDNS Progress Monitor ==="
echo "Watching: $LOGFILE"
echo ""

tail -f "$LOGFILE" | while read line; do
    if echo "$line" | grep -q " A "; then
        DOMAIN=$(echo "$line" | awk '{print $1}')
        IP=$(echo "$line" | awk '{print $3}')
        echo "[+] $DOMAIN -> $IP"
    fi
done
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: Massive Subdomain Enumeration

**Objective:** Discover all subdomains of a target domain for a comprehensive penetration test.

**Step 1: Prepare Input List**
```bash
# Combine multiple sources
subfinder -d target.com -silent > subs_subfinder.txt
amass enum -passive -d target.com > subs_amass.txt
assetfinder --subs-only target.com > subs_assetfinder.txt

# Combine and deduplicate
cat subs_*.txt | sort -u > all_subdomains.txt
echo "[+] Total unique subdomains: $(wc -l < all_subdomains.txt)"
```

**Step 2: Prepare Resolvers**
```bash
# Create optimized resolver list
cat > resolvers.txt << 'EOF'
8.8.8.8
8.8.4.4
1.1.1.1
1.0.0.1
9.9.9.9
149.112.112.112
208.67.222.222
208.67.220.220
64.6.64.6
64.6.65.6
EOF
```

**Step 3: Resolve with MassDNS**
```bash
# High-performance resolution
massdns -r resolvers.txt \
    -t A \
    -o S \
    -s 1000 \
    -w resolved.txt \
    all_subdomains.txt

echo "[+] Resolution complete"
```

**Step 4: Filter Active Subdomains**
```bash
# Extract active (resolved) subdomains
grep " A " resolved.txt | awk '{print $1}' | sed 's/\.$//' | sort -u > active_subs.txt

# Extract IP addresses
grep " A " resolved.txt | awk '{print $3}' | sort -u > unique_ips.txt

echo "[+] Active subdomains: $(wc -l < active_subs.txt)"
echo "[+] Unique IPs: $(wc -l < unique_ips.txt)"
```

**Step 5: Generate Report**
```bash
cat > subdomain_report.txt << EOF
=== Subdomain Enumeration Report: target.com ===
Date: $(date)

Total subdomains generated: $(wc -l < all_subdomains.txt)
Active subdomains: $(wc -l < active_subs.txt)
Unique IPs: $(wc -l < unique_ips.txt)

Active subdomains:
$(cat active_subs.txt)

Unique IPs:
$(cat unique_ips.txt)
EOF
```

### Scenario 2: CTF DNS Challenge — "Resolve the Flag"

**Objective:** A CTF challenge where subdomains contain flag pieces that must be resolved and combined.

**Step 1: Generate Subdomain Candidates**
```bash
# The challenge hint says "the flag is split across subdomains"
# Generate common subdomain patterns
for i in $(seq 0 9999); do
    printf "flag%04d.target.com\n" $i
    printf "piece%04d.target.com\n" $i
    printf "part%04d.target.com\n" $i
done > ctf_subdomains.txt
```

**Step 2: Resolve with MassDNS**
```bash
massdns -r resolvers.txt -t A -o S -w ctf_resolved.txt ctf_subdomains.txt
```

**Step 3: Find Active Subdomains**
```bash
grep " A " ctf_resolved.txt | awk '{print $1}' | sed 's/\.$//' | sort -u > ctf_active.txt
cat ctf_active.txt
```

Output:
```
piece0042.target.com
piece0137.target.com
piece0289.target.com
piece0456.target.com
piece0678.target.com
piece0912.target.com
```

**Step 4: Query TXT Records for Flag**
```bash
while read sub; do
    echo "--- $sub ---"
    dig "$sub" TXT +short
done < ctf_active.txt
```

### Scenario 3: Domain Validation for Threat Intelligence

**Objective:** Check which domains from a threat intelligence feed are currently active.

**Step 1: Prepare IOC List**
```bash
# Download or prepare IOC list
cat > threat_indicators.txt << 'EOF'
malware-c2.example.com
phishing-bank.example.com
dropper-update.example.com
c2-command.example.com
exfil-data.example.com
EOF
```

**Step 2: Resolve All Indicators**
```bash
massdns -r resolvers.txt -t A -o S -w ioc_resolved.txt threat_indicators.txt
```

**Step 3: Identify Active Indicators**
```bash
grep " A " ioc_resolved.txt | awk '{print $1, $3}' > active_iocs.txt

echo "=== Active Threat Indicators ==="
cat active_iocs.txt

echo ""
echo "=== Summary ==="
echo "Total IOCs: $(wc -l < threat_indicators.txt)"
echo "Active: $(wc -l < active_iocs.txt)"
echo "Inactive: $(( $(wc -l < threat_indicators.txt) - $(wc -l < active_iocs.txt) ))"
```

**Step 4: Generate Report**
```bash
cat > ioc_report.json << EOF
{
  "timestamp": "$(date -Iseconds)",
  "total_iocs": $(wc -l < threat_indicators.txt),
  "active_iocs": $(wc -l < active_iocs.txt),
  "active": [
$(while read domain ip; do
    echo "    {\"domain\": \"$domain\", \"ip\": \"$ip\"},"
done < active_iocs.txt | sed '$ s/,$//')
  ]
}
EOF
```

---

## Chapter 7: Detection & Defense

### How MassDNS Activity Is Detected

**DNS Traffic Analysis:**
- High volume of DNS queries from single source IP
- Queries to many different domains in short time
- UDP floods to port 53 from single source
- Unusual query patterns (e.g., sequential subdomain enumeration)
- Queries for non-existent domains (NXDOMAIN responses)

**Network Signatures:**
```
# Suricata rule for mass DNS enumeration
alert udp $HOME_NET any -> any 53 (msg:"Potential Mass DNS Enumeration"; \
  threshold:type both, track by_src, count 100, seconds 10; \
  sid:1000010; rev:1;)

# Rule for high NXDOMAIN rate
alert dns any any -> any any (msg:"High NXDOMAIN Rate"; \
  dns.flags.rcode:3; \
  threshold:type both, track by_src, count 50, seconds 5; \
  sid:1000011; rev:1;)
```

**Log Analysis:**
```bash
# Detect mass DNS queries in DNS server logs
awk '{print $1}' /var/log/named/query.log | \
    sort | uniq -c | sort -rn | head -20

# Detect high NXDOMAIN rate
grep "NXDOMAIN" /var/log/named/query.log | \
    awk '{print $5}' | sort | uniq -c | sort -rn | head -20
```

### Blue Team Defensive Measures

**1. DNS Rate Limiting:**
```bash
# BIND rate limiting configuration
options {
    rate-limit {
        responses-per-second 10;
        window 5;
        log-only no;
        qps-scale 256;
        slip 1;
        ipv4-prefix-length 24;
        max-table-size 20000;
    };
};
```

**2. DNS Firewall (RPZ):**
```
// Response Policy Zone to block known malicious domains
zone "rpz.example.com" {
    type master;
    file "rpz.example.com.zone";
    allow-query { none; };
    allow-query-on { any; };
};
```

**3. Anomaly Detection:**
```bash
#!/bin/bash
# dns_anomaly_detect.sh — Detect mass DNS enumeration

LOG="/var/log/named/query.log"
THRESHOLD=100
WINDOW=60

# Detect sources with high query rate
awk -v threshold=$THRESHOLD -v window=$WINDOW '
{
    split($1, t, /[\/:]/);
    ts = mktime(t[1]" "t[2]" "t[3]" "t[4]" "t[5]" "t[6]);
    src = $5;
    if (!(src in first_ts) || ts - first_ts[src] > window) {
        count[src] = 1;
        first_ts[src] = ts;
    } else {
        count[src]++;
    }
    if (count[src] > threshold) {
        print "ALERT: " src " made " count[src] " queries in " window " seconds";
    }
}' "$LOG"
```

**4. Resolver Configuration:**
```bash
# Limit recursion to authorized networks
options {
    allow-recursion { 10.0.0.0/8; 192.168.0.0/16; };
    allow-query-cache { 10.0.0.0/8; 192.168.0.0/16; };
    recursion yes;
};
```

---

## Chapter 8: Troubleshooting

### Common Errors

#### "No resolvers available"

**Cause:** Resolver file is empty, doesn't exist, or contains invalid IPs.

**Solutions:**
```bash
# Verify resolver file
cat resolvers.txt

# Test resolver functionality
for resolver in $(cat resolvers.txt); do
    echo -n "$resolver: "
    dig @"$resolver" example.com +short +time=2
done

# Recreate with known good resolvers
cat > resolvers.txt << 'EOF'
8.8.8.8
8.8.4.4
1.1.1.1
9.9.9.9
EOF
```

#### "Too many open files"

**Cause:** Socket count exceeds system file descriptor limit.

**Solutions:**
```bash
# Check current limit
ulimit -n

# Increase temporarily
ulimit -n 65535

# Increase permanently
echo "* soft nofile 65535" | sudo tee -a /etc/security/limits.conf
echo "* hard nofile 65535" | sudo tee -a /etc/security/limits.conf

# Or reduce socket count
massdns -s 100 domains.txt  # Reduced from default 500
```

#### "Timeout"

**Cause:** Resolvers are slow or network latency is high.

**Solutions:**
```bash
# Increase timeout
massdns -t 10 -c 5 domains.txt

# Use faster resolvers
# Test resolver speed
for r in 8.8.8.8 1.1.1.1 9.9.9.9; do
    time dig @"$r" example.com +short
done

# Reduce socket count to decrease network congestion
massdns -s 100 domains.txt
```

#### "malloc: unable to allocate"

**Cause:** Insufficient memory for large input files.

**Solutions:**
```bash
# Check available memory
free -h

# Split large input files
split -l 100000 large_file.txt chunk_
for chunk in chunk_*; do
    massdns -r resolvers.txt -t A -o S -w "result_${chunk}.txt" "$chunk"
done

# Reduce batch size
massdns -b 1000 domains.txt
```

#### "Socket operation timed out"

**Cause:** Network or firewall blocking UDP/53 traffic.

**Solutions:**
```bash
# Test UDP connectivity to resolvers
nc -zvu 8.8.8.8 53

# Check firewall rules
sudo iptables -L -n | grep -E "53|dns"

# Try with TCP (note: MassDNS primarily uses UDP)
dig @8.8.8.8 example.com +tcp
```

### Performance Optimization

```bash
# Benchmark different configurations
#!/bin/bash
# benchmark_massdns.sh — Find optimal MassDNS settings

INPUT="test_domains.txt"  # 10,000 domains for testing
RESOLVERS="resolvers.txt"

echo "=== MassDNS Performance Benchmark ==="
echo ""

for sockets in 100 500 1000 5000; do
    for batch in 1000 5000 10000; do
        echo -n "s=$sockets b=$batch: "
        START=$(date +%s%N)
        massdns -r "$RESOLVERS" -t A -o S -s $sockets -b $batch -q -w /dev/null "$INPUT" 2>/dev/null
        END=$(date +%s%N)
        ELAPSED=$(( (END - START) / 1000000 ))
        echo "${ELAPSED}ms"
    done
done
```

### FAQ

**Q: How many domains can MassDNS resolve per second?**
A: Depending on network conditions and resolver quality, MassDNS can resolve 100,000 to 1,000,000+ domains per second with proper configuration.

**Q: Do I need root privileges to run MassDNS?**
A: No. MassDNS uses unprivileged UDP sockets. Root is only needed for source compilation or if you want to bind to privileged ports.

**Q: Can MassDNS resolve DNSSEC-protected domains?**
A: MassDNS performs standard resolution without DNSSEC validation. It will return the records but doesn't verify signatures.

**Q: What's the difference between `-s` and `-b` flags?**
A: `-s` (sockets) controls the number of concurrent UDP sockets. `-b` (batch) controls how many queries are sent before waiting for responses. Batch should be larger than socket count.

**Q: How do I handle rate limiting by resolvers?**
A: Use more resolvers, reduce socket count, increase timeout, or rotate through resolver lists. You can also use your own DNS resolvers to avoid public rate limits.

**Q: Can MassDNS do DNS-over-TLS or DNS-over-HTTPS?**
A: MassDNS primarily uses UDP. For DoH/DoT, consider using `dnsx` or custom resolver configurations.

---

## Resources

- [MassDNS GitHub Repository](https://github.com/blechschmidt/massdns)
- [MassDNS Paper — ACM CODASPY 2018](https://dl.acm.org/doi/10.1145/3243734.3243746)
- [MassDNS Usage Guide](https://github.com/blechschmidt/massdns/blob/master/README.md)
- [RFC 1035 — Domain Names: Implementation and Specification](https://datatracker.ietf.org/doc/html/rfc1035)
- [RFC 7719 — DNS Terminology](https://datatracker.ietf.org/doc/html/rfc7719)
- [ProjectDiscovery Subfinder](https://github.com/projectdiscovery/subfinder)
- [httpx — HTTP Prober](https://github.com/projectdiscovery/httpx)
- [Nuclei — Vulnerability Scanner](https://github.com/projectdiscovery/nuclei)
- [Kali Linux MassDNS Package](https://www.kali.org/tools/massdns/)
- [DNS Enumeration Best Practices — OWASP](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/02-Configuration_and_Deployment_Management_Testing/05-Enumerate_Infrastructure_and_Application_Admin_Interfaces)
