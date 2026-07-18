# Gobuster Cheatsheet

## Quick Reference

| Category | Command | Description |
|----------|---------|-------------|
| **Dir Mode** | `gobuster dir -u http://target -w wordlist.txt` | Directory discovery |
| **DNS Mode** | `gobuster dns -d target.com -w wordlist.txt` | DNS subdomain brute-force |
| **VHost Mode** | `gobuster vhost -u http://target -w wordlist.txt` | Virtual host discovery |
| **Extensions** | `gobuster dir -u http://target -w wordlist.txt -x php,html` | Add file extensions |
| **Filter** | `gobuster dir -u http://target -w wordlist.txt -b 404` | Exclude status codes |
| **Threads** | `gobuster dir -u http://target -w wordlist.txt -t 20` | Set thread count |
| **Verbose** | `gobuster dir -u http://target -w wordlist.txt -v` | Verbose output |

---

## Mode Commands

### Directory Mode
```bash
gobuster dir -u http://target.example.com -w /usr/share/wordlists/dirb/common.txt
```

### DNS Mode
```bash
gobuster dns -d target.example.com -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt
```

### Virtual Host Mode
```bash
gobuster vhost -u http://target.example.com -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt
```

---

## Essential Flags

### Dir Mode Flags
| Flag | Description |
|------|-------------|
| `-u` | Target URL |
| `-w` | Wordlist path |
| `-x` | File extensions (e.g., php,html,txt) |
| `-s` | Status codes to include |
| `-b` | Status codes to exclude |
| `-r` | Follow redirects |
| `-k` | Skip TLS verification |
| `-f` | Append / to every request |
| `-l` | Include response length |
| `--wildcard` | Force wildcard mode |
| `--no-tls-validation` | Skip TLS validation |
| `-o` | Output file |
| `-v` | Verbose output |
| `-n` | Don't print progress |
| `-e` | Print full URLs |

### DNS Mode Flags
| Flag | Description |
|------|-------------|
| `-d` | Target domain |
| `-w` | Wordlist path |
| `-r` | Custom DNS resolver |
| `--wildcard` | Force wildcard mode |
| `-c` | Show CNAME records |
| `--show-ips` | Show IP addresses |
| `-o` | Output file |
| `-v` | Verbose output |

### VHost Mode Flags
| Flag | Description |
|------|-------------|
| `-u` | Target URL |
| `-w` | Wordlist path |
| `-k` | Skip TLS verification |
| `--wildcard` | Force wildcard mode |
| `--append-domain` | Append domain to words |
| `--exclude-length` | Exclude response length |
| `-o` | Output file |
| `-v` | Verbose output |

### Global Flags
| Flag | Description |
|------|-------------|
| `-t` | Number of threads (default: 10) |
| `--timeout` | HTTP request timeout |
| `--delay` | Delay between requests |
| `-p` | URL path prefix |
| `--proxy` | HTTP proxy |
| `-a` | User-Agent string |
| `-c` | Cookie data |
| `-U` | Basic auth username |
| `-P` | Basic auth password |
| `-k` | Skip TLS verification |
| `-x` | File extensions |
| `-o` | Output file |
| `-v` | Verbose output |

---

## Common Combinations

```bash
# Directory discovery with extensions
gobuster dir -u http://target -w wordlist.txt -x php,html,txt,bak

# Hide 404 and 403
gobuster dir -u http://target -w wordlist.txt -b 404,403

# Only show 200 and 301
gobuster dir -u http://target -w wordlist.txt -s 200,301

# Recursive scan
gobuster dir -u http://target -w wordlist.txt -r

# With proxy
gobuster dir -u http://target -w wordlist.txt --proxy http://proxy:8080

# DNS with custom resolver
gobuster dns -d target.com -w wordlist.txt -r 8.8.8.8 --show-ips

# VHost with response filtering
gobuster vhost -u http://target -w wordlist.txt --exclude-length 0
```

---

## Wordlist Locations

| Wordlist | Path |
|----------|------|
| DirB common | `/usr/share/wordlists/dirb/common.txt` |
| DirB big | `/usr/share/wordlists/dirb/big.txt` |
| SecLists common | `/usr/share/wordlists/seclists/Discovery/Web-Content/common.txt` |
| SecLists big | `/usr/share/wordlists/seclists/Discovery/Web-Content/big.txt` |
| DNS subdomains 5k | `/usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt` |
| DNS subdomains 20k | `/usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-20000.txt` |
| DNS subdomains 110k | `/usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-110000.txt` |

---

## Quick One-Liners

```bash
# Find admin panels
gobuster dir -u http://target -w /usr/share/wordlists/dirb/common.txt -s 200,301,302

# Find backup files
gobuster dir -u http://target -w /usr/share/wordlists/seclists/Discovery/Web-Content/backup.txt -b 404

# Find config files
gobuster dir -u http://target -w /usr/share/wordlists/dirb/common.txt -x config,conf,ini,yml,yaml,json -b 404

# Find hidden directories
gobuster dir -u http://target -w /usr/share/wordlists/dirb/big.txt -b 404

# Enumerate subdomains
gobuster dns -d target.com -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt --show-ips

# Find virtual hosts
gobuster vhost -u http://target -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt

# Find API endpoints
gobuster dir -u http://target/api -w /usr/share/wordlists/seclists/Discovery/Web-Content/api/api-endpoints.txt -b 404
```

---

## Output Processing

```bash
# Extract only found paths
gobuster dir -u http://target -w wordlist.txt | grep "/" | awk '{print $1}'

# Count results by status code
gobuster dir -u http://target -w wordlist.txt | grep -oP 'Status: \K[0-9]+' | sort | uniq -c

# Sort by response size
gobuster dir -u http://target -w wordlist.txt | sort -t'[' -k2 -n

# Save and analyze
gobuster dir -u http://target -w wordlist.txt -o results.txt
cat results.txt | grep -v "^=" | grep -v "^Gobuster" | grep -v "^$" > clean_results.txt
```

---

## Performance Tuning

```bash
# Maximum speed
gobuster dir -u http://target -w wordlist.txt -t 100

# Balanced
gobuster dir -u http://target -w wordlist.txt -t 20

# Stealth (slow)
gobuster dir -u http://target -w wordlist.txt -t 5 --delay 200ms

# Large wordlist
gobuster dir -u http://target -w /usr/share/wordlists/dirb/big.txt -t 50 -n
```

---

## Evasion Techniques

```bash
# Rate limiting
gobuster dir -u http://target -w wordlist.txt --delay 500ms -t 5

# Use proxy
gobuster dir -u http://target -w wordlist.txt --proxy http://proxy:8080

# Random User-Agent
gobuster dir -u http://target -w wordlist.txt --random-agent

# Custom User-Agent
gobuster dir -u http://target -w wordlist.txt -a "Mozilla/5.0 (Windows NT 10.0; Win64; x64)"

# With cookies
gobuster dir -u http://target -w wordlist.txt -c "session=abc123"

# Basic auth
gobuster dir -u http://target -w wordlist.txt -U admin -P password
```

---

## Error Handling

```bash
# Skip TLS errors
gobuster dir -u https://target -w wordlist.txt -k

# Force wildcard mode
gobuster dir -u http://target -w wordlist.txt --wildcard

# Suppress errors
gobuster dir -u http://target -w wordlist.txt --no-error

# Custom timeout
gobuster dir -u http://target -w wordlist.txt --timeout 30s
```
