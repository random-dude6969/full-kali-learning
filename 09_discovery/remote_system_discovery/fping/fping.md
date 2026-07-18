---
title: "fping — Fast Parallel Ping Utility"
description: "Rapidly ping multiple hosts simultaneously, generate IP ranges, and perform network sweeps far faster than standard ping."
mitre_tactic: discovery
mitre_tactic_id: TA0007
mitre_subcategory: remote_system_discovery
categories:
  - network-discovery
  - host-discovery
  - ping
tags:
  - ping
  - parallel
  - sweep
  - host-discovery
  - network
install_command: "sudo apt install fping"
version: "5.2-1"
github_repo: "https://github.com/schweikert/fping"
kali_tools_page: "https://www.kali.org/tools/fping/"
difficulty_level: beginner
requires_root: false
gui_available: false
related_tools:
  - hping3
  - nmap
  - arping
---

# fping — Fast Parallel Ping Utility

## 1. Introduction

**fping** is a high-performance ping utility designed to send ICMP Echo Request packets to multiple hosts simultaneously. Unlike standard ping which contacts one host at a time, fping parallelizes pinging across an entire subnet in seconds.

**Use cases:**
- Rapid host discovery across large networks
- Generate IP ranges for input to other tools
- Monitor host availability in real time
- Scripted network health checks
- Identify live hosts before detailed scanning

**Key advantage:** fping can ping hundreds of hosts in parallel, completing in seconds what would take minutes with standard ping.

**Performance comparison:**
- Standard ping: 1 host at a time, ~1 second per host
- fping: 254 hosts in ~2 seconds (parallel execution)

---

## 2. Installation

**Install via apt:**
```bash
sudo apt install fping
```

**Verify installation:**
```bash
fping -v
```

**Note:** fping works without root for basic pinging. Sending to broadcast addresses or generating ranges requires root.

---

## 3. Core Concepts

### 3.1 Parallel Execution

fping sends ICMP requests to all targets concurrently using non-blocking I/O. It tracks responses and reports results as they arrive.

### 3.2 Range Generation

The `-g` flag generates IP ranges from CIDR notation:
```bash
fping -g 192.168.1.0/24
```
fping internally generates 192.168.1.1 through 192.168.1.254.

### 3.3 Output Modes

| Mode | Flag | Description |
|------|------|-------------|
| Alive | `-a` | Print only alive hosts |
| Unreachable | `-u` | Print only unreachable hosts |
| Verbose | `-v` | Verbose output with statistics |
| Summary | `-s` | Print statistics summary |

### 3.4 ICMP vs ARP

fping uses ICMP Echo (Layer 3), so it can ping hosts across subnets. This differs from arping which is limited to local ARP (Layer 2).

---

## 4. Command Reference

### Basic Usage

```bash
# Ping a single host
fping 192.168.1.1

# Ping multiple hosts
fping 192.168.1.1 192.168.1.2 192.168.1.3

# Generate range and ping
fping -g 192.168.1.0/24

# Show only alive hosts
fping -a -g 192.168.1.0/24
```

### Flags

| Flag | Description |
|------|-------------|
| `-g GENERATE` | Generate target list from CIDR or start-end |
| `-f FILE` | Read targets from file (one per line) |
| `-a` | Show alive hosts only |
| `-u` | Show unreachable hosts only |
| `-c COUNT` | Number of pings per target |
| `-t MS` | Timeout per target in milliseconds |
| `-p MS` | Period (delay between pings) in milliseconds |
| `-s` | Print statistics summary |
| `-S IP` | Source IP address |
| `-b MIN` | Minimum data bytes |
| `-B MAC` | Broadcast MAC address |
| `-n` | Do not resolve hostnames |
| `-d` | Enable DNS lookups |
| `-r COUNT` | Retry count |
| `-i MS` | Interval between pings |
| `-q` | Quiet mode |
| `-v` | Verbose output |

---

## 5. Examples

### Basic

**Ping a subnet:**
```bash
fping -g 192.168.1.0/24
```
Output:
```
192.168.1.1 is alive
192.168.1.50 is alive
192.166.1.100 is alive
192.168.1.2 is unreachable
192.168.1.3 is unreachable
```

**Show only alive hosts:**
```bash
fping -a -g 192.168.1.0/24
```

**Ping from file:**
```bash
fping -f targets.txt
```

### Intermediate

**Count and timeout:**
```bash
fping -a -c 3 -t 2000 -g 192.168.1.0/24
```
Sends 3 pings per host with 2-second timeout.

**Statistics summary:**
```bash
fping -a -s -g 192.168.1.0/24
```
Output:
```
192.168.1.1 is alive
192.168.1.50 is alive

       2 targets
       2 alive
       0 unreachable
     0.00 ms (min round trip timing)
     1.23 ms (avg round trip timing)
     2.46 ms (max round trip timing)
```

**No DNS resolution:**
```bash
fping -n -a -g 192.168.1.0/24
```

### Advanced

**Custom source IP:**
```bash
sudo fping -S 192.168.1.100 -a -g 192.168.1.0/24
```

**Range by start-end:**
```bash
fping -g 192.168.1.1 192.168.1.254
```

**Multiple pings with interval:**
```bash
fping -a -c 5 -i 100 -g 192.168.1.0/24
```
5 pings per host with 100ms interval.

**Pipe output for further processing:**
```bash
fping -a -g 192.168.1.0/24 | xargs -I {} nmap -sV -T4 {}
```

### Real-World

**Network inventory script:**
```bash
#!/bin/bash
SUBNET="$1"
OUTPUT="alive_$(date +%F).txt"
fping -a -s -g "$SUBNET" > "$OUTPUT" 2>&1
echo "Results saved to $OUTPUT"
cat "$OUTPUT"
```

**Continuous monitoring:**
```bash
while true; do
    echo "=== $(date) ==="
    fping -a -g 192.168.1.0/24
    sleep 60
done
```

**Pre-scan host discovery:**
```bash
# Find alive hosts, then scan them
ALIVE=$(fping -a -g 192.168.1.0/24)
echo "$ALIVE" | xargs -I {} nmap -sV -T4 -p- {}
```

---

## 6. Advanced

### 6.1 Performance Tuning

**Aggressive timing:**
```bash
fping -a -t 500 -r 1 -i 10 -g 192.168.1.0/24
```
500ms timeout, 1 retry, 10ms interval — fast but may miss slow hosts.

**Conservative timing:**
```bash
fping -a -t 5000 -r 3 -i 500 -g 192.168.1.0/24
```
5-second timeout, 3 retries, 500ms interval — thorough but slow.

### 6.2 Input Methods

**File input (one host per line):**
```bash
cat > targets.txt << 'EOF'
192.168.1.1
192.168.1.50
10.0.0.1
10.0.0.2
EOF
fping -a -f targets.txt
```

**Mixed input:**
```bash
fping -a 192.168.1.1 192.168.1.2 -g 192.168.2.0/24
```

### 6.3 Scripting Patterns

**Parallel nmap scan:**
```bash
fping -a -g 192.168.1.0/24 | xargs -P 10 -I {} nmap -sV -T4 {}
```
Parallel nmap scans of 10 hosts simultaneously.

**CSV export:**
```bash
fping -a -g 192.168.1.0/24 | while read ip; do
    echo "$ip,$(dig +short -x $ip | head -1)"
done > hosts.csv
```

**Web server discovery:**
```bash
fping -a -g 192.168.1.0/24 | xargs -I {} sh -c 'curl -s -o /dev/null -w "%{http_code} %s\n" http://{} 2>/dev/null'
```

### 6.4 Combining with Other Tools

**fping + arping for dual-layer discovery:**
```bash
# ICMP discovery
fping -a -g 192.168.1.0/24 > icmp_alive.txt

# ARP discovery (Layer 2)
for ip in $(seq 1 254); do
    arping -c 1 -f 192.168.1.$ip 2>/dev/null && echo "192.168.1.$ip" >> arp_alive.txt
done

# Compare lists
comm -23 arp_alive.txt icmp_alive.txt  # ARP-only (ICMP blocked)
comm -13 arp_alive.txt icmp_alive.txt  # ICMP-only (ARP filtered)
```

---

## 7. Detection and Defense

### 7.1 Detection

**Monitor ICMP traffic:**
```bash
sudo tcpdump -i eth0 icmp -n
```

**Detect fping patterns:**
```bash
# fping sends rapid ICMP requests — look for burst patterns
sudo tcpdump -i eth0 icmp -n -l | awk '{count[$3]++} END {for (ip in count) if (count[ip]>10) print ip, count[ip], "packets"}'
```

**ICMP rate anomaly:**
```bash
# Count ICMP packets per second
sudo tcpdump -i eth0 icmp -n -l | awk '{print $1}' | cut -d. -f1-4 | sort | uniq -c
```

### 7.2 Defense

**Rate limit ICMP:**
```bash
# iptables rate limiting
sudo iptables -A INPUT -p icmp --icmp-type echo-request -m limit --limit 1/s -j ACCEPT
sudo iptables -A INPUT -p icmp --icmp-type echo-request -j DROP
```

**Block ICMP entirely:**
```bash
sudo iptables -A INPUT -p icmp --icmp-type echo-request -j DROP
```

**ICMP echo-reply only:**
```bash
sudo iptables -A INPUT -p icmp --icmp-type echo-reply -j ACCEPT
sudo iptables -A INPUT -p icmp --icmp-type echo-request -j DROP
```

**Network monitoring:**
Deploy IDS/IPS to detect ICMP sweep patterns indicative of reconnaissance.

---

## 8. Troubleshooting

### 8.1 Common Issues

**"Operation not permitted":**
```bash
sudo fping -a -g 192.168.1.0/24
```
Some operations (broadcast, source IP) require root.

**"No such device":**
```bash
ip -br link show
fping -S 192.168.1.100 -a -g 192.168.1.0/24
```
Specify source IP if multiple interfaces exist.

**All hosts unreachable:**
```bash
# Test single host
fping 192.168.1.1

# Check network
ping -c 1 192.168.1.1
ip route get 192.168.1.1
```

**No output:**
- Check if targets are valid IPs
- Verify network connectivity
- Try with `-v` for verbose output

### 8.2 Performance Issues

**Scan too slow:**
```bash
fping -a -t 1000 -r 0 -i 10 -g 192.168.1.0/24
```
Reduce timeout and retries, increase interval.

**Scan too fast (missing hosts):**
```bash
fping -a -t 5000 -r 3 -i 100 -g 192.168.1.0/24
```
Increase timeout and retries.

### 8.3 Virtualization Issues

Virtual networks may not pass ICMP between virtual interfaces. Ensure proper network configuration and firewall rules.

---

## Resources

- [fping GitHub](https://github.com/schweikert/fping)
- [fping man page](https://linux.die.net/man/8/fping)
- [Kali fping tool page](https://www.kali.org/tools/fping/)
- [ICMP Protocol (RFC 792)](https://tools.ietf.org/html/rfc792)
- [fping performance benchmarks](https://github.com/schweikert/fping/wiki)
