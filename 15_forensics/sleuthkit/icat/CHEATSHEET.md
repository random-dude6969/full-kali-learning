# icat — Forensic File Content Extraction by Inode | CHEATSHEET

**Part of The Sleuth Kit (TSK)** | **MITRE: TA0007 Analysis** | **GitHub: sleuthkit/sleuthkit**

---

## Quick Syntax

```
icat [options] <image> <inode>
```

---

## Essential Flags

| Flag | Purpose | Example |
|------|---------|---------|
| `-f TYPE` | Force filesystem type | `icat -f ext4 image.dd 58493` |
| `-o OFFSET` | Byte offset to FS | `icat -o 2048 image.dd 58493` |
| `-i` | Show inode info only | `icat -i image.dd 58493` |
| `-s` | Extract slack space | `icat -s image.dd 58493` |
| `-e` | Extract NTFS ADS | `icat -e -f ntfs image.dd 67` |
| `-r` | Raw block extraction | `icat -r image.dd 58493` |
| `-v` | Verbose output | `icat -v image.dd 58493` |
| `-h` | Help | `icat -h` |
| `-V` | Version | `icat -V` |

---

## Common Workflows

### Basic File Extraction
```bash
icat usb_image.dd 58493 > recovered_file.docx
```

### Display File Contents
```bash
icat usb_image.dd 58493
```

### Check Inode Info
```bash
icat -i usb_image.dd 58493
```

### Extract with Path Preservation
```bash
# Find original path
ffind usb_image.dd 58493
# Output: /Documents/resume.docx

# Extract to that path
icat usb_image.dd 58493 > Documents/resume.docx
```

### Recover Deleted Files
```bash
# 1. Find deleted file inode
fls -r -d usb_image.dd | grep "secret.txt"
# Output: r/r * 58493: secret.txt

# 2. Extract
icat usb_image.dd 58493 > recovered_secret.txt
```

### Batch Recovery
```bash
# Extract all inodes from a list
while read inode; do
    icat usb_image.dd "$inode" > "file_$inode.bin"
done < inodes.txt
```

### Hash Verification
```bash
# Extract and hash
icat usb_image.dd 58493 | sha256sum

# Compare with known hash
icat usb_image.dd 58493 | sha256sum | grep -q "KNOWN_HASH" && echo "MATCH"
```

### Analyze Extracted File
```bash
icat usb_image.dd 58493 | file -
icat usb_image.dd 58493 | strings | head -20
icat usb_image.dd 58493 | xxd | head
```

---

## Complete TSK Workflow

```
┌─────────┐     ┌─────────┐     ┌─────────┐     ┌─────────┐
│ fsstat  │────▶│  fls    │────▶│ ffind   │────▶│  icat   │
│ (what   │     │ (list   │     │(inode → │     │(extract │
│  FS?)   │     │  files) │     │  path)  │     │ content)│
└─────────┘     └─────────┘     └─────────┘     └─────────┘
```

---

## Quick Reference: What Each Flag Does

| You Want To... | Use Flag |
|----------------|----------|
| Extract a file by inode | (no flags needed) |
| Force filesystem type | `-f TYPE` |
| Handle partitioned image | `-o OFFSET` |
| See inode metadata only | `-i` |
| Extract slack space | `-s` |
| Extract NTFS alternate stream | `-e` |
| Extract raw block data | `-r` |
| Get verbose output | `-v` |

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

## Exit Codes

| Code | Meaning |
|------|---------|
| `0` | Success |
| `1` | Error / inode not found |
| `2` | Invalid image/FS |

---

## One-Liners

```bash
# Extract file
icat usb_image.dd 58493 > file.bin

# Display contents
icat usb_image.dd 58493

# Show inode info
icat -i usb_image.dd 58493

# Force ext4
icat -f ext4 usb_image.dd 58493 > file.txt

# Extract from partition
icat -o 2048 disk.dd 58493 > file.txt

# Extract slack
icat -s usb_image.dd 58493 > slack.bin

# Extract NTFS ADS
icat -e -f ntfs image.dd 67 > ads.txt

# Extract and verify
icat usb_image.dd 58493 | sha256sum

# Extract and check type
icat usb_image.dd 58493 | file -

# Bulk extract from fls output
fls -r -d -m "/" usb_image.dd | awk -F'|' '{print $4}' | while read i; do icat usb_image.dd "$i" > "file_$i.bin"; done
```

---

## Troubleshooting

| Problem | Solution |
|---------|----------|
| Empty output file | Check inode with `istat`, try `-f TYPE` |
| Corrupted output | Verify filesystem type, check `istat` |
| Slow extraction | Use local disk, check file size with `istat` |
| "Inode not found" | Use `fls -d` to confirm inode exists |
| Wrong file type | Use `file` command to verify extraction |
| Can't read image | Use `fsstat` to verify filesystem |

---

## Verification Commands

```bash
# Verify extraction worked
file recovered_file.bin
ls -la recovered_file.bin
sha256sum recovered_file.bin

# Compare with original hash
sha256sum recovered_file.bin
# Should match known hash of original file

# Check file type
icat usb_image.dd 58493 | file -
```

---

## Key Concepts

- **icat** = forensic `cat` that reads by inode number
- Extracts **deleted files** that `cat` cannot access
- No mounting required — works on raw images
- Use with `fls` to get inodes, `ffind` to get paths
- Always **verify** extracted files with `file` and hashes

---

## icat vs Other Tools

| Tool | Purpose | Use When |
|------|---------|----------|
| `icat` | Extract single file by inode | You know the inode number |
| `tsk_recover` | Bulk extract all files | Need to recover everything |
| `cat` | Read mounted file | Filesystem is mounted normally |
| `dd` | Raw block copy | Need raw data, not file content |

---

**TSK Install**: `sudo apt install sleuthkit` | **Man Page**: `man icat`
