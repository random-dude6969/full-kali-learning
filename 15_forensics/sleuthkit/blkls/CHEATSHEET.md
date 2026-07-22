# blkls Cheatsheet — List Block Entries

## Quick Command

```bash
blkls [options] image [start-stop]
```

## Installation

```bash
sudo apt install sleuthkit
```

## Most Common Commands

```bash
# Extract unallocated blocks (raw binary)
blkls -A disk.img > unalloc.bin

# List unallocated blocks (human-readable)
blkls -A -l disk.img

# Extract slack space
blkls -s disk.img > slack.bin

# List allocated blocks
blkls -a -l disk.img

# Every block including metadata
blkls -e -A disk.img
```

## Options Reference

| Option | Description |
|--------|-------------|
| `-a` | Allocated blocks only |
| `-A` | Unallocated blocks only |
| `-e` | Every block (including metadata) |
| `-l` | List format (timestamps + paths) |
| `-s` | Slack space only |
| `-f fstype` | Filesystem type |
| `-o OFFSET` | Offset to filesystem in sectors |
| `-b SIZE` | Sector size in bytes |
| `-v` | Verbose output |
| `-V` | Print version |

## Output Modes

| Mode | Flag | Output |
|------|------|--------|
| Raw (default) | (none) | Binary data to stdout |
| List | `-l` | Human-readable with timestamps |
| Slack only | `-s` | Unused portion of last blocks |

## Unallocated Block Recovery Workflow

```bash
# 1. Extract unallocated blocks
blkls -A -f ext4 -o 2048 disk.img > unalloc.bin

# 2. Search for strings
srch_strings unalloc.bin | grep -i "password"

# 3. Carve files
foremost -i unalloc.bin -o carved/

# 4. Review results
ls -la carved/
```

## Slack Space Workflow

```bash
# 1. Extract slack space
blkls -s -f ext4 -o 2048 disk.img > slack.bin

# 2. Search for residual data
srch_strings slack.bin | grep -i "secret"

# 3. Check file types
file slack.bin
```

## File Carving Integration

```bash
# blkls → foremost
blkls -A disk.img > unalloc.bin && foremost -i unalloc.bin -o carved/

# blkls → scalpel
blkls -A disk.img > unalloc.bin && scalpel -o scalpel_out/ unalloc.bin

# blkls → bulk_extractor
blkls -A disk.img > unalloc.bin && bulk_extractor unalloc.bin -o bulk/
```

## Search Patterns in Unallocated Blocks

```bash
# Email addresses
blkls -A disk.img | srch_strings | grep -E "[a-z0-9.]+@[a-z0-9.]+"

# URLs
blkls -A disk.img | srch_strings | grep -i "http"

# Passwords
blkls -A disk.img | srch_strings | grep -i "password\|passwd\|pwd"

# File signatures
blkls -A disk.img | grep -P "PK\x03\x04"    # ZIP
blkls -A disk.img | grep -P "\x89PNG"        # PNG
blkls -A disk.img | grep -P "%PDF"           # PDF
```

## Troubleshooting

| Problem | Fix |
|---------|-----|
| Output too large | Use `-l` for list format or redirect to file |
| No slack space | Filesystem may use delayed allocation |
| All zeros in output | Normal on clean filesystem — use `srch_strings` to filter |
| Wrong filesystem | Specify `-f` and `-o` explicitly |
| Need metadata blocks | Use `-e` flag |
