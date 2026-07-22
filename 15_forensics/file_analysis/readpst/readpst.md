# ReadPST — Outlook PST/OST Reader

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

ReadPST (part of the libpff project) is a command-line tool designed to read and extract data from Microsoft Outlook PST (Personal Storage Table) and OST (Offline Storage Table) files. PST files are used by Microsoft Outlook to store emails, contacts, calendar entries, and other data locally. OST files are offline copies of Exchange mailbox data.

ReadPST is one of the most reliable and widely used tools for forensic analysis of Outlook data files. It can extract emails, attachments, contacts, calendar entries, and notes from PST/OST files, preserving the original structure and metadata. Unlike some tools, ReadPST supports both ANSI (Outlook 97-2002) and Unicode (Outlook 2003+) formats.

### Key Features

- **PST/OST Support**: Reads both PST and OST file formats
- **Unicode and ANSI**: Supports both modern and legacy formats
- **Complete Extraction**: Emails, attachments, contacts, calendar
- **Non-Destructive**: Read-only analysis, preserves original data
- **Cross-Platform**: Works on Linux, macOS, and Windows
- **Forensic Standard**: Widely accepted in forensic communities

### When to Use ReadPST

| Scenario | Use ReadPST? | Notes |
|----------|-------------|-------|
| Analyzing Outlook PST files | Yes | Primary use case |
| Reading OST files | Yes | Exchange offline cache |
| Extracting email metadata | Yes | Headers, timestamps |
| E-discovery investigations | Yes | Good for email review |
| Incident response | Yes | Analyzing suspect email |
| Corrupted PST files | Partial | May recover some data |
| Encrypted PST files | No | Cannot decrypt |

---

## Installation

### Kali Linux

```bash
# ReadPST is pre-installed on Kali Linux
sudo apt update
sudo apt install readpst libpff-utils

# Verify installation
readpst -v
```

### Other Linux Distributions

```bash
# Debian/Ubuntu
sudo apt install readpst libpff-utils

# Fedora/RHEL
sudo dnf install libpff-utils

# From source
git clone https://github.com/libyal/libpff.git
cd libpff
./configure
make
sudo make install
```

### Building from Source

```bash
# Install dependencies
sudo apt install build-essential git autotools-dev

# Clone repository
git clone https://github.com/libyal/libpff.git
cd libpff

# Build
./autogen.sh
./configure
make -j$(nproc)

# Install
sudo make install

# Update library cache
sudo ldconfig

# Verify
readpst -v
```

---

## Core Concepts

### PST File Structure

PST files contain:

- **Email Messages**: Complete email content with headers
- **Attachments**: Embedded or referenced files
- **Contacts**: Name, email, phone, address information
- **Calendar Entries**: Appointments, meetings, reminders
- **Notes**: Sticky notes and other items
- **Folders**: Hierarchical folder organization

### OST vs PST

| Feature | PST | OST |
|---------|-----|-----|
| Purpose | Local storage | Exchange cache |
| Access | Standalone | Requires Exchange |
| Location | User-defined | Default location |
| Size Limit | 50GB (Unicode) | Varies |
| Forensic Value | High | High |

### Forensic Value of Outlook Files

Outlook files can contain:

- **Email Communications**: Internal and external correspondence
- **Attachments**: Documents, images, and other files
- **Contact Information**: Names, email addresses, phone numbers
- **Calendar Data**: Meetings, appointments, schedules
- **Deleted Items**: Recoverable deleted messages
- **Metadata**: Timestamps, sender/recipient information
- **Journal Entries**: Activity logs and notes

---

## Command-Line Reference

### Basic Syntax

```bash
readpst [options] <pst_file>
```

### Options

| Option | Description |
|--------|-------------|
| `-h`, `--help` | Show help message |
| `-V`, `--version` | Show version |
| `-o`, `--output-dir <dir>` | Output directory |
| `-M`, `--msg-file` | Output as Outlook .msg files |
| `-r`, `--recursive` | Process subfolders |
| `-q`, `--quiet` | Quiet mode |
| `-D`, `--diagnostic` | Diagnostic mode |
| `-e`, `--export` | Export to Mbox format |
| `-w`, `--write` | Write output files |
| `-b`, `--body` | Include email body |
| `-t`, `-- attachments` | Include attachments |

---

## Basic Usage

### Display PST File Information

```bash
# Show PST file structure
readpst /path/to/archive.pst

# Show detailed information
readpst -D /path/to/archive.pst
```

### Extract Email Messages

```bash
# Create output directory
mkdir -p /evidence/extracted

# Extract all emails in Mbox format
readpst -e -o /evidence/extracted/ /path/to/archive.pst

# Extract as Outlook .msg files
readpst -M -o /evidence/extracted/ /path/to/archive.pst
```

### Extract with Attachments

```bash
# Extract emails with attachments
readpst -r -o /evidence/extracted/ /path/to/archive.pst

# Extract to Mbox format with attachments
readpst -e -r -o /evidence/extracted/ /path/to/archive.pst
```

---

## Advanced Usage

### Command-Line Scenarios

#### Scenario 1: Complete PST Analysis

```bash
#!/bin/bash
# Comprehensive PST analysis with ReadPST

PST_FILE="/evidence/suspect_email.pst"
OUTPUT="/evidence/pst_analysis"
REPORT="$OUTPUT/report.txt"

# Create output directory
mkdir -p "$OUTPUT"

# Generate report header
echo "PST Analysis Report (ReadPST)" > "$REPORT"
echo "=============================" >> "$REPORT"
echo "File: $PST_FILE" >> "$REPORT"
echo "Date: $(date)" >> "$REPORT"
echo "" >> "$REPORT"

# Display PST information
echo "PST Structure:" >> "$REPORT"
readpst "$PST_FILE" >> "$REPORT" 2>&1
echo "" >> "$REPORT"

# Extract emails in Mbox format
echo "Extracting emails..."
readpst -e -r -o "$OUTPUT/mbox/" "$PST_FILE"

# Extract as MSG files
echo "Extracting as MSG files..."
readpst -M -r -o "$OUTPUT/msg/" "$PST_FILE"

echo "Analysis complete: $(date)" >> "$REPORT"
echo "Files extracted: $(find $OUTPUT -type f | wc -l)" >> "$REPORT"
```

#### Scenario 2: Mbox Format for Email Clients

```bash
# Extract to Mbox format for analysis in Thunderbird
readpst -e -o /evidence/mbox/ /evidence/archive.pst

# Each folder becomes a separate Mbox file
ls -la /evidence/mbox/

# Import Mbox files into Thunderbird for review
```

#### Scenario 3: MSG File Analysis

```bash
# Extract as MSG files for individual analysis
readpst -M -r -o /evidence/msg/ /evidence/archive.pst

# Analyze MSG files with other tools
# Use libpff-utils for detailed inspection
pffinfo /evidence/msg/*.msg
```

#### Scenario 4: Forensic Recovery with Documentation

```bash
#!/bin/bash
# Documented forensic recovery

PST_FILE="/evidence/evidence.pst"
CASE_ID="CASE-2026-001"
OUTPUT="/cases/$CASE_ID/evidence/outlook"
LOG="/cases/$CASE_ID/logs/readpst_$(date +%Y%m%d_%H%M%S).log"

# Create directories
mkdir -p "$OUTPUT" "$(dirname $LOG)"

# Document start
echo "ReadPST Recovery Started: $(date)" | tee "$LOG"
echo "PST File: $PST_FILE" | tee -a "$LOG"

# Run ReadPST
readpst -e -r -o "$OUTPUT" "$PST_FILE" 2>&1 | tee -a "$LOG"

# Document completion
echo "Recovery Completed: $(date)" | tee -a "$LOG"
echo "Files extracted: $(find $OUTPUT -type f | wc -l)" | tee -a "$LOG"
```

### Analyzing Results

```bash
# Count extracted emails
find /evidence/extracted/ -type f -name "*.eml" | wc -l

# List Mbox files (one per folder)
ls -la /evidence/mbox/

# Extract email subjects from Mbox files
grep -h "^Subject:" /evidence/mbox/* | sort

# Find largest attachments
find /evidence/extracted/ -type f -exec ls -lh {} \; | sort -k5 -h

# Extract email addresses
grep -h "^From:\|^To:\|^Cc:" /evidence/mbox/* | sort -u
```

---

## Forensics Workflow Integration

### Phase 1: Evidence Acquisition

```bash
# 1. Locate PST/OST files on suspect system
find /mnt/evidence/ -name "*.pst" -o -name "*.ost" 2>/dev/null

# 2. Copy files (preserve metadata)
cp --preserve=all /mnt/evidence/Users/*/AppData/Local/Microsoft/Outlook/*.pst \
    /evidence/outlook_files/

# 3. Calculate hashes
find /evidence/outlook_files/ -name "*.pst" -exec md5sum {} \; > /evidence/pst_hashes.txt
```

### Phase 2: Analysis with ReadPST

```bash
# 1. Create analysis directory structure
mkdir -p /evidence/outlook_analysis/{mbox,msg,reports}

# 2. Analyze each PST file
for pst in /evidence/outlook_files/*.pst; do
    echo "Analyzing: $pst"
    
    # Generate report
    readpst "$pst" > "/evidence/outlook_analysis/reports/$(basename $pst).txt"
    
    # Extract to Mbox format
    readpst -e -r -o "/evidence/outlook_analysis/mbox/$(basename $pst .pst)/" "$pst"
    
    # Extract as MSG files
    readpst -M -r -o "/evidence/outlook_analysis/msg/$(basename $pst .pst)/" "$pst"
done
```

### Phase 3: Post-Analysis

```bash
# 1. Generate comprehensive report
cat > /evidence/outlook_analysis/summary.txt << EOF
ReadPST Analysis Summary
========================
Date: $(date)
PST Files Analyzed: $(find /evidence/outlook_files/ -name "*.pst" | wc -l)
Total Mbox Files: $(find /evidence/outlook_analysis/mbox/ -type f | wc -l)
Total MSG Files: $(find /evidence/outlook_analysis/msg/ -type f | wc -l)
EOF

# 2. Create email timeline
find /evidence/outlook_analysis/mbox/ -type f -exec grep -l "Date:" {} \; | \
    xargs grep "Date:" | sort -k2 > /evidence/outlook_analysis/timeline.txt

# 3. Generate file hashes
find /evidence/outlook_analysis/ -type f -exec md5sum {} \; > /evidence/outlook_analysis/hashes.txt
```

### Integration with Other Tools

```bash
# ReadPST + Vinetto
# Compare results from both tools
vinetto -e -o /evidence/vinetto_output/ /evidence/archive.pst

# ReadPST + Autopsy
# Import Mbox files into Autopsy for timeline analysis

# ReadPST + YARA
# Scan extracted emails and attachments for malicious content
yara -r /path/to/rules/ /evidence/outlook_analysis/

# ReadPST + The Sleuth Kit
# Analyze PST file structure with TSK tools

# ReadPST + Thunderbird
# Import Mbox files into Thunderbird for manual review
```

---

## Limitations and Considerations

### What ReadPST Cannot Do

1. **Decrypt encrypted PSTs**: Cannot bypass password protection
2. **Repair corrupted PSTs**: Limited recovery from damaged files
3. **Recover overwritten data**: Cannot recover data that has been overwritten
4. **Handle very large PSTs**: May struggle with 50GB+ files
5. **Parse third-party formats**: Only handles Microsoft PST/OST

### Known Limitations

- **Large file performance**: May be slow on very large PSTs
- **Memory usage**: High memory consumption for large files
- **Limited deleted recovery**: Cannot recover permanently deleted items
- **No GUI**: Command-line only
- **Complex setup**: Building from source can be challenging

### When to Use Alternatives

| Situation | Better Tool | Reason |
|-----------|-------------|--------|
| Need GUI | Autopsy | Full GUI interface |
| Need deleted recovery | Specialized tools | Better recovery |
| Corrupted PST | scanpst (Windows) | Microsoft repair tool |
| Need batch processing | Scripts | Better automation |

---

## Detection and Defense

### For System Administrators

```bash
# Monitor for PST/OST file access
find / -name "*.pst" -o -name "*.ost" 2>/dev/null

# Check for unauthorized ReadPST usage
sudo ausearch -k readpst -ts recent

# Monitor email client usage
ps aux | grep -i "outlook\|thunderbird"

# Check command history
cat ~/.bash_history | grep -i "readpst\|pst"
```

### For Forensic Investigators

```bash
# Check if ReadPST was used as anti-forensics tool
# Look for PST file manipulation
find / -name "*.pst" -mtime -1 2>/dev/null

# Check command history
cat /root/.bash_history | grep -i "readpst"

# Examine system logs
sudo journalctl | grep -i "readpst\|pst"
```

### Defensive Measures

```bash
# 1. Encrypt PST files
# Use Outlook's built-in encryption

# 2. Enable audit logging
# Monitor PST file access

# 3. Restrict PST file creation
# Group Policy settings

# 4. Regular backup of PST files
# Prevent data loss

# 5. Use Exchange Online
# Reduce local PST storage
```

---

## Troubleshooting

### Common Issues and Solutions

#### "Permission denied"

```bash
# Check file permissions
ls -la /path/to/archive.pst

# Fix permissions
chmod 644 /path/to/archive.pst

# Run with sudo if needed
sudo readpst /path/to/archive.pst
```

#### "Cannot parse PST file"

```bash
# Check if file is valid PST
file /path/to/archive.pst

# Try with diagnostic mode
readpst -D /path/to/archive.pst

# Check for corruption
# May need Microsoft's scanpst tool
```

#### "No emails extracted"

```bash
# Check PST structure
readpst /path/to/archive.pst

# Verify extraction options
readpst -e -v /path/to/archive.pst

# Check output directory permissions
ls -la /evidence/extracted/
```

#### "Memory error"

```bash
# For large PST files
# Monitor memory usage
free -h

# Process in chunks if possible
# Split PST file using other tools

# Use a machine with more RAM
```

---

## Best Practices

### For Forensic Investigations

1. **Always work on copies**: Never analyze original PST files
2. **Preserve metadata**: Use `cp --preserve=all` when copying
3. **Try multiple formats**: Extract as both Mbox and MSG
4. **Document everything**: Record commands and outputs
5. **Verify hashes**: MD5/SHA256 before and after

### For Email Analysis

1. **Start with Mbox format**: Easier to analyze in email clients
2. **Use MSG for individual analysis**: Better for detailed inspection
3. **Extract metadata first**: Get overview before detailed analysis
4. **Focus on key dates**: Identify relevant time periods
5. **Create timelines**: Map out communication patterns

### Configuration Tips

```bash
# 1. Always use -r for recursive extraction
# 2. Start with Mbox format for overview
# 3. Use MSG format for detailed analysis
# 4. Document all commands
# 5. Verify extracted data
```

---

## Comparison with Other Tools

### ReadPST vs Vinetto vs Autopsy

| Feature | ReadPST | Vinetto | Autopsy |
|---------|---------|---------|---------|
| PST Support | Excellent | Good | Excellent |
| OST Support | Yes | No | Yes |
| Output Formats | Mbox/MSG | Text | GUI |
| Speed | Fast | Fast | Medium |
| Ease of Use | Medium | Medium | Easy |
| GUI | No | No | Yes |
| Active Development | High | Low | High |
| Kali Pre-installed | Yes | No | Yes |

### When to Choose ReadPST

- When you need reliable PST/OST parsing
- When you need Mbox output for email clients
- When working with large PST files
- When you need forensic-grade extraction
- As a standard tool for Outlook analysis

---

## References

- **ReadPST/libpff Documentation**: https://github.com/libyal/libpff
- **Kali Linux Package**: https://www.kali.org/tools/libpff-utils/
- **PST File Format**: https://docs.microsoft.com/en-us/openspecs/exchange_server_protocols/ms-pst
- **Digital Forensics Framework**: https://www.digitalforensicsframework.org/
- **Email Forensics Guide**: https://www.sans.org/reading-room/email-forensics-1423
- **NIST SP 800-86**: Guide to Integrating Forensic Techniques into Incident Response

---

## Quick Reference Card

| Operation | Command |
|-----------|---------|
| Show PST info | `readpst archive.pst` |
| Extract Mbox | `readpst -e -o output/ archive.pst` |
| Extract MSG | `readpst -M -o output/ archive.pst` |
| Recursive | `readpst -r -e -o output/ archive.pst` |
| Diagnostic | `readpst -D archive.pst` |
| Quiet mode | `readpst -q -e -o output/ archive.pst` |
| View structure | `readpst archive.pst` |
| Count emails | `find output/ -name "*.eml" \| wc -l` |

---

*Document Version: 1.0*
*Last Updated: 2026*
*Author: Digital Forensics Documentation Project*
*Classification: Educational*
