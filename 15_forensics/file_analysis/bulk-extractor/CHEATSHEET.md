# Bulk Extractor — Quick Reference Cheatsheet

## Essential Commands

```bash
# Basic extraction
bulk_extractor -o output/ image.raw

# With parallel jobs
bulk_extractor -j 8 -o output/ image.raw

# Extract emails only
bulk_extractor -e email -o output/ image.raw

# Extract URLs only
bulk_extractor -e url -o output/ image.raw

# Extract phone numbers only
bulk_extractor -e phone -o output/ image.raw

# From device (use image first for forensics!)
sudo bulk_extractor -o output/ /dev/sda
```

## Installation

```bash
# Kali Linux (pre-installed)
sudo apt install bulk-extractor

# From source
git clone https://github.com/simsong/bulk_extractor.git
cd bulk_extractor && ./configure && make && sudo make install

# Verify
bulk_extractor --version
```

## Key Options

| Option | Description |
|--------|-------------|
| `-o <dir>` | Output directory |
| `-j <n>` | Number of parallel jobs |
| `-e <scanner>` | Enable scanner |
| `-x <scanner>` | Disable scanner |
| `-s <offset>` | Start offset |
| `-l <length>` | Length to process |
| `-q` | Quiet mode |

## Extracted Data Types

| Scanner | Output File | Description |
|---------|-------------|-------------|
| email | email.txt | Email addresses |
| phone | phone.txt | Phone numbers |
| url | url.txt | Web URLs |
| ccn | ccn.txt | Credit card numbers |
| ip | ip.txt | IP addresses |
| mac | mac.txt | MAC addresses |
| domain | domain.txt | Domain names |

## Common Scenarios

### Complete Triage
```bash
bulk_extractor -j $(nproc) -o output/ image.raw
```

### Email Analysis
```bash
bulk_extractor -e email -o output/ image.raw
cat output/email.txt | awk -F@ '{print $2}' | sort | uniq -c | sort -rn
```

### Memory Dump Analysis
```bash
bulk_extractor -o memory_analysis/ memory.dmp
cat memory_analysis/email.txt | sort -u > emails_unique.txt
```

## Output Structure

```
output/
├── email.txt           # All email addresses
├── phone.txt           # Phone numbers
├── url.txt             # URLs
├── ccn.txt             # Credit card numbers
├── ip.txt              # IP addresses
├── mac.txt             # MAC addresses
├── domain.txt          # Domain names
├── jpg/                # Carved JPEG files
├── pdf/                # Carved PDF files
├── zip/                # Carved ZIP files
└── wordlist.txt        # Extracted words
```

## Workflow

### Forensic Triage

```
1. Image:       sudo dd if=/dev/sda of=image.raw bs=4M
2. Hash:        md5sum image.raw
3. Extract:     bulk_extractor -j $(nproc) -o output/ image.raw
4. Analyze:     Review email.txt, url.txt, etc.
5. Carve:       Check carved files in jpg/, pdf/, zip/
6. Filter:      Remove false positives
7. Report:      Document findings
```

## Integration

```bash
# With The Sleuth Kit
fls -r -d image.raw > tsk_output.txt

# With Scalpel (additional carving)
scalpel image.raw -o scalpel_output/

# With YARA (malware scanning)
yara -r rules/ output/jpg/

# With ExifTool (metadata)
find output/ -name "*.jpg" -exec exiftool {} \;
```

## Result Analysis

```bash
# Count by type
wc -l output/*.txt

# Email domain analysis
cat output/email.txt | awk -F@ '{print $2}' | sort | uniq -c | sort -rn

# URL domain analysis
cat output/url.txt | grep -oP 'https?://[^/]+' | sort | uniq -c | sort -rn

# Find specific patterns
grep -i "password" output/url.txt
grep "@suspicious.com" output/email.txt
```

## Troubleshooting

| Problem | Solution |
|---------|----------|
| Permission denied | Use `sudo` |
| No space left | Use selective scanning |
| Too slow | Use `-j $(nproc)` |
| Too many false positives | Filter results manually |
| Large output | Use selective scanning |

---

*Quick reference for digital forensics examinations.*
*Always verify procedures against your organization's forensic standards.*
