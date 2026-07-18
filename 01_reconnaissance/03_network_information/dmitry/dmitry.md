---
tool_name: "dmitry"
display_name: "DMitry"
mitre_tactic: "reconnaissance"
mitre_tactic_id: "TA0043"
mitre_subcategory: "network_information"
install_command: "sudo apt install dmitry"
version: "1.6"
github_repo: "https://github.com/jmk-foofus/DeepMagic"
kali_tools_page: "https://www.kali.org/tools/dmitry/"
difficulty_level: "beginner"
requires_root: false
gui_available: false
tags: ["recon", "dns", "email", "subdomain", "whois"]
related_tools: ["theharvester", "dnsenum", "amass"]
---

# DMitry

## Chapter 1: Introduction & Overview

### What is DMitry?
DMitry (DeepMagic Information Gathering Tool) is a UNIX/Linux command-line tool for information gathering. It can gather information about a target host including subdomains, email addresses, whois data, and port scans.

### History & Background
- **Created:** 2001 by James Foofus (jmk-foofus)
- **Maintained:** Foofus.net
- **License:** GPL
- **Language:** C
- **Purpose:** Multi-purpose information gathering
- **Architecture:** Modular design with multiple scan types
- **Features:** Whois, DNS, email, subdomain, and port scanning

### When to Use This Tool
- **Host reconnaissance:** Gather basic host information
- **Email harvesting:** Find email addresses associated with a domain
- **Subdomain discovery:** Find subdomains of a target
- **Whois lookup:** Get registration information
- **Port scanning:** Basic port scan
- **CTF challenges:** Find hidden information in host data

### When NOT to Use This Tool
- Without authorization on target systems
- For malicious reconnaissance
- For any illegal activity

### Legal and Ethical Considerations
- Get written permission before scanning
- Respect target systems and networks
- Document authorization
- Stay within the defined scope of engagement
- Consider the impact on target infrastructure

### Key Concepts
- **Whois:** Domain registration information database
- **Subdomain:** A domain that is part of a larger domain
- **Port scan:** Check which ports are open on a target
- **DNS lookup:** Resolving domain names to IP addresses
- **Banner grabbing:** Identifying service versions from responses

---

## Chapter 2: Installation & Setup

### System Requirements
- **OS:** Linux, macOS
- **RAM:** 64 MB
- **Disk Space:** 10 MB
- **Network:** Internet connection
- **Privileges:** Normal user (root for some features)

### Installation

#### Method 1: APT
```bash
sudo apt update
sudo apt install dmitry
```
**Expected output:**
```
Reading package lists... Done
Building dependency tree... Done
The following NEW packages will be installed:
  dmitry
0 upgraded, 1 newly installed, 0 to remove and 0 not upgraded.
Need to get 156 kB of archives.
After this operation, 456 kB of additional disk space will be used.
Get:1 http://kali.download/kali kali-rolling/main amd64 dmitry amd64 1.6.1-1kali1 [156 kB]
Fetched 156 kB in 0.1s (1.2 MB/s)
Selecting previously unselected package dmitry.
(Reading database ... 1234567 files and directories currently installed.)
Preparing to unpack .../dmitry_1.6.1-1kali1_amd64.deb ...
Unpacking dmitry (1.6.1-1kali1) ...
Setting up dmitry (1.6.1-1kali1) ...
```

#### Method 2: From Source
```bash
git clone https://github.com/jmk-foofus/DeepMagic.git
cd DeepMagic
./configure
make
sudo make install
```
**Expected output:**
```
Cloning into 'DeepMagic'...
remote: Enumerating objects: 567, done.
remote: Counting objects: 100% (56/56), done.
remote: Compressing objects: 100% (23/23), done.
remote: Total 567 (delta 45), reused 50 (delta 40)
Receiving objects: 100% (567/567), 123.45 KiB | 1.56 MiB/s, done.
Resolving deltas: 100% (45/45), done.
checking for gcc... gcc
checking for C compiler default output file name... a.out
checking whether we are using the GNU C compiler... yes
...
make[1]: Leaving directory '/root/DeepMagic'
```

### Verification
```bash
dmitry --version
```
**Expected output:**
```
DeepMagic Information Gathering Tool
Version 1.6.1
```

---

## Chapter 3: Basic Usage

### First Run
```bash
dmitry target.com
```
**Output:**
```
DeepMagic Information Gathering Tool
Version 1.6.1

Write results to dmitry.txt
HostIP:      93.184.216.34
HostName:    target.com

Gathered Inet-whois information for target.com
----------------------------------------------
   registrant: Target Corp
   country: US
   ...
```

### Essential Commands

#### Full Information Gathering
```bash
dmitry -winsepfb target.com
```
**Explanation:** Performs all gathering functions: whois, IP, subdomains, emails, ports, banner grabbing.

#### Whois Lookup
```bash
dmitry -w target.com
```
**Explanation:** Retrieves WHOIS registration information for the target domain.

#### Subdomain Search
```bash
dmitry -s target.com
```
**Explanation:** Searches for subdomains using DNS lookups.

#### Email Search
```bash
dmitry -e target.com
```
**Explanation:** Searches for email addresses associated with the domain.

#### Port Scan
```bash
dmitry -p target.com
```
**Explanation:** Performs a basic port scan on the target.

#### Banner Grabbing
```bash
dmitry -p -b target.com
```
**Explanation:** Grabs service banners from open ports.

### Complete Flags Reference

| Flag | Description | Example |
|------|-------------|---------|
| `-w` | Whois lookup | `dmitry -w target.com` |
| `-i` | IP of target | `dmitry -i target.com` |
| `-n` | Subdomain search | `dmitry -n target.com` |
| `-e` | Email search | `dmitry -e target.com` |
| `-s` | Subdomain + email | `dmitry -s target.com` |
| `-f` | Perform DNS lookup | `dmitry -f target.com` |
| `-p` | Port scan | `dmitry -p target.com` |
| `-b` | Banner grabbing | `dmitry -b target.com` |
| `-o` | Output to file | `dmitry -o output.txt target.com` |
| `-i` | Input from file | `dmitry -i targets.txt` |
| `-t` | TCP port range | `dmitry -p -t 80,443 target.com` |
| `-F` | Filter results | `dmitry -F target.com` |

---

## Chapter 4: Intermediate Usage

### Bash Automation
```bash
#!/bin/bash
# Multi-target recon
TARGETS=("target1.com" "target2.com")
for target in "${TARGETS[@]}"; do
    echo "[*] Scanning: $target"
    dmitry -winsepfb -o "dmitry_${target}.txt" "$target"
done
```

### Tool Chaining
```bash
# Extract subdomains for further enumeration
dmitry -s target.com | grep "Found" | awk '{print $3}' > subdomains.txt
```

### Python Scripting
```python
import subprocess
import re

def dmitry_scan(target, flags="-winsepfb"):
    """Run DMitry scan on a target."""
    cmd = ["dmitry", flags, target]
    result = subprocess.run(cmd, capture_output=True, text=True)
    return result.stdout

def extract_emails(output):
    """Extract emails from DMitry output."""
    return re.findall(r'[\w\.-]+@[\w\.-]+', output)

def extract_subdomains(output):
    """Extract subdomains from DMitry output."""
    return re.findall(r'Found DNS A record: ([\w\.-]+)', output)

# Usage
output = dmitry_scan("target.com")
emails = extract_emails(output)
subdomains = extract_subdomains(output)
print(f"Emails: {emails}")
print(f"Subdomains: {subdomains}")
```

### Advanced Bash Script
```bash
#!/bin/bash
# Comprehensive DMitry recon script
TARGET=$1
OUTPUT_DIR="dmitry_$(date +%Y%m%d)"
mkdir -p "$OUTPUT_DIR"

echo "[*] Starting DMitry scan on: $TARGET"

# Whois lookup
echo "[+] Running whois lookup..."
dmitry -w "$TARGET" > "$OUTPUT_DIR/whois.txt" 2>&1

# Subdomain enumeration
echo "[+] Enumerating subdomains..."
dmitry -n "$TARGET" > "$OUTPUT_DIR/subdomains.txt" 2>&1

# Email harvesting
echo "[+] Harvesting emails..."
dmitry -e "$TARGET" > "$OUTPUT_DIR/emails.txt" 2>&1

# Port scanning
echo "[+] Port scanning..."
dmitry -p -b "$TARGET" > "$OUTPUT_DIR/ports.txt" 2>&1

# Generate report
echo "=== DMitry Recon Report ===" > "$OUTPUT_DIR/report.md"
echo "Target: $TARGET" >> "$OUTPUT_DIR/report.md"
echo "Date: $(date)" >> "$OUTPUT_DIR/report.md"
echo "" >> "$OUTPUT_DIR/report.md"
echo "## Subdomains Found:" >> "$OUTPUT_DIR/report.md"
grep "Found" "$OUTPUT_DIR/subdomains.txt" >> "$OUTPUT_DIR/report.md" 2>/dev/null
echo "" >> "$OUTPUT_DIR/report.md"
echo "## Emails Found:" >> "$OUTPUT_DIR/report.md"
grep "@" "$OUTPUT_DIR/emails.txt" >> "$OUTPUT_DIR/report.md" 2>/dev/null

echo "[+] Scan complete. Results in $OUTPUT_DIR/"
```

---

## Chapter 5: Advanced Usage

### Port Scanning
```bash
# Scan specific ports
dmitry -p -t 21,22,80,443,8080 target.com

# Banner grabbing
dmitry -p -b target.com
```

### Recursive Subdomain Search
```bash
# Combine with DNS brute-forcing
dmitry -n -f target.com
```

### Docker Deployment
```bash
# Run DMitry in Docker
docker run -it --rm \
  -v $(pwd)/results:/results \
  dmitry:latest -winsepfb -o /results/dmitry.txt target.com
```

### Combining with Other Tools
```bash
# DMitry → Nmap
dmitry -n target.com | grep "Found" | awk '{print $3}' | \
    xargs -I{} nmap -sV -p- {}

# DMitry → TheHarvester
dmitry -e target.com | grep "@" | sort -u > dmitry_emails.txt
theharvester -d target.com -b google 2>/dev/null | grep "@" > harvester_emails.txt
sort -u dmitry_emails.txt harvester_emails.txt > all_emails.txt
```

### DMitry Output Analysis
```bash
# Parse DMitry output for quick summary
dmitry -winsepfb target.com -o dmitry_full.txt

echo "=== DMitry Summary ==="
echo "Subdomains: $(grep -c "Found DNS" dmitry_full.txt)"
echo "Emails: $(grep -c "@" dmitry_full.txt)"
echo "Open Ports: $(grep -c "Open Port" dmitry_full.txt)"
```

### Automated Recon Script
```bash
#!/bin/bash
# Comprehensive recon using DMitry and other tools
TARGET=$1
WORKDIR="recon_$(date +%Y%m%d_%H%M%S)"
mkdir -p "$WORKDIR"

echo "[*] Starting recon on: $TARGET"

# Phase 1: DMitry
echo "[+] Phase 1: DMitry scan"
dmitry -winsepfb "$TARGET" -o "$WORKDIR/dmitry.txt" 2>&1

# Phase 2: Extract subdomains
echo "[+] Phase 2: Extracting subdomains"
grep "Found DNS" "$WORKDIR/dmitry.txt" | awk '{print $NF}' > "$WORKDIR/subdomains.txt"

# Phase 3: Nmap scan
echo "[+] Phase 3: Nmap scan"
nmap -sV -sC -oN "$WORKDIR/nmap.txt" "$TARGET" 2>&1

echo "[+] Recon complete. Results in $WORKDIR/"
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: Initial Recon
```bash
# Step 1: Gather all info
dmitry -winsepfb -o full_recon.txt target.com

# Step 2: Review findings
echo "=== Subdomains ===" 
grep "Subdomain" full_recon.txt

echo "=== Emails ===" 
grep "Email" full_recon.txt

echo "=== Open Ports ==="
grep -A5 "Port" full_recon.txt
```

### Scenario 2: CTF Recon
```bash
# CTF: Find hidden information
dmitry -winsefb target.com | tee ctf_recon.txt

# Search for flags
grep -i "flag" ctf_recon.txt
grep -i "CTF" ctf_recon.txt
```

### Scenario 3: Email Harvesting
```bash
# Harvest emails for social engineering assessment
dmitry -e target.com | grep "@" | sort -u > emails.txt

# Verify emails
while read email; do
    echo "Checking: $email"
    ping -c 1 $(echo $email | cut -d@ -f2) 2>/dev/null && echo "  [+] Domain valid" || echo "  [-] Domain invalid"
done < emails.txt
```

### Scenario 4: Comprehensive Host Analysis
```bash
# Full analysis of a target host
TARGET="target.com"
OUTPUT="analysis_$(date +%Y%m%d).txt"

echo "=== DMitry Comprehensive Analysis ===" > "$OUTPUT"
echo "Target: $TARGET" >> "$OUTPUT"
echo "Date: $(date)" >> "$OUTPUT"
echo "" >> "$OUTPUT"

echo "[*] Running whois lookup..."
dmitry -w "$TARGET" >> "$OUTPUT" 2>&1

echo "[*] Running DNS lookup..."
dmitry -f "$TARGET" >> "$OUTPUT" 2>&1

echo "[*] Enumerating subdomains..."
dmitry -n "$TARGET" >> "$OUTPUT" 2>&1

echo "[*] Harvesting emails..."
dmitry -e "$TARGET" >> "$OUTPUT" 2>&1

echo "[*] Port scanning..."
dmitry -p -b "$TARGET" >> "$OUTPUT" 2>&1

echo "[+] Analysis complete: $OUTPUT"
```

---

## Chapter 7: Detection & Defense

### Detection
- Sequential port scan patterns
- Whois query patterns
- Subdomain enumeration requests
- DNS lookup patterns
- Banner grabbing attempts

### IDS/IPS Signatures
```
# Snort rule for DMitry port scanning
alert tcp any any -> any any (msg:"DMitry Port Scan"; \
  flags:S; threshold:type both, track by_src, count 20, seconds 60; \
  classtype:reconnaissance; sid:1000001; rev:1;)

# Suricata rule for DNS enumeration
alert dns any any -> any any (msg:"DMitry Subdomain Enumeration"; \
  dns.query; content:"target.com"; \
  classtype:reconnaissance; sid:1000002; rev:1;)
```

### Mitigation
- Rate limit whois lookups
- Monitor for port scan activity
- Use IDS to detect enumeration
- Implement DNS query logging
- Deploy honeypots to detect reconnaissance
- Set up alerts for unusual DNS patterns

### Blue Team Response
1. Monitor for sequential port scan patterns
2. Log and alert on bulk whois queries
3. Implement progressive delays for DNS lookups
4. Deploy network segmentation to limit reconnaissance
5. Use threat intelligence feeds to detect known scanner IPs
6. Set up canary tokens to detect reconnaissance activity

---

## Chapter 8: Troubleshooting

### Common Errors

#### "Connection refused"
**Solution:** Check target accessibility and firewall rules.

#### "No subdomains found"
**Solution:** Try DNS brute-forcing with dnsenum or amass.

#### "Port scan timeout"
**Solution:** Increase timeout or reduce port range.

#### "Permission denied"
**Solution:** Some features require root. Use `sudo`.

### Performance Tips
- Use `-F` to filter results
- Limit port range with `-t`
- Use `-o` to save results to file
- Combine with faster tools for large targets

### FAQ
**Q: How is DMitry different from Nmap?**
A: DMitry focuses on OSINT (whois, email, subdomains) while Nmap focuses on port scanning.

**Q: Can DMitry scan large networks?**
A: DMitry is designed for single hosts. Use Nmap or Masscan for network scanning.

**Q: How do I update DMitry?**
A: Run `sudo apt update && sudo apt upgrade dmitry` or rebuild from source.

---

## Resources

### Official Documentation
- [DMitry GitHub](https://github.com/jmk-foofus/DeepMagic)
- [Foofus.net](https://www.foofus.net)

### Related Tools
- **TheHarvester** — Email and subdomain harvester
- **DNSenum** — DNS enumeration tool
- **Amass** — Network mapping of attack surfaces
- **Nmap** — Network scanner

### Practice
- Use with authorized penetration testing engagements
- Practice on intentionally vulnerable applications
- Join OSINT communities for tips and techniques
