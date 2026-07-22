# Vinetto — PST File Parser for Forensics

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

Vinetto is a command-line tool designed to parse and analyze Microsoft Outlook PST (Personal Storage Table) files. PST files are used by Microsoft Outlook to store emails, contacts, calendar entries, and other data locally on a user's computer. In digital forensics, PST files are valuable sources of evidence as they contain communications, schedules, and other personal information.

Vinetto provides a forensic-friendly interface for examining PST files without modifying them. It can extract email headers, body content, attachments, and metadata from PST files, making it useful for incident response, e-discovery, and criminal investigations.

### Key Features

- **PST Parsing**: Reads and parses Microsoft Outlook PST files
- **Email Extraction**: Extracts email headers, body, and attachments
- **Metadata Analysis**: Displays file and message metadata
- **Non-Destructive**: Read-only analysis, never modifies source files
- **Cross-Platform**: Works on Linux and other Unix-like systems
- **Command-Line Interface**: Scriptable and automatable

### When to Use Vinetto

| Scenario | Use Vinetto? | Notes |
|----------|-------------|-------|
| Analyzing PST email archives | Yes | Primary use case |
| Extracting email metadata | Yes | Headers, timestamps, recipients |
| E-discovery investigations | Yes | Good for email review |
| Incident response | Yes | Analyzing suspect email |
| Corrupted PST files | Limited | May not parse corrupted files |
| Encrypted PST files | No | Cannot decrypt password-protected PSTs |

---

## Installation

### Kali Linux

```bash
# Vinetto may need to be installed manually
sudo apt update
sudo apt install vinetto

# If not in repositories, build from source
```

### Other Linux Distributions

```bash
# Debian/Ubuntu
sudo apt install vinetto

# From source
git clone https://github.com/misterx/vinetto.git
cd vinetto
python setup.py install
```

### Building from Source

```bash
# Install Python dependencies
sudo apt install python3 python3-pip
pip3 install vinetto

# Or from GitHub
git clone https://github.com/misterx/vinetto.git
cd vinetto
pip3 install -r requirements.txt
python3 setup.py install

# Verify
vinetto --version
```

---

## Core Concepts

### PST File Structure

PST files contain:

- **Email Messages**: Subject, body, headers, attachments
- **Contacts**: Name, email, phone, address
- **Calendar Entries**: Appointments, meetings, reminders
- **Folders**: Hierarchical folder structure
- **Metadata**: Creation date, modification date, size

### Forensic Value of PST Files

PST files can contain:

- **Email Communications**: Internal and external correspondence
- **Attachments**: Documents, images, and other files
- **Contact Information**: Names, email addresses, phone numbers
- **Calendar Data**: Meetings, appointments, schedules
- **Deleted Items**: Recoverable deleted messages
- **Metadata**: Timestamps, sender/recipient information

### PST Versions

- **PST (97-2002)**: ANSI format, 2GB limit
- **PST (2003+)**: Unicode format, 50GB limit
- **OST**: Offline Storage Table (Exchange cache)

---

## Command-Line Reference

### Basic Syntax

```bash
vinetto [options] <pst_file>
```

### Options

| Option | Description |
|--------|-------------|
| `-h`, `--help` | Show help message |
| `-v`, `--version` | Show version |
| `-o`, `--output <dir>` | Output directory for extracted data |
| `-e`, `--emails` | Extract email messages |
| `-a`, `--attachments` | Extract attachments |
| `-m`, `--metadata` | Display metadata only |
| `-f`, `--folders` | List folder structure |
| `-t`, `--timestamps` | Display timestamps |
| `-q`, `--quiet` | Quiet mode |
| `-d`, `--debug` | Debug mode |

---

## Basic Usage

### Display PST File Information

```bash
# Show PST file metadata
vinetto /path/to/archive.pst

# Show folder structure
vinetto -f /path/to/archive.pst

# Show timestamps
vinetto -t /path/to/archive.pst
```

### Extract Email Messages

```bash
# Create output directory
mkdir -p /evidence/extracted

# Extract all emails
vinetto -e -o /evidence/extracted/ /path/to/archive.pst

# Extract emails with attachments
vinetto -e -a -o /evidence/extracted/ /path/to/archive.pst
```

### Analyze Specific Folders

```bash
# List all folders
vinetto -f /path/to/archive.pst

# Extract from specific folder
vinetto -e -o /evidence/inbox/ /path/to/archive.pst --folder "Inbox"
```

---

## Advanced Usage

### Command-Line Scenarios

#### Scenario 1: Complete PST Analysis

```bash
#!/bin/bash
# Comprehensive PST analysis

PST_FILE="/evidence/suspect_email.pst"
OUTPUT="/evidence/pst_analysis"
REPORT="$OUTPUT/report.txt"

# Create output directory
mkdir -p "$OUTPUT"

# Generate report header
echo "PST Analysis Report" > "$REPORT"
echo "==================" >> "$REPORT"
echo "File: $PST_FILE" >> "$REPORT"
echo "Date: $(date)" >> "$REPORT"
echo "" >> "$REPORT"

# Display metadata
echo "Metadata:" >> "$REPORT"
vinetto "$PST_FILE" >> "$REPORT" 2>&1
echo "" >> "$REPORT"

# List folders
echo "Folder Structure:" >> "$REPORT"
vinetto -f "$PST_FILE" >> "$REPORT" 2>&1
echo "" >> "$REPORT"

# Extract emails
echo "Extracting emails..."
vinetto -e -o "$OUTPUT/emails/" "$PST_FILE"

# Extract attachments
echo "Extracting attachments..."
vinetto -a -o "$OUTPUT/attachments/" "$PST_FILE"

echo "Analysis complete: $(date)" >> "$REPORT"
```

#### Scenario 2: Email Header Analysis

```bash
# Extract and analyze email headers
vinetto -e -o /evidence/emails/ /evidence/archive.pst

# Parse headers for IP addresses
grep -h "Received:" /evidence/emails/*.eml | \
    grep -oE '[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+' | sort -u

# Find emails from specific sender
grep -l "From:.*suspect@evil.com" /evidence/emails/*.eml
```

#### Scenario 3: Attachment Analysis

```bash
# Extract all attachments
vinetto -a -o /evidence/attachments/ /evidence/archive.pst

# List all extracted attachments
find /evidence/attachments/ -type f > /evidence/attachment_list.txt

# Check attachment types
find /evidence/attachments/ -type f -exec file {} \; > /evidence/attachment_types.txt

# Scan attachments for malware
yara -r /path/to/malware_rules/ /evidence/attachments/
```

#### Scenario 4: Timeline Analysis

```bash
# Extract timestamps from PST
vinetto -t /evidence/archive.pst > /evidence/timestamps.txt

# Create timeline
cat /evidence/timestamps.txt | sort -k2 > /evidence/timeline.txt

# Analyze email patterns
grep -oP '\d{4}-\d{2}-\d{2}' /evidence/timeline.txt | sort | uniq -c
```

### Analyzing Results

```bash
# Count extracted emails
find /evidence/emails/ -type f | wc -l

# List email subjects
grep -h "^Subject:" /evidence/emails/*.eml | sort

# Find largest attachments
find /evidence/attachments/ -type f -exec ls -lh {} \; | sort -k5 -h

# Extract email addresses
grep -h "^From:\|^To:\|^Cc:" /evidence/emails/*.eml | sort -u
```

---

## Forensics Workflow Integration

### Phase 1: Evidence Acquisition

```bash
# 1. Locate PST files on suspect system
find /mnt/evidence/ -name "*.pst" -type f

# 2. Copy PST files (preserve metadata)
cp --preserve=all /mnt/evidence/Users/*/AppData/Local/Microsoft/Outlook/*.pst \
    /evidence/pst_files/

# 3. Calculate hashes
find /evidence/pst_files/ -name "*.pst" -exec md5sum {} \; > /evidence/pst_hashes.txt
```

### Phase 2: Analysis with Vinetto

```bash
# 1. Create analysis directory
mkdir -p /evidence/pst_analysis/{emails,attachments,reports}

# 2. Analyze each PST file
for pst in /evidence/pst_files/*.pst; do
    echo "Analyzing: $pst"
    
    # Generate report
    vinetto "$pst" > "/evidence/pst_analysis/reports/$(basename $pst).txt"
    
    # Extract emails
    vinetto -e -o "/evidence/pst_analysis/emails/$(basename $pst .pst)/" "$pst"
    
    # Extract attachments
    vinetto -a -o "/evidence/pst_analysis/attachments/$(basename $pst .pst)/" "$pst"
done
```

### Phase 3: Post-Analysis

```bash
# 1. Generate comprehensive report
cat > /evidence/pst_analysis/summary.txt << EOF
PST Analysis Summary
====================
Date: $(date)
PST Files Analyzed: $(find /evidence/pst_files/ -name "*.pst" | wc -l)
Total Emails Extracted: $(find /evidence/pst_analysis/emails/ -type f | wc -l)
Total Attachments: $(find /evidence/pst_analysis/attachments/ -type f | wc -l)
EOF

# 2. Create timeline
find /evidence/pst_analysis/emails/ -name "*.eml" -exec grep -l "Date:" {} \; | \
    xargs grep "Date:" | sort -k2 > /evidence/pst_analysis/timeline.txt

# 3. Generate file hashes
find /evidence/pst_analysis/ -type f -exec md5sum {} \; > /evidence/pst_analysis/hashes.txt
```

### Integration with Other Tools

```bash
# Vinetto + readpst
# Compare results from both tools
readpst -o /evidence/readpst_output/ /evidence/archive.pst
vinetto -e -o /evidence/vinetto_output/ /evidence/archive.pst

# Vinetto + Autopsy
# Import extracted emails into Autopsy for timeline analysis

# Vinetto + YARA
# Scan extracted emails and attachments for malicious content
yara -r /path/to/rules/ /evidence/pst_analysis/

# Vinetto + The Sleuth Kit
# Analyze PST file structure with TSK tools
```

---

## Limitations and Considerations

### What Vinetto Cannot Do

1. **Decrypt encrypted PSTs**: Cannot bypass password protection
2. **Repair corrupted PSTs**: Cannot fix damaged files
3. **Parse OST files**: Only handles PST format
4. **Recover deleted items**: Limited deleted item recovery
5. **Handle very large PSTs**: May struggle with 50GB+ files

### Known Limitations

- **Limited documentation**: Fewer resources than mainstream tools
- **No GUI**: Command-line only
- **No resume capability**: Must restart if interrupted
- **Limited error handling**: May crash on corrupted files
- **Python dependency**: Requires Python environment

### When to Use Alternatives

| Situation | Better Tool | Reason |
|-----------|-------------|--------|
| Encrypted PSTs | readpst | Better encryption support |
| Corrupted PSTs | scanpst (Windows) | Microsoft repair tool |
| OST files | readpst | OST support |
| GUI needed | Autopsy | Full GUI interface |
| Batch processing | readpst | Better scripting |

---

## Detection and Defense

### For System Administrators

```bash
# Monitor for PST file access
find / -name "*.pst" -type f 2>/dev/null

# Check for unauthorized PST extraction
sudo ausearch -k vinetto -ts recent

# Monitor email client usage
ps aux | grep -i "outlook\|thunderbird"

# Check command history
cat ~/.bash_history | grep -i "vinetto\|pst"
```

### For Forensic Investigators

```bash
# Check if Vinetto was used as anti-forensics tool
# Look for PST file manipulation
find / -name "*.pst" -mtime -1 2>/dev/null

# Check command history
cat /root/.bash_history | grep -i "vinetto"

# Examine system logs
sudo journalctl | grep -i "vinetto\|pst"
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
sudo vinetto /path/to/archive.pst
```

#### "Cannot parse PST file"

```bash
# Check if file is valid PST
file /path/to/archive.pst

# Try different PST version
vinetto --version97 /path/to/archive.pst

# Check for corruption
# Use Microsoft's scanpst tool if available
```

#### "No emails extracted"

```bash
# Check folder structure
vinetto -f /path/to/archive.pst

# Verify extraction options
vinetto -e -v /path/to/archive.pst

# Check output directory permissions
ls -la /evidence/extracted/
```

#### "Memory error"

```bash
# For large PST files, process in chunks
# Split PST file if possible

# Use a machine with more RAM
# Monitor memory usage
free -h
```

---

## Best Practices

### For Forensic Investigations

1. **Always work on copies**: Never analyze original PST files
2. **Preserve metadata**: Use `cp --preserve=all` when copying
3. **Document everything**: Record commands and outputs
4. **Verify hashes**: MD5/SHA256 before and after
5. **Combine with other tools**: Use Vinetto alongside readpst and Autopsy

### For Email Analysis

1. **Extract metadata first**: Get overview before detailed analysis
2. **Focus on key dates**: Identify relevant time periods
3. **Analyze attachments**: Check for malware or evidence
4. **Review deleted items**: May contain relevant evidence
5. **Create timelines**: Map out communication patterns

### Configuration Tips

```bash
# 1. Start with metadata overview
# 2. Extract emails before attachments
# 3. Use verbose mode for debugging
# 4. Document all commands
# 5. Verify extracted data
```

---

## Comparison with Other Tools

### Vinetto vs readpst vs Autopsy

| Feature | Vinetto | readpst | Autopsy |
|---------|---------|---------|---------|
| Speed | Fast | Fast | Medium |
| Ease of use | Medium | Easy | Easy |
| Output format | Text | Mbox/Text | GUI |
| Batch processing | Yes | Yes | Yes |
| GUI | No | No | Yes |
| Active development | Low | Medium | High |

### When to Choose Vinetto

- When you need quick PST analysis
- When working with Python environments
- When you need scriptable output
- As a first-pass analysis tool

---

## References

- **Vinetto Documentation**: https://github.com/misterx/vinetto
- **Kali Linux Forensics**: https://www.kali.org/tools/vinetto/
- **PST File Format**: https://docs.microsoft.com/en-us/openspecs/exchange_server_protocols/ms-pst
- **Digital Forensics Framework**: https://www.digitalforensicsframework.org/
- **Email Forensics Guide**: https://www.sans.org/reading-room/email-forensics-1423

---

## Quick Reference Card

| Operation | Command |
|-----------|---------|
| Show metadata | `vinetto archive.pst` |
| List folders | `vinetto -f archive.pst` |
| Extract emails | `vinetto -e -o output/ archive.pst` |
| Extract attachments | `vinetto -a -o output/ archive.pst` |
| Show timestamps | `vinetto -t archive.pst` |
| Full analysis | `vinetto -e -a -o output/ archive.pst` |
| Verbose mode | `vinetto -v archive.pst` |

---

*Document Version: 1.0*
*Last Updated: 2026*
*Author: Digital Forensics Documentation Project*
*Classification: Educational*
