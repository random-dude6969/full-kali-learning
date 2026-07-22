# fls — Forensic File and Directory Listing | CHEATSHEET

**Part of The Sleuth Kit (TSK)** | **MITRE: TA0007 Analysis** | **GitHub: sleuthkit/sleuthkit**

---

## Quick Syntax

```
fls [options] <image> [path]
```

---

## Essential Flags

| Flag | Purpose | Example |
|------|---------|---------|
| `-r` | Recursive listing | `fls -r image.dd` |
| `-d` | Show deleted files | `fls -d image.dd` |
| `-l` | Long format (metadata) | `fls -l image.dd` |
| `-m PATH` | Mactime body format | `fls -m "/" image.dd` |
| `-s` | Show inode numbers | `fls -s image.dd` |
| `-e` | Deleted entries only | `fls -e image.dd` |
| `-C` | CSV output | `fls -C image.dd` |
| `-f TYPE` | Force filesystem type | `fls -f ext4 image.dd` |
| `-o OFFSET` | Byte offset to filesystem | `fls -o 2048 image.dd` |
| `-k` | Show metadata | `fls -k image.dd` |
| `-h` | Help | `fls -h` |
| `-V` | Version | `fls -V` |

---

## Output Format Guide

```
r/r * 12:       resume.doc
││ │  │        │
││ │  │        └── File name
││ │  └── Inode number
││ └── Deleted flag (* = deleted)
│└── Directory type (r=regular, d=directory)
└── File type (r=regular, d=directory, v=virtual)
```

---

## Common Workflows

### Basic Directory Listing
```bash
fls usb_image.dd
```

### Find Deleted Files
```bash
fls -d usb_image.dd
```

### Full Recursive Inventory
```bash
fls -r -d -s -m "/" usb_image.dd > full_inventory.csv
```

### Timeline Analysis
```bash
fls -r -m "/" usb_image.dd > body_file.txt
mactime -b body_file.txt -d timeline/
```

### Find Files by Type
```bash
fls -r -d usb_image.dd | grep -iE "\.(pdf|docx|xlsx)$"
```

### Recover After Finding
```bash
# 1. Find inode
fls -r -d usb_image.dd | grep "secret.txt"
# Output: r/r * 58493: secret.txt

# 2. Extract file
icat usb_image.dd 58493 > recovered_secret.txt
```

---

## Output Types

| Code | Meaning |
|------|---------|
| `r/r` | Regular file |
| `r/d` | Directory |
| `v/v` | Virtual file |
| `c/c` | Character device |
| `b/b` | Block device |
| `*` | Deleted (unallocated) |

---

## Quick Reference: What Each Flag Does

| You Want To... | Use Flag |
|----------------|----------|
| List all files recursively | `-r` |
| See deleted files | `-d` |
 | Get detailed metadata | `-l` |
| Output for timeline | `-m "/"` |
| Show inode numbers | `-s` |
| Export as CSV | `-C` |
| Force filesystem type | `-f TYPE` |
| Handle multi-partition image | `-o OFFSET` |
| See only deleted entries | `-e` |

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
│   fsstat    │────▶│  fls    │────▶│ ffind   │────▶│  icat   │
│  (what FS?) │     │ (list   │     │(inode → │     │(extract │
│             │     │  files) │     │  path)  │     │ content)│
└─────────────┘     └─────────┘     └─────────┘     └─────────┘
```

---

## Exit Codes

| Code | Meaning |
|------|---------|
| `0` | Success |
| `1` | Error or no files |
| `2` | Invalid image/FS |

---

## One-Liners

```bash
# List root directory
fls usb_image.dd

# List deleted files recursively
fls -r -d usb_image.dd

# Long format with inodes
fls -r -d -l -s usb_image.dd

# Timeline format
fls -r -m "/" usb_image.dd

# Find PDFs
fls -r -d usb_image.dd | grep -i "\.pdf$"

# List specific subdirectory
fls usb_image.dd Documents/

# CSV export
fls -r -C usb_image.dd > files.csv

# Deleted files with metadata
fls -d -l -k usb_image.dd

# Multiple partitions
mmls disk.dd  # Find offsets
fls -o 2048 disk.dd  # Partition 1
fls -o 206848 disk.dd  # Partition 2
```

---

## Troubleshooting

| Problem | Solution |
|---------|----------|
| No files shown | Use `mmls` to find partition offset, try `-o OFFSET` |
| Wrong file type | Use `fsstat` to identify, try `-f TYPE` |
| Slow performance | Use `-f` to skip auto-detection |
| Can't find deleted files | Ensure using `-d` flag |
| Need file contents | Use `icat` with inode from fls output |
| Permission denied | Use `sudo` (rare, usually not needed) |

---

## Key Concepts

- **fls** = forensic `ls` that reads disk images directly
- Shows **deleted files** that `ls` cannot see
- No mounting required — works on raw images
- Output includes inode numbers for use with `icat` and `ffind`
- Use `-m` for timeline analysis with `mactime`

---

**TSK Install**: `sudo apt install sleuthkit` | **Man Page**: `man fls`
