---
title: "mxcheck Quick Reference"
description: "Quick reference guide for mxcheck email server testing"
categories:
  - "email"
  - "cheatsheet"
tags:
  - "quick-reference"
  - "email"
  - "mx"
  - "smtp"
---

# mxcheck CHEATSHEET

## Quick Start

**Basic MX check**: `mxcheck example.com`

**SMTP test**: `mxcheck -s example.com`

**Verbose output**: `mxcheck -v example.com`

**Custom port**: `mxcheck -p 587 example.com`

**Save results**: `mxcheck -o results.txt example.com`

## Common Flags

| Flag | Description | Example |
|------|-------------|---------|
| `-p PORT` | SMTP port | `mxcheck -p 587 example.com` |
| `-s` | SMTP test | `mxcheck -s example.com` |
| `-v` | Verbose | `mxcheck -v example.com` |
| `-t TIMEOUT` | Timeout (sec) | `mxcheck -t 10 example.com` |
| `-d` | Debug mode | `mxcheck -d example.com` |
| `-f EMAIL` | FROM address | `mxcheck -f sender@example.com example.com` |
| `-r EMAIL` | RCPT TO address | `mxcheck -r recipient@example.com example.com` |
| `-h HOSTNAME` | HELO hostname | `mxcheck -h mail.example.com example.com` |
| `-o FILE` | Output file | `mxcheck -o results.txt example.com` |
| `-q` | Quiet mode | `mxcheck -q example.com` |
| `--no-colour` | No colors | `mxcheck --no-colour example.com` |

## Examples

### Basic MX Record Check
```bash
mxcheck example.com
```

### SMTP Test
```bash
mxcheck -s example.com
```

### Verbose Output
```bash
mxcheck -v example.com
```

### Custom Port
```bash
mxcheck -p 587 example.com
```

### SMTP with Email Addresses
```bash
mxcheck -s -f sender@example.com -r recipient@example.com example.com
```

### Debug Mode
```bash
mxcheck -d example.com
```

### Save Results
```bash
mxcheck -o results.txt example.com
```

### Extended Timeout
```bash
mxcheck -t 10 example.com
```

### Multiple Domains
```bash
for domain in example.com test.com demo.com; do
  echo "=== $domain ==="
  mxcheck -v "$domain"
done
```

### Custom Hostname
```bash
mxcheck -h mail.example.com example.com
```

## Chaining

### Chain with dig
```bash
# Query DNS records, then test SMTP
echo "=== MX Records ==="
dig MX example.com +short

echo "=== SPF Record ==="
dig TXT example.com +short | grep -i spf

echo "=== DKIM Record ==="
dig TXT default._domainkey.example.com +short

echo "=== DMARC Record ==="
dig TXT _dmarc.example.com +short

echo "=== SMTP Test ==="
mxcheck -s example.com
```

### Chain with swaks
```bash
# Find MX records, then test with swaks
MX=$(dig MX example.com +short | awk '{print $2}')
swaks --to test@example.com --server "$MX"
```

### Chain with smtp-user-enum
```bash
# Find mail servers, then enumerate users
MX=$(dig MX example.com +short | awk '{print $2}')
smtp-user-enum -M VRFY -U users.txt -t "$MX"
```

### Batch Processing
```bash
while read domain; do
  echo "=== $domain ==="
  mxcheck -s "$domain" > "$domain.txt"
done < domains.txt
```

### Email Security Audit
```bash
#!/bin/bash
DOMAIN="example.com"
echo "=== MX ==="
dig MX "$DOMAIN" +short
echo "=== SPF ==="
dig TXT "$DOMAIN" +short | grep -i spf
echo "=== DKIM ==="
dig TXT "default._domainkey.$DOMAIN" +short
echo "=== DMARC ==="
dig TXT "_dmarc.$DOMAIN" +short
mxcheck -s "$DOMAIN"
```

## Troubleshooting

### No MX records
- Verify domain: `dig MX example.com +short`
- Check domain exists

### SMTP connection failed
- Verify port: `nmap -p 25,587 example.com`
- Check firewall
- Verify mail server running

### Authentication errors
- Check credentials
- Verify TLS/SSL requirements
- Check permissions

### Timeout issues
- Increase timeout: `mxcheck -t 10 example.com`
- Check network connectivity
- Server may be overloaded

### SPF/DKIM/DMARC issues
- Verify DNS records: `dig TXT example.com +short`
- Check record syntax
- Wait for DNS propagation

### SMTP response errors
- Check server configuration
- Verify relay settings
- Check mail queue
