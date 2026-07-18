# wafw00f — Cheatsheet

## Quick Reference

**Purpose**: Identify which WAF (Web Application Firewall) protects a web application.

**Requires**: Python 3.10+, no root needed

**Install**: `pip3 install wafw00f` or `sudo apt install wafw00f`

---

## Core Syntax

```bash
wafw00f [options] <target_url>
```

---

## Essential Commands

### Basic Detection

```bash
# Detect WAF on target
wafw00f https://example.org

# Verbose output
wafw00f -v https://example.org

# Run all plugins (thorough)
wafw00f -a https://example.org
```

### Output Formats

```bash
# JSON output
wafw00f --json https://example.org

# XML output
wafw00f --xml https://example.org

# Save to file
wafw00f -o results.json --json https://example.org
```

### Subdomain Scanning

```bash
# Find and scan subdomains
wafw00f -f -r https://example.org

# Find subdomains only
wafw00f -f https://example.org
```

---

## Flag Reference

| Flag | Long Form | Description |
|------|-----------|-------------|
| `-l` | `--list` | List all WAF plugins |
| `-a` | `--all` | Run all plugins |
| `-f` | `--find-subdomains` | Discover subdomains |
| `-r` | `--recursion` | Recurse into subdomains |
| `-p <plugin>` | `--plugin` | Run specific plugin |
| `-o <file>` | `--output` | Save results to file |
| `-v` | `--verbose` | Verbose output |
| `-d` | `--debug` | Debug output |
| `--proxy <url>` | | Use HTTP/SOCKS5 proxy |
| `--delay <sec>` | | Delay between requests |
| `-t <num>` | `--threads` | Concurrent threads |
| `--json` | | JSON output |
| `--xml` | | XML output |
| `--user-agent <ua>` | | Custom User-Agent |
| `-c <cookies>` | `--cookie` | Send cookies |
| `-H <header>` | `--header` | Custom header |
| `--no-check-ssl` | | Skip SSL verification |
| `--follow-redirects` | | Follow redirects |

---

## Common Scenarios

### Penetration Test Recon

```bash
# Identify WAF before attack
wafw00f https://target.com

# Run all plugins if first attempt fails
wafw00f -a https://target.com

# Verbose for detailed info
wafw00f -v -a https://target.com
```

### Bug Bounty Enumeration

```bash
# Enumerate WAFs across subdomains
wafw00f -f -r --json -o waf_enum.json https://program.example.com
```

### Authenticated Target Scanning

```bash
wafw00f \
  -H "Authorization: Bearer eyJhbGc..." \
  -c "session=abc123; token=xyz789" \
  https://app.example.com
```

### Proxy-Based Scanning (Burp Suite)

```bash
wafw00f --proxy http://127.0.0.1:8080 https://target.com
```

### Stealth Scanning

```bash
# Custom User-Agent + delay
wafw00f \
  --user-agent "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36" \
  --delay 5 \
  -t 1 \
  https://target.com
```

---

## WAF Detection Output

| Output | Meaning |
|--------|---------|
| `[+] The site ... is behind <WAF>` | WAF detected |
| `[-] No WAF detected` | No WAF or unknown WAF |
| `[!] Error connecting` | Connection failed |

---

## Workflow

```bash
# Step 1: Identify WAF
wafw00f https://target.com

# Step 2: If detected, note the WAF type
# Common: Cloudflare, ModSecurity, Akamai, Imperva, Wordfence

# Step 3: Run all plugins for confirmation
wafw00f -a https://target.com

# Step 4: Save report
wafw00f -a --json -o target_waf.json https://target.com

# Step 5: Research bypass techniques for detected WAF
```

---

## Chaining with Other Tools

```bash
# wafw00f + nikto
wafw00f https://target.com && nikto -h https://target.com

# wafw00f + nmap
wafw00f https://target.com && nmap -sV -p 80,443 --script http-server-header target.com

# wafw00f + sqlmap (after WAF identification)
wafw00f https://target.com
sqlmap -u "https://target.com/?id=1" --tamper=space2comment --random-agent
```

---

## Troubleshooting

```bash
# "No WAF detected" → run all plugins
wafw00f -a -v https://target.com

# SSL error → disable verification
wafw00f --no-check-ssl https://target.com

# Timeout → increase timeout + delay
wafw00f --timeout 30 --delay 5 https://target.com

# Rate limited → use proxy + reduce threads
wafw00f --proxy http://127.0.0.1:8080 -t 1 https://target.com

# Missing dependencies → reinstall
pip3 install --upgrade wafw00f
```

---

## One-Liners

```bash
# Quick detect
wafw00f https://target.com

# Thorough detect
wafw00f -a https://target.com

# JSON report
wafw00f --json https://target.com > report.json

# Subdomain enum
wafw00f -f -r https://target.com

# Through proxy
wafw00f --proxy http://127.0.0.1:8080 https://target.com
```

---

## WAF Categories

| Category | Examples |
|----------|----------|
| **Cloud-based** | Cloudflare, Akamai, AWS WAF, Azure Front Door |
| **Hardware** | F5 BIG-IP ASM, Citrix NetScaler |
| **Software** | ModSecurity, Wordfence, Sucuri |
| **CDN-integrated** | Imperva, Incapsula, StackPath |
| **Self-hosted** | NAXSI, Shadow Daemon, Wallarm |
