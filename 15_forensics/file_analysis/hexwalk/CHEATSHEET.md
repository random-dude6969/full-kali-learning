# HexWalk — Quick Reference Cheatsheet

## Essential Commands

```bash
# View file in hex
hexwalk file.bin

# View first 1024 bytes
hexwalk -l 1024 file.bin

# View at specific offset
hexwalk -o 1000 file.bin

# Search for hex pattern
hexwalk -s "FF D8 FF" file.bin

# Search for ASCII string
hexwalk -s "password" file.bin

# Compare two files
hexwalk -c file2.bin file.bin

# Export hex data
hexwalk -e output.hex file.bin
```

## Installation

```bash
# From pip
pip3 install hexwalk

# From source
git clone https://github.com/cr-marcstevens/hexwalk.git
cd hexwalk && python3 setup.py install

# Verify
hexwalk --version
```

## Key Options

| Option | Description |
|--------|-------------|
| `-o <offset>` | Start at offset |
| `-l <length>` | Number of bytes |
| `-s <pattern>` | Search for pattern |
| `-c <file>` | Compare with file |
| `-e <file>` | Export to file |
| `-b <offset>` | Add bookmark |

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
| RAR | `52 61 72 21` |

## Common Scenarios

### Identify File Type
```bash
hexwalk -l 16 file.bin
# Check first bytes against known signatures
```

### Find Hidden Data
```bash
hexwalk -s "PK" file.bin      # Find ZIP
hexwalk -s "%PDF" file.bin    # Find PDF
hexwalk -s "MZ" file.bin      # Find PE
```

### Compare Files
```bash
hexwalk -c original.bin modified.bin
```

## Hex Dump Format

```
Offset    00 01 02 03 04 05 06 07  08 09 0A 0B 0C 0D 0E 0F  ASCII
00000000  89 50 4E 47 0D 0A 1A 0A  00 00 00 0D 49 48 44 52  .PNG........IHDR
```

## Workflow

### File Analysis

```
1. View header:    hexwalk -l 64 file.bin
2. Identify type:  Compare magic bytes
3. Search:         hexwalk -s "pattern" file.bin
4. Compare:        hexwalk -c file2.bin file.bin
5. Export:         hexwalk -e analysis.hex file.bin
6. Document:       Record findings
```

## Integration

```bash
# With MissIdentify (file identification)
missidentify file.bin

# With strings (string extraction)
strings file.bin | head -50

# With binwalk (embedded files)
binwalk file.bin

# With file (type verification)
file file.bin
```

## Pattern Search

### Common Searches

```bash
# Find JPEG headers
hexwalk -s "FF D8 FF" file.bin

# Find PNG headers
hexwalk -s "89 50 4E 47" file.bin

# Find PDF headers
hexwalk -s "25 50 44 46" file.bin

# Find ZIP headers
hexwalk -s "50 4B 03 04" file.bin

# Find ELF headers
hexwalk -s "7F 45 4C 46" file.bin
```

### Wildcard Search

```bash
# Search with wildcard
hexwalk -s "FF ?? FF" file.bin
```

## Troubleshooting

| Problem | Solution |
|---------|----------|
| Permission denied | Use `sudo` or fix permissions |
| File too large | Use `-l` to limit bytes |
| No output | Verify file exists |
| Slow performance | Use offsets to focus |

---

*Quick reference for digital forensics examinations.*
*Always verify procedures against your organization's forensic standards.*
