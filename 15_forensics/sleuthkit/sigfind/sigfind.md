---
tool_name: sigfind
display_name: "TSK sigfind"
mitre_tactic: "analysis"
mitre_tactic_id: "TA0007"
install_command: "sudo apt install sleuthkit"
github_repo: "https://github.com/sleuthkit/sleuthkit"
kali_tools_page: "https://www.kali.org/tools/sleuthkit/"
difficulty_level: "intermediate"
requires_root: false
gui_available: false
tags: ["forensics", "sleuthkit", "disk-image", "hex-signature", "magic-number", "file-carving", "analysis"]
related_tools: ["tsk_recover", "fls", "icat", "blkcat", "sorter"]
---

# sigfind — Hex Signature Discovery in Disk Images

## Chapter 1: Introduction & Overview

### What Is sigfind?

sigfind is a command-line forensic utility that is part of The Sleuth Kit (TSK), a comprehensive collection of filesystem forensic tools originally developed by Brian Carrier. sigfind searches a raw disk image or a specific file for a given hexadecimal byte sequence (commonly called a "magic number" or "file signature") and reports the byte offset where each match occurs. The tool operates at the raw byte level, bypassing filesystem structures entirely, making it indispensable for scenarios where the filesystem metadata is corrupted, overwritten, or intentionally destroyed.

### Why sigfind Matters in Digital Forensics

Every file type begins with a distinctive byte sequence. JPEG images start with `FF D8 FF`, PDF documents begin with `25 50 44 46 2D` (the ASCII string `%PDF-`), and ZIP archives open with `50 4B 03 04`. When an investigator encounters a raw disk image — whether from a seized hard drive, a memory dump, or a mobile device backup — sigfind can locate every occurrence of a specific signature without needing to understand the underlying filesystem layout. This capability is critical in several forensic contexts:

- **File Carving**: Before tools like scalpel or foremost can carve files, analysts sometimes need to verify signature locations across an image, especially when standard carving tools miss fragmented or non-standard files.
- **Steganography Detection**: Hidden data embedded within carrier files may begin at unexpected offsets. sigfind can scan for the signatures of hidden archives, images, or encrypted volumes.
- **Anti-Forensics Investigation**: Attackers may wipe filesystem metadata but forget to scrub embedded file signatures. sigfind can reveal these overlooked artifacts.
- **Corrupted Media Recovery**: When a filesystem is partially destroyed by hardware failure or deliberate formatting, sigfind locates surviving file headers that can guide manual recovery efforts.

### How sigfind Differs from Other TSK Tools

Unlike fls, which lists files by traversing filesystem metadata, or icat, which extracts a specific file by inode number, sigfind operates entirely at the byte level. It does not interpret filesystem structures — it reads the raw image sequentially and matches a user-supplied hex pattern against every byte position. This makes it filesystem-agnostic: it works on FAT12, FAT16, FAT32, NTFS, ext2/ext3/ext4, HFS+, APFS, UFS, and even unformatted or encrypted surfaces.

Compared to the Unix `strings` command, which extracts printable character sequences, sigfind operates on arbitrary binary patterns. Compared to `hexdump` or `xxd`, which display raw contents, sigfind automates the search across potentially multi-gigabyte images.

### Architecture and Design

sigfind reads the input image as a flat byte stream. It accepts a hex string argument (e.g., `FF D8 FF E0` for a JPEG file header) and scans the image from a user-specified offset to the end of the file (or to a user-specified limit). For every byte position where the pattern matches, sigfind prints the absolute offset within the image. The tool supports searching in big-endian or little-endian byte order, which matters when the signature itself encodes multi-byte integer values.

### Core Concepts

| Concept | Description |
|---------|-------------|
| **Magic Number** | A unique byte sequence at the start of a file that identifies its type (e.g., `4D 5A` for EXE/DLL). |
| **Byte Offset** | The position within the image (measured in bytes from the start) where a match is found. |
| **Block Size** | The unit of I/O when reading the image; defaults to the optimal value based on image size. |
| **Endianness** | Byte order interpretation — big-endian (network byte order) vs. little-endian (x86 native). |

### Use Cases Across Forensic Disciplines

| Discipline | Use Case |
|------------|----------|
| Digital forensics | Locate file headers in a wiped or reformatted disk image. |
| Incident response | Find embedded executables or scripts within a compromised system image. |
| Mobile forensics | Identify app database signatures in a raw NAND dump. |
| Memory forensics | Locate PE headers in a RAM capture to identify loaded executables. |
| Anti-forensics research | Verify that wiping procedures were complete and no signatures remain. |
| Malware analysis | Find embedded resources (DLLs, images, configs) packed inside a sample. |

---

## Chapter 2: Installation & Setup

### Installing sigfind via Package Manager

sigfind ships as part of The Sleuth Kit package. On Debian-based systems (including Kali Linux, Ubuntu, and Parrot OS):

```bash
sudo apt update
sudo apt install sleuthkit
```

After installation, verify the binary is available:

```bash
which sigfind
sigfind --help
```

### Building from Source

For the latest development version or to compile on unsupported platforms:

```bash
git clone https://github.com/sleuthkit/sleuthkit.git
cd sleuthkit
./bootstrap
./configure
make
sudo make install
sudo ldconfig
```

Build dependencies include `autoconf`, `automake`, `libtool`, `g++`, and `zlib1g-dev`.

### Verifying the Installation

```bash
# Check version information
sigfind -V

# Quick test: search for a common signature in any file
echo -n "PK" > /tmp/test.bin
echo -n "This is a test document" >> /tmp/test.bin
sigfind PK /tmp/test.bin
# Expected output: Found signature at offset 0
```

### Environment Considerations

sigfind works on any file that can be read as a byte stream — raw disk images (`.dd`, `.raw`, `.img`, `.E01` converted to raw), partition images, or individual files. It does not require root privileges to read files the current user has read access to, though forensic images are typically stored in directories requiring elevated permissions.

For large images (multi-terabyte), ensure sufficient RAM is available for the I/O cache. While sigfind itself has a small memory footprint, the operating system's page cache will benefit from available RAM when scanning large images sequentially.

### Kali Linux Integration

On Kali Linux, sigfind integrates with the broader forensic toolkit. Common workflows involve combining sigfind with other TSK utilities:

```bash
# Typical Kali forensics workflow
# 1. Image the suspect drive
sudo dd if=/dev/sdb of=/evidence/suspect.raw bs=4M status=progress

# 2. Identify partition offsets with mmls
mmls /evidence/suspect.raw

# 3. Search for file signatures in the raw image
sigfind FF\ D8\ FF\ E0 /evidence/suspect.raw
```

---

## Chapter 3: Basic Usage

### Command Syntax

```
sigfind [options] <hex_signature> <image>
```

Where `<hex_signature>` is a sequence of hexadecimal byte values (space-separated, with or without `0x` prefix) and `<image>` is the path to the disk image or file to search.

### Complete Flag Reference

| Flag | Long Form | Description | Example |
|------|-----------|-------------|---------|
| `-o <num>` | `--offset <num>` | Start searching at this byte offset within the image (default: 0). | `sigfind -o 512 FF D8 image.raw` |
| `-b <num>` | `--block-size <num>` | Set the block size for reading the image in chunks (bytes). Affects I/O performance. | `sigfind -b 65536 FF D8 image.raw` |
| `-l <num>` | `--length <num>` | Limit the search to this many bytes from the starting offset. | `sigfind -l 1048576 FF D8 image.raw` |
| `-B` | `--bigend` | Interpret the signature as a big-endian integer for matching. | `sigfind -B 89504E47 image.raw` |
| `-L` | `--littleend` | Interpret the signature as a little-endian integer for matching. | `sigfind -L 89504E47 image.raw` |
| `-V` | `--version` | Display the sigfind version information and exit. | `sigfind -V` |
| `-h` | `--help` | Display usage information and exit. | `sigfind -h` |

### Common File Signatures

| File Type | Hex Signature | ASCII Equivalent | Notes |
|-----------|---------------|------------------|-------|
| JPEG/JFIF | `FF D8 FF E0` | ÿØÿà | Standard JPEG with JFIF marker |
| JPEG/EXIF | `FF D8 FF E1` | ÿØÿÑ | JPEG with EXIF metadata |
| PNG | `89 50 4E 47 0D 0A 1A 0A` | .PNG.... | 8-byte magic number |
| GIF87a | `47 49 46 38 37 61` | GIF87a | GIF version 87a |
| GIF89a | `47 49 46 38 39 61` | GIF89a | GIF version 89a |
| PDF | `25 50 44 46 2D` | %PDF- | Always begins with this |
| ZIP/JAR/APK | `50 4B 03 04` | PK.. | PK zip format |
| RAR | `52 61 72 21 1A 07 00` | Rar!... | RAR archive |
| 7-Zip | `37 7A BC AF 27 1C` | 7z¼¯'.. | 7z compressed archive |
| TIFF (LE) | `49 49 2A 00` | II*. | Little-endian TIFF |
| TIFF (BE) | `4D 4D 00 2A` | MM.* | Big-endian TIFF |
| BMP | `42 4D` | BM | Windows bitmap |
| ELF | `7F 45 4C 46` | .ELF | Linux executable |
| PE/EXE | `4D 5A` | MZ | Windows executable |
| Mach-O (32) | `FE ED FA CE` | þíúÎ | macOS 32-bit binary |
| Mach-O (64) | `FE ED FA CF` | þíúÏ | macOS 64-bit binary |
| Linux swap | `53 57 41 50 53 50 41 43 45 32` | SWAPSPACE2 | Linux swap partition |
| ext2/ext3/ext4 | `53 EF` | Sï | Superblock magic |
| NTFS | `EB 52 90 4E 54 46 53` | ëR NTFS | NTFS boot sector |
| FAT boot sector | `EB 58 90` or `EB 3C 90` | ëX or ë< | FAT bootstrap jump |
| MySQL FRM | `FE 01` | þ. | MySQL table definition |

### Basic Example: Finding JPEG Images

```bash
# Search for JPEG file headers in a disk image
sigfind FF\ D8\ FF\ E0 suspect.raw
```

Expected output:

```
Found signature at offset 1048576
Found signature at offset 3145728
Found signature at offset 5242880
```

Each line indicates a byte offset (in decimal) from the start of the image where the JPEG signature was found. These offsets represent the beginning of potential JPEG files.

### Searching at a Specific Offset

```bash
# Start searching at offset 1 MB (1048576 bytes)
sigfind -o 1048576 FF\ D8\ FF\ E0 suspect.raw
```

### Limiting the Search Range

```bash
# Search only the first 100 MB of the image
sigfind -l 104857600 FF\ D8\ FF\ E0 suspect.raw
```

### Searching for PNG Images

```bash
# Search for the PNG magic number (8 bytes)
sigfind 89\ 50\ 4E\ 47\ 0D\ 0A\ 1A\ 0A suspect.raw
```

### Searching for PDF Files

```bash
# Search for PDF file headers
sigfind 25\ 50\ 44\ 46\ 2D suspect.raw
```

### Searching for ZIP Archives

```bash
# Search for ZIP/JAR/APK file headers
sigfind 50\ 4B\ 03\ 04 suspect.raw
```

### Using Byte Values with 0x Prefix

sigfind accepts hex values with or without the `0x` prefix:

```bash
# Both of these are equivalent
sigfind FF\ D8\ FF\ E0 suspect.raw
sigfind 0xFF\ 0xD8\ 0xFF\ 0xE0 suspect.raw
```

### Searching for Windows Executables

```bash
# Search for PE/EXE headers (MZ signature)
sigfind 4D\ 5A suspect.raw
```

Note: The two-byte pattern `4D 5A` is very common and will produce many matches. Consider combining with offset analysis — valid PE headers typically have the PE\0\0 signature at a known offset from the MZ header.

---

## Chapter 4: Intermediate Usage

### Bash Scripting with sigfind

#### Batch Signature Search Script

```bash
#!/bin/bash
# sigfind_batch.sh — Search for multiple file signatures in a disk image
# Usage: ./sigfind_batch.sh <image_file>

IMAGE="$1"

if [ -z "$IMAGE" ] || [ ! -f "$IMAGE" ]; then
    echo "Usage: $0 <image_file>"
    exit 1
fi

# Define signature names and their hex patterns
declare -A SIGNATURES=(
    ["JPEG"]="FF D8 FF E0"
    ["PNG"]="89 50 4E 47 0D 0A 1A 0A"
    ["GIF"]="47 49 46 38 39 61"
    ["PDF"]="25 50 44 46 2D"
    ["ZIP"]="50 4B 03 04"
    ["RAR"]="52 61 72 21 1A 07 00"
    ["EXE"]="4D 5A"
    ["ELF"]="7F 45 4C 46"
    ["BMP"]="42 4D"
    ["TIFF"]="49 49 2A 00"
    ["DOCX"]="50 4B 03 04"  # DOCX is a ZIP
    ["7z"]="37 7A BC AF 27 1C"
)

echo "=== Signature Scan Report ==="
echo "Image: $IMAGE"
echo "Size: $(stat -c%s "$IMAGE") bytes"
echo "Date: $(date)"
echo "================================"
echo ""

for TYPE in "${!SIGNATURES[@]}"; do
    HEX_SIG="${SIGNATURES[$TYPE]}"
    COUNT=$(sigfind $HEX_SIG "$IMAGE" 2>/dev/null | wc -l)
    echo "[$TYPE] Signature: $HEX_SIG"
    echo "  Matches found: $COUNT"
    if [ "$COUNT" -gt 0 ] && [ "$COUNT" -le 20 ]; then
        sigfind $HEX_SIG "$IMAGE" 2>/dev/null | while read line; do
            OFFSET=$(echo "$line" | awk '{print $NF}')
            echo "  -> Offset: $OFFSET"
        done
    elif [ "$COUNT" -gt 20 ]; then
        echo "  -> (showing first 20 of $COUNT matches)"
        sigfind $HEX_SIG "$IMAGE" 2>/dev/null | head -20 | while read line; do
            OFFSET=$(echo "$line" | awk '{print $NF}')
            echo "  -> Offset: $OFFSET"
        done
    fi
    echo ""
done
```

#### Verifying Signature Matches with icat

```bash
#!/bin/bash
# verify_jpeg.sh — Find JPEG signatures and attempt extraction with icat
# Usage: ./verify_jpeg.sh <image_file> <partition_offset>

IMAGE="$1"
OFFSET="${2:-0}"

# Find JPEG signatures
sigfind FF\ D8\ FF\ E0 "$IMAGE" | while read line; do
    MATCH_OFFSET=$(echo "$line" | awk '{print $NF}')
    
    # Extract 16 bytes starting at the match offset to verify
    dd if="$IMAGE" bs=1 skip="$MATCH_OFFSET" count=16 2>/dev/null | xxd | head -1
    
    # Use icat (if available) to attempt file extraction
    # This requires the filesystem offset
    echo "Potential JPEG at offset $MATCH_OFFSET"
    echo "---"
done
```

#### Converting Sigfind Offsets to Sector Numbers

```bash
#!/bin/bash
# offset_to_sector.sh — Convert byte offsets to sector numbers
# Usage: ./offset_to_sector.sh <image_file> [sector_size]

IMAGE="$1"
SECTOR_SIZE="${2:-512}"

echo "Converting sigfind offsets to sectors (sector size: $SECTOR_SIZE)"
echo ""

sigfind FF\ D8\ FF\ E0 "$IMAGE" | while read line; do
    OFFSET=$(echo "$line" | awk '{print $NF}')
    SECTOR=$((OFFSET / SECTOR_SIZE))
    REMAINDER=$((OFFSET % SECTOR_SIZE))
    echo "Offset $OFFSET -> Sector $SECTOR (remainder: $REMAINDER bytes into sector)"
done
```

### Python Scripting with sigfind

#### Using subprocess to Call sigfind

```python
#!/usr/bin/env python3
"""
sigfind_scanner.py — Automated multi-signature scanner using sigfind.
Searches a disk image for common file signatures and generates a report.
"""

import subprocess
import sys
import json
from datetime import datetime
from pathlib import Path


# Common file signatures: {type_name: hex_pattern}
SIGNATURES = {
    "JPEG":       "FF D8 FF E0",
    "JPEG_EXIF":  "FF D8 FF E1",
    "PNG":        "89 50 4E 47 0D 0A 1A 0A",
    "GIF89a":     "47 49 46 38 39 61",
    "PDF":        "25 50 44 46 2D",
    "ZIP":        "50 4B 03 04",
    "RAR":        "52 61 72 21 1A 07 00",
    "7z":         "37 7A BC AF 27 1C",
    "EXE_PE":     "4D 5A",
    "ELF":        "7F 45 4C 46",
    "BMP":        "42 4D",
    "TIFF_LE":    "49 49 2A 00",
    "TIFF_BE":    "4D 4D 00 2A",
    "MACHO_32":   "FE ED FA CE",
    "MACHO_64":   "FE ED FA CF",
    "SWAP":       "53 57 41 50 53 50 41 43 45 32",
    "EXT_FS":     "53 EF",
    "NTFS":       "EB 52 90 4E 54 46 53",
}


def run_sigfind(hex_pattern: str, image_path: str,
                offset: int = 0, length: int = 0) -> list:
    """Run sigfind and return a list of matching byte offsets."""
    cmd = ["sigfind", hex_pattern, image_path]
    if offset > 0:
        cmd.insert(1, "-o")
        cmd.insert(2, str(offset))
    if length > 0:
        cmd.insert(1, "-l")
        cmd.insert(2, str(length))

    try:
        result = subprocess.run(
            cmd, capture_output=True, text=True, timeout=300
        )
        offsets = []
        for line in result.stdout.strip().splitlines():
            if "Found signature at offset" in line:
                offset_val = line.split()[-1]
                offsets.append(int(offset_val))
        return offsets
    except (subprocess.TimeoutExpired, FileNotFoundError) as e:
        print(f"  [ERROR] sigfind failed for pattern {hex_pattern}: {e}",
              file=sys.stderr)
        return []


def scan_image(image_path: str) -> dict:
    """Scan a disk image for all known file signatures."""
    image_size = Path(image_path).stat().st_size
    report = {
        "image": image_path,
        "image_size": image_size,
        "scan_time": datetime.now().isoformat(),
        "findings": {}
    }

    print(f"Scanning: {image_path} ({image_size:,} bytes)")
    print(f"Patterns to check: {len(SIGNATURES)}")
    print("-" * 60)

    for file_type, hex_pattern in sorted(SIGNATURES.items()):
        offsets = run_sigfind(hex_pattern, image_path)
        report["findings"][file_type] = {
            "signature": hex_pattern,
            "count": len(offsets),
            "offsets": offsets
        }
        status = f"  {file_type}: {len(offsets)} match(es)"
        if offsets:
            status += f" [first: {offsets[0]:,}]"
        print(status)

    return report


def print_report(report: dict):
    """Print a formatted summary report."""
    print("\n" + "=" * 60)
    print("SCAN REPORT")
    print("=" * 60)
    print(f"Image:     {report['image']}")
    print(f"Size:      {report['image_size']:,} bytes")
    print(f"Scanned:   {report['scan_time']}")
    print("-" * 60)

    total_matches = sum(
        f["count"] for f in report["findings"].values()
    )
    print(f"Total matches across all signatures: {total_matches}")
    print()

    # Show only types with matches
    for ftype, data in sorted(report["findings"].items()):
        if data["count"] > 0:
            print(f"  {ftype:15s}  {data['count']:>5d} match(es)"
                  f"  sig=[{data['signature']}]")
            if data["count"] <= 10:
                for off in data["offsets"]:
                    print(f"                        offset: {off:>14,}")

    print("=" * 60)


def main():
    if len(sys.argv) < 2:
        print(f"Usage: {sys.argv[0]} <image_file>")
        print("Scans a disk image for common file signatures using sigfind.")
        sys.exit(1)

    image_path = sys.argv[1]
    if not Path(image_path).is_file():
        print(f"Error: File not found: {image_path}")
        sys.exit(1)

    report = scan_image(image_path)
    print_report(report)

    # Optionally save JSON report
    json_path = Path(image_path).stem + "_sigfind_report.json"
    with open(json_path, "w") as f:
        json.dump(report, f, indent=2)
    print(f"\nJSON report saved to: {json_path}")


if __name__ == "__main__":
    main()
```

#### Python Parser for sigfind Output

```python
#!/usr/bin/env python3
"""
parse_sigfind.py — Parse sigfind output into structured data.
Handles raw stdout from sigfind and converts to a Python list.
"""

import subprocess
import re
from dataclasses import dataclass, field
from typing import List, Optional


@dataclass
class SigfindResult:
    """Represents a single sigfind match."""
    offset: int
    hex_pattern: str
    image_path: str
    line_number: int = 0


def parse_sigfind_output(output: str, hex_pattern: str,
                         image_path: str) -> List[SigfindResult]:
    """Parse raw sigfind stdout into SigfindResult objects."""
    results = []
    pattern = re.compile(r"Found signature at offset (\d+)")
    for line_num, line in enumerate(output.splitlines(), 1):
        match = pattern.search(line)
        if match:
            results.append(SigfindResult(
                offset=int(match.group(1)),
                hex_pattern=hex_pattern,
                image_path=image_path,
                line_number=line_num
            ))
    return results


def search_and_parse(hex_pattern: str, image_path: str,
                     offset: int = 0) -> List[SigfindResult]:
    """Run sigfind and parse the output."""
    cmd = ["sigfind", hex_pattern, image_path]
    if offset > 0:
        cmd.insert(1, "-o")
        cmd.insert(2, str(offset))

    result = subprocess.run(cmd, capture_output=True, text=True)
    return parse_sigfind_output(result.stdout, hex_pattern, image_path)


def find_gaps(results: List[SigfindResult],
              max_gap: int = 1048576) -> List[tuple]:
    """Identify large gaps between consecutive matches.
    Large gaps may indicate unallocated space between files."""
    gaps = []
    for i in range(1, len(results)):
        gap = results[i].offset - results[i - 1].offset
        if gap > max_gap:
            gaps.append((results[i - 1].offset, results[i].offset, gap))
    return gaps


# Example usage
if __name__ == "__main__":
    results = search_and_parse("FF D8 FF E0", "/evidence/suspect.raw")
    print(f"Found {len(results)} JPEG signatures")
    for r in results:
        print(f"  Offset: {r.offset:>14,}")

    gaps = find_gaps(results, max_gap=10_485_760)
    if gaps:
        print(f"\nLarge gaps (>10MB) between matches:")
        for start, end, size in gaps:
            print(f"  {start:>14,} -> {end:>14,} ({size:>14,} bytes)")
```

---

## Chapter 5: Advanced Usage

### Endianness-Aware Searching

Some file signatures encode multi-byte integer values that change depending on byte order. sigfind supports both big-endian and little-endian interpretation:

```bash
# Search for a signature in big-endian byte order
sigfind -B 00000001 image.raw

# Search for the same value in little-endian byte order
sigfind -L 01000000 image.raw
```

This is particularly useful when searching for:
- Partition table signatures that may be stored as 32-bit integers
- Custom file headers that encode version numbers or flags as multi-byte values
- Database file signatures where the magic number is an integer field

### Combined Offset and Block Analysis

When analyzing a disk image with known partition layout, combine sigfind with mmls for targeted searching:

```bash
#!/bin/bash
# targeted_search.sh — Search for signatures within a specific partition
# Usage: ./targeted_search.sh <image> <partition_offset> <partition_size>

IMAGE="$1"
PART_OFFSET="$2"
PART_SIZE="$3"

echo "Searching partition at offset $PART_OFFSET, size $PART_SIZE"

# Adjust sigfind offset to account for the partition start
# sigfind -o starts relative to the beginning of the image
sigfind -o "$PART_OFFSET" -l "$PART_SIZE" FF\ D8\ FF\ E0 "$IMAGE"
```

### Scanning Memory Dumps for Loaded Executables

```bash
# Search for PE headers in a Windows memory dump
sigfind 4D\ 5A /evidence/memory.raw

# Search for ELF headers in a Linux memory dump
sigfind 7F\ 45\ 4C\ 46 /evidence/linux_mem.raw

# Search for Mach-O headers in a macOS memory dump
sigfind FE\ ED\ FA\ CF /evidence/mac_mem.raw
```

### Searching Encrypted or Compressed Images

sigfind can scan raw images even when they are encrypted or compressed at the filesystem level. If an encrypted volume container has a known file type embedded within it, the signatures of those inner files may be visible:

```bash
# Search for signatures in a VeraCrypt container (after decryption)
# First decrypt with veracrypt, then scan the mapped device
veracrypt --mount /evidence/container.hc /mnt/veracrypt
sigfind FF\ D8\ FF\ E0 /dev/mapper/veracrypt1
```

### Performance Tuning for Large Images

For images exceeding 100 GB, adjust the block size to optimize I/O:

```bash
# Use a larger block size for faster sequential reads
sigfind -b 1048576 FF\ D8\ FF\ E0 /evidence/large_image.raw

# For SSDs, a smaller block size may be more efficient
sigfind -b 4096 FF\ D8\ FF\ E0 /evidence/ssd_image.raw
```

### Integrating sigfind with awk for Offset Analysis

```bash
# Extract only offset values and perform arithmetic
sigfind FF\ D8\ FF\ E0 image.raw | awk '/Found signature at offset/ {
    offset = $NF;
    sector = int(offset / 512);
    print "Offset:", offset, "Sector:", sector;
}'
```

### Generating a Heatmap of Signature Distribution

```bash
#!/bin/bash
# signature_heatmap.sh — Visualize where signatures cluster in an image
# Divides the image into 1MB blocks and counts signatures per block

IMAGE="$1"
BLOCK_SIZE=1048576  # 1 MB
IMAGE_SIZE=$(stat -c%s "$IMAGE")
NUM_BLOCKS=$((IMAGE_SIZE / BLOCK_SIZE + 1))

echo "Image size: $IMAGE_SIZE bytes, Blocks: $NUM_BLOCKS"
echo ""

for ((i=0; i<NUM_BLOCKS; i++)); do
    START=$((i * BLOCK_SIZE))
    COUNT=$(sigfind -o "$START" -l "$BLOCK_SIZE" FF\ D8\ FF\ E0 "$IMAGE" 2>/dev/null | wc -l)
    if [ "$COUNT" -gt 0 ]; then
        BAR=""
        for ((j=0; j<COUNT && j<50; j++)); do
            BAR="${BAR}#"
        done
        printf "Block %5d (offset %12d): %3d %s\n" "$i" "$START" "$COUNT" "$BAR"
    fi
done
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: Forensic Triage — Quick File Type Inventory

**Context**: A seized laptop needs rapid triage. The analyst has a raw disk image of the primary partition and needs to quickly inventory what file types are present without mounting the filesystem.

**Approach**: Use sigfind to scan for common file signatures and generate a summary report.

```bash
# Step 1: Identify the partition layout
mmls /evidence/laptop.raw
# Output shows partition 2 (NTFS) starts at sector 2048 (offset 1048576)

# Step 2: Scan the partition for major file types
PART_OFFSET=1048576

echo "=== JPEG Images ==="
sigfind -o $PART_OFFSET FF\ D8\ FF\ E0 /evidence/laptop.raw | wc -l

echo "=== PDF Documents ==="
sigfind -o $PART_OFFSET 25\ 50\ 44\ 46\ 2D /evidence/laptop.raw | wc -l

echo "=== ZIP/Office Documents ==="
sigfind -o $PART_OFFSET 50\ 4B\ 03\ 04 /evidence/laptop.raw | wc -l

echo "=== Executables ==="
sigfind -o $PART_OFFSET 4D\ 5A /evidence/laptop.raw | wc -l
```

**Outcome**: The analyst identifies 847 JPEG images, 23 PDF documents, 156 ZIP archives (including Office documents), and 1,204 executables, providing a rapid overview of the system's contents.

### Scenario 2: Anti-Forensics Detection — Verifying Data Wiping

**Context**: A suspect claims to have used a secure deletion tool on their hard drive before seizure. The forensic team needs to verify whether any recoverable file signatures remain.

**Approach**: Scan the entire raw image for common file signatures. If the wiping was effective, no signatures should be found.

```bash
# Create a comprehensive wiping verification script
#!/bin/bash
IMAGE="$1"
EFFECTIVE=true

declare -A CHECK_SIGNS=(
    ["JPEG"]="FF D8 FF E0"
    ["PDF"]="25 50 44 46 2D"
    ["ZIP"]="50 4B 03 04"
    ["DOC"]="D0 CF 11 E0 A1 B1 1A E1"
    ["TXT"]="FF FE"  # UTF-16 LE BOM
    ["PNG"]="89 50 4E 47 0D 0A 1A 0A"
    ["EXE"]="4D 5A"
    ["SQLITE"]="53 51 4C 69 74 65 20 66 6F 72 6D 61 74 20 33"
)

echo "=== Wiping Verification Report ==="
echo "Image: $IMAGE"
echo "Date: $(date)"
echo ""

for TYPE in "${!CHECK_SIGNS[@]}"; do
    HEX="${CHECK_SIGNS[$TYPE]}"
    COUNT=$(sigfind "$HEX" "$IMAGE" 2>/dev/null | wc -l)
    if [ "$COUNT" -gt 0 ]; then
        echo "[FAIL] $TYPE: $COUNT signature(s) found — wiping incomplete"
        EFFECTIVE=false
    else
        echo "[PASS] $TYPE: No signatures found"
    fi
done

echo ""
if [ "$EFFECTIVE" = true ]; then
    echo "RESULT: No file signatures detected. Wiping appears effective."
else
    echo "RESULT: File signatures remain. Wiping was incomplete."
fi
```

### Scenario 3: Malware Embedded Resource Extraction

**Context**: A malware sample is suspected of containing embedded DLL files, configuration data, and secondary payloads packed within the main executable. The analyst needs to locate these embedded resources by their file signatures.

**Approach**: Scan the malware binary for signatures of common embedded file types.

```bash
# Step 1: Create a raw copy of the malware sample
cp /evidence/suspicious.exe /tmp/malware_raw.bin

# Step 2: Scan for embedded PE files (DLLs, other executables)
echo "=== Embedded PE files ==="
sigfind 4D\ 5A /tmp/malware_raw.bin

# Step 3: Scan for embedded archives (payloads)
echo "=== Embedded ZIP archives ==="
sigfind 50\ 4B\ 03\ 04 /tmp/malware_raw.bin

echo "=== Embedded RAR archives ==="
sigfind 52\ 61\ 72\ 21 /tmp/malware_raw.bin

# Step 4: Scan for embedded scripts
echo "=== Embedded VBScript (encoded) ==="
sigfind FF\ FE /tmp/malware_raw.bin  # UTF-16 BOM

# Step 5: Extract suspected payloads using dd
# If offset 12345 contains a suspected ZIP:
echo "Extracting suspected payload at offset 12345..."
dd if=/tmp/malware_raw.bin bs=1 skip=12345 count=524288 of=/tmp/extracted_payload.zip 2>/dev/null
file /tmp/extracted_payload.zip
```

---

## Chapter 7: Detection & Defense

### How Defenders Can Use sigfind

From a defensive perspective, sigfind and similar tools help blue teams:

- **Verify backup integrity**: Scan backup images for expected file signatures to confirm successful backup of critical files.
- **Detect data exfiltration artifacts**: Search for signatures of compressed archives or encrypted containers in system images that may indicate data staging.
- **Validate secure deletion**: Confirm that sensitive files have been properly wiped from decommissioned drives.
- **Incident response triage**: Quickly inventory file types on a compromised system to scope the investigation.

### Detecting sigfind Usage

sigfind itself leaves minimal forensic traces. However, investigators should be aware that:

- The tool reads the image sequentially, which generates a predictable I/O pattern.
- Command-line history (`.bash_history`) records sigfind invocations.
- The tool does not modify the image, so its use is undetectable from the image itself.

### Defensive Countermeasures Against Signature-Based Recovery

Adversaries who want to prevent signature-based recovery may:

- Overwrite file headers with random data (though this corrupts the files).
- Encrypt entire disk volumes (preventing raw signature scanning).
- Use deniable encryption (VeraCrypt hidden volumes).
- Fill unallocated space with junk data to obscure file boundaries.

---

## Chapter 8: Troubleshooting (FAQ)

### Q1: sigfind says "Found signature at offset 0" — is this always a valid file?

Not necessarily. The signature at offset 0 could be a coincidence — for example, if the disk image begins with bytes that happen to match a common pattern. To verify, extract a few hundred bytes starting at the reported offset and examine them with `xxd` or a hex editor:

```bash
dd if=image.raw bs=1 skip=0 count=256 2>/dev/null | xxd | head -16
```

If the bytes following the signature are consistent with the expected file format, the match is likely valid.

### Q2: Why does sigfind return no matches when I know files exist?

Common causes include:

1. **Wrong signature**: Verify the hex signature is correct for the target file type. Some formats have variant signatures (e.g., JPEG has multiple possible second bytes).
2. **Partition offset not accounted for**: If searching a raw disk image, the filesystem may start at a non-zero offset. Use `mmls` or `fdisk -l` to find the correct offset and use the `-o` flag.
3. **Filesystem metadata-only**: The file exists in the filesystem but its data blocks are in a different location than expected (e.g., compressed NTFS files, sparse files).
4. **Encryption**: If the filesystem is encrypted (BitLocker, LUKS), the file data is not in plaintext.

### Q3: sigfind is very slow on a 1TB image. How can I speed it up?

- Use a larger block size: `sigfind -b 1048576` (1 MB blocks).
- Limit the search to specific partition offsets using `-o` and `-l`.
- Search only for specific signature types rather than scanning for all possible patterns.
- Run the scan on an SSD rather than a spinning disk.

### Q4: How do I search for a signature that contains spaces in the hex pattern?

Use a backslash to escape the space character:

```bash
sigfind FF\ D8\ FF\ E0 image.raw
# Or equivalently:
sigfind "FF D8 FF E0" image.raw
```

### Q5: Can sigfind search for Unicode text patterns?

sigfind operates on raw bytes, so you can search for any byte sequence, including UTF-16 encoded text. For example, to search for the UTF-16 LE encoded string "SECRET" (`53 00 45 00 43 00 52 00 45 00 54 00`):

```bash
sigfind 53\ 00\ 45\ 00\ 43\ 00\ 52\ 00\ 45\ 00\ 54\ 00 image.raw
```

### Q6: What happens if the hex signature is longer than the image?

sigfind handles this gracefully — if the pattern length exceeds the remaining bytes in the image, no matches are found. The tool does not error out.

### Q7: Can I pipe sigfind output to other tools?

Yes. sigfind writes to stdout, so you can pipe its output:

```bash
# Count matches
sigfind FF\ D8\ FF\ E0 image.raw | wc -l

# Extract just the offsets
sigfind FF\ D8\ FF\ E0 image.raw | awk '{print $NF}'

# Save to a file
sigfind FF\ D8\ FF\ E0 image.raw > jpeg_offsets.txt
```

---

## Resources

### Official Documentation

- The Sleuth Kit Manual: https://sleuthkit.org/sleuthkit/man/
- TSK GitHub Repository: https://github.com/sleuthkit/sleuthkit
- Brian Carrier's Forensic Wiki: https://wiki.sleuthkit.org/

### Related TSK Tools

| Tool | Purpose |
|------|---------|
| fls | List files and directories in a filesystem image |
| icat | Extract a specific file by inode number |
| blkcat | Read raw blocks from a disk image |
| tsk_recover | Recover deleted files from a filesystem image |
| sorter | Sort and categorize files from a disk image |
| mmls | Display partition table layout |

### File Signature References

- File Magic Numbers Database: https://en.wikipedia.org/wiki/List_of_file_signatures
- TrID File Identifier: https://mark0.net/soft-trid-en.html
- Forensics Wiki File Signatures: https://forensicswiki.org/wiki/List_of_File_Signatures

### Books and Papers

- "File System Forensic Analysis" by Brian Carrier (Addison-Wesley, 2005)
- "The Sleuth Kit and Digital Forensics" — Carrier's academic publications
- NIST SP 800-86: Guide to Integrating Forensic Techniques into Incident Response

### Practice Images

- Digital Forensics Challenge (DFC) images
- NIST Computer Forensics Tool Testing (CFTT) datasets
- CyberDefenders.org challenge images
- CICIDS datasets for network-integrated forensic exercises
