# Foremost — Forensic File Recovery Tool

## Table of Contents

- [Overview](#overview)
- [Installation](#installation)
- [Core Concepts](#core-concepts)
- [Command-Line Reference](#command-line-reference)
- [Configuration File](#configuration-file)
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

Foremost is a console program to recover files based on their headers, footers, and internal data structures. Originally developed by the United States Air Force Office of Special Investigations (AFOSI), Foremost is now maintained by the opensourceforensics community. It is one of the most widely used file carving tools in digital forensics due to its simplicity, reliability, and effectiveness.

Foremost works by scanning raw disk data for known file signatures and extracting files based on those signatures. It supports a wide range of file types and provides detailed output about the recovery process. Foremost is particularly valued in forensic investigations for its straightforward approach and documented methodology.

### Key Features

- **Header/Footer Based Carving**: Recovers files by identifying known file signatures
- **Multi-Format Support**: Can carve dozens of file types simultaneously
- **Configurable**: Extensive configuration file for custom file types
- **Simple Interface**: Easy-to-use command-line tool
- **Cross-Platform**: Works on Linux, macOS, Windows, and other Unix systems
- **Forensic Standard**: Widely accepted in forensic communities

### When to Use Foremost

| Scenario | Use Foremost? | Notes |
|----------|--------------|-------|
| Deleted files on healthy disk | Yes | Good for quick recovery |
| Formatted drive recovery | Yes | Works at byte level |
| Corrupted filesystem | Yes | Ignores filesystem metadata |
| Need specific file types | Yes | Configurable file types |
| Damaged disk (bad sectors) | No | Use ddrescue first |
| Large disk images | Yes | Fast scanning |
| Need forensic documentation | Yes | Well-documented tool |

---

## Installation

### Kali Linux

```bash
# Foremost is pre-installed on Kali Linux
sudo apt update
sudo apt install foremost

# Verify installation
foremost -v
```

### Other Linux Distributions

```bash
# Debian/Ubuntu
sudo apt install foremost

# Fedora/RHEL
sudo dnf install foremost

# Arch Linux
sudo pacman -S foremost

# From source
git clone https://github.com/korczis/foremost.git
cd foremost
make
sudo make install
```

### Building from Source

```bash
# Install build dependencies
sudo apt install build-essential git

# Clone repository
git clone https://github.com/korczis/foremost.git
cd foremost

# Build
make

# Install
sudo make install

# Verify
foremost -v
```

---

## Core Concepts

### How Foremost Works

Foremost operates by:

1. **Scanning raw data** for known file headers (magic bytes)
2. **Extracting files** from the header to the next header or end of scan area
3. **Saving recovered files** to an output directory organized by file type
4. **Logging results** with detailed statistics

### File Signatures

Foremost identifies files by their header and footer signatures:

| File Type | Header (Hex) | Footer (Hex) |
|-----------|--------------|--------------|
| JPEG | `FF D8 FF E0` | `FF D9` |
| PNG | `89 50 4E 47` | `AE 42 60 82` |
| PDF | `25 50 44 46` | `25 25 45 4F 46` |
| ZIP | `50 4B 03 04` | `50 4B 05 06` |
| GIF | `47 49 46 38` | `3B` |
| MP4 | `00 00 00 18` | Varies |

### Carving Modes

Foremost supports different carving strategies:

- **Normal mode**: Standard carving with header and footer
- **Audit mode**: Only logs what would be carved (no extraction)
- **Verbose mode**: Detailed output during carving

---

## Command-Line Reference

### Basic Syntax

```bash
foremost [options] <image_file>
```

### Options

| Option | Description |
|--------|-------------|
| `-i <image>` | Input file (disk image or device) |
| `-o <output>` | Output directory |
| `-c <config>` | Configuration file |
| `-t <types>` | File types to recover |
| `-b` | Carve contiguous blocks only |
| `-d` | Enable debug mode |
| `-f` | Carve forward from header |
| `-h` | Show help message |
| `-l` | Show license |
| `-m` | Process multiple images |
| `-n` | Don't add extensions |
| `-p` | Preallocate disk space |
| `-q` | Quiet mode |
| `-s` | Skip header/footer validation |
| `-t` | Carve tail of files |
| `-v` | Verbose output |
| `-V` | Show version |

### Input Sources

```bash
# From disk image
foremost -i evidence.raw -o output/

# From device
sudo foremost -i /dev/sda -o output/

# From E01 image (need to mount first)
sudo ewfmount evidence.E01 /mnt/ewf
foremost -i /mnt/ewf/ewf1 -o output/
```

---

## Configuration File

### Default Configuration Location

```bash
# System-wide config
/etc/foremost.conf

# User config (copy and modify)
cp /etc/foremost.conf ./my_foremost.conf
```

### Configuration File Format

```bash
# Foremost configuration file
# Format: extension  case_sensitive  header  footer  max_size

# Graphics
jpg       y       0xffd8ff        0xffd9          20000000
gif       y       0x47494638      0x3b            30000000
png       y       0x89504e47      0xae426082      30000000
bmp       y       0x424d          0xffffffff      0
tiff      y       0x49492a00      0x00000000      0
psd       y       0x38425053      0x00            0

# Documents
pdf       y       0x25504446      0x2525454f46    50000000
doc       y       0xd0cf11e0      0x00000000      0
docx      y       0x504b0304      0x504b0506      50000000
xlsx      y       0x504b0304      0x504b0506      50000000
pptx      y       0x504b0304      0x504b0506      50000000
odt       y       0x504b0304      0x504b0506      0

# Archives
zip       y       0x504b0304      0x504b0506      100000000
rar       y       0x52617221      0x00            0
7z        y       0x377abcaf      0x00            0
tar       y       0x75737461      0x00000000      0
gz        y       0x1f8b          0x00            0

# Audio
mp3       y       0x494433        0x00000000      0
wav       y       0x52494646      0x00000000      0
ogg       y       0x4f676753      0x00000000      0
flac      y       0x664c6143      0x00000000      0

# Video
mp4       y       0x00000018      0x00000000      0
avi       y       0x52494646      0x00000000      0
wmv       y       0x3026b275      0x00000000      0
mov       y       0x00000014      0x00000000      0

# Executables
elf       y       0x7f454c46      0x00000000      0
exe       y       0x4d5a          0x00000000      0
dll       y       0x4d5a          0x00000000      0

# Email
pst       y       0x2142444e      0x00000000      0
dbx       y       0xcfad12fe      0x00000000      0

# Databases
mdb       y       0x00010000      0x00000000      0
sqlite    y       0x53514c69      0x00000000      0
```

### Creating Custom File Types

```bash
# Add a custom file type to the configuration
# Format: extension  case_sensitive  header  footer  max_size

# Custom XML file
xml       y       0x3c3f786d6c    0x3c2f786d6c3e  0

# Custom log format
mylog     y       0x4c4f473a      0x0a0a0a        0
```

---

## Basic Usage

### Simple File Recovery

```bash
# Create output directory
mkdir output

# Recover all file types from disk image
foremost -i evidence.raw -o output/

# Recover from device
sudo foremost -i /dev/sda -o output/
```

### Recovering Specific File Types

```bash
# Recover only JPEG and PNG files
foremost -i evidence.raw -o output/ -t jpg,png

# Recover only documents
foremost -i evidence.raw -o output/ -t pdf,doc,docx
```

### Recovering from Disk Images

```bash
# From raw disk image
foremost -i evidence.raw -o output/

# From E01 image
sudo ewfmount evidence.E01 /mnt/ewf
foremost -i /mnt/ewf/ewf1 -o output/
sudo umount /mnt/ewf
```

---

## Advanced Usage

### Command-Line Scenarios

#### Scenario 1: Maximum Recovery

```bash
# Recover all file types with verbose output
foremost -i evidence.raw -o output/ -v

# Check results
cat output/audit.txt
```

#### Scenario 2: Selective Recovery

```bash
# Only recover specific file types
foremost -i evidence.raw -o output/ -t jpg,png,pdf,zip

# Skip header validation
foremost -i evidence.raw -o output/ -s
```

#### Scenario 3: Audit Mode

```bash
# See what would be recovered without extracting
foremost -i evidence.raw -o output/ -T

# Review audit log
cat output/audit.txt
```

#### Scenario 4: Forensic Recovery with Documentation

```bash
#!/bin/bash
# Documented forensic recovery

IMAGE="/evidence/evidence.raw"
OUTPUT="/evidence/recovered_$(date +%Y%m%d)"
LOG="/evidence/foremost_$(date +%Y%m%d_%H%M%S).log"

# Create output directory
mkdir -p "$OUTPUT"

# Run Foremost with documentation
echo "Foremost Recovery Started: $(date)" | tee "$LOG"
echo "Image: $IMAGE" | tee -a "$LOG"

foremost -i "$IMAGE" -o "$OUTPUT" -v 2>&1 | tee -a "$LOG"

echo "Recovery Completed: $(date)" | tee -a "$LOG"
echo "Files recovered: $(find $OUTPUT -type f | wc -l)" | tee -a "$LOG"

# Copy audit log
cp "$OUTPUT/audit.txt" /evidence/
```

### Analyzing Results

```bash
# View audit log
cat output/audit.txt

# Count recovered files by type
find output/ -type f | sed 's/.*\.//' | sort | uniq -c | sort -rn

# Check file sizes
find output/ -type f -exec ls -lh {} \;

# Verify file types
find output/ -type f -exec file {} \;
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

### Phase 2: Recovery with Foremost

```bash
# 1. Create working copy
cp /evidence/original.raw /evidence/working.raw

# 2. Create output directory
mkdir -p /evidence/recovered

# 3. Run Foremost
foremost -i /evidence/working.raw -o /evidence/recovered/ -v

# 4. Document results
cp /evidence/recovered/audit.txt /evidence/
```

### Phase 3: Post-Recovery Analysis

```bash
# 1. Organize recovered files
mkdir -p /evidence/sorted/{images,documents,archives,other}

find /evidence/recovered/ -name "*.jpg" -exec mv {} /evidence/sorted/images/ \;
find /evidence/recovered/ -name "*.png" -exec mv {} /evidence/sorted/images/ \;
find /evidence/recovered/ -name "*.pdf" -exec mv {} /evidence/sorted/documents/ \;
find /evidence/recovered/ -name "*.zip" -exec mv {} /evidence/sorted/archives/ \;

# 2. Generate file hashes
find /evidence/recovered/ -type f -exec md5sum {} \; > /evidence/recovered_hashes.txt

# 3. Extract metadata
find /evidence/recovered/ -name "*.jpg" -exec exiftool {} \; > /evidence/metadata.txt

# 4. Create forensic report
cat > /evidence/forensic_report.txt << EOF
Foremost Recovery Report
========================
Case: CASE-2026-001
Date: $(date)
Investigator: [Name]
Image: /evidence/original.raw
MD5: $(md5sum /evidence/original.raw | awk '{print $1}')

Recovery Results:
- Total files recovered: $(find /evidence/recovered -type f | wc -l)
- Images: $(find /evidence/sorted/images -type f | wc -l)
- Documents: $(find /evidence/sorted/documents -type f | wc -l)
- Archives: $(find /evidence/sorted/archives -type f | wc -l)

Tool: Foremost $(foremost -v | head -1)
Command: foremost -i /evidence/working.raw -o /evidence/recovered/ -v
EOF
```

### Integration with Other Tools

```bash
# Foremost + TestDisk
# Use TestDisk for partition recovery, Foremost for file recovery
sudo testdisk /evidence/original.raw
foremost -i /evidence/original.raw -o /evidence/foremost_output/

# Foremost + The Sleuth Kit
# Use TSK for filesystem analysis
fls -r -d /evidence/original.raw > /evidence/tsk_output.txt

# Foremost + Scalpel/PhotoRec
# Use multiple carvers for maximum recovery
scalpel /evidence/original.raw -o /evidence/scalpel_output/
photorec /evidence/original.raw

# Foremost + YARA
# Scan recovered files for malicious content
yara -r /path/to/rules/ /evidence/recovered/

# Foremost + ExifTool
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
    foremost -i "$chunk" -o "output_$chunk/" &
done
wait
```

### Selective Scanning

```bash
# Only recover specific file types
foremost -i evidence.raw -o output/ -t jpg,png,pdf

# Use custom configuration
foremost -i evidence.raw -o output/ -c my_config.conf
```

### Using Faster Storage

```bash
# Write to SSD for faster I/O
foremost -i evidence.raw -o /ssd/output/

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
top -p $(pgrep foremost)
```

---

## Limitations and Considerations

### What Foremost Cannot Do

1. **Recover fragmented files**: Cannot reconstruct files stored in non-contiguous blocks
2. **Recover file names**: Original filenames are lost
3. **Recover directory structure**: Directory hierarchy is not preserved
4. **Handle encrypted files**: Cannot decrypt or recover encrypted content
5. **Recover from severely damaged media**: Use ddrescue first
6. **Guarantee accuracy**: May produce false positives

### Known Limitations

- **No filesystem awareness**: Works at raw block level only
- **No filename recovery**: Files are named sequentially
- **No directory structure**: All files saved to flat directories
- **Limited resume capability**: Must restart if interrupted
- **No GUI**: Command-line only

### When to Use Alternatives

| Situation | Better Tool | Reason |
|-----------|-------------|--------|
| Need filenames | TestDisk | Filesystem-aware |
| Need directory structure | TestDisk | Preserves hierarchy |
| Need resume capability | PhotoRec | Better resume |
| Need maximum file types | PhotoRec | 480+ formats |

---

## Detection and Defense

### For System Administrators

```bash
# Monitor for Foremost usage
which foremost
sudo ausearch -k foremost -ts recent

# Check for unauthorized file recovery
find /tmp /var/tmp /home -name "jpg" -o -name "pdf" -type d

# Check command history
cat ~/.bash_history | grep -i "foremost\|recover"

# Monitor disk I/O
sudo iotop
```

### For Forensic Investigators

```bash
# Check if Foremost was used as anti-forensics tool
# Look for recovery artifacts
find / -name "audit.txt" -type f 2>/dev/null

# Check command history
cat /root/.bash_history | grep -i "foremost"

# Examine system logs
sudo journalctl | grep -i "foremost\|recover"

# Check for Foremost output directories
find / -name "jpg" -o -name "pdf" -type d 2>/dev/null
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
# Check if image is readable
file evidence.raw

# Try different file types
foremost -i evidence.raw -o output/ -t jpg

# Check configuration
cat /etc/foremost.conf | grep -v "^#" | grep -v "^$"
```

#### "Permission denied"

```bash
# Run with sudo
sudo foremost -i /dev/sda -o output/

# Check file permissions
ls -la evidence.raw
chmod 644 evidence.raw
```

#### "Output directory already exists"

```bash
# Foremost requires a fresh output directory
rm -rf output/
mkdir output/
foremost -i evidence.raw -o output/
```

#### Too many false positives

```bash
# Enable footer matching in configuration
# Edit foremost.conf: set footer for each file type

# Use audit mode to see what would be carved
foremost -i evidence.raw -o output/ -T
```

#### Recovery is slow

```bash
# Only recover specific file types
foremost -i evidence.raw -o output/ -t jpg,png

# Use faster storage for output
foremost -i evidence.raw -o /ssd/output/
```

---

## Best Practices

### For Forensic Investigations

1. **Always work on images**: Never run Foremost on original evidence
2. **Document everything**: Record commands, outputs, and findings
3. **Use write blockers**: Prevent accidental modification of evidence
4. **Verify hashes**: MD5/SHA256 before and after
5. **Combine with other tools**: Use Foremost alongside Scalpel and PhotoRec

### For Data Recovery

1. **Stop using the affected device**: Prevent overwriting deleted data
2. **Create an image first**: Work on copies, not originals
3. **Check the audit log**: Review audit.txt for detailed results
4. **Verify recovered files**: Use `file` command to verify types
5. **Have a backup plan**: If Foremost fails, try other tools

### Configuration Tips

```bash
# 1. Start with default configuration
# 2. Customize for specific file types
# 3. Use audit mode to test configuration
# 4. Document all configuration changes
# 5. Verify results with other tools
```

---

## Comparison with Other Tools

### Foremost vs Scalpel vs PhotoRec

| Feature | Foremost | Scalpel | PhotoRec |
|---------|----------|---------|----------|
| Configuration | Simple | Complex | Simple |
| Speed | Fast | Fast | Medium |
| File types | Many | Many | Many |
| Audit mode | Yes | No | No |
| Resume | No | No | Yes |
| GUI | No | No | Yes |
| Active development | Medium | High | High |
| Kali pre-installed | Yes | Yes | Yes |

### When to Choose Foremost

- When you need a simple, reliable tool
- When you want audit mode for testing
- When you need a well-documented forensic tool
- When working with standard file types
- As a first-pass recovery tool

---

## References

- **Official Foremost Documentation**: https://foremost.sourceforge.net/
- **Foremost GitHub**: https://github.com/korczis/foremost
- **Kali Linux Package**: https://www.kali.org/tools/foremost/
- **File Carving Techniques**: https://www.sans.org/reading-room/forensic-analysis-file-carving-1411
- **Digital Forensics Framework**: https://www.digitalforensicsframework.org/
- **NIST SP 800-86**: Guide to Integrating Forensic Techniques into Incident Response

---

## Quick Reference Card

| Operation | Command |
|-----------|---------|
| Basic recovery | `foremost -i image.raw -o output/` |
| From device | `sudo foremost -i /dev/sda -o output/` |
| Specific types | `foremost -i image.raw -o output/ -t jpg,png` |
| Custom config | `foremost -i image.raw -o output/ -c config.conf` |
| Audit mode | `foremost -i image.raw -o output/ -T` |
| Verbose mode | `foremost -i image.raw -o output/ -v` |
| View results | `cat output/audit.txt` |
| Count files | `find output/ -type f \| wc -l` |

---

*Document Version: 1.0*
*Last Updated: 2026*
*Author: Digital Forensics Documentation Project*
*Classification: Educational*
