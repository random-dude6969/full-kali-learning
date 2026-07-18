# The Harvester — Cheatsheet

## Quick Start

```bash
# Basic scan with all sources
theharvester -d target.com -b all

# Save to HTML report
theharvester -d target.com -b all -f results.html

# With DNS brute-force
theharvester -d target.com -b all -c
```

## Essential Commands

| Command | Description |
|---------|-------------|
| `theharvester -d domain -b all` | Scan all data sources |
| `theharvester -d domain -b google` | Google-only scan |
| `theharvester -d domain -b all -c` | With DNS brute-force |
| `theharvester -d domain -b all -v` | With virtual host search |
| `theharvester -d domain -b all -l 500` | Limit to 500 results |
| `theharvester -d domain -b all -f report.html` | Save HTML report |
| `theharvester -d domain -b all --json report.json` | Save JSON report |

## Data Sources

| Source | API Key | Notes |
|--------|---------|-------|
| `all` | No | All available sources |
| `google` | No | Default search |
| `bing` | Optional | Better with API key |
| `duckduckgo` | No | Privacy-focused |
| `crtsh` | No | Certificate Transparency |
| `shodan` | Yes | IoT/infrastructure |
| `hunter` | Yes | Email verification |
| `github-code` | Optional | Code leaks |
| `virustotal` | Yes | Threat intelligence |
| `censys` | Yes | Internet-wide scan |

## Flag Reference

| Flag | Description |
|------|-------------|
| `-d` | Target domain (required) |
| `-b` | Data source(s) |
| `-l` | Result limit |
| `-c` | DNS brute-force |
| `-n` | DNS reverse lookup |
| `-r` | Resolver file |
| `-t` | TLD expansion |
| `-v` | Virtual host search |
| `-S` | SHODAN host discovery |
| `-f` | Output file (HTML) |
| `--json` | JSON output |
| `-h` | Help |
| `--version` | Version info |

## Multi-Source Examples

```bash
# Multiple specific sources
theharvester -d target.com -b google,bing,crtsh

# API-enhanced sources
theharvester -d target.com -b bingapi,hunter,censys,shodan

# Quick passive recon
theharvester -d target.com -b crtsh,threatminer
```

## Output Parsing

```bash
# Extract emails
theharvester -d target.com -b all 2>/dev/null | \
    grep -oE '[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}'

# Extract subdomains
theharvester -d target.com -b all 2>/dev/null | \
    grep -oE '[a-zA-Z0-9.-]+\.target\.com' | sort -u
```

## Automation

```bash
# Multi-domain batch scan
for domain in $(cat domains.txt); do
    theharvester -d "$domain" -b all -f "${domain}.html" 2>/dev/null
    sleep 30  # Rate limit
done

# Pipe to file
theharvester -d target.com -b all 2>/dev/null | tee harvest_output.txt
```

## Workflow Integration

```bash
# Recon → Subdomain → Port Scan
theharvester -d target.com -b all 2>/dev/null | \
    grep -oE '[a-zA-Z0-9.-]+\.target\.com' | \
    sort -u > subs.txt && \
    nmap -sV -iL <(dig +short $(cat subs.txt) | sort -u)
```
