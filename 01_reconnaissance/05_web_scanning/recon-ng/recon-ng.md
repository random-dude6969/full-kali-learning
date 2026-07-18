---
tool_name: "recon-ng"
display_name: "Recon-ng"
mitre_tactic: "reconnaissance"
mitre_tactic_id: "TA0043"
mitre_subcategory: "web_scanning"
install_command: "sudo apt install recon-ng"
version: "5.1.2"
github_repo: "https://github.com/lanmaster53/recon-ng"
kali_tools_page: "https://www.kali.org/tools/recon-ng/"
difficulty_level: "advanced"
requires_root: false
gui_available: false
tags: ["framework", "osint", "recon", "modular"]
related_tools: ["spiderfoot", "maltego", "theharvester"]
---

# Recon-ng

## Chapter 1: Introduction & Overview

### What is Recon-ng?

Recon-ng is a full-featured reconnaissance framework designed to provide a powerful environment to conduct open source web-based reconnaissance quickly and thoroughly. It has a modular architecture similar to Metasploit, with database integration, workspace management, and an extensive marketplace of reconnaissance modules. Each module is designed to perform a specific task — from discovering subdomains to scraping email addresses to enumerating public data breaches.

Recon-ng stands out from other reconnaissance tools because of its framework approach. Rather than being a single-purpose tool, it provides a structured environment where multiple reconnaissance tasks can be organized, executed, and correlated within workspaces. Results are stored in an internal SQLite database, allowing for SQL queries across all collected data.

### History & Background

- **Created:** 2012 by Tim Tomes (lanmaster53)
- **Maintained:** Tim Tomes and community contributors
- **License:** GPL-3.0
- **Language:** Python 3
- **Latest version:** 5.1.2
- **Upstream repository:** https://github.com/lanmaster53/recon-ng

Recon-ng was inspired by Metasploit's modular design and aims to bring the same structured approach to reconnaissance. Version 5 was a complete rewrite in Python 3 with a new marketplace system for modules, improved database integration, and better workspace management.

### When to Use This Tool

- **OSINT investigations:** Comprehensive open-source intelligence gathering
- **Penetration testing:** Structured information-gathering phase
- **Bug bounty:** Multi-source reconnaissance across many data providers
- **Threat intelligence:** Target profiling and indicator gathering
- **Corporate investigations:** Due diligence and background research
- **Digital forensics:** Evidence gathering from public sources
- **CTF challenges:** Multi-source reconnaissance challenges

### Legal and Ethical Considerations

- Modules interact with third-party APIs and services
- Respect API rate limits and terms of service for each provider
- Some modules may query services that require paid subscriptions
- Use only for authorized investigations and assessments
- All queries to third-party services may be logged
- Document all reconnaissance activities for compliance
- Recon-ng does not actively exploit vulnerabilities

### Key Concepts

| Concept | Description |
|---------|-------------|
| **Workspace** | Isolated environment for organizing a specific engagement's data |
| **Module** | A Python script that performs a specific reconnaissance task |
| **Marketplace** | Repository of installable modules maintained by the community |
| **Database** | SQLite database per workspace storing all discovered data |
| **Keys** | API keys for third-party services used by modules |
| **Host** | A discovered network host (IP address) |
| **Domain** | A discovered domain name |
| **Contact** | A discovered person (email, name, title) |
| **Credential** | A discovered credential (username, password, hash) |
| **Pushpins** | Geolocation data points |

---

## Chapter 2: Installation & Setup

### System Requirements

- Operating system: Linux (recommended), macOS, or Windows
- RAM: 512 MB minimum
- Disk space: < 100 MB
- Python: 3.6+
- Network: Internet access for module API calls
- Privileges: Non-root is sufficient

### Installation via APT (Kali/Debian/Ubuntu)

```bash
sudo apt update
sudo apt install recon-ng
```

Expected output:
```
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following NEW packages will be installed:
  recon-ng
0 upgraded, 1 newly installed, 0 to remove and 0 not upgraded.
Need to get 456 kB of archives.
After this operation, 2.3 MB of additional disk space will be used.
Get:1 http://http.kali.org/kali kali-rolling/main amd64 recon-ng 5.1.2-1kali1 [456 kB]
Fetched 456 kB in 0s (2.28 MB/s)
Selecting previously unselected package recon-ng.
(Reading database ... 1234567 files and directories currently installed.)
Preparing to unpack .../recon-ng_5.1.2-1kali1_all.deb ...
Unpacking recon-ng (5.1.2-1kali1) ...
Setting up recon-ng (5.1.2-1kali1) ...
```

### Installation from Source

```bash
# Clone repository
git clone https://github.com/lanmaster53/recon-ng.git
cd recon-ng

# Install dependencies
pip3 install -r REQUIREMENTS

# Launch
./recon-ng
```

### Docker Installation

```bash
docker pull reconng/recon-ng
docker run -it -v $(pwd)/data:/opt/recon-ng/data reconng/recon-ng
```

### First-Time Setup

```bash
# Launch Recon-ng
recon-ng

# The first launch creates the workspace directory structure
```

Expected output:
```
[!] The following modules require marketplace installation:
    exploit-db/google_hacking_database...
[!] The following modules require API keys:
    recon/domains-hosts/bing_domain_web...
```

### Marketplace Installation

```bash
# Inside Recon-ng console:

# Install all marketplace modules
marketplace install all

# Or install specific modules
marketplace install recon/domains-hosts/hackertarget
marketplace install recon/contacts-contacts/mailtester
marketplace install recon/hosts-hosts/resolve
```

### API Key Configuration

```bash
# Inside Recon-ng console:

# Add API keys
keys add shodan_api YOUR_SHODAN_KEY
keys add censys_api_id YOUR_CENSYS_ID
keys add censys_api_secret YOUR_CENSYS_SECRET
keys add google_api_key YOUR_GOOGLE_KEY
keys add github_api_key YOUR_GITHUB_TOKEN
keys add hunterio_api_key YOUR_HUNTER_KEY
keys add fullhunt_api_key YOUR_FULLHUNT_KEY
keys add virustotal_api_key YOUR_VT_KEY
keys add securitytrails_api_key YOUR_ST_KEY

# List configured keys
keys list
```

### Verification

```bash
recon-ng --version
```

Expected output:
```
5.1.2
```

Or verify by launching and checking:

```bash
recon-ng --no-check  # Skip update check
```

---

## Chapter 3: Basic Usage

### First Run

```bash
recon-ng
```

Expected output:
```
[!] No workspace selected.
[!] Use the 'workspaces create' command to create a new workspace.
[recon-ng] >
```

### Essential Commands

#### Create Workspace

```bash
workspaces create target_recon
```

Expected output:
```
[+] Workspace 'target_recon' created.
[+] Workspace 'target_recon' selected.
```

#### Search for Modules

```bash
modules search github
```

Expected output:
```
---------------------
    github_users
    recon/domains-hosts/hackertarget
    recon/contacts-contacts/mailtester
    recon/profiles-profiles/mywot
    ...
---------------------
[57] modules found
```

#### Load Module

```bash
modules load recon/domains-hosts/hackertarget
```

Expected output:
```
[+] Module 'recon/domains-hosts/hackertarget' loaded.
```

#### Set Module Options

```bash
options set SOURCE target.com
```

Expected output:
```
SOURCE => target.com
```

#### Run Module

```bash
run
```

Expected output:
```
[*] Module: recon/domains-hosts/hackertarget
[*] Target: target.com
[*] Running module...
---------------------
| Host            |
---------------------
| sub1.target.com |
| sub2.target.com |
| sub3.target.com |
---------------------
[+] 3 hosts found.
[+] 3 results added to the database.
```

#### Query Database

```bash
db query SELECT * FROM hosts
```

Expected output:
```
+----+------------------+----------------+
| rowid | host             | ip_address     |
+----+------------------+----------------+
| 1  | sub1.target.com  | 192.168.1.10   |
| 2  | sub2.target.com  | 192.168.1.11   |
| 3  | sub3.target.com  | 192.168.1.12   |
+----+------------------+----------------+
```

### Complete CLI Reference

#### Workspace Commands

| Command | Description | Example |
|---------|-------------|---------|
| `workspaces create NAME` | Create new workspace | `workspaces create target_recon` |
| `workspaces list` | List all workspaces | `workspaces list` |
| `workspaces select NAME` | Select workspace | `workspaces select target_recon` |
| `workspaces remove NAME` | Delete workspace | `workspaces remove old_recon` |
| `workspaces rename NAME` | Rename workspace | `workspaces rename new_name` |

#### Module Commands

| Command | Description | Example |
|---------|-------------|---------|
| `modules search QUERY` | Search modules | `modules search github` |
| `modules load PATH` | Load module | `modules load recon/domains-hosts/hackertarget` |
| `modules reload` | Reload current module | `modules reload` |
| `modules info PATH` | Show module info | `modules info recon/domains-hosts/hackertarget` |
| `modules install PATH` | Install from marketplace | `modules install recon/domains-hosts/hackertarget` |
| `modules remove PATH` | Remove module | `modules remove recon/domains-hosts/hackertarget` |
| `options set KEY VALUE` | Set module option | `options set SOURCE target.com` |
| `options unset KEY` | Unset option | `options unset SOURCE` |
| `options list` | List all options | `options list` |
| `run` | Execute loaded module | `run` |

#### Database Commands

| Command | Description | Example |
|---------|-------------|---------|
| `db query SQL` | Run SQL query | `db query SELECT * FROM hosts` |
| `db inserts TABLE` | Show insert syntax | `db inserts hosts` |
| `db schema` | Show database schema | `db schema` |
| `db snapshot` | Create DB snapshot | `db snapshot` |
| `db drop TABLE` | Drop table | `db drop hosts` |
| `db count TABLE` | Count rows | `db count hosts` |
| `db dump TABLE` | Export table to CSV | `db dump hosts` |
| `db export TABLE FILE` | Export to file | `db export hosts /tmp/hosts.csv` |

#### Marketplace Commands

| Command | Description | Example |
|---------|-------------|---------|
| `marketplace search QUERY` | Search marketplace | `marketplace search github` |
| `marketplace install PATH` | Install module | `marketplace install recon/domains-hosts/hackertarget` |
| `marketplace remove PATH` | Remove module | `marketplace remove recon/domains-hosts/hackertarget` |
| `marketplace update all` | Update all modules | `marketplace update all` |
| `marketplace info PATH` | Show module info | `marketplace info recon/domains-hosts/hackertarget` |

#### Key Commands

| Command | Description | Example |
|---------|-------------|---------|
| `keys add KEY VALUE` | Add API key | `keys add shodan_api YOUR_KEY` |
| `keys list` | List all keys | `keys list` |
| `keys remove KEY` | Remove key | `keys remove shodan_api` |

#### General Commands

| Command | Description | Example |
|---------|-------------|---------|
| `help` | Show help | `help` |
| `help COMMAND` | Command-specific help | `help modules` |
| `shell` | Drop to system shell | `shell` |
| `pdb` | Python debugger | `pdb` |
| `spool start FILE` | Start spooling output | `spool start output.txt` |
| `spool stop` | Stop spooling | `spool stop` |
| `resource FILE` | Execute command file | `resource commands.rc` |
| `exit` | Exit Recon-ng | `exit` |

---

## Chapter 4: Intermediate Usage

### Module Chaining

```bash
# Step 1: Create workspace
workspaces create full_recon

# Step 2: Find subdomains
modules load recon/domains-hosts/hackertarget
options set SOURCE target.com
run

# Step 3: Resolve discovered hosts
modules load recon/hosts-hosts/resolve
run

# Step 4: Check for port开放
modules load recon/hosts-ports/shodan_hostname
run

# Step 5: Find email contacts
modules load recon/contacts-contacts/mailtester
run

# Step 6: Review all results
db query SELECT * FROM hosts
db query SELECT * FROM contacts
db query SELECT * FROM ports
```

### Database Queries

```bash
# Query all hosts
db query SELECT host, ip_address FROM hosts

# Query hosts with open ports
db query SELECT h.host, p.port, p.protocol FROM hosts h JOIN ports p ON h.rowid = p.host_rowid

# Export results
db query SELECT host FROM hosts INTO OUTFILE '/tmp/hosts.csv'

# Count results by table
db query SELECT COUNT(*) as total FROM hosts

# Find unique IP ranges
db query SELECT DISTINCT SUBSTR(ip_address, 1, INSTR(ip_address, '.', -1)-1) as subnet FROM hosts

# Find hosts by pattern
db query SELECT * FROM hosts WHERE host LIKE '%admin%'
```

### Command Scripts (Resource Files)

```bash
# Create a resource file for automated execution
cat > recon_script.rc << 'EOF'
workspaces create automated_recon
modules load recon/domains-hosts/hackertarget
options set SOURCE target.com
run
modules load recon/hosts-hosts/resolve
run
db query SELECT host, ip_address FROM hosts
exit
EOF

# Execute resource file
recon-ng -r recon_script.rc
```

### Python API Integration

```python
#!/usr/bin/env python3
"""
recon_ng_automation.py — Programmatic Recon-ng integration
"""

import subprocess
import time

def run_recon_ng_commands(commands):
    """Execute Recon-ng commands programmatically."""
    # Build command script
    script = "\n".join(commands) + "\nexit\n"
    
    # Run Recon-ng with commands via stdin
    proc = subprocess.Popen(
        ["recon-ng", "--no-check"],
        stdin=subprocess.PIPE,
        stdout=subprocess.PIPE,
        stderr=subprocess.PIPE,
        text=True
    )
    
    stdout, stderr = proc.communicate(input=script)
    return stdout, stderr

# Example: Automated reconnaissance
commands = [
    "workspaces create auto_recon",
    "modules load recon/domains-hosts/hackertarget",
    "options set SOURCE example.com",
    "run",
    "modules load recon/hosts-hosts/resolve",
    "run",
    "db query SELECT host, ip_address FROM hosts"
]

stdout, stderr = run_recon_ng_commands(commands)
print("Output:", stdout)
if stderr:
    print("Errors:", stderr)
```

### Tool Chaining

```bash
# Inside Recon-ng, chain with external tools
# After running modules, export and process with bash

# Export hosts
db query SELECT host FROM hosts > /tmp/hosts.txt

# Run massdns from bash shell
shell
massdns -r resolvers.txt -t A -o S /tmp/hosts.txt > /tmp/resolved.txt
exit

# Import resolved IPs back
db query "INSERT INTO hosts (host, ip_address) VALUES ('new.host.com', '1.2.3.4')"
```

---

## Chapter 5: Advanced Usage

### Custom Module Development

```python
# ~/.recon-ng/modules/custom/my_module.py

from recon.core.module import BaseModule

class Module(BaseModule):

    meta = {
        'name': 'Custom Domain Enumerator',
        'author': 'Your Name',
        'description': 'Custom module to enumerate domains',
        'query': 'SELECT DISTINCT domain FROM domains WHERE domain IS NOT NULL',
    }

    def __init__(self, params):
        BaseModule.__init__(self, params)
        self.info = self.meta

    def module_run(self, domain):
        # Your custom logic here
        self.insert_hosts(host=f'sub.{domain}', ip_address='0.0.0.0')
```

### Database Schema

```bash
# View complete database schema
db schema
```

Expected output:
```
+--------+--------------+------+-----+---------+----------------+
| Field  | Type         | Null | Key | Default | Extra          |
+--------+--------------+------+-----+---------+----------------+
| rowid  | INTEGER      | NO   |     | NULL    | autoincrement  |
| domain | VARCHAR(255)| YES  |     | NULL    |                |
+--------+--------------+------+-----+---------+----------------+

+------------+--------------+------+-----+---------+----------------+
| Field      | Type         | Null | Key | Default | Extra          |
+------------+--------------+------+-----+---------+----------------+
| rowid      | INTEGER      | NO   |     | NULL    | autoincrement  |
| host       | VARCHAR(255)| YES  |     | NULL    |                |
| ip_address | VARCHAR(45) | YES  |     | NULL    |                |
+------------+--------------+------+-----+---------+----------------+
```

### Marketplace Module Categories

| Category | Description | Example Modules |
|----------|-------------|-----------------|
| `recon/domains-hosts/` | Find hosts for domains | hackertarget, bing_domain_web, threatminer |
| `recon/hosts-hosts/` | Enrich host data | resolve, reverse_resolve, shodan_hostname |
| `recon/hosts-ports/` | Find open ports | shodan_hostname, netcraft_search |
| `recon/contacts-contacts/` | Find contacts | mailtester, whois_pocs |
| `recon/contacts-emails/` | Find email addresses | hunterio, ahmia_search |
| `recon/profiles-profiles/` | Find social profiles | mywot, github_profiles |
| `recon/pushpins/` | Geolocation data | googlemaps, flickr |
| `discovery/` | General discovery | info_disclosure, static_malware_analysis |

### Workspace Management Best Practices

```bash
# Create workspace per engagement
workspaces create client_a_recon
workspaces create client_b_recon
workspaces create ctf_challenge

# Use descriptive naming
workspaces create target_comprehensive_20240718

# Snapshot before major operations
db snapshot

# Clean up completed workspaces
workspaces remove old_engagement_recon
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: Full Recon Workflow

**Objective:** Comprehensive reconnaissance of a target organization.

**Step 1: Setup Workspace and Keys**
```bash
recon-ng

# Create workspace
workspaces create target_full_recon

# Add API keys
keys add shodan_api YOUR_SHODAN_KEY
keys add hunterio_api_key YOUR_HUNTER_KEY
keys add github_api_key YOUR_GITHUB_TOKEN
```

**Step 2: Domain Enumeration**
```bash
# Find subdomains via multiple sources
modules load recon/domains-hosts/hackertarget
options set SOURCE target.com
run

modules load recon/domains-hosts/bing_domain_web
options set SOURCE target.com
run

modules load recon/domains-hosts/certificate_transparency
options set SOURCE target.com
run
```

**Step 3: Host Enrichment**
```bash
# Resolve hostnames to IPs
modules load recon/hosts-hosts/resolve
run

# Get IP info from Shodan
modules load recon/hosts-ports/shodan_hostname
run
```

**Step 4: Contact Discovery**
```bash
# Find email addresses
modules load recon/contacts-contacts/mailtester
options set SOURCE target.com
run

# Search for employee profiles
modules load recon/profiles-profiles/mywot
options set SOURCE target.com
run
```

**Step 5: Generate Report**
```bash
# Query all data
db query SELECT host, ip_address FROM hosts
db query SELECT first_name, last_name, email FROM contacts

# Export to files
db query SELECT host FROM hosts INTO OUTFILE '/tmp/target_hosts.csv'
db query SELECT email FROM contacts INTO OUTFILE '/tmp/target_emails.csv'
```

### Scenario 2: CTF Challenge — "Gather Intel"

**Objective:** Use Recon-ng to gather intelligence on a CTF target.

**Step 1: Create Workspace**
```bash
workspaces create ctf_intel
```

**Step 2: Enumerate**
```bash
modules load recon/domains-hosts/hackertarget
options set SOURCE ctf-target.com
run

# Check results
db query SELECT * FROM hosts
```

**Step 3: Enrich**
```bash
modules load recon/hosts-hosts/resolve
run

# Look for interesting patterns
db query SELECT host FROM hosts WHERE host LIKE '%admin%'
db query SELECT host FROM hosts WHERE host LIKE '%flag%'
db query SELECT host FROM hosts WHERE host LIKE '%secret%'
```

**Step 4: External Lookup**
```bash
# Use Shodan module if API key available
modules load recon/hosts-ports/shodan_hostname
run

db query SELECT host, port, protocol FROM ports
```

### Scenario 3: Bug Bounty Multi-Source Recon

**Objective:** Maximize subdomain discovery for bug bounty scope.

**Step 1: Multi-Source Enumeration**
```bash
workspaces create bugbounty_recon

# Source 1: HackerTarget
modules load recon/domains-hosts/hackertarget
options set SOURCE target.com
run

# Source 2: Bing
modules load recon/domains-hosts/bing_domain_web
options set SOURCE target.com
run

# Source 3: Certificate Transparency
modules load recon/domains-hosts/certificate_transparency
options set SOURCE target.com
run

# Source 4: Netcraft
modules load recon/domains-hosts/netcraft_search
options set SOURCE target.com
run
```

**Step 2: Deduplicate and Analyze**
```bash
# Check total unique hosts
db query SELECT COUNT(DISTINCT host) as unique_hosts FROM hosts

# Find subdomains by prefix
db query SELECT SUBSTR(host, 1, INSTR(host, '.')-1) as prefix, COUNT(*) as count FROM hosts GROUP BY prefix ORDER BY count DESC
```

**Step 3: Export for Further Testing**
```bash
# Export all hosts
db query SELECT host FROM hosts > all_hosts.txt

# Count results
wc -l all_hosts.txt
```

---

## Chapter 7: Detection & Defense

### How Recon-ng Activity Is Detected

**API Call Monitoring:**
- Recon-ng makes API calls to third-party services (Shodan, Hunter.io, etc.)
- These calls originate from your IP, not the target's network
- Third-party services log API usage

**User-Agent Patterns:**
- Recon-ng modules may use default User-Agent strings
- Some modules use custom or configurable User-Agent headers

**Query Patterns:**
- Sequential queries to multiple API services
- Pattern of queries targeting the same domain
- High frequency of reconnaissance-related queries

### Blue Team Defensive Measures

**1. Monitor Third-Party Data Sources:**
```bash
# Check what's exposed about your organization
# crt.sh — Certificate Transparency
curl "https://crt.sh/?q=%.target.com&output=json" | jq .

# SecurityTrails
# Shodan
# Censys
```

**2. Limit Public DNS Information:**
```bash
# Minimize DNS records exposed
# Remove internal hostnames from public DNS
# Use split-horizon DNS for internal/external views
```

**3. Set Up OSINT Monitoring:**
```bash
#!/bin/bash
# osint_monitor.sh — Monitor for OSINT gathering about your organization

# Monitor certificate transparency
curl -s "https://crt.sh/?q=%.company.com&output=json" | \
    jq -r '.[] | .name_value' | sort -u > current_certs.txt

# Compare with previous scan
if [ -f previous_certs.txt ]; then
    NEW=$(comm -13 previous_certs.txt current_certs.txt)
    if [ -n "$NEW" ]; then
        echo "New certificates discovered:"
        echo "$NEW"
    fi
fi
mv current_certs.txt previous_certs.txt
```

**4. Control Information Leakage:**
```bash
# Review what's publicly accessible
# Check email format disclosure
# Review social media profiles of employees
# Audit public code repositories
```

**5. Respond to Reconnaissance:**
```bash
# When reconnaissance is detected, consider:
# 1. Document the activity
# 2. Check for related scanning activity
# 3. Review exposed information
# 4. Remediate any misconfigurations found
# 5. Update security policies
```

---

## Chapter 8: Troubleshooting

### Common Errors

#### "Module not found"

**Cause:** Module not installed from marketplace.

**Solutions:**
```bash
# Install all modules
marketplace install all

# Or install specific module
marketplace install recon/domains-hosts/hackertarget

# Verify installation
modules search hackertarget
```

#### "API key required"

**Cause:** Module needs an API key that hasn't been configured.

**Solutions:**
```bash
# Add the required key
keys add shodan_api YOUR_SHODAN_KEY

# List configured keys
keys list

# Check module requirements
modules info recon/hosts-ports/shodan_hostname
```

#### "Database error" or "database is locked"

**Cause:** Database corruption or concurrent access issues.

**Solutions:**
```bash
# Create snapshot first
db snapshot

# If locked, try:
db drop TABLE_NAME
# Then re-run the module

# If corrupted, recreate workspace
workspaces remove current_workspace
workspaces create new_workspace
```

#### "Module failed to load"

**Cause:** Python dependency issues or module errors.

**Solutions:**
```bash
# Check Python version
python3 --version

# Install missing dependencies
pip3 install requests beautifulsoup4

# Update Recon-ng
sudo apt update && sudo apt install --reinstall recon-ng
```

#### "No results returned"

**Cause:** Module found no data, API key invalid, or source is unavailable.

**Solutions:**
```bash
# Verify API keys are valid
keys list

# Test API connectivity manually
curl -H "apikey: YOUR_KEY" "https://api.shodan.io/dns/domain/target.com"

# Try alternative modules
modules search domains
```

### Performance Optimization

```bash
# Use multiple workspaces for parallel operations
# Window 1: workspaces create recon_part1
# Window 2: workspaces create recon_part2

# Batch module execution
cat > batch_recon.rc << 'EOF'
workspaces create batch_recon
modules load recon/domains-hosts/hackertarget
options set SOURCE target.com
run
modules load recon/hosts-hosts/resolve
run
modules load recon/contacts-contacts/mailtester
options set SOURCE target.com
run
db query SELECT * FROM hosts
db query SELECT * FROM contacts
exit
EOF

recon-ng -r batch_recon.rc
```

### FAQ

**Q: Recon-ng vs Maltego — which is better?**
A: Recon-ng is command-line focused, scriptable, and better for automation. Maltego is GUI-based with better visualization. Recon-ng is more flexible for custom workflows.

**Q: Can Recon-ng run without API keys?**
A: Yes, many modules work without API keys (hackertarget, certificate_transparency, resolve). But API keys significantly expand capability.

**Q: How do I update Recon-ng?**
A: `sudo apt update && sudo apt install --reinstall recon-ng` for APT installs, or `git pull` for source installs.

**Q: Can Recon-ng import data from other tools?**
A: Yes, use `db query "INSERT INTO table ..."` to import data from external sources.

**Q: How do I backup a workspace?**
A: Use `db snapshot` to create a backup, or copy the SQLite database file from `~/.recon-ng/workspaces/WORKSPACE_NAME/`.

**Q: Can Recon-ng be used non-interactively?**
A: Yes, use resource files with `recon-ng -r script.rc` for non-interactive execution.

---

## Resources

- [Recon-ng GitHub Repository](https://github.com/lanmaster53/recon-ng)
- [Recon-ng Documentation](https://recon-ng.com)
- [Recon-ng Wiki](https://github.com/lanmaster53/recon-ng/wiki)
- [Recon-ng Marketplace](https://github.com/lanmaster53/recon-ng/wiki/Marketplace)
- [Recon-ng Module Development Guide](https://github.com/lanmaster53/recon-ng/wiki/Development)
- [Tim Tomes — Recon-ng Presentation](https://www.youtube.com/results?search_query=recon-ng+tim+tomes)
- [Kali Linux Recon-ng Package](https://www.kali.org/tools/recon-ng/)
- [OSINT Framework](https://osintframework.com)
- [SpiderFoot — Alternative Recon Framework](https://github.com/smicallef/spiderfoot)
- [Maltego — GUI Reconnaissance Tool](https://www.maltego.com)
