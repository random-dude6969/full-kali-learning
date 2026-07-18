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

## Resources

- **GitHub:** https://github.com/P0cL4bs/wifipumpkin3
- **Documentation:** https://wifipumpkin3.github.io
- **Kali Documentation:** https://www.kali.org/tools/wifipumpkin3/
- **Discord:** https://discord.gg/jywYskR
- **Hostapd:** https://w1.fi/hostapd/
- **dnsmasq:** http://www.thekelleys.org.uk/dnsmasq/doc.html
