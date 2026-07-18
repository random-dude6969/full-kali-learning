---
tool_name: "dirsearch"
display_name: "DirSearch Cheatsheet"
---

# DirSearch — Quick Reference Cheatsheet

## Basic Usage

```bash
# Basic scan
dirsearch -u https://target.com

# Custom wordlist
dirsearch -u https://target.com -w /usr/share/wordlists/dirb/big.txt

# Custom extensions
dirsearch -u https://target.com -e php,html,txt,bak

# Recursive scan
dirsearch -u https://target.com -r -R 2
```

## Output

```bash
# Save to file
dirsearch -u https://target.com -o results.txt

# JSON output
dirsearch -u https://target.com --json results.json

# CSV output
dirsearch -u https://target.com --csv results.csv

# Quiet mode
dirsearch -u https://target.com -q

# Verbose
dirsearch -u https://target.com -v
```

## HTTP Methods

```bash
# GET (default)
dirsearch -u https://target.com

# POST
dirsearch -u https://target.com -m POST -d "test=test"

# PUT
dirsearch -u https://target.com -m PUT -d "test=test"

# Custom method
dirsearch -u https://target.com -m PATCH -d '{"test":"test"}'
```

## Headers & Cookies

```bash
# Custom header
dirsearch -u https://target.com -H "Authorization: Bearer token"

# Multiple headers
dirsearch -u https://target.com -H "Auth: tok" -H "X-Api: key"

# Headers from file
dirsearch -u https://target.com --header-file headers.txt

# Custom cookies
dirsearch -u https://target.com --cookie "session=abc123"

# Random User-Agent
dirsearch -u https://target.com --random-agent
```

## Filtering

```bash
# Exclude status codes
dirsearch -u https://target.com --exclude-status 404,403

# Include only specific codes
dirsearch -u https://target.com --include-status 200,301,302

# Exclude response sizes
dirsearch -u https://target.com --exclude-sizes 0,1234

# Exclude text in response
dirsearch -u https://target.com --exclude-text "not found"

# Exclude regex
dirsearch -u https://target.com --exclude-regex "error.*"

# Min/max response size
dirsearch -u https://target.com --minimal 100 --maximal 5000
```

## Network

```bash
# HTTP proxy
dirsearch -u https://target.com --proxy http://127.0.0.1:8080

# SOCKS5 proxy
dirsearch -u https://target.com --socks5 socks5://127.0.0.1:9050

# Tor
dirsearch -u https://target.com --tor

# Rotating proxies
dirsearch -u https://target.com --proxy-file proxies.txt

# Timeout
dirsearch -u https://target.com --timeout 15

# Delay
dirsearch -u https://target.com --delay 1

# Rate limit
dirsearch -u https://target.com --rate-limit 100
```

## Recursive Scanning

```bash
# Recursive
dirsearch -u https://target.com -r

# Recursive with depth
dirsearch -u https://target.com -r -R 3

# Recursive with extensions
dirsearch -u https://target.com -r -e php -R 2
```

## Configuration

```bash
# Custom config file
dirsearch -u https://target.com -c my_config.ini

# Force extensions
dirsearch -u https://target.com --force-extensions

# No extensions
dirsearch -u https://target.com --no-extension

# Lowercase only
dirsearch -u https://target.com --lowercase

# Follow redirects
dirsearch -u https://target.com -F
```

## Integration Pipelines

```bash
# DirSearch + sqlmap
dirsearch -u https://target.com -e php -o dirs.txt && \
grep "^  " dirs.txt | awk '{print $4}' | \
    sed 's|^|https://target.com|' | \
    xargs -I{} sqlmap -u "{}" --batch

# DirSearch + nuclei
dirsearch -u https://target.com -o dirs.txt && \
grep "^  " dirs.txt | awk '{print $4}' | \
    sed 's|^|https://target.com|' | \
    nuclei -l - -t nuclei-templates/

# DirSearch + httpx
dirsearch -u https://target.com -o dirs.txt && \
grep "^  " dirs.txt | awk '{print $4}' | \
    sed 's|^|https://target.com|' | \
    httpx -silent

# DirSearch + ffuf
dirsearch -u https://target.com -e php -o dirs.txt && \
grep "^  " dirs.txt | awk '{print $4}' | \
    sed 's|^|https://target.com|' | \
    ffuf -u FUZZ -w - -mc 200
```

## Common Flag Combinations

| Use Case | Command |
|----------|---------|
| Quick scan | `dirsearch -u URL` |
| Thorough scan | `dirsearch -u URL -w big.txt -e php,html,txt -r -R 2` |
| Authenticated | `dirsearch -u URL -H "Auth: Bearer TOKEN" --cookie "session=ID"` |
| API testing | `dirsearch -u URL/api/ -e json,xml -m POST -d '{}'` |
| Stealth | `dirsearch -u URL --proxy http://127.0.0.1:8080 --random-agent --delay 1` |
| Backup files | `dirsearch -u URL -e bak,old,backup,save --exclude-status 404` |
| Recursive deep | `dirsearch -u URL -r -R 4 -e php -t 20 -o results.txt` |
