# Firewalk — Cheatsheet

## Quick Reference

**Purpose**: Enumerate firewall rules by sending TTL-limited probes through a gateway.

**Requires**: Root privileges, libdumbnet, libpcap, libnet

**Install**: `sudo apt install firewalk`

---

## Core Syntax

```bash
sudo firewalk [options] <gateway_ip> <target_ip>
```

**First argument** = gateway (default router), **Second argument** = target host behind gateway.

---

## Essential Commands

### Basic Port Scan

```bash
# Scan HTTP/HTTPS through gateway
sudo firewalk -S80,443 -n -pTCP 192.168.1.1 192.168.0.1

# Scan well-known ports
sudo firewalk -S1-1024 -n -pTCP 192.168.1.1 192.168.0.1

# Scan specific common ports
sudo firewalk -S21,22,23,25,53,80,110,143,443,993,995 -n -pTCP 192.168.1.1 192.168.0.1
```

### Protocol Variants

```bash
# TCP scan (most common)
sudo firewalk -S80,443 -n -pTCP 192.168.1.1 192.168.0.1

# UDP scan
sudo firewalk -S53,123,161 -n -pUDP 192.168.1.1 192.168.0.1

# ICMP scan
sudo firewalk -S0 -n -pICMP 192.168.1.1 192.168.0.1
```

---

## Flag Reference

| Flag | Description | Example |
|------|-------------|---------|
| `-S <ports>` | Port range to scan | `-S80,443,8080` |
| `-S 1-1024` | Port range (dash syntax) | `-S1-1024` |
| `-p <proto>` | Protocol (TCP/UDP/ICMP) | `-pTCP` |
| `-i <iface>` | Network interface | `-i eth0` |
| `-n` | Disable DNS resolution | `-n` |
| `-s <port>` | Source port | `-s 54321` |
| `-T <ms>` | Timeout in milliseconds | `-T5000` |
| `-t <ttl>` | Manual TTL override | `-t 64` |
| `-d <port>` | Default destination port | `-d 80` |
| `-r` | Strict RFC compliance | `-r` |
| `-x <hops>` | Manual expire vector | `-x 4` |
| `-v` | Verbose output | `-v` |
| `-h` | Help | `-h` |

---

## Common Scenarios

### Pre-Attack Reconnaissance

```bash
# Quick scan of common attack vectors
sudo firewalk -S22,80,443,3389,8080,8443 -n -pTCP 192.168.1.1 192.168.0.50

# Full well-known port enumeration
sudo firewalk -S1-1024 -n -pTCP 192.168.1.1 192.168.0.50
```

### Firewall Policy Verification

```bash
# Verify only authorized ports are open
sudo firewalk -S1-65535 -n -pTCP 192.168.1.1 192.168.0.50 > audit_report.txt

# Check specific compliance ports
sudo firewalk -S22,80,443,3306,5432 -n -pTCP 192.168.1.1 192.168.0.50
```

### Non-Standard Topology

```bash
# Manual TTL for multi-hop network
sudo firewalk -S80 -n -pTCP -t 3 192.168.1.1 192.168.0.1

# Override expire vector
sudo firewalk -S80 -n -pTCP -x 5 192.168.1.1 192.168.0.1
```

---

## Output Interpretation

| Result | Meaning |
|--------|---------|
| **open** | Gateway forwarded probe; target port is open |
| **closed** | Gateway forwarded probe; target port responded with RST |
| **filtered** | Gateway dropped probe; port is blocked |
| **unreachable** | ICMP unreachable received |

---

## Chaining Workflow

```bash
# Step 1: Discover gateway
ip route show default
traceroute -n 192.168.0.1

# Step 2: Enumerate firewall
sudo firewalk -S1-1024 -n -pTCP 192.168.1.1 192.168.0.1

# Step 3: Deep scan open ports
nmap -sV -sC -p <open_ports> 192.168.0.1

# Step 4: Capture traffic during scan
sudo tcpdump -i eth0 -w firewalk_capture.pcap host 192.168.0.1 &
```

---

## Troubleshooting

```bash
# "No route to host" → verify connectivity
ping -c 3 192.168.1.1 && ping -c 3 192.168.0.1

# All filtered → try manual TTL
sudo firewalk -S80 -n -pTCP -t 3 192.168.1.1 192.168.0.1

# Permission denied → run as root
sudo firewalk -S80 -n -pTCP 192.168.1.1 192.168.0.1

# Inconsistent results → increase timeout
sudo firewalk -S80 -n -pTCP -T10000 192.168.1.1 192.168.0.1
```

---

## One-Liners

```bash
# Quick web port check
sudo firewalk -S80,443 -n -pTCP 192.168.1.1 192.168.0.1

# Full well-known scan
sudo firewalk -S1-1024 -n -pTCP 192.168.1.1 192.168.0.1

# Database ports only
sudo firewalk -S3306,5432,1433,27017 -n -pTCP 192.168.1.1 192.168.0.1

# Remote access ports
sudo firewalk -S22,23,3389,5900 -n -pTCP 192.168.1.1 192.168.0.1

# Verbose with timeout
sudo firewalk -S80,443 -n -pTCP -v -T10000 192.168.1.1 192.168.0.1
```
