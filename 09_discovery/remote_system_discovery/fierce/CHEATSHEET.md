---
title: "fierce Cheatsheet"
description: "Quick reference for fierce DNS enumeration and subdomain discovery"
mitre_tactic: discovery
mitre_tactic_id: TA0007
mitre_subcategory: remote_system_discovery
tags:
  - dns
  - subdomain
  - cheatsheet
  - enumeration
---

# fierce Cheatsheet

## Quick Start

```bash
# Basic domain enumeration
fierce --domain example.com

# With custom wordlist
fierce --domain example.com --subdomain-file wordlist.txt

# Wide scan with traversal
fierce --domain example.com --wide --traverse 25
```

## Common Flags

| Flag | Description | Example |
|------|-------------|---------|
| `--domain DOMAIN` | Target domain | `fierce --domain example.com` |
| `--subdomain-file FILE` | Wordlist | `fierce --subdomain-file wordlist.txt --domain example.com` |
| `--delay SECS` | Delay between requests | `fierce --delay 0.5 --domain example.com` |
| `--threads NUM` | Concurrent threads | `fierce --threads 20 --domain example.com` |
| `--traverse NUM` | IPs to traverse | `fierce --traverse 25 --domain example.com` |
| `--lookup` | DNS lookups | `fierce --lookup --domain example.com` |
| `--connect` | TCP connections | `fierce --connect --domain example.com` |
| `--wide` | Wider IP ranges | `fierce --wide --domain example.com` |
| `--dns-servers SERVER` | Custom DNS | `fierce --dns-servers 8.8.8.8 --domain example.com` |
| `--xml FILE` | XML output | `fierce --xml out.xml --domain example.com` |
| `--quiet` | Minimal output | `fierce --quiet --domain example.com` |

## Host Discovery

```bash
# Basic enumeration
fierce --domain example.com

# Full enumeration
fierce --domain example.com --wide --traverse 50 --lookup --connect

# Extract IPs only
fierce --domain example.com 2>&1 | grep -oP '\d+\.\d+\.\d+\.\d+' | sort -u

# Extract subdomains only
fierce --domain example.com 2>&1 | grep -oP '\S+\.example\.com' | sort -u
```

## Examples

**Basic enumeration:**
```bash
fierce --domain example.com
```

**Custom wordlist:**
```bash
fierce --domain example.com --subdomain-file /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt
```

**Rate-limited:**
```bash
fierce --domain example.com --threads 5 --delay 1
```

**With XML output:**
```bash
fierce --domain example.com --xml results.xml --wide --traverse 25
```

## Chaining

```bash
# fierce + nmap
fierce --domain example.com 2>&1 | grep -oP '\d+\.\d+\.\d+\.\d+' | sort -u | \
    xargs -I {} nmap -sV -T4 {}

# fierce + httpx
fierce --domain example.com 2>&1 | grep -oP '\S+\.example\.com' | \
    httpx -silent

# fierce + dig for zone transfer
fierce --domain example.com
dig @ns1.example.com example.com AXFR
```

## Troubleshooting

| Problem | Fix |
|---------|-----|
| No DNS servers found | Use `--dns-servers 8.8.8.8` |
| Zone transfer fails | Normal — most servers block it |
| Connection refused | Try different DNS server |
| No subdomains found | Try larger wordlist, verify domain exists |
| Slow enumeration | Increase `--threads`, reduce `--delay` |
