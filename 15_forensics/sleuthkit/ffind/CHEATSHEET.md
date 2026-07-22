# ffind — Forensic File Name Lookup by Inode | CHEATSHEET

**Part of The Sleuth Kit (TSK)** | **MITRE: TA0007 Analysis** | **GitHub: sleuthkit/sleuthkit**

---

## Quick Syntax

```
ffind [options] <image> <inode>
```

---

## Essential Flags

| Flag | Purpose | Example |
|------|---------|---------|
| `-f TYPE` | Force filesystem type | `ffind -f ext4 image.dd 12` |
| `-d` | Include deleted directory entries | `ffind -d image.dd 58493` |
| `-v` | Verbose output | `ffind -v image.dd 58493` |
| `-e` | Search embedded entries (NTFS ADS) | `ffind -e -f ntfs image.dd 67` |
| `-s` | Search slack space | `ffind -s image.dd 58493` |
| `-B` | Body file output format | `ffind -B image.dd 58493` |
| `-C` | CSV output format | `ffind -C image.dd 58493` |
| `-h` | Help | `ffind -h` |
| `-V` | Version | `ffind -V` |

---

## Common Workflows

### Basic Inode Lookup
```bash
ffind usb_image.dd 58493
# Output: /Documents/resume.doc
```

### Find Deleted File Paths
```bash
ffind -d usb_image.dd 58493
# Includes entries from deleted directories
```

### Batch Resolve from ils Output
```bash
ils image.dd | awk -F, 'NR>1{print $1}' | while read inode; do
    ffind -d image.dd "$inode"
done
```

### Cross-Reference with Other Tools
```bash
# 1. List files, get inodes
fls -r -m "/" image.dd

# 2. Find specific inode
ffind image.dd <inode_number>

# 3. Extract file content
icat image.dd <inode_number> > recovered_file
```

### Find Hard Links
```bash
ffind image.dd 12
# Multiple paths = hard links to same inode
```

---

## Quick Reference: What Each Flag Does

| You Want To... | Use Flag |
|----------------|----------|
| Find a file path from an inode | (no flags needed) |
| Also find paths in deleted directories | `-d` |
| Force filesystem type when auto-detect fails | `-f TYPE` |
| Get detailed debug information | `-v` |
| Search NTFS alternate data streams | `-e` |
| Search slack space for directory entries | `-s` |
| Output for timeline analysis | `-B` |
| Output as CSV for scripting | `-C` |

---

## Filesystem Types

| Code | Filesystem |
|------|------------|
| ext2 | Linux ext2 |
| ext3 | Linux ext3 |
| ext4 | Linux ext4 |
| fat | FAT (auto-detect FAT12/16/32) |
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
│   fsstat    │────▶│   fls   │────▶│  ffind  │────▶│  icat   │
│  (identify  │     │ (list   │     │ (inode  │     │(extract │
│  filesystem)│     │  files) │     │  → path)│     │ content)│
└─────────────┘     └─────────┘     └─────────┘     └─────────┘
```

---

## Exit Codes

| Code | Meaning |
|------|---------|
| `0` | Success — inode found |
| `1` | Error — inode not found |
| `2` | Error — invalid image/filesystem |

---

## One-Liners

```bash
# Find path for inode 58493
ffind usb_image.dd 58493

# Find including deleted, verbose
ffind -v -d usb_image.dd 58493

# Force ext4, find all hard links
ffind -f ext4 usb_image.dd 12

# CSV output for scripting
ffind -C usb_image.dd 58493

# Resolve all inodes from ils
ils -e image.dd | awk -F, 'NR>1{print $1}' | xargs -I{} ffind -d image.dd {}

# Find and recover in one shot
path=$(ffind -d image.dd 58493) && icat image.dd 58493 > "recovered_$(basename $path)"
```

---

## Troubleshooting

| Problem | Solution |
|---------|----------|
| No output for valid inode | Try `-d` flag, or specify `-f TYPE` |
| "Invalid filesystem" | Use `mmls` to find partition offset, then `-o OFFSET` |
| Slow on large images | Use `-f` to skip auto-detection |
| Multiple paths shown | Normal — these are hard links |
| Permission denied | Use `sudo` (rare, usually not needed) |

---

## Key Concepts

- **Inode**: Numeric identifier for file metadata (not the name)
- **Directory Entry**: Maps a name → inode number
- **Hard Links**: Multiple names pointing to same inode
- **Deleted Files**: Directory entry removed, inode may persist
- **ffind Purpose**: Reverse the name → inode mapping to find the path

---

**TSK Install**: `sudo apt install sleuthkit` | **Man Page**: `man ffind`
