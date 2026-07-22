---
tool_name: sorter
display_name: "TSK sorter"
mitre_tactic: "analysis"
mitre_tactic_id: "TA0007"
install_command: "sudo apt install sleuthkit"
github_repo: "https://github.com/sleuthkit/sleuthkit"
kali_tools_page: "https://www.kali.org/tools/sleuthkit/"
difficulty_level: "intermediate"
requires_root: true
gui_available: false
tags: ["forensics", "sleuthkit", "disk-image", "file-sorting", "hash-analysis", "categorization", "triage"]
related_tools: ["sigfind", "fls", "icat", "tsk_recover", "md5deep"]
---

# sorter — Automated File Sorting and Categorization for Disk Images

## Chapter 1: Introduction & Overview

### What Is sorter?

sorter is a command-line forensic utility from The Sleuth Kit (TSK) that automatically categorizes files extracted from a disk image into directories based on file type, hash value, or other classification criteria. Developed by Brian Carrier as part of the TSK suite, sorter reads a filesystem image, extracts each file, computes its hash (MD5 by default, with SHA-1 and SHA-256 options), determines its file type using magic-number detection, and copies the file into a categorized output directory structure.

### Why sorter Matters in Digital Forensics

When investigators process a disk image, they often face thousands or millions of files. Manually reviewing each file is impractical. sorter automates the initial triage by:

- **Organizing by file type**: Images, documents, executables, archives, and other categories are placed in separate directories, allowing investigators to focus on relevant categories.
- **Hash-based deduplication**: Files with identical hashes are grouped together, eliminating redundant analysis of duplicate files.
- **Known-file filtering**: Using reference hash databases (like NSRL or custom whitelists), sorter can separate known-good files from unknown files that warrant closer examination.
- **Triage acceleration**: In incident response, quickly identifying unusual file types or executables on a compromised system is critical.

### How sorter Works — Architecture

sorter operates in several stages:

1. **Filesystem traversal**: Uses TSK's library (libtsk) to walk the filesystem structure, similar to how `fls` lists files.
2. **File extraction**: For each file entry, sorter reads the file content from the image using its inode number.
3. **Hash computation**: Computes a cryptographic hash (MD5, SHA-1, or SHA-256) of the file content.
4. **File type detection**: Uses the `file` command's magic-number database to identify the file type.
5. **Categorization and copy**: Copies the file into the appropriate output subdirectory based on its type and/or hash.

### sorter vs. Similar Tools

| Feature | sorter | fls + icat | foremost/scalpel | md5deep |
|---------|--------|------------|------------------|---------|
| Works on disk images | Yes | Yes | Yes | No (files only) |
| Auto-categorizes by type | Yes | No | Yes (carving) | No |
| Hash-based grouping | Yes | No | No | Yes |
| Reads filesystem metadata | Yes | Yes | No (raw carving) | No |
| Extracts deleted files | No (live files only) | Depends | Yes (carving) | No |
| Requires root | Often yes | Yes | Yes | No |

### Use Cases

| Scenario | How sorter Helps |
|----------|-----------------|
| Incident response triage | Quickly isolate executables, scripts, and unusual file types |
| Child exploitation material (CSAM) investigations | Hash-matching against known databases to identify confirmed material |
| Intellectual property cases | Group files by type to find documents, spreadsheets, and source code |
| Malware investigations | Identify executable files and archives that may contain payloads |
| Corporate espionage | Separate email files, documents, and database files for targeted review |
| General evidence processing | Create an organized file repository from a raw disk image |

---

## Chapter 2: Installation & Setup

### Installing sorter

sorter is part of The Sleuth Kit and is installed alongside all other TSK tools:

```bash
sudo apt update
sudo apt install sleuthkit
```

Verify installation:

```bash
which sorter
sorter --help
```

### Building from Source

For the latest development version:

```bash
git clone https://github.com/sleuthkit/sleuthkit.git
cd sleuthkit
./bootstrap
./configure --prefix=/usr/local
make
sudo make install
sudo ldconfig
```

### Dependencies

sorter relies on:

- **libtsk** (The Sleuth Kit library) — for filesystem access
- **file** command (libmagic) — for file type detection
- **md5sum / sha1sum / sha256sum** — for hash computation (or TSK's built-in hashing)

On Debian/Kali, these are satisfied by installing the `sleuthkit` and `file` packages:

```bash
sudo apt install sleuthkit file
```

### Required Privileges

sorter typically requires root privileges because:

- It reads raw disk images that may have restricted file permissions.
- It may need to access device files directly.
- The output directory often requires elevated permissions for bulk file creation.

Always run sorter with `sudo` when processing forensic images:

```bash
sudo sorter [options] <image>
```

### Preparing the Output Directory

sorter creates its output directory structure automatically, but it's good practice to create a clean parent directory:

```bash
mkdir -p /evidence/sorted_output
# sorter will create subdirectories within this path
```

---

## Chapter 3: Basic Usage

### Command Syntax

```
sorter [options] <image>
```

Where `<image>` is the path to a disk image or filesystem image.

### Complete Flag Reference

| Flag | Long Form | Description | Example |
|------|-----------|-------------|---------|
| `-a` | `--all` | Output all files, including those with no recognized type. Files are placed in an "unknown" directory. | `sorter -a image.raw` |
| `-h <size>` | `--hash <size>` | Set the hash size in bits. Use `md5` (default), `sha1`, or `sha256`. Affects both deduplication and output naming. | `sorter -h sha256 image.raw` |
| `-k <file>` | `--known <file>` | Path to a file containing known-good hashes (one per line). Files matching these hashes are excluded from the output, allowing analysts to focus on unknown files. | `sorter -k known_hashes.txt image.raw` |
| `-o <dir>` | `--output <dir>` | Output directory for sorted files. Created automatically if it does not exist. | `sorter -o /evidence/sorted image.raw` |
| `-s` | `--summary` | Print a summary of file counts per category to stdout when sorting completes. | `sorter -s image.raw` |
| `-V` | `--version` | Display version information and exit. | `sorter -V` |
| `-h` | `--help` | Display usage information and exit. | `sorter -h` |

### Basic Example: Sorting Files by Type

```bash
# Sort all files in a disk image by file type
sudo sorter -o /evidence/sorted /evidence/suspect.raw
```

This creates a directory structure like:

```
/evidence/sorted/
├── application/
│   ├── pdf/
│   ├── zip/
│   └── x-executable/
├── image/
│   ├── jpeg/
│   ├── png/
│   └── gif/
├── text/
│   ├── plain/
│   └── html/
├── video/
│   └── mp4/
└── unknown/
    └── (files with unrecognized types)
```

### Sorting with SHA-256 Hashing

```bash
# Use SHA-256 instead of MD5 for stronger hash values
sudo sorter -h sha256 -o /evidence/sorted /evidence/suspect.raw
```

### Generating a Summary Report

```bash
# Sort and print a summary of file counts per category
sudo sorter -s -o /evidence/sorted /evidence/suspect.raw
```

Example summary output:

```
Sorted 15847 files:
  application/pdf: 234
  application/zip: 1892
  application/x-executable: 456
  image/jpeg: 4521
  image/png: 892
  image/gif: 234
  text/plain: 2341
  text/html: 567
  video/mp4: 89
  unknown: 4321
```

### Excluding Known-Good Files

```bash
# Create a known-hashes file (one MD5 hash per line)
md5deep -r /evidence/known_safe_files/ > /evidence/known_hashes.txt

# Sort, excluding files matching known-good hashes
sudo sorter -k /evidence/known_hashes.txt -o /evidence/sorted /evidence/suspect.raw
```

### Sorting Without the `-a` Flag (Default Behavior)

By default, sorter only copies files that it can recognize by type. To include files with unrecognized types (placed in an "unknown" directory), use the `-a` flag:

```bash
# Default: only recognized file types
sudo sorter -o /evidence/sorted /evidence/suspect.raw

# Include unrecognized files in an "unknown" directory
sudo sorter -a -o /evidence/sorted /evidence/suspect.raw
```

---

## Chapter 4: Intermediate Usage

### Bash Scripting with sorter

#### Comprehensive Triage Pipeline

```bash
#!/bin/bash
# forensic_triage.sh — Full triage pipeline using sorter and TSK tools
# Usage: ./forensic_triage.sh <raw_image> <output_base>

IMAGE="$1"
OUTPUT_BASE="$2"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
CASE_DIR="${OUTPUT_BASE}/case_${TIMESTAMP}"

if [ -z "$IMAGE" ] || [ ! -f "$IMAGE" ]; then
    echo "Usage: $0 <raw_image> <output_base>"
    exit 1
fi

echo "=== Forensic Triage Pipeline ==="
echo "Image: $IMAGE"
echo "Output: $CASE_DIR"
echo "Started: $(date)"
echo ""

# Create case directory structure
mkdir -p "$CASE_DIR"/{sorted,reports,hashes,memory}

# Step 1: Image information
echo "[1/5] Gathering image information..."
echo "Image size: $(stat -c%s "$IMAGE") bytes" > "$CASE_DIR/reports/image_info.txt"
file "$IMAGE" >> "$CASE_DIR/reports/image_info.txt"
mmls "$IMAGE" >> "$CASE_DIR/reports/partition_table.txt" 2>/dev/null

# Step 2: Compute full image hash
echo "[2/5] Computing image hash..."
md5sum "$IMAGE" > "$CASE_DIR/hashes/image_md5.txt"
sha256sum "$IMAGE" > "$CASE_DIR/hashes/image_sha256.txt"

# Step 3: Run sorter with SHA-256
echo "[3/5] Sorting files by type and hash..."
sudo sorter -h sha256 -s -a -o "$CASE_DIR/sorted" "$IMAGE" \
    > "$CASE_DIR/reports/sorter_summary.txt"

# Step 4: Generate file listing
echo "[4/5] Generating file listing..."
fls -r -d "$IMAGE" > "$CASE_DIR/reports/file_listing.txt" 2>/dev/null

# Step 5: Hash all extracted files
echo "[5/5] Hashing extracted files..."
find "$CASE_DIR/sorted" -type f -exec md5sum {} \; \
    > "$CASE_DIR/hashes/extracted_files_md5.txt"

echo ""
echo "=== Triage Complete ==="
echo "Results in: $CASE_DIR"
echo "Sorted files: $CASE_DIR/sorted/"
echo "Reports: $CASE_DIR/reports/"
echo "Hashes: $CASE_DIR/hashes/"
echo "Completed: $(date)"
```

#### Automated NSRL Database Comparison

```bash
#!/bin/bash
# nsrl_filter.sh — Filter sorter output using NSRL reference hashes
# Downloads the NSRL minimal hash set and uses it to identify known files

IMAGE="$1"
OUTPUT_DIR="$2"
NSRL_HASHES="/usr/share/nsrl/hashes.txt"  # NSRL hash file location

if [ ! -f "$NSRL_HASHES" ]; then
    echo "NSRL hash file not found at $NSRL_HASHES"
    echo "Download from: https://www.nist.gov/itssrd/project/hash-value-repository"
    echo "Or use: https://github.com/rjbs/NSRL-Metadata/blob/master/nsrl_minimal.txt"
    exit 1
fi

echo "=== NSRL Filtered Sort ==="
echo "Excluding known-good files (NSRL database)..."

sudo sorter -k "$NSRL_HASHES" -s -a -o "$OUTPUT_DIR" "$IMAGE"

echo ""
echo "Files in 'unknown/' directory require manual review."
echo "These are files NOT found in the NSRL database."
```

#### Parallel Sorting with Partition Detection

```bash
#!/bin/bash
# parallel_sort.sh — Sort each partition of a disk image in parallel
# Usage: ./parallel_sort.sh <full_disk_image>

IMAGE="$1"
OUTPUT_BASE="/evidence/parallel_sorted"
mkdir -p "$OUTPUT_BASE"

echo "Detecting partitions..."
mmls "$IMAGE" 2>/dev/null | grep "^0x" | while read line; do
    # Parse partition info (simplified)
    SECTOR_START=$(echo "$line" | awk '{print $3}')
    SECTOR_SIZE=$(echo "$line" | awk '{print $4}')
    PART_TYPE=$(echo "$line" | awk '{print $7}')
    
    BYTE_OFFSET=$((SECTOR_START * 512))
    BYTE_SIZE=$((SECTOR_SIZE * 512))
    
    echo "Partition: type=$PART_TYPE offset=$BYTE_OFFSET size=$BYTE_SIZE"
    
    PART_DIR="$OUTPUT_BASE/partition_${SECTOR_START}"
    mkdir -p "$PART_DIR"
    
    # Extract partition as raw image
    dd if="$IMAGE" bs=512 skip="$SECTOR_START" count="$SECTOR_SIZE" \
        of="$PART_DIR/partition.raw" 2>/dev/null
    
    # Sort the partition in the background
    sudo sorter -s -a -o "$PART_DIR/sorted" "$IMAGE" &
done

wait
echo "All partitions sorted."
```

### Python Scripting with sorter

#### Automating sorter with subprocess

```python
#!/usr/bin/env python3
"""
tsk_sorter.py — Python wrapper for the sorter tool.
Runs sorter, parses the summary output, and generates a structured report.
"""

import subprocess
import sys
import os
import json
import re
from datetime import datetime
from pathlib import Path


def run_sorter(image_path: str, output_dir: str,
               hash_algo: str = "sha256",
               known_hashes_file: str = None,
               include_unknown: bool = True) -> dict:
    """
    Run the sorter tool on a disk image and return structured results.
    
    Args:
        image_path: Path to the disk image file.
        output_dir: Directory where sorted files will be placed.
        hash_algo: Hash algorithm — 'md5', 'sha1', or 'sha256'.
        known_hashes_file: Optional path to a file of known-good hashes.
        include_unknown: If True, include files with unrecognized types.
    
    Returns:
        Dictionary with sorter results and metadata.
    """
    cmd = ["sudo", "sorter"]
    
    # Hash algorithm
    cmd.extend(["-h", hash_algo])
    
    # Output directory
    cmd.extend(["-o", output_dir])
    
    # Include unknown file types
    if include_unknown:
        cmd.append("-a")
    
    # Known hashes exclusion
    if known_hashes_file and os.path.isfile(known_hashes_file):
        cmd.extend(["-k", known_hashes_file])
    
    # Image path (must be last)
    cmd.append(image_path)
    
    print(f"Running: {' '.join(cmd)}")
    
    result = {
        "image": image_path,
        "output_dir": output_dir,
        "hash_algorithm": hash_algo,
        "known_hashes_file": known_hashes_file,
        "include_unknown": include_unknown,
        "start_time": datetime.now().isoformat(),
        "categories": {},
        "total_files": 0,
        "errors": []
    }
    
    try:
        proc = subprocess.run(
            cmd, capture_output=True, text=True, timeout=3600
        )
        
        # Parse summary output for file counts per category
        summary_pattern = re.compile(
            r"(\S+/\S+):\s+(\d+)"
        )
        
        total_match = re.search(
            r"Sorted\s+(\d+)\s+files", proc.stdout
        )
        if total_match:
            result["total_files"] = int(total_match.group(1))
        
        for match in summary_pattern.finditer(proc.stdout):
            category = match.group(1)
            count = int(match.group(2))
            result["categories"][category] = count
        
        if proc.returncode != 0:
            result["errors"].append(
                f"sorter exited with code {proc.returncode}: {proc.stderr}"
            )
    
    except subprocess.TimeoutExpired:
        result["errors"].append("sorter timed out after 1 hour")
    except FileNotFoundError:
        result["errors"].append("sorter command not found — is sleuthkit installed?")
    
    result["end_time"] = datetime.now().isoformat()
    return result


def print_report(result: dict):
    """Print a formatted sorter report."""
    print("\n" + "=" * 60)
    print("SORTER REPORT")
    print("=" * 60)
    print(f"Image:           {result['image']}")
    print(f"Output:          {result['output_dir']}")
    print(f"Hash algorithm:  {result['hash_algorithm']}")
    print(f"Total files:     {result['total_files']:,}")
    print(f"Start:           {result['start_time']}")
    print(f"End:             {result['end_time']}")
    
    if result["errors"]:
        print("\nERRORS:")
        for err in result["errors"]:
            print(f"  - {err}")
    
    print("\nCATEGORIES:")
    print("-" * 60)
    
    sorted_cats = sorted(
        result["categories"].items(),
        key=lambda x: x[1], reverse=True
    )
    
    for category, count in sorted_cats:
        bar = "#" * min(count // 10, 50)
        print(f"  {category:35s}  {count:>6d}  {bar}")
    
    print("=" * 60)


def main():
    if len(sys.argv) < 2:
        print(f"Usage: {sys.argv[0]} <image_file> [output_dir]")
        print("Wraps the TSK sorter tool with structured output.")
        sys.exit(1)
    
    image_path = sys.argv[1]
    output_dir = sys.argv[2] if len(sys.argv) > 2 else \
        f"{Path(image_path).stem}_sorted"
    
    if not os.path.isfile(image_path):
        print(f"Error: Image file not found: {image_path}")
        sys.exit(1)
    
    os.makedirs(output_dir, exist_ok=True)
    
    result = run_sorter(image_path, output_dir)
    print_report(result)
    
    # Save JSON report
    report_path = os.path.join(output_dir, "sorter_report.json")
    with open(report_path, "w") as f:
        json.dump(result, f, indent=2)
    print(f"\nReport saved to: {report_path}")


if __name__ == "__main__":
    main()
```

#### Hash Database Builder for sorter

```python
#!/usr/bin/env python3
"""
hash_db_builder.py — Build known-hash databases for use with sorter.
Computes hashes of known-good files and outputs them in a format
compatible with sorter's -k flag.
"""

import hashlib
import os
import sys
from pathlib import Path
from concurrent.futures import ThreadPoolExecutor, as_completed


def compute_file_hash(filepath: str, algorithm: str = "md5") -> str:
    """Compute the hash of a file using the specified algorithm."""
    h = hashlib.new(algorithm)
    try:
        with open(filepath, "rb") as f:
            while True:
                chunk = f.read(8192)
                if not chunk:
                    break
                h.update(chunk)
        return h.hexdigest()
    except (PermissionError, OSError):
        return None


def scan_directory(directory: str, algorithm: str = "md5",
                   max_workers: int = 8) -> list:
    """Scan a directory recursively and compute hashes for all files."""
    results = []
    files = []
    
    for root, dirs, filenames in os.walk(directory):
        for fname in filenames:
            files.append(os.path.join(root, fname))
    
    print(f"Computing {algorithm.upper()} hashes for {len(files)} files...")
    
    with ThreadPoolExecutor(max_workers=max_workers) as executor:
        futures = {
            executor.submit(compute_file_hash, f, algorithm): f
            for f in files
        }
        for future in as_completed(futures):
            filepath = futures[future]
            file_hash = future.result()
            if file_hash:
                results.append(file_hash)
    
    return results


def build_known_hashes(known_dirs: list, output_file: str,
                       algorithm: str = "md5"):
    """Build a known-hashes database from one or more directories."""
    all_hashes = set()
    
    for directory in known_dirs:
        if os.path.isdir(directory):
            hashes = scan_directory(directory, algorithm)
            all_hashes.update(hashes)
            print(f"  {directory}: {len(hashes)} hashes computed")
        else:
            print(f"  Warning: {directory} is not a directory, skipping")
    
    # Write hashes to file (one per line, as expected by sorter -k)
    with open(output_file, "w") as f:
        for h in sorted(all_hashes):
            f.write(f"{h}\n")
    
    print(f"\nTotal unique hashes: {len(all_hashes)}")
    print(f"Output written to: {output_file}")
    return len(all_hashes)


if __name__ == "__main__":
    if len(sys.argv) < 3:
        print(f"Usage: {sys.argv[0]} <output_file> <dir1> [dir2] ...")
        print("Builds a known-hashes database from the given directories.")
        sys.exit(1)
    
    output_file = sys.argv[1]
    directories = sys.argv[2:]
    
    build_known_hashes(directories, output_file)
```

#### Post-Sorter Analysis Script

```python
#!/usr/bin/env python3
"""
post_sort_analysis.py — Analyze sorter output directory.
Counts files per category, identifies suspicious patterns,
and generates an actionable summary.
"""

import os
import sys
import json
from pathlib import Path
from collections import defaultdict


SUSPICIOUS_TYPES = {
    "application/x-executable": "Potential malware or tools",
    "application/x-dosexec": "Windows executable (DOS stub)",
    "application/x-ms-dos-executable": "Windows executable",
    "application/x-pearl": "Perl script",
    "application/x-php": "PHP script",
    "application/x-python": "Python script",
    "application/javascript": "JavaScript file",
    "text/x-shellscript": "Shell script",
    "text/x-perl": "Perl script",
    "application/x-bat": "Windows batch file",
    "application/vnd.ms-cab-compressed": "CAB archive (possible payload)",
    "application/x-rar": "RAR archive (possible encrypted payload)",
    "application/x-7z-compressed": "7z archive (possible payload)",
}

KNOWN_IMAGE_TYPES = {
    "image/jpeg", "image/png", "image/gif",
    "image/bmp", "image/tiff", "image/webp"
}

KNOWN_DOC_TYPES = {
    "application/pdf", "application/msword",
    "application/vnd.openxmlformats-officedocument",
    "text/plain", "text/html"
}


def analyze_sorter_output(sorted_dir: str) -> dict:
    """Analyze the sorter output directory and generate a summary."""
    analysis = {
        "sorted_dir": sorted_dir,
        "total_files": 0,
        "categories": defaultdict(int),
        "suspicious_files": [],
        "image_count": 0,
        "document_count": 0,
        "executable_count": 0,
        "archive_count": 0,
        "unknown_count": 0,
    }
    
    for root, dirs, files in os.walk(sorted_dir):
        for f in files:
            filepath = os.path.join(root, f)
            rel_path = os.path.relpath(filepath, sorted_dir)
            
            # Determine category from directory path
            parts = rel_path.split(os.sep)
            category = "/".join(parts[:2]) if len(parts) >= 2 else "unknown"
            
            analysis["categories"][category] += 1
            analysis["total_files"] += 1
            
            # Check if suspicious
            for susp_type, reason in SUSPICIOUS_TYPES.items():
                if susp_type in category:
                    analysis["suspicious_files"].append({
                        "path": rel_path,
                        "type": category,
                        "reason": reason
                    })
                    analysis["executable_count"] += 1
                    break
            
            # Categorize by broad type
            if any(category.startswith(t) for t in KNOWN_IMAGE_TYPES):
                analysis["image_count"] += 1
            elif any(category.startswith(t) for t in KNOWN_DOC_TYPES):
                analysis["document_count"] += 1
            elif "unknown" in category:
                analysis["unknown_count"] += 1
    
    return dict(analysis)


def print_analysis(analysis: dict):
    """Print the analysis report."""
    print("\n" + "=" * 60)
    print("POST-SORT ANALYSIS REPORT")
    print("=" * 60)
    print(f"Sorted directory: {analysis['sorted_dir']}")
    print(f"Total files:      {analysis['total_files']:,}")
    print()
    
    print(f"  Images:          {analysis['image_count']:>8,}")
    print(f"  Documents:       {analysis['document_count']:>8,}")
    print(f"  Executables:     {analysis['executable_count']:>8,}")
    print(f"  Archives:        {analysis['archive_count']:>8,}")
    print(f"  Unknown type:    {analysis['unknown_count']:>8,}")
    print()
    
    if analysis["suspicious_files"]:
        print(f"SUSPICIOUS FILES ({len(analysis['suspicious_files'])}):")
        print("-" * 60)
        for item in analysis["suspicious_files"][:50]:
            print(f"  [{item['type']}] {item['path']}")
            print(f"    -> {item['reason']}")
        if len(analysis["suspicious_files"]) > 50:
            print(f"  ... and {len(analysis['suspicious_files']) - 50} more")
    else:
        print("No suspicious file types detected.")
    
    print("=" * 60)


if __name__ == "__main__":
    if len(sys.argv) < 2:
        print(f"Usage: {sys.argv[0]} <sorted_output_directory>")
        sys.exit(1)
    
    analysis = analyze_sorter_output(sys.argv[1])
    print_analysis(analysis)
    
    # Save as JSON
    output_path = os.path.join(sys.argv[1], "analysis_report.json")
    with open(output_path, "w") as f:
        json.dump(analysis, f, indent=2, default=str)
    print(f"\nJSON report: {output_path}")
```

---

## Chapter 5: Advanced Usage

### Using sorter with Known File Exclusion Databases

The `-k` flag is one of sorter's most powerful features for triage. By providing a file of known-good hashes, sorter excludes those files from its output, leaving only unknown or potentially suspicious files:

```bash
# Build a known-hashes database from a trusted system
sudo find /usr /lib /etc -type f -exec md5sum {} \; \
    | awk '{print $1}' | sort -u > /evidence/known_good_hashes.txt

# Sort, excluding known-good system files
sudo sorter -k /evidence/known_good_hashes.txt \
    -s -a -o /evidence/sorted_unknown /evidence/suspect.raw
```

### Integration with NSRL (National Software Reference Library)

The NSRL provides a hash database of known software. sorter can use this directly:

```bash
# Download NSRL minimal hash set
wget https://s3.amazonaws.com/nsrlrds/REPO/RDS_2.82_Lite.zip
unzip RDS_2.82_Lite.zip

# Extract MD5 hashes from the NSRL CSV
awk -F',' 'NR>1 {print $1}' hash-values.csv > /evidence/nsrl_hashes.txt

# Sort, excluding NSRL-known software
sudo sorter -k /evidence/nsrl_hashes.txt -s -a -o /evidence/nsrl_filtered /evidence/suspect.raw
```

### Combining sorter with Other TSK Tools

```bash
#!/bin/bash
# full_analysis.sh — Comprehensive analysis pipeline
# Uses mmls, fls, sorter, and tsk_recover in sequence

IMAGE="$1"
WORK_DIR="/evidence/analysis"
mkdir -p "$WORK_DIR"

echo "=== Full Disk Analysis Pipeline ==="

# Phase 1: Partition layout
echo "[Phase 1] Partition layout..."
mmls "$IMAGE" > "$WORK_DIR/partitions.txt" 2>/dev/null

# Phase 2: Full file listing
echo "[Phase 2] File listing..."
fls -r -d -m "" "$IMAGE" > "$WORK_DIR/file_list.csv" 2>/dev/null

# Phase 3: Sort files by type and hash
echo "[Phase 3] Sorting files..."
sudo sorter -h sha256 -s -a -o "$WORK_DIR/sorted" "$IMAGE" \
    > "$WORK_DIR/sorter_summary.txt"

# Phase 4: Recover deleted files
echo "[Phase 4] Recovering deleted files..."
sudo tsk_recover -e raw -o 2048 "$IMAGE" "$WORK_DIR/recovered/" 2>/dev/null

# Phase 5: Hash all recovered files
echo "[Phase 5] Hashing recovered files..."
find "$WORK_DIR/recovered" -type f -exec sha256sum {} \; \
    > "$WORK_DIR/recovered_hashes.txt"

echo "=== Pipeline Complete ==="
echo "Results in: $WORK_DIR"
```

### Custom Category Mapping

sorter uses the system's `file` command (libmagic) to determine file types. You can customize the categorization by modifying the magic database or post-processing the output:

```bash
# After sorter runs, rename categories for case-specific needs
cd /evidence/sorted

# Rename categories for the specific case
mv application/x-executable case_evidence/executables/ 2>/dev/null
mv application/pdf case_evidence/documents/ 2>/dev/null
mv image/jpeg case_evidence/photos/ 2>/dev/null
```

### Performance Optimization

```bash
# For very large images, use a dedicated output on a fast disk
sudo sorter -h md5 -o /fast_disk/evidence/sorted /evidence/large_image.raw

# Monitor progress (sorter doesn't show progress by default)
# Run in a screen/tmux session
screen -S sorter_session
sudo sorter -h sha256 -s -a -o /evidence/sorted /evidence/huge_image.raw
# Detach with Ctrl+A, D
```

### Handling Multiple Images in a Case

```bash
#!/bin/bash
# batch_sort.sh — Sort multiple disk images for a case
# Usage: ./batch_sort.sh <output_base> <image1> <image2> ...

OUTPUT_BASE="$1"
shift

for IMAGE in "$@"; do
    BASENAME=$(basename "$IMAGE" .raw)
    echo "Processing: $IMAGE"
    sudo sorter -h sha256 -s -a \
        -o "$OUTPUT_BASE/$BASENAME" \
        "$IMAGE" \
        > "$OUTPUT_BASE/${BASENAME}_summary.txt"
    echo "Done: $BASENAME"
    echo ""
done
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: Corporate Incident Response — Insider Threat Investigation

**Context**: An employee is suspected of exfiltrating proprietary source code and design documents. The IT security team has created a forensic image of the employee's workstation hard drive. The investigation team needs to quickly identify all source code files, documents, and any evidence of data staging (compressed archives).

**Approach**: Use sorter to categorize all files by type, then examine the relevant categories for evidence of data staging.

```bash
# Step 1: Create the image
sudo dd if=/dev/sdb of=/evidence/employee_ws.raw bs=4M status=progress

# Step 2: Identify partitions
mmls /evidence/employee_ws.raw
# Output shows: sda1 (NTFS, offset 2048), sda2 (Linux, offset 100000000)

# Step 3: Sort files, excluding known system files
sudo find /usr /lib -type f -exec md5sum {} \; 2>/dev/null \
    | awk '{print $1}' > /evidence/known_system_hashes.txt

sudo sorter -h sha256 -k /evidence/known_system_hashes.txt \
    -s -a -o /evidence/employee_sorted \
    /evidence/employee_ws.raw \
    > /evidence/sorter_output.txt

# Step 4: Examine suspicious categories
# Check for compressed archives (potential data staging)
ls -la /evidence/employee_sorted/application/zip/
ls -la /evidence/employee_sorted/application/x-rar/
ls -la /evidence/employee_sorted/application/x-7z-compressed/

# Check for source code files
find /evidence/employee_sorted -name "*.py" -o -name "*.java" \
    -o -name "*.cpp" -o -name "*.c" -o -name "*.h"

# Step 5: Generate hash report of all extracted files
find /evidence/employee_sorted -type f -exec sha256sum {} \; \
    > /evidence/extracted_file_hashes.txt
```

**Outcome**: The analysis reveals 847 compressed archives (mostly ZIP files) in the employee's home directory, several of which contain source code files and design documents matching the company's proprietary projects. The timestamps on these archives correlate with the period of suspected data exfiltration.

### Scenario 2: Child Safety Investigation — Hash-Based Triage

**Context**: A child exploitation investigation requires processing a large hard drive image. Known CSAM material has been cataloged with hash values. The investigation team needs to quickly identify all unknown images that haven't been previously cataloged.

**Approach**: Use sorter with SHA-256 hashing and the known-hash database to isolate unknown images.

```bash
# Step 1: Prepare the known-hashes database
# (Hashes of previously cataloged CSAM from national databases)
cat /evidence/national_hash_db.txt /evidence/case_hashes.txt \
    | sort -u > /evidence/combined_known_hashes.txt

# Step 2: Sort the disk image
sudo sorter -h sha256 -k /evidence/combined_known_hashes.txt \
    -s -a -o /evidence/sorted_output \
    /evidence/suspect_drive.raw \
    > /evidence/sorter_log.txt

# Step 3: Focus on image files (the unknown ones)
echo "=== Unknown Image Files ==="
find /evidence/sorted_output/image/ -type f | wc -l

# Step 4: Generate a listing for review
find /evidence/sorted_output/image/ -type f -exec sha256sum {} \; \
    > /evidence/unknown_image_hashes.txt

# Step 5: Create a reviewable HTML gallery
# (Using a custom script)
python3 /scripts/image_gallery.py \
    /evidence/sorted_output/image/ \
    /evidence/review_gallery.html
```

**Outcome**: The sorter tool identified 15,234 image files on the suspect drive, of which 12,891 matched known hashes and were automatically excluded. The remaining 2,343 unknown images were cataloged, hashed, and prepared for expert review.

### Scenario 3: Malware Triage — Identifying Payloads in Disk Image

**Context**: A malware analyst has been given a disk image from a compromised server. The goal is to identify all executable files, scripts, and archives that could contain malware, while filtering out known operating system files.

**Approach**: Use sorter with a known-hashes database of clean OS files, then analyze the remaining executables and scripts.

```bash
# Step 1: Build a known-clean database from a fresh OS install
# (On a clean reference system matching the compromised server)
sudo find / -type f \( -perm -111 -o -name "*.sh" -o -name "*.py" \
    -o -name "*.pl" -o -name "*.rb" \) 2>/dev/null \
    -exec md5sum {} \; > /evidence/clean_os_hashes.txt

# Step 2: Sort the compromised image, excluding known-clean files
sudo sorter -h md5 -k /evidence/clean_os_hashes.txt \
    -s -a -o /evidence/malware_sorted \
    /evidence/compromised_server.raw \
    > /evidence/sorter_log.txt

# Step 3: Examine executables
echo "=== Unknown Executables ==="
ls -la /evidence/malware_sorted/application/x-executable/
ls -la /evidence/malware_sorted/application/x-dosexec/

# Step 4: Examine scripts
echo "=== Unknown Scripts ==="
find /evidence/malware_sorted -name "*.sh" -o -name "*.py" \
    -o -name "*.pl" -o -name "*.rb" -o -name "*.php"

# Step 5: Examine archives (may contain second-stage payloads)
echo "=== Unknown Archives ==="
find /evidence/malware_sorted/application/zip \
    /evidence/malware_sorted/application/x-7z-compressed \
    -type f 2>/dev/null

# Step 6: Run each unknown executable against VirusTotal (if authorized)
for f in /evidence/malware_sorted/application/x-executable/*; do
    echo "Checking: $(basename $f)"
    md5sum "$f"
    # vt submit "$f"  # VirusTotal submission (if API key available)
done
```

**Outcome**: The analysis identified 23 unknown executables, 15 scripts (including 3 obfuscated Python scripts), and 7 archives that were not present in the clean reference system. The 3 obfuscated Python scripts were found to be command-and-control beacons, and the archives contained second-stage payloads including a keylogger and a reverse shell.

---

## Chapter 7: Detection & Defense

### How Defenders Can Use sorter

- **Baseline monitoring**: Create a sorted snapshot of a clean system, then periodically compare against current state to detect new files.
- **Incident response**: Quickly triage a compromised system to identify all file types and isolate suspicious categories.
- **Data loss prevention**: Sort backups to identify sensitive file types (source code, databases, financial records) that need additional protection.
- **Malware hunting**: Identify executables and scripts that don't match known-good hashes.

### Detecting sorter Usage

From an adversary awareness perspective:

- sorter does not modify the source image, so its use is undetectable from the image itself.
- Command-line history records sorter invocations.
- The output directory structure reveals which file types were of interest.

### Defensive Measures Against Automated File Sorting

Adversaries may try to evade detection by sorter:

- **File type obfuscation**: Renaming files to disguise their true type (though sorter uses magic numbers, not extensions, so this is ineffective).
- **Encryption**: Encrypting files so their content doesn't match any known type signature.
- **Steganography**: Hiding data within image or audio files.
- **File splitting**: Breaking large files into small chunks that individually don't match any type signature.

---

## Chapter 8: Troubleshooting (FAQ)

### Q1: sorter fails with "Permission denied" errors

sorter requires root privileges to read raw disk images. Run with `sudo`:

```bash
sudo sorter -o /evidence/sorted /evidence/image.raw
```

If the image file has restricted permissions, fix them first:

```bash
sudo chmod 644 /evidence/image.raw
```

### Q2: sorter outputs "Cannot determine filesystem type"

This usually means the image is a full disk image (not a partition image) and sorter cannot find a filesystem at offset 0. Use `mmls` to find the partition offset, then extract the partition with `dd`:

```bash
mmls /evidence/disk.raw
# Partition starts at sector 2048
dd if=/evidence/disk.raw bs=512 skip=2048 of=/evidence/partition.raw
sudo sorter -o /evidence/sorted /evidence/partition.raw
```

### Q3: How do I control the output directory structure?

sorter's directory structure is determined by the `file` command's MIME type output. You cannot directly customize the subdirectory names, but you can post-process the output:

```bash
# After sorting, rename/reorganize directories as needed
cd /evidence/sorted
mv application/x-executable suspicious_executables/
mv image/ recovered_images/
```

### Q4: sorter is extremely slow on a large image

sorter reads every file in the image, which can be time-consuming on large images. Optimize by:

1. Sorting a partition image instead of the full disk image.
2. Using `md5` instead of `sha256` for faster hashing (less critical for triage).
3. Running on faster storage (SSD vs. HDD).
4. Running in a `screen` or `tmux` session to avoid disconnection issues.

### Q5: The `-k` known-hashes file doesn't seem to work

Ensure the known-hashes file contains one hash per line, with no extra whitespace or formatting. The hash format must match what sorter computes (plain hex string):

```bash
# Correct format:
d41d8cd98f00b204e9800998ecf8427e
5d41402abc4b2a76b9719d911017c592

# Incorrect (has spaces, filenames, or formatting):
d41d8cd98f00b204e9800998ecf8427e  /path/to/file
MD5: d41d8cd98f00b204e9800998ecf8427e
```

### Q6: sorter creates an empty output directory

This typically means no files were found in the image, or all files matched the known-hashes exclusion list. Check:

1. The image contains a valid filesystem: `fls /evidence/image.raw`
2. The image is not encrypted: try `icat /evidence/image.raw <inode>` to extract a file.
3. The known-hashes file doesn't accidentally contain all the image's file hashes.

---

## Resources

### Official Documentation

- The Sleuth Kit Manual: https://sleuthkit.org/sleuthkit/man/
- TSK GitHub Repository: https://github.com/sleuthkit/sleuthkit
- Brian Carrier's File System Forensic Analysis (book, 2005)

### Related TSK Tools

| Tool | Purpose |
|------|---------|
| fls | List files and directories in a filesystem image |
| icat | Extract a specific file by inode number |
| sigfind | Find hex signatures at specific offsets in an image |
| tsk_recover | Recover deleted files from a filesystem image |
| mmls | Display partition table layout |
| md5deep | Compute recursive directory hashes |

### Hash Databases

| Database | URL | Description |
|----------|-----|-------------|
| NSRL | https://www.nist.gov/itssrd/project/hash-value-repository | National Software Reference Library |
| HashLib | https://hashlib.sh | Community-maintained hash sets |
| VirusTotal | https://www.virustotal.com | Malware hash lookups (API required) |

### Books and References

- "File System Forensic Analysis" by Brian Carrier (Addison-Wesley, 2005)
- NIST SP 800-86: Guide to Integrating Forensic Techniques into Incident Response
- "The Art of Memory Forensics" by Ligh et al. (Wiley, 2014)
- SANS FOR508: Advanced Incident Response and Threat Hunting

### Practice Resources

- Digital Forensics Challenge (DFC) images
- NIST Computer Forensics Tool Testing (CFTT) datasets
- CyberDefenders.org challenge images
- SANS forensic practice images
