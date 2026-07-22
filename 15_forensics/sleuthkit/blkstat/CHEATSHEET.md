# blkstat Cheatsheet — Display Block Statistics

## Quick Command

```bash
blkstat [options] image block_address
```

## Installation

```bash
sudo apt install sleuthkit
```

## Most Common Commands

```bash
# Examine a specific block
blkstat -f ext4 -o 2048 disk.img 2048

# NTFS block
blkstat -f ntfs -o 63 disk.img 1000

# Verbose output
blkstat -v -f ext4 -o 2048 disk.img 2048
```

## Options Reference

| Option | Description |
|--------|-------------|
| `-f fstype` | Filesystem type (ntfs, ext4, fat, etc.) |
| `-i imgtype` | Image format (raw, ewf, aff, vmdk) |
| `-b SIZE` | Sector size in bytes |
| `-o OFFSET` | Offset to filesystem in sectors |
| `-k PASS` | Decryption password |
| `-v` | Verbose output |
| `-V` | Print version |

## Output Fields

| Field | Meaning |
|-------|---------|
| Block Size | Bytes per block |
| Block | Block number examined |
| Allocated/Unallocated | Block status |
| Inode | Owning inode (if allocated) |
| MFT Entry | NTFS Master File Table entry |
| Attribute | NTFS attribute type |

## Block Size Reference

| Filesystem | Default Block Size |
|------------|-------------------|
| ext2/3/4 | 4096 (4 KB) |
| NTFS | 4096 (4 KB) |
| FAT32 | 4096 (4 KB) |
| HFS+ | 4096 (4 KB) |

## Quick Workflow

```bash
# 1. Find which block to check
# (from istat, fls, or blkls output)

# 2. Examine the block
blkstat -f ntfs -o 63 disk.img 2048

# 3. View block contents
blkcat -f ntfs -o 63 disk.img 2048 > block_data.bin

# 4. If allocated, trace to inode
# (inode shown in blkstat output)

# 5. Examine the owning inode
istat -f ntfs -o 63 disk.img <inode_from_blkstat>
```

## Troubleshooting

| Problem | Fix |
|---------|-----|
| Block not found | Check filesystem bounds with `fsstat` |
| Wrong output | Specify `-f` and `-o` explicitly |
| Need block contents | Use `blkcat` instead of `blkstat` |
| Need block listing | Use `blkls` to list blocks |
| Block shows unallocated | File may be deleted — check with `ils -A` |
| Slow performance | Specify exact block number, avoid ranges |

## Block Analysis Reference

| Tool | Purpose | Example |
|------|---------|---------|
| `blkstat` | Block owner info | `blkstat -f ext4 -o 2048 disk.img 2048` |
| `blkcat` | Block contents | `blkcat -f ext4 -o 2048 disk.img 2048` |
| `blkls` | List blocks | `blkls -A -f ext4 -o 2048 disk.img` |
| `blkcalc` | Convert block numbers | `blkcalc -u 5000 -f ext4 -o 2048 disk.img` |
| `fsstat` | Filesystem stats | `fsstat -f ext4 -o 2048 disk.img` |
| `istat` | Inode details | `istat -f ext4 -o 2048 disk.img 12` |

## Cross-Reference Workflow

```bash
# 1. Find interesting blocks
blkls -A -f ext4 -o 2048 disk.img > unalloc.bin

# 2. Check block ownership
blkstat -f ext4 -o 2048 disk.img 5000

# 3. If allocated, find inode
# (shown in blkstat output)

# 4. Examine inode
istat -f ext4 -o 2048 disk.img <inode>

# 5. Extract file
icat -f ext4 -o 2048 disk.img <inode> > recovered.dat
```
