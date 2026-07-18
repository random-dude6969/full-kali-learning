---
tool_name: "spiderfoot"
display_name: "SpiderFoot"
mitre_tactic: "reconnaissance"
mitre_tactic_id: "TA0043"
mitre_subcategory: "host_information"
install_command: "sudo apt install spiderfoot"
version: "3.0"
github_repo: "https://github.com/smicallef/spiderfoot"
kali_tools_page: "https://www.kali.org/tools/spiderfoot/"
difficulty_level: "beginner-to-advanced"
requires_root: false
gui_available: true
tags: ["osint", "automation", "reconnaissance", "multi-tool", "api"]
related_tools: ["amass", "recon-ng", "theharvester"]
---

# SpiderFoot — OSINT Automation Framework

## Chapter 1: Introduction & Overview

### What is SpiderFoot?
SpiderFoot is an open-source OSINT automation tool with over 200 modules for gathering intelligence about IP addresses, domain names, email addresses, names, and more. It can automatically query dozens of data sources and correlate findings, making it one of the most comprehensive reconnaissance frameworks available.

### History & Background
- **Created:** 2012 by Steve Micallef
- **Maintained:** SpiderFoot community and contributors
- **License:** MIT
- **Language:** Python 3
- **Purpose:** Automated OSINT data collection and correlation
- **Architecture:** Client-server model with SQLite/PostgreSQL backend
- **Modules:** 200+ data sources and analysis modules

### When to Use This Tool
- **Host reconnaissance:** Gather comprehensive information about targets
- **OSINT automation:** Automate data collection from multiple sources
- **Threat intelligence:** Build profiles of threats and actors
- **Network mapping:** Discover relationships between assets
- **Correlation analysis:** Link disparate data points together
- **Continuous monitoring:** Set up recurring scans for ongoing intelligence
- **Penetration testing:** Pre-engagement reconnaissance phase

### When NOT to Use This Tool
- Without proper authorization for the investigation
- For unauthorized access to systems or data
- For harassment or privacy violations
- Against production systems without permission
- For any illegal activity

### Legal and Ethical Considerations
- Only access publicly available information
- Use with proper authorization for investigations
- Document all activities for legal compliance
- Respect platform terms of service
- Consider privacy implications of data collection
- Obtain written permission before scanning target networks
- Stay within the defined scope of engagement

### Key Concepts
- **Module:** A data source or analysis component that extracts specific information
- **Scan:** An automated investigation process using one or more modules
- **Correlation:** Linking data from multiple sources to build a complete picture
- **Footprint:** Complete digital profile of a target entity
- **Data source:** An external API or service queried for information
- **Event:** A piece of information discovered during a scan
- **Entity:** A target being investigated (IP, domain, email, name, etc.)

---

## Chapter 2: Installation & Setup

### System Requirements
- **Python:** 3.8+
- **RAM:** 2 GB minimum (4 GB recommended)
- **Disk Space:** 1 GB
- **Network:** Internet connection
- **Privileges:** Normal user (root optional for full functionality)
- **OS:** Linux, macOS, Windows

### Installation Methods

#### Method 1: APT (Recommended)
```bash
sudo apt update
sudo apt install spiderfoot
```
**Expected output:**
```
Reading package lists... Done
Building dependency tree... Done
The following NEW packages will be installed:
  spiderfoot
0 upgraded, 1 newly installed, 0 to remove and 0 not upgraded.
Need to get 12.3 MB of archives.
After this operation, 45.2 MB of additional disk space will be used.
Get:1 http://kali.download/kali kali-rolling/main amd64 spiderfoot amd64 3.0-0kali1 [12.3 MB]
Fetched 12.3 MB in 2s (5.6 MB/s)
Selecting previously unselected package spiderfoot.
(Reading database ... 1234567 files and directories currently installed.)
Preparing to unpack .../spiderfoot_3.0-0kali1_amd64.deb ...
Unpacking spiderfoot (3.0-0kali1) ...
Setting up spiderfoot (3.0-0kali1) ...
Processing triggers for man-db (2.10.2-1) ...
```

#### Method 2: From Source (Latest)
```bash
git clone https://github.com/smicallef/spiderfoot.git
cd spiderfoot
pip3 install -r requirements.txt
```
**Expected output:**
```
Cloning into 'spiderfoot'...
remote: Enumerating objects: 12345, done.
remote: Counting objects: 100% (1234/1234), done.
remote: Compressing objects: 100% (567/567), done.
remote: Total 12345 (delta 890), reused 1100 (delta 780)
Receiving objects: 100% (12345/12345), 5.67 MiB | 4.32 MiB/s, done.
Resolving deltas: 100% (890/890), done.
Collecting requests>=2.25.1
  Downloading requests-2.28.1-py3-none-any.whl (62 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 62.3/62.3 kB 2.1 MB/s eta 0:00:00
Installing collected packages: requests, urllib3, certifi, idna, chardet
Successfully installed requests-2.28.1 urllib3-1.26.12 certifi-2022.6.15 idna-3.4 chardet-5.0.0
```

#### Method 3: Docker
```bash
docker pull spiderfoot/spiderfoot
docker run -p 5001:5001 spiderfoot/spiderfoot
```
**Expected output:**
```
Using default tag: latest
latest: Pulling from spiderfoot/spiderfoot
a2abf6c4d29d: Already exists
a9edb18cadd1: Already exists
589b7251471a: Already exists
186b1aaa4aa6: Pull complete
b4df32aa5a72: Pull complete
a0bcbecc962e: Pull complete
Digest: sha256:4a1597d70c901740055ac70b89f23f2b1a03e05b8c56c3e1d8a2b2c1d2e3f4a5
Status: Downloaded newer image for spiderfoot/spiderfoot:latest
docker.io/spiderfoot/spiderfoot:latest
```

#### Method 4: pip with virtual environment
```bash
python3 -m venv spiderfoot-env
source spiderfoot-env/bin/activate
pip install spiderfoot
```

### Configuration
- Default port: 5001
- Configuration via web UI or config files
- Database: SQLite (default) or PostgreSQL
- Config file location: `/etc/spiderfoot/spiderfoot.conf`

### Verification
```bash
spiderfoot --version
```
**Expected output:**
```
SpiderFoot 3.0
```

---

## Chapter 3: Basic Usage

### First Run
```bash
spiderfoot -l 127.0.0.1:5001
```
**Output:**
```
SpiderFoot 3.0 starting...
[*] Starting web server on 127.0.0.1:5001
[*] Access SpiderFoot at http://127.0.0.1:5001
[*] Database connection established
[*] Loading modules...
[*] 200+ modules loaded successfully
```

### Essential Commands

#### Start SpiderFoot Web UI
```bash
spiderfoot -l 127.0.0.1:5001
```
**Explanation:** Launches the web interface for interactive use on port 5001.

#### Start with Root Account
```bash
spiderfoot -l 127.0.0.1:5001 -a
```
**Explanation:** Creates default admin account for web UI access.

#### CLI Scan
```bash
spiderfoot -s target.com -m sfp_dnsresolve,sfp_whois
```
**Explanation:** Runs specific modules against a target via command line.

#### List Available Modules
```bash
spiderfoot -l list
```
**Explanation:** Displays all available SpiderFoot modules.

#### Start on Different Port
```bash
spiderfoot -l 0.0.0.0:8080
```
**Explanation:** Starts SpiderFoot on port 8080, accessible from all interfaces.

#### Enable Debug Mode
```bash
spiderfoot -d -l 127.0.0.1:5001
```
**Explanation:** Enables verbose debug output for troubleshooting.

### Complete Flags Reference

#### Server Options
| Flag | Description | Example |
|------|-------------|---------|
| `-l`, `--listen` | Listen address:port | `spiderfoot -l 0.0.0.0:5001` |
| `-a`, `--auto-create` | Auto-create admin account | `spiderfoot -a -l 127.0.0.1:5001` |
| `-d`, `--debug` | Enable debug mode | `spiderfoot -d -l 127.0.0.1:5001` |

#### Scan Options
| Flag | Description | Example |
|------|-------------|---------|
| `-s`, `--scan-target` | Target to scan | `spiderfoot -s target.com` |
| `-m`, `--modules` | Comma-separated modules | `-m sfp_dnsresolve,sfp_whois` |
| `-t`, `--scan-type` | Scan type | `-t DOMAIN_NAME` |
| `-o`, `--output` | Output format | `-o json` |

#### Module Options
| Flag | Description | Example |
|------|-------------|---------|
| `--list-modules` | List all modules | `spiderfoot --list-modules` |
| `--module-list` | Show module details | `spiderfoot --module-list sfp_dnsresolve` |

#### General
| Flag | Description | Example |
|------|-------------|---------|
| `-h`, `--help` | Show help | `spiderfoot -h` |
| `--version` | Show version | `spiderfoot --version` |

#### Web Interface Options
| Flag | Description | Example |
|------|-------------|---------|
| `-l`, `--listen` | Bind address | `-l 127.0.0.1:5001` |
| `-a`, `--auto-create` | Auto-create admin | `-a` |
| `-d`, `--debug` | Debug mode | `-d` |

---

## Chapter 4: Intermediate Usage

### Bash Scripting & Automation

#### Automated Multi-Target Scan
```bash
#!/bin/bash
# spiderfoot_batch.sh
TARGETS=("target1.com" "target2.com" "192.168.1.0/24")
OUTPUT_DIR="results/$(date +%Y%m%d)"
mkdir -p "$OUTPUT_DIR"

for target in "${TARGETS[@]}"; do
    clean=$(echo "$target" | tr '/.' '_')
    echo "[*] Scanning: $target"
    spiderfoot -s "$target" -m sfp_dnsresolve,sfp_whois,sfp_shodan \
        -o json > "$OUTPUT_DIR/${clean}.json"
    sleep 10  # Rate limiting
done
echo "[+] All scans complete. Results in $OUTPUT_DIR"
```

#### Extract Specific Data
```bash
# Extract all emails from scan results
spiderfoot -s target.com -m sfp_email -o json | jq '.results[].data'
```

#### Generate HTML Report
```bash
# Convert JSON results to HTML report
spiderfoot -s target.com -m sfp_dnsresolve,sfp_whois -o json > results.json
python3 -c "
import json
with open('results.json') as f:
    data = json.load(f)
html = '<html><body><h1>SpiderFoot Report</h1>'
for item in data.get('results', []):
    html += f'<p>{item[\"type\"]}: {item[\"data\"]}</p>'
html += '</body></html>'
with open('report.html', 'w') as f:
    f.write(html)
"
```

### Python Scripting

#### Programmatic Usage
```python
import subprocess
import json

def scan_target(target, modules=None):
    """Run SpiderFoot scan on a target."""
    cmd = ["spiderfoot", "-s", target, "-o", "json"]
    if modules:
        cmd.extend(["-m", ",".join(modules)])
    result = subprocess.run(cmd, capture_output=True, text=True)
    return json.loads(result.stdout) if result.stdout else None

# Example
results = scan_target("example.com", ["sfp_dnsresolve", "sfp_whois"])
if results:
    for item in results.get("results", []):
        print(f"[+] {item['type']}: {item['data']}")
```

#### Advanced Python Scanner
```python
import subprocess
import json
import csv
from datetime import datetime

class SpiderFootScanner:
    def __init__(self):
        self.results = {}
    
    def scan(self, target, modules=None):
        cmd = ["spiderfoot", "-s", target, "-o", "json"]
        if modules:
            cmd.extend(["-m", ",".join(modules)])
        result = subprocess.run(cmd, capture_output=True, text=True)
        if result.returncode == 0 and result.stdout:
            self.results[target] = json.loads(result.stdout)
        return self.results.get(target)
    
    def export_csv(self, filename):
        with open(filename, 'w', newline='') as f:
            writer = csv.writer(f)
            writer.writerow(['Target', 'Type', 'Data', 'Module'])
            for target, data in self.results.items():
                for item in data.get('results', []):
                    writer.writerow([target, item['type'], item['data'], item.get('module', '')])

# Usage
scanner = SpiderFootScanner()
scanner.scan("example.com", ["sfp_dnsresolve", "sfp_whois"])
scanner.export_csv("results.csv")
```

### Tool Chaining

#### SpiderFoot + Nmap
```bash
# Run SpiderFoot for host info, then Nmap for port scanning
spiderfoot -s target.com -m sfp_dnsresolve -o json | \
    jq -r '.results[].data' | sort -u | \
    xargs -I{} nmap -sV -p- {}
```

#### SpiderFoot + TheHarvester
```bash
# Combine email findings from both tools
spiderfoot -s target.com -m sfp_email -o json | jq -r '.results[].data' > sf_emails.txt
theharvester -d target.com -b google 2>/dev/null | grep -E "@" > th_emails.txt
sort -u sf_emails.txt th_emails.txt > all_emails.txt
```

#### SpiderFoot + Amass
```bash
# Combine subdomain findings
spiderfoot -s target.com -m sfp_dnsresolve -o json | jq -r '.results[].data' > sf_subs.txt
amass enum -d target.com 2>/dev/null > amass_subs.txt
sort -u sf_subs.txt amass_subs.txt > all_subs.txt
```

---

## Chapter 5: Advanced Usage

### Custom Module Configuration
```bash
# Create custom scan profile with specific modules
spiderfoot -s target.com -m sfp_dnsresolve,sfp_whois,sfp_shodan,sfp_virustotal,sfp_cymru
```

### API Integration
```python
import requests

# SpiderFoot API example
API_URL = "http://127.0.0.1:5001"

# Create scan
response = requests.post(f"{API_URL}/api/scan", json={
    "target": "example.com",
    "modules": ["sfp_dnsresolve", "sfp_whois"],
    "scan_name": "API Scan"
})
scan_id = response.json()["id"]

# Check status
status = requests.get(f"{API_URL}/api/scan/{scan_id}/status")
print(f"Scan status: {status.json()['status']}")

# Get results
results = requests.get(f"{API_URL}/api/scan/{scan_id}/results")
for item in results.json():
    print(f"[+] {item['type']}: {item['data']}")
```

### Scheduled Scanning
```bash
# Cron job for daily scans
# Add to crontab: crontab -e
0 2 * * * /usr/bin/spiderfoot -s target.com -m sfp_dnsresolve -o json > /var/log/spiderfoot_daily.json
```

### Docker Deployment
```bash
# Run SpiderFoot in Docker with persistent data
docker run -d \
  --name spiderfoot \
  -p 5001:5001 \
  -v spiderfoot-data:/etc/spiderfoot \
  spiderfoot/spiderfoot
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: Comprehensive Target Reconnaissance
```bash
# Step 1: Start SpiderFoot
spiderfoot -l 127.0.0.1:5001 &

# Step 2: Access web UI at http://127.0.0.1:5001
# Step 3: Create new scan with target "example.com"
# Step 4: Select all modules for comprehensive scan
# Step 5: Review results in web UI

# CLI equivalent:
spiderfoot -s example.com -m sfp_dnsresolve,sfp_whois,sfp_shodan,sfp_virustotal \
    -o json > recon_results.json

# Step 6: Extract key findings
jq -r '.results[] | select(.type=="IP_ADDRESS") | .data' recon_results.json | sort -u
jq -r '.results[] | select(.type=="EMAILADDR") | .data' recon_results.json | sort -u
```

### Scenario 2: Threat Actor Investigation
```bash
# Step 1: Scan known IP addresses
spiderfoot -s 198.51.100.0/24 -m sfp_dnsresolve,sfp_whois -o json > threat_ips.json

# Step 2: Correlate findings
jq -r '.results[].data' threat_ips.json | sort -u > unique_hosts.txt

# Step 3: Deep dive on interesting findings
while read host; do
    echo "[*] Deep scan: $host"
    spiderfoot -s "$host" -m sfp_shodan,sfp_virustotal -o json > "deep_${host}.json"
done < unique_hosts.txt

# Step 4: Generate consolidated report
echo "=== Threat Intelligence Report ===" > report.md
echo "Date: $(date)" >> report.md
echo "" >> report.md
echo "## Unique Hosts" >> report.md
cat unique_hosts.txt >> report.md
```

### Scenario 3: Corporate Network Mapping
```bash
# Step 1: Scan corporate domain
spiderfoot -s corp.example.com -m sfp_dnsresolve -o json > corp_dns.json

# Step 2: Extract subdomains
jq -r '.results[].data' corp_dns.json | grep -E "\." | sort -u > subdomains.txt

# Step 3: Scan discovered hosts for vulnerabilities
spiderfoot -s corp.example.com -m sfp_shodan -o json > corp_vulns.json

# Step 4: Cross-reference with threat intelligence
while read sub; do
    spiderfoot -s "$sub" -m sfp_virustotal -o json > "vt_${sub}.json"
done < subdomains.txt
```

---

## Chapter 7: Detection & Defense

### How Defenders Detect SpiderFoot
- **Request patterns:** Automated queries to multiple OSINT sources
- **User-Agent:** Default Python requests User-Agent
- **Volume:** High number of DNS lookups and API calls
- **Source IPs:** Requests from known scanner IPs
- **Timing:** Regular intervals between requests
- **DNS patterns:** Queries to Shodan, VirusTotal, etc.

### IDS/IPS Signatures
```
# Snort rule for SpiderFoot DNS queries
alert dns any any -> any any (msg:"SpiderFoot DNS Lookup Pattern"; \
  dns.query; content:"shodan.io"; \
  classtype:reconnaissance; sid:1000001; rev:1;)

# Suricata rule for HTTP requests
alert http any any -> any any (msg:"SpiderFoot HTTP Request"; \
  http.user_agent; content:"python-requests"; \
  classtype:reconnaissance; sid:1000002; rev:1;)
```

### Mitigation Strategies
- Monitor DNS queries for OSINT tool patterns
- Implement rate limiting on DNS servers
- Log and alert on bulk reconnaissance activity
- Use DNS filtering to block known OSINT sources
- Deploy network segmentation to limit lateral movement
- Use threat intelligence feeds to identify known scanner IPs

### Blue Team Response
1. Monitor for automated DNS query patterns
2. Log access to OSINT APIs from internal networks
3. Implement network segmentation to limit reconnaissance
4. Use threat intelligence feeds to detect known scanner IPs
5. Set up alerts for unusual DNS query volumes
6. Monitor for connections to known OSINT services

---

## Chapter 8: Troubleshooting

### Common Errors

#### "Port 5001 already in use"
```
Error: Port 5001 already in use
```
**Solution:** Kill existing process or use different port:
```bash
lsof -i :5001 | awk 'NR>1 {print $2}' | xargs kill -9
spiderfoot -l 127.0.0.1:5002
```

#### "Module not found"
```
ModuleNotFoundError: No module named 'requests'
```
**Solution:** Install dependencies:
```bash
pip3 install -r requirements.txt
```

#### "Feed update failed"
```
[!] Failed to update feeds
```
**Solution:** Check internet connectivity and try:
```bash
sudo gvm-feed-update --verbose
```

#### "Database connection error"
```
Error: Unable to connect to database
```
**Solution:** Check database configuration and restart services.

### Performance Tips
- Use `--debug` to troubleshoot slow scans
- Limit modules to reduce scan time
- Use local DNS cache to speed up resolution
- Schedule scans during off-peak hours
- Use PostgreSQL instead of SQLite for large datasets
- Enable connection pooling for concurrent scans

### FAQ
**Q: How do I add custom modules?**
A: Create Python modules in the modules directory following the SpiderFoot module API.

**Q: Can I integrate SpiderFoot with other tools?**
A: Yes, use the JSON output format and pipe to other tools via command line.

**Q: How do I secure the web interface?**
A: Use authentication, bind to localhost, and set up HTTPS reverse proxy.

**Q: What's the difference between CLI and web UI?**
A: The web UI provides interactive visualization and correlation, while CLI is better for automation.

**Q: How do I update SpiderFoot?**
A: Run `sudo apt update && sudo apt upgrade spiderfoot` or `git pull && pip3 install -r requirements.txt`.

---

## Resources

### Official Documentation
- [SpiderFoot GitHub](https://github.com/smicallef/spiderfoot)
- [SpiderFoot Documentation](https://www.spiderfoot.net/documentation/)
- [SpiderFoot API Reference](https://www.spiderfoot.net/documentation/api/)

### Related Tools
- **Recon-ng** — Web reconnaissance framework
- **TheHarvester** — Email and subdomain harvester
- **Amass** — Network mapping of attack surfaces
- **Maltego** — Interactive data mining tool

### Practice
- Use with authorized OSINT labs
- Practice on your own domains and assets
- Test against intentionally vulnerable applications
- Join OSINT communities for tips and techniques
