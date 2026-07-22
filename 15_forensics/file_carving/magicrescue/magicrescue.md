# MagicRescue — File Recovery by Magic Bytes

## Table of Contents

- [Overview](#overview)
- [Installation](#installation)
- [Core Concepts](#core-concepts)
- [Command-Line Reference](#command-line-reference)
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

MagicRescue is a file recovery tool that scans a raw device for file headers (magic bytes) and extracts files based on those signatures. Unlike filesystem-aware tools, MagicRescue works at the byte level, making it effective against formatted, corrupted, or damaged media. Developed by Jonas Jensen, MagicRescue is particularly useful for recovering specific file types from disk images or raw devices.

MagicRescue's approach is similar to other carving tools like Scalpel and Foremost, but it has its own unique features and configuration system. It supports a wide range of file types and provides detailed output about the recovery process.

### Key Features

- **Magic Byte Scanning**: Identifies files by their header signatures
- **Configurable File Types**: Extensive configuration for different file formats
- **Batch Processing**: Can process multiple devices or images
- **Detailed Logging**: Provides comprehensive recovery statistics
- **Cross-Platform**: Works on Linux and other Unix-like systems
- **Low Resource Usage**: Efficient scanning with minimal memory requirements

### When to Use MagicRescue

| Scenario | Use MagicRescue? | Notes |
|----------|-----------------|-------|
| Deleted files on healthy disk | Yes | Good for specific file types |
| Formatted drive recovery | Yes | Works at byte level |
| Corrupted filesystem | Yes | Ignores filesystem metadata |
| Need specific file types | Yes | Highly configurable |
| Damaged disk (bad sectors) | No | Use ddrescue first |
| Large disk images | Yes | Fast scanning |
| Need batch processing | Yes | Good for multiple devices |

---

## Installation

### Kali Linux

```bash
# MagicRescue may need to be installed manually
sudo apt update
sudo apt install magicrescue

# If not in repositories, build from source
```

### Other Linux Distributions

```bash
# Debian/Ubuntu
sudo apt install magicrescue

# From source
git clone https://github.com/jbj/magicrescue.git
cd magicrescue
make
sudo make install
```

### Building from Source

```bash
# Install build dependencies
sudo apt install build-essential git

# Clone repository
git clone https://github.com/jbj/magicrescue.git
cd magicrescue

# Build
make

# Install
sudo make install

# Verify
magicrescue --version
```

---

## Core Concepts

### How MagicRescue Works

MagicRescue operates by:

1. **Scanning raw data** for known file headers (magic bytes)
2. **Extracting files** starting from the identified header
3. **Using file-specific recipes** to determine file boundaries
4. **Saving recovered files** to an output directory

### File Recipes

MagicRescue uses "recipes" to define how to extract each file type:

```
# Recipe format: name, header, footer, max_size
jpeg    ffd8ffe0    ffd9    20000000
png     89504e47    ae426082    30000000
pdf     25504446    2525454f46    50000000
```

### Supported File Types

MagicRescue supports many file types including:

- Images: JPEG, PNG, GIF, TIFF, BMP
- Documents: PDF, DOC, DOCX, ODT
- Archives: ZIP, RAR, 7Z, TAR.GZ
- Audio: MP3, WAV, FLAC, OGG
- Video: MP4, AVI, MOV, MKV
- And many more

---

## Command-Line Reference

### Basic Syntax

```bash
magicrescue [options] <device_or_image>
```

### Options

| Option | Description |
|--------|-------------|
| `-d`, `--output-dir <dir>` | Output directory |
| `-r`, `--recipes <dir>` | Recipe directory |
| `-R`, `--recipe <file>` | Single recipe file |
| `-b`, `--blocksize <size>` | Block size in bytes |
| `-s`, `--start <offset>` | Start offset |
| `-l`, `--length <size>` | Length to scan |
| `-v`, `--verbose` | Verbose output |
| `-q`, `--quiet` | Quiet mode |
| `-h`, `--help` | Show help |

### Device Specification

```bash
# Scan entire device
magicrescue -d /output/ /dev/sda

# Scan disk image
magicrescue -d /output/ /path/to/image.raw

# Scan specific offset and length
magicrescue -d /output/ -s 0 -l 1G /dev/sda
```

---

## Basic Usage

### Simple File Recovery

```bash
# Create output directory
mkdir -p /evidence/recovered

# Scan device for all recipe types
sudo magicrescue -d /evidence/recovered/ /dev/sda

# Check results
ls -la /evidence/recovered/
```

### Recovering Specific File Types

```bash
# Use specific recipes
sudo magicrescue -d /evidence/recovered/ -R /usr/share/magicrescue/recipes/jpeg /dev/sda

# Use multiple recipes
sudo magicrescue -d /evidence/recovered/ -r /usr/share/magicrescue/recipes/ /dev/sda
```

### Recovering from Disk Images

```bash
# Create image first
sudo dd if=/dev/sda of=/evidence/disk.img bs=4M status=progress

# Recover from image
magicrescue -d /evidence/recovered/ /evidence/disk.img
```

---

## Advanced Usage

### Command-Line Scenarios

#### Scenario 1: Recover Only JPEG Files

```bash
# Create custom recipe
cat > /tmp/jpeg.recipe << 'EOF'
jpeg    ffd8ffe0    ffd9    20000000
EOF

# Use custom recipe
sudo magicrescue -d /evidence/jpegs/ -R /tmp/jpeg.recipe /dev/sda
```

#### Scenario 2: Batch Processing

```bash
#!/bin/bash
# Process multiple devices

DEVICES=("/dev/sda" "/dev/sdb" "/dev/sdc")
OUTPUT="/evidence/batch_recovery"

for device in "${DEVICES[@]}"; do
    echo "Processing $device..."
    sudo magicrescue -d "$OUTPUT/$(basename $device)" "$device"
done
```

#### Scenario 3: Recover from Specific Offset

```bash
# Recover from middle of disk
sudo magicrescue -d /evidence/middle/ -s 500000000 -l 100000000 /dev/sda
```

#### Scenario 4: Forensic Recovery with Documentation

```bash
#!/bin/bash
# Documented forensic recovery

DEVICE="/dev/sda"
CASE_ID="CASE-2026-001"
OUTPUT="/cases/$CASE_ID/evidence/recovered"
LOG="/cases/$CASE_ID/logs/magicrescue_$(date +%Y%m%d_%H%M%S).log"

# Create directories
mkdir -p "$OUTPUT" "$(dirname $LOG)"

# Document start
echo "Recovery started: $(date)" | tee "$LOG"
echo "Device: $DEVICE" | tee -a "$LOG"

# Run recovery
sudo magicrescue -d "$OUTPUT" "$DEVICE" -v 2>&1 | tee -a "$LOG"

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

### Phase 2: Recovery with MagicRescue

```bash
# 1. Create working copy
cp /evidence/original.raw /evidence/working.raw

# 2. Create output directory
mkdir -p /evidence/recovered

# 3. Run MagicRescue
sudo magicrescue -d /evidence/recovered/ /evidence/working.raw

# 4. Document results
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
echo "MagicRescue Recovery Report" > /evidence/report.txt
echo "Case: CASE-2026-001" >> /evidence/report.txt
echo "Date: $(date)" >> /evidence/report.txt
echo "Device: /dev/sda" >> /evidence/report.txt
echo "Files recovered: $(find /evidence/recovered -type f | wc -l)" >> /evidence/report.txt
```

### Integration with Other Tools

```bash
# MagicRescue + The Sleuth Kit
# Use TSK for filesystem analysis
fls -r -d /evidence/original.raw > /evidence/tsk_output.txt

# MagicRescue + Scalpel/PhotoRec
# Use other carvers for additional recovery
scalpel /evidence/original.raw -o /evidence/scalpel_output/

# MagicRescue + YARA
# Scan recovered files for malicious content
yara -r /path/to/rules/ /evidence/recovered/

# MagicRescue + ExifTool
# Extract metadata from recovered images
find /evidence/recovered/ -name "*.jpg" -exec exiftool {} \;
```

---

## Performance Optimization

### Parallel Processing

```bash
# Split image for parallel processing
split -b 1G evidence.raw chunk_

# Process each chunk
for chunk in chunk_*; do
    mkdir -p "output_$chunk"
    magicrescue -d "output_$chunk/" "$chunk" &
done
wait
```

### Selective Scanning

```bash
# Only scan specific file types
sudo magicrescue -d /output/ -R /path/to/jpeg.recipe /dev/sda

# Scan specific sector range
sudo magicrescue -d /output/ -s 0 -l 1G /dev/sda
```

### Using Faster Storage

```bash
# Write to SSD for faster I/O
magicrescue -d /ssd/output/ /dev/sda

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
top -p $(pgrep magicrescue)
```

---

## Limitations and Considerations

### What MagicRescue Cannot Do

1. **Recover fragmented files**: Cannot reconstruct files stored in non-contiguous blocks
2. **Handle encrypted files**: Cannot decrypt or recover encrypted content
3. **Recover from severely damaged media**: Use ddrescue first
4. **Guarantee accuracy**: May produce false positives
5. **Recover overwritten data**: Cannot recover data that has been overwritten

### Known Limitations

- **No filesystem awareness**: Works at raw block level only
- **Limited resume capability**: Must restart if interrupted
- **No GUI**: Command-line only
- **Limited documentation**: Fewer resources than mainstream tools
- **Recipe-dependent**: Only recovers file types with defined recipes

### When to Use Alternatives

| Situation | Better Tool | Reason |
|-----------|-------------|--------|
| Need many file types | Scalpel | More default types |
| Need resume capability | PhotoRec | Better resume |
| Need filesystem awareness | TestDisk | Filesystem-aware |
| Need maximum recovery | Multiple tools | Combine approaches |

---

## Detection and Defense

### For System Administrators

```bash
# Monitor for MagicRescue usage
which magicrescue
sudo ausearch -k magicrescue -ts recent

# Check for unauthorized file recovery
find /tmp /var/tmp /home -name "*.jpg" -o -name "*.pdf" -mtime -1

# Check command history
cat ~/.bash_history | grep -i "magicrescue\|recover"

# Monitor disk I/O
sudo iotop
```

### For Forensic Investigators

```bash
# Check if MagicRescue was used as anti-forensics tool
# Look for recovery artifacts
find / -name "recovered" -type d 2>/dev/null

# Check command history
cat /root/.bash_history | grep -i "magicrescue"

# Examine system logs
sudo journalctl | grep -i "magicrescue\|recover"
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

# Try different recipes
ls /usr/share/magicrescue/recipes/

# Create custom recipe
cat > /tmp/custom.recipe << 'EOF'
jpeg    ffd8ffe0    ffd9    20000000
png     89504e47    ae426082    30000000
EOF
sudo magicrescue -d /output/ -R /tmp/custom.recipe /dev/sda
```

#### "Permission denied"

```bash
# Run with sudo
sudo magicrescue -d /output/ /dev/sda

# Check device permissions
ls -la /dev/sda
```

#### "Recipe not found"

```bash
# Check recipe directory
ls /usr/share/magicrescue/recipes/

# Use absolute path to recipe
sudo magicrescue -d /output/ -R /usr/share/magicrescue/recipes/jpeg /dev/sda
```

#### Recovery is slow

```bash
# Scan specific sector range
sudo magicrescue -d /output/ -s 0 -l 1G /dev/sda

# Use disk image instead of live device
sudo dd if=/dev/sda of=/tmp/image.raw bs=4M
magicrescue -d /output/ /tmp/image.raw
```

---

## Best Practices

### For Forensic Investigations

1. **Always work on images**: Never run MagicRescue on original evidence
2. **Document everything**: Record commands, outputs, and findings
3. **Use write blockers**: Prevent accidental modification of evidence
4. **Verify hashes**: MD5/SHA256 before and after
5. **Combine with other tools**: Use MagicRescue alongside Scalpel and PhotoRec

### For Data Recovery

1. **Stop using the affected device**: Prevent overwriting deleted data
2. **Create an image first**: Work on copies, not originals
3. **Try different recipes**: Different recipes may yield different results
4. **Check results**: Verify recovered files are valid
5. **Have a backup plan**: If MagicRescue fails, try other tools

### Configuration Tips

```bash
# 1. Start with common recipes
# 2. Create custom recipes for specific needs
# 3. Use verbose mode to monitor progress
# 4. Document all commands and outputs
# 5. Verify recovered files with 'file' command
```

---

## Comparison with Other Tools

### MagicRescue vs Scalpel vs PhotoRec

| Feature | MagicRescue | Scalpel | PhotoRec |
|---------|-------------|---------|----------|
| Configuration | Recipe-based | Config file | Simple |
| Speed | Fast | Fast | Medium |
| File types | Many | Many | Many |
| Resume | Limited | No | Yes |
| GUI | No | No | Yes |
| Active development | Low | High | High |

### When to Choose MagicRescue

- When you need recipe-based configuration
- When working with batch processing
- When you need detailed logging
- When other tools fail due to configuration issues
- When you need a lightweight tool

---

## References

- **MagicRescue Documentation**: https://www.chiark.greenend.org.uk/~sgtatham/magicrescue/
- **MagicRescue GitHub**: https://github.com/jbj/magicrescue
- **Kali Linux Forensics**: https://www.kali.org/tools/magicrescue/
- **File Carving Techniques**: https://www.sans.org/reading-room/forensic-analysis-file-carving-1411
- **Digital Forensics Framework**: https://www.digitalforensicsframework.org/
- **NIST SP 800-86**: Guide to Integrating Forensic Techniques into Incident Response

---

## Quick Reference Card

| Operation | Command |
|-----------|---------|
| Basic recovery | `sudo magicrescue -d /output/ /dev/sda` |
| With recipes | `sudo magicrescue -d /output/ -r /recipes/ /dev/sda` |
| Single recipe | `sudo magicrescue -d /output/ -R jpeg.recipe /dev/sda` |
| From image | `magicrescue -d /output/ image.raw` |
| Specific offset | `sudo magicrescue -d /output/ -s 0 -l 1G /dev/sda` |
| Verbose mode | `sudo magicrescue -d /output/ -v /dev/sda` |
| View results | `ls -la /output/` |
| Count files | `find /output/ -type f \| wc -l` |

---

*Document Version: 1.0*
*Last Updated: 2026*
*Author: Digital Forensics Documentation Project*
*Classification: Educational*
