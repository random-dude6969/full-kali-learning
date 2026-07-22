# sigfind — Quick Reference Cheatsheet

## Command Syntax
```
sigfind [options] <hex_signature> <image>
```

## Flags

| Flag | Description | Example |
|------|-------------|---------|
| `-o <num>` | Start at byte offset | `sigfind -o 512 FF D8 image.raw` |
| `-b <num>` | Block size (I/O) | `sigfind -b 1048576 FF D8 image.raw` |
| `-l <num>` | Limit search length | `sigfind -l 104857600 FF D8 image.raw` |
| `-B` | Big-endian match | `sigfind -B 89504E47 image.raw` |
| `-L` | Little-endian match | `sigfind -L 89504E47 image.raw` |
| `-V` | Version | `sigfind -V` |
| `-h` | Help | `sigfind -h` |

## Common File Signatures

| Type | Hex Signature | Notes |
|------|---------------|-------|
| JPEG | `FF D8 FF E0` | Standard JFIF |
| JPEG/EXIF | `FF D8 FF E1` | With EXIF metadata |
| PNG | `89 50 4E 47 0D 0A 1A 0A` | 8-byte magic |
| GIF89a | `47 49 46 38 39 61` | |
| PDF | `25 50 44 46 2D` | `%PDF-` in ASCII |
| ZIP/DOCX | `50 4B 03 04` | Also JAR, APK, XLSX |
| RAR | `52 61 72 21 1A 07 00` | |
| 7-Zip | `37 7A BC AF 27 1C` | |
| BMP | `42 4D` | `BM` in ASCII |
| TIFF (LE) | `49 49 2A 00` | Intel byte order |
| TIFF (BE) | `4D 4D 00 2A` | Motorola byte order |
| ELF | `7F 45 4C 46` | Linux binary |
| PE/EXE | `4D 5A` | Windows executable |
| Mach-O 64 | `FE ED FA CF` | macOS 64-bit |
| ext2/3/4 | `53 EF` | Superblock magic |
| NTFS | `EB 52 90 4E 54 46 53` | Boot sector |
| SWAP | `53 57 41 50 53 50 41 43 45 32` | Linux swap |
| SQLite | `53 51 4C 69 74 65 20 66 6F 72 6D 61 74 20 33` | `SQLite format 3` |
| DOC (OLE) | `D0 CF 11 E0 A1 B1 1A E1` | MS Office 97-2003 |

## Quick Examples

```bash
# Find all JPEGs in a raw image
sigfind FF\ D8\ FF\ E0 suspect.raw

# Find ZIPs starting at offset 1MB, search 100MB
sigfind -o 1048576 -l 104857600 50\ 4B\ 03\ 04 suspect.raw

# Find all PDFs, output count only
sigfind 25\ 50\ 44\ 46\ 2D suspect.raw | wc -l

# Find EXEs, extract offset values only
sigfind 4D\ 5A suspect.raw | awk '{print $NF}'

# Search in specific partition (use mmls first to find offset)
mmls suspect.raw
sigfind -o 2048 FF\ D8\ FF\ E0 suspect.raw
```

## Workflow: Partition-Aware Search

```bash
# 1. Find partition layout
mmls suspect.raw

# 2. Calculate partition byte offset (sector × 512)
# Partition at sector 2048 → offset 1048576

# 3. Scan within partition
sigfind -o 1048576 FF\ D8\ FF\ E0 suspect.raw
```

## Bash: Batch Multi-Signature Scan

```bash
for sig in "FF D8 FF E0:JPEG" "89 50 4E 47:PNG" "25 50 44 46 2D:PDF" \
           "50 4B 03 04:ZIP" "4D 5A:EXE" "7F 45 4C 46:ELF"; do
    HEX="${sig%%:*}"
    TYPE="${sig##*:}"
    COUNT=$(sigfind "$HEX" image.raw 2>/dev/null | wc -l)
    printf "%-10s %s\n" "$TYPE" "$COUNT"
done
```

## Python: Run sigfind from Script

```python
import subprocess

def sigfind_offsets(hex_pattern, image_path, offset=0):
    cmd = ["sigfind", hex_pattern, image_path]
    if offset > 0:
        cmd.insert(1, "-o")
        cmd.insert(2, str(offset))
    result = subprocess.run(cmd, capture_output=True, text=True)
    offsets = []
    for line in result.stdout.splitlines():
        if "Found signature at offset" in line:
            offsets.append(int(line.split()[-1]))
    return offsets

# Usage
matches = sigfind_offsets("FF D8 FF E0", "suspect.raw")
print(f"Found {len(matches)} JPEG signatures")
```

## Troubleshooting

| Problem | Fix |
|---------|-----|
| No matches found | Check partition offset with `mmls`; verify correct hex signature |
| Very slow on large images | Use `-b 1048576` for larger block size; narrow search with `-o` and `-l` |
| Too many false positives | Use longer signatures (4+ bytes); verify matches with `xxd` |
| Space in hex pattern fails | Escape: `FF\ D8\ FF\ E0` or quote: `"FF D8 FF E0"` |

## Key Points

- sigfind is filesystem-agnostic — works on any raw byte stream
- Does NOT modify the input image
- Output: "Found signature at offset <decimal_byte_offset>"
- Offset is relative to the start of the file/image (unless `-o` shifts it)
- Combine with `mmls`, `fls`, `icat`, and `blkcat` for full forensic workflow
