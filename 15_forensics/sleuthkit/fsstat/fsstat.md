---
tool_name: fsstat
display_name: fsstat
mitre_tactic: "analysis"
mitre_tactic_id: "TA0007"
install_command: "sudo apt install sleuthkit"
github_repo: "https://github.com/sleuthkit/sleuthkit"
kali_tools_page: "https://www.kali.org/tools/sleuthkit/"
difficulty_level: "beginner"
requires_root: false
gui_available: false
tags: ["forensics", "filesystem", "sleuthkit", "tsk", "filesystem-info", "metadata", "analysis", "volume"]
related_tools: ["fls", "ffind", "icat", "ils", "mmls", "mmstat", "dbstat", "dls"]
---

# fsstat — Forensic Filesystem Information Display

## Chapter 1: Introduction & Overview

### What fsstat Does

fsstat is a command-line utility from The Sleuth Kit (TSK) that displays general filesystem information from a disk image or device. It reads the filesystem's superblock, journal, group descriptors, and other metadata structures to provide a comprehensive overview of how the filesystem is organized.

Think of fsstat as the first tool you run when examining a new piece of evidence. Before you list files with `fls` or extract data with `icat`, you need to understand what kind of filesystem you're dealing with, how it's structured, and what its parameters are. fsstat answers these fundamental questions:

- What filesystem type is this? (ext4, NTFS, FAT32, etc.)
- What is the block size?
- How many inodes are there?
- How much space is used vs. free?
- What is the volume label?
- When was the filesystem created and last modified?
- Is there a journal, and what state is it in?

### Why Filesystem Metadata Matters

The filesystem superblock and metadata contain critical forensic information:

1. **Timeline evidence**: Creation time, last mount time, last write time, and check time provide a timeline of filesystem activity.
2. **Usage patterns**: Free blocks, used blocks, and inode counts reveal how the filesystem was used.
3. **User identification**: The volume label and UUID can identify the device owner.
4. **Integrity indicators**: Journal state and unmounted-clean flags indicate whether the filesystem was properly shut down or crashed.
5. **Hidden partitions**: Understanding the filesystem layout helps identify hidden or hidden-in-plain-sight data.

### Position in The Sleuth Kit

fsstat is the starting point for filesystem analysis:

| Layer | Tools | Purpose |
|-------|-------|---------|
| Volume System | `mmls`, `mmstat` | Partition table analysis |
| **Filesystem Info** | **`fsstat`** | **Filesystem metadata overview** |
| Filesystem Metadata | `dbstat`, `dls` | Raw block-level analysis |
| Filesystem Analysis | `fls`, `ffind`, `ils` | File and inode operations |
| File Recovery | `icat`, `tsk_recover` | Data extraction |

fsstat provides the context needed before running other TSK tools. Knowing the filesystem type, block size, and layout is essential for interpreting results from `fls`, `ils`, and `icat`.

### Comparison with Standard Tools

| Feature | `dumpe2fs` | `ntfsinfo` | `fsstat` |
|---------|-----------|-----------|----------|
| Linux ext2/3/4 | Yes | No | Yes |
| Windows NTFS | No | Yes | Yes |
| FAT12/16/32 | No | No | Yes |
| HFS+ | No | No | Yes |
| Works on images | Limited | Limited | Yes |
| Unified output | No | No | Yes |

fsstat provides a consistent interface across all supported filesystems, making it invaluable for cross-platform investigations.

---

## Chapter 2: Installation & Setup

### Installing The Sleuth Kit

fsstat is included in the Sleuth Kit package:

```bash
# Update package lists
sudo apt update

# Install Sleuth Kit
sudo apt install sleuthkit

# Verify installation
which fsstat
fsstat -V
```

### Verifying the Installation

```bash
# Check fsstat is available
which fsstat

# Display version
fsstat -V

# List all TSK tools
dpkg -L sleuthkit | grep bin/

# Quick help check
fsstat -h
```

### Building from Source

For the latest development version:

```bash
# Install build dependencies
sudo apt install build-essential git autoconf automake libtool

# Clone repository
git clone https://github.com/sleuthkit/sleuthkit.git
cd sleuthkit

# Build
./bootstrap
./configure
make -j$(nproc)

# Install
sudo make install
sudo ldconfig
```

### Preparing a Working Environment

```bash
# Create case directory
mkdir -p ~/forensics/cases/case_001
cd ~/forensics/cases/case_001

# Create verified evidence copy
dc3dd if=/dev/sdb hash=sha256 log=evidence.log of=usb_image.dd

# Verify hash
sha256sum usb_image.dd

# Create working copy
cp usb_image.dd usb_image_work.dd

# Document chain of custody
cat > chain_of_custody.txt << EOF
Evidence: /dev/sdb
Image: usb_image.dd
Created: $(date -u)
Hash: $(sha256sum usb_image.dd | cut -d' ' -f1)
Analyst: $(whoami)
EOF
```

### Filesystem Support Matrix

fsstat supports these filesystem types:

| Filesystem | Full Support | Superblock | Journal | Notes |
|------------|:------------:|:----------:|:-------:|-------|
| ext2 | Yes | Yes | No | No journal |
| ext3 | Yes | Yes | Yes | Journaled ext2 |
| ext4 | Yes | Yes | Yes | Most common Linux FS |
| FAT12 | Yes | Yes | No | Floppy disks |
| FAT16 | Yes | Yes | No | Older removable |
| FAT32 | Yes | Yes | No | USB drives, SD cards |
| NTFS | Yes | Yes | Yes | Windows primary FS |
| HFS+ | Yes | Yes | Yes | macOS |
| UFS | Yes | Yes | No | BSD |
| YAFFS2 | Partial | Yes | No | Android NAND |

---

## Chapter 3: Basic Usage

### Command Syntax

```
fsstat [options] <image>
```

**Positional Arguments:**

| Argument | Description |
|----------|-------------|
| `<image>` | Path to filesystem image or device node |

### Complete Flag Reference

| Flag | Long Form | Description |
|------|-----------|-------------|
| `-f` | `--fstype` | Specify the filesystem type |
| `-o` | `--offset` | Byte offset to the filesystem |
| `-s` | `--super` | Display superblock information only |
| `-l` | `--log` | Display journal/log information |
| `-I` | `--inode` | Display inode information |
| `-i` | `--info` | Display filesystem information (default) |
| `-m` | `--meta` | Display metadata allocation info |
| `-b` | `--block` | Display block allocation info |
| `-d` | `--detail` | Display detailed information |
| `-h` | `--help` | Display help and exit |
| `-V` | `--version` | Print version and exit |
| `-v` | `--verbose` | Verbose output |
| `-t` | `--text` | Text output format (default) |
| `-C` | `--csv` | CSV output format |
| `-z` | `--zone` | Timezone for timestamps |

### Example 1: Basic Filesystem Information

```bash
# Display filesystem information
fsstat usb_image.dd
```

**Output:**
```
FILE SYSTEM INFORMATION
--------------------------------------------
File System Type: ext4
Volume Name:
Volume Serial Number: 5a3b8c2d-1e4f-5a6b-7c8d-9e0f1a2b3c4d
File System Type/ID: 0xEF53 (EXT3/EXT4)

FILE SYSTEM SIZE AND OFFSETS
--------------------------------------------
Block Size: 4096 bytes
Fragment Size: 4096 bytes
Blocks: 1228800
Fragments: 1228800
Reserved Blocks: 61440
Free Blocks: 876543
Free Fragments: 876543

INODE INFORMATION
--------------------------------------------
Number of Inodes: 307200
Free Inodes: 298765

TIMESTAMPS
--------------------------------------------
Last Mounted At: Mon Jan 16 14:30:00 2024
Last Written At: Mon Jan 16 15:45:22 2024
Last Checked At: Mon Jan 16 14:30:00 2024
Last Mount Time: Mon Jan 16 14:30:00 2024
Last Unmounted At: Mon Jan 16 15:50:00 2024

STATE FLAGS
--------------------------------------------
Clean: Yes
Mount Count: 12
Maximum Mount Count: -1
Check Interval: 0
Reserved Blocks UID: 0 (root)
Reserved Blocks GID: 0 (root)
```

### Example 2: Superblock Only

```bash
# Display only superblock information
fsstat -s usb_image.dd
```

### Example 3: Journal Information

```bash
# Display journal/log information
fsstat -l usb_image.dd
```

**Output (ext3/4):**
```
JOURNAL INFORMATION
--------------------------------------------
Journal Inode: 8
Journal Backup Inode: 0
Journal Device: 0x0
Journal Length: 33554432 bytes
Journal Sequence: 0x00001234
Journal Start: 1234
Journal Flags: 0x0

JOURNAL DEVICE INFO
--------------------------------------------
Device: 0x0
Mount ID: 0
Number of Transactions: 4567
Max Transaction Length: 0
```

### Example 4: Inode Allocation

```bash
# Display inode allocation information
fsstat -I usb_image.dd
```

### Example 5: Block Allocation

```bash
# Display block allocation information
fsstat -b usb_image.dd
```

**Output:**
```
BLOCK ALLOCATION INFORMATION
--------------------------------------------
Blocks Group 0:
  Block Bitmap: 7 - 7
  Inode Bitmap: 8 - 8
  Inode Table: 9 - 104
  Data Blocks: 105 - 16383
  Free Blocks: 12345
  Used Blocks: 4938
  Free Inodes: 7654
  Used Inodes: 1889

Blocks Group 1:
  Block Bitmap: 16384 - 16384
  Inode Bitmap: 16385 - 16385
  Inode Table: 16386 - 17409
  Data Blocks: 17410 - 32767
  Free Blocks: 13000
  Used Blocks: 15368
  Free Inodes: 7800
  Used Inodes: 1843
```

### Example 6: Detailed Output

```bash
# Full detailed filesystem information
fsstat -d usb_image.dd
```

### Example 7: Specify Filesystem Type

```bash
# Force filesystem type when auto-detection fails
fsstat -f ext4 usb_image.dd
```

### Example 8: Partition Offset

```bash
# For multi-partition images, specify offset
# First, find partition layout
mmls disk_image.dd

# Then analyze specific partition
fsstat -o 2048 disk_image.dd
```

### Exit Codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | Error or invalid filesystem |
| 2 | Cannot open image |

---

## Chapter 4: Intermediate Usage

### Bash Scripting with fsstat

#### Script 1: Automated Filesystem Profiling

```bash
#!/bin/bash
# fs_profile.sh — Generate a comprehensive filesystem profile

IMAGE="$1"
REPORT="fs_profile_$(basename "$IMAGE" | sed 's/\.[^.]*$//').txt"

if [ -z "$IMAGE" ]; then
    echo "Usage: $0 <filesystem_image>"
    exit 1
fi

echo "=== Filesystem Profile: $(basename "$IMAGE") ===" > "$REPORT"
echo "Generated: $(date -u)" >> "$REPORT"
echo "" >> "$REPORT"

# Basic filesystem info
echo "--- Filesystem Information ---" >> "$REPORT"
fsstat "$IMAGE" >> "$REPORT" 2>/dev/null
echo "" >> "$REPORT"

# Superblock details
echo "--- Superblock Details ---" >> "$REPORT"
fsstat -s "$IMAGE" >> "$REPORT" 2>/dev/null
echo "" >> "$REPORT"

# Journal information (if applicable)
echo "--- Journal Information ---" >> "$REPORT"
fsstat -l "$IMAGE" >> "$REPORT" 2>/dev/null
echo "" >> "$REPORT"

# Summary statistics
echo "--- Summary ---" >> "$REPORT"
echo "Image: $IMAGE" >> "$REPORT"
echo "Image Size: $(ls -lh "$IMAGE" | awk '{print $5}')" >> "$REPORT"
echo "MD5: $(md5sum "$IMAGE" | cut -d' ' -f1)" >> "$REPORT"
echo "SHA256: $(sha256sum "$IMAGE" | cut -d' ' -f1)" >> "$REPORT"

echo ""
echo "Profile complete: $REPORT"
```

**Usage:**
```bash
chmod +x fs_profile.sh
./fs_profile.sh usb_image.dd
```

#### Script 2: Multi-Partition Analysis

```bash
#!/bin/bash
# analyze_partitions.sh — Analyze all partitions in a disk image

DISK_IMAGE="$1"
OUTPUT_DIR="partition_analysis_$(date +%Y%m%d)"

if [ -z "$DISK_IMAGE" ]; then
    echo "Usage: $0 <disk_image>"
    exit 1
fi

mkdir -p "$OUTPUT_DIR"

echo "=== Analyzing: $(basename "$DISK_IMAGE") ==="

# Get partition layout
echo "Partition Layout:" | tee "$OUTPUT_DIR/layout.txt"
mmls "$DISK_IMAGE" | tee -a "$OUTPUT_DIR/layout.txt"

# Analyze each partition
mmls "$DISK_IMAGE" | awk 'NR>3 && $2 != "0s" {print $3}' | while read offset; do
    echo ""
    echo "=== Analyzing partition at offset: $offset ==="

    # Try to determine filesystem type
    FS_TYPE=$(fsstat -o "$((offset * 512))" "$DISK_IMAGE" 2>/dev/null | grep "File System Type:" | awk '{print $NF}')

    if [ -z "$FS_TYPE" ]; then
        echo "  Could not determine filesystem type"
        continue
    fi

    echo "  Filesystem type: $FS_TYPE"

    # Analyze filesystem
    fsstat -o "$((offset * 512))" "$DISK_IMAGE" > "$OUTPUT_DIR/partition_${offset}.txt" 2>/dev/null

    # Extract key information
    BLOCK_SIZE=$(grep "Block Size:" "$OUTPUT_DIR/partition_${offset}.txt" | awk '{print $3}')
    FREE_BLOCKS=$(grep "Free Blocks:" "$OUTPUT_DIR/partition_${offset}.txt" | awk '{print $3}')
    TOTAL_BLOCKS=$(grep "^Blocks:" "$OUTPUT_DIR/partition_${offset}.txt" | awk '{print $2}')

    echo "  Block size: $BLOCK_SIZE"
    echo "  Free blocks: $FREE_BLOCKS"
    echo "  Total blocks: $TOTAL_BLOCKS"
done

echo ""
echo "Analysis complete. Results in: $OUTPUT_DIR"
```

#### Script 3: Detect Filesystem Type for All Images

```bash
#!/bin/bash
# detect_fs.sh — Detect filesystem types for a batch of images

IMAGE_DIR="$1"

if [ -z "$IMAGE_DIR" ]; then
    echo "Usage: $0 <directory_with_images>"
    exit 1
fi

echo "Filesystem Type Detection Report"
echo "================================"
echo "Generated: $(date)"
echo ""

for image in "$IMAGE_DIR"/*.dd "$IMAGE_DIR"/*.raw "$IMAGE_DIR"/*.img; do
    [ -f "$image" ] || continue

    echo "File: $(basename "$image")"
    echo "Size: $(ls -lh "$image" | awk '{print $5}')"

    # Detect filesystem type
    FS_TYPE=$(fsstat "$image" 2>/dev/null | grep "File System Type:" | head -1)

    if [ -n "$FS_TYPE" ]; then
        echo "  $FS_TYPE"
    else
        echo "  Could not detect filesystem type"
    fi

    # Quick size info
    BLOCK_INFO=$(fsstat "$image" 2>/dev/null | grep -E "(Block Size|Free Blocks)" | head -2)
    if [ -n "$BLOCK_INFO" ]; then
        echo "$BLOCK_INFO" | sed 's/^/  /'
    fi

    echo ""
done
```

### Python Scripting with fsstat

#### Python 1: Filesystem Information Extractor

```python
#!/usr/bin/env python3
"""
fsstat_extractor.py — Extract and parse filesystem information using fsstat.
"""

import subprocess
import re
from dataclasses import dataclass, field
from typing import Optional, Dict, Any
from pathlib import Path
from datetime import datetime


@dataclass
class FilesystemInfo:
    """Structured representation of filesystem information."""
    fs_type: Optional[str] = None
    volume_name: Optional[str] = None
    volume_serial: Optional[str] = None
    block_size: Optional[int] = None
    fragment_size: Optional[int] = None
    total_blocks: Optional[int] = None
    free_blocks: Optional[int] = None
    total_inodes: Optional[int] = None
    free_inodes: Optional[int] = None
    last_mounted: Optional[str] = None
    last_written: Optional[str] = None
    last_checked: Optional[str] = None
    clean: Optional[bool] = None
    mount_count: Optional[int] = None
    max_mount_count: Optional[int] = None
    reserved_blocks_uid: Optional[int] = None
    reserved_blocks_gid: Optional[int] = None
    journal_inode: Optional[int] = None
    journal_length: Optional[int] = None
    raw_output: str = ""


class FsstatExtractor:
    """Extractor class for fsstat command output."""

    def __init__(self, image_path: str, filesystem_type: Optional[str] = None,
                 offset: Optional[int] = None):
        self.image_path = Path(image_path)
        self.filesystem_type = filesystem_type
        self.offset = offset
        self._validate_image()

    def _validate_image(self):
        if not self.image_path.exists():
            raise FileNotFoundError(f"Image not found: {self.image_path}")

    def get_filesystem_info(self) -> FilesystemInfo:
        """
        Extract comprehensive filesystem information.

        Returns:
            FilesystemInfo object with parsed data
        """
        cmd = ["fsstat"]

        if self.filesystem_type:
            cmd.extend(["-f", self.filesystem_type])

        if self.offset is not None:
            cmd.extend(["-o", str(self.offset)])

        cmd.append(str(self.image_path))

        result = subprocess.run(cmd, capture_output=True, text=True, timeout=60)

        info = FilesystemInfo()
        info.raw_output = result.stdout

        # Parse the output
        self._parse_output(result.stdout, info)

        return info

    def _parse_output(self, output: str, info: FilesystemInfo):
        """Parse fsstat output into FilesystemInfo object."""
        lines = output.split("\n")

        for line in lines:
            line = line.strip()

            # File System Type
            if "File System Type:" in line:
                match = re.search(r"File System Type:\s*(.+)", line)
                if match:
                    info.fs_type = match.group(1).strip()

            # Volume Name
            elif "Volume Name:" in line:
                match = re.search(r"Volume Name:\s*(.*)", line)
                if match:
                    info.volume_name = match.group(1).strip() or None

            # Volume Serial Number
            elif "Volume Serial Number:" in line:
                match = re.search(r"Volume Serial Number:\s*(.+)", line)
                if match:
                    info.volume_serial = match.group(1).strip()

            # Block Size
            elif "Block Size:" in line:
                match = re.search(r"Block Size:\s*(\d+)", line)
                if match:
                    info.block_size = int(match.group(1))

            # Fragment Size
            elif "Fragment Size:" in line:
                match = re.search(r"Fragment Size:\s*(\d+)", line)
                if match:
                    info.fragment_size = int(match.group(1))

            # Total Blocks
            elif line.startswith("Blocks:") or "Blocks:" in line:
                match = re.search(r"Blocks:\s*(\d+)", line)
                if match:
                    info.total_blocks = int(match.group(1))

            # Free Blocks
            elif "Free Blocks:" in line:
                match = re.search(r"Free Blocks:\s*(\d+)", line)
                if match:
                    info.free_blocks = int(match.group(1))

            # Number of Inodes
            elif "Number of Inodes:" in line:
                match = re.search(r"Number of Inodes:\s*(\d+)", line)
                if match:
                    info.total_inodes = int(match.group(1))

            # Free Inodes
            elif "Free Inodes:" in line:
                match = re.search(r"Free Inodes:\s*(\d+)", line)
                if match:
                    info.free_inodes = int(match.group(1))

            # Last Mounted At
            elif "Last Mounted At:" in line:
                match = re.search(r"Last Mounted At:\s*(.+)", line)
                if match:
                    info.last_mounted = match.group(1).strip()

            # Last Written At
            elif "Last Written At:" in line:
                match = re.search(r"Last Written At:\s*(.+)", line)
                if match:
                    info.last_written = match.group(1).strip()

            # Last Checked At
            elif "Last Checked At:" in line:
                match = re.search(r"Last Checked At:\s*(.+)", line)
                if match:
                    info.last_checked = match.group(1).strip()

            # Clean flag
            elif "Clean:" in line:
                info.clean = "Yes" in line

            # Mount Count
            elif "Mount Count:" in line:
                match = re.search(r"Mount Count:\s*(\d+|-1)", line)
                if match:
                    val = match.group(1)
                    info.mount_count = int(val) if val != "-1" else None

            # Journal Inode
            elif "Journal Inode:" in line:
                match = re.search(r"Journal Inode:\s*(\d+)", line)
                if match:
                    info.journal_inode = int(match.group(1))

            # Journal Length
            elif "Journal Length:" in line:
                match = re.search(r"Journal Length:\s*(\d+)", line)
                if match:
                    info.journal_length = int(match.group(1))

    def get_usage_statistics(self) -> Dict[str, Any]:
        """
        Calculate filesystem usage statistics.

        Returns:
            Dictionary with usage metrics
        """
        info = self.get_filesystem_info()

        stats = {
            "filesystem_type": info.fs_type,
            "volume_name": info.volume_name,
            "block_size": info.block_size,
        }

        if info.total_blocks and info.free_blocks:
            used_blocks = info.total_blocks - info.free_blocks
            stats["total_blocks"] = info.total_blocks
            stats["used_blocks"] = used_blocks
            stats["free_blocks"] = info.free_blocks
            stats["usage_percent"] = (used_blocks / info.total_blocks) * 100

        if info.total_inodes and info.free_inodes:
            used_inodes = info.total_inodes - info.free_inodes
            stats["total_inodes"] = info.total_inodes
            stats["used_inodes"] = used_inodes
            stats["free_inodes"] = info.free_inodes
            stats["inode_usage_percent"] = (used_inodes / info.total_inodes) * 100

        if info.block_size and info.total_blocks:
            total_size_bytes = info.block_size * info.total_blocks
            stats["total_size_bytes"] = total_size_bytes
            stats["total_size_gb"] = total_size_bytes / (1024**3)

        return stats

    def export_json(self, output_path: str):
        """Export filesystem information to JSON."""
        import json

        info = self.get_filesystem_info()
        stats = self.get_usage_statistics()

        output = {
            "image_path": str(self.image_path),
            "analysis_timestamp": datetime.utcnow().isoformat(),
            "filesystem_info": {
                "type": info.fs_type,
                "volume_name": info.volume_name,
                "volume_serial": info.volume_serial,
                "block_size": info.block_size,
                "total_blocks": info.total_blocks,
                "free_blocks": info.free_blocks,
                "total_inodes": info.total_inodes,
                "free_inodes": info.free_inodes,
                "last_mounted": info.last_mounted,
                "last_written": info.last_written,
                "clean": info.clean,
                "journal_inode": info.journal_inode,
            },
            "usage_statistics": stats
        }

        with open(output_path, "w") as f:
            json.dump(output, f, indent=2)


def main():
    import argparse

    parser = argparse.ArgumentParser(description="Extract filesystem information")
    parser.add_argument("image", help="Path to filesystem image")
    parser.add_argument("-f", "--fstype", help="Filesystem type")
    parser.add_argument("-o", "--offset", type=int, help="Byte offset to filesystem")
    parser.add_argument("--json", help="Export to JSON file")
    parser.add_argument("--summary", action="store_true", help="Show summary only")

    args = parser.parse_args()

    extractor = FsstatExtractor(args.image, args.fstype, args.offset)

    if args.json:
        extractor.export_json(args.json)
        print(f"Exported to {args.json}")
    elif args.summary:
        stats = extractor.get_usage_statistics()
        print(f"Filesystem: {stats.get('filesystem_type', 'Unknown')}")
        print(f"Volume: {stats.get('volume_name', 'N/A')}")
        print(f"Block Size: {stats.get('block_size', 'N/A')} bytes")
        if "usage_percent" in stats:
            print(f"Space Usage: {stats['usage_percent']:.1f}%")
        if "inode_usage_percent" in stats:
            print(f"Inode Usage: {stats['inode_usage_percent']:.1f}%")
        if "total_size_gb" in stats:
            print(f"Total Size: {stats['total_size_gb']:.2f} GB")
    else:
        info = extractor.get_filesystem_info()
        print(info.raw_output)


if __name__ == "__main__":
    main()
```

**Usage:**
```bash
# Basic information
python3 fsstat_extractor.py usb_image.dd

# Summary view
python3 fsstat_extractor.py --summary usb_image.dd

# Export to JSON
python3 fsstat_extractor.py --json fs_info.json usb_image.dd

# With filesystem type specified
python3 fsstat_extractor.py -f ext4 usb_image.dd
```

---

## Chapter 5: Advanced Usage

### Advanced Flag Combinations

```bash
# Complete filesystem profile
fsstat -d -l -s usb_image.dd

# Journal analysis only
fsstat -l usb_image.dd

# Block allocation details
fsstat -b usb_image.dd

# Inode allocation details
fsstat -I usb_image.dd

# Superblock with journal
fsstat -s -l usb_image.dd
```

### Working with Multi-Partition Images

```bash
# Identify partition layout
mmls disk_image.dd

# Analyze each partition
# Partition 1 (offset 2048 sectors)
fsstat -o $((2048 * 512)) disk_image.dd

# Partition 2 (offset 206848 sectors)
fsstat -o $((206848 * 512)) disk_image.dd
```

### Extracting Specific Metadata

```bash
# Extract superblock information to file
fsstat -s usb_image.dd > superblock_info.txt

# Extract journal information
fsstat -l usb_image.dd > journal_info.txt

# Extract block allocation
fsstat -b usb_image.dd > block_allocation.txt
```

### Combining with Other TSK Tools

```bash
# Complete forensic analysis workflow
echo "=== Step 1: Filesystem Information ==="
fsstat usb_image.dd

echo ""
echo "=== Step 2: File Listing ==="
fls -r -m "/" usb_image.dd

echo ""
echo "=== Step 3: Inode Analysis ==="
ils usb_image.dd

echo ""
echo "=== Step 4: Timeline Generation ==="
fls -r -m "/" usb_image.dd > body_file.txt
ils -e -m usb_image.dd >> body_file.txt
mactime -b body_file.txt -d timeline/
```

### Batch Processing Multiple Images

```bash
#!/bin/bash
# batch_fsstat.sh — Analyze filesystems for multiple images

IMAGE_DIR="$1"
OUTPUT_DIR="fsstat_results_$(date +%Y%m%d)"

mkdir -p "$OUTPUT_DIR"

for image in "$IMAGE_DIR"/*.dd "$IMAGE_DIR"/*.raw "$IMAGE_DIR"/*.E01; do
    [ -f "$image" ] || continue

    basename=$(basename "$image" | sed 's/\.[^.]*$//')
    echo "Analyzing: $basename"

    # Generate full filesystem report
    fsstat "$image" > "$OUTPUT_DIR/${basename}_full.txt" 2>/dev/null

    # Generate summary
    {
        echo "=== $basename ==="
        echo "Filesystem Type: $(fsstat "$image" 2>/dev/null | grep "File System Type:" | head -1)"
        echo "Block Size: $(fsstat "$image" 2>/dev/null | grep "Block Size:" | head -1)"
        echo "Last Written: $(fsstat "$image" 2>/dev/null | grep "Last Written At:" | head -1)"
    } > "$OUTPUT_DIR/${basename}_summary.txt"
done

echo "Batch analysis complete. Results in: $OUTPUT_DIR"
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: Initial Triage of Unknown Evidence

**Background**: A new evidence item arrives at the lab. Before any detailed analysis, you need to understand what you're working with.

**Investigation Steps:**

```bash
# 1. Verify image integrity
echo "=== Evidence Verification ==="
sha256sum evidence.dd
# Compare with documented hash

# 2. Identify partition layout
echo ""
echo "=== Partition Layout ==="
mmls evidence.dd

# 3. For each partition, identify filesystem
echo ""
echo "=== Filesystem Identification ==="
for offset in $(mmls evidence.dd | awk 'NR>3{print $3}' | grep -v "0s"); do
    echo "Partition at sector $offset:"
    fsstat -o $((offset * 512)) evidence.dd 2>/dev/null | head -5
    echo ""
done

# 4. Analyze primary filesystem in detail
echo "=== Primary Filesystem Analysis ==="
fsstat evidence.dd

# 5. Document findings
{
    echo "Evidence Triage Report"
    echo "====================="
    echo "Image: evidence.dd"
    echo "Date: $(date -u)"
    echo ""
    echo "Partition Layout:"
    mmls evidence.dd
    echo ""
    echo "Primary Filesystem:"
    fsstat evidence.dd
} > triage_report.txt
```

### Scenario 2: Filesystem Corruption Analysis

**Background**: A disk image shows signs of corruption. You need to determine the extent of damage and potential for recovery.

**Investigation Steps:**

```bash
# 1. Basic filesystem check
echo "=== Filesystem Health Check ==="
fsstat -d corrupted.dd

# 2. Check for clean unmount flag
CLEAN=$(fsstat corrupted.dd 2>/dev/null | grep "Clean:" | awk '{print $2}')
echo "Clean unmount: $CLEAN"

# 3. Analyze journal state
echo ""
echo "=== Journal State ==="
fsstat -l corrupted.dd

# 4. Check inode allocation for anomalies
echo ""
echo "=== Inode Allocation ==="
fsstat -I corrupted.dd

# 5. Block allocation analysis
echo ""
echo "=== Block Allocation ==="
fsstat -b corrupted.dd

# 6. Attempt file listing to see what's accessible
echo ""
echo "=== Accessible Files ==="
fls -r corrupted.dd 2>/dev/null | head -50

# 7. Document corruption assessment
{
    echo "Corruption Assessment"
    echo "====================="
    echo "Clean flag: $CLEAN"
    echo "Files accessible: $(fls corrupted.dd 2>/dev/null | wc -l)"
    echo "Journal state:"
    fsstat -l corrupted.dd 2>/dev/null | head -10
} > corruption_report.txt
```

### Scenario 3: Multi-Platform Evidence Analysis

**Background**: An investigation involves evidence from multiple operating systems — a Linux server, a Windows workstation, and a macOS laptop.

**Investigation Steps:**

```bash
# 1. Analyze Linux server image
echo "=== Linux Server Filesystem ==="
fsstat linux_server.dd

# 2. Analyze Windows workstation image
echo "=== Windows Workstation Filesystem ==="
fsstat windows_workstation.dd

# 3. Analyze macOS laptop image
echo "=== macOS Laptop Filesystem ==="
fsstat macos_laptop.dd

# 4. Compare filesystem characteristics
echo ""
echo "=== Comparative Analysis ==="
echo "Linux: $(fsstat linux_server.dd 2>/dev/null | grep "File System Type:" | head -1)"
echo "Windows: $(fsstat windows_workstation.dd 2>/dev/null | grep "File System Type:" | head -1)"
echo "macOS: $(fsstat macos_laptop.dd 2>/dev/null | grep "File System Type:" | head -1)"

# 5. Check last activity timestamps
echo ""
echo "=== Last Activity ==="
echo "Linux last write: $(fsstat linux_server.dd 2>/dev/null | grep "Last Written At:" | head -1)"
echo "Windows last write: $(fsstat windows_workstation.dd 2>/dev/null | grep "Last Written At:" | head -1)"
echo "macOS last write: $(fsstat macos_laptop.dd 2>/dev/null | grep "Last Written At:" | head -1)"
```

---

## Chapter 7: Detection & Defense

### Understanding Filesystem Metadata for Defenders

fsstat reveals information that attackers might want to hide:

1. **Timestamps**: Creation, modification, and access times can reveal when malicious activity occurred.
2. **Journal state**: A clean journal after suspicious activity might indicate tampering.
3. **Free space**: Unusual free space patterns could indicate hidden data.
4. **Volume labels**: Identifying information about the device owner.

### Anti-Forensics Techniques

Attackers may try to manipulate filesystem metadata:

| Technique | Effect on fsstat | Detection Method |
|-----------|------------------|------------------|
| Timestomping | Modifies timestamps | Cross-reference multiple timestamp sources |
| Journal clearing | Removes journal data | Check journal state flag |
| Superblock modification | Changes filesystem parameters | Compare with expected values |
| Full disk encryption | Prevents fsstat from reading | Requires decryption |
| Filesystem wiping | Destroys all metadata | fsstat fails to identify FS |

### Defensive Monitoring

```bash
# Monitor filesystem changes (Linux)
inotifywait -m -r /home /tmp /var

# Check filesystem integrity regularly
e2fsck -n /dev/sda2  # Non-destructive check
dumpe2fs /dev/sda2 > fs_backup.txt  # Backup superblock info

# Compare current state with baseline
diff fs_current.txt fs_baseline.txt
```

### Forensic Readiness

```bash
# Create filesystem baseline
fsstat /dev/sda2 > fs_baseline.txt

# Periodic checks
fsstat /dev/sda2 > fs_current.txt
diff fs_baseline.txt fs_current.txt

# Document filesystem state before and after incidents
```

---

## Chapter 8: Troubleshooting

### FAQ

**Q1: fsstat says "invalid filesystem" or cannot determine filesystem type.**

A: Common causes and solutions:
- Wrong offset for multi-partition image: Use `mmls` to find correct offset.
- Corrupted image: Try `-f` to force filesystem type.
- Non-raw image format: Convert E01 or other formats to raw first.
- Encrypted filesystem: Decrypt before analysis.

```bash
# Diagnostic steps
mmls disk_image.dd  # Find partitions
fsstat -f ext4 -o 2048 disk_image.dd  # Force type and offset
```

**Q2: fsstat shows different information than expected.**

A: This can happen when:
- Working with a copy that's been modified.
- The filesystem was modified between imaging and analysis.
- Multiple filesystems exist on the same image.
- Auto-detection selected wrong filesystem type.

```bash
# Verify with forced type
fsstat -f ext4 usb_image.dd

# Check for other filesystems
mmls disk_image.dd
```

**Q3: fsstat is slow on large images.**

A: fsstat reads the entire superblock and metadata structures. For very large images:
- Specify filesystem type with `-f` to skip auto-detection.
- Use local disk, not network storage.
- The initial run may be slow due to metadata indexing.

```bash
# Speed improvement
fsstat -f ext4 usb_image.dd
```

**Q4: How do I interpret the journal information?**

A: Journal information tells you about filesystem transaction logging:
- **Journal Inode**: The inode containing the journal data.
- **Journal Length**: Size of the journal in bytes.
- **Journal Start**: Current position in the journal.
- **Clean flag**: Whether the filesystem was unmounted cleanly.

A dirty journal (Clean: No) may indicate a crash or improper shutdown.

**Q5: What's the difference between fsstat and dumpe2fs?**

A: `dumpe2fs` is Linux-specific and only works with ext2/3/4 filesystems. `fsstat` is part of TSK and works with multiple filesystem types (ext2/3/4, FAT, NTFS, HFS+). For ext filesystems, `dumpe2fs` provides more detail, but `fsstat` is more versatile for cross-platform investigations.

**Q6: Can fsstat recover deleted files?**

A: No. fsstat only displays filesystem metadata and information. To recover deleted files, use:
- `fls -d` to list deleted file names
- `icat` to extract file contents by inode
- `tsk_recover` for bulk recovery

---

## Resources

### Official Documentation

- **TSK Manual**: https://sleuthkit.org/sleuthkit/man/
- **fsstat Man Page**: `man fsstat`
- **GitHub**: https://github.com/sleuthkit/sleuthkit
- **Kali Tools**: https://www.kali.org/tools/sleuthkit/

### Books and References

- *File System Forensic Analysis* by Brian Carrier — Comprehensive TSK reference
- *The Sleuth Kit and Autopsy* by Brian Carrier — Practical TSK guide
- *Linux Filesystem Forensics* by Kevin Karen — Deep dive into ext filesystem forensics

### Related TSK Tools

| Tool | Purpose | Complements fsstat |
|------|---------|-------------------|
| `fls` | List files | Lists files after fsstat identifies the FS |
| `ffind` | Find file by inode | Resolves paths after fsstat provides context |
| `icat` | Extract file | Recovers content after fsstat analysis |
| `ils` | List inodes | Provides inode details after fsstat overview |
| `mmls` | Partition table | Identifies partitions before fsstat analysis |
| `dbstat` | Database statistics | Provides detailed block-level info |

### Online Communities

- **Sleuth Kit Forums**: https://digitalforensicsforum.com/
- **DFIR Reddit**: https://reddit.com/r/DigitalForensics
- **SANS DFIR**: https://www.sans.org/dfir/

---

**TSK Install**: `sudo apt install sleuthkit` | **Man Page**: `man fsstat`
