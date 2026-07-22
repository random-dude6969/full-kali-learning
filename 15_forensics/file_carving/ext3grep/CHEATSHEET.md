# ext3grep Quick Reference

## Installation
```bash
sudo apt install ext3grep
```

## Basic Commands

| Command | Description |
|---------|-------------|
| `ext3grep --version` | Show version |
| `ext3grep --help` | Show help |
| `ext3grep /dev/sda1 --restore-all` | Recover all deleted files |
| `ext3grep /dev/sda1 --restore-file FILE` | Recover specific file |
| `ext3grep /dev/sda1 --list-deleted-inodes` | List deleted inodes |
| `ext3grep /dev/sda1 --list-deleted-entries` | List deleted directory entries |
| `ext3grep /dev/sda1 --show-journal` | Show journal entries |
| `ext3grep /dev/sda1 --superblock` | Show superblock info |

## Key Flags

| Flag | Description |
|------|-------------|
| `--restore-all` | Recover all deleted files |
| `--restore-file PATH` | Recover specific file |
| `--output-dir DIR` | Set output directory |
| `--after TIMESTAMP` | Filter by time (after) |
| `--before TIMESTAMP` | Filter by time (before) |
| `--list-deleted-inodes` | Show deleted inodes |
| `--list-deleted-entries` | Show deleted entries |
| `--details` | Show detailed info |
| `--show-journal` | Show journal entries |
| `--journal-block BLOCK` | Start journal at block |
| `--block-range START-END` | Process block range |
| `--inode INODE` | Show specific inode |
| `--block BLOCK` | Show specific block |

## Common Workflows

### Recover Files from Time Window
```bash
# Files deleted between timestamps (Unix time)
ext3grep /dev/sda1 --restore-all --after 1705312800 --before 1705316400
```

### Recover to Specific Directory
```bash
ext3grep /dev/sda1 --restore-all --output-dir /forensics/recovered
```

### Full Analysis Workflow
```bash
# 1. List deleted inodes
ext3grep /dev/sda1 --list-deleted-inodes --details > deleted.txt

# 2. List deleted entries
ext3grep /dev/sda1 --list-deleted-entries > entries.txt

# 3. Recover all
ext3grep /dev/sda1 --restore-all --output-dir /recovery

# 4. Export journal
ext3grep /dev/sda1 --show-journal > journal.txt
```

### Large Filesystem Recovery
```bash
# Process in chunks
ext3grep /dev/sda1 --restore-all --block-range 0-1000000 --output-dir /chunk1
ext3grep /dev/sda1 --restore-all --block-range 1000000-2000000 --output-dir /chunk2
```

## Recovered Files Location
- Default: `./RESTORED_FILES/` in current directory
- Custom: Use `--output-dir /path/to/output`

## Time Filters (Unix Timestamps)
```bash
# Convert date to timestamp
date -d "2024-01-15 14:00:00" +%s
# Output: 1705312800

# Recover files from last 24 hours
AFTER=$(date -d "24 hours ago" +%s)
ext3grep /dev/sda1 --restore-all --after $AFTER
```

## Filesystem Requirements
- ext3 ONLY (not ext4)
- Root access required
- Read-only recommended for evidence

## Common Errors
| Error | Solution |
|-------|----------|
| "Not ext3" | Use ext4magic/extundelete for ext4 |
| "Journal not found" | Try `--journal-block` option |
| "Permission denied" | Run with `sudo` |
| "Device busy" | Unmount first: `umount /dev/sda1` |

## Quick Example
```bash
# Complete recovery from ext3 partition
sudo ext3grep /dev/sda1 --restore-all --output-dir /evidence/recovered
ls -la /evidence/recovered/RESTORED_FILES/
```

## Tips
1. Always work on forensic images, not live systems
2. Recover to a different device than the source
3. Check filesystem type with `blkid` first
4. Use `--details` for more verbose output
5. Combine with `foremost` for additional carving
