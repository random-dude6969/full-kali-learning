# TestDisk — Disk and Partition Recovery Tool

## Table of Contents

- [Overview](#overview)
- [Installation](#installation)
- [Core Concepts](#core-concepts)
- [Command-Line Reference](#command-line-reference)
- [Interactive Mode Usage](#interactive-mode-usage)
- [Partition Recovery](#partition-recovery)
- [File System Repair](#file-system-repair)
- [Advanced Use Cases](#advanced-use-cases)
- [Forensics Workflow Integration](#forensics-workflow-integration)
- [Limitations and Considerations](#limitations-and-considerations)
- [Detection and Defense](#detection-and-defense)
- [Troubleshooting](#troubleshooting)
- [Best Practices](#best-practices)
- [References](#references)

---

## Overview

TestDisk is a powerful free data recovery software originally designed to help recover lost partitions and make non-booting disks bootable again. Developed by Christophe Grenier, TestDisk is included in the Kali Linux forensics toolkit and is one of the most versatile disk recovery utilities available. Along with its companion tool PhotoRec, TestDisk provides comprehensive data recovery capabilities that span partition recovery, file system repair, and undeletion of files from common file systems.

TestDisk operates at a low level, directly reading and writing disk structures including partition tables, boot sectors, and file system metadata. This makes it exceptionally powerful for recovery scenarios but also requires careful handling to avoid further data loss. The tool supports a wide range of file systems including NTFS, FAT12/FAT16/FAT32, ext2/ext3/ext4, HFS+, and many others.

### Key Features

- **Partition Recovery**: Recover deleted or lost partitions from partition tables
- **File System Repair**: Fix corrupted boot sectors and file system structures
- **Disk Wipe**: Securely wipe disks to prevent data recovery
- **Non-Destructive**: Default operations are read-only until you confirm writes
- **Cross-Platform**: Works on Windows, Linux, macOS, and other Unix-like systems
- **Supports Major File Systems**: NTFS, FAT, ext2/3/4, HFS+, ReiserFS, JFS, and more

### When to Use TestDisk

| Scenario | Use TestDisk? | Notes |
|----------|--------------|-------|
| Lost partitions after OS reinstall | Yes | Primary use case |
| Corrupted partition table | Yes | Can rebuild from backup |
| Accidentally formatted drive | Partial | Better for partition-level recovery |
| Deleted individual files | No | Use PhotoRec instead |
| Corrupted boot sector | Yes | Can restore from backup |
| Damaged disk (bad sectors) | No | Use ddrescue or safecopy first |

---

## Installation

### Kali Linux

```bash
# TestDisk is pre-installed on Kali Linux
sudo apt update
sudo apt install testdisk

# Verify installation
testdisk --version
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
sudo apt install build-essential libncurses-dev libjpeg-dev libpng-dev

# Download and build
wget https://www.cgsecurity.org/testdisk-7.2.tar.bz2
tar xjf testdisk-7.2.tar.bz2
cd testdisk-7.2
./configure --enable-efidebian --prefix=/usr
make -j$(nproc)
sudo make install
```

---

## Core Concepts

### Partition Tables

TestDisk works primarily with partition tables, which are data structures on a disk that describe how the disk is divided into partitions. There are two main partition table types:

**MBR (Master Boot Record)**:
- Located in the first sector of the disk (sector 0)
- Supports up to 4 primary partitions
- Maximum disk size of 2 TB
- Uses 32-bit entries

**GPT (GUID Partition Table)**:
- Modern standard replacing MBR
- Supports up to 128 partitions
- Supports disks larger than 2 TB
- Uses GUIDs for partition identification
- Includes backup header at end of disk

### File System Structures

TestDisk understands the internal structures of many file systems:

- **Boot Sector/Volume Boot Record**: First sector of each partition containing file system parameters
- **Superblock (ext2/3/4)**: Contains file system metadata and allocation information
- **Master File Table (NTFS)**: Database of all files and directories
- **File Allocation Table (FAT)**: Cluster chain mapping for FAT file systems

### Recovery Principles

1. **Write operations are destructive**: Always work on disk images when possible
2. **Backup before repair**: Create a sector-by-sector backup before any modifications
3. **Verify before writing**: TestDisk shows what changes it will make before applying them
4. **Order matters**: Fix partition tables before attempting file system repair

---

## Command-Line Reference

### Basic Syntax

```bash
testdisk [options] [device]
```

### Options

| Option | Description |
|--------|-------------|
| `-d`, `--dumb` | Disable interactive mode (not recommended for recovery) |
| `-b`, `--batch` | Run in batch mode for scripting |
| `-c`, `--color` | Enable colored output (default on terminals) |
| `-p` | Start in partition search mode |
| `-r` | Start in repair mode |
| `-h`, `--help` | Display help message |
| `-v`, `--version` | Display version information |

### Device Specification

```bash
# Entire disk
testdisk /dev/sda

# Specific partition
testdisk /dev/sda1

# Disk image file
testdisk /path/to/disk_image.dd

# Using device by-id (preferred for consistency)
testdisk /dev/disk/by-id/ata-WD_WD10EZEX-08W
```

---

## Interactive Mode Usage

### Starting TestDisk

```bash
sudo testdisk /dev/sda
```

### Navigation

TestDisk uses a text-based interactive interface:

- **Arrow keys**: Navigate menus
- **Enter**: Select option
- **Arrow Right**: Go back / Next step
- **Arrow Left**: Previous step
- **Quit**: Exit TestDisk (safely returns to shell)

### Step-by-Step Recovery Process

#### Step 1: Select Disk

```
TestDisk 7.2, Data Recovery Utility
Disk /dev/sda - 500 GB / 465 GiB

  [Proceed]  [  Quit  ]

>Select disk: /dev/sda
```

Select the disk you want to recover. Use arrow keys to highlight and Enter to proceed.

#### Step 2: Select Partition Table Type

```
Please select the partition table type:

  Intel/PC partition
  Mac partition map
  EFI GPT partition map
  [Nothing] No partition table type

>[Intel/PC partition]
```

TestDisk usually auto-detects the correct type. For most cases, use Intel/PC (MBR) or EFI GPT.

#### Step 3: Analyze Current Partition Structure

```
Analyse disk /dev/sda - 500 GB / 465 GiB
Geometry : 60801 cylinders / 255 heads / 63 sectors per sector

Disk /dev/sda - 500 GB / 465 GiB

  Partition                  Start        End    Size in sectors
  >P Linux                    2048   1026047    1024000
   P Linux               1026048  976771071  975745024
```

#### Step 4: Search for Lost Partitions

```
[Analyse]  [Advanced]  [Geometry]  [Options]  [Backup]

>Analysis
```

Select "Analyse" to search for lost partitions. TestDisk will scan the disk surface.

#### Step 5: Quick Search

```
Disk /dev/sda - 500 GB / 465 GiB

  Partition                  Start        End    Size in sectors
  D Linux                    2048   1026047    1024000     HPFS/NTFS
   D Linux               1026048  976771071  975745024  Linux

  [Deeper Search]  [Write]
```

- **D** = Deleted (candidate for recovery)
- Use "Deeper Search" if partitions aren't found

#### Step 6: Write Changes

```
Write partition table? (this might destroy data)
>[Yes]  [ No ]

Confirm to write:
  Linux                    2048   1026047    1024000
  Linux               1026048  976771071  975745024
```

**WARNING**: Writing changes modifies the disk directly. Always work on images when possible.

---

## Partition Recovery

### Scenario 1: Recovering a Deleted Partition

```bash
# Step 1: Create a disk image first (critical for forensics)
sudo dd if=/dev/sda of=/tmp/disk_image.dd bs=4M status=progress

# Step 2: Work on the image
sudo testdisk /tmp/disk_image.dd

# Step 3: Follow interactive prompts to find and recover partition
```

### Scenario 2: Fixing a Corrupted Partition Table

```bash
# Start TestDisk
sudo testdisk /dev/sda

# Select Advanced mode
# Choose the damaged partition
# Select boot sector recovery
# Compare boot sector with backup
# Restore from backup if corrupted
```

### Scenario 3: Rebuilding GPT from Backup

```bash
# TestDisk can rebuild GPT from the backup header at the end of disk
sudo testdisk /dev/sda

# Select EFI GPT partition map
# Analyse will find the backup GPT header
# Review and write the recovered table
```

---

## File System Repair

### NTFS Boot Sector Repair

```bash
sudo testdisk /dev/sda1

# Select Advanced
# Choose the NTFS partition
# Select "Boot" to repair boot sector
# Options:
#   - Restore boot sector from backup
#   - Repair MFT (Master File Table)
#   - Repair $MFT mirror
```

### FAT File System Repair

```bash
sudo testdisk /dev/sda1

# Select Advanced
# Choose the FAT partition
# Select "Boot" to repair
# TestDisk can:
#   - Restore FAT32 backup boot sector
#   - Repair FAT table
#   - Fix cluster chains
```

### ext2/3/4 Superblock Repair

```bash
# TestDisk can compare and repair superblocks
sudo testdisk /dev/sda2

# Select Advanced
# Choose ext4 partition
# Superblock repair options
```

---

## Advanced Use Cases

### Disk Wiping

```bash
# TestDisk includes a secure wipe function
sudo testdisk /dev/sda

# Select "Tools"
# Choose "Wipe" to zero out the disk
# Useful before disposing of drives
```

### PhotoRec Integration

```bash
# TestDisk bundles PhotoRec for file carving
# From within TestDisk:
# Select "Advanced" → Select partition → "PhotoRec"

# Or run PhotoRec directly
sudo photorec /dev/sda1
```

### Working with Disk Images

```bash
# Create a raw disk image
sudo dd if=/dev/sda of=/evidence/disk_image.raw bs=4M status=progress

# Create an E01 (Expert Witness Format) image using dc3dd or Guymager
sudo dc3dd if=/dev/sda hof=/evidence/disk_image.E01 log=/evidence/dc3dd.log

# Analyze the image with TestDisk
sudo testdisk /evidence/disk_image.raw
```

### Scripted Recovery with TestDisk

```bash
#!/bin/bash
# Automated partition search (limited functionality)
# TestDisk is primarily interactive, but some operations can be scripted

DEVICE="/dev/sda"
LOG_FILE="/tmp/testdisk_recovery.log"

echo "Starting TestDisk recovery on $DEVICE" | tee "$LOG_FILE"
echo "Date: $(date)" | tee -a "$LOG_FILE"

# Create backup first
sudo dd if="$DEVICE" of="/tmp/partition_backup.bin" bs=512 count=2048

# Launch TestDisk (interactive)
sudo testdisk "$DEVICE" 2>&1 | tee -a "$LOG_FILE"
```

---

## Forensics Workflow Integration

### Phase 1: Evidence Preservation

```bash
# 1. Document the original state
sudo fdisk -l /dev/sda > /evidence/fdisk_output.txt
sudo parted /dev/sda print > /evidence/parted_output.txt

# 2. Create write blocker (software approach)
sudo blockdev --setro /dev/sda  # Set read-only

# 3. Create bit-for-bit image
sudo dd if=/dev/sda of=/evidence/original.raw bs=4M status=progress

# 4. Verify image integrity
md5sum /dev/sda /evidence/original.raw
sha256sum /dev/sda /evidence/original.raw
```

### Phase 2: Analysis with TestDisk

```bash
# 1. Work only on the image
sudo testdisk /evidence/original.raw

# 2. Document all findings
#    - Screenshot each step
#    - Record partition structures found
#    - Note any deleted partitions

# 3. If recovery needed, create a working copy
cp /evidence/original.raw /evidence/working.raw
sudo testdisk /evidence/working.raw
```

### Phase 3: Recovery and Documentation

```bash
# 1. Recover partition to working image
#    (Use TestDisk interactive mode)

# 2. Mount recovered partitions
sudo mkdir -p /mnt/recovered
sudo mount -o loop,ro /evidence/working.raw@offset /mnt/recovered

# 3. Document recovered files
find /mnt/recovered -type f > /evidence/recovered_files.txt

# 4. Calculate hashes of recovered files
find /mnt/recovered -type f -exec md5sum {} \; > /evidence/file_hashes.txt
```

### Integration with Other Forensic Tools

```bash
# TestDisk + The Sleuth Kit
# After TestDisk recovers partitions, use TSK for file-level analysis
fls -r -d /evidence/recovered.raw

# TestDisk + Autopsy
# Import recovered image into Autopsy for comprehensive analysis

# TestDisk + Volatility
# If memory dump available, correlate partition info with memory artifacts
```

---

## Limitations and Considerations

### What TestDisk Cannot Do

1. **Recover files from formatted drives**: TestDisk works at the partition level, not individual files (use PhotoRec for that)
2. **Recover from physically damaged disks**: Use ddrescue or safecopy first to create an image
3. **Decrypt encrypted volumes**: TestDisk cannot bypass encryption
4. **Recover from RAID without proper configuration**: Requires understanding of RAID layout
5. **Guarantee 100% recovery**: Some data may be overwritten or corrupted

### Known Limitations

- **Interactive-only operation**: Cannot be fully automated for scripted recovery
- **Limited file system support**: Some newer or exotic file systems may not be supported
- **No network recovery**: Cannot recover data from network-attached storage directly
- **Performance**: Deep searches on large disks can take hours
- **GPT limitations**: Some GPT recovery scenarios may not work perfectly

---

## Detection and Defense

### For System Administrators

```bash
# Monitor for unauthorized partition changes
# Install and configure aide (Advanced Intrusion Detection Environment)
sudo apt install aide
sudo aideinit
sudo cp /var/lib/aide/aide.db.new /var/lib/aide/aide.db

# Monitor partition table changes
inotifywait -m /dev/sda -e modify,create,delete

# Regular backup of partition tables
sudo sfdisk -d /dev/sda > /backup/partition_table_backup.txt
```

### For Forensic Investigators

```bash
# Document evidence of partition manipulation
# Check for TestDisk artifacts
# TestDisk may leave traces in:
#   - System logs (dmesg, syslog)
#   - Mount history (/etc/mtab, /proc/mounts)
#   - Command history (bash_history, .bash_history)

# Check for evidence of partition table modification
sudo dmesg | grep -i "partition\|sda\|table"

# Examine system logs for disk operations
sudo journalctl | grep -i "block\|partition\|disk"

# Check command history for recovery tools
cat /root/.bash_history | grep -i "testdisk\|photorec\|dd"
```

### Defensive Measures

```bash
# 1. Enable LVM thin provisioning for easier snapshot-based recovery
# 2. Use GPT with backup headers for automatic recovery
# 3. Regular backup of partition metadata
sudo sfdisk -d /dev/sda > /backup/mbr_backup_$(date +%Y%m%d).txt
sudo sgdisk --backup=/backup/gpt_backup_$(date +%Y%m%d).bin /dev/sda

# 4. Enable disk monitoring
sudo smartctl -a /dev/sda  # Check S.M.A.R.T. data

# 5. Use LUKS encryption for sensitive data
sudo cryptsetup luksFormat /dev/sda2
sudo cryptsetup luksOpen /dev/sda2 encrypted_volume
```

---

## Troubleshooting

### Common Issues and Solutions

#### "No partition found" during search

```bash
# Try deeper search
# In TestDisk: Analyse → Deeper Search

# If still not found, try alternative approaches
# Check if disk is detected at all
sudo fdisk -l /dev/sda
sudo hdparm -I /dev/sda
```

#### "Read error" during scanning

```bash
# Disk may have bad sectors
# First, create image with error handling
sudo ddrescue /dev/sda /tmp/rescue_image.raw /tmp/ddrescue.log

# Then run TestDisk on the image
sudo testdisk /tmp/rescue_image.raw
```

#### TestDisk hangs or crashes

```bash
# Check available memory
free -h

# Try with different options
sudo testdisk -d /dev/sda  # Dumb mode (less interactive)

# Run on a smaller section
sudo testdisk /dev/sda1  # Specific partition instead of whole disk
```

#### Partition recovered but won't mount

```bash
# Check file system integrity
sudo fsck /dev/sda2

# For NTFS
sudo ntfsfix /dev/sda2

# For ext4
sudo e2fsck -f /dev/sda2

# If mount fails, try with different options
sudo mount -o ro,norecovery /dev/sda2 /mnt/recovered
```

---

## Best Practices

### For Forensic Investigations

1. **Always work on images**: Never operate TestDisk on original evidence
2. **Maintain chain of custody**: Document every step of the process
3. **Use write blockers**: Hardware or software write blockers prevent accidental modification
4. **Hash before and after**: Verify evidence integrity at every stage
5. **Take screenshots**: Document the TestDisk interface at each decision point

### For Data Recovery

1. **Stop using the affected drive**: Prevent further data overwrite
2. **Create an image first**: Use ddrescue for damaged drives, dd for healthy ones
3. **Try Quick Search first**: Only use Deeper Search if Quick Search fails
4. **Don't write changes unless necessary**: Confirm before modifying partition tables
5. **Verify recovery**: After writing, check that the recovered partitions mount correctly

### Performance Tips

```bash
# For large disks, use block size optimization
sudo dd if=/dev/sda of=/tmp/image.raw bs=64K status=progress

# Monitor I/O during TestDisk operation
iostat -x 1

# For SSDs, ensure TRIM is disabled during recovery
sudo hdparm -I /dev/sda | grep TRIM
```

---

## References

- **Official Documentation**: https://www.cgsecurity.org/wiki/TestDisk
- **TestDisk GitHub**: https://github.com/cgsecurity/testdisk
- **PhotoRec Documentation**: https://www.cgsecurity.org/wiki/PhotoRec
- **Kali Linux Forensics**: https://www.kali.org/tools/testdisk/
- **SANS Forensics Poster**: https://www.sans.org/posters/windows-forensic-analysis/
- **NIST SP 800-86**: Guide to Integrating Forensic Techniques into Incident Response
- **Digital Forensics Framework**: https://www.digitalforensicsframework.org/

---

## Quick Reference Card

| Operation | Command |
|-----------|---------|
| Start on disk | `sudo testdisk /dev/sda` |
| Start on image | `sudo testdisk /path/to/image.raw` |
| Start on partition | `sudo testdisk /dev/sda1` |
| Quick search | Analyse → Quick Search |
| Deep search | Analyse → Deeper Search |
| Write changes | Select "Write" → Confirm "Yes" |
| Repair boot | Advanced → Select partition → Boot |
| Launch PhotoRec | Advanced → Select partition → PhotoRec |
| Backup partition table | `sudo sfdisk -d /dev/sda > backup.txt` |
| Restore partition table | `sudo sfdisk /dev/sda < backup.txt` |

---

*Document Version: 1.0*
*Last Updated: 2026*
*Author: Digital Forensics Documentation Project*
*Classification: Educational*
