# ReadPST — Quick Reference Cheatsheet

## Essential Commands

```bash
# Show PST file structure
readpst archive.pst

# Extract emails in Mbox format
readpst -e -o output/ archive.pst

# Extract as MSG files
readpst -M -o output/ archive.pst

# Recursive extraction (all folders)
readpst -r -e -o output/ archive.pst

# Diagnostic mode
readpst -D archive.pst

# Quiet mode
readpst -q -e -o output/ archive.pst
```

## Installation

```bash
# Kali Linux (pre-installed)
sudo apt install readpst libpff-utils

# Verify
readpst -V
```

## Key Options

| Option | Description |
|--------|-------------|
| `-e` | Export to Mbox format |
| `-M` | Output as MSG files |
| `-r` | Process subfolders recursively |
| `-o <dir>` | Output directory |
| `-D` | Diagnostic mode |
| `-q` | Quiet mode |
| `-h` | Help |

## Output Formats

| Format | Flag | Use Case |
|--------|------|----------|
| Mbox | `-e` | Email client import (Thunderbird) |
| MSG | `-M` | Individual file analysis |

## Common Scenarios

### Basic Extraction
```bash
readpst -e -o output/ archive.pst
```

### Complete Forensic Extraction
```bash
readpst -r -e -M -o output/ archive.pst
```

### Analyze OST File
```bash
readpst -e -r -o output/ exchange_cache.ost
```

### Extract Specific Folder
```bash
readpst -e -o output/ archive.pst
# Then navigate to specific folder in output/
```

## Workflow

### Forensic Analysis

```
1. Locate:    find /evidence/ -name "*.pst" -o -name "*.ost"
2. Copy:      cp --preserve=all source.pst /evidence/
3. Hash:      md5sum /evidence/source.pst
4. Structure: readpst /evidence/source.pst
5. Extract:   readpst -r -e -M -o /evidence/output/ /evidence/source.pst
6. Analyze:   Import Mbox into Thunderbird or review MSG files
7. Timeline:  Create timeline from email dates
8. Report:    Document all findings
```

## Output Structure

### Mbox Format
```
output/
├── Inbox
├── Sent
├── Deleted Items
└── ...
```

### MSG Format
```
output/
├── Inbox/
│   ├── message1.msg
│   └── ...
├── Sent/
└── ...
```

## Integration

```bash
# With Vinetto (comparison)
vinetto -e -o vinetto_output/ archive.pst

# With Autopsy (GUI analysis)
# Import Mbox files into Autopsy

# With Thunderbird (email review)
# File → Import → Mbox format

# With YARA (malware scanning)
yara -r rules/ output/

# With ExifTool (metadata extraction)
find output/ -name "*.msg" -exec exiftool {} \;
```

## Email Analysis

```bash
# Find emails from specific sender
grep -rl "From:.*suspect@evil.com" output/

# Extract IP addresses from headers
grep -rh "Received:" output/ | \
    grep -oE '[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+' | sort -u

# List all email subjects
grep -rh "^Subject:" output/ | sort
```

## PST vs OST

| Feature | PST | OST |
|---------|-----|-----|
| Purpose | Local storage | Exchange cache |
| Access | Standalone | Requires Exchange |
| Location | User-defined | Default location |
| Forensic Value | High | High |

## Troubleshooting

| Problem | Solution |
|---------|----------|
| Permission denied | Use `sudo` |
| Cannot parse PST | Check with `file`, try `-D` |
| No emails extracted | Check structure with `readpst archive.pst` |
| Memory error | Use machine with more RAM |
| Large PST slow | Process in chunks |

---

*Quick reference for digital forensics examinations.*
*Always verify procedures against your organization's forensic standards.*
