# blkstat — Display Block Statistics

## Table of Contents

1. [Overview](#overview)
2. [Understanding Disk Blocks](#understanding-disk-blocks)
3. [Installation](#installation)
4. [Command Syntax](#command-syntax)
5. [Options Reference](#options-reference)
6. [Understanding the Output](#understanding-the-output)
7. [Practical Lab Exercises](#practical-lab-exercises)
8. [Integration with Other TSK Tools](#integration-with-other-tsk-tools)
9. [Common Forensic Scenarios](#common-forensic-scenarios)
10. [Troubleshooting](#troubleshooting)
11. [Best Practices](#best-practices)
12. [Summary](#summary)

---

## Overview

`blkstat` is a command-line utility that is part of **The Sleuth Kit (TSK)**. Its purpose is to display detailed information about a specific **block** (or sector) on a filesystem, including which inode or metadata structure owns that block.

In digital forensics, understanding block-level details is critical for:

- Determining which file a specific block belongs to
- Analyzing unallocated blocks for deleted file fragments
- Understanding filesystem structure and layout
- Verifying file integrity
- Investigating disk-level artifacts

---

## Understanding Disk Blocks

### What Is a Block?

A **block** (also called a **cluster** on NTFS/FAT or a **sector** on raw disks) is the fundamental unit of storage on a disk. Every file's data is stored in one or more blocks, and filesystem metadata (inodes, directory entries, journal) also occupies blocks.

### Block Size

| Filesystem | Typical Block Size |
|------------|-------------------|
| FAT12/16 | 512 bytes – 32 KB |
| FAT32 | 512 bytes – 64 KB |
| NTFS | 512 bytes – 64 KB (default 4 KB) |
| ext2/3/4 | 1024 – 65536 bytes (default 4 KB) |
| HFS+ | 512 bytes – 2048 KB |

### Block Allocation

Blocks can be in one of these states:

- **Allocated**: Belongs to a file or metadata structure
- **Unallocated**: Free for use by the filesystem
- **Slack**: Partially used (file doesn't fill the entire block)

### Block Addressing

Blocks are addressed sequentially starting from 0:

```
Block 0: Boot sector / superblock
Block 1-N: Metadata blocks (inodes, bitmaps, journal)
Block N+: Data blocks (file contents)
```

---

## Installation

```bash
sudo apt install sleuthkit
```

---

## Command Syntax

```bash
blkstat [options] image addr
```

The `addr` is the block number to examine.

### Basic Usage

```bash
# Display info for block 2048
blkstat -f ext4 -o 2048 disk.img 2048

# Display with NTFS
blkstat -f ntfs -o 63 disk.img 1000
```

---

## Options Reference

| Option | Description |
|--------|-------------|
| `-f fstype` | Filesystem type (use `-f list` for supported types) |
| `-i imgtype` | Image file format (use `-i list` for supported formats) |
| `-b dev_sector_size` | Device sector size in bytes |
| `-o imgoffset` | Offset to filesystem in sectors |
| `-P pooltype` | Pool container type |
| `-B pool_volume_block` | Starting block for pool volumes |
| `-k password` | Decryption password for encrypted volumes |
| `-v` | Verbose output |
| `-V` | Print version |

---

## Understanding the Output

### Standard Output

```
Block Size: 4096
Block: 2048
Allocated
Inode: 12
```

### Output Fields

| Field | Description |
|-------|-------------|
| **Block Size** | Size of each block in bytes |
| **Block** | Block number being examined |
| **Allocated/Unallocated** | Whether the block is in use |
| **Inode** | The inode that owns this block (if applicable) |

### On NTFS

```
Block Size: 4096
Block: 2048
Allocated
MFT Entry: 32
Attribute: $DATA
```

### What the Output Tells You

- **Which file owns the block** — if an inode number is shown, the block belongs to that file
- **Allocation status** — determines if the block is free or in use
- **Metadata association** — for NTFS, shows which MFT entry and attribute

---

## Practical Lab Exercises

### Lab 1: Basic Block Examination

```bash
# Find out which file owns block 2048
blkstat -f ext4 -o 2048 disk.img 2048

# Examine multiple blocks
for block in $(seq 2048 2058); do
    echo "=== Block $block ==="
    blkstat -f ext4 -o 2048 disk.img $block
done
```

### Lab 2: Analyzing Unallocated Blocks

```bash
# Find unallocated blocks
blkls -A -f ext4 -o 2048 disk.img > unallocated_blocks.bin

# Check specific unallocated blocks
blkstat -f ext4 -o 2048 disk.img 5000
blkstat -f ext4 -o 2048 disk.img 5001
```

### Lab 3: Tracing File Blocks

```bash
# First, find a file's inode
fls -f ntfs -o 63 disk.img

# Then examine its inode to see block addresses
istat -f ntfs -o 63 disk.img 12

# Use blkstat to verify each block
blkstat -f ntfs -o 63 disk.img 2048
blkstat -f ntfs -o 63 disk.img 2049
```

### Lab 4: NTFS Metadata Blocks

```bash
# Examine MFT blocks
blkstat -f ntfs -o 63 disk.img 0    # MFT itself
blkstat -f ntfs -o 63 disk.img 1    # MFT mirror
blkstat -f ntfs -o 63 disk.img 2    # Log file
```

---

## Integration with Other TSK Tools

### Complete Block Analysis Workflow

```bash
# Step 1: Identify partition
mmls disk.img

# Step 2: Examine a specific block
blkstat -f ext4 -o 2048 disk.img 3000

# Step 3: If allocated, check which inode owns it
# (shown in blkstat output)

# Step 4: Examine that inode
istat -f ext4 -o 2048 disk.img 12

# Step 5: Extract the file
icat -f ext4 -o 2048 disk.img 12 > file_content
```

### blkstat + blkcat

```bash
# blkstat tells you about the block
blkstat -f ext4 -o 2048 disk.img 3000

# blkcat shows you the block's contents
blkcat -f ext4 -o 2048 disk.img 3000 > block_content.bin
```

### blkstat + blkls

```bash
# blkls lists unallocated blocks
blkls -A -f ext4 -o 2048 disk.img > unalloc.bin

# Use blkstat to check specific blocks
blkstat -f ext4 -o 2048 disk.img $(blkls -A -f ext4 -o 2048 disk.img | head -1)
```

### blkstat + blkcalc

```bash
# blkcalc converts between block number systems
# If you know a block number from blkls output, convert it
blkcalc -u <blkls_block_number> -f ext4 -o 2048 disk.img
```

---

## Common Forensic Scenarios

### Scenario 1: Recovering Deleted File Fragments

```bash
# Extract unallocated blocks
blkls -A -f ext4 -o 2048 disk.img > unalloc.bin

# Search for file signatures in unallocated blocks
srch_strings unalloc.bin | grep -i "password\|secret\|confidential"

# Check specific blocks
blkstat -f ext4 -o 2048 disk.img 5000
blkcat -f ext4 -o 2048 disk.img 5000 > fragment.bin
```

### Scenario 2: Verifying File Integrity

```bash
# Check all blocks of a file
inode=$(fls -f ntfs -o 63 disk.img | grep "important.doc" | awk -F'[: ]' '{print $2}')
istat -f ntfs -o 63 disk.img $inode

# Verify each block
for block in $(istat -f ntfs -o 63 disk.img $inode | grep "^[0-9]" | awk '{print $1}'); do
    blkstat -f ntfs -o 63 disk.img $block
done
```

### Scenario 3: NTFS Metadata Analysis

```bash
# Examine MFT entry blocks
blkstat -f ntfs -o 63 disk.img 0   # MFT
blkstat -f ntfs -o 63 disk.img 1   # MFT mirror
blkstat -f ntfs -o 63 disk.img 2   # $LogFile
blkstat -f ntfs -o 63 disk.img 3   # $Volume
```

### Scenario 4: Investigating Slack Space

```bash
# Find the last block of a file
istat -f ext4 -o 2048 disk.img 12 | tail -1

# Check that block
blkstat -f ext4 -o 2048 disk.img <last_block>

# The block may contain residual data from previous files
```

---

## Troubleshooting

### "Block not found"

- The block number may be outside the filesystem range
- Check the filesystem size with `fsstat`
- Verify the offset with `mmls`

### "Cannot determine filesystem type"

```bash
# Specify filesystem explicitly
blkstat -f ntfs -o 63 disk.img 1000
blkstat -f ext4 -o 2048 disk.img 1000
```

### Block shows as "Unallocated" but expected allocated

- The block may have been freed by the filesystem
- Check if the file was deleted
- Verify with `ils` to see inode status

---

## Best Practices

1. **Always specify filesystem type and offset** — don't rely on auto-detection
2. **Use blkstat before blkcat** — understand the block before viewing its contents
3. **Chain tools together** — blkstat → istat → icat for file recovery
4. **Check unallocated blocks** — deleted file data often resides here
5. **Document block numbers** — maintain a mapping for your investigation
6. **Note the block size** — it affects how you interpret the data
7. **Verify allocation status** — allocated blocks belong to active files

---

## Summary

`blkstat` provides block-level details about filesystem storage. Key capabilities:

- Identifies which inode owns a specific block
- Shows block allocation status
- Works with all TSK-supported filesystems
- Essential for deep forensic analysis

### Quick Reference

```bash
# Examine a specific block
blkstat -f ext4 -o 2048 disk.img 2048

# NTFS block
blkstat -f ntfs -o 63 disk.img 1000

# Check blocks from unallocated space
blkls -A -f ext4 -o 2048 disk.img | head -5 | while read b; do
    blkstat -f ext4 -o 2048 disk.img $b
done
```

---

## Advanced Block Analysis

### Understanding Block Allocation Bitmaps

Filesystems maintain **bitmaps** that track which blocks are allocated and which are free. Understanding these bitmaps helps in forensic analysis:

```bash
# On ext2/3/4, the block bitmap is stored in block groups
# Each bit represents one block: 0 = free, 1 = allocated

# Examine the superblock for bitmap locations
fsstat -f ext4 -o 2048 disk.img

# Use blkcat to examine bitmap blocks
blkcat -f ext4 -o 2048 disk.img 2  # Block bitmap for block group 0
```

### Block Group Analysis

ext2/3/4 filesystems are divided into **block groups**, each containing:

- Superblock copy
- Group descriptor
- Block bitmap
- Inode bitmap
- Inode table
- Data blocks

```bash
# Get filesystem statistics
fsstat -f ext4 -o 2048 disk.img

# Examine specific block group's bitmap
blkstat -f ext4 -o 2048 disk.img 2  # Block bitmap
blkstat -f ext4 -o 2048 disk.img 3  # Inode bitmap
```

### NTFS Cluster Analysis

On NTFS, blocks are called **clusters** and are tracked differently:

```bash
# NTFS uses a cluster bitmap ($Bitmap)
# Examine cluster allocation
blkstat -f ntfs -o 63 disk.img 0   # Boot sector
blkstat -f ntfs -o 63 disk.img 1   # MFT mirror area
```

### Cross-Referencing Blocks

When investigating a specific block, cross-reference with multiple tools:

```bash
# Step 1: Check block allocation
blkstat -f ext4 -o 2048 disk.img 5000

# Step 2: If allocated, find owning inode
# (shown in blkstat output)

# Step 3: Examine the inode
istat -f ext4 -o 2048 disk.img <inode_number>

# Step 4: Check all blocks of that file
for block in $(istat -f ext4 -o 2048 disk.img <inode_number> | grep "^[0-9]" | awk '{print $1}'); do
    blkstat -f ext4 -o 2048 disk.img $block
done

# Step 5: Extract the file content
icat -f ext4 -o 2048 disk.img <inode_number> > recovered_file
```

### Block Analysis in Disk Images

```bash
# For E01 (EnCase) images
blkstat -i ewf -f ntfs -o 63 evidence.E01 1000

# For AFF images
blkstat -i aff -f ext4 -o 2048 evidence.aff 2048

# For VMDK images
blkstat -i vmdk -f ntfs -o 63 vm_disk.vmdk 500
```

### Block-Level Timeline Analysis

```bash
# Find blocks modified around a specific time
# First, check inode timestamps
istat -f ext4 -o 2048 disk.img 12

# Then examine the blocks
blkstat -f ext4 -o 2048 disk.img 2048
blkstat -f ext4 -o 2048 disk.img 2049
```

### Recovering Data from Specific Blocks

```bash
# If blkstat shows a block is allocated to a deleted inode
# Extract the block content
blkcat -f ext4 -o 2048 disk.img 5000 > block_5000.bin

# Analyze the content
file block_5000.bin
strings block_5000.bin
hexdump -C block_5000.bin | head -50
```

### Block Size Considerations

```bash
# Different block sizes affect analysis
# 1024-byte blocks: Small files, more overhead
# 4096-byte blocks: Standard, good balance
# 65536-byte blocks: Large files, less overhead but more slack

# Check current block size
fsstat -f ext4 -o 2048 disk.img | grep -i "block size"
```

### Fragments and extents

Modern filesystems use **extents** instead of individual block pointers:

```bash
# NTFS uses run lists (extent-based)
istat -f ntfs -o 63 -r disk.img 12

# ext4 uses extents
istat -f ext4 -o 2048 disk.img 12
```

---

## Block Analysis Reference Table

| Scenario | Tool | Command |
|----------|------|---------|
| Find block owner | blkstat | `blkstat -f ext4 -o 2048 disk.img <block>` |
| View block content | blkcat | `blkcat -f ext4 -o 2048 disk.img <block>` |
| List unallocated | blkls | `blkls -A -f ext4 -o 2048 disk.img` |
| List allocated | blkls | `blkls -a -f ext4 -o 2048 disk.img` |
| Extract slack | blkls | `blkls -s -f ext4 -o 2048 disk.img` |
| Convert block numbers | blkcalc | `blkcalc -u <block> -f ext4 -o 2048 disk.img` |
| Filesystem stats | fsstat | `fsstat -f ext4 -o 2048 disk.img` |
| Inode details | istat | `istat -f ext4 -o 2048 disk.img <inode>` |

---

## References

- The Sleuth Kit Documentation: https://www.sleuthkit.org/sleuthkit/man/
- Brian Carrier, "File System Forensic Analysis," Addison-Wesley, 2005.
- NIST SP 800-86: Guide to Integrating Forensic Techniques into Incident Response.
- Kali Linux Sleuth Kit: https://www.kali.org/tools/sleuthkit/
