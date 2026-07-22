# mmls Cheatsheet — Display Partition Table

## Quick Command

```bash
mmls [options] disk_image
```

## Installation

```bash
sudo apt install sleuthkit
```

## Most Common Commands

```bash
# Auto-detect and display partition table
mmls disk.img

# Force MBR partition table analysis
mmls -t dos disk.img

# Force GPT analysis
mmls -t gpt disk.img

# Show sizes in bytes
mmls -B disk.img

# Show only unallocated space
mmls -A disk.img

# Recurse to find nested partition tables
mmls -r disk.img

# List all supported partition types
mmls -t list

# List all supported image formats
mmls -i list
```

## Options Reference

| Option | Description |
|--------|-------------|
| `-t vstype` | Volume system type (`dos`, `gpt`, `mac`, `bsd`, `sun`) |
| `-i imgtype` | Image format (`raw`, `ewf`, `aff`, `vmdk`, `vhdi`) |
| `-b SIZE` | Sector size in bytes (default 512, use 4096 for advanced format) |
| `-o OFFSET` | Offset in sectors to partition table |
| `-B` | Print sizes in bytes instead of sectors |
| `-r` | Recurse (find nested partition tables) |
| `-a` | Show only allocated volumes |
| `-A` | Show only unallocated volumes |
| `-m` | Show metadata volumes |
| `-M` | Hide metadata volumes |
| `-v` | Verbose output |
| `-V` | Print version |

## Output Format

```
Disposition  Start        End          Length       Description
----------   ----------   ----------   ----------   -----------
000:  -----  0000000000   0000000062   0000000063   Unallocated
001:  -----  0000000063   0000409662   0000409600   NTFS (0x07)
002:  -----  0000409663   0000819262   0000409600   Linux (0x83)
003:  -----  0000819263   0000838862   0000019600   Linux Swap (0x82)
```

### Columns

| Column | Meaning |
|--------|---------|
| Disposition | Entry number + allocation status (`-----` = allocated) |
| Start | Starting sector |
| End | Ending sector |
| Length | Total sectors |
| Description | Partition type name + hex code |

## Converting Sectors to Bytes

```
Byte Offset = Sector Number × 512
```

| Sectors | Bytes | Human |
|---------|-------|-------|
| 63 | 32,256 | 31.5 KB |
| 2048 | 1,048,576 | 1 MB |
| 1048576 | 536,870,912 | 512 MB |

## Using -o Offset with Other TSK Tools

```bash
# mmls shows partition at sector 63
# Use that as -o for other tools:
fsstat -f ntfs -o 63 disk.img
fls -f ntfs -o 63 disk.img
icat -f ntfs -o 63 disk.img <inode>
ils -f ntfs -o 63 disk.img
```

## Quick Workflow

```bash
# 1. Identify partitions
mmls disk.img

# 2. Pick partition offset from output

# 3. Examine filesystem
fsstat -f <type> -o <offset> disk.img

# 4. List files
fls -f <type> -o <offset> -r -m / disk.img

# 5. Generate timeline
mactime -b body.txt -d > timeline.csv
```

## Troubleshooting

| Problem | Fix |
|---------|-----|
| Cannot determine type | Force with `-t dos` or `-t gpt` |
| Wrong sector size | Try `-b 4096` for advanced format drives |
| Only one entry shown (GPT) | Use `-t gpt` explicitly |
| Nested partitions | Use `-r` to recurse |
| Image not recognized | Check `-i list` for supported formats |
