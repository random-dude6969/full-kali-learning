# istat — Display Inode Information

## Table of Contents

1. [Overview](#overview)
2. [Understanding Inodes](#understanding-inodes)
3. [Installation](#installation)
4. [Command Syntax](#command-syntax)
5. [Options Reference](#options-reference)
6. [Understanding the Output](#understanding-the-output)
7. [Inodes Across Filesystems](#inodes-across-filesystems)
8. [Practical Lab Exercises](#practical-lab-exercises)
9. [Integration with Other TSK Tools](#integration-with-other-tsk-tools)
10. [Common Forensic Scenarios](#common-forensic-scenarios)
11. [Troubleshooting](#troubleshooting)
12. [Best Practices](#best-practices)
13. [Summary](#summary)

---

## Overview

`istat` is a command-line utility that is part of **The Sleuth Kit (TSK)**. Its purpose is to display detailed information about a specific **inode** (index node) or metadata structure within a filesystem.

An inode is the fundamental data structure that stores all the metadata about a file or directory — except for the filename itself and the actual file content. When you need to examine a specific file's metadata in detail — its permissions, ownership, timestamps, size, and block addresses — `istat` is the tool to use.

In digital forensics, `istat` is invaluable for:

- Examining the metadata of a specific file of interest
- Understanding file allocation status
- Determining which blocks a file occupies on disk
- Verifying timestamps
- Recovering deleted files by examining unallocated inodes
- Understanding NTFS MFT entries

---

## Understanding Inodes

### What Is an Inode?

An **inode** (index node) is a data structure on a filesystem that stores metadata about a file or directory. On Unix/Linux filesystems (ext2/3/4, UFS, etc.), every file has exactly one inode that contains:

- File type (regular file, directory, symlink, etc.)
- Permissions (read, write, execute for owner/group/other)
- Owner UID and group GID
- File size
- Timestamps (accessed, modified, changed, created)
- Link count
- Pointers to data blocks on disk

### What an Inode Does NOT Contain

- The filename (stored in directory entries)
- The actual file content (stored in data blocks referenced by the inode)

This is why a file can have multiple names (hard links) pointing to the same inode, and why you can recover a file's content even after its directory entry is deleted — as long as the inode still exists.

### NTFS MFT Entries

On NTFS filesystems, the equivalent of an inode is the **MFT (Master File Table) entry**. Each MFT entry contains:

- `$STANDARD_INFORMATION` — timestamps, permissions, file attributes
- `$FILE_NAME` — filename and additional timestamps
- `$DATA` — file content (inline or as extent runs)

`istat` works with both Unix inodes and NTFS MFT entries.

### Inode Numbers

- On ext2/3/4: inode numbers are sequential (1, 2, 3, ...)
- On NTFS: MFT entry numbers (0, 1, 2, ...) where entry 0 is the MFT itself
- Inode 0/1 are typically reserved for filesystem metadata
- The root directory is typically inode 2 on ext2/3/4

---

## Installation

```bash
sudo apt install sleuthkit
```

Verify:

```bash
istat -V
```

---

## Command Syntax

```bash
istat [options] image inum
```

The `image` is the disk image file, and `inum` is the inode number to examine.

### Basic Usage

```bash
# Display inode 12 from an NTFS partition
istat -f ntfs -o 63 disk.img 12

# Display inode 2 (root directory) from an ext4 partition
istat -f ext4 -o 2048 disk.img 2

# Display with verbose output
istat -v -f ntfs -o 63 disk.img 12
```

---

## Options Reference

| Option | Description |
|--------|-------------|
| `-f fstype` | Filesystem type (use `-f list` for supported types) |
| `-i imgtype` | Image file format (use `-i list` for supported formats) |
| `-b dev_sector_size` | Device sector size in bytes |
| `-o imgoffset` | Offset to filesystem in image (in sectors) |
| `-N num` | Force display of NUM address of block pointers |
| `-r` | Display run list instead of block address list |
| `-z zone` | Timezone of original machine (e.g., EST5EDT, GMT) |
| `-s seconds` | Time skew of original machine (in seconds) |
| `-P pooltype` | Pool container type |
| `-B pool_volume_block` | Starting block for pool volumes |
| `-S snap_id` | Snapshot ID (for APFS only) |
| `-k password` | Decryption password for encrypted volumes |
| `-v` | Verbose output to stderr |
| `-V` | Print version |

### The `-N` Option

For NTFS, `istat` can display different attribute types within an MFT entry. The `-N` option forces display of a specific attribute:

```bash
# Display attribute #3 of MFT entry 12
istat -f ntfs -o 63 -N 3 disk.img 12
```

### The `-r` Option (Run List)

Instead of listing individual block addresses, display the run list (compressed extent representation):

```bash
istat -f ntfs -o 63 -r disk.img 12
```

This is useful for understanding NTFS sparse files and compressed data.

### The `-z` and `-s` Options

Adjust timestamps for the original system's timezone or time skew:

```bash
# Original system was in Eastern Time
istat -f ntfs -o 63 -z EST5EDT disk.img 12

# Original system clock was 3600 seconds fast
istat -f ntfs -o 63 -s -3600 disk.img 12
```

---

## Understanding the Output

### Standard Inode Output

```
Directory Entry: 12
Allocated
Type:    File
Size:    512
Permissions: r/rrw-r--r--
UID:     1000
GID:     1000
Links:   1

Directory Entry: 12
Allocated
Type:    File
Size:    512
Permissions: r/rrw-r--r--
UID:     1000
GID:     1000
Links:   1

Timestamps:
Accessed:     2024-01-15 10:30:00.000000000
File Modified: 2024-01-15 10:30:00.000000000
Inode Modified: 2024-01-15 10:30:00.000000000
Deleted:       1969-12-31 19:00:00.000000000

Block Addresses:
  2048
```

### Output Fields Explained

| Field | Description |
|-------|-------------|
| **Directory Entry** | Inode/MFT entry number |
| **Allocated/Unallocated** | Whether the inode is in use |
| **Type** | File type (File, Directory, etc.) |
| **Size** | File size in bytes |
| **Permissions** | Unix-style permissions |
| **UID/GID** | Owner and group IDs |
| **Links** | Hard link count |
| **Accessed** | Last access time |
| **File Modified** | Last content modification time |
| **Inode Modified** | Last metadata change time (ctime) |
| **Deleted** | Deletion timestamp (if applicable) |
| **Block Addresses** | Disk blocks containing file data |

### NTFS-Specific Output

For NTFS MFT entries, `istat` shows:

```
MFT Entry: 12
Type:    FILE
Flags:   In Use

$STANDARD_INFORMATION Attribute:
  Created:       2024-01-15 10:00:00.000000000
  Last Modified: 2024-01-15 10:30:00.000000000
  MFT Modified:  2024-01-15 10:30:00.000000000
  Last Access:   2024-01-15 10:30:00.000000000

$FILE_NAME Attribute:
  Created:       2024-01-15 09:55:00.000000000
  Last Modified: 2024-01-15 10:30:00.000000000
  MFT Modified:  2024-01-15 10:30:00.000000000
  Last Access:   2024-01-15 10:30:00.000000000

Attribute:    $DATA
  Size: 512
  Non-Resident
  Run List:
    VCN 0: LCN 2048-2049
```

### Key NTFS Concepts

- **$STANDARD_INFORMATION** — The "real" timestamps used by Windows
- **$FILE_NAME** — Backup timestamps (harder to modify)
- **$DATA** — File content attribute
- **Run List** — Compressed representation of disk blocks
- **LCN** — Logical Cluster Number (block address)
- **VCN** — Virtual Cluster Number (offset within the file)

---

## Inodes Across Filesystems

### ext2/3/4

```bash
istat -f ext4 -o 2048 disk.img 12
```

- Inodes are numbered sequentially (2, 3, 4, ...)
- Inode 2 is the root directory
- Inodes 1-10 are typically reserved

### NTFS

```bash
istat -f ntfs -o 63 disk.img 12
```

- MFT entry 0 is the MFT itself
- MFT entry 5 is the root directory
- Entries 16-31 are reserved system files
- Custom file entries start at 32+

### FAT

```bash
istat -f fat -o 63 disk.img 12
```

- FAT doesn't use traditional inodes
- TSK creates synthetic inode entries for compatibility
- Timestamps are less precise (2-second resolution)

### UFS (BSD)

```bash
istat -f ufs -o 63 disk.img 12
```

- Similar to ext inodes
- UFS1 and UFS2 have slightly different layouts

---

## Practical Lab Exercises

### Lab 1: Examining a Specific File

```bash
# First, list files to find the inode number
fls -f ntfs -o 63 disk.img

# Output might show:
# r/r 12: document.txt

# Now examine that inode
istat -f ntfs -o 63 disk.img 12
```

### Lab 2: Comparing Allocated vs Unallocated Inodes

```bash
# List all inodes
ils -f ext4 -o 2048 disk.img > inodes.txt

# Examine an allocated inode
istat -f ext4 -o 2048 disk.img 12

# Examine an unallocated inode (deleted file)
istat -f ext4 -o 2048 disk.img 15
```

### Lab 3: NTFS Timestamp Analysis

```bash
# Examine an NTFS MFT entry
istat -f ntfs -o 63 disk.img 32

# Compare $STANDARD_INFORMATION vs $FILE_NAME timestamps
# $SI timestamps are what Windows modifies
# $FN timestamps are harder to alter (anti-tampering)
```

### Lab 4: Finding the Root Directory Inode

```bash
# ext2/3/4: root is inode 2
istat -f ext4 -o 2048 disk.img 2

# NTFS: root is MFT entry 5
istat -f ntfs -o 63 disk.img 5
```

### Lab 5: Examining Deleted File Inodes

```bash
# Find deleted files
fls -f ntfs -o 63 -d disk.img

# Examine a deleted file's inode
# The inode may still contain valid data
istat -f ntfs -o 63 disk.img [deleted_inode]

# Extract the file content using icat
icat -f ntfs -o 63 disk.img [deleted_inode] > recovered_file.dat
```

---

## Integration with Other TSK Tools

### Workflow: Finding and Examining a File

```bash
# Step 1: Identify partition
mmls disk.img

# Step 2: List files and find inode
fls -f ntfs -o 63 -r disk.img
# Output: r/r 12: document.txt

# Step 3: Examine the inode
istat -f ntfs -o 63 disk.img 12

# Step 4: Extract the file
icat -f ntfs -o 63 disk.img 12 > document.txt

# Step 5: Check for other references
ffind -f ntfs -o 63 disk.img 12
```

### Using `istat` with `ils`

```bash
# List all inodes to find interesting ones
ils -f ntfs -o 63 -e disk.img

# Examine specific inodes from the list
istat -f ntfs -o 63 disk.img 32
istat -f ntfs -o 63 disk.img 64
```

### Using `istat` with `icat`

```bash
# istat tells you WHERE the data is
istat -f ext4 -o 2048 disk.img 12
# Shows block addresses: 2048, 2049, 2050

# icat extracts the data using that information
icat -f ext4 -o 2048 disk.img 12 > file_content
```

### Using `istat` with `ffind`

```bash
# Find the filename for an inode
ffind -f ntfs -o 63 disk.img 12

# Or find all references (including deleted entries)
ffind -a -f ntfs -o 63 disk.img 12
```

---

## Common Forensic Scenarios

### Scenario 1: Recovering a Deleted File

```bash
# Find the inode of the deleted file
ils -e -f ntfs -o 63 disk.img | grep "12:"

# Examine the inode
istat -f ntfs -o 63 disk.img 12

# If inode is unallocated but has block addresses, extract it
icat -f ntfs -o 63 disk.img 12 > recovered.exe
```

### Scenario 2: Verifying File Integrity

```bash
# Check if timestamps match expected values
istat -f ntfs -o 63 disk.img 12

# Compare $SI and $FN timestamps
# Significant differences may indicate tampering
```

### Scenario 3: Examining System Files

```bash
# Examine the MFT itself (inode 0 on NTFS)
istat -f ntfs -o 63 disk.img 0

# Examine the root directory (inode 5 on NTFS)
istat -f ntfs -o 63 disk.img 5

# Examine the $LogFile (inode 6 on NTFS)
istat -f ntfs -o 63 disk.img 6
```

### Scenario 4: Analyzing Suspicious Executables

```bash
# Find the executable's inode from file listing
fls -r -m / disk.img | grep -i ".exe"

# Examine the inode for creation time and size
istat -f ntfs -o 63 disk.img [inode]

# Extract for malware analysis
icat -f ntfs -o 63 disk.img [inode] > suspect.exe
```

---

## Troubleshooting

### "Inode not found"

- The inode number may be incorrect
- Verify the inode exists with `ils` or `fls`
- Check if you're using the correct filesystem offset

### "Cannot determine filesystem type"

```bash
# Try specifying the filesystem explicitly
istat -f ntfs -o 63 disk.img 12
istat -f ext4 -o 2048 disk.img 12
```

### "Bad block address"

The inode may reference corrupted or unreadable blocks. This can indicate:
- Filesystem corruption
- Incomplete deletion
- Deliberate data destruction

### Timestamps show 1969 or far future

This usually means the timestamp is zero or corrupted. Check:
- Is the inode actually allocated?
- Is the filesystem damaged?
- Is this a synthetic entry?

---

## Best Practices

1. **Always specify the filesystem type and offset** — don't rely on auto-detection
2. **Check both allocated and unallocated inodes** — deleted files may still have valid metadata
3. **Compare $SI and $FN timestamps on NTFS** — discrepancies indicate tampering
4. **Use `-z` to set the correct timezone** — timestamps are meaningless without context
5. **Document inode numbers** — keep a mapping of inodes to filenames
6. **Use `istat` before `icat`** — understand the inode before extracting data
7. **Check link count** — zero link count means the file has no directory entry (deleted)

---

## Summary

`istat` is essential for examining individual file metadata in detail. Key capabilities:

- Displays complete inode/MFT entry information
- Shows timestamps, permissions, ownership, and block addresses
- Supports all major filesystem types (NTFS, ext, FAT, UFS)
- Works with both allocated and unallocated inodes
- Provides NTFS-specific attribute display

### Quick Reference

```bash
# Examine an inode
istat -f ntfs -o 63 disk.img 12

# With timezone
istat -f ext4 -o 2048 -z EST5EDT disk.img 12

# Show run list (NTFS)
istat -f ntfs -o 63 -r disk.img 12

# Verbose output
istat -v -f ntfs -o 63 disk.img 12
```

---

## References

- The Sleuth Kit Documentation: https://www.sleuthkit.org/sleuthkit/man/
- Brian Carrier, "File System Forensic Analysis," Addison-Wesley, 2005.
- Microsoft NTFS Documentation: https://docs.microsoft.com/en-us/windows/win32/fileio/ntfs-technical-reference
