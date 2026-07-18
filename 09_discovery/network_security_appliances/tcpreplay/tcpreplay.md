---
title: "tcpreplay"
description: "Suite of tools for replaying previously captured network traffic from pcap files to test IDS/IPS, firewalls, and network infrastructure under realistic load conditions."
categories: ["Network Security Appliances"]
tags: ["traffic-replay", "pcap", "ids", "ips", "firewall-testing", "network-forensics"]
mitre_tactic: "discovery"
mitre_tactic_id: "TA0007"
mitre_subcategory: "network_security_appliances"
install_command: "sudo apt install tcpreplay"
version: "4.4.4"
github_repo: "https://github.com/appneta/tcpreplay"
kali_tools_page: "https://www.kali.org/tools/tcpreplay/"
difficulty_level: "intermediate"
requires_root: true
gui_available: false
related_tools: ["tcpdump", "wireshark", "hping3"]
---

# tcpreplay

## Chapter 1: Introduction & Overview

### What is tcpreplay?

tcpreplay is a comprehensive suite of tools designed to replay previously captured network traffic (stored in pcap files) onto a live network. It consists of three primary components:

1. **tcpreplay** — Replays captured traffic at configurable speeds onto a network interface
2. **tcprewrite** — Modifies packets in pcap files before replay (MAC addresses, IP addresses, ports, etc.)
3. **tcpprep** — Pre-processes pcap files to create cache files that tell tcpreplay which packets to send on which interface

The suite is indispensable for testing network security appliances — firewalls, IDS/IPS systems, load balancers, and WAFs — by replaying real traffic patterns to validate detection rules, measure performance, and identify configuration gaps.

### History & Background

tcpreplay was originally created by Aaron Turner in 2001. It has become the de facto standard for pcap-based traffic replay in the security and networking industries.

**Key milestones:**
- **2001**: Initial release by Aaron Turner
- **2005**: Merged with `tcprewrite` (originally a separate tool)
- **2010**: AppNeta acquired and maintained the project
- **2020s**: Continued maintenance with support for modern pcap formats (pcapng)

**License**: BSD-3-Clause

**Platform support**: Linux (primary), macOS, FreeBSD

### When to Use This Tool

tcpreplay is appropriate for:

- **IDS/IPS testing**: Validate that intrusion detection systems correctly identify malicious traffic patterns
- **Firewall rule testing**: Replay attack traffic to verify firewall blocking rules
- **Network performance benchmarking**: Test how infrastructure handles specific traffic patterns
- **Security appliance validation**: Confirm that new security devices correctly process traffic
- **Incident response**: Replay suspicious traffic to understand attack behavior in controlled environments
- **Regression testing**: Ensure network changes don't break existing functionality
- **Load testing**: Generate realistic traffic loads for capacity planning

tcpreplay is **not** appropriate for:
- Generating new traffic (use hping3 or Scapy)
- Real-time packet capture (use tcpdump or Wireshark)
- Active exploitation (use Metasploit)

### Key Concepts

**Pcap Files**: Binary files containing captured network packets. Created by tools like tcpdump, Wireshark, or tshark. Pcap files preserve full packet headers and optionally payloads.

**Cache Files**: Pre-processed files created by tcpprep that map packets to output interfaces. Essential for bridged or dual-interface replay.

**Packet Editing**: Modifying packet fields (MAC, IP, port) in pcap files before replay. This allows replaying captured traffic against different targets.

**Replay Modes**: tcpreplay supports multiple speed control methods:
- **Top speed**: Replay as fast as possible
- **Packet-per-second**: Fixed rate
- **Mbps**: Fixed bandwidth
- **Multiplier**: Multiply original inter-packet timing

## Chapter 2: Installation & Setup

### System Requirements

**Minimum requirements:**
- **OS**: Kali Linux, Ubuntu, Debian, or any modern Linux distribution
- **RAM**: 512 MB (more for large pcap files)
- **Disk space**: 10 MB for tcpreplay; additional for pcap files
- **Network**: At least one Ethernet interface; two for bridged mode
- **Privileges**: Root access required for raw packet injection

**Dependencies:**
- `libpcap` — packet capture library
- `libnet` — packet construction library
- `libdumbnet` — network utilities

### Installation Methods

**Method 1: Kali Linux repositories (recommended)**

```bash
sudo apt update
sudo apt install tcpreplay
```

**Method 2: Build from source**

```bash
# Install build dependencies
sudo apt install build-essential libpcap-dev libnet1-dev libdumbnet-dev

# Download and compile
wget https://github.com/appneta/tcpreplay/releases/download/v4.4.4/tcpreplay-4.4.4.tar.gz
tar -xzf tcpreplay-4.4.4.tar.gz
cd tcpreplay-4.4.4

# Configure, build, install
./configure
make
sudo make install
```

**Method 3: Install all subtools**

```bash
# tcpreplay includes tcpprep and tcprewrite
# Verify all three are installed
which tcpreplay tcpprep tcprewrite
```

### Configuration

tcpreplay is configured entirely via command-line flags. No configuration file is needed.

**Interface preparation:**
```bash
# Identify available interfaces
ip link show

# Set interface to promiscuous mode (required for some replay modes)
sudo ip link set eth0 promisc on

# Disable offloading features that interfere with replay
sudo ethtool -K eth0 tso off gso off gro off
```

### Verifying Installation

```bash
# Check version
tcpreplay --version

# Check all subtools
tcpreplay --help
tcprewrite --help
tcpprep --help

# Quick test with a small pcap
sudo tcpreplay -i eth0 --topspeed test.pcap
```

## Chapter 3: Core Concepts & Architecture

### How It Works

tcpreplay's operation involves three distinct phases:

**Phase 1: Capture (external)**
Network traffic is captured using tcpdump, Wireshark, or similar tools and saved as a pcap file. This step happens outside tcpreplay.

**Phase 2: Pre-processing (tcpprep)**
The pcap file is analyzed to determine packet classification. tcpprep creates a cache file that maps each packet to an output interface (for dual-interface mode) or marks it for replay.

**Phase 3: Replay (tcpreplay)**
Packets are read from the pcap file (and cache file if applicable) and injected onto the network interface at the specified speed. tcpreplay maintains packet timing relationships based on the selected speed mode.

**Optional Phase 4: Packet Editing (tcprewrite)**
Before replay, tcprewrite can modify packet headers — changing source/destination IPs, MAC addresses, port numbers, or removing specific fields. This is essential when replaying traffic captured in one environment against a different target.

### Protocols & Standards

**Pcap Format (libpcap)**:
- Standard binary format for packet capture
- Stores packet headers and payloads with timestamps
- Pcapng is the newer variant with more metadata

**Ethernet/IP Layer**:
- tcpreplay operates at Layer 2-3, injecting raw Ethernet frames
- Supports IPv4 and IPv6 (with limitations)

**Transport Layer**:
- TCP, UDP, ICMP, and other IP protocol numbers
- Full packet payloads are preserved during replay

### Network Architecture

```
[Pcap File] ──> [tcprewrite] ──> [tcpreplay] ──> [Network Interface]
                     │                  │
                     │                  ├── Single-interface mode
                     │                  │   (all packets to one NIC)
                     │                  │
                     │                  └── Dual-interface mode
                     │                      (requires tcpprep cache)
                     │                      packets split by source
                     │
                 [tcpprep]
                     │
                     └── Creates cache file mapping
                         packets to interfaces
```

**Single-interface mode**: All packets are replayed through one interface. Simplest setup, used for testing a single target.

**Dual-interface mode**: Packets are split between two interfaces based on source IP. Used for testing inline devices (firewalls, IDS/IPS) where traffic must flow through the device under test.

## Chapter 4: Command Reference

### tcpreplay Flags

| Flag | Long Form | Description |
|------|-----------|-------------|
| `-i <iface>` | `--intf=<iface>` | Output interface |
| `-I <iface>` | `--intf2=<iface>` | Second interface (dual mode) |
| `--topspeed` | | Replay at maximum speed |
| `--pps=<num>` | `--packets-per-second` | Packets per second |
| `--mbps=<num>` | `--mbps` | Replay at specified Mbps |
| `--multiplier=<num>` | `--multiplier` | Multiply original timing |
| `-z <pps>` | `--pps` | Packets per second |
| `-M` | `--mbps` | Megabits per second |
| `-x <num>` | `--multiplier` | Timing multiplier |
| `--loop=<num>` | `--loop` | Repeat pcap N times |
| `--loopdelay=<ms>` | `--loopdelay` | Delay between loops (ms) |
| `-c <cache>` | `--cachefile` | Cache file from tcpprep |
| `-C <file>` | `--cachefile` | Create cache file (for tcpprep) |
| `-D` | `--dual` | Dual interface mode |
| `-d` | `--deterministic` | Deterministic packet ordering |
| `-a` | `--accurate` | Maintain accurate timing |
| `-A` | `--accurate` | Same as -a |
| `--draft` | | Use draft mode |
| `-t` | `--notimestamps` | Don't maintain timestamps |
| `-v` | `--verbose` | Verbose output |
| `-V` | `--version` | Display version |
| `-h` | `--help` | Display help |

### tcprewrite Flags

| Flag | Long Form | Description |
|------|-----------|-------------|
| `--enet-dmac=<mac>` | | Rewrite destination MAC |
| `--enet-smac=<mac>` | | Rewrite source MAC |
| `--endpoints=<ip>` | | Rewrite IP endpoints |
| `--ipmap=<map>` | | IP address mapping |
| `--portmap=<map>` | | TCP/UDP port mapping |
| `-i <input>` | `--infile` | Input pcap file |
| `-o <output>` | `--outfile` | Output pcap file |
| `-c <cache>` | `--cachefile` | Cache file |

### tcpprep Flags

| Flag | Long Form | Description |
|------|-----------|-------------|
| `-i <input>` | `--infile` | Input pcap file |
| `-o <output>` | `--outfile` | Cache file output |
| `-c <cidr>` | `--cidr` | Client IP CIDR |
| `-s <cidr>` | `--server` | Server IP CIDR |
| `-p` | `--port` | Use port to determine client/server |
| `-r` | `--regex` | Regex for server matching |
| `-e` | `--reverse` | Reverse client/server |
| `-P` | `--proto` | Protocol-based classification |

## Chapter 5: Usage Examples

### Basic Usage

**Replay a pcap at top speed:**

```bash
sudo tcpreplay -i eth0 --topspeed capture.pcap
```

Replays all packets from `capture.pcap` through interface `eth0` as fast as possible.

**Replay at original timing:**

```bash
sudo tcpreplay -i eth0 -a capture.pcap
```

Maintains the original inter-packet timing from the capture.

**Replay at specific packet rate:**

```bash
sudo tcpreplay -i eth0 --pps=1000 capture.pcap
```

Sends 1000 packets per second regardless of original timing.

### Intermediate Usage

**Edit pcap before replay:**

```bash
# Change destination IP to test target
sudo tcprewrite --endpoints=192.168.0.100 -i original.pcap -o modified.pcap

# Replay modified pcap
sudo tcpreplay -i eth0 --topspeed modified.pcap
```

**Loop replay multiple times:**

```bash
# Replay 10 times with 1 second delay between loops
sudo tcpreplay -i eth0 --loop=10 --loopdelay=1000 capture.pcap
```

**Replay at specific bandwidth:**

```bash
# Replay at 100 Mbps
sudo tcpreplay -i eth0 --mbps=100 capture.pcap

# Replay at 2x original speed
sudo tcpreplay -i eth0 --multiplier=2 capture.pcap
```

### Advanced Usage

**Dual-interface mode for inline device testing:**

```bash
# Step 1: Create cache file
sudo tcpprep -i capture.pcap -o cache.cache -c 192.168.0.0/24

# Step 2: Replay through both interfaces
sudo tcpreplay -i eth0 -I eth1 -c cache.cache --topspeed capture.pcap
```

This replays traffic through an inline firewall/IDS positioned between eth0 and eth1.

**Create cache with port-based classification:**

```bash
# Classify by port (clients use high ports, servers use low ports)
sudo tcpprep -i capture.pcap -o cache.cache -p

# Replay
sudo tcpreplay -i eth0 -I eth1 -c cache.cache --topspeed capture.pcap
```

**Rewrite multiple packet fields:**

```bash
# Change source MAC, destination MAC, and IP addresses
sudo tcprewrite \
  --enet-smac=00:11:22:33:44:55 \
  --enet-dmac=aa:bb:cc:dd:ee:ff \
  --endpoints=192.168.0.200 \
  -i original.pcap \
  -o rewritten.pcap

sudo tcpreplay -i eth0 --topspeed rewritten.pcap
```

### Real-World Scenarios

**Scenario 1: Test IDS/IPS detection rules**

```bash
# Replay attack traffic against IDS
sudo tcpprep -i attack_traffic.pcap -o attack.cache -c 10.0.0.0/8
sudo tcpreplay -i eth0 -I eth1 -c attack.cache --topspeed attack_traffic.pcap

# Check IDS alerts
tail -f /var/log/suricata/eve.json | jq 'select(.event_type=="alert")'
```

**Scenario 2: Validate firewall blocking rules**

```bash
# Replay known malicious traffic
sudo tcprewrite --endpoints=192.168.0.50 -i malicious.pcap -o test.pcap
sudo tcpreplay -i eth0 --topspeed test.pcap

# Verify packets are blocked
sudo tcpdump -i eth0 -nn host 192.168.0.50
```

**Scenario 3: Load test a web application firewall**

```bash
# Replay mixed legitimate and attack traffic
sudo tcprewrite --endpoints=192.168.0.100 -i mixed_traffic.pcap -o waf_test.pcap
sudo tcpreplay -i eth0 --pps=500 waf_test.pcap

# Monitor WAF logs
tail -f /var/log/waf/access.log
```

**Scenario 4: Baseline comparison after network changes**

```bash
# Capture baseline traffic
sudo tcpdump -i eth0 -w baseline.pcap -c 10000

# After changes, replay and compare
sudo tcpreplay -i eth0 --topspeed baseline.pcap

# Capture during replay
sudo tcpdump -i eth0 -w after_changes.pcap -c 10000

# Compare with Wireshark or tshark
tshark -r baseline.pcap -q -z io,stat,1
tshark -r after_changes.pcap -q -z io,stat,1
```

## Chapter 6: Advanced Techniques

### Automation & Scripting

**Batch replay script for multiple pcaps:**

```bash
#!/bin/bash
# replay_batch.sh — replay multiple pcap files sequentially
INTERFACE="eth0"
PCAP_DIR="/opt/pcaps"
RESULTS_DIR="/opt/results"

mkdir -p "$RESULTS_DIR"

for pcap in "$PCAP_DIR"/*.pcap; do
    filename=$(basename "$pcap" .pcap)
    echo "=== Replaying $filename ==="

    # Rewrite for target
    sudo tcprewrite --endpoints=192.168.0.100 -i "$pcap" -o "/tmp/$filename.pcap"

    # Replay
    sudo tcpreplay -i "$INTERFACE" --topspeed "/tmp/$filename.pcap" \
        > "$RESULTS_DIR/$filename.log" 2>&1

    echo "Completed: $filename"
done
```

**Automated IDS testing:**

```bash
#!/bin/bash
# test_ids.sh — replay attacks and check IDS detection
INTERFACE="eth0"
TARGET="192.168.0.50"
PCAP="attack.pcap"

# Clear previous alerts
sudo truncate -s 0 /var/log/suricata/eve.json

# Start replay in background
sudo tcprewrite --endpoints=$TARGET -i $PCAP -o /tmp/test.pcap
sudo tcpreplay -i $INTERFACE --topspeed /tmp/test.pcap &

# Wait for replay to finish
wait

# Check detection
ALERTS=$(jq 'select(.event_type=="alert")' /var/log/suricata/eve.json | wc -l)
echo "IDS detected $ALERTS alerts during replay"

# List detected signatures
jq -r 'select(.event_type=="alert") | .alert.signature' /var/log/suricata/eve.json | sort -u
```

### Chaining with Other Tools

**Capture → Edit → Replay workflow:**

```bash
# Step 1: Capture attack traffic
sudo tcpdump -i eth0 -w attack_raw.pcap host 192.168.0.100

# Step 2: Edit pcap for target environment
sudo tcprewrite --endpoints=192.168.1.50 --enet-smac=de:ad:be:ef:00:01 -i attack_raw.pcap -o attack_ready.pcap

# Step 3: Replay against test target
sudo tcpreplay -i eth0 --topspeed attack_ready.pcap

# Step 4: Verify with tcpdump
sudo tcpdump -i eth0 -nn host 192.168.1.50 -w result.pcap
```

**tcpreplay + hping3 for combined testing:**

```bash
# Replay captured traffic
sudo tcpreplay -i eth0 --pps=100 captured.pcap &

# Simultaneously generate additional probes
sudo hping3 -S -p 80 -c 100 -i u10000 192.168.0.100

# Wait for both to complete
wait
```

**tcpreplay + iptables for testing rate limiting:**

```bash
# Set up rate limiting rule
sudo iptables -A INPUT -p tcp --dport 80 -m limit --limit 10/s -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 80 -j DROP

# Replay traffic hitting port 80
sudo tcprewrite --portmap=80:80 -i traffic.pcap -o /tmp/test.pcap
sudo tcpreplay -i eth0 --pps=100 /tmp/test.pcap

# Check how many packets were accepted
sudo iptables -L -v -n | grep ACCEPT
```

### Custom Configurations

**Modify pcap for different network segment:**

```bash
# Rewrite source and destination IPs for lab environment
sudo tcprewrite \
  --ipmap=10.0.0.0/8:192.168.1.0/24 \
  -i production_capture.pcap \
  -o lab_ready.pcap
```

**Remove specific packet types:**

```bash
# Use editcap (part of Wireshark) to filter before replay
editcap -f tcp -Y "tcp.port == 80" large_capture.pcap http_only.pcap
sudo tcpreplay -i eth0 --topspeed http_only.pcap
```

### Performance Tuning

**Optimize for large pcap files:**

```bash
# Use read-buf to increase buffer size for large files
sudo tcpreplay -i eth0 --topspeed --read-buf=1048576 large_capture.pcap

# Pre-process with tcpprep for faster replay
sudo tcpprep -i large.pcap -o large.cache -c 192.168.0.0/24
sudo tcpreplay -i eth0 -c large.cache --topspeed large.pcap
```

**Disable NIC offloading for accurate replay:**

```bash
# Disable TCP segmentation offload
sudo ethtool -K eth0 tso off

# Disable generic segmentation offload
sudo ethtool -K eth0 gso off

# Disable generic receive offload
sudo ethtool -K eth0 gro off

# Disable large receive offload
sudo ethtool -K eth0 lro off
```

## Chapter 7: Detection & Defense

### How to Detect

tcpreplay-generated traffic has detectable characteristics:

1. **Packet Timing Anomalies**: Replayed traffic may have unrealistic timing — either too fast (topspeed mode) or too regular (multiplier mode). Normal traffic has variable inter-packet gaps.

2. **MAC Address Mismatch**: tcprewrite-modified packets may have MAC addresses that don't match the sender's real MAC. ARP inspection can detect this.

3. **Replay Fingerprints**: The same pcap file replayed multiple times produces identical packet sequences. This repetition is detectable.

4. **Interface Promiscuous Mode**: The sending interface must be in promiscuous mode, which is detectable via ARP monitoring.

### Defensive Measures

**MAC address filtering:**
```bash
# Block replayed traffic with mismatched MAC/IP
sudo ebtables -A FORWARD -p arp --arp-ip-src 192.168.0.100 -j DROP
```

**ARP inspection:**
```bash
# Dynamic ARP Inspection on switches
# Cisco: ip arp inspection vlan 10
```

**Rate limiting:**
```bash
# Limit new connections per source
sudo iptables -A INPUT -p tcp --syn -m limit --limit 50/s -j ACCEPT
sudo iptables -A INPUT -p tcp --syn -j DROP
```

### IDS/IPS Signatures

**Snort rule for replay detection:**

```
# Detect identical packet sequences (replay indicator)
alert tcp any any -> any any (msg:"Possible Traffic Replay"; \
  threshold:type both, track by_src, count 3, seconds 10; \
  sid:1000003; rev:1;)
```

**Suricata rule:**

```
alert http any any -> any any (msg:"HTTP Replay Detected"; \
  http.header; content:"X-Request-Id"; \
  threshold:type limit, track by_src, count 1, seconds 60; \
  sid:2000001; rev:1;)
```

### Log Analysis

```bash
# Monitor for duplicate packets
sudo tcpdump -i eth0 -nn -c 1000 | awk '{print $3}' | sort | uniq -d

# Check for unusual timing patterns
sudo tcpdump -i eth0 -tt -nn | awk '{print $1}' | awk -F. 'NR>1{diff=$1-prev; if(diff<0.001) print "Suspiciously fast:", $0} {prev=$1}'

# Analyze pcap replay patterns
tcpreplay --listnics  # verify interfaces
sudo tcpreplay -i eth0 -t -v --topspeed test.pcap  # verbose replay with timestamps
```

## Chapter 8: Troubleshooting & FAQ

### Common Issues

**Problem: "No such device" error**

Cause: Specified interface does not exist or is down.

Solution:
```bash
# List available interfaces
ip link show

# Bring interface up
sudo ip link set eth0 up

# Verify interface is active
ip addr show eth0
```

**Problem: Packets sent but not received**

Cause: NIC offloading features interfering with raw packet injection.

Solution:
```bash
# Disable offloading
sudo ethtool -K eth0 tso off gso off gro off lro off

# Set interface to promiscuous mode
sudo ip link set eth0 promisc on

# Verify with tcpdump
sudo tcpdump -i eth0 -c 5
```

**Problem: "Permission denied" when running**

Cause: tcpreplay requires root privileges for raw packet injection.

Solution:
```bash
sudo tcpreplay -i eth0 --topspeed capture.pcap
```

**Problem: Segmentation fault with large pcap files**

Cause: Insufficient memory or buffer too small.

Solution:
```bash
# Increase read buffer
sudo tcpreplay -i eth0 --topspeed --read-buf=4194304 large.pcap

# Split large pcap into smaller files
editcap -c 100000 large.pcap split_
for f in split_*.pcap; do
    sudo tcpreplay -i eth0 --topspeed "$f"
done
```

**Problem: Timing is inaccurate**

Cause: Default mode sacrifices timing accuracy for speed.

Solution:
```bash
# Use accurate timing mode
sudo tcpreplay -i eth0 -a capture.pcap

# Or use multiplier with accurate flag
sudo tcpreplay -i eth0 -a -x 2 capture.pcap
```

### FAQ

**Q: Can tcpreplay replay encrypted traffic?**
A: tcpreplay replays packets as-is, including encrypted payloads. It does not decrypt or modify encrypted content. Use tcprewrite to modify unencrypted headers only.

**Q: Does tcpreplay work with pcapng files?**
A: Yes. Modern versions of tcpreplay support pcapng format. Convert with: `editcap -F pcap old.pcap new.pcapng` if needed.

**Q: Can I replay traffic to a different subnet?**
A: Yes. Use tcprewrite to modify IP addresses: `sudo tcprewrite --endpoints=NEW_TARGET_IP -i original.pcap -o modified.pcap`

**Q: What's the maximum replay speed?**
A: `--topspeed` replays as fast as the system and NIC allow. Typical speeds are 100K-1M packets per second depending on hardware and packet size.

**Q: Can tcpreplay handle VLAN-tagged traffic?**
A: Yes, tcpreplay supports 802.1Q VLAN tags. Use tcprewrite to modify VLAN IDs: `--vlan=new_vlan_id`

**Q: Does tcpreplay preserve TCP state?**
A: No. tcpreplay injects raw packets without maintaining TCP state. TCP handshakes in the pcap may be replayed as-is, which can cause RST responses. Use `--dual` mode with a stateful device in the path for proper TCP flow.

**Q: Can I replay only specific packets from a pcap?**
A: Use editcap or tshark to filter before replay: `tshark -r input.pcap -Y "tcp.port == 80" -w http_only.pcap`

## Resources

### Official Documentation
- **tcpreplay manual**: `man tcpreplay`, `man tcprewrite`, `man tcpprep`
- **GitHub repository**: https://github.com/appneta/tcpreplay
- **Kali tools page**: https://www.kali.org/tools/tcpreplay/

### Online Resources
- **AppNeta tcpreplay guide**: https://www.appneta.com/
- **Wireshark Wiki — capinfos**: https://wiki.wireshark.org/
- **tcpdump documentation**: https://www.tcpdump.org/

### Related Tools
- **tcpdump** — Command-line packet capture
- **Wireshark** — GUI packet analyzer
- **hping3** — Packet crafting and sending
- **editcap** — pcap file editing (Wireshark suite)
- **Scapy** — Python packet manipulation library
- **ngrep** — Network grep for packet content matching
