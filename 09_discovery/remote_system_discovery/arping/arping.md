---
title: "arping — ARP-Level Ping Utility"
description: "Send ARP requests to discover hosts at Layer 2, bypass ICMP blocks, and detect duplicate IP addresses on local networks."
mitre_tactic: discovery
mitre_tactic_id: TA0007
mitre_subcategory: remote_system_discovery
categories:
  - network-discovery
  - arp
  - layer2
tags:
  - arp
  - ping
  - layer2
  - network
  - discovery
install_command: "sudo apt install arping"
version: "2.21-6"
github_repo: ""
kali_tools_page: "https://www.kali.org/tools/arping/"
difficulty_level: beginner
requires_root: false
gui_available: false
related_tools:
  - hping3
  - nmap
  - netdiscover
  - arp-scan
---

# arping — ARP-Level Ping Utility

## 1. Introduction

**arping** is a Layer 2 utility that sends ARP (Address Resolution Protocol) requests to discover whether a host is online on the local network. Unlike traditional ICMP ping, arping operates at the data link layer and can detect hosts that block ICMP traffic but still respond to ARP requests.

**Use cases:**
- Verify host reachability when ICMP is blocked by firewalls
- Detect duplicate IP addresses on a subnet
- Track ARP table changes and host migration
- Confirm host presence without relying on Layer 3 connectivity

**Key distinction:** ARP requests only work on local broadcast domains. Routers do not forward ARP traffic, so arping cannot reach hosts across subnets without being on the same Layer 2 segment.

**Limitation:** Requires the source interface to be on the same subnet as the target. ARP is a local-only protocol — it does not traverse routers.

---

## 2. Installation

**Install via apt:**
```bash
sudo apt install arping
```

**Verify installation:**
```bash
arping -V
```

**Check available interfaces:**
```bash
ip -br link
```

**Permissions:** arping can run without root on most systems, but sending ARP packets with spoofed source addresses requires root privileges.

---

## 3. Core Concepts

### 3.1 ARP Protocol Basics

ARP resolves IP addresses to MAC addresses on local networks. When a host wants to send data to an IP on the same subnet, it broadcasts an ARP request: "Who has this IP? Tell my MAC address."

**ARP request flow:**
1. Host A broadcasts: "Who has 192.168.1.50?"
2. All hosts on the subnet receive the broadcast
3. Host B (192.168.1.50) responds: "I am 192.168.1.50, my MAC is aa:bb:cc:dd:ee:ff"
4. Host A records the mapping in its ARP cache

### 3.2 arping vs ping

| Feature | arping | ping |
|---------|--------|------|
| Layer | 2 (Data Link) | 3 (Network) |
| Protocol | ARP | ICMP |
| Cross-subnet | No | Yes |
| Bypasses ICMP blocks | Yes | No |
| Spoofed source IP | Supported | Not supported |

### 3.3 Duplicate IP Detection

arping's `-D` flag sends ARP requests and listens for multiple replies. If two hosts claim the same IP, both respond, signaling a duplicate address conflict.

### 3.4 ARP Table Impact

Every arping reply updates the sender's ARP table. Sending arping with `-S` (update source) forces ARP cache updates on the target, which can be useful for tracking host movement across interfaces.

---

## 4. Command Reference

### Basic Usage

```bash
# Send 4 ARP requests to a target
arping -c 4 192.168.1.1

# Ping a specific interface
arping -c 4 -i eth0 192.168.1.1

# Specify source IP (spoofed)
arping -c 4 -s 192.168.1.100 192.168.1.1
```

### Flags

| Flag | Description |
|------|-------------|
| `-c COUNT` | Number of ARP requests to send |
| `-s SOURCE` | Source IP address (spoofed, no ARP update) |
| `-S SOURCE` | Source IP address (updates ARP cache on target) |
| `-i INTERFACE` | Interface to send on |
| `-W WAIT` | Wait time in microseconds between requests |
| `-w DEADLINE` | Overall timeout in seconds |
| `-f` | Quit on first reply received |
| `-b` | Broadcast mode (send to broadcast address) |
| `-q` | Quiet mode — minimal output |
| `-D` | Duplicate address detection mode |
| `-V` | Print version and exit |
| `-h` | Print help and exit |

---

## 5. Examples

### Basic

**Ping a single host:**
```bash
arping -c 4 192.168.1.1
```
Output:
```
ARPING to 192.168.1.1 from 192.168.1.50 via eth0
Unicast reply from 192.168.1.1 [08:00:27:a1:b2:c3]  1.234ms
Unicast reply from 192.168.1.1 [08:00:27:a1:b2:c3]  0.987ms
Unicast reply from 192.168.1.1 [08:00:27:a1:b2:c3]  1.123ms
Unicast reply from 192.168.1.1 [08:00:27:a1:b2:c3]  0.876ms
Sent 4 probes (1 broadcast(s)), got 4 response(s)
```

**Specify source interface:**
```bash
arping -c 4 -i eth0 192.168.1.1
```

### Intermediate

**Detect duplicate IPs:**
```bash
arping -D -c 3 192.168.1.1
```
If multiple MAC addresses respond, a duplicate exists.

**Update ARP cache on target:**
```bash
arping -S 192.168.1.100 -c 4 192.168.1.1
```
This forces the target to update its ARP table with the spoofed source.

**Quit on first reply:**
```bash
arping -f -c 1 192.168.1.1
```
Useful for fast existence checks in scripts.

### Advanced

**Scripted host discovery:**
```bash
for ip in $(seq 1 254); do
    arping -c 1 -f -W 1 192.168.1.$ip 2>/dev/null && echo "192.168.1.$ip is alive"
done
```

**Detect host migration between subnets:**
```bash
arping -D -c 1 -i eth0 192.168.1.50
arping -D -c 1 -i eth1 192.168.1.50
```
If both interfaces see the same IP, the host may have migrated or is dual-homed.

**Custom timing with delay:**
```bash
arping -c 10 -W 200000 192.168.1.1
```
Sends 10 probes with a 200ms delay between each.

### Real-World

**Network audit — find all active hosts on a subnet:**
```bash
for ip in $(seq 1 254); do
    result=$(arping -c 1 -W 1 192.168.1.$ip 2>/dev/null | grep "Unicast reply")
    if [ -n "$result" ]; then
        mac=$(echo "$result" | grep -oP '\[.*?\]' | tr -d '[]')
        echo "192.168.1.$ip | MAC: $mac"
    fi
done
```

**Quick connectivity test through firewall:**
```bash
ping -c 1 192.168.1.1    # Fails — ICMP blocked
arping -c 1 192.168.1.1   # Succeeds — ARP still works
```

---

## 6. Advanced

### 6.1 Spoofed Source IP

arping supports spoofing the source IP in ARP requests. Use `-s` for spoofing without affecting the target's ARP cache, or `-S` to force an ARP cache update on the target.

**Key difference:**
- `-s 10.0.0.1`: Sends ARP from spoofed IP, target does NOT update its ARP table
- `-S 10.0.0.1`: Sends ARP from spoofed IP, target DOES update its ARP table

### 6.2 Combining with tcpdump

Capture ARP traffic while arping runs:
```bash
# Terminal 1: capture ARP
sudo tcpdump -i eth0 arp -w arp.pcap

# Terminal 2: send ARP requests
arping -c 4 192.168.1.1
```

### 6.3 Automation and Scripting

**Ping sweep with output parsing:**
```bash
#!/bin/bash
SUBNET="192.168.1"
OUTPUT="alive_hosts.txt"
> "$OUTPUT"

for i in $(seq 1 254); do
    if arping -c 1 -W 1 "${SUBNET}.${i}" &>/dev/null; then
        echo "${SUBNET}.${i}" >> "$OUTPUT"
    fi
done

echo "Alive hosts:"
cat "$OUTPUT"
```

**Rate-limited scanning:**
```bash
arping -c 1 -W 500000 192.168.1.$ip
```
500ms between probes avoids overwhelming the network.

### 6.4 IPv6 with arping

For IPv6 neighbor discovery, use `ndisc6` or `ping6` instead. arping is limited to IPv4 ARP.

---

## 7. Detection and Defense

### 7.1 Detection

**Monitor ARP traffic:**
```bash
sudo tcpdump -i eth0 arp -n
```

**Check ARP cache:**
```bash
arp -a
ip neigh show
```

**Unusual ARP patterns:**
- Multiple IPs claiming the same MAC (ARP spoofing)
- ARP requests from unexpected source IPs
- High volume of ARP broadcasts

### 7.2 Defense

**Static ARP entries:**
```bash
sudo arp -s 192.168.1.1 08:00:27:a1:b2:c3
```
Lock critical hosts to prevent ARP spoofing.

**DHCP snooping:**
Configure DHCP snooping on managed switches to validate DHCP assignments.

**Dynamic ARP Inspection (DAI):**
Enable DAI on switches to drop unauthorized ARP packets.

**arpwatch:**
Deploy arpwatch to monitor and alert on ARP changes:
```bash
sudo arpwatch -i eth0
```

---

## 8. Troubleshooting

### 8.1 Common Issues

**"Network is unreachable":**
```bash
ip route show
arping -i eth0 -c 4 192.168.1.1
```
Verify the interface exists and has a route to the target subnet.

**"No ARP replies received":**
- Target is offline
- ICMP or ARP is blocked
- Wrong interface selected

**"Permission denied":**
```bash
sudo arping -c 4 192.168.1.1
```
Spoofed source IPs require root.

**All probes show 0 responses:**
- Check cable connection
- Verify interface is up: `ip link show eth0`
- Confirm IP configuration: `ip addr show eth0`

### 8.2 Performance Issues

**Slow responses:**
```bash
arping -c 10 -W 50000 192.168.1.1
```
Increase the wait time (`-W`) for congested networks.

**Too many timeouts:**
```bash
arping -c 10 -W 500000 192.168.1.1
```
Reduce timeout for fast networks, increase for slow ones.

### 8.3 Virtualization Issues

Virtual bridges may not propagate ARP correctly. Ensure the virtual switch is in promiscuous mode or bridged mode for arping to work.

**Docker containers:**
```bash
# arping inside a container requires --net=host or CAP_NET_RAW
docker run --rm --net=host arping -c 4 192.168.1.1
```

**KVM/QEMU bridges:**
```bash
sudo ip link set virbr0 promisc on
arping -i virbr0 -c 4 192.168.122.1
```

**VMware/VirtualBox NAT:**
ARP may not work through NAT mode. Switch to bridged mode for arping to function correctly.

### 8.4 Security Considerations

**ARP spoofing risks:**
arping with `-S` can update target ARP tables with spoofed entries. This is useful for testing but can be malicious in unauthorized contexts.

**Network monitoring:**
ARP responses can be monitored to detect rogue devices:
```bash
sudo tcpdump -i eth0 arp -n -l | \
    awk '/reply from/ {print $NF, $(NF-2)}' | sort -u
```

**ARP table manipulation:**
```bash
# View current ARP table
arp -a
ip neigh show

# Flush ARP cache (requires root)
sudo ip neigh flush all

# Add static entry
sudo arp -s 192.168.1.1 08:00:27:a1:b2:c3
```

### 8.5 Comparison with Similar Tools

| Tool | Layer | Speed | Cross-subnet | Spoofing |
|------|-------|-------|--------------|----------|
| arping | L2 (ARP) | Fast | No | Yes |
| ping | L3 (ICMP) | Slow | Yes | No |
| nping | L3/L4 | Medium | Yes | Yes |
| nmap -sn | Mixed | Medium | Yes | Limited |
| hping3 | L3/L4 | Fast | Yes | Yes |

Use arping when Layer 2 reachability is the primary concern and the target is on the same subnet.

---

## Resources

- [arping man page](https://linux.die.net/man/8/arping)
- [ARP Protocol (RFC 826)](https://tools.ietf.org/html/rfc826)
- [Kali arping tool page](https://www.kali.org/tools/arping/)
- [arping GitHub (Thomas Habets)](https://github.com/troglobit/arping)
- [Network scanning with arping (SANS)](https://www.sans.org/)
