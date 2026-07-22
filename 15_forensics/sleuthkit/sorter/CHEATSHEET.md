# sorter — Quick Reference Cheatsheet

## Command Syntax
```
sudo sorter [options] <image>
```

## Flags

| Flag | Description | Example |
|------|-------------|---------|
| `-a` | Include files with unrecognized type | `sudo sorter -a image.raw` |
| `-h <algo>` | Hash algorithm: `md5` (default), `sha1`, `sha256` | `sudo sorter -h sha256 image.raw` |
| `-k <file>` | Exclude known-good hashes (one hash per line) | `sudo sorter -k known.txt image.raw` |
| `-o <dir>` | Output directory | `sudo sorter -o /out image.raw` |
| `-s` | Print summary of file counts | `sudo sorter -s image.raw` |
| `-V` | Version | `sorter -V` |
| `-h` | Help | `sorter -h` |

## Quick Examples

```bash
# Basic sort by file type
sudo sorter -o /evidence/sorted suspect.raw

# Sort with SHA-256, include unknowns, show summary
sudo sorter -h sha256 -s -a -o /evidence/sorted suspect.raw

# Exclude known-good system files
sudo sorter -k known_hashes.txt -s -a -o /evidence/sorted suspect.raw

# Sort a partition (not full disk)
mmls suspect.raw  # find partition offset
dd if=suspect.raw bs=512 skip=2048 of=partition.raw
sudo sorter -o /evidence/sorted partition.raw
```

## Workflow: Partition-Aware Sort

```bash
# 1. Find partition layout
mmls suspect.raw

# 2. Extract partition to raw file
dd if=suspect.raw bs=512 skip=2048 count=10000000 of=part.raw

# 3. Sort the partition
sudo sorter -h sha256 -s -a -o /evidence/sorted part.raw
```

## Workflow: Known-File Exclusion (NSRL)

```bash
# Build known-hashes from a clean system
sudo find /usr /lib -type f -exec md5sum {} \; 2>/dev/null \
    | awk '{print $1}' | sort -u > known.txt

# Sort, excluding known-good files (shows only unknowns)
sudo sorter -k known.txt -s -a -o /evidence/unknowns suspect.raw
```

## Workflow: Full Triage Pipeline

```bash
# 1. Image hash for chain of custody
md5sum suspect.raw > evidence_md5.txt
sha256sum suspect.raw > evidence_sha256.txt

# 2. Partition layout
mmls suspect.raw > partition_table.txt

# 3. Sort all files
sudo sorter -h sha256 -s -a -o evidence_sorted suspect.raw \
    > sorter_summary.txt

# 4. File listing
fls -r -d suspect.raw > file_listing.txt

# 5. Recover deleted files
sudo tsk_recover -e raw suspect.raw evidence_recovered/
```

## Output Directory Structure

```
sorted/
├── application/
│   ├── pdf/          ← PDF documents
│   ├── zip/          ← ZIP, DOCX, XLSX, APK, JAR
│   ├── x-executable/ ← ELF binaries
│   ├── x-dosexec/    ← PE/EXE files
│   ├── x-rar/        ← RAR archives
│   └── x-7z-compressed/ ← 7z archives
├── image/
│   ├── jpeg/         ← JPEG photos
│   ├── png/          ← PNG images
│   └── gif/          ← GIF images
├── text/
│   ├── plain/        ← TXT files
│   └── html/         ← HTML files
├── video/
│   └── mp4/          ← Video files
└── unknown/          ← Unrecognized types (with -a flag)
```

## Bash: Batch Sort with Summary

```bash
#!/bin/bash
IMAGE="$1"
OUT="/evidence/sorted_$(date +%Y%m%d)"
sudo sorter -h sha256 -s -a -o "$OUT" "$IMAGE" > "$OUT/summary.txt"
echo "Sorted to: $OUT"
cat "$OUT/summary.txt"
```

## Python: Run sorter from Script

```python
import subprocess

def run_sorter(image, output_dir, hash_algo="sha256", known_file=None):
    cmd = ["sudo", "sorter", "-h", hash_algo, "-s", "-a", "-o", output_dir, image]
    if known_file:
        cmd.extend(["-k", known_file])
    result = subprocess.run(cmd, capture_output=True, text=True)
    return result.stdout

output = run_sorter("suspect.raw", "/evidence/sorted")
print(output)
```

## Troubleshooting

| Problem | Fix |
|---------|-----|
| Permission denied | Run with `sudo` |
| Cannot determine filesystem type | Extract partition with `dd` first (use `mmls` for offset) |
| Empty output | Check image with `fls`; ensure it's not encrypted |
| Too many files | Use `-k` to exclude known-good hashes |
| Slow on large image | Extract specific partition; use `md5` instead of `sha256` |

## Key Points

- Requires root (`sudo`) for raw disk image access
- Uses `file` command (libmagic) for type detection, not file extensions
- `-k` flag accepts one hash per line (plain hex, no filenames)
- Deleted files are NOT recovered — use `tsk_recover` for that
- Combine with `fls`, `icat`, `sigfind`, and `tsk_recover` for complete workflow
