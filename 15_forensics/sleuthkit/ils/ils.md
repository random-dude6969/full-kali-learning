# ils — List Inode Entries

## Table of Contents

1. [Overview](#overview)
2. [Understanding Inodes and Allocation](#understanding-inodes-and-allocation)
3. [Installation](#installation)
4. [Command Syntax](#command-syntax)
5. [Options Reference](#options-reference)
6. [Understanding the Output](#understanding-the-output)
7. [Filtering Inodes](#filtering-inodes)
8. [Practical Lab Exercises](#practical-lab-exercises)
9. [Integration with Other TSK Tools](#integration-with-other-tsk-tools)
10. [Common Forensic Scenarios](#common-forensic-scenarios)
11. [Troubleshooting](#troubleshooting)
12. [Best Practices](#best-practices)
13. [Summary](#summary)

---

## Overview

`ils` is a command-line utility that is part of **The Sleuth Kit (TSK)**. Its purpose is to **list all inode (metadata structure) entries** from a filesystem, showing their allocation status and key attributes.

While `fls` lists filenames and `istat` examines a single inode, `ils` provides a comprehensive view of **all metadata structures** on the filesystem, regardless of whether they have filenames. This makes `ils` particularly powerful for:

- Finding deleted files (unallocated inodes that may still contain data)
- Identifying orphan inodes (inodes with no filename references)
- Understanding filesystem usage patterns
- Generating body file data for timeline analysis

### `ils` vs `fls` vs `istat`

| Tool | Purpose | Input | Output |
|------|---------|-------|--------|
| `fls` | List filenames | Directory entries | Filenames + inodes |
| `ils` | List all inodes | Filesystem metadata | Inode entries + attributes |
| `istat` | Examine one inode | Single inode number | Detailed inode info |

`ils` sees everything at the metadata level, including entries that `fls` might miss (like deleted directory entries that have been overwritten).

---

## Understanding Inodes and Allocation

### Inode Allocation States

Every inode on a filesystem exists in one of these states:

**Allocated:**
- The inode is actively in use by a file or directory
- It has a valid filename reference in a directory
- The filesystem considers it "live"

**Unallocated:**
- The inode was previously used but is now marked as free
- Its data blocks may still contain old file data
- The filesystem may reuse this inode for new files
- This is where deleted file recovery happens

**Linked:**
- The inode has at least one directory entry pointing to it
- This is the normal state for active files

**Unlinked:**
- The inode has no directory entries pointing to it
- May still be in use by a running process (open but deleted file)
- The data is still accessible until the process closes the file

**Orphan:**
- The inode is unallocated AND has no directory entries
- Purely orphaned metadata — no filename reference exists
- Data blocks may still contain recoverable content

### Why Allocation Status Matters in Forensics

- **Unallocated inodes** are the primary source of deleted file evidence
- **Orphan inodes** indicate files that were deleted long ago
- **Unlinked inodes** may indicate active malware hiding from directory listings
- **Allocated inodes with zero link count** indicate recently deleted files

---

## Installation

```bash
sudo apt install sleuthkit
```

---

## Command Syntax

```bash
ils [options] image [inum[-end]]
```

### Basic Usage

```bash
# List all inodes
ils -f ntfs -o 63 disk.img

# List only unallocated inodes (deleted files)
ils -e -f ntfs -o 63 disk.img

# List a specific range of inodes
ils -f ntfs -o 63 disk.img 32-100

# Output in mactime body format
ils -m -f ntfs -o 63 disk.img
```

---

## Options Reference

### Display Options

| Option | Description |
|--------|-------------|
| `-e` | Display **all** inodes (allocated and unallocated) |
| `-m` | Output in **mactime body file** format |
| `-O` | Display inodes that are unallocated but were still open (UFS/ext only) |
| `-p` | Display **orphan** inodes only |

### Allocation Filters

| Option | Description |
|--------|-------------|
| `-a` | Display only **allocated** inodes |
| `-A` | Display only **unallocated** inodes |
| `-l` | Display only **linked** inodes |
| `-L` | Display only **unlinked** inodes |
| `-z` | Display only **unused** inodes |
| `-Z` | Display only **used** inodes |

### General Options

| Option | Description |
|--------|-------------|
| `-f fstype` | Filesystem type |
| `-i imgtype` | Image file format |
| `-b dev_sector_size` | Device sector size |
| `-o imgoffset` | Offset to filesystem in sectors |
| `-s seconds` | Time skew adjustment |
| `-P pooltype` | Pool container type |
| `-B pool_volume_block` | Starting block for pool volumes |
| `-v` | Verbose output |
| `-V` | Print version |

---

## Understanding the Output

### Default Output Format

```
Meta    Num  Alloc  Use    Fmt  Fsi  Size     uid/gid    Mode     crtime           amtime           ctime           dtime
000:    0    a      u      fsl  ---  0        0/0        --------- 2024-01-15 10:00 2024-01-15 10:00 2024-01-15 10:00 0000-00-00 00:00
001:    1    a      u      fsl  ---  0        0/0        --------- 2024-01-15 10:00 2024-01-15 10:00 2024-01-15 10:00 0000-00-00 00:00
002:    2    a      u      fsl  DIR  4096     0/0        drwxr-xr-x 2024-01-15 10:00 2024-01-15 10:00 2024-01-15 10:00 0000-00-00 00:00
003:    3    a      u      fsl  ---  0        0/0        --------- 2024-01-15 10:00 2024-01-15 10:00 2024-01-15 10:00 0000-00-00 00:00
004:    4    a      u      fsl  DIR  4096     0/0        drwx------ 2024-01-15 10:00 2024-01-15 10:00 2024-01-15 10:00 0000-00-00 00:00
005:    5    a      u      fsl  DIR  4096     0/0        drwxr-xr-x 2024-01-15 10:00 2024-01-15 10:00 2024-01-15 10:00 0000-00-00 00:00
```

### Column Definitions

| Column | Description |
|--------|-------------|
| **Meta** | Internal metadata address (may differ from Num on some FS) |
| **Num** | Inode number (the number you'd use with `istat` and `icat`) |
| **Alloc** | Allocation status: `a` = allocated, `u` = unallocated |
| **Use** | Usage: `u` = used, `z` = unused |
| **Fmt** | Format: `fsl` = filesystem, etc. |
| **Fsi** | File type indicator: `DIR` = directory, `---` = file, etc. |
| **Size** | File size in bytes |
| **uid/gid** | Owner UID and group GID |
| **Mode** | Permissions (Unix-style) |
| **crtime** | Creation/birth time |
| **amtime** | Access time |
| **ctime** | Changed time (metadata change) |
| **dtime** | Deleted time (when the inode was freed) |

### Interpreting the "dtime" Column

The **dtime** (deleted time) column is particularly important:

- `0000-00-00 00:00` — The inode is currently allocated (not deleted)
- A valid timestamp — The inode was deleted at that time
- This helps establish when files were deleted

### Mactime Output Format (`-m`)

When using the `-m` flag, `ils` outputs in the body file format compatible with `mactime`:

```
0|12|r/rrw-r--r--|1000|1000|512|1704067200|1704067200|1704067200|0|/home/user/document.txt
```

---

## Filtering Inodes

### Finding Deleted Files (Unallocated Inodes)

```bash
# Show only unallocated inodes
ils -A -f ntfs -o 63 disk.img

# Show all inodes and filter manually
ils -e -f ntfs -o 63 disk.img | grep " u "
```

### Finding Active Files (Allocated Inodes)

```bash
# Show only allocated inodes
ils -a -f ntfs -o 63 disk.img
```

### Finding Orphan Inodes

```bash
# Show only orphan inodes (no filename references)
ils -p -f ntfs -o 63 disk.img
```

### Finding Unlinked Inodes (Open but Deleted Files)

```bash
# Show unlinked inodes
ils -L -f ntfs -o 63 disk.img
```

### Finding Non-Zero Sized Unallocated Inodes

```bash
# Most interesting for forensics: deleted files with data
ils -A -f ntfs -o 63 disk.img | awk '$6 != 0'
```

### Inode Range Queries

```bash
# Examine inodes 32-64 (NTFS user files start at 32)
ils -f ntfs -o 63 disk.img 32-64

# Examine first 100 inodes
ils -f ext4 -o 2048 disk.img 0-100
```

---

## Practical Lab Exercises

### Lab 1: Basic Inode Listing

```bash
# List all inodes on a filesystem
ils -f ntfs -o 63 disk.img

# Count total inodes
ils -e -f ntfs -o 63 disk.img | wc -l
```

### Lab 2: Finding Deleted Files

```bash
# Find all deleted file inodes
ils -A -f ntfs -o 63 disk.img

# Find deleted files with non-zero size (most interesting)
ils -A -f ntfs -o 63 disk.img | awk '$6 != 0' | sort -k6 -rn

# For each deleted file, examine its inode
for inode in $(ils -A -f ntfs -o 63 disk.img | awk '$6 != 0 {print $2}'); do
    echo "=== Inode $inode ==="
    istat -f ntfs -o 63 disk.img $inode
done
```

### Lab 3: Generating Body File for Timeline

```bash
# Generate mactime body file from all inodes
ils -m -f ntfs -o 63 disk.img > body.txt

# Create timeline
mactime -b body.txt -d > timeline.csv
```

### Lab 4: Identifying Orphan Files

```bash
# Find orphan inodes (no directory reference)
ils -p -f ntfs -o 63 disk.img

# These are files with no filename — may contain recoverable data
for inode in $(ils -p -f ntfs -o 63 disk.img | awk '{print $2}'); do
    echo "=== Inode $inode ==="
    istat -f ntfs -o 63 disk.img $inode
    icat -f ntfs -o 63 disk.img $inode > "orphan_$inode.dat" 2>/dev/null
done
```

### Lab 5: Comparing ext4 and NTFS Inode Listings

```bash
# ext4 inodes
ils -f ext4 -o 2048 disk.img

# NTFS MFT entries
ils -f ntfs -o 63 disk.img

# Note differences in:
# - Numbering scheme
# - Timestamp precision
# - Available attributes
```

---

## Integration with Other TSK Tools

### `ils` → `istat` → `icat` Workflow

```bash
# Step 1: Find interesting inodes with ils
ils -A -f ntfs -o 63 disk.img | awk '$6 != 0' > deleted_inodes.txt

# Step 2: Examine each inode with istat
while read line; do
    inode=$(echo $line | awk '{print $2}')
    istat -f ntfs -o 63 disk.img $inode
done < deleted_inodes.txt

# Step 3: Extract files with icat
while read line; do
    inode=$(echo $line | awk '{print $2}')
    icat -f ntfs -o 63 disk.img $inode > "recovered_$inode.dat" 2>/dev/null
done < deleted_inodes.txt
```

### `ils` with `fls` Comparison

```bash
# fls shows filenames, ils shows all inodes
# Comparing reveals orphan inodes

fls -f ntfs -o 63 -r disk.img | awk -F'[: ]' '{print $2}' | sort -n > fls_inodes.txt
ils -e -f ntfs -o 63 disk.img | awk '{print $2}' | sort -n > ils_inodes.txt

# Find inodes in ils but not in fls (orphans)
comm -23 ils_inodes.txt fls_inodes.txt
```

### `ils` with `mactime` for Timeline

```bash
# Generate timeline from all inodes
ils -m -f ntfs -o 63 disk.img | mactime -

# Generate timeline of only deleted files
ils -m -A -f ntfs -o 63 disk.img | mactime -
```

### `ils` with `blkstat` for Block Analysis

```bash
# Find which blocks an unallocated inode references
ils -A -f ntfs -o 63 disk.img | awk '$6 != 0 {print $2}' | while read inode; do
    echo "=== Inode $inode ==="
    istat -f ntfs -o 63 -r disk.img $inode
done
```

---

## Common Forensic Scenarios

### Scenario 1: Deleted File Recovery

```bash
# Find recently deleted files (check dtime)
ils -A -f ntfs -o 63 disk.img | awk '$6 != 0'

# Files deleted most recently have the most recoverable data
# Extract them immediately before any overwrite occurs
```

### Scenario 2: Anti-Forensics Detection

```bash
# Look for files with suspicious inode patterns:
# - Very large files recently deleted
# - Files with zero size but non-zero dtime
# - Multiple inodes with identical timestamps (mass deletion)

ils -A -f ntfs -o 63 disk.img | sort -k6 -rn | head -20
```

### Scenario 3: Malware Investigation

```bash
# Find executable files (check mode for execute permission)
ils -e -f ntfs -o 63 disk.img | grep "x"

# Find files created recently (check crtime)
ils -e -f ntfs -o 63 disk.img | grep "2024-01-15"
```

### Scenario 4: Data Exfiltration

```bash
# Find large deleted files (data staging before exfiltration)
ils -A -f ext4 -o 2048 disk.img | awk '$6 > 1048576' | sort -k6 -rn
```

### Scenario 5: Timeline Analysis

```bash
# Generate full timeline from all inodes
ils -m -f ntfs -o 63 disk.img > body.txt
mactime -b body.txt -d > full_timeline.csv

# Generate timeline of deleted files only
ils -m -A -f ntfs -o 63 disk.img > deleted_body.txt
mactime -b deleted_body.txt -d > deleted_timeline.csv
```

---

## Troubleshooting

### "No inodes found"

- Check the filesystem type and offset
- Verify the image is readable: `img_stat disk.img`
- Try specifying `-f` explicitly

### "Too many results"

Use filtering options to narrow the output:

```bash
# Only unallocated
ils -A -f ntfs -o 63 disk.img

# Only non-zero sized
ils -e -f ntfs -o 63 disk.img | awk '$6 != 0'

# Only specific inode range
ils -f ntfs -o 63 disk.img 32-64
```

### Output is Difficult to Parse

Use `-m` for mactime-compatible output, or pipe through `awk`/`cut`:

```bash
ils -e -f ntfs -o 63 disk.img | awk '{print $2, $4, $6, $NF}'
```

### "Orphan inodes" Not Showing

Orphan inodes may be rare on well-maintained filesystems. Try:

```bash
ils -p -f ntfs -o 63 disk.img
# If empty, the filesystem has no orphan inodes
```

---

## Best Practices

1. **Start with `-e` to see everything** — then filter as needed
2. **Focus on unallocated inodes with data** — `ils -A | awk '$6 != 0'`
3. **Use `-m` for timeline integration** — generates mactime body format
4. **Check dtime values** — recently deleted files are most recoverable
5. **Use `-p` for orphan detection** — files with no name reference
6. **Combine with `istat` and `icat`** — examine and extract interesting inodes
7. **Compare `ils` with `fls`** — find orphans by difference
8. **Save raw output** — always keep the full `ils` output for later analysis
9. **Document which inodes you recover** — maintain chain of evidence
10. **Work quickly with deleted data** — unallocated inodes can be overwritten

---

## Summary

`ils` provides a comprehensive view of all metadata structures on a filesystem. Key capabilities:

- Lists all inodes with allocation status
- Identifies deleted (unallocated) inodes
- Detects orphan and unlinked inodes
- Generates mactime body file format
- Supports inode range queries
- Works with all TSK-supported filesystems

### Quick Reference

```bash
# List all inodes
ils -e -f ntfs -o 63 disk.img

# Only deleted (unallocated) inodes
ils -A -f ntfs -o 63 disk.img

# Deleted files with data
ils -A -f ntfs -o 63 disk.img | awk '$6 != 0'

# Orphan inodes
ils -p -f ntfs -o 63 disk.img

# Mactime body format
ils -m -f ntfs -o 63 disk.img

# Specific inode range
ils -f ntfs -o 63 disk.img 32-64
```

---

## References

- The Sleuth Kit Documentation: https://www.sleuthkit.org/sleuthkit/man/
- Brian Carrier, "File System Forensic Analysis," Addison-Wesley, 2005.
- NIST SP 800-86: Guide to Integrating Forensic Techniques into Incident Response.
