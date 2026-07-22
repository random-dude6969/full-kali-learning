# blkls — List Block Entries

## Table of Contents

1. [Overview](#overview)
2. [Understanding Block Allocation](#understanding-block-allocation)
3. [Installation](#installation)
4. [Command Syntax](#command-syntax)
5. [Options Reference](#options-reference)
6. [Understanding the Output](#understanding-the-output)
7. [Working with Unallocated Blocks](#working-with-unallocated-blocks)
8. [Slack Space Analysis](#slack-space-analysis)
9. [Practical Lab Exercises](#practical-lab-exercises)
10. [Integration with Other TSK Tools](#integration-with-other-tsk-tools)
11. [Common Forensic Scenarios](#common-forensic-scenarios)
12. [Troubleshooting](#troubleshooting)
13. [Best Practices](#best-practices)
14. [Summary](#summary)

---

## Overview

`blkls` is a command-line utility that is part of **The Sleuth Kit (TSK)**. Its purpose is to **list or output file system data blocks**, allowing you to examine allocated blocks, unallocated blocks, or slack space.

`blkls` is one of the most important tools for deleted file recovery. When a file is deleted from a filesystem, the inode and directory entry are removed, but the actual data blocks often remain on disk until they are overwritten by new data. `blkls` can extract these unallocated blocks, which may contain recoverable evidence.

### Key Use Cases

- Extracting unallocated blocks for deleted file recovery
- Analyzing slack space for residual data
- Creating a "raw" extract of file system data
- Generating body file data for timeline analysis
- Searching for file signatures in unallocated space

---

## Understanding Block Allocation

### Block States

On a filesystem, every block is in one of these states:

**Allocated:**
- Currently in use by a file or metadata structure
- Referenced by an inode or filesystem metadata
- Data is valid and part of an active file

**Unallocated:**
- Previously used but now marked as free
- May still contain data from deleted files
- Filesystem may overwrite these blocks at any time
- **Primary target for deleted file recovery**

**Slack Space:**
- The unused portion of the last block of a file
- For example, if a file is 5000 bytes and blocks are 4096 bytes, the file uses 2 blocks (8192 bytes), but only 5000 bytes are file data. The remaining 3192 bytes is slack space.
- May contain residual data from previously deleted files

### Why Unallocated Blocks Matter

When you delete a file:
1. The directory entry is removed
2. The inode is marked as unallocated
3. The data blocks are marked as free
4. **The actual data remains on disk until overwritten**

This means unallocated blocks are a goldmine for forensic investigators. They may contain:
- Deleted documents, images, emails
- Fragments of files that were partially overwritten
- Credentials, passwords, encryption keys
- Malware artifacts
- Chat logs, browser history

---

## Installation

```bash
sudo apt install sleuthkit
```

---

## Command Syntax

```bash
blkls [options] image [start-stop]
```

The optional `start-stop` parameter allows you to specify a range of blocks to examine.

### Basic Usage

```bash
# Output all blocks to stdout (raw binary)
blkls disk.img

# List only unallocated blocks
blkls -A disk.img

# List only allocated blocks
blkls -a disk.img

# Output in list format with details
blkls -l disk.img

# Show slack space only
blkls -s disk.img
```

**Important:** By default, `blkls` outputs raw block data to stdout. Use the `-l` flag to get a human-readable list format instead.

---

## Options Reference

### Display Modes

| Option | Description |
|--------|-------------|
| (default) | Output raw block data to stdout |
| `-l` | List format with timestamps and details |
| `-e` | Include every block (including filesystem metadata) |

### Allocation Filters

| Option | Description |
|--------|-------------|
| `-a` | Display only **allocated** blocks |
| `-A` | Display only **unallocated** blocks |
| `-s` | Display **slack space** only (ignores other flags) |

### General Options

| Option | Description |
|--------|-------------|
| `-f fstype` | Filesystem type |
| `-i imgtype` | Image file format |
| `-b dev_sector_size` | Device sector size in bytes |
| `-o imgoffset` | Offset to filesystem in sectors |
| `-P pooltype` | Pool container type |
| `-B pool_volume_block` | Starting block for pool volumes |
| `-v` | Verbose output |
| `-V` | Print version |

### The `-e` Flag (Every Block)

The `-e` flag includes filesystem metadata blocks (superblocks, inode tables, bitmaps, journals) in addition to data blocks. This provides a more complete picture of the filesystem:

```bash
blkls -e -l disk.img
```

---

## Understanding the Output

### Raw Output (Default)

By default, `blkls` outputs raw binary data to stdout. This is useful for piping to other tools:

```bash
# Save unallocated blocks to a file
blkls -A disk.img > unallocated.bin

# Search for strings in unallocated blocks
blkls -A disk.img | srch_strings | grep -i "password"

# Search for specific file signatures
blkls -A disk.img | grep -P "PK\x03\x04"  # ZIP files
```

### List Format (`-l`)

```
2048  4096  2024-01-15 10:30:00 2024-01-15 10:30:00 2024-01-15 10:30:00  /home/user/document.txt
2049  4096  2024-01-15 10:30:00 2024-01-15 10:30:00 2024-01-15 10:30:00  /home/user/document.txt
```

**Note:** The list format with timestamps only works on allocated blocks.

### Raw Output for Analysis

```bash
# Extract unallocated blocks for signature analysis
blkls -A disk.img > unalloc.raw

# Use foremost/scalpel for file carving
foremost -i unalloc.raw -o carved_files/

# Or use bulk_extractor
bulk_extractor unalloc.raw -o bulk_output/
```

---

## Working with Unallocated Blocks

### Extracting Unallocated Blocks

```bash
# Save all unallocated blocks
blkls -A disk.img > unallocated.bin

# With filesystem info
blkls -A -f ntfs -o 63 disk.img > ntfs_unalloc.bin
blkls -A -f ext4 -o 2048 disk.img > ext4_unalloc.bin
```

### Searching Unallocated Blocks

```bash
# Search for text strings
blkls -A disk.img | srch_strings -n 8

# Search for specific keywords
blkls -A disk.img | srch_strings | grep -i "password\|secret\|confidential"

# Search for email addresses
blkls -A disk.img | srch_strings | grep -E "[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}"

# Search for URLs
blkls -A disk.img | srch_strings | grep -i "http\|https\|ftp"
```

### File Carving from Unallocated Blocks

```bash
# Extract unallocated blocks
blkls -A disk.img > unalloc.bin

# Use foremost for file carving
foremost -t all -i unalloc.bin -o carved/

# Use scalpel for more targeted carving
scalpel -o scalpel_output/ unalloc.bin

# Use photorec for photo recovery
photorec /cmd unalloc.bin
```

---

## Slack Space Analysis

### What Is Slack Space?

When a file is stored on a filesystem, it occupies one or more full blocks. If the file size isn't a perfect multiple of the block size, the last block will have unused space — this is **slack space**.

```
File size: 5000 bytes
Block size: 4096 bytes
Blocks used: 2 (8192 bytes total)
Slack space: 8192 - 5000 = 3192 bytes
```

### Extracting Slack Space

```bash
# Extract all slack space
blkls -s disk.img > slack.bin

# With filesystem info
blkls -s -f ext4 -o 2048 disk.img > ext4_slack.bin
```

### What Slack Space Contains

Slack space may contain:

- **Previous file data** — data from files that previously occupied those blocks
- **Deleted information** — fragments of deleted files
- **System artifacts** — temporary data left by applications
- **Sensitive data** — passwords, keys, personal information

### Analyzing Slack Space

```bash
# Search for text in slack space
blkls -s disk.img | srch_strings

# Search for specific patterns
blkls -s disk.img | srch_strings | grep -i "password"

# Save for further analysis
blkls -s disk.img > slack_analysis.bin
```

---

## Practical Lab Exercises

### Lab 1: Basic Block Listing

```bash
# List allocated blocks
blkls -a -l -f ext4 -o 2048 disk.img

# List unallocated blocks
blkls -A -l -f ext4 -o 2048 disk.img
```

### Lab 2: Extracting Unallocated Space

```bash
# Create a test scenario
dd if=/dev/zero of=test.img bs=1M count=100
mkfs.ext4 test.img
mkdir /tmp/mnt
sudo mount test.img /tmp/mnt

# Create and delete files
echo "Top secret data" > /tmp/mnt/secret.txt
echo "Confidential report" > /tmp/mnt/report.doc
sudo rm /tmp/mnt/secret.txt
sudo rm /tmp/mnt/report.doc
sudo umount /tmp/mnt

# Extract unallocated blocks
blkls -A -f ext4 -o 2048 test.img > unalloc.bin

# Search for deleted content
srch_strings unalloc.bin
```

### Lab 3: Slack Space Recovery

```bash
# Create a file that doesn't fill its last block
dd if=/dev/zero of=test.img bs=1M count=100
mkfs.ext4 test.img
sudo mount test.img /tmp/mnt
echo "Small file" > /tmp/mnt/small.txt
# small.txt is ~11 bytes, but takes 4096 bytes (1 block)

# Write something else to a different block
dd if=/dev/urandom of=/tmp/mnt/random.bin bs=1M count=10
sudo umount /tmp/mnt

# Extract slack space
blkls -s -f ext4 -o 2048 test.img > slack.bin
srch_strings slack.bin
```

### Lab 4: File Carving Workflow

```bash
# Full workflow: blkls → file carving → analysis

# Step 1: Extract unallocated blocks
blkls -A -f ntfs -o 63 evidence.img > unalloc.bin

# Step 2: Carve files
foremost -t all -i unalloc.bin -o carved/

# Step 3: Analyze carved files
ls -la carved/

# Step 4: Check file types
file carved/*

# Step 5: Review content
strings carved/*
```

### Lab 5: Combining Allocated and Unallocated Analysis

```bash
# Get a complete picture of all blocks
blkls -e -l -f ext4 -o 2048 disk.img > all_blocks.txt

# Compare allocated vs unallocated
echo "Allocated blocks:"
blkls -a -f ext4 -o 2048 disk.img | wc -c

echo "Unallocated blocks:"
blkls -A -f ext4 -o 2048 disk.img | wc -c
```

---

## Integration with Other TSK Tools

### blkls + blkstat

```bash
# List unallocated blocks
blkls -A -f ext4 -o 2048 disk.img > unalloc_blocks.bin

# Check specific unallocated blocks
blkstat -f ext4 -o 2048 disk.img 5000
blkstat -f ext4 -o 2048 disk.img 5001
```

### blkls + blkcat

```bash
# blkls lists blocks, blkcat shows their contents
blkls -A -f ext4 -o 2048 disk.img | head -5 | while read block; do
    echo "=== Block $block ==="
    blkcat -f ext4 -o 2048 disk.img $block | head -20
done
```

### blkls + blkcalc

```bash
# Convert between block number systems
# If blkls shows block 5000
blkcalc -u 5000 -f ext4 -o 2048 disk.img
```

### blkls + mactime

```bash
# Generate body file from allocated blocks
fls -r -m / -f ext4 -o 2048 disk.img > body.txt
mactime -b body.txt -d > timeline.csv

# Timeline of what was deleted
ils -m -A -f ext4 -o 2048 disk.img > deleted_body.txt
mactime -b deleted_body.txt -d > deleted_timeline.csv
```

### blkls + foremost/scalpel

```bash
# Complete file carving workflow
blkls -A -f ntfs -o 63 disk.img > unalloc.bin
foremost -i unalloc.bin -o carved/
```

---

## Common Forensic Scenarios

### Scenario 1: Recovering Deleted Documents

```bash
# Extract unallocated blocks
blkls -A -f ntfs -o 63 disk.img > unalloc.bin

# Search for document signatures
# DOC: D0 CF 11 E0 A1 B1 1A E1
# PDF: 25 50 44 46
# XLS: D0 CF 11 E0 A1 B1 1A E1

# Carve files
foremost -t doc,pdf,xls -i unalloc.bin -o documents/
```

### Scenario 2: Password Recovery

```bash
# Search for password-related strings
blkls -A disk.img | srch_strings | grep -i "password\|passwd\|pwd\|secret\|key"

# Search in slack space too
blkls -s disk.img | srch_strings | grep -i "password"
```

### Scenario 3: Email Recovery

```bash
# Search for email patterns
blkls -A disk.img | srch_strings | grep -E "[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}"

# Search for email headers
blkls -A disk.img | srch_strings | grep -i "From:\|To:\|Subject:\|Received:"
```

### Scenario 4: Browser History Recovery

```bash
# Search for URLs
blkls -A disk.img | srch_strings | grep -i "http\|https\|www\."

# Search for common browser artifacts
blkls -A disk.img | srch_strings | grep -i "google\|facebook\|twitter\|login"
```

### Scenario 5: Complete Disk Analysis

```bash
# Full analysis workflow
# 1. Partition table
mmls disk.img

# 2. Extract unallocated blocks
blkls -A -f ext4 -o 2048 disk.img > unalloc.bin

# 3. Extract slack space
blkls -s -f ext4 -o 2048 disk.img > slack.bin

# 4. File carving
foremost -i unalloc.bin -o carved/
foremost -i slack.bin -o slack_carved/

# 5. Timeline analysis
fls -r -m / -f ext4 -o 2048 disk.img > body.txt
mactime -b body.txt -d > timeline.csv

# 6. String analysis
srch_strings unalloc.bin > strings_unalloc.txt
srch_strings slack.bin > strings_slack.txt
```

---

## Troubleshooting

### Raw Output is Too Large

Use the `-l` flag for list format, or redirect to a file:

```bash
# List format instead of raw
blkls -A -l -f ext4 -o 2048 disk.img

# Or save raw to file
blkls -A -f ext4 -o 2048 disk.img > unalloc.bin
```

### "Cannot determine filesystem type"

Specify the filesystem explicitly:

```bash
blkls -A -f ntfs -o 63 disk.img
blkls -A -f ext4 -o 2048 disk.img
```

### Slack Space Extraction Returns Nothing

- The filesystem may use delayed allocation (ext4)
- Try with `-e` to include metadata blocks
- Verify the filesystem isn't full

### Unallocated Blocks Contain Mostly Zeros

- This is normal on a clean filesystem
- Focus on blocks with actual data
- Use `srch_strings` to filter for meaningful content

---

## Best Practices

1. **Use `-A` for unallocated blocks** — this is where deleted file data lives
2. **Use `-s` for slack space** — often overlooked but valuable
3. **Always use `-f` and `-o`** — don't rely on auto-detection
4. **Combine with file carving tools** — foremost, scalpel, photorec
5. **Search before carving** — strings analysis can quickly identify interesting blocks
6. **Save raw output** — always keep the unallocated block extract
7. **Document your process** — record which blocks you examined and why
8. **Check both unallocated AND slack** — deleted data can be in either
9. **Use `-e` for complete analysis** — includes metadata blocks
10. **Work quickly** — unallocated blocks can be overwritten at any time

---

## Summary

`blkls` is essential for block-level forensic analysis. Key capabilities:

- Lists allocated and unallocated blocks
- Extracts slack space for residual data analysis
- Outputs raw data for file carving tools
- Provides list format for human review
- Supports all TSK-supported filesystems

### Quick Reference

```bash
# Extract unallocated blocks
blkls -A disk.img > unalloc.bin

# Extract slack space
blkls -s disk.img > slack.bin

# List blocks (human-readable)
blkls -A -l disk.img

# All blocks including metadata
blkls -e -A disk.img

# Search for content in unallocated
blkls -A disk.img | srch_strings | grep -i "keyword"

# File carving workflow
blkls -A disk.img > unalloc.bin
foremost -i unalloc.bin -o carved/
```

---

## References

- The Sleuth Kit Documentation: https://www.sleuthkit.org/sleuthkit/man/
- Brian Carrier, "File System Forensic Analysis," Addison-Wesley, 2005.
- NIST SP 800-86: Guide to Integrating Forensic Techniques into Incident Response.
- File Carving Techniques: https://www.uberle.utwente.nl/ds/file_carving/
