# RecordMyDesktop - Desktop Recording Tool

## Complete Educational Guide for Security Professionals

---

## 1. Introduction

RecordMyDesktop is a simple desktop recording tool for Linux that captures screen activity and audio to create video files. For cybersecurity professionals, RecordMyDesktop serves as an essential tool for documenting penetration testing demonstrations, creating security training videos, recording exploit execution, and capturing evidence of vulnerabilities in action.

### 1.1 What is RecordMyDesktop?

RecordMyDesktop is a lightweight desktop recording application that:
- Records screen activity to video files
- Captures audio (system and microphone)
- Outputs OGG/Theora format
- Provides simple GTK interface
- Supports command-line operation

### 1.2 Project Information

| Property | Value |
|----------|-------|
| Developer | John Varouhakis |
| License | GNU GPL v2 |
| Language | C |
| Output Format | OGG/Theora/Vorbis |
| Platform | Linux (X11) |
| Dependencies | libvorbis, libtheora, libvorbisenc, libvorbisfile |

### 1.3 RecordMyDesktop vs Alternatives

| Feature | RecordMyDesktop | ffmpeg | OBS Studio | Kazam |
|---------|-----------------|--------|------------|-------|
| GUI | Yes | No | Yes | Yes |
| Ease of Use | Very Easy | Complex | Medium | Easy |
| Features | Basic | Advanced | Advanced | Medium |
| Output | OGG | Multiple | Multiple | Multiple |
| Audio | Yes | Yes | Yes | Yes |

### 1.4 Why Use RecordMyDesktop for Cybersecurity?

- **Evidence Capture:** Record exploit execution for reports
- **Training Videos:** Create security awareness content
- **Demonstration:** Show vulnerabilities to clients
- **Documentation:** Visual proof of findings
- **Collaboration:** Share findings with team
- **Education:** Create tutorial content

---

## 2. Installation

### 2.1 Kali Linux

```bash
# Install via apt
sudo apt update
sudo apt install gtk-recordmydesktop

# Launch from terminal
recordmydesktop

# Or from application menu
Applications > Sound & Video > gtk-recordMyDesktop
```

### 2.2 Command-Line Only

```bash
# Install CLI version
sudo apt install recordmydesktop

# Launch
recordmydesktop
```

### 2.3 Ubuntu/Debian

```bash
# Install dependencies
sudo apt install libvorbis-dev libtheora-dev \
    libvorbisenc-dev libvorbisfile-dev

# Download source
wget https://sourceforge.net/projects/recordmydesktop/files/recordmydesktop/0.3.8/recordmydesktop-0.3.8.tar.gz

# Extract and build
tar -xzf recordmydesktop-0.3.8.tar.gz
cd recordmydesktop-0.3.8
./configure
make
sudo make install
```

---

## 3. Interface Overview

### 3.1 GTK Interface

```
+------------------------------------------------------------------+
|  RecordMyDesktop                                                  |
+------------------------------------------------------------------+
|  [Select Area]  [Full Screen]  [Window Selection]               |
+------------------------------------------------------------------+
|  Sound Settings                                                   |
|  [x] Capture Sound  [x] Advanced Settings                       |
+------------------------------------------------------------------+
|  Video Settings                                                   |
|  Quality: [========|---]  FPS: [15]                             |
+------------------------------------------------------------------+
|  [Record]  [Settings]  [About]                                   |
+------------------------------------------------------------------+
```

### 3.2 Key Components

- **Select Area:** Choose recording region
- **Full Screen:** Record entire screen
- **Window Selection:** Record specific window
- **Sound Settings:** Audio capture options
- **Video Settings:** Quality and FPS
- **Record Button:** Start recording

---

## 4. Basic Usage

### 4.1 GUI Usage

1. Launch `gtk-recordmydesktop`
2. Select recording area or full screen
3. Configure audio settings
4. Click "Record"
5. Perform your demonstration
6. Click "Stop" in system tray

### 4.2 Command-Line Usage

```bash
# Basic recording
recordmydesktop

# Record to specific file
recordmydesktop -o output.ogv

# Record with audio
recordmydesktop --sound

# Record full screen
recordmydesktop --full-screen

# Record specific area
recordmydesktop -x 100 -y 100 --width 1920 --height 1080
```

---

## 5. Command-Line Options

### 5.1 Output Options

| Option | Description |
|--------|-------------|
| `-o FILE` | Output filename |
| `--on-the-fly` | Encode while recording |
| `--overwrite` | Overwrite existing file |

### 5.2 Display Options

| Option | Description |
|--------|-------------|
| `--full-screen` | Record full screen |
| `-x N` | X position |
| `-y N` | Y position |
| `--width N` | Recording width |
| `--height N` | Recording height |

### 5.3 Audio Options

| Option | Description |
|--------|-------------|
| `--sound` | Capture audio |
| `--no-sound` | No audio |
| `--device DEVICE` | Audio device |
| `--channels N` | Audio channels |

### 5.4 Video Options

| Option | Description |
|--------|-------------|
| `--fps N` | Frames per second (default: 15) |
| `--v-quality N` | Video quality (0-63) |
| `--freq N` | Audio frequency |

### 5.5 Control Options

| Option | Description |
|--------|-------------|
| `--pause-shortcut KEY` | Pause shortcut |
| `--stop-shortcut KEY` | Stop shortcut |
| `--delay N` | Delay before recording |

---

## 6. Recording Strategies

### 6.1 Full Screen Recording

```bash
# Record entire desktop
recordmydesktop --full-screen -o full_desktop.ogv

# With audio
recordmydesktop --full-screen --sound -o full_with_audio.ogv
```

### 6.2 Window Recording

```bash
# Record specific window
# 1. Find window ID
xwininfo

# 2. Record using window ID
recordmydesktop -window_id WINDOW_ID -o window.ogv
```

### 6.3 Area Recording

```bash
# Record specific region
recordmydesktop -x 0 -y 0 --width 800 --height 600 -o area.ogv

# Record terminal session
recordmydesktop -x 0 -y 0 --width 800 --height 400 -o terminal.ogv
```

### 6.4 Delayed Recording

```bash
# Start recording after 5 seconds
recordmydesktop --delay 5000 -o delayed.ogv

# Useful for preparation time
```

---

## 7. Security Demonstration Recording

### 7.1 Recording Exploit Execution

```bash
# Prepare recording
recordmydesktop --full-screen --sound -o exploit_demo.ogv &

# Perform exploit
msfconsole -q
use exploit/multi/handler
set PAYLOAD windows/meterpreter/reverse_tcp
set LHOST 192.168.1.100
exploit

# Stop recording
# Click system tray icon or use shortcut
```

### 7.2 Recording Vulnerability Scan

```bash
# Start recording
recordmydesktop -o vuln_scan.ogv &

# Run vulnerability scanner
nikto -h target.com
nmap -sV -sC target.com

# Stop recording when done
```

### 7.3 Recording Tool Usage

```bash
# Record Burp Suite session
recordmydesktop --full-screen -o burp_session.ogv &

# Open Burp Suite
# Configure proxy
# Perform testing

# Stop recording
```

---

## 8. Audio Settings

### 8.1 System Audio

```bash
# Record system audio (PulseAudio)
recordmydesktop --sound --device pulse -o system_audio.ogv

# Record ALSA
recordmydesktop --sound --device hw:0,0 -o alsa_audio.ogv
```

### 8.2 Microphone

```bash
# Record microphone
recordmydesktop --sound --device default -o mic.ogv

# Combined audio
recordmydesktop --sound --device pulse -o combined.ogv
```

### 8.3 No Audio

```bash
# Record without audio
recordmydesktop --no-sound -o video_only.ogv
```

---

## 9. Video Settings

### 9.1 Quality Settings

```bash
# High quality (larger file)
recordmydesktop --v-quality 63 -o high_quality.ogv

# Medium quality
recordmydesktop --v-quality 40 -o medium_quality.ogv

# Low quality (smaller file)
recordmydesktop --v-quality 10 -o low_quality.ogv
```

### 9.2 Frame Rate

```bash
# High FPS (smooth, larger file)
recordmydesktop --fps 30 -o high_fps.ogv

# Medium FPS
recordmydesktop --fps 15 -o medium_fps.ogv

# Low FPS (choppy, smaller file)
recordmydesktop --fps 5 -o low_fps.ogv
```

### 9.3 Resolution

```bash
# Full resolution
recordmydesktop --full-screen -o full_res.ogv

# Reduced resolution
recordmydesktop --width 1280 --height 720 -o hd.ogv

# Small resolution
recordmydesktop --width 640 --height 480 -o sd.ogv
```

---

## 10. Encoding and Conversion

### 10.1 On-the-Fly Encoding

```bash
# Encode while recording (uses more CPU)
recordmydesktop --on-the-fly -o realtime.ogv
```

### 10.2 Post-Processing

```bash
# Convert OGG to MP4
ffmpeg -i recording.ogv -c:v libx264 -c:a aac output.mp4

# Convert to WebM
ffmpeg -i recording.ogv -c:v libvpx -c:a libvorbis output.webm

# Compress video
ffmpeg -i recording.ogv -crf 23 -preset medium compressed.mp4
```

### 10.3 Extract Audio

```bash
# Extract audio track
ffmpeg -i recording.ogv -vn -acodec libmp3lame audio.mp3
```

---

## 11. Scripting and Automation

### 11.1 Automated Recording Script

```bash
#!/bin/bash
# auto_record.sh

OUTPUT=$1
DURATION=${2:-60}

if [ -z "$OUTPUT" ]; then
    echo "Usage: $0 <output_file> [duration_seconds]"
    exit 1
fi

# Start recording in background
recordmydesktop --full-screen --sound -o "$OUTPUT" &
RECORD_PID=$!

echo "Recording started: $OUTPUT"
echo "Duration: $DURATION seconds"
echo "Press Ctrl+C to stop early"

# Wait for duration or until killed
sleep "$DURATION"

# Stop recording
kill "$RECORD_PID" 2>/dev/null
echo "Recording saved: $OUTPUT"
```

### 11.2 Batch Recording

```bash
#!/bin/bash
# batch_record.sh

Demos=("exploit1" "scan1" "vuln1")
DURATION=60

for demo in "${Demos[@]}"; do
    echo "Recording: $demo"
    recordmydesktop --full-screen --sound -o "${demo}.ogv" &
    PID=$!
    sleep "$DURATION"
    kill "$PID"
    sleep 2
done

echo "All recordings complete!"
```

### 11.3 Keyboard Shortcut Control

```bash
# Set custom shortcuts
recordmydesktop --pause-shortcut Ctrl+Shift+P \
                --stop-shortcut Ctrl+Shift+S \
                -o recording.ogv
```

---

## 12. Best Practices

### 12.1 Recording Workflow

1. **Plan:** Know what you want to demonstrate
2. **Prepare:** Set up environment, clear desktop
3. **Record:** Start recording before action
4. **Perform:** Execute demonstration clearly
5. **Review:** Check recording quality
6. **Edit:** Trim unnecessary parts
7. **Export:** Convert to suitable format

### 12.2 Quality Tips

- Use adequate lighting (screen brightness)
- Record at appropriate resolution
- Use microphone for narration
- Keep mouse movements visible
- Perform actions slowly and clearly

### 12.3 File Management

```bash
# Organize recordings
mkdir -p recordings/$(date +%Y%m%d)

# Use descriptive names
recordmydesktop -o "$(date +%Y%m%d)_exploit_demo.ogv"

# Clean up old recordings
find . -name "*.ogv" -mtime +30 -delete
```

---

## 13. Troubleshooting

### Common Issues

**Problem:** Black screen recording

```bash
# Check X11 display
echo $DISPLAY

# Record specific display
recordmydesktop -display :0 -o output.ogv
```

**Problem:** No audio

```bash
# List audio devices
arecord -l

# Specify device
recordmydesktop --sound --device hw:0,0 -o output.ogv

# Check PulseAudio
pactl list sources
```

**Problem:** Choppy video

```bash
# Reduce quality
recordmydesktop --v-quality 20 -o output.ogv

# Reduce FPS
recordmydesktop --fps 10 -o output.ogv

# Reduce resolution
recordmydesktop --width 1280 --height 720 -o output.ogv
```

**Problem:** File too large

```bash
# Reduce quality
recordmydesktop --v-quality 30 -o output.ogv

# Convert to compressed format
ffmpeg -i output.ogv -c:v libx264 -crf 28 compressed.mp4
```

---

## References

- [RecordMyDesktop Website](http://recordmydesktop.sourceforge.net/)
- [RecordMyDesktop Documentation](http://recordmydesktop.sourceforge.net/docs.html)
- [Kali Linux RecordMyDesktop Package](https://www.kali.org/tools/recordmydesktop/)
- [FFmpeg Documentation](https://ffmpeg.org/ffmpeg.html)
