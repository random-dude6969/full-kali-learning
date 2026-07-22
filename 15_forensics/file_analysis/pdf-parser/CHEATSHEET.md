# PDF-Parser — Quick Reference Cheatsheet

## Essential Commands

```bash
# Show PDF structure overview
python3 pdf-parser.py document.pdf

# Show metadata
python3 pdf-parser.py -m document.pdf

# Find JavaScript
python3 pdf-parser.py -j document.pdf

# Find embedded files
python3 pdf-parser.py -f document.pdf

# Extract and decode streams
python3 pdf-parser.py -e -d document.pdf

# Show all objects
python3 pdf-parser.py -a document.pdf

# Count objects
python3 pdf-parser.py -c document.pdf
```

## Installation

```bash
# From pip
pip3 install pdf-parser

# From Didier Stevens' suite
git clone https://github.com/DidierStevens/DidierStevensSuite.git
cd DidierStevensSuite

# Verify
python3 pdf-parser.py --version
```

## Key Options

| Option | Description |
|--------|-------------|
| `-m` | Show metadata |
| `-j` | Find JavaScript |
| `-f` | Find embedded files |
| `-e` | Extract streams |
| `-d` | Decode streams |
| `-a` | Show all objects |
| `-c` | Count objects |
| `-s <type>` | Search for object type |
| `-o <id>` | Show specific object |
| `-r` | Show raw content |

## Common Scenarios

### Quick Security Check
```bash
python3 pdf-parser.py -j -f document.pdf
```

### Malicious PDF Analysis
```bash
# Full analysis
python3 pdf-parser.py -m -j -f -e -d document.pdf

# Extract JavaScript
python3 pdf-parser.py -s JavaScript -e document.pdf

# Extract embedded files
python3 pdf-parser.py -f -e -d -o output/ document.pdf
```

### Batch Analysis
```bash
for pdf in *.pdf; do
    echo "=== $pdf ==="
    python3 pdf-parser.py -j "$pdf"
    python3 pdf-parser.py -f "$pdf"
done
```

## PDF Object Types

| Type | Description | Risk |
|------|-------------|------|
| JavaScript | Embedded JS code | High |
| EmbeddedFile | Attached files | High |
| OpenAction | Auto-run action | High |
| AcroForm | Interactive forms | Medium |
| RichMedia | Flash content | High |
| Page | Document page | Low |
| Font | Text fonts | Low |

## Workflow

### Malicious PDF Analysis

```
1. Hash:      md5sum document.pdf
2. Metadata:  python3 pdf-parser.py -m document.pdf
3. JavaScript: python3 pdf-parser.py -j document.pdf
4. Embedded:  python3 pdf-parser.py -f document.pdf
5. Streams:   python3 pdf-parser.py -e -d document.pdf
6. Extract:   Extract suspicious objects
7. Analyze:   Analyze extracted content
8. Report:    Document findings
```

## Integration

```bash
# With pdfid (quick identification)
pdfid document.pdf

# With YARA (malware rules)
yara -r rules/ document.pdf

# With pdftotext (text extraction)
pdftotext document.pdf text.txt

# With strings (string extraction)
strings document.pdf | head -50
```

## Suspicious Indicators

```bash
# Check for JavaScript
python3 pdf-parser.py -j document.pdf

# Check for obfuscation patterns
python3 pdf-parser.py -a document.pdf | grep -i "eval\|unescape\|fromCharCode"

# Check for auto-open actions
python3 pdf-parser.py -a document.pdf | grep -i "OpenAction\|AA"

# Check for embedded files
python3 pdf-parser.py -f document.pdf
```

## Troubleshooting

| Problem | Solution |
|---------|----------|
| Cannot parse PDF | Check if valid PDF with `file` |
| No JavaScript found | Try with `-r` for raw content |
| Memory error | Use machine with more RAM |
| Obfuscated code | Extract and analyze manually |

## Comparison

| Tool | Best For | Speed |
|------|----------|-------|
| PDF-Parser | Detailed analysis | Medium |
| pdfid | Quick identification | Fast |
| PDFStreamDumper | Stream analysis | Medium |
| OleTools | Office documents | Fast |

---

*Quick reference for digital forensics examinations.*
*Always verify procedures against your organization's forensic standards.*
