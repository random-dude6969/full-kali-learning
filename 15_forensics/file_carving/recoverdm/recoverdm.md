# RecoverDM — Recover Deleted Files from Hard Disk

## Table of Contents

- [Overview](#overview)
- [Installation](#installation)
- [Core Concepts](#core-concepts)
- [Command-Line Reference](#command-line-reference)
- [Basic Usage](#basic-usage)
- [Advanced Usage](#advanced-usage)
- [Forensics Workflow Integration](#forensics-workflow-integration)
- [Limitations and Considerations](#limitations-and-considerations)
- [Detection and Defense](#detection-and-defense)
- [Troubleshooting](#troubleshooting)
- [Best Practices](#best-practices)
- [Comparison with Other Tools](#comparison-with-other-tools)
- [References](#references)

---

## Overview

RecoverDM is a command-line utility designed to recover deleted files from hard disk drives and other block devices. Unlike filesystem-aware tools that rely on metadata, RecoverDM works at the raw block level, scanning disk surfaces for file signatures to identify and extract deleted files. It is particularly useful when filesystem metadata has been damaged or overwritten.

Developed by Gnuckies, RecoverDM focuses on simplicity and efficiency. It supports multiple file types and can scan entire disks or specific partitions for deleted content. The tool is lightweight and fast, making it suitable for quick recovery operations in digital forensics investigations.

### Key Features

- **Raw Block Scanning**: Scans disk at the block level for file signatures
- **Multiple File Types**: Supports JPEG, PNG, GIF, PDF, ZIP, and other common formats
- **Simple Interface**: Straightforward command-line operation
- **Fast Scanning**: Optimized for quick disk scanning
- **Cross-Platform**: Works on Linux and other Unix-like systems
- **Low Resource Usage**: Minimal memory and CPU requirements

### When to Use RecoverDM

| Scenario | Use RecoverDM? | Notes |
|----------|---------------|-------|
| Deleted files on healthy disk | Yes | Good for quick recovery |
| Formatted drive recovery | Yes | Works at block level |
| Corrupted filesystem | Yes | Ignores filesystem metadata |
| Need specific file types | Yes | Configurable file types |
| Damaged disk (bad sectors) | No | Use ddrescue first |
| Large disk images | Yes | Fast scanning |
| Need carving statistics | Yes | Provides detailed output |

---

## Installation

### Kali Linux

```bash
# RecoverDM may need to be installed manually
sudo apt update
sudo apt install recoverdm

# If not in repositories, build from source
```

### Other Linux Distributions

```bash
# Debian/Ubuntu
sudo apt install recoverdm

# From source
git clone https://github.com/Gnuckies/recoverdm.git
cd recoverdm
make
sudo make install
```

### Building from Source

```bash
# Install build dependencies
sudo apt install build-essential git

# Clone repository
git clone https://github.com/Gnuckies/recoverdm.git
cd recoverdm

# Build
make

# Install (may need sudo)
sudo cp recoverdm /usr/local/bin/

# Verify
recoverdm --version
```

---

## Core Concepts

### How RecoverDM Works

RecoverDM operates by:

1. **Reading raw blocks** from the disk or image file
2. **Scanning for file headers** (magic bytes) that identify file types
3. **Extracting data** from the header to the next header or end of scan area
4. **Saving recovered files** to an output directory

### File Signatures

RecoverDM identifies files by their header signatures:

| File Type | Header (Hex) | Description |
|-----------|--------------|-------------|
| JPEG | `FF D8 FF E0` | JPEG image |
| PNG | `89 50 4E 47` | PNG image |
| GIF | `47 49 46 38` | GIF image |
| PDF | `25 50 44 46` | PDF document |
| ZIP | `50 4B 03 04` | ZIP archive |
| RAR | `52 61 72 21` | RAR archive |
| MP3 | `49 44 33` | MP3 audio |
| MP4 | `00 00 00 18` | MP4 video |

### Recovery Approach

```
Disk Surface
[Sector 0] [Sector 1] ... [Sector N]
    |           |              |
    v           v              v
[Scan for headers] → [Extract between headers] → [Save files]
```

---

## Command-Line Reference

### Basic Syntax

```bash
recoverdm [options] <device_or_image>
```

### Options

| Option | Description |
|--------|-------------|
| `-d`, `--device <device>` | Device to scan |
| `-o`, `--output <dir>` | Output directory |
| `-t`, `--type <type>` | File type to recover |
| `-s`, `--start <sector>` | Start sector |
| `-e`, `--end <sector>` | End sector |
| `-b`, `--blocksize <size>` | Block size in bytes |
| `-v`, `--verbose` | Verbose output |
| `-h`, `--help` | Show help |
| `-V`, `--version` | Show version |

### Device Specification

```bash
# Scan entire disk
recoverdm -d /dev/sda -o /output/

# Scan specific partition
recoverdm -d /dev/sda1 -o /output/

# Scan disk image
recoverdm -d image.raw -o /output/

# Scan specific sector range
recoverdm -d /dev/sda -s 0 -e 1000000 -o /output/
```

---

## Basic Usage

### Simple File Recovery

```bash
# Create output directory
mkdir -p /evidence/recovered

# Scan entire disk for all supported file types
sudo recoverdm -d /dev/sda -o /evidence/recovered/

# Check results
ls -la /evidence/recovered/
```

### Recovering Specific File Types

```bash
# Recover only JPEG files
sudo recoverdm -d /dev/sda -t jpg -o /evidence/jpegs/

# Recover only PDF files
sudo recoverdm -d /dev/sda -t pdf -o /evidence/pdfs/

# Recover only ZIP files
sudo recoverdm -d /dev/sda -t zip -o /evidence/zips/
```

### Scanning a Disk Image

```bash
# Create image first
sudo dd if=/dev/sda of=/evidence/disk.img bs=4M status=progress

# Scan the image
recoverdm -d /evidence/disk.img -o /evidence/recovered/

# Verify recovered files
find /evidence/recovered/ -type f -exec file {} \;
```

---

## Advanced Usage

### Command-Line Scenarios

#### Scenario 1: Recover from Formatted Drive

```bash
# After formatting, filesystem metadata is destroyed
# RecoverDM can still find files by scanning for headers

sudo recoverdm -d /dev/sda1 -o /evidence/formatted_recovery/

# Check what was recovered
find /evidence/formatted_recovery/ -type f | head -20
```

#### Scenario 2: Recover Specific Sector Range

```bash
# If you know approximately where deleted files were
# Scan only that region

sudo recoverdm -d /dev/sda -s 1000000 -e 2000000 -o /evidence/specific/
```

#### Scenario 3: Batch Recovery

```bash
#!/bin/bash
# Recover multiple file types in batch

DEVICE="/dev/sda"
OUTPUT="/evidence/batch_recovery"
TYPES=("jpg" "png" "gif" "pdf" "zip")

for type in "${TYPES[@]}"; do
    echo "Recovering $type files..."
    sudo recoverdm -d "$DEVICE" -t "$type" -o "$OUTPUT/$type/"
done

echo "Recovery complete"
find "$OUTPUT" -type f | wc -l
```

#### Scenario 4: Forensic Recovery with Documentation

```bash
#!/bin/bash
# Documented forensic recovery

DEVICE="/dev/sda"
CASE_ID="CASE-2026-001"
OUTPUT="/cases/$CASE_ID/evidence/recovered"
LOG="/cases/$CASE_ID/logs/recoverdm_$(date +%Y%m%d_%H%M%S).log"

# Create directories
mkdir -p "$OUTPUT" "$(dirname $LOG)"

# Document start
echo "Recovery started: $(date)" | tee "$LOG"
echo "Device: $DEVICE" | tee -a "$LOG"

# Run recovery
sudo recoverdm -d "$DEVICE" -o "$OUTPUT" -v 2>&1 | tee -a "$LOG"

# Document completion
echo "Recovery completed: $(date)" | tee -a "$LOG"
echo "Files recovered: $(find $OUTPUT -type f | wc -l)" | tee -a "$LOG"
```

### Analyzing Results

```bash
# Count recovered files by type
find /evidence/recovered/ -type f -exec file {} \; | \
    awk -F: '{print $2}' | sort | uniq -c | sort -rn

# Check file sizes
find /evidence/recovered/ -type f -exec ls -lh {} \; | \
    awk '{print $5, $9}' | sort -h

# Verify file integrity
find /evidence/recovered/ -name "*.jpg" -exec jpeginfo -c {} \;
find /evidence/recovered/ -name "*.pdf" -exec pdfinfo {} \;
```

---

## Forensics Workflow Integration

### Phase 1: Evidence Preparation

```bash
# 1. Document original state
sudo fdisk -l /dev/sda > /evidence/fdisk_output.txt
sudo smartctl -a /dev/sda > /evidence/smart_output.txt

# 2. Create write blocker
sudo blockdev --setro /dev/sda

# 3. Create disk image
sudo dd if=/dev/sda of=/evidence/original.raw bs=4M status=progress

# 4. Verify hash
md5sum /dev/sda /evidence/original.raw
```

### Phase 2: Recovery with RecoverDM

```bash
# 1. Create working copy
cp /evidence/original.raw /evidence/working.raw

# 2. Run RecoverDM on working copy
sudo recoverdm -d /evidence/working.raw -o /evidence/recovered/ -v

# 3. Document results
find /evidence/recovered/ -type f > /evidence/recovered_files.txt
```

### Phase 3: Post-Recovery Analysis

```bash
# 1. Sort recovered files
mkdir -p /evidence/sorted/{images,documents,archives,other}
find /evidence/recovered/ -name "*.jpg" -exec mv {} /evidence/sorted/images/ \;
find /evidence/recovered/ -name "*.pdf" -exec mv {} /evidence/sorted/documents/ \;

# 2. Generate file hashes
find /evidence/recovered/ -type f -exec md5sum {} \; > /evidence/recovered_hashes.txt

# 3. Create forensic report
echo "RecoverDM Recovery Report" > /evidence/report.txt
echo "Case: CASE-2026-001" >> /evidence/report.txt
echo "Date: $(date)" >> /evidence/report.txt
echo "Device: /dev/sda" >> /evidence/report.txt
echo "Files recovered: $(find /evidence/recovered -type f | wc -l)" >> /evidence/report.txt
```

### Integration with Other Tools

```bash
# RecoverDM + The Sleuth Kit
# Use TSK for filesystem-aware analysis
fls -r -d /evidence/original.raw > /evidence/tsk_output.txt

# RecoverDM + Scalpel/PhotoRec
# Use other carvers for additional recovery
scalpel /evidence/original.raw -o /evidence/scalpel_output/

# RecoverDM + YARA
# Scan recovered files for malicious content
yara -r /path/to/rules/ /evidence/recovered/

# RecoverDM + ExifTool
# Extract metadata from recovered images
find /evidence/recovered/ -name "*.jpg" -exec exiftool {} \;
```

---

## Limitations and Considerations

### What RecoverDM Cannot Do

1. **Recover fragmented files**: Cannot reconstruct files stored in non-contiguous blocks
2. **Handle encrypted files**: Cannot decrypt or recover encrypted content
3. **Recover from severely damaged media**: Use ddrescue first
4. **Guarantee accuracy**: May produce false positives
5. **Recover overwritten data**: Cannot recover data that has been overwritten

### Known Limitations

- **No filesystem awareness**: Works at raw block level only
- **Limited file type support**: Fewer types than Scalpel or PhotoRec
- **No resume capability**: Must restart if interrupted
- **No GUI**: Command-line only
- **Limited documentation**: Fewer resources than mainstream tools

### When to Use Alternatives

| Situation | Better Tool | Reason |
|-----------|-------------|--------|
| Many file types needed | Scalpel | More file types |
| Need resume capability | Safecopy | Better error handling |
| Need GUI | PhotoRec | Has GUI option |
| Need filesystem awareness | TestDisk | Filesystem-aware |
| Need maximum recovery | Multiple tools | Combine approaches |

---

## Detection and Defense

### For System Administrators

```bash
# Monitor for RecoverDM usage
which recoverdm
sudo ausearch -k recoverdm -ts recent

# Check for unauthorized file recovery
find /tmp /var/tmp /home -name "*.jpg" -o -name "*.pdf" -mtime -1

# Check command history
cat ~/.bash_history | grep -i "recoverdm\|recover"

# Monitor disk I/O
sudo iotop
```

### For Forensic Investigators

```bash
# Check if RecoverDM was used as anti-forensics tool
# Look for recovery artifacts
find / -name "recovered" -type d 2>/dev/null

# Check command history
cat /root/.bash_history | grep -i "recoverdm"

# Examine system logs
sudo journalctl | grep -i "recoverdm\|recover"
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

#### "Permission denied"

```bash
# Run with sudo
sudo recoverdm -d /dev/sda -o /evidence/recovered/

# Check device permissions
ls -la /dev/sda
```

#### "No files recovered"

```bash
# Check if device is readable
sudo dd if=/dev/sda of=/dev/null bs=512 count=1

# Try different file types
sudo recoverdm -d /dev/sda -t jpg -o /evidence/

# Scan specific sectors
sudo recoverdm -d /dev/sda -s 0 -e 100000 -o /evidence/
```

#### "Device not found"

```bash
# List available devices
lsblk
sudo fdisk -l

# Check if device is mounted
mount | grep /dev/sda
```

#### Recovery is slow

```bash
# Scan specific sectors instead of entire disk
sudo recoverdm -d /dev/sda -s 0 -e 1000000 -o /evidence/

# Use disk image instead of live device
sudo dd if=/dev/sda of=/tmp/image.raw bs=4M
recoverdm -d /tmp/image.raw -o /evidence/
```

---

## Best Practices

### For Forensic Investigations

1. **Always work on images**: Never run RecoverDM on original evidence
2. **Document everything**: Record commands, outputs, and findings
3. **Use write blockers**: Prevent accidental modification of evidence
4. **Verify hashes**: MD5/SHA256 before and after
5. **Combine with other tools**: Use multiple recovery tools for best results

### For Data Recovery

1. **Stop using the affected device**: Prevent overwriting deleted data
2. **Create an image first**: Work on copies, not originals
3. **Try multiple file types**: Different types may yield different results
4. **Check results**: Verify recovered files are valid
5. **Have a backup plan**: If RecoverDM fails, try other tools

### Configuration Tips

```bash
# 1. Start with common file types
# 2. Scan specific sectors if you know where files were
# 3. Use verbose mode to monitor progress
# 4. Document all commands and outputs
# 5. Verify recovered files with 'file' command
```

---

## Comparison with Other Tools

### RecoverDM vs Scalpel vs PhotoRec

| Feature | RecoverDM | Scalpel | PhotoRec |
|---------|-----------|---------|----------|
| File types | Few | Many | Many |
| Speed | Fast | Fast | Medium |
| Configuration | Simple | Complex | Medium |
| Resume | No | No | Yes |
| GUI | No | No | Yes |
| Active development | Low | High | High |

### When to Choose RecoverDM

- When you need quick, simple recovery
- When you're recovering common file types
- When you want minimal configuration
- When working with limited system resources
- As a first-pass recovery tool before more complex tools

---

## References

- **RecoverDM GitHub**: https://github.com/Gnuckies/recoverdm
- **Kali Linux Forensics**: https://www.kali.org/tools/recoverdm/
- **Data Recovery Guide**: https://www.sans.org/reading-room/data-recovery-techniques-1543
- **Digital Forensics Framework**: https://www.digitalforensicsframework.org/
- **File Carving Techniques**: https://www.sans.org/reading-room/forensic-analysis-file-carving-1411

---

## Quick Reference Card

| Operation | Command |
|-----------|---------|
| Basic recovery | `sudo recoverdm -d /dev/sda -o output/` |
| JPEG only | `sudo recoverdm -d /dev/sda -t jpg -o output/` |
| From image | `recoverdm -d image.raw -o output/` |
| Specific sectors | `sudo recoverdm -d /dev/sda -s 0 -e 1000000 -o output/` |
| Verbose mode | `sudo recoverdm -d /dev/sda -o output/ -v` |
| View results | `ls -la output/` |
| Count files | `find output/ -type f \| wc -l` |
| Verify types | `find output/ -type f -exec file {} \;` |

---

*Document Version: 1.0*
*Last Updated: 2026*
*Author: Digital Forensics Documentation Project*
*Classification: Educational*
