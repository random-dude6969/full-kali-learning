# Findomain Cheatsheet

## Quick Reference

| Category | Command | Description |
|----------|---------|-------------|
| **Basic Scan** | `findomain -t target.com` | Basic subdomain discovery |
| **Brute-Force** | `findomain -t target.com -b` | DNS brute-forcing |
| **Custom Wordlist** | `findomain -t target.com -b -w wordlist.txt` | Custom wordlist |
| **Check Live** | `findomain -t target.com --check` | Check live subdomains |
| **Multiple Targets** | `findomain -f targets.txt` | Scan from file |
| **JSON Output** | `findomain -t target.com --json` | JSON output |
| **Resolve IPs** | `findomain -t target.com --resolve` | Resolve to IPs |

---

## Basic Commands

### Subdomain Discovery
```bash
findomain -t target.example.com
```

### DNS Brute-Forcing
```bash
findomain -t target.example.com -b
```

### Custom Wordlist
```bash
findomain -t target.example.com -b -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt
```

### Check Live Subdomains
```bash
findomain -t target.example.com --check
```

### Multiple Targets
```bash
findomain -f targets.txt
```

### Resolve to IPs
```bash
findomain -t target.example.com --resolve
```

---

## Essential Flags

### Target Options
| Flag | Description |
|------|-------------|
| `-t` | Target domain |
| `-f` | Targets file |
| `--subdomains-only` | Only show subdomains |
| `--ips-only` | Only show IPs |
| `--resolved` | Show resolved IPs |

### Source Options
| Flag | Description |
|------|-------------|
| `--ct` | Use Certificate Transparency |
| `--dns` | Use DNS databases |
| `-b` | Enable DNS brute-forcing |
| `-w` | Custom wordlist for brute-forcing |
| `--shodan-key` | Shodan API key |
| `--censys-ids` | Censys API credentials |
| `--virustotal-key` | VirusTotal API key |
| `--securitytrails-key` | SecurityTrails API key |
| `--facebook` | Use Facebook CT |
| `--hackertarget` | Use HackerTarget |
| `--rapiddns` | Use RapidDNS |
| `--sitedossier` | Use SiteDossier |
| `--no-ct` | Disable CT logs |
| `--no-dns` | Disable DNS databases |
| `--ct-only` | Only use CT logs |
| `--dns-only` | Only use DNS databases |

### DNS Options
| Flag | Description |
|------|-------------|
| `-r` | Custom DNS resolvers |
| `--dns-resolver` | Single DNS resolver |
| `--dns-timeout` | DNS query timeout |
| `--dns-retries` | DNS query retries |

### Output Options
| Flag | Description |
|------|-------------|
| `-o` | Output file |
| `--json` | JSON output |
| `--csv` | CSV output |
| `--a` | Show all results |
| `--q` | Quiet mode |
| `-v` | Verbose output |

### Performance Options
| Flag | Description |
|------|-------------|
| `--threads` | Number of threads |
| `--timeout` | Connection timeout |
| `--rate-limit` | Rate limit (req/sec) |
| `--connections-limit` | Max connections |

### Check Options
| Flag | Description |
|------|-------------|
| `--check` | Check if subdomains are live |
| `--subdomains-output` | Output live subdomains |
| `--ips-output` | Output resolved IPs |

---

## Common Combinations

```bash
# Full scan with brute-force
findomain -t target.com -b -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-110000.txt

# Check live and save
findomain -t target.com --check --subdomains-output live_subs.txt

# JSON output with threads
findomain -t target.com --json --threads 100

# Custom DNS resolvers
findomain -t target.com -r 8.8.8.8,8.8.4.4,1.1.1.1

# Rate limited scan
findomain -t target.com --rate-limit 10 --threads 20

# Only Certificate Transparency
findomain -t target.com --ct-only

# Only DNS databases
findomain -t target.com --dns-only

# Multiple targets with live check
findomain -f targets.txt --check --subdomains-output live_subs.txt
```

---

## Wordlist Locations

| Wordlist | Path |
|----------|------|
| DNS 5k | `/usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt` |
| DNS 20k | `/usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-20000.txt` |
| DNS 110k | `/usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-110000.txt` |

---

## Quick One-Liners

```bash
# Quick subdomain scan
findomain -t target.com -o subs.txt

# Deep brute-force
findomain -t target.com -b -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-110000.txt -o deep_subs.txt

# Find live subdomains
findomain -t target.com --check --subdomains-output live.txt

# Get IPs for subdomains
findomain -t target.com --resolve --json -o subs.json

# Scan multiple domains
findomain -f domains.txt --check --subdomains-output live_subs.txt

# Fast scan with fewer threads
findomain -t target.com --threads 10 --timeout 30

# Custom DNS resolver
findomain -t target.com -r 8.8.8.8 --show-ips
```

---

## Batch Processing

```bash
# Scan multiple domains
for domain in target1.com target2.com target3.com; do
    findomain -t "$domain" -b -o "${domain}_subs.txt"
done

# Scan from file
findomain -f domains.txt -o all_subs.txt

# Parallel scanning
cat domains.txt | xargs -P 5 -I {} findomain -t {} -b -o {}_subs.txt

# Combine with live check
findomain -f domains.txt --check --subdomains-output live_subs.txt
```

---

## Integration with Other Tools

```bash
# Findomain + Subfinder
findomain -t target.com -o findomain_subs.txt
subfinder -d target.com -o subfinder_subs.txt
cat findomain_subs.txt subfinder_subs.txt | sort -u > all_subs.txt

# Findomain + Amass
findomain -t target.com -o findomain_subs.txt
amass enum -passive -d target.com -o amass_subs.txt
cat findomain_subs.txt amass_subs.txt | sort -u > all_subs.txt

# Findomain + httpx
findomain -t target.com --check --subdomains-output subs.txt
httpx -l subs.txt -o live_hosts.txt -silent

# Findomain + Nmap
findomain -t target.com --check --subdomains-output subs.txt
nmap -iL subs.txt -p 80,443,8080,8443 --open -oN ports.txt

# Findomain + Nuclei
findomain -t target.com --check --subdomains-output subs.txt
nuclei -l subs.txt -t cves/
```

---

## API Key Configuration

```bash
# Set environment variables
export SHODAN_API_KEY="your_key"
export CENSYS_API_ID="your_id"
export CENSYS_API_SECRET="your_secret"
export VIRUSTOTAL_API_KEY="your_key"
export SECURITYTRAILS_API_KEY="your_key"

# Or use command-line flags
findomain -t target.com \
    --shodan-key YOUR_KEY \
    --censys-ids YOUR_ID:YOUR_SECRET \
    --virustotal-key YOUR_KEY
```

---

## Error Handling

```bash
# Increase timeout
findomain -t target.com --timeout 60

# Use custom DNS resolver
findomain -t target.com -r 8.8.8.8,8.8.4.4

# Reduce threads
findomain -t target.com --threads 10

# Reduce connections
findomain -t target.com --connections-limit 50

# Skip specific sources
findomain -t target.com --no-ct --no-dns
```
