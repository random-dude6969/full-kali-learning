# istat Cheatsheet — Display Inode Information

## Quick Command

```bash
istat [options] image inum
```

## Installation

```bash
sudo apt install sleuthkit
```

## Most Common Commands

```bash
# Examine an NTFS MFT entry
istat -f ntfs -o 63 disk.img 12

# Examine an ext4 inode
istat -f ext4 -o 2048 disk.img 12

# Root directory on ext4 (inode 2)
istat -f ext4 -o 2048 disk.img 2

# Root directory on NTFS (MFT entry 5)
istat -f ntfs -o 63 disk.img 5

# With timezone adjustment
istat -f ntfs -o 63 -z EST5EDT disk.img 12

# Verbose output
istat -v -f ntfs -o 63 disk.img 12
```

## Options Reference

| Option | Description |
|--------|-------------|
| `-f fstype` | Filesystem type (ntfs, ext4, fat, ufs, etc.) |
| `-i imgtype` | Image format (raw, ewf, aff, vmdk) |
| `-b SIZE` | Device sector size in bytes |
| `-o OFFSET` | Offset to filesystem in sectors |
| `-N num` | Force display of specific attribute number |
| `-r` | Show run list instead of block addresses |
| `-z ZONE` | Timezone of original system |
| `-s SECS` | Time skew in seconds |
| `-k PASS` | Decryption password for encrypted volumes |
| `-v` | Verbose output |
| `-V` | Print version |

## Output Fields

| Field | Meaning |
|-------|---------|
| Inode/MFT number | Unique identifier |
| Allocated/Unallocated | Whether the inode is in use |
| Type | File type (File, Directory, etc.) |
| Size | File size in bytes |
| Permissions | Unix-style rwx permissions |
| UID/GID | Owner and group IDs |
| Links | Hard link count |
| Accessed | Last access time |
| File Modified | Last content modification (mtime) |
| Inode Modified | Last metadata change (ctime) |
| Deleted | Deletion timestamp |
| Block Addresses | Disk blocks holding file data |

## NTFS-Specific Output

| Field | Meaning |
|-------|---------|
| `$STANDARD_INFORMATION` | Primary timestamps (modified by Windows) |
| `$FILE_NAME` | Backup timestamps (harder to alter) |
| `$DATA` | File content attribute |
| Run List | Compressed block map (LCN ranges) |
| LCN | Logical Cluster Number (block address) |

## Key Inode Numbers

### ext2/3/4

| Inode | Purpose |
|-------|---------|
| 1 | Bad block inode |
| 2 | Root directory |
| 8 | Journal |

### NTFS

| MFT Entry | Purpose |
|-----------|---------|
| 0 | MFT itself |
| 1 | MFT mirror |
| 2 | Log file |
| 3 | Volume |
| 5 | Root directory |
| 6 | Attribute list |
| 7-15 | Reserved system files |
| 32+ | User files |

## Workflow

```bash
# 1. Find the inode number
fls -f ntfs -o 63 -r disk.img
# Output: r/r 12: document.txt

# 2. Examine the inode
istat -f ntfs -o 63 disk.img 12

# 3. Extract the file
icat -f ntfs -o 63 disk.img 12 > document.txt

# 4. Find filename references
ffind -f ntfs -o 63 disk.img 12
```

## Troubleshooting

| Problem | Fix |
|---------|-----|
| Inode not found | Verify with `ils` or `fls` |
| Wrong filesystem | Specify `-f ntfs` or `-f ext4` explicitly |
| Bad block address | Filesystem may be corrupted |
| Timestamps look wrong | Use `-z` to set correct timezone |
| Zero timestamps | Inode may be corrupted or synthetic |
