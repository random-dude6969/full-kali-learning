---
tool_name: ext3grep
display_name: "Ext3 File System Recovery"
mitre_tactic: "analysis"
mitre_tactic_id: "TA0007"
install_command: "sudo apt install ext3grep"
github_repo: "https://github.com/extundelete/extundelete"
kali_tools_page: "https://www.kali.org/tools/ext3grep/"
difficulty_level: "intermediate"
requires_root: true
gui_available: false
tags:
  - forensics
  - file-carving
  - ext3
  - data-recovery
  - journal-analysis
  - linux
related_tools:
  - ext4magic
  - extundelete
  - foremost
  - photorec
  - scalpel
---

# Ext3grep - Ext3 Filesystem Forensic Recovery Tool

## Chapter 1: Introduction & Overview

### What is ext3grep?

ext3grep is a command-line forensic recovery tool designed specifically for recovering deleted files from ext3 filesystems. Unlike general-purpose file carving tools that scan raw disk images for file headers and footers, ext3grep leverages the ext3 journal to reconstruct deleted file metadata and data with remarkable precision. The tool was originally developed by Joachim Metz and has become a staple in Linux digital forensics workflows.

### Historical Context

The ext3grep project emerged in the mid-2000s when ext3 was the dominant Linux filesystem. Before ext3grep, recovering deleted files from ext3 required manual journal parsing or expensive commercial tools. The ext3 journal (JBD - Journaling Block Device) stores metadata about filesystem changes, and ext3grep exploits this to recover files that other tools might miss.

While ext3grep is primarily an ext3 tool, related projects like extundelete and ext4magic have extended similar journal-based recovery techniques to ext4 filesystems. Understanding ext3grep provides foundational knowledge for working with these derivative tools.

### When to Use ext3grep

ext3grep excels in scenarios where:

- The target filesystem uses ext3 (not ext4)
- Files were deleted recently (journal entries still present)
- You need to recover specific files by name or path
- You need to recover files from a specific time window
- The filesystem has not been heavily written to since deletion
- You need to recover files from unallocated inodes

ext3grep is NOT suitable when:

- The filesystem is ext4 or another type (use ext4magic or extundelete)
- The journal has been overwritten or corrupted
- The filesystem has been reformatted
- You need raw data carving without filesystem metadata

### Key Concepts

**ext3 Journal (JBD)**: The ext3 filesystem uses journaling to maintain consistency. Every metadata change (file creation, deletion, modification) is first written to the journal. ext3grep reads these journal entries to identify what was deleted and where.

**Inode**: Each file in ext3 has an inode containing metadata (permissions, size, block pointers). When a file is deleted, its inode is marked as free but often retains useful information.

**Directory Entries**: Directory entries map filenames to inode numbers. When a file is deleted, the directory entry is removed, but ext3grep can recover these from the journal.

**Block Pointers**: ext3 inodes store direct, indirect, double-indirect, and triple-indirect block pointers that point to the actual data blocks. ext3grep can reconstruct these pointer chains from journal entries.

**Unallocated Blocks**: Data blocks that have been freed but not yet overwritten. ext3grep can scan these to find data belonging to deleted files.

### Technical Architecture

ext3grep operates in several modes:

1. **Journal Scanning Mode**: Parses the ext3 journal to find deleted file metadata
2. **Inode Recovery Mode**: Recovers files by their inode number
3. **Directory Scanning Mode**: Recovers directory structures
4. **Block Scanning Mode**: Scans unallocated blocks for file signatures
5. **File Recovery Mode**: Recovers files by name or pattern matching

## Chapter 2: Installation & Setup

### System Requirements

- Linux operating system (Kali Linux recommended)
- ext3 filesystem (physical or image file)
- Minimum 512 MB RAM (2 GB+ recommended for large images)
- Sufficient disk space for recovered files
- Root or sudo access

### Installation Methods

#### Method 1: Kali Linux Package Manager (Recommended)

```bash
sudo apt update
sudo apt install ext3grep
```

#### Method 2: Debian/Ubuntu Package Manager

```bash
sudo apt update
sudo apt install ext3grep
```

#### Method 3: Compile from Source

```bash
# Install build dependencies
sudo apt install build-essential libext2fs-dev

# Clone the repository (related project)
git clone https://github.com/extundelete/extundelete.git
cd extundelete

# Note: For the original ext3grep, check package repositories
# as the original source may not be publicly available
```

#### Method 4: Using Kali Linux Pre-installed

Kali Linux often includes ext3grep in its forensics toolkit:

```bash
# Check if already installed
which ext3grep

# If not found, install
sudo apt install ext3grep
```

### Verification

After installation, verify ext3grep is working:

```bash
# Check version
ext3grep --version

# Expected output
# ext3grep version 0.10.2

# Check help
ext3grep --help

# Quick test with a small ext3 image
dd if=/dev/zero of=/tmp/test.img bs=1M count=10
mkfs.ext3 /tmp/test.img
```

### Preparing a Test Environment

Create a safe test environment before working with real forensic images:

```bash
# Create a test ext3 filesystem
dd if=/dev/zero of=/tmp/test_ext3.img bs=1M count=100
mkfs.ext3 /tmp/test_ext3.img

# Mount it
mkdir /tmp/test_mount
sudo mount -o loop /tmp/test_ext3.img /tmp/test_mount

# Create some test files
echo "Test file 1" | sudo tee /tmp/test_mount/file1.txt
echo "Test file 2" | sudo tee /tmp/test_mount/file2.txt
dd if=/dev/urandom of=/tmp/test_mount/binary_file.bin bs=1K count=10

# Unmount and delete
sudo umount /tmp/test_mount
```

## Chapter 3: Basic Usage

### Essential Commands

#### Recover All Deleted Files

```bash
# Basic recovery - scan entire filesystem
ext3grep /dev/sda1 --restore-all

# Recover from an image file
ext3grep /path/to/filesystem.img --restore-all
```

**Sample Output:**

```
Running ext3grep version 0.10.2
Number of groups: 16
Loading group metadata from /dev/sda1
Checking group 0
Loading journal from /dev/sda1
Number of journal blocks: 32768
Found journal at block 32768
Journal has 12847 valid entries (32768 blocks)
Found 47 deleted inode entries
Found 23 deleted directory entries
Restoring file1.txt
Restoring file2.txt
Restoring binary_file.bin
Successfully restored 3 files
```

#### Recover Specific File by Name

```bash
# Recover a specific file
ext3grep /dev/sda1 --restore-file path/to/deleted/file.txt

# Recover file with wildcard
ext3grep /dev/sda1 --restore-file "*.pdf"

# Recover file from specific directory
ext3grep /dev/sda1 --restore-file "documents/secret.pdf"
```

#### List Deleted Files

```bash
# List all deleted inodes
ext3grep /dev/sda1 --list-deleted-inodes

# List deleted directory entries
ext3grep /dev/sda1 --list-deleted-entries

# List with detailed information
ext3grep /dev/sda1 --list-deleted-inodes --details
```

**Sample Output:**

```
Deleted inodes found:
Inode  Type  Size    Blocks  Deleted Time
12345  REG   1024    8       2024-01-15 14:32:00
12346  REG   2048    16      2024-01-15 14:32:15
12347  DIR   4096    8       2024-01-15 14:31:45
12348  REG   512     8       2024-01-15 13:45:00
```

### Complete Flag Reference

| Flag | Description | Example |
|------|-------------|---------|
| `--restore-all` | Recover all deleted files | `ext3grep /dev/sda1 --restore-all` |
| `--restore-file PATH` | Recover specific file | `ext3grep /dev/sda1 --restore-file doc.pdf` |
| `--list-deleted-inodes` | Show deleted inodes | `ext3grep /dev/sda1 --list-deleted-inodes` |
| `--list-deleted-entries` | Show deleted dir entries | `ext3grep /dev/sda1 --list-deleted-entries` |
| `--details` | Show detailed inode info | `ext3grep /dev/sda1 --list-deleted-inodes --details` |
| `--output-dir DIR` | Set output directory | `ext3grep /dev/sda1 --restore-all --output-dir /recovered` |
| `--after TIMESTAMP` | Filter by time (after) | `ext3grep /dev/sda1 --restore-all --after 1705312800` |
| `--before TIMESTAMP` | Filter by time (before) | `ext3grep /dev/sda1 --restore-all --before 1705399200` |
| `--show-blocks` | Display block numbers | `ext3grep /dev/sda1 --list-deleted-inodes --show-blocks` |
| `--show-journal` | Show journal entries | `ext3grep /dev/sda1 --show-journal` |
| `--journal-block BLOCK` | Start journal at block | `ext3grep /dev/sda1 --show-journal --journal-block 32768` |
| `--inode INODE` | Show specific inode | `ext3grep /dev/sda1 --inode 12345` |
| `--block BLOCK` | Show specific block | `ext3grep /dev/sda1 --block 65536` |
| `--superblock` | Show superblock info | `ext3grep /dev/sda1 --superblock` |
| `--version` | Show version | `ext3grep --version` |
| `--help` | Show help | `ext3grep --help` |

### Basic Recovery Scenarios

#### Scenario 1: Recover Recently Deleted File

```bash
# Step 1: Identify the partition
fdisk -l /dev/sda
# Output shows /dev/sda1 is ext3

# Step 2: List deleted files
sudo ext3grep /dev/sda1 --list-deleted-inodes --details

# Step 3: Recover specific file
sudo ext3grep /dev/sda1 --restore-file important_document.doc

# Step 4: Check recovered files
ls -la RESTORED_FILES/
```

#### Scenario 2: Recover All Files from Time Window

```bash
# Files deleted between 2024-01-15 14:00 and 15:00
# Convert to Unix timestamps:
# 2024-01-15 14:00:00 = 1705312800
# 2024-01-15 15:00:00 = 1705316400

sudo ext3grep /dev/sda1 --restore-all \
  --after 1705312800 \
  --before 1705316400 \
  --output-dir /forensics/recovered
```

## Chapter 4: Intermediate Usage

### Advanced Journal Analysis

#### Parsing Journal Entries Manually

```bash
# Show raw journal entries
sudo ext3grep /dev/sda1 --show-journal --journal-block 32768

# Filter journal for specific inode
sudo ext3grep /dev/sda1 --show-journal | grep "inode 12345"

# Extract journal to file for analysis
sudo ext3grep /dev/sda1 --show-journal > journal_dump.txt
```

#### Journal Structure Analysis

```bash
# Show journal superblock
sudo ext3grep /dev/sda1 --show-journal --details

# Analyze journal blocks
sudo ext3grep /dev/sda1 --block 32768 --details
sudo ext3grep /dev/sda1 --block 32769 --details
```

### Scripting with ext3grep

#### Bash Script: Automated Recovery

```bash
#!/bin/bash
# ext3_auto_recover.sh - Automated ext3grep recovery script

DEVICE=$1
OUTPUT_DIR=$2
TIMESTAMP_AFTER=$3
TIMESTAMP_BEFORE=$4

if [ -z "$DEVICE" ] || [ -z "$OUTPUT_DIR" ]; then
    echo "Usage: $0 <device> <output_dir> [after_timestamp] [before_timestamp]"
    exit 1
fi

# Create output directory
mkdir -p "$OUTPUT_DIR"

echo "[*] Starting recovery on $DEVICE"
echo "[*] Output directory: $OUTPUT_DIR"

# Step 1: List deleted inodes
echo "[*] Listing deleted inodes..."
sudo ext3grep "$DEVICE" --list-deleted-inodes --details > "$OUTPUT_DIR/deleted_inodes.txt"

# Step 2: List deleted directory entries
echo "[*] Listing deleted entries..."
sudo ext3grep "$DEVICE" --list-deleted-entries > "$OUTPUT_DIR/deleted_entries.txt"

# Step 3: Recover files with optional time filter
echo "[*] Recovering files..."
if [ -n "$TIMESTAMP_AFTER" ] && [ -n "$TIMESTAMP_BEFORE" ]; then
    sudo ext3grep "$DEVICE" --restore-all \
        --after "$TIMESTAMP_AFTER" \
        --before "$TIMESTAMP_BEFORE" \
        --output-dir "$OUTPUT_DIR/files"
else
    sudo ext3grep "$DEVICE" --restore-all \
        --output-dir "$OUTPUT_DIR/files"
fi

# Step 4: Generate summary
echo "[*] Generating summary..."
cat > "$OUTPUT_DIR/recovery_summary.txt" << EOF
Recovery Summary
================
Device: $DEVICE
Date: $(date)
Files recovered: $(find "$OUTPUT_DIR/files" -type f | wc -l)
Total size: $(du -sh "$OUTPUT_DIR/files" | cut -f1)
EOF

echo "[+] Recovery complete. Summary: $OUTPUT_DIR/recovery_summary.txt"
```

#### Python Script: Advanced Analysis

```python
#!/usr/bin/env python3
"""
ext3_analyzer.py - Python wrapper for ext3grep analysis
"""

import subprocess
import json
import re
import sys
from datetime import datetime
from pathlib import Path

class Ext3Analyzer:
    def __init__(self, device):
        self.device = device
        self.deleted_inodes = []
        self.deleted_entries = []
    
    def list_deleted_inodes(self):
        """Parse deleted inodes from ext3grep output"""
        cmd = ['sudo', 'ext3grep', self.device, 
               '--list-deleted-inodes', '--details']
        result = subprocess.run(cmd, capture_output=True, text=True)
        
        for line in result.stdout.split('\n'):
            if line.strip() and not line.startswith('Deleted'):
                parts = line.split()
                if len(parts) >= 5:
                    inode_info = {
                        'inode': int(parts[0]),
                        'type': parts[1],
                        'size': int(parts[2]),
                        'blocks': int(parts[3]),
                        'deleted_time': ' '.join(parts[4:])
                    }
                    self.deleted_inodes.append(inode_info)
        
        return self.deleted_inodes
    
    def list_deleted_entries(self):
        """Parse deleted directory entries"""
        cmd = ['sudo', 'ext3grep', self.device, 
               '--list-deleted-entries']
        result = subprocess.run(cmd, capture_output=True, text=True)
        
        for line in result.stdout.split('\n'):
            if line.strip() and not line.startswith('Deleted'):
                self.deleted_entries.append(line.strip())
        
        return self.deleted_entries
    
    def recover_file(self, filepath, output_dir):
        """Recover a specific file"""
        cmd = ['sudo', 'ext3grep', self.device, 
               '--restore-file', filepath,
               '--output-dir', output_dir]
        result = subprocess.run(cmd, capture_output=True, text=True)
        return result.returncode == 0
    
    def recover_by_time(self, after, before, output_dir):
        """Recover files within a time window"""
        cmd = ['sudo', 'ext3grep', self.device, 
               '--restore-all',
               '--after', str(after),
               '--before', str(before),
               '--output-dir', output_dir]
        result = subprocess.run(cmd, capture_output=True, text=True)
        return result.returncode == 0
    
    def generate_report(self, output_file):
        """Generate JSON report of findings"""
        report = {
            'device': self.device,
            'timestamp': datetime.now().isoformat(),
            'deleted_inodes': self.deleted_inodes,
            'deleted_entries': self.deleted_entries
        }
        
        with open(output_file, 'w') as f:
            json.dump(report, f, indent=2)
        
        return report

def main():
    if len(sys.argv) < 2:
        print(f"Usage: {sys.argv[0]} <device>")
        sys.exit(1)
    
    device = sys.argv[1]
    analyzer = Ext3Analyzer(device)
    
    print(f"[*] Analyzing {device}...")
    
    # List deleted inodes
    print("[*] Listing deleted inodes...")
    inodes = analyzer.list_deleted_inodes()
    print(f"[+] Found {len(inodes)} deleted inodes")
    
    # List deleted entries
    print("[*] Listing deleted entries...")
    entries = analyzer.list_deleted_entries()
    print(f"[+] Found {len(entries)} deleted entries")
    
    # Generate report
    report_file = f"ext3_analysis_{datetime.now().strftime('%Y%m%d_%H%M%S')}.json"
    analyzer.generate_report(report_file)
    print(f"[+] Report saved to {report_file}")

if __name__ == '__main__':
    main()
```

### Combining with Other Tools

#### Workflow: ext3grep + foremost

```bash
# Step 1: Use ext3grep for journal-based recovery
sudo ext3grep /dev/sda1 --restore-all --output-dir /recovery/ext3grep

# Step 2: Use foremost for additional carving on unallocated blocks
sudo foremost -t all -i /dev/sda1 -o /recovery/foremost

# Step 3: Compare results
echo "ext3grep recovered: $(find /recovery/ext3grep -type f | wc -l) files"
echo "foremost recovered: $(find /recovery/foremost -type f | wc -l) files"
```

#### Workflow: ext3grep + photorec

```bash
# Create forensic image first
sudo dd if=/dev/sda1 of=/forensics/evidence/sda1.img bs=4M status=progress

# Use ext3grep on image
sudo ext3grep /forensics/evidence/sda1.img --restore-all --output-dir /recovery/ext3grep

# Use photorec on same image for comparison
sudo photorec /d /recovery/photorec /cmd /forensics/evidence/sda1.img search
```

#### Workflow: ext3grep + dd for evidence preservation

```bash
# Create bit-for-bit copy of evidence
sudo dd if=/dev/sda1 of=/forensics/evidence/sda1.img bs=4M status=progress

# Verify image integrity
md5sum /dev/sda1
md5sum /forensics/evidence/sda1.img

# Work only on the copy
sudo ext3grep /forensics/evidence/sda1.img --restore-all \
  --output-dir /forensics/recovered
```

## Chapter 5: Advanced Usage

### Performance Optimization

#### Large Filesystem Recovery

```bash
# For filesystems > 100GB, use block range scanning
sudo ext3grep /dev/sda1 --restore-all --output-dir /recovery \
  --block-range 0-1000000

# Process in chunks
for i in {0..10}; do
    START=$((i * 1000000))
    END=$(((i + 1) * 1000000))
    sudo ext3grep /dev/sda1 --restore-all --output-dir "/recovery/chunk_$i" \
      --block-range $START-$END
done
```

#### Parallel Recovery

```bash
# Split recovery across multiple processes
# Find total blocks
TOTAL_BLOCKS=$(sudo ext3grep /dev/sda1 --superblock | grep "Block count" | awk '{print $3}')
CHUNK=$((TOTAL_BLOCKS / 4))

# Run 4 parallel recoveries
for i in {0..3}; do
    START=$((i * CHUNK))
    END=$(((i + 1) * CHUNK))
    sudo ext3grep /dev/sda1 --restore-all --output-dir "/recovery/part_$i" \
      --block-range $START-$END &
done
wait
```

### Integration in Pentest Workflow

#### Digital Forensics Pipeline

```bash
#!/bin/bash
# forensics_pipeline.sh - Complete forensic analysis pipeline

EVIDENCE=$1
CASE_ID=$(date +%Y%m%d_%H%M%S)

echo "=== Forensic Analysis Pipeline ==="
echo "Evidence: $EVIDENCE"
echo "Case ID: $CASE_ID"

# Create working directory
WORK_DIR="/forensics/cases/$CASE_ID"
mkdir -p "$WORK_DIR"/{images,recovered,reports,logs}

# Step 1: Create forensic image
echo "[1/6] Creating forensic image..."
sudo dd if="$EVIDENCE" of="$WORK_DIR/images/evidence.img" bs=4M status=progress
echo "$(md5sum "$EVIDENCE")" > "$WORK_DIR/reports/original_md5.txt"
echo "$(md5sum "$WORK_DIR/images/evidence.img")" > "$WORK_DIR/reports/image_md5.txt"

# Step 2: Identify filesystem type
echo "[2/6] Identifying filesystem..."
FS_TYPE=$(sudo blkid "$EVIDENCE" | grep -o 'TYPE="[^"]*"' | cut -d'"' -f2)
echo "Filesystem type: $FS_TYPE"

# Step 3: Run appropriate recovery tool
echo "[3/6] Running recovery tools..."
if [ "$FS_TYPE" = "ext3" ]; then
    sudo ext3grep "$WORK_DIR/images/evidence.img" --restore-all \
        --output-dir "$WORK_DIR/recovered/ext3grep"
elif [ "$FS_TYPE" = "ext4" ]; then
    sudo ext4magic "$WORK_DIR/images/evidence.img" -f / \
        -o "$WORK_DIR/recovered/ext4magic"
fi

# Also run foremost for additional carving
sudo foremost -t all -i "$WORK_DIR/images/evidence.img" \
    -o "$WORK_DIR/recovered/foremost"

# Step 4: Analyze results
echo "[4/6] Analyzing recovered files..."
find "$WORK_DIR/recovered" -type f -exec file {} \; > "$WORK_DIR/reports/file_types.txt"

# Step 5: Generate report
echo "[5/6] Generating report..."
cat > "$WORK_DIR/reports/summary.txt" << EOF
Forensic Analysis Report
========================
Case ID: $CASE_ID
Evidence: $EVIDENCE
Analysis Date: $(date)
Filesystem: $FS_TYPE

Recovery Results:
- ext3grep: $(find "$WORK_DIR/recovered/ext3grep" -type f 2>/dev/null | wc -l) files
- foremost: $(find "$WORK_DIR/recovered/foremost" -type f 2>/dev/null | wc -l) files

Total recovered: $(find "$WORK_DIR/recovered" -type f | wc -l) files
Total size: $(du -sh "$WORK_DIR/recovered" | cut -f1)
EOF

echo "[6/6] Pipeline complete."
echo "Results: $WORK_DIR"
```

### Custom Configuration

#### Creating Recovery Profiles

```bash
# Create a profile for different scenarios
cat > /etc/ext3grep/recovery_profiles/incident_response.sh << 'EOF'
#!/bin/bash
# Incident Response Profile
DEVICE=$1
OUTPUT=$2

# Recover all deleted files
sudo ext3grep "$DEVICE" --restore-all --output-dir "$OUTPUT/all_deleted"

# Recover files from last 24 hours
AFTER=$(date -d "24 hours ago" +%s)
sudo ext3grep "$DEVICE" --restore-all --after "$AFTER" \
    --output-dir "$OUTPUT/last_24h"

# List all deleted inodes with details
sudo ext3grep "$DEVICE" --list-deleted-inodes --details \
    > "$OUTPUT/deleted_inodes.txt"

# Export journal for analysis
sudo ext3grep "$DEVICE" --show-journal > "$OUTPUT/journal_dump.txt"
EOF
chmod +x /etc/ext3grep/recovery_profiles/incident_response.sh
```

## Chapter 6: Real-World Scenarios

### Scenario 1: CTF Challenge - "Deleted Secrets"

**Context**: You've acquired a disk image from a CTF challenge. The description says "The admin deleted some important files before shutting down. Can you recover them?"

**Step 1: Initial Reconnaissance**

```bash
# Identify the filesystem
file challenge.img
# Output: Linux rev 1.0 ext3 filesystem

# Mount read-only to examine
mkdir /tmp/mnt
sudo mount -o loop,ro challenge.img /tmp/mnt

# Check what's currently on the filesystem
ls -la /tmp/mnt/
# Only see a few files, none look like flags

sudo umount /tmp/mnt
```

**Step 2: Run ext3grep**

```bash
# List all deleted inodes
sudo ext3grep challenge.img --list-deleted-inodes --details

# Output shows several deleted files including:
# 12345  REG  1024  8  2024-01-15 14:32:00  flag.txt
# 12346  REG  2048  16  2024-01-15 14:32:15  secrets/credentials.db
# 12347  DIR  4096  8  2024-01-15 14:31:45  secrets/

# Recover the flag file
sudo ext3grep challenge.img --restore-file flag.txt --output-dir /tmp/recovered

# Check the recovered file
cat /tmp/recovered/RESTORED_FILES/flag.txt
# Output: CTF{r3c0v3r3d_f0rm_j0urn4l_12345}
```

**Step 3: Advanced Recovery**

```bash
# The credentials database might contain more flags
sudo ext3grep challenge.img --restore-file secrets/credentials.db \
    --output-dir /tmp/recovered

# Analyze the database
sqlite3 /tmp/recovered/RESTORED_FILES/secrets/credentials.db ".dump"
# Contains additional flags or hints
```

### Scenario 2: Incident Response - Ransomware Attack

**Context**: A company was hit by ransomware. The attacker deleted original files after encrypting them. You need to recover the original files from the ext3 filesystem.

**Step 1: Evidence Preservation**

```bash
# NEVER work on original evidence
sudo dd if=/dev/sdb1 of=/forensics/evidence/sdb1.img bs=4M status=progress

# Verify integrity
echo "Original: $(sudo md5sum /dev/sdb1 | awk '{print $1}')"
echo "Copy: $(md5sum /forensics/evidence/sdb1.img | awk '{print $1}')"
```

**Step 2: Analysis**

```bash
# Determine when the attack occurred (from logs)
# Attack timestamp: 2024-01-15 03:00:00 UTC = 1705284000

# List files deleted around attack time
sudo ext3grep /forensics/evidence/sdb1.img --list-deleted-inodes --details \
    | grep -A5 "Jan 15"

# Recover files from before the attack
AFTER=$(date -d "2024-01-14" +%s)
BEFORE=$(date -d "2024-01-15 03:00:00" +%s)

sudo ext3grep /forensics/evidence/sdb1.img --restore-all \
    --after $AFTER --before $BEFORE \
    --output-dir /forensics/recovered/pre_attack
```

**Step 3: Validation**

```bash
# Verify recovered files
find /forensics/recovered/pre_attack -type f -exec file {} \;

# Check for integrity
md5sum /forensics/recovered/pre_attack/*.docx > recovered_md5.txt

# Compare with backup records if available
```

### Scenario 3: Digital Forensics - Employee Termination

**Context**: An employee was terminated for data exfiltration. They deleted sensitive company files before leaving. Recover evidence for legal proceedings.

**Step 1: Chain of Custody**

```bash
# Document everything
echo "Evidence Collection Log" > /forensics/chain_of_custody.txt
echo "Date: $(date)" >> /forensics/chain_of_custody.txt
echo "Collector: Forensic Analyst" >> /forensics/chain_of_custody.txt
echo "Source: /dev/sdc1 (employee workstation)" >> /forensics/chain_of_custody.txt

# Create forensic image
sudo dd if=/dev/sdc1 of=/forensics/evidence/sdc1.img bs=4M status=progress
echo "Image MD5: $(md5sum /forensics/evidence/sdc1.img)" >> /forensics/chain_of_custody.txt
```

**Step 2: Comprehensive Recovery**

```bash
# Recover all deleted files
sudo ext3grep /forensics/evidence/sdc1.img --restore-all \
    --output-dir /forensics/recovered/all

# Focus on company documents
sudo ext3grep /forensics/evidence/sdc1.img --restore-file "*.docx" \
    --output-dir /forensics/recovered/documents
sudo ext3grep /forensics/evidence/sdc1.img --restore-file "*.xlsx" \
    --output-dir /forensics/recovered/spreadsheets
sudo ext3grep /forensics/evidence/sdc1.img --restore-file "*.pdf" \
    --output-dir /forensics/recovered/pdfs

# Generate detailed report
sudo ext3grep /forensics/evidence/sdc1.img --list-deleted-inodes --details \
    > /forensics/reports/deleted_files_detail.txt
```

**Step 3: Analysis for Legal Team**

```bash
# Create summary report
cat > /forensics/reports/executive_summary.txt << EOF
Digital Forensics Report - Employee Termination Case
=====================================================
Date: $(date)
Evidence: /dev/sdc1 (employee workstation disk)

Recovery Summary:
- Total files recovered: $(find /forensics/recovered/all -type f | wc -l)
- Document files: $(find /forensics/recovered/documents -type f | wc -l)
- Spreadsheet files: $(find /forensics/recovered/spreadsheets -type f | wc -l)
- PDF files: $(find /forensics/recovered/pdfs -type f | wc -l)

Notable Recoveries:
- confidential_report_Q4.docx (deleted 2024-01-15 14:32:00)
- salary_data_2024.xlsx (deleted 2024-01-15 14:32:15)
- client_list_confidential.pdf (deleted 2024-01-15 14:32:30)

File hashes for evidence integrity:
$(find /forensics/recovered/all -type f -exec md5sum {} \;)
EOF
```

## Chapter 7: Detection & Defense

### How Defenders Detect ext3grep Usage

#### System Monitoring

```bash
# Monitor for ext3grep execution
auditctl -w /usr/bin/ext3grep -p x -k ext3grep_usage

# Check audit logs
ausearch -k ext3grep_usage

# Monitor process execution
ps aux | grep ext3grep
```

#### Log Analysis

```bash
# Check for ext3grep in system logs
grep -r "ext3grep" /var/log/syslog
grep -r "ext3grep" /var/log/auth.log

# Monitor disk access patterns
iotop -a -d 1 | grep ext3grep
```

#### Filesystem Indicators

```bash
# Check for RESTORED_FILES directories
find / -name "RESTORED_FILES" -type d 2>/dev/null

# Look for ext3grep output files
find / -name "*.img" -newer /var/log/syslog 2>/dev/null
```

### Mitigation Strategies

#### Prevent Unauthorized Recovery

```bash
# 1. Use ext4 with encryption
cryptsetup luksFormat /dev/sda1
cryptsetup open /dev/sda1 encrypted_volume
mkfs.ext4 /dev/mapper/encrypted_volume

# 2. Regularly overwrite free space
sudo shred -vfz -n 3 /dev/sda1

# 3. Disable journal (reduces recovery chances)
sudo tune2fs -O ^has_journal /dev/sda1

# 4. Mount with noexec
mount -o remount,noexec /dev/sda1
```

#### Detection Rules (YARA)

```yara
rule ext3grep_Usage {
    meta:
        description = "Detects ext3grep execution artifacts"
        author = "Security Team"
        date = "2024-01-15"
    
    strings:
        $s1 = "RESTORED_FILES" ascii
        $s2 = "ext3grep" ascii
        $s3 = "deleted inode" ascii
    
    condition:
        2 of ($s*)
}
```

### Blue Team Response

#### Incident Response Playbook

```bash
#!/bin/bash
# detect_recovery_tools.sh - Detect forensic tool usage

echo "=== Scanning for Recovery Tool Artifacts ==="

# Check for ext3grep artifacts
echo "[*] Checking for ext3grep artifacts..."
if find / -name "RESTORED_FILES" -type d 2>/dev/null | grep -q .; then
    echo "[!] ALERT: RESTORED_FILES directory found"
    find / -name "RESTORED_FILES" -type d 2>/dev/null
fi

# Check running processes
echo "[*] Checking for recovery tools in process list..."
if ps aux | grep -E "(ext3grep|ext4magic|extundelete|foremost)" | grep -v grep; then
    echo "[!] ALERT: Recovery tool currently running"
fi

# Check recent file modifications
echo "[*] Checking for recently modified filesystem images..."
find / -name "*.img" -mtime -1 -type f 2>/dev/null

# Check shell history
echo "[*] Checking shell history for recovery commands..."
grep -l "ext3grep\|ext4magic\|extundelete\|foremost" /home/*/.bash_history 2>/dev/null

echo "=== Scan Complete ==="
```

## Chapter 8: Troubleshooting

### Common Errors and Solutions

#### Error: "Not an ext3 filesystem"

```bash
# Problem: ext3grep returns "Not an ext3 filesystem"
# Cause: Filesystem is ext4 or another type

# Solution: Check filesystem type
sudo blkid /dev/sda1
# Output: /dev/sda1: UUID="..." TYPE="ext4"

# Use ext4magic or extundelete instead for ext4
sudo ext4magic /dev/sda1 -f / -o /recovery
```

#### Error: "Journal not found"

```bash
# Problem: ext3grep cannot find the journal
# Cause: Journal may be external or corrupted

# Solution 1: Specify journal block manually
sudo ext3grep /dev/sda1 --show-journal --journal-block 32768

# Solution 2: Check if journal exists
sudo dumpe2fs /dev/sda1 | grep -i journal
```

#### Error: "Permission denied"

```bash
# Problem: ext3grep requires root access
# Cause: Running as regular user

# Solution: Use sudo
sudo ext3grep /dev/sda1 --restore-all

# Or add to sudoers for specific user
echo "username ALL=(ALL) NOPASSWD: /usr/bin/ext3grep" | sudo tee /etc/sudoers.d/ext3grep
```

#### Error: "Device or resource busy"

```bash
# Problem: Cannot read device because it's mounted
# Cause: Filesystem is currently mounted

# Solution: Unmount first or use read-only mount
sudo umount /dev/sda1
# Or mount read-only
sudo mount -o remount,ro /dev/sda1
```

### Performance Issues

#### Slow Recovery on Large Filesystems

```bash
# Problem: Recovery takes too long on large filesystems
# Solution: Use block range scanning

# Find total blocks
BLOCKS=$(sudo dumpe2fs /dev/sda1 | grep "Block count" | awk '{print $NF}')

# Process in smaller chunks
CHUNK=100000
for ((i=0; i<BLOCKS; i+=CHUNK)); do
    END=$((i + CHUNK))
    sudo ext3grep /dev/sda1 --restore-all \
        --block-range $i-$END \
        --output-dir "/recovery/chunk_$i"
done
```

#### High Memory Usage

```bash
# Problem: ext3grep uses too much memory
# Solution: Process in batches

# Limit memory usage
ulimit -v 1048576  # Limit to 1GB virtual memory
sudo ext3grep /dev/sda1 --restore-all --output-dir /recovery
```

### FAQ

**Q1: Can ext3grep recover files from ext4 filesystems?**

A: No, ext3grep is designed specifically for ext3 filesystems. For ext4, use ext4magic or extundelete which support ext4 journal analysis.

**Q2: How successful is ext3grep at recovering deleted files?**

A: Success depends on several factors: how recently the file was deleted, whether the journal entries are still present, and whether the data blocks have been overwritten. For recently deleted files with intact journal entries, recovery success can be 90%+.

**Q3: Can ext3grep recover encrypted files?**

A: ext3grep can recover the encrypted file data, but it cannot decrypt it. You would need the encryption key to access the file contents.

**Q4: Is it safe to run ext3grep on a live filesystem?**

A: It's generally safe to READ from a live filesystem, but writing recovered files to the same filesystem could overwrite other deleted data. Always recover to a different device.

**Q5: How do I recover files deleted a long time ago?**

A: Recovery of old deletions is less likely because journal entries may have been overwritten. However, if the data blocks haven't been overwritten, ext3grep might still find them through block scanning.

**Q6: Can ext3grep recover files from a RAID array?**

A: ext3grep works on individual filesystems. For RAID arrays, you need to first assemble the RAID and then run ext3grep on the resulting logical volume.

## Resources

### Official Documentation

- ext3grep Manual Page: `man ext3grep`
- ext3grep GitHub: https://github.com/extundelete/extundelete
- Linux ext3 Documentation: https://www.kernel.org/doc/html/latest/filesystems/ext3.html

### Tutorials and Guides

- SANS Digital Forensics: File Carving Techniques
- Linux Forensics Workshop Materials
- Ext3 Filesystem Internals Documentation

### Related Tools

- **ext4magic**: Extended version supporting ext4 filesystems
- **extundelete**: Another ext3/ext4 recovery tool using journal
- **photorec**: General-purpose file carving tool
- **foremost**: File carving based on headers/footers
- **scalpel**: Another file carving tool
- **testdisk**: Partition recovery and file undeletion

### Practice Resources

- Digital Forensics Challenge Images
- CTF Forensics Challenges
- Practice ext3 filesystems with test data
- Volatility Foundation Memory Analysis Practice

### Books and Papers

- "Linux Forensics" by Phil Rhodes
- "File System Forensic Analysis" by Brian Carrier
- "The Art of Memory Forensics" by Michael Hale Ligh
