---
tool_name: "autopsy"
display_name: "Autopsy"
mitre_tactic: "forensics"
mitre_tactic_id: "TA0007"
mitre_subcategory: "digital_forensics"
install_command: "sudo apt install autopsy"
version: "4.21.0"
github_repo: "https://github.com/sleuthkit/autopsy"
kali_tools_page: "https://www.kali.org/tools/autopsy/"
difficulty_level: "intermediate-to-advanced"
requires_root: false
gui_available: true
tags: ["forensics", "disk", "filesystem", "timeline", "investigation"]
related_tools: ["sleuthkit", "volatility", "binwalk"]
---

# Autopsy

## Chapter 1: Introduction & Overview

### What is Autopsy?
Autopsy is a digital forensics platform and graphical interface to The Sleuth Kit (TSK). It's used to analyze disk images, file systems, and perform forensic investigations.

### History & Background
- **Created:** 2003 by Basis Technology
- **Maintained:** Basis Technology / Sleuth Kit
- **License:** Apache License 2.0
- **Purpose:** Digital forensics investigation

Autopsy is the industry standard for disk forensics.

### When to Use This Tool
- **Incident response:** Analyze compromised systems
- **Digital forensics:** Recover deleted files
- **Malware analysis:** Analyze suspicious files
- **Law enforcement:** Criminal investigations

### Key Concepts
- **Disk Image:** Forensic copy of storage device
- **File System:** FAT, NTFS, ext4, etc.
- **Timeline:** Chronological event list
- **Hash:** File integrity verification
- **Metadata:** File system metadata

---

## Chapter 2: Installation & Setup

### System Requirements
- **RAM:** 8 GB minimum (16 GB+ recommended)
- **Disk Space:** 1 GB
- **Java:** 17+

### Installation Methods

#### Method 1: APT (Recommended)
```bash
sudo apt update
sudo apt install autopsy
```

#### Method 2: From Source
```bash
git clone https://github.com/sleuthkit/autopsy.git
cd autopsy
./gradlew build
```

### Verification
```bash
autopsy
```

---

## Chapter 3: Basic Usage

### First Analysis

#### Start Autopsy
```bash
autopsy
```
**Explanation:** Opens web interface at http://localhost:9999/autopsy

#### Create Case
1. Click "New Case"
2. Enter case name
3. Set case directory
4. Click "Create"

#### Add Data Source
1. Click "Add Data Source"
2. Select source type (Disk Image, Local Disk, etc.)
3. Browse to file
4. Click "Add"

### Essential Commands

#### Command Line Interface
```bash
# Analyze disk image
fls -r image.dd > file_list.txt

# Recover deleted files
icat image.dd inode > recovered_file

# Create timeline
fls -r -m "/" image.dd > timeline.txt
```

### Complete Tool Reference

#### Sleuth Kit Tools
| Tool | Description | Example |
|------|-------------|---------|
| `fls` | File listing | `fls -r image.dd` |
| `icat` | Inode cat | `icat image.dd inode > file` |
| `ls` | List files | `ls image.dd` |
| `stat` | File stats | `stat image.dd inode` |
| `mmls` | Partition list | `mmls image.dd` |
| `mmstat` | Partition stats | `mmstat image.dd` |
| `fsstat` | Filesystem info | `fsstat image.dd` |
| `tsk_recover` | Recover files | `tsk_recover image.dd output/` |

#### Autopsy Modules
| Module | Description | Usage |
|--------|-------------|-------|
| **File Analyze** | File analysis | Automatic |
| **Hash Lookup** | Known file check | Automatic |
| **Keyword Search** | Text search | Manual |
| **File Type ID** | File identification | Automatic |
| **Interesting Items** | Flagged items | Automatic |
| **Email Parser** | Email analysis | Automatic |
| **Registry Analyzer** | Windows registry | Automatic |
| **Timeline** | Time-based analysis | Manual |
| **Web Artifacts** | Browser data | Automatic |
| **Hash Database** | NSRL, custom | Automatic |

---

## Chapter 4: Intermediate Usage

### Advanced Analysis Techniques

#### Timeline Analysis
```bash
# Create timeline
fls -r -m "/" image.dd > timeline.txt

# Convert to timeline format
mactime -b timeline.txt -d > timeline.csv

# Analyze with date range
mactime -b timeline.txt 2024-01-01..2024-01-31
```

#### File Recovery
```bash
# List deleted files
fls -r -d image.dd

# Recover specific file
icat image.dd inode > recovered_file

# Recover all deleted files
tsk_recover image.dd recovered/
```

### Scripting & Automation

#### Bash Scripting
```bash
#!/bin/bash
# Automated forensic analysis
IMAGE="evidence.dd"
OUTPUT="analysis_$(date +%Y%m%d_%H%M%S)"
mkdir -p $OUTPUT

echo "[*] Analyzing $IMAGE"

# List files
fls -r $IMAGE > $OUTPUT/file_list.txt

# Find deleted files
fls -r -d $IMAGE > $OUTPUT/deleted_files.txt

# Create timeline
fls -r -m "/" $IMAGE > $OUTPUT/timeline.txt
mactime -b $OUTPUT/timeline.txt -d > $OUTPUT/timeline.csv

# Recover deleted files
tsk_recover $IMAGE $OUTPUT/recovered/

echo "[+] Analysis complete: $OUTPUT"
```

#### Python Scripting
```python
#!/usr/bin/env python3
import subprocess
import os

def analyze_image(image_path, output_dir):
    """Run forensic analysis"""
    os.makedirs(output_dir, exist_ok=True)
    
    # File listing
    cmd = f"fls -r {image_path}"
    result = subprocess.run(cmd, shell=True, capture_output=True, text=True)
    
    with open(f"{output_dir}/file_list.txt", "w") as f:
        f.write(result.stdout)
    
    return result.stdout

# Analyze image
analyze_image("evidence.dd", "analysis")
```

### Combining with Other Tools

#### With Volatility
```bash
# Analyze disk with Autopsy
# Analyze memory with Volatility
volatility -f memory.raw --profile=Win7SP1x64 pslist
```

#### With YARA
```bash
# Find suspicious files
find recovered/ -type f | xargs yara -r malware_rules.yar
```

### Advanced Configurations

#### Custom Hash Databases
```bash
# Create custom hash database
hfind -e evidence.dd

# Add to Autopsy
# Tools → Options → Hash Databases → Add
```

### Performance Optimization

#### Large Images
```bash
# Use specific modules
fls -r evidence.dd

# Limit analysis
fls -r -m "/" evidence.dd | head -1000
```

---

## Chapter 5: Advanced Usage

### Advanced Analysis Techniques

#### Registry Analysis
```bash
# Extract registry hives
fls -r evidence.dd | grep -i "system\|sam\|software\|security"

# Analyze with regripper
regripper -r SYSTEM -r SAM -r SOFTWARE -r SECURITY
```

#### Web Artifact Analysis
```bash
# Find browser history
find recovered/ -name "*.sqlite" -exec sqlite3 {} ".tables" \;

# Extract cookies
find recovered/ -name "cookies*" -exec cat {} \;
```

### Integration in Pentest Workflow

#### Phase 1: Acquisition
```bash
# Create forensic image
dd if=/dev/sda of=evidence.dd bs=4M
# or
dcfldd if=/dev/sda of=evidence.dd bs=4M hash=sha256
```

#### Phase 2: Analysis
```bash
# Open in Autopsy
autopsy

# Analyze file system
fls -r evidence.dd

# Find deleted files
fls -r -d evidence.dd

# Create timeline
fls -r -m "/" evidence.dd > timeline.txt
```

#### Phase 3: Reporting
```bash
# Generate report
# Autopsy → Generate Report

# Or use command line
fls -r evidence.dd > report.txt
```

### Advanced Configurations

#### Custom Modules
```bash
# Create custom Autopsy module
# See Autopsy documentation for API
```

### Performance Optimization

#### Parallel Analysis
```bash
# Run multiple analyses
fls -r evidence.dd > files.txt &
mmls evidence.dd > partitions.txt &
wait
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: CTF Challenge
**Objective:** Find hidden evidence

**Steps:**
```bash
# 1. Open in Autopsy
autopsy

# 2. Add evidence
# 3. Browse file system
# 4. Check deleted files
# 5. Search for keywords
# 6. Analyze timeline
```

### Scenario 2: Incident Response
**Objective:** Analyze compromised system

**Steps:**
```bash
# 1. Create forensic image
dd if=/dev/sda of=evidence.dd bs=4M

# 2. Open in Autopsy
autopsy

# 3. Analyze file system
# 4. Check for malware
# 5. Review timeline
# 6. Extract evidence
```

### Scenario 3: Digital Forensics
**Objective:** Recover deleted files

**Steps:**
```bash
# 1. List deleted files
fls -r -d evidence.dd

# 2. Recover files
tsk_recover evidence.dd recovered/

# 3. Analyze recovered files
# 4. Document findings
```

---

## Chapter 7: Detection & Defense

### How Defenders Detect Forensic Analysis

#### System Indicators
```
# Disk image creation
# Unusual file access
# Timeline analysis
```

#### Log Analysis
```bash
# Monitor for disk access
# Check for image creation
# Alert on forensic tools
```

### Mitigation Strategies

#### Anti-Forensics
```bash
# Use encryption
# Secure delete files
# Clear logs
```

#### Detection
```bash
# Monitor disk access
# Detect forensic tools
# Alert on anomalies
```

### Blue Team Response
1. **Encrypt sensitive data** - Prevent recovery
2. **Secure delete files** - Remove traces
3. **Monitor disk access** - Detect analysis
4. **Use anti-forensics** - Hide evidence
5. **Regular audits** - Find issues early

---

## Chapter 8: Troubleshooting

### Common Errors

#### Error: "Image not found"
**Cause:** Wrong path
**Solution:**
```bash
# Check path
ls -la evidence.dd

# Use full path
autopsy
```

#### Error: "Permission denied"
**Cause:** Insufficient permissions
**Solution:**
```bash
# Run as root
sudo autopsy

# Or fix permissions
sudo chmod 644 evidence.dd
```

### Performance Optimization

#### Slow Analysis
```bash
# Use specific tools
fls -r evidence.dd

# Limit output
fls -r evidence.dd | head -50
```

### FAQ

**Q: Is autopsy legal to use?**
A: Yes, autopsy is a legitimate forensic tool. However, analyzing systems without authorization is illegal. Always get written permission before analysis.

**Q: How do I update autopsy?**
A: `sudo apt update && sudo apt upgrade autopsy`

**Q: What file systems does Autopsy support?**
A: FAT, NTFS, ext2/3/4, HFS+, and more.

**Q: Can I recover deleted files?**
A: Yes, use `tsk_recover` or Autopsy's built-in recovery.

**Q: How do I create a forensic image?**
A: Use `dd`, `dcfldd`, or `Guymager`.

---

## Resources

### Official Documentation
- [Autopsy Homepage](https://www.autopsy.com/)
- [GitHub Repository](https://github.com/sleuthkit/autopsy)
- [Autopsy Documentation](https://sleuthkit.github.io/autopsy-docs/)

### Tutorials
- [Autopsy Tutorial - HackTricks](https://book.hacktricks.xyz/generic-methodologies-and-resources/forensics)
- [Autopsy Cheat Sheet](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Forensic.md)

### Related Tools
- **Sleuth Kit:** Command-line forensic tools
- **Volatility:** Memory forensics
- **Binwalk:** Firmware analysis
- **Foremost:** File carving

### Practice Resources
- [DFIR Training](https://www.dfir.training/)
- [NIST Forensics](https://www.nist.gov/itl/ssd/software-quality-group/computer-forensics-tool-testing-program-cftt)
- [HackTheBox](https://www.hackthebox.com/)
- [TryHackMe](https://tryhackme.com/)
