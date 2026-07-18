---
tool_name: "metagoofil"
display_name: "Metagoofil"
mitre_tactic: "reconnaissance"
mitre_tactic_id: "TA0043"
mitre_subcategory: "host_information"
install_command: "sudo apt install metagoofil"
version: "2.6"
github_repo: "https://github.com/laramies/metagoofil"
kali_tools_page: "https://www.kali.org/tools/metagoofil/"
difficulty_level: "beginner-to-intermediate"
requires_root: false
gui_available: false
tags: ["metadata", "osint", "documents", "pdf", "docx", "forensics"]
related_tools: ["spiderfoot", "theharvester", "photon"]
---

# Metagoofil — Metadata Extraction from Public Documents

## What This Tool Does (in 30 seconds)

Metagoofil crawls the internet for publicly available documents (PDF, DOC, DOCX, PPT, ODT, XLS) hosted on a target domain, downloads them, and extracts metadata. That metadata can reveal employee names, email addresses, software versions, internal server paths, OS information, and more — all without touching the target's infrastructure.

---

# Chapter 1: Introduction & Overview

## What is Metagoofil?

Metagoofil is an open-source OSINT (Open Source Intelligence) tool that extracts metadata from publicly available files found through Google dorking. It searches Google for documents hosted on a target domain, downloads them, and runs metadata extraction to pull out hidden information that organizations often forget to sanitize.

The name comes from "metadata" + "goofil" (a play on "goofle" — Google dorking). It was created by Christian Martorella (laramies) from the Edge-Security team.

## History & Background

- **Created:** 2009 by Christian Martorella (Edge-Security)
- **Maintained:** Community/open-source
- **License:** GNU General Public License v2
- **Language:** Python
- **Purpose:** Extract metadata from public documents for OSINT reconnaissance

## How It Works (The Big Picture)

```
Google Dork Query → Find PDFs/DOCs on target.com → Download Documents →
Extract Metadata → Reveal Employee Names, Emails, Software Versions, Internal Paths
```

1. **Search Phase:** Uses Google search operators to find documents on the target domain
2. **Download Phase:** Fetches the documents it finds
3. **Extraction Phase:** Uses `pdfinfo`, `exiftool`, `OOXMLParser`, and other tools to pull metadata
4. **Reporting Phase:** Outputs a structured report with all extracted data

## What Metadata Reveals

| Metadata Field | What It Tells You | Security Impact |
|----------------|-------------------|-----------------|
| `Author` | Employee who created the document | Social engineering targets |
| `Creator` | Software used (e.g., Microsoft Word 2016) | Software version vulnerabilities |
| `Producer` | PDF producer (e.g., Adobe Acrobat) | Specific tool vulnerabilities |
| `Title` | Internal document title | Business intelligence |
| `Subject` | Document subject/purpose | Organizational structure |
| `Keywords` | Internal naming conventions | Project names, codenames |
| `LastModifiedBy` | Last editor's name | More employee names |
| `CreatedDate` | When document was created | Timeline of activities |
| `ModifiedDate` | Last modification date | Activity patterns |
| `TotalPage` | Number of pages | Document complexity |
| `Path` | Internal file paths (e.g., `C:\Users\jsmith\...`) | Username disclosure, internal structure |

## When to Use Metagoofil

- **Pre-engagement reconnaissance:** Before a pentest, gather intel about the target organization
- **OSINT investigations:** Research a company's employees, technology stack, and internal processes
- **Bug bounty recon:** Find information that might help identify vulnerabilities
- **Competitive intelligence:** Understand a competitor's tools and processes (legally)
- **Social engineering prep:** Identify real employees for phishing campaigns (with authorization)

## When NOT to Use Metagoofil

- **Without authorization:** Scraping and downloading documents from a target you don't have permission to test
- **Against privacy regulations:** GDPR, CCPA, and other privacy laws may restrict this type of data gathering
- **For stalking/harassment:** Using extracted personal information for malicious purposes
- **On internal-only targets:** If documents aren't publicly indexed, Metagoofil won't find them

## Legal and Ethical Considerations

### What's Legal
- Searching Google for publicly indexed documents
- Downloading publicly available files
- Extracting metadata from publicly available documents

### What's Gray Area
- Mass-downloading documents from a target (may violate ToS)
- Using extracted employee names for social engineering
- Publishing extracted metadata

### What's Illegal
- Downloading documents not intended for public access
- Using metadata to gain unauthorized access
- Distributing private information extracted from documents

### Best Practices
1. **Get written authorization** before conducting reconnaissance on a target
2. **Minimize data collection** — only download what you need
3. **Secure your findings** — metadata can contain sensitive info
4. **Report findings responsibly** — don't expose employee info publicly
5. **Respect robots.txt** — though Metagoofil doesn't check it, you should

## Key Concepts

- **Metadata:** Data about data — information embedded in files that describes the file's properties, creation details, and content
- **Google Dorking:** Using advanced Google search operators to find specific types of content
- **OSINT:** Open Source Intelligence — gathering intelligence from publicly available sources
- **Document Fingerprinting:** Identifying the software and version used to create a document
- **PII (Personally Identifiable Information):** Information that can identify a specific individual (names, emails, etc.)

---

# Chapter 2: Installation & Setup

## System Requirements

- **OS:** Linux (Kali Linux recommended), macOS, or Windows with WSL
- **Python:** Python 3.x
- **RAM:** 512 MB minimum (1 GB recommended for large document downloads)
- **Disk Space:** 1-10 GB depending on how many documents you download
- **Network:** Internet access (for Google searching and document downloading)
- **Privileges:** Normal user (no root required)

## External Dependencies

Metagoofil relies on several external tools for metadata extraction:

| Tool | Purpose | Install Command |
|------|---------|-----------------|
| `pdfinfo` | PDF metadata extraction | `sudo apt install poppler-utils` |
| `exiftool` | Universal metadata extraction | `sudo apt install libimage-exiftool-perl` |
| `readpst` | Outlook PST file parsing | `sudo apt install pst-utils` |
| `Olefile` | Office document parsing | `pip3 install olefile` |
| `libreoffice` | Office document conversion | `sudo apt install libreoffice` |

## Installation Methods

### Method 1: APT (Recommended)

```bash
# Update package lists
sudo apt update

# Install metagoofil
sudo apt install metagoofil

# Verify installation
metagoofil -h
```

### Method 2: From GitHub (Latest Version)

```bash
# Clone the repository
git clone https://github.com/laramies/metagoofil.git

# Navigate to the directory
cd metagoofil

# Install Python dependencies
pip3 install -r requirements.txt

# Make it executable
chmod +x metagoofil.py

# Test it
python3 metagoofil.py -h
```

### Method 3: Using Docker

```bash
# Build the image
docker build -t metagoofil .

# Run metagoofil
docker run -it metagoofil -d target.com -f pdf -o /output/results.html
```

## Installing Dependencies on Kali

```bash
# Install all required tools at once
sudo apt install -y \
    metagoofil \
    poppler-utils \
    libimage-exiftool-perl \
    pst-utils \
    libreoffice

# Install Python dependencies
pip3 install -r /usr/share/metagoofil/requirements.txt
```

## Verifying Installation

```bash
# Check metagoofil version
metagoofil -h

# Expected output:
# Metagoofil v2.6 - Web document metadata harvester
# Usage: metagoofil [options]

# Verify pdfinfo is available
pdfinfo -v

# Verify exiftool is available
exiftool -ver
```

## Configuration

### Default Configuration

Metagoofil works out of the box with no configuration needed. However, you can customize:

```bash
# Set Google API key (for higher rate limits)
export GOOGLE_API_KEY="your-api-key"

# Set custom user agent (to avoid blocking)
export METAGOOFIL_USERAGENT="Mozilla/5.0 (compatible; research bot)"
```

### Rate Limiting Configuration

```bash
# Metagoofil has built-in delays to avoid Google blocking
# Default: 10 second delay between requests
# You can modify this in the source code or use -d flag

# Slower (safer) scanning
metagoofil -d target.com -d 15  # 15 second delay

# Faster scanning (risky - may get blocked)
metagoofil -d target.com -d 5   # 5 second delay
```

## Troubleshooting Installation

### Issue: "pdfinfo: command not found"
```bash
sudo apt install poppler-utils
```

### Issue: "exiftool: command not found"
```bash
sudo apt install libimage-exiftool-perl
```

### Issue: "ModuleNotFoundError: No module named 'olefile'"
```bash
pip3 install olefile
```

### Issue: "Permission denied" on output directory
```bash
# Create output directory with proper permissions
mkdir -p ~/metagoofil_output
chmod 755 ~/metagoofil_output
```

---

# Chapter 3: Basic Usage

## First Run

```bash
# Search for PDF files on a target domain
metagoofil -d example.com -f pdf -o results/
```

**What happens:**
1. Metagoofil searches Google for `site:example.com filetype:pdf`
2. Downloads all PDF files found
3. Extracts metadata from each PDF
4. Saves results to `results/` directory

## Essential Commands

### Basic Document Search

```bash
# Search for PDFs on a domain
metagoofil -d example.com -f pdf -o results/

# Search for PDFs AND DOC files
metagoofil -d example.com -f pdf,doc -o results/

# Search for all supported file types
metagoofil -d example.com -f pdf,doc,docx,ppt,pptx,odt,xls,xlsx -o results/
```

### Limiting Results

```bash
# Limit to 50 documents maximum
metagoofil -d example.com -f pdf -l 50 -o results/

# Limit to 10 documents (for quick testing)
metagoofil -d example.com -f pdf -l 10 -o results/
```

### Output Control

```bash
# Save results in HTML format (default)
metagoofil -d example.com -f pdf -o results/

# Save results in CSV format
metagoofil -d example.com -f pdf -o results/ -F csv

# Save results in JSON format
metagoofil -d example.com -f pdf -o results/ -F json
```

## Command-Line Options Reference

### Target Specification

| Flag | Description | Example |
|------|-------------|---------|
| `-d` | Target domain | `-d example.com` |
| `-t` | File types to search | `-t pdf,doc,docx` |
| `-f` | File type filter (same as -t) | `-f pdf` |

### Search Control

| Flag | Description | Example |
|------|-------------|---------|
| `-l` | Maximum number of results | `-l 100` |
| `-n` | Number of results per page | `-n 10` |
| `-s` | Start result number | `-s 20` |
| `-p` | Pages to search | `-p 5` |

### Output Options

| Flag | Description | Example |
|------|-------------|---------|
| `-o` | Output directory | `-o /tmp/results` |
| `-F` | Output format (csv/html/json) | `-F csv` |
| `-e` | Export metadata to file | `-e metadata.txt` |

### Download Control

| Flag | Description | Example |
|------|-------------|---------|
| `-d` | Delay between requests (seconds) | `-d 10` |
| `-u` | Custom user agent | `-u "MyBot/1.0"` |
| `-c` | Cookie file for authentication | `-c cookies.txt` |

### Advanced Options

| Flag | Description | Example |
|------|-------------|---------|
| `--timeout` | Download timeout | `--timeout 30` |
| `--threads` | Number of threads | `--threads 5` |
| `--verbose` | Verbose output | `--verbose` |
| `--debug` | Debug mode | `--debug` |

## Complete Example: Basic Scan

```bash
# Step 1: Create output directory
mkdir -p ~/metagoofil_results

# Step 2: Run metagoofil
metagoofil \
    -d example.com \
    -f pdf,doc,docx \
    -l 100 \
    -o ~/metagoofil_results \
    -F html

# Step 3: View results
ls ~/metagoofil_results/
# index.html  (main report)
# downloads/  (downloaded documents)
# metadata/   (extracted metadata files)
```

## Understanding the Output

### HTML Report Structure

```html
<!-- The HTML report contains: -->
1. Summary Table
   - Total files found
   - Files downloaded
   - Metadata extracted

2. Document Details
   - File name
   - Download URL
   - File type
   - File size
   - Metadata fields

3. Metadata Summary
   - All authors found
   - All email addresses found
   - All software versions found
   - All internal paths found
```

### Key Metadata Fields to Look For

```
Author: John Smith (jsmith@company.com)
Creator: Microsoft Office Word
Producer: Adobe PDF Library 15.0
Title: Q3 Financial Report - CONFIDENTIAL
Subject: Internal financial projections
Path: C:\Users\jsmith\Documents\Q3_Financial_Report.docx
CreatedDate: 2024-01-15
ModifiedDate: 2024-03-20
LastModifiedBy: jsmith
TotalPage: 15
```

## Quick Reference: Common Use Cases

### Scenario 1: Quick Reconnaissance
```bash
# Find 10 PDFs, extract metadata, save as HTML
metagoofil -d target.com -f pdf -l 10 -o quick_results/
```

### Scenario 2: Comprehensive Document Mining
```bash
# Find all document types, up to 500 files, save as CSV
metagoofil -d target.com -f pdf,doc,docx,ppt,pptx,odt,xls,xlsx -l 500 -o comprehensive_results/ -F csv
```

### Scenario 3: Targeted Employee Research
```bash
# Search for documents by specific author
metagoofil -d target.com -f pdf -l 50 -o targeted_results/
# Then grep the metadata for specific names
grep -r "Author:" targeted_results/metadata/
```

---

# Chapter 4: Intermediate Usage

## Output Format Deep Dive

### HTML Output (Default)

The HTML report is the most readable format:

```bash
metagoofil -d example.com -f pdf -o results/ -F html
```

**Structure:**
```
results/
├── index.html          # Main report with all metadata
├── downloads/          # Downloaded documents
│   ├── document1.pdf
│   └── document2.docx
└── metadata/           # Extracted metadata files
    ├── document1.json
    └── document2.json
```

### CSV Output (For Analysis)

```bash
metagoofil -d example.com -f pdf -o results/ -F csv
```

**CSV Structure:**
```csv
filename,url,filetype,author,creator,producer,title,subject,keywords,path,created,modified
report.pdf,https://example.com/report.pdf,PDF,John Smith,Microsoft Word,Adobe Acrobat,Q3 Report,Financial,quarterly,C:\Users\jsmith\docs,2024-01-15,2024-03-20
```

**Use Cases:**
- Import into Excel/Google Sheets for analysis
- Sort by author, date, or software version
- Filter for specific patterns

### JSON Output (For Automation)

```bash
metagoofil -d example.com -f pdf -o results/ -F json
```

**JSON Structure:**
```json
{
  "domain": "example.com",
  "total_files": 25,
  "files": [
    {
      "filename": "report.pdf",
      "url": "https://example.com/report.pdf",
      "metadata": {
        "Author": "John Smith",
        "Creator": "Microsoft Word",
        "Producer": "Adobe Acrobat",
        "Title": "Q3 Financial Report",
        "Path": "C:\\Users\\jsmith\\docs\\report.pdf"
      }
    }
  ]
}
```

**Use Cases:**
- Programmatic processing with Python/JavaScript
- Integration with other security tools
- Custom reporting and visualization

## Python Automation Scripts

### Script 1: Batch Metadata Analysis

```python
#!/usr/bin/env python3
"""
Batch analyze Metagoofil JSON output
Extracts unique authors, emails, software versions, and file paths
"""

import json
import os
from collections import Counter
from pathlib import Path

def analyze_metadata(json_file):
    """Parse Metagoofil JSON output and extract insights"""
    with open(json_file, 'r') as f:
        data = json.load(f)
    
    authors = []
    emails = []
    software = []
    paths = []
    
    for file_info in data.get('files', []):
        metadata = file_info.get('metadata', {})
        
        if 'Author' in metadata:
            authors.append(metadata['Author'])
        
        if 'Creator' in metadata:
            software.append(metadata['Creator'])
        
        if 'Path' in metadata:
            paths.append(metadata['Path'])
            
        # Extract emails from all metadata fields
        for value in metadata.values():
            if isinstance(value, str) and '@' in value:
                emails.append(value)
    
    return {
        'authors': Counter(authors),
        'emails': list(set(emails)),
        'software': Counter(software),
        'paths': paths
    }

def generate_report(analysis, output_file):
    """Generate a human-readable report"""
    with open(output_file, 'w') as f:
        f.write("=== Metagoofil Analysis Report ===\n\n")
        
        f.write("--- Unique Authors ---\n")
        for author, count in analysis['authors'].most_common():
            f.write(f"  {author}: {count} documents\n")
        
        f.write("\n--- Email Addresses Found ---\n")
        for email in analysis['emails']:
            f.write(f"  {email}\n")
        
        f.write("\n--- Software Versions ---\n")
        for sw, count in analysis['software'].most_common():
            f.write(f"  {sw}: {count} documents\n")
        
        f.write("\n--- Internal File Paths ---\n")
        for path in analysis['paths'][:20]:  # Limit to 20
            f.write(f"  {path}\n")

if __name__ == '__main__':
    import sys
    if len(sys.argv) != 2:
        print(f"Usage: {sys.argv[0]} <metagoofil_output.json>")
        sys.exit(1)
    
    analysis = analyze_metadata(sys.argv[1])
    generate_report(analysis, 'analysis_report.txt')
    print(f"Report generated: analysis_report.txt")
```

### Script 2: Employee Discovery

```python
#!/usr/bin/env python3
"""
Extract employee information from Metagoofil metadata
Finds names, emails, and organizational patterns
"""

import json
import re
from collections import defaultdict

def extract_employees(json_file):
    """Extract employee information from metadata"""
    with open(json_file, 'r') as f:
        data = json.load(f)
    
    employees = defaultdict(lambda: {'emails': [], 'documents': [], 'software': set()})
    
    for file_info in data.get('files', []):
        metadata = file_info.get('metadata', {})
        filename = file_info.get('filename', 'unknown')
        
        author = metadata.get('Author', '')
        email = metadata.get('Email', '')
        creator = metadata.get('Creator', '')
        
        if author:
            employees[author]['documents'].append(filename)
            employees[author]['software'].add(creator)
            if email:
                employees[author]['emails'].append(email)
    
    return employees

def find_domain_emails(domain, json_file):
    """Find all email addresses belonging to a specific domain"""
    with open(json_file, 'r') as f:
        data = json.load(f)
    
    domain_emails = set()
    
    for file_info in data.get('files', []):
        metadata = file_info.get('metadata', {})
        
        for value in metadata.values():
            if isinstance(value, str):
                # Find emails matching domain
                emails = re.findall(rf'[\w.-]+@{re.escape(domain)}', value)
                domain_emails.update(emails)
    
    return list(domain_emails)

if __name__ == '__main__':
    import sys
    if len(sys.argv) != 3:
        print(f"Usage: {sys.argv[0]} <metagoofil_output.json> <domain>")
        sys.exit(1)
    
    employees = extract_employees(sys.argv[1])
    print("=== Employee Analysis ===\n")
    for name, info in sorted(employees.items()):
        print(f"Name: {name}")
        print(f"  Documents: {len(info['documents'])}")
        print(f"  Software: {', '.join(info['software'])}")
        print()
    
    emails = find_domain_emails(sys.argv[2], sys.argv[1])
    print(f"\n=== Emails Found on {sys.argv[2]} ===")
    for email in sorted(emails):
        print(f"  {email}")
```

## Bash Automation Scripts

### Script 1: Automated Report Generator

```bash
#!/bin/bash
# automate_metagoofil.sh - Automated Metagoofil scanning and reporting

TARGET_DOMAIN="$1"
OUTPUT_DIR="metagoofil_$(date +%Y%m%d_%H%M%S)"
FILE_TYPES="pdf,doc,docx,ppt,pptx,odt,xls,xlsx"
MAX_RESULTS=100

if [ -z "$TARGET_DOMAIN" ]; then
    echo "Usage: $0 <domain>"
    echo "Example: $0 example.com"
    exit 1
fi

echo "[*] Starting Metagoofil scan on $TARGET_DOMAIN"
echo "[*] Output directory: $OUTPUT_DIR"

# Create output directory
mkdir -p "$OUTPUT_DIR"

# Run Metagoofil
echo "[*] Searching for documents..."
metagoofil \
    -d "$TARGET_DOMAIN" \
    -f "$FILE_TYPES" \
    -l "$MAX_RESULTS" \
    -o "$OUTPUT_DIR" \
    -F csv \
    -d 10

echo "[*] Scan complete. Results saved to $OUTPUT_DIR"

# Generate summary
echo "[*] Generating summary..."
echo "=== Metagoofil Summary ===" > "$OUTPUT_DIR/summary.txt"
echo "Target: $TARGET_DOMAIN" >> "$OUTPUT_DIR/summary.txt"
echo "Date: $(date)" >> "$OUTPUT_DIR/summary.txt"
echo "Files found: $(find "$OUTPUT_DIR/downloads" -type f 2>/dev/null | wc -l)" >> "$OUTPUT_DIR/summary.txt"
echo "" >> "$OUTPUT_DIR/summary.txt"

# Extract unique authors
echo "--- Unique Authors ---" >> "$OUTPUT_DIR/summary.txt"
grep -h "Author:" "$OUTPUT_DIR/metadata/"*.txt 2>/dev/null | sort | uniq -c | sort -rn >> "$OUTPUT_DIR/summary.txt"

# Extract email addresses
echo "" >> "$OUTPUT_DIR/summary.txt"
echo "--- Email Addresses ---" >> "$OUTPUT_DIR/summary.txt"
grep -rhoE '[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}' "$OUTPUT_DIR/" 2>/dev/null | sort | uniq >> "$OUTPUT_DIR/summary.txt"

echo "[*] Summary saved to $OUTPUT_DIR/summary.txt"
cat "$OUTPUT_DIR/summary.txt"
```

### Script 2: Monitor and Alert

```bash
#!/bin/bash
# monitor_metagoofil.sh - Monitor for new documents on target domain

TARGET_DOMAIN="$1"
INTERVAL=3600  # Check every hour
LOG_FILE="metagoofil_monitor.log"

if [ -z "$TARGET_DOMAIN" ]; then
    echo "Usage: $0 <domain>"
    exit 1
fi

echo "[*] Starting Metagoofil monitor for $TARGET_DOMAIN"
echo "[*] Checking every $INTERVAL seconds"

while true; do
    TIMESTAMP=$(date +"%Y-%m-%d %H:%M:%S")
    
    # Run Metagoofil with small limit
    OUTPUT_DIR="/tmp/metagoofil_monitor_$(date +%s)"
    mkdir -p "$OUTPUT_DIR"
    
    metagoofil \
        -d "$TARGET_DOMAIN" \
        -f pdf,doc,docx \
        -l 10 \
        -o "$OUTPUT_DIR" \
        -F json \
        -d 5 2>/dev/null
    
    # Check for new documents
    if [ -f "$OUTPUT_DIR/results.json" ]; then
        NEW_FILES=$(python3 -c "
import json
with open('$OUTPUT_DIR/results.json') as f:
    data = json.load(f)
print(len(data.get('files', [])))
" 2>/dev/null)
        
        echo "[$TIMESTAMP] Found $NEW_FILES documents" >> "$LOG_FILE"
        
        if [ "$NEW_FILES" -gt 0 ]; then
            echo "[$TIMESTAMP] NEW DOCUMENTS FOUND on $TARGET_DOMAIN!"
            # Add your alert mechanism here (email, Slack, etc.)
        fi
    fi
    
    # Cleanup
    rm -rf "$OUTPUT_DIR"
    
    # Wait before next check
    sleep "$INTERVAL"
done
```

## Integration with Other Tools

### Integration with TheHarvester

```bash
# Step 1: Gather email addresses with TheHarvester
theharvester -d example.com -b google -f harvester_results.html

# Step 2: Extract emails from Metagoofil metadata
metagoofil -d example.com -f pdf -o meta_results/

# Step 3: Combine results
cat harvester_results.html | grep -oE '[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}' | sort | uniq > emails.txt
grep -rhoE '[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}' meta_results/ | sort | uniq >> emails.txt
sort emails.txt | uniq > combined_emails.txt
```

### Integration with SpiderFoot

```bash
# Use SpiderFoot to discover the target's assets
spiderfoot -s example.com

# Then use Metagoofil to mine documents
metagoofil -d example.com -f pdf,doc -o metagoofil_results/

# Cross-reference findings
echo "SpiderFoot found these hosts:"
spiderfoot -s example.com | grep "Host"

echo "Metagoofil found these documents:"
metagoofil -d example.com -f pdf -l 50 -o /tmp/results/
ls /tmp/results/downloads/
```

### Integration with Nmap

```bash
# Discover web servers
nmap -p 80,443,8080,8443 -oG - 192.168.1.0/24 | awk '/open/{print $2}' > web_hosts.txt

# For each discovered web server, run Metagoofil
while read host; do
    echo "[*] Scanning $host"
    metagoofil -d "$host" -f pdf -l 20 -o "results_$host/"
done < web_hosts.txt
```

## Advanced Output Processing

### Parsing CSV with Python

```python
#!/usr/bin/env python3
"""Parse Metagoofil CSV output for analysis"""

import csv
import sys
from collections import Counter

def analyze_csv(csv_file):
    """Analyze Metagoofil CSV output"""
    with open(csv_file, 'r') as f:
        reader = csv.DictReader(f)
        
        authors = Counter()
        software = Counter()
        file_types = Counter()
        
        for row in reader:
            if row.get('author'):
                authors[row['author']] += 1
            if row.get('creator'):
                software[row['creator']] += 1
            if row.get('filetype'):
                file_types[row['filetype']] += 1
    
    print("=== Authors ===")
    for author, count in authors.most_common(10):
        print(f"  {author}: {count}")
    
    print("\n=== Software ===")
    for sw, count in software.most_common(10):
        print(f"  {sw}: {count}")
    
    print("\n=== File Types ===")
    for ft, count in file_types.most_common():
        print(f"  {ft}: {count}")

if __name__ == '__main__':
    if len(sys.argv) != 2:
        print(f"Usage: {sys.argv[0]} <metagoofil_results.csv>")
        sys.exit(1)
    analyze_csv(sys.argv[1])
```

### Converting to Other Formats

```bash
# Convert HTML report to PDF
wkhtmltopdf results/index.html results/report.pdf

# Extract specific metadata fields
grep -h "Author:" results/metadata/*.txt | sort | uniq

# Find documents with specific software versions
grep -h "Creator: Microsoft Word 2016" results/metadata/*.txt

# Find documents with internal paths
grep -h "Path: C:\\" results/metadata/*.txt
```

---

# Chapter 5: Advanced Usage

## Custom Google Dorking

Metagoofil uses Google dorks internally, but you can control and extend them:

### Understanding the Default Dorks

```
# Metagoofil generates these Google queries:
site:example.com filetype:pdf
site:example.com filetype:doc
site:example.com filetype:docx
site:example.com filetype:ppt
site:example.com filetype:pptx
site:example.com filetype:odt
site:example.com filetype:xls
site:example.com filetype:xlsx
```

### Custom File Type Searches

```bash
# Search for specific file extensions
metagoofil -d example.com -t pdf,doc,docx -o results/

# Search for less common types
metagoofil -d example.com -t rtf,wpd,indd -o results/
```

### Combining with External Dork Lists

```bash
# Use SecLists for advanced dorking
# First, manually search with dorks, then feed URLs to Metagoofil

# Create a custom dork list
cat > custom_dorks.txt << 'EOF'
site:example.com filetype:pdf "confidential"
site:example.com filetype:doc "internal"
site:example.com filetype:xls "financial"
EOF

# Use with Google manually, then pipe results to Metagoofil
```

## Evasion Techniques

### Rate Limit Evasion

```bash
# Slow down requests to avoid Google blocking
metagoofil -d example.com -f pdf -o results/ -d 30  # 30 second delay

# Random delay (modify source code)
# Edit metagoofil.py and add:
import random
time.sleep(random.randint(10, 30))
```

### User-Agent Rotation

```bash
# Use different user agents
UA_LIST=("Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36"
         "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7)"
         "Mozilla/5.0 (X11; Linux x86_64)")

for ua in "${UA_LIST[@]}"; do
    metagoofil -d example.com -f pdf -u "$ua" -o "results_$(echo $ua | cut -d'(' -f2)/"
    sleep 60
done
```

### Proxy Integration

```bash
# Route through proxy
export http_proxy="http://proxy:8080"
export https_proxy="http://proxy:8080"
metagoofil -d example.com -f pdf -o results/
```

## Scaling for Large Targets

### Parallel Processing

```bash
#!/bin/bash
# parallel_metagoofil.sh - Run multiple Metagoofil instances

DOMAINS=("example.com" "target.com" "victim.org")

for domain in "${DOMAINS[@]}"; do
    echo "[*] Starting scan on $domain"
    metagoofil -d "$domain" -f pdf,doc,docx -l 200 -o "results_$domain/" &
done

echo "[*] All scans started. Waiting for completion..."
wait
echo "[*] All scans complete"
```

### Resource Management

```bash
# Limit memory usage
ulimit -v 2000000

# Limit CPU usage
cpulimit -l 50 -p $(pgrep -f metagoofil)

# Run in background with nohup
nohup metagoofil -d example.com -f pdf -o results/ > scan.log 2>&1 &
```

## Custom Metadata Extraction

### Extracting Additional Fields

```python
#!/usr/bin/env python3
"""
Extended metadata extraction - fields Metagoofil might miss
"""

import subprocess
import json
from pathlib import Path

def extract_extended_metadata(file_path):
    """Extract metadata using exiftool for more fields"""
    try:
        result = subprocess.run(
            ['exiftool', '-json', str(file_path)],
            capture_output=True, text=True
        )
        return json.loads(result.stdout)[0]
    except Exception as e:
        return {'error': str(e)}

def process_directory(directory):
    """Process all files in a directory"""
    results = []
    
    for file_path in Path(directory).rglob('*'):
        if file_path.is_file():
            metadata = extract_extended_metadata(file_path)
            results.append({
                'file': str(file_path),
                'metadata': metadata
            })
    
    return results

if __name__ == '__main__':
    import sys
    if len(sys.argv) != 2:
        print(f"Usage: {sys.argv[0]} <metagoofil_downloads_dir>")
        sys.exit(1)
    
    results = process_directory(sys.argv[1])
    
    # Save extended metadata
    with open('extended_metadata.json', 'w') as f:
        json.dump(results, f, indent=2, default=str)
    
    print(f"Processed {len(results)} files")
    print("Extended metadata saved to extended_metadata.json")
```

### Custom Script Integration

```bash
#!/bin/bash
# custom_extract.sh - Extract metadata not covered by Metagoofil

RESULTS_DIR="$1"

if [ -z "$RESULTS_DIR" ]; then
    echo "Usage: $0 <metagoofil_results_dir>"
    exit 1
fi

echo "[*] Extracting extended metadata..."

# Find all downloaded files
find "$RESULTS_DIR/downloads" -type f | while read file; do
    echo "[*] Processing: $file"
    
    # Extract EXIF data
    exiftool "$file" > "$RESULTS_DIR/metadata/$(basename "$file").exif" 2>/dev/null
    
    # Extract PDF specific metadata
    if [[ "$file" == *.pdf ]]; then
        pdfinfo "$file" > "$RESULTS_DIR/metadata/$(basename "$file").pdfinfo" 2>/dev/null
    fi
    
    # Extract Office document metadata
    if [[ "$file" == *.docx ]] || [[ "$file" == *.xlsx ]]; then
        python3 -c "
from zipfile import ZipFile
import xml.etree.ElementTree as ET

with ZipFile('$file') as z:
    with z.open('docProps/core.xml') as f:
        tree = ET.parse(f)
        root = tree.getroot()
        for child in root:
            print(f'{child.tag}: {child.text}')
" > "$RESULTS_DIR/metadata/$(basename "$file").ooxml" 2>/dev/null
    fi
done

echo "[*] Extended metadata extraction complete"
```

## Automation Frameworks

### Ansible Playbook

```yaml
---
# ansible_metagoofil.yml
- name: Run Metagoofil on multiple targets
  hosts: localhost
  vars:
    targets:
      - example.com
      - target.com
      - victim.org
    output_base: /opt/metagoofil_results
  
  tasks:
    - name: Create output directory
      file:
        path: "{{ output_base }}"
        state: directory
        
    - name: Run Metagoofil
      command: >
        metagoofil -d {{ item }} -f pdf,doc,docx 
        -l 200 -o {{ output_base }}/{{ item }}
      loop: "{{ targets }}"
      register: results
      
    - name: Generate summary report
      shell: |
        for dir in {{ output_base }}/*/; do
          domain=$(basename "$dir")
          count=$(find "$dir/downloads" -type f 2>/dev/null | wc -l)
          echo "$domain: $count files"
        done
      register: summary
      
    - name: Display results
      debug:
        var: summary.stdout_lines
```

### Cron Job Setup

```bash
# Edit crontab
crontab -e

# Add daily scan
0 2 * * * /opt/scripts/metagoofil_auto.sh example.com >> /var/log/metagoofil.log 2>&1

# Add weekly report generation
0 8 * * 1 /opt/scripts/metagoofil_report.sh >> /var/log/metagoofil_report.log 2>&1
```

## Advanced Analysis Techniques

### Cross-Referencing with Other Data Sources

```python
#!/usr/bin/env python3
"""
Cross-reference Metagoofil results with other OSINT sources
"""

import json
import requests
from typing import Dict, List

def check_email_breach(email: str) -> Dict:
    """Check if email appears in known breaches"""
    # Note: This is a placeholder - use Have I Been Pwned API or similar
    return {'email': email, 'breaches': []}

def analyze_metagoofil_results(json_file: str) -> Dict:
    """Analyze Metagoofil results and enrich with external data"""
    with open(json_file, 'r') as f:
        data = json.load(f)
    
    enriched = {
        'emails': [],
        'employees': {},
        'software_versions': [],
        'internal_paths': []
    }
    
    for file_info in data.get('files', []):
        metadata = file_info.get('metadata', {})
        
        # Extract emails
        for value in metadata.values():
            if isinstance(value, str) and '@' in value:
                if value not in enriched['emails']:
                    enriched['emails'].append(value)
        
        # Extract employees
        author = metadata.get('Author', '')
        if author:
            if author not in enriched['employees']:
                enriched['employees'][author] = {
                    'documents': [],
                    'software': set()
                }
            enriched['employees'][author]['documents'].append(file_info.get('filename'))
            enriched['employees'][author]['software'].add(metadata.get('Creator', 'Unknown'))
        
        # Extract software versions
        creator = metadata.get('Creator', '')
        if creator and creator not in enriched['software_versions']:
            enriched['software_versions'].append(creator)
        
        # Extract internal paths
        path = metadata.get('Path', '')
        if path and 'C:\\' in path:
            enriched['internal_paths'].append(path)
    
    # Convert sets to lists for JSON serialization
    for emp in enriched['employees'].values():
        emp['software'] = list(emp['software'])
    
    return enriched

if __name__ == '__main__':
    import sys
    if len(sys.argv) != 2:
        print(f"Usage: {sys.argv[0]} <metagoofil_results.json>")
        sys.exit(1)
    
    results = analyze_metagoofil_results(sys.argv[1])
    
    print("=== Enriched Analysis ===")
    print(f"\nEmails found: {len(results['emails'])}")
    for email in results['emails']:
        print(f"  - {email}")
    
    print(f"\nEmployees found: {len(results['employees'])}")
    for name, info in results['employees'].items():
        print(f"  - {name}: {len(info['documents'])} documents")
    
    print(f"\nSoftware versions: {len(results['software_versions'])}")
    for sw in results['software_versions']:
        print(f"  - {sw}")
    
    print(f"\nInternal paths: {len(results['internal_paths'])}")
    for path in results['internal_paths'][:10]:  # Limit to 10
        print(f"  - {path}")
    
    # Save enriched results
    with open('enriched_results.json', 'w') as f:
        json.dump(results, f, indent=2)
    print("\nEnriched results saved to enriched_results.json")
```

---

# Chapter 6: Real-World Scenarios

## Scenario 1: Pre-Engagement Reconnaissance

### Objective
Before a penetration test, gather intelligence about the target organization from publicly available documents.

### Steps

```bash
# Phase 1: Domain Discovery
# First, identify all domains owned by the target
amass enum -passive -d target.com > domains.txt

# Phase 2: Document Discovery
# Search for documents across all identified domains
while read domain; do
    echo "[*] Scanning $domain"
    metagoofil -d "$domain" -f pdf,doc,docx,ppt,pptx,odt,xls,xlsx -l 100 -o "results_$domain/" -F csv
done < domains.txt

# Phase 3: Metadata Analysis
# Extract key intelligence from metadata
echo "=== Unique Authors ==="
find results_* -name "*.csv" -exec cat {} \; | cut -d',' -f4 | sort | uniq -c | sort -rn

echo "=== Email Addresses ==="
find results_* -name "*.csv" -exec cat {} \; | grep -oE '[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}' | sort | uniq

echo "=== Software Versions ==="
find results_* -name "*.csv" -exec cat {} \; | cut -d',' -f6 | sort | uniq -c | sort -rn

echo "=== Internal Paths ==="
find results_* -name "*.csv" -exec cat {} \; | grep "C:\\\\\" | cut -d',' -f10 | sort | uniq
```

### Expected Output
- List of employee names (for social engineering)
- Email addresses (for phishing)
- Software versions (for vulnerability research)
- Internal file paths (for understanding infrastructure)

### Value to Pentester
- **Social Engineering:** Real employee names and emails for targeted phishing
- **Technology Stack:** Software versions to identify known vulnerabilities
- **Internal Structure:** File paths reveal internal naming conventions and infrastructure

---

## Scenario 2: Bug Bounty Reconnaissance

### Objective
Find information that might lead to vulnerabilities in a bug bounty program.

### Steps

```bash
# Step 1: Run Metagoofil on the target
metagoofil -d target.com -f pdf,doc,docx -l 200 -o bounty_results/

# Step 2: Analyze for potential vulnerabilities
echo "=== Potential Findings ==="

# Find documents mentioning "internal", "confidential", or "draft"
grep -ri "internal\|confidential\|draft\|test\|staging" bounty_results/metadata/ 2>/dev/null

# Find documents with old software versions
grep -h "Creator:" bounty_results/metadata/*.txt 2>/dev/null | sort | uniq -c | sort -rn

# Find documents with internal paths
grep -h "Path: C:\\\\" bounty_results/metadata/*.txt 2>/dev/null | head -20

# Step 3: Check for exposed credentials or sensitive info
grep -ri "password\|api.key\|token\|secret" bounty_results/downloads/ 2>/dev/null
```

### What to Look For
1. **Draft documents** — May contain unpatched vulnerabilities or business logic flaws
2. **Test documents** — Often left on production servers with weak access controls
3. **Old software versions** — May have known CVEs
4. **Internal paths** — Can reveal admin panels or internal tools

---

## Scenario 3: Social Engineering Campaign

### Objective
Gather employee information for a social engineering assessment.

### Steps

```bash
# Step 1: Discover employee documents
metagoofil -d target.com -f pdf,doc,docx -l 500 -o se_results/

# Step 2: Extract employee information
python3 << 'EOF'
import csv
import json
from collections import defaultdict

employees = defaultdict(lambda: {
    'documents': [],
    'software': set(),
    'paths': []
})

with open('se_results/results.csv', 'r') as f:
    reader = csv.DictReader(f)
    for row in reader:
        author = row.get('author', '')
        if author:
            employees[author]['documents'].append(row.get('filename', ''))
            employees[author]['software'].add(row.get('creator', ''))
            employees[author]['paths'].append(row.get('path', ''))

# Generate employee profiles
for name, info in employees.items():
    print(f"\n=== {name} ===")
    print(f"  Documents: {len(info['documents'])}")
    print(f"  Software: {', '.join(info['software'])}")
    if info['paths']:
        print(f"  Working Directory: {info['paths'][0]}")
EOF
```

### Social Engineering Vectors
- **Email addresses** for phishing campaigns
- **Employee names** for impersonation
- **Software versions** for targeted exploits
- **Internal paths** for pretexting (e.g., "I see you're working on the Q3 report...")

---

## Scenario 4: Competitive Intelligence

### Objective
Research a competitor's technology stack and processes.

### Steps

```bash
# Step 1: Gather competitor documents
metagoofil -d competitor.com -f pdf,doc,docx,ppt,pptx -l 300 -o competitor_results/

# Step 2: Analyze technology stack
echo "=== Technology Analysis ==="
echo ""
echo "Software Versions:"
grep -h "Creator:" competitor_results/metadata/*.txt 2>/dev/null | sort | uniq -c | sort -rn

echo ""
echo "PDF Producers:"
grep -h "Producer:" competitor_results/metadata/*.txt 2>/dev/null | sort | uniq -c | sort -rn

echo ""
echo "Document Patterns:"
grep -h "Title:" competitor_results/metadata/*.txt 2>/dev/null | sort | uniq
```

### Intelligence Value
- **Technology stack** — What tools they use
- **Software versions** — Potential vulnerabilities
- **Document patterns** — Business processes and workflows
- **Employee structure** — Organizational insights

---

## Scenario 5: Incident Response Investigation

### Objective
Investigate a data breach by analyzing metadata from leaked documents.

### Steps

```bash
# Step 1: Collect leaked documents
mkdir -p investigation/leaked_docs
# (Documents provided by incident response team)

# Step 2: Extract metadata
for file in investigation/leaked_docs/*; do
    exiftool "$file" > "investigation/metadata/$(basename "$file").meta" 2>/dev/null
done

# Step 3: Analyze for indicators
echo "=== Investigation Findings ==="

# Find document authors
echo "Document Authors:"
grep -h "Author:" investigation/metadata/*.meta 2>/dev/null | sort | uniq -c | sort -rn

# Find creation dates
echo "Creation Dates:"
grep -h "Create Date:" investigation/metadata/*.meta 2>/dev/null | sort

# Find software versions
echo "Software Versions:"
grep -h "Creator:" investigation/metadata/*.meta 2>/dev/null | sort | uniq -c | sort -rn

# Find internal paths
echo "Internal Paths:"
grep -h "Internal File Name:" investigation/metadata/*.meta 2>/dev/null | sort | uniq
```

### Forensic Value
- **Timeline reconstruction** — When documents were created/modified
- **Attribution** — Who created the documents
- **Scope assessment** — How many documents were compromised
- **Infrastructure mapping** — Internal paths reveal network structure

---

# Chapter 7: Detection & Defense

## How Defenders Detect Metagoofil

### Google Rate Limiting Detection

Google actively monitors for automated search queries. Metagoofil can be detected when:

1. **High query rate** — Too many searches in short time
2. **Unusual patterns** — Systematic `filetype:` queries
3. **User-Agent anomalies** — Non-browser user agents
4. **IP reputation** — Known scanning IPs

### Network Signatures

```
# Google search patterns from Metagoofil:
GET /search?q=site:example.com+filetype:pdf HTTP/1.1
GET /search?q=site:example.com+filetype:doc HTTP/1.1
GET /search?q=site:example.com+filetype:docx HTTP/1.1

# Download patterns:
GET /path/to/document.pdf HTTP/1.1
User-Agent: Mozilla/5.0 (compatible; Python-urllib/3.x)
```

### Log Analysis

```bash
# Check web server logs for document downloads
grep -E "(\.pdf|\.doc|\.docx|\.ppt|\.pptx|\.xls|\.xlsx)" /var/log/apache2/access.log

# Look for unusual download patterns
awk '{print $1}' /var/log/apache2/access.log | sort | uniq -c | sort -rn | head -20

# Check for automated user agents
grep -i "python\|urllib\|metagoofil" /var/log/apache2/access.log
```

### IDS/IPS Rules

```
# Snort/Suricata rules to detect Metagoofil-like activity
alert http $EXTERNAL_NET any -> $HOME_NET any (
    msg:"Possible Document Metadata Harvesting";
    content:"GET"; http_method;
    content:"filetype="; http_uri;
    content:"site="; http_uri;
    threshold:type both,track by_src,count 10,seconds 60;
    sid:1000100; rev:1;
)

alert http $EXTERNAL_NET any -> $HOME_NET any (
    msg:"Bulk Document Download Detected";
    content:"GET"; http_method;
    pcre:"/\.(pdf|doc|docx|ppt|pptx|odt|xls|xlsx)$/i";
    threshold:type both,track by_src,count 20,seconds 300;
    sid:1000101; rev:1;
)
```

## Mitigation Strategies

### 1. Document Sanitization

```bash
#!/bin/bash
# sanitize_metadata.sh - Remove metadata from documents before publishing

INPUT_DIR="$1"
OUTPUT_DIR="$2"

if [ -z "$INPUT_DIR" ] || [ -z "$OUTPUT_DIR" ]; then
    echo "Usage: $0 <input_dir> <output_dir>"
    exit 1
fi

mkdir -p "$OUTPUT_DIR"

find "$INPUT_DIR" -type f \( -name "*.pdf" -o -name "*.docx" -o -name "*.xlsx" \) | while read file; do
    echo "[*] Sanitizing: $file"
    filename=$(basename "$file")
    
    case "$file" in
        *.pdf)
            # Use qpdf to remove metadata
            qpdf --replace-input --no-original-object-ids "$file" 2>/dev/null || \
            cp "$file" "$OUTPUT_DIR/$filename"
            ;;
        *.docx|*.xlsx)
            # Use python-docx to remove metadata
            python3 -c "
from docx import Document
doc = Document('$file')
doc.core_properties.author = ''
doc.core_properties.title = ''
doc.core_properties.subject = ''
doc.core_properties.keywords = ''
doc.core_properties.comments = ''
doc.save('$OUTPUT_DIR/$filename')
" 2>/dev/null || cp "$file" "$OUTPUT_DIR/$filename"
            ;;
        *)
            cp "$file" "$OUTPUT_DIR/$filename"
            ;;
    esac
done

echo "[*] Sanitization complete. Output in: $OUTPUT_DIR"
```

### 2. Web Server Configuration

```apache
# Apache configuration to limit document access
<VirtualHost *:80>
    ServerName example.com
    
    <!-- Block common file types from being indexed -->
    <Directory /var/www/html/documents>
        Options -Indexes
        AllowOverride None
        
        <!-- Restrict access to specific IPs -->
        Require ip 192.168.1.0/24
    </Directory>
    
    <!-- Block file type queries -->
    RewriteEngine On
    RewriteCond %{QUERY_STRING} filetype= [NC]
    RewriteRule .* - [F,L]
</VirtualHost>
```

```nginx
# Nginx configuration
server {
    listen 80;
    server_name example.com;
    
    # Block document directory indexing
    location /documents/ {
        autoindex off;
        allow 192.168.1.0/24;
        deny all;
    }
    
    # Block file type queries
    if ($query_string ~* "filetype=") {
        return 403;
    }
}
```

### 3. Rate Limiting

```bash
# iptables rate limiting
iptables -A INPUT -p tcp --dport 80 -m limit --limit 100/min --limit-burst 200 -j ACCEPT
iptables -A INPUT -p tcp --dport 80 -j DROP

# Fail2Ban for web requests
cat > /etc/fail2ban/filter.d/metagoofil.conf << 'EOF'
[Definition]
failregex = ^<HOST> -.*"(GET|POST).*filetype=.*$
            ^<HOST> -.*"(GET|POST).*site=.*filetype=.*$
ignoreregex =
EOF

# Add to jail.local
cat >> /etc/fail2ban/jail.local << 'EOF'
[metagoofil]
enabled = true
filter = metagoofil
logpath = /var/log/apache2/access.log
maxretry = 10
findtime = 600
bantime = 3600
EOF

systemctl restart fail2ban
```

### 4. Document Management System (DMS)

```bash
# Use a DMS with access controls instead of direct file hosting
# Example: Nextcloud with metadata stripping

# Install Nextcloud
docker run -d \
    --name nextcloud \
    -p 8080:80 \
    -v /data/nextcloud:/var/www/html \
    nextcloud

# Configure metadata stripping in Nextcloud
# Settings → Admin → Additional settings → Document metadata
# Enable: "Strip metadata from documents on upload"
```

### 5. Honeypot Documents

```bash
# Create decoy documents to detect Metagoofil
# These documents contain tracking beacons

cat > create_honeypot.py << 'EOF'
from docx import Document
from docx.shared import Pt
import hashlib

def create_honeypot_doc(target_info, output_file):
    """Create a document with embedded tracking information"""
    doc = Document()
    
    # Add visible content
    doc.add_heading('Confidential Internal Report', 0)
    doc.add_paragraph('This document contains sensitive information.')
    doc.add_paragraph(f'Target: {target_info["domain"]}')
    
    # Add invisible tracking (steganography)
    # In reality, you'd embed a unique identifier
    tracking_id = hashlib.md5(target_info['domain'].encode()).hexdigest()
    
    # Save with metadata
    doc.core_properties.author = f'Honeypot-{tracking_id}'
    doc.core_properties.title = f'Internal Report - {target_info["domain"]}'
    doc.core_properties.subject = 'Confidential'
    
    doc.save(output_file)
    print(f"[*] Created honeypot: {output_file}")
    print(f"[*] Tracking ID: {tracking_id}")

if __name__ == '__main__':
    create_honeypot_doc(
        {'domain': 'example.com'},
        'honeypot_report.docx'
    )
EOF
```

## Blue Team Response

### Detection Checklist

1. **Monitor Google Search Console** — Look for unusual crawl patterns
2. **Check web server logs** — Search for bulk document downloads
3. **Review DNS logs** — Identify automated query patterns
4. **Analyze network traffic** — Look for scraping patterns
5. **Monitor document access** — Track who downloads what

### Response Steps

1. **Identify the source** — Find the IP address conducting the reconnaissance
2. **Block the IP** — Update firewall rules
3. **Review exposed documents** — Assess what information was leaked
4. **Sanitize documents** — Remove metadata from publicly available files
5. **Implement controls** — Add rate limiting and access controls
6. **Document the incident** — Record timeline and actions taken

### Forensic Investigation

```bash
# Check for evidence of Metagoofil activity
echo "=== Investigating Metagoofil Activity ==="

# 1. Check web server logs
grep -E "filetype=" /var/log/apache2/access.log | head -20

# 2. Check for bulk downloads
awk '{print $1}' /var/log/apache2/access.log | sort | uniq -c | sort -rn | head -10

# 3. Check for automated user agents
grep -i "python\|urllib\|bot\|crawler" /var/log/apache2/access.log | head -20

# 4. Review firewall logs
grep "DROP" /var/log/ufw.log | tail -20

# 5. Check for document access patterns
find /var/www/html -name "*.pdf" -o -name "*.doc" | xargs ls -la | sort -k6,7
```

---

# Chapter 8: Troubleshooting

## Common Errors

### Error: "Google has blocked your queries"

**Cause:** Too many requests in short time period.

**Solution:**
```bash
# 1. Wait 24 hours for the block to expire

# 2. Use a different IP (VPN/proxy)
export http_proxy="http://your-vpn-proxy:8080"
metagoofil -d target.com -f pdf -o results/

# 3. Reduce query rate
metagoofil -d target.com -f pdf -o results/ -d 30  # 30 second delay

# 4. Use fewer file types
metagoofil -d target.com -f pdf -o results/  # Only PDFs
```

### Error: "No results found"

**Cause:** Target domain has no publicly indexed documents.

**Solution:**
```bash
# 1. Verify domain exists
whois example.com

# 2. Check if documents are actually indexed
google search "site:example.com filetype:pdf"

# 3. Try alternative domains
metagoofil -d www.example.com -f pdf -o results/
metagoofil -d api.example.com -f pdf -o results/

# 4. Check for subdomains
subfinder -d example.com
```

### Error: "Permission denied: output directory"

**Cause:** Insufficient permissions to write to the output directory.

**Solution:**
```bash
# 1. Create directory with proper permissions
mkdir -p ~/metagoofil_results
chmod 755 ~/metagoofil_results

# 2. Use a directory you own
metagoofil -d example.com -f pdf -o ~/results/

# 3. Run with sudo (not recommended for security reasons)
sudo metagoofil -d example.com -f pdf -o /opt/results/
```

### Error: "pdfinfo: command not found"

**Cause:** Poppler utilities not installed.

**Solution:**
```bash
# Install poppler-utils
sudo apt install poppler-utils

# Verify installation
pdfinfo --version
```

### Error: "exiftool: command not found"

**Cause:** ExifTool not installed.

**Solution:**
```bash
# Install exiftool
sudo apt install libimage-exiftool-perl

# Verify installation
exiftool -ver
```

### Error: "ModuleNotFoundError: No module named 'olefile'"

**Cause:** Python olefile module not installed.

**Solution:**
```bash
# Install olefile
pip3 install olefile

# Or install all requirements
pip3 install -r /usr/share/metagoofil/requirements.txt
```

### Error: "Connection timeout" or "SSL Error"

**Cause:** Network connectivity issues.

**Solution:**
```bash
# 1. Check internet connection
ping google.com

# 2. Test DNS resolution
nslookup example.com

# 3. Check proxy settings
echo $http_proxy
echo $https_proxy

# 4. Increase timeout
metagoofil -d example.com -f pdf -o results/ --timeout 60

# 5. Disable SSL verification (not recommended for production)
export PYTHONHTTPSVERIFY=0
```

### Error: "Memory error" or "Out of memory"

**Cause:** Too many documents being processed simultaneously.

**Solution:**
```bash
# 1. Limit the number of results
metagoofil -d example.com -f pdf -l 50 -o results/

# 2. Process in batches
metagoofil -d example.com -f pdf -l 25 -o batch1/
metagoofil -d example.com -f pdf -l 25 -o batch2/ -s 25

# 3. Increase swap space
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```

## Performance Optimization

### Speed Up Scanning

```bash
# 1. Use fewer file types (faster search)
metagoofil -d example.com -f pdf -o results/  # Only PDFs

# 2. Limit results
metagoofil -d example.com -f pdf -l 50 -o results/

# 3. Use faster network connection
# Connect to a closer Google server or use a faster VPN

# 4. Run during off-peak hours
# Google rate limits are stricter during peak hours
```

### Optimize Resource Usage

```bash
# 1. Limit CPU usage
cpulimit -l 50 -p $(pgrep -f metagoofil)

# 2. Limit memory usage
ulimit -v 2000000

# 3. Run in background with logging
nohup metagoofil -d example.com -f pdf -o results/ > scan.log 2>&1 &

# 4. Monitor resource usage
htop -p $(pgrep -f metagoofil)
```

### Parallel Processing

```bash
#!/bin/bash
# parallel_scan.sh - Run multiple Metagoofil instances

DOMAINS=("example.com" "target.com" "victim.org")
MAX_PARALLEL=3

for domain in "${DOMAINS[@]}"; do
    # Run in background with resource limits
    (
        ulimit -v 1000000
        cpulimit -l 30 -p $$ 2>/dev/null
        metagoofil -d "$domain" -f pdf -l 100 -o "results_$domain/"
    ) &
    
    # Limit parallel jobs
    while [ $(jobs -r | wc -l) -ge $MAX_PARALLEL ]; do
        sleep 5
    done
done

wait
echo "[*] All scans complete"
```

## FAQ

### Q: Is Metagoofil legal to use?
**A:** Yes, Metagoofil is a legitimate OSINT tool. However, scraping and downloading documents from targets without authorization may violate terms of service or laws. Always get written permission before conducting reconnaissance.

### Q: How accurate is the metadata extraction?
**A:** Metadata extraction is generally accurate for standard file formats. However:
- PDF metadata can be manually edited or stripped
- Some documents may have corrupted metadata
- Not all metadata fields are present in every document

### Q: Can Metagoofil find documents behind authentication?
**A:** No, Metagoofil only finds publicly indexed documents. It cannot access:
- Password-protected documents
- Documents behind login pages
- Documents in private repositories

### Q: How do I avoid getting blocked by Google?
**A:** Several strategies:
1. Use a VPN or proxy to rotate IPs
2. Add delays between requests (30+ seconds)
3. Limit the number of queries
4. Use different user agents
5. Run during off-peak hours

### Q: Can Metagoofil extract metadata from non-standard file types?
**A:** Metagoofil supports common formats (PDF, DOC, DOCX, PPT, PPTX, ODT, XLS, XLSX). For other formats, use exiftool directly:
```bash
exiftool -json unusual_file.xyz
```

### Q: How do I process large numbers of documents?
**A:** Use batch processing:
```bash
# Split into smaller batches
metagoofil -d example.com -f pdf -l 100 -o batch1/
metagoofil -d example.com -f pdf -l 100 -o batch2/ -s 100
metagoofil -d example.com -f pdf -l 100 -o batch3/ -s 200

# Combine results
cat batch1/results.csv batch2/results.csv batch3/results.csv > combined.csv
```

### Q: Can I customize the Google dorks used?
**A:** Not directly through the command line. However, you can modify the source code to customize search queries. Edit the `metagoofil.py` file and look for the search query construction.

### Q: What's the difference between Metagoofil and other metadata tools?
**A:** Metagoofil is specialized for:
1. **Automated discovery** via Google dorking
2. **Bulk processing** of multiple documents
3. **Structured reporting** of metadata

For single-file analysis, use exiftool directly. For web-based metadata, use photon or other crawlers.

---

# Resources

## Official Documentation
- [Metagoofil GitHub](https://github.com/laramies/metagoofil)
- [Edge-Security](https://www.edge-security.com/)
- [Kali Linux Metagoofil Package](https://www.kali.org/tools/metagoofil/)

## Tutorials
- [Metagoofil Tutorial - SecurityTube](https://www.securitytube.net/)
- [OSINT Framework](https://osintframework.com/)
- [Bellingcat Online Investigation Toolkit](https://docs.google.com/document/d/1BfDPJiK1KvqaUpKlJg7kEe8UgUPwzAI3S3cKlS5aI_0/)

## Books
- "Open Source Intelligence Techniques" by Michael Bazzell
- "Intelligence Driven Incident Response" by Scott J. Roberts
- "Digital Forensics with Kali Linux" by Shiva V. Parasram

## Related Tools
- **exiftool** — Universal metadata extraction
- **pdfinfo** — PDF-specific metadata
- **photon** — Fast web crawler for OSINT
- **theharvester** — Email and subdomain harvesting
- **spiderfoot** — OSINT automation framework

## Practice Resources
- [Metagoofil Test Files](https://github.com/laramies/metagoofil/tree/master/test)
- [OSINT Challenges](https://www.osintchallenges.com/)
- [TryHackMe OSINT Rooms](https://tryhackme.com/)

## Certifications That Use Metagoofil
- OSCP (Offensive Security Certified Professional)
- CEH (Certified Ethical Hacker)
- OSINT Professional Certification
- GIAC Open Source Intelligence (GOSI)
