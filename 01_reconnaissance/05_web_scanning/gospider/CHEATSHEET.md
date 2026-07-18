# GoSpider Cheatsheet

## Quick Reference

| Category | Command | Description |
|----------|---------|-------------|
| **Basic Crawl** | `gospider -s http://target.com` | Basic web crawling |
| **Depth Limit** | `gospider -s http://target.com -d 3` | Crawl depth limit |
| **JS Analysis** | `gospider -s http://target.com --js` | JavaScript analysis |
| **Thread Control** | `gospider -s http://target.com -t 20` | Set thread count |
| **Output File** | `gospider -s http://target.com -o results.txt` | Save to file |
| **Subdomains** | `gospider -s http://target.com --include-subdomains` | Include subdomains |
| **Proxy** | `gospider -s http://target.com --proxy http://proxy:8080` | Use proxy |

---

## Basic Commands

### Web Crawling
```bash
gospider -s http://target.example.com
```

### Crawling with Depth Limit
```bash
gospider -s http://target.example.com -d 3
```

### JavaScript Analysis
```bash
gospider -s http://target.example.com --js
```

### Multiple URLs
```bash
gospider -s http://target.example.com -S urls.txt
```

### Include Subdomains
```bash
gospider -s http://target.example.com --include-subdomains
```

### Output to File
```bash
gospider -s http://target.example.com -o results.txt
```

### JSON Output
```bash
gospider -s http://target.example.com -j
```

---

## Essential Flags

### Target Options
| Flag | Description |
|------|-------------|
| `-s` | Starting URL |
| `-S` | URLs file |
| `--sitemap` | Follow sitemap.xml |
| `--robots` | Follow robots.txt |

### Crawling Options
| Flag | Description |
|------|-------------|
| `-d` | Crawl depth |
| `-t` | Number of threads |
| `--include-subdomains` | Include subdomains |
| `--js` | Analyze JavaScript |
| `--source` | Analyze source code |
| `--preload` | Preload all pages |
| `--no-redirect` | Don't follow redirects |
| `--no-head` | Don't use HEAD requests |
| `--no-ext` | Don't extract external links |

### Filtering Options
| Flag | Description |
|------|-------------|
| `--blacklist` | Blacklist keywords |
| `--whitelist` | Whitelist keywords |
| `--exclude` | Exclude URL patterns |
| `--regex` | Regex filter |
| `--filter-extensions` | Filter file extensions |

### Network Options
| Flag | Description |
|------|-------------|
| `-a` | User-Agent string |
| `--random-agent` | Random User-Agent |
| `--proxy` | HTTP proxy |
| `--cookie` | Cookie string |
| `--timeout` | Connection timeout |
| `--retry` | Retry count |
| `--delay` | Delay between requests |
| `--rate-limit` | Rate limit (req/sec) |

### Output Options
| Flag | Description |
|------|-------------|
| `-o` | Output file |
| `-j` | JSON output |
| `-c` | CSV output |
| `-v` | Verbose output |
| `-q` | Quiet mode |
| `--no-color` | Disable colors |

### Authentication Options
| Flag | Description |
|------|-------------|
| `-u` | Basic auth username |
| `-p` | Basic auth password |
| `--bearer` | Bearer token |

---

## Common Combinations

```bash
# Full crawl with JS analysis
gospider -s http://target.com --js --source -o results.txt

# Deep crawl with subdomains
gospider -s http://target.com -d 5 --include-subdomains -o deep.txt

# Crawl with proxy
gospider -s http://target.com --proxy http://proxy:8080 --random-agent

# Crawl with cookies
gospider -s http://target.com --cookie "session=abc123"

# Crawl with rate limiting
gospider -s http://target.com --delay 2 --rate-limit 5 -t 5

# Crawl with blacklisting
gospider -s http://target.com --blacklist "logout,admin,test"

# Crawl with whitelisting
gospider -s http://target.com --whitelist "api,v1,v2"

# Crawl with regex filter
gospider -s http://target.com --regex "api/.*"
```

---

## Quick One-Liners

```bash
# Quick crawl
gospider -s http://target.com -o links.txt

# Find JavaScript files
gospider -s http://target.com --js | grep '\.js' | sort -u > js_files.txt

# Find API endpoints
gospider -s http://target.com --js | grep -oP '/api/[^"?]+' | sort -u > api_endpoints.txt

# Find subdomains
gospider -s http://target.com --include-subdomains | \
    grep -oP 'https?://([a-zA-Z0-9-]+\.)+[a-zA-Z]{2,}' | \
    sed 's|https\?://||' | sort -u > subdomains.txt

# Find parameters
gospider -s http://target.com | grep -oP '[?&][a-zA-Z0-9_]+=' | \
    sed 's/[?&]=//' | sort -u > parameters.txt

# Deep crawl
gospider -s http://target.com -d 5 --include-subdomains --js -o full_crawl.txt

# Crawl with verbose output
gospider -s http://target.com -v -o verbose_crawl.txt

# Crawl multiple URLs
gospider -S urls.txt -o multi_crawl.txt
```

---

## Output Processing

```bash
# Extract only URLs
gospider -s http://target.com | grep '^http' | sort -u > urls.txt

# Extract paths only
gospider -s http://target.com | grep -oP 'https?://target\.com\K/[^"?]+' | sort -u > paths.txt

# Extract JavaScript files
gospider -s http://target.com --js | grep '\.js' | sort -u > js_files.txt

# Extract API endpoints
gospider -s http://target.com --js | grep -oP '/api/[^"?]+' | sort -u > api_endpoints.txt

# Extract subdomains
gospider -s http://target.com --include-subdomains | \
    grep -oP 'https?://([a-zA-Z0-9-]+\.)+[a-zA-Z]{2,}' | \
    sed 's|https\?://||' | sort -u > subdomains.txt

# Count results
gospider -s http://target.com | wc -l
```

---

## Batch Processing

```bash
# Crawl multiple targets
for url in http://target1.com http://target2.com http://target3.com; do
    gospider -s "$url" -o "$(echo $url | sed 's|https\?://||' | tr '/' '_').txt"
done

# Crawl from file
gospider -S urls.txt -o all_links.txt

# Parallel crawling
cat urls.txt | xargs -P 5 -I {} gospider -s {} -o {}.txt

# Crawl with depth
while read url; do
    gospider -s "$url" -d 3 -o "$(echo $url | md5sum | cut -d' ' -f1).txt"
done < urls.txt
```

---

## Integration with Other Tools

```bash
# GoSpider + FFuF
gospider -s http://target.com --js -o links.txt
cat links.txt | while read url; do
    path=$(echo "$url" | sed 's|http://target.com||')
    ffuf -u "http://target.com${path}/FUZZ" -w wordlist.txt -fc 404 -o "ffuf_$path.json" -of json
done

# GoSpider + Gobuster
gospider -s http://target.com -o links.txt
grep -oP 'https?://target\.com\K/[^"?]+' links.txt | sort -u > paths.txt
gobuster dir -u http://target.com -w paths.txt

# GoSpider + Nuclei
gospider -s http://target.com --js -o endpoints.txt
nuclei -l endpoints.txt -t cves/

# GoSpider + httpx
gospider -s http://target.com --include-subdomains -o all_links.txt
grep -oP 'https?://([a-zA-Z0-9-]+\.)+[a-zA-Z]{2,}' all_links.txt | \
    sed 's|https\?://||' | sort -u > subdomains.txt
httpx -l subdomains.txt -o live_hosts.txt -silent

# GoSpider + Amass
gospider -s http://target.com --include-subdomains -o gospider_subs.txt
amass enum -passive -d target.com -o amass_subs.txt
cat gospider_subs.txt amass_subs.txt | sort -u > all_subs.txt
```

---

## Performance Tuning

```bash
# Maximum speed
gospider -s http://target.com -t 100 --no-redirect

# Balanced
gospider -s http://target.com -t 20

# Stealth (slow)
gospider -s http://target.com -t 5 --delay 2 --rate-limit 5

# Memory efficient
gospider -s http://target.com -d 2 -t 10

# Large crawl
gospider -s http://target.com -d 10 --include-subdomains --js -t 50 -o large_crawl.txt
```

---

## Evasion Techniques

```bash
# Rate limiting
gospider -s http://target.com --delay 2 --rate-limit 5 -t 5

# Use proxy
gospider -s http://target.com --proxy http://proxy:8080

# Random User-Agent
gospider -s http://target.com --random-agent

# Custom User-Agent
gospider -s http://target.com -a "Mozilla/5.0 (Windows NT 10.0; Win64; x64)"

# With cookies
gospider -s http://target.com --cookie "session=abc123"

# With basic auth
gospider -s http://target.com -u admin -p password

# With bearer token
gospider -s http://target.com --bearer "token123"
```

---

## Error Handling

```bash
# Increase timeout
gospider -s http://target.com --timeout 60

# Retry failed requests
gospider -s http://target.com --retry 3

# Skip redirects
gospider -s http://target.com --no-redirect

# Use custom User-Agent
gospider -s http://target.com -a "Mozilla/5.0 (Windows NT 10.0; Win64; x64)"

# Use proxy
gospider -s http://target.com --proxy http://proxy:8080
```
