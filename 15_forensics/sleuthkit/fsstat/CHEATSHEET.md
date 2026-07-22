# fsstat — Forensic Filesystem Information Display | CHEATSHEET

**Part of The Sleuth Kit (TSK)** | **MITRE: TA0007 Analysis** | **GitHub: sleuthkit/sleuthkit**

---

## Quick Syntax

```
fsstat [options] <image>
```

---

## Essential Flags

| Flag | Purpose | Example |
|------|---------|---------|
| `-s` | Superblock info only | `fsstat -s image.dd` |
| `-l` | Journal/log info | `fsstat -l image.dd` |
| `-I` | Inode allocation | `fsstat -I image.dd` |
| `-b` | Block allocation | `fsstat -b image.dd` |
| `-d` | Detailed output | `fsstat -d image.dd` |
| `-f TYPE` | Force filesystem type | `fsstat -f ext4 image.dd` |
| `-o OFFSET` | Byte offset to FS | `fsstat -o 2048 image.dd` |
| `-h` | Help | `fsstat -h` |
| `-V` | Version | `fsstat -V` |

---

## Common Workflows

### Basic Filesystem Info
```bash
fsstat usb_image.dd
```

### Quick FS Type Check
```bash
fsstat usb_image.dd | grep "File System Type:"
```

### Superblock Details
```bash
fsstat -s usb_image.dd
```

### Journal State Check
```bash
fsstat -l usb_image.dd | grep -E "(Journal|Clean)"
```

### Multi-Partition Analysis
```bash
mmls disk.dd            # Find offsets
fsstat -o 2048 disk.dd  # Partition 1
fsstat -o 206848 disk.dd  # Partition 2
```

### Full Forensic Workflow
```bash
fsstat image.dd         # 1. Identify filesystem
fls -r -m "/" image.dd  # 2. List files
ils -e image.dd         # 3. Find inodes
icat image.dd <inode>   # 4. Extract file
```

---

## Output Sections

| Section | Contains |
|---------|----------|
| **File System Type** | ext4, NTFS, FAT32, etc. |
| **Volume Information** | Name, serial number |
| **Size/Offsets** | Block size, total/free blocks |
| **Inode Info** | Total/free inodes |
| **Timestamps** | Last mount, write, check |
| **State Flags** | Clean/unmounted, mount count |
| **Journal** | Journal inode, length, state |

---

## Quick Reference: What Each Flag Does

| You Want To... | Use Flag |
|----------------|----------|
| See basic filesystem info | (no flags) |
| See superblock only | `-s` |
| Check journal state | `-l` |
| See inode allocation | `-I` |
| See block allocation | `-b` |
| Force filesystem type | `-f TYPE` |
| Handle partitioned image | `-o OFFSET` |
| Get maximum detail | `-d` |

---

## Filesystem Types

| Code | Filesystem |
|------|------------|
| ext2 | Linux ext2 |
| ext3 | Linux ext3 |
| ext4 | Linux ext4 |
| fat | FAT (auto-detect) |
| fat12 | FAT12 |
| fat16 | FAT16 |
| fat32 | FAT32 |
| ntfs | Windows NTFS |
| hfs+ | macOS HFS+ |
| ufs | BSD UFS |

---

## TSK Tool Pipeline

```
┌─────────────┐     ┌─────────┐     ┌─────────┐     ┌─────────┐
│   mmls      │────▶│  fsstat │────▶│  fls    │────▶│  icat   │
│  (find      │     │ (what   │     │ (list   │     │(extract │
│  partition) │     │  FS?)   │     │  files) │     │ content)│
└─────────────┘     └─────────┘     └─────────┘     └─────────┘
```

---

## Exit Codes

| Code | Meaning |
|------|---------|
| `0` | Success |
| `1` | Error / invalid filesystem |
| `2` | Cannot open image |

---

## One-Liners

```bash
# Basic filesystem info
fsstat usb_image.dd

# Filesystem type only
fsstat usb_image.dd | grep "File System Type:"

# Block size
fsstat usb_image.dd | grep "Block Size:"

# Free space
fsstat usb_image.dd | grep "Free Blocks:"

# Last write timestamp
fsstat usb_image.dd | grep "Last Written At:"

# Clean unmount flag
fsstat usb_image.dd | grep "Clean:"

# Journal inode
fsstat -l usb_image.dd | grep "Journal Inode:"

# Force ext4 on offset
fsstat -f ext4 -o 2048 disk.dd

# Full profile to file
fsstat -d -l -s usb_image.dd > fs_profile.txt
```

---

## Troubleshooting

| Problem | Solution |
|---------|----------|
| "Invalid filesystem" | Use `mmls` to find offset, try `-o OFFSET` |
| Wrong FS detected | Use `-f TYPE` to force correct type |
| Slow on large images | Use `-f` to skip auto-detection |
| Can't read journal | Try `-l` flag explicitly |
| Partitioned image | Use `mmls` first, then `-o OFFSET` |

---

## Key Concepts

- **fsstat** = filesystem identification and metadata overview
- **Always run first** before other TSK tools
- Provides **block size**, **inode counts**, **timestamps**
- Shows **journal state** (clean/dirty)
- Use with `mmls` for multi-partition images

---

## Common Output Fields

| Field | Meaning |
|-------|---------|
| `Block Size` | Size of each filesystem block |
| `Free Blocks` | Available space blocks |
| `Free Inodes` | Available inode slots |
| `Last Written At` | Most recent write timestamp |
| `Clean` | Whether FS was unmounted properly |
| `Mount Count` | Times filesystem has been mounted |
| `Journal Inode` | Inode of the journal file |

---

**TSK Install**: `sudo apt install sleuthkit` | **Man Page**: `man fsstat`
