# MissIdentify — Quick Reference Cheatsheet

## Essential Commands

```bash
# Identify file type
missidentify file.exe

# Check extension mismatch
missidentify -x file.exe

# Scan directory recursively
missidentify -r /path/to/dir/

# Find all extension mismatches
missidentify -r -x /path/to/dir/

# List available file types
missidentify -l

# Show file signatures
missidentify -s file.exe
```

## Installation

```bash
# Kali Linux
sudo apt install missidentify

# From source
git clone https://github.com/cr-marcstevens/missidentify.git
cd missidentify && make && sudo make install
```

## Key Options

| Option | Description |
|--------|-------------|
| `-r` | Recursive directory scan |
| `-x` | Check extension matches |
| `-l` | List file types |
| `-s` | Show signatures |
| `-q` | Quiet mode |
| `-a` | Show all matches |

## Common Scenarios

### Find Disguised Malware
```bash
missidentify -r -x /evidence/suspicious/ | grep -v "OK"
```

### Identify Recovered Files
```bash
for file in /evidence/carved/*; do
    missidentify "$file"
done
```

### Batch File Analysis
```bash
missidentify -r /evidence/files/ > types.txt
```

## Extension Mismatches

### What They Mean
```
file.exe → Actually a JPEG (SUSPICIOUS)
file.jpg → Actually a PDF (SUSPICIOUS)
file.pdf → Actually a ZIP (SUSPICIOUS)
```

### Output Format
```
/path/to/file.exe JPEG image data
/path/to/file.jpg PDF document
/path/to/file.pdf Zip archive data
```

## Workflow

### File Identification

```
1. Collect:     Gather all files
2. Hash:        md5sum files/*
3. Identify:    missidentify -r files/
4. Mismatches:  missidentify -r -x files/
5. Analyze:     Examine suspicious files
6. Verify:      Use file command for confirmation
7. Report:      Document findings
```

## Integration

```bash
# With file command (verification)
file suspicious_file

# With YARA (malware scanning)
yara -r rules/ suspicious_file

# With ClamAV (antivirus)
clamscan suspicious_file

# With strings (content extraction)
strings suspicious_file | head -50
```

## Common File Signatures

| Type | Magic Bytes (Hex) |
|------|-------------------|
| JPEG | `FF D8 FF E0` |
| PNG | `89 50 4E 47` |
| PDF | `25 50 44 46` |
| ZIP | `50 4B 03 04` |
| ELF | `7F 45 4C 46` |
| PE/EXE | `4D 5A` |
| GIF | `47 49 46 38` |

## Troubleshooting

| Problem | Solution |
|---------|----------|
| Permission denied | Use `sudo` or fix permissions |
| File not found | Check path, use absolute path |
| No output | Verify file exists and is readable |
| Unknown type | File may be corrupted or unique |

---

*Quick reference for digital forensics examinations.*
*Always verify procedures against your organization's forensic standards.*
