# HTTrack Website Copier Cheatsheet

## Quick Start

```bash
# Mirror a website
httrack http://www.example.com

# Mirror to specific directory
httrack http://www.example.com -O /tmp/mirror

# Continue interrupted mirror
httrack -i

# Update existing mirror
httrack --update
```

## Essential Commands

| Command | Description |
|---------|-------------|
| `httrack <URL>` | Mirror website |
| `httrack <URL> -O <dir>` | Mirror to directory |
| `httrack -i` | Continue interrupted mirror |
| `httrack --update` | Update existing mirror |
| `httrack --clean` | Clear cache and logs |
| `httrack <URL> --spider` | Test links without downloading |
| `httrack <URL> -g` | Get specific file only |

## Mirroring Options

```bash
# Full mirror
httrack http://www.example.com

# With depth limit
httrack http://www.example.com -r6

# Same domain only
httrack http://www.example.com -d

# Same address only
httrack http://www.example.com -a

# All links from first pages
httrack http://www.example.com -Y
```

## Filtering

```bash
# Include specific file types
httrack http://www.example.com +*.html +*.css +*.js

# Exclude file types
httrack http://www.example.com -mime:video/* -mime:application/*

# Include URL patterns
httrack http://www.example.com +*.example.com/admin/*

# Exclude URL patterns
httrack http://www.example.com -logout -admin -delete
```

## Depth and Scope

| Flag | Description | Example |
|------|-------------|---------|
| `-rN` | Mirror depth | `-r6` |
| `%eN` | External link depth | `%e2` |
| `-a` | Stay on same address | `-a` |
| `-d` | Stay on same domain | `-d` |
| `-l` | Stay on same TLD | `-l` |
| `-e` | Go everywhere | `-e` |
| `-S` | Stay on same directory | `-S` |
| `-D` | Can only go down | `-D` |
| `-U` | Can only go up | `-U` |
| `-B` | Can go up and down | `-B` |

## Flow Control

```bash
# Connection limit
httrack http://www.example.com c16

# Timeout
httrack http://www.example.com T60

# Max file size (5MB)
httrack http://www.example.com m5000000

# Max mirror size (100MB)
httrack http://www.example.com M100000000

# Transfer rate limit (10KB/s)
httrack http://www.example.com A10000

# Connections per second
httrack http://www.example.com %c5
```

## Browser ID Spoofing

```bash
# Custom User-Agent
httrack http://www.example.com -F "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36"

# Custom Referer
httrack http://www.example.com %R "http://www.google.com"

# Custom headers
httrack http://www.example.com %X "X-Forwarded-For: 1.2.3.4"
```

## Proxy Usage

```bash
# HTTP proxy
httrack http://www.example.com -P proxy.example.com:8080

# Authenticated proxy
httrack http://www.example.com -P user:pass@proxy.example.com:8080
```

## Structure Types

| Type | Description |
|------|-------------|
| `N0` | Original structure (default) |
| `N1` | HTML in web/, images in web/images/ |
| `N2` | HTML in web/HTML, images in web/images |
| `N3` | HTML in web/, images in web/ |
| `N4` | HTML in web/, files by extension |
| `N99` | All files in web/, random names |

```bash
# Custom structure
httrack http://www.example.com -N "%h%p/%n%q.%t"
```

## Spider/Testing

```bash
# Spider mode (test links only)
httrack http://www.example.com --spider

# Test specific URL
httrack http://www.example.com --test

# Test all links
httrack http://www.example.com --testlinks
```

## Quick Attack Workflows

### Clone Login Page for Phishing

```bash
httrack "https://login.target.com/*" \
  -O /tmp/phishing_clone \
  -F "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36" \
  +*.target.com/login* \
  +*.target.com/static/* \
  -mime:application/* \
  -r4

# Serve the clone
cd /tmp/phishing_clone && python3 -m http.server 8080
```

### Offline Vulnerability Analysis

```bash
httrack http://internal-app.local -O /tmp/app_mirror -r6

# Search for vulnerabilities
grep -rn "mysql_query\|eval\|exec" /tmp/app_mirror --include="*.php"
grep -rn "password\|api_key\|secret" /tmp/app_mirror --include="*.js"
```

### Technology Fingerprinting

```bash
httrack http://www.target.com --spider -v

# Check technologies
grep -rn "wp-content\|joomla\|drupal" /tmp/mirror --include="*.html"
grep -rn "react\|angular\|vue\|jquery" /tmp/mirror --include="*.js"
```

### Complete Site Archive

```bash
httrack "https://docs.company.com/*" \
  -O /tmp/docs_archive \
  -r8 \
  -L1 \
  -c32 \
  -T60 \
  +*.html +*.pdf +*.css +*.js +*.png +*.jpg \
  --search-index
```

## Output Structure

```
mirrored-site/
├── index.html
├── index.html(original)
├── hts-cache/
│   ├── new.txt
│   └── test.txt
├── hts-log.txt
├── images/
├── css/
├── js/
└── subdirectory/
```

## Shortcuts Reference

| Shortcut | Description |
|----------|-------------|
| `--mirror` | Mirror websites (default) |
| `--get` | Get files without seeking other URLs |
| `--mirrorlinks` | Mirror all links in first-level pages |
| `--testlinks` | Test links in pages |
| `--spider` | Spider site(s) to test links |
| `--skeleton` | Mirror only HTML files |
| `--update` | Update mirror without confirmation |
| `--continue` | Continue mirror without confirmation |
| `--catchurl` | Create temporary proxy to capture URL |
| `--clean` | Erase cache and log files |

## WebHTTrack (GUI)

```bash
# Start web interface
sudo webhttrack

# Or start server directly
htsserver /usr/share/httrack/

# Access at http://localhost:8080/
```

## Troubleshooting

```bash
# Not following links
httrack http://www.example.com -r10 -v

# Login-protected content
httrack http://www.example.com -b "session_id=ABC123"

# Too many files
httrack http://www.example.com m1000000 +*.html +*.css

# Broken mirror
httrack http://www.example.com N0 K4

# Connection refused
httrack http://www.example.com T120 c4

# Debug
httrack http://www.example.com -vz 2>&1 | tee debug.log
httrack http://www.example.com %H
```

## Cache Management

```bash
# Clear cache
httrack --clean
rm -rf hts-cache/

# Force re-download
httrack http://www.example.com C0

# View cache
ls hts-cache/
cat hts-cache/new.txt
```

## Log Analysis

```bash
# View log
cat hts-log.txt

# Search errors
grep -i "error\|404\|403" hts-log.txt

# Count files
wc -l hts-log.txt

# Statistics
tail -20 hts-log.txt
```

## Packages

| Package | Description |
|---------|-------------|
| `httrack` | CLI website copier |
| `httrack-doc` | HTML documentation |
| `libhttrack-dev` | Development headers |
| `libhttrack2` | Core library |
| `webhttrack` | Web-based GUI |
| `proxytrack` | HTTP cache proxy |

## Related Tools

| Tool | Purpose |
|------|---------|
| `wget` | Simple file download |
| `curl` | URL transfer |
| `photon` | Web crawler |
| `gowitness` | Screenshot tool |
| `burpsuite` | Web proxy |
