---
title: "p0f — Passive OS Fingerprinting Tool"
description: "Identify remote operating systems, network distance, NAT, firewalls, and load balancers by passively analyzing TCP/IP packets with zero network footprint."
mitre_tactic: discovery
mitre_tactic_id: TA0007
mitre_subcategory: remote_system_discovery
categories:
  - os-fingerprinting
  - passive-analysis
  - network-reconnaissance
tags:
  - os-detection
  - passive
  - fingerprinting
  - tcp
  - ttl
install_command: "sudo apt install p0f"
version: "4.03-1"
github_repo: "https://github.com/p0f/p0f"
kali_tools_page: "https://www.kali.org/tools/p0f/"
difficulty_level: intermediate
requires_root: true
gui_available: false
related_tools:
  - nmap -O
  - SinFP
  - xprobe2
---

# p0f — Passive OS Fingerprinting Tool

## 1. Introduction

**p0f** is a passive OS fingerprinting tool that identifies the operating system of network hosts by analyzing TCP/IP packet characteristics. Unlike active scanners, p0f generates zero network traffic — it only observes existing packets.

**Use cases:**
- Identify operating systems without sending any packets
- Detect network distance (hop count) to remote hosts
- Discover NAT devices, firewalls, and load balancers
- Monitor network connections in real time
- Forensic analysis of network traffic captures

**Key advantage:** p0f is completely invisible — it never transmits anything, making it impossible to detect by IDS/IPS systems.

**Fingerprinting signatures:**
- **TTL**: Initial TTL and observed TTL indicate OS
- **Window size**: TCP window size varies by OS and options
- **DF bit**: Don't Fragment flag (set by most modern OSes)
- **SYN options**: TCP options in SYN packets (MSS, window scaling, etc.)
- **Packet timing**: Inter-packet delays

---

## 2. Installation

**Install via apt:**
```bash
sudo apt install p0f
```

**Verify installation:**
```bash
p0f -v
```

**Permissions:** p0f requires root to capture raw packets from network interfaces.

---

## 3. Core Concepts

### 3.1 Passive Fingerprinting

p0f analyzes packets without sending anything. It examines:

| Field | Significance |
|-------|--------------|
| TTL | Initial TTL indicates OS family (64=Linux, 128=Windows, 255=Network) |
| Window size | Varies by OS and TCP options |
| DF bit | Don't Fragment — most OSes set this |
| MSS | Maximum Segment Size |
| Window scaling | TCP window scaling factor |
| SACK | Selective Acknowledgement support |
| Timestamp | TCP timestamp option |

### 3.2 OS Detection Profiles

p0f maintains signature databases:
- **p0f.fp**: Main fingerprint database (Linux, Windows, macOS, etc.)
- **p0f.os**: OS identification database
- **p0f.app**: Application identification database

### 3.3 Network Characteristics

p0f detects:
- **Distance**: TTL-based hop count estimation
- **NAT**: Network Address Translation detection
- **Firewall**: Packet filtering detection
- **Load balancer**: Server-side load balancing
- **Ethernet vs Wi-Fi**: Link type detection

### 3.4 Signature Format

```
signature = ver:flags:opt:mss:win:ttl:datalen:options
```

Example signature:
```
8:64:0:*:mss*20,10:0
```
- Version 8, Linux (TTL 64), any MSS, window multiplier 20, 10 options

---

## 4. Command Reference

### Basic Usage

```bash
# Listen on interface
sudo p0f -i eth0

# Read from pcap file
sudo p0f -r capture.pcap

# Filter by port
sudo p0f -i eth0 -P 80
```

### Flags

| Flag | Description |
|------|-------------|
| `-i IFACE` | Network interface to monitor |
| `-f FILE` | Custom fingerprint file |
| `-r FILE` | Read from pcap file |
| `-o FILE` | Write output to file |
| `-p` | Promiscuous mode |
| `-l` | Log to stdout |
| `-d` | Run as daemon |
| `--lookup` | Look up a signature |
| `-S` | Show SYN packets only |
| `-P PORT` | Filter by port |
| `-Q` | Quiet mode |
| `-t SECS` | Timeout for connections |
| `-v` | Verbose output |
| `-V` | Print version |
| `-h` | Print help |

---

## 5. Examples

### Basic

**Listen on interface:**
```bash
sudo p0f -i eth0
```
Output:
```
p0f: passive OS fingerprinting v4.03 by Michal Zalewski <lcamtuf@coredump.cx>

  eth0: link=ethernet mtu=1500 snaplen=65535

  .-----------------------------------------------------------------
  | client          | server          | app          | OS
  |-----------------|-----------------|--------------|------------------
  | 192.168.1.50:   | 93.184.216.34:  |              | Linux 4.15-5.x
  |   45231         |   443           |              | (13 hops, eth)
  '-----------------------------------------------------------------
```

**Read from pcap:**
```bash
sudo p0f -r captured.pcap
```

### Intermediate

**Filter SYN packets only:**
```bash
sudo p0f -i eth0 -S
```

**Filter specific port:**
```bash
sudo p0f -i eth0 -P 443
```

**Verbose output:**
```bash
sudo p0f -i eth0 -v
```

**Quiet mode:**
```bash
sudo p0f -i eth0 -Q
```
Only prints fingerprint matches, no banner.

### Advanced

**Custom fingerprint database:**
```bash
sudo p0f -i eth0 -f /path/to/custom.fp
```

**Daemon mode with logging:**
```bash
sudo p0f -i eth0 -d -o /var/log/p0f.log
```

**Analyze specific connection:**
```bash
sudo p0f -r capture.pcap -S -P 443
```

### Real-World

**Monitor incoming connections:**
```bash
sudo p0f -i eth0 -S -P 443 -v 2>&1 | tee connection_monitor.log
```

**Identify OS of web visitors:**
```bash
sudo p0f -i eth0 -P 80 -S -v
```

**Forensic analysis of capture:**
```bash
sudo p0f -r suspicious_traffic.pcap -v -o p0f_analysis.txt
```

**Continuous monitoring with timestamp:**
```bash
while true; do
    echo "=== $(date) ===" >> p0f_monitor.log
    sudo timeout 300 p0f -i eth0 -Q >> p0f_monitor.log
done
```

---

## 6. Advanced

### 6.1 Fingerprint Analysis

**TTL-based OS identification:**
| Observed TTL | Likely Initial TTL | OS Family |
|--------------|-------------------|-----------|
| 64 | 64 | Linux/Unix |
| 128 | 128 | Windows |
| 255 | 255 | Network device (Cisco, etc.) |
| 30-32 | 32 | Windows (older) |

**Window size patterns:**
| OS | Typical Window Size |
|----|-------------------|
| Linux | 29200, 58400 |
| Windows | 8192, 65535 |
| macOS | 65535 |
| FreeBSD | 65535 |

### 6.2 pcap Analysis

**Capture with tcpdump, analyze with p0f:**
```bash
# Capture
sudo tcpdump -i eth0 -w capture.pcap -c 10000

# Analyze
sudo p0f -r capture.pcap -v
```

**Analyze specific traffic:**
```bash
# Capture HTTP traffic
sudo tcpdump -i eth0 port 80 -w http.pcap -c 5000

# Identify OS of HTTP clients
sudo p0f -r http.pcap -P 80 -S
```

### 6.3 Scripted Analysis

**Extract unique OS fingerprints:**
```bash
sudo p0f -r capture.pcap -Q 2>&1 | grep -oP 'OS: \K.*' | sort -u
```

**Count connections by OS:**
```bash
sudo p0f -r capture.pcap -Q 2>&1 | grep -oP 'OS: \K.*' | sort | uniq -c | sort -rn
```

**Monitor web server visitors:**
```bash
#!/bin/bash
sudo p0f -i eth0 -P 443 -S -Q 2>&1 | while read line; do
    if echo "$line" | grep -q "client"; then
        echo "$(date): $line" >> visitor_log.txt
    fi
done
```

### 6.4 Integration with Other Tools

**p0f + nmap for comparison:**
```bash
# Passive identification
sudo p0f -i eth0 -P 443 -S -Q > passive_os.txt

# Active verification
nmap -O -sV 192.168.1.50 > active_os.txt

# Compare results
diff passive_os.txt active_os.txt
```

**p0f + Wireshark:**
```bash
# Capture for Wireshark + p0f analysis
sudo tcpdump -i eth0 -w full_capture.pcap
# Open in Wireshark for visual analysis
# Run p0f for OS identification
sudo p0f -r full_capture.pcap -v
```

---

## 7. Detection and Defense

### 7.1 Detection

**p0f is passive — it cannot be detected.** This is its primary advantage. However, you can detect the network activity p0f analyzes.

**Monitor what p0f would see:**
```bash
sudo tcpdump -i eth0 -n 'tcp[tcpflags] & tcp-syn != 0' -c 100
```

### 7.2 Defense

**OS obfuscation:**
- Modify TTL values: `sysctl -w net.ipv4.ip_default_ttl=128`
- Change TCP window sizes: `sysctl -w net.ipv4.tcp_window_scaling=0`
- Disable TCP options: Custom kernel compilation

**TTL manipulation:**
```bash
# Linux
sudo sysctl -w net.ipv4.ip_default_ttl=128

# Per-connection TTL (iptables)
sudo iptables -t mangle -A POSTROUTING -p tcp --sport 80 -j TTL --ttl-set 128
```

**TCP option modification:**
```bash
# Disable window scaling
sudo sysctl -w net.ipv4.tcp_window_scaling=0

# Disable timestamps
sudo sysctl -w net.ipv4.tcp_timestamps=0
```

**Firewall-based obfuscation:**
```bash
# Randomize TTL
sudo iptables -t mangle -A POSTROUTING -p tcp -j TTL --ttl-inc $((RANDOM % 10))
```

---

## 8. Troubleshooting

### 8.1 Common Issues

**"No suitable interface":**
```bash
ip -br link show
sudo p0f -i eth0
```
Verify the interface name and that it exists.

**"Permission denied":**
```bash
sudo p0f -i eth0
```
p0f requires root for packet capture.

**No fingerprints appearing:**
- Verify traffic exists: `sudo tcpdump -i eth0 -c 10`
- Check interface is up: `ip link show eth0`
- Try with `-v` for verbose output

**pcap file errors:**
```bash
# Verify pcap is valid
tcpdump -r capture.pcap -c 5
sudo p0f -r capture.pcap
```

### 8.2 Performance Issues

**High CPU usage:**
```bash
sudo p0f -i eth0 -Q  # Quiet mode
```
Reduce output to lower CPU.

**Missed packets:**
```bash
sudo p0f -i eth0 -p  # Promiscuous mode
```
Enable promiscuous mode to capture all traffic.

### 8.3 Analysis Issues

**Ambiguous OS identification:**
- p0f may return multiple possible OSes
- Use `-v` for detailed signature information
- Compare with nmap -O for active verification

**Wrong OS detection:**
- OS may be spoofing TCP options
- NAT may change TTL values
- Virtualization may alter fingerprints

---

## Resources

- [p0f GitHub](https://github.com/p0f/p0f)
- [p0f documentation](https://github.com/p0f/p0f/blob/master/README)
- [Kali p0f tool page](https://www.kali.org/tools/p0f/)
- [TCP/IP Fingerprinting Techniques](https://en.wikipedia.org/wiki/TCP/IP_stack_fingerprinting)
- [p0f signature database](https://github.com/p0f/p0f/tree/master/fingerprints)
