# Scalpel — File Carving Tool for Digital Forensics

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
- [Comparison with Other Carvers](#comparison-with-other-carvers)
- [References](#references)

---

## Overview

Scalpel is an open-source file carving utility used in digital forensics to recover deleted or lost files from disk images or raw device files. Developed originally by Simson Garfinkel and later maintained by the opensourceforensics community, Scalpel is a fast, lightweight file carver that works by scanning raw disk data for file headers and footers, then extracting the data between them.

Unlike filesystem-aware tools, Scalpel operates at the raw byte level, making it effective against formatted, corrupted, or overwritten media. It can recover files even when the filesystem metadata has been completely destroyed. Scalpel supports carving multiple file types simultaneously and is highly configurable through its configuration file.

### Key Features

- **Header/Footer Based Carving**: Recovers files by identifying known file signatures
- **Multi-Format Support**: Can carve dozens of file types simultaneously
- **Configurable**: Extensive configuration file for custom file types
- **Fast Performance**: Optimized scanning engine for large disk images
- **Cross-Platform**: Works on Linux, macOS, Windows, and other Unix systems
- **Active Development**: Maintained and updated regularly

### When to Use Scalpel

| Scenario | Use Scalpel? | Notes |
|----------|-------------|-------|
| Deleted files on healthy filesystem | Yes | Good for bulk recovery |
| Formatted drive recovery | Yes | Excellent choice |
| Corrupted filesystem | Yes | Works at byte level |
| Need specific file types | Yes | Highly configurable |
| Damaged disk (bad sectors) | No | Use ddrescue first |
| Large disk images | Yes | Fast scanning |
| Need carving statistics | Yes | Provides detailed reports |

---

## Installation

### Kali Linux

```bash
# Scalpel is pre-installed on Kali Linux
sudo apt update
sudo apt install scalpel

# Verify installation
scalpel --version
```

### Other Linux Distributions

```bash
# Debian/Ubuntu
sudo apt install scalpel

# Fedora/RHEL
sudo dnf install scalpel

# Arch Linux (from AUR)
yay -S scalpel

# From source
git clone https://github.com/sleuthkit/scalpel.git
cd scalpel
mkdir build && cd build
cmake ..
make -j$(nproc)
sudo make install
```

### Building from Source

```bash
# Install dependencies
sudo apt install build-essential cmake git

# Clone repository
git clone https://github.com/sleuthkit/scalpel.git
cd scalpel

# Build
mkdir build && cd build
cmake -DCMAKE_BUILD_TYPE=Release ..
make -j$(nproc)

# Install
sudo make install

# Verify
scalpel --version
```

---

## Core Concepts

### How File Carving Works

File carving works by scanning raw disk data for known file signatures:

1. **Header Search**: Scan for known file headers (magic bytes)
2. **Footer Search**: Look for corresponding file footer
3. **Data Extraction**: Extract data between header and footer
4. **Validation**: Optionally verify file integrity

### File Signatures

Each file type has a unique header (and often footer) signature:

| File Type | Header (Hex) | Footer (Hex) |
|-----------|--------------|--------------|
| JPEG | `FF D8 FF E0` | `FF D9` |
| PNG | `89 50 4E 47` | `AE 42 60 82` |
| PDF | `25 50 44 46` | `25 25 45 4F 46` |
| ZIP | `50 4B 03 04` | `50 4B 05 06` |
| GIF | `47 49 46 38` | `3B` |
| MP4 | `00 00 00 18 66 74 79 70` | Varies |
| DOCX | `50 4B 03 04` | `50 4B 05 06` |

### Carving Modes

Scalpel supports different carving modes:

- **Footer Mode**: Requires both header and footer (most accurate)
- **Header Only Mode**: Only requires header (more files recovered, more false positives)
- **Configurable**: Each file type can specify its own mode

---

## Command-Line Reference

### Basic Syntax

```bash
scalpel [options] <image_file>
```

### Options

| Option | Description |
|--------|-------------|
| `-o <output_dir>` | Output directory (must not exist) |
| `-c <config_file>` | Custom configuration file |
| `-b` | Carve block/fragment mode |
| `-e` | Carve tail (end) of files |
| `-f` | Carve forward from header |
| `-h` | Show help message |
| `-i` | Display file type identifiers |
| `-n` | Don't add extensions to carved files |
| `-o` | Set output directory |
| `-p` | Preallocate disk space for carved files |
| `-q` | Quiet mode (minimal output) |
| `-s` | Skip header/footer validation |
| `-t` | Carve contiguous blocks only |
| `-u` | Unhide deleted files |
| `-v` | Verbose output |
| `-w` | Carve all allocated files (not just deleted) |
| `-C <config>` | Use custom config file |
| `-D` | Debug mode |

---

## Configuration File

### Default Configuration Location

```bash
# System-wide config
/etc/scalpel/scalpel.conf

# User config (copy and modify)
cp /etc/scalpel/scalpel.conf ./my_scalpel.conf
```

### Configuration File Format

```
# File type  header  footer  case_sensitive  max_size
# header and footer are hex strings
# max_size: 0 = no limit

# Graphics
jpg     y       0xffd8ff      0xffd9          20000000
gif     y       0x47494638    0x3b            30000000
png     y       0x89504e47    0xae426082      30000000
bmp     y       0x424d        \xff\xff\xff     0
tiff    y       0x49492a00    0x00000000      0
tiff    y       0x4d4d002a    0x00000000      0
psd     y       0x38425053    0x00            0

# Documents
pdf     y       0x25504446    0x2525454f46    50000000
doc     y       0xd0cf11e0    0x00000000      0
docx    y       0x504b0304    0x504b0506      50000000
xlsx    y       0x504b0304    0x504b0506      50000000
pptx    y       0x504b0304    0x504b0506      50000000
odt     y       0x504b0304    0x504b0506      0
ods     y       0x504b0304    0x504b0506      0

# Archives
zip     y       0x504b0304    0x504b0506      100000000
rar     y       0x52617221    0x00            0
7z      y       0x377abcaf    0x00            0
tar     y       0x75737461    0x00000000      0
gz      y       0x1f8b        0x00            0

# Audio
mp3     y       0x494433      0x00000000      0
wav     y       0x52494646    0x00000000      0
ogg     y       0x4f676753    0x00000000      0
flac    y       0x664c6143    0x00000000      0

# Video
mp4     y       0x00000018    0x00000000      0
avi     y       0x52494646    0x00000000      0
wmv     y       0x3026b275    0x00000000      0
mov     y       0x00000014    0x00000000      0
flv     y       0x464c5601    0x00000000      0

# Executables
elf     y       0x7f454c46    0x00000000      0
exe     y       0x4d5a        0x00000000      0
dll     y       0x4d5a        0x00000000      0

# Email
pst     y       0x2142444e    0x00000000      0
dbx     y       0xcfad12fe    0x00000000      0

# Databases
mdb     y       0x00010000    0x00000000      0
sqlite  y       0x53514c69    0x00000000      0
```

### Creating Custom File Types

```
# Add a custom file type to the configuration
# Format: name  case_sensitive  header  footer  max_size

# Custom XML file
xml     y       0x3c3f786d6c   0x3c2f786d6c3e   0

# Custom log format
mylog   y       0x4c4f473a     0x0a0a0a         0
```

---

## Basic Usage

### Simple Carving

```bash
# Create output directory
mkdir output

# Carve all file types from disk image
scalpel image.raw -o output/

# Carve specific file types (modify config first)
cp /etc/scalpel/scalpel.conf ./custom.conf
# Edit custom.conf to enable only desired types
scalpel -c custom.conf image.raw -o output/
```

### Carving from a Device

```bash
# Carve from a partition
sudo scalpel /dev/sda1 -o output/

# Carve from raw disk (create image first for forensics)
sudo scalpel /dev/sda -o output/
```

### Carving from a Disk Image

```bash
# From a raw disk image
scalpel evidence.raw -o output/

# From an E01 image (need to convert first)
ewfmount evidence.E01 /mnt/ewf
scalpel /mnt/ewf/ewf1 -o output/
```

---

## Advanced Usage

### Command-Line Scenarios

#### Scenario 1: Recover Only JPEG Files

```bash
# Create custom config with only JPEG
cat > jpeg_only.conf << 'EOF'
# Scalpel configuration for JPEG recovery only
jpg     y       0xffd8ff      0xffd9          20000000
EOF

scalpel -c jpeg_only.conf evidence.raw -o output/
```

#### Scenario 2: Aggressive Carving (More Files, More False Positives)

```bash
# Use header-only mode for more recovery
cat > aggressive.conf << 'EOF'
# Header-only carving for maximum recovery
jpg     n       0xffd8ff      0xffd9          0
png     n       0x89504e47    0xae426082      0
pdf     n       0x25504446    0x2525454f46    0
EOF

scalpel -c aggressive.conf evidence.raw -o output/
```

#### Scenario 3: Carve with Maximum File Size

```bash
# Recover large files (e.g., video files)
cat > large_files.conf << 'EOF'
# Large file recovery
mp4     y       0x00000018    0x00000000      0
avi     y       0x52494646    0x00000000      0
mov     y       0x00000014    0x00000000      0
EOF

scalpel -c large_files.conf evidence.raw -o output/
```

#### Scenario 4: Skip Allocated Files

```bash
# Only carve unallocated/deleted files
scalpel -u evidence.raw -o output/

# Skip header validation for damaged files
scalpel -s evidence.raw -o output/
```

### Analyzing Results

```bash
# View carving statistics
cat output/scalpel-output.txt

# Count recovered files by type
find output/ -type f | sed 's/.*\.//' | sort | uniq -c | sort -rn

# Check file sizes
find output/ -type f -exec ls -lh {} \; | awk '{print $5, $9}'

# Verify file types with file command
find output/ -type f -exec file {} \;
```

---

## Forensics Workflow Integration

### Phase 1: Evidence Preparation

```bash
# 1. Create a bit-for-bit image
sudo dd if=/dev/sda of=/evidence/original.raw bs=4M status=progress

# 2. Verify image integrity
md5sum /dev/sda /evidence/original.raw

# 3. Create working copy
cp /evidence/original.raw /evidence/working.raw

# 4. Document the process
echo "Image created: $(date)" > /evidence/log.txt
echo "MD5: $(md5sum /evidence/original.raw)" >> /evidence/log.txt
```

### Phase 2: Carving with Scalpel

```bash
# 1. Prepare carving configuration
cp /etc/scalpel/scalpel.conf /evidence/scalpel_custom.conf
# Edit configuration for case-specific file types

# 2. Create output directory
mkdir -p /evidence/carved_files

# 3. Run Scalpel
scalpel -c /evidence/scalpel_custom.conf \
        /evidence/working.raw \
        -o /evidence/carved_files/

# 4. Document results
cp /evidence/carved_files/scalpel-output.txt /evidence/
```

### Phase 3: Post-Carving Analysis

```bash
# 1. Sort recovered files
mkdir -p /evidence/sorted/{images,documents,archives,other}

find /evidence/carved_files/ -type f -name "*.jpg" -exec mv {} /evidence/sorted/images/ \;
find /evidence/carved_files/ -type f -name "*.pdf" -exec mv {} /evidence/sorted/documents/ \;
find /evidence/carved_files/ -type f -name "*.zip" -exec mv {} /evidence/sorted/archives/ \;

# 2. Generate file hashes
find /evidence/carved_files/ -type f -exec md5sum {} \; > /evidence/carved_hashes.txt

# 3. Filter false positives
find /evidence/carved_files/ -type f -size 0 -delete  # Remove empty files
find /evidence/carved_files/ -type f -size -100c -delete  # Remove tiny files

# 4. Visual inspection of images
# Use display, eog, or similar for manual review
```

### Integration with Other Tools

```bash
# Scalpel + The Sleuth Kit
# Use TSK to identify carved file metadata
fls -r -d /evidence/working.raw > /evidence/tsk_output.txt

# Scalpel + Autopsy
# Import carved files into Autopsy for timeline analysis

# Scalpel + YARA
# Scan carved files for malicious content
yara -r /path/to/rules/ /evidence/carved_files/

# Scalpel + ExifTool
# Extract metadata from carved images
find /evidence/carved_files/ -name "*.jpg" -exec exiftool {} \;
```

---

## Performance Optimization

### Parallel Processing

```bash
# Split disk image for parallel carving
# Divide image into chunks
split -b 1G evidence.raw chunk_

# Carve each chunk in parallel
for chunk in chunk_*; do
    mkdir -p "output_$chunk"
    scalpel -c scalpel.conf "$chunk" -o "output_$chunk/" &
done
wait

# Merge results
# (Note: may miss files spanning chunk boundaries)
```

### Memory and I/O Optimization

```bash
# Increase buffer size for faster I/O
# Modify scalpel.conf or use environment variables

# Use faster storage for output
scalpel evidence.raw -o /ssd/output/

# Monitor resource usage
top -p $(pgrep scalpel)
iostat -x 1
```

### Selective Carving

```bash
# Only carve specific sectors
dd if=evidence.raw of=subset.raw bs=512 skip=1000000 count=100000
scalpel subset.raw -o output/

# Only carve deleted files (skip allocated)
scalpel -u evidence.raw -o output/
```

---

## Limitations and Considerations

### What Scalpel Cannot Do

1. **Recover files spanning fragmented sectors**: Cannot reconstruct fragmented files
2. **Handle encrypted files**: Cannot decrypt or carve encrypted content
3. **Recover from severely damaged media**: Use ddrescue first for damaged disks
4. **Guarantee accuracy**: May produce false positives (junk files)
5. **Recover overwritten data**: Cannot recover data that has been overwritten

### Known Limitations

- **Fragmentation**: Files stored in non-contiguous blocks cannot be fully recovered
- **False Positives**: Header/footer matching can produce junk files
- **Speed on Large Images**: Very large disk images (1TB+) can take hours
- **Memory Usage**: High memory consumption on large images
- **No File System Awareness**: Cannot use filesystem metadata for recovery

### Comparison with PhotoRec

| Feature | Scalpel | PhotoRec |
|---------|---------|----------|
| Speed | Faster | Slower |
| Accuracy | Higher (with proper config) | Good |
| Configuration | Highly configurable | Less configurable |
| Fragmentation | No support | No support |
| Active Development | Yes | Yes |
| Default File Types | Fewer | More |

---

## Detection and Defense

### For System Administrators

```bash
# Monitor for file carving tools
which scalpel photorec foremost

# Check for unauthorized carving activity
sudo ausearch -k scalpel -ts recent
sudo ausearch -k photorec -ts recent

# Monitor disk I/O for unusual patterns
sudo iotop

# Check for carved output directories
find /tmp /var/tmp /home -name "output" -type d -mtime -1
```

### For Forensic Investigators

```bash
# Check if Scalpel was used as an anti-forensics tool
# Look for carving artifacts in:
cat ~/.bash_history | grep -i "scalpel\|photorec\|foremost"

# Check for carving configuration files
find / -name "scalpel.conf" -o -name "scalpel_output*" 2>/dev/null

# Examine system logs for carving activity
sudo journalctl | grep -i "scalpel\|carving"

# Check for evidence of file overwriting
sudo debugfs -R "ls -d" /dev/sda2
```

### Defensive Measures

```bash
# 1. Use full-disk encryption to prevent carving
sudo cryptsetup luksFormat /dev/sda2

# 2. Implement secure deletion
shred -vfz -n 5 sensitive_file.txt

# 3. Use TRIM on SSDs to prevent recovery
sudo fstrim -v /

# 4. Monitor for unauthorized tools
sudo apt install rkhunter chkrootkit
sudo rkhunter --check
```

---

## Troubleshooting

### Common Issues and Solutions

#### "No files carved"

```bash
# Check configuration file
cat /etc/scalpel/scalpel.conf | grep -v "^#" | grep -v "^$"

# Verify image is readable
file evidence.raw
hexdump -C evidence.raw | head -20

# Try different file types
scalpel -c /etc/scalpel/scalpel.conf evidence.raw -o output/
```

#### "Permission denied"

```bash
# Run with sudo
sudo scalpel /dev/sda -o output/

# Check file permissions
ls -la evidence.raw
chmod 644 evidence.raw
```

#### "Output directory already exists"

```bash
# Scalpel requires a fresh output directory
rm -rf output/
mkdir output/
scalpel evidence.raw -o output/
```

#### "Too many false positives"

```bash
# Enable footer matching
# Edit scalpel.conf: set footer for each file type
# Use case-sensitive matching: set 'y' in case_sensitive column
```

#### "Scalpel is too slow"

```bash
# Use SSD for output directory
scalpel evidence.raw -o /ssd/output/

# Limit file types in configuration
# Only enable carving for needed file types

# Use parallel processing (see Performance Optimization)
```

---

## Best Practices

### For Forensic Investigations

1. **Always work on images**: Never carve directly from original evidence
2. **Document configuration**: Record exactly which file types were carved
3. **Preserve original output**: Keep Scalpel's raw output before filtering
4. **Verify file types**: Use `file` command to verify carved file types
5. **Generate hashes**: MD5/SHA256 all recovered files

### For Data Recovery

1. **Start with the right tool**: Use Scalpel for bulk file recovery
2. **Configure properly**: Edit scalpel.conf for your specific needs
3. **Check results**: Manually verify critical recovered files
4. **Use multiple tools**: Combine Scalpel with PhotoRec for best results
5. **Work from images**: Always work on copies, not originals

### Configuration Tips

```bash
# 1. Start with minimal config, add as needed
# 2. Use footer matching for accuracy
# 3. Set appropriate max_size to prevent huge junk files
# 4. Test configuration on small samples first
# 5. Document all configuration changes
```

---

## Comparison with Other Carvers

### Scalpel vs PhotoRec vs Foremost

| Feature | Scalpel | PhotoRec | Foremost |
|---------|---------|----------|----------|
| Speed | Fast | Medium | Medium |
| Configuration | High | Medium | Medium |
| Default Types | Many | Many | Few |
| Accuracy | High | High | Medium |
| Recovery Rate | Good | Good | Good |
| Active Development | Yes | Yes | Yes |
| Kali Pre-installed | Yes | Yes | Yes |
| Learning Curve | Medium | Low | Low |

### When to Choose Scalpel

- When you need maximum speed
- When you need custom file type support
- When you want detailed control over carving parameters
- When processing very large disk images

---

## References

- **Official Scalpel Repository**: https://github.com/sleuthkit/scalpel
- **Scalpel Documentation**: https://github.com/sleuthkit/scalpel/blob/master/README
- **Kali Linux Package**: https://www.kali.org/tools/scalpel/
- **Digital Forensics Framework**: https://www.digitalforensicsframework.org/
- **NIST SP 800-86**: Guide to Integrating Forensic Techniques into Incident Response
- **File Carving Techniques**: https://www.sans.org/reading-room/forensic-analysis-file-carving-1411

---

## Quick Reference Card

| Operation | Command |
|-----------|---------|
| Basic carving | `scalpel image.raw -o output/` |
| With custom config | `scalpel -c config.conf image.raw -o output/` |
| From device | `sudo scalpel /dev/sda -o output/` |
| Skip allocated | `scalpel -u image.raw -o output/` |
| Verbose mode | `scalpel -v image.raw -o output/` |
| View results | `cat output/scalpel-output.txt` |
| Count carved files | `find output/ -type f \| wc -l` |
| Verify file types | `find output/ -type f -exec file {} \;` |

---

*Document Version: 1.0*
*Last Updated: 2026*
*Author: Digital Forensics Documentation Project*
*Classification: Educational*
