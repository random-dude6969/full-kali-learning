---
tool_name: icat
display_name: icat
mitre_tactic: "analysis"
mitre_tactic_id: "TA0007"
install_command: "sudo apt install sleuthkit"
github_repo: "https://github.com/sleuthkit/sleuthkit"
kali_tools_page: "https://www.kali.org/tools/sleuthkit/"
difficulty_level: "beginner"
requires_root: false
gui_available: false
tags: ["forensics", "filesystem", "sleuthkit", "tsk", "inode", "file-extraction", "recovery", "data-carving"]
related_tools: ["fls", "ffind", "fsstat", "ils", "istat", "tsk_recover", "blkls"]
---

# icat — Forensic File Content Extraction by Inode

## Chapter 1: Introduction & Overview

### What icat Does

icat is a command-line utility from The Sleuth Kit (TSK) that extracts the contents of a file from a filesystem image based on its inode number. Think of it as a forensic version of `cat` — but instead of reading a file by its path, it reads directly from the inode, which is the internal numeric identifier that the filesystem uses to track files.

This distinction is critical in forensics because:

1. **Deleted files** lose their directory entries (names) but their inodes may persist until overwritten.
2. **Files can be recovered** if you know their inode number, even if the name is gone.
3. **Bypasses OS restrictions** — icat reads raw disk structures, not the mounted filesystem.
4. **Works on images** — no need to mount the filesystem, which preserves evidence integrity.

icat is the extraction tool in the TSK pipeline. After `fsstat` identifies the filesystem, `fls` lists files and their inodes, and `ffind` resolves inode-to-path mappings, `icat` pulls out the actual file contents.

### Why Inode-Based Extraction Matters

Traditional file access requires the file to have a valid name and path. Forensic investigators often work with:

- **Deleted files**: Names removed, inodes still present
- **Corrupted filesystems**: Directory structures damaged, inodes intact
- **Hidden files**: Names obscured or in unusual locations
- **Alternate data streams** (NTFS): Hidden content behind visible files
- **Slack space**: Data残留 in unused portions of allocated blocks

icat accesses the filesystem at the inode level, bypassing all these restrictions.

### Position in The Sleuth Kit

icat is the primary data extraction tool:

| Layer | Tools | Purpose |
|-------|-------|---------|
| Volume System | `mmls`, `mmstat` | Partition table analysis |
| Filesystem Metadata | `dbstat`, `dls` | Raw block-level analysis |
| Filesystem Analysis | `fls`, `ffind`, `ils` | File and inode operations |
| **File Recovery** | **`icat`**, `tsk_recover` | **Data extraction** |

icat is typically the final tool in the analysis pipeline, used after identifying files of interest with `fls` or `ils`.

### Comparison with Standard Tools

| Feature | `cat` | `icat` |
|---------|-------|--------|
| Requires mounting | Yes | No |
| Works on images | No | Yes |
| Accesses deleted files | No | Yes |
| Bypasses OS permissions | No | Yes |
| Works on raw disk | No | Yes |
| Handles corrupted FS | Limited | Better |

---

## Chapter 2: Installation & Setup

### Installing The Sleuth Kit

icat is included in the Sleuth Kit package:

```bash
# Update package lists
sudo apt update

# Install Sleuth Kit
sudo apt install sleuthkit

# Verify installation
which icat
icat -V
```

### Verifying the Installation

```bash
# Check icat is available
which icat

# Display version
icat -V

# List all TSK tools
dpkg -L sleuthkit | grep bin/

# Quick help check
icat -h
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

# Create output directory for recovered files
mkdir -p recovered_files/

# Document chain of custody
cat > chain_of_custody.txt << EOF
Evidence: /dev/sdb
Image: usb_image.dd
Created: $(date -u)
Hash: $(sha256sum usb_image.dd | cut -d' ' -f1)
Analyst: $(whoami)
Recovery started: $(date -u)
EOF
```

### Filesystem Support Matrix

icat supports these filesystem types:

| Filesystem | Full Support | Deleted Files | Notes |
|------------|:------------:|:-------------:|-------|
| ext2 | Yes | Yes | Primary Linux FS |
| ext3 | Yes | Yes | With journal |
| ext4 | Yes | Yes | Most common Linux FS |
| FAT12 | Yes | Yes | Floppy disks |
| FAT16 | Yes | Yes | Older removable |
| FAT32 | Yes | Yes | USB drives, SD cards |
| NTFS | Yes | Yes | Windows primary FS |
| HFS+ | Yes | Yes | macOS |
| UFS | Yes | Yes | BSD |
| YAFFS2 | Partial | Limited | Android NAND |

---

## Chapter 3: Basic Usage

### Command Syntax

```
icat [options] <image> <inode>
```

**Positional Arguments:**

| Argument | Description |
|----------|-------------|
| `<image>` | Path to filesystem image or device node |
| `<inode>` | The inode number of the file to extract |

### Complete Flag Reference

| Flag | Long Form | Description |
|------|-----------|-------------|
| `-f` | `--fstype` | Specify the filesystem type |
| `-o` | `--offset` | Byte offset to the filesystem |
| `-i` | `--inode` | Display inode information (don't extract) |
| `-r` | `--block` | Extract as raw block (no inode interpretation) |
| `-s` | `--slack` | Extract slack space |
| `-e` | `--embed` | Extract embedded data (NTFS ADS) |
| `-t` | `--text` | Display as text (default) |
| `-b` | `--binary` | Output as binary (default for files) |
| `-c` | `--csv` | Output in CSV format |
| `-h` | `--help` | Display help and exit |
| `-V` | `--version` | Print version and exit |
| `-v` | `--verbose` | Verbose output |
| `-z` | `--zone` | Timezone for timestamps |

### Example 1: Basic File Extraction

```bash
# Extract file content by inode number
icat usb_image.dd 58493 > recovered_document.docx
```

This extracts the contents of inode 58493 and saves it as `recovered_document.docx`.

### Example 2: Display File Contents

```bash
# Display file contents to stdout
icat usb_image.dd 58493
```

If the file is text, its contents will display in the terminal.

### Example 3: Specify Filesystem Type

```bash
# Force filesystem type when auto-detection fails
icat -f ext4 usb_image.dd 58493 > recovered_file.txt
```

### Example 4: With Partition Offset

```bash
# For multi-partition images, specify offset
mmls disk_image.dd  # Find partition layout

# Extract from partition at offset 2048 sectors
icat -o $((2048 * 512)) disk_image.dd 58493 > recovered_file.txt
```

### Example 5: Show Inode Information

```bash
# Display inode info without extracting content
icat -i usb_image.dd 58493
```

**Output:**
```
Inode: 58493
Allocated: Yes
Flags: 0
Size: 4096
uid: 1000
gid: 1000
Mode: 0100644
Links: 1
Block Count: 8
Access Time: Mon Jan 16 14:32:01 2024
Modify Time: Mon Jan 16 15:45:22 2024
Change Time: Mon Jan 16 15:45:22 2024
```

### Example 6: Extract Slack Space

```bash
# Extract slack space (data in unused portion of allocated blocks)
icat -s usb_image.dd 58493 > slack_data.bin
```

### Example 7: NTFS Alternate Data Streams

```bash
# Extract NTFS alternate data stream
icat -e -f ntfs windows_image.dd 67 > hidden_stream.txt
```

### Example 8: Batch Extraction

```bash
# Extract multiple files by inode
while read inode; do
    icat usb_image.dd "$inode" > "recovered_$inode.bin"
done < inodes.txt
```

### Example 9: Pipe to Analysis Tools

```bash
# Extract and immediately analyze
icat usb_image.dd 58493 | file -
icat usb_image.dd 58493 | strings | head -20
icat usb_image.dd 58493 | sha256sum
```

### Example 10: Preserve Directory Structure

```bash
# Extract with path preservation
icat -p usb_image.dd 58493 > "Documents/resume.docx"
```

### Exit Codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | Error or inode not found |
| 2 | Invalid image or filesystem |

---

## Chapter 4: Intermediate Usage

### Bash Scripting with icat

#### Script 1: Recover All Deleted Files

```bash
#!/bin/bash
# recover_all_deleted.sh — Recover all deleted files from filesystem image

IMAGE="$1"
OUTPUT_DIR="recovered_$(date +%Y%m%d_%H%M%S)"

if [ -z "$IMAGE" ]; then
    echo "Usage: $0 <filesystem_image>"
    exit 1
fi

mkdir -p "$OUTPUT_DIR"

echo "=== Recovering Deleted Files ==="
echo "Image: $IMAGE"
echo "Output: $OUTPUT_DIR"
echo ""

# Get list of deleted files with inodes
fls -r -d -m "/" "$IMAGE" | while IFS='|' read -r timestamp name inode flags uid gid size; do
    # Skip header and empty lines
    [[ "$inode" == "inode" ]] && continue
    [ -z "$inode" ] && continue

    # Clean up the name
    clean_name="${name#/}"
    dir_path=$(dirname "$clean_name")
    mkdir -p "$OUTPUT_DIR/$dir_path"

    # Extract file
    output_file="$OUTPUT_DIR/$clean_name"
    icat "$IMAGE" "$inode" > "$output_file" 2>/dev/null

    if [ $? -eq 0 ] && [ -s "$output_file" ]; then
        echo "[OK] $clean_name (inode: $inode, size: $(wc -c < "$output_file") bytes)"
    else
        echo "[FAIL] $clean_name (inode: $inode)"
        rm -f "$output_file"
    fi
done

echo ""
echo "Recovery complete."
echo "Total files recovered: $(find "$OUTPUT_DIR" -type f | wc -l)"
echo "Total size: $(du -sh "$OUTPUT_DIR" | cut -f1)"
```

**Usage:**
```bash
chmod +x recover_all_deleted.sh
./recover_all_deleted.sh usb_image.dd
```

#### Script 2: Recover Specific File Types

```bash
#!/bin/bash
# recover_by_type.sh — Recover files by extension

IMAGE="$1"
EXTENSION="$2"
OUTPUT_DIR="recovered_${EXTENSION}_$(date +%Y%m%d)"

if [ -z "$IMAGE" ] || [ -z "$EXTENSION" ]; then
    echo "Usage: $0 <image> <extension>"
    echo "Example: $0 usb_image.dd pdf"
    exit 1
fi

mkdir -p "$OUTPUT_DIR"

echo "=== Recovering *.$EXTENSION files ==="

# Find files with matching extension
fls -r -d -m "/" "$IMAGE" | grep -i "\.${EXTENSION}$" | while IFS='|' read -r timestamp name inode flags uid gid size; do
    [ -z "$inode" ] && continue

    clean_name="${name#/}"
    filename=$(basename "$clean_name")
    icat "$IMAGE" "$inode" > "$OUTPUT_DIR/$filename" 2>/dev/null

    if [ -s "$OUTPUT_DIR/$filename" ]; then
        echo "Recovered: $filename (inode: $inode)"
    else
        echo "Failed: $filename (inode: $inode)"
        rm -f "$OUTPUT_DIR/$filename"
    fi
done

echo ""
echo "Recovered $(ls "$OUTPUT_DIR" | wc -l) files to $OUTPUT_DIR"
```

**Usage:**
```bash
chmod +x recover_by_type.sh
./recover_by_type.sh usb_image.dd pdf
./recover_by_type.sh usb_image.dd docx
```

#### Script 3: Forensic Evidence Extraction

```bash
#!/bin/bash
# extract_evidence.sh — Extract specific evidence files with verification

IMAGE="$1"
OUTPUT_DIR="evidence_$(date +%Y%m%d_%H%M%S)"
EVIDENCE_LOG="$OUTPUT_DIR/evidence_log.txt"

if [ -z "$IMAGE" ]; then
    echo "Usage: $0 <filesystem_image>"
    exit 1
fi

mkdir -p "$OUTPUT_DIR"

{
    echo "=== Evidence Extraction Log ==="
    echo "Image: $IMAGE"
    echo "Image MD5: $(md5sum "$IMAGE" | cut -d' ' -f1)"
    echo "Image SHA256: $(sha256sum "$IMAGE" | cut -d' ' -f1)"
    echo "Extraction Date: $(date -u)"
    echo "Analyst: $(whoami)"
    echo ""
    echo "=== Extracted Files ==="
} > "$EVIDENCE_LOG"

# List all files with inodes
echo "Enumerating files..."
fls -r -d -m "/" "$IMAGE" > "$OUTPUT_DIR/all_files.csv"

# Extract each file
while IFS='|' read -r timestamp name inode flags uid gid size; do
    [[ "$inode" == "inode" ]] && continue
    [ -z "$inode" ] && continue

    clean_name="${name#/}"
    dir_path=$(dirname "$clean_name")
    mkdir -p "$OUTPUT_DIR/files/$dir_path"

    output_file="$OUTPUT_DIR/files/$clean_name"
    icat "$IMAGE" "$inode" > "$output_file" 2>/dev/null

    if [ -s "$output_file" ]; then
        file_hash=$(sha256sum "$output_file" | cut -d' ' -f1)
        echo "EXTRACTED|$clean_name|$inode|$file_hash|$(wc -c < "$output_file")" >> "$EVIDENCE_LOG"
    fi
done < "$OUTPUT_DIR/all_files.csv"

echo "" >> "$EVIDENCE_LOG"
echo "=== Summary ===" >> "$EVIDENCE_LOG"
echo "Total files extracted: $(find "$OUTPUT_DIR/files" -type f | wc -l)" >> "$EVIDENCE_LOG"
echo "Total size: $(du -sh "$OUTPUT_DIR/files" | cut -f1)" >> "$EVIDENCE_LOG"

echo "Evidence extraction complete. Log: $EVIDENCE_LOG"
```

### Python Scripting with icat

#### Python 1: icat Extraction Module

```python
#!/usr/bin/env python3
"""
icat_extractor.py — Python wrapper for icat forensic file extraction.
"""

import subprocess
import hashlib
from dataclasses import dataclass
from typing import Optional, List, Dict
from pathlib import Path
from datetime import datetime


@dataclass
class ExtractionResult:
    """Result of a file extraction operation."""
    inode: int
    original_path: Optional[str]
    output_path: Optional[str]
    size: int
    sha256: Optional[str]
    md5: Optional[str]
    success: bool
    error: Optional[str] = None


class IcatExtractor:
    """Wrapper class for the icat forensic tool."""

    def __init__(self, image_path: str, filesystem_type: Optional[str] = None,
                 offset: Optional[int] = None):
        self.image_path = Path(image_path)
        self.filesystem_type = filesystem_type
        self.offset = offset
        self._validate_image()

    def _validate_image(self):
        if not self.image_path.exists():
            raise FileNotFoundError(f"Image not found: {self.image_path}")

    def extract_by_inode(self, inode: int, output_path: str,
                         preserve_name: bool = True) -> ExtractionResult:
        """
        Extract file contents by inode number.

        Args:
            inode: The inode number to extract
            output_path: Directory or file path for output
            preserve_name: Whether to use original filename

        Returns:
            ExtractionResult with extraction details
        """
        cmd = ["icat"]

        if self.filesystem_type:
            cmd.extend(["-f", self.filesystem_type])

        if self.offset is not None:
            cmd.extend(["-o", str(self.offset)])

        cmd.extend([str(self.image_path), str(inode)])

        try:
            result = subprocess.run(cmd, capture_output=True, timeout=120)

            if result.returncode == 0 and result.stdout:
                output = Path(output_path)
                if output.is_dir():
                    output = output / f"inode_{inode}"

                with open(output, "wb") as f:
                    f.write(result.stdout)

                # Calculate hashes
                sha256_hash = hashlib.sha256(result.stdout).hexdigest()
                md5_hash = hashlib.md5(result.stdout).hexdigest()

                return ExtractionResult(
                    inode=inode,
                    original_path=None,
                    output_path=str(output),
                    size=len(result.stdout),
                    sha256=sha256_hash,
                    md5=md5_hash,
                    success=True
                )
            else:
                return ExtractionResult(
                    inode=inode,
                    original_path=None,
                    output_path=None,
                    size=0,
                    sha256=None,
                    md5=None,
                    success=False,
                    error=result.stderr.decode() if result.stderr else "Extraction failed"
                )

        except subprocess.TimeoutExpired:
            return ExtractionResult(
                inode=inode,
                original_path=None,
                output_path=None,
                size=0,
                sha256=None,
                md5=None,
                success=False,
                error="Extraction timed out"
            )

    def extract_multiple(self, inodes: List[int], output_dir: str) -> List[ExtractionResult]:
        """
        Extract multiple files by inode numbers.

        Args:
            inodes: List of inode numbers to extract
            output_dir: Directory to save extracted files

        Returns:
            List of ExtractionResult objects
        """
        output_path = Path(output_dir)
        output_path.mkdir(parents=True, exist_ok=True)

        results = []
        for inode in inodes:
            result = self.extract_by_inode(inode, str(output_path))
            results.append(result)

        return results

    def extract_with_ffind(self, inode: int, output_dir: str) -> ExtractionResult:
        """
        Extract file and attempt to preserve original path.

        Args:
            inode: The inode number to extract
            output_dir: Base directory for extraction

        Returns:
            ExtractionResult with path information
        """
        # First, find the original path
        cmd = ["ffind", str(self.image_path), str(inode)]
        try:
            result = subprocess.run(cmd, capture_output=True, text=True, timeout=30)
            original_path = result.stdout.strip() if result.returncode == 0 else None
        except Exception:
            original_path = None

        # Prepare output path
        if original_path:
            output_path = Path(output_dir) / original_path.lstrip("/")
        else:
            output_path = Path(output_dir) / f"inode_{inode}"

        output_path.parent.mkdir(parents=True, exist_ok=True)

        # Extract the file
        return self.extract_by_inode(inode, str(output_path))

    def analyze_file(self, inode: int) -> Dict:
        """
        Extract file and perform basic analysis.

        Args:
            inode: The inode number to analyze

        Returns:
            Dictionary with extraction and analysis results
        """
        # Extract to temporary location
        import tempfile
        with tempfile.NamedTemporaryFile(delete=False) as tmp:
            tmp_path = tmp.name

        result = self.extract_by_inode(inode, tmp_path)

        if not result.success:
            return {"success": False, "error": result.error}

        analysis = {
            "inode": inode,
            "size": result.size,
            "sha256": result.sha256,
            "md5": result.md5,
            "output_path": result.output_path,
        }

        # Determine file type
        try:
            type_result = subprocess.run(
                ["file", result.output_path],
                capture_output=True, text=True
            )
            analysis["file_type"] = type_result.stdout.split(":")[1].strip()
        except Exception:
            analysis["file_type"] = "unknown"

        # Check if text file
        try:
            with open(result.output_path, "rb") as f:
                sample = f.read(1024)
                is_text = all(32 <= byte < 127 or byte in (9, 10, 13) for byte in sample)
                analysis["is_text"] = is_text
        except Exception:
            analysis["is_text"] = False

        # Clean up
        Path(tmp_path).unlink(missing_ok=True)

        return analysis


def main():
    import argparse

    parser = argparse.ArgumentParser(description="Forensic file extraction wrapper")
    parser.add_argument("image", help="Path to filesystem image")
    parser.add_argument("inodes", nargs="+", type=int, help="Inode numbers to extract")
    parser.add_argument("-o", "--output", default="recovered", help="Output directory")
    parser.add_argument("-f", "--fstype", help="Filesystem type")
    parser.add_argument("--offset", type=int, help="Byte offset to filesystem")
    parser.add_argument("--analyze", action="store_true", help="Analyze extracted files")
    parser.add_argument("--csv", action="store_true", help="Output as CSV")

    args = parser.parse_args()

    extractor = IcatExtractor(args.image, args.fstype, args.offset)

    if args.analyze:
        for inode in args.inodes:
            result = extractor.analyze_file(inode)
            if args.csv:
                print(f"{inode},{result.get('size', 0)},{result.get('sha256', 'N/A')},{result.get('file_type', 'N/A')}")
            else:
                print(f"Inode {inode}:")
                for key, value in result.items():
                    print(f"  {key}: {value}")
                print()
    else:
        results = extractor.extract_multiple(args.inodes, args.output)
        if args.csv:
            print("inode,output_path,size,sha256,success")
            for r in results:
                print(f"{r.inode},{r.output_path or 'N/A'},{r.size},{r.sha256 or 'N/A'},{r.success}")
        else:
            for r in results:
                status = "OK" if r.success else "FAIL"
                print(f"[{status}] Inode {r.inode}: {r.output_path or r.error}")


if __name__ == "__main__":
    main()
```

**Usage:**
```bash
# Basic extraction
python3 icat_extractor.py usb_image.dd 58493 12 100

# Extract with analysis
python3 icat_extractor.py --analyze usb_image.dd 58493

# CSV output
python3 icat_extractor.py --csv usb_image.dd 58493 12

# Custom output directory
python3 icat_extractor.py -o evidence/ usb_image.dd 58493
```

---

## Chapter 5: Advanced Usage

### Advanced Flag Combinations

```bash
# Extract with inode information
icat -i usb_image.dd 58493

# Extract slack space only
icat -s usb_image.dd 58493 > slack_data.bin

# Extract NTFS alternate data stream
icat -e -f ntfs windows_image.dd 67 > ads_content.txt

# Force filesystem type with offset
icat -f ext4 -o 2048 disk_image.dd 58493 > recovered.txt

# Verbose extraction
icat -v usb_image.dd 58493 > recovered_file.bin
```

### Working with Multi-Partition Images

```bash
# Find partition layout
mmls disk_image.dd

# Extract from specific partition
icat -o $((2048 * 512)) disk_image.dd 58493 > recovered.txt

# Extract from second partition
icat -o $((206848 * 512)) disk_image.dd 1234 > recovered2.txt
```

### Bulk Extraction Workflow

```bash
#!/bin/bash
# bulk_extract.sh — Complete forensic extraction pipeline

IMAGE="$1"
OUTPUT_DIR="extraction_$(date +%Y%m%d_%H%M%S)"

mkdir -p "$OUTPUT_DIR"

echo "=== Complete Extraction Pipeline ==="

# Step 1: Filesystem information
echo "Step 1: Filesystem analysis..."
fsstat "$IMAGE" > "$OUTPUT_DIR/fs_info.txt"

# Step 2: List all files
echo "Step 2: File enumeration..."
fls -r -d -m "/" "$IMAGE" > "$OUTPUT_DIR/all_files.csv"

# Step 3: List deleted files
echo "Step 3: Deleted file enumeration..."
fls -r -d -s "$IMAGE" > "$OUTPUT_DIR/deleted_files.txt"

# Step 4: Extract all files
echo "Step 4: File extraction..."
mkdir -p "$OUTPUT_DIR/files"

while IFS='|' read -r timestamp name inode flags uid gid size; do
    [[ "$inode" == "inode" ]] && continue
    [ -z "$inode" ] && continue

    clean_name="${name#/}"
    dir_path=$(dirname "$clean_name")
    mkdir -p "$OUTPUT_DIR/files/$dir_path"

    icat "$IMAGE" "$inode" > "$OUTPUT_DIR/files/$clean_name" 2>/dev/null
done < "$OUTPUT_DIR/all_files.csv"

# Step 5: Generate summary
echo "Step 5: Generating summary..."
{
    echo "=== Extraction Summary ==="
    echo "Image: $IMAGE"
    echo "Date: $(date -u)"
    echo "Total files: $(find "$OUTPUT_DIR/files" -type f | wc -l)"
    echo "Total size: $(du -sh "$OUTPUT_DIR/files" | cut -f1)"
    echo ""
    echo "File Types:"
    find "$OUTPUT_DIR/files" -type f -exec file {} \; | awk -F: '{print $2}' | sort | uniq -c | sort -rn
} > "$OUTPUT_DIR/summary.txt"

echo "Pipeline complete. Results in: $OUTPUT_DIR"
```

### Integrating with Hash Databases

```bash
#!/bin/bash
# hash_check.sh — Extract and check files against known hash databases

IMAGE="$1"
HASH_DB="$2"  # File containing known hashes (one per line)
OUTPUT_DIR="hash_check_results"

mkdir -p "$OUTPUT_DIR"

echo "=== Hash Database Check ==="

# Extract all files
fls -r -m "/" "$IMAGE" | while IFS='|' read -r timestamp name inode flags uid gid size; do
    [[ "$inode" == "inode" ]] && continue
    [ -z "$inode" ] && continue

    clean_name="${name#/}"
    temp_file="/tmp/extracted_$inode"

    # Extract file
    icat "$IMAGE" "$inode" > "$temp_file" 2>/dev/null

    if [ -s "$temp_file" ]; then
        file_hash=$(sha256sum "$temp_file" | cut -d' ' -f1)

        # Check against hash database
        if grep -q "$file_hash" "$HASH_DB"; then
            echo "[MATCH] $clean_name (inode: $inode, hash: $file_hash)"
            cp "$temp_file" "$OUTPUT_DIR/match_${inode}_$(basename "$clean_name")"
        fi
    fi

    rm -f "$temp_file"
done

echo ""
echo "Check complete. Matches saved to: $OUTPUT_DIR"
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: Ransomware Recovery

**Background**: A company was hit by ransomware. The original files were deleted and replaced with encrypted versions. You need to recover the original files.

**Investigation Steps:**

```bash
# 1. Image the affected system
dc3dd if=/dev/sda hash=sha256 log=ransomware_evidence.log of=workstation.dd

# 2. Get filesystem information
fsstat workstation.dd

# 3. List deleted files (originals before encryption)
fls -r -d -m "/" workstation.dd > deleted_files.csv

# 4. Filter for common document types
grep -iE "\.(docx?|xlsx?|pdf|txt|csv)$" deleted_files.csv > documents.csv

# 5. Extract each deleted document
mkdir -p recovered_documents/
while IFS='|' read -r timestamp name inode flags uid gid size; do
    [ -z "$inode" ] && continue
    clean_name="${name#/}"
    icat workstation.dd "$inode" > "recovered_documents/$(basename "$clean_name")" 2>/dev/null
    if [ -s "recovered_documents/$(basename "$clean_name")" ]; then
        echo "Recovered: $clean_name"
    fi
done < documents.csv

# 6. Verify recovery
echo ""
echo "Recovery Summary:"
echo "Files recovered: $(ls recovered_documents/ | wc -l)"
echo "Total size: $(du -sh recovered_documents/ | cut -f1)"
```

### Scenario 2: Malware Sample Recovery

**Background**: A malware sample dropped files during execution. You need to recover these dropped files for analysis.

**Investigation Steps:**

```bash
# 1. Image the compromised system
dc3dd if=/dev/sdb hash=sha256 log=malware_evidence.log of=compromised.dd

# 2. List all files including deleted
fls -r -d -m "/" compromised.dd > all_files.csv

# 3. Look for suspicious file types
grep -iE "\.(exe|dll|scr|bat|ps1|vbs|js|hta)$" all_files.csv > suspicious.csv

# 4. Check temp directories
fls -r -d compromised.dd /tmp/ >> suspicious.csv
fls -r -d compromised.dd /var/tmp/ >> suspicious.csv

# 5. Extract suspicious files
mkdir -p malware_samples/
while IFS='|' read -r timestamp name inode flags uid gid size; do
    [ -z "$inode" ] && continue
    clean_name="${name#/}"
    icat compromised.dd "$inode" > "malware_samples/sample_$inode" 2>/dev/null
    if [ -s "malware_samples/sample_$inode" ]; then
        echo "Extracted: $clean_name (inode: $inode)"
    fi
done < suspicious.csv

# 6. Analyze extracted samples
echo ""
echo "=== Sample Analysis ==="
for sample in malware_samples/*; do
    echo "--- $(basename "$sample") ---"
    file "$sample"
    sha256sum "$sample"
    strings "$sample" | head -5
    echo ""
done
```

### Scenario 3: Data Breach Investigation

**Background**: An employee is suspected of exfiltrating sensitive data. You need to recover evidence of data transfer.

**Investigation Steps:**

```bash
# 1. Image the employee's workstation
dc3dd if=/dev/sda hash=sha256 log=breach_evidence.log of=workstation.dd

# 2. Get filesystem overview
fsstat workstation.dd

# 3. List all files including deleted
fls -r -d -m "/" workstation.dd > all_files.csv

# 4. Search for archive files (common exfiltration method)
grep -iE "\.(zip|rar|7z|tar|gz)$" all_files.csv > archives.csv

# 5. Search for cloud storage sync folders
fls -r -d workstation.dd /home/*/Dropbox/ >> archives.csv
fls -r -d workstation.dd /home/*/Google\ Drive/ >> archives.csv
fls -r -d workstation.dd /home/*/OneDrive/ >> archives.csv

# 6. Extract archives
mkdir -p recovered_archives/
while IFS='|' read -r timestamp name inode flags uid gid size; do
    [ -z "$inode" ] && continue
    clean_name="${name#/}"
    icat workstation.dd "$inode" > "recovered_archives/$(basename "$clean_name")" 2>/dev/null
    if [ -s "recovered_archives/$(basename "$clean_name")" ]; then
        echo "Extracted: $clean_name (size: $(wc -c < "recovered_archives/$(basename "$clean_name")") bytes)"
    fi
done < archives.csv

# 7. Check for USB device mounts
echo ""
echo "=== USB Device Activity ==="
fls -r -d workstation.dd /media/ 2>/dev/null
fls -r -d workstation.dd /mnt/ 2>/dev/null

# 8. Generate evidence report
{
    echo "=== Data Breach Evidence Report ==="
    echo "Image: workstation.dd"
    echo "Date: $(date -u)"
    echo ""
    echo "Archives found:"
    ls -la recovered_archives/
    echo ""
    echo "Total archive size: $(du -sh recovered_archives/ | cut -f1)"
} > evidence_report.txt
```

---

## Chapter 7: Detection & Defense

### How Attackers Avoid Detection

Understanding how icat works helps defenders understand attack techniques:

1. **File Wiping**: Tools like `shred` overwrite file data blocks, making icat recovery impossible.
2. **Inode Overwriting**: Writing new data overwrites the inode, replacing original content.
3. **Full Disk Encryption**: Prevents icat from accessing filesystem structures.
4. **Secure Deletion**: Multiple overwrite passes destroy recoverable data.
5. **Timestamp Manipulation**: Modifying inode timestamps confuses forensic analysis.

### Defensive Measures

```bash
# Monitor file deletions (Linux audit)
auditctl -w /home -p w -k file_deletions
auditctl -w /tmp -p w -k temp_files

# Enable filesystem journaling for better recovery
tune2fs -j /dev/sda2

# Use file integrity monitoring
aide --init
sudo mv /var/lib/aide/aide.db.new /var/lib/aide/aide.db

# Regular forensic imaging of critical systems
dc3dd if=/dev/sda2 hash=sha256 log=backup.log of=backups/system_$(date +%Y%m%d).dd
```

### Forensic Best Practices

```bash
# Always work on copies
cp usb_image.dd usb_image_work.dd

# Document everything
echo "Extraction: $(date -u)" >> case_log.txt
echo "Inode: 58493" >> case_log.txt
echo "Original path: /Documents/resume.docx" >> case_log.txt

# Verify hashes
sha256sum usb_image.dd
# Compare with documented hash
```

---

## Chapter 8: Troubleshooting

### FAQ

**Q1: icat outputs nothing or produces an empty file.**

A: Common causes and solutions:
- Inode doesn't exist or has been reallocated.
- Filesystem type is wrong. Use `-f` to specify.
- Image is corrupted. Verify with `fsstat`.
- Try with `-v` for verbose output.

```bash
# Diagnostic steps
istat usb_image.dd 58493  # Check inode exists
fsstat -f ext4 usb_image.dd  # Verify filesystem
icat -v usb_image.dd 58493  # Verbose extraction
```

**Q2: icat produces corrupted output.**

A: Possible causes:
- Filesystem corruption — try `e2fsck` on the original (not the image).
- Wrong filesystem type specified.
- Inode points to corrupted blocks.

```bash
# Verify inode metadata
istat usb_image.dd 58493

# Check file type
icat usb_image.dd 58493 | file -

# Try extracting with different filesystem type
icat -f ext3 usb_image.dd 58493 > test.txt
```

**Q3: icat is very slow on large files.**

A: Large files require more I/O. Tips:
- Use local disk, not network storage.
- For very large files, consider `tsk_recover` for bulk extraction.
- Ensure the image is not compressed or encrypted.

```bash
# Check file size
istat usb_image.dd 58493 | grep "Size:"

# For bulk extraction
tsk_recover usb_image.dd recovered/
```

**Q4: How do I know if the extraction was successful?**

A: Verify by:
- Checking file size matches expected size from `istat`.
- Comparing hash with known value.
- Running `file` command to verify type.
- Opening the file in appropriate application.

```bash
# Verify extraction
icat usb_image.dd 58493 > recovered.docx
file recovered.docx
sha256sum recovered.docx
ls -la recovered.docx
```

**Q5: Can icat recover encrypted files?**

A: icat recovers the encrypted content, but you'll need the decryption key to access the actual data. The extracted file will be in its encrypted form.

**Q6: What's the difference between icat and tsk_recover?**

A: `icat` extracts a single file by inode number. `tsk_recover` bulk-recovers all files from a filesystem image. Use `icat` for targeted extraction, `tsk_recover` for full recovery.

```bash
# Single file extraction
icat usb_image.dd 58493 > single_file.txt

# Bulk recovery
tsk_recover usb_image.dd recovered_directory/
```

---

## Resources

### Official Documentation

- **TSK Manual**: https://sleuthkit.org/sleuthkit/man/
- **icat Man Page**: `man icat`
- **GitHub**: https://github.com/sleuthkit/sleuthkit
- **Kali Tools**: https://www.kali.org/tools/sleuthkit/

### Books and References

- *File System Forensic Analysis* by Brian Carrier — Comprehensive TSK reference
- *The Sleuth Kit and Autopsy* by Brian Carrier — Practical TSK guide
- *Linux Forensics* by Phillip Andrews — Linux-specific forensics

### Related TSK Tools

| Tool | Purpose | Complements icat |
|------|---------|-----------------|
| `fls` | List files | Provides inodes for icat extraction |
| `ffind` | Find file by inode | Resolves paths for icat targets |
| `ils` | List inodes | Identifies inodes for icat extraction |
| `istat` | Inode statistics | Shows metadata for icat targets |
| `fsstat` | Filesystem statistics | Provides context for icat analysis |
| `tsk_recover` | Bulk file recovery | Alternative to individual icat calls |

### Online Communities

- **Sleuth Kit Forums**: https://digitalforensicsforum.com/
- **DFIR Reddit**: https://reddit.com/r/DigitalForensics
- **SANS DFIR**: https://www.sans.org/dfir/

---

**TSK Install**: `sudo apt install sleuthkit` | **Man Page**: `man icat`
