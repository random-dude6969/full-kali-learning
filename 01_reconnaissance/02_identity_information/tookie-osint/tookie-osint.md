---
tool_name: "tookie-osint"
display_name: "Tookie OSINT"
mitre_tactic: "reconnaissance"
mitre_tactic_id: "TA0043"
mitre_subcategory: "identity_information"
install_command: "sudo apt install tookie-osint"
version: "1.0"
github_repo: "https://github.com/soxoj/tookie-osint"
kali_tools_page: "https://www.kali.org/tools/tookie-osint/"
difficulty_level: "beginner"
requires_root: false
gui_available: false
tags: ["osint", "social-media", "identity", "username"]
related_tools: ["sherlock", "maigret", "whatsmyname"]
---

# Tookie OSINT

## Chapter 1: Introduction & Overview

### What is Tookie OSINT?
Tookie OSINT is a comprehensive OSINT tool for gathering intelligence about users across social networks and websites. It provides automated information gathering from multiple sources including social media, email services, and public databases.

### History & Background
- **Created:** 2021 by soxoj
- **Maintained:** Community contributors
- **License:** MIT
- **Language:** Python 3
- **Purpose:** Automated OSINT data collection across multiple platforms
- **Architecture:** Modular design with async HTTP requests
- **Features:** Username, email, and phone number lookup

### When to Use This Tool
- **User profiling:** Gather comprehensive info about a target
- **Social media analysis:** Map online presence across platforms
- **Email investigation:** Trace email addresses to social accounts
- **Threat intelligence:** Build profiles of potential threats
- **Background research:** Verify identities and claims
- **CTF challenges:** Find flags hidden in social media profiles

### When NOT to Use This Tool
- Without proper authorization
- For stalking, harassment, or doxing
- To access unauthorized information
- For any illegal activity

### Legal and Ethical Considerations
- Only accesses publicly available information
- Use with proper authorization for investigations
- Document all activities for legal compliance
- Respect platform terms of service
- Consider privacy implications of data collection
- Obtain written permission before investigating individuals

### Key Concepts
- **OSINT:** Open Source Intelligence gathering
- **Digital footprint:** Trail of online activity left by users
- **Correlation:** Linking data from multiple sources
- **Metadata:** Data about data (timestamps, locations, etc.)
- **Username enumeration:** Checking username existence across platforms
- **Email header analysis:** Extracting information from email headers

---

## Chapter 2: Installation & Setup

### System Requirements
- **Python:** 3.7+
- **RAM:** 512 MB
- **Disk Space:** 100 MB
- **Network:** Internet connection
- **Privileges:** Normal user (no root required)

### Installation Methods

#### Method 1: APT
```bash
sudo apt update
sudo apt install tookie-osint
```
**Expected output:**
```
Reading package lists... Done
Building dependency tree... Done
The following NEW packages will be installed:
  tookie-osint
0 upgraded, 1 newly installed, 0 to remove and 0 not upgraded.
Need to get 856 kB of archives.
After this operation, 3.2 MB of additional disk space will be used.
Get:1 http://kali.download/kali kali-rolling/main amd64 tookie-osint all 1.0-1kali1 [856 kB]
Fetched 856 kB in 0.3s (2.8 MB/s)
Selecting previously unselected package tookie-osint.
(Reading database ... 1234567 files and directories currently installed.)
Preparing to unpack .../tookie-osint_1.0-1kali1_all.deb ...
Unpacking tookie-osint (1.0-1kali1) ...
Setting up tookie-osint (1.0-1kali1) ...
```

#### Method 2: From Source
```bash
git clone https://github.com/soxoj/tookie-osint.git
cd tookie-osint
pip3 install -r requirements.txt
```
**Expected output:**
```
Cloning into 'tookie-osint'...
remote: Enumerating objects: 3456, done.
remote: Counting objects: 100% (345/345), done.
remote: Compressing objects: 100% (123/123), done.
remote: Total 3456 (delta 234), reused 300 (delta 200)
Receiving objects: 100% (3456/3456), 1.23 MiB | 2.15 MiB/s, done.
Resolving deltas: 100% (234/234), done.
Collecting aiohttp>=3.8.0
  Downloading aiohttp-3.8.1-cp39-cp39-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (1.2 MB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 1.2/1.2 MB 3.45 MB/s eta 0:00:00
Installing collected packages: aiohttp, aiosignal, frozenlist, attrs, multidict, yarl, async-timeout
Successfully installed aiohttp-3.8.1 aiosignal-1.2.1 frozenlist-1.3.1 attrs-22.1.0 multidict-6.0.2 yarl-1.8.1 async-timeout-4.0.2
```

#### Method 3: pip
```bash
pip3 install tookie-osint
```

### Configuration
- Default settings work out of the box
- API keys can be set via environment variables
- Tor integration available for anonymity

### Verification
```bash
tookie --version
```
**Expected output:**
```
Tookie OSINT 1.0
```

---

## Chapter 3: Basic Usage

### First Run
```bash
tookie osint username
```
**Output:**
```
[*] Starting OSINT for: username
[*] Checking social networks...
[+] Found: Instagram - https://instagram.com/username
[+] Found: Twitter - https://twitter.com/username
[+] Found: GitHub - https://github.com/username
[*] OSINT complete. 3 results found.
```

### Essential Commands

#### Basic OSINT Search
```bash
tookie osint target_username
```
**Explanation:** Searches across multiple platforms for the given username.

#### Email Lookup
```bash
tookie email target@email.com
```
**Explanation:** Investigates an email address for associated accounts.

#### Verbose Mode
```bash
tookie osint --verbose target_username
```
**Explanation:** Shows detailed progress and findings.

#### Output to File
```bash
tookie osint --output results.txt target_username
```
**Explanation:** Saves results to a text file.

#### JSON Output
```bash
tookie osint --json results.json target_username
```
**Explanation:** Saves results in JSON format for programmatic use.

#### Quiet Mode
```bash
tookie osint --quiet target_username
```
**Explanation:** Minimal output, only shows results.

### Complete Flags Reference

#### Target Specification
| Flag | Description | Example |
|------|-------------|---------|
| positional | Target username/email | `tookie osint target` |
| `--username` | Username to search | `tookie osint --username target` |
| `--email` | Email to investigate | `tookie email target@email.com` |

#### Output Options
| Flag | Description | Example |
|------|-------------|---------|
| `-o`, `--output` | Output file | `tookie osint -o results.txt target` |
| `--json` | JSON output | `tookie osint --json results.json target` |
| `--csv` | CSV output | `tookie osint --csv target` |
| `-v`, `--verbose` | Verbose output | `tookie osint -v target` |
| `-q`, `--quiet` | Quiet mode | `tookie osint -q target` |

#### Network Options
| Flag | Description | Example |
|------|-------------|---------|
| `--timeout` | Request timeout | `tookie osint --timeout 15 target` |
| `--proxy` | Use proxy | `tookie osint --proxy socks5://127.0.0.1:9050 target` |
| `--tor` | Use Tor network | `tookie osint --tor target` |

#### Site Selection
| Flag | Description | Example |
|------|-------------|---------|
| `--site` | Filter specific sites | `tookie osint --site instagram target` |
| `--no-site` | Exclude specific sites | `tookie osint --no-site facebook target` |

#### General
| Flag | Description | Example |
|------|-------------|---------|
| `--help` | Show help | `tookie osint --help` |
| `--version` | Show version | `tookie --version` |
| `--debug` | Debug mode | `tookie osint --debug target` |

---

## Chapter 4: Intermediate Usage

### Bash Scripting

#### Automated Multi-User Search
```bash
#!/bin/bash
# osint_batch.sh
USERS=("target1" "target2" "target3")
REPORT="osint_report_$(date +%Y%m%d).txt"

echo "=== OSINT Report ===" > "$REPORT"
echo "Date: $(date)" >> "$REPORT"
echo "" >> "$REPORT"

for user in "${USERS[@]}"; do
    echo "[*] Investigating: $user"
    echo "--- $user ---" >> "$REPORT"
    tookie osint "$user" >> "$REPORT" 2>&1
    echo "" >> "$REPORT"
    sleep 3
done

echo "[+] Report saved to $REPORT"
```

#### Email Investigation Pipeline
```bash
# Investigate email and cross-reference
tookie email suspect@email.com --json | jq '.social_media[] | .url' | \
    xargs -I{} curl -s -I {} | grep -i "location:"
```

### Python Scripting

#### Programmatic Usage
```python
import subprocess
import json

def investigate_target(target, output_format="json"):
    """Run Tookie OSINT on a target."""
    cmd = ["tookie", "osint", f"--{output_format}", "-", target]
    result = subprocess.run(cmd, capture_output=True, text=True)
    return result.stdout

# Example
results = investigate_target("suspicious_user")
print(results)
```

#### Advanced Python Script
```python
import subprocess
import json
import csv
from datetime import datetime

class TookieAnalyzer:
    def __init__(self):
        self.results = {}
    
    def search_username(self, username):
        cmd = ["tookie", "osint", "--json", "-", username]
        result = subprocess.run(cmd, capture_output=True, text=True)
        if result.returncode == 0 and result.stdout:
            try:
                self.results[username] = json.loads(result.stdout)
            except json.JSONDecodeError:
                self.results[username] = {}
    
    def search_email(self, email):
        cmd = ["tookie", "email", "--json", "-", email]
        result = subprocess.run(cmd, capture_output=True, text=True)
        if result.returncode == 0 and result.stdout:
            try:
                self.results[email] = json.loads(result.stdout)
            except json.JSONDecodeError:
                self.results[email] = {}
    
    def get_social_media(self, target):
        if target not in self.results:
            return []
        return self.results[target].get('social_media', [])
    
    def export_report(self, filename):
        with open(filename, 'w') as f:
            f.write("# Tookie OSINT Report\n\n")
            f.write(f"Generated: {datetime.now().isoformat()}\n\n")
            for target, data in self.results.items():
                f.write(f"## Target: {target}\n\n")
                social = self.get_social_media(target)
                f.write(f"Found {len(social)} social media profiles:\n\n")
                for profile in social:
                    f.write(f"- {profile.get('site', 'Unknown')}: {profile.get('url', 'N/A')}\n")

# Usage
analyzer = TookieAnalyzer()
analyzer.search_username("target_user")
analyzer.export_report("tookie_report.md")
```

### Tool Chaining

#### Sherlock + Tookie OSINT
```bash
# Combine results from both tools
sherlock --json sherlock.json target
tookie osint --json tookie.json target

# Merge and deduplicate
jq -s '.[0] + .[1] | to_entries | unique_by(.key) | from_entries' \
    sherlock.json tookie.json > combined.json
```

#### Tookie + Amass
```bash
# Combine subdomain findings
tookie osint target.com --json | jq -r '.subdomains[]?' > tookie_subs.txt
amass enum -d target.com 2>/dev/null > amass_subs.txt
sort -u tookie_subs.txt amass_subs.txt > all_subs.txt
```

---

## Chapter 5: Advanced Usage

### Custom Configuration
Tookie OSINT supports custom configuration for API keys and rate limits:

```bash
# Set API keys for enhanced results
export SHODAN_API_KEY="your_key"
export HUNTER_API_KEY="your_key"
export VIRUSTOTAL_API_KEY="your_key"
```

### Tor Integration
```bash
# Anonymize all requests
tookie osint --tor --timeout 30 target
```

### Bulk Processing
```bash
# Process multiple targets from file
while read target; do
    tookie osint --json "results_${target}.json" "$target"
    sleep 5
done < targets.txt
```

### Docker Deployment
```bash
# Run Tookie OSINT in Docker
docker run -it --rm \
  -v $(pwd)/results:/results \
  tookie-osint:latest osint target_user
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: Threat Actor Profiling
```bash
# Step 1: Get username from threat intel
TARGET="threat_actor_handle"

# Step 2: Full OSINT sweep
tookie osint --json --output threat_actor.json "$TARGET"

# Step 3: Extract key findings
jq '.social_media | length' threat_actor.json  # Count platforms
jq '.emails[]?' threat_actor.json              # Found emails
jq '.phone_numbers[]?' threat_actor.json       # Found phones

# Step 4: Generate report
echo "=== Threat Actor Profile ===" > profile.md
echo "Username: $TARGET" >> profile.md
echo "Social Media Accounts: $(jq '.social_media | length' threat_actor.json)" >> profile.md
echo "Email Addresses: $(jq '.emails | length' threat_actor.json)" >> profile.md
```

### Scenario 2: CTF OSINT Challenge
```bash
# CTF: "Find the flag hidden in the target's online presence"

# Step 1: Enumerate the target
tookie osint --verbose ctf_player

# Step 2: Check each found profile for clues
# Look for bios, posts, images with hidden info

# Step 3: Deep dive on specific platforms
tookie osint --site instagram ctf_player
tookie osint --site twitter ctf_player
```

### Scenario 3: Due Diligence Check
```bash
# Verify someone's claimed online presence
CLAIMED="claimed_username"

# Step 1: Check what actually exists
tookie osint --output verification.txt "$CLAIMED"

# Step 2: Compare with claimed profiles
diff <(grep -oP 'https?://\S+' verification.txt) claimed_profiles.txt

# Step 3: Generate discrepancy report
echo "=== Verification Report ===" > verification_report.md
echo "Target: $CLAIMED" >> verification_report.md
echo "Date: $(date)" >> verification_report.md
```

---

## Chapter 7: Detection & Defense

### How Defenders Detect Tookie OSINT
- **Request patterns:** Sequential access to profile pages
- **User-Agent:** Python-based User-Agent string
- **Volume:** Multiple platform checks in rapid succession
- **Geographic anomalies:** Requests from unusual locations
- **Timing patterns:** Regular intervals between requests

### Mitigation Strategies
- Implement rate limiting on profile endpoints
- Use CAPTCHA for repeated lookups
- Monitor for automated access patterns
- Log and alert on bulk profile page requests
- Deploy WAF rules to block known scraper User-Agents

### Blue Team Response
1. Track username lookup attempts per IP
2. Implement progressive delays
3. Require authentication for profile views
4. Use WAF to block automated access
5. Monitor for unusual geographic request patterns
6. Set up alerts for bulk enumeration attempts

---

## Chapter 8: Troubleshooting

### Common Errors

#### "Module not found"
```
ModuleNotFoundError: No module named 'requests'
```
**Solution:** `pip3 install -r requirements.txt`

#### "Connection refused"
```
ConnectionRefusedError: [Errno 111] Connection refused
```
**Solution:** Check proxy settings or network connectivity.

#### "Rate limited"
```
[!] Rate limit exceeded on site.com
```
**Solution:** Add `--timeout 30` or use `--tor`.

#### "JSON decode error"
```
json.decoder.JSONDecodeError: Expecting value: line 1 column 1
```
**Solution:** Check if the tool output is valid JSON, or use `--json` flag.

### Performance Tips
- Use `--timeout 10` for faster results
- Limit searches with `--sites` flag
- Use Tor for anonymity (slower)
- Cache results to avoid repeat lookups
- Use `--quiet` to reduce output noise

### FAQ
**Q: What's the difference between Tookie OSINT and Sherlock?**
A: Tookie has email and phone lookup features, while Sherlock focuses on username enumeration.

**Q: Can Tookie access private profiles?**
A: No. Tookie only checks publicly visible profile pages.

**Q: How do I add new sites?**
A: Edit the site configuration files in the Tookie data directory.

**Q: How do I update Tookie?**
A: Run `sudo apt update && sudo apt upgrade tookie-osint` or `git pull && pip3 install -r requirements.txt`.

---

## Resources

### Official Documentation
- [Tookie OSINT GitHub](https://github.com/soxoj/tookie-osint)
- [OSINT Framework](https://osintframework.com)

### Related Tools
- **Sherlock** — Username enumeration across 400+ sites
- **Maigret** — Extended username checker
- **SpiderFoot** — Comprehensive OSINT automation
- **Amass** — Network mapping of attack surfaces

### Practice
- Use with authorized OSINT labs
- Practice on your own accounts and assets
- Join OSINT communities for tips and techniques
