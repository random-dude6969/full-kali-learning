---
tool_name: "chirp"
display_name: "CHIRP"
mitre_tactic: "reconnaissance"
mitre_tactic_id: "TA0043"
mitre_subcategory: "radio_frequency"
install_command: "sudo apt install chirp"
version: "20240118"
github_repo: "https://github.com/kk7ds/chirp"
kali_tools_page: "https://www.kali.org/tools/chirp/"
difficulty_level: "beginner"
requires_root: false
gui_available: true
tags: ["radio", "programming", "amateur-radio", "ham", "reconnaissance", "chirp"]
related_tools: ["gqrx", "hackrf", "rtl-sdr", "uv3r-programmer"]
---

# CHIRP — Radio Programming Software

## Chapter 1: Introduction & Overview

### What is CHIRP?

CHIRP is a free, open-source tool for programming amateur radio transceivers. It provides both a graphical user interface (GUI) and a command-line interface (CLI) for reading, writing, and managing radio configurations across a wide range of manufacturers and models. CHIRP normalizes the programming experience using a common image format, replacing vendor-specific tools like Kenwood ARC, Baofeng UV-5R Programmer, and WouXun KG-UV6P.

### Purpose

- **Vendor fragmentation**: Replaces dozens of proprietary tools with one application
- **Backup and recovery**: Read radio configurations into `.img` files for backup
- **Batch programming**: Clone master configurations to fleet radios
- **Data portability**: CSV export/import for spreadsheet editing
- **Cross-radio migration**: Transfer channels between different radio models

### Key Features

**Core Capabilities:**
1. Memory channel programming (name, frequency, offset, tone, power)
2. Band plan configuration
3. CTCSS/DCS tone management
4. Power level control per channel
5. Image-based cloning (full radio state in `.img` file)
6. CSV import/export for spreadsheet editing
7. Cross-radio conversion between compatible models
8. Duplex and offset programming
9. Memory names (up to 16 characters, model-dependent)
10. Scan skip lists

**Interface Modes:**
- **GUI (wxPython)**: Full graphical interface with tabbed channel editor
- **CLI**: Headless operation for scripting and automation
- **CSV editor**: Edit channel lists without a radio connected

### Supported Radio Manufacturers

| Manufacturer | Example Models |
|-------------|----------------|
| Baofeng | UV-5R, UV-82, UV-9G, GT-5, BF-F8HP |
| Kenwood | TH-D7A/G, TH-D72A, TH-D74A, TM-D700 |
| Yaesu | FT-60, FT-65, FT-817, FT-857, FT-891, FT-991 |
| Icom | IC-2100, IC-2200, IC-2730, IC-7000, IC-7100 |
| WouXun | KG-UV6P, KG-UV7D, KG-UV8D, KG-UV9D |
| TYT | TH-9000D, TH-UV3R, TH-UVF1 |
| Alinco | DX-77, DX-SR8T |
| Puxing | PX-777, PX-888 |
| Luiton | LT-5R, LT-725UV |
| QYT | KT-8900, KT-9800Plus |

### Architecture

- **pyserial**: Serial/USB communication with radios
- **wxPython**: Cross-platform GUI toolkit
- **libusb**: Direct USB access for serial-emulating radios
- **Driver modules**: Per-radio classes under `chirp/drivers/` with vendor-specific memory maps

### File Formats

| Format | Extension | Description |
|--------|-----------|-------------|
| CHIRP image | `.img` | Binary radio memory image (radio-specific) |
| CSV | `.csv` | Tabular channel data (universal interchange) |
| VX7/VX5 | `.vx7` | Legacy Yaesu format (import only) |

### Communication Protocol

CHIRP communicates over serial (typically USB-to-serial adapters) using manufacturer-specific protocols:
- **Kenwood**: Command-response ASCII at 9600 baud
- **Baofeng/UV-5R**: Custom binary protocol with memory address read/write
- **Icom CI-V**: Standardized CI-V bus at 38400 baud

**Cable types:** K-type (2-pin, used by Baofeng/WouXun/Kenwood), Yaesu SCU-35, Icom OPC-478

### MITRE ATT&CK Mapping

| Tactic | Technique | ID |
|--------|-----------|-----|
| Reconnaissance | Gather Victim Network Information | T1590 |
| Collection | Data from Information Repositories | T1005 |

## Chapter 2: Installation & Setup

### System Requirements

- **OS**: Linux, macOS, or Windows
- **Python**: 3.6+ (for CLI)
- **Serial port**: USB-to-serial adapter for radio communication
- **Hardware**: Programming cable for target radio

### Method 1: Package Manager

```bash
# Kali Linux / Debian / Ubuntu
sudo apt update
sudo apt install chirp

# Verify
chirp --version
```

### Method 2: pip Installation

```bash
pip3 install chirp
```

### Method 3: Download from Website

```bash
# Download latest daily build
wget https://trac.chirpmyradio.com/chirp_daily/daily/LATEST/chirp-latest.tar.bz2
tar xjf chirp-latest.tar.bz2
cd chirp-latest
python3 chirpc
```

### Method 4: Build from Source

```bash
# Install dependencies
sudo apt install -y python3-wxgtk4.0 python3-serial python3-libusb1 git

# Clone repository
git clone https://github.com/kk7ds/chirp.git
cd chirp

# Install
python3 setup.py install
```

### Programming Cable Setup

```bash
# Check serial port detection
ls /dev/ttyUSB*
# Typical: /dev/ttyUSB0

# Add user to dialout group for serial access
sudo usermod -a -G dialout $USER

# Check permissions
ls -la /dev/ttyUSB0

# Test with screen/minicom
screen /dev/ttyUSB0 9600
```

### Verify Installation

```bash
# List supported radios
chirp --list

# Launch GUI
chirp

# CLI test (with radio connected)
chirp --clone-read -p /dev/ttyUSB0 -r baofeng_uv5r -o test.img
```

### Uninstallation

```bash
# Package manager
sudo apt remove chirp

# pip
pip3 uninstall chirp
```

## Chapter 3: Basic Usage

### GUI Operations

```bash
# Launch CHIRP GUI
chirp

# Or with specific config
chirp --config ~/.chirp/config.ini
```

**GUI Workflow:**
1. Connect programming cable to radio
2. Launch CHIRP
3. Radio → Download From Radio
4. Select port, vendor, model
5. Click OK to read radio
6. Edit channels in the tabbed interface
7. Radio → Upload To Radio to write changes

### CLI Operations

```bash
# Read radio to image file
chirp --clone-read -p /dev/ttyUSB0 -r baofeng_uv5r -o backup.img

# Write image file to radio
chirp --clone-write -p /dev/ttyUSB0 -r baofeng_uv5r backup.img

# Export image to CSV
chirp --export -i backup.img -o channels.csv

# Import CSV back to image
chirp --import -i channels.csv -o updated.img

# List supported radios
chirp --list
```

### Working with Images

```bash
# Read radio image
chirp --clone-read -p /dev/ttyUSB0 -r baofeng_uv5r -o radio.img

# Edit image with GUI
chirp -i radio.img

# View image contents
chirp --info -i radio.img

# Compare two images
diff <(chirp --info -i image1.img) <(chirp --info -i image2.img)
```

### Working with CSV

```bash
# Export to CSV
chirp --export -i backup.img -o channels.csv

# CSV format:
# Name,Frequency,Duplex,Offset,Tone,Tone Frequency,DCS Code,DCS Polarity,DTCS Code,DTCS Polarity,Power,Comment,Cross-Tone,Cross-DCS
# "Local Repeater",446.000000,+,0.600,TSQL,67.0,023,N,023,N,Low,,,
```

### Basic Channel Programming

```python
#!/usr/bin/env python3
"""Program channels programmatically"""

import chirp

# Load image
radio = chirp.load_image('radio.img')

# Access memory channels
for i in range(1, 128):
    mem = radio.get_memory(i)
    if mem:
        print(f"Channel {i}: {mem.name} - {mem.freq} MHz")

# Modify a channel
mem = radio.get_memory(1)
mem.name = "EMERGENCY"
mem.freq = 146.520000  # MHz
mem.offset = 0.600
mem.duplex = "+"
mem.tmode = "TSQL"
mem.rtone = 100.0
mem.power = "High"
radio.set_memory(mem)

# Save modified image
radio.save('modified.img')
```

## Chapter 4: Intermediate Usage

### Batch Programming

```bash
#!/bin/bash
# Batch program multiple radios

IMAGE="master_config.img"
PORT="/dev/ttyUSB0"
RADIO="baofeng_uv5r"

for radio in radio1 radio2 radio3; do
    echo "Programming $radio..."
    chirp --clone-write -p "$PORT" -r "$RADIO" "$IMAGE"
    if [ $? -eq 0 ]; then
        echo "  Success: $radio"
    else
        echo "  Failed: $radio"
    fi
    sleep 5
    read -p "Swap radio and press Enter..."
done
```

### CSV Workflow

```python
#!/usr/bin/env python3
"""CSV channel management"""

import csv

def create_channels_csv(filename):
    """Create a channels CSV file"""
    headers = [
        'Name', 'Frequency', 'Duplex', 'Offset', 'Tone',
        'Tone Frequency', 'DCS Code', 'DCS Polarity', 'DTCS Code',
        'DTCS Polarity', 'Power', 'Comment', 'Cross-Tone', 'Cross-DCS'
    ]

    channels = [
        ['Local Repeater', '446.000000', '+', '0.600', 'TSQL', '67.0',
         '023', 'N', '023', 'N', 'Low', 'Local 70cm', '', ''],
        ['Emergency', '146.520000', '', '0.000', 'None', '88.5',
         '023', 'N', '023', 'N', 'High', '2m calling', '', ''],
        ['DMR Repeater', '442.500000', '+', '5.000', 'TSQL', '100.0',
         '023', 'N', '023', 'N', 'Medium', 'DMR TS1', '', ''],
    ]

    with open(filename, 'w', newline='') as f:
        writer = csv.writer(f)
        writer.writerow(headers)
        writer.writerows(channels)

def merge_csvs(base_csv, overlay_csv, output_csv):
    """Merge two CSV files, overlay overriding base"""
    base = {}
    with open(base_csv, 'r') as f:
        reader = csv.DictReader(f)
        for row in reader:
            key = (row['Name'], row['Frequency'])
            base[key] = row

    with open(overlay_csv, 'r') as f:
        reader = csv.DictReader(f)
        for row in reader:
            key = (row['Name'], row['Frequency'])
            base[key] = row

    headers = list(next(iter(base.values())).keys())
    with open(output_csv, 'w', newline='') as f:
        writer = csv.DictWriter(f, fieldnames=headers)
        writer.writeheader()
        writer.writerows(base.values())

# Usage
create_channels_csv('my_channels.csv')
```

### Cross-Radio Conversion

```bash
# Read from Baofeng UV-5R
chirp --clone-read -p /dev/ttyUSB0 -r baofeng_uv5r -o source.img

# Convert to CSV for editing
chirp --export -i source.img -o channels.csv

# Edit CSV in spreadsheet
# ...

# Import back
chirp --import -i channels.csv -o target.img

# Write to Kenwood TH-D74
chirp --clone-write -p /dev/ttyUSB0 -r kenwood_thd74 target.img
```

### Automated Backup Script

```bash
#!/bin/bash
# Backup all radio configurations

BACKUP_DIR="/backup/radios/$(date +%Y%m%d)"
mkdir -p "$BACKUP_DIR"

# Define radios
declare -A RADIOS
RADIOS["["UV-5R"]="/dev/ttyUSB0:baofeng_uv5r"
RADIOS["TH-D74"]="/dev/ttyUSB1:kenwood_thd74"

for name in "${!RADIOS[@]}"; do
    IFS=':' read -r port model <<< "${RADIOS[$name]}"
    echo "Backing up $name ($model)..."
    chirp --clone-read -p "$port" -r "$model" \
        "$BACKUP_DIR/${name//[^a-zA-Z0-9]/_}.img"
done

echo "Backup complete: $BACKUP_DIR"
```

## Chapter 5: Advanced Usage

### Python API for Radio Programming

```python
#!/usr/bin/env python3
"""Advanced radio programming with CHIRP API"""

import chirp
import chirp.directory
from chirp.memmap import MemoryMap
import os

class RadioProgrammer:
    def __init__(self, image_path):
        self.image_path = image_path
        self.radio = chirp.load_image(image_path)

    def list_channels(self):
        """List all programmed channels"""
        channels = []
        for i in range(1, 200):
            try:
                mem = self.radio.get_memory(i)
                if mem and mem.freq:
                    channels.append({
                        'number': i,
                        'name': mem.name,
                        'frequency': mem.freq,
                        'offset': mem.offset,
                        'duplex': mem.duplex,
                        'tmode': mem.tmode,
                        'rtone': mem.rtone,
                        'power': mem.power
                    })
            except:
                continue
        return channels

    def program_channel(self, number, name, freq, offset=0, duplex='',
                       tone_mode='None', tone=88.5, power='Low'):
        """Program a single channel"""
        mem = self.radio.get_memory(number)
        mem.name = name[:16]  # Max 16 chars
        mem.freq = freq
        mem.offset = offset
        mem.duplex = duplex
        mem.tmode = tone_mode
        mem.rtone = tone
        mem.power = power
        self.radio.set_memory(mem)

    def bulk_program(self, channels):
        """Program multiple channels at once"""
        for ch in channels:
            self.program_channel(**ch)
        self.radio.save(self.image_path)

    def export_csv(self, csv_path):
        """Export channels to CSV"""
        import csv
        channels = self.list_channels()

        with open(csv_path, 'w', newline='') as f:
            writer = csv.DictWriter(f, fieldnames=channels[0].keys())
            writer.writeheader()
            writer.writerows(channels)

    def backup(self, backup_path):
        """Create backup of current image"""
        import shutil
        shutil.copy2(self.image_path, backup_path)

# Usage
programmer = RadioProgrammer('uv5r.img')
channels = programmer.list_channels()
for ch in channels:
    print(f"CH {ch['number']}: {ch['name']} - {ch['frequency']} MHz")

# Program new channel
programmer.program_channel(
    number=100,
    name='EMERGENCY',
    freq=146.520000,
    tone_mode='TSQL',
    tone=100.0,
    power='High'
)
```

### Fleet Management Script

```python
#!/usr/bin/env python3
"""Fleet radio management"""

import os
import shutil
from datetime import datetime

class FleetManager:
    def __init__(self, master_image, radio_configs):
        self.master_image = master_image
        self.radio_configs = radio_configs

    def generate_configs(self, output_dir):
        """Generate individual radio configs from master"""
        os.makedirs(output_dir, exist_ok=True)

        for radio_name, overrides in self.radio_configs.items():
            # Load master
            radio = chirp.load_image(self.master_image)

            # Apply overrides
            for channel_num, settings in overrides.items():
                mem = radio.get_memory(channel_num)
                for key, value in settings.items():
                    setattr(mem, key, value)
                radio.set_memory(mem)

            # Save
            output_path = os.path.join(output_dir, f'{radio_name}.img')
            radio.save(output_path)
            print(f"Generated: {output_path}")

    def verify_all(self, config_dir):
        """Verify all configs are valid"""
        for filename in os.listdir(config_dir):
            if filename.endswith('.img'):
                filepath = os.path.join(config_dir, filename)
                try:
                    radio = chirp.load_image(filepath)
                    channels = sum(1 for i in range(1, 200)
                                  if radio.get_memory(i) and radio.get_memory(i).freq)
                    print(f"OK: {filename} ({channels} channels)")
                except Exception as e:
                    print(f"ERROR: {filename}: {e}")

# Usage
configs = {
    'radio_alpha': {1: {'name': 'ALPHA-WATCH', 'power': 'High'}},
    'radio_bravo': {1: {'name': 'BRAVO-WATCH', 'power': 'Medium'}},
}

manager = FleetManager('master.img', configs)
manager.generate_configs('./configs/')
```

### Frequency Planning

```python
#!/usr/bin/env python3
"""Frequency planning and allocation"""

class FrequencyPlanner:
    def __init__(self):
        self.bands = {
            '2m': {'start': 144.000, 'end': 148.000, 'step': 0.0125},
            '70cm': {'start': 420.000, 'end': 450.000, 'step': 0.025},
            'FRS': {'start': 462.5625, 'end': 467.7500, 'step': 0.00625},
            'GMRS': {'start': 462.5625, 'end': 467.7500, 'step': 0.00625},
        }
        self.allocated = set()

    def allocate_channel(self, band, name, offset=0.600):
        """Allocate next available channel in band"""
        band_info = self.bands[band]
        freq = band_info['start']

        while freq < band_info['end']:
            if freq not in self.allocated:
                self.allocated.add(freq)
                return {
                    'name': name,
                    'frequency': freq,
                    'offset': offset,
                    'band': band
                }
            freq += band_info['step']

        raise ValueError(f"No available channels in {band}")

    def generate_plan(self, channels_per_band=10):
        """Generate frequency plan"""
        plan = {}
        for band in self.bands:
            plan[band] = []
            for i in range(channels_per_band):
                try:
                    ch = self.allocate_channel(band, f"{band}_{i+1}")
                    plan[band].append(ch)
                except ValueError:
                    break
        return plan

# Usage
planner = FrequencyPlanner()
plan = planner.generate_plan(5)
for band, channels in plan.items():
    print(f"\n{band}:")
    for ch in channels:
        print(f"  {ch['name']}: {ch['frequency']:.4f} MHz")
```

## Chapter 6: Real-World Applications

### Emergency Services Programming

```python
#!/usr/bin/env python3
"""Program emergency frequencies into radio"""

emergency_channels = [
    {'number': 1, 'name': 'CALL 2M', 'freq': 146.520000, 'power': 'High'},
    {'number': 2, 'name': 'CALL 70CM', 'freq': 446.000000, 'power': 'High'},
    {'number': 3, 'name': 'SKYWARN', 'freq': 146.940000, 'tone_mode': 'TSQL', 'tone': 100.0},
    {'number': 4, 'name': 'ARES', 'freq': 145.490000, 'tone_mode': 'TSQL', 'tone': 100.0},
    {'number': 5, 'name': 'NWS WX', 'freq': 162.550000, 'power': 'High'},
]

# Program into radio
programmer = RadioProgrammer('emergency.img')
programmer.bulk_program(emergency_channels)
```

### Club Repeater Setup

```python
#!/usr/bin/env python3
"""Setup club repeater channels"""

repeater_channels = [
    {'number': 1, 'name': 'CLUB 2M RPT', 'freq': 145.230000,
     'offset': 0.600, 'duplex': '-', 'tone_mode': 'TSQL', 'tone': 100.0},
    {'number': 2, 'name': 'CLUB 70CM RPT', 'freq': 444.550000,
     'offset': 5.000, 'duplex': '+', 'tone_mode': 'TSQL', 'tone': 100.0},
    {'number': 3, 'name': 'CLUB 10M', 'freq': 28.450000,
     'tone_mode': 'None', 'power': 'High'},
]

programmer = RadioProgrammer('club.img')
programmer.bulk_program(repeater_channels)
```

### Event Communication Setup

```python
#!/usr/bin/env python3
"""Setup communications for events (marathon, parade, etc.)"""

event_channels = [
    {'number': 1, 'name': 'EVENT CMD', 'freq': 451.125000,
     'tone_mode': 'TSQL', 'tone': 100.0, 'power': 'High'},
    {'number': 2, 'name': 'EVENT SEC', 'freq': 451.375000,
     'tone_mode': 'TSQL', 'tone': 100.0, 'power': 'Medium'},
    {'number': 3, 'name': 'EVENT MED', 'freq': 451.625000,
     'tone_mode': 'TSQL', 'tone': 100.0, 'power': 'Medium'},
    {'number': 4, 'name': 'EVENT LOG', 'freq': 451.875000,
     'tone_mode': 'TSQL', 'tone': 100.0, 'power': 'Low'},
]

programmer = RadioProgrammer('event.img')
programmer.bulk_program(event_channels)
```

### Multi-Radio Fleet Programming

```bash
#!/bin/bash
# Program entire fleet from master image

MASTER="fleet_master.img"
PORT="/dev/ttyUSB0"

# Radio definitions
declare -A RADIOS
RADIOS["cmd-radio"]="baofeng_uv5r"
RADIOS["sec-radio"]="baofeng_uv5r"
RADIOS["med-radio"]="baofeng_uv5r"

for name in "${!RADIOS[@]}"; do
    model="${RADIOS[$name]}"
    echo "Programming $name ($model)..."

    # Create custom image for this radio
    python3 -c "
import chirp
radio = chirp.load_image('$MASTER')
# Apply any radio-specific customizations here
radio.save('/tmp/${name}.img')
"

    # Write to radio
    chirp --clone-write -p "$PORT" -r "$model" "/tmp/${name}.img"

    if [ $? -eq 0 ]; then
        echo "  Success!"
    else
        echo "  Failed!"
    fi

    read -p "Swap radio and press Enter..."
done
```

## Chapter 7: Detection & Defense

### RF Security Implications

CHIRP is a legitimate tool for programming radios, but understanding its capabilities helps in security assessments:

**Potential Misuse:**
- Programming radios to unauthorized frequencies
- Cloning configurations between radios
- Setting up unauthorized repeaters

**Defensive Measures:**
- Monitor for unauthorized frequency usage
- Implement radio access controls
- Regular audits of radio configurations
- Use encrypted communications where possible

### Detecting Unauthorized Radio Programming

```bash
# Monitor serial port activity
sudo lsof /dev/ttyUSB*

# Check for CHIRP processes
ps aux | grep chirp

# Monitor USB activity
lsusb -v | grep -A5 "CHIRP\|Programming"
```

### Radio Security Best Practices

1. **Access Control**: Restrict who can program radios
2. **Configuration Backup**: Regular backups of authorized configurations
3. **Frequency Monitoring**: Regular scans of expected frequency bands
4. **Audit Trail**: Log all radio programming activities
5. **Encryption**: Use encrypted modes where supported

### Incident Response

1. Detect unauthorized radio programming
2. Identify affected radios
3. Restore authorized configurations from backups
4. Implement additional access controls
5. Document findings

## Chapter 8: Troubleshooting & Resources

### Common Errors

**"Could not find port"**
```bash
# Check serial devices
ls /dev/ttyUSB*

# Check permissions
ls -la /dev/ttyUSB0

# Add user to dialout group
sudo usermod -a -G dialout $USER

# Check if device is in use
lsof /dev/ttyUSB0
```

**"Radio not responding"**
```bash
# Verify cable connection
# Check cable type matches radio
# Try different USB port
# Check radio is powered on
# Verify correct vendor/model selection in CHIRP
```

**"Image is corrupted"**
```bash
# Re-read from radio
chirp --clone-read -p /dev/ttyUSB0 -r baofeng_uv5r -o fresh.img

# Verify image
chirp --info -i fresh.img
```

**"Import/Export errors"**
```bash
# Check CSV format
head -1 channels.csv

# Verify encoding
file -i channels.csv

# Fix line endings
dos2unix channels.csv
```

### FAQ

**Q: Is CHIRP legal to use?**
A: Yes, CHIRP is a legitimate radio programming tool. However, programming radios to frequencies outside your licensed privileges is illegal.

**Q: Which programming cable do I need?**
A: Most Baofeng/WouXun/TYT radios use K-type (2-pin) cables. Kenwood, Yaesu, and Icom have specific cables. Check your radio's documentation.

**Q: Can CHIRP damage my radio?**
A: CHIRP reads and writes memory data. If you write an incorrect image, you could potentially program the radio to transmit on unauthorized frequencies, but the radio hardware itself is unlikely to be damaged.

**Q: How do I backup my radio?**
A: Connect the radio, launch CHIRP, select Radio → Download From Radio, choose your port/vendor/model, and save the .img file.

**Q: Can I program DMR radios?**
A: CHIRP has limited DMR support. For full DMR programming, use manufacturer-specific software (e.g., CPS for TYT, BDEdit for Baofeng DMR radios).

### Resources

- **CHIRP Website**: https://chirpmyradio.com
- **GitHub**: https://github.com/kk7ds/chirp
- **Documentation**: https://chirpmyradio.com/wiki
- **Supported Radios**: https://chirpmyradio.com/wiki/SupportedRadios
- **Programming Cables**: https://www.amazon.com/s?k=baofeng+programming+cable
- **Amateur Radio**: https://www.arrl.org

### License

CHIRP is released under the GNU General Public License version 2 (GPLv2).
