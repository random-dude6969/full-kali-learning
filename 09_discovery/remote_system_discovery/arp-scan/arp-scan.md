---
title: "arp-scan — ARP Network Scanner with OUI Fingerprinting"
description: "Scan local networks using ARP requests, identify hosts by MAC vendor (OUI), and detect unknown devices."
mitre_tactic: discovery
mitre_tactic_id: TA0007
mitre_subcategory: remote_system_discovery
categories:
  - network-scanning
  - arp
  - fingerprinting
tags:
  - arp
  - scanner
  - fingerprinting
  - oui
  - network
install_command: "sudo apt install arp-scan"
version: "1.9.2"
github_repo: "https://github.com/royhills/arp-scan"
kali_tools_page: "https://www.kali.org/tools/arp-scan/"
difficulty_level: beginner
requires_root: true
gui_available: false
related_tools:
  - arping
  - netdiscover
  - nmap -sn
---

# arp-scan — ARP Network Scanner with OUI Fingerprinting

## 1. Introduction

**arp-scan** is a fast Layer 2 network scanner that uses ARP requests to discover hosts on local networks. It sends raw ARP packets and collects responses, identifying each host by IP, MAC address, and vendor (via OUI database lookup).

**Use cases:**
- Rapid host discovery on local subnets
- Vendor identification via MAC address OUI prefix
- Detect rogue or unauthorized devices
- Map network topology at Layer 2
- Compare scanned results against expected devices

**Key advantage:** ARP scans are faster and more reliable than ICMP ping sweeps because ARP responses are mandatory on local networks — even firewalled hosts must reply to ARP requests.

---

## 2. Installation

**Install via apt:**
```bash
sudo apt install arp-scan
```

**Verify installation:**
```bash
arp-scan --version
```

**Update OUI database:**
```bash
sudo wget https://raw.githubusercontent.com/wireshark/wireshark/master/manuf -O /usr/share/arp-scan/ieee-oui.txt
```

**Permissions:** arp-scan requires root to craft and send raw ARP packets.

---

## 3. Core Concepts

### 3.1 ARP Scanning Process

1. arp-scan sends ARP "who-has" requests to each IP in the target range
2. Live hosts reply with their MAC addresses
3. arp-scan matches MAC prefixes against its OUI database
4. Results display IP, MAC, vendor, and response time

### 3.2 OUI Fingerprinting

The first 3 bytes (24 bits) of a MAC address are the OUI (Organizationally Unique Identifier), assigned by IEEE to hardware vendors. arp-scan uses this to identify device manufacturers:

| OUI Prefix | Vendor |
|------------|--------|
| 00:50:56 | VMware |
| 08:00:27 | VirtualBox |
| b8:27:eb | Raspberry Pi |
| 00:1a:2b | Various |
| fc:fb:fb | Facebook |
| 00:16:3e | Xen |

### 3.3 ARP vs Nmap Host Discovery

| Feature | arp-scan | nmap -sn |
|---------|----------|----------|
| Speed | Faster (ARP-only) | Slower (multi-probe) |
| Subnet restriction | Local only | Cross-subnet |
| OUI lookup | Built-in | Separate tool |
| Port scanning | No | Yes |
| Root required | Yes | Only for ARP |

### 3.4 Broadcast vs Unicast

arp-scan sends ARP requests as Ethernet broadcasts (destination ff:ff:ff:ff:ff:ff). All hosts on the subnet receive and process the request.

---

## 4. Command Reference

### Basic Usage

```bash
# Scan a specific subnet
sudo arp-scan -I eth0 192.168.1.0/24

# Scan local network (auto-detect interface and range)
sudo arp-scan -l

# Scan a single host
sudo arp-scan -I eth0 192.168.1.1
```

### Flags

| Flag | Description |
|------|-------------|
| `-I IFACE` | Network interface to use |
| `-l` | Auto-detect local network (interface, range) |
| `-N` | No reverse DNS resolution |
| `-r RATE` | Number of packets per second |
| `-t TIMEOUT` | Response timeout in milliseconds |
| `-d DELAY` | Delay between packets in milliseconds |
| `-b RATE` | Bandwidth limit in bits/sec |
| `-m RATE` | Set packet rate (alias for -r) |
| `-n` | Numeric output only (no DNS) |
| `-o FILE` | Read OUI file from FILE |
| `--localnet` | Scan entire local subnet |
| `--arpspa IP` | Source IP for ARP requests |
| `--retry COUNT` | Number of retries per host |
| `--numeric` | Same as -n |
| `--resolve` | Resolve MAC to vendor name |
| `--ignoredups` | Ignore duplicate responses |
| `--ignoredone` | Skip hosts that don't respond |

---

## 5. Examples

### Basic

**Scan a /24 subnet:**
```bash
sudo arp-scan -I eth0 192.168.1.0/24
```
Output:
```
Interface: eth0, type: EN10MB, MAC: 08:00:27:a1:b2:c3
Starting arp-scan 1.9.2 with 256 hosts on 1 network

192.168.1.1    08:00:27:a1:b2:c3    Cadmus Technology Ltd
192.168.1.50   00:50:56:c0:00:08    VMware, Inc.
192.168.1.100  b8:27:eb:12:34:56    Raspberry Pi Foundation

5 packets received by filter, 0 packets dropped by kernel
Ending arp-scan: 256 hosts scanned in 2.034 seconds (125.86 hosts/sec)
```

**Auto-detect local network:**
```bash
sudo arp-scan -l
```

### Intermediate

**Scan without DNS resolution:**
```bash
sudo arp-scan -I eth0 -N 192.168.1.0/24
```

**Set custom OUI file:**
```bash
sudo arp-scan -I eth0 -o /path/to/oui.txt 192.168.1.0/24
```

**Scan with custom source IP:**
```bash
sudo arp-scan -I eth0 --arpspa 192.168.1.100 192.168.1.0/24
```

**Rate-limited scan:**
```bash
sudo arp-scan -I eth0 -r 10 192.168.1.0/24
```
Sends 10 packets per second to avoid overwhelming the network.

### Advanced

**Scan multiple ranges:**
```bash
sudo arp-scan -I eth0 192.168.1.0/24 192.168.2.0/24
```

**High-speed scan with timeout tuning:**
```bash
sudo arp-scan -I eth0 -r 100 -t 200 192.168.1.0/24
```
100 pps with 200ms timeout per host.

**Pipe results for filtering:**
```bash
sudo arp-scan -I eth0 -N 192.168.1.0/24 | grep -i "vmware"
```

**Compare against known devices:**
```bash
sudo arp-scan -I eth0 -N 192.168.1.0/24 | awk '{print $2}' | sort > scanned.txt
comm -23 known_macs.txt scanned.txt > unknown_devices.txt
```

### Real-World

**Network audit — find all devices:**
```bash
sudo arp-scan -I eth0 -l -N > arp_audit_$(date +%F).txt
```

**Rogue device detection:**
```bash
sudo arp-scan -I eth0 -N 192.168.1.0/24 | while read ip mac vendor; do
    if ! grep -q "$mac" whitelist.txt; then
        echo "UNKNOWN: $ip | $mac | $vendor"
    fi
done
```

**Quick inventory with timestamps:**
```bash
sudo arp-scan -I eth0 -l | tee -a network_inventory.log
```

---

## 6. Advanced

### 6.1 OUI Database Management

**Download latest OUI database:**
```bash
sudo wget -O /usr/share/arp-scan/ieee-oui.txt \
    https://raw.githubusercontent.com/wireshark/wireshark/master/manuf
```

**Use custom OUI file:**
```bash
sudo arp-scan -I eth0 -o /etc/arp-scan/custom-oui.txt 192.168.1.0/24
```

### 6.2 Scripted Automation

**Daily network inventory:**
```bash
#!/bin/bash
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
OUTPUT="inventory_${TIMESTAMP}.txt"

sudo arp-scan -I eth0 -l -N > "$OUTPUT"

# Compare with yesterday
PREV=$(ls -t inventory_*.txt | head -2 | tail -1)
if [ -n "$PREV" ]; then
    diff <(awk '{print $2}' "$PREV" | sort) <(awk '{print $2}' "$OUTPUT" | sort)
fi
```

**CSV export:**
```bash
sudo arp-scan -I eth0 -N 192.168.1.0/24 | awk '{
    gsub(/ /, ",", $0);
    print $0
}' > scan_results.csv
```

### 6.3 Combining with Other Tools

**arp-scan + nmap for detailed scan:**
```bash
sudo arp-scan -I eth0 -N 192.168.1.0/24 | awk '{print $1}' | \
    xargs -I {} nmap -sV -T4 {}
```

**arp-scan + arping for verification:**
```bash
sudo arp-scan -I eth0 -N 192.168.1.0/24 | awk 'NR>2{print $1}' | \
    while read ip; do
        arping -c 1 -f "$ip" && echo "$ip confirmed"
    done
```

---

## 7. Detection and Defense

### 7.1 Detection

**Monitor ARP traffic:**
```bash
sudo tcpdump -i eth0 arp -n -c 100
```

**High ARP volume alerts:**
```bash
sudo tcpdump -i eth0 arp 2>/dev/null | \
    awk '{count[$3]++} END {for (ip in count) if (count[ip]>10) print ip, count[ip], "requests"}'
```

### 7.2 Defense

**ARP rate limiting on switches:**
Configure ARP rate limiting to prevent ARP flood scans from overwhelming the network.

**Dynamic ARP Inspection (DAI):**
Enable DAI on managed switches to validate ARP packets against DHCP snooping bindings.

**Static ARP for critical hosts:**
```bash
sudo arp -s 192.168.1.1 08:00:27:a1:b2:c3
```

**MAC address filtering:**
Use ACLs to restrict which MAC addresses can communicate on the network.

---

## 8. Troubleshooting

### 8.1 Common Issues

**"No interface specified":**
```bash
ip -br link show
sudo arp-scan -I eth0 192.168.1.0/24
```

**"Permission denied":**
```bash
sudo arp-scan -I eth0 192.168.1.0/24
```
arp-scan requires root to send raw packets.

**No responses received:**
- Verify interface is up: `ip link show eth0`
- Check subnet matches: `ip addr show eth0`
- Try `-l` flag for auto-detection

**OUI shows "Unknown":**
Update the OUI database:
```bash
sudo wget -O /usr/share/arp-scan/ieee-oui.txt \
    https://raw.githubusercontent.com/wireshark/wireshark/master/manuf
```

### 8.2 Performance Tuning

**Slow scan on large subnet:**
```bash
sudo arp-scan -I eth0 -r 200 -t 500 10.0.0.0/16
```
Increase rate and timeout for larger subnets.

**Packet loss:**
```bash
sudo arp-scan -I eth0 --retry 3 -r 50 192.168.1.0/24
```
Retry failed hosts with lower rate.

### 8.3 Virtualization Issues

Virtual bridges may require promiscuous mode:
```bash
sudo ip link set eth0 promisc on
sudo arp-scan -I eth0 192.168.1.0/24
```

**Docker networks:**
```bash
# Scan Docker bridge network
sudo arp-scan -I docker0 172.17.0.0/16

# Scan custom Docker network
docker network ls
sudo arp-scan -I br-$(docker network inspect mynet --format '{{.Id}}' | head -c 12) 172.18.0.0/16
```

**KVM virtual networks:**
```bash
# List KVM networks
virsh net-list

# Scan KVM default network
sudo arp-scan -I virbr0 192.168.122.0/24
```

### 8.4 Security Considerations

**ARP scanning detection:**
arp-scan generates ARP broadcasts that are visible on the network. IDS systems may flag high volumes of ARP requests from a single source.

**Rate limiting detection:**
```bash
# Monitor for ARP floods
sudo tcpdump -i eth0 arp -n -c 100 | \
    awk '{count[$3]++} END {for (ip in count) if (count[ip]>20) print "FLOOD from", ip, count[ip], "packets"}'
```

**ARP spoofing via arp-scan:**
arp-scan with `--arpspa` can spoof the source IP in ARP requests. This is useful for testing but can be exploited for ARP cache poisoning.

**Defensive ARP monitoring:**
```bash
# Install arpwatch for continuous monitoring
sudo apt install arpwatch
sudo arpwatch -i eth0 -e admin@company.com
```

### 8.5 Output Processing

**JSON export (using jq):**
```bash
sudo arp-scan -I eth0 -N 192.168.1.0/24 | awk 'NR>2{print "{\"ip\":\""$1"\",\"mac\":\""$2"\",\"vendor\":\""$3"\"}"}' | \
    jq -s '.' > scan_results.json
```

**Filter by vendor:**
```bash
sudo arp-scan -I eth0 -N 192.168.1.0/24 | grep -i "vmware"
```

**Sort by IP:**
```bash
sudo arp-scan -I eth0 -N 192.168.1.0/24 | awk 'NR>2{print $0}' | sort -t. -k1,1n -k2,2n -k3,3n -k4,4n
```

### 8.6 Comparison with Similar Tools

| Feature | arp-scan | nmap -sn | netdiscover | arping |
|---------|----------|----------|-------------|--------|
| Speed | Fast | Medium | Medium | Slow |
| OUI lookup | Built-in | External | Built-in | No |
| Passive mode | No | No | Yes | No |
| Subnet scan | Yes | Yes | Yes | No |
| Hostname | Optional | Yes | Yes | No |
| Root required | Yes | Only ARP | Yes | No |

Use arp-scan for rapid Layer 2 discovery with vendor identification. Use nmap when you need hostnames and additional service information.

---

## Resources

- [arp-scan GitHub](https://github.com/royhills/arp-scan)
- [arp-scan man page](https://linux.die.net/man/8/arp-scan)
- [IEEE OUI Database](https://standards-oui.ieee.org/)
- [Kali arp-scan tool page](https://www.kali.org/tools/arp-scan/)
- [ARP Protocol (RFC 826)](https://tools.ietf.org/html/rfc826)
