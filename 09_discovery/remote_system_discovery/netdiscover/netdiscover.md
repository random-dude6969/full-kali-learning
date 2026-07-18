---
title: "netdiscover — Active/Passive ARP Network Discovery"
description: "Discover hosts on local networks using ARP requests or passive sniffing, displaying hostnames, vendors, and OS information."
mitre_tactic: discovery
mitre_tactic_id: TA0007
mitre_subcategory: remote_system_discovery
categories:
  - network-discovery
  - arp
  - host-enumeration
tags:
  - arp
  - scanner
  - passive
  - host-discovery
  - network
install_command: "sudo apt install netdiscover"
version: "0.3-pre-beta7-2"
github_repo: ""
kali_tools_page: "https://www.kali.org/tools/netdiscover/"
difficulty_level: beginner
requires_root: true
gui_available: true
related_tools:
  - arp-scan
  - arping
  - nmap
---

# netdiscover — Active/Passive ARP Network Discovery

## 1. Introduction

**netdiscover** is an ARP-based network discovery tool that operates in both active and passive modes. It sends ARP requests to discover hosts (active) or sniffs ARP traffic to learn about existing hosts (passive), displaying hostnames, MAC vendors, and OS hints.

**Use cases:**
- Discover all hosts on local subnets
- Passively monitor network traffic for host information
- Identify device vendors via MAC address lookup
- Detect unauthorized devices on the network
- Map network topology at Layer 2

**Key advantage:** netdiscover's passive mode provides host information without generating any network traffic — completely invisible to detection systems.

---

## 2. Installation

**Install via apt:**
```bash
sudo apt install netdiscover
```

**Verify installation:**
```bash
netdiscover --version
```

**Permissions:** netdiscover requires root for raw packet operations in both active and passive modes.

**GUI availability:** netdiscover includes an ncurses-based interactive interface for real-time monitoring.

---

## 3. Core Concepts

### 3.1 Active Mode

In active mode, netdiscover sends ARP requests to a specified IP range and collects responses:
```bash
sudo netdiscover -i eth0 -r 192.168.1.0/24
```

### 3.2 Passive Mode

In passive mode, netdiscover sniffs all ARP traffic on the network without sending anything:
```bash
sudo netdiscover -i eth0 -p
```
This learns about hosts by observing ARP requests and replies from other devices.

### 3.3 ARP Cache Building

netdiscover builds and maintains an ARP cache showing:
- IP address
- MAC address
- Vendor (OUI lookup)
- Hostname (if resolvable)
- Packet count

### 3.4 Output Format

| Field | Description |
|-------|-------------|
| IP | Discovered IP address |
| MAC | MAC address |
| Vendor | Manufacturer from OUI |
| Hostname | DNS or NetBIOS name |
| Count | Number of ARP packets seen |

---

## 4. Command Reference

### Basic Usage

```bash
# Active scan on subnet
sudo netdiscover -i eth0 -r 192.168.1.0/24

# Passive monitoring
sudo netdiscover -i eth0 -p

# Auto-detect interface
sudo netdiscover -i eth0
```

### Flags

| Flag | Description |
|------|-------------|
| `-i IFACE` | Network interface |
| `-r RANGE` | Target IP range (CIDR) |
| `-p` | Passive mode |
| `-s SOURCE` | Source MAC address |
| `-c COUNT` | Number of requests per host |
| `-P` | Promiscuous mode |
| `-N` | No hostname resolution |
| `-n` | Numeric output |
| `-d DELAY` | Delay between requests (ms) |
| `--write FILE` | Write results to file |
| `--browse` | Interactive ncurses interface |
| `--semi passive` | Semi-passive mode |
| `-f` | Fast mode — fewer requests |

---

## 5. Examples

### Basic

**Scan a subnet:**
```bash
sudo netdiscover -i eth0 -r 192.168.1.0/24
```
Output:
```
 Currently scanning: 192.168.1.0/24   |   Screen View: Unique Hosts

 2   192.168.1.1     08:00:27:a1:b2:c3   Cadmus Technolog    1
 1   192.168.1.50    00:50:56:c0:00:08   VMware, Inc.        3
 5   192.168.1.100   b8:27:eb:12:34:56   Raspberry Pi Found  2

 3 packets captured by filter
```

**Passive mode:**
```bash
sudo netdiscover -i eth0 -p
```

### Intermediate

**Custom range with delay:**
```bash
sudo netdiscover -i eth0 -r 192.168.1.0/24 -d 1000
```
1 second delay between requests to avoid detection.

**No hostname resolution:**
```bash
sudo netdiscover -i eth0 -r 192.168.1.0/24 -N
```

**Write results to file:**
```bash
sudo netdiscover -i eth0 -r 192.168.1.0/24 --write results.txt
```

### Advanced

**Interactive mode:**
```bash
sudo netdiscover -i eth0 --browse
```
Opens ncurses interface showing real-time host discovery.

**Semi-passive mode:**
```bash
sudo netdiscover -i eth0 --semi passive -r 192.168.1.0/24
```
Combines active scanning with passive observation.

**Custom source MAC:**
```bash
sudo netdiscover -i eth0 -r 192.168.1.0/24 -s 00:11:22:33:44:55
```

### Real-World

**Network inventory:**
```bash
#!/bin/bash
sudo netdiscover -i eth0 -r 192.168.1.0/24 -N --write "inventory_$(date +%F).txt"
echo "Inventory saved to inventory_$(date +%F).txt"
```

**Continuous monitoring:**
```bash
while true; do
    sudo netdiscover -i eth0 -r 192.168.1.0/24 -N --write "scan_$(date +%H%M).txt"
    sleep 300
done
```

**Passive monitoring for unknown devices:**
```bash
sudo netdiscover -i eth0 -p --browse
```
Watch for new devices appearing on the network.

---

## 6. Advanced

### 6.1 Active vs Passive Comparison

| Feature | Active | Passive |
|---------|--------|---------|
| Generates traffic | Yes | No |
| Detectable | Yes | No |
| Speed | Faster | Slower |
| Complete scan | Yes | Partial |
| Stealth | Low | High |

### 6.2 OUI Database

netdiscover uses its internal OUI database for vendor identification. To update:
```bash
sudo wget -O /usr/share/netdiscover/ieee-oui.txt \
    https://raw.githubusercontent.com/wireshark/wireshark/master/manuf
```

### 6.3 Scripted Automation

**Export to CSV:**
```bash
sudo netdiscover -i eth0 -r 192.168.1.0/24 -N | awk 'NR>4{print $1","$2","$3}' > hosts.csv
```

**Diff between scans:**
```bash
sudo netdiscover -i eth0 -r 192.168.1.0/24 -N > scan1.txt
sudo netdiscover -i eth0 -r 192.168.1.0/24 -N > scan2.txt
diff <(sort scan1.txt) <(sort scan2.txt)
```

### 6.4 Combining with Other Tools

**netdiscover + arping for verification:**
```bash
sudo netdiscover -i eth0 -r 192.168.1.0/24 -N | awk 'NR>4{print $1}' | \
    while read ip; do
        arping -c 1 -f "$ip" 2>/dev/null && echo "$ip verified"
    done
```

**netdiscover + nmap for deep scan:**
```bash
sudo netdiscover -i eth0 -r 192.168.1.0/24 -N | awk 'NR>4{print $1}' | \
    xargs -I {} nmap -sV -T4 {}
```

---

## 7. Detection and Defense

### 7.1 Detection

**Active mode detection:**
```bash
sudo tcpdump -i eth0 arp -n | grep "who-has"
```
Active netdiscover sends ARP requests visible on the network.

**Passive mode detection:**
Passive mode is undetectable — it only listens, never transmits.

**ARP anomaly detection:**
```bash
# Monitor for ARP floods
sudo tcpdump -i eth0 arp -n -l | \
    awk '{count[$3]++} END {for (ip in count) if (count[ip]>10) print ip, count[ip]}'
```

### 7.2 Defense

**ARP rate limiting:**
```bash
sudo iptables -A INPUT -p arp -m limit --limit 10/s -j ACCEPT
sudo iptables -A INPUT -p arp -j DROP
```

**Dynamic ARP Inspection (DAI):**
Enable DAI on managed switches to validate ARP packets against DHCP snooping bindings.

**Static ARP for critical hosts:**
```bash
sudo arp -s 192.168.1.1 08:00:27:a1:b2:c3
```

**Port security:**
Configure switch port security to limit MAC addresses per port.

---

## 8. Troubleshooting

### 8.1 Common Issues

**"No interface specified":**
```bash
ip -br link show
sudo netdiscover -i eth0 -r 192.168.1.0/24
```

**"Permission denied":**
```bash
sudo netdiscover -i eth0 -r 192.168.1.0/24
```

**No hosts found:**
- Verify interface is up: `ip link show eth0`
- Check subnet: `ip addr show eth0`
- Try passive mode: `sudo netdiscover -i eth0 -p`

**Interface in use by other process:**
```bash
sudo fuser eth0 2>/dev/null
sudo netdiscover -i eth0 -r 192.168.1.0/24
```

### 8.2 Performance Issues

**Slow discovery:**
```bash
sudo netdiscover -i eth0 -r 192.168.1.0/24 -d 100 -c 1
```
Reduce delay and count for faster scanning.

**Missing hosts in passive mode:**
- Increase capture time
- Try active mode for complete scan
- Check if hosts are generating ARP traffic

### 8.3 Interface Issues

**Virtual interfaces:**
```bash
sudo ip link set eth0 promisc on
sudo netdiscover -i eth0 -r 192.168.1.0/24
```
Enable promiscuous mode for virtual interfaces.

**Docker networks:**
```bash
# Find Docker bridge interface
docker network inspect bridge | grep "Name"
sudo netdiscover -i br-xxxxxxxxxxxx -r 172.17.0.0/16
```

**Wi-Fi interfaces:**
```bash
# Monitor mode for wireless
sudo ip link set wlan0 down
sudo iw dev wlan0 set monitor control
sudo ip link set wlan0 up
sudo netdiscover -i wlan0 -r 192.168.1.0/24
```

### 8.4 Security Considerations

**Detection of active scanning:**
Active netdiscover generates ARP requests visible on the network. Use passive mode for stealth:
```bash
sudo netdiscover -i eth0 -p  # Undetectable
```

**ARP flood detection:**
```bash
# Monitor for ARP floods
sudo tcpdump -i eth0 arp -n -l | \
    awk '{count[$3]++} END {for (ip in count) if (count[ip]>50) print "FLOOD from", ip, count[ip]}'
```

**Defensive monitoring:**
Deploy arpwatch to detect unauthorized ARP activity:
```bash
sudo apt install arpwatch
sudo arpwatch -i eth0 -e admin@company.com
```

### 8.5 Comparison with Similar Tools

| Feature | netdiscover | arp-scan | arping | nmap -sn |
|---------|-------------|----------|--------|----------|
| Passive mode | Yes | No | No | No |
| Interactive UI | Yes | No | No | No |
| OUI lookup | Yes | Yes | No | No |
| Cross-subnet | No | No | No | Yes |
| Speed | Medium | Fast | Slow | Medium |
| Stealth (passive) | High | Low | Low | Low |

Use netdiscover for passive monitoring and interactive network discovery. Use arp-scan for faster active scanning. Use arping for individual host verification.

### 8.6 Output Formatting

**Export to CSV:**
```bash
sudo netdiscover -i eth0 -r 192.168.1.0/24 -N | awk 'NR>4{print $1","$2","$3","$4}' > hosts.csv
```

**Extract IPs only:**
```bash
sudo netdiscover -i eth0 -r 192.168.1.0/24 -N | awk 'NR>4{print $1}' > ips.txt
```

**Filter by vendor:**
```bash
sudo netdiscover -i eth0 -r 192.168.1.0/24 -N | grep -i "vmware"
```

**JSON export:**
```bash
sudo netdiscover -i eth0 -r 192.168.1.0/24 -N | awk 'NR>4{print "{\"ip\":\""$1"\",\"mac\":\""$2"\",\"vendor\":\""$3"\"}"}'
```

---

## Resources

- [netdiscover man page](https://linux.die.net/man/8/netdiscover)
- [netdiscover source](https://sourceforge.net/projects/netdiscover/)
- [Kali netdiscover tool page](https://www.kali.org/tools/netdiscover/)
- [ARP Protocol (RFC 826)](https://tools.ietf.org/html/rfc826)
- [OUI lookup](https://macvendors.com/)
