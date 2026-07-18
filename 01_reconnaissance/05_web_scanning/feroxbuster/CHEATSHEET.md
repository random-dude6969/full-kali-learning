---
tool_name: "feroxbuster"
display_name: "Feroxbuster Cheatsheet"
---

# Feroxbuster — Quick Reference Cheatsheet

## Basic Usage

```bash
# Basic scan (uses default wordlist)
feroxbuster -u https://target.com

# Scan with specific wordlist
feroxbuster -u https://target.com -w /usr/share/wordlists/dirb/common.txt

# Recursive scan
feroxbuster -u https://target.com -r

# Recursive with depth
feroxbuster -u https://target.com -r --depth 3
```

## Wordlists & Extensions

```bash
# Custom wordlist
feroxbuster -u https://target.com -w /usr/share/wordlists/dirb/big.txt

# Test extensions
feroxbuster -u https://target.com -x php,html,txt

# Exclude extensions
feroxbuster -u https://target.com -X pdf,png,jpg

# Force extensions on all URLs
feroxbuster -u https://target.com --force-extensions

# No extensions
feroxbuster -u https://target.com --no-extension
```

## Headers & Cookies

```bash
# Custom header
feroxbuster -u https://target.com -H "Authorization: Bearer token"

# Multiple headers
feroxbuster -u https://target.com -H "Auth: tok" -H "X-Api: key"

# Custom cookies
feroxbuster -u https://target.com -b "session=abc123; user=admin"

# Random User-Agent
feroxbuster -u https://target.com --random-agent

# Custom User-Agent
feroxbuster -u https://target.com --user-agent "Mozilla/5.0"
```

## HTTP Methods

```bash
# GET (default)
feroxbuster -u https://target.com

# POST
feroxbuster -u https://target.com -m POST -d "FUZZ=test"

# PUT
feroxbuster -u https://target.com -m PUT -d "FUZZ=test"
```

## Filtering

```bash
# Include status codes
feroxbuster -u https://target.com -s 200,301,302

# Exclude status codes
feroxbuster -u https://target.com -S 404

# Word count filter
feroxbuster -u https://target.com -W 42

# Line count filter
feroxbuster -u https://target.com -L 10-100

# Size filter
feroxbuster -u https://target.com --size 500

# Auto-calibrate (filter custom 404s)
feroxbuster -u https://target.com -C

# Auto-tune
feroxbuster -u https://target.com --auto-tune

# Smart filter
feroxbuster -u https://target.com --smart-filter
```

## Recursion

```bash
# Enable recursion
feroxbuster -u https://target.com -r

# Recursive depth
feroxbuster -u https://target.com -r --depth 3

# Limit concurrent scans
feroxbuster -u https://target.com -r --scan-limit 5

# Parallel scans
feroxbuster -u https://target.com -r --parallel-scan 10

# Force recursion
feroxbuster -u https://target.com --force-recursion

# Disable recursion
feroxbuster -u https://target.com --no-recursion
```

## Performance

```bash
# Thread count
feroxbuster -u https://target.com -t 100

# Rate limit
feroxbuster -u https://target.com --rate-limit 100

# Timeout
feroxbuster -u https://target.com --timeout 10

# Delay between requests
feroxbuster -u https://target.com --delay 100ms

# Concurrency
feroxbuster -u https://target.com -c 20
```

## Output

```bash
# Save to file
feroxbuster -u https://target.com -o results.txt

# JSON output
feroxbuster -u https://target.com -O results.json

# CSV output
feroxbuster -u https://target.com --csv -o results.csv

# Silent mode (URLs only)
feroxbuster -u https://target.com --silent

# Quiet mode
feroxbuster -u https://target.com --quiet

# Verbose
feroxbuster -u https://target.com -v

# Very verbose
feroxbuster -u https://target.com -vv

# Debug mode
feroxbuster -u https://target.com --debug

# JSON to stdout
feroxbuster -u https://target.com --json
```

## Network

```bash
# HTTP proxy
feroxbuster -u https://target.com --proxy http://127.0.0.1:8080

# SOCKS5 proxy
feroxbuster -u https://target.com --proxy socks5://127.0.0.1:9050

# Skip TLS verification
feroxbuster -u https://target.com -k

# Force HTTP/1.1
feroxbuster -u https://target.com --http
```

## Configuration

```bash
# Custom config file
feroxbuster -u https://target.com --config my_config.toml

# Resume interrupted scan
feroxbuster --resume ferox-state.json

# Crawl for links
feroxbuster -u https://target.com --crawl
```

## Integration Pipelines

```bash
# Feroxbuster + nuclei
feroxbuster -u http://target.com --silent | \
    nuclei -l - -t nuclei-templates/

# Feroxbuster + sqlmap
feroxbuster -u http://target.com -e php --silent | \
    xargs -I{} sqlmap -u "{}?id=1" --batch

# Feroxbuster + httpx
feroxbuster -u http://target.com --silent | \
    httpx -silent

# Feroxbuster + ffuf
feroxbuster -u http://target.com --silent | \
    ffuf -u FUZZ -w - -mc 200

# Feroxbuster + waybackurls
feroxbuster -u http://target.com --silent | \
    waybackurls | sort -u
```

## Common Flag Combinations

| Use Case | Command |
|----------|---------|
| Quick scan | `feroxbuster -u URL` |
| Thorough scan | `feroxbuster -u URL -w big.txt -r --depth 3` |
| Authenticated | `feroxbuster -u URL -H "Auth: Bearer TOKEN" -b "session=ID"` |
| API testing | `feroxbuster -u URL/api/ -e json,xml -m POST -d '{}'` |
| Stealth | `feroxbuster -u URL --proxy http://127.0.0.1:8080 --random-agent --rate-limit 10` |
| Backup files | `feroxbuster -u URL -x bak,old,backup --silent` |
| Recursive deep | `feroxbuster -u URL -r -R 4 -e php -t 20 -o results.txt` |
| Auto-calibrated | `feroxbuster -u URL -C -r --depth 2` |
| JSON output | `feroxbuster -u URL -O results.json -s 200,301` |
| Resume scan | `feroxbuster --resume ferox-state.json` |

## Batch Operations

```bash
# Scan multiple targets
for url in $(cat targets.txt); do
    feroxbuster -u "$url" -o "${url##*/}.txt"
done

# Parallel scanning
cat targets.txt | xargs -P 5 -I{} sh -c 'feroxbuster -u "{}" -o "{}_dirs.txt"'

# Multi-wordlist scan
for wl in common.txt big.txt; do
    feroxbuster -u https://target.com -w /usr/share/wordlists/dirb/$wl -o "ferox_$wl.txt"
done
```
