# Pdfid — Quick Reference Cheatsheet

## Essential Commands

```bash
# Quick PDF scan
python3 pdfid.py document.pdf

# Scan directory recursively
python3 pdfid.py -s /path/to/pdfs/

# List plugins
python3 pdfid.py -l

# Use specific plugin
python3 pdfid.py -p plugin_name document.pdf

# Force non-PDF analysis
python3 pdfid.py -f document.pdf

# Quiet mode
python3 pdfid.py -q document.pdf
```

## Installation

```bash
# From pip
pip3 install pdfid

# From Didier Stevens' suite
git clone https://github.com/DidierStevens/DidierStevensSuite.git
cd DidierStevensSuite

# Verify
python3 pdfid.py --version
```

## Key Keywords

| Keyword | Risk | Description |
|---------|------|-------------|
| JavaScript | High | Embedded JS code |
| OpenAction | High | Auto-run action |
| AA | High | Additional actions |
| Launch | High | Launch programs |
| RichMedia | High | Flash content |
| AcroForm | Medium | Interactive forms |
| EmbeddedFile | Medium | Attached files |
| SubmitForm | Medium | Form submission |
| URI | Low | External links |

## Common Scenarios

### Quick Malware Check
```bash
python3 pdfid.py suspicious.pdf | grep -E "JavaScript|OpenAction|Launch"
```

### Batch Triage
```bash
for pdf in *.pdf; do
    echo "=== $pdf ==="
    python3 pdfid.py "$pdf"
done
```

### Find Suspicious PDFs
```bash
find /evidence/ -name "*.pdf" -exec python3 pdfid.py {} \; | \
    grep -B1 "JavaScript.*[1-9]"
```

## Workflow

### Triage Process

```
1. Collect:     Find all PDFs
2. Hash:        md5sum *.pdf
3. Triage:      python3 pdfid.py *.pdf
4. Filter:      Find PDFs with high-risk features
5. Analyze:     Use PDF-Parser on suspicious files
6. Extract:     Extract malicious content
7. Report:      Document findings
```

## Risk Assessment

### High Risk (Immediate Analysis)
- JavaScript present
- OpenAction present
- Launch action
- RichMedia (Flash)

### Medium Risk (Review)
- AcroForm with scripts
- EmbeddedFile
- SubmitForm

### Low Risk (Standard)
- URI links
- Standard PDF features
- No suspicious keywords

## Integration

```bash
# With PDF-Parser (detailed analysis)
python3 pdf-parser.py -j suspicious.pdf

# With YARA (malware rules)
yara -r rules/ suspicious.pdf

# With ClamAV (antivirus)
clamscan suspicious.pdf

# With OleTools (Office analysis)
oleid suspicious.pdf
```

## Batch Processing

### Find Suspicious PDFs
```bash
find /evidence/ -name "*.pdf" | while read pdf; do
    if python3 pdfid.py "$pdf" 2>/dev/null | grep -qE "JavaScript.*[1-9]|OpenAction.*[1-9]"; then
        echo "SUSPICIOUS: $pdf"
    fi
done
```

### Generate Report
```bash
echo "PDF Triage Report" > report.txt
for pdf in *.pdf; do
    echo "--- $pdf ---" >> report.txt
    python3 pdfid.py "$pdf" >> report.txt
    echo "" >> report.txt
done
```

## Troubleshooting

| Problem | Solution |
|---------|----------|
| Not a PDF | Check with `file` command |
| No output | Verify file exists and is readable |
| Permission denied | Check file permissions |
| False positives | Use PDF-Parser for confirmation |

---

*Quick reference for digital forensics examinations.*
*Always verify procedures against your organization's forensic standards.*
