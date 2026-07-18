# Gqrx SDR — Quick Reference Cheatsheet

## Installation
```bash
sudo apt install gqrx-sdr        # Install
gqrx --version                   # Verify
gqrx                             # Launch
```

## Launch Options
```bash
gqrx                             # Default launch
gqrx --args "driver=rtlsdr"     # Specific device
gqrx --args "hackrf=0"          # HackRF device
gqrx -c ~/.config/gqrx/gqrx.conf  # Load config
```

## Device Selection
| Device | I/O Setting |
|--------|------------|
| RTL-SDR | `rtl=0` |
| HackRF | `hackrf=0` |
| Airspy | `airspy=0` |
| USRP | `uhd` |
| SoapySDR | `soapy=0` |

## Sample Rates
| Device | Recommended |
|--------|-------------|
| RTL-SDR | 1.0, 1.2, 1.6, 2.0, 2.4 MSPS |
| HackRF | 2.0, 4.0, 8.0, 10.0, 16.0 MSPS |
| Airspy | 2.5, 5.0, 6.0, 10.0 MSPS |

## Keyboard Shortcuts

### Frequency
| Key | Action |
|-----|--------|
| `Up/Down` | Tune 100 Hz |
| `Page Up/Down` | Tune 1 kHz |
| `Shift+Up/Down` | Tune 10 kHz |
| `Ctrl+Up/Down` | Tune 100 kHz |

### Modes
| Key | Mode |
|-----|------|
| `1` | AM |
| `2` | FM |
| `3` | WFM |
| `4` | LSB |
| `5` | USB |
| `6` | CW |
| `7` | Raw |

### General
| Key | Action |
|-----|--------|
| `Space` | Toggle squelch bypass |
| `Ctrl+R` | Start/stop recording |
| `Ctrl+D` | Device settings |
| `+/-` | Volume up/down |

## Demodulation Modes Quick Guide
| Mode | Use Case | Bandwidth |
|------|----------|-----------|
| AM | Airband, AM broadcast | 6-10 kHz |
| FM | Ham radio, business | 12.5-25 kHz |
| WFM | Broadcast FM | 180-200 kHz |
| LSB/USB | Amateur HF | 2.4-3 kHz |
| CW | Morse code | 200-500 Hz |

## Squelch Guide
| Level | Description |
|-------|-------------|
| -inf | Off (always audio) |
| -50 dB | Very sensitive |
| -30 dB | Moderate |
| -10 dB | Strong signals only |

## Remote Control (Port 7356)
```bash
# Enable: Tools -> Remote Control
telnet localhost 7356

F 100500000      # Set frequency
f                # Get frequency
M FM 12500       # Set mode
m                # Get mode
L SQL 50         # Set squelch
l                # Get signal level
```

## Frequency Reference
| Service | Freq (MHz) | Mode |
|---------|------------|------|
| FM Broadcast | 88-108 | WFM |
| Airband | 118-137 | AM |
| NOAA Weather | 162.4-162.55 | FM |
| 2m Ham | 144-148 | FM |
| 70cm Ham | 420-450 | FM |
| Marine VHF | 156-162 | FM |
| ADS-B | 1090 | AM |

## Common Scenarios

### FM Radio
```
Freq: 100000000 (100 MHz)
Mode: WFM
```

### Airband ATIS
```
Freq: 120200000 (120.2 MHz)
Mode: AM
Bandwidth: 8000
SQL: -35
```

### Ham Repeater
```
Freq: 146940000 (146.94 MHz)
Mode: FM
SQL: -35
```

### NOAA Weather
```
Freq: 162550000 (162.55 MHz)
Mode: FM
SQL: -30
```

## Python Remote Control
```python
import socket

sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
sock.connect(("localhost", 7356))
sock.send(b"F 100500000\n")  # Set 100.5 MHz
sock.send(b"M WFM 200000\n") # Wide FM
sock.send(b"f\n")             # Query frequency
print(sock.recv(1024).decode())
```

## Recording
```bash
# Audio: Ctrl+R (WAV format, 48kHz)
# IQ: File -> Record I/Q (CU8/CS16/CF32)
# Storage: ~1.1 GB/min at 2.4 MSPS
```

## Bookmarks
```bash
# Add: Right-click spectrum -> Add Bookmark
# Edit: Tools -> Bookmarks -> Edit
# File: ~/.config/gqrx/bookmarks.csv
# Format: frequency,name,modulation,bandwidth,active
```

## Troubleshooting
| Problem | Solution |
|---------|----------|
| No device found | Check USB, blacklist DVB drivers |
| Device not responding | Kill competing SDR processes |
| No audio | Check PulseAudio: `pactl list sinks short` |
| Blank waterfall | Reduce FFT size, check GPU drivers |
| High CPU | Reduce sample rate, close other apps |
| Gqrx won't start | Reset config: `rm -rf ~/.config/gqrx/` |

## Config Location
```bash
~/.config/gqrx/gqrx.conf         # Main config
~/.config/gqrx/bookmarks.csv     # Saved frequencies
~/.config/gqrx/session.conf      # Last session
```

## Related Tools
| Tool | Purpose |
|------|---------|
| `hackrf` | SDR transceiver |
| `rtl_sdr` | RTL-SDR command line |
| `gnuradio` | Visual signal processing |
| `sdrangel` | Multi-band SDR receiver |
| `dump1090` | ADS-B decoder |

## Legal
- Receiving publicly broadcast signals is generally legal
- Unauthorized interception of private communications is illegal
- Always check local regulations before monitoring
