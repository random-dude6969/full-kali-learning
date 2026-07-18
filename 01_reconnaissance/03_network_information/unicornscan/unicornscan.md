---
tool_name: "unicornscan"
display_name: "Unicornscan"
mitre_tactic: "reconnaissance"
mitre_tactic_id: "TA0043"
mitre_subcategory: "network_information"
install_command: "sudo apt install unicornscan"
version: "0.4.7"
github_repo: "https://github.com/gontie/unicornscan"
kali_tools_page: "https://www.kali.org/tools/unicornscan/"
difficulty_level: "intermediate"
requires_root: true
gui_available: false
tags: ["port-scanning", "asynchronous", "syn-scan", "network", "reconnaissance"]
related_tools: ["nmap", "masscan", "zmap"]
---

# Unicornscan

## Chapter 1: Introduction and Overview

### What is Unicornscan?

Unicornscan is a high-performance, asynchronous TCP/UDP port scanner designed for large-scale network reconnaissance. It uses a novel approach to port scanning by leveraging asynchronous I/O and state machine processing, allowing it to scan enormous numbers of ports and hosts far faster than traditional scanners like Nmap.

Unicornscan sends packets at extremely high rates and collects responses asynchronously, making it ideal for mapping large networks quickly. It supports SYN, ACK, and other TCP scan types, as well as UDP scanning.

### History and Background

- **Author:** Gontie (originally developed by security researchers)
- **Language:** C
- **License:** GNU General Public License v2
- **Platform:** Linux (requires raw socket access)
- **Key Innovation:** Asynchronous scanning architecture for extreme speed

Unicornscan was developed as an alternative to Nmap for scenarios where scanning speed is critical. While Nmap focuses on comprehensive service detection and scripting, Unicornscan prioritizes raw scanning throughput.

### When to Use This Tool

- **Large network scans:** Scanning entire /16 or /8 networks quickly
- **Port sweep operations:** Identifying all listening services across thousands of hosts
- **High-speed reconnaissance:** When time is critical and you need fast results
- **Network mapping:** Discovering live hosts and open ports at scale
- **Firewall testing:** Probing firewall rules with various scan types
- **Supplement to Nmap:** Use Unicornscan for speed, then Nmap for detailed service detection

### When NOT to Use This Tool

- Without root privileges (raw socket access required)
- When detailed service version detection is needed (use Nmap instead)
- For stealthy scans (Unicornscan is noisy by design)
- When NSE scripting capabilities are required
- On systems without raw socket support

### Legal and Ethical Considerations

- **Root Access Required:** Unicornscan requires root privileges to craft raw packets
- **Network Impact:** High-speed scans can trigger IDS/IPS alerts and potentially disrupt networks
- **Authorization:** Always obtain written permission before scanning
- **Rate Control:** Use appropriate timing options to avoid network disruption
- **Documentation:** Record all scanning activities for legal compliance

### Key Concepts

- **Asynchronous I/O:** Non-blocking packet sending and receiving for maximum throughput
- **SYN Scan:** Half-open scanning that doesn't complete TCP handshake
- **State Machine:** Processes responses independently of sending
- **Raw Sockets:** Direct packet construction and sending
- **PPS (Packets Per Second):** Metric for scanning speed
- **Fire-and-Forget:** Send packets without waiting for individual responses

---

## Chapter 2: Installation and Setup

### System Requirements

- **Operating System:** Linux (required for raw sockets)
- **Privileges:** Root access mandatory
- **RAM:** 256 MB minimum (1 GB recommended for large scans)
- **CPU:** Multi-core recommended for high-speed scanning
- **Network:** Fast network interface (1 Gbps+ recommended)
- **Kernel:** Linux kernel with raw socket support

### Installation Methods

#### Method 1: APT (Recommended)

```bash
sudo apt update
sudo apt install unicornscan
```

#### Method 2: From Source

```bash
git clone https://github.com/gontie/unicornscan.git
cd unicornscan
./configure
make
sudo make install
```

#### Method 3: Compile with Optimizations

```bash
git clone https://github.com/gontie/unicornscan.git
cd unicornscan
CFLAGS="-O3 -march=native" ./configure
make -j$(nproc)
sudo make install
```

### Configuration

Unicornscan uses command-line options rather than configuration files. Key considerations:

- **Firewall:** Ensure raw socket access is available
- **Kernel Parameters:** Tune network stack for high performance:
```bash
# Increase network buffer sizes
sudo sysctl -w net.core.rmem_max=16777216
sudo sysctl -w net.core.wmem_max=16777216
sudo sysctl -w net.ipv4.tcp_rmem="4096 87380 16777216"
sudo sysctl -w net.ipv4.tcp_wmem="4096 65536 16777216"
```

### Verification

```bash
unicornscan --version
# Output: unicornscan 0.4.7

unicornscan --help
# Displays all available options
```

---

## Chapter 3: Basic Usage

### First Scan

```bash
sudo unicornscan 192.168.1.1
```

**Output:** Scans default TCP ports on the target using SYN scan.

### Essential Commands

#### Simple TCP Scan

```bash
sudo unicornscan 192.168.1.1
```

**Explanation:** Performs a default SYN scan of common TCP ports.

#### Scan Specific Ports

```bash
sudo unicornscan 192.168.1.1:80,443,22
```

**Explanation:** Scans ports 80, 443, and 22 on the target.

#### Scan Port Range

```bash
sudo unicornscan 192.168.1.1:1-1024
```

**Explanation:** Scans TCP ports 1 through 1024.

#### UDP Scan

```bash
sudo unicornscan -U 192.168.1.1:53,161,162
```

**Explanation:** Scans UDP ports 53, 161, and 162.

#### Network Scan

```bash
sudo unicornscan 192.168.1.0/24:80
```

**Explanation:** Scans port 80 on all hosts in the /24 subnet.

#### Verbose Output

```bash
sudo unicornscan -v 192.168.1.1:80,443
```

**Explanation:** Shows detailed scan information including timing.

#### High-Speed Scan

```bash
sudo unicornscan -r 3000 192.168.1.1:1-65535
```

**Explanation:** Scans all TCP ports at 3000 packets per second.

### Complete Flags Reference

#### Scan Types

| Flag | Description | Example |
|------|-------------|---------|
| (default) | SYN scan | `unicornscan 192.168.1.1` |
| `-sS` | SYN scan (explicit) | `unicornscan -sS 192.168.1.1` |
| `-sT` | TCP connect scan | `unicornscan -sT 192.168.1.1` |
| `-sA` | ACK scan | `unicornscan -sA 192.168.1.1` |
| `-sU` | UDP scan | `unicornscan -U 192.168.1.1` |
| `-sI` | Idle scan | `unicornscan -sI zombie:port target` |

#### Target Specification

| Flag | Description | Example |
|------|-------------|---------|
| `-p` | Port specification | `unicornscan -p 80,443 target` |
| `-m` | Module (scan type) | `unicornscan -mT target:80` |
| `-r` | Packets per second | `unicornscan -r 3000 target` |
| `-c` | Count of packets | `unicornscan -c 100 target` |
| `-l` | Log file | `unicornscan -l scan.log target` |
| `-L` | Log level | `unicornscan -L 3 target` |

#### Port Specification

| Format | Description | Example |
|--------|-------------|---------|
| `target:port` | Single port | `unicornscan 192.168.1.1:80` |
| `target:p1,p2,p3` | Multiple ports | `unicornscan 192.168.1.1:80,443,22` |
| `target:p1-p2` | Port range | `unicornscan 192.168.1.1:1-1024` |
| `target:all` | All ports | `unicornscan 192.168.1.1:all` |
| `target:*` | All ports (alt) | `unicornscan 192.168.1.1:*` |

#### Timing Options

| Flag | Description | Example |
|------|-------------|---------|
| `-r` | Packets per second | `unicornscan -r 5000 target` |
| `-d` | Delay between probes | `unicornscan -d 100 target` |
| `-w` | Wait time for responses | `unicornscan -w 5000 target` |
| `-W` | Window size | `unicornscan -W 1024 target` |
| `-T` | Timeout (ms) | `unicornscan -T 3000 target` |

#### Output Options

| Flag | Description | Example |
|------|-------------|---------|
| `-v` | Verbose | `unicornscan -v target` |
| `-vv` | Very verbose | `unicornscan -vv target` |
| `-l` | Log to file | `unicornscan -l output.log target` |
| `-L` | Log level (0-5) | `unicornscan -L 3 target` |
| `-o` | Output format | `unicornscan -o text target` |

#### Miscellaneous

| Flag | Description | Example |
|------|-------------|---------|
| `-h` | Help | `unicornscan -h` |
| `-V` | Version | `unicornscan -V` |
| `-n` | No DNS resolution | `unicornscan -n target` |
| `-D` | Debug mode | `unicornscan -D target` |

---

## Chapter 4: Intermediate Usage

### Bash Scripting with Unicornscan

#### Network Sweep Script

```bash
#!/bin/bash
# network_sweep.sh - Sweep entire network for open ports

NETWORK=$1
PORTS=${2:-"22,80,443,8080,3306"}
OUTPUT_FILE="sweep_$(date +%Y%m%d).txt"

echo "[+] Starting network sweep: $NETWORK"
echo "[+] Scanning ports: $PORTS"
echo "[+] Results: $OUTPUT_FILE"

# Perform sweep
sudo unicornscan -r 1000 -mT "$NETWORK:${PORTS}" 2>/dev/null | \
    tee "$OUTPUT_FILE"

# Extract live hosts
grep -oE "([0-9]{1,3}\.){3}[0-9]{1,3}" "$OUTPUT_FILE" | \
    sort -u > "live_hosts_${NETWORK}.txt"

echo "[+] Sweep complete!"
echo "[+] Live hosts: $(wc -l < "live_hosts_${NETWORK}.txt")"
```

#### High-Speed Full Port Scan

```bash
#!/bin/bash
# full_port_scan.sh - Scan all 65535 TCP ports quickly

TARGET=$1
OUTPUT="fullport_${TARGET}.txt"

echo "[+] Scanning all ports on $TARGET at 5000 PPS..."

# Scan all TCP ports
sudo unicornscan -r 5000 -mT "$TARGET:*" -l "$OUTPUT" 2>/dev/null

# Extract open ports
grep "open" "$OUTPUT" | awk '{print $2}' | sort -n > "open_ports_${TARGET}.txt"

echo "[+] Open ports found:"
cat "open_ports_${TARGET}.txt"
```

#### UDP Service Discovery

```bash
#!/bin/bash
# udp_discover.sh - Discover UDP services

TARGET=$1
COMMON_UDP_PORTS="53,67,68,69,123,135,137,138,161,162,389,500,514,520,623,1900,4500,5353"

echo "[+] Scanning common UDP ports on $TARGET..."

sudo unicornscan -r 500 -mU "$TARGET:${COMMON_UDP_PORTS}" 2>/dev/null | \
    tee "udp_results_${TARGET}.txt"

echo "[+] UDP scan complete!"
```

### Tool Chaining

#### Unicornscan → Nmap Pipeline

```bash
# Phase 1: Fast port discovery with Unicornscan
echo "[*] Phase 1: Fast port discovery..."
sudo unicornscan -r 3000 -mT "192.168.1.0/24:21,22,23,25,53,80,110,135,139,143,443,993,995,1723,3306,3389,5900,8080" 2>/dev/null | \
    grep "open" | awk '{print $1}' | sort -u > live_hosts.txt

# Phase 2: Detailed service detection with Nmap
echo "[*] Phase 2: Service detection..."
nmap -sV -sC -iL live_hosts.txt -oN nmap_results.txt
```

#### Unicornscan → Metasploit Pipeline

```bash
# Discover hosts and ports
sudo unicornscan -r 1000 -mT "10.0.0.0/24:21,22,80,443,445,3389" 2>/dev/null | \
    grep "open" | awk '{print $1}' | sort -u > targets.txt

# Import into Metasploit
echo "use auxiliary/scanner/portscan/tcp" > msf_scan.rc
echo "set RHOSTS file:targets.txt" >> msf_scan.rc
echo "set PORTS 21,22,80,443,445,3389" >> msf_scan.rc
echo "run" >> msf_scan.rc
echo "exit" >> msf_scan.rc

msfconsole -r msf_scan.rc
```

### Output Processing

#### Parse Scan Results

```bash
# Run scan and parse results
sudo unicornscan -r 1000 192.168.1.1:1-1024 2>/dev/null | \
    awk '/open/ {print $1, $2}' | \
    sort -t: -k2 -n > open_ports.txt

# Display formatted results
echo "=== Open Ports ==="
while read line; do
    port=$(echo "$line" | cut -d: -f2)
    service=$(echo "$line" | awk '{print $2}')
    echo "Port $port: $service"
done < open_ports.txt
```

#### CSV Export

```bash
# Export results to CSV
echo "IP,Port,Service,State" > scan_results.csv
sudo unicornscan -r 1000 192.168.1.1:22,80,443 2>/dev/null | \
    awk '/open/ {gsub(":",","); print $1","$2","$3","$4}' >> scan_results.csv
```

---

## Chapter 5: Advanced Usage

### Custom Scan Configurations

#### Optimized Large-Scale Scan

```bash
# Optimize kernel for high-speed scanning
sudo sysctl -w net.core.rmem_max=16777216
sudo sysctl -w net.core.wmem_max=16777216
sudo sysctl -w net.ipv4.tcp_rmem="4096 87380 16777216"
sudo sysctl -w net.ipv4.tcp_wmem="4096 65536 16777216"
sudo sysctl -w net.ipv4.tcp_window_scaling=1
sudo sysctl -w net.ipv4.tcp_timestamps=1
sudo sysctl -w net.ipv4.tcp_sack=1

# Perform optimized scan
sudo unicornscan -r 10000 -w 2000 -mT "10.0.0.0/8:80" 2>/dev/null
```

#### Stealth Scan Configuration

```bash
# Low-and-slow scan to avoid detection
sudo unicornscan -r 10 -d 100 -mT "192.168.1.1:1-1024" 2>/dev/null
```

#### Fragmented Packets

```bash
# Use fragmented packets to evade IDS
sudo unicornscan -f -r 500 -mT "192.168.1.1:1-1024" 2>/dev/null
```

### Integration with Pentest Frameworks

#### Custom Scan Profile

```bash
#!/bin/bash
# custom_scan.sh - Custom scan with multiple techniques

TARGET=$1

echo "[+] Custom scan profile for $TARGET"

# Phase 1: Quick SYN scan
echo "[*] Phase 1: Quick SYN scan..."
sudo unicornscan -r 5000 -mT "$TARGET:21,22,23,25,53,80,110,143,443,993,995,3306,3389,5432,8080,8443" 2>/dev/null > syn_results.txt

# Phase 2: UDP scan for critical ports
echo "[*] Phase 2: UDP scan..."
sudo unicornscan -r 500 -mU "$TARGET:53,123,161,162,500,514,1900" 2>/dev/null > udp_results.txt

# Phase 3: Full TCP port scan (high speed)
echo "[*] Phase 3: Full port scan..."
sudo unicornscan -r 10000 -mT "$TARGET:*" 2>/dev/null > fullport_results.txt

# Phase 4: Combine and analyze
echo "[+] Combined results:"
cat syn_results.txt udp_results.txt fullport_results.txt | \
    grep "open" | sort -u > all_open.txt

cat all_open.txt
```

#### Network Mapping Script

```bash
#!/bin/bash
# netmap.sh - Map entire network infrastructure

NETWORK=$1
OUTPUT_DIR="netmap_$(date +%Y%m%d)"

mkdir -p "$OUTPUT_DIR"

echo "[+] Mapping network: $NETWORK"

# Scan common ports across network
sudo unicornscan -r 5000 -mT "$NETWORK:22,80,443,445,3389" 2>/dev/null > "$OUTPUT_DIR/live_hosts.txt"

# Extract unique hosts
grep -oE "([0-9]{1,3}\.){3}[0-9]{1,3}" "$OUTPUT_DIR/live_hosts.txt" | \
    sort -u > "$OUTPUT_DIR/host_list.txt"

# Per-host detailed scan
while read host; do
    echo "[*] Detailed scan: $host"
    sudo unicornscan -r 1000 -mT "$host:1-1024" 2>/dev/null > "$OUTPUT_DIR/${host}_detail.txt"
done < "$OUTPUT_DIR/host_list.txt"

echo "[+] Network mapping complete!"
echo "[+] Results in: $OUTPUT_DIR/"
```

### Performance Tuning

#### Kernel Parameter Optimization

```bash
#!/bin/bash
# optimize_network.sh - Optimize kernel for high-speed scanning

echo "[+] Optimizing kernel parameters..."

# Increase buffer sizes
sudo sysctl -w net.core.rmem_default=262144
sudo sysctl -w net.core.rmem_max=16777216
sudo sysctl -w net.core.wmem_default=262144
sudo sysctl -w net.core.wmem_max=16777216

# TCP optimizations
sudo sysctl -w net.ipv4.tcp_rmem="4096 87380 16777216"
sudo sysctl -w net.ipv4.tcp_wmem="4096 65536 16777216"
sudo sysctl -w net.ipv4.tcp_window_scaling=1
sudo sysctl -w net.ipv4.tcp_timestamps=1
sudo sysctl -w net.ipv4.tcp_sack=1
sudo sysctl -w net.ipv4.tcp_no_metrics_save=1

# Connection tracking
sudo sysctl -w net.netfilter.nf_conntrack_max=262144

# Network queue
sudo sysctl -w net.core.netdev_max_backlog=5000

echo "[+] Kernel optimized for high-speed scanning"
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: Large Corporate Network Audit

**Objective:** Quickly map a /16 corporate network to identify all live hosts and open ports.

```bash
#!/bin/bash
# corporate_audit.sh - Large-scale network audit

NETWORK="10.0.0.0/16"
OUTPUT_DIR="corp_audit_$(date +%Y%m%d)"
PORTS="21,22,23,25,53,80,110,135,139,143,443,445,993,995,1433,3306,3389,5432,8080,8444"

mkdir -p "$OUTPUT_DIR"

echo "[+] Starting corporate network audit: $NETWORK"
echo "[+] Date: $(date)"

# Phase 1: Fast host discovery
echo "[*] Phase 1: Host discovery..."
sudo unicornscan -r 10000 -mT "$NETWORK:80,443" 2>/dev/null | \
    grep "open" | awk '{print $1}' | sort -u > "$OUTPUT_DIR/live_hosts.txt"

echo "[+] Live hosts found: $(wc -l < "$OUTPUT_DIR/live_hosts.txt")"

# Phase 2: Port scan on live hosts
echo "[*] Phase 2: Port scanning..."
while read host; do
    sudo unicornscan -r 5000 -mT "$host:${PORTS}" 2>/dev/null >> "$OUTPUT_DIR/port_results.txt"
done < "$OUTPUT_DIR/live_hosts.txt"

# Phase 3: Analysis
echo "[*] Phase 3: Analysis..."
cat "$OUTPUT_DIR/port_results.txt" | \
    grep "open" | \
    awk '{print $1, $2, $3}' | \
    sort -t: -k2 -n > "$OUTPUT_DIR/summary.txt"

echo "[+] Audit complete!"
echo "[+] Summary: $OUTPUT_DIR/summary.txt"
```

### Scenario 2: CTF Challenge Reconnaissance

**Objective:** Quickly identify services on a CTF target machine.

```bash
# Fast service discovery
sudo unicornscan -r 5000 -mT "10.10.10.10:*" 2>/dev/null | \
    grep "open" | sort -t: -k2 -n

# Focus on interesting ports
sudo unicornscan -r 3000 -mT "10.10.10.10:21,22,25,53,80,110,139,143,443,993,995,1433,3306,3389,5432,8080,8443,8888,9090" 2>/dev/null

# UDP services
sudo unicornscan -r 500 -mU "10.10.10.10:53,69,123,161,162,500,514,1900" 2>/dev/null
```

### Scenario 3: Bug Bounty Network Discovery

**Objective:** Map the external infrastructure of a bug bounty target.

```bash
#!/bin/bash
# bugbounty_recon.sh - Network discovery for bug bounty

TARGET_DOMAIN=$1

# Resolve domain to IP
TARGET_IP=$(dig +short "$TARGET_DOMAIN" | head -1)

echo "[+] Target: $TARGET_DOMAIN ($TARGET_IP)"

# Quick port scan
echo "[*] Scanning common web ports..."
sudo unicornscan -r 3000 -mT "$TARGET_IP:80,443,8080,8443,8888,9090,3000,5000,8000,9000" 2>/dev/null | \
    tee "ports_${TARGET_DOMAIN}.txt"

# Expand to adjacent IPs if possible
SUBNET=$(echo "$TARGET_IP" | cut -d. -f1-3).0/24
echo "[*] Scanning adjacent subnet: $SUBNET"
sudo unicornscan -r 5000 -mT "$SUBNET:80,443" 2>/dev/null | \
    grep "open" | awk '{print $1}' | sort -u > "adjacent_hosts.txt"

echo "[+] Reconnaissance complete!"
```

---

## Chapter 7: Detection and Defense

### Detection Signatures

#### Network-Based Detection

```bash
# Snort rule to detect Unicornscan
alert tcp $HOME_NET any -> $EXTERNAL_NET any (
    msg:"SCAN Unicornscan SYN scan detected";
    flags:S,12;
    threshold:type both,track by_src, count 100, seconds 10;
    sid:1000002;
    rev:1;
)

# Suricata rule
alert tcp any any -> any any (
    msg:"ET SCAN Unicornscan detected";
    tcp.flags.syn == 1;
    tcp.flags.ack == 0;
    threshold: type both, track by_src, count 500, seconds 10;
    sid:2000001;
    rev:1;
)
```

#### Log Indicators

- **High connection rate:** Hundreds of SYN packets from single source
- **Port sweep pattern:** Sequential port access across multiple hosts
- **No completed handshakes:** SYN packets without ACK responses
- **Timing patterns:** Regular intervals between probes

### Mitigation Strategies

#### Firewall Configuration

```bash
# iptables rate limiting
iptables -A INPUT -p tcp --syn -m limit --limit 100/minute --limit-burst 200 -j ACCEPT
iptables -A INPUT -p tcp --syn -j DROP

# nftables rate limiting
nft add rule inet filter input tcp flags syn meter syn-flood { limit rate 100/minute burst 200 packets } accept
nft add rule inet filter input tcp flags syn drop
```

#### IDS/IPS Tuning

```yaml
# Suricata configuration for scan detection
- detect-engine:
    - profile: medium
    - custom-values:
        toclient-src-groups: 2
        toclient-dst-groups: 2
        toserver-src-groups: 2
        toserver-dst-groups: 2
    - sgh-mpm-context: auto
```

#### Network Architecture

- Deploy network segmentation to limit scan impact
- Use VLANs to isolate sensitive systems
- Implement micro-segmentation for critical assets
- Deploy honeypots to detect scanning activity

---

## Chapter 8: Troubleshooting

### Common Errors

#### Error: "Operation not permitted"

**Cause:** Running without root privileges.

**Solution:**
```bash
sudo unicornscan 192.168.1.1:80
```

#### Error: "Network is unreachable"

**Cause:** Network interface misconfiguration.

**Solution:**
```bash
# Check network interface
ip addr show

# Specify interface
sudo unicornscan -e eth0 192.168.1.1:80

# Check routing
ip route show
```

#### Error: "No route to host"

**Cause:** Target is unreachable or blocked by firewall.

**Solution:**
```bash
# Test connectivity first
ping -c 3 192.168.1.1

# Try different scan type
sudo unicornscan -sA 192.168.1.1:80

# Check firewall rules
sudo iptables -L -n
```

#### Error: "Connection timed out"

**Cause:** Network latency or high packet loss.

**Solution:**
```bash
# Reduce scanning speed
sudo unicornscan -r 100 -w 5000 192.168.1.1:80

# Check network latency
ping -c 10 192.168.1.1
```

### Performance Optimization

- **Reduce PPS on slow networks:** Start with `-r 1000` and increase gradually
- **Use appropriate timeouts:** Set `-w` based on network latency
- **Limit port ranges:** Scan specific ports rather than all 65535
- **Monitor system resources:** Use `top` to check CPU/memory usage
- **Check network interface:** Ensure NIC can handle the packet rate

### FAQ

**Q: Is Unicornscan better than Nmap?**
A: Unicornscan is faster for large-scale port scanning, but Nmap provides more comprehensive service detection and scripting. Use Unicornscan for speed, Nmap for detail.

**Q: Can I use Unicornscan on Windows?**
A: No, Unicornscan requires Linux raw socket support. Use WSL2 on Windows if needed.

**Q: Why does Unicornscan require root?**
A: Raw socket access for crafting custom TCP/IP packets requires root privileges on Linux.

**Q: How do I avoid detection while scanning?**
A: Use lower rates (`-r 100`), add delays (`-d 100`), and use VPN/proxy. However, in authorized tests, detection is expected.

---

## Resources

### Official Documentation

- **GitHub Repository:** https://github.com/gontie/unicornscan
- **Kali Linux Tools Page:** https://www.kali.org/tools/unicornscan/
- **Man Pages:** `man unicornscan`

### Books

- *Network Security Assessment* by Chris McNab
- *Penetration Testing* by Georgia Weidman
- *The Art of Network Security Monitoring* by Chris Sanders

### Video Tutorials

- **Unicornscan vs Nmap** by LiveOverflow
- **Network Scanning Techniques** by HackerSploit
- **High-Speed Port Scanning** by NetworkChuck

### Practice Labs

- **HackTheBox:** Network scanning challenges
- **TryHackMe:** Network security rooms
- **VulnHub:** Network reconnaissance exercises
- **PentesterLab:** Network assessment labs
