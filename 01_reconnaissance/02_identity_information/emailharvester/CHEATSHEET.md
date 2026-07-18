# The Harvester — Quick Reference Cheatsheet

## Basic Commands

```bash
# Harvest all sources
theharvester -d example.com -b all

# Harvest specific source
theharvester -d example.com -b google

# Limit results
theharvester -d example.com -b all -l 200

# Multiple sources
theharvester -d example.com -b google,bing,duckduckgo

# Save to file
theharvester -d example.com -b all -f report.html
```

## Common Flags

| Flag | Description | Example |
|------|-------------|---------|
| `-d` | Target domain | `-d example.com` |
| `-b` | Data source | `-b google` |
| `-l` | Limit results | `-l 200` |
| `-f` | Output file | `-f report.html` |
| `-S` | Start result number | `-S 100` |
| `-p` | Proxy | `-p http://127.0.0.1:8080` |
| `-s` | Shodan enrichment | `-s` |
| `-c` | DNS brute-force | `-c` |
| `-e` | DNS server | `-e 8.8.8.8` |
| `-r` | Resolver file | `-r resolvers.txt` |
| `-v` | Verify hosts | `-v` |
| `-n` | Reverse DNS | `-n` |

## Data Sources

| Source | API Key? | Type |
|--------|----------|------|
| `all` | No | All sources |
| `google` | No | Search engine |
| `bing` | No | Search engine |
| `duckduckgo` | No | Search engine |
| `baidu` | No | Search engine |
| `yahoo` | No | Search engine |
| `crt.sh` | No | Certificate transparency |
| `dnsdumpster` | No | DNS database |
| `rapiddns` | No | DNS database |
| `threatminer` | No | Threat intel |
| `hackertarget` | No | DNS lookup |
| `urlscan` | No | URL scanning |
| `otx` | No | Threat intel |
| `shodan` | Yes | IoT search |
| `virustotal` | Yes | Threat intel |
| `censys` | Yes | Certificate search |
| `securityTrails` | Yes | DNS history |
| `github-code` | Yes | Code search |
| `hunter` | Yes | Email finder |
| `fullhunt` | Yes | Attack surface |

## Quick Scenarios

```bash
# Quick email discovery
theharvester -d target.com -b google,bing -l 100

# Comprehensive recon
theharvester -d target.com -b all -l 500 -f full_report.html

# Subdomain focus
theharvester -d target.com -b crt.sh,dnsdumpster -f subdomains.html

# With Shodan enrichment
theharvester -d target.com -b all -s -f enriched.html

# DNS brute-force
theharvester -d target.com -b all -c -f brute.html

# Custom DNS
theharvester -d target.com -b all -e 8.8.8.8 -f custom.html

# Stealth mode with proxy
theharvester -d target.com -b all -p http://127.0.0.1:8080

# Pagination (start at result 100)
theharvester -d target.com -b google -S 100 -l 100
```

## Automation Scripts

### Batch Harvest

```bash
#!/bin/bash
for domain in $(cat domains.txt); do
    theharvester -d "$domain" -b all -f "results_${domain}.html"
    sleep 30
done
```

### Extract Emails

```bash
theharvester -d target.com -b all -f results.html
grep -oE '[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}' results.html | sort | uniq > emails.txt
```

### Extract Subdomains

```bash
theharvester -d target.com -b crt.sh -f results.html
grep -oE '[a-zA-Z0-9.-]+\.target\.com' results.html | sort | uniq > subdomains.txt
```

### Full Recon Pipeline

```bash
#!/bin/bash
TARGET="$1"
theharvester -d "$TARGET" -b all -l 500 -f harvester.html
grep -oE '[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}' harvester.html | sort | uniq > emails.txt
grep -oE "[a-zA-Z0-9.-]+\.$TARGET" harvester.html | sort | uniq > subdomains.txt
httpx -l subdomains.txt -silent > live_hosts.txt
nmap -iL live_hosts.txt -T4 -F -oA nmap_results
```

## API Keys Setup

```yaml
# ~/.theHarvester/api-keys.yaml
virustotal: YOUR_VT_KEY
shodan: YOUR_SHODAN_KEY
censys_id: YOUR_CENSYS_ID
censys_secret: YOUR_CENSYS_SECRET
securitytrails: YOUR_ST_KEY
github: YOUR_GITHUB_TOKEN
```

## Troubleshooting

| Problem | Solution |
|---------|----------|
| No results | Try more sources, increase `-l` |
| Rate limited | Wait 15-60 min, use proxy |
| Module not found | `pip3 install theHarvester` |
| Connection timeout | Check network, disable proxy |
| Command not found | Add to PATH or use full path |

## Integration

```bash
# With Sherlock (username hunting)
theharvester -d target.com -b all -f results.html
grep -oE '[a-zA-Z0-9._%+-]+@' results.html | sed 's/@//' | sort | uniq > usernames.txt
sherlock $(cat usernames.txt) --print-found

# With Amass (deep subdomain recon)
theharvester -d target.com -b crt.sh -f harvester.html
amass enum -d target.com -active -o amass.txt
cat harvester_subs.txt amass.txt | sort | uniq > combined.txt

# With Nmap (port scanning)
theharvester -d target.com -b crt.sh -f subs.html
grep -oE '[a-zA-Z0-9.-]+\.target\.com' subs.html | sort | uniq > subs.txt
nmap -iL subs.txt -T4 -F -oA scan_results
```

## Output Formats

| Format | Flag | Use Case |
|--------|------|----------|
| HTML | `-f report.html` | Human-readable reports |
| XML | `-f report.xml` | Machine parsing |
| CSV | `-f report.csv` | Spreadsheet import |
| JSON | `-f report.json` | Programmatic access |

## Legal Reminders

- Only use on authorized targets
- Document your methodology
- Use findings responsibly
- Follow local laws and regulations
- Obtain written permission before testing
