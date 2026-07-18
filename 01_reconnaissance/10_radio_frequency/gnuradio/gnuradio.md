---
tool_name: "gnuradio"
display_name: "GNU Radio"
mitre_tactic: "reconnaissance"
mitre_tactic_id: "TA0043"
mitre_subcategory: "radio_frequency"
install_command: "sudo apt install gnuradio"
version: "3.10"
github_repo: "https://github.com/gnuradio/gnuradio"
kali_tools_page: "https://www.kali.org/tools/gnuradio/"
difficulty_level: "intermediate-to-advanced"
requires_root: false
gui_available: true
tags: ["sdr", "signal-processing", "flowgraph", "python", "c++", "dsp", "modulation", "demodulation"]
related_tools: ["hackrf", "gqrx", "uhd", "gr-osmosdr", "inspectrum"]
---

# GNU Radio

## Chapter 1: Introduction & Overview

### What is GNU Radio?

GNU Radio is a free and open-source software development toolkit that provides signal processing blocks to implement software-defined radios (SDR) and signal processing systems. It is the most widely used open-source SDR framework in the world, employed by researchers, engineers, government agencies, military organizations, and hobbyists for over two decades.

GNU Radio allows users to design and simulate signal processing systems using a flowgraph-based architecture. Flowgraphs consist of signal processing blocks connected together, where each block performs a specific operation on the data stream. The toolkit supports both a graphical user interface called GNU Radio Companion (GRC) and a Python/C++ API for programmatic construction of flowgraphs.

The core of GNU Radio is written in C++ for performance, with Python bindings that allow rapid prototyping and scripting. This dual-language approach provides the speed of compiled code with the flexibility of an interpreted language, making it ideal for both real-time radio applications and offline signal analysis.

### History & Background

- **Created:** 1999 by Eric Blossom as part of the GNU Project
- **License:** GNU General Public License v3 (GPLv3)
- **Language:** C++ core with Python bindings
- **Website:** https://www.gnuradio.org
- **Repository:** https://github.com/gnuradio/gnuradio

**Key Milestones:**
- 1999: Initial release by Eric Blossom
- 2004: Integration with UHD (USRP Hardware Driver) for hardware support
- 2012: Migration from Python 2 to Python 3 began
- 2016: GNU Radio 3.7 introduced a major API redesign
- 2018: GNU Radio 3.8 moved to Python 3 exclusively
- 2022: GNU Radio 3.10 added improved block discovery and new filter designs
- 2024: Continued community-driven development and hardware support

### When to Use This Tool

- **Signal processing prototyping:** Rapidly build and test DSP algorithms
- **SDR development:** Create custom radio receivers and transmitters
- **Protocol decoding:** Implement decoders for ADS-B, AIS, LoRa, P25, TETRA
- **RF security testing:** Analyze and test wireless protocols in authorized engagements
- **Spectrum monitoring:** Build wideband spectrum analyzers and monitoring systems
- **Education:** Learn DSP concepts, modulation theory, and RF engineering
- **Research:** Academic and commercial signal processing research

### When NOT to Use This Tool

- Simple receive-only tasks (use gqrx or SDR# instead)
- Tasks that don't require signal processing (use protocol-specific tools)
- Without understanding of DSP fundamentals (steep learning curve)

### Legal and Ethical Considerations

- **Always obtain authorization** before transmitting on any frequency
- **Receiving publicly broadcast signals** is generally legal
- **Transmitting requires authorization** in most jurisdictions (FCC, Ofcom, etc.)
- **Use shielded environments** for RF testing when possible
- **Document all activities** and maintain records of authorization
- **Follow responsible disclosure** when discovering wireless vulnerabilities

### Key Concepts

- **Flowgraph:** A directed acyclic graph (DAG) of connected signal processing blocks
- **Block:** A signal processing element with input and output ports
- **Stream:** Continuous data flowing between blocks through circular buffers
- **Message Passing:** Asynchronous inter-block communication via PMT objects
- **Sample Rate:** Number of samples per second; determines bandwidth (Nyquist: BW = SR/2)
- **I/Q Data:** Complex (In-phase/Quadrature) representation of RF signals
- **Decimation:** Reducing sample rate by an integer factor
- **Interpolation:** Increasing sample rate by an integer factor

---

## Chapter 2: Installation & Setup

### System Requirements

- **OS:** Linux (Kali, Ubuntu, Debian, Fedora, Arch), macOS, or Windows (limited)
- **CPU:** x86_64, multi-core recommended
- **RAM:** 2 GB minimum, 4 GB recommended
- **Disk Space:** ~500 MB for base install, ~2 GB with development packages
- **GPU:** OpenGL 2.1+ for GUI rendering

### Installation Methods

#### Method 1: APT (Kali/Ubuntu/Debian — Recommended)

```bash
sudo apt update
sudo apt install gnuradio
```

This installs GNU Radio along with Python bindings, GRC, and common blocks.

#### Method 2: Flatpak (Universal Linux)

```bash
flatpak install flathub org.gnuradio.GNURadio
flatpak run org.gnuradio.GNURadio
```

#### Method 3: PyBOMBS (Advanced Users)

```bash
# Install PyBOMBS
pip3 install pybombs

# Initialize recipes
pybombs recipes add-defaults

# Create a prefix (isolated install directory)
pybombs prefix init ~/prefix -a myprefix

# Install GNU Radio
pybombs install gnuradio

# Activate the prefix
source ~/prefix/setup_env.sh
```

#### Method 4: Build from Source

```bash
# Install dependencies
sudo apt install -y \
    build-essential cmake git \
    libboost-all-dev libcppunit-dev liblog4cpp5-dev \
    libfftw3-dev libgmp-dev libcodec2-dev \
    libsndfile1-dev libvolk2-dev \
    python3-dev python3-numpy python3-scipy python3-pygccxml \
    swig doxygen

# Clone repository
git clone https://github.com/gnuradio/gnuradio.git
cd gnuradio

# Create build directory
mkdir build && cd build

# Configure
cmake -DCMAKE_INSTALL_PREFIX=/usr \
      -DCMAKE_BUILD_TYPE=Release \
      -DENABLE_GR_WAVELET=OFF \
      ..

# Build (use -j flag for parallel compilation)
make -j$(nproc)

# Install
sudo make install
sudo ldconfig
```

### Installing Hardware Drivers

#### RTL-SDR

```bash
sudo apt install rtl-sdr librtlsdr-dev
# Blacklist DVB-T driver
sudo tee /etc/modprobe.d/blacklist-rtlsdr.conf << 'EOF'
blacklist dvb_usb_rtl28xxu
blacklist rtl2832
blacklist rtl2830
EOF
```

#### HackRF

```bash
sudo apt install hackrf libhackrf-dev
```

#### USRP (Ettus Research)

```bash
sudo apt install uhd-host libuhd-dev uhd-examples
uhd_images_downloader
```

#### gr-osmosdr (Multi-Device Support)

```bash
sudo apt install gr-osmosdr
```

### udev Rules for Non-Root Access

```bash
sudo tee /etc/udev/rules.d/20-sdr.rules << 'EOF'
# RTL-SDR
SUBSYSTEM=="usb", ATTR{idVendor}=="0bda", ATTR{idProduct}=="2838", MODE="0666"
# HackRF
SUBSYSTEM=="usb", ATTR{idVendor}=="1d50", ATTR{idProduct}=="6089", MODE="0666"
# Airspy
SUBSYSTEM=="usb", ATTR{idVendor}=="1d50", ATTR{idProduct}=="60a1", MODE="0666"
# BladeRF
SUBSYSTEM=="usb", ATTR{idVendor}=="2cf0", ATTR{idProduct}=="0200", MODE="0666"
EOF

sudo udevadm control --reload-rules
sudo udevadm trigger
```

### Verification

```bash
# Check installation
gnuradio-companion --version

# Launch GRC
gnuradio-companion

# Verify Python bindings
python3 -c "from gnuradio import gr; print(gr.version())"

# Check available blocks
python3 -c "from gnuradio import blocks, analog, filter, fft; print('Blocks loaded successfully')"
```

---

## Chapter 3: Basic Usage

### GNU Radio Companion (GRC)

GRC is the graphical flowgraph editor. Launch it with:

```bash
gnuradio-companion
```

#### GRC Interface Elements

| Element | Description |
|---------|-------------|
| Block Library | Left panel with searchable block categories |
| Canvas | Central area where flowgraphs are built |
| Properties Panel | Right panel showing selected block parameters |
| Variable Editor | Bottom panel for quick variable editing |
| Generate | F button or toolbar — generates Python from flowgraph |
| Run | F6 or toolbar — executes the generated Python script |

#### Building a Basic Flowgraph

**Example: Signal Source to File Sink**

1. Open GRC: `gnuradio-companion`
2. Add blocks:
   - **Signal Source** (Add Block → Sources → Signal Source)
   - **File Sink** (Add Block → Sinks → File Sink)
3. Connect Signal Source output to File Sink input
4. Set Signal Source parameters:
   - Sample Rate: `32000`
   - Waveform: `Cosine`
   - Frequency: `1000`
   - Amplitude: `1.0`
5. Set File Sink parameters:
   - File: `/tmp/signal.raw`
   - Type: `complex`
6. Click Generate (F), then Run (F6)

### Command-Line Flowgraph Execution

```bash
# Generate Python from GRC (if using .grc file)
grcc my_flowgraph.grc -d /tmp

# Execute the generated Python
python3 /tmp/my_flowgraph.py

# Run with custom parameters
python3 /tmp/my_flowgraph.py --samp-rate=2400000 --freq=100e6
```

### Essential Block Reference

#### Source Blocks

| Block | Purpose | Key Parameters |
|-------|---------|----------------|
| `analog.sig_source_c` | Generate complex sinusoid | samp_rate, waveform, freq, ampl |
| `analog.sig_source_f` | Generate real sinusoid | samp_rate, waveform, freq, ampl |
| `osmosdr.source` | SDR hardware receiver | args, samp_rate, center_freq, gain |
| `uhd.usrp_source` | USRP hardware receiver | args, stream_args, center_freq, gain |
| `blocks.file_source` | Read from file | item_size, file, repeat |
| `analog.noise_source_c` | Generate noise | type, ampl, seed |
| `analog.random_source_c` | Random samples | min, max, num_samps |

#### Sink Blocks

| Block | Purpose | Key Parameters |
|-------|---------|----------------|
| `blocks.null_sink` | Discard data | item_size |
| `blocks.file_sink` | Write to file | item_size, file, unbuffered |
| `blocks.head` | Limit samples | size, num_items |
| `audio.sink` | Audio output | sampling_rate, device |
| `qtgui.time_sink_c` | Time display | size, samp_rate, name |
| `qtgui.freq_sink_c` | FFT display | size, samp_rate, center_freq |

#### Processing Blocks

| Block | Purpose | Key Parameters |
|-------|---------|----------------|
| `filter.fir_filter_ccf` | FIR filter | decim, taps |
| `filter.freq_xlating_fir_filter_ccc` | Frequency-translating filter | decim, taps, center_freq |
| `fft.fft_vcc` | FFT computation | fft_size, forward, window |
| `blocks.complex_to_mag_squared` | Power detection | vlen |
| `analog.agc2_cc` | Automatic gain control | rate, reference, gain |
| `digital.clock_recovery_mm` | Symbol timing recovery | omega, gain_omega, mu, gain_mu |

### Complete Example: FM Broadcast Receiver

```python
#!/usr/bin/env python3
"""FM broadcast receiver using GNU Radio + RTL-SDR"""

from gnuradio import gr, blocks, analog, filter, audio
from gnuradio.filter import firdes
import osmosdr

class fm_receiver(gr.top_block):
    def __init__(self):
        gr.top_block.__init__(self, "FM Receiver")

        # Parameters
        samp_rate = 2.4e6
        center_freq = 98.1e6
        audio_rate = 48000

        # SDR Source
        self.source = osmosdr.source(args="rtl=0")
        self.source.set_sample_rate(samp_rate)
        self.source.set_center_freq(center_freq)
        self.source.set_gain(40)

        # Low-pass filter
        lpf_taps = firdes.low_pass(
            1.0,           # gain
            samp_rate,     # sample rate
            100e3,         # cutoff
            25e3,          # transition width
            firdes.WIN_HAMMING
        )
        self.lpf = filter.fir_filter_ccf(1, lpf_taps)

        # FM demodulator
        self.fm_demod = analog.wfm_rcv(
            quad_rate=250e3,
            audio_decimation=5
        )

        # Audio filter
        audio_taps = firdes.low_pass(
            1.0,
            audio_rate,
            15000,
            5000,
            firdes.WIN_HAMMING
        )
        self.audio_filter = filter.fir_filter_fff(1, audio_taps)

        # Audio output
        self.audio_sink = audio.sink(audio_rate, "", True)

        # Connections
        self.connect(self.source, self.lpf)
        self.connect(self.lpf, self.fm_demod)
        self.connect(self.fm_demod, self.audio_filter)
        self.connect(self.audio_filter, self.audio_sink)


if __name__ == '__main__':
    tb = fm_receiver()
    tb.start()
    input("Press Enter to stop...")
    tb.stop()
    tb.wait()
```

### Complete Example: Spectrum Analyzer

```python
#!/usr/bin/env python3
"""Wideband spectrum analyzer using GNU Radio"""

from gnuradio import gr, blocks, analog, filter, fft, qtgui
from gnuradio.filter import firdes
from PyQt5 import Qt
import sys
import osmosdr

class spectrum_analyzer(gr.top_block, Qt.QWidget):
    def __init__(self):
        gr.top_block.__init__(self, "Spectrum Analyzer")
        Qt.QWidget.__init__(self)

        # Parameters
        self.samp_rate = 2.4e6
        self.center_freq = 100e6
        self.fft_size = 1024

        # SDR Source
        self.source = osmosdr.source(args="rtl=0")
        self.source.set_sample_rate(self.samp_rate)
        self.source.set_center_freq(self.center_freq)
        self.source.set_gain(40)

        # FFT display
        self.fft_sink = qtgui.freq_sink_c(
            self.fft_size,
            firdes.WIN_BLACKMAN_HARRIS,
            self.center_freq,
            self.samp_rate,
            "Spectrum",
            1,
            None
        )

        # Connection
        self.connect(self.source, self.fft_sink)


if __name__ == '__main__':
    from PyQt5 import Qt as qqt
    qapp = qqt.QApplication(sys.argv)
    tb = spectrum_analyzer()
    tb.start()
    tb.show()
    qapp.exec_()
    tb.stop()
    tb.wait()
```

---

## Chapter 4: Intermediate Usage — Python & Bash Scripting

### Python Scripting

#### Building Flowgraphs Programmatically

```python
#!/usr/bin/env python3
"""Programmatic flowgraph construction"""

from gnuradio import gr, blocks, analog, filter, fft
from gnuradio.filter import firdes

class iq_recorder(gr.top_block):
    """Record IQ data from an SDR device"""

    def __init__(self, device="rtl=0", freq=100e6, samp_rate=2.4e6, duration=10, output="/tmp/capture.raw"):
        gr.top_block.__init__(self, "IQ Recorder")

        # Source
        self.source = osmosdr.source(args=device)
        self.source.set_sample_rate(samp_rate)
        self.source.set_center_freq(freq)
        self.source.set_gain(40)

        # Head block to limit duration
        num_samples = int(samp_rate * duration * 2)  # 2 bytes per complex sample
        self.head = blocks.head(gr.sizeof_gr_complex, num_samples)

        # File sink
        self.sink = blocks.file_sink(gr.sizeof_gr_complex, output, False)
        self.sink.set_unbuffered(True)

        # Connections
        self.connect(self.source, self.head)
        self.connect(self.head, self.sink)

# Usage
# recorder = iq_recorder(freq=433e6, duration=5)
# recorder.start()
# recorder.wait()
```

#### PMT (Polymorphic Type) Usage

```python
from gnuradio import gr

# Create PMT objects
pmt_int = gr.pmt.from_long(42)
pmt_float = gr.pmt.from_double(3.14)
pmt_string = gr.pmt.intern("hello")
pmt_complex = gr.pmt.from_complex(1+2j)
pmt_dict = gr.pmt.make_dict()
pmt_dict = gr.pmt.dict_add(pmt_dict, gr.pmt.intern("freq"), gr.pmt.from_double(100e6))

# Convert back to Python types
python_int = gr.pmt.to_long(pmt_int)
python_str = gr.pmt.symbol_to_string(pmt_string)

# Create vector types
v = gr.pmt.init_c16vector(1024, [0]*1024)
pmt_vec = gr.pmt.init_c32vector(1024, [0]*1024)
```

#### Message Passing

```python
# In a custom block, register message handlers
def __init__(self):
    # Register input message port
    self.message_register_port_in(gr.pmt.intern("freq_in"))

    # Register output message port
    self.message_register_port_out(gr.pmt.intern("freq_out"))

def handle_freq_msg(self, msg):
    """Handle incoming frequency message"""
    if gr.pmt.is_number(msg):
        freq = gr.pmt.to_double(msg)
        self.set_center_freq(freq)
        # Send acknowledgment
        self.message_port_pub(
            gr.pmt.intern("freq_out"),
            gr.pmt.intern("ack")
        )
```

#### Custom Block Implementation

```python
#!/usr/bin/env python3
"""Custom GNU Radio block example"""

from gnuradio import gr
import numpy as np

class power_detector(gr.sync_block):
    """Detect power level of input signal"""

    def __init__(self, averaging=100):
        gr.sync_block.__init__(
            self,
            name="Power Detector",
            in_sig=[np.complex64],
            out_sig=[np.float32]
        )
        self.averaging = averaging
        self.buffer = []

    def work(self, input_items, output_items):
        in0 = input_items[0]
        out = output_items[0]

        # Calculate instantaneous power
        power = np.abs(in0) ** 2

        # Simple moving average
        if len(power) > self.averaging:
            kernel = np.ones(self.averaging) / self.averaging
            out[:] = np.convolve(power, kernel, mode='same')
        else:
            out[:] = power

        return len(out)
```

### Bash Scripting

#### Automated Capture and Analysis

```bash
#!/bin/bash
# gnuradio_capture.sh — Automated GNU Radio capture and analysis

FREQ=${1:-100000000}
DURATION=${2:-10}
OUTPUT_DIR="capture_$(date +%Y%m%d_%H%M%S)"
mkdir -p "$OUTPUT_DIR"

echo "=== GNU Radio Automated Capture ==="
echo "Frequency: $((FREQ/1000000)) MHz"
echo "Duration: $DURATION seconds"

# Generate flowgraph from template
cat > "$OUTPUT_DIR/recorder.py" << EOF
#!/usr/bin/env python3
from gnuradio import gr, blocks, analog, osmosdr
import sys

class recorder(gr.top_block):
    def __init__(self):
        gr.top_block.__init__(self)
        self.source = osmosdr.source(args="rtl=0")
        self.source.set_sample_rate(2.4e6)
        self.source.set_center_freq($FREQ)
        self.source.set_gain(40)
        self.head = blocks.head(gr.sizeof_gr_complex, int(2.4e6 * $DURATION * 2))
        self.sink = blocks.file_sink(gr.sizeof_gr_complex, "$OUTPUT_DIR/capture.raw", False)
        self.connect(self.source, self.head, self.sink)

recorder().run()
EOF

# Execute
python3 "$OUTPUT_DIR/recorder.py"

# Analyze
python3 << 'PYEOF'
import numpy as np
data = np.fromfile("OUTPUT_DIR/capture.raw".replace("OUTPUT_DIR", "$(echo $OUTPUT_DIR)"), dtype=np.complex64)
print(f"Samples: {len(data)}")
print(f"Power: {np.mean(np.abs(data)**2):.2f}")
PYEOF

echo "Capture saved to: $OUTPUT_DIR/"
```

#### Multi-Band Scanning with GNU Radio

```bash
#!/bin/bash
# Multi-band scanner using GNU Radio

OUTPUT_DIR="scan_$(date +%Y%m%d_%H%M%S)"
mkdir -p "$OUTPUT_DIR"

declare -A BANDS
BANDS["FM"]="88000000:108000000"
BANDS["Airband"]="118000000:137000000"
BANDS["ISM_433"]="433000000:435000000"
BANDS["ISM_915"]="915000000:917000000"

for band in "${!BANDS[@]}"; do
    IFS=':' read -r start stop <<< "${BANDS[$band]}"
    mid_freq=$(( (start + stop) / 2 ))
    bandwidth=$(( stop - start ))

    echo "Scanning $band ($((start/1000000))-$((stop/1000000)) MHz)..."

    python3 -c "
from gnuradio import gr, blocks, analog, fft, osmosdr
import numpy as np

class Scanner(gr.top_block):
    def __init__(self):
        gr.top_block.__init__(self)
        self.source = osmosdr.source(args='rtl=0')
        self.source.set_sample_rate(2.4e6)
        self.source.set_center_freq($mid_freq)
        self.source.set_gain(40)
        self.head = blocks.head(gr.sizeof_gr_complex, int(2.4e6 * 0.5 * 2))
        self.sink = blocks.null_sink(gr.sizeof_gr_complex)
        self.connect(self.source, self.head, self.sink)

Scanner().run()
print('Done')
"

    sleep 2
done

echo "Scan complete: $OUTPUT_DIR/"
```

---

## Chapter 5: Advanced Usage

### Out-of-Tree (OOT) Module Development

OOT modules extend GNU Radio with custom blocks. They follow a standard directory structure:

```bash
# Create OOT module
gr_modtool newmod my_module

# Add a new block
cd my_module
gr_modtool newblock my_block

# Build
cd build
cmake ..
make -j$(nproc)
sudo make install
```

#### OOT Module Structure

```
gr-my_module/
├── CMakeLists.txt
├── include/
│   └── gnuradio/
│       └── my_module/
│           └── api.h
├── lib/
│   ├── CMakeLists.txt
│   ├── my_block_impl.cc
│   └── my_block_impl.h
├── python/
│   ├── CMakeLists.txt
│   └── my_module/
│       └── __init__.py
├── grc/
│   └── my_module_my_block.block.yml
└── swig/
    └── CMakeLists.txt
```

### Performance Optimization

#### VOLK (Vector-Optimized Library of Kernels)

```bash
# Check VOLK availability
python3 -c "import volk; print(volk.__version__)"

# Profile block execution
volk_profile

# Use VOLK-optimized operations in custom blocks
import volk
import numpy as np

# VOLK multiply
a = np.random.randn(1024).astype(np.float32)
b = np.random.randn(1024).astype(np.float32)
result = volk.multiply(a, b)
```

#### Scheduler Tuning

```bash
# Set threading model
export GR_THREAD_COUNT=4

# In flowgraph Python script
from gnuradio import gr
gr.top_block.__init__(self, "flowgraph", gr.sizeof_gr_complex, 4)  # 4 threads
```

### Integration with External Tools

#### GNU Radio + HackRF

```python
from gnuradio import gr, osmosdr, blocks, analog

class hackrf_sink_test(gr.top_block):
    def __init__(self):
        gr.top_block.__init__(self)
        # HackRF sink
        self.sink = osmosdr.sink(args="hackrf=0")
        self.sink.set_sample_rate(2e6)
        self.sink.set_center_freq(433e6)
        self.sink.set_gain(40)
        # Signal source
        self.source = analog.sig_source_c(2e6, analog.GR_COS_WAVE, 1000, 0.5)
        self.connect(self.source, self.sink)
```

#### GNU Radio + SoapySDR

```bash
# Install SoapySDR support for GNU Radio
sudo apt install gr-soapy

# In Python
from gnuradio import gr, soapysdr
from SoapySDR import SOAPY_SDR_RX, SOAPY_SDR_CF32

# Use SoapySDR block
self.source = soapysdr.source(args="driver=rtlsdr")
```

### Network Streaming

```python
# Stream IQ data over network
from gnuradio import gr, blocks, network

# Server (sender)
self.file_source = blocks.file_source(gr.sizeof_gr_complex, "/tmp/iq.raw")
self.udp_sink = blocks.udp_sink(gr.sizeof_gr_complex, "127.0.0.1", 7654)
self.connect(self.file_source, self.udp_sink)

# Client (receiver)
self.udp_source = blocks.udp_source(gr.sizeof_gr_complex, "127.0.0.1", 7654)
self.file_sink = blocks.file_sink(gr.sizeof_gr_complex, "/tmp/received.raw")
self.connect(self.udp_source, self.file_sink)
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: ADS-B Aircraft Tracking

```python
#!/usr/bin/env python3
"""ADS-B receiver using GNU Radio + RTL-SDR"""

from gnuradio import gr, blocks, analog, filter, digital
from gnuradio.filter import firdes
import osmosdr
import struct

class adsb_receiver(gr.top_block):
    def __init__(self):
        gr.top_block.__init__(self, "ADS-B Receiver")

        # Parameters
        self.samp_rate = 2.4e6
        self.freq = 1090e6

        # SDR Source
        self.source = osmosdr.source(args="rtl=0")
        self.source.set_sample_rate(self.samp_rate)
        self.source.set_center_freq(self.freq)
        self.source.set_gain(48)

        # PPM correction
        self.source.set_freq_correction(0)

        # Low-pass filter for Mode S
        lpf_taps = firdes.low_pass(
            1.0, self.samp_rate, 1e6, 500e3, firdes.WIN_HAMMING
        )
        self.lpf = filter.fir_filter_ccf(1, lpf_taps)

        # Power detection
        self.power_detect = blocks.complex_to_mag_squared(1)

        # Output to file for analysis
        self.sink = blocks.file_sink(gr.sizeof_float, "/tmp/adsb_power.raw")
        self.sink.set_unbuffered(True)

        # Connections
        self.connect(self.source, self.lpf)
        self.connect(self.lpf, self.power_detect)
        self.connect(self.power_detect, self.sink)


if __name__ == '__main__':
    tb = adsb_receiver()
    tb.start()
    print("Receiving ADS-B on 1090 MHz...")
    input("Press Enter to stop...")
    tb.stop()
    tb.wait()
    print("Capture saved to /tmp/adsb_power.raw")
```

### Scenario 2: AIS Ship Tracking

```python
#!/usr/bin/env python3
"""AIS receiver using GNU Radio"""

from gnuradio import gr, blocks, analog, filter
from gnuradio.filter import firdes
import osmosdr

class ais_receiver(gr.top_block):
    def __init__(self):
        gr.top_block.__init__(self, "AIS Receiver")

        self.samp_rate = 2.4e6
        self.freq = 161.975e6  # AIS Channel 1

        self.source = osmosdr.source(args="rtl=0")
        self.source.set_sample_rate(self.samp_rate)
        self.source.set_center_freq(self.freq)
        self.source.set_gain(40)

        # FM demodulator for GMSK
        self.demod = analog.nbfm_rx(
            audio_rate=48000,
            quad_rate=192000,
            max_dev=4800,
            tau=75e-6
        )

        # Audio output
        from gnuradio import audio
        self.audio = audio.sink(48000, "", True)

        self.connect(self.source, self.demod, self.audio)


if __name__ == '__main__':
    tb = ais_receiver()
    tb.start()
    print("Receiving AIS on 161.975 MHz...")
    input("Press Enter to stop...")
    tb.stop()
    tb.wait()
```

### Scenario 3: LoRa Packet Decoder

```python
#!/usr/bin/env python3
"""LoRa signal detector using GNU Radio (simplified)"""

from gnuradio import gr, blocks, analog, filter, fft
from gnuradio.filter import firdes
import numpy as np
import osmosdr

class lora_detector(gr.top_block):
    def __init__(self, freq=868.1e6):
        gr.top_block.__init__(self, "LoRa Detector")

        self.samp_rate = 2.4e6
        self.freq = freq

        # SDR Source
        self.source = osmosdr.source(args="rtl=0")
        self.source.set_sample_rate(self.samp_rate)
        self.source.set_center_freq(self.freq)
        self.source.set_gain(40)

        # Decimation filter
        decimation = 20  # 120 kHz effective bandwidth
        lpf_taps = firdes.low_pass(
            1.0, self.samp_rate, 60e3, 20e3, firdes.WIN_HAMMING
        )
        self.decimator = filter.fir_filter_ccf(decimation, lpf_taps)

        # Power detection
        self.power = blocks.complex_to_mag_squared(1)

        # Head block
        self.head = blocks.head(gr.sizeof_float, 100000)

        # File output
        self.sink = blocks.file_sink(gr.sizeof_float, "/tmp/lora_power.raw")

        self.connect(self.source, self.decimator)
        self.connect(self.decimator, self.power)
        self.connect(self.power, self.head)
        self.connect(self.head, self.sink)


if __name__ == '__main__':
    tb = lora_detector(freq=868.1e6)
    tb.start()
    tb.wait()
    print("LoRa power data saved to /tmp/lora_power.raw")
```

---

## Chapter 7: Detection & Defense

### Detecting Unauthorized Transmissions

```python
#!/usr/bin/env python3
"""RF anomaly detector using GNU Radio"""

from gnuradio import gr, blocks, analog, filter, fft
from gnuradio.filter import firdes
import numpy as np
import osmosdr
import time

class rf_anomaly_detector(gr.top_block):
    def __init__(self, center_freq=433e6, threshold_db=20):
        gr.top_block.__init__(self, "RF Anomaly Detector")

        self.samp_rate = 2.4e6
        self.threshold_db = threshold_db

        # SDR Source
        self.source = osmosdr.source(args="rtl=0")
        self.source.set_sample_rate(self.samp_rate)
        self.source.set_center_freq(center_freq)
        self.source.set_gain(40)

        # FFT size
        self.fft_size = 1024

        # Vector to stream conversion
        self.head = blocks.head(gr.sizeof_gr_complex, self.fft_size * 100)

        # Null sink (we analyze in Python)
        self.sink = blocks.null_sink(gr.sizeof_gr_complex)

        self.connect(self.source, self.head, self.sink)

    def analyze(self, threshold_db=20):
        """Analyze captured data for anomalies"""
        # This would normally process the captured data
        # For demonstration, we show the approach
        print(f"Monitoring for signals above {threshold_db} dB threshold")
        print("In production, this would use stream tags and message passing")


class spectrum_monitor:
    """Monitor spectrum for anomalies using RTL-SDR"""

    def __init__(self, center_freq=433e6, samp_rate=2.4e6):
        self.center_freq = center_freq
        self.samp_rate = samp_rate

    def scan(self, duration=5):
        """Scan and analyze spectrum"""
        import subprocess

        # Use rtl_power for quick sweep
        cmd = [
            "rtl_power",
            "-f", f"{(self.center_freq-1e6)/1e6}:{(self.center_freq+1e6)/1e6}:100k",
            "-g", "40",
            "-i", str(duration),
            "-e", str(duration),
            "/tmp/spectrum.csv"
        ]

        subprocess.run(cmd, capture_output=True)

        # Analyze results
        import csv
        frequencies = []
        powers = []

        with open("/tmp/spectrum.csv") as f:
            reader = csv.reader(f)
            for row in reader:
                try:
                    freq = float(row[0])
                    power = float(row[3])
                    frequencies.append(freq)
                    powers.append(power)
                except (ValueError, IndexError):
                    continue

        if not powers:
            return []

        powers = np.array(powers)
        mean_power = np.mean(powers)
        std_power = np.std(powers)
        threshold = mean_power + 3 * std_power

        anomalies = [(f, p) for f, p in zip(frequencies, powers) if p > threshold]
        return anomalies


# Usage
if __name__ == '__main__':
    monitor = spectrum_monitor(center_freq=433e6)
    anomalies = monitor.scan(duration=10)

    if anomalies:
        print(f"\nDetected {len(anomalies)} anomalous signals:")
        for freq, power in anomalies:
            print(f"  {freq/1e6:.3f} MHz: {power:.1f} dB")
    else:
        print("No anomalies detected")
```

### RF Shielding Verification

```python
#!/usr/bin/env python3
"""Verify RF shielding effectiveness"""

from gnuradio import gr, blocks, analog, osmosdr
import numpy as np

def measure_shielded_attenuation(freq=433e6, samp_rate=2.4e6):
    """Measure signal attenuation through shielding"""

    # Measure outside shield
    print("Measuring outside shield...")
    outside_power = measure_power(freq, samp_rate)

    # Measure inside shield (with shielding in place)
    print("Measuring inside shield...")
    inside_power = measure_power(freq, samp_rate)

    # Calculate attenuation
    attenuation_db = outside_power - inside_power
    print(f"Shielding attenuation: {attenuation_db:.1f} dB")
    return attenuation_db


def measure_power(freq, samp_rate, duration=1):
    """Measure average power at a frequency"""
    import subprocess
    cmd = [
        "rtl_power",
        "-f", f"{freq/1e6}:{freq/1e6}:100k",
        "-g", "40",
        "-i", str(duration),
        "-e", "1",
        "/tmp/power测量.csv"
    ]
    subprocess.run(cmd, capture_output=True)

    # Read result
    import csv
    with open("/tmp/power测量.csv") as f:
        reader = csv.reader(f)
        for row in reader:
            try:
                return float(row[3])
            except (ValueError, IndexError):
                continue
    return -100
```

---

## Chapter 8: Troubleshooting

### Common Errors

#### "ImportError: No module named gnuradio"

```bash
# Verify installation
python3 -c "import gnuradio; print(gnuradio.__path__)"

# If not found, check Python path
export PYTHONPATH=/usr/lib/python3/dist-packages:$PYTHONPATH

# Reinstall
sudo apt install --reinstall gnuradio python3-gi python3-gi-cairo gir1.2-gtk-3.0
```

#### "Cannot connect to device"

```bash
# Check device connection
lsusb | grep -i "rtl\|hackrf\|airspy\|usrp"

# Check udev rules
sudo udevadm control --reload-rules
sudo udevadm trigger

# Test with device-specific tool
rtl_test -t        # RTL-SDR
hackrf_info         # HackRF
uhd_find_devices    # USRP
```

#### "Flowgraph won't generate"

```bash
# Check for syntax errors in GRC
# Verify all blocks have required parameters
# Check block connections (port types must match)

# Generate with verbose output
grcc my_flowgraph.grc -d /tmp -v
```

#### "Low CPU performance / dropped samples"

```bash
# Reduce sample rate
# Use decimation blocks early in flowgraph
# Reduce FFT size
# Close other applications
# Use fewer threads if oversubscribed

# Profile with
python3 -c "
import time
from gnuradio import gr
start = time.time()
# Your flowgraph
elapsed = time.time() - start
print(f'Execution time: {elapsed:.2f}s')
"
```

#### "Audio not playing"

```bash
# Check PulseAudio
pactl list sinks short
pactl set-default-sink @DEFAULT_SINK@

# Check ALSA
aplay -l
arecord -l

# Test audio
speaker-test -t wav -c 2
```

### Frequently Asked Questions

#### Q: What's the difference between GRC and Python scripting?

**A:** GRC is a visual editor that generates Python. For simple flowgraphs, GRC is faster. For complex logic, loops, or conditional processing, Python scripting provides more control. GRC-generated Python can be edited directly.

#### Q: Can GNU Radio transmit?

**A:** Yes, GNU Radio supports both receive and transmit. Use `osmosdr.sink()` or `uhd.usrp_sink()` for hardware output. Always ensure you have proper authorization before transmitting.

#### Q: How do I add a new block type?

**A:** Use `gr_modtool newblock` to create an OOT module, then implement the block in C++ or Python. See Chapter 5 for details on OOT development.

#### Q: What SDR hardware does GNU Radio support?

**A:** Through gr-osmosdr: RTL-SDR, HackRF, Airspy, BladeRF, LimeSDR, PlutoSDR. Through UHD: All Ettus Research USRP devices. Through SoapySDR: Virtually any SDR device.

#### Q: How do I debug a flowgraph?

**A:** Use `blocks.probe_signal_*` blocks to tap into streams, `blocks.file_sink` to capture intermediate data, and `qtgui_*` blocks for visual debugging. Enable debug logging with `GR_LOG_LEVEL=debug`.

---

## Resources

### Official Documentation

- GNU Radio Wiki: https://wiki.gnuradio.org
- GNU Radio Tutorials: https://wiki.gnuradio.org/index.php/Tutorials
- Block Reference: https://wiki.gnuradio.org/index.php/Block_Components
- GRC Guide: https://wiki.gnuradio.org/index.php/Guided_Tutorial_GRC

### Books & Papers

- "GNU Radio: A History" by Eric Blossom
- "Software Radio: A Modern Approach to Radio Engineering" by Jeffrey Reed
- "Understanding Software Defined Radio" (GNU Radio documentation)

### Community

- GNU Radio Discuss: https://discuss.gnuradio.org
- GNU Radio Chat: https://chat.gnuradio.org
- Mailing Lists: https://lists.gnu.org/mailman/listinfo/discuss-gnuradio
- GNU Radio Conference (GRCon): Annual event

### Related Tools

- **gr-osmosdr:** Hardware abstraction layer for multiple SDR devices
- **gr-satellites:** Satellite communication processing blocks
- **gr-inspector:** Automatic signal detection and classification
- **gr-dvbs2:** DVB-S2 digital television processing
- **gr-iqtlms:** IQ tools and TLM (Telemetry) processing
