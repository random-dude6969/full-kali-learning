# EyeWitness - Quick Reference Cheatsheet

---

## Quick Start

```bash
# Single URL
eyewitness --single http://example.com -d screenshots --headless

# From file
eyewitness -f urls.txt -d screenshots --headless

# From Nmap XML
eyewitness -x scan.xml -d screenshots --headless
```

---

## Command-Line Options

### Input
| Option | Description |
|--------|-------------|
| `-f FILE` | URLs file (one per line) |
| `-x FILE` | Nmap/Nessus XML file |
| `--single URL` | Single URL |

### Output
| Option | Description |
|--------|-------------|
| `-d DIR` | Output directory |
| `--results N` | Hosts per page (default: 25) |
| `--no-prompt` | Don't prompt to open |

### Timing
| Option | Description |
|--------|-------------|
| `--timeout N` | Timeout seconds (default: 7) |
| `--delay N` | Delay before screenshot |
| `--jitter N` | Random delay between requests |
| `--max-retries N` | Max retries on timeout |

### Threading
| Option | Description |
|--------|-------------|
| `--threads N` | Number of threads (default: 1) |

### Web Options
| Option | Description |
|--------|-------------|
| `--user-agent AGENT` | Custom user agent |
| `--cookies KEY=VAL` | Add cookies |
| `--proxy-ip IP` | Proxy IP |
| `--proxy-port PORT` | Proxy port |
| `--proxy-type TYPE` | Proxy type (socks5/http) |
| `--no-dns` | Skip DNS resolution |
| `--prepend-https` | Add http:// to URLs |
| `--add-http-ports PORTS` | Additional HTTP ports |
| `--add-https-ports PORTS` | Additional HTTPS ports |
| `--only-ports PORTS` | Exclusive ports |

### Resume
| Option | Description |
|--------|-------------|
| `--resume ew.db` | Resume from database |

---

## Common Use Cases

```bash
# Mass scan with threading
eyewitness -f urls.txt -d screenshots --headless --threads 10

# Nmap XML with custom ports
eyewitness -x scan.xml -d screenshots \
    --add-http-ports 8080,8443 --add-https-ports 443,8443

# Authenticated pages
eyewitness -f urls.txt -d screenshots \
    --cookies "session=abc123; logged_in=true"

# With proxy
eyewitness -f urls.txt -d screenshots \
    --proxy-ip 127.0.0.1 --proxy-port 8080 --proxy-type http

# Custom user agent
eyewitness -f urls.txt -d screenshots \
    --user-agent "Mozilla/5.0 (compatible; MSIE 10.0)"
```

---

## Nmap Integration

```bash
# Generate Nmap XML
nmap -sV -sC -p- -oX scan.xml target.com

# Process with EyeWitness
eyewitness -x scan.xml -d screenshots --headless

# Multiple targets
nmap -iL targets.txt -sV -oX all_scans.xml
eyewitness -x all_scans.xml -d screenshots --headless --threads 10
```

---

## Output Structure

```
screenshots/
├── Eyewitness.html        # Main report
├── style.css              # Report styling
├── screenshots/           # Screenshot images
│   ├── http___example_com.png
│   └── ...
└── Evidence.db            # SQLite database
```

---

## Batch Script

```bash
#!/bin/bash
INPUT="urls.txt"
OUTPUT="screenshots_$(date +%Y%m%d)"
mkdir -p "$OUTPUT"

eyewitness -f "$INPUT" -d "$OUTPUT" --headless --threads 5 --no-prompt

echo "Report: $OUTPUT/Eyewitness.html"
xdg-open "$OUTPUT/Eyewitness.html"
```

---

## Troubleshooting

```bash
# Check dependencies
pip3 install -r requirements.txt

# Install Firefox/Geckodriver
sudo apt install firefox geckodriver

# Fix blank screenshots (increase timeout)
eyewitness -f urls.txt -d screenshots --timeout 15

# Debug with visible browser
eyewitness -f urls.txt -d screenshots --show-selenium

# Resume interrupted scan
eyewitness -f urls.txt -d screenshots --resume ew.db
```

---

## Alternatives

| Tool | Language | Browser | Speed |
|------|----------|---------|-------|
| EyeWitness | Python | Firefox | Medium |
| GoWitness | Go | Chrome | Fast |
| Aquatone | Go | Chrome | Fast |
| CutyCapt | C++ | QtWebKit | Medium |

---

## Integration Examples

```bash
# Full recon workflow
nmap -sV -sC -p- -oX scan.xml target.com
eyewitness -x scan.xml -d recon --headless

# Bug bounty workflow
subfinder -d target.com -o subs.txt
cat subs.txt | httpx -o live.txt
eyewitness -f live.txt -d bugbounty --headless --threads 5

# Combine with other tools
eyewitness -x scan.xml -d screenshots --headless
whatweb -i urls.txt > technologies.txt
nikto -h urls.txt > nikto_results.txt
```
