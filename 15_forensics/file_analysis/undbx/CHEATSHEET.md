# Undbx — Quick Reference Cheatsheet

## Essential Commands

```bash
# Show DBX metadata
undbx mailbox.dbx

# Extract emails
undbx -e -o output/ mailbox.dbx

# Include deleted emails
undbx -e -d -o output/ mailbox.dbx

# Extract attachments
undbx -a -o output/ mailbox.dbx

# Full extraction (emails + deleted + attachments)
undbx -e -d -a -o output/ mailbox.dbx

# Metadata only
undbx -m mailbox.dbx
```

## Installation

```bash
# Kali Linux
sudo apt install undbx

# From source
git clone https://github.com/joachimmetz/undbx.git
cd undbx && make && sudo make install
```

## Key Options

| Option | Description |
|--------|-------------|
| `-e` | Extract email messages |
| `-d` | Include deleted emails |
| `-a` | Extract attachments |
| `-m` | Display metadata only |
| `-o <dir>` | Output directory |
| `-v` | Verbose output |
| `-r` | Process subdirectories |

## Common Scenarios

### Analyze DBX File
```bash
undbx mailbox.dbx
undbx -m mailbox.dbx
```

### Extract All Data (Including Deleted)
```bash
mkdir -p output/
undbx -e -d -a -o output/ mailbox.dbx
```

### Recover Deleted Emails
```bash
undbx -e -d -o recovered/ mailbox.dbx
```

### Batch Process Multiple Files
```bash
for dbx in *.dbx; do
    undbx -e -d -o "output_$(basename $dbx .dbx)/" "$dbx"
done
```

## Workflow

### Forensic Analysis

```
1. Locate:    find /evidence/ -name "*.dbx"
2. Copy:      cp --preserve=all source.dbx /evidence/
3. Hash:      md5sum /evidence/source.dbx
4. Metadata:  undbx /evidence/source.dbx
5. Extract:   undbx -e -d -a -o /evidence/output/ /evidence/source.dbx
6. Analyze:   Review extracted emails and attachments
7. Timeline:  Create timeline from email dates
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
# With Vinetto (PST analysis)
vinetto -e -o vinetto_output/ archive.pst

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

## DBX vs PST

| Feature | DBX | PST |
|---------|-----|-----|
| Application | Outlook Express | Outlook |
| Max Size | 2GB | 50GB |
| Recovery | Good | Good |
| Modern Use | Rare | Common |

## Troubleshooting

| Problem | Solution |
|---------|----------|
| Permission denied | Use `sudo` or fix permissions |
| Cannot parse DBX | Check file validity with `file` |
| No emails extracted | Check file size and options |
| Memory error | Use machine with more RAM |

## Comparison

| Tool | Best For | Format |
|------|----------|--------|
| Undbx | Outlook Express | DBX |
| Vinetto | Outlook PST | PST |
| readpst | Outlook (all) | PST/OST |
| libpff | Low-level parsing | PST/OST |

---

*Quick reference for digital forensics examinations.*
*Always verify procedures against your organization's forensic standards.*
