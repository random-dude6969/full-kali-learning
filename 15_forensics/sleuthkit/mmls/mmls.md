# mmls — Display Partition Table

## Table of Contents

1. [Overview](#overview)
2. [Understanding Partition Tables](#understanding-partition-tables)
3. [Installation](#installation)
4. [Command Syntax](#command-syntax)
5. [Options Reference](#options-reference)
6. [Understanding the Output](#understanding-the-output)
7. [Supported Volume System Types](#supported-volume-system-types)
8. [Supported Image Types](#supported-image-types)
9. [Common Use Cases](#common-use-cases)
10. [Working with Disk Images](#working-with-disk-images)
11. [Working with Live Disks](#working-with-live-disks)
12. [Practical Lab Exercises](#practical-lab-exercises)
13. [Integration with Other TSK Tools](#integration-with-other-tsk-tools)
14. [Advanced Scenarios](#advanced-scenarios)
15. [Troubleshooting](#troubleshooting)
16. [Best Practices](#best-practices)
17. [Summary](#summary)

---

## Overview

`mmls` is a command-line utility that is part of **The Sleuth Kit (TSK)**, an open-source digital forensics toolkit developed by Brian Carrier. The name `mmls` stands for **"media management list"** and its primary purpose is to display the **partition table layout** of a disk image or physical storage device.

In digital forensics, understanding the partition structure of a storage medium is almost always the **first critical step**. Before you can analyze files, recover deleted data, or examine filesystem metadata, you must understand how the disk is organized — where partitions begin, where they end, what type each partition is, and whether any space is unallocated.

`mmls` reads the volume system (partition table) information from a disk image and displays it in a human-readable tabular format. It supports a wide range of partition table formats including DOS/MBR, GPT, BSD disklabels, Apple Partition Map, and Sun VTOC.

### Why Partition Analysis Matters

When you receive a disk for forensic examination, you need answers to fundamental questions:

- How many partitions exist on this disk?
- What are the boundaries of each partition?
- What filesystem types are used?
- Is there unallocated space between partitions?
- Are there hidden or nested partitions?
- What is the byte offset to each partition's filesystem?

`mmls` answers all of these questions with a single command. The output provides the foundational map that guides every subsequent forensic operation.

### Relationship to Other TSK Tools

`mmls` is the entry point in the TSK workflow. After using `mmls` to identify partitions, you would typically proceed with:

- `fsstat` — to display filesystem details
- `fls` — to list files and directories
- `icat` — to extract file contents
- `tsk_gettimes` — to create timeline data
- `mmcat` — to output the raw contents of a partition

Think of `mmls` as reading the table of contents before diving into the chapters of a book.

---

## Understanding Partition Tables

### What Is a Partition Table?

A **partition table** is a data structure stored on a disk that describes how the disk is divided into logical sections called **partitions**. Each partition can contain a filesystem, swap space, or raw data. The partition table acts as a map, telling the operating system where each partition begins and ends.

### MBR (Master Boot Record)

The **MBR** partition table is the oldest and most widely compatible partition scheme. It is located in the first 512 bytes of the disk (sector 0) and supports:

- Up to **4 primary partitions**, or
- 3 primary partitions + 1 extended partition (which can contain logical partitions)
- Maximum disk size of **2 TB** (due to 32-bit LBA addressing)
- Partition type identifiers (0x83 for Linux, 0x07 for NTFS, etc.)

The MBR structure consists of:
- Bootstrap code (446 bytes)
- Partition table entries (64 bytes — 4 entries × 16 bytes each)
- Boot signature (2 bytes — 0x55AA)

### GPT (GUID Partition Table)

The **GPT** scheme was introduced as part of the UEFI specification and overcomes MBR limitations:

- Supports up to **128 partitions** (by default on Linux)
- No 2 TB disk size limit
- Uses **GUIDs** (Globally Unique Identifiers) to identify partition types
- Includes redundancy — stores a backup copy of the partition table at the end of the disk
- Uses CRC32 checksums for integrity verification

### Apple Partition Map (APM)

Used on older Macintosh systems with PowerPC processors. It uses a linked-list structure where each partition is described by a partition entry that can also function as a partition itself.

### BSD Disklabel

BSD systems (FreeBSD, NetBSD, OpenBSD) use their own partitioning scheme called a **disklabel**. On x86 systems, BSD typically uses an MBR partition (type 0xA5) that contains the BSD disklabel with up to 8 partitions (labeled a-h).

### Sun VTOC (Volume Table of Contents)

Used on Sun Microsystems/Solaris systems. Similar to BSD disklabels in that it subdivides an MBR partition.

---

## Installation

### On Kali Linux

```bash
sudo apt update
sudo apt install sleuthkit
```

### Verify Installation

```bash
mmls -V
```

Output:

```
mmls: The Sleuth Kit v4.14.0
```

### Installation from Source

```bash
git clone https://github.com/sleuthkit/sleuthkit.git
cd sleuthkit
./bootstrap
./configure
make
sudo make install
```

---

## Command Syntax

```
mmls [options] image [images]
```

The `image` argument is the path to the disk image file (or a `/dev/sdX` device node for a live disk). The `[images]` part allows for concatenated images (for spanned disks).

### Basic Usage

```bash
# Display partition table from a disk image
mmls evidence.E01

# Display from a raw dd image
mmls disk.img

# Display from a live disk (requires root)
mmls /dev/sda
```

---

## Options Reference

### Core Options

| Option | Description |
|--------|-------------|
| `-t vstype` | Specify the volume system (partition table) type. Use `-t list` to see all supported types. |
| `-i imgtype` | Specify the image format. Use `-i list` to see all supported formats. |
| `-b dev_sector_size` | Specify the device sector size in bytes (default is 512). |
| `-o imgoffset` | Offset (in sectors) to the start of the volume system. |
| `-B` | Print the rounded length in bytes instead of sector counts. |
| `-r` | Recurse and look for nested partition tables within partitions (DOS only). |
| `-v` | Verbose output to stderr. |
| `-V` | Print the version number and exit. |

### Display Filter Options

| Option | Description |
|--------|-------------|
| `-a` | Show only **allocated** volumes (partitions with data). |
| `-A` | Show only **unallocated** volumes (free space between partitions). |
| `-m` | Show **metadata** volumes (partition table entries that describe the table itself). |
| `-M` | **Hide** metadata volumes from output. |

### Getting Help

```bash
# List all supported volume system types
mmls -t list

# List all supported image formats
mmls -i list
```

### Using `-o` (Offset)

The `-o` option is critical when dealing with disk images that have nested partition schemes. For example, if you have a disk image where the partition table is not at byte 0 but starts at a certain offset:

```bash
# Partition table starts at sector 63 (common for older systems)
mmls -o 63 disk.img

# Nested partition within an outer partition
mmls -o 2048 outer_disk.img
```

### Using `-B` (Bytes)

By default, `mmls` displays sizes in sectors. The `-B` flag converts all size values to bytes, which can be easier to read:

```bash
mmls -B disk.img
```

### Using `-r` (Recurse)

The `-r` option tells `mmls` to recursively search for partition tables within existing partitions. This is useful for:

- Finding nested partition tables (e.g., an MBR inside a GPT partition)
- Detecting VM disk images within a filesystem
- Identifying encrypted volume containers

```bash
mmls -r disk.img
```

---

## Understanding the Output

### Output Format

`mmls` displays a table with the following columns:

```
Disposition  Start        End          Length       Description
----------   ----------   ----------   ----------   -----------
000:  -----  0000000000   0000000062   0000000063   Unallocated
001:  -----  0000000063   0000409662   0000409600   NTFS (0x07)
002:  -----  0000409663   0000819262   0000409600   Linux (0x83)
003:  -----  0000819263   0000838862   0000019600   Linux Swap (0x82)
```

### Column Definitions

| Column | Description |
|--------|-------------|
| **Disposition** | A three-digit entry number (000, 001, etc.) and allocation status. `-----` means allocated. `[000]` bracketed means metadata entry. |
| **Start** | Starting sector of the partition. |
| **End** | Ending sector of the partition. |
| **Length** | Total length of the partition in sectors. |
| **Description** | Human-readable description including the partition type name and hex type code. |

### Allocation Status Symbols

- `-----` : Allocated partition (contains data/filesystem)
- `[###]` : Metadata volume entry (describes the partition table itself)
- Unallocated entries show free space between partitions

### Interpreting the Numbers

To convert sector numbers to byte offsets:

```
Byte Offset = Sector Number × Sector Size (usually 512)
```

For example, if a partition starts at sector 2048:

```
Byte Offset = 2048 × 512 = 1,048,576 bytes (1 MB)
```

This byte offset is what you would pass to `fls`, `fsstat`, `icat`, and other TSK filesystem tools via the `-o` option.

---

## Supported Volume System Types

Use `mmls -t list` to see all supported types. Common types include:

| Type Code | Description |
|-----------|-------------|
| dos | DOS/Windows partition table (MBR) |
| gpt | GUID Partition Table |
| mac | Apple Partition Map |
| bsd | BSD disklabel |
| sun | Sun VTOC (Volume Table of Contents) |
| apfs | Apple File System |

### When to Specify the Type

In most cases, `mmls` automatically detects the partition table type. You would manually specify it with `-t` when:

- The auto-detection fails or gives wrong results
- You want to force analysis of a specific format
- The image contains multiple nested partition schemes

---

## Supported Image Types

Use `mmls -i list` to see all supported image formats. Common formats include:

| Format | Extension | Description |
|--------|-----------|-------------|
| raw/dd | .img, .dd, .raw | Raw disk image (bit-for-bit copy) |
| ewf | .E01, .Ex01 | Expert Witness Format (EnCase) |
| aff | .aff, .afd | Advanced Forensics Format |
| vmdk | .vmdk | VMware Virtual Machine Disk |
| vhdi | .vhd, .vhdx | Microsoft Virtual Hard Disk |

### When to Specify the Image Type

`mmls` usually auto-detects the image format. Specify `-i` when:

- The file extension doesn't match the actual format
- The auto-detection is ambiguous
- You're working with a concatenated image set

---

## Common Use Cases

### Scenario 1: Initial Evidence Assessment

When you first receive a disk image for analysis:

```bash
# Step 1: Identify the partition table
mmls evidence.E01

# Step 2: If auto-detection fails, try forcing types
mmls -t dos disk.img
mmls -t gpt disk.img

# Step 3: Note the partition offsets for further analysis
# Then use those offsets with filesystem tools:
fsstat -o 2048 evidence.E01
fls -o 2048 evidence.E01
```

### Scenario 2: Finding Hidden Partitions

```bash
# Use recursion to find nested partition tables
mmls -r disk.img

# Show unallocated space between partitions
mmls -A disk.img

# Show metadata entries
mmls -m disk.img
```

### Scenario 3: Working with Large Disks

```bash
# Use -B for byte-based sizes (easier to read for large disks)
mmls -B large_disk.img

# For disks > 2TB, ensure you're using GPT
mmls -t gpt large_disk.img
```

### Scenario 4: Analyzing Virtual Machine Disks

```bash
# VMware VMDK
mmls -i vmdk vm_disk.vmdk

# Hyper-V VHD
mmls -i vhdi virtual_disk.vhd

# VirtualBox (uses VMDK format)
mmls -i vmdk virtualbox_disk.vmdk
```

### Scenario 5: Examining Encrypted Containers

If you suspect a disk image contains an encrypted volume container:

```bash
# Look for unusual partition types
mmls disk.img

# Search for nested partition tables (TrueCrypt/VeraCrypt)
mmls -r -t dos disk.img

# Use sigfind to look for encryption signatures
sigfind -t dospart disk.img
```

---

## Working with Disk Images

### Creating a Disk Image

Before analysis, always create a forensic image of the evidence:

```bash
# Using dc3dd with hashing
dc3dd if=/dev/sda of=/evidence/disk.img hash=sha256 log=/evidence/imaging.log

# Using dd (less forensic-specific)
dd if=/dev/sda of=/evidence/disk.img bs=4M status=progress

# Using dcfldd (enhanced dd)
dcfldd if=/dev/sda of=/evidence/disk.img hash=sha256
```

### Analyzing the Image

```bash
# Always work with the copy, never the original
mmls /evidence/disk.img

# If the image is segmented (e.g., E01 format)
mmls evidence.E01
```

### Common Image Formats and mmls

```bash
# Raw image (dd output)
mmls disk.img

# EnCase EWF format
mmls evidence.E01

# AFF format
mmls evidence.aff

# Split raw images (img.000, img.001, etc.)
mmls img.000
```

---

## Working with Live Disks

### Reading a Physical Disk

```bash
# Must run as root
sudo mmls /dev/sda

# Check the disk geometry
mmls -v /dev/sda

# Force sector size if auto-detection fails
mmls -b 4096 /dev/nvme0n1
```

### Important Warning

**Never run `mmls` with write options on a live evidence disk.** `mmls` is a read-only tool, but always exercise caution when working with physical evidence. Use a write blocker!

```bash
# SAFE - mmls is always read-only
mmls /dev/sda

# Verify your write blocker is active
sudo hdparm -r /dev/sda
```

---

## Practical Lab Exercises

### Lab 1: Basic Partition Table Analysis

```bash
# Create a test disk image with multiple partitions
sudo dd if=/dev/zero of=test_disk.img bs=1M count=500

# Create MBR partition table
sudo parted test_disk.img mklabel msdos

# Create partitions
sudo parted test_disk.img mkpart primary ext4 1MiB 200MiB
sudo parted test_disk.img mkpart primary ntfs 200MiB 350MiB
sudo parted test_disk.img mkpart primary linux-swap 350MiB 500MiB

# Analyze with mmls
mmls test_disk.img

# Expected output should show 3 allocated partitions and
# unallocated space at the beginning and end
```

### Lab 2: Nested Partition Detection

```bash
# Create an image with a nested partition table
# (simulating a VM disk inside a partition)

# First, create outer image
dd if=/dev/zero of=outer.img bs=1M count=1000

# Create inner image
dd if=/dev/zero of=inner.img bs=1M count=500
mkfs.ext4 inner.img

# Embed inner image at a known offset
dd if=inner.img of=outer.img bs=1M seek=100 conv=notrunc

# Use mmls with recursion
mmls -r outer.img
```

### Lab 3: GPT Partition Analysis

```bash
# Create a GPT disk image
dd if=/dev/zero of=gpt_disk.img bs=1M count=1000
parted gpt_disk.img mklabel gpt
parted gpt_disk.img mkpart primary ext4 1MiB 300MiB
parted gpt_disk.img mkpart primary ntfs 300MiB 700MiB

# Analyze with mmls
mmls gpt_disk.img

# Compare MBR vs GPT output formats
mmls -t dos gpt_disk.img  # Should show the GPT protective MBR
mmls -t gpt gpt_disk.img  # Shows actual GPT entries
```

### Lab 4: Working with Real-World Formats

```bash
# If you have an E01 file
mmls evidence.E01

# Check supported formats
mmls -i list

# Force a specific image type
mmls -i raw -t dos custom_disk.img
```

---

## Integration with Other TSK Tools

### The TSK Workflow

```
mmls (partition table)
  ↓
fsstat (filesystem info)
  ↓
fls (file listing)
  ↓
icat (file extraction)
  ↓
ils / istat (inode analysis)
  ↓
mactime (timeline generation)
```

### Step-by-Step Analysis Example

```bash
# Step 1: Identify partitions
mmls disk.img

# Output shows:
# 001: -----  0000000063   0001048575   0001048513   NTFS (0x07)

# Step 2: Examine the NTFS filesystem (offset = 63 sectors)
fsstat -f ntfs -o 63 disk.img

# Step 3: List files
fls -f ntfs -o 63 -r -p / disk.img

# Step 4: Extract a specific file
icat -f ntfs -o 63 disk.img [inode_number] > recovered_file.dat

# Step 5: Check inodes
ils -f ntfs -o 63 disk.img
istat -f ntfs -o 63 disk.img [inode_number]

# Step 6: Generate timeline
tsk_gettimes -f ntfs -o 63 disk.img > body.txt
mactime -b body.txt -d > timeline.csv
```

### Using -o (Offset) Correctly

The `-o` option in filesystem tools must match the **starting sector** of the partition as shown by `mmls`:

```bash
# mmls shows partition starts at sector 63
# So for fsstat, fls, icat, etc., use -o 63
fls -f ntfs -o 63 disk.img

# For the second partition starting at sector 1048576
fls -f ext4 -o 1048576 disk.img
```

---

## Advanced Scenarios

### Multiple Partition Tables

Some disks contain multiple partition tables (e.g., a Mac disk with both APM and GPT):

```bash
# Try each type
mmls -t mac disk.img
mmls -t gpt disk.img

# Use recursion to find nested tables
mmls -r disk.img
```

### Extended Partitions

MBR extended partitions (containing logical partitions) are displayed as a single entry by `mmls`. To see logical partitions within an extended partition, use the `-r` flag:

```bash
mmls -r disk.img
```

### Detecting Disk Encryption

Look for these indicators of encrypted volumes:

- Large partition with type `0x00` (unknown/empty)
- Unusually large unallocated spaces
- Nested partition tables within a single partition
- Known encryption container signatures

```bash
# Search for TrueCrypt/VeraCrypt signatures
sigfind -t dospart disk.img

# Look for LUKS signatures
sigfind 4c554b53bab6 disk.img
```

### Recovering from Corrupted Partition Tables

If the partition table is damaged:

```bash
# Try different types
mmls -t dos disk.img
mmls -t gpt disk.img

# Use verbose mode for more detail
mmls -v disk.img

# Try with a different sector size (for advanced format drives)
mmls -b 4096 disk.img

# Use sigfind to locate partition signatures manually
sigfind -t dospart disk.img
sigfind -t ext2 disk.img
```

---

## Troubleshooting

### "Cannot determine partition type"

This usually means `mmls` cannot auto-detect the partition table type.

**Solutions:**

```bash
# Force a specific type
mmls -t dos disk.img
mmls -t gpt disk.img

# Check if it's a nested table
mmls -o <offset> disk.img

# Try with verbose output
mmls -v disk.img
```

### "Invalid sector size"

The sector size auto-detection may be wrong for advanced format drives (4K sector).

**Solutions:**

```bash
# Force 4096-byte sectors
mmls -b 4096 disk.img

# Force 512-byte sectors
mmls -b 512 disk.img
```

### "No such file or directory"

The image file path may be incorrect.

**Solutions:**

```bash
# Verify the file exists
ls -la /path/to/disk.img

# Check if it's a segmented image
ls /path/to/evidence.*

# For E01 files, point to the first segment
mmls /path/to/evidence.E01
```

### "Cannot read image"

The image may be corrupted or in an unsupported format.

**Solutions:**

```bash
# Check supported formats
mmls -i list

# Force the image type
mmls -i raw disk.img

# Check image integrity
img_stat disk.img
```

### Output Shows Only One Entry

This can happen with GPT disks when viewing the protective MBR.

**Solution:**

```bash
# Explicitly request GPT analysis
mmls -t gpt disk.img
```

---

## Best Practices

### 1. Always Work with Copies

Never analyze the original evidence disk directly. Create a forensic image first:

```bash
dc3dd if=/dev/sda of=/evidence/copy.img hash=sha256 log=/evidence/hash.log
mmls /evidence/copy.img
```

### 2. Document Your Findings

Record the output of every `mmls` command:

```bash
mmls disk.img | tee case_001_partition_table.txt
```

### 3. Verify Write Protection

Before connecting evidence media, ensure write blockers are in place:

```bash
# Check device is read-only
sudo hdparm -r /dev/sdb
```

### 4. Use the Correct Offset

Always use the partition starting sector from `mmls` output as the `-o` offset in filesystem tools:

```bash
# mmls shows: 001: -----  0000000063
# Use 63 as the offset
fls -f ntfs -o 63 disk.img
```

### 5. Consider Sector Size

Modern drives often use 4096-byte sectors. If `mmls` output looks wrong, try:

```bash
mmls -b 4096 disk.img
```

### 6. Chain Your Commands

Use a consistent workflow:

```bash
# 1. Partition table
mmls disk.img > partition_table.txt

# 2. For each partition
fsstat -f <type> -o <offset> disk.img > fs_info.txt
fls -f <type> -o <offset> -r -m / disk.img > body.txt
ils -f <type> -o <offset> disk.img > inodes.txt

# 3. Timeline
mactime -b body.txt -d > timeline.csv
```

### 7. Use Hash Verification

Always verify your evidence image hash before and after analysis:

```bash
# Before
sha256sum disk.img > original_hash.txt

# After analysis
sha256sum disk.img > verification_hash.txt

# Compare
diff original_hash.txt verification_hash.txt
```

### 8. Check All Partition Table Types

Don't assume you know the partition type. Check the most common ones:

```bash
mmls disk.img          # auto-detect
mmls -t dos disk.img   # MBR
mmls -t gpt disk.img   # GPT
```

---

## Summary

`mmls` is an essential first step in any digital forensics investigation involving disk analysis. It provides:

- **Partition identification** — know how the disk is divided
- **Boundary information** — exact start and end sectors
- **Filesystem types** — what filesystem each partition contains
- **Unallocated space** — free space between partitions
- **Foundation for analysis** — the offsets needed for all subsequent tools

Key takeaways:

1. Always start your disk examination with `mmls`
2. Record the partition offsets for use with `fls`, `fsstat`, `icat`, and other tools
3. Use `-t list` and `-i list` when auto-detection fails
4. Work with forensic copies, never original evidence
5. Document every command and its output

### Quick Start Command Reference

```bash
# Basic partition table display
mmls disk.img

# Force specific partition table type
mmls -t gpt disk.img

# Show in bytes
mmls -B disk.img

# Show only allocated partitions
mmls -a disk.img

# Show only unallocated space
mmls -A disk.img

# Recursive (find nested partition tables)
mmls -r disk.img

# With offset for nested tables
mmls -o <sectors> disk.img

# List supported types
mmls -t list
mmls -i list
```

---

## References

- The Sleuth Kit Documentation: https://www.sleuthkit.org/sleuthkit/man/
- Brian Carrier, "File System Forensic Analysis," Addison-Wesley, 2005.
- NIST Special Publication 800-86: Guide to Integrating Forensic Techniques into Incident Response.
- Kali Linux Tools: https://www.kali.org/tools/sleuthkit/
