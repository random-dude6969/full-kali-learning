---
tool_name: tsk_recover
display_name: "TSK tsk_recover"
mitre_tactic: "analysis"
mitre_tactic_id: "TA0007"
install_command: "sudo apt install sleuthkit"
github_repo: "https://github.com/sleuthkit/sleuthkit"
kali_tools_page: "https://www.kali.org/tools/sleuthkit/"
difficulty_level: "intermediate"
requires_root: true
gui_available: false
tags: ["forensics", "sleuthkit", "disk-image", "file-recovery", "deleted-files", "undelete", "carving"]
related_tools: ["sigfind", "sorter", "fls", "icat", "foremost", "scalpel"]
---

# tsk_recover — Deleted File Recovery from Filesystem Images

## Chapter 1: Introduction & Overview

### What Is tsk_recover?

tsk_recover is a command-line forensic utility that is part of The Sleuth Kit (TSK), an open-source digital forensic toolkit developed by Brian Carrier and contributors. tsk_recover recovers deleted files from a filesystem image by copying their data to an output directory. Unlike file carving tools (such as foremost or scalpel) that work at the raw byte level, tsk_recover leverages filesystem metadata — specifically, it reads the inode table and directory entries to identify files that have been marked as deleted but whose data blocks still contain readable content.

### Why tsk_recover Matters in Digital Forensics

When a file is "deleted" on most filesystems, the data is not immediately overwritten. Instead, the filesystem marks the file's inode as free and may remove the directory entry, but the actual data blocks remain intact until they are reused by new files. tsk_recover exploits this behavior by:

1. **Scanning the inode table** for inodes that are marked as unallocated (deleted).
2. **Reading the data blocks** pointed to by those inodes.
3. **Writing the recovered data** to a specified output directory with auto-generated filenames.

This makes tsk_recover invaluable for recovering files that were accidentally deleted, intentionally removed to hide evidence, or lost due to filesystem corruption.

### tsk_recover vs. Other Recovery Tools

| Feature | tsk_recover | foremost | scalpel | PhotoRec |
|---------|-------------|----------|---------|----------|
| Uses filesystem metadata | Yes | No | No | No |
| Recovers by inode | Yes | No | No | No |
| Preserves original filenames | Partially | No | No | No |
| Works on raw images | Yes | Yes | Yes | Yes |
| Works on deleted files only | Yes (by default) | No (all) | No (all) | No (all) |
| Supports NTFS | Yes | Limited | Limited | Yes |
| Supports ext2/3/4 | Yes | Limited | Limited | Yes |
| Supports FAT | Yes | Yes | Yes | Yes |
| Supports HFS+ | Yes | No | No | Yes |
| Requires root | Yes | No | No | No |

### Key Distinction: tsk_recover vs. Carving Tools

Carving tools like foremost and scalpel scan the raw image byte-by-byte for file headers and footers, reconstructing files based on content patterns alone. This approach works even when all filesystem metadata is destroyed. However, carving produces many false positives, cannot determine original filenames, and may fail on fragmented files.

tsk_recover takes a different approach: it reads the filesystem's own metadata structures to find deleted files. This means:

- It can recover files even if their content doesn't have a well-known header/footer pattern.
- It can preserve the original directory structure (when possible).
- It has fewer false positives because it relies on the filesystem's own bookkeeping.
- It fails when the inode table or metadata structures are overwritten.

The best practice is to use both approaches: tsk_recover for metadata-based recovery and foremost/scalpel for raw carving.

### Supported Filesystems

tsk_recover supports all filesystems that The Sleuth Kit can read:

| Filesystem | Support Level | Notes |
|------------|---------------|-------|
| FAT12 | Full | floppy disks, SD cards |
| FAT16 | Full | older USB drives |
| FAT32 | Full | USB drives, memory cards |
| NTFS | Full | Windows systems |
| ext2 | Full | older Linux systems |
| ext3 | Full | Linux with journaling |
| ext4 | Full | modern Linux systems |
| HFS+ | Full | older macOS systems |
| UFS | Full | BSD systems |
| YAFFS2 | Full | Android NAND flash |
| ISO9660 | Full | CD/DVD images |
| ReiserFS | Partial | some features unsupported |

### How tsk_recover Works Internally

1. Opens the filesystem image using libtsk.
2. Walks the inode table, examining each inode.
3. Identifies inodes that are marked as unallocated (deleted).
4. For each deleted inode, reads the data blocks it points to.
5. Writes the data to the output directory.
6. Assigns auto-generated filenames based on the inode number.

The output files are named using the pattern `img-<inode>-<seq>.<ext>`, where `<inode>` is the original inode number and `<seq>` is a sequence number for files that share the same inode (common with NTFS alternate data streams).

---

## Chapter 2: Installation & Setup

### Installing tsk_recover

tsk_recover is part of The Sleuth Kit and installs alongside all other TSK tools:

```bash
sudo apt update
sudo apt install sleuthkit
```

Verify installation:

```bash
which tsk_recover
tsk_recover --help
```

### Building from Source

For the latest development features:

```bash
git clone https://github.com/sleuthkit/sleuthkit.git
cd sleuthkit
./bootstrap
./configure --prefix=/usr/local
make
sudo make install
sudo ldconfig
```

Build dependencies include `autoconf`, `automake`, `libtool`, `g++`, `zlib1g-dev`, and `libsqlite3-dev`.

### Verifying the Installation

```bash
# Check version
tsk_recover -V

# Quick test: recover from a test image
# Create a minimal test filesystem
dd if=/dev/zero of=/tmp/test.img bs=1M count=10
mkfs.ext4 /tmp/test.img
mkdir /tmp/testmnt
sudo mount /tmp/test.img /tmp/testmnt
echo "test file content" | sudo tee /tmp/testmnt/testfile.txt
sudo umount /tmp/testmnt

# Create an image, delete the file, then recover
mkdir /tmp/recovered
sudo tsk_recover -e raw /tmp/test.img /tmp/recovered/
ls /tmp/recovered/
```

### Required Privileges

tsk_recover requires root access to:

- Read raw disk images that may have restricted permissions.
- Access block device files directly.
- Create output directories with appropriate permissions.

Always run with `sudo`:

```bash
sudo tsk_recover [options] <image> <output_dir>
```

### Preparing the Output Directory

tsk_recover creates the output directory if it doesn't exist, but it's good practice to create a clean directory:

```bash
mkdir -p /evidence/recovered
```

**Important**: Always recover to a DIFFERENT drive than the source image. Writing recovered files to the same physical drive can overwrite deleted data you're trying to recover.

---

## Chapter 3: Basic Usage

### Command Syntax

```
sudo tsk_recover [options] <image> <output_dir>
```

Where `<image>` is the path to the filesystem image and `<output_dir>` is the directory where recovered files will be written.

### Complete Flag Reference

| Flag | Long Form | Description | Example |
|------|-----------|-------------|---------|
| `-e <type>` | `--evidence <type>` | Set the evidence type. Use `raw` for raw/dd images (default) or `raw_split` for split raw images. | `sudo tsk_recover -e raw image.raw /recovered/` |
| `-o <num>` | `--offset <num>` | Byte offset to the start of the filesystem within the image. Useful when the image contains a partition table and the filesystem doesn't start at offset 0. | `sudo tsk_recover -o 1048576 image.raw /recovered/` |
| `-t <type>` | `--fstype <type>` | Force the filesystem type. Useful when auto-detection fails. Supported values: `ntfs`, `fat`, `ext2`, `ext3`, `ext4`, `hfs`, `ufs`, `raw` (for raw scanning). | `sudo tsk_recover -t ext4 image.raw /recovered/` |
| `-V` | `--version` | Display version information and exit. | `tsk_recover -V` |
| `-h` | `--help` | Display usage information and exit. | `tsk_recover -h` |

### Basic Example: Recover Deleted Files

```bash
# Recover all deleted files from a filesystem image
sudo tsk_recover -e raw /evidence/suspect.raw /evidence/recovered/
```

### Recovering from a Specific Partition

```bash
# First, identify partition layout
mmls /evidence/disk.raw

# If the filesystem starts at sector 2048 (offset 1048576 bytes):
sudo tsk_recover -o 1048576 /evidence/disk.raw /evidence/recovered/
```

### Forcing Filesystem Type

```bash
# If auto-detection fails, force NTFS
sudo tsk_recover -t ntfs /evidence/image.raw /evidence/recovered/

# Force ext4
sudo tsk_recover -t ext4 /evidence/image.raw /evidence/recovered/
```

### Recovering from a Split Image

```bash
# If the image is split into segments (e.g., E01 converted to raw split)
sudo tsk_recover -e raw_split /evidence/image.raw.000 /evidence/recovered/
```

### Listing Recoverable Files First

Before running a full recovery, you can preview what tsk_recover would recover using `fls` with the `-d` flag (which shows deleted entries):

```bash
# Preview deleted files before recovery
fls -d -r /evidence/suspect.raw

# Then recover
sudo tsk_recover -e raw /evidence/suspect.raw /evidence/recovered/
```

---

## Chapter 4: Intermediate Usage

### Bash Scripting with tsk_recover

#### Complete Recovery Pipeline

```bash
#!/bin/bash
# recover_pipeline.sh — Full file recovery pipeline
# Usage: ./recover_pipeline.sh <raw_image> <output_base>

IMAGE="$1"
OUTPUT_BASE="$2"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
CASE_DIR="${OUTPUT_BASE}/recovery_${TIMESTAMP}"

if [ -z "$IMAGE" ] || [ ! -f "$IMAGE" ]; then
    echo "Usage: $0 <raw_image> <output_base>"
    exit 1
fi

echo "=== File Recovery Pipeline ==="
echo "Image: $IMAGE"
echo "Output: $CASE_DIR"
echo "Started: $(date)"
echo ""

# Create case directory structure
mkdir -p "$CASE_DIR"/{recovered,reports,hashes}

# Step 1: Image integrity verification
echo "[1/6] Verifying image integrity..."
md5sum "$IMAGE" > "$CASE_DIR/hashes/image_md5.txt"
sha256sum "$IMAGE" > "$CASE_DIR/hashes/image_sha256.txt"
IMAGE_SIZE=$(stat -c%s "$IMAGE")
echo "Image size: $IMAGE_SIZE bytes"

# Step 2: Identify partitions
echo "[2/6] Identifying partitions..."
mmls "$IMAGE" > "$CASE_DIR/reports/partition_table.txt" 2>/dev/null || \
    echo "No partition table found (filesystem image?)" > "$CASE_DIR/reports/partition_table.txt"

# Step 3: Preview deleted files
echo "[3/6] Previewing deleted files..."
fls -d -r "$IMAGE" > "$CASE_DIR/reports/deleted_files_preview.txt" 2>/dev/null
DELETED_COUNT=$(wc -l < "$CASE_DIR/reports/deleted_files_preview.txt")
echo "Deleted entries found: $DELETED_COUNT"

# Step 4: Detect filesystem type
echo "[4/6] Detecting filesystem type..."
FS_TYPE=$(mmstat "$IMAGE" 2>/dev/null | awk '{print $1}')
echo "Detected filesystem: $FS_TYPE"
echo "Filesystem type: $FS_TYPE" >> "$CASE_DIR/reports/filesystem_info.txt"

# Step 5: Recover deleted files
echo "[5/6] Recovering deleted files..."
sudo tsk_recover -e raw "$IMAGE" "$CASE_DIR/recovered/" 2>"$CASE_DIR/reports/tsk_recover_errors.txt"
RECOVERED_COUNT=$(find "$CASE_DIR/recovered/" -type f 2>/dev/null | wc -l)
echo "Files recovered: $RECOVERED_COUNT"

# Step 6: Hash all recovered files
echo "[6/6] Hashing recovered files..."
find "$CASE_DIR/recovered/" -type f -exec sha256sum {} \; \
    > "$CASE_DIR/hashes/recovered_files_sha256.txt"

# Generate summary
echo ""
echo "=== Recovery Summary ==="
echo "Image: $IMAGE"
echo "Image hash: $(cat "$CASE_DIR/hashes/image_md5.txt")"
echo "Filesystem: $FS_TYPE"
echo "Deleted entries previewed: $DELETED_COUNT"
echo "Files recovered: $RECOVERED_COUNT"
echo "Output directory: $CASE_DIR/recovered/"
echo "Completed: $(date)"
```

#### Multi-Partition Recovery

```bash
#!/bin/bash
# multi_part_recover.sh — Recover files from all partitions in a disk image
# Usage: ./multi_part_recover.sh <disk_image> <output_dir>

IMAGE="$1"
OUTPUT_DIR="$2"

if [ -z "$IMAGE" ] || [ ! -f "$IMAGE" ]; then
    echo "Usage: $0 <disk_image> <output_dir>"
    exit 1
fi

echo "=== Multi-Partition Recovery ==="
echo "Image: $IMAGE"
echo ""

# Parse partition table
mmls "$IMAGE" 2>/dev/null | grep "^0x" | while read -r line; do
    # Extract partition info
    PART_NUM=$(echo "$line" | awk '{print $1}')
    SECTOR_START=$(echo "$line" | awk '{print $3}')
    SECTOR_SIZE=$(echo "$line" | awk '{print $4}')
    PART_TYPE=$(echo "$line" | awk '{print $7}')
    
    BYTE_OFFSET=$((SECTOR_START * 512))
    
    echo "--- Partition $PART_NUM ---"
    echo "  Type: $PART_TYPE"
    echo "  Offset: $BYTE_OFFSET bytes (sector $SECTOR_START)"
    echo "  Size: $((SECTOR_SIZE * 512)) bytes"
    
    PART_OUT="$OUTPUT_DIR/partition_${PART_NUM}"
    mkdir -p "$PART_OUT"
    
    # Recover from this partition
    sudo tsk_recover -e raw -o "$BYTE_OFFSET" "$IMAGE" "$PART_OUT/recovered/" 2>/dev/null
    
    RECOVERED=$(find "$PART_OUT/recovered/" -type f 2>/dev/null | wc -l)
    echo "  Recovered files: $RECOVERED"
    echo ""
done

echo "=== Recovery Complete ==="
```

#### Recovery with Carving Comparison

```bash
#!/bin/bash
# compare_recovery.sh — Compare tsk_recover with foremost carving results
# Shows files recovered by each method for completeness analysis

IMAGE="$1"
OUTPUT_DIR="$3"
mkdir -p "$OUTPUT_DIR"/{tsk_recover,foremost}

echo "=== Recovery Method Comparison ==="

# Method 1: tsk_recover (metadata-based)
echo "[1/2] Running tsk_recover..."
sudo tsk_recover -e raw "$IMAGE" "$OUTPUT_DIR/tsk_recover/"
TSK_COUNT=$(find "$OUTPUT_DIR/tsk_recover/" -type f | wc -l)
echo "  tsk_recover recovered: $TSK_COUNT files"

# Method 2: foremost (carving-based)
echo "[2/2] Running foremost..."
foremost -i "$IMAGE" -o "$OUTPUT_DIR/foremost/" -q 2>/dev/null
FM_COUNT=$(find "$OUTPUT_DIR/foremost/" -type f -not -name "audit.txt" | wc -l)
echo "  foremost recovered: $FM_COUNT files"

echo ""
echo "=== Comparison Summary ==="
echo "  tsk_recover (metadata-based): $TSK_COUNT files"
echo "  foremost (carving-based):    $FM_COUNT files"

# Find unique files from each method (by hash)
echo ""
echo "Computing hashes for comparison..."
find "$OUTPUT_DIR/tsk_recover/" -type f -exec sha256sum {} \; \
    | awk '{print $1}' | sort -u > "$OUTPUT_DIR/tsk_hashes.txt"
find "$OUTPUT_DIR/foremost/" -type f -exec sha256sum {} \; \
    | awk '{print $1}' | sort -u > "$OUTPUT_DIR/fm_hashes.txt"

TSK_UNIQUE=$(comm -23 "$OUTPUT_DIR/tsk_hashes.txt" "$OUTPUT_DIR/fm_hashes.txt" | wc -l)
FM_UNIQUE=$(comm -13 "$OUTPUT_DIR/tsk_hashes.txt" "$OUTPUT_DIR/fm_hashes.txt" | wc -l)
BOTH=$(comm -12 "$OUTPUT_DIR/tsk_hashes.txt" "$OUTPUT_DIR/fm_hashes.txt" | wc -l)

echo "  Unique to tsk_recover: $TSK_UNIQUE"
echo "  Unique to foremost:    $FM_UNIQUE"
echo "  Found by both:         $BOTH"
```

### Python Scripting with tsk_recover

#### Python Wrapper for tsk_recover

```python
#!/usr/bin/env python3
"""
tsk_recover_wrapper.py — Python wrapper for tsk_recover.
Runs recovery, generates reports, and analyzes recovered files.
"""

import subprocess
import sys
import os
import json
import hashlib
from datetime import datetime
from pathlib import Path
from collections import defaultdict


def run_tsk_recover(image_path: str, output_dir: str,
                    offset: int = 0, fstype: str = None,
                    evidence_type: str = "raw") -> dict:
    """
    Run tsk_recover and return structured results.
    
    Args:
        image_path: Path to the filesystem image.
        output_dir: Directory for recovered files.
        offset: Byte offset to the filesystem (0 for single-partition images).
        fstype: Force filesystem type (None for auto-detect).
        evidence_type: 'raw' for dd images, 'raw_split' for split images.
    
    Returns:
        Dictionary with recovery results and metadata.
    """
    cmd = ["sudo", "tsk_recover"]
    
    # Evidence type
    cmd.extend(["-e", evidence_type])
    
    # Offset
    if offset > 0:
        cmd.extend(["-o", str(offset)])
    
    # Filesystem type override
    if fstype:
        cmd.extend(["-t", fstype])
    
    # Image path and output directory
    cmd.append(image_path)
    cmd.append(output_dir)
    
    print(f"Running: {' '.join(cmd)}")
    
    result = {
        "image": image_path,
        "output_dir": output_dir,
        "offset": offset,
        "fstype_override": fstype,
        "evidence_type": evidence_type,
        "start_time": datetime.now().isoformat(),
        "recovered_files": [],
        "recovered_count": 0,
        "total_size": 0,
        "file_types": defaultdict(int),
        "errors": []
    }
    
    os.makedirs(output_dir, exist_ok=True)
    
    try:
        proc = subprocess.run(
            cmd, capture_output=True, text=True, timeout=3600
        )
        
        if proc.returncode != 0 and proc.stderr:
            result["errors"].append(proc.stderr.strip())
        
        # Scan the output directory for recovered files
        for root, dirs, files in os.walk(output_dir):
            for f in files:
                filepath = os.path.join(root, f)
                rel_path = os.path.relpath(filepath, output_dir)
                
                file_size = os.path.getsize(filepath)
                file_hash = compute_hash(filepath)
                
                # Determine file type by extension or magic bytes
                file_type = detect_file_type(filepath)
                
                file_info = {
                    "filename": f,
                    "relative_path": rel_path,
                    "size": file_size,
                    "md5": file_hash,
                    "file_type": file_type,
                    "inode_from_name": extract_inode_from_name(f)
                }
                
                result["recovered_files"].append(file_info)
                result["recovered_count"] += 1
                result["total_size"] += file_size
                result["file_types"][file_type] += 1
    
    except subprocess.TimeoutExpired:
        result["errors"].append("tsk_recover timed out after 1 hour")
    except FileNotFoundError:
        result["errors"].append(
            "tsk_recover not found — is sleuthkit installed?"
        )
    
    result["end_time"] = datetime.now().isoformat()
    return result


def compute_hash(filepath: str, algorithm: str = "md5") -> str:
    """Compute the hash of a file."""
    h = hashlib.new(algorithm)
    try:
        with open(filepath, "rb") as f:
            while True:
                chunk = f.read(8192)
                if not chunk:
                    break
                h.update(chunk)
        return h.hexdigest()
    except OSError:
        return None


def detect_file_type(filepath: str) -> str:
    """Detect file type using the file command."""
    try:
        result = subprocess.run(
            ["file", "-b", "--mime-type", filepath],
            capture_output=True, text=True
        )
        return result.stdout.strip()
    except (FileNotFoundError, subprocess.SubprocessError):
        # Fallback to extension-based detection
        ext = Path(filepath).suffix.lower()
        return EXTENSION_MAP.get(ext, "unknown")


def extract_inode_from_name(filename: str) -> int:
    """Extract inode number from tsk_recover filename format.
    Format: img-<inode>-<seq>.<ext>"""
    if filename.startswith("img-"):
        parts = filename.split("-")
        if len(parts) >= 2:
            try:
                return int(parts[1])
            except ValueError:
                pass
    return None


# Extension to MIME type fallback map
EXTENSION_MAP = {
    ".txt": "text/plain",
    ".jpg": "image/jpeg",
    ".jpeg": "image/jpeg",
    ".png": "image/png",
    ".gif": "image/gif",
    ".pdf": "application/pdf",
    ".doc": "application/msword",
    ".docx": "application/vnd.openxmlformats-officedocument.wordprocessingml.document",
    ".xls": "application/vnd.ms-excel",
    ".xlsx": "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet",
    ".zip": "application/zip",
    ".rar": "application/x-rar-compressed",
    ".exe": "application/x-dosexec",
    ".dll": "application/x-dosexec",
    ".html": "text/html",
    ".htm": "text/html",
    ".mp4": "video/mp4",
    ".mp3": "audio/mpeg",
}


def print_recovery_report(result: dict):
    """Print a formatted recovery report."""
    print("\n" + "=" * 60)
    print("TSK_RECOVER RECOVERY REPORT")
    print("=" * 60)
    print(f"Image:          {result['image']}")
    print(f"Output:         {result['output_dir']}")
    print(f"Offset:         {result['offset']}")
    print(f"Evidence type:  {result['evidence_type']}")
    print(f"FSType:         {result['fstype_override'] or 'auto-detect'}")
    print(f"Start:          {result['start_time']}")
    print(f"End:            {result['end_time']}")
    print(f"Files recovered: {result['recovered_count']:,}")
    print(f"Total size:     {result['total_size']:,} bytes")
    
    if result["errors"]:
        print("\nERRORS:")
        for err in result["errors"]:
            print(f"  - {err}")
    
    print("\nFILE TYPES:")
    print("-" * 60)
    sorted_types = sorted(
        result["file_types"].items(),
        key=lambda x: x[1], reverse=True
    )
    for ftype, count in sorted_types:
        print(f"  {ftype:40s}  {count:>6d}")
    
    # Show inode distribution
    inodes = [f["inode_from_name"] for f in result["recovered_files"]
              if f["inode_from_name"] is not None]
    if inodes:
        print(f"\nINODE RANGE: {min(inodes)} - {max(inodes)}")
    
    print("=" * 60)


def main():
    if len(sys.argv) < 2:
        print(f"Usage: {sys.argv[0]} <image> [output_dir] [offset]")
        print("Wraps tsk_recover with structured reporting.")
        sys.exit(1)
    
    image_path = sys.argv[1]
    output_dir = sys.argv[2] if len(sys.argv) > 2 else \
        f"{Path(image_path).stem}_recovered"
    offset = int(sys.argv[3]) if len(sys.argv) > 3 else 0
    
    if not os.path.isfile(image_path):
        print(f"Error: Image file not found: {image_path}")
        sys.exit(1)
    
    result = run_tsk_recover(image_path, output_dir, offset=offset)
    print_recovery_report(result)
    
    # Save JSON report
    report_path = os.path.join(output_dir, "recovery_report.json")
    with open(report_path, "w") as f:
        json.dump(result, f, indent=2, default=str)
    print(f"\nJSON report: {report_path}")


if __name__ == "__main__":
    main()
```

#### Inode Analysis Script

```python
#!/usr/bin/env python3
"""
inode_analysis.py — Analyze inode usage in a filesystem image.
Shows allocated vs. unallocated inodes and estimates recovery potential.
Uses fls output for analysis.
"""

import subprocess
import re
import sys
from collections import defaultdict
from pathlib import Path


def run_fls(image_path: str, offset: int = 0) -> dict:
    """
    Run fls to enumerate all inodes and their allocation status.
    Returns a dict with 'allocated' and 'deleted' inode lists.
    """
    cmd = ["fls", "-r", "-d"]
    if offset > 0:
        cmd.extend(["-o", str(offset // 512)])
    cmd.append(image_path)
    
    result = subprocess.run(cmd, capture_output=True, text=True)
    
    allocated = []
    deleted = []
    
    for line in result.stdout.splitlines():
        # Parse fls output format: "r/r * inode: name"
        # * indicates deleted entry
        is_deleted = "*" in line
        
        # Extract inode number
        inode_match = re.search(r'(\d+):', line)
        if inode_match:
            inode = int(inode_match.group(1))
            if is_deleted:
                deleted.append(inode)
            else:
                allocated.append(inode)
    
    return {
        "allocated": sorted(set(allocated)),
        "deleted": sorted(set(deleted)),
        "total_unique_inodes": len(set(allocated + deleted))
    }


def analyze_inode_gaps(inodes: list) -> list:
    """Find gaps in the inode sequence that may indicate freed inodes."""
    if not inodes:
        return []
    
    gaps = []
    for i in range(1, len(inodes)):
        gap_start = inodes[i - 1] + 1
        gap_end = inodes[i] - 1
        if gap_start <= gap_end:
            gaps.append((gap_start, gap_end, gap_end - gap_start + 1))
    
    return gaps


def print_inode_report(fls_data: dict):
    """Print a formatted inode analysis report."""
    allocated = fls_data["allocated"]
    deleted = fls_data["deleted"]
    
    print("\n" + "=" * 60)
    print("INODE ANALYSIS REPORT")
    print("=" * 60)
    print(f"Allocated inodes:  {len(allocated):,}")
    print(f"Deleted inodes:    {len(deleted):,}")
    print(f"Total unique:      {fls_data['total_unique_inodes']:,}")
    
    if allocated:
        print(f"Allocated range:   {min(allocated):,} - {max(allocated):,}")
    if deleted:
        print(f"Deleted range:     {min(deleted):,} - {max(deleted):,}")
        print(f"First 20 deleted:  {deleted[:20]}")
    
    # Recovery potential
    total = len(allocated) + len(deleted)
    if total > 0:
        pct_deleted = (len(deleted) / total) * 100
        print(f"\nRecovery potential: {pct_deleted:.1f}% of inodes are deleted")
    
    # Gap analysis
    gaps = analyze_inode_gaps(allocated + deleted)
    if gaps:
        print(f"\nInode gaps (freed/unused): {len(gaps)}")
        for start, end, size in gaps[:10]:
            print(f"  Inode {start:,} - {end:,} ({size:,} inodes)")
    
    print("=" * 60)


if __name__ == "__main__":
    if len(sys.argv) < 2:
        print(f"Usage: {sys.argv[0]} <filesystem_image>")
        sys.exit(1)
    
    image = sys.argv[1]
    print(f"Analyzing inodes in: {image}")
    
    fls_data = run_fls(image)
    print_inode_report(fls_data)
```

#### Recovery Validation Script

```python
#!/usr/bin/env python3
"""
validate_recovery.py — Validate recovered files from tsk_recover.
Checks file integrity, identifies corrupt files, and generates a validation report.
"""

import os
import sys
import json
import hashlib
import subprocess
from pathlib import Path
from collections import defaultdict


def validate_file(filepath: str) -> dict:
    """Validate a single recovered file."""
    info = {
        "path": filepath,
        "filename": os.path.basename(filepath),
        "size": os.path.getsize(filepath),
        "valid": True,
        "issues": []
    }
    
    # Check if file is empty
    if info["size"] == 0:
        info["issues"].append("empty file (0 bytes)")
    
    # Check file type consistency
    try:
        file_result = subprocess.run(
            ["file", "-b", filepath],
            capture_output=True, text=True
        )
        detected_type = file_result.stdout.strip()
        info["file_type"] = detected_type
        
        # Check for common corruption indicators
        if "data" in detected_type.lower() and info["size"] > 0:
            info["issues"].append("unrecognized content (may be corrupt)")
        
        if "ASCII text" in detected_type or "UTF-8" in detected_type:
            # Text file — check for null bytes (corruption indicator)
            with open(filepath, "rb") as f:
                content = f.read(min(8192, info["size"]))
                null_count = content.count(b'\x00')
                if null_count > 0:
                    pct_null = (null_count / len(content)) * 100
                    if pct_null > 5:
                        info["issues"].append(
                            f"high null byte ratio: {pct_null:.1f}%"
                        )
    
    except (FileNotFoundError, subprocess.SubprocessError):
        info["file_type"] = "unknown"
        info["issues"].append("could not determine file type")
    
    # Compute MD5
    try:
        h = hashlib.md5()
        with open(filepath, "rb") as f:
            while True:
                chunk = f.read(8192)
                if not chunk:
                    break
                h.update(chunk)
        info["md5"] = h.hexdigest()
    except OSError as e:
        info["issues"].append(f"hash error: {e}")
        info["valid"] = False
    
    if info["issues"]:
        info["valid"] = False
    
    return info


def validate_recovery_directory(recovery_dir: str) -> dict:
    """Validate all files in a recovery directory."""
    results = {
        "directory": recovery_dir,
        "total_files": 0,
        "valid_files": 0,
        "invalid_files": 0,
        "empty_files": 0,
        "type_distribution": defaultdict(int),
        "files": []
    }
    
    for root, dirs, files in os.walk(recovery_dir):
        for f in files:
            filepath = os.path.join(root, f)
            info = validate_file(filepath)
            results["files"].append(info)
            results["total_files"] += 1
            
            if info["valid"]:
                results["valid_files"] += 1
            else:
                results["invalid_files"] += 1
            
            if info["size"] == 0:
                results["empty_files"] += 1
            
            ftype = info.get("file_type", "unknown")
            results["type_distribution"][ftype] += 1
    
    return results


def print_validation_report(results: dict):
    """Print the validation report."""
    print("\n" + "=" * 60)
    print("RECOVERY VALIDATION REPORT")
    print("=" * 60)
    print(f"Directory:  {results['directory']}")
    print(f"Total:      {results['total_files']}")
    print(f"Valid:      {results['valid_files']}")
    print(f"Invalid:    {results['invalid_files']}")
    print(f"Empty:      {results['empty_files']}")
    
    print("\nFILE TYPE DISTRIBUTION:")
    for ftype, count in sorted(results["type_distribution"].items(),
                                key=lambda x: x[1], reverse=True):
        print(f"  {ftype:45s}  {count:>5d}")
    
    # Show invalid files
    invalid = [f for f in results["files"] if not f["valid"]]
    if invalid:
        print(f"\nINVALID FILES ({len(invalid)}):")
        for item in invalid[:30]:
            print(f"  {item['filename']} ({item['size']} bytes)")
            for issue in item["issues"]:
                print(f"    -> {issue}")
        if len(invalid) > 30:
            print(f"  ... and {len(invalid) - 30} more")
    
    print("=" * 60)


if __name__ == "__main__":
    if len(sys.argv) < 2:
        print(f"Usage: {sys.argv[0]} <recovery_directory>")
        sys.exit(1)
    
    results = validate_recovery_directory(sys.argv[1])
    print_validation_report(results)
    
    # Save report
    report_path = os.path.join(sys.argv[1], "validation_report.json")
    with open(report_path, "w") as f:
        json.dump(results, f, indent=2, default=str)
    print(f"\nJSON report: {report_path}")
```

---

## Chapter 5: Advanced Usage

### Combining tsk_recover with Carving Tools

The most thorough recovery approach uses both metadata-based recovery (tsk_recover) and raw carving (foremost/scalpel):

```bash
#!/bin/bash
# dual_recovery.sh — Use both tsk_recover and foremost for maximum recovery
IMAGE="$1"
OUTPUT="/evidence/dual_recovery"
mkdir -p "$OUTPUT"/{tsk,carved,combined}

# Method 1: Metadata-based recovery
echo "Running tsk_recover..."
sudo tsk_recover -e raw "$IMAGE" "$OUTPUT/tsk/"

# Method 2: Raw carving
echo "Running foremost..."
foremost -i "$IMAGE" -o "$OUTPUT/carved/" -q

# Combine and deduplicate by hash
echo "Deduplicating..."
find "$OUTPUT/tsk/" "$OUTPUT/carved/" -type f -exec sha256sum {} \; \
    | sort -k1 | uniq -w64 > "$OUTPUT/combined/all_hashes.txt"

echo "Combined unique files: $(wc -l < "$OUTPUT/combined/all_hashes.txt")"
```

### Using tsk_recover on Specific Filesystem Types

```bash
# NTFS — recovers alternate data streams, MFT entries
sudo tsk_recover -t ntfs -e raw -o 1048576 /evidence/ntfs.raw /evidence/recovered/

# ext4 — recovers deleted inodes
sudo tsk_recover -t ext4 -e raw -o 2048 /evidence/ext4.raw /evidence/recovered/

# FAT32 — recovers from chain allocation
sudo tsk_recover -t fat -e raw -o 63 /evidence/fat32.raw /evidence/recovered/
```

### Recovery with Offset Calibration

When working with complex partition layouts, precise offset calculation is critical:

```bash
#!/bin/bash
# calibrate_offset.sh — Find the correct filesystem offset

IMAGE="$1"

echo "=== Partition Table ==="
mmls "$IMAGE"

echo ""
echo "=== Attempting recovery at various offsets ==="

# Try common partition offsets
for OFFSET in 0 63 2048 4096 8192 65536 1048576; do
    BYTE_OFFSET=$((OFFSET * 512))
    echo ""
    echo "--- Offset: sector $OFFSET (byte $BYTE_OFFSET) ---"
    
    OUTPUT="/tmp/recovery_test_$OFFSET"
    rm -rf "$OUTPUT"
    mkdir -p "$OUTPUT"
    
    sudo tsk_recover -e raw -o "$BYTE_OFFSET" "$IMAGE" "$OUTPUT/" 2>/dev/null
    COUNT=$(find "$OUTPUT/" -type f 2>/dev/null | wc -l)
    echo "  Files recovered: $COUNT"
done
```

### Automated Evidence Processing with tsk_recover

```bash
#!/bin/bash
# auto_evidence.sh — Process multiple evidence images automatically
# Usage: ./auto_evidence.sh <evidence_dir>

EVIDENCE_DIR="$1"
OUTPUT_BASE="/evidence/processed"
mkdir -p "$OUTPUT_BASE"

for IMAGE in "$EVIDENCE_DIR"/*.raw "$EVIDENCE_DIR"/*.dd "$EVIDENCE_DIR"/*.img; do
    [ -f "$IMAGE" ] || continue
    
    BASENAME=$(basename "$IMAGE" | sed 's/\.[^.]*$//')
    CASE_OUTPUT="$OUTPUT_BASE/$BASENAME"
    mkdir -p "$CASE_OUTPUT"
    
    echo "=== Processing: $IMAGE ==="
    
    # Hash the image
    md5sum "$IMAGE" > "$CASE_OUTPUT/image_hash.txt"
    
    # Identify partitions
    mmls "$IMAGE" > "$CASE_OUTPUT/partitions.txt" 2>/dev/null
    
    # Recover deleted files
    sudo tsk_recover -e raw "$IMAGE" "$CASE_OUTPUT/recovered/" 2>/dev/null
    
    RECOVERED=$(find "$CASE_OUTPUT/recovered/" -type f | wc -l)
    echo "  Recovered: $RECOVERED files"
    echo ""
done
```

### Verifying Recovery Integrity

```bash
#!/bin/bash
# verify_recovery.sh — Verify integrity of recovered files
RECOVERY_DIR="$1"

echo "=== Recovery Verification ==="
echo "Directory: $RECOVERY_DIR"
echo ""

TOTAL=0
EMPTY=0
CORRUPT=0

find "$RECOVERY_DIR" -type f | while read -r f; do
    SIZE=$(stat -c%s "$f")
    TOTAL=$((TOTAL + 1))
    
    # Check for empty files
    if [ "$SIZE" -eq 0 ]; then
        echo "EMPTY: $f"
        EMPTY=$((EMPTY + 1))
        continue
    fi
    
    # Check for null-byte-heavy files (corruption indicator)
    NULLS=$(xxd -l 4096 "$f" | grep -c "0000" 2>/dev/null || echo 0)
    if [ "$NULLS" -gt 50 ]; then
        echo "POSSIBLY CORRUPT: $f (high null byte ratio)"
        CORRUPT=$((CORRUPT + 1))
    fi
done

echo ""
echo "=== Summary ==="
echo "Total files checked: $TOTAL"
echo "Empty files: $EMPTY"
echo "Potentially corrupt: $CORRUPT"
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: Accidental File Deletion Recovery

**Context**: A user accidentally deleted important financial spreadsheets from their Windows laptop. The laptop's hard drive has been imaged to preserve evidence. The NTFS filesystem still contains the deleted file's metadata in the MFT (Master File Table), and the data blocks have not yet been overwritten.

**Approach**: Use tsk_recover with NTFS support to recover the deleted files.

```bash
# Step 1: Create forensic image of the drive
sudo dd if=/dev/sdb of=/evidence/laptop.raw bs=4M status=progress

# Step 2: Identify the NTFS partition
mmls /evidence/laptop.raw
# Output: partition 1 (NTFS) starts at sector 2048

# Step 3: Preview deleted files with fls
fls -d -r -o 2048 /evidence/laptop.raw | grep -i "\.xlsx\|\.xls\|\.csv"

# Step 4: Recover deleted files
sudo tsk_recover -e raw -o 1048576 /evidence/laptop.raw /evidence/recovered/

# Step 5: Find and verify the recovered spreadsheets
find /evidence/recovered/ -type f -name "img-*" | while read f; do
    file "$f"
done | grep -i "excel\|spreadsheet"

# Step 6: Compute hashes for chain of custody
find /evidence/recovered/ -type f -exec sha256sum {} \; \
    > /evidence/recovered_hashes.txt
```

**Outcome**: tsk_recover recovered 47 deleted files from the NTFS MFT. Among them, 12 were Excel spreadsheets matching the user's financial records. The data blocks were intact because the user deleted the files recently and the drive hadn't been heavily used since deletion.

### Scenario 2: Anti-Forensics — Recovering After Disk Wipe Attempt

**Context**: A suspect attempted to destroy evidence by running a quick format on the drive before seizure. The investigation team suspects that a quick format (as opposed to a full format) only overwrites the filesystem metadata but not the data blocks.

**Approach**: Use tsk_recover to attempt recovery, and compare with carving tools to assess what was recoverable.

```bash
# Step 1: Image the drive
sudo dd if=/dev/sdb of=/evidence/formatted.raw bs=4M status=progress

# Step 2: Examine the filesystem after formatting
# A quick-formatted NTFS will show a fresh MFT
mmls /evidence/formatted.raw

# Step 3: Attempt tsk_recover (metadata-based)
sudo tsk_recover -e raw /evidence/formatted.raw /evidence/recovered_meta/
META_COUNT=$(find /evidence/recovered_meta/ -type f | wc -l)
echo "Metadata-based recovery: $META_COUNT files"

# Step 4: Attempt carving (raw byte scanning)
foremost -i /evidence/formatted.raw -o /evidence/recovered_carved/ -q
CARVED_COUNT=$(find /evidence/recovered_carved/ -type f -not -name audit.txt | wc -l)
echo "Carving-based recovery: $CARVED_COUNT files"

# Step 5: Compare results
echo ""
echo "Comparison:"
echo "  Metadata-based: $META_COUNT files (requires intact inode table)"
echo "  Raw carving:    $CARVED_COUNT files (works even if metadata destroyed)"

# Step 6: Hash all recovered files for deduplication
find /evidence/recovered_meta/ /evidence/recovered_carved/ \
    -type f -exec sha256sum {} \; | sort -k1 | uniq -w64 \
    > /evidence/all_recovered_hashes.txt
UNIQUE=$(wc -l < /evidence/all_recovered_hashes.txt)
echo "Unique recovered files: $UNIQUE"
```

### Scenario 3: Mobile Device Forensics — NAND Flash Recovery

**Context**: A mobile device was recovered from a suspect. The device's NAND flash chip was desoldered and imaged. The filesystem is YAFFS2 (common on older Android devices). Several apps and user data files have been deleted.

**Approach**: Use tsk_recover with YAFFS2 filesystem support.

```bash
# Step 1: Image the NAND chip (using a chip reader)
sudo dd if=/dev/mtdblock0 of=/evidence/nand.raw bs=4096

# Step 2: Identify the filesystem
tsk_recover -V  # Check YAFFS2 support

# Step 3: Examine with fls
fls -r -d /evidence/nand.raw

# Step 4: Recover deleted files
sudo tsk_recover -e raw /evidence/nand.raw /evidence/recovered/

# Step 5: Categorize recovered files
echo "=== Recovered File Types ==="
find /evidence/recovered/ -type f -exec file {} \; | \
    awk -F: '{print $2}' | sort | uniq -c | sort -rn

# Step 6: Focus on messaging and contacts databases
find /evidence/recovered/ -type f \( -name "*.db" -o -name "*.sqlite" \
    -o -name "*.txt" -o -name "*.vcard" \)
```

---

## Chapter 7: Detection & Defense

### How Defenders Can Use tsk_recover

- **Data recovery**: Recover accidentally deleted files before they're overwritten.
- **Incident response**: Recover files deleted by attackers to cover their tracks.
- **Compliance**: Ensure deleted sensitive data is actually unrecoverable before disposing of drives.
- **Insider threat**: Recover files that employees attempted to destroy.

### Detecting tsk_recover Usage

- tsk_recover reads the image but doesn't modify it, so its use is undetectable from the image.
- Command-line history records tsk_recover invocations.
- The output directory contains recovered files with auto-generated names.

### Defensive Measures Against Recovery

To prevent file recovery by tsk_recover or similar tools:

1. **Use full-disk encryption**: Encrypt the entire drive so deleted file data is also encrypted.
2. **Secure deletion**: Use tools like `shred` or `sdelete` that overwrite data blocks, not just metadata.
3. **SSD TRIM**: On SSDs, the TRIM command immediately frees data blocks, making recovery much harder.
4. **Physical destruction**: For the highest security requirements, physically destroy the storage medium.

### Limitations of tsk_recover

tsk_recover has inherent limitations:

- It cannot recover files whose data blocks have been overwritten.
- It cannot recover files from encrypted filesystems without the decryption key.
- It may not recover fragmented files correctly (the metadata for fragment locations may be destroyed).
- Filesystem-specific limitations apply (e.g., NTFS alternate data streams may not be fully recovered).

---

## Chapter 8: Troubleshooting (FAQ)

### Q1: tsk_recover recovers 0 files

Common causes:

1. **Wrong offset**: The filesystem doesn't start at offset 0. Use `mmls` to find the correct partition offset and use the `-o` flag.
2. **Filesystem type mismatch**: Auto-detection fails. Use `-t` to force the filesystem type.
3. **Fully overwritten data**: If the data blocks have been overwritten, there's nothing to recover from metadata.
4. **Encrypted filesystem**: tsk_recover cannot read encrypted data without the key.

```bash
# Debug: check what the filesystem looks like
fls /evidence/image.raw  # If this shows no files, the filesystem may be wrong
mmstat /evidence/image.raw  # Check detected filesystem type
```

### Q2: Recovered files are corrupt or partially readable

This is normal for deleted files. Possible causes:

- **Partial overwrite**: Some data blocks were reused before the image was taken.
- **Fragmented files**: File fragments may be scattered and incomplete.
- **Metadata corruption**: The inode or MFT entry may be partially overwritten.

Try combining tsk_recover with carving tools (foremost/scalpel) for a more complete recovery.

### Q3: tsk_recover says "Cannot determine filesystem type"

```bash
# Try forcing the filesystem type
sudo tsk_recover -t ntfs /evidence/image.raw /output/
sudo tsk_recover -t ext4 /evidence/image.raw /output/
sudo tsk_recover -t fat /evidence/image.raw /output/

# Check what the filesystem looks like
mmstat /evidence/image.raw
fsstat /evidence/image.raw
```

### Q4: How do I recover only specific file types?

tsk_recover doesn't have a built-in file-type filter. Recover everything, then filter the output:

```bash
# Recover all deleted files
sudo tsk_recover -e raw /evidence/image.raw /evidence/all_recovered/

# Keep only images
find /evidence/all_recovered/ -type f \( -name "*.jpg" -o -name "*.png" \
    -o -name "*.gif" \) -exec mv {} /evidence/images/ \;

# Or use file command to filter by content type
find /evidence/all_recovered/ -type f -exec sh -c \
    'file "$1" | grep -q "JPEG" && mv "$1" /evidence/jpegs/' _ {} \;
```

### Q5: Recovered files have strange names like "img-12345-0.jpg"

This is expected. tsk_recover uses the inode number to generate filenames since the original filenames may have been deleted. The format is `img-<inode>-<sequence>.<extension>`.

To map recovered files back to their original names, use `fls` to list the deleted directory entries:

```bash
# Show deleted entries with their original names and inode numbers
fls -d -r /evidence/image.raw | grep "12345"
# Output might show: "r/r * 12345: important_document.xlsx"
# This means inode 12345 was originally "important_document.xlsx"
```

### Q6: Can tsk_recover recover files from a RAID array?

tsk_recover works on individual filesystem images, not RAID arrays. First, reconstruct the RAID:

```bash
# Example: reconstruct a RAID-5 array with three disks
mdadm --assemble /dev/md0 /dev/sdb1 /dev/sdc1 /dev/sdd1

# Then image the assembled device
dd if=/dev/md0 of=/evidence/raid.raw bs=4M

# Now recover
sudo tsk_recover -e raw /evidence/raid.raw /evidence/recovered/
```

### Q7: What is the difference between `-e raw` and no `-e` flag?

The `-e` flag specifies the evidence type. When omitted, tsk_recover defaults to `raw` mode. The alternative is `raw_split` for split image files (e.g., `.000`, `.001`, etc.). Always use `-e raw` explicitly for clarity:

```bash
sudo tsk_recover -e raw /evidence/image.raw /output/
# Equivalent to:
sudo tsk_recover /evidence/image.raw /output/
```

---

## Resources

### Official Documentation

- The Sleuth Kit Manual: https://sleuthkit.org/sleuthkit/man/
- TSK GitHub Repository: https://github.com/sleuthkit/sleuthkit
- Brian Carrier's File System Forensic Analysis (book, 2005)

### Related TSK Tools

| Tool | Purpose |
|------|---------|
| fls | List files and directories (including deleted entries with `-d`) |
| icat | Extract a specific file by inode number |
| fsstat | Display filesystem statistics and metadata |
| sigfind | Find hex signatures at specific offsets |
| sorter | Sort and categorize files by type and hash |
| mmls | Display partition table layout |

### Complementary Recovery Tools

| Tool | Purpose |
|------|---------|
| foremost | File carving based on headers/footers |
| scalpel | Advanced file carving with configurable rules |
| PhotoRec | Photo and multimedia file recovery |
| testdisk | Partition recovery and boot sector repair |
| ddrescue | Damaged drive imaging with error recovery |

### Books and References

- "File System Forensic Analysis" by Brian Carrier (Addison-Wesley, 2005)
- "The Art of Memory Forensics" by Ligh et al. (Wiley, 2014)
- "Hands-On Data Recovery and Digital Forensics" by Various Authors
- NIST SP 800-86: Guide to Integrating Forensic Techniques into Incident Response
- SANS FOR508: Advanced Incident Response and Threat Hunting

### Practice Resources

- Digital Forensics Challenge (DFC) images
- NIST Computer Forensics Tool Testing (CFTT) datasets
- CyberDefenders.org challenge images
- SANS forensic practice images
- CICIDS datasets for integrated forensic exercises

### Filesystem-Specific References

- NTFS Forensics: https://docs.microsoft.com/en-us/windows/win32/fileio/ntfs-technical-reference
- ext4 Technical Overview: https://www.kernel.org/doc/html/latest/filesystems/ext4.html
- FAT Filesystem Specification: https://www.microsoft.com/en-us/download/details.aspx?id=13032
- YAFFS2 Documentation: https://www.yaffs.net/
