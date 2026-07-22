# PhotoRec — File Recovery Utility

## Table of Contents

- [Overview](#overview)
- [Installation](#installation)
- [Core Concepts](#core-concepts)
- [Command-Line Reference](#command-line-reference)
- [Interactive Mode Usage](#interactive-mode-usage)
- [Basic Usage](#basic-usage)
- [Advanced Usage](#advanced-usage)
- [Forensics Workflow Integration](#forensics-workflow-integration)
- [Performance Optimization](#performance-optimization)
- [Limitations and Considerations](#limitations-and-considerations)
- [Detection and Defense](#detection-and-defense)
- [Troubleshooting](#troubleshooting)
- [Best Practices](#best-practices)
- [Comparison with Other Tools](#comparison-with-other-tools)
- [References](#references)

---

## Overview

PhotoRec is a free, open-source data recovery utility designed to recover lost files from hard disks, CD-ROMs, and digital camera memory. Despite its name, PhotoRec recovers much more than just photos — it can recover videos, documents, archives, and dozens of other file types. Developed by Christophe Grenier (the same author as TestDisk), PhotoRec is bundled with TestDisk in the Kali Linux forensics toolkit.

PhotoRec uses file type-specific headers and footers to identify and extract files from raw disk data. It ignores the filesystem completely, making it effective even when the filesystem has been severely damaged, formatted, or overwritten. This makes PhotoRec one of the most versatile file recovery tools available for digital forensics.

### Key Features

- **Filesystem Independent**: Works on raw disk data, ignores filesystem structures
- **480+ File Types**: Supports recovery of hundreds of file formats
- **Cross-Platform**: Works on Windows, Linux, macOS, and other Unix systems
- **Non-Destructive**: Read-only recovery, never writes to source media
- **Bootable Media**: Can be run from a bootable USB or CD
- **Free and Open Source**: No licensing restrictions

### When to Use PhotoRec

| Scenario | Use PhotoRec? | Notes |
|----------|--------------|-------|
| Deleted files on healthy disk | Yes | Excellent choice |
| Formatted drive recovery | Yes | Primary use case |
| Corrupted filesystem | Yes | Works without filesystem |
| Need many file types | Yes | 480+ formats supported |
| Damaged disk (bad sectors) | No | Use ddrescue first |
| Large disk images | Yes | Fast scanning |
| Need file metadata | Partial | Limited metadata recovery |

---

## Installation

### Kali Linux

```bash
# PhotoRec is pre-installed on Kali Linux
sudo apt update
sudo apt install testdisk  # PhotoRec is bundled with TestDisk

# Verify installation
photorec --version
```

### Other Linux Distributions

```bash
# Debian/Ubuntu
sudo apt install testdisk

# Fedora/RHEL
sudo dnf install testdisk

# Arch Linux
sudo pacman -S testdisk

# From source
wget https://www.cgsecurity.org/testdisk-7.2.tar.bz2
tar xjf testdisk-7.2.tar.bz2
cd testdisk-7.2
./configure
make
sudo make install
```

### Building from Source

```bash
# Install dependencies
sudo apt install build-essential libncurses-dev

# Download and build
wget https://www.cgsecurity.org/testdisk-7.2.tar.bz2
tar xjf testdisk-7.2.tar.bz2
cd testdisk-7.2

# Configure and build
./configure --prefix=/usr
make -j$(nproc)
sudo make install

# Verify
photorec --version
```

### Running from Live Media

```bash
# Download Kali Linux ISO
# Write to USB drive
sudo dd if=kali-linux-2026.1-installer-amd64.iso of=/dev/sdb bs=4M status=progress

# Boot from USB
# PhotoRec will be available in the forensics menu
```

---

## Core Concepts

### How PhotoRec Works

PhotoRec operates by:

1. **Scanning raw data**: Reads disk sectors sequentially
2. **Identifying file signatures**: Matches known file headers (magic bytes)
3. **Extracting file data**: Copies data from header to footer or next header
4. **Saving recovered files**: Writes extracted data to output directory

### File Signatures

PhotoRec recognizes over 480 file types through their signatures:

| Category | Examples | Header Signature |
|----------|----------|-----------------|
| Images | JPEG, PNG, GIF, TIFF | Varies per format |
| Videos | MP4, AVI, MOV, MKV | Varies per format |
| Documents | PDF, DOC, DOCX, ODT | Varies per format |
| Archives | ZIP, RAR, 7Z, TAR.GZ | Varies per format |
| Audio | MP3, WAV, FLAC, OGG | Varies per format |
| Database | SQLite, MDB, DBF | Varies per format |

### Carving Modes

PhotoRec supports different carving strategies:

- **Whole**: Extract files completely (default)
- **Tail**: Extract only file tails
- **Unused**: Only scan unallocated space

### Filesystem Support

PhotoRec works with (but ignores) these filesystems:

- NTFS, FAT12/FAT16/FAT32
- ext2/ext3/ext4
- HFS+, HFS
- ReiserFS, JFS
- XFS, UFS
- And more

---

## Command-Line Reference

### Basic Syntax

```bash
photorec [options] <device_or_image>
```

### Options

| Option | Description |
|--------|-------------|
| `/cmd` | Command-line mode with arguments |
| `/dlog` | Debug log file |
| `/d` | Destination directory |
| `/e` | File extension filter |
| `/f` | Filesystem type |
| `/i` | Image file |
| `/o` | Output directory |
| `/r` | Resume previous recovery |
| `/s` | Scan for file types |
| `/t` | Target file size |
| `/h` | Help |

### Command-Line Mode

```bash
# Syntax: photorec /cmd <device> <options>

# Recover all file types
sudo photorec /cmd /dev/sda /d /output/ whole

# Recover specific file types
sudo photorec /cmd /dev/sda /d /output/ /e "jpg,png,pdf" whole

# Scan specific partition
sudo photorec /cmd /dev/sda1 /d /output/ whole

# Scan disk image
sudo photorec /cmd /path/to/image.raw /d /output/ whole
```

---

## Interactive Mode Usage

### Starting PhotoRec

```bash
# Start interactive mode
sudo photorec /dev/sda

# Or on a disk image
sudo photorec /path/to/image.raw
```

### Interactive Navigation

PhotoRec uses a text-based interface:

- **Arrow keys**: Navigate menus
- **Enter**: Select option
- **Arrow Right**: Next step
- **Arrow Left**: Previous step
- **Quit**: Exit PhotoRec

### Step-by-Step Interactive Recovery

#### Step 1: Select Disk

```
PhotoRec 7.2, Data Recovery Utility
Disk /dev/sda - 500 GB / 465 GiB

>[Proceed]  [  Quit  ]
```

Select the disk containing lost files.

#### Step 2: Select Partition

```
Disk /dev/sda - 500 GB / 465 GiB

  Partition                  Start        End    Size in sectors
  >Linux                    2048   1026047    1024000
   Linux               1026048  976771071  975745024
   [Search]  [  Quit  ]
```

Select the partition to scan (or "Search" for whole disk).

#### Step 3: Select File System

```
Please select the file system type

>[ext2/ext3]   Linux
  FAT12/FAT16   Windows
  NTFS          Windows
  HFS+          Mac
  [Unknown]     Other
```

Select the filesystem type (helps optimize scanning).

#### Step 4: Select Free or Whole Space

```
>[Free]     Scan for file from unallocated space only
  Whole     Scan whole partition (including allocated files)
```

- **Free**: Only scan unallocated space (faster, finds deleted files)
- **Whole**: Scan entire partition (slower, finds all files)

#### Step 5: Select Output Directory

```
Please select a destination directory:

>[/home/user/recovered]
  [/tmp]
  [/evidence]
```

Select where to save recovered files.

#### Step 6: Recovery in Progress

```
PhotoRec 7.2, Data Recovery Utility
Recovery in progress...

  Reading sector 12345678 of 976771072
  12345 recovered files
  
  [  Stop  ]
```

Wait for recovery to complete.

---

## Basic Usage

### Simple File Recovery

```bash
# Create output directory
mkdir -p /evidence/recovered

# Run PhotoRec on disk image
sudo photorec /evidence/image.raw

# Or use command-line mode
sudo photorec /cmd /evidence/image.raw /d /evidence/recovered/ whole
```

### Recovering from a Device

```bash
# Recover from a partition
sudo photorec /dev/sda1

# Recover from entire disk
sudo photorec /dev/sda
```

### Recovering Specific File Types

```bash
# Recover only JPEG and PNG files
sudo photorec /cmd /dev/sda /d /output/ /e "jpg,png" whole

# Recover only documents
sudo photorec /cmd /dev/sda /d /output/ /e "pdf,doc,docx,xlsx" whole
```

### Recovering from Disk Images

```bash
# Create image first
sudo dd if=/dev/sda of=/evidence/disk.img bs=4M status=progress

# Recover from image
sudo photorec /evidence/disk.img

# Or command-line mode
sudo photorec /cmd /evidence/disk.img /d /evidence/recovered/ whole
```

---

## Advanced Usage

### Command-Line Scenarios

#### Scenario 1: Maximum Recovery

```bash
# Scan entire disk, all file types, unallocated space
sudo photorec /cmd /dev/sda /d /evidence/recovered/ free

# Also scan allocated space
sudo photorec /cmd /dev/sda /d /evidence/recovered/ whole
```

#### Scenario 2: Selective Recovery

```bash
# Only recover specific file types
sudo photorec /cmd /dev/sda /d /output/ /e "jpg,png,gif,mp4" whole

# Only recover from specific partition
sudo photorec /cmd /dev/sda2 /d /output/ whole
```

#### Scenario 3: Recover from E01 Image

```bash
# Mount E01 image
sudo ewfmount evidence.E01 /mnt/ewf

# Recover from mounted image
sudo photorec /mnt/ewf/ewf1

# Unmount when done
sudo umount /mnt/ewf
```

#### Scenario 4: Scripted Recovery

```bash
#!/bin/bash
# Automated PhotoRec recovery

DEVICE="/dev/sda"
OUTPUT="/evidence/recovered_$(date +%Y%m%d)"
TYPES="jpg,png,gif,pdf,doc,zip,mp4"

# Create output directory
mkdir -p "$OUTPUT"

# Run PhotoRec in command-line mode
sudo photorec /cmd "$DEVICE" /d "$OUTPUT" /e "$TYPES" whole

# Generate report
echo "PhotoRec Recovery Report" > "$OUTPUT/report.txt"
echo "Date: $(date)" >> "$OUTPUT/report.txt"
echo "Device: $DEVICE" >> "$OUTPUT/report.txt"
echo "File types: $TYPES" >> "$OUTPUT/report.txt"
echo "Files recovered: $(find $OUTPUT -type f | wc -l)" >> "$OUTPUT/report.txt"
```

### Analyzing Results

```bash
# Count recovered files by type
find /evidence/recovered/ -type f | sed 's/.*\.//' | sort | uniq -c | sort -rn

# Check file sizes
find /evidence/recovered/ -type f -exec ls -lh {} \;

# Verify file types
find /evidence/recovered/ -type f -exec file {} \;

# Check for corrupted files
find /evidence/recovered/ -name "*.jpg" -exec jpeginfo -c {} \; 2>/dev/null
find /evidence/recovered/ -name "*.pdf" -exec pdfinfo {} \; 2>/dev/null
```

---

## Forensics Workflow Integration

### Phase 1: Evidence Preparation

```bash
# 1. Document original evidence
sudo fdisk -l /dev/sda > /evidence/fdisk_output.txt
sudo smartctl -a /dev/sda > /evidence/smart_output.txt

# 2. Create write blocker
sudo blockdev --setro /dev/sda

# 3. Create bit-for-bit image
sudo dd if=/dev/sda of=/evidence/original.raw bs=4M status=progress

# 4. Verify integrity
md5sum /dev/sda /evidence/original.raw
sha256sum /dev/sda /evidence/original.raw
```

### Phase 2: Recovery with PhotoRec

```bash
# 1. Create working copy
cp /evidence/original.raw /evidence/working.raw

# 2. Create output directory
mkdir -p /evidence/recovered

# 3. Run PhotoRec
sudo photorec /cmd /evidence/working.raw /d /evidence/recovered/ whole

# 4. Document results
cp /evidence/recovered/recup_dir.*/report.txt /evidence/
```

### Phase 3: Post-Recovery Analysis

```bash
# 1. Organize recovered files
mkdir -p /evidence/sorted/{images,videos,documents,archives,audio,other}

find /evidence/recovered/ -name "*.jpg" -exec mv {} /evidence/sorted/images/ \;
find /evidence/recovered/ -name "*.png" -exec mv {} /evidence/sorted/images/ \;
find /evidence/recovered/ -name "*.mp4" -exec mv {} /evidence/sorted/videos/ \;
find /evidence/recovered/ -name "*.pdf" -exec mv {} /evidence/sorted/documents/ \;

# 2. Generate file hashes
find /evidence/recovered/ -type f -exec md5sum {} \; > /evidence/recovered_hashes.txt

# 3. Extract metadata
find /evidence/recovered/ -name "*.jpg" -exec exiftool {} \; > /evidence/metadata.txt

# 4. Create forensic report
cat > /evidence/forensic_report.txt << EOF
PhotoRec Recovery Report
========================
Case: CASE-2026-001
Date: $(date)
Investigator: [Name]
Device: /dev/sda (500GB)
Image: /evidence/original.raw
MD5: $(md5sum /evidence/original.raw | awk '{print $1}')

Recovery Results:
- Total files recovered: $(find /evidence/recovered -type f | wc -l)
- Images: $(find /evidence/sorted/images -type f | wc -l)
- Videos: $(find /evidence/sorted/videos -type f | wc -l)
- Documents: $(find /evidence/sorted/documents -type f | wc -l)
- Archives: $(find /evidence/sorted/archives -type f | wc -l)
- Audio: $(find /evidence/sorted/audio -type f | wc -l)

Tool: PhotoRec 7.2
Command: photorec /cmd /evidence/working.raw /d /evidence/recovered/ whole
EOF
```

### Integration with Other Tools

```bash
# PhotoRec + TestDisk
# Use TestDisk for partition recovery, PhotoRec for file recovery
sudo testdisk /evidence/original.raw
sudo photorec /evidence/original.raw

# PhotoRec + The Sleuth Kit
# Use TSK for filesystem analysis, PhotoRec for carving
fls -r -d /evidence/original.raw > /evidence/tsk_output.txt

# PhotoRec + Scalpel
# Use both for maximum recovery
scalpel /evidence/original.raw -o /evidence/scalpel_output/

# PhotoRec + Autopsy
# Import recovered files into Autopsy for timeline analysis
```

---

## Performance Optimization

### Parallel Processing

```bash
# Split image for parallel recovery
split -b 1G evidence.raw chunk_

# Process each chunk
for chunk in chunk_*; do
    mkdir -p "output_$chunk"
    sudo photorec /cmd "$chunk" /d "output_$chunk/" whole &
done
wait
```

### Selective Scanning

```bash
# Only scan unallocated space (faster)
sudo photorec /cmd /dev/sda /d /output/ free

# Only scan specific file types
sudo photorec /cmd /dev/sda /d /output/ /e "jpg,png" whole
```

### Using Faster Storage

```bash
# Write to SSD
sudo photorec /cmd /dev/sda /d /ssd/output/ whole

# Then copy to long-term storage
cp -r /ssd/output/ /nas/evidence/recovered/
```

### Monitoring Progress

```bash
# Monitor output directory
watch -n 5 'find /output/ -type f | wc -l'

# Check disk I/O
iostat -x 1

# Monitor CPU usage
top -p $(pgrep photorec)
```

---

## Limitations and Considerations

### What PhotoRec Cannot Do

1. **Recover fragmented files**: Cannot reconstruct files stored in non-contiguous blocks
2. **Recover file names**: Original filenames are lost
3. **Recover directory structure**: Directory hierarchy is not preserved
4. **Handle encrypted files**: Cannot decrypt or recover encrypted content
5. **Recover from severely damaged media**: Use ddrescue first
6. **Guarantee accuracy**: May produce false positives

### Known Limitations

- **No filesystem awareness**: Ignores filesystem metadata
- **No filename recovery**: Files are named sequentially (f0000001.jpg, f0000002.jpg)
- **No directory structure**: All files saved to flat directory
- **Large output**: Can produce many duplicate or junk files
- **Memory usage**: High memory consumption on large disks

### When to Use Alternatives

| Situation | Better Tool | Reason |
|-----------|-------------|--------|
| Need filenames | TestDisk | Filesystem-aware |
| Need directory structure | TestDisk | Preserves hierarchy |
| Need less false positives | Scalpel | Better configuration |
| Need resume capability | Safecopy | Better error handling |

---

## Detection and Defense

### For System Administrators

```bash
# Monitor for PhotoRec usage
which photorec
sudo ausearch -k photorec -ts recent

# Check for unauthorized file recovery
find /tmp /var/tmp /home -name "recup_dir.*" -type d

# Check command history
cat ~/.bash_history | grep -i "photorec\|testdisk"

# Monitor disk I/O
sudo iotop
```

### For Forensic Investigators

```bash
# Check if PhotoRec was used as anti-forensics tool
# Look for recovery artifacts
find / -name "recup_dir.*" -type d 2>/dev/null

# Check command history
cat /root/.bash_history | grep -i "photorec"

# Examine system logs
sudo journalctl | grep -i "photorec\|recovery"
```

### Defensive Measures

```bash
# 1. Use full-disk encryption
sudo cryptsetup luksFormat /dev/sda2

# 2. Implement secure deletion
shred -vfz -n 5 sensitive_file.txt

# 3. Enable TRIM on SSDs
sudo fstrim -v /

# 4. Monitor physical access
# 5. Use hardware write blockers for evidence
```

---

## Troubleshooting

### Common Issues and Solutions

#### "No files recovered"

```bash
# Check if device is readable
sudo dd if=/dev/sda of=/dev/null bs=512 count=1

# Try scanning entire disk instead of partition
sudo photorec /dev/sda

# Try different file system type
sudo photorec /cmd /dev/sda /d /output/ /dlog /output/debug.log whole
```

#### "Permission denied"

```bash
# Run with sudo
sudo photorec /dev/sda

# Check device permissions
ls -la /dev/sda
```

#### "Output directory already exists"

```bash
# PhotoRec requires a fresh output directory
rm -rf output/
mkdir output/
sudo photorec /dev/sda
```

#### Recovery is slow

```bash
# Only scan unallocated space
sudo photorec /cmd /dev/sda /d /output/ free

# Only scan specific file types
sudo photorec /cmd /dev/sda /d /output/ /e "jpg,png" whole

# Use faster storage for output
sudo photorec /cmd /dev/sda /d /ssd/output/ whole
```

#### Too many false positives

```bash
# Verify recovered files
find /output/ -type f -exec file {} \;

# Remove junk files
find /output/ -type f -size 0 -delete
find /output/ -type f -size -100c -delete

# Use file command to verify
find /output/ -name "*.jpg" -exec jpeginfo -c {} \;
```

---

## Best Practices

### For Forensic Investigations

1. **Always work on images**: Never run PhotoRec on original evidence
2. **Document everything**: Record commands, outputs, and findings
3. **Use write blockers**: Prevent accidental modification of evidence
4. **Verify hashes**: MD5/SHA256 before and after
5. **Combine with other tools**: Use PhotoRec alongside TestDisk and TSK

### For Data Recovery

1. **Stop using the affected device**: Prevent overwriting deleted data
2. **Create an image first**: Work on copies, not originals
3. **Try both free and whole modes**: Different modes find different files
4. **Check results**: Verify recovered files are valid
5. **Use multiple tools**: Combine PhotoRec with Scalpel and Foremost

### Configuration Tips

```bash
# 1. Start with common file types
# 2. Use free mode first (faster)
# 3. Then use whole mode for comprehensive recovery
# 4. Document all commands and outputs
# 5. Verify recovered files with 'file' command
```

---

## Comparison with Other Tools

### PhotoRec vs Scalpel vs Foremost

| Feature | PhotoRec | Scalpel | Foremost |
|---------|----------|---------|----------|
| File types | 480+ | Many | Many |
| Speed | Medium | Fast | Medium |
| Configuration | Simple | Complex | Medium |
| Filename recovery | No | No | No |
| GUI | Yes (ncurses) | No | No |
| Resume capability | Yes | No | No |
| Active development | Yes | Yes | Yes |
| Kali pre-installed | Yes | Yes | Yes |

### When to Choose PhotoRec

- When you need maximum file type support
- When you want a simple, guided interface
- When you need resume capability
- When working with camera memory cards
- When you want a bootable recovery solution

---

## References

- **Official PhotoRec Documentation**: https://www.cgsecurity.org/wiki/PhotoRec
- **PhotoRec GitHub**: https://github.com/cgsecurity/testdisk
- **Kali Linux Package**: https://www.kali.org/tools/testdisk/
- **File Carving Techniques**: https://www.sans.org/reading-room/forensic-analysis-file-carving-1411
- **Digital Forensics Framework**: https://www.digitalforensicsframework.org/
- **NIST SP 800-86**: Guide to Integrating Forensic Techniques into Incident Response

---

## Quick Reference Card

| Operation | Command |
|-----------|---------|
| Interactive mode | `sudo photorec /dev/sda` |
| Command-line mode | `sudo photorec /cmd /dev/sda /d /output/ whole` |
| Free space only | `sudo photorec /cmd /dev/sda /d /output/ free` |
| Specific file types | `sudo photorec /cmd /dev/sda /d /output/ /e "jpg,png" whole` |
| From disk image | `sudo photorec /cmd image.raw /d /output/ whole` |
| From E01 image | Mount with ewfmount, then photorec |
| View results | `find output/ -type f \| wc -l` |
| Verify files | `find output/ -type f -exec file {} \;` |

---

*Document Version: 1.0*
*Last Updated: 2026*
*Author: Digital Forensics Documentation Project*
*Classification: Educational*
