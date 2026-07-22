# extundelete Quick Reference

## Installation
```bash
sudo apt install extundelete
```

## Basic Commands

| Command | Description |
|---------|-------------|
| `extundelete --version` | Show version |
| `extundelete --help` | Show help |
| `extundelete /dev/sda1 --restore-all` | Recover all deleted files |
| `extundelete /dev/sda1 --restore-file FILE` | Recover specific file |
| `extundelete /dev/sda1 --restore-inode INODE` | Recover by inode number |
| `extundelete /dev/sda1 --list-inodes deleted` | List deleted inodes |
| `extundelete /dev/sda1 --list-inodes all` | List all inodes |

## Key Flags

| Flag | Description |
|------|-------------|
| `--restore-all` | Recover all deleted files |
| `--restore-file PATH` | Recover specific file |
| `--restore-inode INODE` | Recover specific inode |
| `--output-dir DIR` | Set output directory |
| `--list-inodes TYPE` | List inodes (deleted/all) |
| `--after TIMESTAMP` | Recover files after time |
| `--before TIMESTAMP` | Recover files before time |
| `--no-dir-links` | Don't recover dir links |
| `--no-links` | Don't recover links |

## Common Workflows

### Recover Files from Time Window
```bash
# Files deleted between timestamps (Unix time)
extundelete /dev/sda1 --restore-all --after 1705312800 --before 1705316400
```

### Recover to Specific Directory
```bash
extundelete /dev/sda1 --restore-all --output-dir /forensics/recovered
```

### Full Analysis Workflow
```bash
# 1. List deleted inodes
extundelete /dev/sda1 --list-inodes deleted > deleted.txt

# 2. Recover all
extundelete /dev/sda1 --restore-all --output-dir /recovery

# 3. Check recovered files
ls -la /recovery/RECOVERED_FILES/
```

### Recover Specific Directory
```bash
# Recover files from specific path
extundelete /dev/sda1 --restore-file "home/user/documents" --output-dir /recovery
```

### Recover by Inode
```bash
# Recover specific inode
extundelete /dev/sda1 --restore-inode 12345 --output-dir /recovery

# Recover multiple inodes
extundelete /dev/sda1 --restore-inode 12345,12346,12347
```

## Recovered Files Location
- Default: `./RECOVERED_FILES/` in current directory
- Custom: Use `--output-dir /path/to/output`

## Time Filters (Unix Timestamps)
```bash
# Convert date to timestamp
date -d "2024-01-15 14:00:00" +%s
# Output: 1705312800

# Recover files from last 24 hours
AFTER=$(date -d "24 hours ago" +%s)
extundelete /dev/sda1 --restore-all --after $AFTER
```

## Filesystem Support
- ext3: Full support
- ext4: Full support
- Root access required

## Common Errors
| Error | Solution |
|-------|----------|
| "Not ext3/ext4" | Check type with `blkid` |
| "Journal not found" | Try `--no-dir-links` option |
| "Permission denied" | Run with `sudo` |
| "Device busy" | Unmount first: `umount /dev/sda1` |

## Quick Example
```bash
# Complete recovery from ext4 partition
sudo extundelete /dev/sda1 --restore-all --output-dir /evidence/recovered
ls -la /evidence/recovered/RECOVERED_FILES/
```

## Tips
1. Always work on forensic images, not live systems
2. Recover to a different device than the source
3. Check filesystem type with `blkid` first
4. Use `--output-dir` for custom location
5. Combine with `foremost` for additional carving
6. Use `--after` and `--before` for time-based filtering
7. Check `--list-inodes deleted` before recovery
