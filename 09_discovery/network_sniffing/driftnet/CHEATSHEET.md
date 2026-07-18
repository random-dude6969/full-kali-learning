---
title: "driftnet - Quick Reference Cheatsheet"
description: "Quick reference card for driftnet image and audio sniffer"
mitre_tactic: discovery
mitre_tactic_id: TA0007
mitre_subcategory: network_sniffing
tags: [driftnet, cheatsheet, image-capture, audio-capture, network-sniffer]
---

# driftnet — Cheatsheet

## Quick Start

```bash
# Install
sudo apt install driftnet

# Capture images (X11 display)
sudo driftnet -i eth0

# Save images to directory
sudo driftnet -i eth0 -m /tmp/images

# No display (file-only)
sudo driftnet -i eth0 -D -m /tmp/images
```

## Core Commands

| Command | Description |
|---------|-------------|
| `sudo driftnet -i eth0` | Capture with X11 display |
| `sudo driftnet -i eth0 -m /dir` | Save to directory |
| `sudo driftnet -i eth0 -D` | No display (file-only) |
| `driftnet -p file.pcap` | Analyze pcap file |
| `sudo driftnet -i eth0 -a` | Capture audio |
| `sudo driftnet -i eth0 -A` | Capture all media |

## Common Flags

| Flag | Description |
|------|-------------|
| `-i` | Network interface |
| `-p` | Read from pcap file |
| `-m` | Save directory |
| `-d` | Enable X11 display |
| `-D` | Disable X11 display |
| `-v` | Verbose output |
| `-a` | Capture audio |
| `-A` | Capture all media |
| `-f` | BPF filter |
| `-c` | Max packet count |
| `-J` | JPEG only |
| `-G` | GIF only |
| `-P` | PNG only |

## Supported Formats

| Format | Type | Magic Bytes |
|--------|------|-------------|
| JPEG | Image | FF D8 FF |
| GIF | Image | GIF87a/GIF89a |
| PNG | Image | 89 50 4E 47 |
| BMP | Image | 42 4D |
| WAV | Audio | 52 49 46 46 |
| MP3 | Audio | FF FB/FF F3 |

## Examples

```bash
# Basic image capture
sudo driftnet -i eth0

# Capture from specific host
sudo driftnet -i eth0 -f "host 192.168.1.100"

# Capture audio only
sudo driftnet -i eth0 -a -m /tmp/audio

# File-only with verbose
sudo driftnet -i eth0 -D -m /tmp/images -v

# Analyze existing pcap
driftnet -p capture.pcap -m /tmp/images -D

# Capture JPEG only
sudo driftnet -i eth0 -J -m /tmp/jpeg

# Full monitoring
sudo driftnet -i eth0 -m /tmp/images -a -v
```

## Chaining

```bash
# driftnet + tcpdump
sudo driftnet -i eth0 -m /tmp/images -D &
sudo tcpdump -i eth0 -w /tmp/full.pcap &
wait

# driftnet + ngrep for text
sudo driftnet -i eth0 -m /tmp/images -D &
sudo ngrep -l -q -d eth0 "password|secret" port 80 &
wait

# Capture from pcap
tcpdump -r capture.pcap -w - | driftnet -p - -m /tmp/images -D
```

## Capture Filters (BPF)

```bash
# By host
sudo driftnet -i eth0 -f "host 192.168.1.100"

# By subnet
sudo driftnet -i eth0 -f "net 192.168.1.0/24"

# HTTP only
sudo driftnet -i eth0 -f "port 80"

# Exclude management
sudo driftnet -i eth0 -f "not port 22 and not port 3389"
```

## Output Structure

```
/tmp/images/
├── driftnet-000001.jpg
├── driftnet-000002.gif
├── driftnet-000003.png
└── ...
```

## Troubleshooting

| Problem | Solution |
|---------|----------|
| No X11 display | Use `-D` flag or set `export DISPLAY=:0` |
| No images captured | Check for HTTP traffic: `tcpdump -i eth0 port 80` |
| Permission denied | Run with sudo |
| Images corrupted | Use verbose: `-v`, check stream reassembly |
| No audio captured | Check audio support: compile with `AUDIO=1` |
| High CPU usage | Add BPF filter to reduce traffic |
