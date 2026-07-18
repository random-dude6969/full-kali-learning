---
title: "hping3 — TCP/IP Packet Assembler and Sender"
description: "Craft and send custom TCP, UDP, ICMP, and ARP packets for advanced network discovery, firewall testing, and OS fingerprinting."
mitre_tactic: discovery
mitre_tactic_id: TA0007
mitre_subcategory: remote_system_discovery
categories:
  - packet-crafting
  - network-testing
  - port-scanning
tags:
  - tcp
  - icmp
  - packet-crafting
  - firewall-testing
  - traceroute
install_command: "sudo apt install hping3"
version: "3.a2.ds2-9"
github_repo: ""
kali_tools_page: "https://www.kali.org/tools/hping3/"
difficulty_level: intermediate
requires_root: true
gui_available: false
related_tools:
  - nping
  - scapy
  - nmap
  - arping
---

# hping3 — TCP/IP Packet Assembler and Sender

## 1. Introduction

**hping3** is a versatile TCP/IP packet assembler and sender that can craft custom packets at nearly every level of the network stack. It supports TCP, UDP, ICMP, RAW-IP, and ARP protocols, making it an essential tool for network testing, firewall auditing, and advanced host discovery.

**Use cases:**
- Firewall rule testing and bypass
- Advanced port scanning (SYN, FIN, XMAS scans)
- Traceroute through non-standard ports
- OS fingerprinting via TCP options
- DDoS testing (controlled, authorized environments)
- Custom packet injection and manipulation

**Key advantage:** hping3 provides granular control over every packet field — flags, sequence numbers, window sizes, TTL, and payload — enabling precise testing of network behavior.

**Warning:** hping3 is a powerful tool. Use only on networks you own or have explicit authorization to test. Unauthorized use is illegal.

---

## 2. Installation

**Install via apt:**
```bash
sudo apt install hping3
```

**Verify installation:**
```bash
hping3 --version
```

**Permissions:** hping3 requires root to craft and send raw packets.

---

## 3. Core Concepts

### 3.1 Packet Crafting

hping3 lets you construct packets field by field:
- **Protocol**: TCP, UDP, ICMP, RAW-IP, ARP
- **Flags**: SYN, ACK, FIN, RST, PSH, URG
- **Sequence numbers**: Custom starting sequence
- **Window size**: TCP window size manipulation
- **TTL**: Time-to-live for TTL-based fingerprinting
- **Payload**: Custom data in packet body

### 3.2 Scan Types

| Scan | Flag | Description |
|------|------|-------------|
| SYN scan | `-S` | Stealthy half-open scan |
| FIN scan | `-F` | Closed ports reply with RST |
| XMAS scan | `-FPU` | FIN+PSH+URG flags |
| NULL scan | (no flags) | No flags set |
| ACK scan | `-A` | Maps firewall rules |
| Window scan | `-W` | Uses TCP window size |

### 3.3 Traceroute

hping3 traceroute uses TCP packets instead of UDP/ICMP, allowing traversal through firewalls that block standard traceroute:
```bash
hping3 -z -p 80 192.168.1.1
```

### 3.4 OS Fingerprinting

By analyzing TCP options, window sizes, and TTL values in responses, hping3 can identify remote operating systems.

---

## 4. Command Reference

### Basic Usage

```bash
# SYN scan on port 80
sudo hping3 -S 192.168.1.1 -p 80

# ICMP ping
sudo hping3 -1 192.168.1.1

# Traceroute on port 80
sudo hping3 -z -p 80 192.168.1.1
```

### Flags

| Flag | Description |
|------|-------------|
| `-c COUNT` | Send COUNT packets |
| `-i U` | Interval between packets (u=micro, m=milli) |
| `-n` | Numeric output (no DNS) |
| `-I IFACE` | Interface to use |
| `-q` | Quiet output — show only results |
| `-V` | Verbose output |
| `-D` | Enable debug mode |
| `-z` | Traceroute mode |
| `-Z` | Set TCP flags (SYN, ACK, etc.) |
| `-S` | Set SYN flag |
| `-A` | Set ACK flag |
| `-P` | Set PSH flag |
| `-U` | Set URG flag |
| `-F` | Set FIN flag |
| `-R` | Set RST flag |
| `-W` | Set window size |
| `-p PORT` | Destination port |
| `-s SIZE` | Packet size |
| `-a IP` | Spoof source IP |
| `--flood` | Send packets as fast as possible |
| `--rand-source` | Random source IP |
| `-1` | ICMP mode |
| `-2` | UDP mode |
| `-9` | Listen mode |

---

## 5. Examples

### Basic

**ICMP ping:**
```bash
sudo hping3 -1 192.168.1.1
```
Output:
```
HPING in eth0: icmp mode set, 28 headers + 0 data
len=46 ip=192.168.1.1 ttl=64 id=12345 icmp_seq=0 rtt=0.5ms
```

**TCP SYN to port 80:**
```bash
sudo hping3 -S 192.168.1.1 -p 80
```

**Send 10 packets:**
```bash
sudo hping3 -S 192.168.1.1 -p 80 -c 10
```

### Intermediate

**Traceroute on port 80:**
```bash
sudo hping3 -z -p 80 -c 1 192.168.1.$i
```
Output:
```
hop=1 TTL=0 in=40 ttl=64 id=12345 ip=192.168.1.1 rtt=0.5 ms
hop=2 TTL=1 in=40 ttl=63 id=12346 ip=10.0.0.1 rtt=1.2 ms
```

**ACK scan to map firewall:**
```bash
sudo hping3 -A 192.168.1.1 -p 80 -c 3
```
- RST response = port open (no firewall)
- No response = port filtered (firewall blocking)

**FIN scan:**
```bash
sudo hping3 -F 192.168.1.1 -p 80 -c 3
```

**Spoofed source IP:**
```bash
sudo hping3 -S 192.168.1.1 -a 10.0.0.100 -p 80
```

### Advanced

**Custom flags (SYN+ACK):**
```bash
sudo hping3 -SA 192.168.1.1 -p 80 -c 3
```

**Set window size:**
```bash
sudo hping3 -S 192.168.1.1 -p 80 -W 1024 -c 3
```

**Custom packet size:**
```bash
sudo hping3 -S 192.168.1.1 -p 80 -s 1000 -c 3
```

**Flood mode (testing only):**
```bash
sudo hping3 -S 192.168.1.1 -p 80 --flood
```
**Warning:** Use only in controlled testing environments.

**Random source IPs:**
```bash
sudo hping3 -S 192.168.1.1 -p 80 --rand-source -c 100
```

### Real-World

**Firewall rule audit:**
```bash
#!/bin/bash
TARGET="192.168.1.1"
for port in 22 80 443 3306 8080; do
    result=$(sudo hping3 -S $TARGET -p $port -c 1 -n 2>&1 | grep "flags=SA")
    if [ -n "$result" ]; then
        echo "Port $port: OPEN (SYN+ACK received)"
    else
        echo "Port $port: FILTERED/CLOSED"
    fi
done
```

**Custom traceroute through firewall:**
```bash
sudo hping3 -z -p 443 -c 3 192.168.1.1
```
Traceroute using HTTPS port instead of standard ICMP.

**OS fingerprinting via TTL:**
```bash
sudo hping3 -1 -c 1 192.168.1.1
```
Analyze TTL in response:
- TTL 64: Linux/Unix
- TTL 128: Windows
- TTL 255: Network device

---

## 6. Advanced

### 6.1 Packet Manipulation

**Set TTL:**
```bash
sudo hping3 -1 --ttl 128 192.168.1.1 -c 3
```

**Set IP ID:**
```bash
sudo hping3 -1 --id 12345 192.168.1.1 -c 3
```

**Custom TCP options:**
```bash
sudo hping3 -S 192.168.1.1 -p 80 -M 1 -m 1460 -c 3
```
Sets MSS (Maximum Segment Size) to 1460.

### 6.2 Protocol Combinations

**UDP scan:**
```bash
sudo hping3 -2 192.168.1.1 -p 53 -c 3
```

**ICMP timestamp request:**
```bash
sudo hping3 -1 --icmptype 13 192.168.1.1 -c 3
```

**ARP mode:**
```bash
sudo hping3 -8 -H arp 192.168.1.1
```

### 6.3 Listening Mode

**Capture and respond to packets:**
```bash
sudo hping3 -9 eth0
```
Listens on eth0 and responds to incoming packets — useful for debugging.

### 6.4 Scripted Testing

**Port range scan:**
```bash
for port in $(seq 1 1024); do
    sudo hping3 -S 192.168.1.1 -p $port -c 1 -n 2>&1 | grep -q "flags=SA" && echo "Port $port: OPEN"
done
```

**Response time measurement:**
```bash
sudo hping3 -1 192.168.1.1 -c 100 | grep rtt | awk '{print $NF}' | tr -d 'ms' | \
    awk '{sum+=$1; count++} END {print "Avg RTT:", sum/count, "ms"}'
```

---

## 7. Detection and Defense

### 7.1 Detection

**Monitor for hping3 traffic:**
```bash
sudo tcpdump -i eth0 -n 'tcp[tcpflags] & (tcp-syn|tcp-ack) == tcp-syn and tcp[tcpflags] & tcp-ack == 0' -c 100
```

**Detect SYN floods:**
```bash
sudo tcpdump -i eth0 'tcp[tcpflags] == tcp-syn' -n | \
    awk '{print $3}' | cut -d. -f1-4 | sort | uniq -c | sort -rn
```

**Identify spoofed IPs:**
- Multiple source IPs from same MAC (ARP table)
- Impossible IP/MAC combinations
- High volume from single source

### 7.2 Defense

**Rate limit SYN packets:**
```bash
sudo iptables -A INPUT -p tcp --syn -m limit --limit 10/s -j ACCEPT
sudo iptables -A INPUT -p tcp --syn -j DROP
```

**SYN cookies:**
```bash
echo 1 > /proc/sys/net/ipv4/tcp_syncookies
```

**Connection rate limiting:**
```bash
sudo iptables -A INPUT -p tcp --syn -m connlimit --connlimit-above 50 -j DROP
```

**Firewall rules:**
```bash
# Block hping3-style floods
sudo iptables -A INPUT -p tcp --tcp-flags ALL NONE -j DROP
sudo iptables -A INPUT -p tcp --tcp-flags ALL ALL -j DROP
```

---

## 8. Troubleshooting

### 8.1 Common Issues

**"You need raw socket access":**
```bash
sudo hping3 -S 192.168.1.1 -p 80
```
Always use root for raw packet operations.

**"No route to host":**
```bash
ip route get 192.168.1.1
ping -c 1 192.168.1.1
```
Verify routing and connectivity.

**No responses received:**
- Firewall may be blocking hping3 traffic
- Try different scan type (FIN, XMAS)
- Check with tcpdump: `sudo tcpdump -i eth0 host 192.168.1.1`

**Interface errors:**
```bash
sudo hping3 -I eth0 -S 192.168.1.1 -p 80
```
Specify the correct interface.

### 8.2 Performance Issues

**Slow scanning:**
```bash
sudo hping3 -S 192.168.1.1 -p 80 -i u100 -c 100
```
Use microsecond intervals for faster sending.

**Packet loss:**
```bash
sudo hping3 -S 192.168.1.1 -p 80 -i m100 -c 100
```
Increase interval to 100ms for more reliable responses.

### 8.3 Firewall Issues

**Firewall dropping packets:**
```bash
# Try different scan types
sudo hping3 -F 192.168.1.1 -p 80 -c 3  # FIN scan
sudo hping3 -S -PU 192.168.1.1 -p 80 -c 3  # SYN+UDP
```

---

## Resources

- [hping3 man page](https://linux.die.net/man/8/hping3)
- [hping3 homepage](https://www.hping.org/)
- [Kali hping3 tool page](https://www.kali.org/tools/hping3/)
- [TCP/IP Protocol Suite (RFC 793)](https://tools.ietf.org/html/rfc793)
- [Nmap port scanning reference](https://nmap.org/book/port-scanning.html)
