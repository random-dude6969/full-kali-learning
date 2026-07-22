# Vinetto — Quick Reference Cheatsheet

## Essential Commands

```bash
# Show PST metadata
vinetto archive.pst

# List folder structure
vinetto -f archive.pst

# Extract emails
vinetto -e -o output/ archive.pst

# Extract attachments
vinetto -a -o output/ archive.pst

# Full extraction (emails + attachments)
vinetto -e -a -o output/ archive.pst

# Show timestamps
vinetto -t archive.pst
```

## Installation

```bash
# Kali Linux
sudo apt install vinetto

# From source
git clone https://github.com/misterx/vinetto.git
cd vinetto && pip3 install -r requirements.txt
python3 setup.py install
```

## Key Options

| Option | Description |
|--------|-------------|
| `-e` | Extract email messages |
| `-a` | Extract attachments |
| `-f` | List folder structure |
| `-m` | Display metadata only |
| `-t` | Show timestamps |
| `-o <dir>` | Output directory |
| `-v` | Verbose output |

## Common Scenarios

### Analyze PST File
```bash
vinetto archive.pst
vinetto -f archive.pst
vinetto -t archive.pst
```

### Extract All Data
```bash
mkdir -p output/
vinetto -e -a -o output/ archive.pst
```

### Extract Emails Only
```bash
vinetto -e -o emails/ archive.pst
```

### Extract Attachments Only
```bash
vinetto -a -o attachments/ archive.pst
```

## Workflow

### Forensic Analysis

```
1. Locate:    find /evidence/ -name "*.pst"
2. Copy:      cp --preserve=all source.pst /evidence/
3. Hash:      md5sum /evidence/source.pst
4. Metadata:  vinetto /evidence/source.pst
5. Folders:   vinetto -f /evidence/source.pst
6. Extract:   vinetto -e -a -o /evidence/output/ /evidence/source.pst
7. Analyze:   Review extracted emails and attachments
8. Report:    Document all findings
```

## Output Structure

```
output/
├── emails/
│   ├── 00000001.eml
│   └── ...
├── attachments/
│   ├── document.pdf
│   └── image.jpg
└── metadata.txt
```

## Integration

```bash
# With readpst (compare results)
readpst -o readpst_output/ archive.pst

# With Autopsy (GUI analysis)
# Import extracted data into Autopsy

# With YARA (malware scanning)
yara -r rules/ output/attachments/

# With ExifTool (metadata extraction)
find output/ -name "*.jpg" -exec exiftool {} \;
```

## Email Analysis

```bash
# Find emails from specific sender
grep -l "From:.*suspect@evil.com" output/emails/*.eml

# Extract IP addresses from headers
grep -h "Received:" output/emails/*.eml | \
    grep -oE '[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+' | sort -u

# List all email subjects
grep -h "^Subject:" output/emails/*.eml | sort
```

## Troubleshooting

| Problem | Solution |
|---------|----------|
| Permission denied | Use `sudo` or fix permissions |
| Cannot parse PST | Check file validity with `file` |
| No emails extracted | Check folder structure with `-f` |
| Memory error | Use machine with more RAM |
| Corrupted PST | Try Microsoft's scanpst tool |

## Comparison

| Tool | Best For | GUI |
|------|----------|-----|
| Vinetto | Quick analysis, Python | No |
| readpst | Batch processing | No |
| Autopsy | Full investigation | Yes |
| libpff | Low-level parsing | No |

---

*Quick reference for digital forensics examinations.*
*Always verify procedures against your organization's forensic standards.*
