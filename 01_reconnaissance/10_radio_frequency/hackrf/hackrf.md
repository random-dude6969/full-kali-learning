---
tool_name: "hackrf"
display_name: "HackRF One"
mitre_tactic: "reconnaissance"
mitre_tactic_id: "TA0043"
mitre_subcategory: "radio_frequency"
install_command: "sudo apt install hackrf"
version: "2023.01.1"
github_repo: "https://github.com/greatscottgadgets/hackrf"
kali_tools_page: "https://www.kali.org/tools/hackrf/"
difficulty_level: "intermediate"
requires_root: false
gui_available: false
tags: ["sdr", "radio", "hackrf", "reconnaissance", "rf", "transceiver"]
related_tools: ["gqrx", "rfcat", "gnuradio", "inspectrum", "urh", "sdrangel"]
---

# HackRF One — Software Defined Radio Platform

## Chapter 1: Introduction & Overview

### What is HackRF One?

HackRF One is an open-source Software Defined Radio (SDR) peripheral developed by Great Scott Gadgets (Michael Ossmann). It serves as both a radio transmitter and receiver, enabling exploration of the RF spectrum from 1 MHz to 6 GHz. Unlike traditional radios locked to specific protocols, HackRF One allows users to transmit and receive radio signals through software.

### History and Development

- **2012**: Original HackRF prototype developed, Kickstarter campaign raised $300,000+
- **2013**: First production units shipped
- **2014**: HackRF One released with improved specifications
- **2015-2020**: Community-driven firmware improvements
- **2020+**: Continued maintenance and community support

### Key Specifications

| Parameter | Value |
|-----------|-------|
| Frequency Range | 1 MHz - 6 GHz |
| ADC/DAC Resolution | 8-bit |
| Sample Rate | 20 MS/s to 22 MS/s (native) |
| Duplex Mode | Half-duplex (TX OR RX) |
| Bandwidth | Up to 20 MHz |
| TX Power | -10 dBm to +14 dBm (varies) |
| RX Sensitivity | -10 dBm to -51 dBm (varies) |
| Connector | SMA female (50Ω) |
| USB | 2.0 High-Speed (480 Mbps) |
| Power | USB bus-powered, 500 mA max |
| Dimensions | 95 mm × 45 mm × 17 mm |
| Weight | ~50 grams |
| License | Open-source hardware (CERN OHL), GPL software |

### Hardware Architecture

```
Antenna Port (SMA)
       │
       ├──► TX Path ──► Power Amplifier ──► DAC
       │
       ├──► RX Path ──► LNA ──► Mixer ──► ADC
       │
       └──► Clock Reference Input
```

**RF Front-End**: Direct conversion architecture with quadrature mixer, wideband LNA (0-40 dB programmable), VGA (0-62 dB), and power amplifier.

**Digital Backend**: Xilinx XC2C256 CPLD for clock distribution and IF processing; STM32F303 ARM Cortex-M4 microcontroller for USB communication.

### Supported Protocols

- **Wi-Fi (802.11b/g/n)**: 2.4 GHz band analysis
- **Bluetooth (Classic/BLE)**: Short-range protocol testing
- **ISM Band**: 315 MHz, 433 MHz, 868 MHz, 915 MHz
- **GPS (L1/L2)**: Navigation signal analysis (receive only)
- **Cellular**: GSM, LTE band analysis (limited by sample rate)
- **Radio**: FM/AM radio, digital radio
- **Satellite**: L-band satellite communications (receive)
- **IoT**: Zigbee, Z-Wave, LoRa, Sigfox

### Comparison with Other SDR Platforms

| Feature | HackRF One | RTL-SDR | USRP B200 | Airspy |
|---------|-----------|---------|-----------|--------|
| TX Capability | Yes | No | Yes | No |
| Frequency Range | 1 MHz - 6 GHz | 24 MHz - 1.7 GHz | 70 MHz - 6 GHz | 24 MHz - 1.8 GHz |
| Sample Rate | 20 MS/s | 2.4 MS/s | 61.44 MS/s | 10 MS/s |
| ADC Resolution | 8-bit | 8-bit | 12-bit | 12-bit |
| Half/Full Duplex | Half | RX Only | Full | RX Only |
| Price | ~$300 | ~$25 | ~$1,300 | ~$170 |
| Open Source | Yes | Partial | Partial | No |

### MITRE ATT&CK Mapping

| Tactic | Technique | ID |
|--------|-----------|-----|
| Reconnaissance | Active Scanning: Wireless Spectrum Analysis | T1590.004 |
| Reconnaissance | Gather Victim Network Information | T1590 |
| Collection | Data from Information Repositories | T1005 |

## Chapter 2: Installation & Setup

### System Requirements

- **OS**: Linux (Kali, Ubuntu, Debian, Arch, Fedora)
- **Kernel**: 3.10 or newer
- **USB**: USB 2.0 port (USB 3.0 compatible)
- **Disk**: ~50 MB for tools, ~500 MB for development
- **Memory**: 512 MB RAM minimum

### Verify USB Connection

```bash
lsusb | grep -i "hackrf\|1d50:6089"
# Expected: Bus XXX Device YYY: ID 1d50:6089 OpenMoko, Inc. HackRF One
```

### Method 1: Package Manager (Recommended)

```bash
# Kali/Debian/Ubuntu
sudo apt update && sudo apt install -y hackrf
hackrf_info

# Fedora/RHEL
sudo dnf install -y hackrf

# Arch Linux
sudo pacman -S hackrf
```

### Method 2: Build from Source

```bash
# Install dependencies
sudo apt install -y build-essential cmake git libusb-1.0-0-dev pkg-config

# Clone and build
git clone https://github.com/greatscottgadgets/hackrf.git
cd hackrf/host && mkdir build && cd build
cmake .. -DCMAKE_INSTALL_PREFIX=/usr
make -j$(nproc)
sudo make install
sudo ldconfig
hackrf_info
```

### Method 3: Docker

```bash
docker pull docker.io/hackrf/hackrf
docker run -it --privileged --device=/dev/bus/usb hackrf/hackrf hackrf_info
```

### Python Bindings

```bash
pip3 install pyhackrf

# SoapySDR integration
sudo apt install -y soapysdr-tools soapysdr-module-hackrf

# GNU Radio integration
sudo apt install -y gnuradio gr-osmosdr
```

### udev Rules

```bash
sudo tee /etc/udev/rules.d/53-hackrf.rules << 'EOF'
SUBSYSTEM=="usb", ATTR{idVendor}=="1d50", ATTR{idProduct}=="6089", MODE="0666", GROUP="plugdev"
EOF
sudo udevadm control --reload-rules && sudo udevadm trigger
sudo usermod -a -G plugdev $USER
```

### Firmware Update

```bash
# Check current firmware
hackrf_info

# Download latest firmware
wget https://github.com/greatscottgadgets/hackrf/releases/latest/download/firmware-bin/hackrf_one_usb.bin

# Enter DFU mode: Unplug, hold DFU button, plug in, release after 2s
# Flash firmware
sudo hackrf_spiflash -w hackrf_one_usb.bin

# Verify
hackrf_info
```

## Chapter 3: Basic Usage

### Device Information

```bash
# Basic device info
hackrf_info

# List connected devices
hackrf_info -d

# Verbose output
hackrf_info -v
```

### Receiving Signals

```bash
# Basic receive - capture 10 seconds at 2.4 GHz
hackrf_transfer -r capture.raw -s 20000000 -f 2400000000 -n 200000000

# Receive with gain settings
hackrf_transfer -r capture.raw \
    -s 20000000 \
    -f 433000000 \
    -l 32 \      # LNA gain (0-40 dB, 8 dB steps)
    -g 20 \      # VGA gain (0-62 dB, 2 dB steps)
    -a 1 \       # RF amplifier on (+14 dB)
    -n 100000000

# Receive until interrupted
hackrf_transfer -r capture.raw -s 20000000 -f 100000000
```

### Transmitting Signals

```bash
# Basic transmit
hackrf_transfer -t capture.raw -s 20000000 -f 433000000 -x 30

# WARNING: Use low gain initially!
# -x: TX VGA gain (0-62 dB)
# -a: RF amplifier (0=off, 1=on)

# Repeat transmission
while true; do
    hackrf_transfer -t signal.raw -s 20000000 -f 433000000 -x 30 -n 10000000
    sleep 0.1
done
```

### Data Format

HackRF uses 8-bit signed I/Q samples:
```
Format: I1 + Q1 + I2 + Q2 + I3 + Q3 + ...
Size: 2 bytes per sample (1 byte I + 1 byte Q)
Range: -128 to +127
```

**Format conversion:**

```python
import numpy as np

# Read 8-bit I/Q
data = np.fromfile('capture.raw', dtype=np.int8)

# Convert to complex float
i_data = data[0::2].astype(np.float32) / 128.0
q_data = data[1::2].astype(np.float32) / 128.0
complex_data = i_data + 1j * q_data
complex_data.tofile('capture_cf32.raw')
```

### Spectrum Sweep

```bash
# Sweep 1 MHz to 6 GHz
hackrf_sweep > sweep_results.csv

# Sweep specific range with parameters
hackrf_sweep -f 100:6000000000 -w 1000000 -l 32 -g 20 -a 1 -r sweep.csv

# Real-time display
hackrf_sweep -f 100:6000000000 -w 1000000 | \
    awk -F, '{printf "\r%d MHz: %.1f dBm", $1/1e6, $4}'
```

### hackrf_transfer Parameters Reference

| Parameter | Description | Range |
|-----------|-------------|-------|
| `-r file` | Output file for RX | - |
| `-t file` | Input file for TX | - |
| `-s rate` | Sample rate | 2 MS/s to 22 MS/s |
| `-f freq` | Center frequency | 1 MHz to 6 GHz |
| `-l gain` | LNA gain | 0-40 dB (8 dB steps) |
| `-g gain` | VGA gain | 0-62 dB (2 dB steps) |
| `-x gain` | TX VGA gain | 0-62 dB (2 dB steps) |
| `-a enable` | RF amplifier | 0=off, 1=on |
| `-n samples` | Number of samples | 0 = unlimited |

### Gain Settings Guide

```bash
# Total RX Gain = LNA + VGA + (Amp ? 14 : 0)
# Strong signal: Low gain (-l 0 -g 0 -a 0)
# Weak signal: High gain (-l 32 -g 40 -a 1)
# General purpose: (-l 32 -g 20 -a 1)

# TX: Start low! (-x 20), max (-x 62 -a 1) with dummy load only
```

## Chapter 4: Intermediate Usage

### FM Radio Capture

```bash
# Capture FM broadcast
hackrf_transfer -r fm_capture.bin -f 100000000 -s 2000000 -a 1 -g 40

# Demodulate with GNU Radio
python3 fm_demod.py fm_capture.bin fm_audio.wav
```

### ADS-B Capture

```bash
# Capture 1090 MHz for ADS-B
hackrf_transfer -r adsb_capture.bin -f 1090000000 -s 2000000 -a 1 -g 40
dump1090 --interactive --ifile adsb_capture.bin
```

### Band Scanner

```bash
#!/bin/bash
# ISM band scanner

declare -A BANDS
BANDS["315MHz"]="315000000:316000000"
BANDS["433MHz"]="433000000:434000000"
BANDS["868MHz"]="868000000:869000000"
BANDS["915MHz"]="915000000:916000000"
BANDS["2.4GHz"]="2400000000:2500000000"

for band in "${!BANDS[@]}"; do
    IFS=':' read -r start stop <<< "${BANDS[$band]}"
    echo "Sweeping $band..."
    hackrf_sweep -f "$start:$stop" -w 1000000 -l 32 -g 20 -a 1 > "${band}.csv"
    max_power=$(awk -F, '{print $2}' "${band}.csv" | sort -n | tail -1)
    echo "  Max power: ${max_power} dBm"
done
```

### Signal Generation

```python
import numpy as np

# Generate sine wave
samp_rate = 2000000
freq = 1000000
duration = 1.0
samples = int(samp_rate * duration)
t = np.linspace(0, duration, samples, False)
signal = np.exp(1j * 2 * np.pi * freq * t) * 0.5
signal = (signal * 127).astype(np.int8)
signal.tofile('sine.bin')
```

```bash
hackrf_transfer -t sine.bin -f 100000000 -s 2000000 -a 1 -x 40
```

### Clock Control

```bash
# Set external clock source
hackrf_clock -o 1  # External
hackrf_clock -o 0  # Internal
hackrf_clock -r    # Check status
```

## Chapter 5: Advanced Usage

### IQ Capture Analysis

```python
import numpy as np
import matplotlib.pyplot as plt

def analyze_iq(filename, samp_rate=2000000):
    data = np.fromfile(filename, dtype=np.int8)
    iq = data[0::2] + 1j * data[1::2]

    fft_result = np.fft.fftshift(np.fft.fft(iq))
    freqs = np.fft.fftshift(np.fft.fftfreq(len(iq), 1/samp_rate))
    magnitude = np.abs(fft_result)

    plt.figure(figsize=(12, 6))
    plt.subplot(2, 1, 1)
    plt.plot(freqs/1e6, 20*np.log10(magnitude))
    plt.xlabel('Frequency (MHz)')
    plt.ylabel('Magnitude (dB)')
    plt.title('Spectrum')

    plt.subplot(2, 1, 2)
    plt.plot(iq[:1000].real, iq[:1000].imag)
    plt.xlabel('I')
    plt.ylabel('Q')
    plt.title('IQ Constellation')

    plt.tight_layout()
    plt.savefig('iq_analysis.png')
```

### Signal Detection

```python
import numpy as np

def detect_signals(filename, threshold_db=20):
    data = np.fromfile(filename, dtype=np.int8)
    iq = data[0::2] + 1j * data[1::2]

    fft_result = np.fft.fftshift(np.fft.fft(iq))
    magnitude = np.abs(fft_result)
    power_db = 20 * np.log10(magnitude + 1e-10)

    threshold = np.max(power_db) - threshold_db
    peaks = np.where(power_db > threshold)[0]
    freqs = np.fft.fftshift(np.fft.fftfreq(len(iq), 1/2000000))

    signals = []
    for peak in peaks:
        signals.append({'frequency': freqs[peak] * 1e6, 'power': power_db[peak]})
    return signals
```

### Protocol Decoder

```python
import numpy as np

class SimpleDecoder:
    def __init__(self, bit_rate=4800, samp_rate=2000000):
        self.bit_rate = bit_rate
        self.samp_rate = samp_rate
        self.samples_per_bit = int(samp_rate / bit_rate)

    def decode_ook(self, iq_data):
        envelope = np.abs(iq_data)
        threshold = np.mean(envelope) * 2
        bits = (envelope > threshold).astype(int)
        bits_resampled = bits[::self.samples_per_bit]

        bytes_data = []
        for i in range(0, len(bits_resampled) - 8, 8):
            byte = 0
            for bit in bits_resampled[i:i+8]:
                byte = (byte << 1) | bit
            bytes_data.append(byte)
        return bytes(bytes_data)
```

### Custom Firmware

```bash
# Clone and build firmware
git clone https://github.com/greatscottgadgets/hackrf.git
cd hackrf/firmware
make
hackrf_spiflash -w hackrf_one_usb.bin
hackrf_info  # Verify
```

## Chapter 6: Real-World Applications

### Key Fob Signal Research

```bash
#!/bin/bash
# Key fob capture and analysis
# FOR AUTHORIZED SECURITY TESTING ONLY

FREQ=${1:-433920000}
DURATION=${2:-5}
OUTPUT_DIR="keyfob_$(date +%Y%m%d_%H%M%S)"
mkdir -p "$OUTPUT_DIR"

hackrf_transfer \
    -r "$OUTPUT_DIR/keyfob.raw" \
    -s 20000000 \
    -f $FREQ \
    -l 32 -g 30 -a 1 \
    -n $((20000000 * 2 * DURATION))

# Analyze for packet structure
python3 << EOF
import numpy as np

data = np.fromfile('$OUTPUT_DIR/keyfob.raw', dtype=np.int8)
i_data = data[0::2].astype(np.float32) / 128.0
q_data = data[1::2].astype(np.float32) / 128.0
power = np.sqrt(i_data**2 + q_data**2)
power_db = 20 * np.log10(power / 128 + 1e-10)

noise_floor = np.percentile(power_db, 5)
threshold = noise_floor + 20
active_samples = np.where(power_db > threshold)[0]

if len(active_samples) > 0:
    groups = np.split(active_samples, np.where(np.diff(active_samples) > 1000)[0] + 1)
    print(f"Detected {len(groups)} potential packet(s)")
    for i, group in enumerate(groups):
        duration_us = len(group) / 20000000 * 1e6
        print(f"  Packet {i+1}: {duration_us:.0f} us")
EOF
```

### Wireless Protocol Analysis Workflow

```bash
#!/bin/bash
TARGET_FREQ=${1:-433000000}
DURATION=${2:-10}
OUTPUT_DIR="analysis_$(date +%Y%m%d_%H%M%S)"
mkdir -p "$OUTPUT_DIR"

# Step 1: Wideband scan
hackrf_sweep -f $((TARGET_FREQ - 10000000)):$((TARGET_FREQ + 10000000)) \
    -w 100000 > "$OUTPUT_DIR/sweep.csv"

# Step 2: Capture signal
hackrf_transfer -r "$OUTPUT_DIR/capture.raw" -s 20000000 -f $TARGET_FREQ \
    -l 32 -g 20 -a 1 -n $((20000000 * 2 * $DURATION))

# Step 3: Analyze
python3 << EOF
import numpy as np
data = np.fromfile('$OUTPUT_DIR/capture.raw', dtype=np.int8)
i_data = data[0::2].astype(np.float32) / 128.0
q_data = data[1::2].astype(np.float32) / 128.0
power_db = 20 * np.log10(np.sqrt(i_data**2 + q_data**2) / 128 + 1e-10)
print(f"Average power: {np.mean(power_db):.1f} dBFS")
print(f"Peak power: {np.max(power_db):.1f} dBFS")
EOF
```

### IoT Device Testing

```bash
#!/bin/bash
# Common IoT frequencies
declare -A IOT_FREQS
IOT_FREQS["Zigbee"]=2480000000
IOT_FREQS["Z-Wave"]=908400000
IOT_FREQS["LoRa"]=868000000
IOT_FREQS["BLE"]=2426000000

for protocol in "${!IOT_FREQS[@]}"; do
    freq=${IOT_FREQS[$protocol]}
    echo "Testing $protocol at $((freq/1000000)) MHz..."
    hackrf_transfer -r "${protocol}.raw" -s 20000000 -f $freq -l 32 -g 25 -a 1 \
        -n $((20000000 * 2 * 10))
done
```

### Signal Replay Testing

```bash
#!/bin/bash
# FOR AUTHORIZED TESTING ONLY

FREQ=${1:-433000000}
OUTPUT_DIR="replay_$(date +%Y%m%d_%H%M%S)"
mkdir -p "$OUTPUT_DIR"

# Capture
hackrf_transfer -r "$OUTPUT_DIR/original.raw" -s 20000000 -f $FREQ -l 32 -g 30 -a 1 \
    -n $((20000000 * 2 * 5))

# Replay
hackrf_transfer -t "$OUTPUT_DIR/original.raw" -s 20000000 -f $FREQ -x 30 -a 1
```

## Chapter 7: Detection & Defense

### Detecting Unauthorized RF Transmissions

```bash
# Wideband monitoring
rtl_power -f 24M:1766M:1M -g 40 -i 10 -e 300 spectrum.csv
```

### HackRF Signature Detection

```python
import numpy as np

class HackRFSignatureDetector:
    def analyze_iq_data(self, iq_data):
        results = {'likely_hackrf': False, 'confidence': 0, 'indicators': []}

        # Check for DC offset (direct conversion artifact)
        dc_offset = np.mean(iq_data)
        if abs(dc_offset) > 0.01:
            results['indicators'].append('dc_offset')
            results['confidence'] += 0.3

        # Check for I/Q imbalance
        i_std = np.std(np.real(iq_data))
        q_std = np.std(np.imag(iq_data))
        iq_imbalance = abs(i_std - q_std) / max(i_std, q_std)
        if iq_imbalance > 0.05:
            results['indicators'].append('iq_imbalance')
            results['confidence'] += 0.2

        # 8-bit quantization artifacts (~48 dB dynamic range)
        dynamic_range = 20 * np.log10(np.max(np.abs(iq_data)) /
                                        np.std(iq_data[iq_data != 0]))
        if 40 < dynamic_range < 55:
            results['indicators'].append('quantization')
            results['confidence'] += 0.2

        results['likely_hackrf'] = results['confidence'] > 0.5
        return results
```

### RF Security Measures

**Physical Security:**
- Secure antennas against unauthorized relocation
- Use tamper-evident seals on connections
- Monitor cable integrity
- Lock SDR equipment in secure locations

**Protocol Security:**
- Always encrypt sensitive data (AES-128+)
- Implement mutual authentication
- Use rolling code schemes
- Employ frequency hopping and spread spectrum

**RF Shielding:**

```python
import numpy as np

def faraday_cage_attenuation(frequency, material, thickness_mm):
    sigma_values = {'copper': 5.8e7, 'aluminum': 3.77e7, 'steel': 1e7}
    mu_r = 100 if material == 'steel' else 1.0
    sigma = sigma_values.get(material, 1e7)
    thickness = thickness_mm / 1000

    skin_depth = np.sqrt(2 / (2 * np.pi * frequency * mu_r * 4e-7 * np.pi * sigma))
    attenuation_db = 20 * np.log10(np.exp(1)) * thickness / skin_depth
    return attenuation_db
```

### Counter-Surveillance

Scan for surveillance devices on common frequencies:
- Wireless cameras: 900 MHz, 1.2 GHz, 2.4 GHz, 5.8 GHz
- Audio transmitters: 100-500 MHz
- GPS trackers: 1.5-1.6 GHz
- Cellular bugs: 800 MHz - 2.5 GHz

## Chapter 8: Troubleshooting & Resources

### Common Errors

**"libusb_open() error" / "No HackRF boards found"**
```bash
lsusb | grep -i hackrf
sudo tee /etc/udev/rules.d/52-hackrf.rules << 'EOF'
SUBSYSTEM=="usb", ATTRS{idVendor}=="1d50", ATTRS{idProduct}=="6089", MODE="0666", GROUP="plugdev"
EOF
sudo udevadm control --reload-rules && sudo udevadm trigger
sudo usermod -aG plugdev $USER
```

**"Firmware too old"**
```bash
hackrf_info | grep "Firmware"
hackrf_spiflash -w hackrf_one_usb.bin
```

**"transfer stalled" / "USB transfer error"**
```bash
lsusb -t  # Check USB bandwidth
hackrf_transfer -r capture.raw -f 100000000 -s 10000000 -g 40  # Reduce sample rate
```

**"Device not responding"**
```bash
hackrf_operaclean
# Kill competing processes
pkill -f "hackrf_transfer\|hackrf_sweep\|gqrx\|gnuradio"
hackrf_spiflash -w hackrf_one_usb.bin  # Last resort
```

### FAQ

**Q: Can HackRF transmit and receive simultaneously?**
A: No. HackRF One is half-duplex. Use two HackRFs or USRP B200/B210 for full-duplex.

**Q: What antenna should I use?**
A: Depends on frequency: 1-30 MHz (long wire/dipole), 30-300 MHz (discone/vertical), 300 MHz-1 GHz (discone/Yagi), 1-6 GHz (horn/patch/dish).

**Q: What's the maximum range?**
A: TX power is ~-10 to +15 dBm. Line of sight: 433 MHz (1-5 km), 2.4 GHz (100m-1 km).

**Q: Is HackRF legal?**
A: Receiving is generally legal. Transmitting requires authorization. HackRF is NOT FCC/CE certified for transmission.

### Resources

- **GitHub**: https://github.com/greatscottgadgets/hackrf
- **Wiki**: https://github.com/greatscottgadgets/hackrf/wiki
- **GNU Radio**: https://www.gnuradio.org
- **Great Scott Gadgets**: https://greatscottgadgets.com
- **Michael Ossmann's Tutorials**: Great Scott Gadgets training materials
- **Reddit r/hackrf**: Community support
