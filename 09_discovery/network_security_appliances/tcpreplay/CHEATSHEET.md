# tcpreplay — Cheatsheet

## Quick Reference

**Purpose**: Replay captured network traffic from pcap files to test IDS/IPS, firewalls, and network infrastructure.

**Requires**: Root privileges, libpcap, libnet

**Install**: `sudo apt install tcpreplay`

**Subtools**: `tcpreplay`, `tcprewrite`, `tcpprep`

---

## Core Syntax

```bash
sudo tcpreplay [options] <pcap_file>
sudo tcprewrite [options] -i <input.pcap> -o <output.pcap>
sudo tcpprep [options] -i <input.pcap> -o <cache_file>
```

---

## Essential Commands

### Basic Replay

```bash
# Replay at top speed
sudo tcpreplay -i eth0 --topspeed capture.pcap

# Replay at original timing
sudo tcpreplay -i eth0 -a capture.pcap

# Replay at 1000 packets/sec
sudo tcpreplay -i eth0 --pps=1000 capture.pcap

# Replay at specific bandwidth
sudo tcpreplay -i eth0 --mbps=100 capture.pcap

# Replay at 2x original speed
sudo tcpreplay -i eth0 --multiplier=2 capture.pcap
```

### Packet Editing

```bash
# Change destination IP
sudo tcprewrite --endpoints=192.168.0.100 -i original.pcap -o modified.pcap

# Change source MAC
sudo tcprewrite --enet-smac=00:11:22:33:44:55 -i in.pcap -o out.pcap

# Change destination MAC
sudo tcprewrite --enet-dmac=aa:bb:cc:dd:ee:ff -i in.pcap -o out.pcap

# Rewrite both MACs and IP
sudo tcprewrite \
  --enet-smac=00:11:22:33:44:55 \
  --enet-dmac=aa:bb:cc:dd:ee:ff \
  --endpoints=192.168.0.200 \
  -i in.pcap -o out.pcap
```

### Cache Creation (for dual-interface mode)

```bash
# Create cache with CIDR-based classification
sudo tcpprep -i capture.pcap -o cache.cache -c 192.168.0.0/24

# Create cache with port-based classification
sudo tcpprep -i capture.pcap -o cache.cache -p

# Create cache with regex-based server identification
sudo tcpprep -i capture.pcap -o cache.cache -r "^10\.0\.0\."
```

### Dual-Interface Replay

```bash
# Replay through inline device (firewall/IDS between eth0 and eth1)
sudo tcpreplay -i eth0 -I eth1 -c cache.cache --topspeed capture.pcap
```

---

## Flag Reference

### tcpreplay

| Flag | Description | Example |
|------|-------------|---------|
| `-i <iface>` | Output interface | `-i eth0` |
| `-I <iface>` | Second interface (dual mode) | `-I eth1` |
| `--topspeed` | Maximum speed | `--topspeed` |
| `--pps=<num>` | Packets per second | `--pps=1000` |
| `--mbps=<num>` | Megabits per second | `--mbps=100` |
| `--multiplier=<num>` | Timing multiplier | `--multiplier=2` |
| `--loop=<num>` | Repeat N times | `--loop=10` |
| `--loopdelay=<ms>` | Delay between loops | `--loopdelay=1000` |
| `-c <cache>` | Cache file | `-c cache.cache` |
| `-D` | Dual interface mode | `-D` |
| `-a` | Accurate timing | `-a` |
| `-t` | No timestamps | `-t` |
| `-v` | Verbose | `-v` |
| `-d` | Deterministic ordering | `-d` |

### tcprewrite

| Flag | Description | Example |
|------|-------------|---------|
| `--endpoints=<ip>` | Rewrite IP endpoints | `--endpoints=192.168.0.100` |
| `--enet-smac=<mac>` | Rewrite source MAC | `--enet-smac=00:11:22:33:44:55` |
| `--enet-dmac=<mac>` | Rewrite destination MAC | `--enet-dmac=aa:bb:cc:dd:ee:ff` |
| `--ipmap=<map>` | IP address mapping | `--ipmap=10.0.0.0/8:192.168.1.0/24` |
| `--portmap=<map>` | Port mapping | `--portmap=8080:80` |
| `-i <file>` | Input file | `-i input.pcap` |
| `-o <file>` | Output file | `-o output.pcap` |

### tcpprep

| Flag | Description | Example |
|------|-------------|---------|
| `-c <cidr>` | Client CIDR | `-c 192.168.0.0/24` |
| `-s <cidr>` | Server CIDR | `-s 10.0.0.0/8` |
| `-p` | Port-based classification | `-p` |
| `-r <regex>` | Regex for server | `-r "^10\.0"` |
| `-e` | Reverse classification | `-e` |
| `-i <file>` | Input file | `-i input.pcap` |
| `-o <file>` | Output file | `-o cache.cache` |

---

## Common Scenarios

### IDS/IPS Testing

```bash
# Replay attack traffic through inline IDS
sudo tcpprep -i attacks.pcap -o attacks.cache -c 10.0.0.0/8
sudo tcpreplay -i eth0 -I eth1 -c attacks.cache --topspeed attacks.pcap

# Check alerts
tail -f /var/log/suricata/eve.json | jq 'select(.event_type=="alert")'
```

### Firewall Validation

```bash
# Rewrite for test target and replay
sudo tcprewrite --endpoints=192.168.0.50 -i malicious.pcap -o test.pcap
sudo tcpreplay -i eth0 --topspeed test.pcap
```

### Load Testing

```bash
# Generate sustained load
sudo tcpreplay -i eth0 --pps=500 --loop=100 --loopdelay=1000 traffic.pcap
```

### Lab Environment Replay

```bash
# Rewrite for lab network
sudo tcprewrite --ipmap=10.0.0.0/8:192.168.1.0/24 \
  --enet-smac=de:ad:be:ef:00:01 \
  -i prod_capture.pcap -o lab.pcap

sudo tcpreplay -i eth0 --topspeed lab.pcap
```

---

## Workflow

```bash
# Step 1: Capture traffic
sudo tcpdump -i eth0 -w capture.pcap -c 10000

# Step 2: Edit for target environment
sudo tcprewrite --endpoints=192.168.0.100 -i capture.pcap -o ready.pcap

# Step 3: (Optional) Create cache for dual mode
sudo tcpprep -i ready.pcap -o cache.cache -c 192.168.0.0/24

# Step 4: Replay
sudo tcpreplay -i eth0 --topspeed ready.pcap

# Step 5: Verify
sudo tcpdump -i eth0 -nn host 192.168.0.100 -c 10
```

---

## Troubleshooting

```bash
# "No such device" → check interfaces
ip link show

# "Permission denied" → run as root
sudo tcpreplay -i eth0 --topspeed capture.pcap

# Packets not received → disable offloading
sudo ethtool -K eth0 tso off gso off gro off

# Segfault on large files → increase buffer
sudo tcpreplay -i eth0 --topspeed --read-buf=4194304 large.pcap

# Timing inaccurate → use accurate mode
sudo tcpreplay -i eth0 -a capture.pcap

# Convert pcapng to pcap
editcap -F pcap input.pcapng output.pcap
```

---

## One-Liners

```bash
# Quick replay
sudo tcpreplay -i eth0 --topspeed capture.pcap

# Replay with original timing
sudo tcpreplay -i eth0 -a capture.pcap

# Edit and replay
sudo tcprewrite --endpoints=192.168.0.100 -i in.pcap -o out.pcap && sudo tcpreplay -i eth0 --topspeed out.pcap

# Loop 5 times
sudo tcpreplay -i eth0 --loop=5 --topspeed capture.pcap

# Create cache and dual replay
sudo tcpprep -i cap.pcap -o c.cache -c 192.168.0.0/24 && sudo tcpreplay -i eth0 -I eth1 -c c.cache --topspeed cap.pcap
```
