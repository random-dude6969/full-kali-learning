# tsk_recover — Quick Reference Cheatsheet

## Command Syntax
```
sudo tsk_recover [options] <image> <output_dir>
```

## Flags

| Flag | Description | Example |
|------|-------------|---------|
| `-e <type>` | Evidence type: `raw` (default) or `raw_split` | `sudo tsk_recover -e raw image.raw /out/` |
| `-o <num>` | Byte offset to filesystem start | `sudo tsk_recover -o 1048576 image.raw /out/` |
| `-t <type>` | Force filesystem type (`ntfs`, `ext4`, `fat`, etc.) | `sudo tsk_recover -t ntfs image.raw /out/` |
| `-V` | Version | `tsk_recover -V` |
| `-h` | Help | `tsk_recover -h` |

## Quick Examples

```bash
# Basic recovery from a filesystem image
sudo tsk_recover -e raw image.raw /evidence/recovered/

# Recovery from a partition within a full disk image
sudo tsk_recover -e raw -o 1048576 disk.raw /evidence/recovered/

# Force filesystem type when auto-detect fails
sudo tsk_recover -t ext4 image.raw /evidence/recovered/

# Split raw image
sudo tsk_recover -e raw_split image.raw.000 /evidence/recovered/

# Preview deleted files before recovery
fls -d -r image.raw
```

## Workflow: Full Disk Partition Recovery

```bash
# 1. Find partition layout
mmls disk.raw

# 2. Identify filesystem type
mmstat disk.raw

# 3. Preview deleted entries
fls -d -r -o 2048 disk.raw

# 4. Recover
sudo tsk_recover -e raw -o 1048576 disk.raw /evidence/recovered/

# 5. Verify
find /evidence/recovered/ -type f | wc -l
```

## Workflow: Dual Recovery (Metadata + Carving)

```bash
# Metadata-based recovery (uses inode/MFT data)
sudo tsk_recover -e raw image.raw /evidence/tsk/

# Carving-based recovery (uses byte patterns)
foremost -i image.raw -o /evidence/carved/ -q

# Compare: find unique files from each method
find /evidence/tsk/ -type f -exec sha256sum {} \; | awk '{print $1}' | sort -u > tsk_hashes.txt
find /evidence/carved/ -type f -exec sha256sum {} \; | awk '{print $1}' | sort -u > carved_hashes.txt
echo "TSK-unique: $(comm -23 tsk_hashes.txt carved_hashes.txt | wc -l)"
echo "Carved-unique: $(comm -13 tsk_hashes.txt carved_hashes.txt | wc -l)"
```

## Workflow: Recovery with Verification

```bash
# 1. Recover
sudo tsk_recover -e raw image.raw /evidence/recovered/

# 2. Count recovered files
TOTAL=$(find /evidence/recovered/ -type f | wc -l)
echo "Total recovered: $TOTAL"

# 3. Check for empty/corrupt files
find /evidence/recovered/ -type f -empty | wc -l   # empty files
find /evidence/recovered/ -type f -size 0 | wc -l  # zero-byte files

# 4. File type breakdown
find /evidence/recovered/ -type f -exec file {} \; | awk -F: '{print $2}' | sort | uniq -c | sort -rn

# 5. Hash all recovered files
find /evidence/recovered/ -type f -exec sha256sum {} \; > recovered_hashes.txt
```

## Mapping Recovered Files to Original Names

tsk_recover uses auto-generated names: `img-<inode>-<seq>.<ext>`

```bash
# Show deleted entries with original names and inode numbers
fls -d -r image.raw

# Example output: "r/r * 12345: Documents/report.xlsx"
# Map inode 12345 to recovered file img-12345-0.xlsx

# Bulk mapping script
fls -d -r -m "" image.raw | grep "^r/r \*" | while IFS= read -r line; do
    INODE=$(echo "$line" | grep -oP '\d+(?=:)')
    NAME=$(echo "$line" | grep -oP ': \K.*')
    RECOVERED=$(find /evidence/recovered/ -name "img-${INODE}-*")
    if [ -n "$RECOVERED" ]; then
        echo "$RECOVERED -> $NAME"
    fi
done
```

## Supported Filesystems

| Filesystem | Support | Notes |
|------------|---------|-------|
| FAT12/16/32 | Full | USB drives, SD cards |
| NTFS | Full | Windows; recovers ADS, MFT entries |
| ext2/3/4 | Full | Linux filesystems |
| HFS+ | Full | Older macOS |
| UFS | Full | BSD systems |
| YAFFS2 | Full | Android NAND flash |
| ISO9660 | Full | CD/DVD images |
| ReiserFS | Partial | Some features unsupported |

## Bash: Batch Recovery

```bash
#!/bin/bash
IMAGE="$1"
OUT="/evidence/$(basename "$IMAGE" .raw)_recovered"
sudo tsk_recover -e raw "$IMAGE" "$OUT/"
echo "Recovered $(find "$OUT/" -type f | wc -l) files to $OUT"
```

## Python: Run tsk_recover from Script

```python
import subprocess

def recover(image, output_dir, offset=0):
    cmd = ["sudo", "tsk_recover", "-e", "raw"]
    if offset > 0:
        cmd.extend(["-o", str(offset)])
    cmd.extend([image, output_dir])
    result = subprocess.run(cmd, capture_output=True, text=True)
    return result.returncode == 0

recover("suspect.raw", "/evidence/recovered", offset=1048576)
```

## Troubleshooting

| Problem | Fix |
|---------|-----|
| 0 files recovered | Check offset with `mmls`; try `-t` to force filesystem type |
| Corrupt output files | Normal for partially overwritten data; try carving tools too |
| "Cannot determine filesystem" | Use `-t ntfs`, `-t ext4`, or `-t fat` to force type |
| Split image not working | Use `-e raw_split` and point to first segment file |
| Recovered files have strange names | Normal: format is `img-<inode>-<seq>.<ext>` |
| Very slow on large image | Extract specific partition with `dd` first, then recover |

## Key Points

- Requires root (`sudo`)
- Recovers deleted files using filesystem metadata (inodes/MFT)
- Always recover to a DIFFERENT drive than the source image
- Combine with `fls -d` to preview deletable files before recovery
- Combine with `foremost`/`scalpel` for maximum recovery (metadata + carving)
- Auto-generated filenames: `img-<inode>-<seq>.<ext>` — use `fls -d` to map back
- Does NOT work on encrypted filesystems without the decryption key
