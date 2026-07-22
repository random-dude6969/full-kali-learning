# ext4magic Quick Reference

## Installation
```bash
sudo apt install ext4magic
```

## Basic Commands

| Command | Description |
|---------|-------------|
| `ext4magic -V` | Show version |
| `ext4magic -h` | Show help |
| `ext4magic /dev/sda1 -r -o /out` | Recover all deleted files |
| `ext4magic /dev/sda1 -f FILE` | Recover specific file |
| `ext4magic /dev/sda1 -l` | List deleted inodes |
| `ext4magic /dev/sda1 -L` | List deleted directory entries |
| `ext4magic /dev/sda1 -j` | Show journal info |
| `ext4magic /dev/sda1 -a` | Show all files (including live) |

## Key Flags

| Flag | Description |
|------|-------------|
| `-r` | Recover all deleted files |
| `-f FILE` | Recover specific file |
| `-l` | List deleted inodes |
| `-L` | List deleted entries |
| `-d DATE` | Files deleted after DATE |
| `-D DATE` | Files deleted before DATE |
| `-o DIR` | Output directory |
| `-a` | Show all files |
| `-v` | Verbose output |
| `-t` | Show file types |
| `-s START` | Start block |
| `-e END` | End block |
| `-j` | Show journal info |
| `-I INODE` | Show specific inode |
| `-B BLOCK` | Show specific block |

## Common Workflows

### Recover Files from Date Range
```bash
# Files deleted between dates
ext4magic /dev/sda1 -r -d 2024-01-15 -D 2024-01-16 -o /recovery
```

### Recover to Specific Directory
```bash
ext4magic /dev/sda1 -r -o /forensics/recovered
```

### Full Analysis Workflow
```bash
# 1. List deleted inodes
ext4magic /dev/sda1 -l -d > deleted.txt

# 2. List deleted entries
ext4magic /dev/sda1 -L > entries.txt

# 3. Recover all
ext4magic /dev/sda1 -r -o /recovery

# 4. Analyze journal
ext4magic /dev/sda1 -j > journal.txt
```

### Large Filesystem Recovery
```bash
# Find total blocks
BLOCKS=$(sudo dumpe2fs /dev/sda1 | grep "Block count" | awk '{print $NF}')

# Process in chunks
for i in $(seq 0 1000000 $BLOCKS); do
    END=$((i + 1000000))
    ext4magic /dev/sda1 -r -s $i -e $END -o "/chunk_$i"
done
```

### Block Range Recovery
```bash
# Recover specific block range
ext4magic /dev/sda1 -r -s 0 -e 1000000 -o /recovery
```

## Date Format
```bash
# ext4magic uses YYYY-MM-DD format
ext4magic /dev/sda1 -r -d 2024-01-15 -D 2024-01-16
```

## Recovered Files Location
- Default: Current directory
- Custom: Use `-o /path/to/output`

## Filesystem Support
- ext3: Full support
- ext4: Full support (including extents)
- Root access required

## Common Errors
| Error | Solution |
|-------|----------|
| "Unsupported filesystem" | Check type with `blkid` |
| "Journal not found" | Use `--no-journal` option |
| "Permission denied" | Run with `sudo` |
| "Device busy" | Unmount first: `umount /dev/sda1` |

## Quick Example
```bash
# Complete recovery from ext4 partition
sudo ext4magic /dev/sda1 -r -o /evidence/recovered
ls -la /evidence/recovered/
```

## Tips
1. Always work on forensic images, not live systems
2. Recover to a different device than the source
3. Check filesystem type with `blkid` first
4. Use `-v` for more verbose output
5. Combine with `extundelete` for better coverage
6. Use `-d` and `-D` for time-based filtering
7. Check journal with `-j` before recovery
