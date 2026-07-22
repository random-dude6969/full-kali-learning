---
tool_name: recoverjpeg
display_name: RecoverJPEG
mitre_tactic: "analysis"
mitre_tactic_id: "TA0007"
install_command: "sudo apt install recoverjpeg"
github_repo: "https://github.com/lahanas/recoverjpeg"
kali_tools_page: "https://www.kali.org/tools/recoverjpeg/"
difficulty_level: "beginner"
requires_root: true
gui_available: false
tags:
  - file-carving
  - jpeg-recovery
  - forensic-analysis
  - data-recovery
  - image-recovery
related_tools:
  - photorec
  - foremost
  - scalpel
  - magicrescue
---

# RecoverJPEG - Comprehensive Forensic Documentation

## Table of Contents

1. [Introduction & Overview](#chapter-1-introduction--overview)
2. [Installation & Setup](#chapter-2-installation--setup)
3. [Basic Usage](#chapter-3-basic-usage)
4. [Intermediate Usage](#chapter-4-intermediate-usage)
5. [Advanced Usage](#chapter-5-advanced-usage)
6. [Real-World Scenarios](#chapter-6-real-world-scenarios)
7. [Detection & Defense](#chapter-7-detection--defense)
8. [Troubleshooting](#chapter-8-troubleshooting)
9. [Resources](#resources)

---

## Chapter 1: Introduction & Overview

### 1.1 What is RecoverJPEG?

RecoverJPEG is a specialized forensic file carving tool designed exclusively for recovering JPEG image files from damaged, corrupted, or formatted storage media. Unlike general-purpose recovery tools, RecoverJPEG focuses solely on JPEG format recovery, making it highly optimized and efficient for this specific task.

The tool works by scanning raw disk images or physical devices byte-by-byte, looking for JPEG file signatures (magic bytes) that mark the beginning and end of JPEG files. It identifies JPEG files by searching for:

- **JFIF headers**: The standard JPEG File Interchange Format marker (`FF D8 FF E0`)
- **EXIF headers**: The Exchangeable Image File Format marker (`FF D8 FF E1`)
- **Other JPEG variants**: Including Adobe JPEG (`FF D8 FF DB`) and other APP markers

### 1.2 History and Development

RecoverJPEG was developed by Nikos Lahanas as a lightweight, purpose-built tool for digital forensic investigators who needed快速 JPEG recovery without the overhead of full forensic suites. The tool has been included in major Linux distributions, including Kali Linux, making it readily available to forensic practitioners.

### 1.3 Core Capabilities

| Capability | Description |
|------------|-------------|
| JPEG-specific carving | Optimized exclusively for JPEG file recovery |
| Raw device scanning | Reads directly from block devices and disk images |
| Header-based detection | Identifies JPEG files by their magic bytes |
| Sequential recovery | Extracts files in the order they appear on disk |
| Minimal dependencies | Lightweight tool with no external library requirements |
| Cross-platform | Works on Linux, macOS, and other Unix-like systems |

### 1.4 When to Use RecoverJPEG

RecoverJPEG is ideal for:

- **Accidental deletion**: Recovering JPEG files deleted from cameras, phones, or computers
- **Formatted media**: Extracting images from reformatted SD cards or USB drives
- **Corrupted filesystems**: Recovering photos from damaged storage devices
- **Disk images**: Analyzing forensic disk images for JPEG content
- **Raw device analysis**: Scanning physical drives for deleted images

### 1.5 Limitations

- **JPEG only**: Cannot recover other file formats (use photorec or foremost for multi-format recovery)
- **No file system awareness**: Does not understand filesystem structures (ext4, NTFS, FAT32)
- **Sequential output**: Files are saved numerically (file001.jpg, file002.jpg, etc.)
- **No preview**: Cannot preview recovered files during the carving process
- **Limited metadata recovery**: May not preserve original filenames or directory structures

### 1.6 Comparison with Similar Tools

| Feature | RecoverJPEG | PhotoRec | Foremost | Scalpel |
|---------|-------------|----------|----------|---------|
| JPEG recovery | Excellent | Good | Good | Good |
| Multi-format | No | Yes | Yes | Yes |
| Speed | Fast | Medium | Fast | Fast |
| Configuration | Minimal | Configurable | Configurable | Configurable |
| File system aware | No | No | No | No |

---

## Chapter 2: Installation & Setup

### 2.1 Installing on Kali Linux

RecoverJPEG is available in the Kali Linux repositories and can be installed using the apt package manager:

```bash
# Update package lists
sudo apt update

# Install recoverjpeg
sudo apt install recoverjpeg

# Verify installation
recoverjpeg --version
```

### 2.2 Installing from Source

For the latest development version or to customize the build:

```bash
# Install build dependencies
sudo apt install build-essential git

# Clone the repository
git clone https://github.com/lahanas/recoverjpeg.git
cd recoverjpeg

# Build the tool
make

# Install to system (optional)
sudo make install

# Verify installation
./recoverjpeg --help
```

### 2.3 Installation Verification

After installation, verify that recoverjpeg is working correctly:

```bash
# Check version
recoverjpeg --version

# Display help
recoverjpeg --help

# Check if tool is in PATH
which recoverjpeg
```

### 2.4 Setting Up a Test Environment

Before using RecoverJPEG in forensic scenarios, set up a safe test environment:

```bash
# Create a working directory
mkdir -p ~/forensic_work/recoverjpeg_test
cd ~/forensic_work/recoverjpeg_test

# Create a test disk image (10MB)
dd if=/dev/zero of=test_disk.img bs=1M count=10

# Format with ext4 (for testing)
mkfs.ext4 test_disk.img

# Mount the image
sudo mkdir -p /mnt/test_disk
sudo mount -o loop test_disk.img /mnt/test_disk

# Copy some test JPEG files
sudo cp /path/to/test_images/*.jpg /mnt/test_disk/

# Unmount
sudo umount /mnt/test_disk
```

### 2.5 Working Directory Structure

Organize your forensic workspace:

```
forensic_work/
├── case_001/
│   ├── evidence/          # Original evidence (read-only)
│   ├── working/           # Working copies
│   ├── recovered/         # Recovered files
│   ├── logs/              # Command logs
│   └── reports/           # Documentation
├── tools/
│   └── recoverjpeg/       # Tool installation
└── templates/             # Report templates
```

### 2.6 Permission Configuration

RecoverJPEG requires root access to read raw devices:

```bash
# Add user to disk group (temporary)
sudo usermod -aG disk $USER

# Log out and back in for group changes
# Or use newgrp
newgrp disk

# Verify group membership
groups
```

---

## Chapter 3: Basic Usage

### 3.1 Command Syntax

The basic syntax for RecoverJPEG is:

```bash
recoverjpeg [options] <device_or_image>
```

### 3.2 Essential Commands

#### Recover from a physical device

```bash
# Recover JPEG files from a USB drive
sudo recoverjpeg /dev/sdb1

# Recover from an SD card
sudo recoverjpeg /dev/mmcblk0p1
```

#### Recover from a disk image

```bash
# Recover from a raw disk image
sudo recoverjpeg disk_image.dd

# Recover from an E01 forensic image (requires ewfmount)
ewfmount evidence.E01 /mnt/ewf
sudo recoverjpeg /mnt/ewf/ewf1
```

### 3.3 Command Options Reference

| Option | Long Form | Description | Example |
|--------|-----------|-------------|---------|
| `-o` | `--output-directory` | Specify output directory for recovered files | `-o /path/to/output` |
| `-v` | `--verbose` | Enable verbose output | `-v` |
| `-h` | `--help` | Display help information | `-h` |
| `-V` | `--version` | Display version information | `-V` |

### 3.4 Output Directory Management

By default, RecoverJPEG saves recovered files to the current directory. Use the `-o` option to specify a different location:

```bash
# Create output directory
mkdir -p ~/recovered_jpegs

# Recover to specific directory
sudo recoverjpeg -o ~/recovered_jpegs /dev/sdb1

# Verify recovered files
ls -la ~/recovered_jpegs/
```

### 3.5 Understanding Output Files

Recovered files are saved with sequential numbering:

```
file00001.jpg
file00002.jpg
file00003.jpg
...
```

The numbering corresponds to the order in which files were found on the disk, starting from the beginning of the device.

### 3.6 Reading from Standard Input

RecoverJPEG can read from standard input, allowing integration with other tools:

```bash
# Pipe from dd
dd if=/dev/sdb1 bs=4096 | recoverjpeg -o output/

# Pipe from other imaging tools
cat disk_image.dd | recoverjpeg -o output/
```

### 3.7 Basic Recovery Workflow

Complete basic recovery workflow:

```bash
# Step 1: Identify the target device
lsblk
sudo fdisk -l

# Step 2: Create a working copy (NEVER work on original evidence)
sudo dd if=/dev/sdb1 of=~/evidence_backup.dd bs=4096 status=progress

# Step 3: Verify the backup
md5sum /dev/sdb1 ~/evidence_backup.dd

# Step 4: Run RecoverJPEG
sudo recoverjpeg -o ~/recovered ~/evidence_backup.dd

# Step 5: Document the recovery
echo "Recovery completed: $(date)" > ~/recovery_log.txt
echo "Files recovered: $(ls ~/recovered/*.jpg | wc -l)" >> ~/recovery_log.txt
```

### 3.8 Exit Codes

| Code | Meaning |
|------|---------|
| 0 | Successful recovery |
| 1 | Error occurred |
| 2 | Invalid arguments |

---

## Chapter 4: Intermediate Usage

### 4.1 Bash Scripting for Automated Recovery

#### Batch Processing Multiple Devices

```bash
#!/bin/bash
# batch_recovery.sh - Recover JPEGs from multiple devices

OUTPUT_BASE="/home/forensic/recovered"
LOG_FILE="$OUTPUT_BASE/recovery_log_$(date +%Y%m%d_%H%M%S).log"

# Function to log messages
log_message() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG_FILE"
}

# Function to recover from a single device
recover_device() {
    local device=$1
    local output_dir="$OUTPUT_BASE/$(basename $device)"
    
    mkdir -p "$output_dir"
    
    log_message "Starting recovery from $device"
    
    # Run recoverjpeg
    sudo recoverjpeg -o "$output_dir" "$device" 2>&1 | tee -a "$LOG_FILE"
    
    # Count recovered files
    local file_count=$(ls "$output_dir"/*.jpg 2>/dev/null | wc -l)
    log_message "Recovered $file_count files from $device"
    
    return $file_count
}

# Main execution
log_message "=== Batch Recovery Started ==="

# Find all USB storage devices
for device in /dev/sd[b-z]; do
    if [ -b "$device" ]; then
        recover_device "$device"
    fi
done

log_message "=== Batch Recovery Complete ==="
log_message "Total files recovered: $(find $OUTPUT_BASE -name '*.jpg' | wc -l)"
```

#### Recovery with Integrity Verification

```bash
#!/bin/bash
# verified_recovery.sh - Recovery with MD5 verification

DEVICE=$1
OUTPUT_DIR=$2
CHECKSUM_FILE="$OUTPUT_DIR/checksums.md5"

if [ -z "$DEVICE" ] || [ -z "$OUTPUT_DIR" ]; then
    echo "Usage: $0 <device> <output_dir>"
    exit 1
fi

# Create output directory
mkdir -p "$OUTPUT_DIR"

# Generate source checksum
echo "Generating source checksum..."
md5sum "$DEVICE" > "$DEVICE.md5"

# Run recovery
echo "Starting recovery..."
sudo recoverjpeg -o "$OUTPUT_DIR" "$DEVICE"

# Generate checksums for recovered files
echo "Generating checksums for recovered files..."
cd "$OUTPUT_DIR"
find . -name "*.jpg" -exec md5sum {} \; > "$CHECKSUM_FILE"

# Compare with expected checksums if available
if [ -f "expected_checksums.md5" ]; then
    echo "Verifying checksums..."
    md5sum -c expected_checksums.md5
fi

echo "Recovery complete. Checksums saved to $CHECKSUM_FILE"
```

### 4.2 Python Integration

#### Automated Recovery Script with Logging

```python
#!/usr/bin/env python3
"""
automated_recovery.py - Python wrapper for RecoverJPEG with logging
"""

import subprocess
import os
import logging
from datetime import datetime
from pathlib import Path
import hashlib

class RecoverJPEGWrapper:
    def __init__(self, output_base="/home/forensic/recovered"):
        self.output_base = Path(output_base)
        self.output_base.mkdir(parents=True, exist_ok=True)
        
        # Setup logging
        log_file = self.output_base / f"recovery_{datetime.now():%Y%m%d_%H%M%S}.log"
        logging.basicConfig(
            level=logging.INFO,
            format='%(asctime)s - %(levelname)s - %(message)s',
            handlers=[
                logging.FileHandler(log_file),
                logging.StreamHandler()
            ]
        )
        self.logger = logging.getLogger(__name__)
    
    def calculate_md5(self, filepath):
        """Calculate MD5 hash of a file"""
        hash_md5 = hashlib.md5()
        with open(filepath, "rb") as f:
            for chunk in iter(lambda: f.read(4096), b""):
                hash_md5.update(chunk)
        return hash_md5.hexdigest()
    
    def recover(self, device, case_id=None):
        """
        Recover JPEG files from a device or image
        
        Args:
            device: Path to device or disk image
            case_id: Optional case identifier
        
        Returns:
            dict: Recovery statistics
        """
        # Create case-specific output directory
        if case_id:
            output_dir = self.output_base / case_id
        else:
            output_dir = self.output_base / datetime.now().strftime("%Y%m%d_%H%M%S")
        
        output_dir.mkdir(parents=True, exist_ok=True)
        
        self.logger.info(f"Starting recovery from {device}")
        self.logger.info(f"Output directory: {output_dir}")
        
        # Generate source hash
        source_hash = self.calculate_md5(device)
        self.logger.info(f"Source MD5: {source_hash}")
        
        # Run recoverjpeg
        cmd = ["sudo", "recoverjpeg", "-o", str(output_dir), device]
        
        try:
            result = subprocess.run(
                cmd,
                capture_output=True,
                text=True,
                check=True
            )
            self.logger.info("Recovery command executed successfully")
        except subprocess.CalledProcessError as e:
            self.logger.error(f"Recovery failed: {e.stderr}")
            return None
        
        # Count recovered files
        recovered_files = list(output_dir.glob("*.jpg"))
        file_count = len(recovered_files)
        
        # Calculate checksums
        checksums = {}
        for f in recovered_files:
            checksums[f.name] = self.calculate_md5(f)
        
        # Save checksums
        checksum_file = output_dir / "checksums.md5"
        with open(checksum_file, "w") as f:
            for filename, hash_val in checksums.items():
                f.write(f"{hash_val}  {filename}\n")
        
        self.logger.info(f"Recovery complete: {file_count} files recovered")
        
        return {
            "device": device,
            "output_dir": str(output_dir),
            "source_hash": source_hash,
            "files_recovered": file_count,
            "checksums": checksums
        }

def main():
    import argparse
    
    parser = argparse.ArgumentParser(description="Automated JPEG Recovery")
    parser.add_argument("device", help="Device or image to recover from")
    parser.add_argument("-o", "--output", default="/home/forensic/recovered",
                        help="Output directory")
    parser.add_argument("-c", "--case-id", help="Case identifier")
    
    args = parser.parse_args()
    
    recovery = RecoverJPEGWrapper(args.output)
    result = recovery.recover(args.device, args.case_id)
    
    if result:
        print(f"\nRecovery Summary:")
        print(f"  Files recovered: {result['files_recovered']}")
        print(f"  Output directory: {result['output_dir']}")
        print(f"  Source hash: {result['source_hash']}")
    else:
        print("Recovery failed!")
        exit(1)

if __name__ == "__main__":
    main()
```

#### Integration with Forensic Framework

```python
#!/usr/bin/env python3
"""
forensic_integration.py - Integrate RecoverJPEG with forensic workflow
"""

import subprocess
import json
from datetime import datetime
from dataclasses import dataclass
from typing import List, Dict
import os

@dataclass
class RecoveryCase:
    case_id: str
    investigator: str
    device: str
    description: str
    
@dataclass
class RecoveryResult:
    case_id: str
    device: str
    files_recovered: int
    output_dir: str
    timestamp: str
    checksums: Dict[str, str]

class ForensicRecoveryManager:
    def __init__(self, base_dir="/home/forensic"):
        self.base_dir = base_dir
        self.cases_dir = os.path.join(base_dir, "cases")
        os.makedirs(self.cases_dir, exist_ok=True)
    
    def create_case(self, case: RecoveryCase) -> str:
        """Create a new forensic case"""
        case_dir = os.path.join(self.cases_dir, case.case_id)
        os.makedirs(case_dir, exist_ok=True)
        
        # Save case metadata
        metadata = {
            "case_id": case.case_id,
            "investigator": case.investigator,
            "device": case.device,
            "description": case.description,
            "created": datetime.now().isoformat()
        }
        
        metadata_file = os.path.join(case_dir, "case_metadata.json")
        with open(metadata_file, "w") as f:
            json.dump(metadata, f, indent=2)
        
        return case_dir
    
    def execute_recovery(self, case: RecoveryCase) -> RecoveryResult:
        """Execute JPEG recovery for a case"""
        case_dir = self.create_case(case)
        output_dir = os.path.join(case_dir, "recovered")
        os.makedirs(output_dir, exist_ok=True)
        
        # Execute recoverjpeg
        cmd = ["sudo", "recoverjpeg", "-o", output_dir, case.device]
        result = subprocess.run(cmd, capture_output=True, text=True)
        
        # Count recovered files
        recovered_files = [f for f in os.listdir(output_dir) if f.endswith('.jpg')]
        
        return RecoveryResult(
            case_id=case.case_id,
            device=case.device,
            files_recovered=len(recovered_files),
            output_dir=output_dir,
            timestamp=datetime.now().isoformat(),
            checksums={}
        )
    
    def generate_report(self, result: RecoveryResult) -> str:
        """Generate a forensic recovery report"""
        report = f"""
FORENSIC RECOVERY REPORT
========================

Case ID: {result.case_id}
Device: {result.device}
Timestamp: {result.timestamp}
Files Recovered: {result.files_recovered}
Output Directory: {result.output_dir}

SUMMARY
-------
JPEG recovery completed successfully using RecoverJPEG.

FILES RECOVERED
---------------
Total files: {result.files_recovered}

RECOMMENDATIONS
---------------
1. Verify recovered files manually
2. Cross-reference with other recovery tools if needed
3. Document any corrupted or incomplete files

---
Report generated: {datetime.now().isoformat()}
"""
        return report

# Example usage
if __name__ == "__main__":
    manager = ForensicRecoveryManager()
    
    case = RecoveryCase(
        case_id="CASE-2024-001",
        investigator="John Doe",
        device="/dev/sdb1",
        description="SD card from digital camera - potential evidence"
    )
    
    result = manager.execute_recovery(case)
    report = manager.generate_report(result)
    
    print(report)
```

### 4.3 Combining with Other Tools

#### Workflow with dd and md5sum

```bash
#!/bin/bash
# complete_forensic_workflow.sh

DEVICE=$1
CASE_ID=$2
WORK_DIR="/home/forensic/cases/$CASE_ID"

# Create case directory structure
mkdir -p "$WORK_DIR"/{evidence,working,recovered,logs,reports}

# Step 1: Create forensic image
echo "Creating forensic image..."
sudo dd if=$DEVICE of="$WORK_DIR/evidence/original.dd" bs=4096 status=progress

# Step 2: Generate hash
echo "Generating hash..."
md5sum "$WORK_DIR/evidence/original.dd" > "$WORK_DIR/evidence/original.md5"

# Step 3: Run RecoverJPEG
echo "Running RecoverJPEG..."
sudo recoverjpeg -o "$WORK_DIR/recovered" "$WORK_DIR/evidence/original.dd"

# Step 4: Run PhotoRec for comparison
echo "Running PhotoRec for comparison..."
sudo photorec /cmd "$WORK_DIR/evidence/original.dd" fileopt,everything,disable,jpg,enable search "$WORK_DIR/recovered/photorec/"

# Step 5: Compare results
echo "Comparing results..."
RECOVERJEEP_COUNT=$(ls "$WORK_DIR/recovered"/*.jpg 2>/dev/null | wc -l)
PHOTOREC_COUNT=$(find "$WORK_DIR/recovered/photorec" -name "*.jpg" 2>/dev/null | wc -l)

echo "RecoverJPEG: $RECOVERJEEP_COUNT files"
echo "PhotoRec: $PHOTOREC_COUNT files"

# Step 6: Generate report
cat > "$WORK_DIR/reports/recovery_report.txt" << EOF
Forensic Recovery Report
========================
Case ID: $CASE_ID
Date: $(date)
Device: $DEVICE

RecoverJPEG Results: $RECOVERJEEP_COUNT files
PhotoRec Results: $PHOTOREC_COUNT files

EOF

echo "Workflow complete. Reports saved to $WORK_DIR/reports/"
```

---

## Chapter 5: Advanced Usage

### 5.1 Working with Forensic Images

#### E01 (Expert Witness Format) Images

```bash
# Mount E01 image
sudo apt install ewf-tools
sudo ewfmount evidence.E01 /mnt/ewf

# Run RecoverJPEG on mounted image
sudo recoverjpeg -o ~/recovered /mnt/ewf/ewf1

# Unmount when done
sudo umount /mnt/ewf
```

#### AFF (Advanced Forensic Format) Images

```bash
# Install AFF tools
sudo apt install afflib-utils

# Mount AFF image
sudo affmount evidence.aff /mnt/aff

# Run RecoverJPEG
sudo recoverjpeg -o ~/recovered /mnt/aff/affdata

# Unmount
sudo umount /mnt/aff
```

### 5.2 Parallel Processing

#### Multi-threaded Recovery

```bash
#!/bin/bash
# parallel_recovery.sh - Parallel recovery from disk image segments

IMAGE="disk_image.dd"
OUTPUT_BASE="/home/forensic/recovered"
SEGMENT_SIZE=$((1024 * 1024 * 1024))  # 1GB segments

# Get image size
IMAGE_SIZE=$(stat -c%s "$IMAGE")
NUM_SEGMENTS=$((IMAGE_SIZE / SEGMENT_SIZE + 1))

echo "Image size: $IMAGE_SIZE bytes"
echo "Segment size: $SEGMENT_SIZE bytes"
echo "Number of segments: $NUM_SEGMENTS"

# Create segment images and recover in parallel
for i in $(seq 0 $((NUM_SEGMENTS - 1))); do
    START=$((i * SEGMENT_SIZE))
    SEGMENT_FILE="segment_${i}.dd"
    
    # Extract segment
    dd if="$IMAGE" of="$SEGMENT_FILE" bs=4096 skip=$((START / 4096)) count=$((SEGMENT_SIZE / 4096)) &
    
    # Wait for dd to complete before recovery
    wait
    
    # Run recovery in background
    sudo recoverjpeg -o "$OUTPUT_BASE/segment_$i" "$SEGMENT_FILE" &
done

# Wait for all recovery jobs to complete
wait

echo "Parallel recovery complete"
echo "Total files recovered: $(find $OUTPUT_BASE -name '*.jpg' | wc -l)"
```

### 5.3 Custom File Signatures

While RecoverJPEG is JPEG-specific, you can modify the source to add custom signatures:

```c
// Example: Adding custom JPEG variant detection
// In recoverjpeg source code

// Standard JPEG markers
#define JPEG_MARKER_FF    0xFF
#define JPEG_MARKER_SOI   0xD8
#define JPEG_MARKER_APP0  0xE0  // JFIF
#define JPEG_MARKER_APP1  0xE1  // EXIF

// Custom markers to detect
#define JPEG_MARKER_APP2  0xE2  // ICC Profile
#define JPEG_MARKER_APP14 0xEE  // Adobe

// Detection function
int is_jpeg_header(unsigned char *header) {
    if (header[0] == JPEG_MARKER_FF && 
        header[1] == JPEG_MARKER_SOI) {
        
        // Check for specific APP markers
        if (header[2] == JPEG_MARKER_APP0 ||  // JFIF
            header[2] == JPEG_MARKER_APP1 ||  // EXIF
            header[2] == JPEG_MARKER_APP2 ||  // ICC
            header[2] == JPEG_MARKER_APP14) { // Adobe
            return 1;
        }
    }
    return 0;
}
```

### 5.4 Integration with Memory Analysis

```bash
# Recover JPEGs from memory dump
# First, convert memory dump to raw format if needed
volatility -f memory.dump imageinfo

# Extract process memory
volatility -f memory.dump --profile=Win7SP1x64 procdump -D /tmp/procdump/

# Run RecoverJPEG on extracted processes
for proc in /tmp/procdump/*.dmp; do
    sudo recoverjpeg -o "/tmp/recovered_$(basename $proc)" "$proc"
done
```

### 5.5 Network-Based Recovery

```bash
#!/bin/bash
# network_recovery.sh - Recover JPEGs from network shares

REMOTE_HOST="192.168.1.100"
REMOTE_PATH="/evidence/usb_backup"
LOCAL_OUTPUT="/home/forensic/recovered/network"

# Mount remote share
sudo mkdir -p /mnt/remote_evidence
sudo mount -t cifs //$REMOTE_HOST/$REMOTE_PATH /mnt/remote_evidence \
    -o username=forensic,password=secure,domain=WORKGROUP

# Run RecoverJPEG
sudo recoverjpeg -o "$LOCAL_OUTPUT" /mnt/remote_evidence/disk_image.dd

# Unmount
sudo umount /mnt/remote_evidence

echo "Network recovery complete"
```

### 5.6 Automated Evidence Processing Pipeline

```python
#!/usr/bin/env python3
"""
evidence_pipeline.py - Automated evidence processing pipeline
"""

import os
import subprocess
import hashlib
import json
from datetime import datetime
from pathlib import Path
import logging

class EvidencePipeline:
    def __init__(self, base_dir="/home/forensic"):
        self.base_dir = Path(base_dir)
        self.evidence_dir = self.base_dir / "evidence"
        self.recovered_dir = self.base_dir / "recovered"
        self.logs_dir = self.base_dir / "logs"
        
        # Create directories
        for dir_path in [self.evidence_dir, self.recovered_dir, self.logs_dir]:
            dir_path.mkdir(parents=True, exist_ok=True)
        
        # Setup logging
        self.setup_logging()
    
    def setup_logging(self):
        log_file = self.logs_dir / f"pipeline_{datetime.now():%Y%m%d_%H%M%S}.log"
        logging.basicConfig(
            level=logging.INFO,
            format='%(asctime)s - %(levelname)s - %(message)s',
            handlers=[
                logging.FileHandler(log_file),
                logging.StreamHandler()
            ]
        )
        self.logger = logging.getLogger(__name__)
    
    def calculate_hash(self, filepath):
        """Calculate SHA-256 hash"""
        sha256_hash = hashlib.sha256()
        with open(filepath, "rb") as f:
            for byte_block in iter(lambda: f.read(4096), b""):
                sha256_hash.update(byte_block)
        return sha256_hash.hexdigest()
    
    def process_evidence(self, evidence_path, case_id):
        """Process a single evidence item"""
        self.logger.info(f"Processing evidence: {evidence_path}")
        
        # Create case directory
        case_dir = self.recovered_dir / case_id
        case_dir.mkdir(exist_ok=True)
        
        # Calculate hash
        evidence_hash = self.calculate_hash(evidence_path)
        self.logger.info(f"Evidence hash: {evidence_hash}")
        
        # Run RecoverJPEG
        output_dir = case_dir / "recoverjpeg"
        output_dir.mkdir(exist_ok=True)
        
        cmd = ["sudo", "recoverjpeg", "-o", str(output_dir), str(evidence_path)]
        result = subprocess.run(cmd, capture_output=True, text=True)
        
        if result.returncode != 0:
            self.logger.error(f"RecoverJPEG failed: {result.stderr}")
            return None
        
        # Count recovered files
        recovered_files = list(output_dir.glob("*.jpg"))
        
        # Generate metadata
        metadata = {
            "case_id": case_id,
            "evidence_path": str(evidence_path),
            "evidence_hash": evidence_hash,
            "recovery_timestamp": datetime.now().isoformat(),
            "files_recovered": len(recovered_files),
            "output_dir": str(output_dir)
        }
        
        # Save metadata
        metadata_file = case_dir / "recovery_metadata.json"
        with open(metadata_file, "w") as f:
            json.dump(metadata, f, indent=2)
        
        self.logger.info(f"Recovery complete: {len(recovered_files)} files")
        return metadata
    
    def batch_process(self, evidence_list):
        """Process multiple evidence items"""
        results = []
        
        for evidence_path, case_id in evidence_list:
            result = self.process_evidence(evidence_path, case_id)
            if result:
                results.append(result)
        
        return results

# Example usage
if __name__ == "__main__":
    pipeline = EvidencePipeline()
    
    evidence_list = [
        ("/dev/sdb1", "CASE-2024-001"),
        ("/home/evidence/usb_image.dd", "CASE-2024-002"),
    ]
    
    results = pipeline.batch_process(evidence_list)
    
    print(f"\nProcessed {len(results)} evidence items")
    for result in results:
        print(f"  {result['case_id']}: {result['files_recovered']} files")
```

---

## Chapter 6: Real-World Scenarios

### 6.1 Scenario: Digital Camera SD Card Recovery

**Background**: A photographer's SD card was accidentally formatted, losing hundreds of photos from a wedding shoot.

**Objective**: Recover all JPEG images from the formatted SD card.

**Steps**:

```bash
# Step 1: Identify the SD card
lsblk
# Output shows: /dev/mmcblk0p1 (16GB SD card)

# Step 2: Create forensic image (NEVER work on original)
sudo dd if=/dev/mmcblk0p1 of=~/cases/wedding_sdcard.dd bs=4096 status=progress

# Step 3: Verify the image
md5sum /dev/mmcblk0p1 ~/cases/wedding_sdcard.dd

# Step 4: Run RecoverJPEG
sudo recoverjpeg -o ~/cases/wedding/recovered ~/cases/wedding_sdcard.dd

# Step 5: Check results
ls -la ~/cases/wedding/recovered/
# Output: file00001.jpg through file00247.jpg (247 files)

# Step 6: Verify file integrity
file ~/cases/wedding/recovered/file00001.jpg
# Output: JPEG image data, JFIF standard 1.01

# Step 7: Generate report
echo "Wedding SD Card Recovery Report" > ~/cases/wedding/report.txt
echo "Date: $(date)" >> ~/cases/wedding/report.txt
echo "Files recovered: $(ls ~/cases/wedding/recovered/*.jpg | wc -l)" >> ~/cases/wedding/report.txt
```

**Results**: 247 JPEG files recovered, all with valid JPEG headers.

### 6.2 Scenario: Deleted Photos from Android Phone

**Background**: A user accidentally deleted photos from their Android phone's internal storage. The phone was rooted for forensic access.

**Objective**: Recover deleted JPEG photos from the phone's internal storage.

**Steps**:

```bash
# Step 1: Access phone storage via ADB
adb shell su -c "dd if=/dev/block/mmcblk0p28 of=/sdcard/backup.dd bs=4096"

# Step 2: Pull backup to computer
adb pull /sdcard/backup.dd ~/cases/android/

# Step 3: Run RecoverJPEG
sudo recoverjpeg -o ~/cases/android/recovered ~/cases/android/backup.dd

# Step 4: Analyze results
echo "Files recovered: $(ls ~/cases/android/recovered/*.jpg | wc -l)"

# Step 5: Check for EXIF data
exiftool ~/cases/android/recovered/file00001.jpg | grep -E "GPS|DateTime"
```

**Results**: 156 JPEG files recovered with EXIF metadata including GPS coordinates and timestamps.

### 6.3 Scenario: Corrupted Hard Drive Recovery

**Background**: A computer's hard drive developed bad sectors, making it impossible to mount. The user needs to recover family photos.

**Objective**: Recover JPEG images from a corrupted hard drive.

**Steps**:

```bash
# Step 1: Use safecopy to create a reliable image
sudo safecopy -s 0 /dev/sdb ~/cases/hdd/evidence.dd

# Step 2: Run RecoverJPEG on the image
sudo recoverjpeg -o ~/cases/hdd/recovered ~/cases/hdd/evidence.dd

# Step 3: Cross-reference with PhotoRec
sudo photorec /cmd ~/cases/hdd/evidence.dd fileopt,everything,disable,jpg,enable search ~/cases/hdd/photorec/

# Step 4: Compare results
RECOVERJEEP_COUNT=$(ls ~/cases/hdd/recovered/*.jpg 2>/dev/null | wc -l)
PHOTOREC_COUNT=$(find ~/cases/hdd/photorec -name "*.jpg" 2>/dev/null | wc -l)

echo "RecoverJPEG: $RECOVERJEEP_COUNT files"
echo "PhotoRec: $PHOTOREC_COUNT files"

# Step 5: Merge unique files
# Create a script to deduplicate based on content
python3 << 'EOF'
import os
import hashlib

def file_hash(filepath):
    with open(filepath, 'rb') as f:
        return hashlib.md5(f.read()).hexdigest()

recoverjpeg_dir = os.path.expanduser("~/cases/hdd/recovered")
photorec_dir = os.path.expanduser("~/cases/hdd/photorec")

# Get hashes from RecoverJPEG
rj_hashes = {}
for f in os.listdir(recoverjpeg_dir):
    if f.endswith('.jpg'):
        rj_hashes[file_hash(os.path.join(recoverjpeg_dir, f))] = f

# Find unique files in PhotoRec
unique_files = []
for root, dirs, files in os.walk(photorec_dir):
    for f in files:
        if f.endswith('.jpg'):
            filepath = os.path.join(root, f)
            h = file_hash(filepath)
            if h not in rj_hashes:
                unique_files.append(filepath)

print(f"Unique files from PhotoRec: {len(unique_files)}")
EOF
```

**Results**: RecoverJPEG found 89 files, PhotoRec found 102 files, with 12 unique to PhotoRec. Total unique recovery: 101 files.

---

## Chapter 7: Detection & Defense

### 7.1 Forensic Artifacts

When RecoverJPEG is used, it leaves several artifacts:

```bash
# Log files (if logging is enabled)
/var/log/syslog
~/.bash_history

# Recovered files
~/recovered/*.jpg

# Temporary files
/tmp/recoverjpeg.*
```

### 7.2 Anti-Forensics Considerations

To prevent JPEG recovery:

1. **Secure deletion**: Use `shred` or `secure-delete` tools
2. **Full disk encryption**: Encrypt storage before use
3. **Wiping free space**: Use `sfill` or `dd` to overwrite free space

```bash
# Securely delete a file
shred -vfz -n 3 sensitive_photo.jpg

# Wipe free space on a partition
sudo dd if=/dev/zero of=/dev/sdb1 bs=4096
sudo dd if=/dev/urandom of=/dev/sdb1 bs=4096

# Using sfill (more secure)
sudo apt install secure-delete
sudo sfill /mount/point/
```

### 7.3 Detecting JPEG Recovery Attempts

Forensic analysts can detect if RecoverJPEG was used:

```bash
# Check command history
history | grep recoverjpeg

# Look for recovered files
find / -name "file0*.jpg" -type f 2>/dev/null

# Check for recovery directories
find / -type d -name "recovered" 2>/dev/null

# Analyze file timestamps
stat recovered_file.jpg
```

### 7.4 Preventing Unauthorized Recovery

```bash
# Disable USB storage (prevent unauthorized device access)
echo "blacklist usb-storage" | sudo tee /etc/modprobe.d/blacklist-usb-storage.conf

# Monitor for device connections
tail -f /var/log/syslog | grep -i "usb\|sd"

# Set up audit rules
sudo auditctl -w /dev/sd* -p rwxa -k usb_access
```

---

## Chapter 8: Troubleshooting

### 8.1 Frequently Asked Questions

**Q1: RecoverJPEG says "No JPEG files found" but I know there are photos on the device.**

A: This can happen if:
- The device is formatted with a filesystem RecoverJPEG doesn't recognize
- The JPEG files are fragmented across the disk
- The file headers are corrupted

Try using PhotoRec or Scalpel as alternatives, or use `dd` to create a raw image first:

```bash
sudo dd if=/dev/sdb1 of=image.dd bs=4096
sudo recoverjpeg -o output/ image.dd
```

**Q2: Recovered files are corrupted or can't be opened.**

A: This usually indicates:
- The JPEG file was fragmented (only partial data recovered)
- The file header was corrupted before deletion
- The file was overwritten partially

Try these solutions:
```bash
# Check file integrity
file recovered_file.jpg

# Use jpeghfix to attempt repair
sudo apt install jpeginfo
jpeginfo -c recovered_file.jpg

# Try alternative recovery tools
sudo photorec image.dd
```

**Q3: RecoverJPEG runs very slowly.**

A: Performance can be improved by:
- Using a faster storage device for output
- Running from a RAM disk for temporary files
- Reducing I/O contention

```bash
# Create RAM disk for output
sudo mkdir -p /tmp/ramdisk
sudo mount -t tmpfs -o size=2G tmpfs /tmp/ramdisk

# Run recovery to RAM disk
sudo recoverjpeg -o /tmp/ramdisk /dev/sdb1

# Copy results when complete
cp -r /tmp/ramdisk/* ~/recovered/
sudo umount /tmp/ramdisk
```

**Q4: How do I recover JPEG files from a specific time range?**

A: RecoverJPEG doesn't have time-based filtering, but you can post-process results:

```bash
# Recover all files first
sudo recoverjpeg -o output/ image.dd

# Filter by modification time
find output/ -name "*.jpg" -newermt "2024-01-01" ! -newermt "2024-12-31"

# Use exiftool to filter by EXIF date
exiftool -DateTimeOriginal -filename output/*.jpg | grep "2024"
```

**Q5: Can RecoverJPEG recover RAW camera files (CR2, NEF, ARW)?**

A: No, RecoverJPEG only recovers JPEG files. For RAW files, use:

```bash
# PhotoRec supports many RAW formats
sudo photorec image.dd

# Or use specific RAW recovery tools
sudo apt install dc3dd
sudo dc3dd if=/dev/sdb1 hof=raw_files.raw
```

**Q6: How do I verify that recovered files are authentic and not planted?**

A: Forensic integrity verification:

```bash
# Generate hash of original evidence
md5sum /dev/sdb1 > evidence.md5

# Verify hash before recovery
md5sum -c evidence.md5

# Document the chain of custody
echo "Evidence handled by: $(whoami)" > chain_of_custody.txt
echo "Date: $(date)" >> chain_of_custody.txt
echo "Command executed: recoverjpeg -o output/ /dev/sdb1" >> chain_of_custody.txt
```

**Q7: RecoverJPEG failed with "Permission denied" error.**

A: Ensure you have root privileges:

```bash
# Use sudo
sudo recoverjpeg /dev/sdb1

# Or become root
su -
recoverjpeg /dev/sdb1

# Check device permissions
ls -la /dev/sdb1
```

### 8.2 Common Error Messages

| Error | Cause | Solution |
|-------|-------|----------|
| `No such device` | Device doesn't exist | Check `lsblk` for correct device name |
| `Permission denied` | Insufficient privileges | Use `sudo` |
| `No JPEG files found` | No recoverable JPEGs | Try alternative tools or check device |
| `Read error` | Bad sectors on device | Use `safecopy` first to create reliable image |
| `No space left on device` | Output disk full | Free space or use different output location |

### 8.3 Performance Optimization

```bash
# Increase I/O priority
sudo ionice -c2 -n0 -p$$ recoverjpeg -o output/ /dev/sdb1

# Use direct I/O to bypass cache
sudo recoverjpeg -O direct -o output/ /dev/sdb1

# Monitor performance
iostat -x 1
```

---

## Resources

### Official Documentation

- [RecoverJPEG GitHub Repository](https://github.com/lahanas/recoverjpeg)
- [Kali Linux Tools - RecoverJPEG](https://www.kali.org/tools/recoverjpeg/)
- [JPEG File Format Specification](https://www.w3.org/Graphics/JPEG/itu-t81.pdf)

### Related Tools

- [PhotoRec](https://www.cgsecurity.org/wiki/PhotoRec) - Multi-format file recovery
- [Foremost](https://www.kali.org/tools/foremost/) - File carving tool
- [Scalpel](https://github.com/sleuthkit/scalpel) - Fast file carving
- [Safecopy](https://github.com/snicca/safecopy) - Data recovery with bad sectors

### Books and References

- "File System Forensic Analysis" by Brian Carrier
- "Digital Forensics with Kali Linux" by Shiva V. Parasram
- "The Art of Memory Forensics" by Michael Hale Ligh

### Online Resources

- [SANS Digital Forensics](https://www.sans.org/dfir/)
- [Forensic Focus](https://www.forensicfocus.com/)
- [Reddit r/forensics](https://www.reddit.com/r/forensics/)

### Community and Support

- [Kali Linux Forums](https://forums.kali.org/)
- [Sleuth Kit mailing list](https://www.sleuthkit.org/)
- [Digital Forensics Discord](https://discord.gg/forensics)

---

**Document Version**: 1.0
**Last Updated**: 2024
**Author**: Kali Linux Forensic Documentation Team
**Classification**: Public
