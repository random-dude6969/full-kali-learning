---
title: "fierce — Domain Enumeration and DNS Reconnaissance"
description: "Enumerate subdomains, discover DNS zones, and map IP ranges using DNS brute-forcing and zone transfer attempts."
mitre_tactic: discovery
mitre_tactic_id: TA0007
mitre_subcategory: remote_system_discovery
categories:
  - dns-reconnaissance
  - subdomain-enumeration
  - network-discovery
tags:
  - dns
  - subdomain
  - enumeration
  - brute-force
  - zone-transfer
install_command: "sudo apt install fierce"
version: "2.0.0"
github_repo: "https://github.com/mschwager/fierce"
kali_tools_page: "https://www.kali.org/tools/fierce/"
difficulty_level: intermediate
requires_root: false
gui_available: false
related_tools:
  - dnsenum
  - dnsrecon
  - amass
  - subfinder
---

# fierce — Domain Enumeration and DNS Reconnaissance

## 1. Introduction

**fierce** is a DNS reconnaissance and subdomain enumeration tool designed to quickly map the attack surface of a target domain. It performs DNS zone transfer attempts, brute-force subdomain discovery, and IP range calculation.

**Use cases:**
- Discover hidden subdomains not listed in public records
- Attempt DNS zone transfers to extract full zone data
- Map IP ranges associated with a target domain
- Identify additional attack surfaces (staging servers, dev environments)
- Enumerate mail servers, web servers, and other services

**Key advantage:** fierce combines multiple DNS enumeration techniques into a single tool, providing rapid subdomain discovery without requiring API keys or authentication.

---

## 2. Installation

**Install via apt:**
```bash
sudo apt install fierce
```

**Install via pip:**
```bash
pip install fierce
```

**Verify installation:**
```bash
fierce --version
```

**Update wordlist:**
```bash
# fierce uses a built-in wordlist, but you can supply custom ones
fierce --subdomain-file /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt --domain example.com
```

---

## 3. Core Concepts

### 3.1 DNS Zone Transfer

A DNS zone transfer (AXFR) replicates DNS records from a primary server to a secondary server. If misconfigured, any host can request a zone transfer, revealing all DNS records for the domain.

**Zone transfer request:**
```bash
dig @ns1.example.com example.com AXFR
```

### 3.2 Subdomain Brute-Forcing

fierce attempts to resolve common subdomain names (www, mail, ftp, vpn, etc.) against the target domain. Each successful resolution reveals a new host.

### 3.3 IP Range Discovery

After finding subdomains, fierce performs reverse DNS lookups to identify the IP ranges associated with the target organization. This reveals the full network footprint.

### 3.4 DNS Record Types

| Record | Purpose |
|--------|---------|
| A | Maps hostname to IPv4 address |
| AAAA | Maps hostname to IPv6 address |
| MX | Mail server |
| NS | Name server |
| CNAME | Canonical name (alias) |
| TXT | Text records (SPF, DKIM, etc.) |

---

## 4. Command Reference

### Basic Usage

```bash
# Basic domain enumeration
fierce --domain example.com

# Brute-force with built-in wordlist
fierce --domain example.com --subdomain-file wordlist.txt
```

### Flags

| Flag | Description |
|------|-------------|
| `--domain DOMAIN` | Target domain to enumerate |
| `--subdomain-file FILE` | Custom subdomain wordlist |
| `--delay SECONDS` | Delay between requests |
| `--threads NUM` | Number of concurrent threads |
| `--traverse NUM` | Number of IPs to traverse near found subdomains |
| `--lookup` | Perform DNS lookups on found hosts |
| `--connect` | Attempt TCP connection to found hosts |
| `--wide` | Generate wider IP ranges from found subdomains |
| `--wordlist FILE` | Alias for --subdomain-file |
| `--dns-servers SERVER` | Comma-separated DNS servers to use |
| `--quiet` | Suppress output |
| `--xml FILE` | Output results to XML file |
| `--trace` | Show DNS trace output |
| `--help` | Show help message |

---

## 5. Examples

### Basic

**Simple domain enumeration:**
```bash
fierce --domain example.com
```
Output:
```
DNS Servers for example.com:
  ns1.example.com
  ns2.example.com

Trying zone transfer on ns1.example.com... failed
Trying zone transfer on ns2.example.com... failed

Brute-forcing subdomains... found 12

www.example.com       -> 93.184.216.34
mail.example.com      -> 93.184.216.35
vpn.example.com       -> 93.184.216.36
```

### Intermediate

**Use custom wordlist:**
```bash
fierce --domain example.com --subdomain-file /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt
```

**Limit threads and add delay:**
```bash
fierce --domain example.com --threads 5 --delay 0.5
```

**Attempt TCP connections:**
```bash
fierce --domain example.com --connect --traverse 10
```

### Advanced

**Full enumeration with wide range:**
```bash
fierce --domain example.com --wide --traverse 20 --lookup
```

**Use specific DNS server:**
```bash
fierce --domain example.com --dns-servers 8.8.8.8,8.8.4.4
```

**Output to XML:**
```bash
fierce --domain example.com --xml fierce_results.xml
```

### Real-World

**Bug bounty reconnaissance:**
```bash
#!/bin/bash
DOMAIN="$1"
fierce --domain "$DOMAIN" \
    --subdomain-file /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-10000.txt \
    --traverse 25 \
    --wide \
    --lookup \
    --connect \
    --xml "fierce_${DOMAIN}.xml" \
    --threads 10
```

**Network mapping for pentest:**
```bash
fierce --domain target.com --traverse 50 --wide --lookup 2>&1 | tee fierce_$(date +%F).txt
```

---

## 6. Advanced

### 6.1 Zone Transfer Exploitation

If zone transfer succeeds, you get the complete DNS zone:
```bash
# Manual zone transfer with dig
dig @ns1.example.com example.com AXFR
```

**What zone transfer reveals:**
- All subdomains and their IP addresses
- Mail server configurations
- Text records (SPF, DKIM, DMARC)
- TTL values and other metadata

### 6.2 Subdomain Wordlist Optimization

**High-quality wordlists:**
```bash
# SecLists subdomain lists
fierce --domain example.com --subdomain-file \
    /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt

# Custom wordlist based on target industry
cat > custom_wordlist.txt << 'EOF'
api
staging
dev
test
admin
portal
internal
corporate
vpn
mail
webmail
smtp
pop
imap
EOF

fierce --domain example.com --subdomain-file custom_wordlist.txt
```

### 6.3 Output Parsing

**Parse fierce output for IPs:**
```bash
fierce --domain example.com 2>&1 | grep -oP '\d+\.\d+\.\d+\.\d+' | sort -u
```

**Extract subdomains:**
```bash
fierce --domain example.com 2>&1 | grep -oP '\S+\.example\.com' | sort -u
```

### 6.4 Integration with Other Tools

**Feed into nmap:**
```bash
fierce --domain example.com 2>&1 | grep -oP '\d+\.\d+\.\d+\.\d+' | sort -u | \
    xargs -I {} nmap -sV -T4 {}
```

**Feed into httpx for web scanning:**
```bash
fierce --domain example.com 2>&1 | grep -oP '\S+\.example\.com' | \
    httpx -silent
```

---

## 7. Detection and Defense

### 7.1 Detection

**Monitor DNS queries:**
```bash
sudo tcpdump -i eth0 port 53 -n -c 100
```

**Look for zone transfer attempts:**
```bash
sudo tcpdump -i eth0 port 53 -n | grep "AXFR"
```

**DNS query volume spikes:**
```bash
# Count DNS queries per second
sudo tcpdump -i eth0 port 53 -n -l | awk '{print $NF}' | sort | uniq -c
```

### 7.2 Defense

**Disable zone transfers:**
```bash
# BIND named.conf
zone "example.com" {
    allow-transfer { none; };
};
```

**DNS server hardening:**
- Restrict zone transfers to authorized secondary servers
- Implement DNS Response Rate Limiting (RRL)
- Use DNSSEC for record integrity
- Monitor DNS query logs for anomalies

**Rate limiting:**
Configure DNS servers to rate-limit queries from single sources.

---

## 8. Troubleshooting

### 8.1 Common Issues

**"No DNS servers found":**
```bash
dig NS example.com
fierce --domain example.com --dns-servers 8.8.8.8
```

**Zone transfer always fails:**
This is normal — most production DNS servers block zone transfers. fierce will still brute-force subdomains.

**"Connection refused":**
- DNS server may be blocking queries
- Try different DNS server: `--dns-servers 8.8.8.8`

**No subdomains found:**
- Try a larger wordlist
- Check if domain exists: `dig example.com`

### 8.2 Performance Issues

**Slow enumeration:**
```bash
fierce --domain example.com --threads 20 --delay 0.1
```
Increase threads and reduce delay for faster results.

**Too many results:**
```bash
fierce --domain example.com --quiet
```
Use quiet mode to reduce output noise.

### 8.3 Network Issues

**Firewall blocking DNS:**
```bash
# Test DNS connectivity
dig @8.8.8.8 example.com
nslookup example.com 8.8.8.8
```

**DNS over HTTPS (DoH):**
Some networks redirect DNS traffic. fierce uses standard DNS, so it may not work on DoH-only networks.

**DNS over TLS (DoT):**
Port 853 may be blocked. fierce uses standard DNS on port 53.

### 8.4 Rate Limiting Issues

**DNS server rate limiting:**
```bash
# Slow down requests
fierce --domain example.com --delay 2 --threads 3

# Use multiple DNS servers
fierce --domain example.com --dns-servers 8.8.8.8,1.1.1.1,9.9.9.9
```

**IP blocking after excessive queries:**
If your IP gets blocked, wait or use a different DNS server.

### 8.5 DNS Enumeration Alternatives

| Tool | Speed | Wordlist | Zone Transfer | Passive |
|------|-------|----------|---------------|---------|
| fierce | Fast | Yes | Yes | No |
| dnsenum | Medium | Yes | Yes | No |
| dnsrecon | Fast | Yes | Yes | Yes |
| amass | Slow | Yes | Limited | Yes |
| subfinder | Fast | Limited | No | Yes |
| amass (passive) | Fast | No | No | Yes |

Use fierce for quick active enumeration. Combine with passive tools (amass, subfinder) for comprehensive coverage.

### 8.6 DNS Record Deep Dive

**Important DNS record types for enumeration:**

| Record | Purpose | Example |
|--------|---------|---------|
| A | IPv4 address | example.com → 93.184.216.34 |
| AAAA | IPv6 address | example.com → 2606:2800:220:1:... |
| MX | Mail server | mail.example.com → 93.184.216.35 |
| NS | Name server | ns1.example.com → 93.184.216.36 |
| TXT | Text records | SPF, DKIM, DMARC, verification |
| SOA | Start of Authority | Zone metadata |
| CNAME | Alias | www → example.com |
| SRV | Service locator | _sip._tcp.example.com |

fierce discovers A records by default. Use dig for deeper record inspection:
```bash
dig example.com ANY
dig example.com TXT
dig example.com MX
```

---

## Resources

- [fierce GitHub](https://github.com/mschwager/fierce)
- [fierce documentation](https://github.com/mschwager/fierce/wiki)
- [Kali fierce tool page](https://www.kali.org/tools/fierce/)
- [DNS zone transfer (RFC 5936)](https://tools.ietf.org/html/rfc5936)
- [Subdomain enumeration techniques](https://github.com/appsecco/awesome-web-hacking)
