---
tool_name: "sherlock"
display_name: "Sherlock"
mitre_tactic: "reconnaissance"
mitre_tactic_id: "TA0043"
mitre_subcategory: "identity_information"
install_command: "sudo apt install sherlock"
version: "0.15.0"
github_repo: "https://github.com/sherlock-project/sherlock"
kali_tools_page: "https://www.kali.org/tools/sherlock/"
difficulty_level: "beginner"
requires_root: false
gui_available: false
tags: ["osint", "username", "social-media", "identity"]
related_tools: ["maigret", "whatsmyname", "namechk"]
---

# Sherlock

## Chapter 1: Introduction & Overview

### What is Sherlock?
Sherlock is an open-source OSINT tool that hunts down social media accounts by username across over 400 social networks. Given a username, Sherlock checks its presence on numerous platforms and reports which ones have matching profiles.

### History & Background
- **Created:** 2019 by Siddharth Dushyant Garg
- **Maintained:** Sherlock Project (community-driven)
- **License:** MIT
- **Language:** Python 3
- **Purpose:** Username enumeration across social networks for OSINT investigations
- **Modules:** 400+ social network checks
- **Architecture:** Asynchronous HTTP requests for speed

### When to Use This Tool
- **Identity investigations:** Find all social media profiles for a target username
- **Threat intelligence:** Map a threat actor's online presence
- **Brand monitoring:** Check if usernames are taken across platforms
- **Social engineering assessments:** Gather intel on a target
- **Background checks:** Verify someone's online footprint
- **CTF challenges:** Find flags hidden in social media profiles

### When NOT to Use This Tool
- Without proper authorization for the investigation
- To harass, stalk, or dox individuals
- For unauthorized access to accounts
- For any illegal activity

### Legal and Ethical Considerations
- Sherlock only checks PUBLICLY available profile pages
- It does not attempt to access private accounts
- Use only with proper authorization and for legitimate purposes
- Respect platform terms of service
- Document your authorization for all investigations
- Consider privacy implications of data collection

### Key Concepts
- **Username enumeration:** Checking if a username exists across multiple platforms
- **OSINT:** Open Source Intelligence — gathering information from publicly available sources
- **False positive:** A platform reporting a match when no real account exists
- **Rate limiting:** Platforms may block excessive requests
- **Profile page:** Publicly accessible page showing user information
- **Claimed username:** A username that has been registered on a platform

---

## Chapter 2: Installation & Setup

### System Requirements
- **Python:** 3.6 or higher
- **RAM:** 512 MB minimum
- **Disk Space:** 50 MB
- **Network:** Internet connection required
- **Privileges:** Normal user (no root required)
- **OS:** Linux, macOS, Windows

### Installation Methods

#### Method 1: APT (Recommended)
```bash
sudo apt update
sudo apt install sherlock
```
**Expected output:**
```
Reading package lists... Done
Building dependency tree... Done
The following NEW packages will be installed:
  sherlock
0 upgraded, 1 newly installed, 0 to remove and 0 not upgraded.
Need to get 1.2 MB of archives.
After this operation, 4.5 MB of additional disk space will be used.
Get:1 http://kali.download/kali kali-rolling/main amd64 sherlock all 0.15.0-1kali1 [1.2 MB]
Fetched 1.2 MB in 0.5s (2.4 MB/s)
Selecting previously unselected package sherlock.
(Reading database ... 1234567 files and directories currently installed.)
Preparing to unpack .../sherlock_0.15.0-1kali1_all.deb ...
Unpacking sherlock (0.15.0-1kali1) ...
Setting up sherlock (0.15.0-1kali1) ...
```

#### Method 2: From Source (Latest)
```bash
git clone https://github.com/sherlock-project/sherlock.git
cd sherlock
pip3 install -r requirements.txt
```
**Expected output:**
```
Cloning into 'sherlock'...
remote: Enumerating objects: 5678, done.
remote: Counting objects: 100% (567/567), done.
remote: Compressing objects: 100% (234/234), done.
remote: Total 5678 (delta 345), reused 500 (delta 300)
Receiving objects: 100% (5678/5678), 2.34 MiB | 3.21 MiB/s, done.
Resolving deltas: 100% (345/345), done.
Collecting requests>=2.28.0
  Downloading requests-2.28.1-py3-none-any.whl (62 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 62.3/62.3 kB 1.8 MB/s eta 0:00:00
Installing collected packages: requests, urllib3, certifi, idna, chardet
Successfully installed requests-2.28.1 urllib3-1.26.12 certifi-2022.6.15 idna-3.4 chardet-5.0.0
```

#### Method 3: pip
```bash
pip3 install sherlock-project
```
**Expected output:**
```
Collecting sherlock-project
  Downloading sherlock_project-0.15.0-py3-none-any.whl (15 kB)
Installing collected packages: sherlock-project
Successfully installed sherlock-project-0.15.0
```

### Configuration
- Sherlock uses a `tor.json` file for Tor proxy configuration
- Default timeout: 30 seconds per request
- Configurable via command-line arguments
- Site database: `data.json` in the sherlock data directory

### Verification
```bash
sherlock --version
```
**Expected output:**
```
sherlock 0.15.0
```

---

## Chapter 3: Basic Usage

### First Run
```bash
sherlock john_doe
```
**Output:**
```
[*] Checking username john_doe on:
[+] Instagram: https://instagram.com/john_doe
[+] Twitter: https://twitter.com/john_doe
[+] GitHub: https://github.com/john_doe
[+] Reddit: https://reddit.com/user/john_doe
[+] LinkedIn: https://linkedin.com/in/john_doe
[*] Search completed with 5 results
```

### Essential Commands

#### Basic Username Search
```bash
sherlock target_username
```
**Explanation:** Searches the default list of ~400 sites for the given username.

#### Search with Timeout
```bash
sherlock --timeout 15 target_username
```
**Explanation:** Sets a 15-second timeout per site request.

#### Verbose Output
```bash
sherlock --verbose target_username
```
**Explanation:** Shows detailed output including sites being checked and errors.

#### Output to File
```bash
sherlock --output results.txt target_username
```
**Explanation:** Saves results to a text file.

#### JSON Output
```bash
sherlock --json results.json target_username
```
**Explanation:** Saves results in JSON format for programmatic use.

#### CSV Output
```bash
sherlock --csv target_username
```
**Explanation:** Outputs results in CSV format for spreadsheet import.

#### Search Specific Sites
```bash
sherlock -s instagram,twitter,github target_username
```
**Explanation:** Only checks the specified platforms.

#### List Supported Sites
```bash
sherlock --list
```
**Explanation:** Displays all 400+ supported social networks.

### Complete Flags Reference

#### Target Specification
| Flag | Description | Example |
|------|-------------|---------|
| positional | Username to search | `sherlock john_doe` |
| `--usernames` | File with usernames (one per line) | `sherlock --usernames list.txt` |
| `--csv` | Write output as CSV | `sherlock --csv target` |

#### Output Options
| Flag | Description | Example |
|------|-------------|---------|
| `-o`, `--output` | Output file path | `sherlock -o results.txt target` |
| `--json` | JSON output file | `sherlock --json out.json target` |
| `--csv` | CSV output | `sherlock --csv target` |
| `--print-found` | Only print found results | `sherlock --print-found target` |
| `--no-color` | Disable colored output | `sherlock --no-color target` |

#### Network Options
| Flag | Description | Example |
|------|-------------|---------|
| `--timeout` | Request timeout in seconds | `sherlock --timeout 10 target` |
| `--proxy` | Proxy URL | `sherlock --proxy socks5://127.0.0.1:9050 target` |
| `--tor` | Use Tor network | `sherlock --tor target` |
| `--unique-tor` | New Tor circuit per request | `sherlock --unique-tor target` |

#### Site Selection
| Flag | Description | Example |
|------|-------------|---------|
| `-s`, `--sites` | Site filter (comma-separated) | `sherlock -s instagram,twitter target` |
| `--print-all` | Print all sites checked | `sherlock --print-all target` |
| `--list` | List all supported sites | `sherlock --list` |
| `-w`, `--wordlist` | Custom wordlist of sites | `sherlock -w custom.txt target` |

#### General
| Flag | Description | Example |
|------|-------------|---------|
| `-v`, `--verbose` | Verbose output | `sherlock -v target` |
| `--version` | Show version | `sherlock --version` |
| `-d`, `--debug` | Debug mode | `sherlock -d target` |

---

## Chapter 4: Intermediate Usage

### Bash Scripting & Automation

#### Search Multiple Usernames
```bash
#!/bin/bash
# multi_search.sh - Search multiple usernames
USERNAMES=("john_doe" "jane_smith" "admin")
OUTPUT_DIR="results/$(date +%Y%m%d)"
mkdir -p "$OUTPUT_DIR"

for user in "${USERNAMES[@]}"; do
    echo "[*] Searching: $user"
    sherlock --output "$OUTPUT_DIR/${user}.txt" --json "$OUTPUT_DIR/${user}.json" "$user"
    sleep 5  # Rate limiting
done
echo "[+] All searches complete. Results in $OUTPUT_DIR"
```

#### Filter Results
```bash
# Search and filter for specific platforms
sherlock --print-found target 2>/dev/null | grep -E "(Instagram|Twitter|Facebook)" > filtered.txt
```

#### Generate Username Variations
```bash
#!/bin/bash
# Generate and search username variations
FIRST="john"
LAST="doe"
VARIATIONS=("${FIRST}.${LAST}" "${FIRST}_${LAST}" "${FIRST}${LAST}" "${FIRST:0:1}${LAST}")

for user in "${VARIATIONS[@]}"; do
    echo "[*] Checking: $user"
    sherlock --print-found "$user" 2>/dev/null | grep -E "^\[" >> username_report.txt
done
```

### Python Scripting

#### Programmatic Usage
```python
import subprocess
import json

def search_username(username):
    """Search for a username across platforms."""
    cmd = ["sherlock", "--json", "-", username]
    result = subprocess.run(cmd, capture_output=True, text=True)
    
    if result.returncode == 0:
        try:
            return json.loads(result.stdout)
        except json.JSONDecodeError:
            return None
    return None

# Example usage
results = search_username("target_user")
if results:
    for site, data in results.items():
        if data.get("status") == "Claimed":
            print(f"[+] {site}: {data['url_user']}")
```

#### Using sherlock as a library
```python
from sherlock import sherlock

# Direct API usage (advanced)
query = sherlock.Sherlock()
query.main(["target_username"])
```

#### Advanced Python Script
```python
import subprocess
import json
import csv
from datetime import datetime

class SherlockAnalyzer:
    def __init__(self):
        self.results = {}
    
    def search(self, username, sites=None):
        cmd = ["sherlock", "--json", "-"]
        if sites:
            cmd.extend(["-s", ",".join(sites)])
        cmd.append(username)
        
        result = subprocess.run(cmd, capture_output=True, text=True)
        if result.returncode == 0 and result.stdout:
            try:
                self.results[username] = json.loads(result.stdout)
            except json.JSONDecodeError:
                self.results[username] = {}
    
    def get_found_sites(self, username):
        if username not in self.results:
            return []
        return [site for site, data in self.results[username].items() 
                if data.get("status") == "Claimed"]
    
    def export_report(self, filename):
        with open(filename, 'w') as f:
            f.write("# Sherlock OSINT Report\n\n")
            f.write(f"Generated: {datetime.now().isoformat()}\n\n")
            for username, data in self.results.items():
                f.write(f"## Username: {username}\n\n")
                found = self.get_found_sites(username)
                f.write(f"Found on {len(found)} sites:\n\n")
                for site in found:
                    url = data[site].get("url_user", "N/A")
                    f.write(f"- {site}: {url}\n")

# Usage
analyzer = SherlockAnalyzer()
analyzer.search("target_user", ["instagram", "twitter", "github"])
analyzer.export_report("sherlock_report.md")
```

### Tool Chaining

#### Sherlock + Recon-ng
```bash
# Get usernames from Recon-ng and check with Sherlock
recon-ng -x "modules/recon/profiles/profiles_all/site.py" -C "EXECUTE" 2>/dev/null | \
    grep -oP '"username":\s*"\K[^"]+' | \
    xargs -I{} sherlock --output "sherlock_{}.txt" {}
```

#### Sherlock + TheHarvester
```bash
# Extract names from TheHarvester and generate username variations
theharvester -d target.com -b google 2>/dev/null | \
    grep -oP '\b[A-Za-z]+\b' | sort -u | \
    xargs -I{} sherlock --output "sherlock_{}.txt" {}
```

#### Sherlock + Maigret
```bash
# Use both tools for comprehensive coverage
sherlock --output sherlock_results.txt target_user
maigret --json maigret_results.json target_user
```

---

## Chapter 5: Advanced Usage

### Custom Site Configuration
Sherlock uses a `data.json` file to define sites. You can add custom sites:

```json
{
  "CustomSite": {
    "url": "https://example.com/user/{}",
    "errorType": "message",
    "errorMsg": "User not found",
    "regexCheck": "^[a-zA-Z0-9_]{3,20}$"
  }
}
```

### Tor Integration
```bash
# Route all traffic through Tor
sherlock --tor --unique-tor target_username
```

### Proxy Configuration
```bash
# SOCKS5 proxy
sherlock --proxy socks5://127.0.0.1:1080 target

# HTTP proxy
sherlock --proxy http://proxy:8080 target
```

### Bulk Username Checking
```bash
# Create usernames.txt with one username per line
sherlock --usernames usernames.txt --output bulk_results.txt --csv
```

### Regex Filtering
```bash
# Only show results matching specific patterns
sherlock target 2>/dev/null | grep -E "https://(instagram|twitter|github)\.com/"
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: OSINT Investigation
```bash
# Step 1: Gather target username from intelligence
TARGET="suspicious_user"

# Step 2: Run Sherlock with Tor for anonymity
sherlock --tor --json results.json --output results.txt "$TARGET"

# Step 3: Analyze results
cat results.txt | grep -E "(LinkedIn|GitHub|Twitter)" > high_value.txt

# Step 4: Deep dive on found profiles
while read url; do
    curl -s "$url" | grep -oP '(?<=<title>)[^<]+' 
done < high_value.txt
```

### Scenario 2: CTF Challenge
```bash
# CTF: Find the flag by discovering a user's social media
# Hint: "The admin uses the same username everywhere"

# Step 1: Get the target username from the challenge
TARGET="ctf_admin"

# Step 2: Run Sherlock
sherlock --print-found "$TARGET"

# Step 3: Check each found profile for hidden flags
# Look in bios, pinned posts, about sections
```

### Scenario 3: Brand Security Assessment
```bash
# Check if brand username is taken across platforms
BRAND="mycompany"

# Step 1: Check all platforms
sherlock --output brand_check.txt --json brand_check.json "$BRAND"

# Step 2: Generate report
echo "=== Brand Username Security Report ===" > report.md
echo "Date: $(date)" >> report.md
echo "Target: $BRAND" >> report.md
echo "" >> report.md
echo "Platforms with matching profiles:" >> report.md
grep -E "^\[" brand_check.txt >> report.md
```

---

## Chapter 7: Detection & Defense

### How Defenders Detect Sherlock
- **Request patterns:** Sherlock makes rapid requests to multiple platforms
- **User-Agent:** Default Python requests User-Agent
- **Volume:** Hundreds of requests in short timeframe
- **No session cookies:** Each request appears as a new visitor
- **Source IPs:** Requests from known VPN/Tor exit nodes

### IDS/IPS Signatures
```
alert http any any -> any any (msg:"Sherlock Username Enumeration"; \
  content:"User-Agent"; http_header; content:"python-requests"; \
  classtype:web-application-attack; sid:1000001; rev:1;)
```

### Mitigation Strategies
- **Rate limiting:** Limit requests per IP from username lookup endpoints
- **CAPTCHA:** Implement CAPTCHA after multiple lookups
- **Account enumeration prevention:** Return generic responses for non-existent users
- **WAF rules:** Block automated access to profile pages
- **User-Agent filtering:** Block known scraper User-Agents

### Blue Team Response
1. Monitor for rapid sequential profile page access
2. Log and alert on bulk username lookups
3. Implement progressive delays for repeated lookups
4. Consider requiring authentication for profile views
5. Deploy CAPTCHA on sensitive endpoints
6. Monitor for Tor exit node connections

---

## Chapter 8: Troubleshooting

### Common Errors

#### "Connection timeout"
```
[!] Timeout: site.com - Request timed out
```
**Solution:** Increase timeout with `--timeout 30` or check network connectivity.

#### "SSL Error"
```
[SSL: CERTIFICATE_VERIFY_FAILED]
```
**Solution:** Update certificates or use `--no-verify-ssl` (not recommended for production).

#### "Rate limited"
```
[!] Rate limited: site.com
```
**Solution:** Add delays between requests or use Tor with `--unique-tor`.

#### "Module not found"
```
ModuleNotFoundError: No module named 'sherlock'
```
**Solution:** Reinstall with `pip3 install sherlock-project` or use APT.

### Performance Tips
- Use `--timeout 10` for faster (but less reliable) results
- Use `--sites` to limit to specific platforms
- Use `--print-found` to reduce output noise
- Run with `--tor` for anonymity (slower but private)
- Use `--json` for programmatic processing

### FAQ
**Q: How accurate are results?**
A: Sherlock reports if a profile EXISTS, not if it belongs to your target. Verify manually.

**Q: Can Sherlock access private profiles?**
A: No. Sherlock only checks publicly visible profile pages.

**Q: How do I add new sites?**
A: Edit the `data.json` file in the sherlock data directory.

**Q: What's the difference between Sherlock and Maigret?**
A: Maigret has more sites and better error handling, but Sherlock is simpler and faster.

**Q: How do I update Sherlock?**
A: Run `sudo apt update && sudo apt upgrade sherlock` or `git pull && pip3 install -r requirements.txt`.

---

## Resources

### Official Documentation
- [Sherlock GitHub](https://github.com/sherlock-project/sherlock)
- [Sherlock Wiki](https://github.com/sherlock-project/sherlock/wiki)
- [Supported Sites List](https://github.com/sherlock-project/sherlock/blob/master/data.json)

### Related Tools
- **Maigret** — Extended username checker with more sites
- **WhatsMyName** — Web-based username enumeration
- **Namechk** — Username and domain availability checker
- **SpiderFoot** — Comprehensive OSINT automation

### Practice
- Use with authorized OSINT labs
- Practice on your own accounts across platforms
- Join OSINT communities for tips and techniques
