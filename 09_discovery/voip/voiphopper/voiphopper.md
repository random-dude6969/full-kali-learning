---
title: "voiphopper - VoIP VLAN Hopping Tool"
description: "VLAN hopping tool for discovering and joining voice VLANs via CDP, LLDP, and DTP protocols"
mitre_tactic: discovery
mitre_tactic_id: TA0007
mitre_subcategory: voip
categories:
  - discovery
  - voip
  - network
  - vlan-hopping
tags:
  - voip
  - vlan
  - cdp
  - lldp
  - dtp
  - network-discovery
install_command: "sudo apt install voiphopper"
version: "0.4"
github_repo: "https://github.com/sec51/voiphopper"
kali_tools_page: "https://tools.kali.org/information-gathering-tools/voiphopper"
difficulty_level: intermediate
requires_root: true
gui_available: false
related_tools:
  - yersinia
  - scapy
  - dsniff
  - ngrep
  - tcpdump
---

# voiphopper - VoIP VLAN Hopping Tool

voiphopper is a specialized VLAN hopping tool designed to discover and automatically join voice VLANs. It leverages CDP (Cisco Discovery Protocol), LLDP (Link Layer Discovery Protocol), and DTP (Dynamic Trunking Protocol) to identify and access VLANs typically reserved for VoIP traffic.

## 1. Introduction

VoIP devices often operate on dedicated voice VLANs separate from data traffic. These VLANs may contain sensitive voice communications, management interfaces, and configuration data. voiphopper enables security professionals to discover and access these isolated network segments during authorized assessments.

The tool automates the process of:
1. **Discovery**: Sniffing CDP/LLDP packets to identify voice VLANs
2. **VLAN Hopping**: Using DTP to negotiate trunk port access
3. **Auto-Joining**: Automatically switching to discovered voice VLANs
4. **Bypass**: Circumventing VLAN segmentation controls

### Use Cases

| Scenario | Description |
|----------|-------------|
| Network Assessment | Discover voice VLAN segmentation |
| Penetration Testing | Access isolated VoIP networks |
| Security Audit | Validate VLAN security controls |
| Forensic Analysis | Investigate voice network traffic |
| Compliance Testing | Verify network segmentation |

### Protocol Support

| Protocol | Description | Usage |
|----------|-------------|-------|
| CDP | Cisco Discovery Protocol | VLAN discovery |
| LLDP | Link Layer Discovery Protocol | VLAN discovery |
| DTP | Dynamic Trunking Protocol | VLAN hopping |
| 802.1Q | VLAN tagging | Traffic manipulation |

## 2. Installation

### Kali Linux

```bash
sudo apt update
sudo apt install voiphopper
```

### From Source

```bash
git clone https://github.com/sec51/voiphopper.git
cd voiphopper
make
sudo ./voiphopper
```

### Verify Installation

```bash
voiphopper --version
which voiphopper
```

### Dependencies

| Package | Purpose |
|---------|---------|
| libpcap | Packet capture |
| libnet | Packet construction |
| libcdp | CDP protocol |

### Package Details

| Property | Value |
|----------|-------|
| Package Name | voiphopper |
| Default Install | No |
| Dependencies | libpcap-dev, libnet1-dev |
| Architecture | amd64, i386 |
| License | GPLv2 |
| Requires Root | Yes |

## 3. Core Concepts

### Voice VLAN Model

```
┌─────────────────────────────────────────────────────┐
│                  Network Architecture                 │
├─────────────────────────────────────────────────────┤
│                                                      │
│  ┌──────────┐                                       │
│  │   PBX   │                                       │
│  │  Server  │                                       │
│  └────┬─────┘                                       │
│       │                                              │
│  ┌────┴────┐                                        │
│  │  Voice  │  VLAN 100                              │
│  │  VLAN   │  (192.168.100.0/24)                   │
│  └────┬────┘                                        │
│       │                                              │
│  ┌────┴────────────────────────────┐                │
│  │         Access Switch           │                │
│  │   ┌──────┬──────┬──────┐       │                │
│  │   │Port 1│Port 2│Port 3│       │                │
│  │   │VLAN  │VLAN  │VLAN  │       │                │
│  │   │  1   │ 100  │  1   │       │                │
│  │   └──┬───┴──┬───┴──┬───┘       │                │
│  │      │      │      │           │                │
│  └──────┼──────┼──────┼───────────┘                │
│         │      │      │                             │
│    ┌────┴┐  ┌──┴──┐  ┌┴────┐                       │
│    │Data │  │Phone│  │Data │                       │
│    │ PC  │  │     │  │ PC  │                       │
│    └─────┘  └─────┘  └─────┘                       │
│                                                      │
│  voiphopper targets Voice VLAN (100)                │
│                                                      │
└─────────────────────────────────────────────────────┘
```

### Discovery Methods

#### CDP (Cisco Discovery Protocol)

```
┌─────────────────────────────────────────────────────┐
│                CDP Packet Structure                   │
├─────────────────────────────────────────────────────┤
│  ┌──────────────────────────────────────────────┐   │
│  │  Destination MAC: 01:00:0C:CC:CC:CC         │   │
│  │  Source MAC: [Switch MAC]                     │   │
│  │  EtherType: 0x2000 (CDP)                     │   │
│  ├──────────────────────────────────────────────┤   │
│  │  TLV: Device ID                              │   │
│  │  TLV: Platform (e.g., Cisco IOS)             │   │
│  │  TLV: Addresses                              │   │
│  │  TLV: Port ID (e.g., GigabitEthernet0/1)    │   │
│  │  TLV: VLAN ID (Voice VLAN)                   │   │
│  │  TLV: Native VLAN                            │   │
│  └──────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────┘
```

#### LLDP (Link Layer Discovery Protocol)

```
┌─────────────────────────────────────────────────────┐
│                LLDP Packet Structure                  │
├─────────────────────────────────────────────────────┤
│  ┌──────────────────────────────────────────────┐   │
│  │  Destination MAC: 01:80:C2:00:00:0E         │   │
│  │  Source MAC: [Switch MAC]                     │   │
│  │  EtherType: 0x88CC (LLDP)                    │   │
│  ├──────────────────────────────────────────────┤   │
│  │  TLV: Chassis ID                             │   │
│  │  TLV: Port ID                                │   │
│  │  TLV: TTL                                    │   │
│  │  TLV: System Name                            │   │
│  │  TLV: VLAN Name (Voice VLAN)                 │   │
│  │  TLV: VLAN ID                                │   │
│  └──────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────┘
```

#### DTP (Dynamic Trunking Protocol)

```
┌─────────────────────────────────────────────────────┐
│                DTP Operation                         │
├─────────────────────────────────────────────────────┤
│                                                      │
│  voiphopper                    Switch               │
│       │                           │                 │
│       │--- DTP Desirable -------->│                 │
│       │                           │                 │
│       │<-- DTP Auto -------------│                 │
│       │                           │                 │
│       │--- DTP Trunk Init ------->│                 │
│       │                           │                 │
│       │<-- Trunk Established ----│                 │
│       │                           │                 │
│       │   [Port becomes trunk]    │                 │
│       │                           │                 │
│       │--- 802.1Q VLAN XX ------->│                 │
│       │   [Access Voice VLAN]     │                 │
│                                                      │
└─────────────────────────────────────────────────────┘
```

### VLAN Hopping Attack

| Phase | Description | Protocol |
|-------|-------------|----------|
| 1 | Send DTP frames | DTP |
| 2 | Negotiate trunk | DTP |
| 3 | Tag frames | 802.1Q |
| 4 | Access target VLAN | 802.1Q |

## 4. Command Reference

### Basic Syntax

```
voiphopper [options]
```

### Interface Options

| Flag | Description | Example |
|------|-------------|---------|
| `-i` | Network interface | `-i eth0` |
| `-v` | Target VLAN | `-v 100` |
| `-d` | Daemon mode | `-d` |

### Protocol Options

| Flag | Description | Example |
|------|-------------|---------|
| `-c` | CDP mode | `-c` |
| `-l` | LLDP mode | `-l` |
| `-a` | All protocols | `-a` |

### Timing Options

| Flag | Description | Example |
|------|-------------|---------|
| `-t` | Timeout (seconds) | `-t 60` |
| `-r` | Retry count | `-r 3` |

### Control Options

| Flag | Description |
|------|-------------|
| `-b` | Bypass VLAN |
| `-q` | Quiet mode |
| `-v` | Verbose output |

### Output Options

| Flag | Description |
|------|-------------|
| `-o` | Output file |
| `-w` | PCAP capture |

## 5. Examples

### Basic VLAN Discovery

```bash
# Discover voice VLAN via CDP
sudo voiphopper -i eth0 -c

# Discover voice VLAN via LLDP
sudo voiphopper -i eth0 -l

# Try all protocols
sudo voiphopper -i eth0 -a
```

### VLAN Hopping

```bash
# Hop to specific VLAN
sudo voiphopper -i eth0 -v 100

# Auto-discover and hop
sudo voiphopper -i eth0 -a

# Bypass mode
sudo voiphopper -i eth0 -b
```

### Timing and Control

```bash
# Extended timeout
sudo voiphopper -i eth0 -t 120

# Retry discovery
sudo voiphopper -i eth0 -r 5

# Quiet mode
sudo voiphopper -i eth0 -q
```

### Output and Capture

```bash
# Save results
sudo voiphopper -i eth0 -o results.txt

# Capture traffic
sudo voiphopper -i eth0 -w voice_traffic.pcap

# Verbose output
sudo voiphopper -i eth0 -v
```

### Advanced Scenarios

```bash
# Daemon mode (background)
sudo voiphopper -i eth0 -d

# Specific VLAN with verbose
sudo voiphopper -i eth0 -v 100 -v

# LLDP with timeout
sudo voiphopper -i eth0 -l -t 60
```

## 6. Advanced Usage

### Automation Scripts

```bash
#!/bin/bash
# Automated voice VLAN discovery
INTERFACE="eth0"
OUTPUT="/tmp/voip_vlans_$(date +%Y%m%d).txt"

echo "[*] Starting voice VLAN discovery on $INTERFACE"

# CDP discovery
echo "[*] Trying CDP..."
sudo voiphopper -i $INTERFACE -c -o ${OUTPUT}_cdp.txt

# LLDP discovery
echo "[*] Trying LLDP..."
sudo voiphopper -i $INTERFACE -l -o ${OUTPUT}_lldp.txt

# Parse results
echo "[*] Discovered VLANs:"
grep -i "vlan" ${OUTPUT}_*.txt

echo "[+] Discovery complete"
```

### Network Reconnaissance

```bash
#!/bin/bash
# Full VoIP network reconnaissance
INTERFACE="eth0"

# Step 1: Discover voice VLAN
echo "[*] Step 1: Voice VLAN Discovery"
sudo voiphopper -i $INTERFACE -a -o discovery.txt

# Step 2: Extract VLAN ID
VLAN=$(grep -oP 'VLAN\s+\K\d+' discovery.txt | head -1)
echo "[*] Found Voice VLAN: $VLAN"

# Step 3: Hop to VLAN
echo "[*] Step 2: VLAN Hopping"
sudo voiphopper -i $INTERFACE -v $VLAN

# Step 4: Scan voice VLAN
echo "[*] Step 3: Scanning Voice VLAN"
sudo svmap 192.168.$VLAN.0/24

echo "[+] Reconnaissance complete"
```

### Integration with Other Tools

```bash
# voiphopper + tcpdump
sudo voiphopper -i eth0 -v 100 &
sudo tcpdump -i eth0 -w voice.pcap &
wait

# voiphopper + nmap
sudo voiphopper -i eth0 -v 100
nmap -sU -p 5060 192.168.100.0/24

# voiphopper + sipvicious
sudo voiphopper -i eth0 -v 100
svmap 192.168.100.0/24
svwar -e 1000-9999 192.168.100.100
```

## 7. Detection and Defense

### Detection Indicators

| Indicator | Description |
|-----------|-------------|
| CDP/LLDP | Unexpected protocol traffic |
| DTP | Trunk negotiation attempts |
| VLAN Mismatch | Untagged traffic on wrong VLAN |
| MAC Address | Unknown device on voice VLAN |

### Defensive Measures

**Switch Configuration:**

```
! Cisco IOS - Disable DTP
interface GigabitEthernet0/1
  switchport mode access
  switchport nonegotiate
  spanning-tree portfast

! Disable unused ports
interface GigabitEthernet0/24
  shutdown
  switchport mode access
  switchport access vlan 999

! Restrict CDP/LLDP
no cdp run
no lldp run
```

**VLAN Security:**

```
! Private VLAN
vlan 100
  private-vlan primary
  private-vlan association 101

! VLAN ACL
ip access-list extended VOICE-VLAN
  permit udp 192.168.100.0 0.0.0.255 any eq 5060
  deny ip any any
```

**Monitoring:**

```bash
# Detect DTP
tcpdump -i eth0 -n ether proto 0x2004

# Detect CDP
tcpdump -i eth0 -n ether proto 0x2000

# Detect LLDP
tcpdump -i eth0 -n ether proto 0x88CC

# Monitor VLAN changes
watch -n 1 "bridge fdb show vlan 100"
```

### Countermeasures for Testers

- Use `-c` for CDP-only (less noisy)
- Use `-l` for LLDP-only
- Avoid `-b` bypass in production
- Document all VLAN hops
- Revert changes after assessment

## 8. Troubleshooting

### Common Issues

#### No Voice VLAN Found

```bash
# Verify interface
ip link show eth0

# Check for CDP/LLDP
tcpdump -i eth0 -n ether proto 0x2000 or ether proto 0x88CC

# Try different protocol
sudo voiphopper -i eth0 -a

# Increase timeout
sudo voiphopper -i eth0 -t 120
```

#### VLAN Hopping Fails

```bash
# Check DTP is enabled on switch
# (Requires switch access)

# Try different VLAN
sudo voiphopper -i eth0 -v 100

# Verify trunk negotiation
tcpdump -i eth0 -n ether proto 0x2004
```

#### No Traffic After Hop

```bash
# Verify VLAN assignment
ip link show eth0.100

# Check routing
ip route show

# Verify DHCP
dhclient eth0.100

# Manual IP configuration
sudo ip addr add 192.168.100.10/24 dev eth0.100
sudo ip link set eth0.100 up
```

#### Permission Denied

```bash
# Run as root
sudo voiphopper -i eth0

# Check capabilities
getcap /usr/bin/voiphopper

# Add capabilities
sudo setcap cap_net_raw,cap_net_admin+eip /usr/bin/voiphopper
```

### Performance Issues

| Symptom | Solution |
|---------|----------|
| Slow discovery | Increase `-t` timeout |
| No response | Check switch configuration |
| VLAN hop fails | Verify DTP is enabled |
| No traffic | Check VLAN routing |

### Debug Commands

```bash
# Verbose output
sudo voiphopper -i eth0 -v

# Packet capture
sudo voiphopper -i eth0 -w debug.pcap

# Check interface status
ip -d link show eth0

# Verify VLAN
bridge vlan show
```

### Cleanup

```bash
# Remove VLAN interface
sudo ip link delete eth0.100

# Reset interface
sudo ip link set eth0 down
sudo ip link set eth0 up

# Restore original state
sudo dhclient eth0
```

## Resources

### Official Documentation
- [voiphopper GitHub](https://github.com/sec51/voiphopper)
- [Sec51 Tools](https://www.sec51.com/)

### Related Tools
- **yersinia**: Layer 2 attack framework
- **scapy**: Packet manipulation
- **dsniff**: Network sniffing
- **ngrep**: Network grep

### Protocol References
- [CDP Protocol](https://www.cisco.com/c/en/us/td/docs/routers/access/wireless/software/guide/SYSCFG.html)
- [LLDP Protocol](https://www.ieee802.org/1/pages/802.1ab.html)
- [DTP Protocol](https://www.cisco.com/c/en/us/td/docs/switches/lan/catalyst/configuration_guides.html)

### Learning Resources
- [VLAN Hacking](https://www.cisco.com/c/en/us/support/docs/lan-switching/vlans/13423-VLAN-Hopping.html)
- [VoIP Security](https://www.owasp.org/index.php/VoIP)
- [Network Segmentation](https://www.sans.org/white-papers/network-segmentation/)

### Related MITRE ATT&CK Techniques
- **T1557 - Adversary-in-the-Middle**: VLAN hopping for MitM
- **T1040 - Network Sniffing**: Capturing voice traffic
- **T1572 - Protocol Tunneling**: VLAN hopping
- **T1090 - Proxy**: Network pivoting
