---
title: rtpbreak
description: RTP stream detector and audio extractor
mitre_tactic: discovery
mitre_tactic_id: TA0007
mitre_subcategory: voip
categories:
  - voip
  - traffic-analysis
  - audio-extraction
  - discovery
tags:
  - rtp
  - voip
  - traffic-analysis
  - audio
  - pcapture
install_command: sudo apt install rtpbreak
version: "1.0"
github_repo: https://github.com/0x90/rtpbreak
kali_tools_page: https://www.kali.org/tools/rtpbreak/
difficulty_level: intermediate
requires_root: true
gui_available: false
related_tools:
  - wireshark
  - voiphopper
  - rtpinsertsound
---

# rtpbreak

## 1. Introduction

### What is rtpbreak?

rtpbreak is a network traffic analysis tool that detects RTP (Real-time Transport Protocol) streams within captured network traffic, separates them from control traffic, and can extract audio data from the streams. It is designed to work with pcap files and can identify VoIP conversations for analysis.

### History

rtpbreak was developed to address the challenge of extracting usable audio from VoIP packet captures. Traditional packet analysis tools like Wireshark can display RTP streams, but rtpbreak automates the process of stream detection, separation, and audio extraction, making it invaluable for VoIP security assessments.

### When to Use

- **VoIP traffic analysis** for security assessments
- **Audio extraction** from captured VoIP calls
- **Stream identification** in mixed network traffic
- **Evidence collection** during investigations
- **Protocol analysis** for compliance and auditing

### VoIP Concepts

**RTP Protocol:**

RTP (RFC 3550) is the standard protocol for delivering audio and video over IP networks. It operates over UDP and provides sequence numbers, timestamps, and payload type identification for real-time data.

**RTP Packet Structure:**

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|V=2|P|X|  CC   |M|     PT      |       sequence number         |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                           timestamp                           |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|           synchronization source (SSRC) identifier            |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

**Audio Codecs:**

| Codec | Payload Type | Sampling Rate | Bandwidth |
|-------|--------------|---------------|-----------|
| PCMU (G.711u) | 0 | 8 kHz | 64 kbps |
| PCMA (G.711a) | 8 | 8 kHz | 64 kbps |
| G.722 | 9 | 16 kHz | 64 kbps |
| GSM | 3 | 8 kHz | 13 kbps |
| G.729 | 18 | 8 kHz | 8 kbps |

---

## 2. Installation

### Kali Linux (Default)

```bash
sudo apt update
sudo apt install rtpbreak
```

### Verify Installation

```bash
rtpbreak --help
```

### Dependencies

rtpbreak requires libpcap for packet capture:

```bash
sudo apt install libpcap-dev
```

---

## 3. Core Concepts

### RTP Stream Detection

```
+-----------+    Pcap File    +------------------+
| rtpbreak  | <------------- | Network Capture  |
+-----------+                +------------------+
     |                              |
     |    Extract RTP Streams       |
     +------------------------------+
     |
     v
+-----------+    Audio Files  +------------------+
| Output    | --------------> | WAV/RAW Files    |
+-----------+                +------------------+
```

**Detection Process:**

1. **Parse pcap**: Read captured network traffic
2. **Identify RTP**: Detect RTP packets by header structure and port ranges
3. **Separate Streams**: Group packets by SSRC (Synchronization Source)
4. **Extract Audio**: Convert RTP payloads to playable audio files

**Stream Identification:**

| Field | Purpose |
|-------|---------|
| SSRC | Unique stream identifier |
| Sequence Number | Packet ordering |
| Timestamp | Timing information |
| Payload Type | Codec identification |

---

## 4. Command Reference

### Basic Syntax

```
rtpbreak [options] <pcap_file>
```

### Command-Line Options

| Flag | Long Form | Description |
|------|-----------|-------------|
| `-i` | `--input` | Input pcap file |
| `-a` | `--auto-detect` | Auto-detect RTP streams |
| `-f` | `--filter` | BPF filter expression |
| `-o` | `--output` | Output directory for audio |
| `-d` | `--debug` | Debug output |
| `-v` | `--verbose` | Verbose information |
| `-w` | `--write-pcap` | Write extracted streams to pcap |
| `-s` | `--ssrc` | Target specific SSRC |
| `-c` | `--codec` | Specify codec for extraction |

---

## 5. Examples

### Example 1: Basic Stream Detection

```bash
sudo rtpbreak -i capture.pcap -a
```

### Example 2: Extract Audio

```bash
sudo rtpbreak -i capture.pcap -a -o ./extracted_audio/
```

### Example 3: With BPF Filter

```bash
sudo rtpbreak -i capture.pcap -a -f "host 192.168.1.100" -o ./audio/
```

### Example 4: Target Specific Stream

```bash
sudo rtpbreak -i capture.pcap -s 0x12345678 -o ./target_stream/
```

### Example 5: Write Extracted Streams

```bash
sudo rtpbreak -i capture.pcap -a -w -o ./extracted/
```

### Example 6: Live Capture Analysis

```bash
# Capture traffic first
sudo tcpdump -i eth0 -w live_capture.pcap portrange 10000-20000 &

# Then analyze
sudo rtpbreak -i live_capture.pcap -a -o ./live_audio/
```

---

## 6. Advanced Usage

### Custom Codec Handling

```bash
# Specify codec for extraction
sudo rtpbreak -i capture.pcap -a -c pcmu -o ./audio/
sudo rtpbreak -i capture.pcap -a -c gsm -o ./audio/
```

### Batch Processing

```bash
#!/bin/bash
for pcap in *.pcap; do
    echo "Processing: $pcap"
    sudo rtpbreak -i "$pcap" -a -o "./audio_$(basename $pcap .pcap)/"
done
```

### Integration with Wireshark

```bash
# Extract streams, then open in Wireshark
sudo rtpbreak -i capture.pcap -a -w -o ./streams/

# Open extracted streams
wireshark ./streams/*.pcap
```

### Audio Post-Processing

```bash
# Convert RAW to WAV using sox
sox -r 8000 -c 1 -e a-law extracted.raw output.wav

# Batch conversion
for raw in *.raw; do
    sox -r 8000 -c 1 -e a-law "$raw" "${raw%.raw}.wav"
done
```

---

## 7. Detection & Defense

### Detection Methods

**Monitoring RTP Traffic:**

```bash
# Detect high-volume RTP traffic
sudo tcpdump -i eth0 -n "udp and (portrange 10000-20000)" -c 100

# Analyze RTP packet distribution
sudo tcpdump -i eth0 -n "udp" -w analysis.pcap
```

**IDS Signatures:**

```
alert udp any any -> 192.168.1.0/24 10000:20000 (msg:"RTP Stream Detected"; content:"|80|"; depth:1; sid:1000004; rev:1;)
```

### Defense Strategies

**SRTP (Secure RTP):**

SRTP (RFC 3711) encrypts RTP payloads, preventing audio extraction:

```bash
# Enable SRTP in Asterisk
; /etc/asterisk/sip.conf
[general]
encryption=yes
```

**Network Segmentation:**

- Isolate VoIP traffic on separate VLANs
- Implement 802.1X for VoIP devices
- Use VPN for remote VoIP connections

**TLS for Signaling:**

- Use SIP over TLS (SIPS)
- Implement mutual authentication
- Enable certificate pinning

---

## 8. Troubleshooting

### Common Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| No streams detected | Wrong port range | Use `-f` filter for correct ports |
| Garbled audio | Wrong codec | Specify codec with `-c` |
| Permission denied | Root required | Use `sudo` |
| No output files | Invalid output dir | Check directory permissions |

### Debug Commands

```bash
# Verbose mode
sudo rtpbreak -i capture.pcap -a -v -d

# Verify pcap contents
tcpdump -r capture.pcap -n -c 20

# Check RTP ports
tcpdump -r capture.pcap -n "udp" | awk '{print $5}' | grep -oP '\d+$' | sort -n | uniq
```

### Audio Quality Issues

```bash
# Check audio file properties
file extracted.raw
soxi extracted.raw

# Resample if needed
sox extracted.raw -r 16000 resampled.raw
```

---

## Resources

- [RTP RFC 3550](https://tools.ietf.org/html/rfc3550)
- [SRTP RFC 3711](https://tools.ietf.org/html/rfc3711)
- [Wireshark RTP Analysis](https://www.wireshark.org/)
- [VoIP Security Guide](https://www.sans.org/white-papers/voip-security/)
