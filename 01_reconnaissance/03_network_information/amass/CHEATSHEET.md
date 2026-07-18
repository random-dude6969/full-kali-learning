# Amass Cheatsheet

## Quick Start

```bash
# Passive enumeration (fast, no direct queries)
amass enum -passive -d example.com

# Active enumeration
amass enum -d example.com

# Save results
amass enum -d example.com -o results.txt
```

## Core Commands

| Command | Description |
|---------|-------------|
| `amass intel -org "Name"` | Find organization domains |
| `amass intel -asn 13335` | Discover by ASN |
| `amass intel -addr 10.0.0.0/8` | Find by IP range |
| `amass enum -passive -d DOMAIN` | Passive subdomain enum |
| `amass enum -d DOMAIN` | Active subdomain enum |
| `amass db -show` | View database contents |
| `amass db -init` | Initialize fresh database |
| `amass viz -d3 -d DOMAIN` | Generate D3.js visualization |
| `amass track -d DOMAIN` | Track changes over time |

## Target Specification

| Flag | Description | Example |
|------|-------------|---------|
| `-d` | Target domain | `-d example.com` |
| `-dL` | File with domains | `-dL domains.txt` |
| `-org` | Organization name | `-org "Corp Name"` |
| `-asn` | ASN number | `-asn 13335` |
| `-cidr` | CIDR range | `-cidr 192.168.0.0/16` |
| `-addr` | IP address/CIDR | `-addr 10.0.0.0/8` |

## Enumeration Options

| Flag | Description | Example |
|------|-------------|---------|
| `-passive` | Passive only | `-passive` |
| `-active` | Active enumeration | `-active` |
| `-src` | Show data sources | `-src` |
| `-v` | Verbose output | `-v` |
| `-timeout` | Query timeout (sec) | `-timeout 30` |
| `-max-dns-queries` | Rate limit | `-max-dns-queries 100` |
| `-rf` | Resolver file | `-rf resolvers.txt` |
| `-r` | Specific resolvers | `-r 8.8.8.8,1.1.1.1` |

## Output Options

| Flag | Description | Example |
|------|-------------|---------|
| `-o` | Output file | `-o results.txt` |
| `-format json` | JSON output | `-format json` |
| `-format graphml` | GraphML output | `-format graphml` |
| `-nolocal` | Exclude local IPs | `-nolocal` |

## Data Source Options

| Flag | Description | Example |
|------|-------------|---------|
| `-src` | Show source of each result | `-src` |
| `-include` | Specific sources only | `-include shodan,censys` |
| `-exclude` | Skip sources | `-exclude archiveit` |
| `-list` | List all available sources | `-list` |

## Common Pipelines

```bash
# Amass → httpx → nmap
amass enum -d DOMAIN | httpx | nmap -iL - -oA scan

# Amass → nuclei
amass enum -d DOMAIN | sed 's/^/https:\/\//' | nuclei

# Amass → subjack (subdomain takeover)
amass enum -d DOMAIN -o subs.txt
subjack -w subs.txt -t 100 -timeout 30 -ssl -c fingerprints.json
```

## Automation Scripts

### Quick Recon Pipeline
```bash
#!/bin/bash
DOMAIN=$1
mkdir -p recon_$DOMAIN

amass enum -d "$DOMAIN" -src -o recon_$DOMAIN/amass.json -format json
jq -r '.[].name' recon_$DOMAIN/amass.json | sort -u > recon_$DOMAIN/subs.txt
cat recon_$DOMAIN/subs.txt | httpx -silent -o recon_$DOMAIN/live.txt
nmap -iL recon_$DOMAIN/live.txt -T4 -Pn -sV -oA recon_$DOMAIN/nmap
```

### Multi-Target Enumeration
```bash
#!/bin/bash
while IFS= read -r domain; do
    echo "[*] Enumerating: $domain"
    amass enum -passive -d "$domain" -o "${domain}_subs.txt"
    sleep 30
done < domains.txt
```

## Evasion Techniques

```bash
# Slow enumeration
amass enum -d DOMAIN -max-dns-queries 50 -passive

# Route through Tor
amass enum -d DOMAIN -proxy socks5://127.0.0.1:9050

# Use only trusted sources
amass enum -d DOMAIN -include shodan,censys,virustotal

# Rotate proxies
for proxy in $(cat proxies.txt); do
    amass enum -passive -d DOMAIN -proxy "$proxy" -o /dev/null
    sleep 60
done
```

## Database Operations

```bash
amass db -init                    # Initialize database
amass db -show                    # Show all data
amass db -show -d DOMAIN          # Show domain data
amass db -show -o export.json -format json  # Export JSON
```

## Configuration

```bash
# Config file location
~/.config/amass/config.ini

# Environment variables
export AMASS_SHODAN_API_KEY="key"
export AMASS_CENSYS_API_KEY="key"
export AMASS_SECURITYTRAILS_API_KEY="key"
export AMASS_VIRUSTOTAL_API_KEY="key"
```

## Troubleshooting

```bash
# Command not found
export PATH=$PATH:$(go env GOPATH)/bin

# DNS issues
amass enum -d DOMAIN -r 8.8.8.8,1.1.1.1

# Database locked
pkill -f amass
rm -f ~/.config/amass/graph/*.lock

# Out of memory
amass enum -d DOMAIN -passive

# Verbose debugging
amass enum -d DOMAIN -v 2>&1 | tee debug.log
```
