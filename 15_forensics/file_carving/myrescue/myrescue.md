# MyRescue — Rescue Data from Damaged Disks

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

MyRescue is a lightweight data recovery tool designed to rescue data from damaged or failing hard disks. Developed by Thomas Wilke (the same author as Safecopy), MyRescue provides a simple yet effective approach to recovering data from media with bad sectors, surface damage, or other physical issues. It is particularly useful for creating disk images from damaged drives where standard tools like dd would fail due to I/O errors.

MyRescue works by attempting to read disk sectors and skipping those that cause errors, then retrying failed sectors with different strategies. This approach allows it to recover as much data as possible from damaged media while minimizing the risk of further damage.

### Key Features

- **Simple Interface**: Easy-to-use command-line tool
- **Error Recovery**: Gracefully handles I/O errors
- **Retry Logic**: Automatically retries failed sectors
- **Progress Reporting**: Shows recovery progress and statistics
- **Low Resource Usage**: Minimal memory and CPU requirements
- **Cross-Platform**: Works on Linux and other Unix-like systems

### When to Use MyRescue

| Scenario | Use MyRescue? | Notes |
|----------|--------------|-------|
| Damaged hard drive | Yes | Primary use case |
| Failing media | Yes | Good for failing drives |
| Bad sectors | Yes | Handles I/O errors gracefully |
| Forensic image of damaged disk | Yes | Use before other tools |
| Healthy disk | No | Use dd instead |
| Need file recovery | No | Use photorec/scalpel after imaging |

---

## Installation

### Kali Linux

```bash
# MyRescue may need to be installed manually
sudo apt update
sudo apt install myrescue

# If not in repositories, build from source
```

### Other Linux Distributions

```bash
# Debian/Ubuntu
sudo apt install myrescue

# From source
git clone https://github.com/thomaswilke/myrescue.git
cd myrescue
make
sudo make install
```

### Building from Source

```bash
# Install build dependencies
sudo apt install build-essential git

# Clone repository
git clone https://github.com/thomaswilke/myrescue.git
cd myrescue

# Build
make

# Install
sudo cp myrescue /usr/local/bin/

# Verify
myrescue --version
```

---

## Core Concepts

### How MyRescue Works

MyRescue operates by:

1. **Reading sectors sequentially** from the source device
2. **Handling I/O errors** by skipping failed sectors
3. **Retrying failed sectors** with different read strategies
4. **Recording error positions** for later analysis
5. **Saving recovered data** to the output file

### Error Handling Strategy

MyRescue uses a multi-stage approach:

- **First attempt**: Standard read
- **Second attempt**: Retry with different parameters
- **Third attempt**: Skip and record position
- **Final attempt**: Mark as unrecoverable

### Recovery Statistics

MyRescue provides detailed statistics:

```
[INFO] Starting recovery from /dev/sda
[INFO] Output: /evidence/image.raw
[INFO] Total sectors: 976773168
[INFO] Recovered: 976770000 (99.99%)
[INFO] Errors: 3168
[INFO] Unrecoverable: 0
```

---

## Command-Line Reference

### Basic Syntax

```bash
myrescue [options] <source> <destination>
```

### Options

| Option | Description |
|--------|-------------|
| `-b`, `--blocksize <size>` | Block size in bytes (default: 512) |
| `-r`, `--retries <n>` | Number of retries per sector (default: 3) |
| `-s`, `--start <sector>` | Start sector |
| `-e`, `--end <sector>` | End sector |
| `-v`, `--verbose` | Verbose output |
| `-q`, `--quiet` | Quiet mode |
| `-h`, `--help` | Show help |
| `-V`, `--version` | Show version |

### Device Specification

```bash
# Recover entire disk
myrescue /dev/sda /evidence/image.raw

# Recover specific partition
myrescue /dev/sda1 /evidence/partition.raw

# Recover specific sector range
myrescue -s 1000000 -e 2000000 /dev/sda /evidence/partial.raw
```

---

## Basic Usage

### Creating a Disk Image

```bash
# Create output directory
mkdir -p /evidence

# Recover entire disk
sudo myrescue /dev/sda /evidence/damaged_disk.raw

# Check results
ls -lh /evidence/damaged_disk.raw
```

### Recovering Specific Sectors

```bash
# Recover first 1GB of disk
sudo myrescue -s 0 -e 2097152 /dev/sda /evidence/first_1gb.raw

# Recover from middle of disk
sudo myrescue -s 500000000 -e 502097152 /dev/sda /evidence/middle.raw
```

### Recovering from Optical Media

```bash
# Recover from scratched CD
sudo myrescue /dev/cdrom /evidence/cd_image.raw

# Recover from DVD
sudo myrescue /dev/dvd /evidence/dvd_image.raw
```

---

## Advanced Usage

### Command-Line Scenarios

#### Scenario 1: Maximum Recovery from Failing Drive

```bash
# Run with verbose output and multiple retries
sudo myrescue -r 5 -v /dev/sda /evidence/image.raw

# Check statistics
echo "Recovery complete at $(date)" >> /evidence/log.txt
ls -lh /evidence/image.raw >> /evidence/log.txt
```

#### Scenario 2: Recover Specific Partition

```bash
# Calculate partition offset
# Example: Partition starts at sector 2048
sudo myrescue -s 2048 -e 1026047 /dev/sda /evidence/partition.raw
```

#### Scenario 3: Resume Interrupted Recovery

```bash
# If myrescue is interrupted, check what was recovered
ls -lh /evidence/image.raw

# MyRescue can resume by skipping already-recovered sectors
# Simply run the same command again
sudo myrescue /dev/sda /evidence/image.raw
```

#### Scenario 4: Debugging I/O Errors

```bash
# Run with maximum verbosity
sudo myrescue -v -r 5 /dev/sda /evidence/debug_image.raw

# Analyze error patterns
grep -i "error" /evidence/debug_image.raw.log 2>/dev/null
```

### Analyzing Results

```bash
# Check recovered file size
ls -lh /evidence/image.raw

# Verify image is readable
file /evidence/image.raw

# Check for bad sectors in image
hexdump -C /evidence/image.raw | grep "00 00 00 00 00 00 00 00" | head -20

# Analyze with file system tools
sudo fsstat /evidence/image.raw
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

# 3. Document S.M.A.R.T. data
sudo smartctl -A /dev/sda > /evidence/smart_attributes.txt
```

### Phase 2: Image Creation with MyRescue

```bash
# 1. Create output directory structure
mkdir -p /evidence/{images,logs,analysis}

# 2. Run MyRescue with documentation
sudo myrescue -v /dev/sda /evidence/images/original.raw \
    2>&1 | tee /evidence/logs/myrescue_output.txt

# 3. Verify image creation
ls -lh /evidence/images/original.raw

# 4. Check recovery statistics
echo "Recovery completed: $(date)" >> /evidence/logs/summary.txt
echo "Image size: $(ls -lh /evidence/images/original.raw | awk '{print $5}')" >> /evidence/logs/summary.txt
```

### Phase 3: Post-Recovery Analysis

```bash
# 1. Analyze image with file system tools
sudo fsstat /evidence/images/original.raw

# 2. List files in image
fls -r -d /evidence/images/original.raw > /evidence/analysis/file_list.txt

# 3. Carve files if needed
scalpel /evidence/images/original.raw -o /evidence/analysis/carved/

# 4. Generate forensic report
cat > /evidence/analysis/report.txt << EOF
MyRescue Recovery Report
========================
Case: CASE-2026-001
Date: $(date)
Source: /dev/sda
Image: /evidence/images/original.raw
Image Size: $(ls -lh /evidence/images/original.raw | awk '{print $5}')
EOF
```

### Integration with Other Tools

```bash
# MyRescue + TestDisk
# After creating image, use TestDisk for partition recovery
sudo testdisk /evidence/images/original.raw

# MyRescue + PhotoRec
# After creating image, use PhotoRec for file carving
sudo photorec /evidence/images/original.raw

# MyRescue + The Sleuth Kit
# Analyze image with TSK tools
fls -r -d /evidence/images/original.raw
icat /evidence/images/original.raw inode_number > recovered_file

# MyRescue + Safecopy
# Compare results from both tools
diff <(md5sum /evidence/myrescue_image.raw) <(md5sum /evidence/safecopy_image.raw)
```

---

## Performance Optimization

### Block Size Optimization

```bash
# Default block size (512 bytes)
myrescue /dev/sda /evidence/image.raw

# Larger block size for faster reading of good sectors
myrescue -b 4096 /dev/sda /evidence/image.raw

# Smaller block size for more precise error recovery
myrescue -b 256 /dev/sda /evidence/image.raw
```

### Retry Optimization

```bash
# Default retries (3)
myrescue /dev/sda /evidence/image.raw

# More retries for difficult media
myrescue -r 10 /dev/sda /evidence/image.raw

# Fewer retries for faster recovery
myrescue -r 1 /dev/sda /evidence/image.raw
```

### Using Faster Storage

```bash
# Write to SSD for faster I/O
myrescue /dev/sda /ssd/evidence/image.raw

# Then copy to long-term storage
cp /ssd/evidence/image.raw /nas/evidence/image.raw
```

### Monitoring Progress

```bash
# Monitor MyRescue progress
watch -n 5 'ls -lh /evidence/image.raw'

# Monitor system I/O
iostat -x 1

# Check for errors in real-time
tail -f /evidence/myrescue.log | grep -i "error"
```

---

## Limitations and Considerations

### What MyRescue Cannot Do

1. **Recover overwritten data**: Cannot recover data that has been overwritten
2. **Fix hardware failures**: Cannot repair physically damaged media
3. **Decrypt encrypted volumes**: Cannot bypass encryption
4. **Recover fragmented files**: Creates raw image, not file-level recovery
5. **Guarantee 100% recovery**: Some data may be unrecoverable

### Known Limitations

- **Sequential reading**: May be slow on very large disks
- **No compression**: Output is uncompressed raw data
- **No encryption**: Output is not encrypted
- **Limited file system awareness**: Works at block level only
- **No GUI**: Command-line only

### When to Use Alternatives

| Situation | Better Tool | Reason |
|-----------|-------------|--------|
| Healthy disk | dd | Faster, simpler |
| Need file recovery | photorec | File-level recovery |
| Need partition recovery | testdisk | Partition-level recovery |
| Need compression | dcfldd | Built-in compression |
| Need hashing | dc3dd | Built-in hashing |

---

## Detection and Defense

### For System Administrators

```bash
# Monitor for MyRescue usage
which myrescue
sudo ausearch -k myrescue -ts recent

# Check for unusual disk imaging activity
sudo iotop
sudo iostat -x 1

# Check for image files in unusual locations
find /tmp /var/tmp /home -name "*.raw" -o -name "*.dd" -mtime -1

# Monitor command history
cat ~/.bash_history | grep -i "myrescue\|dd\|safecopy"
```

### For Forensic Investigators

```bash
# Check if MyRescue was used as anti-forensics tool
# Look for evidence of disk imaging
find / -name "*.raw" -o -name "*.dd" -o -name "*.img" 2>/dev/null

# Check for command history
cat /root/.bash_history | grep -i "myrescue\|dd"

# Examine system logs
sudo journalctl | grep -i "myrescue\|block\|disk"

# Check for hidden image files
find / -name ".*.raw" -o -name ".*.dd" 2>/dev/null
```

### Defensive Measures

```bash
# 1. Use full-disk encryption
sudo cryptsetup luksFormat /dev/sda2

# 2. Enable secure boot to prevent unauthorized imaging
# 3. Monitor physical access to devices
# 4. Use hardware write blockers for evidence
# 5. Implement device access controls
```

---

## Troubleshooting

### Common Issues and Solutions

#### "Permission denied"

```bash
# Run with sudo
sudo myrescue /dev/sda /evidence/image.raw

# Check device permissions
ls -la /dev/sda
```

#### "No space left on device"

```bash
# Check available space
df -h

# Check source device size
sudo blockdev --getsize64 /dev/sda

# Ensure destination has enough space
```

#### "Input/output error"

```bash
# This is expected for damaged media
# MyRescue will continue past errors
# Check the log for recovery statistics
```

#### Recovery seems stuck

```bash
# Check if myrescue is still running
ps aux | grep myrescue

# Check I/O activity
iostat -x 1

# If truly stuck, try with different block size
sudo myrescue -b 4096 /dev/sda /evidence/image.raw
```

---

## Best Practices

### For Forensic Investigations

1. **Always document the process**: Record every command and its output
2. **Use write blockers**: Hardware or software write blockers
3. **Work in a controlled environment**: Clean, static-free workspace
4. **Create multiple backups**: Don't rely on a single image
5. **Verify hashes**: MD5/SHA256 before and after

### For Data Recovery

1. **Stop using the damaged device**: Prevent further degradation
2. **Try multiple retry counts**: Different media needs different retry strategies
3. **Be patient**: Recovery can take hours or days
4. **Check results**: Verify recovered data integrity
5. **Have a backup plan**: If MyRescue fails, try Safecopy or ddrescue

### Safety Rules

```bash
# 1. Never run on original evidence without documentation
# 2. Always create a working copy first
# 3. Verify hash values before and after
# 4. Use write blockers when possible
# 5. Document everything
```

---

## Comparison with Other Tools

### MyRescue vs Safecopy vs ddrescue

| Feature | MyRescue | Safecopy | ddrescue |
|---------|----------|----------|----------|
| Simplicity | Excellent | Good | Good |
| Error handling | Good | Excellent | Excellent |
| Resume capability | Yes | Yes | Yes |
| Progress reporting | Basic | Good | Excellent |
| Speed | Fast | Medium | Fast |
| Learning curve | Very low | Low | Medium |
| Forensics use | Yes | Yes | Yes |

### When to Choose MyRescue

- When you need the simplest possible tool
- When working with limited system resources
- When you need a quick first-pass recovery
- When other tools fail due to complexity
- When you need minimal configuration

---

## References

- **MyRescue Documentation**: https://www.jmillerinc.com/myrescue.html
- **MyRescue GitHub**: https://github.com/thomaswilke/myrescue
- **Kali Linux Forensics**: https://www.kali.org/tools/myrescue/
- **Data Recovery Guide**: https://www.sans.org/reading-room/data-recovery-techniques-1543
- **Digital Forensics Framework**: https://www.digitalforensicsframework.org/
- **NIST SP 800-86**: Guide to Integrating Forensic Techniques into Incident Response

---

## Quick Reference Card

| Operation | Command |
|-----------|---------|
| Basic recovery | `sudo myrescue /dev/sda image.raw` |
| With retries | `sudo myrescue -r 5 /dev/sda image.raw` |
| Specific sectors | `sudo myrescue -s 0 -e 1000000 /dev/sda image.raw` |
| From optical media | `sudo myrescue /dev/cdrom image.raw` |
| Verbose output | `sudo myrescue -v /dev/sda image.raw` |
| View results | `ls -lh image.raw` |
| Verify image | `file image.raw` |
| Check size | `sudo blockdev --getsize64 /dev/sda` |

---

*Document Version: 1.0*
*Last Updated: 2026*
*Author: Digital Forensics Documentation Project*
*Classification: Educational*
