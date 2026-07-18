# FinalRecon Cheatsheet

## Quick Reference

| Category | Command | Description |
|----------|---------|-------------|
| **Full Scan** | `python3 finalrecon.py -d target.com --full` | Complete reconnaissance |
| **Subdomain** | `python3 finalrecon.py -d target.com --subdomain` | Subdomain discovery only |
| **Ports** | `python3 finalrecon.py -d target.com --ports` | Port scanning only |
| **Tech** | `python3 finalrecon.py -d target.com --tech` | Technology detection |
| **SSL** | `python3 finalrecon.py -d target.com --ssl` | SSL certificate analysis |
| **WHOIS** | `python3 finalrecon.py -d target.com --whois` | WHOIS lookup |
| **Dirs** | `python3 finalrecon.py -d target.com --dirs` | Directory discovery |
| **Emails** | `python3 finalrecon.py -d target.com --emails` | Email harvesting |

---

## Module Commands

### Full Reconnaissance
```bash
python3 finalrecon.py -d target.example.com --full
```

### Subdomain Discovery
```bash
python3 finalrecon.py -d target.example.com --subdomain
```

### Port Scanning
```bash
python3 finalrecon.py -d target.example.com --ports
```

### Technology Detection
```bash
python3 finalrecon.py -d target.example.com --tech
```

### SSL Certificate Analysis
```bash
python3 finalrecon.py -d target.example.com --ssl
```

### WHOIS Lookup
```bash
python3 finalrecon.py -d target.example.com --whois
```

### Directory Discovery
```bash
python3 finalrecon.py -d target.example.com --dirs
```

### Email Harvesting
```bash
python3 finalrecon.py -d target.example.com --emails
```

### DNS Enumeration
```bash
python3 finalrecon.py -d target.example.com --dns
```

---

## Essential Flags

### Target Options
| Flag | Description |
|------|-------------|
| `-d` | Target domain |
| `--full` | Full reconnaissance |
| `--subdomain` | Subdomain discovery only |
| `--ports` | Port scanning only |
| `--tech` | Technology detection only |
| `--ssl` | SSL certificate analysis |
| `--whois` | WHOIS lookup only |
| `--dirs` | Directory discovery only |
| `--emails` | Email harvesting only |
| `--dns` | DNS enumeration only |

### Output Options
| Flag | Description |
|------|-------------|
| `-o` | Output file |
| `--json` | JSON output |
| `-v` | Verbose mode |
| `--quiet` | Quiet mode |
| `--no-color` | Disable colors |

### Performance Options
| Flag | Description |
|------|-------------|
| `--threads` | Number of threads |
| `--timeout` | Connection timeout |
| `--delay` | Delay between requests |

### Network Options
| Flag | Description |
|------|-------------|
| `--proxy` | HTTP proxy |
| `--useragent` | Custom User-Agent |
| `--random-agent` | Random User-Agent |

### Module Options
| Flag | Description |
|------|-------------|
| `--wordlist` | Custom wordlist |
| `--recursive` | Recursive scanning |
| `--extensions` | File extensions |

---

## Common Combinations

```bash
# Full scan with JSON output
python3 finalrecon.py -d target.com --full --json -o results.json

# Subdomain + Port scan
python3 finalrecon.py -d target.com --subdomain --ports

# Technology + Directory discovery
python3 finalrecon.py -d target.com --tech --dirs

# SSL + WHOIS
python3 finalrecon.py -d target.com --ssl --whois

# Custom wordlist for directories
python3 finalrecon.py -d target.com --dirs --wordlist /path/to/wordlist.txt

# With proxy
python3 finalrecon.py -d target.com --full --proxy http://proxy:8080

# With custom threads
python3 finalrecon.py -d target.com --full --threads 50
```

---

## Output Formats

```bash
# Text output
python3 finalrecon.py -d target.com --full -o results.txt

# JSON output
python3 finalrecon.py -d target.com --full --json -o results.json

# Verbose output
python3 finalrecon.py -d target.com --full -v

# Quiet output
python3 finalrecon.py -d target.com --full --quiet
```

---

## Quick One-Liners

```bash
# Quick recon
python3 finalrecon.py -d target.com --full -v

# Find subdomains
python3 finalrecon.py -d target.com --subdomain -o subdomains.txt

# Check SSL certificates
python3 finalrecon.py -d target.com --ssl -o ssl_info.txt

# Get WHOIS info
python3 finalrecon.py -d target.com --whois -o whois.txt

# Find hidden directories
python3 finalrecon.py -d target.com --dirs --recursive

# Discover technologies
python3 finalrecon.py -d target.com --tech -o technologies.txt

# Harvest emails
python3 finalrecon.py -d target.com --emails -o emails.txt
```

---

## Batch Processing

```bash
# Scan multiple domains
for domain in target1.com target2.com target3.com; do
    python3 finalrecon.py -d "$domain" --full --json -o "${domain}_results.json"
done

# Scan from file
while read domain; do
    python3 finalrecon.py -d "$domain" --full -o "${domain}_results.txt"
done < domains.txt

# Parallel scanning
cat domains.txt | xargs -P 5 -I {} python3 finalrecon.py -d {} --full -o {}_results.txt
```

---

## Integration with Other Tools

```bash
# FinalRecon + Subfinder
python3 finalrecon.py -d target.com --subdomain -o finalrecon_subs.txt
subfinder -d target.com -o subfinder_subs.txt
cat finalrecon_subs.txt subfinder_subs.txt | sort -u > all_subs.txt

# FinalRecon + Nmap
python3 finalrecon.py -d target.com --ports -o ports.txt
nmap -sV -p $(grep -oP '\d+/tcp' ports.txt | cut -d'/' -f1 | tr '\n' ',') target.com

# FinalRecon + FFuF
python3 finalrecon.py -d target.com --subdomain -o subs.txt
while read sub; do
    ffuf -u "http://$sub/FUZZ" -w wordlist.txt -fc 404 -o "ffuf_$sub.json" -of json
done < subs.txt

# FinalRecon + Nuclei
python3 finalrecon.py -d target.com --full -o recon.txt
nuclei -u http://target.com -t cves/
```

---

## API Key Configuration

```bash
# Create config file
mkdir -p ~/.config/finalrecon
cat > ~/.config/finalrecon/api_keys.json << 'EOF'
{
    "shodan": "YOUR_SHODAN_API_KEY",
    "censys": "YOUR_CENSYS_API_ID:YOUR_CENSYS_API_SECRET",
    "virustotal": "YOUR_VIRUSTOTAL_API_KEY",
    "securitytrails": "YOUR_SECURITYTRAILS_API_KEY"
}
EOF
```

---

## Error Handling

```bash
# Increase timeout
python3 finalrecon.py -d target.com --full --timeout 60

# Reduce threads
python3 finalrecon.py -d target.com --full --threads 5

# Skip specific modules
python3 finalrecon.py -d target.com --subdomain --ports --tech

# Use proxy
python3 finalrecon.py -d target.com --full --proxy http://proxy:8080
```
