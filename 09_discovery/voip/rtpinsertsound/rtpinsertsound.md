---
title: rtpinsertsound
description: Audio injection tool for active RTP streams
mitre_tactic: discovery
mitre_tactic_id: TA0007
mitre_subcategory: voip
categories:
  - voip
  - audio-injection
  - rtp-manipulation
  - discovery
tags:
  - rtp
  - voip
  - audio
  - injection
  - man-in-the-middle
install_command: sudo apt install rtpinsertsound
version: "1.0"
github_repo: https://github.com/0x90/rtpinsertsound
kali_tools_page: https://www.kali.org/tools/rtpinsertsound/
difficulty_level: advanced
requires_root: true
gui_available: false
related_tools:
  - rtpmixsound
  - rtpbreak
  - wireshark
---

# rtpinsertsound

## 1. Introduction

### What is rtpinsertsound?

rtpinsertsound is a VoIP security testing tool that injects audio into active RTP (Real-time Transport Protocol) streams. It allows security researchers to insert arbitrary audio into ongoing VoIP calls, demonstrating the lack of integrity protection in unencrypted RTP streams. This tool is essential for testing VoIP security and demonstrating man-in-the-middle attack capabilities.

### History

rtpinsertsound was developed as part of VoIP security research to demonstrate the vulnerabilities of unencrypted RTP streams. It complements tools like rtpbreak (which detects streams) and rtpmixsound (which mixes audio) by providing a direct injection capability for security testing.

### When to Use

- **VoIP security assessments** demonstrating injection vulnerabilities
- **Penetration testing** of unencrypted VoIP infrastructure
- **Security awareness training** showing RTP vulnerabilities
- **Compliance testing** for encrypted voice requirements
- **Research** into VoIP attack vectors

### VoIP Concepts

**RTP Stream Manipulation:**

Without encryption, RTP streams can be intercepted and modified. rtpinsertsound exploits this by:

1. Identifying active RTP streams on the network
2. Crafting RTP packets with injected audio
3. Injecting packets into the stream
4. Maintaining sequence numbers and timing

**Attack Scenario:**

```
+--------+    RTP Stream    +----------+
| Caller | <--------------->| Callee  |
+--------+                  +----------+
     ^                           ^
     |                           |
     |    +-----------------+    |
     +----| rtpinsertsound  |----+
          +-----------------+
          Injects audio into
          both directions
```

---

## 2. Installation

### Kali Linux (Default)

```bash
sudo apt update
sudo apt install rtpinsertsound
```

### Verify Installation

```bash
rtpinsertsound --help
```

### Dependencies

```bash
sudo apt install libpcap-dev libasound2-dev
```

---

## 3. Core Concepts

### Audio Injection Architecture

```
+-----------+    RTP Stream    +------------------+
| rtpinsert | <-------------> | Active VoIP Call |
+-----------+                 +------------------+
     |                              |
     |    Inject Audio Packets      |
     +------------------------------+
     |
     v
+-----------+    Audio File   +------------------+
| WAV/RAW   | -------------> | RTP Stream       |
+-----------+                 +------------------+
```

**Injection Methods:**

| Method | Description | Use Case |
|--------|-------------|----------|
| Stream Replace | Replace original audio | Complete takeover |
| Stream Mix | Mix with original | Background injection |
| Sequential | Insert at specific point | Targeted injection |

**Packet Crafting:**

rtpinsertsound crafts RTP packets with:
- Correct SSRC (Synchronization Source)
- Proper sequence numbers
- Matching timestamps
- Correct payload type

---

## 4. Command Reference

### Basic Syntax

```
rtpinsertsound [options] <target>
```

### Command-Line Options

| Flag | Long Form | Description |
|------|-----------|-------------|
| `-i` | `--interface` | Network interface |
| `-f` | `--file` | Audio file to inject (WAV/RAW) |
| `-r` | `--rtp-port` | Target RTP port |
| `-s` | `--source` | Source IP address |
| `-d` | `--destination` | Destination IP address |
| `-p` | `--payload-type` | RTP payload type |
| `-v` | `--verbose` | Verbose output |
| `-l` | `--loop` | Loop audio file |
| `-g` | `--gain` | Audio gain (dB) |

---

## 5. Examples

### Example 1: Basic Audio Injection

Inject audio into an RTP stream between two hosts:

```bash
sudo rtpinsertsound -i eth0 -f injection.wav -r 10000 -s 192.168.1.100 -d 192.168.1.200
```

### Example 2: Loop Injection

Continuously inject audio:

```bash
sudo rtpinsertsound -i eth0 -f loop.wav -r 10000 -l -s 192.168.1.100 -d 192.168.1.200
```

### Example 3: With Gain Adjustment

Boost audio volume during injection:

```bash
sudo rtpinsertsound -i eth0 -f quiet.wav -r 10000 -g 10 -s 192.168.1.100 -d 192.168.1.200
```

### Example 4: Specific Payload Type

Match target codec:

```bash
sudo rtpinsertsound -i eth0 -f audio.wav -r 10000 -p 0 -s 192.168.1.100 -d 192.168.1.200
```

### Example 5: With Stream Detection

First detect streams, then inject:

```bash
# Detect streams
sudo rtpbreak -i capture.pcap -a -v

# Inject into detected stream
sudo rtpinsertsound -i eth0 -f injection.wav -r 10000 -s 192.168.1.100 -d 192.168.1.200
```

---

## 6. Advanced Usage

### Audio Format Conversion

```bash
# Convert WAV to RAW for injection
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

# Step 3: Inject audio
sudo rtpinsertsound -i eth0 -f injection.wav -r 10000 -s 192.168.1.100 -d 192.168.1.200
```

### Automation Script

```bash
#!/bin/bash
INTERFACE="eth0"
AUDIO_FILE="injection.wav"
RTP_PORT="10000"
TARGET_SRC="192.168.1.100"
TARGET_DST="192.168.1.200"

echo "Starting audio injection..."
sudo rtpinsertsound -i $INTERFACE -f $AUDIO_FILE -r $RTP_PORT -s $TARGET_SRC -d $TARGET_DST -v
```

---

## 7. Detection & Defense

### Detection Methods

**Network Monitoring:**

```bash
# Detect unusual RTP traffic
sudo tcpdump -i eth0 "udp and port 10000-20000" -n -c 100

# Monitor for injected packets
sudo tcpdump -i eth0 "udp and port 10000" -v | grep -i "payload"
```

**IDS Signatures:**

```
alert udp 192.168.1.0/24 any -> 192.168.1.0/24 10000:20000 (msg:"RTP Injection Detected"; dsize:>200; threshold:type both, track by_src, count 100, seconds 10; sid:1000005; rev:1;)
```

### Defense Strategies

**SRTP Encryption:**

Enable SRTP to prevent injection:

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
| No injection | Wrong RTP port | Use rtpbreak to detect streams |
| Garbled audio | Wrong codec | Match payload type with `-p` |
| Permission denied | Root required | Use `sudo` |
| Injection fails | Firewall blocking | Check iptables rules |

### Debug Commands

```bash
# Verbose mode
sudo rtpinsertsound -i eth0 -f audio.wav -r 10000 -v

# Capture injected traffic
sudo tcpdump -i eth0 "udp and port 10000" -w injected.pcap

# Verify audio format
file audio.wav
soxi audio.wav
```

### Audio Quality Issues

```bash
# Check audio properties
soxi injection.wav

# Convert to correct format
sox injection.wav -r 8000 -c 1 -e a-law injection.raw

# Test audio playback
aplay injection.wav
```

---

## Resources

- [RTP RFC 3550](https://tools.ietf.org/html/rfc3550)
- [SRTP RFC 3711](https://tools.ietf.org/html/rfc3711)
- [VoIP Security Testing](https://www.sans.org/white-papers/voip-security/)
- [OWASP VoIP Security](https://owasp.org/)
