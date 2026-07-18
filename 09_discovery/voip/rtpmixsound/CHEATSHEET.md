---
title: "rtpmixsound Cheatsheet"
description: Quick reference for rtpmixsound audio mixing tool
tool: rtpmixsound
category: voip
difficulty: advanced
---

# rtpmixsound Cheatsheet

## Quick Start

```bash
# Basic mixing
sudo rtpmixsound -i eth0 -f background.wav -r 10000 -s 192.168.1.100 -d 192.168.1.200

# Low volume
sudo rtpmixsound -i eth0 -f background.wav -r 10000 -v 25 -s 192.168.1.100 -d 192.168.1.200

# Loop mode
sudo rtpmixsound -i eth0 -f background.wav -r 10000 -l -s 192.168.1.100 -d 192.168.1.200
```

## Common Flags

| Flag | Description | Example |
|------|-------------|---------|
| `-i` | Interface | `-i eth0` |
| `-f` | Audio file | `-f background.wav` |
| `-r` | RTP port | `-r 10000` |
| `-s` | Source IP | `-s 192.168.1.100` |
| `-d` | Destination IP | `-d 192.168.1.200` |
| `-v` | Volume (0-100) | `-v 30` |
| `-l` | Loop audio | `-l` |
| `-g` | Gain (dB) | `-g 5` |
| `-p` | Payload type | `-p 0` |

## Examples

### Background Audio (25%)
```bash
sudo rtpmixsound -i eth0 -f music.wav -r 10000 -v 25 -s 192.168.1.100 -d 192.168.1.200
```

### Equal Volume Mix (50%)
```bash
sudo rtpmixsound -i eth0 -f alarm.wav -r 10000 -v 50 -s 192.168.1.100 -d 192.168.1.200
```

### Convert Audio First
```bash
sox input.wav -r 8000 -c 1 -e a-law background.raw
sudo rtpmixsound -i eth0 -f background.raw -r 10000 -v 30 -s 192.168.1.100 -d 192.168.1.200
```

## Chaining

```bash
# Detect streams then mix
sudo rtpbreak -i capture.pcap -a -v
# Note the RTP port from output
sudo rtpmixsound -i eth0 -f background.wav -r <port> -v 30 -s 192.168.1.100 -d 192.168.1.200

# ARP spoof + mix
sudo arpspoof -i eth0 -t 192.168.1.100 192.168.1.200 &
echo 1 > /proc/sys/net/ipv4/ip_forward
sudo rtpmixsound -i eth0 -f background.wav -r 10000 -v 30 -s 192.168.1.100 -d 192.168.1.200
```

## Troubleshooting

| Problem | Fix |
|---------|-----|
| No mixing | Detect streams with rtpbreak first |
| Garbled audio | Match payload type with `-p` |
| Permission denied | Run with `sudo` |
| Wrong format | Convert with `sox` |

```bash
# Check audio format
file background.wav
soxi background.wav

# Verify RTP port
sudo tcpdump -i eth0 "udp and portrange 10000-20000" -n -c 10
```
