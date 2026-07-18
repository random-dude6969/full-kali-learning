---
title: "voiphopper Cheatsheet"
description: "Quick reference for voiphopper VoIP VLAN hopping tool"
mitre_tactic: discovery
mitre_tactic_id: TA0007
mitre_subcategory: voip
categories:
  - cheatsheet
  - voip
  - vlan-hopping
tags:
  - voiphopper
  - vlan
  - cdp
  - lldp
  - quick-reference
---

# voiphopper Cheatsheet

## Quick Start

```bash
# Discover voice VLAN (CDP)
sudo voiphopper -i eth0 -c

# Discover voice VLAN (LLDP)
sudo voiphopper -i eth0 -l

# Try all protocols
sudo voiphopper -i eth0 -a

# Hop to specific VLAN
sudo voiphopper -i eth0 -v 100
```

## Common Flags

### Interface

| Flag | Description | Example |
|------|-------------|---------|
| `-i` | Network interface | `-i eth0` |
| `-v` | Target VLAN | `-v 100` |

### Protocol

| Flag | Description |
|------|-------------|
| `-c` | CDP mode |
| `-l` | LLDP mode |
| `-a` | All protocols |

### Timing

| Flag | Description | Example |
|------|-------------|---------|
| `-t` | Timeout (seconds) | `-t 60` |
| `-r` | Retry count | `-r 3` |

### Control

| Flag | Description |
|------|-------------|
| `-b` | Bypass mode |
| `-q` | Quiet mode |
| `-d` | Daemon mode |

### Output

| Flag | Description |
|------|-------------|
| `-o` | Output file |
| `-w` | PCAP capture |

## Examples

### VLAN Discovery

```bash
# CDP discovery
sudo voiphopper -i eth0 -c

# LLDP discovery
sudo voiphopper -i eth0 -l

# All protocols
sudo voiphopper -i eth0 -a

# With timeout
sudo voiphopper -i eth0 -t 120
```

### VLAN Hopping

```bash
# Hop to VLAN
sudo voiphopper -i eth0 -v 100

# Auto-discover and hop
sudo voiphopper -i eth0 -a

# Bypass mode
sudo voiphopper -i eth0 -b
```

### Output and Capture

```bash
# Save results
sudo voiphopper -i eth0 -o results.txt

# Capture traffic
sudo voiphopper -i eth0 -w voice.pcap

# Verbose
sudo voiphopper -i eth0 -v
```

### Advanced

```bash
# Daemon mode
sudo voiphopper -i eth0 -d

# Retry discovery
sudo voiphopper -i eth0 -r 5

# Quiet mode
sudo voiphopper -i eth0 -q
```

## Chaining

### With SIP Tools

```bash
# voiphopper → svmap
sudo voiphopper -i eth0 -v 100
svmap 192.168.100.0/24

# voiphopper → svwar
sudo voiphopper -i eth0 -v 100
svwar -e 1000-9999 192.168.100.100

# voiphopper → svcrack
sudo voiphopper -i eth0 -v 100
svcrack -u 1000 -d passwords.txt 192.168.100.100
```

### With Network Tools

```bash
# voiphopper + tcpdump
sudo voiphopper -i eth0 -v 100 &
sudo tcpdump -i eth0 -w voice.pcap &

# voiphopper + nmap
sudo voiphopper -i eth0 -v 100
nmap -sU -p 5060 192.168.100.0/24

# voiphopper + arp-scan
sudo voiphopper -i eth0 -v 100
arp-scan -l -I eth0 192.168.100.0/24
```

### Full Recon Script

```bash
#!/bin/bash
# Voice VLAN reconnaissance
sudo voiphopper -i eth0 -a -o discovery.txt
VLAN=$(grep -oP 'VLAN\s+\K\d+' discovery.txt | head -1)
sudo voiphopper -i eth0 -v $VLAN
svmap 192.168.$VLAN.0/24
svwar -e 1000-9999 192.168.$VLAN.100
```

## Troubleshooting

### No Voice VLAN Found

```bash
# Check interface
ip link show eth0

# Monitor CDP/LLDP
tcpdump -i eth0 -n ether proto 0x2000 or ether proto 0x88CC

# Try all protocols
sudo voiphopper -i eth0 -a

# Increase timeout
sudo voiphopper -i eth0 -t 120
```

### VLAN Hopping Fails

```bash
# Check DTP traffic
tcpdump -i eth0 -n ether proto 0x2004

# Try different VLAN
sudo voiphopper -i eth0 -v 100

# Verify switch allows DTP
# (Requires switch access)
```

### No Traffic After Hop

```bash
# Verify VLAN interface
ip link show eth0.100

# Check routing
ip route show

# Manual IP config
sudo ip addr add 192.168.100.10/24 dev eth0.100
sudo ip link set eth0.100 up

# Request DHCP
dhclient eth0.100
```

### Permission Denied

```bash
# Run as root
sudo voiphopper -i eth0

# Check capabilities
getcap /usr/bin/voiphopper

# Add capabilities
sudo setcap cap_net_raw,cap_net_admin+eip /usr/bin/voiphopper
```

### Debug Commands

```bash
# Verbose
sudo voiphopper -i eth0 -v

# Packet capture
sudo voiphopper -i eth0 -w debug.pcap

# Interface details
ip -d link show eth0

# VLAN info
bridge vlan show
```

### Cleanup

```bash
# Remove VLAN interface
sudo ip link delete eth0.100

# Reset interface
sudo ip link set eth0 down
sudo ip link set eth0 up

# Restore DHCP
sudo dhclient eth0
```

---

**Last Updated**: 2026
**Tool Version**: voiphopper 0.4
**Category**: VoIP VLAN Hopping
