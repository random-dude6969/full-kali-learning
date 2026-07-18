---
title: "rtpbreak Cheatsheet"
description: Quick reference for rtpbreak RTP stream detector
tool: rtpbreak
category: voip
difficulty: intermediate
---

# rtpbreak Cheatsheet

## Quick Start

```bash
# Detect RTP streams
sudo rtpbreak -i capture.pcap -a

# Extract audio
sudo rtpbreak -i capture.pcap -a -o ./audio/

# With filter
sudo rtpbreak -i capture.pcap -a -f "host 192.168.1.100"
```

## RTP Codec Reference

| Codec | Payload Type | Rate | Bandwidth |
|-------|--------------|------|-----------|
| PCMU | 0 | 8kHz | 64kbps |
| PCMA | 8 | 8kHz | 64kbps |
| G.722 | 9 | 16kHz | 64kbps |
| GSM | 3 | 8kHz | 13kbps |
| G.729 | 18 | 8kHz | 8kbps |

## Common Flags

| Flag | Description | Example |
|------|-------------|---------|
| `-i` | Input pcap | `-i capture.pcap` |
| `-a` | Auto-detect | `-a` |
| `-f` | BPF filter | `-f "host 192.168.1.100"` |
| `-o` | Output dir | `-o ./audio/` |
| `-v` | Verbose | `-v` |
| `-w` | Write pcap | `-w` |
| `-s` | Target SSRC | `-s 0x12345678` |
| `-c` | Codec | `-c pcmu` |

## Examples

### Extract All Audio
```bash
sudo rtpbreak -i capture.pcap -a -o ./extracted/
```

### Target Specific Host
```bash
sudo rtpbreak -i capture.pcap -a -f "host 192.168.1.100" -o ./host_audio/
```

### Write Extracted Streams
```bash
sudo rtpbreak -i capture.pcap -a -w -o ./streams/
```

### Specify Codec
```bash
sudo rtpbreak -i capture.pcap -a -c gsm -o ./gsm_audio/
```

## Chaining

```bash
# Capture then extract
sudo tcpdump -i eth0 -w rtp_capture.pcap portrange 10000-20000 &
sleep 30
sudo rtpbreak -i rtp_capture.pcap -a -o ./audio/

# Convert raw to wav
for f in *.raw; do
    sox -r 8000 -c 1 -e a-law "$f" "${f%.raw}.wav"
done
```

## Troubleshooting

| Problem | Fix |
|---------|-----|
| No streams found | Use `-f` filter for correct ports |
| Garbled audio | Specify codec with `-c` |
| Permission denied | Run with `sudo` |
| No output | Check output directory exists |

```bash
# Verify pcap
tcpdump -r capture.pcap -n -c 10

# Check RTP ports
tcpdump -r capture.pcap -n "udp" | awk '{print $5}' | grep -oP '\d+$' | sort -n | uniq
```
