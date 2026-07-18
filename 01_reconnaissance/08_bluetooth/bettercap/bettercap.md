---
tool_name: "bettercap"
display_name: "BetterCap"
mitre_tactic: "reconnaissance"
mitre_tactic_id: "TA0043"
mitre_subcategory: "bluetooth"
install_command: "sudo apt install bettercap"
version: "2.40"
github_repo: "https://github.com/bettercap/bettercap"
kali_tools_page: "https://www.kali.org/tools/bettercap/"
difficulty_level: "intermediate-to-advanced"
requires_root: true
gui_available: false
tags: ["bluetooth", "wifi", "network", "mitm", "sniffing", "ble", "arp-spoofing", "dns-spoofing", "reconnaissance"]
related_tools: ["aircrack-ng", "bluelog", "spooftooph", "wireshark", "nmap"]
---

# BetterCap — Network Attack & Monitoring Framework

## What This Tool Does (in 30 seconds)

BetterCap is a powerful, flexible framework for network attack and monitoring. It supports WiFi, Bluetooth, Ethernet, and CAN bus reconnaissance, with built-in modules for MITM attacks, packet sniffing, and network discovery. Written in Go, it provides a modern, modular architecture with both CLI and interactive web UI.

---

## Chapter 1: Introduction & Overview

### What is BetterCap?

BetterCap is a modern, modular network attack and monitoring framework. It provides:
- **WiFi reconnaissance** — Scan and monitor wireless networks
- **Bluetooth scanning** — Discover Bluetooth and BLE devices
- **Network sniffing** — Capture and analyze traffic
- **MITM attacks** — Man-in-the-middle capabilities (ARP spoofing, DNS spoofing, HTTPS proxy)
- **Packet manipulation** — Modify network traffic in real-time
- **HID attacks** — Keyboard/mouse injection via Bluetooth

### Key Features

- Modular architecture with hot-loadable modules
- Interactive JavaScript-based scripting engine (caplets)
- Built-in web UI for real-time monitoring
- Support for multiple protocols (802.11, Ethernet, BLE, CAN)
- Real-time network mapping and device tracking
- Pcap file generation for offline analysis
- Event-driven architecture for automation
- Full duplex ARP spoofing support

### History & Background

- **Created:** 2013 by Simone Margaritelli (evilsocket)
- **Rewritten:** 2018 in Go (v2.0)
- **License:** GPL-3.0
- **Purpose:** Network penetration testing, security auditing, reconnaissance

### When to Use BetterCap

- **Network reconnaissance** — Discover devices and services on wired/wireless networks
- **WiFi auditing** — Test wireless network security, capture handshakes
- **Bluetooth scanning** — Find nearby Bluetooth and BLE devices
- **Penetration testing** — Network-level MITM attacks, credential harvesting
- **Security assessments** — Evaluate network defenses, test segmentation
- **IoT security** — Discover and enumerate IoT devices on networks

### When NOT to Use This Tool

- Without explicit written authorization from the network owner
- On production networks without proper planning and change management
- For malicious purposes or unauthorized access to systems/data
- In environments where interference could cause safety issues

### Legal and Ethical Considerations

- **Always obtain written permission** before attacking networks you don't own
- **Understand your scope** — know exactly what you're authorized to test
- **Be careful with timing** — ARP spoofing and DNS spoofing can disrupt services
- **Document everything** — keep records of your authorization and activities
- **Follow responsible disclosure** — report vulnerabilities properly
- **Local laws vary** — Bluetooth interception laws differ by jurisdiction

### Key Concepts

- **Caplet:** A BetterCap scripting file (JavaScript-like syntax) for automating attacks
- **Module:** A self-contained component (arp.spoof, wifi.recon, ble.recon, etc.)
- **BLE:** Bluetooth Low Energy — the modern, low-power Bluetooth protocol
- **ARP Spoofing:** Sending fake ARP replies to redirect traffic through your machine
- **DNS Spoofing:** Responding to DNS queries with malicious IP addresses
- **Pcap:** Packet capture file format for offline analysis

---

## Chapter 2: Installation & Setup

### System Requirements

- **Minimum RAM:** 1 GB (2 GB recommended)
- **Disk Space:** 100 MB for base install
- **Network:** Any network interface (wired or wireless)
- **Privileges:** Root required for all attack modules
- **Optional:** Wireless adapter for WiFi/BLE features, multiple adapters for advanced attacks

### Installation Methods

#### Method 1: APT (Recommended)

```bash
sudo apt update
sudo apt install bettercap
```

#### Method 2: From Source

```bash
# Install dependencies
sudo apt install libnetfilter-queue-dev libpcap-dev

# Clone repository
git clone https://github.com/bettercap/bettercap.git
cd bettercap

# Build
make
sudo make install
```

#### Method 3: Go Install

```bash
# Requires Go 1.18+
go install github.com/bettercap/bettercap/v2@latest
```

#### Method 4: Docker

```bash
docker run -it --net=host bettercap/bettercap
```

### Configuration

BetterCap uses several configuration files and directories:
- `/usr/share/bettercap/` — Default caplets and resources
- `~/.bettercap/` — User configuration
- Default caplet path: `/usr/share/bettercap/caplets/`

### Verification

```bash
sudo bettercap -version
# Output: bettercap v2.40 ( ... )
```

### Interface Requirements

```bash
# List available interfaces
ip link show

# Check wireless capabilities
iwconfig

# Check Bluetooth adapter
hciconfig -a

# For BLE, ensure Bluetooth stack is running
sudo systemctl start bluetooth
```

---

## Chapter 3: Basic Usage

### First Run

```bash
# Start interactive session (auto-detects interface)
sudo bettercap

# Start with specific interface
sudo bettercap -iface wlan0

# Start with Bluetooth interface
sudo bettercap -iface hci0
```

### Essential Commands

```bash
# Network discovery
sudo bettercap -eval "net.probe on; net.show"

# WiFi reconnaissance
sudo bettercap -eval "wifi.recon on; wifi.show"

# Bluetooth Low Energy scanning
sudo bettercap -eval "ble.recon on; ble.show"

# ARP spoofing
sudo bettercap -eval "set arp.spoof.targets 192.168.1.100; arp.spoof on"

# DNS spoofing
sudo bettercap -eval "set dns.spoof.domains example.com; set dns.spoof.address 10.0.0.1; dns.spoof on"

# Packet sniffing
sudo bettercap -eval "net.sniff on"
```

### Module Reference

| Module | Description | Command |
|--------|-------------|---------|
| `net.probe` | Network discovery via ARP/ICMP | `net.probe on` |
| `net.sniff` | Packet capture and analysis | `net.sniff on` |
| `arp.spoof` | ARP spoofing for MITM | `arp.spoof on` |
| `dns.spoof` | DNS spoofing | `dns.spoof on` |
| `wifi.recon` | WiFi scanning and monitoring | `wifi.recon on` |
| `wifi.deauth` | WiFi deauthentication attacks | `wifi.deauth on` |
| `ble.recon` | Bluetooth Low Energy scanning | `ble.recon on` |
| `ble.replay` | BLE replay attacks | `ble.replay on` |
| `hid.keyboard` | HID keyboard injection | `hid.keyboard on` |

### Complete Flags Reference

#### General Options

| Flag | Description | Example |
|------|-------------|---------|
| `-iface` | Network interface | `-iface wlan0` |
| `-eval` | Execute caplet commands | `-eval "net.probe on"` |
| `-caplet` | Run caplet file | `-caplet recon.cap` |
| `-server` | Web UI server | `-server http://127.0.0.1:8080` |
| `-debug` | Enable debug mode | `-debug` |
| `-no-color` | Disable colors | `-no-color` |
| `-silent` | Suppress banner | `-silent` |

#### Module Options

| Module | Option | Description | Example |
|--------|--------|-------------|---------|
| `arp.spoof` | `targets` | Target IPs | `set arp.spoof.targets 192.168.1.0/24` |
| `arp.spoof` | `fullduplex` | Bidirectional spoof | `set arp.spoof.fullduplex true` |
| `dns.spoof` | `domains` | Target domains | `set dns.spoof.domains *.example.com` |
| `dns.spoof` | `address` | Spoofed IP | `set dns.spoof.address 10.0.0.1` |
| `wifi.recon` | `channel` | WiFi channel | `set wifi.recon.channel 6` |
| `wifi.deauth` | `ap` | Target AP BSSID | `set wifi.deauth.ap AA:BB:CC:DD:EE:FF` |
| `ble.recon` | `uuid` | Filter by UUID | `set ble.recon.uuid 0x180D` |

---

## Chapter 4: Intermediate Usage

### Caplet Scripting

Caplets are BetterCap's automation language — JavaScript-like syntax for sequencing operations.

```javascript
// recon.cap - Network reconnaissance caplet
net.probe on
sleep 5
net.show
net.probe off
```

```javascript
// mitm_attack.cap - ARP spoofing with sniffing
set arp.spoof.targets 192.168.1.100
set arp.spoof.fullduplex true
set net.sniff.output captured.pcap
arp.spoof on
net.sniff on
```

```bash
# Run caplet
sudo bettercap -caplet recon.cap
```

### Python Automation

```python
#!/usr/bin/env python3
"""Automate BetterCap via subprocess"""

import subprocess
import json

def run_bettercap(command, interface=None):
    """Run BetterCap command and return output"""
    cmd = ['sudo', 'bettercap', '-eval', command]
    if interface:
        cmd.extend(['-iface', interface])
    result = subprocess.run(cmd, capture_output=True, text=True, timeout=60)
    return result.stdout

# Network discovery
output = run_bettercap("net.probe on; sleep 10; net.show; net.probe off")
print(output)

# BLE scanning
ble_output = run_bettercap("ble.recon on; sleep 30; ble.show; ble.recon off")
print(ble_output)
```

### Integration with Other Tools

#### BetterCap + Nmap

```bash
# Step 1: Discover hosts with BetterCap
sudo bettercap -eval "net.probe on; sleep 15; net.probe off" 2>/dev/null

# Step 2: Export discovered hosts
sudo bettercap -eval "net.probe on; sleep 15; events.dump hosts.json; net.probe off"

# Step 3: Port scan discovered hosts
nmap -iL discovered_hosts.txt -T4 -F
```

#### BetterCap + Wireshark

```bash
# Step 1: Capture traffic with BetterCap
sudo bettercap -eval "net.sniff on; pcap.save traffic.pcap"

# Step 2: Analyze in Wireshark
wireshark traffic.pcap
```

#### BetterCap + Aircrack-ng

```bash
# Step 1: Capture WiFi handshake
sudo bettercap -iface wlan0mon -eval "
wifi.recon on
set wifi.handshakes.file handshakes.pcap
"

# Step 2: Crack captured handshake
aircrack-ng -w wordlist.txt handshakes.pcap
```

---

## Chapter 5: Advanced Usage

### Custom Caplets

```javascript
// advanced_recon.cap - Comprehensive network reconnaissance

// Set parameters
set net.probe.timeout 10000
set arp.spoof.fullduplex true

// Start discovery
net.probe on
sleep 10

// Save results
events.dump net.probe.results.json

// Start ARP spoofing
set arp.spoof.targets 192.168.1.0/24
arp.spoof on

// Start sniffing
set net.sniff.output sniffed.pcap
net.sniff on

// Log credentials
set net.sniff.verbose true
```

### Advanced MITM Attacks

```bash
# Full duplex ARP spoofing with DNS spoofing
sudo bettercap -eval "
set arp.spoof.fullduplex true
set arp.spoof.targets 192.168.1.100
arp.spoof on

set dns.spoof.domains *.google.com
set dns.spoof.address 10.0.0.100
dns.spoof on

net.sniff on
"
```

### BLE Advanced Operations

```bash
# BLE sniffing with UUID filtering
sudo bettercap -eval "
ble.recon on
set ble.recon.uuid 0x180D
sleep 60
ble.show
ble.recon off
"

# BLE HID keyboard injection (requires authorization)
sudo bettercap -eval "
hid.keyboard on
hid.keyboard.type 'Hello World'
"
```

### Scaling with Multi-Interface

```bash
#!/bin/bash
# multi_interface.sh - Scan multiple interfaces simultaneously

for iface in wlan0 wlan1; do
    echo "[*] Scanning: $iface"
    sudo bettercap -iface "$iface" -eval "net.probe on; sleep 10; net.probe off" > "hosts_$iface.txt" &
done

wait
echo "[*] All scans complete"
cat hosts_*.txt > all_hosts.txt
```

### Event-Driven Automation

```javascript
// event_handler.cap - React to events
on net.probe.new host/host,arp_addr;
    log "New host: " + host.host + " (" + host.arp_addr + ")";
    # Add to watchlist
    echo host.host >> /tmp/watchlist.txt
end

net.probe on
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: Corporate Network Audit

```bash
# Step 1: Discover all devices
sudo bettercap -eval "net.probe on; sleep 30; net.show; net.probe off"

# Step 2: Identify services
nmap -sV -iL discovered_hosts.txt -T4

# Step 3: Test for vulnerabilities
nmap --script vuln -iL discovered_hosts.txt

# Step 4: Document findings
sudo bettercap -eval "net.probe on; sleep 30; events.dump audit_results.json; net.probe off"
```

### Scenario 2: WiFi Security Assessment

```bash
# Step 1: Monitor wireless networks
sudo bettercap -iface wlan0 -eval "wifi.recon on; wifi.show"

# Step 2: Capture handshakes
sudo bettercap -iface wlan0 -eval "
wifi.recon on
set wifi.handshakes.file handshakes.pcap
"

# Step 3: Deauth and capture
sudo bettercap -iface wlan0 -eval "
wifi.recon on
set wifi.deauth.ap TARGET_BSSID
wifi.deauth on
"

# Step 4: Test captured handshakes
aircrack-ng -w wordlist.txt handshakes.pcap
```

### Scenario 3: Bluetooth Device Discovery

```bash
# Step 1: Scan for BLE devices
sudo bettercap -eval "
ble.recon on
sleep 60
ble.show
"

# Step 2: Identify devices by UUID
sudo bettercap -eval "
ble.recon on
set ble.recon.uuid 0x180F
sleep 30
ble.show
"
```

### Scenario 4: IoT Device Enumeration

```bash
# Step 1: Network scan for IoT devices
sudo bettercap -eval "
net.probe on
sleep 30
net.show
net.probe off
"

# Step 2: Fingerprint devices
sudo bettercap -eval "
net.sniff on
set net.sniff.verbose true
" -eval "net.probe on; sleep 30; net.probe off"
```

---

## Chapter 7: Detection & Defense

### Detection Methods

BetterCap generates distinctive network patterns that can be detected:

1. **ARP requests** — Gratuitous ARP replies for spoofing (multiple IPs from one MAC)
2. **WiFi probe requests** — Active scanning probe frames
3. **Bluetooth LE scans** — BLE discovery packets with specific patterns
4. **Network scans** — ARP/ICMP discovery probes on new subnets
5. **DNS responses** — Fake DNS replies with incorrect IP mappings

### Mitigation Strategies

#### 1. ARP Spoofing Detection

```bash
# Install ARP monitoring
sudo apt install arpon

# Enable ARP protection (ARPON)
sudo arpon -g

# Manual ARP monitoring
arp-scan --localnet | sort

# Detect duplicate MACs
arp -a | awk '{print $NF}' | sort | uniq -c | sort -rn | head
```

#### 2. WiFi Security

```bash
# Enable WPA3 (if supported)
# Use strong, unique passwords
# Enable wireless IDS (WIDS)

# Monitor for deauth attacks
sudo airodump-ng wlan0mon --output-format pcap
```

#### 3. Bluetooth Security

```bash
# Make devices non-discoverable
bluetoothctl discoverable off

# Disable Bluetooth when not in use
bluetoothctl power off

# Use Bluetooth 5.0 with LE Secure Connections
# Enable MITM protection
btmgmt ssp on
```

#### 4. Network Monitoring

```bash
# Monitor for ARP anomalies
tcpdump -i eth0 arp -n | awk '{print $NF}' | sort | uniq -c | sort -rn

# Monitor for WiFi probe requests
tcpdump -i wlan0 -s 256 type mgt subtype probe-req

# Monitor for BLE scanning
btmon -t | grep -i "scan\|inquiry"
```

#### 5. Network Segmentation

```bash
# VLAN separation for IoT devices
# Private VLANs for untrusted devices
# Network access control (NAC)
# 802.1X authentication
```

---

## Chapter 8: Troubleshooting & Resources

### Common Errors

#### Error: "Permission denied"

**Cause:** BetterCap requires root privileges for all attack modules.

**Solution:**
```bash
sudo bettercap -iface wlan0
```

#### Error: "No such interface"

**Cause:** Network interface not found or not up.

**Solution:**
```bash
# List interfaces
ip link show

# Check wireless
iwconfig

# Bring interface up
sudo ip link set wlan0 up
```

#### Error: "Module not found"

**Cause:** Required module not loaded or not compiled.

**Solution:**
```bash
# List available modules in interactive mode
help

# Load specific module
net.probe on

# Check module status
net.probe.status
```

#### Error: "No Bluetooth adapter"

**Cause:** Bluetooth adapter not detected or drivers missing.

**Solution:**
```bash
# Check Bluetooth adapter
hciconfig -a

# Install Bluetooth stack
sudo apt install bluez

# Restart Bluetooth service
sudo systemctl restart bluetooth
```

#### Error: "pcap.save failed"

**Cause:** Insufficient disk space or permissions.

**Solution:**
```bash
# Check disk space
df -h

# Check directory permissions
ls -la /path/to/output/

# Use writable directory
set net.sniff.output /tmp/capture.pcap
```

### Performance Optimization

```bash
# Limit ARP spoofing to specific targets
set arp.spoof.targets 192.168.1.100-192.168.1.110

# Reduce sniffing overhead
set net.sniff.verbose false

# Use packet filtering
set net.sniff.filter "tcp port 80"
```

### FAQ

#### Q: BetterCap can't find my WiFi adapter
```bash
# Check if adapter supports monitor mode
iw list | grep -A 5 "Supported interface modes"

# Put adapter in monitor mode
sudo ip link set wlan0 down
sudo iw wlan0 set monitor control
sudo ip link set wlan0 up

# Verify
iwconfig wlan0
```

#### Q: ARP spoofing not working
```bash
# Check IP forwarding is enabled
cat /proc/sys/net/ipv4/ip_forward
# Should be 1, if not:
sudo sysctl -w net.ipv4.ip_forward=1

# Verify target is reachable
ping 192.168.1.100

# Check ARP table
arp -a
```

#### Q: BLE scanning finds no devices
```bash
# Ensure Bluetooth is enabled
sudo systemctl start bluetooth
sudo hciconfig hci0 up

# Check if adapter supports BLE
hciconfig hci0 features | grep -i "low energy"

# Scan with increased duration
ble.recon on
sleep 60
ble.show
```

#### Q: How to capture credentials with BetterCap?
```bash
# Enable DNS spoofing for credential capture
set dns.spoof.domains *.target.com
set dns.spoof.address 10.0.0.100
dns.spoof on

# Start HTTPS proxy for SSL stripping
set net.sniff.ssl true
net.sniff on

# Note: Only use with authorization on networks you own
```

#### Q: BetterCap crashes during long scans
```bash
# Increase memory limits
ulimit -v unlimited

# Run with reduced modules
sudo bettercap -eval "net.probe on; sleep 300; net.probe off"

# Check system resources
free -h
top -bn1 | head -20
```

### Tips and Tricks

```bash
# Save session state
events.dump session_state.json

# Load previous session
events.load session_state.json

# Quick network map
sudo bettercap -eval "net.probe on; sleep 10; net.show; net.probe off; events.dump network_map.json"

# Monitor specific protocol
sudo bettercap -eval "net.sniff on; set net.sniff.filter 'tcp port 443'"

# Export to pcap for Wireshark
sudo bettercap -eval "net.sniff on; set net.sniff.output capture.pcap"
```

### Resources

- [BetterCap GitHub](https://github.com/bettercap/bettercap)
- [BetterCap Documentation](https://www.bettercap.org/)
- [BetterCap Modules](https://www.bettercap.org/modules/)
- [WiFi Hacking Guide](https://www.wi-fi.org/)
- [Bluetooth Security](https://www.bluetooth.com/learn-about-bluetooth/bluetooth-basics/)
- [Kali Linux Bluetooth Tools](https://www.kali.org/tools/)

---

## Resources

| Resource | URL |
|----------|-----|
| Official Repository | https://github.com/bettercap/bettercap |
| Documentation | https://www.bettercap.org/ |
| Kali Tools Page | https://www.kali.org/tools/bettercap/ |
| Module Reference | https://www.bettercap.org/modules/ |
| Bluetooth SIG | https://www.bluetooth.com/ |
| MITRE ATT&CK | https://attack.mitre.org/techniques/T1557/ |
