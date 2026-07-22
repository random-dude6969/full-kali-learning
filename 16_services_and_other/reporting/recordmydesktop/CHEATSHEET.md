# RecordMyDesktop - Quick Reference Cheatsheet

---

## Quick Start

```bash
# Launch GUI
gtk-recordmydesktop

# Basic recording
recordmydesktop

# Record with audio
recordmydesktop --sound -o output.ogv

# Record full screen
recordmydesktop --full-screen -o fullscreen.ogv
```

---

## Default Configuration

| Setting | Value |
|---------|-------|
| Output Format | OGG/Theora |
| Default FPS | 15 |
| Default Quality | 40 |
| Audio Device | default |
| Platform | Linux (X11) |

---

## Command-Line Options

### Output
| Option | Description |
|--------|-------------|
| `-o FILE` | Output filename |
| `--on-the-fly` | Encode while recording |
| `--overwrite` | Overwrite existing file |

### Display
| Option | Description |
|--------|-------------|
| `--full-screen` | Record full screen |
| `-x N` | X position |
| `-y N` | Y position |
| `--width N` | Recording width |
| `--height N` | Recording height |
| `-display :N` | X display |

### Audio
| Option | Description |
|--------|-------------|
| `--sound` | Capture audio |
| `--no-sound` | No audio |
| `--device DEV` | Audio device |
| `--channels N` | Audio channels |

### Video
| Option | Description |
|--------|-------------|
| `--fps N` | Frames per second |
| `--v-quality N` | Video quality (0-63) |
| `--freq N` | Audio frequency |

### Control
| Option | Description |
|--------|-------------|
| `--delay N` | Delay before recording (ms) |
| `--pause-shortcut KEY` | Pause shortcut |
| `--stop-shortcut KEY` | Stop shortcut |

---

## Common Use Cases

```bash
# Record terminal session
recordmydesktop -x 0 -y 0 --width 800 --height 400 -o terminal.ogv

# Record exploit demo
recordmydesktop --full-screen --sound -o exploit.ogv

# Record with custom FPS
recordmydesktop --fps 30 -o smooth.ogv

# High quality recording
recordmydesktop --v-quality 63 --fps 30 -o hq.ogv

# Low quality (small file)
recordmydesktop --v-quality 10 --fps 10 -o lq.ogv

# Delayed start
recordmydesktop --delay 5000 -o delayed.ogv
```

---

## Automated Recording Script

```bash
#!/bin/bash
OUTPUT=$1
DURATION=${2:-60}

recordmydesktop --full-screen --sound -o "$OUTPUT" &
PID=$!
sleep "$DURATION"
kill $PID
echo "Saved: $OUTPUT"
```

---

## Format Conversion

```bash
# OGG to MP4
ffmpeg -i recording.ogv -c:v libx264 -c:a aac output.mp4

# OGG to WebM
ffmpeg -i recording.ogv -c:v libvpx -c:a libvorbis output.webm

# Compress video
ffmpeg -i recording.ogv -crf 23 -preset medium compressed.mp4

# Extract audio
ffmpeg -i recording.ogv -vn -acodec libmp3lame audio.mp3

# Resize video
ffmpeg -i recording.ogv -vf scale=1280:720 resized.ogv
```

---

## Keyboard Shortcuts

| Shortcut | Action |
|----------|--------|
| Ctrl+Shift+P | Pause recording |
| Ctrl+Shift+S | Stop recording |
| Click tray icon | Stop recording |

---

## Recording Workflow

```bash
# 1. Prepare environment
# Clear desktop, set up tools

# 2. Start recording
recordmydesktop --full-screen --sound -o demo.ogv &

# 3. Perform demonstration
# Execute commands, show vulnerabilities

# 4. Stop recording
# Use keyboard shortcut or tray icon

# 5. Convert if needed
ffmpeg -i demo.ogv -c:v libx264 demo.mp4
```

---

## Quality Settings

| Quality | FPS | Use Case |
|---------|-----|----------|
| 63 (High) | 30 | Final demos |
| 40 (Medium) | 15 | General use |
| 10 (Low) | 10 | Quick captures |

---

## Audio Devices

```bash
# List devices
arecord -l

# PulseAudio
recordmydesktop --sound --device pulse -o output.ogv

# ALSA
recordmydesktop --sound --device hw:0,0 -o output.ogv
```

---

## Troubleshooting

```bash
# Check display
echo $DISPLAY

# List audio devices
arecord -l
pactl list sources

# Fix black screen
recordmydesktop -display :0 -o output.ogv

# Reduce file size
recordmydesktop --v-quality 20 --fps 10 -o small.ogv

# Convert to smaller format
ffmpeg -i output.ogv -c:v libx264 -crf 28 compressed.mp4
```
