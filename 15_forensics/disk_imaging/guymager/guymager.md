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
  - gnome-disk-utility
---

# Guymager — GUI Forensic Imaging Tool

## Chapter 1: Overview

Guymager is a free, open-source GUI forensic disk imaging tool written in Pascal using the Free Pascal Compiler and the Lazarus IDE. It provides a graphical interface for creating forensic images of storage devices, with built-in hash verification and multiple output format support.

### Why Guymager Exists

While command-line tools like `dd`, `dc3dd`, and `ddrescue` are powerful, they require memorizing complex flag combinations and are prone to operator error. Guymager provides a point-and-click interface that:

1. Lists all connected storage devices with details
2. Allows one-click forensic imaging with preset configurations
3. Computes and verifies hashes automatically
4. Supports multiple output formats (raw, EWF, AFF)
5. Logs all operations for chain of custody

### Core Capabilities

| Feature | Description |
|---|---|
| Device listing | Auto-detects all block devices with size/model info |
| Multiple formats | RAW (dd), E01 (EWF), AFF, split raw |
| Hash algorithms | MD5, SHA-1, SHA-256, SHA-512 (any combination) |
| Progress tracking | Real-time progress bar, speed, ETA |
| Image verification | Automatic post-imaging hash verification |
| Split images | Split large images into manageable chunks |
| Case metadata | Enter case number, examiner name, notes |
| Log generation | Detailed operation log for chain of custody |

### When to Use Guymager

- Quick forensic imaging when command-line tools are overkill
- Training environments where visual feedback helps
- Incidents where speed and ease-of-use matter
- Situations requiring E01 (Expert Witness Format) output
- When you need a GUI for non-technical examiners
- Quick triage imaging before detailed command-line work

### Output Format Comparison

| Format | Extension | Pros | Cons |
|---|---|---|---|
| RAW | .dd, .raw | Universal, mountable | Large, no metadata |
| EWF (E01) | .E01 | Compression, metadata, split | Proprietary, slower |
| AFF | .aff | Open standard, metadata, compression | Less tool support |
| Split RAW | .001, .002 | FAT32-friendly | Multiple files |

### Limitations

- GUI-only (no command-line scripting capability)
- Limited compared to ddrescue for damaged media
- No advanced options like pattern matching
- Cannot resume interrupted imaging (unlike ddrescue)
- Linux only (no Windows or macOS version)
- Development pace is slower than alternatives

## Chapter 2: Installation

### On Kali Linux

```bash
sudo apt update
sudo apt install guymager
```

### Launch

```bash
# Launch with GUI
guymager &

# Or from applications menu
# Applications -> Forensics -> Forensic Imaging Tools -> guymager
```

### Dependencies

Guymager automatically pulls in:
- Free Pascal Compiler runtime
- Lazarus components
- libcrypto (for hashing)
- libewf (for E01 support)

### Verify Installation

```bash
guymager --version
# or
dpkg -l | grep guymager
```

### From Source

```bash
sudo apt install fpc lazarus libewf-dev libcrypto++-dev
git clone https://gitlab.com/guymager/guymager.git
cd guymager
lazbuild guymager.lpi
sudo cp output/guymager /usr/local/bin/
```

## Chapter 3: Basic Usage

### Launching Guymager

```bash
# Normal launch
guymager &

# As root (required for raw device access)
sudo guymager &
```

### Interface Overview

When Guymager launches, it displays:

1. **Device List Panel**: Shows all detected block devices
   - Device path (/dev/sda, /dev/sdb, etc.)
   - Size
   - Model/Description
   - Serial number (if available)

2. **Action Panel**: Available operations for selected device
   - Acquire Image (forensic imaging)
   - View / Information (device details)

3. **Log Panel**: Real-time operation log
4. **Progress Panel**: Progress bar, speed, ETA

### Core Workflow

#### Step 1: Select Device

From the device list, click on the source device you want to image. The device details appear in the information panel.

#### Step 2: Click "Acquire Image"

Click the "Acquire Image" button to open the imaging configuration dialog.

#### Step 3: Configure Imaging

The configuration dialog offers:

| Option | Description |
|---|---|
| Target Directory | Where to save the image |
| Target Filename | Base name for image files |
| Image Format | RAW (.dd), E01 (.E01), AFF (.aff) |
| Split Size | Max file size (for split images) |
| Hash Algorithm(s) | MD5, SHA-1, SHA-256, SHA-512 |
| Case Number | Case identifier |
| Examiner | Examiner name |
| Description | Free-text notes |

#### Step 4: Start Imaging

Click "Start" to begin. Guymager shows:
- Real-time progress bar
- Current speed (MB/s)
- Estimated time remaining
- Bytes copied

#### Step 5: Verify

After imaging completes, Guymager automatically verifies the hashes and reports the result.

### Command-Line Equivalent

Guymager doesn't have a command-line interface, but the operations it performs correspond to:

```bash
# RAW image with MD5 hash
dc3dd if=/dev/sda of=/evidence/image.dd hash=md5 log=/evidence/image.log

# E01 image (requires ewfacquire)
ewfacquire /dev/sda -t /evidence/image -C case001 -D "description" -e examiner
```

### Keyboard Shortcuts

| Key | Action |
|---|---|
| `F1` | Help |
| `F5` | Refresh device list |
| `Esc` | Cancel current operation |
| `Ctrl+Q` | Quit |

### Sample Configuration Dialog

```
+-----------------------------------------------+
|         Forensic Image Acquisition            |
+-----------------------------------------------+
| Target Directory: [/evidence/case001]         |
| Target Filename:  [suspect_laptop]            |
|                                               |
| Format:     ( ) RAW (.dd)                     |
|             (X) EWF (.E01)                    |
|             ( ) AFF (.aff)                    |
|                                               |
| Split Size: [4096 MB]                         |
|                                               |
| Hash Algorithms:                              |
|   [X] MD5                                     |
|   [X] SHA-1                                   |
|   [X] SHA-256                                 |
|   [ ] SHA-512                                 |
|                                               |
| Case Number: [CASE-2026-0042]                 |
| Examiner:    [jdoe]                           |
| Description: [Suspect laptop hard drive]      |
|                                               |
|          [Start]  [Cancel]                    |
+-----------------------------------------------+
```

## Chapter 4: Intermediate Usage and Scripting

### Automation Limitations

Guymager is a GUI-only tool and cannot be directly scripted. However, you can:

1. **Use ewfacquire** for E01 format scripting:
```bash
# Equivalent to Guymager E01 acquisition
ewfacquire /dev/sdb \
    -t /evidence/image \
    -C "CASE-2026-0042" \
    -D "Suspect laptop" \
    -e "jdoe" \
    -c \
    -f 2048 \
    -S 4GiB \
    -m 2 \
    -M "First,Last" \
    -l /evidence/acquire.log
```

2. **Use dc3dd** for RAW format scripting:
```bash
# Equivalent to Guymager RAW acquisition
dc3dd if=/dev/sdb \
    of=/evidence/image.dd \
    hash=md5,sha1,sha256 \
    log=/evidence/image.log
```

### Batch Imaging Script (Using Alternatives)

```bash
#!/bin/bash
# batch_image.sh — Batch forensic imaging (Guymager alternative)

EVIDENCE="/evidence/batch_$(date +%Y%m%d_%H%M%S)"
mkdir -p "${EVIDENCE}"

for dev in /dev/sd{b,c,d}; do
    [ -b "${dev}" ] || continue
    name=$(basename "${dev}")
    
    echo "[${name}] Starting imaging..."
    
    # RAW with multiple hashes
    dc3dd if="${dev}" \
        of="${EVIDENCE}/${name}_image.dd" \
        hash=md5,sha1,sha256 \
        log="${EVIDENCE}/${name}_image.log"
    
    echo "[${name}] Complete."
done
```

### Monitoring Guymager Operations

```bash
# Guymager creates log files
find /tmp -name "guymager*" -type f 2>/dev/null

# Check Guymager process
ps aux | grep guymager

# Check disk I/O during imaging
iostat -x 1 5
```

### Using Guymager in Virtual Environments

```bash
# In VM environments, pass through physical devices
# or use raw device files

# Create a raw device file from VM disk
qemu-img convert -f qcow2 -O raw vm_disk.qcow2 /evidence/vm_raw.dd

# Then image the raw file with Guymager
```

## Chapter 5: Advanced Configurations

### E01 Format Details

E01 (Expert Witness Format) is the most common forensic image format. Key features:

- **Compression**: Uses zlib compression (typically 2:1 ratio)
- **Metadata**: Stores case info, examiner, timestamps
- **Segmentation**: Automatically splits into manageable files
- **Integrity**: Built-in CRC checks per segment
- **Encryption**: Optional AES-256 encryption

### AFF Format Details

AFF (Advanced Forensics Format) is an open standard:

- **Open format**: No proprietary dependencies
- **Compression**: Uses zlib
- **Metadata**: XML-based metadata storage
- **Encryption**: Optional AES encryption
- **Verification**: Built-in hash verification

### Split Image Configuration

| Split Size | Use Case |
|---|---|
| 640 MB | CD-sized segments |
| 2 GB | FAT32 limit |
| 4 GB | Standard forensic split |
| 16 GB | Large drive backup |
| No split | Single file (ext4/NTFS) |

### Case Metadata Best Practices

Always fill in:
- **Case Number**: Your organization's case ID
- **Examiner**: Your name or badge number
- **Description**: Brief description of evidence
- **Equipment**: Write blocker make/model
- **Date/Time**: Guymager records this automatically

### Hash Algorithm Selection

| Algorithm | Speed | Security | Recommendation |
|---|---|---|---|
| MD5 | Fast | Weak (collisions found) | Quick verification only |
| SHA-1 | Fast | Moderate | Minimum for forensic work |
| SHA-256 | Moderate | Strong | Recommended for all cases |
| SHA-512 | Slower | Very strong | High-security cases |

### Multi-Algorithm Strategy

Select multiple hash algorithms for defense in depth:
- **MD5 + SHA-256**: Standard forensic practice
- **SHA-1 + SHA-256**: Good balance of speed and security
- **MD5 + SHA-1 + SHA-256**: Maximum verification

## Chapter 6: Real-World Scenarios

### Scenario 1: First Responder — Quick Triage Imaging

**Context**: You arrive at a crime scene and need to quickly image a suspect's laptop before powering it down.

**Approach**:
```bash
# 1. Connect write blocker
# 2. Connect laptop drive to forensic workstation
# 3. Launch Guymager
sudo guymager &

# 4. Select the laptop drive
# 5. Click "Acquire Image"
# 6. Configure:
#    - Format: E01 (compressed, saves time)
#    - Split: 4096 MB
#    - Hashes: MD5 + SHA-256
#    - Case Number: [fill in]
#    - Description: "Laptop from suspect's desk, scene #42"
# 7. Click "Start"
# 8. While imaging, document the chain of custody
```

### Scenario 2: Lab — Standard Forensic Imaging

**Context**: In a forensic lab, you need to create a forensically sound image of a seized hard drive with full documentation.

**Approach**:
```bash
# 1. Connect hardware write blocker
# 2. Launch Guymager with root privileges
sudo guymager &

# 3. Select the evidence drive
# 4. Click "Acquire Image"
# 5. Configure:
#    - Format: RAW (.dd) — for maximum compatibility
#    - Split: 4096 MB (for FAT32 transfer)
#    - Hashes: MD5 + SHA-1 + SHA-256
#    - Case Number: CASE-2026-0042
#    - Examiner: [your name]
#    - Description: [detailed description]
# 6. Start imaging
# 7. After completion, verify hashes
# 8. Create a backup copy to separate media
# 9. Document everything in case notes
```

### Scenario 3: Training — Teaching Forensic Imaging

**Context**: Teaching a digital forensics course. Students need to understand the imaging process.

**Approach**:
```bash
# 1. Provide each student with a USB drive (evidence)
# 2. Demonstrate Guymager workflow:
#    - Launch: sudo guymager &
#    - Identify devices
#    - Configure imaging parameters
#    - Start imaging
#    - Verify results

# 3. Have students compare with command-line:
dc3dd if=/dev/sdb of=/evidence/image.dd hash=md5 log=/evidence/image.log

# 4. Discuss output formats (RAW vs E01 vs AFF)
# 5. Practice hash verification
# 6. Review chain of custody documentation
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
find / -name "guymager*" -type f 2>/dev/null

# E01 image files
find / -name "*.E01" -o -name "*.e01" 2>/dev/null

# AFF image files
find / -name "*.aff" -o -name "*.AFF" 2>/dev/null
```

**Process Monitoring**:
```bash
# Monitor for Guymager
ps aux | grep guymager
pstree | grep guymager
```

### Defensive Measures

- Monitor for `guymager` execution in audit logs
- Alert on GUI forensic tool launches
- Implement USB device control policies
- Monitor for E01/AFF file creation
- Track write blocker connections

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
| `No devices found` | No block devices visible | Check connections, use `lsblk` |
| `Write error` | Destination full or read-only | Check destination space |
| `Hash mismatch` | Corruption during imaging | Re-image with verified media |
| `GUI not starting` | Display server issue | Check X11/Wayland, try `xhost +` |

### Performance Issues

```bash
# Check I/O utilization
iostat -x 1 5

# Check for competing processes
iotop -o

# Ensure write blocker is working properly
dmesg | tail -20
```

### Device Not Detected

```bash
# Check device visibility
lsblk
sudo fdisk -l

# Check kernel messages
dmesg | tail -20

# Ensure device is not mounted
sudo umount /dev/sdb*

# Try with different permissions
sudo chmod 666 /dev/sdb
```

### GUI Issues

```bash
# If GUI doesn't start, check display
echo $DISPLAY
export DISPLAY=:0

# Try with explicit display
sudo DISPLAY=:0 guymager &

# Check for missing libraries
ldd $(which guymager) | grep "not found"
```

### Hash Verification Failure

```bash
# Re-verify manually
md5sum /evidence/image.dd
sha256sum /evidence/image.dd

# Compare against Guymager's recorded hashes
grep -A5 "Hash" /evidence/image.log

# If mismatch, re-image with fresh media
```

## Resources

- **Guymager Homepage**: https://guymager.sourceforge.io/
- **Guymager Documentation**: https://guymager.sourceforge.io/user.html
- **SANS DFIR**: https://www.sans.org/dfir/
- **Kali Linux Documentation**: https://www.kali.org/docs/
- **Forensic Focus**: https://www.forensicfocus.com/

---

*Last updated: 2026-07-18*
