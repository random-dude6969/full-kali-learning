---
tool_name: extundelete
display_name: "Ext3/Ext4 File System Recovery"
mitre_tactic: "analysis"
mitre_tactic_id: "TA0007"
install_command: "sudo apt install extundelete"
github_repo: "https://github.com/extundelete/extundelete"
kali_tools_page: "https://www.kali.org/tools/extundelete/"
difficulty_level: "beginner"
requires_root: true
gui_available: false
tags:
  - forensics
  - file-carving
  - ext4
  - ext3
  - data-recovery
  - journal-analysis
  - linux
related_tools:
  - ext3grep
  - ext4magic
  - foremost
  - photorec
  - scalpel
---

# Extundelete - Ext3/Ext4 Filesystem Forensic Recovery Tool

## Chapter 1: Introduction & Overview

### What is extundelete?

extundelete is a command-line forensic recovery tool designed to recover deleted files from ext3 and ext4 filesystems. It works by analyzing the filesystem journal to identify deleted files and restore them to their original state or to a specified location. extundelete is known for its simplicity and effectiveness, making it a popular choice for both beginners and experienced forensic analysts.

### Historical Context

extundelete was developed as an open-source alternative to commercial file recovery tools. It builds upon the principles of journal-based file recovery pioneered by ext3grep but adds support for ext4 filesystems. The tool has become a standard utility in Linux forensics distributions like Kali Linux.

Key milestones:

- Initial release: 2008
- ext4 support added: 2009
- Included in Kali Linux: 2012
- Current stable version: 0.2.4

### When to Use extundelete

extundelete excels in scenarios where:

- The target filesystem is ext3 or ext4
- Files were deleted recently (journal entries still present)
- You need a simple, straightforward recovery process
- You want to recover files to their original paths
- You need to recover all deleted files at once
- The filesystem journal is intact

extundelete is NOT suitable when:

- The journal has been corrupted or overwritten
- The filesystem has been reformatted
- You need raw data carving without filesystem metadata (use foremost/photorec)
- The filesystem uses encryption (ext4crypt)
- You need advanced filtering options (use ext4magic)

### Key Concepts

**ext3/ext4 Journal (JBD/JBD2)**: Both ext3 and ext4 use journaling to maintain filesystem consistency. The journal records metadata changes before they're committed to the main filesystem. extundelete reads these journal entries to identify deleted files.

**Inode**: Each file has an inode containing metadata (permissions, timestamps, block pointers). When a file is deleted, its inode is marked as free but often retains useful information.

**Directory Entries**: Directory entries map filenames to inode numbers. extundelete can recover these from the journal.

**Block Pointers**: ext3/ext4 inodes store pointers to data blocks. extundelete can reconstruct these pointer chains from journal entries.

**Orphan Inodes**: ext4 maintains an orphan inode list for files that were open when deleted. extundelete can recover these files.

### Technical Architecture

extundelete operates in several modes:

1. **Filesystem Analysis Mode**: Scans the entire filesystem for deleted files
2. **Inode Recovery Mode**: Recovers specific files by inode number
3. **Directory Recovery Mode**: Recovers directory structures
4. **Restore-All Mode**: Recovers all deleted files at once

## Chapter 2: Installation & Setup

### System Requirements

- Linux operating system (Kali Linux recommended)
- ext3 or ext4 filesystem (physical or image file)
- Minimum 512 MB RAM (2 GB+ recommended for large images)
- Sufficient disk space for recovered files
- Root or sudo access

### Installation Methods

#### Method 1: Kali Linux Package Manager (Recommended)

```bash
sudo apt update
sudo apt install extundelete
```

#### Method 2: Debian/Ubuntu Package Manager

```bash
sudo apt update
sudo apt install extundelete
```

#### Method 3: Compile from Source

```bash
# Install build dependencies
sudo apt install build-essential libext2fs-dev

# Clone the repository
git clone https://github.com/extundelete/extundelete.git
cd extundelete

# Build
./configure
make

# Install
sudo make install
```

#### Method 4: Using Docker

```bash
# Pull the extundelete container
docker pull extundelete/extundelete

# Run with device access
docker run --privileged -v /dev:/dev -v /output:/output extundelete/extundelete /dev/sda1 -o /output
```

### Verification

After installation, verify extundelete is working:

```bash
# Check version
extundelete --version

# Expected output
# extundelete 0.2.4

# Check help
extundelete --help

# Quick test with a small ext4 image
dd if=/dev/zero of=/tmp/test.img bs=1M count=10
mkfs.ext4 /tmp/test.img
```

### Preparing a Test Environment

Create a safe test environment before working with real forensic images:

```bash
# Create a test ext4 filesystem
dd if=/dev/zero of=/tmp/test_ext4.img bs=1M count=100
mkfs.ext4 /tmp/test_ext4.img

# Mount it
mkdir /tmp/test_mount
sudo mount -o loop /tmp/test_ext4.img /tmp/test_mount

# Create some test files
echo "Test file 1" | sudo tee /tmp/test_mount/file1.txt
echo "Test file 2" | sudo tee /tmp/test_mount/file2.txt
dd if=/dev/urandom of=/tmp/test_mount/binary_file.bin bs=1K count=10
sudo mkdir /tmp/test_mount/documents
echo "Confidential report" | sudo tee /tmp/test_mount/documents/report.pdf

# Unmount and delete
sudo umount /tmp/test_mount
```

## Chapter 3: Basic Usage

### Essential Commands

#### Recover All Deleted Files

```bash
# Basic recovery - scan entire filesystem
sudo extundelete /dev/sda1 --restore-all

# Recover from an image file
sudo extundelete /path/to/filesystem.img --restore-all
```

**Sample Output:**

```
NOTICE: Extended attributes are not enabled.
The directory size of the journal inode is 32768.
Loading journal descriptors...
The journal contained 12847 descriptors.
Journal descriptor block 32768 is at 134217728.
Found 47 deleted inode entries
Found 23 deleted directory entries

Restoring file1.txt
Restoring file2.txt
Restoring documents/report.pdf
Restoring binary_file.bin

Recovery complete:
- Files recovered: 4
- Directories recovered: 1
```

#### Recover Specific File by Name

```bash
# Recover a specific file
sudo extundelete /dev/sda1 --restore-file path/to/deleted/file.txt

# Recover file from specific directory
sudo extundelete /dev/sda1 --restore-file documents/secret.pdf
```

#### Recover Specific Inode

```bash
# Recover by inode number
sudo extundelete /dev/sda1 --restore-inode 12345

# Recover multiple inodes
sudo extundelete /dev/sda1 --restore-inode 12345,12346,12347
```

#### List Deleted Files

```bash
# List all deleted inodes
sudo extundelete /dev/sda1 --list-inodes deleted

# List all inodes (including deleted)
sudo extundelete /dev/sda1 --list-inodes all

# List specific inode
sudo extundelete /dev/sda1 --list-inodes 12345
```

**Sample Output:**

```
NOTICE: Extended attributes are not enabled.
Inode  Type  Size    Blocks  Deleted Time          Path
12345  REG   1024    8       2024-01-15 14:32:00  file1.txt
12346  REG   2048    16      2024-01-15 14:32:15  file2.txt
12347  DIR   4096    8       2024-01-15 14:31:45  documents/
12348  REG   512     8       2024-01-15 13:45:00  binary_file.bin
12349  REG   8192    32      2024-01-15 14:33:00  documents/report.pdf
```

### Complete Flag Reference

| Flag | Description | Example |
|------|-------------|---------|
| `--restore-all` | Recover all deleted files | `sudo extundelete /dev/sda1 --restore-all` |
| `--restore-file PATH` | Recover specific file | `sudo extundelete /dev/sda1 --restore-file file.txt` |
| `--restore-inode INODE` | Recover specific inode | `sudo extundelete /dev/sda1 --restore-inode 12345` |
| `--output-dir DIR` | Set output directory | `sudo extundelete /dev/sda1 --restore-all --output-dir /recovery` |
| `--list-inodes TYPE` | List inodes (deleted/all) | `sudo extundelete /dev/sda1 --list-inodes deleted` |
| `--after TIMESTAMP` | Recover files after time | `sudo extundelete /dev/sda1 --restore-all --after 1705312800` |
| `--before TIMESTAMP` | Recover files before time | `sudo extundelete /dev/sda1 --restore-all --before 1705316400` |
| `--no-dir-links` | Don't recover dir links | `sudo extundelete /dev/sda1 --restore-all --no-dir-links` |
| `--no-links` | Don't recover links | `sudo extundelete /dev/sda1 --restore-all --no-links` |
| `--version` | Show version | `extundelete --version` |
| `--help` | Show help | `extundelete --help` |

### Basic Recovery Scenarios

#### Scenario 1: Recover Recently Deleted File

```bash
# Step 1: Identify the partition
sudo fdisk -l /dev/sda
# Output shows /dev/sda1 is ext4

# Step 2: List deleted files
sudo extundelete /dev/sda1 --list-inodes deleted

# Step 3: Recover specific file
sudo extundelete /dev/sda1 --restore-file important_document.docx

# Step 4: Check recovered files
ls -la RECOVERED_FILES/
```

#### Scenario 2: Recover All Files from Time Window

```bash
# Files deleted between 2024-01-15 14:00 and 15:00
# Convert to Unix timestamps:
# 2024-01-15 14:00:00 = 1705312800
# 2024-01-15 15:00:00 = 1705316400

sudo extundelete /dev/sda1 --restore-all \
  --after 1705312800 \
  --before 1705316400 \
  --output-dir /forensics/recovered
```

## Chapter 4: Intermediate Usage

### Advanced Journal Analysis

#### Understanding Journal Descriptors

```bash
# List journal descriptors
sudo extundelete /dev/sda1 --list-inodes deleted 2>&1 | head -20

# Analyze journal structure
sudo dumpe2fs /dev/sda1 | grep -i journal
```

#### Journal Recovery Techniques

```bash
# If journal is corrupted, try with --no-dir-links
sudo extundelete /dev/sda1 --restore-all --no-dir-links

# For partial journal recovery
sudo extundelete /dev/sda1 --restore-all --no-links
```

### Scripting with extundelete

#### Bash Script: Automated Recovery

```bash
#!/bin/bash
# ext_undelete_auto.sh - Automated extundelete recovery script

DEVICE=$1
OUTPUT_DIR=$2
TIMESTAMP_AFTER=$3
TIMESTAMP_BEFORE=$4

if [ -z "$DEVICE" ] || [ -z "$OUTPUT_DIR" ]; then
    echo "Usage: $0 <device> <output_dir> [after_timestamp] [before_timestamp]"
    echo "Example: $0 /dev/sda1 /recovery 1705312800 1705316400"
    exit 1
fi

# Create output directory
mkdir -p "$OUTPUT_DIR"

echo "[*] Starting recovery on $DEVICE"
echo "[*] Output directory: $OUTPUT_DIR"

# Step 1: List deleted inodes
echo "[*] Listing deleted inodes..."
sudo extundelete "$DEVICE" --list-inodes deleted > "$OUTPUT_DIR/deleted_inodes.txt"

# Step 2: Recover files with optional time filter
echo "[*] Recovering files..."
if [ -n "$TIMESTAMP_AFTER" ] && [ -n "$TIMESTAMP_BEFORE" ]; then
    sudo extundelete "$DEVICE" --restore-all \
        --after "$TIMESTAMP_AFTER" \
        --before "$TIMESTAMP_BEFORE" \
        --output-dir "$OUTPUT_DIR/files"
else
    sudo extundelete "$DEVICE" --restore-all \
        --output-dir "$OUTPUT_DIR/files"
fi

# Step 3: Generate summary
echo "[*] Generating summary..."
cat > "$OUTPUT_DIR/recovery_summary.txt" << EOF
Recovery Summary
================
Device: $DEVICE
Date filter: ${TIMESTAMP_AFTER:-all} to ${TIMESTAMP_BEFORE:-now}
Files recovered: $(find "$OUTPUT_DIR/files" -type f 2>/dev/null | wc -l)
Total size: $(du -sh "$OUTPUT_DIR/files" 2>/dev/null | cut -f1)
EOF

echo "[+] Recovery complete. Summary: $OUTPUT_DIR/recovery_summary.txt"
```

#### Python Script: Advanced Analysis

```python
#!/usr/bin/env python3
"""
ext_undelete_analyzer.py - Python wrapper for extundelete analysis
"""

import subprocess
import json
import re
import sys
from datetime import datetime
from pathlib import Path

class ExtUndeleteAnalyzer:
    def __init__(self, device):
        self.device = device
        self.deleted_inodes = []
        self.recovered_files = []
    
    def list_deleted_inodes(self):
        """Parse deleted inodes from extundelete output"""
        cmd = ['sudo', 'extundelete', self.device, 
               '--list-inodes', 'deleted']
        result = subprocess.run(cmd, capture_output=True, text=True)
        
        for line in result.stdout.split('\n'):
            if line.strip() and not line.startswith('NOTICE'):
                parts = line.split()
                if len(parts) >= 5 and parts[0].isdigit():
                    inode_info = {
                        'inode': int(parts[0]),
                        'type': parts[1],
                        'size': int(parts[2]),
                        'blocks': int(parts[3]),
                        'deleted_time': ' '.join(parts[4:-1]),
                        'path': parts[-1]
                    }
                    self.deleted_inodes.append(inode_info)
        
        return self.deleted_inodes
    
    def recover_file(self, filepath, output_dir):
        """Recover a specific file"""
        cmd = ['sudo', 'extundelete', self.device, 
               '--restore-file', filepath,
               '--output-dir', output_dir]
        result = subprocess.run(cmd, capture_output=True, text=True)
        return result.returncode == 0
    
    def recover_by_time(self, after, before, output_dir):
        """Recover files within a time window"""
        cmd = ['sudo', 'extundelete', self.device, 
               '--restore-all',
               '--after', str(after),
               '--before', str(before),
               '--output-dir', output_dir]
        result = subprocess.run(cmd, capture_output=True, text=True)
        return result.returncode == 0
    
    def recover_all(self, output_dir):
        """Recover all deleted files"""
        cmd = ['sudo', 'extundelete', self.device, 
               '--restore-all',
               '--output-dir', output_dir]
        result = subprocess.run(cmd, capture_output=True, text=True)
        return result.returncode == 0
    
    def generate_report(self, output_file):
        """Generate JSON report of findings"""
        report = {
            'device': self.device,
            'timestamp': datetime.now().isoformat(),
            'deleted_inodes': self.deleted_inodes
        }
        
        with open(output_file, 'w') as f:
            json.dump(report, f, indent=2)
        
        return report

def main():
    if len(sys.argv) < 2:
        print(f"Usage: {sys.argv[0]} <device>")
        sys.exit(1)
    
    device = sys.argv[1]
    analyzer = ExtUndeleteAnalyzer(device)
    
    print(f"[*] Analyzing {device}...")
    
    # List deleted inodes
    print("[*] Listing deleted inodes...")
    inodes = analyzer.list_deleted_inodes()
    print(f"[+] Found {len(inodes)} deleted inodes")
    
    # Generate report
    report_file = f"extundelete_analysis_{datetime.now().strftime('%Y%m%d_%H%M%S')}.json"
    analyzer.generate_report(report_file)
    print(f"[+] Report saved to {report_file}")

if __name__ == '__main__':
    main()
```

### Combining with Other Tools

#### Workflow: extundelete + foremost

```bash
# Step 1: Use extundelete for journal-based recovery
sudo extundelete /dev/sda1 --restore-all --output-dir /recovery/extundelete

# Step 2: Use foremost for additional carving on unallocated blocks
sudo foremost -t all -i /dev/sda1 -o /recovery/foremost

# Step 3: Compare results
echo "extundelete recovered: $(find /recovery/extundelete -type f 2>/dev/null | wc -l) files"
echo "foremost recovered: $(find /recovery/foremost -type f 2>/dev/null | wc -l) files"
```

#### Workflow: extundelete + photorec

```bash
# Create forensic image first
sudo dd if=/dev/sda1 of=/forensics/evidence/sda1.img bs=4M status=progress

# Use extundelete on image
sudo extundelete /forensics/evidence/sda1.img --restore-all \
    --output-dir /recovery/extundelete

# Use photorec on same image for comparison
sudo photorec /d /recovery/photorec /cmd /forensics/evidence/sda1.img search
```

#### Workflow: extundelete + ext4magic

```bash
# Use both tools for comprehensive recovery
sudo extundelete /dev/sda1 --restore-all --output-dir /recovery/extundelete
sudo ext4magic /dev/sda1 -r -o /recovery/ext4magic

# Compare results
echo "extundelete recovered: $(find /recovery/extundelete -type f 2>/dev/null | wc -l) files"
echo "ext4magic recovered: $(find /recovery/ext4magic -type f 2>/dev/null | wc -l) files"

# Merge unique files
find /recovery/extundelete /recovery/ext4magic -type f -exec md5sum {} \; | \
    sort | uniq -w32 -d > /recovery/duplicates.txt
```

## Chapter 5: Advanced Usage

### Performance Optimization

#### Large Filesystem Recovery

```bash
# For large filesystems, use time-based filtering
sudo extundelete /dev/sda1 --restore-all \
    --after 1705312800 \
    --before 1705316400 \
    --output-dir /recovery/time_filtered

# Recover specific directories only
sudo extundelete /dev/sda1 --restore-file "home/user/documents" \
    --output-dir /recovery/documents
```

#### Parallel Recovery

```bash
# Recover different directories in parallel
sudo extundelete /dev/sda1 --restore-file "home/user" \
    --output-dir /recovery/user &
sudo extundelete /dev/sda1 --restore-file "var/log" \
    --output-dir /recovery/logs &
sudo extundelete /dev/sda1 --restore-file "etc" \
    --output-dir /recovery/etc &
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

# Step 3: Run recovery tool based on filesystem
echo "[3/6] Running recovery tools..."
case "$FS_TYPE" in
    ext3|ext4)
        sudo extundelete "$WORK_DIR/images/evidence.img" --restore-all \
            --output-dir "$WORK_DIR/recovered/extundelete"
        ;;
    *)
        echo "Unsupported filesystem: $FS_TYPE"
        ;;
esac

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
- extundelete: $(find "$WORK_DIR/recovered/extundelete" -type f 2>/dev/null | wc -l) files
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
cat > /etc/extundelete/recovery_profiles/incident_response.sh << 'EOF'
#!/bin/bash
# Incident Response Profile
DEVICE=$1
OUTPUT=$2

# Recover all deleted files
sudo extundelete "$DEVICE" --restore-all --output-dir "$OUTPUT/all_deleted"

# Recover files from last 24 hours
AFTER=$(date -d "24 hours ago" +%s)
sudo extundelete "$DEVICE" --restore-all --after "$AFTER" \
    --output-dir "$OUTPUT/last_24h"

# List all deleted inodes with details
sudo extundelete "$DEVICE" --list-inodes deleted > "$OUTPUT/deleted_inodes.txt"
EOF
chmod +x /etc/extundelete/recovery_profiles/incident_response.sh
```

## Chapter 6: Real-World Scenarios

### Scenario 1: CTF Challenge - "Recover the Flag"

**Context**: You've acquired a disk image from a CTF challenge. The description says "The admin deleted the flag file. Can you recover it?"

**Step 1: Initial Reconnaissance**

```bash
# Identify the filesystem
file challenge.img
# Output: Linux rev 1.0 ext4 filesystem

# Mount read-only to examine
mkdir /tmp/mnt
sudo mount -o loop,ro challenge.img /tmp/mnt

# Check what's currently on the filesystem
ls -la /tmp/mnt/
# Only see a few files, none look like flags

sudo umount /tmp/mnt
```

**Step 2: Run extundelete**

```bash
# List all deleted inodes
sudo extundelete challenge.img --list-inodes deleted

# Output shows several deleted files including:
# 12345  REG  1024  8  2024-01-15 14:32:00  flag.txt
# 12346  REG  2048  16  2024-01-15 14:32:15  hints.txt
# 12347  DIR  4096  8  2024-01-15 14:31:45  hidden/

# Recover the flag file
sudo extundelete challenge.img --restore-file flag.txt --output-dir /tmp/recovered

# Check the recovered file
cat /tmp/recovered/RECOVERED_FILES/flag.txt
# Output: CTF{3xtund3l3t3_r3c0v3ry_m4st3r}
```

**Step 3: Advanced Recovery**

```bash
# The hidden directory might contain more flags
sudo extundelete challenge.img --restore-file "hidden/secret.txt" \
    --output-dir /tmp/recovered

# Check the recovered file
cat /tmp/recovered/RECOVERED_FILES/hidden/secret.txt
# Output: CTF{j0urn4l_b4s3d_r3c0v3ry_pr0}
```

### Scenario 2: Incident Response - Ransomware Attack

**Context**: A company was hit by ransomware. The attacker deleted original files after encrypting them. You need to recover the original files.

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
sudo extundelete /forensics/evidence/sdb1.img --list-inodes deleted

# Recover files from before the attack
AFTER=$(date -d "2024-01-14" +%s)
BEFORE=$(date -d "2024-01-15 03:00:00" +%s)

sudo extundelete /forensics/evidence/sdb1.img --restore-all \
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
sudo extundelete /forensics/evidence/sdc1.img --restore-all \
    --output-dir /forensics/recovered/all

# Focus on company documents
sudo extundelete /forensics/evidence/sdc1.img --restore-file "*.docx" \
    --output-dir /forensics/recovered/documents
sudo extundelete /forensics/evidence/sdc1.img --restore-file "*.xlsx" \
    --output-dir /forensics/recovered/spreadsheets
sudo extundelete /forensics/evidence/sdc1.img --restore-file "*.pdf" \
    --output-dir /forensics/recovered/pdfs

# Generate detailed report
sudo extundelete /forensics/evidence/sdc1.img --list-inodes deleted \
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

### How Defenders Detect extundelete Usage

#### System Monitoring

```bash
# Monitor for extundelete execution
auditctl -w /usr/bin/extundelete -p x -k extundelete_usage

# Check audit logs
ausearch -k extundelete_usage

# Monitor process execution
ps aux | grep extundelete
```

#### Log Analysis

```bash
# Check for extundelete in system logs
grep -r "extundelete" /var/log/syslog
grep -r "extundelete" /var/log/auth.log

# Monitor disk access patterns
iotop -a -d 1 | grep extundelete
```

#### Filesystem Indicators

```bash
# Check for RECOVERED_FILES directories
find / -name "RECOVERED_FILES" -type d 2>/dev/null

# Look for extundelete output files
find / -name "*.img" -newer /var/log/syslog 2>/dev/null
```

### Mitigation Strategies

#### Prevent Unauthorized Recovery

```bash
# 1. Use encryption
cryptsetup luksFormat /dev/sda1
cryptsetup open /dev/sda1 encrypted_volume
mkfs.ext4 /dev/mapper/encrypted_volume

# 2. Regularly overwrite free space
sudo shred -vfz -n 3 /dev/sda1

# 3. Disable journal (reduces recovery chances)
sudo tune2fs -O ^has_journal /dev/sda1

# 4. Use secure deletion
sudo shred -vfz -n 3 /path/to/sensitive/file
```

#### Detection Rules (YARA)

```yara
rule extundelete_Usage {
    meta:
        description = "Detects extundelete execution artifacts"
        author = "Security Team"
        date = "2024-01-15"
    
    strings:
        $s1 = "RECOVERED_FILES" ascii
        $s2 = "extundelete" ascii
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

# Check for extundelete artifacts
echo "[*] Checking for extundelete artifacts..."
if find / -name "RECOVERED_FILES" -type d 2>/dev/null | grep -q .; then
    echo "[!] ALERT: RECOVERED_FILES directory found"
    find / -name "RECOVERED_FILES" -type d 2>/dev/null
fi

# Check running processes
echo "[*] Checking for recovery tools in process list..."
if ps aux | grep -E "(extundelete|ext3grep|ext4magic|foremost)" | grep -v grep; then
    echo "[!] ALERT: Recovery tool currently running"
fi

# Check recent file modifications
echo "[*] Checking for recently modified filesystem images..."
find / -name "*.img" -mtime -1 -type f 2>/dev/null

# Check shell history
echo "[*] Checking shell history for recovery commands..."
grep -l "extundelete\|ext3grep\|ext4magic\|foremost" /home/*/.bash_history 2>/dev/null

echo "=== Scan Complete ==="
```

## Chapter 8: Troubleshooting

### Common Errors and Solutions

#### Error: "Not an ext3/ext4 filesystem"

```bash
# Problem: extundelete returns "Not an ext3/ext4 filesystem"
# Cause: Filesystem is not ext3/ext4

# Solution: Check filesystem type
sudo blkid /dev/sda1
# Output: /dev/sda1: UUID="..." TYPE="ext4"

# Ensure you're using the correct device
lsblk
```

#### Error: "Journal not found"

```bash
# Problem: extundelete cannot find the journal
# Cause: Journal may be external or corrupted

# Solution 1: Check journal location
sudo dumpe2fs /dev/sda1 | grep -i journal

# Solution 2: Try with --no-dir-links
sudo extundelete /dev/sda1 --restore-all --no-dir-links
```

#### Error: "Permission denied"

```bash
# Problem: extundelete requires root access
# Cause: Running as regular user

# Solution: Use sudo
sudo extundelete /dev/sda1 --restore-all

# Or add to sudoers for specific user
echo "username ALL=(ALL) NOPASSWD: /usr/bin/extundelete" | sudo tee /etc/sudoers.d/extundelete
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
# Solution: Use time-based filtering

# Recover only recent files
AFTER=$(date -d "24 hours ago" +%s)
sudo extundelete /dev/sda1 --restore-all --after $AFTER --output-dir /recovery/recent

# Or recover specific directories
sudo extundelete /dev/sda1 --restore-file "home/user" --output-dir /recovery/user
```

#### High Memory Usage

```bash
# Problem: extundelete uses too much memory
# Solution: Process in batches

# Recover specific file types
sudo extundelete /dev/sda1 --restore-file "*.pdf" --output-dir /recovery/pdfs
sudo extundelete /dev/sda1 --restore-file "*.docx" --output-dir /recovery/docs
```

### FAQ

**Q1: Can extundelete recover files from ext4 filesystems?**

A: Yes, extundelete supports both ext3 and ext4 filesystems.

**Q2: How successful is extundelete at recovering deleted files?**

A: Success depends on several factors: how recently the file was deleted, whether the journal entries are still present, and whether the data blocks have been overwritten. For recently deleted files with intact journal entries, recovery success can be 90%+.

**Q3: Can extundelete recover encrypted files?**

A: extundelete can recover the encrypted file data, but it cannot decrypt it. You would need the encryption key to access the file contents.

**Q4: Is it safe to run extundelete on a live filesystem?**

A: It's generally safe to READ from a live filesystem, but writing recovered files to the same filesystem could overwrite other deleted data. Always recover to a different device.

**Q5: How do I recover files deleted a long time ago?**

A: Recovery of old deletions is less likely because journal entries may have been overwritten. However, if the data blocks haven't been overwritten, extundelete might still find them through inode scanning.

**Q6: Can extundelete recover files from a RAID array?**

A: extundelete works on individual filesystems. For RAID arrays, you need to first assemble the RAID and then run extundelete on the resulting logical volume.

## Resources

### Official Documentation

- extundelete Manual Page: `man extundelete`
- extundelete GitHub: https://github.com/extundelete/extundelete
- Linux ext3/ext4 Documentation: https://www.kernel.org/doc/html/latest/filesystems/

### Tutorials and Guides

- SANS Digital Forensics: File Carving Techniques
- Linux Forensics Workshop Materials
- Ext3/Ext4 Filesystem Internals Documentation

### Related Tools

- **ext3grep**: Original ext3 recovery tool
- **ext4magic**: Extended version supporting ext4 filesystems
- **photorec**: General-purpose file carving tool
- **foremost**: File carving based on headers/footers
- **scalpel**: Another file carving tool
- **testdisk**: Partition recovery and file undeletion

### Practice Resources

- Digital Forensics Challenge Images
- CTF Forensics Challenges
- Practice ext3/ext4 filesystems with test data
- Volatility Foundation Memory Analysis Practice

### Books and Papers

- "Linux Forensics" by Phil Rhodes
- "File System Forensic Analysis" by Brian Carrier
- "The Art of Memory Forensics" by Michael Hale Ligh
