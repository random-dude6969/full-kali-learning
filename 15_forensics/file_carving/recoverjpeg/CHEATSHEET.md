# RecoverJPEG - Quick Reference Cheatsheet

## Overview

RecoverJPEG is a specialized tool for recovering JPEG images from damaged storage devices by scanning for JFIF/EXIF headers.

## Installation

```bash
sudo apt install recoverjpeg
```

## Basic Commands

| Command | Description |
|---------|-------------|
| `recoverjpeg /dev/sdb1` | Recover from USB drive |
| `recoverjpeg disk_image.dd` | Recover from disk image |
| `recoverjpeg -o output/ /dev/sdb1` | Recover to specific directory |
| `recoverjpeg -v /dev/sdb1` | Verbose output |

## Common Options

| Option | Description |
|--------|-------------|
| `-o DIR` | Output directory |
| `-v` | Verbose mode |
| `-h` | Help |
| `-V` | Version |

## Quick Workflow

```bash
# 1. Identify device
lsblk

# 2. Create forensic image
sudo dd if=/dev/sdb1 of=evidence.dd bs=4096 status=progress

# 3. Verify hash
md5sum /dev/sdb1 evidence.dd

# 4. Run recovery
sudo recoverjpeg -o recovered/ evidence.dd

# 5. Check results
ls -la recovered/*.jpg
```

## Forensic Image Formats

```bash
# E01 format
ewfmount evidence.E01 /mnt/ewf
sudo recoverjpeg /mnt/ewf/ewf1

# AFF format
affmount evidence.aff /mnt/aff
sudo recoverjpeg /mnt/aff/affdata
```

## Output Files

Recovered files are saved as:
```
file00001.jpg
file00002.jpg
file00003.jpg
...
```

## Batch Processing Script

```bash
#!/bin/bash
for device in /dev/sd[b-z]; do
    if [ -b "$device" ]; then
        echo "Recovering from $device..."
        sudo recoverjpeg -o "recovered_$(basename $device)" "$device"
    fi
done
```

## Common Issues & Fixes

| Issue | Solution |
|-------|----------|
| No JPEG files found | Try PhotoRec or Scalpel |
| Permission denied | Use `sudo` |
| Corrupted output | Files may be fragmented; try other tools |
| Slow performance | Use RAM disk for output |

## Alternatives

- **PhotoRec**: Multi-format recovery
- **Foremost**: Configurable file carving
- **Scalpel**: Fast file carving with custom headers

## Forensic Best Practices

1. Never work on original evidence
2. Always create forensic images first
3. Document chain of custody
4. Verify hashes before and after recovery
5. Use write blockers when possible

## Useful Combinations

```bash
# With safecopy for damaged media
sudo safecopy /dev/sdb1 evidence.dd
sudo recoverjpeg -o recovered/ evidence.dd

# With exiftool for metadata
exiftool recovered/*.jpg

# With jpeginfo for integrity check
jpeginfo -c recovered/*.jpg
```

## Quick Verification

```bash
# Check file validity
file recovered_file.jpg

# View EXIF data
exiftool recovered_file.jpg

# Check for corruption
jpeginfo -c recovered_file.jpg
```

## Memory Aids

- **Remember**: JPEG headers are `FF D8 FF` (SOI marker)
- **Always**: Create disk images before recovery
- **Never**: Run recovery directly on evidence device
- **Verify**: Check file integrity after recovery

## One-Liners

```bash
# Count recovered files
ls recovered/*.jpg | wc -l

# Find largest recovered files
ls -lhS recovered/*.jpg | head -10

# Extract EXIF dates
exiftool -DateTimeOriginal -filename recovered/*.jpg

# Find duplicate files
fdupes -r recovered/
```

---

**Version**: 1.0 | **Tool**: recoverjpeg | **Category**: File Carving
