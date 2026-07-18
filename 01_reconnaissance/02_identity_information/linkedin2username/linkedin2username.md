---
tool_name: "linkedin2username"
display_name: "LinkedIn2Username"
mitre_tactic: "reconnaissance"
mitre_tactic_id: "TA0043"
mitre_subcategory: "identity_information"
install_command: "pip3 install linkedin2username"
version: "2.0"
github_repo: "https://github.com/initstring/linkedin2username"
kali_tools_page: "https://www.kali.org/tools/linkedin2username/"
difficulty_level: "beginner"
requires_root: false
gui_available: false
tags: ["osint", "linkedin", "username", "corporate"]
related_tools: ["sherlock", "hunter", "theharvester"]
---

# LinkedIn2Username

## Chapter 1: Introduction & Overview

### What is LinkedIn2Username?
LinkedIn2Username is an OSINT tool that scrapes company employee data from LinkedIn's public search results and converts names into potential username formats used by corporate IT systems. It helps penetration testers generate username lists for password spraying and brute-force attacks.

### History & Background
- **Created:** 2018 by initstring
- **Maintained:** Community contributors
- **License:** MIT
- **Language:** Python 3
- **Purpose:** Generate corporate username lists from LinkedIn data
- **Architecture:** Web scraping with Selenium/browser automation
- **Features:** Multiple username format generation

### When to Use This Tool
- **Penetration testing:** Generate username lists for password spraying
- **Red team operations:** Build target username databases
- **OSINT investigations:** Map corporate employee structures
- **Social engineering:** Identify naming conventions
- **CTF challenges:** Find employee username patterns

### When NOT to Use This Tool
- Without written authorization from the target organization
- For unauthorized access attempts
- For harassment or stalking
- For any illegal activity

### Legal and Ethical Considerations
- Requires LinkedIn account (free or premium)
- Use only with explicit authorization
- LinkedIn may restrict scraping activity
- Document authorization for all tests
- Stay within the defined scope of engagement
- Consider privacy implications of employee data collection

### Key Concepts
- **Username format:** Corporate naming convention (first.last, firstlast, flast, etc.)
- **Password spraying:** Testing common passwords against many accounts
- **Employee enumeration:** Identifying employees of an organization
- **LinkedIn scraping:** Extracting data from LinkedIn's public search results
- **Username generation:** Converting names into potential login credentials

---

## Chapter 2: Installation & Setup

### System Requirements
- **Python:** 3.6+
- **RAM:** 512 MB
- **Network:** LinkedIn access
- **Browser:** Chrome/Firefox (for Selenium)
- **Privileges:** Normal user (no root required)

### Installation

#### Method 1: pip
```bash
pip3 install linkedin2username
```
**Expected output:**
```
Collecting linkedin2username
  Downloading linkedin2username-2.0-py3-none-any.whl (15 kB)
Installing collected packages: linkedin2username
Successfully installed linkedin2username-2.0
```

#### Method 2: From Source
```bash
git clone https://github.com/initstring/linkedin2username.git
cd linkedin2username
pip3 install -r requirements.txt
```
**Expected output:**
```
Cloning into 'linkedin2username'...
remote: Enumerating objects: 1234, done.
remote: Counting objects: 100% (123/123), done.
remote: Compressing objects: 100% (45/45), done.
remote: Total 1234 (delta 89), reused 100 (delta 70)
Receiving objects: 100% (1234/1234), 456.78 KiB | 1.23 MiB/s, done.
Resolving deltas: 100% (89/89), done.
Collecting selenium>=4.0.0
  Downloading selenium-4.5.0-py3-none-any.whl (896 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 896.3/896.3 kB 2.45 MB/s eta 0:00:00
Installing collected packages: selenium, urllib3, certifi
Successfully installed selenium-4.5.0 urllib3-1.26.12 certifi-2022.6.15
```

### Verification
```bash
linkedin2username --help
```
**Expected output:**
```
LinkedIn2Username v2.0 - Generate corporate usernames from LinkedIn

Usage: linkedin2username [options]

Options:
  -c, --company       Company name to search (required)
  -o, --output        Output file
  -f, --format        Username format template
  -l, --limit         Maximum results
  -u, --user          LinkedIn username
  -p, --password      LinkedIn password
  --no-recurse        Don't recurse into subsidiaries
  -v, --verbose       Verbose output
  -a, --all-formats   Generate all format variations
  -h, --help          Show this help message
```

### Username Format Reference
| Format Template | Example Output | Description |
|-----------------|----------------|-------------|
| `{first}.{last}` | john.doe | First name dot last name |
| `{first}_{last}` | john_doe | First name underscore last name |
| `{first}{last}` | johndoe | First name last name combined |
| `{f}{last}` | jdoe | First initial + last name |
| `{first}{l}` | johnd | First name + last initial |
| `{last}.{first}` | doe.john | Last name dot first name |
| `{last}_{first}` | doe_john | Last name underscore first name |
| `{first}` | john | First name only |
| `{last}` | doe | Last name only |

---

## Chapter 3: Basic Usage

### First Run
```bash
linkedin2username -c "Target Company"
```
**Output:**
```
[*] Searching LinkedIn for: Target Company
[*] Found 150 employees
[*] Generating username variations...
[+] Generated 450 usernames
[*] Results saved to output.txt
```

### Essential Commands

#### Basic Company Search
```bash
linkedin2username -c "Acme Corp"
```
**Explanation:** Scrapes LinkedIn for employees of "Acme Corp" and generates username variations.

#### Output to File
```bash
linkedin2username -c "Acme Corp" -o output.txt
```
**Explanation:** Saves generated usernames to a file.

#### Custom Username Formats
```bash
linkedin2username -c "Acme Corp" -f "{first}.{last}"
```
**Explanation:** Uses custom format for username generation.

#### Limit Results
```bash
linkedin2username -c "Acme Corp" -l 100
```
**Explanation:** Limits to first 100 results.

#### Verbose Output
```bash
linkedin2username -c "Acme Corp" -v
```
**Explanation:** Shows detailed progress and findings.

### Complete Flags Reference

| Flag | Description | Example |
|------|-------------|---------|
| `-c`, `--company` | Company name to search | `linkedin2username -c "Acme"` |
| `-o`, `--output` | Output file | `linkedin2username -c "Acme" -o out.txt` |
| `-f`, `--format` | Username format | `-f "{first}.{last}"` |
| `-l`, `--limit` | Max results | `-l 200` |
| `-u`, `--user` | LinkedIn username | `-u user@email.com` |
| `-p`, `--password` | LinkedIn password | `-p pass123` |
| `--no-recurse` | Don't recurse into subsidiaries | `--no-recurse` |
| `-v`, `--verbose` | Verbose output | `-v` |
| `-a`, `--all-formats` | Generate all format variations | `-a` |
| `--help` | Show help | `--help` |

---

## Chapter 4: Intermediate Usage

### Username Format Templates
```bash
# Dot format
linkedin2username -c "Acme" -f "{first}.{last}"

# Underscore format
linkedin2username -c "Acme" -f "{first}_{last}"

# First initial + last
linkedin2username -c "Acme" -f "{f}{last}"

# First + last initial
linkedin2username -c "Acme" -f "{first}{l}"

# All combinations
linkedin2username -c "Acme" -a
```

### Bash Automation
```bash
#!/bin/bash
# Company OSINT pipeline
COMPANIES=("Acme Corp" "Globex" "Initech")
for company in "${COMPANIES[@]}"; do
    echo "[*] Processing: $company"
    clean=$(echo "$company" | tr ' ' '_')
    linkedin2username -c "$company" -o "usernames_${clean}.txt" -a
done
```

### Tool Chaining
```bash
# Generate usernames then check with Sherlock
linkedin2username -c "Acme Corp" -o users.txt
cat users.txt | xargs -I{} sherlock {}
```

### Advanced Python Script
```python
import subprocess
import json
import csv
from datetime import datetime

class LinkedInUsernameGenerator:
    def __init__(self):
        self.results = {}
    
    def generate(self, company, format_template=None, limit=None):
        cmd = ["linkedin2username", "-c", company]
        if format_template:
            cmd.extend(["-f", format_template])
        if limit:
            cmd.extend(["-l", str(limit)])
        cmd.append("--json")
        
        result = subprocess.run(cmd, capture_output=True, text=True)
        if result.returncode == 0 and result.stdout:
            try:
                self.results[company] = json.loads(result.stdout)
            except json.JSONDecodeError:
                self.results[company] = {}
    
    def get_usernames(self, company):
        if company not in self.results:
            return []
        return self.results[company].get('usernames', [])
    
    def export_csv(self, filename):
        with open(filename, 'w', newline='') as f:
            writer = csv.writer(f)
            writer.writerow(['Company', 'Name', 'Username'])
            for company, data in self.results.items():
                for name, username in data.get('usernames', {}).items():
                    writer.writerow([company, name, username])

# Usage
generator = LinkedInUsernameGenerator()
generator.generate("Acme Corp", "{first}.{last}")
generator.export_csv("linkedin_usernames.csv")
```

---

## Chapter 5: Advanced Usage

### Authentication
```bash
# Use LinkedIn credentials for better results
linkedin2username -c "Acme" -u user@email.com -p password
```

### Combining with Password Spraying
```bash
# Generate users, then spray with common passwords
linkedin2username -c "Acme" -o users.txt
crackmapexec smb target -u users.txt -p 'Password123!'
```

### Docker Deployment
```bash
# Run LinkedIn2Username in Docker
docker run -it --rm \
  -v $(pwd)/results:/results \
  linkedin2username:latest -c "Acme Corp" -o /results/users.txt
```

### Custom Format with Delimiters
```bash
# Email format
linkedin2username -c "Acme" -f "{first}.{last}@acme.com"

# Windows domain format
linkedin2username -c "Acme" -f "ACME\\{first}.{last}"
```

### Bulk Company Processing
```bash
#!/bin/bash
# Process multiple companies
COMPANIES="companies.txt"
OUTPUT_DIR="bulk_$(date +%Y%m%d)"
mkdir -p "$OUTPUT_DIR"

while read company; do
    echo "[*] Processing: $company"
    clean=$(echo "$company" | tr ' ' '_' | tr -d ',')
    linkedin2username -c "$company" -a -o "$OUTPUT_DIR/${clean}.txt"
    sleep 10  # Rate limiting
done < "$COMPANIES"

# Merge all results
cat "$OUTPUT_DIR"/*.txt | sort -u > "$OUTPUT_DIR/all_usernames.txt"
echo "[+] Total unique usernames: $(wc -l < $OUTPUT_DIR/all_usernames.txt)"
```

### Username Format Comparison
```bash
# Generate all formats and compare
COMPANY="Target Corp"

linkedin2username -c "$COMPANY" -f "{first}.{last}" -o first.last.txt
linkedin2username -c "$COMPANY" -f "{first}_{last}" -o first_last.txt
linkedin2username -c "$COMPANY" -f "{f}{last}" -o flast.txt
linkedin2username -c "$COMPANY" -f "{first}{l}" -o firstl.txt

echo "=== Format Comparison ==="
echo "first.last: $(wc -l < first.last.txt)"
echo "first_last: $(wc -l < first_last.txt)"
echo "flast: $(wc -l < flast.txt)"
echo "firstl: $(wc -l < firstl.txt)"
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: Penetration Test Username Generation
```bash
# Step 1: Get target company name
TARGET="Target Corp"

# Step 2: Generate username list
linkedin2username -c "$TARGET" -a -o target_users.txt

# Step 3: Clean up duplicates
sort -u target_users.txt > target_users_unique.txt

# Step 4: Use with password spraying
hydra -L target_users_unique.txt -P passwords.txt smb://target

# Step 5: Generate report
echo "=== Username Generation Report ===" > report.md
echo "Target: $TARGET" >> report.md
echo "Date: $(date)" >> report.md
echo "Total usernames: $(wc -l < target_users_unique.txt)" >> report.md
```

### Scenario 2: CTF Challenge
```bash
# CTF: "Find the employee username format"
linkedin2username -c "CTF Company" -o ctf_users.txt

# Check which format matches the hint
head -20 ctf_users.txt

# Try common formats
linkedin2username -c "CTF Company" -f "{first}.{last}" -o first.last.txt
linkedin2username -c "CTF Company" -f "{f}{last}" -o flast.txt
```

### Scenario 3: Red Team Recon
```bash
# Step 1: Enumerate employees
linkedin2username -c "Target Inc" -o employees.txt

# Step 2: Find naming convention
head -20 employees.txt  # Look for patterns

# Step 3: Generate variations
linkedin2username -c "Target Inc" -f "{f}{last}" -o flast.txt
linkedin2username -c "Target Inc" -f "{first}.{last}" -o first.last.txt

# Step 4: Cross-reference with other tools
cat employees.txt | xargs -I{} sherlock --output "sherlock_{}.txt" {}
```

---

## Chapter 7: Detection & Defense

### Detection
- LinkedIn may detect and block scraping activity
- Unusual access patterns from corporate accounts
- High volume of profile views
- Rapid sequential profile access
- Requests from known VPN/Tor exit nodes

### Mitigation
- Limit LinkedIn enumeration during tests
- Use authenticated sessions when possible
- Monitor for bulk employee data access
- Implement rate limiting on profile views
- Deploy alerts for unusual LinkedIn activity
- Use LinkedIn's official API where possible

### Blue Team Response
1. Monitor for bulk LinkedIn profile access
2. Set up alerts for unusual employee data queries
3. Implement employee training on social engineering
4. Monitor for credential spraying attempts
5. Deploy account lockout policies
6. Use threat intelligence feeds to detect known attack patterns

---

## Chapter 8: Troubleshooting

### Common Errors

#### "LinkedIn login required"
**Solution:** Provide credentials with `-u` and `-p` flags.

#### "No results found"
**Solution:** Try different company name variations or use `-a` for all formats.

#### "Rate limited"
**Solution:** Add delays or use authenticated session.

#### "Selenium not found"
**Solution:** Install ChromeDriver: `pip3 install chromedriver-autoinstaller`

### Performance Tips
- Use authenticated sessions for better results
- Limit results with `-l` for faster processing
- Use `-a` to generate all format variations
- Add delays between requests to avoid rate limiting

### FAQ
**Q: Do I need a LinkedIn Premium account?**
A: No, a free account works, but Premium may provide more results.

**Q: How accurate are the generated usernames?**
A: Generated usernames are guesses based on common formats. Verify manually.

**Q: Can I use this for any company?**
A: Only companies with public LinkedIn profiles. Private companies may not be found.

**Q: How do I update LinkedIn2Username?**
A: Run `pip3 install --upgrade linkedin2username` or `git pull && pip3 install -r requirements.txt`.

---

## Resources

### Official Documentation
- [LinkedIn2Username GitHub](https://github.com/initstring/linkedin2username)
- [LinkedIn OSINT Guide](https://osintframework.com)

### Related Tools
- **Sherlock** — Username enumeration across 400+ sites
- **Hunter** — Email finder and verifier
- **TheHarvester** — Email and subdomain harvester
- **Maigret** — Extended username checker

### Practice
- Use with authorized penetration testing engagements
- Practice on your own company's LinkedIn page
- Join OSINT communities for tips and techniques
