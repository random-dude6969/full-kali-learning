---
tool_name: "gqrx"
display_name: "Gqrx SDR"
mitre_tactic: "reconnaissance"
mitre_tactic_id: "TA0043"
mitre_subcategory: "radio_frequency"
install_command: "sudo apt install gqrx-sdr"
version: "2.17.5"
github_repo: "https://github.com/gqrx-sdr/gqrx"
kali_tools_page: "https://www.kali.org/tools/gqrx-sdr/"
difficulty_level: "beginner"
requires_root: false
gui_available: true
tags: ["sdr", "radio", "receiver", "reconnaissance", "spectrum", "gui"]
related_tools: ["hackrf", "rfcat", "gnuradio", "sdrangel", "sdrr"]
---

# Gqrx SDR — Open Source Software Defined Radio Receiver

## Chapter 1: Introduction & Overview

### What is Gqrx?

Gqrx is a free, open-source Software Defined Radio (SDR) receiver powered by GNU Radio and Qt. Created by Alexandru Csete (OZ9AEC) in 2011, it provides a graphical interface for receiving and demodulating radio signals across a wide frequency range, making SDR accessible to both hobbyists and professionals.

The name "Gqrx" derives from "G" (GNU Radio) and "qrx" (Q-code meaning "shall I wait?").

### Development Timeline

- **2011**: Initial release as basic FM receiver
- **2012-2013**: Multiple demodulation modes, HackRF support
- **2014-2016**: Remote control, Airspy support, SoapySDR integration
- **2017-2019**: Qt5 migration, macOS support, GNU Radio 3.8 compatibility
- **2020-2024**: Qt6 support, continuous improvements

### Key Features

**Hardware Support:**

| Device | Interface | Frequency Range | Sample Rate |
|--------|-----------|-----------------|-------------|
| RTL-2832U | librtlsdr | 24-1766 MHz | Up to 2.4 MSPS |
| HackRF One | libhackrf | 1-6000 MHz | Up to 20 MSPS |
| Airspy | libairspy | 24-1800 MHz | Up to 10 MSPS |
| USRP (various) | UHD | DC-6 GHz | Device dependent |
| SoapySDR | SoapySDR | Device dependent | Device dependent |
| BladeRF | libbladeRF | 300-3800 MHz | Up to 40 MSPS |

**Demodulation Modes:**
- AM (Amplitude Modulation) — Airband, AM broadcast
- FM (Narrowband FM) — Ham radio, business band
- WFM (Wideband FM) — Broadcast FM radio
- LSB/USB (Sideband) — Amateur radio HF
- CW (Continuous Wave) — Morse code, beacons
- Raw (IQ output) — External processing

**Signal Processing:**
- Real-time FFT spectrum display with waterfall spectrogram
- Noise reduction, squelch control, AGC
- Variable bandwidth filters
- Audio and IQ recording (WAV/raw formats)
- TCP remote control interface on port 7356

### Architecture

```
┌─────────────────────────────────────────────────┐
│                  Gqrx GUI (Qt)                  │
├─────────────────────────────────────────────────┤
│              GNU Radio Signal Processing         │
├─────────────────────────────────────────────────┤
│           Device Drivers (UHD, RTL, etc.)       │
├─────────────────────────────────────────────────┤
│               SDR Hardware                       │
└─────────────────────────────────────────────────┘
```

### MITRE ATT&CK Mapping

| Tactic | Technique | ID |
|--------|-----------|-----|
| Reconnaissance | Active Scanning: Wireless Spectrum Analysis | T1590.004 |
| Reconnaissance | Gather Victim Network Information | T1590 |
| Collection | Data from Wireless Capture | T1005 |

### Use Cases

- **Amateur Radio**: Monitor ham bands, receive digital modes, track APRS
- **Signal Intelligence**: Spectrum monitoring, signal identification
- **Education**: Learning RF concepts, DSP demonstrations
- **Broadcast Reception**: FM/AM radio, weather radio, airband
- **Professional**: RF environment assessment, interference hunting

## Chapter 2: Installation & Setup

### Method 1: Package Manager (Recommended)

```bash
# Ubuntu/Debian/Kali
sudo apt update
sudo apt install gqrx-sdr

# Install common SDR tools alongside
sudo apt install rtl-sdr hackrf soapysdr-tools

# Verify
gqrx --version
```

### Method 2: AppImage (Portable)

```bash
mkdir -p ~/sdr-tools && cd ~/sdr-tools
wget https://github.com/gqrx-sdr/gqrx/releases/download/v2.17.5/gqrx-2.17.5-x86_64.AppImage
chmod +x gqrx-2.17.5-x86_64.AppImage
./gqrx-2.17.5-x86_64.AppImage
```

### Method 3: Build from Source

```bash
# Install dependencies
sudo apt install -y build-essential cmake git libboost-all-dev \
    libcppunit-dev liblog4cpp5-dev libfftw3-dev libqt5opengl5-dev \
    libqt5svg5-dev libqt5x11extras5-dev libusb-1.0-0-dev \
    libsoapysdr-dev libpulse-dev gnuradio-dev gr-osmosdr

# Clone and build
git clone https://github.com/gqrx-sdr/gqrx.git
cd gqrx && mkdir build && cd build
cmake -DCMAKE_INSTALL_PREFIX=/usr -DCMAKE_BUILD_TYPE=Release ..
make -j$(nproc)
sudo make install
```

### Device Driver Installation

```bash
# RTL-SDR
sudo apt install rtl-sdr librtlsdr-dev
sudo bash -c 'echo "blacklist dvb_usb_rtl28xxu" > /etc/modprobe.d/blacklist-rtlsdr.conf'
sudo rmmod dvb_usb_rtl28xxu 2>/dev/null
rtl_test -t

# HackRF
sudo apt install hackrf libhackrf-dev
sudo usermod -a -G plugdev $USER
hackrf_info

# Airspy
sudo apt install airspy libairspy-dev
airspy_info

# USRP
sudo apt install uhd-host libuhd-dev uhd-examples
uhd_find_devices
```

### udev Rules

```bash
sudo tee /etc/udev/rules.d/99-sdr.rules << 'EOF'
# RTL-SDR
SUBSYSTEM=="usb", ATTRS{idVendor}=="0bda", MODE="0666"
# HackRF
SUBSYSTEM=="usb", ATTRS{idVendor}=="1d50", MODE="0666"
# Airspy
SUBSYSTEM=="usb", ATTRS{idVendor}=="1d50", MODE="0666"
# BladeRF
SUBSYSTEM=="usb", ATTRS{idVendor}=="2cf0", MODE="0666"
# USRP
SUBSYSTEM=="usb", ATTRS{idVendor}=="2500", MODE="0666"
EOF
sudo udevadm control --reload-rules && sudo udevadm trigger
```

### First Launch

1. Connect SDR device via USB
2. Launch: `gqrx`
3. Select device from I/O device dropdown
4. Set sample rate appropriate for device
5. Click OK, then Power button to start receiving

### Configuration

- Config directory: `~/.config/gqrx/`
- Key files: `gqrx.conf`, `bookmarks.csv`, `session.conf`
- Automatic save of last session state

## Chapter 3: Basic Usage

### Interface Overview

```
┌─────────────────────────────────────────────────────────────┐
│  Menu Bar: File, View, Tools, DSP, Extras, Help            │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────────────────────────────────────────────────┐│
│  │  FFT Display (Spectrum/Waterfall)                      ││
│  │  Click to tune, mouse wheel to zoom                    ││
│  └─────────────────────────────────────────────────────────┘│
├─────────────────────────────────────────────────────────────┤
│  [Power] [145.800 000] Hz  [AM] [Squelch] [Volume]        │
│  [AF Gain] [RF Gain] [LNB LO] [Signal Meter]              │
├─────────────────────────────────────────────────────────────┤
│  Remote Control: TCP port 7356                              │
│  Status: Streaming | Device: rtlsdr | Sample Rate: 2.4 MSPS│
└─────────────────────────────────────────────────────────────┘
```

### Frequency Tuning

**Methods:**
- Direct entry: Click frequency display, type, press Enter
- Keyboard: Up/Down (100 Hz), Page (1 kHz), Shift+Up/Down (10 kHz), Ctrl+Up/Down (100 kHz)
- Mouse: Click on waterfall, scroll wheel on frequency

**Number Pad Entry:** Type digits directly (e.g., `145800000` for 145.8 MHz)

### Demodulation Modes

| Mode | Shortcut | Use Case | Bandwidth |
|------|----------|----------|-----------|
| AM | `1` | Airband, AM broadcast | 6-10 kHz |
| FM | `2` | Ham radio, business | 12.5-25 kHz |
| WFM | `3` | Broadcast FM radio | 180-200 kHz |
| LSB | `4` | Amateur HF | 2.4-3 kHz |
| USB | `5` | Amateur HF, satellite | 2.4-3 kHz |
| CW | `6` | Morse code, beacons | 200-500 Hz |
| Raw | `7` | Raw IQ output | - |

### Squelch Control

- `-inf`: Squelch off (always audio)
- `-50 dB`: Very sensitive
- `-30 dB`: Moderate sensitivity
- `-10 dB`: Only strong signals

**Procedure:** Tune to quiet frequency, increase squelch until audio mutes — this is just above noise floor.

### Basic Operations

**FM Broadcast Radio:**
1. Tune to 100 MHz (example)
2. Set mode to WFM
3. Adjust AF Gain volume

**Amateur Radio:**
1. Tune to 146.940 MHz (2m repeater)
2. Set mode to FM
3. Set squelch appropriately

**Airband Monitoring:**
1. Tune to 118.1 MHz (ATIS)
2. Set mode to AM
3. Set bandwidth to ~8 kHz

### Keyboard Shortcuts

| Key | Action |
|-----|--------|
| `1-7` | Select demodulation mode |
| `Space` | Toggle squelch bypass |
| `Ctrl+R` | Start/stop recording |
| `Ctrl+D` | Device settings |
| `Up/Down` | Tune 100 Hz steps |
| `Page Up/Down` | Tune 1 kHz steps |

## Chapter 4: Intermediate Usage

### Audio Recording

```bash
# Start recording: Ctrl+R or Audio Recording button
# Format: WAV, 48000 Hz
# Location: Configured in Tools -> Audio Recording
```

### IQ Recording

```bash
# Record raw IQ: File -> Record I/Q or Ctrl+I
# Formats: CU8, CS16, CF32
# Storage: ~1.1 GB/min at 2.4 MSPS
```

### Bookmarks

```bash
# Add: Right-click spectrum -> Add Bookmark
# Edit: Tools -> Bookmarks -> Edit
# File: ~/.config/gqrx/bookmarks.csv
# Format: frequency,name,modulation,bandwidth,active
```

### Remote Control

Enable via Tools -> Remote Control (default port 7356):

```bash
# Telnet interface
telnet localhost 7356
F 100500000    # Set frequency
f              # Get frequency
M FM 12500     # Set mode
m              # Get mode
L SQL 50       # Set squelch
l              # Get signal level
```

### Python Remote Control

```python
import socket

class GQRXController:
    def __init__(self, host="localhost", port=7356):
        self.sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        self.sock.connect((host, port))

    def set_frequency(self, freq):
        self.sock.send(f"F {freq}\n".encode())
        return self.sock.recv(1024).decode().strip()

    def get_frequency(self):
        self.sock.send(b"f\n")
        return self.sock.recv(1024).decode().strip()

    def set_mode(self, mode, bandwidth):
        self.sock.send(f"M {mode} {bandwidth}\n".encode())
        return self.sock.recv(1024).decode().strip()

    def get_signal_level(self):
        self.sock.send(b"l\n")
        return self.sock.recv(1024).decode().strip()

    def close(self):
        self.sock.close()

# Usage
gqrx = GQRXController()
gqrx.set_frequency(100500000)  # 100.5 MHz
gqrx.set_mode("WFM", 200000)   # Wide FM
print(f"Frequency: {gqrx.get_frequency()}")
gqrx.close()
```

### FFT Display Adjustments

- **FFT Size**: Larger = better resolution, more CPU (typical: 4096-8192)
- **Frame Rate**: Higher = smoother waterfall (typical: 15-30 fps)
- **dB Range**: Adjust Y-axis scale for signal visibility
- **Color Scheme**: View -> Waterfall Colors (Classic, Turbo, Rainbow)

## Chapter 5: Advanced Usage

### Band Scanner

```python
import socket
import time
import csv
from datetime import datetime

class BandScanner:
    def __init__(self, host='localhost', port=7356):
        self.sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        self.sock.connect((host, port))
        self.sock.settimeout(0.5)

    def send(self, cmd):
        self.sock.sendall((cmd + '\n').encode())
        try:
            return self.sock.recv(1024).decode().strip()
        except:
            return None

    def scan_band(self, name, start_mhz, end_mhz, step_khz, mode, dwell_ms=100):
        print(f"\nScanning {name} ({start_mhz}-{end_mhz} MHz)")
        self.send(f'MOD {mode}')
        results = []
        freq = start_mhz

        while freq <= end_mhz:
            freq_hz = int(freq * 1e6)
            self.send(f'FREQ {freq_hz}')
            time.sleep(dwell_ms / 1000.0)
            results.append({
                'timestamp': datetime.now().isoformat(),
                'band': name,
                'frequency_mhz': freq,
                'mode': mode
            })
            freq += step_khz / 1000.0

        return results

# Usage
scanner = BandScanner()
scanner.scan_band('Airband', 118.0, 137.0, 25, 'AM')
scanner.scan_band('VHF', 144.0, 148.0, 12.5, 'FM')
```

### Signal Monitor

```python
import time
import json
from datetime import datetime

class SignalMonitor:
    def __init__(self, host='localhost', port=7356):
        self.sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        self.sock.connect((host, port))
        self.history = []

    def monitor_frequency(self, freq, duration=60, interval=1.0):
        self.sock.send(f'F {freq}\n'.encode())
        start_time = time.time()
        while time.time() - start_time < duration:
            self.sock.send(b'l\n')
            signal = self.sock.recv(1024).decode().strip()
            self.history.append({
                'timestamp': datetime.now().isoformat(),
                'frequency': freq,
                'signal': float(signal) if signal else 0
            })
            time.sleep(interval)

    def get_statistics(self):
        signals = [h['signal'] for h in self.history]
        return {
            'min_signal': min(signals),
            'max_signal': max(signals),
            'avg_signal': sum(signals) / len(signals)
        }

monitor = SignalMonitor()
monitor.monitor_frequency(100500000, duration=300)
print(monitor.get_statistics())
```

### Custom Decoders

```python
import numpy as np
from scipy import signal as sig

class FMDecoder:
    def __init__(self, samp_rate=2000000):
        self.samp_rate = samp_rate

    def decode(self, iq_data):
        signal = iq_data[1:] * np.conj(iq_data[:-1])
        demodulated = np.angle(signal)
        b, a = sig.butter(5, 150000 / self.samp_rate)
        audio = sig.lfilter(b, a, demodulated)
        return audio
```

### Spectrum Analysis

```python
import numpy as np
import matplotlib.pyplot as plt

def analyze_spectrum(filename, samp_rate=2000000):
    data = np.fromfile(filename, dtype=np.float32)
    fft = np.fft.fftshift(np.fft.fft(data))
    freqs = np.fft.fftshift(np.fft.fftfreq(len(data), 1/samp_rate))
    magnitude = 20 * np.log10(np.abs(fft) + 1e-10)

    threshold = np.max(magnitude) - 30
    signals = np.where(magnitude > threshold)[0]

    plt.figure(figsize=(12, 6))
    plt.plot(freqs/1e6, magnitude)
    plt.xlabel('Frequency (MHz)')
    plt.ylabel('Magnitude (dB)')
    plt.title('Spectrum Analysis')
    plt.grid(True)
    plt.savefig('spectrum.png')

    return freqs[signals]/1e6
```

## Chapter 6: Real-World Applications

### Aviation Monitoring

| Service | Frequency (MHz) | Mode |
|---------|-----------------|------|
| Tower | 118.0-121.9 | AM |
| ATIS | 118.0-136.975 | AM |
| Ground | 121.7-121.9 | AM |
| Emergency | 121.5 | AM |

```bash
# Setup for ATIS
FREQ 120200000   # Boston Logan ATIS
MOD AM
# Bandwidth: 8000
SQL -35
```

### Maritime Monitoring

| Channel | Frequency (MHz) | Use |
|---------|-----------------|-----|
| 16 | 156.800 | Distress/Safety |
| 13 | 156.650 | Bridge-to-bridge |
| 09 | 156.450 | Calling |

### Weather Monitoring

```bash
# NOAA Weather Radio (162.400-162.550 MHz)
FREQ 162550000
MOD FM
SQL -30

# Weather Satellite (NOAA APT)
# NOAA 15: 137.620 MHz, NOAA 18: 137.9125 MHz, NOAA 19: 137.100 MHz
FREQ 137912500  # NOAA 18
MOD FM
```

### ADS-B Aircraft Tracking

```bash
FREQ 1090000000  # 1090 MHz
MOD AM
# Use dump1090 or readsb for decoding
```

### Band Scanning Scripts

```python
#!/usr/bin/env python3
"""Multi-band scanner with logging"""

BANDS = [
    {'name': 'Airband Low', 'start': 118.0, 'end': 128.0, 'step': 25, 'mode': 'AM'},
    {'name': 'Airband High', 'start': 128.0, 'end': 137.0, 'step': 25, 'mode': 'AM'},
    {'name': 'VHF Low', 'start': 144.0, 'end': 148.0, 'step': 12.5, 'mode': 'FM'},
    {'name': 'UHF', 'start': 420.0, 'end': 450.0, 'step': 25, 'mode': 'FM'},
]

scanner = BandScanner()
scanner.connect()
for band in BANDS:
    scanner.scan_band(**band)
```

## Chapter 7: Detection & Defense

### RF Security Fundamentals

**Common RF Surveillance Methods:**
- Passive listening to unencrypted communications
- Direction finding and geolocation
- Signal fingerprinting
- Traffic analysis

**Legal Considerations:** Unauthorized interception of communications is illegal in most jurisdictions. Always obtain written authorization before RF testing.

### Detecting Hidden Transmitters

```bash
# Scan for covert devices
# Common frequencies: 49 MHz, 900 MHz, 1.2 GHz, 2.4 GHz, 5.8 GHz

# In Gqrx: Set wide bandwidth, look for anomalies
FREQ 2400000000  # 2.4 GHz
MOD RAW
Sample Rate: 10 MSPS
```

### Signal Identification

| Modulation | Visual Pattern | Typical Use |
|------------|----------------|-------------|
| AM | Carrier with sidebands | Airband, AM broadcast |
| FM | Constant amplitude | Ham radio, business |
| WFM | Wide constant amplitude | FM broadcast |
| SSB | Asymmetric, no carrier | Ham HF |
| FSK | Two distinct frequencies | Data modes |
| OFDM | Wide, flat spectrum | WiFi, digital TV |

### WiFi and Bluetooth Monitoring

```bash
# 2.4 GHz WiFi band
FREQ 2437000000  # Center of 2.4 GHz band
MOD RAW
Sample Rate: 10 MSPS

# Bluetooth (frequency hopping, 79 channels)
FREQ 2441000000  # Center of Bluetooth band
MOD RAW
Sample Rate: 10 MSPS
```

### Counter-Surveillance Scan

```python
#!/usr/bin/env python3
"""Automated RF monitoring for security"""

import socket
import time
import logging
from datetime import datetime

class RFMonitor:
    def __init__(self, host='localhost', port=7356):
        self.sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        self.sock.connect((host, port))
        self.baseline = {}

    def send(self, cmd):
        self.sock.sendall((cmd + '\n').encode())
        try:
            return self.sock.recv(1024).decode().strip()
        except:
            return None

    def establish_baseline(self, frequencies):
        for freq_info in frequencies:
            self.send(f'FREQ {freq_info["freq"]}')
            self.send(f'MOD {freq_info["mode"]}')
            time.sleep(0.5)
            self.baseline[freq_info['freq']] = self.send('RSSI?')

    def detect_changes(self, frequencies, threshold_db=6):
        changes = []
        for freq_info in frequencies:
            self.send(f'FREQ {freq_info["freq"]}')
            time.sleep(0.5)
            current = self.send('RSSI?')
            baseline = self.baseline.get(freq_info['freq'])
            if baseline and current:
                diff = abs(float(current) - float(baseline))
                if diff > threshold_db:
                    changes.append({
                        'frequency': freq_info['freq'],
                        'change_db': diff
                    })
        return changes

# Usage
monitor = RFMonitor()
frequencies = [
    {'freq': 49000000, 'mode': 'AM'},
    {'freq': 900000000, 'mode': 'FM'},
    {'freq': 2400000000, 'mode': 'RAW'},
]
monitor.establish_baseline(frequencies)
changes = monitor.detect_changes(frequencies)
```

### Wireless Camera Detection

| Frequency | Camera Type |
|-----------|------------|
| 900 MHz | Older wireless cameras |
| 1.2 GHz | Analog video transmitters |
| 2.4 GHz | WiFi cameras, analog video |
| 5.8 GHz | WiFi cameras, analog video |

### Tracking Device Detection

- **GPS Trackers**: Monitor 1575.42 MHz (GPS L1) for nearby transmitters
- **RFID/NFC**: LF 125 kHz, HF 13.56 MHz, UHF 860-960 MHz

## Chapter 8: Troubleshooting & Resources

### Common Errors

**"No device found"**
```bash
lsusb | grep -i "rtl\|hackrf\|airspy"
# RTL-SDR: Blacklist DVB-T drivers
sudo rmmod dvb_usb_rtl28xxu 2>/dev/null
rtl_test -t
# Check SoapySDR
SoapySDRUtil --find
```

**"Device not responding"**
```bash
# Kill competing processes
pkill -f "rtl_sdr\|rtl_test\|hackrf"
# Try different USB port (USB 2.0 often more reliable)
```

**Audio issues**
```bash
pactl list sinks short
pactl set-default-sink @DEFAULT_SINK@
```

**FFT display issues**
```bash
export QT_QPA_PLATFORM=xcb
export LIBGL_ALWAYS_SOFTWARE=1
```

**Performance issues**
```bash
# Reduce sample rate
# Reduce FFT size
# Close other programs
sensors  # Check CPU temperature
```

**Gqrx won't start**
```bash
ldd $(which gqrx) | grep "not found"
rm -f ~/.config/gqrx/*.conf  # Reset config
export QT_DEBUG_PLUGINS=1
gqrx 2>&1 | head -50
```

### Device-Specific Issues

**RTL-SDR:**
- Use R820T/R820T2 based dongles
- Adjust PPM offset via Tools -> I/Q Correction
- Enable direct sampling for HF (0-24 MHz)

**HackRF:**
- Half-duplex only (cannot TX+RX simultaneously)
- Max sample rate: 20 MS/s
- Use powered USB hub if power issues

### FAQ

**Q: How do I update Gqrx?**
```bash
sudo apt update && sudo apt upgrade gqrx
```

**Q: How do I use multiple devices?**
A: Gqrx supports one device at a time. Use SoapySDR for device selection: `gqrx -d "driver=rtlsdr,serial=00000001"`

**Q: How do I record IQ data?**
A: File -> Record I/Q or Ctrl+I. Use CU8 format for compatibility with other tools.

**Q: How do I save frequency bookmarks?**
A: Stored in `~/.config/gqrx/bookmarks.csv`. Edit via Tools -> Bookmarks -> Edit.

### Resources

- **Website**: https://gqrx.dk
- **Source Code**: https://github.com/gqrx-sdr/gqrx
- **Documentation**: https://github.com/gqrx-sdr/gqrx/wiki
- **GNU Radio**: https://www.gnuradio.org
- **RTL-SDR Blog**: https://www.rtl-sdr.com
- **Reddit r/RTLSDR**: https://www.reddit.com/r/RTLSDR/
- **Amateur Radio**: https://www.arrl.org

### License

Gqrx is released under the GNU General Public License (GPL) version 3.
