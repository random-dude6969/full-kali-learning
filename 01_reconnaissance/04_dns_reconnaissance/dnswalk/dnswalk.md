---
tool_name: "dnswalk"
display_name: "DNSwalk"
mitre_tactic: "reconnaissance"
mitre_tactic_id: "TA0043"
mitre_subcategory: "dns_reconnaissance"
install_command: "sudo apt install dnswalk"
version: "0.6.1"
github_repo: "https://github.com/fgarber/dnswalk"
kali_tools_page: "https://www.kali.org/tools/dnswalk/"
difficulty_level: "beginner"
requires_root: false
gui_available: false
tags: ["dns", "zone-transfer", "recon"]
related_tools: ["dnsenum", "dnsrecon", "dnstracer"]
---

# DNSwalk

## Chapter 1: Introduction & Overview

### What is DNSwalk?

DNSwalk is a DNS database query tool primarily designed to test for DNS zone transfer vulnerabilities. It attempts to perform AXFR (zone transfer) requests against DNS servers to enumerate all records in a zone. When zone transfers are improperly configured and allowed from unauthorized sources, DNSwalk can retrieve the entire DNS database for a domain, revealing subdomains, mail servers, internal hostnames, and other infrastructure details that should remain private.

Beyond zone transfer testing, DNSwalk also performs basic DNS record queries to build a comprehensive picture of a domain's DNS configuration. It checks for common misconfigurations such as missing glue records, CNAME loops, and delegation inconsistencies. The tool is lightweight, fast, and requires minimal configuration, making it a go-to choice for quick DNS security assessments.

### History & Background

- **Created:** Early 2000s by fgarber
- **Maintained:** Community contributors
- **License:** Public Domain / BSD
- **Language:** Perl
- **Package availability:** Kali Linux, Debian, Ubuntu
- **Upstream repository:** https://github.com/fgarber/dnswalk

DNSwalk was one of the earliest dedicated zone transfer testing tools. It was written in Perl to be portable across Unix systems and has been included in security distributions like Kali Linux for over a decade. While newer tools like `dnsenum` offer more features, DNSwalk remains valued for its simplicity and reliability.

### When to Use This Tool

- **Zone transfer testing:** Check if DNS servers allow unauthorized AXFR requests — the primary use case
- **DNS enumeration:** When zone transfers succeed, get a complete listing of all DNS records
- **Security auditing:** Identify DNS misconfigurations that expose internal infrastructure
- **Penetration testing:** Gather comprehensive DNS intelligence passively
- **Compliance checking:** Verify that DNS servers comply with security best practices
- **CTF challenges:** Solve challenges involving DNS misconfigurations

### Legal and Ethical Considerations

- Zone transfer requests are standard DNS protocol operations
- A successful zone transfer indicates a misconfiguration, not a hack
- Document all findings for remediation reports
- Only test domains you own or have written authorization to test
- Zone transfer attempts are logged by properly configured DNS servers
- Some DNS servers may rate-limit or block repeated AXFR attempts
- This tool is for authorized security assessments only

### Key Concepts

| Concept | Description |
|---------|-------------|
| **Zone Transfer (AXFR)** | Full replication of a DNS zone from a primary server to a secondary server |
| **IXFR** | Incremental zone transfer — only changes since last transfer |
| **Primary Server** | The authoritative DNS server that holds the master copy of zone data |
| **Secondary Server** | A DNS server that receives zone data via transfer from the primary |
| **SOA Record** | Start of Authority — contains zone metadata including primary NS and serial |
| **Glue Records** | NS records provided by the parent zone to avoid circular dependencies |
| **CNAME Loop** | A circular chain of CNAME records that can cause resolution failures |

---

## Chapter 2: Installation & Setup

### System Requirements

- Operating system: Linux, macOS, or Windows (with Perl)
- RAM: Minimal (< 5 MB)
- Disk space: < 1 MB
- Network: UDP/TCP port 53 access to target DNS servers
- Perl: Version 5.10 or higher (usually pre-installed on Linux)

### Installation via APT (Kali/Debian/Ubuntu)

```bash
sudo apt update
sudo apt install dnswalk
```

Expected output:
```
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following NEW packages will be installed:
  dnswalk
0 upgraded, 1 newly installed, 0 to remove and 0 not upgraded.
Need to get 19.2 kB of archives.
After this operation, 98.3 kB of additional disk space will be used.
Get:1 http://http.kali.org/kali kali-rolling/main all dnswalk 0.6.1-1kali1 [19.2 kB]
Fetched 19.2 kB in 0s (96.5 kB/s)
Selecting previously unselected package dnswalk.
(Reading database ... 1234567 files and directories currently installed.)
Preparing to unpack .../dnswalk_0.6.1-1kali1_all.deb ...
Unpacking dnswalk (0.6.1-1kali1) ...
Setting up dnswalk (0.6.1-1kali1) ...
```

### Installation from Source

```bash
# Install Perl if not present
sudo apt install perl

# Clone the repository
git clone https://github.com/fgarber/dnswalk.git
cd dnswalk

# Build (if needed)
make

# Install to /usr/local/bin
sudo cp dnswalk /usr/local/bin/
sudo chmod +x /usr/local/bin/dnswalk
```

### Installation via CPAN

```bash
sudo cpan install DNS::ZoneParse
# Then clone and run dnswalk directly
```

### Configuration

DNSwalk uses Perl's `Net::DNS` module. Ensure it's installed:

```bash
sudo apt install libnet-dns-perl
```

Verify the dependency:

```bash
perl -e 'use Net::DNS; print "Net::DNS OK\n"'
```

Expected output:
```
Net::DNS OK
```

### Verification

```bash
dnswalk --help 2>&1 | head -20
```

Expected output:
```
Usage: dnswalk [options] domain.
Options:
  -d      Debug mode
  -f      Force (don't ask confirmation)
  -l      Log to file
  -m      Check for mail server
  -r      Recursive check
  -v      Verbose
```

---

## Chapter 3: Basic Usage

### First Run

```bash
dnswalk target.com.
```

**Important:** DNSwalk requires the domain to end with a trailing dot (FQDN format). This is standard DNS notation for a fully qualified domain name.

Sample output when zone transfer is refused:
```
Getting zone transfer of target.com. from ns1.target.com. ...
AXFR denied by server.
Getting zone transfer of target.com. from ns2.target.com. ...
AXFR denied by server.
```

Sample output when zone transfer succeeds:
```
Getting zone transfer of target.com. from ns1.target.com. ...
Received 247 records.
target.com.              SOA     ns1.target.com. admin.target.com. 2024010101 3600 900 604800 86400
target.com.              NS      ns1.target.com.
target.com.              NS      ns2.target.com.
target.com.              MX      10 mail.target.com.
target.com.              MX      20 mail2.target.com.
target.com.              A       192.168.1.100
www.target.com.          A       192.168.1.100
mail.target.com.         A       192.168.1.101
mail2.target.com.        A       192.168.1.102
ftp.target.com.          A       192.168.1.103
internal.target.com.     A       10.0.0.50
vpn.target.com.          A       10.0.0.10
dev.target.com.          A       192.168.2.50
staging.target.com.      A       192.168.2.100
...
```

### Essential Commands

#### Basic Zone Transfer Test

```bash
dnswalk target.com.
```

**Explanation:** Attempts zone transfer from all authoritative nameservers for target.com. Reports success or failure for each server.

#### Verbose Output

```bash
dnswalk -v target.com.
```

**Explanation:** Shows detailed output including each DNS query performed and the responses received. Useful for understanding exactly what DNSwalk is doing.

#### Force Mode (Skip Confirmation)

```bash
dnswalk -f target.com.
```

**Explanation:** Automatically proceeds without asking for user confirmation. Essential for scripting and automation.

#### Check Specific DNS Server

```bash
dnswalk @ns1.target.com. target.com.
```

**Explanation:** Tests zone transfer against a specific nameserver rather than discovering and testing all authoritative servers.

#### Debug Mode

```bash
dnswalk -d target.com.
```

**Explanation:** Shows raw DNS packet data and internal debugging information. Useful for diagnosing why zone transfers fail.

#### Log Output to File

```bash
dnswalk -l output.txt target.com.
```

**Explanation:** Writes the complete output to a file for later analysis or inclusion in reports.

#### Check Mail Server Configuration

```bash
dnswalk -m target.com.
```

**Explanation:** Additionally checks the mail server (MX) configuration for common issues like missing A records for mail servers.

#### Recursive Check

```bash
dnswalk -r target.com.
```

**Explanation:** Recursively checks all discovered subdomains for zone transfer vulnerabilities.

### Complete Flags Reference

| Flag | Long Form | Description | Default | Example |
|------|-----------|-------------|---------|---------|
| `-d` | `--debug` | Enable debug mode with verbose packet output | off | `dnswalk -d target.com.` |
| `-f` | `--force` | Skip confirmation prompt | off | `dnswalk -f target.com.` |
| `-l` | `--log` | Log output to specified file | none | `dnswalk -l output.txt target.com.` |
| `-m` | `--mail` | Check MX record configuration | off | `dnswalk -m target.com.` |
| `-r` | `--recursive` | Recursively check discovered subdomains | off | `dnswalk -r target.com.` |
| `-v` | `--verbose` | Enable verbose output | off | `dnswalk -v target.com.` |
| `@server` | — | Test specific DNS server | auto-discover | `dnswalk @ns1.target.com. target.com.` |

### Output Format

DNSwalk output includes several record types when zone transfer succeeds:

```
domain.com.              SOA     ns1.domain.com. admin.domain.com. SERIAL REFRESH RETRY EXPIRE MINIMUM
domain.com.              NS      ns1.domain.com.
domain.com.              NS      ns2.domain.com.
domain.com.              MX      priority mailserver.domain.com.
domain.com.              A       IP_ADDRESS
subdomain.domain.com.    A       IP_ADDRESS
subdomain.domain.com.    CNAME   target.domain.com.
```

**Record types in zone transfer output:**
- **SOA:** Start of Authority — zone metadata
- **NS:** Nameserver records
- **MX:** Mail exchange records
- **A:** IPv4 address records
- **AAAA:** IPv6 address records
- **CNAME:** Canonical name (alias) records
- **TXT:** Text records (SPF, DKIM, etc.)
- **SRV:** Service records
- **PTR:** Pointer records (reverse DNS)

---

## Chapter 4: Intermediate Usage

### Bash Automation Scripts

#### Batch Zone Transfer Testing

```bash
#!/bin/bash
# zone_transfer_test.sh — Test zone transfers for multiple domains

DOMAINS=("example.com" "target.org" "test.net" "company.co.uk")
OUTPUT_DIR="zone_transfer_results/$(date +%Y%m%d)"
mkdir -p "$OUTPUT_DIR"

echo "=== Zone Transfer Testing ==="
echo "Started: $(date)"
echo ""

for domain in "${DOMAINS[@]}"; do
    echo "[*] Testing: $domain"
    
    # Run dnswalk with force mode and logging
    dnswalk -f -l "$OUTPUT_DIR/${domain}_axfr.txt" "${domain}." 2>&1
    
    # Check if zone transfer succeeded
    if grep -q "Received" "$OUTPUT_DIR/${domain}_axfr.txt" 2>/dev/null; then
        RECORD_COUNT=$(grep -c "^[a-zA-Z]" "$OUTPUT_DIR/${domain}_axfr.txt" 2>/dev/null)
        echo "[!] VULNERABLE: $domain — $RECORD_COUNT records obtained"
        echo "$domain" >> "$OUTPUT_DIR/vulnerable_domains.txt"
    else
        echo "[-] Secure: $domain — zone transfer denied"
    fi
    echo ""
done

echo "=== Testing Complete ==="
echo "Results saved to $OUTPUT_DIR"
if [ -f "$OUTPUT_DIR/vulnerable_domains.txt" ]; then
    echo "Vulnerable domains:"
    cat "$OUTPUT_DIR/vulnerable_domains.txt"
fi
```

Expected output:
```
=== Zone Transfer Testing ===
Started: 2024-07-18 14:30:00

[*] Testing: example.com
[-] Secure: example.com — zone transfer denied

[*] Testing: target.org
[!] VULNERABLE: target.org — 247 records obtained

[*] Testing: test.net
[-] Secure: test.net — zone transfer denied

[*] Testing: company.co.uk
[-] Secure: company.co.uk — zone transfer denied

=== Testing Complete ===
Results saved to zone_transfer_results/20240718
Vulnerable domains:
target.org
```

#### Automated DNS Security Check

```bash
#!/bin/bash
# dns_security_check.sh — Comprehensive DNS security check

DOMAIN="${1:?Usage: $0 <domain>}"

echo "=== DNS Security Check: $DOMAIN ==="
echo ""

# Step 1: Get nameservers
echo "[1/4] Discovering nameservers..."
NS_SERVERS=$(dig NS "${DOMAIN}" +short)
echo "  Found nameservers:"
echo "$NS_SERVERS" | while read ns; do
    echo "    - $ns"
done

# Step 2: Test zone transfer on each
echo ""
echo "[2/4] Testing zone transfers..."
echo "$NS_SERVERS" | while read ns; do
    ns_clean=$(echo "$ns" | tr -d '.')
    echo "  Testing $ns..."
    RESULT=$(dnswalk -f "@${ns}" "${DOMAIN}." 2>&1)
    if echo "$RESULT" | grep -q "Received"; then
        echo "    [!] ZONE TRANSFER ALLOWED from $ns"
    else
        echo "    [+] Zone transfer denied by $ns"
    fi
done

# Step 3: Check DNSSEC
echo ""
echo "[3/4] Checking DNSSEC..."
DNSKEY=$(dig DNSKEY "${DOMAIN}" +dnssec +short)
if [ -n "$DNSKEY" ]; then
    echo "  [+] DNSSEC is enabled"
else
    echo "  [-] DNSSEC is NOT enabled"
fi

# Step 4: Check for open resolvers
echo ""
echo "[4/4] Checking for open resolvers..."
for ns in $NS_SERVERS; do
    ns_ip=$(dig A "$ns" +short | head -1)
    if [ -n "$ns_ip" ]; then
        RESULT=$(dig @${ns_ip} google.com +short 2>/dev/null)
        if [ -n "$RESULT" ]; then
            echo "  [!] Open resolver detected: $ns_ip"
        fi
    fi
done

echo ""
echo "=== Check Complete ==="
```

### Tool Chaining

#### Chain with dnsenum for Complete DNS Recon

```bash
# Step 1: Test zone transfer
dnswalk -f target.com. > zone_transfer.txt 2>&1

# Step 2: If zone transfer succeeded, extract records
if grep -q "Received" zone_transfer.txt; then
    echo "[!] Zone transfer successful!"
    grep -E "^[a-zA-Z]" zone_transfer.txt | awk '{print $1}' | sort -u > discovered_hosts.txt
    echo "[+] $(wc -l < discovered_hosts.txt) unique hosts discovered"
else
    echo "[-] Zone transfer denied, using alternative enumeration"
    dnsenum target.com >> zone_transfer.txt
fi
```

#### Chain with nmap for Service Discovery

```bash
# Step 1: Get hosts from zone transfer
dnswalk -f target.com. 2>&1 | grep -E "A\s+" | awk '{print $3}' > target_ips.txt

# Step 2: Scan discovered hosts
nmap -sV -p 80,443,22,21,25,110 -iL target_ips.txt -oA zone_transfer_scan
```

#### Chain with nikto for Web Vulnerability Scanning

```bash
# Step 1: Get web servers from zone transfer
dnswalk -f target.com. 2>&1 | grep -E "A\s+" | awk '{print $1, $3}' | while read hostname ip; do
    echo "[*] Scanning $hostname ($ip)..."
    nikto -h "$ip" -vhost "$hostname" -o "nikto_${hostname}.txt"
done
```

### Python Integration

```python
#!/usr/bin/env python3
"""
dnswalk_automation.py — Automated zone transfer testing with Python
"""

import subprocess
import re
import json
from datetime import datetime

def test_zone_transfer(domain, nameserver=None):
    """Test zone transfer using dnswalk."""
    cmd = ["dnswalk", "-f", "-v"]
    if nameserver:
        cmd.append(f"@{nameserver}")
    cmd.append(f"{domain}.")
    
    result = subprocess.run(cmd, capture_output=True, text=True, timeout=60)
    
    records = []
    for line in result.stdout.splitlines():
        # Parse zone transfer records
        parts = line.split()
        if len(parts) >= 4 and not line.startswith(('Getting', 'AXFR', 'Received', 'Checking')):
            records.append({
                "name": parts[0],
                "type": parts[1] if len(parts) > 1 else "",
                "value": " ".join(parts[2:]) if len(parts) > 2 else ""
            })
    
    vuln = "Received" in result.stdout
    
    return {
        "domain": domain,
        "nameserver": nameserver,
        "vulnerable": vuln,
        "record_count": len(records),
        "records": records,
        "timestamp": datetime.now().isoformat()
    }

# Example usage
domains = ["example.com", "vulnerable-domain.com"]
results = []

for domain in domains:
    print(f"[*] Testing {domain}...")
    result = test_zone_transfer(domain)
    results.append(result)
    
    if result["vulnerable"]:
        print(f"  [!] VULNERABLE — {result['record_count']} records obtained")
        for rec in result["records"][:5]:
            print(f"      {rec['name']} {rec['type']} {rec['value']}")
    else:
        print(f"  [+] Secure — zone transfer denied")

# Save report
with open("zone_transfer_report.json", "w") as f:
    json.dump(results, f, indent=2)
print(f"\n[+] Report saved to zone_transfer_report.json")
```

---

## Chapter 5: Advanced Usage

### Testing All Nameservers Independently

```bash
#!/bin/bash
# test_all_ns.sh — Test zone transfer against each nameserver independently

DOMAIN="${1:?Usage: $0 <domain>}"

echo "=== Testing all nameservers for $DOMAIN ==="

# Get all nameservers
NS_LIST=$(dig NS "${DOMAIN}" +short)

for ns in $NS_LIST; do
    echo ""
    echo "--- Testing: $ns ---"
    ns_ip=$(dig A "${ns}" +short | head -1)
    
    if [ -z "$ns_ip" ]; then
        echo "  Could not resolve $ns to IP"
        continue
    fi
    
    echo "  IP: $ns_ip"
    
    # Test zone transfer
    RESULT=$(dnswalk -f "@${ns}" "${DOMAIN}." 2>&1)
    echo "$RESULT" | head -5
    
    # Check for records
    RECORD_COUNT=$(echo "$RESULT" | grep -c "^[a-zA-Z]")
    if [ "$RECORD_COUNT" -gt 0 ]; then
        echo "  [!] $RECORD_COUNT records obtained from $ns"
    fi
    
    # Additional DNS queries to the nameserver
    echo "  SOA: $(dig @${ns_ip} ${DOMAIN} SOA +short)"
    echo "  NS: $(dig @${ns_ip} ${DOMAIN} NS +short)"
done
```

### Automated Zone Transfer with Retry Logic

```bash
#!/bin/bash
# resilient_zone_transfer.sh — Zone transfer with retries and fallbacks

DOMAIN="${1:?Usage: $0 <domain>}"
MAX_RETRIES=3
RETRY_DELAY=5

echo "=== Resilient Zone Transfer: $DOMAIN ==="

# Get nameservers
NS_LIST=$(dig NS "${DOMAIN}" +short)

for ns in $NS_LIST; do
    ns_ip=$(dig A "${ns}" +short | head -1)
    [ -z "$ns_ip" ] && continue
    
    for attempt in $(seq 1 $MAX_RETRIES); do
        echo "[*] Attempt $attempt/$MAX_RETRIES for $ns ($ns_ip)..."
        
        RESULT=$(dnswalk -f "@${ns}" "${DOMAIN}." 2>&1)
        
        if echo "$RESULT" | grep -q "Received"; then
            echo "[!] Zone transfer succeeded from $ns!"
            echo "$RESULT" > "zone_${ns_ip}.txt"
            break
        elif echo "$RESULT" | grep -q "timed out"; then
            echo "  [!] Timeout, retrying in ${RETRY_DELAY}s..."
            sleep $RETRY_DELAY
        else
            echo "  [-] Zone transfer denied"
            break
        fi
    done
done
```

### Custom DNS Server Configuration

```bash
# Create a list of DNS servers to test against
cat > dns_servers.txt << 'EOF'
8.8.8.8
8.8.4.4
1.1.1.1
9.9.9.9
208.67.222.222
EOF

# Test zone transfer from each configured server
while read server; do
    echo "=== Testing via $server ==="
    dnswalk -f -v "@${server}" target.com. 2>&1 | tee "dnswalk_${server}.txt"
done < dns_servers.txt
```

### Integration with Logging Systems

```bash
#!/bin/bash
# dnswalk_syslog.sh — Send zone transfer results to syslog

DOMAIN="${1:?Usage: $0 <domain>}"

RESULT=$(dnswalk -f "${DOMAIN}." 2>&1)

if echo "$RESULT" | grep -q "Received"; then
    RECORD_COUNT=$(echo "$RESULT" | grep -c "^[a-zA-Z]")
    logger -p security.warning "DNSWALK: Zone transfer succeeded for ${DOMAIN} — ${RECORD_COUNT} records exposed"
    echo "[!] ALERT: Zone transfer successful for $DOMAIN"
else
    logger -p security.info "DNSWALK: Zone transfer denied for ${DOMAIN}"
    echo "[+] Zone transfer properly restricted for $DOMAIN"
fi
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: Comprehensive DNS Security Audit

**Objective:** Perform a full DNS security audit for a client's domain, including zone transfer testing, DNSSEC validation, and misconfiguration detection.

**Step 1: Discover All Nameservers**
```bash
echo "=== DNS Security Audit: target.com ==="
echo ""
echo "[1/6] Discovering nameservers..."
dig NS target.com +short | tee audit_ns.txt
```

Expected output:
```
ns1.target.com.
ns2.target.com.
ns3.cloudflare.com.
```

**Step 2: Test Zone Transfers Against All Nameservers**
```bash
echo ""
echo "[2/6] Testing zone transfers..."
while read ns; do
    echo "  Testing $ns..."
    dnswalk -f "@${ns}" target.com. 2>&1 | tee -a audit_axfr.txt
    echo ""
done < audit_ns.txt
```

**Step 3: Analyze Zone Transfer Results**
```bash
echo ""
echo "[3/6] Analyzing results..."
if grep -q "Received" audit_axfr.txt; then
    echo "[!] CRITICAL: Zone transfer vulnerability detected!"
    grep "Received" audit_axfr.txt
    RECORD_COUNT=$(grep -c "^[a-zA-Z]" audit_axfr.txt)
    echo "  Total records exposed: $RECORD_COUNT"
    
    # Extract sensitive records
    echo ""
    echo "  Internal hosts found:"
    grep -E "\.(internal|dev|staging|admin)" audit_axfr.txt || echo "    None"
    
    echo ""
    echo "  Mail servers:"
    grep "MX" audit_axfr.txt || echo "    None"
else
    echo "[+] Zone transfers properly restricted"
fi
```

**Step 4: Check DNSSEC**
```bash
echo ""
echo "[4/6] Checking DNSSEC..."
DNSKEY=$(dig DNSKEY target.com +dnssec +short)
DS=$(dig DS target.com +short)
if [ -n "$DNSKEY" ]; then
    echo "  [+] DNSSEC enabled"
    echo "  DNSKEY records: $(echo "$DNSKEY" | wc -l)"
else
    echo "  [-] DNSSEC NOT enabled — recommend enabling"
fi
```

**Step 5: Check for Common Misconfigurations**
```bash
echo ""
echo "[5/6] Checking for misconfigurations..."

# Check for missing glue records
echo "  Checking glue records..."
for ns in $(dig NS target.com +short); do
    ns_ip=$(dig A "$ns" +short | head -1)
    if [ -z "$ns_ip" ]; then
        echo "  [!] Missing glue record for $ns"
    else
        echo "  [+] Glue record OK: $ns -> $ns_ip"
    fi
done

# Check for CNAME loops
echo "  Checking for CNAME loops..."
dig CNAME target.com +short | grep -q "target.com" && echo "  [!] Possible CNAME loop" || echo "  [+] No CNAME loops"
```

**Step 6: Generate Report**
```bash
echo ""
echo "[6/6] Generating report..."
cat > audit_report.txt << EOF
DNS Security Audit Report: target.com
Date: $(date)
Auditor: $(whoami)

Summary:
- Nameservers found: $(wc -l < audit_ns.txt)
- Zone transfer: $([ -f audit_axfr.txt ] && grep -q "Received" audit_axfr.txt && echo "VULNERABLE" || echo "SECURE")
- DNSSEC: $([ -n "$DNSKEY" ] && echo "ENABLED" || echo "DISABLED")

Detailed findings in:
- audit_ns.txt
- audit_axfr.txt
EOF
echo "[+] Report saved to audit_report.txt"
```

### Scenario 2: CTF Challenge — "The Misconfigured DNS Server"

**Objective:** Exploit a DNS misconfiguration to find hidden flags in a CTF challenge.

**Step 1: Initial Reconnaissance**
```bash
# The challenge hints at a DNS issue
echo "[*] Step 1: Testing zone transfer..."
dnswalk -v ctf-target.com. 2>&1 | tee ctf_dns.txt
```

**Step 2: Analyze Results**
```bash
# If zone transfer succeeds
cat ctf_dns.txt | grep -E "(A|TXT|CNAME|MX)" | head -20
```

Sample output:
```
flag.ctf-target.com.    TXT     "dGhlIGZsYWcgaXMgZmxhZzEyMzQ1"
admin.ctf-target.com.   A       10.10.10.50
secret.ctf-target.com.  CNAME   flag.ctf-target.com.
```

**Step 3: Decode the Flag**
```bash
# Decode base64 TXT record
echo "dGhlIGZsYWcgaXMgZmxhZzEyMzQ1" | base64 -d
```

Output:
```
the flag is flag12345
```

**Step 4: Additional Enumeration**
```bash
# Check all subdomains for more flags
grep "\.ctf-target\.com\." ctf_dns.txt | awk '{print $1}' > ctf_hosts.txt
while read host; do
    echo "--- $host ---"
    dig "$host" +short
done < ctf_hosts.txt
```

### Scenario 3: Penetration Test — Pre-Engagement DNS Assessment

**Objective:** Assess DNS security before a penetration test engagement.

**Step 1: Map DNS Infrastructure**
```bash
# Discover all DNS infrastructure
echo "[*] Mapping DNS infrastructure..."
dig NS target.com +short > preeng_dns_ns.txt
dig MX target.com +short > preeng_dns_mx.txt

# Trace each nameserver's IP
while read ns; do
    echo "$ns -> $(dig A "$ns" +short | head -1)"
done < preeng_dns_ns.txt
```

**Step 2: Test Zone Transfers**
```bash
# Test zone transfers
echo "[*] Testing zone transfers..."
dnswalk -f target.com. 2>&1 > preeng_zone_transfer.txt

if grep -q "Received" preeng_zone_transfer.txt; then
    echo "[!] CRITICAL FINDING: Zone transfer allowed"
    # Extract all hostnames for further testing
    grep -E "^[a-zA-Z]" preeng_zone_transfer.txt | awk '{print $1}' | sort -u > preeng_hosts.txt
    echo "[+] $(wc -l < preeng_hosts.txt) hosts discovered via zone transfer"
else
    echo "[+] Zone transfers properly restricted"
fi
```

**Step 3: Identify Attack Surface**
```bash
# Analyze discovered hosts
echo "[*] Analyzing attack surface..."
if [ -f preeng_hosts.txt ]; then
    # Find internal/dev/staging hosts
    echo "Internal hosts:"
    grep -E "\.(internal|dev|staging|admin|test)" preeng_hosts.txt || echo "  None"
    
    # Find mail servers
    echo "Mail infrastructure:"
    grep -E "(mail|smtp|imap|pop)" preeng_hosts.txt || echo "  None"
    
    # Find VPN/remote access
    echo "Remote access:"
    grep -E "(vpn|remote|ssh|rdp)" preeng_hosts.txt || echo "  None"
fi
```

**Step 4: Generate Pre-Engagement Report**
```bash
cat > preengagement_dns_report.md << EOF
# Pre-Engagement DNS Assessment: target.com
**Date:** $(date)
**Assessor:** $(whoami)

## DNS Infrastructure
$(cat preeng_dns_ns.txt)

## Zone Transfer Test
$([ -q preeng_zone_transfer.txt ] && echo "VULNERABLE" || echo "SECURE")

## Attack Surface
$(wc -l < preeng_hosts.txt 2>/dev/null || echo "0") hosts discovered

## Recommendations
1. Restrict zone transfers to authorized servers only
2. Enable DNSSEC
3. Remove internal hostnames from public DNS
4. Implement DNS monitoring
EOF
echo "[+] Report generated: preengagement_dns_report.md"
```

---

## Chapter 7: Detection & Defense

### How Zone Transfer Attempts Are Detected

**DNS Server Logs:**
- Most DNS servers (BIND, Microsoft DNS, PowerDNS) log zone transfer attempts
- BIND logs: `named[pid]: client @0x... ip#port (domain): query (AXFR) denied`
- Windows DNS: Event ID 6528 — Zone transfer request denied

**Network Signatures:**
- TCP connection to port 53 (zone transfers use TCP, not UDP)
- AXFR query type in DNS packet
- Large response payloads (zone transfers return complete zone data)
- Sequential queries from same source IP

**IDS/IPS Rules:**
```
# Suricata rule for detecting zone transfer attempts
alert tcp any any -> any 53 (msg:"DNS Zone Transfer Attempt"; \
  content:"|00|"; offset:2; depth:1; \
  content:"|FC|"; offset:0; depth:2; \
  sid:1000002; rev:1;)

# Snort rule
alert udp any any -> any 53 (msg:"DNS IXFR/AXFR Query"; \
  content:"|00|"; offset:0; depth:2; \
  content:"|FC|"; offset:0; depth:2; \
  sid:1000003; rev:1;)
```

**Log Analysis:**
```bash
# Detect zone transfer attempts in BIND logs
grep "AXFR\|IXFR" /var/log/named/query.log | \
    awk '{print $1, $5, $7}' | sort | uniq -c | sort -rn

# Detect repeated AXFR attempts from same source
grep "AXFR" /var/log/named/query.log | \
    awk '{print $5}' | sort | uniq -c | sort -rn | head -10
```

### Blue Team Defensive Measures

**1. Restrict Zone Transfers:**

BIND configuration:
```
// named.conf
zone "target.com" {
    type master;
    file "target.com.zone";
    allow-transfer { 
        key "transfer-key";
        10.0.0.0/8;  // Only allow from internal network
    };
    also-notify { 10.0.0.2; };  // Notify secondary DNS
};
```

**2. Implement TSIG for Zone Transfers:**
```bash
# Generate TSIG key
tsig-keygen transfer-key > /etc/bind/transfer-key.conf

# Use key in zone configuration
# allow-transfer { key transfer-key; };
```

**3. Deploy DNSSEC:**
```bash
# Sign zone with DNSSEC
dnssec-keygen -a ECDSAP256SHA256 -n ZONE target.com
dnssec-keygen -a ECDSAP256SHA256 -n ZONE -f KSK target.com
dnssec-signzone -o target.com target.com.zone
```

**4. Enable DNS Query Rate Limiting:**
```bash
# BIND rate limiting options
options {
    rate-limit {
        responses-per-second 10;
        window 5;
        log-only no;
    };
};
```

**5. Monitor and Alert:**
```bash
#!/bin/bash
# dns_monitor.sh — Monitor for zone transfer attempts

LOG="/var/log/named/query.log"
ALERT_EMAIL="admin@company.com"

tail -f "$LOG" | while read line; do
    if echo "$line" | grep -q "AXFR\|IXFR"; then
        SOURCE=$(echo "$line" | awk '{print $5}')
        DOMAIN=$(echo "$line" | grep -oP '\(\K[^)]+')
        echo "ALERT: Zone transfer attempt from $SOURCE for $DOMAIN" | \
            mail -s "DNS Zone Transfer Alert" "$ALERT_EMAIL"
        logger -p security.crit "DNS Zone Transfer attempt from $SOURCE for $DOMAIN"
    fi
done
```

**6. Split-Horizon DNS:**
```
// Serve different records to internal vs external queries
view "internal" {
    match-clients { 10.0.0.0/8; };
    zone "target.com" {
        type master;
        file "target.com.internal.zone";  // Contains internal records
    };
};

view "external" {
    match-clients { any; };
    zone "target.com" {
        type master;
        file "target.com.external.zone";  // No internal records
    };
};
```

---

## Chapter 8: Troubleshooting

### Common Errors

#### "Zone transfer refused"

**Cause:** DNS server is properly configured to deny unauthorized transfers. This is the expected, secure response.

**Solutions:**
```bash
# This is actually GOOD — it means DNS is secure
# Try alternative enumeration methods:
dnsenum target.com          # Uses multiple enumeration techniques
dnsrecon -d target.com      # Comprehensive DNS recon
dig NS target.com +short    # Basic NS enumeration
```

#### "Domain must end with dot"

**Cause:** DNSwalk requires FQDN format with trailing dot.

**Solutions:**
```bash
# Wrong
dnswalk target.com

# Correct
dnswalk target.com.

# Automated fix
dnswalk "${DOMAIN%.}."
```

#### "Connection timed out"

**Cause:** DNS server is unreachable or blocking queries.

**Solutions:**
```bash
# Check DNS server accessibility
dig @ns1.target.com target.com

# Check firewall rules
sudo iptables -L -n | grep 53

# Try alternate DNS server
dnswalk @8.8.8.8 target.com.
```

#### "Can't connect to DNS server"

**Cause:** Network connectivity issue or DNS server is down.

**Solutions:**
```bash
# Verify network connectivity
ping target.com
traceroute target.com

# Check if DNS server is running
dig @ns1.target.com target.com +short

# Try different port
dig @ns1.target.com target.com -p 5353
```

#### "No nameservers found"

**Cause:** Domain doesn't exist or NS records are missing.

**Solutions:**
```bash
# Verify domain registration
whois target.com

# Check for NS records at parent
dig NS target.com +norecurse +norec

# Try manual specification
dnswalk @ns1.target.com. target.com.
```

#### Perl Module Errors

**Cause:** Missing Perl dependencies.

**Solutions:**
```bash
# Install required modules
sudo apt install libnet-dns-perl libnet-ip-perl

# Or via CPAN
sudo cpan Net::DNS Net::IP

# Verify installation
perl -e 'use Net::DNS; use Net::IP; print "OK\n"'
```

### Performance Optimization

```bash
# For large-scale testing, use parallel execution
#!/bin/bash
# parallel_zone_test.sh — Test multiple domains in parallel

MAX_PARALLEL=10
DOMAINS=$(cat domains.txt)

echo "$DOMAINS" | xargs -P $MAX_PARALLEL -I {} bash -c '
    DOMAIN="{}"
    RESULT=$(dnswalk -f "${DOMAIN}." 2>&1)
    if echo "$RESULT" | grep -q "Received"; then
        echo "[!] VULNERABLE: $DOMAIN"
    fi
'
```

### FAQ

**Q: Why does DNSwalk need a trailing dot?**
A: In DNS protocol, a trailing dot signifies a fully qualified domain name (FQDN). Without it, the domain might be treated as relative to the current search domain, causing incorrect queries.

**Q: Is a successful zone transfer always a vulnerability?**
A: In most cases, yes — zone transfers should be restricted. However, some public DNS zones intentionally allow transfers for secondary DNS providers.

**Q: Can DNSwalk test for DNSSEC?**
A: Not directly. Use `dig +dnssec` or `delv` for DNSSEC validation.

**Q: What's the difference between DNSwalk and DNSenum?**
A: DNSwalk focuses primarily on zone transfer testing. DNSenum performs zone transfers AND additional enumeration techniques (brute-force, Google scraping, etc.).

**Q: Does DNSwalk work with split-horizon DNS?**
A: It will only test the view accessible from your network position. Internal views may differ from external views.

**Q: How many records can a zone transfer return?**
A: There's no hard limit, but large zones can contain hundreds of thousands of records. Some DNS servers limit zone transfer responses to prevent resource exhaustion.

---

## Resources

- [DNSwalk Man Page](https://linux.die.net/man/1/dnswalk)
- [DNSwalk GitHub Repository](https://github.com/fgarber/dnswalk)
- [RFC 5936 — DNS Zone Transfer Protocol (AXFR)](https://datatracker.ietf.org/doc/html/rfc5936)
- [RFC 1995 — Incremental Zone Transfer (IXFR)](https://datatracker.ietf.org/doc/html/rfc1995)
- [DNS Security Best Practices — ISC](https://kb.isc.org/docs/aa-00882)
- [BIND 9 Security Considerations](https://kb.isc.org/docs/aa-00632)
- [OWASP DNS Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/DNS_Security_Cheat_Sheet.html)
- [NIST SP 800-81 — Secure DNS Deployment](https://csrc.nist.gov/publications/detail/sp/800-81/final)
- [Kali Linux DNSwalk Package](https://www.kali.org/tools/dnswalk/)
