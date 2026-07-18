---
tool_name: "assetfinder"
display_name: "Assetfinder Cheatsheet"
---

# Assetfinder — Quick Reference Cheatsheet

## Basic Usage

```bash
# Basic subdomain discovery
assetfinder example.com

# Only subdomains (exclude related)
assetfinder --subs-only example.com

# Include related domains (default)
assetfinder --include-related example.com
```

## Output

```bash
# Save to file
assetfinder example.com > subdomains.txt

# Count results
assetfinder example.com | wc -l

# Sort and deduplicate
assetfinder example.com | sort -u
```

## Filtering

```bash
# Filter by keyword
assetfinder example.com | grep -i "api"

# Filter by TLD
assetfinder example.com | grep '\.com$'

# Remove wildcards
assetfinder example.com | grep -v '^\*'

# Get only deep subdomains
assetfinder example.com | awk -F. 'NF>3'
```

## Integration Pipelines

```bash
# Assetfinder + httpx (find live hosts)
assetfinder example.com | httpx -silent

# Assetfinder + nmap (port scan)
assetfinder example.com | httpx -silent | nmap -iL - -T4

# Assetfinder + feroxbuster (directory brute-force)
assetfinder example.com | httpx -silent | xargs -I{} feroxbuster -u {} -q

# Assetfinder + waybackurls (historical URLs)
assetfinder example.com | waybackurls | sort -u

# Assetfinder + gau (all URLs)
assetfinder example.com | gau | sort -u

# Assetfinder + nuclei (vulnerability scanning)
assetfinder example.com | httpx -silent | nuclei -t nuclei-templates/
```

## Multi-Tool Recon

```bash
# Merge results from multiple tools
assetfinder example.com > af.txt
subfinder -d example.com -silent > sf.txt
amass enum -passive -d example.com > am.txt

# Combine and deduplicate
cat af.txt sf.txt am.txt | sort -u > all_subdomains.txt
```

## Quick One-Liners

| Use Case | Command |
|----------|---------|
| Quick subdomain list | `assetfinder example.com` |
| Live subdomains only | `assetfinder example.com \| httpx -silent` |
| Save to file | `assetfinder example.com > subs.txt` |
| Count subdomains | `assetfinder example.com \| wc -l` |
| Filter for API subs | `assetfinder example.com \| grep api` |
| Full recon pipeline | `assetfinder example.com \| httpx -silent \| nmap -iL - -T4 -oA recon` |
| Historical URLs | `assetfinder example.com \| waybackurls` |
| JSON output | `assetfinder example.com \| jq -R -s 'split("\n") \| map(select(. != ""))'` |

## Batch Operations

```bash
# Scan multiple domains
for domain in $(cat domains.txt); do
    echo "[*] $domain"
    assetfinder "$domain" > "${domain}_subs.txt"
done

# Parallel scanning
cat domains.txt | xargs -P 5 -I{} sh -c 'assetfinder "{}" > "{}_subs.txt"'
```
