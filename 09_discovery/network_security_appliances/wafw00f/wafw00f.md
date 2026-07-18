---
title: "wafw00f"
description: "Web Application Firewall fingerprinting tool that identifies which WAF product protects a web application by analyzing HTTP response patterns and behavioral signatures."
categories: ["Network Security Appliances"]
tags: ["waf", "fingerprinting", "web-application", "firewall", "detection"]
mitre_tactic: "discovery"
mitre_tactic_id: "TA0007"
mitre_subcategory: "network_security_appliances"
install_command: "pip3 install wafw00f"
version: "2.4.2"
github_repo: "https://github.com/EnableSecurity/wafw00f"
kali_tools_page: "https://www.kali.org/tools/wafw00f/"
difficulty_level: "beginner"
requires_root: false
gui_available: false
related_tools: ["nikto", "whatweb", "nmap"]
---

# wafw00f

## Chapter 1: Introduction & Overview

### What is wafw00f?

wafw00f (pronounced "waff-waff") is a Web Application Firewall (WAF) fingerprinting tool that identifies which WAF product protects a given web application. It works by sending specially crafted HTTP requests and analyzing the responses to determine the WAF vendor and version.

wafw00f operates by:
1. Sending normal HTTP requests to establish baseline behavior
2. Sending malicious-looking HTTP requests (SQL injection, XSS patterns, etc.)
3. Analyzing response headers, status codes, and body content for WAF-specific signatures
4. Matching observed patterns against a database of 100+ WAF signatures

The tool is essential for penetration testers and security researchers who need to understand what defensive measures protect a target before launching application-layer attacks.

### History & Background

wafw00f was created by Wendel G. Henrique and is maintained by EnableSecurity.

**Key milestones:**
- **2013**: Initial release as a Python 2 tool
- **2018**: Major rewrite for Python 3 support
- **2020**: Plugin architecture introduced for extensible WAF detection
- **2023–present**: Version 2.4.x with 100+ WAF plugins, active maintenance

**License**: BSD-3-Clause

**GitHub**: https://github.com/EnableSecurity/wafw00f (6,500+ stars)

**Dependencies**: Python 3.10+, requests, dnspython, pldns

### When to Use This Tool

wafw00f is appropriate for:

- **Penetration testing** — identifying WAF protection before attempting SQL injection, XSS, or other web attacks
- **Security auditing** — verifying that WAF deployment is correctly configured and detectable
- **Red team operations** — understanding defensive posture before crafting bypass techniques
- **Bug bounty reconnaissance** — knowing what WAF protects a target helps choose appropriate attack vectors
- **Blue team validation** — confirming that WAF is correctly identified and cannot be easily bypassed
- **Compliance verification** — documenting WAF presence for security assessments

wafw00f is **not** appropriate for:
- Exploiting WAF vulnerabilities (use specialized tools)
- Testing WAF performance or throughput
- Replacing comprehensive vulnerability scanners

### Key Concepts

**WAF Signatures**: Each WAF product leaves identifiable traces in HTTP responses — specific headers, cookie names, error page content, and status code patterns. wafw00f maintains a plugin-based signature database.

**Detection Methods**: wafw00f uses three complementary techniques:
1. **Passive analysis**: Examining response headers and cookies from normal requests
2. **Active probing**: Sending crafted payloads that trigger WAF rules
3. **Pattern matching**: Comparing observed responses against known WAF fingerprints

**False Positives**: WAF detection is probabilistic. Some WAFs may not be detected, and some normal web servers may trigger false detections. Always verify results with manual inspection.

**Plugin Architecture**: Each WAF detection is implemented as a Python plugin. The community contributes new plugins for emerging WAF products.

## Chapter 2: Installation & Setup

### System Requirements

**Minimum requirements:**
- **OS**: Any system with Python 3.10+ (Kali Linux recommended)
- **RAM**: 256 MB
- **Disk space**: 50 MB
- **Network**: Internet access for target scanning
- **Privileges**: Not required (runs as standard user)

**Dependencies:**
- Python 3.10 or later
- `requests` — HTTP library
- `dnspython` — DNS resolution
- `pldns` — parallel DNS lookups
- `colorama` — colored terminal output

### Installation Methods

**Method 1: pip (recommended)**

```bash
pip3 install wafw00f
```

**Method 2: Kali Linux repositories**

```bash
sudo apt update
sudo apt install wafw00f
```

**Method 3: Build from source**

```bash
git clone https://github.com/EnableSecurity/wafw00f.git
cd wafw00f
pip3 install -e .
```

**Method 4: Docker**

```bash
docker pull enablesecurity/wafw00f
docker run enablesecurity/wafw00f https://example.org
```

### Configuration

wafw00f requires no configuration file. All options are specified via command-line flags.

**Proxy configuration (optional):**
```bash
# Route through Burp Suite for analysis
wafw00f --proxy http://127.0.0.1:8080 https://target.com

# SOCKS5 proxy
wafw00f --proxy socks5://127.0.0.1:1080 https://target.com
```

**Custom User-Agent:**
```bash
wafw00f --user-agent "Mozilla/5.0 (Custom)" https://target.com
```

### Verifying Installation

```bash
# Check version
wafw00f --version

# List all WAF plugins
wafw00f -l

# Quick test against a known WAF-protected site
wafw00f https://example.org
```

## Chapter 3: Core Concepts & Architecture

### How It Works

wafw00f's detection process follows a systematic approach:

**Stage 1: Normal Request**
```http
GET / HTTP/1.1
Host: target.com
User-Agent: wafw00f/2.4.2
```
The tool sends a benign HTTP request and records the response headers, status code, and body.

**Stage 2: Malicious Payload Requests**
wafw00f sends requests with patterns that typically trigger WAF rules:
- SQL injection: `?id=1' OR '1'='1`
- XSS: `<script>alert(1)</script>`
- Path traversal: `../../etc/passwd`
- Command injection: `; cat /etc/passwd`

**Stage 3: Response Analysis**
For each response, wafw00f checks:
- Response headers for WAF-specific headers (e.g., `X-CDN`, `X-WAF-Status`)
- Cookie names and values (e.g., `__cfduid` for Cloudflare)
- Response body for WAF-specific error pages
- Status codes (403, 406, 501 are common WAF responses)
- Response time anomalies

**Stage 4: Plugin Matching**
Each WAF plugin analyzes the collected data and determines if its specific WAF is present. Plugins use:
- Header signature matching
- Cookie pattern detection
- Error page fingerprinting
- Behavioral analysis

### Protocols & Standards

**HTTP/1.1 (RFC 7230-7235)**:
- wafw00f sends standard HTTP/1.1 requests
- Supports HTTP/2 if the target server advertises it
- Analyzes standard HTTP headers and status codes

**WAF Detection Taxonomy**:
- **Cloud-based**: Cloudflare, Akamai, AWS WAF, Azure Front Door
- **Hardware appliances**: F5 BIG-IP ASM, Citrix NetScaler
- **Software**: ModSecurity, Wordfence, Sucuri
- **CDN-integrated**: Imperva, Incapsula, StackPath

### Network Architecture

```
[wafw00f] ──HTTP──> [Target Server] ──> [WAF/CDN]
    │                    │                    │
    │  Normal request    │                    │
    │  ───────────────>  │                    │
    │  <───────────────  │                    │
    │                    │                    │
    │  Malicious request │                    │
    │  ───────────────>  │                    │
    │                    │  ──> Blocked       │
    │  <── 403 Forbidden │  <── WAF Error     │
    │                    │                    │
    │  Response analysis │                    │
    │  Headers: X-CDN: Cloudflare            │
    │  Cookies: __cfduid                     │
    │  Body: "Attention Required!"           │
    │  => WAF: Cloudflare                    │
```

## Chapter 4: Command Reference

### All Flags/Options

| Flag | Long Form | Description |
|------|-----------|-------------|
| `-l` | `--list` | List all available WAF detection plugins |
| `-a` | `--all` | Run all plugins (slower, more thorough) |
| `-f` | `--find-subdomains` | Find subdomains of target |
| `-r` | `--recursion` | Recursively scan discovered subdomains |
| `-p <plugin>` | `--plugin` | Run specific plugin(s) |
| `-o <file>` | `--output` | Output results to file |
| `-v` | `--verbose` | Verbose output |
| `-d` | `--debug` | Debug output |
| `--proxy <url>` | | Use proxy (HTTP/SOCKS5) |
| `--delay <sec>` | | Delay between requests |
| `-t <num>` | `--threads` | Number of concurrent threads |
| `--json` | | Output in JSON format |
| `--xml` | | Output in XML format |
| `--user-agent <ua>` | | Custom User-Agent string |
| `-c <cookies>` | `--cookie` | Send cookies with requests |
| `-H <header>` | `--header` | Custom HTTP header |
| `-e <email>` | `--email` | Email for reporting |
| `--no-check-ssl` | | Disable SSL certificate verification |
| `--follow-redirects` | | Follow HTTP redirects |
| `-h` | `--help` | Display help |

### Output Formats

**Standard output:**
```
[*] https://example.org
[+] The site https://example.org is behind Cloudflare (Cloudflare Inc.)
```

**JSON output:**
```json
{
  "https://example.org": {
    "detected": true,
    "firewall": "Cloudflare",
    "manufacturer": "Cloudflare Inc."
  }
}
```

**XML output:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<wafw00f>
  <target url="https://example.org">
    <firewall name="Cloudflare" manufacturer="Cloudflare Inc."/>
  </target>
</wafw00f>
```

## Chapter 5: Usage Examples

### Basic Usage

**Scan a single target:**

```bash
wafw00f https://example.org
```

**Scan with verbose output:**

```bash
wafw00f -v https://example.org
```

**Scan without SSL verification:**

```bash
wafw00f --no-check-ssl https://self-signed.example.com
```

### Intermediate Usage

**Run all plugins for thorough detection:**

```bash
wafw00f -a https://example.org
```

**Run specific plugins:**

```bash
# List available plugins first
wafw00f -l

# Run Cloudflare and ModSecurity plugins only
wafw00f -p cloudflare -p modsecurity https://example.org
```

**Save results to file:**

```bash
wafw00f -o results.json --json https://example.org
```

**Use proxy for analysis:**

```bash
# Route through Burp Suite
wafw00f --proxy http://127.0.0.1:8080 https://example.org
```

### Advanced Usage

**Scan subdomains:**

```bash
# Find and scan subdomains
wafw00f -f -r https://example.org
```

**Custom headers and cookies:**

```bash
wafw00f \
  -H "Authorization: Bearer token123" \
  -c "session=abc123" \
  https://protected.example.com
```

**Batch scan with delay:**

```bash
# Scan multiple targets with 2-second delay
for target in https://site1.com https://site2.com https://site3.com; do
    wafw00f --delay 2 "$target"
done
```

**Threaded scanning:**

```bash
# Use 10 threads for faster scanning
wafw00f -t 10 https://example.org
```

### Real-World Scenarios

**Scenario 1: Penetration test — identify WAF before SQL injection**

```bash
# Step 1: Identify WAF
wafw00f https://target.com

# Step 2: If WAF detected, check for bypass techniques
# Cloudflare → try origin IP bypass
# ModSecurity → try encoding bypass
# Wordfence → try IP whitelisting
```

**Scenario 2: Bug bounty — enumerate WAF across subdomains**

```bash
wafw00f -f -r --json -o waf_report.json https://program.example.com
```

**Scenario 3: Compliance audit — document WAF presence**

```bash
# Generate XML report for management
wafw00f --xml -o compliance_report.xml https://app.example.com
wafw00f --xml -o compliance_report2.xml https://api.example.com
```

**Scenario 4: Blue team — verify WAF detection resistance**

```bash
# Check if WAF can be fingerprinted
wafw00f -v https://corporate.example.com

# If detected, investigate header leakage
wafw00f -v -d https://corporate.example.com
```

## Chapter 6: Advanced Techniques

### Automation & Scripting

**Batch WAF detection script:**

```bash
#!/bin/bash
# detect_wafs.sh — scan multiple targets and generate report
TARGETS_FILE="targets.txt"
OUTPUT="waf_report_$(date +%Y%m%d).json"

echo "[]" > "$OUTPUT"

while IFS= read -r target; do
    echo "=== Scanning $target ==="
    wafw00f --json "$target" >> "$OUTPUT"
    sleep 1
done < "$TARGETS_FILE"

echo "Report saved to $OUTPUT"
```

**Automated WAF bypass testing:**

```bash
#!/bin/bash
# test_waf_bypass.sh
TARGET="https://target.com"
WAF=$(wafw00f "$TARGET" 2>&1 | grep -oP 'behind \K[^ ]+')

echo "Detected WAF: $WAF"

case "$WAF" in
    "Cloudflare")
        echo "Trying origin IP discovery..."
        dig +short target.com
        # Try direct IP connection
        ;;
    "ModSecurity")
        echo "Trying encoding bypass..."
        # Test various encoding techniques
        ;;
    "Wordfence")
        echo "Trying IP whitelisting..."
        # Test with different source IPs
        ;;
    *)
        echo "Unknown WAF or no WAF detected"
        ;;
esac
```

### Chaining with Other Tools

**wafw00f + nikto for comprehensive scanning:**

```bash
# Step 1: Identify WAF
wafw00f https://target.com

# Step 2: Run nikto with WAF-aware settings
nikto -h https://target.com -Tuning 1234567890abc -Display V

# Step 3: If WAF blocks nikto, use wafw00f proxy
wafw00f --proxy http://127.0.0.1:8080 https://target.com
```

**wafw00f + nmap for service fingerprinting:**

```bash
# Step 1: Identify WAF
wafw00f https://target.com

# Step 2: Scan for web server details
nmap -sV -p 80,443 --script http-server-header,http-title target.com

# Step 3: Check for WAF-related open ports
nmap -p 80,443,8080,8443 target.com
```

**wafw00f + sqlmap for WAF-aware SQL injection:**

```bash
# Step 1: Identify WAF
wafw00f https://target.com

# Step 2: Use sqlmap with WAF bypass techniques
sqlmap -u "https://target.com/?id=1" --tamper=space2comment,between --random-agent
```

### Custom Configurations

**Custom User-Agent for stealth:**

```bash
# Mimic real browser
wafw00f --user-agent "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36" https://target.com
```

**Custom headers for authenticated scanning:**

```bash
wafw00f \
  -H "X-Forwarded-For: 127.0.0.1" \
  -H "X-Real-IP: 192.168.1.100" \
  https://target.com
```

### Performance Tuning

**Increase thread count for faster scanning:**

```bash
wafw00f -t 20 https://target.com
```

**Reduce delay between requests:**

```bash
wafw00f --delay 0.5 https://target.com
```

**Skip DNS resolution:**

```bash
wafw00f --no-dns https://192.168.1.100
```

## Chapter 7: Detection & Defense

### How to Detect

WAF fingerprinting generates detectable traffic patterns:

1. **HTTP Request Patterns**: wafw00f sends requests with SQL injection and XSS payloads. These are flagged by the WAF itself.

2. **Sequential Probing**: wafw00f probes multiple endpoints with different payloads. This pattern is detectable by request logging.

3. **User-Agent Fingerprint**: The default wafw00f User-Agent (`wafw00f/2.4.2`) is easily fingerprinted.

4. **Request Timing**: Automated requests have more regular timing than human browsing.

### Defensive Measures

**WAF rule updates:**
```
# ModSecurity: Add rule to block wafw00f
SecRule REQUEST_HEADERS:User-Agent "@contains wafw00f" \
  "id:1000001,phase:1,deny,status:403,\
  msg:'WAF fingerprinting attempt blocked'"
```

**Rate limiting:**
```bash
# Limit requests per IP
sudo iptables -A INPUT -p tcp --dport 80 -m limit --limit 30/min -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 80 -j DROP
```

**Request validation:**
```
# Block requests with SQL injection patterns in URL
SecRule ARGS "@rx (?:union|select|insert|update|delete|drop)" \
  "id:1000002,phase:2,deny,status:403"
```

### IDS/IPS Signatures

**Snort rule for wafw00f detection:**

```
alert http any any -> any $HTTP_PORTS (msg:"WAF Fingerprinting - wafw00f"; \
  content:"wafw00f"; http_user_agent; \
  sid:1000004; rev:1; classtype:web-application-attack;)
```

**Suricata rule:**

```
alert http any any -> any any (msg:"WAF Fingerprinting Attempt"; \
  http.user_agent; content:"wafw00f"; nocase; \
  sid:2000002; rev:1;)
```

### Log Analysis

**Apache/Nginx access log monitoring:**
```bash
# Detect wafw00f User-Agent
grep -i "wafw00f" /var/log/apache2/access.log

# Detect SQL injection patterns in requests
grep -E "(union|select|insert|update|delete|drop)" /var/log/nginx/access.log

# Detect sequential probing
awk '{print $1}' /var/log/apache2/access.log | sort | uniq -c | sort -rn | head -20
```

**WAF log analysis:**
```bash
# ModSecurity audit log
grep "wafw00f" /var/log/modsec_audit.log

# Check blocked requests
grep "403" /var/log/modsec_audit.log | tail -20
```

## Chapter 8: Troubleshooting & FAQ

### Common Issues

**Problem: "No WAF detected" for a site behind a WAF**

Cause: The WAF may not be responding to wafw00f's probes, or the site may be using a non-standard WAF configuration.

Solution:
```bash
# Run all plugins for thorough detection
wafw00f -a https://target.com

# Try with verbose output
wafw00f -v https://target.com

# Check if the site responds at all
curl -I https://target.com

# Try different User-Agent
wafw00f --user-agent "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36" https://target.com
```

**Problem: SSL certificate verification error**

Cause: Self-signed or invalid SSL certificate on the target.

Solution:
```bash
# Disable SSL verification
wafw00f --no-check-ssl https://target.com

# Or use HTTP instead of HTTPS
wafw00f http://target.com
```

**Problem: Connection timeout**

Cause: Target is blocking or rate-limiting connections.

Solution:
```bash
# Increase timeout
wafw00f --timeout 30 https://target.com

# Use proxy to avoid IP blocking
wafw00f --proxy http://127.0.0.1:8080 https://target.com

# Add delay between requests
wafw00f --delay 5 https://target.com
```

**Problem: "ModuleNotFoundError" for dependencies**

Cause: Missing Python packages.

Solution:
```bash
# Reinstall with dependencies
pip3 install --upgrade wafw00f

# Install specific dependencies
pip3 install requests dnspython pldns colorama
```

**Problem: Rate limiting by target WAF**

Cause: Target WAF is blocking wafw00f after detecting automated requests.

Solution:
```bash
# Use proxy rotation
wafw00f --proxy socks5://proxy1:1080 https://target.com

# Increase delay
wafw00f --delay 10 https://target.com

# Reduce thread count
wafw00f -t 1 https://target.com
```

### FAQ

**Q: Can wafw00f bypass WAFs?**
A: No. wafw00f only identifies WAFs. Use tools like sqlmap with tamper scripts for bypass attempts.

**Q: Does wafw00f work with non-HTTP services?**
A: No. wafw00f is designed exclusively for HTTP/HTTPS web applications.

**Q: Can wafw00f detect WAF version?**
A: Some plugins can detect specific versions, but not all WAFs expose version information.

**Q: Is wafw00f legal to use?**
A: wafw00f is a legitimate security tool. Use it only on systems you own or have authorization to test.

**Q: Can wafw00f detect cloud WAFs?**
A: Yes. wafw00f detects cloud WAFs including Cloudflare, Akamai, AWS WAF, Azure Front Door, and many others.

**Q: How accurate is wafw00f?**
A: Accuracy varies by WAF. Well-known WAFs (Cloudflare, ModSecurity) have high detection rates. Custom or niche WAFs may not be detected. Always verify results manually.

**Q: Does wafw00f send attack payloads to the target?**
A: Yes. wafw00f sends SQL injection and XSS payloads that may be logged by the WAF. Use with caution in production environments.

## Resources

### Official Documentation
- **wafw00f GitHub**: https://github.com/EnableSecurity/wafw00f
- **wafw00f documentation**: https://github.com/EnableSecurity/wafw00f/blob/master/README.md
- **Kali tools page**: https://www.kali.org/tools/wafw00f/

### Online Resources
- **OWASP WAF**: https://owasp.org/www-community/
- **WAF Bypass Techniques**: https://www.contextis.com/
- **SecurityHeaders.com**: https://securityheaders.com/

### Related Tools
- **nikto** — Web server vulnerability scanner
- **whatweb** — Web technology fingerprinting
- **nmap** — Network scanner with HTTP scripts
- **sqlmap** — SQL injection automation
- **Wapiti** — Web application vulnerability scanner
- **Arachni** — Web application security scanner
