# MissIdentify — File Identification Tool

## Table of Contents

- [Overview](#overview)
- [Installation](#installation)
- [Core Concepts](#core-concepts)
- [Command-Line Reference](#command-line-reference)
- [Basic Usage](#basic-usage)
- [Advanced Usage](#advanced-usage)
- [Forensics Workflow Integration](#forensics-workflow-integration)
- [Limitations and Considerations](#limitations-and-considerations)
- [Detection and Defense](#detection-and-defense)
- [Troubleshooting](#troubleshooting)
- [Best Practices](#best-practices)
- [Comparison with Other Tools](#comparison-with-other-tools)
- [References](#references)

---

## Overview

MissIdentify is a command-line tool designed to identify files by their magic bytes (file signatures) rather than their file extensions. In digital forensics, attackers often disguise malicious files by changing file extensions. MissIdentify helps analysts identify the true file type regardless of the extension, making it an essential tool for file identification and malware analysis.

MissIdentify scans files and compares their headers against a database of known file signatures. This allows it to identify files even when they have been renamed or have incorrect extensions. It is particularly useful for identifying disguised malware, analyzing file systems, and verifying file types in forensic investigations.

### Key Features

- **Magic Byte Identification**: Identifies files by their headers
- **Extension Mismatch Detection**: Finds files with wrong extensions
- **Batch Processing**: Can scan multiple files or directories
- **Custom Signatures**: Supports custom file signature databases
- **Non-Destructive**: Read-only analysis
- **Cross-Platform**: Works on Linux and other Unix-like systems

### When to Use MissIdentify

| Scenario | Use MissIdentify? | Notes |
|----------|------------------|-------|
| Identifying disguised files | Yes | Primary use case |
| Detecting extension mismatches | Yes | Finds renamed malware |
| Batch file analysis | Yes | Process multiple files |
| Malware triage | Yes | Quick identification |
| File system analysis | Yes | Verify file types |
| Recovered file identification | Yes | Identify carved files |

---

## Installation

### Kali Linux

```bash
# MissIdentify may need to be installed manually
sudo apt update
sudo apt install missidentify

# If not in repositories, build from source
```

### Other Linux Distributions

```bash
# Debian/Ubuntu
sudo apt install missidentify

# From source
git clone https://github.com/cr-marcstevens/missidentify.git
cd missidentify
make
sudo make install
```

### Building from Source

```bash
# Install build dependencies
sudo apt install build-essential git

# Clone repository
git clone https://github.com/cr-marcstevens/missidentify.git
cd missidentify

# Build
make

# Install
sudo make install

# Verify
missidentify --version
```

---

## Core Concepts

### Magic Bytes

Every file type has a unique header signature (magic bytes):

| File Type | Magic Bytes (Hex) | ASCII |
|-----------|-------------------|-------|
| JPEG | `FF D8 FF E0` | ÿØÿà |
| PNG | `89 50 4E 47` | .PNG |
| PDF | `25 50 44 46` | %PDF |
| ZIP | `50 4B 03 04` | PK.. |
| GIF | `47 49 46 38` | GIF8 |
| ELF | `7F 45 4C 46` | .ELF |
| PE | `4D 5A` | MZ |

### Extension Mismatches

MissIdentify detects when file extensions don't match actual content:

```
file.exe → Actually a JPEG image
file.jpg → Actually a PDF document
file.pdf → Actually a ZIP archive
```

### Forensic Value

File identification helps:

- **Detect malware**: Malware often disguised as benign files
- **Verify evidence**: Ensure files are what they claim to be
- **Recover files**: Identify carved files without extensions
- **Analyze file systems**: Understand actual file types

---

## Command-Line Reference

### Basic Syntax

```bash
missidentify [options] <file_or_directory>
```

### Options

| Option | Description |
|--------|-------------|
| `-h`, `--help` | Show help message |
| `-v`, `--version` | Show version |
| `-r`, `--recursive` | Scan directories recursively |
| `-q`, `--quiet` | Quiet mode |
| `-l`, `--list` | List all file types |
| `-a`, `--all` | Show all matches |
| `-x`, `--extension` | Check extension matches |
| `-s`, `--signature` | Show file signatures |

---

## Basic Usage

### Identify Single File

```bash
# Identify file type
missidentify /path/to/file.exe

# Check extension mismatch
missidentify -x /path/to/file.exe
```

### Scan Directory

```bash
# Scan directory recursively
missidentify -r /path/to/files/

# Find all mismatches
missidentify -r -x /path/to/files/
```

### Batch Processing

```bash
# Scan all files in current directory
missidentify *

# Find suspicious files
missidentify -r -x /evidence/files/ | grep -v "OK"
```

---

## Advanced Usage

### Command-Line Scenarios

#### Scenario 1: Malware Triage

```bash
#!/bin/bash
# Triage files for disguised malware

DIRECTORY="/evidence/suspicious_files"
REPORT="/evidence/triage_report.txt"

echo "File Triage Report" > "$REPORT"
echo "Date: $(date)" >> "$REPORT"
echo "" >> "$REPORT"

# Find all extension mismatches
missidentify -r -x "$DIRECTORY" > "$REPORT.tmp"

# Analyze results
echo "Extension Mismatches:" >> "$REPORT"
grep -v "OK" "$REPORT.tmp" >> "$REPORT"
echo "" >> "$REPORT"

# Count by type
echo "File Types Found:" >> "$REPORT"
missidentify -r "$DIRECTORY" | awk '{print $2}' | sort | uniq -c | sort -rn >> "$REPORT"

rm "$REPORT.tmp"
```

#### Scenario 2: Recovered File Identification

```bash
# Identify carved files without extensions
for file in /evidence/carved_files/*; do
    TYPE=$(missidentify "$file" | awk '{print $2}')
    echo "$file: $TYPE"
done

# Rename files based on actual type
for file in /evidence/carved_files/*; do
    TYPE=$(missidentify "$file" | awk '{print $2}')
    if [ ! -z "$TYPE" ]; then
        mv "$file" "${file}.${TYPE}"
    fi
done
```

#### Scenario 3: Batch Analysis

```bash
#!/bin/bash
# Comprehensive file analysis

DIRECTORY="/evidence/files"
OUTPUT="/evidence/analysis"

mkdir -p "$OUTPUT"

# Find all mismatches
echo "Finding extension mismatches..."
missidentify -r -x "$DIRECTORY" > "$OUTPUT/mismatches.txt"

# Find all file types
echo "Identifying file types..."
missidentify -r "$DIRECTORY" > "$OUTPUT/types.txt"

# Generate summary
echo "File Analysis Summary" > "$OUTPUT/summary.txt"
echo "Total files: $(find $DIRECTORY -type f | wc -l)" >> "$OUTPUT/summary.txt"
echo "Mismatches: $(grep -v "OK" $OUTPUT/mismatches.txt | wc -l)" >> "$OUTPUT/summary.txt"
```

#### Scenario 4: Forensic File Verification

```bash
#!/bin/bash
# Verify file types in evidence

EVIDENCE="/evidence/files"
REPORT="/evidence/verification.txt"

echo "File Verification Report" > "$REPORT"
echo "Date: $(date)" >> "$REPORT"
echo "" >> "$REPORT"

find "$EVIDENCE" -type f | while read file; do
    # Get declared type from extension
    DECLARED=$(file -b --mime-type "$file" | cut -d'/' -f2)
    
    # Get actual type from magic bytes
    ACTUAL=$(missidentify "$file" | awk '{print $2}')
    
    # Compare
    if [ "$DECLARED" = "$ACTUAL" ]; then
        echo "OK: $file" >> "$REPORT"
    else
        echo "MISMATCH: $file (declared: $DECLARED, actual: $ACTUAL)" >> "$REPORT"
    fi
done
```

### Analyzing Results

```bash
# Count mismatches
missidentify -r -x /evidence/ | grep -v "OK" | wc -l

# List all JPEG files
missidentify -r /evidence/ | grep "JPEG"

# Find executable files
missidentify -r /evidence/ | grep -E "PE|ELF"

# Find documents
missidentify -r /evidence/ | grep -E "PDF|DOC|XLS"
```

---

## Forensics Workflow Integration

### Phase 1: Evidence Acquisition

```bash
# 1. Locate files on suspect system
find /mnt/evidence/ -type f

# 2. Copy files (preserve metadata)
cp --preserve=all -r /mnt/evidence/Users/*/Documents/* \
    /evidence/files/

# 3. Calculate hashes
find /evidence/files/ -type f -exec md5sum {} \; > /evidence/file_hashes.txt
```

### Phase 2: Analysis with MissIdentify

```bash
# 1. Create analysis directory
mkdir -p /evidence/analysis/{mismatches,types,reports}

# 2. Identify all file types
missidentify -r /evidence/files/ > /evidence/analysis/types.txt

# 3. Find extension mismatches
missidentify -r -x /evidence/files/ > /evidence/analysis/mismatches/mismatches.txt

# 4. Extract suspicious files
grep -v "OK" /evidence/analysis/mismatches/mismatches.txt | \
    awk '{print $1}' > /evidence/analysis/suspicious_files.txt
```

### Phase 3: Post-Analysis

```bash
# 1. Generate comprehensive report
cat > /evidence/analysis/summary.txt << EOF
File Identification Summary
===========================
Date: $(date)
Total Files: $(find /evidence/files/ -type f | wc -l)
Extension Mismatches: $(grep -v "OK" /evidence/analysis/mismatches/mismatches.txt | wc -l)
Suspicious Files: $(wc -l < /evidence/analysis/suspicious_files.txt)
EOF

# 2. Categorize by file type
missidentify -r /evidence/files/ | awk '{print $2}' | sort | uniq -c | sort -rn > /evidence/analysis/type_summary.txt

# 3. Generate file hashes for suspicious files
while read file; do
    md5sum "$file"
done < /evidence/analysis/suspicious_files.txt > /evidence/analysis/suspicious_hashes.txt
```

### Integration with Other Tools

```bash
# MissIdentify + YARA
# Scan suspicious files with YARA rules
while read file; do
    yara -r /path/to/rules/ "$file"
done < /evidence/analysis/suspicious_files.txt

# MissIdentify + ClamAV
# Scan suspicious files with antivirus
clamscan -i -l /evidence/analysis/clamscan.log -f /evidence/analysis/suspicious_files.txt

# MissIdentify + strings
# Extract strings from suspicious files
while read file; do
    echo "=== $file ==="
    strings "$file" | head -20
done < /evidence/analysis/suspicious_files.txt

# MissIdentify + pdf-parser
# Analyze suspicious PDFs
while read file; do
    if [[ "$file" == *.pdf ]]; then
        python3 pdf-parser.py "$file"
    fi
done < /evidence/analysis/suspicious_files.txt
```

---

## Limitations and Considerations

### What MissIdentify Cannot Do

1. **Execute files**: Read-only analysis
2. **Analyze content**: Only identifies file types
3. **Detect all malware**: May miss sophisticated malware
4. **Repair files**: Cannot fix corrupted files
5. **Decrypt files**: Cannot bypass encryption

### Known Limitations

- **Limited file types**: May not recognize all formats
- **False positives**: Some matches may be incorrect
- **No content analysis**: Only identifies headers
- **No GUI**: Command-line only
- **Limited documentation**: Fewer resources than mainstream tools

### When to Use Alternatives

| Situation | Better Tool | Reason |
|-----------|-------------|--------|
| Need content analysis | strings, hexdump | More detail |
| Need malware detection | ClamAV, YARA | Malware-specific |
| Need file carving | scalpel, foremost | File recovery |
| Need GUI | Autopsy | Full interface |

---

## Detection and Defense

### For System Administrators

```bash
# Monitor for MissIdentify usage
which missidentify

# Check for unauthorized file analysis
sudo ausearch -k missidentify -ts recent

# Monitor file type changes
find / -type f -mtime -1 2>/dev/null

# Check command history
cat ~/.bash_history | grep -i "missidentify"
```

### For Forensic Investigators

```bash
# Check if MissIdentify was used as anti-forensics tool
# Look for file manipulation
find / -type f -mtime -1 2>/dev/null

# Check command history
cat /root/.bash_history | grep -i "missidentify"

# Examine system logs
sudo journalctl | grep -i "missidentify"
```

### Defensive Measures

```bash
# 1. Enable file type validation
# Configure applications to verify file types

# 2. Use application whitelisting
# Only allow approved file types

# 3. Monitor for extension changes
# Alert on suspicious file operations

# 4. Regular security scans
# Check for disguised malware

# 5. User education
# Train users to recognize suspicious files
```

---

## Troubleshooting

### Common Issues and Solutions

#### "Permission denied"

```bash
# Check file permissions
ls -la /path/to/file

# Fix permissions
chmod 644 /path/to/file

# Run with sudo if needed
sudo missidentify /path/to/file
```

#### "File not found"

```bash
# Check if file exists
ls -la /path/to/file

# Check for typos
ls -la /path/to/fil*

# Use absolute path
missidentify /full/path/to/file
```

#### "No output"

```bash
# Check if file is readable
file /path/to/file

# Try with verbose mode
missidentify -v /path/to/file

# Check file size
ls -lh /path/to/file
```

---

## Best Practices

### For Forensic Investigations

1. **Always work on copies**: Never analyze original evidence
2. **Preserve metadata**: Use `cp --preserve=all` when copying
3. **Document everything**: Record commands and outputs
4. **Verify hashes**: MD5/SHA256 before and after
5. **Combine with other tools**: Use MissIdentify alongside YARA and ClamAV

### For File Identification

1. **Check all files**: Don't assume extensions are correct
2. **Focus on mismatches**: Extension mismatches are suspicious
3. **Verify manually**: Double-check suspicious findings
4. **Create reports**: Document all findings
5. **Archive results**: Keep identification logs

### Configuration Tips

```bash
# 1. Start with recursive scan
# 2. Focus on extension mismatches
# 3. Use verbose mode for debugging
# 4. Document all commands
# 5. Verify results with file command
```

---

## Comparison with Other Tools

### MissIdentify vs file vs TrID

| Feature | MissIdentify | file | TrID |
|---------|--------------|------|------|
| Purpose | Mismatch detection | General identification | General identification |
| Extension Check | Yes | No | No |
| Batch Processing | Yes | Yes | Yes |
| Custom Signatures | Limited | Yes | Yes |
| Speed | Fast | Fast | Fast |
| Ease of Use | Easy | Easy | Easy |

### When to Choose MissIdentify

- When you need to find extension mismatches
- When analyzing recovered files
- When doing initial file triage
- As a quick identification tool

---

## References

- **MissIdentify Documentation**: https://github.com/cr-marcstevens/missidentify
- **Kali Linux Forensics**: https://www.kali.org/tools/missidentify/
- **File Signature Database**: https://en.wikipedia.org/wiki/List_of_file_signatures
- **Digital Forensics Framework**: https://www.digitalforensicsframework.org/
- **Malware Analysis Guide**: https://www.sans.org/reading-room/malware-analysis-techniques-1556

---

## Quick Reference Card

| Operation | Command |
|-----------|---------|
| Identify file | `missidentify file.exe` |
| Check mismatches | `missidentify -x file.exe` |
| Scan directory | `missidentify -r /path/to/dir/` |
| Find all mismatches | `missidentify -r -x /path/to/dir/` |
| List file types | `missidentify -l` |
| Show signatures | `missidentify -s file.exe` |
| Quiet mode | `missidentify -q file.exe` |

---

*Document Version: 1.0*
*Last Updated: 2026*
*Author: Digital Forensics Documentation Project*
*Classification: Educational*
