---
tool_name: ext4magic
display_name: "Ext4 File System Recovery"
mitre_tactic: "analysis"
mitre_tactic_id: "TA0007"
install_command: "sudo apt install ext4magic"
github_repo: "https://github.com/log204/ext4magic"
kali_tools_page: "https://www.kali.org/tools/ext4magic/"
difficulty_level: "intermediate"
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
  - extundelete
  - foremost
  - photorec
  - scalpel
---

# Ext4magic - Ext3/Ext4 Filesystem Forensic Recovery Tool

## Chapter 1: Introduction & Overview

### What is ext4magic?

ext4magic is a command-line forensic recovery tool designed to recover deleted files from both ext3 and ext4 filesystems. It is an enhanced fork of ext3grep that extends support to ext4 filesystems, which are now the default on most modern Linux distributions. ext4magic leverages the filesystem journal (JBD/JBD2) to recover deleted files with high accuracy by analyzing metadata changes recorded in the journal.

### Historical Context

When ext4 became the default filesystem in Linux distributions around 2008-2010, the forensic community faced a gap: ext3grep couldn't handle ext4 filesystems. ext4magic was developed to address this limitation while maintaining the proven journal-based recovery approach. The tool has evolved to support both ext3 and ext4, making it versatile for modern forensic investigations.

ext4magic builds upon the principles established by ext3grep but adds:

- Full ext4 journal (JBD2) support
- Improved time-based recovery
- Better handling of extents (ext4's block allocation method)
- Enhanced file type detection
- Support for larger filesystems

### When to Use ext4magic

ext4magic excels in scenarios where:

- The target filesystem is ext4 (most modern Linux systems)
- Files were deleted recently (journal entries still present)
- You need to recover files by name, path, or inode
- You need time-based filtering for targeted recovery
- The filesystem journal is intact
- You need to recover both ext3 and ext4 filesystems with one tool

ext4magic is NOT suitable when:

- The journal has been corrupted or overwritten
- The filesystem has been reformatted
- You need raw data carving without filesystem metadata (use foremost/photorec)
- The filesystem uses encryption (ext4crypt)

### Key Concepts

**ext4 Journal (JBD2)**: ext4 uses JBD2 (Journaling Block Device 2) for journaling. It records metadata changes before they're committed to the main filesystem, allowing recovery of deleted file information.

**Extents**: ext4 uses extents instead of traditional block pointers. An extent is a contiguous range of blocks, which improves performance and reduces metadata overhead. ext4magic can parse extent information from journal entries.

**Inode**: Each file has an inode containing metadata (permissions, timestamps, block/extent pointers). ext4magic recovers inode information from the journal.

**Directory Entries**: ext4 uses htree-indexed directories for performance. ext4magic can recover deleted directory entries from journal records.

**Orphan Inodes**: ext4 maintains an orphan inode list for files that were open when deleted. ext4magic can recover these files.

### Technical Architecture

ext4magic operates in several modes:

1. **Filesystem Analysis Mode**: Scans the entire filesystem for deleted files
2. **Time-Based Recovery Mode**: Recovers files deleted within a specific time window
3. **Inode Recovery Mode**: Recovers specific files by inode number
4. **Directory Recovery Mode**: Recovers directory structures
5. **Journal Analysis Mode**: Dumps and analyzes journal contents

## Chapter 2: Installation & Setup

### System Requirements

- Linux operating system (Kali Linux recommended)
- ext3 or ext4 filesystem (physical or image file)
- Minimum 1 GB RAM (4 GB+ recommended for large images)
- Sufficient disk space for recovered files
- Root or sudo access

### Installation Methods

#### Method 1: Kali Linux Package Manager (Recommended)

```bash
sudo apt update
sudo apt install ext4magic
```

#### Method 2: Debian/Ubuntu Package Manager

```bash
sudo apt update
sudo apt install ext4magic
```

#### Method 3: Compile from Source

```bash
# Install build dependencies
sudo apt install build-essential libext2fs-dev

# Clone the repository
git clone https://github.com/log204/ext4magic.git
cd ext4magic

# Build
./configure
make

# Install
sudo make install
```

#### Method 4: Using Docker

```bash
# Pull the ext4magic container
docker pull ext4magic/ext4magic

# Run with device access
docker run --privileged -v /dev:/dev -v /output:/output ext4magic/ext4magic /dev/sda1 -o /output
```

### Verification

After installation, verify ext4magic is working:

```bash
# Check version
ext4magic --version

# Expected output
# ext4magic version 2.0.0

# Check help
ext4magic --help

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
sudo ext4magic /dev/sda1 -r -o /recovery/output

# Recover from an image file
sudo ext4magic /path/to/filesystem.img -r -o /recovery/output
```

**Sample Output:**

```
ext4magic version 2.0.0
Filesystem: ext4
Device: /dev/sda1
Block size: 4096
Inodes: 6553600

Scanning journal...
Found 847 deleted inode entries
Found 234 deleted directory entries

Recovering files...
Restoring file1.txt (inode 12345)
Restoring file2.txt (inode 12346)
Restoring documents/report.pdf (inode 12347)
Restoring binary_file.bin (inode 12348)

Recovery complete:
- Files recovered: 4
- Directories recovered: 1
- Total size: 24 KB
```

#### Recover Specific File by Name

```bash
# Recover a specific file
sudo ext4magic /dev/sda1 -f path/to/deleted/file.txt -o /recovery

# Recover file from specific directory
sudo ext4magic /dev/sda1 -f documents/secret.pdf -o /recovery
```

#### Recover Files by Time Window

```bash
# Recover files deleted after a specific time
sudo ext4magic /dev/sda1 -r -o /recovery -d 2024-01-15

# Recover files deleted before a specific time
sudo ext4magic /dev/sda1 -r -o /recovery -D 2024-01-16

# Recover files within a time range
sudo ext4magic /dev/sda1 -r -o /recovery -d 2024-01-15 -D 2024-01-16
```

#### List Deleted Files

```bash
# List all deleted inodes
sudo ext4magic /dev/sda1 -l

# List with details
sudo ext4magic /dev/sda1 -l -d

# List deleted directory entries
sudo ext4magic /dev/sda1 -L
```

**Sample Output:**

```
Deleted inodes found:
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
| `-r` | Recover all deleted files | `sudo ext4magic /dev/sda1 -r -o /out` |
| `-f FILE` | Recover specific file | `sudo ext4magic /dev/sda1 -f file.txt` |
| `-l` | List deleted inodes | `sudo ext4magic /dev/sda1 -l` |
| `-L` | List deleted entries | `sudo ext4magic /dev/sda1 -L` |
| `-d DATE` | Files deleted after DATE | `sudo ext4magic /dev/sda1 -r -d 2024-01-15` |
| `-D DATE` | Files deleted before DATE | `sudo ext4magic /dev/sda1 -r -D 2024-01-16` |
| `-o DIR` | Output directory | `sudo ext4magic /dev/sda1 -r -o /recovery` |
| `-a` | Show all files (including live) | `sudo ext4magic /dev/sda1 -a` |
| `-v` | Verbose output | `sudo ext4magic /dev/sda1 -r -v` |
| `-t` | Show file types | `sudo ext4magic /dev/sda1 -l -t` |
| `-s START` | Start block | `sudo ext4magic /dev/sda1 -r -s 0` |
| `-e END` | End block | `sudo ext4magic /dev/sda1 -r -e 1000000` |
| `-j` | Show journal info | `sudo ext4magic /dev/sda1 -j` |
| `-I INODE` | Show specific inode | `sudo ext4magic /dev/sda1 -I 12345` |
| `-B BLOCK` | Show specific block | `sudo ext4magic /dev/sda1 -B 65536` |
| `-V` | Show version | `ext4magic -V` |
| `-h` | Show help | `ext4magic -h` |

### Basic Recovery Scenarios

#### Scenario 1: Recover Recently Deleted File

```bash
# Step 1: Identify the partition
sudo fdisk -l /dev/sda
# Output shows /dev/sda1 is ext4

# Step 2: List deleted files
sudo ext4magic /dev/sda1 -l -d

# Step 3: Recover specific file
sudo ext4magic /dev/sda1 -f important_document.docx -o /recovery

# Step 4: Check recovered files
ls -la /recovery/
```

#### Scenario 2: Recover All Files from Time Window

```bash
# Files deleted between 2024-01-15 14:00 and 15:00
sudo ext4magic /dev/sda1 -r \
  -d 2024-01-15 \
  -D 2024-01-16 \
  -o /forensics/recovered

# Check results
find /forensics/recovered -type f | wc -l
```

## Chapter 4: Intermediate Usage

### Advanced Journal Analysis

#### Parsing Journal Entries Manually

```bash
# Show journal information
sudo ext4magic /dev/sda1 -j

# Dump journal to file for analysis
sudo ext4magic /dev/sda1 -j > journal_info.txt

# Analyze journal blocks
sudo ext4magic /dev/sda1 -B 32768
sudo ext4magic /dev/sda1 -B 32769
```

#### Journal Structure Analysis

```bash
# Show journal superblock details
sudo ext4magic /dev/sda1 -j -v

# Analyze journal transactions
sudo ext4magic /dev/sda1 -j | grep -i transaction

# Find journal start block
sudo dumpe2fs /dev/sda1 | grep -i journal
```

### Scripting with ext4magic

#### Bash Script: Automated Recovery

```bash
#!/bin/bash
# ext4_auto_recover.sh - Automated ext4magic recovery script

DEVICE=$1
OUTPUT_DIR=$2
DATE_AFTER=$3
DATE_BEFORE=$4

if [ -z "$DEVICE" ] || [ -z "$OUTPUT_DIR" ]; then
    echo "Usage: $0 <device> <output_dir> [after_date] [before_date]"
    echo "Example: $0 /dev/sda1 /recovery 2024-01-15 2024-01-16"
    exit 1
fi

# Create output directory
mkdir -p "$OUTPUT_DIR"

echo "[*] Starting recovery on $DEVICE"
echo "[*] Output directory: $OUTPUT_DIR"

# Step 1: List deleted inodes
echo "[*] Listing deleted inodes..."
sudo ext4magic "$DEVICE" -l -d > "$OUTPUT_DIR/deleted_inodes.txt"

# Step 2: List deleted directory entries
echo "[*] Listing deleted entries..."
sudo ext4magic "$DEVICE" -L > "$OUTPUT_DIR/deleted_entries.txt"

# Step 3: Recover files with optional date filter
echo "[*] Recovering files..."
if [ -n "$DATE_AFTER" ] && [ -n "$DATE_BEFORE" ]; then
    sudo ext4magic "$DEVICE" -r \
        -d "$DATE_AFTER" \
        -D "$DATE_BEFORE" \
        -o "$OUTPUT_DIR/files"
elif [ -n "$DATE_AFTER" ]; then
    sudo ext4magic "$DEVICE" -r \
        -d "$DATE_AFTER" \
        -o "$OUTPUT_DIR/files"
else
    sudo ext4magic "$DEVICE" -r \
        -o "$OUTPUT_DIR/files"
fi

# Step 4: Generate summary
echo "[*] Generating summary..."
cat > "$OUTPUT_DIR/recovery_summary.txt" << EOF
Recovery Summary
================
Device: $DEVICE
Date filter: ${DATE_AFTER:-all} to ${DATE_BEFORE:-now}
Files recovered: $(find "$OUTPUT_DIR/files" -type f 2>/dev/null | wc -l)
Total size: $(du -sh "$OUTPUT_DIR/files" 2>/dev/null | cut -f1)
EOF

echo "[+] Recovery complete. Summary: $OUTPUT_DIR/recovery_summary.txt"
```

#### Python Script: Advanced Analysis

```python
#!/usr/bin/env python3
"""
ext4_analyzer.py - Python wrapper for ext4magic analysis
"""

import subprocess
import json
import re
import sys
from datetime import datetime
from pathlib import Path

class Ext4Analyzer:
    def __init__(self, device):
        self.device = device
        self.deleted_inodes = []
        self.deleted_entries = []
        self.recovered_files = []
    
    def list_deleted_inodes(self):
        """Parse deleted inodes from ext4magic output"""
        cmd = ['sudo', 'ext4magic', self.device, '-l', '-d']
        result = subprocess.run(cmd, capture_output=True, text=True)
        
        for line in result.stdout.split('\n'):
            if line.strip() and not line.startswith('Inode'):
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
    
    def list_deleted_entries(self):
        """Parse deleted directory entries"""
        cmd = ['sudo', 'ext4magic', self.device, '-L']
        result = subprocess.run(cmd, capture_output=True, text=True)
        
        for line in result.stdout.split('\n'):
            if line.strip() and not line.startswith('Inode'):
                self.deleted_entries.append(line.strip())
        
        return self.deleted_entries
    
    def recover_file(self, filepath, output_dir):
        """Recover a specific file"""
        cmd = ['sudo', 'ext4magic', self.device, 
               '-f', filepath,
               '-o', output_dir]
        result = subprocess.run(cmd, capture_output=True, text=True)
        return result.returncode == 0
    
    def recover_by_date(self, after_date=None, before_date=None, output_dir='.'):
        """Recover files within a date range"""
        cmd = ['sudo', 'ext4magic', self.device, '-r', '-o', output_dir]
        
        if after_date:
            cmd.extend(['-d', after_date])
        if before_date:
            cmd.extend(['-D', before_date])
        
        result = subprocess.run(cmd, capture_output=True, text=True)
        return result.returncode == 0
    
    def analyze_journal(self):
        """Analyze journal information"""
        cmd = ['sudo', 'ext4magic', self.device, '-j']
        result = subprocess.run(cmd, capture_output=True, text=True)
        return result.stdout
    
    def generate_report(self, output_file):
        """Generate JSON report of findings"""
        report = {
            'device': self.device,
            'timestamp': datetime.now().isoformat(),
            'deleted_inodes': self.deleted_inodes,
            'deleted_entries': self.deleted_entries,
            'journal_info': self.analyze_journal()
        }
        
        with open(output_file, 'w') as f:
            json.dump(report, f, indent=2)
        
        return report

def main():
    if len(sys.argv) < 2:
        print(f"Usage: {sys.argv[0]} <device>")
        sys.exit(1)
    
    device = sys.argv[1]
    analyzer = Ext4Analyzer(device)
    
    print(f"[*] Analyzing {device}...")
    
    # List deleted inodes
    print("[*] Listing deleted inodes...")
    inodes = analyzer.list_deleted_inodes()
    print(f"[+] Found {len(inodes)} deleted inodes")
    
    # List deleted entries
    print("[*] Listing deleted entries...")
    entries = analyzer.list_deleted_entries()
    print(f"[+] Found {len(entries)} deleted entries")
    
    # Analyze journal
    print("[*] Analyzing journal...")
    journal = analyzer.analyze_journal()
    print(f"[+] Journal analysis complete")
    
    # Generate report
    report_file = f"ext4_analysis_{datetime.now().strftime('%Y%m%d_%H%M%S')}.json"
    analyzer.generate_report(report_file)
    print(f"[+] Report saved to {report_file}")

if __name__ == '__main__':
    main()
```

### Combining with Other Tools

#### Workflow: ext4magic + foremost

```bash
# Step 1: Use ext4magic for journal-based recovery
sudo ext4magic /dev/sda1 -r -o /recovery/ext4magic

# Step 2: Use foremost for additional carving on unallocated blocks
sudo foremost -t all -i /dev/sda1 -o /recovery/foremost

# Step 3: Compare results
echo "ext4magic recovered: $(find /recovery/ext4magic -type f 2>/dev/null | wc -l) files"
echo "foremost recovered: $(find /recovery/foremost -type f 2>/dev/null | wc -l) files"
```

#### Workflow: ext4magic + photorec

```bash
# Create forensic image first
sudo dd if=/dev/sda1 of=/forensics/evidence/sda1.img bs=4M status=progress

# Use ext4magic on image
sudo ext4magic /forensics/evidence/sda1.img -r -o /recovery/ext4magic

# Use photorec on same image for comparison
sudo photorec /d /recovery/photorec /cmd /forensics/evidence/sda1.img search
```

#### Workflow: ext4magic + extundelete

```bash
# Use both tools for comprehensive recovery
sudo ext4magic /dev/sda1 -r -o /recovery/ext4magic
sudo extundelete /dev/sda1 --restore-all --output-dir /recovery/extundelete

# Merge and deduplicate results
find /recovery/ext4magic /recovery/extundelete -type f -exec md5sum {} \; | \
    sort | uniq -w32 -d > /recovery/duplicates.txt
```

## Chapter 5: Advanced Usage

### Performance Optimization

#### Large Filesystem Recovery

```bash
# For filesystems > 100GB, use block range scanning
sudo ext4magic /dev/sda1 -r -o /recovery \
    -s 0 -e 1000000

# Process in chunks
TOTAL_BLOCKS=$(sudo dumpe2fs /dev/sda1 | grep "Block count" | awk '{print $NF}')
CHUNK=$((TOTAL_BLOCKS / 4))

for i in {0..3}; do
    START=$((i * CHUNK))
    END=$(((i + 1) * CHUNK))
    sudo ext4magic /dev/sda1 -r -o "/recovery/chunk_$i" \
        -s $START -e $END &
done
wait
```

#### Parallel Recovery

```bash
# Find total blocks
TOTAL_BLOCKS=$(sudo dumpe2fs /dev/sda1 | grep "Block count" | awk '{print $NF}')
CHUNK=$((TOTAL_BLOCKS / 8))

# Run 8 parallel recoveries
for i in {0..7}; do
    START=$((i * CHUNK))
    END=$(((i + 1) * CHUNK))
    sudo ext4magic /dev/sda1 -r -o "/recovery/part_$i" \
        -s $START -e $END &
done
wait

# Merge results
for i in {0..7}; do
    cp -r /recovery/part_$i/* /recovery/merged/ 2>/dev/null
done
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
    ext3)
        sudo ext4magic "$WORK_DIR/images/evidence.img" -r \
            -o "$WORK_DIR/recovered/ext4magic"
        ;;
    ext4)
        sudo ext4magic "$WORK_DIR/images/evidence.img" -r \
            -o "$WORK_DIR/recovered/ext4magic"
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
- ext4magic: $(find "$WORK_DIR/recovered/ext4magic" -type f 2>/dev/null | wc -l) files
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
cat > /etc/ext4magic/recovery_profiles/incident_response.sh << 'EOF'
#!/bin/bash
# Incident Response Profile
DEVICE=$1
OUTPUT=$2

# Recover all deleted files
sudo ext4magic "$DEVICE" -r -o "$OUTPUT/all_deleted"

# Recover files from last 24 hours
AFTER=$(date -d "24 hours ago" +%Y-%m-%d)
sudo ext4magic "$DEVICE" -r -d "$AFTER" -o "$OUTPUT/last_24h"

# List all deleted inodes with details
sudo ext4magic "$DEVICE" -l -d > "$OUTPUT/deleted_inodes.txt"

# Analyze journal
sudo ext4magic "$DEVICE" -j > "$OUTPUT/journal_analysis.txt"
EOF
chmod +x /etc/ext4magic/recovery_profiles/incident_response.sh
```

## Chapter 6: Real-World Scenarios

### Scenario 1: CTF Challenge - "Hidden in Plain Sight"

**Context**: You've acquired a disk image from a CTF challenge. The description says "The attacker tried to cover their tracks by deleting files. Find the flag."

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

**Step 2: Run ext4magic**

```bash
# List all deleted inodes
sudo ext4magic challenge.img -l -d

# Output shows several deleted files including:
# 12345  REG  1024  8  2024-01-15 14:32:00  flag.txt
# 12346  REG  2048  16  2024-01-15 14:32:15  hidden/secret.txt
# 12347  DIR  4096  8  2024-01-15 14:31:45  hidden/

# Recover the flag file
sudo ext4magic challenge.img -f flag.txt -o /tmp/recovered

# Check the recovered file
cat /tmp/recovered/flag.txt
# Output: CTF{3xt4mag1c_r3c0v3ry_m4st3r}
```

**Step 3: Advanced Recovery**

```bash
# The hidden directory might contain more flags
sudo ext4magic challenge.img -f hidden/secret.txt -o /tmp/recovered

# Check the recovered file
cat /tmp/recovered/hidden/secret.txt
# Output: CTF{j0urn4l_4n4ly51s_pr0}
```

### Scenario 2: Incident Response - Insider Threat

**Context**: A company suspects an employee of stealing sensitive data. The employee deleted files before leaving. You need to recover evidence.

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
# Determine when the employee last accessed the system
# From logs: 2024-01-15 17:00:00

# List files deleted around that time
sudo ext4magic /forensics/evidence/sdb1.img -l -d | grep "Jan 15"

# Recover files from that day
sudo ext4magic /forensics/evidence/sdb1.img -r \
    -d 2024-01-15 \
    -D 2024-01-16 \
    -o /forensics/recovered/employee_data
```

**Step 3: Validation**

```bash
# Verify recovered files
find /forensics/recovered/employee_data -type f -exec file {} \;

# Check for sensitive documents
ls -la /forensics/recovered/employee_data/
# Found: financial_report.xlsx, client_list.docx, confidential_presentation.pptx

# Generate report for legal team
cat > /forensics/reports/employee_theft.txt << EOF
Digital Forensics Report - Insider Threat
==========================================
Employee: John Doe
Last Access: 2024-01-15 17:00:00

Recovered Files:
- financial_report.xlsx (deleted 2024-01-15 17:30:00)
- client_list.docx (deleted 2024-01-15 17:31:00)
- confidential_presentation.pptx (deleted 2024-01-15 17:32:00)

File hashes for evidence integrity:
$(find /forensics/recovered/employee_data -type f -exec md5sum {} \;)
EOF
```

### Scenario 3: Digital Forensics - Data Breach Investigation

**Context**: A company experienced a data breach. The attacker gained access and deleted logs and evidence. Recover the deleted files to understand the attack.

**Step 1: Evidence Collection**

```bash
# Collect all relevant evidence
EVIDENCE_DEVICES="/dev/sda1 /dev/sdb1 /dev/sdc1"
CASE_ID="BREACH_$(date +%Y%m%d)"

mkdir -p /forensics/$CASE_ID/{images,recovered,reports}

for DEV in $EVIDENCE_DEVICES; do
    echo "Imaging $DEV..."
    sudo dd if=$DEV of="/forensics/$CASE_ID/images/$(basename $DEV).img" bs=4M status=progress
done
```

**Step 2: Comprehensive Recovery**

```bash
# Run ext4magic on each image
for IMG in /forensics/$CASE_ID/images/*.img; do
    echo "Processing $IMG..."
    sudo ext4magic $IMG -r -o "/forensics/$CASE_ID/recovered/$(basename $IMG)"
done

# Also recover logs specifically
sudo ext4magic /forensics/$CASE_ID/images/sda1.img -f var/log/syslog -o /forensics/$CASE_ID/recovered/logs
sudo ext4magic /forensics/$CASE_ID/images/sda1.img -f var/log/auth.log -o /forensics/$CASE_ID/recovered/logs
```

**Step 3: Attack Analysis**

```bash
# Analyze recovered logs
cat /forensics/$CASE_ID/recovered/logs/var/log/syslog | grep -i "failed\|error\|attack"

# Look for suspicious files
find /forensics/$CASE_ID/recovered -type f -name "*.sh" -o -name "*.py" -o -name "*.pl" | \
    xargs file | grep -i "script\|executable"

# Generate attack timeline
cat > /forensics/$CASE_ID/reports/attack_timeline.txt << EOF
Data Breach Investigation - Attack Timeline
=============================================
Case: $CASE_ID
Analysis Date: $(date)

Timeline:
- 2024-01-15 02:00:00 - Initial unauthorized access detected
- 2024-01-15 02:15:00 - Attacker escalated privileges
- 2024-01-15 02:30:00 - Sensitive data accessed
- 2024-01-15 03:00:00 - Logs deleted to cover tracks
- 2024-01-15 03:05:00 - Backdoor installed

Recovered Evidence:
- Deleted syslog entries showing unauthorized access
- Backdoor script (shell.sh) in /tmp
- Exfiltrated data fragments in /var/tmp
EOF
```

## Chapter 7: Detection & Defense

### How Defenders Detect ext4magic Usage

#### System Monitoring

```bash
# Monitor for ext4magic execution
auditctl -w /usr/bin/ext4magic -p x -k ext4magic_usage

# Check audit logs
ausearch -k ext4magic_usage

# Monitor process execution
ps aux | grep ext4magic
```

#### Log Analysis

```bash
# Check for ext4magic in system logs
grep -r "ext4magic" /var/log/syslog
grep -r "ext4magic" /var/log/auth.log

# Monitor disk access patterns
iotop -a -d 1 | grep ext4magic
```

#### Filesystem Indicators

```bash
# Check for recovery directories
find / -name "recovered" -type d 2>/dev/null
find / -name "RESTORED_FILES" -type d 2>/dev/null

# Look for ext4magic output files
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
rule ext4magic_Usage {
    meta:
        description = "Detects ext4magic execution artifacts"
        author = "Security Team"
        date = "2024-01-15"
    
    strings:
        $s1 = "ext4magic" ascii
        $s2 = "deleted inode" ascii
        $s3 = "recovering" ascii
    
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

# Check for ext4magic artifacts
echo "[*] Checking for ext4magic artifacts..."
if find / -name "recovered" -type d 2>/dev/null | grep -q .; then
    echo "[!] ALERT: Recovery directory found"
    find / -name "recovered" -type d 2>/dev/null
fi

# Check running processes
echo "[*] Checking for recovery tools in process list..."
if ps aux | grep -E "(ext4magic|ext3grep|extundelete|foremost)" | grep -v grep; then
    echo "[!] ALERT: Recovery tool currently running"
fi

# Check recent file modifications
echo "[*] Checking for recently modified filesystem images..."
find / -name "*.img" -mtime -1 -type f 2>/dev/null

# Check shell history
echo "[*] Checking shell history for recovery commands..."
grep -l "ext4magic\|ext3grep\|extundelete\|foremost" /home/*/.bash_history 2>/dev/null

echo "=== Scan Complete ==="
```

## Chapter 8: Troubleshooting

### Common Errors and Solutions

#### Error: "Unsupported filesystem"

```bash
# Problem: ext4magic returns "Unsupported filesystem"
# Cause: Filesystem type not recognized

# Solution: Check filesystem type
sudo blkid /dev/sda1
# Output: /dev/sda1: UUID="..." TYPE="ext4"

# Ensure you're using the latest version
sudo apt update && sudo apt upgrade ext4magic
```

#### Error: "Journal not found"

```bash
# Problem: ext4magic cannot find the journal
# Cause: Journal may be external or corrupted

# Solution 1: Check journal location
sudo dumpe2fs /dev/sda1 | grep -i journal

# Solution 2: Try recovering without journal analysis
sudo ext4magic /dev/sda1 -r -o /recovery --no-journal
```

#### Error: "Permission denied"

```bash
# Problem: ext4magic requires root access
# Cause: Running as regular user

# Solution: Use sudo
sudo ext4magic /dev/sda1 -r -o /recovery

# Or add to sudoers for specific user
echo "username ALL=(ALL) NOPASSWD: /usr/bin/ext4magic" | sudo tee /etc/sudoers.d/ext4magic
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
    sudo ext4magic /dev/sda1 -r \
        -s $i -e $END \
        -o "/recovery/chunk_$i"
done
```

#### High Memory Usage

```bash
# Problem: ext4magic uses too much memory
# Solution: Process in batches

# Limit memory usage
ulimit -v 2097152  # Limit to 2GB virtual memory
sudo ext4magic /dev/sda1 -r -o /recovery
```

### FAQ

**Q1: Can ext4magic recover files from ext3 filesystems?**

A: Yes, ext4magic supports both ext3 and ext4 filesystems. It's a superset of ext3grep functionality.

**Q2: How successful is ext4magic at recovering deleted files?**

A: Success depends on several factors: how recently the file was deleted, whether the journal entries are still present, and whether the data blocks have been overwritten. For recently deleted files with intact journal entries, recovery success can be 90%+.

**Q3: Can ext4magic recover encrypted files?**

A: ext4magic can recover the encrypted file data, but it cannot decrypt it. You would need the encryption key to access the file contents.

**Q4: Is it safe to run ext4magic on a live filesystem?**

A: It's generally safe to READ from a live filesystem, but writing recovered files to the same filesystem could overwrite other deleted data. Always recover to a different device.

**Q5: How do I recover files deleted a long time ago?**

A: Recovery of old deletions is less likely because journal entries may have been overwritten. However, if the data blocks haven't been overwritten, ext4magic might still find them through block scanning.

**Q6: Can ext4magic recover files from a RAID array?**

A: ext4magic works on individual filesystems. For RAID arrays, you need to first assemble the RAID and then run ext4magic on the resulting logical volume.

**Q7: What's the difference between ext4magic and extundelete?**

A: Both recover deleted files from ext3/ext4 using journal analysis. ext4magic offers more options for time-based filtering and block range scanning, while extundelete has a simpler interface. Using both can improve recovery rates.

## Resources

### Official Documentation

- ext4magic Manual Page: `man ext4magic`
- ext4magic GitHub: https://github.com/log204/ext4magic
- Linux ext4 Documentation: https://www.kernel.org/doc/html/latest/filesystems/ext4.html

### Tutorials and Guides

- SANS Digital Forensics: File Carving Techniques
- Linux Forensics Workshop Materials
- Ext4 Filesystem Internals Documentation

### Related Tools

- **ext3grep**: Original ext3 recovery tool (ext4magic is its successor)
- **extundelete**: Another ext3/ext4 recovery tool using journal
- **photorec**: General-purpose file carving tool
- **foremost**: File carving based on headers/footers
- **scalpel**: Another file carving tool
- **testdisk**: Partition recovery and file undeletion

### Practice Resources

- Digital Forensics Challenge Images
- CTF Forensics Challenges
- Practice ext4 filesystems with test data
- Volatility Foundation Memory Analysis Practice

### Books and Papers

- "Linux Forensics" by Phil Rhodes
- "File System Forensic Analysis" by Brian Carrier
- "The Art of Memory Forensics" by Michael Hale Ligh
