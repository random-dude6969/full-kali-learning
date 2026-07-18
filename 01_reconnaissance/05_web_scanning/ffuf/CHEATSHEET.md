# FFuF Cheatsheet

## Quick Reference

| Category | Command | Description |
|----------|---------|-------------|
| **Basic Scan** | `ffuf -u http://target/FUZZ -w wordlist.txt` | Basic directory discovery |
| **Extensions** | `ffuf -u http://target/FUZZ -w wordlist.txt -e .php,.html` | Add file extensions |
| **Filter 404** | `ffuf -u http://target/FUZZ -w wordlist.txt -fc 404` | Hide 404 responses |
| **Filter Size** | `ffuf -u http://target/FUZZ -w wordlist.txt -fs 0` | Hide zero-size responses |
| **VHost** | `ffuf -u http://IP -H "Host: FUZZ.target.com" -w subs.txt` | Virtual host discovery |
| **POST Data** | `ffuf -u http://target/login -X POST -d "user=admin&pass=FUZZ" -w passwords.txt` | POST fuzzing |
| **Parameter** | `ffuf -u "http://target/page?id=FUZZ" -w params.txt` | Parameter fuzzing |
| **JSON Output** | `ffuf -u http://target/FUZZ -w wordlist.txt -o results.json -of json` | Save as JSON |

---

## Mode Commands

### Directory Discovery
```bash
ffuf -u http://target.example.com/FUZZ -w /usr/share/wordlists/dirb/common.txt
```

### Virtual Host Discovery
```bash
ffuf -u http://192.168.1.1 -H "Host: FUZZ.target.com" -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt -fs 0
```

### Parameter Fuzzing
```bash
ffuf -u "http://target.example.com/page?id=FUZZ" -w /usr/share/wordlists/seclists/Fuzzing/dentions.txt
```

### POST Data Fuzzing
```bash
ffuf -u http://target.example.com/login -X POST -d "user=admin&pass=FUZZ" -w /usr/share/wordlists/rockyou.txt -fc 401
```

### Subdomain Brute-Force
```bash
ffuf -u http://IP_ADDRESS -H "Host: FUZZ.target.com" -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt -fs 0
```

---

## Essential Flags

### Input Options
| Flag | Description |
|------|-------------|
| `-w` | Wordlist path |
| `-u` | Target URL with FUZZ keyword |
| `-e` | File extensions to append |
| `-enc` | Encode payloads (url,html,base64) |
| `-recursion` | Enable recursion |
| `-recursion-depth` | Max recursion depth |
| `-ic` | Ignore comments in wordlist |
| `-request` | Request file |

### Matcher Options
| Flag | Description |
|------|-------------|
| `-mc` | Match status codes (default: 200,204,301,302,303,307,401,403,405) |
| `-ms` | Match response size |
| `-mw` | Match word count |
| `-ml` | Match line count |
| `-mr` | Match regex |
| `-mt` | Match response time |

### Filter Options
| Flag | Description |
|------|-------------|
| `-fc` | Filter status codes |
| `-fs` | Filter response size |
| `-fw` | Filter word count |
| `-fl` | Filter line count |
| `-fr` | Filter regex |
| `-ft` | Filter response time |

### Output Options
| Flag | Description |
|------|-------------|
| `-o` | Output file |
| `-of` | Output format (json,ejson,html,md,csv,all) |
| `-od` | Output directory |

### General Options
| Flag | Description |
|------|-------------|
| `-t` | Number of threads (default: 40) |
| `-p` | Delay between requests |
| `-rate` | Requests per second |
| `-s` | Silent mode |
| `-v` | Verbose output |
| `-c` | Colorize output |
| `-ac` | Auto-calibrate |
| `-ic` | Ignore comments |
| `-timeout` | Request timeout (seconds) |

### HTTP Options
| Flag | Description |
|------|-------------|
| `-X` | HTTP method |
| `-d` | POST data |
| `-H` | Custom header |
| `-b` | Cookie data |
| `-auth` | Basic auth |
| `-r` | Follow redirects |
| `-x` | HTTP proxy |

---

## Common Filter Combinations

```bash
# Hide default pages (404 + common sizes)
ffuf -u http://target/FUZZ -w wordlist.txt -fc 404 -fs 0

# Only show 200 and 301
ffuf -u http://target/FUZZ -w wordlist.txt -mc 200,301

# Hide 403 and 404
ffuf -u http://target/FUZZ -w wordlist.txt -fc 403,404

# Match specific size (exclude default page)
ffuf -u http://target/FUZZ -w wordlist.txt -fs 1234

# Only show responses with "admin" in content
ffuf -u http://target/FUZZ -w wordlist.txt -mr "admin"
```

---

## Wordlist Locations

| Wordlist | Path |
|----------|------|
| DirB common | `/usr/share/wordlists/dirb/common.txt` |
| DirB big | `/usr/share/wordlists/dirb/big.txt` |
| SecLists common | `/usr/share/wordlists/seclists/Discovery/Web-Content/common.txt` |
| SecLists big | `/usr/share/wordlists/seclists/Discovery/Web-Content/big.txt` |
| SecLists API | `/usr/share/wordlists/seclists/Discovery/Web-Content/api/` |
| DNS subdomains | `/usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt` |
| Parameters | `/usr/share/wordlists/seclists/Discovery/Web-Content/burp-parameter-names.txt` |

---

## Output Formats

```bash
# JSON
ffuf -u http://target/FUZZ -w wordlist.txt -o results.json -of json

# HTML report
ffuf -u http://target/FUZZ -w wordlist.txt -o results.html -of html

# CSV
ffuf -u http://target/FUZZ -w wordlist.txt -o results.csv -of csv

# Markdown
ffuf -u http://target/FUZZ -w wordlist.txt -o results.md -of md

# All formats
ffuf -u http://target/FUZZ -w wordlist.txt -o results -of all
```

---

## Quick One-Liners

```bash
# Find admin panels
ffuf -u http://target/FUZZ -w /usr/share/wordlists/dirb/common.txt -fc 404 -mc 200,301,302

# Find backup files
ffuf -u http://target/FUZZ -w /usr/share/wordlists/seclists/Discovery/Web-Content/backup.txt -fc 404

# Find config files
ffuf -u http://target/FUZZ -e .config,.conf,.ini,.yml,.yaml,.json -fc 404

# Find API endpoints
ffuf -u http://target/api/FUZZ -w /usr/share/wordlists/seclists/Discovery/Web-Content/api/api-endpoints.txt -fc 404

# Test for SQL injection
ffuf -u "http://target/page?id=FUZZ" -w /usr/share/wordlists/seclists/Fuzzing/SQL/SQLite_Special.txt -fc 404 -mc 200,500

# Find hidden directories with recursion
ffuf -u http://target/FUZZ -w wordlist.txt -recursion -recursion-depth 2 -fc 404
```

---

## Evasion Techniques

```bash
# Rate limiting
ffuf -u http://target/FUZZ -w wordlist.txt -rate 100 -p 0.5

# Use proxy
ffuf -u http://target/FUZZ -w wordlist.txt -x http://proxy:8080

# Custom User-Agent (via header)
ffuf -u http://target/FUZZ -w wordlist.txt -H "User-Agent: Mozilla/5.0"

# Delay between threads
ffuf -u http://target/FUZZ -w wordlist.txt -S -p 2

# Reduce threads
ffuf -u http://target/FUZZ -w wordlist.txt -t 10
```
