---
title: "driftnet - Network Image and Audio Sniffer"
description: "Passive sniffer that extracts images and audio from HTTP traffic streams in real-time"
mitre_tactic: discovery
mitre_tactic_id: TA0007
mitre_subcategory: network_sniffing
categories:
  - Traffic Analysis
  - Media Extraction
  - Passive Monitoring
  - HTTP Analysis
tags:
  - images
  - audio
  - http
  - sniffer
  - media-extraction
  - passive
  - x11
install_command: "sudo apt install driftnet"
version: "0.9.5"
kali_tools_page: "https://www.kali.org/tools/driftnet/"
difficulty_level: beginner
requires_root: true
gui_available: true
related_tools:
  - tcpdump
  - mitmproxy
  - ngrep
  - urlsnarf
  - pyxplot
---

# driftnet — Network Image and Audio Sniffer

## Chapter 1: Introduction

### What is driftnet

**driftnet** is a passive network sniffer that extracts images and audio from HTTP traffic streams in real-time. It monitors a network interface, identifies media content within unencrypted HTTP traffic, and displays extracted images in an X11 window while saving them to disk.

driftnet operates entirely passively — it reads packets flowing through the network interface but does not generate or inject any traffic. It specifically targets HTTP traffic (port 80) and extracts JPEG, GIF, PNG, BMP images and audio files from the data streams.

The tool is particularly useful for visual reconnaissance during penetration testing, allowing testers to see exactly what images users are viewing without intercepting or modifying traffic.

### Historical Context

driftnet was created by Chris Wright at the University of Adelaide in 2000. It was one of the first tools to automatically extract media from network traffic, filling a unique niche between packet analyzers (tcpdump) and full-featured proxy tools (mitmproxy).

The tool gained popularity in the early 2000s when HTTP was the dominant web protocol and HTTPS was rare. It provided a visual way to monitor network traffic that was more intuitive than reading hex dumps or protocol logs.

While the shift to HTTPS has reduced driftnet's effectiveness against modern web traffic, it remains useful for:

- Monitoring unencrypted HTTP traffic on internal networks
- Analyzing IoT devices that often lack TLS
- Capturing media from legacy applications
- Educational demonstrations of network sniffing

### When to Use driftnet

- **Visual reconnaissance**: See what images users are viewing on the network
- **Network monitoring**: Passively capture media content from HTTP traffic
- **IoT analysis**: Extract media from unencrypted IoT device communications
- **Penetration testing**: Demonstrate the risks of unencrypted HTTP
- **Education**: Learn about network protocols and media extraction
- **Incident response**: Recover images from captured network traffic

### Core Concepts

- **Passive capture**: Reads packets without generating traffic
- **Media extraction**: Identifies and extracts image/audio formats from HTTP streams
- **Real-time display**: Shows images as they are captured in an X11 window
- **File saving**: Saves extracted media to a specified directory
- **BPF filtering**: Uses Berkeley Packet Filter to target specific traffic
- **Protocol awareness**: Understands HTTP content types and transfer encoding

---

## Chapter 2: Installation

### System Requirements

| Requirement | Minimum | Recommended |
|------------|---------|-------------|
| OS | Kali Linux 2020+ | Kali Linux 2024+ |
| RAM | 64MB | 256MB+ |
| CPU | Single core | Single core |
| Disk | 10MB | 100MB+ for captured images |
| Display | X11 server | X11 with image viewer |
| Root | Required | Required |

### Installation Methods

**Method 1: Kali Package Manager (Recommended)**

```bash
sudo apt update
sudo apt install driftnet
```

**Method 2: From Source**

```bash
sudo apt install git build-essential libpcap-dev libgtk2.0-dev
git clone https://github.com/irdo/driftnet.git
cd driftnet
make
sudo cp driftnet /usr/local/bin/
```

**Method 3: Compile with Audio Support**

```bash
sudo apt install git build-essential libpcap-dev libgtk2.0-dev \
    libavformat-dev libavcodec-dev libavutil-dev libswscale-dev
git clone https://github.com/irdo/driftnet.git
cd driftnet
make AUDIO=1
sudo cp driftnet /usr/local/bin/
```

### Post-Installation Verification

```bash
# Verify installation
driftnet --help

# Check X11 display
echo $DISPLAY

# Test basic functionality
driftnet -i lo &
sleep 2
# Generate test traffic
curl http://example.com/image.jpg
```

### Permission Setup

```bash
# Ensure packet capture capabilities
sudo setcap cap_net_raw,cap_net_admin=eip /usr/bin/driftnet

# Or run with sudo
sudo driftnet -i eth0
```

---

## Chapter 3: Core Concepts

### How driftnet Works

driftnet operates through a multi-stage pipeline:

```
[Network Interface] → [Packet Capture] → [HTTP Stream Reassembly] → [Media Detection] → [Extraction/Display]
```

1. **Packet capture**: Uses libpcap to capture all packets on the interface
2. **Stream reassembly**: Reconstructs TCP streams from individual packets
3. **HTTP parsing**: Identifies HTTP responses and their content types
4. **Media detection**: Examines content-type headers and file signatures
5. **Extraction**: Extracts image/audio data from HTTP bodies
6. **Display**: Shows images in X11 window and saves to disk

### Supported Media Formats

| Format | Type | Extension | Detection Method |
|--------|------|-----------|------------------|
| JPEG | Image | .jpeg, .jpg | Magic bytes: FF D8 FF |
| GIF | Image | .gif | Magic bytes: GIF87a/GIF89a |
| PNG | Image | .png | Magic bytes: 89 50 4E 47 |
| BMP | Image | .bmp | Magic bytes: 42 4D |
| WAV | Audio | .wav | Magic bytes: 52 49 46 46 |
| MP3 | Audio | .mp3 | Magic bytes: FF FB/FF F3 |
| MPEG | Video | .mpeg | Magic bytes: 00 00 01 BA |

### Architecture

```
┌─────────────────────────────────────────────┐
│              driftnet                        │
├─────────────┬─────────────┬─────────────────┤
│  Capture    │  Processing │  Output         │
│  Engine     │  Engine     │  Engine         │
├─────────────┼─────────────┼─────────────────┤
│ libpcap     │ TCP stream  │ X11 display     │
│ Promiscuous │ reassembly  │ File saving     │
│ mode        │ HTTP parse  │ Console output  │
│ BPF filters │ Media       │ Logging         │
│             │ detection   │                 │
└─────────────┴─────────────┴─────────────────┘
```

### Display Modes

**X11 display mode (default):**

```bash
driftnet -i eth0
# Images appear in a window as they are captured
```

**File-only mode (no X11):**

```bash
driftnet -i eth0 -m /tmp/images -d
# Images saved to directory, no display
```

---

## Chapter 4: Command Reference

### Basic Syntax

```bash
driftnet [OPTIONS] -i <interface>
```

### Core Flags

| Flag | Long | Description | Example |
|------|------|-------------|---------|
| `-i` | `--interface` | Network interface | `-i eth0` |
| `-p` | `--pcapfile` | Read from pcap file | `-p capture.pcap` |
| `-m` | `--savedir` | Save directory | `-m /tmp/images` |
| `-d` | `--display` | Enable/disable X11 | `-d` or `-D` |
| `-v` | `--verbose` | Verbose output | `-v` |
| `-a` | `--audio` | Capture audio | `-a` |
| `-A` | `--all` | Capture all media | `-A` |

### Capture Flags

| Flag | Description | Example |
|------|-------------|---------|
| `-i` | Interface to capture | `-i eth0` |
| `-p` | Read from pcap file | `-p file.pcap` |
| `-f` | BPF filter expression | `-f "port 80"` |
| `-c` | Maximum packet count | `-c 10000` |

### Output Flags

| Flag | Description | Example |
|------|-------------|---------|
| `-m` | Save directory for images | `-m /tmp/captured` |
| `-d` | Enable X11 display | `-d` |
| `-D` | Disable X11 display | `-D` |
| `-v` | Verbose output | `-v` |

### Media Flags

| Flag | Description | Example |
|------|-------------|---------|
| `-a` | Capture audio | `-a` |
| `-A` | Capture all media types | `-A` |
| `-J` | JPEG only | `-J` |
| `-G` | GIF only | `-G` |
| `-P` | PNG only | `-P` |

---

## Chapter 5: Examples

### Basic Examples

**Capture images from interface:**

```bash
sudo driftnet -i eth0
```

**Save images to directory:**

```bash
sudo driftnet -i eth0 -m /tmp/images
```

**Read from pcap file:**

```bash
driftnet -p capture.pcap -m /tmp/images -D
```

### Intermediate Examples

**Filter specific traffic:**

```bash
# Only capture from specific host
sudo driftnet -i eth0 -f "host 192.168.1.100"

# Only HTTP traffic
sudo driftnet -i eth0 -f "port 80"

# Exclude management traffic
sudo driftnet -i eth0 -f "not port 22"
```

**Capture audio only:**

```bash
sudo driftnet -i eth0 -a -m /tmp/audio
```

**No display (file-only):**

```bash
sudo driftnet -i eth0 -D -m /tmp/images -v
```

### Advanced Examples

**Full monitoring pipeline:**

```bash
#!/bin/bash
# driftnet_monitor.sh — Automated media capture

INTERFACE="eth0"
SAVEDIR="/var/lib/driftnet/$(date +%Y%m%d)"
LOGFILE="/var/log/driftnet/$(date +%Y%m%d).log"

# Create directories
mkdir -p "$SAVEDIR" /var/log/driftnet

# Start driftnet
sudo driftnet -i "$INTERFACE" \
    -m "$SAVEDIR" \
    -D \
    -v \
    -a \
    2>&1 | tee "$LOGFILE" &

DPID=$!
echo "driftnet running as PID $DPID"
echo "Images saved to: $SAVEDIR"
echo "Logs: $LOGFILE"

# Monitor for new files
while true; do
    NEW_FILES=$(find "$SAVEDIR" -newer /tmp/driftnet_marker -type f 2>/dev/null | wc -l)
    if [ "$NEW_FILES" -gt 0 ]; then
        echo "$(date): $NEW_FILES new files captured" >> "$LOGFILE"
    fi
    touch /tmp/driftnet_marker
    sleep 60
done
```

**Batch analysis of pcap files:**

```bash
#!/bin/bash
# batch_driftnet.sh — Extract images from multiple pcaps

CAPTURE_DIR="/evidence/captures"
OUTPUT_BASE="/evidence/images"
mkdir -p "$OUTPUT_BASE"

for pcap in "$CAPTURE_DIR"/*.pcap; do
    name=$(basename "$pcap" .pcap)
    echo "Processing: $name"
    mkdir -p "$OUTPUT_BASE/$name"
    driftnet -p "$pcap" -m "$OUTPUT_BASE/$name" -D -v
done

# Generate summary
echo "=== Image Extraction Summary ===" > "$OUTPUT_BASE/summary.txt"
for dir in "$OUTPUT_BASE"/*/; do
    count=$(ls "$dir" 2>/dev/null | wc -l)
    echo "$(basename "$dir"): $count files" >> "$OUTPUT_BASE/summary.txt"
done
```

**Capture with protocol analysis:**

```bash
# Start driftnet for visual monitoring
sudo driftnet -i eth0 -m /tmp/images -v &

# Simultaneously capture full packets
sudo tcpdump -i eth0 -w /tmp/full_capture.pcap &

# Monitor HTTP requests
sudo ngrep -l -q -d eth0 "GET|POST" port 80 &

wait
```

### Real-World Scenarios

**Penetration test — demonstrate HTTP risks:**

```bash
# During authorized engagement, capture images to demonstrate
# the risks of unencrypted HTTP
sudo driftnet -i eth0 -m /tmp/pt_evidence -v

# Present captured images as evidence of data exposure
ls -la /tmp/pt_evidence/
```

**IoT analysis — extract media from devices:**

```bash
# Many IoT devices use unencrypted HTTP
# Capture images from security cameras, digital frames, etc.
sudo driftnet -i eth0 -m /tmp/iot_media -f "net 192.168.1.0/24" -A -v
```

**Incident response — recover images from traffic:**

```bash
# Analyze captured pcap for image content
driftnet -p /evidence/capture.pcap -m /tmp/recovered_images -D -v

# Review recovered images
ls -la /tmp/recovered_images/
file /tmp/recovered_images/*
```

---

## Chapter 6: Advanced Usage

### Scripting with driftnet

**Automated capture with notification:**

```bash
#!/bin/bash
# auto_capture.sh — Notify on new image capture

INTERFACE="eth0"
SAVEDIR="/tmp/captured_images"
NOTIFY_DIR="/tmp/notify"

mkdir -p "$SAVEDIR" "$NOTIFY_DIR"

# Create marker file
touch /tmp/driftnet_marker

# Start driftnet
sudo driftnet -i "$INTERFACE" -m "$SAVEDIR" -D &

DPID=$!

# Monitor for new files
while kill -0 $DPID 2>/dev/null; do
    NEW_FILES=$(find "$SAVEDIR" -newer /tmp/driftnet_marker -type f 2>/dev/null)

    for file in $NEW_FILES; do
        if [ -f "$file" ]; then
            # Log new capture
            echo "$(date): Captured $(basename "$file")" >> "$NOTIFY_DIR/captures.log"

            # Desktop notification (if available)
            notify-send "Image Captured" "$(basename "$file")" 2>/dev/null
        fi
    done

    touch /tmp/driftnet_marker
    sleep 5
done
```

### Chaining with Other Tools

**driftnet + tcpdump for comprehensive capture:**

```bash
# driftnet for image extraction
sudo driftnet -i eth0 -m /tmp/images -D -v &

# tcpdump for full packet capture
sudo tcpdump -i eth0 -w /tmp/full_capture.pcap -s 0 &

# Wait for user interrupt
wait
```

**driftnet + ngrep for content analysis:**

```bash
# driftnet for images
sudo driftnet -i eth0 -m /tmp/images -D &

# ngrep for text content
sudo ngrep -l -q -d eth0 "password|secret|token" port 80 &

wait
```

**driftnet + mitmproxy for full interception:**

```bash
# Start mitmproxy as transparent proxy
sudo mitmproxy --mode transparent --showhost &

# Start driftnet for image capture
sudo driftnet -i eth0 -m /tmp/images &

# Configure iptables redirect
sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-port 8080

wait
```

### Configuration Tuning

**Optimizing for high-traffic networks:**

```bash
# Apply BPF filter to reduce processing
sudo driftnet -i eth0 -f "port 80 and not host 192.168.1.1" -m /tmp/images

# Reduce memory usage
sudo driftnet -i eth0 -c 1000 -m /tmp/images
```

**Capture specific image types:**

```bash
# Only JPEG images
sudo driftnet -i eth0 -J -m /tmp/jpeg_images

# Only PNG images
sudo driftnet -i eth0 -P -m /tmp/png_images
```

---

## Chapter 7: Detection and Defense

### How driftnet Appears in Logs

driftnet is a passive tool — it doesn't generate network traffic. However, its presence can be detected:

**Process artifacts:**

```bash
# Detect driftnet running
ps aux | grep driftnet
pgrep -a driftnet

# Check for X11 connections
lsof -i -X | grep X11
```

**File system artifacts:**

```bash
# Check for captured images
find /tmp -name "*.jpg" -o -name "*.png" -o -name "*.gif" 2>/dev/null
ls -la /tmp/images/
```

**Log artifacts:**

```bash
# Check for driftnet logs
find /var/log -name "driftnet*" 2>/dev/null
journalctl | grep driftnet
```

### Defensive Monitoring

**Monitor for media extraction tools:**

```bash
#!/bin/bash
# detect_driftnet.sh

# Watch for driftnet binary execution
auditctl -w /usr/bin/driftnet -p x -k driftnet_exec

# Watch for image capture directories
auditctl -w /tmp/images/ -p wa -k image_capture

# Monitor for unusual image creation
inotifywait -m /tmp/ -e create --format '%f' | \
while read file; do
    if [[ "$file" =~ \.(jpg|png|gif|bmp)$ ]]; then
        logger -t driftnet_monitor "Image captured: $file"
    fi
done
```

### Evasion Considerations

Media extraction can be prevented through:

- **HTTPS encryption**: TLS prevents plaintext HTTP inspection
- **Content encoding**: Transfer-Encoding: chunked may disrupt extraction
- **Protocol obfuscation**: Using non-standard ports or protocols
- **Image obfuscation**: Serving images through data URIs or base64 encoding

**Defensive countermeasures:**

```bash
# Enforce HTTPS everywhere
# Use HSTS headers
# Deploy TLS inspection proxies
# Monitor for HTTP traffic on unusual ports
# Implement network encryption (IPsec, MACsec)
```

---

## Chapter 8: Troubleshooting and FAQ

### Common Issues

**"No X11 display" error:**

```bash
# Problem: X11 not available or DISPLAY not set
# Solution 1: Set DISPLAY variable
export DISPLAY=:0
driftnet -i eth0

# Solution 2: Use file-only mode
driftnet -i eth0 -D -m /tmp/images
```

**No images captured:**

```bash
# Problem: No HTTP traffic or all HTTPS
# Diagnosis
sudo tcpdump -i eth0 port 80 -c 10

# Solution: Check for HTTP traffic
# Most modern web traffic is HTTPS (encrypted)
# driftnet only works with unencrypted HTTP
```

**"Permission denied" error:**

```bash
# Problem: Insufficient privileges
# Solution: Run with sudo
sudo driftnet -i eth0 -m /tmp/images
```

**Images corrupted or incomplete:**

```bash
# Problem: Stream reassembly issues
# Solution: Capture full packets
sudo driftnet -i eth0 -m /tmp/images -v

# Check if traffic is being fragmented
sudo tcpdump -i eth0 "tcp[tcpflags] & (tcp-syn|tcp-fin) != 0" -c 100
```

### FAQ

**Q: Does driftnet work with HTTPS?**
A: No. driftnet only captures media from unencrypted HTTP traffic. HTTPS traffic is encrypted and cannot be inspected without decryption.

**Q: Can driftnet capture video?**
A: driftnet can capture some video formats (MPEG), but it's primarily designed for images and audio. For video, consider using mitmproxy or Wireshark.

**Q: Is driftnet passive?**
A: Yes. driftnet only reads packets from the network interface. It does not generate or inject any traffic.

**Q: What image formats are supported?**
A: JPEG, GIF, PNG, BMP. Audio formats include WAV and MP3.

**Q: Can driftnet capture images from encrypted traffic?**
A: No. driftnet cannot decrypt TLS/SSL traffic. Use a transparent proxy (mitmproxy) for encrypted traffic analysis.

---

## Resources

- **Kali Tools Page**: https://www.kali.org/tools/driftnet/
- **Related Tools**: tcpdump, mitmproxy, ngrep, urlsnarf
- **HTTP Security**: RFC 7230 (HTTP/1.1), RFC 8446 (TLS 1.3)
- **MITRE ATT&CK**: [TA0007 - Discovery](https://attack.mitre.org/tactics/TA0007/)
- **Network Forensics**: SANS Network Forensics poster
- **Image Forensics**: ExifTool, StegDetect
