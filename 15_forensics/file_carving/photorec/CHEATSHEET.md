# PhotoRec — Quick Reference Cheatsheet

## Essential Commands

```bash
# Interactive mode (recommended for beginners)
sudo photorec /dev/sda

# Command-line mode (all file types)
sudo photorec /cmd /dev/sda /d /output/ whole

# Only unallocated space (faster)
sudo photorec /cmd /dev/sda /d /output/ free

# Specific file types
sudo photorec /cmd /dev/sda /d /output/ /e "jpg,png,pdf" whole

# From disk image
sudo photorec /cmd image.raw /d /output/ whole

# With debug log
sudo photorec /cmd /dev/sda /d /output/ /dlog /output/debug.log whole
```

## Installation

```bash
# Kali Linux (pre-installed with TestDisk)
sudo apt install testdisk

# Verify
photorec --version
```

## Interactive Workflow

### Step-by-Step

```
1. sudo photorec /dev/sda
2. Select disk → Enter
3. Select partition → [Search]
4. Select filesystem type → Enter
5. Select [Free] or [Whole] → Enter
6. Select output directory → Enter
7. Wait for recovery to complete
8. Press Enter to exit
```

### Navigation Keys

| Key | Action |
|-----|--------|
| Arrow keys | Navigate menus |
| Enter | Select/Confirm |
| Right arrow | Next step |
| Left arrow | Previous step |
| Q | Quit |

## Command-Line Syntax

```
photorec /cmd <device> /d <output_dir> <mode> [options]
```

### Modes

| Mode | Description |
|------|-------------|
| `whole` | Scan entire partition |
| `free` | Scan unallocated space only |

### Options

| Option | Description |
|--------|-------------|
| `/d <dir>` | Output directory |
| `/e <types>` | File extension filter |
| `/dlog <file>` | Debug log file |
| `/f <fs>` | Filesystem type |

## Common Scenarios

### Recover from Formatted Drive
```bash
sudo photorec /cmd /dev/sda1 /d /output/ whole
```

### Recover Only Images
```bash
sudo photorec /cmd /dev/sda /d /output/ /e "jpg,png,gif" whole
```

### Recover from E01 Image
```bash
sudo ewfmount evidence.E01 /mnt/ewf
sudo photorec /cmd /mnt/ewf/ewf1 /d /output/ whole
sudo umount /mnt/ewf
```

### Maximum Recovery (Free + Whole)
```bash
# First pass: unallocated space
sudo photorec /cmd /dev/sda /d /output_free/ free

# Second pass: entire partition
sudo photorec /cmd /dev/sda /d /output_whole/ whole
```

## Output Structure

```
output/
├── recup_dir.1/
│   ├── f0000001.jpg
│   ├── f0000002.jpg
│   └── ...
├── recup_dir.2/
│   └── ...
└── report.txt
```

## Result Analysis

```bash
# Count by type
find output/ -type f | sed 's/.*\.//' | sort | uniq -c | sort -rn

# Verify file types
find output/ -type f -exec file {} \;

# Check sizes
find output/ -type f -exec ls -lh {} \;

# Remove empty/junk files
find output/ -type f -size 0 -delete

# Extract metadata
find output/ -name "*.jpg" -exec exiftool {} \;
```

## Integration

```bash
# With TestDisk (partition recovery first)
sudo testdisk /dev/sda
sudo photorec /dev/sda

# With The Sleuth Kit
fls -r -d image.raw > tsk_output.txt

# With Scalpel (additional carving)
scalpel image.raw -o scalpel_output/

# With ExifTool (metadata extraction)
find output/ -name "*.jpg" -exec exiftool {} \;

# With YARA (malware scanning)
yara -r rules/ output/
```

## Workflow

### Forensic Recovery

```
1. Document:      sudo fdisk -l /dev/sda
2. Write block:   sudo blockdev --setro /dev/sda
3. Image:         sudo dd if=/dev/sda of=raw.img bs=4M
4. Hash:          md5sum raw.img
5. Recover:       sudo photorec /cmd raw.img /d output/ whole
6. Organize:      Sort files by type
7. Verify:        Check file integrity
8. Hash files:    find output/ -type f -exec md5sum {} \;
9. Report:        Document all steps
```

## Troubleshooting

| Problem | Solution |
|---------|----------|
| No files recovered | Try `whole` mode, check device is readable |
| Permission denied | Use `sudo` |
| Output dir exists | Delete it first |
| Slow recovery | Use `free` mode, limit file types |
| Too many false positives | Verify with `file` command |
| Disk full | Use larger output drive |

## Comparison

| Tool | Best For | Interface |
|------|----------|-----------|
| PhotoRec | Maximum recovery, many types | ncurses/CLI |
| Scalpel | Fast, configurable | CLI |
| Foremost | Simple, quick | CLI |
| TestDisk | Partition recovery | ncurses/CLI |

## Key Differences

- **PhotoRec**: 480+ file types, filesystem-independent, resume capable
- **Scalpel**: Faster, more configurable, fewer default types
- **Foremost**: Simplest configuration, moderate file types
- **TestDisk**: Filesystem-aware, preserves filenames

---

*Quick reference for digital forensics examinations.*
*Always verify procedures against your organization's forensic standards.*
