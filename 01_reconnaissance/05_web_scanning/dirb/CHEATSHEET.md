---
tool_name: "dirb"
display_name: "DIRB Cheatsheet"
---

# DIRB — Quick Reference Cheatsheet

## Basic Usage

```bash
# Basic scan (default wordlist)
dirb https://target.com

# Custom wordlist
dirb https://target.com /usr/share/wordlists/dirb/big.txt

# Custom extensions
dirb https://target.com -X .php,.html,.txt

# Recursive scan
dirb https://target.com -r
```

## Wordlists

```bash
# Default wordlist
dirb https://target.com

# Big wordlist
dirb https://target.com /usr/share/wordlists/dirb/big.txt

# Custom wordlist
dirb https://target.com /path/to/wordlist.txt

# Recursive with depth limit
dirb https://target.com -r -R 3
```

## Extensions

```bash
# Test PHP files
dirb https://target.com -X .php

# Multiple extensions
dirb https://target.com -X .php,.html,.txt,.bak

# Force extensions on all URLs
dirb https://target.com -Z

# Backup files
dirb https://target.com -X .bak,.old,.backup,.save
```

## Authentication

```bash
# Custom User-Agent
dirb https://target.com -a "Mozilla/5.0 (Windows NT 10.0)"

# Custom header
dirb https://target.com -H "Authorization: Bearer token"

# Custom cookies
dirb https://target.com -z "session=abc123; user=admin"
```

## Network

```bash
# HTTP proxy
dirb https://target.com -p http://127.0.0.1:8080

# Timeout
dirb https://target.com -t 30

# Don't wait between requests
dirb https://target.com -w
```

## Output

```bash
# Save to file
dirb https://target.com -o results.txt

# Verbose mode
dirb https://target.com -v

# Silent mode
dirb https://target.com -S
```

## Ignore Response Codes

```bash
# Ignore 404
dirb https://target.com -N 404

# Ignore 403 and 404
dirb https://target.com -N 403,404

# Ignore multiple codes
dirb https://target.com -N 403,404,500
```

## Integration Pipelines

```bash
# DIRB + grep (filter 200 OK)
dirb https://target.com -o results.txt
grep "(CODE:200" results.txt

# DIRB + sqlmap
dirb https://target.com -o dirs.txt
grep "^+ " dirs.txt | awk '{print $2}' | while read url; do
    sqlmap -u "$url?id=1" --batch
done

# DIRB + nikto
dirb https://target.com -o dirs.txt
grep "^+ " dirs.txt | awk '{print $2}' | while read url; do
    nikto -h "$url"
done

# DIRB + nmap
dirb https://target.com -o dirs.txt
grep "^+ " dirs.txt | awk '{print $2}' | \
    grep -oP 'https?://[^/]+' | sort -u | \
    nmap -iL - -T4 -p 80,443
```

## Common Flag Combinations

| Use Case | Command |
|----------|---------|
| Quick scan | `dirb https://target.com` |
| Thorough scan | `dirb https://target.com /usr/share/wordlists/dirb/big.txt -r` |
| PHP files only | `dirb https://target.com -X .php` |
| Backup files | `dirb https://target.com -X .bak,.old,.backup` |
| Authenticated | `dirb https://target.com -H "Auth: Bearer TOKEN"` |
| Stealth | `dirb https://target.com -p http://127.0.0.1:8080 -a "Mozilla/5.0" -w` |
| Recursive | `dirb https://target.com -r -R 2 -o results.txt` |
