---
tool_name: fls
display_name: fls
mitre_tactic: "analysis"
mitre_tactic_id: "TA0007"
install_command: "sudo apt install sleuthkit"
github_repo: "https://github.com/sleuthkit/sleuthkit"
kali_tools_page: "https://www.kali.org/tools/sleuthkit/"
difficulty_level: "beginner"
requires_root: false
gui_available: false
tags: ["forensics", "filesystem", "sleuthkit", "tsk", "file-listing", "deleted-files", "directory", "analysis"]
related_tools: ["ffind", "icat", "fsstat", "ils", "istat", "tsk_recover", "mmls"]
---

# fls — Forensic File and Directory Listing

## Chapter 1: Introduction & Overview

### What fls Does

fls is a forensic file listing command from The Sleuth Kit (TSK) that displays file and directory names within a filesystem image. Think of it as a forensic version of `ls` — but with critical differences. While `ls` only shows active, mounted files, `fls` operates directly on disk images and can recover deleted file names that no longer appear in the live filesystem.

fls reads the filesystem metadata structures (inodes, directory entries, MFT records) directly from the raw image, bypassing the operating system's file access layer. This means it can see:

- **Active files and directories** — standard filesystem contents
- **Deleted file names** — directory entries marked as deleted but not yet overwritten
- **Deleted directory entries** — remnants of removed folders
- **Files in unallocated space** — fragments of directory structures in free space
- **Files across partitions** — when pointed at the correct filesystem offset

### Why fls is Essential for Forensics

Standard file operations go through the OS kernel, which only shows currently allocated files. Forensic analysts need to see the complete picture, including what has been removed. fls provides this by directly parsing filesystem structures.

Key advantages over standard file listing:

1. **No mounting required** — works on raw disk images without mounting
2. **Deleted file recovery** — shows names of files that have been deleted
3. **Cross-platform** — handles ext2/3/4, FAT, NTFS, HFS+, and more
4. **Timeline-ready** — output can be formatted for timeline analysis tools
5. **Scriptable** — clean text output for automated analysis pipelines

### Position in The Sleuth Kit

fls is one of the most frequently used TSK tools. It sits in the filesystem analysis layer:

| Layer | Tools | Purpose |
|-------|-------|---------|
| Volume System | `mmls`, `mmstat` | Partition table analysis |
| Filesystem Metadata | `dbstat`, `dls` | Raw block-level analysis |
| **Filesystem Analysis** | **`fls`, `ffind`, `ils`** | **File and inode operations** |
| File Recovery | `icat`, `tsk_recover` | Data extraction |

fls is typically the first tool run after identifying a filesystem with `fsstat` or `mmls`. It provides the file listing that guides all subsequent investigation.

### Comparison with Standard Tools

| Feature | `ls` | `fls` |
|---------|------|-------|
| Requires mounting | Yes | No |
| Shows deleted files | No | Yes |
| Works on disk images | No | Yes |
| Shows file metadata | Limited | Full |
| Timeline format output | No | Yes |
| Works on Windows images | No | Yes |

---

## Chapter 2: Installation & Setup

### Installing The Sleuth Kit

fls is included in the Sleuth Kit package:

```bash
# Update package lists
sudo apt update

# Install Sleuth Kit (includes fls and all TSK tools)
sudo apt install sleuthkit

# Verify fls is available
which fls
fls -V
```

### Verifying the Installation

```bash
# Check fls is in PATH
which fls

# Display version
fls -V

# List all installed TSK tools
dpkg -L sleuthkit | grep bin/

# Quick test with fls help
fls -h
```

### Building from Source

For the latest development version:

```bash
# Install dependencies
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

Forensic best practice requires working on copies, never originals:

```bash
# Create case directory
mkdir -p ~/forensics/cases/case_001
cd ~/forensics/cases/case_001

# Create verified evidence copy
dc3dd if=/dev/sdb hash=sha256 log=evidence.log of=usb_image.dd

# Verify hash
sha256sum usb_image.dd

# Create working copy (preserve original)
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

fls supports these filesystem types:

| Filesystem | Full Support | Deleted Files | Notes |
|------------|:------------:|:-------------:|-------|
| ext2 | Yes | Yes | Primary Linux FS |
| ext3 | Yes | Yes | With journal |
| ext4 | Yes | Yes | Most common Linux FS |
| FAT12 | Yes | Yes | Floppy, small media |
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
fls [options] <image> [path]
```

**Positional Arguments:**

| Argument | Description |
|----------|-------------|
| `<image>` | Path to filesystem image or device node |
| `[path]` | Optional directory path within image to list |

### Complete Flag Reference

| Flag | Long Form | Description |
|------|-----------|-------------|
| `-r` | `--recursive` | List directories recursively (like `ls -R`) |
| `-d` | `--deleted` | Show deleted files and directories |
| `-a` | `--all` | Show all entries (allocated and unallocated) |
| `-l` | `--long` | Long listing format (detailed metadata) |
| `-m` | `--mac` | Output in mactime format for timeline analysis |
| `-t` | `--text` | Display results as text (default) |
| `-f` | `--fstype` | Specify filesystem type |
| `-o` | `--offset` | Byte offset to the filesystem |
| `-s` | `--inode` | Display inode number for each entry |
| `-i` | `--indir` | List the inode number of directories |
| `-e` | `--entry` | List deleted entries only |
| `-k` | `--meta` | Display metadata for each entry |
| `-p` | `--path` | Display full path for each entry |
| `-C` | `--csv` | CSV output format |
| `-B` | `--body` | Body file format for mactime |
| `-h` | `--help` | Display help and exit |
| `-V` | `--version` | Print version and exit |
| `-v` | `--verbose` | Verbose output |
| `-z` | `--zone` | Time zone for timestamps |

### Example 1: Basic Directory Listing

```bash
# List files at root of filesystem image
fls usb_image.dd
```

**Output:**
```
r/r * 12:       resume.doc
r/r * 13:       notes.txt
r/d * 14:       Documents
r/d * 15:       Pictures
r/r * 16:       .bashrc
```

The format is: type/link-count flags inode-number: name
- `r/r` = regular file
- `r/d` = directory
- `*` = has been deleted (unallocated)

### Example 2: Recursive Listing

```bash
# List all files recursively
fls -r usb_image.dd
```

**Output:**
```
r/d * 14:       Documents
r/r * 20:       Documents/resume.doc
r/r * 21:       Documents/cover_letter.docx
r/d * 15:       Pictures
r/r * 30:       Pictures/photo.jpg
r/r * 31:       Pictures/screenshot.png
```

### Example 3: Showing Deleted Files

```bash
# Show only deleted files and directories
fls -d usb_image.dd
```

**Output:**
```
r/r * 58493:    old_report.docx
r/r * 58494:    financial_data.xlsx
r/d * 58495:    deleted_folder
r/r * 58496:    deleted_folder/secret.txt
```

### Example 4: Long Format Listing

```bash
# Detailed listing with metadata
fls -l usb_image.dd
```

**Output:**
```
r/r * 12:       resume.doc       1024    1000    1000    4096    16 Jan 2024 14:32:01
r/r * 13:       notes.txt        512     1000    1000    2048    16 Jan 2024 15:45:22
r/d * 14:       Documents        4096    1000    1000    4096    16 Jan 2024 14:30:00
```

### Example 5: Timeline Format

```bash
# Output in mactime body file format
fls -m "/" usb_image.dd
```

**Output:**
```
0|/resume.doc|12|r/r++|1000|1000|4096|1705405921|1705405921|1705405921|1705405921
0|/notes.txt|13|r/r++|1000|1000|2048|1705409122|1705409122|1705409122|1705409122
```

### Example 6: Specifying a Subdirectory

```bash
# List contents of a specific directory
fls usb_image.dd Documents
```

### Example 7: With Inode Numbers

```bash
# Show inode numbers alongside file names
fls -s usb_image.dd
```

**Output:**
```
r/r * 12 (FILE:resume.doc):       resume.doc
r/r * 13 (FILE:notes.txt):        notes.txt
```

### Example 8: CSV Output

```bash
# CSV format for spreadsheet analysis
fls -C usb_image.dd
```

**Output:**
```
type,inode,deleted,name,size,uid,gid,mode,atime,mtime,ctime
r/r,12,y,resume.doc,4096,1000,1000,0100644,1705405921,1705405921,1705405921
```

### Exit Codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | Error or no files found |
| 2 | Invalid image or filesystem |

---

## Chapter 4: Intermediate Usage

### Bash Scripting with fls

#### Script 1: Complete File Inventory

Generate a comprehensive file listing with all metadata:

```bash
#!/bin/bash
# file_inventory.sh — Generate complete forensic file inventory

IMAGE="$1"
OUTPUT_DIR="inventory_$(date +%Y%m%d_%H%M%S)"

if [ -z "$IMAGE" ]; then
    echo "Usage: $0 <filesystem_image>"
    exit 1
fi

mkdir -p "$OUTPUT_DIR"

echo "=== Forensic File Inventory ==="
echo "Image: $IMAGE"
echo "Output: $OUTPUT_DIR"
echo ""

# Full recursive listing
echo "Generating recursive listing..."
fls -r -d -s -m "/" "$IMAGE" > "$OUTPUT_DIR/all_files.csv"

# Active files only
echo "Generating active files list..."
fls -r -s "$IMAGE" > "$OUTPUT_DIR/active_files.txt"

# Deleted files only
echo "Generating deleted files list..."
fls -r -d -s "$IMAGE" > "$OUTPUT_DIR/deleted_files.txt"

# Timeline data
echo "Generating timeline data..."
fls -r -m "/" "$IMAGE" > "$OUTPUT_DIR/timeline.body"

# Summary statistics
echo "" > "$OUTPUT_DIR/summary.txt"
echo "=== Summary ===" >> "$OUTPUT_DIR/summary.txt"
echo "Total entries: $(wc -l < "$OUTPUT_DIR/all_files.csv")" >> "$OUTPUT_DIR/summary.txt"
echo "Active files: $(grep -c "^r/r" "$OUTPUT_DIR/active_files.txt")" >> "$OUTPUT_DIR/summary.txt"
echo "Deleted files: $(grep -c "^\*" "$OUTPUT_DIR/deleted_files.txt" || echo 0)" >> "$OUTPUT_DIR/summary.txt"
echo "Directories: $(grep -c "^r/d" "$OUTPUT_DIR/all_files.csv")" >> "$OUTPUT_DIR/summary.txt"

echo ""
cat "$OUTPUT_DIR/summary.txt"
echo ""
echo "Inventory complete."
```

**Usage:**
```bash
chmod +x file_inventory.sh
./file_inventory.sh usb_image.dd
```

#### Script 2: Recover Deleted Files

```bash
#!/bin/bash
# recover_deleted.sh — Recover all deleted files from a filesystem image

IMAGE="$1"
OUTPUT_DIR="recovered_$(date +%Y%m%d_%H%M%S)"

if [ -z "$IMAGE" ]; then
    echo "Usage: $0 <filesystem_image>"
    exit 1
fi

mkdir -p "$OUTPUT_DIR"

echo "=== Recovering Deleted Files ==="

# Get list of deleted files with inodes
fls -r -d -m "/" "$IMAGE" | while IFS='|' read -r timestamp name inode flags uid gid size; do
    # Skip header line
    [[ "$inode" == "inode" ]] && continue

    # Skip empty lines
    [ -z "$inode" ] && continue

    # Clean up the name (remove leading slash)
    clean_name="${name#/}"

    # Create output subdirectory structure
    dir_path=$(dirname "$clean_name")
    mkdir -p "$OUTPUT_DIR/$dir_path"

    # Extract file using icat
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
echo "Recovery complete. Files saved to: $OUTPUT_DIR"
echo "Total recovered: $(find "$OUTPUT_DIR" -type f | wc -l)"
```

**Usage:**
```bash
chmod +x recover_deleted.sh
./recover_deleted.sh usb_image.dd
```

#### Script 3: Find Specific File Types

```bash
#!/bin/bash
# find_filetypes.sh — Find files by extension in forensic image

IMAGE="$1"
PATTERN="$2"
RECURSIVE="${3:--r}"

if [ -z "$IMAGE" ] || [ -z "$PATTERN" ]; then
    echo "Usage: $0 <image> <pattern> [recursive_flag]"
    echo "Example: $0 usb_image.dd '*.docx'"
    echo "Example: $0 usb_image.dd '*.py' -r"
    exit 1
fi

echo "=== Searching for: $PATTERN ==="

# List all files and filter by pattern
fls $RECURSIVE -d -m "/" "$IMAGE" | grep -i "$PATTERN" | while IFS='|' read -r timestamp name inode flags uid gid size; do
    [ -z "$inode" ] && continue
    echo "$(echo "$name" | sed 's/^0\///') (inode: $inode, size: $size bytes)"
done
```

**Usage:**
```bash
chmod +x find_filetypes.sh
./find_filetypes.sh usb_image.dd '\.pdf$'
./find_filetypes.sh usb_image.dd '\.docx$' -r
```

### Python Scripting with fls

#### Python 1: fls Output Parser

```python
#!/usr/bin/env python3
"""
fls_parser.py — Parse fls output into structured data.
"""

import subprocess
import re
from dataclasses import dataclass, field
from typing import List, Optional
from pathlib import Path
from datetime import datetime


@dataclass
class FlsEntry:
    """Represents a single fls output entry."""
    entry_type: str          # r/r, r/d, v/v, etc.
    inode: int
    deleted: bool
    name: str
    full_path: Optional[str] = None
    size: Optional[int] = None
    uid: Optional[int] = None
    gid: Optional[int] = None
    mode: Optional[int] = None
    atime: Optional[datetime] = None
    mtime: Optional[datetime] = None
    ctime: Optional[datetime] = None


class FlsParser:
    """Parser for fls command output."""

    def __init__(self, image_path: str, filesystem_type: Optional[str] = None):
        self.image_path = Path(image_path)
        self.filesystem_type = filesystem_type
        self.entries: List[FlsEntry] = []

    def list_directory(self, path: Optional[str] = None,
                       recursive: bool = False,
                       show_deleted: bool = True,
                       long_format: bool = False) -> List[FlsEntry]:
        """
        List directory contents using fls.

        Args:
            path: Subdirectory path within image (None for root)
            recursive: List recursively
            show_deleted: Include deleted entries
            long_format: Use long format for metadata

        Returns:
            List of FlsEntry objects
        """
        cmd = ["fls"]

        if recursive:
            cmd.append("-r")

        if show_deleted:
            cmd.append("-d")

        if long_format:
            cmd.append("-l")

        cmd.append("-m")
        cmd.append("/")

        cmd.append(str(self.image_path))

        if path:
            cmd.append(path)

        result = subprocess.run(cmd, capture_output=True, text=True)

        self.entries = self._parse_output(result.stdout)
        return self.entries

    def _parse_output(self, output: str) -> List[FlsEntry]:
        """Parse fls mactime output into FlsEntry objects."""
        entries = []

        for line in output.strip().split("\n"):
            if not line or line.startswith("0|"):
                # Parse mactime body format
                parts = line.split("|")
                if len(parts) >= 4:
                    try:
                        entry = FlsEntry(
                            entry_type=parts[2] if len(parts) > 2 else "unknown",
                            inode=int(parts[3]) if parts[3].isdigit() else 0,
                            deleted="++" in parts[2],
                            name=parts[1],
                            size=int(parts[7]) if len(parts) > 7 and parts[7].isdigit() else None,
                            uid=int(parts[4]) if len(parts) > 4 and parts[4].isdigit() else None,
                            gid=int(parts[5]) if len(parts) > 5 and parts[5].isdigit() else None,
                        )
                        entries.append(entry)
                    except (ValueError, IndexError):
                        continue

        return entries

    def get_deleted_files(self) -> List[FlsEntry]:
        """Return only deleted entries."""
        return [e for e in self.entries if e.deleted]

    def get_directories(self) -> List[FlsEntry]:
        """Return only directory entries."""
        return [e for e in self.entries if "d" in e.entry_type]

    def get_files_by_extension(self, extension: str) -> List[FlsEntry]:
        """Filter entries by file extension."""
        return [e for e in self.entries if e.name.endswith(extension)]

    def get_large_files(self, min_size: int) -> List[FlsEntry]:
        """Filter entries by minimum size."""
        return [e for e in self.entries if e.size and e.size >= min_size]

    def export_csv(self, output_path: str):
        """Export entries to CSV file."""
        import csv

        with open(output_path, "w", newline="") as f:
            writer = csv.writer(f)
            writer.writerow(["type", "inode", "deleted", "name", "size", "uid", "gid"])
            for entry in self.entries:
                writer.writerow([
                    entry.entry_type,
                    entry.inode,
                    entry.deleted,
                    entry.name,
                    entry.size or "",
                    entry.uid or "",
                    entry.gid or ""
                ])

    def summary(self) -> dict:
        """Generate summary statistics."""
        total = len(self.entries)
        deleted = len(self.get_deleted_files())
        directories = len(self.get_directories())
        files = total - directories

        return {
            "total_entries": total,
            "files": files,
            "directories": directories,
            "deleted": deleted,
            "allocated": total - deleted
        }


def main():
    import argparse

    parser = argparse.ArgumentParser(description="Forensic file listing parser")
    parser.add_argument("image", help="Path to filesystem image")
    parser.add_argument("-p", "--path", help="Subdirectory path within image")
    parser.add_argument("-r", "--recursive", action="store_true", help="Recursive listing")
    parser.add_argument("-d", "--deleted", action="store_true", help="Show deleted files")
    parser.add_argument("-l", "--long", action="store_true", help="Long format")
    parser.add_argument("-f", "--fstype", help="Filesystem type")
    parser.add_argument("--csv", help="Export to CSV file")
    parser.add_argument("--summary", action="store_true", help="Show summary statistics")
    parser.add_argument("--ext", help="Filter by file extension")
    parser.add_argument("--min-size", type=int, help="Filter by minimum size")

    args = parser.parse_args()

    parser_obj = FlsParser(args.image, args.fstype)
    entries = parser_obj.list_directory(
        path=args.path,
        recursive=args.recursive,
        show_deleted=args.deleted,
        long_format=args.long
    )

    if args.summary:
        stats = parser_obj.summary()
        print(f"Total entries: {stats['total_entries']}")
        print(f"Files: {stats['files']}")
        print(f"Directories: {stats['directories']}")
        print(f"Deleted: {stats['deleted']}")
        print(f"Allocated: {stats['allocated']}")
    elif args.ext:
        filtered = parser_obj.get_files_by_extension(args.ext)
        for entry in filtered:
            del_mark = " [DELETED]" if entry.deleted else ""
            print(f"{entry.name} (inode: {entry.inode}){del_mark}")
    elif args.min_size:
        filtered = parser_obj.get_large_files(args.min_size)
        for entry in filtered:
            print(f"{entry.name}: {entry.size} bytes (inode: {entry.inode})")
    else:
        for entry in entries:
            del_mark = " [DELETED]" if entry.deleted else ""
            type_mark = " [DIR]" if "d" in entry.entry_type else ""
            print(f"{entry.name} (inode: {entry.inode}){type_mark}{del_mark}")

    if args.csv:
        parser_obj.export_csv(args.csv)
        print(f"\nExported to {args.csv}")


if __name__ == "__main__":
    main()
```

**Usage:**
```bash
# Basic listing
python3 fls_parser.py usb_image.dd

# Recursive listing with summary
python3 fls_parser.py -r --summary usb_image.dd

# Find deleted files
python3 fls_parser.py -r -d usb_image.dd

# Filter by extension
python3 fls_parser.py --ext .pdf usb_image.dd

# Export to CSV
python3 fls_parser.py -r --csv files.csv usb_image.dd
```

---

## Chapter 5: Advanced Usage

### Advanced Flag Combinations

```bash
# Maximum detail: recursive, deleted, long format, with inodes
fls -r -d -l -s usb_image.dd

# Timeline-ready output for all deleted files
fls -r -d -m "/" usb_image.dd > deleted_timeline.body

# Only show deleted directories and their contents
fls -r -e usb_image.dd

# Long format with metadata for deleted files
fls -d -l -k usb_image.dd

# CSV output with inodes for scripting
fls -r -d -C usb_image.dd > deleted_files.csv
```

### Working with Multiple Partitions

```bash
# First, identify partitions
mmls disk_image.dd

# List files in each partition
fls -o 2048 disk_image.dd     # Partition 1 (offset 2048 sectors)
fls -o 206848 disk_image.dd   # Partition 2 (offset 206848 sectors)
```

### Building Timelines

fls with mactime format is the foundation of forensic timeline analysis:

```bash
# Generate body file from fls
fls -r -m "/" usb_image.dd > body_file.txt

# Add timeline from ils (deleted file timestamps)
ils -e -m usb_image.dd >> body_file.txt

# Generate timeline visualization
mactime -b body_file.txt -d timeline_output/
```

### Combining with Other TSK Tools

```bash
# Complete forensic analysis pipeline
echo "=== Filesystem Info ==="
fsstat usb_image.dd

echo "=== All Files ==="
fls -r -m "/" usb_image.dd

echo "=== Deleted Files ==="
fls -r -d -m "/" usb_image.dd

echo "=== Inode Information ==="
ils -e usb_image.dd

echo "=== Specific File Recovery ==="
# Find inode of interest
INODE=$(fls -r -d usb_image.dd | grep "secret.txt" | grep -oP '\d+')
# Recover it
icat usb_image.dd "$INODE" > recovered_secret.txt
```

### Processing Large Evidence Sets

```bash
#!/bin/bash
# batch_flss.sh — Process multiple filesystem images

IMAGE_DIR="$1"
OUTPUT_DIR="batch_results_$(date +%Y%m%d)"

mkdir -p "$OUTPUT_DIR"

for image in "$IMAGE_DIR"/*.dd "$IMAGE_DIR"/*.raw "$IMAGE_DIR"/*.E01; do
    [ -f "$image" ] || continue

    basename=$(basename "$image" | sed 's/\.[^.]*$//')
    echo "Processing: $basename"

    # Generate complete inventory
    fls -r -d -m "/" "$image" > "$OUTPUT_DIR/${basename}_all.csv"

    # Generate deleted files list
    fls -r -d -s "$image" > "$OUTPUT_DIR/${basename}_deleted.txt"

    # Generate summary
    {
        echo "=== $basename ==="
        echo "Total entries: $(wc -l < "$OUTPUT_DIR/${basename}_all.csv")"
        echo "Deleted: $(grep -c "^\*" "$OUTPUT_DIR/${basename}_deleted.txt" || echo 0)"
    } > "$OUTPUT_DIR/${basename}_summary.txt"
done

echo "Batch processing complete. Results in: $OUTPUT_DIR"
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: Insider Threat Investigation

**Background**: An employee is suspected of copying proprietary data before resignation. Company policy requires investigation of all data transfers.

**Investigation Steps:**

```bash
# 1. Image the employee's workstation drive
dc3dd if=/dev/sda hash=sha256 log=evidence.log of=workstation.dd

# 2. Get filesystem overview
fsstat workstation.dd

# 3. List all files including deleted
fls -r -d -m "/" workstation.dd > full_listing.csv

# 4. Focus on user directories
fls -r -d workstation.dd /home/employee/ > user_files.txt

# 5. Find recently deleted files (likely data exfiltration)
fls -r -d -l workstation.dd /home/employee/ | grep -E "\.(xlsx|pdf|docx|zip|tar|7z)$"

# 6. Check USB mount points for copied data
fls -r -d workstation.dd /media/ > usb_files.txt

# 7. Recover specific suspicious files
INODE=$(fls -r -d workstation.dd /home/employee/ | grep "project_export" | grep -oP '\d+')
icat workstation.dd "$INODE" > recovered_export.zip

# 8. Verify the recovery
sha256sum recovered_export.zip
```

### Scenario 2: Malware Analysis — Recovering Dropped Files

**Background**: A malware sample was identified dropping files during execution. The system was imaged for analysis.

**Investigation Steps:**

```bash
# 1. Image the compromised system
dc3dd if=/dev/sdb hash=sha256 log=malware_evidence.log of=compromised.dd

# 2. List all files to identify the malware's activity
fls -r -d -m "/" compromised.dd > all_files.csv

# 3. Look for suspicious file types
fls -r -d compromised.dd | grep -iE "\.(exe|dll|scr|bat|ps1|vbs|js|hta)$"

# 4. Check temp directories (common malware drop location)
fls -r -d compromised.dd /tmp/
fls -r -d compromised.dd /var/tmp/
fls -r -d compromised.dd /home/*/.cache/

# 5. Find recently deleted files (malware cleanup)
fls -r -d -l compromised.dd | grep -E "Apr|May|Jun"  # Recent months

# 6. Extract suspicious files
for inode in $(fls -r -d compromised.dd | grep -iE "\.(exe|dll)$" | grep -oP '\d+'); do
    filename="sample_$inode"
    icat compromised.dd "$inode" > "$filename"
    file "$filename"
    sha256sum "$filename"
done

# 7. Analyze recovered samples
for f in sample_*; do
    echo "=== $f ==="
    file "$f"
    strings "$f" | head -20
done
```

### Scenario 3: Mobile Device Forensics

**Background**: A suspect's Android phone is seized. The device image contains app data that may contain evidence.

**Investigation Steps:**

```bash
# 1. Work with the extracted filesystem image
# (Assuming the phone was imaged with appropriate tools)

# 2. Identify filesystem type
fsstat android_data.dd

# 3. List all files recursively
fls -r -d -m "/" android_data.dd > android_files.csv

# 4. Search for messaging app data
fls -r -d android_data.dd | grep -iE "(whatsapp|messenger|telegram|signal|sms)"

# 5. Search for media files
fls -r -d android_data.dd | grep -iE "\.(jpg|jpeg|png|mp4|mp3|webm)$"

# 6. Search for deleted messages
fls -r -d android_data.dd /data/data/ | grep -i "databases"

# 7. Recover deleted message databases
for inode in $(fls -r -d android_data.dd | grep "msgstore" | grep -oP '\d+'); do
    icat android_data.dd "$inode" > "msgstore_$inode.db"
done

# 8. Search for browser history
fls -r -d android_data.dd | grep -i "chrome\|browser\|history"
```

---

## Chapter 7: Detection & Defense

### Understanding How fls Works (For Defenders)

fls reads filesystem metadata structures directly. This means:

1. **Deleted files leave traces**: Directory entries are marked deleted but not immediately overwritten.
2. **Timestamps persist**: Even after deletion, creation/modification timestamps may survive in inodes.
3. **Free space contains data**: Unallocated blocks may contain directory fragments.
4. **Journaling helps recovery**: ext3/4 journals can contain file metadata.

### Anti-Forensics Awareness

Attackers may try to prevent fls from finding evidence:

| Technique | Effect on fls | Defense |
|-----------|---------------|---------|
| Secure deletion (shred) | Overwrites data blocks | Use write blockers, image before deletion |
| Filesystem wiping | Destroys metadata | Regular backups, monitoring |
| Timestamp manipulation | Confuses timeline | Cross-reference multiple timestamp sources |
| Partition table modification | Hides partitions | Use `mmls` to verify partition layout |
| Steganography | Hides data in files | Hash analysis, entropy analysis |

### Defensive Monitoring

```bash
# Monitor for suspicious file deletions (Linux audit)
auditctl -w /home -p w -k deletions
auditctl -w /tmp -p w -k temp_files

# Check for evidence of anti-forensics
# Look for tools like shred, secure-delete in logs
grep -r "shred\|srm\|wipe" /var/log/

# Monitor USB device connections
journalctl -k | grep -i "usb\|sd[a-z]"
```

### File Integrity Monitoring

```bash
# Use AIDE to detect unauthorized file changes
aide --check

# Compare current state with baseline
aide --compare
```

---

## Chapter 8: Troubleshooting

### FAQ

**Q1: fls shows no files or shows very few files.**

A: Common causes and solutions:
- Wrong filesystem type: Use `fsstat` to identify, then `-f` to specify.
- Wrong offset: Use `mmls` to find partition offset, then `-o OFFSET`.
- Image corruption: Verify image integrity with hash comparison.
- Empty filesystem: The image may not contain a valid filesystem.

```bash
# Diagnostic steps
mmls disk_image.dd        # Find partition layout
fsstat -o 2048 disk_image.dd  # Check filesystem at offset
fls -f ext4 -o 2048 disk_image.dd  # Force type and offset
```

**Q2: fls is very slow on large images.**

A: fls scans all directory blocks. Speed tips:
- Specify filesystem type with `-f` to skip auto-detection.
- Use local disk, not network storage.
- Avoid unnecessary recursion if you only need root listing.
- Consider using `tsk_recover` for bulk extraction.

```bash
# Speed improvement
fls -f ext4 usb_image.dd

# If very slow, extract all files at once
tsk_recover usb_image.dd recovered/
```

**Q3: fls shows files I can't access with ffind or icat.**

A: This usually means the directory entry exists but the inode has been reallocated. The file name is visible but the data is gone.

```bash
# Check if inode exists
istat usb_image.dd <inode_number>

# If inode shows different file, original data is overwritten
```

**Q4: How do I interpret the file type codes in fls output?**

A: fls uses type codes like `r/r`, `r/d`, `v/v`:
- First character: file type (r=regular, d=directory, v=virtual, c=character device, b=block device)
- Second character: directory status (r=regular, d=directory)
- Example: `r/r` = regular file in a regular directory

**Q5: fls shows different results than `ls` on a mounted system.**

A: This is expected. `ls` shows only allocated files via the kernel. fls reads raw disk structures and shows deleted files, hidden entries, and metadata that `ls` cannot access.

**Q6: Can fls recover file contents?**

A: No. fls only lists file names and metadata. To recover file contents, use `icat` with the inode number from fls output.

```bash
# Step 1: Find the file with fls
fls -r -d usb_image.dd | grep "secret.txt"
# Output: r/r * 58493: secret.txt

# Step 2: Recover with icat
icat usb_image.dd 58493 > recovered_secret.txt
```

**Q7: What does the `*` mean in fls output?**

A: The `*` indicates the entry is unallocated (deleted). Without `*`, the entry is allocated (active).

---

## Resources

### Official Documentation

- **TSK Manual**: https://sleuthkit.org/sleuthkit/man/
- **fls Man Page**: `man fls`
- **GitHub**: https://github.com/sleuthkit/sleuthkit
- **Kali Tools**: https://www.kali.org/tools/sleuthkit/

### Books and References

- *File System Forensic Analysis* by Brian Carrier — Comprehensive TSK reference
- *The Sleuth Kit and Autopsy* by Brian Carrier — Practical TSK guide
- *Linux Forensics* by Phillip Andrews — Linux-specific forensics

### Related TSK Tools

| Tool | Purpose | Complements fls |
|------|---------|-----------------|
| `ffind` | Find file name by inode | Resolves paths for inodes found in fls |
| `icat` | Extract file by inode | Recovers content of files listed by fls |
| `ils` | List inodes | Provides inode details for fls entries |
| `istat` | Inode statistics | Shows metadata for specific inodes |
| `fsstat` | Filesystem statistics | Provides context for fls analysis |
| `tsk_recover` | Bulk file recovery | Alternative to individual icat calls |

### Online Communities

- **Sleuth Kit Forums**: https://digitalforensicsforum.com/
- **DFIR Reddit**: https://reddit.com/r/DigitalForensics
- **SANS DFIR**: https://www.sans.org/dfir/

---

**TSK Install**: `sudo apt install sleuthkit` | **Man Page**: `man fls`
