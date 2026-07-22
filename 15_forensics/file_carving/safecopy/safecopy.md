# Safecopy — Data Recovery Tool for Damaged Media

## Table of Contents

- [Overview](#overview)
- [Installation](#installation)
- [Core Concepts](#core-concepts)
- [Command-Line Reference](#command-line-reference)
- [Recovery Stages](#recovery-stages)
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

Safecopy is a data recovery tool designed to salvage data from damaged or failing media. Unlike standard copy utilities that would abort on I/O errors, Safecopy uses a three-stage recovery process to maximize data recovery from media with bad sectors, surface damage, or other physical issues. Originally developed by Thomas Wilke, Safecopy is an essential tool in the digital forensics and data recovery toolkit.

Safecopy's three-stage approach progressively attempts to recover data, from reading only undamaged areas to attempting increasingly aggressive reads of damaged sectors. This makes it particularly valuable for:
- Recovering data from failing hard drives
- Salvaging data from scratched CDs/DVDs
- Creating images of damaged flash media
- Forensic acquisition of damaged evidence

### Key Features

- **Three-Stage Recovery**: Progressive recovery from damaged media
- **Bad Sector Handling**: Gracefully handles I/O errors without aborting
- **Progress Reporting**: Detailed progress output with statistics
- **Resume Capability**: Can resume interrupted recovery operations
- **Cross-Platform**: Works on Linux, macOS, and other Unix systems
- **Lightweight**: Minimal resource requirements

### When to Use Safecopy

| Scenario | Use Safecopy? | Notes |
|----------|--------------|-------|
| Damaged hard drive | Yes | Primary use case |
| Scratched optical media | Yes | Excellent for CDs/DVDs |
| Failing flash media | Yes | Good for USB drives, SD cards |
| Forensic image of damaged disk | Yes | Use before other tools |
| Healthy disk imaging | No | Use dd or dc3dd instead |
| File carving preparation | Yes | Create image first |

---

## Installation

### Kali Linux

```bash
# Safecopy is pre-installed on Kali Linux
sudo apt update
sudo apt install safecopy

# Verify installation
safecopy --version
```

### Other Linux Distributions

```bash
# Debian/Ubuntu
sudo apt install safecopy

# Fedora/RHEL
sudo dnf install safecopy

# Arch Linux
sudo pacman -S safecopy

# From source
git clone https://github.com/claunia/safecopy.git
cd safecopy
./configure
make
sudo make install
```

### Building from Source

```bash
# Install dependencies
sudo apt install build-essential git

# Clone repository
git clone https://github.com/claunia/safecopy.git
cd safecopy

# Build
./autogen.sh
./configure
make -j$(nproc)

# Install
sudo make install

# Verify
safecopy --version
```

---

## Core Concepts

### Three-Stage Recovery

Safecopy's unique advantage is its three-stage recovery process:

#### Stage 1: Fast Read
- Reads only sectors that respond quickly
- Skips sectors that cause I/O errors
- Fastest stage, recovers undamaged data

#### Stage 2: Restricted Read
- Attempts to read sectors that failed in Stage 1
- Uses shorter timeouts
- Recovers partially damaged sectors

#### Stage 3: Unrestricted Read
- Aggressive read attempts on all failed sectors
- Uses maximum timeouts
- Last resort for badly damaged sectors

### Error Handling

Safecopy handles errors gracefully:

```
[ERROR] read: Input/output error at offset 123456789
[STAGE] Skipping to next good sector
[PROGRESS] 45.2% complete (123456789/274877906944 bytes)
```

### Recovery Statistics

Safecopy provides detailed statistics:

```
[STATS] Stage 1: 98.5% recovered (270845038848/274877906944 bytes)
[STATS] Stage 2: 0.3% recovered (824633721/274877906944 bytes)
[STATS] Stage 3: 0.1% recovered (274877906/274877906944 bytes)
[STATS] Total: 98.9% recovered (271944550375/274877906944 bytes)
```

---

## Command-Line Reference

### Basic Syntax

```bash
safecopy [options] <source> <destination>
```

### Options

| Option | Description |
|--------|-------------|
| `-s`, `--stage1` | Stage 1 only (fast read) |
| `-S`, `--stage2` | Stage 2 only (restricted read) |
| `-U`, `--stage3` | Stage 3 only (unrestricted read) |
| `-1` | Stage 1 |
| `-2` | Stage 2 |
| `-3` | Stage 3 |
| `-l`, `--loglevel <level>` | Set log level (0-3) |
| `-f`, `--force` | Force overwriting destination |
| `-r`, `--retries <n>` | Number of retries per sector |
| `-t`, `--trypartial` | Try partial reads |
| `-b`, `--blocksize <size>` | Block size in bytes |
| `-o`, `--offset <n>` | Start offset in source |
| `-l`, `--length <n>` | Length to copy |
| `-h`, `--help` | Show help message |
| `-v`, `--version` | Show version |
| `-x`, `--debug` | Enable debug output |

### Source and Destination

```bash
# Device to file
safecopy /dev/sda /evidence/disk_image.raw

# Device to device
safecopy /dev/sda /dev/sdb

# File to file
safecopy damaged.img recovered.img

# Using dd-style notation
safecopy if=/dev/sda of=/evidence/image.raw
```

---

## Recovery Stages

### Stage 1: Fast Read (Default)

```bash
# Run Stage 1 only
safecopy -s /dev/sda /evidence/image.raw

# Or equivalently
safecopy -1 /dev/sda /evidence/image.raw
```

**What happens:**
- Reads sectors sequentially
- Skips sectors that cause I/O errors
- Records positions of failed sectors
- Fastest stage

**When to use:**
- Initial attempt at recovery
- When most of the disk is readable
- Time-sensitive recovery

### Stage 2: Restricted Read

```bash
# Run Stage 2 only (requires Stage 1 log)
safecopy -S /dev/sda /evidence/image.raw

# Or equivalently
safecopy -2 /dev/sda /evidence/image.raw
```

**What happens:**
- Attempts to read sectors that failed in Stage 1
- Uses shorter I/O timeouts
- Tries different read strategies
- Moderate speed

**When to use:**
- After Stage 1 completes
- When Stage 1 left gaps
- Partially damaged media

### Stage 3: Unrestricted Read

```bash
# Run Stage 3 only (requires previous stage logs)
safecopy -U /dev/sda /evidence/image.raw

# Or equivalently
safecopy -3 /dev/sda /evidence/image.raw
```

**What happens:**
- Aggressive read attempts on all failed sectors
- Uses maximum I/O timeouts
- Attempts partial sector reads
- Slowest stage

**When to use:**
- After Stage 2 completes
- When maximum recovery is needed
- Severely damaged media

### Running All Stages

```bash
# Run all three stages sequentially
safecopy /dev/sda /evidence/image.raw

# This automatically runs Stage 1 → Stage 2 → Stage 3
```

---

## Basic Usage

### Creating a Disk Image from Damaged Media

```bash
# Step 1: Create output directory
mkdir -p /evidence

# Step 2: Run safecopy (all stages)
sudo safecopy /dev/sda /evidence/damaged_disk.raw

# Step 3: Check results
ls -lh /evidence/damaged_disk.raw*
cat /evidence/damaged_disk.raw.log
```

### Recovering from Optical Media

```bash
# Recover data from a scratched CD
sudo safecopy /dev/cdrom /evidence/cd_image.raw

# For DVDs
sudo safecopy /dev/dvd /evidence/dvd_image.raw
```

### Recovering from Flash Media

```bash
# USB drive
sudo safecopy /dev/sdb /evidence/usb_image.raw

# SD card
sudo safecopy /dev/mmcblk0 /evidence/sd_image.raw
```

### Creating a Partial Image

```bash
# Copy only specific sectors
sudo safecopy -o 1000000 -l 500000 /dev/sda /evidence/partial.raw

# This copies 500000 sectors starting from sector 1000000
```

---

## Advanced Usage

### Command-Line Scenarios

#### Scenario 1: Maximum Recovery from Failing Drive

```bash
# Create output directory
mkdir -p /evidence/failing_drive

# Run all stages with verbose output
sudo safecopy -l 3 /dev/sda /evidence/failing_drive/image.raw

# Check statistics
grep "STATS" /evidence/failing_drive/image.raw.log
```

#### Scenario 2: Recover Specific Partition

```bash
# Calculate partition offset and size
# Example: Partition starts at sector 2048, size 100GB
OFFSET=$((2048 * 512))  # Offset in bytes
SIZE=$((100 * 1024 * 1024 * 1024))  # 100GB in bytes

# Recover just that partition
sudo safecopy -o $OFFSET -l $SIZE /dev/sda /evidence/partition.raw
```

#### Scenario 3: Resume Interrupted Recovery

```bash
# If safecopy is interrupted, it can resume
# Simply run the same command again
sudo safecopy /dev/sda /evidence/image.raw

# Safecopy will skip already-recovered sectors
```

#### Scenario 4: Debugging I/O Errors

```bash
# Run with maximum verbosity
sudo safecopy -l 3 -x /dev/sda /evidence/debug_image.raw

# Analyze the log
cat /evidence/debug_image.raw.log | grep -i "error\|fail\|skip"
```

### Analyzing Results

```bash
# Check recovered file size
ls -lh /evidence/image.raw

# Analyze log for error patterns
cat /evidence/image.raw.log | grep "\[ERROR\]" | wc -l

# Find sectors with most errors
cat /evidence/image.raw.log | grep "\[ERROR\]" | awk '{print $4}' | sort -n

# Check recovery percentage
grep "Stage" /evidence/image.raw.log
```

---

## Forensics Workflow Integration

### Phase 1: Evidence Preparation

```bash
# 1. Document the original evidence
sudo fdisk -l /dev/sda > /evidence/fdisk_output.txt 2>&1
sudo smartctl -a /dev/sda > /evidence/smart_output.txt 2>&1

# 2. Create write blocker (if possible)
sudo blockdev --setro /dev/sda  # Software write blocker

# 3. Document S.M.A.R.T. data
sudo smartctl -A /dev/sda > /evidence/smart_attributes.txt
```

### Phase 2: Image Creation with Safecopy

```bash
# 1. Create output directory structure
mkdir -p /evidence/{images,logs,analysis}

# 2. Run safecopy with documentation
sudo safecopy -l 3 /dev/sda /evidence/images/original.raw \
    2>&1 | tee /evidence/logs/safecopy_output.txt

# 3. Verify image creation
ls -lh /evidence/images/original.raw

# 4. Check for bad sectors
grep "\[ERROR\]" /evidence/images/original.raw.log | wc -l
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
echo "Safecopy Recovery Report" > /evidence/analysis/report.txt
echo "Date: $(date)" >> /evidence/analysis/report.txt
echo "Source: /dev/sda" >> /evidence/analysis/report.txt
echo "Image: /evidence/images/original.raw" >> /evidence/analysis/report.txt
echo "Bad sectors: $(grep '\[ERROR\]' /evidence/images/original.raw.log | wc -l)" >> /evidence/analysis/report.txt
```

### Integration with Other Tools

```bash
# Safecopy + TestDisk
# After creating image, use TestDisk for partition recovery
sudo testdisk /evidence/images/original.raw

# Safecopy + PhotoRec
# After creating image, use PhotoRec for file carving
sudo photorec /evidence/images/original.raw

# Safecopy + The Sleuth Kit
# Analyze image with TSK tools
fls -r -d /evidence/images/original.raw
icat /evidence/images/original.raw inode_number > recovered_file

# Safecopy + Autopsy
# Import image into Autopsy for comprehensive analysis
```

---

## Performance Optimization

### Block Size Optimization

```bash
# Default block size (512 bytes)
safecopy /dev/sda /evidence/image.raw

# Larger block size for faster reading of good sectors
safecopy -b 4096 /dev/sda /evidence/image.raw

# Smaller block size for more precise error recovery
safecopy -b 256 /dev/sda /evidence/image.raw
```

### Parallel Recovery

```bash
# For multiple devices, run in parallel
sudo safecopy /dev/sda /evidence/sda.raw &
sudo safecopy /dev/sdb /evidence/sdb.raw &
wait
```

### Using Faster Storage

```bash
# Write to SSD for faster I/O
sudo safecopy /dev/sda /ssd/evidence/image.raw

# Then copy to long-term storage
cp /ssd/evidence/image.raw /nas/evidence/image.raw
```

### Monitoring Progress

```bash
# Monitor safecopy progress
watch -n 5 'ls -lh /evidence/image.raw'

# Monitor system I/O
iostat -x 1

# Check for errors in real-time
tail -f /evidence/image.raw.log | grep -i "error"
```

---

## Limitations and Considerations

### What Safecopy Cannot Do

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
# Monitor for Safecopy usage
which safecopy
sudo ausearch -k safecopy -ts recent

# Check for unusual disk imaging activity
sudo iotop
sudo iostat -x 1

# Check for image files in unusual locations
find /tmp /var/tmp /home -name "*.raw" -o -name "*.dd" -mtime -1

# Monitor command history
cat ~/.bash_history | grep -i "safecopy\|dd\|dcfldd"
```

### For Forensic Investigators

```bash
# Check if Safecopy was used as anti-forensics tool
# Look for evidence of disk imaging
find / -name "*.raw" -o -name "*.dd" -o -name "*.img" 2>/dev/null

# Check for command history
cat /root/.bash_history | grep -i "safecopy\|dd"

# Examine system logs
sudo journalctl | grep -i "safecopy\|block\|disk"

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
sudo safecopy /dev/sda /evidence/image.raw

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
# Safecopy will continue past errors
# Check the log for recovery statistics
```

#### Recovery seems stuck

```bash
# Check if safecopy is still running
ps aux | grep safecopy

# Check I/O activity
iostat -x 1

# If truly stuck, try with different block size
sudo safecopy -b 4096 /dev/sda /evidence/image.raw
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
2. **Run all three stages**: Maximum recovery requires all stages
3. **Be patient**: Recovery can take hours or days
4. **Check results**: Verify recovered data integrity
5. **Have a backup plan**: If Safecopy fails, try other tools

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

### Safecopy vs dd vs ddrescue

| Feature | Safecopy | dd | ddrescue |
|---------|----------|-----|----------|
| Error handling | Excellent | Poor | Excellent |
| Bad sectors | Graceful skip | Abort | Graceful skip |
| Resume capability | Yes | No | Yes |
| Progress reporting | Good | Basic | Excellent |
| Speed | Medium | Fast | Fast |
| Learning curve | Low | Low | Medium |
| Forensics use | Yes | Yes | Yes |

### When to Choose Safecopy

- When you need maximum recovery from damaged media
- When dd fails due to I/O errors
- When you need a simple, reliable tool
- When working with optical media (CDs/DVDs)
- When you need resume capability

---

## References

- **Safecopy Documentation**: https://safecopy.sourceforge.net/
- **Safecopy GitHub**: https://github.com/claunia/safecopy
- **Kali Linux Package**: https://www.kali.org/tools/safecopy/
- **Data Recovery Guide**: https://www.sans.org/reading-room/data-recovery-techniques-1543
- **Digital Forensics Framework**: https://www.digitalforensicsframework.org/
- **NIST SP 800-86**: Guide to Integrating Forensic Techniques into Incident Response

---

## Quick Reference Card

| Operation | Command |
|-----------|---------|
| Basic recovery | `safecopy /dev/sda image.raw` |
| Stage 1 only | `safecopy -s /dev/sda image.raw` |
| Stage 2 only | `safecopy -S /dev/sda image.raw` |
| Stage 3 only | `safecopy -U /dev/sda image.raw` |
| With block size | `safecopy -b 4096 /dev/sda image.raw` |
| From optical media | `safecopy /dev/cdrom cd_image.raw` |
| From USB drive | `safecopy /dev/sdb usb_image.raw` |
| Resume interrupted | Run same command again |
| Verbose output | `safecopy -l 3 /dev/sda image.raw` |

---

*Document Version: 1.0*
*Last Updated: 2026*
*Author: Digital Forensics Documentation Project*
*Classification: Educational*
