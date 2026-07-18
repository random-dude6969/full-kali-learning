---
tool_name: "arjun"
display_name: "Arjun Cheatsheet"
---

# Arjun — Quick Reference Cheatsheet

## Basic Usage

```bash
# Basic parameter discovery
arjun -u https://target.com

# GET parameters
arjun -u https://target.com -m GET

# POST parameters
arjun -u https://target.com -m POST -d "test=test"

# JSON parameters
arjun -u https://target.com -m JSON -d '{"test":"test"}'

# XML parameters
arjun -u https://target.com -m XML -d '<test>test</test>'
```

## Output

```bash
# Save to JSON
arjun -u https://target.com -o results.json

# Verbose output
arjun -u https://target.com -v

# Silent mode
arjun -u https://target.com --no-logging
```

## Scan Control

```bash
# Custom wordlist
arjun -u https://target.com -w /path/to/wordlist.txt

# Thread count
arjun -u https://target.com -t 20

# Timeout
arjun -u https://target.com --timeout 15

# Delay between requests
arjun -u https://target.com --delay 1000

# Stable mode
arjun -u https://target.com --stable

# Skip known params
arjun -u https://target.com --skip "id,token"
```

## Headers & Cookies

```bash
# Custom header
arjun -u https://target.com -H "Authorization: Bearer token"

# Multiple headers
arjun -u https://target.com -H "Auth: tok" -H "X-Api: key"

# Custom cookies
arjun -u https://target.com -b "session=abc123"

# Cookies (alternate)
arjun -u https://target.com --cookies "lang=en; theme=dark"
```

## Network

```bash
# HTTP proxy
arjun -u https://target.com --proxy http://127.0.0.1:8080

# Tor
arjun -u https://target.com --tor
```

## Comparison Mode

```bash
# Enable comparison mode
arjun -u https://target.com --compare

# Compare with existing params
arjun -u https://target.com --compare --include "lang,theme"
```

## Batch Scanning

```bash
# Scan multiple URLs from file
arjun -i urls.txt

# Batch scan with output
arjun -i urls.txt -o batch_results.json
```

## Integration Pipelines

```bash
# Arjun + sqlmap
arjun -u https://target.com -o params.json && \
cat params.json | python3 -c "
import sys,json
d=json.load(sys.stdin)
for u,m in d.items():
    for met,p in m.items():
        for param in p:
            print(f'{u}?{param}=test')
" | xargs -I{} sqlmap -u "{}" --batch --random-agent

# Arjun + nuclei
arjun -u https://target.com -o p.json && \
cat p.json | python3 -c "
import sys,json; d=json.load(sys.stdin)
for u in d: print(u)
" | nuclei -l - -t nuclei-templates/
```

## Common Flag Combinations

| Use Case | Command |
|----------|---------|
| Quick scan | `arjun -u URL` |
| Thorough scan | `arjun -u URL -t 5 --timeout 30 --stable` |
| Authenticated scan | `arjun -u URL -H "Auth: Bearer TOKEN" -b "session=ID"` |
| JSON API testing | `arjun -u URL -m JSON -d '{"test":"1"}'` |
| Stealth scan | `arjun -u URL -t 2 --delay 2000 --proxy http://127.0.0.1:8080` |
| Full audit | `arjun -u URL -m GET -o get.json && arjun -u URL -m POST -d "t=1" -o post.json && arjun -u URL -m JSON -d '{"t":"1"}' -o json.json` |
