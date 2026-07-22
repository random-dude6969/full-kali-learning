---
tool_name: guymager
display_name: Guymager
mitre_tactic: collection
mitre_tactic_id: TA0009
install_command: sudo apt install guymager
github_repo: https://guymager.sourceforge.io/
kali_tools_page: https://www.kali.org/tools/guymager/
difficulty_level: beginner
tags:
  - forensics
  - disk-imaging
  - gui
  - evidence
  - dfir
related_tools:
  - dc3dd
  - dcfldd
  - ddrescue
  - autopsy
---

# Guymager — GUI Forensic Disk Imager

## Chapter 1: Overview

Guymager is a free, open-source graphical forensic disk imaging tool written in Free Pascal. It provides a user-friendly GUI for creating forensic disk images with built-in hash verification, logging, and EWF (Expert Witness Format) support. It is commonly included in forensic Linux distributions like CAINE and Kali Linux.

### Why Guymager Exists

Command-line imaging tools like dc3dd and dcfldd are powerful but require memorizing complex syntax. Guymager provides the same forensic capabilities through a point-and-click interface, making forensic imaging accessible to analysts who may not be comfortable with the command line.

### Core Capabilities

| Feature | Description |
|---|---|
| GUI Interface | Point-and-click forensic imaging |
| Hash Verification | MD5, SHA-1, SHA-256 computed during imaging |
| EWF Support | Expert Witness Format (E01) output |
| Raw DD Support | Traditional dd image output |
| AFF Support | Advanced Forensic Format output |
| Log Generation | Detailed imaging logs with chain of custody |
| Split Images | Automatic image splitting at configurable sizes |
| Pause/Resume | Pause and resume imaging operations |

### Output Formats

| Format | Extension | Description |
|---|---|---|
| raw | .dd, .raw | Bit-for-bit copy, universally compatible |
| EWF | .E01 | Expert Witness Format, compressed, metadata support |
| AFF | .aff | Advanced Forensic Format, open standard |

### When to Use Guymager

- Forensic imaging when a GUI is preferred
- Training new analysts who are not comfortable with CLI
- Quick imaging with visual confirmation of progress
- When E01 format output is required for compatibility with other tools
- Lab environments where multiple analysts share imaging stations

### Comparison with CLI Tools

| Feature | dc3dd | dcfldd | ddrescue | guymager |
|---|---|---|---|---|
| Interface | CLI | CLI | CLI | GUI |
| E01 output | No | No | No | Yes |
| Hash verification | Yes | Yes | No | Yes |
| Bad sector handling | Basic | Basic | Advanced | Basic |
| Split output | Yes | Yes | Yes | Yes |
| Cost | Free | Free | Free | Free |

### Limitations

- No advanced bad sector handling (use ddrescue for damaged media)
- GUI dependency makes it unsuitable for headless servers
- Fewer command-line options than dc3dd or dcfldd
- E01 format may have licensing considerations in some jurisdictions
- Slower than CLI alternatives for very large images
- Limited scripting/automation capabilities

## Chapter 2: Installation

### On Kali Linux

```bash
sudo apt update
sudo apt install guymager
```

### Verify Installation

```bash
guymager --version
```

### Launch Guymager

```bash
# As root (required for device access)
sudo guymager
```

### Dependencies

- Free Pascal Runtime
- GTK+ 2.x or 3.x
- libewf (for E01 support)
- libaff (for AFF support)

### Post-Installation Verification

```bash
which guymager
dpkg -l | grep guymager
```

## Chapter 3: Basic Usage

### Launching Guymager

```bash
sudo guymager
```

The GUI opens with three main sections:
1. **Device List**: Shows all detected block devices
2. **Info Panel**: Shows selected device details
3. **Action Panel**: Imaging controls and progress

### GUI Layout

```
+----------------------------------------------------------+
| Guymager - Forensic Imager                               |
+----------------------------------------------------------+
| Devices:                                                  |
| [/dev/sda] 500 GB  -  WDC WD5000LPVX-22V0TT0            |
| [/dev/sdb] 1 TB   -  Seagate ST1000LM035-1RK172         |
| [/dev/sdc] 16 GB  -  Kingston DataTraveler 3.0           |
+----------------------------------------------------------+
| Device Info:                                              |
| Model: Kingston DataTraveler 3.0                         |
| Serial: 0019E06C5ACDB8400854                             |
| Size: 15.8 GB                                            |
| Sector Size: 512 bytes                                   |
+----------------------------------------------------------+
| [Image Device]  [Hash Only]  [Info Only]                 |
+----------------------------------------------------------+
```

### Basic Imaging Steps

1. Launch Guymager: `sudo guymager`
2. Select the source device from the device list
3. Click "Image Device"
4. Configure output settings:
   - Destination path
   - Output format (raw, E01, AFF)
   - Hash algorithms (MD5, SHA-1, SHA-256)
   - Split size (if needed)
5. Click "Start" to begin imaging
6. Monitor progress in the status bar

### Command-Line Options

```
guymager [options]

Options:
  -h, --help          Show help message
  -v, --version       Show version
  -i, --info          Show device info only (no imaging)
  --device <device>   Specify device to image
  --target <path>     Specify output path
  --format <fmt>      Output format: raw, ewf, aff
  --hash <algo>       Hash algorithm: md5, sha1, sha256
  --split <size>      Split output at size (MB)
  --log <path>        Log file path
```

### Imaging Workflow

#### Step 1: Select Source Device
Click on the device in the list. The info panel shows details.

#### Step 2: Choose Output Settings
Click "Image Device" to open the imaging dialog:

| Setting | Options | Recommended |
|---|---|---|
| Output Path | Browse to destination | Separate physical drive |
| Format | Raw (dd), EWF (E01), AFF | E01 for forensics |
| Hash | MD5, SHA-1, SHA-256 | MD5 + SHA-256 |
| Split Size | None, 650MB, 2GB, 4GB, custom | 4GB for FAT32 compatibility |
| Description | Free text | Case number and operator |

#### Step 3: Start Imaging
Click "Start" to begin. Progress is shown in real-time.

#### Step 4: Verify
When complete, Guymager displays:
- Source hash
- Destination hash
- Verification status (pass/fail)
- Detailed log

### Sample Imaging Output

```
Imaging started at 2026-07-18 10:30:15
Source: /dev/sdc (Kingston DataTraveler 3.0)
Destination: /evidence/case001/usb_image.E01
Format: EWF (E01)
Hash algorithms: MD5, SHA-256
Split size: 4294967296 bytes (4 GB)

Progress: [============================] 100%
Speed: 85.3 MB/s
Elapsed: 02:34:12
Remaining: 00:00:00

Source MD5:    3d4f2bf07dc1faba79b92a3796499564
Dest MD5:      3d4f2bf07dc1faba79b92a3796499564
Verification: PASS

Source SHA256:  7b8e9a1c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b1c2d3e4f5a6b7c8d9e0f1a
Dest SHA256:    7b8e9a1c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b1c2d3e4f5a6b7c8d9e0f1a
Verification: PASS

Log saved to: /evidence/case001/usb_image.E01.log
```

## Chapter 4: Intermediate Usage and Scripting

### Automating Guymager from Command Line

While Guymager is primarily a GUI tool, it can be scripted:

```bash
#!/bin/bash
# auto_image.sh — Automated Guymager imaging

DEVICE="$1"
OUTPUT="$2"
FORMAT="${3:-ewf}"
HASH="${4:-md5,sha256}"

# Launch Guymager with parameters
guymager \
    --device "${DEVICE}" \
    --target "${OUTPUT}" \
    --format "${FORMAT}" \
    --hash "${HASH}" &
    
GUIPID=$!

# Wait for completion
wait ${GUIPID}
echo "[*] Imaging complete."
```

### Batch Imaging Script

```bash
#!/bin/bash
# batch_image.sh — Image multiple devices sequentially

EVIDENCE="/evidence/batch_$(date +%Y%m%d_%H%M%S)"
mkdir -p "${EVIDENCE}"

for device in /dev/sd{b,c,d}; do
    [ -b "${device}" ] || continue
    name=$(basename "${device}")
    
    echo "[*] Imaging ${device}..."
    sudo guymager \
        --device "${device}" \
        --target "${EVIDENCE}/${name}.E01" \
        --format ewf \
        --hash md5,sha256
    
    echo "[*] ${device} complete."
done

echo "[*] All imaging complete."
```

### Monitoring Imaging Progress

```bash
#!/bin/bash
# monitor_image.sh — Monitor Guymager imaging progress

LOGFILE="$1"
INTERVAL=10

echo "Monitoring Guymager progress..."
echo "Press Ctrl-C to stop."
echo ""

while pgrep -x guymager > /dev/null; do
    if [ -f "${LOGFILE}" ]; then
        PROGRESS=$(grep -oP 'Progress: \K[0-9]+' "${LOGFILE}" 2>/dev/null || echo "0")
        SPEED=$(grep -oP 'Speed: \K[0-9.]+ [A-Z]+/s' "${LOGFILE}" 2>/dev/null || echo "N/A")
        ELAPSED=$(grep -oP 'Elapsed: \K[0-9:]+' "${LOGFILE}" 2>/dev/null || echo "00:00:00")
        echo "[$(date +%H:%M:%S)] Progress: ${PROGRESS}% | Speed: ${SPEED} | Elapsed: ${ELAPSED}"
    fi
    sleep "${INTERVAL}"
done

echo "[*] Imaging complete."
```

### Post-Imaging Verification Script

```bash
#!/bin/bash
# verify_image.sh — Verify Guymager image against log

IMAGE="$1"
LOGFILE="${1}.log"

if [ ! -f "${LOGFILE}" ]; then
    echo "[!] Log file not found: ${LOGFILE}"
    exit 1
fi

echo "[*] Verifying image: ${IMAGE}"
echo "[*] Log file: ${LOGFILE}"

# Extract hashes from log
SRC_MD5=$(grep -oP 'Source MD5:\s+\K[0-9a-f]+' "${LOGFILE}" 2>/dev/null)
DST_MD5=$(grep -oP 'Dest MD5:\s+\K[0-9a-f]+' "${LOGFILE}" 2>/dev/null)

if [ -n "${SRC_MD5}" ] && [ -n "${DST_MD5}" ]; then
    if [ "${SRC_MD5}" = "${DST_MD5}" ]; then
        echo "[PASS] MD5 verification successful"
    else
        echo "[FAIL] MD5 mismatch"
        echo "  Source: ${SRC_MD5}"
        echo "  Dest:   ${DST_MD5}"
    fi
else
    echo "[!] Could not extract hashes from log"
fi
```

## Chapter 5: Advanced Configurations

### Custom Split Sizes

Guymager supports automatic image splitting:
- 650 MB (CD size)
- 2 GB (FAT16 limit)
- 4 GB (FAT32 limit)
- Custom size in MB

### E01 Format Advantages

Expert Witness Format (E01) provides several advantages over raw dd images:
- Built-in compression (typically 40-60% reduction)
- Metadata storage (case info, operator, notes)
- Built-in hash verification
- Password protection
- Segment splitting
- Used by EnCase, FTK, and most forensic tools

### AFF Format

Advanced Forensic Format (AFF) is an open standard:
- Compression support
- Metadata fields
- Digital signatures
- No proprietary licensing

### Integration with Other Tools

```bash
# Image with Guymager, analyze with Autopsy
guymager --device /dev/sdb --target /evidence/image.E01 --format ewf
autopsy /evidence/image.E01

# Image with Guymager, analyze with Sleuth Kit
guymager --device /dev/sdb --target /evidence/image.dd --format raw
fls -r /evidence/image.dd

# Image with Guymager, search with YARA
guymager --device /dev/sdb --target /evidence/image.dd --format raw
yara -r /path/to/rules /evidence/image.dd
```

### Network Transfer

```bash
# Image to network share
guymager --device /dev/sdb --target /mnt/nas/evidence/image.E01 --format ewf

# Image via SSH
guymager --device /dev/sdb --target /tmp/image.E01 --format ewf
rsync -avz /tmp/image.E01 investigator@192.168.1.100:/evidence/
```

## Chapter 6: Real-World Scenarios

### Scenario 1: Corporate Incident Response

**Context**: A corporate laptop is suspected of containing malware. The IT team needs to image the drive for analysis.

**Approach**:
```bash
# 1. Boot from forensic USB (CAINE or Kali)
# 2. Connect the suspect laptop via write blocker
# 3. Launch Guymager
sudo guymager

# 4. Select the laptop drive
# 5. Configure:
#    - Output: /evidence/corp_incident_001/laptop.E01
#    - Format: EWF (E01)
#    - Hash: MD5 + SHA-256
#    - Split: 4GB

# 6. Start imaging
# 7. Wait for completion and verification
# 8. Review log file
cat /evidence/corp_incident_001/laptop.E01.log
```

### Scenario 2: Law Enforcement Evidence Collection

**Context**: A detective needs to image a suspect's desktop computer for a criminal investigation.

**Approach**:
```bash
# 1. Document the chain of custody
cat > /evidence/case_2026_0042/custody.txt << EOF
Case ID: 2026-0042
Date: $(date -u +"%Y-%m-%d %H:%M:%S UTC")
Officer: Det. Smith
Evidence: Suspect desktop hard drive
Tool: Guymager
Write Blocker: Tableau T35689iu
EOF

# 2. Launch Guymager with documentation
sudo guymager

# 3. Select the drive
# 4. Set description field to case number
# 5. Image to evidence drive
# 6. Verify hashes
# 7. Store evidence drive securely
```

### Scenario 3: Training Lab Setup

**Context**: Setting up a forensic training lab where students practice imaging.

**Approach**:
```bash
# 1. Prepare training drives
for i in {1..5}; do
    echo "Preparing training drive ${i}..."
    dd if=/dev/zero of=/dev/usb${i} bs=1M count=100
    mkfs.ext4 /dev/usb${i}
    
    # Add training data
    mount /dev/usb${i} /mnt
    cp -r /training/data/* /mnt/
    umount /mnt
done

# 2. Launch Guymager for each drive
for i in {1..5}; do
    sudo guymager &
    # Students practice imaging via GUI
done
```

## Chapter 7: Detection and Defense

### Detecting Guymager Usage

**System Monitoring**:
```bash
# Check process history
lastcomm guymager
sa -c | grep guymager

# Check system logs
grep -r "guymager" /var/log/syslog* /var/log/auth.log* 2>/dev/null

# Check bash history
grep guymager ~/.bash_history
```

**File System Indicators**:
```bash
# Guymager log files
find / -name "*.E01.log" -o -name "*.dd.log" -o -name "*.aff.log" 2>/dev/null

# E01 image files
find / -name "*.E01" -size +1G 2>/dev/null

# AFF image files
find / -name "*.aff" -size +1G 2>/dev/null
```

**Process Monitoring**:
```bash
# Monitor for Guymager processes
ps aux | grep guymager
pstree | grep guymager
```

### Defensive Measures

- Monitor for `guymager` in command execution logs
- Alert on E01 file creation
- Implement USB device control policies
- Use filesystem monitoring on evidence directories

### Audit Configuration

```bash
# Add to /etc/audit/rules.d/forensics.rules
-a always,exit -F arch=b64 -S execve -F path=/usr/bin/guymager -F key=forensics_guymager
-a always,exit -F arch=b64 -S openat -F path=/dev/sd* -F key=forensics_devices
```

## Chapter 8: Troubleshooting

### Common Issues

| Error | Cause | Solution |
|---|---|---|
| `Permission denied` | Not root | Use `sudo guymager` |
| `Device not found` | Drive not detected | Check with `lsblk` |
| `Cannot write to destination` | Permissions or space | Check destination path and disk space |
| `Hash mismatch` | Corruption or tampering | Re-image with verified write blocker |
| `GUI not displaying` | No X11 forwarding | Use VNC or direct display |

### Performance Issues

```bash
# Check I/O utilization
iostat -x 1 5

# Check for competing processes
iotop -o

# Use raw format for faster imaging
guymager --device /dev/sdb --target /evidence/image.dd --format raw --hash md5
```

### GUI Issues

```bash
# If GUI does not display over SSH
ssh -X user@host sudo guymager

# Or use VNC
vncserver :1
DISPLAY=:1 sudo guymager

# Or use X11 forwarding
ssh -X user@host
sudo guymager
```

### Device Not Detected

```bash
# Check if drive is visible
lsblk
sudo fdisk -l

# Check kernel messages
dmesg | tail -20

# Try without write blocker
sudo guymager
```

### E01 Format Issues

```bash
# If E01 file is corrupted, try converting
# Use ewfverify to check
ewfverify /evidence/image.E01

# Or use ewfexport to convert to raw
ewfexport -t /evidence/image.raw /evidence/image.E01
```

## Resources

- **Guymager Homepage**: https://guymager.sourceforge.io/
- **CAINE Linux**: https://www.caine-live.net/
- **SANS DFIR**: https://www.sans.org/dfir/
- **Kali Linux Documentation**: https://www.kali.org/docs/
- **EWF Format Specification**: https://www.forensicswiki.org/wiki/Expert_Witness_Format

---

*Last updated: 2026-07-18*
