# PDF-Parser — PDF Structure Analysis Tool

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

PDF-Parser is a Python-based command-line tool designed to analyze and parse the internal structure of PDF (Portable Document Format) files. Developed by Didier Stevens, PDF-Parser is an essential tool in the digital forensics and malware analysis toolkit. It allows analysts to examine PDF objects, streams, and metadata without executing the PDF, making it safe for analyzing potentially malicious documents.

PDF files are complex containers that can embed various types of content including JavaScript, Flash, embedded files, and other ActiveX objects. Attackers frequently use PDF files as vectors for malware delivery. PDF-Parser helps analysts understand the internal structure of suspicious PDFs, identify malicious components, and extract embedded payloads.

### Key Features

- **PDF Structure Analysis**: Parses and displays PDF object hierarchy
- **Stream Extraction**: Extracts and decodes embedded streams
- **JavaScript Detection**: Identifies embedded JavaScript code
- **Object Enumeration**: Lists all PDF objects and their types
- **Metadata Extraction**: Displays PDF metadata and properties
- **Non-Destructive**: Read-only analysis, never executes PDF content

### When to Use PDF-Parser

| Scenario | Use PDF-Parser? | Notes |
|----------|----------------|-------|
| Analyzing suspicious PDFs | Yes | Primary use case |
| Malware analysis | Yes | Identifies malicious components |
| Extracting embedded files | Yes | Can extract payloads |
| Checking for JavaScript | Yes | Common attack vector |
| PDF structure validation | Yes | Verify PDF integrity |
| Recovering damaged PDFs | Partial | May recover some data |
| Editing PDFs | No | Read-only tool |

---

## Installation

### Kali Linux

```bash
# PDF-Parser may need to be installed manually
sudo apt update
sudo apt install pdf-parser

# Or install from pip
pip3 install pdf-parser

# Verify installation
pdf-parser.py --version
```

### Other Methods

```bash
# From Didier Stevens' website
wget https://didierstevens.com/files/software/pdf-parser_V0_7_5.zip
unzip pdf-parser_V0_7_5.zip

# From GitHub
git clone https://github.com/DidierStevens/DidierStevensSuite.git
cd DidierStevensSuite

# Verify
python3 pdf-parser.py --version
```

### Dependencies

```bash
# PDF-Parser requires Python 3
sudo apt install python3 python3-pip

# Optional dependencies
pip3 install oletools  # For additional Office document analysis
```

---

## Core Concepts

### PDF File Structure

PDF files consist of:

- **Header**: Version information (e.g., %PDF-1.7)
- **Body**: Objects containing content
- **Cross-Reference Table**: Index of objects
- **Trailer**: Root object reference and metadata

### PDF Objects

PDF objects include:

- **Pages**: Document pages
- **Streams**: Compressed or encoded data
- **JavaScript**: Embedded code (potential malware)
- **Embedded Files**: Attached documents
- **Actions**: Automated behaviors
- **Annotations**: Comments and markup

### Malicious PDF Components

Common attack vectors in PDFs:

- **JavaScript**: Can execute code
- **Embedded Files**: May contain malware
- **Launch Actions**: Auto-execute programs
- **AcroForm**: Interactive forms with scripts
- **RichMedia**: Flash content
- **OpenAction**: Auto-run on open

---

## Command-Line Reference

### Basic Syntax

```bash
pdf-parser.py [options] <pdf_file>
```

### Options

| Option | Description |
|--------|-------------|
| `-h`, `--help` | Show help message |
| `-v`, `--version` | Show version |
| `-a`, `--all` | Show all objects |
| `-o`, `--object <id>` | Show specific object |
| `-s`, `--search <type>` | Search for object type |
| `-e`, `--extract` | Extract streams |
| `-d`, `--decode` | Decode streams |
| `-j`, `--javascript` | Find JavaScript |
| `-f`, `--files` | Find embedded files |
| `-m`, `--metadata` | Show metadata |
| `-r`, `--raw` | Show raw content |
| `-c`, `--count` | Count objects |

---

## Basic Usage

### Display PDF Structure

```bash
# Show PDF object overview
python3 pdf-parser.py /path/to/document.pdf

# Show all objects
python3 pdf-parser.py -a /path/to/document.pdf

# Show metadata
python3 pdf-parser.py -m /path/to/document.pdf
```

### Find JavaScript

```bash
# Search for JavaScript objects
python3 pdf-parser.py -j /path/to/document.pdf

# Search for specific object type
python3 pdf-parser.py -s JavaScript /path/to/document.pdf
```

### Extract Embedded Files

```bash
# Find embedded files
python3 pdf-parser.py -f /path/to/document.pdf

# Extract all streams
python3 pdf-parser.py -e -d /path/to/document.pdf
```

---

## Advanced Usage

### Command-Line Scenarios

#### Scenario 1: Malicious PDF Analysis

```bash
#!/bin/bash
# Comprehensive malicious PDF analysis

PDF_FILE="/evidence/suspicious.pdf"
OUTPUT="/evidence/pdf_analysis"
REPORT="$OUTPUT/report.txt"

# Create output directory
mkdir -p "$OUTPUT"

# Generate report header
echo "PDF Malware Analysis Report" > "$REPORT"
echo "===========================" >> "$REPORT"
echo "File: $PDF_FILE" >> "$REPORT"
echo "Date: $(date)" >> "$REPORT"
echo "MD5: $(md5sum $PDF_FILE | awk '{print $1}')" >> "$REPORT"
echo "SHA256: $(sha256sum $PDF_FILE | awk '{print $1}')" >> "$REPORT"
echo "" >> "$REPORT"

# Show metadata
echo "Metadata:" >> "$REPORT"
python3 pdf-parser.py -m "$PDF_FILE" >> "$REPORT" 2>&1
echo "" >> "$REPORT"

# Find JavaScript
echo "JavaScript Analysis:" >> "$REPORT"
python3 pdf-parser.py -j "$PDF_FILE" >> "$REPORT" 2>&1
echo "" >> "$REPORT"

# Find embedded files
echo "Embedded Files:" >> "$REPORT"
python3 pdf-parser.py -f "$PDF_FILE" >> "$REPORT" 2>&1
echo "" >> "$REPORT"

# Extract and decode streams
echo "Extracting streams..."
python3 pdf-parser.py -e -d -o "$OUTPUT/streams/" "$PDF_FILE"

echo "Analysis complete: $(date)" >> "$REPORT"
```

#### Scenario 2: JavaScript Extraction

```bash
# Find all JavaScript objects
python3 pdf-parser.py -j /evidence/document.pdf

# Extract JavaScript content
python3 pdf-parser.py -s JavaScript -e /evidence/document.pdf

# Search for obfuscated JavaScript
python3 pdf-parser.py -s JavaScript -r /evidence/document.pdf | \
    grep -i "eval\|unescape\|fromCharCode"
```

#### Scenario 3: Embedded File Extraction

```bash
# List all embedded files
python3 pdf-parser.py -f /evidence/document.pdf

# Extract embedded files
python3 pdf-parser.py -f -e -d -o /evidence/extracted/ /evidence/document.pdf

# Analyze extracted files
find /evidence/extracted/ -type f -exec file {} \;
```

#### Scenario 4: Batch Analysis

```bash
#!/bin/bash
# Analyze multiple PDFs

PDF_DIR="/evidence/pdfs"
OUTPUT="/evidence/analysis"
REPORT="$OUTPUT/batch_report.txt"

echo "Batch PDF Analysis" > "$REPORT"
echo "Date: $(date)" >> "$REPORT"
echo "" >> "$REPORT"

for pdf in "$PDF_DIR"/*.pdf; do
    echo "Analyzing: $pdf"
    echo "File: $pdf" >> "$REPORT"
    
    # Check for JavaScript
    JS_COUNT=$(python3 pdf-parser.py -j "$pdf" 2>/dev/null | grep -c "JavaScript")
    echo "  JavaScript objects: $JS_COUNT" >> "$REPORT"
    
    # Check for embedded files
    FILE_COUNT=$(python3 pdf-parser.py -f "$pdf" 2>/dev/null | grep -c "EmbeddedFile")
    echo "  Embedded files: $FILE_COUNT" >> "$REPORT"
    
    echo "" >> "$REPORT"
done
```

### Analyzing Results

```bash
# Count PDF objects
python3 pdf-parser.py -c /evidence/document.pdf

# List object types
python3 pdf-parser.py -a /evidence/document.pdf | grep -oP '/Type \/\w+' | sort | uniq -c

# Find specific objects
python3 pdf-parser.py -s Page /evidence/document.pdf
python3 pdf-parser.py -s Font /evidence/document.pdf

# Extract specific object
python3 pdf-parser.py -o 123 /evidence/document.pdf
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

### Phase 2: Analysis with PDF-Parser

```bash
# 1. Create analysis directory
mkdir -p /evidence/pdf_analysis/{streams,extracted,reports}

# 2. Analyze each PDF file
for pdf in /evidence/pdf_files/*.pdf; do
    echo "Analyzing: $pdf"
    
    # Generate report
    python3 pdf-parser.py "$pdf" > "/evidence/pdf_analysis/reports/$(basename $pdf).txt"
    
    # Check for JavaScript
    python3 pdf-parser.py -j "$pdf" >> "/evidence/pdf_analysis/reports/javascript.txt"
    
    # Check for embedded files
    python3 pdf-parser.py -f "$pdf" >> "/evidence/pdf_analysis/reports/embedded.txt"
    
    # Extract streams
    python3 pdf-parser.py -e -d -o "/evidence/pdf_analysis/streams/$(basename $pdf .pdf)/" "$pdf"
done
```

### Phase 3: Post-Analysis

```bash
# 1. Generate comprehensive report
cat > /evidence/pdf_analysis/summary.txt << EOF
PDF Analysis Summary
====================
Date: $(date)
PDF Files Analyzed: $(find /evidence/pdf_files/ -name "*.pdf" | wc -l)
Files with JavaScript: $(grep -l "JavaScript" /evidence/pdf_analysis/reports/*.txt | wc -l)
Files with Embedded Files: $(grep -l "EmbeddedFile" /evidence/pdf_analysis/reports/*.txt | wc -l)
Total Streams Extracted: $(find /evidence/pdf_analysis/streams/ -type f | wc -l)
EOF

# 2. Create malicious PDF list
grep -l "JavaScript" /evidence/pdf_analysis/reports/*.txt > /evidence/pdf_analysis/suspicious.txt

# 3. Generate file hashes
find /evidence/pdf_analysis/ -type f -exec md5sum {} \; > /evidence/pdf_analysis/hashes.txt
```

### Integration with Other Tools

```bash
# PDF-Parser + YARA
# Scan PDFs with YARA rules for known malware
yara -r /path/to/malware_rules/ /evidence/pdf_files/

# PDF-Parser + OleTools
# Analyze PDF with additional Office tools
oleid /evidence/suspicious.pdf

# PDF-Parser + pdftotext
# Extract text content
pdftotext /evidence/document.pdf /evidence/text.txt

# PDF-Parser + pdfid
# Quick identification of PDF features
pdfid /evidence/document.pdf

# PDF-Parser + strings
# Extract readable strings
strings /evidence/document.pdf | head -50
```

---

## Limitations and Considerations

### What PDF-Parser Cannot Do

1. **Execute PDF content**: Read-only analysis, cannot run JavaScript
2. **Decrypt encrypted PDFs**: Cannot bypass password protection
3. **Repair damaged PDFs**: Cannot fix corrupted files
4. **Parse all PDF versions**: May struggle with very new formats
5. **Replace specialized tools**: May need additional tools for complete analysis

### Known Limitations

- **Complex PDFs**: May not parse highly obfuscated files
- **Large files**: May be slow on very large PDFs
- **Limited documentation**: Fewer resources than commercial tools
- **No GUI**: Command-line only
- **Python dependency**: Requires Python environment

### When to Use Alternatives

| Situation | Better Tool | Reason |
|-----------|-------------|--------|
| Need GUI | PDF Stream Dumper | GUI interface |
| Need decryption | Specialized tools | Password bypass |
| Need PDF editing | Adobe Acrobat | Full PDF editor |
| Need text extraction | pdftotext | Better text handling |

---

## Detection and Defense

### For System Administrators

```bash
# Monitor for PDF-Parser usage
which pdf-parser.py

# Check for unauthorized PDF analysis
sudo ausearch -k pdf-parser -ts recent

# Monitor PDF file creation
find / -name "*.pdf" -mtime -1 2>/dev/null

# Check command history
cat ~/.bash_history | grep -i "pdf-parser\|pdf"
```

### For Forensic Investigators

```bash
# Check if PDF-Parser was used as anti-forensics tool
# Look for PDF file manipulation
find / -name "*.pdf" -mtime -1 2>/dev/null

# Check command history
cat /root/.bash_history | grep -i "pdf-parser"

# Examine system logs
sudo journalctl | grep -i "pdf-parser\|pdf"
```

### Defensive Measures

```bash
# 1. Block suspicious PDFs at email gateway
# Configure mail server to block PDFs with JavaScript

# 2. Use PDF sanitization tools
# Strip JavaScript and embedded files from incoming PDFs

# 3. Enable PDF viewers' security settings
# Disable JavaScript execution in Adobe Reader

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

# Run with appropriate user
python3 pdf-parser.py /path/to/document.pdf
```

#### "Cannot parse PDF"

```bash
# Check if file is valid PDF
file /path/to/document.pdf
head -5 /path/to/document.pdf

# Try with verbose mode
python3 pdf-parser.py -v /path/to/document.pdf

# Check for encryption
python3 pdf-parser.py -m /path/to/document.pdf | grep -i "encrypt"
```

#### "No JavaScript found"

```bash
# JavaScript may be obfuscated
python3 pdf-parser.py -j -r /path/to/document.pdf

# Search for common obfuscation patterns
python3 pdf-parser.py -a /path/to/document.pdf | grep -i "eval\|unescape"
```

#### "Memory error"

```bash
# For large PDF files
# Monitor memory usage
free -h

# Process in chunks if possible
# Split PDF using pdftk or similar tool
```

---

## Best Practices

### For Forensic Investigations

1. **Always work on copies**: Never analyze original PDF files
2. **Preserve metadata**: Use `cp --preserve=all` when copying
3. **Document everything**: Record commands and outputs
4. **Verify hashes**: MD5/SHA256 before and after
5. **Combine with other tools**: Use PDF-Parser alongside YARA and pdftotext

### For Malware Analysis

1. **Check JavaScript first**: Common attack vector
2. **Look for embedded files**: May contain payloads
3. **Examine streams**: May contain obfuscated code
4. **Check metadata**: May reveal author or creation tool
5. **Extract and analyze**: Get payloads for further analysis

### Configuration Tips

```bash
# 1. Start with metadata overview
# 2. Check for JavaScript before other analysis
# 3. Use verbose mode for debugging
# 4. Document all commands
# 5. Verify extracted data
```

---

## Comparison with Other Tools

### PDF-Parser vs pdfid vs PDFStreamDumper

| Feature | PDF-Parser | pdfid | PDFStreamDumper |
|---------|------------|-------|-----------------|
| Purpose | Structure analysis | Quick identification | Stream analysis |
| JavaScript Detection | Yes | Yes | Yes |
| Stream Extraction | Yes | No | Yes |
| GUI | No | No | Yes |
| Speed | Medium | Fast | Medium |
| Ease of Use | Medium | Easy | Easy |

### When to Choose PDF-Parser

- When you need detailed structure analysis
- When you need to extract and decode streams
- When working with complex PDFs
- As a comprehensive analysis tool

---

## References

- **PDF-Parser Documentation**: https://blog.didierstevens.com/2009/12/28/pdf-analysis/
- **Didier Stevens' Tools**: https://github.com/DidierStevens/DidierStevensSuite
- **Kali Linux Forensics**: https://www.kali.org/tools/pdf-parser/
- **PDF File Format**: https://www.adobe.com/devnet/pdf/pdf_reference.html
- **Malicious PDF Analysis**: https://www.sans.org/reading-room/malicious-pdf-analysis-1615

---

## Quick Reference Card

| Operation | Command |
|-----------|---------|
| Show structure | `python3 pdf-parser.py document.pdf` |
| Show metadata | `python3 pdf-parser.py -m document.pdf` |
| Find JavaScript | `python3 pdf-parser.py -j document.pdf` |
| Find embedded files | `python3 pdf-parser.py -f document.pdf` |
| Extract streams | `python3 pdf-parser.py -e -d document.pdf` |
| Show all objects | `python3 pdf-parser.py -a document.pdf` |
| Count objects | `python3 pdf-parser.py -c document.pdf` |
| Search type | `python3 pdf-parser.py -s Page document.pdf` |

---

*Document Version: 1.0*
*Last Updated: 2026*
*Author: Digital Forensics Documentation Project*
*Classification: Educational*
