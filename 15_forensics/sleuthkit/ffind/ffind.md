---
tool_name: ffind
display_name: ffind
mitre_tactic: "analysis"
mitre_tactic_id: "TA0007"
install_command: "sudo apt install sleuthkit"
github_repo: "https://github.com/sleuthkit/sleuthkit"
kali_tools_page: "https://www.kali.org/tools/sleuthkit/"
difficulty_level: "beginner"
requires_root: false
gui_available: false
tags: ["forensics", "filesystem", "sleuthkit", "tsk", "inode", "file-recovery", "analysis"]
related_tools: ["fls", "icat", "fsstat", "ils", "istat", "mmls", "mmstat", "blkls"]
---

# ffind — Forensic File Name Lookup by Inode

## Chapter 1: Introduction & Overview

### What ffind Does

ffind is a command-line utility from The Sleuth Kit (TSK) that recovers the file name associated with a given inode number on a filesystem image. In traditional filesystems, every file and directory is tracked internally by an inode — a numeric identifier that stores metadata like permissions, ownership, timestamps, and pointers to data blocks. The human-readable file name is a separate mapping stored in directory entries. When a file is deleted, the directory entry is typically removed but the inode may persist on disk until its space is reclaimed.

ffind bridges that gap. Given an inode number and a filesystem image, it searches the filesystem's directory structures to find every directory entry that points to that inode. This is essential in forensic investigations where you have an inode number from tools like `ils` (inode list) but need to recover the original path and file name.

### Why Inode-Based Investigation Matters

Standard file browsing only shows active, allocated files. Forensic analysts need to work at a lower level:

- **Deleted files** lose their directory entries but retain inodes until overwritten. ffind can sometimes still locate residual references.
- **Deleted directory entries** may still contain inode references in unallocated space.
- **Cross-referencing** between `ils` output and actual file paths requires inode-to-name resolution.
- **Malware analysis** often involves files hidden with alternate data streams or unusual directory structures where the inode path is the only reliable anchor.

### Position in The Sleuth Kit

The Sleuth Kit provides a layered toolkit for filesystem forensics:

| Layer | Tools | Purpose |
|-------|-------|---------|
| Volume System | `mmls`, `mmstat`, `mmcat` | Partition table analysis |
| Filesystem Metadata | `dls`, `dcat`, `dbstat` | Raw filesystem block analysis |
| Filesystem Analysis | `fls`, `ffind`, `icat`, `ils`, `istat`, `fsstat` | File and inode operations |
| File Content | `icat`, `tsk_recover` | Data extraction |

ffind sits in the filesystem analysis layer. It depends on `fls` having built an index and on `fsstat` having identified the filesystem type, but it operates independently on a per-inode basis.

### Common Use Cases

1. **Post-incident response**: An attacker deleted logs. `ils` found inode 58493 still active. ffind determines it was `/var/log/auth.log`.
2. **Child exploitation investigations**: Investigators find an inode from a hash database match. ffind recovers the original path.
3. **Data recovery**: A user accidentally deleted a project directory. ffind locates the inode to reconstruct the path.
4. **Timeline analysis**: Correlating inode timestamps with directory entry timestamps reveals when files were created versus when they were last referenced.

### Key Concepts

Before using ffind, understand these foundational concepts:

**Inode**: A data structure on Unix/Linux filesystems (ext2/3/4, UFS, XFS) that stores all metadata about a file except its name. Each inode has a unique number within its filesystem.

**Directory Entry (dentry)**: A mapping of a human-readable name to an inode number. Directories are themselves files that contain arrays of these entries.

**Allocation Status**: An inode can be allocated (in use by an active file) or unallocated (deleted or free). ffind works on both states but results differ.

**Hard Links**: Multiple directory entries can point to the same inode. ffind returns all such entries, which is why one inode may map to multiple paths.

---

## Chapter 2: Installation & Setup

### Installing The Sleuth Kit on Kali Linux

ffind is bundled with The Sleuth Kit package. On Kali Linux:

```bash
# Update package lists
sudo apt update

# Install the full Sleuth Kit
sudo apt install sleuthkit

# Verify installation
ffind --version
```

The package installs all TSK tools into `/usr/bin/`, so ffind is immediately available in your PATH.

### Verifying the Installation

```bash
# Check that ffind is available
which ffind

# Check the version
ffind -V

# List all TSK tools installed
dpkg -L sleuthkit | grep bin/
```

### Building from Source (Advanced)

If you need the latest development version or特定 features:

```bash
# Install build dependencies
sudo apt install build-essential git autoconf automake libtool

# Clone the repository
git clone https://github.com/sleuthkit/sleuthkit.git
cd sleuthkit

# Build
./bootstrap
./configure
make -j$(nproc)

# Install system-wide
sudo make install

# Update library paths
sudo ldconfig
```

### Setting Up a Working Environment

For forensic work, always work on copies of evidence, never originals:

```bash
# Create a working directory
mkdir -p ~/forensics/cases/case_001
cd ~/forensics/cases/case_001

# Create a verified copy of the evidence image
# (Use dc3dd or dcfldd for verified imaging)
dc3dd if=/dev/sdb hash=sha256 log=evidence.log of=usb_image.dd

# Create a working copy (preserve the original)
cp usb_image.dd usb_image_work.dd

# Document the chain of custody
echo "Original evidence: /dev/sdb" > chain_of_custody.txt
echo "Image created: $(date)" >> chain_of_custody.txt
echo "Hash: $(sha256sum usb_image.dd)" >> chain_of_custody.txt
```

### Understanding Filesystem Support

ffind supports the following filesystem types:

| Filesystem | Support Level | Notes |
|------------|---------------|-------|
| ext2 | Full | Primary support |
| ext3 | Full | Extends ext2 with journaling |
| ext4 | Full | Most common Linux filesystem |
| FAT12 | Full | Floppy disks, small media |
| FAT16 | Full | Older removable media |
| FAT32 | Full | USB drives, SD cards |
| NTFS | Full | Windows primary filesystem |
| HFS+ | Full | macOS filesystem |
| UFS | Full | BSD systems |
| YAFFS2 | Partial | Android NAND flash |

---

## Chapter 3: Basic Usage

### Command Syntax

```
ffind [options] <image> <inode>
```

**Positional Arguments:**

| Argument | Description |
|----------|-------------|
| `<image>` | Path to the filesystem image or device node |
| `<inode>` | The inode number to search for (decimal) |

### Complete Flag Reference

| Flag | Long Form | Description |
|------|-----------|-------------|
| `-f` | `--fstype` | Specify the filesystem type (e.g., ext4, ntfs, fat) |
| `-i` | `--inode` | Display inode number along with file name (default behavior, but explicit) |
| `-e` | `--embed` | Search embedded entries in the directory (for NTFS alternate data streams) |
| `-s` | `--slack` | Search slack space for directory entries |
| `-u` | `--unalloc` | Include unallocated directory entries in search |
| `-a` | `--all` | Search all directory entries including deleted |
| `-o` | `--offset` | Byte offset to start the search |
| `-b` | `--block-size` | Block size for the filesystem (auto-detected in most cases) |
| `-d` | `--delent` | Show deleted directory entries |
| `-V` | `--version` | Print version information and exit |
| `-h` | `--help` | Display help message and exit |
| `-v` | `--verbose` | Enable verbose output |
| `-z` | `--zone` | Specify the timezone for timestamp display |
| `-t` | `--text` | Display output as text (default) |
| `-B` | `--body-file` | Output in mactime body file format |
| `-C` | `--csv` | Output in CSV format |
| `-l` | `--list` | List all inodes (used with -a for full enumeration) |

### Example 1: Basic Inode Lookup

```bash
# Find what file corresponds to inode 12 on an ext4 image
ffind usb_image.dd 12
```

**Output:**
```
/resume.doc
```

This tells you inode 12 is the file `resume.doc` located at the root of the filesystem.

### Example 2: Verbose Output

```bash
# Use verbose mode for detailed information
ffind -v usb_image.dd 58493
```

**Output:**
```
 Searching for inode 58493...
 Filesystem type: ext4
 Block size: 4096
 Found match in directory block 245768
  Entry: resume.doc -> inode 58493
  Path: /Documents/resume.doc
```

### Example 3: Specifying Filesystem Type

```bash
# When auto-detection fails, specify the filesystem type
ffind -f ext4 usb_image.dd 58493
```

### Example 4: Including Deleted Entries

```bash
# Search including deleted directory entries
ffind -d usb_image.dd 58493
```

**Output:**
```
/Documents/resume.doc
/deleted_files/resume.doc.bak
```

The second entry was from a deleted directory.

### Example 5: Working with NTFS

```bash
# NTFS image — find the MFT record
ffind -f ntfs windows_image.dd 67
```

### Example 6: Multiple Inodes from a File

```bash
# Find all inode references from an ils output file
while read inode; do
    echo "=== Inode $inode ==="
    ffind image.dd "$inode"
done < inodes.txt
```

### Example 7: Hard Link Detection

```bash
# An inode with multiple links (hard links)
ffind -i usb_image.dd 12
```

**Output:**
```
/resume.doc
/archived/resume.doc
/backup/resume.doc
```

Three directory entries point to the same inode — hard links.

### Exit Codes

| Code | Meaning |
|------|---------|
| 0 | Success — inode found |
| 1 | Error — inode not found or invalid |
| 2 | Error — invalid image or filesystem |

---

## Chapter 4: Intermediate Usage

### Bash Scripting with ffind

#### Script 1: Batch Inode Resolution

Resolve a list of inodes from an `ils` output file:

```bash
#!/bin/bash
# resolve_inodes.sh — Resolve inodes from ils output to file names

IMAGE="$1"
ILS_FILE="$2"

if [ -z "$IMAGE" ] || [ -z "$ILS_FILE" ]; then
    echo "Usage: $0 <image> <ils_output_file>"
    exit 1
fi

echo "Inode,File Name,Path"
echo "----,---------,----"

while IFS=, read -r meta_addr seq flags size uid gid mode crtime szptime atime mtime ctime; do
    # Skip header line
    [[ "$meta_addr" == "meta_addr" ]] && continue

    # Skip empty lines
    [[ -z "$meta_addr" ]] && continue

    # Extract inode number
    inode=$(echo "$meta_addr" | tr -d ' ')

    # Look up file name
    result=$(ffind "$IMAGE" "$inode" 2>/dev/null)

    if [ -n "$result" ]; then
        echo "$inode,${result##*/},$result"
    else
        echo "$inode,UNKNOWN,NOT_FOUND"
    fi
done < "$ILS_FILE"
```

**Usage:**
```bash
ils usb_image.dd > inodes.txt
chmod +x resolve_inodes.sh
./resolve_inodes.sh usb_image.dd inodes.txt > resolved_inodes.csv
```

#### Script 2: Recover Deleted Files by Inode

```bash
#!/bin/bash
# recover_deleted.sh — Recover a deleted file using ffind + icat

IMAGE="$1"
INODE="$2"
OUTPUT_DIR="$3"

if [ -z "$IMAGE" ] || [ -z "$INODE" ] || [ -z "$OUTPUT_DIR" ]; then
    echo "Usage: $0 <image> <inode> <output_dir>"
    exit 1
fi

mkdir -p "$OUTPUT_DIR"

# First, find the original path
ORIGINAL_PATH=$(ffind -d "$IMAGE" "$INODE" 2>/dev/null)
echo "Original path: $ORIGINAL_PATH"

# Extract just the filename
FILENAME=$(basename "$ORIGINAL_PATH" 2>/dev/null)
[ -z "$FILENAME" ] && FILENAME="inode_${INODE}_unknown"

# Recover the file content using icat
icat "$IMAGE" "$INODE" > "$OUTPUT_DIR/$FILENAME"

if [ $? -eq 0 ]; then
    echo "Recovered to: $OUTPUT_DIR/$FILENAME"
    echo "File size: $(wc -c < "$OUTPUT_DIR/$FILENAME") bytes"
    echo "MD5: $(md5sum "$OUTPUT_DIR/$FILENAME" | cut -d' ' -f1)"
else
    echo "Error: Failed to recover inode $INODE"
    rm -f "$OUTPUT_DIR/$FILENAME"
    exit 1
fi
```

**Usage:**
```bash
chmod +x recover_deleted.sh
./recover_deleted.sh usb_image.dd 58493 recovered_files/
```

#### Script 3: Comprehensive Forensic Report

```bash
#!/bin/bash
# forensic_report.sh — Generate a report linking inodes to paths

IMAGE="$1"
REPORT="forensic_report_$(date +%Y%m%d_%H%M%S).txt"

if [ -z "$IMAGE" ]; then
    echo "Usage: $0 <filesystem_image>"
    exit 1
fi

echo "=== Forensic Report ===" > "$REPORT"
echo "Image: $IMAGE" >> "$REPORT"
echo "Generated: $(date)" >> "$REPORT"
echo "" >> "$REPORT"

# Get filesystem info
echo "--- Filesystem Information ---" >> "$REPORT"
fsstat "$IMAGE" >> "$REPORT" 2>/dev/null
echo "" >> "$REPORT"

# List all files with inodes
echo "--- Active Files ---" >> "$REPORT"
fls -m "/" "$IMAGE" >> "$REPORT" 2>/dev/null
echo "" >> "$REPORT"

# Search for unallocated inodes
echo "--- Potentially Deleted Files ---" >> "$REPORT"
ils -e "$IMAGE" 2>/dev/null | while read line; do
    inode=$(echo "$line" | awk '{print $1}' | grep -E '^[0-9]+$')
    if [ -n "$inode" ]; then
        fname=$(ffind -d "$IMAGE" "$inode" 2>/dev/null)
        if [ -n "$fname" ]; then
            echo "  Inode $inode -> $fname (deleted)" >> "$REPORT"
        else
            echo "  Inode $inode -> [path not recoverable]" >> "$REPORT"
        fi
    fi
done

echo "" >> "$REPORT"
echo "Report complete: $REPORT"
```

### Python Scripting with ffind

#### Python 1: Inode Resolution Module

```python
#!/usr/bin/env python3
"""
ffind_wrapper.py — Python wrapper for ffind forensic file name resolution.
"""

import subprocess
import re
from dataclasses import dataclass
from typing import Optional, List
from pathlib import Path


@dataclass
class FfindResult:
    """Represents a single ffind lookup result."""
    inode: int
    path: Optional[str]
    original_name: Optional[str]
    parent_directory: Optional[str]
    found: bool


class FfindWrapper:
    """Wrapper class for the ffind forensic tool."""

    def __init__(self, image_path: str, filesystem_type: Optional[str] = None):
        self.image_path = Path(image_path)
        self.filesystem_type = filesystem_type
        self._validate_image()

    def _validate_image(self):
        if not self.image_path.exists():
            raise FileNotFoundError(f"Image not found: {self.image_path}")

    def find_by_inode(self, inode: int, include_deleted: bool = False) -> FfindResult:
        """
        Find the file path for a given inode number.

        Args:
            inode: The inode number to search for
            include_deleted: Whether to include deleted directory entries

        Returns:
            FfindResult with path information
        """
        cmd = ["ffind"]

        if self.filesystem_type:
            cmd.extend(["-f", self.filesystem_type])

        if include_deleted:
            cmd.append("-d")

        cmd.extend([str(self.image_path), str(inode)])

        try:
            result = subprocess.run(
                cmd,
                capture_output=True,
                text=True,
                timeout=30
            )

            if result.returncode == 0 and result.stdout.strip():
                path = result.stdout.strip()
                parts = path.rsplit("/", 1)
                parent = parts[0] if len(parts) > 1 else "/"
                name = parts[-1] if len(parts) > 1 else path

                return FfindResult(
                    inode=inode,
                    path=path,
                    original_name=name,
                    parent_directory=parent,
                    found=True
                )
            else:
                return FfindResult(
                    inode=inode,
                    path=None,
                    original_name=None,
                    parent_directory=None,
                    found=False
                )

        except subprocess.TimeoutExpired:
            raise TimeoutError(f"ffind timed out for inode {inode}")
        except subprocess.CalledProcessError as e:
            raise RuntimeError(f"ffind failed: {e.stderr}")

    def find_multiple(self, inodes: List[int], include_deleted: bool = False) -> List[FfindResult]:
        """
        Resolve multiple inodes in batch.

        Args:
            inodes: List of inode numbers to resolve
            include_deleted: Whether to include deleted directory entries

        Returns:
            List of FfindResult objects
        """
        results = []
        for inode in inodes:
            result = self.find_by_inode(inode, include_deleted)
            results.append(result)
        return results

    def find_with_recovery(self, inode: int, output_dir: str) -> dict:
        """
        Find the file and recover its contents using icat.

        Args:
            inode: The inode number to recover
            output_dir: Directory to write recovered file

        Returns:
            Dictionary with path and recovery status
        """
        # First, find the file path
        ffind_result = self.find_by_inode(inode, include_deleted=True)

        output_path = Path(output_dir)
        output_path.mkdir(parents=True, exist_ok=True)

        if ffind_result.found:
            filename = ffind_result.original_name or f"inode_{inode}"
            recovered_path = output_path / filename
        else:
            recovered_path = output_path / f"inode_{inode}_unknown"

        # Recover using icat
        cmd = ["icat", str(self.image_path), str(inode)]
        try:
            with open(recovered_path, "wb") as f:
                result = subprocess.run(cmd, capture_output=True, timeout=60)
                f.write(result.stdout)

            return {
                "inode": inode,
                "original_path": ffind_result.path,
                "recovered_path": str(recovered_path),
                "size": recovered_path.stat().st_size,
                "success": True
            }

        except Exception as e:
            return {
                "inode": inode,
                "original_path": ffind_result.path,
                "recovered_path": None,
                "error": str(e),
                "success": False
            }


def main():
    import argparse

    parser = argparse.ArgumentParser(description="Forensic inode resolution wrapper")
    parser.add_argument("image", help="Path to filesystem image")
    parser.add_argument("inodes", nargs="+", type=int, help="Inode numbers to resolve")
    parser.add_argument("-f", "--fstype", help="Filesystem type")
    parser.add_argument("-d", "--deleted", action="store_true",
                        help="Include deleted entries")
    parser.add_argument("-r", "--recover", help="Recover files to this directory")
    parser.add_argument("--csv", action="store_true", help="Output as CSV")

    args = parser.parse_args()

    wrapper = FfindWrapper(args.image, args.fstype)

    if args.csv:
        print("inode,path,found")

    for inode in args.inodes:
        if args.recover:
            result = wrapper.find_with_recovery(inode, args.recover)
            if args.csv:
                print(f"{inode},{result.get('recovered_path', 'N/A')},{result['success']}")
            else:
                if result["success"]:
                    print(f"[OK] Inode {inode}: {result['original_path']} -> {result['recovered_path']}")
                else:
                    print(f"[FAIL] Inode {inode}: {result.get('error', 'Unknown error')}")
        else:
            result = wrapper.find_by_inode(inode, args.deleted)
            if args.csv:
                print(f"{inode},{result.path or 'N/A'},{result.found}")
            else:
                if result.found:
                    print(f"Inode {inode}: {result.path}")
                else:
                    print(f"Inode {inode}: NOT FOUND")


if __name__ == "__main__":
    main()
```

**Usage:**
```bash
# Basic resolution
python3 ffind_wrapper.py usb_image.dd 58493 12 100

# CSV output
python3 ffind_wrapper.py --csv usb_image.dd 58493 12

# Recovery mode
python3 ffind_wrapper.py -r recovered/ usb_image.dd 58493 12 100

# With filesystem type
python3 ffind_wrapper.py -f ext4 usb_image.dd 58493
```

---

## Chapter 5: Advanced Usage

### Cross-Referencing with Other TSK Tools

The real power of ffind comes from combining it with other Sleuth Kit tools.

#### Workflow: From Inode to Full Recovery

```bash
# Step 1: Get filesystem information
fsstat usb_image.dd

# Step 2: List all inodes (including deleted)
ils -e usb_image.dd > all_inodes.csv

# Step 3: Find suspicious inodes (e.g., recently deleted large files)
awk -F, '$12 > 1000000 && $3 == "a"' all_inodes.csv | head -20

# Step 4: Resolve inode paths
while IFS=, read -r inode rest; do
    [ "$inode" = "meta_addr" ] && continue
    path=$(ffind -d usb_image.dd "$inode" 2>/dev/null)
    [ -n "$path" ] && echo "Inode $inode: $path"
done < suspicious_inodes.csv

# Step 5: Recover specific files
icat usb_image.dd 58493 > recovered_document.doc
```

### Advanced Flag Combinations

```bash
# Search slack space for directory entries (recovering overwritten directories)
ffind -s usb_image.dd 58493

# Combine verbose with deleted entries for maximum information
ffind -v -d usb_image.dd 58493

# Force specific filesystem type when auto-detection fails
ffind -f ext4 -v usb_image.dd 58493

# Search embedded NTFS entries (alternate data streams)
ffind -e -f ntfs windows_image.dd 67
```

### Processing Large Case Loads

When dealing with hundreds or thousands of inodes:

```bash
#!/bin/bash
# batch_resolve.sh — Parallel inode resolution for large datasets

IMAGE="$1"
INODE_LIST="$2"
MAX_JOBS=8

export IMAGE

resolve_one() {
    local inode=$1
    local path
    path=$(ffind -d "$IMAGE" "$inode" 2>/dev/null)
    if [ -n "$path" ]; then
        echo "$inode,$path"
    else
        echo "$inode,NOT_FOUND"
    fi
}

export -f resolve_one

# Process in parallel
cat "$INODE_LIST" | xargs -P "$MAX_JOBS" -I{} bash -c 'resolve_one {}'
```

### Automount and Live System Analysis

```bash
# On a live Linux system, resolve inodes from the running filesystem
# WARNING: Only do this if the system is compromised and you have authorization

# Find an inode number from /proc or other sources
INODE=$(stat -c %i /etc/passwd)
echo "passwd inode: $INODE"

# Resolve it using ffind on the live mount
ffind /dev/sda2 "$INODE"

# For live systems, you can also use the device directly
ffind /dev/sda2 2
```

### Integration with Autopsy/Sleuth Kit GUI

ffind results can be imported into Autopsy or other forensic GUIs:

```bash
# Export ffind results in body file format for timeline analysis
ffind -B usb_image.dd 58493 >> body_file.csv

# Merge with mactime for timeline visualization
mactime -b body_file.csv -d output_dir/
```

### Error Handling and Edge Cases

```bash
# Handle case where image is not a valid filesystem
ffind corrupted_image.dd 12 2>&1
# Expected: error message about invalid filesystem

# Handle case where inode is out of range
ffind usb_image.dd 99999999 2>&1
# Expected: "inode not found" or similar

# Handle case where image requires elevated permissions
# (rare, but some encrypted or protected images may)
sudo ffind protected_image.dd 12
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: Ransomware Incident Response

**Background**: A company workstation was encrypted by ransomware. The IT team suspects the malware was downloaded from an email attachment.

**Investigation Steps:**

```bash
# 1. Image the affected disk
dc3dd if=/dev/sda hash=sha256 log=workstation_evidence.log of=workstation.dd

# 2. Get filesystem information
fsstat -f ext4 workstation.dd

# 3. List deleted files (ransomware often deletes originals)
ils -e workstation.dd | grep -E "^a," > allocated_inodes.txt
ils -e workstation.dd | grep -E "^u," > unallocated_inodes.txt

# 4. Search for suspicious recently-deleted executables
fls -r -d -m "/" workstation.dd | grep -iE "\.(exe|dll|scr|bat|ps1|vbs)"

# 5. For each suspicious inode, resolve the path
for inode in $(awk -F, 'NR>1{print $1}' suspicious_inodes.txt); do
    result=$(ffind -d workstation.dd "$inode")
    echo "Inode $inode: $result"
done

# 6. Recover the malware sample
icat workstation.dd $SUSPICIOUS_INODE > malware_sample.exe

# 7. Hash the recovered file for IOC database
sha256sum malware_sample.exe
md5sum malware_sample.exe
```

### Scenario 2: Child Exploitation Material Investigation

**Background**: A tip from NCMEC identifies a specific hash. investigators need to find the original file path on the suspect's computer.

**Investigation Steps:**

```bash
# 1. Forensically image the suspect drive
# (Using write blocker and verified imaging tool)

# 2. Run hash matching against known CEI databases
# (Using tools like hashdeep against NCMEC hash sets)

# 3. When a hash match is found, note the inode from the TSK database
# Example: hash match found at inode 847291

# 4. Resolve the full path
ffind -d suspect_drive.dd 847291
# Output: /home/user/Downloads/.hidden/archive.dat

# 5. Verify the match by recovering and hashing
icat suspect_drive.dd 847291 | sha256sum
# Compare with known hash

# 6. Document the chain of custody for this specific file
echo "File: /home/user/Downloads/.hidden/archive.dat" > evidence_doc.txt
echo "Inode: 847291" >> evidence_doc.txt
echo "Hash match: CONFIRMED" >> evidence_doc.txt
echo "Recovery timestamp: $(date -u)" >> evidence_doc.txt
```

### Scenario 3: Corporate Data Exfiltration

**Background**: An employee is suspected of copying proprietary source code before leaving the company. USB device records show a device was connected.

**Investigation Steps:**

```bash
# 1. Image the USB device that was connected
dc3dd if=/dev/sdb hash=sha256 log=usb_evidence.log of=usb_device.dd

# 2. Identify filesystem type
fsstat usb_device.dd

# 3. List all files including deleted
fls -r -d -m "/" usb_device.dd > usb_file_list.txt

# 4. Search for source code files that were deleted
grep -iE "\.(py|java|cpp|h|c|rb|go|rs)$" usb_file_list.txt

# 5. Resolve each suspicious inode
while read line; do
    inode=$(echo "$line" | grep -oP 'r/\*\s+\K[0-9]+')
    if [ -n "$inode" ]; then
        path=$(ffind -d usb_device.dd "$inode")
        echo "$path (inode $inode)"
    fi
done < suspicious_files.txt

# 6. Recover the deleted source code files
mkdir -p recovered_source/
for inode in $(cat suspicious_inodes.txt); do
    ffind -d usb_device.dd "$inode" | while read path; do
        filename=$(basename "$path")
        icat usb_device.dd "$inode" > "recovered_source/$filename"
    done
done

# 7. Generate a summary report
echo "=== Data Exfiltration Evidence ===" > exfiltration_report.txt
echo "USB Device: /dev/sdb" >> exfiltration_report.txt
echo "Files found: $(ls recovered_source/ | wc -l)" >> exfiltration_report.txt
echo "Total size: $(du -sh recovered_source/ | cut -f1)" >> exfiltration_report.txt
ls -la recovered_source/ >> exfiltration_report.txt
```

---

## Chapter 7: Detection & Defense

### How Attackers Avoid ffind Detection

Understanding how ffind works helps defenders understand attack techniques:

1. **Inode Overwriting**: Writing new data over the same blocks reuses the inode. ffind returns the new file, not the original.
2. **Directory Entry Corruption**: Manually corrupting directory entries can make ffind unable to resolve paths.
3. **Full Disk Encryption**: If the entire partition is encrypted, ffind cannot parse the filesystem structure.
4. **Filesystem Wiping**: Tools like `shred` or `dd` overwrite the entire filesystem, destroying all inodes.
5. **Journal Manipulation**: On ext3/4, the journal can be cleared, removing filesystem transaction history.

### Defensive Measures

```bash
# 1. Enable filesystem journaling (ext3/4) for better recovery
tune2fs -j /dev/sda2

# 2. Use file integrity monitoring (AIDE, Tripwire)
aide --init
sudo mv /var/lib/aide/aide.db.new /var/lib/aide/aide.db

# 3. Enable audit logging for file deletions
auditctl -w /home -p w -k file_deletions

# 4. Use append-only logs (stored on separate media)
# 5. Implement USB device whitelisting via udev rules
# 6. Regular forensic imaging of critical systems
```

### Forensic Anti-Tampering

When you suspect ffind results may have been tampered with:

```bash
# Verify filesystem integrity before analysis
dls usb_image.dd | sha256sum

# Compare with original image hash
# If hashes differ, the image has been modified

# Use multiple tools to cross-validate
fls usb_image.dd | head
ffind usb_image.dd 12
# Results should be consistent
```

---

## Chapter 8: Troubleshooting

### FAQ

**Q1: ffind returns nothing for a valid inode number.**

A: Several possible causes:
- The filesystem type may not be auto-detected correctly. Use `-f` to specify it.
- The inode may be truly unallocated with no residual directory entries.
- The image may be corrupted. Verify with `fsstat`.
- Try adding `-d` to include deleted directory entries.

```bash
# Diagnostic steps
fsstat -f ext4 usb_image.dd
ffind -v -f ext4 usb_image.dd 58493
```

**Q2: ffind says "invalid filesystem" or fails to open the image.**

A: The image may not be a raw filesystem image. Check:
- Is it a full disk image (needs offset to filesystem)?
- Is it a partition image (should work directly)?
- Use `mmls` to find the partition offset if needed.

```bash
# Find partition offsets
mmls disk_image.dd

# Use offset if filesystem is inside a partition
ffind -o 2048 disk_image.dd 58493
```

**Q3: ffind is very slow on large images.**

A: ffind scans directory blocks sequentially. For large images:
- Ensure you're using a local disk, not network storage.
- The filesystem type specification can help skip unnecessary checks.
- Consider using TSK's `tsk_recover` first to extract files, then look up inodes.

```bash
# Speed up by specifying filesystem type
ffind -f ext4 usb_image.dd 58493

# Check image size and verify it's local
ls -lh usb_image.dd
df -h .
```

**Q4: ffind shows different paths for the same inode.**

A: This is normal and indicates hard links. Multiple directory entries point to the same inode. All results are valid.

```bash
# Example output showing hard links
ffind usb_image.dd 12
# /resume.doc
# /archived/resume.doc
# /backup/resume.doc
```

**Q5: How do I know if ffind found the correct file?**

A: Cross-reference with other tools:
- Use `istat` to verify inode metadata matches the expected file.
- Use `icat` to extract the file and verify its contents match.
- Check timestamps in the inode versus the directory entry.

```bash
# Verify inode metadata
istat usb_image.dd 58493

# Extract and verify contents
icat usb_image.dd 58493 | file -
icat usb_image.dd 58493 | sha256sum
```

**Q6: Can ffind work on encrypted filesystems?**

A: No. ffind requires direct access to the filesystem structure. Encrypted containers (LUKS, BitLocker, FileVault) must be decrypted first. Mount the decrypted volume and image it again, or use tools that can handle the encryption layer.

**Q7: What's the difference between ffind and fls?**

A: `fls` lists all files and directories in a filesystem image (similar to `ls`). `ffind` takes a specific inode number and finds which file name(s) reference it. Use `fls` for enumeration, `ffind` for targeted lookup.

---

## Resources

### Official Documentation

- **TSK Manual**: https://sleuthkit.org/sleuthkit/man/
- **ffind Man Page**: `man ffind` (after installation)
- **GitHub Repository**: https://github.com/sleuthkit/sleuthkit
- **Kali Tools Page**: https://www.kali.org/tools/sleuthkit/

### Books and References

- *File System Forensic Analysis* by Brian Carrier — The definitive reference for TSK and filesystem forensics.
- *The Sleuth Kit and Autopsy* by Brian Carrier — Practical guide to TSK tools.
- *Linux Filesystem Forensics* by Kevin Karen — Deep dive into ext filesystem forensics.

### Related TSK Tools

| Tool | Purpose | Relationship to ffind |
|------|---------|----------------------|
| `fls` | List files in filesystem image | Provides inode numbers for ffind lookup |
| `icat` | Extract file by inode | Recovers file content after ffind locates it |
| `ils` | List inode information | Identifies inodes to search for with ffind |
| `istat` | Display inode details | Shows metadata for the inode ffind located |
| `fsstat` | Filesystem statistics | Provides context for ffind analysis |
| `tsk_recover` | Recover files from image | Bulk alternative to individual ffind+icat |

### Online Communities

- **Sleuth Kit Forums**: https://digitalforensicsforum.com/
- **DFIR Reddit**: https://reddit.com/r/DigitalForensics
- **SANS DFIR**: https://www.sans.org/dfir/

### Quick Reference Card Summary

ffind is a focused tool with a single purpose: inode-to-name resolution. Master it by understanding inodes, filesystem structures, and how it integrates with the broader TSK toolkit. The key flags to remember are `-d` (deleted entries), `-f` (filesystem type), and `-v` (verbose output). Always work on copies of evidence and document your methodology.
