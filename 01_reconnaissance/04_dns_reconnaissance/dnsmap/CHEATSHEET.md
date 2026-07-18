# DNSmap — Cheatsheet

## Quick Start

```bash
# Basic subdomain scan
dnsmap example.com

# Use custom wordlist
dnsmap example.com -w wordlist.txt

# Limit threads and save results
dnsmap example.com -t 10 -r results.txt
```

## Essential Commands

| Command | Description |
|---------|-------------|
| `dnsmap target.com` | Basic scan with built-in wordlist |
| `dnsmap target.com -w wordlist.txt` | Custom wordlist scan |
| `dnsmap target.com -t 10` | Limit to 10 threads |
| `dnsmap target.com -r results.txt` | Save results to file |
| `dnsmap target.com -c results.csv` | Save as CSV |
| `dnsmap target.com -i 10.0.0.0/8` | Filter by IP range |

## Flag Reference

| Flag | Description |
|------|-------------|
| (positional) | Target domain |
| `-w` | Custom wordlist file |
| `-t` | Number of threads |
| `-l` | Maximum wordlist length |
| `-r` | Results file |
| `-c` | CSV output |
| `-i` | IP range filter |
| `-h` | Help |

## Wordlist Locations

```bash
# Kali default wordlists
/usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt
/usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-20000.txt
/usr/share/wordlists/amass/subdomains-top1mil-5000.txt

# Custom wordlist location
~/custom_wordlists/subdomains.txt
```

## Automation

```bash
# Multi-domain scan
for domain in $(cat domains.txt); do
    dnsmap "$domain" -r "${domain}_results.txt" 2>/dev/null
    sleep 5
done

# Scan and extract subdomains
dnsmap example.com -r results.txt 2>/dev/null && \
    grep -oE "[a-zA-Z0-9.-]+\.example\.com" results.txt | sort -u

# Scan and resolve IPs
dnsmap example.com 2>/dev/null | \
    grep "found:" | awk '{print $3}' | \
    while read sub; do
        echo "$sub,$(dig +short $sub | head -1)"
    done > resolved.csv
```

## Workflow Integration

```bash
# DNSmap → Nmap pipeline
dnsmap example.com 2>/dev/null | \
    grep "found:" | awk '{print $3}' | \
    sort -u | while read sub; do
        dig +short "$sub" | head -1
    done | sort -u > ips.txt && \
    nmap -sV -iL ips.txt

# DNSmap + Subfinder combination
dnsmap example.com -r dnsmap.txt 2>/dev/null
subfinder -d example.com -silent > subfinder.txt
cat dnsmap.txt subfinder.txt | grep -oE "[a-zA-Z0-9.-]+\.example\.com" | sort -u > all_subs.txt
```

## Output Formats

```bash
# Text output (default)
dnsmap example.com -r results.txt

# CSV output
dnsmap example.com -c results.csv

# Combine both
dnsmap example.com -r results.txt -c results.csv
```
