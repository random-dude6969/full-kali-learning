# Pdfid — PDF Identification Tool

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

Pdfid is a lightweight command-line tool designed to quickly identify and list the key elements within PDF files. Developed by Didier Stevens (the same author as PDF-Parser), pdfid provides a fast overview of PDF features, including JavaScript, embedded files, actions, and other potentially dangerous components. It is an essential first-pass tool for triaging PDF files in forensic investigations and malware analysis.

Unlike PDF-Parser, which provides detailed structure analysis, pdfid focuses on rapid identification. It scans PDF files and reports the presence of specific keywords and features, making it ideal for quickly assessing whether a PDF file contains suspicious elements.

### Key Features

- **Quick Identification**: Fast scanning of PDF files
- **Keyword Detection**: Identifies specific PDF features
- **Batch Processing**: Can scan multiple files
- **Plugin System**: Extensible with custom plugins
- **Non-Destructive**: Read-only analysis
- **Cross-Platform**: Works on Linux, macOS, and Windows

### When to Use Pdfid

| Scenario | Use Pdfid? | Notes |
|----------|-----------|-------|
| Quick PDF triage | Yes | Primary use case |
| Malware screening | Yes | Fast identification |
| Batch analysis | Yes | Process multiple files |
| Initial assessment | Yes | First pass before PDF-Parser |
| Detailed analysis | No | Use PDF-Parser instead |
| Stream extraction | No | Use PDF-Parser instead |

---

## Installation

### Kali Linux

```bash
# Pdfid may need to be installed manually
sudo apt update
sudo apt install pdfid

# Or install from pip
pip3 install pdfid

# Verify installation
pdfid.py --version
```

### Other Methods

```bash
# From Didier Stevens' website
wget https://didierstevens.com/files/software/pdfid_V0_2_5.zip
unzip pdfid_V0_2_5.zip

# From GitHub
git clone https://github.com/DidierStevens/DidierStevensSuite.git
cd DidierStevensSuite

# Verify
python3 pdfid.py --version
```

---

## Core Concepts

### PDF Keywords

Pdfid searches for specific keywords in PDF files:

| Keyword | Risk Level | Description |
|---------|------------|-------------|
| JavaScript | High | Embedded JavaScript code |
| OpenAction | High | Auto-run action on open |
| AA | High | Additional actions |
| AcroForm | Medium | Interactive forms |
| RichMedia | High | Flash content |
| EmbeddedFile | Medium | Attached files |
| Launch | High | Launch external programs |
| URI | Low | External links |
| SubmitForm | Medium | Form submission |
| ImportData | Medium | Data import |

### Risk Assessment

Pdfid helps assess PDF risk:

- **High Risk**: JavaScript, OpenAction, Launch, RichMedia
- **Medium Risk**: AcroForm, EmbeddedFile, SubmitForm
- **Low Risk**: URI, ImportData, standard features

---

## Command-Line Reference

### Basic Syntax

```bash
pdfid.py [options] <pdf_file>
```

### Options

| Option | Description |
|--------|-------------|
| `-h`, `--help` | Show help message |
| `-v`, `--version` | Show version |
| `-f`, `--force` | Force analysis of non-PDF files |
| `-s`, `--scan` | Scan directory recursively |
| `-p`, `--plugin` | Load custom plugin |
| `-l`, `--list` | List available plugins |
| `-n`, `--nozero` | Don't show zero counts |
| `-q`, `--quiet` | Quiet mode |

---

## Basic Usage

### Scan Single PDF

```bash
# Quick identification
python3 pdfid.py /path/to/document.pdf

# Show all features
python3 pdfid.py -n /path/to/document.pdf
```

### Scan Directory

```bash
# Scan all PDFs in directory
python3 pdfid.py -s /path/to/pdfs/

# Recursive scan
python3 pdfid.py -s -r /path/to/pdfs/
```

### Filter Results

```bash
# Show only high-risk features
python3 pdfid.py /path/to/document.pdf | grep -E "JavaScript|OpenAction|Launch"

# Count suspicious features
python3 pdfid.py /path/to/document.pdf | awk '{sum += $2} END {print sum}'
```

---

## Advanced Usage

### Command-Line Scenarios

#### Scenario 1: Batch PDF Triage

```bash
#!/bin/bash
# Batch triage PDF files for malware indicators

PDF_DIR="/evidence/pdfs"
REPORT="/evidence/pdf_triage.txt"

echo "PDF Triage Report" > "$REPORT"
echo "Date: $(date)" >> "$REPORT"
echo "" >> "$REPORT"

for pdf in "$PDF_DIR"/*.pdf; do
    echo "Scanning: $pdf"
    echo "File: $(basename $pdf)" >> "$REPORT"
    
    # Run pdfid
    RESULT=$(python3 pdfid.py "$pdf" 2>/dev/null)
    
    # Check for high-risk features
    if echo "$RESULT" | grep -q "JavaScript.*[1-9]"; then
        echo "  [!] JavaScript detected" >> "$REPORT"
    fi
    if echo "$RESULT" | grep -q "OpenAction.*[1-9]"; then
        echo "  [!] OpenAction detected" >> "$REPORT"
    fi
    if echo "$RESULT" | grep -q "EmbeddedFile.*[1-9]"; then
        echo "  [!] EmbeddedFile detected" >> "$REPORT"
    fi
    
    echo "$RESULT" >> "$REPORT"
    echo "" >> "$REPORT"
done

echo "Triage complete: $(date)" >> "$REPORT"
```

#### Scenario 2: Quick Malware Check

```bash
# Quick check for suspicious features
python3 pdfid.py suspicious.pdf | grep -E "JavaScript|OpenAction|Launch|RichMedia"

# If any high-risk features found, analyze further
if python3 pdfid.py suspicious.pdf | grep -qE "JavaScript|OpenAction"; then
    echo "Suspicious features detected - running PDF-Parser"
    python3 pdf-parser.py -j -f suspicious.pdf
fi
```

#### Scenario 3: Plugin Usage

```bash
# List available plugins
python3 pdfid.py -l

# Use specific plugin
python3 pdfid.py -p plugin_name document.pdf

# Create custom plugin
cat > my_plugin.py << 'EOF'
# Custom plugin for pdfid
keywords = ['MyKeyword', 'AnotherKeyword']
EOF
```

#### Scenario 4: Forensic Triage

```bash
#!/bin/bash
# Forensic PDF triage workflow

EVIDENCE="/evidence"
OUTPUT="$EVIDENCE/pdf_triage"
LOG="$OUTPUT/triage.log"

mkdir -p "$OUTPUT"

echo "PDF Forensic Triage" | tee "$LOG"
echo "Date: $(date)" | tee -a "$LOG"

# Find all PDFs
find "$EVIDENCE" -name "*.pdf" -type f | while read pdf; do
    echo "Analyzing: $pdf" | tee -a "$LOG"
    
    # Run pdfid
    python3 pdfid.py "$pdf" > "$OUTPUT/$(basename $pdf).txt" 2>&1
    
    # Check for suspicious features
    if grep -qE "JavaScript.*[1-9]|OpenAction.*[1-9]" "$OUTPUT/$(basename $pdf).txt"; then
        echo "  [SUSPICIOUS]" | tee -a "$LOG"
        cp "$pdf" "$OUTPUT/suspicious/"
    fi
done

echo "Triage complete: $(date)" | tee -a "$LOG"
```

### Analyzing Results

```bash
# Count suspicious PDFs
grep -l "\[SUSPICIOUS\]" /evidence/pdf_triage/*.txt | wc -l

# List PDFs with JavaScript
grep -l "JavaScript.*[1-9]" /evidence/pdf_triage/*.txt

# List PDFs with embedded files
grep -l "EmbeddedFile.*[1-9]" /evidence/pdf_triage/*.txt

# Sort by risk level
cat /evidence/pdf_triage/*.txt | grep -E "JavaScript|OpenAction|Launch" | sort
```

---

## Forensics Workflow Integration

### Phase 1: Evidence Acquisition

```bash
# 1. Locate PDF files on suspect system
find /mnt/evidence/ -name "*.pdf" -type f

# 2. Copy PDF files (preserve metadata)
cp --preserve=all /mnt/evidence/Users/*/Documents/*.pdf \
    /evidence/pdf_files/

# 3. Calculate hashes
find /evidence/pdf_files/ -name "*.pdf" -exec md5sum {} \; > /evidence/pdf_hashes.txt
```

### Phase 2: Triage with Pdfid

```bash
# 1. Create triage directory
mkdir -p /evidence/pdf_triage/{results,suspicious}

# 2. Triage each PDF
for pdf in /evidence/pdf_files/*.pdf; do
    echo "Triage: $pdf"
    
    # Run pdfid
    python3 pdfid.py "$pdf" > "/evidence/pdf_triage/results/$(basename $pdf).txt"
    
    # Check for suspicious features
    if grep -qE "JavaScript.*[1-9]|OpenAction.*[1-9]" "/evidence/pdf_triage/results/$(basename $pdf).txt"; then
        echo "  [SUSPICIOUS] Moving to suspicious directory"
        cp "$pdf" /evidence/pdf_triage/suspicious/
    fi
done
```

### Phase 3: Post-Triage

```bash
# 1. Generate triage report
cat > /evidence/pdf_triage/summary.txt << EOF
PDF Triage Summary
==================
Date: $(date)
Total PDFs: $(find /evidence/pdf_files/ -name "*.pdf" | wc -l)
Suspicious PDFs: $(find /evidence/pdf_triage/suspicious/ -name "*.pdf" | wc -l)
With JavaScript: $(grep -l "JavaScript.*[1-9]" /evidence/pdf_triage/results/*.txt | wc -l)
With Embedded Files: $(grep -l "EmbeddedFile.*[1-9]" /evidence/pdf_triage/results/*.txt | wc -l)
EOF

# 2. List suspicious PDFs for detailed analysis
echo "PDFs requiring detailed analysis:" > /evidence/pdf_triage/further_analysis.txt
find /evidence/pdf_triage/suspicious/ -name "*.pdf" >> /evidence/pdf_triage/further_analysis.txt
```

### Integration with Other Tools

```bash
# Pdfid + PDF-Parser
# Use pdfid for triage, PDF-Parser for detailed analysis
if python3 pdfid.py suspicious.pdf | grep -q "JavaScript"; then
    python3 pdf-parser.py -j suspicious.pdf
fi

# Pdfid + YARA
# Scan PDFs with YARA rules
yara -r /path/to/pdf_rules/ /evidence/pdf_files/

# Pdfid + ClamAV
# Scan PDFs with antivirus
clamscan -r /evidence/pdf_files/

# Pdfid + OleTools
# Additional Office document analysis
oleid /evidence/suspicious.pdf
```

---

## Limitations and Considerations

### What Pdfid Cannot Do

1. **Detailed analysis**: Only identifies features, doesn't analyze content
2. **Extract streams**: Use PDF-Parser for extraction
3. **Decode obfuscated code**: Limited obfuscation handling
4. **Execute content**: Read-only analysis
5. **Repair damaged PDFs**: Cannot fix corrupted files

### Known Limitations

- **False positives**: May flag benign features
- **Limited context**: Doesn't explain what features do
- **No extraction**: Cannot extract embedded content
- **No decoding**: Cannot decode obfuscated JavaScript
- **No GUI**: Command-line only

### When to Use Alternatives

| Situation | Better Tool | Reason |
|-----------|-------------|--------|
| Detailed analysis | PDF-Parser | Full structure analysis |
| Stream extraction | PDF-Parser | Can decode streams |
| GUI needed | PDFStream Dumper | GUI interface |
| Batch processing | Scripts | Better automation |

---

## Detection and Defense

### For System Administrators

```bash
# Monitor for pdfid usage
which pdfid.py

# Check for unauthorized PDF analysis
sudo ausearch -k pdfid -ts recent

# Monitor PDF file creation
find / -name "*.pdf" -mtime -1 2>/dev/null

# Check command history
cat ~/.bash_history | grep -i "pdfid\|pdf"
```

### For Forensic Investigators

```bash
# Check if pdfid was used as anti-forensics tool
# Look for PDF file manipulation
find / -name "*.pdf" -mtime -1 2>/dev/null

# Check command history
cat /root/.bash_history | grep -i "pdfid"

# Examine system logs
sudo journalctl | grep -i "pdfid\|pdf"
```

### Defensive Measures

```bash
# 1. Block suspicious PDFs at email gateway
# Configure mail server to block PDFs with JavaScript

# 2. Use PDF sanitization tools
# Strip JavaScript from incoming PDFs

# 3. Enable PDF viewers' security settings
# Disable JavaScript in Adobe Reader

# 4. Use application whitelisting
# Only allow approved PDF viewers

# 5. Regular security updates
# Keep PDF viewers patched
```

---

## Troubleshooting

### Common Issues and Solutions

#### "Permission denied"

```bash
# Check file permissions
ls -la /path/to/document.pdf

# Fix permissions
chmod 644 /path/to/document.pdf
```

#### "Not a PDF file"

```bash
# Check if file is valid PDF
file /path/to/document.pdf

# Force analysis
python3 pdfid.py -f /path/to/document.pdf
```

#### "No output"

```bash
# Check if file exists
ls -la /path/to/document.pdf

# Try with verbose mode
python3 pdfid.py -v /path/to/document.pdf

# Check file size
ls -lh /path/to/document.pdf
```

---

## Best Practices

### For Forensic Investigations

1. **Use for triage first**: Quick identification before detailed analysis
2. **Document everything**: Record commands and outputs
3. **Combine with PDF-Parser**: Use pdfid for triage, PDF-Parser for details
4. **Verify hashes**: MD5/SHA256 before and after
5. **Batch process**: Scan multiple files efficiently

### For Malware Analysis

1. **Check JavaScript first**: High-risk feature
2. **Look for OpenAction**: Auto-run on open
3. **Check for embedded files**: May contain payloads
4. **Use plugins**: Extend functionality
5. **Combine with other tools**: YARA, ClamAV

### Configuration Tips

```bash
# 1. Start with default keywords
# 2. Use plugins for custom detection
# 3. Batch process for efficiency
# 4. Document all commands
# 5. Verify results with PDF-Parser
```

---

## Comparison with Other Tools

### Pdfid vs PDF-Parser vs pdfid

| Feature | Pdfid | PDF-Parser | pdfid |
|---------|-------|------------|-------|
| Speed | Fast | Medium | Fast |
| Detail | Low | High | Low |
| Extraction | No | Yes | No |
| Batch | Yes | Yes | Yes |
| Plugins | No | No | Yes |
| GUI | No | No | No |

### When to Choose Pdfid

- When you need quick triage
- When processing many PDFs
- When you need initial assessment
- As a first-pass tool before PDF-Parser

---

## References

- **Pdfid Documentation**: https://blog.didierstevens.com/2009/12/30/pdfid/
- **Didier Stevens' Tools**: https://github.com/DidierStevens/DidierStevensSuite
- **Kali Linux Forensics**: https://www.kali.org/tools/pdfid/
- **PDF Security**: https://www.adobe.com/devnet/security.html
- **Malicious PDF Analysis**: https://www.sans.org/reading-room/malicious-pdf-analysis-1615

---

## Quick Reference Card

| Operation | Command |
|-----------|---------|
| Quick scan | `python3 pdfid.py document.pdf` |
| Scan directory | `python3 pdfid.py -s /path/to/pdfs/` |
| List plugins | `python3 pdfid.py -l` |
| Use plugin | `python3 pdfid.py -p plugin document.pdf` |
| Filter high-risk | `python3 pdfid.py document.pdf \| grep -E "JavaScript\|OpenAction"` |
| Force analysis | `python3 pdfid.py -f document.pdf` |
| Quiet mode | `python3 pdfid.py -q document.pdf` |

---

*Document Version: 1.0*
*Last Updated: 2026*
*Author: Digital Forensics Documentation Project*
*Classification: Educational*
