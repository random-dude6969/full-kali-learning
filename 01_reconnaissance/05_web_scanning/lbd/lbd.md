---
tool_name: "lbd"
display_name: "LBD (Load Balancing Detector)"
mitre_tactic: "reconnaissance"
mitre_tactic_id: "TA0043"
mitre_subcategory: "web_scanning"
install_command: "sudo apt install lbd"
version: "0.4"
github_repo: "https://github.com/cyberious/lbd"
kali_tools_page: "https://www.kali.org/tools/lbd/"
difficulty_level: "beginner"
requires_root: false
gui_available: false
tags: ["web", "load-balancer", "dns", "http"]
related_tools: ["whatweb", "wafw00f"]
---

# LBD (Load Balancing Detector)

## Chapter 1: Introduction & Overview

### What is LBD?

LBD (Load Balancing Detector) is a tool that detects if a domain uses DNS load balancing, HTTP/HTTPS load balancing, or both. Load balancing distributes incoming network traffic across multiple servers to ensure high availability, reliability, and performance. Understanding whether and how a target uses load balancing is critical for penetration testing because it reveals the number and diversity of backend servers, potential for direct backend access, and the overall architecture of the web infrastructure.

LBD works by performing multiple DNS queries to detect DNS-based load balancing (multiple A records for the same domain) and by making multiple HTTP requests with timing analysis to detect HTTP-based load balancing (different servers responding to different requests, indicated by varying Server headers, Set-Cookie values, or response timing).

### History & Background

- **Created:** Early 2000s
- **Maintained:** Community contributors
- **License:** GPL
- **Language:** Bash/Perl
- **Latest version:** 0.4
- **Upstream repository:** https://github.com/cyberious/lbd

LBD has been a standard tool in penetration testing distributions for years due to its simplicity and reliability. It provides a quick way to understand target infrastructure before beginning deeper testing.

### When to Use This Tool

- **Infrastructure mapping:** Identify load balancers and their configuration
- **Penetration testing:** Find direct backend server IPs for testing
- **Security assessment:** Identify load balancer misconfigurations
- **Architecture analysis:** Understand how traffic is distributed
- **CDN detection:** Determine if a CDN is in use
- **CTF challenges:** Solve infrastructure-based challenges
- **Pre-engagement:** Understand target complexity before deeper testing

### Legal and Ethical Considerations

- LBD performs minimal network activity (DNS queries and HTTP requests)
- This is considered passive/low-impact reconnaissance
- Use only on authorized targets
- Multiple requests may be logged by target systems
- Some load balancers may flag repeated requests as suspicious

### Key Concepts

| Concept | Description |
|---------|-------------|
| **DNS Load Balancing** | Distributing traffic by returning different IP addresses for the same domain |
| **HTTP Load Balancing** | Distributing traffic at the HTTP layer (different servers respond to requests) |
| **Round-Robin** | DNS technique cycling through a list of IP addresses |
| **Session Persistence** | Sticky sessions that keep users on the same backend server |
| **Health Checks** | Load balancer monitoring of backend server status |
| **Backend Server** | The actual web server behind the load balancer |
| **VIP** | Virtual IP — the load balancer's public-facing IP address |
| **CDN** | Content Delivery Network — a distributed load balancing system |

---

## Chapter 2: Installation & Setup

### System Requirements

- Operating system: Linux (recommended), macOS, or Windows (WSL)
- RAM: < 10 MB
- Disk space: < 1 MB
- Network: DNS and HTTP access to target
- Dependencies: `dig`, `curl`, `host` (usually pre-installed)
- Privileges: Non-root is sufficient

### Installation via APT (Kali/Debian/Ubuntu)

```bash
sudo apt update
sudo apt install lbd
```

Expected output:
```
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following NEW packages will be installed:
  lbd
0 upgraded, 1 newly installed, 0 to remove and 0 not upgraded.
Need to get 8.2 kB of archives.
After this operation, 32.8 kB of additional disk space will be used.
Get:1 http://http.kali.org/kali kali-rolling/main amd64 lbd 0.4-1kali1 [8.2 kB]
Fetched 8.2 kB in 0s (41 kB/s)
Selecting previously unselected package lbd.
(Reading database ... 1234567 files and directories currently installed.)
Preparing to unpack .../lbd_0.4-1kali1_amd64.deb ...
Unpacking lbd (0.4-1kali1) ...
Setting up lbd (0.4-1kali1) ...
```

### Installation from Source

```bash
git clone https://github.com/cyberious/lbd.git
cd lbd
chmod +x lbd
sudo cp lbd /usr/local/bin/
```

### Dependencies Check

```bash
# Verify required tools are installed
which dig curl host nslookup
```

Expected output:
```
/usr/bin/dig
/usr/bin/curl
/usr/bin/host
/usr/bin/nslookup
```

If any are missing:

```bash
sudo apt install dnsutils curl net-tools
```

### Verification

```bash
lbd --help
```

Expected output:
```
Load Balancing Detector 0.4 - detects DNS and HTTP Load Balancing (Server/Network/Cache)
Usage: lbd [-q] [-h] host

  -q quiet
  -h this help

  host the hostname to check
```

Or:

```bash
lbd 2>&1 | head -3
```

---

## Chapter 3: Basic Usage

### First Run

```bash
lbd example.com
```

Sample output:
```
Load Balancing Detector 0.4 - detects DNS and HTTP Load Balancing (Server/Network/Cache)

Checking for DNS load balancing...
DNS-Detection: NO DNS-LoadBalancing found! (Only 1 DNS A-Record)

Checking for HTTP load balancing...
HTTP-Server:  server
HTTP-Server:  server
HTTP-Server:  server
HTTP-Server:  server
HTTP-Server:  server
HTTP-Server:  server
HTTP-Server:  server
HTTP-Server:  server
HTTP-Detection: NO HTTP-LoadBalancing found! (All responses have the same server header)

Total time: 2.34 seconds
```

### DNS Load Balancing Detection

```bash
lbd example.com
```

**DNS Load Balancing Detected output:**
```
Checking for DNS load balancing...
DNS-Detection: DNS-LoadBalancing found! (4 DNS A-Records)
  192.168.1.1
  192.168.1.2
  192.168.1.3
  192.168.1.4
```

### HTTP Load Balancing Detection

```bash
lbd example.com
```

**HTTP Load Balancing Detected output:**
```
Checking for HTTP load balancing...
HTTP-Server:  Apache/2.4.41 (Ubuntu)
HTTP-Server:  nginx/1.18.0
HTTP-Server:  Apache/2.4.41 (Ubuntu)
HTTP-Server:  nginx/1.18.0
HTTP-Detection: HTTP-LoadBalancing found! (Different Server headers detected)
```

### Verbose Mode

```bash
lbd -v example.com
```

**Explanation:** Shows detailed output including timing information, response headers, and intermediate results.

### Quiet Mode

```bash
lbd -q example.com
```

**Explanation:** Minimal output — only shows load balancing status without detailed information.

### Complete Flags Reference

| Flag | Description | Default | Example |
|------|-------------|---------|---------|
| (none) | Standard detection mode | on | `lbd example.com` |
| `-q` | Quiet mode — minimal output | off | `lbd -q example.com` |
| `-h` | Show help message | — | `lbd -h` |
| positional | Target domain | none (required) | `lbd example.com` |

### How LBD Detection Works

**DNS Detection Method:**
1. Performs multiple DNS A-record queries for the target domain
2. Compares the returned IP addresses
3. If multiple unique IPs are returned → DNS load balancing detected
4. If only one IP → No DNS load balancing

**HTTP Detection Method:**
1. Makes multiple HTTP requests to the target
2. Compares response headers (Server, Set-Cookie, X-Powered-By, etc.)
3. If different headers are returned → HTTP load balancing detected
4. If all headers are identical → No HTTP load balancing

---

## Chapter 4: Intermediate Usage

### Bash Automation Scripts

#### Batch Load Balancer Detection

```bash
#!/bin/bash
# batch_lbd.sh — Check load balancing for multiple domains

DOMAINS_FILE="${1:?Usage: $0 <domains_file>}"
OUTPUT_DIR="lbd_results/$(date +%Y%m%d)"
mkdir -p "$OUTPUT_DIR"

echo "=== Load Balancer Detection ==="
echo "Domains file: $DOMAINS_FILE"
echo ""

DNS_LB=()
HTTP_LB=()
NO_LB=()

while IFS= read -r domain; do
    [ -z "$domain" ] && continue
    echo -n "[*] $domain: "
    
    RESULT=$(lbd "$domain" 2>&1)
    
    DNS_STATUS=$(echo "$RESULT" | grep "DNS-Detection:" | head -1)
    HTTP_STATUS=$(echo "$RESULT" | grep "HTTP-Detection:" | head -1)
    
    if echo "$DNS_STATUS" | grep -q "DNS-LoadBalancing found"; then
        DNS_IPS=$(echo "$RESULT" | grep -A 20 "DNS-Detection:" | grep "  [0-9]" | head -5)
        echo "DNS LB: YES ($DNS_IPS)"
        DNS_LB+=("$domain")
    else
        echo "DNS LB: NO"
    fi
    
    if echo "$HTTP_STATUS" | grep -q "HTTP-LoadBalancing found"; then
        HTTP_SERVERS=$(echo "$RESULT" | grep "HTTP-Server:" | head -5)
        echo "  HTTP LB: YES ($HTTP_SERVERS)"
        HTTP_LB+=("$domain")
    else
        echo "  HTTP LB: NO"
    fi
    
    echo "$RESULT" > "$OUTPUT_DIR/${domain}.txt"
    
done < "$DOMAINS_FILE"

# Summary
echo ""
echo "=== Summary ==="
echo "DNS Load Balanced: ${#DNS_LB[@]}"
echo "HTTP Load Balanced: ${#HTTP_LB[@]}"
echo "No Load Balancing: ${#NO_LB[@]}"
```

#### Automated Infrastructure Assessment

```bash
#!/bin/bash
# infra_assess.sh — Comprehensive load balancer and CDN assessment

TARGET="${1:?Usage: $0 <domain>}"
REPORT="infra_report_${TARGET}_$(date +%Y%m%d).md"

echo "=== Infrastructure Assessment: $TARGET ==="

cat > "$REPORT" << EOF
# Infrastructure Assessment: $TARGET
**Date:** $(date)
**Assessor:** $(whoami)

## Load Balancer Detection
EOF

# Run LBD
LBD_OUTPUT=$(lbd "$TARGET" 2>&1)
echo '```' >> "$REPORT"
echo "$LBD_OUTPUT" >> "$REPORT"
echo '```' >> "$REPORT"

# Check for CDN
echo ""
echo "[*] Checking for CDN..."
CDN_CHECK=$(curl -sI "http://$TARGET" | grep -iE "cf-ray|x-cdn|x-cache|x-served-by|via|server")
if [ -n "$CDN_CHECK" ]; then
    echo "  CDN detected:"
    echo "$CDN_CHECK"
    echo "" >> "$REPORT"
    echo "## CDN Detection" >> "$REPORT"
    echo '```' >> "$REPORT"
    echo "$CDN_CHECK" >> "$REPORT"
    echo '```' >> "$REPORT"
fi

# Check multiple IPs
echo ""
echo "[*] Checking resolved IPs..."
IPS=$(dig A "$TARGET" +short | grep -E "^[0-9]")
echo "$IPS"

echo "" >> "$REPORT"
echo "## Resolved IPs" >> "$REPORT"
echo "$IPS" >> "$REPORT"

echo ""
echo "[+] Report saved to $REPORT"
```

### Tool Chaining

#### LBD → Nmap Pipeline

```bash
#!/bin/bash
# lbd_nmap.sh — Detect load balancing then scan backend servers

TARGET="${1:?Usage: $0 <domain>}"

echo "[*] Detecting load balancing..."
LBD_OUTPUT=$(lbd "$TARGET" 2>&1)

# Extract IPs if DNS load balancing found
if echo "$LBD_OUTPUT" | grep -q "DNS-LoadBalancing found"; then
    echo "[!] DNS load balancing detected"
    IPS=$(echo "$LBD_OUTPUT" | grep -E "^\s+[0-9]" | awk '{print $1}')
    
    echo "[*] Scanning backend servers..."
    echo "$IPS" > target_ips.txt
    nmap -sV -p 80,443 -iL target_ips.txt -oA backend_scan
    
    echo "[*] Comparing server versions..."
    while IFS= read -r ip; do
        echo "--- $ip ---"
        curl -sI "http://$ip" | grep -i "server:"
    done <<< "$IPS"
else
    echo "[-] No DNS load balancing detected"
fi
```

#### LBD → Nikto Pipeline

```bash
#!/bin/bash
# lbd_nikto.sh — Scan each backend server with nikto

TARGET="${1:?Usage: $0 <domain>}"

# Get all IPs
IPS=$(dig A "$TARGET" +short | grep -E "^[0-9]")

echo "[*] Found $(echo "$IPS" | wc -l) IP addresses"
echo "[*] Running nikto on each..."

for ip in $IPS; do
    echo "[*] Scanning $ip..."
    nikto -h "$ip" -vhost "$TARGET" -o "nikto_${ip}.txt" 2>/dev/null
done
```

### Python Integration

```python
#!/usr/bin/env python3
"""
lbd_automation.py — Programmatic LBD integration
"""

import subprocess
import re
import json
from typing import Dict, List

def detect_load_balancing(domain: str) -> Dict:
    """Run LBD and parse results."""
    cmd = ["lbd", domain]
    result = subprocess.run(cmd, capture_output=True, text=True, timeout=60)
    
    output = result.stdout
    
    # Parse DNS load balancing
    dns_lb = "DNS-LoadBalancing found" in output
    dns_ips = []
    if dns_lb:
        ip_pattern = re.compile(r'^\s+(\d+\.\d+\.\d+\.\d+)', re.MULTILINE)
        dns_ips = ip_pattern.findall(output)
    
    # Parse HTTP load balancing
    http_lb = "HTTP-LoadBalancing found" in output
    http_servers = []
    if http_lb:
        server_pattern = re.compile(r'HTTP-Server:\s+(.+)', re.MULTILINE)
        http_servers = server_pattern.findall(output)
    
    # Get unique servers
    unique_servers = list(set(http_servers)) if http_servers else []
    
    return {
        "domain": domain,
        "dns_load_balancing": dns_lb,
        "dns_ips": dns_ips,
        "dns_ip_count": len(dns_ips),
        "http_load_balancing": http_lb,
        "http_servers": unique_servers,
        "http_server_count": len(unique_servers),
        "raw_output": output
    }

# Example usage
domains = ["google.com", "cloudflare.com", "example.com"]
results = []

for domain in domains:
    print(f"[*] Checking {domain}...")
    result = detect_load_balancing(domain)
    results.append(result)
    
    print(f"    DNS LB: {result['dns_load_balancing']} ({result['dns_ip_count']} IPs)")
    print(f"    HTTP LB: {result['http_load_balancing']} ({result['http_server_count']} servers)")

# Save report
with open("lbd_report.json", "w") as f:
    json.dump(results, f, indent=2)
print(f"\n[+] Report saved to lbd_report.json")
```

Expected output:
```
[*] Checking google.com...
    DNS LB: True (15 IPs)
    HTTP LB: True (3 servers)
[*] Checking cloudflare.com...
    DNS LB: True (8 IPs)
    HTTP LB: True (2 servers)
[*] Checking example.com...
    DNS LB: False (0 IPs)
    HTTP LB: False (0 servers)
```

---

## Chapter 5: Advanced Usage

### Detailed Timing Analysis

```bash
#!/bin/bash
# timing_analysis.sh — Analyze response timing for load balancing detection

TARGET="${1:?Usage: $0 <domain>}"
REQUESTS=20

echo "=== Timing Analysis: $TARGET ==="
echo "Making $REQUESTS requests..."

declare -A TIMINGS

for i in $(seq 1 $REQUESTS); do
    START=$(date +%s%N)
    curl -sI "http://$TARGET" > /dev/null 2>&1
    END=$(date +%s%N)
    ELAPSED=$(( (END - START) / 1000000 ))
    
    # Get response headers
    SERVER=$(curl -sI "http://$TARGET" 2>/dev/null | grep -i "^server:" | head -1 | tr -d '\r')
    
    echo "Request $i: ${ELAPSED}ms - $SERVER"
done
```

### Load Balancer Fingerprinting

```bash
#!/bin/bash
# lb_fingerprint.sh — Identify load balancer type

TARGET="${1:?Usage: $0 <domain>}"

echo "=== Load Balancer Fingerprinting: $TARGET ==="

# Check response headers
echo "[*] Response headers:"
HEADERS=$(curl -sI "http://$TARGET" 2>/dev/null)
echo "$HEADERS"

# Identify load balancer indicators
echo ""
echo "[*] Load balancer indicators:"

# F5 BigIP
echo "$HEADERS" | grep -qi "BigIP\|BIGipServer" && echo "  [!] F5 BigIP detected"

# Citrix NetScaler
echo "$HEADERS" | grep -qi "NS-CACHE\|NetScaler" && echo "  [!] Citrix NetScaler detected"

# HAProxy
echo "$HEADERS" | grep -qi "haproxy" && echo "  [!] HAProxy detected"

# Nginx
echo "$HEADERS" | grep -qi "nginx" && echo "  [i] Nginx detected (may be reverse proxy)"

# Apache
echo "$HEADERS" | grep -qi "apache" && echo "  [i] Apache detected"

# Cloudflare
echo "$HEADERS" | grep -qi "cf-ray\|cloudflare" && echo "  [!] Cloudflare CDN detected"

# AWS ALB/ELB
echo "$HEADERS" | grep -qi "awselb\|aws" && echo "  [!] AWS Load Balancer detected"

# Check for session persistence
echo ""
echo "[*] Session persistence check:"
COOKIES=$(curl -sI "http://$TARGET" 2>/dev/null | grep -i "set-cookie")
if echo "$COOKIES" | grep -qi "BIGipServer\|SERVERID\|JSESSIONID\|stickysession"; then
    echo "  [!] Session persistence (sticky sessions) detected"
    echo "$COOKIES"
else
    echo "  [+] No session persistence indicators"
fi
```

### Multi-Method Detection

```bash
#!/bin/bash
# multi_method_detect.sh — Comprehensive load balancer detection

TARGET="${1:?Usage: $0 <domain>}"

echo "=== Comprehensive Load Balancer Detection: $TARGET ==="

# Method 1: LBD
echo ""
echo "[Method 1] LBD Detection"
lbd "$TARGET" 2>&1 | grep -E "Detection:|DNS-LoadBalancing|HTTP-LoadBalancing"

# Method 2: DNS Analysis
echo ""
echo "[Method 2] DNS Analysis"
echo "A records:"
dig A "$TARGET" +short
echo ""
echo "AAAA records:"
dig AAAA "$TARGET" +short

# Method 3: HTTP Header Analysis
echo ""
echo "[Method 3] HTTP Header Analysis"
for i in $(seq 1 5); do
    curl -sI "http://$TARGET" 2>/dev/null | grep -iE "^server:|^x-powered-by:|^via:" | head -1
done | sort -u

# Method 4: Cookie Analysis
echo ""
echo "[Method 4] Cookie Analysis"
curl -sI "http://$TARGET" 2>/dev/null | grep -i "set-cookie" | head -5

# Method 5: Response Timing
echo ""
echo "[Method 5] Response Timing"
for i in $(seq 1 5); do
    curl -so /dev/null -w "Request $i: %{time_total}s - %{remote_ip}\n" "http://$TARGET" 2>/dev/null
done
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: Target Infrastructure Assessment

**Objective:** Understand the load balancing infrastructure of a target before a penetration test.

**Step 1: Basic LBD Check**
```bash
echo "[*] Running LBD..."
lbd target.com 2>&1 | tee lbd_basic.txt
```

Expected output:
```
Load Balancing Detector 0.4 - detects DNS and HTTP Load Balancing (Server/Network/Cache)

Checking for DNS load balancing...
DNS-Detection: DNS-LoadBalancing found! (3 DNS A-Records)
  104.16.100.1
  104.16.101.1
  104.16.102.1

Checking for HTTP load balancing...
HTTP-Server:  cloudflare
HTTP-Server:  cloudflare
HTTP-Server:  cloudflare
HTTP-Server:  cloudflare
HTTP-Detection: NO HTTP-LoadBalancing found! (All responses have the same server header)

Total time: 4.56 seconds
```

**Step 2: Analyze DNS Load Balancing**
```bash
echo "[*] Analyzing DNS records..."
dig A target.com +short
dig AAAA target.com +short
dig NS target.com +short
```

**Step 3: Check for CDN**
```bash
echo "[*] Checking for CDN..."
curl -sI http://target.com | grep -iE "cf-ray|x-cdn|x-cache|via|server"
```

**Step 4: Test Direct Backend Access**
```bash
echo "[*] Testing direct backend access..."
IPS=$(dig A target.com +short)
for ip in $IPS; do
    echo "--- $ip ---"
    curl -sI "http://$ip" -H "Host: target.com" 2>/dev/null | head -5
done
```

**Step 5: Generate Report**
```bash
cat > infrastructure_report.md << EOF
# Infrastructure Assessment: target.com

## DNS Load Balancing
- Detected: YES
- IPs: $(dig A target.com +short | tr '\n' ', ')

## HTTP Load Balancing
- Detected: $(grep -q "HTTP-LoadBalancing found" lbd_basic.txt && echo "YES" || echo "NO")

## CDN
- Detected: $(curl -sI http://target.com | grep -qi cloudflare && echo "YES (Cloudflare)" || echo "NO")

## Recommendations
1. Test each backend IP directly
2. Check for inconsistent security configurations
3. Test for CDN bypass vulnerabilities
EOF
```

### Scenario 2: CTF Challenge — "The Load Balancer is Misconfigured"

**Objective:** Exploit a load balancer misconfiguration in a CTF.

**Step 1: Initial Detection**
```bash
lbd ctf-target.com 2>&1 | tee ctf_lbd.txt
```

**Step 2: Identify Backend Servers**
```bash
# Extract all IP addresses
IPS=$(dig A ctf-target.com +short)
echo "Backend IPs: $IPS"

# Test each backend
for ip in $IPS; do
    echo "--- $ip ---"
    curl -sI "http://$ip" -H "Host: ctf-target.com" 2>/dev/null
done
```

**Step 3: Find Differences**
```bash
# Compare responses from each backend
for ip in $IPS; do
    echo "=== $ip ==="
    curl -s "http://$ip" -H "Host: ctf-target.com" 2>/dev/null | head -20
done
```

**Step 4: Check for Direct Access**
```bash
# Some backends might have direct access enabled
for ip in $IPS; do
    echo -n "$ip: "
    curl -s -o /dev/null -w "%{http_code}" "http://$ip" -H "Host: ctf-target.com" 2>/dev/null
    echo ""
done
```

### Scenario 3: CDN Bypass Assessment

**Objective:** Determine if a CDN can be bypassed to reach origin servers.

**Step 1: Check DNS Resolution**
```bash
echo "[*] DNS Resolution:"
dig A target.com +short
dig A www.target.com +short
```

**Step 2: Check HTTP Headers**
```bash
echo "[*] HTTP Headers:"
curl -sI http://target.com | head -20
```

**Step 3: Try Direct IP Access**
```bash
echo "[*] Direct IP access..."
ORIGIN_IPS=$(dig A target.com +short)

for ip in $ORIGIN_IPS; do
    echo "--- $ip ---"
    # Try accessing with different Host headers
    curl -sI "http://$ip" -H "Host: target.com" 2>/dev/null | head -3
    curl -sI "http://$ip" -H "Host: origin.target.com" 2>/dev/null | head -3
done
```

**Step 4: Check Common Bypass Techniques**
```bash
# Check for origin server on different ports
for port in 8080 8443 8000 3000 5000; do
    echo -n "Port $port: "
    curl -s -o /dev/null -w "%{http_code}" --max-time 2 "http://target.com:$port" 2>/dev/null
    echo ""
done

# Check for alternate hostnames
for host in origin.target.com direct.target.com backend.target.com; do
    echo -n "$host: "
    dig A "$host" +short
done
```

---

## Chapter 7: Detection & Defense

### How LBD Activity Is Detected

**DNS Query Patterns:**
- Multiple DNS queries for the same domain in short succession
- Queries from the same source IP
- Pattern of A-record queries followed by HTTP requests

**HTTP Request Patterns:**
- Multiple HTTP requests to the same domain with identical paths
- Requests arriving in rapid succession
- Requests from the same User-Agent

**Network Signatures:**
```
# Suricata rule for detecting load balancer enumeration
alert dns any any -> any 53 (msg:"DNS Load Balancer Enumeration"; \
  threshold:type both, track by_src, count 10, seconds 5; \
  sid:1000030; rev:1;)
```

### Blue Team Defensive Measures

**1. Consistent Backend Configuration:**
```bash
# Ensure all backend servers have identical configurations
# Same software versions, headers, and security settings
# Example: All backends should return the same Server header

# Apache configuration on all backends:
# ServerTokens Prod
# ServerSignature Off
```

**2. Remove Identifying Headers:**
```apache
# Apache - remove server information
ServerTokens Prod
ServerSignature Off
Header unset X-Powered-By
Header unset Server

# Nginx - remove server information
server_tokens off;
proxy_hide_header X-Powered-By;
```

**3. Standardize Response Behavior:**
```bash
# Ensure all backends return consistent:
# - Status codes
# - Response headers
# - Cookie names
# - Error pages
```

**4. Monitor for Enumeration:**
```bash
#!/bin/bash
# monitor_lb_enum.sh — Monitor for load balancer enumeration

LOG="/var/log/nginx/access.log"

# Detect multiple rapid requests from same IP
awk '{print $1}' "$LOG" | sort | uniq -c | sort -rn | head -20

# Detect header-based probing
grep -E "^(GET|HEAD)" "$LOG" | \
    awk '{print $1, $7}' | \
    sort | uniq -c | sort -rn | head -20
```

**5. Deploy Web Application Firewall:**
```nginx
# Nginx WAF rules to detect enumeration
limit_req_zone $binary_remote_addr zone=enum:10m rate=5r/s;

server {
    location / {
        limit_req zone=enum burst=10 nodelay;
    }
}
```

---

## Chapter 8: Troubleshooting

### Common Errors

#### "No load balancing"

**Cause:** Domain genuinely doesn't use load balancing, or detection method failed.

**Solutions:**
```bash
# Verify with manual checks
dig A target.com +short  # Should show multiple IPs for DNS LB
curl -sI target.com | grep -i server  # Compare headers

# Try more requests for HTTP LB detection
for i in $(seq 1 20); do
    curl -sI "http://target.com" | grep -i "^server:"
done | sort -u

# Check for CDN-based load balancing
curl -sI "http://target.com" | grep -iE "cf-ray|x-cache|via"
```

#### "Connection refused"

**Cause:** Target is unreachable or blocking connections.

**Solutions:**
```bash
# Check connectivity
ping target.com
curl -I http://target.com

# Check DNS resolution
dig A target.com +short

# Try HTTPS
lbd https://target.com
```

#### "Timeout"

**Cause:** Target is slow or blocking requests.

**Solutions:**
```bash
# Check network connectivity
traceroute target.com

# Try with different DNS server
dig @8.8.8.8 target.com +short

# Check if target is behind CDN
dig A target.com +short | xargs -I {} curl -sI "http://{}" -H "Host: target.com" 2>/dev/null | head -5
```

#### "DNS resolution failed"

**Cause:** Domain doesn't exist or DNS is misconfigured.

**Solutions:**
```bash
# Verify domain exists
whois target.com

# Check DNS records
dig NS target.com +short
dig A target.com +short

# Try alternate DNS servers
dig @8.8.8.8 target.com +short
dig @1.1.1.1 target.com +short
```

### Performance Optimization

```bash
# For batch operations, add delays
for domain in $(cat domains.txt); do
    lbd "$domain" > "lbd_${domain}.txt" 2>&1
    sleep 2  # Rate limiting
done

# Use parallel execution for faster batch processing
cat domains.txt | xargs -P 5 -I {} bash -c '
    lbd "{}" > "lbd_$(echo {}).txt" 2>&1
'
```

### FAQ

**Q: Can LBD detect specific load balancer brands?**
A: LBD detects load balancing presence but doesn't identify specific brands. Use `whatweb` or header analysis for load balancer fingerprinting.

**Q: Does LBD work with HTTPS?**
A: Yes, but you may need to handle certificate issues. Try `lbd https://target.com` or use curl with `-k` flag.

**Q: Can LBD detect CDN-based load balancing?**
A: LBD detects DNS-based load balancing (which CDNs use). For CDN-specific detection, check for CDN headers (cf-ray, x-cache, etc.).

**Q: How many requests does LBD make?**
A: LBD makes approximately 10-20 requests per detection method. For HTTP detection, it varies based on response patterns.

**Q: Does LBD work with IP addresses?**
A: LBD is designed for domain names. For IP-based testing, use direct HTTP requests.

**Q: Can LBD detect geo-based load balancing?**
A: Not directly. Geo-based DNS returns different IPs based on client location. LBD sees the IPs from its own location.

---

## Resources

- [LBD Man Page](https://linux.die.net/man/1/lbd)
- [LBD GitHub Repository](https://github.com/cyberious/lbd)
- [HAProxy Documentation](https://docs.haproxy.org)
- [Nginx Load Balancing](https://docs.nginx.com/nginx/admin-guide/load-balancer/http-load-balancer/)
- [F5 BIG-IP Documentation](https://www.f5.com/services/evaluations/big-ip)
- [AWS ELB Documentation](https://docs.aws.amazon.com/elasticloadbalancing/)
- [Cloudflare Learning Center](https://www.cloudflare.com/learning/)
- [OWASP Testing Guide — Infrastructure](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/02-Configuration_and_Deployment_Management_Testing)
- [Kali Linux LBD Package](https://www.kali.org/tools/lbd/)
- [CDN Detection — WhatIsMyIPAddress](https://www.whatismyipaddress.com)
